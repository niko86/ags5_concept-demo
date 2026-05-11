# ags5_concept — HTTP-streaming `.ags5db` demo

**Live demo:** <https://niko86.github.io/ags5_concept-demo/>

A multi-megabyte AGS5 geotechnical DuckDB file, opened entirely in your
browser. [DuckDB-wasm](https://duckdb.org/docs/api/wasm/) uses HTTP Range
requests over `httpfs` so only the bytes a given query touches travel
over the network. **No backend, no Python in the browser, no
GINT/HoleBASE license.** Open DevTools → Network and watch the partial
fetches.

This is the **public companion repo** for
[niko86/ags5_concept](https://github.com/niko86/ags5_concept) — that's
where the actual library + builder + Python tooling live. This repo
exists only to host the demo's static assets and the synthetic dataset.

## What's here

| File | Purpose |
|---|---|
| `index.html` | The entire demo. ~13 KB. Loads DuckDB-wasm from jsDelivr at runtime. |
| `large_synthetic.ags5db` (~96 MB) | Synthetic dataset: 50 boreholes × 1 sample × 1 triaxial test × 22,500 TREL readings = 1.125M rows of typed columnar geotechnical data. |
| `.nojekyll` | Suppresses GitHub Pages' Jekyll processing. |

## How the demo works

1. Page loads. The browser pulls `@duckdb/duckdb-wasm` from jsDelivr.
2. DuckDB-wasm ATTACHes `large_synthetic.ags5db` over `https://` via `httpfs`.
3. Every SQL query is compiled in-browser and serviced by HTTP Range
   requests against the file's storage segments. A `COUNT(*)` reads
   a few KB of metadata; a `SELECT * FROM g_loca LIMIT 50` reads just
   the LOCA blocks.
4. The page drives off the file's own `_spec_*` self-describing
   tables — it never needs the Python `ags5_models` registry, so any
   Phase-6.5 `.ags5db` opens the same way.

## Trying it locally

```bash
git clone https://github.com/niko86/ags5_concept-demo
cd ags5_concept-demo
python -m http.server 8000
# open http://localhost:8000
```

## Pointing at your own file

Override the dataset URL via the input on the page or a query string:

    https://niko86.github.io/ags5_concept-demo/?url=https%3A%2F%2Fexample.com%2Fmy.ags5db

The host has to provide **CORS** (`Access-Control-Allow-Origin`) and
**HTTP Range** (`Accept-Ranges: bytes`) support, which
`raw.githubusercontent.com` does natively. S3 with the right
configuration works too. Private GitHub URLs do **not** — DuckDB-wasm
can't supply auth headers.

## Source

Everything is open source under the main repo. Look in
[niko86/ags5_concept](https://github.com/niko86/ags5_concept) for:

- `demo/index.html` — the page this repo serves
- `examples/build_synthetic_demo.py` — the dataset builder
- `packages/ags5-db/` — the typed-column DuckDB writer/reader
- `packages/ags5-models/` — the registry-driven msgspec models

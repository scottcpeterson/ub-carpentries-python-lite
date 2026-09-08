# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A browser-only Python environment (JupyterLite) that serves as a fallback for learners who
can't install Python locally. It covers **both** UC Carpentries Python lessons:
`lc-python-intro` (Library Carpentry) and `python-novice-gapminder` (Software Carpentry).
It is **not** a Carpentries Workbench lesson — ignore the sandpaper/pegboard conventions in the
parent `~/projects/lessons/` CLAUDE.md files. There is no lesson prose here; the content is the
bundled data plus a welcome notebook.

Live site: <http://www.tim-dennis.com/uc-lc-python-lite/> (GitHub Pages, custom domain).
GitHub: `ucla-data-science-center/uc-carpentries-python-lite` (was `jt14den/uc-lc-python-lite`
— that older repo is stale).

## Build and preview

```bash
pip install -r requirements.txt
jupyter lite build \
  --contents content/welcome.ipynb \
  --contents content/lc-python \
  --contents content/gapminder
python -m http.server -d _output 8000   # open http://localhost:8000

# clean rebuild
rm -rf _output .jupyterlite.doit.db cache
```

One `--contents` per lesson. Each copies that dir's *contents* into the site's `files/`, so both
lessons' `data/` subfolders merge into one flat `files/data/` — which is what makes the
lesson-hardcoded paths (`data/2011_circ.csv`, `data/gapminder_gdp_oceania.csv`) work with no
prefix. Consequence: **two lessons must never ship a same-named file** or one silently wins.
Needs `micromamba`/`mamba`/`conda` on PATH for the kernel solve.

`_output/`, `cache/`, `.jupyterlite.doit.db` are git-ignored build artifacts. Most changes
don't need a local build — push to `main` and CI publishes (`.github/workflows/deploy.yml`,
also runnable via workflow_dispatch). There is no `gh-pages` branch; Pages source is
"GitHub Actions".

## The two-environment split (this is the main gotcha)

| File | Controls | Edit when |
|---|---|---|
| `environment.yml` | The in-browser WASM kernel — packages learners' code runs against. Resolved from `emscripten-forge` (channel `prefix.dev/emscripten-forge-4x`), `conda-forge` as fallback. | A learner needs a new package (e.g. `scikit-learn`). Verify it exists on emscripten-forge first — not every PyPI package is built for WASM. |
| `requirements.txt` | Build tools that run in CI only (`jupyterlite`, `jupyterlite-xeus`, `jupyter-server`), version-pinned. | Bumping the JupyterLite toolchain. `jupyterlite-xeus` caps `jupyterlite-core <0.9`. |

Preinstalled kernel packages: `pandas`, `numpy`, `matplotlib` (both lessons), `seaborn`, `plotly`
(`lc-python-intro` only). No `!` shell commands work in JupyterLite (no `!pip install`, no `!ls`).

## Content

Everything under `content/` is downloaded into the learner's browser on first load, so keep it
small. Source is organised per lesson; the build flattens it (see above).

- `content/lc-python/data/` — 12 CSVs (`2011_circ.csv` … `2022_circ.csv`, Chicago Public Library
  circulation) plus `all_years.pkl` / `df_long.pkl` (pre-built artifacts; regenerate from CSVs,
  don't hand-edit).
- `content/gapminder/data/` — Gapminder GDP CSVs, copied from
  `swcarpentry/python-novice-gapminder/episodes/data/`.
- `content/welcome.ipynb` — opens automatically on site load (`defaultUrl` in `jupyter-lite.json`).
  Test code cells against the **deployed** WASM kernel, not a desktop Jupyter; behavior differs.

## Adding / repurposing lessons

To add a lesson: new `content/<lesson>/data/` folder (check no filename collides with an existing
lesson), add `--contents content/<lesson>` to `deploy.yml` + README + this file, add packages to
`environment.yml`, update the welcome notebook. To repurpose the whole site: fork, swap `content/`,
rewrite `environment.yml`, adjust `defaultUrl`. Full maintainer guide (data refresh, toolchain
bumps, deploy troubleshooting) is in `MAINTAINING.md`.

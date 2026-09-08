# Maintaining this JupyterLite site

How to update content, change packages, and keep the deploy healthy. For an overview and learner-facing notes, see [README.md](README.md).

This site serves two lessons: Library Carpentry `lc-python-intro` and Software Carpentry `python-novice-gapminder`.

## How content is organised

Each lesson keeps its data in its own folder:

```
content/
  welcome.ipynb
  lc-python/data/     12 circulation CSVs + all_years.pkl, df_long.pkl
  gapminder/data/     Gapminder GDP CSVs
```

The build passes one `--contents` flag per lesson:

```
jupyter lite build \
  --contents content/welcome.ipynb \
  --contents content/lc-python \
  --contents content/gapminder
```

`jupyter lite build` copies the *contents* of each `--contents` directory into the site's `files/`. Both lessons' `data/` folders therefore land on top of each other as one flat `files/data/`. That's deliberate: the lessons hardcode paths like `data/2011_circ.csv` and `data/gapminder_gdp_oceania.csv`, and a flat `data/` makes those work with no prefix and no per-learner setup.

The one rule this imposes: **no two lessons may ship a file with the same name**, or one silently overwrites the other in the build. The current names don't collide. Check when you add a lesson.

## How the deploy works

`.github/workflows/deploy.yml` runs on every push to `main` (and on manual dispatch). It:

1. Installs micromamba (needed to solve the emscripten-forge kernel environment).
2. Installs the build tools from `requirements.txt`.
3. Runs the `jupyter lite build` above, producing a static site in `_output/`.
4. Uploads `_output/` as a Pages artifact and deploys it.

There's no separate `gh-pages` branch. Deployment goes straight to GitHub Pages from the artifact. Pages must be set to **Source: GitHub Actions** in the repo settings (Settings → Pages). The custom domain is configured there too.

Most pushes don't need a local build; commit the change and let CI publish. Build locally when you want to preview before pushing, or when you've touched `environment.yml` or `requirements.txt`.

## Update the lesson data

1. Add, replace, or remove files in `content/lc-python/data/` or `content/gapminder/data/`.
2. If you rename a file, update any reference to it in `content/welcome.ipynb`, and tell instructors (lesson paths are hardcoded).
3. Commit and push. CI rebuilds and redeploys.

Keep the data small. Everything is downloaded into the learner's browser on first load, so large files mean slow starts. The `.pkl` files are pre-built `lc-python-intro` artifacts; regenerate them from the CSVs if the underlying data changes, rather than editing by hand.

The Gapminder CSVs come from `swcarpentry/python-novice-gapminder/episodes/data/`. To refresh them, copy that folder's `*.csv` over `content/gapminder/data/`.

## Change the preinstalled packages

Two separate files control two separate environments. This trips people up:

* **`environment.yml`**: the **in-browser kernel** (what learners' code runs against). Edit this when learners need a new package like `scikit-learn`. Packages must exist for the Emscripten/WASM target, so they come from `emscripten-forge` (`https://prefix.dev/emscripten-forge-4x`) with `conda-forge` as fallback. Not every PyPI package is available for WASM, so check emscripten-forge before promising a package. Current set: `numpy`, `pandas`, `matplotlib` (both lessons), plus `seaborn` and `plotly` (`lc-python-intro`).
* **`requirements.txt`**: the **build tools** that run in CI (`jupyterlite`, `jupyterlite-xeus`, `jupyter-server`), version-pinned. Edit this only to bump the JupyterLite toolchain itself.

After editing `environment.yml`, push and confirm the new package imports in the deployed welcome notebook.

## Bump the JupyterLite toolchain

`requirements.txt` is pinned to a known-good set (`jupyterlite >=0.8.3,<0.9`, which ships JupyterLab 4.6). To upgrade:

1. Widen or bump the pins in `requirements.txt`.
2. `pip install -r requirements.txt` and run the build locally.
3. Serve `_output/` and run the welcome notebook end to end (kernel starts, data loads, a plot renders).
4. Commit the new pins.

`jupyterlite-xeus` currently requires `jupyterlite-core <0.9`, so don't cross that boundary without checking the xeus release notes.

## Edit the welcome notebook

`content/welcome.ipynb` is the first thing learners see (set via `defaultUrl` in `jupyter-lite.json`). Keep it short: confirm the environment works, point at both lessons, and flag the JupyterLite limitations (no `!` shell commands). Test any code cell against the deployed site, not a local desktop Jupyter, since the WASM kernel behaves differently.

## Build and preview locally

```bash
pip install -r requirements.txt
jupyter lite build \
  --contents content/welcome.ipynb \
  --contents content/lc-python \
  --contents content/gapminder
python -m http.server -d _output 8000   # open http://localhost:8000
```

You need `micromamba` (or `mamba`/`conda`) on PATH for the kernel solve. To start clean, delete the build cache and output:

```bash
rm -rf _output .jupyterlite.doit.db cache
```

All three are git-ignored.

## Troubleshooting

* **A package won't import in the browser:** it's probably not built for WASM. Confirm it's on emscripten-forge; if not, it can't be added to the kernel.
* **Deploy succeeded but the site is stale:** hard-refresh; JupyterLite caches aggressively in the browser and service worker. Try a private window to rule out cache.
* **Build fails in CI:** check the Actions log for the `jupyter lite build` step. A bad notebook JSON or a package that doesn't resolve for the Emscripten target are the usual causes.
* **A data file is missing from the site:** check it isn't shadowed by a same-named file in the other lesson's folder (see "How content is organised").
* **Learner work disappeared:** expected. Notebooks save to browser local storage, not a server. There's no recovery; remind learners to download files they want to keep.

## Add another lesson

1. Create `content/<lesson>/data/` and drop its data in. Confirm no filename collides with an existing lesson's data.
2. Add any packages it needs to `environment.yml` (check emscripten-forge first).
3. Add `--contents content/<lesson>` to the build in `.github/workflows/deploy.yml` (and to the commands in this file and the README).
4. Update `content/welcome.ipynb` and the README tables: mention the new dataset and add the lesson link.

To repurpose the whole site for a different set of lessons, fork it, swap `content/`, rewrite `environment.yml`, and adjust `defaultUrl` in `jupyter-lite.json` if the welcome notebook is renamed.

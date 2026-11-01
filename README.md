# UB Carpentries: Python (JupyterLite fallback)

A browser-based Python environment for UB Carpentries Python workshops. It runs entirely in the browser using [JupyterLite][jupyterlite]: no install, no account, no setup. It exists as a fallback for learners who can't install Python locally (locked-down laptops, install trouble during a workshop, or wanting to start fast).

It covers the Library Carpentry Python lesson:

- [Library Carpentry: Introduction to Python][lc] (`lc-python-intro`)

**Live site:** <https://ucla-data-science-center.github.io/uc-carpentries-python-lite/>

The lesson's data and a welcome notebook are bundled in, so file paths like `data/2011_circ.csv` work exactly as the lesson text writes them.

---

## For learners

1. Open the live site: <https://ucla-data-science-center.github.io/uc-carpentries-python-lite/>. It loads JupyterLab in your browser. First load can take a few seconds while the Python kernel downloads.
2. The `welcome.ipynb` notebook opens automatically. Run its first code cell to confirm everything works.
3. To start a fresh notebook: click **+** at the top of the file browser, then choose **Python (XPython)**.
4. Follow the lesson: <https://librarycarpentry.github.io/lc-python-intro/>

The datasets live in `data/`: the 12 Chicago Public Library circulation CSVs (2011-2022) plus `all_years.pkl` and `df_long.pkl`.

### What's different from a normal Jupyter install

* **No shell commands.** Lines starting with `!` (such as `!pip install` or `!ls`) don't work in JupyterLite. Everything you need is preinstalled.
* **Preinstalled packages:** `pandas`, `numpy`, `matplotlib`, `seaborn`, `plotly`. If a lesson asks you to install something else, it won't be available here.
* **Your work lives in the browser.** Notebooks you create are saved to your browser's local storage, not to a server. Clearing site data wipes them. Download anything you want to keep (right-click the file → Download).

---

## For maintainers

This repo is a JupyterLite site that GitHub Actions builds and publishes to GitHub Pages on every push to `main`.

### Layout

| Path | Purpose |
|---|---|
| `content/welcome.ipynb` | The notebook that opens on load. |
| `content/lc-python/data/` | Library Carpentry datasets. |
| `environment.yml` | The in-browser kernel and its packages (xeus-python + the scientific stack), resolved from emscripten-forge. |
| `requirements.txt` | The build-time tools that run in CI (`jupyterlite`, `jupyterlite-xeus`, `jupyter-server`), version-pinned. |
| `jupyter-lite.json` | JupyterLite config. Sets the notebook that opens on load (`welcome.ipynb`). |
| `.github/workflows/deploy.yml` | Build + deploy to Pages. |

The build passes one `--contents` flag per content folder. The lesson's `data/` subfolder lands in `data/` in the built site, so lesson-hardcoded paths work without a prefix.

### Build locally

```bash
pip install -r requirements.txt
jupyter lite build \
  --contents content/welcome.ipynb \
  --contents content/lc-python
# serve the result
python -m http.server -d _output 8000   # then open http://localhost:8000
```

`_output/` is git-ignored; it's a build artifact.

See [MAINTAINING.md](MAINTAINING.md) for how to update the data, add a lesson, change packages, edit the welcome notebook, and troubleshoot the deploy.

[jupyterlite]: https://jupyterlite.readthedocs.io/
[lc]: https://librarycarpentry.github.io/lc-python-intro/

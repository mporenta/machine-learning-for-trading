# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository purpose

Companion code for the 2nd edition of *Machine Learning for Trading* by Stefan Jansen. The repo is **>150 Jupyter notebooks** organized by chapter, not an installable Python package. There is no test suite, no linter config, and no build step — each notebook is a self-contained example. Most notebooks are checked in **already executed** so outputs are visible without rerunning.

## Repository layout

- `NN_<topic>/` (e.g. `04_alpha_factor_research/`, `08_ml4t_workflow/`) — one directory per book chapter, each with its own `README.md` and the notebooks for that chapter. Numeric prefixes mirror the book's chapter order.
- `data/` — data-sourcing notebooks (`create_datasets.ipynb`, `create_stooq_data.ipynb`, etc.). Run these first; many later chapters expect data fetched here.
- `24_alpha_factor_library/` — appendix with 100+ alpha factor implementations referenced from earlier chapters.
- `installation/` — OS-specific conda/pip environment files. `ml4t-base.yml` / `ml4t-base.txt` are the OS-agnostic versions and the current recommendation; `linux/`, `macosx/`, `windows/` hold pinned versions.
- `utils.py` (repo root) — small shared utilities, notably `MultipleTimeSeriesCV`, a purged time-series cross-validator that operates on a pandas MultiIndex with `symbol` and `date` levels. Imported by many notebooks.
- `figures/`, `assets/` — color versions of book figures; not used by code.

## Environment

Python **3.8** with the `ml4t` conda env. Create it once and reuse across chapters; do not try to install every dependency at once (the book README explicitly warns this causes conflicts — install per-chapter as needed).

```bash
conda create -n ml4t python=3.8
mamba env update -n ml4t -f installation/ml4t-base.yml
conda activate ml4t
```

Backtesting chapters (8, 11, 12, etc.) additionally require Zipline data ingestion, which depends on a Quandl API key (`QUANDL_API_KEY` env var):

```bash
zipline ingest -b quandl
```

Zipline stores its data under `~/.zipline/`. The repo also defines custom Zipline bundles under `08_ml4t_workflow/04_ml4t_workflow_with_zipline/01_custom_bundles/` — chapters that use them have their own ingest steps in the chapter README.

## Running notebooks

```bash
jupyter notebook        # or: jupyter lab
```

To execute a notebook headlessly or convert it:

```bash
jupyter nbconvert --to notebook --execute path/to/notebook.ipynb
jupyter nbconvert --to script   path/to/notebook.ipynb
```

The book's notebooks are formatted for the **Table of Contents (2)** nbextension (`jupyter_contrib_nbextensions`); rendering is fine without it.

## Working in this codebase

- **Notebook-first**: features and bug fixes generally mean editing notebooks, not adding modules. Don't refactor notebook code into a package unless explicitly asked — it would break the book/notebook correspondence.
- **Per-chapter READMEs** are authoritative for that chapter's data, env requirements, and run order. Read them before touching code in a chapter.
- **Pinned, stale dependencies**: the OS-specific env files target library versions from ~2021 (pandas <1.3, gensim <4.0, TF 2.2, Zipline-reloaded). Don't "modernize" imports/APIs to newer library versions without confirming the user wants the upgrade.
- **Data is not in the repo**: most notebooks read from paths populated by the `data/` sourcing notebooks or by `zipline ingest`. If a notebook errors on missing files, point the user at the relevant sourcing step rather than fabricating data.
- **External services** referenced by notebooks (Quandl, Stooq, Algoseek, SEC EDGAR, Twitter, Yelp) may have changed since publication — note this when a download step fails; don't assume the code is broken.

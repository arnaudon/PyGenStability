# CLAUDE.md

Orientation for Claude Code sessions working in this repo.

## Project

PyGenStability is a Python package for multiscale community detection via
**generalized Markov Stability**. The core loop scans a range of Markov times,
runs many generalized-Louvain (or Leiden) optimizations per scale, and
post-processes the partitions (robustness, NVI, optimal scale selection).

A C++ Louvain implementation is shipped as a pybind11 extension.

## Layout

- `src/pygenstability/` — package source
- `tests/` — pytest suite (100% coverage enforced, see `tox.ini`)
- `examples/` — notebooks and scripts, including `real_examples/` (powergrid, protein)
- `docs/` — Sphinx + nbsphinx
- `extra/lemon/`, `generalizedLouvain/` — C++ headers / sources used by the extension

## Key modules (`src/pygenstability/`)

- `pygenstability.py` — `run()` orchestrates the scale scan, parallelism, and post-processing
- `constructors.py` — quality-matrix / null-model builders (linearized, continuous, directed, signed, …)
- `optimal_scales.py` — robustness-based optimal scale selection
- `plotting.py` — matplotlib plots; optional plotly/networkx branches
- `data_clustering.py` — end-to-end data-to-graph clustering workflow
- `app.py` — Click CLI; console script `pygenstability=pygenstability.app:cli`
- `io.py` — pickle save/load helpers
- `contrib/sankey.py` — interactive Sankey diagram (optional plotly)

## Build / test

```bash
pip install -e .             # builds pybind11 C++ extension (-std=c++11)
pytest                       # needs the extension built first
tox                          # full matrix
tox -e py310 | py311 | py312 # single interpreter
tox -e lint                  # pylint + pycodestyle + pydocstyle + isort + black --check
tox -e format                # black + isort (writes)
tox -e docs                  # Sphinx build
```

Supported Python: **3.10 / 3.11 / 3.12** (see `tox.ini`).

## Gotchas

- The C++ extension is declared `optional=True` in `setup.py`. If it fails to
  compile, `import pygenstability.generalized_louvain` silently fails and
  Louvain is unavailable; Leiden fallback only kicks in if `leidenalg` is
  installed (see `_check_method` in `pygenstability.py`).
- BLAS/OpenMP thread counts are pinned to 1 inside worker processes
  (`_limit_numpy` in `constructors.py`) to avoid multiprocessing contention.
  Don't remove this without understanding the parallelism model.
- Optional extras: `[plotly]`, `[leiden]` (igraph + leidenalg), `[networkx]`,
  `[all]`.
- Tests write expected-output YAMLs under `tests/data/`. When changing
  numerics, regenerate fixtures deliberately rather than blindly accepting
  diffs.

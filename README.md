# Adaptive-mixture distributionally robust load shedding

Reproducibility code for **Distributionally Robust Chance-Constrained Optimal Load Shedding with Adaptive Mixture Weights and phi-Divergence Ambiguity Sets**.

This repository contains the optimization engine, fixed experiment configurations, complete recorded solver outcomes, and scripts for rebuilding numerical tables and figures. The manuscript is distributed separately.

## Quick start: verify the recorded results

Run all commands from this repository directory. Verification needs Python 3.11 or newer and uses only the standard library:

```sh
python reproduce.py verify
```

The main experiment contains 18 grid cells and 5,184 method-hour records. The supplement contains 70 variants and 2,106 records. Failed or infeasible outcomes remain in the data; verification does not remove them.

## Rebuild tables and figures without solving

```sh
python -m pip install -r requirements.txt
python reproduce.py rebuild
```

Each rebuild creates a fresh `outputs/rebuild_<timestamp>/` workspace. It checks the full capacity grid, reconstructs main and supplementary reports, and verifies duplicated controls and chronological coverage. It does not overwrite the recorded reference results or compile a manuscript. Plots and LaTeX table fragments are under the new workspace's `paper/`; CSV reports are under its `code/julia_drcc_ols/results/`.

## Replicate optimization

The recorded experiments used Julia 1.12.1. Package versions are pinned in `code/julia_drcc_ols/Manifest.toml`. Install dependencies and the exact input files first:

```sh
julia --project=code/julia_drcc_ols -e 'using Pkg; Pkg.instantiate()'
python reproduce.py install-data --source /path/to/input-directory
python reproduce.py check-data
```

On PowerShell, the Julia command also accepts double quotes around `using Pkg; Pkg.instantiate()`.

The input directory must contain `RBTS/` and `RTS_79/`. See [data sources and file requirements](docs/DATA.md). Raw third-party inputs are not bundled in the upload archive.

Preview the 18 saved configurations without launching a solver:

```sh
python reproduce.py main --dry-run
```

Run one case or the complete main grid:

```sh
python reproduce.py main --case rbts_s100_c100 --name rbts_replication
python reproduce.py main --name main_replication
```

Run a supplementary group:

```sh
python reproduce.py supplement --group sensitivity --name sensitivity_replication
python reproduce.py supplement --group chronological_rbts --name rbts_chronology
python reproduce.py supplement --group chronological_rts79 --name rts79_chronology
```

Each name must be new. Outputs go to `code/julia_drcc_ols/results/replications/<name>/`. The wrapper uses saved configurations/manifests and only redirects output paths. It sets one Julia thread; the supplementary runner also sets one BLAS thread. Solver timing can differ across machines. Rebuild uses recorded reference results; fresh replication results remain separately available for comparison.

## Layout

| Path | Contents |
|---|---|
| `reproduce.py` | Verification, data installation, report rebuilding, replication commands |
| `code/julia_drcc_ols/src/DRCCOLS.jl` | Original optimization and evaluation engine |
| `code/julia_drcc_ols/config/` | Shared RBTS/RTS79 input configuration templates |
| `code/julia_drcc_ols/scripts/` | Main and supplementary runners and reporting functions |
| `code/julia_drcc_ols/results/wind_control_final_20260911/` | Exact 18 configurations, manifest, complete main results |
| `code/julia_drcc_ols/results/wind_control_final_report/` | Recorded main aggregate tables and provenance |
| `code/julia_drcc_ols/results/review_supplement_20260913/` | Fixed supplementary manifest and complete results |
| `data_manifest.json` | Required input filenames, sizes and SHA-256 checksums |
| `checksums.json` | Checksums of distributed code and reference results |
| `docs/PROTOCOL.md` | Experiment definitions and interpretation |

The dated result directory names identify fixed reference datasets. `review_supplement` is a supplementary-experiment identifier. The low-level `run_wind_isolation.jl` is retained as the historical design generator; use `reproduce.py` for fresh runs so that reference results are not overwritten. The two templates in `config/` alone are not the final grid: main replications use the 18 saved TOML files.

## Scope and provenance

The main comparison is a wind-only controlled stress grid. It is not an annual reliability estimate. Read future-error and same-hour outcomes separately, and report availability alongside conditional common-hour comparisons. Details are in [the protocol](docs/PROTOCOL.md).

Numerical records and the optimization engine are retained from the experiment archive. Workspace-specific absolute paths have been converted to repository-relative paths; checksums describe the distributed version. Report rebuilds run in isolated workspaces. No new optimization experiments were performed while packaging this repository.

Third-party inputs and dependencies retain their respective rights and attribution requirements. No new blanket license is assigned to third-party data by this repository. See [data documentation](docs/DATA.md).

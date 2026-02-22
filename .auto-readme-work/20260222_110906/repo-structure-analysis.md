## Project Summary
- `iccp_test` is an experiment-heavy research repository for inverse metasurface design, centered on S4/RCWA simulation plus PyTorch training/evaluation workflows.
- The primary reproducible pipeline in the current docs/scripts is: generate S4 spectra and shape files -> merge/pivot CSVs with shape vertices -> preprocess to `.npz` -> train three-stage transmittance models -> evaluate/visualize.
- The codebase also contains a large secondary branch of AVIRIS hyperspectral experiments (`AVIRIS*` folders and `aviris_*.py`) and many archived/deprecated variants.
- Repository is data-heavy (many multi-GB result/cache folders), not a lightweight Python package.

## Repository Map
```text
.
├── README.md (+ README.<lang>.md variants)
├── how_to_run.md
├── commands.md, commands_updated.md
├── iccp.yaml
├── pip_requirements.txt
├── ms.sh, ms_final.sh, ms_resume*.sh
├── metasurface_*.lua
├── merge.py, merge_s4_data_full.py, merge_robust.py, merge_s4_data_local.py
├── three_stage_transmittance.py
├── three_stage_transmittance_evaluation.py
├── FilterShapeS4_Evaluator_Transmittance.py
├── partial_crys_data/
├── shapes/
├── results/
├── merged_csvs/
├── outputs_three_stage_*/
├── AVIRIS*/
├── aviris_*.py
├── deprecated/, deprecated_code/, deprecated-part2/, archived/
└── many timestamped experiment output folders (evaluation/noise/filter/plot artifacts)
```

## Key Components
- **S4 simulation launchers**
  - `ms.sh`: runs `../build/S4` over `NQ=1..4` with `metasurface_seed.lua`.
  - `ms_final.sh`: parameterized run (`-ns`, `-r`, `-p`, `-g`, `-bo`, `-ro`) calling `metasurface_final.lua` in parallel over `NQ=1..4`.
  - `ms_resume_allargs.sh`: similar interface, calling `metasurface_allargs_resume.lua` for resumable runs.
- **Lua simulation logic**
  - `metasurface_final.lua` (header comments still mention `metasurface_allargs_resume.lua`) parses S4 arg string, loads `partial_crys_data/*.csv`, generates C4 polygons, writes `results/*.csv` and `shapes/.../outer_shape*.txt`, and supports resume semantics via prefix/output file continuation.
- **Data merging/preprocessing**
  - `merge_s4_data_full.py`: merges `results/*_output_nQ*_nS*.csv`, pivots `R@*`/`T@*`, attaches `vertices_str` from shape files, outputs `merged_s4_shapes_<prefix>.csv`.
  - `merge.py` and `merge_robust.py`: alternate merge strategies with different filename/folder matching assumptions.
  - `three_stage_transmittance.py --preprocess`: converts merged CSVs into compressed `.npz` (`uids`, `spectra`, `shapes`) after filtering/packing Q1 vertices into fixed `4x3` tensors.
- **Modeling/training**
  - `three_stage_transmittance.py`: defines data loaders and transformer-based models:
    - Stage A: shape -> spectrum
    - Stage B: spectrum -> shape
    - Stage C: spectrum -> shape -> spectrum chain tuning
  - Outputs to `outputs_three_stage_<timestamp>/stageA|stageB|stageC` with checkpoints and loss curves.
- **Evaluation/visualization**
  - `three_stage_transmittance_evaluation.py`: loads stage checkpoints, computes MSE metrics, writes `evaluation_metrics.csv`/`metrics_summary.csv`, and exports figures/training curves.
  - `FilterShapeS4_Evaluator_Transmittance.py`: compares network-predicted shapes/spectra with fresh S4 runs (calls `../build/S4 ... metasurface_fixed_shape_and_c_value.lua`).
- **Secondary research tracks**
  - Extensive AVIRIS and noise/reconstruction experiments (`aviris_*.py`, `noise_experiment_*.py`, many result directories), indicating multi-project use of one monorepo-like workspace.

## Setup Signals
- Environment files present:
  - `iccp.yaml` (Conda env `iccp`, Python 3.9, PyTorch/CUDA stack, Lua, scientific packages).
  - `pip_requirements.txt` (large pinned list; appears to reflect a broad workstation environment rather than a minimal project lockfile).
- Execution assumptions from scripts/docs:
  - Linux + Bash workflow.
  - S4 binary expected at `../build/S4` relative to repo root.
  - Core Python runtime uses PyTorch, NumPy, pandas, matplotlib, tqdm.
- No standard packaging metadata detected (`pyproject.toml`, `setup.py`, `setup.cfg`, `requirements.txt` absent).

## Usage Signals
- Documented flow in `README.md` and `how_to_run.md` aligns with script CLIs:
  1. Run S4 generation (`ms_final.sh` or related launcher).
  2. Merge generated data (`merge_s4_data_full.py --prefix ...`).
  3. Preprocess CSV folder (`three_stage_transmittance.py --preprocess --input_folder ... --output_npz ...`).
  4. Train (`three_stage_transmittance.py --data_npz ... --num_epochs ... --batch_size ...`).
  5. Evaluate (`three_stage_transmittance_evaluation.py --model_dir ... --data_npz ...`).
  6. Optional S4-vs-network check (`FilterShapeS4_Evaluator_Transmittance.py ...`).
- CLI interfaces are script-centric and explicit via `argparse`; no unified task runner (e.g., Makefile/poetry/nox) is present.
- Existing `README.md` already contains a polished pipeline narrative and references inverse metasurface design; multilingual README variants are also present.

## Gaps/Unknowns
- Scope boundary is broad: metasurface inverse design and AVIRIS/hyperspectral tracks coexist; README should clarify what is primary vs auxiliary.
- Repository contains very large local datasets/results; no clear data download/source policy or storage guidance is formalized.
- Some script naming/comments are inconsistent (`metasurface_final.lua` comment header references `metasurface_allargs_resume.lua`), so README should favor observed CLI behavior over comments when describing commands.
- Multiple merge/evaluation variants exist; canonical “blessed” scripts should be explicitly chosen in README.
- No explicit license file detected.
- Pipeline context requires future README to use H1 `Inverse Design of Metasurface for Spectral Imaging` and include the provided BibTeX citation verbatim.

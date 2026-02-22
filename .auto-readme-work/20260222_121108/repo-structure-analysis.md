## Project Summary
This repository is a script-first research codebase for inverse metasurface design using RCWA (S4) simulation plus PyTorch training. The core workflow is:
1. Generate optical responses from parametrized C4-symmetric shapes (`ms*.sh` + `metasurface*.lua`, writing into `results/` and `shapes/`).
2. Merge S4 outputs with shape vertices (`merge_s4_data_full.py`, producing `merged_s4_shapes*.csv`).
3. Preprocess merged CSVs into NPZ tensors (`three_stage_transmittance.py --preprocess`).
4. Train a three-stage pipeline (`three_stage_transmittance.py`) and evaluate checkpoints (`three_stage_transmittance_evaluation.py`, `FilterShapeS4_Evaluator_Transmittance.py`).

The repository also contains extensive side branches (AVIRIS/hyperspectral experiments, noise experiments, archived scripts, and many timestamped output folders).

## Repository Map
```text
.
├── README.md
├── README.en.md / README.zh-CN.md / ... (multilingual variants)
├── how_to_run.md
├── commands.md
├── iccp.yaml
├── pip_requirements.txt
│
├── ms.sh
├── ms_final.sh
├── ms_resume.sh
├── ms_resume_allargs.sh
├── ms_resume_random_state.sh
├── metasurface_seed.lua
├── metasurface_final.lua
├── metasurface_allargs_resume.lua
├── metasurface_resume_random_state.lua
├── metasurface_fixed_shape_and_c_value.lua
│
├── merge_s4_data_full.py
├── three_stage_transmittance.py
├── three_stage_transmittance_evaluation.py
├── FilterShapeS4_Evaluator_Transmittance.py
│
├── partial_crys_data/                 # material optical tables (C=0.0..1.0 CSVs)
├── results/                           # raw S4 outputs (many CSVs)
├── shapes/                            # saved polygon vertices per run
├── merged_csvs/                       # merged training CSVs
├── outputs_three_stage_*/             # stageA/stageB/stageC checkpoints + curves
│
├── AVIRIS*/                           # AVIRIS data processing/experiments
├── noise_experiment*/                 # noise robustness experiment families
├── archived/                          # historical snapshots/older outputs
└── many timestamped result/eval dirs  # experiment artifacts
```

## Key Components
- `ms.sh`: basic launcher for `metasurface_seed.lua` across `NQ=1..4` in parallel using `../build/S4`.
- `ms_final.sh`: parameterized launcher (`num_shapes`, `seed`, `prefix`, `num_g`, `base_outer`, `rand_outer`) calling `metasurface_final.lua`.
- `ms_resume_allargs.sh`: same CLI pattern, invoking `metasurface_allargs_resume.lua` (resume-aware naming/continuation logic).
- `ms_resume_random_state.sh`: launcher for `metasurface_resume_random_state.lua` with random-state resume argument layout.
- `metasurface_seed.lua` / `metasurface_final.lua` / `metasurface_allargs_resume.lua`: S4 simulation programs that generate C4 polygons, run wavelength sweeps from `partial_crys_data/*.csv`, and write outputs to `results/*.csv`; shape vertices are persisted in `shapes/...`.
- `merge_s4_data_full.py`: merges result CSVs, extracts `NQ/nS/c`, pivots spectral columns into `R@...` and `T@...`, and attaches `vertices_str` from `shapes/.../outer_shape*.txt`.
- `three_stage_transmittance.py`:
  - preprocessing mode converts merged CSVs into `npz` arrays (`uids`, `spectra`, `shapes`),
  - training mode runs Stage A (`shape->spec`), Stage B (`spec->shape`), Stage C (`spec->shape->spec`) and writes `outputs_three_stage_<timestamp>/stage{A,B,C}`.
- `three_stage_transmittance_evaluation.py`: loads stage checkpoints, computes per-sample/average metrics CSVs, and generates visualization/curve figures.
- `FilterShapeS4_Evaluator_Transmittance.py`: compares neural predictions against fresh S4 simulation via `metasurface_fixed_shape_and_c_value.lua`.

## Setup Signals
- Environment specs exist:
  - `iccp.yaml` (Conda env `iccp`, Python 3.9, CUDA/PyTorch stack, GDAL/Jupyter-heavy set).
  - `pip_requirements.txt` (broad mixed package list; appears environment-export style rather than minimal project lockfile).
- Runtime assumptions from scripts:
  - Linux/Bash shell scripts.
  - S4 binary expected at `../build/S4`.
  - Data folders expected in repo root (`partial_crys_data`, `results`, `shapes`, `merged_csvs`).
- Existing docs signaling setup/usage:
  - `README.md` provides end-to-end commands.
  - `how_to_run.md` provides preprocess/train commands.

## Usage Signals
- Simulation generation:
  - `./ms.sh -ns <num_shapes> -r <seed>` (seed/basic path).
  - `./ms_final.sh ...` / `./ms_resume_allargs.sh ...` for configurable/resumable runs.
- Merge step:
  - `python merge_s4_data_full.py --prefix <run_prefix>` (or no prefix for all).
- Training data preprocessing:
  - `python three_stage_transmittance.py --preprocess --input_folder merged_csvs --output_npz preprocessed_t_data.npz`.
- Training:
  - `python three_stage_transmittance.py --data_npz <file.npz> --num_epochs <N> --batch_size <B>`.
- Evaluation:
  - `python three_stage_transmittance_evaluation.py --model_dir outputs_three_stage_<timestamp> --data_npz <file.npz>`.
  - Optional S4 consistency check via `FilterShapeS4_Evaluator_Transmittance.py`.
- Data-contract signals in code:
  - merged CSV consumed by training expects columns including `prefix`, `nQ`, `nS`, `shape_idx`, `c`, `vertices_str`, and many `T@...` columns.
  - preprocessing keeps only shape groups with exactly 11 `c` rows and 1..4 Q1 points after shifting.

## Gaps/Unknowns
- No clear packaging structure (`src/`, installable module, or entry-point console scripts); workflow is script-driven.
- No single canonical minimal dependency file; both `iccp.yaml` and `pip_requirements.txt` are very broad.
- Core S4 dependency is external (`../build/S4`) and not vendored/built in-repo.
- Repository includes large binary/experiment artifacts and many historical branches; not all directories are part of the primary inverse_metasurface pipeline.
- Limited test signals for the main three-stage pipeline (some `test_*.py` files exist, but no unified test runner/config discovered).
- Naming/schema continuity can require care across steps (merge/training column conventions are strict in code).

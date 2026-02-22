# Project Summary
`iccp_test` is a research codebase for inverse metasurface design combining S4/RCWA simulation (Lua + Bash orchestration) with PyTorch training/evaluation pipelines.

Primary workflow visible in code/docs:
1. Generate spectra with S4 (`ms*.sh` + `metasurface*.lua`) into `results/` and shape files in `shapes/`.
2. Merge raw simulation CSVs + vertices into wide tables (`merge.py`, `merge_s4_data_*.py`, `merge_robust.py`).
3. Preprocess merged CSVs into NPZ and train three-stage models (`three_stage_transmittance.py`).
4. Evaluate checkpoints/plots/metrics (`three_stage_transmittance_evaluation.py`, `FilterShapeS4_Evaluator_Transmittance.py`).

The repository is experiment-heavy: many timestamped output/result directories, AVIRIS/hyperspectral branches, and deprecated archives are kept in-tree.

# Repository Map
```text
iccp_test/
├─ README.md, README.en.md, README.fr.md, README.de.md, ... (multilingual READMEs)
├─ how_to_run.md, commands.md
├─ iccp.yaml, pip_requirements.txt
├─ ms.sh, ms_final.sh, ms_resume_allargs.sh, ms_resume_random_state*.sh
├─ metasurface_seed.lua, metasurface_final.lua, metasurface_allargs_resume.lua
├─ merge.py, merge_s4_data_full.py, merge_s4_data_local.py, merge_robust.py
├─ three_stage_transmittance.py
├─ three_stage_transmittance_evaluation.py
├─ FilterShapeS4_Evaluator_Transmittance.py, filter2shape2filter_pipeline.py
├─ aviris_*.py, noise_experiment_*.py, recon_task_*.py, shape2*.py, spectra2*.py
├─ partial_crys_data/, gsst_partial_crys_data/, nk_crystalline_amorphous/
├─ results/, shapes/, merged_csvs/, outputs/, outputs_three_stage_*/
├─ AVIRIS*/, cache*/, data/, formatted_csvs/, split_csvs/
├─ archived/, deprecated/, deprecated-part2/, deprecated_code/, deprecated-scripts/
├─ cvpr_loss_plots/, visualization/, spectra_plots/, optical_plots_*/
└─ .auto-readme-work/
```

# Key Components
- **S4 simulation runners**: `ms.sh` (simple seed run), `ms_final.sh` and `ms_resume_allargs.sh` (parameterized/resumable runs over `nQ=1..4` in parallel using `../build/S4`).
- **Lua simulation cores**: `metasurface_seed.lua`, `metasurface_final.lua`, `metasurface_allargs_resume.lua` parse S4 args, read `partial_crys_data/*.csv`, generate C4-symmetric polygon shapes, and write CSV results + per-shape vertex files.
- **Data merge utilities**:
  - `merge.py`: merges a chosen prefix from `results/`, attaches vertices from matching `shapes/`, pivots R/T spectra to `R@*`, `T@*` columns.
  - `merge_s4_data_full.py` / `merge_s4_data_local.py`: broad merge variants using glob patterns.
  - `merge_robust.py`: more tolerant filename/folder matching.
- **Training pipeline**: `three_stage_transmittance.py` contains:
  - preprocessing (`--preprocess`) from merged CSV folder to compressed NPZ,
  - dataset classes,
  - model definitions (`ShapeToSpectraModel`, `Spectra2ShapeVarLen`, chained Stage C wrapper),
  - three-stage train loops (A/B/C) with checkpoint + curve outputs under `outputs_three_stage_<timestamp>/stage{A,B,C}`.
- **Evaluation/visualization**:
  - `three_stage_transmittance_evaluation.py` loads stage checkpoints, computes per-sample/average MSE metrics, exports CSV summaries and figures.
  - `FilterShapeS4_Evaluator_Transmittance.py` compares network-predicted shapes/spectra against direct S4 runs.
- **Research branches**: numerous `aviris_*`, `noise_experiment_*`, `recon_task_*` scripts indicate parallel tracks for hyperspectral compression/reconstruction and robustness studies.

# Setup Signals
- **Environment file present**: `iccp.yaml` defines conda env `iccp` (Python 3.9, PyTorch/CUDA stack, scientific/python tooling).
- **Additional pip list**: `pip_requirements.txt` exists but looks broader/system-mixed; likely supplementary rather than sole canonical setup.
- **External binary dependency**: shell runners hardcode `../build/S4`; project assumes S4 already built/available outside repo root.
- **Platform assumptions**: Bash scripts, Linux path conventions, and parallel background process usage.
- **Large artifact handling**: `.gitignore` excludes many heavy outputs (`*.csv`, `*.npz`, `*.pt`, `results*`, `cache*`, images/videos).

# Usage Signals
- `README.md` and `how_to_run.md` document an end-to-end path: S4 generation -> merge -> preprocess -> three-stage training -> evaluation.
- `commands.md` captures practical command presets for local/3090 runs and resume patterns.
- `three_stage_transmittance.py` CLI supports:
  - preprocessing mode (`--preprocess --input_folder --output_npz`),
  - training mode (`--data_npz` or `--csv_file`, plus `--num_epochs`, `--batch_size`).
- `three_stage_transmittance_evaluation.py` CLI supports:
  - model-dir based evaluation (`--model_dir` required),
  - data input via `--data_npz` or `--csv_file`,
  - optional `--plot_only`, configurable `--sample_count`, `--batch_size`, and output directory.
- Test signals are lightweight (`test.py`, `test_filter2shape2filter.py`, `test_shape_rotate*.py`), not a full automated suite.

# Gaps/Unknowns
- No explicit license file is present (`README` badge says license unset).
- No single package layout (`src/`, `pyproject.toml`) or formal install target; repository is script-centric.
- Multiple merge scripts use slightly different schema conventions (`prefix`/`folder_key`, `nQ`/`NQ`), so compatibility depends on which merge output is fed into training.
- Root contains a very large amount of timestamped outputs and branch scripts; canonical “main” experiment path is implied but not strictly enforced by structure.
- Hardware/runtime requirements are only partially explicit (GPU optional in docs, but practical runtime for larger runs is not quantified in-repo).

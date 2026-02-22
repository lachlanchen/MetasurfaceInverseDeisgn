# Project Summary
This repository is an experiment-heavy codebase for **inverse metasurface research** centered on S4/RCWA simulation and neural inverse-design pipelines, with a substantial secondary branch for **AVIRIS hyperspectral compression/noise experiments**.  

The inverse metasurface workflow appears to be:
1. Generate RCWA data with S4 (`metasurface_*.lua` launched via `ms*.sh`),
2. Merge/reshape raw S4 outputs into training-ready tabular formats (`merge*.py`),
3. Train multi-stage neural models for shape↔spectrum/transmittance tasks (`three_stage_transmittance.py`, related scripts),
4. Evaluate neural predictions against direct S4 simulation (`shape2filter_with_s4.py`, `FilterShapeS4_Evaluator_*.py`).

No root `README` was found; operational knowledge is mostly in scripts and `how_to_run.md`/`commands*.md`.

# Repository Map
```text
iccp_test/
├─ .auto-readme-work/...
├─ how_to_run.md
├─ commands.md
├─ commands_updated.md
├─ iccp.yaml
├─ pip_requirements.txt
├─ .gitignore
├─ ms.sh
├─ ms_final.sh
├─ ms_resume_allargs.sh
├─ ms_resume_random_state.sh
├─ metasurface_final.lua
├─ metasurface_seed.lua
├─ metasurface_allargs_resume.lua
├─ metasurface_resume_random_state.lua
├─ metasurface_fixed_shape_and_c_value.lua
├─ three_stage_transmittance.py
├─ three_stage_transmittance_evaluation.py
├─ three_stage_transmittance_loss_plot.py
├─ shape2filter_with_s4.py
├─ shape2filter_with_s4_comparison.py
├─ FilterShapeS4_Evaluator_Transmittance.py
├─ FilterShapeS4_Evaluator_Random.py
├─ filter2shape2filter_pipeline.py
├─ merge.py
├─ merge_s4_data_full.py
├─ merge_s4_data_local.py
├─ partial_crys_data/
├─ gsst_partial_crys_data/
├─ merged_csvs/
├─ shapes/
├─ shapes_seeded/
├─ results/                         (large run outputs)
├─ outputs/                         (model outputs)
├─ outputs_three_stage_*/           (stage A/B/C runs)
├─ AVIRIS*/                         (many hyperspectral datasets/variants)
├─ aviris_*.py                      (AVIRIS preprocessing/training/analysis)
├─ noise_experiment_*.py            (many denoising/reconstruction experiments)
├─ deprecated/
├─ deprecated-part2/
├─ deprecated_code/
└─ archived/
```

# Key Components
- **S4 launch orchestration (shell):** `ms.sh`, `ms_final.sh`, `ms_resume_allargs.sh`, `ms_resume_random_state.sh` run 4 parallel S4 jobs (`NQ=1..4`) via `../build/S4 -t 32 ...`.
- **S4 simulation logic (Lua):** `metasurface_final.lua`, `metasurface_allargs_resume.lua`, `metasurface_resume_random_state.lua` parse CLI args from `S4.arg`, generate C4-symmetric polygons, load `partial_crys_data/*.csv`, and write per-run optical spectra outputs with resume behavior.
- **Data consolidation:** `merge.py` and `merge_s4_data_full.py` merge per-run `results/*_output_nQ*_nS*.csv` into wide tables with `R@...` / `T@...` columns and attach polygon vertices from `shapes/.../outer_shape*.txt`.
- **Main inverse model pipeline:** `three_stage_transmittance.py` provides:
  - preprocessing from merged CSV to `.npz`,
  - Stage A shape→spectrum model,
  - Stage B spectrum→shape model,
  - Stage C chained spectrum→shape→spectrum fine-tuning,
  - saved checkpoints/curves/samples under timestamped `outputs_three_stage_*`.
- **Physics-vs-network comparison:** `shape2filter_with_s4.py` wraps direct S4 calls (`metasurface_fixed_shape_and_c_value.lua`) in a PyTorch-like module and compares S4 spectra to neural predictions; evaluator scripts in `FilterShapeS4_Evaluator_*.py` and comparison plotting scripts support analysis.
- **AVIRIS branch:** `aviris_*.py` and many `AVIRIS*` folders implement independent hyperspectral tile processing, caching, training, and visualization workflows.

# Setup Signals
- **Primary environment artifacts:** `iccp.yaml` (Conda env named `iccp`, Python 3.9, CUDA/PyTorch stack, Jupyter/scientific libs) and `pip_requirements.txt` (very broad machine-level package list).
- **Runtime assumptions from scripts:**
  - `../build/S4` binary must exist relative to repo path,
  - Lua scripts rely on `partial_crys_data/` and write/read `results/` + `shapes/`,
  - GPU is optional but expected (`torch.cuda.is_available()` checks in training scripts).
- **Python dependencies inferred from imports:** `torch`, `numpy`, `pandas`, `matplotlib`, `scipy`, `tqdm`, plus AVIRIS-related libraries like `spectral`.
- **Ignore policy hints (`.gitignore`):** large data/artifacts (`*.csv`, `*.pt`, `*.npz`, `results*`, `AVIRIS*`) are mostly ignored, indicating local heavy-data workflows.

# Usage Signals
- **Documented training flow (`how_to_run.md`):**
  - preprocess merged CSVs to NPZ with `three_stage_transmittance.py --preprocess`,
  - train three-stage model from NPZ via `--data_npz`, `--num_epochs`, `--batch_size`.
- **Documented S4 run flow (`how_to_run.md`, `commands*.md`):**
  - launch dataset generation with `ms.sh`/`ms_final.sh`,
  - resume/parameterized runs with `ms_resume_allargs.sh` or `ms_resume_random_state.sh`,
  - conventions around prefixes/seeds/NumG/overlap parameters are captured in command notes.
- **Script-level CLIs:**
  - `three_stage_transmittance.py` supports `--preprocess`, `--input_folder`, `--output_npz`, `--data_npz`, `--csv_file`, `--num_epochs`, `--batch_size`.
  - evaluator/merge scripts use prefix-driven arguments (`--prefix`, checkpoint paths, sample counts, output folders).
- **Typical lifecycle inferred from files:** S4 simulation → merge/pivot into training table → preprocess NPZ → train stage A/B/C → evaluate with S4-backed scripts → produce plots/results directories.

# Gaps/Unknowns
- No canonical root `README` or single “blessed” entrypoint for newcomers.
- No explicit repository-level install steps for S4 build/provenance; scripts assume `../build/S4` is prebuilt.
- Multiple overlapping experiment scripts (especially `noise_experiment_*`, `aviris_*`, evaluator variants) with no clear stability ranking.
- No formal test harness/CI config observed; only ad-hoc test scripts (`test.py`, `test_filter2shape2filter.py`, etc.).
- Data prerequisites are implicit (expected CSV schemas/column names such as `T@*`, `vertices_str`, folder naming conventions).
- Repository contains very large output/history footprint, so a clean “minimal reproducible subset” is not defined in-tree.

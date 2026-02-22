## Project Summary
This repository is a large research workspace for **inverse metasurface design for spectral imaging**, with a script-first pipeline built around S4 RCWA simulation, CSV merging, NPZ preprocessing, and three-stage PyTorch training/evaluation.

The current root `README.md` already targets this project as **"Inverse Design of Metasurface for Spectral Imaging"** and documents practical run steps, model stages, and citation text. The repo also includes many experimental branches (especially AVIRIS/hyperspectral and noise experiments) and many timestamped output folders.

## Repository Map
```text
.
├── README.md + README.<lang>.md           # multilingual project docs
├── how_to_run.md                          # short preprocess/train instructions
├── commands.md / commands_updated.md      # run command notes (manual workflow)
├── iccp.yaml                              # main Conda environment (PyTorch/CUDA)
├── pip_requirements.txt                   # broad pip freeze-style dependency list
├── ms.sh / ms_final.sh / ms_resume*.sh    # S4 launchers (Bash)
├── metasurface_*.lua                      # S4 simulation scripts
├── merge_s4_data_full.py                  # merge/pivot S4 CSV + attach shape vertices
├── three_stage_transmittance.py           # preprocess + Stage A/B/C training
├── three_stage_transmittance_evaluation.py# checkpoint evaluation/plots/metrics
├── FilterShapeS4_Evaluator_Transmittance.py # neural vs S4 comparison tool
├── shapes/                                # generated polygon vertex files
├── results/                               # S4 raw outputs (input to merge script)
├── merged_csvs/                           # merged datasets for preprocessing
├── outputs_three_stage_*/                 # training outputs/checkpoints/curves
├── partial_crys_data*/                    # crystallization-state optical tables
├── AVIRIS*/                               # AVIRIS processing/analysis datasets
├── archived/ deprecated*/                 # legacy scripts and historical variants
├── numerous *_2025... folders             # experiment/evaluation artifacts
└── large NPZ artifacts at root            # e.g., preprocessed_data.npz, preprocessed_t_data.npz
```

## Key Components
- **S4 simulation orchestration**
  - `ms.sh`: runs `../build/S4` for `NQ=1..4` with `metasurface_seed.lua`.
  - `ms_final.sh`: parameterized launcher (`--numshapes`, `--seed`, `--prefix`, `--numg`, `--baseouter`, `--randouter`) using `metasurface_final.lua`.
  - `ms_resume_allargs.sh`: same CLI style, using `metasurface_allargs_resume.lua`.
- **Data merge and shaping**
  - `merge_s4_data_full.py`: reads `results/*_output_nQ*_nS*.csv`, parses run metadata, pivots `R@...`/`T@...`, and appends `vertices_str` from matching files in `shapes/<folder>-poly-wo-hollow-nQ*-nS*`.
- **Training pipeline (canonical transmittance path)**
  - `three_stage_transmittance.py`:
    - Preprocess mode: CSV folder -> compressed NPZ (`uids`, `spectra`, `shapes`).
    - Stage A: shape->spectra (`shape2spec_stageA.pt`).
    - Stage B: spectra->shape (`spec2shape_stageB.pt`).
    - Stage C: chain fine-tuning spectra->shape->spectra (`spec2shape_stageC.pt`).
    - Outputs stored under `outputs_three_stage_<timestamp>/stageA|stageB|stageC`.
- **Evaluation and visualization**
  - `three_stage_transmittance_evaluation.py`: loads stage checkpoints, computes per-sample and average MSE metrics, saves figures and CSV summaries.
  - `FilterShapeS4_Evaluator_Transmittance.py`: runs predicted polygons back through S4 (`metasurface_fixed_shape_and_c_value.lua`) and compares neural vs physics outputs.
- **Secondary research tracks**
  - Root contains many additional experiment families (`noise_experiment_*`, `aviris_*`, `two_stage_*`, `three_stage_train_*`, plotting scripts), indicating iterative research beyond one frozen production pipeline.

## Setup Signals
- **Primary environment signal**: `iccp.yaml` (Conda, Python 3.9, PyTorch 2.3.0, CUDA-enabled stack, numpy/pandas/matplotlib/scikit-learn, plus pip extras like `pyro-ppl`, `opencv-python-headless`, `cvxpy`).
- **Alternative dependency snapshot**: `pip_requirements.txt` appears to be a broad machine-level freeze and is less targeted than `iccp.yaml`.
- **System assumptions from scripts/docs**:
  - Linux + Bash execution.
  - External S4 binary expected at `../build/S4` (hardcoded in launchers/evaluator).
  - GPU optional but supported by training/evaluation scripts via `torch.cuda.is_available()`.
- **Repository hygiene signal**:
  - `.gitignore` excludes generated artifacts (`*.csv`, `*.npz`, `*.pt`, `results*`, `AVIRIS*`, caches), consistent with data-heavy local experimentation.

## Usage Signals
- **Documented flow in README/how_to_run + script CLIs**:
  1. Run S4 generation via `ms_final.sh` or `ms_resume_allargs.sh`.
  2. Merge outputs with `merge_s4_data_full.py --prefix <run_prefix>`.
  3. Place merged CSVs in `merged_csvs/`.
  4. Preprocess: `three_stage_transmittance.py --preprocess --input_folder ... --output_npz ...`.
  5. Train: `three_stage_transmittance.py --data_npz ... --num_epochs ... --batch_size ...`.
  6. Evaluate: `three_stage_transmittance_evaluation.py --model_dir ... --data_npz ...`.
  7. Optional neural-vs-S4 validation: `FilterShapeS4_Evaluator_Transmittance.py ...`.
- **Data contract implied by preprocessing code**:
  - Requires transmission columns `T@...` and ID columns used to form `shape_uid`.
  - Expects exactly 11 crystallization rows per unique shape and uses `vertices_str` for geometry parsing.
  - Shape representation normalized to up to 4 Q1 points as `(presence, x, y)`.
- **Scale/organization signal**:
  - Top-level alone contains hundreds of directories (many timestamped runs), indicating reproducibility depends on choosing one intended path among many historical outputs.

## Gaps/Unknowns
- No root `LICENSE` file detected.
- No single minimal "clean" source package/module layout; most logic is in standalone scripts.
- Multiple overlapping pipelines and many historical outputs make the canonical workflow easy to confuse without explicit scoping.
- `merge_s4_data_full.py` emits `folder_key`/`NQ` column names, while training preprocess expects `prefix`/`nQ` fields; compatibility may rely on post-merge normalization (as noted in README), but not enforced inside scripts.
- S4 build/install steps are not provided in-repo beyond assuming `../build/S4` exists.
- Automated tests/CI are not evident for the main end-to-end transmittance path.

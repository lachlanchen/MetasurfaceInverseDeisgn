English | 中文繁體 | 中文简体 | 日本語 | 한국어 | Tiếng Việt | العربية | Français | Español | Deutsch | Русский

# inverse_metasurface

<div align="center">

[![Status](https://img.shields.io/badge/status-experimental-orange)](#-roadmap)
[![Python](https://img.shields.io/badge/python-3.9-blue)](#-prerequisites)
[![Platform](https://img.shields.io/badge/platform-linux-lightgrey)](#-prerequisites)
[![S4](https://img.shields.io/badge/S4-required-success)](#-prerequisites)
[![License](https://img.shields.io/badge/license-unset-lightgrey)](#-license)

</div>

A research workspace for **inverse metasurface design** using **S4/RCWA simulation** and **multi-stage neural models**.

It supports:
- High-throughput S4 data generation for C4-symmetric polygon metasurfaces.
- Data merge + preprocessing into training-ready NPZ.
- Three-stage learning: **shape -> spectrum**, **spectrum -> shape**, **chain-tuned spectrum -> shape -> spectrum**.
- Evaluation and optional direct **neural-vs-S4** comparison.

## 🌟 At A Glance

| Area | What this repo provides |
|---|---|
| Physics simulation | Bash + Lua scripts to run S4 in parallel across `nQ=1..4` |
| Dataset tooling | Merge scripts that attach polygon vertices and pivot spectra |
| ML pipeline | `three_stage_transmittance.py` with preprocess + train modes |
| Evaluation | `three_stage_transmittance_evaluation.py` with metrics + figures |
| Research branch | AVIRIS/hyperspectral and noise/compression experiments |

## 🧠 Research Context

This repository targets inverse design for photonic metasurfaces: infer geometry from desired spectra (and vice versa).

Core assumptions used by the baseline pipeline:
- C4 symmetry via Q1-point parameterization.
- 11 crystallization states (`c` in `[0.0, 1.0]`).
- Each sample spectrum stored as `11 x 100` transmission rows.
- Shapes represented as up to 4 Q1 points (`4 x 3` tensor: presence, x, y).

Three-stage training logic:
1. **Stage A**: shape -> spectrum (`shape2spec_stageA.pt`)
2. **Stage B**: spectrum -> shape (`spec2shape_stageB.pt`)
3. **Stage C**: spectrum -> shape -> spectrum chain fine-tuning (`spec2shape_stageC.pt`)

## 🗂 Project Structure

```text
iccp_test/
├─ ms.sh / ms_final.sh / ms_resume_allargs.sh / ms_resume_random_state.sh
├─ metasurface_seed.lua / metasurface_final.lua / metasurface_allargs_resume.lua
├─ merge.py / merge_s4_data_full.py / merge_s4_data_local.py
├─ three_stage_transmittance.py
├─ three_stage_transmittance_evaluation.py
├─ shape2filter_with_s4.py
├─ FilterShapeS4_Evaluator_Transmittance.py
├─ filter2shape2filter_pipeline.py
├─ partial_crys_data/                # optical constants by crystallization level
├─ results/                          # raw S4 outputs (usually untracked)
├─ shapes/                           # generated polygon vertex files
├─ merged_csvs/                      # merged tables used for preprocessing
├─ outputs_three_stage_*/            # model artifacts per run
├─ AVIRIS*/ + aviris_*.py            # hyperspectral branch
├─ how_to_run.md / commands*.md
├─ iccp.yaml
└─ pip_requirements.txt
```

## ✅ Prerequisites

| Requirement | Notes |
|---|---|
| OS | Linux (scripts assume Bash + Linux paths) |
| Python | 3.9 (from `iccp.yaml`) |
| Env manager | Conda recommended |
| S4 binary | Expected at `../build/S4` relative to repo root |
| GPU (optional) | CUDA speeds up training/evaluation |

## ⚙️ Installation

### 1) Clone and enter

```bash
git clone <your-repo-url> inverse_metasurface
cd inverse_metasurface
```

### 2) Create environment

```bash
conda env create -f iccp.yaml
conda activate iccp
```

### 3) Verify S4 path

```bash
ls -l ../build/S4
```

If this path does not exist, either build/place S4 there or adjust script paths.

## 🚀 Quick Start (End-to-End)

```bash
# 1) Generate S4 data
./ms.sh -ns 10000 -r 12345

# 2) Merge one run by prefix
python merge.py --prefix 20250123_155420

# 3) Move merged CSV to preprocessing folder
mkdir -p merged_csvs
mv merged_s4_shapes_20250123_155420.csv merged_csvs/

# 4) Preprocess to NPZ
python three_stage_transmittance.py --preprocess \
  --input_folder merged_csvs \
  --output_npz preprocessed_t_data.npz

# 5) Train three-stage model
python three_stage_transmittance.py \
  --data_npz preprocessed_t_data.npz \
  --num_epochs 100 \
  --batch_size 1024

# 6) Evaluate model outputs
python three_stage_transmittance_evaluation.py \
  --model_dir outputs_three_stage_YYYYMMDD_HHMMSS \
  --data_npz preprocessed_t_data.npz \
  --sample_count 8
```

## 🧪 Usage Details

### A) S4 generation

Minimal seeded run:

```bash
./ms.sh -ns 10000 -r 88888
```

Parameterized run:

```bash
./ms_final.sh \
  -ns 100000 \
  -r 88888 \
  -p iccpOv100kG80 \
  -g 80 \
  -bo 0.35 \
  -ro 0.30
```

Resume-style run:

```bash
./ms_resume_allargs.sh \
  -ns 100000 \
  -r 88888 \
  -p iccpOv100kG80 \
  -g 80 \
  -bo 0.35 \
  -ro 0.30
```

### B) Merge raw S4 CSVs

```bash
python merge.py --prefix 20250123_155420
```

Alternative utility:

```bash
python merge_s4_data_full.py --prefix 20250123_155420
```

### C) Preprocess merged CSVs -> NPZ

```bash
python three_stage_transmittance.py --preprocess \
  --input_folder merged_csvs \
  --output_npz preprocessed_t_data.npz
```

### D) Train

```bash
python three_stage_transmittance.py \
  --data_npz preprocessed_t_data.npz \
  --num_epochs 100 \
  --batch_size 1024
```

Training outputs are stored in:
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageA`
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageB`
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageC`

### E) Evaluate

```bash
python three_stage_transmittance_evaluation.py \
  --model_dir outputs_three_stage_YYYYMMDD_HHMMSS \
  --data_npz preprocessed_t_data.npz \
  --sample_count 8
```

Generates:
- `evaluation_metrics.csv`
- `metrics_summary.csv`
- stage visualizations (`.png`, `.pdf`)
- training curve plots

### F) Neural vs direct S4 comparison (optional)

```bash
python FilterShapeS4_Evaluator_Transmittance.py \
  --npz_file preprocessed_t_data.npz \
  --spec2shape_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageC/spec2shape_stageC.pt \
  --shape2spec_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageA/shape2spec_stageA.pt \
  --n_samples 4
```

## 🔧 Key CLI Options

### S4 runner flags (`ms_final.sh`, `ms_resume_allargs.sh`)

| Flag | Meaning | Default |
|---|---|---|
| `-ns`, `--numshapes` | Number of shapes | `100000` |
| `-r`, `--seed` | Random seed | `88888` |
| `-p`, `--prefix` | Run prefix / resume key | empty |
| `-g`, `--numg` | Geometry/basis parameter | `80` |
| `-bo`, `--baseouter` | Base outer offset | `0.25` |
| `-ro`, `--randouter` | Outer random offset | `0.20` |

### Training flags (`three_stage_transmittance.py`)

| Flag | Meaning | Default |
|---|---|---|
| `--preprocess` | Run preprocessing mode | off |
| `--input_folder` | Input CSV folder | empty |
| `--output_npz` | Output NPZ path | `preprocessed_data.npz` |
| `--data_npz` | NPZ used for train/eval | empty |
| `--csv_file` | CSV used directly | empty |
| `--num_epochs` | Epochs per stage | `10` |
| `--batch_size` | Batch size | `4096` |
| `--test` | Test mode placeholder | off |

### Evaluation flags (`three_stage_transmittance_evaluation.py`)

| Flag | Meaning | Default |
|---|---|---|
| `--model_dir` | Root trained run directory | required |
| `--data_npz` | Evaluation NPZ | empty |
| `--csv_file` | Evaluation CSV | empty |
| `--output_dir` | Output folder | `model_dir/evaluation_<timestamp>` |
| `--sample_count` | Number of visualized samples | `4` |
| `--seed` | Sampling seed | `23` |
| `--font_scale` | Plot font scale | `1.0` |
| `--batch_size` | Eval batch size | `32` |
| `--plot_only` | Plot curves only | off |

## 🧭 Troubleshooting

| Symptom | Likely cause | Fix |
|---|---|---|
| `../build/S4: No such file or directory` | S4 binary path mismatch | Build/place S4 at `../build/S4` or patch scripts |
| `Must specify either --data_npz or --csv_file` | Missing dataset source | Pass one of those flags explicitly |
| `No matching CSVs found in 'results/'` | Prefix mismatch | Confirm prefix and output naming |
| Very few/zero preprocessed records | Missing/invalid `T@...` or `vertices_str` rows | Validate merged CSV schema and per-shape row grouping |
| CUDA OOM | Batch too large | Reduce `--batch_size` (e.g. `1024 -> 256`) |

## 🧱 Development Notes

- This is an experiment-heavy repo; many generated artifacts are intentionally untracked.
- Several script variants exist for similar tasks (`merge_*`, `aviris_*`, `noise_experiment_*`).
- Baseline inverse-metasurface workflow is the path documented in this README.
- Training scripts set seeds (`42`), but strict determinism still depends on hardware/backend behavior.
- There is no unified CI + full automated test suite at the moment.

## 🛣 Roadmap

- Add a compact canonical dataset for quick smoke tests.
- Consolidate duplicate pipeline scripts.
- Add checks for merge/preprocess integrity and checkpoint loadability.
- Add CI for lint + smoke train/eval.
- Document S4 build/version pinning more explicitly.

## 🤝 Contribution

1. Create a feature branch.
2. Keep PR scope tight (one pipeline/experiment concern per PR).
3. Include exact runnable commands and expected output paths.
4. Avoid committing large generated artifacts unless required.
5. Add reproducibility notes (seed, data source, checkpoint path).

## 📄 License

No `LICENSE` file is currently present in this repository.

Until a license file is added, reuse and redistribution terms should be treated as **undetermined**.

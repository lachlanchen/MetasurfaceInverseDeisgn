<p>
  <b>Languages:</b>
  <a href="README.md">English</a>
  · <a href="README.zh-TW.md">中文（繁體）</a>
  · <a href="README.zh-CN.md">中文 (简体)</a>
  · <a href="README.ja.md">日本語</a>
  · <a href="README.ko.md">한국어</a>
  · <a href="README.vi.md">Tiếng Việt</a>
  · <a href="README.ar.md">العربية</a>
  · <a href="README.fr.md">Français</a>
  · <a href="README.es.md">Español</a>
  · <a href="README.de.md">Deutsch</a>
  · <a href="README.ru.md">Русский</a>
</p>


# Inverse Design of Metasurface for Spectral Imaging

<p align="center">
  <img alt="Status" src="https://img.shields.io/badge/Status-Research%20Prototype-f59e0b">
  <img alt="Python" src="https://img.shields.io/badge/Python-3.9-3776AB">
  <img alt="Framework" src="https://img.shields.io/badge/Framework-PyTorch-EE4C2C">
  <img alt="Simulator" src="https://img.shields.io/badge/RCWA-S4-16a34a">
  <img alt="Platform" src="https://img.shields.io/badge/Platform-Linux%2FBash-6b7280">
</p>

A script-first research repository (historically referenced as `inverse_metasurface`) for **C4-symmetric metasurface inverse design** in spectral imaging, spanning:

- RCWA data generation with S4 (`.lua` + shell launchers)
- data merging and preprocessing (`.csv` -> `.npz`)
- three-stage neural training (shape->spectra, spectra->shape, chain fine-tuning)
- evaluation and optional neural-vs-S4 checks

## ✨ At a Glance

| Item | Details |
|---|---|
| Core target | Predict geometry from target transmission spectra |
| Core dataset shape | spectra: `11 x 100`, shape: `4 x 3` |
| Main training script | `three_stage_transmittance.py` |
| Main evaluation script | `three_stage_transmittance_evaluation.py` |
| RCWA launcher | `ms_final.sh`, `ms_resume_allargs.sh` |
| Merge script | `merge_s4_data_full.py` |

## 🧠 Research Context

This project focuses on inverse design of metasurfaces for spectral imaging. The training pipeline uses S4-generated transmission spectra across crystallization states and learns both forward and inverse mappings:

1. **Stage A**: shape -> spectra
2. **Stage B**: spectra -> shape
3. **Stage C**: spectra -> shape -> spectra (chain loss fine-tuning)

The preprocessing/training code currently assumes:

- 11 crystallization states (`c = 0.0 ... 1.0`)
- 100 wavelength bins per state
- shape representation as up to 4 Q1 points with `[presence, x, y]`

## 🗂️ Repository Layout (Core Path)

```text
.
├── ms.sh / ms_final.sh / ms_resume_allargs.sh
├── metasurface_final.lua / metasurface_allargs_resume.lua / metasurface_seed.lua
├── merge_s4_data_full.py
├── three_stage_transmittance.py
├── three_stage_transmittance_evaluation.py
├── FilterShapeS4_Evaluator_Transmittance.py
├── results/                # raw S4 output CSVs
├── shapes/                 # generated polygon vertices
├── merged_csvs/            # merged CSVs used for preprocessing
├── outputs_three_stage_*/  # checkpoints, losses, visualizations
├── partial_crys_data/
└── iccp.yaml
```

## ⚙️ Prerequisites

| Dependency | Requirement |
|---|---|
| OS | Linux |
| Shell | Bash |
| Python | 3.9 |
| Env manager | Conda (recommended) |
| RCWA binary | `../build/S4` (relative to repo root) |
| GPU | Optional, recommended for faster training |

## 🚀 Setup

```bash
git clone <repo-url> inverse_metasurface
cd inverse_metasurface

conda env create -f iccp.yaml
conda activate iccp

# Required by launcher scripts
ls -l ../build/S4
```

Optional:

```bash
chmod +x ms.sh ms_final.sh ms_resume_allargs.sh
```

## 🧪 End-to-End Usage

### 1) Generate RCWA data with S4

```bash
./ms_final.sh \
  -ns 10000 \
  -r 12345 \
  -p myrun \
  -g 80 \
  -bo 0.35 \
  -ro 0.30
```

Notes:

- Launchers call `../build/S4` with `-t 32` and run `NQ=1..4` in parallel.
- `ms_final.sh` uses `metasurface_final.lua`.
- `ms_resume_allargs.sh` uses `metasurface_allargs_resume.lua`.

### 2) Merge RCWA outputs with shape vertices

```bash
python merge_s4_data_full.py --prefix myrun
# -> merged_s4_shapes_myrun.csv
```

### 3) Normalize merged column names for training (if needed)

`merge_s4_data_full.py` writes `folder_key` / `NQ`, while the training pipeline expects `prefix` / `nQ`.

```bash
python -c "import pandas as pd; p='merged_s4_shapes_myrun.csv'; df=pd.read_csv(p); df=df.rename(columns={'folder_key':'prefix','NQ':'nQ'}); df.to_csv(p,index=False)"
```

### 4) Preprocess CSV -> NPZ

```bash
mkdir -p merged_csvs
mv merged_s4_shapes_myrun.csv merged_csvs/

python three_stage_transmittance.py \
  --preprocess \
  --input_folder merged_csvs \
  --output_npz preprocessed_t_data.npz
```

### 5) Train three-stage models

```bash
python three_stage_transmittance.py \
  --data_npz preprocessed_t_data.npz \
  --num_epochs 100 \
  --batch_size 1024
```

Main output structure:

- `outputs_three_stage_YYYYMMDD_HHMMSS/stageA`
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageB`
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageC`

### 6) Evaluate checkpoints

```bash
python three_stage_transmittance_evaluation.py \
  --model_dir outputs_three_stage_YYYYMMDD_HHMMSS \
  --data_npz preprocessed_t_data.npz \
  --sample_count 8
```

### 7) Optional: compare neural predictions with S4

```bash
python FilterShapeS4_Evaluator_Transmittance.py \
  --npz_file preprocessed_t_data.npz \
  --spec2shape_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageC/spec2shape_stageC.pt \
  --shape2spec_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageA/shape2spec_stageA.pt \
  --n_samples 4
```

## 🎛️ CLI Reference

### S4 launcher flags (`ms_final.sh`, `ms_resume_allargs.sh`)

| Flag | Meaning | Default |
|---|---|---|
| `-ns`, `--numshapes` | Number of shapes | `100000` |
| `-r`, `--seed` | Random seed | `88888` |
| `-p`, `--prefix` | Run prefix / resume key | `""` |
| `-g`, `--numg` | Geometry basis parameter | `80` |
| `-bo`, `--baseouter` | Base outer offset | `0.25` |
| `-ro`, `--randouter` | Random outer offset | `0.20` |

### Training flags (`three_stage_transmittance.py`)

| Flag | Purpose |
|---|---|
| `--preprocess` | Run preprocessing mode |
| `--input_folder` | Folder of merged CSV files |
| `--output_npz` | Output preprocessed NPZ filename |
| `--data_npz` | NPZ dataset for training |
| `--csv_file` | CSV dataset alternative |
| `--test` | Test mode |
| `--num_epochs` | Training epochs |
| `--batch_size` | Batch size |

### Evaluation flags (`three_stage_transmittance_evaluation.py`)

| Flag | Purpose |
|---|---|
| `--model_dir` | Checkpoint root directory (required) |
| `--data_npz` / `--csv_file` | Evaluation data source |
| `--output_dir` | Evaluation output folder |
| `--sample_count` | Number of visualized samples |
| `--seed` | Random seed for sample selection |
| `--font_scale` | Plot font scaling |
| `--batch_size` | Evaluation batch size |
| `--plot_only` | Only regenerate training-curve plots |

## 🧾 Data Contract (for preprocessing)

The preprocessing path in `three_stage_transmittance.py` expects merged CSVs containing:

- ID columns: `prefix`, `nQ`, `nS`, `shape_idx`, `c`
- geometry text: `vertices_str`
- spectral columns: `T@...`

Quality checks applied by code:

- grouped by `shape_uid = prefix_nQ_nS_shape_idx`
- each group must contain exactly 11 rows
- only shapes with Q1 point count in `[1, 4]` are retained

## 🛠️ Troubleshooting

- `../build/S4: No such file or directory`
  - Build/link S4 at `../build/S4` or edit launcher scripts to your actual S4 path.
- `No matching CSVs found in 'results/'`
  - Verify `--prefix` and output naming in `results/*_output_nQ*_nS*.csv`.
- `No transmission columns found`
  - Ensure merged CSV contains `T@...` columns.
- Preprocess yields zero records
  - Verify required columns and that each shape UID has 11 crystallization rows.
- GPU OOM during training
  - Reduce `--batch_size` (for example `256` or `128`).
- Evaluation cannot find checkpoints
  - Confirm the following exist under `--model_dir`:
    - `stageA/shape2spec_stageA.pt`
    - `stageB/spec2shape_stageB.pt`
    - `stageC/spec2shape_stageC.pt`

## 📚 Citation

If this repository contributes to your research, please cite:

```bibtex
@article{chen2025inverse,
  title={Inverse Design of Metasurface for Spectral Imaging},
  author={Chen, Rongzhou and Nie, Haitao and Zhu, Shuo and Zhao, Yaping and Wang, Chutian and Lam, Edmund Y},
  journal={arXiv preprint arXiv:2510.21924},
  year={2025}
}
```

## 🌐 Language Variants

Additional README variants are available in this repository, including:

- `README.en.md`, `README.de.md`, `README.es.md`, `README.fr.md`
- `README.ru.md`, `README.ja.md`, `README.ko.md`, `README.vi.md`
- `README.ar.md`, `README.zh-CN.md`, `README.zh-TW.md`

## 📌 Notes

- This is a research workspace with many archived and exploratory scripts.
- The canonical transmittance path is centered on:
  - `ms_final.sh` / `ms_resume_allargs.sh`
  - `merge_s4_data_full.py`
  - `three_stage_transmittance.py`
  - `three_stage_transmittance_evaluation.py`
- No explicit license file is currently defined at repository root.

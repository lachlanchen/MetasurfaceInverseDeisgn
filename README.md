<p>
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

<p align="center">
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
  <img alt="Status" src="https://img.shields.io/badge/Status-Research%20Prototype-f59e0b?style=for-the-badge">
  <img alt="Python" src="https://img.shields.io/badge/Python-3.9-3776AB?style=for-the-badge&logo=python&logoColor=white">
  <img alt="PyTorch" src="https://img.shields.io/badge/PyTorch-2.x-EE4C2C?style=for-the-badge&logo=pytorch&logoColor=white">
  <img alt="Simulator" src="https://img.shields.io/badge/RCWA-S4-16a34a?style=for-the-badge">
  <img alt="Platform" src="https://img.shields.io/badge/Platform-Linux%2FBash-6b7280?style=for-the-badge&logo=gnu-bash&logoColor=white">
</p>

A script-first research repository (historically referenced as `inverse_metasurface`) for inverse metasurface design in spectral imaging. The pipeline couples **S4-based RCWA simulation** with a **three-stage PyTorch workflow** for forward and inverse mapping between geometry and optical transmission spectra.

## ✨ At a Glance

| Item | Details |
|---|---|
| 🎯 Goal | Predict C4-symmetric metasurface geometry from target transmittance spectra |
| 🔬 Physics | RCWA simulation with S4 (`../build/S4`) |
| 🧠 Learning pipeline | Stage A `shape -> spectra`, Stage B `spectra -> shape`, Stage C `spectra -> shape -> spectra` |
| 📦 Data form | Merged CSV (`T@...`, shape metadata) -> compressed NPZ (`uids`, `spectra`, `shapes`) |
| 🧪 Evaluation | MSE metrics, qualitative plots, optional S4 re-simulation checks |

## 🧭 End-to-End Workflow

1. Generate metasurface optical responses with S4 (`.lua` + shell launchers).
2. Merge raw simulation CSV files and attach polygon vertices.
3. Convert merged CSV files to training NPZ.
4. Train the three-stage transmittance pipeline.
5. Evaluate checkpoints and visualize Stage A/B/C behavior.
6. Optionally compare neural predictions against fresh S4 simulations.

## 🧱 Repository Structure

```text
.
├── README.md
├── how_to_run.md
├── iccp.yaml
├── pip_requirements.txt
│
├── ms.sh
├── ms_final.sh
├── ms_resume_allargs.sh
├── metasurface_seed.lua
├── metasurface_final.lua
├── metasurface_allargs_resume.lua
│
├── merge_s4_data_full.py
├── three_stage_transmittance.py
├── three_stage_transmittance_evaluation.py
├── FilterShapeS4_Evaluator_Transmittance.py
│
├── results/                # S4 raw CSV outputs
├── shapes/                 # polygon vertex files used during merge
├── merged_csvs/            # merged CSV datasets
├── outputs_three_stage_*/  # training checkpoints and curves
├── partial_crys_data/      # crystallization-state optical tables
│
├── AVIRIS*/                # secondary hyperspectral experiments
├── noise_experiment_*/     # robustness/noise branches
└── archived/               # historical scripts and snapshots
```

## 🛠️ Prerequisites

- Linux + Bash
- Conda (recommended)
- Python 3.9
- S4 binary available at `../build/S4`
- Optional: CUDA-enabled GPU for faster training

## 🚀 Setup

```bash
git clone <your-repo-url> inverse_metasurface
cd inverse_metasurface

conda env create -f iccp.yaml
conda activate iccp

# Verify simulator path expected by scripts
ls -l ../build/S4
```

Optional:

```bash
chmod +x ms.sh ms_final.sh ms_resume_allargs.sh
```

## ▶️ Practical Usage

### 1) Generate RCWA data

`ms_final.sh` and `ms_resume_allargs.sh` each launch 4 parallel S4 jobs (`NQ=1..4`):

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
- `ms.sh` is a simpler launcher path.
- Launchers assume `../build/S4` and use `-t 32`.

### 2) Merge S4 outputs + shape vertices

```bash
python merge_s4_data_full.py --prefix myrun
# output: merged_s4_shapes_myrun.csv
```

### 3) Normalize column names for training compatibility

`merge_s4_data_full.py` writes `folder_key` / `NQ`, while `three_stage_transmittance.py` expects `prefix` / `nQ`.

```bash
python -c "import pandas as pd; p='merged_s4_shapes_myrun.csv'; df=pd.read_csv(p); df=df.rename(columns={'folder_key':'prefix','NQ':'nQ'}); df.to_csv(p,index=False)"
```

### 4) Preprocess CSV to NPZ

```bash
mkdir -p merged_csvs
mv merged_s4_shapes_myrun.csv merged_csvs/

python three_stage_transmittance.py \
  --preprocess \
  --input_folder merged_csvs \
  --output_npz preprocessed_t_data.npz
```

### 5) Train the three-stage pipeline

```bash
python three_stage_transmittance.py \
  --data_npz preprocessed_t_data.npz \
  --num_epochs 100 \
  --batch_size 1024
```

Expected output pattern:
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageA`
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageB`
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageC`

### 6) Evaluate Stage A/B/C models

```bash
python three_stage_transmittance_evaluation.py \
  --model_dir outputs_three_stage_YYYYMMDD_HHMMSS \
  --data_npz preprocessed_t_data.npz \
  --sample_count 8
```

### 7) Optional neural-vs-S4 consistency check

```bash
python FilterShapeS4_Evaluator_Transmittance.py \
  --npz_file preprocessed_t_data.npz \
  --spec2shape_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageC/spec2shape_stageC.pt \
  --shape2spec_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageA/shape2spec_stageA.pt \
  --n_samples 4
```

## ⚙️ Key CLI Options

### S4 Launcher (`ms_final.sh`, `ms_resume_allargs.sh`)

| Flag | Meaning | Default |
|---|---|---|
| `-ns`, `--numshapes` | Number of shapes to generate | `100000` |
| `-r`, `--seed` | Random seed | `88888` |
| `-p`, `--prefix` | Prefix/resume key | `""` |
| `-g`, `--numg` | Basis/geometry parameter | `80` |
| `-bo`, `--baseouter` | Base outer boundary offset | `0.25` |
| `-ro`, `--randouter` | Random outer offset | `0.20` |

### Training (`three_stage_transmittance.py`)

| Flag | Purpose | Default |
|---|---|---|
| `--preprocess` | Run preprocessing mode | `False` |
| `--input_folder` | Folder with merged CSV files | `""` |
| `--output_npz` | Output NPZ path | `preprocessed_data.npz` |
| `--data_npz` | NPZ used for training | `""` |
| `--csv_file` | CSV fallback if NPZ is not used | `""` |
| `--test` | Test mode (skip training) | `False` |
| `--num_epochs` | Number of epochs | `10` |
| `--batch_size` | Batch size | `4096` |

### Evaluation (`three_stage_transmittance_evaluation.py`)

| Flag | Purpose | Default |
|---|---|---|
| `--model_dir` | Root directory containing `stageA/B/C` | required |
| `--data_npz` | NPZ input for evaluation | `""` |
| `--csv_file` | CSV input for evaluation | `""` |
| `--output_dir` | Output directory override | auto in `model_dir` |
| `--sample_count` | Number of visualized samples | `4` |
| `--seed` | Random seed for sampling | `23` |
| `--font_scale` | Plot font scaling | `1.0` |
| `--batch_size` | Eval batch size | `32` |
| `--plot_only` | Plot curves only | `False` |

### S4 Consistency Evaluator (`FilterShapeS4_Evaluator_Transmittance.py`)

| Flag | Purpose | Default |
|---|---|---|
| `--npz_file` | Preprocessed NPZ file | `preprocessed_t_data.npz` |
| `--spec2shape_ckpt` | Stage C checkpoint | `outputs_three_stage_20250322_145925/stageC/spec2shape_stageC.pt` |
| `--shape2spec_ckpt` | Stage A checkpoint | `outputs_three_stage_20250322_145925/stageA/shape2spec_stageA.pt` |
| `--n_samples` | Samples to inspect | `4` |
| `--seed` | Random seed | `23` |
| `--max_workers` | Parallel S4 workers | `4` |
| `--out_folder` | Output folder | auto timestamp |

## 🧪 Quick Smoke Run

```bash
./ms_final.sh -ns 1000 -r 42 -p smoke -g 40 -bo 0.25 -ro 0.20
python merge_s4_data_full.py --prefix smoke
python -c "import pandas as pd; p='merged_s4_shapes_smoke.csv'; d=pd.read_csv(p).rename(columns={'folder_key':'prefix','NQ':'nQ'}); d.to_csv(p,index=False)"
python three_stage_transmittance.py --preprocess --input_folder . --output_npz smoke.npz
python three_stage_transmittance.py --data_npz smoke.npz --num_epochs 5 --batch_size 128
```

## 🔬 Research Context

The core inverse-design setting infers C4-symmetric metasurface geometry from transmittance spectra across crystallization states. The current transmittance path assumes:

- 11 crystallization states per sample (`c` values, sorted during preprocessing)
- 100 wavelength bins per state (`T@...` columns)
- up to 4 Q1 vertices encoded as `(presence, x, y)`

This repository also includes exploratory branches (for example `AVIRIS*`, `noise_experiment_*`, and `archived/`) beyond the canonical transmittance pipeline.

## 📚 Citation

If you use this repository or build upon this work, please cite:

```bibtex
@article{chen2025inverse,
  title={Inverse Design of Metasurface for Spectral Imaging},
  author={Chen, Rongzhou and Nie, Haitao and Zhu, Shuo and Zhao, Yaping and Wang, Chutian and Lam, Edmund Y},
  journal={arXiv preprint arXiv:2510.21924},
  year={2025}
}
```

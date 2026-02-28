[English](README.md) · [العربية](i18n/README.ar.md) · [Español](i18n/README.es.md) · [Français](i18n/README.fr.md) · [日本語](i18n/README.ja.md) · [한국어](i18n/README.ko.md) · [Tiếng Việt](i18n/README.vi.md) · [中文 (简体)](i18n/README.zh-Hans.md) · [中文（繁體）](i18n/README.zh-Hant.md) · [Deutsch](i18n/README.de.md) · [Русский](i18n/README.ru.md)


[![LazyingArt banner](https://github.com/lachlanchen/lachlanchen/raw/main/figs/banner.png)](https://github.com/lachlanchen/lachlanchen/blob/main/figs/banner.png)

# Inverse Design of Metasurface for Spectral Imaging

<p align="center">
  <img alt="Status" src="https://img.shields.io/badge/Status-Research%20Prototype-f59e0b?style=for-the-badge">
  <img alt="Python" src="https://img.shields.io/badge/Python-3.9-3776AB?style=for-the-badge&logo=python&logoColor=white">
  <img alt="PyTorch" src="https://img.shields.io/badge/PyTorch-2.x-EE4C2C?style=for-the-badge&logo=pytorch&logoColor=white">
  <img alt="Simulator" src="https://img.shields.io/badge/Simulator-S4%20RCWA-16a34a?style=for-the-badge">
  <img alt="Platform" src="https://img.shields.io/badge/Platform-Linux%20%2B%20Bash-6b7280?style=for-the-badge&logo=gnu-bash&logoColor=white">
  <img alt="ArXiv" src="https://img.shields.io/badge/arXiv-2510.21924-b31b1b?style=for-the-badge&logo=arxiv&logoColor=white">
</p>

A script-first research repository (historically referred to as `inverse_metasurface`) for inverse metasurface design in spectral imaging.

The core workflow couples:
- Physics-grounded RCWA simulation (`S4` + Lua)
- Data assembly and shape attachment
- Three-stage PyTorch learning (`shape -> spectrum`, `spectrum -> shape`, chained fine-tuning)
- Quantitative and qualitative evaluation, with optional neural-vs-S4 consistency checks

> [!IMPORTANT]
> Canonical behavior and commands are preserved from the existing project scripts/docs. Where historical references point to absent files, those references are intentionally kept with explicit notes for compatibility.

## 📑 Contents

- [🌟 Snapshot](#-snapshot)
- [✨ At a Glance](#-at-a-glance)
- [🌍 Internationalization (i18n)](#-internationalization-i18n)
- [✨ Features](#-features)
- [🧭 End-to-End Workflow](#-end-to-end-workflow)
- [🧱 Project Structure](#-project-structure)
- [🛠️ Prerequisites](#️-prerequisites)
- [🚀 Installation](#-installation)
- [▶️ Usage](#️-usage)
- [⚙️ Configuration](#️-configuration)
- [🧪 Examples](#-examples)
- [🔬 Research Context](#-research-context)
- [🧑‍💻 Development Notes](#-development-notes)
- [🧯 Troubleshooting](#-troubleshooting)
- [🗺️ Roadmap](#️-roadmap)
- [🤝 Contribution](#-contribution)
- [📄 License](#-license)
- [📚 Citation](#-citation)

## 🌟 Snapshot

| Focus | Status |
|---|---|
| 🧠 Objective | Inverse reconstruction of C4-symmetric metasurface geometry from spectral data |
| 🔧 Core stack | S4 RCWA (`Lua`) + PyTorch training + optional geometry-to-spectrum revalidation |
| 🧪 Data pipeline | CSV merge/shape attachment → NPZ (`uids`, `spectra`, `shapes`) |
| 🚀 Readiness | Research prototype; scripts and docs kept compatible with historical references |

## ✨ At a Glance

| Item | Details |
|---|---|
| 🎯 Main task | Infer C4-symmetric metasurface geometry from target transmittance spectra |
| 🔬 Simulator | `../build/S4` called by shell launchers and `.lua` scripts |
| 🧠 Learning pipeline | Stage A `shape -> spectra`, Stage B `spectra -> shape`, Stage C `spectra -> shape -> spectra` |
| 📦 Data contract | Merged CSV (`T@...`, metadata, `vertices_str`) -> compressed NPZ (`uids`, `spectra`, `shapes`) |
| 🧪 Evaluation | MSE metrics, stage visualizations, optional fresh S4 re-simulation |
| 🌐 i18n status | Root-level multilingual README files + existing `i18n/` directory |

## 🌍 Internationalization (i18n)

- Multilingual READMEs are maintained at repository root as `README.<lang>.md` files.
- `i18n/` directory exists in this repository snapshot.
- This file keeps a single language-options line at the top to avoid duplicated language bars.
- `README.en.md` also exists in the repo; this `README.md` remains the canonical base for this update pass.

## ✨ Features

- End-to-end inverse-design path from S4 simulation output to trained inverse model.
- C4-symmetric polygon parameterization and Q1-point encoding (`4x3`: `presence, x, y`).
- Three-stage model training in one script (`three_stage_transmittance.py`).
- Merge tooling that preserves spectral precision and attaches per-shape vertices.
- Optional evaluator that compares learned predictions with fresh S4 simulation.
- Extensive exploratory branches (AVIRIS, SWIR/noise, GSST, archived/deprecated variants).

## 🧭 End-to-End Workflow

1. Generate simulation outputs in `results/` and polygon files in `shapes/`.
2. Merge S4 CSV files and attach shape vertices.
3. Normalize merged column names for training compatibility.
4. Preprocess merged CSV files into NPZ tensors.
5. Train Stage A/B/C models.
6. Evaluate checkpoints and visualize behavior.
7. Optionally compare predicted-shape spectra against new S4 runs.

## 🧱 Project Structure

```text
.
├── README.md
├── README.<lang>.md
├── how_to_run.md
├── commands.md
├── commands_updated.md
├── iccp.yaml
│
├── ms.sh
├── ms_final.sh
├── ms_resume.sh
├── ms_resume_allargs.sh
├── ms_resume_random_state.sh
├── ms_resume_random_state_nir.sh
│
├── metasurface_seed.lua
├── metasurface_final.lua
├── metasurface_seed_resume.lua
├── metasurface_allargs_resume.lua
├── metasurface_resume_random_state.lua
├── metasurface_resume_random_state_nir.lua
├── metasurface_fixed_shape_and_c_value.lua
├── metasurface_unique_shape.lua
├── metasurface_gsst_nir.lua
├── run_prediction.lua
│
├── merge_s4_data_full.py
├── three_stage_transmittance.py
├── FilterShapeS4_Evaluator_Transmittance.py
├── FilterShapeS4_Evaluator_Transmittance_Five_Rows.py
├── FilterShapeS4_Evaluator_Transmittance_Five_Rows_Inferno.py
│
├── shapes/
├── gsst_partial_crys_data/
├── outputs_three_stage_*/
├── blind_noise_experiment_all_*/
├── FilterShapeS4_Evaluator_Transmittance_*/
├── AVIRIS / aviris_*.py
├── noise_experiment*.py
├── archived/
├── deprecated/
├── deprecated-part2/
├── deprecated-scripts/
└── deprecated_code/
```

## 🛠️ Prerequisites

| Dependency | Notes |
|---|---|
| Linux + Bash | Launcher scripts target shell execution |
| Python 3.9 | Matches `iccp.yaml` (`python=3.9.18`) |
| Conda | Recommended for reproducibility |
| S4 binary | Expected at `../build/S4` |
| CUDA GPU (optional) | Speeds up training/evaluation |

## 🚀 Installation

### 1) Clone and enter

```bash
git clone <your-repo-url> inverse_metasurface
cd inverse_metasurface
```

### 2) Create environment (recommended)

```bash
conda env create -f iccp.yaml
conda activate iccp
```

Alternative note:

```bash
# Historical README reference (file may be absent in this snapshot)
pip install -r pip_requirements.txt
```

### 3) Verify simulator path expected by scripts

```bash
ls -l ../build/S4
```

### 4) (Optional) make launchers executable

```bash
chmod +x ms.sh ms_final.sh ms_resume.sh ms_resume_allargs.sh ms_resume_random_state.sh ms_resume_random_state_nir.sh
```

## ▶️ Usage

### A) Generate RCWA simulation data

Simple launcher:

```bash
./ms.sh -ns 10000 -r 12345
```

Parameterized launcher:

```bash
./ms_final.sh \
  -ns 10000 \
  -r 12345 \
  -p myrun \
  -g 80 \
  -bo 0.35 \
  -ro 0.30
```

Resume-oriented launcher:

```bash
./ms_resume_allargs.sh \
  -ns 10000 \
  -r 12345 \
  -p myrun \
  -g 80 \
  -bo 0.35 \
  -ro 0.30
```

Additional resume/random-state example (from command docs):

```bash
./ms_resume_random_state.sh \
  -p iccp100kG20Ov \
  -r 88888 \
  -g 20 \
  -bo 0.35 \
  -ro 0.3 \
  -ns 100000
```

Notes:
- Launchers run `NQ=1..4` in parallel.
- Scripts call `../build/S4` with `-t 32`.

### B) Merge S4 outputs and attach shape vertices

```bash
python merge_s4_data_full.py --prefix myrun
# output: merged_s4_shapes_myrun.csv
```

### C) Normalize columns for training compatibility

`merge_s4_data_full.py` writes `folder_key` and `NQ`, while the training path expects `prefix` and `nQ`.

```bash
python -c "import pandas as pd; p='merged_s4_shapes_myrun.csv'; df=pd.read_csv(p); df=df.rename(columns={'folder_key':'prefix','NQ':'nQ'}); df.to_csv(p,index=False)"
```

### D) Preprocess CSV -> NPZ

```bash
mkdir -p merged_csvs
mv merged_s4_shapes_myrun.csv merged_csvs/

python three_stage_transmittance.py \
  --preprocess \
  --input_folder merged_csvs \
  --output_npz preprocessed_t_data.npz
```

### E) Train Stage A/B/C

```bash
python three_stage_transmittance.py \
  --data_npz preprocessed_t_data.npz \
  --num_epochs 100 \
  --batch_size 1024
```

Outputs are written to:
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageA`
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageB`
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageC`

### F) Evaluate trained models

Historical README command (script name retained for compatibility with prior docs):

```bash
python three_stage_transmittance_evaluation.py \
  --model_dir outputs_three_stage_YYYYMMDD_HHMMSS \
  --data_npz preprocessed_t_data.npz \
  --sample_count 8
```

Repository status note: `three_stage_transmittance_evaluation.py` is not present in this snapshot. Use `FilterShapeS4_Evaluator_Transmittance.py` for available evaluation functionality.

### G) Optional neural-vs-S4 consistency check

```bash
python FilterShapeS4_Evaluator_Transmittance.py \
  --npz_file preprocessed_t_data.npz \
  --spec2shape_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageC/spec2shape_stageC.pt \
  --shape2spec_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageA/shape2spec_stageA.pt \
  --n_samples 4
```

## ⚙️ Configuration

### S4 launchers (`ms_final.sh`, `ms_resume_allargs.sh`)

| Flag | Meaning | Default |
|---|---|---|
| `-ns`, `--numshapes` | Number of shapes to generate | `100000` |
| `-r`, `--seed` | Random seed | `88888` |
| `-p`, `--prefix` | Prefix/resume key | `""` |
| `-g`, `--numg` | Basis/grid parameter | `80` |
| `-bo`, `--baseouter` | Base outer boundary offset | `0.25` |
| `-ro`, `--randouter` | Random outer boundary offset | `0.20` |

### Training (`three_stage_transmittance.py`)

| Flag | Meaning | Default |
|---|---|---|
| `--preprocess` | Run preprocessing mode | `False` |
| `--input_folder` | Folder containing merged CSV files | `""` |
| `--output_npz` | Output NPZ path | `preprocessed_data.npz` |
| `--data_npz` | NPZ dataset for training | `""` |
| `--csv_file` | CSV fallback if NPZ not used | `""` |
| `--test` | Test mode | `False` |
| `--num_epochs` | Number of training epochs | `10` |
| `--batch_size` | Batch size | `4096` |

### Historical evaluation config (`three_stage_transmittance_evaluation.py`)

| Flag | Meaning | Default |
|---|---|---|
| `--model_dir` | Directory containing `stageA/B/C` | required |
| `--data_npz` | NPZ input | `""` |
| `--csv_file` | CSV input fallback | `""` |
| `--output_dir` | Output directory override | auto under `model_dir` |
| `--sample_count` | Number of visualized samples | `4` |
| `--seed` | Random seed | `23` |
| `--font_scale` | Plot font scaling | `1.0` |
| `--batch_size` | Evaluation batch size | `32` |
| `--plot_only` | Plot training curves only | `False` |

### S4 consistency evaluator (`FilterShapeS4_Evaluator_Transmittance.py`)

| Flag | Meaning | Default |
|---|---|---|
| `--npz_file` | Input NPZ file | `preprocessed_t_data.npz` |
| `--spec2shape_ckpt` | Stage C checkpoint path | `outputs_three_stage_20250322_145925/stageC/spec2shape_stageC.pt` |
| `--shape2spec_ckpt` | Stage A checkpoint path | `outputs_three_stage_20250322_145925/stageA/shape2spec_stageA.pt` |
| `--n_samples` | Number of evaluated samples | `4` |
| `--seed` | Random seed | `23` |
| `--max_workers` | S4 worker threads | `4` |
| `--out_folder` | Output directory | auto timestamp |

## 🧪 Examples

### Smoke run

```bash
./ms_final.sh -ns 1000 -r 42 -p smoke -g 40 -bo 0.25 -ro 0.20
python merge_s4_data_full.py --prefix smoke
python -c "import pandas as pd; p='merged_s4_shapes_smoke.csv'; d=pd.read_csv(p).rename(columns={'folder_key':'prefix','NQ':'nQ'}); d.to_csv(p,index=False)"
mkdir -p merged_csvs && mv merged_s4_shapes_smoke.csv merged_csvs/
python three_stage_transmittance.py --preprocess --input_folder merged_csvs --output_npz smoke.npz
python three_stage_transmittance.py --data_npz smoke.npz --num_epochs 5 --batch_size 128
```

### Run-profile examples (from `commands.md` / `commands_updated.md`)

```bash
# no overlap, G=40
./ms_resume_allargs.sh -ns 10000 -r 12345 -p fast_without_overlap -g 40 -bo 0.25 -ro 0.2

# overlap, G=80
./ms_resume_allargs.sh -ns 10000 -r 12345 -p overlap -g 80 -bo 0.35 -ro 0.3

# large run, overlap, G=80
./ms_resume_allargs.sh -ns 100000 -r 12345 -p more_basis_overlap -g 80 -bo 0.35 -ro 0.3
```

## 🔬 Research Context

The current inverse-design setup learns to recover C4-symmetric geometry from transmittance across crystallization states. The transmittance pipeline currently assumes:

- 11 crystallization rows per shape sample (grouped by unique `shape_uid`)
- 100 wavelength bins per crystallization state (`T@...` columns)
- Up to 4 Q1 control points encoded as a `4x3` tensor: `(presence, x, y)`
- Polygon reconstruction under C4 symmetry for shape visualization and consistency checks

The repository also contains exploratory branches (`AVIRIS*`, `noise_experiment*`, `archived/`) beyond the primary transmittance training path.

## 🧑‍💻 Development Notes

- This is a script-centric research repository rather than a packaged Python module.
- Core scripts assume relative paths (especially `../build/S4`, `results/`, `shapes/`).
- `.gitignore` excludes many generated experiment artifacts (`*.csv`, `*.npz`, `*.pt`, run folders).
- Some files/directories in historical docs are currently absent in this snapshot; these references are intentionally preserved with notes for compatibility.
- macOS sidecar files (`._*`) are present and may be non-functional metadata artifacts.

## 🧯 Troubleshooting

| Symptom | Likely Cause | Fix |
|---|---|---|
| `../build/S4: No such file or directory` | S4 binary missing at expected relative path | Build or link S4 at `../build/S4`, or update launcher paths |
| `No transmission columns found` | CSV missing `T@...` columns | Re-check merge output format |
| `Must specify either --data_npz or --csv_file` | Missing training/eval data argument | Provide one input explicitly |
| `No valid shapes => SHIFT->Q1->UpTo4` | Invalid/empty `vertices_str` or Q1 filtering removes all samples | Validate shape files and merge output |
| Empty merge output for `--prefix` | Prefix does not match files in `results/` | Check exact filename prefix and rerun merge |
| Evaluation checkpoint missing | Missing `stageA/B/C` checkpoint files | Verify `--model_dir` points to complete output folder |
| `three_stage_transmittance_evaluation.py` not found | Script referenced by historical docs but absent now | Use `FilterShapeS4_Evaluator_Transmittance.py` or restore that script from prior commits |

## 🗺️ Roadmap

- Improve reproducibility with an explicit data-versioning manifest and pinned run configs.
- Consolidate canonical entry points for transmittance, AVIRIS, and noise branches.
- Add automated smoke tests for preprocessing and one mini training epoch.
- Add clearer experiment registry linking output folders to exact command lines.
- Expand multilingual README synchronization workflow (root language files and `i18n/`).

## 🤝 Contribution

Contributions are welcome, especially for reproducibility, testing, and documentation quality.

Suggested process:

1. Open an issue with scope and expected behavior.
2. Create a focused branch.
3. Submit a pull request with runnable commands and outputs.
4. Keep changes scoped to one workflow where possible.

## ❤️ Support

| Donate | PayPal | Stripe |
| --- | --- | --- |
| [![Donate](https://camo.githubusercontent.com/24a4914f0b42c6f435f9e101621f1e52535b02c225764b2f6cc99416926004b7/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f446f6e6174652d4c617a79696e674172742d3045413545393f7374796c653d666f722d7468652d6261646765266c6f676f3d6b6f2d6669266c6f676f436f6c6f723d7768697465)](https://chat.lazying.art/donate) | [![PayPal](https://camo.githubusercontent.com/d0f57e8b016517a4b06961b24d0ca87d62fdba16e18bbdb6aba28e978dc0ea21/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f50617950616c2d526f6e677a686f754368656e2d3030343537433f7374796c653d666f722d7468652d6261646765266c6f676f3d70617970616c266c6f676f436f6c6f723d7768697465)](https://paypal.me/RongzhouChen) | [![Stripe](https://camo.githubusercontent.com/1152dfe04b6943afe3a8d2953676749603fb9f95e24088c92c97a01a897b4942/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f5374726970652d446f6e6174652d3633354246463f7374796c653d666f722d7468652d6261646765266c6f676f3d737472697065266c6f676f436f6c6f723d7768697465)](https://buy.stripe.com/aFadR8gIaflgfQV6T4fw400) |

## 📄 License

No `LICENSE` file is currently present at repository root in this snapshot. Add one to define usage and redistribution terms.

## 📚 Citation

If you use this repository or build on this work, please cite:

```bibtex
@article{chen2025inverse,
  title={Inverse Design of Metasurface for Spectral Imaging},
  author={Chen, Rongzhou and Nie, Haitao and Zhu, Shuo and Zhao, Yaping and Wang, Chutian and Lam, Edmund Y},
  journal={arXiv preprint arXiv:2510.21924},
  year={2025}
}
```

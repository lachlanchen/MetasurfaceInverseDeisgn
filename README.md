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

# Inverse Design of Metasurface for Spectral Imaging

<p align="center">
  <img alt="Status" src="https://img.shields.io/badge/Status-Research%20Prototype-f59e0b?style=for-the-badge">
  <img alt="Python" src="https://img.shields.io/badge/Python-3.9-3776AB?style=for-the-badge&logo=python&logoColor=white">
  <img alt="PyTorch" src="https://img.shields.io/badge/PyTorch-2.x-EE4C2C?style=for-the-badge&logo=pytorch&logoColor=white">
  <img alt="RCWA" src="https://img.shields.io/badge/Simulator-S4%20RCWA-16a34a?style=for-the-badge">
  <img alt="Platform" src="https://img.shields.io/badge/Platform-Linux%20%2B%20Bash-6b7280?style=for-the-badge&logo=gnu-bash&logoColor=white">
</p>

A script-first research repository (historically referred to as `inverse_metasurface`) for inverse metasurface design in spectral imaging. The core workflow couples:

- physics-grounded RCWA simulation (S4 + Lua)
- data assembly and shape attachment
- three-stage PyTorch learning (`shape -> spectrum`, `spectrum -> shape`, chained fine-tuning)
- quantitative and qualitative evaluation, with optional neural-vs-S4 consistency checks

## ✨ At a Glance

| Item | Details |
|---|---|
| 🎯 Main task | Infer C4-symmetric metasurface geometry from target transmittance spectra |
| 🔬 Simulator | `../build/S4` called by shell launchers and `.lua` scripts |
| 🧠 Learning pipeline | Stage A `shape -> spectra`, Stage B `spectra -> shape`, Stage C `spectra -> shape -> spectra` |
| 📦 Data contract | merged CSV (`T@...`, metadata, `vertices_str`) -> compressed NPZ (`uids`, `spectra`, `shapes`) |
| 🧪 Evaluation | MSE metrics, stage visualizations, optional fresh S4 re-simulation |

## 🧭 End-to-End Workflow

1. Generate simulation outputs in `results/` and polygon files in `shapes/`.
2. Merge S4 CSV files and attach shape vertices.
3. Normalize merged column names for training compatibility.
4. Preprocess merged CSV files into NPZ tensors.
5. Train Stage A/B/C models.
6. Evaluate checkpoints and visualize behavior.
7. Optionally compare predicted-shape spectra against new S4 runs.

## 🧱 Repository Layout

```text
.
├── README.md
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
│
├── metasurface_seed.lua
├── metasurface_final.lua
├── metasurface_seed_resume.lua
├── metasurface_allargs_resume.lua
├── metasurface_resume_random_state.lua
├── metasurface_fixed_shape_and_c_value.lua
│
├── merge_s4_data_full.py
├── three_stage_transmittance.py
├── three_stage_transmittance_evaluation.py
├── FilterShapeS4_Evaluator_Transmittance.py
│
├── partial_crys_data/
├── results/
├── shapes/
├── merged_csvs/
├── outputs_three_stage_*/
│
├── AVIRIS*/
├── noise_experiment*/
└── archived/
```

## 🛠️ Prerequisites

| Dependency | Notes |
|---|---|
| Linux + Bash | Launcher scripts target shell execution |
| Python 3.9 | Matches `iccp.yaml` |
| Conda | Recommended for reproducibility |
| S4 binary | Expected at `../build/S4` |
| CUDA GPU (optional) | Speeds up training/evaluation |

## 🚀 Setup

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

Alternative (less controlled):

```bash
pip install -r pip_requirements.txt
```

### 3) Verify simulator path expected by scripts

```bash
ls -l ../build/S4
```

### 4) (Optional) make launchers executable

```bash
chmod +x ms.sh ms_final.sh ms_resume.sh ms_resume_allargs.sh ms_resume_random_state.sh
```

## ▶️ Practical Usage

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

```bash
python three_stage_transmittance_evaluation.py \
  --model_dir outputs_three_stage_YYYYMMDD_HHMMSS \
  --data_npz preprocessed_t_data.npz \
  --sample_count 8
```

### G) Optional neural-vs-S4 consistency check

```bash
python FilterShapeS4_Evaluator_Transmittance.py \
  --npz_file preprocessed_t_data.npz \
  --spec2shape_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageC/spec2shape_stageC.pt \
  --shape2spec_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageA/shape2spec_stageA.pt \
  --n_samples 4
```

## ⚙️ Key CLI Options

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

### Evaluation (`three_stage_transmittance_evaluation.py`)

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

## 🧪 Smoke Run

```bash
./ms_final.sh -ns 1000 -r 42 -p smoke -g 40 -bo 0.25 -ro 0.20
python merge_s4_data_full.py --prefix smoke
python -c "import pandas as pd; p='merged_s4_shapes_smoke.csv'; d=pd.read_csv(p).rename(columns={'folder_key':'prefix','NQ':'nQ'}); d.to_csv(p,index=False)"
mkdir -p merged_csvs && mv merged_s4_shapes_smoke.csv merged_csvs/
python three_stage_transmittance.py --preprocess --input_folder merged_csvs --output_npz smoke.npz
python three_stage_transmittance.py --data_npz smoke.npz --num_epochs 5 --batch_size 128
```

## 🔬 Research Context

The current inverse-design setup learns to recover C4-symmetric geometry from transmittance across crystallization states. The transmittance pipeline currently assumes:

- 11 crystallization rows per shape sample (grouped by unique `shape_uid`)
- 100 wavelength bins per crystallization state (`T@...` columns)
- up to 4 Q1 control points encoded as a `4x3` tensor: `(presence, x, y)`
- polygon reconstruction under C4 symmetry for shape visualization and consistency checks

The repository also contains exploratory branches (`AVIRIS*`, `noise_experiment*`, `archived/`) beyond the primary transmittance training path.

## 🧯 Troubleshooting

| Symptom | Likely Cause | Fix |
|---|---|---|
| `../build/S4: No such file or directory` | S4 binary missing at expected relative path | Build or link S4 at `../build/S4`, or update launcher paths |
| `No transmission columns found` | CSV missing `T@...` columns | Re-check merge output format |
| `Must specify either --data_npz or --csv_file` | Missing training/eval data argument | Provide one input explicitly |
| `No valid shapes => SHIFT->Q1->UpTo4` | Invalid/empty `vertices_str` or Q1 filtering removes all samples | Validate shape files and merge output |
| Empty merge output for `--prefix` | Prefix does not match files in `results/` | Check exact filename prefix and rerun merge |
| Evaluation checkpoint missing | Missing `stageA/B/C` checkpoint files | Verify `--model_dir` points to complete output folder |

## 🤝 Contribution

Contributions are welcome, especially for reproducibility, testing, and documentation quality.

Suggested process:

1. Open an issue with scope and expected behavior.
2. Create a focused branch.
3. Submit a pull request with runnable commands and outputs.
4. Keep changes scoped to one workflow where possible.

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

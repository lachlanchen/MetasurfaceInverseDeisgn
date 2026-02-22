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


# inverse_metasurface ✨

[![Status](https://img.shields.io/badge/Status-Research%20Prototype-orange)](#-projektumfang)
[![Python](https://img.shields.io/badge/Python-3.9-3776AB?logo=python&logoColor=white)](#-umgebung-einrichten)
[![Platform](https://img.shields.io/badge/Platform-Linux-2f2f2f?logo=linux&logoColor=white)](#-voraussetzungen)
[![RCWA](https://img.shields.io/badge/RCWA-S4%20Required-1f9d55)](#-voraussetzungen)
[![PyTorch](https://img.shields.io/badge/Framework-PyTorch-EE4C2C?logo=pytorch&logoColor=white)](#-training-und-evaluierung)
[![License](https://img.shields.io/badge/License-Not%20Specified-lightgrey)](#-lizenz)

Eine skriptzentrierte Forschungs-Codebasis fuer **inverses Metasurface-Design** unter C4-Symmetrie.
Sie kombiniert:

- 🔬 S4/RCWA-Simulation (`.lua` + Bash-Launcher)
- 🧱 Datenzusammenfuehrung und Preprocessing aus rohen Spektren + Form-Vertices
- 🧠 Drei-stufiges PyTorch-Training (`shape -> spectra`, `spectra -> shape`, Chain-Tuning)
- 📊 Evaluierung und Plotting, inklusive Konsistenzchecks zwischen Neuronalen Netzen und S4

## 📌 Inhaltsverzeichnis

- [Projektumfang](#-projektumfang)
- [Forschungskontext](#-forschungskontext)
- [Repository-Struktur](#-repository-struktur)
- [Voraussetzungen](#-voraussetzungen)
- [Umgebung einrichten](#-umgebung-einrichten)
- [Schnellstart](#-schnellstart)
- [End-to-End-Pipeline](#-end-to-end-pipeline)
- [Training und Evaluierung](#-training-und-evaluierung)
- [Wichtige CLI-Optionen](#-wichtige-cli-optionen)
- [Fehlerbehebung](#-fehlerbehebung)
- [Roadmap](#-roadmap)
- [Zitation](#-zitation)
- [Lizenz](#-lizenz)

## 🎯 Projektumfang

Dieses Repository ist stark experimentgetrieben und auf ausfuehrbare Skripte ausgerichtet (keine paketierte Bibliothek).
Der stabilste Workflow ist:

1. S4 ausfuehren, um rohe optische Ausgaben in `results/` und Formen in `shapes/` zu erzeugen
2. CSV-Dateien pro Run zusammenfuehren und pivotieren, sodass trainingsfaehige Tabellendaten entstehen
3. Zusammengefuehrte CSV-Dateien in komprimierte `.npz` konvertieren
4. Eine 3-stufige Transmittanz-Pipeline trainieren
5. Checkpoints evaluieren und Plots/Metriken exportieren

## 🧪 Forschungskontext

### Problemstellung

Die zentrale Inverse-Design-Aufgabe besteht darin, die Metasurface-Geometrie aus Ziel-Transmissionsspektren zu rekonstruieren (und umgekehrt), mit C4-Symmetrieeinschraenkungen und partiellen Kristallisations-Sweeps.

### Datenannahmen in der Transmittanz-Pipeline

| Item | Value |
|---|---|
| Crystallization states per shape | 11 (`c` from `0.0` to `1.0`) |
| Spectral bins per state | 100 |
| Spectrum tensor per sample | `11 x 100` |
| Shape tensor per sample | `4 x 3` (`[presence, x, y]`) |

### Lernziele der drei Stufen

| Stage | Direction | Typical checkpoint |
|---|---|---|
| A | `shape -> spectrum` | `stageA/shape2spec_stageA.pt` |
| B | `spectrum -> shape` | `stageB/spec2shape_stageB.pt` |
| C | `spectrum -> shape -> spectrum` (chain-tuned) | `stageC/spec2shape_stageC.pt` |

## 🗂️ Repository-Struktur

```text
.
├── ms.sh / ms_final.sh / ms_resume_allargs.sh
├── metasurface_seed.lua / metasurface_final.lua / metasurface_allargs_resume.lua
├── merge.py / merge_s4_data_full.py / merge_s4_data_local.py / merge_robust.py
├── three_stage_transmittance.py
├── three_stage_transmittance_evaluation.py
├── FilterShapeS4_Evaluator_Transmittance.py
├── partial_crys_data/
├── results/                          # raw S4 outputs
├── shapes/                           # generated polygon vertices
├── outputs_three_stage_*/            # checkpoints + training artifacts
├── AVIRIS*/ and aviris_*.py          # related hyperspectral experiments
├── commands.md / how_to_run.md
├── iccp.yaml
└── pip_requirements.txt
```

## 🧩 Voraussetzungen

| Dependency | Notes |
|---|---|
| Linux + Bash | Shell scripts assume Linux-style paths |
| Conda | Recommended env management (`iccp.yaml`) |
| Python 3.9 | Primary runtime for training/eval scripts |
| S4 binary | Expected at `../build/S4` relative to repo root |
| CUDA (optional) | Speeds up training and evaluation |

## ⚙️ Umgebung einrichten

```bash
git clone <repo-url> inverse_metasurface
cd inverse_metasurface

conda env create -f iccp.yaml
conda activate iccp

# verify S4 path expected by shell runners
ls -l ../build/S4
```

Optionales Ausfuehrungsbit fuer Shell-Launcher:

```bash
chmod +x ms.sh ms_final.sh ms_resume_allargs.sh ms_resume.sh
```

## 🚀 Schnellstart

Wenn `preprocessed_t_data.npz` bereits vorhanden ist:

```bash
python three_stage_transmittance.py \
  --data_npz preprocessed_t_data.npz \
  --num_epochs 100 \
  --batch_size 1024

python three_stage_transmittance_evaluation.py \
  --model_dir outputs_three_stage_YYYYMMDD_HHMMSS \
  --data_npz preprocessed_t_data.npz \
  --sample_count 8
```

## 🔁 End-to-End-Pipeline

### 1) S4-Daten erzeugen

```bash
./ms_final.sh \
  -ns 10000 \
  -r 12345 \
  -p myrun \
  -g 80 \
  -bo 0.35 \
  -ro 0.30
```

### 2) S4-Ausgaben + Form-Vertices zusammenfuehren

```bash
python merge_s4_data_full.py --prefix myrun
# -> merged_s4_shapes_myrun.csv
```

### 3) CSV-Spaltennamen fuer das Preprocessing normalisieren

Das Preprocessing in `three_stage_transmittance.py` erwartet `prefix` und `nQ`, waehrend zusammengefuehrte Ausgaben `folder_key` und `NQ` enthalten koennen.

```bash
python -c "import pandas as pd; p='merged_s4_shapes_myrun.csv'; df=pd.read_csv(p); df=df.rename(columns={'folder_key':'prefix','NQ':'nQ'}); df.to_csv(p,index=False)"
```

### 4) CSV -> NPZ-Preprocessing

```bash
mkdir -p merged_csvs
mv merged_s4_shapes_myrun.csv merged_csvs/

python three_stage_transmittance.py --preprocess \
  --input_folder merged_csvs \
  --output_npz preprocessed_t_data.npz
```

### 5) Drei Stufen trainieren

```bash
python three_stage_transmittance.py \
  --data_npz preprocessed_t_data.npz \
  --num_epochs 100 \
  --batch_size 1024
```

### 6) Evaluieren und plotten

```bash
python three_stage_transmittance_evaluation.py \
  --model_dir outputs_three_stage_YYYYMMDD_HHMMSS \
  --data_npz preprocessed_t_data.npz \
  --sample_count 8
```

### 7) Optional: Vergleich Neuronales Netz vs. S4

```bash
python FilterShapeS4_Evaluator_Transmittance.py \
  --npz_file preprocessed_t_data.npz \
  --spec2shape_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageC/spec2shape_stageC.pt \
  --shape2spec_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageA/shape2spec_stageA.pt \
  --n_samples 4
```

## 🧠 Training und Evaluierung

### Hauptskripte

| Script | Purpose |
|---|---|
| `three_stage_transmittance.py` | preprocess + train stages A/B/C |
| `three_stage_transmittance_evaluation.py` | evaluate checkpoints, compute metrics, save plots |
| `FilterShapeS4_Evaluator_Transmittance.py` | compare learned predictions with S4 behavior |

### Typische Ausgaben

| Artifact | Location |
|---|---|
| Stage checkpoints | `outputs_three_stage_*/stageA|stageB|stageC/` |
| Evaluation figures | `outputs_three_stage_*/evaluation_<timestamp>/` |
| Metrics CSV | `evaluation_metrics.csv`, `metrics_summary.csv` |

## 🛠️ Wichtige CLI-Optionen

### S4-Launcher (`ms_final.sh`, `ms_resume_allargs.sh`)

| Flag | Meaning | Default |
|---|---|---|
| `-ns`, `--numshapes` | number of shapes | `100000` |
| `-r`, `--seed` | random seed | `88888` |
| `-p`, `--prefix` | run prefix/resume key | empty |
| `-g`, `--numg` | geometry basis setting | `80` |
| `-bo`, `--baseouter` | base outer offset | `0.25` |
| `-ro`, `--randouter` | random outer offset | `0.20` |

### Training (`three_stage_transmittance.py`)

| Flag | Meaning | Default |
|---|---|---|
| `--preprocess` | switch to preprocessing mode | off |
| `--input_folder` | folder containing merged CSVs | `""` |
| `--output_npz` | preprocessing output file | `preprocessed_data.npz` |
| `--data_npz` | NPZ input for training | `""` |
| `--csv_file` | direct CSV training input | `""` |
| `--test` | test mode toggle | off |
| `--num_epochs` | epochs per stage | `10` |
| `--batch_size` | batch size | `4096` |

### Evaluierung (`three_stage_transmittance_evaluation.py`)

| Flag | Meaning | Default |
|---|---|---|
| `--model_dir` | trained run directory (required) | - |
| `--data_npz` | NPZ evaluation input | `""` |
| `--csv_file` | CSV evaluation input | `""` |
| `--output_dir` | custom output directory | auto |
| `--sample_count` | number of visualized samples | `4` |
| `--seed` | random seed for sample selection | `23` |
| `--font_scale` | plot font multiplier | `1.0` |
| `--batch_size` | eval dataloader batch size | `32` |
| `--plot_only` | regenerate plots only | off |

## 🧯 Fehlerbehebung

| Symptom | Likely cause | Fix |
|---|---|---|
| `../build/S4: No such file or directory` | S4 binary not at expected relative path | Place/build S4 at `../build/S4` or edit scripts |
| `Must specify either --data_npz or --csv_file` | Missing training/eval dataset arg | Provide exactly one dataset input |
| `No transmission columns found` | Merged CSV lacks `T@...` columns | Re-run merge/pivot and verify headers |
| `KeyError: 'prefix'` in preprocess | Merge output still uses `folder_key`/`NQ` | Rename columns to `prefix`/`nQ` before preprocess |
| GPU OOM | batch too large | Lower `--batch_size` |
| Missing checkpoints during eval | stage checkpoints absent/wrong path | Verify stageA/B/C files under selected `--model_dir` |

## 🧭 Roadmap

- Standardisiere das zusammengefuehrte CSV-Schema ueber alle Merge-Skripte hinweg (`prefix`, `nQ` Benennung)
- Fuege automatisierte Tests fuer Merge/Preprocessing/Checkpoint-Laden hinzu
- Biete einen einzigen CLI-Entrypoint fuer die komplette Pipeline-Orchestrierung
- Fuege Datensatz-/Run-Manifeste fuer Reproduzierbarkeit hinzu
- Fuege eine explizite Open-Source-Lizenz hinzu

## 📚 Zitation

Wenn dieses Repository zu deiner Forschung beitraegt, zitiere bitte:

```bibtex
@article{chen2025inverse,
  title={Inverse Design of Metasurface for Spectral Imaging},
  author={Chen, Rongzhou and Nie, Haitao and Zhu, Shuo and Zhao, Yaping and Wang, Chutian and Lam, Edmund Y},
  journal={arXiv preprint arXiv:2510.21924},
  year={2025}
}
```

## 📄 Lizenz

Derzeit ist in diesem Repository keine `LICENSE`-Datei vorhanden. Nutzungs- und Weiterverteilungsrechte sind daher nicht festgelegt, bis eine Lizenz hinzugefuegt wird.

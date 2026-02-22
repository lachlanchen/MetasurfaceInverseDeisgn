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
  <img alt="Simulator" src="https://img.shields.io/badge/RCWA-S4-16a34a?style=for-the-badge">
  <img alt="Platform" src="https://img.shields.io/badge/Platform-Linux%2FBash-6b7280?style=for-the-badge&logo=gnu-bash&logoColor=white">
</p>

Ein skriptzentriertes Forschungs-Repository (historisch als `inverse_metasurface` bezeichnet) für das inverse Design von Metasurfaces im Bereich Spektralbildgebung. Die Pipeline kombiniert **S4-basierte RCWA-Simulation** mit einem **dreistufigen PyTorch-Workflow** für die Vorwärts- und Inversabbildung zwischen Geometrie und optischen Transmissionsspektren.

## ✨ Auf einen Blick

| Punkt | Details |
|---|---|
| 🎯 Ziel | Vorhersage C4-symmetrischer Metasurface-Geometrie aus Ziel-Transmissionsspektren |
| 🔬 Physik | RCWA-Simulation mit S4 (`../build/S4`) |
| 🧠 Lernpipeline | Stufe A `shape -> spectra`, Stufe B `spectra -> shape`, Stufe C `spectra -> shape -> spectra` |
| 📦 Datenformat | Zusammengeführte CSV (`T@...`, Shape-Metadaten) -> komprimierte NPZ (`uids`, `spectra`, `shapes`) |
| 🧪 Auswertung | MSE-Metriken, qualitative Plots, optionale S4-Neusimulations-Checks |

## 🧭 End-to-End-Workflow

1. Erzeuge optische Metasurface-Antworten mit S4 (`.lua` + Shell-Launcher).
2. Führe rohe Simulations-CSV-Dateien zusammen und ergänze Polygon-Vertices.
3. Konvertiere zusammengeführte CSV-Dateien in Trainings-NPZ.
4. Trainiere die dreistufige Transmittanz-Pipeline.
5. Evaluiere Checkpoints und visualisiere das Verhalten von Stufe A/B/C.
6. Vergleiche optional neuronale Vorhersagen mit frischen S4-Simulationen.

## 🧱 Repository-Struktur

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
├── results/                # rohe S4-CSV-Ausgaben
├── shapes/                 # beim Mergen verwendete Polygon-Vertex-Dateien
├── merged_csvs/            # zusammengeführte CSV-Datensätze
├── outputs_three_stage_*/  # Trainings-Checkpoints und Kurven
├── partial_crys_data/      # optische Tabellen für Kristallisationszustände
│
├── AVIRIS*/                # sekundäre hyperspektrale Experimente
├── noise_experiment_*/     # Robustheits-/Rausch-Zweige
└── archived/               # historische Skripte und Snapshots
```

## 🛠️ Voraussetzungen

- Linux + Bash
- Conda (empfohlen)
- Python 3.9
- S4-Binary verfügbar unter `../build/S4`
- Optional: CUDA-fähige GPU für schnelleres Training

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

## ▶️ Praktische Nutzung

### 1) RCWA-Daten erzeugen

`ms_final.sh` und `ms_resume_allargs.sh` starten jeweils 4 parallele S4-Jobs (`NQ=1..4`):

```bash
./ms_final.sh \
  -ns 10000 \
  -r 12345 \
  -p myrun \
  -g 80 \
  -bo 0.35 \
  -ro 0.30
```

Hinweise:
- `ms.sh` ist ein einfacherer Launcher-Pfad.
- Die Launcher gehen von `../build/S4` aus und verwenden `-t 32`.

### 2) S4-Ausgaben + Shape-Vertices zusammenführen

```bash
python merge_s4_data_full.py --prefix myrun
# output: merged_s4_shapes_myrun.csv
```

### 3) Spaltennamen für Trainingskompatibilität normalisieren

`merge_s4_data_full.py` schreibt `folder_key` / `NQ`, während `three_stage_transmittance.py` `prefix` / `nQ` erwartet.

```bash
python -c "import pandas as pd; p='merged_s4_shapes_myrun.csv'; df=pd.read_csv(p); df=df.rename(columns={'folder_key':'prefix','NQ':'nQ'}); df.to_csv(p,index=False)"
```

### 4) CSV zu NPZ vorverarbeiten

```bash
mkdir -p merged_csvs
mv merged_s4_shapes_myrun.csv merged_csvs/

python three_stage_transmittance.py \
  --preprocess \
  --input_folder merged_csvs \
  --output_npz preprocessed_t_data.npz
```

### 5) Dreistufige Pipeline trainieren

```bash
python three_stage_transmittance.py \
  --data_npz preprocessed_t_data.npz \
  --num_epochs 100 \
  --batch_size 1024
```

Erwartetes Ausgabemuster:
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageA`
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageB`
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageC`

### 6) Modelle von Stufe A/B/C evaluieren

```bash
python three_stage_transmittance_evaluation.py \
  --model_dir outputs_three_stage_YYYYMMDD_HHMMSS \
  --data_npz preprocessed_t_data.npz \
  --sample_count 8
```

### 7) Optionaler Konsistenzcheck: Neuronales Modell vs. S4

```bash
python FilterShapeS4_Evaluator_Transmittance.py \
  --npz_file preprocessed_t_data.npz \
  --spec2shape_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageC/spec2shape_stageC.pt \
  --shape2spec_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageA/shape2spec_stageA.pt \
  --n_samples 4
```

## ⚙️ Wichtige CLI-Optionen

### S4-Launcher (`ms_final.sh`, `ms_resume_allargs.sh`)

| Flag | Bedeutung | Standard |
|---|---|---|
| `-ns`, `--numshapes` | Anzahl zu erzeugender Shapes | `100000` |
| `-r`, `--seed` | Zufalls-Seed | `88888` |
| `-p`, `--prefix` | Präfix/Fortsetzungs-Schlüssel | `""` |
| `-g`, `--numg` | Basis-/Geometrieparameter | `80` |
| `-bo`, `--baseouter` | Basis-Offset der äußeren Grenze | `0.25` |
| `-ro`, `--randouter` | Zufälliger äußerer Offset | `0.20` |

### Training (`three_stage_transmittance.py`)

| Flag | Zweck | Standard |
|---|---|---|
| `--preprocess` | Vorverarbeitungsmodus ausführen | `False` |
| `--input_folder` | Ordner mit zusammengeführten CSV-Dateien | `""` |
| `--output_npz` | Ausgabepfad für NPZ | `preprocessed_data.npz` |
| `--data_npz` | Für das Training verwendete NPZ | `""` |
| `--csv_file` | CSV-Fallback, falls NPZ nicht verwendet wird | `""` |
| `--test` | Testmodus (Training überspringen) | `False` |
| `--num_epochs` | Anzahl der Epochen | `10` |
| `--batch_size` | Batch-Größe | `4096` |

### Auswertung (`three_stage_transmittance_evaluation.py`)

| Flag | Zweck | Standard |
|---|---|---|
| `--model_dir` | Wurzelverzeichnis mit `stageA/B/C` | erforderlich |
| `--data_npz` | NPZ-Eingabe für die Auswertung | `""` |
| `--csv_file` | CSV-Eingabe für die Auswertung | `""` |
| `--output_dir` | Überschreiben des Ausgabeverzeichnisses | automatisch in `model_dir` |
| `--sample_count` | Anzahl visualisierter Samples | `4` |
| `--seed` | Zufalls-Seed für Sampling | `23` |
| `--font_scale` | Schrift-Skalierung für Plots | `1.0` |
| `--batch_size` | Eval-Batch-Größe | `32` |
| `--plot_only` | Nur Kurven plotten | `False` |

### S4-Konsistenz-Evaluator (`FilterShapeS4_Evaluator_Transmittance.py`)

| Flag | Zweck | Standard |
|---|---|---|
| `--npz_file` | Vorverarbeitete NPZ-Datei | `preprocessed_t_data.npz` |
| `--spec2shape_ckpt` | Stage-C-Checkpoint | `outputs_three_stage_20250322_145925/stageC/spec2shape_stageC.pt` |
| `--shape2spec_ckpt` | Stage-A-Checkpoint | `outputs_three_stage_20250322_145925/stageA/shape2spec_stageA.pt` |
| `--n_samples` | Zu prüfende Samples | `4` |
| `--seed` | Zufalls-Seed | `23` |
| `--max_workers` | Parallele S4-Worker | `4` |
| `--out_folder` | Ausgabeordner | automatisch per Zeitstempel |

## 🧪 Schneller Smoke-Run

```bash
./ms_final.sh -ns 1000 -r 42 -p smoke -g 40 -bo 0.25 -ro 0.20
python merge_s4_data_full.py --prefix smoke
python -c "import pandas as pd; p='merged_s4_shapes_smoke.csv'; d=pd.read_csv(p).rename(columns={'folder_key':'prefix','NQ':'nQ'}); d.to_csv(p,index=False)"
python three_stage_transmittance.py --preprocess --input_folder . --output_npz smoke.npz
python three_stage_transmittance.py --data_npz smoke.npz --num_epochs 5 --batch_size 128
```

## 🔬 Forschungskontext

Das zentrale Inverse-Design-Setup erschließt C4-symmetrische Metasurface-Geometrie aus Transmissionsspektren über verschiedene Kristallisationszustände. Der aktuelle Transmittanz-Pfad setzt voraus:

- 11 Kristallisationszustände pro Sample (`c`-Werte, während der Vorverarbeitung sortiert)
- 100 Wellenlängen-Bins pro Zustand (`T@...`-Spalten)
- bis zu 4 Q1-Vertices, codiert als `(presence, x, y)`

Dieses Repository enthält außerdem explorative Zweige (zum Beispiel `AVIRIS*`, `noise_experiment_*` und `archived/`) jenseits der kanonischen Transmittanz-Pipeline.

## 📚 Zitation

Wenn Sie dieses Repository verwenden oder auf dieser Arbeit aufbauen, zitieren Sie bitte:

```bibtex
@article{chen2025inverse,
  title={Inverse Design of Metasurface for Spectral Imaging},
  author={Chen, Rongzhou and Nie, Haitao and Zhu, Shuo and Zhao, Yaping and Wang, Chutian and Lam, Edmund Y},
  journal={arXiv preprint arXiv:2510.21924},
  year={2025}
}
```

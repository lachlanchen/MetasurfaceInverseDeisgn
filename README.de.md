English | 中文繁體 | 中文简体 | 日本語 | 한국어 | Tiếng Việt | العربية | Français | Español | Deutsch | Русский

# inverse_metasurface

<div align="center">

[![Status](https://img.shields.io/badge/status-experimental-orange)](#-roadmap)
[![Python](https://img.shields.io/badge/python-3.9-blue)](#-prerequisites)
[![Platform](https://img.shields.io/badge/platform-linux-lightgrey)](#-prerequisites)
[![S4](https://img.shields.io/badge/S4-required-success)](#-prerequisites)
[![License](https://img.shields.io/badge/license-unset-lightgrey)](#-license)

</div>

Eine Forschungsumgebung für **inverses Metasurface-Design** mit **S4/RCWA-Simulation** und **mehrstufigen neuronalen Modellen**.

Unterstützt werden:
- Hochdurchsatz-S4-Datengenerierung für C4-symmetrische Polygon-Metasurfaces.
- Datenzusammenführung + Vorverarbeitung in trainingsbereite NPZ-Dateien.
- Drei Lernstufen: **shape -> spectrum**, **spectrum -> shape**, **kettentuning spectrum -> shape -> spectrum**.
- Auswertung und optionaler direkter **neural-vs-S4**-Vergleich.

## 🌟 Auf einen Blick

| Bereich | Was dieses Repo bereitstellt |
|---|---|
| Physiksimulation | Bash- + Lua-Skripte, um S4 parallel über `nQ=1..4` auszuführen |
| Dataset-Tooling | Merge-Skripte, die Polygon-Vertices und Pivot-Spektren zusammenführen |
| ML-Pipeline | `three_stage_transmittance.py` mit Preprocess- + Train-Modi |
| Auswertung | `three_stage_transmittance_evaluation.py` mit Metriken + Abbildungen |
| Forschungszweig | AVIRIS-/Hyperspektral- und Rausch-/Komprimierungsexperimente |

## 🧠 Forschungskontext

Dieses Repository richtet sich auf inverses Design für photonische Metasurfaces: Geometrie aus gewünschten Spektren ableiten (und umgekehrt).

Kernannahmen der Baseline-Pipeline:
- C4-Symmetrie über Q1-Punkt-Parametrisierung.
- 11 Kristallisationszustände (`c` in `[0.0, 1.0]`).
- Jedes Sample-Spektrum als `11 x 100`-Transmissionszeilen gespeichert.
- Formen als bis zu 4 Q1-Punkte dargestellt (`4 x 3`-Tensor: Präsenz, x, y).

Dreistufige Trainingslogik:
1. **Stage A**: shape -> spectrum (`shape2spec_stageA.pt`)
2. **Stage B**: spectrum -> shape (`spec2shape_stageB.pt`)
3. **Stage C**: spectrum -> shape -> spectrum chain fine-tuning (`spec2shape_stageC.pt`)

## 🗂 Projektstruktur

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

## ✅ Voraussetzungen

| Anforderung | Hinweise |
|---|---|
| OS | Linux (Skripte setzen Bash + Linux-Pfade voraus) |
| Python | 3.9 (aus `iccp.yaml`) |
| Umgebungsmanager | Conda empfohlen |
| S4-Binary | Erwartet unter `../build/S4` relativ zum Repo-Root |
| GPU (optional) | CUDA beschleunigt Training/Auswertung |

## ⚙️ Installation

### 1) Klonen und wechseln

```bash
git clone <your-repo-url> inverse_metasurface
cd inverse_metasurface
```

### 2) Umgebung erstellen

```bash
conda env create -f iccp.yaml
conda activate iccp
```

### 3) S4-Pfad prüfen

```bash
ls -l ../build/S4
```

Wenn dieser Pfad nicht existiert, baue/platziere S4 dort oder passe die Skriptpfade an.

## 🚀 Schnellstart (End-to-End)

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

## 🧪 Nutzungsdetails

### A) S4-Generierung

Minimaler Lauf mit Seed:

```bash
./ms.sh -ns 10000 -r 88888
```

Parametrisierter Lauf:

```bash
./ms_final.sh \
  -ns 100000 \
  -r 88888 \
  -p iccpOv100kG80 \
  -g 80 \
  -bo 0.35 \
  -ro 0.30
```

Resume-ähnlicher Lauf:

```bash
./ms_resume_allargs.sh \
  -ns 100000 \
  -r 88888 \
  -p iccpOv100kG80 \
  -g 80 \
  -bo 0.35 \
  -ro 0.30
```

### B) Roh-S4-CSVs zusammenführen

```bash
python merge.py --prefix 20250123_155420
```

Alternative Utility:

```bash
python merge_s4_data_full.py --prefix 20250123_155420
```

### C) Zusammengeführte CSVs vorverarbeiten -> NPZ

```bash
python three_stage_transmittance.py --preprocess \
  --input_folder merged_csvs \
  --output_npz preprocessed_t_data.npz
```

### D) Trainieren

```bash
python three_stage_transmittance.py \
  --data_npz preprocessed_t_data.npz \
  --num_epochs 100 \
  --batch_size 1024
```

Training-Outputs werden gespeichert unter:
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageA`
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageB`
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageC`

### E) Auswerten

```bash
python three_stage_transmittance_evaluation.py \
  --model_dir outputs_three_stage_YYYYMMDD_HHMMSS \
  --data_npz preprocessed_t_data.npz \
  --sample_count 8
```

Erzeugt:
- `evaluation_metrics.csv`
- `metrics_summary.csv`
- Stage-Visualisierungen (`.png`, `.pdf`)
- Training-Curve-Plots

### F) Neural vs direkte S4-Vergleichsauswertung (optional)

```bash
python FilterShapeS4_Evaluator_Transmittance.py \
  --npz_file preprocessed_t_data.npz \
  --spec2shape_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageC/spec2shape_stageC.pt \
  --shape2spec_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageA/shape2spec_stageA.pt \
  --n_samples 4
```

## 🔧 Wichtige CLI-Optionen

### S4-Runner-Flags (`ms_final.sh`, `ms_resume_allargs.sh`)

| Flag | Bedeutung | Standard |
|---|---|---|
| `-ns`, `--numshapes` | Anzahl der Formen | `100000` |
| `-r`, `--seed` | Zufalls-Seed | `88888` |
| `-p`, `--prefix` | Run-Präfix / Resume-Key | leer |
| `-g`, `--numg` | Geometrie-/Basisparameter | `80` |
| `-bo`, `--baseouter` | Basis-Außenoffset | `0.25` |
| `-ro`, `--randouter` | Zufälliger Außenoffset | `0.20` |

### Trainings-Flags (`three_stage_transmittance.py`)

| Flag | Bedeutung | Standard |
|---|---|---|
| `--preprocess` | Vorverarbeitungsmodus ausführen | aus |
| `--input_folder` | Eingabe-CSV-Ordner | leer |
| `--output_npz` | Ausgabe-NPZ-Pfad | `preprocessed_data.npz` |
| `--data_npz` | NPZ für Training/Auswertung | leer |
| `--csv_file` | Direkt verwendete CSV | leer |
| `--num_epochs` | Epochen pro Stage | `10` |
| `--batch_size` | Batch-Größe | `4096` |
| `--test` | Platzhalter für Testmodus | aus |

### Evaluations-Flags (`three_stage_transmittance_evaluation.py`)

| Flag | Bedeutung | Standard |
|---|---|---|
| `--model_dir` | Root-Verzeichnis eines trainierten Runs | erforderlich |
| `--data_npz` | NPZ für Auswertung | leer |
| `--csv_file` | CSV für Auswertung | leer |
| `--output_dir` | Ausgabeordner | `model_dir/evaluation_<timestamp>` |
| `--sample_count` | Anzahl visualisierter Samples | `4` |
| `--seed` | Sampling-Seed | `23` |
| `--font_scale` | Font-Skalierung für Plots | `1.0` |
| `--batch_size` | Eval-Batch-Größe | `32` |
| `--plot_only` | Nur Kurven plotten | aus |

## 🧭 Fehlerbehebung

| Symptom | Wahrscheinliche Ursache | Lösung |
|---|---|---|
| `../build/S4: No such file or directory` | S4-Binary-Pfad passt nicht | S4 unter `../build/S4` bauen/ablegen oder Skripte anpassen |
| `Must specify either --data_npz or --csv_file` | Fehlende Dataset-Quelle | Einen dieser Flags explizit setzen |
| `No matching CSVs found in 'results/'` | Präfix stimmt nicht überein | Präfix und Output-Benennung prüfen |
| Sehr wenige/keine vorverarbeiteten Records | Fehlende/ungültige `T@...`- oder `vertices_str`-Zeilen | Zusammengeführtes CSV-Schema und Zeilengruppierung pro Shape validieren |
| CUDA OOM | Batch zu groß | `--batch_size` reduzieren (z. B. `1024 -> 256`) |

## 🧱 Hinweise zur Entwicklung

- Dies ist ein stark experimentelles Repo; viele erzeugte Artefakte sind absichtlich nicht versioniert.
- Für ähnliche Aufgaben existieren mehrere Skriptvarianten (`merge_*`, `aviris_*`, `noise_experiment_*`).
- Der dokumentierte Baseline-Workflow für inverse Metasurfaces ist der in dieser README beschriebene Pfad.
- Trainingsskripte setzen Seeds (`42`), strikte Deterministik hängt aber weiterhin von Hardware-/Backend-Verhalten ab.
- Es gibt derzeit keine einheitliche CI + vollständige automatisierte Testsuite.

## 🛣 Roadmap

- Ein kompaktes kanonisches Dataset für schnelle Smoke-Tests hinzufügen.
- Doppelte Pipeline-Skripte konsolidieren.
- Checks für Merge-/Preprocess-Integrität und Checkpoint-Ladefähigkeit hinzufügen.
- CI für Lint + Smoke-Train/Eval hinzufügen.
- S4-Build-/Versions-Pinning expliziter dokumentieren.

## 🤝 Beitrag

1. Einen Feature-Branch erstellen.
2. PR-Umfang eng halten (ein Pipeline-/Experiment-Thema pro PR).
3. Exakt ausführbare Befehle und erwartete Output-Pfade angeben.
4. Große erzeugte Artefakte nur committen, wenn erforderlich.
5. Reproduzierbarkeitsnotizen ergänzen (Seed, Datenquelle, Checkpoint-Pfad).

## 📄 Lizenz

Derzeit ist keine `LICENSE`-Datei in diesem Repository vorhanden.

Bis eine Lizenzdatei hinzugefügt wird, sollten Wiederverwendung und Weiterverteilung als **nicht festgelegt** betrachtet werden.

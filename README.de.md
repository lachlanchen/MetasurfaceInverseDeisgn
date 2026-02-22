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

Ein skriptorientiertes Forschungs-Repository (historisch als `inverse_metasurface` bezeichnet) für das **inverse Design C4-symmetrischer Metaflächen** in der Spektralbildgebung, einschließlich:

- RCWA-Datengenerierung mit S4 (`.lua` + Shell-Launcher)
- Datenzusammenführung und Vorverarbeitung (`.csv` -> `.npz`)
- dreistufiges neuronales Training (Form->Spektren, Spektren->Form, Ketten-Feinabstimmung)
- Auswertung und optionale Neural-vs-S4-Prüfungen

## ✨ Auf einen Blick

| Punkt | Details |
|---|---|
| Kernziel | Geometrie aus vorgegebenen Transmissionsspektren vorhersagen |
| Kern-Datenform | Spektren: `11 x 100`, Form: `4 x 3` |
| Haupt-Trainingsskript | `three_stage_transmittance.py` |
| Haupt-Auswertungsskript | `three_stage_transmittance_evaluation.py` |
| RCWA-Launcher | `ms_final.sh`, `ms_resume_allargs.sh` |
| Merge-Skript | `merge_s4_data_full.py` |

## 🧠 Forschungskontext

Dieses Projekt konzentriert sich auf das inverse Design von Metaflächen für die Spektralbildgebung. Die Trainingspipeline verwendet mit S4 erzeugte Transmissionsspektren über verschiedene Kristallisationszustände hinweg und lernt sowohl Vorwärts- als auch inverse Abbildungen:

1. **Stufe A**: Form -> Spektren
2. **Stufe B**: Spektren -> Form
3. **Stufe C**: Spektren -> Form -> Spektren (Kettenverlust-Feinabstimmung)

Der aktuelle Vorverarbeitungs-/Trainingscode setzt Folgendes voraus:

- 11 Kristallisationszustände (`c = 0.0 ... 1.0`)
- 100 Wellenlängen-Bins pro Zustand
- Formrepräsentation als bis zu 4 Q1-Punkte mit `[presence, x, y]`

## 🗂️ Repository-Struktur (Kernpfad)

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

## ⚙️ Voraussetzungen

| Abhängigkeit | Anforderung |
|---|---|
| OS | Linux |
| Shell | Bash |
| Python | 3.9 |
| Env-Manager | Conda (empfohlen) |
| RCWA-Binary | `../build/S4` (relativ zum Repository-Root) |
| GPU | Optional, für schnelleres Training empfohlen |

## 🚀 Einrichtung

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

## 🧪 End-to-End-Nutzung

### 1) RCWA-Daten mit S4 erzeugen

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

- Die Launcher rufen `../build/S4` mit `-t 32` auf und führen `NQ=1..4` parallel aus.
- `ms_final.sh` verwendet `metasurface_final.lua`.
- `ms_resume_allargs.sh` verwendet `metasurface_allargs_resume.lua`.

### 2) RCWA-Ausgaben mit Form-Vertices zusammenführen

```bash
python merge_s4_data_full.py --prefix myrun
# -> merged_s4_shapes_myrun.csv
```

### 3) Zusammengeführte Spaltennamen für das Training normalisieren (falls nötig)

`merge_s4_data_full.py` schreibt `folder_key` / `NQ`, während die Trainingspipeline `prefix` / `nQ` erwartet.

```bash
python -c "import pandas as pd; p='merged_s4_shapes_myrun.csv'; df=pd.read_csv(p); df=df.rename(columns={'folder_key':'prefix','NQ':'nQ'}); df.to_csv(p,index=False)"
```

### 4) CSV -> NPZ vorverarbeiten

```bash
mkdir -p merged_csvs
mv merged_s4_shapes_myrun.csv merged_csvs/

python three_stage_transmittance.py \
  --preprocess \
  --input_folder merged_csvs \
  --output_npz preprocessed_t_data.npz
```

### 5) Dreistufige Modelle trainieren

```bash
python three_stage_transmittance.py \
  --data_npz preprocessed_t_data.npz \
  --num_epochs 100 \
  --batch_size 1024
```

Wichtige Ausgabestruktur:

- `outputs_three_stage_YYYYMMDD_HHMMSS/stageA`
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageB`
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageC`

### 6) Checkpoints auswerten

```bash
python three_stage_transmittance_evaluation.py \
  --model_dir outputs_three_stage_YYYYMMDD_HHMMSS \
  --data_npz preprocessed_t_data.npz \
  --sample_count 8
```

### 7) Optional: neuronale Vorhersagen mit S4 vergleichen

```bash
python FilterShapeS4_Evaluator_Transmittance.py \
  --npz_file preprocessed_t_data.npz \
  --spec2shape_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageC/spec2shape_stageC.pt \
  --shape2spec_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageA/shape2spec_stageA.pt \
  --n_samples 4
```

## 🎛️ CLI-Referenz

### S4-Launcher-Flags (`ms_final.sh`, `ms_resume_allargs.sh`)

| Flag | Bedeutung | Standard |
|---|---|---|
| `-ns`, `--numshapes` | Anzahl der Formen | `100000` |
| `-r`, `--seed` | Zufalls-Seed | `88888` |
| `-p`, `--prefix` | Run-Präfix / Resume-Schlüssel | `""` |
| `-g`, `--numg` | Geometriebasis-Parameter | `80` |
| `-bo`, `--baseouter` | Basis-Outer-Offset | `0.25` |
| `-ro`, `--randouter` | Zufälliger Outer-Offset | `0.20` |

### Trainings-Flags (`three_stage_transmittance.py`)

| Flag | Zweck |
|---|---|
| `--preprocess` | Vorverarbeitungsmodus ausführen |
| `--input_folder` | Ordner mit zusammengeführten CSV-Dateien |
| `--output_npz` | Dateiname der vorverarbeiteten NPZ-Ausgabe |
| `--data_npz` | NPZ-Datensatz für das Training |
| `--csv_file` | Alternative CSV-Datenquelle |
| `--test` | Testmodus |
| `--num_epochs` | Trainings-Epochen |
| `--batch_size` | Batch-Größe |

### Auswertungs-Flags (`three_stage_transmittance_evaluation.py`)

| Flag | Zweck |
|---|---|
| `--model_dir` | Checkpoint-Root-Verzeichnis (erforderlich) |
| `--data_npz` / `--csv_file` | Datenquelle für die Auswertung |
| `--output_dir` | Ausgabeordner der Auswertung |
| `--sample_count` | Anzahl visualisierter Samples |
| `--seed` | Zufalls-Seed für die Stichprobenauswahl |
| `--font_scale` | Schrift-Skalierung für Plots |
| `--batch_size` | Batch-Größe für die Auswertung |
| `--plot_only` | Nur Trainingskurven-Plots neu erzeugen |

## 🧾 Datenvertrag (für die Vorverarbeitung)

Der Vorverarbeitungspfad in `three_stage_transmittance.py` erwartet zusammengeführte CSV-Dateien mit:

- ID-Spalten: `prefix`, `nQ`, `nS`, `shape_idx`, `c`
- Geometrietext: `vertices_str`
- Spektralspalten: `T@...`

Qualitätsprüfungen im Code:

- Gruppierung nach `shape_uid = prefix_nQ_nS_shape_idx`
- jede Gruppe muss genau 11 Zeilen enthalten
- nur Formen mit Q1-Punktanzahl in `[1, 4]` werden beibehalten

## 🛠️ Fehlerbehebung

- `../build/S4: No such file or directory`
  - S4 unter `../build/S4` bauen/verlinken oder die Launcher-Skripte auf den tatsächlichen S4-Pfad anpassen.
- `No matching CSVs found in 'results/'`
  - `--prefix` und das Ausgabe-Namensschema in `results/*_output_nQ*_nS*.csv` prüfen.
- `No transmission columns found`
  - Sicherstellen, dass die zusammengeführte CSV `T@...`-Spalten enthält.
- Vorverarbeitung liefert null Datensätze
  - Erforderliche Spalten prüfen und sicherstellen, dass jede Shape-UID 11 Kristallisationszeilen hat.
- GPU OOM während des Trainings
  - `--batch_size` reduzieren (zum Beispiel auf `256` oder `128`).
- Auswertung findet keine Checkpoints
  - Prüfen, ob unter `--model_dir` Folgendes existiert:
    - `stageA/shape2spec_stageA.pt`
    - `stageB/spec2shape_stageB.pt`
    - `stageC/spec2shape_stageC.pt`

## 📚 Zitation

Wenn dieses Repository zu Ihrer Forschung beiträgt, zitieren Sie bitte:

```bibtex
@article{chen2025inverse,
  title={Inverse Design of Metasurface for Spectral Imaging},
  author={Chen, Rongzhou and Nie, Haitao and Zhu, Shuo and Zhao, Yaping and Wang, Chutian and Lam, Edmund Y},
  journal={arXiv preprint arXiv:2510.21924},
  year={2025}
}
```

## 🌐 Sprachvarianten

Zusätzliche README-Varianten sind in diesem Repository verfügbar, darunter:

- `README.en.md`, `README.de.md`, `README.es.md`, `README.fr.md`
- `README.ru.md`, `README.ja.md`, `README.ko.md`, `README.vi.md`
- `README.ar.md`, `README.zh-CN.md`, `README.zh-TW.md`

## 📌 Hinweise

- Dies ist ein Forschungs-Workspace mit vielen archivierten und explorativen Skripten.
- Der kanonische Transmittance-Pfad ist auf Folgendes fokussiert:
  - `ms_final.sh` / `ms_resume_allargs.sh`
  - `merge_s4_data_full.py`
  - `three_stage_transmittance.py`
  - `three_stage_transmittance_evaluation.py`
- Es gibt derzeit keine explizite Lizenzdatei im Repository-Root.

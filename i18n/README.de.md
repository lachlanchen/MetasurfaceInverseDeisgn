[English](../README.md) · [العربية](README.ar.md) · [Español](README.es.md) · [Français](README.fr.md) · [日本語](README.ja.md) · [한국어](README.ko.md) · [Tiếng Việt](README.vi.md) · [中文 (简体)](README.zh-Hans.md) · [中文（繁體）](README.zh-Hant.md) · [Deutsch](README.de.md) · [Русский](README.ru.md)


# Inverses Design von Metaflächen für spektrale Bildgebung

<p align="center">
  <img alt="Status" src="https://img.shields.io/badge/Status-Research%20Prototype-f59e0b?style=for-the-badge">
  <img alt="Python" src="https://img.shields.io/badge/Python-3.9-3776AB?style=for-the-badge&logo=python&logoColor=white">
  <img alt="PyTorch" src="https://img.shields.io/badge/PyTorch-2.x-EE4C2C?style=for-the-badge&logo=pytorch&logoColor=white">
  <img alt="Simulator" src="https://img.shields.io/badge/Simulator-S4%20RCWA-16a34a?style=for-the-badge">
  <img alt="Platform" src="https://img.shields.io/badge/Platform-Linux%20%2B%20Bash-6b7280?style=for-the-badge&logo=gnu-bash&logoColor=white">
  <img alt="ArXiv" src="https://img.shields.io/badge/arXiv-2510.21924-b31b1b?style=for-the-badge&logo=arxiv&logoColor=white">
</p>

Ein script-zentriertes Forschungs-Repository (historisch als `inverse_metasurface` bezeichnet) für das inverse Design von Metaflächen in der spektralen Bildgebung.

Der Kern-Workflow kombiniert:
- physikbasierte RCWA-Simulation (`S4` + Lua)
- Datenzusammenführung und Anreicherung mit Forminformationen
- dreistufiges Lernen mit PyTorch (`shape -> spectrum`, `spectrum -> shape`, verkettetes Fine-Tuning)
- quantitative und qualitative Auswertung, optional mit Konsistenzprüfung neuronales Modell vs. S4

> [!IMPORTANT]
> Kanonisches Verhalten und Befehle aus den bestehenden Projekt-Skripten/-Docs bleiben erhalten. Wo historische Verweise auf fehlende Dateien zeigen, bleiben diese Verweise aus Kompatibilitätsgründen bewusst mit expliziten Hinweisen bestehen.

## 📑 Inhalt

- [✨ Auf einen Blick](#-auf-einen-blick)
- [🌍 Internationalisierung (i18n)](#-internationalisierung-i18n)
- [✨ Funktionen](#-funktionen)
- [🧭 End-to-End-Workflow](#-end-to-end-workflow)
- [🧱 Projektstruktur](#-projektstruktur)
- [🛠️ Voraussetzungen](#️-voraussetzungen)
- [🚀 Installation](#-installation)
- [▶️ Verwendung](#️-verwendung)
- [⚙️ Konfiguration](#️-konfiguration)
- [🧪 Beispiele](#-beispiele)
- [🔬 Forschungskontext](#-forschungskontext)
- [🧑‍💻 Entwicklungshinweise](#-entwicklungshinweise)
- [🧯 Fehlerbehebung](#-fehlerbehebung)
- [🗺️ Roadmap](#️-roadmap)
- [🤝 Mitwirken](#-mitwirken)
- [📄 Lizenz](#-lizenz)
- [📚 Zitation](#-zitation)

## ✨ Auf einen Blick

| Element | Details |
|---|---|
| 🎯 Hauptaufgabe | C4-symmetrische Metaflächen-Geometrie aus Ziel-Transmissionsspektren inferieren |
| 🔬 Simulator | `../build/S4`, aufgerufen durch Shell-Launcher und `.lua`-Skripte |
| 🧠 Lernpipeline | Stufe A `shape -> spectra`, Stufe B `spectra -> shape`, Stufe C `spectra -> shape -> spectra` |
| 📦 Datenvertrag | Zusammengeführte CSV (`T@...`, Metadaten, `vertices_str`) -> komprimierte NPZ (`uids`, `spectra`, `shapes`) |
| 🧪 Auswertung | MSE-Metriken, Stufen-Visualisierungen, optionale erneute S4-Simulation |
| 🌐 i18n-Status | Mehrsprachige README-Dateien im Repo-Root + vorhandenes Verzeichnis `i18n/` |

## 🌍 Internationalisierung (i18n)

- Mehrsprachige READMEs werden im Repository-Root als `README.<lang>.md` gepflegt.
- Das Verzeichnis `i18n/` ist in diesem Repository-Snapshot vorhanden.
- Diese Datei enthält oben genau eine Sprach-Navigationszeile, um doppelte Sprachleisten zu vermeiden.
- `README.en.md` existiert ebenfalls im Repo; diese `README.md` bleibt für diesen Update-Durchlauf die kanonische Basis.

## ✨ Funktionen

- End-to-End-Pfad für inverses Design von S4-Simulationsergebnissen bis zum trainierten inversen Modell.
- C4-symmetrische Polygon-Parametrisierung und Q1-Punktkodierung (`4x3`: `presence, x, y`).
- Dreistufiges Modelltraining in einem Skript (`three_stage_transmittance.py`).
- Merge-Tooling, das Spektralpräzision erhält und pro Form die Eckpunkte anhängt.
- Optionaler Evaluator, der gelernte Vorhersagen mit neuer S4-Simulation vergleicht.
- Umfangreiche explorative Zweige (AVIRIS, SWIR/Rauschen, GSST, archivierte/veraltete Varianten).

## 🧭 End-to-End-Workflow

1. Simulationsergebnisse in `results/` und Polygon-Dateien in `shapes/` erzeugen.
2. S4-CSV-Dateien zusammenführen und Form-Eckpunkte anhängen.
3. Spaltennamen der Merge-Ausgabe für Trainingskompatibilität normalisieren.
4. Zusammengeführte CSV-Dateien in NPZ-Tensoren vorverarbeiten.
5. Modelle der Stufen A/B/C trainieren.
6. Checkpoints auswerten und Verhalten visualisieren.
7. Optional Spektren vorhergesagter Formen mit neuen S4-Läufen vergleichen.

## 🧱 Projektstruktur

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

## 🛠️ Voraussetzungen

| Abhängigkeit | Hinweise |
|---|---|
| Linux + Bash | Launcher-Skripte sind auf Shell-Ausführung ausgelegt |
| Python 3.9 | Entspricht `iccp.yaml` (`python=3.9.18`) |
| Conda | Für Reproduzierbarkeit empfohlen |
| S4-Binary | Erwarteter Pfad: `../build/S4` |
| CUDA-GPU (optional) | Beschleunigt Training/Auswertung |

## 🚀 Installation

### 1) Klonen und wechseln

```bash
git clone <your-repo-url> inverse_metasurface
cd inverse_metasurface
```

### 2) Umgebung erstellen (empfohlen)

```bash
conda env create -f iccp.yaml
conda activate iccp
```

Alternativer Hinweis:

```bash
# Historical README reference (file may be absent in this snapshot)
pip install -r pip_requirements.txt
```

### 3) Von Skripten erwarteten Simulatorpfad prüfen

```bash
ls -l ../build/S4
```

### 4) (Optional) Launcher ausführbar machen

```bash
chmod +x ms.sh ms_final.sh ms_resume.sh ms_resume_allargs.sh ms_resume_random_state.sh ms_resume_random_state_nir.sh
```

## ▶️ Verwendung

### A) RCWA-Simulationsdaten erzeugen

Einfacher Launcher:

```bash
./ms.sh -ns 10000 -r 12345
```

Parametrisierter Launcher:

```bash
./ms_final.sh \
  -ns 10000 \
  -r 12345 \
  -p myrun \
  -g 80 \
  -bo 0.35 \
  -ro 0.30
```

Launcher für Resume-Szenarien:

```bash
./ms_resume_allargs.sh \
  -ns 10000 \
  -r 12345 \
  -p myrun \
  -g 80 \
  -bo 0.35 \
  -ro 0.30
```

Zusätzliches Resume/Random-State-Beispiel (aus den Befehlsdokumenten):

```bash
./ms_resume_random_state.sh \
  -p iccp100kG20Ov \
  -r 88888 \
  -g 20 \
  -bo 0.35 \
  -ro 0.3 \
  -ns 100000
```

Hinweise:
- Launcher führen `NQ=1..4` parallel aus.
- Skripte rufen `../build/S4` mit `-t 32` auf.

### B) S4-Ausgaben mergen und Form-Eckpunkte anhängen

```bash
python merge_s4_data_full.py --prefix myrun
# output: merged_s4_shapes_myrun.csv
```

### C) Spalten für Trainingskompatibilität normalisieren

`merge_s4_data_full.py` schreibt `folder_key` und `NQ`, während der Trainingspfad `prefix` und `nQ` erwartet.

```bash
python -c "import pandas as pd; p='merged_s4_shapes_myrun.csv'; df=pd.read_csv(p); df=df.rename(columns={'folder_key':'prefix','NQ':'nQ'}); df.to_csv(p,index=False)"
```

### D) CSV -> NPZ vorverarbeiten

```bash
mkdir -p merged_csvs
mv merged_s4_shapes_myrun.csv merged_csvs/

python three_stage_transmittance.py \
  --preprocess \
  --input_folder merged_csvs \
  --output_npz preprocessed_t_data.npz
```

### E) Stufen A/B/C trainieren

```bash
python three_stage_transmittance.py \
  --data_npz preprocessed_t_data.npz \
  --num_epochs 100 \
  --batch_size 1024
```

Die Ausgaben werden geschrieben nach:
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageA`
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageB`
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageC`

### F) Trainierte Modelle auswerten

Historischer README-Befehl (Skriptname zur Kompatibilität mit älteren Docs beibehalten):

```bash
python three_stage_transmittance_evaluation.py \
  --model_dir outputs_three_stage_YYYYMMDD_HHMMSS \
  --data_npz preprocessed_t_data.npz \
  --sample_count 8
```

Hinweis zum Repository-Status: `three_stage_transmittance_evaluation.py` ist in diesem Snapshot nicht vorhanden. Für verfügbare Auswertungsfunktionen `FilterShapeS4_Evaluator_Transmittance.py` verwenden.

### G) Optionaler Konsistenzcheck neuronales Modell vs. S4

```bash
python FilterShapeS4_Evaluator_Transmittance.py \
  --npz_file preprocessed_t_data.npz \
  --spec2shape_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageC/spec2shape_stageC.pt \
  --shape2spec_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageA/shape2spec_stageA.pt \
  --n_samples 4
```

## ⚙️ Konfiguration

### S4-Launcher (`ms_final.sh`, `ms_resume_allargs.sh`)

| Flag | Bedeutung | Standard |
|---|---|---|
| `-ns`, `--numshapes` | Anzahl der zu erzeugenden Formen | `100000` |
| `-r`, `--seed` | Zufalls-Seed | `88888` |
| `-p`, `--prefix` | Präfix/Resume-Schlüssel | `""` |
| `-g`, `--numg` | Basis-/Grid-Parameter | `80` |
| `-bo`, `--baseouter` | Basis-Offset der äußeren Grenze | `0.25` |
| `-ro`, `--randouter` | Zufälliger Offset der äußeren Grenze | `0.20` |

### Training (`three_stage_transmittance.py`)

| Flag | Bedeutung | Standard |
|---|---|---|
| `--preprocess` | Vorverarbeitungsmodus ausführen | `False` |
| `--input_folder` | Ordner mit zusammengeführten CSV-Dateien | `""` |
| `--output_npz` | Ausgabe-NPZ-Pfad | `preprocessed_data.npz` |
| `--data_npz` | NPZ-Datensatz fürs Training | `""` |
| `--csv_file` | CSV-Fallback, wenn NPZ nicht genutzt wird | `""` |
| `--test` | Testmodus | `False` |
| `--num_epochs` | Anzahl Trainings-Epochen | `10` |
| `--batch_size` | Batch-Größe | `4096` |

### Historische Auswertungskonfiguration (`three_stage_transmittance_evaluation.py`)

| Flag | Bedeutung | Standard |
|---|---|---|
| `--model_dir` | Verzeichnis mit `stageA/B/C` | erforderlich |
| `--data_npz` | NPZ-Eingabe | `""` |
| `--csv_file` | CSV-Eingabe-Fallback | `""` |
| `--output_dir` | Ausgabeverzeichnis-Override | automatisch unter `model_dir` |
| `--sample_count` | Anzahl visualisierter Samples | `4` |
| `--seed` | Zufalls-Seed | `23` |
| `--font_scale` | Schrift-Skalierung für Plots | `1.0` |
| `--batch_size` | Batch-Größe für Auswertung | `32` |
| `--plot_only` | Nur Trainingskurven plotten | `False` |

### S4-Konsistenz-Evaluator (`FilterShapeS4_Evaluator_Transmittance.py`)

| Flag | Bedeutung | Standard |
|---|---|---|
| `--npz_file` | Eingabe-NPZ-Datei | `preprocessed_t_data.npz` |
| `--spec2shape_ckpt` | Stage-C-Checkpoint-Pfad | `outputs_three_stage_20250322_145925/stageC/spec2shape_stageC.pt` |
| `--shape2spec_ckpt` | Stage-A-Checkpoint-Pfad | `outputs_three_stage_20250322_145925/stageA/shape2spec_stageA.pt` |
| `--n_samples` | Anzahl ausgewerteter Samples | `4` |
| `--seed` | Zufalls-Seed | `23` |
| `--max_workers` | S4-Worker-Threads | `4` |
| `--out_folder` | Ausgabeverzeichnis | automatischer Zeitstempel |

## 🧪 Beispiele

### Smoke-Run

```bash
./ms_final.sh -ns 1000 -r 42 -p smoke -g 40 -bo 0.25 -ro 0.20
python merge_s4_data_full.py --prefix smoke
python -c "import pandas as pd; p='merged_s4_shapes_smoke.csv'; d=pd.read_csv(p).rename(columns={'folder_key':'prefix','NQ':'nQ'}); d.to_csv(p,index=False)"
mkdir -p merged_csvs && mv merged_s4_shapes_smoke.csv merged_csvs/
python three_stage_transmittance.py --preprocess --input_folder merged_csvs --output_npz smoke.npz
python three_stage_transmittance.py --data_npz smoke.npz --num_epochs 5 --batch_size 128
```

### Run-Profil-Beispiele (aus `commands.md` / `commands_updated.md`)

```bash
# no overlap, G=40
./ms_resume_allargs.sh -ns 10000 -r 12345 -p fast_without_overlap -g 40 -bo 0.25 -ro 0.2

# overlap, G=80
./ms_resume_allargs.sh -ns 10000 -r 12345 -p overlap -g 80 -bo 0.35 -ro 0.3

# large run, overlap, G=80
./ms_resume_allargs.sh -ns 100000 -r 12345 -p more_basis_overlap -g 80 -bo 0.35 -ro 0.3
```

## 🔬 Forschungskontext

Das aktuelle Setup für inverses Design lernt, C4-symmetrische Geometrie aus der Transmission über Kristallisationszustände hinweg zu rekonstruieren. Die Transmissions-Pipeline setzt derzeit voraus:

- 11 Kristallisations-Zeilen pro Form-Sample (gruppiert nach eindeutiger `shape_uid`)
- 100 Wellenlängen-Bins pro Kristallisationszustand (`T@...`-Spalten)
- Bis zu 4 Q1-Kontrollpunkte codiert als `4x3`-Tensor: `(presence, x, y)`
- Polygon-Rekonstruktion unter C4-Symmetrie für Form-Visualisierung und Konsistenzprüfungen

Das Repository enthält außerdem explorative Zweige (`AVIRIS*`, `noise_experiment*`, `archived/`) außerhalb des primären Transmissions-Trainingspfads.

## 🧑‍💻 Entwicklungshinweise

- Dieses Repository ist skriptzentrierte Forschung und kein paketiertes Python-Modul.
- Kernskripte setzen relative Pfade voraus (insbesondere `../build/S4`, `results/`, `shapes/`).
- `.gitignore` schließt viele erzeugte Experiment-Artefakte aus (`*.csv`, `*.npz`, `*.pt`, Run-Ordner).
- Einige Dateien/Verzeichnisse aus historischen Docs fehlen aktuell in diesem Snapshot; diese Verweise werden zur Kompatibilität bewusst mit Hinweisen beibehalten.
- macOS-Sidecar-Dateien (`._*`) sind vorhanden und können nicht-funktionale Metadaten-Artefakte sein.

## 🧯 Fehlerbehebung

| Symptom | Wahrscheinliche Ursache | Lösung |
|---|---|---|
| `../build/S4: No such file or directory` | S4-Binary fehlt am erwarteten relativen Pfad | S4 unter `../build/S4` bauen/verlinken oder Launcher-Pfade anpassen |
| `No transmission columns found` | CSV ohne `T@...`-Spalten | Ausgabeformat des Merges prüfen |
| `Must specify either --data_npz or --csv_file` | Fehlendes Datenargument für Training/Auswertung | Eine Eingabe explizit angeben |
| `No valid shapes => SHIFT->Q1->UpTo4` | Ungültiges/leeres `vertices_str` oder Q1-Filter entfernt alle Samples | Form-Dateien und Merge-Ausgabe validieren |
| Leere Merge-Ausgabe für `--prefix` | Präfix passt nicht zu Dateien in `results/` | Exaktes Dateipräfix prüfen und Merge erneut ausführen |
| Evaluation-Checkpoint fehlt | Fehlende Checkpoint-Dateien für `stageA/B/C` | Prüfen, ob `--model_dir` auf vollständigen Output-Ordner zeigt |
| `three_stage_transmittance_evaluation.py` nicht gefunden | In historischen Docs referenziertes Skript fehlt aktuell | `FilterShapeS4_Evaluator_Transmittance.py` verwenden oder Skript aus früheren Commits wiederherstellen |

## 🗺️ Roadmap

- Reproduzierbarkeit mit explizitem Data-Versioning-Manifest und fixierten Run-Konfigurationen verbessern.
- Kanonische Entry-Points für Transmissions-, AVIRIS- und Rausch-Zweige konsolidieren.
- Automatisierte Smoke-Tests für Vorverarbeitung und eine Mini-Trainingsepoche ergänzen.
- Klareres Experiment-Register hinzufügen, das Ausgabeordner mit exakten Befehlszeilen verknüpft.
- Workflow zur Synchronisierung mehrsprachiger READMEs erweitern (Root-Sprachdateien und `i18n/`).

## 🤝 Mitwirken

Beiträge sind willkommen, besonders zu Reproduzierbarkeit, Tests und Dokumentationsqualität.

Empfohlener Ablauf:

1. Issue mit Umfang und erwartetem Verhalten eröffnen.
2. Fokussierten Branch erstellen.
3. Pull Request mit ausführbaren Befehlen und Outputs einreichen.
4. Änderungen nach Möglichkeit auf einen Workflow begrenzen.

## 📄 Lizenz

Im Repository-Root ist in diesem Snapshot derzeit keine `LICENSE`-Datei vorhanden. Ergänze eine, um Nutzungs- und Weitergabebedingungen festzulegen.

## 📚 Zitation

Wenn du dieses Repository verwendest oder auf dieser Arbeit aufbaust, zitiere bitte:

```bibtex
@article{chen2025inverse,
  title={Inverse Design of Metasurface for Spectral Imaging},
  author={Chen, Rongzhou and Nie, Haitao and Zhu, Shuo and Zhao, Yaping and Wang, Chutian and Lam, Edmund Y},
  journal={arXiv preprint arXiv:2510.21924},
  year={2025}
}
```

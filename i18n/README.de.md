[English](../README.md) · [العربية](README.ar.md) · [Español](README.es.md) · [Français](README.fr.md) · [日本語](README.ja.md) · [한국어](README.ko.md) · [Tiếng Việt](README.vi.md) · [中文 (简体)](README.zh-Hans.md) · [中文（繁體）](README.zh-Hant.md) · [Deutsch](README.de.md) · [Русский](README.ru.md)


[![LazyingArt banner](https://github.com/lachlanchen/lachlanchen/raw/main/figs/banner.png)](https://github.com/lachlanchen/lachlanchen/blob/main/figs/banner.png)

# Inverses Design von Metasurfaces für spektrale Bildgebung

<p align="center">
  <img alt="Status" src="https://img.shields.io/badge/Status-Research%20Prototype-f59e0b?style=for-the-badge">
  <img alt="Python" src="https://img.shields.io/badge/Python-3.9-3776AB?style=for-the-badge&logo=python&logoColor=white">
  <img alt="PyTorch" src="https://img.shields.io/badge/PyTorch-2.x-EE4C2C?style=for-the-badge&logo=pytorch&logoColor=white">
  <img alt="Simulator" src="https://img.shields.io/badge/Simulator-S4%20RCWA-16a34a?style=for-the-badge">
  <img alt="Platform" src="https://img.shields.io/badge/Platform-Linux%20%2B%20Bash-6b7280?style=for-the-badge&logo=gnu-bash&logoColor=white">
  <img alt="ArXiv" src="https://img.shields.io/badge/arXiv-2510.21924-b31b1b?style=for-the-badge&logo=arxiv&logoColor=white">
</p>

Ein skriptbasiertes Forschungs-Repository (historisch als `inverse_metasurface` bezeichnet) für inverses Design von Metasurfaces in der spektralen Bildgebung.

Der Kern-Workflow kombiniert:
- Physik-basierte RCWA-Simulation (`S4` + Lua)
- Datenzusammenführung und Formanreicherung
- Drei-stufiges PyTorch-Lernen (`shape -> spectrum`, `spectrum -> shape`, verkettetes Fine-Tuning)
- Quantitative und qualitative Auswertung, optional mit Konsistenzprüfung neural vs. S4

> [!IMPORTANT]
> Das kanonische Verhalten und die Befehle bleiben aus den bestehenden Projekt-Skripten/Docs erhalten. Wo historische Verweise auf nicht vorhandene Dateien zeigen, werden diese Verweise aus Kompatibilitätsgründen mit expliziten Hinweisen bewusst beibehalten.

## 📑 Inhalte

- [🌟 Snapshot](#-snapshot)
- [✨ Auf einen Blick](#-auf-einen-blick)
- [🌍 Internationalisierung (i18n)](#-internationalisierung-i18n)
- [✨ Funktionen](#-funktionen)
- [🧭 End-to-End-Workflow](#-end-to-end-workflow)
- [🧱 Projektstruktur](#-projektstruktur)
- [🛠️ Voraussetzungen](#️-voraussetzungen)
- [🚀 Installation](#-installation)
- [▶️ Nutzung](#️-nutzung)
- [⚙️ Konfiguration](#️-konfiguration)
- [🧪 Beispiele](#-beispiele)
- [🔬 Forschungskontext](#-forschungskontext)
- [🧑‍💻 Entwicklungshinweise](#-entwicklungshinweise)
- [🧯 Fehlerbehebung](#-fehlerbehebung)
- [🗺️ Roadmap](#️-roadmap)
- [🤝 Beitrag](#-beitrag)
- [📄 Lizenz](#-lizenz)
- [📚 Zitierung](#-zitierung)

## 🌟 Snapshot

| Fokus | Status |
|---|---|
| 🧠 Ziel | Inverse Rekonstruktion der C4-symmetrischen Metasurface-Geometrie aus Spektraldaten |
| 🔧 Kern-Stack | S4 RCWA (`Lua`) + PyTorch-Training + optionale Geometrie-zu-Spektrum-Neubewertung |
| 🧪 Datenpipeline | CSV-Merge/Form-Anreicherung → NPZ (`uids`, `spectra`, `shapes`) |
| 🚀 Reifegrad | Forschungsprototyp; Skripte und Dokumentation bleiben kompatibel mit historischen Verweisen |

## ✨ Auf einen Blick

| Punkt | Details |
|---|---|
| 🎯 Hauptaufgabe | Rekonstruktion der C4-symmetrischen Metasurface-Geometrie aus Ziel-Transmissionsspektren |
| 🔬 Simulator | `../build/S4`, aufgerufen durch Shell-Launcher und `.lua`-Skripte |
| 🧠 Lernpipeline | Stufe A `shape -> spectra`, Stufe B `spectra -> shape`, Stufe C `spectra -> shape -> spectra` |
| 📦 Datenvertrag | Zusammengeführte CSV (`T@...`, Metadaten, `vertices_str`) → komprimierte NPZ (`uids`, `spectra`, `shapes`) |
| 🧪 Bewertung | MSE-Metriken, Stufen-Visualisierungen, optionale frische S4-Neusimulation |
| 🌐 i18n-Status | Mehrsprachige README-Dateien im Repo-Root + bestehendes `i18n/`-Verzeichnis |

## 🌍 Internationalisierung (i18n)

- Die mehrsprachigen READMEs werden auf Repository-Ebene als Dateien `README.<lang>.md` gepflegt.
- Das Verzeichnis `i18n/` ist in diesem Repository-Snapshot vorhanden.
- Diese Datei enthält nur eine Sprachoptionszeile am Anfang, um doppelte Sprachleisten zu vermeiden.
- `README.en.md` ist ebenfalls im Repo vorhanden; diese `README.md` bleibt für diesen Aktualisierungslauf die kanonische Grundlage.

## ✨ Funktionen

- End-to-End-Inversdesign-Pfad von den S4-Simulationsausgaben bis zum trainierten inversen Modell.
- C4-symmetrische Polygon-Parametrisierung und Q1-Punktekodierung (`4x3`: `presence, x, y`).
- Drei-stufiges Modelltraining in einem Skript (`three_stage_transmittance.py`).
- Merge-Werkzeuge, die spektrale Präzision erhalten und Scheitelpunkte pro Form anhängen.
- Optionaler Evaluator, der gelernten Vorhersagen mit neuer S4-Simulation vergleicht.
- Umfangreiche Explorationszweige (AVIRIS, SWIR/Rauschen, GSST, archivierte/veraltete Varianten).

## 🧭 End-to-End-Workflow

1. Generiere Simulationsausgaben in `results/` und Polygon-Dateien in `shapes/`.
2. Führe S4-CSV-Dateien zusammen und hänge Form-Vertices an.
3. Normalisiere zusammengeführte Spaltennamen für Training-Kompatibilität.
4. Verarbeite zusammengeführte CSV-Dateien in NPZ-Tensoren vor.
5. Trainiere Modelle der Stufen A/B/C.
6. Bewerte Checkpoints und visualisiere das Verhalten.
7. Vergleiche optional die Spektren von vorhergesagten Formen mit neuen S4-Läufen.

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
| Linux + Bash | Starter-Skripte sind auf Shell-Ausführung ausgelegt |
| Python 3.9 | Entspricht `iccp.yaml` (`python=3.9.18`) |
| Conda | Empfohlen für Reproduzierbarkeit |
| S4-Binary | Erwarteter Pfad: `../build/S4` |
| CUDA-GPU (optional) | Beschleunigt Training und Auswertung |

## 🚀 Installation

### 1) Klonen und betreten

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

### 3) Simulatorpfad prüfen, der von den Skripten erwartet wird

```bash
ls -l ../build/S4
```

### 4) (Optional) Launcher ausführbar machen

```bash
chmod +x ms.sh ms_final.sh ms_resume.sh ms_resume_allargs.sh ms_resume_random_state.sh ms_resume_random_state_nir.sh
```

## ▶️ Nutzung

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

Resume-orientierter Launcher:

```bash
./ms_resume_allargs.sh \
  -ns 10000 \
  -r 12345 \
  -p myrun \
  -g 80 \
  -bo 0.35 \
  -ro 0.30
```

Zusätzliches Resume/Random-State-Beispiel (aus Befehlsdokumenten):

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

### B) S4-Ausgaben zusammenführen und Form-Vertices anhängen

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

### E) Stufe A/B/C trainieren

```bash
python three_stage_transmittance.py \
  --data_npz preprocessed_t_data.npz \
  --num_epochs 100 \
  --batch_size 1024
```

Ausgaben werden geschrieben nach:
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageA`
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageB`
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageC`

### F) Trainierte Modelle auswerten

Historischer README-Befehl (Skriptname zur Kompatibilität mit früheren Docs beibehalten):

```bash
python three_stage_transmittance_evaluation.py \
  --model_dir outputs_three_stage_YYYYMMDD_HHMMSS \
  --data_npz preprocessed_t_data.npz \
  --sample_count 8
```

Hinweis zum Repository-Stand: `three_stage_transmittance_evaluation.py` ist in diesem Snapshot nicht vorhanden. Verwende `FilterShapeS4_Evaluator_Transmittance.py` für die vorhandene Auswertungsfunktionalität.

### G) Optionaler Neural-versus-S4-Konsistenzcheck

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
| `-ns`, `--numshapes` | Anzahl zu generierender Formen | `100000` |
| `-r`, `--seed` | Zufalls-Seed | `88888` |
| `-p`, `--prefix` | Präfix/Resume-Schlüssel | `""` |
| `-g`, `--numg` | Basis-/Raster-Parameter | `80` |
| `-bo`, `--baseouter` | Offset der äußeren Basisgrenze | `0.25` |
| `-ro`, `--randouter` | Zufälliger Offset der äußeren Grenze | `0.20` |

### Training (`three_stage_transmittance.py`)

| Flag | Bedeutung | Standard |
|---|---|---|
| `--preprocess` | Vorverarbeitungsmodus ausführen | `False` |
| `--input_folder` | Ordner mit zusammengeführten CSV-Dateien | `""` |
| `--output_npz` | Ausgabe-NPZ-Pfad | `preprocessed_data.npz` |
| `--data_npz` | NPZ-Datensatz für Training | `""` |
| `--csv_file` | CSV-Fallback, falls NPZ nicht genutzt wird | `""` |
| `--test` | Testmodus | `False` |
| `--num_epochs` | Anzahl der Epochen | `10` |
| `--batch_size` | Batch-Größe | `4096` |

### Historische Auswertungskonfiguration (`three_stage_transmittance_evaluation.py`)

| Flag | Bedeutung | Standard |
|---|---|---|
| `--model_dir` | Verzeichnis mit `stageA/B/C` | erforderlich |
| `--data_npz` | NPZ-Eingabe | `""` |
| `--csv_file` | CSV-Eingabefallback | `""` |
| `--output_dir` | Ausgabeverzeichnis-Override | automatisch unter `model_dir` |
| `--sample_count` | Anzahl visualisierter Samples | `4` |
| `--seed` | Zufalls-Seed | `23` |
| `--font_scale` | Plot-Schriftgröße | `1.0` |
| `--batch_size` | Evaluations-Batchgröße | `32` |
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
| `--out_folder` | Ausgabeordner | automatischer Zeitstempel |

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

Der aktuelle Inversdesign-Workflow lernt, C4-symmetrische Geometrie aus der Transmittanz über Kristallisationszustände hinweg zurückzuerhalten. Die Transmittanz-Pipeline geht derzeit von:

- 11 Kristallisationszeilen pro Formsample (gruppiert nach eindeutiger `shape_uid`)
- 100 Wellenlängen-Bins pro Kristallisationszustand (`T@...`-Spalten)
- Bis zu 4 Q1-Kontrollpunkten, kodiert als `4x3`-Tensor: `(presence, x, y)`
- Polygon-Rekonstruktion unter C4-Symmetrie für Form-Visualisierung und Konsistenzprüfungen

Das Repository enthält außerdem explorative Zweige (`AVIRIS*`, `noise_experiment*`, `archived/`) über den primären Transmitanz-Trainingspfad hinaus.

## 🧑‍💻 Entwicklungshinweise

- Dies ist ein skriptzentriertes Forschungs-Repository statt eines gepackten Python-Moduls.
- Kernskripte erwarten relative Pfade (insbesondere `../build/S4`, `results/`, `shapes/`).
- `.gitignore` schließt viele generierte Experiment-Artefakte aus (`*.csv`, `*.npz`, `*.pt`, Run-Ordner).
- Einige Dateien/Verzeichnisse in historischen Docs sind in diesem Snapshot derzeit nicht vorhanden; diese Verweise werden aus Kompatibilitätsgründen absichtlich mit Hinweisen beibehalten.
- macOS-Sidecar-Dateien (`._*`) sind vorhanden und können Nicht-Funktions-Metadaten-Artefakte sein.

## 🧯 Fehlerbehebung

| Symptom | Wahrscheinliche Ursache | Lösung |
|---|---|---|
| `../build/S4: No such file or directory` | S4-Binary fehlt am erwarteten relativen Pfad | S4 unter `../build/S4` bauen/verlinken oder Launcher-Pfade aktualisieren |
| `No transmission columns found` | CSV ohne `T@...`-Spalten | Merge-Ausgabeformat erneut prüfen |
| `Must specify either --data_npz or --csv_file` | Fehlendes Trainings-/Evaluations-Datenargument | Explizit genau einen Eingabeparameter angeben |
| `No valid shapes => SHIFT->Q1->UpTo4` | Ungültiges/leerers `vertices_str` oder Q1-Filter entfernt alle Samples | Form-Dateien und Merge-Ausgabe validieren |
| Leere Merge-Ausgabe für `--prefix` | Präfix passt nicht zu Dateien in `results/` | Exaktes Dateipräfix prüfen und Merge erneut ausführen |
| Evaluations-Checkpoint fehlt | Fehlende `stageA/B/C`-Checkpoint-Dateien | Prüfen, dass `--model_dir` auf vollständigen Ausgabeordner zeigt |
| `three_stage_transmittance_evaluation.py` nicht gefunden | Skript, auf das historische Docs verweisen, ist derzeit nicht vorhanden | `FilterShapeS4_Evaluator_Transmittance.py` verwenden oder dieses Skript aus früheren Commits wiederherstellen |

## 🗺️ Roadmap

- Reproduzierbarkeit mit einem expliziten Datenversionierungs-Manifest und festen Run-Konfigurationen verbessern.
- Kanonische Einstiegspunkte für Transmitanz-, AVIRIS- und Rausch-Zweige konsolidieren.
- Automatisierte Smoke-Tests für Vorverarbeitung und eine Mini-Trainings-Epoche hinzufügen.
- Klareres Experiment-Register ergänzen, das Ausgabeverzeichnisse mit exakten Befehlszeilen verknüpft.
- Workflow zur Synchronisation mehrsprachiger READMEs erweitern (Root-Sprachdateien + `i18n/`).

## 🤝 Beitrag

Beiträge sind willkommen, besonders bei Reproduzierbarkeit, Tests und Dokumentationsqualität.

Empfohlener Ablauf:

1. Issue mit Umfang und erwartbarem Verhalten eröffnen.
2. Einen fokussierten Branch erstellen.
3. Einen Pull Request mit ausführbaren Befehlen und Outputs einreichen.
4. Änderungen nach Möglichkeit auf einen Workflow begrenzen.

## 📄 Lizenz

Eine `LICENSE`-Datei ist in diesem Repository-Snapshot derzeit im Wurzelverzeichnis nicht vorhanden. Bitte ergänze eine Datei, um Nutzungs- und Weitergabe-Bedingungen festzulegen.

## 📚 Zitierung

Wenn du dieses Repository verwendest oder auf dieser Arbeit aufbaust, bitte zitieren:

```bibtex
@article{chen2025inverse,
  title={Inverse Design of Metasurface for Spectral Imaging},
  author={Chen, Rongzhou and Nie, Haitao and Zhu, Shuo and Zhao, Yaping and Wang, Chutian and Lam, Edmund Y},
  journal={arXiv preprint arXiv:2510.21924},
  year={2025}
}
```


## ❤️ Support

| Donate | PayPal | Stripe |
| --- | --- | --- |
| [![Donate](https://camo.githubusercontent.com/24a4914f0b42c6f435f9e101621f1e52535b02c225764b2f6cc99416926004b7/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f446f6e6174652d4c617a79696e674172742d3045413545393f7374796c653d666f722d7468652d6261646765266c6f676f3d6b6f2d6669266c6f676f436f6c6f723d7768697465)](https://chat.lazying.art/donate) | [![PayPal](https://camo.githubusercontent.com/d0f57e8b016517a4b06961b24d0ca87d62fdba16e18bbdb6aba28e978dc0ea21/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f50617950616c2d526f6e677a686f754368656e2d3030343537433f7374796c653d666f722d7468652d6261646765266c6f676f3d70617970616c266c6f676f436f6c6f723d7768697465)](https://paypal.me/RongzhouChen) | [![Stripe](https://camo.githubusercontent.com/1152dfe04b6943afe3a8d2953676749603fb9f95e24088c92c97a01a897b4942/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f5374726970652d446f6e6174652d3633354246463f7374796c653d666f722d7468652d6261646765266c6f676f3d737472697065266c6f676f436f6c6f723d7768697465)](https://buy.stripe.com/aFadR8gIaflgfQV6T4fw400) |

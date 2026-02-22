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

Ein script-orientiertes Forschungs-Repository (historisch als `inverse_metasurface` bezeichnet) für das inverse Metasurface-Design in der Spektralbildgebung. Der zentrale Workflow kombiniert:

- physikgestützte RCWA-Simulation (S4 + Lua)
- Datenzusammenführung und Anfügen von Formen
- dreistufiges PyTorch-Lernen (`shape -> spectrum`, `spectrum -> shape`, verkettetes Fine-Tuning)
- quantitative und qualitative Auswertung, optional mit Konsistenzprüfung zwischen Neuronalem Netz und S4

## ✨ Auf einen Blick

| Punkt | Details |
|---|---|
| 🎯 Hauptaufgabe | C4-symmetrische Metasurface-Geometrie aus Ziel-Transmissionsspektren ableiten |
| 🔬 Simulator | `../build/S4`, aufgerufen durch Shell-Launcher und `.lua`-Skripte |
| 🧠 Lernpipeline | Stufe A `shape -> spectra`, Stufe B `spectra -> shape`, Stufe C `spectra -> shape -> spectra` |
| 📦 Datenvertrag | zusammengeführte CSV (`T@...`, Metadaten, `vertices_str`) -> komprimierte NPZ (`uids`, `spectra`, `shapes`) |
| 🧪 Auswertung | MSE-Metriken, Stufen-Visualisierungen, optionale erneute S4-Simulation |

## 🧭 End-to-End-Workflow

1. Simulationsergebnisse in `results/` und Polygondateien in `shapes/` erzeugen.
2. S4-CSV-Dateien zusammenführen und Form-Vertices anhängen.
3. Spaltennamen der zusammengeführten Datei für Trainingskompatibilität normalisieren.
4. Zusammengeführte CSV-Dateien in NPZ-Tensoren vorverarbeiten.
5. Modelle der Stufen A/B/C trainieren.
6. Checkpoints auswerten und Verhalten visualisieren.
7. Optional Spektren vorhergesagter Formen mit neuen S4-Läufen vergleichen.

## 🧱 Repository-Struktur

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

## 🛠️ Voraussetzungen

| Abhängigkeit | Hinweise |
|---|---|
| Linux + Bash | Launcher-Skripte sind auf Shell-Ausführung ausgelegt |
| Python 3.9 | Entspricht `iccp.yaml` |
| Conda | Für Reproduzierbarkeit empfohlen |
| S4-Binärdatei | Erwartet unter `../build/S4` |
| CUDA-GPU (optional) | Beschleunigt Training/Auswertung |

## 🚀 Einrichtung

### 1) Klonen und Verzeichnis betreten

```bash
git clone <your-repo-url> inverse_metasurface
cd inverse_metasurface
```

### 2) Umgebung erstellen (empfohlen)

```bash
conda env create -f iccp.yaml
conda activate iccp
```

Alternative (weniger kontrolliert):

```bash
pip install -r pip_requirements.txt
```

### 3) Von Skripten erwarteten Simulatorpfad prüfen

```bash
ls -l ../build/S4
```

### 4) (Optional) Launcher ausführbar machen

```bash
chmod +x ms.sh ms_final.sh ms_resume.sh ms_resume_allargs.sh ms_resume_random_state.sh
```

## ▶️ Praktische Nutzung

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

Launcher mit Resume-Fokus:

```bash
./ms_resume_allargs.sh \
  -ns 10000 \
  -r 12345 \
  -p myrun \
  -g 80 \
  -bo 0.35 \
  -ro 0.30
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

```bash
python three_stage_transmittance_evaluation.py \
  --model_dir outputs_three_stage_YYYYMMDD_HHMMSS \
  --data_npz preprocessed_t_data.npz \
  --sample_count 8
```

### G) Optionale Konsistenzprüfung Neuronalem Netz vs. S4

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
| `-ns`, `--numshapes` | Anzahl zu erzeugender Formen | `100000` |
| `-r`, `--seed` | Zufalls-Seed | `88888` |
| `-p`, `--prefix` | Prefix/Resume-Schlüssel | `""` |
| `-g`, `--numg` | Basis-/Grid-Parameter | `80` |
| `-bo`, `--baseouter` | Basis-Offset der äußeren Grenze | `0.25` |
| `-ro`, `--randouter` | Zufälliger Offset der äußeren Grenze | `0.20` |

### Training (`three_stage_transmittance.py`)

| Flag | Bedeutung | Standard |
|---|---|---|
| `--preprocess` | Vorverarbeitungsmodus ausführen | `False` |
| `--input_folder` | Ordner mit zusammengeführten CSV-Dateien | `""` |
| `--output_npz` | Ausgabepfad für NPZ | `preprocessed_data.npz` |
| `--data_npz` | NPZ-Datensatz für das Training | `""` |
| `--csv_file` | CSV-Fallback, falls NPZ nicht verwendet wird | `""` |
| `--test` | Testmodus | `False` |
| `--num_epochs` | Anzahl der Trainingsepochen | `10` |
| `--batch_size` | Batch-Größe | `4096` |

### Auswertung (`three_stage_transmittance_evaluation.py`)

| Flag | Bedeutung | Standard |
|---|---|---|
| `--model_dir` | Verzeichnis mit `stageA/B/C` | erforderlich |
| `--data_npz` | NPZ-Eingabe | `""` |
| `--csv_file` | CSV-Eingabe-Fallback | `""` |
| `--output_dir` | Ausgabeverzeichnis überschreiben | automatisch unter `model_dir` |
| `--sample_count` | Anzahl visualisierter Samples | `4` |
| `--seed` | Zufalls-Seed | `23` |
| `--font_scale` | Schrift-Skalierung für Plots | `1.0` |
| `--batch_size` | Batch-Größe für Auswertung | `32` |
| `--plot_only` | Nur Trainingskurven plotten | `False` |

### S4-Konsistenz-Evaluator (`FilterShapeS4_Evaluator_Transmittance.py`)

| Flag | Bedeutung | Standard |
|---|---|---|
| `--npz_file` | NPZ-Eingabedatei | `preprocessed_t_data.npz` |
| `--spec2shape_ckpt` | Checkpoint-Pfad für Stufe C | `outputs_three_stage_20250322_145925/stageC/spec2shape_stageC.pt` |
| `--shape2spec_ckpt` | Checkpoint-Pfad für Stufe A | `outputs_three_stage_20250322_145925/stageA/shape2spec_stageA.pt` |
| `--n_samples` | Anzahl ausgewerteter Samples | `4` |
| `--seed` | Zufalls-Seed | `23` |
| `--max_workers` | S4-Worker-Threads | `4` |
| `--out_folder` | Ausgabeverzeichnis | automatischer Zeitstempel |

## 🧪 Smoke-Run

```bash
./ms_final.sh -ns 1000 -r 42 -p smoke -g 40 -bo 0.25 -ro 0.20
python merge_s4_data_full.py --prefix smoke
python -c "import pandas as pd; p='merged_s4_shapes_smoke.csv'; d=pd.read_csv(p).rename(columns={'folder_key':'prefix','NQ':'nQ'}); d.to_csv(p,index=False)"
mkdir -p merged_csvs && mv merged_s4_shapes_smoke.csv merged_csvs/
python three_stage_transmittance.py --preprocess --input_folder merged_csvs --output_npz smoke.npz
python three_stage_transmittance.py --data_npz smoke.npz --num_epochs 5 --batch_size 128
```

## 🔬 Forschungskontext

Das aktuelle Inverse-Design-Setup lernt, C4-symmetrische Geometrie aus der Transmission über Kristallisationszustände hinweg zu rekonstruieren. Die aktuelle Transmissionspipeline nimmt an:

- 11 Kristallisationszeilen pro Form-Sample (gruppiert über eindeutige `shape_uid`)
- 100 Wellenlängen-Bins pro Kristallisationszustand (`T@...`-Spalten)
- bis zu 4 Q1-Kontrollpunkte kodiert als `4x3`-Tensor: `(presence, x, y)`
- Polygonrekonstruktion unter C4-Symmetrie für Formvisualisierung und Konsistenzprüfungen

Das Repository enthält außerdem explorative Zweige (`AVIRIS*`, `noise_experiment*`, `archived/`) außerhalb des primären Trainingspfads für Transmission.

## 🧯 Fehlerbehebung

| Symptom | Wahrscheinliche Ursache | Lösung |
|---|---|---|
| `../build/S4: No such file or directory` | S4-Binärdatei fehlt am erwarteten relativen Pfad | S4 unter `../build/S4` bauen oder verlinken, oder Launcher-Pfade aktualisieren |
| `No transmission columns found` | CSV enthält keine `T@...`-Spalten | Ausgabeformat des Merge-Schritts prüfen |
| `Must specify either --data_npz or --csv_file` | Trainings-/Auswertungsdatenargument fehlt | Eine Eingabe explizit angeben |
| `No valid shapes => SHIFT->Q1->UpTo4` | Ungültiges/leeres `vertices_str` oder Q1-Filter entfernt alle Samples | Shape-Dateien und Merge-Ausgabe validieren |
| Leere Merge-Ausgabe für `--prefix` | Prefix passt nicht zu Dateien in `results/` | Exakten Dateinamens-Prefix prüfen und Merge erneut ausführen |
| Auswertungs-Checkpoint fehlt | Fehlende Checkpoint-Dateien in `stageA/B/C` | Prüfen, ob `--model_dir` auf einen vollständigen Output-Ordner zeigt |

## 🤝 Beitrag

Beiträge sind willkommen, insbesondere für Reproduzierbarkeit, Tests und Dokumentationsqualität.

Empfohlener Ablauf:

1. Ein Issue mit Umfang und erwartetem Verhalten öffnen.
2. Einen fokussierten Branch erstellen.
3. Einen Pull Request mit ausführbaren Befehlen und Ausgaben einreichen.
4. Änderungen nach Möglichkeit auf einen Workflow beschränken.

## 📄 Lizenz

Im aktuellen Snapshot liegt im Repository-Root keine `LICENSE`-Datei vor. Füge eine hinzu, um Nutzungs- und Weitergabebedingungen festzulegen.

## 📚 Zitierung

Wenn du dieses Repository nutzt oder auf dieser Arbeit aufbaust, zitiere bitte:

```bibtex
@article{chen2025inverse,
  title={Inverse Design of Metasurface for Spectral Imaging},
  author={Chen, Rongzhou and Nie, Haitao and Zhu, Shuo and Zhao, Yaping and Wang, Chutian and Lam, Edmund Y},
  journal={arXiv preprint arXiv:2510.21924},
  year={2025}
}
```

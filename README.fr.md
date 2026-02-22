English | 中文繁體 | 中文简体 | 日本語 | 한국어 | Tiếng Việt | العربية | Français | Español | Deutsch | Русский

# inverse_metasurface

<div align="center">

[![Status](https://img.shields.io/badge/status-experimental-orange)](#-feuille-de-route)
[![Python](https://img.shields.io/badge/python-3.9-blue)](#-prérequis)
[![Platform](https://img.shields.io/badge/platform-linux-lightgrey)](#-prérequis)
[![S4](https://img.shields.io/badge/S4-required-success)](#-prérequis)
[![License](https://img.shields.io/badge/license-unset-lightgrey)](#-licence)

</div>

Un espace de travail de recherche pour la **conception inverse de métasurfaces** utilisant la **simulation S4/RCWA** et des **modèles neuronaux multi-étapes**.

Fonctionnalités prises en charge:
- Génération de données S4 à haut débit pour des métasurfaces polygonales à symétrie C4.
- Fusion des données + prétraitement vers un NPZ prêt pour l'entraînement.
- Apprentissage en trois étapes: **forme -> spectre**, **spectre -> forme**, **affinage en chaîne spectre -> forme -> spectre**.
- Évaluation et comparaison directe optionnelle **réseau neuronal vs S4**.

## 🌟 Vue d'ensemble

| Domaine | Ce que fournit ce dépôt |
|---|---|
| Simulation physique | Scripts Bash + Lua pour exécuter S4 en parallèle sur `nQ=1..4` |
| Outillage dataset | Scripts de fusion qui attachent les sommets de polygones et les spectres pivot |
| Pipeline ML | `three_stage_transmittance.py` avec modes prétraitement + entraînement |
| Évaluation | `three_stage_transmittance_evaluation.py` avec métriques + figures |
| Branche recherche | Expériences AVIRIS/hyperspectral et bruit/compression |

## 🧠 Contexte de recherche

Ce dépôt cible la conception inverse de métasurfaces photoniques: inférer la géométrie à partir de spectres souhaités (et inversement).

Hypothèses de base utilisées par le pipeline de référence:
- Symétrie C4 via une paramétrisation par point Q1.
- 11 états de cristallisation (`c` dans `[0.0, 1.0]`).
- Le spectre de chaque échantillon est stocké sous forme de lignes de transmission `11 x 100`.
- Les formes sont représentées par jusqu'à 4 points Q1 (tenseur `4 x 3`: présence, x, y).

Logique d'entraînement en trois étapes:
1. **Étape A**: forme -> spectre (`shape2spec_stageA.pt`)
2. **Étape B**: spectre -> forme (`spec2shape_stageB.pt`)
3. **Étape C**: affinage en chaîne spectre -> forme -> spectre (`spec2shape_stageC.pt`)

## 🗂 Structure du projet

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

## ✅ Prérequis

| Exigence | Notes |
|---|---|
| OS | Linux (les scripts supposent Bash + chemins Linux) |
| Python | 3.9 (depuis `iccp.yaml`) |
| Gestionnaire d'environnement | Conda recommandé |
| Binaire S4 | Attendu à `../build/S4` par rapport à la racine du dépôt |
| GPU (optionnel) | CUDA accélère l'entraînement/l'évaluation |

## ⚙️ Installation

### 1) Cloner et entrer dans le dossier

```bash
git clone <your-repo-url> inverse_metasurface
cd inverse_metasurface
```

### 2) Créer l'environnement

```bash
conda env create -f iccp.yaml
conda activate iccp
```

### 3) Vérifier le chemin S4

```bash
ls -l ../build/S4
```

Si ce chemin n'existe pas, construisez/placez S4 à cet emplacement ou ajustez les chemins dans les scripts.

## 🚀 Démarrage rapide (de bout en bout)

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

## 🧪 Détails d'utilisation

### A) Génération S4

Exécution minimale avec seed:

```bash
./ms.sh -ns 10000 -r 88888
```

Exécution paramétrée:

```bash
./ms_final.sh \
  -ns 100000 \
  -r 88888 \
  -p iccpOv100kG80 \
  -g 80 \
  -bo 0.35 \
  -ro 0.30
```

Exécution de reprise:

```bash
./ms_resume_allargs.sh \
  -ns 100000 \
  -r 88888 \
  -p iccpOv100kG80 \
  -g 80 \
  -bo 0.35 \
  -ro 0.30
```

### B) Fusion des CSV S4 bruts

```bash
python merge.py --prefix 20250123_155420
```

Utilitaire alternatif:

```bash
python merge_s4_data_full.py --prefix 20250123_155420
```

### C) Prétraitement des CSV fusionnés -> NPZ

```bash
python three_stage_transmittance.py --preprocess \
  --input_folder merged_csvs \
  --output_npz preprocessed_t_data.npz
```

### D) Entraînement

```bash
python three_stage_transmittance.py \
  --data_npz preprocessed_t_data.npz \
  --num_epochs 100 \
  --batch_size 1024
```

Les sorties d'entraînement sont stockées dans:
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageA`
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageB`
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageC`

### E) Évaluation

```bash
python three_stage_transmittance_evaluation.py \
  --model_dir outputs_three_stage_YYYYMMDD_HHMMSS \
  --data_npz preprocessed_t_data.npz \
  --sample_count 8
```

Génère:
- `evaluation_metrics.csv`
- `metrics_summary.csv`
- visualisations par étape (`.png`, `.pdf`)
- courbes d'entraînement

### F) Comparaison réseau neuronal vs S4 direct (optionnel)

```bash
python FilterShapeS4_Evaluator_Transmittance.py \
  --npz_file preprocessed_t_data.npz \
  --spec2shape_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageC/spec2shape_stageC.pt \
  --shape2spec_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageA/shape2spec_stageA.pt \
  --n_samples 4
```

## 🔧 Options CLI clés

### Flags du lanceur S4 (`ms_final.sh`, `ms_resume_allargs.sh`)

| Flag | Signification | Valeur par défaut |
|---|---|---|
| `-ns`, `--numshapes` | Nombre de formes | `100000` |
| `-r`, `--seed` | Graine aléatoire | `88888` |
| `-p`, `--prefix` | Préfixe d'exécution / clé de reprise | empty |
| `-g`, `--numg` | Paramètre géométrie/base | `80` |
| `-bo`, `--baseouter` | Décalage externe de base | `0.25` |
| `-ro`, `--randouter` | Décalage externe aléatoire | `0.20` |

### Flags d'entraînement (`three_stage_transmittance.py`)

| Flag | Signification | Valeur par défaut |
|---|---|---|
| `--preprocess` | Exécuter le mode prétraitement | off |
| `--input_folder` | Dossier CSV d'entrée | empty |
| `--output_npz` | Chemin NPZ de sortie | `preprocessed_data.npz` |
| `--data_npz` | NPZ utilisé pour entraînement/évaluation | empty |
| `--csv_file` | CSV utilisé directement | empty |
| `--num_epochs` | Epochs par étape | `10` |
| `--batch_size` | Taille de batch | `4096` |
| `--test` | Espace réservé mode test | off |

### Flags d'évaluation (`three_stage_transmittance_evaluation.py`)

| Flag | Signification | Valeur par défaut |
|---|---|---|
| `--model_dir` | Dossier racine de l'exécution entraînée | required |
| `--data_npz` | NPZ d'évaluation | empty |
| `--csv_file` | CSV d'évaluation | empty |
| `--output_dir` | Dossier de sortie | `model_dir/evaluation_<timestamp>` |
| `--sample_count` | Nombre d'échantillons visualisés | `4` |
| `--seed` | Graine d'échantillonnage | `23` |
| `--font_scale` | Échelle de police des graphiques | `1.0` |
| `--batch_size` | Taille de batch en évaluation | `32` |
| `--plot_only` | Tracer uniquement les courbes | off |

## 🧭 Dépannage

| Symptôme | Cause probable | Correctif |
|---|---|---|
| `../build/S4: No such file or directory` | Chemin du binaire S4 incorrect | Construire/placer S4 à `../build/S4` ou corriger les scripts |
| `Must specify either --data_npz or --csv_file` | Source de dataset manquante | Passer explicitement l'un de ces flags |
| `No matching CSVs found in 'results/'` | Incohérence de préfixe | Vérifier le préfixe et le schéma de nommage des sorties |
| Très peu/zéro enregistrements prétraités | Lignes `T@...` ou `vertices_str` manquantes/invalides | Valider le schéma CSV fusionné et le regroupement des lignes par forme |
| CUDA OOM | Batch trop grand | Réduire `--batch_size` (ex. `1024 -> 256`) |

## 🧱 Notes de développement

- Ce dépôt est fortement orienté expérimentation; de nombreux artefacts générés ne sont volontairement pas suivis.
- Plusieurs variantes de scripts existent pour des tâches similaires (`merge_*`, `aviris_*`, `noise_experiment_*`).
- Le workflow de référence pour la conception inverse de métasurfaces est celui documenté dans ce README.
- Les scripts d'entraînement fixent des seeds (`42`), mais un déterminisme strict dépend encore du matériel/comportement backend.
- Il n'existe pas encore de CI unifiée + suite de tests automatisés complète.

## 🛣 Feuille de route

- Ajouter un dataset canonique compact pour des smoke tests rapides.
- Consolider les scripts de pipeline en doublon.
- Ajouter des vérifications d'intégrité fusion/prétraitement et de chargeabilité des checkpoints.
- Ajouter une CI pour lint + smoke train/eval.
- Documenter plus explicitement le build/version pinning de S4.

## 🤝 Contribution

1. Créez une branche de fonctionnalité.
2. Gardez la portée de la PR resserrée (une préoccupation pipeline/expérience par PR).
3. Incluez des commandes exécutables exactes et les chemins de sortie attendus.
4. Évitez de commiter de gros artefacts générés sauf nécessité.
5. Ajoutez des notes de reproductibilité (seed, source de données, chemin checkpoint).

## 📄 Licence

Aucun fichier `LICENSE` n'est actuellement présent dans ce dépôt.

Tant qu'un fichier de licence n'est pas ajouté, les conditions de réutilisation et de redistribution doivent être considérées comme **indéterminées**.

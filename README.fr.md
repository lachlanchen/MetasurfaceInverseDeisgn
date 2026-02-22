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

Un dépôt de recherche orienté scripts (historiquement référencé comme `inverse_metasurface`) pour la **conception inverse de métasurfaces à symétrie C4** en imagerie spectrale, couvrant :

- la génération de données RCWA avec S4 (`.lua` + lanceurs shell)
- la fusion et le prétraitement des données (`.csv` -> `.npz`)
- un entraînement neuronal en trois étapes (forme->spectres, spectres->forme, affinement de la chaîne)
- l'évaluation et, en option, des vérifications réseau-vs-S4

## ✨ Vue d'ensemble

| Élément | Détails |
|---|---|
| Objectif principal | Prédire la géométrie à partir d'un spectre de transmission cible |
| Forme principale du jeu de données | spectres : `11 x 100`, forme : `4 x 3` |
| Script d'entraînement principal | `three_stage_transmittance.py` |
| Script d'évaluation principal | `three_stage_transmittance_evaluation.py` |
| Lanceur RCWA | `ms_final.sh`, `ms_resume_allargs.sh` |
| Script de fusion | `merge_s4_data_full.py` |

## 🧠 Contexte de recherche

Ce projet se concentre sur la conception inverse de métasurfaces pour l'imagerie spectrale. Le pipeline d'entraînement utilise des spectres de transmission générés par S4 à travers différents états de cristallisation et apprend à la fois les mappings direct et inverse :

1. **Étape A** : forme -> spectres
2. **Étape B** : spectres -> forme
3. **Étape C** : spectres -> forme -> spectres (affinement avec perte en chaîne)

Le code actuel de prétraitement/entraînement suppose :

- 11 états de cristallisation (`c = 0.0 ... 1.0`)
- 100 bins de longueur d'onde par état
- une représentation de forme sous forme d'au plus 4 points Q1 avec `[presence, x, y]`

## 🗂️ Structure du dépôt (chemins principaux)

```text
.
├── ms.sh / ms_final.sh / ms_resume_allargs.sh
├── metasurface_final.lua / metasurface_allargs_resume.lua / metasurface_seed.lua
├── merge_s4_data_full.py
├── three_stage_transmittance.py
├── three_stage_transmittance_evaluation.py
├── FilterShapeS4_Evaluator_Transmittance.py
├── results/                # sorties CSV S4 brutes
├── shapes/                 # sommets de polygones générés
├── merged_csvs/            # CSV fusionnés utilisés pour le prétraitement
├── outputs_three_stage_*/  # checkpoints, pertes, visualisations
├── partial_crys_data/
└── iccp.yaml
```

## ⚙️ Prérequis

| Dépendance | Exigence |
|---|---|
| OS | Linux |
| Shell | Bash |
| Python | 3.9 |
| Gestionnaire d'environnement | Conda (recommandé) |
| Binaire RCWA | `../build/S4` (relatif à la racine du dépôt) |
| GPU | Optionnel, recommandé pour un entraînement plus rapide |

## 🚀 Installation

```bash
git clone <repo-url> inverse_metasurface
cd inverse_metasurface

conda env create -f iccp.yaml
conda activate iccp

# Required by launcher scripts
ls -l ../build/S4
```

Optionnel :

```bash
chmod +x ms.sh ms_final.sh ms_resume_allargs.sh
```

## 🧪 Utilisation de bout en bout

### 1) Générer des données RCWA avec S4

```bash
./ms_final.sh \
  -ns 10000 \
  -r 12345 \
  -p myrun \
  -g 80 \
  -bo 0.35 \
  -ro 0.30
```

Remarques :

- Les lanceurs appellent `../build/S4` avec `-t 32` et exécutent `NQ=1..4` en parallèle.
- `ms_final.sh` utilise `metasurface_final.lua`.
- `ms_resume_allargs.sh` utilise `metasurface_allargs_resume.lua`.

### 2) Fusionner les sorties RCWA avec les sommets des formes

```bash
python merge_s4_data_full.py --prefix myrun
# -> merged_s4_shapes_myrun.csv
```

### 3) Normaliser les noms de colonnes fusionnées pour l'entraînement (si nécessaire)

`merge_s4_data_full.py` écrit `folder_key` / `NQ`, tandis que le pipeline d'entraînement attend `prefix` / `nQ`.

```bash
python -c "import pandas as pd; p='merged_s4_shapes_myrun.csv'; df=pd.read_csv(p); df=df.rename(columns={'folder_key':'prefix','NQ':'nQ'}); df.to_csv(p,index=False)"
```

### 4) Prétraiter CSV -> NPZ

```bash
mkdir -p merged_csvs
mv merged_s4_shapes_myrun.csv merged_csvs/

python three_stage_transmittance.py \
  --preprocess \
  --input_folder merged_csvs \
  --output_npz preprocessed_t_data.npz
```

### 5) Entraîner les modèles en trois étapes

```bash
python three_stage_transmittance.py \
  --data_npz preprocessed_t_data.npz \
  --num_epochs 100 \
  --batch_size 1024
```

Structure de sortie principale :

- `outputs_three_stage_YYYYMMDD_HHMMSS/stageA`
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageB`
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageC`

### 6) Évaluer les checkpoints

```bash
python three_stage_transmittance_evaluation.py \
  --model_dir outputs_three_stage_YYYYMMDD_HHMMSS \
  --data_npz preprocessed_t_data.npz \
  --sample_count 8
```

### 7) Optionnel : comparer les prédictions du réseau avec S4

```bash
python FilterShapeS4_Evaluator_Transmittance.py \
  --npz_file preprocessed_t_data.npz \
  --spec2shape_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageC/spec2shape_stageC.pt \
  --shape2spec_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageA/shape2spec_stageA.pt \
  --n_samples 4
```

## 🎛️ Référence CLI

### Options des lanceurs S4 (`ms_final.sh`, `ms_resume_allargs.sh`)

| Option | Signification | Par défaut |
|---|---|---|
| `-ns`, `--numshapes` | Nombre de formes | `100000` |
| `-r`, `--seed` | Graine aléatoire | `88888` |
| `-p`, `--prefix` | Préfixe d'exécution / clé de reprise | `""` |
| `-g`, `--numg` | Paramètre de base géométrique | `80` |
| `-bo`, `--baseouter` | Décalage externe de base | `0.25` |
| `-ro`, `--randouter` | Décalage externe aléatoire | `0.20` |

### Options d'entraînement (`three_stage_transmittance.py`)

| Option | Objectif |
|---|---|
| `--preprocess` | Exécuter le mode prétraitement |
| `--input_folder` | Dossier des fichiers CSV fusionnés |
| `--output_npz` | Nom du fichier NPZ prétraité en sortie |
| `--data_npz` | Jeu de données NPZ pour l'entraînement |
| `--csv_file` | Alternative : jeu de données CSV |
| `--test` | Mode test |
| `--num_epochs` | Nombre d'époques d'entraînement |
| `--batch_size` | Taille de lot |

### Options d'évaluation (`three_stage_transmittance_evaluation.py`)

| Option | Objectif |
|---|---|
| `--model_dir` | Répertoire racine des checkpoints (requis) |
| `--data_npz` / `--csv_file` | Source des données d'évaluation |
| `--output_dir` | Dossier de sortie de l'évaluation |
| `--sample_count` | Nombre d'échantillons visualisés |
| `--seed` | Graine aléatoire pour la sélection d'échantillons |
| `--font_scale` | Mise à l'échelle de la police des graphiques |
| `--batch_size` | Taille de lot pour l'évaluation |
| `--plot_only` | Régénérer uniquement les courbes d'entraînement |

## 🧾 Contrat de données (pour le prétraitement)

Le chemin de prétraitement dans `three_stage_transmittance.py` attend des CSV fusionnés contenant :

- colonnes d'identifiant : `prefix`, `nQ`, `nS`, `shape_idx`, `c`
- texte de géométrie : `vertices_str`
- colonnes spectrales : `T@...`

Contrôles qualité appliqués par le code :

- groupement par `shape_uid = prefix_nQ_nS_shape_idx`
- chaque groupe doit contenir exactement 11 lignes
- seules les formes avec un nombre de points Q1 dans `[1, 4]` sont conservées

## 🛠️ Dépannage

- `../build/S4: No such file or directory`
  - Compilez/liennez S4 à `../build/S4` ou modifiez les scripts de lancement vers votre chemin S4 réel.
- `No matching CSVs found in 'results/'`
  - Vérifiez `--prefix` et la convention de nommage dans `results/*_output_nQ*_nS*.csv`.
- `No transmission columns found`
  - Vérifiez que le CSV fusionné contient des colonnes `T@...`.
- Le prétraitement renvoie zéro enregistrement
  - Vérifiez les colonnes requises et que chaque UID de forme a 11 lignes de cristallisation.
- OOM GPU pendant l'entraînement
  - Réduisez `--batch_size` (par exemple `256` ou `128`).
- L'évaluation ne trouve pas les checkpoints
  - Vérifiez la présence des fichiers suivants sous `--model_dir` :
    - `stageA/shape2spec_stageA.pt`
    - `stageB/spec2shape_stageB.pt`
    - `stageC/spec2shape_stageC.pt`

## 📚 Citation

Si ce dépôt contribue à vos travaux de recherche, veuillez citer :

```bibtex
@article{chen2025inverse,
  title={Inverse Design of Metasurface for Spectral Imaging},
  author={Chen, Rongzhou and Nie, Haitao and Zhu, Shuo and Zhao, Yaping and Wang, Chutian and Lam, Edmund Y},
  journal={arXiv preprint arXiv:2510.21924},
  year={2025}
}
```

## 🌐 Variantes de langue

D'autres variantes de README sont disponibles dans ce dépôt, notamment :

- `README.en.md`, `README.de.md`, `README.es.md`, `README.fr.md`
- `README.ru.md`, `README.ja.md`, `README.ko.md`, `README.vi.md`
- `README.ar.md`, `README.zh-CN.md`, `README.zh-TW.md`

## 📌 Notes

- Il s'agit d'un espace de recherche avec de nombreux scripts archivés et exploratoires.
- Le chemin canonique pour la transmission est centré sur :
  - `ms_final.sh` / `ms_resume_allargs.sh`
  - `merge_s4_data_full.py`
  - `three_stage_transmittance.py`
  - `three_stage_transmittance_evaluation.py`
- Aucun fichier de licence explicite n'est actuellement défini à la racine du dépôt.

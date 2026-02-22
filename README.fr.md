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

[![Status](https://img.shields.io/badge/Status-Research%20Prototype-orange)](#-project-scope)
[![Python](https://img.shields.io/badge/Python-3.9-3776AB?logo=python&logoColor=white)](#-environment-setup)
[![Platform](https://img.shields.io/badge/Platform-Linux-2f2f2f?logo=linux&logoColor=white)](#-prerequisites)
[![RCWA](https://img.shields.io/badge/RCWA-S4%20Required-1f9d55)](#-prerequisites)
[![PyTorch](https://img.shields.io/badge/Framework-PyTorch-EE4C2C?logo=pytorch&logoColor=white)](#-training-and-evaluation)
[![License](https://img.shields.io/badge/License-Not%20Specified-lightgrey)](#-license)

Une base de code de recherche orientée scripts pour la **conception inverse de métasurfaces** sous symétrie C4.
Elle combine :

- 🔬 Simulation S4/RCWA (`.lua` + lanceurs Bash)
- 🧱 Fusion et prétraitement des données à partir de spectres bruts + sommets de formes
- 🧠 Entraînement PyTorch en trois étapes (`shape -> spectra`, `spectra -> shape`, réglage de la chaîne)
- 📊 Évaluation et visualisation, y compris des vérifications de cohérence réseau neuronal vs S4

## 📌 Table des matières

- [Périmètre du projet](#-project-scope)
- [Contexte de recherche](#-research-context)
- [Structure du dépôt](#-repository-layout)
- [Prérequis](#-prerequisites)
- [Configuration de l’environnement](#-environment-setup)
- [Démarrage rapide](#-quick-start)
- [Pipeline de bout en bout](#-end-to-end-pipeline)
- [Entraînement et évaluation](#-training-and-evaluation)
- [Principales options CLI](#-key-cli-options)
- [Dépannage](#-troubleshooting)
- [Feuille de route](#-roadmap)
- [Citation](#-citation)
- [Licence](#-license)

## 🎯 Périmètre du projet

Ce dépôt est fortement orienté expérimentation et centré sur des scripts exécutables (et non sur une bibliothèque packagée).
Le workflow le plus stable est :

1. Exécuter S4 pour générer les sorties optiques brutes dans `results/` et les formes dans `shapes/`
2. Fusionner et pivoter les fichiers CSV par exécution en données tabulaires prêtes pour l’entraînement
3. Convertir les CSV fusionnés en `.npz` compressé
4. Entraîner un pipeline de transmittance en 3 étapes
5. Évaluer les checkpoints et exporter les graphes/métriques

## 🧪 Contexte de recherche

### Cadre du problème

La tâche centrale de conception inverse consiste à reconstruire la géométrie de métasurface à partir de spectres de transmission cibles (et inversement), avec contraintes de symétrie C4 et balayages de cristallisation partielle.

### Hypothèses de données dans le pipeline de transmittance

| Élément | Valeur |
|---|---|
| États de cristallisation par forme | 11 (`c` de `0.0` à `1.0`) |
| Bins spectraux par état | 100 |
| Tenseur de spectre par échantillon | `11 x 100` |
| Tenseur de forme par échantillon | `4 x 3` (`[presence, x, y]`) |

### Objectifs d’apprentissage en trois étapes

| Étape | Direction | Checkpoint typique |
|---|---|---|
| A | `shape -> spectrum` | `stageA/shape2spec_stageA.pt` |
| B | `spectrum -> shape` | `stageB/spec2shape_stageB.pt` |
| C | `spectrum -> shape -> spectrum` (chaîne affinée) | `stageC/spec2shape_stageC.pt` |

## 🗂️ Structure du dépôt

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

## 🧩 Prérequis

| Dépendance | Notes |
|---|---|
| Linux + Bash | Les scripts shell supposent des chemins de style Linux |
| Conda | Gestion d’environnement recommandée (`iccp.yaml`) |
| Python 3.9 | Runtime principal pour les scripts d’entraînement/évaluation |
| Binaire S4 | Attendu à `../build/S4` par rapport à la racine du dépôt |
| CUDA (optionnel) | Accélère l’entraînement et l’évaluation |

## ⚙️ Configuration de l’environnement

```bash
git clone <repo-url> inverse_metasurface
cd inverse_metasurface

conda env create -f iccp.yaml
conda activate iccp

# verify S4 path expected by shell runners
ls -l ../build/S4
```

Bit exécutable optionnel pour les lanceurs shell :

```bash
chmod +x ms.sh ms_final.sh ms_resume_allargs.sh ms_resume.sh
```

## 🚀 Démarrage rapide

Si vous avez déjà `preprocessed_t_data.npz` :

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

## 🔁 Pipeline de bout en bout

### 1) Générer les données S4

```bash
./ms_final.sh \
  -ns 10000 \
  -r 12345 \
  -p myrun \
  -g 80 \
  -bo 0.35 \
  -ro 0.30
```

### 2) Fusionner les sorties S4 + sommets de formes

```bash
python merge_s4_data_full.py --prefix myrun
# -> merged_s4_shapes_myrun.csv
```

### 3) Normaliser les noms de colonnes CSV pour le prétraitement

Le prétraitement de `three_stage_transmittance.py` attend `prefix` et `nQ`, tandis que la sortie fusionnée peut contenir `folder_key` et `NQ`.

```bash
python -c "import pandas as pd; p='merged_s4_shapes_myrun.csv'; df=pd.read_csv(p); df=df.rename(columns={'folder_key':'prefix','NQ':'nQ'}); df.to_csv(p,index=False)"
```

### 4) Prétraitement CSV -> NPZ

```bash
mkdir -p merged_csvs
mv merged_s4_shapes_myrun.csv merged_csvs/

python three_stage_transmittance.py --preprocess \
  --input_folder merged_csvs \
  --output_npz preprocessed_t_data.npz
```

### 5) Entraîner les trois étapes

```bash
python three_stage_transmittance.py \
  --data_npz preprocessed_t_data.npz \
  --num_epochs 100 \
  --batch_size 1024
```

### 6) Évaluer et tracer

```bash
python three_stage_transmittance_evaluation.py \
  --model_dir outputs_three_stage_YYYYMMDD_HHMMSS \
  --data_npz preprocessed_t_data.npz \
  --sample_count 8
```

### 7) Optionnel : comparaison réseau neuronal vs S4

```bash
python FilterShapeS4_Evaluator_Transmittance.py \
  --npz_file preprocessed_t_data.npz \
  --spec2shape_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageC/spec2shape_stageC.pt \
  --shape2spec_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageA/shape2spec_stageA.pt \
  --n_samples 4
```

## 🧠 Entraînement et évaluation

### Scripts principaux

| Script | Objectif |
|---|---|
| `three_stage_transmittance.py` | prétraitement + entraînement des étapes A/B/C |
| `three_stage_transmittance_evaluation.py` | évaluer les checkpoints, calculer les métriques, sauvegarder les graphes |
| `FilterShapeS4_Evaluator_Transmittance.py` | comparer les prédictions apprises au comportement S4 |

### Sorties typiques

| Artefact | Emplacement |
|---|---|
| Checkpoints des étapes | `outputs_three_stage_*/stageA|stageB|stageC/` |
| Figures d’évaluation | `outputs_three_stage_*/evaluation_<timestamp>/` |
| CSV de métriques | `evaluation_metrics.csv`, `metrics_summary.csv` |

## 🛠️ Principales options CLI

### Lanceurs S4 (`ms_final.sh`, `ms_resume_allargs.sh`)

| Option | Signification | Par défaut |
|---|---|---|
| `-ns`, `--numshapes` | nombre de formes | `100000` |
| `-r`, `--seed` | graine aléatoire | `88888` |
| `-p`, `--prefix` | préfixe d’exécution/clé de reprise | vide |
| `-g`, `--numg` | paramètre de base géométrique | `80` |
| `-bo`, `--baseouter` | offset externe de base | `0.25` |
| `-ro`, `--randouter` | offset externe aléatoire | `0.20` |

### Entraînement (`three_stage_transmittance.py`)

| Option | Signification | Par défaut |
|---|---|---|
| `--preprocess` | bascule en mode prétraitement | désactivé |
| `--input_folder` | dossier contenant les CSV fusionnés | `""` |
| `--output_npz` | fichier de sortie du prétraitement | `preprocessed_data.npz` |
| `--data_npz` | entrée NPZ pour l’entraînement | `""` |
| `--csv_file` | entrée CSV directe pour l’entraînement | `""` |
| `--test` | bascule mode test | désactivé |
| `--num_epochs` | époques par étape | `10` |
| `--batch_size` | taille de lot | `4096` |

### Évaluation (`three_stage_transmittance_evaluation.py`)

| Option | Signification | Par défaut |
|---|---|---|
| `--model_dir` | dossier d’exécution entraîné (obligatoire) | - |
| `--data_npz` | entrée NPZ pour l’évaluation | `""` |
| `--csv_file` | entrée CSV pour l’évaluation | `""` |
| `--output_dir` | dossier de sortie personnalisé | auto |
| `--sample_count` | nombre d’échantillons visualisés | `4` |
| `--seed` | graine aléatoire pour la sélection d’échantillons | `23` |
| `--font_scale` | multiplicateur de police des graphes | `1.0` |
| `--batch_size` | taille de lot du dataloader d’évaluation | `32` |
| `--plot_only` | régénérer uniquement les graphes | désactivé |

## 🧯 Dépannage

| Symptôme | Cause probable | Correctif |
|---|---|---|
| `../build/S4: No such file or directory` | Binaire S4 absent du chemin relatif attendu | Placer/compiler S4 à `../build/S4` ou éditer les scripts |
| `Must specify either --data_npz or --csv_file` | Argument de dataset d’entraînement/évaluation manquant | Fournir exactement une seule entrée de dataset |
| `No transmission columns found` | Le CSV fusionné ne contient pas de colonnes `T@...` | Relancer merge/pivot et vérifier les en-têtes |
| `KeyError: 'prefix'` in preprocess | La sortie de fusion utilise encore `folder_key`/`NQ` | Renommer les colonnes en `prefix`/`nQ` avant le prétraitement |
| GPU OOM | lot trop grand | Réduire `--batch_size` |
| Missing checkpoints during eval | checkpoints d’étape absents/chemin erroné | Vérifier les fichiers stageA/B/C sous le `--model_dir` sélectionné |

## 🧭 Feuille de route

- Standardiser le schéma CSV fusionné entre les scripts de fusion (nommage `prefix`, `nQ`)
- Ajouter des tests automatisés pour merge/prétraitement/chargement des checkpoints
- Fournir un point d’entrée CLI unique pour l’orchestration du pipeline complet
- Ajouter des manifests de dataset/exécution pour la reproductibilité
- Ajouter une licence open source explicite

## 📚 Citation

Si ce dépôt contribue à votre recherche, veuillez citer :

```bibtex
@article{chen2025inverse,
  title={Inverse Design of Metasurface for Spectral Imaging},
  author={Chen, Rongzhou and Nie, Haitao and Zhu, Shuo and Zhao, Yaping and Wang, Chutian and Lam, Edmund Y},
  journal={arXiv preprint arXiv:2510.21924},
  year={2025}
}
```

## 📄 Licence

Aucun fichier `LICENSE` n’est actuellement présent dans ce dépôt. Les droits d’usage et de redistribution ne sont donc pas spécifiés tant qu’une licence n’est pas ajoutée.

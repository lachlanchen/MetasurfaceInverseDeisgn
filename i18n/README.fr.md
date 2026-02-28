[English](../README.md) · [العربية](README.ar.md) · [Español](README.es.md) · [Français](README.fr.md) · [日本語](README.ja.md) · [한국어](README.ko.md) · [Tiếng Việt](README.vi.md) · [中文 (简体)](README.zh-Hans.md) · [中文（繁體）](README.zh-Hant.md) · [Deutsch](README.de.md) · [Русский](README.ru.md)


# Conception Inverse de Métasurfaces pour l'Imagerie Spectrale

<p align="center">
  <img alt="Status" src="https://img.shields.io/badge/Status-Research%20Prototype-f59e0b?style=for-the-badge">
  <img alt="Python" src="https://img.shields.io/badge/Python-3.9-3776AB?style=for-the-badge&logo=python&logoColor=white">
  <img alt="PyTorch" src="https://img.shields.io/badge/PyTorch-2.x-EE4C2C?style=for-the-badge&logo=pytorch&logoColor=white">
  <img alt="Simulator" src="https://img.shields.io/badge/Simulator-S4%20RCWA-16a34a?style=for-the-badge">
  <img alt="Platform" src="https://img.shields.io/badge/Platform-Linux%20%2B%20Bash-6b7280?style=for-the-badge&logo=gnu-bash&logoColor=white">
  <img alt="ArXiv" src="https://img.shields.io/badge/arXiv-2510.21924-b31b1b?style=for-the-badge&logo=arxiv&logoColor=white">
</p>

Un dépôt de recherche centré sur des scripts (historiquement nommé `inverse_metasurface`) pour la conception inverse de métasurfaces en imagerie spectrale.

Le flux de travail principal combine :
- Simulation RCWA fondée sur la physique (`S4` + Lua)
- Assemblage des données et rattachement des formes
- Apprentissage PyTorch en trois étapes (`shape -> spectrum`, `spectrum -> shape`, affinement chaîné)
- Évaluation quantitative et qualitative, avec vérifications optionnelles de cohérence réseau-vs-S4

> [!IMPORTANT]
> Le comportement canonique et les commandes sont conservés à partir des scripts/docs existants du projet. Lorsque des références historiques pointent vers des fichiers absents, ces références sont volontairement conservées avec des notes explicites pour la compatibilité.

## 📑 Sommaire

- [✨ Vue d'ensemble](#-vue-densemble)
- [🌍 Internationalisation (i18n)](#-internationalisation-i18n)
- [✨ Fonctionnalités](#-fonctionnalités)
- [🧭 Flux de travail de bout en bout](#-flux-de-travail-de-bout-en-bout)
- [🧱 Structure du projet](#-structure-du-projet)
- [🛠️ Prérequis](#️-prérequis)
- [🚀 Installation](#-installation)
- [▶️ Utilisation](#️-utilisation)
- [⚙️ Configuration](#️-configuration)
- [🧪 Exemples](#-exemples)
- [🔬 Contexte de recherche](#-contexte-de-recherche)
- [🧑‍💻 Notes de développement](#-notes-de-développement)
- [🧯 Dépannage](#-dépannage)
- [🗺️ Feuille de route](#️-feuille-de-route)
- [🤝 Contribution](#-contribution)
- [📄 Licence](#-licence)
- [📚 Citation](#-citation)

## ✨ Vue d'ensemble

| Élément | Détails |
|---|---|
| 🎯 Tâche principale | Déduire la géométrie de métasurface à symétrie C4 à partir de spectres de transmittance cibles |
| 🔬 Simulateur | `../build/S4` appelé par les lanceurs shell et les scripts `.lua` |
| 🧠 Pipeline d'apprentissage | Étape A `shape -> spectra`, Étape B `spectra -> shape`, Étape C `spectra -> shape -> spectra` |
| 📦 Contrat de données | CSV fusionné (`T@...`, métadonnées, `vertices_str`) -> NPZ compressé (`uids`, `spectra`, `shapes`) |
| 🧪 Évaluation | Métriques MSE, visualisations par étape, re-simulation S4 fraîche optionnelle |
| 🌐 État i18n | Fichiers README multilingues à la racine + répertoire `i18n/` existant |

## 🌍 Internationalisation (i18n)

- Les README multilingues sont maintenus à la racine du dépôt sous forme de fichiers `README.<lang>.md`.
- Le répertoire `i18n/` existe dans cet instantané du dépôt.
- Ce fichier conserve une seule ligne d'options de langue en tête pour éviter les barres de langue dupliquées.
- `README.en.md` existe aussi dans le dépôt ; ce `README.md` reste la base canonique pour cette passe de mise à jour.

## ✨ Fonctionnalités

- Chaîne de conception inverse de bout en bout, de la sortie de simulation S4 jusqu'au modèle inverse entraîné.
- Paramétrisation polygonale à symétrie C4 et encodage des points Q1 (`4x3` : `presence, x, y`).
- Entraînement de modèles en trois étapes dans un seul script (`three_stage_transmittance.py`).
- Outils de fusion qui préservent la précision spectrale et rattachent les sommets à chaque forme.
- Évaluateur optionnel qui compare les prédictions apprises à une nouvelle simulation S4.
- Nombreuses branches exploratoires (AVIRIS, SWIR/noise, GSST, variantes archivées/dépréciées).

## 🧭 Flux de travail de bout en bout

1. Générer les sorties de simulation dans `results/` et les fichiers de polygones dans `shapes/`.
2. Fusionner les fichiers CSV S4 et rattacher les sommets des formes.
3. Normaliser les noms de colonnes fusionnés pour la compatibilité avec l'entraînement.
4. Prétraiter les fichiers CSV fusionnés en tenseurs NPZ.
5. Entraîner les modèles des étapes A/B/C.
6. Évaluer les checkpoints et visualiser le comportement.
7. Optionnellement, comparer les spectres de formes prédites avec de nouvelles exécutions S4.

## 🧱 Structure du projet

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

## 🛠️ Prérequis

| Dépendance | Notes |
|---|---|
| Linux + Bash | Les scripts de lancement visent une exécution shell |
| Python 3.9 | Correspond à `iccp.yaml` (`python=3.9.18`) |
| Conda | Recommandé pour la reproductibilité |
| Binaire S4 | Attendu à `../build/S4` |
| GPU CUDA (optionnel) | Accélère l'entraînement/l'évaluation |

## 🚀 Installation

### 1) Cloner et entrer dans le dossier

```bash
git clone <your-repo-url> inverse_metasurface
cd inverse_metasurface
```

### 2) Créer l'environnement (recommandé)

```bash
conda env create -f iccp.yaml
conda activate iccp
```

Note alternative :

```bash
# Historical README reference (file may be absent in this snapshot)
pip install -r pip_requirements.txt
```

### 3) Vérifier le chemin du simulateur attendu par les scripts

```bash
ls -l ../build/S4
```

### 4) (Optionnel) rendre les lanceurs exécutables

```bash
chmod +x ms.sh ms_final.sh ms_resume.sh ms_resume_allargs.sh ms_resume_random_state.sh ms_resume_random_state_nir.sh
```

## ▶️ Utilisation

### A) Générer des données de simulation RCWA

Lanceur simple :

```bash
./ms.sh -ns 10000 -r 12345
```

Lanceur paramétré :

```bash
./ms_final.sh \
  -ns 10000 \
  -r 12345 \
  -p myrun \
  -g 80 \
  -bo 0.35 \
  -ro 0.30
```

Lanceur orienté reprise :

```bash
./ms_resume_allargs.sh \
  -ns 10000 \
  -r 12345 \
  -p myrun \
  -g 80 \
  -bo 0.35 \
  -ro 0.30
```

Exemple supplémentaire reprise/état aléatoire (issu de la doc de commandes) :

```bash
./ms_resume_random_state.sh \
  -p iccp100kG20Ov \
  -r 88888 \
  -g 20 \
  -bo 0.35 \
  -ro 0.3 \
  -ns 100000
```

Notes :
- Les lanceurs exécutent `NQ=1..4` en parallèle.
- Les scripts appellent `../build/S4` avec `-t 32`.

### B) Fusionner les sorties S4 et rattacher les sommets des formes

```bash
python merge_s4_data_full.py --prefix myrun
# output: merged_s4_shapes_myrun.csv
```

### C) Normaliser les colonnes pour la compatibilité avec l'entraînement

`merge_s4_data_full.py` écrit `folder_key` et `NQ`, alors que le pipeline d'entraînement attend `prefix` et `nQ`.

```bash
python -c "import pandas as pd; p='merged_s4_shapes_myrun.csv'; df=pd.read_csv(p); df=df.rename(columns={'folder_key':'prefix','NQ':'nQ'}); df.to_csv(p,index=False)"
```

### D) Prétraiter CSV -> NPZ

```bash
mkdir -p merged_csvs
mv merged_s4_shapes_myrun.csv merged_csvs/

python three_stage_transmittance.py \
  --preprocess \
  --input_folder merged_csvs \
  --output_npz preprocessed_t_data.npz
```

### E) Entraîner les étapes A/B/C

```bash
python three_stage_transmittance.py \
  --data_npz preprocessed_t_data.npz \
  --num_epochs 100 \
  --batch_size 1024
```

Les sorties sont écrites dans :
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageA`
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageB`
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageC`

### F) Évaluer les modèles entraînés

Commande issue du README historique (nom de script conservé pour compatibilité avec les documents antérieurs) :

```bash
python three_stage_transmittance_evaluation.py \
  --model_dir outputs_three_stage_YYYYMMDD_HHMMSS \
  --data_npz preprocessed_t_data.npz \
  --sample_count 8
```

Note sur l'état du dépôt : `three_stage_transmittance_evaluation.py` n'est pas présent dans cet instantané. Utilisez `FilterShapeS4_Evaluator_Transmittance.py` pour les fonctionnalités d'évaluation disponibles.

### G) Vérification optionnelle de cohérence réseau-vs-S4

```bash
python FilterShapeS4_Evaluator_Transmittance.py \
  --npz_file preprocessed_t_data.npz \
  --spec2shape_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageC/spec2shape_stageC.pt \
  --shape2spec_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageA/shape2spec_stageA.pt \
  --n_samples 4
```

## ⚙️ Configuration

### Lanceurs S4 (`ms_final.sh`, `ms_resume_allargs.sh`)

| Flag | Signification | Valeur par défaut |
|---|---|---|
| `-ns`, `--numshapes` | Nombre de formes à générer | `100000` |
| `-r`, `--seed` | Graine aléatoire | `88888` |
| `-p`, `--prefix` | Préfixe/clé de reprise | `""` |
| `-g`, `--numg` | Paramètre de base/grille | `80` |
| `-bo`, `--baseouter` | Décalage de frontière externe de base | `0.25` |
| `-ro`, `--randouter` | Décalage aléatoire de frontière externe | `0.20` |

### Entraînement (`three_stage_transmittance.py`)

| Flag | Signification | Valeur par défaut |
|---|---|---|
| `--preprocess` | Exécuter le mode prétraitement | `False` |
| `--input_folder` | Dossier contenant les CSV fusionnés | `""` |
| `--output_npz` | Chemin NPZ de sortie | `preprocessed_data.npz` |
| `--data_npz` | Jeu de données NPZ pour l'entraînement | `""` |
| `--csv_file` | Repli CSV si NPZ non utilisé | `""` |
| `--test` | Mode test | `False` |
| `--num_epochs` | Nombre d'époques d'entraînement | `10` |
| `--batch_size` | Taille de lot | `4096` |

### Configuration historique d'évaluation (`three_stage_transmittance_evaluation.py`)

| Flag | Signification | Valeur par défaut |
|---|---|---|
| `--model_dir` | Répertoire contenant `stageA/B/C` | requis |
| `--data_npz` | Entrée NPZ | `""` |
| `--csv_file` | Entrée CSV de repli | `""` |
| `--output_dir` | Surcharge du répertoire de sortie | auto sous `model_dir` |
| `--sample_count` | Nombre d'échantillons visualisés | `4` |
| `--seed` | Graine aléatoire | `23` |
| `--font_scale` | Mise à l'échelle de police des graphes | `1.0` |
| `--batch_size` | Taille de lot pour l'évaluation | `32` |
| `--plot_only` | Tracer uniquement les courbes d'entraînement | `False` |

### Évaluateur de cohérence S4 (`FilterShapeS4_Evaluator_Transmittance.py`)

| Flag | Signification | Valeur par défaut |
|---|---|---|
| `--npz_file` | Fichier NPZ d'entrée | `preprocessed_t_data.npz` |
| `--spec2shape_ckpt` | Chemin du checkpoint de l'étape C | `outputs_three_stage_20250322_145925/stageC/spec2shape_stageC.pt` |
| `--shape2spec_ckpt` | Chemin du checkpoint de l'étape A | `outputs_three_stage_20250322_145925/stageA/shape2spec_stageA.pt` |
| `--n_samples` | Nombre d'échantillons évalués | `4` |
| `--seed` | Graine aléatoire | `23` |
| `--max_workers` | Threads de travail S4 | `4` |
| `--out_folder` | Répertoire de sortie | horodatage auto |

## 🧪 Exemples

### Exécution smoke test

```bash
./ms_final.sh -ns 1000 -r 42 -p smoke -g 40 -bo 0.25 -ro 0.20
python merge_s4_data_full.py --prefix smoke
python -c "import pandas as pd; p='merged_s4_shapes_smoke.csv'; d=pd.read_csv(p).rename(columns={'folder_key':'prefix','NQ':'nQ'}); d.to_csv(p,index=False)"
mkdir -p merged_csvs && mv merged_s4_shapes_smoke.csv merged_csvs/
python three_stage_transmittance.py --preprocess --input_folder merged_csvs --output_npz smoke.npz
python three_stage_transmittance.py --data_npz smoke.npz --num_epochs 5 --batch_size 128
```

### Exemples de profils d'exécution (depuis `commands.md` / `commands_updated.md`)

```bash
# no overlap, G=40
./ms_resume_allargs.sh -ns 10000 -r 12345 -p fast_without_overlap -g 40 -bo 0.25 -ro 0.2

# overlap, G=80
./ms_resume_allargs.sh -ns 10000 -r 12345 -p overlap -g 80 -bo 0.35 -ro 0.3

# large run, overlap, G=80
./ms_resume_allargs.sh -ns 100000 -r 12345 -p more_basis_overlap -g 80 -bo 0.35 -ro 0.3
```

## 🔬 Contexte de recherche

La configuration actuelle de conception inverse apprend à reconstruire une géométrie à symétrie C4 à partir de la transmittance mesurée sur des états de cristallisation. Le pipeline de transmittance suppose actuellement :

- 11 lignes de cristallisation par échantillon de forme (regroupées par `shape_uid` unique)
- 100 bins de longueur d'onde par état de cristallisation (colonnes `T@...`)
- Jusqu'à 4 points de contrôle Q1 codés comme tenseur `4x3` : `(presence, x, y)`
- Reconstruction polygonale sous symétrie C4 pour la visualisation des formes et les vérifications de cohérence

Le dépôt contient aussi des branches exploratoires (`AVIRIS*`, `noise_experiment*`, `archived/`) au-delà du chemin principal d'entraînement sur la transmittance.

## 🧑‍💻 Notes de développement

- Il s'agit d'un dépôt de recherche centré scripts plutôt que d'un module Python packagé.
- Les scripts principaux supposent des chemins relatifs (notamment `../build/S4`, `results/`, `shapes/`).
- `.gitignore` exclut de nombreux artefacts d'expériences générés (`*.csv`, `*.npz`, `*.pt`, dossiers d'exécution).
- Certains fichiers/répertoires des documents historiques sont actuellement absents dans cet instantané ; ces références sont volontairement conservées avec des notes pour la compatibilité.
- Des fichiers auxiliaires macOS (`._*`) sont présents et peuvent être des artefacts de métadonnées non fonctionnels.

## 🧯 Dépannage

| Symptôme | Cause probable | Correctif |
|---|---|---|
| `../build/S4: No such file or directory` | Binaire S4 absent du chemin relatif attendu | Compiler ou lier S4 à `../build/S4`, ou mettre à jour les chemins des lanceurs |
| `No transmission columns found` | CSV sans colonnes `T@...` | Revérifier le format de sortie de fusion |
| `Must specify either --data_npz or --csv_file` | Argument de données d'entraînement/évaluation manquant | Fournir explicitement une entrée |
| `No valid shapes => SHIFT->Q1->UpTo4` | `vertices_str` invalide/vide ou filtrage Q1 supprimant tous les échantillons | Valider les fichiers de formes et la sortie de fusion |
| Sortie de fusion vide pour `--prefix` | Le préfixe ne correspond pas aux fichiers dans `results/` | Vérifier le préfixe exact des noms de fichiers et relancer la fusion |
| Checkpoint d'évaluation manquant | Fichiers de checkpoint `stageA/B/C` manquants | Vérifier que `--model_dir` pointe vers un dossier de sortie complet |
| `three_stage_transmittance_evaluation.py` introuvable | Script référencé par les docs historiques mais absent actuellement | Utiliser `FilterShapeS4_Evaluator_Transmittance.py` ou restaurer ce script depuis d'anciens commits |

## 🗺️ Feuille de route

- Améliorer la reproductibilité avec un manifeste explicite de versionnement des données et des configurations d'exécution figées.
- Consolider les points d'entrée canoniques pour les branches transmittance, AVIRIS et noise.
- Ajouter des smoke tests automatisés pour le prétraitement et une mini époque d'entraînement.
- Ajouter un registre d'expériences plus clair reliant les dossiers de sortie aux lignes de commande exactes.
- Étendre le workflow de synchronisation README multilingue (fichiers de langue à la racine et `i18n/`).

## 🤝 Contribution

Les contributions sont bienvenues, en particulier pour la reproductibilité, les tests et la qualité de la documentation.

Processus suggéré :

1. Ouvrir une issue avec le périmètre et le comportement attendu.
2. Créer une branche ciblée.
3. Soumettre une pull request avec des commandes et sorties reproductibles.
4. Garder les changements centrés sur un seul workflow lorsque c'est possible.

## 📄 Licence

Aucun fichier `LICENSE` n'est actuellement présent à la racine du dépôt dans cet instantané. Ajoutez-en un pour définir les conditions d'utilisation et de redistribution.

## 📚 Citation

Si vous utilisez ce dépôt ou vous appuyez sur ce travail, veuillez citer :

```bibtex
@article{chen2025inverse,
  title={Inverse Design of Metasurface for Spectral Imaging},
  author={Chen, Rongzhou and Nie, Haitao and Zhu, Shuo and Zhao, Yaping and Wang, Chutian and Lam, Edmund Y},
  journal={arXiv preprint arXiv:2510.21924},
  year={2025}
}
```

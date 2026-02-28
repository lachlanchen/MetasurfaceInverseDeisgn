[English](../README.md) · [العربية](README.ar.md) · [Español](README.es.md) · [Français](README.fr.md) · [日本語](README.ja.md) · [한국어](README.ko.md) · [Tiếng Việt](README.vi.md) · [中文 (简体)](README.zh-Hans.md) · [中文（繁體）](README.zh-Hant.md) · [Deutsch](README.de.md) · [Русский](README.ru.md)


[![LazyingArt banner](https://github.com/lachlanchen/lachlanchen/raw/main/figs/banner.png)](https://github.com/lachlanchen/lachlanchen/blob/main/figs/banner.png)

# Conception inverse de métasurfaces pour l'imagerie spectrale

<p align="center">
  <img alt="Status" src="https://img.shields.io/badge/Status-Research%20Prototype-f59e0b?style=for-the-badge">
  <img alt="Python" src="https://img.shields.io/badge/Python-3.9-3776AB?style=for-the-badge&logo=python&logoColor=white">
  <img alt="PyTorch" src="https://img.shields.io/badge/PyTorch-2.x-EE4C2C?style=for-the-badge&logo=pytorch&logoColor=white">
  <img alt="Simulator" src="https://img.shields.io/badge/Simulator-S4%20RCWA-16a34a?style=for-the-badge">
  <img alt="Platform" src="https://img.shields.io/badge/Platform-Linux%20%2B%20Bash-6b7280?style=for-the-badge&logo=gnu-bash&logoColor=white">
  <img alt="ArXiv" src="https://img.shields.io/badge/arXiv-2510.21924-b31b1b?style=for-the-badge&logo=arxiv&logoColor=white">
</p>

Un dépôt de recherche orienté scripts (historiquement nommé `inverse_metasurface`) pour la conception inverse de métasurfaces en imagerie spectrale.

Le flux de travail principal combine :
- Une simulation RCWA basée sur la physique (`S4` + Lua)
- L'assemblage des données et l'attachement des formes
- Un apprentissage PyTorch en trois étapes (`shape -> spectrum`, `spectrum -> shape`, réglage fin en chaîne)
- Une évaluation quantitative et qualitative, avec des vérifications optionnelles de cohérence réseau neuronal vs S4

> [!IMPORTANT]
> Le comportement canonique et les commandes sont préservés depuis les scripts/docs existants du projet. Lorsque des références historiques pointent vers des fichiers absents, ces références sont conservées volontairement avec des notes explicites pour rester compatibles.

## 📑 Contenu

- [🌟 Aperçu](#-aperçu)
- [✨ En un coup d'œil](#-en-un-coup-doeil)
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
- [📚 Référence](#-référence)

## 🌟 Aperçu

| Focus | Statut |
|---|---|
| 🧠 Objectif | Reconstruction inverse de la géométrie d'une métasurface C4-symétrique à partir de données spectrales |
| 🔧 Pile centrale | S4 RCWA (`Lua`) + entraînement PyTorch + revalidation optionnelle géométrie-vers-spectre |
| 🧪 Pipeline de données | Fusion CSV (`T@...`, métadonnées, `vertices_str`) -> NPZ compressé (`uids`, `spectra`, `shapes`) |
| 🚀 Niveau de préparation | Prototype de recherche ; scripts et docs conservés en compatibilité avec les références historiques |

## ✨ En un coup d'œil

| Élément | Détails |
|---|---|
| 🎯 Tâche principale | Inférer la géométrie C4-symétrique d'une métasurface à partir de spectres de transmittance cibles |
| 🔬 Simulateur | `../build/S4` appelé par les lanceurs shell et les scripts `.lua` |
| 🧠 Pipeline d'apprentissage | Étape A `shape -> spectra`, Étape B `spectra -> shape`, Étape C `spectra -> shape -> spectra` |
| 📦 Contrat de données | CSV fusionné (`T@...`, métadonnées, `vertices_str`) -> NPZ compressé (`uids`, `spectra`, `shapes`) |
| 🧪 Évaluation | Métriques MSE, visualisations par étape, re-simulation S4 optionnelle |
| 🌐 Statut i18n | README multilingues au niveau racine + répertoire `i18n/` existant |

## 🌍 Internationalisation (i18n)

- Les README multilingues sont maintenus à la racine du dépôt sous forme de `README.<lang>.md`.
- Le répertoire `i18n/` existe dans cette version du dépôt.
- Ce fichier conserve une seule ligne d'options de langue en tête afin d'éviter la duplication des barres de langue.
- `README.en.md` existe aussi dans le dépôt ; ce `README.md` reste la base canonique pour cette passe de mise à jour.

## ✨ Fonctionnalités

- Parcours complet de conception inverse, de la sortie de simulation S4 au modèle inverse entraîné.
- Paramétrisation polygonale à symétrie C4 et encodage de points Q1 (`4x3` : `presence, x, y`).
- Entraînement de modèle en trois étapes dans un même script (`three_stage_transmittance.py`).
- Outils de fusion qui conservent la précision spectrale et attachent les sommets par forme.
- Évaluateur optionnel comparant les prédictions apprises avec une simulation S4 fraîche.
- Branches exploratoires étendues (AVIRIS, SWIR/bruit, GSST, variantes archivées/dépréciées).

## 🧭 Flux de travail de bout en bout

1. Générer les sorties de simulation dans `results/` et les fichiers polygonaux dans `shapes/`.
2. Fusionner les CSV S4 et attacher les sommets de forme.
3. Normaliser les noms de colonnes fusionnées pour compatibilité avec l'entraînement.
4. Prétraiter les CSV fusionnés en tenseurs NPZ.
5. Entraîner les modèles des étapes A/B/C.
6. Évaluer les points de contrôle et visualiser le comportement.
7. Comparer optionnellement les spectres de formes prédites avec de nouveaux runs S4.

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

| Dépendance | Remarques |
|---|---|
| Linux + Bash | Les lanceurs ciblent une exécution en shell |
| Python 3.9 | Conforme à `iccp.yaml` (`python=3.9.18`) |
| Conda | Recommandé pour la reproductibilité |
| Binaire S4 | Attendu dans `../build/S4` |
| GPU CUDA (optionnel) | Accélère l'entraînement et l'évaluation |

## 🚀 Installation

### 1) Cloner et entrer

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
# Référence historique du README (fichier possiblement absent dans cet état)
pip install -r pip_requirements.txt
```

### 3) Vérifier le chemin du simulateur attendu par les scripts

```bash
ls -l ../build/S4
```

### 4) Rendre les lanceurs exécutables (optionnel)

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

Autre exemple de reprise/seed aléatoire (depuis la documentation des commandes) :

```bash
./ms_resume_random_state.sh \
  -p iccp100kG20Ov \
  -r 88888 \
  -g 20 \
  -bo 0.35 \
  -ro 0.3 \
  -ns 100000
```

Remarques :
- Les lanceurs exécutent `NQ=1..4` en parallèle.
- Les scripts appellent `../build/S4` avec `-t 32`.

### B) Fusionner les sorties S4 et attacher les sommets

```bash
python merge_s4_data_full.py --prefix myrun
# output: merged_s4_shapes_myrun.csv
```

### C) Normaliser les colonnes pour la compatibilité de l'entraînement

`merge_s4_data_full.py` écrit `folder_key` et `NQ`, tandis que le flux d'entraînement attend `prefix` et `nQ`.

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

Commande historique du README (nom de script conservé pour compatibilité avec les docs antérieures) :

```bash
python three_stage_transmittance_evaluation.py \
  --model_dir outputs_three_stage_YYYYMMDD_HHMMSS \
  --data_npz preprocessed_t_data.npz \
  --sample_count 8
```

Note d'état du dépôt : `three_stage_transmittance_evaluation.py` n'est pas présent dans cette instantané. Utiliser `FilterShapeS4_Evaluator_Transmittance.py` pour la fonctionnalité d'évaluation disponible.

### G) Vérification optionnelle de cohérence neural-vs-S4

```bash
python FilterShapeS4_Evaluator_Transmittance.py \
  --npz_file preprocessed_t_data.npz \
  --spec2shape_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageC/spec2shape_stageC.pt \
  --shape2spec_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageA/shape2spec_stageA.pt \
  --n_samples 4
```

## ⚙️ Configuration

### Lanceurs S4 (`ms_final.sh`, `ms_resume_allargs.sh`)

| Drapeau | Signification | Par défaut |
|---|---|---|
| `-ns`, `--numshapes` | Nombre de formes à générer | `100000` |
| `-r`, `--seed` | Graine aléatoire | `88888` |
| `-p`, `--prefix` | Préfixe/clé de reprise | `""` |
| `-g`, `--numg` | Paramètre de base/grille | `80` |
| `-bo`, `--baseouter` | Décalage de frontière extérieure de base | `0.25` |
| `-ro`, `--randouter` | Décalage aléatoire de frontière extérieure | `0.20` |

### Entraînement (`three_stage_transmittance.py`)

| Drapeau | Signification | Par défaut |
|---|---|---|
| `--preprocess` | Exécuter le mode prétraitement | `False` |
| `--input_folder` | Dossier contenant les CSV fusionnés | `""` |
| `--output_npz` | Chemin NPZ de sortie | `preprocessed_data.npz` |
| `--data_npz` | NPZ dataset pour l'entraînement | `""` |
| `--csv_file` | Fallback CSV si NPZ non utilisé | `""` |
| `--test` | Mode test | `False` |
| `--num_epochs` | Nombre d'époques d'entraînement | `10` |
| `--batch_size` | Taille de lot | `4096` |

### Configuration d'évaluation historique (`three_stage_transmittance_evaluation.py`)

| Drapeau | Signification | Par défaut |
|---|---|---|
| `--model_dir` | Répertoire contenant `stageA/B/C` | required |
| `--data_npz` | Entrée NPZ | `""` |
| `--csv_file` | Entrée CSV de secours | `""` |
| `--output_dir` | Surcharge du répertoire de sortie | auto sous `model_dir` |
| `--sample_count` | Nombre d'échantillons visualisés | `4` |
| `--seed` | Graine aléatoire | `23` |
| `--font_scale` | Échelle de police des tracés | `1.0` |
| `--batch_size` | Taille de lot d'évaluation | `32` |
| `--plot_only` | Tracer uniquement les courbes d'entraînement | `False` |

### Évaluateur de cohérence S4 (`FilterShapeS4_Evaluator_Transmittance.py`)

| Drapeau | Signification | Par défaut |
|---|---|---|
| `--npz_file` | NPZ d'entrée | `preprocessed_t_data.npz` |
| `--spec2shape_ckpt` | Chemin de checkpoint de l'étape C | `outputs_three_stage_20250322_145925/stageC/spec2shape_stageC.pt` |
| `--shape2spec_ckpt` | Chemin de checkpoint de l'étape A | `outputs_three_stage_20250322_145925/stageA/shape2spec_stageA.pt` |
| `--n_samples` | Nombre d'échantillons évalués | `4` |
| `--seed` | Graine aléatoire | `23` |
| `--max_workers` | Threads workers S4 | `4` |
| `--out_folder` | Répertoire de sortie | auto horodatage |

## 🧪 Exemples

### Exécution rapide

```bash
./ms_final.sh -ns 1000 -r 42 -p smoke -g 40 -bo 0.25 -ro 0.20
python merge_s4_data_full.py --prefix smoke
python -c "import pandas as pd; p='merged_s4_shapes_smoke.csv'; d=pd.read_csv(p).rename(columns={'folder_key':'prefix','NQ':'nQ'}); d.to_csv(p,index=False)"
mkdir -p merged_csvs && mv merged_s4_shapes_smoke.csv merged_csvs/
python three_stage_transmittance.py --preprocess --input_folder merged_csvs --output_npz smoke.npz
python three_stage_transmittance.py --data_npz smoke.npz --num_epochs 5 --batch_size 128
```

### Exemples de profil d'exécution (depuis `commands.md` / `commands_updated.md`)

```bash
# no overlap, G=40
./ms_resume_allargs.sh -ns 10000 -r 12345 -p fast_without_overlap -g 40 -bo 0.25 -ro 0.2

# overlap, G=80
./ms_resume_allargs.sh -ns 10000 -r 12345 -p overlap -g 80 -bo 0.35 -ro 0.3

# large run, overlap, G=80
./ms_resume_allargs.sh -ns 100000 -r 12345 -p more_basis_overlap -g 80 -bo 0.35 -ro 0.3
```

## 🔬 Contexte de recherche

La configuration actuelle de conception inverse apprend à reconstruire une géométrie C4-symétrique à partir de la transmittance selon les états de cristallisation. Le pipeline de transmittance suppose actuellement :

- 11 lignes de cristallisation par échantillon de forme (groupées par `shape_uid` unique)
- 100 bins de longueur d'onde par état de cristallisation (`T@...` colonnes)
- Jusqu'à 4 points de contrôle Q1 encodés comme tenseur `4x3` : (`presence, x, y`)
- Reconstruction polygonale sous symétrie C4 pour la visualisation des formes et les vérifications de cohérence

Le dépôt contient également des branches exploratoires (`AVIRIS*`, `noise_experiment*`, `archived/`) au-delà du flux principal d'entraînement en transmittance.

## 🧑‍💻 Notes de développement

- Il s'agit d'un dépôt de recherche centré sur des scripts plutôt que d'un module Python packagé.
- Les scripts principaux supposent des chemins relatifs (notamment `../build/S4`, `results/`, `shapes/`).
- `.gitignore` exclut de nombreux artefacts d'expériences générés (`*.csv`, `*.npz`, `*.pt`, dossiers de runs).
- Certains fichiers/répertoires cités dans la documentation historique sont actuellement absents de cette version ; ces références sont conservées intentionnellement avec des notes de compatibilité.
- Les fichiers sidecar macOS (`._*`) sont présents et peuvent être de simples artefacts de métadonnées non fonctionnels.

## 🧯 Dépannage

| Symptôme | Cause probable | Correctif |
|---|---|---|
| `../build/S4: No such file or directory` | Binaire S4 absent au chemin relatif attendu | Construire ou lier S4 dans `../build/S4`, ou mettre à jour les chemins des lanceurs |
| `No transmission columns found` | CSV sans colonnes `T@...` | Vérifier le format de sortie de la fusion |
| `Must specify either --data_npz or --csv_file` | Argument d'entrée `data_npz`/`csv_file` manquant | Fournir explicitement une source de données |
| `No valid shapes => SHIFT->Q1->UpTo4` | `vertices_str` invalide/vides ou filtrage Q1 supprimant tous les échantillons | Valider les fichiers de forme et la sortie de fusion |
| Sortie vide de la fusion pour `--prefix` | Le préfixe ne correspond à aucun fichier dans `results/` | Vérifier le préfixe exact du nom de fichier et relancer la fusion |
| Checkpoint d'évaluation manquant | Fichiers de checkpoint `stageA/B/C` manquants | Vérifier que `--model_dir` pointe vers un dossier de sortie complet |
| `three_stage_transmittance_evaluation.py` non trouvé | Script référencé par des docs historiques mais absent actuellement | Utiliser `FilterShapeS4_Evaluator_Transmittance.py` ou restaurer ce script depuis des versions antérieures |

## 🗺️ Feuille de route

- Améliorer la reproductibilité via un manifeste explicite de version des données et des configurations de run figées.
- Consolider les points d'entrée canoniques pour les branches transmittance, AVIRIS et bruit.
- Ajouter des mini-tests automatisés de prétraitement et d'une mini époque d'entraînement.
- Ajouter un registre d'expériences plus clair liant les dossiers de sortie aux lignes de commande exactes.
- Étendre le flux de synchronisation des README multilingues (fichiers racine et `i18n/`).

## 🤝 Contribution

Les contributions sont les bienvenues, notamment pour la reproductibilité, les tests et la qualité de la documentation.

Processus conseillé :

1. Ouvrir une issue avec le périmètre et le comportement attendu.
2. Créer une branche ciblée.
3. Soumettre une pull request avec commandes exécutables et sorties attendues.
4. Limiter les changements à un flux de travail unique chaque fois que possible.

## 📄 Licence

Aucun fichier `LICENSE` n'est actuellement présent à la racine de ce dépôt dans cette version. Ajoutez-en un pour définir les conditions d'utilisation et de redistribution.

## 📚 Référence

Si vous utilisez ce dépôt ou partez de ce travail, veuillez citer :

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

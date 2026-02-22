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

Un dépôt de recherche orienté scripts (historiquement appelé `inverse_metasurface`) pour la conception inverse de métasurfaces en imagerie spectrale. Le flux de travail principal combine :

- une simulation RCWA fondée sur la physique (S4 + Lua)
- l’assemblage des données et l’association des formes
- un apprentissage PyTorch en trois étapes (`shape -> spectrum`, `spectrum -> shape`, affinage chaîné)
- une évaluation quantitative et qualitative, avec vérifications optionnelles de cohérence neural-vs-S4

## ✨ Vue d’ensemble

| Élément | Détails |
|---|---|
| 🎯 Tâche principale | Déduire une géométrie de métasurface à symétrie C4 à partir de spectres de transmittance cibles |
| 🔬 Simulateur | `../build/S4` appelé par des lanceurs shell et des scripts `.lua` |
| 🧠 Pipeline d’apprentissage | Étape A `shape -> spectra`, étape B `spectra -> shape`, étape C `spectra -> shape -> spectra` |
| 📦 Contrat de données | CSV fusionné (`T@...`, métadonnées, `vertices_str`) -> NPZ compressé (`uids`, `spectra`, `shapes`) |
| 🧪 Évaluation | Métriques MSE, visualisations par étape, re-simulation S4 optionnelle |

## 🧭 Flux de travail de bout en bout

1. Générer les sorties de simulation dans `results/` et les fichiers de polygones dans `shapes/`.
2. Fusionner les CSV S4 et y associer les sommets de forme.
3. Normaliser les noms de colonnes fusionnées pour la compatibilité avec l’entraînement.
4. Prétraiter les CSV fusionnés en tenseurs NPZ.
5. Entraîner les modèles des étapes A/B/C.
6. Évaluer les checkpoints et visualiser le comportement.
7. Optionnellement, comparer les spectres de formes prédites à de nouvelles exécutions S4.

## 🧱 Structure du dépôt

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

## 🛠️ Prérequis

| Dépendance | Notes |
|---|---|
| Linux + Bash | Les scripts de lancement ciblent une exécution shell |
| Python 3.9 | Correspond à `iccp.yaml` |
| Conda | Recommandé pour la reproductibilité |
| Binaire S4 | Attendu à l’emplacement `../build/S4` |
| GPU CUDA (optionnel) | Accélère l’entraînement/l’évaluation |

## 🚀 Installation

### 1) Cloner et entrer dans le dossier

```bash
git clone <your-repo-url> inverse_metasurface
cd inverse_metasurface
```

### 2) Créer l’environnement (recommandé)

```bash
conda env create -f iccp.yaml
conda activate iccp
```

Alternative (moins contrôlée) :

```bash
pip install -r pip_requirements.txt
```

### 3) Vérifier le chemin du simulateur attendu par les scripts

```bash
ls -l ../build/S4
```

### 4) (Optionnel) rendre les lanceurs exécutables

```bash
chmod +x ms.sh ms_final.sh ms_resume.sh ms_resume_allargs.sh ms_resume_random_state.sh
```

## ▶️ Utilisation pratique

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

Notes :
- Les lanceurs exécutent `NQ=1..4` en parallèle.
- Les scripts appellent `../build/S4` avec `-t 32`.

### B) Fusionner les sorties S4 et associer les sommets de forme

```bash
python merge_s4_data_full.py --prefix myrun
# output: merged_s4_shapes_myrun.csv
```

### C) Normaliser les colonnes pour la compatibilité avec l’entraînement

`merge_s4_data_full.py` écrit `folder_key` et `NQ`, alors que le pipeline d’entraînement attend `prefix` et `nQ`.

```bash
python -c "import pandas as pd; p='merged_s4_shapes_myrun.csv'; df=pd.read_csv(p); df=df.rename(columns={'folder_key':'prefix','NQ':'nQ'}); df.to_csv(p,index=False)"
```

### D) Prétraitement CSV -> NPZ

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

```bash
python three_stage_transmittance_evaluation.py \
  --model_dir outputs_three_stage_YYYYMMDD_HHMMSS \
  --data_npz preprocessed_t_data.npz \
  --sample_count 8
```

### G) Vérification optionnelle de cohérence neural-vs-S4

```bash
python FilterShapeS4_Evaluator_Transmittance.py \
  --npz_file preprocessed_t_data.npz \
  --spec2shape_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageC/spec2shape_stageC.pt \
  --shape2spec_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageA/shape2spec_stageA.pt \
  --n_samples 4
```

## ⚙️ Principales options CLI

### Lanceurs S4 (`ms_final.sh`, `ms_resume_allargs.sh`)

| Option | Signification | Défaut |
|---|---|---|
| `-ns`, `--numshapes` | Nombre de formes à générer | `100000` |
| `-r`, `--seed` | Graine aléatoire | `88888` |
| `-p`, `--prefix` | Préfixe/clé de reprise | `""` |
| `-g`, `--numg` | Paramètre de base/grille | `80` |
| `-bo`, `--baseouter` | Décalage de la frontière externe de base | `0.25` |
| `-ro`, `--randouter` | Décalage aléatoire de la frontière externe | `0.20` |

### Entraînement (`three_stage_transmittance.py`)

| Option | Signification | Défaut |
|---|---|---|
| `--preprocess` | Exécuter le mode prétraitement | `False` |
| `--input_folder` | Dossier contenant les CSV fusionnés | `""` |
| `--output_npz` | Chemin du NPZ de sortie | `preprocessed_data.npz` |
| `--data_npz` | Jeu de données NPZ pour l’entraînement | `""` |
| `--csv_file` | Repli CSV si NPZ non utilisé | `""` |
| `--test` | Mode test | `False` |
| `--num_epochs` | Nombre d’époques d’entraînement | `10` |
| `--batch_size` | Taille de lot | `4096` |

### Évaluation (`three_stage_transmittance_evaluation.py`)

| Option | Signification | Défaut |
|---|---|---|
| `--model_dir` | Dossier contenant `stageA/B/C` | requis |
| `--data_npz` | Entrée NPZ | `""` |
| `--csv_file` | Entrée CSV de repli | `""` |
| `--output_dir` | Surcharge du dossier de sortie | auto sous `model_dir` |
| `--sample_count` | Nombre d’échantillons visualisés | `4` |
| `--seed` | Graine aléatoire | `23` |
| `--font_scale` | Mise à l’échelle de police des figures | `1.0` |
| `--batch_size` | Taille de lot pour l’évaluation | `32` |
| `--plot_only` | Tracer uniquement les courbes d’entraînement | `False` |

### Évaluateur de cohérence S4 (`FilterShapeS4_Evaluator_Transmittance.py`)

| Option | Signification | Défaut |
|---|---|---|
| `--npz_file` | Fichier NPZ d’entrée | `preprocessed_t_data.npz` |
| `--spec2shape_ckpt` | Chemin du checkpoint étape C | `outputs_three_stage_20250322_145925/stageC/spec2shape_stageC.pt` |
| `--shape2spec_ckpt` | Chemin du checkpoint étape A | `outputs_three_stage_20250322_145925/stageA/shape2spec_stageA.pt` |
| `--n_samples` | Nombre d’échantillons évalués | `4` |
| `--seed` | Graine aléatoire | `23` |
| `--max_workers` | Threads de calcul S4 | `4` |
| `--out_folder` | Dossier de sortie | horodatage auto |

## 🧪 Exécution de validation rapide

```bash
./ms_final.sh -ns 1000 -r 42 -p smoke -g 40 -bo 0.25 -ro 0.20
python merge_s4_data_full.py --prefix smoke
python -c "import pandas as pd; p='merged_s4_shapes_smoke.csv'; d=pd.read_csv(p).rename(columns={'folder_key':'prefix','NQ':'nQ'}); d.to_csv(p,index=False)"
mkdir -p merged_csvs && mv merged_s4_shapes_smoke.csv merged_csvs/
python three_stage_transmittance.py --preprocess --input_folder merged_csvs --output_npz smoke.npz
python three_stage_transmittance.py --data_npz smoke.npz --num_epochs 5 --batch_size 128
```

## 🔬 Contexte de recherche

La configuration actuelle de conception inverse apprend à reconstruire une géométrie à symétrie C4 à partir de la transmittance sur différents états de cristallisation. Le pipeline de transmittance suppose actuellement :

- 11 lignes de cristallisation par échantillon de forme (regroupées par `shape_uid` unique)
- 100 bins de longueur d’onde par état de cristallisation (colonnes `T@...`)
- jusqu’à 4 points de contrôle Q1 encodés en tenseur `4x3` : `(presence, x, y)`
- reconstruction polygonale sous symétrie C4 pour la visualisation des formes et les vérifications de cohérence

Le dépôt contient aussi des branches exploratoires (`AVIRIS*`, `noise_experiment*`, `archived/`) au-delà du chemin principal d’entraînement sur la transmittance.

## 🧯 Dépannage

| Symptôme | Cause probable | Correctif |
|---|---|---|
| `../build/S4: No such file or directory` | Binaire S4 absent du chemin relatif attendu | Compiler ou lier S4 vers `../build/S4`, ou mettre à jour les chemins des lanceurs |
| `No transmission columns found` | CSV sans colonnes `T@...` | Revérifier le format de sortie de fusion |
| `Must specify either --data_npz or --csv_file` | Argument de données d’entraînement/évaluation manquant | Fournir explicitement une entrée |
| `No valid shapes => SHIFT->Q1->UpTo4` | `vertices_str` invalide/vide ou filtrage Q1 supprimant tous les échantillons | Valider les fichiers de formes et la sortie de fusion |
| Sortie de fusion vide pour `--prefix` | Le préfixe ne correspond pas aux fichiers dans `results/` | Vérifier le préfixe exact du nom de fichier et relancer la fusion |
| Checkpoint d’évaluation introuvable | Fichiers de checkpoint `stageA/B/C` manquants | Vérifier que `--model_dir` pointe vers un dossier de sortie complet |

## 🤝 Contribution

Les contributions sont bienvenues, en particulier sur la reproductibilité, les tests et la qualité de la documentation.

Processus suggéré :

1. Ouvrir une issue avec le périmètre et le comportement attendu.
2. Créer une branche ciblée.
3. Soumettre une pull request avec des commandes exécutables et leurs sorties.
4. Garder des modifications limitées à un seul flux de travail lorsque c’est possible.

## 📄 Licence

Aucun fichier `LICENSE` n’est actuellement présent à la racine du dépôt dans cet instantané. Ajoutez-en un pour définir les conditions d’utilisation et de redistribution.

## 📚 Citation

Si vous utilisez ce dépôt ou prolongez ce travail, merci de citer :

```bibtex
@article{chen2025inverse,
  title={Inverse Design of Metasurface for Spectral Imaging},
  author={Chen, Rongzhou and Nie, Haitao and Zhu, Shuo and Zhao, Yaping and Wang, Chutian and Lam, Edmund Y},
  journal={arXiv preprint arXiv:2510.21924},
  year={2025}
}
```

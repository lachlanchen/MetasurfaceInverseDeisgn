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
  <img alt="Simulator" src="https://img.shields.io/badge/RCWA-S4-16a34a?style=for-the-badge">
  <img alt="Platform" src="https://img.shields.io/badge/Platform-Linux%2FBash-6b7280?style=for-the-badge&logo=gnu-bash&logoColor=white">
</p>

Un dépôt de recherche orienté scripts (historiquement référencé sous le nom `inverse_metasurface`) pour la conception inverse de métasurfaces en imagerie spectrale. Le pipeline couple une **simulation RCWA basée sur S4** avec un **workflow PyTorch en trois étapes** pour l’apprentissage direct et inverse entre géométrie et spectres de transmission optique.

## ✨ En Bref

| Élément | Détails |
|---|---|
| 🎯 Objectif | Prédire la géométrie de métasurface à symétrie C4 à partir de spectres de transmittance cibles |
| 🔬 Physique | Simulation RCWA avec S4 (`../build/S4`) |
| 🧠 Pipeline d’apprentissage | Étape A `shape -> spectra`, Étape B `spectra -> shape`, Étape C `spectra -> shape -> spectra` |
| 📦 Format des données | CSV fusionné (`T@...`, métadonnées de forme) -> NPZ compressé (`uids`, `spectra`, `shapes`) |
| 🧪 Évaluation | Métriques MSE, visualisations qualitatives, vérifications optionnelles par re-simulation S4 |

## 🧭 Workflow De Bout En Bout

1. Générer les réponses optiques des métasurfaces avec S4 (`.lua` + scripts shell de lancement).
2. Fusionner les fichiers CSV bruts de simulation et y rattacher les sommets des polygones.
3. Convertir les CSV fusionnés en NPZ d’entraînement.
4. Entraîner le pipeline de transmittance en trois étapes.
5. Évaluer les checkpoints et visualiser le comportement des étapes A/B/C.
6. Optionnellement, comparer les prédictions des réseaux à de nouvelles simulations S4.

## 🧱 Structure Du Dépôt

```text
.
├── README.md
├── how_to_run.md
├── iccp.yaml
├── pip_requirements.txt
│
├── ms.sh
├── ms_final.sh
├── ms_resume_allargs.sh
├── metasurface_seed.lua
├── metasurface_final.lua
├── metasurface_allargs_resume.lua
│
├── merge_s4_data_full.py
├── three_stage_transmittance.py
├── three_stage_transmittance_evaluation.py
├── FilterShapeS4_Evaluator_Transmittance.py
│
├── results/                # sorties CSV brutes S4
├── shapes/                 # fichiers de sommets des polygones utilisés lors de la fusion
├── merged_csvs/            # jeux de données CSV fusionnés
├── outputs_three_stage_*/  # checkpoints d'entraînement et courbes
├── partial_crys_data/      # tables optiques d’état de cristallisation
│
├── AVIRIS*/                # expériences hyperspectrales secondaires
├── noise_experiment_*/     # branches robustesse/bruit
└── archived/               # scripts historiques et snapshots
```

## 🛠️ Prérequis

- Linux + Bash
- Conda (recommandé)
- Python 3.9
- Binaire S4 disponible à `../build/S4`
- Optionnel: GPU compatible CUDA pour accélérer l’entraînement

## 🚀 Installation

```bash
git clone <your-repo-url> inverse_metasurface
cd inverse_metasurface

conda env create -f iccp.yaml
conda activate iccp

# Verify simulator path expected by scripts
ls -l ../build/S4
```

Optionnel:

```bash
chmod +x ms.sh ms_final.sh ms_resume_allargs.sh
```

## ▶️ Utilisation Pratique

### 1) Générer les données RCWA

`ms_final.sh` et `ms_resume_allargs.sh` lancent chacun 4 jobs S4 en parallèle (`NQ=1..4`):

```bash
./ms_final.sh \
  -ns 10000 \
  -r 12345 \
  -p myrun \
  -g 80 \
  -bo 0.35 \
  -ro 0.30
```

Remarques:
- `ms.sh` est un chemin de lancement plus simple.
- Les scripts de lancement supposent `../build/S4` et utilisent `-t 32`.

### 2) Fusionner les sorties S4 + sommets de formes

```bash
python merge_s4_data_full.py --prefix myrun
# output: merged_s4_shapes_myrun.csv
```

### 3) Normaliser les noms de colonnes pour la compatibilité avec l’entraînement

`merge_s4_data_full.py` écrit `folder_key` / `NQ`, tandis que `three_stage_transmittance.py` attend `prefix` / `nQ`.

```bash
python -c "import pandas as pd; p='merged_s4_shapes_myrun.csv'; df=pd.read_csv(p); df=df.rename(columns={'folder_key':'prefix','NQ':'nQ'}); df.to_csv(p,index=False)"
```

### 4) Prétraiter le CSV vers NPZ

```bash
mkdir -p merged_csvs
mv merged_s4_shapes_myrun.csv merged_csvs/

python three_stage_transmittance.py \
  --preprocess \
  --input_folder merged_csvs \
  --output_npz preprocessed_t_data.npz
```

### 5) Entraîner le pipeline en trois étapes

```bash
python three_stage_transmittance.py \
  --data_npz preprocessed_t_data.npz \
  --num_epochs 100 \
  --batch_size 1024
```

Patron de sortie attendu:
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageA`
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageB`
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageC`

### 6) Évaluer les modèles des étapes A/B/C

```bash
python three_stage_transmittance_evaluation.py \
  --model_dir outputs_three_stage_YYYYMMDD_HHMMSS \
  --data_npz preprocessed_t_data.npz \
  --sample_count 8
```

### 7) Vérification optionnelle de cohérence réseau-vs-S4

```bash
python FilterShapeS4_Evaluator_Transmittance.py \
  --npz_file preprocessed_t_data.npz \
  --spec2shape_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageC/spec2shape_stageC.pt \
  --shape2spec_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageA/shape2spec_stageA.pt \
  --n_samples 4
```

## ⚙️ Principales Options CLI

### Lanceurs S4 (`ms_final.sh`, `ms_resume_allargs.sh`)

| Flag | Signification | Défaut |
|---|---|---|
| `-ns`, `--numshapes` | Nombre de formes à générer | `100000` |
| `-r`, `--seed` | Graine aléatoire | `88888` |
| `-p`, `--prefix` | Préfixe/clé de reprise | `""` |
| `-g`, `--numg` | Paramètre de base/géométrie | `80` |
| `-bo`, `--baseouter` | Décalage de la frontière extérieure de base | `0.25` |
| `-ro`, `--randouter` | Décalage extérieur aléatoire | `0.20` |

### Entraînement (`three_stage_transmittance.py`)

| Flag | Rôle | Défaut |
|---|---|---|
| `--preprocess` | Exécuter le mode prétraitement | `False` |
| `--input_folder` | Dossier contenant les CSV fusionnés | `""` |
| `--output_npz` | Chemin de sortie du NPZ | `preprocessed_data.npz` |
| `--data_npz` | NPZ utilisé pour l’entraînement | `""` |
| `--csv_file` | CSV de secours si le NPZ n’est pas utilisé | `""` |
| `--test` | Mode test (sans entraînement) | `False` |
| `--num_epochs` | Nombre d’époques | `10` |
| `--batch_size` | Taille de batch | `4096` |

### Évaluation (`three_stage_transmittance_evaluation.py`)

| Flag | Rôle | Défaut |
|---|---|---|
| `--model_dir` | Répertoire racine contenant `stageA/B/C` | requis |
| `--data_npz` | Entrée NPZ pour l’évaluation | `""` |
| `--csv_file` | Entrée CSV pour l’évaluation | `""` |
| `--output_dir` | Surcharge du répertoire de sortie | auto dans `model_dir` |
| `--sample_count` | Nombre d’échantillons visualisés | `4` |
| `--seed` | Graine aléatoire pour l’échantillonnage | `23` |
| `--font_scale` | Échelle de police des tracés | `1.0` |
| `--batch_size` | Taille de batch en évaluation | `32` |
| `--plot_only` | Tracer uniquement les courbes | `False` |

### Évaluateur De Cohérence S4 (`FilterShapeS4_Evaluator_Transmittance.py`)

| Flag | Rôle | Défaut |
|---|---|---|
| `--npz_file` | Fichier NPZ prétraité | `preprocessed_t_data.npz` |
| `--spec2shape_ckpt` | Checkpoint de l’étape C | `outputs_three_stage_20250322_145925/stageC/spec2shape_stageC.pt` |
| `--shape2spec_ckpt` | Checkpoint de l’étape A | `outputs_three_stage_20250322_145925/stageA/shape2spec_stageA.pt` |
| `--n_samples` | Échantillons à inspecter | `4` |
| `--seed` | Graine aléatoire | `23` |
| `--max_workers` | Workers S4 en parallèle | `4` |
| `--out_folder` | Dossier de sortie | horodatage auto |

## 🧪 Exécution Rapide (Smoke Test)

```bash
./ms_final.sh -ns 1000 -r 42 -p smoke -g 40 -bo 0.25 -ro 0.20
python merge_s4_data_full.py --prefix smoke
python -c "import pandas as pd; p='merged_s4_shapes_smoke.csv'; d=pd.read_csv(p).rename(columns={'folder_key':'prefix','NQ':'nQ'}); d.to_csv(p,index=False)"
python three_stage_transmittance.py --preprocess --input_folder . --output_npz smoke.npz
python three_stage_transmittance.py --data_npz smoke.npz --num_epochs 5 --batch_size 128
```

## 🔬 Contexte De Recherche

Le cadre central de conception inverse infère la géométrie de métasurface à symétrie C4 à partir de spectres de transmittance sur différents états de cristallisation. Le chemin de transmittance actuel suppose:

- 11 états de cristallisation par échantillon (valeurs `c`, triées au prétraitement)
- 100 bins de longueur d’onde par état (colonnes `T@...`)
- jusqu’à 4 sommets Q1 encodés en `(presence, x, y)`

Ce dépôt inclut également des branches exploratoires (par exemple `AVIRIS*`, `noise_experiment_*` et `archived/`) au-delà du pipeline canonique de transmittance.

## 📚 Citation

Si vous utilisez ce dépôt ou vous appuyez sur ce travail, merci de citer:

```bibtex
@article{chen2025inverse,
  title={Inverse Design of Metasurface for Spectral Imaging},
  author={Chen, Rongzhou and Nie, Haitao and Zhu, Shuo and Zhao, Yaping and Wang, Chutian and Lam, Edmund Y},
  journal={arXiv preprint arXiv:2510.21924},
  year={2025}
}
```

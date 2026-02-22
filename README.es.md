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

Un repositorio de investigación orientado a scripts (referenciado históricamente como `inverse_metasurface`) para el diseño inverso de metasuperficies en imagen espectral. El flujo integra **simulación RCWA basada en S4** con un **pipeline de PyTorch en tres etapas** para el mapeo directo e inverso entre geometría y espectros de transmisión óptica.

## ✨ Resumen Rápido

| Elemento | Detalles |
|---|---|
| 🎯 Objetivo | Predecir la geometría de metasuperficie con simetría C4 a partir de espectros de transmitancia objetivo |
| 🔬 Física | Simulación RCWA con S4 (`../build/S4`) |
| 🧠 Pipeline de aprendizaje | Etapa A `shape -> spectra`, Etapa B `spectra -> shape`, Etapa C `spectra -> shape -> spectra` |
| 📦 Formato de datos | CSV combinados (`T@...`, metadatos de forma) -> NPZ comprimido (`uids`, `spectra`, `shapes`) |
| 🧪 Evaluación | Métricas MSE, gráficas cualitativas y comprobaciones opcionales de re-simulación con S4 |

## 🧭 Flujo de Trabajo de Extremo a Extremo

1. Generar respuestas ópticas de metasuperficie con S4 (`.lua` + lanzadores de shell).
2. Combinar archivos CSV de simulación en bruto y adjuntar vértices de polígonos.
3. Convertir los CSV combinados a NPZ de entrenamiento.
4. Entrenar el pipeline de transmitancia en tres etapas.
5. Evaluar checkpoints y visualizar el comportamiento de las etapas A/B/C.
6. Opcionalmente, comparar predicciones neuronales con simulaciones S4 nuevas.

## 🧱 Estructura del Repositorio

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
├── results/                # S4 raw CSV outputs
├── shapes/                 # polygon vertex files used during merge
├── merged_csvs/            # merged CSV datasets
├── outputs_three_stage_*/  # training checkpoints and curves
├── partial_crys_data/      # crystallization-state optical tables
│
├── AVIRIS*/                # secondary hyperspectral experiments
├── noise_experiment_*/     # robustness/noise branches
└── archived/               # historical scripts and snapshots
```

## 🛠️ Requisitos Previos

- Linux + Bash
- Conda (recomendado)
- Python 3.9
- Binario S4 disponible en `../build/S4`
- Opcional: GPU con CUDA para un entrenamiento más rápido

## 🚀 Instalación

```bash
git clone <your-repo-url> inverse_metasurface
cd inverse_metasurface

conda env create -f iccp.yaml
conda activate iccp

# Verify simulator path expected by scripts
ls -l ../build/S4
```

Opcional:

```bash
chmod +x ms.sh ms_final.sh ms_resume_allargs.sh
```

## ▶️ Uso Práctico

### 1) Generar datos RCWA

`ms_final.sh` y `ms_resume_allargs.sh` lanzan cada uno 4 trabajos S4 en paralelo (`NQ=1..4`):

```bash
./ms_final.sh \
  -ns 10000 \
  -r 12345 \
  -p myrun \
  -g 80 \
  -bo 0.35 \
  -ro 0.30
```

Notas:
- `ms.sh` es una ruta de lanzamiento más simple.
- Los lanzadores asumen `../build/S4` y usan `-t 32`.

### 2) Combinar salidas S4 + vértices de forma

```bash
python merge_s4_data_full.py --prefix myrun
# output: merged_s4_shapes_myrun.csv
```

### 3) Normalizar nombres de columnas para compatibilidad de entrenamiento

`merge_s4_data_full.py` escribe `folder_key` / `NQ`, mientras `three_stage_transmittance.py` espera `prefix` / `nQ`.

```bash
python -c "import pandas as pd; p='merged_s4_shapes_myrun.csv'; df=pd.read_csv(p); df=df.rename(columns={'folder_key':'prefix','NQ':'nQ'}); df.to_csv(p,index=False)"
```

### 4) Preprocesar CSV a NPZ

```bash
mkdir -p merged_csvs
mv merged_s4_shapes_myrun.csv merged_csvs/

python three_stage_transmittance.py \
  --preprocess \
  --input_folder merged_csvs \
  --output_npz preprocessed_t_data.npz
```

### 5) Entrenar el pipeline de tres etapas

```bash
python three_stage_transmittance.py \
  --data_npz preprocessed_t_data.npz \
  --num_epochs 100 \
  --batch_size 1024
```

Patrón de salida esperado:
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageA`
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageB`
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageC`

### 6) Evaluar modelos de las etapas A/B/C

```bash
python three_stage_transmittance_evaluation.py \
  --model_dir outputs_three_stage_YYYYMMDD_HHMMSS \
  --data_npz preprocessed_t_data.npz \
  --sample_count 8
```

### 7) Comprobación opcional de consistencia red neuronal vs S4

```bash
python FilterShapeS4_Evaluator_Transmittance.py \
  --npz_file preprocessed_t_data.npz \
  --spec2shape_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageC/spec2shape_stageC.pt \
  --shape2spec_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageA/shape2spec_stageA.pt \
  --n_samples 4
```

## ⚙️ Opciones Clave de CLI

### Lanzadores S4 (`ms_final.sh`, `ms_resume_allargs.sh`)

| Flag | Significado | Valor por defecto |
|---|---|---|
| `-ns`, `--numshapes` | Número de formas a generar | `100000` |
| `-r`, `--seed` | Semilla aleatoria | `88888` |
| `-p`, `--prefix` | Prefijo/clave de reanudación | `""` |
| `-g`, `--numg` | Parámetro de base/geometría | `80` |
| `-bo`, `--baseouter` | Desplazamiento base del borde exterior | `0.25` |
| `-ro`, `--randouter` | Desplazamiento aleatorio del borde exterior | `0.20` |

### Entrenamiento (`three_stage_transmittance.py`)

| Flag | Propósito | Valor por defecto |
|---|---|---|
| `--preprocess` | Ejecutar modo de preprocesamiento | `False` |
| `--input_folder` | Carpeta con archivos CSV combinados | `""` |
| `--output_npz` | Ruta del NPZ de salida | `preprocessed_data.npz` |
| `--data_npz` | NPZ usado para entrenamiento | `""` |
| `--csv_file` | CSV alternativo si no se usa NPZ | `""` |
| `--test` | Modo prueba (omite entrenamiento) | `False` |
| `--num_epochs` | Número de épocas | `10` |
| `--batch_size` | Tamaño de lote | `4096` |

### Evaluación (`three_stage_transmittance_evaluation.py`)

| Flag | Propósito | Valor por defecto |
|---|---|---|
| `--model_dir` | Directorio raíz que contiene `stageA/B/C` | required |
| `--data_npz` | Entrada NPZ para evaluación | `""` |
| `--csv_file` | Entrada CSV para evaluación | `""` |
| `--output_dir` | Sobrescritura del directorio de salida | auto in `model_dir` |
| `--sample_count` | Número de muestras visualizadas | `4` |
| `--seed` | Semilla aleatoria para muestreo | `23` |
| `--font_scale` | Escala de fuente para gráficas | `1.0` |
| `--batch_size` | Tamaño de lote en evaluación | `32` |
| `--plot_only` | Solo graficar curvas | `False` |

### Evaluador de consistencia S4 (`FilterShapeS4_Evaluator_Transmittance.py`)

| Flag | Propósito | Valor por defecto |
|---|---|---|
| `--npz_file` | Archivo NPZ preprocesado | `preprocessed_t_data.npz` |
| `--spec2shape_ckpt` | Checkpoint de la etapa C | `outputs_three_stage_20250322_145925/stageC/spec2shape_stageC.pt` |
| `--shape2spec_ckpt` | Checkpoint de la etapa A | `outputs_three_stage_20250322_145925/stageA/shape2spec_stageA.pt` |
| `--n_samples` | Muestras a inspeccionar | `4` |
| `--seed` | Semilla aleatoria | `23` |
| `--max_workers` | Workers S4 en paralelo | `4` |
| `--out_folder` | Carpeta de salida | auto timestamp |

## 🧪 Ejecución Rápida de Verificación

```bash
./ms_final.sh -ns 1000 -r 42 -p smoke -g 40 -bo 0.25 -ro 0.20
python merge_s4_data_full.py --prefix smoke
python -c "import pandas as pd; p='merged_s4_shapes_smoke.csv'; d=pd.read_csv(p).rename(columns={'folder_key':'prefix','NQ':'nQ'}); d.to_csv(p,index=False)"
python three_stage_transmittance.py --preprocess --input_folder . --output_npz smoke.npz
python three_stage_transmittance.py --data_npz smoke.npz --num_epochs 5 --batch_size 128
```

## 🔬 Contexto de Investigación

El escenario central de diseño inverso infiere geometrías de metasuperficie con simetría C4 a partir de espectros de transmitancia a través de distintos estados de cristalización. La ruta actual de transmitancia asume:

- 11 estados de cristalización por muestra (valores `c`, ordenados durante el preprocesamiento)
- 100 bins de longitud de onda por estado (columnas `T@...`)
- hasta 4 vértices Q1 codificados como `(presence, x, y)`

Este repositorio también incluye ramas exploratorias (por ejemplo `AVIRIS*`, `noise_experiment_*` y `archived/`) más allá del pipeline canónico de transmitancia.

## 📚 Citación

Si usas este repositorio o te basas en este trabajo, por favor cita:

```bibtex
@article{chen2025inverse,
  title={Inverse Design of Metasurface for Spectral Imaging},
  author={Chen, Rongzhou and Nie, Haitao and Zhu, Shuo and Zhao, Yaping and Wang, Chutian and Lam, Edmund Y},
  journal={arXiv preprint arXiv:2510.21924},
  year={2025}
}
```

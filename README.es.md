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

[![Status](https://img.shields.io/badge/Status-Research%20Prototype-orange)](#-alcance-del-proyecto)
[![Python](https://img.shields.io/badge/Python-3.9-3776AB?logo=python&logoColor=white)](#-configuración-del-entorno)
[![Platform](https://img.shields.io/badge/Platform-Linux-2f2f2f?logo=linux&logoColor=white)](#-requisitos-previos)
[![RCWA](https://img.shields.io/badge/RCWA-S4%20Required-1f9d55)](#-requisitos-previos)
[![PyTorch](https://img.shields.io/badge/Framework-PyTorch-EE4C2C?logo=pytorch&logoColor=white)](#-entrenamiento-y-evaluación)
[![License](https://img.shields.io/badge/License-Not%20Specified-lightgrey)](#-licencia)

Un repositorio de investigación centrado en scripts para el **diseño inverso de metasuperficies** bajo simetría C4.
Combina:

- 🔬 Simulación S4/RCWA (`.lua` + launchers Bash)
- 🧱 Fusión y preprocesamiento de datos desde espectros crudos + vértices de forma
- 🧠 Entrenamiento PyTorch en tres etapas (`shape -> spectra`, `spectra -> shape`, ajuste en cadena)
- 📊 Evaluación y visualización, incluyendo comprobaciones de consistencia red neuronal vs S4

## 📌 Tabla de Contenidos

- [Alcance del Proyecto](#-alcance-del-proyecto)
- [Contexto de Investigación](#-contexto-de-investigación)
- [Estructura del Repositorio](#-estructura-del-repositorio)
- [Requisitos Previos](#-requisitos-previos)
- [Configuración del Entorno](#-configuración-del-entorno)
- [Inicio Rápido](#-inicio-rápido)
- [Pipeline de Extremo a Extremo](#-pipeline-de-extremo-a-extremo)
- [Entrenamiento y Evaluación](#-entrenamiento-y-evaluación)
- [Opciones CLI Clave](#-opciones-cli-clave)
- [Solución de Problemas](#-solución-de-problemas)
- [Hoja de Ruta](#-hoja-de-ruta)
- [Citación](#-citación)
- [Licencia](#-licencia)

## 🎯 Alcance del Proyecto

Este repositorio está muy orientado a experimentación y se centra en scripts ejecutables (no en una librería empaquetada).
El flujo de trabajo más estable es:

1. Ejecutar S4 para generar salidas ópticas crudas en `results/` y formas en `shapes/`
2. Fusionar y pivotar archivos CSV por ejecución en datos tabulares listos para entrenamiento
3. Convertir CSV fusionados en `.npz` comprimido
4. Entrenar un pipeline de transmitancia en 3 etapas
5. Evaluar checkpoints y exportar gráficas/métricas

## 🧪 Contexto de Investigación

### Planteamiento del problema

La tarea central de diseño inverso consiste en recuperar la geometría de la metasuperficie a partir de espectros de transmisión objetivo (y viceversa), con restricciones de simetría C4 y barridos de cristalización parcial.

### Supuestos de datos en el pipeline de transmitancia

| Ítem | Valor |
|---|---|
| Estados de cristalización por forma | 11 (`c` de `0.0` a `1.0`) |
| Bins espectrales por estado | 100 |
| Tensor de espectro por muestra | `11 x 100` |
| Tensor de forma por muestra | `4 x 3` (`[presence, x, y]`) |

### Objetivos de aprendizaje en tres etapas

| Etapa | Dirección | Checkpoint típico |
|---|---|---|
| A | `shape -> spectrum` | `stageA/shape2spec_stageA.pt` |
| B | `spectrum -> shape` | `stageB/spec2shape_stageB.pt` |
| C | `spectrum -> shape -> spectrum` (ajustado en cadena) | `stageC/spec2shape_stageC.pt` |

## 🗂️ Estructura del Repositorio

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

## 🧩 Requisitos Previos

| Dependencia | Notas |
|---|---|
| Linux + Bash | Los scripts de shell asumen rutas estilo Linux |
| Conda | Gestión de entorno recomendada (`iccp.yaml`) |
| Python 3.9 | Entorno principal para scripts de entrenamiento/evaluación |
| Binario S4 | Se espera en `../build/S4` relativo a la raíz del repo |
| CUDA (opcional) | Acelera entrenamiento y evaluación |

## ⚙️ Configuración del Entorno

```bash
git clone <repo-url> inverse_metasurface
cd inverse_metasurface

conda env create -f iccp.yaml
conda activate iccp

# verify S4 path expected by shell runners
ls -l ../build/S4
```

Permiso de ejecución opcional para los launchers de shell:

```bash
chmod +x ms.sh ms_final.sh ms_resume_allargs.sh ms_resume.sh
```

## 🚀 Inicio Rápido

Si ya tienes `preprocessed_t_data.npz`:

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

## 🔁 Pipeline de Extremo a Extremo

### 1) Generar datos S4

```bash
./ms_final.sh \
  -ns 10000 \
  -r 12345 \
  -p myrun \
  -g 80 \
  -bo 0.35 \
  -ro 0.30
```

### 2) Fusionar salidas S4 + vértices de forma

```bash
python merge_s4_data_full.py --prefix myrun
# -> merged_s4_shapes_myrun.csv
```

### 3) Normalizar nombres de columnas CSV para el preprocesamiento

El preprocesamiento en `three_stage_transmittance.py` espera `prefix` y `nQ`, mientras que la salida fusionada puede contener `folder_key` y `NQ`.

```bash
python -c "import pandas as pd; p='merged_s4_shapes_myrun.csv'; df=pd.read_csv(p); df=df.rename(columns={'folder_key':'prefix','NQ':'nQ'}); df.to_csv(p,index=False)"
```

### 4) Preprocesamiento CSV -> NPZ

```bash
mkdir -p merged_csvs
mv merged_s4_shapes_myrun.csv merged_csvs/

python three_stage_transmittance.py --preprocess \
  --input_folder merged_csvs \
  --output_npz preprocessed_t_data.npz
```

### 5) Entrenar las tres etapas

```bash
python three_stage_transmittance.py \
  --data_npz preprocessed_t_data.npz \
  --num_epochs 100 \
  --batch_size 1024
```

### 6) Evaluar y graficar

```bash
python three_stage_transmittance_evaluation.py \
  --model_dir outputs_three_stage_YYYYMMDD_HHMMSS \
  --data_npz preprocessed_t_data.npz \
  --sample_count 8
```

### 7) Opcional: comparación red neuronal vs S4

```bash
python FilterShapeS4_Evaluator_Transmittance.py \
  --npz_file preprocessed_t_data.npz \
  --spec2shape_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageC/spec2shape_stageC.pt \
  --shape2spec_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageA/shape2spec_stageA.pt \
  --n_samples 4
```

## 🧠 Entrenamiento y Evaluación

### Scripts principales

| Script | Propósito |
|---|---|
| `three_stage_transmittance.py` | preprocesar + entrenar etapas A/B/C |
| `three_stage_transmittance_evaluation.py` | evaluar checkpoints, calcular métricas, guardar gráficas |
| `FilterShapeS4_Evaluator_Transmittance.py` | comparar predicciones aprendidas con comportamiento S4 |

### Salidas típicas

| Artefacto | Ubicación |
|---|---|
| Checkpoints por etapa | `outputs_three_stage_*/stageA|stageB|stageC/` |
| Figuras de evaluación | `outputs_three_stage_*/evaluation_<timestamp>/` |
| CSV de métricas | `evaluation_metrics.csv`, `metrics_summary.csv` |

## 🛠️ Opciones CLI Clave

### Launchers S4 (`ms_final.sh`, `ms_resume_allargs.sh`)

| Flag | Significado | Predeterminado |
|---|---|---|
| `-ns`, `--numshapes` | número de formas | `100000` |
| `-r`, `--seed` | semilla aleatoria | `88888` |
| `-p`, `--prefix` | prefijo de ejecución/clave de reanudación | vacío |
| `-g`, `--numg` | parámetro de base geométrica | `80` |
| `-bo`, `--baseouter` | desplazamiento externo base | `0.25` |
| `-ro`, `--randouter` | desplazamiento externo aleatorio | `0.20` |

### Entrenamiento (`three_stage_transmittance.py`)

| Flag | Significado | Predeterminado |
|---|---|---|
| `--preprocess` | cambia a modo preprocesamiento | off |
| `--input_folder` | carpeta con CSV fusionados | `""` |
| `--output_npz` | archivo de salida del preprocesamiento | `preprocessed_data.npz` |
| `--data_npz` | entrada NPZ para entrenamiento | `""` |
| `--csv_file` | entrada CSV directa para entrenamiento | `""` |
| `--test` | activar modo test | off |
| `--num_epochs` | épocas por etapa | `10` |
| `--batch_size` | tamaño de batch | `4096` |

### Evaluación (`three_stage_transmittance_evaluation.py`)

| Flag | Significado | Predeterminado |
|---|---|---|
| `--model_dir` | directorio de ejecución entrenada (requerido) | - |
| `--data_npz` | entrada NPZ para evaluación | `""` |
| `--csv_file` | entrada CSV para evaluación | `""` |
| `--output_dir` | directorio de salida personalizado | auto |
| `--sample_count` | número de muestras visualizadas | `4` |
| `--seed` | semilla aleatoria para selección de muestras | `23` |
| `--font_scale` | multiplicador de fuente en gráficas | `1.0` |
| `--batch_size` | batch size del dataloader de evaluación | `32` |
| `--plot_only` | regenerar solo gráficas | off |

## 🧯 Solución de Problemas

| Síntoma | Causa probable | Solución |
|---|---|---|
| `../build/S4: No such file or directory` | Binario S4 no está en la ruta relativa esperada | Coloca/compila S4 en `../build/S4` o edita los scripts |
| `Must specify either --data_npz or --csv_file` | Falta argumento de dataset para entrenamiento/evaluación | Proporciona exactamente una entrada de dataset |
| `No transmission columns found` | El CSV fusionado no tiene columnas `T@...` | Reejecuta merge/pivot y verifica encabezados |
| `KeyError: 'prefix'` in preprocess | La salida de merge aún usa `folder_key`/`NQ` | Renombra columnas a `prefix`/`nQ` antes del preprocesamiento |
| GPU OOM | Batch demasiado grande | Reduce `--batch_size` |
| Missing checkpoints during eval | Faltan checkpoints de etapas o ruta incorrecta | Verifica archivos stageA/B/C bajo `--model_dir` seleccionado |

## 🧭 Hoja de Ruta

- Estandarizar el esquema del CSV fusionado en todos los scripts de merge (nombres `prefix`, `nQ`)
- Añadir tests automatizados para merge/preprocesamiento/carga de checkpoints
- Proveer un único entrypoint CLI para orquestar el pipeline completo
- Añadir manifiestos de datasets/ejecuciones para reproducibilidad
- Añadir una licencia open source explícita

## 📚 Citación

Si este repositorio contribuye a tu investigación, por favor cita:

```bibtex
@article{chen2025inverse,
  title={Inverse Design of Metasurface for Spectral Imaging},
  author={Chen, Rongzhou and Nie, Haitao and Zhu, Shuo and Zhao, Yaping and Wang, Chutian and Lam, Edmund Y},
  journal={arXiv preprint arXiv:2510.21924},
  year={2025}
}
```

## 📄 Licencia

Actualmente no hay un archivo `LICENSE` en este repositorio. Por tanto, los derechos de uso y redistribución no están especificados hasta que se agregue una licencia.

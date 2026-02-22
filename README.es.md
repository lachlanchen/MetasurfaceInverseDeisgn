English | 中文繁體 | 中文简体 | 日本語 | 한국어 | Tiếng Việt | العربية | Français | Español | Deutsch | Русский

# inverse_metasurface

<div align="center">

[![Status](https://img.shields.io/badge/status-experimental-orange)](#-roadmap)
[![Python](https://img.shields.io/badge/python-3.9-blue)](#-prerequisites)
[![Platform](https://img.shields.io/badge/platform-linux-lightgrey)](#-prerequisites)
[![S4](https://img.shields.io/badge/S4-required-success)](#-prerequisites)
[![License](https://img.shields.io/badge/license-unset-lightgrey)](#-license)

</div>

Un entorno de investigación para el **diseño inverso de metasuperficies** usando **simulación S4/RCWA** y **modelos neuronales de múltiples etapas**.

Incluye:
- Generación de datos S4 de alto rendimiento para metasuperficies poligonales con simetría C4.
- Fusión de datos + preprocesamiento en NPZ listo para entrenamiento.
- Aprendizaje en tres etapas: **forma -> espectro**, **espectro -> forma**, **ajuste en cadena espectro -> forma -> espectro**.
- Evaluación y comparación opcional directa **neuronal-vs-S4**.

## 🌟 Resumen General

| Área | Qué ofrece este repositorio |
|---|---|
| Simulación física | Scripts Bash + Lua para ejecutar S4 en paralelo en `nQ=1..4` |
| Herramientas de dataset | Scripts de fusión que adjuntan vértices de polígonos y reorganizan espectros |
| Pipeline de ML | `three_stage_transmittance.py` con modos de preprocesamiento + entrenamiento |
| Evaluación | `three_stage_transmittance_evaluation.py` con métricas + figuras |
| Rama de investigación | Experimentos AVIRIS/hiperespectrales y de ruido/compresión |

## 🧠 Contexto de Investigación

Este repositorio está orientado al diseño inverso de metasuperficies fotónicas: inferir geometría a partir de espectros objetivo (y viceversa).

Suposiciones centrales usadas por el pipeline base:
- Simetría C4 mediante parametrización de puntos Q1.
- 11 estados de cristalización (`c` en `[0.0, 1.0]`).
- El espectro de cada muestra se almacena como filas de transmisión `11 x 100`.
- Las formas se representan como hasta 4 puntos Q1 (tensor `4 x 3`: presencia, x, y).

Lógica de entrenamiento en tres etapas:
1. **Etapa A**: forma -> espectro (`shape2spec_stageA.pt`)
2. **Etapa B**: espectro -> forma (`spec2shape_stageB.pt`)
3. **Etapa C**: ajuste fino en cadena espectro -> forma -> espectro (`spec2shape_stageC.pt`)

## 🗂 Estructura del Proyecto

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
├─ partial_crys_data/                # constantes ópticas por nivel de cristalización
├─ results/                          # salidas crudas de S4 (normalmente sin seguimiento)
├─ shapes/                           # archivos de vértices de polígonos generados
├─ merged_csvs/                      # tablas fusionadas usadas para preprocesamiento
├─ outputs_three_stage_*/            # artefactos del modelo por ejecución
├─ AVIRIS*/ + aviris_*.py            # rama hiperespectral
├─ how_to_run.md / commands*.md
├─ iccp.yaml
└─ pip_requirements.txt
```

## ✅ Requisitos Previos

| Requisito | Notas |
|---|---|
| SO | Linux (los scripts asumen Bash + rutas Linux) |
| Python | 3.9 (según `iccp.yaml`) |
| Gestor de entornos | Se recomienda Conda |
| Binario S4 | Se espera en `../build/S4` relativo a la raíz del repositorio |
| GPU (opcional) | CUDA acelera entrenamiento/evaluación |

## ⚙️ Instalación

### 1) Clonar y entrar

```bash
git clone <your-repo-url> inverse_metasurface
cd inverse_metasurface
```

### 2) Crear entorno

```bash
conda env create -f iccp.yaml
conda activate iccp
```

### 3) Verificar ruta de S4

```bash
ls -l ../build/S4
```

Si esta ruta no existe, compila/ubica S4 allí o ajusta las rutas en los scripts.

## 🚀 Inicio Rápido (Extremo a Extremo)

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

## 🧪 Detalles de Uso

### A) Generación con S4

Ejecución mínima con semilla:

```bash
./ms.sh -ns 10000 -r 88888
```

Ejecución parametrizada:

```bash
./ms_final.sh \
  -ns 100000 \
  -r 88888 \
  -p iccpOv100kG80 \
  -g 80 \
  -bo 0.35 \
  -ro 0.30
```

Ejecución de tipo reanudación:

```bash
./ms_resume_allargs.sh \
  -ns 100000 \
  -r 88888 \
  -p iccpOv100kG80 \
  -g 80 \
  -bo 0.35 \
  -ro 0.30
```

### B) Fusionar CSVs crudos de S4

```bash
python merge.py --prefix 20250123_155420
```

Utilidad alternativa:

```bash
python merge_s4_data_full.py --prefix 20250123_155420
```

### C) Preprocesar CSVs fusionados -> NPZ

```bash
python three_stage_transmittance.py --preprocess \
  --input_folder merged_csvs \
  --output_npz preprocessed_t_data.npz
```

### D) Entrenamiento

```bash
python three_stage_transmittance.py \
  --data_npz preprocessed_t_data.npz \
  --num_epochs 100 \
  --batch_size 1024
```

Las salidas de entrenamiento se guardan en:
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageA`
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageB`
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageC`

### E) Evaluación

```bash
python three_stage_transmittance_evaluation.py \
  --model_dir outputs_three_stage_YYYYMMDD_HHMMSS \
  --data_npz preprocessed_t_data.npz \
  --sample_count 8
```

Genera:
- `evaluation_metrics.csv`
- `metrics_summary.csv`
- visualizaciones por etapa (`.png`, `.pdf`)
- gráficas de curvas de entrenamiento

### F) Comparación neuronal vs S4 directo (opcional)

```bash
python FilterShapeS4_Evaluator_Transmittance.py \
  --npz_file preprocessed_t_data.npz \
  --spec2shape_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageC/spec2shape_stageC.pt \
  --shape2spec_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageA/shape2spec_stageA.pt \
  --n_samples 4
```

## 🔧 Opciones CLI Clave

### Banderas del ejecutor S4 (`ms_final.sh`, `ms_resume_allargs.sh`)

| Flag | Significado | Valor por defecto |
|---|---|---|
| `-ns`, `--numshapes` | Número de formas | `100000` |
| `-r`, `--seed` | Semilla aleatoria | `88888` |
| `-p`, `--prefix` | Prefijo de ejecución / clave de reanudación | vacío |
| `-g`, `--numg` | Parámetro de geometría/base | `80` |
| `-bo`, `--baseouter` | Desplazamiento exterior base | `0.25` |
| `-ro`, `--randouter` | Desplazamiento exterior aleatorio | `0.20` |

### Banderas de entrenamiento (`three_stage_transmittance.py`)

| Flag | Significado | Valor por defecto |
|---|---|---|
| `--preprocess` | Ejecutar modo de preprocesamiento | off |
| `--input_folder` | Carpeta de entrada CSV | vacío |
| `--output_npz` | Ruta del NPZ de salida | `preprocessed_data.npz` |
| `--data_npz` | NPZ usado para entrenamiento/evaluación | vacío |
| `--csv_file` | CSV usado directamente | vacío |
| `--num_epochs` | Épocas por etapa | `10` |
| `--batch_size` | Tamaño de lote | `4096` |
| `--test` | Marcador de modo test | off |

### Banderas de evaluación (`three_stage_transmittance_evaluation.py`)

| Flag | Significado | Valor por defecto |
|---|---|---|
| `--model_dir` | Directorio raíz de la ejecución entrenada | requerido |
| `--data_npz` | NPZ para evaluación | vacío |
| `--csv_file` | CSV para evaluación | vacío |
| `--output_dir` | Carpeta de salida | `model_dir/evaluation_<timestamp>` |
| `--sample_count` | Número de muestras visualizadas | `4` |
| `--seed` | Semilla de muestreo | `23` |
| `--font_scale` | Escala de fuente para gráficas | `1.0` |
| `--batch_size` | Tamaño de lote en evaluación | `32` |
| `--plot_only` | Solo graficar curvas | off |

## 🧭 Solución de Problemas

| Síntoma | Causa probable | Solución |
|---|---|---|
| `../build/S4: No such file or directory` | Ruta del binario S4 no coincide | Compila/ubica S4 en `../build/S4` o ajusta scripts |
| `Must specify either --data_npz or --csv_file` | Falta fuente de dataset | Pasa explícitamente una de esas banderas |
| `No matching CSVs found in 'results/'` | Prefijo no coincide | Verifica el prefijo y el nombre de salida |
| Muy pocos/cero registros preprocesados | Filas `T@...` o `vertices_str` faltantes/inválidas | Valida el esquema del CSV fusionado y la agrupación por forma |
| CUDA OOM | Lote demasiado grande | Reduce `--batch_size` (p. ej., `1024 -> 256`) |

## 🧱 Notas de Desarrollo

- Este es un repositorio con muchos experimentos; varios artefactos generados se dejan sin seguimiento intencionalmente.
- Existen varias variantes de scripts para tareas similares (`merge_*`, `aviris_*`, `noise_experiment_*`).
- El flujo base de metasuperficie inversa es el que se documenta en este README.
- Los scripts de entrenamiento fijan semillas (`42`), pero el determinismo estricto sigue dependiendo del hardware/backend.
- Actualmente no hay CI unificado ni una suite completa de pruebas automatizadas.

## 🛣 Hoja de Ruta

- Añadir un dataset canónico compacto para pruebas rápidas de humo.
- Consolidar scripts de pipeline duplicados.
- Añadir verificaciones de integridad en merge/preprocesamiento y carga de checkpoints.
- Añadir CI para lint + smoke train/eval.
- Documentar de forma más explícita la compilación de S4 y el versionado fijado.

## 🤝 Contribución

1. Crea una rama de feature.
2. Mantén el alcance del PR acotado (una preocupación de pipeline/experimento por PR).
3. Incluye comandos ejecutables exactos y rutas de salida esperadas.
4. Evita commitear artefactos generados grandes, salvo que sea necesario.
5. Añade notas de reproducibilidad (semilla, fuente de datos, ruta de checkpoint).

## 📄 Licencia

Actualmente no hay un archivo `LICENSE` en este repositorio.

Hasta que se agregue un archivo de licencia, los términos de reutilización y redistribución deben considerarse **indeterminados**.

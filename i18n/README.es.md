[English](../README.md) · [العربية](README.ar.md) · [Español](README.es.md) · [Français](README.fr.md) · [日本語](README.ja.md) · [한국어](README.ko.md) · [Tiếng Việt](README.vi.md) · [中文 (简体)](README.zh-Hans.md) · [中文（繁體）](README.zh-Hant.md) · [Deutsch](README.de.md) · [Русский](README.ru.md)


# Diseño Inverso de Metasuperficies para Imagen Espectral

<p align="center">
  <img alt="Status" src="https://img.shields.io/badge/Status-Research%20Prototype-f59e0b?style=for-the-badge">
  <img alt="Python" src="https://img.shields.io/badge/Python-3.9-3776AB?style=for-the-badge&logo=python&logoColor=white">
  <img alt="PyTorch" src="https://img.shields.io/badge/PyTorch-2.x-EE4C2C?style=for-the-badge&logo=pytorch&logoColor=white">
  <img alt="Simulator" src="https://img.shields.io/badge/Simulator-S4%20RCWA-16a34a?style=for-the-badge">
  <img alt="Platform" src="https://img.shields.io/badge/Platform-Linux%20%2B%20Bash-6b7280?style=for-the-badge&logo=gnu-bash&logoColor=white">
  <img alt="ArXiv" src="https://img.shields.io/badge/arXiv-2510.21924-b31b1b?style=for-the-badge&logo=arxiv&logoColor=white">
</p>

Un repositorio de investigación orientado a scripts (históricamente llamado `inverse_metasurface`) para diseño inverso de metasuperficies en imagen espectral.

El flujo de trabajo principal acopla:
- Simulación RCWA basada en física (`S4` + Lua)
- Ensamblado de datos y asociación de formas
- Aprendizaje PyTorch en tres etapas (`shape -> spectrum`, `spectrum -> shape`, ajuste fino encadenado)
- Evaluación cuantitativa y cualitativa, con comprobaciones opcionales de consistencia red neuronal vs S4

> [!IMPORTANT]
> El comportamiento canónico y los comandos se conservan de los scripts/documentación existentes del proyecto. Cuando las referencias históricas apuntan a archivos ausentes, esas referencias se mantienen intencionalmente con notas explícitas por compatibilidad.

## 📑 Contenidos

- [✨ Vista general rápida](#-vista-general-rápida)
- [🌍 Internacionalización (i18n)](#-internacionalización-i18n)
- [✨ Características](#-características)
- [🧭 Flujo de trabajo de extremo a extremo](#-flujo-de-trabajo-de-extremo-a-extremo)
- [🧱 Estructura del proyecto](#-estructura-del-proyecto)
- [🛠️ Requisitos previos](#️-requisitos-previos)
- [🚀 Instalación](#-instalación)
- [▶️ Uso](#️-uso)
- [⚙️ Configuración](#️-configuración)
- [🧪 Ejemplos](#-ejemplos)
- [🔬 Contexto de investigación](#-contexto-de-investigación)
- [🧑‍💻 Notas de desarrollo](#-notas-de-desarrollo)
- [🧯 Solución de problemas](#-solución-de-problemas)
- [🗺️ Hoja de ruta](#️-hoja-de-ruta)
- [🤝 Contribución](#-contribución)
- [📄 Licencia](#-licencia)
- [📚 Citación](#-citación)

## ✨ Vista general rápida

| Elemento | Detalles |
|---|---|
| 🎯 Tarea principal | Inferir geometría de metasuperficie con simetría C4 a partir de espectros objetivo de transmitancia |
| 🔬 Simulador | `../build/S4` invocado por lanzadores de shell y scripts `.lua` |
| 🧠 Pipeline de aprendizaje | Etapa A `shape -> spectra`, Etapa B `spectra -> shape`, Etapa C `spectra -> shape -> spectra` |
| 📦 Contrato de datos | CSV fusionado (`T@...`, metadatos, `vertices_str`) -> NPZ comprimido (`uids`, `spectra`, `shapes`) |
| 🧪 Evaluación | Métricas MSE, visualizaciones por etapa, re-simulación S4 opcional |
| 🌐 Estado de i18n | README multilingüe en la raíz + directorio `i18n/` existente |

## 🌍 Internacionalización (i18n)

- Los README multilingües se mantienen en la raíz del repositorio como archivos `README.<lang>.md`.
- El directorio `i18n/` existe en esta instantánea del repositorio.
- Este archivo mantiene una única línea de opciones de idioma en la parte superior para evitar barras de idioma duplicadas.
- `README.en.md` también existe en el repositorio; este `README.md` sigue siendo la base canónica para esta pasada de actualización.

## ✨ Características

- Ruta completa de diseño inverso desde la salida de simulación S4 hasta el modelo inverso entrenado.
- Parametrización de polígonos con simetría C4 y codificación de puntos Q1 (`4x3`: `presence, x, y`).
- Entrenamiento de modelos en tres etapas en un único script (`three_stage_transmittance.py`).
- Herramientas de fusión que conservan la precisión espectral y adjuntan vértices por forma.
- Evaluador opcional que compara predicciones aprendidas con simulación S4 nueva.
- Amplias ramas exploratorias (AVIRIS, SWIR/ruido, GSST, variantes archivadas/obsoletas).

## 🧭 Flujo de trabajo de extremo a extremo

1. Generar salidas de simulación en `results/` y archivos de polígonos en `shapes/`.
2. Fusionar archivos CSV de S4 y adjuntar vértices de forma.
3. Normalizar nombres de columnas fusionadas para compatibilidad con entrenamiento.
4. Preprocesar CSV fusionados a tensores NPZ.
5. Entrenar modelos de las etapas A/B/C.
6. Evaluar checkpoints y visualizar comportamiento.
7. Opcionalmente comparar espectros de forma predicha frente a nuevas ejecuciones de S4.

## 🧱 Estructura del proyecto

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

## 🛠️ Requisitos previos

| Dependencia | Notas |
|---|---|
| Linux + Bash | Los scripts lanzadores apuntan a ejecución en shell |
| Python 3.9 | Coincide con `iccp.yaml` (`python=3.9.18`) |
| Conda | Recomendado para reproducibilidad |
| Binario S4 | Se espera en `../build/S4` |
| GPU CUDA (opcional) | Acelera entrenamiento/evaluación |

## 🚀 Instalación

### 1) Clonar y entrar

```bash
git clone <your-repo-url> inverse_metasurface
cd inverse_metasurface
```

### 2) Crear entorno (recomendado)

```bash
conda env create -f iccp.yaml
conda activate iccp
```

Nota alternativa:

```bash
# Historical README reference (file may be absent in this snapshot)
pip install -r pip_requirements.txt
```

### 3) Verificar la ruta del simulador esperada por los scripts

```bash
ls -l ../build/S4
```

### 4) (Opcional) hacer ejecutables los lanzadores

```bash
chmod +x ms.sh ms_final.sh ms_resume.sh ms_resume_allargs.sh ms_resume_random_state.sh ms_resume_random_state_nir.sh
```

## ▶️ Uso

### A) Generar datos de simulación RCWA

Lanzador simple:

```bash
./ms.sh -ns 10000 -r 12345
```

Lanzador parametrizado:

```bash
./ms_final.sh \
  -ns 10000 \
  -r 12345 \
  -p myrun \
  -g 80 \
  -bo 0.35 \
  -ro 0.30
```

Lanzador orientado a reanudación:

```bash
./ms_resume_allargs.sh \
  -ns 10000 \
  -r 12345 \
  -p myrun \
  -g 80 \
  -bo 0.35 \
  -ro 0.30
```

Ejemplo adicional de reanudación/estado aleatorio (de la documentación de comandos):

```bash
./ms_resume_random_state.sh \
  -p iccp100kG20Ov \
  -r 88888 \
  -g 20 \
  -bo 0.35 \
  -ro 0.3 \
  -ns 100000
```

Notas:
- Los lanzadores ejecutan `NQ=1..4` en paralelo.
- Los scripts llaman a `../build/S4` con `-t 32`.

### B) Fusionar salidas de S4 y adjuntar vértices de forma

```bash
python merge_s4_data_full.py --prefix myrun
# output: merged_s4_shapes_myrun.csv
```

### C) Normalizar columnas para compatibilidad con entrenamiento

`merge_s4_data_full.py` escribe `folder_key` y `NQ`, mientras que la ruta de entrenamiento espera `prefix` y `nQ`.

```bash
python -c "import pandas as pd; p='merged_s4_shapes_myrun.csv'; df=pd.read_csv(p); df=df.rename(columns={'folder_key':'prefix','NQ':'nQ'}); df.to_csv(p,index=False)"
```

### D) Preprocesar CSV -> NPZ

```bash
mkdir -p merged_csvs
mv merged_s4_shapes_myrun.csv merged_csvs/

python three_stage_transmittance.py \
  --preprocess \
  --input_folder merged_csvs \
  --output_npz preprocessed_t_data.npz
```

### E) Entrenar Etapas A/B/C

```bash
python three_stage_transmittance.py \
  --data_npz preprocessed_t_data.npz \
  --num_epochs 100 \
  --batch_size 1024
```

Las salidas se escriben en:
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageA`
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageB`
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageC`

### F) Evaluar modelos entrenados

Comando del README histórico (el nombre del script se conserva por compatibilidad con documentación previa):

```bash
python three_stage_transmittance_evaluation.py \
  --model_dir outputs_three_stage_YYYYMMDD_HHMMSS \
  --data_npz preprocessed_t_data.npz \
  --sample_count 8
```

Nota sobre el estado del repositorio: `three_stage_transmittance_evaluation.py` no está presente en esta instantánea. Use `FilterShapeS4_Evaluator_Transmittance.py` para la funcionalidad de evaluación disponible.

### G) Comprobación opcional de consistencia red neuronal vs S4

```bash
python FilterShapeS4_Evaluator_Transmittance.py \
  --npz_file preprocessed_t_data.npz \
  --spec2shape_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageC/spec2shape_stageC.pt \
  --shape2spec_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageA/shape2spec_stageA.pt \
  --n_samples 4
```

## ⚙️ Configuración

### Lanzadores S4 (`ms_final.sh`, `ms_resume_allargs.sh`)

| Flag | Significado | Predeterminado |
|---|---|---|
| `-ns`, `--numshapes` | Número de formas a generar | `100000` |
| `-r`, `--seed` | Semilla aleatoria | `88888` |
| `-p`, `--prefix` | Prefijo/clave de reanudación | `""` |
| `-g`, `--numg` | Parámetro base/cuadrícula | `80` |
| `-bo`, `--baseouter` | Desplazamiento base del límite exterior | `0.25` |
| `-ro`, `--randouter` | Desplazamiento aleatorio del límite exterior | `0.20` |

### Entrenamiento (`three_stage_transmittance.py`)

| Flag | Significado | Predeterminado |
|---|---|---|
| `--preprocess` | Ejecutar modo de preprocesamiento | `False` |
| `--input_folder` | Carpeta que contiene CSV fusionados | `""` |
| `--output_npz` | Ruta de salida NPZ | `preprocessed_data.npz` |
| `--data_npz` | Conjunto NPZ para entrenamiento | `""` |
| `--csv_file` | CSV alternativo si no se usa NPZ | `""` |
| `--test` | Modo de prueba | `False` |
| `--num_epochs` | Número de épocas de entrenamiento | `10` |
| `--batch_size` | Tamaño de lote | `4096` |

### Configuración de evaluación histórica (`three_stage_transmittance_evaluation.py`)

| Flag | Significado | Predeterminado |
|---|---|---|
| `--model_dir` | Directorio que contiene `stageA/B/C` | required |
| `--data_npz` | Entrada NPZ | `""` |
| `--csv_file` | CSV alternativo de entrada | `""` |
| `--output_dir` | Sobrescritura de directorio de salida | auto under `model_dir` |
| `--sample_count` | Número de muestras visualizadas | `4` |
| `--seed` | Semilla aleatoria | `23` |
| `--font_scale` | Escalado de fuente en gráficos | `1.0` |
| `--batch_size` | Tamaño de lote para evaluación | `32` |
| `--plot_only` | Solo graficar curvas de entrenamiento | `False` |

### Evaluador de consistencia S4 (`FilterShapeS4_Evaluator_Transmittance.py`)

| Flag | Significado | Predeterminado |
|---|---|---|
| `--npz_file` | Archivo NPZ de entrada | `preprocessed_t_data.npz` |
| `--spec2shape_ckpt` | Ruta de checkpoint de Etapa C | `outputs_three_stage_20250322_145925/stageC/spec2shape_stageC.pt` |
| `--shape2spec_ckpt` | Ruta de checkpoint de Etapa A | `outputs_three_stage_20250322_145925/stageA/shape2spec_stageA.pt` |
| `--n_samples` | Número de muestras evaluadas | `4` |
| `--seed` | Semilla aleatoria | `23` |
| `--max_workers` | Hilos de trabajo S4 | `4` |
| `--out_folder` | Directorio de salida | auto timestamp |

## 🧪 Ejemplos

### Ejecución de humo

```bash
./ms_final.sh -ns 1000 -r 42 -p smoke -g 40 -bo 0.25 -ro 0.20
python merge_s4_data_full.py --prefix smoke
python -c "import pandas as pd; p='merged_s4_shapes_smoke.csv'; d=pd.read_csv(p).rename(columns={'folder_key':'prefix','NQ':'nQ'}); d.to_csv(p,index=False)"
mkdir -p merged_csvs && mv merged_s4_shapes_smoke.csv merged_csvs/
python three_stage_transmittance.py --preprocess --input_folder merged_csvs --output_npz smoke.npz
python three_stage_transmittance.py --data_npz smoke.npz --num_epochs 5 --batch_size 128
```

### Ejemplos de perfiles de ejecución (de `commands.md` / `commands_updated.md`)

```bash
# no overlap, G=40
./ms_resume_allargs.sh -ns 10000 -r 12345 -p fast_without_overlap -g 40 -bo 0.25 -ro 0.2

# overlap, G=80
./ms_resume_allargs.sh -ns 10000 -r 12345 -p overlap -g 80 -bo 0.35 -ro 0.3

# large run, overlap, G=80
./ms_resume_allargs.sh -ns 100000 -r 12345 -p more_basis_overlap -g 80 -bo 0.35 -ro 0.3
```

## 🔬 Contexto de investigación

La configuración actual de diseño inverso aprende a recuperar geometría con simetría C4 a partir de la transmitancia en estados de cristalización. El pipeline de transmitancia actualmente asume:

- 11 filas de cristalización por muestra de forma (agrupadas por `shape_uid` único)
- 100 bins de longitud de onda por estado de cristalización (columnas `T@...`)
- Hasta 4 puntos de control Q1 codificados como un tensor `4x3`: `(presence, x, y)`
- Reconstrucción de polígonos bajo simetría C4 para visualización de formas y comprobaciones de consistencia

El repositorio también contiene ramas exploratorias (`AVIRIS*`, `noise_experiment*`, `archived/`) además de la ruta principal de entrenamiento de transmitancia.

## 🧑‍💻 Notas de desarrollo

- Este es un repositorio de investigación centrado en scripts, no un módulo Python empaquetado.
- Los scripts principales asumen rutas relativas (especialmente `../build/S4`, `results/`, `shapes/`).
- `.gitignore` excluye muchos artefactos generados de experimentos (`*.csv`, `*.npz`, `*.pt`, carpetas de ejecución).
- Algunos archivos/directorios en documentación histórica están ausentes en esta instantánea; estas referencias se conservan intencionalmente con notas por compatibilidad.
- Archivos sidecar de macOS (`._*`) están presentes y pueden ser artefactos de metadatos no funcionales.

## 🧯 Solución de problemas

| Síntoma | Causa probable | Solución |
|---|---|---|
| `../build/S4: No such file or directory` | Falta el binario S4 en la ruta relativa esperada | Compile o enlace S4 en `../build/S4`, o actualice rutas de lanzadores |
| `No transmission columns found` | El CSV no contiene columnas `T@...` | Vuelva a revisar el formato de salida de fusión |
| `Must specify either --data_npz or --csv_file` | Falta argumento de datos para entrenamiento/evaluación | Proporcione explícitamente una entrada |
| `No valid shapes => SHIFT->Q1->UpTo4` | `vertices_str` inválido/vacío o el filtrado Q1 elimina todas las muestras | Valide archivos de forma y salida de fusión |
| Salida de fusión vacía para `--prefix` | El prefijo no coincide con archivos en `results/` | Verifique el prefijo exacto del nombre de archivo y vuelva a ejecutar la fusión |
| Falta checkpoint de evaluación | Faltan archivos de checkpoint `stageA/B/C` | Verifique que `--model_dir` apunte a la carpeta de salida completa |
| `three_stage_transmittance_evaluation.py` not found | Script referenciado por documentación histórica pero actualmente ausente | Use `FilterShapeS4_Evaluator_Transmittance.py` o restaure ese script desde commits anteriores |

## 🗺️ Hoja de ruta

- Mejorar la reproducibilidad con un manifiesto explícito de versionado de datos y configuraciones de ejecución fijadas.
- Consolidar puntos de entrada canónicos para ramas de transmitancia, AVIRIS y ruido.
- Añadir pruebas smoke automatizadas para preprocesamiento y una mini época de entrenamiento.
- Añadir un registro de experimentos más claro que vincule carpetas de salida con líneas de comando exactas.
- Ampliar el flujo de sincronización de README multilingüe (archivos de idioma en raíz y `i18n/`).

## 🤝 Contribución

Se aceptan contribuciones, especialmente para reproducibilidad, pruebas y calidad de documentación.

Proceso sugerido:

1. Abra un issue con alcance y comportamiento esperado.
2. Cree una rama enfocada.
3. Envíe un pull request con comandos ejecutables y salidas.
4. Mantenga los cambios acotados a un flujo de trabajo cuando sea posible.

## 📄 Licencia

Actualmente no hay un archivo `LICENSE` en la raíz del repositorio en esta instantánea. Agregue uno para definir términos de uso y redistribución.

## 📚 Citación

Si utiliza este repositorio o se basa en este trabajo, cite:

```bibtex
@article{chen2025inverse,
  title={Inverse Design of Metasurface for Spectral Imaging},
  author={Chen, Rongzhou and Nie, Haitao and Zhu, Shuo and Zhao, Yaping and Wang, Chutian and Lam, Edmund Y},
  journal={arXiv preprint arXiv:2510.21924},
  year={2025}
}
```

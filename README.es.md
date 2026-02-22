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

Un repositorio de investigación orientado a scripts (referenciado históricamente como `inverse_metasurface`) para el **diseño inverso de metasuperficies con simetría C4** en imagen espectral, que abarca:

- generación de datos RCWA con S4 (`.lua` + lanzadores de shell)
- fusión y preprocesamiento de datos (`.csv` -> `.npz`)
- entrenamiento neuronal en tres etapas (forma->espectro, espectro->forma, ajuste fino en cadena)
- evaluación y comprobaciones opcionales red neuronal vs S4

## ✨ Resumen Rápido

| Elemento | Detalles |
|---|---|
| Objetivo principal | Predecir la geometría a partir de espectros de transmisión objetivo |
| Forma principal del dataset | espectro: `11 x 100`, forma: `4 x 3` |
| Script principal de entrenamiento | `three_stage_transmittance.py` |
| Script principal de evaluación | `three_stage_transmittance_evaluation.py` |
| Lanzador RCWA | `ms_final.sh`, `ms_resume_allargs.sh` |
| Script de fusión | `merge_s4_data_full.py` |

## 🧠 Contexto de Investigación

Este proyecto se centra en el diseño inverso de metasuperficies para imagen espectral. El pipeline de entrenamiento usa espectros de transmisión generados con S4 a través de estados de cristalización y aprende tanto el mapeo directo como el inverso:

1. **Etapa A**: forma -> espectro
2. **Etapa B**: espectro -> forma
3. **Etapa C**: espectro -> forma -> espectro (ajuste fino con pérdida en cadena)

El código actual de preprocesamiento/entrenamiento asume:

- 11 estados de cristalización (`c = 0.0 ... 1.0`)
- 100 bins de longitud de onda por estado
- representación de forma como hasta 4 puntos Q1 con `[presence, x, y]`

## 🗂️ Estructura del Repositorio (Ruta Principal)

```text
.
├── ms.sh / ms_final.sh / ms_resume_allargs.sh
├── metasurface_final.lua / metasurface_allargs_resume.lua / metasurface_seed.lua
├── merge_s4_data_full.py
├── three_stage_transmittance.py
├── three_stage_transmittance_evaluation.py
├── FilterShapeS4_Evaluator_Transmittance.py
├── results/                # CSVs crudos de salida de S4
├── shapes/                 # vértices de polígonos generados
├── merged_csvs/            # CSVs fusionados usados para preprocesamiento
├── outputs_three_stage_*/  # checkpoints, pérdidas, visualizaciones
├── partial_crys_data/
└── iccp.yaml
```

## ⚙️ Requisitos Previos

| Dependencia | Requisito |
|---|---|
| SO | Linux |
| Shell | Bash |
| Python | 3.9 |
| Gestor de entorno | Conda (recomendado) |
| Binario RCWA | `../build/S4` (relativo a la raíz del repositorio) |
| GPU | Opcional, recomendada para entrenar más rápido |

## 🚀 Instalación

```bash
git clone <repo-url> inverse_metasurface
cd inverse_metasurface

conda env create -f iccp.yaml
conda activate iccp

# Required by launcher scripts
ls -l ../build/S4
```

Opcional:

```bash
chmod +x ms.sh ms_final.sh ms_resume_allargs.sh
```

## 🧪 Uso de Extremo a Extremo

### 1) Generar datos RCWA con S4

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

- Los lanzadores llaman a `../build/S4` con `-t 32` y ejecutan `NQ=1..4` en paralelo.
- `ms_final.sh` usa `metasurface_final.lua`.
- `ms_resume_allargs.sh` usa `metasurface_allargs_resume.lua`.

### 2) Fusionar salidas RCWA con vértices de forma

```bash
python merge_s4_data_full.py --prefix myrun
# -> merged_s4_shapes_myrun.csv
```

### 3) Normalizar nombres de columnas fusionadas para entrenamiento (si hace falta)

`merge_s4_data_full.py` escribe `folder_key` / `NQ`, mientras que el pipeline de entrenamiento espera `prefix` / `nQ`.

```bash
python -c "import pandas as pd; p='merged_s4_shapes_myrun.csv'; df=pd.read_csv(p); df=df.rename(columns={'folder_key':'prefix','NQ':'nQ'}); df.to_csv(p,index=False)"
```

### 4) Preprocesar CSV -> NPZ

```bash
mkdir -p merged_csvs
mv merged_s4_shapes_myrun.csv merged_csvs/

python three_stage_transmittance.py \
  --preprocess \
  --input_folder merged_csvs \
  --output_npz preprocessed_t_data.npz
```

### 5) Entrenar modelos de tres etapas

```bash
python three_stage_transmittance.py \
  --data_npz preprocessed_t_data.npz \
  --num_epochs 100 \
  --batch_size 1024
```

Estructura principal de salida:

- `outputs_three_stage_YYYYMMDD_HHMMSS/stageA`
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageB`
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageC`

### 6) Evaluar checkpoints

```bash
python three_stage_transmittance_evaluation.py \
  --model_dir outputs_three_stage_YYYYMMDD_HHMMSS \
  --data_npz preprocessed_t_data.npz \
  --sample_count 8
```

### 7) Opcional: comparar predicciones neuronales con S4

```bash
python FilterShapeS4_Evaluator_Transmittance.py \
  --npz_file preprocessed_t_data.npz \
  --spec2shape_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageC/spec2shape_stageC.pt \
  --shape2spec_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageA/shape2spec_stageA.pt \
  --n_samples 4
```

## 🎛️ Referencia de CLI

### Flags del lanzador S4 (`ms_final.sh`, `ms_resume_allargs.sh`)

| Flag | Significado | Valor por defecto |
|---|---|---|
| `-ns`, `--numshapes` | Número de formas | `100000` |
| `-r`, `--seed` | Semilla aleatoria | `88888` |
| `-p`, `--prefix` | Prefijo de ejecución / clave de reanudación | `""` |
| `-g`, `--numg` | Parámetro base de geometría | `80` |
| `-bo`, `--baseouter` | Desplazamiento externo base | `0.25` |
| `-ro`, `--randouter` | Desplazamiento externo aleatorio | `0.20` |

### Flags de entrenamiento (`three_stage_transmittance.py`)

| Flag | Propósito |
|---|---|
| `--preprocess` | Ejecutar modo de preprocesamiento |
| `--input_folder` | Carpeta de archivos CSV fusionados |
| `--output_npz` | Nombre de archivo NPZ preprocesado de salida |
| `--data_npz` | Dataset NPZ para entrenamiento |
| `--csv_file` | Alternativa de dataset CSV |
| `--test` | Modo de prueba |
| `--num_epochs` | Épocas de entrenamiento |
| `--batch_size` | Tamaño de lote |

### Flags de evaluación (`three_stage_transmittance_evaluation.py`)

| Flag | Propósito |
|---|---|
| `--model_dir` | Directorio raíz de checkpoints (requerido) |
| `--data_npz` / `--csv_file` | Fuente de datos para evaluación |
| `--output_dir` | Carpeta de salida de evaluación |
| `--sample_count` | Número de muestras visualizadas |
| `--seed` | Semilla aleatoria para seleccionar muestras |
| `--font_scale` | Escalado de fuente de las gráficas |
| `--batch_size` | Tamaño de lote para evaluación |
| `--plot_only` | Regenerar solo las gráficas de curvas de entrenamiento |

## 🧾 Contrato de Datos (para preprocesamiento)

La ruta de preprocesamiento en `three_stage_transmittance.py` espera CSVs fusionados que contengan:

- columnas de ID: `prefix`, `nQ`, `nS`, `shape_idx`, `c`
- texto de geometría: `vertices_str`
- columnas espectrales: `T@...`

Comprobaciones de calidad aplicadas por el código:

- agrupación por `shape_uid = prefix_nQ_nS_shape_idx`
- cada grupo debe contener exactamente 11 filas
- solo se conservan formas con conteo de puntos Q1 en `[1, 4]`

## 🛠️ Resolución de Problemas

- `../build/S4: No such file or directory`
  - Compila/enlaza S4 en `../build/S4` o edita los scripts lanzadores con tu ruta real de S4.
- `No matching CSVs found in 'results/'`
  - Verifica `--prefix` y el esquema de nombres de salida en `results/*_output_nQ*_nS*.csv`.
- `No transmission columns found`
  - Asegúrate de que el CSV fusionado contenga columnas `T@...`.
- El preprocesamiento produce cero registros
  - Verifica las columnas requeridas y que cada UID de forma tenga 11 filas de cristalización.
- GPU OOM durante entrenamiento
  - Reduce `--batch_size` (por ejemplo `256` o `128`).
- La evaluación no encuentra checkpoints
  - Confirma que existan los siguientes archivos bajo `--model_dir`:
    - `stageA/shape2spec_stageA.pt`
    - `stageB/spec2shape_stageB.pt`
    - `stageC/spec2shape_stageC.pt`

## 📚 Citación

Si este repositorio contribuye a tu investigación, cita por favor:

```bibtex
@article{chen2025inverse,
  title={Inverse Design of Metasurface for Spectral Imaging},
  author={Chen, Rongzhou and Nie, Haitao and Zhu, Shuo and Zhao, Yaping and Wang, Chutian and Lam, Edmund Y},
  journal={arXiv preprint arXiv:2510.21924},
  year={2025}
}
```

## 🌐 Variantes de Idioma

Hay variantes adicionales del README disponibles en este repositorio, incluyendo:

- `README.en.md`, `README.de.md`, `README.es.md`, `README.fr.md`
- `README.ru.md`, `README.ja.md`, `README.ko.md`, `README.vi.md`
- `README.ar.md`, `README.zh-CN.md`, `README.zh-TW.md`

## 📌 Notas

- Este es un espacio de trabajo de investigación con muchos scripts archivados y exploratorios.
- La ruta canónica de transmitancia se centra en:
  - `ms_final.sh` / `ms_resume_allargs.sh`
  - `merge_s4_data_full.py`
  - `three_stage_transmittance.py`
  - `three_stage_transmittance_evaluation.py`
- Actualmente no hay un archivo de licencia explícito definido en la raíz del repositorio.

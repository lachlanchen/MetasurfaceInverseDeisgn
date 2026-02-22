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

Un repositorio de investigación centrado en scripts (históricamente llamado `inverse_metasurface`) para el diseño inverso de metasuperficies en imagen espectral. El flujo de trabajo principal acopla:

- simulación RCWA basada en física (S4 + Lua)
- ensamblado de datos y vinculación de geometrías
- aprendizaje en tres etapas con PyTorch (`shape -> spectrum`, `spectrum -> shape`, ajuste fino encadenado)
- evaluación cuantitativa y cualitativa, con comprobaciones opcionales de consistencia red neuronal vs S4

## ✨ Resumen Rápido

| Elemento | Detalles |
|---|---|
| 🎯 Tarea principal | Inferir la geometría de metasuperficie con simetría C4 a partir de espectros objetivo de transmitancia |
| 🔬 Simulador | `../build/S4` invocado por lanzadores shell y scripts `.lua` |
| 🧠 Pipeline de aprendizaje | Etapa A `shape -> spectra`, Etapa B `spectra -> shape`, Etapa C `spectra -> shape -> spectra` |
| 📦 Contrato de datos | CSV fusionado (`T@...`, metadatos, `vertices_str`) -> NPZ comprimido (`uids`, `spectra`, `shapes`) |
| 🧪 Evaluación | Métricas MSE, visualizaciones por etapa, re-simulación opcional con S4 |

## 🧭 Flujo de Trabajo End-to-End

1. Generar salidas de simulación en `results/` y archivos de polígonos en `shapes/`.
2. Fusionar archivos CSV de S4 y adjuntar vértices de geometría.
3. Normalizar nombres de columnas fusionadas para compatibilidad con entrenamiento.
4. Preprocesar los CSV fusionados a tensores NPZ.
5. Entrenar los modelos de las etapas A/B/C.
6. Evaluar checkpoints y visualizar comportamiento.
7. Opcionalmente comparar espectros de geometrías predichas contra nuevas ejecuciones S4.

## 🧱 Estructura del Repositorio

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

## 🛠️ Requisitos Previos

| Dependencia | Notas |
|---|---|
| Linux + Bash | Los scripts de lanzamiento apuntan a ejecución en shell |
| Python 3.9 | Coincide con `iccp.yaml` |
| Conda | Recomendado para reproducibilidad |
| Binario S4 | Esperado en `../build/S4` |
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

Alternativa (menos controlada):

```bash
pip install -r pip_requirements.txt
```

### 3) Verificar la ruta del simulador esperada por los scripts

```bash
ls -l ../build/S4
```

### 4) (Opcional) hacer ejecutables los lanzadores

```bash
chmod +x ms.sh ms_final.sh ms_resume.sh ms_resume_allargs.sh ms_resume_random_state.sh
```

## ▶️ Uso Práctico

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

Notas:
- Los lanzadores ejecutan `NQ=1..4` en paralelo.
- Los scripts llaman a `../build/S4` con `-t 32`.

### B) Fusionar salidas de S4 y adjuntar vértices de geometría

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

```bash
python three_stage_transmittance_evaluation.py \
  --model_dir outputs_three_stage_YYYYMMDD_HHMMSS \
  --data_npz preprocessed_t_data.npz \
  --sample_count 8
```

### G) Comprobación opcional de consistencia red neuronal vs S4

```bash
python FilterShapeS4_Evaluator_Transmittance.py \
  --npz_file preprocessed_t_data.npz \
  --spec2shape_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageC/spec2shape_stageC.pt \
  --shape2spec_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageA/shape2spec_stageA.pt \
  --n_samples 4
```

## ⚙️ Opciones CLI Clave

### Lanzadores S4 (`ms_final.sh`, `ms_resume_allargs.sh`)

| Flag | Significado | Valor por defecto |
|---|---|---|
| `-ns`, `--numshapes` | Número de geometrías a generar | `100000` |
| `-r`, `--seed` | Semilla aleatoria | `88888` |
| `-p`, `--prefix` | Prefijo/clave de reanudación | `""` |
| `-g`, `--numg` | Parámetro de base/cuadrícula | `80` |
| `-bo`, `--baseouter` | Desplazamiento base del límite externo | `0.25` |
| `-ro`, `--randouter` | Desplazamiento aleatorio del límite externo | `0.20` |

### Entrenamiento (`three_stage_transmittance.py`)

| Flag | Significado | Valor por defecto |
|---|---|---|
| `--preprocess` | Ejecutar modo de preprocesamiento | `False` |
| `--input_folder` | Carpeta con archivos CSV fusionados | `""` |
| `--output_npz` | Ruta del NPZ de salida | `preprocessed_data.npz` |
| `--data_npz` | Dataset NPZ para entrenamiento | `""` |
| `--csv_file` | CSV de respaldo si no se usa NPZ | `""` |
| `--test` | Modo prueba | `False` |
| `--num_epochs` | Número de épocas de entrenamiento | `10` |
| `--batch_size` | Tamaño de batch | `4096` |

### Evaluación (`three_stage_transmittance_evaluation.py`)

| Flag | Significado | Valor por defecto |
|---|---|---|
| `--model_dir` | Directorio que contiene `stageA/B/C` | requerido |
| `--data_npz` | NPZ de entrada | `""` |
| `--csv_file` | CSV de entrada de respaldo | `""` |
| `--output_dir` | Sobrescritura de directorio de salida | automático bajo `model_dir` |
| `--sample_count` | Número de muestras visualizadas | `4` |
| `--seed` | Semilla aleatoria | `23` |
| `--font_scale` | Escala de fuente en gráficas | `1.0` |
| `--batch_size` | Tamaño de batch de evaluación | `32` |
| `--plot_only` | Solo graficar curvas de entrenamiento | `False` |

### Evaluador de consistencia S4 (`FilterShapeS4_Evaluator_Transmittance.py`)

| Flag | Significado | Valor por defecto |
|---|---|---|
| `--npz_file` | Archivo NPZ de entrada | `preprocessed_t_data.npz` |
| `--spec2shape_ckpt` | Ruta del checkpoint de Etapa C | `outputs_three_stage_20250322_145925/stageC/spec2shape_stageC.pt` |
| `--shape2spec_ckpt` | Ruta del checkpoint de Etapa A | `outputs_three_stage_20250322_145925/stageA/shape2spec_stageA.pt` |
| `--n_samples` | Número de muestras evaluadas | `4` |
| `--seed` | Semilla aleatoria | `23` |
| `--max_workers` | Hilos de trabajo S4 | `4` |
| `--out_folder` | Directorio de salida | marca de tiempo automática |

## 🧪 Ejecución de Humo

```bash
./ms_final.sh -ns 1000 -r 42 -p smoke -g 40 -bo 0.25 -ro 0.20
python merge_s4_data_full.py --prefix smoke
python -c "import pandas as pd; p='merged_s4_shapes_smoke.csv'; d=pd.read_csv(p).rename(columns={'folder_key':'prefix','NQ':'nQ'}); d.to_csv(p,index=False)"
mkdir -p merged_csvs && mv merged_s4_shapes_smoke.csv merged_csvs/
python three_stage_transmittance.py --preprocess --input_folder merged_csvs --output_npz smoke.npz
python three_stage_transmittance.py --data_npz smoke.npz --num_epochs 5 --batch_size 128
```

## 🔬 Contexto de Investigación

La configuración actual de diseño inverso aprende a recuperar geometría con simetría C4 a partir de la transmitancia en distintos estados de cristalización. Actualmente, el pipeline de transmitancia asume:

- 11 filas de cristalización por muestra de geometría (agrupadas por `shape_uid` único)
- 100 bins de longitud de onda por estado de cristalización (columnas `T@...`)
- hasta 4 puntos de control Q1 codificados como tensor `4x3`: `(presence, x, y)`
- reconstrucción de polígonos bajo simetría C4 para visualización y comprobaciones de consistencia

El repositorio también contiene ramas exploratorias (`AVIRIS*`, `noise_experiment*`, `archived/`) además de la ruta principal de entrenamiento de transmitancia.

## 🧯 Solución de Problemas

| Síntoma | Causa probable | Solución |
|---|---|---|
| `../build/S4: No such file or directory` | Falta el binario S4 en la ruta relativa esperada | Compilar o enlazar S4 en `../build/S4`, o actualizar rutas de los lanzadores |
| `No transmission columns found` | El CSV no contiene columnas `T@...` | Revisar de nuevo el formato de salida de la fusión |
| `Must specify either --data_npz or --csv_file` | Falta el argumento de datos para entrenamiento/evaluación | Proporcionar explícitamente una entrada |
| `No valid shapes => SHIFT->Q1->UpTo4` | `vertices_str` inválido/vacío o el filtrado Q1 elimina todas las muestras | Validar archivos de geometría y salida de fusión |
| Salida de fusión vacía para `--prefix` | El prefijo no coincide con archivos en `results/` | Verificar prefijo exacto de nombre de archivo y repetir la fusión |
| Falta checkpoint de evaluación | Faltan archivos de checkpoint `stageA/B/C` | Verificar que `--model_dir` apunte a una carpeta de salida completa |

## 🤝 Contribución

Las contribuciones son bienvenidas, especialmente en reproducibilidad, pruebas y calidad de documentación.

Proceso sugerido:

1. Abrir un issue con el alcance y comportamiento esperado.
2. Crear una rama enfocada.
3. Enviar un pull request con comandos y salidas reproducibles.
4. Mantener los cambios acotados a un flujo de trabajo cuando sea posible.

## 📄 Licencia

Actualmente no existe un archivo `LICENSE` en la raíz del repositorio en esta instantánea. Añade uno para definir términos de uso y redistribución.

## 📚 Cita

Si usas este repositorio o construyes sobre este trabajo, por favor cita:

```bibtex
@article{chen2025inverse,
  title={Inverse Design of Metasurface for Spectral Imaging},
  author={Chen, Rongzhou and Nie, Haitao and Zhu, Shuo and Zhao, Yaping and Wang, Chutian and Lam, Edmund Y},
  journal={arXiv preprint arXiv:2510.21924},
  year={2025}
}
```

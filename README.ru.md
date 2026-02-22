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

Репозиторий исследовательского типа с акцентом на скрипты (исторически упоминается как `inverse_metasurface`) для **обратного проектирования метаповерхностей с C4-симметрией** в спектральной визуализации, включая:

- генерацию данных RCWA через S4 (`.lua` + shell-скрипты запуска)
- объединение и предобработку данных (`.csv` -> `.npz`)
- трёхэтапное обучение нейросети (shape->spectra, spectra->shape, chain fine-tuning)
- оценку качества и опциональные проверки neural-vs-S4

## ✨ Кратко

| Пункт | Детали |
|---|---|
| Основная цель | Предсказание геометрии по целевым спектрам пропускания |
| Формат основного датасета | spectra: `11 x 100`, shape: `4 x 3` |
| Главный скрипт обучения | `three_stage_transmittance.py` |
| Главный скрипт оценки | `three_stage_transmittance_evaluation.py` |
| Скрипты запуска RCWA | `ms_final.sh`, `ms_resume_allargs.sh` |
| Скрипт объединения | `merge_s4_data_full.py` |

## 🧠 Исследовательский контекст

Проект посвящён обратному проектированию метаповерхностей для спектральной визуализации. Конвейер обучения использует спектры пропускания, сгенерированные S4 для разных состояний кристаллизации, и изучает как прямое, так и обратное отображение:

1. **Этап A**: shape -> spectra
2. **Этап B**: spectra -> shape
3. **Этап C**: spectra -> shape -> spectra (дообучение с chain loss)

Текущий код предобработки и обучения предполагает:

- 11 состояний кристаллизации (`c = 0.0 ... 1.0`)
- 100 бинов длины волны на состояние
- представление формы как до 4 точек Q1 с `[presence, x, y]`

## 🗂️ Структура репозитория (основной путь)

```text
.
├── ms.sh / ms_final.sh / ms_resume_allargs.sh
├── metasurface_final.lua / metasurface_allargs_resume.lua / metasurface_seed.lua
├── merge_s4_data_full.py
├── three_stage_transmittance.py
├── three_stage_transmittance_evaluation.py
├── FilterShapeS4_Evaluator_Transmittance.py
├── results/                # raw S4 output CSVs
├── shapes/                 # generated polygon vertices
├── merged_csvs/            # merged CSVs used for preprocessing
├── outputs_three_stage_*/  # checkpoints, losses, visualizations
├── partial_crys_data/
└── iccp.yaml
```

## ⚙️ Требования

| Зависимость | Требование |
|---|---|
| ОС | Linux |
| Shell | Bash |
| Python | 3.9 |
| Менеджер окружений | Conda (рекомендуется) |
| Бинарник RCWA | `../build/S4` (относительно корня репозитория) |
| GPU | Необязательно, но рекомендуется для ускорения обучения |

## 🚀 Установка

```bash
git clone <repo-url> inverse_metasurface
cd inverse_metasurface

conda env create -f iccp.yaml
conda activate iccp

# Required by launcher scripts
ls -l ../build/S4
```

Опционально:

```bash
chmod +x ms.sh ms_final.sh ms_resume_allargs.sh
```

## 🧪 Сквозной сценарий использования

### 1) Сгенерировать RCWA-данные через S4

```bash
./ms_final.sh \
  -ns 10000 \
  -r 12345 \
  -p myrun \
  -g 80 \
  -bo 0.35 \
  -ro 0.30
```

Примечания:

- Скрипты запуска вызывают `../build/S4` с `-t 32` и параллельно выполняют `NQ=1..4`.
- `ms_final.sh` использует `metasurface_final.lua`.
- `ms_resume_allargs.sh` использует `metasurface_allargs_resume.lua`.

### 2) Объединить RCWA-выходы с вершинами формы

```bash
python merge_s4_data_full.py --prefix myrun
# -> merged_s4_shapes_myrun.csv
```

### 3) Нормализовать имена столбцов для обучения (при необходимости)

`merge_s4_data_full.py` записывает `folder_key` / `NQ`, а обучающий конвейер ожидает `prefix` / `nQ`.

```bash
python -c "import pandas as pd; p='merged_s4_shapes_myrun.csv'; df=pd.read_csv(p); df=df.rename(columns={'folder_key':'prefix','NQ':'nQ'}); df.to_csv(p,index=False)"
```

### 4) Предобработка CSV -> NPZ

```bash
mkdir -p merged_csvs
mv merged_s4_shapes_myrun.csv merged_csvs/

python three_stage_transmittance.py \
  --preprocess \
  --input_folder merged_csvs \
  --output_npz preprocessed_t_data.npz
```

### 5) Обучить трёхэтапные модели

```bash
python three_stage_transmittance.py \
  --data_npz preprocessed_t_data.npz \
  --num_epochs 100 \
  --batch_size 1024
```

Основная структура выходных данных:

- `outputs_three_stage_YYYYMMDD_HHMMSS/stageA`
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageB`
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageC`

### 6) Оценить чекпойнты

```bash
python three_stage_transmittance_evaluation.py \
  --model_dir outputs_three_stage_YYYYMMDD_HHMMSS \
  --data_npz preprocessed_t_data.npz \
  --sample_count 8
```

### 7) Опционально: сравнить предсказания нейросети с S4

```bash
python FilterShapeS4_Evaluator_Transmittance.py \
  --npz_file preprocessed_t_data.npz \
  --spec2shape_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageC/spec2shape_stageC.pt \
  --shape2spec_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageA/shape2spec_stageA.pt \
  --n_samples 4
```

## 🎛️ Справка по CLI

### Флаги скриптов запуска S4 (`ms_final.sh`, `ms_resume_allargs.sh`)

| Флаг | Значение | По умолчанию |
|---|---|---|
| `-ns`, `--numshapes` | Количество форм | `100000` |
| `-r`, `--seed` | Seed генератора случайных чисел | `88888` |
| `-p`, `--prefix` | Префикс запуска / ключ возобновления | `""` |
| `-g`, `--numg` | Параметр геометрического базиса | `80` |
| `-bo`, `--baseouter` | Базовый внешний сдвиг | `0.25` |
| `-ro`, `--randouter` | Случайный внешний сдвиг | `0.20` |

### Флаги обучения (`three_stage_transmittance.py`)

| Флаг | Назначение |
|---|---|
| `--preprocess` | Запуск режима предобработки |
| `--input_folder` | Папка с объединёнными CSV-файлами |
| `--output_npz` | Имя выходного NPZ после предобработки |
| `--data_npz` | NPZ-датасет для обучения |
| `--csv_file` | Альтернатива: CSV-датасет |
| `--test` | Тестовый режим |
| `--num_epochs` | Количество эпох обучения |
| `--batch_size` | Размер батча |

### Флаги оценки (`three_stage_transmittance_evaluation.py`)

| Флаг | Назначение |
|---|---|
| `--model_dir` | Корневая директория чекпойнтов (обязательно) |
| `--data_npz` / `--csv_file` | Источник данных для оценки |
| `--output_dir` | Папка для результатов оценки |
| `--sample_count` | Число визуализируемых примеров |
| `--seed` | Seed для выбора примеров |
| `--font_scale` | Масштаб шрифтов на графиках |
| `--batch_size` | Размер батча при оценке |
| `--plot_only` | Только переcоздать графики кривых обучения |

## 🧾 Контракт данных (для предобработки)

Путь предобработки в `three_stage_transmittance.py` ожидает объединённые CSV-файлы, содержащие:

- ID-столбцы: `prefix`, `nQ`, `nS`, `shape_idx`, `c`
- текст геометрии: `vertices_str`
- спектральные столбцы: `T@...`

Проверки качества, применяемые кодом:

- группировка по `shape_uid = prefix_nQ_nS_shape_idx`
- каждая группа должна содержать ровно 11 строк
- сохраняются только формы с числом точек Q1 в `[1, 4]`

## 🛠️ Устранение неполадок

- `../build/S4: No such file or directory`
  - Соберите/свяжите S4 в `../build/S4` или измените скрипты запуска под ваш фактический путь к S4.
- `No matching CSVs found in 'results/'`
  - Проверьте `--prefix` и шаблон именования выходных файлов в `results/*_output_nQ*_nS*.csv`.
- `No transmission columns found`
  - Убедитесь, что в объединённом CSV есть столбцы `T@...`.
- Preprocess yields zero records
  - Проверьте обязательные столбцы и что у каждого shape UID есть 11 строк кристаллизации.
- GPU OOM during training
  - Уменьшите `--batch_size` (например, до `256` или `128`).
- Evaluation cannot find checkpoints
  - Убедитесь, что в `--model_dir` существуют:
    - `stageA/shape2spec_stageA.pt`
    - `stageB/spec2shape_stageB.pt`
    - `stageC/spec2shape_stageC.pt`

## 📚 Цитирование

Если этот репозиторий используется в вашем исследовании, пожалуйста, цитируйте:

```bibtex
@article{chen2025inverse,
  title={Inverse Design of Metasurface for Spectral Imaging},
  author={Chen, Rongzhou and Nie, Haitao and Zhu, Shuo and Zhao, Yaping and Wang, Chutian and Lam, Edmund Y},
  journal={arXiv preprint arXiv:2510.21924},
  year={2025}
}
```

## 🌐 Языковые версии

В этом репозитории также доступны другие варианты README, включая:

- `README.en.md`, `README.de.md`, `README.es.md`, `README.fr.md`
- `README.ru.md`, `README.ja.md`, `README.ko.md`, `README.vi.md`
- `README.ar.md`, `README.zh-CN.md`, `README.zh-TW.md`

## 📌 Примечания

- Это исследовательская рабочая область со множеством архивных и экспериментальных скриптов.
- Канонический путь для сценария transmittance сосредоточен вокруг:
  - `ms_final.sh` / `ms_resume_allargs.sh`
  - `merge_s4_data_full.py`
  - `three_stage_transmittance.py`
  - `three_stage_transmittance_evaluation.py`
- В корне репозитория пока нет явно заданного файла лицензии.

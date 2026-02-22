English | 中文繁體 | 中文简体 | 日本語 | 한국어 | Tiếng Việt | العربية | Français | Español | Deutsch | Русский

# inverse_metasurface

<div align="center">

[![Status](https://img.shields.io/badge/status-experimental-orange)](#-roadmap)
[![Python](https://img.shields.io/badge/python-3.9-blue)](#-prerequisites)
[![Platform](https://img.shields.io/badge/platform-linux-lightgrey)](#-prerequisites)
[![S4](https://img.shields.io/badge/S4-required-success)](#-prerequisites)
[![License](https://img.shields.io/badge/license-unset-lightgrey)](#-license)

</div>

Исследовательское рабочее пространство для **обратного проектирования метаповерхностей** с использованием **моделирования S4/RCWA** и **многоэтапных нейросетевых моделей**.

Поддерживается:
- Высокопроизводительная генерация данных S4 для полигональных метаповерхностей с симметрией C4.
- Объединение данных и предобработка в NPZ, готовый для обучения.
- Трёхэтапное обучение: **shape -> spectrum**, **spectrum -> shape**, **chain-tuned spectrum -> shape -> spectrum**.
- Оценка качества и опциональное прямое сравнение **neural-vs-S4**.

## 🌟 Краткий обзор

| Область | Что предоставляет этот репозиторий |
|---|---|
| Физическое моделирование | Bash- и Lua-скрипты для параллельного запуска S4 по `nQ=1..4` |
| Инструменты датасета | Скрипты объединения, добавляющие вершины полигонов и опорные спектры |
| ML-конвейер | `three_stage_transmittance.py` с режимами preprocess и train |
| Оценка | `three_stage_transmittance_evaluation.py` с метриками и графиками |
| Исследовательская ветка | Эксперименты AVIRIS/гиперспектральные и noise/compression |

## 🧠 Исследовательский контекст

Этот репозиторий ориентирован на обратное проектирование фотонных метаповерхностей: восстановление геометрии по целевым спектрам (и наоборот).

Ключевые предположения базового конвейера:
- Симметрия C4 через параметризацию Q1-point.
- 11 состояний кристаллизации (`c` в `[0.0, 1.0]`).
- Спектр каждого образца хранится как строки пропускания `11 x 100`.
- Формы представлены максимум 4 точками Q1 (`4 x 3` tensor: presence, x, y).

Логика трёхэтапного обучения:
1. **Stage A**: shape -> spectrum (`shape2spec_stageA.pt`)
2. **Stage B**: spectrum -> shape (`spec2shape_stageB.pt`)
3. **Stage C**: spectrum -> shape -> spectrum chain fine-tuning (`spec2shape_stageC.pt`)

## 🗂 Структура проекта

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
├─ partial_crys_data/                # optical constants by crystallization level
├─ results/                          # raw S4 outputs (usually untracked)
├─ shapes/                           # generated polygon vertex files
├─ merged_csvs/                      # merged tables used for preprocessing
├─ outputs_three_stage_*/            # model artifacts per run
├─ AVIRIS*/ + aviris_*.py            # hyperspectral branch
├─ how_to_run.md / commands*.md
├─ iccp.yaml
└─ pip_requirements.txt
```

## ✅ Требования

| Требование | Примечания |
|---|---|
| OS | Linux (скрипты предполагают Bash и Linux-пути) |
| Python | 3.9 (из `iccp.yaml`) |
| Менеджер окружений | Рекомендуется Conda |
| S4 binary | Ожидается по пути `../build/S4` относительно корня репозитория |
| GPU (опционально) | CUDA ускоряет обучение/оценку |

## ⚙️ Установка

### 1) Клонирование и переход в каталог

```bash
git clone <your-repo-url> inverse_metasurface
cd inverse_metasurface
```

### 2) Создание окружения

```bash
conda env create -f iccp.yaml
conda activate iccp
```

### 3) Проверка пути к S4

```bash
ls -l ../build/S4
```

Если этого пути нет, соберите/поместите S4 туда или скорректируйте пути в скриптах.

## 🚀 Быстрый старт (End-to-End)

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

## 🧪 Детали использования

### A) Генерация S4

Минимальный запуск с фиксированным seed:

```bash
./ms.sh -ns 10000 -r 88888
```

Параметризованный запуск:

```bash
./ms_final.sh \
  -ns 100000 \
  -r 88888 \
  -p iccpOv100kG80 \
  -g 80 \
  -bo 0.35 \
  -ro 0.30
```

Запуск в стиле resume:

```bash
./ms_resume_allargs.sh \
  -ns 100000 \
  -r 88888 \
  -p iccpOv100kG80 \
  -g 80 \
  -bo 0.35 \
  -ro 0.30
```

### B) Объединение исходных S4 CSV

```bash
python merge.py --prefix 20250123_155420
```

Альтернативная утилита:

```bash
python merge_s4_data_full.py --prefix 20250123_155420
```

### C) Предобработка объединённых CSV -> NPZ

```bash
python three_stage_transmittance.py --preprocess \
  --input_folder merged_csvs \
  --output_npz preprocessed_t_data.npz
```

### D) Обучение

```bash
python three_stage_transmittance.py \
  --data_npz preprocessed_t_data.npz \
  --num_epochs 100 \
  --batch_size 1024
```

Артефакты обучения сохраняются в:
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageA`
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageB`
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageC`

### E) Оценка

```bash
python three_stage_transmittance_evaluation.py \
  --model_dir outputs_three_stage_YYYYMMDD_HHMMSS \
  --data_npz preprocessed_t_data.npz \
  --sample_count 8
```

Создаёт:
- `evaluation_metrics.csv`
- `metrics_summary.csv`
- визуализации по этапам (`.png`, `.pdf`)
- графики кривых обучения

### F) Сравнение Neural vs direct S4 (опционально)

```bash
python FilterShapeS4_Evaluator_Transmittance.py \
  --npz_file preprocessed_t_data.npz \
  --spec2shape_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageC/spec2shape_stageC.pt \
  --shape2spec_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageA/shape2spec_stageA.pt \
  --n_samples 4
```

## 🔧 Ключевые CLI-параметры

### Флаги запуска S4 (`ms_final.sh`, `ms_resume_allargs.sh`)

| Flag | Meaning | Default |
|---|---|---|
| `-ns`, `--numshapes` | Number of shapes | `100000` |
| `-r`, `--seed` | Random seed | `88888` |
| `-p`, `--prefix` | Run prefix / resume key | empty |
| `-g`, `--numg` | Geometry/basis parameter | `80` |
| `-bo`, `--baseouter` | Base outer offset | `0.25` |
| `-ro`, `--randouter` | Outer random offset | `0.20` |

### Флаги обучения (`three_stage_transmittance.py`)

| Flag | Meaning | Default |
|---|---|---|
| `--preprocess` | Run preprocessing mode | off |
| `--input_folder` | Input CSV folder | empty |
| `--output_npz` | Output NPZ path | `preprocessed_data.npz` |
| `--data_npz` | NPZ used for train/eval | empty |
| `--csv_file` | CSV used directly | empty |
| `--num_epochs` | Epochs per stage | `10` |
| `--batch_size` | Batch size | `4096` |
| `--test` | Test mode placeholder | off |

### Флаги оценки (`three_stage_transmittance_evaluation.py`)

| Flag | Meaning | Default |
|---|---|---|
| `--model_dir` | Root trained run directory | required |
| `--data_npz` | Evaluation NPZ | empty |
| `--csv_file` | Evaluation CSV | empty |
| `--output_dir` | Output folder | `model_dir/evaluation_<timestamp>` |
| `--sample_count` | Number of visualized samples | `4` |
| `--seed` | Sampling seed | `23` |
| `--font_scale` | Plot font scale | `1.0` |
| `--batch_size` | Eval batch size | `32` |
| `--plot_only` | Plot curves only | off |

## 🧭 Устранение неполадок

| Симптом | Вероятная причина | Что делать |
|---|---|---|
| `../build/S4: No such file or directory` | Несовпадение пути к S4 binary | Соберите/поместите S4 в `../build/S4` или исправьте скрипты |
| `Must specify either --data_npz or --csv_file` | Не указан источник датасета | Явно передайте один из этих флагов |
| `No matching CSVs found in 'results/'` | Несовпадение prefix | Проверьте prefix и схему именования выходных файлов |
| Очень мало/ноль записей после предобработки | Отсутствуют/некорректны строки `T@...` или `vertices_str` | Проверьте схему merged CSV и группировку строк по shape |
| CUDA OOM | Слишком большой batch | Уменьшите `--batch_size` (например, `1024 -> 256`) |

## 🧱 Заметки по разработке

- Это репозиторий с акцентом на эксперименты; многие сгенерированные артефакты намеренно не отслеживаются.
- Есть несколько вариантов скриптов для похожих задач (`merge_*`, `aviris_*`, `noise_experiment_*`).
- Базовый workflow inverse-metasurface описан в этом README.
- Скрипты обучения выставляют seed (`42`), но строгая детерминированность всё равно зависит от железа/бэкенда.
- Единого CI и полного автоматизированного набора тестов сейчас нет.

## 🛣 Дорожная карта

- Добавить компактный канонический датасет для быстрых smoke-тестов.
- Консолидировать дублирующиеся скрипты конвейера.
- Добавить проверки целостности merge/preprocess и загрузки checkpoint-файлов.
- Добавить CI для lint и smoke train/eval.
- Явно задокументировать сборку/пиннинг версии S4.

## 🤝 Участие

1. Создайте feature-ветку.
2. Держите scope PR узким (одна задача конвейера/эксперимента на PR).
3. Добавляйте точные воспроизводимые команды и ожидаемые выходные пути.
4. Не коммитьте большие сгенерированные артефакты без необходимости.
5. Добавляйте заметки по воспроизводимости (seed, источник данных, путь к checkpoint).

## 📄 Лицензия

В этом репозитории сейчас отсутствует файл `LICENSE`.

Пока файл лицензии не добавлен, условия повторного использования и распространения следует считать **неопределёнными**.

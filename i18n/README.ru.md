[English](../README.md) · [العربية](README.ar.md) · [Español](README.es.md) · [Français](README.fr.md) · [日本語](README.ja.md) · [한국어](README.ko.md) · [Tiếng Việt](README.vi.md) · [中文 (简体)](README.zh-Hans.md) · [中文（繁體）](README.zh-Hant.md) · [Deutsch](README.de.md) · [Русский](README.ru.md)


# Обратное проектирование метаповерхности для спектральной визуализации

<p align="center">
  <img alt="Status" src="https://img.shields.io/badge/Status-Research%20Prototype-f59e0b?style=for-the-badge">
  <img alt="Python" src="https://img.shields.io/badge/Python-3.9-3776AB?style=for-the-badge&logo=python&logoColor=white">
  <img alt="PyTorch" src="https://img.shields.io/badge/PyTorch-2.x-EE4C2C?style=for-the-badge&logo=pytorch&logoColor=white">
  <img alt="Simulator" src="https://img.shields.io/badge/Simulator-S4%20RCWA-16a34a?style=for-the-badge">
  <img alt="Platform" src="https://img.shields.io/badge/Platform-Linux%20%2B%20Bash-6b7280?style=for-the-badge&logo=gnu-bash&logoColor=white">
  <img alt="ArXiv" src="https://img.shields.io/badge/arXiv-2510.21924-b31b1b?style=for-the-badge&logo=arxiv&logoColor=white">
</p>

Исследовательский репозиторий в формате script-first (исторически обозначался как `inverse_metasurface`) для обратного проектирования метаповерхностей в задачах спектральной визуализации.

Основной рабочий конвейер объединяет:
- Физически обоснованное RCWA-моделирование (`S4` + Lua)
- Сборку данных и привязку геометрий
- Трёхэтапное обучение в PyTorch (`shape -> spectrum`, `spectrum -> shape`, связанная донастройка)
- Количественную и качественную оценку, с опциональной проверкой согласованности нейросети и S4

> [!IMPORTANT]
> Каноническое поведение и команды сохранены из существующих скриптов/документации проекта. Там, где исторические ссылки указывают на отсутствующие файлы, эти ссылки намеренно оставлены с явными примечаниями для совместимости.

## 📑 Содержание

- [✨ Кратко](#-кратко)
- [🌍 Интернационализация (i18n)](#-интернационализация-i18n)
- [✨ Возможности](#-возможности)
- [🧭 Сквозной рабочий процесс](#-сквозной-рабочий-процесс)
- [🧱 Структура проекта](#-структура-проекта)
- [🛠️ Предварительные требования](#️-предварительные-требования)
- [🚀 Установка](#-установка)
- [▶️ Использование](#️-использование)
- [⚙️ Конфигурация](#️-конфигурация)
- [🧪 Примеры](#-примеры)
- [🔬 Исследовательский контекст](#-исследовательский-контекст)
- [🧑‍💻 Примечания для разработки](#-примечания-для-разработки)
- [🧯 Устранение неполадок](#-устранение-неполадок)
- [🗺️ Дорожная карта](#️-дорожная-карта)
- [🤝 Вклад](#-вклад)
- [📄 Лицензия](#-лицензия)
- [📚 Цитирование](#-цитирование)

## ✨ Кратко

| Пункт | Детали |
|---|---|
| 🎯 Основная задача | Восстановление C4-симметричной геометрии метаповерхности по целевым спектрам пропускания |
| 🔬 Симулятор | `../build/S4`, вызываемый shell-лаунчерами и `.lua`-скриптами |
| 🧠 Конвейер обучения | Этап A `shape -> spectra`, этап B `spectra -> shape`, этап C `spectra -> shape -> spectra` |
| 📦 Контракт данных | Объединённый CSV (`T@...`, метаданные, `vertices_str`) -> сжатый NPZ (`uids`, `spectra`, `shapes`) |
| 🧪 Оценка | Метрики MSE, визуализации этапов, опциональная повторная симуляция в S4 |
| 🌐 Статус i18n | Многоязычные README в корне + существующий каталог `i18n/` |

## 🌍 Интернационализация (i18n)

- Многоязычные README поддерживаются в корне репозитория как файлы `README.<lang>.md`.
- В этом снимке репозитория присутствует каталог `i18n/`.
- В этом файле сохранена одна строка выбора языка вверху, чтобы избежать дублирования языковых панелей.
- В репозитории также есть `README.en.md`; в этом проходе обновления канонической базой остаётся `README.md`.

## ✨ Возможности

- Сквозной путь обратного проектирования от выходов симуляции S4 до обученной инверсной модели.
- Параметризация C4-симметричного полигона и кодирование точек Q1 (`4x3`: `presence, x, y`).
- Трёхэтапное обучение модели в одном скрипте (`three_stage_transmittance.py`).
- Инструменты слияния, сохраняющие спектральную точность и добавляющие вершины для каждой формы.
- Опциональный оценщик, сравнивающий предсказания модели со свежей симуляцией S4.
- Обширные экспериментальные ветки (AVIRIS, SWIR/noise, GSST, archived/deprecated variants).

## 🧭 Сквозной рабочий процесс

1. Сгенерировать выходы симуляции в `results/` и файлы полигонов в `shapes/`.
2. Объединить CSV-файлы S4 и прикрепить вершины геометрий.
3. Нормализовать названия столбцов для совместимости с обучением.
4. Предобработать объединённые CSV-файлы в NPZ-тензоры.
5. Обучить модели этапов A/B/C.
6. Оценить чекпоинты и визуализировать поведение.
7. Опционально сравнить спектры предсказанных форм с новыми прогонами S4.

## 🧱 Структура проекта

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

## 🛠️ Предварительные требования

| Зависимость | Примечания |
|---|---|
| Linux + Bash | Скрипты-лаунчеры ориентированы на выполнение в shell |
| Python 3.9 | Соответствует `iccp.yaml` (`python=3.9.18`) |
| Conda | Рекомендуется для воспроизводимости |
| S4 binary | Ожидается по пути `../build/S4` |
| CUDA GPU (опционально) | Ускоряет обучение/оценку |

## 🚀 Установка

### 1) Клонирование и переход в каталог

```bash
git clone <your-repo-url> inverse_metasurface
cd inverse_metasurface
```

### 2) Создание окружения (рекомендуется)

```bash
conda env create -f iccp.yaml
conda activate iccp
```

Альтернативная заметка:

```bash
# Historical README reference (file may be absent in this snapshot)
pip install -r pip_requirements.txt
```

### 3) Проверка пути к симулятору, который ожидают скрипты

```bash
ls -l ../build/S4
```

### 4) (Опционально) сделать лаунчеры исполняемыми

```bash
chmod +x ms.sh ms_final.sh ms_resume.sh ms_resume_allargs.sh ms_resume_random_state.sh ms_resume_random_state_nir.sh
```

## ▶️ Использование

### A) Генерация данных RCWA-симуляции

Простой лаунчер:

```bash
./ms.sh -ns 10000 -r 12345
```

Параметризованный лаунчер:

```bash
./ms_final.sh \
  -ns 10000 \
  -r 12345 \
  -p myrun \
  -g 80 \
  -bo 0.35 \
  -ro 0.30
```

Лаунчер для возобновления прогона:

```bash
./ms_resume_allargs.sh \
  -ns 10000 \
  -r 12345 \
  -p myrun \
  -g 80 \
  -bo 0.35 \
  -ro 0.30
```

Дополнительный пример resume/random-state (из command docs):

```bash
./ms_resume_random_state.sh \
  -p iccp100kG20Ov \
  -r 88888 \
  -g 20 \
  -bo 0.35 \
  -ro 0.3 \
  -ns 100000
```

Примечания:
- Лаунчеры запускают `NQ=1..4` параллельно.
- Скрипты вызывают `../build/S4` с `-t 32`.

### B) Объединение выходов S4 и привязка вершин форм

```bash
python merge_s4_data_full.py --prefix myrun
# output: merged_s4_shapes_myrun.csv
```

### C) Нормализация столбцов для совместимости с обучением

`merge_s4_data_full.py` записывает `folder_key` и `NQ`, тогда как путь обучения ожидает `prefix` и `nQ`.

```bash
python -c "import pandas as pd; p='merged_s4_shapes_myrun.csv'; df=pd.read_csv(p); df=df.rename(columns={'folder_key':'prefix','NQ':'nQ'}); df.to_csv(p,index=False)"
```

### D) Предобработка CSV -> NPZ

```bash
mkdir -p merged_csvs
mv merged_s4_shapes_myrun.csv merged_csvs/

python three_stage_transmittance.py \
  --preprocess \
  --input_folder merged_csvs \
  --output_npz preprocessed_t_data.npz
```

### E) Обучение этапов A/B/C

```bash
python three_stage_transmittance.py \
  --data_npz preprocessed_t_data.npz \
  --num_epochs 100 \
  --batch_size 1024
```

Результаты записываются в:
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageA`
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageB`
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageC`

### F) Оценка обученных моделей

Команда из исторического README (имя скрипта сохранено для совместимости с прежней документацией):

```bash
python three_stage_transmittance_evaluation.py \
  --model_dir outputs_three_stage_YYYYMMDD_HHMMSS \
  --data_npz preprocessed_t_data.npz \
  --sample_count 8
```

Примечание о текущем состоянии репозитория: `three_stage_transmittance_evaluation.py` отсутствует в этом снимке. Для доступной функциональности оценки используйте `FilterShapeS4_Evaluator_Transmittance.py`.

### G) Опциональная проверка согласованности нейросети и S4

```bash
python FilterShapeS4_Evaluator_Transmittance.py \
  --npz_file preprocessed_t_data.npz \
  --spec2shape_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageC/spec2shape_stageC.pt \
  --shape2spec_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageA/shape2spec_stageA.pt \
  --n_samples 4
```

## ⚙️ Конфигурация

### S4-лаунчеры (`ms_final.sh`, `ms_resume_allargs.sh`)

| Флаг | Значение | По умолчанию |
|---|---|---|
| `-ns`, `--numshapes` | Количество генерируемых форм | `100000` |
| `-r`, `--seed` | Случайное зерно | `88888` |
| `-p`, `--prefix` | Префикс/ключ возобновления | `""` |
| `-g`, `--numg` | Параметр базиса/сетки | `80` |
| `-bo`, `--baseouter` | Базовое смещение внешней границы | `0.25` |
| `-ro`, `--randouter` | Случайное смещение внешней границы | `0.20` |

### Обучение (`three_stage_transmittance.py`)

| Флаг | Значение | По умолчанию |
|---|---|---|
| `--preprocess` | Запуск режима предобработки | `False` |
| `--input_folder` | Папка с объединёнными CSV-файлами | `""` |
| `--output_npz` | Путь к выходному NPZ | `preprocessed_data.npz` |
| `--data_npz` | NPZ-датасет для обучения | `""` |
| `--csv_file` | Резервный CSV, если NPZ не используется | `""` |
| `--test` | Тестовый режим | `False` |
| `--num_epochs` | Количество эпох обучения | `10` |
| `--batch_size` | Размер батча | `4096` |

### Историческая конфигурация оценки (`three_stage_transmittance_evaluation.py`)

| Флаг | Значение | По умолчанию |
|---|---|---|
| `--model_dir` | Каталог, содержащий `stageA/B/C` | required |
| `--data_npz` | Входной NPZ | `""` |
| `--csv_file` | Резервный входной CSV | `""` |
| `--output_dir` | Переопределение выходного каталога | auto under `model_dir` |
| `--sample_count` | Количество визуализируемых примеров | `4` |
| `--seed` | Случайное зерно | `23` |
| `--font_scale` | Масштаб шрифта графиков | `1.0` |
| `--batch_size` | Размер батча для оценки | `32` |
| `--plot_only` | Только построение кривых обучения | `False` |

### Оценщик согласованности с S4 (`FilterShapeS4_Evaluator_Transmittance.py`)

| Флаг | Значение | По умолчанию |
|---|---|---|
| `--npz_file` | Входной NPZ-файл | `preprocessed_t_data.npz` |
| `--spec2shape_ckpt` | Путь к чекпоинту этапа C | `outputs_three_stage_20250322_145925/stageC/spec2shape_stageC.pt` |
| `--shape2spec_ckpt` | Путь к чекпоинту этапа A | `outputs_three_stage_20250322_145925/stageA/shape2spec_stageA.pt` |
| `--n_samples` | Количество оцениваемых примеров | `4` |
| `--seed` | Случайное зерно | `23` |
| `--max_workers` | Потоки worker для S4 | `4` |
| `--out_folder` | Выходной каталог | auto timestamp |

## 🧪 Примеры

### Smoke run

```bash
./ms_final.sh -ns 1000 -r 42 -p smoke -g 40 -bo 0.25 -ro 0.20
python merge_s4_data_full.py --prefix smoke
python -c "import pandas as pd; p='merged_s4_shapes_smoke.csv'; d=pd.read_csv(p).rename(columns={'folder_key':'prefix','NQ':'nQ'}); d.to_csv(p,index=False)"
mkdir -p merged_csvs && mv merged_s4_shapes_smoke.csv merged_csvs/
python three_stage_transmittance.py --preprocess --input_folder merged_csvs --output_npz smoke.npz
python three_stage_transmittance.py --data_npz smoke.npz --num_epochs 5 --batch_size 128
```

### Примеры профилей запуска (из `commands.md` / `commands_updated.md`)

```bash
# no overlap, G=40
./ms_resume_allargs.sh -ns 10000 -r 12345 -p fast_without_overlap -g 40 -bo 0.25 -ro 0.2

# overlap, G=80
./ms_resume_allargs.sh -ns 10000 -r 12345 -p overlap -g 80 -bo 0.35 -ro 0.3

# large run, overlap, G=80
./ms_resume_allargs.sh -ns 100000 -r 12345 -p more_basis_overlap -g 80 -bo 0.35 -ro 0.3
```

## 🔬 Исследовательский контекст

Текущая схема обратного проектирования учится восстанавливать C4-симметричную геометрию по пропусканию на разных состояниях кристаллизации. Текущий конвейер пропускания предполагает:

- 11 строк кристаллизации на один образец формы (группировка по уникальному `shape_uid`)
- 100 бинов длины волны на каждое состояние кристаллизации (столбцы `T@...`)
- До 4 управляющих точек Q1, закодированных как тензор `4x3`: `(presence, x, y)`
- Восстановление полигона при C4-симметрии для визуализации формы и проверок согласованности

Репозиторий также содержит экспериментальные ветки (`AVIRIS*`, `noise_experiment*`, `archived/`) помимо основного пути обучения по пропусканию.

## 🧑‍💻 Примечания для разработки

- Это исследовательский репозиторий, ориентированный на скрипты, а не упакованный Python-модуль.
- Базовые скрипты предполагают относительные пути (особенно `../build/S4`, `results/`, `shapes/`).
- `.gitignore` исключает многие сгенерированные артефакты экспериментов (`*.csv`, `*.npz`, `*.pt`, run folders).
- Некоторые файлы/каталоги из исторической документации сейчас отсутствуют в этом снимке; эти ссылки намеренно сохранены с примечаниями для совместимости.
- Встречаются sidecar-файлы macOS (`._*`), которые могут быть нефункциональными метаданными.

## 🧯 Устранение неполадок

| Симптом | Вероятная причина | Решение |
|---|---|---|
| `../build/S4: No such file or directory` | Бинарник S4 отсутствует по ожидаемому относительному пути | Соберите или подключите S4 в `../build/S4`, либо обновите пути в лаунчерах |
| `No transmission columns found` | В CSV отсутствуют столбцы `T@...` | Повторно проверьте формат выхода merge |
| `Must specify either --data_npz or --csv_file` | Не передан аргумент с данными для обучения/оценки | Явно укажите один вход |
| `No valid shapes => SHIFT->Q1->UpTo4` | Некорректный/пустой `vertices_str` или фильтрация Q1 удаляет все образцы | Проверьте файлы форм и выход merge |
| Empty merge output for `--prefix` | Префикс не совпадает с файлами в `results/` | Проверьте точный префикс имени файла и повторите merge |
| Evaluation checkpoint missing | Отсутствуют файлы чекпоинтов `stageA/B/C` | Убедитесь, что `--model_dir` указывает на полный выходной каталог |
| `three_stage_transmittance_evaluation.py` not found | Скрипт упоминается в исторической документации, но сейчас отсутствует | Используйте `FilterShapeS4_Evaluator_Transmittance.py` или восстановите скрипт из прошлых коммитов |

## 🗺️ Дорожная карта

- Улучшить воспроизводимость с явным манифестом версий данных и фиксированными конфигурациями запусков.
- Сконсолидировать канонические entry points для веток transmittance, AVIRIS и noise.
- Добавить автоматизированные smoke-тесты для предобработки и одной мини-эпохи обучения.
- Добавить более прозрачный реестр экспериментов, связывающий выходные папки с точными командами запуска.
- Расширить workflow синхронизации многоязычных README (языковые файлы в корне и `i18n/`).

## 🤝 Вклад

Вклад приветствуется, особенно в областях воспроизводимости, тестирования и качества документации.

Рекомендуемый процесс:

1. Откройте issue с областью изменений и ожидаемым поведением.
2. Создайте сфокусированную ветку.
3. Отправьте pull request с воспроизводимыми командами и результатами.
4. По возможности ограничивайте изменения одним рабочим конвейером.

## 📄 Лицензия

В текущем снимке в корне репозитория отсутствует файл `LICENSE`. Добавьте его, чтобы определить условия использования и распространения.

## 📚 Цитирование

Если вы используете этот репозиторий или развиваете эту работу, пожалуйста, цитируйте:

```bibtex
@article{chen2025inverse,
  title={Inverse Design of Metasurface for Spectral Imaging},
  author={Chen, Rongzhou and Nie, Haitao and Zhu, Shuo and Zhao, Yaping and Wang, Chutian and Lam, Edmund Y},
  journal={arXiv preprint arXiv:2510.21924},
  year={2025}
}
```

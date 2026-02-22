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


# Обратное проектирование метаповерхности для спектральной визуализации

[![Status](https://img.shields.io/badge/status-research%20prototype-orange)](#обзор)
[![Python](https://img.shields.io/badge/python-3.9-blue)](#предварительные-требования)
[![Platform](https://img.shields.io/badge/platform-linux-lightgrey)](#предварительные-требования)
[![RCWA](https://img.shields.io/badge/RCWA-S4-required-success)](#предварительные-требования)
[![PyTorch](https://img.shields.io/badge/framework-pytorch-ee4c2c)](#возможности)
[![License](https://img.shields.io/badge/license-not%20specified-lightgrey)](#лицензия)

Исследовательский репозиторий с акцентом на скрипты для обратного проектирования метаповерхностей при симметрии C4, с полным рабочим циклом от моделирования S4/RCWA до нейросетевого обратного моделирования и оценки.

## Обзор

Этот репозиторий (исторически называвшийся `inverse_metasurface`) организован как рабочее пространство для экспериментов, а не как Python-пакет. Наиболее стабильный рабочий процесс:

1. Сгенерировать спектры пропускания и файлы форм с помощью S4 (`.lua` + Bash-лаунчеры).
2. Объединить и развернуть необработанные CSV-результаты симуляций.
3. Предобработать объединенные CSV в компактные тензоры `.npz`.
4. Обучить трехэтапный нейросетевой конвейер:
   - Этап A: shape -> spectrum
   - Этап B: spectrum -> shape
   - Этап C: spectrum -> shape -> spectrum (дообучение с chain loss)
5. Провести оценку по метрикам/графикам и при необходимости выполнить повторную проверку через S4.

## Возможности

- Лаунчеры S4 с воспроизводимыми seed и поддержкой префикса для возобновления запуска.
- Конвейер обработки данных от сырых `results/*.csv` + `shapes/*` до тензоров, готовых к обучению.
- Модели на основе Transformer для прямого и обратного оптического проектирования.
- Поэтапное обучение, кривые валидации и визуализация примеров.
- Скрипт оценки для метрик по этапам и графиков в стиле публикаций.
- Опциональная проверка согласованности нейросети и S4 через RCWA-перезапуск с фиксированными формами.
- Дополнительные ветки экспериментов AVIRIS/шумов в том же репозитории.

## Исследовательский контекст

Центральная задача — обратное проектирование метаповерхностей для спектральной визуализации: предсказание геометрии по целевым спектральным сигнатурам с сохранением физически осмысленной структуры. Код обучения работает с тензорами формы и спектра, сгенерированными из RCWA-результатов для нескольких состояний кристаллизации.

Текущие допущения конвейера пропускания в коде:

- 11 состояний кристаллизации (`c=0.0 ... 1.0`)
- 100 бинов длины волны на состояние
- Форма тензора спектра: `11 x 100`
- Форма тензора геометрии: `4 x 3` (`[presence, x, y]` для максимум 4 точек Q1)

## Структура проекта

```text
.
├── ms.sh / ms_final.sh / ms_resume_allargs.sh
├── metasurface_seed.lua / metasurface_final.lua / metasurface_allargs_resume.lua
├── merge_s4_data_full.py
├── three_stage_transmittance.py
├── three_stage_transmittance_evaluation.py
├── FilterShapeS4_Evaluator_Transmittance.py
├── partial_crys_data/
├── results/                      # необработанные выходы S4
├── shapes/                       # сгенерированные вершины полигонов
├── merged_csvs/                  # объединенные CSV для обучения
├── outputs_three_stage_*/        # чекпоинты + графики этапов
├── AVIRIS*/ and aviris_*.py      # дополнительные гиперспектральные workflows
├── how_to_run.md / commands*.md
├── iccp.yaml
└── pip_requirements.txt
```

## Предварительные требования

- Linux + Bash
- Conda (рекомендуется)
- Python 3.9
- Бинарник S4 доступен по пути `../build/S4` (относительно корня репозитория)
- Опционально GPU с поддержкой CUDA для ускорения обучения

Допущение: сам S4 собирается вне этого репозитория (ожидается в `../build/S4`).

## Установка

```bash
git clone <repo-url> inverse_metasurface
cd inverse_metasurface

conda env create -f iccp.yaml
conda activate iccp

# verify RCWA binary path used by shell launchers
ls -l ../build/S4
```

Опционально:

```bash
chmod +x ms.sh ms_final.sh ms_resume_allargs.sh
```

## Использование

### 1) Запуск генерации данных S4

Пример: режим overlap с пользовательским префиксом.

```bash
./ms_final.sh \
  -ns 10000 \
  -r 12345 \
  -p myrun \
  -g 80 \
  -bo 0.35 \
  -ro 0.30
```

### 2) Объединение выходов S4 и вершин формы

```bash
python merge_s4_data_full.py --prefix myrun
# output: merged_s4_shapes_myrun.csv
```

### 3) Нормализация имен колонок объединенного файла (если нужно)

`three_stage_transmittance.py` ожидает `prefix` и `nQ`, тогда как объединенный файл обычно содержит `folder_key` и `NQ`.

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

### 5) Обучение трехэтапных моделей

```bash
python three_stage_transmittance.py \
  --data_npz preprocessed_t_data.npz \
  --num_epochs 100 \
  --batch_size 1024
```

Выходные файлы создаются в директории с временной меткой:

- `outputs_three_stage_YYYYMMDD_HHMMSS/stageA`
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageB`
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageC`

### 6) Оценка чекпоинтов

```bash
python three_stage_transmittance_evaluation.py \
  --model_dir outputs_three_stage_YYYYMMDD_HHMMSS \
  --data_npz preprocessed_t_data.npz \
  --sample_count 8
```

### 7) Опционально: сравнение нейросети и S4

```bash
python FilterShapeS4_Evaluator_Transmittance.py \
  --npz_file preprocessed_t_data.npz \
  --spec2shape_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageC/spec2shape_stageC.pt \
  --shape2spec_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageA/shape2spec_stageA.pt \
  --n_samples 4
```

## Конфигурация

### Флаги лаунчера S4 (`ms_final.sh`, `ms_resume_allargs.sh`)

- `-ns`, `--numshapes`: число форм (по умолчанию `100000`)
- `-r`, `--seed`: seed генератора случайных чисел (по умолчанию `88888`)
- `-p`, `--prefix`: префикс запуска / ключ возобновления
- `-g`, `--numg`: параметр базиса геометрии (по умолчанию `80`)
- `-bo`, `--baseouter`: базовый внешний offset (по умолчанию `0.25`)
- `-ro`, `--randouter`: случайный внешний offset (по умолчанию `0.20`)

### Флаги обучения (`three_stage_transmittance.py`)

- `--preprocess`: режим предобработки
- `--input_folder`: папка с объединенными CSV-файлами
- `--output_npz`: целевой предобработанный файл
- `--data_npz`: предобработанный датасет для обучения
- `--csv_file`: прямой CSV-вход (альтернатива NPZ)
- `--num_epochs`: количество эпох обучения
- `--batch_size`: размер батча

### Флаги оценки (`three_stage_transmittance_evaluation.py`)

- `--model_dir` (обязательный): корневая папка чекпоинтов
- `--data_npz` или `--csv_file`: датасет для оценки
- `--output_dir`: папка выходов оценки (создается автоматически, если не указана)
- `--sample_count`: число визуализируемых примеров
- `--plot_only`: только перегенерировать графики кривых обучения

## Примеры

### Быстрое обучение из существующего NPZ

```bash
python three_stage_transmittance.py --data_npz preprocessed_t_data.npz --num_epochs 50 --batch_size 2048
```

### Режим только графиков для существующего запуска

```bash
python three_stage_transmittance_evaluation.py \
  --model_dir outputs_three_stage_20250409_123456 \
  --plot_only
```

### Совместимая с возобновлением генерация RCWA

```bash
./ms_resume_allargs.sh -ns 100000 -r 88888 -p myrun -g 80 -bo 0.25 -ro 0.2
```

## Примечания по разработке

- Это исследовательское рабочее пространство с большим количеством исторических/экспериментальных скриптов.
- Канонические скрипты конвейера пропускания:
  - `ms_final.sh` / `ms_resume_allargs.sh`
  - `merge_s4_data_full.py`
  - `three_stage_transmittance.py`
  - `three_stage_transmittance_evaluation.py`
- Репозиторий содержит много данных (крупные папки результатов/кэша). Для совместной работы крупные артефакты лучше хранить во внешнем или игнорируемом хранилище.
- В корне нет метаданных сборки пакета (`pyproject.toml`, `setup.py`).

## Устранение неполадок

- `../build/S4: No such file or directory`
  - Соберите или подключите S4 так, чтобы бинарник был доступен по `../build/S4`, либо обновите скрипты запуска под фактический путь к S4.
- `No matching CSVs found in 'results/'`
  - Убедитесь, что префикс запуска в `merge_s4_data_full.py --prefix <prefix>` совпадает с файлами в `results/`.
- `No transmission columns found`
  - Убедитесь, что объединенный CSV содержит колонки `T@...` из RCWA-выхода.
- Предобработка возвращает ноль записей
  - Проверьте, что объединенный CSV содержит необходимые колонки: `prefix`, `nQ`, `nS`, `shape_idx`, `c`, `vertices_str` и 11 строк на каждый UID формы.
- GPU OOM во время обучения
  - Уменьшите `--batch_size` (например, до `256` или `128`) и повторите запуск.
- Скрипт оценки не находит файлы чекпоинтов
  - Убедитесь, что `--model_dir` содержит:
    - `stageA/shape2spec_stageA.pt`
    - `stageB/spec2shape_stageB.pt`
    - `stageC/spec2shape_stageC.pt`

## Дорожная карта

- Добавить минимальное, жестко закрепленное окружение только для основного конвейера пропускания.
- Добавить единый оркестратор для сценария simulation -> merge -> preprocess -> train -> evaluate.
- Добавить легковесные smoke-тесты для предобработки и загрузки моделей.
- Добавить стандартизованные метаданные датасета/версии для воспроизводимости.
- Разделить конвейеры AVIRIS и metasurface на более понятные подпроекты.

## Вклад

Приветствуются исправления ошибок, улучшения воспроизводимости и улучшения документации.

Рекомендуемый рабочий процесс:

1. Откройте issue с описанием проблемы или предложения.
2. Держите изменения сфокусированными и совместимыми со скриптовыми соглашениями текущего конвейера.
3. При изменении CLI-поведения добавляйте примеры команд и ожидаемые результаты.
4. Если меняете формат данных, добавьте заметки о миграции в `README.md` и связанных скриптах.

## Цитирование

Если этот репозиторий помог вашему исследованию, пожалуйста, процитируйте:

```bibtex
@article{chen2025inverse,
  title={Inverse Design of Metasurface for Spectral Imaging},
  author={Chen, Rongzhou and Nie, Haitao and Zhu, Shuo and Zhao, Yaping and Wang, Chutian and Lam, Edmund Y},
  journal={arXiv preprint arXiv:2510.21924},
  year={2025}
}
```

## Лицензия

На данный момент в этом репозитории отсутствует файл лицензии.

Допущение: по умолчанию все права защищены, пока сопровождающие проекта не добавят лицензию.

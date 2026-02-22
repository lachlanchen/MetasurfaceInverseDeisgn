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

Репозиторий исследовательского типа со ставкой на скрипты (исторически упоминается как `inverse_metasurface`) для обратного дизайна метаповерхностей в задачах спектральной визуализации. Конвейер объединяет **RCWA-моделирование на базе S4** и **трехэтапный PyTorch-процесс** для прямого и обратного отображения между геометрией и спектрами оптического пропускания.

## ✨ Кратко

| Пункт | Детали |
|---|---|
| 🎯 Цель | Предсказание геометрии C4-симметричной метаповерхности по целевым спектрам пропускания |
| 🔬 Физическая модель | RCWA-моделирование с S4 (`../build/S4`) |
| 🧠 Обучающий конвейер | Этап A `shape -> spectra`, этап B `spectra -> shape`, этап C `spectra -> shape -> spectra` |
| 📦 Формат данных | Объединенный CSV (`T@...`, метаданные формы) -> сжатый NPZ (`uids`, `spectra`, `shapes`) |
| 🧪 Оценка | Метрики MSE, качественные графики, опциональная повторная проверка через S4 |

## 🧭 Сквозной рабочий процесс

1. Сгенерировать оптические отклики метаповерхности в S4 (`.lua` + shell-скрипты запуска).
2. Объединить исходные CSV-файлы моделирования и добавить вершины полигонов.
3. Преобразовать объединенные CSV в обучающий NPZ.
4. Обучить трехэтапный конвейер пропускания.
5. Оценить checkpoints и визуализировать поведение этапов A/B/C.
6. При необходимости сравнить нейросетевые предсказания со свежими симуляциями S4.

## 🧱 Структура репозитория

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

## 🛠️ Требования

- Linux + Bash
- Conda (рекомендуется)
- Python 3.9
- Бинарный файл S4 доступен по пути `../build/S4`
- Опционально: GPU с поддержкой CUDA для более быстрого обучения

## 🚀 Установка

```bash
git clone <your-repo-url> inverse_metasurface
cd inverse_metasurface

conda env create -f iccp.yaml
conda activate iccp

# Verify simulator path expected by scripts
ls -l ../build/S4
```

Опционально:

```bash
chmod +x ms.sh ms_final.sh ms_resume_allargs.sh
```

## ▶️ Практическое использование

### 1) Сгенерировать RCWA-данные

`ms_final.sh` и `ms_resume_allargs.sh` запускают по 4 параллельные задания S4 (`NQ=1..4`):

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
- `ms.sh` — более простой вариант запуска.
- Скрипты запуска предполагают путь `../build/S4` и используют `-t 32`.

### 2) Объединить результаты S4 + вершины форм

```bash
python merge_s4_data_full.py --prefix myrun
# output: merged_s4_shapes_myrun.csv
```

### 3) Нормализовать имена столбцов для совместимости с обучением

`merge_s4_data_full.py` записывает `folder_key` / `NQ`, тогда как `three_stage_transmittance.py` ожидает `prefix` / `nQ`.

```bash
python -c "import pandas as pd; p='merged_s4_shapes_myrun.csv'; df=pd.read_csv(p); df=df.rename(columns={'folder_key':'prefix','NQ':'nQ'}); df.to_csv(p,index=False)"
```

### 4) Предобработать CSV в NPZ

```bash
mkdir -p merged_csvs
mv merged_s4_shapes_myrun.csv merged_csvs/

python three_stage_transmittance.py \
  --preprocess \
  --input_folder merged_csvs \
  --output_npz preprocessed_t_data.npz
```

### 5) Обучить трехэтапный конвейер

```bash
python three_stage_transmittance.py \
  --data_npz preprocessed_t_data.npz \
  --num_epochs 100 \
  --batch_size 1024
```

Ожидаемая структура вывода:
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageA`
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageB`
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageC`

### 6) Оценить модели этапов A/B/C

```bash
python three_stage_transmittance_evaluation.py \
  --model_dir outputs_three_stage_YYYYMMDD_HHMMSS \
  --data_npz preprocessed_t_data.npz \
  --sample_count 8
```

### 7) Опционально: проверка согласованности нейросети с S4

```bash
python FilterShapeS4_Evaluator_Transmittance.py \
  --npz_file preprocessed_t_data.npz \
  --spec2shape_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageC/spec2shape_stageC.pt \
  --shape2spec_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageA/shape2spec_stageA.pt \
  --n_samples 4
```

## ⚙️ Ключевые параметры CLI

### Запуск S4 (`ms_final.sh`, `ms_resume_allargs.sh`)

| Флаг | Значение | По умолчанию |
|---|---|---|
| `-ns`, `--numshapes` | Количество генерируемых форм | `100000` |
| `-r`, `--seed` | Случайное зерно | `88888` |
| `-p`, `--prefix` | Префикс/ключ для возобновления | `""` |
| `-g`, `--numg` | Параметр базиса/геометрии | `80` |
| `-bo`, `--baseouter` | Смещение внешней базовой границы | `0.25` |
| `-ro`, `--randouter` | Случайное внешнее смещение | `0.20` |

### Обучение (`three_stage_transmittance.py`)

| Флаг | Назначение | По умолчанию |
|---|---|---|
| `--preprocess` | Запуск режима предобработки | `False` |
| `--input_folder` | Папка с объединенными CSV-файлами | `""` |
| `--output_npz` | Путь к выходному NPZ | `preprocessed_data.npz` |
| `--data_npz` | NPZ, используемый для обучения | `""` |
| `--csv_file` | Резервный CSV, если NPZ не используется | `""` |
| `--test` | Тестовый режим (без обучения) | `False` |
| `--num_epochs` | Количество эпох | `10` |
| `--batch_size` | Размер батча | `4096` |

### Оценка (`three_stage_transmittance_evaluation.py`)

| Флаг | Назначение | По умолчанию |
|---|---|---|
| `--model_dir` | Корневая папка, содержащая `stageA/B/C` | required |
| `--data_npz` | NPZ-вход для оценки | `""` |
| `--csv_file` | CSV-вход для оценки | `""` |
| `--output_dir` | Переопределение папки вывода | auto in `model_dir` |
| `--sample_count` | Число визуализируемых примеров | `4` |
| `--seed` | Случайное зерно для выборки | `23` |
| `--font_scale` | Масштаб шрифта на графиках | `1.0` |
| `--batch_size` | Размер батча для оценки | `32` |
| `--plot_only` | Только построение графиков | `False` |

### Проверка согласованности S4 (`FilterShapeS4_Evaluator_Transmittance.py`)

| Флаг | Назначение | По умолчанию |
|---|---|---|
| `--npz_file` | Предобработанный NPZ-файл | `preprocessed_t_data.npz` |
| `--spec2shape_ckpt` | Checkpoint этапа C | `outputs_three_stage_20250322_145925/stageC/spec2shape_stageC.pt` |
| `--shape2spec_ckpt` | Checkpoint этапа A | `outputs_three_stage_20250322_145925/stageA/shape2spec_stageA.pt` |
| `--n_samples` | Количество проверяемых примеров | `4` |
| `--seed` | Случайное зерно | `23` |
| `--max_workers` | Параллельные воркеры S4 | `4` |
| `--out_folder` | Папка вывода | auto timestamp |

## 🧪 Быстрый smoke-запуск

```bash
./ms_final.sh -ns 1000 -r 42 -p smoke -g 40 -bo 0.25 -ro 0.20
python merge_s4_data_full.py --prefix smoke
python -c "import pandas as pd; p='merged_s4_shapes_smoke.csv'; d=pd.read_csv(p).rename(columns={'folder_key':'prefix','NQ':'nQ'}); d.to_csv(p,index=False)"
python three_stage_transmittance.py --preprocess --input_folder . --output_npz smoke.npz
python three_stage_transmittance.py --data_npz smoke.npz --num_epochs 5 --batch_size 128
```

## 🔬 Исследовательский контекст

Базовая постановка обратного дизайна здесь выводит геометрию C4-симметричной метаповерхности по спектрам пропускания на разных состояниях кристаллизации. Текущий путь для пропускания предполагает:

- 11 состояний кристаллизации на образец (значения `c`, сортируются во время предобработки)
- 100 бинов длины волны на состояние (столбцы `T@...`)
- до 4 вершин Q1, закодированных как `(presence, x, y)`

В репозитории также есть экспериментальные ветки (например, `AVIRIS*`, `noise_experiment_*` и `archived/`) помимо канонического конвейера пропускания.

## 📚 Цитирование

Если вы используете этот репозиторий или развиваете данную работу, пожалуйста, приведите ссылку:

```bibtex
@article{chen2025inverse,
  title={Inverse Design of Metasurface for Spectral Imaging},
  author={Chen, Rongzhou and Nie, Haitao and Zhu, Shuo and Zhao, Yaping and Wang, Chutian and Lam, Edmund Y},
  journal={arXiv preprint arXiv:2510.21924},
  year={2025}
}
```

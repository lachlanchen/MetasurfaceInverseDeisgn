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

Исследовательский репозиторий со скриптовым подходом (исторически назывался `inverse_metasurface`) для обратного проектирования метаповерхностей в спектральной визуализации. Базовый рабочий процесс объединяет:

- физически обоснованное моделирование RCWA (S4 + Lua)
- сборку данных и привязку форм
- трехэтапное обучение в PyTorch (`shape -> spectrum`, `spectrum -> shape`, цепочечная донастройка)
- количественную и качественную оценку, с опциональной проверкой согласованности нейросети и S4

## ✨ Кратко

| Пункт | Описание |
|---|---|
| 🎯 Основная задача | Восстановление C4-симметричной геометрии метаповерхности по целевым спектрам пропускания |
| 🔬 Симулятор | `../build/S4`, вызывается shell-лаунчерами и `.lua`-скриптами |
| 🧠 Пайплайн обучения | Этап A `shape -> spectra`, этап B `spectra -> shape`, этап C `spectra -> shape -> spectra` |
| 📦 Контракт данных | объединенный CSV (`T@...`, метаданные, `vertices_str`) -> сжатый NPZ (`uids`, `spectra`, `shapes`) |
| 🧪 Оценка | Метрики MSE, визуализации этапов, опциональная повторная симуляция в S4 |

## 🧭 Сквозной рабочий процесс

1. Сгенерируйте результаты моделирования в `results/` и файлы полигонов в `shapes/`.
2. Объедините CSV-файлы S4 и добавьте вершины форм.
3. Нормализуйте имена столбцов для совместимости с обучением.
4. Предобработайте объединенные CSV-файлы в NPZ-тензоры.
5. Обучите модели этапов A/B/C.
6. Оцените чекпойнты и визуализируйте поведение.
7. При необходимости сравните спектры предсказанных форм с новыми запусками S4.

## 🧱 Структура репозитория

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

## 🛠️ Требования

| Зависимость | Примечания |
|---|---|
| Linux + Bash | Скрипты запуска ориентированы на выполнение в shell |
| Python 3.9 | Соответствует `iccp.yaml` |
| Conda | Рекомендуется для воспроизводимости |
| Бинарник S4 | Ожидается по пути `../build/S4` |
| CUDA GPU (опционально) | Ускоряет обучение/оценку |

## 🚀 Установка

### 1) Клонируйте репозиторий и перейдите в каталог

```bash
git clone <your-repo-url> inverse_metasurface
cd inverse_metasurface
```

### 2) Создайте окружение (рекомендуется)

```bash
conda env create -f iccp.yaml
conda activate iccp
```

Альтернатива (менее контролируемая):

```bash
pip install -r pip_requirements.txt
```

### 3) Проверьте путь к симулятору, который ожидают скрипты

```bash
ls -l ../build/S4
```

### 4) (Опционально) сделайте лаунчеры исполняемыми

```bash
chmod +x ms.sh ms_final.sh ms_resume.sh ms_resume_allargs.sh ms_resume_random_state.sh
```

## ▶️ Практическое использование

### A) Генерация данных моделирования RCWA

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

Лаунчер для возобновления:

```bash
./ms_resume_allargs.sh \
  -ns 10000 \
  -r 12345 \
  -p myrun \
  -g 80 \
  -bo 0.35 \
  -ro 0.30
```

Примечания:
- Лаунчеры запускают `NQ=1..4` параллельно.
- Скрипты вызывают `../build/S4` с `-t 32`.

### B) Объединение результатов S4 и добавление вершин форм

```bash
python merge_s4_data_full.py --prefix myrun
# output: merged_s4_shapes_myrun.csv
```

### C) Нормализация столбцов для совместимости с обучением

`merge_s4_data_full.py` записывает `folder_key` и `NQ`, тогда как пайплайн обучения ожидает `prefix` и `nQ`.

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

```bash
python three_stage_transmittance_evaluation.py \
  --model_dir outputs_three_stage_YYYYMMDD_HHMMSS \
  --data_npz preprocessed_t_data.npz \
  --sample_count 8
```

### G) Опциональная проверка согласованности нейросети и S4

```bash
python FilterShapeS4_Evaluator_Transmittance.py \
  --npz_file preprocessed_t_data.npz \
  --spec2shape_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageC/spec2shape_stageC.pt \
  --shape2spec_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageA/shape2spec_stageA.pt \
  --n_samples 4
```

## ⚙️ Ключевые CLI-параметры

### Лаунчеры S4 (`ms_final.sh`, `ms_resume_allargs.sh`)

| Flag | Meaning | Default |
|---|---|---|
| `-ns`, `--numshapes` | Number of shapes to generate | `100000` |
| `-r`, `--seed` | Random seed | `88888` |
| `-p`, `--prefix` | Prefix/resume key | `""` |
| `-g`, `--numg` | Basis/grid parameter | `80` |
| `-bo`, `--baseouter` | Base outer boundary offset | `0.25` |
| `-ro`, `--randouter` | Random outer boundary offset | `0.20` |

### Обучение (`three_stage_transmittance.py`)

| Flag | Meaning | Default |
|---|---|---|
| `--preprocess` | Run preprocessing mode | `False` |
| `--input_folder` | Folder containing merged CSV files | `""` |
| `--output_npz` | Output NPZ path | `preprocessed_data.npz` |
| `--data_npz` | NPZ dataset for training | `""` |
| `--csv_file` | CSV fallback if NPZ not used | `""` |
| `--test` | Test mode | `False` |
| `--num_epochs` | Number of training epochs | `10` |
| `--batch_size` | Batch size | `4096` |

### Оценка (`three_stage_transmittance_evaluation.py`)

| Flag | Meaning | Default |
|---|---|---|
| `--model_dir` | Directory containing `stageA/B/C` | required |
| `--data_npz` | NPZ input | `""` |
| `--csv_file` | CSV input fallback | `""` |
| `--output_dir` | Output directory override | auto under `model_dir` |
| `--sample_count` | Number of visualized samples | `4` |
| `--seed` | Random seed | `23` |
| `--font_scale` | Plot font scaling | `1.0` |
| `--batch_size` | Evaluation batch size | `32` |
| `--plot_only` | Plot training curves only | `False` |

### Оценщик согласованности S4 (`FilterShapeS4_Evaluator_Transmittance.py`)

| Flag | Meaning | Default |
|---|---|---|
| `--npz_file` | Input NPZ file | `preprocessed_t_data.npz` |
| `--spec2shape_ckpt` | Stage C checkpoint path | `outputs_three_stage_20250322_145925/stageC/spec2shape_stageC.pt` |
| `--shape2spec_ckpt` | Stage A checkpoint path | `outputs_three_stage_20250322_145925/stageA/shape2spec_stageA.pt` |
| `--n_samples` | Number of evaluated samples | `4` |
| `--seed` | Random seed | `23` |
| `--max_workers` | S4 worker threads | `4` |
| `--out_folder` | Output directory | auto timestamp |

## 🧪 Быстрый прогон

```bash
./ms_final.sh -ns 1000 -r 42 -p smoke -g 40 -bo 0.25 -ro 0.20
python merge_s4_data_full.py --prefix smoke
python -c "import pandas as pd; p='merged_s4_shapes_smoke.csv'; d=pd.read_csv(p).rename(columns={'folder_key':'prefix','NQ':'nQ'}); d.to_csv(p,index=False)"
mkdir -p merged_csvs && mv merged_s4_shapes_smoke.csv merged_csvs/
python three_stage_transmittance.py --preprocess --input_folder merged_csvs --output_npz smoke.npz
python three_stage_transmittance.py --data_npz smoke.npz --num_epochs 5 --batch_size 128
```

## 🔬 Исследовательский контекст

Текущая конфигурация обратного проектирования обучается восстанавливать C4-симметричную геометрию по пропусканию в разных состояниях кристаллизации. Сейчас пайплайн пропускания предполагает:

- 11 строк кристаллизации на один образец формы (группировка по уникальному `shape_uid`)
- 100 бинов длины волны на состояние кристаллизации (столбцы `T@...`)
- до 4 управляющих точек Q1, закодированных как тензор `4x3`: `(presence, x, y)`
- реконструкцию полигона с C4-симметрией для визуализации формы и проверок согласованности

Репозиторий также содержит экспериментальные ветки (`AVIRIS*`, `noise_experiment*`, `archived/`) помимо основного пути обучения по пропусканию.

## 🧯 Устранение неполадок

| Симптом | Вероятная причина | Решение |
|---|---|---|
| `../build/S4: No such file or directory` | Бинарник S4 отсутствует по ожидаемому относительному пути | Соберите или добавьте ссылку на S4 в `../build/S4`, либо обновите пути в лаунчерах |
| `No transmission columns found` | В CSV отсутствуют столбцы `T@...` | Проверьте формат выходного файла после объединения |
| `Must specify either --data_npz or --csv_file` | Не указан аргумент с данными для обучения/оценки | Явно передайте один из входов |
| `No valid shapes => SHIFT->Q1->UpTo4` | Некорректный/пустой `vertices_str` или фильтрация Q1 удаляет все образцы | Проверьте файлы форм и результат объединения |
| Пустой результат объединения для `--prefix` | Префикс не совпадает с файлами в `results/` | Проверьте точный префикс имени файла и запустите объединение снова |
| Отсутствует checkpoint для оценки | Отсутствуют checkpoint-файлы `stageA/B/C` | Убедитесь, что `--model_dir` указывает на полный каталог результатов |

## 🤝 Вклад

Вклад приветствуется, особенно в области воспроизводимости, тестирования и качества документации.

Рекомендуемый процесс:

1. Создайте issue с границами задачи и ожидаемым поведением.
2. Создайте отдельную ветку под конкретную задачу.
3. Отправьте pull request с воспроизводимыми командами и результатами.
4. По возможности ограничивайте изменения одним рабочим процессом.

## 📄 Лицензия

В текущем снимке репозитория в корне отсутствует файл `LICENSE`. Добавьте его, чтобы определить условия использования и распространения.

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

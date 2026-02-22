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

مستودع بحثي يعتمد على السكربتات أولًا (ويُشار إليه تاريخيًا باسم `inverse_metasurface`) من أجل **التصميم العكسي لميتاسطح متناظر C4** في التصوير الطيفي، ويغطي:

- توليد بيانات RCWA باستخدام S4 (`.lua` مع مُشغّلات shell)
- دمج البيانات والمعالجة المسبقة (`.csv` -> `.npz`)
- تدريب عصبي ثلاثي المراحل (shape->spectra، ثم spectra->shape، ثم ضبط دقيق للسلسلة)
- التقييم مع خيار المقارنة بين التنبؤات العصبية وS4

## ✨ نظرة سريعة

| العنصر | التفاصيل |
|---|---|
| الهدف الأساسي | التنبؤ بالهندسة من أطياف النفاذية المستهدفة |
| شكل مجموعة البيانات الأساسية | spectra: `11 x 100`، shape: `4 x 3` |
| سكربت التدريب الرئيسي | `three_stage_transmittance.py` |
| سكربت التقييم الرئيسي | `three_stage_transmittance_evaluation.py` |
| مشغّل RCWA | `ms_final.sh`, `ms_resume_allargs.sh` |
| سكربت الدمج | `merge_s4_data_full.py` |

## 🧠 السياق البحثي

يركّز هذا المشروع على التصميم العكسي للميتاسطوح في التصوير الطيفي. يستخدم خط التدريب أطياف نفاذية مولّدة عبر S4 على حالات تبلور مختلفة، ويتعلم كلًا من التحويل الأمامي والعكسي:

1. **المرحلة A**: shape -> spectra
2. **المرحلة B**: spectra -> shape
3. **المرحلة C**: spectra -> shape -> spectra (ضبط دقيق عبر chain loss)

يفترض كود المعالجة المسبقة/التدريب حاليًا ما يلي:

- 11 حالة تبلور (`c = 0.0 ... 1.0`)
- 100 خانة طول موجي لكل حالة
- تمثيل الشكل كحد أقصى 4 نقاط Q1 بصيغة `[presence, x, y]`

## 🗂️ بنية المستودع (المسار الأساسي)

```text
.
├── ms.sh / ms_final.sh / ms_resume_allargs.sh
├── metasurface_final.lua / metasurface_allargs_resume.lua / metasurface_seed.lua
├── merge_s4_data_full.py
├── three_stage_transmittance.py
├── three_stage_transmittance_evaluation.py
├── FilterShapeS4_Evaluator_Transmittance.py
├── results/                # ملفات CSV الخام من S4
├── shapes/                 # رؤوس المضلعات المُولّدة
├── merged_csvs/            # ملفات CSV المدمجة المستخدمة في المعالجة المسبقة
├── outputs_three_stage_*/  # checkpoints, losses, visualizations
├── partial_crys_data/
└── iccp.yaml
```

## ⚙️ المتطلبات المسبقة

| الاعتماد | المتطلب |
|---|---|
| نظام التشغيل | Linux |
| الطرفية | Bash |
| Python | 3.9 |
| مدير البيئة | Conda (موصى به) |
| ملف RCWA التنفيذي | `../build/S4` (نسبيًا إلى جذر المستودع) |
| GPU | اختياري، وموصى به لتدريب أسرع |

## 🚀 الإعداد

```bash
git clone <repo-url> inverse_metasurface
cd inverse_metasurface

conda env create -f iccp.yaml
conda activate iccp

# Required by launcher scripts
ls -l ../build/S4
```

اختياري:

```bash
chmod +x ms.sh ms_final.sh ms_resume_allargs.sh
```

## 🧪 الاستخدام الكامل من البداية للنهاية

### 1) توليد بيانات RCWA عبر S4

```bash
./ms_final.sh \
  -ns 10000 \
  -r 12345 \
  -p myrun \
  -g 80 \
  -bo 0.35 \
  -ro 0.30
```

ملاحظات:

- تستدعي المشغلات `../build/S4` مع `-t 32` وتُشغّل `NQ=1..4` بالتوازي.
- يستخدم `ms_final.sh` الملف `metasurface_final.lua`.
- يستخدم `ms_resume_allargs.sh` الملف `metasurface_allargs_resume.lua`.

### 2) دمج مخرجات RCWA مع رؤوس الأشكال

```bash
python merge_s4_data_full.py --prefix myrun
# -> merged_s4_shapes_myrun.csv
```

### 3) توحيد أسماء الأعمدة للتدريب (عند الحاجة)

يكتب `merge_s4_data_full.py` عمودي `folder_key` / `NQ`، بينما يتوقع خط التدريب `prefix` / `nQ`.

```bash
python -c "import pandas as pd; p='merged_s4_shapes_myrun.csv'; df=pd.read_csv(p); df=df.rename(columns={'folder_key':'prefix','NQ':'nQ'}); df.to_csv(p,index=False)"
```

### 4) المعالجة المسبقة من CSV إلى NPZ

```bash
mkdir -p merged_csvs
mv merged_s4_shapes_myrun.csv merged_csvs/

python three_stage_transmittance.py \
  --preprocess \
  --input_folder merged_csvs \
  --output_npz preprocessed_t_data.npz
```

### 5) تدريب نماذج المراحل الثلاث

```bash
python three_stage_transmittance.py \
  --data_npz preprocessed_t_data.npz \
  --num_epochs 100 \
  --batch_size 1024
```

هيكل المخرجات الرئيسي:

- `outputs_three_stage_YYYYMMDD_HHMMSS/stageA`
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageB`
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageC`

### 6) تقييم checkpoints

```bash
python three_stage_transmittance_evaluation.py \
  --model_dir outputs_three_stage_YYYYMMDD_HHMMSS \
  --data_npz preprocessed_t_data.npz \
  --sample_count 8
```

### 7) اختياري: مقارنة التنبؤات العصبية مع S4

```bash
python FilterShapeS4_Evaluator_Transmittance.py \
  --npz_file preprocessed_t_data.npz \
  --spec2shape_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageC/spec2shape_stageC.pt \
  --shape2spec_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageA/shape2spec_stageA.pt \
  --n_samples 4
```

## 🎛️ مرجع سطر الأوامر (CLI)

### خيارات مشغّل S4 (`ms_final.sh`, `ms_resume_allargs.sh`)

| الخيار | المعنى | الافتراضي |
|---|---|---|
| `-ns`, `--numshapes` | عدد الأشكال | `100000` |
| `-r`, `--seed` | البذرة العشوائية | `88888` |
| `-p`, `--prefix` | بادئة التشغيل / مفتاح الاستئناف | `""` |
| `-g`, `--numg` | معامل أساس الهندسة | `80` |
| `-bo`, `--baseouter` | إزاحة الحافة الخارجية الأساسية | `0.25` |
| `-ro`, `--randouter` | إزاحة خارجية عشوائية | `0.20` |

### خيارات التدريب (`three_stage_transmittance.py`)

| الخيار | الغرض |
|---|---|
| `--preprocess` | تشغيل وضع المعالجة المسبقة |
| `--input_folder` | مجلد ملفات CSV المدمجة |
| `--output_npz` | اسم ملف NPZ الناتج بعد المعالجة |
| `--data_npz` | مجموعة بيانات NPZ للتدريب |
| `--csv_file` | بديل مجموعة بيانات CSV |
| `--test` | وضع الاختبار |
| `--num_epochs` | عدد عصور التدريب |
| `--batch_size` | حجم الدفعة |

### خيارات التقييم (`three_stage_transmittance_evaluation.py`)

| الخيار | الغرض |
|---|---|
| `--model_dir` | الدليل الجذر للـ checkpoint (مطلوب) |
| `--data_npz` / `--csv_file` | مصدر بيانات التقييم |
| `--output_dir` | مجلد مخرجات التقييم |
| `--sample_count` | عدد العينات المعروضة بصريًا |
| `--seed` | البذرة العشوائية لاختيار العينات |
| `--font_scale` | مقياس خط الرسومات |
| `--batch_size` | حجم دفعة التقييم |
| `--plot_only` | إعادة توليد منحنيات التدريب فقط |

## 🧾 عقد البيانات (للمعالجة المسبقة)

مسار المعالجة المسبقة في `three_stage_transmittance.py` يتوقع ملفات CSV مدمجة تحتوي على:

- أعمدة المعرّف: `prefix`, `nQ`, `nS`, `shape_idx`, `c`
- نص الهندسة: `vertices_str`
- أعمدة الطيف: `T@...`

فحوصات الجودة المطبقة في الكود:

- التجميع بواسطة `shape_uid = prefix_nQ_nS_shape_idx`
- يجب أن تحتوي كل مجموعة على 11 صفًا بالضبط
- يتم الاحتفاظ فقط بالأشكال ذات عدد نقاط Q1 ضمن `[1, 4]`

## 🛠️ استكشاف الأخطاء وإصلاحها

- `../build/S4: No such file or directory`
  - أنشئ/اربط S4 في `../build/S4` أو عدّل سكربتات التشغيل إلى مسار S4 الفعلي لديك.
- `No matching CSVs found in 'results/'`
  - تحقّق من `--prefix` ونمط التسمية في `results/*_output_nQ*_nS*.csv`.
- `No transmission columns found`
  - تأكّد أن ملف CSV المدمج يحتوي على أعمدة `T@...`.
- تعطي المعالجة المسبقة صفر سجلات
  - تحقّق من الأعمدة المطلوبة وأن كل shape UID لديه 11 صفًا لحالات التبلور.
- نفاد ذاكرة GPU أثناء التدريب
  - قلّل `--batch_size` (مثلًا `256` أو `128`).
- لا يستطيع التقييم العثور على checkpoints
  - تأكّد من وجود التالي داخل `--model_dir`:
    - `stageA/shape2spec_stageA.pt`
    - `stageB/spec2shape_stageB.pt`
    - `stageC/spec2shape_stageC.pt`

## 📚 الاستشهاد

إذا ساهم هذا المستودع في بحثك، يُرجى الاستشهاد به كما يلي:

```bibtex
@article{chen2025inverse,
  title={Inverse Design of Metasurface for Spectral Imaging},
  author={Chen, Rongzhou and Nie, Haitao and Zhu, Shuo and Zhao, Yaping and Wang, Chutian and Lam, Edmund Y},
  journal={arXiv preprint arXiv:2510.21924},
  year={2025}
}
```

## 🌐 نسخ اللغات

تتوفر في هذا المستودع نسخ إضافية من README، منها:

- `README.en.md`, `README.de.md`, `README.es.md`, `README.fr.md`
- `README.ru.md`, `README.ja.md`, `README.ko.md`, `README.vi.md`
- `README.ar.md`, `README.zh-CN.md`, `README.zh-TW.md`

## 📌 ملاحظات

- هذه مساحة عمل بحثية تضم العديد من السكربتات المؤرشفة والاستكشافية.
- المسار القياسي الخاص بالنفاذية يتمحور حول:
  - `ms_final.sh` / `ms_resume_allargs.sh`
  - `merge_s4_data_full.py`
  - `three_stage_transmittance.py`
  - `three_stage_transmittance_evaluation.py`
- لا يوجد حاليًا ملف ترخيص صريح معرّف في جذر المستودع.

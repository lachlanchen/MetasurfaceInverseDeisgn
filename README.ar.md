English | 中文繁體 | 中文简体 | 日本語 | 한국어 | Tiếng Việt | العربية | Français | Español | Deutsch | Русский

# inverse_metasurface

<div align="center">

[![Status](https://img.shields.io/badge/status-experimental-orange)](#-roadmap)
[![Python](https://img.shields.io/badge/python-3.9-blue)](#-prerequisites)
[![Platform](https://img.shields.io/badge/platform-linux-lightgrey)](#-prerequisites)
[![S4](https://img.shields.io/badge/S4-required-success)](#-prerequisites)
[![License](https://img.shields.io/badge/license-unset-lightgrey)](#-license)

</div>

مساحة عمل بحثية لتصميم **metasurface عكسي** باستخدام **محاكاة S4/RCWA** و**نماذج عصبية متعددة المراحل**.

يدعم المستودع:
- توليد بيانات S4 عالي الإنتاجية لأسطح metasurface متعددة الأضلاع ذات تناظر C4.
- دمج البيانات + المعالجة المسبقة إلى ملفات NPZ جاهزة للتدريب.
- تعلم من ثلاث مراحل: **shape -> spectrum** و**spectrum -> shape** و**chain-tuned spectrum -> shape -> spectrum**.
- التقييم مع مقارنة اختيارية مباشرة بين **neural-vs-S4**.

## 🌟 نظرة سريعة

| المجال | ما الذي يقدمه هذا المستودع |
|---|---|
| المحاكاة الفيزيائية | سكربتات Bash + Lua لتشغيل S4 بالتوازي عبر `nQ=1..4` |
| أدوات البيانات | سكربتات دمج تربط رؤوس المضلعات مع الأطياف المرجعية |
| خط أنابيب التعلم الآلي | `three_stage_transmittance.py` مع وضعي المعالجة المسبقة + التدريب |
| التقييم | `three_stage_transmittance_evaluation.py` مع المقاييس + الرسوم |
| الفرع البحثي | تجارب AVIRIS/الطيف فائق الدقة وتجارب الضوضاء/الضغط |

## 🧠 السياق البحثي

يستهدف هذا المستودع التصميم العكسي للـ photonic metasurfaces: استنتاج الهندسة من الأطياف المطلوبة (وبالعكس).

الافتراضات الأساسية المستخدمة في خط الأنابيب المرجعي:
- تناظر C4 عبر تمثيل نقاط Q1.
- 11 حالة تبلور (`c` ضمن `[0.0, 1.0]`).
- يُخزَّن طيف كل عينة كصفوف نفاذية `11 x 100`.
- تُمثَّل الأشكال بما يصل إلى 4 نقاط Q1 (موتر `4 x 3`: الوجود، x، y).

منطق التدريب ثلاثي المراحل:
1. **Stage A**: shape -> spectrum (`shape2spec_stageA.pt`)
2. **Stage B**: spectrum -> shape (`spec2shape_stageB.pt`)
3. **Stage C**: spectrum -> shape -> spectrum chain fine-tuning (`spec2shape_stageC.pt`)

## 🗂 هيكل المشروع

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

## ✅ المتطلبات المسبقة

| المتطلب | ملاحظات |
|---|---|
| نظام التشغيل | Linux (السكربتات تفترض Bash ومسارات Linux) |
| Python | الإصدار 3.9 (وفق `iccp.yaml`) |
| مدير البيئة | يُنصح باستخدام Conda |
| ملف S4 التنفيذي | متوقَّع في `../build/S4` نسبةً إلى جذر المستودع |
| GPU (اختياري) | CUDA يسرّع التدريب/التقييم |

## ⚙️ التثبيت

### 1) استنساخ المستودع والدخول إليه

```bash
git clone <your-repo-url> inverse_metasurface
cd inverse_metasurface
```

### 2) إنشاء البيئة

```bash
conda env create -f iccp.yaml
conda activate iccp
```

### 3) التحقق من مسار S4

```bash
ls -l ../build/S4
```

إذا لم يكن هذا المسار موجودًا، فإما أن تقوم ببناء/وضع S4 هناك أو تعدّل مسارات السكربتات.

## 🚀 بداية سريعة (من البداية إلى النهاية)

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

## 🧪 تفاصيل الاستخدام

### A) توليد S4

تشغيل أساسي مع seed:

```bash
./ms.sh -ns 10000 -r 88888
```

تشغيل بمعاملات مخصّصة:

```bash
./ms_final.sh \
  -ns 100000 \
  -r 88888 \
  -p iccpOv100kG80 \
  -g 80 \
  -bo 0.35 \
  -ro 0.30
```

تشغيل بأسلوب الاستكمال (resume):

```bash
./ms_resume_allargs.sh \
  -ns 100000 \
  -r 88888 \
  -p iccpOv100kG80 \
  -g 80 \
  -bo 0.35 \
  -ro 0.30
```

### B) دمج ملفات CSV الخام من S4

```bash
python merge.py --prefix 20250123_155420
```

أداة بديلة:

```bash
python merge_s4_data_full.py --prefix 20250123_155420
```

### C) المعالجة المسبقة لملفات CSV المدمجة -> NPZ

```bash
python three_stage_transmittance.py --preprocess \
  --input_folder merged_csvs \
  --output_npz preprocessed_t_data.npz
```

### D) التدريب

```bash
python three_stage_transmittance.py \
  --data_npz preprocessed_t_data.npz \
  --num_epochs 100 \
  --batch_size 1024
```

تُخزَّن مخرجات التدريب في:
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageA`
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageB`
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageC`

### E) التقييم

```bash
python three_stage_transmittance_evaluation.py \
  --model_dir outputs_three_stage_YYYYMMDD_HHMMSS \
  --data_npz preprocessed_t_data.npz \
  --sample_count 8
```

يولّد:
- `evaluation_metrics.csv`
- `metrics_summary.csv`
- تصورات المراحل (`.png`, `.pdf`)
- رسوم منحنيات التدريب

### F) مقارنة neural مع S4 المباشر (اختياري)

```bash
python FilterShapeS4_Evaluator_Transmittance.py \
  --npz_file preprocessed_t_data.npz \
  --spec2shape_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageC/spec2shape_stageC.pt \
  --shape2spec_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageA/shape2spec_stageA.pt \
  --n_samples 4
```

## 🔧 خيارات CLI الأساسية

### أعلام مشغّل S4 (`ms_final.sh`, `ms_resume_allargs.sh`)

| Flag | المعنى | الافتراضي |
|---|---|---|
| `-ns`, `--numshapes` | عدد الأشكال | `100000` |
| `-r`, `--seed` | البذرة العشوائية | `88888` |
| `-p`, `--prefix` | بادئة التشغيل / مفتاح الاستكمال | فارغ |
| `-g`, `--numg` | معامل الهندسة/الأساس | `80` |
| `-bo`, `--baseouter` | إزاحة الحافة الخارجية الأساسية | `0.25` |
| `-ro`, `--randouter` | إزاحة عشوائية للحافة الخارجية | `0.20` |

### أعلام التدريب (`three_stage_transmittance.py`)

| Flag | المعنى | الافتراضي |
|---|---|---|
| `--preprocess` | تشغيل وضع المعالجة المسبقة | معطّل |
| `--input_folder` | مجلد CSV الإدخالي | فارغ |
| `--output_npz` | مسار NPZ الناتج | `preprocessed_data.npz` |
| `--data_npz` | ملف NPZ المستخدم للتدريب/التقييم | فارغ |
| `--csv_file` | ملف CSV مستخدم مباشرة | فارغ |
| `--num_epochs` | عدد العصور لكل مرحلة | `10` |
| `--batch_size` | حجم الدفعة | `4096` |
| `--test` | عنصر placeholder لوضع الاختبار | معطّل |

### أعلام التقييم (`three_stage_transmittance_evaluation.py`)

| Flag | المعنى | الافتراضي |
|---|---|---|
| `--model_dir` | الدليل الجذري لعملية التدريب | مطلوب |
| `--data_npz` | ملف NPZ للتقييم | فارغ |
| `--csv_file` | ملف CSV للتقييم | فارغ |
| `--output_dir` | مجلد المخرجات | `model_dir/evaluation_<timestamp>` |
| `--sample_count` | عدد العينات التي تُعرض بصريًا | `4` |
| `--seed` | بذرة أخذ العينات | `23` |
| `--font_scale` | مقياس الخط في الرسوم | `1.0` |
| `--batch_size` | حجم دفعة التقييم | `32` |
| `--plot_only` | رسم المنحنيات فقط | معطّل |

## 🧭 استكشاف الأخطاء وإصلاحها

| العَرَض | السبب المحتمل | الحل |
|---|---|---|
| `../build/S4: No such file or directory` | عدم تطابق مسار ملف S4 التنفيذي | ابنِ/ضع S4 في `../build/S4` أو عدّل السكربتات |
| `Must specify either --data_npz or --csv_file` | مصدر بيانات مفقود | مرّر أحد هذين الخيارين صراحةً |
| `No matching CSVs found in 'results/'` | عدم تطابق البادئة | تحقّق من البادئة وتسمية المخرجات |
| عدد سجلات المعالجة المسبقة قليل جدًا أو صفر | صفوف `T@...` أو `vertices_str` مفقودة/غير صالحة | تحقّق من مخطط CSV المدمج وتجميع الصفوف لكل شكل |
| CUDA OOM | حجم الدفعة كبير جدًا | خفّض `--batch_size` (مثلًا `1024 -> 256`) |

## 🧱 ملاحظات تطوير

- هذا مستودع ذو طابع تجريبي؛ كثير من المخرجات المولَّدة غير متتبّعة عمدًا.
- توجد عدة نسخ سكربتات لمهام متشابهة (`merge_*`, `aviris_*`, `noise_experiment_*`).
- مسار العمل المرجعي للتصميم العكسي للـ metasurface هو الموثّق في هذا README.
- سكربتات التدريب تضبط البذور (`42`)، لكن الحتمية الصارمة ما زالت تعتمد على العتاد/سلوك الـ backend.
- لا يوجد حاليًا CI موحّد + مجموعة اختبارات آلية كاملة.

## 🛣 خارطة الطريق

- إضافة مجموعة بيانات معيارية صغيرة لاختبارات smoke السريعة.
- توحيد سكربتات خط الأنابيب المكررة.
- إضافة فحوصات لسلامة الدمج/المعالجة المسبقة وقابلية تحميل checkpoints.
- إضافة CI لفحوصات lint + smoke train/eval.
- توثيق بناء S4/تثبيت الإصدار بشكل أوضح.

## 🤝 المساهمة

1. أنشئ فرع ميزة (feature branch).
2. أبقِ نطاق الـ PR ضيقًا (جانب واحد من pipeline/experiment لكل PR).
3. أدرج أوامر قابلة للتشغيل بدقة ومسارات المخرجات المتوقعة.
4. تجنّب رفع مخرجات مولَّدة كبيرة إلا عند الحاجة.
5. أضف ملاحظات القابلية لإعادة الإنتاج (seed، مصدر البيانات، مسار checkpoint).

## 📄 الترخيص

لا يوجد حاليًا ملف `LICENSE` في هذا المستودع.

إلى أن يُضاف ملف ترخيص، ينبغي اعتبار شروط إعادة الاستخدام وإعادة التوزيع **غير محددة**.

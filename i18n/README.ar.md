[English](../README.md) · [العربية](README.ar.md) · [Español](README.es.md) · [Français](README.fr.md) · [日本語](README.ja.md) · [한국어](README.ko.md) · [Tiếng Việt](README.vi.md) · [中文 (简体)](README.zh-Hans.md) · [中文（繁體）](README.zh-Hant.md) · [Deutsch](README.de.md) · [Русский](README.ru.md)


# التصميم العكسي للميتا-سطح للتصوير الطيفي

<p align="center">
  <img alt="Status" src="https://img.shields.io/badge/Status-Research%20Prototype-f59e0b?style=for-the-badge">
  <img alt="Python" src="https://img.shields.io/badge/Python-3.9-3776AB?style=for-the-badge&logo=python&logoColor=white">
  <img alt="PyTorch" src="https://img.shields.io/badge/PyTorch-2.x-EE4C2C?style=for-the-badge&logo=pytorch&logoColor=white">
  <img alt="Simulator" src="https://img.shields.io/badge/Simulator-S4%20RCWA-16a34a?style=for-the-badge">
  <img alt="Platform" src="https://img.shields.io/badge/Platform-Linux%20%2B%20Bash-6b7280?style=for-the-badge&logo=gnu-bash&logoColor=white">
  <img alt="ArXiv" src="https://img.shields.io/badge/arXiv-2510.21924-b31b1b?style=for-the-badge&logo=arxiv&logoColor=white">
</p>

مستودع بحثي يعتمد أسلوب السكربتات أولًا (وكان يُشار إليه تاريخيًا باسم `inverse_metasurface`) للتصميم العكسي للميتا-سطح في التصوير الطيفي.

يربط مسار العمل الأساسي بين:
- محاكاة RCWA المرتكزة على الفيزياء (`S4` + Lua)
- تجميع البيانات وإرفاق الشكل الهندسي
- تعلّم PyTorch ثلاثي المراحل (`shape -> spectrum`، `spectrum -> shape`، وضبط دقيق متسلسل)
- تقييم كمي ونوعي، مع خيار فحص الاتساق بين النموذج العصبي وS4

> [!IMPORTANT]
> تم الحفاظ على السلوك والأوامر المرجعية كما في السكربتات/الوثائق الحالية للمشروع. وعندما تشير المراجع التاريخية إلى ملفات غير موجودة، تُترك هذه الإشارات عمدًا مع ملاحظات صريحة للحفاظ على التوافق.

## 📑 المحتويات

- [✨ لمحة سريعة](#-لمحة-سريعة)
- [🌍 تعدد اللغات (i18n)](#-تعدد-اللغات-i18n)
- [✨ الميزات](#-الميزات)
- [🧭 مسار العمل من البداية إلى النهاية](#-مسار-العمل-من-البداية-إلى-النهاية)
- [🧱 بنية المشروع](#-بنية-المشروع)
- [🛠️ المتطلبات المسبقة](#️-المتطلبات-المسبقة)
- [🚀 التثبيت](#-التثبيت)
- [▶️ الاستخدام](#️-الاستخدام)
- [⚙️ الإعداد](#️-الإعداد)
- [🧪 أمثلة](#-أمثلة)
- [🔬 السياق البحثي](#-السياق-البحثي)
- [🧑‍💻 ملاحظات التطوير](#-ملاحظات-التطوير)
- [🧯 استكشاف الأخطاء وإصلاحها](#-استكشاف-الأخطاء-وإصلاحها)
- [🗺️ خارطة الطريق](#️-خارطة-الطريق)
- [🤝 المساهمة](#-المساهمة)
- [📄 الترخيص](#-الترخيص)
- [📚 الاستشهاد](#-الاستشهاد)

## ✨ لمحة سريعة

| العنصر | التفاصيل |
|---|---|
| 🎯 المهمة الرئيسية | استنتاج هندسة ميتا-سطح ذات تناظر C4 من أطياف النفاذية الهدف |
| 🔬 المحاكي | `../build/S4` ويُستدعى عبر مشغلات shell وسكربتات `.lua` |
| 🧠 مسار التعلّم | المرحلة A `shape -> spectra`، المرحلة B `spectra -> shape`، المرحلة C `spectra -> shape -> spectra` |
| 📦 عقد البيانات | CSV مدمج (`T@...`، وmetadata، و`vertices_str`) -> NPZ مضغوط (`uids`، `spectra`، `shapes`) |
| 🧪 التقييم | مقاييس MSE، ورسوم مراحل التدريب، مع خيار إعادة محاكاة S4 جديدة |
| 🌐 حالة i18n | ملفات README متعددة اللغات على مستوى الجذر + مجلد `i18n/` موجود |

## 🌍 تعدد اللغات (i18n)

- تتم صيانة ملفات README متعددة اللغات على جذر المستودع بصيغة `README.<lang>.md`.
- مجلد `i18n/` موجود في هذه اللقطة من المستودع.
- يحافظ هذا الملف على سطر خيارات لغة واحد في الأعلى لتجنب تكرار شريط اللغات.
- الملف `README.en.md` موجود أيضًا في المستودع؛ ويظل هذا `README.md` هو الأساس المرجعي لهذا التحديث.

## ✨ الميزات

- مسار تصميم عكسي كامل من مخرجات محاكاة S4 إلى نموذج عكسي مُدرّب.
- بارامتريّة مضلع متناظر C4 وترميز نقاط Q1 (`4x3`: `presence, x, y`).
- تدريب نموذج ثلاثي المراحل ضمن سكربت واحد (`three_stage_transmittance.py`).
- أدوات دمج تحافظ على الدقة الطيفية وتربط الرؤوس بكل شكل.
- مُقيِّم اختياري يقارن تنبؤات النموذج بمحاكاة S4 جديدة.
- فروع استكشافية واسعة (AVIRIS، SWIR/noise، GSST، ومتغيرات archived/deprecated).

## 🧭 مسار العمل من البداية إلى النهاية

1. توليد مخرجات المحاكاة في `results/` وملفات المضلعات في `shapes/`.
2. دمج ملفات CSV الصادرة من S4 وإرفاق رؤوس الأشكال.
3. توحيد أسماء الأعمدة لتوافق مسار التدريب.
4. معالجة ملفات CSV المدمجة مسبقًا إلى موترات NPZ.
5. تدريب نماذج المراحل A/B/C.
6. تقييم نقاط الحفظ (checkpoints) وتصور السلوك.
7. اختياريًا مقارنة أطياف الأشكال المتنبأ بها مقابل تشغيلات S4 جديدة.

## 🧱 بنية المشروع

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

## 🛠️ المتطلبات المسبقة

| الاعتمادية | الملاحظات |
|---|---|
| Linux + Bash | سكربتات التشغيل موجهة لتنفيذ shell |
| Python 3.9 | يطابق `iccp.yaml` (`python=3.9.18`) |
| Conda | موصى به لقابلية إعادة الإنتاج |
| S4 binary | متوقع في `../build/S4` |
| CUDA GPU (اختياري) | يسرّع التدريب/التقييم |

## 🚀 التثبيت

### 1) استنساخ المستودع والدخول إليه

```bash
git clone <your-repo-url> inverse_metasurface
cd inverse_metasurface
```

### 2) إنشاء البيئة (موصى به)

```bash
conda env create -f iccp.yaml
conda activate iccp
```

ملاحظة بديلة:

```bash
# Historical README reference (file may be absent in this snapshot)
pip install -r pip_requirements.txt
```

### 3) التحقق من مسار المحاكي المتوقع في السكربتات

```bash
ls -l ../build/S4
```

### 4) (اختياري) جعل سكربتات التشغيل قابلة للتنفيذ

```bash
chmod +x ms.sh ms_final.sh ms_resume.sh ms_resume_allargs.sh ms_resume_random_state.sh ms_resume_random_state_nir.sh
```

## ▶️ الاستخدام

### A) توليد بيانات محاكاة RCWA

مشغّل بسيط:

```bash
./ms.sh -ns 10000 -r 12345
```

مشغّل بمعاملات:

```bash
./ms_final.sh \
  -ns 10000 \
  -r 12345 \
  -p myrun \
  -g 80 \
  -bo 0.35 \
  -ro 0.30
```

مشغّل موجّه للاستئناف:

```bash
./ms_resume_allargs.sh \
  -ns 10000 \
  -r 12345 \
  -p myrun \
  -g 80 \
  -bo 0.35 \
  -ro 0.30
```

مثال إضافي للاستئناف/الحالة العشوائية (من وثائق الأوامر):

```bash
./ms_resume_random_state.sh \
  -p iccp100kG20Ov \
  -r 88888 \
  -g 20 \
  -bo 0.35 \
  -ro 0.3 \
  -ns 100000
```

ملاحظات:
- المشغلات تنفذ `NQ=1..4` بالتوازي.
- السكربتات تستدعي `../build/S4` مع `-t 32`.

### B) دمج مخرجات S4 وإرفاق رؤوس الأشكال

```bash
python merge_s4_data_full.py --prefix myrun
# output: merged_s4_shapes_myrun.csv
```

### C) توحيد الأعمدة للتوافق مع التدريب

يكتب `merge_s4_data_full.py` العمودين `folder_key` و`NQ`، بينما يتوقع مسار التدريب `prefix` و`nQ`.

```bash
python -c "import pandas as pd; p='merged_s4_shapes_myrun.csv'; df=pd.read_csv(p); df=df.rename(columns={'folder_key':'prefix','NQ':'nQ'}); df.to_csv(p,index=False)"
```

### D) المعالجة المسبقة CSV -> NPZ

```bash
mkdir -p merged_csvs
mv merged_s4_shapes_myrun.csv merged_csvs/

python three_stage_transmittance.py \
  --preprocess \
  --input_folder merged_csvs \
  --output_npz preprocessed_t_data.npz
```

### E) تدريب المرحلة A/B/C

```bash
python three_stage_transmittance.py \
  --data_npz preprocessed_t_data.npz \
  --num_epochs 100 \
  --batch_size 1024
```

تُكتب المخرجات إلى:
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageA`
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageB`
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageC`

### F) تقييم النماذج المدرّبة

أمر README تاريخي (تم الإبقاء على اسم السكربت للتوافق مع الوثائق السابقة):

```bash
python three_stage_transmittance_evaluation.py \
  --model_dir outputs_three_stage_YYYYMMDD_HHMMSS \
  --data_npz preprocessed_t_data.npz \
  --sample_count 8
```

ملاحظة حالة المستودع: `three_stage_transmittance_evaluation.py` غير موجود في هذه اللقطة. استخدم `FilterShapeS4_Evaluator_Transmittance.py` لوظائف التقييم المتاحة.

### G) فحص اتساق اختياري بين النموذج العصبي وS4

```bash
python FilterShapeS4_Evaluator_Transmittance.py \
  --npz_file preprocessed_t_data.npz \
  --spec2shape_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageC/spec2shape_stageC.pt \
  --shape2spec_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageA/shape2spec_stageA.pt \
  --n_samples 4
```

## ⚙️ الإعداد

### مشغلات S4 (`ms_final.sh`, `ms_resume_allargs.sh`)

| Flag | المعنى | القيمة الافتراضية |
|---|---|---|
| `-ns`, `--numshapes` | عدد الأشكال المراد توليدها | `100000` |
| `-r`, `--seed` | البذرة العشوائية | `88888` |
| `-p`, `--prefix` | مفتاح البادئة/الاستئناف | `""` |
| `-g`, `--numg` | معامل الأساس/الشبكة | `80` |
| `-bo`, `--baseouter` | إزاحة الحد الخارجي الأساسي | `0.25` |
| `-ro`, `--randouter` | إزاحة الحد الخارجي العشوائي | `0.20` |

### التدريب (`three_stage_transmittance.py`)

| Flag | المعنى | القيمة الافتراضية |
|---|---|---|
| `--preprocess` | تشغيل وضع المعالجة المسبقة | `False` |
| `--input_folder` | المجلد الذي يحتوي ملفات CSV المدمجة | `""` |
| `--output_npz` | مسار ملف NPZ الناتج | `preprocessed_data.npz` |
| `--data_npz` | مجموعة بيانات NPZ للتدريب | `""` |
| `--csv_file` | CSV احتياطي إذا لم يُستخدم NPZ | `""` |
| `--test` | وضع الاختبار | `False` |
| `--num_epochs` | عدد حقب التدريب | `10` |
| `--batch_size` | حجم الدفعة | `4096` |

### إعداد تقييم تاريخي (`three_stage_transmittance_evaluation.py`)

| Flag | المعنى | القيمة الافتراضية |
|---|---|---|
| `--model_dir` | المجلد الذي يحتوي `stageA/B/C` | مطلوب |
| `--data_npz` | إدخال NPZ | `""` |
| `--csv_file` | إدخال CSV احتياطي | `""` |
| `--output_dir` | تجاوز لمجلد الإخراج | تلقائي تحت `model_dir` |
| `--sample_count` | عدد العينات المعروضة بصريًا | `4` |
| `--seed` | البذرة العشوائية | `23` |
| `--font_scale` | مقياس خط الرسم | `1.0` |
| `--batch_size` | حجم دفعة التقييم | `32` |
| `--plot_only` | عرض منحنيات التدريب فقط | `False` |

### مُقيِّم اتساق S4 (`FilterShapeS4_Evaluator_Transmittance.py`)

| Flag | المعنى | القيمة الافتراضية |
|---|---|---|
| `--npz_file` | ملف NPZ المُدخل | `preprocessed_t_data.npz` |
| `--spec2shape_ckpt` | مسار checkpoint للمرحلة C | `outputs_three_stage_20250322_145925/stageC/spec2shape_stageC.pt` |
| `--shape2spec_ckpt` | مسار checkpoint للمرحلة A | `outputs_three_stage_20250322_145925/stageA/shape2spec_stageA.pt` |
| `--n_samples` | عدد العينات المُقيّمة | `4` |
| `--seed` | البذرة العشوائية | `23` |
| `--max_workers` | خيوط عامل S4 | `4` |
| `--out_folder` | مجلد الإخراج | تلقائي مع طابع زمني |

## 🧪 أمثلة

### تشغيل تجريبي سريع

```bash
./ms_final.sh -ns 1000 -r 42 -p smoke -g 40 -bo 0.25 -ro 0.20
python merge_s4_data_full.py --prefix smoke
python -c "import pandas as pd; p='merged_s4_shapes_smoke.csv'; d=pd.read_csv(p).rename(columns={'folder_key':'prefix','NQ':'nQ'}); d.to_csv(p,index=False)"
mkdir -p merged_csvs && mv merged_s4_shapes_smoke.csv merged_csvs/
python three_stage_transmittance.py --preprocess --input_folder merged_csvs --output_npz smoke.npz
python three_stage_transmittance.py --data_npz smoke.npz --num_epochs 5 --batch_size 128
```

### أمثلة ملفات تشغيل (من `commands.md` / `commands_updated.md`)

```bash
# no overlap, G=40
./ms_resume_allargs.sh -ns 10000 -r 12345 -p fast_without_overlap -g 40 -bo 0.25 -ro 0.2

# overlap, G=80
./ms_resume_allargs.sh -ns 10000 -r 12345 -p overlap -g 80 -bo 0.35 -ro 0.3

# large run, overlap, G=80
./ms_resume_allargs.sh -ns 100000 -r 12345 -p more_basis_overlap -g 80 -bo 0.35 -ro 0.3
```

## 🔬 السياق البحثي

إعداد التصميم العكسي الحالي يتعلم استرجاع هندسة متناظرة C4 من النفاذية عبر حالات التبلور. ويفترض مسار النفاذية حاليًا ما يلي:

- 11 صف تبلور لكل عينة شكل (مجمّعة عبر `shape_uid` فريد)
- 100 خانة طول موجي لكل حالة تبلور (أعمدة `T@...`)
- حتى 4 نقاط تحكم Q1 ممثلة بموتر `4x3`: `(presence, x, y)`
- إعادة بناء المضلع تحت تناظر C4 لعرض الشكل وفحوص الاتساق

يحتوي المستودع أيضًا على فروع استكشافية (`AVIRIS*`، `noise_experiment*`، `archived/`) إلى جانب مسار التدريب الرئيسي القائم على النفاذية.

## 🧑‍💻 ملاحظات التطوير

- هذا مستودع بحثي قائم على السكربتات وليس حزمة Python مُعلبة.
- السكربتات الأساسية تفترض مسارات نسبية (خصوصًا `../build/S4`، `results/`، `shapes/`).
- ملف `.gitignore` يستثني العديد من نواتج التجارب المتولدة (`*.csv`، `*.npz`، `*.pt`، ومجلدات التشغيل).
- بعض الملفات/المجلدات المذكورة في الوثائق التاريخية غير موجودة حاليًا في هذه اللقطة؛ وتم الإبقاء على هذه الإشارات عمدًا مع ملاحظات للتوافق.
- ملفات sidecar الخاصة بـ macOS (`._*`) موجودة وقد تكون مجرد بيانات وصفية غير وظيفية.

## 🧯 استكشاف الأخطاء وإصلاحها

| العَرَض | السبب المرجّح | الحل |
|---|---|---|
| `../build/S4: No such file or directory` | ملف S4 التنفيذي مفقود في المسار النسبي المتوقع | أنشئ/اربط S4 في `../build/S4` أو حدّث مسارات المشغلات |
| `No transmission columns found` | ملف CSV لا يحتوي أعمدة `T@...` | أعد التحقق من صيغة خرج الدمج |
| `Must specify either --data_npz or --csv_file` | لم يتم تمرير وسيطة بيانات للتدريب/التقييم | مرّر أحد المُدخلين صراحةً |
| `No valid shapes => SHIFT->Q1->UpTo4` | `vertices_str` غير صالح/فارغ أو ترشيح Q1 يزيل كل العينات | تحقّق من ملفات الأشكال ومخرجات الدمج |
| خرج دمج فارغ مع `--prefix` | البادئة لا تطابق ملفات داخل `results/` | تحقّق من مطابقة البادئة لاسم الملفات وأعد التشغيل |
| ملف checkpoint للتقييم مفقود | ملفات checkpoints للمراحل `stageA/B/C` غير موجودة | تأكّد أن `--model_dir` يشير إلى مجلد مخرجات مكتمل |
| `three_stage_transmittance_evaluation.py` غير موجود | السكربت مشار إليه في وثائق تاريخية لكنه غير موجود الآن | استخدم `FilterShapeS4_Evaluator_Transmittance.py` أو استعد السكربت من commits أقدم |

## 🗺️ خارطة الطريق

- تحسين قابلية إعادة الإنتاج عبر ملف صريح لإصدارات البيانات وتثبيت إعدادات التشغيل.
- توحيد نقاط الدخول المرجعية لمسارات transmittance وAVIRIS وفروع الضوضاء.
- إضافة اختبارات smoke آلية للمعالجة المسبقة وحقبة تدريب مصغرة واحدة.
- إضافة سجل تجارب أوضح يربط مجلدات المخرجات بسطور الأوامر الدقيقة.
- توسيع سير عمل مزامنة README متعدد اللغات (ملفات اللغات في الجذر و`i18n/`).

## 🤝 المساهمة

المساهمات مرحب بها، خاصة فيما يتعلق بقابلية إعادة الإنتاج والاختبار وجودة التوثيق.

العملية المقترحة:

1. افتح issue يوضح النطاق والسلوك المتوقع.
2. أنشئ فرعًا مركزًا على تغيير واحد.
3. أرسل pull request مع أوامر قابلة للتشغيل ومخرجاتها.
4. احرص على إبقاء التغييرات ضمن مسار عمل واحد قدر الإمكان.

## 📄 الترخيص

لا يوجد حاليًا ملف `LICENSE` في جذر المستودع ضمن هذه اللقطة. أضف ملف ترخيص لتحديد شروط الاستخدام وإعادة التوزيع.

## 📚 الاستشهاد

إذا استخدمت هذا المستودع أو بنيت على هذا العمل، يُرجى الاستشهاد بـ:

```bibtex
@article{chen2025inverse,
  title={Inverse Design of Metasurface for Spectral Imaging},
  author={Chen, Rongzhou and Nie, Haitao and Zhu, Shuo and Zhao, Yaping and Wang, Chutian and Lam, Edmund Y},
  journal={arXiv preprint arXiv:2510.21924},
  year={2025}
}
```

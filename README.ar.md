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


# inverse_metasurface ✨

[![Status](https://img.shields.io/badge/Status-Research%20Prototype-orange)](#-project-scope)
[![Python](https://img.shields.io/badge/Python-3.9-3776AB?logo=python&logoColor=white)](#-environment-setup)
[![Platform](https://img.shields.io/badge/Platform-Linux-2f2f2f?logo=linux&logoColor=white)](#-prerequisites)
[![RCWA](https://img.shields.io/badge/RCWA-S4%20Required-1f9d55)](#-prerequisites)
[![PyTorch](https://img.shields.io/badge/Framework-PyTorch-EE4C2C?logo=pytorch&logoColor=white)](#-training-and-evaluation)
[![License](https://img.shields.io/badge/License-Not%20Specified-lightgrey)](#-license)

قاعدة شيفرة بحثية تعتمد على السكربتات أولًا من أجل **التصميم العكسي للميتاسطوح** تحت تماثل C4.
وتجمع بين:

- 🔬 محاكاة S4/RCWA (`.lua` مع مشغلات Bash)
- 🧱 دمج البيانات والمعالجة المسبقة من الأطياف الخام + إحداثيات رؤوس الأشكال
- 🧠 تدريب PyTorch بثلاث مراحل (`shape -> spectra`، ثم `spectra -> shape`، ثم ضبط السلسلة)
- 📊 التقييم والرسم، بما في ذلك فحوصات الاتساق بين النموذج العصبي وS4

<a id="-table-of-contents"></a>
## 📌 جدول المحتويات

- [نطاق المشروع](#-project-scope)
- [السياق البحثي](#-research-context)
- [هيكل المستودع](#-repository-layout)
- [المتطلبات المسبقة](#-prerequisites)
- [إعداد البيئة](#-environment-setup)
- [بدء سريع](#-quick-start)
- [خط الأنابيب الكامل](#-end-to-end-pipeline)
- [التدريب والتقييم](#-training-and-evaluation)
- [أهم خيارات سطر الأوامر](#-key-cli-options)
- [استكشاف الأخطاء وإصلاحها](#-troubleshooting)
- [خارطة الطريق](#-roadmap)
- [الاستشهاد](#-citation)
- [الترخيص](#-license)

<a id="-project-scope"></a>
## 🎯 نطاق المشروع

هذا المستودع غني بالتجارب ومتمحور حول السكربتات التنفيذية (وليس مكتبة مُغلَّفة كحزمة).
وسير العمل الأكثر استقرارًا هو:

1. تشغيل S4 لتوليد المخرجات البصرية الخام داخل `results/` والأشكال داخل `shapes/`
2. دمج ملفات CSV لكل تشغيل وإعادة تشكيلها إلى بيانات جدوليّة جاهزة للتدريب
3. تحويل ملفات CSV المدمجة إلى `.npz` مضغوط
4. تدريب خط أنابيب النفاذية ثلاثي المراحل
5. تقييم نقاط الحفظ وتصدير الرسومات/المقاييس

<a id="-research-context"></a>
## 🧪 السياق البحثي

### إعداد المشكلة

المهمة الأساسية للتصميم العكسي هي استرجاع هندسة الميتاسطح من أطياف نفاذية مستهدفة (والعكس صحيح)، مع قيود تماثل C4 ومسوح تبلور جزئي.

### افتراضات البيانات في خط أنابيب النفاذية

| العنصر | القيمة |
|---|---|
| حالات التبلور لكل شكل | 11 (`c` من `0.0` إلى `1.0`) |
| الحاويات الطيفية لكل حالة | 100 |
| موتر الطيف لكل عينة | `11 x 100` |
| موتر الشكل لكل عينة | `4 x 3` (`[presence, x, y]`) |

### أهداف التعلم عبر المراحل الثلاث

| المرحلة | الاتجاه | نقطة الحفظ النموذجية |
|---|---|---|
| A | `shape -> spectrum` | `stageA/shape2spec_stageA.pt` |
| B | `spectrum -> shape` | `stageB/spec2shape_stageB.pt` |
| C | `spectrum -> shape -> spectrum` (ضبط تسلسلي) | `stageC/spec2shape_stageC.pt` |

<a id="-repository-layout"></a>
## 🗂️ هيكل المستودع

```text
.
├── ms.sh / ms_final.sh / ms_resume_allargs.sh
├── metasurface_seed.lua / metasurface_final.lua / metasurface_allargs_resume.lua
├── merge.py / merge_s4_data_full.py / merge_s4_data_local.py / merge_robust.py
├── three_stage_transmittance.py
├── three_stage_transmittance_evaluation.py
├── FilterShapeS4_Evaluator_Transmittance.py
├── partial_crys_data/
├── results/                          # raw S4 outputs
├── shapes/                           # generated polygon vertices
├── outputs_three_stage_*/            # checkpoints + training artifacts
├── AVIRIS*/ and aviris_*.py          # related hyperspectral experiments
├── commands.md / how_to_run.md
├── iccp.yaml
└── pip_requirements.txt
```

<a id="-prerequisites"></a>
## 🧩 المتطلبات المسبقة

| الاعتمادية | ملاحظات |
|---|---|
| Linux + Bash | سكربتات الصدفة تفترض مسارات بنمط Linux |
| Conda | إدارة البيئة الموصى بها (`iccp.yaml`) |
| Python 3.9 | بيئة التشغيل الأساسية لسكربتات التدريب/التقييم |
| S4 binary | متوقع في `../build/S4` نسبةً إلى جذر المستودع |
| CUDA (اختياري) | يسرّع التدريب والتقييم |

<a id="-environment-setup"></a>
## ⚙️ إعداد البيئة

```bash
git clone <repo-url> inverse_metasurface
cd inverse_metasurface

conda env create -f iccp.yaml
conda activate iccp

# verify S4 path expected by shell runners
ls -l ../build/S4
```

اختياري: تفعيل صلاحية التنفيذ لمشغلات السكربتات:

```bash
chmod +x ms.sh ms_final.sh ms_resume_allargs.sh ms_resume.sh
```

<a id="-quick-start"></a>
## 🚀 بدء سريع

إذا كان لديك `preprocessed_t_data.npz` مسبقًا:

```bash
python three_stage_transmittance.py \
  --data_npz preprocessed_t_data.npz \
  --num_epochs 100 \
  --batch_size 1024

python three_stage_transmittance_evaluation.py \
  --model_dir outputs_three_stage_YYYYMMDD_HHMMSS \
  --data_npz preprocessed_t_data.npz \
  --sample_count 8
```

<a id="-end-to-end-pipeline"></a>
## 🔁 خط الأنابيب الكامل

### 1) توليد بيانات S4

```bash
./ms_final.sh \
  -ns 10000 \
  -r 12345 \
  -p myrun \
  -g 80 \
  -bo 0.35 \
  -ro 0.30
```

### 2) دمج مخرجات S4 مع رؤوس الأشكال

```bash
python merge_s4_data_full.py --prefix myrun
# -> merged_s4_shapes_myrun.csv
```

### 3) توحيد أسماء أعمدة CSV قبل المعالجة المسبقة

المعالجة المسبقة في `three_stage_transmittance.py` تتوقع `prefix` و `nQ`، بينما قد يحتوي الإخراج المدمج على `folder_key` و `NQ`.

```bash
python -c "import pandas as pd; p='merged_s4_shapes_myrun.csv'; df=pd.read_csv(p); df=df.rename(columns={'folder_key':'prefix','NQ':'nQ'}); df.to_csv(p,index=False)"
```

### 4) معالجة CSV إلى NPZ

```bash
mkdir -p merged_csvs
mv merged_s4_shapes_myrun.csv merged_csvs/

python three_stage_transmittance.py --preprocess \
  --input_folder merged_csvs \
  --output_npz preprocessed_t_data.npz
```

### 5) تدريب المراحل الثلاث

```bash
python three_stage_transmittance.py \
  --data_npz preprocessed_t_data.npz \
  --num_epochs 100 \
  --batch_size 1024
```

### 6) التقييم والرسم

```bash
python three_stage_transmittance_evaluation.py \
  --model_dir outputs_three_stage_YYYYMMDD_HHMMSS \
  --data_npz preprocessed_t_data.npz \
  --sample_count 8
```

### 7) اختياري: مقارنة النموذج العصبي مع S4

```bash
python FilterShapeS4_Evaluator_Transmittance.py \
  --npz_file preprocessed_t_data.npz \
  --spec2shape_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageC/spec2shape_stageC.pt \
  --shape2spec_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageA/shape2spec_stageA.pt \
  --n_samples 4
```

<a id="-training-and-evaluation"></a>
## 🧠 التدريب والتقييم

### السكربتات الأساسية

| السكربت | الغرض |
|---|---|
| `three_stage_transmittance.py` | معالجة مسبقة + تدريب المراحل A/B/C |
| `three_stage_transmittance_evaluation.py` | تقييم نقاط الحفظ، حساب المقاييس، حفظ الرسومات |
| `FilterShapeS4_Evaluator_Transmittance.py` | مقارنة تنبؤات النموذج المتعلَّم مع سلوك S4 |

### المخرجات المعتادة

| المخرج | الموقع |
|---|---|
| نقاط حفظ المراحل | `outputs_three_stage_*/stageA|stageB|stageC/` |
| رسومات التقييم | `outputs_three_stage_*/evaluation_<timestamp>/` |
| ملفات مقاييس CSV | `evaluation_metrics.csv`, `metrics_summary.csv` |

<a id="-key-cli-options"></a>
## 🛠️ أهم خيارات سطر الأوامر

### مشغلات S4 (`ms_final.sh`, `ms_resume_allargs.sh`)

| الخيار | المعنى | الافتراضي |
|---|---|---|
| `-ns`, `--numshapes` | عدد الأشكال | `100000` |
| `-r`, `--seed` | البذرة العشوائية | `88888` |
| `-p`, `--prefix` | بادئة التشغيل/مفتاح الاستكمال | فارغ |
| `-g`, `--numg` | إعداد قاعدة الهندسة | `80` |
| `-bo`, `--baseouter` | إزاحة الحافة الخارجية الأساسية | `0.25` |
| `-ro`, `--randouter` | إزاحة الحافة الخارجية العشوائية | `0.20` |

### التدريب (`three_stage_transmittance.py`)

| الخيار | المعنى | الافتراضي |
|---|---|---|
| `--preprocess` | التبديل إلى وضع المعالجة المسبقة | off |
| `--input_folder` | مجلد ملفات CSV المدمجة | `""` |
| `--output_npz` | ملف خرج المعالجة المسبقة | `preprocessed_data.npz` |
| `--data_npz` | دخل NPZ للتدريب | `""` |
| `--csv_file` | دخل CSV مباشر للتدريب | `""` |
| `--test` | تفعيل وضع الاختبار | off |
| `--num_epochs` | عدد العصور لكل مرحلة | `10` |
| `--batch_size` | حجم الدفعة | `4096` |

### التقييم (`three_stage_transmittance_evaluation.py`)

| الخيار | المعنى | الافتراضي |
|---|---|---|
| `--model_dir` | مجلد التشغيل المدرّب (إلزامي) | - |
| `--data_npz` | دخل NPZ للتقييم | `""` |
| `--csv_file` | دخل CSV للتقييم | `""` |
| `--output_dir` | مجلد خرج مخصص | تلقائي |
| `--sample_count` | عدد العينات المعروضة بصريًا | `4` |
| `--seed` | بذرة عشوائية لاختيار العينات | `23` |
| `--font_scale` | معامل تكبير الخط في الرسومات | `1.0` |
| `--batch_size` | حجم دفعة DataLoader في التقييم | `32` |
| `--plot_only` | إعادة توليد الرسومات فقط | off |

<a id="-troubleshooting"></a>
## 🧯 استكشاف الأخطاء وإصلاحها

| العَرَض | السبب المُرجّح | الحل |
|---|---|---|
| `../build/S4: No such file or directory` | ملف S4 التنفيذي ليس في المسار النسبي المتوقع | ضع/ابنِ S4 في `../build/S4` أو عدّل السكربتات |
| `Must specify either --data_npz or --csv_file` | مفقود وسيط بيانات للتدريب/التقييم | مرّر مصدر بيانات واحدًا فقط |
| `No transmission columns found` | ملف CSV المدمج يفتقد أعمدة `T@...` | أعد الدمج/إعادة التشكيل وتحقق من العناوين |
| `KeyError: 'prefix'` in preprocess | خرج الدمج لا يزال يستخدم `folder_key`/`NQ` | أعد تسمية الأعمدة إلى `prefix`/`nQ` قبل المعالجة المسبقة |
| GPU OOM | حجم الدفعة كبير جدًا | خفّض `--batch_size` |
| Missing checkpoints during eval | نقاط حفظ المراحل غير موجودة/المسار خاطئ | تحقق من وجود ملفات stageA/B/C ضمن `--model_dir` المحدد |

<a id="-roadmap"></a>
## 🧭 خارطة الطريق

- توحيد مخطط CSV المدمج عبر سكربتات الدمج (`prefix` و `nQ`)
- إضافة اختبارات آلية للدمج/المعالجة المسبقة/تحميل نقاط الحفظ
- توفير نقطة دخول CLI واحدة لتنسيق خط الأنابيب بالكامل
- إضافة بيانات تعريف dataset/run لتحسين قابلية إعادة الإنتاج
- إضافة ترخيص مفتوح المصدر بشكل صريح

<a id="-citation"></a>
## 📚 الاستشهاد

إذا كان هذا المستودع مفيدًا في بحثك، يُرجى الاستشهاد به:

```bibtex
@article{chen2025inverse,
  title={Inverse Design of Metasurface for Spectral Imaging},
  author={Chen, Rongzhou and Nie, Haitao and Zhu, Shuo and Zhao, Yaping and Wang, Chutian and Lam, Edmund Y},
  journal={arXiv preprint arXiv:2510.21924},
  year={2025}
}
```

<a id="-license"></a>
## 📄 الترخيص

لا يوجد ملف `LICENSE` حاليًا في هذا المستودع. لذلك تظل حقوق الاستخدام وإعادة التوزيع غير محددة إلى أن تتم إضافة ترخيص.

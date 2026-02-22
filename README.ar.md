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

مستودع بحثي يعتمد على السكربتات أولاً (عُرف تاريخياً باسم `inverse_metasurface`) للتصميم العكسي للميتاسطح في التصوير الطيفي. سير العمل الأساسي يدمج بين:

- محاكاة RCWA مرتكزة على الفيزياء (S4 + Lua)
- تجميع البيانات وربط معلومات الأشكال
- تعلّم PyTorch على ثلاث مراحل (`shape -> spectrum`، ثم `spectrum -> shape`، ثم الضبط الدقيق المتسلسل)
- تقييم كمي ونوعي، مع خيار فحص الاتساق بين النموذج العصبي و S4

## ✨ نظرة سريعة

| العنصر | التفاصيل |
|---|---|
| 🎯 المهمة الرئيسية | استنتاج هندسة ميتاسطح متناظرة C4 من أطياف النفاذية المستهدفة |
| 🔬 المحاكي | `../build/S4` ويُستدعى بواسطة launchers في الصدفة وملفات `.lua` |
| 🧠 خط أنابيب التعلّم | المرحلة A `shape -> spectra`، المرحلة B `spectra -> shape`، المرحلة C `spectra -> shape -> spectra` |
| 📦 عقد البيانات | CSV مدمج (`T@...`، بيانات وصفية، `vertices_str`) -> NPZ مضغوط (`uids`، `spectra`، `shapes`) |
| 🧪 التقييم | مقاييس MSE، تصورات لكل مرحلة، مع خيار إعادة محاكاة S4 جديدة |

## 🧭 سير العمل من البداية إلى النهاية

1. توليد مخرجات المحاكاة في `results/` وملفات المضلعات في `shapes/`.
2. دمج ملفات CSV الناتجة من S4 وربطها برؤوس الأشكال.
3. توحيد أسماء الأعمدة لتتوافق مع التدريب.
4. معالجة ملفات CSV المدمجة مسبقاً وتحويلها إلى موترات NPZ.
5. تدريب نماذج المراحل A/B/C.
6. تقييم checkpoints وعرض السلوك بصرياً.
7. اختيارياً مقارنة أطياف الأشكال المتوقعة مع تشغيلات S4 جديدة.

## 🧱 هيكل المستودع

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

## 🛠️ المتطلبات المسبقة

| الاعتمادية | ملاحظات |
|---|---|
| Linux + Bash | سكربتات التشغيل موجّهة للتنفيذ عبر الصدفة |
| Python 3.9 | متوافق مع `iccp.yaml` |
| Conda | موصى به لضمان قابلية إعادة الإنتاج |
| S4 binary | متوقع في المسار `../build/S4` |
| CUDA GPU (اختياري) | يسرّع التدريب والتقييم |

## 🚀 الإعداد

### 1) الاستنساخ والدخول

```bash
git clone <your-repo-url> inverse_metasurface
cd inverse_metasurface
```

### 2) إنشاء البيئة (موصى به)

```bash
conda env create -f iccp.yaml
conda activate iccp
```

بديل (أقل انضباطاً):

```bash
pip install -r pip_requirements.txt
```

### 3) التحقق من مسار المحاكي المتوقع من السكربتات

```bash
ls -l ../build/S4
```

### 4) (اختياري) جعل launchers قابلة للتنفيذ

```bash
chmod +x ms.sh ms_final.sh ms_resume.sh ms_resume_allargs.sh ms_resume_random_state.sh
```

## ▶️ الاستخدام العملي

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

مشغّل مخصص للاستكمال:

```bash
./ms_resume_allargs.sh \
  -ns 10000 \
  -r 12345 \
  -p myrun \
  -g 80 \
  -bo 0.35 \
  -ro 0.30
```

ملاحظات:
- المشغّلات تنفذ `NQ=1..4` بالتوازي.
- السكربتات تستدعي `../build/S4` مع `-t 32`.

### B) دمج مخرجات S4 وربط رؤوس الأشكال

```bash
python merge_s4_data_full.py --prefix myrun
# output: merged_s4_shapes_myrun.csv
```

### C) توحيد الأعمدة للتوافق مع مسار التدريب

يكتب `merge_s4_data_full.py` العمودين `folder_key` و `NQ`، بينما يتوقع مسار التدريب `prefix` و `nQ`.

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

### E) تدريب المراحل A/B/C

```bash
python three_stage_transmittance.py \
  --data_npz preprocessed_t_data.npz \
  --num_epochs 100 \
  --batch_size 1024
```

تُكتب المخرجات في:
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageA`
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageB`
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageC`

### F) تقييم النماذج المدرّبة

```bash
python three_stage_transmittance_evaluation.py \
  --model_dir outputs_three_stage_YYYYMMDD_HHMMSS \
  --data_npz preprocessed_t_data.npz \
  --sample_count 8
```

### G) (اختياري) فحص الاتساق بين النموذج العصبي و S4

```bash
python FilterShapeS4_Evaluator_Transmittance.py \
  --npz_file preprocessed_t_data.npz \
  --spec2shape_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageC/spec2shape_stageC.pt \
  --shape2spec_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageA/shape2spec_stageA.pt \
  --n_samples 4
```

## ⚙️ خيارات CLI الأساسية

### مشغلات S4 (`ms_final.sh`, `ms_resume_allargs.sh`)

| Flag | المعنى | الافتراضي |
|---|---|---|
| `-ns`, `--numshapes` | عدد الأشكال المراد توليدها | `100000` |
| `-r`, `--seed` | البذرة العشوائية | `88888` |
| `-p`, `--prefix` | مفتاح البادئة/الاستكمال | `""` |
| `-g`, `--numg` | معامل الأساس/الشبكة | `80` |
| `-bo`, `--baseouter` | إزاحة الحد الخارجي الأساسي | `0.25` |
| `-ro`, `--randouter` | إزاحة الحد الخارجي العشوائي | `0.20` |

### التدريب (`three_stage_transmittance.py`)

| Flag | المعنى | الافتراضي |
|---|---|---|
| `--preprocess` | تشغيل وضع المعالجة المسبقة | `False` |
| `--input_folder` | المجلد الذي يحتوي ملفات CSV المدمجة | `""` |
| `--output_npz` | مسار ملف NPZ الناتج | `preprocessed_data.npz` |
| `--data_npz` | مجموعة بيانات NPZ للتدريب | `""` |
| `--csv_file` | بديل CSV عند عدم استخدام NPZ | `""` |
| `--test` | وضع الاختبار | `False` |
| `--num_epochs` | عدد عصور التدريب | `10` |
| `--batch_size` | حجم الدفعة | `4096` |

### التقييم (`three_stage_transmittance_evaluation.py`)

| Flag | المعنى | الافتراضي |
|---|---|---|
| `--model_dir` | مجلد يحتوي `stageA/B/C` | مطلوب |
| `--data_npz` | إدخال NPZ | `""` |
| `--csv_file` | بديل إدخال CSV | `""` |
| `--output_dir` | تجاوز مجلد الإخراج | تلقائي داخل `model_dir` |
| `--sample_count` | عدد العينات المعروضة بصرياً | `4` |
| `--seed` | البذرة العشوائية | `23` |
| `--font_scale` | تحجيم خط الرسوم | `1.0` |
| `--batch_size` | حجم دفعة التقييم | `32` |
| `--plot_only` | عرض منحنيات التدريب فقط | `False` |

### مقيم الاتساق مع S4 (`FilterShapeS4_Evaluator_Transmittance.py`)

| Flag | المعنى | الافتراضي |
|---|---|---|
| `--npz_file` | ملف NPZ للإدخال | `preprocessed_t_data.npz` |
| `--spec2shape_ckpt` | مسار checkpoint للمرحلة C | `outputs_three_stage_20250322_145925/stageC/spec2shape_stageC.pt` |
| `--shape2spec_ckpt` | مسار checkpoint للمرحلة A | `outputs_three_stage_20250322_145925/stageA/shape2spec_stageA.pt` |
| `--n_samples` | عدد العينات المقيّمة | `4` |
| `--seed` | البذرة العشوائية | `23` |
| `--max_workers` | خيوط عامل S4 | `4` |
| `--out_folder` | مجلد الإخراج | طابع زمني تلقائي |

## 🧪 تشغيل اختبار سريع

```bash
./ms_final.sh -ns 1000 -r 42 -p smoke -g 40 -bo 0.25 -ro 0.20
python merge_s4_data_full.py --prefix smoke
python -c "import pandas as pd; p='merged_s4_shapes_smoke.csv'; d=pd.read_csv(p).rename(columns={'folder_key':'prefix','NQ':'nQ'}); d.to_csv(p,index=False)"
mkdir -p merged_csvs && mv merged_s4_shapes_smoke.csv merged_csvs/
python three_stage_transmittance.py --preprocess --input_folder merged_csvs --output_npz smoke.npz
python three_stage_transmittance.py --data_npz smoke.npz --num_epochs 5 --batch_size 128
```

## 🔬 السياق البحثي

إعداد التصميم العكسي الحالي يتعلّم استرجاع هندسة متناظرة C4 من النفاذية عبر حالات التبلور. خط أنابيب النفاذية يفترض حالياً:

- 11 صف تبلور لكل عينة شكل (مجمعة حسب `shape_uid` فريد)
- 100 خانة طول موجي لكل حالة تبلور (أعمدة `T@...`)
- حتى 4 نقاط تحكم Q1 مُرمّزة على شكل موتر `4x3`: `(presence, x, y)`
- إعادة بناء مضلع بتناظر C4 لأغراض تصور الشكل وفحوصات الاتساق

يحتوي المستودع أيضاً على مسارات استكشافية (`AVIRIS*`، `noise_experiment*`، `archived/`) خارج مسار تدريب النفاذية الأساسي.

## 🧯 استكشاف الأعطال وإصلاحها

| العَرَض | السبب المرجح | الحل |
|---|---|---|
| `../build/S4: No such file or directory` | ملف S4 التنفيذي غير موجود في المسار النسبي المتوقع | ابنِ أو اربط S4 في `../build/S4`، أو حدّث مسارات المشغلات |
| `No transmission columns found` | ملف CSV يفتقد أعمدة `T@...` | أعد التحقق من تنسيق خرج الدمج |
| `Must specify either --data_npz or --csv_file` | معامل بيانات التدريب/التقييم غير مُحدد | مرّر أحد المدخلين بشكل صريح |
| `No valid shapes => SHIFT->Q1->UpTo4` | قيمة `vertices_str` غير صالحة/فارغة أو ترشيح Q1 أزال كل العينات | تحقّق من ملفات الأشكال وخرج الدمج |
| Empty merge output for `--prefix` | قيمة البادئة لا تطابق ملفات داخل `results/` | تحقّق من البادئة الدقيقة في أسماء الملفات وأعد الدمج |
| Evaluation checkpoint missing | ملفات checkpoints للمراحل `stageA/B/C` مفقودة | تحقّق أن `--model_dir` يشير إلى مجلد مخرجات مكتمل |

## 🤝 المساهمة

نرحب بالمساهمات، خصوصاً في قابلية إعادة الإنتاج والاختبارات وجودة التوثيق.

العملية المقترحة:

1. افتح issue يوضح النطاق والسلوك المتوقع.
2. أنشئ فرعاً مركزاً على تغيير محدد.
3. أرسل pull request يتضمن أوامر قابلة للتشغيل ومخرجاتها.
4. اجعل التغييرات ضمن سير عمل واحد قدر الإمكان.

## 📄 الترخيص

لا يوجد حالياً ملف `LICENSE` في جذر المستودع ضمن هذه اللقطة. أضف ملفاً لتحديد شروط الاستخدام وإعادة التوزيع.

## 📚 الاستشهاد

إذا استخدمت هذا المستودع أو بنيت على هذا العمل، يرجى الاستشهاد بما يلي:

```bibtex
@article{chen2025inverse,
  title={Inverse Design of Metasurface for Spectral Imaging},
  author={Chen, Rongzhou and Nie, Haitao and Zhu, Shuo and Zhao, Yaping and Wang, Chutian and Lam, Edmund Y},
  journal={arXiv preprint arXiv:2510.21924},
  year={2025}
}
```

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

مستودع بحثي يعتمد على السكربتات (ويُشار إليه تاريخيًا باسم `inverse_metasurface`) للتصميم العكسي للميتاسطوح في التصوير الطيفي. يدمج هذا المسار بين **محاكاة RCWA المبنية على S4** و**سير عمل PyTorch من ثلاث مراحل** للنمذجة الأمامية والعكسية بين الهندسة وطيف النفاذية البصرية.

## ✨ نظرة سريعة

| العنصر | التفاصيل |
|---|---|
| 🎯 الهدف | التنبؤ بهندسة ميتاسطح متناظرة C4 انطلاقًا من أطياف النفاذية المستهدفة |
| 🔬 الفيزياء | محاكاة RCWA باستخدام S4 (`../build/S4`) |
| 🧠 مسار التعلم | المرحلة A: `shape -> spectra`، المرحلة B: `spectra -> shape`، المرحلة C: `spectra -> shape -> spectra` |
| 📦 شكل البيانات | CSV مدمج (`T@...` مع بيانات الشكل) -> NPZ مضغوط (`uids`, `spectra`, `shapes`) |
| 🧪 التقييم | مقاييس MSE، ورسوم نوعية، وفحوصات إعادة محاكاة S4 اختيارية |

## 🧭 سير العمل من البداية إلى النهاية

1. توليد الاستجابات البصرية للميتاسطح باستخدام S4 (ملفات `.lua` مع مشغلات shell).
2. دمج ملفات CSV الخام الناتجة من المحاكاة وإلحاق رؤوس المضلعات.
3. تحويل ملفات CSV المدمجة إلى NPZ للتدريب.
4. تدريب خط أنابيب النفاذية ثلاثي المراحل.
5. تقييم نقاط الحفظ وعرض سلوك المراحل A/B/C.
6. اختياريًا، مقارنة تنبؤات الشبكة العصبية مع محاكاة S4 جديدة.

## 🧱 هيكل المستودع

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

## 🛠️ المتطلبات المسبقة

- Linux + Bash
- Conda (موصى به)
- Python 3.9
- توفر ملف S4 التنفيذي في `../build/S4`
- اختياري: GPU يدعم CUDA لتسريع التدريب

## 🚀 الإعداد

```bash
git clone <your-repo-url> inverse_metasurface
cd inverse_metasurface

conda env create -f iccp.yaml
conda activate iccp

# Verify simulator path expected by scripts
ls -l ../build/S4
```

اختياري:

```bash
chmod +x ms.sh ms_final.sh ms_resume_allargs.sh
```

## ▶️ الاستخدام العملي

### 1) توليد بيانات RCWA

الملفان `ms_final.sh` و`ms_resume_allargs.sh` يُشغّلان 4 وظائف S4 متوازية (`NQ=1..4`):

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
- `ms.sh` هو مسار تشغيل أبسط.
- تفترض ملفات التشغيل وجود `../build/S4` وتستخدم `-t 32`.

### 2) دمج مخرجات S4 مع رؤوس الأشكال

```bash
python merge_s4_data_full.py --prefix myrun
# output: merged_s4_shapes_myrun.csv
```

### 3) توحيد أسماء الأعمدة للتوافق مع التدريب

ينتج `merge_s4_data_full.py` العمودين `folder_key` و`NQ`، بينما يتوقع `three_stage_transmittance.py` العمودين `prefix` و`nQ`.

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

### 5) تدريب خط الأنابيب ثلاثي المراحل

```bash
python three_stage_transmittance.py \
  --data_npz preprocessed_t_data.npz \
  --num_epochs 100 \
  --batch_size 1024
```

نمط المخرجات المتوقع:
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageA`
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageB`
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageC`

### 6) تقييم نماذج المراحل A/B/C

```bash
python three_stage_transmittance_evaluation.py \
  --model_dir outputs_three_stage_YYYYMMDD_HHMMSS \
  --data_npz preprocessed_t_data.npz \
  --sample_count 8
```

### 7) فحص اتساق الشبكة العصبية مقابل S4 (اختياري)

```bash
python FilterShapeS4_Evaluator_Transmittance.py \
  --npz_file preprocessed_t_data.npz \
  --spec2shape_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageC/spec2shape_stageC.pt \
  --shape2spec_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageA/shape2spec_stageA.pt \
  --n_samples 4
```

## ⚙️ خيارات CLI الأساسية

### مشغلات S4 (`ms_final.sh`, `ms_resume_allargs.sh`)

| الوسيط | المعنى | الافتراضي |
|---|---|---|
| `-ns`, `--numshapes` | عدد الأشكال المراد توليدها | `100000` |
| `-r`, `--seed` | البذرة العشوائية | `88888` |
| `-p`, `--prefix` | بادئة/مفتاح الاستئناف | `""` |
| `-g`, `--numg` | معامل الأساس/الهندسة | `80` |
| `-bo`, `--baseouter` | إزاحة الحد الخارجي الأساسي | `0.25` |
| `-ro`, `--randouter` | إزاحة خارجية عشوائية | `0.20` |

### التدريب (`three_stage_transmittance.py`)

| الوسيط | الغرض | الافتراضي |
|---|---|---|
| `--preprocess` | تشغيل وضع المعالجة المسبقة | `False` |
| `--input_folder` | المجلد الذي يحتوي على ملفات CSV المدمجة | `""` |
| `--output_npz` | مسار ملف NPZ الناتج | `preprocessed_data.npz` |
| `--data_npz` | ملف NPZ المستخدم في التدريب | `""` |
| `--csv_file` | CSV بديل عند عدم استخدام NPZ | `""` |
| `--test` | وضع الاختبار (تخطي التدريب) | `False` |
| `--num_epochs` | عدد الحِقب | `10` |
| `--batch_size` | حجم الدفعة | `4096` |

### التقييم (`three_stage_transmittance_evaluation.py`)

| الوسيط | الغرض | الافتراضي |
|---|---|---|
| `--model_dir` | المجلد الجذر الذي يحتوي `stageA/B/C` | مطلوب |
| `--data_npz` | ملف NPZ كمدخل للتقييم | `""` |
| `--csv_file` | ملف CSV كمدخل للتقييم | `""` |
| `--output_dir` | تجاوز مجلد الإخراج | تلقائي داخل `model_dir` |
| `--sample_count` | عدد العينات المعروضة بصريًا | `4` |
| `--seed` | البذرة العشوائية لأخذ العينات | `23` |
| `--font_scale` | مقياس خط الرسم | `1.0` |
| `--batch_size` | حجم دفعة التقييم | `32` |
| `--plot_only` | عرض المنحنيات فقط | `False` |

### مُقيّم الاتساق مع S4 (`FilterShapeS4_Evaluator_Transmittance.py`)

| الوسيط | الغرض | الافتراضي |
|---|---|---|
| `--npz_file` | ملف NPZ بعد المعالجة المسبقة | `preprocessed_t_data.npz` |
| `--spec2shape_ckpt` | نقطة حفظ المرحلة C | `outputs_three_stage_20250322_145925/stageC/spec2shape_stageC.pt` |
| `--shape2spec_ckpt` | نقطة حفظ المرحلة A | `outputs_three_stage_20250322_145925/stageA/shape2spec_stageA.pt` |
| `--n_samples` | عدد العينات المراد فحصها | `4` |
| `--seed` | البذرة العشوائية | `23` |
| `--max_workers` | عدد عمليات S4 المتوازية | `4` |
| `--out_folder` | مجلد الإخراج | طابع زمني تلقائي |

## 🧪 تشغيل سريع (Smoke Run)

```bash
./ms_final.sh -ns 1000 -r 42 -p smoke -g 40 -bo 0.25 -ro 0.20
python merge_s4_data_full.py --prefix smoke
python -c "import pandas as pd; p='merged_s4_shapes_smoke.csv'; d=pd.read_csv(p).rename(columns={'folder_key':'prefix','NQ':'nQ'}); d.to_csv(p,index=False)"
python three_stage_transmittance.py --preprocess --input_folder . --output_npz smoke.npz
python three_stage_transmittance.py --data_npz smoke.npz --num_epochs 5 --batch_size 128
```

## 🔬 السياق البحثي

تتمحور صياغة التصميم العكسي الأساسية حول استنتاج هندسة ميتاسطح متناظرة C4 من أطياف النفاذية عبر حالات التبلور. يفترض مسار النفاذية الحالي ما يلي:

- 11 حالة تبلور لكل عينة (قيم `c` يتم ترتيبها أثناء المعالجة المسبقة)
- 100 خانة طول موجي لكل حالة (أعمدة `T@...`)
- حتى 4 رؤوس Q1 ممثلة بالشكل `(presence, x, y)`

يتضمن هذا المستودع أيضًا فروعًا استكشافية (مثل `AVIRIS*` و`noise_experiment_*` و`archived/`) إلى جانب مسار النفاذية القياسي.

## 📚 الاقتباس

إذا استخدمت هذا المستودع أو بنيت على هذا العمل، يُرجى الاستشهاد بما يلي:

```bibtex
@article{chen2025inverse,
  title={Inverse Design of Metasurface for Spectral Imaging},
  author={Chen, Rongzhou and Nie, Haitao and Zhu, Shuo and Zhao, Yaping and Wang, Chutian and Lam, Edmund Y},
  journal={arXiv preprint arXiv:2510.21924},
  year={2025}
}
```

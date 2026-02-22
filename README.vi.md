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

Một codebase nghiên cứu ưu tiên chạy script cho bài toán **thiết kế ngược metasurface** dưới ràng buộc đối xứng C4.
Repository kết hợp:

- 🔬 Mô phỏng S4/RCWA (`.lua` + trình chạy Bash)
- 🧱 Gộp dữ liệu và tiền xử lý từ phổ thô + đỉnh hình học
- 🧠 Huấn luyện PyTorch ba giai đoạn (`shape -> spectra`, `spectra -> shape`, tinh chỉnh chuỗi)
- 📊 Đánh giá và vẽ biểu đồ, gồm cả kiểm tra độ nhất quán giữa mạng nơ-ron và S4

## 📌 Mục lục

- [Project Scope](#-project-scope)
- [Research Context](#-research-context)
- [Repository Layout](#-repository-layout)
- [Prerequisites](#-prerequisites)
- [Environment Setup](#-environment-setup)
- [Quick Start](#-quick-start)
- [End-to-End Pipeline](#-end-to-end-pipeline)
- [Training and Evaluation](#-training-and-evaluation)
- [Key CLI Options](#-key-cli-options)
- [Troubleshooting](#-troubleshooting)
- [Roadmap](#-roadmap)
- [Citation](#-citation)
- [License](#-license)

## 🎯 Project Scope

Repository này thiên về thử nghiệm và xoay quanh các script thực thi (không phải thư viện đóng gói).
Quy trình ổn định nhất:

1. Chạy S4 để tạo đầu ra quang học thô trong `results/` và hình học trong `shapes/`
2. Gộp và pivot các file CSV theo từng lượt chạy thành dữ liệu bảng sẵn sàng cho huấn luyện
3. Chuyển các file CSV đã gộp sang định dạng nén `.npz`
4. Huấn luyện pipeline truyền qua ba giai đoạn
5. Đánh giá checkpoint và xuất biểu đồ/chỉ số

## 🧪 Research Context

### Problem setting

Bài toán thiết kế ngược cốt lõi là khôi phục hình học metasurface từ phổ truyền qua mục tiêu (và ngược lại), với ràng buộc đối xứng C4 và quét kết tinh một phần.

### Data assumptions in the transmittance pipeline

| Item | Value |
|---|---|
| Crystallization states per shape | 11 (`c` from `0.0` to `1.0`) |
| Spectral bins per state | 100 |
| Spectrum tensor per sample | `11 x 100` |
| Shape tensor per sample | `4 x 3` (`[presence, x, y]`) |

### Three-stage learning targets

| Stage | Direction | Typical checkpoint |
|---|---|---|
| A | `shape -> spectrum` | `stageA/shape2spec_stageA.pt` |
| B | `spectrum -> shape` | `stageB/spec2shape_stageB.pt` |
| C | `spectrum -> shape -> spectrum` (chain-tuned) | `stageC/spec2shape_stageC.pt` |

## 🗂️ Repository Layout

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

## 🧩 Prerequisites

| Dependency | Notes |
|---|---|
| Linux + Bash | Script shell giả định đường dẫn kiểu Linux |
| Conda | Khuyến nghị để quản lý môi trường (`iccp.yaml`) |
| Python 3.9 | Runtime chính cho script huấn luyện/đánh giá |
| S4 binary | Mặc định ở `../build/S4` so với thư mục gốc repo |
| CUDA (optional) | Giúp tăng tốc huấn luyện và đánh giá |

## ⚙️ Environment Setup

```bash
git clone <repo-url> inverse_metasurface
cd inverse_metasurface

conda env create -f iccp.yaml
conda activate iccp

# verify S4 path expected by shell runners
ls -l ../build/S4
```

Có thể cấp quyền thực thi cho các script shell:

```bash
chmod +x ms.sh ms_final.sh ms_resume_allargs.sh ms_resume.sh
```

## 🚀 Quick Start

Nếu bạn đã có `preprocessed_t_data.npz`:

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

## 🔁 End-to-End Pipeline

### 1) Generate S4 data

```bash
./ms_final.sh \
  -ns 10000 \
  -r 12345 \
  -p myrun \
  -g 80 \
  -bo 0.35 \
  -ro 0.30
```

### 2) Merge S4 outputs + shape vertices

```bash
python merge_s4_data_full.py --prefix myrun
# -> merged_s4_shapes_myrun.csv
```

### 3) Normalize CSV column names for preprocessing

Phần tiền xử lý trong `three_stage_transmittance.py` yêu cầu `prefix` và `nQ`, trong khi dữ liệu đã gộp có thể dùng `folder_key` và `NQ`.

```bash
python -c "import pandas as pd; p='merged_s4_shapes_myrun.csv'; df=pd.read_csv(p); df=df.rename(columns={'folder_key':'prefix','NQ':'nQ'}); df.to_csv(p,index=False)"
```

### 4) CSV -> NPZ preprocessing

```bash
mkdir -p merged_csvs
mv merged_s4_shapes_myrun.csv merged_csvs/

python three_stage_transmittance.py --preprocess \
  --input_folder merged_csvs \
  --output_npz preprocessed_t_data.npz
```

### 5) Train three stages

```bash
python three_stage_transmittance.py \
  --data_npz preprocessed_t_data.npz \
  --num_epochs 100 \
  --batch_size 1024
```

### 6) Evaluate and plot

```bash
python three_stage_transmittance_evaluation.py \
  --model_dir outputs_three_stage_YYYYMMDD_HHMMSS \
  --data_npz preprocessed_t_data.npz \
  --sample_count 8
```

### 7) Optional: neural vs S4 comparison

```bash
python FilterShapeS4_Evaluator_Transmittance.py \
  --npz_file preprocessed_t_data.npz \
  --spec2shape_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageC/spec2shape_stageC.pt \
  --shape2spec_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageA/shape2spec_stageA.pt \
  --n_samples 4
```

## 🧠 Training and Evaluation

### Main scripts

| Script | Purpose |
|---|---|
| `three_stage_transmittance.py` | tiền xử lý + huấn luyện các giai đoạn A/B/C |
| `three_stage_transmittance_evaluation.py` | đánh giá checkpoint, tính metric, lưu biểu đồ |
| `FilterShapeS4_Evaluator_Transmittance.py` | so sánh dự đoán của mô hình với hành vi S4 |

### Typical outputs

| Artifact | Location |
|---|---|
| Stage checkpoints | `outputs_three_stage_*/stageA|stageB|stageC/` |
| Evaluation figures | `outputs_three_stage_*/evaluation_<timestamp>/` |
| Metrics CSV | `evaluation_metrics.csv`, `metrics_summary.csv` |

## 🛠️ Key CLI Options

### S4 launchers (`ms_final.sh`, `ms_resume_allargs.sh`)

| Flag | Meaning | Default |
|---|---|---|
| `-ns`, `--numshapes` | số lượng hình học | `100000` |
| `-r`, `--seed` | seed ngẫu nhiên | `88888` |
| `-p`, `--prefix` | tiền tố lượt chạy / khóa resume | empty |
| `-g`, `--numg` | tham số cơ sở hình học | `80` |
| `-bo`, `--baseouter` | độ lệch outer cơ sở | `0.25` |
| `-ro`, `--randouter` | độ lệch outer ngẫu nhiên | `0.20` |

### Training (`three_stage_transmittance.py`)

| Flag | Meaning | Default |
|---|---|---|
| `--preprocess` | chuyển sang chế độ tiền xử lý | off |
| `--input_folder` | thư mục chứa CSV đã gộp | `""` |
| `--output_npz` | file đầu ra tiền xử lý | `preprocessed_data.npz` |
| `--data_npz` | NPZ đầu vào cho huấn luyện | `""` |
| `--csv_file` | đầu vào CSV trực tiếp cho huấn luyện | `""` |
| `--test` | bật/tắt chế độ test | off |
| `--num_epochs` | số epoch cho mỗi giai đoạn | `10` |
| `--batch_size` | kích thước batch | `4096` |

### Evaluation (`three_stage_transmittance_evaluation.py`)

| Flag | Meaning | Default |
|---|---|---|
| `--model_dir` | thư mục run đã huấn luyện (bắt buộc) | - |
| `--data_npz` | NPZ đầu vào đánh giá | `""` |
| `--csv_file` | CSV đầu vào đánh giá | `""` |
| `--output_dir` | thư mục đầu ra tùy chỉnh | auto |
| `--sample_count` | số mẫu trực quan hóa | `4` |
| `--seed` | seed ngẫu nhiên khi chọn mẫu | `23` |
| `--font_scale` | hệ số cỡ chữ cho biểu đồ | `1.0` |
| `--batch_size` | batch size của eval dataloader | `32` |
| `--plot_only` | chỉ tạo lại biểu đồ | off |

## 🧯 Troubleshooting

| Symptom | Likely cause | Fix |
|---|---|---|
| `../build/S4: No such file or directory` | Không có binary S4 ở đường dẫn tương đối mong đợi | Đặt/build S4 tại `../build/S4` hoặc sửa script |
| `Must specify either --data_npz or --csv_file` | Thiếu tham số dữ liệu cho train/eval | Cung cấp đúng một đầu vào dữ liệu |
| `No transmission columns found` | CSV đã gộp không có các cột `T@...` | Chạy lại merge/pivot và kiểm tra header |
| `KeyError: 'prefix'` in preprocess | Dữ liệu gộp vẫn dùng `folder_key`/`NQ` | Đổi tên cột thành `prefix`/`nQ` trước tiền xử lý |
| GPU OOM | batch quá lớn | Giảm `--batch_size` |
| Missing checkpoints during eval | thiếu checkpoint stage hoặc sai đường dẫn | Kiểm tra file stageA/B/C trong `--model_dir` đã chọn |

## 🧭 Roadmap

- Chuẩn hóa schema CSV đã gộp giữa các script merge (đặt tên `prefix`, `nQ`)
- Thêm kiểm thử tự động cho merge/preprocess/nạp checkpoint
- Cung cấp một CLI entrypoint duy nhất để điều phối toàn bộ pipeline
- Thêm manifest cho dataset/run để tăng khả năng tái lập
- Thêm giấy phép mã nguồn mở rõ ràng

## 📚 Citation

Nếu repository này đóng góp cho nghiên cứu của bạn, vui lòng trích dẫn:

```bibtex
@article{chen2025inverse,
  title={Inverse Design of Metasurface for Spectral Imaging},
  author={Chen, Rongzhou and Nie, Haitao and Zhu, Shuo and Zhao, Yaping and Wang, Chutian and Lam, Edmund Y},
  journal={arXiv preprint arXiv:2510.21924},
  year={2025}
}
```

## 📄 License

Repository hiện chưa có file `LICENSE`. Vì vậy, quyền sử dụng và phân phối lại hiện chưa được quy định rõ cho đến khi có giấy phép được thêm vào.

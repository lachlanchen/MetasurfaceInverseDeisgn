English | 中文繁體 | 中文简体 | 日本語 | 한국어 | Tiếng Việt | العربية | Français | Español | Deutsch | Русский

# inverse_metasurface

<div align="center">

[![Status](https://img.shields.io/badge/status-experimental-orange)](#-roadmap)
[![Python](https://img.shields.io/badge/python-3.9-blue)](#-prerequisites)
[![Platform](https://img.shields.io/badge/platform-linux-lightgrey)](#-prerequisites)
[![S4](https://img.shields.io/badge/S4-required-success)](#-prerequisites)
[![License](https://img.shields.io/badge/license-unset-lightgrey)](#-license)

</div>

Không gian nghiên cứu cho **thiết kế metasurface nghịch đảo** sử dụng **mô phỏng S4/RCWA** và **mô hình mạng nơ-ron nhiều giai đoạn**.

Repo này hỗ trợ:
- Tạo dữ liệu S4 thông lượng cao cho metasurface đa giác đối xứng C4.
- Gộp dữ liệu + tiền xử lý thành NPZ sẵn sàng cho huấn luyện.
- Học ba giai đoạn: **shape -> spectrum**, **spectrum -> shape**, **chuỗi tinh chỉnh spectrum -> shape -> spectrum**.
- Đánh giá và tùy chọn so sánh trực tiếp **neural-vs-S4**.

## 🌟 Tổng Quan Nhanh

| Khu vực | Repo này cung cấp gì |
|---|---|
| Mô phỏng vật lý | Script Bash + Lua để chạy S4 song song trên `nQ=1..4` |
| Công cụ dữ liệu | Script gộp dữ liệu gắn đỉnh đa giác và phổ pivot |
| Pipeline ML | `three_stage_transmittance.py` với chế độ preprocess + train |
| Đánh giá | `three_stage_transmittance_evaluation.py` với metric + hình vẽ |
| Nhánh nghiên cứu | AVIRIS/hyperspectral và thí nghiệm noise/compression |

## 🧠 Bối Cảnh Nghiên Cứu

Kho mã này hướng tới thiết kế nghịch đảo cho metasurface quang tử: suy ra hình học từ phổ mục tiêu (và ngược lại).

Các giả định cốt lõi được dùng trong pipeline baseline:
- Đối xứng C4 thông qua tham số hóa điểm Q1.
- 11 trạng thái kết tinh (`c` trong `[0.0, 1.0]`).
- Phổ của mỗi mẫu được lưu dưới dạng các hàng truyền qua `11 x 100`.
- Hình dạng được biểu diễn bằng tối đa 4 điểm Q1 (tensor `4 x 3`: presence, x, y).

Logic huấn luyện ba giai đoạn:
1. **Stage A**: shape -> spectrum (`shape2spec_stageA.pt`)
2. **Stage B**: spectrum -> shape (`spec2shape_stageB.pt`)
3. **Stage C**: tinh chỉnh chuỗi spectrum -> shape -> spectrum (`spec2shape_stageC.pt`)

## 🗂 Cấu Trúc Dự Án

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

## ✅ Điều Kiện Tiên Quyết

| Yêu cầu | Ghi chú |
|---|---|
| OS | Linux (script giả định Bash + đường dẫn Linux) |
| Python | 3.9 (từ `iccp.yaml`) |
| Trình quản lý môi trường | Khuyến nghị Conda |
| S4 binary | Dự kiến tại `../build/S4` tương đối so với thư mục gốc repo |
| GPU (tùy chọn) | CUDA giúp tăng tốc huấn luyện/đánh giá |

## ⚙️ Cài Đặt

### 1) Clone và vào thư mục

```bash
git clone <your-repo-url> inverse_metasurface
cd inverse_metasurface
```

### 2) Tạo môi trường

```bash
conda env create -f iccp.yaml
conda activate iccp
```

### 3) Kiểm tra đường dẫn S4

```bash
ls -l ../build/S4
```

Nếu đường dẫn này không tồn tại, hãy build/đặt S4 tại đó hoặc chỉnh lại đường dẫn trong script.

## 🚀 Khởi Động Nhanh (End-to-End)

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

## 🧪 Chi Tiết Sử Dụng

### A) Tạo dữ liệu S4

Chạy tối thiểu với seed:

```bash
./ms.sh -ns 10000 -r 88888
```

Chạy có tham số:

```bash
./ms_final.sh \
  -ns 100000 \
  -r 88888 \
  -p iccpOv100kG80 \
  -g 80 \
  -bo 0.35 \
  -ro 0.30
```

Chạy kiểu resume:

```bash
./ms_resume_allargs.sh \
  -ns 100000 \
  -r 88888 \
  -p iccpOv100kG80 \
  -g 80 \
  -bo 0.35 \
  -ro 0.30
```

### B) Gộp các CSV S4 thô

```bash
python merge.py --prefix 20250123_155420
```

Tiện ích thay thế:

```bash
python merge_s4_data_full.py --prefix 20250123_155420
```

### C) Tiền xử lý CSV đã gộp -> NPZ

```bash
python three_stage_transmittance.py --preprocess \
  --input_folder merged_csvs \
  --output_npz preprocessed_t_data.npz
```

### D) Huấn luyện

```bash
python three_stage_transmittance.py \
  --data_npz preprocessed_t_data.npz \
  --num_epochs 100 \
  --batch_size 1024
```

Kết quả huấn luyện được lưu tại:
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageA`
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageB`
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageC`

### E) Đánh giá

```bash
python three_stage_transmittance_evaluation.py \
  --model_dir outputs_three_stage_YYYYMMDD_HHMMSS \
  --data_npz preprocessed_t_data.npz \
  --sample_count 8
```

Sinh ra:
- `evaluation_metrics.csv`
- `metrics_summary.csv`
- trực quan hóa theo từng stage (`.png`, `.pdf`)
- biểu đồ đường cong huấn luyện

### F) So sánh Neural và S4 trực tiếp (tùy chọn)

```bash
python FilterShapeS4_Evaluator_Transmittance.py \
  --npz_file preprocessed_t_data.npz \
  --spec2shape_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageC/spec2shape_stageC.pt \
  --shape2spec_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageA/shape2spec_stageA.pt \
  --n_samples 4
```

## 🔧 Tùy Chọn CLI Chính

### Cờ cho S4 runner (`ms_final.sh`, `ms_resume_allargs.sh`)

| Flag | Ý nghĩa | Mặc định |
|---|---|---|
| `-ns`, `--numshapes` | Số lượng shape | `100000` |
| `-r`, `--seed` | Seed ngẫu nhiên | `88888` |
| `-p`, `--prefix` | Tiền tố run / khóa resume | empty |
| `-g`, `--numg` | Tham số hình học/basis | `80` |
| `-bo`, `--baseouter` | Độ lệch outer cơ sở | `0.25` |
| `-ro`, `--randouter` | Độ lệch outer ngẫu nhiên | `0.20` |

### Cờ huấn luyện (`three_stage_transmittance.py`)

| Flag | Ý nghĩa | Mặc định |
|---|---|---|
| `--preprocess` | Chạy chế độ tiền xử lý | off |
| `--input_folder` | Thư mục CSV đầu vào | empty |
| `--output_npz` | Đường dẫn NPZ đầu ra | `preprocessed_data.npz` |
| `--data_npz` | NPZ dùng cho train/eval | empty |
| `--csv_file` | CSV dùng trực tiếp | empty |
| `--num_epochs` | Số epoch mỗi stage | `10` |
| `--batch_size` | Kích thước batch | `4096` |
| `--test` | Placeholder cho chế độ test | off |

### Cờ đánh giá (`three_stage_transmittance_evaluation.py`)

| Flag | Ý nghĩa | Mặc định |
|---|---|---|
| `--model_dir` | Thư mục gốc của run đã huấn luyện | required |
| `--data_npz` | NPZ để đánh giá | empty |
| `--csv_file` | CSV để đánh giá | empty |
| `--output_dir` | Thư mục đầu ra | `model_dir/evaluation_<timestamp>` |
| `--sample_count` | Số mẫu được trực quan hóa | `4` |
| `--seed` | Seed lấy mẫu | `23` |
| `--font_scale` | Tỉ lệ font cho biểu đồ | `1.0` |
| `--batch_size` | Batch size khi eval | `32` |
| `--plot_only` | Chỉ vẽ đường cong | off |

## 🧭 Khắc Phục Sự Cố

| Triệu chứng | Nguyên nhân có thể | Cách khắc phục |
|---|---|---|
| `../build/S4: No such file or directory` | Sai đường dẫn S4 binary | Build/đặt S4 tại `../build/S4` hoặc sửa script |
| `Must specify either --data_npz or --csv_file` | Thiếu nguồn dữ liệu | Truyền rõ một trong hai cờ đó |
| `No matching CSVs found in 'results/'` | Prefix không khớp | Kiểm tra prefix và quy ước đặt tên đầu ra |
| Rất ít/không có bản ghi sau tiền xử lý | Thiếu hoặc lỗi các dòng `T@...` hay `vertices_str` | Kiểm tra schema CSV đã gộp và nhóm dòng theo từng shape |
| CUDA OOM | Batch quá lớn | Giảm `--batch_size` (ví dụ `1024 -> 256`) |

## 🧱 Ghi Chú Phát Triển

- Đây là repo nhiều thí nghiệm; nhiều artifact sinh ra được cố ý không theo dõi.
- Có nhiều biến thể script cho tác vụ tương tự (`merge_*`, `aviris_*`, `noise_experiment_*`).
- Luồng inverse-metasurface baseline là lộ trình được ghi trong README này.
- Script huấn luyện có đặt seed (`42`), nhưng tính tất định nghiêm ngặt vẫn phụ thuộc phần cứng/backend.
- Hiện chưa có CI thống nhất + bộ test tự động đầy đủ.

## 🛣 Lộ Trình

- Thêm bộ dữ liệu chuẩn gọn nhẹ để smoke test nhanh.
- Hợp nhất các script pipeline bị trùng lặp.
- Thêm kiểm tra tính toàn vẹn merge/preprocess và khả năng nạp checkpoint.
- Thêm CI cho lint + smoke train/eval.
- Tài liệu hóa build/version pinning của S4 rõ ràng hơn.

## 🤝 Đóng Góp

1. Tạo feature branch.
2. Giữ phạm vi PR gọn (mỗi PR tập trung một vấn đề pipeline/thí nghiệm).
3. Đính kèm lệnh chạy chính xác và đường dẫn đầu ra mong đợi.
4. Tránh commit artifact sinh ra có dung lượng lớn trừ khi cần thiết.
5. Bổ sung ghi chú tái lập (seed, nguồn dữ liệu, đường dẫn checkpoint).

## 📄 Giấy Phép

Hiện tại chưa có file `LICENSE` trong repo này.

Cho đến khi có file giấy phép, điều khoản tái sử dụng và phân phối lại nên được xem là **chưa xác định**.

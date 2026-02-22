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

Kho mã nghiên cứu theo định hướng script (trước đây thường được gọi là `inverse_metasurface`) cho bài toán thiết kế ngược metasurface trong ảnh phổ. Quy trình kết hợp **mô phỏng RCWA dựa trên S4** với **pipeline PyTorch ba giai đoạn** để ánh xạ thuận và nghịch giữa hình học cấu trúc và phổ truyền qua quang học.

## ✨ Tổng Quan Nhanh

| Hạng mục | Chi tiết |
|---|---|
| 🎯 Mục tiêu | Dự đoán hình học metasurface đối xứng C4 từ phổ truyền qua mục tiêu |
| 🔬 Vật lý | Mô phỏng RCWA với S4 (`../build/S4`) |
| 🧠 Pipeline học máy | Giai đoạn A `shape -> spectra`, Giai đoạn B `spectra -> shape`, Giai đoạn C `spectra -> shape -> spectra` |
| 📦 Dạng dữ liệu | CSV hợp nhất (`T@...`, metadata hình dạng) -> NPZ nén (`uids`, `spectra`, `shapes`) |
| 🧪 Đánh giá | Chỉ số MSE, biểu đồ định tính, tùy chọn kiểm tra mô phỏng lại bằng S4 |

## 🧭 Quy Trình End-to-End

1. Sinh đáp ứng quang học metasurface bằng S4 (`.lua` + script shell khởi chạy).
2. Gộp các file CSV mô phỏng thô và gắn các đỉnh đa giác.
3. Chuyển CSV đã gộp sang NPZ phục vụ huấn luyện.
4. Huấn luyện pipeline truyền qua ba giai đoạn.
5. Đánh giá checkpoint và trực quan hành vi Stage A/B/C.
6. Tùy chọn so sánh dự đoán của mạng với mô phỏng S4 mới.

## 🧱 Cấu Trúc Kho Mã

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
├── results/                # đầu ra CSV thô từ S4
├── shapes/                 # file đỉnh đa giác dùng trong bước merge
├── merged_csvs/            # bộ dữ liệu CSV đã hợp nhất
├── outputs_three_stage_*/  # checkpoint huấn luyện và đường cong
├── partial_crys_data/      # bảng quang học theo trạng thái kết tinh
│
├── AVIRIS*/                # thí nghiệm siêu phổ phụ
├── noise_experiment_*/     # nhánh thử độ bền/nhiễu
└── archived/               # script và snapshot lịch sử
```

## 🛠️ Điều Kiện Tiên Quyết

- Linux + Bash
- Conda (khuyến nghị)
- Python 3.9
- Có sẵn binary S4 tại `../build/S4`
- Tùy chọn: GPU hỗ trợ CUDA để huấn luyện nhanh hơn

## 🚀 Cài Đặt

```bash
git clone <your-repo-url> inverse_metasurface
cd inverse_metasurface

conda env create -f iccp.yaml
conda activate iccp

# Kiểm tra đường dẫn simulator mà script kỳ vọng
ls -l ../build/S4
```

Tùy chọn:

```bash
chmod +x ms.sh ms_final.sh ms_resume_allargs.sh
```

## ▶️ Cách Dùng Thực Tế

### 1) Tạo dữ liệu RCWA

`ms_final.sh` và `ms_resume_allargs.sh` mỗi script sẽ chạy 4 tác vụ S4 song song (`NQ=1..4`):

```bash
./ms_final.sh \
  -ns 10000 \
  -r 12345 \
  -p myrun \
  -g 80 \
  -bo 0.35 \
  -ro 0.30
```

Lưu ý:
- `ms.sh` là đường chạy đơn giản hơn.
- Các launcher giả định `../build/S4` và dùng `-t 32`.

### 2) Hợp nhất đầu ra S4 + đỉnh hình dạng

```bash
python merge_s4_data_full.py --prefix myrun
# output: merged_s4_shapes_myrun.csv
```

### 3) Chuẩn hóa tên cột để tương thích huấn luyện

`merge_s4_data_full.py` ghi `folder_key` / `NQ`, trong khi `three_stage_transmittance.py` kỳ vọng `prefix` / `nQ`.

```bash
python -c "import pandas as pd; p='merged_s4_shapes_myrun.csv'; df=pd.read_csv(p); df=df.rename(columns={'folder_key':'prefix','NQ':'nQ'}); df.to_csv(p,index=False)"
```

### 4) Tiền xử lý CSV sang NPZ

```bash
mkdir -p merged_csvs
mv merged_s4_shapes_myrun.csv merged_csvs/

python three_stage_transmittance.py \
  --preprocess \
  --input_folder merged_csvs \
  --output_npz preprocessed_t_data.npz
```

### 5) Huấn luyện pipeline ba giai đoạn

```bash
python three_stage_transmittance.py \
  --data_npz preprocessed_t_data.npz \
  --num_epochs 100 \
  --batch_size 1024
```

Mẫu đầu ra dự kiến:
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageA`
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageB`
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageC`

### 6) Đánh giá mô hình Stage A/B/C

```bash
python three_stage_transmittance_evaluation.py \
  --model_dir outputs_three_stage_YYYYMMDD_HHMMSS \
  --data_npz preprocessed_t_data.npz \
  --sample_count 8
```

### 7) Tùy chọn kiểm tra độ nhất quán mạng vs S4

```bash
python FilterShapeS4_Evaluator_Transmittance.py \
  --npz_file preprocessed_t_data.npz \
  --spec2shape_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageC/spec2shape_stageC.pt \
  --shape2spec_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageA/shape2spec_stageA.pt \
  --n_samples 4
```

## ⚙️ Tùy Chọn CLI Chính

### S4 Launcher (`ms_final.sh`, `ms_resume_allargs.sh`)

| Flag | Ý nghĩa | Mặc định |
|---|---|---|
| `-ns`, `--numshapes` | Số lượng shape cần tạo | `100000` |
| `-r`, `--seed` | Hạt giống ngẫu nhiên | `88888` |
| `-p`, `--prefix` | Prefix/resume key | `""` |
| `-g`, `--numg` | Tham số basis/hình học | `80` |
| `-bo`, `--baseouter` | Độ lệch biên ngoài cơ sở | `0.25` |
| `-ro`, `--randouter` | Độ lệch ngoài ngẫu nhiên | `0.20` |

### Huấn Luyện (`three_stage_transmittance.py`)

| Flag | Mục đích | Mặc định |
|---|---|---|
| `--preprocess` | Chạy chế độ tiền xử lý | `False` |
| `--input_folder` | Thư mục chứa CSV đã hợp nhất | `""` |
| `--output_npz` | Đường dẫn NPZ đầu ra | `preprocessed_data.npz` |
| `--data_npz` | NPZ dùng để huấn luyện | `""` |
| `--csv_file` | CSV dự phòng nếu không dùng NPZ | `""` |
| `--test` | Chế độ test (bỏ qua huấn luyện) | `False` |
| `--num_epochs` | Số epoch | `10` |
| `--batch_size` | Kích thước batch | `4096` |

### Đánh Giá (`three_stage_transmittance_evaluation.py`)

| Flag | Mục đích | Mặc định |
|---|---|---|
| `--model_dir` | Thư mục gốc chứa `stageA/B/C` | bắt buộc |
| `--data_npz` | NPZ đầu vào để đánh giá | `""` |
| `--csv_file` | CSV đầu vào để đánh giá | `""` |
| `--output_dir` | Ghi đè thư mục đầu ra | tự động trong `model_dir` |
| `--sample_count` | Số mẫu được trực quan hóa | `4` |
| `--seed` | Hạt giống ngẫu nhiên để lấy mẫu | `23` |
| `--font_scale` | Tỷ lệ font cho biểu đồ | `1.0` |
| `--batch_size` | Kích thước batch khi eval | `32` |
| `--plot_only` | Chỉ vẽ đường cong | `False` |

### Bộ Đánh Giá Nhất Quán S4 (`FilterShapeS4_Evaluator_Transmittance.py`)

| Flag | Mục đích | Mặc định |
|---|---|---|
| `--npz_file` | File NPZ đã tiền xử lý | `preprocessed_t_data.npz` |
| `--spec2shape_ckpt` | Checkpoint Stage C | `outputs_three_stage_20250322_145925/stageC/spec2shape_stageC.pt` |
| `--shape2spec_ckpt` | Checkpoint Stage A | `outputs_three_stage_20250322_145925/stageA/shape2spec_stageA.pt` |
| `--n_samples` | Số mẫu cần kiểm tra | `4` |
| `--seed` | Hạt giống ngẫu nhiên | `23` |
| `--max_workers` | Số worker S4 song song | `4` |
| `--out_folder` | Thư mục đầu ra | tự động theo timestamp |

## 🧪 Chạy Smoke Test Nhanh

```bash
./ms_final.sh -ns 1000 -r 42 -p smoke -g 40 -bo 0.25 -ro 0.20
python merge_s4_data_full.py --prefix smoke
python -c "import pandas as pd; p='merged_s4_shapes_smoke.csv'; d=pd.read_csv(p).rename(columns={'folder_key':'prefix','NQ':'nQ'}); d.to_csv(p,index=False)"
python three_stage_transmittance.py --preprocess --input_folder . --output_npz smoke.npz
python three_stage_transmittance.py --data_npz smoke.npz --num_epochs 5 --batch_size 128
```

## 🔬 Bối Cảnh Nghiên Cứu

Thiết lập thiết kế ngược cốt lõi suy ra hình học metasurface đối xứng C4 từ phổ truyền qua trên các trạng thái kết tinh. Nhánh truyền qua hiện tại giả định:

- 11 trạng thái kết tinh cho mỗi mẫu (giá trị `c`, được sắp xếp trong bước tiền xử lý)
- 100 bin bước sóng cho mỗi trạng thái (các cột `T@...`)
- tối đa 4 đỉnh Q1 được mã hóa dạng `(presence, x, y)`

Kho mã này cũng bao gồm các nhánh khám phá (ví dụ `AVIRIS*`, `noise_experiment_*`, và `archived/`) ngoài pipeline truyền qua chuẩn.

## 📚 Trích Dẫn

Nếu bạn sử dụng kho mã này hoặc phát triển tiếp từ công trình này, vui lòng trích dẫn:

```bibtex
@article{chen2025inverse,
  title={Inverse Design of Metasurface for Spectral Imaging},
  author={Chen, Rongzhou and Nie, Haitao and Zhu, Shuo and Zhao, Yaping and Wang, Chutian and Lam, Edmund Y},
  journal={arXiv preprint arXiv:2510.21924},
  year={2025}
}
```

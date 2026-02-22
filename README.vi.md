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

Đây là kho nghiên cứu theo hướng script-first (trước đây thường được gọi là `inverse_metasurface`) cho **thiết kế ngược metasurface đối xứng C4** trong ảnh phổ, bao gồm:

- sinh dữ liệu RCWA bằng S4 (`.lua` + script chạy shell)
- gộp dữ liệu và tiền xử lý (`.csv` -> `.npz`)
- huấn luyện mạng nơ-ron ba giai đoạn (shape->spectra, spectra->shape, tinh chỉnh theo chuỗi)
- đánh giá và tùy chọn đối chiếu mô hình nơ-ron với S4

## ✨ Tổng Quan Nhanh

| Hạng mục | Chi tiết |
|---|---|
| Mục tiêu cốt lõi | Dự đoán hình học từ phổ truyền qua mục tiêu |
| Kích thước dữ liệu cốt lõi | spectra: `11 x 100`, shape: `4 x 3` |
| Script huấn luyện chính | `three_stage_transmittance.py` |
| Script đánh giá chính | `three_stage_transmittance_evaluation.py` |
| Script khởi chạy RCWA | `ms_final.sh`, `ms_resume_allargs.sh` |
| Script gộp dữ liệu | `merge_s4_data_full.py` |

## 🧠 Bối Cảnh Nghiên Cứu

Dự án này tập trung vào thiết kế ngược metasurface cho ảnh phổ. Pipeline huấn luyện dùng phổ truyền qua do S4 sinh ra trên nhiều trạng thái kết tinh, đồng thời học cả ánh xạ thuận và nghịch:

1. **Giai đoạn A**: shape -> spectra
2. **Giai đoạn B**: spectra -> shape
3. **Giai đoạn C**: spectra -> shape -> spectra (tinh chỉnh chain loss)

Mã tiền xử lý/huấn luyện hiện giả định:

- 11 trạng thái kết tinh (`c = 0.0 ... 1.0`)
- 100 bin bước sóng cho mỗi trạng thái
- biểu diễn shape gồm tối đa 4 điểm Q1 với định dạng `[presence, x, y]`

## 🗂️ Cấu Trúc Repository (Đường Dẫn Cốt Lõi)

```text
.
├── ms.sh / ms_final.sh / ms_resume_allargs.sh
├── metasurface_final.lua / metasurface_allargs_resume.lua / metasurface_seed.lua
├── merge_s4_data_full.py
├── three_stage_transmittance.py
├── three_stage_transmittance_evaluation.py
├── FilterShapeS4_Evaluator_Transmittance.py
├── results/                # các file CSV đầu ra S4 thô
├── shapes/                 # đỉnh đa giác được sinh ra
├── merged_csvs/            # CSV đã gộp dùng cho tiền xử lý
├── outputs_three_stage_*/  # checkpoint, loss, trực quan hóa
├── partial_crys_data/
└── iccp.yaml
```

## ⚙️ Điều Kiện Tiên Quyết

| Phụ thuộc | Yêu cầu |
|---|---|
| Hệ điều hành | Linux |
| Shell | Bash |
| Python | 3.9 |
| Trình quản lý môi trường | Conda (khuyến nghị) |
| Binary RCWA | `../build/S4` (tương đối từ thư mục gốc repo) |
| GPU | Không bắt buộc, khuyến nghị để huấn luyện nhanh hơn |

## 🚀 Cài Đặt

```bash
git clone <repo-url> inverse_metasurface
cd inverse_metasurface

conda env create -f iccp.yaml
conda activate iccp

# Bắt buộc cho các script khởi chạy
ls -l ../build/S4
```

Tùy chọn:

```bash
chmod +x ms.sh ms_final.sh ms_resume_allargs.sh
```

## 🧪 Quy Trình Sử Dụng End-to-End

### 1) Sinh dữ liệu RCWA bằng S4

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

- Các launcher gọi `../build/S4` với `-t 32` và chạy song song `NQ=1..4`.
- `ms_final.sh` sử dụng `metasurface_final.lua`.
- `ms_resume_allargs.sh` sử dụng `metasurface_allargs_resume.lua`.

### 2) Gộp đầu ra RCWA với các đỉnh shape

```bash
python merge_s4_data_full.py --prefix myrun
# -> merged_s4_shapes_myrun.csv
```

### 3) Chuẩn hóa tên cột đã gộp cho huấn luyện (nếu cần)

`merge_s4_data_full.py` ghi `folder_key` / `NQ`, trong khi pipeline huấn luyện kỳ vọng `prefix` / `nQ`.

```bash
python -c "import pandas as pd; p='merged_s4_shapes_myrun.csv'; df=pd.read_csv(p); df=df.rename(columns={'folder_key':'prefix','NQ':'nQ'}); df.to_csv(p,index=False)"
```

### 4) Tiền xử lý CSV -> NPZ

```bash
mkdir -p merged_csvs
mv merged_s4_shapes_myrun.csv merged_csvs/

python three_stage_transmittance.py \
  --preprocess \
  --input_folder merged_csvs \
  --output_npz preprocessed_t_data.npz
```

### 5) Huấn luyện mô hình ba giai đoạn

```bash
python three_stage_transmittance.py \
  --data_npz preprocessed_t_data.npz \
  --num_epochs 100 \
  --batch_size 1024
```

Cấu trúc đầu ra chính:

- `outputs_three_stage_YYYYMMDD_HHMMSS/stageA`
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageB`
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageC`

### 6) Đánh giá checkpoint

```bash
python three_stage_transmittance_evaluation.py \
  --model_dir outputs_three_stage_YYYYMMDD_HHMMSS \
  --data_npz preprocessed_t_data.npz \
  --sample_count 8
```

### 7) Tùy chọn: so sánh dự đoán nơ-ron với S4

```bash
python FilterShapeS4_Evaluator_Transmittance.py \
  --npz_file preprocessed_t_data.npz \
  --spec2shape_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageC/spec2shape_stageC.pt \
  --shape2spec_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageA/shape2spec_stageA.pt \
  --n_samples 4
```

## 🎛️ Tham Chiếu CLI

### Cờ launcher S4 (`ms_final.sh`, `ms_resume_allargs.sh`)

| Cờ | Ý nghĩa | Mặc định |
|---|---|---|
| `-ns`, `--numshapes` | Số lượng shape | `100000` |
| `-r`, `--seed` | Seed ngẫu nhiên | `88888` |
| `-p`, `--prefix` | Prefix phiên chạy / khóa resume | `""` |
| `-g`, `--numg` | Tham số cơ sở hình học | `80` |
| `-bo`, `--baseouter` | Độ lệch outer cơ sở | `0.25` |
| `-ro`, `--randouter` | Độ lệch outer ngẫu nhiên | `0.20` |

### Cờ huấn luyện (`three_stage_transmittance.py`)

| Cờ | Mục đích |
|---|---|
| `--preprocess` | Chạy chế độ tiền xử lý |
| `--input_folder` | Thư mục chứa các file CSV đã gộp |
| `--output_npz` | Tên file NPZ tiền xử lý đầu ra |
| `--data_npz` | Dữ liệu NPZ dùng để huấn luyện |
| `--csv_file` | Tùy chọn dữ liệu CSV thay thế |
| `--test` | Chế độ kiểm thử |
| `--num_epochs` | Số epoch huấn luyện |
| `--batch_size` | Kích thước batch |

### Cờ đánh giá (`three_stage_transmittance_evaluation.py`)

| Cờ | Mục đích |
|---|---|
| `--model_dir` | Thư mục gốc checkpoint (bắt buộc) |
| `--data_npz` / `--csv_file` | Nguồn dữ liệu đánh giá |
| `--output_dir` | Thư mục đầu ra đánh giá |
| `--sample_count` | Số mẫu được trực quan hóa |
| `--seed` | Seed ngẫu nhiên cho chọn mẫu |
| `--font_scale` | Tỉ lệ font cho biểu đồ |
| `--batch_size` | Kích thước batch khi đánh giá |
| `--plot_only` | Chỉ tạo lại biểu đồ đường cong huấn luyện |

## 🧾 Data Contract (cho tiền xử lý)

Nhánh tiền xử lý trong `three_stage_transmittance.py` kỳ vọng các CSV đã gộp có chứa:

- cột ID: `prefix`, `nQ`, `nS`, `shape_idx`, `c`
- trường hình học dạng text: `vertices_str`
- cột phổ: `T@...`

Các kiểm tra chất lượng được mã áp dụng:

- nhóm theo `shape_uid = prefix_nQ_nS_shape_idx`
- mỗi nhóm phải chứa đúng 11 dòng
- chỉ giữ các shape có số điểm Q1 trong khoảng `[1, 4]`

## 🛠️ Khắc Phục Sự Cố

- `../build/S4: No such file or directory`
  - Build/liên kết S4 tại `../build/S4` hoặc chỉnh script launcher theo đường dẫn S4 thực tế của bạn.
- `No matching CSVs found in 'results/'`
  - Kiểm tra `--prefix` và quy ước đặt tên đầu ra trong `results/*_output_nQ*_nS*.csv`.
- `No transmission columns found`
  - Đảm bảo CSV đã gộp có các cột `T@...`.
- Tiền xử lý cho ra 0 bản ghi
  - Kiểm tra các cột bắt buộc và đảm bảo mỗi shape UID có 11 dòng trạng thái kết tinh.
- GPU OOM khi huấn luyện
  - Giảm `--batch_size` (ví dụ `256` hoặc `128`).
- Đánh giá không tìm thấy checkpoint
  - Xác nhận các file sau tồn tại trong `--model_dir`:
    - `stageA/shape2spec_stageA.pt`
    - `stageB/spec2shape_stageB.pt`
    - `stageC/spec2shape_stageC.pt`

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

## 🌐 Các Phiên Bản Ngôn Ngữ

Repository này có thêm các biến thể README, gồm:

- `README.en.md`, `README.de.md`, `README.es.md`, `README.fr.md`
- `README.ru.md`, `README.ja.md`, `README.ko.md`, `README.vi.md`
- `README.ar.md`, `README.zh-CN.md`, `README.zh-TW.md`

## 📌 Ghi Chú

- Đây là workspace nghiên cứu với nhiều script lưu trữ và thử nghiệm.
- Luồng transmittance chuẩn tập trung vào:
  - `ms_final.sh` / `ms_resume_allargs.sh`
  - `merge_s4_data_full.py`
  - `three_stage_transmittance.py`
  - `three_stage_transmittance_evaluation.py`
- Hiện chưa có file license rõ ràng ở thư mục gốc repository.

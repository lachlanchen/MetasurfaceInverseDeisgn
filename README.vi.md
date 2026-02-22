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

Kho mã nghiên cứu ưu tiên chạy bằng script (trước đây thường gọi là `inverse_metasurface`) cho bài toán thiết kế ngược metasurface trong ảnh phổ. Quy trình cốt lõi kết hợp:

- mô phỏng RCWA bám sát vật lý (S4 + Lua)
- tổng hợp dữ liệu và gắn thông tin hình học
- pipeline học PyTorch 3 giai đoạn (`shape -> spectrum`, `spectrum -> shape`, fine-tune chuỗi)
- đánh giá định lượng và định tính, kèm tùy chọn kiểm tra độ nhất quán giữa mạng nơ-ron và S4

## ✨ Tổng Quan Nhanh

| Mục | Chi tiết |
|---|---|
| 🎯 Tác vụ chính | Suy ra hình học metasurface đối xứng C4 từ phổ truyền qua mục tiêu |
| 🔬 Bộ mô phỏng | `../build/S4` được gọi bởi các launcher shell và script `.lua` |
| 🧠 Pipeline học | Giai đoạn A `shape -> spectra`, Giai đoạn B `spectra -> shape`, Giai đoạn C `spectra -> shape -> spectra` |
| 📦 Hợp đồng dữ liệu | CSV hợp nhất (`T@...`, metadata, `vertices_str`) -> NPZ nén (`uids`, `spectra`, `shapes`) |
| 🧪 Đánh giá | Chỉ số MSE, trực quan hóa theo giai đoạn, tùy chọn chạy lại S4 mới |

## 🧭 Quy Trình End-to-End

1. Tạo kết quả mô phỏng trong `results/` và file polygon trong `shapes/`.
2. Hợp nhất các file CSV từ S4 và gắn đỉnh hình học.
3. Chuẩn hóa tên cột đã hợp nhất để tương thích huấn luyện.
4. Tiền xử lý CSV đã hợp nhất thành tensor NPZ.
5. Huấn luyện mô hình Stage A/B/C.
6. Đánh giá checkpoint và trực quan hóa hành vi mô hình.
7. (Tùy chọn) So sánh phổ từ hình dạng dự đoán với kết quả S4 chạy mới.

## 🧱 Cấu Trúc Repository

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

## 🛠️ Yêu Cầu Trước Khi Chạy

| Phụ thuộc | Ghi chú |
|---|---|
| Linux + Bash | Các script launcher được thiết kế để chạy qua shell |
| Python 3.9 | Khớp với `iccp.yaml` |
| Conda | Khuyến nghị để đảm bảo khả năng tái lập |
| S4 binary | Được kỳ vọng tại `../build/S4` |
| CUDA GPU (tùy chọn) | Tăng tốc huấn luyện/đánh giá |

## 🚀 Thiết Lập

### 1) Clone và vào thư mục

```bash
git clone <your-repo-url> inverse_metasurface
cd inverse_metasurface
```

### 2) Tạo môi trường (khuyến nghị)

```bash
conda env create -f iccp.yaml
conda activate iccp
```

Phương án khác (ít được kiểm soát hơn):

```bash
pip install -r pip_requirements.txt
```

### 3) Kiểm tra đường dẫn bộ mô phỏng mà script yêu cầu

```bash
ls -l ../build/S4
```

### 4) (Tùy chọn) cấp quyền thực thi cho launcher

```bash
chmod +x ms.sh ms_final.sh ms_resume.sh ms_resume_allargs.sh ms_resume_random_state.sh
```

## ▶️ Cách Dùng Thực Tế

### A) Sinh dữ liệu mô phỏng RCWA

Launcher đơn giản:

```bash
./ms.sh -ns 10000 -r 12345
```

Launcher có tham số:

```bash
./ms_final.sh \
  -ns 10000 \
  -r 12345 \
  -p myrun \
  -g 80 \
  -bo 0.35 \
  -ro 0.30
```

Launcher phục vụ tiếp tục chạy (resume):

```bash
./ms_resume_allargs.sh \
  -ns 10000 \
  -r 12345 \
  -p myrun \
  -g 80 \
  -bo 0.35 \
  -ro 0.30
```

Lưu ý:
- Launcher chạy song song `NQ=1..4`.
- Script gọi `../build/S4` với `-t 32`.

### B) Hợp nhất đầu ra S4 và gắn đỉnh hình học

```bash
python merge_s4_data_full.py --prefix myrun
# output: merged_s4_shapes_myrun.csv
```

### C) Chuẩn hóa cột để tương thích huấn luyện

`merge_s4_data_full.py` ghi ra `folder_key` và `NQ`, trong khi pipeline huấn luyện cần `prefix` và `nQ`.

```bash
python -c "import pandas as pd; p='merged_s4_shapes_myrun.csv'; df=pd.read_csv(p); df=df.rename(columns={'folder_key':'prefix','NQ':'nQ'}); df.to_csv(p,index=False)"
```

### D) Tiền xử lý CSV -> NPZ

```bash
mkdir -p merged_csvs
mv merged_s4_shapes_myrun.csv merged_csvs/

python three_stage_transmittance.py \
  --preprocess \
  --input_folder merged_csvs \
  --output_npz preprocessed_t_data.npz
```

### E) Huấn luyện Stage A/B/C

```bash
python three_stage_transmittance.py \
  --data_npz preprocessed_t_data.npz \
  --num_epochs 100 \
  --batch_size 1024
```

Đầu ra được ghi vào:
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageA`
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageB`
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageC`

### F) Đánh giá mô hình đã huấn luyện

```bash
python three_stage_transmittance_evaluation.py \
  --model_dir outputs_three_stage_YYYYMMDD_HHMMSS \
  --data_npz preprocessed_t_data.npz \
  --sample_count 8
```

### G) (Tùy chọn) kiểm tra độ nhất quán giữa mạng và S4

```bash
python FilterShapeS4_Evaluator_Transmittance.py \
  --npz_file preprocessed_t_data.npz \
  --spec2shape_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageC/spec2shape_stageC.pt \
  --shape2spec_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageA/shape2spec_stageA.pt \
  --n_samples 4
```

## ⚙️ Tùy Chọn CLI Chính

### S4 launchers (`ms_final.sh`, `ms_resume_allargs.sh`)

| Flag | Ý nghĩa | Mặc định |
|---|---|---|
| `-ns`, `--numshapes` | Số lượng shape cần tạo | `100000` |
| `-r`, `--seed` | Seed ngẫu nhiên | `88888` |
| `-p`, `--prefix` | Khóa prefix/resume | `""` |
| `-g`, `--numg` | Tham số basis/grid | `80` |
| `-bo`, `--baseouter` | Độ lệch biên ngoài cơ sở | `0.25` |
| `-ro`, `--randouter` | Độ lệch biên ngoài ngẫu nhiên | `0.20` |

### Huấn luyện (`three_stage_transmittance.py`)

| Flag | Ý nghĩa | Mặc định |
|---|---|---|
| `--preprocess` | Chạy chế độ tiền xử lý | `False` |
| `--input_folder` | Thư mục chứa CSV đã hợp nhất | `""` |
| `--output_npz` | Đường dẫn NPZ đầu ra | `preprocessed_data.npz` |
| `--data_npz` | Dataset NPZ dùng cho huấn luyện | `""` |
| `--csv_file` | CSV dự phòng nếu không dùng NPZ | `""` |
| `--test` | Chế độ kiểm thử | `False` |
| `--num_epochs` | Số epoch huấn luyện | `10` |
| `--batch_size` | Kích thước batch | `4096` |

### Đánh giá (`three_stage_transmittance_evaluation.py`)

| Flag | Ý nghĩa | Mặc định |
|---|---|---|
| `--model_dir` | Thư mục chứa `stageA/B/C` | bắt buộc |
| `--data_npz` | NPZ đầu vào | `""` |
| `--csv_file` | CSV đầu vào dự phòng | `""` |
| `--output_dir` | Ghi đè thư mục đầu ra | tự động dưới `model_dir` |
| `--sample_count` | Số mẫu trực quan hóa | `4` |
| `--seed` | Seed ngẫu nhiên | `23` |
| `--font_scale` | Hệ số cỡ chữ cho biểu đồ | `1.0` |
| `--batch_size` | Batch size khi đánh giá | `32` |
| `--plot_only` | Chỉ vẽ đường cong huấn luyện | `False` |

### Bộ đánh giá nhất quán S4 (`FilterShapeS4_Evaluator_Transmittance.py`)

| Flag | Ý nghĩa | Mặc định |
|---|---|---|
| `--npz_file` | File NPZ đầu vào | `preprocessed_t_data.npz` |
| `--spec2shape_ckpt` | Đường dẫn checkpoint Stage C | `outputs_three_stage_20250322_145925/stageC/spec2shape_stageC.pt` |
| `--shape2spec_ckpt` | Đường dẫn checkpoint Stage A | `outputs_three_stage_20250322_145925/stageA/shape2spec_stageA.pt` |
| `--n_samples` | Số mẫu được đánh giá | `4` |
| `--seed` | Seed ngẫu nhiên | `23` |
| `--max_workers` | Số luồng worker S4 | `4` |
| `--out_folder` | Thư mục đầu ra | timestamp tự động |

## 🧪 Chạy Smoke Test

```bash
./ms_final.sh -ns 1000 -r 42 -p smoke -g 40 -bo 0.25 -ro 0.20
python merge_s4_data_full.py --prefix smoke
python -c "import pandas as pd; p='merged_s4_shapes_smoke.csv'; d=pd.read_csv(p).rename(columns={'folder_key':'prefix','NQ':'nQ'}); d.to_csv(p,index=False)"
mkdir -p merged_csvs && mv merged_s4_shapes_smoke.csv merged_csvs/
python three_stage_transmittance.py --preprocess --input_folder merged_csvs --output_npz smoke.npz
python three_stage_transmittance.py --data_npz smoke.npz --num_epochs 5 --batch_size 128
```

## 🔬 Bối Cảnh Nghiên Cứu

Thiết lập thiết kế ngược hiện tại học cách khôi phục hình học đối xứng C4 từ phổ truyền qua trên các trạng thái kết tinh. Pipeline truyền qua hiện giả định:

- 11 hàng trạng thái kết tinh cho mỗi mẫu shape (nhóm theo `shape_uid` duy nhất)
- 100 bin bước sóng cho mỗi trạng thái kết tinh (các cột `T@...`)
- tối đa 4 điểm điều khiển Q1 được mã hóa thành tensor `4x3`: `(presence, x, y)`
- tái tạo polygon dưới ràng buộc đối xứng C4 để trực quan hóa shape và kiểm tra nhất quán

Repository cũng chứa các nhánh thử nghiệm (`AVIRIS*`, `noise_experiment*`, `archived/`) ngoài luồng huấn luyện truyền qua chính.

## 🧯 Khắc Phục Sự Cố

| Triệu chứng | Nguyên nhân khả dĩ | Cách khắc phục |
|---|---|---|
| `../build/S4: No such file or directory` | Thiếu S4 binary ở đường dẫn tương đối kỳ vọng | Build hoặc link S4 tại `../build/S4`, hoặc cập nhật đường dẫn launcher |
| `No transmission columns found` | CSV thiếu các cột `T@...` | Kiểm tra lại định dạng đầu ra khi merge |
| `Must specify either --data_npz or --csv_file` | Thiếu tham số dữ liệu cho train/eval | Cung cấp rõ ràng một đầu vào |
| `No valid shapes => SHIFT->Q1->UpTo4` | `vertices_str` rỗng/không hợp lệ hoặc bước lọc Q1 loại hết mẫu | Xác thực file shape và đầu ra merge |
| Merge trả về rỗng với `--prefix` | Prefix không khớp file trong `results/` | Kiểm tra chính xác prefix trong tên file rồi chạy lại merge |
| Thiếu checkpoint khi đánh giá | Thiếu file checkpoint `stageA/B/C` | Xác nhận `--model_dir` trỏ đúng thư mục output đầy đủ |

## 🤝 Đóng Góp

Rất hoan nghênh đóng góp, đặc biệt cho tính tái lập, kiểm thử và chất lượng tài liệu.

Quy trình gợi ý:

1. Mở issue nêu phạm vi và hành vi kỳ vọng.
2. Tạo branch tập trung cho một mục tiêu.
3. Gửi pull request kèm lệnh chạy được và kết quả đầu ra.
4. Giữ phạm vi thay đổi gọn trong một workflow khi có thể.

## 📄 Giấy Phép

Hiện chưa có file `LICENSE` ở thư mục gốc repository trong snapshot này. Hãy thêm file giấy phép để xác định điều khoản sử dụng và phân phối lại.

## 📚 Trích Dẫn

Nếu bạn sử dụng repository này hoặc phát triển tiếp từ công trình này, vui lòng trích dẫn:

```bibtex
@article{chen2025inverse,
  title={Inverse Design of Metasurface for Spectral Imaging},
  author={Chen, Rongzhou and Nie, Haitao and Zhu, Shuo and Zhao, Yaping and Wang, Chutian and Lam, Edmund Y},
  journal={arXiv preprint arXiv:2510.21924},
  year={2025}
}
```

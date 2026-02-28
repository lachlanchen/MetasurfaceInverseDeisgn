[English](../README.md) · [العربية](README.ar.md) · [Español](README.es.md) · [Français](README.fr.md) · [日本語](README.ja.md) · [한국어](README.ko.md) · [Tiếng Việt](README.vi.md) · [中文 (简体)](README.zh-Hans.md) · [中文（繁體）](README.zh-Hant.md) · [Deutsch](README.de.md) · [Русский](README.ru.md)


# Thiết kế ngược metasurface cho ảnh phổ

<p align="center">
  <img alt="Status" src="https://img.shields.io/badge/Status-Research%20Prototype-f59e0b?style=for-the-badge">
  <img alt="Python" src="https://img.shields.io/badge/Python-3.9-3776AB?style=for-the-badge&logo=python&logoColor=white">
  <img alt="PyTorch" src="https://img.shields.io/badge/PyTorch-2.x-EE4C2C?style=for-the-badge&logo=pytorch&logoColor=white">
  <img alt="Simulator" src="https://img.shields.io/badge/Simulator-S4%20RCWA-16a34a?style=for-the-badge">
  <img alt="Platform" src="https://img.shields.io/badge/Platform-Linux%20%2B%20Bash-6b7280?style=for-the-badge&logo=gnu-bash&logoColor=white">
  <img alt="ArXiv" src="https://img.shields.io/badge/arXiv-2510.21924-b31b1b?style=for-the-badge&logo=arxiv&logoColor=white">
</p>

Đây là kho nghiên cứu ưu tiên script (trước đây được gọi là `inverse_metasurface`) cho bài toán thiết kế ngược metasurface trong ảnh phổ.

Quy trình cốt lõi kết hợp:
- Mô phỏng RCWA bám sát vật lý (`S4` + Lua)
- Lắp ghép dữ liệu và gắn hình dạng
- Học PyTorch ba giai đoạn (`shape -> spectrum`, `spectrum -> shape`, fine-tuning chuỗi)
- Đánh giá định lượng và định tính, kèm tùy chọn kiểm tra nhất quán neural-vs-S4

> [!IMPORTANT]
> Hành vi chuẩn và các lệnh được giữ nguyên từ script/tài liệu hiện có của dự án. Với các tham chiếu lịch sử trỏ tới file không còn trong snapshot, các tham chiếu đó vẫn được giữ lại kèm ghi chú rõ ràng để đảm bảo tương thích.

## 📑 Mục lục

- [✨ Tổng quan nhanh](#-tổng-quan-nhanh)
- [🌍 Quốc tế hóa (i18n)](#-quốc-tế-hóa-i18n)
- [✨ Tính năng](#-tính-năng)
- [🧭 Quy trình end-to-end](#-quy-trình-end-to-end)
- [🧱 Cấu trúc dự án](#-cấu-trúc-dự-án)
- [🛠️ Điều kiện tiên quyết](#️-điều-kiện-tiên-quyết)
- [🚀 Cài đặt](#-cài-đặt)
- [▶️ Cách sử dụng](#️-cách-sử-dụng)
- [⚙️ Cấu hình](#️-cấu-hình)
- [🧪 Ví dụ](#-ví-dụ)
- [🔬 Bối cảnh nghiên cứu](#-bối-cảnh-nghiên-cứu)
- [🧑‍💻 Ghi chú phát triển](#-ghi-chú-phát-triển)
- [🧯 Khắc phục sự cố](#-khắc-phục-sự-cố)
- [🗺️ Lộ trình](#️-lộ-trình)
- [🤝 Đóng góp](#-đóng-góp)
- [📄 Giấy phép](#-giấy-phép)
- [📚 Trích dẫn](#-trích-dẫn)

## ✨ Tổng quan nhanh

| Mục | Chi tiết |
|---|---|
| 🎯 Nhiệm vụ chính | Suy luận hình học metasurface đối xứng C4 từ phổ truyền qua mục tiêu |
| 🔬 Trình mô phỏng | `../build/S4` được gọi bởi launcher shell và script `.lua` |
| 🧠 Pipeline học | Giai đoạn A `shape -> spectra`, giai đoạn B `spectra -> shape`, giai đoạn C `spectra -> shape -> spectra` |
| 📦 Hợp đồng dữ liệu | CSV đã gộp (`T@...`, metadata, `vertices_str`) -> NPZ nén (`uids`, `spectra`, `shapes`) |
| 🧪 Đánh giá | Chỉ số MSE, trực quan hóa theo giai đoạn, tùy chọn mô phỏng lại S4 mới |
| 🌐 Trạng thái i18n | README đa ngôn ngữ ở mức root + thư mục `i18n/` hiện có |

## 🌍 Quốc tế hóa (i18n)

- Các README đa ngôn ngữ được duy trì ở root repository dưới dạng `README.<lang>.md`.
- Thư mục `i18n/` tồn tại trong snapshot repository này.
- File này giữ một dòng tùy chọn ngôn ngữ duy nhất ở đầu để tránh trùng lặp thanh ngôn ngữ.
- `README.en.md` cũng tồn tại trong repo; `README.md` này vẫn là bản chuẩn cho đợt cập nhật hiện tại.

## ✨ Tính năng

- Luồng thiết kế ngược end-to-end từ đầu ra mô phỏng S4 tới mô hình nghịch đảo đã huấn luyện.
- Tham số hóa đa giác đối xứng C4 và mã hóa điểm Q1 (`4x3`: `presence, x, y`).
- Huấn luyện mô hình ba giai đoạn trong một script (`three_stage_transmittance.py`).
- Công cụ gộp giữ nguyên độ chính xác phổ và gắn vertices theo từng shape.
- Trình đánh giá tùy chọn để so sánh dự đoán đã học với mô phỏng S4 mới.
- Nhiều nhánh khám phá (AVIRIS, SWIR/noise, GSST, biến thể archived/deprecated).

## 🧭 Quy trình end-to-end

1. Sinh đầu ra mô phỏng trong `results/` và file đa giác trong `shapes/`.
2. Gộp các file CSV từ S4 và gắn vertices của shape.
3. Chuẩn hóa tên cột đã gộp để tương thích huấn luyện.
4. Tiền xử lý CSV đã gộp thành tensor NPZ.
5. Huấn luyện mô hình giai đoạn A/B/C.
6. Đánh giá checkpoint và trực quan hóa hành vi.
7. Tùy chọn so sánh phổ của shape dự đoán với các lần chạy S4 mới.

## 🧱 Cấu trúc dự án

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

## 🛠️ Điều kiện tiên quyết

| Phụ thuộc | Ghi chú |
|---|---|
| Linux + Bash | Các launcher nhắm tới thực thi shell |
| Python 3.9 | Khớp với `iccp.yaml` (`python=3.9.18`) |
| Conda | Khuyến nghị để tái lập kết quả |
| S4 binary | Dự kiến tại `../build/S4` |
| CUDA GPU (tùy chọn) | Tăng tốc huấn luyện/đánh giá |

## 🚀 Cài đặt

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

Ghi chú phương án thay thế:

```bash
# Historical README reference (file may be absent in this snapshot)
pip install -r pip_requirements.txt
```

### 3) Xác minh đường dẫn simulator mà script kỳ vọng

```bash
ls -l ../build/S4
```

### 4) (Tùy chọn) cấp quyền thực thi cho launcher

```bash
chmod +x ms.sh ms_final.sh ms_resume.sh ms_resume_allargs.sh ms_resume_random_state.sh ms_resume_random_state_nir.sh
```

## ▶️ Cách sử dụng

### A) Tạo dữ liệu mô phỏng RCWA

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

Launcher hướng resume:

```bash
./ms_resume_allargs.sh \
  -ns 10000 \
  -r 12345 \
  -p myrun \
  -g 80 \
  -bo 0.35 \
  -ro 0.30
```

Ví dụ resume/random-state bổ sung (từ tài liệu lệnh):

```bash
./ms_resume_random_state.sh \
  -p iccp100kG20Ov \
  -r 88888 \
  -g 20 \
  -bo 0.35 \
  -ro 0.3 \
  -ns 100000
```

Ghi chú:
- Launcher chạy `NQ=1..4` song song.
- Script gọi `../build/S4` với `-t 32`.

### B) Gộp đầu ra S4 và gắn vertices của shape

```bash
python merge_s4_data_full.py --prefix myrun
# output: merged_s4_shapes_myrun.csv
```

### C) Chuẩn hóa cột để tương thích huấn luyện

`merge_s4_data_full.py` ghi `folder_key` và `NQ`, trong khi luồng huấn luyện kỳ vọng `prefix` và `nQ`.

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

### E) Huấn luyện giai đoạn A/B/C

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

Lệnh trong README lịch sử (tên script được giữ để tương thích với tài liệu cũ):

```bash
python three_stage_transmittance_evaluation.py \
  --model_dir outputs_three_stage_YYYYMMDD_HHMMSS \
  --data_npz preprocessed_t_data.npz \
  --sample_count 8
```

Ghi chú trạng thái repository: `three_stage_transmittance_evaluation.py` không có trong snapshot này. Hãy dùng `FilterShapeS4_Evaluator_Transmittance.py` cho chức năng đánh giá hiện có.

### G) Tùy chọn kiểm tra nhất quán neural-vs-S4

```bash
python FilterShapeS4_Evaluator_Transmittance.py \
  --npz_file preprocessed_t_data.npz \
  --spec2shape_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageC/spec2shape_stageC.pt \
  --shape2spec_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageA/shape2spec_stageA.pt \
  --n_samples 4
```

## ⚙️ Cấu hình

### S4 launcher (`ms_final.sh`, `ms_resume_allargs.sh`)

| Cờ | Ý nghĩa | Mặc định |
|---|---|---|
| `-ns`, `--numshapes` | Số lượng shape cần tạo | `100000` |
| `-r`, `--seed` | Seed ngẫu nhiên | `88888` |
| `-p`, `--prefix` | Prefix/resume key | `""` |
| `-g`, `--numg` | Tham số basis/grid | `80` |
| `-bo`, `--baseouter` | Độ lệch biên ngoài cơ sở | `0.25` |
| `-ro`, `--randouter` | Độ lệch biên ngoài ngẫu nhiên | `0.20` |

### Huấn luyện (`three_stage_transmittance.py`)

| Cờ | Ý nghĩa | Mặc định |
|---|---|---|
| `--preprocess` | Chạy chế độ tiền xử lý | `False` |
| `--input_folder` | Thư mục chứa CSV đã gộp | `""` |
| `--output_npz` | Đường dẫn NPZ đầu ra | `preprocessed_data.npz` |
| `--data_npz` | Dataset NPZ cho huấn luyện | `""` |
| `--csv_file` | CSV dự phòng nếu không dùng NPZ | `""` |
| `--test` | Chế độ test | `False` |
| `--num_epochs` | Số epoch huấn luyện | `10` |
| `--batch_size` | Kích thước batch | `4096` |

### Cấu hình đánh giá lịch sử (`three_stage_transmittance_evaluation.py`)

| Cờ | Ý nghĩa | Mặc định |
|---|---|---|
| `--model_dir` | Thư mục chứa `stageA/B/C` | bắt buộc |
| `--data_npz` | NPZ đầu vào | `""` |
| `--csv_file` | CSV đầu vào dự phòng | `""` |
| `--output_dir` | Ghi đè thư mục đầu ra | tự động dưới `model_dir` |
| `--sample_count` | Số mẫu trực quan hóa | `4` |
| `--seed` | Seed ngẫu nhiên | `23` |
| `--font_scale` | Tỷ lệ font biểu đồ | `1.0` |
| `--batch_size` | Kích thước batch đánh giá | `32` |
| `--plot_only` | Chỉ vẽ đường cong huấn luyện | `False` |

### Trình đánh giá nhất quán S4 (`FilterShapeS4_Evaluator_Transmittance.py`)

| Cờ | Ý nghĩa | Mặc định |
|---|---|---|
| `--npz_file` | File NPZ đầu vào | `preprocessed_t_data.npz` |
| `--spec2shape_ckpt` | Đường dẫn checkpoint giai đoạn C | `outputs_three_stage_20250322_145925/stageC/spec2shape_stageC.pt` |
| `--shape2spec_ckpt` | Đường dẫn checkpoint giai đoạn A | `outputs_three_stage_20250322_145925/stageA/shape2spec_stageA.pt` |
| `--n_samples` | Số mẫu được đánh giá | `4` |
| `--seed` | Seed ngẫu nhiên | `23` |
| `--max_workers` | Luồng worker S4 | `4` |
| `--out_folder` | Thư mục đầu ra | timestamp tự động |

## 🧪 Ví dụ

### Chạy smoke test

```bash
./ms_final.sh -ns 1000 -r 42 -p smoke -g 40 -bo 0.25 -ro 0.20
python merge_s4_data_full.py --prefix smoke
python -c "import pandas as pd; p='merged_s4_shapes_smoke.csv'; d=pd.read_csv(p).rename(columns={'folder_key':'prefix','NQ':'nQ'}); d.to_csv(p,index=False)"
mkdir -p merged_csvs && mv merged_s4_shapes_smoke.csv merged_csvs/
python three_stage_transmittance.py --preprocess --input_folder merged_csvs --output_npz smoke.npz
python three_stage_transmittance.py --data_npz smoke.npz --num_epochs 5 --batch_size 128
```

### Ví dụ profile chạy (từ `commands.md` / `commands_updated.md`)

```bash
# no overlap, G=40
./ms_resume_allargs.sh -ns 10000 -r 12345 -p fast_without_overlap -g 40 -bo 0.25 -ro 0.2

# overlap, G=80
./ms_resume_allargs.sh -ns 10000 -r 12345 -p overlap -g 80 -bo 0.35 -ro 0.3

# large run, overlap, G=80
./ms_resume_allargs.sh -ns 100000 -r 12345 -p more_basis_overlap -g 80 -bo 0.35 -ro 0.3
```

## 🔬 Bối cảnh nghiên cứu

Thiết lập thiết kế ngược hiện tại học cách khôi phục hình học đối xứng C4 từ truyền qua theo các trạng thái kết tinh. Pipeline truyền qua hiện giả định:

- 11 hàng mức kết tinh cho mỗi mẫu shape (nhóm theo `shape_uid` duy nhất)
- 100 bin bước sóng cho mỗi trạng thái kết tinh (các cột `T@...`)
- Tối đa 4 điểm điều khiển Q1 mã hóa thành tensor `4x3`: `(presence, x, y)`
- Tái tạo đa giác dưới đối xứng C4 cho trực quan hóa shape và kiểm tra nhất quán

Repository cũng chứa các nhánh khám phá (`AVIRIS*`, `noise_experiment*`, `archived/`) ngoài luồng huấn luyện truyền qua chính.

## 🧑‍💻 Ghi chú phát triển

- Đây là repository nghiên cứu thiên về script thay vì Python module đóng gói.
- Script cốt lõi giả định đường dẫn tương đối (đặc biệt `../build/S4`, `results/`, `shapes/`).
- `.gitignore` loại trừ nhiều artifact thí nghiệm được sinh ra (`*.csv`, `*.npz`, `*.pt`, thư mục run).
- Một số file/thư mục trong tài liệu lịch sử hiện không có trong snapshot này; các tham chiếu đó được giữ chủ ý kèm ghi chú để tương thích.
- Có các file sidecar macOS (`._*`) và có thể chỉ là artifact metadata không chức năng.

## 🧯 Khắc phục sự cố

| Triệu chứng | Nguyên nhân khả dĩ | Cách khắc phục |
|---|---|---|
| `../build/S4: No such file or directory` | Thiếu binary S4 ở đường dẫn tương đối được kỳ vọng | Build hoặc link S4 tại `../build/S4`, hoặc cập nhật đường dẫn launcher |
| `No transmission columns found` | CSV thiếu cột `T@...` | Kiểm tra lại định dạng đầu ra khi gộp |
| `Must specify either --data_npz or --csv_file` | Thiếu đối số dữ liệu cho train/eval | Cung cấp rõ một đầu vào |
| `No valid shapes => SHIFT->Q1->UpTo4` | `vertices_str` không hợp lệ/rỗng hoặc lọc Q1 loại hết mẫu | Xác thực file shape và đầu ra gộp |
| Merge output rỗng với `--prefix` | Prefix không khớp file trong `results/` | Kiểm tra chính xác prefix tên file và chạy lại merge |
| Thiếu evaluation checkpoint | Thiếu file checkpoint `stageA/B/C` | Xác nhận `--model_dir` trỏ đúng thư mục output đầy đủ |
| Không tìm thấy `three_stage_transmittance_evaluation.py` | Script được tham chiếu trong tài liệu lịch sử nhưng hiện không có | Dùng `FilterShapeS4_Evaluator_Transmittance.py` hoặc khôi phục script đó từ commit cũ |

## 🗺️ Lộ trình

- Cải thiện khả năng tái lập bằng manifest version dữ liệu rõ ràng và cấu hình chạy đã pin.
- Hợp nhất các điểm vào chuẩn cho nhánh transmittance, AVIRIS và noise.
- Thêm smoke test tự động cho tiền xử lý và một epoch huấn luyện mini.
- Thêm registry thí nghiệm rõ ràng, liên kết thư mục output với chính xác command line đã chạy.
- Mở rộng quy trình đồng bộ README đa ngôn ngữ (file ngôn ngữ ở root và `i18n/`).

## 🤝 Đóng góp

Mọi đóng góp đều được hoan nghênh, đặc biệt cho khả năng tái lập, kiểm thử và chất lượng tài liệu.

Quy trình đề xuất:

1. Mở issue nêu phạm vi và hành vi kỳ vọng.
2. Tạo một branch tập trung.
3. Gửi pull request kèm lệnh và đầu ra có thể chạy lại.
4. Nếu có thể, giữ phạm vi thay đổi trong một workflow.

## 📄 Giấy phép

Hiện chưa có file `LICENSE` ở root repository trong snapshot này. Hãy thêm để xác định điều khoản sử dụng và phân phối lại.

## 📚 Trích dẫn

Nếu bạn sử dụng repository này hoặc phát triển dựa trên công trình này, vui lòng trích dẫn:

```bibtex
@article{chen2025inverse,
  title={Inverse Design of Metasurface for Spectral Imaging},
  author={Chen, Rongzhou and Nie, Haitao and Zhu, Shuo and Zhao, Yaping and Wang, Chutian and Lam, Edmund Y},
  journal={arXiv preprint arXiv:2510.21924},
  year={2025}
}
```

[English](../README.md) · [العربية](README.ar.md) · [Español](README.es.md) · [Français](README.fr.md) · [日本語](README.ja.md) · [한국어](README.ko.md) · [Tiếng Việt](README.vi.md) · [中文 (简体)](README.zh-Hans.md) · [中文（繁體）](README.zh-Hant.md) · [Deutsch](README.de.md) · [Русский](README.ru.md)


[![LazyingArt banner](https://github.com/lachlanchen/lachlanchen/raw/main/figs/banner.png)](https://github.com/lachlanchen/lachlanchen/blob/main/figs/banner.png)

# Thiết kế nghịch đảo Metasurface cho Chụp ảnh quang phổ

<p align="center">
  <img alt="Status" src="https://img.shields.io/badge/Status-Research%20Prototype-f59e0b?style=for-the-badge">
  <img alt="Python" src="https://img.shields.io/badge/Python-3.9-3776AB?style=for-the-badge&logo=python&logoColor=white">
  <img alt="PyTorch" src="https://img.shields.io/badge/PyTorch-2.x-EE4C2C?style=for-the-badge&logo=pytorch&logoColor=white">
  <img alt="Simulator" src="https://img.shields.io/badge/Simulator-S4%20RCWA-16a34a?style=for-the-badge">
  <img alt="Platform" src="https://img.shields.io/badge/Platform-Linux%20%2B%20Bash-6b7280?style=for-the-badge&logo=gnu-bash&logoColor=white">
  <img alt="ArXiv" src="https://img.shields.io/badge/arXiv-2510.21924-b31b1b?style=for-the-badge&logo=arxiv&logoColor=white">
</p>

Kho lưu trữ nghiên cứu hướng script (được nhắc đến lịch sử là `inverse_metasurface`) cho thiết kế nghịch đảo metasurface trong chụp ảnh quang phổ.

Luồng làm việc cốt lõi kết hợp:
- Mô phỏng RCWA dựa trên vật lý (`S4` + Lua)
- Hợp nhất dữ liệu và gán hình dạng
- Học PyTorch ba giai đoạn (`shape -> spectrum`, `spectrum -> shape`, fine-tuning nối tiếp)
- Đánh giá định lượng và định tính, kèm kiểm tra nhất quán neural-vs-S4 nếu cần

> [!IMPORTANT]
> Hành vi chuẩn và các lệnh được giữ nguyên từ các script/tài liệu của dự án hiện tại. Khi tài liệu lịch sử trỏ đến file không còn tồn tại, các tham chiếu đó vẫn được giữ nguyên kèm ghi chú rõ ràng để đảm bảo khả năng tương thích.

## 📑 Contents

- [🌟 Snapshot](#-snapshot)
- [✨ At a Glance](#-at-a-glance)
- [🌍 Internationalization (i18n)](#-internationalization-i18n)
- [✨ Features](#-features)
- [🧭 End-to-End Workflow](#-end-to-end-workflow)
- [🧱 Project Structure](#-project-structure)
- [🛠️ Prerequisites](#️-prerequisites)
- [🚀 Installation](#-installation)
- [▶️ Usage](#️-usage)
- [⚙️ Configuration](#️-configuration)
- [🧪 Examples](#-examples)
- [🔬 Research Context](#-research-context)
- [🧑‍💻 Development Notes](#-development-notes)
- [🧯 Troubleshooting](#-troubleshooting)
- [🗺️ Roadmap](#️-roadmap)
- [🤝 Contribution](#-contribution)
- [📄 License](#-license)
- [📚 Citation](#-citation)

## 🌟 Snapshot

| Focus | Status |
|---|---|
| 🧠 Objective | Tái tạo nghịch đảo hình học metasurface đối xứng C4 từ dữ liệu phổ |
| 🔧 Core stack | S4 RCWA (`Lua`) + huấn luyện PyTorch + tái xác nhận geometry-to-spectrum tùy chọn |
| 🧪 Data pipeline | Ghép CSV/shape attachment → NPZ (`uids`, `spectra`, `shapes`) |
| 🚀 Readiness | Mô hình nguyên mẫu nghiên cứu; script và tài liệu giữ tương thích với tham chiếu lịch sử |

## ✨ At a Glance

| Item | Details |
|---|---|
| 🎯 Main task | Suy diễn hình học metasurface đối xứng C4 từ phổ truyền qua mục tiêu |
| 🔬 Simulator | `../build/S4` được gọi bởi trình khởi chạy shell và các script `.lua` |
| 🧠 Learning pipeline | Giai đoạn A `shape -> spectra`, Giai đoạn B `spectra -> shape`, Giai đoạn C `spectra -> shape -> spectra` |
| 📦 Data contract | CSV đã ghép (`T@...`, metadata, `vertices_str`) -> NPZ nén (`uids`, `spectra`, `shapes`) |
| 🧪 Evaluation | Số liệu MSE, trực quan hóa theo giai đoạn, mô phỏng lại S4 mới nếu cần |
| 🌐 i18n status | File README đa ngôn ngữ ở root + thư mục `i18n/` hiện hữu |

## 🌍 Internationalization (i18n)

- Các README đa ngôn ngữ được duy trì ở thư mục gốc dưới dạng `README.<lang>.md`.
- Thư mục `i18n/` tồn tại trong snapshot repository này.
- File này giữ đúng một hàng tuỳ chọn ngôn ngữ ở đầu để tránh lặp lại các thanh ngôn ngữ.
- `README.en.md` cũng tồn tại trong repo; `README.md` vẫn là bản gốc cho lượt cập nhật này.

## ✨ Features

- Đường dẫn thiết kế nghịch đảo end-to-end từ output mô phỏng S4 đến mô hình inverse đã huấn luyện.
- Tham số hóa đa giác đối xứng C4 và mã hóa điểm Q1 (`4x3`: `presence, x, y`).
- Huấn luyện ba giai đoạn trong cùng một script (`three_stage_transmittance.py`).
- Công cụ ghép dữ liệu giữ độ chính xác phổ và gắn đỉnh theo từng shape.
- Bộ đánh giá tùy chọn so sánh dự đoán học với mô phỏng S4 mới.
- Các nhánh thử nghiệm mở rộng (AVIRIS, SWIR/noise, GSST, các biến thể đã lưu trữ/đã lỗi thời).

## 🧭 End-to-End Workflow

1. Tạo đầu ra mô phỏng trong `results/` và file đa giác trong `shapes/`.
2. Ghép file CSV của S4 và gắn đỉnh hình dạng.
3. Chuẩn hóa tên cột đã ghép để tương thích huấn luyện.
4. Tiền xử lý file CSV đã ghép thành tensor NPZ.
5. Huấn luyện mô hình giai đoạn A/B/C.
6. Đánh giá checkpoint và trực quan hóa hành vi.
7. Tùy chọn so sánh spectra của shape dự đoán với các lần chạy S4 mới.

## 🧱 Project Structure

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

## 🛠️ Prerequisites

| Dependency | Notes |
|---|---|
| Linux + Bash | Các script launcher nhắm tới thực thi trên shell |
| Python 3.9 | Khớp với `iccp.yaml` (`python=3.9.18`) |
| Conda | Được khuyến nghị để tái lập |
| S4 binary | Dự kiến nằm tại `../build/S4` |
| CUDA GPU (optional) | Tăng tốc huấn luyện/đánh giá |

## 🚀 Installation

### 1) Clone and enter

```bash
git clone <your-repo-url> inverse_metasurface
cd inverse_metasurface
```

### 2) Tạo môi trường (khuyến nghị)

```bash
conda env create -f iccp.yaml
conda activate iccp
```

Alternative note:

```bash
# Tham chiếu trong README lịch sử (có thể thiếu file trong snapshot này)
pip install -r pip_requirements.txt
```

### 3) Kiểm tra đường dẫn mô phỏng mà script kỳ vọng

```bash
ls -l ../build/S4
```

### 4) (Tùy chọn) cấp quyền thực thi cho launcher

```bash
chmod +x ms.sh ms_final.sh ms_resume.sh ms_resume_allargs.sh ms_resume_random_state.sh ms_resume_random_state_nir.sh
```

## ▶️ Usage

### A) Tạo dữ liệu mô phỏng RCWA

Simple launcher:

```bash
./ms.sh -ns 10000 -r 12345
```

Parameterized launcher:

```bash
./ms_final.sh \
  -ns 10000 \
  -r 12345 \
  -p myrun \
  -g 80 \
  -bo 0.35 \
  -ro 0.30
```

Resume-oriented launcher:

```bash
./ms_resume_allargs.sh \
  -ns 10000 \
  -r 12345 \
  -p myrun \
  -g 80 \
  -bo 0.35 \
  -ro 0.30
```

Ví dụ resume/random-state bổ sung (từ command docs):

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
- Các script gọi `../build/S4` với `-t 32`.

### B) Ghép output S4 và gắn đỉnh hình dạng

```bash
python merge_s4_data_full.py --prefix myrun
# output: merged_s4_shapes_myrun.csv
```

### C) Chuẩn hóa cột cho khả năng tương thích huấn luyện

`merge_s4_data_full.py` ghi `folder_key` và `NQ`, trong khi đường dẫn huấn luyện kỳ vọng `prefix` và `nQ`.

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

Outputs are written to:
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageA`
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageB`
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageC`

### F) Đánh giá mô hình đã huấn luyện

Historical README command (script name retained for compatibility with prior docs):

```bash
python three_stage_transmittance_evaluation.py \
  --model_dir outputs_three_stage_YYYYMMDD_HHMMSS \
  --data_npz preprocessed_t_data.npz \
  --sample_count 8
```

Ghi chú về trạng thái repo: `three_stage_transmittance_evaluation.py` không có trong snapshot này. Sử dụng `FilterShapeS4_Evaluator_Transmittance.py` cho chức năng đánh giá hiện có.

### G) Kiểm tra nhất quán neural-vs-S4 tùy chọn

```bash
python FilterShapeS4_Evaluator_Transmittance.py \
  --npz_file preprocessed_t_data.npz \
  --spec2shape_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageC/spec2shape_stageC.pt \
  --shape2spec_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageA/shape2spec_stageA.pt \
  --n_samples 4
```

## ⚙️ Configuration

### S4 launchers (`ms_final.sh`, `ms_resume_allargs.sh`)

| Flag | Meaning | Default |
|---|---|---|
| `-ns`, `--numshapes` | Số lượng shapes cần tạo | `100000` |
| `-r`, `--seed` | Seed ngẫu nhiên | `88888` |
| `-p`, `--prefix` | Prefix/key tiếp tục thực thi | `""` |
| `-g`, `--numg` | Tham số basis/grid | `80` |
| `-bo`, `--baseouter` | Độ lệch biên ngoài cơ sở | `0.25` |
| `-ro`, `--randouter` | Độ lệch biên ngoài ngẫu nhiên | `0.20` |

### Training (`three_stage_transmittance.py`)

| Flag | Meaning | Default |
|---|---|---|
| `--preprocess` | Chạy chế độ tiền xử lý | `False` |
| `--input_folder` | Thư mục chứa file CSV đã ghép | `""` |
| `--output_npz` | Đường dẫn output NPZ | `preprocessed_data.npz` |
| `--data_npz` | Bộ dữ liệu NPZ cho huấn luyện | `""` |
| `--csv_file` | CSV dự phòng khi không dùng NPZ | `""` |
| `--test` | Chế độ test | `False` |
| `--num_epochs` | Số epoch huấn luyện | `10` |
| `--batch_size` | Kích thước batch | `4096` |

### Historical evaluation config (`three_stage_transmittance_evaluation.py`)

| Flag | Meaning | Default |
|---|---|---|
| `--model_dir` | Thư mục chứa `stageA/B/C` | required |
| `--data_npz` | Input NPZ | `""` |
| `--csv_file` | CSV input fallback | `""` |
| `--output_dir` | Ghi đè thư mục đầu ra | auto dưới `model_dir` |
| `--sample_count` | Số mẫu hiển thị | `4` |
| `--seed` | Random seed | `23` |
| `--font_scale` | Tỷ lệ phông chữ | `1.0` |
| `--batch_size` | Batch kích thước đánh giá | `32` |
| `--plot_only` | Chỉ vẽ đường cong huấn luyện | `False` |

### S4 consistency evaluator (`FilterShapeS4_Evaluator_Transmittance.py`)

| Flag | Meaning | Default |
|---|---|---|
| `--npz_file` | Input NPZ file | `preprocessed_t_data.npz` |
| `--spec2shape_ckpt` | Đường dẫn checkpoint giai đoạn C | `outputs_three_stage_20250322_145925/stageC/spec2shape_stageC.pt` |
| `--shape2spec_ckpt` | Đường dẫn checkpoint giai đoạn A | `outputs_three_stage_20250322_145925/stageA/shape2spec_stageA.pt` |
| `--n_samples` | Số mẫu được đánh giá | `4` |
| `--seed` | Random seed | `23` |
| `--max_workers` | Số luồng làm việc S4 | `4` |
| `--out_folder` | Thư mục đầu ra | tự động timestamp |

## 🧪 Examples

### Smoke run

```bash
./ms_final.sh -ns 1000 -r 42 -p smoke -g 40 -bo 0.25 -ro 0.20
python merge_s4_data_full.py --prefix smoke
python -c "import pandas as pd; p='merged_s4_shapes_smoke.csv'; d=pd.read_csv(p).rename(columns={'folder_key':'prefix','NQ':'nQ'}); d.to_csv(p,index=False)"
mkdir -p merged_csvs && mv merged_s4_shapes_smoke.csv merged_csvs/
python three_stage_transmittance.py --preprocess --input_folder merged_csvs --output_npz smoke.npz
python three_stage_transmittance.py --data_npz smoke.npz --num_epochs 5 --batch_size 128
```

### Run-profile examples (from `commands.md` / `commands_updated.md`)

```bash
# no overlap, G=40
./ms_resume_allargs.sh -ns 10000 -r 12345 -p fast_without_overlap -g 40 -bo 0.25 -ro 0.2

# overlap, G=80
./ms_resume_allargs.sh -ns 10000 -r 12345 -p overlap -g 80 -bo 0.35 -ro 0.3

# large run, overlap, G=80
./ms_resume_allargs.sh -ns 100000 -r 12345 -p more_basis_overlap -g 80 -bo 0.35 -ro 0.3
```

## 🔬 Research Context

Thiết lập thiết kế nghịch đảo hiện tại học cách khôi phục hình học đối xứng C4 từ phổ truyền qua trên các trạng thái kết tinh. Pipeline truyền qua hiện tại giả sử:

- 11 hàng kết tinh cho mỗi mẫu shape (gom nhóm theo `shape_uid` duy nhất)
- 100 cột bước sóng cho mỗi trạng thái kết tinh (`T@...`)
- Tối đa 4 điểm điều khiển Q1 được mã hóa dưới dạng tensor `4x3`: `(presence, x, y)`
- Tái tạo đa giác theo đối xứng C4 cho trực quan hóa hình dạng và kiểm tra nhất quán

Repository cũng chứa các nhánh thử nghiệm (`AVIRIS*`, `noise_experiment*`, `archived/`) ngoài đường dẫn huấn luyện truyền qua chính.

## 🧑‍💻 Development Notes

- Đây là repository nghiên cứu theo hướng script thay vì module Python đóng gói.
- Các script cốt lõi giả định đường dẫn tương đối (đặc biệt `../build/S4`, `results/`, `shapes/`).
- `.gitignore` loại trừ nhiều artifact thí nghiệm được tạo (`*.csv`, `*.npz`, `*.pt`, thư mục run).
- Một số file/thư mục trong tài liệu lịch sử hiện không có trong snapshot này; các tham chiếu này vẫn được giữ nguyên cùng ghi chú nhằm đảm bảo tương thích.
- File sidecar macOS (`._*`) có mặt và có thể chỉ là metadata không hoạt động.

## 🧯 Troubleshooting

| Symptom | Likely Cause | Fix |
|---|---|---|
| `../build/S4: No such file or directory` | Thiếu binary S4 tại đường dẫn tương đối mong đợi | Build hoặc link S4 tại `../build/S4`, hoặc cập nhật đường dẫn launcher |
| `No transmission columns found` | CSV thiếu cột `T@...` | Kiểm tra lại định dạng output sau ghép |
| `Must specify either --data_npz or --csv_file` | Thiếu tham số dữ liệu huấn luyện/đánh giá | Cung cấp rõ ràng một trong hai đầu vào |
| `No valid shapes => SHIFT->Q1->UpTo4` | `vertices_str` không hợp lệ/rỗng hoặc lọc Q1 loại bỏ toàn bộ mẫu | Kiểm tra file shape và kết quả merge |
| Empty merge output for `--prefix` | `--prefix` không khớp file trong `results/` | Kiểm tra chính xác tiền tố tên và chạy lại merge |
| Evaluation checkpoint missing | Thiếu file checkpoint `stageA/B/C` | Xác nhận `--model_dir` trỏ đúng thư mục output hoàn chỉnh |
| `three_stage_transmittance_evaluation.py` not found | Script được nhắc trong tài liệu lịch sử nhưng hiện không có | Dùng `FilterShapeS4_Evaluator_Transmittance.py` hoặc phục hồi script đó từ commit trước |

## 🗺️ Roadmap

- Cải thiện khả năng tái lập bằng manifest phiên bản dữ liệu rõ ràng và pinned run config.
- Hợp nhất các điểm vào lối vào chuẩn cho nhánh truyền qua, AVIRIS và noise.
- Thêm smoke test tự động cho tiền xử lý và một epoch huấn luyện mini.
- Rõ ràng hơn registry thí nghiệm, liên kết thư mục output với lệnh chạy tương ứng.
- Mở rộng quy trình đồng bộ hóa README đa ngôn ngữ (file root theo ngôn ngữ và `i18n/`).

## 🤝 Contribution

Đóng góp được hoan nghênh, đặc biệt cho tái lập, thử nghiệm và chất lượng tài liệu.

Quy trình gợi ý:

1. Mở issue với phạm vi và hành vi mong đợi.
2. Tạo branch tập trung.
3. Tạo pull request kèm lệnh có thể chạy và đầu ra minh chứng.
4. Giữ thay đổi tập trung vào một workflow khi có thể.

## ❤️ Support

| Donate | PayPal | Stripe |
| --- | --- | --- |
| [![Donate](https://camo.githubusercontent.com/24a4914f0b42c6f435f9e101621f1e52535b02c225764b2f6cc99416926004b7/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f446f6e6174652d4c617a79696e674172742d3045413545393f7374796c653d666f722d7468652d6261646765266c6f676f3d6b6f2d6669266c6f676f436f6c6f723d7768697465)](https://chat.lazying.art/donate) | [![PayPal](https://camo.githubusercontent.com/d0f57e8b016517a4b06961b24d0ca87d62fdba16e18bbdb6aba28e978dc0ea21/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f50617950616c2d526f6e677a686f754368656e2d3030343537433f7374796c653d666f722d7468652d6261646765266c6f676f3d70617970616c266c6f676f436f6c6f723d7768697465)](https://paypal.me/RongzhouChen) | [![Stripe](https://camo.githubusercontent.com/1152dfe04b6943afe3a8d2953676749603fb9f95e24088c92c97a01a897b4942/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f5374726970652d446f6e6174652d3633354246463f7374796c653d666f722d7468652d6261646765266c6f676f3d737472697065266c6f676f436f6c6f723d7768697465)](https://buy.stripe.com/aFadR8gIaflgfQV6T4fw400) |

## 📄 License

No `LICENSE` file is currently present at repository root in this snapshot. Add one to define usage and redistribution terms.

## 📚 Citation

If you use this repository or build on this work, please cite:

```bibtex
@article{chen2025inverse,
  title={Inverse Design of Metasurface for Spectral Imaging},
  author={Chen, Rongzhou and Nie, Haitao and Zhu, Shuo and Zhao, Yaping and Wang, Chutian and Lam, Edmund Y},
  journal={arXiv preprint arXiv:2510.21924},
  year={2025}
}
```

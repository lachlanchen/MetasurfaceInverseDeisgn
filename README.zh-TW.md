English | 中文繁體 | 中文简体 | 日本語 | 한국어 | Tiếng Việt | العربية | Français | Español | Deutsch | Русский

# inverse_metasurface

<div align="center">

[![Status](https://img.shields.io/badge/status-experimental-orange)](#-roadmap)
[![Python](https://img.shields.io/badge/python-3.9-blue)](#-prerequisites)
[![Platform](https://img.shields.io/badge/platform-linux-lightgrey)](#-prerequisites)
[![S4](https://img.shields.io/badge/S4-required-success)](#-prerequisites)
[![License](https://img.shields.io/badge/license-unset-lightgrey)](#-license)

</div>

這是一個以 **S4/RCWA 模擬**與**多階段神經模型**為核心，用於**逆向超表面設計（inverse metasurface design）**的研究工作區。

支援內容：
- 針對 C4 對稱多邊形超表面的高吞吐量 S4 資料生成。
- 資料合併與前處理，輸出可直接訓練的 NPZ。
- 三階段學習流程：**shape -> spectrum**、**spectrum -> shape**、**chain-tuned spectrum -> shape -> spectrum**。
- 評估流程，以及可選的 **neural-vs-S4** 直接比較。

## 🌟 At A Glance

| 區塊 | 此儲存庫提供內容 |
|---|---|
| 物理模擬 | Bash + Lua 腳本，可在 `nQ=1..4` 上平行執行 S4 |
| 資料集工具 | 合併腳本，附加多邊形頂點與 pivot spectra |
| ML pipeline | `three_stage_transmittance.py`，含 preprocess + train 模式 |
| 評估 | `three_stage_transmittance_evaluation.py`，含指標與圖表輸出 |
| 研究分支 | AVIRIS/hyperspectral 與 noise/compression 實驗 |

## 🧠 Research Context

此儲存庫聚焦於光子超表面的逆向設計：根據目標光譜推回幾何形狀（以及反向任務）。

基線流程採用的核心假設：
- 透過 Q1 點參數化實現 C4 對稱。
- 11 個 crystallization 狀態（`c` in `[0.0, 1.0]`）。
- 每個樣本光譜儲存為 `11 x 100` 的 transmission 列。
- 形狀以最多 4 個 Q1 點表示（`4 x 3` tensor：presence, x, y）。

三階段訓練邏輯：
1. **Stage A**: shape -> spectrum (`shape2spec_stageA.pt`)
2. **Stage B**: spectrum -> shape (`spec2shape_stageB.pt`)
3. **Stage C**: spectrum -> shape -> spectrum chain fine-tuning (`spec2shape_stageC.pt`)

## 🗂 Project Structure

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

## ✅ Prerequisites

| 需求 | 說明 |
|---|---|
| OS | Linux（腳本預設使用 Bash + Linux 路徑） |
| Python | 3.9（依 `iccp.yaml`） |
| Env manager | 建議使用 Conda |
| S4 binary | 預期位於儲存庫根目錄相對路徑 `../build/S4` |
| GPU（可選） | CUDA 可加速訓練與評估 |

## ⚙️ Installation

### 1) Clone and enter

```bash
git clone <your-repo-url> inverse_metasurface
cd inverse_metasurface
```

### 2) Create environment

```bash
conda env create -f iccp.yaml
conda activate iccp
```

### 3) Verify S4 path

```bash
ls -l ../build/S4
```

若此路徑不存在，請將 S4 建置/放置到該位置，或調整腳本中的路徑設定。

## 🚀 Quick Start (End-to-End)

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

## 🧪 Usage Details

### A) S4 generation

最小化的 seeded run：

```bash
./ms.sh -ns 10000 -r 88888
```

參數化執行：

```bash
./ms_final.sh \
  -ns 100000 \
  -r 88888 \
  -p iccpOv100kG80 \
  -g 80 \
  -bo 0.35 \
  -ro 0.30
```

續跑模式：

```bash
./ms_resume_allargs.sh \
  -ns 100000 \
  -r 88888 \
  -p iccpOv100kG80 \
  -g 80 \
  -bo 0.35 \
  -ro 0.30
```

### B) Merge raw S4 CSVs

```bash
python merge.py --prefix 20250123_155420
```

替代工具：

```bash
python merge_s4_data_full.py --prefix 20250123_155420
```

### C) Preprocess merged CSVs -> NPZ

```bash
python three_stage_transmittance.py --preprocess \
  --input_folder merged_csvs \
  --output_npz preprocessed_t_data.npz
```

### D) Train

```bash
python three_stage_transmittance.py \
  --data_npz preprocessed_t_data.npz \
  --num_epochs 100 \
  --batch_size 1024
```

訓練輸出會儲存於：
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageA`
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageB`
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageC`

### E) Evaluate

```bash
python three_stage_transmittance_evaluation.py \
  --model_dir outputs_three_stage_YYYYMMDD_HHMMSS \
  --data_npz preprocessed_t_data.npz \
  --sample_count 8
```

會產生：
- `evaluation_metrics.csv`
- `metrics_summary.csv`
- 各 stage 視覺化圖（`.png`, `.pdf`）
- 訓練曲線圖

### F) Neural vs direct S4 comparison (optional)

```bash
python FilterShapeS4_Evaluator_Transmittance.py \
  --npz_file preprocessed_t_data.npz \
  --spec2shape_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageC/spec2shape_stageC.pt \
  --shape2spec_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageA/shape2spec_stageA.pt \
  --n_samples 4
```

## 🔧 Key CLI Options

### S4 runner flags (`ms_final.sh`, `ms_resume_allargs.sh`)

| 旗標 | 含義 | 預設值 |
|---|---|---|
| `-ns`, `--numshapes` | 形狀數量 | `100000` |
| `-r`, `--seed` | 隨機種子 | `88888` |
| `-p`, `--prefix` | 執行前綴 / 續跑 key | empty |
| `-g`, `--numg` | 幾何/基底參數 | `80` |
| `-bo`, `--baseouter` | 外輪廓基礎偏移 | `0.25` |
| `-ro`, `--randouter` | 外輪廓隨機偏移 | `0.20` |

### Training flags (`three_stage_transmittance.py`)

| 旗標 | 含義 | 預設值 |
|---|---|---|
| `--preprocess` | 啟用前處理模式 | off |
| `--input_folder` | 輸入 CSV 資料夾 | empty |
| `--output_npz` | 輸出 NPZ 路徑 | `preprocessed_data.npz` |
| `--data_npz` | 用於訓練/評估的 NPZ | empty |
| `--csv_file` | 直接使用的 CSV | empty |
| `--num_epochs` | 每個 stage 的 epoch 數 | `10` |
| `--batch_size` | 批次大小 | `4096` |
| `--test` | 測試模式預留旗標 | off |

### Evaluation flags (`three_stage_transmittance_evaluation.py`)

| 旗標 | 含義 | 預設值 |
|---|---|---|
| `--model_dir` | 已訓練 run 的根目錄 | required |
| `--data_npz` | 評估用 NPZ | empty |
| `--csv_file` | 評估用 CSV | empty |
| `--output_dir` | 輸出資料夾 | `model_dir/evaluation_<timestamp>` |
| `--sample_count` | 視覺化樣本數 | `4` |
| `--seed` | 取樣種子 | `23` |
| `--font_scale` | 圖表字體比例 | `1.0` |
| `--batch_size` | 評估批次大小 | `32` |
| `--plot_only` | 僅繪製曲線 | off |

## 🧭 Troubleshooting

| 症狀 | 常見原因 | 修正方式 |
|---|---|---|
| `../build/S4: No such file or directory` | S4 binary 路徑不一致 | 將 S4 建置/放到 `../build/S4`，或修補腳本路徑 |
| `Must specify either --data_npz or --csv_file` | 缺少資料集來源 | 明確傳入其中一個旗標 |
| `No matching CSVs found in 'results/'` | prefix 不匹配 | 確認 prefix 與輸出命名 |
| 前處理後資料筆數極少或為 0 | 缺少/無效的 `T@...` 或 `vertices_str` 列 | 檢查 merged CSV schema 與每個 shape 的列分組 |
| CUDA OOM | batch 過大 | 調小 `--batch_size`（例如 `1024 -> 256`） |

## 🧱 Development Notes

- 這是偏重實驗的儲存庫；許多生成產物刻意不納入版控。
- 針對相近任務存在多個腳本變體（`merge_*`, `aviris_*`, `noise_experiment_*`）。
- 本 README 文件化的路徑是基線 inverse-metasurface workflow。
- 訓練腳本會設定隨機種子（`42`），但嚴格可重現仍取決於硬體與後端行為。
- 目前尚未建立統一 CI 與完整自動化測試套件。

## 🛣 Roadmap

- 新增精簡且標準的資料集，供快速 smoke test。
- 整併重複的 pipeline 腳本。
- 新增 merge/preprocess 完整性與 checkpoint 可載入性檢查。
- 建立 lint + smoke train/eval 的 CI。
- 更明確文件化 S4 的建置/版本鎖定方式。

## 🤝 Contribution

1. 建立 feature branch。
2. 保持 PR 範圍精準（每個 PR 聚焦一個 pipeline/experiment 議題）。
3. 附上可直接執行的完整命令與預期輸出路徑。
4. 除非必要，避免提交大型生成產物。
5. 補充可重現性說明（seed、data source、checkpoint path）。

## 📄 License

此儲存庫目前尚未提供 `LICENSE` 檔案。

在新增授權檔案之前，重用與再散佈條款應視為**未定義**。

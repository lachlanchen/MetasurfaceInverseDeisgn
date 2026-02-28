[English](../README.md) · [العربية](README.ar.md) · [Español](README.es.md) · [Français](README.fr.md) · [日本語](README.ja.md) · [한국어](README.ko.md) · [Tiếng Việt](README.vi.md) · [中文 (简体)](README.zh-Hans.md) · [中文（繁體）](README.zh-Hant.md) · [Deutsch](README.de.md) · [Русский](README.ru.md)


# 光譜成像之超表面反向設計

<p align="center">
  <img alt="Status" src="https://img.shields.io/badge/Status-Research%20Prototype-f59e0b?style=for-the-badge">
  <img alt="Python" src="https://img.shields.io/badge/Python-3.9-3776AB?style=for-the-badge&logo=python&logoColor=white">
  <img alt="PyTorch" src="https://img.shields.io/badge/PyTorch-2.x-EE4C2C?style=for-the-badge&logo=pytorch&logoColor=white">
  <img alt="Simulator" src="https://img.shields.io/badge/Simulator-S4%20RCWA-16a34a?style=for-the-badge">
  <img alt="Platform" src="https://img.shields.io/badge/Platform-Linux%20%2B%20Bash-6b7280?style=for-the-badge&logo=gnu-bash&logoColor=white">
  <img alt="ArXiv" src="https://img.shields.io/badge/arXiv-2510.21924-b31b1b?style=for-the-badge&logo=arxiv&logoColor=white">
</p>

這是一個以腳本為核心的研究型儲存庫（歷史上稱為 `inverse_metasurface`），用於光譜成像中的超表面反向設計。

核心流程結合了：
- 具物理基礎的 RCWA 模擬（`S4` + Lua）
- 資料彙整與形狀附加
- 三階段 PyTorch 學習（`shape -> spectrum`、`spectrum -> shape`、串接微調）
- 定量與定性評估，並可選擇進行 neural-vs-S4 一致性檢查

> [!IMPORTANT]
> 專案腳本與文件中的既有行為與命令均已保留。若歷史參考指向目前不存在的檔案，會為相容性而刻意保留，並附上明確說明。

## 📑 目錄

- [✨ 快速總覽](#-快速總覽)
- [🌍 國際化（i18n）](#-國際化i18n)
- [✨ 功能特色](#-功能特色)
- [🧭 端到端工作流程](#-端到端工作流程)
- [🧱 專案結構](#-專案結構)
- [🛠️ 先決條件](#️-先決條件)
- [🚀 安裝](#-安裝)
- [▶️ 使用方式](#️-使用方式)
- [⚙️ 設定](#️-設定)
- [🧪 範例](#-範例)
- [🔬 研究脈絡](#-研究脈絡)
- [🧑‍💻 開發備註](#-開發備註)
- [🧯 疑難排解](#-疑難排解)
- [🗺️ 路線圖](#️-路線圖)
- [🤝 貢獻](#-貢獻)
- [📄 授權](#-授權)
- [📚 引用](#-引用)

## ✨ 快速總覽

| 項目 | 說明 |
|---|---|
| 🎯 主要任務 | 由目標透射光譜推斷具 C4 對稱的超表面幾何 |
| 🔬 模擬器 | 由 shell 啟動腳本與 `.lua` 腳本呼叫的 `../build/S4` |
| 🧠 學習流程 | Stage A `shape -> spectra`、Stage B `spectra -> shape`、Stage C `spectra -> shape -> spectra` |
| 📦 資料契約 | 合併 CSV（`T@...`、metadata、`vertices_str`）-> 壓縮 NPZ（`uids`、`spectra`、`shapes`） |
| 🧪 評估 | MSE 指標、各階段視覺化、可選的新 S4 重模擬 |
| 🌐 i18n 狀態 | 根目錄多語 README 檔案 + 既有 `i18n/` 目錄 |

## 🌍 國際化（i18n）

- 多語 README 維護於儲存庫根目錄，命名為 `README.<lang>.md`。
- 此儲存庫快照中已存在 `i18n/` 目錄。
- 本檔案在頂部維持單一語言選項列，避免重複語言導覽列。
- 儲存庫中亦有 `README.en.md`；本次更新仍以 `README.md` 作為正典基準。

## ✨ 功能特色

- 從 S4 模擬輸出到反向模型訓練的端到端反向設計路徑。
- C4 對稱多邊形參數化與 Q1 點編碼（`4x3`：`presence, x, y`）。
- 在單一腳本中進行三階段模型訓練（`three_stage_transmittance.py`）。
- 合併工具可保留光譜精度並附加各形狀頂點。
- 可選評估器可將學習預測與新 S4 模擬比較。
- 豐富的探索分支（AVIRIS、SWIR/noise、GSST、archived/deprecated 變體）。

## 🧭 端到端工作流程

1. 在 `results/` 產生模擬輸出，並在 `shapes/` 產生多邊形檔案。
2. 合併 S4 CSV 檔並附加形狀頂點。
3. 正規化合併後欄位名稱，以相容訓練流程。
4. 將合併 CSV 預處理為 NPZ 張量。
5. 訓練 Stage A/B/C 模型。
6. 評估 checkpoint 並進行行為視覺化。
7. 可選地將預測形狀光譜與新的 S4 執行結果比較。

## 🧱 專案結構

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

## 🛠️ 先決條件

| 相依項目 | 說明 |
|---|---|
| Linux + Bash | 啟動腳本以 shell 執行為目標 |
| Python 3.9 | 與 `iccp.yaml`（`python=3.9.18`）一致 |
| Conda | 建議用於可重現性 |
| S4 binary | 預期位於 `../build/S4` |
| CUDA GPU（選用） | 可加速訓練/評估 |

## 🚀 安裝

### 1) 複製並進入目錄

```bash
git clone <your-repo-url> inverse_metasurface
cd inverse_metasurface
```

### 2) 建立環境（建議）

```bash
conda env create -f iccp.yaml
conda activate iccp
```

替代說明：

```bash
# 歷史 README 參考（此快照中檔案可能不存在）
pip install -r pip_requirements.txt
```

### 3) 驗證腳本預期的模擬器路徑

```bash
ls -l ../build/S4
```

### 4)（選用）讓啟動腳本具可執行權限

```bash
chmod +x ms.sh ms_final.sh ms_resume.sh ms_resume_allargs.sh ms_resume_random_state.sh ms_resume_random_state_nir.sh
```

## ▶️ 使用方式

### A) 產生 RCWA 模擬資料

簡易啟動：

```bash
./ms.sh -ns 10000 -r 12345
```

參數化啟動：

```bash
./ms_final.sh \
  -ns 10000 \
  -r 12345 \
  -p myrun \
  -g 80 \
  -bo 0.35 \
  -ro 0.30
```

續跑導向啟動：

```bash
./ms_resume_allargs.sh \
  -ns 10000 \
  -r 12345 \
  -p myrun \
  -g 80 \
  -bo 0.35 \
  -ro 0.30
```

其他續跑/隨機狀態範例（來自命令文件）：

```bash
./ms_resume_random_state.sh \
  -p iccp100kG20Ov \
  -r 88888 \
  -g 20 \
  -bo 0.35 \
  -ro 0.3 \
  -ns 100000
```

備註：
- 啟動腳本會並行執行 `NQ=1..4`。
- 腳本會以 `-t 32` 呼叫 `../build/S4`。

### B) 合併 S4 輸出並附加形狀頂點

```bash
python merge_s4_data_full.py --prefix myrun
# output: merged_s4_shapes_myrun.csv
```

### C) 為訓練相容性正規化欄位

`merge_s4_data_full.py` 會輸出 `folder_key` 與 `NQ`，而訓練流程預期為 `prefix` 與 `nQ`。

```bash
python -c "import pandas as pd; p='merged_s4_shapes_myrun.csv'; df=pd.read_csv(p); df=df.rename(columns={'folder_key':'prefix','NQ':'nQ'}); df.to_csv(p,index=False)"
```

### D) 預處理 CSV -> NPZ

```bash
mkdir -p merged_csvs
mv merged_s4_shapes_myrun.csv merged_csvs/

python three_stage_transmittance.py \
  --preprocess \
  --input_folder merged_csvs \
  --output_npz preprocessed_t_data.npz
```

### E) 訓練 Stage A/B/C

```bash
python three_stage_transmittance.py \
  --data_npz preprocessed_t_data.npz \
  --num_epochs 100 \
  --batch_size 1024
```

輸出會寫入：
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageA`
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageB`
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageC`

### F) 評估已訓練模型

歷史 README 命令（為相容舊版文件而保留腳本名稱）：

```bash
python three_stage_transmittance_evaluation.py \
  --model_dir outputs_three_stage_YYYYMMDD_HHMMSS \
  --data_npz preprocessed_t_data.npz \
  --sample_count 8
```

儲存庫狀態說明：此快照中不存在 `three_stage_transmittance_evaluation.py`。可使用 `FilterShapeS4_Evaluator_Transmittance.py` 取得可用的評估功能。

### G)（選用）neural-vs-S4 一致性檢查

```bash
python FilterShapeS4_Evaluator_Transmittance.py \
  --npz_file preprocessed_t_data.npz \
  --spec2shape_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageC/spec2shape_stageC.pt \
  --shape2spec_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageA/shape2spec_stageA.pt \
  --n_samples 4
```

## ⚙️ 設定

### S4 啟動腳本（`ms_final.sh`、`ms_resume_allargs.sh`）

| Flag | 含義 | 預設值 |
|---|---|---|
| `-ns`, `--numshapes` | 要產生的形狀數量 | `100000` |
| `-r`, `--seed` | 隨機種子 | `88888` |
| `-p`, `--prefix` | 前綴/續跑鍵值 | `""` |
| `-g`, `--numg` | 基底/網格參數 | `80` |
| `-bo`, `--baseouter` | 外邊界基準偏移 | `0.25` |
| `-ro`, `--randouter` | 外邊界隨機偏移 | `0.20` |

### 訓練（`three_stage_transmittance.py`）

| Flag | 含義 | 預設值 |
|---|---|---|
| `--preprocess` | 執行預處理模式 | `False` |
| `--input_folder` | 包含合併 CSV 的資料夾 | `""` |
| `--output_npz` | 輸出 NPZ 路徑 | `preprocessed_data.npz` |
| `--data_npz` | 訓練所用 NPZ 資料集 | `""` |
| `--csv_file` | 不使用 NPZ 時的 CSV 備援 | `""` |
| `--test` | 測試模式 | `False` |
| `--num_epochs` | 訓練 epoch 數 | `10` |
| `--batch_size` | 批次大小 | `4096` |

### 歷史評估設定（`three_stage_transmittance_evaluation.py`）

| Flag | 含義 | 預設值 |
|---|---|---|
| `--model_dir` | 包含 `stageA/B/C` 的目錄 | required |
| `--data_npz` | NPZ 輸入 | `""` |
| `--csv_file` | CSV 輸入備援 | `""` |
| `--output_dir` | 輸出目錄覆寫 | `model_dir` 下自動建立 |
| `--sample_count` | 視覺化樣本數 | `4` |
| `--seed` | 隨機種子 | `23` |
| `--font_scale` | 繪圖字體縮放 | `1.0` |
| `--batch_size` | 評估批次大小 | `32` |
| `--plot_only` | 僅繪製訓練曲線 | `False` |

### S4 一致性評估器（`FilterShapeS4_Evaluator_Transmittance.py`）

| Flag | 含義 | 預設值 |
|---|---|---|
| `--npz_file` | 輸入 NPZ 檔案 | `preprocessed_t_data.npz` |
| `--spec2shape_ckpt` | Stage C checkpoint 路徑 | `outputs_three_stage_20250322_145925/stageC/spec2shape_stageC.pt` |
| `--shape2spec_ckpt` | Stage A checkpoint 路徑 | `outputs_three_stage_20250322_145925/stageA/shape2spec_stageA.pt` |
| `--n_samples` | 評估樣本數 | `4` |
| `--seed` | 隨機種子 | `23` |
| `--max_workers` | S4 worker 執行緒數 | `4` |
| `--out_folder` | 輸出目錄 | 自動時間戳 |

## 🧪 範例

### Smoke run

```bash
./ms_final.sh -ns 1000 -r 42 -p smoke -g 40 -bo 0.25 -ro 0.20
python merge_s4_data_full.py --prefix smoke
python -c "import pandas as pd; p='merged_s4_shapes_smoke.csv'; d=pd.read_csv(p).rename(columns={'folder_key':'prefix','NQ':'nQ'}); d.to_csv(p,index=False)"
mkdir -p merged_csvs && mv merged_s4_shapes_smoke.csv merged_csvs/
python three_stage_transmittance.py --preprocess --input_folder merged_csvs --output_npz smoke.npz
python three_stage_transmittance.py --data_npz smoke.npz --num_epochs 5 --batch_size 128
```

### 執行設定範例（來自 `commands.md` / `commands_updated.md`）

```bash
# no overlap, G=40
./ms_resume_allargs.sh -ns 10000 -r 12345 -p fast_without_overlap -g 40 -bo 0.25 -ro 0.2

# overlap, G=80
./ms_resume_allargs.sh -ns 10000 -r 12345 -p overlap -g 80 -bo 0.35 -ro 0.3

# large run, overlap, G=80
./ms_resume_allargs.sh -ns 100000 -r 12345 -p more_basis_overlap -g 80 -bo 0.35 -ro 0.3
```

## 🔬 研究脈絡

目前的反向設計設定，目標是由跨結晶狀態的透射率來回推具 C4 對稱的幾何形狀。現行透射率流程假設：

- 每個形狀樣本有 11 列結晶狀態（依唯一 `shape_uid` 分組）
- 每個結晶狀態有 100 個波長 bin（`T@...` 欄位）
- 最多 4 個 Q1 控制點，以 `4x3` 張量編碼：`(presence, x, y)`
- 形狀視覺化與一致性檢查時，於 C4 對稱下重建多邊形

儲存庫也包含主要透射率訓練路徑之外的探索分支（`AVIRIS*`、`noise_experiment*`、`archived/`）。

## 🧑‍💻 開發備註

- 這是以腳本為中心的研究儲存庫，而非封裝好的 Python 模組。
- 核心腳本依賴相對路徑（特別是 `../build/S4`、`results/`、`shapes/`）。
- `.gitignore` 會排除大量產生式實驗產物（`*.csv`、`*.npz`、`*.pt`、執行資料夾）。
- 歷史文件中的部分檔案/目錄在此快照中目前不存在；這些參考會刻意保留並附註，確保相容性。
- 儲存庫中存在 macOS sidecar 檔（`._*`），可能只是無功能的中繼資料產物。

## 🧯 疑難排解

| 症狀 | 可能原因 | 修正方式 |
|---|---|---|
| `../build/S4: No such file or directory` | 預期相對路徑缺少 S4 binary | 在 `../build/S4` 建置或連結 S4，或更新啟動腳本路徑 |
| `No transmission columns found` | CSV 缺少 `T@...` 欄位 | 重新檢查合併輸出格式 |
| `Must specify either --data_npz or --csv_file` | 缺少訓練/評估資料參數 | 明確提供其中一個輸入 |
| `No valid shapes => SHIFT->Q1->UpTo4` | `vertices_str` 無效/空值，或 Q1 篩選移除全部樣本 | 檢查 shape 檔與合併輸出 |
| `Empty merge output for --prefix` | Prefix 與 `results/` 內檔名不匹配 | 確認精確檔名前綴後重跑 merge |
| Evaluation checkpoint missing | 缺少 `stageA/B/C` checkpoint 檔案 | 確認 `--model_dir` 指向完整輸出資料夾 |
| `three_stage_transmittance_evaluation.py` not found | 歷史文件引用但目前不存在該腳本 | 使用 `FilterShapeS4_Evaluator_Transmittance.py`，或從舊版 commit 還原該腳本 |

## 🗺️ 路線圖

- 透過明確的資料版本清單與固定執行設定，提升可重現性。
- 整合透射率、AVIRIS 與噪聲分支的正規入口。
- 增加預處理與單次迷你訓練 epoch 的自動化 smoke tests。
- 加入更清楚的實驗註冊機制，將輸出資料夾對應到精確命令列。
- 擴充多語 README 同步流程（根目錄語言檔與 `i18n/`）。

## 🤝 貢獻

歡迎貢獻，特別是可重現性、測試與文件品質相關改進。

建議流程：

1. 先開 issue 說明範圍與預期行為。
2. 建立聚焦的分支。
3. 提交 pull request，附上可執行命令與輸出。
4. 盡可能讓變更聚焦於單一工作流程。

## 📄 授權

此快照的儲存庫根目錄目前沒有 `LICENSE` 檔案。請新增以明確定義使用與再散布條款。

## 📚 引用

若你使用本儲存庫或基於本研究進一步發展，請引用：

```bibtex
@article{chen2025inverse,
  title={Inverse Design of Metasurface for Spectral Imaging},
  author={Chen, Rongzhou and Nie, Haitao and Zhu, Shuo and Zhao, Yaping and Wang, Chutian and Lam, Edmund Y},
  journal={arXiv preprint arXiv:2510.21924},
  year={2025}
}
```

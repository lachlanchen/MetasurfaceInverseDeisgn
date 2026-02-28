[English](../README.md) · [العربية](README.ar.md) · [Español](README.es.md) · [Français](README.fr.md) · [日本語](README.ja.md) · [한국어](README.ko.md) · [Tiếng Việt](README.vi.md) · [中文 (简体)](README.zh-Hans.md) · [中文（繁體）](README.zh-Hant.md) · [Deutsch](README.de.md) · [Русский](README.ru.md)


[![LazyingArt banner](https://github.com/lachlanchen/lachlanchen/raw/main/figs/banner.png)](https://github.com/lachlanchen/lachlanchen/blob/main/figs/banner.png)

# 用於光譜影像的超表面逆向設計

<p align="center">
  <img alt="Status" src="https://img.shields.io/badge/Status-Research%20Prototype-f59e0b?style=for-the-badge">
  <img alt="Python" src="https://img.shields.io/badge/Python-3.9-3776AB?style=for-the-badge&logo=python&logoColor=white">
  <img alt="PyTorch" src="https://img.shields.io/badge/PyTorch-2.x-EE4C2C?style=for-the-badge&logo=pytorch&logoColor=white">
  <img alt="Simulator" src="https://img.shields.io/badge/Simulator-S4%20RCWA-16a34a?style=for-the-badge">
  <img alt="Platform" src="https://img.shields.io/badge/Platform-Linux%20%2B%20Bash-6b7280?style=for-the-badge&logo=gnu-bash&logoColor=white">
  <img alt="ArXiv" src="https://img.shields.io/badge/arXiv-2510.21924-b31b1b?style=for-the-badge&logo=arxiv&logoColor=white">
</p>

這是一個以腳本為先的研究型倉庫（歷史上稱為 `inverse_metasurface`），用於光譜影像中的超表面逆向設計。

核心流程包括：
- 以物理為基礎的 RCWA 模擬（`S4` + Lua）
- 資料彙整與形狀頂點附加
- 三階段 PyTorch 學習（`shape -> spectrum`、`spectrum -> shape`、接續微調）
- 定量與定性評估，並可選擇進行神經網路與 S4 的一致性再檢查

> [!IMPORTANT]
> 規範行為與命令沿用既有專案腳本/文件。歷史引用若指向此快照中不存在的檔案，這些引用會保留並附上明確註記，以維持歷史流程相容性。

## 📑 目錄

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

| 重點 | 狀態 |
|---|---|
| 🧠 目標 | 從光譜資料中反推出 C4 對稱超表面幾何 |
| 🔧 核心技術棧 | S4 RCWA（`Lua`） + PyTorch 訓練 + 可選擇幾何到光譜重驗證 |
| 🧪 資料流程 | CSV 合併/形狀頂點附加 → NPZ（`uids`、`spectra`、`shapes`） |
| 🚀 就緒度 | 研究型原型；腳本與文件保持與歷史參考相容 |

## ✨ At a Glance

| 項目 | 說明 |
|---|---|
| 🎯 主要任務 | 從目標透射譜反推 C4 對稱超表面幾何 |
| 🔬 模擬器 | 由 shell 啟動腳本與 `.lua` 腳本呼叫的 `../build/S4` |
| 🧠 學習流程 | Stage A `shape -> spectra`、Stage B `spectra -> shape`、Stage C `spectra -> shape -> spectra` |
| 📦 資料約定 | 合併 CSV（`T@...`、中介資料、`vertices_str`）→ 壓縮 NPZ（`uids`、`spectra`、`shapes`） |
| 🧪 評估 | MSE 指標、階段視覺化、可選新一輪 S4 重模擬 |
| 🌐 i18n 狀態 | 倉庫根目錄有多語系 README 檔案，且存在 `i18n/` 目錄 |

## 🌍 Internationalization (i18n)

- 多語系 README 維護為位於倉庫根目錄的 `README.<lang>.md` 檔案。
- 倉庫快照中存在 `i18n/` 目錄。
- 本檔僅在頂端保留一列語言連結，避免重複語言導覽列。
- 倉庫中也存在 `README.en.md`；目前 `README.md` 保持此輪更新的基準來源。

## ✨ Features

- 從 S4 模擬輸出到逆向模型訓練的一站式逆向設計流程。
- C4 對稱多邊形參數化與 Q1 點編碼（`4x3`：`presence, x, y`）。
- 在單一腳本中完成三階段模型訓練（`three_stage_transmittance.py`）。
- 合併工具在保留光譜精度的同時，為每個形狀附加頂點。
- 可選擇評估器，用於比較學習預測與新一次 S4 執行結果。
- 包含大量探索分支（AVIRIS、SWIR/noise、GSST、已封存/已棄用變體）。

## 🧭 End-to-End Workflow

1. 在 `results/` 產生模擬輸出，並在 `shapes/` 產生多邊形檔案。
2. 合併 S4 CSV 檔並附加形狀頂點。
3. 正規化合併後欄位名稱以符合訓練需求。
4. 將合併後的 CSV 預處理為 NPZ 張量。
5. 訓練 Stage A/B/C 模型。
6. 評估 checkpoint 並視覺化行為。
7. 可選：比較預測形狀光譜與新執行的 S4 結果。

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

| 依賴項 | 說明 |
|---|---|
| Linux + Bash | 啟動腳本以 shell 執行為目標 |
| Python 3.9 | 與 `iccp.yaml` 一致（`python=3.9.18`） |
| Conda | 建議用於結果可重現 |
| S4 二進位檔 | 預期位於 `../build/S4` |
| CUDA GPU（可選） | 可加速訓練與評估 |

## 🚀 Installation

### 1) 克隆並進入

```bash

git clone <your-repo-url> inverse_metasurface
cd inverse_metasurface
```

### 2) 建立環境（建議）

```bash
conda env create -f iccp.yaml
conda activate iccp
```

補充說明：

```bash
# Historical README reference (file may be absent in this snapshot)
pip install -r pip_requirements.txt
```

### 3) 確認腳本預期的模擬器路徑

```bash
ls -l ../build/S4
```

### 4) （可選）賦予啟動腳本執行權限

```bash
chmod +x ms.sh ms_final.sh ms_resume.sh ms_resume_allargs.sh ms_resume_random_state.sh ms_resume_random_state_nir.sh
```

## ▶️ Usage

### A) 產生 RCWA 模擬資料

基礎啟動器：

```bash
./ms.sh -ns 10000 -r 12345
```

帶參數啟動器：

```bash
./ms_final.sh \
  -ns 10000 \
  -r 12345 \
  -p myrun \
  -g 80 \
  -bo 0.35 \
  -ro 0.30
```

續跑型啟動器：

```bash
./ms_resume_allargs.sh \
  -ns 10000 \
  -r 12345 \
  -p myrun \
  -g 80 \
  -bo 0.35 \
  -ro 0.30
```

額外的 resume/random-state 範例（來自命令文件）：

```bash
./ms_resume_random_state.sh \
  -p iccp100kG20Ov \
  -r 88888 \
  -g 20 \
  -bo 0.35 \
  -ro 0.3 \
  -ns 100000
```

說明：
- 啟動器並行執行 `NQ=1..4`。
- 腳本以 `-t 32` 呼叫 `../build/S4`。

### B) 合併 S4 輸出並附加形狀頂點

```bash
python merge_s4_data_full.py --prefix myrun
# output: merged_s4_shapes_myrun.csv
```

### C) 正規化訓練列名稱

`merge_s4_data_full.py` 輸出 `folder_key` 和 `NQ`，而訓練流程預期 `prefix` 與 `nQ`。

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

### F) 評估訓練模型

歷史 README 命令（保留腳本名稱以維持與先前文件相容）：

```bash
python three_stage_transmittance_evaluation.py \
  --model_dir outputs_three_stage_YYYYMMDD_HHMMSS \
  --data_npz preprocessed_t_data.npz \
  --sample_count 8
```

倉庫狀態說明：`three_stage_transmittance_evaluation.py` 在目前快照中不存在。請改用 `FilterShapeS4_Evaluator_Transmittance.py` 取得可用的評估功能。

### G) 可選：神經網路與 S4 一致性檢查

```bash
python FilterShapeS4_Evaluator_Transmittance.py \
  --npz_file preprocessed_t_data.npz \
  --spec2shape_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageC/spec2shape_stageC.pt \
  --shape2spec_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageA/shape2spec_stageA.pt \
  --n_samples 4
```

## ⚙️ Configuration

### S4 啟動器（`ms_final.sh`、`ms_resume_allargs.sh`）

| 參數 | 含義 | 預設值 |
|---|---|---|
| `-ns`, `--numshapes` | 產生形狀數量 | `100000` |
| `-r`, `--seed` | 隨機種子 | `88888` |
| `-p`, `--prefix` | 前綴/恢復鍵值 | `""` |
| `-g`, `--numg` | 基礎/格點參數 | `80` |
| `-bo`, `--baseouter` | 外部邊界基準偏移 | `0.25` |
| `-ro`, `--randouter` | 外部邊界隨機偏移 | `0.20` |

### 訓練參數（`three_stage_transmittance.py`）

| 參數 | 含義 | 預設值 |
|---|---|---|
| `--preprocess` | 啟用預處理模式 | `False` |
| `--input_folder` | 存放合併 CSV 的資料夾 | `""` |
| `--output_npz` | 輸出 NPZ 路徑 | `preprocessed_data.npz` |
| `--data_npz` | 訓練用 NPZ 資料集 | `""` |
| `--csv_file` | 若未使用 NPZ 的 CSV 回退路徑 | `""` |
| `--test` | 測試模式 | `False` |
| `--num_epochs` | 訓練 epoch 數 | `10` |
| `--batch_size` | 批次大小 | `4096` |

### 歷史評估設定（`three_stage_transmittance_evaluation.py`）

| 參數 | 含義 | 預設值 |
|---|---|---|
| `--model_dir` | 包含 `stageA/B/C` 的目錄 | required |
| `--data_npz` | NPZ 輸入 | `""` |
| `--csv_file` | CSV 回退輸入 | `""` |
| `--output_dir` | 輸出目錄覆蓋 | 自動在 `model_dir` 下產生 |
| `--sample_count` | 要視覺化的樣本數 | `4` |
| `--seed` | 隨機種子 | `23` |
| `--font_scale` | 繪圖字體縮放 | `1.0` |
| `--batch_size` | 評估批次大小 | `32` |
| `--plot_only` | 僅繪製訓練曲線 | `False` |

### S4 一致性評估器（`FilterShapeS4_Evaluator_Transmittance.py`）

| 參數 | 含義 | 預設值 |
|---|---|---|
| `--npz_file` | 輸入 NPZ 檔案 | `preprocessed_t_data.npz` |
| `--spec2shape_ckpt` | Stage C 檢查點路徑 | `outputs_three_stage_20250322_145925/stageC/spec2shape_stageC.pt` |
| `--shape2spec_ckpt` | Stage A 檢查點路徑 | `outputs_three_stage_20250322_145925/stageA/shape2spec_stageA.pt` |
| `--n_samples` | 評估樣本數 | `4` |
| `--seed` | 隨機種子 | `23` |
| `--max_workers` | S4 工作執行緒數 | `4` |
| `--out_folder` | 輸出目錄 | 自動時間戳 |

## 🧪 Examples

### 冒煙運行

```bash
./ms_final.sh -ns 1000 -r 42 -p smoke -g 40 -bo 0.25 -ro 0.20
python merge_s4_data_full.py --prefix smoke
python -c "import pandas as pd; p='merged_s4_shapes_smoke.csv'; d=pd.read_csv(p).rename(columns={'folder_key':'prefix','NQ':'nQ'}); d.to_csv(p,index=False)"
mkdir -p merged_csvs && mv merged_s4_shapes_smoke.csv merged_csvs/
python three_stage_transmittance.py --preprocess --input_folder merged_csvs --output_npz smoke.npz
python three_stage_transmittance.py --data_npz smoke.npz --num_epochs 5 --batch_size 128
```

### 來自 `commands.md` / `commands_updated.md` 的執行範例

```bash
# no overlap, G=40
./ms_resume_allargs.sh -ns 10000 -r 12345 -p fast_without_overlap -g 40 -bo 0.25 -ro 0.2

# overlap, G=80
./ms_resume_allargs.sh -ns 10000 -r 12345 -p overlap -g 80 -bo 0.35 -ro 0.3

# large run, overlap, G=80
./ms_resume_allargs.sh -ns 100000 -r 12345 -p more_basis_overlap -g 80 -bo 0.35 -ro 0.3
```

## 🔬 Research Context

目前的逆向設計設定學習在結晶狀態下由透射率恢復 C4 對稱幾何。透射率流程目前假設：

- 每個形狀樣本有 11 筆結晶狀態列（依唯一 `shape_uid` 分組）
- 每個結晶狀態有 100 個波長採樣點（`T@...` 欄位）
- 使用 `4x3` 張量編碼最高 4 個 Q1 控制點：`(presence, x, y)`
- 透過 C4 對稱下的多邊形重建，用於形狀視覺化與一致性檢查

除主要透射率訓練路徑外，倉庫也包含探索性分支（`AVIRIS*`、`noise_experiment*`、`archived/`）。

## 🧑‍💻 Development Notes

- 這是一個以腳本為核心的研究倉庫，而非打包成的 Python 模組。
- 核心腳本依賴相對路徑（尤其是 `../build/S4`、`results/`、`shapes/`）。
- `.gitignore` 排除大量產生的實驗成果物（`*.csv`、`*.npz`、`*.pt`、執行目錄）。
- 歷史文件中的部分檔案/目錄在目前快照中缺失；這些引用保留並附上相容性說明。
- macOS 副檔（`._*`）可能存在，通常是無功能的中繼資料檔。

## 🧯 Troubleshooting

| 症狀 | 可能原因 | 解法 |
|---|---|---|
| `../build/S4: No such file or directory` | 預期路徑下缺少 S4 二進位檔 | 在 `../build/S4` 建立或連結 S4，或更新啟動器路徑 |
| `No transmission columns found` | CSV 缺少 `T@...` 欄位 | 重新檢查合併輸出格式 |
| `Must specify either --data_npz or --csv_file` | 訓練/評估資料參數缺漏 | 明確提供其中一個輸入 |
| `No valid shapes => SHIFT->Q1->UpTo4` | `vertices_str` 無效或空，或 Q1 過濾後無有效樣本 | 驗證形狀檔與合併輸出 |
| Empty merge output for `--prefix` | `--prefix` 與 `results/` 檔案不匹配 | 檢查前綴名稱是否完全一致後重跑合併 |
| Evaluation checkpoint missing | 缺少 `stageA/B/C` checkpoint 檔 | 確認 `--model_dir` 指向完整輸出資料夾 |
| `three_stage_transmittance_evaluation.py` not found | 歷史文件所引用腳本目前不存在 | 改用 `FilterShapeS4_Evaluator_Transmittance.py`，或自舊版提交還原 |

## 🗺️ Roadmap

- 透過明確的資料版本清單與鎖定執行設定提升可重現性。
- 整併透射率、AVIRIS 與 noise 分支的統一入口。
- 為預處理與單輪小規模訓練補充自動化冒煙測試。
- 改善實驗登錄，將輸出目錄與精確命令列關聯。
- 擴展多語系 README 同步工作流（根目錄語言檔與 `i18n/`）。

## 🤝 Contribution

歡迎投稿，特別是在可重現性、測試與文件品質方面。

建議流程：

1. 開啟 issue，說明範圍與預期行為。
2. 建立專注分支。
3. 提交可執行的指令與輸出。
4. 盡量將變更限定在單一工作流程。

## ❤️ Support

| Donate | PayPal | Stripe |
| --- | --- | --- |
| [![Donate](https://camo.githubusercontent.com/24a4914f0b42c6f435f9e101621f1e52535b02c225764b2f6cc99416926004b7/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f446f6e6174652d4c617a79696e674172742d3045413545393f7374796c653d666f722d7468652d6261646765266c6f676f3d6b6f2d6669266c6f676f436f6c6f723d7768697465)](https://chat.lazying.art/donate) | [![PayPal](https://camo.githubusercontent.com/d0f57e8b016517a4b06961b24d0ca87d62fdba16e18bbdb6aba28e978dc0ea21/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f50617950616c2d526f6e677a686f754368656e2d3030343537433f7374796c653d666f722d7468652d6261646765266c6f676f3d70617970616c266c6f676f436f6c6f723d7768697465)](https://paypal.me/RongzhouChen) | [![Stripe](https://camo.githubusercontent.com/1152dfe04b6943afe3a8d2953676749603fb9f95e24088c92c97a01a897b4942/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f5374726970652d446f6e6174652d3633354246463f7374796c653d666f722d7468652d6261646765266c6f676f3d737472697065266c6f676f436f6c6f723d7768697465)](https://buy.stripe.com/aFadR8gIaflgfQV6T4fw400) |

## 📄 License

目前快照的倉庫根目錄尚未包含 `LICENSE` 檔案。請補充該檔案以定義使用與再散佈條款。

## 📚 Citation

如果您使用本倉庫或在此基礎上展開研究，請引用：

```bibtex
@article{chen2025inverse,
  title={Inverse Design of Metasurface for Spectral Imaging},
  author={Chen, Rongzhou and Nie, Haitao and Zhu, Shuo and Zhao, Yaping and Wang, Chutian and Lam, Edmund Y},
  journal={arXiv preprint arXiv:2510.21924},
  year={2025}
}
```

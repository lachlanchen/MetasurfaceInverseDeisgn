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

[![Status](https://img.shields.io/badge/Status-Research%20Prototype-orange)](#-專案範圍)
[![Python](https://img.shields.io/badge/Python-3.9-3776AB?logo=python&logoColor=white)](#-環境設定)
[![Platform](https://img.shields.io/badge/Platform-Linux-2f2f2f?logo=linux&logoColor=white)](#-先決條件)
[![RCWA](https://img.shields.io/badge/RCWA-S4%20Required-1f9d55)](#-先決條件)
[![PyTorch](https://img.shields.io/badge/Framework-PyTorch-EE4C2C?logo=pytorch&logoColor=white)](#-訓練與評估)
[![License](https://img.shields.io/badge/License-Not%20Specified-lightgrey)](#-授權)

這是一個以腳本為主的研究型程式碼庫，聚焦於 **C4 對稱條件下的反向超表面設計**。
它整合了：

- 🔬 S4/RCWA 模擬（`.lua` + Bash 啟動腳本）
- 🧱 由原始光譜與形狀頂點進行資料合併與前處理
- 🧠 三階段 PyTorch 訓練（`shape -> spectra`、`spectra -> shape`、鏈式微調）
- 📊 評估與繪圖，包含神經網路與 S4 一致性檢查

## 📌 目錄

- [專案範圍](#-專案範圍)
- [研究背景](#-研究背景)
- [儲存庫結構](#-儲存庫結構)
- [先決條件](#-先決條件)
- [環境設定](#-環境設定)
- [快速開始](#-快速開始)
- [端到端流程](#-端到端流程)
- [訓練與評估](#-訓練與評估)
- [重要 CLI 選項](#-重要-cli-選項)
- [疑難排解](#-疑難排解)
- [路線圖](#-路線圖)
- [引用](#-引用)
- [授權](#-授權)

## 🎯 專案範圍

此儲存庫偏重實驗，核心是可直接執行的腳本（不是封裝後的函式庫）。
目前最穩定的工作流程是：

1. 執行 S4，於 `results/` 產生原始光學輸出，並於 `shapes/` 產生形狀資料
2. 將每次執行的 CSV 檔案合併與 pivot 成可訓練表格資料
3. 將合併後 CSV 轉成壓縮 `.npz`
4. 訓練三階段透射率管線
5. 評估 checkpoint 並匯出圖表/指標

## 🧪 研究背景

### 問題設定

核心反向設計任務是在 C4 對稱限制與部分結晶掃描條件下，根據目標透射光譜回推超表面幾何（反之亦然）。

### 透射率管線中的資料假設

| 項目 | 值 |
|---|---|
| 每個形狀的結晶狀態數 | 11（`c` 從 `0.0` 到 `1.0`） |
| 每個狀態的光譜 bins | 100 |
| 每筆樣本的光譜張量 | `11 x 100` |
| 每筆樣本的形狀張量 | `4 x 3`（`[presence, x, y]`） |

### 三階段學習目標

| 階段 | 方向 | 常見 checkpoint |
|---|---|---|
| A | `shape -> spectrum` | `stageA/shape2spec_stageA.pt` |
| B | `spectrum -> shape` | `stageB/spec2shape_stageB.pt` |
| C | `spectrum -> shape -> spectrum`（鏈式微調） | `stageC/spec2shape_stageC.pt` |

## 🗂️ 儲存庫結構

```text
.
├── ms.sh / ms_final.sh / ms_resume_allargs.sh
├── metasurface_seed.lua / metasurface_final.lua / metasurface_allargs_resume.lua
├── merge.py / merge_s4_data_full.py / merge_s4_data_local.py / merge_robust.py
├── three_stage_transmittance.py
├── three_stage_transmittance_evaluation.py
├── FilterShapeS4_Evaluator_Transmittance.py
├── partial_crys_data/
├── results/                          # 原始 S4 輸出
├── shapes/                           # 產生的多邊形頂點
├── outputs_three_stage_*/            # checkpoints + 訓練產物
├── AVIRIS*/ and aviris_*.py          # 相關高光譜實驗
├── commands.md / how_to_run.md
├── iccp.yaml
└── pip_requirements.txt
```

## 🧩 先決條件

| 相依項目 | 說明 |
|---|---|
| Linux + Bash | Shell 腳本預設使用 Linux 風格路徑 |
| Conda | 建議的環境管理方式（`iccp.yaml`） |
| Python 3.9 | 訓練/評估腳本主要執行環境 |
| S4 binary | 預期位於儲存庫根目錄相對路徑 `../build/S4` |
| CUDA（可選） | 可加速訓練與評估 |

## ⚙️ 環境設定

```bash
git clone <repo-url> inverse_metasurface
cd inverse_metasurface

conda env create -f iccp.yaml
conda activate iccp

# verify S4 path expected by shell runners
ls -l ../build/S4
```

可選：為 shell 啟動腳本加上可執行權限：

```bash
chmod +x ms.sh ms_final.sh ms_resume_allargs.sh ms_resume.sh
```

## 🚀 快速開始

如果你已經有 `preprocessed_t_data.npz`：

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

## 🔁 端到端流程

### 1) 產生 S4 資料

```bash
./ms_final.sh \
  -ns 10000 \
  -r 12345 \
  -p myrun \
  -g 80 \
  -bo 0.35 \
  -ro 0.30
```

### 2) 合併 S4 輸出與形狀頂點

```bash
python merge_s4_data_full.py --prefix myrun
# -> merged_s4_shapes_myrun.csv
```

### 3) 正規化 CSV 欄位名稱以供前處理

`three_stage_transmittance.py` 的前處理預期欄位為 `prefix` 與 `nQ`，而合併輸出可能是 `folder_key` 與 `NQ`。

```bash
python -c "import pandas as pd; p='merged_s4_shapes_myrun.csv'; df=pd.read_csv(p); df=df.rename(columns={'folder_key':'prefix','NQ':'nQ'}); df.to_csv(p,index=False)"
```

### 4) CSV -> NPZ 前處理

```bash
mkdir -p merged_csvs
mv merged_s4_shapes_myrun.csv merged_csvs/

python three_stage_transmittance.py --preprocess \
  --input_folder merged_csvs \
  --output_npz preprocessed_t_data.npz
```

### 5) 訓練三個階段

```bash
python three_stage_transmittance.py \
  --data_npz preprocessed_t_data.npz \
  --num_epochs 100 \
  --batch_size 1024
```

### 6) 評估並繪圖

```bash
python three_stage_transmittance_evaluation.py \
  --model_dir outputs_three_stage_YYYYMMDD_HHMMSS \
  --data_npz preprocessed_t_data.npz \
  --sample_count 8
```

### 7) 可選：神經網路與 S4 比較

```bash
python FilterShapeS4_Evaluator_Transmittance.py \
  --npz_file preprocessed_t_data.npz \
  --spec2shape_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageC/spec2shape_stageC.pt \
  --shape2spec_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageA/shape2spec_stageA.pt \
  --n_samples 4
```

## 🧠 訓練與評估

### 主要腳本

| 腳本 | 用途 |
|---|---|
| `three_stage_transmittance.py` | 前處理 + 訓練 A/B/C 三階段 |
| `three_stage_transmittance_evaluation.py` | 評估 checkpoints、計算指標、儲存圖表 |
| `FilterShapeS4_Evaluator_Transmittance.py` | 比較學習模型預測與 S4 行為 |

### 常見輸出

| 產物 | 位置 |
|---|---|
| 各階段 checkpoints | `outputs_three_stage_*/stageA|stageB|stageC/` |
| 評估圖表 | `outputs_three_stage_*/evaluation_<timestamp>/` |
| 指標 CSV | `evaluation_metrics.csv`, `metrics_summary.csv` |

## 🛠️ 重要 CLI 選項

### S4 啟動腳本（`ms_final.sh`, `ms_resume_allargs.sh`）

| 旗標 | 含義 | 預設值 |
|---|---|---|
| `-ns`, `--numshapes` | 形狀數量 | `100000` |
| `-r`, `--seed` | 隨機種子 | `88888` |
| `-p`, `--prefix` | 執行前綴/續跑 key | 空 |
| `-g`, `--numg` | 幾何基底設定 | `80` |
| `-bo`, `--baseouter` | 基礎外偏移 | `0.25` |
| `-ro`, `--randouter` | 隨機外偏移 | `0.20` |

### 訓練（`three_stage_transmittance.py`）

| 旗標 | 含義 | 預設值 |
|---|---|---|
| `--preprocess` | 切換到前處理模式 | off |
| `--input_folder` | 放置合併 CSV 的資料夾 | `""` |
| `--output_npz` | 前處理輸出檔案 | `preprocessed_data.npz` |
| `--data_npz` | 訓練用 NPZ 輸入 | `""` |
| `--csv_file` | 直接以 CSV 訓練輸入 | `""` |
| `--test` | 測試模式切換 | off |
| `--num_epochs` | 每階段 epoch 數 | `10` |
| `--batch_size` | batch size | `4096` |

### 評估（`three_stage_transmittance_evaluation.py`）

| 旗標 | 含義 | 預設值 |
|---|---|---|
| `--model_dir` | 訓練執行目錄（必填） | - |
| `--data_npz` | 評估用 NPZ 輸入 | `""` |
| `--csv_file` | 評估用 CSV 輸入 | `""` |
| `--output_dir` | 自訂輸出目錄 | auto |
| `--sample_count` | 視覺化樣本數 | `4` |
| `--seed` | 樣本選取隨機種子 | `23` |
| `--font_scale` | 圖表字體倍率 | `1.0` |
| `--batch_size` | eval dataloader batch size | `32` |
| `--plot_only` | 僅重新產生圖表 | off |

## 🧯 疑難排解

| 症狀 | 可能原因 | 修正方式 |
|---|---|---|
| `../build/S4: No such file or directory` | S4 binary 不在預期相對路徑 | 將 S4 放置/建置於 `../build/S4`，或修改腳本 |
| `Must specify either --data_npz or --csv_file` | 缺少訓練/評估資料集參數 | 精確提供其中一個資料輸入 |
| `No transmission columns found` | 合併後 CSV 缺少 `T@...` 欄位 | 重新執行 merge/pivot 並確認欄位名稱 |
| `KeyError: 'prefix'` in preprocess | 合併輸出仍使用 `folder_key`/`NQ` | 前處理前先將欄位改為 `prefix`/`nQ` |
| GPU OOM | batch 過大 | 降低 `--batch_size` |
| Missing checkpoints during eval | 缺少階段 checkpoint 或路徑錯誤 | 確認選定 `--model_dir` 下有 stageA/B/C 檔案 |

## 🧭 路線圖

- 在各 merge 腳本間統一合併 CSV schema（`prefix`、`nQ` 命名）
- 為 merge/preprocess/checkpoint 載入新增自動化測試
- 提供單一 CLI 入口以編排完整流程
- 新增資料集/執行 manifest 以提升可重現性
- 補上明確的開源授權條款

## 📚 引用

若本儲存庫對你的研究有所幫助，請引用：

```bibtex
@article{chen2025inverse,
  title={Inverse Design of Metasurface for Spectral Imaging},
  author={Chen, Rongzhou and Nie, Haitao and Zhu, Shuo and Zhao, Yaping and Wang, Chutian and Lam, Edmund Y},
  journal={arXiv preprint arXiv:2510.21924},
  year={2025}
}
```

## 📄 授權

此儲存庫目前沒有 `LICENSE` 檔案。在新增授權之前，使用與再散布權利皆屬未明確定義狀態。

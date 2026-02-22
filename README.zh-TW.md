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

這是一個以腳本為核心的研究型儲存庫（歷史上也稱為 `inverse_metasurface`），聚焦於**用於光譜成像的 C4 對稱超表面反向設計**，涵蓋：

- 使用 S4 進行 RCWA 資料生成（`.lua` + shell 啟動腳本）
- 資料合併與前處理（`.csv` -> `.npz`）
- 三階段神經網路訓練（shape->spectra、spectra->shape、chain fine-tuning）
- 評估與可選的 neural-vs-S4 比對

## ✨ 快速總覽

| 項目 | 說明 |
|---|---|
| 核心目標 | 由目標透射光譜預測幾何形狀 |
| 核心資料形狀 | spectra: `11 x 100`, shape: `4 x 3` |
| 主要訓練腳本 | `three_stage_transmittance.py` |
| 主要評估腳本 | `three_stage_transmittance_evaluation.py` |
| RCWA 啟動腳本 | `ms_final.sh`, `ms_resume_allargs.sh` |
| 合併腳本 | `merge_s4_data_full.py` |

## 🧠 研究背景

本專案著重於光譜成像之超表面反向設計。訓練流程使用 S4 生成的多結晶化狀態透射光譜，同時學習正向與反向映射：

1. **Stage A**: shape -> spectra
2. **Stage B**: spectra -> shape
3. **Stage C**: spectra -> shape -> spectra（以 chain loss 微調）

目前前處理／訓練程式碼假設：

- 11 個結晶化狀態（`c = 0.0 ... 1.0`）
- 每個狀態 100 個波長 bins
- 形狀表示為最多 4 個 Q1 點，每點為 `[presence, x, y]`

## 🗂️ 儲存庫結構（核心路徑）

```text
.
├── ms.sh / ms_final.sh / ms_resume_allargs.sh
├── metasurface_final.lua / metasurface_allargs_resume.lua / metasurface_seed.lua
├── merge_s4_data_full.py
├── three_stage_transmittance.py
├── three_stage_transmittance_evaluation.py
├── FilterShapeS4_Evaluator_Transmittance.py
├── results/                # raw S4 output CSVs
├── shapes/                 # generated polygon vertices
├── merged_csvs/            # merged CSVs used for preprocessing
├── outputs_three_stage_*/  # checkpoints, losses, visualizations
├── partial_crys_data/
└── iccp.yaml
```

## ⚙️ 先決條件

| 依賴項 | 需求 |
|---|---|
| OS | Linux |
| Shell | Bash |
| Python | 3.9 |
| 環境管理 | Conda（建議） |
| RCWA 二進位檔 | `../build/S4`（相對於 repo root） |
| GPU | 可選，建議使用以加速訓練 |

## 🚀 安裝設定

```bash
git clone <repo-url> inverse_metasurface
cd inverse_metasurface

conda env create -f iccp.yaml
conda activate iccp

# Required by launcher scripts
ls -l ../build/S4
```

可選：

```bash
chmod +x ms.sh ms_final.sh ms_resume_allargs.sh
```

## 🧪 端到端使用流程

### 1) 使用 S4 生成 RCWA 資料

```bash
./ms_final.sh \
  -ns 10000 \
  -r 12345 \
  -p myrun \
  -g 80 \
  -bo 0.35 \
  -ro 0.30
```

說明：

- 啟動腳本會以 `-t 32` 呼叫 `../build/S4`，並平行執行 `NQ=1..4`。
- `ms_final.sh` 使用 `metasurface_final.lua`。
- `ms_resume_allargs.sh` 使用 `metasurface_allargs_resume.lua`。

### 2) 合併 RCWA 輸出與形狀頂點

```bash
python merge_s4_data_full.py --prefix myrun
# -> merged_s4_shapes_myrun.csv
```

### 3) 將合併欄位名稱正規化以供訓練（如有需要）

`merge_s4_data_full.py` 會輸出 `folder_key` / `NQ`，但訓練流程預期欄位為 `prefix` / `nQ`。

```bash
python -c "import pandas as pd; p='merged_s4_shapes_myrun.csv'; df=pd.read_csv(p); df=df.rename(columns={'folder_key':'prefix','NQ':'nQ'}); df.to_csv(p,index=False)"
```

### 4) 前處理 CSV -> NPZ

```bash
mkdir -p merged_csvs
mv merged_s4_shapes_myrun.csv merged_csvs/

python three_stage_transmittance.py \
  --preprocess \
  --input_folder merged_csvs \
  --output_npz preprocessed_t_data.npz
```

### 5) 訓練三階段模型

```bash
python three_stage_transmittance.py \
  --data_npz preprocessed_t_data.npz \
  --num_epochs 100 \
  --batch_size 1024
```

主要輸出結構：

- `outputs_three_stage_YYYYMMDD_HHMMSS/stageA`
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageB`
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageC`

### 6) 評估 checkpoints

```bash
python three_stage_transmittance_evaluation.py \
  --model_dir outputs_three_stage_YYYYMMDD_HHMMSS \
  --data_npz preprocessed_t_data.npz \
  --sample_count 8
```

### 7) 可選：比較神經網路預測與 S4

```bash
python FilterShapeS4_Evaluator_Transmittance.py \
  --npz_file preprocessed_t_data.npz \
  --spec2shape_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageC/spec2shape_stageC.pt \
  --shape2spec_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageA/shape2spec_stageA.pt \
  --n_samples 4
```

## 🎛️ CLI 參考

### S4 啟動參數（`ms_final.sh`, `ms_resume_allargs.sh`）

| 旗標 | 意義 | 預設值 |
|---|---|---|
| `-ns`, `--numshapes` | 形狀數量 | `100000` |
| `-r`, `--seed` | 隨機種子 | `88888` |
| `-p`, `--prefix` | 執行前綴／續跑鍵值 | `""` |
| `-g`, `--numg` | 幾何基底參數 | `80` |
| `-bo`, `--baseouter` | 外層基準偏移 | `0.25` |
| `-ro`, `--randouter` | 外層隨機偏移 | `0.20` |

### 訓練參數（`three_stage_transmittance.py`）

| 旗標 | 用途 |
|---|---|
| `--preprocess` | 執行前處理模式 |
| `--input_folder` | 合併 CSV 檔資料夾 |
| `--output_npz` | 輸出前處理 NPZ 檔名 |
| `--data_npz` | 訓練用 NPZ 資料集 |
| `--csv_file` | CSV 資料集替代來源 |
| `--test` | 測試模式 |
| `--num_epochs` | 訓練回合數 |
| `--batch_size` | Batch 大小 |

### 評估參數（`three_stage_transmittance_evaluation.py`）

| 旗標 | 用途 |
|---|---|
| `--model_dir` | checkpoint 根目錄（必填） |
| `--data_npz` / `--csv_file` | 評估資料來源 |
| `--output_dir` | 評估輸出資料夾 |
| `--sample_count` | 視覺化樣本數 |
| `--seed` | 樣本選擇的隨機種子 |
| `--font_scale` | 圖表字體縮放 |
| `--batch_size` | 評估 batch 大小 |
| `--plot_only` | 只重新生成訓練曲線圖 |

## 🧾 資料契約（前處理用）

`three_stage_transmittance.py` 的前處理流程預期合併 CSV 包含：

- ID 欄位：`prefix`, `nQ`, `nS`, `shape_idx`, `c`
- 幾何文字：`vertices_str`
- 光譜欄位：`T@...`

程式碼會執行的品質檢查：

- 以 `shape_uid = prefix_nQ_nS_shape_idx` 分組
- 每個分組必須剛好有 11 列
- 僅保留 Q1 點數在 `[1, 4]` 的形狀

## 🛠️ 疑難排解

- `../build/S4: No such file or directory`
  - 在 `../build/S4` 建置／連結 S4，或將啟動腳本改為你的實際 S4 路徑。
- `No matching CSVs found in 'results/'`
  - 檢查 `--prefix` 與 `results/*_output_nQ*_nS*.csv` 的輸出命名。
- `No transmission columns found`
  - 確認合併 CSV 含有 `T@...` 欄位。
- 前處理結果為零筆
  - 檢查必要欄位，並確認每個 shape UID 皆有 11 列結晶化資料。
- 訓練時 GPU OOM
  - 調小 `--batch_size`（例如 `256` 或 `128`）。
- 評估找不到 checkpoints
  - 確認 `--model_dir` 底下存在：
    - `stageA/shape2spec_stageA.pt`
    - `stageB/spec2shape_stageB.pt`
    - `stageC/spec2shape_stageC.pt`

## 📚 引用

如果此儲存庫對你的研究有幫助，請引用：

```bibtex
@article{chen2025inverse,
  title={Inverse Design of Metasurface for Spectral Imaging},
  author={Chen, Rongzhou and Nie, Haitao and Zhu, Shuo and Zhao, Yaping and Wang, Chutian and Lam, Edmund Y},
  journal={arXiv preprint arXiv:2510.21924},
  year={2025}
}
```

## 🌐 語言版本

此儲存庫另提供多種 README 版本，包括：

- `README.en.md`, `README.de.md`, `README.es.md`, `README.fr.md`
- `README.ru.md`, `README.ja.md`, `README.ko.md`, `README.vi.md`
- `README.ar.md`, `README.zh-CN.md`, `README.zh-TW.md`

## 📌 備註

- 這是研究工作區，包含許多封存與探索性腳本。
- 主要透射率流程聚焦於：
  - `ms_final.sh` / `ms_resume_allargs.sh`
  - `merge_s4_data_full.py`
  - `three_stage_transmittance.py`
  - `three_stage_transmittance_evaluation.py`
- 目前在儲存庫根目錄尚未定義明確的授權檔案。

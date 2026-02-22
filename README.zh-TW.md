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

這是一個以腳本為核心的研究型程式庫（歷史上也稱為 `inverse_metasurface`），用於光譜成像中的超表面反向設計。整體流程將 **基於 S4 的 RCWA 模擬** 與 **三階段 PyTorch 工作流** 結合，用於幾何結構與光學透射光譜之間的正向與反向映射。

## ✨ 快速總覽

| 項目 | 說明 |
|---|---|
| 🎯 目標 | 由目標透射率光譜預測具 C4 對稱性的超表面幾何 |
| 🔬 物理模擬 | 使用 S4 進行 RCWA 模擬（`../build/S4`） |
| 🧠 學習流程 | Stage A `shape -> spectra`、Stage B `spectra -> shape`、Stage C `spectra -> shape -> spectra` |
| 📦 資料形式 | 合併 CSV（`T@...`、形狀中繼資料）-> 壓縮 NPZ（`uids`、`spectra`、`shapes`） |
| 🧪 評估方式 | MSE 指標、定性圖像、可選的 S4 重新模擬比對 |

## 🧭 端到端流程

1. 使用 S4（`.lua` + shell 啟動腳本）生成超表面光學響應。
2. 合併原始模擬 CSV，並附加多邊形頂點資訊。
3. 將合併後 CSV 轉換為訓練用 NPZ。
4. 訓練三階段透射率管線。
5. 評估模型 checkpoint，並視覺化 Stage A/B/C 行為。
6. 可選：將神經網路預測與新一輪 S4 模擬進行比較。

## 🧱 儲存庫結構

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
├── results/                # S4 原始 CSV 輸出
├── shapes/                 # 合併時使用的多邊形頂點檔
├── merged_csvs/            # 合併後 CSV 資料集
├── outputs_three_stage_*/  # 訓練 checkpoint 與曲線
├── partial_crys_data/      # 晶化狀態光學表格
│
├── AVIRIS*/                # 次要高光譜實驗
├── noise_experiment_*/     # 強健性/雜訊分支
└── archived/               # 歷史腳本與快照
```

## 🛠️ 先決條件

- Linux + Bash
- Conda（建議）
- Python 3.9
- `../build/S4` 位置可用的 S4 執行檔
- 可選：支援 CUDA 的 GPU（可加速訓練）

## 🚀 安裝與環境設定

```bash
git clone <your-repo-url> inverse_metasurface
cd inverse_metasurface

conda env create -f iccp.yaml
conda activate iccp

# Verify simulator path expected by scripts
ls -l ../build/S4
```

可選：

```bash
chmod +x ms.sh ms_final.sh ms_resume_allargs.sh
```

## ▶️ 實務使用流程

### 1) 生成 RCWA 資料

`ms_final.sh` 與 `ms_resume_allargs.sh` 會各自啟動 4 個平行 S4 工作（`NQ=1..4`）：

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
- `ms.sh` 是較簡化的啟動路徑。
- 啟動腳本預設使用 `../build/S4`，並帶入 `-t 32`。

### 2) 合併 S4 輸出與形狀頂點

```bash
python merge_s4_data_full.py --prefix myrun
# output: merged_s4_shapes_myrun.csv
```

### 3) 為訓練相容性正規化欄位名稱

`merge_s4_data_full.py` 會輸出 `folder_key` / `NQ`，而 `three_stage_transmittance.py` 需要 `prefix` / `nQ`。

```bash
python -c "import pandas as pd; p='merged_s4_shapes_myrun.csv'; df=pd.read_csv(p); df=df.rename(columns={'folder_key':'prefix','NQ':'nQ'}); df.to_csv(p,index=False)"
```

### 4) 將 CSV 前處理為 NPZ

```bash
mkdir -p merged_csvs
mv merged_s4_shapes_myrun.csv merged_csvs/

python three_stage_transmittance.py \
  --preprocess \
  --input_folder merged_csvs \
  --output_npz preprocessed_t_data.npz
```

### 5) 訓練三階段管線

```bash
python three_stage_transmittance.py \
  --data_npz preprocessed_t_data.npz \
  --num_epochs 100 \
  --batch_size 1024
```

預期輸出路徑模式：
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageA`
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageB`
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageC`

### 6) 評估 Stage A/B/C 模型

```bash
python three_stage_transmittance_evaluation.py \
  --model_dir outputs_three_stage_YYYYMMDD_HHMMSS \
  --data_npz preprocessed_t_data.npz \
  --sample_count 8
```

### 7) 可選：神經網路與 S4 一致性檢查

```bash
python FilterShapeS4_Evaluator_Transmittance.py \
  --npz_file preprocessed_t_data.npz \
  --spec2shape_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageC/spec2shape_stageC.pt \
  --shape2spec_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageA/shape2spec_stageA.pt \
  --n_samples 4
```

## ⚙️ 主要 CLI 選項

### S4 啟動腳本（`ms_final.sh`, `ms_resume_allargs.sh`）

| 旗標 | 說明 | 預設值 |
|---|---|---|
| `-ns`, `--numshapes` | 要生成的形狀數量 | `100000` |
| `-r`, `--seed` | 隨機種子 | `88888` |
| `-p`, `--prefix` | 前綴/續跑鍵值 | `""` |
| `-g`, `--numg` | 基底/幾何參數 | `80` |
| `-bo`, `--baseouter` | 外邊界基礎偏移 | `0.25` |
| `-ro`, `--randouter` | 外邊界隨機偏移 | `0.20` |

### 訓練（`three_stage_transmittance.py`）

| 旗標 | 用途 | 預設值 |
|---|---|---|
| `--preprocess` | 執行前處理模式 | `False` |
| `--input_folder` | 含合併 CSV 的資料夾 | `""` |
| `--output_npz` | 輸出 NPZ 路徑 | `preprocessed_data.npz` |
| `--data_npz` | 訓練用 NPZ | `""` |
| `--csv_file` | 未使用 NPZ 時的 CSV 後備輸入 | `""` |
| `--test` | 測試模式（跳過訓練） | `False` |
| `--num_epochs` | 訓練 epoch 數 | `10` |
| `--batch_size` | 批次大小 | `4096` |

### 評估（`three_stage_transmittance_evaluation.py`）

| 旗標 | 用途 | 預設值 |
|---|---|---|
| `--model_dir` | 包含 `stageA/B/C` 的根目錄 | 必填 |
| `--data_npz` | 評估用 NPZ 輸入 | `""` |
| `--csv_file` | 評估用 CSV 輸入 | `""` |
| `--output_dir` | 覆寫輸出目錄 | 自動設於 `model_dir` |
| `--sample_count` | 視覺化樣本數 | `4` |
| `--seed` | 抽樣隨機種子 | `23` |
| `--font_scale` | 繪圖字體縮放 | `1.0` |
| `--batch_size` | 評估批次大小 | `32` |
| `--plot_only` | 僅繪製曲線 | `False` |

### S4 一致性評估器（`FilterShapeS4_Evaluator_Transmittance.py`）

| 旗標 | 用途 | 預設值 |
|---|---|---|
| `--npz_file` | 前處理後 NPZ 檔案 | `preprocessed_t_data.npz` |
| `--spec2shape_ckpt` | Stage C checkpoint | `outputs_three_stage_20250322_145925/stageC/spec2shape_stageC.pt` |
| `--shape2spec_ckpt` | Stage A checkpoint | `outputs_three_stage_20250322_145925/stageA/shape2spec_stageA.pt` |
| `--n_samples` | 要檢視的樣本數 | `4` |
| `--seed` | 隨機種子 | `23` |
| `--max_workers` | 平行 S4 worker 數 | `4` |
| `--out_folder` | 輸出資料夾 | 自動時間戳 |

## 🧪 快速 Smoke Run

```bash
./ms_final.sh -ns 1000 -r 42 -p smoke -g 40 -bo 0.25 -ro 0.20
python merge_s4_data_full.py --prefix smoke
python -c "import pandas as pd; p='merged_s4_shapes_smoke.csv'; d=pd.read_csv(p).rename(columns={'folder_key':'prefix','NQ':'nQ'}); d.to_csv(p,index=False)"
python three_stage_transmittance.py --preprocess --input_folder . --output_npz smoke.npz
python three_stage_transmittance.py --data_npz smoke.npz --num_epochs 5 --batch_size 128
```

## 🔬 研究背景

核心反向設計設定是：由跨晶化狀態的透射光譜，推回具 C4 對稱性的超表面幾何。當前的透射率流程假設：

- 每個樣本有 11 個晶化狀態（`c` 值，會在前處理時排序）
- 每個狀態含 100 個波長 bin（`T@...` 欄位）
- 最多 4 個 Q1 頂點，以 `(presence, x, y)` 編碼

本儲存庫也包含標準透射率管線以外的探索性分支（例如 `AVIRIS*`、`noise_experiment_*` 與 `archived/`）。

## 📚 引用

如果你使用了此儲存庫，或基於本工作進一步延伸，請引用：

```bibtex
@article{chen2025inverse,
  title={Inverse Design of Metasurface for Spectral Imaging},
  author={Chen, Rongzhou and Nie, Haitao and Zhu, Shuo and Zhao, Yaping and Wang, Chutian and Lam, Edmund Y},
  journal={arXiv preprint arXiv:2510.21924},
  year={2025}
}
```

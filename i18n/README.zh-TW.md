[![LazyingArt banner](https://github.com/lachlanchen/lachlanchen/raw/main/figs/banner.png)](https://github.com/lachlanchen/lachlanchen/blob/main/figs/banner.png)

[English](../README.md) · [العربية](README.ar.md) · [Español](README.es.md) · [Français](README.fr.md) · [日本語](README.ja.md) · [한국어](README.ko.md) · [Tiếng Việt](README.vi.md) · [中文 (简体)](README.zh-Hans.md) · [中文（繁體）](README.zh-Hant.md) · [Deutsch](README.de.md) · [Русский](README.ru.md)


# Inverse Design of Metasurface for Spectral Imaging

<p align="center">
  <img alt="Status" src="https://img.shields.io/badge/Status-Research%20Prototype-f59e0b?style=for-the-badge">
  <img alt="Python" src="https://img.shields.io/badge/Python-3.9-3776AB?style=for-the-badge&logo=python&logoColor=white">
  <img alt="PyTorch" src="https://img.shields.io/badge/PyTorch-2.x-EE4C2C?style=for-the-badge&logo=pytorch&logoColor=white">
  <img alt="RCWA" src="https://img.shields.io/badge/Simulator-S4%20RCWA-16a34a?style=for-the-badge">
  <img alt="Platform" src="https://img.shields.io/badge/Platform-Linux%20%2B%20Bash-6b7280?style=for-the-badge&logo=gnu-bash&logoColor=white">
</p>

這是一個以腳本為核心的研究型儲存庫（歷史上稱為 `inverse_metasurface`），用於光譜成像中的超表面反向設計。核心工作流程結合：

- 以物理為基礎的 RCWA 模擬（S4 + Lua）
- 資料彙整與形狀附加
- 三階段 PyTorch 學習（`shape -> spectrum`、`spectrum -> shape`、串接式微調）
- 定量與定性評估，並可選擇進行神經模型與 S4 一致性檢查

## ✨ 快速總覽

| 項目 | 說明 |
|---|---|
| 🎯 主要任務 | 由目標透射光譜反推出具 C4 對稱的超表面幾何 |
| 🔬 模擬器 | 由 shell 啟動腳本與 `.lua` 腳本呼叫 `../build/S4` |
| 🧠 學習流程 | Stage A `shape -> spectra`、Stage B `spectra -> shape`、Stage C `spectra -> shape -> spectra` |
| 📦 資料契約 | 合併 CSV（`T@...`、metadata、`vertices_str`）-> 壓縮 NPZ（`uids`、`spectra`、`shapes`） |
| 🧪 評估 | MSE 指標、各階段視覺化、可選擇重新以 S4 模擬驗證 |

## 🧭 端到端流程

1. 在 `results/` 產生模擬輸出，並在 `shapes/` 產生多邊形檔案。
2. 合併 S4 CSV 檔，並附加形狀頂點資訊。
3. 正規化合併後的欄位名稱，以相容訓練流程。
4. 將合併後 CSV 預處理為 NPZ 張量。
5. 訓練 Stage A/B/C 模型。
6. 評估 checkpoint 並進行行為視覺化。
7. 視需要以新的 S4 模擬，比對預測形狀的光譜。

## 🧱 儲存庫結構

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

## 🛠️ 先決條件

| 相依項 | 說明 |
|---|---|
| Linux + Bash | 啟動腳本以 shell 執行為目標 |
| Python 3.9 | 與 `iccp.yaml` 一致 |
| Conda | 建議用於可重現環境 |
| S4 binary | 預期位於 `../build/S4` |
| CUDA GPU（可選） | 可加速訓練與評估 |

## 🚀 安裝設定

### 1) Clone 並進入目錄

```bash
git clone <your-repo-url> inverse_metasurface
cd inverse_metasurface
```

### 2) 建立環境（建議）

```bash
conda env create -f iccp.yaml
conda activate iccp
```

替代方案（控制性較低）：

```bash
pip install -r pip_requirements.txt
```

### 3) 確認腳本預期的模擬器路徑

```bash
ls -l ../build/S4
```

### 4)（可選）讓啟動腳本可執行

```bash
chmod +x ms.sh ms_final.sh ms_resume.sh ms_resume_allargs.sh ms_resume_random_state.sh
```

## ▶️ 實務使用方式

### A) 產生 RCWA 模擬資料

簡易啟動：

```bash
./ms.sh -ns 10000 -r 12345
```

可參數化啟動：

```bash
./ms_final.sh \
  -ns 10000 \
  -r 12345 \
  -p myrun \
  -g 80 \
  -bo 0.35 \
  -ro 0.30
```

以續跑為主的啟動：

```bash
./ms_resume_allargs.sh \
  -ns 10000 \
  -r 12345 \
  -p myrun \
  -g 80 \
  -bo 0.35 \
  -ro 0.30
```

說明：
- 啟動腳本會平行執行 `NQ=1..4`。
- 腳本會用 `-t 32` 呼叫 `../build/S4`。

### B) 合併 S4 輸出並附加形狀頂點

```bash
python merge_s4_data_full.py --prefix myrun
# output: merged_s4_shapes_myrun.csv
```

### C) 正規化欄位以相容訓練流程

`merge_s4_data_full.py` 會輸出 `folder_key` 與 `NQ`，但訓練流程預期 `prefix` 與 `nQ`。

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

```bash
python three_stage_transmittance_evaluation.py \
  --model_dir outputs_three_stage_YYYYMMDD_HHMMSS \
  --data_npz preprocessed_t_data.npz \
  --sample_count 8
```

### G)（可選）神經模型與 S4 一致性檢查

```bash
python FilterShapeS4_Evaluator_Transmittance.py \
  --npz_file preprocessed_t_data.npz \
  --spec2shape_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageC/spec2shape_stageC.pt \
  --shape2spec_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageA/shape2spec_stageA.pt \
  --n_samples 4
```

## ⚙️ 重要 CLI 參數

### S4 啟動腳本（`ms_final.sh`、`ms_resume_allargs.sh`）

| 旗標 | 含義 | 預設值 |
|---|---|---|
| `-ns`, `--numshapes` | 要產生的形狀數量 | `100000` |
| `-r`, `--seed` | 隨機種子 | `88888` |
| `-p`, `--prefix` | 前綴／續跑識別鍵 | `""` |
| `-g`, `--numg` | 基底／網格參數 | `80` |
| `-bo`, `--baseouter` | 外邊界基準偏移 | `0.25` |
| `-ro`, `--randouter` | 外邊界隨機偏移 | `0.20` |

### 訓練（`three_stage_transmittance.py`）

| 旗標 | 含義 | 預設值 |
|---|---|---|
| `--preprocess` | 啟用預處理模式 | `False` |
| `--input_folder` | 含合併 CSV 的資料夾 | `""` |
| `--output_npz` | 輸出 NPZ 路徑 | `preprocessed_data.npz` |
| `--data_npz` | 訓練用 NPZ 資料集 | `""` |
| `--csv_file` | 未使用 NPZ 時的 CSV 備援 | `""` |
| `--test` | 測試模式 | `False` |
| `--num_epochs` | 訓練 epoch 數 | `10` |
| `--batch_size` | 批次大小 | `4096` |

### 評估（`three_stage_transmittance_evaluation.py`）

| 旗標 | 含義 | 預設值 |
|---|---|---|
| `--model_dir` | 含 `stageA/B/C` 的目錄 | required |
| `--data_npz` | NPZ 輸入 | `""` |
| `--csv_file` | CSV 輸入備援 | `""` |
| `--output_dir` | 覆寫輸出目錄 | 在 `model_dir` 下自動建立 |
| `--sample_count` | 視覺化樣本數 | `4` |
| `--seed` | 隨機種子 | `23` |
| `--font_scale` | 圖表字體縮放 | `1.0` |
| `--batch_size` | 評估批次大小 | `32` |
| `--plot_only` | 僅繪製訓練曲線 | `False` |

### S4 一致性評估器（`FilterShapeS4_Evaluator_Transmittance.py`）

| 旗標 | 含義 | 預設值 |
|---|---|---|
| `--npz_file` | 輸入 NPZ 檔案 | `preprocessed_t_data.npz` |
| `--spec2shape_ckpt` | Stage C checkpoint 路徑 | `outputs_three_stage_20250322_145925/stageC/spec2shape_stageC.pt` |
| `--shape2spec_ckpt` | Stage A checkpoint 路徑 | `outputs_three_stage_20250322_145925/stageA/shape2spec_stageA.pt` |
| `--n_samples` | 評估樣本數 | `4` |
| `--seed` | 隨機種子 | `23` |
| `--max_workers` | S4 工作執行緒數 | `4` |
| `--out_folder` | 輸出資料夾 | 自動時間戳 |

## 🧪 快速冒煙測試

```bash
./ms_final.sh -ns 1000 -r 42 -p smoke -g 40 -bo 0.25 -ro 0.20
python merge_s4_data_full.py --prefix smoke
python -c "import pandas as pd; p='merged_s4_shapes_smoke.csv'; d=pd.read_csv(p).rename(columns={'folder_key':'prefix','NQ':'nQ'}); d.to_csv(p,index=False)"
mkdir -p merged_csvs && mv merged_s4_shapes_smoke.csv merged_csvs/
python three_stage_transmittance.py --preprocess --input_folder merged_csvs --output_npz smoke.npz
python three_stage_transmittance.py --data_npz smoke.npz --num_epochs 5 --batch_size 128
```

## 🔬 研究背景

目前的反向設計設定，目標是從跨結晶狀態的透射資料中，學習回推具 C4 對稱的幾何形狀。當前透射流程假設：

- 每個 shape sample 有 11 列結晶度資料（以唯一 `shape_uid` 分組）
- 每個結晶狀態有 100 個波長分箱（`T@...` 欄位）
- 最多 4 個 Q1 控制點，編碼為 `4x3` 張量：`(presence, x, y)`
- 在形狀視覺化與一致性檢查中，使用 C4 對稱進行多邊形重建

此儲存庫也包含主要透射訓練流程之外的探索性分支（`AVIRIS*`、`noise_experiment*`、`archived/`）。

## 🧯 疑難排解

| 症狀 | 可能原因 | 修正方式 |
|---|---|---|
| `../build/S4: No such file or directory` | 預期相對路徑不存在 S4 binary | 在 `../build/S4` 建置或建立連結，或更新啟動腳本路徑 |
| `No transmission columns found` | CSV 缺少 `T@...` 欄位 | 重新檢查合併輸出格式 |
| `Must specify either --data_npz or --csv_file` | 缺少訓練／評估資料參數 | 明確提供其中一個輸入 |
| `No valid shapes => SHIFT->Q1->UpTo4` | `vertices_str` 無效／為空，或 Q1 篩選後無樣本 | 驗證形狀檔與合併輸出 |
| 使用 `--prefix` 時合併結果為空 | Prefix 與 `results/` 檔名不匹配 | 檢查精確檔名前綴後重新合併 |
| 找不到評估 checkpoint | 缺少 `stageA/B/C` checkpoint 檔案 | 確認 `--model_dir` 指向完整輸出資料夾 |

## 🤝 貢獻

歡迎各種貢獻，特別是可重現性、測試與文件品質相關改進。

建議流程：

1. 先建立 issue，說明範圍與預期行為。
2. 建立聚焦的分支。
3. 提交 pull request，附上可執行指令與輸出結果。
4. 變更內容盡可能聚焦單一工作流程。

## 📄 授權

在目前這份快照中，儲存庫根目錄尚未包含 `LICENSE` 檔案。請新增授權檔以明確定義使用與再散佈條款。

## 📚 Citation

若你使用此儲存庫或基於此工作延伸研究，請引用：

```bibtex
@article{chen2025inverse,
  title={Inverse Design of Metasurface for Spectral Imaging},
  author={Chen, Rongzhou and Nie, Haitao and Zhu, Shuo and Zhao, Yaping and Wang, Chutian and Lam, Edmund Y},
  journal={arXiv preprint arXiv:2510.21924},
  year={2025}
}
```


## ❤️ Support

| Donate | PayPal | Stripe |
| --- | --- | --- |
| [![Donate](https://camo.githubusercontent.com/24a4914f0b42c6f435f9e101621f1e52535b02c225764b2f6cc99416926004b7/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f446f6e6174652d4c617a79696e674172742d3045413545393f7374796c653d666f722d7468652d6261646765266c6f676f3d6b6f2d6669266c6f676f436f6c6f723d7768697465)](https://chat.lazying.art/donate) | [![PayPal](https://camo.githubusercontent.com/d0f57e8b016517a4b06961b24d0ca87d62fdba16e18bbdb6aba28e978dc0ea21/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f50617950616c2d526f6e677a686f754368656e2d3030343537433f7374796c653d666f722d7468652d6261646765266c6f676f3d70617970616c266c6f676f436f6c6f723d7768697465)](https://paypal.me/RongzhouChen) | [![Stripe](https://camo.githubusercontent.com/1152dfe04b6943afe3a8d2953676749603fb9f95e24088c92c97a01a897b4942/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f5374726970652d446f6e6174652d3633354246463f7374796c653d666f722d7468652d6261646765266c6f676f3d737472697065266c6f676f436f6c6f723d7768697465)](https://buy.stripe.com/aFadR8gIaflgfQV6T4fw400) |

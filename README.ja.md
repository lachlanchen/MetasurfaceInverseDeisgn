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

スペクトルイメージング向けメタサーフェス逆設計のための、スクリプト中心の研究用リポジトリ（過去には `inverse_metasurface` として参照）。このパイプラインは、**S4 ベースの RCWA シミュレーション**と、形状と光透過スペクトルの順問題・逆問題を扱う**3 段階の PyTorch ワークフロー**を組み合わせています。

## ✨ 概要

| 項目 | 詳細 |
|---|---|
| 🎯 目的 | 目標透過スペクトルから C4 対称メタサーフェス形状を予測 |
| 🔬 物理 | S4 (`../build/S4`) による RCWA シミュレーション |
| 🧠 学習パイプライン | Stage A `shape -> spectra`、Stage B `spectra -> shape`、Stage C `spectra -> shape -> spectra` |
| 📦 データ形式 | 統合 CSV（`T@...`、形状メタデータ） -> 圧縮 NPZ（`uids`、`spectra`、`shapes`） |
| 🧪 評価 | MSE 指標、定性プロット、任意で S4 再シミュレーション照合 |

## 🧭 エンドツーエンドのワークフロー

1. S4（`.lua` + シェルランチャー）でメタサーフェスの光学応答を生成。
2. 生シミュレーション CSV を統合し、多角形頂点情報を付与。
3. 統合 CSV を学習用 NPZ に変換。
4. 3 段階透過パイプラインを学習。
5. チェックポイントを評価し、Stage A/B/C の挙動を可視化。
6. 必要に応じて、ニューラル予測と新規 S4 シミュレーションを比較。

## 🧱 リポジトリ構成

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
├── results/                # S4 raw CSV outputs
├── shapes/                 # polygon vertex files used during merge
├── merged_csvs/            # merged CSV datasets
├── outputs_three_stage_*/  # training checkpoints and curves
├── partial_crys_data/      # crystallization-state optical tables
│
├── AVIRIS*/                # secondary hyperspectral experiments
├── noise_experiment_*/     # robustness/noise branches
└── archived/               # historical scripts and snapshots
```

## 🛠️ 前提条件

- Linux + Bash
- Conda（推奨）
- Python 3.9
- `../build/S4` に配置された S4 バイナリ
- 任意: 学習高速化のための CUDA 対応 GPU

## 🚀 セットアップ

```bash
git clone <your-repo-url> inverse_metasurface
cd inverse_metasurface

conda env create -f iccp.yaml
conda activate iccp

# Verify simulator path expected by scripts
ls -l ../build/S4
```

任意:

```bash
chmod +x ms.sh ms_final.sh ms_resume_allargs.sh
```

## ▶️ 実運用での使い方

### 1) RCWA データを生成

`ms_final.sh` と `ms_resume_allargs.sh` は、それぞれ 4 並列の S4 ジョブ（`NQ=1..4`）を起動します。

```bash
./ms_final.sh \
  -ns 10000 \
  -r 12345 \
  -p myrun \
  -g 80 \
  -bo 0.35 \
  -ro 0.30
```

補足:
- `ms.sh` はよりシンプルなランチャーパスです。
- ランチャーは `../build/S4` を前提とし、`-t 32` を使用します。

### 2) S4 出力 + 形状頂点を統合

```bash
python merge_s4_data_full.py --prefix myrun
# output: merged_s4_shapes_myrun.csv
```

### 3) 学習互換のため列名を正規化

`merge_s4_data_full.py` は `folder_key` / `NQ` を出力しますが、`three_stage_transmittance.py` は `prefix` / `nQ` を期待します。

```bash
python -c "import pandas as pd; p='merged_s4_shapes_myrun.csv'; df=pd.read_csv(p); df=df.rename(columns={'folder_key':'prefix','NQ':'nQ'}); df.to_csv(p,index=False)"
```

### 4) CSV を NPZ に前処理

```bash
mkdir -p merged_csvs
mv merged_s4_shapes_myrun.csv merged_csvs/

python three_stage_transmittance.py \
  --preprocess \
  --input_folder merged_csvs \
  --output_npz preprocessed_t_data.npz
```

### 5) 3 段階パイプラインを学習

```bash
python three_stage_transmittance.py \
  --data_npz preprocessed_t_data.npz \
  --num_epochs 100 \
  --batch_size 1024
```

想定される出力パターン:
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageA`
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageB`
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageC`

### 6) Stage A/B/C モデルを評価

```bash
python three_stage_transmittance_evaluation.py \
  --model_dir outputs_three_stage_YYYYMMDD_HHMMSS \
  --data_npz preprocessed_t_data.npz \
  --sample_count 8
```

### 7) 任意: ニューラル予測と S4 の整合性チェック

```bash
python FilterShapeS4_Evaluator_Transmittance.py \
  --npz_file preprocessed_t_data.npz \
  --spec2shape_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageC/spec2shape_stageC.pt \
  --shape2spec_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageA/shape2spec_stageA.pt \
  --n_samples 4
```

## ⚙️ 主な CLI オプション

### S4 ランチャー（`ms_final.sh`, `ms_resume_allargs.sh`）

| フラグ | 意味 | デフォルト |
|---|---|---|
| `-ns`, `--numshapes` | 生成する形状数 | `100000` |
| `-r`, `--seed` | 乱数シード | `88888` |
| `-p`, `--prefix` | プレフィックス/再開キー | `""` |
| `-g`, `--numg` | 基底/形状パラメータ | `80` |
| `-bo`, `--baseouter` | 外側境界オフセットの基準値 | `0.25` |
| `-ro`, `--randouter` | 外側オフセットのランダム成分 | `0.20` |

### 学習（`three_stage_transmittance.py`）

| フラグ | 目的 | デフォルト |
|---|---|---|
| `--preprocess` | 前処理モードを実行 | `False` |
| `--input_folder` | 統合 CSV があるフォルダ | `""` |
| `--output_npz` | 出力 NPZ パス | `preprocessed_data.npz` |
| `--data_npz` | 学習に使う NPZ | `""` |
| `--csv_file` | NPZ を使わない場合の CSV フォールバック | `""` |
| `--test` | テストモード（学習をスキップ） | `False` |
| `--num_epochs` | エポック数 | `10` |
| `--batch_size` | バッチサイズ | `4096` |

### 評価（`three_stage_transmittance_evaluation.py`）

| フラグ | 目的 | デフォルト |
|---|---|---|
| `--model_dir` | `stageA/B/C` を含むルートディレクトリ | 必須 |
| `--data_npz` | 評価用 NPZ 入力 | `""` |
| `--csv_file` | 評価用 CSV 入力 | `""` |
| `--output_dir` | 出力ディレクトリ上書き | `model_dir` 配下で自動 |
| `--sample_count` | 可視化サンプル数 | `4` |
| `--seed` | サンプリング用乱数シード | `23` |
| `--font_scale` | プロットのフォント倍率 | `1.0` |
| `--batch_size` | 評価バッチサイズ | `32` |
| `--plot_only` | 曲線プロットのみ実行 | `False` |

### S4 整合性評価（`FilterShapeS4_Evaluator_Transmittance.py`）

| フラグ | 目的 | デフォルト |
|---|---|---|
| `--npz_file` | 前処理済み NPZ ファイル | `preprocessed_t_data.npz` |
| `--spec2shape_ckpt` | Stage C チェックポイント | `outputs_three_stage_20250322_145925/stageC/spec2shape_stageC.pt` |
| `--shape2spec_ckpt` | Stage A チェックポイント | `outputs_three_stage_20250322_145925/stageA/shape2spec_stageA.pt` |
| `--n_samples` | 確認サンプル数 | `4` |
| `--seed` | 乱数シード | `23` |
| `--max_workers` | 並列 S4 ワーカー数 | `4` |
| `--out_folder` | 出力フォルダ | タイムスタンプで自動生成 |

## 🧪 クイックスモーク実行

```bash
./ms_final.sh -ns 1000 -r 42 -p smoke -g 40 -bo 0.25 -ro 0.20
python merge_s4_data_full.py --prefix smoke
python -c "import pandas as pd; p='merged_s4_shapes_smoke.csv'; d=pd.read_csv(p).rename(columns={'folder_key':'prefix','NQ':'nQ'}); d.to_csv(p,index=False)"
python three_stage_transmittance.py --preprocess --input_folder . --output_npz smoke.npz
python three_stage_transmittance.py --data_npz smoke.npz --num_epochs 5 --batch_size 128
```

## 🔬 研究コンテキスト

中核となる逆設計設定では、結晶化状態にまたがる透過スペクトルから C4 対称メタサーフェス形状を推定します。現在の透過パイプラインは次を前提としています。

- サンプルごとに 11 の結晶化状態（前処理時に `c` 値をソート）
- 状態ごとに 100 波長ビン（`T@...` 列）
- 最大 4 つの Q1 頂点を `(presence, x, y)` で符号化

本リポジトリには、標準の透過パイプライン以外にも、探索的な分岐（例: `AVIRIS*`、`noise_experiment_*`、`archived/`）が含まれます。

## 📚 引用

このリポジトリを利用または本研究を発展させる場合は、以下を引用してください。

```bibtex
@article{chen2025inverse,
  title={Inverse Design of Metasurface for Spectral Imaging},
  author={Chen, Rongzhou and Nie, Haitao and Zhu, Shuo and Zhao, Yaping and Wang, Chutian and Lam, Edmund Y},
  journal={arXiv preprint arXiv:2510.21924},
  year={2025}
}
```

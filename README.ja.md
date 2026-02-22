English | 中文繁體 | 中文简体 | 日本語 | 한국어 | Tiếng Việt | العربية | Français | Español | Deutsch | Русский

# inverse_metasurface

<div align="center">

[![Status](https://img.shields.io/badge/status-experimental-orange)](#-roadmap)
[![Python](https://img.shields.io/badge/python-3.9-blue)](#-prerequisites)
[![Platform](https://img.shields.io/badge/platform-linux-lightgrey)](#-prerequisites)
[![S4](https://img.shields.io/badge/S4-required-success)](#-prerequisites)
[![License](https://img.shields.io/badge/license-unset-lightgrey)](#-license)

</div>

**S4/RCWA シミュレーション**と**多段ニューラルモデル**を用いた、**逆メタサーフェス設計**のための研究ワークスペースです。

主な対応内容:
- C4 対称ポリゴンメタサーフェス向けの、高スループットな S4 データ生成。
- 学習可能な NPZ を作るためのデータ統合 + 前処理。
- 3 段階学習: **shape -> spectrum**、**spectrum -> shape**、**chain-tuned spectrum -> shape -> spectrum**。
- 評価と、任意での直接 **neural-vs-S4** 比較。

## 🌟 概要

| 領域 | このリポジトリが提供するもの |
|---|---|
| 物理シミュレーション | `nQ=1..4` で S4 を並列実行する Bash + Lua スクリプト |
| データセット処理 | ポリゴン頂点とピボットスペクトルを付与する統合スクリプト |
| ML パイプライン | 前処理 + 学習モードを持つ `three_stage_transmittance.py` |
| 評価 | メトリクス + 図を出力する `three_stage_transmittance_evaluation.py` |
| 研究ブランチ | AVIRIS/ハイパースペクトルおよびノイズ/圧縮実験 |

## 🧠 研究コンテキスト

このリポジトリはフォトニック・メタサーフェスの逆設計を対象としています。すなわち、目標スペクトルから形状を推定し（またはその逆を行い）ます。

ベースラインパイプラインで使う主な前提:
- Q1 点パラメータ化による C4 対称性。
- 11 個の結晶化状態（`c` in `[0.0, 1.0]`）。
- 各サンプルのスペクトルは `11 x 100` の透過率行として保存。
- 形状は最大 4 つの Q1 点で表現（`4 x 3` テンソル: presence, x, y）。

3 段階学習ロジック:
1. **Stage A**: shape -> spectrum (`shape2spec_stageA.pt`)
2. **Stage B**: spectrum -> shape (`spec2shape_stageB.pt`)
3. **Stage C**: spectrum -> shape -> spectrum 連鎖のファインチューニング (`spec2shape_stageC.pt`)

## 🗂 プロジェクト構成

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

## ✅ 前提条件

| 要件 | 補足 |
|---|---|
| OS | Linux（スクリプトは Bash + Linux パスを前提） |
| Python | 3.9（`iccp.yaml` に準拠） |
| 環境管理 | Conda 推奨 |
| S4 バイナリ | リポジトリルート相対で `../build/S4` を想定 |
| GPU（任意） | CUDA により学習/評価を高速化 |

## ⚙️ インストール

### 1) クローンして移動

```bash
git clone <your-repo-url> inverse_metasurface
cd inverse_metasurface
```

### 2) 環境作成

```bash
conda env create -f iccp.yaml
conda activate iccp
```

### 3) S4 パス確認

```bash
ls -l ../build/S4
```

このパスが存在しない場合は、S4 をそこへビルド/配置するか、スクリプト内のパスを調整してください。

## 🚀 クイックスタート（エンドツーエンド）

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

## 🧪 利用詳細

### A) S4 生成

最小シード実行:

```bash
./ms.sh -ns 10000 -r 88888
```

パラメータ指定実行:

```bash
./ms_final.sh \
  -ns 100000 \
  -r 88888 \
  -p iccpOv100kG80 \
  -g 80 \
  -bo 0.35 \
  -ro 0.30
```

再開形式の実行:

```bash
./ms_resume_allargs.sh \
  -ns 100000 \
  -r 88888 \
  -p iccpOv100kG80 \
  -g 80 \
  -bo 0.35 \
  -ro 0.30
```

### B) 生 S4 CSV の統合

```bash
python merge.py --prefix 20250123_155420
```

代替ユーティリティ:

```bash
python merge_s4_data_full.py --prefix 20250123_155420
```

### C) 統合 CSV の前処理 -> NPZ

```bash
python three_stage_transmittance.py --preprocess \
  --input_folder merged_csvs \
  --output_npz preprocessed_t_data.npz
```

### D) 学習

```bash
python three_stage_transmittance.py \
  --data_npz preprocessed_t_data.npz \
  --num_epochs 100 \
  --batch_size 1024
```

学習出力は次に保存されます:
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageA`
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageB`
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageC`

### E) 評価

```bash
python three_stage_transmittance_evaluation.py \
  --model_dir outputs_three_stage_YYYYMMDD_HHMMSS \
  --data_npz preprocessed_t_data.npz \
  --sample_count 8
```

生成物:
- `evaluation_metrics.csv`
- `metrics_summary.csv`
- 各ステージ可視化（`.png`, `.pdf`）
- 学習曲線プロット

### F) Neural vs direct S4 comparison（任意）

```bash
python FilterShapeS4_Evaluator_Transmittance.py \
  --npz_file preprocessed_t_data.npz \
  --spec2shape_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageC/spec2shape_stageC.pt \
  --shape2spec_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageA/shape2spec_stageA.pt \
  --n_samples 4
```

## 🔧 主要 CLI オプション

### S4 runner フラグ（`ms_final.sh`, `ms_resume_allargs.sh`）

| Flag | 意味 | デフォルト |
|---|---|---|
| `-ns`, `--numshapes` | 形状数 | `100000` |
| `-r`, `--seed` | 乱数シード | `88888` |
| `-p`, `--prefix` | 実行プレフィックス / 再開キー | empty |
| `-g`, `--numg` | ジオメトリ/基底パラメータ | `80` |
| `-bo`, `--baseouter` | 外側オフセット基準値 | `0.25` |
| `-ro`, `--randouter` | 外側ランダムオフセット | `0.20` |

### 学習フラグ（`three_stage_transmittance.py`）

| Flag | 意味 | デフォルト |
|---|---|---|
| `--preprocess` | 前処理モードを実行 | off |
| `--input_folder` | 入力 CSV フォルダ | empty |
| `--output_npz` | 出力 NPZ パス | `preprocessed_data.npz` |
| `--data_npz` | 学習/評価で使う NPZ | empty |
| `--csv_file` | 直接使用する CSV | empty |
| `--num_epochs` | 各ステージのエポック数 | `10` |
| `--batch_size` | バッチサイズ | `4096` |
| `--test` | テストモード（プレースホルダ） | off |

### 評価フラグ（`three_stage_transmittance_evaluation.py`）

| Flag | 意味 | デフォルト |
|---|---|---|
| `--model_dir` | 学習済み実行ディレクトリのルート | required |
| `--data_npz` | 評価用 NPZ | empty |
| `--csv_file` | 評価用 CSV | empty |
| `--output_dir` | 出力フォルダ | `model_dir/evaluation_<timestamp>` |
| `--sample_count` | 可視化サンプル数 | `4` |
| `--seed` | サンプリングシード | `23` |
| `--font_scale` | プロット文字スケール | `1.0` |
| `--batch_size` | 評価バッチサイズ | `32` |
| `--plot_only` | 曲線プロットのみ | off |

## 🧭 トラブルシューティング

| 症状 | 想定原因 | 対処 |
|---|---|---|
| `../build/S4: No such file or directory` | S4 バイナリのパス不一致 | `../build/S4` に S4 をビルド/配置、またはスクリプトを修正 |
| `Must specify either --data_npz or --csv_file` | データセット入力が未指定 | どちらかのフラグを明示指定 |
| `No matching CSVs found in 'results/'` | プレフィックス不一致 | プレフィックスと出力命名規則を確認 |
| 前処理済みレコードが極端に少ない/ゼロ | `T@...` または `vertices_str` 行の欠損/不正 | 統合 CSV スキーマと形状単位の行グルーピングを確認 |
| CUDA OOM | バッチが大きすぎる | `--batch_size` を下げる（例: `1024 -> 256`） |

## 🧱 開発メモ

- このリポジトリは実験中心であり、多くの生成物は意図的に追跡対象外です。
- 類似タスク向けに複数のスクリプト派生が存在します（`merge_*`, `aviris_*`, `noise_experiment_*`）。
- 逆メタサーフェスのベースラインワークフローは本 README に記載した経路です。
- 学習スクリプトはシード（`42`）を設定しますが、厳密な決定性はハードウェア/バックエンド依存です。
- 現時点で統一された CI + 完全自動テストスイートはありません。

## 🛣 ロードマップ

- クイックスモークテスト用のコンパクトな正準データセットを追加。
- 重複するパイプラインスクリプトを整理。
- 統合/前処理の整合性とチェックポイント読込可能性の検証を追加。
- lint + 学習/評価スモークテストの CI を追加。
- S4 のビルド/バージョン固定手順をより明示的に文書化。

## 🤝 コントリビューション

1. 機能ブランチを作成。
2. PR のスコープを小さく保つ（PR ごとに 1 つのパイプライン/実験課題）。
3. 実行可能コマンドと期待される出力パスを正確に記載。
4. 必要がない限り大きな生成物をコミットしない。
5. 再現性メモ（シード、データソース、チェックポイントパス）を追加。

## 📄 ライセンス

現在、このリポジトリには `LICENSE` ファイルが存在しません。

ライセンスファイルが追加されるまでは、再利用および再配布条件は**未確定**として扱ってください。

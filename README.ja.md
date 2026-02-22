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

[![Status](https://img.shields.io/badge/Status-Research%20Prototype-orange)](#-project-scope)
[![Python](https://img.shields.io/badge/Python-3.9-3776AB?logo=python&logoColor=white)](#-environment-setup)
[![Platform](https://img.shields.io/badge/Platform-Linux-2f2f2f?logo=linux&logoColor=white)](#-prerequisites)
[![RCWA](https://img.shields.io/badge/RCWA-S4%20Required-1f9d55)](#-prerequisites)
[![PyTorch](https://img.shields.io/badge/Framework-PyTorch-EE4C2C?logo=pytorch&logoColor=white)](#-training-and-evaluation)
[![License](https://img.shields.io/badge/License-Not%20Specified-lightgrey)](#-license)

**C4対称性**のもとで行う**メタサーフェス逆設計**のための、スクリプト中心の研究コードベースです。
主な構成は以下のとおりです。

- 🔬 S4/RCWAシミュレーション（`.lua` + Bashランチャー）
- 🧱 生スペクトル + 形状頂点からのデータ結合と前処理
- 🧠 3段階のPyTorch学習（`shape -> spectra`、`spectra -> shape`、連鎖チューニング）
- 📊 評価と可視化（ニューラル予測とS4の整合性チェックを含む）

## 📌 目次

- [プロジェクト範囲](#-project-scope)
- [研究背景](#-research-context)
- [リポジトリ構成](#-repository-layout)
- [前提条件](#-prerequisites)
- [環境セットアップ](#-environment-setup)
- [クイックスタート](#-quick-start)
- [エンドツーエンド・パイプライン](#-end-to-end-pipeline)
- [学習と評価](#-training-and-evaluation)
- [主要CLIオプション](#-key-cli-options)
- [トラブルシューティング](#-troubleshooting)
- [ロードマップ](#-roadmap)
- [引用](#-citation)
- [ライセンス](#-license)

## 🎯 プロジェクト範囲

このリポジトリは実験志向が強く、パッケージ化されたライブラリではなく実行可能スクリプトを中心に構成されています。
最も安定したワークフローは次のとおりです。

1. S4を実行して、生の光学出力を `results/`、形状を `shapes/` に生成する
2. 実行ごとのCSVを結合・ピボットして学習用の表形式データにする
3. 結合済みCSVを圧縮 `.npz` に変換する
4. 3段階の透過率パイプラインを学習する
5. チェックポイントを評価し、図や指標を出力する

## 🧪 研究背景

### 問題設定

中心となる逆設計タスクは、C4対称制約と部分結晶化スイープの条件下で、目標透過スペクトルからメタサーフェス形状を復元すること（およびその逆方向）です。

### 透過率パイプラインにおけるデータ前提

| 項目 | 値 |
|---|---|
| 形状あたりの結晶化状態数 | 11（`c` は `0.0` から `1.0`） |
| 状態あたりのスペクトルビン数 | 100 |
| サンプルごとのスペクトルテンソル | `11 x 100` |
| サンプルごとの形状テンソル | `4 x 3`（`[presence, x, y]`） |

### 3段階学習の目標

| ステージ | 方向 | 代表的なチェックポイント |
|---|---|---|
| A | `shape -> spectrum` | `stageA/shape2spec_stageA.pt` |
| B | `spectrum -> shape` | `stageB/spec2shape_stageB.pt` |
| C | `spectrum -> shape -> spectrum`（連鎖チューニング） | `stageC/spec2shape_stageC.pt` |

## 🗂️ リポジトリ構成

```text
.
├── ms.sh / ms_final.sh / ms_resume_allargs.sh
├── metasurface_seed.lua / metasurface_final.lua / metasurface_allargs_resume.lua
├── merge.py / merge_s4_data_full.py / merge_s4_data_local.py / merge_robust.py
├── three_stage_transmittance.py
├── three_stage_transmittance_evaluation.py
├── FilterShapeS4_Evaluator_Transmittance.py
├── partial_crys_data/
├── results/                          # raw S4 outputs
├── shapes/                           # generated polygon vertices
├── outputs_three_stage_*/            # checkpoints + training artifacts
├── AVIRIS*/ and aviris_*.py          # related hyperspectral experiments
├── commands.md / how_to_run.md
├── iccp.yaml
└── pip_requirements.txt
```

## 🧩 前提条件

| 依存関係 | 補足 |
|---|---|
| Linux + Bash | シェルスクリプトはLinux形式パスを前提 |
| Conda | 環境管理として推奨（`iccp.yaml`） |
| Python 3.9 | 学習・評価スクリプトの主要ランタイム |
| S4 binary | リポジトリルート基準で `../build/S4` にある想定 |
| CUDA（任意） | 学習と評価を高速化 |

## ⚙️ 環境セットアップ

```bash
git clone <repo-url> inverse_metasurface
cd inverse_metasurface

conda env create -f iccp.yaml
conda activate iccp

# verify S4 path expected by shell runners
ls -l ../build/S4
```

シェルランチャーに実行権限を付与する場合:

```bash
chmod +x ms.sh ms_final.sh ms_resume_allargs.sh ms_resume.sh
```

## 🚀 クイックスタート

すでに `preprocessed_t_data.npz` がある場合:

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

## 🔁 エンドツーエンド・パイプライン

### 1) S4データ生成

```bash
./ms_final.sh \
  -ns 10000 \
  -r 12345 \
  -p myrun \
  -g 80 \
  -bo 0.35 \
  -ro 0.30
```

### 2) S4出力 + 形状頂点の結合

```bash
python merge_s4_data_full.py --prefix myrun
# -> merged_s4_shapes_myrun.csv
```

### 3) 前処理向けにCSV列名を正規化

`three_stage_transmittance.py` の前処理は `prefix` と `nQ` を想定していますが、結合結果には `folder_key` と `NQ` が含まれる場合があります。

```bash
python -c "import pandas as pd; p='merged_s4_shapes_myrun.csv'; df=pd.read_csv(p); df=df.rename(columns={'folder_key':'prefix','NQ':'nQ'}); df.to_csv(p,index=False)"
```

### 4) CSV -> NPZ 前処理

```bash
mkdir -p merged_csvs
mv merged_s4_shapes_myrun.csv merged_csvs/

python three_stage_transmittance.py --preprocess \
  --input_folder merged_csvs \
  --output_npz preprocessed_t_data.npz
```

### 5) 3段階学習

```bash
python three_stage_transmittance.py \
  --data_npz preprocessed_t_data.npz \
  --num_epochs 100 \
  --batch_size 1024
```

### 6) 評価とプロット

```bash
python three_stage_transmittance_evaluation.py \
  --model_dir outputs_three_stage_YYYYMMDD_HHMMSS \
  --data_npz preprocessed_t_data.npz \
  --sample_count 8
```

### 7) 任意: ニューラル予測とS4の比較

```bash
python FilterShapeS4_Evaluator_Transmittance.py \
  --npz_file preprocessed_t_data.npz \
  --spec2shape_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageC/spec2shape_stageC.pt \
  --shape2spec_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageA/shape2spec_stageA.pt \
  --n_samples 4
```

## 🧠 学習と評価

### 主なスクリプト

| スクリプト | 目的 |
|---|---|
| `three_stage_transmittance.py` | 前処理 + A/B/Cステージ学習 |
| `three_stage_transmittance_evaluation.py` | チェックポイント評価、指標計算、プロット保存 |
| `FilterShapeS4_Evaluator_Transmittance.py` | 学習済み予測とS4挙動の比較 |

### 代表的な出力

| 成果物 | 保存先 |
|---|---|
| ステージチェックポイント | `outputs_three_stage_*/stageA|stageB|stageC/` |
| 評価図 | `outputs_three_stage_*/evaluation_<timestamp>/` |
| 指標CSV | `evaluation_metrics.csv`, `metrics_summary.csv` |

## 🛠️ 主要CLIオプション

### S4ランチャー（`ms_final.sh`, `ms_resume_allargs.sh`）

| フラグ | 意味 | 既定値 |
|---|---|---|
| `-ns`, `--numshapes` | 形状数 | `100000` |
| `-r`, `--seed` | 乱数シード | `88888` |
| `-p`, `--prefix` | 実行プレフィックス/再開キー | empty |
| `-g`, `--numg` | 幾何基底設定 | `80` |
| `-bo`, `--baseouter` | ベース外側オフセット | `0.25` |
| `-ro`, `--randouter` | ランダム外側オフセット | `0.20` |

### 学習（`three_stage_transmittance.py`）

| フラグ | 意味 | 既定値 |
|---|---|---|
| `--preprocess` | 前処理モードへ切替 | off |
| `--input_folder` | 結合CSVを含むフォルダ | `""` |
| `--output_npz` | 前処理出力ファイル | `preprocessed_data.npz` |
| `--data_npz` | 学習用NPZ入力 | `""` |
| `--csv_file` | 直接CSV学習入力 | `""` |
| `--test` | テストモード切替 | off |
| `--num_epochs` | ステージごとのエポック数 | `10` |
| `--batch_size` | バッチサイズ | `4096` |

### 評価（`three_stage_transmittance_evaluation.py`）

| フラグ | 意味 | 既定値 |
|---|---|---|
| `--model_dir` | 学習済みrunディレクトリ（必須） | - |
| `--data_npz` | 評価用NPZ入力 | `""` |
| `--csv_file` | 評価用CSV入力 | `""` |
| `--output_dir` | カスタム出力ディレクトリ | auto |
| `--sample_count` | 可視化サンプル数 | `4` |
| `--seed` | サンプル選択用乱数シード | `23` |
| `--font_scale` | プロット文字サイズ倍率 | `1.0` |
| `--batch_size` | eval dataloader のバッチサイズ | `32` |
| `--plot_only` | プロットのみ再生成 | off |

## 🧯 トラブルシューティング

| 症状 | 主な原因 | 対処 |
|---|---|---|
| `../build/S4: No such file or directory` | S4バイナリが想定の相対パスにない | `../build/S4` に配置/ビルドするかスクリプトを修正 |
| `Must specify either --data_npz or --csv_file` | 学習/評価データセット引数が不足 | データセット入力を1つだけ指定 |
| `No transmission columns found` | 結合CSVに `T@...` 列がない | merge/pivot を再実行しヘッダを確認 |
| `KeyError: 'prefix'` in preprocess | 結合出力が `folder_key`/`NQ` のまま | 前処理前に `prefix`/`nQ` に列名変更 |
| GPU OOM | バッチサイズ過大 | `--batch_size` を下げる |
| Missing checkpoints during eval | ステージチェックポイントが無い/パス誤り | 指定 `--model_dir` 配下の stageA/B/C を確認 |

## 🧭 ロードマップ

- mergeスクリプト間で結合CSVスキーマを標準化（`prefix`, `nQ` 命名）
- merge/前処理/チェックポイント読込の自動テスト追加
- フルパイプラインを統合実行できる単一CLIエントリポイントを提供
- 再現性向上のためデータセット/runマニフェストを追加
- 明示的なオープンソースライセンスを追加

## 📚 引用

このリポジトリが研究に役立った場合は、以下を引用してください。

```bibtex
@article{chen2025inverse,
  title={Inverse Design of Metasurface for Spectral Imaging},
  author={Chen, Rongzhou and Nie, Haitao and Zhu, Shuo and Zhao, Yaping and Wang, Chutian and Lam, Edmund Y},
  journal={arXiv preprint arXiv:2510.21924},
  year={2025}
}
```

## 📄 ライセンス

現在、このリポジトリには `LICENSE` ファイルがありません。したがって、ライセンスが追加されるまでは利用および再配布の権利は未規定です。

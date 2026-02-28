[English](../README.md) · [العربية](README.ar.md) · [Español](README.es.md) · [Français](README.fr.md) · [日本語](README.ja.md) · [한국어](README.ko.md) · [Tiếng Việt](README.vi.md) · [中文 (简体)](README.zh-Hans.md) · [中文（繁體）](README.zh-Hant.md) · [Deutsch](README.de.md) · [Русский](README.ru.md)


# スペクトルイメージングのためのメタサーフェス逆設計

<p align="center">
  <img alt="Status" src="https://img.shields.io/badge/Status-Research%20Prototype-f59e0b?style=for-the-badge">
  <img alt="Python" src="https://img.shields.io/badge/Python-3.9-3776AB?style=for-the-badge&logo=python&logoColor=white">
  <img alt="PyTorch" src="https://img.shields.io/badge/PyTorch-2.x-EE4C2C?style=for-the-badge&logo=pytorch&logoColor=white">
  <img alt="Simulator" src="https://img.shields.io/badge/Simulator-S4%20RCWA-16a34a?style=for-the-badge">
  <img alt="Platform" src="https://img.shields.io/badge/Platform-Linux%20%2B%20Bash-6b7280?style=for-the-badge&logo=gnu-bash&logoColor=white">
  <img alt="ArXiv" src="https://img.shields.io/badge/arXiv-2510.21924-b31b1b?style=for-the-badge&logo=arxiv&logoColor=white">
</p>

スペクトルイメージングにおけるメタサーフェス逆設計のための、スクリプト中心の研究リポジトリです（過去には `inverse_metasurface` と呼称）。

コアワークフローは次を結合しています。
- 物理ベースの RCWA シミュレーション（`S4` + Lua）
- データ統合と形状情報の付与
- 3 段階の PyTorch 学習（`shape -> spectrum`、`spectrum -> shape`、連結ファインチューニング）
- 定量・定性評価（必要に応じて neural-vs-S4 整合性チェック）

> [!IMPORTANT]
> 既存プロジェクトのスクリプト/ドキュメントにおける正規の挙動とコマンドを保持しています。過去の参照先が現時点で存在しないファイルを指す場合でも、互換性のために明示的な注記付きで意図的に残しています。

## 📑 目次

- [✨ 概要](#-概要)
- [🌍 国際化 (i18n)](#-国際化-i18n)
- [✨ 特徴](#-特徴)
- [🧭 エンドツーエンドのワークフロー](#-エンドツーエンドのワークフロー)
- [🧱 プロジェクト構成](#-プロジェクト構成)
- [🛠️ 前提条件](#️-前提条件)
- [🚀 インストール](#-インストール)
- [▶️ 使い方](#️-使い方)
- [⚙️ 設定](#️-設定)
- [🧪 例](#-例)
- [🔬 研究コンテキスト](#-研究コンテキスト)
- [🧑‍💻 開発ノート](#-開発ノート)
- [🧯 トラブルシューティング](#-トラブルシューティング)
- [🗺️ ロードマップ](#️-ロードマップ)
- [🤝 コントリビューション](#-コントリビューション)
- [📄 ライセンス](#-ライセンス)
- [📚 引用](#-引用)

## ✨ 概要

| 項目 | 詳細 |
|---|---|
| 🎯 主タスク | 目標透過スペクトルから C4 対称メタサーフェス形状を推定 |
| 🔬 シミュレータ | シェルランチャーおよび `.lua` スクリプトから呼び出される `../build/S4` |
| 🧠 学習パイプライン | Stage A `shape -> spectra`、Stage B `spectra -> shape`、Stage C `spectra -> shape -> spectra` |
| 📦 データ契約 | 統合 CSV（`T@...`、メタデータ、`vertices_str`）-> 圧縮 NPZ（`uids`、`spectra`、`shapes`） |
| 🧪 評価 | MSE 指標、各ステージ可視化、必要に応じた S4 再シミュレーション |
| 🌐 i18n 状態 | ルート階層の多言語 README + 既存の `i18n/` ディレクトリ |

## 🌍 国際化 (i18n)

- 多言語 README はリポジトリルートの `README.<lang>.md` として管理されています。
- このリポジトリスナップショットには `i18n/` ディレクトリが存在します。
- このファイルは重複した言語バーを避けるため、先頭に 1 行だけ言語オプションを配置しています。
- `README.en.md` もリポジトリ内に存在しますが、今回の更新ではこの `README.md` を正規ベースとして扱います。

## ✨ 特徴

- S4 シミュレーション出力から逆問題モデル学習までのエンドツーエンド逆設計フロー。
- C4 対称ポリゴンのパラメータ化と Q1 点エンコーディング（`4x3`: `presence, x, y`）。
- 1 本のスクリプト（`three_stage_transmittance.py`）で 3 段階モデルを学習。
- スペクトル精度を保持しつつ形状ごとの頂点情報を付与するマージツール。
- 学習予測と新規 S4 シミュレーションを比較できるオプション評価器。
- 幅広い探索ブランチ（AVIRIS、SWIR/noise、GSST、archived/deprecated 変種）。

## 🧭 エンドツーエンドのワークフロー

1. `results/` にシミュレーション出力、`shapes/` にポリゴンファイルを生成。
2. S4 の CSV をマージし、形状頂点を付与。
3. 学習互換性のためにマージ後カラム名を正規化。
4. マージ済み CSV を NPZ テンソルへ前処理。
5. Stage A/B/C モデルを学習。
6. チェックポイント評価と挙動可視化を実施。
7. 必要に応じて、予測形状スペクトルを新規 S4 実行結果と比較。

## 🧱 プロジェクト構成

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

## 🛠️ 前提条件

| 依存関係 | 注記 |
|---|---|
| Linux + Bash | ランチャースクリプトはシェル実行を前提 |
| Python 3.9 | `iccp.yaml`（`python=3.9.18`）と整合 |
| Conda | 再現性のため推奨 |
| S4 binary | `../build/S4` に配置されている想定 |
| CUDA GPU（任意） | 学習/評価を高速化 |

## 🚀 インストール

### 1) クローンして移動

```bash
git clone <your-repo-url> inverse_metasurface
cd inverse_metasurface
```

### 2) 環境作成（推奨）

```bash
conda env create -f iccp.yaml
conda activate iccp
```

補足（代替）:

```bash
# Historical README reference (file may be absent in this snapshot)
pip install -r pip_requirements.txt
```

### 3) スクリプトが想定するシミュレータパスを確認

```bash
ls -l ../build/S4
```

### 4) （任意）ランチャーに実行権限を付与

```bash
chmod +x ms.sh ms_final.sh ms_resume.sh ms_resume_allargs.sh ms_resume_random_state.sh ms_resume_random_state_nir.sh
```

## ▶️ 使い方

### A) RCWA シミュレーションデータを生成

シンプルなランチャー:

```bash
./ms.sh -ns 10000 -r 12345
```

パラメータ指定ランチャー:

```bash
./ms_final.sh \
  -ns 10000 \
  -r 12345 \
  -p myrun \
  -g 80 \
  -bo 0.35 \
  -ro 0.30
```

再開向けランチャー:

```bash
./ms_resume_allargs.sh \
  -ns 10000 \
  -r 12345 \
  -p myrun \
  -g 80 \
  -bo 0.35 \
  -ro 0.30
```

追加の resume/random-state 例（コマンドドキュメントより）:

```bash
./ms_resume_random_state.sh \
  -p iccp100kG20Ov \
  -r 88888 \
  -g 20 \
  -bo 0.35 \
  -ro 0.3 \
  -ns 100000
```

注記:
- ランチャーは `NQ=1..4` を並列実行します。
- スクリプトは `../build/S4` を `-t 32` 付きで呼び出します。

### B) S4 出力をマージし、形状頂点を付与

```bash
python merge_s4_data_full.py --prefix myrun
# output: merged_s4_shapes_myrun.csv
```

### C) 学習互換性のためにカラムを正規化

`merge_s4_data_full.py` は `folder_key` と `NQ` を出力しますが、学習側は `prefix` と `nQ` を想定しています。

```bash
python -c "import pandas as pd; p='merged_s4_shapes_myrun.csv'; df=pd.read_csv(p); df=df.rename(columns={'folder_key':'prefix','NQ':'nQ'}); df.to_csv(p,index=False)"
```

### D) CSV -> NPZ を前処理

```bash
mkdir -p merged_csvs
mv merged_s4_shapes_myrun.csv merged_csvs/

python three_stage_transmittance.py \
  --preprocess \
  --input_folder merged_csvs \
  --output_npz preprocessed_t_data.npz
```

### E) Stage A/B/C を学習

```bash
python three_stage_transmittance.py \
  --data_npz preprocessed_t_data.npz \
  --num_epochs 100 \
  --batch_size 1024
```

出力先:
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageA`
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageB`
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageC`

### F) 学習済みモデルを評価

過去 README のコマンド（従来ドキュメント互換のためスクリプト名を保持）:

```bash
python three_stage_transmittance_evaluation.py \
  --model_dir outputs_three_stage_YYYYMMDD_HHMMSS \
  --data_npz preprocessed_t_data.npz \
  --sample_count 8
```

リポジトリ状態に関する注記: このスナップショットには `three_stage_transmittance_evaluation.py` は存在しません。利用可能な評価機能として `FilterShapeS4_Evaluator_Transmittance.py` を使用してください。

### G) （任意）neural-vs-S4 整合性チェック

```bash
python FilterShapeS4_Evaluator_Transmittance.py \
  --npz_file preprocessed_t_data.npz \
  --spec2shape_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageC/spec2shape_stageC.pt \
  --shape2spec_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageA/shape2spec_stageA.pt \
  --n_samples 4
```

## ⚙️ 設定

### S4 ランチャー（`ms_final.sh`, `ms_resume_allargs.sh`）

| Flag | 意味 | Default |
|---|---|---|
| `-ns`, `--numshapes` | 生成する形状数 | `100000` |
| `-r`, `--seed` | 乱数シード | `88888` |
| `-p`, `--prefix` | プレフィックス/再開キー | `""` |
| `-g`, `--numg` | 基底/グリッドパラメータ | `80` |
| `-bo`, `--baseouter` | 外周境界の基準オフセット | `0.25` |
| `-ro`, `--randouter` | 外周境界のランダムオフセット | `0.20` |

### 学習（`three_stage_transmittance.py`）

| Flag | 意味 | Default |
|---|---|---|
| `--preprocess` | 前処理モードを実行 | `False` |
| `--input_folder` | マージ済み CSV を含むフォルダ | `""` |
| `--output_npz` | 出力 NPZ パス | `preprocessed_data.npz` |
| `--data_npz` | 学習用 NPZ データセット | `""` |
| `--csv_file` | NPZ 不使用時の CSV フォールバック | `""` |
| `--test` | テストモード | `False` |
| `--num_epochs` | 学習エポック数 | `10` |
| `--batch_size` | バッチサイズ | `4096` |

### 過去評価設定（`three_stage_transmittance_evaluation.py`）

| Flag | 意味 | Default |
|---|---|---|
| `--model_dir` | `stageA/B/C` を含むディレクトリ | required |
| `--data_npz` | NPZ 入力 | `""` |
| `--csv_file` | CSV 入力フォールバック | `""` |
| `--output_dir` | 出力ディレクトリ上書き | `model_dir` 配下で自動 |
| `--sample_count` | 可視化サンプル数 | `4` |
| `--seed` | 乱数シード | `23` |
| `--font_scale` | プロット文字スケール | `1.0` |
| `--batch_size` | 評価バッチサイズ | `32` |
| `--plot_only` | 学習曲線のみ描画 | `False` |

### S4 整合性評価器（`FilterShapeS4_Evaluator_Transmittance.py`）

| Flag | 意味 | Default |
|---|---|---|
| `--npz_file` | 入力 NPZ ファイル | `preprocessed_t_data.npz` |
| `--spec2shape_ckpt` | Stage C チェックポイントパス | `outputs_three_stage_20250322_145925/stageC/spec2shape_stageC.pt` |
| `--shape2spec_ckpt` | Stage A チェックポイントパス | `outputs_three_stage_20250322_145925/stageA/shape2spec_stageA.pt` |
| `--n_samples` | 評価サンプル数 | `4` |
| `--seed` | 乱数シード | `23` |
| `--max_workers` | S4 ワーカースレッド数 | `4` |
| `--out_folder` | 出力ディレクトリ | タイムスタンプで自動 |

## 🧪 例

### スモークラン

```bash
./ms_final.sh -ns 1000 -r 42 -p smoke -g 40 -bo 0.25 -ro 0.20
python merge_s4_data_full.py --prefix smoke
python -c "import pandas as pd; p='merged_s4_shapes_smoke.csv'; d=pd.read_csv(p).rename(columns={'folder_key':'prefix','NQ':'nQ'}); d.to_csv(p,index=False)"
mkdir -p merged_csvs && mv merged_s4_shapes_smoke.csv merged_csvs/
python three_stage_transmittance.py --preprocess --input_folder merged_csvs --output_npz smoke.npz
python three_stage_transmittance.py --data_npz smoke.npz --num_epochs 5 --batch_size 128
```

### 実行プロファイル例（`commands.md` / `commands_updated.md` より）

```bash
# no overlap, G=40
./ms_resume_allargs.sh -ns 10000 -r 12345 -p fast_without_overlap -g 40 -bo 0.25 -ro 0.2

# overlap, G=80
./ms_resume_allargs.sh -ns 10000 -r 12345 -p overlap -g 80 -bo 0.35 -ro 0.3

# large run, overlap, G=80
./ms_resume_allargs.sh -ns 100000 -r 12345 -p more_basis_overlap -g 80 -bo 0.35 -ro 0.3
```

## 🔬 研究コンテキスト

現在の逆設計設定は、結晶化状態にわたる透過率データから C4 対称形状を復元する学習を行います。現行の透過率パイプラインは次を前提とします。

- 1 形状サンプルあたり 11 行の結晶化状態（`shape_uid` 単位でグルーピング）
- 各結晶化状態につき 100 波長ビン（`T@...` カラム）
- 最大 4 つの Q1 制御点を `4x3` テンソル `(presence, x, y)` としてエンコード
- 形状可視化および整合性チェックのための C4 対称ポリゴン再構成

リポジトリには、主要な透過率学習パス以外に探索ブランチ（`AVIRIS*`, `noise_experiment*`, `archived/`）も含まれます。

## 🧑‍💻 開発ノート

- これは Python パッケージではなく、スクリプト中心の研究リポジトリです。
- コアスクリプトは相対パス（特に `../build/S4`, `results/`, `shapes/`）を前提にしています。
- `.gitignore` は多くの生成実験成果物（`*.csv`, `*.npz`, `*.pt`, 実行フォルダ）を除外しています。
- 過去ドキュメントにある一部ファイル/ディレクトリは、このスナップショットでは欠けています。これらの参照は互換性のため注記付きで意図的に保持しています。
- macOS のサイドカーファイル（`._*`）が存在し、機能しないメタデータ成果物の可能性があります。

## 🧯 トラブルシューティング

| 症状 | 想定原因 | 対処 |
|---|---|---|
| `../build/S4: No such file or directory` | 期待される相対パスに S4 バイナリがない | `../build/S4` に S4 をビルド/リンクするか、ランチャーパスを更新 |
| `No transmission columns found` | CSV に `T@...` カラムがない | マージ出力形式を再確認 |
| `Must specify either --data_npz or --csv_file` | 学習/評価データ引数が未指定 | いずれかの入力を明示 |
| `No valid shapes => SHIFT->Q1->UpTo4` | `vertices_str` が無効/空、または Q1 フィルタで全サンプル除外 | 形状ファイルとマージ出力を検証 |
| `Empty merge output for --prefix` | `results/` 内ファイルと prefix が一致しない | ファイル名プレフィックスを正確に確認して再実行 |
| Evaluation checkpoint missing | `stageA/B/C` チェックポイントが不足 | `--model_dir` が完全な出力フォルダを指しているか確認 |
| `three_stage_transmittance_evaluation.py` not found | 過去ドキュメント参照のスクリプトが現状存在しない | `FilterShapeS4_Evaluator_Transmittance.py` を使うか、過去コミットから復元 |

## 🗺️ ロードマップ

- 明示的なデータバージョニングマニフェストと固定 run config により再現性を向上。
- transmittance、AVIRIS、noise 各ブランチの正規エントリポイントを整理。
- 前処理と最小学習 1 エポックに対する自動スモークテストを追加。
- 出力フォルダと正確なコマンドラインを紐づける実験レジストリを明確化。
- 多言語 README 同期ワークフロー（ルート言語ファイルおよび `i18n/`）を拡張。

## 🤝 コントリビューション

再現性、テスト、ドキュメント品質の改善に関する貢献を歓迎します。

推奨プロセス:

1. スコープと期待挙動を issue で共有。
2. 目的を絞ったブランチを作成。
3. 実行可能なコマンドと出力を添えて pull request を提出。
4. 可能な限り 1 つのワークフローにスコープを限定。

## 📄 ライセンス

このスナップショットのリポジトリルートには現在 `LICENSE` ファイルがありません。利用および再配布条件を定義するため、追加してください。

## 📚 引用

このリポジトリを利用する、または本研究を基に発展させる場合は、以下を引用してください。

```bibtex
@article{chen2025inverse,
  title={Inverse Design of Metasurface for Spectral Imaging},
  author={Chen, Rongzhou and Nie, Haitao and Zhu, Shuo and Zhao, Yaping and Wang, Chutian and Lam, Edmund Y},
  journal={arXiv preprint arXiv:2510.21924},
  year={2025}
}
```

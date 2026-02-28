[English](../README.md) · [العربية](README.ar.md) · [Español](README.es.md) · [Français](README.fr.md) · [日本語](README.ja.md) · [한국어](README.ko.md) · [Tiếng Việt](README.vi.md) · [中文 (简体)](README.zh-Hans.md) · [中文（繁體）](README.zh-Hant.md) · [Deutsch](README.de.md) · [Русский](README.ru.md)


[![LazyingArt banner](https://github.com/lachlanchen/lachlanchen/raw/main/figs/banner.png)](https://github.com/lachlanchen/lachlanchen/blob/main/figs/banner.png)

# スペクトルイメージングのためのメタサーフェス逆設計

<p align="center">
  <img alt="Status" src="https://img.shields.io/badge/Status-Research%20Prototype-f59e0b?style=for-the-badge">
  <img alt="Python" src="https://img.shields.io/badge/Python-3.9-3776AB?style=for-the-badge&logo=python&logoColor=white">
  <img alt="PyTorch" src="https://img.shields.io/badge/PyTorch-2.x-EE4C2C?style=for-the-badge&logo=pytorch&logoColor=white">
  <img alt="Simulator" src="https://img.shields.io/badge/Simulator-S4%20RCWA-16a34a?style=for-the-badge">
  <img alt="Platform" src="https://img.shields.io/badge/Platform-Linux%20%2B%20Bash-6b7280?style=for-the-badge&logo=gnu-bash&logoColor=white">
  <img alt="ArXiv" src="https://img.shields.io/badge/arXiv-2510.21924-b31b1b?style=for-the-badge&logo=arxiv&logoColor=white">
</p>

スペクトルイメージング向けのメタサーフェス逆設計を行う、スクリプト中心の研究リポジトリです（過去には `inverse_metasurface` と呼ばれていました）。

中核ワークフローは以下で構成されます。
- 物理ベースの RCWA シミュレーション（`S4` + Lua）
- データ統合と形状情報の付与
- 3 段階の PyTorch 学習（`shape -> spectrum`、`spectrum -> shape`、連結ファインチューニング）
- 定量・定性評価。必要に応じてニューラル予測と S4 の整合性を再確認

> [!IMPORTANT]
> 既存のプロジェクトスクリプト/ドキュメントで保証されている挙動とコマンドは維持しています。履歴参照先がこのスナップショットでは欠落している場合でも、互換性のために注釈付きで意図的に残しています。

## 📑 目次

- [🌟 Snapshot](#-snapshot)
- [✨ At a Glance](#-at-a-glance)
- [🌍 国際化 (i18n)](#-internationalization-i18n)
- [✨ 特徴](#-features)
- [🧭 エンドツーエンドワークフロー](#-end-to-end-workflow)
- [🧱 プロジェクト構成](#-project-structure)
- [🛠️ 前提条件](#️-prerequisites)
- [🚀 インストール](#-installation)
- [▶️ 使い方](#️-usage)
- [⚙️ 設定](#️-configuration)
- [🧪 例](#-examples)
- [🔬 研究コンテキスト](#-research-context)
- [🧑‍💻 開発ノート](#-development-notes)
- [🧯 トラブルシューティング](#-troubleshooting)
- [🗺️ ロードマップ](#️-roadmap)
- [🤝 コントリビューション](#-contribution)
- [📄 ライセンス](#-license)
- [📚 引用](#-citation)

## 🌟 Snapshot

| 項目 | 内容 |
|---|---|
| 🧠 目標 | 結晶化スペクトルから C4 対称メタサーフェス形状を逆推定 |
| 🔧 コア技術 | S4 RCWA（`Lua`） + PyTorch 学習 + 任意の形状→スペクトル再検証 |
| 🧪 データパイプライン | CSV 統合/形状頂点付与 → NPZ（`uids`, `spectra`, `shapes`） |
| 🚀 実装段階 | 研究プロトタイプ。履歴参照と互換性を保つため、スクリプトとドキュメントを旧仕様ベースで整備 |

## ✨ At a Glance

| 項目 | 詳細 |
|---|---|
| 🎯 主タスク | 目標透過率スペクトルから C4 対称メタサーフェス形状を復元 |
| 🔬 シミュレータ | シェル起動スクリプトと `.lua` により呼び出される `../build/S4` |
| 🧠 学習パイプライン | Stage A `shape -> spectra`、Stage B `spectra -> shape`、Stage C `spectra -> shape -> spectra` |
| 📦 データ規約 | 統合 CSV（`T@...`, メタ情報, `vertices_str`）→ 圧縮 NPZ（`uids`, `spectra`, `shapes`） |
| 🧪 評価 | MSE 指標、各ステージ可視化、任意の最新 S4 再シミュレーション |
| 🌐 i18n 状態 | ルートの多言語 README と既存 `i18n/` ディレクトリ |

## 🌍 国際化 (i18n)

- 多言語 README は `README.<lang>.md` としてルートに配置されています。
- このリポジトリには `i18n/` ディレクトリが存在します。
- このファイルは重複した言語リンクを避けるため、先頭に言語行を 1 行だけ置いています。
- `README.en.md` も存在します。今回の更新では `README.md` をベースにしています。

## ✨ 特徴

- S4 シミュレーション出力から逆問題モデルの学習までをつなぐ、エンドツーエンドの逆設計フロー。
- C4 対称ポリゴンのパラメータ化と Q1 点エンコーディング（`4x3`: `presence, x, y`）。
- 1 つのスクリプト（`three_stage_transmittance.py`）で 3 段階のモデル学習。
- スペクトル精度を維持し、形状ごとの頂点を付与するマージ処理。
- ニューラル予測と新規 S4 シミュレーションを比較できる任意評価機能。
- AVIRIS、SWIR/noise、GSST、archived/deprecated 系など、探索ブランチを広く保持。

## 🧭 エンドツーエンドワークフロー

1. `results/` にシミュレーション結果を生成し、`shapes/` にポリゴンファイルを作成。
2. S4 の CSV をマージして形状頂点を付与。
3. 学習互換のために列名を正規化。
4. マージ済み CSV を NPZ テンソルへ前処理。
5. Stage A/B/C モデルを学習。
6. チェックポイントを評価し、挙動を可視化。
7. 必要に応じて、予測形状スペクトルを新規 S4 実行結果と比較。

## 🧱 プロジェクト構造

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
| Python 3.9 | `iccp.yaml`（`python=3.9.18`）と一致 |
| Conda | 再現性向上のため推奨 |
| S4 バイナリ | `../build/S4` を前提 |
| CUDA GPU（任意） | 学習・評価を高速化 |

## 🚀 インストール

### 1) クローンして移動

```bash
git clone <your-repo-url> inverse_metasurface
cd inverse_metasurface
```

### 2) 環境の作成（推奨）

```bash
conda env create -f iccp.yaml
conda activate iccp
```

補足:

```bash
# Historical README reference (file may be absent in this snapshot)
pip install -r pip_requirements.txt
```

### 3) スクリプト想定のシミュレーターパスを確認

```bash
ls -l ../build/S4
```

### 4) （任意）ランチャーを実行可能にする

```bash
chmod +x ms.sh ms_final.sh ms_resume.sh ms_resume_allargs.sh ms_resume_random_state.sh ms_resume_random_state_nir.sh
```

## ▶️ 使い方

### A) RCWA シミュレーションデータの生成

シンプルランチャー:

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

追加 resume/random-state 例（コマンドドキュメントより）:

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

### B) S4 出力のマージと形状頂点の付与

```bash
python merge_s4_data_full.py --prefix myrun
# output: merged_s4_shapes_myrun.csv
```

### C) 学習互換のための列名正規化

`merge_s4_data_full.py` は `folder_key` と `NQ` を出力しますが、学習側は `prefix` と `nQ` を想定します。

```bash
python -c "import pandas as pd; p='merged_s4_shapes_myrun.csv'; df=pd.read_csv(p); df=df.rename(columns={'folder_key':'prefix','NQ':'nQ'}); df.to_csv(p,index=False)"
```

### D) CSV -> NPZ の前処理

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

### F) 学習済みモデルの評価

履歴 README のコマンド（互換性確保のためスクリプト名は保持）:

```bash
python three_stage_transmittance_evaluation.py \
  --model_dir outputs_three_stage_YYYYMMDD_HHMMSS \
  --data_npz preprocessed_t_data.npz \
  --sample_count 8
```

リポジトリ状態の注意:
`three_stage_transmittance_evaluation.py` はこのスナップショットでは存在しません。利用できる評価機能として `FilterShapeS4_Evaluator_Transmittance.py` を使用してください。

### G) 任意の neural-vs-S4 整合性チェック

```bash
python FilterShapeS4_Evaluator_Transmittance.py \
  --npz_file preprocessed_t_data.npz \
  --spec2shape_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageC/spec2shape_stageC.pt \
  --shape2spec_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageA/shape2spec_stageA.pt \
  --n_samples 4
```

## ⚙️ 設定

### S4 ランチャー（`ms_final.sh`, `ms_resume_allargs.sh`）

| Flag | 意味 | デフォルト |
|---|---|---|
| `-ns`, `--numshapes` | 生成する形状数 | `100000` |
| `-r`, `--seed` | 乱数シード | `88888` |
| `-p`, `--prefix` | プレフィックス/再開キー | `""` |
| `-g`, `--numg` | 基底/グリッドパラメータ | `80` |
| `-bo`, `--baseouter` | ベース外周境界のオフセット | `0.25` |
| `-ro`, `--randouter` | ランダム外周境界オフセット | `0.20` |

### 学習（`three_stage_transmittance.py`）

| Flag | 意味 | デフォルト |
|---|---|---|
| `--preprocess` | 前処理モードを実行 | `False` |
| `--input_folder` | 統合済み CSV を含むフォルダ | `""` |
| `--output_npz` | 出力 NPZ のパス | `preprocessed_data.npz` |
| `--data_npz` | 学習用 NPZ データセット | `""` |
| `--csv_file` | NPZ を使わない場合の CSV フォールバック | `""` |
| `--test` | テストモード | `False` |
| `--num_epochs` | エポック数 | `10` |
| `--batch_size` | バッチサイズ | `4096` |

### 過去の評価設定（`three_stage_transmittance_evaluation.py`）

| Flag | 意味 | デフォルト |
|---|---|---|
| `--model_dir` | `stageA/B/C` を含むディレクトリ | required |
| `--data_npz` | NPZ 入力 | `""` |
| `--csv_file` | CSV 入力フォールバック | `""` |
| `--output_dir` | 出力ディレクトリ上書き | `model_dir` 配下で自動 |
| `--sample_count` | 可視化サンプル数 | `4` |
| `--seed` | 乱数シード | `23` |
| `--font_scale` | プロット文字倍率 | `1.0` |
| `--batch_size` | 評価バッチサイズ | `32` |
| `--plot_only` | 学習曲線のみ描画 | `False` |

### S4 整合性評価器（`FilterShapeS4_Evaluator_Transmittance.py`）

| Flag | 意味 | デフォルト |
|---|---|---|
| `--npz_file` | 入力 NPZ ファイル | `preprocessed_t_data.npz` |
| `--spec2shape_ckpt` | Stage C チェックポイントパス | `outputs_three_stage_20250322_145925/stageC/spec2shape_stageC.pt` |
| `--shape2spec_ckpt` | Stage A チェックポイントパス | `outputs_three_stage_20250322_145925/stageA/shape2spec_stageA.pt` |
| `--n_samples` | 評価サンプル数 | `4` |
| `--seed` | 乱数シード | `23` |
| `--max_workers` | S4 ワーカー数 | `4` |
| `--out_folder` | 出力ディレクトリ | タイムスタンプで自動 |

## 🧪 例

### スモーク実行

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

現在の逆設計設定は、結晶化状態にわたる透過率データから C4 対称幾何を回帰で復元することを学習しています。透過率パイプラインは現在、次を前提とします。

- 形状サンプルあたり 11 個の結晶化行（`shape_uid` ごとにグループ化）
- 各結晶化状態あたり 100 波長ビンの透過率列（`T@...` カラム）
- 最大 4 つの Q1 制御点を `4x3` テンソルとしてエンコード（`presence`, `x`, `y`）
- 形状の可視化と整合性チェックのための C4 対称ポリゴン再構成

主要な透過率学習パス以外にも、探索系（`AVIRIS*`、`noise_experiment*`、`archived/`）が存在します。

## 🧑‍💻 開発ノート

- これはパッケージ化された Python モジュールではなく、スクリプト中心の研究リポジトリです。
- コアスクリプトは相対パス（特に `../build/S4`、`results/`、`shapes/`）を前提にしています。
- `.gitignore` では多くの生成済み実験成果物（`*.csv`、`*.npz`、`*.pt`、実行フォルダ）を除外しています。
- 歴史的ドキュメントにあるファイル/ディレクトリは、このスナップショットでは欠落している場合があります。互換性のために注記付きで保持しています。
- macOS 側面（`._*`）ファイルは存在する場合があり、機能しないメタデータ断片である可能性があります。

## 🧯 トラブルシューティング

| 症状 | 想定原因 | 対処 |
|---|---|---|
| `../build/S4: No such file or directory` | 期待位置に S4 バイナリがない | `../build/S4` に S4 をビルド/リンクするか、ランチャーのパスを更新 |
| `No transmission columns found` | CSV に `T@...` 列がない | マージ結果のフォーマットを再確認 |
| `Must specify either --data_npz or --csv_file` | 学習/評価データ引数が未指定 | いずれかを明示的に指定 |
| `No valid shapes => SHIFT->Q1->UpTo4` | `vertices_str` が無効・空、または Q1 フィルタで有効サンプルがない | 形状ファイルとマージ結果を検証 |
| `Empty merge output for --prefix` | `results/` 内のファイルが `--prefix` と一致しない | プレフィックスを正確に確認してマージを再実行 |
| Evaluation checkpoint missing | `stageA/B/C` チェックポイントが不足 | `--model_dir` が完全な出力フォルダを指しているか確認 |
| `three_stage_transmittance_evaluation.py` not found | 歴史的ドキュメントでは参照されるが現状スクリプトが存在しない | `FilterShapeS4_Evaluator_Transmittance.py` を使うか、過去コミットから復元 |

## 🗺️ ロードマップ

- 明示的なデータバージョン管理マニフェストと固定済み実験設定で再現性を高める。
- transmittance、AVIRIS、noise の各ブランチ向けに正式なエントリポイントを整理。
- 前処理と 1 エポックのミニ学習向けの自動スモークテストを追加。
- 出力フォルダとコマンドラインを厳密に紐づける実験レジストリを明確化。
- 多言語 README 同期フロー（ルート言語ファイルと `i18n/`）を拡張。

## 🤝 コントリビューション

再現性、検証、ドキュメント品質を重視した貢献を歓迎します。

推奨フロー:

1. 対象範囲と期待される挙動を issue で共有。
2. フォーカスしたブランチを作成。
3. 実行可能なコマンドと結果を添えたプルリクエストを提出。
4. 可能であれば 1 つのワークフローに変更を限定。

## 📄 ライセンス

このスナップショット時点ではルートに `LICENSE` ファイルがありません。利用規約や再配布条件を定義するために追加してください。

## 📚 引用

このリポジトリを利用するか、本研究を発展させる場合は次を引用してください。

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

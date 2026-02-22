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

スペクトルイメージング向け **C4 対称メタサーフェス逆設計** のための、スクリプト中心の研究用リポジトリ（旧称 `inverse_metasurface`）です。対象範囲は次のとおりです。

- S4 を用いた RCWA データ生成（`.lua` + シェルランチャー）
- データ結合と前処理（`.csv` -> `.npz`）
- 3 段階のニューラル学習（shape->spectra、spectra->shape、chain fine-tuning）
- 評価と任意の neural-vs-S4 比較

## ✨ 概要

| 項目 | 詳細 |
|---|---|
| 主目的 | 目標透過スペクトルから形状を予測 |
| 主要データ形状 | spectra: `11 x 100`, shape: `4 x 3` |
| 主な学習スクリプト | `three_stage_transmittance.py` |
| 主な評価スクリプト | `three_stage_transmittance_evaluation.py` |
| RCWA ランチャー | `ms_final.sh`, `ms_resume_allargs.sh` |
| 結合スクリプト | `merge_s4_data_full.py` |

## 🧠 研究コンテキスト

本プロジェクトは、スペクトルイメージング向けメタサーフェスの逆設計を対象としています。学習パイプラインでは、結晶化状態ごとに S4 で生成した透過スペクトルを用い、順方向・逆方向の両マッピングを学習します。

1. **Stage A**: shape -> spectra
2. **Stage B**: spectra -> shape
3. **Stage C**: spectra -> shape -> spectra（連鎖損失でファインチューニング）

現在の前処理／学習コードは、以下を前提としています。

- 11 段階の結晶化状態（`c = 0.0 ... 1.0`）
- 各状態あたり 100 の波長ビン
- 形状表現は最大 4 個の Q1 点（`[presence, x, y]`）

## 🗂️ リポジトリ構成（主要パス）

```text
.
├── ms.sh / ms_final.sh / ms_resume_allargs.sh
├── metasurface_final.lua / metasurface_allargs_resume.lua / metasurface_seed.lua
├── merge_s4_data_full.py
├── three_stage_transmittance.py
├── three_stage_transmittance_evaluation.py
├── FilterShapeS4_Evaluator_Transmittance.py
├── results/                # 生の S4 出力 CSV
├── shapes/                 # 生成されたポリゴン頂点
├── merged_csvs/            # 前処理で使用する結合済み CSV
├── outputs_three_stage_*/  # チェックポイント、損失、可視化
├── partial_crys_data/
└── iccp.yaml
```

## ⚙️ 前提条件

| 依存関係 | 要件 |
|---|---|
| OS | Linux |
| Shell | Bash |
| Python | 3.9 |
| 環境管理 | Conda（推奨） |
| RCWA バイナリ | `../build/S4`（リポジトリルート相対） |
| GPU | 任意（学習高速化のため推奨） |

## 🚀 セットアップ

```bash
git clone <repo-url> inverse_metasurface
cd inverse_metasurface

conda env create -f iccp.yaml
conda activate iccp

# Required by launcher scripts
ls -l ../build/S4
```

任意:

```bash
chmod +x ms.sh ms_final.sh ms_resume_allargs.sh
```

## 🧪 エンドツーエンド利用手順

### 1) S4 で RCWA データを生成

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

- ランチャーは `../build/S4` を `-t 32` で呼び出し、`NQ=1..4` を並列実行します。
- `ms_final.sh` は `metasurface_final.lua` を使用します。
- `ms_resume_allargs.sh` は `metasurface_allargs_resume.lua` を使用します。

### 2) RCWA 出力と形状頂点を結合

```bash
python merge_s4_data_full.py --prefix myrun
# -> merged_s4_shapes_myrun.csv
```

### 3) 学習用に結合列名を正規化（必要な場合）

`merge_s4_data_full.py` は `folder_key` / `NQ` を出力しますが、学習パイプラインは `prefix` / `nQ` を想定しています。

```bash
python -c "import pandas as pd; p='merged_s4_shapes_myrun.csv'; df=pd.read_csv(p); df=df.rename(columns={'folder_key':'prefix','NQ':'nQ'}); df.to_csv(p,index=False)"
```

### 4) CSV -> NPZ を前処理

```bash
mkdir -p merged_csvs
mv merged_s4_shapes_myrun.csv merged_csvs/

python three_stage_transmittance.py \
  --preprocess \
  --input_folder merged_csvs \
  --output_npz preprocessed_t_data.npz
```

### 5) 3 段階モデルを学習

```bash
python three_stage_transmittance.py \
  --data_npz preprocessed_t_data.npz \
  --num_epochs 100 \
  --batch_size 1024
```

主な出力構成:

- `outputs_three_stage_YYYYMMDD_HHMMSS/stageA`
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageB`
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageC`

### 6) チェックポイントを評価

```bash
python three_stage_transmittance_evaluation.py \
  --model_dir outputs_three_stage_YYYYMMDD_HHMMSS \
  --data_npz preprocessed_t_data.npz \
  --sample_count 8
```

### 7) 任意: ニューラル予測と S4 を比較

```bash
python FilterShapeS4_Evaluator_Transmittance.py \
  --npz_file preprocessed_t_data.npz \
  --spec2shape_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageC/spec2shape_stageC.pt \
  --shape2spec_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageA/shape2spec_stageA.pt \
  --n_samples 4
```

## 🎛️ CLI リファレンス

### S4 ランチャーフラグ（`ms_final.sh`, `ms_resume_allargs.sh`）

| Flag | 意味 | Default |
|---|---|---|
| `-ns`, `--numshapes` | 形状数 | `100000` |
| `-r`, `--seed` | 乱数シード | `88888` |
| `-p`, `--prefix` | 実行プレフィックス / 再開キー | `""` |
| `-g`, `--numg` | 幾何基底パラメータ | `80` |
| `-bo`, `--baseouter` | 基本外側オフセット | `0.25` |
| `-ro`, `--randouter` | ランダム外側オフセット | `0.20` |

### 学習フラグ（`three_stage_transmittance.py`）

| Flag | 用途 |
|---|---|
| `--preprocess` | 前処理モードを実行 |
| `--input_folder` | 結合 CSV ファイルのフォルダ |
| `--output_npz` | 出力する前処理済み NPZ ファイル名 |
| `--data_npz` | 学習用 NPZ データセット |
| `--csv_file` | 代替の CSV データセット |
| `--test` | テストモード |
| `--num_epochs` | 学習エポック数 |
| `--batch_size` | バッチサイズ |

### 評価フラグ（`three_stage_transmittance_evaluation.py`）

| Flag | 用途 |
|---|---|
| `--model_dir` | チェックポイントのルートディレクトリ（必須） |
| `--data_npz` / `--csv_file` | 評価データソース |
| `--output_dir` | 評価出力フォルダ |
| `--sample_count` | 可視化サンプル数 |
| `--seed` | サンプル選択用乱数シード |
| `--font_scale` | プロットのフォント倍率 |
| `--batch_size` | 評価バッチサイズ |
| `--plot_only` | 学習曲線プロットのみ再生成 |

## 🧾 データ契約（前処理用）

`three_stage_transmittance.py` の前処理パスは、次を含む結合 CSV を想定しています。

- ID 列: `prefix`, `nQ`, `nS`, `shape_idx`, `c`
- 幾何テキスト: `vertices_str`
- スペクトル列: `T@...`

コードで適用される品質チェック:

- `shape_uid = prefix_nQ_nS_shape_idx` でグループ化
- 各グループは必ず 11 行を含むこと
- Q1 点数が `[1, 4]` の形状のみ保持

## 🛠️ トラブルシューティング

- `../build/S4: No such file or directory`
  - `../build/S4` に S4 をビルド／配置するか、ランチャースクリプトを実際の S4 パスに合わせて修正してください。
- `No matching CSVs found in 'results/'`
  - `--prefix` と `results/*_output_nQ*_nS*.csv` の出力命名を確認してください。
- `No transmission columns found`
  - 結合 CSV に `T@...` 列が含まれていることを確認してください。
- 前処理結果が 0 件になる
  - 必須列があるか、各 shape UID に結晶化 11 行が揃っているかを確認してください。
- 学習中に GPU OOM が発生
  - `--batch_size` を下げてください（例: `256` または `128`）。
- 評価でチェックポイントが見つからない
  - `--model_dir` 配下に次が存在することを確認してください。
    - `stageA/shape2spec_stageA.pt`
    - `stageB/spec2shape_stageB.pt`
    - `stageC/spec2shape_stageC.pt`

## 📚 Citation

このリポジトリが研究に役立つ場合は、以下を引用してください。

```bibtex
@article{chen2025inverse,
  title={Inverse Design of Metasurface for Spectral Imaging},
  author={Chen, Rongzhou and Nie, Haitao and Zhu, Shuo and Zhao, Yaping and Wang, Chutian and Lam, Edmund Y},
  journal={arXiv preprint arXiv:2510.21924},
  year={2025}
}
```

## 🌐 Language Variants

このリポジトリには、以下を含む追加の README バリアントがあります。

- `README.en.md`, `README.de.md`, `README.es.md`, `README.fr.md`
- `README.ru.md`, `README.ja.md`, `README.ko.md`, `README.vi.md`
- `README.ar.md`, `README.zh-CN.md`, `README.zh-TW.md`

## 📌 Notes

- これは研究用ワークスペースであり、アーカイブ済み・探索的なスクリプトを多く含みます。
- 正準の透過率ワークフローは次を中心に構成されています。
  - `ms_final.sh` / `ms_resume_allargs.sh`
  - `merge_s4_data_full.py`
  - `three_stage_transmittance.py`
  - `three_stage_transmittance_evaluation.py`
- 現時点で、リポジトリルートに明示的なライセンスファイルはありません。

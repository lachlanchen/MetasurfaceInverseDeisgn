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
  <img alt="RCWA" src="https://img.shields.io/badge/Simulator-S4%20RCWA-16a34a?style=for-the-badge">
  <img alt="Platform" src="https://img.shields.io/badge/Platform-Linux%20%2B%20Bash-6b7280?style=for-the-badge&logo=gnu-bash&logoColor=white">
</p>

スペクトルイメージング向けメタサーフェス逆設計のための、スクリプト中心の研究用リポジトリ（旧称 `inverse_metasurface`）です。中核ワークフローは次を統合しています。

- 物理に基づくRCWAシミュレーション（S4 + Lua）
- データ統合と形状情報の付与
- 3段階のPyTorch学習（`shape -> spectrum`、`spectrum -> shape`、連結ファインチューニング）
- 定量・定性評価（必要に応じてニューラル推定とS4再計算の整合性チェック）

## ✨ 概要

| 項目 | 内容 |
|---|---|
| 🎯 主タスク | 目標透過スペクトルから C4 対称メタサーフェス形状を推定 |
| 🔬 シミュレータ | シェルランチャーと `.lua` スクリプトから呼び出される `../build/S4` |
| 🧠 学習パイプライン | Stage A `shape -> spectra`、Stage B `spectra -> shape`、Stage C `spectra -> shape -> spectra` |
| 📦 データ契約 | 統合CSV（`T@...`、メタデータ、`vertices_str`） -> 圧縮NPZ（`uids`、`spectra`、`shapes`） |
| 🧪 評価 | MSE指標、各ステージ可視化、任意のS4再シミュレーション |

## 🧭 エンドツーエンドの流れ

1. `results/` にシミュレーション出力、`shapes/` にポリゴンファイルを生成する。
2. S4 CSVを統合し、形状頂点情報を付与する。
3. 学習互換性のために統合列名を正規化する。
4. 統合CSVをNPZテンソルへ前処理する。
5. Stage A/B/Cモデルを学習する。
6. チェックポイントを評価し、挙動を可視化する。
7. 必要に応じて、予測形状スペクトルと新規S4実行結果を比較する。

## 🧱 リポジトリ構成

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

## 🛠️ 前提条件

| 依存要件 | 備考 |
|---|---|
| Linux + Bash | ランチャースクリプトはシェル実行を前提 |
| Python 3.9 | `iccp.yaml` と整合 |
| Conda | 再現性確保のため推奨 |
| S4バイナリ | `../build/S4` に配置されている想定 |
| CUDA GPU（任意） | 学習・評価を高速化 |

## 🚀 セットアップ

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

代替手順（再現性はやや低下）:

```bash
pip install -r pip_requirements.txt
```

### 3) スクリプトが期待するシミュレータパスを確認

```bash
ls -l ../build/S4
```

### 4) （任意）ランチャーに実行権限を付与

```bash
chmod +x ms.sh ms_final.sh ms_resume.sh ms_resume_allargs.sh ms_resume_random_state.sh
```

## ▶️ 実践的な使い方

### A) RCWAシミュレーションデータを生成

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

再開用途ランチャー:

```bash
./ms_resume_allargs.sh \
  -ns 10000 \
  -r 12345 \
  -p myrun \
  -g 80 \
  -bo 0.35 \
  -ro 0.30
```

補足:
- ランチャーは `NQ=1..4` を並列実行します。
- スクリプトは `-t 32` 付きで `../build/S4` を呼び出します。

### B) S4出力を統合し、形状頂点を付与

```bash
python merge_s4_data_full.py --prefix myrun
# output: merged_s4_shapes_myrun.csv
```

### C) 学習互換のため列名を正規化

`merge_s4_data_full.py` は `folder_key` と `NQ` を出力しますが、学習系では `prefix` と `nQ` が期待されます。

```bash
python -c "import pandas as pd; p='merged_s4_shapes_myrun.csv'; df=pd.read_csv(p); df=df.rename(columns={'folder_key':'prefix','NQ':'nQ'}); df.to_csv(p,index=False)"
```

### D) CSV -> NPZ 前処理

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

```bash
python three_stage_transmittance_evaluation.py \
  --model_dir outputs_three_stage_YYYYMMDD_HHMMSS \
  --data_npz preprocessed_t_data.npz \
  --sample_count 8
```

### G) （任意）ニューラル推定とS4の整合性チェック

```bash
python FilterShapeS4_Evaluator_Transmittance.py \
  --npz_file preprocessed_t_data.npz \
  --spec2shape_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageC/spec2shape_stageC.pt \
  --shape2spec_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageA/shape2spec_stageA.pt \
  --n_samples 4
```

## ⚙️ 主要CLIオプション

### S4ランチャー（`ms_final.sh`, `ms_resume_allargs.sh`）

| Flag | 意味 | Default |
|---|---|---|
| `-ns`, `--numshapes` | 生成する形状数 | `100000` |
| `-r`, `--seed` | 乱数シード | `88888` |
| `-p`, `--prefix` | 接頭辞/再開キー | `""` |
| `-g`, `--numg` | 基底/グリッドパラメータ | `80` |
| `-bo`, `--baseouter` | 外周境界の基準オフセット | `0.25` |
| `-ro`, `--randouter` | 外周境界のランダムオフセット | `0.20` |

### 学習（`three_stage_transmittance.py`）

| Flag | 意味 | Default |
|---|---|---|
| `--preprocess` | 前処理モードを実行 | `False` |
| `--input_folder` | 統合CSVがあるフォルダ | `""` |
| `--output_npz` | 出力NPZパス | `preprocessed_data.npz` |
| `--data_npz` | 学習用NPZデータセット | `""` |
| `--csv_file` | NPZ未使用時のCSV代替入力 | `""` |
| `--test` | テストモード | `False` |
| `--num_epochs` | 学習エポック数 | `10` |
| `--batch_size` | バッチサイズ | `4096` |

### 評価（`three_stage_transmittance_evaluation.py`）

| Flag | 意味 | Default |
|---|---|---|
| `--model_dir` | `stageA/B/C` を含むディレクトリ | required |
| `--data_npz` | NPZ入力 | `""` |
| `--csv_file` | CSV代替入力 | `""` |
| `--output_dir` | 出力先の上書き指定 | `model_dir` 配下に自動生成 |
| `--sample_count` | 可視化サンプル数 | `4` |
| `--seed` | 乱数シード | `23` |
| `--font_scale` | プロットのフォント倍率 | `1.0` |
| `--batch_size` | 評価時バッチサイズ | `32` |
| `--plot_only` | 学習曲線の描画のみ実行 | `False` |

### S4整合性評価（`FilterShapeS4_Evaluator_Transmittance.py`）

| Flag | 意味 | Default |
|---|---|---|
| `--npz_file` | 入力NPZファイル | `preprocessed_t_data.npz` |
| `--spec2shape_ckpt` | Stage Cチェックポイントパス | `outputs_three_stage_20250322_145925/stageC/spec2shape_stageC.pt` |
| `--shape2spec_ckpt` | Stage Aチェックポイントパス | `outputs_three_stage_20250322_145925/stageA/shape2spec_stageA.pt` |
| `--n_samples` | 評価サンプル数 | `4` |
| `--seed` | 乱数シード | `23` |
| `--max_workers` | S4ワーカースレッド数 | `4` |
| `--out_folder` | 出力ディレクトリ | タイムスタンプで自動生成 |

## 🧪 スモーク実行

```bash
./ms_final.sh -ns 1000 -r 42 -p smoke -g 40 -bo 0.25 -ro 0.20
python merge_s4_data_full.py --prefix smoke
python -c "import pandas as pd; p='merged_s4_shapes_smoke.csv'; d=pd.read_csv(p).rename(columns={'folder_key':'prefix','NQ':'nQ'}); d.to_csv(p,index=False)"
mkdir -p merged_csvs && mv merged_s4_shapes_smoke.csv merged_csvs/
python three_stage_transmittance.py --preprocess --input_folder merged_csvs --output_npz smoke.npz
python three_stage_transmittance.py --data_npz smoke.npz --num_epochs 5 --batch_size 128
```

## 🔬 研究コンテキスト

現在の逆設計設定では、結晶化状態にわたる透過率から C4 対称形状を復元する学習を行います。透過率パイプラインは現在、次を前提としています。

- 形状サンプルごとに結晶化行が11行（`shape_uid` 単位でグループ化）
- 結晶化状態ごとに100波長ビン（`T@...` 列）
- 最大4つのQ1制御点を `4x3` テンソルとして符号化: `(presence, x, y)`
- 形状可視化および整合性検証のため、C4対称性を仮定したポリゴン再構成

このリポジトリには、主たる透過率学習パス以外に探索的ブランチ（`AVIRIS*`, `noise_experiment*`, `archived/`）も含まれます。

## 🧯 トラブルシューティング

| 症状 | 想定原因 | 対処 |
|---|---|---|
| `../build/S4: No such file or directory` | 期待相対パスにS4バイナリがない | `../build/S4` にS4をビルド/リンクするか、ランチャーパスを更新 |
| `No transmission columns found` | CSVに `T@...` 列がない | マージ出力形式を再確認 |
| `Must specify either --data_npz or --csv_file` | 学習/評価データ引数の不足 | どちらかの入力を明示指定 |
| `No valid shapes => SHIFT->Q1->UpTo4` | `vertices_str` が不正/空、またはQ1フィルタですべて除外 | 形状ファイルとマージ出力を検証 |
| `--prefix` 指定でマージ結果が空 | `results/` 内ファイルと接頭辞が不一致 | 正確なファイル接頭辞を確認して再実行 |
| 評価チェックポイントが見つからない | `stageA/B/C` のチェックポイント不足 | `--model_dir` が完全な出力フォルダか確認 |

## 🤝 コントリビューション

再現性、テスト、ドキュメント品質の向上につながる貢献を歓迎します。

推奨手順:

1. スコープと期待動作をIssueで共有する。
2. 焦点を絞ったブランチを作成する。
3. 実行可能コマンドと出力を添えてPull Requestを提出する。
4. 可能な限り1つのワークフローに変更範囲を限定する。

## 📄 ライセンス

このスナップショットでは、リポジトリルートに `LICENSE` ファイルはありません。利用・再配布条件を明確にするため追加してください。

## 📚 Citation

このリポジトリを利用した場合、または本研究を基にした場合は、以下を引用してください。

```bibtex
@article{chen2025inverse,
  title={Inverse Design of Metasurface for Spectral Imaging},
  author={Chen, Rongzhou and Nie, Haitao and Zhu, Shuo and Zhao, Yaping and Wang, Chutian and Lam, Edmund Y},
  journal={arXiv preprint arXiv:2510.21924},
  year={2025}
}
```

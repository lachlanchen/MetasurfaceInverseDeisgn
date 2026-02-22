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

这是一个以脚本为中心的研究型仓库（历史上也称为 `inverse_metasurface`），用于**面向光谱成像的 C4 对称超表面逆向设计**，覆盖以下完整流程：

- 使用 S4 进行 RCWA 数据生成（`.lua` + shell 启动脚本）
- 数据合并与预处理（`.csv` -> `.npz`）
- 三阶段神经网络训练（shape->spectra、spectra->shape、链式微调）
- 模型评估以及可选的神经网络与 S4 对比验证

## ✨ 一览

| 项目 | 说明 |
|---|---|
| 核心目标 | 根据目标透射光谱预测几何形状 |
| 核心数据形状 | spectra: `11 x 100`，shape: `4 x 3` |
| 主要训练脚本 | `three_stage_transmittance.py` |
| 主要评估脚本 | `three_stage_transmittance_evaluation.py` |
| RCWA 启动脚本 | `ms_final.sh`, `ms_resume_allargs.sh` |
| 合并脚本 | `merge_s4_data_full.py` |

## 🧠 研究背景

本项目聚焦于光谱成像场景下的超表面逆向设计。训练流程使用 S4 生成不同结晶状态下的透射光谱，并同时学习正向与逆向映射：

1. **Stage A**: shape -> spectra
2. **Stage B**: spectra -> shape
3. **Stage C**: spectra -> shape -> spectra（链式损失微调）

当前预处理/训练代码默认如下设定：

- 11 个结晶状态（`c = 0.0 ... 1.0`）
- 每个状态 100 个波长 bin
- 形状表示为最多 4 个 Q1 点，每个点格式为 `[presence, x, y]`

## 🗂️ 仓库结构（核心路径）

```text
.
├── ms.sh / ms_final.sh / ms_resume_allargs.sh
├── metasurface_final.lua / metasurface_allargs_resume.lua / metasurface_seed.lua
├── merge_s4_data_full.py
├── three_stage_transmittance.py
├── three_stage_transmittance_evaluation.py
├── FilterShapeS4_Evaluator_Transmittance.py
├── results/                # 原始 S4 输出 CSV
├── shapes/                 # 生成的多边形顶点
├── merged_csvs/            # 用于预处理的合并 CSV
├── outputs_three_stage_*/  # checkpoint、loss、可视化结果
├── partial_crys_data/
└── iccp.yaml
```

## ⚙️ 环境依赖

| 依赖 | 要求 |
|---|---|
| OS | Linux |
| Shell | Bash |
| Python | 3.9 |
| 环境管理 | Conda（推荐） |
| RCWA 可执行文件 | `../build/S4`（相对仓库根目录） |
| GPU | 可选，建议用于加速训练 |

## 🚀 安装

```bash
git clone <repo-url> inverse_metasurface
cd inverse_metasurface

conda env create -f iccp.yaml
conda activate iccp

# Required by launcher scripts
ls -l ../build/S4
```

可选：

```bash
chmod +x ms.sh ms_final.sh ms_resume_allargs.sh
```

## 🧪 端到端使用流程

### 1) 使用 S4 生成 RCWA 数据

```bash
./ms_final.sh \
  -ns 10000 \
  -r 12345 \
  -p myrun \
  -g 80 \
  -bo 0.35 \
  -ro 0.30
```

说明：

- 启动脚本会调用 `../build/S4` 并使用 `-t 32`，同时并行运行 `NQ=1..4`。
- `ms_final.sh` 使用 `metasurface_final.lua`。
- `ms_resume_allargs.sh` 使用 `metasurface_allargs_resume.lua`。

### 2) 合并 RCWA 输出与形状顶点

```bash
python merge_s4_data_full.py --prefix myrun
# -> merged_s4_shapes_myrun.csv
```

### 3) 为训练统一合并列名（如有需要）

`merge_s4_data_full.py` 输出列名为 `folder_key` / `NQ`，而训练流程期望 `prefix` / `nQ`。

```bash
python -c "import pandas as pd; p='merged_s4_shapes_myrun.csv'; df=pd.read_csv(p); df=df.rename(columns={'folder_key':'prefix','NQ':'nQ'}); df.to_csv(p,index=False)"
```

### 4) 预处理 CSV -> NPZ

```bash
mkdir -p merged_csvs
mv merged_s4_shapes_myrun.csv merged_csvs/

python three_stage_transmittance.py \
  --preprocess \
  --input_folder merged_csvs \
  --output_npz preprocessed_t_data.npz
```

### 5) 训练三阶段模型

```bash
python three_stage_transmittance.py \
  --data_npz preprocessed_t_data.npz \
  --num_epochs 100 \
  --batch_size 1024
```

主要输出结构：

- `outputs_three_stage_YYYYMMDD_HHMMSS/stageA`
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageB`
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageC`

### 6) 评估 checkpoint

```bash
python three_stage_transmittance_evaluation.py \
  --model_dir outputs_three_stage_YYYYMMDD_HHMMSS \
  --data_npz preprocessed_t_data.npz \
  --sample_count 8
```

### 7) 可选：将神经网络预测与 S4 对比

```bash
python FilterShapeS4_Evaluator_Transmittance.py \
  --npz_file preprocessed_t_data.npz \
  --spec2shape_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageC/spec2shape_stageC.pt \
  --shape2spec_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageA/shape2spec_stageA.pt \
  --n_samples 4
```

## 🎛️ CLI 参考

### S4 启动参数（`ms_final.sh`, `ms_resume_allargs.sh`）

| 参数 | 含义 | 默认值 |
|---|---|---|
| `-ns`, `--numshapes` | 形状数量 | `100000` |
| `-r`, `--seed` | 随机种子 | `88888` |
| `-p`, `--prefix` | 运行前缀 / 续跑 key | `""` |
| `-g`, `--numg` | 几何基参数 | `80` |
| `-bo`, `--baseouter` | 基础外偏移 | `0.25` |
| `-ro`, `--randouter` | 随机外偏移 | `0.20` |

### 训练参数（`three_stage_transmittance.py`）

| 参数 | 用途 |
|---|---|
| `--preprocess` | 启用预处理模式 |
| `--input_folder` | 合并 CSV 文件夹 |
| `--output_npz` | 预处理输出 NPZ 文件名 |
| `--data_npz` | 用于训练的 NPZ 数据集 |
| `--csv_file` | 可替代的 CSV 数据源 |
| `--test` | 测试模式 |
| `--num_epochs` | 训练轮数 |
| `--batch_size` | 批大小 |

### 评估参数（`three_stage_transmittance_evaluation.py`）

| 参数 | 用途 |
|---|---|
| `--model_dir` | checkpoint 根目录（必填） |
| `--data_npz` / `--csv_file` | 评估数据来源 |
| `--output_dir` | 评估输出目录 |
| `--sample_count` | 可视化样本数 |
| `--seed` | 样本选择随机种子 |
| `--font_scale` | 绘图字体缩放 |
| `--batch_size` | 评估批大小 |
| `--plot_only` | 仅重新生成训练曲线图 |

## 🧾 数据约定（预处理）

`three_stage_transmittance.py` 中的预处理流程要求合并 CSV 包含以下字段：

- ID 列：`prefix`, `nQ`, `nS`, `shape_idx`, `c`
- 几何文本：`vertices_str`
- 光谱列：`T@...`

代码内执行的质量检查：

- 按 `shape_uid = prefix_nQ_nS_shape_idx` 分组
- 每组必须恰好包含 11 行
- 仅保留 Q1 点数量在 `[1, 4]` 的形状

## 🛠️ 常见问题排查

- `../build/S4: No such file or directory`
  - 在 `../build/S4` 构建/链接 S4，或将启动脚本改为你的实际 S4 路径。
- `No matching CSVs found in 'results/'`
  - 检查 `--prefix` 及 `results/*_output_nQ*_nS*.csv` 的输出命名。
- `No transmission columns found`
  - 确认合并 CSV 包含 `T@...` 列。
- 预处理得到 0 条记录
  - 检查必需列是否齐全，并确认每个 shape UID 都有 11 条结晶状态记录。
- 训练时 GPU OOM
  - 减小 `--batch_size`（例如 `256` 或 `128`）。
- 评估时找不到 checkpoint
  - 确认 `--model_dir` 下存在：
    - `stageA/shape2spec_stageA.pt`
    - `stageB/spec2shape_stageB.pt`
    - `stageC/spec2shape_stageC.pt`

## 📚 Citation

If this repository contributes to your research, please cite:

```bibtex
@article{chen2025inverse,
  title={Inverse Design of Metasurface for Spectral Imaging},
  author={Chen, Rongzhou and Nie, Haitao and Zhu, Shuo and Zhao, Yaping and Wang, Chutian and Lam, Edmund Y},
  journal={arXiv preprint arXiv:2510.21924},
  year={2025}
}
```

## 🌐 语言版本

本仓库还提供其他 README 版本，包括：

- `README.en.md`, `README.de.md`, `README.es.md`, `README.fr.md`
- `README.ru.md`, `README.ja.md`, `README.ko.md`, `README.vi.md`
- `README.ar.md`, `README.zh-CN.md`, `README.zh-TW.md`

## 📌 说明

- 这是一个研究工作区，包含较多归档与探索性质脚本。
- 当前规范的透射率主流程集中在：
  - `ms_final.sh` / `ms_resume_allargs.sh`
  - `merge_s4_data_full.py`
  - `three_stage_transmittance.py`
  - `three_stage_transmittance_evaluation.py`
- 仓库根目录当前尚未提供明确的许可证文件。

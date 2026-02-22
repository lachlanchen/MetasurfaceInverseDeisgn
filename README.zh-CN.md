English | 中文繁體 | 中文简体 | 日本語 | 한국어 | Tiếng Việt | العربية | Français | Español | Deutsch | Русский

# inverse_metasurface

<div align="center">

[![Status](https://img.shields.io/badge/status-experimental-orange)](#-路线图)
[![Python](https://img.shields.io/badge/python-3.9-blue)](#-前置要求)
[![Platform](https://img.shields.io/badge/platform-linux-lightgrey)](#-前置要求)
[![S4](https://img.shields.io/badge/S4-required-success)](#-前置要求)
[![License](https://img.shields.io/badge/license-unset-lightgrey)](#-许可证)

</div>

这是一个面向**反向超表面设计**的研究型工作区，基于 **S4/RCWA 仿真**与**多阶段神经网络模型**。

它支持：
- 面向 C4 对称多边形超表面的高吞吐 S4 数据生成。
- 数据合并与预处理，输出可直接用于训练的 NPZ。
- 三阶段学习：**shape -> spectrum**、**spectrum -> shape**、**chain-tuned spectrum -> shape -> spectrum**。
- 模型评估，以及可选的直接**neural-vs-S4**对比。

## 🌟 概览

| 模块 | 本仓库提供内容 |
|---|---|
| 物理仿真 | Bash + Lua 脚本，支持在 `nQ=1..4` 上并行运行 S4 |
| 数据集工具 | 合并脚本，附加多边形顶点并透视光谱枢轴化 |
| 机器学习流程 | `three_stage_transmittance.py`，含 preprocess + train 两种模式 |
| 评估 | `three_stage_transmittance_evaluation.py`，提供指标与图像 |
| 研究分支 | AVIRIS/hyperspectral 与 noise/compression 实验 |

## 🧠 研究背景

本仓库聚焦光子超表面的反向设计：根据目标光谱推断几何结构（以及反向过程）。

基线流程采用的核心假设：
- 通过 Q1 点参数化施加 C4 对称。
- 11 个结晶状态（`c` 取值在 `[0.0, 1.0]`）。
- 每个样本光谱以 `11 x 100` 透射率行存储。
- 形状表示为最多 4 个 Q1 点（`4 x 3` 张量：presence, x, y）。

三阶段训练逻辑：
1. **Stage A**: shape -> spectrum (`shape2spec_stageA.pt`)
2. **Stage B**: spectrum -> shape (`spec2shape_stageB.pt`)
3. **Stage C**: spectrum -> shape -> spectrum 链式微调 (`spec2shape_stageC.pt`)

## 🗂 项目结构

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
├─ partial_crys_data/                # 按结晶度划分的光学常数
├─ results/                          # 原始 S4 输出（通常不纳入版本控制）
├─ shapes/                           # 生成的多边形顶点文件
├─ merged_csvs/                      # 用于预处理的合并表
├─ outputs_three_stage_*/            # 每次运行的模型产物
├─ AVIRIS*/ + aviris_*.py            # 高光谱分支
├─ how_to_run.md / commands*.md
├─ iccp.yaml
└─ pip_requirements.txt
```

## ✅ 前置要求

| 要求 | 说明 |
|---|---|
| 操作系统 | Linux（脚本默认 Bash + Linux 路径） |
| Python | 3.9（见 `iccp.yaml`） |
| 环境管理器 | 推荐 Conda |
| S4 可执行文件 | 预期路径：仓库根目录相对路径 `../build/S4` |
| GPU（可选） | CUDA 可加速训练/评估 |

## ⚙️ 安装

### 1) 克隆并进入目录

```bash
git clone <your-repo-url> inverse_metasurface
cd inverse_metasurface
```

### 2) 创建环境

```bash
conda env create -f iccp.yaml
conda activate iccp
```

### 3) 验证 S4 路径

```bash
ls -l ../build/S4
```

如果该路径不存在，请在该位置构建/放置 S4，或相应调整脚本中的路径。

## 🚀 快速开始（端到端）

```bash
# 1) 生成 S4 数据
./ms.sh -ns 10000 -r 12345

# 2) 按 prefix 合并一次运行结果
python merge.py --prefix 20250123_155420

# 3) 将合并后的 CSV 移动到预处理目录
mkdir -p merged_csvs
mv merged_s4_shapes_20250123_155420.csv merged_csvs/

# 4) 预处理为 NPZ
python three_stage_transmittance.py --preprocess \
  --input_folder merged_csvs \
  --output_npz preprocessed_t_data.npz

# 5) 训练三阶段模型
python three_stage_transmittance.py \
  --data_npz preprocessed_t_data.npz \
  --num_epochs 100 \
  --batch_size 1024

# 6) 评估模型输出
python three_stage_transmittance_evaluation.py \
  --model_dir outputs_three_stage_YYYYMMDD_HHMMSS \
  --data_npz preprocessed_t_data.npz \
  --sample_count 8
```

## 🧪 使用细节

### A) S4 数据生成

最小化种子运行：

```bash
./ms.sh -ns 10000 -r 88888
```

参数化运行：

```bash
./ms_final.sh \
  -ns 100000 \
  -r 88888 \
  -p iccpOv100kG80 \
  -g 80 \
  -bo 0.35 \
  -ro 0.30
```

续跑模式：

```bash
./ms_resume_allargs.sh \
  -ns 100000 \
  -r 88888 \
  -p iccpOv100kG80 \
  -g 80 \
  -bo 0.35 \
  -ro 0.30
```

### B) 合并原始 S4 CSV

```bash
python merge.py --prefix 20250123_155420
```

可选工具：

```bash
python merge_s4_data_full.py --prefix 20250123_155420
```

### C) 预处理合并后的 CSV -> NPZ

```bash
python three_stage_transmittance.py --preprocess \
  --input_folder merged_csvs \
  --output_npz preprocessed_t_data.npz
```

### D) 训练

```bash
python three_stage_transmittance.py \
  --data_npz preprocessed_t_data.npz \
  --num_epochs 100 \
  --batch_size 1024
```

训练输出存放于：
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageA`
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageB`
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageC`

### E) 评估

```bash
python three_stage_transmittance_evaluation.py \
  --model_dir outputs_three_stage_YYYYMMDD_HHMMSS \
  --data_npz preprocessed_t_data.npz \
  --sample_count 8
```

将生成：
- `evaluation_metrics.csv`
- `metrics_summary.csv`
- 阶段可视化（`.png`, `.pdf`）
- 训练曲线图

### F) Neural vs direct S4 对比（可选）

```bash
python FilterShapeS4_Evaluator_Transmittance.py \
  --npz_file preprocessed_t_data.npz \
  --spec2shape_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageC/spec2shape_stageC.pt \
  --shape2spec_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageA/shape2spec_stageA.pt \
  --n_samples 4
```

## 🔧 关键 CLI 选项

### S4 运行参数（`ms_final.sh`, `ms_resume_allargs.sh`）

| Flag | 含义 | 默认值 |
|---|---|---|
| `-ns`, `--numshapes` | 形状数量 | `100000` |
| `-r`, `--seed` | 随机种子 | `88888` |
| `-p`, `--prefix` | 运行前缀 / 续跑键 | empty |
| `-g`, `--numg` | 几何/基参数 | `80` |
| `-bo`, `--baseouter` | 外轮廓基准偏移 | `0.25` |
| `-ro`, `--randouter` | 外轮廓随机偏移 | `0.20` |

### 训练参数（`three_stage_transmittance.py`）

| Flag | 含义 | 默认值 |
|---|---|---|
| `--preprocess` | 运行预处理模式 | off |
| `--input_folder` | 输入 CSV 目录 | empty |
| `--output_npz` | 输出 NPZ 路径 | `preprocessed_data.npz` |
| `--data_npz` | 训练/评估使用的 NPZ | empty |
| `--csv_file` | 直接使用的 CSV | empty |
| `--num_epochs` | 每阶段 Epoch 数 | `10` |
| `--batch_size` | 批大小 | `4096` |
| `--test` | 测试模式占位项 | off |

### 评估参数（`three_stage_transmittance_evaluation.py`）

| Flag | 含义 | 默认值 |
|---|---|---|
| `--model_dir` | 训练结果根目录 | required |
| `--data_npz` | 评估 NPZ | empty |
| `--csv_file` | 评估 CSV | empty |
| `--output_dir` | 输出目录 | `model_dir/evaluation_<timestamp>` |
| `--sample_count` | 可视化样本数量 | `4` |
| `--seed` | 采样种子 | `23` |
| `--font_scale` | 绘图字体缩放 | `1.0` |
| `--batch_size` | 评估批大小 | `32` |
| `--plot_only` | 仅绘制曲线 | off |

## 🧭 故障排查

| 现象 | 可能原因 | 解决方法 |
|---|---|---|
| `../build/S4: No such file or directory` | S4 可执行文件路径不匹配 | 在 `../build/S4` 构建/放置 S4，或修改脚本 |
| `Must specify either --data_npz or --csv_file` | 缺少数据集来源 | 显式传入其中一个参数 |
| `No matching CSVs found in 'results/'` | prefix 不匹配 | 检查 prefix 与输出命名 |
| 预处理后记录极少或为 0 | 缺失/无效的 `T@...` 或 `vertices_str` 行 | 校验合并后 CSV 的 schema 与按形状分组行完整性 |
| CUDA OOM | batch 过大 | 下调 `--batch_size`（例如 `1024 -> 256`） |

## 🧱 开发说明

- 这是一个偏实验驱动的仓库；许多生成产物有意不纳入版本控制。
- 同类任务存在多个脚本变体（`merge_*`, `aviris_*`, `noise_experiment_*`）。
- 本 README 记录的是基线 inverse-metasurface 工作流。
- 训练脚本会设定种子（`42`），但严格确定性仍依赖硬件/后端行为。
- 目前尚无统一 CI + 完整自动化测试套件。

## 🛣 路线图

- 增加一个紧凑的规范数据集，用于快速 smoke test。
- 合并重复的 pipeline 脚本。
- 增加 merge/preprocess 完整性与 checkpoint 可加载性检查。
- 增加 CI（lint + smoke train/eval）。
- 更明确地记录 S4 的构建/版本固定方式。

## 🤝 贡献

1. 创建功能分支。
2. 保持 PR 范围聚焦（每个 PR 聚焦一个 pipeline/实验问题）。
3. 提供可直接运行的准确命令与预期输出路径。
4. 非必要不要提交体积较大的生成产物。
5. 添加可复现说明（seed、数据来源、checkpoint 路径）。

## 📄 许可证

本仓库当前不存在 `LICENSE` 文件。

在添加许可证文件之前，代码复用与再分发条款应视为**未确定**。

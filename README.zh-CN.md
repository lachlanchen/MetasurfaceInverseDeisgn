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

[![Status](https://img.shields.io/badge/Status-Research%20Prototype-orange)](#-项目范围)
[![Python](https://img.shields.io/badge/Python-3.9-3776AB?logo=python&logoColor=white)](#-环境配置)
[![Platform](https://img.shields.io/badge/Platform-Linux-2f2f2f?logo=linux&logoColor=white)](#-前置条件)
[![RCWA](https://img.shields.io/badge/RCWA-S4%20Required-1f9d55)](#-前置条件)
[![PyTorch](https://img.shields.io/badge/Framework-PyTorch-EE4C2C?logo=pytorch&logoColor=white)](#-训练与评估)
[![License](https://img.shields.io/badge/License-Not%20Specified-lightgrey)](#-许可证)

这是一个以脚本为核心的研究型代码库，用于 **C4 对称约束下的超表面逆向设计**。
它整合了：

- 🔬 S4/RCWA 仿真（`.lua` + Bash 启动脚本）
- 🧱 从原始光谱与形状顶点出发的数据合并与预处理
- 🧠 三阶段 PyTorch 训练（`shape -> spectra`、`spectra -> shape`、链式微调）
- 📊 评估与绘图（包含神经网络结果与 S4 结果一致性检查）

## 📌 目录

- [项目范围](#-项目范围)
- [研究背景](#-研究背景)
- [仓库结构](#️-仓库结构)
- [前置条件](#-前置条件)
- [环境配置](#️-环境配置)
- [快速开始](#-快速开始)
- [端到端流程](#-端到端流程)
- [训练与评估](#-训练与评估)
- [关键 CLI 参数](#️-关键-cli-参数)
- [故障排查](#-故障排查)
- [路线图](#-路线图)
- [引用](#-引用)
- [许可证](#-许可证)

## 🎯 项目范围

该仓库以实验驱动为主，围绕可执行脚本组织（并非打包后的库）。
当前最稳定的工作流为：

1. 运行 S4，在 `results/` 中生成原始光学输出，并在 `shapes/` 中生成形状
2. 将每次运行的 CSV 合并并透视，整理为可训练的表格数据
3. 将合并后的 CSV 转换为压缩 `.npz`
4. 训练三阶段透射率管线
5. 评估 checkpoint 并导出图表/指标

## 🧪 研究背景

### 问题设定

核心逆向设计任务是在 C4 对称约束和部分结晶扫描条件下，从目标透射光谱恢复超表面几何（反向亦成立）。

### 透射率管线中的数据假设

| 项目 | 数值 |
|---|---|
| 每个形状的结晶状态数 | 11（`c` 从 `0.0` 到 `1.0`） |
| 每个状态的光谱 bin 数 | 100 |
| 每个样本的光谱张量 | `11 x 100` |
| 每个样本的形状张量 | `4 x 3`（`[presence, x, y]`） |

### 三阶段学习目标

| 阶段 | 方向 | 典型 checkpoint |
|---|---|---|
| A | `shape -> spectrum` | `stageA/shape2spec_stageA.pt` |
| B | `spectrum -> shape` | `stageB/spec2shape_stageB.pt` |
| C | `spectrum -> shape -> spectrum`（链式微调） | `stageC/spec2shape_stageC.pt` |

## 🗂️ 仓库结构

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

## 🧩 前置条件

| 依赖项 | 说明 |
|---|---|
| Linux + Bash | Shell 脚本默认使用 Linux 风格路径 |
| Conda | 推荐用于环境管理（`iccp.yaml`） |
| Python 3.9 | 训练/评估脚本的主要运行时 |
| S4 binary | 预期路径为仓库根目录相对路径 `../build/S4` |
| CUDA（可选） | 可加速训练与评估 |

## ⚙️ 环境配置

```bash
git clone <repo-url> inverse_metasurface
cd inverse_metasurface

conda env create -f iccp.yaml
conda activate iccp

# verify S4 path expected by shell runners
ls -l ../build/S4
```

可选：为 Shell 启动脚本添加可执行权限：

```bash
chmod +x ms.sh ms_final.sh ms_resume_allargs.sh ms_resume.sh
```

## 🚀 快速开始

如果你已经有 `preprocessed_t_data.npz`：

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

## 🔁 端到端流程

### 1) 生成 S4 数据

```bash
./ms_final.sh \
  -ns 10000 \
  -r 12345 \
  -p myrun \
  -g 80 \
  -bo 0.35 \
  -ro 0.30
```

### 2) 合并 S4 输出与形状顶点

```bash
python merge_s4_data_full.py --prefix myrun
# -> merged_s4_shapes_myrun.csv
```

### 3) 为预处理规范化 CSV 列名

`three_stage_transmittance.py` 的预处理期望使用 `prefix` 与 `nQ`，而合并输出中可能是 `folder_key` 与 `NQ`。

```bash
python -c "import pandas as pd; p='merged_s4_shapes_myrun.csv'; df=pd.read_csv(p); df=df.rename(columns={'folder_key':'prefix','NQ':'nQ'}); df.to_csv(p,index=False)"
```

### 4) CSV -> NPZ 预处理

```bash
mkdir -p merged_csvs
mv merged_s4_shapes_myrun.csv merged_csvs/

python three_stage_transmittance.py --preprocess \
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

### 6) 评估与绘图

```bash
python three_stage_transmittance_evaluation.py \
  --model_dir outputs_three_stage_YYYYMMDD_HHMMSS \
  --data_npz preprocessed_t_data.npz \
  --sample_count 8
```

### 7) 可选：神经网络与 S4 对比

```bash
python FilterShapeS4_Evaluator_Transmittance.py \
  --npz_file preprocessed_t_data.npz \
  --spec2shape_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageC/spec2shape_stageC.pt \
  --shape2spec_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageA/shape2spec_stageA.pt \
  --n_samples 4
```

## 🧠 训练与评估

### 主要脚本

| 脚本 | 用途 |
|---|---|
| `three_stage_transmittance.py` | 预处理 + 训练 A/B/C 三阶段 |
| `three_stage_transmittance_evaluation.py` | 评估 checkpoint、计算指标并保存图表 |
| `FilterShapeS4_Evaluator_Transmittance.py` | 对比学习模型预测与 S4 行为 |

### 典型输出

| 产物 | 位置 |
|---|---|
| 阶段 checkpoint | `outputs_three_stage_*/stageA|stageB|stageC/` |
| 评估图像 | `outputs_three_stage_*/evaluation_<timestamp>/` |
| 指标 CSV | `evaluation_metrics.csv`, `metrics_summary.csv` |

## 🛠️ 关键 CLI 参数

### S4 启动脚本（`ms_final.sh`, `ms_resume_allargs.sh`）

| 参数 | 含义 | 默认值 |
|---|---|---|
| `-ns`, `--numshapes` | 形状数量 | `100000` |
| `-r`, `--seed` | 随机种子 | `88888` |
| `-p`, `--prefix` | 运行前缀/断点续跑键 | 空 |
| `-g`, `--numg` | 几何基设置 | `80` |
| `-bo`, `--baseouter` | 外轮廓基准偏移 | `0.25` |
| `-ro`, `--randouter` | 外轮廓随机偏移 | `0.20` |

### 训练（`three_stage_transmittance.py`）

| 参数 | 含义 | 默认值 |
|---|---|---|
| `--preprocess` | 切换到预处理模式 | 关闭 |
| `--input_folder` | 包含合并 CSV 的文件夹 | `""` |
| `--output_npz` | 预处理输出文件 | `preprocessed_data.npz` |
| `--data_npz` | 训练使用的 NPZ 输入 | `""` |
| `--csv_file` | 直接使用 CSV 训练输入 | `""` |
| `--test` | 测试模式开关 | 关闭 |
| `--num_epochs` | 每阶段 epoch 数 | `10` |
| `--batch_size` | 批大小 | `4096` |

### 评估（`three_stage_transmittance_evaluation.py`）

| 参数 | 含义 | 默认值 |
|---|---|---|
| `--model_dir` | 已训练运行目录（必填） | - |
| `--data_npz` | 评估使用的 NPZ 输入 | `""` |
| `--csv_file` | 评估使用的 CSV 输入 | `""` |
| `--output_dir` | 自定义输出目录 | 自动 |
| `--sample_count` | 可视化样本数量 | `4` |
| `--seed` | 样本选择随机种子 | `23` |
| `--font_scale` | 绘图字体缩放倍数 | `1.0` |
| `--batch_size` | 评估 dataloader 批大小 | `32` |
| `--plot_only` | 仅重新生成图表 | 关闭 |

## 🧯 故障排查

| 现象 | 常见原因 | 解决方式 |
|---|---|---|
| `../build/S4: No such file or directory` | S4 二进制不在预期相对路径 | 将 S4 放置/编译到 `../build/S4`，或修改脚本路径 |
| `Must specify either --data_npz or --csv_file` | 缺少训练/评估数据集参数 | 严格提供且仅提供一个数据输入 |
| `No transmission columns found` | 合并 CSV 缺少 `T@...` 列 | 重新运行 merge/pivot 并检查表头 |
| 预处理阶段 `KeyError: 'prefix'` | 合并输出仍使用 `folder_key`/`NQ` | 预处理前先重命名为 `prefix`/`nQ` |
| GPU OOM | batch 过大 | 降低 `--batch_size` |
| 评估时缺少 checkpoint | 阶段 checkpoint 缺失或路径错误 | 检查 `--model_dir` 下 stageA/B/C 文件是否齐全 |

## 🧭 路线图

- 统一各 merge 脚本的合并 CSV schema（`prefix`、`nQ` 命名）
- 为 merge/preprocess/checkpoint 加载添加自动化测试
- 提供单一 CLI 入口，编排完整流程
- 添加数据集/运行清单以增强可复现性
- 增加明确的开源许可证

## 📚 引用

如果该仓库对你的研究有帮助，请引用：

```bibtex
@article{chen2025inverse,
  title={Inverse Design of Metasurface for Spectral Imaging},
  author={Chen, Rongzhou and Nie, Haitao and Zhu, Shuo and Zhao, Yaping and Wang, Chutian and Lam, Edmund Y},
  journal={arXiv preprint arXiv:2510.21924},
  year={2025}
}
```

## 📄 许可证

当前仓库中尚未包含 `LICENSE` 文件，因此在添加许可证前，使用与再分发权限均未明确。

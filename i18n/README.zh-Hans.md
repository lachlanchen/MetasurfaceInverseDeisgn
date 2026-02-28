[English](../README.md) · [العربية](README.ar.md) · [Español](README.es.md) · [Français](README.fr.md) · [日本語](README.ja.md) · [한국어](README.ko.md) · [Tiếng Việt](README.vi.md) · [中文 (简体)](README.zh-Hans.md) · [中文（繁體）](README.zh-Hant.md) · [Deutsch](README.de.md) · [Русский](README.ru.md)


# 用于光谱成像的超表面逆向设计

<p align="center">
  <img alt="Status" src="https://img.shields.io/badge/Status-Research%20Prototype-f59e0b?style=for-the-badge">
  <img alt="Python" src="https://img.shields.io/badge/Python-3.9-3776AB?style=for-the-badge&logo=python&logoColor=white">
  <img alt="PyTorch" src="https://img.shields.io/badge/PyTorch-2.x-EE4C2C?style=for-the-badge&logo=pytorch&logoColor=white">
  <img alt="Simulator" src="https://img.shields.io/badge/Simulator-S4%20RCWA-16a34a?style=for-the-badge">
  <img alt="Platform" src="https://img.shields.io/badge/Platform-Linux%20%2B%20Bash-6b7280?style=for-the-badge&logo=gnu-bash&logoColor=white">
  <img alt="ArXiv" src="https://img.shields.io/badge/arXiv-2510.21924-b31b1b?style=for-the-badge&logo=arxiv&logoColor=white">
</p>

这是一个以脚本为核心的研究仓库（历史上称为 `inverse_metasurface`），用于光谱成像中的超表面逆向设计。

核心工作流耦合了：
- 基于物理的 RCWA 仿真（`S4` + Lua）
- 数据汇总与形状信息附加
- 三阶段 PyTorch 学习（`shape -> spectrum`、`spectrum -> shape`、链式微调）
- 定量与定性评估，并可选执行 neural-vs-S4 一致性检查

> [!IMPORTANT]
> 本仓库保留了现有项目脚本/文档中的规范行为与命令。若历史引用指向当前快照中缺失的文件，这些引用会带明确说明并被有意保留，以确保兼容性。

## 📑 目录

- [✨ 快速概览](#-快速概览)
- [🌍 国际化 (i18n)](#-国际化-i18n)
- [✨ 特性](#-特性)
- [🧭 端到端工作流](#-端到端工作流)
- [🧱 项目结构](#-项目结构)
- [🛠️ 前置条件](#️-前置条件)
- [🚀 安装](#-安装)
- [▶️ 使用方法](#️-使用方法)
- [⚙️ 配置](#️-配置)
- [🧪 示例](#-示例)
- [🔬 研究背景](#-研究背景)
- [🧑‍💻 开发说明](#-开发说明)
- [🧯 故障排查](#-故障排查)
- [🗺️ 路线图](#️-路线图)
- [🤝 贡献](#-贡献)
- [📄 许可证](#-许可证)
- [📚 引用](#-引用)

## ✨ 快速概览

| 项目 | 说明 |
|---|---|
| 🎯 主要任务 | 从目标透射谱推断 C4 对称超表面几何形状 |
| 🔬 仿真器 | 由 shell 启动脚本和 `.lua` 脚本调用的 `../build/S4` |
| 🧠 学习流水线 | Stage A `shape -> spectra`，Stage B `spectra -> shape`，Stage C `spectra -> shape -> spectra` |
| 📦 数据约定 | 合并 CSV（`T@...`、元数据、`vertices_str`）-> 压缩 NPZ（`uids`、`spectra`、`shapes`） |
| 🧪 评估 | MSE 指标、分阶段可视化、可选重新运行 S4 对比 |
| 🌐 i18n 状态 | 仓库根目录多语言 README + 已存在的 `i18n/` 目录 |

## 🌍 国际化 (i18n)

- 多语言 README 维护在仓库根目录，命名为 `README.<lang>.md`。
- 当前仓库快照中存在 `i18n/` 目录。
- 本文件在顶部仅保留一行语言选项，避免重复语言栏。
- 仓库中也存在 `README.en.md`；本次更新以 `README.md` 作为规范基准。

## ✨ 特性

- 从 S4 仿真输出到逆向模型训练的端到端逆向设计流程。
- C4 对称多边形参数化与 Q1 点编码（`4x3`: `presence, x, y`）。
- 在单一脚本（`three_stage_transmittance.py`）中完成三阶段模型训练。
- 可在保留光谱精度的同时附加每个形状的顶点信息的合并工具。
- 可选评估器可将学习预测结果与新运行的 S4 仿真进行比较。
- 包含大量探索性分支（AVIRIS、SWIR/noise、GSST、archived/deprecated 变体）。

## 🧭 端到端工作流

1. 在 `results/` 生成仿真输出，并在 `shapes/` 生成多边形文件。
2. 合并 S4 CSV 文件并附加形状顶点。
3. 规范化合并后的列名以兼容训练流程。
4. 将合并 CSV 预处理为 NPZ 张量。
5. 训练 Stage A/B/C 模型。
6. 评估 checkpoint 并进行可视化分析。
7. 可选：将预测形状的光谱与新的 S4 运行结果对比。

## 🧱 项目结构

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

## 🛠️ 前置条件

| 依赖项 | 说明 |
|---|---|
| Linux + Bash | 启动脚本面向 shell 执行 |
| Python 3.9 | 与 `iccp.yaml`（`python=3.9.18`）一致 |
| Conda | 推荐用于复现 |
| S4 binary | 期望位于 `../build/S4` |
| CUDA GPU（可选） | 可加速训练/评估 |

## 🚀 安装

### 1) 克隆并进入目录

```bash
git clone <your-repo-url> inverse_metasurface
cd inverse_metasurface
```

### 2) 创建环境（推荐）

```bash
conda env create -f iccp.yaml
conda activate iccp
```

替代说明：

```bash
# Historical README reference (file may be absent in this snapshot)
pip install -r pip_requirements.txt
```

### 3) 验证脚本期望的仿真器路径

```bash
ls -l ../build/S4
```

### 4)（可选）为启动脚本添加可执行权限

```bash
chmod +x ms.sh ms_final.sh ms_resume.sh ms_resume_allargs.sh ms_resume_random_state.sh ms_resume_random_state_nir.sh
```

## ▶️ 使用方法

### A) 生成 RCWA 仿真数据

简易启动：

```bash
./ms.sh -ns 10000 -r 12345
```

参数化启动：

```bash
./ms_final.sh \
  -ns 10000 \
  -r 12345 \
  -p myrun \
  -g 80 \
  -bo 0.35 \
  -ro 0.30
```

面向续跑的启动：

```bash
./ms_resume_allargs.sh \
  -ns 10000 \
  -r 12345 \
  -p myrun \
  -g 80 \
  -bo 0.35 \
  -ro 0.30
```

额外的续跑/随机状态示例（来自命令文档）：

```bash
./ms_resume_random_state.sh \
  -p iccp100kG20Ov \
  -r 88888 \
  -g 20 \
  -bo 0.35 \
  -ro 0.3 \
  -ns 100000
```

说明：
- 启动脚本会并行运行 `NQ=1..4`。
- 脚本调用 `../build/S4` 时使用 `-t 32`。

### B) 合并 S4 输出并附加形状顶点

```bash
python merge_s4_data_full.py --prefix myrun
# output: merged_s4_shapes_myrun.csv
```

### C) 规范化列名以兼容训练

`merge_s4_data_full.py` 输出 `folder_key` 和 `NQ`，而训练流程期望 `prefix` 和 `nQ`。

```bash
python -c "import pandas as pd; p='merged_s4_shapes_myrun.csv'; df=pd.read_csv(p); df=df.rename(columns={'folder_key':'prefix','NQ':'nQ'}); df.to_csv(p,index=False)"
```

### D) 预处理 CSV -> NPZ

```bash
mkdir -p merged_csvs
mv merged_s4_shapes_myrun.csv merged_csvs/

python three_stage_transmittance.py \
  --preprocess \
  --input_folder merged_csvs \
  --output_npz preprocessed_t_data.npz
```

### E) 训练 Stage A/B/C

```bash
python three_stage_transmittance.py \
  --data_npz preprocessed_t_data.npz \
  --num_epochs 100 \
  --batch_size 1024
```

输出会写入：
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageA`
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageB`
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageC`

### F) 评估训练好的模型

历史 README 命令（为兼容旧文档保留脚本名）：

```bash
python three_stage_transmittance_evaluation.py \
  --model_dir outputs_three_stage_YYYYMMDD_HHMMSS \
  --data_npz preprocessed_t_data.npz \
  --sample_count 8
```

仓库状态说明：当前快照中不存在 `three_stage_transmittance_evaluation.py`。可使用 `FilterShapeS4_Evaluator_Transmittance.py` 获得可用评估功能。

### G) 可选的 neural-vs-S4 一致性检查

```bash
python FilterShapeS4_Evaluator_Transmittance.py \
  --npz_file preprocessed_t_data.npz \
  --spec2shape_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageC/spec2shape_stageC.pt \
  --shape2spec_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageA/shape2spec_stageA.pt \
  --n_samples 4
```

## ⚙️ 配置

### S4 启动脚本（`ms_final.sh`, `ms_resume_allargs.sh`）

| Flag | 含义 | Default |
|---|---|---|
| `-ns`, `--numshapes` | 生成的形状数量 | `100000` |
| `-r`, `--seed` | 随机种子 | `88888` |
| `-p`, `--prefix` | 前缀/续跑键 | `""` |
| `-g`, `--numg` | 基函数/网格参数 | `80` |
| `-bo`, `--baseouter` | 外边界基础偏移 | `0.25` |
| `-ro`, `--randouter` | 外边界随机偏移 | `0.20` |

### 训练（`three_stage_transmittance.py`）

| Flag | 含义 | Default |
|---|---|---|
| `--preprocess` | 运行预处理模式 | `False` |
| `--input_folder` | 包含合并 CSV 的文件夹 | `""` |
| `--output_npz` | 输出 NPZ 路径 | `preprocessed_data.npz` |
| `--data_npz` | 用于训练的 NPZ 数据集 | `""` |
| `--csv_file` | 不使用 NPZ 时的 CSV 回退输入 | `""` |
| `--test` | 测试模式 | `False` |
| `--num_epochs` | 训练轮数 | `10` |
| `--batch_size` | 批大小 | `4096` |

### 历史评估配置（`three_stage_transmittance_evaluation.py`）

| Flag | 含义 | Default |
|---|---|---|
| `--model_dir` | 包含 `stageA/B/C` 的目录 | required |
| `--data_npz` | NPZ 输入 | `""` |
| `--csv_file` | CSV 输入回退 | `""` |
| `--output_dir` | 覆盖输出目录 | 自动位于 `model_dir` 下 |
| `--sample_count` | 可视化样本数 | `4` |
| `--seed` | 随机种子 | `23` |
| `--font_scale` | 绘图字体缩放 | `1.0` |
| `--batch_size` | 评估批大小 | `32` |
| `--plot_only` | 仅绘制训练曲线 | `False` |

### S4 一致性评估器（`FilterShapeS4_Evaluator_Transmittance.py`）

| Flag | 含义 | Default |
|---|---|---|
| `--npz_file` | 输入 NPZ 文件 | `preprocessed_t_data.npz` |
| `--spec2shape_ckpt` | Stage C checkpoint 路径 | `outputs_three_stage_20250322_145925/stageC/spec2shape_stageC.pt` |
| `--shape2spec_ckpt` | Stage A checkpoint 路径 | `outputs_three_stage_20250322_145925/stageA/shape2spec_stageA.pt` |
| `--n_samples` | 评估样本数 | `4` |
| `--seed` | 随机种子 | `23` |
| `--max_workers` | S4 工作线程数 | `4` |
| `--out_folder` | 输出目录 | 自动时间戳 |

## 🧪 示例

### Smoke run

```bash
./ms_final.sh -ns 1000 -r 42 -p smoke -g 40 -bo 0.25 -ro 0.20
python merge_s4_data_full.py --prefix smoke
python -c "import pandas as pd; p='merged_s4_shapes_smoke.csv'; d=pd.read_csv(p).rename(columns={'folder_key':'prefix','NQ':'nQ'}); d.to_csv(p,index=False)"
mkdir -p merged_csvs && mv merged_s4_shapes_smoke.csv merged_csvs/
python three_stage_transmittance.py --preprocess --input_folder merged_csvs --output_npz smoke.npz
python three_stage_transmittance.py --data_npz smoke.npz --num_epochs 5 --batch_size 128
```

### 运行配置示例（来自 `commands.md` / `commands_updated.md`）

```bash
# no overlap, G=40
./ms_resume_allargs.sh -ns 10000 -r 12345 -p fast_without_overlap -g 40 -bo 0.25 -ro 0.2

# overlap, G=80
./ms_resume_allargs.sh -ns 10000 -r 12345 -p overlap -g 80 -bo 0.35 -ro 0.3

# large run, overlap, G=80
./ms_resume_allargs.sh -ns 100000 -r 12345 -p more_basis_overlap -g 80 -bo 0.35 -ro 0.3
```

## 🔬 研究背景

当前逆向设计配置学习从跨结晶状态的透射率数据恢复 C4 对称几何形状。当前透射率流水线默认：

- 每个形状样本对应 11 行结晶状态数据（按唯一 `shape_uid` 分组）
- 每个结晶状态包含 100 个波长 bin（`T@...` 列）
- 最多 4 个 Q1 控制点，编码为 `4x3` 张量：`(presence, x, y)`
- 在 C4 对称约束下进行多边形重建，用于形状可视化和一致性检查

除主透射率训练路径外，仓库还包含探索性分支（`AVIRIS*`、`noise_experiment*`、`archived/`）。

## 🧑‍💻 开发说明

- 这是一个以脚本为中心的研究仓库，而不是打包后的 Python 模块。
- 核心脚本依赖相对路径（尤其是 `../build/S4`、`results/`、`shapes/`）。
- `.gitignore` 排除了大量生成的实验产物（`*.csv`、`*.npz`、`*.pt`、运行目录）。
- 历史文档中的某些文件/目录在当前快照中缺失；这些引用已附说明并有意保留以确保兼容。
- 仓库中存在 macOS sidecar 文件（`._*`），可能是无功能的元数据产物。

## 🧯 故障排查

| 症状 | 可能原因 | 修复方法 |
|---|---|---|
| `../build/S4: No such file or directory` | 预期相对路径下缺少 S4 binary | 在 `../build/S4` 构建或链接 S4，或更新启动脚本路径 |
| `No transmission columns found` | CSV 缺少 `T@...` 列 | 重新检查 merge 输出格式 |
| `Must specify either --data_npz or --csv_file` | 缺少训练/评估数据参数 | 显式提供其中一个输入 |
| `No valid shapes => SHIFT->Q1->UpTo4` | `vertices_str` 无效/为空，或 Q1 过滤后样本全被移除 | 校验形状文件和 merge 输出 |
| Evaluation checkpoint missing | 缺少 `stageA/B/C` checkpoint 文件 | 确认 `--model_dir` 指向完整输出目录 |
| `three_stage_transmittance_evaluation.py` not found | 历史文档引用该脚本，但当前不存在 | 使用 `FilterShapeS4_Evaluator_Transmittance.py`，或从旧提交恢复该脚本 |

## 🗺️ 路线图

- 通过明确的数据版本清单和固定运行配置提升可复现性。
- 整理 transmittance、AVIRIS、noise 分支的规范入口点。
- 为预处理和一次最小训练 epoch 增加自动化 smoke test。
- 增加更清晰的实验注册信息，将输出目录与精确命令行关联。
- 扩展多语言 README 同步工作流（根目录语言文件与 `i18n/`）。

## 🤝 贡献

欢迎贡献，尤其是可复现性、测试和文档质量方面。

建议流程：

1. 先提交 issue，说明范围和预期行为。
2. 创建聚焦的分支。
3. 提交 pull request，并附可运行命令与输出结果。
4. 尽量将变更范围限制在单一工作流内。

## 📄 许可证

当前仓库快照的根目录中没有 `LICENSE` 文件。请添加许可证以明确使用和再分发条款。

## 📚 引用

如果你使用了本仓库或基于本工作继续研究，请引用：

```bibtex
@article{chen2025inverse,
  title={Inverse Design of Metasurface for Spectral Imaging},
  author={Chen, Rongzhou and Nie, Haitao and Zhu, Shuo and Zhao, Yaping and Wang, Chutian and Lam, Edmund Y},
  journal={arXiv preprint arXiv:2510.21924},
  year={2025}
}
```

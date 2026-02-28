[English](../README.md) · [العربية](README.ar.md) · [Español](README.es.md) · [Français](README.fr.md) · [日本語](README.ja.md) · [한국어](README.ko.md) · [Tiếng Việt](README.vi.md) · [中文 (简体)](README.zh-Hans.md) · [中文（繁體）](README.zh-Hant.md) · [Deutsch](README.de.md) · [Русский](README.ru.md)


[![LazyingArt banner](https://github.com/lachlanchen/lachlanchen/raw/main/figs/banner.png)](https://github.com/lachlanchen/lachlanchen/blob/main/figs/banner.png)

# 谱成像中超表面逆向设计

<p align="center">
  <img alt="Status" src="https://img.shields.io/badge/Status-Research%20Prototype-f59e0b?style=for-the-badge">
  <img alt="Python" src="https://img.shields.io/badge/Python-3.9-3776AB?style=for-the-badge&logo=python&logoColor=white">
  <img alt="PyTorch" src="https://img.shields.io/badge/PyTorch-2.x-EE4C2C?style=for-the-badge&logo=pytorch&logoColor=white">
  <img alt="Simulator" src="https://img.shields.io/badge/Simulator-S4%20RCWA-16a34a?style=for-the-badge">
  <img alt="Platform" src="https://img.shields.io/badge/Platform-Linux%20%2B%20Bash-6b7280?style=for-the-badge&logo=gnu-bash&logoColor=white">
  <img alt="ArXiv" src="https://img.shields.io/badge/arXiv-2510.21924-b31b1b?style=for-the-badge&logo=arxiv&logoColor=white">
</p>

这是一个以脚本为先的研究仓库（历史上称作 `inverse_metasurface`），用于谱成像中的超表面逆向设计。

核心流程包括：
- 以物理为基础的 RCWA 仿真（`S4` + Lua）
- 数据汇总与形状顶点附加
- 三阶段 PyTorch 学习（`shape -> spectrum`、`spectrum -> shape`、级联微调）
- 定量与定性评估，并可选进行神经网络与 S4 的一致性再检查

> [!IMPORTANT]
> 规范行为与命令沿用现有项目脚本/文档。历史引用如果指向当前快照中不存在的文件，这些引用会保留并带有明确说明，以兼容历史流程。

## 📑 目录

- [🌟 Snapshot](#-snapshot)
- [✨ At a Glance](#-at-a-glance)
- [🌍 Internationalization (i18n)](#-internationalization-i18n)
- [✨ Features](#-features)
- [🧭 End-to-End Workflow](#-end-to-end-workflow)
- [🧱 Project Structure](#-project-structure)
- [🛠️ Prerequisites](#️-prerequisites)
- [🚀 Installation](#-installation)
- [▶️ Usage](#️-usage)
- [⚙️ Configuration](#️-configuration)
- [🧪 Examples](#-examples)
- [🔬 Research Context](#-research-context)
- [🧑‍💻 Development Notes](#-development-notes)
- [🧯 Troubleshooting](#-troubleshooting)
- [🗺️ Roadmap](#️-roadmap)
- [🤝 Contribution](#-contribution)
- [📄 License](#-license)
- [📚 Citation](#-citation)

## 🌟 Snapshot

| 重点 | 状态 |
|---|---|
| 🧠 目标 | 从光谱数据中反推出 C4 对称超表面几何 |
| 🔧 核心技术栈 | S4 RCWA（`Lua`） + PyTorch 训练 + 可选几何到光谱重验证 |
| 🧪 数据管线 | CSV 合并/形状顶点附加 → NPZ（`uids`、`spectra`、`shapes`） |
| 🚀 就绪度 | 研究型原型；脚本与文档保持与历史参考兼容 |

## ✨ At a Glance

| 项目 | 说明 |
|---|---|
| 🎯 主要任务 | 从目标透射谱反演 C4 对称超表面几何 |
| 🔬 仿真器 | 由 shell 启动脚本和 `.lua` 脚本调用的 `../build/S4` |
| 🧠 学习流水线 | Stage A `shape -> spectra`、Stage B `spectra -> shape`、Stage C `spectra -> shape -> spectra` |
| 📦 数据约定 | 合并 CSV（`T@...`、元数据、`vertices_str`）→ 压缩 NPZ（`uids`、`spectra`、`shapes`） |
| 🧪 评估 | MSE 指标、阶段可视化、可选重新运行 S4 仿真 |
| 🌐 i18n 状态 | 仓库根目录有多语言 README 文件，且存在 `i18n/` 目录 |

## 🌍 Internationalization (i18n)

- 多语言 README 维护为仓库根目录下的 `README.<lang>.md`。
- 仓库快照中存在 `i18n/` 目录。
- 该文件在顶部仅保留一行语言列表，避免重复语言导航。
- 仓库中也存在 `README.en.md`；当前 `README.md` 保持本次更新的规范基线。

## ✨ Features

- 从 S4 仿真输出到逆向模型训练的端到端逆向设计链路。
- C4 对称多边形参数化与 Q1 点编码（`4x3`：`presence, x, y`）。
- 一个脚本内完成三阶段模型训练（`three_stage_transmittance.py`）。
- 合并工具在保留光谱精度的同时附加每个形状的顶点。
- 可选评估器，用于比较学习结果与新跑的 S4 仿真。
- 包含大量探索分支（AVIRIS、SWIR/noise、GSST、已归档/已弃用变体）。

## 🧭 End-to-End Workflow

1. 在 `results/` 生成仿真输出，并在 `shapes/` 生成多边形文件。
2. 合并 S4 CSV 文件并附加形状顶点。
3. 归一化合并后的列名以兼容训练。
4. 将合并后的 CSV 预处理为 NPZ 张量。
5. 训练 Stage A/B/C 模型。
6. 评估检查点并可视化行为。
7. 可选对比预测形状光谱与新一轮 S4 计算。

## 🧱 Project Structure

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

## 🛠️ Prerequisites

| 依赖项 | 说明 |
|---|---|
| Linux + Bash | 启动脚本以 shell 执行为目标 |
| Python 3.9 | 与 `iccp.yaml` 一致（`python=3.9.18`） |
| Conda | 建议用于复现实验 |
| S4 二进制 | 预期位于 `../build/S4` |
| CUDA GPU（可选） | 可加速训练与评估 |

## 🚀 Installation

### 1) 克隆并进入

```bash
git clone <your-repo-url> inverse_metasurface
cd inverse_metasurface
```

### 2) 创建环境（推荐）

```bash
conda env create -f iccp.yaml
conda activate iccp
```

补充说明：

```bash
# Historical README reference (file may be absent in this snapshot)
pip install -r pip_requirements.txt
```

### 3) 核实脚本默认的仿真器路径

```bash
ls -l ../build/S4
```

### 4) （可选）赋予启动脚本可执行权限

```bash
chmod +x ms.sh ms_final.sh ms_resume.sh ms_resume_allargs.sh ms_resume_random_state.sh ms_resume_random_state_nir.sh
```

## ▶️ Usage

### A) 生成 RCWA 仿真数据

基础启动器：

```bash
./ms.sh -ns 10000 -r 12345
```

带参数启动器：

```bash
./ms_final.sh \
  -ns 10000 \
  -r 12345 \
  -p myrun \
  -g 80 \
  -bo 0.35 \
  -ro 0.30
```

继续运行启动器：

```bash
./ms_resume_allargs.sh \
  -ns 10000 \
  -r 12345 \
  -p myrun \
  -g 80 \
  -bo 0.35 \
  -ro 0.30
```

额外的 resume/random-state 示例（来自命令文档）：

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
- 启动器并行运行 `NQ=1..4`。
- 脚本以 `-t 32` 调用 `../build/S4`。

### B) 合并 S4 输出并附加形状顶点

```bash
python merge_s4_data_full.py --prefix myrun
# output: merged_s4_shapes_myrun.csv
```

### C) 规范训练列名

`merge_s4_data_full.py` 输出 `folder_key` 和 `NQ`，而训练链路期望的是 `prefix` 和 `nQ`。

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

输出路径为：
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageA`
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageB`
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageC`

### F) 评估训练模型

历史 README 命令（为兼容历史文档保留脚本名）：

```bash
python three_stage_transmittance_evaluation.py \
  --model_dir outputs_three_stage_YYYYMMDD_HHMMSS \
  --data_npz preprocessed_t_data.npz \
  --sample_count 8
```

仓库状态说明：当前快照中未提供 `three_stage_transmittance_evaluation.py`。可使用 `FilterShapeS4_Evaluator_Transmittance.py` 使用可用的评估功能。

### G) 可选的神经网络与 S4 一致性检查

```bash
python FilterShapeS4_Evaluator_Transmittance.py \
  --npz_file preprocessed_t_data.npz \
  --spec2shape_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageC/spec2shape_stageC.pt \
  --shape2spec_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageA/shape2spec_stageA.pt \
  --n_samples 4
```

## ⚙️ Configuration

### S4 启动器（`ms_final.sh`、`ms_resume_allargs.sh`）

| 参数 | 含义 | 默认值 |
|---|---|---|
| `-ns`, `--numshapes` | 生成形状数量 | `100000` |
| `-r`, `--seed` | 随机种子 | `88888` |
| `-p`, `--prefix` | 前缀 / 恢复标识 | `""` |
| `-g`, `--numg` | 基础/网格参数 | `80` |
| `-bo`, `--baseouter` | 外部边界基准偏移 | `0.25` |
| `-ro`, `--randouter` | 外部边界随机偏移 | `0.20` |

### 学习参数（`three_stage_transmittance.py`）

| 参数 | 含义 | 默认值 |
|---|---|---|
| `--preprocess` | 启用预处理模式 | `False` |
| `--input_folder` | 存放合并 CSV 的文件夹 | `""` |
| `--output_npz` | 输出 NPZ 路径 | `preprocessed_data.npz` |
| `--data_npz` | 训练用 NPZ 数据集 | `""` |
| `--csv_file` | 未使用 NPZ 时的 CSV 回退 | `""` |
| `--test` | 测试模式 | `False` |
| `--num_epochs` | 训练 epoch 数 | `10` |
| `--batch_size` | 批大小 | `4096` |

### 历史评估配置（`three_stage_transmittance_evaluation.py`）

| 参数 | 含义 | 默认值 |
|---|---|---|
| `--model_dir` | 包含 `stageA/B/C` 的目录 | required |
| `--data_npz` | NPZ 输入 | `""` |
| `--csv_file` | CSV 回退输入 | `""` |
| `--output_dir` | 输出目录覆盖 | `model_dir` 下自动生成 |
| `--sample_count` | 可视化样本数 | `4` |
| `--seed` | 随机种子 | `23` |
| `--font_scale` | 绘图字体缩放 | `1.0` |
| `--batch_size` | 评估批量大小 | `32` |
| `--plot_only` | 仅绘制训练曲线 | `False` |

### S4 一致性评估器（`FilterShapeS4_Evaluator_Transmittance.py`）

| 参数 | 含义 | 默认值 |
|---|---|---|
| `--npz_file` | 输入 NPZ 文件 | `preprocessed_t_data.npz` |
| `--spec2shape_ckpt` | Stage C 检查点路径 | `outputs_three_stage_20250322_145925/stageC/spec2shape_stageC.pt` |
| `--shape2spec_ckpt` | Stage A 检查点路径 | `outputs_three_stage_20250322_145925/stageA/shape2spec_stageA.pt` |
| `--n_samples` | 评估样本数 | `4` |
| `--seed` | 随机种子 | `23` |
| `--max_workers` | S4 工作线程数 | `4` |
| `--out_folder` | 输出目录 | 自动时间戳 |

## 🧪 Examples

### 冒烟运行

```bash
./ms_final.sh -ns 1000 -r 42 -p smoke -g 40 -bo 0.25 -ro 0.20
python merge_s4_data_full.py --prefix smoke
python -c "import pandas as pd; p='merged_s4_shapes_smoke.csv'; d=pd.read_csv(p).rename(columns={'folder_key':'prefix','NQ':'nQ'}); d.to_csv(p,index=False)"
mkdir -p merged_csvs && mv merged_s4_shapes_smoke.csv merged_csvs/
python three_stage_transmittance.py --preprocess --input_folder merged_csvs --output_npz smoke.npz
python three_stage_transmittance.py --data_npz smoke.npz --num_epochs 5 --batch_size 128
```

### 运行参数示例（来自 `commands.md` / `commands_updated.md`）

```bash
# no overlap, G=40
./ms_resume_allargs.sh -ns 10000 -r 12345 -p fast_without_overlap -g 40 -bo 0.25 -ro 0.2

# overlap, G=80
./ms_resume_allargs.sh -ns 10000 -r 12345 -p overlap -g 80 -bo 0.35 -ro 0.3

# large run, overlap, G=80
./ms_resume_allargs.sh -ns 100000 -r 12345 -p more_basis_overlap -g 80 -bo 0.35 -ro 0.3
```

## 🔬 Research Context

当前逆向设计流程学习在结晶化状态下从透射率恢复 C4 对称几何。透射率流水线目前假定：

- 每个形状样本有 11 行结晶化状态（按唯一 `shape_uid` 分组）
- 每个结晶化状态有 100 个波长采样点（`T@...` 列）
- 使用 `4x3` 张量编码最多 4 个 Q1 控制点：`(presence, x, y)`
- 为形状可视化和一致性检查进行 C4 对称下的多边形重建

仓库还包含除主要透射率训练路径之外的探索分支（`AVIRIS*`、`noise_experiment*`、`archived/`）。

## 🧑‍💻 Development Notes

- 这是一个以脚本为核心的研究仓库，而非打包后的 Python 模块。
- 核心脚本依赖相对路径（尤其是 `../build/S4`、`results/`、`shapes/`）。
- `.gitignore` 中排除了大量生成的实验产物（`*.csv`、`*.npz`、`*.pt`、运行文件夹）。
- 历史文档中的某些文件/目录在当前快照中缺失；这些引用保留并注明了兼容说明。
- macOS 侧边文件（`._*`）仍可能出现，通常是无功能的元数据文件。

## 🧯 Troubleshooting

| 症状 | 可能原因 | 解决办法 |
|---|---|---|
| `../build/S4: No such file or directory` | 期望路径下缺少 S4 二进制 | 在 `../build/S4` 构建或链接 S4，或更新启动脚本路径 |
| `No transmission columns found` | CSV 中缺少 `T@...` 列 | 复核合并输出格式 |
| `Must specify either --data_npz or --csv_file` | 缺少训练/评估数据参数 | 明确提供其中一个输入 |
| `No valid shapes => SHIFT->Q1->UpTo4` | `vertices_str` 无效或空，或 Q1 过滤后无有效样本 | 检查形状文件和合并输出 |
| `Empty merge output for --prefix` | `--prefix` 在 `results/` 中未匹配到文件 | 检查前缀是否精确一致后重跑合并 |
| Evaluation checkpoint missing | 缺少 `stageA/B/C` 检查点文件 | 确保 `--model_dir` 指向完整输出目录 |
| `three_stage_transmittance_evaluation.py` not found | 历史文档引用的脚本当前不存在 | 改用 `FilterShapeS4_Evaluator_Transmittance.py`，或从历史提交恢复 |

## 🗺️ Roadmap

- 通过明确的数据版本清单和固定运行配置提升可复现性。
- 整理 transmittance、AVIRIS、noise 分支的统一主入口。
- 为预处理和单轮小规模训练补充自动化冒烟测试。
- 澄清实验注册，绑定输出目录与精确命令行。
- 扩展多语言 README 同步工作流（根目录语言文件与 `i18n/`）。

## 🤝 Contribution

欢迎贡献，特别是在复现性、测试与文档质量方面。

建议流程：

1. 在 issue 中说明范围和预期行为。
2. 创建聚焦分支。
3. 提交可运行的命令与输出结果。
4. 尽量将改动限定在单一工作流。

## ❤️ Support

| Donate | PayPal | Stripe |
| --- | --- | --- |
| [![Donate](https://camo.githubusercontent.com/24a4914f0b42c6f435f9e101621f1e52535b02c225764b2f6cc99416926004b7/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f446f6e6174652d4c617a79696e674172742d3045413545393f7374796c653d666f722d7468652d6261646765266c6f676f3d6b6f2d6669266c6f676f436f6c6f723d7768697465)](https://chat.lazying.art/donate) | [![PayPal](https://camo.githubusercontent.com/d0f57e8b016517a4b06961b24d0ca87d62fdba16e18bbdb6aba28e978dc0ea21/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f50617950616c2d526f6e677a686f754368656e2d3030343537433f7374796c653d666f722d7468652d6261646765266c6f676f3d70617970616c266c6f676f436f6c6f723d7768697465)](https://paypal.me/RongzhouChen) | [![Stripe](https://camo.githubusercontent.com/1152dfe04b6943afe3a8d2953676749603fb9f95e24088c92c97a01a897b4942/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f5374726970652d446f6e6174652d3633354246463f7374796c653d666f722d7468652d6261646765266c6f676f3d737472697065266c6f676f436f6c6f723d7768697465)](https://buy.stripe.com/aFadR8gIaflgfQV6T4fw400) |

## 📄 License

当前快照的仓库根目录尚未包含 `LICENSE` 文件。请补充该文件以定义使用与再分发条款。

## 📚 Citation

如果您使用本仓库或基于本工作展开研究，请引用：

```bibtex
@article{chen2025inverse,
  title={Inverse Design of Metasurface for Spectral Imaging},
  author={Chen, Rongzhou and Nie, Haitao and Zhu, Shuo and Zhao, Yaping and Wang, Chutian and Lam, Edmund Y},
  journal={arXiv preprint arXiv:2510.21924},
  year={2025}
}
```

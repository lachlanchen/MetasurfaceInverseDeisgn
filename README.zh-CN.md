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
  <img alt="Simulator" src="https://img.shields.io/badge/RCWA-S4-16a34a?style=for-the-badge">
  <img alt="Platform" src="https://img.shields.io/badge/Platform-Linux%2FBash-6b7280?style=for-the-badge&logo=gnu-bash&logoColor=white">
</p>

这是一个以脚本为核心的研究仓库（历史上也称为 `inverse_metasurface`），用于光谱成像中的超表面逆向设计。整体流程将**基于 S4 的 RCWA 仿真**与**三阶段 PyTorch 工作流**结合起来，在几何结构与光学透射谱之间建立正向与逆向映射。

## ✨ 快速概览

| 项目 | 说明 |
|---|---|
| 🎯 目标 | 根据目标透射率光谱预测具备 C4 对称性的超表面几何结构 |
| 🔬 物理仿真 | 使用 S4 (`../build/S4`) 进行 RCWA 仿真 |
| 🧠 学习流程 | 阶段 A `shape -> spectra`，阶段 B `spectra -> shape`，阶段 C `spectra -> shape -> spectra` |
| 📦 数据形式 | 合并后的 CSV（`T@...`、形状元数据）-> 压缩 NPZ（`uids`、`spectra`、`shapes`） |
| 🧪 评估方式 | MSE 指标、定性可视化、可选的 S4 复仿真一致性检查 |

## 🧭 端到端流程

1. 使用 S4（`.lua` + shell 启动脚本）生成超表面光学响应。
2. 合并原始仿真 CSV 文件并附加多边形顶点信息。
3. 将合并后的 CSV 转为训练用 NPZ。
4. 训练三阶段透射率管线。
5. 评估检查点并可视化阶段 A/B/C 的行为。
6. 可选：将神经网络预测与新的 S4 仿真结果进行比较。

## 🧱 仓库结构

```text
.
├── README.md
├── how_to_run.md
├── iccp.yaml
├── pip_requirements.txt
│
├── ms.sh
├── ms_final.sh
├── ms_resume_allargs.sh
├── metasurface_seed.lua
├── metasurface_final.lua
├── metasurface_allargs_resume.lua
│
├── merge_s4_data_full.py
├── three_stage_transmittance.py
├── three_stage_transmittance_evaluation.py
├── FilterShapeS4_Evaluator_Transmittance.py
│
├── results/                # S4 原始 CSV 输出
├── shapes/                 # 合并阶段使用的多边形顶点文件
├── merged_csvs/            # 合并后的 CSV 数据集
├── outputs_three_stage_*/  # 训练检查点与曲线
├── partial_crys_data/      # 结晶状态光学表
│
├── AVIRIS*/                # 次级高光谱实验
├── noise_experiment_*/     # 鲁棒性/噪声分支
└── archived/               # 历史脚本与快照
```

## 🛠️ 先决条件

- Linux + Bash
- Conda（推荐）
- Python 3.9
- `../build/S4` 路径下可用的 S4 二进制程序
- 可选：支持 CUDA 的 GPU（可加速训练）

## 🚀 环境配置

```bash
git clone <your-repo-url> inverse_metasurface
cd inverse_metasurface

conda env create -f iccp.yaml
conda activate iccp

# 验证脚本预期的仿真器路径
ls -l ../build/S4
```

可选：

```bash
chmod +x ms.sh ms_final.sh ms_resume_allargs.sh
```

## ▶️ 实用用法

### 1) 生成 RCWA 数据

`ms_final.sh` 与 `ms_resume_allargs.sh` 都会并行启动 4 个 S4 任务（`NQ=1..4`）：

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
- `ms.sh` 是更简化的启动路径。
- 启动脚本默认使用 `../build/S4` 且带 `-t 32`。

### 2) 合并 S4 输出与形状顶点

```bash
python merge_s4_data_full.py --prefix myrun
# output: merged_s4_shapes_myrun.csv
```

### 3) 为训练兼容性规范化列名

`merge_s4_data_full.py` 输出 `folder_key` / `NQ`，而 `three_stage_transmittance.py` 期望 `prefix` / `nQ`。

```bash
python -c "import pandas as pd; p='merged_s4_shapes_myrun.csv'; df=pd.read_csv(p); df=df.rename(columns={'folder_key':'prefix','NQ':'nQ'}); df.to_csv(p,index=False)"
```

### 4) 将 CSV 预处理为 NPZ

```bash
mkdir -p merged_csvs
mv merged_s4_shapes_myrun.csv merged_csvs/

python three_stage_transmittance.py \
  --preprocess \
  --input_folder merged_csvs \
  --output_npz preprocessed_t_data.npz
```

### 5) 训练三阶段管线

```bash
python three_stage_transmittance.py \
  --data_npz preprocessed_t_data.npz \
  --num_epochs 100 \
  --batch_size 1024
```

预期输出目录模式：
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageA`
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageB`
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageC`

### 6) 评估阶段 A/B/C 模型

```bash
python three_stage_transmittance_evaluation.py \
  --model_dir outputs_three_stage_YYYYMMDD_HHMMSS \
  --data_npz preprocessed_t_data.npz \
  --sample_count 8
```

### 7) 可选：神经网络与 S4 一致性检查

```bash
python FilterShapeS4_Evaluator_Transmittance.py \
  --npz_file preprocessed_t_data.npz \
  --spec2shape_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageC/spec2shape_stageC.pt \
  --shape2spec_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageA/shape2spec_stageA.pt \
  --n_samples 4
```

## ⚙️ 关键 CLI 选项

### S4 启动脚本（`ms_final.sh`, `ms_resume_allargs.sh`）

| 参数 | 含义 | 默认值 |
|---|---|---|
| `-ns`, `--numshapes` | 生成形状数量 | `100000` |
| `-r`, `--seed` | 随机种子 | `88888` |
| `-p`, `--prefix` | 前缀/续跑键 | `""` |
| `-g`, `--numg` | 基函数/几何参数 | `80` |
| `-bo`, `--baseouter` | 基础外边界偏移 | `0.25` |
| `-ro`, `--randouter` | 随机外边界偏移 | `0.20` |

### 训练（`three_stage_transmittance.py`）

| 参数 | 用途 | 默认值 |
|---|---|---|
| `--preprocess` | 运行预处理模式 | `False` |
| `--input_folder` | 合并 CSV 所在目录 | `""` |
| `--output_npz` | 输出 NPZ 路径 | `preprocessed_data.npz` |
| `--data_npz` | 训练使用的 NPZ | `""` |
| `--csv_file` | 不使用 NPZ 时的 CSV 备选输入 | `""` |
| `--test` | 测试模式（跳过训练） | `False` |
| `--num_epochs` | 训练轮数 | `10` |
| `--batch_size` | 批大小 | `4096` |

### 评估（`three_stage_transmittance_evaluation.py`）

| 参数 | 用途 | 默认值 |
|---|---|---|
| `--model_dir` | 包含 `stageA/B/C` 的根目录 | required |
| `--data_npz` | 评估输入 NPZ | `""` |
| `--csv_file` | 评估输入 CSV | `""` |
| `--output_dir` | 输出目录覆盖 | 在 `model_dir` 中自动生成 |
| `--sample_count` | 可视化样本数量 | `4` |
| `--seed` | 采样随机种子 | `23` |
| `--font_scale` | 图像字体缩放 | `1.0` |
| `--batch_size` | 评估批大小 | `32` |
| `--plot_only` | 仅绘图模式 | `False` |

### S4 一致性评估器（`FilterShapeS4_Evaluator_Transmittance.py`）

| 参数 | 用途 | 默认值 |
|---|---|---|
| `--npz_file` | 预处理后的 NPZ 文件 | `preprocessed_t_data.npz` |
| `--spec2shape_ckpt` | 阶段 C 检查点 | `outputs_three_stage_20250322_145925/stageC/spec2shape_stageC.pt` |
| `--shape2spec_ckpt` | 阶段 A 检查点 | `outputs_three_stage_20250322_145925/stageA/shape2spec_stageA.pt` |
| `--n_samples` | 检查样本数 | `4` |
| `--seed` | 随机种子 | `23` |
| `--max_workers` | 并行 S4 worker 数量 | `4` |
| `--out_folder` | 输出目录 | 自动时间戳 |

## 🧪 快速冒烟运行

```bash
./ms_final.sh -ns 1000 -r 42 -p smoke -g 40 -bo 0.25 -ro 0.20
python merge_s4_data_full.py --prefix smoke
python -c "import pandas as pd; p='merged_s4_shapes_smoke.csv'; d=pd.read_csv(p).rename(columns={'folder_key':'prefix','NQ':'nQ'}); d.to_csv(p,index=False)"
python three_stage_transmittance.py --preprocess --input_folder . --output_npz smoke.npz
python three_stage_transmittance.py --data_npz smoke.npz --num_epochs 5 --batch_size 128
```

## 🔬 研究背景

核心逆向设计任务是：根据跨不同结晶状态的透射率光谱，推断具有 C4 对称性的超表面几何结构。当前透射率流程假设：

- 每个样本有 11 个结晶状态（`c` 值，在预处理时会排序）
- 每个状态有 100 个波长 bin（`T@...` 列）
- 最多 4 个 Q1 顶点，编码为 `(presence, x, y)`

除标准透射率管线外，仓库中还包含探索性分支（例如 `AVIRIS*`、`noise_experiment_*` 和 `archived/`）。

## 📚 引用

如果你使用了本仓库，或在此工作基础上进行扩展，请引用：

```bibtex
@article{chen2025inverse,
  title={Inverse Design of Metasurface for Spectral Imaging},
  author={Chen, Rongzhou and Nie, Haitao and Zhu, Shuo and Zhao, Yaping and Wang, Chutian and Lam, Edmund Y},
  journal={arXiv preprint arXiv:2510.21924},
  year={2025}
}
```

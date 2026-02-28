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

这是一个以脚本为核心的研究型仓库（历史上也称为 `inverse_metasurface`），用于光谱成像中的超表面逆向设计。核心工作流耦合了：

- 基于物理的 RCWA 仿真（S4 + Lua）
- 数据汇总与形状附加
- 三阶段 PyTorch 学习（`shape -> spectrum`、`spectrum -> shape`、链式微调）
- 定量与定性评估，并可选进行神经网络与 S4 一致性检查

## ✨ 快速概览

| 项目 | 说明 |
|---|---|
| 🎯 主要任务 | 从目标透射谱反推具有 C4 对称性的超表面几何形状 |
| 🔬 仿真器 | 由 shell 启动脚本和 `.lua` 脚本调用 `../build/S4` |
| 🧠 学习流程 | 阶段 A `shape -> spectra`、阶段 B `spectra -> shape`、阶段 C `spectra -> shape -> spectra` |
| 📦 数据约定 | 合并 CSV（`T@...`、元数据、`vertices_str`）-> 压缩 NPZ（`uids`、`spectra`、`shapes`） |
| 🧪 评估 | MSE 指标、分阶段可视化、可选全新 S4 重仿真 |

## 🧭 端到端流程

1. 在 `results/` 中生成仿真输出，并在 `shapes/` 中生成多边形文件。
2. 合并 S4 CSV 文件并附加形状顶点。
3. 规范化合并后的列名，以兼容训练流程。
4. 将合并后的 CSV 预处理为 NPZ 张量。
5. 训练阶段 A/B/C 模型。
6. 评估检查点并可视化模型行为。
7. 可选：将预测形状的光谱与新的 S4 运行结果进行对比。

## 🧱 仓库结构

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

## 🛠️ 前置依赖

| 依赖项 | 说明 |
|---|---|
| Linux + Bash | 启动脚本面向 shell 执行 |
| Python 3.9 | 与 `iccp.yaml` 保持一致 |
| Conda | 推荐用于可复现环境 |
| S4 binary | 预期路径为 `../build/S4` |
| CUDA GPU（可选） | 可加速训练与评估 |

## 🚀 环境配置

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

替代方案（可控性较弱）：

```bash
pip install -r pip_requirements.txt
```

### 3) 验证脚本期望的仿真器路径

```bash
ls -l ../build/S4
```

### 4) （可选）赋予启动脚本可执行权限

```bash
chmod +x ms.sh ms_final.sh ms_resume.sh ms_resume_allargs.sh ms_resume_random_state.sh
```

## ▶️ 实用用法

### A) 生成 RCWA 仿真数据

简化启动：

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

面向续跑的启动方式：

```bash
./ms_resume_allargs.sh \
  -ns 10000 \
  -r 12345 \
  -p myrun \
  -g 80 \
  -bo 0.35 \
  -ro 0.30
```

说明：
- 启动脚本会并行运行 `NQ=1..4`。
- 脚本以 `-t 32` 调用 `../build/S4`。

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

### E) 训练阶段 A/B/C

```bash
python three_stage_transmittance.py \
  --data_npz preprocessed_t_data.npz \
  --num_epochs 100 \
  --batch_size 1024
```

输出目录：
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageA`
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageB`
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageC`

### F) 评估已训练模型

```bash
python three_stage_transmittance_evaluation.py \
  --model_dir outputs_three_stage_YYYYMMDD_HHMMSS \
  --data_npz preprocessed_t_data.npz \
  --sample_count 8
```

### G) 可选：神经网络与 S4 一致性检查

```bash
python FilterShapeS4_Evaluator_Transmittance.py \
  --npz_file preprocessed_t_data.npz \
  --spec2shape_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageC/spec2shape_stageC.pt \
  --shape2spec_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageA/shape2spec_stageA.pt \
  --n_samples 4
```

## ⚙️ 关键 CLI 选项

### S4 启动脚本（`ms_final.sh`, `ms_resume_allargs.sh`）

| Flag | 含义 | 默认值 |
|---|---|---|
| `-ns`, `--numshapes` | 生成形状数量 | `100000` |
| `-r`, `--seed` | 随机种子 | `88888` |
| `-p`, `--prefix` | 前缀/续跑键 | `""` |
| `-g`, `--numg` | 基函数/网格参数 | `80` |
| `-bo`, `--baseouter` | 基础外边界偏移 | `0.25` |
| `-ro`, `--randouter` | 随机外边界偏移 | `0.20` |

### 训练（`three_stage_transmittance.py`）

| Flag | 含义 | 默认值 |
|---|---|---|
| `--preprocess` | 运行预处理模式 | `False` |
| `--input_folder` | 含合并 CSV 的目录 | `""` |
| `--output_npz` | 输出 NPZ 路径 | `preprocessed_data.npz` |
| `--data_npz` | 训练用 NPZ 数据集 | `""` |
| `--csv_file` | 未使用 NPZ 时的 CSV 回退输入 | `""` |
| `--test` | 测试模式 | `False` |
| `--num_epochs` | 训练轮数 | `10` |
| `--batch_size` | 批大小 | `4096` |

### 评估（`three_stage_transmittance_evaluation.py`）

| Flag | 含义 | 默认值 |
|---|---|---|
| `--model_dir` | 包含 `stageA/B/C` 的目录 | 必填 |
| `--data_npz` | NPZ 输入 | `""` |
| `--csv_file` | CSV 回退输入 | `""` |
| `--output_dir` | 输出目录覆盖 | 自动位于 `model_dir` 下 |
| `--sample_count` | 可视化样本数 | `4` |
| `--seed` | 随机种子 | `23` |
| `--font_scale` | 绘图字体缩放 | `1.0` |
| `--batch_size` | 评估批大小 | `32` |
| `--plot_only` | 仅绘制训练曲线 | `False` |

### S4 一致性评估器（`FilterShapeS4_Evaluator_Transmittance.py`）

| Flag | 含义 | 默认值 |
|---|---|---|
| `--npz_file` | 输入 NPZ 文件 | `preprocessed_t_data.npz` |
| `--spec2shape_ckpt` | 阶段 C 检查点路径 | `outputs_three_stage_20250322_145925/stageC/spec2shape_stageC.pt` |
| `--shape2spec_ckpt` | 阶段 A 检查点路径 | `outputs_three_stage_20250322_145925/stageA/shape2spec_stageA.pt` |
| `--n_samples` | 评估样本数 | `4` |
| `--seed` | 随机种子 | `23` |
| `--max_workers` | S4 工作线程数 | `4` |
| `--out_folder` | 输出目录 | 自动时间戳 |

## 🧪 快速冒烟测试

```bash
./ms_final.sh -ns 1000 -r 42 -p smoke -g 40 -bo 0.25 -ro 0.20
python merge_s4_data_full.py --prefix smoke
python -c "import pandas as pd; p='merged_s4_shapes_smoke.csv'; d=pd.read_csv(p).rename(columns={'folder_key':'prefix','NQ':'nQ'}); d.to_csv(p,index=False)"
mkdir -p merged_csvs && mv merged_s4_shapes_smoke.csv merged_csvs/
python three_stage_transmittance.py --preprocess --input_folder merged_csvs --output_npz smoke.npz
python three_stage_transmittance.py --data_npz smoke.npz --num_epochs 5 --batch_size 128
```

## 🔬 研究背景

当前逆向设计流程学习从不同结晶态下的透射率恢复 C4 对称几何。当前透射率管线默认以下设定：

- 每个形状样本有 11 行结晶态数据（按唯一 `shape_uid` 分组）
- 每个结晶态有 100 个波长采样点（`T@...` 列）
- 最多 4 个 Q1 控制点，编码为 `4x3` 张量：`(presence, x, y)`
- 在 C4 对称约束下重建多边形，用于形状可视化与一致性检查

仓库还包含主透射率训练路径之外的探索性分支（`AVIRIS*`、`noise_experiment*`、`archived/`）。

## 🧯 故障排查

| 现象 | 可能原因 | 修复建议 |
|---|---|---|
| `../build/S4: No such file or directory` | 预期相对路径下缺少 S4 可执行文件 | 在 `../build/S4` 构建或链接 S4，或更新启动脚本路径 |
| `No transmission columns found` | CSV 缺少 `T@...` 列 | 重新检查合并输出格式 |
| `Must specify either --data_npz or --csv_file` | 缺少训练/评估数据参数 | 显式提供其中一个输入 |
| `No valid shapes => SHIFT->Q1->UpTo4` | `vertices_str` 无效/为空，或 Q1 过滤后样本全被移除 | 校验形状文件与合并输出 |
| `Empty merge output for --prefix` | 前缀与 `results/` 中文件不匹配 | 检查精确文件名前缀并重新合并 |
| Evaluation checkpoint missing | 缺少 `stageA/B/C` 检查点文件 | 确认 `--model_dir` 指向完整输出目录 |

## 🤝 贡献

欢迎贡献，尤其是可复现性、测试和文档质量方面的改进。

建议流程：

1. 提交 issue，说明范围与期望行为。
2. 创建聚焦单一目标的分支。
3. 提交包含可运行命令与输出的 pull request。
4. 尽量将改动限定在单一工作流内。

## 📄 许可证

当前快照下，仓库根目录尚无 `LICENSE` 文件。请添加许可证以明确使用与再分发条款。

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

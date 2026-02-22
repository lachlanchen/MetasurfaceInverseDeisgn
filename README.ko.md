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

스펙트럼 이미징을 위한 **C4 대칭 메타표면 역설계** 연구용 스크립트 중심 저장소(과거 명칭: `inverse_metasurface`)로, 다음 범위를 다룹니다.

- S4 기반 RCWA 데이터 생성 (`.lua` + 셸 런처)
- 데이터 병합 및 전처리 (`.csv` -> `.npz`)
- 3단계 신경망 학습 (shape->spectra, spectra->shape, 체인 미세조정)
- 평가 및 선택적 neural-vs-S4 비교

## ✨ 한눈에 보기

| 항목 | 내용 |
|---|---|
| 핵심 목표 | 목표 투과 스펙트럼으로부터 기하 구조 예측 |
| 핵심 데이터 형태 | spectra: `11 x 100`, shape: `4 x 3` |
| 메인 학습 스크립트 | `three_stage_transmittance.py` |
| 메인 평가 스크립트 | `three_stage_transmittance_evaluation.py` |
| RCWA 런처 | `ms_final.sh`, `ms_resume_allargs.sh` |
| 병합 스크립트 | `merge_s4_data_full.py` |

## 🧠 연구 맥락

이 프로젝트는 스펙트럼 이미징용 메타표면의 역설계에 초점을 둡니다. 학습 파이프라인은 결정화 상태 전반에서 S4로 생성한 투과 스펙트럼을 사용하며, 순방향/역방향 매핑을 모두 학습합니다.

1. **Stage A**: shape -> spectra
2. **Stage B**: spectra -> shape
3. **Stage C**: spectra -> shape -> spectra (chain loss 미세조정)

현재 전처리/학습 코드는 다음 가정을 사용합니다.

- 결정화 상태 11개 (`c = 0.0 ... 1.0`)
- 상태별 파장 bin 100개
- shape 표현: 최대 4개의 Q1 포인트, 각 포인트는 `[presence, x, y]`

## 🗂️ 저장소 구조 (핵심 경로)

```text
.
├── ms.sh / ms_final.sh / ms_resume_allargs.sh
├── metasurface_final.lua / metasurface_allargs_resume.lua / metasurface_seed.lua
├── merge_s4_data_full.py
├── three_stage_transmittance.py
├── three_stage_transmittance_evaluation.py
├── FilterShapeS4_Evaluator_Transmittance.py
├── results/                # raw S4 output CSVs
├── shapes/                 # generated polygon vertices
├── merged_csvs/            # merged CSVs used for preprocessing
├── outputs_three_stage_*/  # checkpoints, losses, visualizations
├── partial_crys_data/
└── iccp.yaml
```

## ⚙️ 사전 요구사항

| 의존성 | 요구사항 |
|---|---|
| OS | Linux |
| Shell | Bash |
| Python | 3.9 |
| 환경 관리자 | Conda (권장) |
| RCWA 바이너리 | `../build/S4` (repo root 기준 상대 경로) |
| GPU | 선택 사항 (빠른 학습을 위해 권장) |

## 🚀 설치

```bash
git clone <repo-url> inverse_metasurface
cd inverse_metasurface

conda env create -f iccp.yaml
conda activate iccp

# Required by launcher scripts
ls -l ../build/S4
```

선택 사항:

```bash
chmod +x ms.sh ms_final.sh ms_resume_allargs.sh
```

## 🧪 전체 워크플로 사용법

### 1) S4로 RCWA 데이터 생성

```bash
./ms_final.sh \
  -ns 10000 \
  -r 12345 \
  -p myrun \
  -g 80 \
  -bo 0.35 \
  -ro 0.30
```

참고:

- 런처는 `../build/S4`를 `-t 32`로 호출하고 `NQ=1..4`를 병렬 실행합니다.
- `ms_final.sh`는 `metasurface_final.lua`를 사용합니다.
- `ms_resume_allargs.sh`는 `metasurface_allargs_resume.lua`를 사용합니다.

### 2) shape vertices와 RCWA 출력 병합

```bash
python merge_s4_data_full.py --prefix myrun
# -> merged_s4_shapes_myrun.csv
```

### 3) 학습용 병합 컬럼명 정규화 (필요 시)

`merge_s4_data_full.py`는 `folder_key` / `NQ`를 기록하지만, 학습 파이프라인은 `prefix` / `nQ`를 기대합니다.

```bash
python -c "import pandas as pd; p='merged_s4_shapes_myrun.csv'; df=pd.read_csv(p); df=df.rename(columns={'folder_key':'prefix','NQ':'nQ'}); df.to_csv(p,index=False)"
```

### 4) CSV -> NPZ 전처리

```bash
mkdir -p merged_csvs
mv merged_s4_shapes_myrun.csv merged_csvs/

python three_stage_transmittance.py \
  --preprocess \
  --input_folder merged_csvs \
  --output_npz preprocessed_t_data.npz
```

### 5) 3단계 모델 학습

```bash
python three_stage_transmittance.py \
  --data_npz preprocessed_t_data.npz \
  --num_epochs 100 \
  --batch_size 1024
```

주요 출력 구조:

- `outputs_three_stage_YYYYMMDD_HHMMSS/stageA`
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageB`
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageC`

### 6) 체크포인트 평가

```bash
python three_stage_transmittance_evaluation.py \
  --model_dir outputs_three_stage_YYYYMMDD_HHMMSS \
  --data_npz preprocessed_t_data.npz \
  --sample_count 8
```

### 7) 선택 사항: 신경망 예측과 S4 비교

```bash
python FilterShapeS4_Evaluator_Transmittance.py \
  --npz_file preprocessed_t_data.npz \
  --spec2shape_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageC/spec2shape_stageC.pt \
  --shape2spec_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageA/shape2spec_stageA.pt \
  --n_samples 4
```

## 🎛️ CLI 레퍼런스

### S4 런처 플래그 (`ms_final.sh`, `ms_resume_allargs.sh`)

| Flag | 의미 | 기본값 |
|---|---|---|
| `-ns`, `--numshapes` | shape 개수 | `100000` |
| `-r`, `--seed` | 랜덤 시드 | `88888` |
| `-p`, `--prefix` | 실행 prefix / 재시작 키 | `""` |
| `-g`, `--numg` | 기하 basis 파라미터 | `80` |
| `-bo`, `--baseouter` | 기본 외곽 오프셋 | `0.25` |
| `-ro`, `--randouter` | 랜덤 외곽 오프셋 | `0.20` |

### 학습 플래그 (`three_stage_transmittance.py`)

| Flag | 용도 |
|---|---|
| `--preprocess` | 전처리 모드 실행 |
| `--input_folder` | 병합 CSV 파일 폴더 |
| `--output_npz` | 전처리 결과 NPZ 파일명 |
| `--data_npz` | 학습용 NPZ 데이터셋 |
| `--csv_file` | CSV 데이터셋 대체 입력 |
| `--test` | 테스트 모드 |
| `--num_epochs` | 학습 epoch 수 |
| `--batch_size` | 배치 크기 |

### 평가 플래그 (`three_stage_transmittance_evaluation.py`)

| Flag | 용도 |
|---|---|
| `--model_dir` | 체크포인트 루트 디렉터리 (필수) |
| `--data_npz` / `--csv_file` | 평가 데이터 소스 |
| `--output_dir` | 평가 출력 폴더 |
| `--sample_count` | 시각화 샘플 수 |
| `--seed` | 샘플 선택용 랜덤 시드 |
| `--font_scale` | 플롯 폰트 스케일 |
| `--batch_size` | 평가 배치 크기 |
| `--plot_only` | 학습 곡선 플롯만 재생성 |

## 🧾 데이터 계약 (전처리 기준)

`three_stage_transmittance.py`의 전처리 경로는 다음 컬럼을 포함한 병합 CSV를 기대합니다.

- ID 컬럼: `prefix`, `nQ`, `nS`, `shape_idx`, `c`
- geometry 텍스트: `vertices_str`
- 스펙트럼 컬럼: `T@...`

코드에서 수행하는 품질 검사:

- `shape_uid = prefix_nQ_nS_shape_idx` 기준 그룹화
- 각 그룹은 정확히 11개 행을 포함해야 함
- Q1 포인트 수가 `[1, 4]` 범위인 shape만 유지

## 🛠️ 문제 해결

- `../build/S4: No such file or directory`
  - `../build/S4`에 S4를 빌드/링크하거나, 실제 S4 경로에 맞게 런처 스크립트를 수정하세요.
- `No matching CSVs found in 'results/'`
  - `results/*_output_nQ*_nS*.csv`에서 `--prefix`와 출력 파일명 규칙을 확인하세요.
- `No transmission columns found`
  - 병합 CSV에 `T@...` 컬럼이 포함되어 있는지 확인하세요.
- 전처리 결과가 0건일 때
  - 필수 컬럼 존재 여부와 각 shape UID가 11개 결정화 행을 갖는지 확인하세요.
- 학습 중 GPU OOM
  - `--batch_size`를 줄이세요(예: `256` 또는 `128`).
- 평가에서 체크포인트를 찾지 못할 때
  - `--model_dir` 아래에 다음 파일이 있는지 확인하세요:
    - `stageA/shape2spec_stageA.pt`
    - `stageB/spec2shape_stageB.pt`
    - `stageC/spec2shape_stageC.pt`

## 📚 인용

이 저장소가 연구에 도움이 되었다면 다음 논문을 인용해 주세요.

```bibtex
@article{chen2025inverse,
  title={Inverse Design of Metasurface for Spectral Imaging},
  author={Chen, Rongzhou and Nie, Haitao and Zhu, Shuo and Zhao, Yaping and Wang, Chutian and Lam, Edmund Y},
  journal={arXiv preprint arXiv:2510.21924},
  year={2025}
}
```

## 🌐 언어 버전

이 저장소에는 다음 README 변형도 포함되어 있습니다.

- `README.en.md`, `README.de.md`, `README.es.md`, `README.fr.md`
- `README.ru.md`, `README.ja.md`, `README.ko.md`, `README.vi.md`
- `README.ar.md`, `README.zh-CN.md`, `README.zh-TW.md`

## 📌 참고

- 이 저장소는 보관용/탐색용 스크립트를 다수 포함한 연구 워크스페이스입니다.
- 표준 transmittance 경로는 다음을 중심으로 구성됩니다.
  - `ms_final.sh` / `ms_resume_allargs.sh`
  - `merge_s4_data_full.py`
  - `three_stage_transmittance.py`
  - `three_stage_transmittance_evaluation.py`
- 현재 저장소 루트에는 명시적인 라이선스 파일이 없습니다.

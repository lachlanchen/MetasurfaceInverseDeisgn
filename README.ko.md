English | 中文繁體 | 中文简体 | 日本語 | 한국어 | Tiếng Việt | العربية | Français | Español | Deutsch | Русский

# inverse_metasurface

<div align="center">

[![Status](https://img.shields.io/badge/status-experimental-orange)](#-roadmap)
[![Python](https://img.shields.io/badge/python-3.9-blue)](#-prerequisites)
[![Platform](https://img.shields.io/badge/platform-linux-lightgrey)](#-prerequisites)
[![S4](https://img.shields.io/badge/S4-required-success)](#-prerequisites)
[![License](https://img.shields.io/badge/license-unset-lightgrey)](#-license)

</div>

**S4/RCWA 시뮬레이션**과 **다단계 신경망 모델**을 활용한 **역 메타표면 설계** 연구 워크스페이스입니다.

지원 기능:
- C4 대칭 다각형 메타표면을 위한 고처리량 S4 데이터 생성.
- 학습 가능한 NPZ 형식으로의 데이터 병합 + 전처리.
- 3단계 학습: **shape -> spectrum**, **spectrum -> shape**, **chain-tuned spectrum -> shape -> spectrum**.
- 평가 및 선택적 **neural-vs-S4** 직접 비교.

## 🌟 한눈에 보기

| 영역 | 이 저장소에서 제공하는 것 |
|---|---|
| 물리 시뮬레이션 | Bash + Lua 스크립트로 `nQ=1..4` 전 구간을 병렬 S4 실행 |
| 데이터셋 툴링 | 다각형 정점과 피벗 스펙트럼을 붙여 병합하는 스크립트 |
| ML 파이프라인 | 전처리 + 학습 모드를 포함한 `three_stage_transmittance.py` |
| 평가 | 지표 + 그림 생성을 포함한 `three_stage_transmittance_evaluation.py` |
| 연구 브랜치 | AVIRIS/초분광 및 노이즈/압축 실험 |

## 🧠 연구 맥락

이 저장소는 광학 메타표면의 역설계를 목표로 합니다. 즉, 원하는 스펙트럼으로부터 구조를 추론하고, 반대로도 추론합니다.

기본 파이프라인의 핵심 가정:
- Q1 포인트 파라미터화를 통한 C4 대칭.
- 11개 결정화 상태 (`c` in `[0.0, 1.0]`).
- 각 샘플 스펙트럼은 `11 x 100` 전송률 행으로 저장.
- 형상은 최대 4개 Q1 포인트로 표현 (`4 x 3` 텐서: presence, x, y).

3단계 학습 로직:
1. **Stage A**: shape -> spectrum (`shape2spec_stageA.pt`)
2. **Stage B**: spectrum -> shape (`spec2shape_stageB.pt`)
3. **Stage C**: spectrum -> shape -> spectrum 체인 미세조정 (`spec2shape_stageC.pt`)

## 🗂 프로젝트 구조

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
├─ partial_crys_data/                # optical constants by crystallization level
├─ results/                          # raw S4 outputs (usually untracked)
├─ shapes/                           # generated polygon vertex files
├─ merged_csvs/                      # merged tables used for preprocessing
├─ outputs_three_stage_*/            # model artifacts per run
├─ AVIRIS*/ + aviris_*.py            # hyperspectral branch
├─ how_to_run.md / commands*.md
├─ iccp.yaml
└─ pip_requirements.txt
```

## ✅ 사전 요구사항

| Requirement | Notes |
|---|---|
| OS | Linux (스크립트는 Bash + Linux 경로를 가정) |
| Python | 3.9 (`iccp.yaml` 기준) |
| Env manager | Conda 권장 |
| S4 binary | 저장소 루트 기준 `../build/S4` 경로를 기대 |
| GPU (optional) | CUDA 사용 시 학습/평가 가속 |

## ⚙️ 설치

### 1) Clone and enter

```bash
git clone <your-repo-url> inverse_metasurface
cd inverse_metasurface
```

### 2) Create environment

```bash
conda env create -f iccp.yaml
conda activate iccp
```

### 3) Verify S4 path

```bash
ls -l ../build/S4
```

해당 경로가 없다면, S4를 그 위치에 빌드/배치하거나 스크립트 경로를 조정하세요.

## 🚀 빠른 시작 (End-to-End)

```bash
# 1) Generate S4 data
./ms.sh -ns 10000 -r 12345

# 2) Merge one run by prefix
python merge.py --prefix 20250123_155420

# 3) Move merged CSV to preprocessing folder
mkdir -p merged_csvs
mv merged_s4_shapes_20250123_155420.csv merged_csvs/

# 4) Preprocess to NPZ
python three_stage_transmittance.py --preprocess \
  --input_folder merged_csvs \
  --output_npz preprocessed_t_data.npz

# 5) Train three-stage model
python three_stage_transmittance.py \
  --data_npz preprocessed_t_data.npz \
  --num_epochs 100 \
  --batch_size 1024

# 6) Evaluate model outputs
python three_stage_transmittance_evaluation.py \
  --model_dir outputs_three_stage_YYYYMMDD_HHMMSS \
  --data_npz preprocessed_t_data.npz \
  --sample_count 8
```

## 🧪 사용 상세

### A) S4 generation

Minimal seeded run:

```bash
./ms.sh -ns 10000 -r 88888
```

Parameterized run:

```bash
./ms_final.sh \
  -ns 100000 \
  -r 88888 \
  -p iccpOv100kG80 \
  -g 80 \
  -bo 0.35 \
  -ro 0.30
```

Resume-style run:

```bash
./ms_resume_allargs.sh \
  -ns 100000 \
  -r 88888 \
  -p iccpOv100kG80 \
  -g 80 \
  -bo 0.35 \
  -ro 0.30
```

### B) Merge raw S4 CSVs

```bash
python merge.py --prefix 20250123_155420
```

Alternative utility:

```bash
python merge_s4_data_full.py --prefix 20250123_155420
```

### C) Preprocess merged CSVs -> NPZ

```bash
python three_stage_transmittance.py --preprocess \
  --input_folder merged_csvs \
  --output_npz preprocessed_t_data.npz
```

### D) Train

```bash
python three_stage_transmittance.py \
  --data_npz preprocessed_t_data.npz \
  --num_epochs 100 \
  --batch_size 1024
```

학습 결과물은 다음 경로에 저장됩니다:
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageA`
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageB`
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageC`

### E) Evaluate

```bash
python three_stage_transmittance_evaluation.py \
  --model_dir outputs_three_stage_YYYYMMDD_HHMMSS \
  --data_npz preprocessed_t_data.npz \
  --sample_count 8
```

생성 항목:
- `evaluation_metrics.csv`
- `metrics_summary.csv`
- stage 시각화 (`.png`, `.pdf`)
- training curve plots

### F) Neural vs direct S4 comparison (optional)

```bash
python FilterShapeS4_Evaluator_Transmittance.py \
  --npz_file preprocessed_t_data.npz \
  --spec2shape_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageC/spec2shape_stageC.pt \
  --shape2spec_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageA/shape2spec_stageA.pt \
  --n_samples 4
```

## 🔧 핵심 CLI 옵션

### S4 runner flags (`ms_final.sh`, `ms_resume_allargs.sh`)

| Flag | Meaning | Default |
|---|---|---|
| `-ns`, `--numshapes` | 형상 개수 | `100000` |
| `-r`, `--seed` | 랜덤 시드 | `88888` |
| `-p`, `--prefix` | 실행 prefix / 재개 키 | empty |
| `-g`, `--numg` | Geometry/basis 파라미터 | `80` |
| `-bo`, `--baseouter` | 기본 outer 오프셋 | `0.25` |
| `-ro`, `--randouter` | outer 랜덤 오프셋 | `0.20` |

### Training flags (`three_stage_transmittance.py`)

| Flag | Meaning | Default |
|---|---|---|
| `--preprocess` | 전처리 모드 실행 | off |
| `--input_folder` | 입력 CSV 폴더 | empty |
| `--output_npz` | 출력 NPZ 경로 | `preprocessed_data.npz` |
| `--data_npz` | 학습/평가에 사용할 NPZ | empty |
| `--csv_file` | 직접 사용할 CSV | empty |
| `--num_epochs` | 단계별 epoch 수 | `10` |
| `--batch_size` | 배치 크기 | `4096` |
| `--test` | 테스트 모드 placeholder | off |

### Evaluation flags (`three_stage_transmittance_evaluation.py`)

| Flag | Meaning | Default |
|---|---|---|
| `--model_dir` | 학습 실행 디렉터리 루트 | required |
| `--data_npz` | 평가용 NPZ | empty |
| `--csv_file` | 평가용 CSV | empty |
| `--output_dir` | 출력 폴더 | `model_dir/evaluation_<timestamp>` |
| `--sample_count` | 시각화 샘플 개수 | `4` |
| `--seed` | 샘플링 시드 | `23` |
| `--font_scale` | 플롯 글꼴 스케일 | `1.0` |
| `--batch_size` | 평가 배치 크기 | `32` |
| `--plot_only` | 곡선 플롯만 생성 | off |

## 🧭 문제 해결

| Symptom | Likely cause | Fix |
|---|---|---|
| `../build/S4: No such file or directory` | S4 바이너리 경로 불일치 | `../build/S4`에 S4를 빌드/배치하거나 스크립트를 수정 |
| `Must specify either --data_npz or --csv_file` | 데이터셋 소스 누락 | 두 플래그 중 하나를 명시적으로 전달 |
| `No matching CSVs found in 'results/'` | prefix 불일치 | prefix와 출력 파일명을 재확인 |
| Very few/zero preprocessed records | `T@...` 또는 `vertices_str` 행 누락/오류 | 병합 CSV 스키마와 shape별 행 그룹핑 검증 |
| CUDA OOM | 배치가 너무 큼 | `--batch_size` 축소 (예: `1024 -> 256`) |

## 🧱 개발 노트

- 이 저장소는 실험 중심이며, 생성 산출물 다수는 의도적으로 추적되지 않습니다.
- 유사 작업을 위한 스크립트 변형이 여러 개 존재합니다 (`merge_*`, `aviris_*`, `noise_experiment_*`).
- 기본 inverse-metasurface 워크플로는 이 README에 문서화된 경로입니다.
- 학습 스크립트에서 시드(`42`)를 설정하지만, 엄밀한 결정론 보장은 하드웨어/백엔드 동작에 좌우됩니다.
- 현재 통합 CI + 완전한 자동 테스트 스위트는 없습니다.

## 🛣 로드맵

- 빠른 스모크 테스트용 소형 canonical 데이터셋 추가.
- 중복 파이프라인 스크립트 통합.
- merge/preprocess 무결성과 체크포인트 로드 가능성 검증 추가.
- lint + 스모크 학습/평가 CI 추가.
- S4 빌드/버전 고정 방식 문서화 강화.

## 🤝 기여

1. 기능 브랜치를 생성하세요.
2. PR 범위는 작게 유지하세요 (PR당 하나의 파이프라인/실험 이슈).
3. 정확히 재실행 가능한 명령과 예상 출력 경로를 포함하세요.
4. 필요한 경우가 아니면 대용량 생성 산출물 커밋을 피하세요.
5. 재현성 정보를 추가하세요 (시드, 데이터 소스, 체크포인트 경로).

## 📄 라이선스

현재 이 저장소에는 `LICENSE` 파일이 없습니다.

라이선스 파일이 추가되기 전까지, 재사용 및 재배포 조건은 **미정**으로 간주해야 합니다.

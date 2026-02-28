[English](../README.md) · [العربية](README.ar.md) · [Español](README.es.md) · [Français](README.fr.md) · [日本語](README.ja.md) · [한국어](README.ko.md) · [Tiếng Việt](README.vi.md) · [中文 (简体)](README.zh-Hans.md) · [中文（繁體）](README.zh-Hant.md) · [Deutsch](README.de.md) · [Русский](README.ru.md)


# 스펙트럴 이미징을 위한 메타표면 역설계

<p align="center">
  <img alt="Status" src="https://img.shields.io/badge/Status-Research%20Prototype-f59e0b?style=for-the-badge">
  <img alt="Python" src="https://img.shields.io/badge/Python-3.9-3776AB?style=for-the-badge&logo=python&logoColor=white">
  <img alt="PyTorch" src="https://img.shields.io/badge/PyTorch-2.x-EE4C2C?style=for-the-badge&logo=pytorch&logoColor=white">
  <img alt="Simulator" src="https://img.shields.io/badge/Simulator-S4%20RCWA-16a34a?style=for-the-badge">
  <img alt="Platform" src="https://img.shields.io/badge/Platform-Linux%20%2B%20Bash-6b7280?style=for-the-badge&logo=gnu-bash&logoColor=white">
  <img alt="ArXiv" src="https://img.shields.io/badge/arXiv-2510.21924-b31b1b?style=for-the-badge&logo=arxiv&logoColor=white">
</p>

스펙트럴 이미징에서 메타표면 역설계를 수행하기 위한 스크립트 중심 연구 저장소입니다(과거 명칭: `inverse_metasurface`).

핵심 워크플로는 다음을 결합합니다.
- 물리 기반 RCWA 시뮬레이션 (`S4` + Lua)
- 데이터 병합 및 형상(shape) 정보 결합
- 3단계 PyTorch 학습 (`shape -> spectrum`, `spectrum -> shape`, 체인 파인튜닝)
- 정량/정성 평가 및 선택적 neural-vs-S4 일관성 검증

> [!IMPORTANT]
> 프로젝트의 기존 스크립트/문서에 정의된 표준 동작과 명령을 유지합니다. 과거 문서가 현재 없는 파일을 참조하는 경우에도, 호환성을 위해 해당 참조를 명시적 주석과 함께 의도적으로 보존합니다.

## 📑 목차

- [✨ 한눈에 보기](#-한눈에-보기)
- [🌍 국제화 (i18n)](#-국제화-i18n)
- [✨ 기능](#-기능)
- [🧭 엔드투엔드 워크플로](#-엔드투엔드-워크플로)
- [🧱 프로젝트 구조](#-프로젝트-구조)
- [🛠️ 사전 요구사항](#️-사전-요구사항)
- [🚀 설치](#-설치)
- [▶️ 사용법](#️-사용법)
- [⚙️ 설정](#️-설정)
- [🧪 예제](#-예제)
- [🔬 연구 맥락](#-연구-맥락)
- [🧑‍💻 개발 노트](#-개발-노트)
- [🧯 문제 해결](#-문제-해결)
- [🗺️ 로드맵](#️-로드맵)
- [🤝 기여](#-기여)
- [📄 라이선스](#-라이선스)
- [📚 인용](#-인용)

## ✨ 한눈에 보기

| 항목 | 내용 |
|---|---|
| 🎯 주요 과제 | 목표 투과율 스펙트럼으로부터 C4 대칭 메타표면 기하 형상 추론 |
| 🔬 시뮬레이터 | 셸 런처와 `.lua` 스크립트에서 호출하는 `../build/S4` |
| 🧠 학습 파이프라인 | Stage A `shape -> spectra`, Stage B `spectra -> shape`, Stage C `spectra -> shape -> spectra` |
| 📦 데이터 규약 | 병합 CSV (`T@...`, 메타데이터, `vertices_str`) -> 압축 NPZ (`uids`, `spectra`, `shapes`) |
| 🧪 평가 | MSE 지표, 단계별 시각화, 선택적 신규 S4 재시뮬레이션 |
| 🌐 i18n 상태 | 루트 레벨 다국어 README 파일 + 기존 `i18n/` 디렉터리 |

## 🌍 국제화 (i18n)

- 다국어 README는 저장소 루트의 `README.<lang>.md` 파일로 관리됩니다.
- 이 저장소 스냅샷에는 `i18n/` 디렉터리가 존재합니다.
- 이 파일은 상단에 단일 언어 옵션 라인만 유지하여 언어 바 중복을 방지합니다.
- `README.en.md`도 저장소에 존재하지만, 이번 업데이트 기준 표준 베이스는 `README.md`입니다.

## ✨ 기능

- S4 시뮬레이션 출력부터 학습된 역모델까지 이어지는 엔드투엔드 역설계 경로.
- C4 대칭 다각형 파라미터화 및 Q1 포인트 인코딩 (`4x3`: `presence, x, y`).
- 단일 스크립트(`three_stage_transmittance.py`)에서 수행되는 3단계 모델 학습.
- 스펙트럼 정밀도를 유지하면서 형상별 정점 정보를 결합하는 병합 도구.
- 학습 예측 결과를 새로운 S4 시뮬레이션과 비교하는 선택적 평가기.
- 폭넓은 탐색 브랜치(AVIRIS, SWIR/noise, GSST, archived/deprecated 변형).

## 🧭 엔드투엔드 워크플로

1. `results/`에 시뮬레이션 출력, `shapes/`에 다각형 파일 생성
2. S4 CSV 파일 병합 및 형상 정점 정보 결합
3. 학습 호환성을 위해 병합 컬럼명 정규화
4. 병합 CSV를 NPZ 텐서로 전처리
5. Stage A/B/C 모델 학습
6. 체크포인트 평가 및 동작 시각화
7. 선택적으로 예측 형상 스펙트럼과 신규 S4 실행 결과 비교

## 🧱 프로젝트 구조

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

## 🛠️ 사전 요구사항

| 의존성 | 비고 |
|---|---|
| Linux + Bash | 런처 스크립트는 셸 실행을 대상으로 함 |
| Python 3.9 | `iccp.yaml` (`python=3.9.18`)과 일치 |
| Conda | 재현성을 위해 권장 |
| S4 binary | `../build/S4` 경로에 있어야 함 |
| CUDA GPU (선택) | 학습/평가 속도 향상 |

## 🚀 설치

### 1) 클론 및 진입

```bash
git clone <your-repo-url> inverse_metasurface
cd inverse_metasurface
```

### 2) 환경 생성 (권장)

```bash
conda env create -f iccp.yaml
conda activate iccp
```

대체 참고:

```bash
# Historical README reference (file may be absent in this snapshot)
pip install -r pip_requirements.txt
```

### 3) 스크립트가 기대하는 시뮬레이터 경로 확인

```bash
ls -l ../build/S4
```

### 4) (선택) 런처 실행 권한 부여

```bash
chmod +x ms.sh ms_final.sh ms_resume.sh ms_resume_allargs.sh ms_resume_random_state.sh ms_resume_random_state_nir.sh
```

## ▶️ 사용법

### A) RCWA 시뮬레이션 데이터 생성

간단 실행:

```bash
./ms.sh -ns 10000 -r 12345
```

파라미터 실행:

```bash
./ms_final.sh \
  -ns 10000 \
  -r 12345 \
  -p myrun \
  -g 80 \
  -bo 0.35 \
  -ro 0.30
```

재개 중심 실행:

```bash
./ms_resume_allargs.sh \
  -ns 10000 \
  -r 12345 \
  -p myrun \
  -g 80 \
  -bo 0.35 \
  -ro 0.30
```

추가 재개/랜덤 상태 예시(명령 문서 기준):

```bash
./ms_resume_random_state.sh \
  -p iccp100kG20Ov \
  -r 88888 \
  -g 20 \
  -bo 0.35 \
  -ro 0.3 \
  -ns 100000
```

참고:
- 런처는 `NQ=1..4`를 병렬 실행합니다.
- 스크립트는 `-t 32` 옵션으로 `../build/S4`를 호출합니다.

### B) S4 출력 병합 및 형상 정점 정보 결합

```bash
python merge_s4_data_full.py --prefix myrun
# output: merged_s4_shapes_myrun.csv
```

### C) 학습 호환성을 위한 컬럼 정규화

`merge_s4_data_full.py`는 `folder_key`와 `NQ`를 기록하지만, 학습 경로는 `prefix`와 `nQ`를 기대합니다.

```bash
python -c "import pandas as pd; p='merged_s4_shapes_myrun.csv'; df=pd.read_csv(p); df=df.rename(columns={'folder_key':'prefix','NQ':'nQ'}); df.to_csv(p,index=False)"
```

### D) CSV -> NPZ 전처리

```bash
mkdir -p merged_csvs
mv merged_s4_shapes_myrun.csv merged_csvs/

python three_stage_transmittance.py \
  --preprocess \
  --input_folder merged_csvs \
  --output_npz preprocessed_t_data.npz
```

### E) Stage A/B/C 학습

```bash
python three_stage_transmittance.py \
  --data_npz preprocessed_t_data.npz \
  --num_epochs 100 \
  --batch_size 1024
```

출력 경로:
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageA`
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageB`
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageC`

### F) 학습 모델 평가

과거 README 명령(기존 문서 호환을 위해 스크립트 이름 유지):

```bash
python three_stage_transmittance_evaluation.py \
  --model_dir outputs_three_stage_YYYYMMDD_HHMMSS \
  --data_npz preprocessed_t_data.npz \
  --sample_count 8
```

저장소 상태 참고: 이 스냅샷에는 `three_stage_transmittance_evaluation.py`가 없습니다. 현재 사용 가능한 평가 기능은 `FilterShapeS4_Evaluator_Transmittance.py`를 사용하세요.

### G) 선택적 neural-vs-S4 일관성 점검

```bash
python FilterShapeS4_Evaluator_Transmittance.py \
  --npz_file preprocessed_t_data.npz \
  --spec2shape_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageC/spec2shape_stageC.pt \
  --shape2spec_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageA/shape2spec_stageA.pt \
  --n_samples 4
```

## ⚙️ 설정

### S4 런처 (`ms_final.sh`, `ms_resume_allargs.sh`)

| Flag | 의미 | 기본값 |
|---|---|---|
| `-ns`, `--numshapes` | 생성할 형상 수 | `100000` |
| `-r`, `--seed` | 랜덤 시드 | `88888` |
| `-p`, `--prefix` | prefix/resume 키 | `""` |
| `-g`, `--numg` | basis/grid 파라미터 | `80` |
| `-bo`, `--baseouter` | 기본 외곽 경계 오프셋 | `0.25` |
| `-ro`, `--randouter` | 랜덤 외곽 경계 오프셋 | `0.20` |

### 학습 (`three_stage_transmittance.py`)

| Flag | 의미 | 기본값 |
|---|---|---|
| `--preprocess` | 전처리 모드 실행 | `False` |
| `--input_folder` | 병합 CSV 폴더 경로 | `""` |
| `--output_npz` | 출력 NPZ 경로 | `preprocessed_data.npz` |
| `--data_npz` | 학습용 NPZ 데이터셋 | `""` |
| `--csv_file` | NPZ 미사용 시 CSV 대체 입력 | `""` |
| `--test` | 테스트 모드 | `False` |
| `--num_epochs` | 학습 epoch 수 | `10` |
| `--batch_size` | 배치 크기 | `4096` |

### 과거 평가 설정 (`three_stage_transmittance_evaluation.py`)

| Flag | 의미 | 기본값 |
|---|---|---|
| `--model_dir` | `stageA/B/C`가 있는 디렉터리 | 필수 |
| `--data_npz` | NPZ 입력 | `""` |
| `--csv_file` | CSV 대체 입력 | `""` |
| `--output_dir` | 출력 디렉터리 덮어쓰기 | `model_dir` 하위 자동 생성 |
| `--sample_count` | 시각화 샘플 수 | `4` |
| `--seed` | 랜덤 시드 | `23` |
| `--font_scale` | 플롯 폰트 배율 | `1.0` |
| `--batch_size` | 평가 배치 크기 | `32` |
| `--plot_only` | 학습 곡선만 플로팅 | `False` |

### S4 일관성 평가기 (`FilterShapeS4_Evaluator_Transmittance.py`)

| Flag | 의미 | 기본값 |
|---|---|---|
| `--npz_file` | 입력 NPZ 파일 | `preprocessed_t_data.npz` |
| `--spec2shape_ckpt` | Stage C 체크포인트 경로 | `outputs_three_stage_20250322_145925/stageC/spec2shape_stageC.pt` |
| `--shape2spec_ckpt` | Stage A 체크포인트 경로 | `outputs_three_stage_20250322_145925/stageA/shape2spec_stageA.pt` |
| `--n_samples` | 평가 샘플 수 | `4` |
| `--seed` | 랜덤 시드 | `23` |
| `--max_workers` | S4 워커 스레드 수 | `4` |
| `--out_folder` | 출력 디렉터리 | 타임스탬프 기반 자동 생성 |

## 🧪 예제

### 스모크 런

```bash
./ms_final.sh -ns 1000 -r 42 -p smoke -g 40 -bo 0.25 -ro 0.20
python merge_s4_data_full.py --prefix smoke
python -c "import pandas as pd; p='merged_s4_shapes_smoke.csv'; d=pd.read_csv(p).rename(columns={'folder_key':'prefix','NQ':'nQ'}); d.to_csv(p,index=False)"
mkdir -p merged_csvs && mv merged_s4_shapes_smoke.csv merged_csvs/
python three_stage_transmittance.py --preprocess --input_folder merged_csvs --output_npz smoke.npz
python three_stage_transmittance.py --data_npz smoke.npz --num_epochs 5 --batch_size 128
```

### 실행 프로필 예시 (`commands.md` / `commands_updated.md` 기준)

```bash
# no overlap, G=40
./ms_resume_allargs.sh -ns 10000 -r 12345 -p fast_without_overlap -g 40 -bo 0.25 -ro 0.2

# overlap, G=80
./ms_resume_allargs.sh -ns 10000 -r 12345 -p overlap -g 80 -bo 0.35 -ro 0.3

# large run, overlap, G=80
./ms_resume_allargs.sh -ns 100000 -r 12345 -p more_basis_overlap -g 80 -bo 0.35 -ro 0.3
```

## 🔬 연구 맥락

현재 역설계 설정은 결정화 상태 전반의 투과율로부터 C4 대칭 기하 형상을 복원하도록 학습됩니다. 현재 투과율 파이프라인은 다음을 가정합니다.

- 형상 샘플당 11개 결정화 행(고유 `shape_uid` 기준 그룹화)
- 결정화 상태당 100개 파장 bin (`T@...` 컬럼)
- 최대 4개의 Q1 제어점을 `4x3` 텐서 `(presence, x, y)`로 인코딩
- 형상 시각화 및 일관성 점검을 위한 C4 대칭 기반 다각형 재구성

저장소에는 주요 투과율 학습 경로 외에도 `AVIRIS*`, `noise_experiment*`, `archived/` 등 탐색 브랜치가 포함되어 있습니다.

## 🧑‍💻 개발 노트

- 이 저장소는 패키지형 Python 모듈보다 스크립트 중심 연구 저장소에 가깝습니다.
- 핵심 스크립트는 상대 경로(특히 `../build/S4`, `results/`, `shapes/`)를 가정합니다.
- `.gitignore`는 다수의 생성 실험 산출물(`*.csv`, `*.npz`, `*.pt`, 실행 폴더)을 제외합니다.
- 과거 문서에서 언급되지만 현재 스냅샷에 없는 파일/디렉터리 참조는 호환성을 위해 주석과 함께 의도적으로 유지됩니다.
- macOS 사이드카 파일(`._*`)이 포함되어 있을 수 있으며, 기능과 무관한 메타데이터 아티팩트일 수 있습니다.

## 🧯 문제 해결

| 증상 | 가능한 원인 | 해결 방법 |
|---|---|---|
| `../build/S4: No such file or directory` | 기대 상대 경로에 S4 binary가 없음 | `../build/S4`에 S4를 빌드/링크하거나 런처 경로 수정 |
| `No transmission columns found` | CSV에 `T@...` 컬럼이 없음 | 병합 출력 형식을 다시 확인 |
| `Must specify either --data_npz or --csv_file` | 학습/평가 데이터 인자 누락 | 입력 하나를 명시적으로 제공 |
| `No valid shapes => SHIFT->Q1->UpTo4` | `vertices_str`가 비어 있거나 유효하지 않음, 또는 Q1 필터링으로 전 샘플 제거 | shape 파일과 병합 출력 검증 |
| `--prefix` 기준 병합 출력이 비어 있음 | prefix가 `results/` 파일과 일치하지 않음 | 정확한 파일명 prefix 확인 후 재실행 |
| 평가 체크포인트 누락 | `stageA/B/C` 체크포인트 파일이 없음 | `--model_dir`가 완전한 출력 폴더를 가리키는지 확인 |
| `three_stage_transmittance_evaluation.py` not found | 과거 문서에 언급되지만 현재 없음 | `FilterShapeS4_Evaluator_Transmittance.py` 사용 또는 이전 커밋에서 해당 스크립트 복원 |

## 🗺️ 로드맵

- 명시적 데이터 버전 관리 매니페스트와 고정 실행 설정을 통해 재현성 향상
- 투과율, AVIRIS, noise 브랜치의 표준 진입점 정리
- 전처리 및 소규모 1 epoch 학습에 대한 자동 스모크 테스트 추가
- 출력 폴더와 정확한 명령줄을 연결하는 실험 레지스트리 개선
- 다국어 README 동기화 워크플로(루트 언어 파일 + `i18n/`) 확장

## 🤝 기여

재현성, 테스트, 문서 품질 개선에 대한 기여를 환영합니다.

권장 절차:

1. 범위와 기대 동작을 담은 이슈를 먼저 등록합니다.
2. 목적이 명확한 브랜치를 생성합니다.
3. 실행 가능한 명령과 결과를 포함해 Pull Request를 제출합니다.
4. 가능하면 하나의 워크플로 범위로 변경사항을 제한합니다.

## 📄 라이선스

현재 스냅샷의 저장소 루트에는 `LICENSE` 파일이 없습니다. 사용 및 재배포 조건을 정의하려면 라이선스 파일을 추가하세요.

## 📚 인용

이 저장소를 사용하거나 본 연구를 바탕으로 작업하는 경우, 다음을 인용해 주세요.

```bibtex
@article{chen2025inverse,
  title={Inverse Design of Metasurface for Spectral Imaging},
  author={Chen, Rongzhou and Nie, Haitao and Zhu, Shuo and Zhao, Yaping and Wang, Chutian and Lam, Edmund Y},
  journal={arXiv preprint arXiv:2510.21924},
  year={2025}
}
```

[English](../README.md) · [العربية](README.ar.md) · [Español](README.es.md) · [Français](README.fr.md) · [日本語](README.ja.md) · [한국어](README.ko.md) · [Tiếng Việt](README.vi.md) · [中文 (简体)](README.zh-Hans.md) · [中文（繁體）](README.zh-Hant.md) · [Deutsch](README.de.md) · [Русский](README.ru.md)


[![LazyingArt banner](https://github.com/lachlanchen/lachlanchen/raw/main/figs/banner.png)](https://github.com/lachlanchen/lachlanchen/blob/main/figs/banner.png)

# 분광 이미징을 위한 메타표면 역설계

<p align="center">
  <img alt="Status" src="https://img.shields.io/badge/Status-Research%20Prototype-f59e0b?style=for-the-badge">
  <img alt="Python" src="https://img.shields.io/badge/Python-3.9-3776AB?style=for-the-badge&logo=python&logoColor=white">
  <img alt="PyTorch" src="https://img.shields.io/badge/PyTorch-2.x-EE4C2C?style=for-the-badge&logo=pytorch&logoColor=white">
  <img alt="Simulator" src="https://img.shields.io/badge/Simulator-S4%20RCWA-16a34a?style=for-the-badge">
  <img alt="Platform" src="https://img.shields.io/badge/Platform-Linux%20%2B%20Bash-6b7280?style=for-the-badge&logo=gnu-bash&logoColor=white">
  <img alt="ArXiv" src="https://img.shields.io/badge/arXiv-2510.21924-b31b1b?style=for-the-badge&logo=arxiv&logoColor=white">
</p>

분광 이미징용 메타표면 역설계를 위한 `inverse_metasurface`(과거 명칭) 스크립트 기반 연구 저장소입니다.

핵심 워크플로우는 다음으로 구성됩니다.
- 물리 기반 RCWA 시뮬레이션 (`S4` + Lua)
- 데이터 통합 및 도형 첨부
- 3단계 PyTorch 학습 (`shape -> spectrum`, `spectrum -> shape`, 연결된 파인 튜닝)
- 정량·정성 평가, 필요 시 신경망-대-S4 일관성 검증

> [!IMPORTANT]
> 기존 프로젝트 스크립트/문서에 보존된 동작과 명령을 유지합니다. 역사적 참조가 현재 스냅샷에서 누락된 파일을 가리키는 경우, 호환성 유지를 위해 해당 참조와 주석을 그대로 둡니다.

## 📑 목차

- [🌟 Snapshot](#-snapshot)
- [✨ 한눈에 보기](#-at-a-glance)
- [🌍 국제화 (i18n)](#-internationalization-i18n)
- [✨ 기능](#-features)
- [🧭 엔드투엔드 워크플로우](#-end-to-end-workflow)
- [🧱 프로젝트 구조](#-project-structure)
- [🛠️ 선행 조건](#️-prerequisites)
- [🚀 설치](#-installation)
- [▶️ 사용법](#️-usage)
- [⚙️ 설정](#️-configuration)
- [🧪 예제](#-examples)
- [🔬 연구 배경](#-research-context)
- [🧑‍💻 개발 노트](#-development-notes)
- [🧯 문제 해결](#-troubleshooting)
- [🗺️ 로드맵](#️-roadmap)
- [🤝 기여](#-contribution)
- [📄 라이선스](#-license)
- [📚 인용](#-citation)

## 🌟 Snapshot

| 항목 | 상태 |
|---|---|
| 🧠 목표 | 전이 스펙트럼 데이터로부터 C4 대칭 메타표면 기하구조를 역으로 재구성 |
| 🔧 핵심 스택 | S4 RCWA (`Lua`) + PyTorch 학습 + 선택적 기하 -> 스펙트럼 재검증 |
| 🧪 데이터 파이프라인 | CSV 병합/도형 정점 첨부 → NPZ (`uids`, `spectra`, `shapes`) |
| 🚀 준비 상태 | 연구 프로토타입; 스크립트와 문서는 역사적 참조와의 호환성 유지 |

## ✨ 한눈에 보기

| 항목 | 세부 내용 |
|---|---|
| 🎯 주요 작업 | 목표 전송율 스펙트럼으로부터 C4 대칭 메타표면 기하구조 추론 |
| 🔬 시뮬레이터 | 쉘 런처와 `.lua` 스크립트로 호출되는 `../build/S4` |
| 🧠 학습 파이프라인 | Stage A `shape -> spectra`, Stage B `spectra -> shape`, Stage C `spectra -> shape -> spectra` |
| 📦 데이터 규약 | 병합 CSV (`T@...`, 메타데이터, `vertices_str`) → 압축 NPZ (`uids`, `spectra`, `shapes`) |
| 🧪 평가 | MSE 지표, 단계별 시각화, 선택적 S4 재시뮬레이션 |
| 🌐 i18n 상태 | 루트 수준 다국어 README 파일 + 기존 `i18n/` 디렉터리 |

## 🌍 국제화 (i18n)

- 다국어 README는 저장소 루트에 `README.<lang>.md` 형태로 관리됩니다.
- 이 저장소 스냅샷에 `i18n/` 디렉터리가 존재합니다.
- 이 파일은 언어 링크가 중복되지 않도록 최상단에 단일 언어 바를 배치합니다.
- `README.en.md`도 존재합니다. 이번 갱신에서는 `README.md`를 기준 원문으로 사용합니다.

## ✨ 기능

- S4 시뮬레이션 결과부터 역모델 학습까지 연결되는 엔드투엔드 역설계 경로.
- C4 대칭 다각형 매개변수화 및 Q1 포인트 인코딩 (`4x3`: `presence, x, y`).
- 하나의 스크립트에서 3단계 모델 학습 수행 (`three_stage_transmittance.py`).
- 스펙트럼 정밀도를 보존하고 샘플별 정점(`vertices`)을 붙이는 병합 도구.
- 학습 예측값을 새로운 S4 시뮬레이션과 비교하는 선택적 평가기.
- AVIRIS, SWIR/노이즈, GSST, archived/deprecated 분기를 포함한 폭넓은 탐색 브랜치.

## 🧭 엔드투엔드 워크플로우

1. `results/`에 시뮬레이션 출력 파일을 생성하고 `shapes/`에 폴리곤 파일을 생성합니다.
2. S4 CSV 파일을 병합하고 도형 정점을 첨부합니다.
3. 학습 호환을 위해 열 이름을 정규화합니다.
4. 병합된 CSV 파일을 NPZ 텐서로 전처리합니다.
5. Stage A/B/C 모델을 학습합니다.
6. 체크포인트를 평가하고 동작을 시각화합니다.
7. 필요 시 예측한 도형 스펙트럼을 새로 실행한 S4 결과와 비교합니다.

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

## 🛠️ 선행 조건

| 의존성 | 비고 |
|---|---|
| Linux + Bash | 런처 스크립트는 쉘 실행을 전제로 함 |
| Python 3.9 | `iccp.yaml`(`python=3.9.18`)과 일치 |
| Conda | 재현성 확보를 위해 권장 |
| S4 바이너리 | `../build/S4` 경로에 존재해야 함 |
| CUDA GPU (선택) | 학습/평가 속도 개선 |

## 🚀 설치

### 1) 클론 후 진입

```bash

git clone <your-repo-url> inverse_metasurface
cd inverse_metasurface
```

### 2) 환경 생성 (권장)

```bash
conda env create -f iccp.yaml
conda activate iccp
```

대체 노트:

```bash
# Historical README reference (file may be absent in this snapshot)
pip install -r pip_requirements.txt
```

### 3) 스크립트에서 기대하는 시뮬레이터 경로 확인

```bash
ls -l ../build/S4
```

### 4) (선택) 런처 실행 권한 부여

```bash
chmod +x ms.sh ms_final.sh ms_resume.sh ms_resume_allargs.sh ms_resume_random_state.sh ms_resume_random_state_nir.sh
```

## ▶️ 사용법

### A) RCWA 시뮬레이션 데이터 생성

단순 런처:

```bash
./ms.sh -ns 10000 -r 12345
```

매개변수 지정 런처:

```bash
./ms_final.sh \
  -ns 10000 \
  -r 12345 \
  -p myrun \
  -g 80 \
  -bo 0.35 \
  -ro 0.30
```

재개 전용 런처:

```bash
./ms_resume_allargs.sh \
  -ns 10000 \
  -r 12345 \
  -p myrun \
  -g 80 \
  -bo 0.35 \
  -ro 0.30
```

추가 resume/랜덤 상태 예시 (명령문서에서 발췌):

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

### B) S4 출력 병합 및 도형 정점 첨부

```bash
python merge_s4_data_full.py --prefix myrun
# output: merged_s4_shapes_myrun.csv
```

### C) 학습 호환을 위한 열 이름 정규화

`merge_s4_data_full.py`는 `folder_key`와 `NQ`를 출력하지만, 학습 경로는 `prefix`와 `nQ`를 기대합니다.

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

출력은 다음에 기록됩니다:
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageA`
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageB`
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageC`

### F) 학습된 모델 평가

이력 README 명령 (호환성을 위해 스크립트 이름은 그대로 유지):

```bash
python three_stage_transmittance_evaluation.py \
  --model_dir outputs_three_stage_YYYYMMDD_HHMMSS \
  --data_npz preprocessed_t_data.npz \
  --sample_count 8
```

저장소 상태 메모:
`three_stage_transmittance_evaluation.py`는 현재 스냅샷에 존재하지 않습니다. 사용 가능한 평가 기능은 `FilterShapeS4_Evaluator_Transmittance.py`를 사용하세요.

### G) 선택적 신경망-vs-S4 일관성 검증

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
| `-ns`, `--numshapes` | 생성할 도형 수 | `100000` |
| `-r`, `--seed` | 난수 시드 | `88888` |
| `-p`, `--prefix` | 접두사/재개 키 | `""` |
| `-g`, `--numg` | 기저/그리드 파라미터 | `80` |
| `-bo`, `--baseouter` | 기본 외곽 경계 오프셋 | `0.25` |
| `-ro`, `--randouter` | 무작위 외곽 경계 오프셋 | `0.20` |

### 학습 (`three_stage_transmittance.py`)

| Flag | 의미 | 기본값 |
|---|---|---|
| `--preprocess` | 전처리 모드 실행 | `False` |
| `--input_folder` | 병합 CSV가 포함된 폴더 | `""` |
| `--output_npz` | 출력 NPZ 경로 | `preprocessed_data.npz` |
| `--data_npz` | 학습용 NPZ 데이터셋 | `""` |
| `--csv_file` | NPZ 미사용 시 대체 CSV | `""` |
| `--test` | 테스트 모드 | `False` |
| `--num_epochs` | 에포크 수 | `10` |
| `--batch_size` | 배치 크기 | `4096` |

### 과거 평가 설정 (`three_stage_transmittance_evaluation.py`)

| Flag | 의미 | 기본값 |
|---|---|---|
| `--model_dir` | `stageA/B/C`를 포함한 디렉터리 | required |
| `--data_npz` | NPZ 입력 | `""` |
| `--csv_file` | CSV 입력 대체 | `""` |
| `--output_dir` | 출력 디렉터리 재설정 | `model_dir` 하위 자동 |
| `--sample_count` | 시각화 샘플 수 | `4` |
| `--seed` | 난수 시드 | `23` |
| `--font_scale` | 플롯 글꼴 배율 | `1.0` |
| `--batch_size` | 평가 배치 크기 | `32` |
| `--plot_only` | 학습 곡선만 그리기 | `False` |

### S4 일관성 평가기 (`FilterShapeS4_Evaluator_Transmittance.py`)

| Flag | 의미 | 기본값 |
|---|---|---|
| `--npz_file` | 입력 NPZ 파일 | `preprocessed_t_data.npz` |
| `--spec2shape_ckpt` | Stage C 체크포인트 경로 | `outputs_three_stage_20250322_145925/stageC/spec2shape_stageC.pt` |
| `--shape2spec_ckpt` | Stage A 체크포인트 경로 | `outputs_three_stage_20250322_145925/stageA/shape2spec_stageA.pt` |
| `--n_samples` | 평가 샘플 수 | `4` |
| `--seed` | 난수 시드 | `23` |
| `--max_workers` | S4 작업자 스레드 | `4` |
| `--out_folder` | 출력 디렉터리 | 타임스탬프 자동 |

## 🧪 예제

### 스모크 실행

```bash
./ms_final.sh -ns 1000 -r 42 -p smoke -g 40 -bo 0.25 -ro 0.20
python merge_s4_data_full.py --prefix smoke
python -c "import pandas as pd; p='merged_s4_shapes_smoke.csv'; d=pd.read_csv(p).rename(columns={'folder_key':'prefix','NQ':'nQ'}); d.to_csv(p,index=False)"
mkdir -p merged_csvs && mv merged_s4_shapes_smoke.csv merged_csvs/
python three_stage_transmittance.py --preprocess --input_folder merged_csvs --output_npz smoke.npz
python three_stage_transmittance.py --data_npz smoke.npz --num_epochs 5 --batch_size 128
```

### 실행 프로필 예시 (`commands.md` / `commands_updated.md`에서)

```bash
# no overlap, G=40
./ms_resume_allargs.sh -ns 10000 -r 12345 -p fast_without_overlap -g 40 -bo 0.25 -ro 0.2

# overlap, G=80
./ms_resume_allargs.sh -ns 10000 -r 12345 -p overlap -g 80 -bo 0.35 -ro 0.3

# large run, overlap, G=80
./ms_resume_allargs.sh -ns 100000 -r 12345 -p more_basis_overlap -g 80 -bo 0.35 -ro 0.3
```

## 🔬 연구 컨텍스트

현재의 역설계 설정은 결정화 상태별 전송율을 통해 C4 대칭 기하를 복원하는 학습을 수행합니다. 현재 전송율 파이프라인은 다음을 가정합니다:

- 형상 샘플당 11개의 결정화 행 (`shape_uid` 기준으로 그룹화)
- 각 결정화 상태당 100개 파장 구간의 스펙트럼 열 (`T@...` 컬럼)
- 최대 4개 Q1 제어점을 `4x3` 텐서로 인코딩: (`presence, x, y`)
- 도형 시각화 및 일관성 검사를 위한 C4 대칭 다각형 재구성

저장소에는 주요 전송율 학습 경로 외에도 탐색 브랜치(`AVIRIS*`, `noise_experiment*`, `archived/`)가 존재합니다.

## 🧑‍💻 개발 노트

- 본 저장소는 패키지형 Python 모듈이 아니라 스크립트 중심의 연구 저장소입니다.
- 핵심 스크립트는 상대 경로(특히 `../build/S4`, `results/`, `shapes/`)를 가정합니다.
- `.gitignore`는 많은 생성된 실험 산출물(`*.csv`, `*.npz`, `*.pt`, 실행 폴더)을 제외합니다.
- 역사 문서에 언급된 일부 파일/디렉터리는 현재 스냅샷에 없습니다. 호환성을 위해 이 참고 항목은 주석과 함께 유지됩니다.
- macOS 사이드카 파일(`._*`)이 존재할 수 있으며, 작동하지 않는 메타데이터 산물일 수 있습니다.

## 🧯 문제 해결

| 증상 | 가능성 있는 원인 | 해결 |
|---|---|---|
| `../build/S4: No such file or directory` | 스크립트에서 기대하는 상대 경로에 S4 바이너리가 없음 | `../build/S4`에서 S4를 빌드/링크하거나 런처 경로를 수정 |
| `No transmission columns found` | CSV에 `T@...` 열이 없음 | 병합 출력 형식 재확인 |
| `Must specify either --data_npz or --csv_file` | 학습/평가 데이터 인자가 누락됨 | 데이터 인자를 하나 명시적으로 지정 |
| `No valid shapes => SHIFT->Q1->UpTo4` | `vertices_str`이 유효하지 않거나 비어있고 Q1 필터로 모든 샘플이 제거됨 | 도형 파일 및 병합 결과를 검증 |
| `Empty merge output for --prefix` | `--prefix`와 일치하는 `results/` 파일이 없음 | 접두어를 정확히 확인하고 병합 재실행 |
| Evaluation checkpoint missing | `stageA/B/C` 체크포인트 누락 | `--model_dir`이 완전한 출력 폴더를 가리키는지 확인 |
| `three_stage_transmittance_evaluation.py` not found | 과거 문서에서 참조되나 현재 존재하지 않음 | `FilterShapeS4_Evaluator_Transmittance.py`를 사용하거나 이전 커밋에서 복원 |

## 🗺️ 로드맵

- 명시적 데이터 버전 매니페스트와 고정된 실행 구성을 통해 재현성을 향상.
- transmittance, AVIRIS, noise 브랜치의 정식 진입점을 정비.
- 전처리와 1 에포크 미니 학습용 자동 스모크 테스트 추가.
- 출력 폴더와 정확한 실행 명령을 연결하는 실험 레지스트리 가시성 강화.
- 다국어 README 동기화 파이프라인 확장(루트 README 및 `i18n/`).

## 🤝 기여

재현성, 테스트, 문서 품질 개선에 대한 기여를 환영합니다.

권장 절차:

1. 이슈에서 범위와 기대 동작 공유
2. 목적이 뚜렷한 브랜치 생성
3. 실행 가능한 명령과 출력 결과를 포함한 PR 제출
4. 가능한 경우 작업 흐름을 하나로 제한

## 📄 라이선스

현재 스냅샷의 저장소 루트에는 `LICENSE` 파일이 없습니다. 사용 조건과 재배포 규정을 정의하려면 추가하세요.

## 📚 인용

이 저장소를 사용하거나 이 작업을 기반으로 확장하는 경우, 다음을 인용해 주세요.

```bibtex
@article{chen2025inverse,
  title={Inverse Design of Metasurface for Spectral Imaging},
  author={Chen, Rongzhou and Nie, Haitao and Zhu, Shuo and Zhao, Yaping and Wang, Chutian and Lam, Edmund Y},
  journal={arXiv preprint arXiv:2510.21924},
  year={2025}
}
```


## ❤️ Support

| Donate | PayPal | Stripe |
| --- | --- | --- |
| [![Donate](https://camo.githubusercontent.com/24a4914f0b42c6f435f9e101621f1e52535b02c225764b2f6cc99416926004b7/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f446f6e6174652d4c617a79696e674172742d3045413545393f7374796c653d666f722d7468652d6261646765266c6f676f3d6b6f2d6669266c6f676f436f6c6f723d7768697465)](https://chat.lazying.art/donate) | [![PayPal](https://camo.githubusercontent.com/d0f57e8b016517a4b06961b24d0ca87d62fdba16e18bbdb6aba28e978dc0ea21/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f50617950616c2d526f6e677a686f754368656e2d3030343537433f7374796c653d666f722d7468652d6261646765266c6f676f3d70617970616c266c6f676f436f6c6f723d7768697465)](https://paypal.me/RongzhouChen) | [![Stripe](https://camo.githubusercontent.com/1152dfe04b6943afe3a8d2953676749603fb9f95e24088c92c97a01a897b4942/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f5374726970652d446f6e6174652d3633354246463f7374796c653d666f722d7468652d6261646765266c6f676f3d737472697065266c6f676f436f6c6f723d7768697465)](https://buy.stripe.com/aFadR8gIaflgfQV6T4fw400) |

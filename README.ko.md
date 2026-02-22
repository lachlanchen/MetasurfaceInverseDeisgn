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

[![Status](https://img.shields.io/badge/Status-Research%20Prototype-orange)](#-project-scope)
[![Python](https://img.shields.io/badge/Python-3.9-3776AB?logo=python&logoColor=white)](#-environment-setup)
[![Platform](https://img.shields.io/badge/Platform-Linux-2f2f2f?logo=linux&logoColor=white)](#-prerequisites)
[![RCWA](https://img.shields.io/badge/RCWA-S4%20Required-1f9d55)](#-prerequisites)
[![PyTorch](https://img.shields.io/badge/Framework-PyTorch-EE4C2C?logo=pytorch&logoColor=white)](#-training-and-evaluation)
[![License](https://img.shields.io/badge/License-Not%20Specified-lightgrey)](#-license)

C4 대칭 조건에서 **메타표면 역설계**를 수행하기 위한 스크립트 중심 연구 코드베이스입니다.
다음 요소를 결합합니다.

- 🔬 S4/RCWA 시뮬레이션 (`.lua` + Bash 실행 스크립트)
- 🧱 원시 스펙트럼 + 형상 버텍스 데이터 병합 및 전처리
- 🧠 3단계 PyTorch 학습 (`shape -> spectra`, `spectra -> shape`, 체인 미세조정)
- 📊 평가 및 시각화(신경망 예측과 S4 일치성 비교 포함)

## 📌 목차

- [프로젝트 범위](#-project-scope)
- [연구 맥락](#-research-context)
- [저장소 구조](#-repository-layout)
- [사전 요구사항](#-prerequisites)
- [환경 설정](#-environment-setup)
- [빠른 시작](#-quick-start)
- [엔드투엔드 파이프라인](#-end-to-end-pipeline)
- [학습 및 평가](#-training-and-evaluation)
- [주요 CLI 옵션](#-key-cli-options)
- [문제 해결](#-troubleshooting)
- [로드맵](#-roadmap)
- [인용](#-citation)
- [라이선스](#-license)

## 🎯 프로젝트 범위

이 저장소는 실험 비중이 높고, 패키지형 라이브러리보다는 실행 가능한 스크립트 중심으로 구성되어 있습니다.
가장 안정적인 워크플로는 다음과 같습니다.

1. S4를 실행해 `results/`에 원시 광학 출력, `shapes/`에 형상 데이터를 생성
2. 실행별 CSV를 병합/피벗하여 학습용 테이블 데이터 생성
3. 병합된 CSV를 압축 `.npz`로 변환
4. 3단계 투과율 파이프라인 학습
5. 체크포인트 평가 후 플롯/지표 내보내기

## 🧪 연구 맥락

### 문제 설정

핵심 역설계 과제는 C4 대칭 제약과 부분 결정화 스윕 조건에서, 목표 투과 스펙트럼으로부터 메타표면 형상을 복원하고(및 그 역방향) 학습하는 것입니다.

### 투과율 파이프라인의 데이터 가정

| 항목 | 값 |
|---|---|
| 형상당 결정화 상태 수 | 11 (`c` from `0.0` to `1.0`) |
| 상태당 스펙트럼 빈 수 | 100 |
| 샘플당 스펙트럼 텐서 | `11 x 100` |
| 샘플당 형상 텐서 | `4 x 3` (`[presence, x, y]`) |

### 3단계 학습 목표

| 단계 | 방향 | 일반적인 체크포인트 |
|---|---|---|
| A | `shape -> spectrum` | `stageA/shape2spec_stageA.pt` |
| B | `spectrum -> shape` | `stageB/spec2shape_stageB.pt` |
| C | `spectrum -> shape -> spectrum` (체인 미세조정) | `stageC/spec2shape_stageC.pt` |

## 🗂️ 저장소 구조

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

## 🧩 사전 요구사항

| 의존성 | 설명 |
|---|---|
| Linux + Bash | 셸 스크립트가 Linux 스타일 경로를 가정함 |
| Conda | 권장 환경 관리 도구 (`iccp.yaml`) |
| Python 3.9 | 학습/평가 스크립트의 주 실행 환경 |
| S4 binary | 저장소 루트 기준 `../build/S4` 경로를 기대 |
| CUDA (optional) | 학습 및 평가 속도 향상 |

## ⚙️ 환경 설정

```bash
git clone <repo-url> inverse_metasurface
cd inverse_metasurface

conda env create -f iccp.yaml
conda activate iccp

# verify S4 path expected by shell runners
ls -l ../build/S4
```

셸 실행 스크립트에 실행 권한이 필요하면:

```bash
chmod +x ms.sh ms_final.sh ms_resume_allargs.sh ms_resume.sh
```

## 🚀 빠른 시작

이미 `preprocessed_t_data.npz`가 있다면:

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

## 🔁 엔드투엔드 파이프라인

### 1) S4 데이터 생성

```bash
./ms_final.sh \
  -ns 10000 \
  -r 12345 \
  -p myrun \
  -g 80 \
  -bo 0.35 \
  -ro 0.30
```

### 2) S4 출력 + 형상 버텍스 병합

```bash
python merge_s4_data_full.py --prefix myrun
# -> merged_s4_shapes_myrun.csv
```

### 3) 전처리를 위한 CSV 컬럼명 정규화

`three_stage_transmittance.py` 전처리는 `prefix`와 `nQ`를 기대하지만, 병합 결과에는 `folder_key`와 `NQ`가 들어있을 수 있습니다.

```bash
python -c "import pandas as pd; p='merged_s4_shapes_myrun.csv'; df=pd.read_csv(p); df=df.rename(columns={'folder_key':'prefix','NQ':'nQ'}); df.to_csv(p,index=False)"
```

### 4) CSV -> NPZ 전처리

```bash
mkdir -p merged_csvs
mv merged_s4_shapes_myrun.csv merged_csvs/

python three_stage_transmittance.py --preprocess \
  --input_folder merged_csvs \
  --output_npz preprocessed_t_data.npz
```

### 5) 3단계 학습

```bash
python three_stage_transmittance.py \
  --data_npz preprocessed_t_data.npz \
  --num_epochs 100 \
  --batch_size 1024
```

### 6) 평가 및 플로팅

```bash
python three_stage_transmittance_evaluation.py \
  --model_dir outputs_three_stage_YYYYMMDD_HHMMSS \
  --data_npz preprocessed_t_data.npz \
  --sample_count 8
```

### 7) 선택 사항: 신경망 vs S4 비교

```bash
python FilterShapeS4_Evaluator_Transmittance.py \
  --npz_file preprocessed_t_data.npz \
  --spec2shape_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageC/spec2shape_stageC.pt \
  --shape2spec_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageA/shape2spec_stageA.pt \
  --n_samples 4
```

## 🧠 학습 및 평가

### 주요 스크립트

| 스크립트 | 목적 |
|---|---|
| `three_stage_transmittance.py` | 전처리 + A/B/C 단계 학습 |
| `three_stage_transmittance_evaluation.py` | 체크포인트 평가, 지표 계산, 플롯 저장 |
| `FilterShapeS4_Evaluator_Transmittance.py` | 학습된 예측과 S4 동작 비교 |

### 일반적인 출력물

| 산출물 | 위치 |
|---|---|
| 단계별 체크포인트 | `outputs_three_stage_*/stageA|stageB|stageC/` |
| 평가 그림 | `outputs_three_stage_*/evaluation_<timestamp>/` |
| 지표 CSV | `evaluation_metrics.csv`, `metrics_summary.csv` |

## 🛠️ 주요 CLI 옵션

### S4 실행 스크립트 (`ms_final.sh`, `ms_resume_allargs.sh`)

| Flag | 의미 | 기본값 |
|---|---|---|
| `-ns`, `--numshapes` | 형상 개수 | `100000` |
| `-r`, `--seed` | 랜덤 시드 | `88888` |
| `-p`, `--prefix` | 실행 prefix/resume key | empty |
| `-g`, `--numg` | 기하학 기준 설정 | `80` |
| `-bo`, `--baseouter` | 기본 외곽 오프셋 | `0.25` |
| `-ro`, `--randouter` | 랜덤 외곽 오프셋 | `0.20` |

### 학습 (`three_stage_transmittance.py`)

| Flag | 의미 | 기본값 |
|---|---|---|
| `--preprocess` | 전처리 모드로 전환 | off |
| `--input_folder` | 병합 CSV가 있는 폴더 | `""` |
| `--output_npz` | 전처리 출력 파일 | `preprocessed_data.npz` |
| `--data_npz` | 학습용 NPZ 입력 | `""` |
| `--csv_file` | 직접 CSV 학습 입력 | `""` |
| `--test` | 테스트 모드 토글 | off |
| `--num_epochs` | 단계당 에폭 수 | `10` |
| `--batch_size` | 배치 크기 | `4096` |

### 평가 (`three_stage_transmittance_evaluation.py`)

| Flag | 의미 | 기본값 |
|---|---|---|
| `--model_dir` | 학습 실행 디렉터리 (필수) | - |
| `--data_npz` | 평가용 NPZ 입력 | `""` |
| `--csv_file` | 평가용 CSV 입력 | `""` |
| `--output_dir` | 사용자 지정 출력 디렉터리 | auto |
| `--sample_count` | 시각화 샘플 수 | `4` |
| `--seed` | 샘플 선택용 랜덤 시드 | `23` |
| `--font_scale` | 플롯 폰트 배율 | `1.0` |
| `--batch_size` | eval dataloader 배치 크기 | `32` |
| `--plot_only` | 플롯만 재생성 | off |

## 🧯 문제 해결

| 증상 | 가능성 높은 원인 | 해결 방법 |
|---|---|---|
| `../build/S4: No such file or directory` | S4 바이너리가 기대한 상대 경로에 없음 | `../build/S4`에 S4를 배치/빌드하거나 스크립트 경로 수정 |
| `Must specify either --data_npz or --csv_file` | 학습/평가 데이터셋 인자가 누락됨 | 데이터 입력 인자를 정확히 하나만 지정 |
| `No transmission columns found` | 병합 CSV에 `T@...` 컬럼이 없음 | 병합/피벗을 다시 실행하고 헤더 확인 |
| `KeyError: 'prefix'` in preprocess | 병합 출력이 여전히 `folder_key`/`NQ`를 사용함 | 전처리 전에 컬럼을 `prefix`/`nQ`로 변경 |
| GPU OOM | 배치가 너무 큼 | `--batch_size` 축소 |
| Missing checkpoints during eval | 단계 체크포인트가 없거나 경로가 잘못됨 | 선택한 `--model_dir` 아래 stageA/B/C 파일 확인 |

## 🧭 로드맵

- 병합 스크립트 전반의 CSV 스키마 통일 (`prefix`, `nQ` 명명)
- 병합/전처리/체크포인트 로딩 자동 테스트 추가
- 전체 파이프라인 오케스트레이션용 단일 CLI 진입점 제공
- 재현성을 위한 데이터셋/실행 매니페스트 추가
- 명시적 오픈소스 라이선스 추가

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

## 📄 라이선스

현재 이 저장소에는 `LICENSE` 파일이 없습니다. 따라서 라이선스가 추가되기 전까지 사용 및 재배포 권한은 명시되어 있지 않습니다.

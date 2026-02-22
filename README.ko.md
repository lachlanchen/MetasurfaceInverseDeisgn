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

스펙트럴 이미징을 위한 메타표면 역설계를 다루는 스크립트 중심 연구 저장소입니다(기존 명칭: `inverse_metasurface`). 이 파이프라인은 **S4 기반 RCWA 시뮬레이션**과 **3단계 PyTorch 워크플로우**를 결합하여, 구조와 광학 투과 스펙트럼 사이의 정방향/역방향 매핑을 학습합니다.

## ✨ 한눈에 보기

| 항목 | 설명 |
|---|---|
| 🎯 목표 | 목표 투과 스펙트럼으로부터 C4 대칭 메타표면 구조를 예측 |
| 🔬 물리 모델 | S4 기반 RCWA 시뮬레이션 (`../build/S4`) |
| 🧠 학습 파이프라인 | Stage A `shape -> spectra`, Stage B `spectra -> shape`, Stage C `spectra -> shape -> spectra` |
| 📦 데이터 형식 | 통합 CSV (`T@...`, shape 메타데이터) -> 압축 NPZ (`uids`, `spectra`, `shapes`) |
| 🧪 평가 | MSE 지표, 정성 시각화, 선택적 S4 재시뮬레이션 검증 |

## 🧭 전체 워크플로우

1. S4(`.lua` + 셸 런처)로 메타표면 광응답 데이터를 생성합니다.
2. 원시 시뮬레이션 CSV를 병합하고 폴리곤 정점 정보를 붙입니다.
3. 병합된 CSV를 학습용 NPZ로 변환합니다.
4. 3단계 투과율 파이프라인을 학습합니다.
5. 체크포인트를 평가하고 Stage A/B/C 동작을 시각화합니다.
6. 필요 시 신경망 예측과 새로운 S4 시뮬레이션 결과를 비교합니다.

## 🧱 저장소 구조

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
├── results/                # S4 원시 CSV 출력
├── shapes/                 # 병합 시 사용하는 폴리곤 정점 파일
├── merged_csvs/            # 병합된 CSV 데이터셋
├── outputs_three_stage_*/  # 학습 체크포인트 및 곡선
├── partial_crys_data/      # 결정화 상태 광학 테이블
│
├── AVIRIS*/                # 보조 하이퍼스펙트럴 실험
├── noise_experiment_*/     # 강건성/노이즈 실험 브랜치
└── archived/               # 과거 스크립트 및 스냅샷
```

## 🛠️ 사전 준비

- Linux + Bash
- Conda (권장)
- Python 3.9
- `../build/S4` 경로에 S4 바이너리
- 선택 사항: 학습 가속용 CUDA GPU

## 🚀 설정

```bash
git clone <your-repo-url> inverse_metasurface
cd inverse_metasurface

conda env create -f iccp.yaml
conda activate iccp

# 스크립트가 기대하는 시뮬레이터 경로 확인
ls -l ../build/S4
```

선택 사항:

```bash
chmod +x ms.sh ms_final.sh ms_resume_allargs.sh
```

## ▶️ 실사용 가이드

### 1) RCWA 데이터 생성

`ms_final.sh`와 `ms_resume_allargs.sh`는 각각 S4 작업 4개를 병렬 실행합니다 (`NQ=1..4`):

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
- `ms.sh`는 더 단순한 실행 경로입니다.
- 런처 스크립트는 `../build/S4`를 가정하며 `-t 32`를 사용합니다.

### 2) S4 출력 + shape 정점 병합

```bash
python merge_s4_data_full.py --prefix myrun
# output: merged_s4_shapes_myrun.csv
```

### 3) 학습 호환을 위한 컬럼명 정규화

`merge_s4_data_full.py`는 `folder_key` / `NQ`를 출력하지만, `three_stage_transmittance.py`는 `prefix` / `nQ`를 기대합니다.

```bash
python -c "import pandas as pd; p='merged_s4_shapes_myrun.csv'; df=pd.read_csv(p); df=df.rename(columns={'folder_key':'prefix','NQ':'nQ'}); df.to_csv(p,index=False)"
```

### 4) CSV를 NPZ로 전처리

```bash
mkdir -p merged_csvs
mv merged_s4_shapes_myrun.csv merged_csvs/

python three_stage_transmittance.py \
  --preprocess \
  --input_folder merged_csvs \
  --output_npz preprocessed_t_data.npz
```

### 5) 3단계 파이프라인 학습

```bash
python three_stage_transmittance.py \
  --data_npz preprocessed_t_data.npz \
  --num_epochs 100 \
  --batch_size 1024
```

예상 출력 패턴:
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageA`
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageB`
- `outputs_three_stage_YYYYMMDD_HHMMSS/stageC`

### 6) Stage A/B/C 모델 평가

```bash
python three_stage_transmittance_evaluation.py \
  --model_dir outputs_three_stage_YYYYMMDD_HHMMSS \
  --data_npz preprocessed_t_data.npz \
  --sample_count 8
```

### 7) 선택 사항: 신경망 vs S4 일관성 검증

```bash
python FilterShapeS4_Evaluator_Transmittance.py \
  --npz_file preprocessed_t_data.npz \
  --spec2shape_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageC/spec2shape_stageC.pt \
  --shape2spec_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageA/shape2spec_stageA.pt \
  --n_samples 4
```

## ⚙️ 주요 CLI 옵션

### S4 런처 (`ms_final.sh`, `ms_resume_allargs.sh`)

| Flag | 의미 | 기본값 |
|---|---|---|
| `-ns`, `--numshapes` | 생성할 shape 개수 | `100000` |
| `-r`, `--seed` | 랜덤 시드 | `88888` |
| `-p`, `--prefix` | Prefix/재개 키 | `""` |
| `-g`, `--numg` | 기저/기하 파라미터 | `80` |
| `-bo`, `--baseouter` | 기본 외곽 경계 오프셋 | `0.25` |
| `-ro`, `--randouter` | 랜덤 외곽 오프셋 | `0.20` |

### 학습 (`three_stage_transmittance.py`)

| Flag | 용도 | 기본값 |
|---|---|---|
| `--preprocess` | 전처리 모드 실행 | `False` |
| `--input_folder` | 병합 CSV 폴더 | `""` |
| `--output_npz` | 출력 NPZ 경로 | `preprocessed_data.npz` |
| `--data_npz` | 학습에 사용할 NPZ | `""` |
| `--csv_file` | NPZ 미사용 시 CSV 대체 입력 | `""` |
| `--test` | 테스트 모드(학습 생략) | `False` |
| `--num_epochs` | 에폭 수 | `10` |
| `--batch_size` | 배치 크기 | `4096` |

### 평가 (`three_stage_transmittance_evaluation.py`)

| Flag | 용도 | 기본값 |
|---|---|---|
| `--model_dir` | `stageA/B/C`를 포함한 루트 디렉터리 | required |
| `--data_npz` | 평가용 NPZ 입력 | `""` |
| `--csv_file` | 평가용 CSV 입력 | `""` |
| `--output_dir` | 출력 디렉터리 재정의 | `model_dir` 내부 자동 생성 |
| `--sample_count` | 시각화 샘플 수 | `4` |
| `--seed` | 샘플링 랜덤 시드 | `23` |
| `--font_scale` | 플롯 폰트 스케일 | `1.0` |
| `--batch_size` | 평가 배치 크기 | `32` |
| `--plot_only` | 곡선만 그리기 | `False` |

### S4 일관성 평가기 (`FilterShapeS4_Evaluator_Transmittance.py`)

| Flag | 용도 | 기본값 |
|---|---|---|
| `--npz_file` | 전처리된 NPZ 파일 | `preprocessed_t_data.npz` |
| `--spec2shape_ckpt` | Stage C 체크포인트 | `outputs_three_stage_20250322_145925/stageC/spec2shape_stageC.pt` |
| `--shape2spec_ckpt` | Stage A 체크포인트 | `outputs_three_stage_20250322_145925/stageA/shape2spec_stageA.pt` |
| `--n_samples` | 확인할 샘플 수 | `4` |
| `--seed` | 랜덤 시드 | `23` |
| `--max_workers` | 병렬 S4 워커 수 | `4` |
| `--out_folder` | 출력 폴더 | 타임스탬프 기준 자동 생성 |

## 🧪 빠른 스모크 실행

```bash
./ms_final.sh -ns 1000 -r 42 -p smoke -g 40 -bo 0.25 -ro 0.20
python merge_s4_data_full.py --prefix smoke
python -c "import pandas as pd; p='merged_s4_shapes_smoke.csv'; d=pd.read_csv(p).rename(columns={'folder_key':'prefix','NQ':'nQ'}); d.to_csv(p,index=False)"
python three_stage_transmittance.py --preprocess --input_folder . --output_npz smoke.npz
python three_stage_transmittance.py --data_npz smoke.npz --num_epochs 5 --batch_size 128
```

## 🔬 연구 맥락

핵심 역설계 문제는 여러 결정화 상태에서의 투과 스펙트럼으로부터 C4 대칭 메타표면 구조를 추정하는 것입니다. 현재 투과율 파이프라인은 다음 가정을 둡니다.

- 샘플당 11개 결정화 상태 (`c` 값, 전처리 시 정렬)
- 상태당 100개 파장 bin (`T@...` 컬럼)
- 최대 4개 Q1 정점을 `(presence, x, y)`로 인코딩

이 저장소에는 정식 투과율 파이프라인 외에도 `AVIRIS*`, `noise_experiment_*`, `archived/` 같은 탐색적 브랜치가 포함되어 있습니다.

## 📚 인용

이 저장소를 사용하거나 본 연구를 기반으로 확장한 경우, 아래 논문을 인용해 주세요.

```bibtex
@article{chen2025inverse,
  title={Inverse Design of Metasurface for Spectral Imaging},
  author={Chen, Rongzhou and Nie, Haitao and Zhu, Shuo and Zhao, Yaping and Wang, Chutian and Lam, Edmund Y},
  journal={arXiv preprint arXiv:2510.21924},
  year={2025}
}
```

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

분광 이미징을 위한 메타표면 역설계를 다루는 스크립트 중심 연구 저장소입니다(과거 명칭: `inverse_metasurface`). 핵심 워크플로는 다음을 결합합니다.

- 물리 기반 RCWA 시뮬레이션(S4 + Lua)
- 데이터 병합 및 형상(shape) 정보 연결
- 3단계 PyTorch 학습(`shape -> spectrum`, `spectrum -> shape`, 연쇄 파인튜닝)
- 정량/정성 평가 및 선택적 neural-vs-S4 일관성 검증

## ✨ 한눈에 보기

| 항목 | 설명 |
|---|---|
| 🎯 주요 과제 | 목표 투과 스펙트럼으로부터 C4 대칭 메타표면 기하를 추정 |
| 🔬 시뮬레이터 | 셸 런처와 `.lua` 스크립트에서 호출되는 `../build/S4` |
| 🧠 학습 파이프라인 | Stage A `shape -> spectra`, Stage B `spectra -> shape`, Stage C `spectra -> shape -> spectra` |
| 📦 데이터 계약 | 병합 CSV(`T@...`, 메타데이터, `vertices_str`) -> 압축 NPZ(`uids`, `spectra`, `shapes`) |
| 🧪 평가 | MSE 지표, 단계별 시각화, 선택적 신규 S4 재시뮬레이션 |

## 🧭 End-to-End 워크플로

1. `results/`에 시뮬레이션 출력, `shapes/`에 폴리곤 파일을 생성합니다.
2. S4 CSV 파일들을 병합하고 shape vertex를 연결합니다.
3. 학습 호환성을 위해 병합 컬럼명을 정규화합니다.
4. 병합 CSV를 NPZ 텐서로 전처리합니다.
5. Stage A/B/C 모델을 학습합니다.
6. 체크포인트를 평가하고 동작을 시각화합니다.
7. 필요 시 예측 shape의 스펙트럼을 신규 S4 실행 결과와 비교합니다.

## 🧱 저장소 구조

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

## 🛠️ 사전 요구사항

| 의존성 | 비고 |
|---|---|
| Linux + Bash | 런처 스크립트는 셸 실행을 기준으로 작성됨 |
| Python 3.9 | `iccp.yaml`과 일치 |
| Conda | 재현성을 위해 권장 |
| S4 바이너리 | `../build/S4` 경로를 기대 |
| CUDA GPU (선택) | 학습/평가 속도 향상 |

## 🚀 설정

### 1) 클론 후 디렉터리 진입

```bash
git clone <your-repo-url> inverse_metasurface
cd inverse_metasurface
```

### 2) 환경 생성(권장)

```bash
conda env create -f iccp.yaml
conda activate iccp
```

대안(재현성 낮음):

```bash
pip install -r pip_requirements.txt
```

### 3) 스크립트가 기대하는 시뮬레이터 경로 확인

```bash
ls -l ../build/S4
```

### 4) (선택) 런처 실행 권한 부여

```bash
chmod +x ms.sh ms_final.sh ms_resume.sh ms_resume_allargs.sh ms_resume_random_state.sh
```

## ▶️ 실전 사용법

### A) RCWA 시뮬레이션 데이터 생성

간단 런처:

```bash
./ms.sh -ns 10000 -r 12345
```

파라미터 지정 런처:

```bash
./ms_final.sh \
  -ns 10000 \
  -r 12345 \
  -p myrun \
  -g 80 \
  -bo 0.35 \
  -ro 0.30
```

재개(resume) 중심 런처:

```bash
./ms_resume_allargs.sh \
  -ns 10000 \
  -r 12345 \
  -p myrun \
  -g 80 \
  -bo 0.35 \
  -ro 0.30
```

참고:
- 런처는 `NQ=1..4`를 병렬 실행합니다.
- 스크립트는 `-t 32` 옵션으로 `../build/S4`를 호출합니다.

### B) S4 출력 병합 및 shape vertex 연결

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

### F) 학습된 모델 평가

```bash
python three_stage_transmittance_evaluation.py \
  --model_dir outputs_three_stage_YYYYMMDD_HHMMSS \
  --data_npz preprocessed_t_data.npz \
  --sample_count 8
```

### G) 선택적 neural-vs-S4 일관성 검증

```bash
python FilterShapeS4_Evaluator_Transmittance.py \
  --npz_file preprocessed_t_data.npz \
  --spec2shape_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageC/spec2shape_stageC.pt \
  --shape2spec_ckpt outputs_three_stage_YYYYMMDD_HHMMSS/stageA/shape2spec_stageA.pt \
  --n_samples 4
```

## ⚙️ 주요 CLI 옵션

### S4 런처 (`ms_final.sh`, `ms_resume_allargs.sh`)

| 플래그 | 의미 | 기본값 |
|---|---|---|
| `-ns`, `--numshapes` | 생성할 shape 개수 | `100000` |
| `-r`, `--seed` | 랜덤 시드 | `88888` |
| `-p`, `--prefix` | 접두어/재개 키 | `""` |
| `-g`, `--numg` | Basis/grid 파라미터 | `80` |
| `-bo`, `--baseouter` | 기본 외곽 경계 오프셋 | `0.25` |
| `-ro`, `--randouter` | 랜덤 외곽 경계 오프셋 | `0.20` |

### 학습 (`three_stage_transmittance.py`)

| 플래그 | 의미 | 기본값 |
|---|---|---|
| `--preprocess` | 전처리 모드 실행 | `False` |
| `--input_folder` | 병합 CSV 파일이 있는 폴더 | `""` |
| `--output_npz` | 출력 NPZ 경로 | `preprocessed_data.npz` |
| `--data_npz` | 학습용 NPZ 데이터셋 | `""` |
| `--csv_file` | NPZ 미사용 시 CSV 대체 입력 | `""` |
| `--test` | 테스트 모드 | `False` |
| `--num_epochs` | 학습 epoch 수 | `10` |
| `--batch_size` | 배치 크기 | `4096` |

### 평가 (`three_stage_transmittance_evaluation.py`)

| 플래그 | 의미 | 기본값 |
|---|---|---|
| `--model_dir` | `stageA/B/C`를 포함한 디렉터리 | 필수 |
| `--data_npz` | NPZ 입력 | `""` |
| `--csv_file` | CSV 대체 입력 | `""` |
| `--output_dir` | 출력 디렉터리 재지정 | `model_dir` 하위 자동 생성 |
| `--sample_count` | 시각화 샘플 수 | `4` |
| `--seed` | 랜덤 시드 | `23` |
| `--font_scale` | 플롯 글꼴 스케일 | `1.0` |
| `--batch_size` | 평가 배치 크기 | `32` |
| `--plot_only` | 학습 곡선만 플로팅 | `False` |

### S4 일관성 평가기 (`FilterShapeS4_Evaluator_Transmittance.py`)

| 플래그 | 의미 | 기본값 |
|---|---|---|
| `--npz_file` | 입력 NPZ 파일 | `preprocessed_t_data.npz` |
| `--spec2shape_ckpt` | Stage C 체크포인트 경로 | `outputs_three_stage_20250322_145925/stageC/spec2shape_stageC.pt` |
| `--shape2spec_ckpt` | Stage A 체크포인트 경로 | `outputs_three_stage_20250322_145925/stageA/shape2spec_stageA.pt` |
| `--n_samples` | 평가 샘플 수 | `4` |
| `--seed` | 랜덤 시드 | `23` |
| `--max_workers` | S4 워커 스레드 수 | `4` |
| `--out_folder` | 출력 디렉터리 | 타임스탬프 기준 자동 생성 |

## 🧪 스모크 런

```bash
./ms_final.sh -ns 1000 -r 42 -p smoke -g 40 -bo 0.25 -ro 0.20
python merge_s4_data_full.py --prefix smoke
python -c "import pandas as pd; p='merged_s4_shapes_smoke.csv'; d=pd.read_csv(p).rename(columns={'folder_key':'prefix','NQ':'nQ'}); d.to_csv(p,index=False)"
mkdir -p merged_csvs && mv merged_s4_shapes_smoke.csv merged_csvs/
python three_stage_transmittance.py --preprocess --input_folder merged_csvs --output_npz smoke.npz
python three_stage_transmittance.py --data_npz smoke.npz --num_epochs 5 --batch_size 128
```

## 🔬 연구 맥락

현재 역설계 구성은 결정화 상태 전반의 투과도를 기반으로 C4 대칭 기하를 복원하도록 학습합니다. 현재 투과도 파이프라인은 다음을 가정합니다.

- shape 샘플당 결정화 행 11개(고유 `shape_uid` 기준 그룹화)
- 결정화 상태당 파장 bin 100개(`T@...` 컬럼)
- `4x3` 텐서 `(presence, x, y)`로 인코딩된 최대 4개의 Q1 제어점
- shape 시각화 및 일관성 검증을 위한 C4 대칭 기반 폴리곤 복원

이 저장소에는 기본 투과도 학습 경로 외에도 탐색적 브랜치(`AVIRIS*`, `noise_experiment*`, `archived/`)가 포함되어 있습니다.

## 🧯 문제 해결

| 증상 | 가능한 원인 | 해결 방법 |
|---|---|---|
| `../build/S4: No such file or directory` | 예상 상대 경로에 S4 바이너리가 없음 | `../build/S4`에 S4를 빌드/링크하거나 런처 경로를 수정 |
| `No transmission columns found` | CSV에 `T@...` 컬럼이 없음 | 병합 출력 포맷을 다시 확인 |
| `Must specify either --data_npz or --csv_file` | 학습/평가 데이터 인자가 누락됨 | 둘 중 하나의 입력을 명시적으로 전달 |
| `No valid shapes => SHIFT->Q1->UpTo4` | `vertices_str`가 비어있거나 유효하지 않음, 또는 Q1 필터링으로 전체 샘플 제거 | shape 파일과 병합 출력을 검증 |
| `--prefix`에 대한 병합 결과가 비어 있음 | 접두어가 `results/` 파일명과 일치하지 않음 | 정확한 파일명 접두어를 확인 후 병합 재실행 |
| 평가 체크포인트 누락 | `stageA/B/C` 체크포인트 파일 없음 | `--model_dir`가 완전한 출력 폴더를 가리키는지 확인 |

## 🤝 기여

재현성, 테스트, 문서 품질 개선을 위한 기여를 환영합니다.

권장 절차:

1. 범위와 기대 동작을 포함해 이슈를 등록합니다.
2. 목적이 명확한 브랜치를 생성합니다.
3. 실행 가능한 명령과 출력 결과를 포함해 Pull Request를 제출합니다.
4. 가능하면 변경 범위를 하나의 워크플로로 제한합니다.

## 📄 라이선스

이 스냅샷의 저장소 루트에는 현재 `LICENSE` 파일이 없습니다. 사용 및 재배포 조건을 명확히 하려면 라이선스 파일을 추가하세요.

## 📚 인용

이 저장소를 사용하거나 본 연구를 기반으로 작업했다면 다음을 인용해 주세요.

```bibtex
@article{chen2025inverse,
  title={Inverse Design of Metasurface for Spectral Imaging},
  author={Chen, Rongzhou and Nie, Haitao and Zhu, Shuo and Zhao, Yaping and Wang, Chutian and Lam, Edmund Y},
  journal={arXiv preprint arXiv:2510.21924},
  year={2025}
}
```

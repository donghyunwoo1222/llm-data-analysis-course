# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 이 저장소는 무엇인가

"LLM 데이터 분석" 강의를 위한 개인 실습 저장소입니다. 결과물은 챕터별 Jupyter 노트북(`notebooks/chNN/` 아래)이며, 데이터 생성 → 정제 → 분석 과정을 다루고 종종 과제 템플릿(예: `notebooks/ch05/chapter05_assignment.md`)을 기준으로 작성됩니다. 내용과 주석은 대부분 한국어입니다. 애플리케이션 코드, 테스트 스위트, lint 설정은 없으며 — 요청받지 않는 한 새로 추가하지 마세요.

## 환경 & 명령어

- Python >=3.11, 가상환경은 이미 `.venv/`에 존재 (Windows: `.venv/Scripts/python.exe`).
- 의존성 설치: `pip install -r requirements.txt` (pandas, numpy, matplotlib, seaborn, scikit-learn, jupyter, google-genai, python-dotenv, faker, python-docx, tabulate, markdown, requests, beautifulsoup4).
- 로컬 `course_utils` 패키지는 `pyproject.toml`(`packages = ["course_utils"]`)을 통해 editable 모드로 설치되어 있어 작업 디렉터리와 무관하게 어떤 노트북에서든 import 가능 — 새로 클론한 환경에서 `import course_utils`가 실패하면 `pip install -e .`를 실행하세요.
- 노트북 실행: 저장소 루트에서 `jupyter lab` 또는 `jupyter notebook`.
- 샘플 이커머스 데이터 재생성: `python scripts/generate_sample_data.py` — 고정 시드(`SEED = 42`)와 `Faker`를 사용해 `data/raw/{customers,products,orders,order_items}.csv`를 생성.
- 타이타닉 데이터셋 다운로드/검증: `python scripts/prepare_titanic_data.py` (재다운로드하려면 `--force` 추가). `pandas-dev/pandas`의 특정 커밋에 고정된 `titanic.csv`를 가져온 뒤 shape, 컬럼, PK 유일성/범위, 타깃 클래스 개수, 결측치 개수를 검증하고 `data/titanic/train.csv`에 저장 — 이 스크립트의 `EXPECTED_*` 상수를 "유효한 데이터셋"의 기준으로 삼으세요.
- 비밀 값은 `.env`(gitignore 대상)에 두고 `.env.example`을 복사해서 사용: `GEMINI_API_KEY`, `GEMINI_MODEL_NAME`, `PUBLIC_DATA_API_KEY`, `NAVER_CLIENT_ID`/`NAVER_CLIENT_SECRET`. `python-dotenv`로 로드됩니다. 실제 키는 `.env.example`이나 노트북 셀에 절대 넣지 마세요.

## 경로 처리 패턴

노트북은 깊이가 제각각인 위치(`notebooks/ch04/`, `notebooks/ch05/` 등)에 있으므로 저장소 루트로의 경로를 하드코딩하면 안 됩니다. 대신 `course_utils/paths.py`를 사용합니다:

- `get_project_root(start_path=None)` — 주어진(또는 현재) 경로를 resolve한 뒤 `data/` 하위 폴더를 가진 디렉터리를 찾을 때까지 부모 방향으로 올라가며 탐색합니다. 파일시스템 루트에 도달할 때까지 찾지 못하면 `FileNotFoundError`를 발생시킵니다.
- `get_data_dir()` → `<project_root>/data/raw`
- `get_report_dir()` → `<project_root>/reports`

저장소 상대 경로가 필요한 새 노트북이나 스크립트를 추가할 때는 `../../data` 같은 상대 경로 대신 이 헬퍼 함수들을 사용하거나 `paths.py`를 같은 방식으로 확장하세요.

## 데이터 구조

- `data/raw/` — `scripts/generate_sample_data.py`로 만든 가상 이커머스 테이블(customers/products/orders/order_items).
- `data/titanic/train.csv` — `scripts/prepare_titanic_data.py`가 생성·검증하는 표준 891행 타이타닉 학습셋.
- `data/processed/`, `data/external/` — 이후 챕터를 위한 빈 스캐폴드 폴더(`.gitkeep`만 존재).
- `automation/`, `prompts/`, `src/`, `reports/` — 향후 챕터를 위한 빈 스캐폴드 폴더. 현재 존재하는 것 이상의 구조를 가정하지 마세요.

## 노트북 구성 관련 참고사항

chapter04 노트북이 두 곳에 존재합니다: `chapter04/chapter04.ipynb`(저장소 루트)와 `notebooks/ch04/chapter04.ipynb`, 그리고 `notebooks/ch04/01.ipynb`, `03.ipynb`, `moduletest.ipynb`도 함께 있습니다. `notebooks/chNN/`이 현재 사용 중인 규칙이므로, 어느 쪽이 정식본이라고 가정하기 전에 실제로 어떤 사본이 수정/참조되고 있는지 확인하세요.

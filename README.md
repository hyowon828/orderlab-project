# OrderLab — Python CI/CD Practice

Python 3.14 기반 주문 계산, CSV 매출 집계, FastAPI 주문 관리 실습 프로젝트입니다. Ruff, pytest, GitHub Actions를 사용해 코드 품질 검사, 자동 테스트, GitHub Pages 정적 보고서 배포를 구현했습니다.

## 1. 프로젝트 구성

| 경로 | 역할 |
| --- | --- |
| `orderlab/pricing.py` | 가격·수량 계산, 할인 및 배송비 처리 |
| `orderlab/csv_report.py` | CSV 주문 데이터 매출 집계 |
| `orderlab/api.py` | FastAPI 주문 생성·조회 API |
| `tests/` | 계산·CSV·API 테스트 및 fixture |
| `data/sample_orders.csv` | 매출 집계용 샘플 데이터 |
| `generate_report.py` | `site/index.html` 정적 보고서 생성 |
| `.github/workflows/ci.yml` | Ruff 검사 및 Ubuntu/Windows 병렬 테스트 |
| `.github/workflows/ci-cd.yml` | CI 검사 후 GitHub Pages 자동 배포 |

## 2. 개발 환경 및 실행 방법

Python 3.14 사용. 프로젝트 루트에서 실행합니다.

```bash
python -m venv .venv
```

Git Bash (Windows):

```bash
source .venv/Scripts/activate
```

macOS / Linux:

```bash
source .venv/bin/activate
```

의존성 설치:

```bash
python -m pip install -r requirements-dev.txt
```

코드 품질 검사:

```bash
ruff check .
ruff format --check .
```

전체 테스트:

```bash
python -m pytest -q
```

커버리지 XML 생성:

```bash
python -m pytest --cov=orderlab --cov-report=xml
```

정적 HTML 보고서 생성:

```bash
python generate_report.py
```

생성 결과는 `site/index.html`입니다.

## 3. 단계별 테스트

### Level 1 — 가격·수량 계산

- 정상 입력: `10,000 × 2 = 20,000`
- 경계값: 0원 입력 허용
- 예외: 음수 입력 시 `ValueError`

### Level 2 — 할인·배송 규칙

- 할인율 0~100% 적용
- 할인 후 금액이 50,000원 이상이면 무료배송
- 0원은 배송비 없음, 0원 초과 50,000원 미만은 배송비 3,000원
- 49,999원 / 50,000원 / 50,001원 경계값 검증
- 할인율 -1% / 101%에서 `ValueError` 검증
- `pytest.mark.parametrize`로 반복 테스트 정리

### Level 3 — CSV 매출 집계

- 정상 CSV, 빈 CSV, 누락된 파일, 숫자 변환 오류 검증
- `tmp_path`로 테스트 파일 격리
- `tests/conftest.py`의 `sample_csv` fixture 재사용
- `coverage.xml`을 GitHub Actions artifact로 보관

### Level 4 — FastAPI 주문 API

- `POST /orders`: 주문 생성, HTTP 201
- `GET /orders/{id}`: 주문 조회, HTTP 200
- 잘못된 수량: HTTP 422
- 존재하지 않는 주문: HTTP 404
- `TestClient`와 fixture로 테스트마다 독립된 앱 생성
- Ubuntu/Windows 병렬 CI 검사

## 4. GitHub Actions CI/CD

### CI

Push 및 Pull Request에서 Ruff lint, Ruff format, pytest를 자동 실행합니다. `.github/workflows/ci.yml`은 Ubuntu/Windows 테스트를 병렬 실행합니다.

### CD

- Pull Request: 품질 검사만 수행하고 Pages 배포는 제외
- `main` 브랜치 Push: `build-report` 성공 후 `deploy` 실행
- CI 실패: `needs: build-report` 조건으로 배포 차단
- 배포 대상: Python으로 생성한 `site/index.html`

## 5. 실패 재현 및 해결 기록

### 의도적인 pytest 실패 — PR #2

1. `experiment/ci-failure` 브랜치에서 계산 결과와 다른 기대값을 설정해 테스트 실패를 재현했습니다.
2. GitHub Actions `OrderLab CI/CD #6`에서 `build-report` 실패 및 `deploy` 미실행을 확인했습니다.
3. 테스트 기대값을 수정하고 Ruff 포맷 검사까지 통과하도록 정리했습니다.
4. CI가 다시 성공한 뒤 PR #2를 `main`에 병합했습니다.

### PR 배포 제외 및 main 배포 — PR #3

1. `experiment/deploy-block` 브랜치에서 PR #3을 생성했습니다.
2. PR의 CI 검사 3개가 성공하고, 배포는 실행되지 않는 것을 확인했습니다.
3. `main` 병합 후 `OrderLab CI/CD #10`에서 `build-report`와 `deploy`가 모두 성공했습니다.

## 6. 결과 및 증빙

- 저장소: https://github.com/hyowon828/orderlab-project
- GitHub Actions 실행 기록: https://github.com/hyowon828/orderlab-project/actions
- 실패 재현 PR #2: https://github.com/hyowon828/orderlab-project/pull/2
- 배포 검증 PR #3: https://github.com/hyowon828/orderlab-project/pull/3
- Pages 배포 URL: https://hyowon828.github.io/orderlab-project/

최종 확인 기록: pytest 18개 통과, Ruff 검사 통과, `main`에서 GitHub Pages 자동 배포 성공.

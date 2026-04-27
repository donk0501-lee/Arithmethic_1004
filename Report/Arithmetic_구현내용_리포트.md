# Arithmetic 구현 내용 리포트

본 문서는 `3.Arithmetic_1004` 프로젝트의 **소스 기준** 구현 내용을 한 문서로 정리한 리포트입니다. (웹 UI·DB 없음, 순수 콘솔 + 순수 연산 모듈 분리)

---

## 1. 구현 범위

| 구분 | 내용 |
|------|------|
| 연산 API | `add`, `subtract`, `multiply`, `divide` (모두 `float` 인자·반환) |
| 나눗셈 | 제수 `b == 0`이면 `ZeroDivisionError("0으로 나눌 수 없습니다.")` |
| 실행 방식 | `main.py` — 표준 입출력으로 두 수·연산자 입력 후 결과 출력 |
| 검증 | `tests/test_operations.py` — `unittest`, 연산 모듈만 대상 (9케이스) |

---

## 2. 디렉터리·파일 역할

```
3.Arithmetic_1004/
├── main.py                    # 콘솔 진입점, I/O·디스패치·예외 메시지
├── arithmetic/
│   ├── __init__.py            # 패키지 공개 API (`__all__`)
│   └── operations.py        # 사칙연산 순수 함수
├── tests/
│   └── test_operations.py    # 단위 테스트
├── Report/
│   ├── Arithmetic_프로젝트_보고서.md
│   └── Arithmetic_구현내용_리포트.md   # 본 문서
└── Prompting/                 # 과제 프롬프트 보관 (런타임 무관)
```

| 파일 | 책임 |
|------|------|
| `arithmetic/operations.py` | 사칙연산 정의만. 입출력·콘솔 의존 없음. |
| `arithmetic/__init__.py` | `from arithmetic import add, …` 지원. |
| `main.py` | `_prompt_float`, `_prompt_operator`, `_OPERATIONS` 맵, `main()` 루프. |
| `tests/test_operations.py` | `operations` 공개 계약 검증. |

---

## 3. 모듈별 동작 요약

### 3.1 `arithmetic.operations`

- **입력**: 피연산자 두 개 `(a, b)`.
- **출력**: 연산 결과 `float`. `divide`만 조건 불만족 시 예외.
- **0으로 나누기**: `if b == 0:` 후 `raise ZeroDivisionError(...)` — 언어 기본 `/`만 두지 않고 메시지를 한곳에서 통일.

### 3.2 `main`

1. 안내 문구 출력 후 첫 번째 수·연산자·두 번째 수 순 입력.
2. 숫자 변환 실패 시 `ValueError` → `입력 오류: 숫자 형식이 아닙니다.`
3. 연산자가 `_OPERATIONS`에 없으면 `지원하지 않는 연산자입니다: …`
4. 매핑된 함수 호출 시 `ZeroDivisionError` → `연산 오류: …`
5. 성공 시 `결과: {result}` 출력.

**연산자 매핑**: `+`, `-`, `*`, `/` 및 곱셈 별칭 `x`, `X`.

---

## 4. 예외·오류 처리 매트릭스

| 상황 | 발생 위치 | 사용자/호출자 관점 |
|------|-----------|-------------------|
| 제수 0 | `divide` | `ZeroDivisionError` (도메인). 콘솔에서는 `연산 오류`로 표시. |
| 비숫자 입력 | `float()` in `_prompt_float` | `main`에서 `입력 오류`로 표시. |
| 미지원 연산자 | `_OPERATIONS.get` → `None` | 안내 문구 후 종료 (예외 아님). |

---

## 5. 단위 테스트 목록

`python -m unittest discover -s tests -v` 기준 테스트 클래스·케이스:

| 클래스 | 케이스 | 검증 내용 |
|--------|--------|-----------|
| `TestAdd` | `test_integers` | 정수 덧셈 |
| `TestAdd` | `test_floats` | 부동소수점 덧셈 (`assertAlmostEqual`) |
| `TestSubtract` | `test_positive_result` | 양수 차 |
| `TestSubtract` | `test_negative_result` | 음수 차 |
| `TestMultiply` | `test_basic` | 일반 곱 |
| `TestMultiply` | `test_by_zero` | 0 곱하기 |
| `TestDivide` | `test_basic` | 일반 나눗셈 |
| `TestDivide` | `test_divide_by_zero_raises` | 0 제수 시 예외 + 메시지에 `0으로 나눌` 포함 |
| `TestDivide` | `test_negative_divisor` | 음수 제수 |

**합계**: 9개 테스트.

---

## 6. 실행 방법

프로젝트 루트(`3.Arithmetic_1004`)에서 실행합니다.

**단위 테스트**

```powershell
Set-Location "c:\Users\usejen_id\Downloads\CursorAI\src\3.Arithmetic_1004"
python -m unittest discover -s tests -v
```

**콘솔 프로그램**

```powershell
python main.py
```

---

## 7. 설계·확장 메모

- **SRP**: 계산 규칙은 `operations`, 사용자와의 상호작용은 `main`에만 둠.
- **확장**: 새 연산은 `operations`에 함수 추가 후 `main._OPERATIONS`에 기호만 등록하면 됨.
- **추가 분리(선택)**: 표현식 파싱·평가를 별도 모듈로 두면 설계 문서의 다층 구조와 같이 확장할 수 있습니다.

---

## 8. 결론

구현은 **순수 로직 패키지 `arithmetic`**과 **콘솔 호스트 `main`**의 이중 구조로 정리되어 있으며, **0으로 나누기**는 도메인에서 예외로 정의하고 UI 계층에서 메시지로 변환합니다. 단위 테스트 9건으로 핵심 연산을 검증합니다.

---

*작성 기준: 저장소 내 `main.py`, `arithmetic/*.py`, `tests/test_operations.py` 내용 반영*

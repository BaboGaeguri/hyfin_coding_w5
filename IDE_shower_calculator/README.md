# IDE_shower_calculator

VS Code의 IDE 기능과 파이썬 패키지 구조를 시연하기 위한 계산기 예제입니다.

## 구조

```
IDE_shower_calculator/
├── operations/
│   ├── __init__.py   # 패키지 진입점, 외부에 노출할 함수 모아두기
│   ├── basic.py      # 사칙연산 (add, subtract, multiply, divide)
│   ├── advanced.py   # 고급 연산 (power, sqrt, factorial)
│   └── finance.py    # 재무 계산 (단리, 복리, 현재가치, 미래가치, CAGR)
├── main.py
└── README.md
```

## 실행

```bash
python main.py
```

## 시연 포인트

### IDE 기능

| 기능 | 확인 방법 |
| --- | --- |
| 파일 탐색기 | 좌측 사이드바에서 `operations/` 폴더 펼치고 파일 간 이동 |
| 자동완성 | `main.py`에서 `add(` 입력 시 매개변수 힌트 |
| 문법 오류 표시 | 일부러 오타(`pirnt`) → 빨간 줄 |
| Go to Definition | `main.py`의 `add`에 Ctrl+클릭 → `basic.py`로 점프 |
| 디버거 | `divide(10, 0)`으로 바꾸고 중단점 → 0 나누기 추적 |
| 내장 터미널 | `python main.py` 실행 |

### 패키지 구조 (`__init__.py`)

`operations/` 폴더 안에 `__init__.py`가 있으면 파이썬은 이 폴더를 **하나의 패키지**로 인식합니다.

`__init__.py`에서 하위 모듈의 함수를 미리 import 해두면, 외부에서는 내부 구조를 몰라도 됩니다.

```python
# operations/__init__.py
from .basic import add, subtract, multiply, divide
from .advanced import power, sqrt, factorial
```

덕분에 `main.py`에서 이렇게 쓸 수 있습니다.

```python
# 좋음 (__init__.py 덕분)
from operations import add, power

# __init__.py가 없거나 비어있다면 이렇게 써야 함
from operations.basic import add
from operations.advanced import power
```

**비교 실습 아이디어:** `__init__.py`를 잠깐 비워보고 `main.py`가 깨지는 걸 보여준 뒤, 다시 채우면 동작하는 흐름.

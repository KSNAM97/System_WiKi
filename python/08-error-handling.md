# Python 08 — 에러 처리

## 에러 메시지 읽는 법

파이썬 코드를 실행하다 오류가 발생하면, 인터프리터는 **어느 파일의 몇 번째 줄에서, 어떤 종류의 오류가, 왜 발생했는지**를 알려주는 메시지를 출력한다. 이 메시지를 읽는 법을 알면 원인을 훨씬 빠르게 찾을 수 있다.

```python
# error_sample.py
def divide(a, b):
    return a / b

result = divide(10, 0)
print(result)
```

```text
(base) C:\Users\guest\project> python error_sample.py
Traceback (most recent call last):
  File "error_sample.py", line 5, in <module>
    result = divide(10, 0)
  File "error_sample.py", line 3, in divide
    return a / b
ZeroDivisionError: division by zero
```

에러 메시지는 항상 다음 구조를 따른다.

```text
Traceback (most recent call last):   <- 오류가 발생하기까지의 호출 경로 시작
  File "파일명", line 줄번호, in 함수/모듈이름
    실제 코드 내용
  ...                                <- 호출 경로가 여러 단계면 이어서 나열
파일명 마지막 줄: 에러타입: 에러 메시지    <- 실제로 발생한 오류의 핵심
```

- 가장 마지막 줄 `ZeroDivisionError: division by zero`가 오류의 핵심이다. **에러 타입**(`ZeroDivisionError`)과 **구체적인 사유**(`division by zero`, 0으로 나눔)를 알려준다.
- 그 위쪽의 `File "error_sample.py", line 3, in divide` 부분은 오류가 실제로 발생한 위치(파일명, 줄 번호, 어느 함수 안이었는지)를 알려준다.
- 오류를 마주치면 가장 먼저 맨 아래 줄에서 **에러 타입**을 확인하고, 그 위에서 **파일명과 줄 번호**를 확인하는 순서로 읽는 것이 효율적이다.

**자주 만나는 에러 타입**

| 에러 타입 | 의미 |
|-----------|------|
| `SyntaxError` | 문법 자체가 잘못됨 |
| `NameError` | 정의되지 않은 이름을 사용함 |
| `TypeError` | 타입이 맞지 않는 연산·호출 |
| `ValueError` | 타입은 맞지만 값 자체가 잘못됨 |
| `IndexError` | 리스트 등에서 범위를 벗어난 인덱스 접근 |
| `KeyError` | 딕셔너리에 없는 키로 접근 |
| `ZeroDivisionError` | 0으로 나눔 |
| `AttributeError` | 객체에 없는 속성·메서드에 접근 |
| `ModuleNotFoundError` | 존재하지 않거나 설치되지 않은 모듈을 import |

**정리**: 파이썬 에러 메시지는 맨 아래 줄에 에러 타입과 구체적인 사유를, 그 위쪽에 오류가 발생한 파일명과 줄 번호를 담고 있으며, 오류를 마주치면 맨 아래부터 위로 읽어 올라가며 "무슨 오류인지 → 어디서 발생했는지" 순서로 파악하는 것이 가장 빠르게 원인을 찾는 방법이다.

## 스택 트레이스 해석

함수가 여러 단계로 중첩되어 호출된 상태에서 오류가 발생하면, `Traceback`에는 오류가 발생하기까지 거쳐온 **모든 호출 단계**가 순서대로 나열된다. 이를 **스택 트레이스(stack trace)**라고 한다.

```python
# stack_trace_sample.py
def level3(n):
    return 10 / n

def level2(n):
    return level3(n)

def level1(n):
    return level2(n)

print(level1(0))
```

```text
(base) C:\Users\guest\project> python stack_trace_sample.py
Traceback (most recent call last):
  File "stack_trace_sample.py", line 10, in <module>
    print(level1(0))
  File "stack_trace_sample.py", line 8, in level1
    return level2(n)
  File "stack_trace_sample.py", line 5, in level2
    return level3(n)
  File "stack_trace_sample.py", line 2, in level3
    return 10 / n
ZeroDivisionError: division by zero
```

- 스택 트레이스는 위에서 아래로 **가장 먼저 호출된 지점부터 실제 오류가 발생한 지점까지**의 순서로 나열된다. 즉 맨 위(`<module>`, 프로그램의 최상위 실행 지점)가 제일 먼저 호출된 곳이고, 맨 아래(`level3` 안의 `return 10 / n`)가 실제로 오류가 터진 지점이다.
- `File "...", line 10, in <module>` → `line 8, in level1` → `line 5, in level2` → `line 2, in level3` 순서로 읽으면, `level1(0)` 호출이 `level2(0)`을 호출하고 그것이 다시 `level3(0)`을 호출했다가 `10 / 0`에서 최종적으로 오류가 발생했다는 호출 경로 전체를 재구성할 수 있다.
- 실전에서 오류를 디버깅할 때는 이 경로를 따라가며 "어느 함수까지는 정상적으로 호출되었고, 어느 지점에서 잘못된 값(`n=0`)이 전달되었는가"를 역추적하는 것이 핵심이다.

**정리**: 스택 트레이스는 함수 호출이 중첩된 상태에서 오류가 발생했을 때 거쳐온 모든 호출 단계를 위(가장 먼저 호출된 지점)에서 아래(실제 오류 발생 지점)로 나열해 보여주며, 이 경로를 순서대로 따라가면 잘못된 값이 어느 함수에서부터 흘러 들어왔는지 역추적하여 근본 원인을 찾을 수 있다.

## try/except로 예외 처리하기

프로그램이 오류로 인해 강제 종료되지 않고 계속 실행되도록 하려면 **try/except** 문으로 **예외 처리(exception handling)**를 한다.

```python
def divide(a, b):
    try:
        result = a / b
        print(f"결과: {result}")
    except ZeroDivisionError:
        print("0으로 나눌 수 없습니다.")

divide(10, 2)
divide(10, 0)
print("프로그램은 계속 실행됩니다.")
```

```text
(base) C:\Users\guest\project> python try_except.py
결과: 5.0
0으로 나눌 수 없습니다.
프로그램은 계속 실행됩니다.
```

- `try` 블록 안의 코드를 실행하다가 지정한 종류의 예외(`ZeroDivisionError`)가 발생하면, 프로그램이 그대로 멈추는 대신 `except` 블록으로 넘어가 처리한 뒤 이후 코드를 계속 실행한다.

**여러 종류의 예외를 각각 처리하기**

```python
def safe_divide(a, b):
    try:
        return a / b
    except ZeroDivisionError:
        print("0으로 나눌 수 없습니다.")
    except TypeError:
        print("숫자가 아닌 값이 입력되었습니다.")

safe_divide(10, 0)
safe_divide(10, "two")
```

```text
(base) C:\Users\guest\project> python multi_except.py
0으로 나눌 수 없습니다.
숫자가 아닌 값이 입력되었습니다.
```

- `except`를 여러 번 이어 쓰면 서로 다른 종류의 오류를 각각 다른 방식으로 처리할 수 있다.

**예외 객체를 변수로 받아 메시지 확인하기**

```python
try:
    numbers = [1, 2, 3]
    print(numbers[5])
except IndexError as e:
    print(f"오류 발생: {e}")
```

```text
(base) C:\Users\guest\project> python except_as.py
오류 발생: list index out of range
```

- `except 예외타입 as 변수명:`으로 예외 객체 자체를 변수에 담으면, 오류 메시지를 직접 활용해 더 자세한 로그를 남기거나 사용자에게 안내할 수 있다.

**else와 finally**

```python
def load_config(value):
    try:
        number = int(value)
    except ValueError:
        print("숫자로 변환할 수 없습니다.")
    else:
        print(f"변환 성공: {number}")
    finally:
        print("설정 로딩 작업 종료.")

load_config("10")
load_config("abc")
```

```text
(base) C:\Users\guest\project> python else_finally.py
변환 성공: 10
설정 로딩 작업 종료.
숫자로 변환할 수 없습니다.
설정 로딩 작업 종료.
```

- `else` 블록은 `try` 블록에서 예외가 전혀 발생하지 않았을 때만 실행된다.
- `finally` 블록은 예외 발생 여부와 관계없이 **항상** 실행되며, 파일 닫기나 연결 종료처럼 성공/실패와 무관하게 반드시 수행해야 하는 마무리 작업에 사용한다.

**정리**: `try` 블록에서 예외가 발생하면 `except` 블록이 대신 실행되어 프로그램이 중단되지 않고 계속 흐를 수 있으며, 여러 `except`로 예외 종류별로 다르게 대응하거나 `as`로 예외 메시지를 확인할 수 있고, `else`는 예외가 없을 때만, `finally`는 예외 발생 여부와 무관하게 항상 실행되어 마무리 작업을 보장한다.

## 에러 발생 후에도 계속 실행해야 하는 경우

일반적인 스크립트는 오류가 나면 프로그램이 멈춰도 큰 문제가 없지만, **웹 서버**처럼 한 번 시작되면 계속 실행되어야 하는 프로그램은 사정이 다르다.

```python
def handle_request(request_id, data):
    try:
        result = 100 / data
        print(f"[요청 {request_id}] 처리 성공: {result}")
    except ZeroDivisionError:
        print(f"[요청 {request_id}] 처리 실패: 잘못된 값(0)")
    except Exception as e:
        print(f"[요청 {request_id}] 알 수 없는 오류: {e}")

requests = [("req-1", 10), ("req-2", 0), ("req-3", 5)]

for request_id, data in requests:
    handle_request(request_id, data)

print("서버는 계속 다음 요청을 받을 준비가 되어 있습니다.")
```

```text
(base) C:\Users\guest\project> python server_simulation.py
[요청 req-1] 처리 성공: 10.0
[요청 req-2] 처리 실패: 잘못된 값(0)
[요청 req-3] 처리 성공: 20.0
서버는 계속 다음 요청을 받을 준비가 되어 있습니다.
```

- 실제 웹 서버는 여러 사용자의 요청을 순서대로(또는 동시에) 처리하는데, 요청 하나에서 오류가 발생했다고 서버 전체가 멈춰버리면 그 뒤에 들어오는 모든 사용자의 요청까지 처리할 수 없게 된다.
- 그래서 요청을 처리하는 로직을 `try/except`로 감싸, 특정 요청에서 오류가 나더라도 그 요청만 실패 처리하고 서버 프로세스 자체는 계속 살아서 다음 요청을 받을 수 있도록 설계하는 것이 일반적인 패턴이다.
- `except Exception as e:`처럼 가장 넓은 범위의 예외 타입을 마지막에 두면, 미처 예상하지 못한 종류의 오류까지 포괄적으로 잡아내어 서버가 예기치 않게 죽는 상황을 방지할 수 있다. 다만 `Exception`으로 모든 것을 뭉뚱그려 잡으면 실제 원인을 놓치기 쉬우므로, 구체적인 예외 타입을 먼저 처리하고 `Exception`은 마지막 안전망으로만 사용하는 것이 바람직하다.

**정리**: 한 번 실행되면 계속 동작해야 하는 웹 서버 같은 프로그램은 요청 하나하나를 처리하는 로직을 `try/except`로 감싸, 개별 요청의 실패가 프로그램 전체의 중단으로 이어지지 않도록 설계해야 하며, 구체적인 예외를 먼저 처리하고 `except Exception`을 마지막 안전망으로 두는 방식이 실무에서 널리 쓰이는 패턴이다.

## VS Code 디버거 기초

`print()`로 값을 하나씩 찍어보며 오류 원인을 추적할 수도 있지만, VS Code의 내장 **디버거(debugger)**를 사용하면 코드를 한 줄씩 실행하며 그 시점의 변수 상태를 직접 눈으로 확인할 수 있어 훨씬 효율적이다.

**중단점(breakpoint) 설정하기**

```text
1. VS Code에서 .py 파일을 연다
2. 확인하고 싶은 줄 번호 왼쪽 여백을 클릭한다 -> 빨간 점(중단점)이 표시된다
3. 상단 메뉴 "Run" -> "Start Debugging" (또는 F5) 클릭
4. 코드 실행이 중단점에서 자동으로 멈춘다
```

- 중단점을 설정한 줄에 도달하면 프로그램 실행이 그 자리에서 일시정지되며, 그 시점까지 계산된 모든 변수의 값을 확인할 수 있다.

**디버거 실행 중 확인할 수 있는 정보**

```text
좌측 "RUN AND DEBUG" 패널 구성
- VARIABLES : 현재 시점의 지역/전역 변수 값 목록
- WATCH     : 직접 지정한 표현식을 계속 감시
- CALL STACK: 현재 어느 함수 호출 경로를 거쳐 여기까지 왔는지 (스택 트레이스와 유사)
```

- `VARIABLES` 패널에서는 중단점이 걸린 시점의 모든 변수 값을 실시간으로 확인할 수 있어, 예상한 값과 실제 값이 다른 지점을 눈으로 바로 찾아낼 수 있다.
- `CALL STACK` 패널은 앞서 다룬 스택 트레이스와 같은 원리로, 현재 실행 위치까지 어떤 함수들을 거쳐왔는지 순서대로 보여준다.

**단계별 실행 버튼**

```text
Step Over (F10)   : 현재 줄을 실행하고 다음 줄로 이동 (함수 호출은 내부로 들어가지 않음)
Step Into (F11)   : 현재 줄이 함수 호출이면 그 함수 내부로 들어감
Step Out (Shift+F11): 현재 함수 실행을 끝까지 마치고 호출한 곳으로 돌아옴
Continue (F5)     : 다음 중단점을 만날 때까지 계속 실행
```

- `Step Over`로 한 줄씩 진행하다가, 의심되는 함수 호출을 만나면 `Step Into`로 그 함수 내부까지 따라 들어가며 값의 흐름을 추적하는 것이 기본적인 디버깅 흐름이다.

**정리**: VS Code 디버거는 원하는 줄에 중단점을 설정해 그 시점에 코드 실행을 멈추고, `VARIABLES` 패널로 그 시점의 변수 값을 직접 확인하며, `Step Over`/`Step Into`/`Step Out`으로 한 줄씩 또는 함수 내부까지 실행 흐름을 따라가며 오류의 원인을 찾을 수 있게 해주는 도구이며, `print()`를 여러 곳에 넣었다 지우는 방식보다 훨씬 체계적으로 디버깅할 수 있다.

## 흔한 실수: 오타와 버전 불일치

**오타로 인한 NameError**

```python
>>> messege = "안녕하세요"
>>> print(message)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
NameError: name 'message' is not defined. Did you mean: 'messege'?
```

- `messege`(오타)로 선언해놓고 `message`(정상 철자)로 읽으려다 발생하는 흔한 실수다. 최신 파이썬은 오류 메시지 뒤에 `Did you mean: '...'?`처럼 비슷한 이름을 제안해주기도 하므로, 이 힌트를 먼저 확인하는 것이 빠르다.

**들여쓰기 오타로 인한 IndentationError**

```python
def greet(name):
    print(f"{name}님, 안녕하세요.")
     print("추가 인사")
```

```text
  File "indent_error.py", line 3
    print("추가 인사")
    ^
IndentationError: unexpected indent
```

- 같은 블록 안의 코드는 들여쓰기 칸 수가 정확히 일치해야 한다. 스페이스와 탭을 섞어 쓰거나 한 칸이라도 어긋나면 `IndentationError`가 발생한다.

**버전 불일치로 인한 오류**

파이썬 버전이나 설치된 라이브러리 버전이 코드가 기대하는 것과 다르면, 문법 자체는 맞는데도 실행되지 않는 경우가 있다.

```python
# 파이썬 3.10 이상 문법(match문)을 3.9 이하에서 실행한 경우
match command:
    case "start":
        print("시작")
    case _:
        print("알 수 없는 명령")
```

```text
  File "match_test.py", line 1
    match command:
          ^
SyntaxError: invalid syntax
```

- `match` 문법은 파이썬 3.10부터 추가된 기능이므로, 그보다 낮은 버전에서 실행하면 문법 오류로 처리된다. 이런 경우 `python --version`으로 현재 실행 중인 버전을 먼저 확인해야 한다(PY-00 문서에서 다룬 것처럼, 설치 환경과 실행 환경이 다르면 같은 명령의 결과도 달라질 수 있다).
- 라이브러리 버전 차이도 마찬가지다. 특정 버전에서만 제공되는 함수나 매개변수를 사용하는 코드를 오래된 버전의 라이브러리에서 실행하면 `AttributeError`나 `TypeError`가 발생할 수 있다.

**정리**: 오타로 인한 `NameError`는 오류 메시지의 `Did you mean:` 힌트를 먼저 확인하고, 들여쓰기가 어긋나면 `IndentationError`가 발생하므로 같은 블록 안에서는 들여쓰기 칸 수를 정확히 맞춰야 하며, 코드가 특정 파이썬 버전이나 라이브러리 버전에서만 지원하는 문법·기능을 사용했는데 실행 환경의 버전이 그보다 낮으면 문법이 맞더라도 오류가 발생할 수 있으므로 버전 확인이 디버깅의 기본 절차 중 하나다.

## 특정 버전으로 pip 설치하기

라이브러리의 최신 버전이 현재 프로젝트와 호환되지 않을 때는, PY-00 문서에서 다룬 것처럼 `==`로 정확한 버전을 지정해 설치할 수 있다.

```text
(base) C:\Users\guest\project> pip install matplotlib==3.7.0
Collecting matplotlib==3.7.0
  Downloading matplotlib-3.7.0-cp312-cp312-win_amd64.whl (7.6 MB)
Installing collected packages: matplotlib
Successfully installed matplotlib-3.7.0
```

**이미 설치된 라이브러리를 특정 버전으로 다시 설치하기**

```text
(base) C:\Users\guest\project> pip install matplotlib==3.7.0 --force-reinstall
```

- 이미 다른 버전이 설치되어 있다면 `--force-reinstall` 옵션을 추가해야 지정한 버전으로 확실히 교체된다.

**설치된 버전 확인하기**

```text
(base) C:\Users\guest\project> pip show matplotlib
Name: matplotlib
Version: 3.7.0
Summary: Python plotting package
```

- `pip show 라이브러리이름`으로 현재 환경에 실제로 설치된 버전을 확인할 수 있다. 오류가 버전 문제 때문인지 의심될 때 가장 먼저 확인해야 하는 명령이다.

**정리**: 라이브러리 버전 불일치로 인한 오류를 해결하려면 `pip install 라이브러리이름==버전번호`로 필요한 버전을 정확히 지정해 설치하고, 이미 다른 버전이 설치되어 있다면 `--force-reinstall` 옵션을 함께 사용하며, `pip show`로 현재 설치된 버전을 먼저 확인하는 것이 버전 관련 오류를 해결하는 출발점이다.

## 실습 예제 (EX1~EX4)

**EX1) 에러 메시지 읽고 원인 지점 찾기**
- 아래 코드를 실행했을 때 나오는 에러 메시지에서 에러 타입과 발생 줄 번호를 확인하고, 원인을 고쳐 정상 동작하게 만드시오.

```python
# ex01_fix_error.py (수정 전)
def get_average(scores):
    return sum(scores) / len(scores)

print(get_average([]))
```

```text
(base) C:\Users\guest\project> python ex01_fix_error.py
Traceback (most recent call last):
  File "ex01_fix_error.py", line 4, in <module>
    print(get_average([]))
  File "ex01_fix_error.py", line 2, in get_average
    return sum(scores) / len(scores)
ZeroDivisionError: division by zero
```

```python
# ex01_fix_error.py (수정 후)
def get_average(scores):
    if not scores:
        return 0
    return sum(scores) / len(scores)

print(get_average([]))
```

```text
(base) C:\Users\guest\project> python ex01_fix_error.py
0
```

- 에러 메시지의 마지막 줄에서 `ZeroDivisionError`를, 그 위 `line 2, in get_average`에서 원인 위치를 확인한 뒤, 빈 리스트가 들어오는 경우를 미리 검사해서 근본 원인을 제거했다.

**EX2) try/except로 사용자 입력 검증하기**
- 문자열을 숫자로 변환하는 과정에서 `ValueError`가 발생할 수 있는 상황을 `try/except`로 안전하게 처리하도록 구현한다.

```python
# ex02_safe_input.py
def parse_number(text):
    try:
        return int(text)
    except ValueError:
        print(f"'{text}'는 숫자로 변환할 수 없습니다. 0으로 처리합니다.")
        return 0

values = ["10", "abc", "25", "3.5"]
for v in values:
    print(parse_number(v))
```

```text
(base) C:\Users\guest\project> python ex02_safe_input.py
10
'abc'는 숫자로 변환할 수 없습니다. 0으로 처리합니다.
25
'3.5'는 숫자로 변환할 수 없습니다. 0으로 처리합니다.
```

**EX3) finally로 자원 정리 흉내내기**
- 파일을 여는 상황을 흉내낸 함수에서, 처리 중 오류가 발생해도 `finally`로 "자원 정리"가 항상 실행되는지 확인한다.

```python
# ex03_finally_cleanup.py
def process_data(data):
    print("자원 확보(파일 열기 흉내)")
    try:
        result = 100 / data
        print(f"처리 결과: {result}")
    except ZeroDivisionError:
        print("처리 중 오류 발생")
    finally:
        print("자원 정리(파일 닫기 흉내)")

process_data(5)
process_data(0)
```

```text
(base) C:\Users\guest\project> python ex03_finally_cleanup.py
자원 확보(파일 열기 흉내)
처리 결과: 20.0
자원 정리(파일 닫기 흉내)
자원 확보(파일 열기 흉내)
처리 중 오류 발생
자원 정리(파일 닫기 흉내)
```

**EX4) 여러 요청을 순회하며 실패해도 계속 진행하는 처리기**
- 여러 개의 나눗셈 요청을 리스트로 받아 순회하며 처리하되, 하나가 실패해도 전체 처리가 멈추지 않고 성공/실패 개수를 마지막에 요약 출력한다.

```python
# ex04_batch_processor.py
def batch_divide(requests):
    success = 0
    failure = 0
    for a, b in requests:
        try:
            print(f"{a} / {b} = {a / b}")
            success += 1
        except ZeroDivisionError:
            print(f"{a} / {b} 처리 실패: 0으로 나눔")
            failure += 1
    print(f"성공 {success}건, 실패 {failure}건")

batch_divide([(10, 2), (5, 0), (9, 3), (7, 0)])
```

```text
(base) C:\Users\guest\project> python ex04_batch_processor.py
10 / 2 = 5.0
5 / 0 처리 실패: 0으로 나눔
9 / 3 = 3.0
7 / 0 처리 실패: 0으로 나눔
성공 2건, 실패 2건
```

**정리**: EX1~EX4는 에러 메시지를 읽어 원인 지점을 찾아 근본적으로 고치는 방법, `try/except`로 사용자 입력을 안전하게 검증하는 방법, `finally`로 성공·실패와 무관하게 정리 작업을 보장하는 방법, 여러 요청을 처리하는 중 일부가 실패해도 전체 흐름이 멈추지 않도록 설계하는 방법까지 이 문서에서 다룬 에러 처리의 핵심 개념을 코드로 직접 확인해보는 예제다. 여기까지가 PY-00부터 PY-08까지 이어진 파이썬 기초 문서의 전체 흐름이며, 이후에는 각자 필요한 분야(웹, 데이터 분석, 자동화 등)에 맞는 라이브러리를 중심으로 학습을 이어가면 된다.

[Python 07 — 모듈과 라이브러리](07-modules-libraries.md) · [Python 09 — 제어 흐름 심화](09-control-flow-advanced.md)

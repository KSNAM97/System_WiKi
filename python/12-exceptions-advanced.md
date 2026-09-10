# Python 12 — 예외 처리 심화

## raise로 예외 직접 발생시키기

PY-08에서는 파이썬이 자동으로 발생시키는 예외를 `try`/`except`로 잡는 방법을 다뤘다. 이번에는 반대로, 특정 조건에서 프로그래머가 **직접** 예외를 발생시키는 `raise` 문을 살펴본다.

```python
def withdraw(balance, amount):
    if amount > balance:
        raise ValueError("잔액이 부족합니다.")
    return balance - amount

print(withdraw(10000, 3000))
print(withdraw(10000, 50000))
```

```text
(base) C:\Users\guest\project> python raise_basic.py
7000
Traceback (most recent call last):
  File "raise_basic.py", line 6, in <module>
    print(withdraw(10000, 50000))
  File "raise_basic.py", line 3, in withdraw
    raise ValueError("잔액이 부족합니다.")
ValueError: 잔액이 부족합니다.
```

- `raise 예외클래스("메시지")` 형태로 쓰면 그 자리에서 즉시 해당 예외가 발생하고, `except`로 잡지 않으면 프로그램이 트레이스백을 출력하며 종료된다.
- 어떤 예외 클래스를 골라야 할지는 상황에 맞춰 정해야 한다. 값이 잘못됐다면 `ValueError`, 타입이 잘못됐다면 `TypeError`, 존재해야 할 키나 인덱스가 없다면 `KeyError`/`IndexError`처럼 이미 있는 내장 예외 중 의미가 가장 가까운 것을 쓰는 것이 관례다.

**except 블록 안에서 raise만 단독으로 쓰기**

```python
def process(value):
    try:
        return 100 / value
    except ZeroDivisionError:
        print("0으로 나눌 수 없습니다. 다시 예외를 발생시킵니다.")
        raise

process(0)
```

```text
(base) C:\Users\guest\project> python raise_reraise.py
0으로 나눌 수 없습니다. 다시 예외를 발생시킵니다.
Traceback (most recent call last):
  File "raise_reraise.py", line 7, in <module>
    process(0)
  File "raise_reraise.py", line 3, in process
    return 100 / value
           ~~~~^~~~~~~
ZeroDivisionError: division by zero
```

- `except` 블록 안에서 인자 없이 `raise`만 쓰면, 방금 잡았던 예외를 **그대로 다시** 발생시킨다. 로그를 남기거나 뒷정리만 하고, 실제 처리는 호출한 쪽에 다시 맡기고 싶을 때 사용한다.

**정리**: `raise 예외클래스("메시지")`는 조건을 검사해 프로그래머가 원하는 시점에 직접 예외를 발생시키는 문법이며, 상황에 가장 의미가 맞는 내장 예외 클래스(`ValueError`, `TypeError` 등)를 고르는 것이 관례이고, `except` 블록 안에서 인자 없이 `raise`만 쓰면 방금 잡은 예외를 그대로 다시 던져 호출자에게 처리를 위임할 수 있다.

## 예외 연쇄 — raise ... from ...

예외를 처리하는 도중에 또 다른 예외가 발생하거나, 낮은 수준의 예외를 잡아서 더 의미 있는 예외로 바꿔 다시 던지고 싶은 경우가 있다. 이때 `raise ... from ...` 문법을 사용하면 두 예외 사이의 인과관계를 명시적으로 남길 수 있다.

```python
def load_config(text):
    try:
        return int(text)
    except ValueError as e:
        raise RuntimeError("설정값을 숫자로 변환할 수 없습니다.") from e

load_config("abc")
```

```text
(base) C:\Users\guest\project> python raise_from.py
Traceback (most recent call last):
  File "raise_from.py", line 3, in load_config
    return int(text)
           ^^^^^^^^^^
ValueError: invalid literal for int() with base 10: 'abc'

The above exception was the direct cause of the following exception:

Traceback (most recent call last):
  File "raise_from.py", line 7, in <module>
    load_config("abc")
  File "raise_from.py", line 5, in load_config
    raise RuntimeError("설정값을 숫자로 변환할 수 없습니다.") from e
RuntimeError: 설정값을 숫자로 변환할 수 없습니다.
```

- `raise 새예외 from 원인예외`로 쓰면, 새로 발생시키는 예외의 `__cause__` 속성에 원인 예외가 저장되고, 트레이스백에도 `The above exception was the direct cause of the following exception:`라는 문구와 함께 두 예외가 모두 출력되어 문제의 근본 원인을 추적하기 쉬워진다.
- 사실 `except` 블록 안에서 `from` 없이 새 예외를 `raise`하기만 해도, 파이썬은 자동으로 원래 예외를 새 예외의 `__context__` 속성에 저장하고 트레이스백에 `During handling of the above exception, another exception occurred:`라는 문구를 함께 보여준다. `__cause__`는 "내가 의도적으로 이 예외 때문에 새 예외를 던졌다"는 명시적 표시이고, `__context__`는 "예외를 처리하는 도중 우연히 또 다른 예외가 발생했다"는 암묵적 기록이라는 차이가 있다.
- 원인 예외 정보를 트레이스백에서 완전히 숨기고 싶다면 `raise 새예외 from None`을 사용하면 된다.

```python
def parse(text):
    try:
        return int(text)
    except ValueError:
        raise RuntimeError("파싱 실패") from None

parse("xyz")
```

```text
(base) C:\Users\guest\project> python raise_from_none.py
Traceback (most recent call last):
  File "raise_from_none.py", line 7, in <module>
    parse("xyz")
  File "raise_from_none.py", line 5, in parse
    raise RuntimeError("파싱 실패") from None
RuntimeError: 파싱 실패
```

**정리**: `raise 새예외 from 원인예외`는 예외를 다른 예외로 바꿔 던지면서도 원래 원인을 `__cause__`에 남겨 트레이스백에서 두 예외의 인과관계를 그대로 보여주며, `from` 없이 `except` 블록 안에서 예외를 새로 던지면 원래 예외가 `__context__`에 암묵적으로 기록되고, 원인 정보를 아예 숨기고 싶을 때는 `from None`을 사용한다.

## 사용자 정의 예외

내장 예외만으로는 의미를 정확히 전달하기 어려운 경우, `Exception`을 상속해 프로젝트에 맞는 예외 클래스를 직접 만들 수 있다. PY-06에서 다룬 클래스 상속 문법을 그대로 활용한다.

```python
class InsufficientBalanceError(Exception):
    """잔액이 부족할 때 발생시키는 예외."""
    pass

class Account:
    def __init__(self, balance):
        self.balance = balance

    def withdraw(self, amount):
        if amount > self.balance:
            raise InsufficientBalanceError(f"잔액 {self.balance}원보다 큰 {amount}원을 출금할 수 없습니다.")
        self.balance -= amount
        return self.balance

acc = Account(5000)
try:
    acc.withdraw(20000)
except InsufficientBalanceError as e:
    print("출금 실패:", e)
```

```text
(base) C:\Users\guest\project> python custom_exception.py
출금 실패: 잔액 5000원보다 큰 20000원을 출금할 수 없습니다.
```

- 사용자 정의 예외는 보통 `pass`만 있는 빈 클래스로도 충분하다. 클래스 이름 자체가 "어떤 상황에서 발생하는 예외인지"를 설명해주고, `Exception`을 상속했기 때문에 메시지 저장·`str()` 변환 등 예외로서 필요한 기능은 그대로 물려받는다.
- 예외 이름 끝에 `Error`를 붙이는 것이 관례이며, 이는 내장 예외(`ValueError`, `TypeError` 등)와 일관된 명명 규칙을 따르기 위함이다.

**여러 커스텀 예외를 계층으로 구성하기**

한 프로그램 안에서 여러 종류의 커스텀 예외가 필요하다면, 공통 부모 예외 클래스를 하나 만들고 그 아래에 구체적인 예외들을 상속시켜 계층 구조로 구성하는 것이 일반적이다.

```python
class BankError(Exception):
    """은행 관련 예외의 공통 부모 클래스."""
    pass

class InsufficientBalanceError(BankError):
    pass

class InvalidAmountError(BankError):
    pass

class Account:
    def __init__(self, balance):
        self.balance = balance

    def withdraw(self, amount):
        if amount <= 0:
            raise InvalidAmountError("출금액은 0보다 커야 합니다.")
        if amount > self.balance:
            raise InsufficientBalanceError("잔액이 부족합니다.")
        self.balance -= amount
        return self.balance

acc = Account(5000)
for amount in [-1000, 20000, 3000]:
    try:
        result = acc.withdraw(amount)
        print(f"출금 성공, 남은 잔액: {result}")
    except BankError as e:
        print(f"{type(e).__name__}: {e}")
```

```text
(base) C:\Users\guest\project> python custom_exception_hierarchy.py
InvalidAmountError: 출금액은 0보다 커야 합니다.
InsufficientBalanceError: 잔액이 부족합니다.
출금 성공, 남은 잔액: 2000
```

- 공통 부모 클래스(`BankError`)를 만들어두면, 호출하는 쪽에서 세부 예외 종류를 일일이 나열하지 않고 `except BankError:` 한 줄로 그 계층에 속한 모든 예외를 한꺼번에 처리할 수 있다. 반대로 세부적으로 구분해서 처리하고 싶을 때는 `except InvalidAmountError:`처럼 구체적인 클래스를 그대로 사용하면 된다.

**정리**: `Exception`을 상속하면 프로젝트 도메인에 맞는 의미 있는 이름의 예외 클래스를 직접 만들 수 있고, 보통 이름 끝에 `Error`를 붙이는 관례를 따르며, 여러 종류의 커스텀 예외가 필요할 때는 공통 부모 예외 클래스를 하나 두고 그 아래 구체적인 예외들을 상속시키는 계층 구조로 구성하면 호출하는 쪽에서 상위 클래스만으로 한꺼번에 잡거나 하위 클래스로 세밀하게 구분해서 잡는 것을 모두 지원할 수 있다.

## 뒷정리 동작 — finally와 with

PY-08에서 `finally` 블록은 예외 발생 여부와 상관없이 항상 실행된다고 배웠다. PY-11에서는 파일을 `with open(...) as f:`로 열면 블록을 벗어날 때 자동으로 닫힌다고 배웠다. 이 둘은 사실 "정리(cleanup) 동작을 보장한다"는 같은 목적을 서로 다른 방식으로 구현한 것이다.

```python
def risky_divide(a, b):
    try:
        return a / b
    finally:
        print("나눗셈 시도가 끝났습니다.")

try:
    risky_divide(10, 0)
except ZeroDivisionError:
    print("0으로 나눌 수 없습니다.")
```

```text
(base) C:\Users\guest\project> python finally_recap.py
나눗셈 시도가 끝났습니다.
0으로 나눌 수 없습니다.
```

- `finally`는 예외가 나든 안 나든, `return`으로 함수를 빠져나가든 상관없이 반드시 실행되는 블록으로, 파일 닫기·네트워크 연결 종료 같은 자원 정리 코드를 넣기에 적합하다.
- `with` 문은 이 정리 과정을 매번 `try`/`finally`로 직접 작성하지 않아도 되도록 자동화한 구문이다. `with open(...) as f:`의 내부 동작은 실제로 "블록에 들어갈 때 파일을 열고, 블록을 어떤 방식으로 빠져나가든(정상 종료든 예외든) `finally`처럼 반드시 파일을 닫는다"는 것과 동일하다.

**with는 사실 컨텍스트 매니저다**

`with` 문이 사용하는 객체를 **컨텍스트 매니저(context manager)**라고 부르며, `__enter__`와 `__exit__`라는 두 개의 메서드를 가진 객체는 모두 `with`와 함께 쓸 수 있다. 파일 객체도 이 두 메서드를 구현하고 있기 때문에 `with open(...) as f:`가 가능한 것이다.

```python
class LoggingContext:
    def __enter__(self):
        print("작업 시작")
        return self

    def __exit__(self, exc_type, exc_value, traceback):
        if exc_type is not None:
            print(f"작업 중 예외 발생: {exc_type.__name__}")
        print("작업 종료(정리 완료)")
        return False  # 예외를 억제하지 않고 그대로 전파

with LoggingContext():
    print("작업 수행 중")
    print(1 / 1)

try:
    with LoggingContext():
        print(1 / 0)
except ZeroDivisionError:
    print("바깥에서 예외를 잡았습니다.")
```

```text
(base) C:\Users\guest\project> python context_manager.py
작업 시작
작업 수행 중
1.0
작업 종료(정리 완료)
작업 시작
작업 중 예외 발생: ZeroDivisionError
작업 종료(정리 완료)
바깥에서 예외를 잡았습니다.
```

- `__enter__`는 `with` 블록에 들어갈 때 호출되며, 반환값이 `as` 뒤의 변수에 저장된다.
- `__exit__`는 블록을 벗어날 때(정상이든 예외든) 항상 호출되며, 예외가 있었다면 그 정보(`exc_type`, `exc_value`, `traceback`)를 인자로 받는다. `False`(또는 `None`)를 반환하면 예외를 그대로 바깥으로 전파하고, `True`를 반환하면 예외를 여기서 억제하고 없었던 일처럼 넘어간다.
- 결국 파일의 `with open(...) as f:`도 내부적으로 `f.__enter__()`가 파일 객체 자신을 반환하고, 블록을 벗어날 때 `f.__exit__()`가 `f.close()`를 호출해주는 구조와 같다.

**정리**: `finally`는 함수나 `try` 블록 수준에서 "무슨 일이 있어도 이 코드는 실행된다"를 보장하는 저수준 도구이고, `with`는 `__enter__`/`__exit__`를 구현한 컨텍스트 매니저 객체를 이용해 자원의 준비와 정리를 한 쌍으로 묶어 자동화한 고수준 도구이며, PY-11에서 다룬 파일의 `with open(...) as f:`도 결국 `__exit__`가 `finally`처럼 항상 실행되면서 `f.close()`를 대신 호출해주는 컨텍스트 매니저의 한 예에 불과하다.

## 여러 예외를 함께 처리하기

**튜플로 여러 예외 타입을 한 번에 잡기**

서로 다른 여러 종류의 예외를 같은 방식으로 처리하고 싶다면, `except` 뒤에 예외 클래스들을 튜플로 묶어서 한 번에 지정할 수 있다.

```python
def safe_convert(value):
    try:
        return int(value) / len(value)
    except (TypeError, ValueError, ZeroDivisionError) as e:
        print(f"{type(e).__name__} 발생: 변환할 수 없거나 계산할 수 없는 값입니다.")
        return None

print(safe_convert("10"))
print(safe_convert("abc"))
print(safe_convert(None))
print(safe_convert(""))
```

```text
(base) C:\Users\guest\project> python except_tuple.py
5.0
ValueError 발생: 변환할 수 없거나 계산할 수 없는 값입니다.
TypeError 발생: 변환할 수 없거나 계산할 수 없는 값입니다.
ValueError 발생: 변환할 수 없거나 계산할 수 없는 값입니다.
```

- `int("10") / len("10")`은 `10 / 2`이므로 `5.0`이 된다. `"abc"`는 `int()` 변환에서 `ValueError`, `None`은 `len()`에 쓸 수 없어 `TypeError`, 빈 문자열 `""`은 `int("")`에서 `ValueError`가 발생한다.

- `except (TypeError, ValueError, ZeroDivisionError) as e:`처럼 소괄호로 묶으면, 그 안에 나열된 예외 타입 중 어떤 것이 발생하더라도 같은 블록에서 처리한다. 각기 다른 예외를 서로 다른 방식으로 처리해야 한다면 `except TypeError:`, `except ValueError:`처럼 따로 나눠 써야 한다.
- 반드시 소괄호로 묶어야 한다는 점에 주의한다. `except TypeError, ValueError:`처럼 쓰면 문법 오류이거나 의도와 다르게 동작한다.

**예외 그룹과 except\* (파이썬 3.11+)**

비동기 작업이나 여러 하위 작업을 동시에 수행하는 코드에서는, 한 번에 서로 관련 없는 여러 예외가 동시에 발생할 수 있다. 파이썬 3.11부터는 이런 상황을 표현하기 위한 `ExceptionGroup`과, 이를 잡기 위한 `except*` 문법이 추가되었다.

```python
def run_checks():
    errors = []
    try:
        raise ValueError("이름이 비어 있습니다.")
    except ValueError as e:
        errors.append(e)
    try:
        raise TypeError("나이는 정수여야 합니다.")
    except TypeError as e:
        errors.append(e)

    if errors:
        raise ExceptionGroup("입력 검증 실패", errors)

try:
    run_checks()
except* ValueError as eg:
    print("ValueError 그룹:", [str(e) for e in eg.exceptions])
except* TypeError as eg:
    print("TypeError 그룹:", [str(e) for e in eg.exceptions])
```

```text
(base) C:\Users\guest\project> python except_star.py
ValueError 그룹: ['이름이 비어 있습니다.']
TypeError 그룹: ['나이는 정수여야 합니다.']
```

- `ExceptionGroup("설명", [예외1, 예외2, ...])`은 서로 독립적인 여러 예외를 하나의 묶음으로 감싼 특수한 예외 객체다.
- `except* 예외타입 as eg:`는 일반 `except`와 달리, 예외 그룹 안에서 해당 타입과 일치하는 예외들만 따로 모아 `eg.exceptions`(튜플)에 담아 넘겨주며, 일치하지 않는 예외가 남아 있으면 다음 `except*` 절이 이어서 검사한다. 이 문법은 파이썬 3.11 이상에서만 사용할 수 있다.

**정리**: 여러 종류의 예외를 같은 방식으로 처리하려면 `except (타입1, 타입2):`처럼 튜플로 묶어서 한 번에 잡을 수 있고, 파이썬 3.11부터는 여러 개의 서로 독립적인 예외가 한꺼번에 발생하는 상황을 `ExceptionGroup`으로 표현하고 `except* 타입:` 문법으로 그룹 안에서 원하는 타입의 예외만 골라 처리할 수 있어, 단일 예외 하나만 가정하던 기존 `try`/`except` 모델로는 다루기 어려웠던 동시다발적 오류 상황을 표현할 수 있게 되었다.

## 예외에 노트 추가하기 — add_note()

파이썬 3.11부터 예외 객체에는 `add_note(문자열)` 메서드가 추가되었다. 이 메서드를 사용하면 예외를 다시 던지기 전에 추가 설명을 붙여, 트레이스백에 원래 메시지와 함께 표시할 수 있다.

```python
def process_record(record):
    try:
        return record["age"] * 2
    except KeyError as e:
        e.add_note(f"레코드 내용: {record}")
        e.add_note("age 키가 누락된 것으로 보입니다.")
        raise

process_record({"name": "김철수"})
```

```text
(base) C:\Users\guest\project> python add_note.py
Traceback (most recent call last):
  File "add_note.py", line 3, in process_record
    return record["age"] * 2
           ~~~~~~^^^^^^^
KeyError: 'age'
레코드 내용: {'name': '김철수'}
age 키가 누락된 것으로 보입니다.
```

- `add_note()`는 여러 번 호출할 수 있으며, 호출한 순서대로 트레이스백 맨 아래에 추가되어 출력된다.
- 예외를 잡아서 로그를 남기거나 재발생시키기 전에 "이 예외가 어떤 맥락에서 발생했는지"에 대한 부가 정보를 덧붙이고 싶을 때 유용하며, 예외 메시지 자체를 새로 만들지 않고도 디버깅에 필요한 문맥을 그대로 보존할 수 있다는 장점이 있다.

**정리**: `add_note()`는 파이썬 3.11부터 예외 객체에 추가된 메서드로, 예외를 다시 던지기 전에 부가 설명을 덧붙여 트레이스백에 원래 메시지와 함께 출력해주며, 예외가 발생한 시점의 맥락 정보(어떤 데이터를 처리하다 실패했는지 등)를 예외 자체에 남겨두고 싶을 때 유용하다.

## 실습 예제 (EX1~EX4)

**EX1) 나이 입력값 검증 함수 만들기**
- 나이를 입력받아 0 이상 150 이하가 아니면 `ValueError`를 직접 발생시키는 함수를 작성한다.

```python
# ex01_validate_age.py
def validate_age(age):
    if not (0 <= age <= 150):
        raise ValueError(f"나이는 0~150 사이여야 합니다: {age}")
    return age

for value in [25, -5, 200]:
    try:
        print(f"{value} -> 유효한 나이: {validate_age(value)}")
    except ValueError as e:
        print(f"{value} -> 오류: {e}")
```

```text
(base) C:\Users\guest\project> python ex01_validate_age.py
25 -> 유효한 나이: 25
-5 -> 오류: 나이는 0~150 사이여야 합니다: -5
200 -> 오류: 나이는 0~150 사이여야 합니다: 200
```

**EX2) 커스텀 예외로 재고 관리하기**
- 재고가 부족할 때 발생시킬 `OutOfStockError`를 정의하고, 주문 처리 함수에서 이를 활용해 구현한다.

```python
# ex02_out_of_stock.py
class OutOfStockError(Exception):
    pass

def order(stock, item, quantity):
    if stock.get(item, 0) < quantity:
        raise OutOfStockError(f"{item} 재고가 부족합니다. (재고: {stock.get(item, 0)}, 요청: {quantity})")
    stock[item] -= quantity
    return stock[item]

stock = {"노트북": 3, "마우스": 10}

for item, quantity in [("마우스", 5), ("노트북", 10)]:
    try:
        remaining = order(stock, item, quantity)
        print(f"{item} 주문 완료, 남은 재고: {remaining}")
    except OutOfStockError as e:
        print("주문 실패:", e)
```

```text
(base) C:\Users\guest\project> python ex02_out_of_stock.py
마우스 주문 완료, 남은 재고: 5
주문 실패: 노트북 재고가 부족합니다. (재고: 3, 요청: 10)
```

**EX3) raise ... from ...으로 예외 원인 남기기**
- 파일을 읽다가 실패했을 때, 원래 예외를 원인으로 남기면서 더 명확한 메시지의 예외를 다시 던지시오.

```python
# ex03_raise_from_file.py
def load_settings(path):
    try:
        with open(path, "r", encoding="utf-8") as f:
            return f.read()
    except FileNotFoundError as e:
        raise RuntimeError(f"설정 파일 '{path}'을(를) 찾을 수 없습니다.") from e

try:
    load_settings("no_such_settings.txt")
except RuntimeError as e:
    print("설정 로딩 실패:", e)
    print("원인:", repr(e.__cause__))
```

```text
(base) C:\Users\guest\project> python ex03_raise_from_file.py
설정 로딩 실패: 설정 파일 'no_such_settings.txt'을(를) 찾을 수 없습니다.
원인: FileNotFoundError(2, 'No such file or directory')
```

**EX4) 여러 필드를 검증하고 예외 그룹으로 한꺼번에 보고하기**
- 여러 개의 입력 필드를 검증해 문제가 있는 필드들을 `ExceptionGroup`으로 묶어 한 번에 정리한다.

```python
# ex04_exception_group.py
def validate_user(data):
    errors = []
    if "name" not in data or not data["name"]:
        errors.append(ValueError("name이 비어 있습니다."))
    if "age" in data and not isinstance(data["age"], int):
        errors.append(TypeError("age는 정수여야 합니다."))
    if errors:
        raise ExceptionGroup("사용자 검증 실패", errors)
    return True

try:
    validate_user({"name": "", "age": "스물다섯"})
except* ValueError as eg:
    for e in eg.exceptions:
        print("ValueError:", e)
except* TypeError as eg:
    for e in eg.exceptions:
        print("TypeError:", e)
```

```text
(base) C:\Users\guest\project> python ex04_exception_group.py
ValueError: name이 비어 있습니다.
TypeError: age는 정수여야 합니다.
```

**정리**: EX1~EX4는 `raise`로 입력값을 직접 검증하는 방법, 커스텀 예외 클래스로 도메인에 맞는 오류 상황을 표현하는 방법, `raise ... from ...`으로 예외의 원인을 `__cause__`에 남겨 디버깅 정보를 보존하는 방법, 그리고 파이썬 3.11의 `ExceptionGroup`과 `except*`로 여러 개의 독립적인 오류를 한 번에 검증하고 보고하는 방법까지, 이 문서에서 다룬 예외 처리 심화 개념을 직접 코드로 확인해보는 예제다.

[Python 11 — 입력과 출력](11-io-files.md) · [Python 13 — 클래스 심화](13-classes-advanced.md)

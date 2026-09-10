# Python 03 — 함수

## def 문법과 함수 호출

**함수(function)**는 여러 줄의 코드를 하나의 이름으로 묶어, 필요할 때마다 반복해서 호출할 수 있게 만든 코드 단위다. 파이썬에서는 `def` 키워드로 함수를 정의한다.

```python
def greet():
    print("안녕하세요!")

greet()
greet()
```

```text
(base) C:\Users\guest\project> python greet.py
안녕하세요!
안녕하세요!
```

- `def 함수이름():` 뒤에 콜론(`:`)을 붙이고, 그 아래 들여쓰기된 줄들이 함수의 본문이다.
- 함수는 정의만으로는 실행되지 않으며, `greet()`처럼 이름 뒤에 괄호를 붙여 **호출(call)**해야 본문이 실행된다. 이 순서 관계는 PY-01 문서의 순차 실행에서 이미 다룬 내용과 같다.

**함수를 쓰는 이유**

```python
# 함수 없이 중복 작성
print("=" * 20)
print("메뉴 안내")
print("=" * 20)

print("=" * 20)
print("주문 확인")
print("=" * 20)
```

```python
# 함수로 중복 제거
def print_banner(title):
    print("=" * 20)
    print(title)
    print("=" * 20)

print_banner("메뉴 안내")
print_banner("주문 확인")
```

```text
(base) C:\Users\guest\project> python banner.py
====================
메뉴 안내
====================
====================
주문 확인
====================
```

- 반복되는 패턴을 함수로 묶으면 코드 중복이 줄고, 나중에 출력 형식을 바꿀 때도 함수 본문 한 곳만 수정하면 된다는 이점이 있다.

**정리**: 함수는 `def 함수이름(매개변수):` 형태로 정의하며 본문은 들여쓰기로 구분되고, 정의된 함수는 이름 뒤에 괄호를 붙여 호출해야 비로소 실행되며, 반복되는 코드를 함수로 묶으면 중복을 줄이고 유지보수를 쉽게 만든다.

## 매개변수와 타입 힌트

함수를 호출할 때 함수 내부로 값을 전달하고 싶다면 **매개변수(parameter)**를 사용한다.

```python
def greet(name):
    print(f"{name}님, 안녕하세요.")

greet("홍길동")
greet("김철수")
```

```text
(base) C:\Users\guest\project> python greet_param.py
홍길동님, 안녕하세요.
김철수님, 안녕하세요.
```

- `greet(name)`에서 `name`이 매개변수이고, `greet("홍길동")`처럼 호출할 때 넘기는 `"홍길동"`을 **인자(argument)**라고 부른다.
- 매개변수는 쉼표로 나열해 여러 개 받을 수 있으며, 호출할 때 넘긴 인자와 순서대로 짝지어진다.

```python
def introduce(name, age, city):
    print(f"이름: {name}, 나이: {age}, 거주지: {city}")

introduce("홍길동", 20, "서울")
```

```text
(base) C:\Users\guest\project> python introduce.py
이름: 홍길동, 나이: 20, 거주지: 서울
```

**키워드 인자로 순서 상관없이 전달하기**

```python
introduce(city="부산", name="김철수", age=25)
```

```text
부산-김철수-25 순서로 넘겼지만 매개변수 이름과 매칭되어 정상 동작
이름: 김철수, 나이: 25, 거주지: 부산
```

- `매개변수이름=값` 형태로 넘기면 **키워드 인자(keyword argument)**가 되어, 정의된 매개변수 순서와 무관하게 이름으로 정확히 매칭된다.

**타입 힌트(type hint)**

파이썬은 변수·매개변수의 타입을 강제하지 않지만, 코드를 읽는 사람(과 편집기)에게 "이 매개변수에는 이런 타입의 값이 들어올 것이다"라는 정보를 남기기 위해 **타입 힌트** 문법을 제공한다.

```python
def introduce(name: str, age: int, city: str) -> None:
    print(f"이름: {name}, 나이: {age}, 거주지: {city}")
```

- `name: str`처럼 매개변수 뒤에 `: 타입`을 붙이고, `-> None`처럼 함수 괄호 뒤에 `-> 반환타입`을 붙인다.
- 타입 힌트는 어디까지나 "힌트"일 뿐이므로, 실제로 다른 타입의 값을 넘겨도 파이썬 인터프리터는 오류를 내지 않는다. 다만 VS Code 같은 편집기는 힌트와 다른 타입이 넘어오면 경고를 표시해준다.

```python
>>> def add(a: int, b: int) -> int:
...     return a + b
...
>>> print(add(1, 2))
3
>>> print(add("1", "2"))    # 타입 힌트를 어겼지만 오류 없이 실행됨
12
```

- 두 번째 호출처럼 문자열을 넘겨도 파이썬은 그대로 실행하며 문자열 `+`(이어붙이기)로 동작한다. 타입 힌트는 런타임에 강제되는 검사가 아니라 사람과 도구를 위한 문서화 장치라는 점을 기억해야 한다.

**정리**: 함수는 매개변수로 외부 값을 전달받으며, 호출 시 위치 순서대로 넘기는 위치 인자와 `이름=값` 형태로 순서 상관없이 넘기는 키워드 인자를 모두 지원한다. 타입 힌트(`매개변수: 타입`, `-> 반환타입`)는 코드의 의도를 명확히 드러내는 문서화 도구이지만 런타임에 타입을 강제로 검사하지는 않는다는 점이 다른 정적 타입 언어와 다르다.

## 기본값 매개변수

매개변수에 **기본값(default value)**을 지정해두면, 호출할 때 해당 인자를 생략해도 기본값이 대신 사용된다.

```python
def greet(name, greeting="안녕하세요"):
    print(f"{greeting}, {name}님.")

greet("홍길동")
greet("김철수", "반갑습니다")
```

```text
(base) C:\Users\guest\project> python default_param.py
안녕하세요, 홍길동님.
반갑습니다, 김철수님.
```

- 첫 번째 호출은 `greeting`을 생략했으므로 기본값 `"안녕하세요"`가 사용된다.
- 두 번째 호출처럼 값을 넘기면 기본값 대신 넘긴 값으로 대체된다.

**기본값이 없는 매개변수는 기본값 있는 매개변수보다 앞에 와야 한다**

```python
def greet(greeting="안녕하세요", name):
    print(f"{greeting}, {name}님.")
```

```text
  File "default_order.py", line 1
    def greet(greeting="안녕하세요", name):
SyntaxError: non-default argument follows default argument
```

- 기본값이 없는 매개변수가 기본값 있는 매개변수 뒤에 오면 `SyntaxError`가 발생한다. 항상 "기본값 없는 매개변수 → 기본값 있는 매개변수" 순서로 정의해야 한다.

**가변 기본값(리스트·딕셔너리)의 함정**

```python
def add_item(item, cart=[]):
    cart.append(item)
    return cart

print(add_item("사과"))
print(add_item("바나나"))
```

```text
(base) C:\Users\guest\project> python mutable_default.py
['사과']
['사과', '바나나']
```

- 기본값으로 지정한 리스트 `[]`는 함수가 정의되는 시점에 딱 한 번만 만들어지고, 이후 호출마다 공유되어 재사용된다. 그래서 `add_item("바나나")`를 호출했을 때 새 빈 리스트가 아니라 이전 호출에서 누적된 리스트가 그대로 이어진다.
- 이런 의도치 않은 동작을 피하려면 기본값을 `None`으로 두고, 함수 내부에서 실제 빈 리스트를 새로 만드는 방식을 사용한다.

```python
def add_item(item, cart=None):
    if cart is None:
        cart = []
    cart.append(item)
    return cart

print(add_item("사과"))
print(add_item("바나나"))
```

```text
(base) C:\Users\guest\project> python safe_default.py
['사과']
['바나나']
```

**정리**: 기본값 매개변수는 `매개변수=기본값` 형태로 지정하며 호출 시 생략하면 기본값이 사용되고, 기본값 없는 매개변수는 반드시 기본값 있는 매개변수보다 먼저 정의해야 한다. 리스트·딕셔너리처럼 가변 자료형을 기본값으로 직접 지정하면 호출 간에 값이 공유되는 함정이 있으므로, 기본값을 `None`으로 두고 함수 내부에서 새로 생성하는 방식이 안전하다.

## 반환값

함수 내부에서 계산한 결과를 호출한 쪽으로 돌려주려면 **`return`** 문을 사용한다.

```python
def add(a, b):
    result = a + b
    return result

total = add(3, 5)
print(total)
```

```text
(base) C:\Users\guest\project> python return_basic.py
8
```

- `return`을 만나면 함수는 그 즉시 실행을 멈추고 지정한 값을 호출한 쪽으로 돌려준다. `return` 뒤에 남은 코드가 있어도 실행되지 않는다.

```python
def check_positive(n):
    if n > 0:
        return "양수"
    return "0 이하"
    print("이 줄은 절대 실행되지 않는다")

print(check_positive(5))
print(check_positive(-2))
```

```text
(base) C:\Users\guest\project> python return_stop.py
양수
0 이하
```

**여러 값을 한 번에 반환하기**

```python
def get_min_max(numbers):
    return min(numbers), max(numbers)

low, high = get_min_max([4, 1, 9, 3])
print(low, high)
```

```text
(base) C:\Users\guest\project> python return_multi.py
1 9
```

- `return`에 쉼표로 값을 나열하면 내부적으로 튜플(tuple)로 묶여 반환되며, 호출한 쪽에서 `low, high = ...`처럼 여러 변수로 나누어 받을 수 있다.

**return이 없는 함수는 None을 반환한다**

```python
def say_hello():
    print("안녕하세요")

result = say_hello()
print(result)
```

```text
(base) C:\Users\guest\project> python return_none.py
안녕하세요
None
```

- `return` 문이 아예 없거나 값 없이 `return`만 적으면, 파이썬은 자동으로 `None`을 반환한다. `print()`가 화면에 값을 "보여주는" 것과 함수가 값을 "반환하는" 것은 서로 다른 동작이라는 점에 주의해야 한다.

**정리**: `return`은 함수의 실행을 즉시 종료하면서 지정한 값을 호출한 쪽으로 돌려주며, 쉼표로 여러 값을 나열해 튜플로 한 번에 반환하고 여러 변수로 나누어 받을 수도 있다. `return`이 없거나 값 없이 쓰인 함수는 항상 `None`을 반환하므로, 화면에 출력만 하고 값을 반환하지 않는 함수의 결과를 변수에 저장해 사용하려 하면 의도치 않게 `None`을 다루게 되는 실수를 조심해야 한다.

## 지역 변수와 전역 변수

함수 내부에서 선언한 변수는 그 함수 안에서만 사용할 수 있는 **지역 변수(local variable)**이며, 함수 바깥에서 선언한 변수는 프로그램 전체에서 접근 가능한 **전역 변수(global variable)**다.

```python
def calculate():
    result = 100   # 지역 변수
    print(result)

calculate()
print(result)
```

```text
(base) C:\Users\guest\project> python scope_error.py
100
Traceback (most recent call last):
  File "scope_error.py", line 6, in <module>
    print(result)
NameError: name 'result' is not defined
```

- `calculate()` 함수 안에서 정의한 `result`는 함수가 끝나는 순간 사라지므로, 함수 바깥에서 같은 이름으로 접근하면 `NameError`가 발생한다.

**함수는 전역 변수를 읽을 수 있다**

```python
count = 10

def show_count():
    print(count)

show_count()
```

```text
(base) C:\Users\guest\project> python read_global.py
10
```

- 함수 내부에서 값을 읽기만 하는 경우, 같은 이름의 지역 변수가 없다면 자동으로 전역 변수를 참조한다.

**함수 내부에서 전역 변수에 값을 대입하려면 global 키워드가 필요하다**

```python
count = 10

def increase():
    count += 1   # 오류 발생
    print(count)

increase()
```

```text
(base) C:\Users\guest\project> python global_error.py
Traceback (most recent call last):
  File "global_error.py", line 4, in <module>
    increase()
  File "global_error.py", line 4, in increase
    count += 1
UnboundLocalError: cannot access local variable 'count' where it is not defined
```

- 함수 내부에서 특정 변수에 값을 대입하는 코드가 하나라도 있으면, 파이썬은 그 변수를 함수 전체에서 지역 변수로 취급한다. `count += 1`은 `count`에 값을 대입하는 코드이므로, `count`는 지역 변수로 간주되고 아직 선언되지 않은 상태에서 읽으려 했기 때문에 오류가 발생한다.
- 함수 내부에서 전역 변수의 값을 실제로 바꾸고 싶다면 `global` 키워드로 그 변수가 전역 변수임을 명시해야 한다.

```python
count = 10

def increase():
    global count
    count += 1
    print(count)

increase()
print(count)
```

```text
(base) C:\Users\guest\project> python global_fix.py
11
11
```

**정리**: 함수 안에서 만든 변수는 그 함수 안에서만 존재하는 지역 변수이며 함수가 끝나면 사라지고, 함수는 같은 이름의 지역 변수가 없다면 전역 변수를 자유롭게 읽을 수 있지만 값을 대입하려면 `global` 키워드로 전역 변수임을 명시해야 한다. 실무에서는 함수가 전역 상태를 직접 수정하면 코드 흐름을 추적하기 어려워지므로, 값을 전달하고 `return`으로 결과를 돌려받는 방식을 우선적으로 고려하는 것이 바람직하다.

## 변수 가림 (shadowing)

지역 변수가 전역 변수와 같은 이름을 가지면, 함수 내부에서는 지역 변수가 전역 변수를 "가려서" 전역 변수에 접근할 수 없게 된다. 이를 **변수 가림(shadowing)**이라고 한다.

```python
name = "전역이름"

def show_name():
    name = "지역이름"
    print(name)

show_name()
print(name)
```

```text
(base) C:\Users\guest\project> python shadowing.py
지역이름
전역이름
```

- `show_name()` 함수 안의 `print(name)`은 같은 이름의 지역 변수 `name`을 먼저 찾아 사용하므로 `"지역이름"`이 출력된다.
- 함수가 끝나면 지역 변수 `name`은 사라지고, 바깥의 `print(name)`은 처음부터 존재했던 전역 변수 `"전역이름"`을 그대로 가리킨다.
- 이렇게 이름이 겹치더라도 전역 변수의 값 자체는 함수 안의 지역 변수에 의해 바뀌지 않는다. 함수 안에서 전역 변수를 가리는 지역 변수는 오직 그 함수 범위 안에서만 유효한 별개의 저장 공간이다.

**매개변수도 같은 방식으로 가림을 일으킨다**

```python
message = "기본 메시지"

def show(message):
    print(message)

show("전달된 메시지")
print(message)
```

```text
(base) C:\Users\guest\project> python param_shadowing.py
전달된 메시지
기본 메시지
```

- 매개변수 `message`는 전역 변수 `message`와 이름이 같지만 완전히 별개의 지역 변수이므로, 함수 안에서는 전달받은 인자 값이 사용되고 전역 변수는 영향을 받지 않는다.

**정리**: 변수 가림은 지역 변수(매개변수 포함)가 전역 변수와 같은 이름을 쓸 때, 함수 내부에서는 지역 변수가 우선적으로 참조되어 전역 변수가 가려지는 현상이며, 함수가 끝나면 지역 변수는 사라지고 전역 변수는 원래 값 그대로 남아 있다. 의도치 않은 가림은 코드를 읽을 때 혼란을 줄 수 있으므로, 전역 변수와 지역 변수의 이름이 겹치지 않도록 의식적으로 구분해서 짓는 습관이 바람직하다.

## 중첩 함수

함수 내부에서 또 다른 함수를 정의할 수 있으며, 이를 **중첩 함수(nested function)**라고 한다.

```python
def outer():
    print("바깥 함수 시작")

    def inner():
        print("안쪽 함수 실행")

    inner()
    print("바깥 함수 종료")

outer()
```

```text
(base) C:\Users\guest\project> python nested_func.py
바깥 함수 시작
안쪽 함수 실행
바깥 함수 종료
```

- `inner()`는 `outer()` 함수 내부에서만 정의되고 호출될 수 있으며, `outer()` 바깥에서 `inner()`를 직접 호출하면 `NameError`가 발생한다.

```python
>>> inner()
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
NameError: name 'inner' is not defined
```

**중첩 함수는 바깥 함수의 지역 변수를 읽을 수 있다**

```python
def make_greeting(name):
    greeting = f"{name}님, 환영합니다"

    def show():
        print(greeting)   # 바깥 함수의 지역 변수를 읽음

    show()

make_greeting("홍길동")
```

```text
(base) C:\Users\guest\project> python nested_closure.py
홍길동님, 환영합니다
```

- 안쪽 함수 `show()`는 바깥 함수 `make_greeting()`의 지역 변수 `greeting`을 자신의 지역 변수처럼 읽어 사용할 수 있다. 이런 성질은 바깥 함수의 상태를 기억하는 함수를 만드는 **클로저(closure)** 개념으로 이어지지만, 본격적인 활용은 이후 심화 문서에서 다룬다.
- 중첩 함수는 특정 함수 안에서만 쓰이는 보조 로직을 캡슐화하거나, 반복적으로 등장하는 짧은 계산을 분리할 때 사용한다.

**정리**: 함수 내부에 정의된 중첩 함수는 그 바깥 함수 범위 안에서만 호출할 수 있으며, 바깥 함수의 지역 변수를 자유롭게 읽을 수 있어 하나의 함수 안에서만 필요한 보조 로직을 캡슐화하는 데 유용하다.

## 상호 재귀 시 주의점

함수는 자기 자신을 호출하는 **재귀(recursion)**뿐 아니라, 두 함수가 서로를 번갈아 호출하는 **상호 재귀(mutual recursion)**도 가능하다.

```python
def is_even(n):
    if n == 0:
        return True
    return is_odd(n - 1)

def is_odd(n):
    if n == 0:
        return False
    return is_even(n - 1)

print(is_even(10))
print(is_odd(7))
```

```text
(base) C:\Users\guest\project> python mutual_recursion.py
True
True
```

- `is_even()`이 `is_odd()`를 호출하고, `is_odd()`는 다시 `is_even()`을 호출하며 `n`을 하나씩 줄여나가다가 `n == 0`에 도달하면 재귀가 멈춘다.

**정의 순서에 주의해야 한다**

파이썬은 함수를 호출하는 시점에 그 이름이 존재하기만 하면 되므로, `is_even()`이 아직 정의되지 않은 `is_odd()`를 함수 본문 안에서 참조하는 것 자체는 문제가 없다. 다만 실제로 **호출되는 시점**에는 두 함수 모두 이미 정의되어 있어야 한다.

```python
def is_even(n):
    if n == 0:
        return True
    return is_odd(n - 1)

is_even(4)   # 이 시점에는 아직 is_odd가 정의되지 않아 오류 발생

def is_odd(n):
    if n == 0:
        return False
    return is_even(n - 1)
```

```text
(base) C:\Users\guest\project> python mutual_recursion_error.py
Traceback (most recent call last):
  File "mutual_recursion_error.py", line 5, in <module>
    is_even(4)
  File "mutual_recursion_error.py", line 3, in is_even
    return is_odd(n - 1)
NameError: name 'is_odd' is not defined
```

- `is_even()` 함수 본문 안에 `is_odd(n - 1)`이라는 코드가 있는 것 자체는 정의 시점에는 문제가 되지 않는다. 함수 본문은 정의될 때가 아니라 호출될 때 실행되기 때문이다.
- 그러나 실제로 `is_even(4)`를 호출하는 시점(파일의 5번째 줄)에는 아직 `is_odd`가 정의되지 않은 상태이므로 `NameError`가 발생한다. 상호 재귀 함수를 사용할 때는 두 함수 모두를 먼저 정의한 뒤에 호출해야 한다.

**종료 조건이 없으면 무한 재귀에 빠진다**

```python
def a(n):
    return b(n)

def b(n):
    return a(n)

a(1)
```

```text
(base) C:\Users\guest\project> python infinite_recursion.py
Traceback (most recent call last):
  ...
RecursionError: maximum recursion depth exceeded
```

- 재귀든 상호 재귀든, `n == 0`과 같이 재귀를 멈추게 하는 **종료 조건(base case)**이 반드시 있어야 한다. 종료 조건 없이 서로를 무한히 호출하면 파이썬이 정해둔 최대 호출 깊이를 넘어서 `RecursionError`가 발생한다.

**정리**: 상호 재귀는 두 함수가 서로를 번갈아 호출하며 문제를 해결하는 방식으로, 함수 본문 안에서 아직 정의되지 않은 다른 함수를 참조하는 것 자체는 괜찮지만 실제 호출 시점에는 양쪽 함수가 모두 정의되어 있어야 하며, 재귀가 끝나는 명확한 종료 조건이 없으면 `RecursionError`로 이어지는 무한 호출에 빠지게 된다.

## 실습 예제 (EX1~EX4)

**EX1) 최댓값을 구하는 함수 (기본값 매개변수 활용)**
- 숫자 리스트를 받아 최댓값을 반환하는 함수를 작성하되, 빈 리스트가 들어오면 기본값으로 지정한 값을 반환하도록 하시오.

```python
# ex01_safe_max.py
def safe_max(numbers, default=None):
    if not numbers:
        return default
    return max(numbers)

print(safe_max([3, 7, 2]))
print(safe_max([], default=0))
```

```text
(base) C:\Users\guest\project> python ex01_safe_max.py
7
0
```

**EX2) 전역 카운터를 증가시키는 함수**
- 함수를 호출할 때마다 전역 변수 `visit_count`를 1씩 증가시키고, 현재까지의 누적 방문 횟수를 출력하는 함수를 작성하시오. `global` 키워드를 사용해야 한다.

```python
# ex02_visit_counter.py
visit_count = 0

def visit():
    global visit_count
    visit_count += 1
    print(f"현재까지 방문 횟수: {visit_count}")

visit()
visit()
visit()
```

```text
(base) C:\Users\guest\project> python ex02_visit_counter.py
현재까지 방문 횟수: 1
현재까지 방문 횟수: 2
현재까지 방문 횟수: 3
```

**EX3) 최댓값·최솟값·평균을 함께 반환하는 함수**
- 숫자 리스트를 받아 최댓값, 최솟값, 평균을 튜플로 한 번에 반환하는 함수를 작성하고 세 변수로 나누어 받으시오.

```python
# ex03_stats.py
def get_stats(numbers):
    highest = max(numbers)
    lowest = min(numbers)
    average = sum(numbers) / len(numbers)
    return highest, lowest, average

scores = [88, 95, 70, 100, 60]
highest, lowest, average = get_stats(scores)
print(f"최고점: {highest}, 최저점: {lowest}, 평균: {average}")
```

```text
(base) C:\Users\guest\project> python ex03_stats.py
최고점: 100, 최저점: 60, 평균: 82.6
```

**EX4) 계승(factorial)을 재귀 함수로 구현**
- `n!`(n의 계승)을 재귀 함수로 구현하시오. 종료 조건은 `n == 0`일 때 `1`을 반환하는 것이다.

```python
# ex04_factorial.py
def factorial(n):
    if n == 0:
        return 1
    return n * factorial(n - 1)

print(factorial(5))
print(factorial(0))
```

```text
(base) C:\Users\guest\project> python ex04_factorial.py
120
1
```

- `factorial(5)`는 `5 * factorial(4)`, `factorial(4)`는 `4 * factorial(3)`처럼 자기 자신을 계속 호출하다가 `factorial(0)`에서 `1`을 반환하며 재귀가 끝나고, 이후 곱셈 결과가 차례로 되돌아오며 최종값 `120`이 계산된다.

**정리**: EX1~EX4는 기본값 매개변수로 빈 입력을 안전하게 처리하는 방법, `global` 키워드로 전역 상태를 관리하는 방법, 여러 값을 튜플로 한 번에 반환하는 방법, 종료 조건을 갖춘 재귀 함수 작성법까지 이 문서에서 다룬 함수의 핵심 개념을 코드로 직접 확인해보는 예제이며, 다음 문서에서는 여러 값을 하나로 묶어 관리하는 리스트와 딕셔너리를 다룬다.

[Python 02 — 변수와 자료형](02-variables.md) · [Python 04 — 리스트와 딕셔너리](04-lists-dictionaries.md)

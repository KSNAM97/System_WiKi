# Python 17 — 언어 심화

PY-09에서는 `*args`, `**kwargs`, 람다, 함수 애너테이션 같은 함수와 관련된 제어 흐름 문법을 다뤘고, PY-16에서는 `logging`, `threading`, `urllib`, `unittest` 같은 조금 더 규모 있는 프로그램에 필요한 표준 라이브러리를 살펴봤다. 이 문서에서는 파이썬 코드를 더 간결하고 안전하게 만들어주는 언어 차원의 도구들, 즉 함수를 감싸는 데코레이터, 클래스를 간결하게 정의하는 dataclasses, 타입 힌트를 한 단계 더 깊게 활용하는 방법, 자주 쓰이는 `functools`/`itertools` 도구, 경로를 객체로 다루는 `pathlib`, 그리고 커맨드라인 인자를 체계적으로 처리하는 `argparse`를 다룬다.

## 데코레이터

**함수를 감싸는 함수 호출의 축약형**

데코레이터는 다른 함수를 인자로 받아 그 동작을 감싸는 함수를 반환하는 함수다. `@` 문법은 사실 새로운 개념이 아니라, 함수를 정의한 뒤 그 결과를 다른 함수로 감싸는 익숙한 호출을 짧게 줄여 쓰는 표기법이다.

```python
def deco(func):
    def wrapper():
        print("함수 실행 전")
        func()
        print("함수 실행 후")
    return wrapper

def say_hello():
    print("안녕하세요")

say_hello = deco(say_hello)
say_hello()
```

```text
(base) C:\Users\guest\project> python deco_manual.py
함수 실행 전
안녕하세요
함수 실행 후
```

```python
def deco(func):
    def wrapper():
        print("함수 실행 전")
        func()
        print("함수 실행 후")
    return wrapper

@deco
def say_hello():
    print("안녕하세요")

say_hello()
```

```text
(base) C:\Users\guest\project> python deco_syntax.py
함수 실행 전
안녕하세요
함수 실행 후
```

- `say_hello = deco(say_hello)`는 `say_hello`라는 이름에, 원래 함수를 `deco()`로 감싼 결과인 `wrapper` 함수를 다시 대입한다. 이후 `say_hello()`를 호출하면 실제로는 `wrapper()`가 실행되고, 그 안에서 원래의 `say_hello`(코드에서는 `func`라는 이름으로 넘어옴)가 호출된다.
- `@deco`를 함수 정의 바로 위에 적으면, 파이썬이 `say_hello = deco(say_hello)`를 자동으로 수행해준다. 두 코드는 완전히 동일하게 동작하며, `@` 문법은 이 대입 과정을 눈에 띄지 않게 숨기는 대신 "이 함수는 `deco`로 감싸져 있다"는 사실을 함수 정의 바로 위에서 명시적으로 보여주는 역할을 한다.

**클로저 기반 wrapper와 임의 인자 전달**

실제로 감싸려는 함수는 인자를 받고 값을 반환하는 경우가 대부분이다. `*args`, `**kwargs`(PY-09)를 활용하면 어떤 함수든 감쌀 수 있는 범용 데코레이터를 만들 수 있다.

```python
def deco(func):
    def wrapper(*args, **kwargs):
        print(f"{func.__name__} 호출 시작, 인자: {args}, {kwargs}")
        result = func(*args, **kwargs)
        print(f"{func.__name__} 호출 종료, 반환값: {result}")
        return result
    return wrapper

@deco
def add(a, b):
    return a + b

print(add(3, 4))
```

```text
(base) C:\Users\guest\project> python deco_args.py
add 호출 시작, 인자: (3, 4), {}
add 호출 종료, 반환값: 7
7
```

- `wrapper(*args, **kwargs)`는 원래 함수가 어떤 개수·형태의 인자를 받든 그대로 받아, `func(*args, **kwargs)`로 그대로 다시 넘겨준다. 이 덕분에 `deco`는 `add`뿐 아니라 인자 구성이 전혀 다른 함수에도 동일하게 적용할 수 있는 범용 데코레이터가 된다.
- `wrapper`가 `func`, `args`, `kwargs`를 함수가 끝난 뒤에도 기억하고 사용할 수 있는 이유는 `wrapper`가 `deco` 내부에서 정의된 **클로저(closure)**이기 때문이다. `deco(add)`가 호출되어 끝난 뒤에도, `deco` 안에서 만들어진 `wrapper` 함수는 자신을 둘러싼 `func`(여기서는 `add`)를 계속 참조할 수 있다.

**functools.wraps로 원본 함수 정보 보존하기**

데코레이터로 감싼 함수는 겉보기에 원래 함수와 이름이 같아 보이지만, 실제로는 `wrapper`라는 새 함수로 완전히 대체된 것이다. 이 때문에 원본 함수의 이름이나 독스트링 같은 메타데이터가 사라지는 문제가 생긴다.

```python
def deco(func):
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)
    return wrapper

@deco
def add(a, b):
    """두 수를 더한다."""
    return a + b

print(add.__name__)
print(add.__doc__)
```

```text
(base) C:\Users\guest\project> python deco_no_wraps.py
wrapper
None
```

- 데코레이터를 적용하고 나면 `add`라는 이름이 가리키는 실제 객체는 `add`가 아니라 `wrapper`이므로, `add.__name__`(PY-09의 함수 애너테이션과 마찬가지로 함수가 가진 속성)을 확인하면 원본 이름 `"add"` 대신 `"wrapper"`가 나오고, `add.__doc__`도 원본의 `"두 수를 더한다."` 대신 `wrapper`에는 독스트링이 없으므로 `None`이 된다. 이 상태로는 `help(add)`를 실행하거나 디버깅 도구로 함수를 살펴봐도 원본 정보를 알 수 없다.

```python
import functools

def deco(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)
    return wrapper

@deco
def add(a, b):
    """두 수를 더한다."""
    return a + b

print(add.__name__)
print(add.__doc__)
```

```text
(base) C:\Users\guest\project> python deco_wraps.py
add
두 수를 더한다.
```

- `functools.wraps(func)`도 그 자체로 하나의 데코레이터이며, `wrapper` 함수 정의 위에 붙이면 `wrapper`의 `__name__`, `__doc__` 같은 메타데이터를 원본 `func`의 것으로 덮어써준다. 그 결과 `add.__name__`이 `"wrapper"`가 아닌 `"add"`로, `add.__doc__`도 원본 독스트링 그대로 유지된다.
- 데코레이터를 직접 작성할 때는 `wrapper` 함수 위에 `@functools.wraps(func)`를 붙이는 것을 사실상 관례로 취급한다. 이를 생략해도 프로그램이 당장 오류를 내지는 않지만, 함수 이름과 문서가 뒤바뀌어 디버깅과 문서화 도구가 오작동하는 원인이 되기 쉽다.

**인자를 받는 데코레이터 — 데코레이터 팩토리**

`@repeat(3)`처럼 데코레이터 자체에 인자를 넘기고 싶을 때는, 함수를 한 겹 더 감싸 "데코레이터를 만들어 반환하는 함수"를 작성한다.

```python
import functools

def repeat(times):
    def deco(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            result = None
            for _ in range(times):
                result = func(*args, **kwargs)
            return result
        return wrapper
    return deco

@repeat(3)
def greet(name):
    print(f"{name}님 안녕하세요")

greet("김철수")
```

```text
(base) C:\Users\guest\project> python deco_factory.py
김철수님 안녕하세요
김철수님 안녕하세요
김철수님 안녕하세요
```

- `repeat(3)`은 데코레이터가 아니라 **데코레이터를 반환하는 함수**다. `repeat(3)`을 먼저 호출하면 `times=3`을 기억하는 `deco` 함수가 반환되고, `@repeat(3)`은 이렇게 반환된 `deco`를 곧바로 `greet` 위에 적용한 것과 같다. 즉 `@repeat(3) def greet(...): ...`는 `greet = repeat(3)(greet)`를 줄여 쓴 것이다.
- 함수가 `times` → `func` → `*args, **kwargs` 순서로 세 겹(`repeat` → `deco` → `wrapper`)으로 중첩되어 있으며, 가장 안쪽의 `wrapper`가 바깥의 두 함수가 기억해둔 `times`와 `func`를 모두 클로저로 참조할 수 있기 때문에 이런 구조가 성립한다.

**정리**: 데코레이터는 `@deco` 문법으로 `함수 = deco(함수)`라는 대입을 축약해 표현한 것으로, `def deco(func): def wrapper(*args, **kwargs): ...; return wrapper`처럼 클로저로 원본 함수를 감싼 `wrapper`를 반환하며, `functools.wraps(func)`를 `wrapper` 위에 붙여 원본 함수의 `__name__`/`__doc__`이 사라지지 않도록 보존하는 것이 관례이고, `repeat(times)`처럼 데코레이터 자체에 인자를 넘기고 싶을 때는 함수를 한 겹 더 감싸 데코레이터를 만들어 반환하는 데코레이터 팩토리 형태로 작성한다.

## dataclasses

**@dataclass가 자동으로 생성해주는 것들**

PY-06에서는 클래스를 정의할 때 `__init__`을 직접 작성해 속성을 초기화했다. 속성 몇 개를 저장하는 용도의 단순한 클래스라면, `dataclasses` 모듈의 `@dataclass` 데코레이터가 반복적으로 작성해야 하는 `__init__`, `__repr__`, `__eq__` 같은 메서드를 자동으로 만들어준다.

```python
from dataclasses import dataclass

@dataclass
class Point:
    x: int
    y: int = 0

p1 = Point(1, 2)
p2 = Point(1, 2)
p3 = Point(x=5)

print(p1)
print(p1 == p2)
print(p3)
```

```text
(base) C:\Users\guest\project> python dataclass_basic.py
Point(x=1, y=2)
True
Point(x=5, y=0)
```

- 클래스 몸체에 `x: int`, `y: int = 0`처럼 PY-09의 함수 애너테이션과 같은 문법으로 필드를 선언하면, `@dataclass`는 이 필드들을 순서대로 받는 `__init__(self, x, y=0)`을 자동으로 생성한다. `y: int = 0`처럼 값을 지정한 필드는 함수의 기본 인자와 마찬가지로 생략할 수 있는 매개변수가 된다.
- `print(p1)`이 `Point(x=1, y=2)`처럼 클래스 이름과 필드 값을 보기 좋게 보여주는 것은, `@dataclass`가 `__repr__`도 함께 자동으로 생성해주기 때문이다. 일반 클래스였다면 PY-06에서처럼 `__repr__`이나 `__str__`을 직접 작성해야 `<__main__.Point object at 0x...>` 대신 이런 출력을 얻을 수 있다.
- `p1 == p2`가 `True`인 것은 `@dataclass`가 `__eq__`도 자동으로 생성해, 같은 클래스의 두 인스턴스는 필드 값이 모두 같을 때 같다고 판단하도록 만들어주기 때문이다. 일반 클래스에서 `__eq__`를 직접 정의하지 않으면 두 인스턴스는 필드 값이 같아도 서로 다른 객체로 취급되어 `==` 비교가 `False`가 된다.

**field(default_factory=...)가 필요한 이유**

PY-03에서 함수의 기본 인자로 리스트 같은 가변(mutable) 객체를 직접 쓰면 안 되는 함정을 다뤘다. dataclass의 필드도 동일한 이유로, 가변 객체를 기본값으로 직접 쓰는 것을 아예 허용하지 않는다.

```python
from dataclasses import dataclass

@dataclass
class Cart:
    items: list = []
```

```text
(base) C:\Users\guest\project> python dataclass_mutable_default.py
Traceback (most recent call last):
  File "dataclass_mutable_default.py", line 4, in <module>
    class Cart:
  File "<string>", line 4, in Cart
ValueError: mutable default <class 'list'> for field items is not allowed: use default_factory
```

- 함수 기본 인자의 가변 객체 함정(PY-03)은 함수가 정의될 때 기본값 리스트가 딱 한 번만 만들어져 모든 호출이 같은 리스트를 공유하게 되는 문제였다. dataclass도 같은 방식으로 필드 기본값이 클래스 정의 시점에 단 하나만 만들어져 모든 인스턴스가 공유하게 되므로, `@dataclass`는 이 위험한 패턴을 아예 클래스 정의 시점에 `ValueError`로 막아버린다.

```python
from dataclasses import dataclass, field

@dataclass
class Cart:
    items: list = field(default_factory=list)

c1 = Cart()
c2 = Cart()
c1.items.append("사과")

print(c1.items)
print(c2.items)
```

```text
(base) C:\Users\guest\project> python dataclass_default_factory.py
['사과']
[]
```

- `field(default_factory=list)`는 "기본값으로 쓸 객체 하나"가 아니라 "인스턴스를 만들 때마다 기본값을 새로 만들어줄 함수"를 지정한다. `list`(인자 없이 호출하면 빈 리스트를 반환하는 함수)를 넘기면, `Cart()`가 호출될 때마다 매번 새로운 빈 리스트가 만들어져 `items`에 대입된다.
- 그 결과 `c1.items.append("사과")`로 `c1`의 리스트에 값을 추가해도 `c2.items`는 여전히 빈 리스트로 남아 있다. 딕셔너리나 세트처럼 다른 가변 타입을 기본값으로 쓸 때도 `field(default_factory=dict)`, `field(default_factory=set)`처럼 동일한 방식을 사용한다.

**frozen=True로 불변 인스턴스 만들기**

```python
from dataclasses import dataclass

@dataclass(frozen=True)
class Coordinate:
    lat: float
    lng: float

c = Coordinate(37.5, 127.0)
print(c)
c.lat = 40.0
```

```text
(base) C:\Users\guest\project> python dataclass_frozen.py
Coordinate(lat=37.5, lng=127.0)
Traceback (most recent call last):
  File "dataclass_frozen.py", line 9, in <module>
    c.lat = 40.0
dataclasses.FrozenInstanceError: cannot assign to field 'lat'
```

- `@dataclass(frozen=True)`로 선언한 클래스의 인스턴스는 한 번 생성된 뒤에는 필드 값을 다시 대입할 수 없다. 필드를 변경하려 시도하면 `dataclasses.FrozenInstanceError`가 발생한다.
- 좌표, 설정값, 색상 값처럼 한 번 만들어진 뒤에는 값이 바뀌지 않아야 의미가 유지되는 데이터를 표현할 때 `frozen=True`를 사용하면, 실수로 값을 덮어쓰는 버그를 애초에 언어 차원에서 막을 수 있다. `frozen=True`인 dataclass는 필드 값만 같다면 서로 다른 인스턴스라도 해시가 가능해져 세트의 원소나 딕셔너리의 키로도 사용할 수 있다.

**정리**: `@dataclass`는 클래스 몸체에 `이름: 타입 = 기본값` 형태로 필드를 선언해두면 `__init__`, `__repr__`, `__eq__`를 자동으로 만들어주어 데이터를 담는 목적의 클래스를 훨씬 짧게 작성할 수 있게 해주며, 리스트·딕셔너리 같은 가변 객체를 기본값으로 직접 쓰면 PY-03에서 다룬 함수 기본 인자 함정과 같은 이유로 `ValueError`가 발생하므로 `field(default_factory=...)`로 인스턴스마다 새 객체를 만들도록 지정해야 하고, `frozen=True`를 주면 필드 값을 재대입할 수 없는 불변 인스턴스를 만들어 값이 실수로 바뀌는 것을 막을 수 있다.

## typing 심화

**Optional과 Union — 값이 없거나 여러 타입 중 하나일 때**

PY-09에서는 `name: str`, `-> str`처럼 하나의 고정된 타입을 명시하는 기본적인 애너테이션을 다뤘다. 실제로는 함수가 값을 찾지 못해 `None`을 반환하거나, 여러 타입 중 하나를 받아들여야 하는 경우가 많다.

```python
from typing import Optional

def find_user(user_id: int) -> Optional[str]:
    users = {1: "김철수", 2: "이영희"}
    return users.get(user_id)

print(find_user(1))
print(find_user(99))
```

```text
(base) C:\Users\guest\project> python typing_optional.py
김철수
None
```

- `Optional[str]`은 `Union[str, None]`, 그리고 이를 더 짧게 쓴 `str | None`과 완전히 동일한 의미다. "이 함수는 `str`을 반환하거나, 값을 찾지 못하면 `None`을 반환할 수 있다"는 것을 반환 타입만 보고도 알 수 있게 해준다.
- `dict.get(key)`는 PY-04에서 다룬 것처럼 키가 없으면 `KeyError` 대신 `None`을 반환하므로, `find_user`의 실제 동작과 `Optional[str]`이라는 선언이 정확히 일치한다.

```python
from typing import Union

def to_number(value: Union[int, str]) -> int | float:
    return float(value) if isinstance(value, str) else value

print(to_number("3.5"))
print(to_number(10))
```

```text
(base) C:\Users\guest\project> python typing_union.py
3.5
10
```

- `Union[int, str]`은 "`int` 또는 `str` 중 하나"라는 뜻이며, 파이썬 3.10부터는 `int | str`처럼 파이프 문자로 더 짧게 쓸 수 있다. 매개변수와 반환값 모두에 `Union[...]`과 `|` 문법을 자유롭게 섞어 쓸 수 있다.
- `isinstance(value, str)`(PY-06에서 다룬 타입 확인 함수)로 실제 타입을 직접 검사해 분기하는 점에 주목한다. 타입 힌트는 어떤 타입이 들어올 수 있는지 **문서화**해줄 뿐, 실제로 어떤 타입이 들어왔는지 구분해 처리하는 로직은 여전히 직접 작성해야 한다.

**list[int], dict[str, int] — 제네릭 컨테이너 타입**

```python
def average(numbers: list[int]) -> float:
    return sum(numbers) / len(numbers)

scores: dict[str, int] = {"김철수": 90, "이영희": 85}

print(average([1, 2, 3]))
print(scores)
```

```text
(base) C:\Users\guest\project> python typing_generics.py
2.0
{'김철수': 90, '이영희': 85}
```

- `list[int]`는 "정수를 담은 리스트", `dict[str, int]`는 "문자열 키와 정수 값을 가진 딕셔너리"라는 의미의 애너테이션이다. 파이썬 3.9부터는 `typing.List`, `typing.Dict` 같은 별도의 타입을 불러오지 않아도 내장 `list`, `dict`에 대괄호로 원소 타입을 바로 표시할 수 있다.
- 함수 매개변수뿐 아니라 `scores: dict[str, int] = {...}`처럼 일반 변수를 선언할 때도 같은 문법을 사용할 수 있다.

**타입 힌트는 런타임에 강제되지 않는다**

```python
def average(numbers: list[int]) -> float:
    return sum(numbers) / len(numbers)

print(average([1.5, 2.5, 3.0]))
```

```text
(base) C:\Users\guest\project> python typing_not_enforced.py
2.3333333333333335
```

- `numbers: list[int]`라고 명시했지만 실제로는 실수(`float`)가 담긴 리스트를 넘겼는데도, 파이썬 인터프리터는 이를 오류로 처리하지 않고 `sum()`과 `len()`이 정상적으로 동작하는 그대로 실행한다. PY-09에서 다룬 것처럼 애너테이션은 사람과 도구를 위한 문서일 뿐, 실행 시점에 값의 타입을 강제로 검사하지 않는다.
- 타입 힌트를 실제로 어겼을 때 오류를 잡아내려면 `mypy` 같은 별도의 정적 타입 검사 도구를 코드 실행 전에 따로 돌리거나, 함수 내부에서 `isinstance()`로 직접 검사해야 한다.

**Protocol — 구조적 타이핑**

`Protocol`은 "어떤 클래스를 상속했는가"가 아니라 "어떤 메서드를 갖추고 있는가"만으로 타입을 정의하는 방법이다. 이런 방식을 **구조적 타이핑(structural typing)**이라 부른다.

```python
from typing import Protocol

class Sized(Protocol):
    def __len__(self) -> int: ...

def describe_length(obj: Sized) -> str:
    return f"길이: {len(obj)}"

print(describe_length([1, 2, 3]))
print(describe_length("hello"))
print(describe_length({"a": 1, "b": 2}))
```

```text
(base) C:\Users\guest\project> python typing_protocol.py
길이: 3
길이: 5
길이: 2
```

- `Sized`는 `list`나 `str`을 상속하지 않았지만, `describe_length`의 매개변수 타입으로 `Sized`를 지정할 수 있다. `list`, `str`, `dict`는 모두 `__len__` 메서드(PY-06에서 다룬 특수 메서드)를 가지고 있어 `Sized`가 요구하는 "구조"를 만족하기 때문이다.
- 구조적 타이핑은 "상속 관계와 무관하게, 필요한 메서드만 갖추고 있으면 같은 타입으로 취급한다"는 발상이며, 서로 상속 관계가 없는 여러 타입을 동일한 방식으로 다루는 함수를 설계할 때 유용하다. `Protocol` 역시 `Optional`, `Union`과 마찬가지로 런타임에 실제로 강제되는 것이 아니라 타입 검사 도구와 문서화를 위한 선언이다.

**정리**: `Optional[X]`(= `X | None`)는 값이 없을 수 있는 경우를, `Union[X, Y]`(= `X | Y`)는 여러 타입 중 하나일 수 있는 경우를 나타내며, `list[int]`나 `dict[str, int]`처럼 내장 컨테이너에 원소 타입을 대괄호로 붙여 제네릭 타입을 표현할 수 있고, `Protocol`은 상속 관계 대신 필요한 메서드의 유무만으로 타입을 정의하는 구조적 타이핑을 가능하게 해주지만, 이 모든 타입 힌트는 실행 시점에 강제되지 않는 문서화 정보일 뿐이라는 점을 항상 기억해야 한다.

## functools와 itertools

**functools.lru_cache — 메모이제이션**

같은 입력에 대해 항상 같은 결과를 반환하는 함수를 반복 호출할 때, 이미 계산한 결과를 저장해두었다가 재사용하는 기법을 **메모이제이션(memoization)**이라 부른다. `functools.lru_cache`는 이를 데코레이터 한 줄로 적용할 수 있게 해준다.

```python
import functools

@functools.lru_cache(maxsize=None)
def fib(n):
    if n < 2:
        return n
    return fib(n - 1) + fib(n - 2)

print(fib(30))
print(fib.cache_info())
```

```text
(base) C:\Users\guest\project> python functools_lru_cache.py
832040
CacheInfo(hits=28, misses=31, maxsize=None, currsize=31)
```

- 캐시 없이 `fib(n)`을 재귀로 계산하면 `fib(n-1)`과 `fib(n-2)`를 계산하는 과정에서 같은 `fib(k)` 값을 셀 수 없이 여러 번 다시 계산하게 되어, `n`이 커질수록 호출 횟수가 기하급수적으로 늘어난다. `@functools.lru_cache`를 붙이면 `fib(k)`가 처음 계산된 이후로는 같은 `k`에 대한 결과를 다시 계산하지 않고 저장해둔 값을 즉시 돌려주므로, 전체 계산량이 `n`에 비례하는 수준으로 크게 줄어든다.
- `fib.cache_info()`는 캐시가 실제로 얼마나 활용되었는지 `hits`(캐시에 저장된 값을 그대로 사용한 횟수)와 `misses`(캐시에 없어 새로 계산한 횟수)로 보여준다. `fib(0)`부터 `fib(30)`까지 각 값은 한 번씩만 새로 계산되므로 `misses`는 31이고, 그 외에 이미 계산된 값을 재사용한 횟수가 `hits`로 28만큼 집계된 것을 확인할 수 있다.
- `maxsize=None`은 캐시 크기에 제한을 두지 않는다는 뜻이다. `maxsize=128`처럼 숫자를 지정하면, 가장 오랫동안 사용되지 않은(least recently used) 항목부터 캐시에서 밀어내면서 캐시 크기를 일정하게 유지한다.

**functools.partial — 인자 일부 고정**

```python
import functools

def power(base, exponent):
    return base ** exponent

square = functools.partial(power, exponent=2)
cube = functools.partial(power, exponent=3)

print(square(5))
print(cube(2))
```

```text
(base) C:\Users\guest\project> python functools_partial.py
25
8
```

- `functools.partial(power, exponent=2)`는 `power` 함수의 `exponent` 인자를 `2`로 미리 고정해둔 새로운 함수 `square`를 만든다. `square(5)`를 호출하면 실질적으로 `power(5, exponent=2)`가 호출된 것과 같다.
- 자주 같은 인자 조합으로 호출하는 함수가 있을 때, 매번 나머지 인자를 반복해서 적어주는 대신 `partial`로 미리 고정한 전용 함수를 만들어두면 호출부가 간결해진다.

**itertools.chain과 itertools.combinations**

`itertools` 모듈은 반복 가능한 객체(iterable)를 다루는 다양한 도구를 제공한다. 그중 자주 쓰이는 두 가지를 살펴본다.

```python
import itertools

fruits = ["사과", "바나나"]
vegetables = ["당근", "감자"]

for item in itertools.chain(fruits, vegetables):
    print(item)
```

```text
(base) C:\Users\guest\project> python itertools_chain.py
사과
바나나
당근
감자
```

- `itertools.chain(fruits, vegetables)`는 두 개(또는 그 이상)의 리스트를 실제로 합친 새 리스트를 만들지 않고도, 마치 하나로 이어진 것처럼 순서대로 순회할 수 있게 해준다. `fruits + vegetables`로 새 리스트를 만드는 것과 순회 결과는 같지만, `chain`은 새로운 리스트를 메모리에 만들지 않고 필요할 때마다 값을 하나씩 꺼내준다는 차이가 있다.

```python
import itertools

for combo in itertools.combinations(["A", "B", "C"], 2):
    print(combo)
```

```text
(base) C:\Users\guest\project> python itertools_combinations.py
('A', 'B')
('A', 'C')
('B', 'C')
```

- `itertools.combinations(["A", "B", "C"], 2)`는 주어진 목록에서 순서를 고려하지 않고 2개씩 뽑을 수 있는 모든 조합을 튜플로 생성한다. `("A", "B")`와 `("B", "A")`는 같은 조합으로 취급되어 하나만 나온다는 점에 주의한다. 순서까지 구분한 모든 배열이 필요하다면 `itertools.permutations`를 사용한다.

**정리**: `functools.lru_cache`는 같은 입력에 대해 반복 계산되는 함수(대표적으로 재귀로 구현한 피보나치 수열)의 결과를 캐시에 저장해두어 중복 계산을 없애고 `cache_info()`로 캐시 활용도까지 확인할 수 있게 해주며, `functools.partial`은 함수의 일부 인자를 미리 고정한 새 함수를 만들어 반복되는 호출을 간결하게 해주고, `itertools.chain`은 여러 반복 가능한 객체를 새 리스트 없이 이어서 순회하게 해주며 `itertools.combinations`는 순서를 고려하지 않는 조합을 자동으로 생성해준다.

## pathlib

**Path 객체와 / 연산자로 경로 다루기**

PY-07에서는 `os.path` 모듈로 파일 경로를 문자열 형태로 다뤘다. `pathlib`은 경로 자체를 하나의 객체로 다루게 해주어, 문자열을 직접 이어붙이는 대신 더 안전하고 읽기 쉬운 방식으로 경로를 조작할 수 있게 해준다.

```python
from pathlib import Path

base = Path("project_data")
file_path = base / "reports" / "notes.txt"

print(file_path)
print(type(file_path))
print(file_path.parent)
print(file_path.name, file_path.suffix, file_path.stem)
```

```text
(base) C:\Users\guest\project> python pathlib_basic.py
project_data\reports\notes.txt
<class 'pathlib.WindowsPath'>
project_data\reports
notes.txt .txt notes
```

- `Path("project_data")`는 문자열 `"project_data"`를 감싼 경로 객체를 만든다. `/` 연산자로 `base / "reports" / "notes.txt"`처럼 경로 조각을 이어붙이면, `os.path.join()`을 여러 번 호출한 것과 같은 결과를 훨씬 짧고 직관적인 문법으로 얻을 수 있다.
- 운영체제마다 경로 구분자가 다르다는 점(Windows는 `\`, macOS·Linux는 `/`)을 `Path`가 알아서 처리해주므로, `/` 연산자로 경로를 조합하는 코드는 운영체제와 무관하게 동일하게 작성할 수 있다. 위 결과는 Windows 환경에서 실행했기 때문에 `\`로 표시된 `WindowsPath`가 출력된 것이며, macOS·Linux에서는 `/`로 표시된 `PosixPath`가 된다.
- `.parent`(상위 폴더), `.name`(마지막 이름), `.suffix`(확장자), `.stem`(확장자를 뺀 이름)처럼 경로의 구성 요소를 속성으로 바로 꺼내 쓸 수 있다. `os.path`였다면 `os.path.splitext()`, `os.path.dirname()`처럼 목적별로 서로 다른 함수를 호출해야 했을 정보다.

**exists(), is_file(), mkdir(), glob()**

```python
from pathlib import Path

base = Path("project_data")
base.mkdir(exist_ok=True)

file_path = base / "notes.txt"
file_path.write_text("안녕하세요", encoding="utf-8")

print(file_path.exists())
print(file_path.is_file())
print(base.is_dir())

for p in base.glob("*.txt"):
    print(p)
```

```text
(base) C:\Users\guest\project> python pathlib_ops.py
True
True
True
project_data\notes.txt
```

- `base.mkdir(exist_ok=True)`는 `base`가 가리키는 폴더를 생성한다. `exist_ok=True`를 주면 이미 같은 이름의 폴더가 있어도 오류를 내지 않고 넘어가며, 지정하지 않으면 이미 존재하는 폴더를 다시 만들려 할 때 `FileExistsError`가 발생한다.
- `file_path.write_text("안녕하세요", encoding="utf-8")`는 `open()`으로 파일을 열고 쓰고 닫는 과정을 한 번에 처리해주는 메서드로, 짧은 텍스트 파일을 만들 때 편리하다.
- `.exists()`는 경로가 실제로 존재하는지, `.is_file()`은 그 경로가 폴더가 아닌 파일인지, `.is_dir()`은 폴더인지를 각각 `True`/`False`로 알려준다.
- `base.glob("*.txt")`는 `base` 폴더 안에서 `*.txt` 패턴(PY-14에서 다룬 `glob` 모듈과 같은 와일드카드 패턴)에 일치하는 항목들을 `Path` 객체로 순회하게 해준다.

**os.path 문자열 방식과의 대비**

```python
import os.path

joined = os.path.join("project_data", "reports", "notes.txt")
print(joined)
print(os.path.basename(joined))
print(os.path.splitext(joined))
```

```text
(base) C:\Users\guest\project> python ospath_compare.py
project_data\reports\notes.txt
notes.txt
('project_data\\reports\\notes', '.txt')
```

- `os.path`는 경로를 처음부터 끝까지 **문자열**로 다룬다. 경로를 구성하려면 `os.path.join()`을, 파일 이름만 뽑으려면 `os.path.basename()`을, 확장자를 분리하려면 `os.path.splitext()`처럼 필요한 기능마다 서로 다른 이름의 함수를 찾아 호출해야 한다.
- `pathlib`은 같은 작업을 `Path` 객체 하나가 가진 `/`, `.name`, `.suffix` 같은 연산자와 속성으로 통일해서 제공하므로, 경로를 다루는 코드가 더 짧고 일관된 방식으로 읽힌다. 새로 작성하는 코드에서는 `pathlib`을 우선 사용하는 것이 권장되며, `os.path`는 기존 코드를 유지보수하거나 문자열 그대로 경로를 다뤄야 하는 특수한 상황에서 여전히 쓰인다.

**정리**: `pathlib`의 `Path` 객체는 `/` 연산자로 경로를 이어붙이고 `.parent`/`.name`/`.suffix`/`.stem`으로 경로의 구성 요소를 바로 꺼내 쓸 수 있게 해주며, `.exists()`/`.is_file()`/`.mkdir()`/`.glob()` 같은 메서드로 파일 존재 확인·폴더 생성·패턴 검색까지 객체 하나로 처리할 수 있어, PY-07에서 다룬 `os.path`의 문자열 기반 함수 조합보다 경로를 다루는 코드를 더 짧고 일관되게 작성하게 해준다.

## argparse

**sys.argv 수동 파싱의 한계**

PY-14에서 다룬 `sys.argv`는 커맨드라인에서 넘긴 인자를 문자열 리스트로 그대로 받아온다. 인자가 하나둘일 때는 `sys.argv[1]`처럼 인덱스로 바로 꺼내 쓸 수 있지만, 인자가 늘어나고 `--옵션` 형태까지 지원하려면 이 리스트를 직접 순회하며 하나하나 해석하는 코드를 작성해야 하고, 필수 인자가 빠졌을 때의 안내 메시지나 `--help` 옵션도 모두 직접 만들어야 한다.

**argparse.ArgumentParser 기본 사용**

`argparse`는 어떤 인자를 받을지 선언만 해두면, 실제 인자 해석과 타입 변환, 오류 메시지, `--help` 출력까지 자동으로 처리해주는 표준 라이브러리다.

```python
# greet_cli.py
import argparse

parser = argparse.ArgumentParser(description="인사말 출력 프로그램")
parser.add_argument("name", help="인사할 대상 이름")
parser.add_argument("--times", type=int, default=1, help="인사말을 반복할 횟수")

args = parser.parse_args()

for _ in range(args.times):
    print(f"{args.name}님 안녕하세요!")
```

```text
(base) C:\Users\guest\project> python greet_cli.py 김철수
김철수님 안녕하세요!
```

```text
(base) C:\Users\guest\project> python greet_cli.py 김철수 --times 2
김철수님 안녕하세요!
김철수님 안녕하세요!
```

- `parser.add_argument("name", ...)`처럼 `-`나 `--`로 시작하지 않는 이름을 넘기면 **위치 인자(positional argument)**로 등록된다. 위치 인자는 순서대로 반드시 넘겨야 하는 필수 값이며, 실행할 때 `python greet_cli.py 김철수`처럼 옵션 이름 없이 값만 적는다.
- `parser.add_argument("--times", type=int, default=1, ...)`처럼 `--`로 시작하는 이름을 넘기면 **옵션 인자(optional argument)**로 등록된다. `type=int`는 커맨드라인에서 문자열로 들어온 값을 자동으로 `int`로 변환하게 하고, `default=1`은 `--times`를 생략했을 때 사용할 기본값을 지정한다.
- `parser.parse_args()`는 `sys.argv`를 내부적으로 읽어 등록해둔 규칙에 따라 해석한 뒤, 각 인자를 속성으로 가진 객체를 반환한다. `--times`처럼 하이픈이 포함된 옵션 이름도 `args.times`처럼 언더스코어로 자동 변환된 이름의 속성으로 접근한다.

**필수 인자 누락 시 자동 오류 메시지와 --help**

```text
(base) C:\Users\guest\project> python greet_cli.py
usage: greet_cli.py [-h] [--times TIMES] name
greet_cli.py: error: the following arguments are required: name
```

```text
(base) C:\Users\guest\project> python greet_cli.py --help
usage: greet_cli.py [-h] [--times TIMES] name

인사말 출력 프로그램

positional arguments:
  name           인사할 대상 이름

options:
  -h, --help     show this help message and exit
  --times TIMES  인사말을 반복할 횟수
```

- 필수 위치 인자인 `name`을 넘기지 않으면, `argparse`는 자동으로 사용법(`usage`)과 어떤 인자가 빠졌는지 알려주는 오류 메시지를 출력하고 프로그램을 종료한다. `sys.argv`를 직접 다룬다면 이런 검증과 메시지 작성을 모두 손수 구현해야 한다.
- `-h`/`--help` 옵션은 `add_argument()`로 등록하지 않아도 `ArgumentParser`가 기본으로 제공한다. `--help`를 실행하면 `description`으로 지정한 설명과, 각 `add_argument()`에 넘긴 `help` 문구를 모아 정리된 사용법 안내를 자동으로 생성해준다.

**정리**: `argparse.ArgumentParser`에 `add_argument()`로 위치 인자와 `--옵션` 형태의 인자를 선언하고 `type`과 `default`로 값의 변환 규칙과 기본값을 지정한 뒤 `parse_args()`를 호출하면, PY-14에서 다룬 `sys.argv`를 직접 순회하며 해석하는 코드 없이도 인자를 속성으로 바로 꺼내 쓸 수 있고, 필수 인자 누락 시의 오류 메시지와 `--help` 안내까지 자동으로 만들어주므로 커맨드라인 도구를 만들 때는 `sys.argv`를 직접 파싱하기보다 `argparse`를 사용하는 것이 훨씬 안전하고 편리하다.

## 실습 예제 (EX1~EX4)

**EX1) 데코레이터로 함수 실행 시간 측정하기**
- 어떤 함수든 감싸 실행 전후 시각의 차이를 출력하는 데코레이터를 작성하고, `functools.wraps`로 원본 함수 이름을 보존한 뒤 확인한다.

```python
# ex01_deco_timer.py
import functools
import time

def timer(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        start = time.time()
        result = func(*args, **kwargs)
        elapsed = time.time() - start
        print(f"{func.__name__} 실행 시간: {elapsed:.4f}초")
        return result
    return wrapper

@timer
def slow_sum(n):
    return sum(range(n))

print(slow_sum(1_000_000))
print(slow_sum.__name__)
```

```text
(base) C:\Users\guest\project> python ex01_deco_timer.py
slow_sum 실행 시간: 0.0123초
499999500000
slow_sum
```

**EX2) dataclass로 학생 성적 관리하기**
- 이름과 점수 목록을 갖는 학생 dataclass를 만들고, `field(default_factory=list)`로 점수 목록의 기본값 문제를 피하면서 평균 점수를 계산하는 메서드를 구현한다.

```python
# ex02_dataclass_student.py
from dataclasses import dataclass, field

@dataclass
class Student:
    name: str
    scores: list = field(default_factory=list)

    def average(self):
        if not self.scores:
            return 0
        return sum(self.scores) / len(self.scores)

kim = Student("김철수")
lee = Student("이영희")

kim.scores.append(90)
kim.scores.append(80)

print(kim)
print(lee)
print(f"{kim.name} 평균: {kim.average()}")
print(f"{lee.name} 평균: {lee.average()}")
```

```text
(base) C:\Users\guest\project> python ex02_dataclass_student.py
Student(name='김철수', scores=[90, 80])
Student(name='이영희', scores=[])
김철수 평균: 85.0
이영희 평균: 0
```

**EX3) lru_cache와 itertools로 소수 조합 정리하기**
- 소수 여부를 판별하는 함수에 `lru_cache`를 적용해 반복 판별을 캐시로 처리하고, `itertools.combinations`로 소수 목록에서 두 개씩 뽑은 조합의 합을 계산한다.

```python
# ex03_prime_combinations.py
import functools
import itertools

@functools.lru_cache(maxsize=None)
def is_prime(n):
    if n < 2:
        return False
    for i in range(2, int(n ** 0.5) + 1):
        if n % i == 0:
            return False
    return True

numbers = range(2, 12)
primes = [n for n in numbers if is_prime(n)]
print("소수 목록:", primes)

for a, b in itertools.combinations(primes, 2):
    print(f"{a} + {b} = {a + b}")

print(is_prime.cache_info())
```

```text
(base) C:\Users\guest\project> python ex03_prime_combinations.py
소수 목록: [2, 3, 5, 7, 11]
2 + 3 = 5
2 + 5 = 7
2 + 7 = 9
2 + 11 = 13
3 + 5 = 8
3 + 7 = 10
3 + 11 = 14
5 + 7 = 12
5 + 11 = 16
7 + 11 = 18
CacheInfo(hits=0, misses=10, maxsize=None, currsize=10)
```

**EX4) argparse와 pathlib로 파일 통계 CLI 만들기**
- 폴더 경로를 인자로 받아 그 안의 특정 확장자 파일 개수를 세는 커맨드라인 도구를 `argparse`와 `pathlib`으로 작성한다.

```python
# ex04_count_files_cli.py
import argparse
from pathlib import Path

parser = argparse.ArgumentParser(description="폴더 안의 파일 개수를 세는 도구")
parser.add_argument("folder", help="검사할 폴더 경로")
parser.add_argument("--ext", default=".txt", help="검사할 확장자 (기본값: .txt)")

args = parser.parse_args()

target = Path(args.folder)
target.mkdir(exist_ok=True)

pattern = f"*{args.ext}"
matched = list(target.glob(pattern))

print(f"{target} 폴더에서 '{pattern}' 패턴과 일치하는 파일: {len(matched)}개")
for p in matched:
    print("-", p.name)
```

```text
(base) C:\Users\guest\project> python ex04_count_files_cli.py project_data --ext .txt
project_data 폴더에서 '*.txt' 패턴과 일치하는 파일: 1개
- notes.txt
```

**정리**: EX1~EX4는 데코레이터로 함수 실행 시간을 측정하며 `functools.wraps`로 원본 이름을 지키는 방법, dataclass와 `field(default_factory=list)`로 안전한 기본값을 가진 학생 성적 클래스를 구현하는 방법, `lru_cache`로 반복되는 소수 판별을 캐시하면서 `itertools.combinations`로 조합을 정리하는 방법, 그리고 `argparse`와 `pathlib`을 함께 사용해 폴더 안의 파일 개수를 세는 실용적인 커맨드라인 도구를 작성하는 방법까지, 이 문서에서 다룬 언어 심화 주제를 실무에 가까운 형태로 조합해보는 예제다.

[Python 16 — 표준 라이브러리 심화와 테스트](16-stdlib-advanced.md)

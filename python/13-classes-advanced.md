# Python 13 — 클래스 심화 (이터레이터·제너레이터)

## 이름과 스코프 — LEGB 규칙

PY-03에서 지역 변수와 전역 변수, 그리고 중첩 함수 안에서 바깥 함수의 변수를 참조하는 예제를 다뤘다. 이 규칙을 하나의 원칙으로 정리한 것이 **LEGB 규칙**이다. 파이썬은 어떤 이름을 참조할 때 다음 네 범위를 순서대로 찾는다.

- **L (Local)**: 현재 실행 중인 함수 내부
- **E (Enclosing)**: 그 함수를 감싸고 있는 바깥 함수(중첩 함수의 경우)
- **G (Global)**: 모듈(파일) 최상위 수준
- **B (Built-in)**: `len`, `print`처럼 파이썬이 기본으로 제공하는 이름

```python
x = "전역"

def outer():
    x = "인클로징"

    def inner():
        x = "지역"
        print("inner:", x)

    inner()
    print("outer:", x)

outer()
print("모듈:", x)
```

```text
(base) C:\Users\guest\project> python legb_basic.py
inner: 지역
outer: 인클로징
모듈: 전역
```

- `inner()` 안에서 `x`를 참조하면 가장 가까운 범위인 지역(L)에 있는 `x`부터 찾으므로 `"지역"`이 출력된다. 만약 `inner()`에 `x = "지역"`이라는 대입이 없었다면, 그다음으로 감싸는 범위인 `outer()`의 `x`(E)를 찾아 `"인클로징"`을 사용했을 것이다.

```python
def outer2():
    x = "인클로징"

    def inner2():
        print("inner2:", x)  # 지역에 x가 없으므로 인클로징(outer2)의 x를 찾는다

    inner2()

outer2()
```

```text
(base) C:\Users\guest\project> python legb_enclosing.py
inner2: 인클로징
```

- `inner2()` 안에는 `x`에 대한 대입문이 없으므로, 파이썬은 지역(L) → 인클로징(E) 순서로 올라가 `outer2()`의 `x`를 찾아 사용한다. 만약 `outer2()`에도 `x`가 없다면 전역(G), 그다음 내장(B) 순서로 계속 찾다가 어디에서도 찾지 못하면 `NameError`가 발생한다.
- PY-03에서 다뤘듯, 함수 내부에서 바깥 범위의 변수에 새로 값을 **대입**하려면 `global`이나 `nonlocal` 키워드를 명시해야 한다. LEGB 규칙은 이름을 **읽을 때** 어떤 순서로 찾는지에 대한 규칙이고, `global`/`nonlocal`은 이름에 **쓸(대입할)** 범위를 지정하는 것이라는 점에서 서로 역할이 다르다.

**정리**: LEGB 규칙은 파이썬이 이름을 참조할 때 지역(Local) → 인클로징(Enclosing) → 전역(Global) → 내장(Built-in) 순서로 가장 가까운 범위부터 찾아나간다는 원칙으로, PY-03에서 다룬 지역/전역 변수 구분과 중첩 함수 안에서 바깥 함수 변수를 읽는 동작이 모두 이 하나의 규칙으로 설명되며, 값을 읽는 것(LEGB 탐색)과 값을 새로 대입하는 것(`global`/`nonlocal` 필요)은 서로 다른 규칙을 따른다는 점을 기억해야 한다.

## 클래스 변수 vs 인스턴스 변수

PY-06에서는 `__init__` 안에서 `self.속성 = 값` 형태로 만드는 **인스턴스 변수**를 다뤘다. 이번에는 클래스 몸체에 직접 선언해 모든 인스턴스가 공유하는 **클래스 변수**를 살펴본다.

```python
class Dog:
    species = "Canis familiaris"  # 클래스 변수: 모든 인스턴스가 공유

    def __init__(self, name):
        self.name = name  # 인스턴스 변수: 각 인스턴스마다 독립적

d1 = Dog("바둑이")
d2 = Dog("초코")

print(d1.name, d1.species)
print(d2.name, d2.species)
print(Dog.species)
```

```text
(base) C:\Users\guest\project> python class_var_basic.py
바둑이 Canis familiaris
초코 Canis familiaris
Canis familiaris
```

- `species`처럼 `__init__` 밖, 클래스 몸체에 직접 선언한 변수는 **클래스 변수**로, 그 클래스로 만든 모든 인스턴스가 같은 값을 공유한다. 인스턴스를 통해서도(`d1.species`) 클래스를 통해서도(`Dog.species`) 접근할 수 있다.
- `name`처럼 `self.name = name`으로 만든 변수는 **인스턴스 변수**로, 인스턴스마다 독립적인 값을 가진다.

**클래스 변수를 변경할 때 주의할 점**

```python
class Counter:
    total = 0  # 클래스 변수: 생성된 인스턴스 수를 센다

    def __init__(self):
        Counter.total += 1  # 클래스 이름으로 접근해서 값을 바꿔야 공유 변수가 실제로 바뀐다

c1 = Counter()
c2 = Counter()
c3 = Counter()

print("생성된 개수:", Counter.total)
```

```text
(base) C:\Users\guest\project> python class_var_shared.py
생성된 개수: 3
```

- 클래스 변수를 실제로 변경하려면 `Counter.total += 1`처럼 **클래스 이름**을 통해 접근해서 바꿔야 한다. 만약 `self.total += 1`처럼 쓰면, 이는 클래스 변수를 바꾸는 것이 아니라 `self`에 `total`이라는 **새로운 인스턴스 변수**를 만들어버리는 것이므로 의도와 다르게 동작한다.

```python
class Counter2:
    total = 0

    def __init__(self):
        self.total += 1  # 주의: 인스턴스 변수를 새로 만드는 것!

c1 = Counter2()
c2 = Counter2()

print("c1.total:", c1.total)
print("c2.total:", c2.total)
print("Counter2.total(클래스 변수는 그대로):", Counter2.total)
```

```text
(base) C:\Users\guest\project> python class_var_mistake.py
c1.total: 1
c2.total: 1
Counter2.total(클래스 변수는 그대로): 0
```

- `self.total += 1`은 `self.total = self.total + 1`과 같은데, 오른쪽의 `self.total`을 읽을 때는 인스턴스에 없으므로 클래스 변수(`0`)를 찾아 읽지만, 왼쪽의 `self.total = ...`는 **대입**이므로 각 인스턴스에 독립적인 `total` 인스턴스 변수를 새로 만들어버린다. 그 결과 `Counter2.total` 클래스 변수 자체는 `0`으로 그대로 남고, 각 인스턴스는 자기만의 `total = 1`을 갖게 된다.

**정리**: 클래스 변수는 클래스 몸체에 직접 선언되어 모든 인스턴스가 값을 공유하는 변수이고, 인스턴스 변수는 `__init__` 안에서 `self.속성 = 값`으로 만들어져 인스턴스마다 독립적인 값을 갖는 변수이며, 클래스 변수를 실제로 변경하려면 `self.변수 = ...`가 아니라 `클래스이름.변수 = ...`로 접근해야 한다는 점을 주의해야 한다. 그렇지 않으면 클래스 변수를 읽어서 새로운 인스턴스 변수를 만드는 실수로 이어지기 쉽다.

## 비공개 변수 — _변수와 __변수

파이썬은 다른 언어처럼 `private` 키워드로 접근을 완전히 차단하는 문법을 제공하지 않는다. 대신 이름 앞에 밑줄을 붙이는 **관례**와, 두 개의 밑줄을 붙였을 때 일어나는 **네임 맹글링(name mangling)**이라는 언어 차원의 기능을 함께 사용한다.

**밑줄 하나(`_변수`) — 관례상의 비공개**

```python
class BankAccount:
    def __init__(self, balance):
        self._balance = balance  # 관례: "내부용이니 직접 건드리지 말 것"

    def deposit(self, amount):
        self._balance += amount
        return self._balance

acc = BankAccount(1000)
print(acc.deposit(500))
print(acc._balance)  # 문법적으로는 접근 가능하다
```

```text
(base) C:\Users\guest\project> python underscore_single.py
1500
1500
```

- `_balance`처럼 밑줄 하나로 시작하는 이름은 "이 변수는 클래스 내부 구현용이니 외부에서 직접 접근하지 않는 것이 좋다"는 **관례적인 신호**일 뿐이다. 파이썬은 이를 문법적으로 막지 않으므로 `acc._balance`처럼 여전히 접근할 수 있다.

**밑줄 두 개(`__변수`) — 네임 맹글링**

```python
class BankAccount2:
    def __init__(self, balance):
        self.__balance = balance

    def deposit(self, amount):
        self.__balance += amount
        return self.__balance

acc = BankAccount2(1000)
print(acc.deposit(500))

try:
    print(acc.__balance)
except AttributeError as e:
    print("접근 오류:", e)

print(acc._BankAccount2__balance)
```

```text
(base) C:\Users\guest\project> python underscore_double.py
1500
접근 오류: 'BankAccount2' object has no attribute '__balance'
1500
```

- `__balance`처럼 밑줄 두 개로 시작하는(끝은 밑줄 두 개가 아닌) 이름은 파이썬 내부적으로 `_클래스이름__balance` 형태로 이름이 자동 변환되는데, 이를 **네임 맹글링**이라고 한다.
- 그 결과 클래스 바깥에서 원래 이름 `acc.__balance`로 접근하면 실제로는 존재하지 않는 이름이 되어 `AttributeError`가 발생한다. 다만 변환된 이름 `acc._BankAccount2__balance`로는 여전히 접근이 가능하므로, 이는 완전한 차단이 아니라 자식 클래스가 실수로 같은 이름을 재정의해 부모의 속성을 덮어써버리는 사고를 방지하기 위한 장치에 가깝다.

**정리**: 파이썬에는 다른 언어의 `private` 같은 강제적인 접근 제한 문법이 없으며, `_변수`는 "내부용이니 건드리지 말라"는 관례적 신호일 뿐 문법적으로는 여전히 접근 가능하고, `__변수`는 `_클래스이름__변수`로 이름이 자동 변환되는 네임 맹글링 덕분에 클래스 바깥이나 자식 클래스에서 같은 이름으로 실수로 충돌하는 것을 막아주지만 완전히 접근을 차단하는 것은 아니다.

## 다중 상속과 메서드 결정 순서(MRO)

PY-07까지는 부모 클래스가 하나뿐인 **단일 상속**만 다뤘다. 파이썬은 `class 자식(부모1, 부모2, ...):`처럼 괄호 안에 클래스를 여러 개 나열하는 **다중 상속**도 지원한다. 문제는 여러 부모가 같은 이름의 메서드를 가지고 있을 때 파이썬이 어떤 부모의 것을 먼저 쓸지인데, 이 순서를 정하는 규칙이 **MRO(Method Resolution Order, 메서드 결정 순서)**다.

```python
class Flyer:
    def move(self):
        return "날아서 이동"

class Swimmer:
    def move(self):
        return "헤엄쳐서 이동"

class Duck(Flyer, Swimmer):
    pass

d = Duck()
print(d.move())
print(Duck.__mro__)
```

```text
(base) C:\Users\guest\project> python mro_basic.py
날아서 이동
(<class '__main__.Duck'>, <class '__main__.Flyer'>, <class '__main__.Swimmer'>, <class 'object'>)
```

- `Duck(Flyer, Swimmer)`처럼 괄호 안에 부모 클래스를 왼쪽부터 나열하면, 같은 이름의 메서드가 여러 부모에 있어도 **먼저 적은 부모**의 것이 우선 적용된다. `Flyer`를 `Swimmer`보다 먼저 썼기 때문에 `d.move()`는 `Flyer.move`를 실행한다.
- `클래스이름.__mro__`(또는 `클래스이름.mro()`)를 출력해보면 파이썬이 속성을 찾을 때 실제로 훑는 클래스 순서를 튜플로 직접 확인할 수 있다. 모든 클래스는 결국 `object`로 끝난다.

**다이아몬드 상속과 `super()`**

여러 부모가 다시 하나의 공통 조상을 공유하는 구조를 **다이아몬드 상속**이라 부른다. 이때 각 부모 클래스가 `super()`로 협력하면, 자식 쪽에서 한 번만 호출해도 부모들이 정해진 순서대로 연쇄적으로 실행된다.

```python
class Animal:
    def describe(self):
        return "동물"

class Flyer2(Animal):
    def describe(self):
        return super().describe() + " > 날 수 있음"

class Swimmer2(Animal):
    def describe(self):
        return super().describe() + " > 헤엄칠 수 있음"

class Duck2(Flyer2, Swimmer2):
    def describe(self):
        return super().describe() + " > 오리"

d2 = Duck2()
print(d2.describe())
for cls in Duck2.__mro__:
    print(cls.__name__, end=" -> ")
print("끝")
```

```text
(base) C:\Users\guest\project> python mro_diamond.py
동물 > 헤엄칠 수 있음 > 날 수 있음 > 오리
Duck2 -> Flyer2 -> Swimmer2 -> Animal -> object -> 끝
```

- `Duck2`, `Flyer2`, `Swimmer2`, `Animal`이 다이아몬드 모양(`Duck2`의 두 부모가 결국 같은 `Animal`을 공유)으로 얽혀 있어도, `Duck2.__mro__`를 출력해보면 파이썬이 **C3 선형화(linearization)** 알고리즘으로 `[Duck2, Flyer2, Swimmer2, Animal, object]`라는 중복 없는 한 줄의 순서를 계산해둔 것을 알 수 있다. 각 부모는 MRO에 정확히 한 번씩만 나타난다.
- `Duck2.describe()`에서 `super().describe()`를 호출하면, 파이썬이 `Duck2`가 아니라 "`self`의 MRO에서 `Duck2` 바로 다음 클래스"인 `Flyer2`의 `describe`를 실행한다. `Flyer2.describe` 역시 자신의 `super().describe()`로 다음 순서인 `Swimmer2`를 호출하고, `Swimmer2`는 다시 `Animal`을 호출한다. 그 결과 `Animal`의 결과부터 거슬러 올라오며 문자열이 이어 붙어, 부모를 두 번 부르지 않고도 모든 조상 클래스의 로직이 한 번씩 순서대로 실행된다.
- 만약 `super()` 없이 각 클래스가 부모를 직접 이름으로 호출(`Animal.describe(self)`)했다면, 다이아몬드 구조에서는 `Animal.describe`가 두 번 호출될 위험이 있다. `super()`가 `self.__class__`가 아니라 **MRO 상의 다음 클래스**를 기준으로 동작하기 때문에, 다중 상속 구조에서도 공통 조상이 중복 실행되지 않는다.
- MRO는 무작정 정해지는 것이 아니라, 자식 클래스를 부모보다 항상 앞에 두고 부모들 사이의 왼쪽-우선 순서를 지키도록 계산되며, 이 규칙을 지킬 수 없게 클래스를 설계하면(예: 부모 나열 순서가 서로 모순되는 경우) `TypeError: Cannot create a consistent method resolution order`가 발생해 클래스 정의 자체가 실패한다.

**정리**: 다중 상속은 `class 자식(부모1, 부모2, ...):`처럼 부모를 여러 개 나열해 여러 클래스의 기능을 동시에 물려받는 방법이며, 같은 이름의 메서드가 여러 부모에 있을 때 어느 것을 쓸지는 `클래스이름.__mro__`로 확인 가능한 **MRO(메서드 결정 순서)**가 정하고, 부모를 나열한 왼쪽 순서를 지키면서 각 조상 클래스가 정확히 한 번씩만 나타나도록 계산된다는 점이 핵심이다. 여러 부모가 공통 조상을 공유하는 다이아몬드 상속 구조에서는 `super()`가 `self.__class__`가 아닌 MRO상의 다음 클래스를 호출해주기 때문에, 자식이 한 번만 호출해도 부모들이 MRO 순서대로 연쇄 협력하며 공통 조상이 중복 실행되지 않는다.

## 이터레이터

`for item in 리스트:`처럼 반복 가능한 객체를 순회할 수 있는 이유는, 그 객체들이 **이터레이터 프로토콜**을 따르기 때문이다. 이 프로토콜은 `__iter__`와 `__next__`라는 두 메서드로 이루어진다.

```python
numbers = [10, 20, 30]

it = iter(numbers)  # __iter__() 호출과 같다
print(next(it))      # __next__() 호출과 같다
print(next(it))
print(next(it))

try:
    print(next(it))
except StopIteration:
    print("더 이상 꺼낼 값이 없습니다.")
```

```text
(base) C:\Users\guest\project> python iterator_basic.py
10
20
30
더 이상 꺼낼 값이 없습니다.
```

- `iter(객체)`는 객체의 `__iter__()` 메서드를 호출해 **이터레이터**를 얻는다. 리스트 자체는 **이터러블(iterable)**이지만 이터레이터는 아니며, `iter()`를 통해 별도의 이터레이터 객체를 만들어낸다.
- `next(이터레이터)`는 이터레이터의 `__next__()` 메서드를 호출해 다음 값을 하나씩 꺼내며, 더 꺼낼 값이 없으면 `StopIteration` 예외를 발생시킨다.
- 사실 `for item in numbers:` 문은 내부적으로 `iter(numbers)`로 이터레이터를 얻고, `StopIteration`이 발생할 때까지 반복해서 `next()`를 호출하는 과정을 자동으로 처리해주는 것이다.

**커스텀 이터레이터 클래스 만들기**

```python
class CountUp:
    """start부터 end까지 1씩 증가하며 값을 내놓는 이터레이터."""

    def __init__(self, start, end):
        self.current = start
        self.end = end

    def __iter__(self):
        return self  # 이터레이터 자신을 반환

    def __next__(self):
        if self.current > self.end:
            raise StopIteration
        value = self.current
        self.current += 1
        return value

for n in CountUp(1, 5):
    print(n, end=" ")
print()

it = CountUp(1, 3)
print(list(it))
```

```text
(base) C:\Users\guest\project> python custom_iterator.py
1 2 3 4 5
[1, 2, 3]
```

- `CountUp`은 `__iter__`(자기 자신을 반환)와 `__next__`(다음 값을 계산하거나 끝났으면 `StopIteration` 발생)를 직접 구현했기 때문에, `for`문으로 바로 순회할 수 있고 `list()`처럼 이터러블을 받는 함수에도 그대로 넘길 수 있다.
- 상태(`self.current`)를 인스턴스 변수로 직접 관리해야 하므로, 반복이 진행될 때마다 "지금 어디까지 왔는지"를 스스로 기억하고 갱신해야 한다는 점이 이터레이터 클래스를 만들 때 신경 써야 할 부분이다.

**정리**: 이터레이터 프로토콜은 `__iter__()`로 이터레이터 객체를 얻고 `__next__()`로 값을 하나씩 꺼내다가 더 이상 값이 없으면 `StopIteration`을 발생시키는 약속이며, `for`문은 이 과정을 자동으로 반복해주는 문법적 설탕(syntactic sugar)에 불과하고, `__iter__`와 `__next__`를 직접 구현하면 리스트나 range처럼 `for`문으로 순회 가능한 커스텀 클래스를 만들 수 있다.

## 제너레이터

이터레이터 클래스를 매번 `__iter__`/`__next__`와 상태 변수를 직접 관리하며 만드는 것은 번거롭다. **제너레이터(generator)**는 `yield` 키워드를 사용해 훨씬 간결하게 같은 결과를 만들어내는 방법이다.

```python
def count_up(start, end):
    current = start
    while current <= end:
        yield current
        current += 1

for n in count_up(1, 5):
    print(n, end=" ")
print()

gen = count_up(1, 3)
print(type(gen))
print(next(gen))
print(next(gen))
print(next(gen))
```

```text
(base) C:\Users\guest\project> python generator_basic.py
1 2 3 4 5
<class 'generator'>
1
2
3
```

- 함수 몸체 안에 `yield`가 하나라도 있으면, 그 함수는 호출해도 즉시 실행되지 않고 **제너레이터 객체**를 반환한다.
- `next()`를 호출할 때마다 함수는 이전에 멈췄던 지점부터 다시 실행되어 다음 `yield`를 만날 때까지 진행하고, 그 값을 반환한 뒤 다시 멈춘다. 함수가 끝까지 실행되어 더 내놓을 값이 없으면 자동으로 `StopIteration`이 발생한다.
- 앞서 만든 `CountUp` 클래스와 동일한 동작을 하지만, `self.current` 같은 상태를 직접 관리할 필요 없이 파이썬이 함수의 실행 위치와 지역 변수 상태를 자동으로 기억해준다.

**제너레이터 함수와 커스텀 이터레이터 클래스 비교**

| 구분 | 커스텀 이터레이터 클래스 | 제너레이터 함수 |
|------|------------------------|-----------------|
| 문법 | `__iter__`/`__next__` 직접 구현 | 함수 안에 `yield` 사용 |
| 상태 관리 | 인스턴스 변수로 직접 관리 | 파이썬이 함수 실행 지점을 자동으로 기억 |
| 코드량 | 상대적으로 많음 | 상대적으로 적음 |
| 재사용 | 클래스를 상속해 확장 가능 | 함수 조합으로 재사용 |

```python
def fibonacci(n):
    a, b = 0, 1
    count = 0
    while count < n:
        yield a
        a, b = b, a + b
        count += 1

print(list(fibonacci(8)))
```

```text
(base) C:\Users\guest\project> python generator_fibonacci.py
[0, 1, 1, 2, 3, 5, 8, 13]
```

- 피보나치 수열처럼 "다음 값이 이전 상태에 따라 정해지는" 반복을 만들 때, 제너레이터는 지역 변수(`a`, `b`, `count`)에 상태를 자연스럽게 담아둘 수 있어 클래스로 만드는 것보다 훨씬 짧고 읽기 쉽다.

**정리**: 제너레이터는 함수 안에 `yield`를 사용해 이터레이터를 간결하게 만드는 방법으로, `__iter__`/`__next__`를 직접 구현하고 상태를 인스턴스 변수로 관리해야 하는 커스텀 이터레이터 클래스와 달리 파이썬이 함수의 실행 위치와 지역 변수를 자동으로 기억해주기 때문에 훨씬 적은 코드로 같은 결과를 얻을 수 있으며, 특히 이전 값에 의존해 다음 값을 계산하는 수열 같은 반복 로직에 적합하다.

## 제너레이터 표현식

PY-10에서 다룬 리스트 컴프리헨션(`[x for x in ...]`)과 매우 비슷하지만 대괄호 대신 소괄호를 쓰는 **제너레이터 표현식**이 있다.

```python
squares_list = [x**2 for x in range(1, 6)]       # 리스트 컴프리헨션: 즉시 전체 계산
squares_gen = (x**2 for x in range(1, 6))          # 제너레이터 표현식: 값을 미리 계산하지 않음

print(squares_list)
print(squares_gen)
print(type(squares_gen))

for value in squares_gen:
    print(value, end=" ")
print()
```

```text
(base) C:\Users\guest\project> python genexpr_basic.py
[1, 4, 9, 16, 25]
<generator object <genexpr> at 0x000001A2B3C4D5E0>
<class 'generator'>
1 4 9 16 25
```

- 리스트 컴프리헨션 `[x**2 for x in range(1, 6)]`은 실행되는 즉시 결괏값 5개를 모두 계산해 하나의 리스트로 메모리에 올려둔다.
- 제너레이터 표현식 `(x**2 for x in range(1, 6))`은 실행 즉시 계산하지 않고, 값이 실제로 필요할 때(예: `for`문에서 순회하거나 `next()`를 호출할 때)마다 하나씩 계산해서 내놓는 제너레이터 객체를 만든다. 출력해보면 계산된 값이 아니라 `<generator object ...>`라는 객체 표현이 보인다는 점이 리스트와의 뚜렷한 차이다.

**메모리 효율 비교**

```python
import sys

list_version = [x for x in range(1000000)]
gen_version = (x for x in range(1000000))

print("리스트 크기(바이트):", sys.getsizeof(list_version))
print("제너레이터 크기(바이트):", sys.getsizeof(gen_version))
```

```text
(base) C:\Users\guest\project> python genexpr_memory.py
리스트 크기(바이트): 8448728
제너레이터 크기(바이트): 200
```

- 백만 개의 값을 담은 리스트는 그 값들을 모두 메모리에 저장해야 하므로 크기가 크지만, 제너레이터는 "값을 어떻게 계산할지에 대한 정보"만 가지고 있을 뿐 실제 값을 미리 계산해서 담아두지 않으므로 데이터 개수와 상관없이 항상 작은 고정 크기를 유지한다. 실제 바이트 수는 파이썬 버전과 실행 환경에 따라 달라질 수 있지만, 리스트가 데이터 개수에 비례해 커지고 제너레이터는 거의 일정하게 유지된다는 경향은 동일하다.
- 값을 한 번씩만 순서대로 사용하고 전체를 동시에 메모리에 들고 있을 필요가 없다면, 특히 값의 개수가 많을 때 제너레이터 표현식이 리스트 컴프리헨션보다 메모리 효율이 훨씬 좋다.
- 단, 제너레이터는 한 번 순회하면 소진되어 다시 처음부터 순회할 수 없다는 점과, 인덱싱(`gen[0]`)이나 `len()`을 바로 사용할 수 없다는 점에서 리스트와 다르므로, 여러 번 재사용하거나 인덱스로 접근해야 하는 데이터라면 리스트가 더 적합하다.

**정리**: 제너레이터 표현식은 `(x for x in ...)` 형태로 리스트 컴프리헨션과 문법이 거의 같지만 결과를 즉시 계산해 메모리에 담아두는 대신 값이 필요할 때마다 하나씩 계산해서 내놓기 때문에, 특히 큰 데이터를 한 번만 순서대로 처리할 때는 메모리 사용량을 크게 줄일 수 있으며, 대신 재순회나 인덱싱이 불가능하다는 제약을 감안해서 상황에 맞게 리스트 컴프리헨션과 선택해서 사용해야 한다.

## 실습 예제 (EX1~EX4)

**EX1) LEGB 규칙 확인하기**
- 같은 이름의 변수를 지역/인클로징/전역에 각각 만들어두고, 중첩 함수에서 어떤 값이 출력되는지 확인한다.

```python
# ex01_legb_check.py
message = "전역 메시지"

def outer():
    message = "인클로징 메시지"

    def inner():
        print("inner에서 읽은 값:", message)

    inner()

outer()
print("모듈에서 읽은 값:", message)
```

```text
(base) C:\Users\guest\project> python ex01_legb_check.py
inner에서 읽은 값: 인클로징 메시지
모듈에서 읽은 값: 전역 메시지
```

**EX2) 클래스 변수로 인스턴스 개수 세기**
- 클래스 변수를 이용해 지금까지 생성된 인스턴스의 개수를 세는 클래스를 작성한다.

```python
# ex02_instance_counter.py
class Employee:
    count = 0

    def __init__(self, name):
        self.name = name
        Employee.count += 1

e1 = Employee("김철수")
e2 = Employee("이영희")
e3 = Employee("박민수")

print(f"현재까지 생성된 직원 수: {Employee.count}")
for e in (e1, e2, e3):
    print(f"- {e.name}")
```

```text
(base) C:\Users\guest\project> python ex02_instance_counter.py
현재까지 생성된 직원 수: 3
- 김철수
- 이영희
- 박민수
```

**EX3) 커스텀 이터레이터로 짝수만 꺼내기**
- 주어진 범위 안에서 짝수만 하나씩 반환하는 이터레이터 클래스를 작성한다.

```python
# ex03_even_iterator.py
class EvenNumbers:
    def __init__(self, start, end):
        self.current = start if start % 2 == 0 else start + 1
        self.end = end

    def __iter__(self):
        return self

    def __next__(self):
        if self.current > self.end:
            raise StopIteration
        value = self.current
        self.current += 2
        return value

print(list(EvenNumbers(1, 10)))
```

```text
(base) C:\Users\guest\project> python ex03_even_iterator.py
[2, 4, 6, 8, 10]
```

**EX4) 제너레이터로 무한 카운터 만들고 필요한 만큼만 꺼내기**
- 끝없이 1씩 증가하는 값을 내놓는 제너레이터를 만들고, 앞에서 5개만 꺼내 살펴본다.

```python
# ex04_infinite_counter.py
def infinite_counter(start=1):
    n = start
    while True:
        yield n
        n += 1

counter = infinite_counter()
first_five = [next(counter) for _ in range(5)]
print(first_five)

# 제너레이터 표현식으로 처음 5개 중 짝수만 골라내기
counter2 = infinite_counter()
even_only = (n for n in (next(counter2) for _ in range(10)) if n % 2 == 0)
print(list(even_only))
```

```text
(base) C:\Users\guest\project> python ex04_infinite_counter.py
[1, 2, 3, 4, 5]
[2, 4, 6, 8, 10]
```

- `infinite_counter()`처럼 끝없이 값을 내놓는 제너레이터는 리스트로는 만들 수 없는 방식이다. 리스트 컴프리헨션으로 `[n for n in infinite_counter()]`를 시도하면 무한히 값을 채우려다 메모리가 소진될 때까지 멈추지 않으므로, 필요한 만큼만 `next()`로 꺼내 쓰는 것이 제너레이터의 핵심 장점 중 하나다.

**정리**: EX1~EX4는 중첩 함수에서 이름이 LEGB 순서로 해석되는 과정, 클래스 변수를 이용해 인스턴스 수를 공유·추적하는 방법, `__iter__`/`__next__`를 직접 구현한 커스텀 이터레이터로 원하는 조건의 값만 순서대로 꺼내는 방법, 그리고 `yield`로 만든 제너레이터가 무한히 이어지는 값도 필요한 만큼만 지연 계산해서 다룰 수 있다는 점까지, 이 문서에서 다룬 스코프·클래스 변수·이터레이터·제너레이터 개념을 직접 코드로 확인해보는 예제다.

[Python 12 — 예외 처리 심화](12-exceptions-advanced.md) · [Python 14 — 표준 라이브러리 살펴보기](14-stdlib-tour.md)

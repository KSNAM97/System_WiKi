# Python 09 — 제어 흐름 심화

## match 문 — 구조적 패턴 매칭

파이썬 3.10부터 `match` 문이 추가되었다. PY-05에서 다룬 `if/elif/else`가 조건식을 순서대로 검사하는 방식이라면, `match`는 값의 **구조와 패턴**을 비교해 가장 먼저 일치하는 `case`를 실행하는 방식이다.

```python
def describe_status(code):
    match code:
        case 200:
            return "성공"
        case 404:
            return "찾을 수 없음"
        case 500:
            return "서버 오류"
        case _:
            return "알 수 없는 상태 코드"

print(describe_status(200))
print(describe_status(404))
print(describe_status(999))
```

```text
(base) C:\Users\guest\project> python match_basic.py
성공
찾을 수 없음
알 수 없는 상태 코드
```

- `match 대상:` 아래에 `case 값:` 을 여러 개 나열하면, 대상이 위에서부터 순서대로 각 패턴과 비교되며 처음 일치하는 `case`의 블록이 실행된다.
- `case _:`는 **와일드카드**로, 어떤 값과도 일치하는 마지막 처리 구간이다. `if/elif/else`의 `else`와 같은 역할을 한다.

**여러 패턴을 하나의 case에서 매칭하기**

```python
def describe_day(day):
    match day:
        case "토" | "일":
            return "주말"
        case "월" | "화" | "수" | "목" | "금":
            return "평일"
        case _:
            return "요일이 아닙니다"

print(describe_day("토"))
print(describe_day("수"))
```

```text
(base) C:\Users\guest\project> python match_multi.py
주말
평일
```

- `|`(파이프)로 여러 값을 하나의 `case`에 묶으면, 그 중 하나라도 일치할 때 해당 블록이 실행된다.

**패턴에 조건(guard) 추가하기**

```python
def classify_number(n):
    match n:
        case int() if n < 0:
            return "음수"
        case 0:
            return "0"
        case int() if n % 2 == 0:
            return "양의 짝수"
        case int():
            return "양의 홀수"
        case _:
            return "정수가 아닙니다"

for value in [-5, 0, 4, 7, 3.5]:
    print(value, "->", classify_number(value))
```

```text
(base) C:\Users\guest\project> python match_guard.py
-5 -> 음수
0 -> 0
4 -> 양의 짝수
7 -> 양의 홀수
3.5 -> 정수가 아닙니다
```

- `case 패턴 if 조건:` 형태로 패턴 뒤에 `if`를 붙이면, 패턴이 일치하더라도 조건식이 `True`일 때만 해당 `case`가 최종적으로 선택된다. 이를 **가드(guard)**라고 부른다.
- `int()`는 타입 패턴으로, 대상 값이 `int` 타입일 때 일치한다.

**시퀀스 패턴 매칭**

```python
def handle_command(command):
    match command:
        case ["이동", x, y]:
            return f"({x}, {y})로 이동"
        case ["회전", angle]:
            return f"{angle}도 회전"
        case ["정지"]:
            return "정지"
        case [action, *rest]:
            return f"알 수 없는 동작 '{action}', 나머지 인자: {rest}"
        case _:
            return "빈 명령"

print(handle_command(["이동", 10, 20]))
print(handle_command(["회전", 90]))
print(handle_command(["정지"]))
print(handle_command(["점프", 1, 2, 3]))
```

```text
(base) C:\Users\guest\project> python match_sequence.py
(10, 20)로 이동
90도 회전
정지
알 수 없는 동작 '점프', 나머지 인자: [1, 2, 3]
```

- 리스트 형태의 패턴 `["이동", x, y]`는 대상이 길이 3인 리스트이고 첫 요소가 `"이동"`일 때 일치하며, 동시에 `x`, `y`에 나머지 요소가 자동으로 바인딩된다.
- `[action, *rest]`처럼 `*변수명`을 쓰면 나머지 요소 전체를 리스트로 받을 수 있다. PY-04에서 다룬 슬라이싱과 비슷한 감각이다.

**클래스 패턴 매칭 기초**

```python
class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y

def locate(point):
    match point:
        case Point(x=0, y=0):
            return "원점"
        case Point(x=0, y=y):
            return f"y축 위, y={y}"
        case Point(x=x, y=0):
            return f"x축 위, x={x}"
        case Point(x=x, y=y):
            return f"일반 좌표 ({x}, {y})"
        case _:
            return "Point가 아닙니다"

print(locate(Point(0, 0)))
print(locate(Point(0, 5)))
print(locate(Point(3, 4)))
```

```text
(base) C:\Users\guest\project> python match_class.py
원점
y축 위, y=5
일반 좌표 (3, 4)
```

- `Point(x=0, y=0)`처럼 클래스 이름과 속성값을 함께 지정하면, 대상이 해당 클래스의 인스턴스이면서 지정한 속성 조건을 만족할 때 일치한다. PY-06에서 다룬 클래스와 `__init__`으로 만든 객체를 그대로 패턴 매칭에 활용할 수 있다.

**정리**: `match` 문은 값을 여러 `case` 패턴과 순서대로 비교해 가장 먼저 일치하는 블록을 실행하며, `|`로 여러 값을 한 번에 묶거나 `if` 가드로 추가 조건을 걸 수 있고, 리스트·클래스 같은 구조를 그대로 패턴으로 표현해 값을 분해하면서 동시에 매칭할 수 있다는 점에서 단순 값 비교를 반복하는 `if/elif`보다 구조가 있는 데이터를 다룰 때 더 간결하다.

## 임의 개수의 위치 인자 *args

PY-03에서는 정해진 개수의 매개변수만 다뤘다. 하지만 몇 개가 들어올지 미리 알 수 없는 경우, `*매개변수명`으로 **임의 개수의 위치 인자**를 하나의 튜플로 받을 수 있다.

```python
def total(*numbers):
    print(f"받은 인자: {numbers}, 타입: {type(numbers)}")
    return sum(numbers)

print(total(1, 2, 3))
print(total(10, 20))
print(total())
```

```text
(base) C:\Users\guest\project> python args_basic.py
받은 인자: (1, 2, 3), 타입: <class 'tuple'>
6
받은 인자: (10, 20), 타입: <class 'tuple'>
30
받은 인자: (), 타입: <class 'tuple'>
0
```

- `*numbers`는 함수를 호출할 때 넘긴 위치 인자를 개수에 상관없이 모두 모아 하나의 **튜플**로 만들어 `numbers`에 담는다.
- 인자를 하나도 넘기지 않으면 빈 튜플이 된다.

**일반 매개변수와 함께 사용하기**

```python
def print_report(title, *items):
    print(f"=== {title} ===")
    for i, item in enumerate(items, start=1):
        print(f"{i}. {item}")

print_report("장보기 목록", "우유", "계란", "빵")
```

```text
(base) C:\Users\guest\project> python args_mixed.py
=== 장보기 목록 ===
1. 우유
2. 계란
3. 빵
```

- `*args`는 반드시 일반(고정) 매개변수 뒤에 위치해야 하며, 그 뒤로 넘어오는 모든 위치 인자를 흡수한다.

**정리**: `*매개변수명`은 함수 호출 시 넘어온 임의 개수의 위치 인자를 튜플 하나로 묶어 받는 문법으로, 인자 개수를 호출 시점까지 확정할 수 없는 함수(예: 합계, 평균, 로그 출력)를 만들 때 사용하며, 고정 매개변수 뒤에 위치시켜 함께 쓸 수 있다.

## 임의 개수의 키워드 인자 **kwargs

`*args`가 위치 인자를 모으는 것과 달리, `**매개변수명`은 `이름=값` 형태로 넘어온 **임의 개수의 키워드 인자**를 하나의 딕셔너리로 받는다.

```python
def print_profile(**info):
    print(f"받은 인자: {info}, 타입: {type(info)}")
    for key, value in info.items():
        print(f"{key}: {value}")

print_profile(name="김철수", age=25, city="서울")
```

```text
(base) C:\Users\guest\project> python kwargs_basic.py
받은 인자: {'name': '김철수', 'age': 25, 'city': '서울'}, 타입: <class 'dict'>
name: 김철수
age: 25
city: 서울
```

- `**info`는 `name="김철수"`처럼 키워드 형태로 넘긴 인자를 모두 모아 딕셔너리로 만든다. PY-04에서 다룬 딕셔너리의 `items()` 메서드로 바로 순회할 수 있다.

**\*args와 \*\*kwargs 함께 사용하기**

```python
def create_user(username, *hobbies, **details):
    print(f"사용자: {username}")
    print(f"취미: {hobbies}")
    print(f"세부 정보: {details}")

create_user("kim", "독서", "등산", age=30, job="개발자")
```

```text
(base) C:\Users\guest\project> python args_kwargs_together.py
사용자: kim
취미: ('독서', '등산')
세부 정보: {'age': 30, 'job': '개발자'}
```

- 매개변수 순서는 `일반 매개변수 → *args → **kwargs` 순으로 고정되어 있다. 이 순서를 바꾸면 `SyntaxError`가 발생한다.

**정리**: `**매개변수명`은 함수 호출 시 넘어온 임의 개수의 키워드 인자(`이름=값`)를 딕셔너리 하나로 묶어 받는 문법이며, `*args`(위치 인자 모으기)와 함께 쓸 때는 `일반 매개변수, *args, **kwargs` 순서를 지켜야 하고, 설정값처럼 어떤 이름의 인자가 몇 개나 들어올지 정해지지 않은 함수를 설계할 때 유용하다.

## 인자 목록 언패킹

`*`와 `**`는 함수를 **정의**할 때뿐 아니라, 함수를 **호출**할 때도 반대 방향으로 사용할 수 있다. 이미 존재하는 리스트나 딕셔너리를 풀어서 개별 인자로 전달하는 것을 **언패킹(unpacking)**이라고 한다.

```python
def add_three(a, b, c):
    return a + b + c

numbers = [1, 2, 3]
print(add_three(*numbers))
```

```text
(base) C:\Users\guest\project> python unpack_list.py
6
```

- `add_three(*numbers)`는 리스트 `numbers`를 풀어 `add_three(1, 2, 3)`을 호출한 것과 동일하게 동작한다. 리스트의 길이가 함수가 받는 매개변수 개수와 정확히 일치해야 한다.

**딕셔너리를 키워드 인자로 언패킹하기**

```python
def introduce(name, age, city):
    print(f"{name}({age}세, {city} 거주)입니다.")

person = {"name": "이영희", "age": 28, "city": "부산"}
introduce(**person)
```

```text
(base) C:\Users\guest\project> python unpack_dict.py
이영희(28세, 부산 거주)입니다.
```

- `**person`은 딕셔너리의 키를 매개변수 이름으로, 값을 인자 값으로 매칭해 전달한다. 딕셔너리의 키 이름이 함수의 매개변수 이름과 정확히 일치해야 한다.

**두 방식을 함께 사용하기**

```python
def create_order(item, quantity, *, price, note="없음"):
    print(f"{item} {quantity}개, 단가 {price}원, 비고: {note}")

args = ["노트북", 2]
kwargs = {"price": 1200000, "note": "당일 배송"}
create_order(*args, **kwargs)
```

```text
(base) C:\Users\guest\project> python unpack_both.py
노트북 2개, 단가 1200000원, 비고: 당일 배송
```

**정리**: 함수 정의부에서 `*args`/`**kwargs`가 인자를 "모으는" 역할이라면, 함수 호출부에서 `*리스트`/`**딕셔너리`는 반대로 이미 있는 데이터를 "풀어서" 전달하는 역할을 하며, 리스트나 튜플에 담긴 값을 위치 인자로, 딕셔너리에 담긴 값을 키워드 인자로 한 번에 넘길 때 인자를 하나씩 나열하지 않아도 되어 코드가 간결해진다.

## 위치 전용 매개변수와 키워드 전용 인자

파이썬 함수의 매개변수는 기본적으로 위치 인자로도, 키워드 인자로도 넘길 수 있다. 하지만 함수 설계자가 이를 강제로 제한하고 싶을 때 `/`와 `*` 문법을 사용한다.

```python
def power(base, exp, /, *, mod=None):
    result = base ** exp
    if mod is not None:
        result = result % mod
    return result

print(power(2, 10))
print(power(2, 10, mod=100))
```

```text
(base) C:\Users\guest\project> python positional_keyword_only.py
1024
24
```

- `/` 앞에 있는 매개변수(`base`, `exp`)는 **위치 전용(positional-only)**으로, 반드시 위치 인자로만 넘겨야 하며 `power(base=2, exp=10)`처럼 키워드로 넘기면 오류가 발생한다.
- `*` 뒤에 있는 매개변수(`mod`)는 **키워드 전용(keyword-only)**으로, 반드시 `mod=값` 형태로만 넘겨야 하며 `power(2, 10, 100)`처럼 위치로 넘기면 오류가 발생한다.

```text
>>> power(base=2, exp=10)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: power() got some positional-only arguments passed as keyword arguments: 'base, exp'
>>> power(2, 10, 100)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: power() takes 2 positional arguments but 3 were given
```

- 이런 제약은 매개변수 이름을 나중에 자유롭게 바꿀 수 있게 하거나(위치 전용), 반대로 호출 코드만 보고도 어떤 값인지 이름으로 알 수 있게 강제하는(키워드 전용) 목적으로 사용된다. `mod=None` 같은 옵션값에 키워드 전용을 적용하면 호출부의 가독성이 크게 좋아진다.

**정리**: 매개변수 목록에서 `/` 앞은 위치 전용, `*` 뒤는 키워드 전용으로 강제되며, 그 사이(또는 `/`와 `*`가 없을 때)는 위치·키워드 어느 쪽으로도 넘길 수 있는 일반 매개변수이고, 이 문법은 함수의 호출 방식을 명확하게 제한해 API를 더 안전하고 읽기 쉽게 설계하고 싶을 때 사용한다.

## 람다 표현식

`lambda`는 이름 없이 한 줄로 정의하는 **익명 함수**다. `def`로 함수를 따로 선언하기엔 너무 간단한 로직을, 다른 함수의 인자로 바로 넘길 때 자주 사용한다.

```python
square = lambda x: x ** 2
add = lambda a, b: a + b

print(square(5))
print(add(3, 4))
```

```text
(base) C:\Users\guest\project> python lambda_basic.py
25
7
```

- `lambda 매개변수들: 반환할_표현식` 형태로 작성하며, `return` 키워드 없이 표현식의 결괏값이 자동으로 반환된다. `def`로 작성하면 다음과 동일하다.

```python
def square(x):
    return x ** 2
```

**정렬 키로 활용하기**

```python
students = [
    {"name": "김철수", "score": 85},
    {"name": "이영희", "score": 92},
    {"name": "박민수", "score": 78},
]

by_score = sorted(students, key=lambda s: s["score"], reverse=True)
for s in by_score:
    print(s["name"], s["score"])
```

```text
(base) C:\Users\guest\project> python lambda_sort_key.py
이영희 92
김철수 85
박민수 78
```

- `sorted()`의 `key` 매개변수에 `lambda s: s["score"]`를 넘기면, 각 요소에서 정렬 기준으로 쓸 값을 뽑아내는 규칙을 한 줄로 정의할 수 있다. 이런 짧은 "값 하나만 뽑아내는" 함수는 굳이 `def`로 이름을 붙일 필요가 없어 `lambda`가 특히 잘 어울린다.

**정리**: `lambda`는 `def`와 동일하게 함수를 만들지만 이름이 없고 표현식 하나만 반환할 수 있는 축약형이며, `sorted()`의 `key`처럼 다른 함수에 짧은 로직을 인자로 즉석에서 넘길 때 유용하지만, 로직이 복잡해지거나 재사용이 필요하다면 이름이 있는 `def` 함수로 작성하는 것이 가독성 면에서 더 낫다.

## 함수 애너테이션

PY-03에서 매개변수에 타입 힌트(`name: str`)를 붙이는 방법을 다뤘다. 이 타입 힌트는 사실 파이썬의 **함수 애너테이션(annotation)** 문법의 한 활용이며, 매개변수뿐 아니라 반환값에도 붙일 수 있고, 함수는 이 정보를 `__annotations__` 속성으로 저장해둔다.

```python
def greet(name: str, times: int = 1) -> str:
    return (f"{name}님 안녕하세요! " * times).strip()

print(greet("김철수", times=2))
print(greet.__annotations__)
```

```text
(base) C:\Users\guest\project> python annotations.py
김철수님 안녕하세요! 김철수님 안녕하세요!
{'name': <class 'str'>, 'times': <class 'int'>, 'return': <class 'str'>}
```

- `-> str`은 이 함수가 문자열을 반환할 것임을 나타내는 **반환값 애너테이션**이다.
- 함수의 `__annotations__` 속성을 통해 각 매개변수와 반환값에 붙은 타입 힌트를 딕셔너리 형태로 직접 확인할 수 있다. 반환값은 `'return'`이라는 키에 저장된다.
- 중요한 점은, 애너테이션은 **문서화와 IDE·타입 검사 도구를 위한 정보**일 뿐 파이썬 인터프리터가 실행 시점에 타입을 강제로 검사하지 않는다는 것이다. 다음처럼 타입을 어겨도 오류 없이 실행된다.

```python
print(greet(123))
```

```text
(base) C:\Users\guest\project> python annotations_ignored.py
123님 안녕하세요!
```

- `name: str`이라고 명시했음에도 정수 `123`을 넘기면 오류 없이 실행되며 결과는 예상과 다르게 나온다. 실제 타입 검증이 필요하다면 `mypy` 같은 별도의 정적 타입 검사 도구를 사용하거나, 함수 내부에서 직접 `isinstance()`로 검사해야 한다.

**정리**: 함수 애너테이션은 매개변수 뒤에 `: 타입`, 함수 정의 뒤에 `-> 타입`을 붙여 그 함수가 어떤 타입을 주고받도록 의도되었는지 명시하는 문법으로, 실행 시점에 강제되지는 않지만 `__annotations__`에 기록되어 코드를 읽는 사람과 IDE·타입 검사 도구에게 유용한 문서 역할을 하며, PY-03에서 다룬 타입 힌트가 바로 이 애너테이션 문법 위에서 동작한다.

## PEP 8 코딩 스타일

PEP 8은 파이썬 공식 커뮤니티가 정한 코드 스타일 가이드다. PY-02에서 변수명에 `snake_case`를 쓴다는 점을 이미 다뤘으므로, 여기서는 함수·클래스 네이밍과 그 외 기본 규칙 위주로 정리한다.

```python
# 좋은 예
def calculate_total_price(items):
    pass

class ShoppingCart:
    pass

MAX_RETRY_COUNT = 5

# 나쁜 예
def CalculateTotalPrice(items):   # 함수는 PascalCase를 쓰지 않는다
    pass

class shopping_cart:              # 클래스는 snake_case를 쓰지 않는다
    pass

maxRetryCount = 5                 # 변경되지 않는 상수는 대문자+언더스코어가 관례
```

- **함수/메서드**: 변수와 동일하게 `snake_case`(소문자와 언더스코어)를 사용한다. 예: `get_user_info()`, `is_valid()`.
- **클래스**: 단어의 첫 글자를 대문자로 붙여 쓰는 `PascalCase`(또는 CapWords)를 사용한다. 예: `ShoppingCart`, `UserProfile`. PY-06에서 만든 예제 클래스명(`Student`, `Animal` 등)도 이 관례를 따른 것이다.
- **상수**: 값이 바뀌지 않는 것으로 취급하는 변수는 모두 대문자와 언더스코어로 작성한다. 예: `MAX_RETRY_COUNT`, `DEFAULT_TIMEOUT`.

**들여쓰기와 한 줄 길이**

```text
- 들여쓰기는 공백(space) 4칸을 사용한다. 탭(tab)과 혼용하지 않는다.
- 한 줄의 길이는 79자를 넘지 않도록 권장한다(실무에서는 88~120자로 완화해 쓰는 경우도 흔하다).
- 함수/클래스 정의 사이에는 빈 줄을 2줄씩 두어 구분한다.
- 연산자 앞뒤로는 공백을 하나씩 둔다: a = 1 + 2 (O), a=1+2 (X)
```

- VS Code의 Python 확장은 저장할 때 이런 규칙을 자동으로 점검하거나(`Pylint`, `flake8`), 자동으로 정렬해주는(`Black`, `autopep8`) 기능을 제공하므로, 실무에서는 규칙을 외우기보다 도구에 맡기는 경우가 많다.

**정리**: PEP 8은 변수뿐 아니라 함수·메서드에는 `snake_case`를, 클래스에는 `PascalCase`를, 값이 바뀌지 않는 상수에는 `모두 대문자`를 사용하도록 권장하며, 여기에 4칸 들여쓰기와 적절한 줄 길이·공백 규칙을 더해 여러 사람이 같은 코드베이스를 다룰 때도 일관된 모양을 유지하게 해주는 커뮤니티 표준 스타일 가이드다.

## 실습 예제 (EX1~EX4)

**EX1) match 문으로 간단한 계산기 만들기**
- 연산자 문자열과 두 숫자를 받아 `match`로 분기하여 사칙연산 결과를 반환하는 함수를 작성하시오. 0으로 나누는 경우와 알 수 없는 연산자는 각각 다른 메시지를 반환한다.

```python
# ex01_match_calculator.py
def calculate(op, a, b):
    match op:
        case "+":
            return a + b
        case "-":
            return a - b
        case "*":
            return a * b
        case "/" if b == 0:
            return "0으로 나눌 수 없습니다."
        case "/":
            return a / b
        case _:
            return f"알 수 없는 연산자입니다: {op}"

print(calculate("+", 10, 3))
print(calculate("/", 10, 0))
print(calculate("%", 10, 3))
```

```text
(base) C:\Users\guest\project> python ex01_match_calculator.py
13
0으로 나눌 수 없습니다.
알 수 없는 연산자입니다: %
```

**EX2) *args로 가변 인자 평균 함수 만들기**
- 몇 개의 숫자가 들어오든 평균을 계산하는 함수를 `*args`로 작성하고, 인자가 없을 때는 0을 반환하도록 예외 상황을 처리하시오.

```python
# ex02_average.py
def average(*numbers):
    if not numbers:
        return 0
    return sum(numbers) / len(numbers)

print(average(10, 20, 30))
print(average(5))
print(average())
```

```text
(base) C:\Users\guest\project> python ex02_average.py
20.0
5.0
0
```

**EX3) **kwargs와 언패킹으로 설정값 병합하기**
- 기본 설정 딕셔너리와 사용자가 넘긴 `**kwargs`를 병합해 최종 설정을 반환하는 함수를 작성하시오. 사용자가 넘긴 값이 기본값을 덮어써야 한다.

```python
# ex03_merge_settings.py
def build_settings(**overrides):
    defaults = {"timeout": 30, "retries": 3, "debug": False}
    merged = {**defaults, **overrides}
    return merged

print(build_settings())
print(build_settings(timeout=60, debug=True))
```

```text
(base) C:\Users\guest\project> python ex03_merge_settings.py
{'timeout': 30, 'retries': 3, 'debug': False}
{'timeout': 60, 'retries': 3, 'debug': True}
```

- `{**defaults, **overrides}`처럼 딕셔너리 리터럴 안에서도 `**`로 언패킹을 사용할 수 있으며, 뒤에 오는 딕셔너리의 값이 같은 키를 가진 앞쪽 값을 덮어쓴다.

**EX4) lambda와 sorted로 여러 기준 정렬하기**
- 상품 목록을 가격 오름차순으로, 가격이 같으면 이름 오름차순으로 정렬하는 코드를 `lambda`로 작성하시오.

```python
# ex04_sort_products.py
products = [
    {"name": "마우스", "price": 15000},
    {"name": "키보드", "price": 15000},
    {"name": "모니터", "price": 250000},
]

sorted_products = sorted(products, key=lambda p: (p["price"], p["name"]))
for p in sorted_products:
    print(p["name"], p["price"])
```

```text
(base) C:\Users\guest\project> python ex04_sort_products.py
키보드 15000
마우스 15000
모니터 250000
```

- `key=lambda p: (p["price"], p["name"])`처럼 튜플을 반환하면, 첫 번째 값으로 먼저 정렬하고 값이 같을 때만 두 번째 값으로 다시 정렬하는 다중 기준 정렬을 한 줄로 표현할 수 있다.

**정리**: EX1~EX4는 `match` 문으로 구조화된 분기 처리를 하는 방법, `*args`로 가변 개수의 인자를 받아 안전하게 처리하는 방법, `**kwargs`와 딕셔너리 언패킹으로 설정값을 병합하는 방법, `lambda`를 `sorted()`의 정렬 키로 활용해 다중 기준 정렬을 구현하는 방법까지 이 문서에서 다룬 제어 흐름 심화 개념을 직접 코드로 확인해보는 예제다.

[Python 08 — 에러 처리](08-error-handling.md) · [Python 10 — 자료구조 심화](10-data-structures-advanced.md)

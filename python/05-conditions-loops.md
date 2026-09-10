# Python 05 — 조건문과 반복문

## if 문 기본 구조

**조건문(if statement)**은 특정 조건이 참(True)일 때만 코드를 실행하도록 흐름을 제어하는 문법이다.

```python
age = 20

if age >= 18:
    print("성인입니다")
```

```text
(base) C:\Users\guest\project> python if_basic.py
성인입니다
```

- `if 조건:` 뒤에 콜론(`:`)을 붙이고, 그 아래 들여쓰기된 줄이 조건이 참일 때만 실행되는 블록이다.
- 조건이 거짓(False)이면 `if` 블록 전체를 건너뛰고 그다음 코드로 넘어간다.

```python
age = 15

if age >= 18:
    print("성인입니다")

print("프로그램 종료")
```

```text
(base) C:\Users\guest\project> python if_skip.py
프로그램 종료
```

**정리**: `if 조건:` 문법은 조건이 참일 때만 들여쓰기된 블록을 실행하며, 조건이 거짓이면 블록 전체를 건너뛰고 다음 코드로 흐름이 이어진다.

## 비교 연산자

조건문에는 두 값을 비교해 참/거짓을 판단하는 **비교 연산자**가 자주 쓰인다.

| 연산자 | 의미 | 예시 | 결과 |
|--------|------|------|------|
| `>` | 크다 | `5 > 3` | `True` |
| `<` | 작다 | `5 < 3` | `False` |
| `>=` | 크거나 같다 | `5 >= 5` | `True` |
| `<=` | 작거나 같다 | `5 <= 4` | `False` |
| `==` | 같다 | `5 == 5` | `True` |
| `!=` | 다르다 | `5 != 3` | `True` |
| `is` | 같은 객체인가 | `a is b` | 객체 동일성 |
| `in` | 포함되어 있는가 | `3 in [1, 2, 3]` | `True` |

```python
>>> print(5 > 3, 5 < 3, 5 == 5, 5 != 3)
True False True True
```

**== 와 is의 차이**

`==`는 두 값이 **같은 값인지**를 비교하고, `is`는 두 변수가 **같은 객체(메모리상 같은 곳)를 가리키는지**를 비교한다.

```python
>>> a = [1, 2, 3]
>>> b = [1, 2, 3]
>>> print(a == b)
True
>>> print(a is b)
False
```

- `a`와 `b`는 값은 똑같은 `[1, 2, 3]`이지만, 서로 다른 리스트 객체로 각각 만들어졌으므로 `is`로 비교하면 `False`다.
- `None`과 비교할 때는 관용적으로 `== None`이 아니라 `is None`을 사용한다. `None`은 프로그램 전체에서 단 하나만 존재하는 특수한 객체이기 때문이다.

```python
>>> value = None
>>> print(value is None)
True
```

**in 연산자로 포함 여부 확인하기**

```python
>>> fruits = ["사과", "바나나", "딸기"]
>>> print("바나나" in fruits)
True
>>> print("포도" in fruits)
False
```

- `in`은 리스트뿐 아니라 문자열, 딕셔너리의 키 등 값을 포함하고 있는지 확인할 수 있는 모든 자료형에 사용할 수 있다.

```python
>>> print("사과" in "나는 사과를 좋아한다")
True
```

**정리**: 비교 연산자는 `> < >= <= == !=`로 두 값의 대소·동등 관계를, `is`로 두 변수가 같은 객체인지(주로 `None` 비교에 사용), `in`으로 어떤 값이 리스트·문자열 등에 포함되어 있는지를 판단하며, 이 연산 결과는 모두 `True` 또는 `False`인 `bool` 값이다.

## 불리언 연산자 (and · or · not)

여러 조건을 조합할 때는 **불리언 연산자** `and`, `or`, `not`을 사용한다.

| 연산자 | 의미 |
|--------|------|
| `and` | 양쪽 모두 참이어야 참 |
| `or` | 하나라도 참이면 참 |
| `not` | 참/거짓을 반대로 뒤집는다 |

```python
age = 20
has_ticket = True

if age >= 18 and has_ticket:
    print("입장 가능합니다")
```

```text
(base) C:\Users\guest\project> python and_test.py
입장 가능합니다
```

```python
>>> print(True and False)
False
>>> print(True or False)
True
>>> print(not True)
False
```

**단락 평가(short-circuit evaluation)**

```python
def check():
    print("check() 호출됨")
    return True

print(False and check())
print(True or check())
```

```text
(base) C:\Users\guest\project> python short_circuit.py
False
True
```

- `False and check()`는 왼쪽이 이미 `False`이므로 `and` 전체 결과가 무조건 `False`가 될 것이 확정되어, 오른쪽의 `check()`는 아예 호출되지 않는다.
- 마찬가지로 `True or check()`는 왼쪽이 이미 `True`이므로 오른쪽을 평가할 필요 없이 `True`로 결정되어 `check()`가 호출되지 않는다.
- 이런 동작을 **단락 평가**라고 하며, 뒤 조건의 함수 호출 비용이 크거나 부작용이 있는 코드라면 이 순서를 의도적으로 활용하기도 한다.

**정리**: `and`는 두 조건이 모두 참일 때, `or`는 둘 중 하나라도 참일 때 전체가 참이 되며 `not`은 참/거짓을 뒤집고, 파이썬은 `and`/`or`의 왼쪽 값만으로 전체 결과가 이미 결정되면 오른쪽 조건을 아예 평가하지 않는 단락 평가를 적용한다.

## elif와 else

**else**는 `if` 조건이 거짓일 때 실행할 블록을, **elif**는 첫 조건이 거짓일 때 추가로 검사할 다른 조건을 지정한다.

```python
score = 75

if score >= 90:
    print("A등급")
elif score >= 80:
    print("B등급")
elif score >= 70:
    print("C등급")
else:
    print("D등급")
```

```text
(base) C:\Users\guest\project> python elif_test.py
C등급
```

- 조건은 위에서부터 순서대로 검사하며, 처음으로 참이 되는 조건의 블록만 실행하고 나머지는 모두 건너뛴다.
- `score = 75`는 `score >= 90`(거짓), `score >= 80`(거짓), `score >= 70`(참)까지 검사한 뒤 "C등급"을 출력하고 이후 `else`는 검사조차 하지 않는다.
- `else`는 앞의 모든 조건이 거짓일 때만 실행되며 자체적인 조건식을 가지지 않는다.

**정리**: `elif`는 앞 조건이 거짓일 때 추가로 검사할 조건을 연결하고 `else`는 모든 조건이 거짓일 때 실행되는 마지막 대안이며, `if-elif-else` 체인은 위에서부터 순서대로 검사해 처음 참이 되는 블록 하나만 실행하고 나머지는 모두 건너뛴다.

## 다중 if문 vs if-elif-else 차이

겉보기에 비슷해 보이는 두 가지 작성 방식은 동작이 명확히 다르다.

**다중 if문 (독립적인 검사)**

```python
score = 95

if score >= 90:
    print("A등급입니다")
if score >= 80:
    print("우수한 성적입니다")
if score >= 70:
    print("합격입니다")
```

```text
(base) C:\Users\guest\project> python multi_if.py
A등급입니다
우수한 성적입니다
합격입니다
```

- 독립된 `if` 문 세 개는 각각 별개로 조건을 검사하므로, 조건을 모두 만족하면 세 블록이 전부 실행된다.

**if-elif-else 체인 (택일)**

```python
score = 95

if score >= 90:
    print("A등급입니다")
elif score >= 80:
    print("우수한 성적입니다")
elif score >= 70:
    print("합격입니다")
```

```text
(base) C:\Users\guest\project> python elif_chain.py
A등급입니다
```

- `elif`로 연결된 체인은 처음 참이 되는 조건 하나만 실행하고 나머지는 검사조차 하지 않으므로, 조건을 여러 개 만족해도 딱 한 블록만 실행된다.

**어느 쪽을 써야 하는가**

- 조건들이 서로 독립적이고 여러 개가 동시에 해당될 수 있으며 모두 실행되어야 한다면 다중 `if`문을 사용한다.
- 조건들이 서로 배타적이어서(하나만 해당) 등급을 나누듯 택일해야 한다면 `if-elif-else`를 사용한다. 등급 판정처럼 성능(불필요한 조건 검사 생략) 면에서도 `elif` 체인이 더 유리하다.

**정리**: 다중 `if`문은 각 조건을 독립적으로 검사해 해당하는 조건 블록을 모두 실행하지만, `if-elif-else` 체인은 조건을 순서대로 검사하다 처음 참이 되는 블록 하나만 실행하고 나머지는 건너뛰므로, 조건이 서로 배타적인 택일 상황에는 `elif` 체인을, 독립적으로 여러 개가 동시에 실행되어야 하는 상황에는 다중 `if`문을 사용해야 한다.

## while 반복문

**while** 반복문은 조건이 참인 동안 블록을 계속 반복 실행한다.

```python
count = 1

while count <= 5:
    print(count)
    count += 1
```

```text
(base) C:\Users\guest\project> python while_basic.py
1
2
3
4
5
```

- `count <= 5`가 참인 동안 블록이 반복되며, 블록 안에서 `count += 1`로 조건에 영향을 주는 변수를 갱신해야 언젠가 조건이 거짓이 되어 반복이 끝난다.
- 조건을 거짓으로 만드는 코드가 빠지면 **무한 루프**에 빠지므로, `while` 사용 시에는 종료 조건이 반드시 갱신되는지 확인해야 한다.

**break로 반복 중간에 빠져나오기**

```python
count = 1

while True:
    if count > 3:
        break
    print(count)
    count += 1
```

```text
(base) C:\Users\guest\project> python while_break.py
1
2
3
```

- `while True:`는 조건이 항상 참이므로 그 자체로는 무한히 반복하지만, 내부에서 특정 조건에 `break`를 만나면 즉시 반복문을 빠져나온다. 사용자 입력을 계속 받다가 특정 명령이 들어오면 종료하는 패턴에서 자주 쓰인다.

**정리**: `while 조건:`은 조건이 참인 동안 블록을 계속 반복하며, 블록 안에서 조건에 영향을 주는 변수를 갱신하지 않으면 무한 루프에 빠지고, `break`를 사용하면 조건과 무관하게 특정 시점에 즉시 반복문을 빠져나올 수 있다.

## for 반복문

**for** 반복문은 리스트, 문자열, `range()` 등 순회 가능한 대상의 값을 하나씩 꺼내며 반복한다.

**range()로 정해진 횟수만큼 반복하기**

```python
for i in range(5):
    print(i)
```

```text
(base) C:\Users\guest\project> python for_range.py
0
1
2
3
4
```

- `range(5)`는 `0`부터 `4`까지(5는 제외) 다섯 개의 숫자를 순서대로 만들어낸다. 슬라이싱과 마찬가지로 끝 값은 포함되지 않는다.
- `range(시작, 끝, 간격)` 형태로 범위와 간격을 지정할 수도 있다.

```python
>>> for i in range(2, 10, 2):
...     print(i)
...
2
4
6
8
```

**리스트를 값으로 순회하기**

```python
fruits = ["사과", "바나나", "딸기"]

for fruit in fruits:
    print(fruit)
```

```text
(base) C:\Users\guest\project> python for_list.py
사과
바나나
딸기
```

**인덱스가 필요할 때: range(len())과 enumerate()**

```python
fruits = ["사과", "바나나", "딸기"]

for i in range(len(fruits)):
    print(i, fruits[i])
```

```text
(base) C:\Users\guest\project> python for_index.py
0 사과
1 바나나
2 딸기
```

- 위 방식도 동작하지만, 파이썬은 인덱스와 값을 함께 꺼낼 수 있는 더 간결한 **enumerate()** 함수를 제공한다.

```python
fruits = ["사과", "바나나", "딸기"]

for i, fruit in enumerate(fruits):
    print(i, fruit)
```

```text
(base) C:\Users\guest\project> python for_enumerate.py
0 사과
1 바나나
2 딸기
```

- `enumerate()`는 매 반복마다 `(인덱스, 값)` 쌍을 만들어주므로, `i, fruit = ...`처럼 두 변수로 바로 나누어 받을 수 있다. `range(len())`보다 더 파이썬다운(관용적인) 작성법으로 널리 쓰인다.

**zip()으로 여러 리스트를 동시에 순회하기**

```python
names = ["홍길동", "김철수", "이영희"]
scores = [90, 85, 95]

for name, score in zip(names, scores):
    print(f"{name}: {score}점")
```

```text
(base) C:\Users\guest\project> python for_zip.py
홍길동: 90점
김철수: 85점
이영희: 95점
```

- `zip()`은 여러 리스트를 같은 인덱스끼리 짝지어 튜플로 묶어주며, 리스트 길이가 다르면 더 짧은 쪽 길이에 맞춰 순회가 끝난다.

**정리**: `for`는 `range()`로 만든 숫자 범위나 리스트·문자열 같은 순회 가능한 대상의 값을 하나씩 꺼내 반복하며, 인덱스와 값을 함께 다루고 싶다면 `range(len())`보다 `enumerate()`가 더 간결하고, 여러 리스트를 같은 인덱스끼리 동시에 순회하고 싶다면 `zip()`을 사용한다.

## 중첩 반복문과 break의 범위

반복문 안에 또 다른 반복문을 넣는 **중첩 반복문**은 2차원 리스트를 다룰 때(PY-04에서 이미 살펴본 것처럼) 자주 사용된다.

```python
for i in range(1, 4):
    for j in range(1, 4):
        print(f"({i}, {j})", end=" ")
    print()
```

```text
(base) C:\Users\guest\project> python nested_loop.py
(1, 1) (1, 2) (1, 3)
(2, 1) (2, 2) (2, 3)
(3, 1) (3, 2) (3, 3)
```

- 바깥 반복문이 한 번 돌 때마다 안쪽 반복문이 처음부터 끝까지 전부 실행되므로, 전체 반복 횟수는 바깥 횟수 × 안쪽 횟수가 된다.

**break는 자신이 속한 가장 안쪽 반복문만 빠져나온다**

```python
for i in range(1, 4):
    for j in range(1, 4):
        if j == 2:
            break
        print(f"({i}, {j})", end=" ")
    print()
```

```text
(base) C:\Users\guest\project> python nested_break.py
(1, 1)
(2, 1)
(3, 1)
```

- 안쪽 반복문에서 `j == 2`일 때 `break`를 실행하면 **안쪽 반복문만** 종료되고, 바깥 반복문은 영향을 받지 않아 계속 다음 `i` 값으로 진행된다.
- 바깥 반복문까지 한 번에 종료하고 싶다면, 별도의 상태 변수(플래그)를 두고 바깥 반복문에서도 조건을 검사해 `break`하는 방식을 추가로 조합해야 한다.

```python
found = False
for i in range(1, 4):
    for j in range(1, 4):
        if i == 2 and j == 2:
            found = True
            break
    if found:
        break
    print(f"i={i} 탐색 완료")
```

```text
(base) C:\Users\guest\project> python nested_break_flag.py
i=1 탐색 완료
```

- 안쪽에서 `found = True`로 표시하고 `break`한 뒤, 바깥 반복문에서 `if found: break`로 한 번 더 확인해 바깥 반복문까지 종료시키는 방식이다.

**정리**: 중첩 반복문은 바깥 반복문 한 번마다 안쪽 반복문이 전체 실행되는 구조이며, `break`는 자신이 속한 가장 안쪽 반복문만 종료시키므로 바깥 반복문까지 함께 종료하려면 플래그 변수를 두고 바깥 반복문에서도 조건을 검사해 추가로 `break`하는 패턴을 사용해야 한다.

## 조건문·반복문 활용 실전 예제

**예제 1) 홀수만 걸러내기**

```python
def filter_odd(numbers):
    result = []
    for n in numbers:
        if n % 2 != 0:
            result.append(n)
    return result

print(filter_odd([1, 2, 3, 4, 5, 6, 7, 8, 9, 10]))
```

```text
(base) C:\Users\guest\project> python odd_filter.py
[1, 3, 5, 7, 9]
```

- `n % 2 != 0`으로 각 숫자를 2로 나눈 나머지가 0이 아닌지(홀수인지) 검사하여, 조건을 만족하는 값만 새 리스트에 담아 반환한다.

**예제 2) 약수 구하기**

```python
def find_divisors(n):
    divisors = []
    for i in range(1, n + 1):
        if n % i == 0:
            divisors.append(i)
    return divisors

print(find_divisors(36))
```

```text
(base) C:\Users\guest\project> python divisors.py
[1, 2, 3, 4, 6, 9, 12, 18, 36]
```

- `1`부터 `n`까지 모든 숫자로 `n`을 나눠보면서 나머지가 `0`이면(나누어떨어지면) 약수로 판정해 리스트에 담는다.

**예제 3) 최대공약수 구하기 (유클리드 호제법)**

```python
def gcd(a, b):
    while b != 0:
        a, b = b, a % b
    return a

print(gcd(48, 18))
print(gcd(100, 75))
```

```text
(base) C:\Users\guest\project> python gcd.py
6
25
```

- 유클리드 호제법은 "두 수 `a`, `b`의 최대공약수는 `b`와 `a % b`의 최대공약수와 같다"는 성질을 반복 적용한다.
- `a, b = b, a % b`는 두 변수를 동시에 교체하는 파이썬의 다중 대입 문법으로, `b`가 `0`이 될 때까지 이 과정을 반복하면 마지막에 남은 `a`가 최대공약수다.

**정리**: 홀수 필터링은 `for`와 `if`를 조합해 조건을 만족하는 값만 골라 새 리스트를 만드는 가장 기본적인 패턴이고, 약수 구하기는 범위 전체를 순회하며 나머지가 0인 경우를 찾는 패턴이며, 최대공약수는 `while`과 나머지 연산을 반복 적용하는 유클리드 호제법으로 구현할 수 있다는 것을 확인했다. 세 예제 모두 이 문서에서 다룬 반복문과 조건문을 함수로 감싸 재사용 가능하게 만드는 흐름을 보여준다.

## pass 문

파이썬은 `if`, `for`, `while`, 함수 정의 등에서 콜론(`:`) 뒤에 반드시 들여쓰기된 블록이 있어야 한다. 아직 내용을 정하지 못했지만 문법상 블록이 필요한 자리에는 **pass** 문을 사용한다.

```python
score = 85

if score >= 90:
    pass   # A등급 처리 로직은 나중에 작성 예정
else:
    print("일단 A등급이 아닙니다")
```

```text
(base) C:\Users\guest\project> python pass_placeholder.py
일단 A등급이 아닙니다
```

- `pass`는 "아무 동작도 하지 않는다"는 의미의 문장이다. 블록을 비워두면 `IndentationError`가 발생하지만, `pass` 하나만 적어두면 문법 오류 없이 그 블록을 통과시킬 수 있다.

**빈 함수·빈 반복문의 자리표시자로 사용하기**

```python
def todo_later():
    pass

for i in range(3):
    pass   # 반복 자체는 하지만 아직 할 일이 없다

print("함수 정의와 반복문 모두 문법 오류 없이 통과했다")
```

```text
(base) C:\Users\guest\project> python pass_stub.py
함수 정의와 반복문 모두 문법 오류 없이 통과했다
```

- 함수나 클래스의 뼈대만 먼저 잡아두고 본문 구현은 나중으로 미루고 싶을 때, `pass`를 넣어두면 일단 코드 전체가 실행 가능한 상태를 유지할 수 있다. `todo_later()`를 호출해도 아무 일도 일어나지 않을 뿐 오류는 나지 않는다.

**정리**: `pass`는 문법상 들여쓰기 블록이 필요하지만 아직 실행할 내용이 없을 때 자리만 채워주는 문장으로, 빈 블록을 남겨두면 발생하는 `IndentationError`를 피하면서 함수·조건문·반복문의 뼈대를 먼저 작성하고 실제 구현은 나중으로 미루는 용도로 자주 쓰인다.

## and·or의 반환값

`and`와 `or`는 결과로 항상 `True`/`False`만 돌려준다고 생각하기 쉽지만, 실제로는 **피연산자 중 하나를 그대로** 반환한다.

```python
print(3 and 5)
print(0 and 5)
print(3 or 5)
print(0 or 5)
print("" or "기본값")
```

```text
(base) C:\Users\guest\project> python and_or_value.py
5
0
3
5
기본값
```

- `and`는 왼쪽 값이 거짓(falsy)이면 왼쪽 값을 그대로 반환하고, 왼쪽이 참(truthy)이면 오른쪽 값을 반환한다. `3 and 5`는 `3`이 참이므로 오른쪽인 `5`가 그대로 반환되고, `0 and 5`는 `0`이 거짓이므로 왼쪽인 `0`이 그대로 반환된다.
- `or`는 왼쪽 값이 참이면 왼쪽 값을 그대로 반환하고, 왼쪽이 거짓이면 오른쪽 값을 반환한다. `3 or 5`는 `3`이 이미 참이므로 `3`이 그대로 반환되고 `5`는 평가조차 되지 않으며, `"" or "기본값"`은 빈 문자열이 거짓이므로 오른쪽인 `"기본값"`이 반환된다.

**a or 기본값 패턴으로 기본값 지정하기**

```python
def greet(name=None):
    display_name = name or "손님"
    print(f"환영합니다, {display_name}님!")

greet("김철수")
greet()
greet("")
```

```text
(base) C:\Users\guest\project> python or_default.py
환영합니다, 김철수님!
환영합니다, 손님님!
환영합니다, 손님님!
```

- `name or "손님"`은 `name`이 참(값이 있는 문자열)이면 `name`을, `name`이 거짓(`None`이나 빈 문자열)이면 `"손님"`을 대신 사용하는 관용적인 표현이다. `if name: display_name = name else: display_name = "손님"`과 같은 동작을 한 줄로 줄인 것이다.
- 다만 `name`에 빈 문자열처럼 "값은 있지만 falsy한" 데이터가 들어올 수 있는 경우에는 `""`도 `"손님"`으로 대체되어버리므로, 의도한 동작인지 주의해서 사용해야 한다.

**정리**: `and`/`or`는 결과를 항상 `True`/`False`로 새로 만드는 것이 아니라 실제로 평가한 피연산자 중 하나를 그대로 반환하며, `and`는 왼쪽이 거짓이면 왼쪽을, 참이면 오른쪽을 반환하고 `or`는 왼쪽이 참이면 왼쪽을, 거짓이면 오른쪽을 반환하므로 이 성질을 이용한 `값 or 기본값` 패턴은 값이 없을 때(`None`, 빈 문자열 등) 대신 쓸 기본값을 한 줄로 지정하는 관용적인 코드로 널리 쓰인다.

## 실습 예제 (EX1~EX4)

**EX1) 성적에 따른 등급 판정 (elif 체인)**
- 점수를 입력받아(변수로 지정) `if-elif-else`로 A/B/C/D 등급을 판단한다.

```python
# ex01_grade.py
def get_grade(score):
    if score >= 90:
        return "A"
    elif score >= 80:
        return "B"
    elif score >= 70:
        return "C"
    else:
        return "D"

for score in [95, 82, 71, 40]:
    print(f"{score}점 -> {get_grade(score)}등급")
```

```text
(base) C:\Users\guest\project> python ex01_grade.py
95점 -> A등급
82점 -> B등급
71점 -> C등급
40점 -> D등급
```

**EX2) 1부터 100까지 3과 5의 배수 합 구하기 (FizzBuzz 변형)**
- `while` 반복문으로 1부터 100까지 순회하며 3 또는 5의 배수인 값만 모두 계산한다.

```python
# ex02_multiples_sum.py
total = 0
n = 1

while n <= 100:
    if n % 3 == 0 or n % 5 == 0:
        total += n
    n += 1

print(f"3 또는 5의 배수 합: {total}")
```

```text
(base) C:\Users\guest\project> python ex02_multiples_sum.py
3 또는 5의 배수 합: 2418
```

**EX3) 구구단 2단~9단을 중첩 반복문으로 출력**
- `for` 중첩 반복문으로 2단부터 9단까지 구구단을 출력한다.

```python
# ex03_multiplication_table.py
for dan in range(2, 10):
    for i in range(1, 10):
        print(f"{dan} x {i} = {dan * i}")
    print("-" * 15)
```

```text
(base) C:\Users\guest\project> python ex03_multiplication_table.py
2 x 1 = 2
2 x 2 = 4
...
2 x 9 = 18
---------------
...
9 x 9 = 81
---------------
```

**EX4) 리스트에서 특정 값을 찾으면 즉시 중단 (break 활용)**
- 이름 리스트를 순회하다가 찾는 이름이 나오면 "찾았습니다"를 출력하고 즉시 반복을 중단하는 과정을 살펴본다.

```python
# ex04_search_break.py
names = ["홍길동", "김철수", "이영희", "박민수"]
target = "이영희"

for i, name in enumerate(names):
    if name == target:
        print(f"{i}번 인덱스에서 {target}을(를) 찾았습니다.")
        break
else:
    print(f"{target}을(를) 찾지 못했습니다.")
```

```text
(base) C:\Users\guest\project> python ex04_search_break.py
2번 인덱스에서 이영희을(를) 찾았습니다.
```

- `for-else` 구문은 `for` 반복문이 `break` 없이 정상적으로 끝까지 순회를 마쳤을 때만 `else` 블록을 실행한다. 중간에 `break`로 빠져나오면 `else` 블록은 건너뛴다는 점에서, 찾기 실패를 처리하는 관용적인 패턴으로 쓰인다.

**정리**: EX1~EX4는 `elif` 체인으로 등급을 판정하고, `while`로 조건에 맞는 값을 누적하고, 중첩 `for`로 구구단표를 만들고, `break`(와 `for-else`)로 탐색을 조기 종료하는 방법까지 이 문서에서 다룬 조건문·반복문의 핵심 패턴을 코드로 직접 확인해보는 예제이며, 다음 문서에서는 데이터와 동작을 하나로 묶는 클래스(class)를 다룬다.

[Python 04 — 리스트와 딕셔너리](04-lists-dictionaries.md) · [Python 06 — 클래스](06-classes.md)

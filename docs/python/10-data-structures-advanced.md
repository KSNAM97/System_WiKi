# Python 10 — 자료구조 심화

## 리스트를 스택으로 사용하기

**스택(stack)**은 마지막에 넣은 데이터가 가장 먼저 나오는(LIFO, Last In First Out) 자료구조다. 파이썬 리스트는 `append()`와 `pop()`만으로 별도의 자료형 없이 스택처럼 사용할 수 있다.

```python
stack = []
stack.append("접시1")
stack.append("접시2")
stack.append("접시3")
print(stack)

print(stack.pop())
print(stack.pop())
print(stack)
```

```text
(base) C:\Users\guest\project> python stack_basic.py
['접시1', '접시2', '접시3']
접시3
접시2
['접시1']
```

- `append()`는 리스트의 맨 끝에 요소를 추가하고, `pop()`은 인덱스를 지정하지 않으면 맨 끝 요소를 꺼내면서 제거한다. 둘 다 리스트의 끝에서만 동작하므로 항상 O(1) 시간에 처리되어 스택 용도로 효율적이다.
- 접시를 쌓아 올리고 위에서부터 꺼내 쓰는 모습과 같아서, "마지막에 넣은 것이 가장 먼저 나온다"는 LIFO 원리를 그대로 보여준다.

**괄호 짝 맞추기 예시**

```python
def is_balanced(text):
    stack = []
    pairs = {")": "(", "]": "[", "}": "{"}
    for ch in text:
        if ch in "([{":
            stack.append(ch)
        elif ch in ")]}":
            if not stack or stack.pop() != pairs[ch]:
                return False
    return not stack

print(is_balanced("(a[b]{c})"))
print(is_balanced("(a[b)c]"))
```

```text
(base) C:\Users\guest\project> python bracket_check.py
True
False
```

**정리**: 파이썬 리스트는 별도의 스택 클래스 없이 `append()`로 데이터를 쌓고 `pop()`으로 맨 위(끝)부터 꺼내는 방식만으로 LIFO 구조의 스택 역할을 그대로 수행할 수 있으며, 두 연산 모두 리스트의 끝에서 처리되므로 성능상으로도 스택 용도에 적합하다.

## 리스트를 큐로 사용하기 — deque

**큐(queue)**는 먼저 넣은 데이터가 먼저 나오는(FIFO, First In First Out) 자료구조다. 리스트로 큐를 흉내낼 수는 있지만, 이때는 성능 문제가 발생한다.

```python
queue = []
queue.append("손님1")
queue.append("손님2")
queue.append("손님3")

print(queue.pop(0))
print(queue.pop(0))
print(queue)
```

```text
(base) C:\Users\guest\project> python queue_with_list.py
손님1
손님2
['손님3']
```

- `queue.pop(0)`으로 맨 앞 요소를 꺼내면 FIFO 동작 자체는 구현되지만, 리스트에서 맨 앞 요소를 제거하면 그 뒤의 모든 요소가 한 칸씩 앞으로 당겨져야 하므로 요소 개수에 비례한 시간(O(n))이 걸린다. 데이터가 많아질수록 이 작업이 점점 느려진다.

**collections.deque로 해결하기**

```python
from collections import deque

queue = deque()
queue.append("손님1")
queue.append("손님2")
queue.append("손님3")
print(queue)

print(queue.popleft())
print(queue.popleft())
print(queue)
```

```text
(base) C:\Users\guest\project> python queue_with_deque.py
deque(['손님1', '손님2', '손님3'])
손님1
손님2
deque(['손님3'])
```

- `deque`(double-ended queue)는 `collections` 모듈이 제공하는 자료구조로, 양쪽 끝(`앞`과 `뒤`) 모두에서 O(1) 시간에 추가·제거가 가능하도록 설계되어 있다.
- `append()`/`pop()`은 오른쪽(뒤)에서, `appendleft()`/`popleft()`는 왼쪽(앞)에서 동작한다. 큐로 사용할 때는 `append()`로 넣고 `popleft()`로 꺼내면 된다.

```python
from collections import deque

dq = deque([1, 2, 3])
dq.appendleft(0)
dq.append(4)
print(dq)
```

```text
(base) C:\Users\guest\project> python deque_both_ends.py
deque([0, 1, 2, 3, 4])
```

**정리**: 리스트로도 `pop(0)`을 사용해 큐를 흉내낼 수 있지만 맨 앞에서 요소를 뺄 때마다 나머지 요소를 앞으로 당겨야 해 데이터가 많을수록 느려지는 반면, `collections.deque`는 양쪽 끝 모두에서 O(1)로 추가·제거가 가능하도록 설계되어 있어 큐(FIFO)나 양방향 자료구조가 필요할 때는 리스트 대신 `deque`를 사용하는 것이 성능상 올바른 선택이다.

## 리스트 컴프리헨션

**리스트 컴프리헨션(list comprehension)**은 `for` 반복문으로 새 리스트를 만드는 코드를 한 줄로 축약하는 문법이다.

```python
squares = []
for x in range(1, 6):
    squares.append(x ** 2)
print(squares)

squares_comp = [x ** 2 for x in range(1, 6)]
print(squares_comp)
```

```text
(base) C:\Users\guest\project> python listcomp_basic.py
[1, 4, 9, 16, 25]
[1, 4, 9, 16, 25]
```

- `[표현식 for 변수 in 반복가능한객체]` 형태로 작성하며, 반복문과 `append()`를 명시적으로 쓰지 않고도 동일한 결과를 만들어낸다.

**조건부 필터링**

```python
numbers = range(1, 21)
even_numbers = [n for n in numbers if n % 2 == 0]
print(even_numbers)
```

```text
(base) C:\Users\guest\project> python listcomp_filter.py
[2, 4, 6, 8, 10, 12, 14, 16, 18, 20]
```

- `[표현식 for 변수 in 반복가능한객체 if 조건]` 형태로 뒤에 `if`를 붙이면, 조건을 만족하는 요소만 걸러서 새 리스트를 만든다.

**if-else를 표현식에 포함하기**

```python
numbers = range(1, 6)
labels = ["짝수" if n % 2 == 0 else "홀수" for n in numbers]
print(labels)
```

```text
(base) C:\Users\guest\project> python listcomp_ifelse.py
['홀수', '짝수', '홀수', '짝수', '홀수']
```

- 뒤쪽의 `if 조건`은 요소를 **걸러내는** 필터이고, 표현식 자리(맨 앞)의 `A if 조건 else B`는 요소마다 **다른 값으로 변환**하는 것이라는 차이를 구분해야 한다.

**정리**: 리스트 컴프리헨션은 `[표현식 for 변수 in 반복가능한객체]`(필요시 `if 조건`까지)로 반복문과 `append()` 호출을 한 줄로 압축하는 문법이며, 단순히 값을 변환하거나 조건에 맞는 요소만 골라 새 리스트를 만드는 상황에서는 일반 `for` 문보다 짧고 파이썬다운(pythonic) 코드로 여겨진다.

## 중첩 리스트 컴프리헨션

리스트 컴프리헨션 안에 또 다른 `for`를 중첩하면, 2차원 리스트를 다루거나 이중 반복문을 한 줄로 표현할 수 있다.

```python
matrix = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]

flattened = [num for row in matrix for num in row]
print(flattened)
```

```text
(base) C:\Users\guest\project> python nested_listcomp_flatten.py
[1, 2, 3, 4, 5, 6, 7, 8, 9]
```

- `for row in matrix for num in row`는 바깥쪽부터 안쪽 순서로 읽는다. 즉 "matrix의 각 row에 대해, 다시 row의 각 num에 대해"로 해석되며, 다음 이중 반복문과 동일하다.

```python
flattened = []
for row in matrix:
    for num in row:
        flattened.append(num)
```

**2차원 리스트 만들기 (중첩 컴프리헨션을 대괄호 안에 중첩)**

```python
grid = [[r * 3 + c for c in range(3)] for r in range(3)]
for row in grid:
    print(row)
```

```text
(base) C:\Users\guest\project> python nested_listcomp_build.py
[0, 1, 2]
[3, 4, 5]
[6, 7, 8]
```

- 바깥쪽 `[... for r in range(3)]`이 3개의 행을 만들고, 그 안의 `[r * 3 + c for c in range(3)]`이 각 행의 3개 열을 만든다. PY-04에서 반복문으로 만들었던 2차원 리스트를 컴프리헨션으로 표현한 것이다.

**정리**: 리스트 컴프리헨션 안에 `for`를 여러 개 이어 쓰면 이중(또는 그 이상) 반복문을 한 줄로 표현할 수 있으며, `for ... for ...`를 나열하면 2차원 리스트를 1차원으로 평탄화(flatten)할 수 있고, 대괄호 안에 컴프리헨션을 중첩하면 반대로 2차원 리스트를 생성할 수 있다. 다만 중첩 단계가 깊어질수록 가독성이 떨어지므로, 2단계를 넘어가면 일반 `for` 문으로 풀어 쓰는 것이 낫다.

## 튜플과 시퀀스

PY-04에서 리스트를 다뤘다면, **튜플(tuple)**은 리스트와 비슷하게 순서 있는 값의 모음이지만 한 번 만들면 내용을 바꿀 수 없는 **불변(immutable)** 자료형이다.

```python
point = (3, 4)
print(point[0], point[1])

point[0] = 10
```

```text
(base) C:\Users\guest\project> python tuple_immutable.py
3 4
Traceback (most recent call last):
  File "tuple_immutable.py", line 4, in <module>
    point[0] = 10
TypeError: 'tuple' object does not support item assignment
```

- 인덱싱(`point[0]`)으로 값을 읽는 것은 리스트와 동일하지만, 요소를 변경하려고 하면 `TypeError`가 발생한다. 좌표나 RGB 색상값처럼 "한번 정해지면 바뀌지 않아야 하는" 데이터를 표현할 때 리스트 대신 튜플을 쓰면 의도치 않은 수정을 막을 수 있다.

**튜플 언패킹**

```python
point = (3, 4)
x, y = point
print(f"x={x}, y={y}")

name, age, *rest = ("김철수", 25, "서울", "개발자")
print(name, age, rest)
```

```text
(base) C:\Users\guest\project> python tuple_unpacking.py
x=3, y=4
김철수 25 ['서울', '개발자']
```

- `x, y = point`처럼 튜플의 각 요소를 변수 여러 개에 한 번에 나눠 담는 것을 **언패킹**이라고 한다. 변수 개수와 튜플의 길이가 정확히 일치해야 하며, PY-09에서 다룬 `*rest`를 함께 쓰면 나머지 요소를 리스트로 묶어 받을 수 있다.
- 함수가 값을 여러 개 반환할 때도 사실은 튜플을 반환하고 이를 언패킹해서 받는 것이다.

```python
def min_max(numbers):
    return min(numbers), max(numbers)

lowest, highest = min_max([4, 2, 9, 1, 7])
print(lowest, highest)
```

```text
(base) C:\Users\guest\project> python tuple_multi_return.py
1 9
```

**정리**: 튜플은 리스트와 마찬가지로 순서가 있는 여러 값을 담지만 한 번 생성되면 내용을 바꿀 수 없는 불변 시퀀스이며, `x, y = point`처럼 여러 변수에 한 번에 값을 나눠 담는 언패킹 문법과 함께 자주 쓰이고, 함수가 여러 값을 반환할 때(`return a, b`) 실제로는 튜플 하나를 반환한 뒤 호출부에서 언패킹하는 방식으로 동작한다.

## 집합(set)

**집합(set)**은 중복을 허용하지 않고 순서가 없는 자료형으로, 데이터의 중복을 제거하거나 여러 그룹 간의 관계(합집합·교집합·차집합)를 계산할 때 사용한다.

```python
fruits = {"사과", "바나나", "사과", "딸기", "바나나"}
print(fruits)
print(len(fruits))
```

```text
(base) C:\Users\guest\project> python set_basic.py
{'사과', '바나나', '딸기'}
3
```

- 중괄호 `{}`로 값을 나열하면 집합이 만들어지며, 이때 중복된 값은 자동으로 하나만 남는다. 딕셔너리도 `{}`를 쓰지만 `키: 값` 쌍이 없으면 집합으로 해석된다는 점에 유의한다.
- 빈 집합은 `{}`가 아니라 `set()`으로 만들어야 한다. `{}`는 빈 딕셔너리로 해석되기 때문이다.

**리스트의 중복 제거하기**

```python
numbers = [1, 3, 2, 3, 1, 5, 2]
unique_numbers = list(set(numbers))
print(sorted(unique_numbers))
```

```text
(base) C:\Users\guest\project> python set_dedupe.py
[1, 2, 3, 5]
```

- 리스트를 `set()`으로 감싸면 중복이 제거된 집합이 되고, 다시 `list()`로 감싸면 리스트로 되돌릴 수 있다. 단, 집합은 순서를 보장하지 않으므로 원래 순서가 중요하다면 이 방법을 사용해선 안 된다.

**합집합, 교집합, 차집합**

```python
a = {1, 2, 3, 4}
b = {3, 4, 5, 6}

print("합집합:", a | b)
print("교집합:", a & b)
print("차집합(a-b):", a - b)
print("대칭차집합:", a ^ b)
```

```text
(base) C:\Users\guest\project> python set_operations.py
합집합: {1, 2, 3, 4, 5, 6}
교집합: {3, 4}
차집합(a-b): {1, 2}
대칭차집합: {1, 2, 5, 6}
```

- `|`(합집합)는 둘 중 하나에라도 속한 모든 요소, `&`(교집합)는 둘 다에 속한 요소, `-`(차집합)는 `a`에는 있지만 `b`에는 없는 요소, `^`(대칭차집합)는 둘 중 한쪽에만 속한 요소를 반환한다. 메서드 형태(`a.union(b)`, `a.intersection(b)`, `a.difference(b)`)로도 동일하게 사용할 수 있다.

**정리**: 집합은 중복을 허용하지 않고 순서가 없는 자료형으로, 리스트를 `set()`으로 감싸는 것만으로 손쉽게 중복을 제거할 수 있으며, `|`(합집합)·`&`(교집합)·`-`(차집합)·`^`(대칭차집합) 연산자로 두 그룹 간의 관계를 한 줄로 계산할 수 있어 태그·권한·회원 명단처럼 겹치거나 구분되는 그룹을 다룰 때 유용하다.

## 딕셔너리 컴프리헨션

리스트 컴프리헨션과 같은 원리로, 딕셔너리도 `{키: 값 for 변수 in 반복가능한객체}` 형태로 한 줄에 만들 수 있다.

```python
numbers = range(1, 6)
squares = {n: n ** 2 for n in numbers}
print(squares)
```

```text
(base) C:\Users\guest\project> python dictcomp_basic.py
{1: 1, 2: 4, 3: 9, 4: 16, 5: 25}
```

**조건부 필터링과 기존 딕셔너리 변형하기**

```python
prices = {"사과": 1500, "바나나": 800, "수박": 15000, "딸기": 6000}

cheap_items = {name: price for name, price in prices.items() if price < 5000}
print(cheap_items)

discounted = {name: int(price * 0.9) for name, price in prices.items()}
print(discounted)
```

```text
(base) C:\Users\guest\project> python dictcomp_filter_transform.py
{'사과': 1500, '바나나': 800}
{'사과': 1350, '바나나': 720, '수박': 13500, '딸기': 5400}
```

- `prices.items()`로 키와 값을 동시에 꺼내면서, `if` 조건으로 걸러내거나 값 부분의 표현식을 바꿔 새로운 딕셔너리를 만들 수 있다. 기존 딕셔너리를 직접 수정하지 않고 조건에 맞는 새 딕셔너리를 얻고 싶을 때 유용하다.

**정리**: 딕셔너리 컴프리헨션은 `{키표현식: 값표현식 for 변수 in 반복가능한객체}` 형태로 리스트 컴프리헨션과 동일한 문법 감각으로 딕셔너리를 생성하며, 기존 딕셔너리의 `items()`를 순회하면서 조건에 맞는 항목만 골라내거나 값을 가공한 새 딕셔너리를 만드는 데 자주 사용된다.

## 루프 테크닉 심화

PY-05에서 `enumerate()`(인덱스와 값을 함께 순회)와 `zip()`(여러 시퀀스를 나란히 순회)의 기초를 다뤘다. 여기서는 짧게 복습만 하고, `sorted()`와 `reversed()`를 중심으로 더 다룬다.

**enumerate(), zip() 짧은 복습**

```python
fruits = ["사과", "바나나", "딸기"]
prices = [1500, 800, 6000]

for i, fruit in enumerate(fruits, start=1):
    print(f"{i}. {fruit}")

for fruit, price in zip(fruits, prices):
    print(f"{fruit}: {price}원")
```

```text
(base) C:\Users\guest\project> python loop_recap.py
1. 사과
2. 바나나
3. 딸기
사과: 1500원
바나나: 800원
딸기: 6000원
```

**sorted()로 정렬하며 순회하기**

`sorted()`는 원본을 바꾸지 않고 정렬된 **새 리스트**를 반환한다는 점에서, 원본 리스트 자체를 정렬하는 `list.sort()`와 다르다.

```python
scores = {"김철수": 85, "이영희": 92, "박민수": 78}

for name, score in sorted(scores.items(), key=lambda x: x[1], reverse=True):
    print(f"{name}: {score}점")
```

```text
(base) C:\Users\guest\project> python loop_sorted.py
이영희: 92점
김철수: 85점
박민수: 78점
```

- `sorted(반복가능한객체, key=함수, reverse=불리언)` 형태로, `key`에 PY-09에서 다룬 람다를 넘겨 정렬 기준을 지정하고 `reverse=True`로 내림차순 정렬할 수 있다.
- 딕셔너리 자체는 순서 정렬 개념이 없으므로, 정렬된 순서로 순회하고 싶다면 `sorted(딕셔너리.items())`처럼 항목을 리스트로 뽑아 정렬해야 한다.

**reversed()로 역순 순회하기**

```python
numbers = [1, 2, 3, 4, 5]

for n in reversed(numbers):
    print(n, end=" ")
print()

for i in reversed(range(1, 6)):
    print(i, end=" ")
print()
```

```text
(base) C:\Users\guest\project> python loop_reversed.py
5 4 3 2 1
5 4 3 2 1
```

- `reversed()`는 시퀀스를 역순으로 순회하는 **이터레이터**를 반환한다. 원본 리스트를 뒤집어 새 리스트를 만드는 것이 아니라 순회 순서만 거꾸로 바꾸므로, `numbers[::-1]`(슬라이싱으로 복사본 생성)보다 메모리를 아낄 수 있다.
- `sorted()`와 `reversed()`를 조합하면 오름차순 정렬의 반대인 내림차순 순회도 만들 수 있지만, `sorted(..., reverse=True)`가 더 직접적이므로 보통 그쪽을 사용한다.

**정리**: `enumerate()`와 `zip()`이 각각 인덱스를 붙이거나 여러 시퀀스를 나란히 묶어 순회하는 도구라면, `sorted()`는 원본을 바꾸지 않고 정렬된 새 시퀀스를 만들어 원하는 기준(`key`)과 방향(`reverse`)으로 순회할 수 있게 해주고, `reversed()`는 시퀀스를 복사하지 않고 역순으로 순회하는 이터레이터를 제공하여, 상황에 맞게 조합하면 반복문 안에서 직접 인덱스를 계산하거나 리스트를 뒤집는 코드를 줄일 수 있다.

## 시퀀스 비교

리스트나 튜플 같은 시퀀스는 `<`, `>`, `==` 같은 비교 연산자로 서로 비교할 수 있으며, 이때 **사전식(lexicographic) 비교** 규칙을 따른다.

```python
print([1, 2, 3] < [1, 2, 4])
print([1, 2, 3] < [1, 2, 3, 0])
print([1, 2, 3] == [1, 2, 3])
print((1, 2) < (1, 3))
print("apple" < "banana")
```

```text
(base) C:\Users\guest\project> python sequence_compare.py
True
True
True
True
True
```

- 사전식 비교는 앞에서부터 요소를 하나씩 비교하다가, 값이 다른 첫 지점에서 승패가 결정된다. `[1, 2, 3]`과 `[1, 2, 4]`는 앞의 두 요소(`1`, `2`)가 같으므로 세 번째 요소(`3`과 `4`)로 비교가 결정되어 `[1, 2, 3]`이 더 작다고 판단된다.
- 길이가 다르고 앞부분이 완전히 같다면, 더 짧은 시퀀스가 더 작은 것으로 취급된다(`[1, 2, 3] < [1, 2, 3, 0]`). 이는 사전에서 `"cat"`이 `"catalog"`보다 앞에 오는 것과 같은 원리다.
- 문자열도 시퀀스이므로 같은 규칙이 적용되며, 글자를 하나씩 유니코드 코드값으로 비교한다.

**정리**: 리스트·튜플·문자열 같은 시퀀스는 앞 요소부터 차례로 비교하다가 값이 처음으로 달라지는 지점에서 대소가 결정되는 사전식 비교 규칙을 따르며, 모든 요소가 같고 길이만 다르면 더 짧은 쪽이 작은 것으로 취급되므로, 버전 번호(`(1, 2, 0) < (1, 3, 0)`)나 순위 비교처럼 여러 값을 한 번에 비교해야 하는 상황에서 튜플 비교를 활용하면 편리하다.

## bisect 모듈 — 정렬된 리스트 유지하기

이미 정렬되어 있는 리스트에 새 값을 넣을 때마다 매번 `sorted()`로 전체를 다시 정렬하는 것은 비효율적이다. `bisect` 모듈은 정렬된 리스트에 알맞은 위치를 빠르게 찾거나, 그 위치에 바로 삽입하는 기능을 제공한다.

```python
import bisect

scores = [60, 75, 90]

bisect.insort(scores, 82)
print(scores)

bisect.insort(scores, 55)
print(scores)
```

```text
(base) C:\Users\guest\project> python bisect_insort.py
[60, 75, 82, 90]
[55, 60, 75, 82, 90]
```

- `bisect.insort(정렬된리스트, 값)`은 리스트가 이미 정렬되어 있다는 전제 하에, 정렬 상태를 유지하면서 값을 알맞은 위치에 삽입한다. `append()` 후 다시 `sort()`를 호출하는 것보다 위치 탐색이 이진 탐색(binary search)으로 이루어져 더 효율적이다.

**bisect_left / bisect_right — 삽입 위치만 확인하기**

```python
import bisect

nums = [10, 20, 20, 20, 30]

print(bisect.bisect_left(nums, 20))
print(bisect.bisect_right(nums, 20))
```

```text
(base) C:\Users\guest\project> python bisect_left_right.py
1
4
```

- `bisect_left(리스트, 값)`은 리스트를 정렬 상태로 유지하면서 그 값을 삽입할 수 있는 **가장 왼쪽** 위치(인덱스)를 반환하고, `bisect_right(리스트, 값)`은 **가장 오른쪽** 위치를 반환한다. 실제로 리스트를 변경하지는 않고 위치만 계산해서 돌려준다.
- `nums`에서 `20`은 인덱스 1~3에 걸쳐 세 개가 있는데, `bisect_left`는 그 앞자리인 `1`을, `bisect_right`는 그 뒷자리인 `4`를 반환한다. 이 차이를 이용하면 리스트 안에 특정 값이 몇 개 있는지(`bisect_right - bisect_left`)도 계산할 수 있다.

**정리**: `bisect` 모듈은 이미 정렬된 리스트를 대상으로 이진 탐색을 이용해 값을 삽입할 위치를 빠르게 찾아주며, `bisect.insort()`는 그 위치에 바로 값을 삽입해 정렬 상태를 유지하고, `bisect_left()`/`bisect_right()`는 리스트를 바꾸지 않고 삽입 가능한 위치(중복 값의 왼쪽 끝/오른쪽 끝)만 알려주므로 순위표나 점수 구간 판정처럼 정렬된 데이터를 계속 유지해야 하는 상황에 유용하다.

## heapq 모듈 — 힙(우선순위 큐)

**힙(heap)**은 가장 작은(또는 가장 큰) 값을 항상 빠르게 꺼낼 수 있도록 정리된 자료구조로, **우선순위 큐(priority queue)**를 구현할 때 널리 쓰인다. `heapq` 모듈은 일반 리스트를 **최소 힙(min-heap)**처럼 다룰 수 있게 해준다.

```python
import heapq

nums = [5, 1, 8, 3, 9, 2]
heapq.heapify(nums)
print(nums)
```

```text
(base) C:\Users\guest\project> python heapq_heapify.py
[1, 3, 2, 5, 9, 8]
```

- `heapq.heapify(리스트)`는 일반 리스트를 그 자리에서(in-place) 힙 순서를 만족하는 형태로 재배열한다. 완전히 정렬된 리스트가 되는 것은 아니지만, 항상 `리스트[0]`에 전체 중 가장 작은 값이 위치하도록 구조가 재정리된다.

**heappush / heappop — 값 추가와 최솟값 꺼내기**

```python
heapq.heappush(nums, 0)
print(nums)

print(heapq.heappop(nums))
print(heapq.heappop(nums))
```

```text
(base) C:\Users\guest\project> python heapq_push_pop.py
[0, 3, 1, 5, 9, 8, 2]
0
1
```

- `heapq.heappush(힙, 값)`은 힙 구조를 유지하면서 새 값을 추가한다. 추가 후에도 여전히 가장 작은 값이 인덱스 `0`에 위치한다.
- `heapq.heappop(힙)`은 힙에서 가장 작은 값을 꺼내면서 제거하고, 남은 요소들로 힙 구조를 다시 정리한다. 매번 `heappop()`을 호출하면 오름차순으로 값을 하나씩 꺼낼 수 있다.
- 리스트를 매번 정렬해서 맨 앞 값을 꺼내는 것보다, `heapq`는 추가·제거 모두 O(log n) 시간에 처리되므로 "가장 작은(우선순위가 높은) 항목을 반복해서 꺼내야 하는" 작업 큐나 스케줄러 구현에 자주 사용된다.

**정리**: `heapq` 모듈은 일반 리스트를 최소 힙으로 다룰 수 있게 해주는 함수들을 제공하며, `heapify()`로 기존 리스트를 힙 구조로 바꾸고 `heappush()`/`heappop()`으로 힙 구조를 유지한 채 값을 추가·제거할 수 있어, 항상 가장 작은(또는 우선순위가 가장 높은) 값을 빠르게 꺼내야 하는 우선순위 큐를 별도의 클래스 구현 없이 리스트 하나로 처리할 수 있다.

## 실습 예제 (EX1~EX4)

**EX1) deque로 최근 방문 기록 N개만 유지하기**
- 최근 방문한 페이지를 기록하되, 항상 최근 3개만 유지되는 방문 기록기를 `deque(maxlen=3)`으로 작성한다.

```python
# ex01_recent_visits.py
from collections import deque

recent = deque(maxlen=3)

for page in ["home", "about", "products", "contact", "cart"]:
    recent.append(page)
    print(f"방문: {page} -> 최근 기록: {list(recent)}")
```

```text
(base) C:\Users\guest\project> python ex01_recent_visits.py
방문: home -> 최근 기록: ['home']
방문: about -> 최근 기록: ['home', 'about']
방문: products -> 최근 기록: ['home', 'about', 'products']
방문: contact -> 최근 기록: ['about', 'products', 'contact']
방문: cart -> 최근 기록: ['products', 'contact', 'cart']
```

- `deque(maxlen=3)`처럼 최대 길이를 지정하면, 그 길이를 넘는 새 요소가 추가될 때 반대쪽 끝의 오래된 요소가 자동으로 제거된다. 최근 N개만 유지하는 로직을 직접 구현할 필요가 없다.

**EX2) 리스트 컴프리헨션으로 구구단 표 만들기**
- 중첩 리스트 컴프리헨션으로 2단부터 9단까지의 구구단 결과를 2차원 리스트로 작성한다.

```python
# ex02_multiplication_table.py
table = [[dan * i for i in range(1, 10)] for dan in range(2, 10)]

for row in table:
    print(row)
```

```text
(base) C:\Users\guest\project> python ex02_multiplication_table.py
[2, 4, 6, 8, 10, 12, 14, 16, 18]
[3, 6, 9, 12, 15, 18, 21, 24, 27]
[4, 8, 12, 16, 20, 24, 28, 32, 36]
[5, 10, 15, 20, 25, 30, 35, 40, 45]
[6, 12, 18, 24, 30, 36, 42, 48, 54]
[7, 14, 21, 28, 35, 42, 49, 56, 63]
[8, 16, 24, 32, 40, 48, 56, 64, 72]
[9, 18, 27, 36, 45, 54, 63, 72, 81]
```

**EX3) 집합으로 두 학급의 공통 수강생 찾기**
- 두 반의 수강생 명단(리스트)에서 양쪽 모두에 속한 학생, 어느 한쪽에만 속한 학생을 각각 집합 연산으로 계산한다.

```python
# ex03_class_sets.py
class_a = ["김철수", "이영희", "박민수", "최지은"]
class_b = ["이영희", "정우성", "박민수", "한소희"]

set_a, set_b = set(class_a), set(class_b)

print("양쪽 모두 수강:", set_a & set_b)
print("A반만 수강:", set_a - set_b)
print("B반만 수강:", set_b - set_a)
print("전체 수강생:", set_a | set_b)
```

```text
(base) C:\Users\guest\project> python ex03_class_sets.py
양쪽 모두 수강: {'이영희', '박민수'}
A반만 수강: {'김철수', '최지은'}
B반만 수강: {'정우성', '한소희'}
전체 수강생: {'김철수', '이영희', '박민수', '최지은', '정우성', '한소희'}
```

**EX4) sorted와 딕셔너리 컴프리헨션으로 순위표 만들기**
- 점수 딕셔너리를 받아 점수 내림차순으로 정렬한 뒤, `{이름: 순위}` 형태의 새 딕셔너리를 딕셔너리 컴프리헨션으로 만드시오.

```python
# ex04_ranking.py
scores = {"김철수": 85, "이영희": 92, "박민수": 78, "최지은": 92}

ranked_names = sorted(scores, key=lambda name: scores[name], reverse=True)
ranking = {name: rank + 1 for rank, name in enumerate(ranked_names)}

print(ranking)
```

```text
(base) C:\Users\guest\project> python ex04_ranking.py
{'이영희': 1, '최지은': 2, '김철수': 3, '박민수': 4}
```

- `sorted(scores, key=...)`처럼 딕셔너리 자체를 `sorted()`에 넘기면 키(이름)들만 정렬 대상이 되며, `key=lambda name: scores[name]`으로 각 이름에 해당하는 점수를 기준으로 정렬한다. 이후 `enumerate()`로 순위를 매겨 딕셔너리 컴프리헨션으로 최종 결과를 만든다.

**정리**: EX1~EX4는 `deque`로 최근 N개 기록만 유지하는 실용적인 패턴, 중첩 리스트 컴프리헨션으로 2차원 표를 생성하는 방법, 집합 연산으로 여러 그룹 간의 관계를 구하는 방법, `sorted()`와 딕셔너리 컴프리헨션을 조합해 순위표를 만드는 방법까지 이 문서에서 다룬 자료구조 심화 개념을 직접 코드로 확인해보는 예제다.

# Python 04 — 리스트와 딕셔너리

## 리스트 생성과 인덱싱

**리스트(list)**는 여러 개의 값을 순서대로 하나의 변수에 담아 관리하는 자료구조다. 대괄호 `[]` 안에 쉼표로 값을 나열해서 만든다.

```python
>>> fruits = ["사과", "바나나", "딸기"]
>>> print(fruits)
['사과', '바나나', '딸기']
>>> print(type(fruits))
<class 'list'>
```

- 리스트는 서로 다른 자료형을 섞어서 담을 수도 있다.

```python
>>> mixed = [1, "두 번째", 3.0, True]
>>> print(mixed)
[1, '두 번째', 3.0, True]
```

**인덱싱(indexing)**

리스트의 각 값은 `0`부터 시작하는 **인덱스(index)**로 접근한다.

```python
>>> fruits = ["사과", "바나나", "딸기"]
>>> print(fruits[0])
사과
>>> print(fruits[2])
딸기
```

- 첫 번째 값은 인덱스 `0`, 두 번째 값은 인덱스 `1`이며, 마지막 값의 인덱스는 `len(리스트) - 1`이다.

**음수 인덱스**

파이썬 리스트는 뒤에서부터 접근하는 음수 인덱스도 지원한다.

```python
>>> print(fruits[-1])
딸기
>>> print(fruits[-2])
바나나
```

- `-1`은 마지막 값, `-2`는 뒤에서 두 번째 값을 가리킨다. 리스트 끝쪽 값에 접근할 때 `len(fruits) - 1`을 계산할 필요 없이 바로 `-1`을 쓸 수 있어 편리하다.

**범위를 벗어난 인덱스**

```python
>>> print(fruits[10])
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
IndexError: list index out of range
```

- 존재하지 않는 인덱스에 접근하면 `IndexError`가 발생한다.

**정리**: 리스트는 대괄호로 값을 나열해 만드는 순서가 있는 자료구조이며, `0`부터 시작하는 양수 인덱스와 `-1`부터 시작해 뒤에서 세는 음수 인덱스로 값에 접근할 수 있고, 존재하지 않는 인덱스에 접근하면 `IndexError`가 발생한다.

## 2차원 리스트

리스트 안에 또 다른 리스트를 담으면 표(table)나 행렬처럼 2차원 구조를 표현할 수 있다.

```python
>>> matrix = [
...     [1, 2, 3],
...     [4, 5, 6],
...     [7, 8, 9],
... ]
>>> print(matrix)
[[1, 2, 3], [4, 5, 6], [7, 8, 9]]
```

- `matrix[0]`은 첫 번째 행 전체(리스트)를 가리킨다.
- `matrix[0][1]`처럼 인덱스를 두 번 연달아 쓰면 행과 열을 순서대로 지정해 개별 값에 접근할 수 있다.

```python
>>> print(matrix[0])
[1, 2, 3]
>>> print(matrix[0][1])
2
>>> print(matrix[2][2])
9
```

**2차원 리스트 순회하기**

```python
matrix = [
    [1, 2, 3],
    [4, 5, 6],
    [7, 8, 9],
]

for row in matrix:
    for value in row:
        print(value, end=" ")
    print()
```

```text
(base) C:\Users\guest\project> python matrix_loop.py
1 2 3
4 5 6
7 8 9
```

- 바깥 반복문은 각 행(리스트)을 하나씩 꺼내고, 안쪽 반복문은 그 행 안의 값을 하나씩 꺼낸다. 반복문(`for`) 문법 자체는 다음 문서에서 본격적으로 다루며, 여기서는 2차원 리스트의 구조를 이해하는 데 집중한다.

**정리**: 2차원 리스트는 리스트 안에 리스트를 중첩시켜 행과 열을 가진 표 형태의 데이터를 표현하는 방식이며, `matrix[행][열]` 형태로 인덱스를 두 번 사용해 개별 값에 접근하고, 중첩 반복문으로 전체 값을 순서대로 순회할 수 있다.

## 슬라이싱

**슬라이싱(slicing)**은 리스트의 일부 구간을 잘라내 새 리스트로 얻는 문법이다. `리스트[시작:끝]` 형태로 사용하며, 시작 인덱스는 포함하고 끝 인덱스는 포함하지 않는다.

```python
>>> numbers = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
>>> print(numbers[2:5])
[2, 3, 4]
```

- `numbers[2:5]`는 인덱스 `2`부터 `4`까지(인덱스 `5`는 제외)를 잘라낸다.

**시작이나 끝을 생략하기**

```python
>>> print(numbers[:3])
[0, 1, 2]
>>> print(numbers[7:])
[7, 8, 9]
>>> print(numbers[:])
[0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
```

- 시작을 생략하면 처음부터, 끝을 생략하면 끝까지를 의미한다. 둘 다 생략한 `[:]`는 리스트 전체를 복사한 새 리스트를 만든다(PY-02에서 다룬 값 복사 vs 참조 공유 내용을 떠올려보면, `[:]`는 원본과 독립된 새 리스트를 만드는 방법이다).

**세 번째 값으로 간격(step) 지정하기**

```python
>>> print(numbers[0:10:2])
[0, 2, 4, 6, 8]
>>> print(numbers[::-1])
[9, 8, 7, 6, 5, 4, 3, 2, 1, 0]
```

- `리스트[시작:끝:간격]`에서 간격을 `2`로 지정하면 두 칸씩 건너뛰며 값을 가져온다.
- 간격을 `-1`로 지정하면 리스트를 거꾸로 뒤집은 결과를 얻는다. `[::-1]`은 리스트를 뒤집는 관용적인 표현이다.

**음수 인덱스와 슬라이싱 함께 쓰기**

```python
>>> print(numbers[-3:])
[7, 8, 9]
>>> print(numbers[:-3])
[0, 1, 2, 3, 4, 5, 6]
```

**정리**: 슬라이싱은 `리스트[시작:끝:간격]` 형태로 리스트의 일부를 새 리스트로 잘라내며 시작 인덱스는 포함, 끝 인덱스는 제외되고, 시작·끝을 생략하면 각각 처음·끝을 의미하며 `[::-1]`처럼 음수 간격을 주면 리스트를 뒤집을 수 있고, `[:]`로 슬라이싱하면 원본과 독립된 새 리스트를 만들 수 있다.

## sum · min · max · len

리스트를 다룰 때 자주 쓰이는 내장 함수 네 가지가 있다.

```python
>>> scores = [88, 95, 70, 100, 60]
>>> print(len(scores))
5
>>> print(sum(scores))
413
>>> print(min(scores))
60
>>> print(max(scores))
100
```

| 함수 | 의미 |
|------|------|
| `len(리스트)` | 리스트의 요소 개수 |
| `sum(리스트)` | 숫자 요소들의 합 |
| `min(리스트)` | 가장 작은 값 |
| `max(리스트)` | 가장 큰 값 |

**평균을 구하는 관용적인 조합**

```python
>>> average = sum(scores) / len(scores)
>>> print(average)
82.6
```

- 파이썬에는 평균을 구하는 전용 내장 함수가 따로 없으므로, `sum()`과 `len()`을 조합해 직접 계산하는 방식이 관용적으로 쓰인다.

**빈 리스트에 min()·max()를 쓰면 오류가 발생한다**

```python
>>> print(min([]))
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
ValueError: min() arg is an empty sequence
```

- 빈 리스트로 최댓값·최솟값을 구하려 하면 비교할 값이 없으므로 `ValueError`가 발생한다. 이런 경우를 대비해 리스트가 비어 있는지 먼저 확인하는 것이 안전하다.

**정리**: `len()`은 요소 개수, `sum()`은 합계, `min()`/`max()`는 최솟값·최댓값을 구하는 파이썬 내장 함수이며 네 함수 모두 리스트뿐 아니라 다른 순회 가능한 자료형에도 그대로 적용할 수 있고, 평균은 `sum() / len()` 조합으로 직접 계산하는 것이 관용적이다.

## 리스트에 값 추가하기

**append()로 끝에 값 하나 추가하기**

```python
>>> fruits = ["사과", "바나나"]
>>> fruits.append("딸기")
>>> print(fruits)
['사과', '바나나', '딸기']
```

- `append()`는 리스트의 맨 끝에 값을 하나 추가한다. 여러 값을 한 번에 추가하는 것이 아니라 인자 하나 전체를 요소 하나로 추가한다는 점에 주의해야 한다.

```python
>>> fruits.append(["포도", "수박"])
>>> print(fruits)
['사과', '바나나', '딸기', ['포도', '수박']]
```

- 리스트를 통째로 `append()`하면 리스트 안에 리스트가 중첩되어 들어간다.

**+ 연산자로 리스트 연결하기**

여러 값을 한 번에 추가하려면 `+` 연산자로 두 리스트를 이어붙일 수 있다.

```python
>>> fruits = ["사과", "바나나"]
>>> fruits = fruits + ["포도", "수박"]
>>> print(fruits)
['사과', '바나나', '포도', '수박']
```

- `+`는 원본 리스트를 바꾸지 않고 두 리스트를 이어붙인 새 리스트를 만들어 반환하므로, 위 예시처럼 결과를 다시 원래 변수에 대입해야 반영된다.

**extend()로 다른 리스트의 요소를 낱개로 추가하기**

```python
>>> fruits = ["사과", "바나나"]
>>> fruits.extend(["포도", "수박"])
>>> print(fruits)
['사과', '바나나', '포도', '수박']
```

- `extend()`는 `append()`와 달리 인자로 받은 리스트의 요소들을 낱개로 풀어서 추가하며, 원본 리스트 자체를 직접 수정한다는 점이 `+` 연산자와 다르다.

**append() vs extend() 비교**

```python
>>> a = [1, 2]
>>> a.append([3, 4])
>>> print(a)
[1, 2, [3, 4]]

>>> b = [1, 2]
>>> b.extend([3, 4])
>>> print(b)
[1, 2, 3, 4]
```

**정리**: `append()`는 인자 전체를 요소 하나로 리스트 끝에 추가하고, `extend()`는 인자로 받은 리스트의 요소들을 낱개로 풀어서 추가하며, `+` 연산자는 원본을 바꾸지 않고 두 리스트를 이어붙인 새 리스트를 반환한다는 차이를 구분해서 사용해야 한다.

## 리스트에서 값 제거·수정하기

**del로 인덱스 또는 슬라이싱 범위 제거하기**

```python
>>> fruits = ["사과", "바나나", "딸기", "포도"]
>>> del fruits[1]
>>> print(fruits)
['사과', '딸기', '포도']
```

- `del 리스트[인덱스]`는 지정한 인덱스의 값을 리스트에서 제거한다.

```python
>>> numbers = [0, 1, 2, 3, 4, 5]
>>> del numbers[1:4]
>>> print(numbers)
[0, 4, 5]
```

- `del`은 슬라이싱 범위를 그대로 넘겨 여러 값을 한 번에 제거할 수도 있다.

**remove()로 값을 지정해서 제거하기**

```python
>>> fruits = ["사과", "바나나", "딸기"]
>>> fruits.remove("바나나")
>>> print(fruits)
['사과', '딸기']
```

- `remove()`는 인덱스가 아니라 값 자체를 지정해서 제거하며, 같은 값이 여러 개 있으면 가장 앞에 있는 것 하나만 제거된다. 리스트에 없는 값을 지정하면 `ValueError`가 발생한다.

**pop()으로 제거와 동시에 값 꺼내기**

```python
>>> fruits = ["사과", "바나나", "딸기"]
>>> last = fruits.pop()
>>> print(last, fruits)
딸기 ['사과', '바나나']
```

- `pop()`은 인자를 생략하면 마지막 값을, 인덱스를 넘기면 해당 위치의 값을 제거하면서 그 값을 반환한다. 제거한 값을 바로 활용해야 할 때 유용하다.

**값 수정하기**

리스트의 특정 인덱스에 새 값을 대입하면 그 위치의 값이 바뀐다.

```python
>>> fruits = ["사과", "바나나", "딸기"]
>>> fruits[1] = "수박"
>>> print(fruits)
['사과', '수박', '딸기']
```

- 슬라이싱 범위에 리스트를 대입하면 여러 값을 한 번에 바꿀 수도 있다.

```python
>>> numbers = [0, 1, 2, 3, 4]
>>> numbers[1:3] = [10, 20, 30]
>>> print(numbers)
[0, 10, 20, 30, 3, 4]
```

- `numbers[1:3]`(인덱스 1, 2 두 자리)에 세 개짜리 리스트를 대입하면, 원래 있던 두 자리가 사라지고 새로 넘긴 세 값으로 대체되어 전체 길이가 바뀔 수 있다.

**정리**: `del`은 인덱스나 슬라이싱 범위를 지정해 값을 제거하고, `remove()`는 값 자체로 첫 번째 일치 항목을 제거하며, `pop()`은 제거와 동시에 그 값을 반환한다는 점에서 서로 다르게 쓰이고, 인덱스나 슬라이싱 범위에 새 값을 대입하면 해당 위치의 값을 수정하거나 교체할 수 있다.

## 딕셔너리 생성과 접근

**딕셔너리(dictionary)**는 값을 순서가 아니라 **키(key)**로 관리하는 자료구조다. 중괄호 `{}` 안에 `키: 값` 쌍을 쉼표로 나열해서 만든다.

```python
>>> student = {"name": "홍길동", "age": 20, "grade": "A"}
>>> print(student)
{'name': '홍길동', 'age': 20, 'grade': 'A'}
>>> print(type(student))
<class 'dict'>
```

**키로 값에 접근하기**

```python
>>> print(student["name"])
홍길동
>>> print(student["age"])
20
```

- 리스트가 인덱스(위치)로 값에 접근한다면, 딕셔너리는 키(이름)로 값에 접근한다는 점이 근본적인 차이다.

```python
>>> print(student["phone"])
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
KeyError: 'phone'
```

- 존재하지 않는 키로 접근하면 `KeyError`가 발생한다. 키가 있는지 확신할 수 없을 때는 `get()` 메서드를 쓰면 키가 없을 때 오류 대신 `None`(또는 지정한 기본값)을 반환받을 수 있다.

```python
>>> print(student.get("phone"))
None
>>> print(student.get("phone", "정보 없음"))
정보 없음
```

**정리**: 딕셔너리는 중괄호와 `키: 값` 쌍으로 만들며 인덱스가 아니라 키로 값에 접근하는 자료구조이고, 존재하지 않는 키로 대괄호 접근을 하면 `KeyError`가 발생하지만 `get()` 메서드를 쓰면 오류 대신 `None`이나 지정한 기본값을 안전하게 돌려받을 수 있다.

## 딕셔너리 추가·삭제와 keys() · values()

**값 추가하기**

```python
>>> student = {"name": "홍길동", "age": 20}
>>> student["grade"] = "A"
>>> print(student)
{'name': '홍길동', 'age': 20, 'grade': 'A'}
```

- 존재하지 않는 키에 값을 대입하면 새 키-값 쌍이 추가된다. 이미 존재하는 키에 대입하면 값이 덮어써진다는 점은 EX 예제에서 다시 확인한다.

```python
>>> student["age"] = 21
>>> print(student)
{'name': '홍길동', 'age': 21, 'grade': 'A'}
```

**del로 삭제하기**

```python
>>> del student["grade"]
>>> print(student)
{'name': '홍길동', 'age': 21}
```

- 리스트와 마찬가지로 `del 딕셔너리[키]` 형태로 특정 키-값 쌍을 제거한다.

**keys() · values() · items()**

```python
>>> student = {"name": "홍길동", "age": 20, "grade": "A"}
>>> print(student.keys())
dict_keys(['name', 'age', 'grade'])
>>> print(student.values())
dict_values(['홍길동', 20, 'A'])
>>> print(student.items())
dict_items([('name', '홍길동'), ('age', 20), ('grade', 'A')])
```

- `keys()`는 모든 키를, `values()`는 모든 값을, `items()`는 `(키, 값)` 쌍을 모두 꺼내온다. 세 결과 모두 `for` 반복문으로 순회할 때 자주 사용되며, 반복문 활용은 다음 문서에서 자세히 다룬다.

**서로 다른 자료형의 값을 함께 저장하기**

딕셔너리 하나에 문자열, 숫자, 불리언, 리스트, 심지어 다른 딕셔너리까지 자유롭게 섞어 담을 수 있다.

```python
>>> profile = {
...     "name": "홍길동",
...     "age": 20,
...     "is_active": True,
...     "hobbies": ["독서", "등산"],
...     "address": {"city": "서울", "zipcode": "12345"},
... }
>>> print(profile["hobbies"])
['독서', '등산']
>>> print(profile["address"]["city"])
서울
```

- `profile["address"]["city"]`처럼 대괄호를 연달아 쓰면 딕셔너리 안에 중첩된 딕셔너리의 값까지 접근할 수 있다.

**정리**: 딕셔너리는 `키 = 값` 형태의 대입으로 새 키를 추가하거나 기존 키의 값을 덮어쓸 수 있고 `del`로 키-값 쌍을 삭제하며, `keys()`/`values()`/`items()`로 각각 키 전체, 값 전체, 키-값 쌍 전체를 꺼낼 수 있고, 값 자리에는 숫자·문자열·리스트·다른 딕셔너리 등 어떤 자료형이든 자유롭게 섞어서 저장할 수 있다.

## 실습 예제 (EX1~EX4)

**EX1) 성적 리스트에서 통계 뽑기**
- 점수 리스트에서 `sum`/`len`/`min`/`max`를 활용해 합계·평균·최고점·최저점을 출력한다.

```python
# ex01_score_stats.py
scores = [76, 88, 92, 64, 100, 55]

total = sum(scores)
average = total / len(scores)
highest = max(scores)
lowest = min(scores)

print(f"합계: {total}, 평균: {average:.2f}, 최고점: {highest}, 최저점: {lowest}")
```

```text
(base) C:\Users\guest\project> python ex01_score_stats.py
합계: 475, 평균: 79.17, 최고점: 100, 최저점: 55
```

**EX2) 2차원 리스트로 좌석표 만들기**
- 3행 3열의 좌석표를 2차원 리스트로 만들고, 특정 좌석을 "예약됨"으로 수정한 뒤 전체 좌석표를 출력한다.

```python
# ex02_seat_table.py
seats = [
    ["빈자리", "빈자리", "빈자리"],
    ["빈자리", "빈자리", "빈자리"],
    ["빈자리", "빈자리", "빈자리"],
]

seats[1][1] = "예약됨"

for row in seats:
    print(row)
```

```text
(base) C:\Users\guest\project> python ex02_seat_table.py
['빈자리', '빈자리', '빈자리']
['빈자리', '예약됨', '빈자리']
['빈자리', '빈자리', '빈자리']
```

**EX3) 장바구니에 물건 추가·삭제 시뮬레이션**
- 빈 리스트에서 시작해 `append()`로 물건을 담고, `remove()`로 하나를 빼고, 슬라이싱으로 앞의 두 개만 확인한다.

```python
# ex03_cart.py
cart = []
cart.append("우유")
cart.append("계란")
cart.append("식빵")
cart.append("사과")
print("담은 직후:", cart)

cart.remove("계란")
print("계란 제거 후:", cart)

print("앞 두 개만:", cart[:2])
```

```text
(base) C:\Users\guest\project> python ex03_cart.py
담은 직후: ['우유', '계란', '식빵', '사과']
계란 제거 후: ['우유', '식빵', '사과']
앞 두 개만: ['우유', '식빵']
```

**EX4) 딕셔너리로 재고 관리하기**
- 상품명을 키로, 수량을 값으로 하는 딕셔너리를 만들고, 값을 덮어써서 수량을 수정하고, `del`로 품절 상품을 제거한 뒤 `items()`로 전체 재고를 출력한다.

```python
# ex04_inventory.py
inventory = {"사과": 10, "바나나": 5, "딸기": 0}

inventory["사과"] = 8    # 판매로 수량 감소 (값 덮어쓰기)
del inventory["딸기"]     # 품절 상품 제거

for name, qty in inventory.items():
    print(f"{name}: {qty}개")
```

```text
(base) C:\Users\guest\project> python ex04_inventory.py
사과: 8개
바나나: 5개
```

- `inventory["사과"] = 8`처럼 이미 존재하는 키에 값을 대입하면 새로 추가되는 것이 아니라 기존 값이 덮어써진다는 점을 확인할 수 있다.

**정리**: EX1~EX4는 리스트 통계 함수 조합, 2차원 리스트 수정, `append()`/`remove()`/슬라이싱을 활용한 장바구니 시뮬레이션, 딕셔너리 값 덮어쓰기와 `del`·`items()` 활용까지 이 문서의 핵심 내용을 코드로 직접 확인해보는 예제이며, 다음 문서에서는 이런 자료구조를 실제로 다루는 데 필수적인 조건문과 반복문을 다룬다.

[Python 03 — 함수](03-functions.md) · [Python 05 — 조건문과 반복문](05-conditions-loops.md)

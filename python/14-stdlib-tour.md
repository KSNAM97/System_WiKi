# Python 14 — 표준 라이브러리 살펴보기

## sys 모듈

`sys` 모듈은 파이썬 인터프리터 자체와 관련된 정보와 기능을 제공한다.

**sys.argv — 커맨드라인 인자**

```python
# argv_demo.py
import sys

print("스크립트 이름:", sys.argv[0])
print("전체 인자 리스트:", sys.argv)
print("인자 개수:", len(sys.argv))

if len(sys.argv) > 1:
    print("첫 번째 인자:", sys.argv[1])
else:
    print("추가 인자가 없습니다.")
```

```text
(base) C:\Users\guest\project> python argv_demo.py hello 123
스크립트 이름: argv_demo.py
전체 인자 리스트: ['argv_demo.py', 'hello', '123']
인자 개수: 3
첫 번째 인자: hello
```

- `sys.argv`는 스크립트를 실행할 때 함께 넘긴 명령줄 인자들을 담은 리스트다. `sys.argv[0]`은 항상 실행된 스크립트 자신의 이름이고, 그 뒤로 공백으로 구분해 넘긴 인자들이 문자열로 순서대로 담긴다. 숫자를 넘기더라도 항상 문자열로 전달되므로, 숫자로 쓰려면 `int(sys.argv[1])`처럼 직접 변환해야 한다.

**sys.exit() — 프로그램 즉시 종료**

```python
# sys_exit_demo.py
import sys

def check_positive(n):
    if n < 0:
        print("오류: 음수는 처리할 수 없습니다.")
        sys.exit(1)  # 0이 아닌 종료 코드는 관례적으로 오류를 의미한다
    return n * 2

print(check_positive(5))
print(check_positive(-3))
print("이 줄은 실행되지 않는다.")
```

```text
(base) C:\Users\guest\project> python sys_exit_demo.py
10
오류: 음수는 처리할 수 없습니다.

(base) C:\Users\guest\project> echo %errorlevel%
1
```

- `sys.exit(종료코드)`는 프로그램을 즉시 종료시킨다. 종료 코드를 생략하면 `0`(정상 종료)이 전달되며, 관례적으로 `0`이 아닌 값은 어떤 형태로든 오류가 있었음을 나타낸다. 셸 스크립트나 다른 프로그램이 이 파이썬 스크립트를 호출했을 때, 이 종료 코드를 보고 성공/실패를 판단할 수 있다.

**정리**: `sys` 모듈은 인터프리터와 실행 환경에 대한 정보를 다루며, `sys.argv`로 커맨드라인에서 넘긴 인자들을 리스트로 받아 스크립트 동작을 제어할 수 있고, `sys.exit(코드)`로 프로그램을 원하는 시점에 즉시 종료하면서 성공/실패 여부를 종료 코드로 남길 수 있어, 명령줄 도구를 만들 때 특히 자주 사용된다.

## glob 모듈

PY-07에서 `os.listdir()`로 디렉터리 안의 파일 목록을 가져오는 방법을 다뤘다. `glob` 모듈은 여기에 **와일드카드 패턴**을 더해, 조건에 맞는 파일만 손쉽게 찾을 수 있게 해준다.

```python
import glob
import os

os.makedirs("data", exist_ok=True)
for name in ["a.txt", "b.txt", "c.csv", "d.log"]:
    with open(os.path.join("data", name), "w", encoding="utf-8") as f:
        f.write("샘플\n")

print("os.listdir() 결과:", sorted(os.listdir("data")))
print("glob 전체(*):", sorted(glob.glob("data/*")))
print("glob .txt만:", sorted(glob.glob("data/*.txt")))
```

```text
(base) C:\Users\guest\project> python glob_demo.py
os.listdir() 결과: ['a.txt', 'b.txt', 'c.csv', 'd.log']
glob 전체(*): ['data\\a.txt', 'data\\b.txt', 'data\\c.csv', 'data\\d.log']
glob .txt만: ['data\\a.txt', 'data\\b.txt']
```

- `os.listdir(경로)`는 해당 경로 안의 모든 항목 이름을 조건 없이 그대로 반환하는 반면, `glob.glob(패턴)`은 `*`(임의의 문자열), `?`(문자 하나), `[...]`(문자 집합) 같은 와일드카드 패턴과 일치하는 경로만 걸러서 반환하며, 반환값에는 지정한 경로가 포함된 전체 경로 문자열이 들어간다.
- `glob.glob("data/*.txt")`처럼 확장자별로 파일을 찾고 싶을 때 `os.listdir()`로 가져온 뒤 직접 `if name.endswith(".txt")`로 걸러내는 것보다 한 줄로 간결하게 처리할 수 있다.

**하위 디렉터리까지 재귀적으로 찾기**

```python
import glob

# recursive=True와 ** 패턴을 함께 쓰면 하위 디렉터리까지 모두 검색한다
for path in sorted(glob.glob("data/**/*.txt", recursive=True)):
    print(path)
```

```text
(base) C:\Users\guest\project> python glob_recursive.py
data\a.txt
data\b.txt
```

- `**` 패턴은 `recursive=True` 옵션과 함께 사용해야 하며, 이 경우 하위 디렉터리를 포함해 모든 깊이에서 패턴과 일치하는 경로를 찾는다.

**정리**: `glob` 모듈은 `os.listdir()`이 제공하는 단순 목록 조회에 `*`/`?`/`[...]` 같은 와일드카드 패턴 매칭을 더해, 특정 확장자나 이름 패턴에 맞는 파일만 골라 찾을 수 있게 해주며, `recursive=True`와 `**` 패턴을 함께 쓰면 하위 디렉터리까지 재귀적으로 검색할 수 있어 여러 폴더에 흩어진 파일을 다룰 때 유용하다.

## re 모듈 — 정규표현식 기초

**정규표현식(regular expression)**은 문자열에서 특정 패턴을 찾거나 검증하기 위한 미니 언어다. 파이썬은 표준 라이브러리 `re` 모듈로 정규표현식을 지원한다.

```python
import re

text = "문의: 010-1234-5678, 이메일: user@example.com"

m1 = re.match(r"\d+", "123abc")
print("match:", m1.group() if m1 else None)

m2 = re.search(r"\d{3}-\d{4}-\d{4}", text)
print("search:", m2.group() if m2 else None)

all_numbers = re.findall(r"\d+", text)
print("findall:", all_numbers)

masked = re.sub(r"\d{3}-\d{4}-\d{4}", "***-****-****", text)
print("sub:", masked)
```

```text
(base) C:\Users\guest\project> python re_demo.py
match: 123
search: 010-1234-5678
findall: ['010', '1234', '5678']
sub: 문의: ***-****-****, 이메일: user@example.com
```

- `re.match(패턴, 문자열)`은 문자열의 **맨 앞**에서부터 패턴이 일치하는지 검사하며, 앞부분이 일치하지 않으면 뒤쪽에 일치하는 부분이 있어도 `None`을 반환한다.
- `re.search(패턴, 문자열)`은 위치와 상관없이 문자열 전체에서 패턴과 일치하는 **첫 번째** 부분을 찾는다.
- `re.findall(패턴, 문자열)`은 일치하는 **모든** 부분을 리스트로 반환한다.
- `re.sub(패턴, 대체문자열, 문자열)`은 일치하는 모든 부분을 지정한 문자열로 치환한다.
- 정규표현식 문자열 앞에 `r`을 붙여 **raw 문자열**로 쓰는 것이 관례인데, 정규표현식에서 자주 쓰이는 `\d`, `\w` 같은 표기가 파이썬의 이스케이프 시퀀스와 겹치는 것을 방지하기 위함이다.
- `\d`는 숫자 하나, `+`는 앞 패턴이 1개 이상 반복, `{3}`은 정확히 3번 반복을 의미한다.

**match 객체 다루기**

```python
import re

m = re.search(r"(\d{3})-(\d{4})-(\d{4})", "010-1234-5678")
if m:
    print("전체:", m.group())
    print("앞 3자리:", m.group(1))
    print("가운데 4자리:", m.group(2))
    print("끝 4자리:", m.group(3))
    print("시작 위치:", m.start())
```

```text
(base) C:\Users\guest\project> python re_groups.py
전체: 010-1234-5678
앞 3자리: 010
가운데 4자리: 1234
끝 4자리: 5678
시작 위치: 0
```

- 패턴 안에서 `()`로 묶은 부분을 **그룹**이라고 하며, `m.group(0)`(또는 인자 없는 `m.group()`)은 일치한 전체 문자열을, `m.group(1)`, `m.group(2)`처럼 번호를 지정하면 각 그룹만 따로 꺼낼 수 있다.

**정리**: `re` 모듈은 문자열 앞부분만 검사하는 `match()`, 문자열 전체에서 첫 일치를 찾는 `search()`, 모든 일치를 리스트로 반환하는 `findall()`, 일치하는 부분을 치환하는 `sub()`를 제공하며, `\d`(숫자)·`+`(1회 이상 반복)·`{n}`(n회 반복) 같은 기본 패턴 문법과 `()`로 묶은 그룹을 조합하면 전화번호나 이메일처럼 정해진 형식을 가진 문자열을 찾거나 검증하고 원하는 부분만 추출하는 작업을 코드 몇 줄로 처리할 수 있다.

## datetime 모듈

`datetime` 모듈은 날짜와 시간을 다루기 위한 표준 라이브러리다.

**현재 날짜/시간 얻기**

```python
import datetime

now = datetime.datetime.now()
print(now)
print(now.year, now.month, now.day)
print(now.hour, now.minute, now.second)
```

```text
(base) C:\Users\guest\project> python datetime_now.py
2026-09-10 14:30:05.123456
2026 9 10
14 30 5
```

- `datetime.datetime.now()`는 현재 시각을 담은 `datetime` 객체를 반환하며, 실행할 때마다 값이 달라진다. `.year`, `.month`, `.day`, `.hour`, `.minute`, `.second` 속성으로 각 구성 요소를 따로 꺼낼 수 있다.

**strftime()으로 원하는 형식의 문자열 만들기**

```python
import datetime

d = datetime.datetime(2026, 9, 10, 14, 30, 0)

print(d.strftime("%Y-%m-%d"))
print(d.strftime("%Y/%m/%d %H:%M:%S"))
print(d.strftime("%A, %B %d, %Y"))
```

```text
(base) C:\Users\guest\project> python strftime_demo.py
2026-09-10
2026/09/10 14:30:00
Thursday, September 10, 2026
```

- `strftime(포맷문자열)`은 `datetime` 객체를 원하는 형식의 **문자열**로 변환한다. `%Y`(4자리 연도), `%m`(2자리 월), `%d`(2자리 일), `%H`/`%M`/`%S`(시/분/초), `%A`(요일 전체 이름), `%B`(월 전체 이름)가 자주 쓰인다.
- 요일·월 이름은 실행 환경의 로케일(locale) 설정에 따라 영어 또는 한국어로 다르게 표시될 수 있으며, 별도 설정을 하지 않은 기본 환경에서는 보통 영어로 표시된다.

**strptime()으로 문자열을 datetime으로 변환하기**

```python
import datetime

text = "2026-01-01"
d = datetime.datetime.strptime(text, "%Y-%m-%d")
print(d)
print(type(d))
```

```text
(base) C:\Users\guest\project> python strptime_demo.py
2026-01-01 00:00:00
<class 'datetime.datetime'>
```

- `strptime(문자열, 포맷문자열)`은 `strftime()`과 반대로, 정해진 형식의 **문자열**을 `datetime` 객체로 변환한다. 포맷 문자열이 실제 문자열의 형식과 정확히 일치해야 하며, 일치하지 않으면 `ValueError`가 발생한다.

**날짜 연산 — timedelta**

```python
import datetime

today = datetime.datetime(2026, 1, 1)
ten_days_later = today + datetime.timedelta(days=10)
one_week_before = today - datetime.timedelta(weeks=1)

print("오늘:", today.strftime("%Y-%m-%d"))
print("10일 후:", ten_days_later.strftime("%Y-%m-%d"))
print("1주일 전:", one_week_before.strftime("%Y-%m-%d"))

d1 = datetime.datetime(2026, 9, 10, 14, 30, 0)
d2 = datetime.datetime(2026, 1, 1, 0, 0, 0)
diff = d1 - d2
print("두 날짜의 차이:", diff)
print("차이 일수:", diff.days)
```

```text
(base) C:\Users\guest\project> python timedelta_demo.py
오늘: 2026-01-01
10일 후: 2026-01-11
1주일 전: 2025-12-25
두 날짜의 차이: 252 days, 14:30:00
차이 일수: 252
```

- `timedelta(days=..., weeks=..., hours=..., ...)`는 날짜/시간의 "간격"을 나타내는 객체이며, `datetime` 객체에 더하거나 빼서 며칠 후/이전의 날짜를 쉽게 계산할 수 있다.
- `datetime` 객체끼리 뺄셈을 하면 그 차이를 나타내는 `timedelta` 객체가 반환되며, `.days` 속성으로 정수 일수만 따로 꺼낼 수 있다.

**정리**: `datetime` 모듈은 `datetime.now()`로 현재 시각을 얻고, `strftime()`으로 `datetime` 객체를 원하는 형식의 문자열로, `strptime()`으로 문자열을 다시 `datetime` 객체로 변환하며, `timedelta`를 이용해 날짜와 시간 사이의 덧셈·뺄셈 연산을 지원하므로 로그 타임스탬프 기록, 마감일 계산, 날짜 형식 변환 등 날짜/시간과 관련된 대부분의 실무 작업을 이 모듈 하나로 처리할 수 있다.

## random 모듈

`random` 모듈은 난수(무작위 값)를 생성하는 기능을 제공한다.

```python
import random

random.seed(42)  # 같은 시드를 주면 항상 같은 결과가 나온다(재현 가능한 난수)

print(random.random())        # 0.0 이상 1.0 미만의 실수
print(random.randint(1, 10))  # 1 이상 10 이하의 정수(양 끝 포함)
print(random.choice(["가위", "바위", "보"]))

numbers = [1, 2, 3, 4, 5]
random.shuffle(numbers)
print(numbers)
```

```text
(base) C:\Users\guest\project> python random_demo.py
0.6394267984578837
1
보
[4, 5, 1, 2, 3]
```

- `random.seed(값)`으로 시드를 고정하면 이후 `random` 함수들이 항상 같은 순서로 같은 값을 만들어낸다. 테스트 코드에서 결과를 재현 가능하게 만들고 싶을 때 유용하며, 시드를 고정하지 않으면 실행할 때마다 다른 결과가 나온다.
- `random.random()`은 `0.0` 이상 `1.0` 미만의 실수를, `random.randint(a, b)`는 `a` 이상 `b` **이하**(양 끝 포함)의 정수를 반환한다.
- `random.choice(시퀀스)`는 리스트나 튜플 등에서 원소 하나를 무작위로 골라 반환하며, `random.shuffle(리스트)`는 리스트의 순서를 무작위로 섞되 **원본 리스트 자체를 직접 변경**하고 반환값은 `None`이다.

**정리**: `random` 모듈은 `random.random()`(0~1 사이 실수), `random.randint(a, b)`(정수 범위), `random.choice(시퀀스)`(무작위 선택), `random.shuffle(리스트)`(무작위 순서 섞기)처럼 상황별로 특화된 난수 함수를 제공하며, `random.seed()`로 시드를 고정하면 같은 순서의 난수를 재현할 수 있어 게임의 무작위 요소부터 테스트 데이터 생성, 샘플링까지 다양한 곳에 활용된다.

## time 모듈로 실행 시간 측정하기

코드가 얼마나 오래 걸리는지 측정하고 싶을 때 `time` 모듈의 `time()` 함수를 간단히 활용할 수 있다.

```python
import time

start = time.time()

total = 0
for i in range(1, 5_000_001):
    total += i

end = time.time()

print("합계:", total)
print(f"소요 시간: {end - start:.4f}초")
```

```text
(base) C:\Users\guest\project> python time_measure.py
합계: 12500002500000
소요 시간: 0.3521초
```

- `time.time()`은 1970년 1월 1일(유닉스 에포크) 이후 지난 시간을 초 단위 실수(타임스탬프)로 반환한다. 작업 시작 직전과 직후에 각각 호출해 그 차이를 구하면 실행 시간을 측정할 수 있다.
- 실제 소요 시간은 실행 환경의 성능에 따라 달라지므로, 위 `0.3521초`라는 값은 환경마다 다르게 나올 수 있는 예시 값이다. 정확한 벤치마크가 필요하다면 표준 라이브러리의 `timeit` 모듈을 사용하는 것이 더 적합하지만, 개발 중 대략적인 실행 시간을 빠르게 확인할 때는 `time.time()`만으로도 충분한 경우가 많다.

**정리**: `time.time()`을 코드 실행 앞뒤에서 호출하고 그 차이를 계산하면 해당 구간의 실행 시간을 간단히 측정할 수 있으며, 개발 중 특정 반복문이나 함수가 얼마나 오래 걸리는지 대략적으로 확인하고 싶을 때 가장 빠르게 시도해볼 수 있는 방법이다.

## collections 모듈 심화

PY-10에서 스택/큐 구현에 사용한 `collections.deque`를 다뤘다. `collections` 모듈에는 이 외에도 유용한 자료구조가 더 있다.

**Counter — 개수 세기 전용 딕셔너리**

```python
from collections import Counter

words = "사과 바나나 사과 포도 바나나 사과".split()

counter = Counter(words)
print(counter)
print(counter["사과"])
print(counter.most_common(2))
```

```text
(base) C:\Users\guest\project> python counter_demo.py
Counter({'사과': 3, '바나나': 2, '포도': 1})
3
[('사과', 3), ('바나나', 2)]
```

- `Counter(이터러블)`은 각 요소가 몇 번 등장했는지 자동으로 세어 딕셔너리와 비슷한 형태로 반환한다. PY-11의 EX4에서 `word_count.get(word, 0) + 1` 패턴으로 직접 구현했던 단어 세기 로직을, `Counter` 하나로 대체할 수 있다.
- `most_common(n)`은 등장 횟수가 많은 순서대로 상위 `n`개를 `(값, 횟수)` 튜플의 리스트로 반환한다. 인자를 생략하면 전체 요소를 등장 횟수 내림차순으로 반환한다.
- 존재하지 않는 키를 조회해도 `KeyError` 대신 `0`을 반환한다는 점도 일반 딕셔너리와 다른 편리한 특징이다.

**defaultdict — 기본값이 있는 딕셔너리**

```python
from collections import defaultdict

groups = defaultdict(list)  # 존재하지 않는 키에 접근하면 자동으로 빈 리스트를 만들어준다

students = [("A반", "김철수"), ("B반", "이영희"), ("A반", "박민수"), ("A반", "최지은")]

for class_name, name in students:
    groups[class_name].append(name)

print(dict(groups))
print(groups["C반"])  # 없는 키를 조회해도 오류 대신 빈 리스트가 생성된다
print(dict(groups))
```

```text
(base) C:\Users\guest\project> python defaultdict_demo.py
{'A반': ['김철수', '박민수', '최지은'], 'B반': ['이영희']}
[]
{'A반': ['김철수', '박민수', '최지은'], 'B반': ['이영희'], 'C반': []}
```

- 일반 딕셔너리에서 `groups[class_name].append(name)`을 쓰려면 미리 `if class_name not in groups: groups[class_name] = []`처럼 키가 있는지 확인하고 없으면 초기화하는 코드가 필요하다. `defaultdict(list)`는 존재하지 않는 키에 접근하는 순간 자동으로 `list()`(빈 리스트)를 만들어 저장해주므로 이런 조건문을 생략할 수 있다.
- `defaultdict(int)`처럼 쓰면 없는 키를 조회할 때 `0`이 자동으로 채워지므로, `Counter`와 비슷하게 개수를 세는 용도로도 활용할 수 있다.
- 다만 `defaultdict`는 존재 여부를 확인하려고 조회만 해도 그 키가 실제로 생성된다는 점에 주의해야 한다(위 예제에서 `groups["C반"]`을 조회한 것만으로 `"C반"` 키가 빈 리스트와 함께 생겨난 것을 볼 수 있다).

**정리**: `collections` 모듈은 PY-10에서 다룬 `deque` 외에도 각 요소의 등장 횟수를 자동으로 세고 `most_common()`으로 상위 항목을 뽑아주는 `Counter`, 그리고 존재하지 않는 키에 접근할 때 지정한 타입의 기본값을 자동으로 생성해주는 `defaultdict`를 제공하며, 두 도구 모두 일반 딕셔너리로 직접 구현하면 반복적으로 등장하는 "키 존재 확인 후 초기화" 패턴을 간결하게 대체해준다.

## struct 모듈 — 이진 데이터 다루기

지금까지 다룬 파일은 대부분 텍스트였지만, 이미지 파일 헤더나 네트워크 프로토콜처럼 **정해진 바이트 구조**를 가진 이진(binary) 데이터를 다뤄야 할 때도 있다. `struct` 모듈은 파이썬 값과 고정된 형식의 바이트열(`bytes`)을 서로 변환해준다.

```python
import struct

packed = struct.pack("i", 1000)
print(packed, len(packed))

value = struct.unpack("i", packed)
print(value)
```

```text
(base) C:\Users\guest\project> python struct_basic.py
b'\xe8\x03\x00\x00' 4
(1000,)
```

- `struct.pack(포맷문자열, 값, ...)`은 파이썬 값을 지정한 형식에 맞춰 고정 길이의 `bytes`로 변환한다. `"i"`는 4바이트 부호 있는 정수(int)를 의미하며, `1000`이 4바이트짜리 이진 데이터로 변환된 것을 확인할 수 있다.
- `struct.unpack(포맷문자열, 바이트열)`은 반대로 이진 데이터를 다시 파이썬 값으로 복원하며, 값이 하나뿐이어도 항상 튜플로 반환한다.

**여러 필드를 한 번에 묶어 pack/unpack하기**

```python
import struct

data = struct.pack("<2sH", b"AB", 300)
print(data)

code, count = struct.unpack("<2sH", data)
print(code, count)
```

```text
(base) C:\Users\guest\project> python struct_multi.py
b'AB,\x01'
b'AB' 300
```

- 포맷 문자열은 여러 형식 지정자를 이어 붙여 한 번에 여러 값을 처리할 수 있다. `<`는 리틀 엔디안(little-endian) 바이트 순서를 뜻하고, `2s`는 2바이트 길이의 문자열(bytes), `H`는 2바이트 부호 없는 정수(unsigned short)를 의미한다.
- `unpack()`의 결과는 `pack()`에 넘긴 값의 개수와 순서에 맞춰 튜플로 반환되므로, 위 예제처럼 여러 변수로 한 번에 언패킹해서 받을 수 있다.

**정리**: `struct` 모듈은 정수·문자열 같은 파이썬 값을 `"i"`(정수)·`"s"`(문자열)·`"H"`(부호 없는 정수) 같은 형식 지정자로 이루어진 포맷 문자열에 맞춰 고정된 바이트 구조로 `pack()`하고, 반대로 그 바이트열을 다시 파이썬 값으로 `unpack()`할 수 있게 해주며, 파일 헤더 파싱이나 네트워크 프로토콜처럼 정해진 바이트 레이아웃을 가진 이진 데이터를 다룰 때 사용한다.

## timeit 모듈 — 정밀한 실행 시간 측정

앞서 `time.time()`으로 실행 시간을 측정하는 방법을 다뤘지만, 이 방식은 시스템의 다른 작업이나 한 번의 측정 오차에 영향을 받기 쉬워 아주 짧게 끝나는 코드 한 줄의 속도를 정밀하게 비교하기에는 부족하다. `timeit` 모듈은 같은 코드를 여러 번 반복 실행해 평균적인 실행 시간을 안정적으로 측정해준다.

```python
import timeit

result = timeit.timeit("x = [i ** 2 for i in range(100)]", number=100000)
print(f"{result:.4f}초 (환경에 따라 달라지는 예시 값)")
```

```text
(base) C:\Users\guest\project> python timeit_basic.py
0.8123초 (환경에 따라 달라지는 예시 값)
```

- `timeit.timeit(코드문자열, number=반복횟수)`는 첫 번째 인자로 넘긴 코드를 `number`에 지정한 횟수만큼 반복 실행한 뒤, 그 **총** 소요 시간을 초 단위 실수로 반환한다. 위 실행 결과(`0.8123초`)는 환경마다 달라지는 예시 값이며, 실제로 실행하면 사용 중인 컴퓨터 성능에 따라 다른 값이 나온다.
- `time.time()`으로 코드 한 줄을 한 번만 측정하면 그 순간 운영체제가 다른 작업에 CPU를 잠깐 내주는 등의 우연한 요인으로 결과가 들쭉날쭉할 수 있다. `timeit`은 같은 코드를 수만~수백만 번 반복 실행해 이런 우연한 오차의 영향을 줄이고, 테스트 대상 코드를 실행하는 동안 가비지 컬렉션을 기본적으로 비활성화하는 등 측정 자체의 정확도를 높이도록 설계되어 있어, 아주 짧게 끝나는 코드 조각들의 상대적인 속도를 비교할 때 `time.time()`보다 신뢰할 수 있는 결과를 준다.

**정리**: `timeit` 모듈은 코드 조각을 지정한 횟수만큼 반복 실행해 그 총 소요 시간을 측정하는 `timeit.timeit(코드, number=횟수)`를 제공하며, 한 번의 측정에 우연한 오차가 섞이기 쉬운 `time.time()`과 달리 반복 실행과 측정 환경 정리를 통해 짧은 코드 조각의 성능을 비교하는 마이크로 벤치마크에 더 신뢰할 수 있는 결과를 제공한다.

## zipfile · gzip 모듈 — 압축

파이썬 표준 라이브러리는 별도 설치 없이 파일을 압축·해제하는 기능도 제공한다. 여러 파일을 하나로 묶는 zip 아카이브는 `zipfile` 모듈로, 단일 스트림 압축은 `gzip` 모듈로 다룰 수 있다.

**zipfile — zip 아카이브 만들고 읽기**

```python
import zipfile

with zipfile.ZipFile("archive.zip", "w") as zf:
    zf.writestr("hello.txt", "안녕하세요\n")
    zf.writestr("memo.txt", "압축 테스트 메모\n")

with zipfile.ZipFile("archive.zip", "r") as zf:
    print(zf.namelist())
    print(zf.read("hello.txt").decode("utf-8"))
```

```text
(base) C:\Users\guest\project> python zipfile_demo.py
['hello.txt', 'memo.txt']
안녕하세요
```

- `zipfile.ZipFile(경로, "w")`로 열면 새 zip 아카이브를 만들 수 있고, `writestr(파일이름, 내용)`은 실제 파일 없이 문자열(또는 바이트열) 내용을 아카이브 안에 바로 파일로 추가한다. 디스크에 이미 존재하는 파일을 담고 싶다면 `zf.write(파일경로)`를 사용한다.
- `zipfile.ZipFile(경로, "r")`로 다시 열면 `namelist()`로 아카이브 안에 담긴 파일 목록을 확인하고, `read(파일이름)`으로 특정 파일의 내용을 바이트열로 꺼낼 수 있다. 텍스트로 다루려면 `.decode("utf-8")`로 디코딩해야 한다.

**gzip — 단일 데이터 압축·해제**

```python
import gzip

data = "반복되는 로그 데이터 " * 100
original_bytes = data.encode("utf-8")
compressed = gzip.compress(original_bytes)

print("원본보다 작아졌는가:", len(compressed) < len(original_bytes))

decompressed = gzip.decompress(compressed).decode("utf-8")
print("압축 해제 결과가 원본과 같은가:", decompressed == data)
```

```text
(base) C:\Users\guest\project> python gzip_demo.py
원본보다 작아졌는가: True
압축 해제 결과가 원본과 같은가: True
```

- `gzip.compress(바이트열)`은 바이트 데이터를 gzip 형식으로 압축한 바이트열을 반환하고, `gzip.decompress(압축된바이트열)`은 이를 원래 바이트열로 되돌린다. 같은 내용이 반복되는 데이터일수록 압축 효율이 높아 원본보다 훨씬 작아지는 경우가 많다.
- 파일 단위로 다루고 싶다면 `gzip.open(경로, "wb")`/`gzip.open(경로, "rb")`을 `open()`과 같은 방식으로 사용할 수 있다.

**정리**: `zipfile`은 여러 파일을 하나의 zip 아카이브로 묶거나(`writestr()`/`write()`) 그 안의 목록과 내용을 꺼내는(`namelist()`/`read()`) 기능을, `gzip`은 바이트 데이터 하나를 통째로 압축·해제하는(`compress()`/`decompress()`) 기능을 제공하며, 둘 다 외부 프로그램이나 추가 설치 없이 표준 라이브러리만으로 로그 파일이나 백업 데이터를 압축해 저장 공간을 절약하는 데 사용할 수 있다.

## 실습 예제 (EX1~EX4)

**EX1) glob으로 특정 확장자 파일만 찾아 개수 세기**
- 여러 확장자의 파일이 섞인 폴더에서 `.txt` 파일만 찾아 개수를 출력한다.

```python
# ex01_glob_count.py
import glob
import os

os.makedirs("reports", exist_ok=True)
for name in ["jan.txt", "feb.txt", "mar.csv", "summary.log"]:
    with open(os.path.join("reports", name), "w", encoding="utf-8") as f:
        f.write("내용\n")

txt_files = glob.glob("reports/*.txt")
print(f"txt 파일 {len(txt_files)}개:", sorted(txt_files))
```

```text
(base) C:\Users\guest\project> python ex01_glob_count.py
txt 파일 2개: ['reports\\feb.txt', 'reports\\jan.txt']
```

**EX2) 정규표현식으로 이메일 주소만 추출하기**
- 여러 줄의 텍스트에서 이메일 형식의 문자열만 모두 찾아 리스트로 출력한다.

```python
# ex02_extract_emails.py
import re

text = """
담당자: 김철수 (chulsoo@example.com)
담당자: 이영희 (younghee.lee@company.co.kr)
문의 전화: 02-1234-5678
"""

emails = re.findall(r"[\w.]+@[\w.]+", text)
print(emails)
```

```text
(base) C:\Users\guest\project> python ex02_extract_emails.py
['chulsoo@example.com', 'younghee.lee@company.co.kr']
```

**EX3) 오늘부터 30일 후가 무슨 요일인지 계산하기**
- 특정 날짜에 `timedelta`를 더해 30일 후의 날짜를 계산하고 형식에 맞춰 출력한다.

```python
# ex03_thirty_days_later.py
import datetime

base_date = datetime.datetime(2026, 9, 10)
target_date = base_date + datetime.timedelta(days=30)

print("기준일:", base_date.strftime("%Y-%m-%d"))
print("30일 후:", target_date.strftime("%Y-%m-%d"))
```

```text
(base) C:\Users\guest\project> python ex03_thirty_days_later.py
기준일: 2026-09-10
30일 후: 2026-10-10
```

**EX4) Counter와 defaultdict로 로그 분석하기**
- 로그 목록에서 상태 코드별 등장 횟수를 세고, 상태 코드별로 관련 URL 목록을 모아 정리한다.

```python
# ex04_log_analysis.py
from collections import Counter, defaultdict

logs = [
    ("200", "/home"),
    ("404", "/missing"),
    ("200", "/about"),
    ("500", "/error"),
    ("200", "/home"),
    ("404", "/old-page"),
]

status_counter = Counter(status for status, _ in logs)
url_by_status = defaultdict(list)
for status, url in logs:
    url_by_status[status].append(url)

print("상태 코드별 횟수:", status_counter)
print("가장 많은 상태 코드:", status_counter.most_common(1))
for status, urls in url_by_status.items():
    print(f"{status}: {urls}")
```

```text
(base) C:\Users\guest\project> python ex04_log_analysis.py
상태 코드별 횟수: Counter({'200': 3, '404': 2, '500': 1})
가장 많은 상태 코드: [('200', 3)]
200: ['/home', '/about', '/home']
404: ['/missing', '/old-page']
500: ['/error']
```

**정리**: EX1~EX4는 `glob`으로 조건에 맞는 파일만 찾아 개수를 세는 방법, `re` 모듈로 텍스트에서 이메일 같은 특정 패턴을 추출하는 방법, `datetime`과 `timedelta`로 날짜 연산을 수행하는 방법, 그리고 `Counter`와 `defaultdict`를 함께 사용해 로그 데이터를 집계하고 분류하는 방법까지, 이 문서에서 다룬 표준 라이브러리 모듈들을 실무에 가까운 형태로 조합해보는 예제다.

[Python 13 — 클래스 심화](13-classes-advanced.md) · [Python 15 — 가상 환경 심화와 부동소수점](15-venv-precision.md)

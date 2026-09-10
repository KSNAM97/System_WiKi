# Python 11 — 입력과 출력

## f-string 포맷 스펙 심화

PY-01에서 `print()`의 `sep`, `end` 옵션을 다뤘다. 이번에는 출력값 자체의 모양을 다듬는 **f-string 포맷 스펙**을 살펴본다. f-string은 `f"...{표현식:포맷스펙}..."` 형태로, 콜론(`:`) 뒤에 정렬·자릿수·진법 등을 지정할 수 있다.

**정렬**

```python
names = ["김", "이영희", "박민수"]

for name in names:
    print(f"[{name:<5}]", f"[{name:>5}]", f"[{name:^5}]")
```

```text
(base) C:\Users\guest\project> python fstring_align.py
[김    ] [    김] [  김  ]
[이영희  ] [  이영희] [ 이영희 ]
[박민수  ] [  박민수] [ 박민수 ]
```

- `<`는 왼쪽 정렬, `>`는 오른쪽 정렬, `^`는 가운데 정렬이며, 그 뒤의 숫자(`5`)는 전체 출력 폭(문자 개수)을 의미한다. 지정한 폭보다 문자열이 짧으면 남는 칸이 공백으로 채워진다.
- 채울 문자를 공백이 아닌 다른 문자로 바꾸고 싶다면 정렬 기호 앞에 그 문자를 적는다: `f"{name:*^10}"`처럼 쓰면 `*`로 채워진다.

```python
print(f"{'제목':*^20}")
print(f"{42:0>5}")
```

```text
(base) C:\Users\guest\project> python fstring_fill.py
********제목********
00042
```

**소수점 자릿수**

```python
pi = 3.14159265
price = 12345.6789

print(f"{pi:.2f}")
print(f"{pi:.4f}")
print(f"{price:,.2f}")
```

```text
(base) C:\Users\guest\project> python fstring_decimal.py
3.14
3.1416
12,345.68
```

- `.2f`는 소수점 이하 2자리까지 고정 표시(반올림)하라는 의미이며, `f`는 고정 소수점(fixed-point) 표기를 뜻한다.
- `,`를 함께 쓰면(`,.2f`) 천 단위마다 콤마 구분자를 추가하여 큰 숫자를 읽기 쉽게 만든다.

**정수 진법 변환**

```python
n = 255

print(f"{n:b}")   # 2진수
print(f"{n:o}")   # 8진수
print(f"{n:x}")   # 16진수(소문자)
print(f"{n:X}")   # 16진수(대문자)
print(f"{n:#x}")  # 0x 접두어 포함
```

```text
(base) C:\Users\guest\project> python fstring_base.py
11111111
377
ff
FF
0xff
```

**퍼센트 표시**

```python
ratio = 0.8567

print(f"{ratio:.1%}")
print(f"{ratio:.2%}")
```

```text
(base) C:\Users\guest\project> python fstring_percent.py
85.7%
85.67%
```

- `%`를 포맷 스펙에 쓰면 값에 100을 곱하고 `%` 기호를 붙여 표시해준다. 값을 직접 100배 계산할 필요가 없다.

**정리**: f-string은 `{표현식:포맷스펙}` 형태로 값을 원하는 폭에 맞춰 정렬(`<`/`>`/`^`)하거나, 채움 문자를 지정하거나, `.Nf`로 소수점 자릿수를 고정하고 `,`로 천 단위 구분자를 추가하거나, `b`/`o`/`x`/`X`로 진법을 바꾸거나, `%`로 백분율을 표시하는 등 값 자체의 계산 없이 표현 방식만 다양하게 바꿀 수 있는 미니 언어를 제공하며, 표나 보고서처럼 정렬과 자릿수가 중요한 출력을 만들 때 특히 유용하다.

## str.format() 메서드

f-string이 등장하기 전(파이썬 3.6 이전)부터 사용되던 방식이 `str.format()` 메서드다. f-string과 포맷 스펙 문법 자체는 거의 동일하지만, 문자열 리터럴 안에 변수를 직접 쓰는 대신 `{}` 자리표시자에 값을 나중에 채워 넣는 방식이다.

```python
template = "{}님은 {}세이고, {} 도시에 거주합니다."
print(template.format("김철수", 25, "서울"))
```

```text
(base) C:\Users\guest\project> python format_basic.py
김철수님은 25세이고, 서울 도시에 거주합니다.
```

- 빈 `{}`는 `format()`에 넘긴 인자를 순서대로 하나씩 채운다.

**위치 인덱스와 이름으로 지정하기**

```python
template = "{0}은(는) {1}점, {0}의 등수는 {2}등입니다."
print(template.format("김철수", 85, 3))

named = "{name}님, 주문번호 {order_id}가 접수되었습니다."
print(named.format(name="이영희", order_id="A1024"))
```

```text
(base) C:\Users\guest\project> python format_index_named.py
김철수은(는) 85점, 김철수의 등수는 3등입니다.
이영희님, 주문번호 A1024가 접수되었습니다.
```

- `{0}`, `{1}`처럼 숫자로 인덱스를 지정하면 같은 값을 여러 번 재사용할 수 있고, `{name}`처럼 이름을 지정하면 `format(name=...)`으로 어떤 값이 어디에 들어가는지 명확하게 알 수 있다.

**포맷 스펙도 동일하게 사용 가능**

```python
print("{:.2f}".format(3.14159))
print("{:>10}".format("결과"))
print("{:,}".format(1234567))
```

```text
(base) C:\Users\guest\project> python format_spec.py
3.14
        결과
1,234,567
```

- f-string의 `:` 뒤 포맷 스펙과 완전히 동일한 문법을 `str.format()`에서도 사용할 수 있다. f-string은 값이 변수로 바로 존재할 때 더 간결하고, `str.format()`은 템플릿 문자열을 미리 정의해두고 값을 나중에 여러 번 다르게 채워 넣어야 할 때(예: 다국어 메시지 템플릿) 여전히 유용하다.

**정리**: `str.format()`은 `{}` 자리표시자를 가진 템플릿 문자열에 `.format()` 인자로 값을 나중에 채워 넣는 방식으로, 위치 인덱스(`{0}`)나 이름(`{name}`)으로 자리표시자를 지정할 수 있고 f-string과 동일한 포맷 스펙을 사용할 수 있으며, 값을 즉석에서 바로 넣는 f-string과 달리 템플릿을 미리 만들어두고 재사용해야 하는 상황에 적합하다.

## % 포맷팅 (레거시 문법)

f-string(파이썬 3.6+)과 `str.format()`이 등장하기 훨씬 전부터 있었던 가장 오래된 문자열 포맷팅 방식이 **% 포맷팅**이다. `"형식문자열" % (값1, 값2, ...)` 형태로 사용한다.

```python
name = "김철수"
age = 25

print("%s는 %d살" % (name, age))
```

```text
(base) C:\Users\guest\project> python percent_format.py
김철수는 25살
```

- `%s`는 문자열, `%d`는 정수를 끼워 넣을 자리를 나타내는 **변환 지정자(conversion specifier)**다. `%` 연산자 오른쪽에 튜플로 값을 순서대로 나열하면 왼쪽 형식 문자열의 `%s`, `%d` 자리에 차례로 채워진다.
- 값이 하나뿐이면 튜플로 감싸지 않고 값 자체를 바로 써도 된다: `"점수: %d" % 90`.

**자주 쓰이는 변환 지정자와 소수점 자릿수**

```python
price = 12345.6789

print("%.2f" % price)
print("이름: %(name)s, 나이: %(age)d" % {"name": "이영희", "age": 30})
```

```text
(base) C:\Users\guest\project> python percent_format_more.py
12345.68
이름: 이영희, 나이: 30
```

- `%.2f`는 f-string의 `:.2f`와 마찬가지로 소수점 이하 2자리까지 반올림해 표시한다.
- `%(이름)s`처럼 괄호 안에 키 이름을 적고 오른쪽에 딕셔너리를 넘기면, 순서가 아니라 이름으로 값을 채울 수도 있다.

**레거시 문법으로 취급되는 이유**

```python
# 값 개수가 안 맞으면 오류가 발생하고 에러 메시지도 직관적이지 않다
"%s는 %d살" % ("김철수",)
```

```text
Traceback (most recent call last):
  File "percent_format_error.py", line 1, in <module>
    "%s는 %d살" % ("김철수",)
TypeError: not enough arguments for format string
```

- `%` 포맷팅은 값과 자리 수가 정확히 맞아야 하고, 튜플이나 딕셔너리를 명시적으로 감싸야 하는 등 f-string에 비해 번거롭고 오류 메시지도 상대적으로 불친절하다. 파이썬 공식 문서에서도 새 코드에서는 f-string이나 `str.format()` 사용을 권장하고 있다.
- 다만 오래된 코드베이스나 일부 로깅 라이브러리에서는 여전히 `%` 포맷팅이 남아 있는 경우가 많으므로, 직접 작성할 때는 지양하더라도 읽고 이해할 수 있어야 한다.

**정리**: `% 포맷팅`은 `"형식문자열" % 값들` 형태로 `%s`(문자열)·`%d`(정수)·`%.2f`(소수점) 같은 변환 지정자에 값을 채워 넣는 가장 오래된 문자열 포맷팅 방식으로, f-string이나 `str.format()`보다 값 개수 검증이 엄격하지 않고 오류 메시지도 불친절해 새 코드에서는 권장되지 않지만, 오래된 코드나 일부 라이브러리에서 여전히 등장하므로 읽을 줄은 알아두어야 하는 레거시 문법이다.

## 파일 읽고 쓰기 — open()과 with

파이썬에서 파일을 다루려면 먼저 `open()`으로 파일을 **연 뒤**, 작업이 끝나면 반드시 **닫아야** 한다. `with` 문을 사용하면 이 닫는 과정을 자동으로 처리해준다.

```python
f = open("memo.txt", "w", encoding="utf-8")
f.write("첫 번째 줄\n")
f.write("두 번째 줄\n")
f.close()
```

```text
(base) C:\Users\guest\project> python open_close_manual.py
(base) C:\Users\guest\project> type memo.txt
첫 번째 줄
두 번째 줄
```

- `open(파일경로, 모드, encoding="utf-8")`로 파일 객체를 얻고, 작업이 끝나면 `close()`로 반드시 닫아야 한다. 만약 `write()`와 `close()` 사이에서 오류가 발생하면 `close()`가 실행되지 않아 파일이 제대로 저장되지 않거나 다른 프로그램이 그 파일을 사용하지 못하는 문제가 생길 수 있다.

**with 문으로 안전하게 다루기**

```python
with open("memo.txt", "w", encoding="utf-8") as f:
    f.write("첫 번째 줄\n")
    f.write("두 번째 줄\n")
# with 블록을 벗어나는 순간 파일이 자동으로 닫힌다
```

```text
(base) C:\Users\guest\project> python with_statement.py
(base) C:\Users\guest\project> type memo.txt
첫 번째 줄
두 번째 줄
```

- `with open(...) as f:` 블록 안에서 오류가 발생하더라도, 블록을 벗어나는 순간 파이썬이 자동으로 `f.close()`를 호출해준다. PY-08에서 다룬 `finally`가 항상 실행되는 것과 같은 원리로 자원을 안전하게 정리해주는 것이며, 실무에서는 `open()`을 `close()`와 직접 짝지어 쓰기보다 항상 `with` 문을 사용하는 것이 표준적인 방법이다.

**주요 파일 모드**

| 모드 | 의미 |
|------|------|
| `"r"` | 읽기 전용(기본값). 파일이 없으면 오류 |
| `"w"` | 쓰기 전용. 파일이 있으면 내용을 덮어씀, 없으면 새로 생성 |
| `"a"` | 추가(append). 파일 끝에 이어서 씀, 없으면 새로 생성 |
| `"x"` | 새로 생성. 파일이 이미 있으면 오류 |
| `"rb"`, `"wb"` | 이진(binary) 모드로 읽기/쓰기. 이미지·동영상 등 텍스트가 아닌 파일에 사용 |

```python
with open("memo.txt", "a", encoding="utf-8") as f:
    f.write("세 번째 줄(추가됨)\n")

with open("memo.txt", "r", encoding="utf-8") as f:
    print(f.read())
```

```text
(base) C:\Users\guest\project> python append_mode.py
첫 번째 줄
두 번째 줄
세 번째 줄(추가됨)
```

- `"w"` 모드로 열면 기존 내용이 전부 사라지고 새로 쓰이므로 주의해야 하며, 기존 내용을 유지한 채 뒤에 이어 쓰고 싶다면 `"a"` 모드를 사용해야 한다.

**정리**: 파일을 다룰 때는 `open(경로, 모드, encoding="utf-8")`로 파일을 열고 작업 후 반드시 닫아야 하는데, `with open(...) as f:` 문을 사용하면 블록을 벗어날 때(오류가 나더라도) 파이썬이 자동으로 파일을 닫아주므로 직접 `close()`를 호출하는 것보다 안전하며, `"r"`(읽기)·`"w"`(덮어쓰기)·`"a"`(추가)·`"rb"`/`"wb"`(이진) 중 상황에 맞는 모드를 선택하는 것이 파일 입출력의 기본이다.

## 파일 객체 메서드

`open()`으로 얻은 파일 객체는 내용을 읽거나 쓰는 여러 메서드를 제공한다.

**read() — 전체 내용을 문자열로 읽기**

```python
with open("memo.txt", "r", encoding="utf-8") as f:
    content = f.read()
    print(content)
    print(type(content))
```

```text
(base) C:\Users\guest\project> python file_read.py
첫 번째 줄
두 번째 줄
세 번째 줄(추가됨)

<class 'str'>
```

- `read()`는 파일 전체 내용을 하나의 문자열로 한 번에 반환한다. 파일이 매우 크면 메모리를 많이 사용하므로 주의해야 한다.

**readline() — 한 줄씩 읽기**

```python
with open("memo.txt", "r", encoding="utf-8") as f:
    line1 = f.readline()
    line2 = f.readline()
    print(repr(line1))
    print(repr(line2))
```

```text
(base) C:\Users\guest\project> python file_readline.py
'첫 번째 줄\n'
'두 번째 줄\n'
```

- `readline()`을 호출할 때마다 파일에서 한 줄씩 읽어오며, 커서가 자동으로 다음 줄로 이동한다. 줄 끝의 개행 문자(`\n`)가 그대로 포함되어 반환되는 점에 유의한다.

**readlines() — 모든 줄을 리스트로 읽기**

```python
with open("memo.txt", "r", encoding="utf-8") as f:
    lines = f.readlines()
    print(lines)

    for line in lines:
        print(line.strip())
```

```text
(base) C:\Users\guest\project> python file_readlines.py
['첫 번째 줄\n', '두 번째 줄\n', '세 번째 줄(추가됨)\n']
첫 번째 줄
두 번째 줄
세 번째 줄(추가됨)
```

- `readlines()`는 파일의 모든 줄을 각각 문자열 요소로 담은 리스트로 반환한다. 각 줄에 개행 문자가 포함되어 있으므로 `strip()`으로 제거하고 사용하는 경우가 많다.
- 사실 `with open(...) as f:` 블록에서 `for line in f:`로 바로 순회하면 `readlines()` 없이도 한 줄씩 순회할 수 있으며, 파일 전체를 메모리에 올리지 않아 큰 파일에 더 적합하다.

```python
with open("memo.txt", "r", encoding="utf-8") as f:
    for line in f:
        print(line.strip())
```

**write() — 문자열 쓰기**

```python
with open("log.txt", "w", encoding="utf-8") as f:
    f.write("로그 시작\n")
    for i in range(3):
        f.write(f"이벤트 {i}: 정상 처리\n")
```

```text
(base) C:\Users\guest\project> python file_write.py
(base) C:\Users\guest\project> type log.txt
로그 시작
이벤트 0: 정상 처리
이벤트 1: 정상 처리
이벤트 2: 정상 처리
```

- `write()`는 `print()`와 달리 자동으로 줄바꿈을 추가하지 않으므로, 줄을 나눠 쓰고 싶다면 문자열 끝에 직접 `\n`을 붙여야 한다.

**정리**: 파일 객체는 전체 내용을 한 번에 문자열로 가져오는 `read()`, 한 줄씩 순차적으로 가져오는 `readline()`, 모든 줄을 리스트로 가져오는 `readlines()`, 그리고 문자열을 파일에 쓰는 `write()`(자동 개행 없음)를 제공하며, 대용량 파일은 `read()`나 `readlines()`로 전체를 메모리에 올리기보다 `for line in f:`로 한 줄씩 순회하는 방식이 더 안전하다.

## json 모듈로 구조적 데이터 다루기

**JSON(JavaScript Object Notation)**은 파이썬의 딕셔너리·리스트와 매우 비슷한 구조를 가진, 언어에 상관없이 널리 쓰이는 데이터 교환 형식이다. 파이썬 표준 라이브러리 `json` 모듈로 파이썬 객체와 JSON 텍스트를 서로 변환할 수 있다.

**json.dump()로 파일에 저장하기**

```python
import json

user = {
    "name": "김철수",
    "age": 25,
    "is_active": True,
    "hobbies": ["독서", "등산"],
}

with open("user.json", "w", encoding="utf-8") as f:
    json.dump(user, f, ensure_ascii=False, indent=2)
```

```text
(base) C:\Users\guest\project> python json_dump.py
(base) C:\Users\guest\project> type user.json
{
  "name": "김철수",
  "age": 25,
  "is_active": true,
  "hobbies": [
    "독서",
    "등산"
  ]
}
```

- `json.dump(파이썬객체, 파일객체, ...)`는 딕셔너리·리스트 등으로 이루어진 파이썬 객체를 JSON 형식의 텍스트로 변환해 파일에 바로 써준다.
- `ensure_ascii=False`를 지정하지 않으면 한글 같은 비 ASCII 문자가 `\uXXXX` 형태의 이스케이프 코드로 저장되므로, 사람이 읽을 수 있는 형태로 저장하려면 이 옵션을 꼭 지정해야 한다.
- `indent=2`는 들여쓰기 폭을 지정해 결과를 읽기 좋은 형태로 정렬해준다. 생략하면 한 줄로 압축되어 저장된다.
- 파이썬의 `True`/`False`/`None`은 각각 JSON의 `true`/`false`/`null`로 자동 변환된다.

**json.load()로 파일에서 불러오기**

```python
import json

with open("user.json", "r", encoding="utf-8") as f:
    loaded = json.load(f)

print(loaded)
print(type(loaded))
print(loaded["name"], loaded["hobbies"][0])
```

```text
(base) C:\Users\guest\project> python json_load.py
{'name': '김철수', 'age': 25, 'is_active': True, 'hobbies': ['독서', '등산']}
<class 'dict'>
김철수 독서
```

- `json.load(파일객체)`는 JSON 텍스트가 담긴 파일을 읽어 파이썬 딕셔너리(또는 리스트)로 변환해준다. 변환된 객체는 일반 딕셔너리와 똑같이 키로 접근하고 순회할 수 있다.

**json.dumps() / json.loads() — 파일 없이 문자열로 변환하기**

```python
import json

data = {"status": "ok", "code": 200}

json_text = json.dumps(data, ensure_ascii=False)
print(json_text, type(json_text))

parsed = json.loads(json_text)
print(parsed, type(parsed))
```

```text
(base) C:\Users\guest\project> python json_dumps_loads.py
{"status": "ok", "code": 200} <class 'str'>
{'status': 'ok', 'code': 200} <class 'dict'>
```

- 끝에 `s`가 붙은 `dumps()`/`loads()`는 파일이 아니라 **문자열**을 대상으로 동작한다. 웹 API 응답처럼 파일이 아니라 문자열 형태로 JSON 데이터를 주고받을 때 사용한다. 이름 구분: `dump`/`load`는 파일 객체, `dumps`/`loads`는 문자열(string)이라고 기억하면 된다.

**정리**: `json` 모듈은 파이썬의 딕셔너리·리스트를 JSON 텍스트로, 또는 그 반대로 변환해주며, 파일과 직접 주고받을 때는 `json.dump()`/`json.load()`를, 문자열로 직접 다룰 때는 `json.dumps()`/`json.loads()`를 사용하고, 한글이 포함된 데이터를 사람이 읽을 수 있는 형태로 저장하려면 `ensure_ascii=False`와 `indent` 옵션을 함께 지정하는 것이 실무에서 자주 쓰는 조합이다.

## 실습 예제 (EX1~EX4)

**EX1) f-string으로 표 형태 출력 만들기**
- 상품명과 가격 목록을 받아, 이름은 왼쪽 정렬 10칸, 가격은 오른쪽 정렬에 천 단위 콤마를 붙여 표 형태로 출력한다.

```python
# ex01_price_table.py
products = [("노트북", 1200000), ("마우스", 15000), ("키보드", 45000)]

for name, price in products:
    print(f"{name:<10}{price:>12,}원")
```

```text
(base) C:\Users\guest\project> python ex01_price_table.py
노트북       1,200,000원
마우스        15,000원
키보드        45,000원
```

**EX2) 여러 줄 텍스트 파일 쓰고 읽어서 줄 수 세기**
- 학생 이름 리스트를 한 줄에 한 명씩 파일로 저장한 뒤, 다시 읽어서 총 몇 줄인지 세는 코드를 작성한다.

```python
# ex02_count_lines.py
students = ["김철수", "이영희", "박민수", "최지은"]

with open("students.txt", "w", encoding="utf-8") as f:
    for name in students:
        f.write(name + "\n")

with open("students.txt", "r", encoding="utf-8") as f:
    lines = f.readlines()

print(f"총 {len(lines)}명이 저장되어 있습니다.")
for i, line in enumerate(lines, start=1):
    print(f"{i}. {line.strip()}")
```

```text
(base) C:\Users\guest\project> python ex02_count_lines.py
총 4명이 저장되어 있습니다.
1. 김철수
2. 이영희
3. 박민수
4. 최지은
```

**EX3) 딕셔너리를 JSON 파일로 저장하고 검증하기**
- 여러 명의 성적 정보를 담은 딕셔너리를 JSON 파일로 저장한 뒤, 다시 불러와 원본과 내용이 같은지 확인한다.

```python
# ex03_json_roundtrip.py
import json

scores = {"김철수": 85, "이영희": 92, "박민수": 78}

with open("scores.json", "w", encoding="utf-8") as f:
    json.dump(scores, f, ensure_ascii=False, indent=2)

with open("scores.json", "r", encoding="utf-8") as f:
    loaded_scores = json.load(f)

print(loaded_scores)
print("원본과 동일한가:", scores == loaded_scores)
```

```text
(base) C:\Users\guest\project> python ex03_json_roundtrip.py
{'김철수': 85, '이영희': 92, '박민수': 78}
원본과 동일한가: True
```

**EX4) 텍스트 파일 읽어서 단어별 등장 횟수를 JSON으로 저장하기**
- 텍스트 파일의 내용을 읽어 공백 기준으로 단어를 나눈 뒤, 각 단어가 몇 번 등장했는지 세어 그 결과를 JSON 파일로 저장하도록 작성한다.

```python
# ex04_word_count.py
import json

with open("memo.txt", "w", encoding="utf-8") as f:
    f.write("파이썬 공부 재미있다 파이썬 문법 공부\n")

with open("memo.txt", "r", encoding="utf-8") as f:
    text = f.read()

words = text.split()
word_count = {}
for word in words:
    word_count[word] = word_count.get(word, 0) + 1

with open("word_count.json", "w", encoding="utf-8") as f:
    json.dump(word_count, f, ensure_ascii=False, indent=2)

print(word_count)
```

```text
(base) C:\Users\guest\project> python ex04_word_count.py
{'파이썬': 2, '공부': 2, '재미있다': 1, '문법': 1}
```

- `word_count.get(word, 0) + 1`은 PY-04에서 다룬 `get()`의 기본값 기능을 활용해, 처음 등장한 단어는 0에서 시작해 1을 더하고 이미 있던 단어는 기존 개수에 1을 더하는 패턴이다.

**정리**: EX1~EX4는 f-string 포맷 스펙으로 표 형태 출력을 정렬하는 방법, `with`와 파일 쓰기/읽기로 텍스트 데이터를 안전하게 저장·조회하는 방법, `json.dump()`/`json.load()`로 파이썬 딕셔너리를 파일에 저장했다가 그대로 복원하는 방법, 텍스트를 분석해 만든 결과를 JSON으로 저장하는 방법까지 이 문서에서 다룬 입출력 심화 개념을 직접 코드로 확인해보는 예제다.

[Python 10 — 자료구조 심화](10-data-structures-advanced.md)

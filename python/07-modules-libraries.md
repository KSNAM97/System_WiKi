# Python 07 — 모듈과 라이브러리

## 모듈이란 무엇인가

**모듈(module)**은 함수, 클래스, 변수 등을 한데 모아둔 파이썬 파일(`.py`) 하나를 가리킨다. 지금까지 작성한 `greet.py`, `student_manager.py` 같은 파일도 모두 모듈이다.

```python
# calc.py
def add(a, b):
    return a + b

def subtract(a, b):
    return a - b

PI = 3.14
```

다른 파일에서 이 `calc.py`에 정의된 함수와 변수를 가져다 쓰고 싶다면, 매번 코드를 복사해 붙여넣는 대신 **import** 문으로 모듈 자체를 불러와 재사용할 수 있다.

```python
# main.py
import calc

print(calc.add(3, 5))
print(calc.subtract(10, 4))
print(calc.PI)
```

```text
(base) C:\Users\guest\project> python main.py
8
6
3.14
```

- `calc.py`와 `main.py`가 같은 폴더에 있어야 `import calc`로 바로 찾을 수 있다.
- 모듈 안의 함수·변수는 `모듈이름.이름` 형태로 접근한다.

**정리**: 모듈은 함수·클래스·변수를 담아둔 파이썬 파일 하나이며, 코드를 여러 파일에 중복해서 쓰는 대신 `import`로 모듈을 불러와 재사용하면 프로젝트를 여러 파일로 깔끔하게 나눠서 관리할 수 있다.

## import 문법

**모듈 전체를 가져오기**

```python
import math

print(math.sqrt(16))
print(math.pi)
```

```text
(base) C:\Users\guest\project> python import_basic.py
4.0
3.141592653589793
```

- `import 모듈이름`은 모듈 전체를 가져오며, 그 안의 요소는 항상 `모듈이름.요소이름` 형태로 접근해야 한다.

**as로 별칭 지정하기**

모듈 이름이 길거나 다른 이름과 겹칠 때는 `as`로 별칭(alias)을 지정할 수 있다.

```python
import math as m

print(m.sqrt(25))
```

```text
(base) C:\Users\guest\project> python import_as.py
5.0
```

- 별칭을 지정하면 이후 코드에서 원래 모듈 이름 대신 별칭으로만 접근해야 한다. `numpy`를 `np`로, `pandas`를 `pd`로 줄여 쓰는 것은 데이터 분석 분야에서 관용적으로 굳어진 별칭이다.

**정리**: `import 모듈이름`은 모듈 전체를 가져와 `모듈이름.요소` 형태로 접근하게 하며, `import 모듈이름 as 별칭`으로 더 짧거나 관용적인 이름을 붙여 사용할 수 있다.

## 특정 함수·클래스·변수만 가져오기

모듈 전체가 아니라 그 안의 특정 요소만 필요하다면 `from ... import ...` 문법을 사용한다.

```python
from math import sqrt, pi

print(sqrt(16))
print(pi)
```

```text
(base) C:\Users\guest\project> python from_import.py
4.0
3.141592653589793
```

- `from 모듈이름 import 이름1, 이름2`로 가져오면, 이후 코드에서는 `math.sqrt()`가 아니라 `sqrt()`처럼 모듈 이름 없이 바로 사용할 수 있다.

**from import에도 별칭을 붙일 수 있다**

```python
from math import sqrt as square_root

print(square_root(9))
```

```text
(base) C:\Users\guest\project> python from_import_as.py
3.0
```

**이름이 겹칠 때 주의할 점**

```python
def sqrt(x):
    return "내가 만든 함수"

from math import sqrt   # 위에서 만든 sqrt 함수를 덮어씀

print(sqrt(16))
```

```text
(base) C:\Users\guest\project> python from_import_shadow.py
4.0
```

- `from ... import 이름` 방식은 해당 이름을 현재 코드의 이름 공간에 그대로 가져오므로, 같은 이름의 변수·함수가 이미 있었다면 덮어써진다. 이런 충돌을 피하기 위해서는 `import math`로 모듈 전체를 가져와 `math.sqrt()`처럼 명시적으로 접근하는 방식이 더 안전할 때가 많다.

**정리**: `from 모듈이름 import 이름`은 모듈 전체가 아니라 필요한 함수·클래스·변수만 골라서 가져오며 이후 모듈 이름 없이 바로 사용할 수 있지만, 현재 코드의 이름 공간을 그대로 덮어쓰므로 이름이 겹치는 경우를 주의해야 하고 필요하면 `as`로 별칭을 붙여 충돌을 피할 수 있다.

## 패키지와 점(dot) 표기법

**패키지(package)**는 여러 모듈을 폴더 단위로 묶어놓은 것이다. 폴더 안에 관련된 모듈 파일들을 모아두면, 점(`.`) 표기법으로 그 안의 특정 모듈에 접근할 수 있다.

```text
project/
├── main.py
└── shapes/
    ├── __init__.py
    ├── rectangle.py
    └── circle.py
```

```python
# shapes/rectangle.py
def area(width, height):
    return width * height
```

```python
# shapes/circle.py
def area(radius):
    return 3.14 * radius ** 2
```

```python
# main.py
import shapes.rectangle
import shapes.circle

print(shapes.rectangle.area(4, 5))
print(shapes.circle.area(3))
```

```text
(base) C:\Users\guest\project> python main.py
20
28.259999999999998
```

- `shapes` 폴더 안의 `__init__.py`는 이 폴더가 일반 폴더가 아니라 파이썬 패키지임을 알리는 파일이며, 내용이 비어 있어도 된다(파이썬 최신 버전에서는 없어도 동작하는 경우가 많지만, 명시적으로 만들어두는 것이 관례다).
- `import shapes.rectangle`처럼 점으로 경로를 이어서 패키지 안의 특정 모듈을 지정한다.

**from으로 패키지 안 모듈의 함수를 바로 가져오기**

```python
from shapes.rectangle import area as rect_area
from shapes.circle import area as circle_area

print(rect_area(4, 5))
print(circle_area(3))
```

```text
(base) C:\Users\guest\project> python main2.py
20
28.259999999999998
```

- 두 모듈 모두 `area`라는 같은 이름의 함수를 가지고 있으므로, `as`로 별칭을 붙이지 않으면 나중에 가져온 `area`가 먼저 가져온 `area`를 덮어쓰게 된다. 이런 상황에서 `as`가 특히 유용하다.

**정리**: 패키지는 관련된 모듈들을 폴더로 묶은 단위이며 `import 패키지.모듈`처럼 점 표기법으로 접근하고, 패키지 폴더에는 관례적으로 `__init__.py` 파일을 두며, 서로 다른 모듈에 같은 이름의 요소가 있을 때는 `as`로 별칭을 붙여 이름 충돌을 피해야 한다.

## 와일드카드 임포트

`from 모듈이름 import *`처럼 별표(`*`)를 사용하면 모듈 안의 모든 이름을 한 번에 가져올 수 있다. 이를 **와일드카드 임포트(wildcard import)**라고 한다.

```python
from math import *

print(sqrt(16))
print(pi)
print(floor(3.7))
```

```text
(base) C:\Users\guest\project> python wildcard.py
4.0
3.141592653589793
3
```

**와일드카드 임포트를 권장하지 않는 이유**

```python
from math import *
from os import *

# math와 os 양쪽 모두에 비슷한 이름의 함수가 있다면
# 어느 모듈에서 온 함수인지 코드만 보고 알기 어렵다
```

- 어떤 이름이 정확히 어느 모듈에서 왔는지 코드만 보고 파악하기 어려워지고, 미처 알지 못한 이름이 함께 딸려와서 기존 변수·함수를 예기치 않게 덮어쓸 위험도 있다.
- 이런 이유로 실무에서는 `from 모듈 import *`보다는 `import 모듈` 또는 필요한 이름만 명시적으로 나열하는 `from 모듈 import 이름1, 이름2`를 사용하는 것이 권장된다. 와일드카드 임포트는 대화형 인터프리터에서 빠르게 여러 기능을 시험해볼 때 정도로만 제한적으로 사용하는 것이 바람직하다.

**정리**: `from 모듈 import *`는 모듈 안의 모든 이름을 한 번에 가져오는 와일드카드 임포트이며 편리해 보이지만, 이름의 출처를 알기 어렵게 만들고 기존 이름을 예기치 않게 덮어쓸 위험이 있어 실무에서는 지양하고 필요한 이름을 명시적으로 나열해서 가져오는 것이 권장된다.

## 표준 라이브러리 실전: math · os · shutil

파이썬은 설치 직후부터 바로 사용할 수 있는 다양한 **표준 라이브러리(standard library)**를 제공한다. 별도 설치(`pip install`) 없이 `import`만으로 바로 쓸 수 있다.

**math: 수학 계산**

```python
import math

print(math.sqrt(2))
print(math.pow(2, 10))
print(math.floor(3.7))
print(math.ceil(3.2))
print(math.factorial(5))
```

```text
(base) C:\Users\guest\project> python math_demo.py
1.4142135623730951
1024.0
3
4
120
```

- `sqrt()`는 제곱근, `pow()`는 거듭제곱(연산자 `**`와 동일한 결과), `floor()`는 내림, `ceil()`은 올림, `factorial()`은 계승을 계산한다.

**os: 운영체제·파일 시스템 정보**

```python
import os

print(os.getcwd())
os.mkdir("new_folder")
print(os.listdir("."))
os.rename("new_folder", "renamed_folder")
os.rmdir("renamed_folder")
```

```text
(base) C:\Users\guest\project> python os_demo.py
C:\Users\guest\project
['main.py', 'new_folder', 'os_demo.py']
```

- `os.getcwd()`는 현재 작업 디렉터리 경로를 반환하고, `os.mkdir()`/`os.rmdir()`은 폴더를 생성·삭제하며, `os.listdir()`은 지정한 경로의 파일·폴더 목록을 리스트로 돌려준다.
- `os.rename()`은 파일이나 폴더의 이름을 바꾼다.

**shutil: 파일·폴더 복사와 이동**

```python
import shutil

shutil.copy("config.txt", "config_backup.txt")
shutil.move("config_backup.txt", "backup/config_backup.txt")
shutil.rmtree("old_folder")
```

```text
(base) C:\Users\guest\project> python shutil_demo.py
```

- `shutil.copy()`는 파일 하나를 복사하고, `shutil.move()`는 파일·폴더를 다른 경로로 이동(또는 이름 변경)하며, `shutil.rmtree()`는 폴더와 그 안의 내용을 통째로 삭제한다(`os.rmdir()`은 빈 폴더만 삭제 가능하지만 `shutil.rmtree()`는 내용이 있어도 삭제한다는 차이가 있다).

**정리**: `math`는 제곱근·거듭제곱·올림·내림·계승 같은 수학 계산을, `os`는 현재 경로 확인·폴더 생성삭제·목록 조회 같은 운영체제 수준의 파일 시스템 조작을, `shutil`은 파일·폴더 단위의 복사·이동·통째 삭제를 제공하는 표준 라이브러리이며 셋 다 별도 설치 없이 `import`만으로 바로 사용할 수 있다.

## pip으로 외부 라이브러리 설치하기

표준 라이브러리에 없는 기능이 필요하면, PY-00 문서에서 다룬 `pip`으로 외부 라이브러리를 설치해서 사용한다. 대표적인 예로 그래프를 그리는 **matplotlib** 라이브러리가 있다.

```text
(base) C:\Users\guest\project> pip install matplotlib
Collecting matplotlib
  Downloading matplotlib-3.9.2-cp312-cp312-win_amd64.whl (7.8 MB)
Installing collected packages: matplotlib
Successfully installed matplotlib-3.9.2
```

**막대그래프 그리기**

```python
import matplotlib.pyplot as plt

names = ["국어", "영어", "수학", "과학"]
scores = [85, 92, 78, 88]

plt.bar(names, scores)
plt.title("과목별 점수")
plt.xlabel("과목")
plt.ylabel("점수")
plt.savefig("scores_bar.png")
print("그래프를 scores_bar.png로 저장했습니다.")
```

```text
(base) C:\Users\guest\project> python bar_chart.py
그래프를 scores_bar.png로 저장했습니다.
```

- `import matplotlib.pyplot as plt`는 관용적으로 굳어진 별칭이다. `plt.bar()`로 막대그래프를 그리고, `plt.title()`/`plt.xlabel()`/`plt.ylabel()`로 제목과 축 이름을 지정한다.
- `plt.show()`는 화면에 그래프 창을 띄우고, `plt.savefig("파일명")`은 화면에 띄우는 대신(또는 함께) 이미지 파일로 저장한다. 원격 서버나 화면 출력이 불가능한 환경에서는 `savefig()`가 특히 유용하다.

**정리**: 표준 라이브러리로 해결되지 않는 기능은 `pip install 라이브러리이름`으로 외부 라이브러리를 설치해 사용하며, `matplotlib` 같은 시각화 라이브러리는 설치 후 `import matplotlib.pyplot as plt`로 가져와 `plt.bar()` 같은 함수로 그래프를 그리고 `plt.savefig()`로 이미지 파일로 저장할 수 있다.

## 필요할 때 공식 문서를 찾아보는 학습 방식

라이브러리마다 제공하는 함수와 옵션의 수는 매우 많아서, 모든 함수의 사용법을 미리 전부 외워둘 필요는 없다. 실무에서 파이썬을 다루는 일반적인 학습 방식은 다음과 같다.

- 우선 지금 풀어야 하는 문제에 어떤 기능이 필요한지 파악한다(예: "막대그래프가 필요하다", "폴더 안 파일 목록이 필요하다").
- 표준 라이브러리라면 `import 모듈이름` 후 `help(모듈이름)`으로, 외부 라이브러리라면 해당 라이브러리의 공식 문서를 검색해 필요한 함수의 사용법과 매개변변수를 그때그때 찾아본다.

```python
>>> import math
>>> help(math.sqrt)
Help on built-in function sqrt in module math:

sqrt(x, /)
    Return the square root of x.
```

- `help()` 함수는 해당 함수·모듈의 설명(독스트링)을 즉시 보여주므로, 브라우저를 열지 않고도 빠르게 사용법을 확인할 수 있다.
- 라이브러리 규모가 크고 예제가 많이 필요할 때는 공식 문서 사이트의 API 레퍼런스와 예제 코드를 함께 참고하는 것이 효율적이다.

**정리**: 라이브러리의 모든 기능을 암기하려 하기보다, 지금 필요한 기능이 무엇인지 먼저 정의한 뒤 `help()`나 공식 문서를 그때그때 찾아보며 해결하는 것이 실무에서 통용되는 효율적인 학습 방식이며, 이런 습관은 새로운 라이브러리를 접했을 때도 빠르게 적응할 수 있게 해준다.

## 실습 예제 (EX1~EX4)

**EX1) 나만의 계산 모듈 만들고 불러오기**
- `add`, `subtract`, `multiply` 함수를 담은 `mymath.py` 모듈을 만들고, 다른 파일에서 `import`해서 사용하는 과정을 확인한다.

```python
# mymath.py
def add(a, b):
    return a + b

def subtract(a, b):
    return a - b

def multiply(a, b):
    return a * b
```

```python
# ex01_use_mymath.py
import mymath

print(mymath.add(3, 4))
print(mymath.subtract(10, 6))
print(mymath.multiply(5, 5))
```

```text
(base) C:\Users\guest\project> python ex01_use_mymath.py
7
4
25
```

**EX2) math 모듈로 원기둥 부피 계산기**
- `math.pi`를 사용해 원기둥의 부피(밑넓이 × 높이)를 계산한다.

```python
# ex02_cylinder_volume.py
import math

def cylinder_volume(radius, height):
    return math.pi * radius ** 2 * height

print(f"{cylinder_volume(3, 10):.2f}")
```

```text
(base) C:\Users\guest\project> python ex02_cylinder_volume.py
282.74
```

**EX3) os 모듈로 현재 폴더의 .py 파일만 골라내기**
- `os.listdir()`로 현재 폴더의 파일 목록을 가져와, 확장자가 `.py`인 파일만 걸러내 출력한다.

```python
# ex03_list_py_files.py
import os

files = os.listdir(".")
py_files = [f for f in files if f.endswith(".py")]

print(py_files)
```

```text
(base) C:\Users\guest\project> python ex03_list_py_files.py
['ex01_use_mymath.py', 'ex02_cylinder_volume.py', 'ex03_list_py_files.py', 'mymath.py']
```

- `[f for f in files if f.endswith(".py")]`는 리스트 컴프리헨션이라는 문법으로, `for`와 `if`를 한 줄로 축약해 조건에 맞는 값만 골라 새 리스트를 만드는 관용적인 표현이다.

**EX4) matplotlib로 월별 판매량 선 그래프 그리기**
- `pip install matplotlib` 설치 후, 월별 판매량 리스트를 선 그래프로 그려 이미지 파일로 저장하도록 작성한다.

```python
# ex04_line_chart.py
import matplotlib.pyplot as plt

months = ["1월", "2월", "3월", "4월", "5월"]
sales = [120, 135, 90, 160, 180]

plt.plot(months, sales, marker="o")
plt.title("월별 판매량")
plt.xlabel("월")
plt.ylabel("판매량")
plt.savefig("sales_line.png")
print("그래프를 sales_line.png로 저장했습니다.")
```

```text
(base) C:\Users\guest\project> pip install matplotlib
(base) C:\Users\guest\project> python ex04_line_chart.py
그래프를 sales_line.png로 저장했습니다.
```

**정리**: EX1~EX4는 직접 만든 모듈을 다른 파일에서 불러와 사용하고, 표준 라이브러리 `math`로 실전 계산을 하고, `os`로 파일 목록을 다루고, 외부 라이브러리 `matplotlib`으로 그래프를 그려 저장하기까지 이 문서에서 다룬 모듈·패키지·외부 라이브러리 활용법을 코드로 직접 확인해보는 예제이며, 다음 문서에서는 코드 실행 중 발생하는 오류를 읽고 처리하는 방법을 다룬다.

[Python 06 — 클래스](06-classes.md) · [Python 08 — 에러 처리](08-error-handling.md)

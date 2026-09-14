# Python 15 — 가상 환경 심화와 부동소수점

## venv/pip 워크플로 정리

PY-00에서는 Anaconda 설치와 함께 가상 환경을 처음 만들어 활성화하는 실습을 다뤘다. 이번에는 표준 라이브러리에 내장된 `venv` 모듈로 가상 환경을 만드는 방법을 정리하고, 여러 사람이 함께 작업할 때 동일한 개발 환경을 재현하는 `requirements.txt` 워크플로를 조금 더 실무에 가깝게 살펴본다.

**가상 환경을 만드는 이유**

프로젝트마다 필요한 라이브러리와 그 버전이 다를 수 있다. 하나의 파이썬 환경에 모든 프로젝트의 라이브러리를 한꺼번에 설치하면, 프로젝트 A가 필요로 하는 `requests==2.28`과 프로젝트 B가 필요로 하는 `requests==2.31`이 충돌할 수 있다. **가상 환경(virtual environment)**은 프로젝트마다 독립된 파이썬 실행 환경과 라이브러리 설치 공간을 만들어 이런 충돌을 막아준다.

**venv로 가상 환경 만들고 활성화하기**

```text
(base) C:\Users\guest\project> python -m venv .venv
(base) C:\Users\guest\project> .venv\Scripts\activate
(.venv) C:\Users\guest\project> python -m pip install --upgrade pip
(.venv) C:\Users\guest\project> python -m pip install requests pandas
```

- `python -m venv .venv`는 현재 폴더 안에 `.venv`라는 이름의 독립된 가상 환경을 생성한다. 폴더 이름은 자유롭게 정할 수 있지만 `.venv` 또는 `venv`가 관례적으로 많이 쓰인다.
- `.venv\Scripts\activate`(윈도우 기준, macOS/Linux는 `source .venv/bin/activate`)로 가상 환경을 활성화하면, 프롬프트 맨 앞에 `(.venv)`가 표시되어 현재 이 가상 환경이 활성화된 상태임을 알 수 있다. 활성화된 상태에서 `pip install`로 설치하는 라이브러리는 시스템 전체가 아니라 이 가상 환경 안에만 설치된다.
- 가상 환경 작업이 끝나면 `deactivate` 명령으로 비활성화할 수 있다.

**requirements.txt로 협업 환경 재현하기**

혼자 작업할 때는 가상 환경을 한 번 만들면 그만이지만, 여러 명이 함께 작업하는 프로젝트에서는 팀원 모두가 정확히 같은 라이브러리와 버전을 설치해야 코드가 똑같이 동작한다. `pip`는 이를 위해 설치된 라이브러리 목록을 텍스트 파일로 저장하고 복원하는 기능을 제공한다.

```text
(.venv) C:\Users\guest\project> python -m pip freeze > requirements.txt
(.venv) C:\Users\guest\project> type requirements.txt
certifi==2024.2.2
charset-normalizer==3.3.2
idna==3.6
numpy==1.26.4
pandas==2.2.1
requests==2.31.0
urllib3==2.2.1
```

- `pip freeze`는 현재 가상 환경에 설치된 모든 라이브러리와 정확한 버전을 `이름==버전` 형태로 나열한다. 이 출력을 `requirements.txt` 파일로 저장해두면, 지금 이 환경을 그대로 재현할 수 있는 "설계도"가 만들어진다.
- 이 파일을 프로젝트 저장소(git 등)에 함께 커밋해두면, 새로 프로젝트에 합류한 팀원은 다음 과정만으로 동일한 환경을 그대로 재현할 수 있다.

```text
(base) C:\Users\guest\project> python -m venv .venv
(base) C:\Users\guest\project> .venv\Scripts\activate
(.venv) C:\Users\guest\project> python -m pip install -r requirements.txt
```

- `pip install -r requirements.txt`는 파일에 나열된 라이브러리들을 지정된 버전 그대로 한 번에 설치해준다. `-r`은 "requirements 파일로부터(from requirements)" 설치한다는 의미다.
- 실무에서는 라이브러리를 새로 추가하거나 업데이트할 때마다 `pip freeze > requirements.txt`로 파일을 다시 갱신하고, 그 변경 사항을 팀원들과 공유하는 흐름을 반복하게 된다. 이렇게 하면 "내 컴퓨터에서는 되는데 다른 사람 컴퓨터에서는 안 되는" 환경 차이 문제를 크게 줄일 수 있다.

**가상 환경 폴더는 저장소에 커밋하지 않는다**

`.venv` 폴더 자체는 팀원마다 운영체제와 파이썬 버전이 다를 수 있는 실행 파일들을 담고 있으므로, git 같은 버전 관리 시스템에 함께 올리지 않는 것이 관례다. 대신 `.gitignore` 파일에 가상 환경 폴더 이름을 등록해 커밋 대상에서 제외하고, 그 자리를 `requirements.txt`가 대신하게 한다.

```text
# .gitignore
.venv/
__pycache__/
*.pyc
```

- 저장소에는 "무엇을 설치해야 하는지"를 적은 `requirements.txt`만 남기고, 실제로 설치된 결과물인 `.venv` 폴더 자체는 각자의 컴퓨터에서 새로 만들어 쓰는 것이 표준적인 방식이다. 이렇게 하면 저장소 용량도 작게 유지되고, 운영체제가 다른 팀원 사이에서도 문제없이 협업할 수 있다.

**여러 프로젝트, 여러 가상 환경**

```text
(base) C:\Users\guest\project-a> python -m venv .venv
(base) C:\Users\guest\project-a> .venv\Scripts\activate
(.venv) C:\Users\guest\project-a> python -m pip install django==4.2

(base) C:\Users\guest\project-b> python -m venv .venv
(base) C:\Users\guest\project-b> .venv\Scripts\activate
(.venv) C:\Users\guest\project-b> python -m pip install django==5.0
```

- 프로젝트 폴더마다 각각 독립된 `.venv`를 만들어두면, `project-a`는 Django 4.2를, `project-b`는 Django 5.0을 동시에 사용하더라도 서로 전혀 영향을 주지 않는다. 활성화된 가상 환경을 바꾸려면 현재 환경을 `deactivate`로 비활성화한 뒤 다른 프로젝트 폴더에서 새로 `activate`하면 된다.
- 활성화를 깜빡하고 `pip install`을 실행하면 원하는 가상 환경이 아니라 시스템 전역 환경(또는 다른 프로젝트의 활성화된 환경)에 라이브러리가 설치될 수 있으므로, 작업을 시작하기 전에 프롬프트 앞에 의도한 가상 환경 이름이 붙어 있는지(`(.venv)`) 확인하는 습관이 중요하다.

**정리**: `venv`로 프로젝트마다 독립된 가상 환경을 만들어 라이브러리 버전 충돌을 방지하고, `pip freeze > requirements.txt`로 현재 설치된 라이브러리 목록과 버전을 파일로 남긴 뒤, 다른 팀원이나 다른 컴퓨터에서 `pip install -r requirements.txt`로 그 환경을 그대로 재현하는 흐름은 협업 프로젝트에서 표준적으로 쓰이는 워크플로이며, 이 파일을 프로젝트 저장소에 함께 관리하는 것이 실무에서 매우 중요한 습관이다.

**자주 쓰는 pip 명령어 정리**

| 명령어 | 의미 |
|--------|------|
| `pip install 이름` | 최신 버전 설치 |
| `pip install 이름==1.2.3` | 특정 버전으로 설치 |
| `pip install --upgrade 이름` | 이미 설치된 라이브러리를 최신 버전으로 업그레이드 |
| `pip uninstall 이름` | 라이브러리 제거 |
| `pip list` | 현재 환경에 설치된 라이브러리 목록 확인 |
| `pip show 이름` | 특정 라이브러리의 버전·설치 위치 등 상세 정보 확인 |
| `pip freeze` | `이름==버전` 형식으로 설치 목록 출력(`requirements.txt` 생성에 사용) |

- `requirements.txt`를 작성할 때 `pandas`처럼 버전을 명시하지 않고 이름만 적으면 설치 시점의 최신 버전이 설치되므로, 시간이 지나 팀원마다 서로 다른 버전을 설치하게 될 수 있다. 협업 프로젝트에서는 `pip freeze`로 얻은 것처럼 `pandas==2.2.1`같이 버전을 명시적으로 고정하는 것이 안전하다.
- 이 문서의 예제들은 모두 `pip install ...` 대신 `python -m pip install ...` 형태를 사용했는데, 이는 지금 명령어를 입력하는 셸에서 `python`으로 실행되는 것과 정확히 같은 파이썬 인터프리터에 연결된 `pip`를 확실히 사용하겠다는 의미다. 시스템에 여러 파이썬 버전이 함께 설치되어 있으면 `pip`만 단독으로 실행했을 때 의도한 것과 다른 파이썬 환경의 `pip`가 실행되는 경우가 있으므로, `python -m pip` 형태로 쓰는 습관을 들이면 이런 혼동을 줄일 수 있다.

**Anaconda의 conda 환경과 venv의 관계**

PY-00에서는 Anaconda를 설치해 `(base)`라는 이름의 환경이 프롬프트 앞에 표시되는 것을 확인했다. Anaconda는 `conda create -n 환경이름`으로 자체적인 가상 환경 관리 도구(conda)를 함께 제공하는데, 이는 이 문서에서 다룬 표준 라이브러리 `venv`와 목적은 같지만(프로젝트별로 독립된 환경 구성) 별개의 도구다. conda는 파이썬 자체의 버전까지 함께 관리하고 파이썬 이외의 프로그램(예: C/C++ 라이브러리)까지 설치할 수 있는 반면, `venv`는 파이썬 표준 라이브러리에 내장되어 있어 별도 설치 없이 바로 쓸 수 있고 순수하게 파이썬 패키지 관리에 집중한다는 차이가 있다. 두 도구 중 어느 쪽을 쓰든 "프로젝트마다 독립된 환경을 만들고 `requirements.txt`(또는 conda의 `environment.yml`)로 그 환경을 재현 가능하게 기록해둔다"는 핵심 원칙은 동일하다.

## 부동소수점 표현 오차

**`0.1 + 0.2`가 `0.3`이 아닌 이유**

```python
print(0.1 + 0.2)
print(0.1 + 0.2 == 0.3)
```

```text
(base) C:\Users\guest\project> python float_precision.py
0.30000000000000004
False
```

- 컴퓨터는 실수를 내부적으로 **2진 부동소수점(binary floating point)** 형식으로 저장한다. 그런데 `0.1`이나 `0.2`처럼 10진법으로는 딱 떨어지는 소수도, 2진법으로 변환하면 `1/3`을 10진법 소수로 정확히 표현할 수 없는 것과 같은 이유로 무한히 반복되는 값이 되어 정확히 표현할 수 없다. 결국 파이썬은 그 값에 가장 가까운 2진 부동소수점 근삿값을 저장하게 되고, 이 근삿값들을 더한 결과가 우리가 기대하는 `0.3`의 근삿값과 미세하게 어긋나면서 `0.30000000000000004`처럼 눈에 보이는 오차가 발생한다.
- 이것은 파이썬만의 문제가 아니라, IEEE 754라는 표준을 따르는 대부분의 프로그래밍 언어(C, Java, JavaScript 등)에서 공통적으로 나타나는 부동소수점 자체의 한계다.
- 그 결과 `0.1 + 0.2 == 0.3`을 그대로 비교하면 `False`가 나오므로, 실수를 다룰 때는 값이 정확히 같은지 `==`로 직접 비교하기보다 오차 범위를 감안해 비교해야 한다.

**0.1이 실제로 저장되는 값 들여다보기**

10진법에서 `1/3`을 소수로 정확히 쓸 수 없어 `0.333...`처럼 무한히 반복되는 것과 마찬가지로, 2진법에서는 `1/10`(즉 `0.1`)이 정확히 떨어지지 않고 무한히 반복되는 2진 소수가 된다. 파이썬의 `float`는 이 무한히 이어지는 값을 유한한 비트 수(64비트) 안에서 가장 가까운 값으로 잘라 저장할 수밖에 없다.

```python
from fractions import Fraction

print(0.1.hex())
print(Fraction(0.1))
print(Fraction(1, 10))
```

```text
(base) C:\Users\guest\project> python float_internal.py
0x1.999999999999ap-4
3602879701896397/36028797018963968
1/10
```

- `float.hex()`는 실수가 내부적으로 어떤 2진 값으로 저장되어 있는지를 16진수로 보여준다. `0.1`을 저장했다고 생각하지만 실제로는 `0x1.999999999999a`라는 값이 저장되어 있음을 확인할 수 있다.
- `Fraction(0.1)`은 파이썬이 실제로 저장하고 있는 `0.1`의 근삿값을 정확한 분수로 변환해서 보여주는데, 우리가 기대하는 `1/10`이 아니라 `3602879701896397/36028797018963968`라는, `1/10`과 매우 가깝지만 정확히 같지는 않은 분수임을 알 수 있다. 이 미세한 차이가 여러 번의 덧셈을 거치며 눈에 보이는 오차로 드러나는 것이다.

**round()의 한계**

```python
print(round(0.1 + 0.2, 1))
print(round(2.675, 2))
```

```text
(base) C:\Users\guest\project> python round_limit.py
0.3
2.67
```

- `round(0.1 + 0.2, 1)`처럼 자릿수를 줄여서 반올림하면 겉보기 오차는 사라져 `0.3`으로 보이지만, 이는 어디까지나 출력할 때 반올림된 결과일 뿐 내부적으로는 여전히 2진 부동소수점 근삿값이 저장되어 있다.
- `round(2.675, 2)`의 결과가 수학적으로 기대하는 `2.68`이 아니라 `2.67`이 나오는 것도 같은 이유다. `2.675`라는 값 자체가 2진 부동소수점으로 저장되는 순간 `2.675`보다 아주 조금 작은 근삿값으로 저장되기 때문에, 그 근삿값을 반올림하면 `2.68`이 아니라 `2.67`이 나온다.
- 또한 파이썬의 `round()`는 `.5`를 무조건 올림하지 않고, 가장 가까운 짝수로 반올림하는 **"은행가의 반올림(banker's rounding)"** 방식을 사용한다는 점도 함께 알아두어야 한다.

```python
print(round(0.5))
print(round(1.5))
print(round(2.5))
```

```text
(base) C:\Users\guest\project> python round_banker.py
0
2
2
```

- `round(0.5)`는 `1`이 아니라 `0`, `round(1.5)`는 `2`, `round(2.5)`도 `2`가 된다. `0.5`에서 더 가까운 짝수인 `0`으로, `1.5`와 `2.5`는 모두 더 가까운 짝수인 `2`로 반올림되는 것이다. 이는 여러 번 반올림을 반복했을 때 결과가 한쪽으로 쏠리는 편향을 줄이기 위한 설계이며, 학교에서 배운 "5는 무조건 올림" 방식과 다르다는 점에 주의해야 한다.
- 수많은 값을 반복해서 반올림해야 하는 통계나 회계 계산에서, 항상 "5는 올림"만 적용하면 값이 계속 커지는 쪽으로 쏠리는 편향이 누적될 수 있다. 짝수 쪽으로 반올림하는 방식은 이런 누적 편향을 평균적으로 상쇄해주므로, 다수의 반올림 결과를 합산해야 하는 상황에서는 오히려 더 안정적인 결과를 준다. 다만 한두 번의 반올림 결과가 익숙한 사칙연산 교육 과정과 다르게 보일 수 있다는 점은 실무에서 자주 혼동을 일으키는 부분이므로 미리 알아두는 것이 좋다.

**decimal.Decimal — 정확한 소수 계산이 필요할 때**

금액 계산처럼 소수점 오차가 절대 허용되지 않는 상황에서는 표준 라이브러리 `decimal` 모듈의 `Decimal`을 사용한다.

```python
from decimal import Decimal

a = Decimal("0.1")
b = Decimal("0.2")

print(a + b)
print(a + b == Decimal("0.3"))
```

```text
(base) C:\Users\guest\project> python decimal_demo.py
0.3
True
```

- `Decimal`은 값을 2진 부동소수점이 아니라 10진수 그대로 정확하게 저장하는 자료형이므로, `Decimal("0.1") + Decimal("0.2")`는 오차 없이 정확히 `Decimal("0.3")`이 된다.
- `Decimal(0.1)`처럼 이미 오차가 생긴 `float` 값을 그대로 넘기면 그 오차까지 함께 가져오므로, 반드시 `Decimal("0.1")`처럼 **문자열**로 값을 넘겨야 정확한 값을 얻을 수 있다는 점에 주의해야 한다.
- 다만 `Decimal`은 일반 `float` 연산보다 느리기 때문에, 과학 계산이나 대량의 연산에는 `float`를, 금액처럼 정확한 소수 표현이 반드시 필요한 곳에는 `Decimal`을 쓰는 것이 합리적인 선택이다.

```python
from decimal import Decimal, getcontext

print(getcontext().prec)  # 기본 유효 자릿수

getcontext().prec = 4
print(Decimal(1) / Decimal(3))
```

```text
(base) C:\Users\guest\project> python decimal_precision.py
28
0.3333
```

- `Decimal`도 `1/3`처럼 무한히 반복되는 값을 완전히 무한한 정밀도로 저장할 수는 없지만, `getcontext().prec`으로 유효 자릿수를 원하는 만큼 조절할 수 있다는 점이 `float`와 다르다. 기본값은 28자리로, 대부분의 금액 계산에 충분한 정밀도를 제공한다.

**math.isclose()로 오차를 감안해 비교하기**

부동소수점 오차를 매번 직접 계산해 비교식을 작성하는 대신, 표준 라이브러리 `math` 모듈의 `isclose()` 함수를 사용하면 오차를 감안한 비교를 한 줄로 처리할 수 있다.

```python
import math

print(math.isclose(0.1 + 0.2, 0.3))
print(0.1 + 0.2 == 0.3)
```

```text
(base) C:\Users\guest\project> python isclose_demo.py
True
False
```

- `math.isclose(a, b)`는 두 값이 정확히 같은지가 아니라 상대적으로 충분히 가까운지를 판단해주며, 기본적으로 상대 오차 `1e-09`(약 10억분의 1) 이내면 같다고 간주한다. `rel_tol`이나 `abs_tol` 인자로 허용 오차 범위를 직접 지정할 수도 있다.
- 앞서 EX2에서 직접 작성한 `almost_equal()` 함수와 목적이 같지만, `math.isclose()`는 표준 라이브러리에 이미 구현되어 있으므로 직접 함수를 작성하지 않고도 바로 사용할 수 있다는 장점이 있다.
- 다만 `rel_tol`(상대 오차)만으로는 값이 `0`에 가까운 경우를 제대로 비교하기 어렵다. 예를 들어 `math.isclose(1e-10, 0)`은 상대 오차 기준으로 `False`가 나오는데, 이런 경우 `abs_tol`(절대 오차) 인자를 함께 지정해 `math.isclose(1e-10, 0, abs_tol=1e-9)`처럼 쓰면 `True`가 된다. 비교 대상 중 하나가 `0`이거나 `0`에 매우 가까울 수 있는 상황이라면 `abs_tol`도 함께 고려해야 한다.

**정리**: `0.1 + 0.2`가 `0.3`과 정확히 같지 않은 것은 파이썬의 버그가 아니라, 실수를 2진 부동소수점으로 저장하는 과정에서 발생하는 구조적인 근사 오차이며, `round()`도 이 근삿값을 기준으로 동작하고 게다가 `.5`를 가장 가까운 짝수로 반올림하는 은행가의 반올림 방식을 쓰기 때문에 기대와 다른 결과가 나올 수 있으므로, 금액 계산처럼 정확한 10진수 소수 연산이 필요한 경우에는 `float` 대신 `decimal.Decimal`(문자열로 값을 생성)을 사용해야 한다.

## 대화형 인터프리터 팁

PY-00과 PY-01에서 REPL(대화형 인터프리터)을 실행하는 방법과 기본적인 사용법을 다뤘다. 여기서는 REPL을 더 효율적으로 쓸 수 있는 몇 가지 팁을 정리한다.

**탭 완성**

```text
>>> import data
```

- REPL에서 변수명, 함수명, 모듈명을 입력하다가 `Tab` 키를 누르면 이어질 수 있는 이름들을 자동으로 완성해주거나 후보 목록을 보여준다. 예를 들어 `import dat`까지 입력하고 `Tab`을 누르면 `datetime` 같은 후보가 자동 완성되며, 긴 변수명이나 모듈명을 매번 전부 입력하지 않아도 된다.

**화살표 키로 히스토리 탐색**

```text
>>> x = 10
>>> y = 20
>>> print(x + y)
30
```

- 위 코드를 입력한 뒤 위쪽 화살표 키(`↑`)를 누르면 방금 입력했던 `print(x + y)`가 그대로 다시 나타나고, 계속 누르면 그 이전에 입력했던 `y = 20`, `x = 10` 순서로 과거 입력 이력을 거슬러 올라갈 수 있다. 아래쪽 화살표 키(`↓`)는 반대로 최근 입력 쪽으로 이동한다. 비슷한 코드를 여러 번 수정하며 반복 실행할 때 매번 새로 타이핑하지 않아도 되므로 REPL 작업 속도를 크게 높여준다.

**밑줄(`_`)에 저장되는 마지막 결괏값**

```text
>>> 3 + 4
7
>>> _
7
>>> _ * 2
14
>>> result = _
>>> result
14
```

- REPL에서(스크립트 파일 실행이 아니라 대화형 모드에서만) 방금 평가된 표현식의 결괏값은 자동으로 `_`(밑줄) 변수에 저장된다. 계산 결과를 별도 변수에 담아두지 않았더라도, 바로 다음 줄에서 `_`를 이용해 그 값을 이어서 사용할 수 있다.
- 단, `_`는 대입문(`x = 3 + 4`처럼 결과를 변수에 직접 저장하는 문장)에는 적용되지 않으며, 값을 출력하는 표현식을 평가했을 때만 갱신된다는 점에 유의해야 한다.

**help()로 즉석에서 문서 확인하기**

```text
>>> help(round)
Help on built-in function round in module builtins:

round(number, ndigits=None)
    Round a number to a given precision in decimal digits.
    ...
```

- REPL에서 `help(대상)`을 실행하면 그 함수나 객체에 대한 설명(독스트링)을 바로 확인할 수 있다. 인터넷 문서를 찾아보러 가지 않아도, 지금 사용 중인 파이썬 버전에 실제로 설치된 함수의 설명을 그 자리에서 확인할 수 있다는 점이 장점이다. 설명을 다 읽었으면 `q`를 눌러 빠져나올 수 있다.
- 인자를 주지 않고 `help()`만 실행하면 대화형 도움말 모드로 들어가며, 여기서 함수 이름이나 모듈 이름을 입력해 계속 탐색할 수 있다. `quit`을 입력하면 도움말 모드에서 빠져나온다.

**정리**: 파이썬 REPL은 `Tab` 키로 변수·함수·모듈 이름을 자동 완성하고, 위/아래 화살표 키로 과거에 입력했던 코드 이력을 다시 불러와 수정·재실행할 수 있으며, 대화형 모드에서 마지막으로 평가된 값이 자동으로 `_` 변수에 저장되어 바로 다음 줄에서 재사용할 수 있다는 점까지 알아두면, 짧은 코드를 여러 번 시험해보며 탐색적으로 개발하는 REPL 작업의 효율을 크게 높일 수 있다.

## 마무리 — PY-00부터 PY-15까지

지금까지 PY-00에서 파이썬 개발 환경을 설치하는 것으로 시작해, PY-01~PY-05에서 출력·변수·함수·자료형·조건문·반복문 같은 기본 문법을 다졌고, PY-06~PY-08에서 클래스와 객체 지향, 모듈과 라이브러리, 에러 처리라는 조금 더 구조적인 개념을 익혔으며, PY-09~PY-11에서 패턴 매칭과 가변 인자, 컴프리헨션과 다양한 자료구조, 파일과 JSON을 다루는 입출력까지 문법의 깊이를 더해왔다. 그리고 이번 PY-12~PY-15에서는 예외를 직접 설계하고 연쇄시키는 방법, 스코프 규칙과 이터레이터·제너레이터로 객체와 반복의 내부 동작 원리를 이해하는 방법, 자주 쓰이는 표준 라이브러리 모듈들을 실전에 활용하는 방법, 그리고 가상 환경과 부동소수점처럼 실무에서 반드시 부딪히게 되는 환경·정밀도 이슈까지 다루면서, 이 위키의 파이썬 시리즈가 다루는 기초 문법과 표준 라이브러리 범위를 마무리했다.

이제부터는 목적에 따라 각자 필요한 분야의 라이브러리와 프레임워크를 찾아 학습을 이어가는 것이 자연스러운 다음 단계다. 웹 개발에 관심이 있다면 웹 서버와 API를 만드는 프레임워크 쪽을, 데이터 분석이나 통계에 관심이 있다면 표 형태 데이터를 다루고 시각화하는 라이브러리 쪽을, 자동화나 반복 업무 처리에 관심이 있다면 파일·스프레드시트·웹 페이지를 다루는 자동화 도구 쪽을, 인공지능이나 머신러닝에 관심이 있다면 수치 연산과 모델 학습을 위한 라이브러리 쪽을 살펴보는 식으로, 이 문서에서 다진 변수·함수·클래스·예외·이터레이터·표준 라이브러리 활용 감각을 바탕 삼아 각자의 목적에 맞는 도구들을 하나씩 넓혀가면 된다.

어떤 분야로 나아가든 이 시리즈에서 다진 기본기는 그대로 밑거름이 된다. PY-03~PY-06에서 다진 함수와 클래스 설계 감각은 어떤 프레임워크를 배우든 그 코드를 읽고 확장하는 기반이 되고, PY-08과 PY-12에서 다진 예외 처리 감각은 외부 API 호출이나 파일 입출력처럼 실패할 수 있는 작업을 안전하게 다루는 데 그대로 쓰이며, PY-13의 이터레이터·제너레이터는 대용량 데이터를 다루는 라이브러리들의 내부 동작 원리를 이해하는 데 도움이 되고, PY-15에서 다룬 가상 환경과 `requirements.txt` 워크플로는 어떤 프로젝트를 시작하든 가장 먼저 적용하게 될 실무 습관이다. 결국 새로운 라이브러리를 배운다는 것은 새로운 문법을 배우는 것이 아니라, 이미 익힌 문법 위에 그 라이브러리가 제공하는 함수와 클래스를 얹어 사용하는 것에 가깝다.

## 실습 예제 (EX1~EX4)

**EX1) requirements.txt 작성 시나리오 정리하기**
- 특정 라이브러리 목록이 주어졌을 때, 이를 `requirements.txt` 형식의 문자열로 만들고 다시 파싱해 딕셔너리로 정리한다.

```python
# ex01_requirements_format.py
packages = {"requests": "2.31.0", "pandas": "2.2.1", "numpy": "1.26.4"}

lines = [f"{name}=={version}" for name, version in packages.items()]
requirements_text = "\n".join(lines)
print(requirements_text)

parsed = {}
for line in requirements_text.split("\n"):
    name, version = line.split("==")
    parsed[name] = version

print(parsed)
print(parsed == packages)
```

```text
(base) C:\Users\guest\project> python ex01_requirements_format.py
requests==2.31.0
pandas==2.2.1
numpy==1.26.4
{'requests': '2.31.0', 'pandas': '2.2.1', 'numpy': '1.26.4'}
True
```

**EX2) 부동소수점 오차 때문에 실패하는 비교를 올바르게 고치기**
- 실수 두 값이 "거의 같은지"를 판단하는 함수를 오차 허용 범위를 이용해 작성한다.

```python
# ex02_almost_equal.py
def almost_equal(a, b, tolerance=1e-9):
    return abs(a - b) < tolerance

x = 0.1 + 0.2
y = 0.3

print("직접 비교(==):", x == y)
print("오차 허용 비교:", almost_equal(x, y))
```

```text
(base) C:\Users\guest\project> python ex02_almost_equal.py
직접 비교(==): False
오차 허용 비교: True
```

**EX3) Decimal로 영수증 합계 정확히 계산하기**
- 여러 상품의 가격을 `float`와 `Decimal`로 각각 합산해 결과를 비교한다.

```python
# ex03_decimal_receipt.py
from decimal import Decimal

prices_float = [0.1, 0.2, 0.3, 1.1]
prices_decimal = [Decimal("0.1"), Decimal("0.2"), Decimal("0.3"), Decimal("1.1")]

print("float 합계:", sum(prices_float))
print("Decimal 합계:", sum(prices_decimal))
print("float == 1.7:", sum(prices_float) == 1.7)
print("Decimal == 1.7:", sum(prices_decimal) == Decimal("1.7"))
```

```text
(base) C:\Users\guest\project> python ex03_decimal_receipt.py
float 합계: 1.7000000000000002
Decimal 합계: 1.7
float == 1.7: False
Decimal == 1.7: True
```

**EX4) REPL에서 _ 변수를 활용해 단계별 계산 이어가기**
- REPL을 직접 열어 `_`를 활용해 연속된 계산을 이어가는 과정을 확인한다.

```text
(base) C:\Users\guest\project> python
>>> 0.1 + 0.2
0.30000000000000004
>>> round(_, 1)
0.3
>>> _ * 1000
300.0
>>> exit()
```

- 첫 번째 줄에서 `0.1 + 0.2`의 결과가 `_`에 저장되고, 두 번째 줄에서 `round(_, 1)`로 그 값을 반올림해 다시 `_`를 갱신하며, 세 번째 줄에서는 갱신된 `_`에 `1000`을 곱하는 식으로, 변수를 따로 만들지 않고도 직전 결과를 계속 이어서 활용할 수 있다.

**정리**: EX1~EX4는 라이브러리 버전 정보를 `requirements.txt` 형식으로 만들고 다시 파싱하는 과정, 부동소수점 오차를 직접 비교(`==`) 대신 오차 허용 범위로 우회하는 방법, `float`와 `Decimal`의 합산 결과 차이를 직접 비교해보는 방법, 그리고 REPL에서 `_` 변수로 단계별 계산을 이어가는 과정까지, 이 문서에서 다룬 협업 환경 재현과 부동소수점·REPL 활용 개념을 직접 코드와 REPL 조작으로 확인해보는 예제다. 이것으로 PY-00부터 PY-15까지 이어진 파이썬 시리즈를 마무리한다.

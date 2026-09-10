# Python 18 — 패키징과 배포

PY-15에서는 `venv`로 가상 환경을 만들고 `requirements.txt`로 여러 라이브러리의 의존성 목록을 관리하는 방법을 다뤘다. 이 문서에서는 한 걸음 더 나아가, 지금까지 작성해온 스크립트나 모듈 자체를 다른 사람이 `pip install`로 설치해 쓸 수 있는 하나의 **패키지**로 만드는 방법을 살펴본다.

## 왜 패키징이 필요한가

**스크립트에서 패키지로**

지금까지의 예제들은 대부분 하나의 `.py` 파일을 직접 실행하거나, 같은 폴더 안의 다른 모듈을 `import`해서 사용하는 형태였다. 이런 방식은 개인 프로젝트나 같은 폴더 안에서 작업할 때는 충분하지만, 몇 가지 한계가 있다.

- 다른 프로젝트에서 이 코드를 재사용하려면 파일을 직접 복사해 와야 한다. 원본이 수정되어도 복사본에는 자동으로 반영되지 않는다.
- 다른 사람에게 이 코드를 전달하려면 폴더 전체를 압축해 보내야 하고, 받은 사람은 어떤 라이브러리가 더 필요한지 직접 알아내야 한다.
- 여러 프로젝트에서 공통으로 쓰는 유틸리티 코드가 있다면, 프로젝트마다 파일을 중복해서 두는 대신 한 번만 설치해두고 어디서든 `import`할 수 있으면 훨씬 편리하다.

**패키징**은 이런 문제를 해결하기 위해, 내 코드와 그 코드가 어떤 이름의 패키지이고 어떤 버전이며 어떤 라이브러리에 의존하는지에 대한 정보를 함께 묶어, `pip install`로 설치할 수 있는 배포 단위로 만드는 작업을 말한다. 이렇게 만들어두면 같은 컴퓨터의 다른 프로젝트에서도, 또는 다른 사람의 컴퓨터에서도 `pip install 패키지이름` 한 줄로 내 코드를 설치해 바로 `import`해 쓸 수 있게 된다.

**requirements.txt와 패키징의 차이**

PY-15에서 다룬 `requirements.txt`와 이 문서에서 다루는 패키징은 목적이 서로 다르다.

- `requirements.txt`는 **내 프로젝트가 다른 라이브러리에 의존하는 목록**을 기록한 파일이다. "이 프로젝트를 실행하려면 `requests==2.31.0`, `pandas==2.2.1`이 필요하다"는 정보를 담을 뿐, 내 코드 자체를 설치 가능한 단위로 만들어주지는 않는다.
- 패키징은 반대로 **내 코드 자체를 설치 가능한 하나의 단위**로 만드는 작업이다. 패키징을 마치면 내가 만든 코드도 `requests`나 `pandas`처럼 `pip install`로 설치되는 대상이 된다.
- 두 개념은 함께 쓰이기도 한다. 내가 만든 패키지도 내부적으로 `requests`나 `pandas` 같은 다른 라이브러리에 의존할 수 있으며, 이런 의존 관계는 잠시 뒤에 살펴볼 `pyproject.toml`의 `dependencies` 항목에 적어둔다.

**정리**: 지금까지 작성한 스크립트는 같은 폴더 안에서만 재사용할 수 있었지만, 패키징을 거치면 코드와 이름·버전·의존성 정보를 하나의 배포 단위로 묶어 `pip install`로 어디서든 설치해 쓸 수 있게 되며, 이는 `requirements.txt`가 담당하는 "내 프로젝트가 무엇에 의존하는가"라는 정보와는 반대로 "내 코드 자체를 다른 사람이 의존할 수 있는 대상으로 만드는 것"이라는 점에서 서로 보완적인 개념이다.

## 표준 패키지 디렉터리 레이아웃

**src 레이아웃**

파이썬 패키지를 만들 때 널리 쓰이는 표준 디렉터리 구조는 `src/` 폴더 아래에 실제 패키지 코드를 두는 **src 레이아웃**이다.

```text
mypackage-project/
├── src/
│   └── mypackage/
│       ├── __init__.py
│       ├── calculator.py
│       └── formatter.py
├── tests/
│   └── test_calculator.py
├── pyproject.toml
├── README.md
└── .gitignore
```

- 최상위 폴더(`mypackage-project/`)는 프로젝트 전체를 담는 폴더로, 이름은 배포되는 패키지 이름과 반드시 같을 필요는 없다.
- `src/mypackage/`처럼 실제로 `import mypackage`했을 때 불러오는 코드는 `src/` 폴더 안에 패키지 이름과 같은 폴더를 만들어 그 안에 둔다.
- `tests/`에는 이 패키지를 검증하는 테스트 코드를 둔다. PY-16에서 다룬 `unittest`로 작성한 테스트 파일들이 여기에 들어간다.
- `pyproject.toml`은 잠시 뒤에 살펴볼, 이 패키지의 이름·버전·의존성 등 모든 설정 정보를 담는 파일이다.

**src 레이아웃을 쓰는 이유**

`src/` 폴더 없이 프로젝트 최상위에 바로 `mypackage/` 폴더를 두는 **flat 레이아웃**도 가능하지만, `src/` 레이아웃을 권장하는 이유가 있다.

- `src/` 없이 최상위에 패키지 폴더를 두면, 프로젝트 폴더 자체에서 파이썬을 실행할 때 실제로 설치된 패키지가 아니라 현재 폴더에 있는 소스 코드가 먼저 `import`될 수 있어, "제대로 설치된 상태에서도 잘 동작하는지"를 착각하기 쉽다.
- `src/` 레이아웃은 이런 혼동을 원천적으로 막아, 반드시 `pip install`로 설치한 뒤에만 `import mypackage`가 동작하도록 강제한다. 이렇게 하면 개발 중에 우연히 동작하던 코드가 실제 배포 후에는 동작하지 않는 문제를 미리 발견할 수 있다.

**\_\_init\_\_.py의 역할**

```python
# src/mypackage/__init__.py
from .calculator import add, divide
from .formatter import format_currency

__version__ = "0.1.0"
```

- `__init__.py` 파일이 폴더 안에 있으면, 파이썬은 그 폴더를 하나의 **패키지**로 인식한다. 이 파일 자체는 비어 있어도 되지만, 보통은 그 패키지를 사용하는 쪽에 어떤 이름들을 공개할지 정리해두는 용도로 쓰인다.
- 위 예제처럼 `calculator.py`의 `add`, `divide`와 `formatter.py`의 `format_currency`를 `__init__.py`에서 미리 `import`해두면, 사용하는 쪽에서는 `from mypackage.calculator import add` 대신 `from mypackage import add`처럼 더 짧고 단순한 경로로 접근할 수 있다. 이렇게 패키지의 공개 API를 한곳에 모아두는 것을 관례로 삼는다.
- `__version__` 같은 변수를 함께 정의해두면, 이 패키지를 설치한 쪽에서 `mypackage.__version__`으로 현재 설치된 버전을 코드에서 바로 확인할 수 있다.

**정리**: 표준적인 파이썬 패키지는 `src/패키지이름/` 폴더 아래에 실제 코드를 두는 src 레이아웃을 따르는 것이 권장되며, 이는 설치되지 않은 소스 코드가 우연히 `import`되어 착각을 일으키는 상황을 방지해주고, 각 패키지 폴더에 있는 `__init__.py`는 그 폴더를 패키지로 인식시킴과 동시에 내부 모듈의 함수·클래스 중 외부에 공개할 것들을 모아 더 간단한 경로로 접근할 수 있게 해주는 역할을 한다.

## pyproject.toml 구조

**설정 파일의 역사 — setup.py에서 pyproject.toml로**

과거의 파이썬 패키징은 `setup.py`라는 파이썬 스크립트 안에 `setuptools.setup(name=..., version=..., ...)` 형태로 패키지 정보를 코드로 작성하는 방식이 표준이었다. 이 방식은 설정 파일 자체가 실행 가능한 코드였기 때문에, 패키지 정보를 읽어오는 도구 입장에서는 그 코드를 실행해야만 값을 알 수 있다는 불편함이 있었다. 현재는 이런 문제를 해결하기 위해 실행 코드가 아닌 순수한 설정 데이터 형식인 **`pyproject.toml`**을 사용하는 방식이 표준으로 자리 잡았다. 이 문서에서는 `pyproject.toml` 기반의 현재 표준 방식만 다룬다.

**[build-system] 섹션**

```toml
[build-system]
requires = ["setuptools>=68.0"]
build-backend = "setuptools.build_meta"
```

- `[build-system]`은 이 패키지를 빌드(배포용 아카이브로 만드는 작업)할 때 어떤 도구를 사용할지 지정하는 섹션이다.
- `requires`는 빌드 과정에 필요한 도구들의 목록이다. 위 예제는 `setuptools` 68.0 이상 버전이 필요하다는 뜻이다. `setuptools` 대신 `hatchling`, `flit_core` 같은 다른 빌드 백엔드를 쓸 수도 있으며, 어떤 것을 쓰든 기본적인 설정 구조는 크게 다르지 않다.
- `build-backend`는 실제로 빌드를 수행하는 백엔드의 경로를 지정한다. `setuptools.build_meta`는 `setuptools`가 제공하는 표준 빌드 백엔드다.

**[project] 섹션**

```toml
[project]
name = "mypackage"
version = "0.1.0"
description = "예제로 만들어보는 간단한 계산 유틸리티 패키지"
readme = "README.md"
requires-python = ">=3.10"
dependencies = [
    "requests>=2.28",
]

[project.optional-dependencies]
dev = ["pytest>=7.0"]
```

- `name`은 `pip install`할 때 쓰일 패키지 이름이다. 이 이름은 PyPI(잠시 뒤에 다룰 패키지 저장소) 전체에서 고유해야 하며, `import`할 때 쓰는 이름(`src/mypackage/`의 `mypackage`)과 반드시 같을 필요는 없지만 혼동을 줄이기 위해 같게 맞추는 경우가 많다.
- `version`은 이 패키지의 현재 버전이다. 버전을 표기하는 관례는 다음 절에서 다룬다.
- `description`은 패키지를 한 줄로 요약하는 설명이며, `readme`는 더 자세한 설명이 담긴 파일(보통 `README.md`)을 가리킨다.
- `requires-python`은 이 패키지가 동작을 보장하는 파이썬 버전 범위다. `">=3.10"`은 파이썬 3.10 이상에서만 설치를 허용한다는 뜻이며, 그보다 낮은 버전에서 설치를 시도하면 `pip`가 설치를 거부하고 오류를 안내한다.
- `dependencies`는 이 패키지가 동작하기 위해 함께 설치되어야 하는 다른 라이브러리 목록이다. `requirements.txt`와 비슷해 보이지만, `dependencies`는 이 패키지를 설치하는 사람에게 "함께 설치해야 할 목록"으로 전달되어 `pip install mypackage`만 실행해도 자동으로 `requests`까지 함께 설치된다는 점에서 차이가 있다.
- `[project.optional-dependencies]`는 필수는 아니지만 특정 용도(위 예제는 개발용 `dev`)로 묶어서 설치할 수 있는 선택적 의존성 그룹이다. `pip install "mypackage[dev]"`처럼 대괄호로 그룹 이름을 지정하면 기본 의존성과 함께 `pytest`까지 설치된다.

**정리**: `pyproject.toml`은 과거 `setup.py`가 코드로 작성하던 패키지 설정 정보를 순수한 TOML 데이터 형식으로 표현한 현재의 표준 설정 파일이며, `[build-system]`에서 이 패키지를 빌드할 도구를, `[project]`에서 패키지 이름·버전·설명·지원 파이썬 버전·의존성 같은 핵심 정보를 지정해두면 `pip`나 빌드 도구가 이 파일 하나만 읽어 패키지를 설치하거나 빌드하는 데 필요한 모든 정보를 얻을 수 있다.

## 로컬 설치와 빌드

**pip install -e . — 개발 모드 설치**

패키지를 PyPI에 올리기 전에, 로컬에서 먼저 이 패키지가 제대로 동작하는지 테스트해보고 싶은 경우가 많다. 이때 쓰는 것이 **editable 설치**다.

```text
(.venv) C:\Users\guest\mypackage-project> python -m pip install -e .
```

```text
(.venv) C:\Users\guest\mypackage-project> python
>>> from mypackage import add
>>> add(2, 3)
5
```

- `pip install -e .`에서 `.`은 현재 폴더(`pyproject.toml`이 있는 폴더)를 뜻하며, `-e`(`--editable`)는 이 패키지를 "편집 가능한 상태"로 설치하라는 옵션이다.
- 일반적인 `pip install`은 그 시점의 코드를 복사해 설치 위치에 넣어두지만, editable 설치는 실제 파일을 복사하지 않고 현재 소스 코드 폴더를 그대로 가리키는 링크만 만들어둔다.
- 그 결과 `src/mypackage/calculator.py`의 코드를 수정하면, 다시 설치하지 않아도 그 변경 사항이 바로 반영된 상태로 `import mypackage`할 수 있다. 이는 패키지를 개발하는 동안 코드를 수정할 때마다 매번 재설치할 필요가 없게 해주므로, 패키징 작업 중 로컬 테스트에 필수적으로 쓰이는 방식이다.

**python -m build — 배포용 아카이브 만들기**

로컬 테스트가 끝나고 이 패키지를 다른 곳에 배포할 준비가 되면, 실제로 배포할 수 있는 아카이브 파일을 만들어야 한다.

```text
(.venv) C:\Users\guest\mypackage-project> python -m pip install build
(.venv) C:\Users\guest\mypackage-project> python -m build
```

```text
* Creating isolated environment: venv+pip...
* Installing packages in isolated environment:
  - setuptools>=68.0
* Building sdist...
* Building wheel from sdist...
Successfully built mypackage-0.1.0.tar.gz and mypackage-0.1.0-py3-none-any.whl
```

- `python -m build`는 `build`라는 별도 패키지가 제공하는 명령으로, 먼저 `pip install build`로 설치해두어야 한다.
- 이 명령을 실행하면 `pyproject.toml`의 `[build-system]` 설정을 읽어 두 가지 형식의 아카이브를 만든다.
- `.tar.gz`(소스 배포판, sdist)는 패키지의 소스 코드 전체를 압축한 파일이다. 이 파일을 설치하는 컴퓨터에서 빌드 과정을 다시 거치게 된다.
- `.whl`(wheel, 빌드된 배포판)은 이미 빌드가 끝난 상태로 배포되는 형식으로, 설치할 때 별도의 빌드 과정 없이 파일을 그대로 풀어 넣기만 하면 되므로 설치 속도가 더 빠르다. 특별한 사정이 없다면 `pip install`은 가능한 경우 `.whl` 파일을 우선적으로 사용한다.
- 빌드가 끝나면 프로젝트 폴더 안에 `dist/` 폴더가 새로 생기고, 그 안에 `mypackage-0.1.0.tar.gz`와 `mypackage-0.1.0-py3-none-any.whl` 두 파일이 만들어진다. 이 `dist/` 폴더 안의 파일들이 실제로 배포에 쓰이는 결과물이다.

**정리**: 패키지를 배포하기 전에는 `pip install -e .`로 로컬에 편집 가능한 상태로 설치해 코드를 수정할 때마다 재설치 없이 바로 테스트해볼 수 있고, 로컬 테스트가 끝나면 `python -m build`로 `pyproject.toml` 설정을 기반으로 `dist/` 폴더 안에 소스 배포판(`.tar.gz`)과 빌드된 배포판(`.whl`)을 만들어 이 파일들을 실제 배포에 사용하게 된다.

## PyPI 업로드 개념

**PyPI란**

**PyPI(Python Package Index)**는 파이썬 커뮤니티가 공식적으로 운영하는 패키지 저장소로, 우리가 지금까지 `pip install`로 설치해온 `requests`, `pandas`, `numpy` 같은 거의 모든 외부 라이브러리가 이곳에 등록되어 있다. `python -m build`로 만든 `dist/` 폴더 안의 아카이브 파일들을 PyPI에 업로드하면, 전 세계 누구나 내 패키지를 `pip install mypackage`로 설치할 수 있게 된다.

**twine upload로 업로드하기**

```text
(.venv) C:\Users\guest\mypackage-project> python -m pip install twine
(.venv) C:\Users\guest\mypackage-project> python -m twine upload dist/*
```

```text
Uploading distributions to https://upload.pypi.org/legacy/
Enter your API token:
Uploading mypackage-0.1.0-py3-none-any.whl
Uploading mypackage-0.1.0.tar.gz

View at:
https://pypi.org/project/mypackage/0.1.0/
```

- `twine`은 PyPI에 패키지를 업로드하는 데 특화된 표준 도구로, `pip install twine`으로 먼저 설치해야 한다.
- `twine upload dist/*`는 `dist/` 폴더 안의 모든 아카이브 파일(`.tar.gz`와 `.whl`)을 PyPI에 업로드한다.
- 업로드하려면 PyPI 계정과 함께 **API 토큰**을 이용한 인증이 필요하다. PyPI는 오래전부터 계정의 아이디·비밀번호를 직접 입력하는 방식 대신, PyPI 웹사이트에서 미리 발급받은 API 토큰을 사용하는 방식을 요구하고 있다. 이 토큰은 계정 설정 페이지에서 발급받아 안전하게 보관해야 하며, 이 문서에서는 실제 계정 생성이나 토큰 발급 절차는 다루지 않는다.
- 업로드가 완료되면 `https://pypi.org/project/패키지이름/버전/` 형태의 주소에서 방금 올린 패키지 페이지를 확인할 수 있고, 이때부터 전 세계 누구나 `pip install mypackage`로 이 패키지를 설치할 수 있게 된다.

**TestPyPI로 먼저 연습하기**

실제 PyPI에 한 번 업로드한 패키지 이름과 버전은 삭제하더라도 같은 이름·버전으로 다시 업로드할 수 없는 등 제약이 있으므로, 업로드 과정 자체가 처음이라면 실제 PyPI가 아닌 **TestPyPI**라는 연습용 저장소를 먼저 사용해보는 것이 권장된다.

```text
(.venv) C:\Users\guest\mypackage-project> python -m twine upload --repository testpypi dist/*
```

- `--repository testpypi` 옵션을 주면 실제 PyPI가 아니라 테스트 전용으로 운영되는 TestPyPI(`test.pypi.org`)에 업로드된다. TestPyPI는 실제 PyPI와 별개의 계정과 API 토큰을 사용한다.
- TestPyPI에서 이름 충돌이나 메타데이터 오류 없이 업로드와 설치(`pip install --index-url https://test.pypi.org/simple/ mypackage`)까지 문제없이 되는 것을 확인한 뒤, 실제 PyPI에 정식으로 업로드하는 순서로 진행하면 실수를 줄일 수 있다.

**정리**: PyPI는 `pip install`로 설치되는 대부분의 외부 라이브러리가 등록된 공식 저장소이며, `python -m build`로 만든 `dist/` 폴더의 아카이브를 `twine upload dist/*` 명령으로 올리면 API 토큰 인증을 거쳐 전 세계에 배포할 수 있게 되고, 업로드 절차 자체가 처음이라면 실제 PyPI 대신 `--repository testpypi` 옵션으로 TestPyPI에 먼저 연습 삼아 올려보는 것이 안전한 접근이다.

## 버전 관리 관례

**시맨틱 버저닝 — MAJOR.MINOR.PATCH**

`pyproject.toml`의 `version` 필드에 어떤 값을 적을지는 임의로 정하는 것이 아니라, 파이썬을 포함한 대부분의 언어 생태계에서 널리 쓰이는 **시맨틱 버저닝(Semantic Versioning)** 관례를 따르는 것이 일반적이다. 버전은 `MAJOR.MINOR.PATCH` 형태의 세 숫자로 구성된다.

- **MAJOR**(주 버전)는 기존 사용 방식과 호환되지 않는 큰 변경이 있을 때 올린다. 예를 들어 함수의 이름이 바뀌거나 인자 순서가 바뀌어, 이전 버전을 쓰던 코드가 새 버전에서는 그대로 동작하지 않게 되는 경우다.
- **MINOR**(부 버전)는 기존 기능과 호환을 유지하면서 새로운 기능을 추가했을 때 올린다. 기존에 이 패키지를 쓰던 코드는 그대로 잘 동작하면서, 새로운 함수나 옵션이 추가로 생긴 경우다.
- **PATCH**(패치 버전)는 기능 추가 없이 버그만 수정했을 때 올린다. 사용하는 쪽에서는 아무것도 바뀐 것을 느끼지 못하지만, 내부적으로 잘못된 동작이 고쳐진 경우다.
- 예를 들어 `1.2.3`에서 사소한 버그를 고치면 `1.2.4`가 되고, 호환되는 새 함수를 추가하면 `1.3.0`(부 버전이 오르면 패치 번호는 0으로 초기화)이 되며, 기존 함수의 사용법 자체를 바꾸는 큰 변경이라면 `2.0.0`이 된다.
- 이 관례를 따르면, 이 패키지를 사용하는 쪽에서는 버전 번호만 보고도 "이번 업데이트가 안전하게 반영해도 되는 수준인지, 아니면 내 코드도 함께 확인해야 하는 큰 변경인지"를 어느 정도 짐작할 수 있다.

**정리**: `pyproject.toml`의 `version`은 `MAJOR.MINOR.PATCH` 형태의 시맨틱 버저닝 관례를 따라, 호환되지 않는 큰 변경에는 MAJOR를, 호환되는 기능 추가에는 MINOR를, 버그 수정에는 PATCH를 올리는 방식으로 관리하면, 이 패키지를 설치해 쓰는 사람들이 버전 번호만으로도 업데이트의 영향 범위를 짐작할 수 있게 된다.

## 실습 예제 (EX1~EX4)

**EX1) src 레이아웃으로 간단한 패키지 구조 작성한다**
- `greetings`라는 이름의 패키지를 src 레이아웃으로 구성하고, 인사말을 만드는 함수를 `__init__.py`에서 바로 접근할 수 있게 정리한다.

```python
# src/greetings/messages.py
def hello(name):
    return f"안녕하세요, {name}님!"

def goodbye(name):
    return f"{name}님, 다음에 또 만나요."
```

```python
# src/greetings/__init__.py
from .messages import hello, goodbye

__version__ = "0.1.0"
```

```text
(base) C:\Users\guest\greetings-project> python -m pip install -e .
(base) C:\Users\guest\greetings-project> python
>>> from greetings import hello, goodbye
>>> hello("영희")
'안녕하세요, 영희님!'
>>> goodbye("영희")
'영희님, 다음에 또 만나요.'
```

**EX2) pyproject.toml을 작성하고 의존성을 확인한다**
- `greetings` 패키지의 `pyproject.toml`을 작성해, 패키지 정보와 함께 하나의 외부 의존성을 지정한다.

```toml
# pyproject.toml
[build-system]
requires = ["setuptools>=68.0"]
build-backend = "setuptools.build_meta"

[project]
name = "greetings"
version = "0.1.0"
description = "인사말을 만들어주는 간단한 예제 패키지"
requires-python = ">=3.10"
dependencies = [
    "colorama>=0.4",
]
```

```text
(base) C:\Users\guest\greetings-project> python -m pip install -e .
(base) C:\Users\guest\greetings-project> python -m pip show greetings
Name: greetings
Version: 0.1.0
Requires: colorama
```

- `pip install -e .`를 실행하면 `dependencies`에 적힌 `colorama`도 함께 설치되며, `pip show greetings`의 `Requires` 항목에서 이 패키지가 `colorama`에 의존하고 있음을 확인한다.

**EX3) python -m build로 배포용 아카이브를 만들어본다**
- `greetings` 패키지를 빌드해 `dist/` 폴더 안에 어떤 파일들이 생성되는지 확인하고 작성한다.

```text
(base) C:\Users\guest\greetings-project> python -m pip install build
(base) C:\Users\guest\greetings-project> python -m build
(base) C:\Users\guest\greetings-project> dir dist
```

```text
2026-09-10  14:30    greetings-0.1.0-py3-none-any.whl
2026-09-10  14:30    greetings-0.1.0.tar.gz
```

- 빌드가 끝난 뒤 `dist/` 폴더 안에 `.whl`(빌드된 배포판)과 `.tar.gz`(소스 배포판) 두 파일이 만들어졌는지 비교한다.

**EX4) 시맨틱 버저닝에 맞춰 버전을 올바르게 계산한다**
- 변경 내용에 따라 `MAJOR.MINOR.PATCH` 중 어느 자리를 올려야 하는지 판단하는 함수를 작성해 여러 시나리오를 비교한다.

```python
# ex04_semver_bump.py
def bump_version(version, change_type):
    major, minor, patch = (int(x) for x in version.split("."))
    if change_type == "major":
        return f"{major + 1}.0.0"
    if change_type == "minor":
        return f"{major}.{minor + 1}.0"
    if change_type == "patch":
        return f"{major}.{minor}.{patch + 1}"
    raise ValueError("change_type은 major/minor/patch 중 하나여야 합니다.")

current = "1.2.3"
print("버그 수정 후:", bump_version(current, "patch"))
print("호환 기능 추가 후:", bump_version(current, "minor"))
print("호환 불가 변경 후:", bump_version(current, "major"))
```

```text
(base) C:\Users\guest\greetings-project> python ex04_semver_bump.py
버그 수정 후: 1.2.4
호환 기능 추가 후: 1.3.0
호환 불가 변경 후: 2.0.0
```

- 부 버전을 올릴 때는 패치 번호가 `0`으로 초기화되고, 주 버전을 올릴 때는 부 버전과 패치 번호가 모두 `0`으로 초기화되는 것을 각 출력에서 확인한다.

**정리**: EX1~EX4는 `greetings`라는 예제 패키지를 src 레이아웃으로 구성해 `__init__.py`로 공개 API를 정리하는 과정, `pyproject.toml`에 패키지 정보와 의존성을 작성하고 `pip install -e .`로 그 의존성이 함께 설치되는지 확인하는 과정, `python -m build`로 실제 배포용 아카이브 두 종류를 만들어보는 과정, 그리고 시맨틱 버저닝 규칙에 따라 변경 유형별로 버전 번호가 어떻게 올라가는지 계산해보는 과정까지, 이 문서에서 다룬 패키징의 전체 흐름을 직접 코드와 명령어로 확인해보는 예제다.

[Python 17 — 언어 심화](17-language-advanced.md)

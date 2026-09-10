# Python 00 — Python 설치

참고: https://www.anaconda.com/ , https://code.visualstudio.com/

## 온라인 컴파일러로 설치 없이 실행

Programiz, Online-Python 같은 사이트는 브라우저에서 바로 파이썬 코드를 작성하고 실행할 수 있는 온라인 컴파일러를 제공한다.

- 별도 설치·회원가입 없이 코드 입력창에 작성 후 실행 버튼만 누르면 결과가 바로 출력된다.
- PC에 설치 권한이 없거나 문법을 잠깐 테스트해볼 때 유용하다.
- 파일 저장, 외부 라이브러리 설치, 여러 파일로 구성된 프로젝트 실행은 지원하지 않거나 제한적이므로, 본격적인 학습에는 로컬 설치가 필요하다.

**정리**: 온라인 컴파일러는 설치 없이 빠르게 문법을 확인하는 용도로 쓰고, 이어지는 절에서 로컬 PC에 Anaconda와 VS Code를 설치해 정식 개발 환경을 구성한다.

## Anaconda 설치 (Windows)

**Anaconda**는 파이썬 인터프리터, 데이터 분석에 자주 쓰이는 라이브러리(NumPy, Pandas 등), 패키지·가상환경 관리 도구(conda)를 한 번에 제공하는 배포판이다.

### 1) 설치 파일 다운로드

Anaconda 공식 홈페이지 접속 후 운영체제에 맞는 설치 파일(Windows/macOS/Linux)을 내려받는다.

### 2) 설치 마법사 진행

```text
1. 설치 파일 실행
2. Welcome 화면 -> Next
3. License Agreement -> I Agree
4. Select Installation Type -> Just Me (recommended)   # 현재 계정에만 설치
5. Choose Install Location -> 기본 경로 유지 (변경 가능)
6. Advanced Installation Options
   [ ] Add Anaconda3 to my PATH environment variable
   [v] Register Anaconda3 as my default Python 3.x
7. Install -> 설치 진행 (수 분 소요)
8. Installation Complete -> Next -> Finish
```

- `Add Anaconda3 to my PATH environment variable`은 기본적으로 체크 해제 상태다. 체크하면 일반 명령 프롬프트(cmd)에서도 바로 `python`, `conda` 명령을 쓸 수 있지만, 시스템에 이미 설치된 다른 파이썬 버전과 충돌할 수 있어 공식적으로는 비권장이다 — 체크하지 않아도 **Anaconda Prompt**에서는 항상 정상적으로 사용할 수 있다.

### 3) Anaconda Prompt로 설치 확인

시작 메뉴에서 **Anaconda Prompt**를 실행한다.

```text
(base) C:\Users\guest> conda --version
conda 24.9.2

(base) C:\Users\guest> python --version
Python 3.12.4
```

- 프롬프트 맨 앞의 `(base)`는 현재 활성화된 conda 가상환경 이름이다. 설치 직후에는 `base`라는 기본 환경이 자동으로 활성화되어 있다.

**정리**: Anaconda는 설치 마법사(Just Me → 기본 경로 → Install)만 그대로 따라가면 되며, 설치 직후에는 시작 메뉴의 Anaconda Prompt를 사용해야 `(base)` 환경이 활성화된 상태로 `python`, `conda` 명령을 바로 쓸 수 있다.

## Visual Studio Code + Python 확장 설치

### 1) VS Code 설치

VS Code 공식 홈페이지에서 설치 파일을 내려받아 실행하고, 설치 마법사의 기본 옵션을 그대로 진행한다("바탕화면에 바로 가기 만들기", "PATH에 추가" 같은 옵션은 필요에 따라 선택).

### 2) Python 확장 설치

```text
1. VS Code 실행
2. 왼쪽 사이드바 -> Extensions(확장) 아이콘 클릭
3. 검색창에 "Python" 입력
4. 게시자가 Microsoft인 "Python" 확장 -> Install
```

확장을 설치하면 VS Code가 시스템에 설치된 파이썬 인터프리터(Anaconda 포함)를 자동으로 인식하고, 코드 실행·디버깅·자동완성 기능이 활성화된다. 화면 우측 하단에 현재 선택된 인터프리터가 표시되며, 클릭하면 다른 버전으로 바꿀 수 있다.

```text
우측 하단 상태 표시줄 예시
Python 3.12.4 ('base': conda)
```

### 3) 동작 확인

VS Code에서 새 파일을 만들어 `hello.py`로 저장한다.

```python
# hello.py
print("Hello, Python!")
```

편집기 우측 상단의 실행(▷) 버튼을 클릭하거나, 통합 터미널에서 직접 실행한다.

```text
PS C:\Users\guest\project> python hello.py
Hello, Python!
```

**정리**: VS Code는 특정 언어에 종속되지 않은 범용 에디터이므로, Python 확장을 설치해야 인터프리터 인식·실행·디버깅 같은 파이썬 전용 기능이 활성화된다.

## 설치 확인

Anaconda와 VS Code 설치가 끝나면 아래 두 가지 실행 방식으로 최종 확인한다.

### 1) 대화형 인터프리터(REPL)

```text
(base) C:\Users\guest> python
Python 3.12.4 (main, Jun 18 2024, 10:07:17) [MSC v.1938 64 bit (AMD64)] on win32
Type "help", "copyright", "credits" or "license" for more information.
>>> print("설치 확인 완료")
설치 확인 완료
>>> exit()
```

### 2) 스크립트 파일 실행

```python
# check_install.py
print("파이썬 실행 환경이 정상적으로 구성되었습니다.")
```

```text
(base) C:\Users\guest\project> python check_install.py
파이썬 실행 환경이 정상적으로 구성되었습니다.
```

**정리**: `python`만 입력하면 한 줄씩 입력하고 바로 결과를 확인하는 **대화형 인터프리터(REPL)**가 실행되고, `python 파일명.py`처럼 파일을 지정하면 스크립트 전체가 위에서부터 순서대로 실행된다. 이 두 실행 방식은 이후 모든 파이썬 문법 학습에서 계속 사용된다.


## pip 기초

**pip**는 파이썬 패키지(외부 라이브러리)를 설치·삭제·관리하는 표준 명령줄 도구다. Anaconda Prompt나 VS Code 터미널에서 `pip` 명령으로 바로 사용할 수 있다.

### 패키지 설치

```text
(base) C:\Users\guest\project> pip install requests
Collecting requests
  Downloading requests-2.32.3-py3-none-any.whl (64 kB)
Installing collected packages: requests
Successfully installed requests-2.32.3
```

- 버전을 지정해서 설치하려면 패키지 이름 뒤에 `==버전번호`를 붙인다.

```text
(base) C:\Users\guest\project> pip install requests==2.31.0
```

### 설치된 패키지 확인

```text
(base) C:\Users\guest\project> pip list
Package            Version
------------------ -------
pip                24.0
requests            2.32.3
```

- 특정 패키지 하나의 상세 정보(버전, 설치 위치, 의존 패키지 등)는 `pip show`로 확인한다.

```text
(base) C:\Users\guest\project> pip show requests
Name: requests
Version: 2.32.3
Summary: Python HTTP for Humans.
Location: C:\Users\guest\anaconda3\Lib\site-packages
Requires: certifi, charset-normalizer, idna, urllib3
```

### 패키지 삭제

```text
(base) C:\Users\guest\project> pip uninstall requests
Found existing installation: requests 2.32.3
Uninstalling requests-2.32.3:
  Would remove:
    ...
Proceed (Y/n)? y
Successfully uninstalled requests-2.32.3
```

### requirements.txt로 패키지 목록 관리

현재 환경에 설치된 패키지 목록을 파일로 저장해두면, 같은 환경을 다른 PC에서도 동일하게 재현할 수 있다.

```text
(base) C:\Users\guest\project> pip freeze > requirements.txt
```

- `requirements.txt`에는 `패키지명==버전` 형식으로 설치된 패키지 목록이 한 줄씩 기록된다.
- 이 파일을 다른 환경에서 그대로 설치하려면 `-r` 옵션을 사용한다.

```text
(base) C:\Users\guest\project> pip install -r requirements.txt
```

**정리**: pip는 `install`로 패키지를 설치하고 `list`/`show`로 설치 상태를 확인하며 `uninstall`로 제거하는 파이썬 표준 패키지 관리 도구이며, `pip freeze > requirements.txt`와 `pip install -r requirements.txt`를 짝지어 사용하면 프로젝트의 패키지 구성을 그대로 옮기거나 공유할 수 있다.

## 가상환경(virtual environment) 기초

여러 프로젝트를 동시에 진행하다 보면 프로젝트마다 필요한 패키지 버전이 서로 다를 수 있다. 하나의 파이썬 환경에 모든 패키지를 섞어서 설치하면, 한 프로젝트에서 특정 패키지를 업그레이드했을 때 다른 프로젝트가 예전 버전에 의존하고 있어 충돌이 발생할 수 있다.

**가상환경(virtual environment)**은 프로젝트별로 독립된 파이썬 실행 환경과 패키지 설치 공간을 따로 만들어, 이런 버전 충돌을 막는 장치다. 가상환경을 만드는 대표적인 방법은 두 가지다.

### 1) 표준 라이브러리 venv 사용

파이썬에는 별도 설치 없이 바로 쓸 수 있는 `venv` 모듈이 내장되어 있다.

```text
PS C:\Users\guest\project> python -m venv venv
```

- 위 명령을 실행하면 현재 폴더에 `venv`라는 이름의 하위 폴더가 만들어지고, 그 안에 독립된 파이썬 실행 파일과 패키지 설치 공간이 구성된다.

**활성화(activate)**

```text
PS C:\Users\guest\project> .\venv\Scripts\Activate.ps1
(venv) PS C:\Users\guest\project>
```

- PowerShell에서는 cmd.exe와 달리 확장자를 자동으로 붙여주지 않으므로, 배치 파일인 `activate.bat`가 아니라 PowerShell 전용 스크립트 `Activate.ps1`을 `.\`(현재 경로) 접두사와 함께 명시해서 실행해야 한다. (명령 프롬프트(cmd.exe)에서는 `venv\Scripts\activate.bat`를 사용한다.)
- 활성화되면 프롬프트 맨 앞에 `(venv)`처럼 현재 활성화된 가상환경 이름이 표시되어, 지금 어떤 환경에서 작업 중인지 바로 확인할 수 있다.
- 이 상태에서 `pip install`로 설치한 패키지는 전역 환경이 아니라 `venv` 폴더 안에만 설치된다.

**비활성화(deactivate)**

```text
(venv) PS C:\Users\guest\project> deactivate
PS C:\Users\guest\project>
```

### 2) Anaconda의 conda 환경 사용

Anaconda를 설치했다면 `conda` 명령으로도 동일한 목적의 가상환경을 만들 수 있다.

```text
(base) C:\Users\guest> conda create -n myenv python=3.12
```

- `-n myenv`는 새로 만들 환경의 이름을 지정하는 옵션이고, `python=3.12`는 그 환경에 설치할 파이썬 버전을 지정한다.

**활성화(activate)**

```text
(base) C:\Users\guest> conda activate myenv
(myenv) C:\Users\guest>
```

- venv와 마찬가지로 프롬프트 맨 앞이 `(base)`에서 `(myenv)`로 바뀌어, 현재 `myenv` 환경이 활성화되었음을 보여준다.

**비활성화(deactivate)**

```text
(myenv) C:\Users\guest> conda deactivate
(base) C:\Users\guest>
```

- conda 환경은 완전히 빠져나오지 않는 이상 항상 어떤 환경이든 하나가 활성화된 상태이므로, `deactivate`를 실행하면 이전 환경(보통 `base`)으로 돌아간다.

**정리**: 가상환경은 프로젝트별로 독립된 패키지 설치 공간을 분리해 버전 충돌을 막는 장치이며, 표준 라이브러리 `python -m venv`와 Anaconda의 `conda create -n`은 만드는 방법만 다를 뿐 목적은 같고, 두 방식 모두 활성화 시 프롬프트 앞에 현재 환경 이름이 표시되며 `deactivate`로 빠져나온다.

## VS Code에서 가상환경 인터프리터 선택하기

앞서 "Visual Studio Code + Python 확장 설치"에서 살펴본 우측 하단 상태 표시줄의 인터프리터 표시는, 새로 만든 가상환경으로도 그대로 전환할 수 있다.

```text
1. VS Code에서 프로젝트 폴더 열기 (venv 또는 conda 환경을 이미 만들어 둔 상태)
2. 화면 우측 하단 상태 표시줄의 파이썬 버전 표시 클릭
   (또는 Ctrl+Shift+P -> "Python: Select Interpreter" 입력)
3. 목록에서 원하는 인터프리터 선택
   예) Python 3.12.4 ('venv': venv)  ./venv/Scripts/python.exe
   예) Python 3.12.4 ('myenv': conda)
```

- 목록에는 시스템에 설치된 파이썬과 함께, 현재 프로젝트 폴더 하위에서 발견된 `venv` 가상환경, 그리고 conda로 만든 환경들이 함께 표시된다.
- 인터프리터를 전환하면 그 이후 통합 터미널에서 새로 여는 세션과 코드 실행(▷) 버튼이 모두 선택한 환경을 기준으로 동작한다.
- 상태 표시줄에 표시되는 이름이 원하는 가상환경 이름과 일치하는지 확인하는 것이 가장 빠른 확인 방법이다.

**정리**: VS Code는 프로젝트 폴더 안의 venv와 시스템에 등록된 conda 환경을 자동으로 탐지하므로, 우측 하단 상태 표시줄이나 "Python: Select Interpreter" 명령으로 원하는 가상환경을 선택하면 이후 실행·디버깅이 모두 그 환경 기준으로 이루어진다.

## 자주 겪는 설치 문제 해결

### `python`이나 `conda` 명령을 찾을 수 없다는 오류

```text
PS C:\Users\guest> python --version
'python'은(는) 내부 또는 외부 명령, 실행할 수 있는 프로그램, 또는
배치 파일이 아닙니다.
```

- Anaconda 설치 시 `Add Anaconda3 to my PATH environment variable`을 체크하지 않았다면, 일반 명령 프롬프트나 PowerShell에서는 `python`, `conda` 명령이 인식되지 않는다.
- 이 경우 시작 메뉴에서 **Anaconda Prompt**를 실행해서 작업하면 별도 설정 없이 바로 사용할 수 있다.

### 여러 파이썬 버전이 설치되어 있을 때

시스템에 파이썬이 여러 개 설치되어 있으면 `python` 명령이 어느 버전을 가리키는지 헷갈릴 수 있다. 이때는 `--version`으로 실제 실행되는 버전을 확인한다.

```text
(base) C:\Users\guest> python --version
Python 3.12.4
```

- 가상환경을 활성화한 상태에서 같은 명령을 실행하면, 그 가상환경에 설치된 파이썬 버전이 대신 출력된다. 즉 `python --version`의 결과는 현재 활성화된 환경에 따라 달라진다.

### pip로 설치한 패키지를 import할 수 없을 때

```text
>>> import requests
ModuleNotFoundError: No module named 'requests'
```

- 패키지를 분명히 `pip install`로 설치했는데도 이런 오류가 나면, 패키지를 설치한 환경과 현재 코드를 실행 중인 환경(인터프리터)이 서로 다른 경우가 대부분이다.
- 예를 들어 `(base)`에 설치했지만 VS Code는 `(myenv)` 인터프리터로 실행 중이라면, `(myenv)`에는 그 패키지가 없어서 오류가 발생한다.
- 해결하려면 코드를 실행할 환경을 활성화한 상태에서 다시 `pip install`을 실행하거나, VS Code의 인터프리터 선택을 패키지가 설치된 환경으로 맞춘다.

**정리**: 설치 관련 문제 대부분은 PATH 미등록(Anaconda Prompt 사용으로 해결), 여러 버전 공존(`--version`으로 실제 실행 버전 확인), 설치 환경과 실행 환경 불일치(가상환경/인터프리터 선택 확인) 세 가지로 좁혀지며, 오류 메시지만 보고 당황하기보다 "지금 어떤 환경이 활성화되어 있는가"를 먼저 확인하는 습관이 문제 해결의 출발점이다.

## 실습 예제 (EX1~EX3)

**EX1) 가상환경 생성과 activate 확인**
- `python -m venv` 로 가상환경을 만들고 activate한 뒤, 프롬프트에 환경 이름이 표시되는지 확인한다.

```text
PS C:\Users\guest\project> python -m venv venv
PS C:\Users\guest\project> .\venv\Scripts\Activate.ps1
(venv) PS C:\Users\guest\project> python --version
Python 3.12.4
```

**EX2) 패키지 설치 후 requirements.txt 생성**
- 활성화한 가상환경에 `requests` 패키지를 설치하고, `pip freeze`로 `requirements.txt`를 만드시오.

```text
(venv) PS C:\Users\guest\project> pip install requests
(venv) PS C:\Users\guest\project> pip freeze > requirements.txt
(venv) PS C:\Users\guest\project> type requirements.txt
requests==2.32.3
```

**EX3) VS Code에서 새 가상환경으로 인터프리터 전환**
- VS Code에서 EX1에서 만든 `venv` 환경을 인터프리터로 선택한 뒤, `hello.py`를 실행해 정상 동작을 확인한다.

```text
1. VS Code 상태 표시줄 클릭 -> "Python: Select Interpreter"
2. Python 3.12.4 ('venv': venv) 선택
3. 통합 터미널 새로 열기 -> 프롬프트 앞에 (venv) 표시 확인
4. hello.py 실행 -> Hello, Python! 출력 확인
```

**정리**: EX1~EX3은 가상환경을 만들고(activate 확인) 패키지를 설치해 requirements.txt로 남긴 뒤, VS Code 인터프리터를 그 가상환경으로 맞추는 흐름을 통해 앞선 pip·가상환경·VS Code 설정 내용을 하나의 작업 순서로 이어서 연습하는 예제다.

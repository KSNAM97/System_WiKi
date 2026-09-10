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

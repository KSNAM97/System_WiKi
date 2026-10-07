# Python 16 — 표준 라이브러리 심화와 테스트

PY-14에서는 `sys`, `glob`, `re`, `datetime`, `random`, `time`, `collections`처럼 실무에서 자주 쓰이는 표준 라이브러리를 살펴봤다. 이 문서에서는 조금 더 규모 있는 프로그램을 만들 때 필요해지는 네 가지 주제, 즉 로그 남기기, 여러 작업을 동시에 실행하기, 인터넷 자원에 접근하기, 그리고 작성한 코드가 올바르게 동작하는지 검증하는 테스트 작성법을 다룬다.

## logging 모듈

**`print()` 디버깅의 한계**

지금까지는 코드 중간에 값을 확인하고 싶을 때 `print()`를 사용해왔다. 개발 초기 단계에서는 이것으로 충분하지만, 프로그램이 커지면 몇 가지 문제가 드러난다.

- 배포된 프로그램에서 디버깅용 `print()`를 지우지 않으면 불필요한 출력이 그대로 섞여 나간다. 반대로 지워버리면 나중에 문제가 생겼을 때 다시 하나하나 추가해야 한다.
- `print()`는 "지금 출력할지 말지"를 선택할 방법이 없다. 개발 중에는 자세한 정보를 보고 싶고 운영 중에는 오류만 보고 싶은 것처럼, 상황에 따라 출력 수준을 조절하는 것이 불가능하다.
- 언제 발생한 메시지인지, 어느 파일의 몇 번째 줄에서 발생했는지 같은 부가 정보를 매번 직접 문자열로 조합해야 한다.

`logging` 모듈은 이런 문제를 해결하기 위한 표준 라이브러리로, 메시지의 중요도에 따라 **로그 레벨**을 지정하고, 실행 시점에 어느 레벨 이상만 출력할지 설정할 수 있게 해준다.

**로그 레벨 — DEBUG/INFO/WARNING/ERROR/CRITICAL**

```python
import logging

logging.basicConfig(level=logging.WARNING)

logging.debug("디버그: 변수 값 확인용 상세 정보")
logging.info("정보: 정상적인 진행 상황 알림")
logging.warning("경고: 예상치 못했지만 계속 진행 가능한 상황")
logging.error("오류: 특정 기능이 실패한 상황")
logging.critical("치명적: 프로그램 전체가 멈출 수 있는 상황")
```

```text
(base) C:\Users\guest\project> python logging_levels.py
WARNING:root:경고: 예상치 못했지만 계속 진행 가능한 상황
ERROR:root:오류: 특정 기능이 실패한 상황
CRITICAL:root:치명적: 프로그램 전체가 멈출 수 있는 상황
```

- `logging` 모듈은 메시지의 중요도를 낮은 순서부터 `DEBUG` < `INFO` < `WARNING` < `ERROR` < `CRITICAL` 다섯 단계로 구분한다. `DEBUG`는 개발 중에만 필요한 세세한 정보, `INFO`는 정상 동작 기록, `WARNING`은 아직 문제는 아니지만 주의가 필요한 상황, `ERROR`는 특정 작업이 실패한 상황, `CRITICAL`은 프로그램 자체가 멈출 수 있는 심각한 상황을 나타낸다.
- `logging.basicConfig(level=...)`로 지정한 레벨보다 **낮은** 레벨의 메시지는 출력되지 않는다. 위 예제에서는 `level=logging.WARNING`으로 설정했으므로, 그보다 낮은 `debug()`와 `info()` 호출은 출력되지 않고 `warning()` 이상만 출력된 것을 확인할 수 있다.
- 기본 출력 형식은 `레벨이름:로거이름:메시지` 형태이며, 로거 이름을 따로 지정하지 않으면 최상위 로거를 뜻하는 `root`가 표시된다.

**logging.getLogger(__name__) 사용 패턴**

실제 프로젝트에서는 최상위 `logging.debug()`, `logging.info()`를 직접 호출하기보다, 모듈마다 이름이 붙은 로거를 따로 만들어 사용하는 것이 관례다.

```python
# mymodule.py
import logging

logger = logging.getLogger(__name__)

def divide(a, b):
    logger.info("나눗셈 계산 시작: %s / %s", a, b)
    if b == 0:
        logger.error("0으로 나눌 수 없습니다.")
        return None
    result = a / b
    logger.info("계산 결과: %s", result)
    return result
```

```python
# main.py
import logging
import mymodule

logging.basicConfig(level=logging.INFO, format="%(asctime)s %(name)s %(levelname)s %(message)s")

mymodule.divide(10, 2)
mymodule.divide(5, 0)
```

```text
(base) C:\Users\guest\project> python main.py
2026-09-10 14:30:00,123 mymodule INFO 나눗셈 계산 시작: 10 / 2
2026-09-10 14:30:00,124 mymodule INFO 계산 결과: 5.0
2026-09-10 14:30:00,125 mymodule INFO 나눗셈 계산 시작: 5 / 0
2026-09-10 14:30:00,126 mymodule ERROR 0으로 나눌 수 없습니다.
```

- `logging.getLogger(__name__)`은 현재 모듈 이름(`__name__`)을 가진 로거 객체를 반환한다. 같은 이름으로 `getLogger()`를 다시 호출하면 매번 새 로거를 만드는 것이 아니라 이미 만들어진 동일한 로거를 재사용한다.
- 이렇게 모듈별로 로거를 나누면, 로그 메시지에 어느 모듈에서 발생한 메시지인지가 자동으로 함께 기록되어 여러 파일로 구성된 프로그램에서 문제가 발생한 위치를 찾기 쉬워진다.
- `logger.info("계산 결과: %s", result)`처럼 문자열 포매팅을 `%s` 자리표시자와 인자로 넘기는 방식도 흔히 쓰인다. f-string으로 미리 문자열을 만드는 것과 최종 결과는 비슷하지만, 이 방식은 실제로 로그가 출력되지 않는 레벨일 때는 문자열 조합 자체를 건너뛸 수 있어 약간의 성능 이점이 있다.
- 실행 시점에 `basicConfig()`는 프로그램 전체에서 **한 번만** 호출하는 것이 일반적이며, 보통 프로그램의 진입점(위 예제의 `main.py`)에서 설정한다. 각 모듈은 설정을 직접 하지 않고 `getLogger(__name__)`으로 로거만 얻어 사용하는 것이 권장되는 패턴이다.

**로그 포맷 지정**

```python
import logging

logging.basicConfig(
    level=logging.DEBUG,
    format="[%(levelname)s] %(asctime)s - %(message)s",
    datefmt="%Y-%m-%d %H:%M:%S",
)

logging.debug("포맷을 지정한 디버그 메시지")
logging.warning("포맷을 지정한 경고 메시지")
```

```text
(base) C:\Users\guest\project> python logging_format.py
[DEBUG] 2026-09-10 14:30:00 - 포맷을 지정한 디버그 메시지
[WARNING] 2026-09-10 14:30:00 - 포맷을 지정한 경고 메시지
```

- `format` 인자에는 `%(levelname)s`(레벨 이름), `%(asctime)s`(발생 시각), `%(name)s`(로거 이름), `%(message)s`(메시지 본문) 같은 자리표시자를 조합해 원하는 형태의 로그 출력을 만들 수 있다.
- `datefmt`는 `%(asctime)s` 부분의 날짜/시간 표기 형식을 `strftime()`과 같은 방식으로 지정한다. 지정하지 않으면 밀리초까지 포함된 기본 형식이 쓰인다.
- `basicConfig()`는 프로그램 안에서 실질적으로 처음 호출될 때만 효과가 있다. 이미 로깅 설정이 구성된 뒤에 다시 호출해도 기본적으로는 무시되므로, 설정은 프로그램 시작 시점에 한 번만 하는 것이 좋다.

**정리**: `logging` 모듈은 `print()`와 달리 메시지의 중요도를 `DEBUG`부터 `CRITICAL`까지 다섯 단계로 구분하고, `basicConfig(level=...)`로 지정한 레벨 이상만 골라서 출력할 수 있게 해주며, 모듈마다 `logging.getLogger(__name__)`으로 이름이 구분된 로거를 만들어 사용하고 `format`/`datefmt`로 시각·로거 이름·레벨을 포함한 출력 형식을 지정하면, 개발 중에는 상세한 로그를, 운영 중에는 경고와 오류 위주의 로그만 남기는 유연한 전환이 가능해져 프로그램이 커질수록 `print()` 디버깅보다 훨씬 관리하기 쉬운 방식이 된다.

## threading 모듈 기초

**스레드가 필요한 상황**

지금까지 작성한 프로그램은 한 번에 한 가지 작업만 순서대로 처리했다. 그런데 파일을 여러 개 내려받거나, 여러 대의 서버에 요청을 보내고 응답을 기다리는 것처럼 **대기 시간이 긴 작업을 여러 개 동시에 처리하고 싶은 상황**이 자주 생긴다. 이런 작업을 하나씩 순서대로 처리하면 전체 대기 시간이 작업 개수만큼 그대로 늘어난다. `threading` 모듈은 하나의 프로그램 안에서 여러 개의 **스레드(thread)**를 만들어, 이런 작업들을 동시에 진행되는 것처럼 처리할 수 있게 해준다.

**threading.Thread 생성/시작/join()**

```python
import threading
import time

def worker(name, delay):
    print(f"{name} 시작")
    time.sleep(delay)
    print(f"{name} 종료")

t1 = threading.Thread(target=worker, args=("작업A", 1))
t2 = threading.Thread(target=worker, args=("작업B", 1))

start = time.time()

t1.start()
t2.start()

t1.join()
t2.join()

print(f"전체 소요 시간: {time.time() - start:.1f}초")
```

```text
(base) C:\Users\guest\project> python thread_basic.py
작업A 시작
작업B 시작
작업A 종료
작업B 종료
전체 소요 시간: 1.0초
```

- `threading.Thread(target=함수, args=(인자들,))`은 지정한 함수를 별도의 스레드에서 실행할 준비를 하는 `Thread` 객체를 만든다. 이 시점에는 아직 스레드가 실제로 실행되지 않는다.
- `start()`를 호출해야 실제로 스레드가 시작되며, `target`으로 지정한 함수가 별도의 흐름으로 실행되기 시작한다. 두 스레드를 연달아 `start()`하면 `작업A 시작`과 `작업B 시작`이 (아주 근접한 시점에) 순서대로 출력되지만, 그 뒤로는 두 작업이 동시에 `time.sleep(1)`로 대기하게 된다.
- `join()`은 해당 스레드가 끝날 때까지 현재 흐름(메인 스레드)을 기다리게 한다. `t1.join()`과 `t2.join()`을 호출하지 않으면 메인 스레드는 두 작업이 끝나기 전에 먼저 "전체 소요 시간" 줄을 출력해버릴 수 있다.
- 위 예제에서 `작업A 종료`와 `작업B 종료`가 출력되는 순서는 두 스레드가 거의 동시에 `sleep(1)`을 마치기 때문에 실행할 때마다 달라질 수 있다. 반면 두 작업을 순서대로(동시성 없이) 처리했다면 총 2초가 걸렸겠지만, 동시에 실행했기 때문에 전체 소요 시간은 약 1초로 단축된다.
- 각 작업이 1초씩 걸리는 두 작업을 순서대로 처리하면 총 2초가 걸리지만, 스레드로 동시에 처리하면 두 작업의 대기 시간이 겹치므로 전체 소요 시간이 약 1초로 줄어든다는 점이 스레드를 쓰는 핵심 이유다.

**GIL(Global Interpreter Lock)에 대한 이해**

CPython(가장 널리 쓰이는 파이썬 구현체)에는 **GIL(Global Interpreter Lock)**이라는 장치가 있다. GIL은 한 번에 하나의 스레드만 파이썬 바이트코드를 실행할 수 있도록 제한하는 잠금 장치다.

- 이 때문에 순수하게 CPU 연산만 반복하는 **CPU-bound 작업**(예: 복잡한 수학 계산을 여러 스레드로 나눠 처리)은 스레드를 여러 개 만들어도 여러 코어를 동시에 활용하지 못해 기대한 만큼 빨라지지 않는 한계가 있다.
- 반면 위 예제의 `time.sleep()`이나 파일/네트워크 입출력처럼, CPU 계산보다는 **응답을 기다리는 시간이 대부분인 I/O-bound 작업**에서는 한 스레드가 대기하는 동안 GIL이 다른 스레드에게 실행 기회를 넘겨줄 수 있으므로, 스레드를 통한 동시 실행이 실질적인 성능 향상으로 이어진다.
- 정리하면 `threading`은 I/O-bound 작업(네트워크 요청, 파일 입출력 등)을 동시에 처리할 때 효과적이고, CPU-bound 작업을 여러 코어로 병렬 처리하고 싶다면 `threading` 대신 별도의 프로세스를 여러 개 띄우는 `multiprocessing` 모듈을 검토하는 것이 더 적합하다.

**threading.Lock으로 공유 자원 보호하기**

여러 스레드가 같은 변수를 동시에 수정하면 예상하지 못한 결과가 나올 수 있다. `Lock`은 한 번에 하나의 스레드만 특정 코드 구간에 접근하도록 막아 이런 문제를 방지한다.

```python
import threading

counter = 0
lock = threading.Lock()

def increment():
    global counter
    for _ in range(100_000):
        with lock:
            counter += 1

threads = [threading.Thread(target=increment) for _ in range(4)]

for t in threads:
    t.start()
for t in threads:
    t.join()

print("최종 counter 값:", counter)
```

```text
(base) C:\Users\guest\project> python thread_lock.py
최종 counter 값: 400000
```

- `counter += 1`은 겉보기엔 한 줄이지만 내부적으로는 "값을 읽고, 1을 더하고, 다시 저장하는" 여러 단계로 이루어져 있다. 여러 스레드가 `lock` 없이 동시에 이 과정을 수행하면, 한 스레드가 읽은 값을 다른 스레드가 미처 반영되기 전의 값으로 덮어써서 일부 증가 연산이 손실될 수 있다(이런 상황을 **경쟁 상태(race condition)**라고 부른다).
- `with lock:` 구문은 이 블록에 진입할 때 잠금을 획득하고 빠져나갈 때 자동으로 해제한다. 잠금이 걸려 있는 동안 다른 스레드가 같은 `lock`으로 진입하려 하면, 먼저 들어간 스레드가 빠져나갈 때까지 대기한다.
- 이렇게 `Lock`으로 보호하면 스레드 4개가 각각 100,000번씩 증가시키더라도 경쟁 상태 없이 정확히 `400000`이라는 값이 보장된다. `Lock`을 사용하지 않고 같은 코드를 실행하면 환경에 따라 `400000`보다 작은 값이 나올 수 있다.

**정리**: `threading` 모듈은 `Thread(target=함수, args=...)`로 스레드를 만들고 `start()`로 시작한 뒤 `join()`으로 종료를 기다리는 방식으로 여러 작업을 동시에 진행시킬 수 있게 해주며, CPython의 GIL 때문에 CPU 연산 위주의 작업에서는 큰 효과를 보기 어렵지만 네트워크 요청이나 파일 입출력처럼 대기 시간이 많은 I/O-bound 작업에서는 전체 소요 시간을 크게 줄여주고, 여러 스레드가 같은 변수를 동시에 수정할 때는 `Lock`으로 해당 구간을 감싸 경쟁 상태를 방지해야 한다.

## urllib을 이용한 기본 인터넷 접근

**urllib.request.urlopen()으로 URL 열기**

파이썬 표준 라이브러리는 별도의 외부 패키지 설치 없이도 인터넷 자원에 접근할 수 있는 `urllib` 모듈을 제공한다. 실무에서는 더 편리한 외부 라이브러리를 쓰는 경우도 많지만, 표준 라이브러리만으로도 기본적인 요청/응답 처리가 가능하다는 것을 알아둘 필요가 있다.

```python
import urllib.request

url = "https://docs.python.org/3/"

with urllib.request.urlopen(url) as response:
    print("상태 코드:", response.status)
    print("Content-Type 헤더:", response.headers.get("Content-Type"))
    html = response.read()
    print("응답 본문 길이(바이트):", len(html))
```

```text
(base) C:\Users\guest\project> python urllib_open.py
상태 코드: 200
Content-Type 헤더: text/html
응답 본문 길이(바이트): 15000
```

- `urllib.request.urlopen(url)`은 지정한 URL에 요청을 보내고, 그 응답을 담은 객체를 반환한다. `with` 문과 함께 쓰면 응답을 다 처리한 뒤 연결을 자동으로 닫아준다.
- `response.status`는 HTTP 상태 코드를 정수로 담고 있으며, `200`은 요청이 정상적으로 처리되었음을 뜻한다. `response.headers`는 서버가 함께 보낸 응답 헤더들을 담고 있어, `.get("Content-Type")`처럼 원하는 헤더 값을 조회할 수 있다.
- `response.read()`는 응답 본문을 **바이트(bytes)** 형태로 반환한다. 텍스트로 다루려면 `html.decode("utf-8")`처럼 문자 인코딩을 지정해 디코딩해야 한다.
- 실제 응답 본문의 정확한 길이나 내용은 서버 상태와 시점에 따라 달라지므로, 위 출력의 구체적인 숫자는 실행할 때마다 다를 수 있는 예시 값이며 중요한 것은 상태 코드와 헤더를 코드로 확인할 수 있다는 점이다.

**간단한 예외 처리 — urllib.error.URLError**

네트워크가 연결되지 않았거나, 존재하지 않는 주소에 접근하거나, 서버가 응답하지 않는 등 여러 이유로 요청이 실패할 수 있다. 이런 상황은 `urllib.error.URLError`(또는 그 하위 클래스인 `HTTPError`)로 처리한다.

```python
import urllib.request
import urllib.error

def fetch_status(url):
    try:
        with urllib.request.urlopen(url, timeout=5) as response:
            return response.status
    except urllib.error.HTTPError as e:
        print(f"HTTP 오류: 상태 코드 {e.code}")
        return None
    except urllib.error.URLError as e:
        print(f"연결 오류: {e.reason}")
        return None

print(fetch_status("https://docs.python.org/3/"))
print(fetch_status("https://this-domain-should-not-exist.invalid/"))
```

```text
(base) C:\Users\guest\project> python urllib_error.py
200
연결 오류: [Errno 11001] getaddrinfo failed
None
```

- `urllib.error.HTTPError`는 서버가 응답은 했지만 `404`(찾을 수 없음), `500`(서버 오류)처럼 오류를 나타내는 상태 코드를 반환했을 때 발생하며, `e.code`로 그 상태 코드를 확인할 수 있다. `HTTPError`는 `URLError`를 상속하므로, 더 구체적인 `HTTPError`를 먼저 `except`로 잡고 그 다음에 일반적인 `URLError`를 잡는 순서가 안전하다.
- `urllib.error.URLError`는 존재하지 않는 도메인이나 네트워크 연결 자체의 문제처럼, 서버로부터 아예 응답을 받지 못했을 때 발생한다. `e.reason`에 구체적인 실패 사유가 담긴다. 이 오류 메시지의 정확한 문구는 운영체제와 네트워크 환경에 따라 달라질 수 있다.
- `urlopen(url, timeout=5)`처럼 `timeout`(초 단위)을 지정하면, 서버가 응답 없이 너무 오래 걸릴 때 무한정 대기하지 않고 일정 시간 뒤에 `URLError`를 발생시키며 실패로 처리할 수 있다. 네트워크를 다루는 코드에서는 `timeout`을 지정하는 것이 안전한 습관이다.

**정리**: `urllib.request.urlopen()`은 외부 패키지 없이도 URL에 요청을 보내고 `status`(상태 코드)와 `headers`(응답 헤더), `read()`(본문)를 확인할 수 있게 해주며, 네트워크 요청은 실패할 가능성이 항상 존재하므로 `urllib.error.HTTPError`(서버가 오류 상태 코드로 응답한 경우)와 `URLError`(연결 자체가 실패한 경우)를 구분해 처리하고 `timeout`을 지정해두는 것이, 인터넷 자원에 접근하는 코드를 안전하게 작성하는 기본기다.

## unittest와 doctest로 테스트 작성하기

**왜 테스트가 필요한가**

지금까지는 함수를 작성한 뒤 직접 몇 가지 값을 넣어 실행해보고 결과를 눈으로 확인하는 방식으로 코드를 검증해왔다. 이 방식은 간단한 스크립트에는 충분하지만, 함수 개수가 늘어나고 코드를 수정할 일이 잦아지면 문제가 생긴다. 어떤 함수를 고쳤을 때 그 함수와 관련된 기존 동작이 여전히 올바른지 매번 손으로 다시 확인하기는 번거롭고 실수하기도 쉽다. **테스트 코드**는 "이 입력에는 이 결과가 나와야 한다"는 검증 절차를 코드로 남겨두어, 언제든 다시 실행해 기존 동작이 깨지지 않았는지 자동으로 확인할 수 있게 해준다.

**unittest.TestCase와 assert 메서드**

```python
# calculator.py
def add(a, b):
    return a + b

def divide(a, b):
    if b == 0:
        raise ValueError("0으로 나눌 수 없습니다.")
    return a / b
```

```python
# test_calculator.py
import unittest
from calculator import add, divide

class TestCalculator(unittest.TestCase):
    def test_add(self):
        self.assertEqual(add(2, 3), 5)
        self.assertEqual(add(-1, 1), 0)

    def test_divide(self):
        self.assertEqual(divide(10, 2), 5)
        self.assertAlmostEqual(divide(1, 3), 0.3333, places=4)

    def test_divide_by_zero(self):
        with self.assertRaises(ValueError):
            divide(5, 0)

if __name__ == "__main__":
    unittest.main()
```

```text
(base) C:\Users\guest\project> python test_calculator.py -v
test_add (__main__.TestCalculator.test_add) ... ok
test_divide (__main__.TestCalculator.test_divide) ... ok
test_divide_by_zero (__main__.TestCalculator.test_divide_by_zero) ... ok

----------------------------------------------------------------------
Ran 3 tests in 0.001s

OK
```

- `unittest.TestCase`를 상속한 클래스 안에 `test_`로 시작하는 이름의 메서드를 정의하면, 각 메서드가 하나의 독립된 테스트로 인식된다.
- `self.assertEqual(실제값, 기대값)`은 두 값이 같은지 확인하고, 다르면 테스트를 실패로 처리하면서 두 값의 차이를 알려주는 오류 메시지를 남긴다. `self.assertAlmostEqual(a, b, places=4)`는 PY-15에서 다룬 부동소수점 오차를 감안해, 소수점 아래 4자리까지 반올림했을 때 같으면 통과시킨다.
- `with self.assertRaises(예외클래스):` 블록은 그 안에서 지정한 예외가 실제로 발생하는지 확인한다. 예외가 발생하지 않거나 다른 종류의 예외가 발생하면 테스트가 실패한다.
- `unittest.main()`은 같은 파일 안에 정의된 모든 테스트를 찾아 실행한다. `-v`(verbose) 옵션을 주면 각 테스트 메서드의 이름과 통과 여부를 한 줄씩 보여주고, 마지막 줄에 전체 테스트 개수와 소요 시간, 그리고 모두 통과했다면 `OK`를 출력한다. `-v` 없이 실행하면 테스트 하나마다 `.`(점) 하나만 찍히고 마찬가지로 마지막에 `OK`가 출력된다.

**테스트가 실패하는 경우**

```python
# test_calculator_fail.py
import unittest
from calculator import add

class TestCalculatorFail(unittest.TestCase):
    def test_add_wrong(self):
        self.assertEqual(add(2, 3), 6)  # 일부러 틀린 기대값

if __name__ == "__main__":
    unittest.main()
```

```text
(base) C:\Users\guest\project> python test_calculator_fail.py
F
======================================================================
FAIL: test_add_wrong (__main__.TestCalculatorFail.test_add_wrong)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "test_calculator_fail.py", line 6, in test_add_wrong
    self.assertEqual(add(2, 3), 6)  # 일부러 틀린 기대값
AssertionError: 5 != 6

----------------------------------------------------------------------
Ran 1 test in 0.001s

FAILED (failures=1)
```

- 테스트가 실패하면 결과 요약 부분에 `F`가 표시되고, 어떤 테스트가 어떤 이유로 실패했는지 `AssertionError`와 함께 실제 값(`5`)과 기대했던 값(`6`)을 보여주는 트레이스백이 출력된다.
- 마지막 줄은 통과했을 때의 `OK` 대신 `FAILED (failures=1)`처럼 실패한 테스트 개수를 보여준다. 코드를 실행하다가 예상치 못한 예외가 발생해 테스트 자체가 중단된 경우에는 `failures` 대신 `errors`로 표시되어, "검증 결과가 틀린 것"과 "테스트 실행 중 오류가 난 것"을 구분해서 볼 수 있다.
- 여러 개의 테스트 메서드를 실행하면 통과한 테스트는 `.`, 실패한 테스트는 `F`, 오류가 난 테스트는 `E`로 요약 줄에 하나씩 표시되며, 그 아래에 실패/오류가 난 테스트들의 자세한 내용이 순서대로 나열된다.

**doctest로 docstring 안의 예제 실행하기**

`doctest`는 함수의 독스트링(docstring) 안에 `>>>`로 시작하는 대화형 인터프리터 예제를 그대로 적어두면, 그 예제를 실제 테스트로 실행해주는 모듈이다.

```python
# math_utils.py
def square(n):
    """
    숫자를 제곱한 값을 반환한다.

    >>> square(3)
    9
    >>> square(-2)
    4
    >>> square(0)
    0
    """
    return n * n

if __name__ == "__main__":
    import doctest
    doctest.testmod()
```

```text
(base) C:\Users\guest\project> python -m doctest math_utils.py
(base) C:\Users\guest\project> python -m doctest math_utils.py -v
Trying:
    square(3)
Expecting:
    9
ok
Trying:
    square(-2)
Expecting:
    4
ok
Trying:
    square(0)
Expecting:
    0
ok
1 items had no tests:
    math_utils
1 items passed all tests:
   3 tests in math_utils.square
3 tests in 2 items.
3 passed and 0 failed.
Test is complete.
```

- 독스트링 안에서 `>>> square(3)` 다음 줄에 적힌 `9`는 "이 코드를 REPL에 입력하면 이 출력이 나와야 한다"는 기대값으로 읽힌다. `python -m doctest 파일명.py`로 실행하면 독스트링 안의 모든 `>>>` 예제를 하나씩 실행해보고 실제 출력과 기대값을 비교한다.
- 모든 예제가 통과하면 `python -m doctest math_utils.py`처럼 옵션 없이 실행했을 때 **아무 것도 출력하지 않는다**(성공했다는 사실 자체가 "조용한 상태"로 표현된다). 반대로 `-v`(verbose) 옵션을 주면 각 예제를 실행하는 과정과 결과를 모두 자세히 보여준다.
- 코드 안에서 `doctest.testmod()`를 직접 호출하는 방법도 있는데, 이렇게 해두면 `python math_utils.py`처럼 스크립트를 직접 실행했을 때도 독스트링 예제가 함께 검증된다.
- `doctest`의 장점은 예제 코드가 곧 문서이자 테스트라는 점이다. 함수 사용법을 설명하는 예제를 독스트링에 적어두면, 그 예제가 실제로 올바르게 동작하는지 `doctest`가 계속 확인해주므로 코드가 바뀌어 문서의 예제가 더 이상 맞지 않게 되는 상황을 방지할 수 있다.
- 다만 `doctest`는 출력 형식이 한 글자라도 다르면 실패로 처리할 만큼 엄격하므로, 복잡한 로직을 촘촘히 검증하기보다는 함수의 기본적인 사용법을 예제 겸 간단한 테스트로 남기는 용도에 더 적합하다. 여러 조건 분기나 예외 상황까지 꼼꼼히 검증하려면 `unittest`가 더 적합하다.

**정리**: `unittest.TestCase`를 상속해 `test_`로 시작하는 메서드 안에 `assertEqual`, `assertAlmostEqual`, `assertRaises` 같은 assert 메서드로 기대 동작을 명시해두면 `unittest.main()`으로 한 번에 실행해 통과 시 `OK`, 실패 시 `AssertionError`와 함께 실패 개수를 보여주는 트레이스백을 얻을 수 있고, `doctest`는 독스트링 안의 `>>>` 예제 자체를 테스트로 실행해 `python -m doctest 파일명.py`로 검증할 수 있어, 두 도구를 상황에 맞게 섞어 쓰면 코드를 수정할 때마다 기존 동작이 깨지지 않았는지 자동으로 확인하는 안전망을 갖출 수 있다.

## 실습 예제 (EX1~EX4)

**EX1) logging으로 함수 호출 기록 남기기**
- 리스트에서 최댓값을 찾는 함수에 로그를 추가해, 호출될 때마다 입력값과 결과가 기록되도록 한다.

```python
# ex01_logging_max.py
import logging

logging.basicConfig(level=logging.INFO, format="%(levelname)s: %(message)s")
logger = logging.getLogger(__name__)

def find_max(numbers):
    logger.info("find_max 호출, 입력: %s", numbers)
    if not numbers:
        logger.warning("빈 리스트가 입력되었습니다.")
        return None
    result = max(numbers)
    logger.info("find_max 결과: %s", result)
    return result

print(find_max([3, 7, 2, 9, 4]))
print(find_max([]))
```

```text
(base) C:\Users\guest\project> python ex01_logging_max.py
INFO: find_max 호출, 입력: [3, 7, 2, 9, 4]
INFO: find_max 결과: 9
9
INFO: find_max 호출, 입력: []
WARNING: 빈 리스트가 입력되었습니다.
None
```

**EX2) threading으로 여러 파일 크기 동시에 확인하기**
- 여러 파일을 만들어두고, 각 파일의 크기를 확인하는 작업을 스레드로 동시에 실행한 뒤 결과를 딕셔너리에 모은다.

```python
# ex02_thread_filesize.py
import threading
import os

os.makedirs("files", exist_ok=True)
for name, content in [("a.txt", "hello"), ("b.txt", "hello world"), ("c.txt", "hi")]:
    with open(os.path.join("files", name), "w", encoding="utf-8") as f:
        f.write(content)

results = {}
lock = threading.Lock()

def check_size(name):
    size = os.path.getsize(os.path.join("files", name))
    with lock:
        results[name] = size

names = ["a.txt", "b.txt", "c.txt"]
threads = [threading.Thread(target=check_size, args=(name,)) for name in names]

for t in threads:
    t.start()
for t in threads:
    t.join()

for name in sorted(results):
    print(f"{name}: {results[name]}바이트")
```

```text
(base) C:\Users\guest\project> python ex02_thread_filesize.py
a.txt: 5바이트
b.txt: 11바이트
c.txt: 2바이트
```

**EX3) urllib로 상태 코드 안전하게 확인하기**
- 정상 URL과 존재하지 않는 URL을 각각 요청해, 예외 처리를 포함한 함수로 상태를 확인한다.

```python
# ex03_urllib_status.py
import urllib.request
import urllib.error

def check_url(url):
    try:
        with urllib.request.urlopen(url, timeout=5) as response:
            return f"정상 응답, 상태 코드: {response.status}"
    except urllib.error.HTTPError as e:
        return f"HTTP 오류, 상태 코드: {e.code}"
    except urllib.error.URLError as e:
        return f"연결 실패: {e.reason}"

urls = [
    "https://docs.python.org/3/",
    "https://this-domain-should-not-exist.invalid/",
]

for url in urls:
    print(url, "->", check_url(url))
```

```text
(base) C:\Users\guest\project> python ex03_urllib_status.py
https://docs.python.org/3/ -> 정상 응답, 상태 코드: 200
https://this-domain-should-not-exist.invalid/ -> 연결 실패: [Errno 11001] getaddrinfo failed
```

**EX4) unittest로 문자열 유틸리티 함수 검증하기**
- 문자열이 회문(palindrome)인지 확인하는 함수를 작성하고, 정상 케이스와 예외 케이스를 unittest로 검증한다.

```python
# ex04_test_palindrome.py
import unittest

def is_palindrome(text):
    normalized = text.replace(" ", "").lower()
    return normalized == normalized[::-1]

class TestPalindrome(unittest.TestCase):
    def test_simple_true(self):
        self.assertTrue(is_palindrome("level"))

    def test_simple_false(self):
        self.assertFalse(is_palindrome("python"))

    def test_with_spaces_and_case(self):
        self.assertTrue(is_palindrome("Was it a car or a cat I saw"))

    def test_empty_string(self):
        self.assertTrue(is_palindrome(""))

if __name__ == "__main__":
    unittest.main()
```

```text
(base) C:\Users\guest\project> python ex04_test_palindrome.py -v
test_empty_string (__main__.TestPalindrome.test_empty_string) ... ok
test_simple_false (__main__.TestPalindrome.test_simple_false) ... ok
test_simple_true (__main__.TestPalindrome.test_simple_true) ... ok
test_with_spaces_and_case (__main__.TestPalindrome.test_with_spaces_and_case) ... ok

----------------------------------------------------------------------
Ran 4 tests in 0.001s

OK
```

**정리**: EX1~EX4는 함수 호출 과정에 `logging`으로 기록을 남기는 방법, `threading`으로 여러 파일 작업을 동시에 처리하고 `Lock`으로 결과를 안전하게 모으는 방법, `urllib`로 URL 상태를 예외 처리와 함께 안전하게 확인하는 방법, 그리고 `unittest`로 문자열 유틸리티 함수의 여러 케이스를 자동으로 검증하는 방법까지, 이 문서에서 다룬 로깅·동시성·네트워크·테스트 주제를 실무에 가까운 형태로 조합해보는 예제다.

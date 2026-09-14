# Shell Script 종료 상태 코드 심화 (Exit Status Advanced)

기본적인 `$?`, `exit 숫자` 사용법은 `08-syntax-master.md`의 "명령어 종료 상태 코드 (exit, $?)" 문서를 먼저 참고한다. 이 문서는 그 위에서 **어떤 숫자가 무엇을 의미하는지**, **파이프라인에서는 종료 코드가 어떻게 처리되는지**를 다룬다.

## 예약된 종료 코드 (Reserved Exit Codes)

종료 코드는 0~255 범위지만, 그중 일부는 관례적으로 Bash와 리눅스 시스템이 이미 의미를 정해놓고 사용한다.

| 코드 | 의미 |
|---|---|
| 0 | 성공 |
| 1 | 일반적인 오류(General error) |
| 2 | 셸 문법 오류, 명령어 잘못된 사용(misuse of shell builtin) |
| 126 | 파일은 찾았지만 실행 권한이 없거나 실행할 수 없음(Permission problem / not executable) |
| 127 | 명령어(파일) 자체를 찾을 수 없음(command not found) |
| 128 + N | 시그널 번호 N에 의해 프로세스가 종료됨 |
| 130 | 128 + 2, 즉 `Ctrl + C`(SIGINT)로 종료됨 |
| 137 | 128 + 9, 즉 `kill -9`(SIGKILL)로 강제 종료됨 |
| 255 | 종료 코드가 0~255 범위를 벗어난 값을 지정했을 때(예: `exit -1`, `exit 300`) Bash가 대신 사용하는 값 |

```bash
[root@Server-A ~]# nonexistent_cmd
bash: nonexistent_cmd: 명령을 찾을 수 없습니다

[root@Server-A ~]# echo $?
127


[root@Server-A ~]# touch /readonly_test && chmod 000 /readonly_test && /readonly_test
bash: /readonly_test: 허가 거부됨

[root@Server-A ~]# echo $?
126


[root@Server-A ~]# vi ./script/exit_range_test.sh
#!/bin/bash
exit 300

:wq

[root@Server-A ~]# chmod +x ./script/exit_range_test.sh

[root@Server-A ~]# ./script/exit_range_test.sh

[root@Server-A ~]# echo $?		# 300 % 256 처럼 8비트로 잘려서 44가 됨
44
```

- 종료 코드는 실제로 8비트(0~255) 정수 하나만 저장할 수 있어서, 256 이상의 값을 `exit`에 넘기면 256으로 나눈 나머지가 실제 종료 코드가 된다.

**정리**: 0(성공)과 1(일반 오류) 외에도 2(문법 오류), 126(권한 없음), 127(명령 없음), 128+N(시그널 종료), 130/137(대표적인 시그널 예)처럼 리눅스/Bash가 관례적으로 정해둔 예약된 종료 코드가 있으며, 이를 알아두면 `$?` 값만 보고도 실패 원인을 짐작할 수 있다.

## 파이프라인과 종료 상태 ($PIPESTATUS · pipefail)

- `명령1 | 명령2` 처럼 파이프로 여러 명령을 연결하면, 기본적으로 **`$?`에는 파이프라인의 마지막 명령의 종료 코드만 남는다.** 앞쪽 명령이 실패해도 마지막 명령이 성공하면 전체 파이프라인은 성공(0)으로 취급된다.

```bash
[root@Server-A ~]# cat /no_such_file | wc -l
cat: /no_such_file: 그런 파일이나 디렉터리가 없습니다
0

[root@Server-A ~]# echo $?		# wc -l 은 성공했으므로 0이 나온다 (cat 실패는 감춰짐)
0
```

- 파이프라인 각 단계의 종료 코드를 모두 확인하고 싶을 때는 배열 변수 **`$PIPESTATUS`**를 사용한다. 파이프라인 실행 직후에만 유효하다.

```bash
[root@Server-A ~]# cat /no_such_file | wc -l
cat: /no_such_file: 그런 파일이나 디렉터리가 없습니다
0

[root@Server-A ~]# echo "${PIPESTATUS[@]}"
1 0				# 앞 : cat의 종료코드(1), 뒤 : wc -l의 종료코드(0)
```

- `set -o pipefail` 옵션을 켜면, 파이프라인 안의 명령 중 **하나라도 실패하면 파이프라인 전체의 `$?`가 실패로 바뀐다.**

```bash
[root@Server-A ~]# set -o pipefail

[root@Server-A ~]# cat /no_such_file | wc -l
cat: /no_such_file: 그런 파일이나 디렉터리가 없습니다
0

[root@Server-A ~]# echo $?		# pipefail 덕분에 cat의 실패가 반영됨
1
```

- `pipefail`은 Bash 전용 옵션이라 `sh`에서는 사용할 수 없다. `sh` 호환이 필요한 스크립트라면 `$PIPESTATUS` 대신 파이프를 쓰지 않고 중간 결과를 임시 파일에 저장하거나 명령을 나눠서 각각 `$?`를 확인하는 방식으로 우회한다.

**정리**: 파이프라인의 `$?`는 기본적으로 마지막 명령 기준이며, 중간 단계의 실패까지 확인하려면 `$PIPESTATUS` 배열을 읽거나 `set -o pipefail`로 파이프라인 전체 판정 기준을 바꿔야 한다.

## && · || 를 이용한 단락 평가 (Short-Circuit)

- `&&`와 `||`는 앞 명령의 **종료 코드**만 보고 뒤 명령의 실행 여부를 결정하는 단락 평가(short-circuit) 연산자다.
- `명령1 && 명령2` : 명령1이 성공(0)해야만 명령2를 실행한다.
- `명령1 || 명령2` : 명령1이 실패(0이 아님)해야만 명령2를 실행한다.

```bash
[root@Server-A ~]# mkdir /backup_test && echo "생성 성공"
생성 성공

[root@Server-A ~]# mkdir /backup_test && echo "생성 성공"	# 이미 있는 디렉터리라 mkdir 실패
mkdir: '/backup_test' 디렉터리를 만들 수 없습니다: 파일이 있습니다
					# && 뒤는 실행되지 않음


[root@Server-A ~]# [ -f /etc/nonexist.conf ] || echo "설정 파일이 없어 기본값을 사용합니다."
설정 파일이 없어 기본값을 사용합니다.
```

- 여러 개를 이어서 "성공하면 계속 진행, 실패하면 그 자리에서 멈추고 대체 명령 실행"하는 패턴으로 자주 쓰인다.

```bash
[root@Server-A ~]# cd /var/log && tar czf /backup/log_$(date +%F).tar.gz . && echo "백업 완료" || echo "백업 실패"
백업 완료
```

**정리**: `&&`/`||`는 직전 명령의 종료 코드를 기준으로 다음 명령 실행 여부를 결정하는 단락 평가 연산자이며, `if`문 없이도 "성공 시 A, 실패 시 B"를 한 줄로 표현할 수 있는 실무에서 매우 자주 쓰이는 패턴이다.

## exit / return 과 함수·서브쉘의 종료 상태

- 스크립트 최상위에서 `exit 숫자`를 쓰면 **스크립트(프로세스) 전체**를 그 코드로 종료한다.
- 함수 안에서 `exit`를 쓰면 함수만 빠져나가는 것이 아니라, 그 함수를 호출한 **스크립트 자체가 그대로 종료**된다는 점에 주의한다. 함수만 빠져나가고 싶다면 `exit`가 아니라 `return`을 사용해야 한다(자세한 내용은 `07-functions.md`의 "return과 함수의 결과값" 참고).

```bash
[root@Server-A ~]# vi ./script/exit_in_func.sh
#!/bin/bash

check () {
    exit 5		# return이 아니라 exit이므로 스크립트 전체가 종료됨
}

check
echo "이 줄은 절대 출력되지 않는다."

:wq


[root@Server-A ~]# chmod +x ./script/exit_in_func.sh

[root@Server-A ~]# ./script/exit_in_func.sh

[root@Server-A ~]# echo $?
5
```

- `( 명령들 )`처럼 소괄호로 묶은 **서브쉘** 안에서 `exit`를 쓰면, 그 서브쉘만 종료되고 바깥 스크립트는 계속 실행된다.

```bash
[root@Server-A ~]# ( echo "서브쉘 시작"; exit 9; echo "여기는 실행 안 됨" )
서브쉘 시작

[root@Server-A ~]# echo $?
9

[root@Server-A ~]# echo "바깥 쉘은 계속 실행됨"
바깥 쉘은 계속 실행됨
```

**정리**: `exit`는 스크립트(프로세스) 전체를 끝내고, 함수 안에서 종료 코드만 알리고 싶을 때는 `return`을 써야 한다. `( )` 서브쉘 안의 `exit`는 그 서브쉘만 종료시키므로 영향 범위가 다르다는 점을 구분해서 사용해야 한다.

## 시그널과 128+N 종료 코드 심화

- 프로세스가 시그널을 받아 종료되면, 쉘은 그 종료 코드를 **128 + 시그널 번호**로 기록한다. 자주 마주치는 시그널을 정리하면 다음과 같다.

| 시그널 | 번호(N) | 종료 코드(128+N) | 의미 |
|---|---|---|---|
| SIGHUP | 1 | 129 | 터미널 연결 종료(hangup) |
| SIGINT | 2 | 130 | `Ctrl + C` 인터럽트 |
| SIGQUIT | 3 | 131 | `Ctrl + \` (코어 덤프 포함 종료) |
| SIGKILL | 9 | 137 | `kill -9`, 강제 종료(트랩 불가) |
| SIGTERM | 15 | 143 | `kill` 기본 시그널(정상 종료 요청) |
| SIGSEGV | 11 | 139 | 세그멘테이션 오류(잘못된 메모리 접근) |

```bash
[root@Server-A ~]# sleep 100 &
[1] 20481

[root@Server-A ~]# kill -TERM %1
[root@Server-A ~]# wait %1
[1]+  종료됨                  sleep 100

[root@Server-A ~]# echo $?
143					# 128 + 15(SIGTERM)


[root@Server-A ~]# sleep 100 &
[1] 20502
[root@Server-A ~]# kill -9 %1
[root@Server-A ~]# wait %1
[1]+  죽었음(신호)               sleep 100

[root@Server-A ~]# echo $?
137					# 128 + 9(SIGKILL)
```

- 시그널 번호는 시스템마다 조금씩 다를 수 있어서(대표적으로 몇몇 아키텍처의 실시간 시그널) 정확한 번호가 필요하면 `kill -l`로 현재 시스템의 목록을 확인하는 것이 안전하다.

```bash
[root@Server-A ~]# kill -l | head -5
 1) SIGHUP	 2) SIGINT	 3) SIGQUIT	 4) SIGILL	 5) SIGTRAP
```

- `SIGKILL`(9)은 프로세스에 절대 가로챌(trap) 수 없는 시그널이라, `kill -9`로 종료된 프로세스는 정리(cleanup) 코드를 실행할 기회조차 없이 즉시 죽는다. 반면 `SIGTERM`(15)은 `trap`으로 가로채서 정리 작업을 먼저 수행한 뒤 종료하도록 만들 수 있다.

**정리**: 시그널에 의한 종료 코드는 `128 + 시그널 번호`로 계산되며, 대표적으로 SIGINT(130), SIGTERM(143), SIGKILL(137)을 알아두면 `$?` 값만 보고도 프로세스가 어떤 시그널로 죽었는지 바로 짐작할 수 있다. `kill -9`(SIGKILL)는 트랩으로 가로챌 수 없다는 점이 `kill`(SIGTERM)과의 핵심 차이다.

## set -e (errexit) 와 종료 상태

- `set -e`(또는 `set -o errexit`)를 켜면, 스크립트 안의 어떤 명령이든 **0이 아닌 종료 코드**를 반환하는 순간 스크립트 전체가 그 자리에서 즉시 종료된다. 매번 `if` 문으로 실패를 검사하지 않아도 되게 해주는 옵션이다.

```bash
[root@Server-A ~]# vi ./script/errexit_test.sh
#!/bin/bash
set -e

echo "1단계 시작"
mkdir /already_exists_dir		# 이미 있는 디렉터리라 실패
echo "2단계 : 이 줄은 실행되지 않는다"

:wq


[root@Server-A ~]# mkdir /already_exists_dir
[root@Server-A ~]# chmod +x ./script/errexit_test.sh

[root@Server-A ~]# ./script/errexit_test.sh
1단계 시작
mkdir: '/already_exists_dir' 디렉터리를 만들 수 없습니다: 파일이 있습니다

[root@Server-A ~]# echo $?
1					# mkdir의 실패 코드가 그대로 스크립트의 종료 코드가 됨
```

- `set -e`에는 예외가 있다. `&&`, `||`의 왼쪽/오른쪽에 있는 명령, `if`/`while`의 조건식으로 쓰이는 명령이 실패하는 것은 "의도된 실패 검사"로 간주되어 `errexit`가 스크립트를 종료시키지 않는다.

```bash
[root@Server-A ~]# vi ./script/errexit_exception.sh
#!/bin/bash
set -e

false || echo "false가 실패해도 종료되지 않음(|| 오른쪽이므로 예외)"

if false; then
    echo "실행 안 됨"
else
    echo "if 조건식의 실패도 예외 처리됨"
fi

echo "여기까지 정상 실행됨"

:wq


[root@Server-A ~]# chmod +x ./script/errexit_exception.sh
[root@Server-A ~]# ./script/errexit_exception.sh
false가 실패해도 종료되지 않음(|| 오른쪽이므로 예외)
if 조건식의 실패도 예외 처리됨
여기까지 정상 실행됨
```

- 파이프라인 안에서는 기본적으로 `set -e`가 **마지막 명령**의 실패만 감지한다(이 문서 위쪽의 `$PIPESTATUS`/`pipefail` 내용과 연결된다). 중간 명령의 실패까지 `errexit`가 잡게 하려면 `set -e`와 `set -o pipefail`을 함께 켜야 한다.

**정리**: `set -e`는 실패한 명령이 나오는 즉시 스크립트를 종료시켜 실패를 방치하지 않게 해주는 안전장치이지만, `&&`/`||`/`if`/`while` 조건식으로 쓰인 명령의 실패는 예외로 취급되며, 파이프라인 중간 실패까지 잡으려면 `pipefail`을 함께 켜야 한다는 점을 기억해야 한다.

## trap ... EXIT 에서 $? 읽기

- `trap '명령' EXIT`는 스크립트가 **정상 종료든 `exit`에 의한 종료든 상관없이 끝나는 시점에** 항상 한 번 실행되는 정리(cleanup) 코드를 등록하는 방법이다.
- `trap`에 등록된 EXIT 핸들러 안에서 `$?`를 읽으면, **스크립트가 어떤 종료 코드로 끝나려던 참인지**를 알 수 있어 성공/실패에 따라 다른 정리 작업을 할 수 있다.

```bash
[root@Server-A ~]# vi ./script/trap_exit.sh
#!/bin/bash

cleanup () {
    local code=$?			# trap 핸들러 안에서 $?는 종료되기 직전의 코드
    if [ "$code" -eq 0 ]; then
        echo "[cleanup] 정상 종료(코드 $code) - 임시파일 삭제"
    else
        echo "[cleanup] 비정상 종료(코드 $code) - 에러 로그 남김"
    fi
    rm -f /tmp/work_$$.tmp
}
trap cleanup EXIT

touch /tmp/work_$$.tmp
echo "작업 중..."

if [ "$1" = "fail" ]; then
    exit 3
fi

echo "작업 정상 완료"

:wq


[root@Server-A ~]# chmod +x ./script/trap_exit.sh

[root@Server-A ~]# ./script/trap_exit.sh
작업 중...
작업 정상 완료
[cleanup] 정상 종료(코드 0) - 임시파일 삭제

[root@Server-A ~]# ./script/trap_exit.sh fail
작업 중...
[cleanup] 비정상 종료(코드 3) - 에러 로그 남김

[root@Server-A ~]# echo $?			# trap이 끝난 뒤에도 원래 종료 코드(3)가 그대로 유지됨
3
```

**정리**: `trap cleanup EXIT`는 스크립트가 어떤 경로로 끝나든(정상 종료, `exit N`, 심지어 에러로 인한 종료까지) 마지막에 반드시 한 번 실행되는 정리 루틴을 만드는 방법이며, 핸들러 안에서 `$?`를 읽으면 종료 코드에 따라 분기해서 다른 마무리 작업을 수행할 수 있다.

## 함수 return vs 스크립트 exit, 중첩 함수 예제

- 함수가 여러 겹으로 중첩되어 서로를 호출하는 상황에서 `return`과 `exit`의 영향 범위 차이가 더 뚜렷하게 드러난다. `return`은 **그 함수 호출 하나만** 끝내고 호출자에게 제어를 돌려주지만, `exit`는 몇 겹으로 중첩되어 있든 **스크립트 전체**를 즉시 끝내버린다.

```bash
[root@Server-A ~]# vi ./script/nested_func.sh
#!/bin/bash

inner () {
    echo "inner 시작"
    return 2			# inner 함수만 종료, 호출자(outer)로 돌아감
}

outer () {
    echo "outer 시작"
    inner
    echo "inner의 리턴 코드 : $?"
    echo "outer 계속 진행"
}

outer
echo "스크립트 마지막 줄까지 도달함"

:wq


[root@Server-A ~]# chmod +x ./script/nested_func.sh
[root@Server-A ~]# ./script/nested_func.sh
outer 시작
inner 시작
inner의 리턴 코드 : 2
outer 계속 진행
스크립트 마지막 줄까지 도달함
```

- 이번엔 `inner`에서 `return` 대신 `exit`를 쓰면, `outer`의 남은 코드는 물론 스크립트의 나머지 줄도 전혀 실행되지 않는다.

```bash
[root@Server-A ~]# vi ./script/nested_func_exit.sh
#!/bin/bash

inner () {
    echo "inner 시작"
    exit 2			# 함수 하나가 아니라 스크립트 전체가 종료됨
}

outer () {
    echo "outer 시작"
    inner
    echo "이 줄은 절대 실행되지 않음"
}

outer
echo "이 줄도 절대 실행되지 않음"

:wq


[root@Server-A ~]# chmod +x ./script/nested_func_exit.sh
[root@Server-A ~]# ./script/nested_func_exit.sh
outer 시작
inner 시작

[root@Server-A ~]# echo $?
2
```

**정리**: `return`은 호출 스택을 한 단계만 거슬러 올라가고, `exit`는 호출 스택이 몇 겹이든 무시하고 프로세스 자체를 끝낸다. 중첩 함수 구조에서 특정 단계만 실패 처리하고 상위 로직은 계속 진행시키고 싶다면 반드시 `return`을 사용해야 한다.

## 실습 예제 (EX1~EX5)

**EX1. 여러 명령을 파이프로 연결하고 PIPESTATUS로 실패 지점 찾기**

요구사항 : 존재하지 않는 파일을 `grep`, `sort`, `uniq`로 이어 처리하면서 어느 단계에서 실패했는지 `PIPESTATUS`로 확인한다.

```bash
[root@Server-A ~]# vi ./script/pipe_debug.sh
#!/bin/bash
grep "error" /no_such_log.txt | sort | uniq -c

for i in "${!PIPESTATUS[@]}"; do
    echo "단계 $((i+1)) 종료코드 : ${PIPESTATUS[$i]}"
done

:wq


[root@Server-A ~]# chmod +x ./script/pipe_debug.sh
[root@Server-A ~]# ./script/pipe_debug.sh
grep: /no_such_log.txt: 그런 파일이나 디렉터리가 없습니다
단계 1 종료코드 : 2
단계 2 종료코드 : 0
단계 3 종료코드 : 0
```

**EX2. trap으로 종료 코드에 따라 다른 정리 작업 수행하기**

요구사항 : 백업 스크립트가 성공하면 완료 메시지를, 실패하면 알림 로그를 남기도록 `trap ... EXIT`를 사용한다.

```bash
[root@Server-A ~]# vi ./script/backup_trap.sh
#!/bin/bash

notify () {
    local code=$?
    if [ "$code" -eq 0 ]; then
        echo "$(date '+%F %T') 백업 성공" >> /var/log/backup_result.log
    else
        echo "$(date '+%F %T') 백업 실패(코드 $code)" >> /var/log/backup_result.log
    fi
}
trap notify EXIT

tar czf /backup/data_$(date +%F).tar.gz /data

:wq


[root@Server-A ~]# chmod +x ./script/backup_trap.sh
[root@Server-A ~]# ./script/backup_trap.sh
[root@Server-A ~]# tail -1 /var/log/backup_result.log
2026-09-09 10:20:11 백업 성공
```

**EX3. set -e와 pipefail을 함께 켜서 중간 실패까지 감지하기**

요구사항 : 파이프라인 중간 명령이 실패하면 스크립트 전체가 즉시 종료되도록 `set -e`와 `set -o pipefail`을 함께 사용한다.

```bash
[root@Server-A ~]# vi ./script/strict_pipe.sh
#!/bin/bash
set -eo pipefail

cat /no_such_file.txt | wc -l
echo "이 줄은 실행되지 않는다"

:wq


[root@Server-A ~]# chmod +x ./script/strict_pipe.sh
[root@Server-A ~]# ./script/strict_pipe.sh
cat: /no_such_file.txt: 그런 파일이나 디렉터리가 없습니다

[root@Server-A ~]# echo $?
1
```

**EX4. 시그널별 종료 코드를 직접 발생시켜 표로 확인하기**

요구사항 : `sleep` 프로세스를 SIGINT, SIGTERM, SIGKILL로 각각 종료시켜 `$?` 값을 비교한다.

```bash
[root@Server-A ~]# sleep 100 &
[1] 21001
[root@Server-A ~]# kill -INT %1; wait %1; echo "SIGINT -> $?"
SIGINT -> 130

[root@Server-A ~]# sleep 100 &
[1] 21010
[root@Server-A ~]# kill -TERM %1; wait %1; echo "SIGTERM -> $?"
SIGTERM -> 143

[root@Server-A ~]# sleep 100 &
[1] 21022
[root@Server-A ~]# kill -KILL %1; wait %1; echo "SIGKILL -> $?"
SIGKILL -> 137
```

**EX5. 중첩 함수에서 return과 exit의 차이를 직접 비교하기**

요구사항 : 동일한 구조의 스크립트를 두 벌 만들어 하나는 `return`, 하나는 `exit`를 사용해 실행 흐름 차이를 확인한다.

```bash
[root@Server-A ~]# ./script/nested_func.sh		# return 버전 : 끝까지 실행됨
outer 시작
inner 시작
inner의 리턴 코드 : 2
outer 계속 진행
스크립트 마지막 줄까지 도달함

[root@Server-A ~]# ./script/nested_func_exit.sh	# exit 버전 : 그 자리에서 즉시 종료
outer 시작
inner 시작
[root@Server-A ~]# echo $?
2
```

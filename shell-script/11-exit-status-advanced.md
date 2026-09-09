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

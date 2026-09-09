# Shell Script 실행 방법 (Script Execution Methods)

같은 스크립트 파일이라도 어떤 방식으로 실행하느냐에 따라 **새 프로세스(서브쉘)에서 도는지, 현재 쉘에서 그대로 도는지**, **Shebang을 참고하는지 무시하는지**가 달라진다. (Shebang 자체 문법은 `08-syntax-master.md`의 "Shebang과 실행 방식" 참고)

## 실행 권한으로 직접 실행 (./script.sh)

- `./script.sh`처럼 파일 경로를 직접 실행하면, 커널이 파일 맨 첫 줄의 **Shebang**을 읽어 그 인터프리터로 스크립트를 실행한다.
- 이 방식은 파일에 **실행 권한(x)**이 반드시 있어야 하며, 실행 권한이 없으면 Shebang이 있어도 거부된다.

```bash
[root@Server-A ~]# vi ./script/run_test.sh
#!/bin/bash
echo "실행 방식 : $0"

:wq


[root@Server-A ~]# ./script/run_test.sh
-bash: ./script/run_test.sh: 허가 거부됨		# 실행 권한이 없어서 거부


[root@Server-A ~]# chmod +x ./script/run_test.sh

[root@Server-A ~]# ./script/run_test.sh
실행 방식 : ./script/run_test.sh
```

**정리**: `./script.sh` 실행은 Shebang과 실행 권한(`chmod +x`)이 모두 갖춰져야 하며, Shebang에 지정된 인터프리터가 새 프로세스를 만들어 스크립트를 실행한다.

## bash · sh 명령으로 실행

- `bash script.sh` 또는 `sh script.sh`처럼 인터프리터를 직접 지정해서 실행하면, **실행 권한이 없어도** 스크립트를 실행할 수 있다.
- 이때는 Shebang을 아예 읽지 않는다. 파일 첫 줄에 `#!/bin/bash`가 적혀 있어도 무시하고, 명령어로 지정한 `bash` 또는 `sh`가 그대로 사용된다.
- `bash`/`sh` 둘 다 **새로운 서브쉘(자식 프로세스)**을 만들어 그 안에서 스크립트를 실행한다.

```bash
[root@Server-A ~]# ls -l ./script/run_test.sh
-rw-r--r-- 1 root root 32  8월 20 10:00 ./script/run_test.sh	# 실행 권한 없음


[root@Server-A ~]# bash ./script/run_test.sh
실행 방식 : ./script/run_test.sh


[root@Server-A ~]# sh ./script/run_test.sh	# Shebang이 #!/bin/bash여도 sh로 강제 실행됨
실행 방식 : ./script/run_test.sh
```

- 서브쉘에서 실행되므로, 스크립트 안에서 `cd`로 디렉터리를 옮기거나 변수를 만들어도 **부모 쉘(현재 터미널)에는 영향을 주지 않는다.**

```bash
[root@Server-A ~]# vi ./script/cd_test.sh
#!/bin/bash
cd /tmp
myvar="hello"

:wq


[root@Server-A ~]# pwd
/root

[root@Server-A ~]# bash ./script/cd_test.sh

[root@Server-A ~]# pwd			# 서브쉘 안에서만 이동했으므로 현재 쉘 위치는 그대로
/root

[root@Server-A ~]# echo $myvar		# 서브쉘의 지역 변수도 부모 쉘로 전달되지 않는다.
(출력 없음)
```

**정리**: `bash script.sh`/`sh script.sh`는 실행 권한 없이도 동작하지만 Shebang을 무시하고 지정한 인터프리터로 강제 실행하며, 항상 새로운 서브쉘에서 돌기 때문에 `cd`나 변수 변경이 현재 쉘에 반영되지 않는다.

## source · . 으로 실행 (현재 쉘에서 실행)

- `source script.sh` 또는 `. script.sh`(점 하나 + 공백 + 경로)는 **새로운 프로세스를 만들지 않고, 현재 실행 중인 쉘 안에서** 스크립트의 명령을 한 줄씩 그대로 실행한다.
- 그래서 스크립트 안에서 `cd`로 이동하거나 변수를 만들면, **그 변화가 현재 쉘에 그대로 남는다.**
- `.bashrc`, `.bash_profile`처럼 환경설정 파일을 현재 쉘에 즉시 반영할 때(`source ~/.bashrc`) 이 방식을 사용한다.

```bash
[root@Server-A ~]# pwd
/root

[root@Server-A ~]# source ./script/cd_test.sh

[root@Server-A ~]# pwd			# 현재 쉘이 실제로 /tmp로 이동함
/tmp

[root@Server-A ~]# echo $myvar		# 현재 쉘에 변수가 그대로 남음
hello


[root@Server-A ~]# cd ~
[root@Server-A ~]# . ./script/cd_test.sh	# '.'은 source와 완전히 동일하게 동작

[root@Server-A /tmp]# pwd
/tmp
```

- `source`/`.`는 실행 권한이 없어도 동작하며, Shebang도 참고하지 않는다(항상 현재 쉘 자체의 문법으로 해석).

**정리**: `source script.sh`(`. script.sh`)는 서브쉘을 만들지 않고 현재 쉘 안에서 스크립트를 그대로 실행하므로, 변수와 `cd` 결과가 호출한 쉘에 그대로 유지된다. 환경설정을 즉시 반영하거나, 여러 스크립트 사이에서 변수를 공유해야 할 때 사용한다.

## 세 가지 실행 방식 비교

| 방식 | 실행 권한 필요 | Shebang 참고 | 실행 위치 | 변수/cd 영향 |
|---|---|---|---|---|
| `./script.sh` | 필요 | O (참고함) | 새 프로세스(서브쉘) | 현재 쉘에 영향 없음 |
| `bash script.sh` / `sh script.sh` | 불필요 | X (무시, 지정한 인터프리터 강제) | 새 프로세스(서브쉘) | 현재 쉘에 영향 없음 |
| `source script.sh` / `. script.sh` | 불필요 | X (현재 쉘 문법으로 해석) | 현재 쉘 그대로 | 현재 쉘에 그대로 반영 |

**정리**: 스크립트를 "독립된 프로그램"으로 실행하려면 `./script.sh`나 `bash script.sh`를, 스크립트의 결과(변수, 현재 디렉터리)를 **현재 쉘에 반영**해야 한다면 `source`/`.`를 사용한다. 세 방식 모두 결과가 같아 보여도 프로세스 구조와 영향 범위가 다르다는 점이 핵심이다.

## 중첩 소싱 (sourced 스크립트가 다른 스크립트를 source)

- `source`로 실행 중인 스크립트 안에서 다시 `source`(또는 `.`)로 또 다른 스크립트를 불러올 수 있다. 어차피 둘 다 **현재 쉘 안에서** 실행되므로, 몇 단계를 거쳐 소싱하든 최종적으로는 전부 같은 하나의 쉘(현재 터미널) 안에 변수와 함수가 쌓인다.
- 설정 파일을 여러 조각으로 나눠 관리할 때(`common.sh`, `db.sh`, `app.sh`를 각각 만들고 `app.sh`가 `common.sh`와 `db.sh`를 소싱) 흔히 쓰는 구조다.

```bash
[root@Server-A ~]# vi ./script/common.sh
#!/bin/bash
LOG_DIR="/var/log/myapp"
echo "common.sh 로딩됨"

:wq


[root@Server-A ~]# vi ./script/db.sh
#!/bin/bash
source ./script/common.sh		# db.sh가 다시 common.sh를 source (중첩)
DB_HOST="192.168.0.10"
echo "db.sh 로딩됨 (LOG_DIR=$LOG_DIR)"

:wq


[root@Server-A ~]# vi ./script/app.sh
#!/bin/bash
source ./script/db.sh
echo "app.sh 시작, DB_HOST=$DB_HOST, LOG_DIR=$LOG_DIR"

:wq


[root@Server-A ~]# source ./script/app.sh
common.sh 로딩됨
db.sh 로딩됨 (LOG_DIR=/var/log/myapp)
app.sh 시작, DB_HOST=192.168.0.10, LOG_DIR=/var/log/myapp

[root@Server-A ~]# echo $DB_HOST		# 최종적으로 현재 쉘에 모두 변수로 남는다
192.168.0.10
```

- 여러 스크립트가 서로를 소싱하다 보면 같은 파일이 두 번 이상 로딩되어 변수를 다시 초기화하거나 함수 재정의 경고가 뜰 수 있다. 이를 막으려면 각 스크립트 맨 앞에 "이미 로딩됐는지" 표시하는 가드 변수를 두는 방법을 흔히 사용한다.

```bash
[root@Server-A ~]# vi ./script/common.sh
#!/bin/bash
if [ -n "$COMMON_SH_LOADED" ]; then
    return 0		# 이미 로딩된 적이 있으면 그냥 종료(중복 로딩 방지)
fi
COMMON_SH_LOADED=1

LOG_DIR="/var/log/myapp"
echo "common.sh 로딩됨"

:wq
```

**정리**: `source`는 새 프로세스를 만들지 않으므로 sourced 스크립트 안에서 또 다른 스크립트를 `source`하는 중첩도 자연스럽게 동작하며, 결국 모든 변수/함수가 최초로 소싱을 시작한 하나의 쉘에 누적된다. 다만 같은 파일이 여러 경로로 반복 소싱될 수 있으므로, 공용 설정 스크립트에는 중복 로딩 방지 가드를 넣는 것이 안전하다.

## source한 스크립트 안의 exit는 현재 쉘을 종료시킨다

- `./script.sh`나 `bash script.sh`로 실행한 스크립트 안의 `exit`는 그 스크립트가 돌던 **서브쉘(자식 프로세스)**만 종료시키고 현재 터미널은 멀쩡히 남는다.
- 그러나 `source script.sh`(`. script.sh`)로 실행하면 스크립트가 **현재 쉘 안에서** 그대로 실행되기 때문에, 스크립트 안의 `exit`가 곧 **현재 쉘(터미널) 자체를 종료**시켜 버린다. 실무에서 자주 발생하는 함정이다.

```bash
[root@Server-A ~]# vi ./script/danger_exit.sh
#!/bin/bash
echo "설정 값을 확인합니다."

if [ ! -f /etc/myapp.conf ]; then
    echo "설정 파일이 없습니다."
    exit 1			# source로 실행되면 이 exit가 현재 쉘을 그대로 종료시킨다
fi

echo "설정 파일 확인 완료"

:wq


[root@Server-A ~]# chmod +x ./script/danger_exit.sh

[root@Server-A ~]# ./script/danger_exit.sh	# 일반 실행이면 서브쉘만 종료, 터미널은 안전
설정 값을 확인합니다.
설정 파일이 없습니다.

[root@Server-A ~]# echo "터미널 살아있음 : $?"
터미널 살아있음 : 1


[root@Server-A ~]# source ./script/danger_exit.sh	# source로 실행하면...
설정 값을 확인합니다.
설정 파일이 없습니다.
Connection to server-a closed.			# 현재 쉘(SSH 세션)이 그대로 종료되어 접속이 끊긴다
```

- 이런 사고를 막기 위해, `source`로 불러 쓰도록 설계된 스크립트(환경설정 스크립트, 함수 모음 등)에서는 실패 처리 시 `exit` 대신 `return`을 사용하는 것이 안전하다. `return`은 함수 밖에서도 소싱되는 스크립트의 최상위 레벨에서 사용하면 "그 스크립트의 소싱만 중단"하는 효과를 낸다.

```bash
[root@Server-A ~]# vi ./script/safe_source.sh
#!/bin/bash
echo "설정 값을 확인합니다."

if [ ! -f /etc/myapp.conf ]; then
    echo "설정 파일이 없습니다."
    return 1 2>/dev/null || exit 1	# source면 return, 직접 실행이면 exit로 안전하게 분기
fi

echo "설정 파일 확인 완료"

:wq


[root@Server-A ~]# source ./script/safe_source.sh	# 터미널이 종료되지 않고 return만 됨
설정 값을 확인합니다.
설정 파일이 없습니다.

[root@Server-A ~]# echo "터미널 살아있음 : $?"
터미널 살아있음 : 1
```

**정리**: `exit`가 들어 있는 스크립트를 무심코 `source`로 실행하면 현재 쉘(터미널 세션)까지 함께 종료되어 SSH 접속이 끊기는 등의 사고로 이어질 수 있다. 다른 스크립트에 `source`되어 쓰일 목적의 스크립트라면 `exit` 대신 `return`을 사용하거나, `return 2>/dev/null || exit` 패턴으로 두 실행 방식 모두에 안전하게 대응해야 한다.

## bash -x / sh -x 로 실행 과정 디버그 추적

- `bash -x script.sh`(또는 `sh -x script.sh`)로 실행하면, 스크립트가 실제로 실행하는 **각 명령어를 치환된 값까지 포함해서** 화면에 그대로 출력해준다. 어느 줄에서 어떤 값으로 분기했는지 추적할 때 매우 유용한 디버깅 방법이다.
- 각 추적 줄 앞에는 기본적으로 `+` 기호가 붙으며, 이 기호는 `PS4` 변수로 바꿀 수 있다.

```bash
[root@Server-A ~]# vi ./script/debug_target.sh
#!/bin/bash
name="hong"
count=3

if [ "$count" -gt 1 ]; then
    echo "$name 님, ${count}개의 알림이 있습니다."
fi

:wq


[root@Server-A ~]# chmod +x ./script/debug_target.sh

[root@Server-A ~]# bash -x ./script/debug_target.sh
+ name=hong
+ count=3
+ '[' 3 -gt 1 ']'
+ echo '홍길동 님, 3개의 알림이 있습니다.'
hong 님, 3개의 알림이 있습니다.
```

- 스크립트 전체가 아니라 **일부 구간만** 추적하고 싶다면, 스크립트 안에서 `set -x`로 켜고 `set +x`로 끄면 된다.

```bash
[root@Server-A ~]# vi ./script/debug_partial.sh
#!/bin/bash
echo "여기는 추적 안 됨"

set -x				# 여기서부터 추적 시작
result=$((3 + 4))
echo "result=$result"
set +x				# 여기서부터 추적 종료

echo "여기도 추적 안 됨"

:wq


[root@Server-A ~]# chmod +x ./script/debug_partial.sh

[root@Server-A ~]# ./script/debug_partial.sh
여기는 추적 안 됨
+ result=7
+ echo result=7
result=7
+ set +x
여기도 추적 안 됨
```

- `PS4`에 줄 번호(`$LINENO`)나 스크립트 이름(`$0`)을 넣어두면, 추적 로그만 보고도 몇 번째 줄인지 바로 알 수 있어 긴 스크립트를 디버깅할 때 유용하다.

```bash
[root@Server-A ~]# PS4='+[${LINENO}] ' bash -x ./script/debug_target.sh
+[2] name=hong
+[3] count=3
+[5] '[' 3 -gt 1 ']'
+[6] echo '홍길동 님, 3개의 알림이 있습니다.'
hong 님, 3개의 알림이 있습니다.
```

**정리**: `bash -x`(또는 `sh -x`)는 스크립트가 실행하는 모든 명령을 변수 치환 결과까지 포함해서 그대로 보여주는 표준 디버깅 방법이며, `set -x`/`set +x`로 특정 구간만 추적하거나 `PS4`에 `$LINENO`를 넣어 줄 번호를 함께 표시할 수 있다.

## 세 가지 방식에 인자(Argument) 전달하기

- `./script.sh a b`, `bash script.sh a b`처럼 실행 명령 뒤에 값을 붙이면, 스크립트 안에서 `$1`, `$2`, `$@`, `$#`로 그 값을 받는다. 이 두 방식은 새 프로세스를 만들기 때문에 인자 전달이 자연스럽게 그 프로세스의 `$1`, `$2`...가 된다.

```bash
[root@Server-A ~]# vi ./script/show_args.sh
#!/bin/bash
echo "인자 개수 : $#"
echo "첫 번째 : $1, 두 번째 : $2"
echo "전체 : $@"

:wq


[root@Server-A ~]# chmod +x ./script/show_args.sh

[root@Server-A ~]# ./script/show_args.sh apple banana
인자 개수 : 2
첫 번째 : apple, 두 번째 : banana
전체 : apple banana

[root@Server-A ~]# bash ./script/show_args.sh apple banana
인자 개수 : 2
첫 번째 : apple, 두 번째 : banana
전체 : apple banana
```

- `source script.sh a b`도 뒤에 값을 붙이면 인자로 전달되지만, `source`는 **현재 쉘** 안에서 도는 것이라 헷갈리기 쉽다. `source`로 스크립트를 실행하는 동안에는 현재 쉘의 `$1`, `$2`가 일시적으로 그 값으로 바뀌고, 스크립트 실행이 끝나도 원래 값으로 자동 복원되지 않는다는 점에 주의해야 한다.

```bash
[root@Server-A ~]# set -- original1 original2	# 현재 쉘의 $1, $2를 미리 설정
[root@Server-A ~]# echo $1 $2
original1 original2

[root@Server-A ~]# source ./script/show_args.sh apple banana
인자 개수 : 2
첫 번째 : apple, 두 번째 : banana
전체 : apple banana

[root@Server-A ~]# echo $1 $2			# source 이후에도 apple banana로 남아있음(자동 복원 안 됨)
apple banana
```

- 그래서 `source`되는 스크립트에서 인자를 다루고 나면, 필요할 경우 `set --`으로 위치 매개변수를 직접 원래대로 되돌리거나 비워줘야 한다.

```bash
[root@Server-A ~]# vi ./script/source_args_safe.sh
#!/bin/bash
echo "받은 인자로 작업 : $1 $2"
# ... 작업 ...
set --			# 위치 매개변수를 깨끗이 비워서 현재 쉘에 흔적을 남기지 않음

:wq
```

**정리**: 세 방식 모두 명령 뒤에 값을 붙이면 `$1`, `$2`, `$@`로 인자를 받을 수 있지만, `./script.sh`와 `bash script.sh`는 별도 프로세스의 인자이므로 끝나면 자동으로 사라지는 반면, `source script.sh`는 **현재 쉘의 위치 매개변수 자체를 바꿔버리고 실행이 끝나도 복원되지 않으므로**, 필요하면 `set --`으로 직접 정리해야 한다.

## 실습 예제 (EX1~EX4)

**EX1. 환경설정 스크립트를 만들어 source로 현재 쉘에 즉시 반영하기**

요구사항 : `PROJECT_HOME`, `PROJECT_LOG` 두 환경변수를 설정하고 `PATH`에 프로젝트 bin 디렉터리를 추가하는 스크립트를 작성하고, `source`로 실행해 현재 쉘에 바로 반영되는지 확인한다.

```bash
[root@Server-A ~]# vi ./script/setenv.sh
#!/bin/bash
export PROJECT_HOME="/opt/myproject"
export PROJECT_LOG="$PROJECT_HOME/logs"
export PATH="$PATH:$PROJECT_HOME/bin"
echo "프로젝트 환경변수가 설정되었습니다."

:wq


[root@Server-A ~]# chmod +x ./script/setenv.sh

[root@Server-A ~]# ./script/setenv.sh		# 그냥 실행하면 서브쉘에서만 설정되고 사라짐
프로젝트 환경변수가 설정되었습니다.
[root@Server-A ~]# echo $PROJECT_HOME
(출력 없음)

[root@Server-A ~]# source ./script/setenv.sh	# source로 실행해야 현재 쉘에 반영된다
프로젝트 환경변수가 설정되었습니다.
[root@Server-A ~]# echo $PROJECT_HOME
/opt/myproject
```

**EX2. 동일 스크립트를 세 가지 방식으로 실행해 결과 차이 확인하기**

요구사항 : 현재 디렉터리를 `/tmp`로 옮기는 스크립트를 `./script.sh`, `bash script.sh`, `source script.sh` 세 가지로 각각 실행하고 `pwd` 결과를 비교한다.

```bash
[root@Server-A ~]# vi ./script/move_tmp.sh
#!/bin/bash
cd /tmp
echo "스크립트 내부 pwd : $(pwd)"

:wq


[root@Server-A ~]# chmod +x ./script/move_tmp.sh

[root@Server-A ~]# pwd
/root
[root@Server-A ~]# ./script/move_tmp.sh
스크립트 내부 pwd : /tmp
[root@Server-A ~]# pwd			# 서브쉘만 이동, 현재 쉘은 그대로
/root

[root@Server-A ~]# bash ./script/move_tmp.sh
스크립트 내부 pwd : /tmp
[root@Server-A ~]# pwd
/root

[root@Server-A ~]# source ./script/move_tmp.sh
스크립트 내부 pwd : /tmp
[root@Server-A /tmp]# pwd		# 현재 쉘 자체가 이동됨
/tmp
```

**EX3. bash -x로 조건문 분기 과정을 추적하기**

요구사항 : 디스크 사용률을 흉내 낸 변수로 임계값을 넘는지 판단하는 스크립트를 `bash -x`로 실행해서 실제 비교 과정을 눈으로 확인한다.

```bash
[root@Server-A ~]# vi ./script/disk_check.sh
#!/bin/bash
usage=87
limit=80

if [ "$usage" -ge "$limit" ]; then
    echo "경고 : 디스크 사용률이 ${usage}% 입니다."
else
    echo "정상 : 디스크 사용률이 ${usage}% 입니다."
fi

:wq


[root@Server-A ~]# chmod +x ./script/disk_check.sh

[root@Server-A ~]# bash -x ./script/disk_check.sh
+ usage=87
+ limit=80
+ '[' 87 -ge 80 ']'
+ echo '경고 : 디스크 사용률이 87% 입니다.'
경고 : 디스크 사용률이 87% 입니다.
```

**EX4. source되는 함수 모음 스크립트에서 exit 대신 return 사용하기**

요구사항 : 필수 명령어(`jq`)가 설치되어 있는지 검사하는 함수 스크립트를 만들되, `source`로 불러써도 현재 쉘이 끊기지 않도록 `exit` 대신 `return`을 사용한다.

```bash
[root@Server-A ~]# vi ./script/check_jq.sh
#!/bin/bash
check_jq () {
    if ! command -v jq >/dev/null 2>&1; then
        echo "jq가 설치되어 있지 않습니다."
        return 1
    fi
    echo "jq 사용 가능"
    return 0
}

:wq


[root@Server-A ~]# source ./script/check_jq.sh
[root@Server-A ~]# check_jq
jq가 설치되어 있지 않습니다.
[root@Server-A ~]# echo "터미널 살아있음, 종료코드=$?"
터미널 살아있음, 종료코드=1
```

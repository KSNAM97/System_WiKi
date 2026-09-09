# Shell Script 대화형/비대화형 & 로그인/비로그인 쉘 (Interactive/Login Shell)

## 대화형(Interactive) vs 비대화형(Non-Interactive) 쉘

- **대화형 쉘**: 터미널에 프롬프트(`$`, `#`)가 뜨고, 사용자가 직접 명령을 입력하고 결과를 즉시 확인하는 쉘. 터미널을 열었을 때의 쉘이 대표적이다.
- **비대화형 쉘**: 스크립트 파일을 실행할 때처럼, 사람이 명령을 하나씩 입력하지 않고 미리 정해진 명령들을 자동으로 실행하는 쉘.

현재 쉘이 대화형인지는 특수 변수 **`$-`**(현재 쉘에 설정된 옵션 플래그 모음) 로 확인한다. 대화형 쉘이면 이 값에 `i` 플래그가 포함된다.

```bash
[root@Server-A ~]# echo $-
himBHs					# 'i'가 포함 -> 대화형 쉘


[root@Server-A ~]# vi ./script/interactive_check.sh
#!/bin/bash

case "$-" in
    *i*)
        echo "대화형 쉘입니다."
        ;;
    *)
        echo "비대화형 쉘입니다."
        ;;
esac

:wq


[root@Server-A ~]# chmod +x ./script/interactive_check.sh

[root@Server-A ~]# ./script/interactive_check.sh	# 스크립트로 실행하면 비대화형
비대화형 쉘입니다.

[root@Server-A ~]# source ./script/interactive_check.sh	# 현재(대화형) 쉘에서 그대로 실행하면 대화형
대화형 쉘입니다.
```

- 프롬프트를 표시할지 여부를 결정하는 **`PS1`** 변수가 설정되어 있는지로도 대화형 여부를 판단할 수 있다. 대화형 쉘에는 `PS1`이 설정되어 있고, 스크립트로 실행되는 비대화형 쉘에는 기본적으로 `PS1`이 비어 있다.

```bash
[root@Server-A ~]# echo "PS1 값 : $PS1"
PS1 값 : [\u@\h \W]\$
```

**비대화형 쉘에서 달라지는 대표적인 동작**

| 항목 | 대화형 쉘 | 비대화형 쉘(스크립트) |
|---|---|---|
| alias | 기본적으로 사용 가능 | 기본적으로 비활성화(스크립트에서 alias 오작동 방지) |
| history 확장(`!!` 등) | 사용 가능 | 비활성화 |
| Job Control (`bg`, `fg`, `suspend`) | 사용 가능 | 사용 불가 (`&` 백그라운드 실행 자체는 가능) |
| 시작 시 읽는 파일 | `~/.bashrc` | 기본적으로 없음 (`BASH_ENV` 변수에 지정된 파일이 있으면 그 파일을 읽음) |

**정리**: 대화형 쉘은 사람이 직접 명령을 주고받는 쉘, 비대화형 쉘은 스크립트처럼 자동으로 명령이 실행되는 쉘이다. `$-`에 `i`가 있는지, `PS1`이 설정되어 있는지로 구분하며, 비대화형 쉘에서는 alias·history 확장·job control이 제한된다.

## 로그인(Login) vs 비로그인(Non-Login) 쉘

**대화형/비대화형**이 "사람이 직접 치는가"의 구분이라면, **로그인/비로그인**은 "이 쉘이 계정 로그인 절차를 거쳐 시작됐는가"의 구분이다.

- **로그인 쉘**: SSH 접속, 콘솔 로그인처럼 사용자 인증(로그인) 과정을 거쳐 시작된 쉘.
- **비로그인 쉘**: 이미 로그인한 상태에서 터미널 에뮬레이터를 새로 열거나, 스크립트 실행처럼 로그인 절차 없이 시작된 쉘.

로그인 쉘인지는 `$0`(현재 쉘의 이름) 앞에 **`-`**가 붙는지로 확인할 수 있다.

```bash
[root@Server-A ~]# echo $0
-bash					# 맨 앞에 '-'가 붙어 있으면 로그인 쉘


[root@Server-A ~]# bash			# 비로그인 쉘로 새로 하나 실행
[root@Server-A ~]# echo $0
bash					# '-'가 없으면 비로그인 쉘


[root@Server-A ~]# exit
```

- `shopt` 로도 확인 가능하다.

```bash
[root@Server-A ~]# shopt login_shell
login_shell     	on		# 로그인 쉘이면 on, 아니면 off
```

- 비로그인 쉘을 로그인 쉘처럼 강제로 띄우려면 `bash -l`, `sh -l`을 사용한다. `su -`, `sudo -i`도 로그인 쉘로 전환하는 대표적인 방법이다.

**정리**: 로그인 쉘은 SSH 접속·콘솔 로그인처럼 인증 절차를 거쳐 시작된 쉘, 비로그인 쉘은 터미널 에뮬레이터·스크립트 실행처럼 이미 로그인된 상태에서 파생된 쉘이다. `$0`의 `-` 접두어나 `shopt login_shell`로 구분한다.

## 설정 파일이 로딩되는 조합 정리

대화형/비대화형, 로그인/비로그인의 조합에 따라 **어떤 설정 파일을 읽는지**가 달라지며, 이 차이 때문에 "SSH로 접속하면 되던 alias가 스크립트에서는 안 먹힌다" 같은 문제가 자주 발생한다.

| 조합 | 읽는 설정 파일 (순서대로) |
|---|---|
| 로그인 + 대화형 (SSH 접속 직후) | `/etc/profile` → `~/.bash_profile`(없으면 `~/.bash_login` → `~/.profile` 순으로 하나만) |
| 비로그인 + 대화형 (터미널 새 탭) | `/etc/bashrc`(배포판에 따라 `/etc/bash.bashrc`) → `~/.bashrc` |
| 비로그인 + 비대화형 (스크립트 실행) | 기본적으로 없음. `BASH_ENV` 변수에 지정된 파일이 있으면 그 파일만 읽음 |

- 대부분의 `~/.bash_profile`은 내부에서 `~/.bashrc`를 다시 `source`하도록 되어 있어서, 실무에서는 로그인 쉘로 접속해도 결과적으로 `.bashrc`의 alias/함수까지 함께 로딩되는 경우가 많다.

```bash
[root@Server-A ~]# cat ~/.bash_profile
# .bash_profile

if [ -f ~/.bashrc ]; then
        . ~/.bashrc
fi

PATH=$PATH:$HOME/.local/bin:$HOME/bin
export PATH
```

- 반대로 스크립트(`./script.sh`)는 비로그인+비대화형이므로 `.bashrc`의 alias나 함수를 전혀 물려받지 못한다. 스크립트 안에서 특정 함수/alias가 필요하면 그 파일을 직접 `source`해야 한다.

```bash
[root@Server-A ~]# vi ./script/need_alias.sh
#!/bin/bash

source ~/.bashrc		# 스크립트는 .bashrc를 자동으로 읽지 않으므로 명시적으로 source

my_alias_function

:wq
```

**정리**: 로그인 쉘은 `/etc/profile` 계열, 비로그인 대화형 쉘은 `/etc/bashrc`·`~/.bashrc` 계열을 읽고, 비로그인 비대화형(스크립트)은 기본적으로 아무 설정 파일도 읽지 않는다. 스크립트에서 alias나 쉘 함수가 필요하면 해당 설정 파일을 직접 `source`해서 불러와야 한다.

## $- 플래그 문자열 완전 정리

- `$-`는 현재 쉘에 켜져 있는 옵션 플래그들을 한 문자열로 이어붙여 보여준다. `i`(대화형) 하나만 보고 넘어가기 쉽지만, 나머지 문자들도 각각 의미가 있다.

| 플래그 | 의미 |
|---|---|
| `i` | 대화형(interactive) 쉘 |
| `h` | 명령어를 찾을 때 경로를 해시테이블에 캐시(`hashall`) |
| `m` | Job Control(모니터 모드) 활성화 |
| `B` | 중괄호 확장(`{a,b,c}`) 활성화 |
| `H` | history 확장(`!!`, `!$`) 활성화 |
| `s` | 표준입력에서 명령을 읽는 중(파이프로 스크립트를 넘길 때 등) |
| `x` | `set -x`(디버그 추적) 활성화 상태 |
| `e` | `set -e`(errexit) 활성화 상태 |

```bash
[root@Server-A ~]# echo $-
himBHs

[root@Server-A ~]# set -x
[root@Server-A ~]# echo $-
himBHsx				# set -x를 켜면 x가 추가됨

[root@Server-A ~]# set +x
```

- 스크립트 안에서 **현재 쉘이 대화형인지**를 판단하는 표준적인 방법은 `case "$-" in *i*)`나 `[[ $- == *i* ]]`처럼 `$-`에 `i`가 포함되어 있는지를 검사하는 것이다. `[[ ]]`의 `==` 오른쪽은 패턴으로 해석되므로 `*i*`처럼 앞뒤에 `*`를 붙여야 한다.

```bash
[root@Server-A ~]# vi ./script/is_interactive.sh
#!/bin/bash

if [[ $- == *i* ]]; then
    echo "대화형 쉘에서 실행 중입니다."
else
    echo "비대화형 쉘(스크립트)에서 실행 중입니다."
fi

:wq


[root@Server-A ~]# chmod +x ./script/is_interactive.sh

[root@Server-A ~]# ./script/is_interactive.sh
비대화형 쉘(스크립트)에서 실행 중입니다.

[root@Server-A ~]# source ./script/is_interactive.sh
대화형 쉘에서 실행 중입니다.
```

**정리**: `$-`는 `i` 외에도 `h`, `m`, `B`, `H`, `x`, `e` 등 현재 쉘에 켜진 여러 옵션을 한 번에 보여주는 문자열이며, `[[ $- == *i* ]]`처럼 부분 문자열 매치로 대화형 여부를 스크립트 안에서 안전하게 판별할 수 있다.

## 4가지 조합별 설정 파일 로딩 순서 전체 표

대화형/비대화형과 로그인/비로그인을 조합하면 이론적으로 4가지 경우가 나온다. 각 경우에 실제로 읽히는 파일을 모두 정리하면 다음과 같다.

| 조합 | 대표 상황 | 읽는 파일(순서대로) |
|---|---|---|
| 로그인 + 대화형 | SSH 접속 직후, 콘솔 로그인 | `/etc/profile` → `/etc/profile.d/*.sh` → (`~/.bash_profile` 또는 `~/.bash_login` 또는 `~/.profile` 중 존재하는 첫 번째 하나만) |
| 로그인 + 비대화형 | `bash -l -c "명령"`처럼 로그인 옵션을 강제로 준 배치 실행 | 위 로그인 쉘과 동일 (`/etc/profile` 계열) |
| 비로그인 + 대화형 | 이미 로그인된 터미널에서 새 탭/새 창을 여는 경우 | `/etc/bashrc`(배포판에 따라 `/etc/bash.bashrc`) → `~/.bashrc` |
| 비로그인 + 비대화형 | `./script.sh` 실행, cron, `at`, SSH로 원격 명령 1줄만 실행 | 기본적으로 없음. `BASH_ENV` 변수가 지정되어 있으면 그 파일 하나만 읽음 |

- 실무에서 가장 헷갈리는 지점은 "로그인 쉘은 `~/.bashrc`를 직접 읽지 않는다"는 것이다. 그래서 대부분의 배포판은 `~/.bash_profile` 안에 `~/.bashrc`를 다시 `source`하는 코드를 기본으로 넣어둬서, 결과적으로 로그인 쉘에서도 `.bashrc`가 함께 로딩되는 것처럼 보이게 만든다(이 문서 위쪽 "설정 파일이 로딩되는 조합 정리"의 `.bash_profile` 예시 참고).
- `~/.bash_login`, `~/.profile`은 `~/.bash_profile`이 없을 때만 순서대로 대신 읽히며, 셋 중 **가장 먼저 발견된 파일 하나만** 읽고 나머지는 무시한다.

```bash
[root@Server-A ~]# ls -a ~ | grep -E 'bash_profile|bash_login|profile'
.bash_profile			# 이 파일이 있으므로 .bash_login, .profile은 무시됨
```

**정리**: 4가지 조합 중 로그인 쉘 2가지(대화형/비대화형)는 `/etc/profile` 계열을, 비로그인 대화형은 `/etc/bashrc`·`~/.bashrc` 계열을, 비로그인 비대화형은 기본적으로 아무것도(또는 `BASH_ENV`만) 읽지 않는다. `~/.bash_profile`, `~/.bash_login`, `~/.profile`은 셋 중 먼저 발견되는 하나만 적용된다는 점이 자주 놓치는 부분이다.

## cron · at : 대표적인 비로그인 비대화형 쉘

- `crontab`으로 등록한 작업과 `at`으로 예약한 작업은 모두 **비로그인 + 비대화형** 쉘에서 실행된다. 사람이 로그인해서 만든 `PATH`, alias, 함수 환경을 전혀 물려받지 못한다는 뜻이다.
- 가장 흔한 실무 사고 사례: 터미널에서는 잘 되던 스크립트가 cron으로 등록하면 `command not found`가 뜨는 경우다. 원인은 대부분 cron의 기본 `PATH`가 대화형 쉘의 `PATH`보다 훨씬 짧기 때문이다.

```bash
[root@Server-A ~]# echo $PATH			# 대화형 로그인 쉘의 PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/.local/bin:/root/bin

[root@Server-A ~]# crontab -e
* * * * * echo "PATH=$PATH" >> /tmp/cron_path.log
:wq

[root@Server-A ~]# sleep 65 && cat /tmp/cron_path.log
PATH=/usr/bin:/bin				# cron 기본 PATH는 훨씬 짧다
```

- alias도 마찬가지다. `~/.bashrc`에 정의한 alias는 cron이 읽는 환경에 전혀 포함되지 않는다.

```bash
[root@Server-A ~]# cat ~/.bashrc | grep myalias
alias myalias='echo hello'

[root@Server-A ~]# crontab -e
* * * * * myalias >> /tmp/cron_alias.log 2>&1
:wq

[root@Server-A ~]# sleep 65 && cat /tmp/cron_alias.log
/bin/sh: myalias: command not found		# .bashrc를 안 읽으므로 alias가 존재하지 않음
```

- 해결책은 두 가지다. 하나는 스크립트/명령에서 **절대 경로**를 사용하는 것, 다른 하나는 cron 작업 맨 앞에서 필요한 환경을 직접 `source`하거나 `PATH`를 명시적으로 지정하는 것이다.

```bash
[root@Server-A ~]# crontab -e
PATH=/usr/local/bin:/usr/bin:/bin
* * * * * . /root/.bash_profile; /root/script/backup.sh >> /var/log/backup.log 2>&1
:wq
```

- SSH로 원격 명령을 한 줄만 실행하는 경우(`ssh server-a "ls /var/log"`)도 비대화형 쉘로 처리된다. 반면 `ssh server-a`처럼 인자 없이 접속하면 로그인 + 대화형 쉘이 열린다.

```bash
[user@client ~]$ ssh server-a "echo \$-"
hBc					# 'i'가 없음 -> 비대화형

[user@client ~]$ ssh server-a "echo \$0"
bash					# '-'가 없음 -> 비로그인

[user@client ~]$ ssh server-a
[root@Server-A ~]# echo $-
himBHs					# 'i' 있음, 대화형 로그인 쉘
```

**정리**: cron 작업, `at` 작업, SSH 원격 명령 한 줄 실행은 모두 비로그인·비대화형 쉘에서 돈다. `.bashrc`의 alias, 사람이 로그인해서 갖춰둔 `PATH`가 전혀 적용되지 않으므로, 자동화 스크립트에서는 절대 경로를 쓰거나 필요한 환경설정 파일을 명시적으로 `source`해야 한다.

## 실습 예제 (EX1~EX4)

**EX1. cron으로 실행되는 스크립트에서 PATH가 다르게 동작하는 것을 확인**

요구사항 : 현재 `PATH`를 파일에 기록하는 스크립트를 만들어 터미널에서 직접 실행한 결과와 cron으로 실행한 결과를 비교한다.

```bash
[root@Server-A ~]# vi ./script/check_path.sh
#!/bin/bash
echo "PATH=$PATH" > /tmp/path_direct.log

:wq


[root@Server-A ~]# chmod +x ./script/check_path.sh
[root@Server-A ~]# ./script/check_path.sh
[root@Server-A ~]# cat /tmp/path_direct.log
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/.local/bin:/root/bin

[root@Server-A ~]# crontab -e
* * * * * /root/script/check_path.sh > /tmp/path_cron.log 2>&1
:wq

[root@Server-A ~]# sleep 65 && cat /tmp/path_cron.log
PATH=/usr/bin:/bin			# 훨씬 짧은 기본 PATH로 실행됨을 확인
```

**EX2. SSH 원격 명령 실행이 비대화형 쉘로 동작하는 것을 확인**

요구사항 : SSH로 접속만 했을 때와, SSH로 명령을 한 줄만 넘겼을 때 `$-` 값을 비교한다.

```bash
[user@client ~]$ ssh server-a "echo \$-"
hBc

[user@client ~]$ ssh server-a
[root@Server-A ~]# echo $-
himBHs

[root@Server-A ~]# exit
```

**EX3. .bashrc의 함수를 cron에서 쓰기 위해 명시적으로 source하기**

요구사항 : `~/.bashrc`에 정의한 백업 함수를 cron 작업에서도 사용할 수 있도록 스크립트 안에서 직접 `source`한다.

```bash
[root@Server-A ~]# cat ~/.bashrc | grep -A2 do_backup
do_backup () {
    tar czf /backup/data_$(date +%F).tar.gz /data
}

[root@Server-A ~]# vi ./script/cron_backup.sh
#!/bin/bash
source /root/.bashrc
do_backup
echo "백업 완료 : $(date)"

:wq


[root@Server-A ~]# chmod +x ./script/cron_backup.sh
[root@Server-A ~]# crontab -e
0 2 * * * /root/script/cron_backup.sh >> /var/log/cron_backup.log 2>&1
:wq
```

**EX4. 로그인/비로그인 쉘을 직접 오가며 $0 값 비교하기**

요구사항 : 현재 쉘의 `$0`을 확인한 뒤 `bash`로 비로그인 쉘을 새로 띄우고, 다시 `bash -l`로 로그인 쉘을 띄워 `$0` 표기 차이를 확인한다.

```bash
[root@Server-A ~]# echo $0
-bash

[root@Server-A ~]# bash
[root@Server-A ~]# echo $0
bash

[root@Server-A ~]# exit

[root@Server-A ~]# bash -l
[root@Server-A ~]# echo $0
-bash

[root@Server-A ~]# exit
```

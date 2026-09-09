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

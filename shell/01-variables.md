# Shell Script 변수 (Variables) & 환경변수

## 목차

1. [커널(Kernel)과 쉘(Shell)](#커널kernel과-쉘shell)
2. [쉘 스크립트(Shell Script)](#쉘-스크립트shell-script)
3. [GUI/CLI 환경 전환 및 쉘 관리](#guicli-환경-전환-및-쉘-관리)
4. [Bash shell 변수](#bash-shell-변수)
5. [변수 실습 예제 (EX1~EX14)](#변수-실습-예제-ex1ex14)
6. [Shell 환경 변수](#shell-환경-변수)
7. [환경변수 실습 예제 (EX1~EX9)](#환경변수-실습-예제-ex1ex9)

## 커널(Kernel)과 쉘(Shell)

**커널(Kernel)**은 운영체제의 핵심 부분이다. 변수는 반복되는 경로나 값을 한 곳에서 관리하고, 환경변수로 승격해 자식 프로세스와 설정을 공유하는 데 쓰인다.

- CPU, 메모리, 디스크, 네트워크 같은 하드웨어를 직접 제어한다.
- 응용 프로그램이 하드웨어를 바로 사용할 수 없기 때문에, 커널이 중간에서 모든 자원을 관리한다.
- 대표 기능
  - 프로세스 관리(프로그램 실행)
  - 메모리 관리
  - 파일 시스템 관리
  - 네트워크 관리
  - 디바이스 관리
- 커널은 OS의 심장(핵심 엔진) 역할을 한다.

**쉘(Shell)의 개념**

**Shell**은 사용자가 입력한 명령을 커널에게 전달하는 번역기 역할이다.

- 사용자는 커널을 직접 다룰 수 없기 때문에, 쉘을 통해 명령을 전달한다.
- 대표적인 쉘 : **sh** , **bash** , **zsh** , **ksh**

**sh**
- 가장 기본적인 전통적인 유닉스 쉘
- 기능은 단순하지만 호환성이 매우 높다.
- 시스템 초기화 스크립트나 POSIX 표준 스크립트에서 자주 사용

**bash**
- 대부분의 리눅스 배포판에서 기본 쉘로 사용된다.
- 기능이 많고 사용자가 가장 많이 사용하며 스크립트 작성도 편리하다.
- 자동완성, 히스토리, 배열, 함수 등 다양한 기능 제공

**zsh**
- bash보다 강력한 자동완성과 편리한 프롬프트 기능을 제공한다.
- 개발자나 파워유저들이 선호하는 고급 쉘이다.
- oh-my-zsh 같은 플러그인 시스템이 유명

**ksh**
- Korn Shell로, 상업용 Unix 계열 시스템(HP-UX, AIX 등)에서 많이 사용되었다.
- 성능이 좋고 스크립트 기능이 강화되어 있다.
- sh과 bash 중간쯤의 기능을 가진 쉘

- **CLI**(Command Line Interface) 형태로 많이 사용한다.

흐름 : 사용자 입력  -->  쉘  -->  커널  -->  하드웨어  -->  결과를 쉘이 화면에 출력

**정리**: 커널은 하드웨어를 관리하는 핵심이고, 쉘은 사용자와 커널 사이에서 명령을 번역/전달하는 역할을 한다.

## 쉘 스크립트(Shell Script)

**쉘 스크립트**는 Shell에서 사용하는 명령어들을 파일로 묶어 자동 실행하는 프로그램이다.

- Bash 명령을 여러 줄 저장한 텍스트 파일로 확장자는 보통 `.sh`을 사용한다.
- 내부에는 평소 터미널에서 치는 명령들 + if, for, while 같은 제어문 + 변수 등을 함께 사용

**쉘 스크립트를 사용하는 이유**
- 반복 작업 자동화
- 매일 로그 백업, 오래된 로그 삭제, 특정 서비스 상태 점검 등
- 여러 명령을 순서대로 묶어서 실행
- 패키지 설치 --> 설정 파일 복사 --> 서비스 재시작 --> 확인까지 한 번에 가능
- 사람이 실수하기 쉬운 작업을 동일한 방식으로 반복할 수 있게 함

예시
```
EX) 파일명: backup.sh
#!/bin/bash
tar cvf /backup/log_$(date +%F).tar /var/log
```

: 매일 log 백업을 자동화
: 크론(cron)과 함께 사용하면 정해진 시간마다 자동 실행 가능

**정리**
- 쉘		: 명령을 해석해서 실행해주는 프로그램
- 쉘 스크립트	: 그 쉘이 실행할 명령들을 파일로 묶어놓은 것

## GUI/CLI 환경 전환 및 쉘 관리

**GUI  --->  CLI 환경으로 변경**

```
[root@Server-A ~]# systemctl  set-default multi-user.target
Removed "/etc/systemd/system/default.target".
Created symlink /etc/systemd/system/default.target → /usr/lib/systemd/system/multi-user.target.


[root@Server-A ~]# reboot		# 처음에 GUI로 부팅되었기때문에 재부팅 후 CLI 환경으로 부팅된다.
```

**CLI  --->  GUI 환경으로 변경**

```
[root@Server-A ~]# systemctl set-default graphical.target
Removed "/etc/systemd/system/default.target".
Created symlink /etc/systemd/system/default.target → /usr/lib/systemd/system/graphical.target.


[root@Server-A ~]# reboot		# 처음에 CLI로 부팅되었기때문에 재부팅 후 GUI 환경으로 부팅된다.
```

**쉘 추가 설치**

```
[root@Server-A ~]# cat /etc/shells
/bin/sh
/bin/bash
/usr/bin/sh
/usr/bin/bash



[root@Server-A ~]# cat /etc/passwd | grep root
root:x:0:0:root:/root:/bin/bash			# 로그인시 적용되는 쉘 = /bin/bash
operator:x:11:0:operator:/root:/sbin/nologin


[root@Server-A ~]# echo $SHELL	# 현재 사용중인 쉘 확인
/bin/bash



[root@Server-A ~]# ls  -l  /bin/??sh
-rwxr-xr-x. 1 root root 1389024  4월 30  2024 /bin/bash
-rws--x--x. 1 root root   23784  1월 22 09:02 /bin/chsh
-rwxr-xr-x. 1 root root        32  4월 30  2024 /bin/hash


[root@Server-A ~]# ls  -l  /bin/?sh
-rwxr-xr-x. 1 root root 917872  5월 20 09:49 /bin/ssh




[root@Server-A ~]# dnf  install  -y  csh  zsh  tcsh	# 추가로 쉘 설치


[root@Server-A ~]# cat  /etc/shells
/bin/sh
/bin/bash
/usr/bin/sh
/usr/bin/bash
/bin/csh
/bin/tcsh
/usr/bin/csh
/usr/bin/tcsh
/usr/bin/zsh
/bin/zsh



[root@Server-A ~]# ls  -l  /bin/?sh
lrwxrwxrwx  1 root root       4  5월 16  2022 /bin/csh -> tcsh
-rwxr-xr-x. 1 root root 917872  5월 20 09:49 /bin/ssh
-rwxr-xr-x  1 root root 997736  5월 15  2022 /bin/zsh


[root@Server-A ~]# ls  -l  /bin/??sh
-rwxr-xr-x. 1 root root 1389024  4월 30  2024 /bin/bash
-rws--x--x. 1 root root   23784  1월 22 09:02 /bin/chsh
-rwxr-xr-x. 1 root root        32  4월 30  2024 /bin/hash
-rwxr-xr-x  1 root root  488080  5월 16  2022 /bin/tcsh
```

리눅스에서 **쉘(Shell)** 은 사용자의 명령을 받아 운영체제에 전달하는 명령 해석기(Command Interpreter) 이다.
하지만 `/etc/쉘`과 `/bin/쉘`은 의미가 다르다.

| 구분 | /bin | /etc |
|------|------|------|
| 역할 | 실제 쉘 프로그램 | 쉘 설정 파일 |
| 내용 | bash, sh, csh, tcsh, zsh 등 | profile, bashrc, shells 등 |
| 기능 | 명령을 실행하는 프로그램 | 쉘의 환경과 동작을 설정 |

**/bin의 Shell**

- `/bin`에는 실제 실행되는 쉘 프로그램이 저장되어 있다.
  - /bin/bash
  - /bin/sh
  - /bin/csh
  - /bin/tcsh
  - /bin/zsh
- 사용자가 로그인하면 `/etc/passwd`에 지정된 쉘 프로그램이 실행된다.
  - guest 사용자는 로그인하면 /bin/bash가 실행된다.

```
[root@Server-A ~]# cat  /etc/passwd | grep guest
guest:x:1000:1000:guest:/home/guest:/bin/bash
```

**/bin과 /etc의 관계**

- 사용자가 로그인하면 다음 순서로 동작한다.
  - 사용자 로그인  -->  /etc/passwd 확인  -->  /bin/bash 실행  -->  /etc/profile 읽음  -->  /etc/bashrc 읽음  -->  사용자가 명령 입력
  - /bin	: 실제로 실행되는 쉘 프로그램
  - /etc	: 실행된 쉘이 사용할 환경을 설정하는 파일

```
login as: guest			# guest 계정으로 로그인
guest@192.168.10.100's password:
Activate the web console with: systemctl enable --now cockpit.socket

Last login: Tue Jul 21 16:00:17 2026 from 192.168.10.1
[guest@Server-A ~]$
[guest@Server-A ~]$ echo $SHELL
/bin/bash				# bash사용 확인
```

**guest 계정의 기본 쉘을 sh로 변경**

```
[guest@Server-A ~]$ cat /etc/passwd | grep guest
guest:x:1000:1000:guest:/home/guest:/bin/bash


[guest@Server-A ~]$ sudo chsh	# root의 shell을 변경하게된다.
Changing shell for root.
New shell [/bin/bash]: ^C 		# Ctrl + c : 실행 취소



[guest@Server-A ~]$ sudo chsh guest
Changing shell for guest.		# guest 계정의 shell을 변경
New shell [/bin/bash]:


[guest@Server-A ~]$ cat  /etc/passwd | grep guest
guest:x:1000:1000:guest:/home/guest:/bin/sh		# 로그인시 적용된는 기본 쉘이 bash이 아니라 sh로 변경


[guest@Server-A ~]$ echo $SHELL
/bin/bash						# 로그인시 bash이 적용되었기때문에 그대로 bash이 사용된다.
```

**다른 세션으로 guest 계정 접속**

```
login as: guest
guest@192.168.10.100's password:
Activate the web console with: systemctl enable --now cockpit.socket

Last login: Tue Jul 21 16:16:46 2026 from 192.168.10.1
[guest@Server-A ~]$
[guest@Server-A ~]$
[guest@Server-A ~]$ echo $SHELL
/bin/sh						# sh이 적용
```

**guest 계정의 기본 쉘을 bash로 변경**

```
[guest@Server-A ~]$ sudo chsh guest
Changing shell for guest.
New shell [/bin/sh]: /bin/bash
Shell changed.


[guest@Server-A ~]$ cat  /etc/passwd | grep guest
guest:x:1000:1000:guest:/home/guest:/bin/bash		# 기본 적용 쉘이 bash로 변경



[guest@Server-A ~]$ echo $SHELL
/bin/sh


[guest@Server-A ~]$ exec /bin/bash			# 바로 적용


[guest@Server-A ~]$ echo $SHELL
/bin/bash						# 적용되는 쉘이 바로 적용된다.
```

**정리**: `/bin`은 실제 실행 파일, `/etc/passwd`의 쉘 항목은 로그인 시 적용되는 기본 쉘이며, `chsh`로 변경 후에는 재로그인해야 반영된다.

## Bash shell 변수

**변수**
- 변수는 어떤 값을 저장하고 재사용하기 위한 이름
- Bash에서 변수는 값을 담는 상자라고 생각하면 된다.
- 문자열, 숫자, 명령어 결과 등 다양한 데이터 저장 가능
- 형식 : `name=value`
- EX) `name="hong"`
- EX) `age=20`
- EX) `echo $name`
- EX) `echo $age`

**echo**는 문자열이나 변수의 값을 화면에 출력하는 명령어이다.
- 형식 : `echo 출력할_내용` , `echo $변수명`

**Bash 변수의 특징**
- =앞 뒤에 공백이 있으면 안된다.
  - `name="kim"`	: 정상
  - `name = "kim"`	: 오류
- 변수명에도 공백이 없어야 한다.
  - `my_name="kim"`	: 정상
  - `my name="kim"`	: 오류
- 기본적으로 모든 값을 문자열(String)로 취급한다.
  - `age=10`
  - `echo $age`
  - 변수안의 값을 계산하려면 `expr`, `let` , `(())` 구문을 사용해야 한다.

**변수명 사용 조건**

- 변수명은 문자(a-z,A-Z), 숫자(0-9), 언더바(_)만 사용할 수 있다.
- 변수명의 첫글자는 문자(a-z,A-Z) 또는 언더바(_)만 가능하다. (숫자로 시작할 수 없다.)
  - `abc1`	: O
  - `_name`	: O
  - `3abc`	: X
- 변수명에도 공백이 없어야 한다.
  - `my_name="kim"`	: 정상
  - `my name="kim"`	: 오류
- 변수명에는 특수문자 사용 불가
  - `!` , `@` , `$` , `%`등 사용 X
  - `my-name`	: X
  - `my$name`	: X
- Bash는 기본적으로 대/소문자를 구분한다.
  - NAME과 name은 서로 다른 변수이다.
- 예약어는 변수명으로 사용할 수 없다.
  - if, then, fi, for, do, done, case...

**정리**: Bash 변수는 `name=value` 형식으로 선언하며 공백·특수문자·예약어를 피해야 하고, 기본적으로 모든 값이 문자열로 취급된다.

## 변수 실습 예제 (EX1~EX14)

EX1) 변수 a 에 hello, 변수 b 에 world 를 저장하고두 값을 공백 하나를 포함하여 한 줄에 출력하시오

```
[root@Server-A ~]# a=hello
[root@Server-A ~]# b=world


[root@Server-A ~]# echo $a $b
hello world
```

EX2) 변수 num1=10, num2=20 을 선언하고 두 값을 더한 결과를 출력하시오

```
[root@Server-A ~]# num1=10
[root@Server-A ~]# num2=20

[root@Server-A ~]# echo $num1+$num2
10+20
```

- 변수는 기본적으로 모든값을 문자열로 저장하기때문에 30이 아니라 10+20이 출력된다.

```
[root@Server-A ~]# echo $((num1 + num2))
30
```

EX3) 변수 x="Linux", y="Server" 를 만들고 Linux-Server 형태로 출력하시오

```
[root@Server-A ~]# x="Linux"
[root@Server-A ~]# y="Server"


[root@Server-A ~]# echo $x-$y
Linux-Server


x=Linux
y="Linux"
```

- 위와같이 단순 문자열 값만사용시 결과가 같지만 특수문자를 사용시에는 서로 다른 의미로 동작한다

EX4) 현재 날짜(date 명령)의 결과를 변수 today 에 저장하고 출력하시오

```
[root@Server-A ~]# date		# 현재 시간을 출력
2026. 07. 21. (화) 17:03:57 KST



[root@Server-A ~]# cal		# 현재의 달력을 출력
      7월 2026
일 월 화 수 목 금 토
            1  2  3  4
 5  6  7  8  9 10 11
12 13 14 15 16 17 18
19 20 21 22 23 24 25
26 27 28 29 30 31



[root@Server-A ~]# cal 8 2026	# 2026년 8월 달력 출력
      8월 2026
일 월 화 수 목 금 토
                        1
 2  3  4  5   6  7   8
 9 10 11 12 13 14 15
16 17 18 19 20 21 22
23 24 25 26 27 28 29
30 31



[root@Server-A ~]# today=date	# date문자열을 today변수에 저장

[root@Server-A ~]# echo $today
date



[root@Server-A ~]# today=$(date)	# $() 치환


root@Server-A ~]# echo $today
2026. 07. 21. (화) 17:08:26 KST




[root@Server-A ~]# to_cal=$(cal)


[root@Server-A ~]# echo $to_cal
7월 2026 일 월 화 수 목 금 토 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29 30 31
```

EX5) 변수 name="guest" 를 만들고 Hello, guest 라고 출력하시오

```
[root@Server-A ~]# name=guest


[root@Server-A ~]# echo Hello, $name
Hello, guest


[root@Server-A ~]# echo "Hello, $name"	# 공백이 있을경우 겹따옴표 사용을 권장한다.
Hello, guest
```

- 단순 텍스트  출력은 결과 같지만 예약된 특수문자가 사용될 경우 결과가 달라질수 있다.
  값에 공백이 있으면 겹따옴표 사용하는것을 권장한다.

EX6) 변수 user1="kim", user2="lee" 를 만들고 kimlee 형태로 붙여서 출력하시오

EX7) 변수 arr="A B C D" 를 만들고 한 줄에 그대로 출력하시오

```
[root@Server-A ~]# arr=A B C D
bash: B: 명령을 찾을 수 없습니다...
```

- arr변수에 A를 저장하고 B라는 명령어를 실행시 인자 C와 D를 전달

```
[root@Server-A ~]# arr="A B C D"	# 값에 공백이 있는경우 겹따옴표 사용해야 한다.


[root@Server-A ~]# echo $arr
A B C D
```

EX8) 변수에 저장된 값을 다른 변수로 복사하기
 - 변수 a에 "Rocky"를 저장하고, 변수 b에 a의 값을 그대로 복사한 뒤 두 변수를 각각 출력하시오.

```
[root@Server-A ~]# a=Rocky
[root@Server-A ~]# b=$a


[root@Server-A ~]# echo $a
Rocky


[root@Server-A ~]# echo $b
Rocky


[root@Server-A ~]# echo $a $b
Rocky Rocky
```

EX9) 변수의 값을 변경(덮어쓰기)하기
 - 변수 msg에 "Hello"를 저장한 후 다시 "Welcome"으로 값을 바꿔서 출력하시오.

```
[root@Server-A ~]# msg=Hello
[root@Server-A ~]# msg=Welcome

[root@Server-A ~]# echo $msg
Welcome
```

EX10) 변수 lang="apple" 위 변수를 사용하여 "I like apple" 문장을 출력하시오

EX11) 변수에 명령 결과 개수 저장
 - ls -l의 명령 결과(line수)를 cnt 변수에 저장

**wc**는 Word Count의 약자로, 파일이나 입력 데이터의 줄 수, 단어 수, 문자 수, 바이트 수를 계산하는 명령어이다.
- 설정 방법 : `wc [옵션] 파일명`

```
[root@Server-A ~]# ls  -l
합계 4
-rw-------. 1 root root 1027  7월  2 12:55 anaconda-ks.cfg
drwxr-xr-x. 2 root root      6  7월  2 14:46 공개
drwxr-xr-x. 2 root root      6  7월  2 14:46 다운로드
drwxr-xr-x. 2 root root      6  7월  2 14:46 문서
drwxr-xr-x. 2 root root      6  7월  2 14:46 바탕화면
drwxr-xr-x. 2 root root      6  7월  2 14:46 비디오
drwxr-xr-x. 2 root root      6  7월  2 14:46 사진
drwxr-xr-x. 2 root root      6  7월  2 14:46 서식
drwxr-xr-x. 2 root root      6  7월  2 14:46 음악



[root@Server-A ~]# ls  -l | wc
     10      83     492			# 줄수, 단어수, 바이트 수

# wc -l	: 줄 수 (lines)
# wc -w	: 단어 수 (words)
# wc -c	: 바이트 수 (bytes)
# wc -m	: 문자 수 (characters)

[root@Server-A ~]# ls  -l | wc -l
10

[root@Server-A ~]# ls  -l | wc -w
83

[root@Server-A ~]# ls  -l | wc -c
492

[root@Server-A ~]# ls  -l | wc -m
428



[root@Server-A ~]# cnt=$(ls  -l  | wc -l)


[root@Server-A ~]# echo $cnt
10
```

EX12) 변수를 사용한 디렉터리 및 파일 생성
 - 변수 dir에 /temp/testdir 경로를 저장해야 한다.
 - 변수 dir을 사용하여 /temp/testdir 디렉터리를 생성해야 한다.
 - /temp/testdir 디렉터리 안에 test.txt 파일을 생성해야 한다.
 - 설정 완료 후 디렉터리와 파일이 정상적으로 생성되었는지 확인해야 한다.

```
[root@Server-A ~]# dir="/temp/testdir"

[root@Server-A ~]# echo $dir
/temp/testdir



[root@Server-A ~]# mkdir  -p  $dir

[root@Server-A ~]# ls  -l  /temp
합계 0
drwxr-xr-x 2 root root 6  7월 21 18:01 testdir



[root@Server-A ~]# touch "$dir/test.txt"

[root@Server-A ~]# ls  -l  /temp/testdir/
합계 0
-rw-r--r-- 1 root root 0  7월 21 18:03 test.txt
[root@Server-A ~]#
```

EX13) 변수 txt=" hello world " 앞뒤 공백을 포함해 그대로 출력하시오.

EX14)  변수 info에 다음 두 줄을 저장하고 출력하시오

## Shell 환경 변수

**환경 변수(environment variable)**는 셸과 그 셸에서 실행되는 프로그램 전체에 영향을 주는 설정 값이다.

- 로그인한 사용자, 홈 디렉터리, 명령어 검색 경로, 사용 언어, 프롬프트 모양 등 여러 가지 환경 정보를 담고 있다.

**환경 변수가 필요한 이유**
- 환경 변수는 셸과 프로그램이 공통으로 사용할 설정값을 전달하기 위해 필요하다.
- 프로그램마다 같은 정보를 각각 설정하지 않고, 운영체제나 셸이 제공하는 값을 함께 사용할 수 있다.
- 주로 다음과 같은 정보를 전달한다.
  - 명령어를 찾을 경로
  - 사용자의 홈 디렉터리
  - 사용 언어와 문자 인코딩
  - 현재 사용자 정보
  - 프로그램 실행에 필요한 설정값
- 환경 변수는 부모 셸에서 실행된 자식 프로그램에게 전달되므로, 여러 프로그램이 동일한 환경 설정을 사용할 수 있다.

**특징**
- 현재 셸과 그 셸에서 실행하는 자식 프로세스(하위 프로그램)에게 자동으로 전달된다.
- 환경 변수 이름은 대문자를 사용하는 경우가 많다(PATH, HOME, USER 등)

**일반 변수 vs 환경 변수**

**일반 변수(로컬 변수)**
- 현재 셸 안에서만 유효한 변수
- 자식 프로세스(새로 실행한 프로그램)에는 전달되지 않는다.
- EX) `name=lee`
- EX) `echo $name`

**환경 변수**
- export로 환경으로 올려놓은 변수
- 현재 셸뿐 아니라, 이 셸에서 실행하는 프로그램에게도 그대로 전달된다.
- EX) `NAME=lee`
- EX) `export NAME` 	: 환경 변수로 승격
- EX) `bash`		: 새로운 셸 실행
- EX) `echo $NAME`		: 여기서도 lee 출력

**PATH**
- 명령어 탐색 경로
- 사용자가 명령어를 입력했을 때, 셸이 어느 디렉터리에서 실행 파일을 찾을지 알려준다.
- EX) `echo $PATH`

- 출력 예시 = `/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin`
- 설명 = `:` 로 구분된 여러 디렉터리 목록
- ls 를 치면, 셸이 이 경로들을 순서대로 찾아가며 /bin/ls , /usr/bin/ls 를 찾는다.

**HOME**
- 로그인한 사용자의 홈 디렉터리 경로
- cd 를 단독으로 입력하면, HOME 으로 이동한다.
- EX) `echo $HOME`		(HOME 으로 이동)

**USER**
- 현재 로그인한 사용자 이름
- EX) `echo $USER`

**SHELL**
- 로그인 시 사용되는 기본 쉘의 경로
- EX) `echo $SHELL`

**LANG / LC_***
- 시스템 기본 언어, 문자 인코딩 설정
- 한글/영문 메시지, 정렬 순서, 날짜 형식 등에 영향
- EX) `echo $LANG`

```
[root@Server-A ~]# date		# 현재의 날짜와 시간을 출력
2026. 07. 22. (수) 09:58:45 KST


[root@Server-A ~]# date +%"Y"	# 연도를 4자리로 출력
2026


[root@Server-A ~]# date +%"y"	# 연도를 2자리로 출력
26


[root@Server-A ~]# date +%"m"	# 월을 출력
07


[root@Server-A ~]# date +%"d"	# 일을 출력
22


[root@Server-A ~]# date +%"Y-%m-%d"	# 년-월-일 형식으로 출력
2026-07-22


[root@Server-A ~]# date +%"Y/%m/%d"	# 년/월/일 형식으로 출력
2026/07/22


[root@Server-A ~]# date +%"H"		# 시간 출력 (24시)
10


[root@Server-A ~]# date +%"I"		# 시간 출력 (12시)
10


[root@Server-A ~]# date +"%p %I"		# 오전 오후 시간 출력 (12시)
오전 10


[root@Server-A ~]# date +%"M"		# 분
08


[root@Server-A ~]# date +%"S"		# 초
26


[root@Server-A ~]# date +"%H:%M:%S"
10:11:15


[root@Server-A ~]# date +"%p %I:%M:%S"
오전 10:11:37


[root@Server-A ~]# date +"%p %I시 %M분 %S초"
오전 10시 12분 35초


[root@Server-A ~]# date +%F	# 날짜만 출력
2026-07-22


[root@Server-A ~]# date +%T	# 시간만 출력
10:13:32


[root@Server-A ~]# date +"%F  %T"
2026-07-22  10:14:36



[root@Server-A ~]# echo $PATH	# 명령어 입력시 찾을 경로가 저장되어 있다.
/root/.local/bin:/root/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin



[root@Server-A ~]# PATH=/temp	# PAHT 환경변수의 값을 변경



[root@Server-A ~]# echo $PATH	# PAHT 환경변수의 경로 확인
/temp



[root@Server-A ~]# ls  -l
bash: ls: 명령을 찾을 수 없습니다...
이 파일이 포함된 패키지:
'coreutils'
'coreutils-single'


[root@Server-A ~]# date
bash: date: 명령을 찾을 수 없습니다...
이 파일이 포함된 패키지:
'coreutils'
'coreutils-single'



[root@Server-A ~]# cp  /usr/bin/*  /temp
bash: cp: 명령을 찾을 수 없습니다...
이 파일이 포함된 패키지:
'coreutils'
'coreutils-single'



	# 원상복구 방법 1
[root@Server-A ~]# PATH=/root/.local/bin:/root/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin  export PATH


	# 원상복구 방법 2
[root@Server-A ~]# export PATH=/root/.local/bin:/root/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin


[root@Server-A ~]# date
2026. 07. 22. (수) 10:35:53 KST


[root@Server-A ~]# ls  -l
합계 4
-rw-------. 1 root root 1027  7월  2 12:55 anaconda-ks.cfg
drwxr-xr-x. 2 root root      6  7월  2 14:46 공개
drwxr-xr-x. 2 root root      6  7월  2 14:46 다운로드
drwxr-xr-x. 2 root root      6  7월  2 14:46 문서
drwxr-xr-x. 2 root root      6  7월  2 14:46 바탕화면
drwxr-xr-x. 2 root root      6  7월  2 14:46 비디오
drwxr-xr-x. 2 root root      6  7월  2 14:46 사진
drwxr-xr-x. 2 root root      6  7월  2 14:46 서식
drwxr-xr-x. 2 root root      6  7월  2 14:46 음악
```

**명령어 경로 확인**

```
[root@Server-A ~]# ls  -l  /usr/bin/cp
-rwxr-xr-x. 1 root root 144400  6월 25 02:37 /usr/bin/cp


[root@Server-A ~]# ls  -l  /usr/bin/date
-rwxr-xr-x. 1 root root 106336  6월 25 02:37 /usr/bin/date


[root@Server-A ~]# ls  -l  /usr/bin/rm
-rwxr-xr-x. 1 root root 61432  6월 25 02:37 /usr/bin/rm


[root@Server-A ~]# ls  -l  /usr/bin/mkdir
-rwxr-xr-x. 1 root root 69752  6월 25 02:37 /usr/bin/mkdir


[root@Server-A ~]# ls  -l  /usr/bin/scp
-rwxr-xr-x. 1 root root 136104  5월 20 09:49 /usr/bin/scp




[root@Server-A ~]# cp  -pr  /usr/bin/*  /temp	# 명령어가 있는 /usr/bin/안의 모든 파일 및 디렉터리를 /temp 디렉터리로 복사


[root@Server-A ~]# PATH=/temp		# PATH의 경로를 /temp 디렉터리로 변경


[root@Server-A ~]# ls -l			# 명령어를 사용할 수 있다.
합계 4
-rw-------. 1 root root 1027  7월  2 12:55 anaconda-ks.cfg
drwxr-xr-x. 2 root root    6  7월  2 14:46 공개
drwxr-xr-x. 2 root root    6  7월  2 14:46 다운로드
drwxr-xr-x. 2 root root    6  7월  2 14:46 문서
drwxr-xr-x. 2 root root    6  7월  2 14:46 바탕화면
drwxr-xr-x. 2 root root    6  7월  2 14:46 비디오
drwxr-xr-x. 2 root root    6  7월  2 14:46 사진
drwxr-xr-x. 2 root root    6  7월  2 14:46 서식
drwxr-xr-x. 2 root root    6  7월  2 14:46 음악


[root@Server-A ~]# date			# 명령어를 사용할 수 있다.
2026. 07. 22. (수) 10:39:44 KST
```

**다시 PATH의 경로를 원상복구**

```
[root@Server-A ~]# export PATH=/root/.local/bin:/root/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin


[root@Server-A ~]# echo $PATH
/root/.local/bin:/root/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin
```

**홈 디렉터리 변경**

```
[root@Server-A ~]# cd  /etc/ssh

[root@Server-A ssh]# pwd
/etc/ssh



[root@Server-A ssh]# cd		# 홈 디렉터리로 이동

[root@Server-A ~]# pwd
/root



[root@Server-A ~]# HOME=/etc/ssh
[root@Server-A root]#		# /root 에있지만 홈 디렉터리가 아닌것으로 출력된다.



[root@Server-A root]# cd		# 홈디렉터리로 이동

[root@Server-A ~]# pwd		# /etc/ssh가 홈 디렉터리로 인식
/etc/ssh




[root@Server-A ~]# export HOME=/root	# 홈 디렉터리를 /root로 변경
[root@Server-A ssh]#


[root@Server-A ssh]# cd			# 홈 디렉터리로 이동

[root@Server-A ~]# pwd
/root


[root@Server-A ~]# env			# 전체 환경변수 확인

[root@Server-A ~]# env | grep HOME
HOME=/root


[root@Server-A ~]# env | grep PATH
DEBUGINFOD_IMA_CERT_PATH=/etc/keys/ima:
PATH=/root/.local/bin:/root/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin
```

**정리**: PATH는 명령어 탐색 경로, HOME은 홈 디렉터리를 결정하는 환경변수이며, 실수로 값을 바꾸면 명령어를 찾지 못하거나 홈 위치가 바뀌므로 원래 값으로 복구하는 절차(export로 재설정)를 알아두어야 한다.

## 환경변수 실습 예제 (EX1~EX9)

EX1) HOME 환경 변수의 값을 출력하시오

```
[root@Server-A ~]# echo $HOME
/root


[guest@Server-A ~]$ echo $HOME
/home/guest
```

EX2) 환경 변수 PATH 값을 출력하시오

```
[root@Server-A ~]# echo $PATH
/root/.local/bin:/root/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin
```

EX3) 환경 변수 NAME 을 lee 로 설정하고, env 명령으로 NAME 이 표시되도록 하시오

```
[root@Server-A ~]# NAME=lee


[root@Server-A ~]# env | grep NAME		# 환경변수로 확인되지 않는다.
HOSTNAME=Server-A
LOGNAME=root


	# 방법 1
[root@Server-A ~]# NAME=lee		# NAME 변수에 값을 대입
[root@Server-A ~]# export NAME		# NAME 변수를 환경변수로 변경

[root@Server-A ~]# env | grep NAME
HOSTNAME=Server-A
NAME=lee
LOGNAME=root


	# 방법 2
[root@Server-A ~]# export NAME=lee		# 환경변수에 값을 대입

[root@Server-A ~]# env | grep NAME
HOSTNAME=Server-A
NAME=lee
LOGNAME=root
```

EX4) 환경 변수 CITY 에 seoul 을 저장하고 echo 로 출력하시오

```
[root@Server-A ~]# export CITY=seoul


[root@Server-A ~]# echo $CITY
seoul


[root@Server-A ~]# env | grep CITY
CITY=seoul
```

EX5) 환경 변수 LANG 값을 출력하시오

```
[root@Server-A ~]# echo $LANG
ko_KR.UTF-8
```

EX6-1) 현재 셸과 로그인 셸 확인 및 일반 변수·환경 변수 비교
 - 현재 셸과 로그인 셸 확인
 - 현재 사용 중인 셸이 Bash인지 확인해야 한다.
 - 현재 Bash에서 tcsh를 실행해야 한다.
 - tcsh에서 현재 사용 중인 셸을 확인해야 한다.
 - 로그인할 때 사용한 기본 셸이 무엇인지 확인해야 한다.
 - tcsh를 종료한 후 다시 Bash로 돌아와야 한다.

```
[root@Server-A ~]# echo $SHELL	# 로그인시 적용된 쉘
/bin/bash


[root@Server-A ~]# echo $0		# 현재 사용되는 쉘
-bash


[root@Server-A ~]# tcsh		# tcsh로 변경 (bash쉘 하위에 자신 쉘인 tcsh이 동작한다.)


[root@Server-A ~]# echo $0		# 현재 사용되는 쉘 확인
tcsh


[root@Server-A ~]# ps -f
UID          PID    PPID  C STIME TTY	TIME 	CMD
root        1879    1878   0 09:28 pts/0    	00:00:00	-bash
root        2192    1879   0 11:05 pts/0    	00:00:00	-csh
root        2225    2192   0 11:07 pts/0    	00:00:00	ps -f



[root@Server-A ~]# exit		# 다시 bash로 전환
exit


[root@Server-A ~]# echo $0		# 다시 부모 쉐인 bash이 사용되는것을 확인
-bash
```

EX6-2) 일반 변수 확인
 - Bash에서 일반 변수 SOL에 desk 값을 저장해야 한다.
 - Bash에서 SOL 변수의 값을 확인해야 한다.
 - Bash에서 tcsh를 실행해야 한다.
 - tcsh에서 일반 변수 SOL을 확인해야 한다.
 - tcsh를 종료한 후 Bash로 돌아와야 한다.

```
[root@Server-A ~]# SOL=desk	# SOL 변수에 값 desk를 대입


[root@Server-A ~]# echo $SOL	# 현재 쉘에서 확인 및 사용 가능
desk


[root@Server-A ~]# echo $SHELL	# 로그인시 기본적으로 적용되는 쉘
/bin/bash


[root@Server-A ~]# echo $0		# 현재 사용중인 쉘
-bash




[root@Server-A ~]# tcsh		# Bash 하위에 자식 쉘 생성 (tcsh)


[root@Server-A ~]# echo $SHELL	# 로그인시 bash을 사용
/bin/bash


[root@Server-A ~]# echo $0		# 현재 tcsh을 사용
tcsh


[root@Server-A ~]# echo $SOL	# 일반 변수는 하위 쉘에서 사용할 수 없다.
SOL: Undefined variable.



[root@Server-A ~]# exit		# tcsh에서 부모 쉘인 bash로 전환
exit


[root@Server-A ~]# echo $SOL
desk
```

EX6-3) 환경 변수 확인
 - Bash에서 SOL 변수에 desk 값을 저장하고 환경 변수로 설정해야 한다.
 - Bash에서 tcsh를 실행해야 한다.
 - tcsh에서 환경 변수 SOL의 값이 출력되는지 확인해야 한다.
 - 환경 변수는 자식 셸에 전달된다는 것을 확인해야 한다.
 - tcsh를 종료한 후 Bash로 돌아와야 한다.

```
[root@Server-A ~]# export SOL=desk		# 환경 변수 SOL 생성


[root@Server-A ~]# echo $SOL
desk


[root@Server-A ~]# echo $0
-bash


[root@Server-A ~]# tcsh			# tcsh로 변경

[root@Server-A ~]# echo $0
tcsh

[root@Server-A ~]# echo $SOL		# bash의 하위 쉘인 tcsh 에서 환경변수 SOL을 사용할 수 있다.
desk


[root@Server-A ~]# exit
```

EX7) EX6에서 만든 SOL 환경 변수를 삭제하시오

```
[root@Server-A ~]# echo $SOL
desk


[root@Server-A ~]# unset SOL



[root@Server-A ~]# echo $SOL
```

EX8) 환경 변수 PATH2 에 "my value" 라는 공백이 있는 값을 추가한 뒤 출력하시오

```
[root@Server-A ~]# export PATH2="my value"


[root@Server-A ~]# echo $PATH2
my value


[root@Server-A ~]# env | grep PATH2
PATH2=my value
```

EX9-1) 프롬프트 변수 PS1 을 "[TEST] " 로 변경하시오

```
[root@Server-A ~]# echo $PS1
[\u@\h \W]\$
```

 - `[` 	: [ 문자를 출력
 - `\u` 	: 현재 사용자 이름
 - `@` 	: @ 문자를 출력
 - `\h` 	: 호스트 이름의 첫 번째 부분
 - `\W`      	: 현재 작업 디렉터리의 마지막 이름
 - `]` 	: ] 문자를 출력
 - `\$`	: root는 #, 일반 사용자는 $ 출력

```
[root@Server-A ~]# PS1='[TEST] '
[TEST]
[TEST]
```

9-2) 현재 Bash 셸의 프롬프트가 다음과 같이 출력되도록 설정
 - 출력 형식 : Server-A:~ #

```
[TEST] PS1='\h:\W \$ '
Server-A:~ #
Server-A:~ #
```

 - `[` 	: [ 문자를 출력
 - `\u` 	: 현재 사용자 이름
 - `@` 	: @ 문자를 출력
 - `\h` 	: 호스트 이름의 첫 번째 부분
 - `\W`      	: 현재 작업 디렉터리의 마지막 이름
 - `\w`      	: 현재 작업 디렉터리의 전체 이름
 - `]` 	: ] 문자를 출력
 - `\$`	: root는 #, 일반 사용자는 $ 출력
 - `\t`	: 현재 시간을 HH:MMSS 형식으로 출력
 - `\d`	: 현재 날짜를 출력

9-3) 현재 Bash 셸의 프롬프트 앞에 현재 시간을 추가하시오.
 - 출력 형식 : [11:30:25 root@Server-A ~]#

```
Server-A:~ # PS1='[\t \u@\h \W]\$ '
[11:58:30 root@Server-A ~]#




11:59:06 root@Server-A ~]# PS1='[\t \u@\h \w]\$ '				# \w로 설정하게되면 경로 이동시 모든 경로가 출력된다.

[12:00:04 root@Server-A ~]# cd  /etc/NetworkManager/system-connections/
[12:00:11 root@Server-A /etc/NetworkManager/system-connections]#			# 전체 경로 확인
```

9-4) PS1 환경변수의 값을 원래의 값으로 변경

```
	# 방법 1
11:59:06 root@Server-A ~]# PS1='[\u@\h \W]\$ '

[root@Server-A ~]#
[root@Server-A ~]#




	# 방법 2
[12:08:38 root@Server-A ~]# vi .bashrc
~~~~~~~ 중간 생략 ~~~~~~~
     17
     18 # User specific aliases and functions
     19
     20 alias rm='rm -i'
     21 alias cp='cp -i'
     22 alias mv='mv -i'
     23 PS1='[\u@\h \W]\$ '	# PS1값을 .bashrc 파일에 저장

:wq


[12:10:46 root@Server-A ~]# source  ~/.bashrc
[root@Server-A ~]#
```

**정리**: PS1은 프롬프트 모양을 결정하는 환경변수이며, `\u`, `\h`, `\W`, `\t` 등의 이스케이프 시퀀스를 조합해 원하는 형태로 커스터마이징할 수 있고, `.bashrc`에 저장하면 영구적으로 적용된다.

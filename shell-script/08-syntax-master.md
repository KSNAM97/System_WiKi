# Shell Script 문법 총정리 (Syntax Master)

## Shebang과 실행 방식

**Shebang(셔뱅)**은 스크립트 파일의 맨 첫 줄에 어떤 인터프리터로 이 파일을 실행할지 지정하는 문법이다.

- 형식 : `#!경로`
- 반드시 파일의 **첫 줄, 첫 컬럼**에 있어야 한다. (앞에 공백/빈 줄이 있으면 무시된다.)

```bash
#!/bin/bash		# bash로 실행
#!/bin/sh		# sh로 실행
#!/usr/bin/env bash	# PATH에서 bash를 찾아 실행 (환경에 따라 경로가 다를 때 사용)
```

- Shebang을 생략해도 `bash  ./script.sh`처럼 직접 인터프리터를 지정해서 실행하면 동작하지만, `./script.sh`처럼 파일 자체를 실행하려면 Shebang과 실행 권한이 모두 필요하다.

```bash
[root@Server-A ~]# vi ./script/shebang_test.sh
#!/bin/bash
echo "현재 사용중인 쉘 : $0"

:wq


[root@Server-A ~]# ./script/shebang_test.sh		# 실행 권한이 없으면 거부됨
bash: ./script/shebang_test.sh: 허가 거부됨


[root@Server-A ~]# chmod +x ./script/shebang_test.sh

[root@Server-A ~]# ./script/shebang_test.sh		# Shebang에 지정된 인터프리터로 실행
현재 사용중인 쉘 : ./script/shebang_test.sh


[root@Server-A ~]# sh ./script/shebang_test.sh	# Shebang을 무시하고 sh로 강제 실행도 가능
현재 사용중인 쉘 : ./script/shebang_test.sh
```

**정리**: Shebang은 스크립트를 실행할 인터프리터를 지정하는 첫 줄이며, `./파일명`으로 직접 실행하려면 Shebang과 실행 권한(`chmod +x`)이 함께 필요하다.

## 변수 타입 지정 (declare)

**declare**는 변수를 선언할 때 타입(읽기전용, 정수, 배열 등)을 지정하는 명령어이다. `typeset`도 동일하게 동작한다.

| 옵션 | 의미 |
|---|---|
| `-r` | 읽기 전용(readonly) 변수로 선언, 이후 값 변경 불가 |
| `-i` | 정수(integer) 변수로 선언, 대입 시 자동으로 산술 연산 수행 |
| `-a` | 인덱스 배열로 선언 |
| `-A` | 연관 배열(associative array)로 선언 |
| `-f` | 함수 목록을 의미 (인자 없이 `declare -f`로 정의된 함수 확인) |
| `-x` | 환경 변수로 선언(= export와 동일한 효과) |

```bash
[root@Server-A ~]# declare -r  version="1.0"

[root@Server-A ~]# echo $version
1.0

[root@Server-A ~]# version="2.0"		# 읽기 전용이므로 변경 시도시 오류
bash: version: 읽기 전용 변수입니다.



[root@Server-A ~]# declare -i  num=10

[root@Server-A ~]# num="5 + 3"		# -i 옵션이 있으면 문자열도 산술식으로 처리
[root@Server-A ~]# echo $num
8



[root@Server-A ~]# declare -a  arr=("a" "b" "c")

[root@Server-A ~]# echo ${arr[@]}
a b c



[root@Server-A ~]# declare -x  MY_VAR="hello"	# export MY_VAR="hello" 와 동일

[root@Server-A ~]# env | grep MY_VAR
MY_VAR=hello


[root@Server-A ~]# declare -f			# 현재 쉘에 정의된 함수 목록/본문 확인
```

**정리**: declare는 변수의 성격(읽기전용/정수/배열/연관배열/환경변수)을 명시적으로 선언하는 명령이며, 특히 `-i`(정수 자동연산)와 `-r`(읽기전용 상수)은 실수를 줄이는 데 유용하다.

## 예약 변수 (Reserved Variables)

Bash가 내부적으로 미리 정의해 두고 자동으로 값을 채워주는 특수 변수들이다.

| 변수 | 의미 |
|---|---|
| `$0` | 현재 실행중인 스크립트(또는 쉘) 이름 |
| `$1 ~ $9` | 스크립트/함수에 전달된 위치 매개변수 |
| `$#` | 전달된 인자의 개수 |
| `$*` | 전달된 인자 전체를 하나의 문자열로 |
| `$@` | 전달된 인자 각각을 개별 값으로 |
| `$$` | 현재 실행중인 프로세스(쉘)의 PID |
| `$!` | 가장 최근에 백그라운드로 실행한 프로세스의 PID |
| `$?` | 바로 직전에 실행한 명령의 종료 상태 코드 |
| `HOME` | 로그인 사용자의 홈 디렉터리 |
| `PATH` | 명령어 탐색 경로 |
| `LANG` | 시스템 기본 언어/인코딩 |
| `UID` | 현재 사용자의 UID(숫자) |
| `SHELL` | 로그인시 사용되는 기본 쉘 경로 |
| `USER` | 현재 로그인한 사용자 이름 |
| `TERM` | 현재 터미널의 종류 |
| `FUNCNAME` | 현재 실행 중인 함수의 이름 (함수 안에서만 유효) |

```bash
[root@Server-A ~]# vi ./script/reserved_var_test.sh
#!/bin/bash

my_func () {
    echo "함수 이름 : ${FUNCNAME[0]}"
}

echo "스크립트 PID : $$"
echo "UID : $UID"
echo "TERM : $TERM"

sleep  30  &				# 백그라운드 실행
echo "방금 백그라운드로 실행한 PID : $!"

my_func

:wq


[root@Server-A ~]# chmod +x ./script/reserved_var_test.sh

[root@Server-A ~]# ./script/reserved_var_test.sh
스크립트 PID : 5231
UID : 0
TERM : xterm
방금 백그라운드로 실행한 PID : 5232
함수 이름 : my_func
```

**정리**: `$0~$9, $#, $*, $@`는 인자 관련, `$$/$!`는 프로세스 관련, `$?`는 종료 상태 관련 예약 변수이며, `HOME/PATH/UID/SHELL/USER/TERM/FUNCNAME` 등은 시스템/실행 환경 정보를 담고 있는 예약 변수이다.

## 변수 관련 명령어 (set · env · export · unset)

| 명령어 | 의미 |
|---|---|
| `set` | 현재 쉘의 모든 변수(지역+환경) 및 쉘 옵션을 확인/설정 |
| `env` | 현재 쉘의 환경 변수만 확인, 또는 환경변수를 지정해 명령 실행 |
| `export` | 일반(지역) 변수를 환경 변수로 승격 |
| `unset` | 변수나 함수를 제거 |

```bash
[root@Server-A ~]# name=guest

[root@Server-A ~]# set | grep ^name=	# set은 지역 변수도 확인 가능
name=guest

[root@Server-A ~]# env | grep name	# env는 지역 변수를 확인할 수 없다.
(출력 없음)


[root@Server-A ~]# export name		# 환경 변수로 승격

[root@Server-A ~]# env | grep name
name=guest


[root@Server-A ~]# unset name		# 변수 삭제

[root@Server-A ~]# echo $name
(출력 없음)
```

**정리**: `set`은 지역/환경 변수를 모두 포함해 확인하고, `env`는 환경 변수만 확인한다. `export`로 지역 변수를 환경 변수로 승격하고, `unset`으로 변수를 완전히 제거한다.

## 이스케이프 문자

echo나 문자열 안에서 특수한 제어 동작을 나타내는 문자로, `echo -e` 옵션과 함께 사용해야 해석된다.

| 이스케이프 | 의미 |
|---|---|
| `\n` | 줄바꿈 (new line) |
| `\t` | 탭(tab) |
| `\r` | 캐리지 리턴 (커서를 줄 맨 앞으로) |
| `\f` | 폼 피드 (다음 페이지/폼으로 이동, 터미널에서는 줄바꿈처럼 보임) |
| `\\` | 백슬래시(\) 문자 자체 출력 |
| `\"` | 겹따옴표(") 문자 자체 출력 |

```bash
[root@Server-A ~]# echo "1번째줄\n2번째줄"		# -e 옵션 없으면 이스케이프가 그대로 문자로 출력
1번째줄\n2번째줄


[root@Server-A ~]# echo -e "1번째줄\n2번째줄"	# -e 옵션을 사용하면 이스케이프 문자가 해석됨
1번째줄
2번째줄


[root@Server-A ~]# echo -e "이름\t나이\t직업"
이름	나이	직업


[root@Server-A ~]# echo -e "경로 : C:\\Users\\guest"
경로 : C:\Users\guest
```

**정리**: 이스케이프 문자는 `echo -e` 옵션과 함께 사용해야 줄바꿈(`\n`)·탭(`\t`) 등이 실제로 해석되어 출력된다. `-e` 없이는 문자 그대로 출력된다.

## 산술 연산 정리 (expr · let · $(( )))

Bash에서 산술 연산을 수행하는 세 가지 방식을 비교 정리한다. (개별 상세 문법은 SH-02 Metacharacters 문서 참고)

| 방식 | 특징 |
|---|---|
| `expr` | 피연산자와 연산자 사이에 반드시 공백이 필요, 곱셈은 `\*`로 이스케이프해야 함, 결과를 변수에 담으려면 역따옴표나 `$()` 필요 |
| `let` | 변수에 직접 대입하는 방식, 공백 없이 사용 가능, `$`없이 변수명 사용 |
| `$(( ))` | 산술 확장(Arithmetic Expansion), 가장 널리 쓰이는 방식, C언어 스타일 연산자(`+ - * / % ++ --`) 그대로 사용 가능 |

```bash
[root@Server-A ~]# result=`expr 10 \* 2`	# expr : 공백 필수, 곱셈은 \* 이스케이프
[root@Server-A ~]# echo $result
20


[root@Server-A ~]# let  result=10*2		# let : 공백 없이 변수=식 형태로 대입
[root@Server-A ~]# echo $result
20


[root@Server-A ~]# result=$((10 * 2))		# $(( )) : 가장 일반적으로 쓰이는 방식
[root@Server-A ~]# echo $result
20


[root@Server-A ~]# num=5
[root@Server-A ~]# ((num++))			# $(( ))와 동일 계열, 증감연산에 자주 사용
[root@Server-A ~]# echo $num
6
```

**정리**: 세 방식 모두 결과는 같지만, `expr`은 공백/이스케이프 제약이 많고, `let`은 대입 전용이며, `$(( ))`가 가독성과 활용도 면에서 실무에서 가장 많이 쓰인다.

## 주석 처리

```bash
# 한 줄 주석은 # 으로 시작한다.

echo "hello"	# 명령어 뒤에 붙는 주석도 가능
```

**여러 줄(블록) 주석**

- Bash 자체에는 블록 주석 문법이 따로 없지만, **heredoc**(`:<<`)을 이용해 여러 줄을 한 번에 주석 처리하는 방법을 관용적으로 사용한다.

```bash
[root@Server-A ~]# vi ./script/comment_test.sh
#!/bin/bash

echo "실행 1"

: <<"END"
echo "이 블록은 실행되지 않는다."
echo "여러 줄을 한번에 주석 처리할 때 사용한다."
END

echo "실행 2"

:wq


[root@Server-A ~]# chmod +x ./script/comment_test.sh

[root@Server-A ~]# ./script/comment_test.sh
실행 1
실행 2
```

**정리**: 한 줄 주석은 `#`, 여러 줄을 한번에 주석 처리할 때는 `: <<"END"  ~  END` 형태의 heredoc을 관용적으로 사용한다. (vi에서는 `:10,20s/^/#/`처럼 블록 지정 후 일괄 치환하는 방법도 있다.)

## 문자열 패턴 치환 (Parameter Expansion)

변수의 값을 그대로 쓰는 것이 아니라, 값이 없을 때 기본값을 주거나 문자열 일부를 잘라내는 등 변형해서 사용하는 문법이다.

| 표현식 | 의미 |
|---|---|
| `${#var}` | 변수 값의 문자열 길이 |
| `${var:-word}` | var가 없거나 비어있으면 word를 대신 사용(var 자체는 변경 안 됨) |
| `${var:=word}` | var가 없거나 비어있으면 word를 var에 대입하고 사용 |
| `${var:+word}` | var가 값이 있으면 word를 사용(var 자체 값은 무시) |
| `${var:?message}` | var가 없거나 비어있으면 message를 출력하고 스크립트 종료 |
| `${var#pattern}` | 앞에서부터 pattern과 일치하는 **최소** 부분 제거 |
| `${var##pattern}` | 앞에서부터 pattern과 일치하는 **최대** 부분 제거 |
| `${var%pattern}` | 뒤에서부터 pattern과 일치하는 **최소** 부분 제거 |
| `${var%%pattern}` | 뒤에서부터 pattern과 일치하는 **최대** 부분 제거 |

```bash
[root@Server-A ~]# name="hong gil dong"

[root@Server-A ~]# echo ${#name}		# 문자열 길이
13



[root@Server-A ~]# unset city
[root@Server-A ~]# echo ${city:-seoul}	# city가 없으므로 seoul을 대신 출력 (city 자체는 그대로 비어있음)
seoul
[root@Server-A ~]# echo $city
(출력 없음)


[root@Server-A ~]# echo ${city:=seoul}	# city가 없으므로 seoul을 대입 후 출력
seoul
[root@Server-A ~]# echo $city
seoul


[root@Server-A ~]# echo ${city:+busan}	# city에 값이 있으므로 busan을 대신 출력
busan


[root@Server-A ~]# unset city
[root@Server-A ~]# echo ${city:?"city 값이 없습니다."}	# 값이 없으므로 메세지 출력 후 스크립트 종료
bash: city: city 값이 없습니다.



[root@Server-A ~]# path="/home/guest/backup/data.tar.gz"

[root@Server-A ~]# echo ${path#*/}		# 처음 '/' 하나까지 최소 제거
home/guest/backup/data.tar.gz

[root@Server-A ~]# echo ${path##*/}		# 마지막 '/' 까지 최대 제거 (파일명만 추출, basename과 동일 효과)
data.tar.gz

[root@Server-A ~]# echo ${path%/*}		# 마지막 '/' 부터 뒤를 최소 제거 (디렉터리 경로만 추출, dirname과 동일 효과)
/home/guest/backup

[root@Server-A ~]# echo ${path%%.*}		# 첫 '.' 부터 뒤를 최대 제거 (확장자 전체 제거)
/home/guest/backup/data
```

**정리**: `:-`, `:=`, `:+`, `:?`는 변수의 존재/공백 여부에 따른 기본값 처리, `#`/`##`/`%`/`%%`는 문자열 앞/뒤에서 패턴을 잘라내는 문법이다. 특히 `${var##*/}`(파일명 추출)와 `${var%/*}`(경로 추출)는 `basename`/`dirname` 명령 대신 자주 쓰이는 실무 패턴이다.

## 연관 배열 (Associative Array)

일반 배열이 숫자 인덱스(0,1,2...)로 값을 저장한다면, **연관 배열**은 문자열을 키(key)로 사용해 값을 저장하는 배열이다. (다른 언어의 Map, 딕셔너리와 동일한 개념)

- 연관 배열은 사용하기 전에 반드시 `declare -A`로 먼저 선언해야 한다.

```bash
[root@Server-A ~]# declare -A  user

[root@Server-A ~]# user[name]="hong"
[root@Server-A ~]# user[age]=20
[root@Server-A ~]# user[job]="developer"


[root@Server-A ~]# echo ${user[name]}
hong

[root@Server-A ~]# echo ${user[age]}
20


[root@Server-A ~]# echo ${user[@]}		# 모든 값 출력
hong 20 developer

[root@Server-A ~]# echo ${!user[@]}		# 모든 키(key) 출력
name age job


[root@Server-A ~]# for  key  in  "${!user[@]}"	# 키를 순회하며 key:value 형태로 출력
> do
>     echo "$key : ${user[$key]}"
> done
name : hong
age : 20
job : developer


[root@Server-A ~]# unset  'user[age]'		# 특정 키 삭제

[root@Server-A ~]# echo ${!user[@]}
name job
```

**정리**: 연관 배열은 `declare -A`로 먼저 선언한 뒤 `배열[키]=값` 형태로 사용하며, `${!arr[@]}`로 키 목록, `${arr[@]}`로 값 목록을 확인한다. `for key in "${!arr[@]}"` 패턴이 key-value를 함께 순회하는 가장 일반적인 방식이다.

## read로 사용자 입력받기

**read**는 사용자로부터 값을 입력받아 변수에 저장하는 명령어이다.

| 옵션 | 의미 |
|---|---|
| `-p` | 입력을 받기 전에 프롬프트 메세지 출력 |
| `-s` | 입력한 값을 화면에 표시하지 않음 (비밀번호 입력 등) |
| `-a` | 입력값을 배열로 저장 |
| `-t` | 입력 대기 시간(초) 제한 |
| (옵션 없음, 변수 여러 개) | 공백으로 구분된 입력을 각 변수에 순서대로 저장 |

```bash
[root@Server-A ~]# read -p "이름을 입력하세요 : " name
이름을 입력하세요 : hong

[root@Server-A ~]# echo $name
hong


[root@Server-A ~]# read -s -p "비밀번호 입력 : " pw
비밀번호 입력 :
[root@Server-A ~]# echo		# -s는 입력값이 화면에 보이지 않는다.
[root@Server-A ~]# echo $pw
1234


[root@Server-A ~]# read  first  last		# 공백으로 구분해 여러 변수에 한번에 저장
hong  gildong
[root@Server-A ~]# echo "성 : $first, 이름 : $last"
성 : hong, 이름 : gildong


[root@Server-A ~]# read  -t  5  -p  "5초 안에 입력하세요 : "  answer	# -t로 입력 대기시간 제한
5초 안에 입력하세요 :
[root@Server-A ~]# echo $?			# 시간 초과시 0이 아닌 값 반환
1
```

**정리**: read는 `-p`(프롬프트), `-s`(비표시, 비밀번호용), `-t`(시간제한), `-a`(배열 저장) 옵션과 함께 자주 쓰이며, 변수를 여러 개 나열하면 입력값이 공백 기준으로 각 변수에 순서대로 저장된다.

## Command Substitution (명령 치환)

**명령 치환**은 명령어의 실행 결과(출력)를 변수에 저장하거나 다른 명령의 일부로 사용하는 문법이다.

| 방식 | 특징 |
|---|---|
| `` `명령어` `` (역따옴표) | 오래된 방식, 중첩 사용 시 백슬래시 이스케이프가 필요해 가독성이 떨어짐 |
| `$(명령어)` | 최신/권장 방식, 중첩 사용이 쉽고 가독성이 좋음 |

```bash
[root@Server-A ~]# today=`date +%F`		# 역따옴표 방식
[root@Server-A ~]# echo $today
2026-08-10


[root@Server-A ~]# today=$(date +%F)		# $() 방식 (권장)
[root@Server-A ~]# echo $today
2026-08-10


[root@Server-A ~]# count=$(ls  -l  /etc  |  wc  -l)	# 파이프가 섞인 명령도 그대로 치환 가능
[root@Server-A ~]# echo "파일 개수 : $count"
파일 개수 : 211


[root@Server-A ~]# echo "현재 디렉터리 : $(pwd), 오늘 날짜 : $(date +%F)"	# 문자열 안에서 여러번 사용 가능
현재 디렉터리 : /root, 오늘 날짜 : 2026-08-10


[root@Server-A ~]# backup_name="backup_$(date +%Y%m%d).tar.gz"	# 파일명 생성에 자주 활용
[root@Server-A ~]# echo $backup_name
backup_20260810.tar.gz
```

**정리**: 명령 치환은 명령어의 실행 결과를 그 자리에서 문자열/변수처럼 사용하는 문법이다. 역따옴표(`` `` ``)도 동작하지만 중첩이 불편해, 실무에서는 `$( )` 방식을 기본으로 사용한다.

## 명령어 종료 상태 코드 (exit, $?)

- 모든 명령어와 스크립트는 실행이 끝나면 **종료 상태 코드(Exit Status)** 를 반환한다.
- **0** : 정상 종료(성공)
- **1 이상(1~255)** : 오류/실패 (구체적인 의미는 명령어/스크립트마다 다르다)
- 직전에 실행한 명령의 종료 상태는 **$?** 로 확인한다.
- 스크립트 안에서 **exit  숫자** 를 사용하면 그 숫자를 종료 상태로 반환하며 스크립트를 즉시 종료한다.

```bash
[root@Server-A ~]# ls  /etc/passwd		# 존재하는 파일 -> 성공
/etc/passwd

[root@Server-A ~]# echo $?
0


[root@Server-A ~]# ls  /etc/no_such_file	# 존재하지 않는 파일 -> 실패
ls: cannot access '/etc/no_such_file': 그런 파일이나 디렉터리가 없습니다

[root@Server-A ~]# echo $?
2



[root@Server-A ~]# vi ./script/exit_test.sh
#!/bin/bash

read -p "종료 코드를 입력하세요(0~255) : " code

echo "입력한 코드로 스크립트를 종료합니다."
exit "$code"

:wq


[root@Server-A ~]# chmod +x ./script/exit_test.sh

[root@Server-A ~]# ./script/exit_test.sh
종료 코드를 입력하세요(0~255) : 3
입력한 코드로 스크립트를 종료합니다.

[root@Server-A ~]# echo $?
3
```

**정리**: 모든 명령/스크립트는 0(성공) 또는 1~255(실패)의 종료 상태 코드를 반환하며, `$?`로 직전 실행 결과를 확인한다. 스크립트 안에서 `exit 숫자`로 임의의 상태 코드를 지정해 종료할 수 있어, 다른 스크립트나 시스템(cron 등)에서 성공/실패 여부를 판단하는 근거로 활용된다.

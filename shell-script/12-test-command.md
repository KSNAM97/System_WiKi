# Shell Script test · [ · [[ 명령 심화 (Test Command In Depth)

숫자/문자열/파일 비교 옵션(`-eq`, `-gt`, `-f`, `-d` 등) 자체의 기본 문법은 `03-conditions.md`의 "exit 코드 & test 명령"에 이미 정리되어 있다. 이 문서는 옵션 나열이 아니라 **test, `[`, `[[`가 각각 무엇이고 서로 어떻게 다른지**에 초점을 맞춘다.

## test, [ , [[ 는 서로 다른 것이다

| 구분 | 정체 | 특징 |
|---|---|---|
| `test 조건식` | 외부(또는 내장) **명령어** | 명령어이므로 인자는 공백으로 구분된 각각의 "단어" |
| `[ 조건식 ]` | `test`와 완전히 동일한 **명령어** | `[`가 명령어 이름이고, 마지막 인자로 반드시 `]`가 필요 |
| `[[ 조건식 ]]` | Bash의 **예약어(keyword)/문법** | 명령어가 아니라 파서가 직접 해석하는 문법이라 더 유연함 |

```bash
[root@Server-A ~]# type test
test는 셸 내장임

[root@Server-A ~]# type [
[는 셸 내장임

[root@Server-A ~]# type [[
[[는 예약어임		# [[는 명령어가 아니라 셸 문법 자체
```

- `test`와 `[`는 명령어이기 때문에, 다른 명령어(예: `ls`, `cp`)와 마찬가지로 **인자를 공백으로 나눠서 받는다.** 이 때문에 변수를 따옴표 없이 쓰면 값에 공백이 있을 때 인자 개수가 달라져 오류가 난다.
- `[[`는 Bash 파서가 직접 처리하는 문법이라, 그 안에서는 단어 분리·경로 확장(globbing)이 일어나지 않는다.

```bash
[root@Server-A ~]# name="hong gil dong"

[root@Server-A ~]# [ $name = "hong gil dong" ]		# 따옴표 없이 쓰면 인자가 3개로 쪼개져 오류
bash: [: 너무 많은 인자입니다.

[root@Server-A ~]# [ "$name" = "hong gil dong" ]		# 반드시 " "로 감싸야 안전
[root@Server-A ~]# echo $?
0


[root@Server-A ~]# [[ $name = "hong gil dong" ]]		# [[ ]]는 따옴표 없이도 통째로 한 값으로 처리됨
[root@Server-A ~]# echo $?
0
```

**정리**: `test`와 `[`는 완전히 같은 **명령어**이고, `[[`는 Bash **문법**이다. `[`는 명령어라서 변수를 반드시 큰따옴표로 감싸야 공백/글롭 문제를 피할 수 있지만, `[[`는 문법 차원에서 처리되므로 상대적으로 안전하다.

## test -t : 터미널 여부 검사

- `test -t FD`(또는 `[ -t FD ]`)는 지정한 파일 디스크립터(FD)가 **터미널에 연결되어 있는지**를 검사한다. FD를 생략하면 기본값은 1(표준출력)이다.
- 스크립트가 사람이 터미널에서 직접 실행한 것인지, 아니면 파이프나 리다이렉션으로 다른 프로그램에 연결되어 실행된 것인지 구분할 때 사용한다. 색상 코드(`\033[...]`)를 출력할지 말지 결정하는 용도로 자주 쓰인다.

```bash
[root@Server-A ~]# vi ./script/tty_check.sh
#!/bin/bash

if [ -t 1 ]
then
    echo "표준출력이 터미널에 연결되어 있습니다."
else
    echo "표준출력이 파일/파이프로 리다이렉션되어 있습니다."
fi

:wq


[root@Server-A ~]# chmod +x ./script/tty_check.sh

[root@Server-A ~]# ./script/tty_check.sh		# 터미널에서 직접 실행
표준출력이 터미널에 연결되어 있습니다.

[root@Server-A ~]# ./script/tty_check.sh | cat		# 파이프로 연결하면 결과가 달라짐
표준출력이 파일/파이프로 리다이렉션되어 있습니다.

[root@Server-A ~]# ./script/tty_check.sh > /tmp/out.txt	# 파일로 리다이렉션해도 마찬가지
[root@Server-A ~]# cat /tmp/out.txt
표준출력이 파일/파이프로 리다이렉션되어 있습니다.
```

**정리**: `[ -t 1 ]`처럼 `test -t`는 특정 FD가 실제 터미널에 붙어 있는지를 확인하며, 스크립트가 사람이 직접 보는 상황인지 다른 프로그램에 연결된 상황인지에 따라 출력 방식(색상, 진행바 등)을 다르게 하고 싶을 때 조건으로 사용한다.

## [ 와 [[ 의 차이 (단어 분리 · 글롭)

| 항목 | `[ ]` (test) | `[[ ]]` |
|---|---|---|
| 변수 인용 | 반드시 `"$var"`로 감싸야 안전 | 안 감싸도 대부분 안전(내부적으로 단어 분리 안 함) |
| 글롭(`*`, `?`) 확장 | 인자 안에서 글롭이 실제로 확장될 수 있음 | 글롭 확장이 일어나지 않음(패턴 매칭 용도로만 사용 가능) |
| 논리 연산자 | `-a`, `-o` (레거시) 또는 `[ ] && [ ]` | `&&`, `||`, `!`를 조건식 안에서 직접 사용 가능 |
| 비교 연산자 `<`, `>` | 리다이렉션으로 오인되어 `\<`, `\>` 이스케이프 필요 | 그대로 문자열 비교 연산자로 사용 가능 |
| 정규식 매치 `=~` | 지원하지 않음 | 지원 |
| 이식성 | POSIX 호환(`sh`에서도 동작) | Bash 전용 문법(`sh`에서는 사용 불가) |

```bash
[root@Server-A ~]# unset x
[root@Server-A ~]# [ -z $x ] && [ -n $x ]	# 따옴표 없이 빈 변수를 쓰면 인자 자체가 사라져 예상과 다르게 동작할 수 있음

[root@Server-A ~]# [[ -z $x && -n $x ]]	# [[ ]] 안에서는 &&를 조건식 안에 그대로 사용 가능
[root@Server-A ~]# echo $?
1


[root@Server-A ~]# a="apple"
[root@Server-A ~]# b="banana"

[root@Server-A ~]# [ $a < $b ]		# '<'가 파일 리다이렉션으로 해석되어 원하는 비교가 안 됨(예상치 못한 동작)

[root@Server-A ~]# [[ $a < $b ]]		# [[ ]] 안에서는 <, >가 문자열 비교 연산자로 정상 동작
[root@Server-A ~]# echo $?
0
```

**정리**: `[[ ]]`는 Bash 문법으로 처리되기 때문에 변수 인용을 깜빡해도 비교적 안전하고, `&&`/`||`/`<`/`>`를 조건식 안에서 그대로 쓸 수 있어 실무 스크립트에서 선호된다. 다만 `sh`(dash 등)와의 호환성이 필요한 스크립트라면 POSIX 표준인 `[ ]`(test)를 사용해야 한다.

## [[ ]]의 정규식 매치 (=~)

- `[[ 문자열 =~ 정규식 ]]`은 문자열이 **확장 정규식(ERE)**과 매치되는지 검사하는 Bash 전용 문법이다. `[ ]`(test)에는 없는 기능이다.
- 매치에 성공하면 종료 코드 0, 실패하면 1을 반환하며, 매치된 그룹은 배열 변수 **`BASH_REMATCH`**에 저장된다.

```bash
[root@Server-A ~]# vi ./script/regex_test.sh
#!/bin/bash

read -p "IP 주소를 입력하세요 : " ip

if [[ "$ip" =~ ^([0-9]{1,3})\.([0-9]{1,3})\.([0-9]{1,3})\.([0-9]{1,3})$ ]]
then
    echo "형식이 올바른 IP입니다."
    echo "첫 옥텟 : ${BASH_REMATCH[1]}"
else
    echo "IP 형식이 아닙니다."
fi

:wq


[root@Server-A ~]# chmod +x ./script/regex_test.sh

[root@Server-A ~]# ./script/regex_test.sh
IP 주소를 입력하세요 : 192.168.0.10
형식이 올바른 IP입니다.
첫 옥텟 : 192

[root@Server-A ~]# ./script/regex_test.sh
IP 주소를 입력하세요 : hello
IP 형식이 아닙니다.
```

**정리**: `[[ =~ ]]`는 Bash에서만 사용할 수 있는 정규식 매치 문법으로, 단순 패턴이 아니라 IP·이메일 형식 검증처럼 복잡한 문자열 형식을 검사할 때 유용하며, 매치 결과는 `BASH_REMATCH` 배열로 그룹별로 꺼내 쓸 수 있다.

## 숫자 비교 vs 문자열 비교, -eq와 = 를 헷갈리면 생기는 버그

- `-eq`, `-ne`, `-gt`, `-lt`, `-ge`, `-le`는 **숫자로 변환해서** 비교하고, `=`, `!=`는 **문자열 그대로** 비교한다. 겉보기에 숫자처럼 보이는 값이어도 이 둘을 바꿔 쓰면 결과가 완전히 달라질 수 있다.

```bash
[root@Server-A ~]# [ "10" = "10.0" ] && echo "같음" || echo "다름"
다름					# 문자열 비교라 "10"과 "10.0"은 다른 문자열

[root@Server-A ~]# [ "010" -eq "10" ] && echo "같음" || echo "다름"
같음					# 숫자 비교라 앞의 0은 무시되고 같은 값으로 취급
```

- `-eq` 계열에 **숫자가 아닌 문자열**을 넣으면 오류가 난다. 반대로 `=`에 숫자를 넣으면 오류 없이 "다른 문자열"로 조용히 처리되어, 의도한 비교가 아닌데도 스크립트가 그냥 넘어가 버리는 경우가 많다.

```bash
[root@Server-A ~]# [ "abc" -eq "10" ]
bash: [: abc: 정수 표현식이 필요합니다	# 최소한 여기서는 오류로 알아챌 수 있음

[root@Server-A ~]# count="5개"			# 실수로 단위까지 붙여 저장한 값
[root@Server-A ~]# [ "$count" -eq 5 ] && echo "5개입니다"
bash: [: 5개: 정수 표현식이 필요합니다
```

- 실무에서 자주 나오는 버그 패턴: 두 값이 숫자인 것처럼 보여서 `=`로 비교했는데, 앞에 0이 붙거나 공백이 섞여 있어서 "분명 같은 숫자인데 다르다고 나온다"는 오작동이다.

```bash
[root@Server-A ~]# port_a="080"
[root@Server-A ~]# port_b="80"

[root@Server-A ~]# [ "$port_a" = "$port_b" ] && echo "같은 포트" || echo "다른 포트"
다른 포트				# 문자열로는 "080" != "80"

[root@Server-A ~]# [ "$port_a" -eq "$port_b" ] && echo "같은 포트" || echo "다른 포트"
같은 포트				# 숫자로 비교하면 080 == 80
```

**정리**: 값이 "숫자로서" 같은지 비교하려면 반드시 `-eq`/`-gt` 등 숫자 비교 연산자를, "문자열로서" 정확히 같은 표기인지 비교하려면 `=`/`!=`를 사용해야 한다. 두 연산자를 혼동하면 `010`과 `10`처럼 겉보기엔 같은 값이 다르다고 판정되거나, 숫자 아닌 값이 조용히 통과되는 등의 버그로 이어진다.

## [[ ]] 안의 복합 조건 (&& · ||) vs [ ]의 -a · -o

- 여러 조건을 하나의 대괄호 안에서 동시에 검사하고 싶을 때, `[ ]`(test)는 전통적으로 `-a`(AND), `-o`(OR)를 사용했지만 이 방식은 대괄호가 여러 개 섞이면 우선순위가 헷갈리기 쉽고, POSIX 표준에서도 사용을 권장하지 않는다.
- `[[ ]]`는 그 안에서 `&&`, `||`, `!`를 프로그래밍 언어처럼 그대로 사용할 수 있어 훨씬 직관적이다.

```bash
[root@Server-A ~]# age=25
[root@Server-A ~]# name="hong"

[root@Server-A ~]# [ "$age" -ge 20 -a "$name" = "hong" ] && echo "조건 만족"	# -a 사용(레거시)
조건 만족

[root@Server-A ~]# [[ "$age" -ge 20 && "$name" == "hong" ]] && echo "조건 만족"	# &&로 더 명확
조건 만족
```

- `[ ]`에서 안전하게 복합 조건을 표현하는 현대적인 방법은 `-a`/`-o` 대신, `[ ]` 두 개를 `&&`/`||`로 쉘 차원에서 이어붙이는 것이다.

```bash
[root@Server-A ~]# [ "$age" -ge 20 ] && [ "$name" = "hong" ] && echo "조건 만족"
조건 만족
```

- 괄호로 우선순위를 명시해야 하는 복잡한 조건에서는 `[[ ]]` 쪽이 훨씬 안전하다. `[ ]`에서 `\(`, `\)`로 괄호를 이스케이프해야 하는 것과 달리, `[[ ]]` 안에서는 괄호를 그대로 쓸 수 있다.

```bash
[root@Server-A ~]# status="active"
[root@Server-A ~]# role="admin"

[root@Server-A ~]# [[ "$status" == "active" && ( "$role" == "admin" || "$role" == "manager" ) ]] && echo "권한 있음"
권한 있음
```

**정리**: `-a`/`-o`는 `[ ]`에서 여러 조건을 묶는 전통적인 방법이지만 우선순위 혼동과 이식성 문제로 지금은 지양되며, `[ 조건1 ] && [ 조건2 ]`처럼 쉘 연산자로 나누어 쓰거나 아예 `[[ 조건1 && 조건2 ]]`를 사용하는 것이 더 안전하고 읽기 쉽다.

## [[ ]]는 안 감싼 변수도 단어분리·글롭이 안 된다

- `[ ]`는 명령어이기 때문에 인자를 넘기기 전에 쉘이 먼저 **단어 분리(word splitting)**와 **글롭 확장(pathname expansion)**을 수행한다. 변수 안에 공백이나 `*` 같은 특수문자가 있으면 예상과 다른 여러 개의 인자로 쪼개질 수 있다.
- `[[ ]]`는 Bash가 문법 차원에서 직접 처리하기 때문에, 안에 있는 변수는 따옴표 없이 써도 단어 분리나 글롭 확장이 일어나지 않는다. 이것이 `[[ ]]`가 실무에서 더 안전하다고 여겨지는 핵심 이유다.

```bash
[root@Server-A ~]# cd /tmp && touch a.txt b.txt

[root@Server-A ~]# pattern="*.txt"

[root@Server-A ~]# [ $pattern = "*.txt" ] && echo "일치"	# 글롭이 확장되어 a.txt b.txt로 쪼개져 인자 오류
bash: [: 너무 많은 인자입니다.

[root@Server-A ~]# [[ $pattern = "*.txt" ]] && echo "일치"	# [[ ]] 안에서는 글롭 확장이 안 됨
일치
```

- 공백이 포함된 변수도 마찬가지다. `[ ]`에서 따옴표를 빼먹으면 값이 여러 단어로 쪼개져 "인자가 너무 많다"는 오류나, 의도치 않은 참(true) 판정이 나올 수 있다.

```bash
[root@Server-A ~]# path="/data/my folder"

[root@Server-A ~]# [ -d $path ] && echo "디렉터리 있음"	# 공백 때문에 -d에 두 개의 인자가 전달됨
bash: [: 너무 많은 인자입니다.

[root@Server-A ~]# [[ -d $path ]] && echo "디렉터리 있음"	# [[ ]]는 공백이 있어도 하나의 값으로 처리
디렉터리 있음
```

**정리**: `[ ]`는 명령어라서 따옴표 없는 변수가 쉘의 단어 분리·글롭 확장을 그대로 통과하지만, `[[ ]]`는 Bash 문법으로 처리되어 변수를 따옴표로 감싸지 않아도 단어 분리와 글롭 확장이 일어나지 않는다. 그래도 이식성이나 습관 차원에서 `[ ]`를 쓸 때는 항상 `"$var"`로 감싸는 것이 안전하다.

## test의 인자 개수별 동작 (0개 · 1개 · 그 이상)

- `test`(`[ ]`)는 인자 개수에 따라 동작 규칙이 조금씩 다르며, 이 규칙을 모르면 빈 변수를 다룰 때 예상 밖의 결과를 얻을 수 있다.

| 인자 개수 | 동작 |
|---|---|
| 0개 (`[ ]`, `test`) | 항상 **거짓**(종료 코드 1) |
| 1개 (`[ "$x" ]`, `test "$x"`) | 그 문자열이 **비어있지 않으면 참**, 비어있으면 거짓 (`-n` 생략과 동일) |
| 2개, 단항 연산자 (`[ -f "$x" ]`) | 해당 단항 연산자의 정의대로 판단 |
| 3개, 이항 연산자 (`[ "$a" = "$b" ]`) | 해당 이항 연산자의 정의대로 판단 |

```bash
[root@Server-A ~]# [ ] ; echo $?
1					# 인자 0개는 항상 거짓

[root@Server-A ~]# x="hello"
[root@Server-A ~]# [ "$x" ] ; echo $?
0					# 인자 1개, 비어있지 않으므로 참

[root@Server-A ~]# unset x
[root@Server-A ~]# [ "$x" ] ; echo $?
1					# 인자 1개, 빈 문자열이므로 거짓
```

- 변수를 따옴표 없이 썼는데 값이 비어있으면, `test`에 전달되는 인자 자체가 사라져서 **다음에 오는 옵션이 인자 0개짜리 `test`처럼 동작**하거나 완전히 다른 의미로 해석될 수 있다. 이것이 "왜 항상 따옴표로 감싸야 하는가"의 실제 이유 중 하나다.

```bash
[root@Server-A ~]# unset x
[root@Server-A ~]# [ -n $x ] ; echo $?		# $x가 사라져서 [ -n ]만 남고, -n은 "문자열 -n이 비었나"로 오판됨
1

[root@Server-A ~]# [ -n "$x" ] ; echo $?	# 따옴표로 감싸면 빈 문자열임을 정확히 검사
1
```

**정리**: `test`/`[ ]`는 인자 0개면 항상 거짓, 인자 1개면 그 문자열이 비어있는지만 검사하는 특수 규칙을 갖고 있으며, 변수를 따옴표 없이 쓰다가 빈 문자열이 되면 인자 개수 자체가 달라져 예상과 다른 판정이 나올 수 있으므로 항상 `"$var"`로 감싸는 습관이 중요하다.

## 실습 예제 (EX1~EX5)

**EX1. 사용자 입력이 숫자인지 문자열인지 검증하는 스크립트**

요구사항 : 입력값이 순수 숫자인지 정규식으로 먼저 검사한 뒤, 숫자면 크기 비교, 아니면 문자열로만 처리하는 스크립트를 작성한다.

```bash
[root@Server-A ~]# vi ./script/validate_input.sh
#!/bin/bash
read -p "값을 입력하세요 : " val

if [[ "$val" =~ ^[0-9]+$ ]]; then
    if [ "$val" -ge 100 ]; then
        echo "100 이상의 숫자입니다."
    else
        echo "100 미만의 숫자입니다."
    fi
else
    echo "숫자가 아닌 문자열입니다."
fi

:wq


[root@Server-A ~]# chmod +x ./script/validate_input.sh
[root@Server-A ~]# ./script/validate_input.sh
값을 입력하세요 : 250
100 이상의 숫자입니다.

[root@Server-A ~]# ./script/validate_input.sh
값을 입력하세요 : hello
숫자가 아닌 문자열입니다.
```

**EX2. 여러 파일 검사 플래그를 조합해 백업 대상 여부 판단**

요구사항 : 파일이 존재하고, 읽기 가능하고, 크기가 0보다 큰 경우에만 백업 대상으로 판단하는 스크립트를 작성한다.

```bash
[root@Server-A ~]# vi ./script/backup_target.sh
#!/bin/bash
file="/data/report.log"

if [[ -e "$file" && -r "$file" && -s "$file" ]]; then
    echo "$file : 백업 대상입니다."
else
    echo "$file : 백업 대상이 아닙니다."
fi

:wq


[root@Server-A ~]# chmod +x ./script/backup_target.sh
[root@Server-A ~]# ls -l /data/report.log
-rw-r--r-- 1 root root 2048  9월  9 10:00 /data/report.log

[root@Server-A ~]# ./script/backup_target.sh
/data/report.log : 백업 대상입니다.
```

**EX3. 포트 번호 두 개를 문자열/숫자 두 방식으로 비교해 차이 확인**

요구사항 : 앞에 0이 붙은 포트 표기와 그렇지 않은 표기를 `=`와 `-eq`로 각각 비교해 결과 차이를 눈으로 확인한다.

```bash
[root@Server-A ~]# vi ./script/port_compare.sh
#!/bin/bash
port_a="080"
port_b="80"

[ "$port_a" = "$port_b" ] && echo "문자열 비교 : 같음" || echo "문자열 비교 : 다름"
[ "$port_a" -eq "$port_b" ] && echo "숫자 비교 : 같음" || echo "숫자 비교 : 다름"

:wq


[root@Server-A ~]# chmod +x ./script/port_compare.sh
[root@Server-A ~]# ./script/port_compare.sh
문자열 비교 : 다름
숫자 비교 : 같음
```

**EX4. [ ]에서 공백 포함 변수를 안전하게/안전하지 않게 다뤄보기**

요구사항 : 공백이 포함된 경로 변수를 따옴표 없이/있이 각각 `[ ]`와 `[[ ]]`로 검사해서 오류 여부를 비교한다.

```bash
[root@Server-A ~]# mkdir -p "/data/my folder"

[root@Server-A ~]# vi ./script/space_test.sh
#!/bin/bash
path="/data/my folder"

echo "-- [ ] 따옴표 없이 --"
[ -d $path ] && echo "디렉터리 있음"

echo "-- [ ] 따옴표 있이 --"
[ -d "$path" ] && echo "디렉터리 있음"

echo "-- [[ ]] 따옴표 없이 --"
[[ -d $path ]] && echo "디렉터리 있음"

:wq


[root@Server-A ~]# chmod +x ./script/space_test.sh
[root@Server-A ~]# ./script/space_test.sh
-- [ ] 따옴표 없이 --
./script/space_test.sh: 줄 5: [: /data/my: 너무 많은 인자입니다.
-- [ ] 따옴표 있이 --
디렉터리 있음
-- [[ ]] 따옴표 없이 --
디렉터리 있음
```

**EX5. test 인자 개수 규칙을 직접 확인하는 스크립트**

요구사항 : 변수가 설정되지 않았을 때, 빈 문자열일 때, 값이 있을 때 각각 `[ "$var" ]`의 결과를 출력해 규칙을 확인한다.

```bash
[root@Server-A ~]# vi ./script/test_arity.sh
#!/bin/bash

check () {
    if [ "$1" ]; then
        echo "참 (비어있지 않음)"
    else
        echo "거짓 (비어있음)"
    fi
}

unset a; check "$a"
b=""; check "$b"
c="value"; check "$c"

:wq


[root@Server-A ~]# chmod +x ./script/test_arity.sh
[root@Server-A ~]# ./script/test_arity.sh
거짓 (비어있음)
거짓 (비어있음)
참 (비어있지 않음)
```

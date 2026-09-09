# Shell Script 함수 (Function)

## 함수(Function)란?

**함수(Function)**는 자주 반복해서 사용하는 명령어들을 하나의 이름으로 묶어놓고, 필요할 때마다 그 이름만 호출해서 실행하는 문법이다.

- 같은 코드를 여러 번 반복해서 작성하는 대신, 함수로 한 번만 정의하고 여러 번 호출해서 사용한다.
- 스크립트가 길어질수록 코드를 기능 단위로 나누어 관리할 수 있어 가독성과 유지보수성이 좋아진다.
- 함수 안에서도 변수, 조건문, 반복문, 위치 매개변수 등 지금까지 배운 문법을 그대로 사용할 수 있다.

**함수를 사용하는 이유**
- 반복되는 로직(로그 출력, 백업, 서비스 점검 등)을 한 곳에서 관리
- 스크립트를 기능별로 나누어 읽기 쉽게 구성
- 같은 함수를 여러 스크립트에서 재사용
- 코드 수정 시 함수 내부만 고치면 되므로 유지보수가 쉬움

**정리**: 함수는 반복되는 명령어 묶음에 이름을 붙여 재사용하는 문법이며, 변수·조건문·반복문과 함께 사용해 실무 스크립트를 기능 단위로 구조화하는 데 쓰인다.

## 함수 정의와 호출

Bash에서 함수를 정의하는 방식은 크게 두 가지다.

**형식 1**

```bash
function 함수이름 {
    실행할 명령
}
```

**형식 2 (더 널리 쓰이는 방식)**

```bash
함수이름 () {
    실행할 명령
}
```

- 함수는 **정의만 해서는 실행되지 않는다.** 반드시 함수이름을 호출해야 실행된다.
- 함수는 스크립트 안에서 **호출하기 전에 먼저 정의**되어 있어야 한다.

```bash
[root@Server-A ~]# vi  ./script/func_example01.sh
#!/bin/bash

hello () {
    echo "Hello, Shell Function!"
}

echo "함수 호출 전"
hello			# 함수 호출
echo "함수 호출 후"

:wq


[root@Server-A ~]# chmod  +x  ./script/func_example01.sh

[root@Server-A ~]# ./script/func_example01.sh
함수 호출 전
Hello, Shell Function!
함수 호출 후
```

- 함수는 여러 번 반복해서 호출할 수 있다.

```bash
[root@Server-A ~]# vi  ./script/func_example02.sh
#!/bin/bash

greet () {
    echo "환영합니다."
}

greet
greet
greet

:wq


[root@Server-A ~]# chmod  +x  ./script/func_example02.sh

[root@Server-A ~]# ./script/func_example02.sh
환영합니다.
환영합니다.
환영합니다.
```

**정리**: 함수는 `함수이름 () { 명령 }` 형식으로 정의하며, 정의만으로는 실행되지 않고 스크립트에서 `함수이름`을 호출해야 실행된다. 호출 전에 정의가 먼저 와야 한다.

## 함수의 매개변수 ($1, $2, $#, $@)

함수도 스크립트처럼 호출할 때 값을 전달할 수 있다. 이때 전달된 값은 스크립트 실행 인자와 마찬가지로 **$1, $2, $#, $@** 같은 위치 매개변수로 함수 안에서 받는다.

- 함수 안에서의 $1, $2, $#, $@는 **스크립트 전체의 인자가 아니라, 그 함수를 호출할 때 넘긴 인자**를 가리킨다.
- 형식 : `함수이름  인자1  인자2 ...`

```bash
[root@Server-A ~]# vi  ./script/func_example03.sh
#!/bin/bash

greet () {
    echo "이름 : $1"
    echo "나이 : $2"
    echo "전달받은 인자 개수 : $#"
    echo "전달받은 인자 전체 : $@"
}

greet  kim  20

:wq


[root@Server-A ~]# chmod  +x  ./script/func_example03.sh

[root@Server-A ~]# ./script/func_example03.sh
이름 : kim
나이 : 20
전달받은 인자 개수 : 2
전달받은 인자 전체 : kim 20
```

- 함수를 여러 다른 인자로 반복 호출할 수 있다.

```bash
[root@Server-A ~]# vi  ./script/func_example04.sh
#!/bin/bash

greet () {
    echo "안녕하세요, $1님. 오늘은 $2입니다."
}

greet  kim  월요일
greet  lee  화요일

:wq


[root@Server-A ~]# chmod  +x  ./script/func_example04.sh

[root@Server-A ~]# ./script/func_example04.sh
안녕하세요, kim님. 오늘은 월요일입니다.
안녕하세요, lee님. 오늘은 화요일입니다.
```

**정리**: 함수 안의 $1, $2, $#, $@는 스크립트 실행 인자가 아니라 함수 호출 시 넘긴 인자를 가리킨다. `함수이름 인자1 인자2`형태로 호출하면 함수 내부에서 위치 매개변수로 받아 사용할 수 있다.

## 지역 변수(local)와 전역 변수

- Bash에서 변수는 기본적으로 **전역(global)** 이다. 함수 안에서 선언한 변수도 특별히 지정하지 않으면 스크립트 전체에서 사용/변경할 수 있다.
- 함수 안에서만 사용할 변수는 **local** 키워드를 붙여 지역 변수로 선언하는 것이 안전하다.
- local로 선언한 변수는 함수가 끝나면 사라지며, 함수 밖의 같은 이름의 변수에 영향을 주지 않는다.

**전역 변수로 인해 값이 덮어써지는 경우**

```bash
[root@Server-A ~]# vi  ./script/func_example05.sh
#!/bin/bash

msg="원본 값"

change_msg () {
    msg="함수 안에서 변경된 값"	# local이 없으면 전역 변수 msg를 그대로 수정
}

echo "함수 호출 전 : $msg"
change_msg
echo "함수 호출 후 : $msg"

:wq


[root@Server-A ~]# chmod  +x  ./script/func_example05.sh

[root@Server-A ~]# ./script/func_example05.sh
함수 호출 전 : 원본 값
함수 호출 후 : 함수 안에서 변경된 값		# 전역 변수가 그대로 변경됨
```

**local로 지역 변수를 사용하는 경우**

```bash
[root@Server-A ~]# vi  ./script/func_example06.sh
#!/bin/bash

msg="원본 값"

change_msg () {
    local msg="함수 안에서만 사용되는 값"	# local로 선언하여 함수 안에서만 유효
    echo "함수 내부 : $msg"
}

echo "함수 호출 전 : $msg"
change_msg
echo "함수 호출 후 : $msg"

:wq


[root@Server-A ~]# chmod  +x  ./script/func_example06.sh

[root@Server-A ~]# ./script/func_example06.sh
함수 호출 전 : 원본 값
함수 내부 : 함수 안에서만 사용되는 값
함수 호출 후 : 원본 값			# 전역 변수 msg는 영향을 받지 않음
```

**정리**: local을 붙이지 않으면 함수 안의 변수도 전역 변수이므로 함수 밖의 같은 이름 변수를 덮어쓸 수 있다. 함수 내부에서만 임시로 쓸 변수는 반드시 `local`로 선언해 의도치 않은 값 변경을 막는 것이 안전하다.

## return과 함수의 결과값

- **return**은 함수를 종료하면서 **종료 상태값(0~255)** 을 반환하는 명령이다. 반복문의 break/continue처럼 함수 자체를 빠져나가는 역할을 한다.
- return의 값은 문자열이나 계산 결과가 아니라 **성공(0)/실패(1 이상)** 같은 상태 코드를 돌려주는 용도이며, 함수 종료 직후 `$?`로 확인할 수 있다.
- return을 생략하면 함수 안에서 마지막으로 실행된 명령의 종료 상태값이 그대로 반환된다.

```bash
[root@Server-A ~]# vi  ./script/func_example07.sh
#!/bin/bash

check_even () {
    if (( $1 % 2 == 0 ))
    then
        return 0		# 짝수이면 성공(0) 반환
    else
        return 1		# 홀수이면 실패(1) 반환
    fi
}

check_even  10

if [ $? -eq 0 ]
then
    echo "10은 짝수입니다."
else
    echo "10은 홀수입니다."
fi

:wq


[root@Server-A ~]# chmod  +x  ./script/func_example07.sh

[root@Server-A ~]# ./script/func_example07.sh
10은 짝수입니다.
```

**함수에서 값을 "돌려받고" 싶을 때 (echo + 명령 치환)**

- 함수는 return으로 문자열이나 계산값을 직접 반환할 수 없다(0~255 범위의 상태값만 가능).
- 함수가 계산한 문자열/숫자 결과를 변수로 받아 쓰려면, 함수 안에서 **echo**로 값을 출력하고 함수를 호출하는 쪽에서 **$( )** 명령 치환으로 그 출력을 받는 방식을 사용한다.

```bash
[root@Server-A ~]# vi  ./script/func_example08.sh
#!/bin/bash

add () {
    local sum=$(( $1 + $2 ))
    echo "$sum"		# 값을 echo로 출력 (return이 아님)
}

result=$(add  10  20)		# 함수의 echo 출력을 명령 치환으로 변수에 저장

echo "10 + 20 = $result"

:wq


[root@Server-A ~]# chmod  +x  ./script/func_example08.sh

[root@Server-A ~]# ./script/func_example08.sh
10 + 20 = 30
```

**정리**: return은 함수의 성공/실패(0~255)를 알리는 상태값이며 $?로 확인한다. 문자열이나 계산 결과처럼 "값"을 돌려받고 싶을 때는 함수 안에서 echo로 출력하고, 호출하는 쪽에서 `$(함수이름 인자)` 명령 치환으로 받아서 사용한다.

## 함수 실습 예제 (EX1~EX6)

**EX1) 인사 함수**
- 이름을 인자로 받아 "안녕하세요, 이름님!"을 출력하는 함수를 작성하시오.

```bash
[root@Server-A ~]# vi  ./script/func_ex01.sh
#!/bin/bash

greet () {
    echo "안녕하세요, $1님!"
}

greet  "홍길동"

:wq


[root@Server-A ~]# chmod  +x  ./script/func_ex01.sh

[root@Server-A ~]# ./script/func_ex01.sh
안녕하세요, 홍길동님!
```

**EX2) 두 수의 합을 반환하는 함수**
- 두 정수를 인자로 받아 합을 echo로 출력하는 함수를 작성하고, 결과를 변수에 저장하여 출력하시오.

```bash
[root@Server-A ~]# vi  ./script/func_ex02.sh
#!/bin/bash

sum () {
    echo $(( $1 + $2 ))
}

total=$(sum  15  25)

echo "합계 : $total"

:wq


[root@Server-A ~]# chmod  +x  ./script/func_ex02.sh

[root@Server-A ~]# ./script/func_ex02.sh
합계 : 40
```

**EX3) 파일 존재 여부 확인 함수**
- 경로를 인자로 받아 파일이 있으면 0, 없으면 1을 return하는 함수를 작성하고 호출 결과에 따라 메세지를 출력하시오.

```bash
[root@Server-A ~]# vi  ./script/func_ex03.sh
#!/bin/bash

check_file () {
    if [ -f "$1" ]
    then
        return 0
    else
        return 1
    fi
}

check_file  "/etc/passwd"

if [ $? -eq 0 ]
then
    echo "파일이 존재합니다."
else
    echo "파일이 존재하지 않습니다."
fi

:wq


[root@Server-A ~]# chmod  +x  ./script/func_ex03.sh

[root@Server-A ~]# ./script/func_ex03.sh
파일이 존재합니다.
```

**EX4) 로그 출력 함수**
- 로그 메세지를 인자로 받아 "[현재시간] 메세지" 형태로 출력하는 log 함수를 작성하고, 여러 번 호출해보시오.

```bash
[root@Server-A ~]# vi  ./script/func_ex04.sh
#!/bin/bash

log () {
    echo "[$(date +"%F %T")] $1"
}

log  "서비스 점검을 시작합니다."
log  "httpd 상태 확인 완료"
log  "서비스 점검을 종료합니다."

:wq


[root@Server-A ~]# chmod  +x  ./script/func_ex04.sh

[root@Server-A ~]# ./script/func_ex04.sh
[2026-08-05 10:20:11] 서비스 점검을 시작합니다.
[2026-08-05 10:20:11] httpd 상태 확인 완료
[2026-08-05 10:20:11] 서비스 점검을 종료합니다.
```

**EX5) 배열을 순회하며 처리하는 함수**
- 배열을 인자로 받아 짝수만 출력하는 함수를 작성하시오. (함수에 배열을 넘길 때는 `"${배열이름[@]}"` 형태로 넘긴다.)

```bash
[root@Server-A ~]# vi  ./script/func_ex05.sh
#!/bin/bash

print_even () {
    local  arr=("$@")	# 함수에 전달된 모든 인자를 배열로 재구성

    for  num  in  "${arr[@]}"
    do
        if (( num % 2 == 0 ))
        then
            echo "$num"
        fi
    done
}

nums=(1 2 3 4 5 6 7 8 9 10)

print_even  "${nums[@]}"

:wq


[root@Server-A ~]# chmod  +x  ./script/func_ex05.sh

[root@Server-A ~]# ./script/func_ex05.sh
2
4
6
8
10
```

**EX6) 서비스 상태 점검 함수**
- 서비스 이름을 인자로 받아 systemctl로 상태를 확인하고, 실행 중이면 "정상", 아니면 "중지됨"을 출력하는 함수를 작성하시오.

```bash
[root@Server-A ~]# vi  ./script/func_ex06.sh
#!/bin/bash

check_service () {
    if  systemctl  is-active  --quiet  "$1"
    then
        echo "$1 : 정상 실행 중"
    else
        echo "$1 : 중지됨"
    fi
}

check_service  sshd
check_service  httpd

:wq


[root@Server-A ~]# chmod  +x  ./script/func_ex06.sh

[root@Server-A ~]# ./script/func_ex06.sh
sshd : 정상 실행 중
httpd : 중지됨
```

**정리**: EX1~EX6은 인사/계산/파일 검사/로그 출력/배열 처리/서비스 점검처럼 실무에서 자주 쓰는 패턴을 함수로 구조화하는 예제다. 함수에 배열을 넘길 때는 `"${배열[@]}"` 형태로 펼쳐서 전달하고, 함수 안에서는 `local  arr=("$@")`로 다시 배열로 묶어 받는 방식이 자주 쓰인다.

## 재귀 함수(Recursive Function)

**재귀 함수**는 함수가 자기 자신을 다시 호출하는 함수이다.

- 반복되는 구조를 for/while 대신 함수 호출로 표현할 때 사용한다.
- 재귀 함수는 반드시 **종료 조건**이 있어야 하며, 종료 조건이 없으면 함수가 무한히 자기 자신을 호출하여 스크립트가 멈추지 않는다.

**EX) 팩토리얼(계승) 계산**
- 정수를 인자로 받아 1부터 그 수까지 곱한 값(팩토리얼)을 재귀 함수로 계산하시오.

```bash
[root@Server-A ~]# vi  ./script/func_recursive01.sh
#!/bin/bash

factorial () {
    local  n=$1

    if (( n <= 1 ))
    then
        echo 1				# 종료 조건 : 1 이하이면 1을 반환하고 재귀 종료
        return
    fi

    local  prev=$(factorial  $((n - 1)))	# 자기 자신을 다시 호출 (재귀)
    echo $(( n * prev ))
}

result=$(factorial  5)

echo "5! = $result"

:wq


[root@Server-A ~]# chmod  +x  ./script/func_recursive01.sh

[root@Server-A ~]# ./script/func_recursive01.sh
5! = 120
```

**정리**: 재귀 함수는 자기 자신을 호출하며 문제를 더 작은 단위로 쪼개 풀어나가는 방식이다. 종료 조건(base case)이 반드시 있어야 하며, 각 호출에서 결과를 echo로 넘기고 명령 치환으로 받아 다음 계산에 사용하는 패턴이 Bash 재귀 함수의 기본 골격이다.

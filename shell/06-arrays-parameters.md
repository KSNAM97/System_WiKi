# Shell Script 배열(Array) & 위치 매개변수 (Positional Parameters)

## 목차

1. [배열 (Array)](#배열-array)
2. [위치 매개변수 (Positional Parameters)](#위치-매개변수-positional-parameters)

## 배열 (Array)

**배열(Array)**은 여러 개의 값을 하나의 변수에 순서대로 저장하는 자료 구조이다. 서버 목록이나 파일 목록처럼 여러 관련 값을 하나로 묶어 관리하거나, 스크립트 실행 시 전달된 인자(위치 매개변수)를 다루는 데 주로 쓰인다.

- 일반 변수는 하나의 값만 저장하지만, 배열은 여러 값을 순서대로 저장할 수 있다.

```bash
fruit1="사과"
fruit2="바나나"
fruit3="오렌지"
```

- 배열을 사용하면 하나의 변수에 여러 값을 저장할 수 있다.

```bash
fruits=("사과" "바나나" "오렌지")
```

### 배열 생성

- 기본 형식 : `배열명=("값1" "값2" "값3" "값4" ...)`
- 배열의 각 요소는 공백을 사용하여 구분한다.

예제:

```bash
num=(10 20 30 40)

name=("kim" "lee" "park")

words=("hello" "world" "welcome")
```

### 배열의 요소(Element) 접근

- 배열에 저장된 각각의 값을 **요소(Element)**라고 한다.
- 배열의 요소는 **index** 번호를 사용하여 접근할 수 있다.
- bash에서 인덱스 번호는 0번부터 시작한다.

```bash
[root@Server-A ~]# name=("kim" "lee" "park" "ryu")

[root@Server-A ~]# echo ${name[0]}
kim

[root@Server-A ~]# echo ${name[1]}
lee

[root@Server-A ~]# echo ${name[2]}
park

[root@Server-A ~]# echo ${name[3]}
ryu
```

### 배열의 값 출력

- 배열의 모든 요소를 출력할때에는 **@** 또는 **\*** 를 사용한다.

```bash
[root@Server-A ~]# echo ${name[@]}
kim lee park ryu

[root@Server-A ~]# echo ${name[*]}
kim lee park ryu
```

- **"${name[@]}"** 는 배열안의 각 요소를 각각의 독립된 값으로 처리한다.
- **"${name[*]}"** 는 배열안의 모든 요소를 하나의 문자열처럼 처리
- 반복문을 사용하여 순회 처리시에는 일반적으로 **"${name[@]}"**를 사용한다.

### 배열의 길이, 인덱스 번호 확인

```bash
[root@Server-A ~]# echo ${name[@]}
kim lee park ryu

[root@Server-A ~]# echo ${!name[@]}	# 배열안의 인덱스 번호 확인
0 1 2 3

[root@Server-A ~]# echo ${#name[@]}	# 배열의 길이를 확인 
4
```

| 명령어 | 의미 | 결과 |
|---|---|---|
| `${name[@]}` | 배열의 모든 값 | kim lee park choi |
| `${#name[@]}` | 배열 요소의 개수 | 4 |
| `${!name[@]}` | 배열의 인덱스 번호 | 0 1 2 3 |

### 배열 요소 추가

방법1) 인덱스 번호를 사용하여 배열 요소 추가

```bash
[root@Server-A ~]# name[4]="hong"

[root@Server-A ~]# echo ${name[@]}
kim lee park ryu hong			# 요소 추가 확인
```

방법2) 배열안에 값을 넣으면 가장 마지막 인덱스 다음 인덱스에 추가된다.

```bash
[root@Server-A ~]# name+=("choi")		# += 연산자를 사용하면 가장 마지막 인덱스 다음에 추가된다.

[root@Server-A ~]# echo ${name[@]}
kim lee park ryu hong choi
```

### 배열 요소 수정

- 배열에 이미 존재하는 인덱스 번호를 지정하여 새로운 값을 저장하면 해당 인덱스의 기존 값이 새로운 값으로 변경된다.
- 배열 전체를 다시 선언하지 않아도 수정할 요소의 인덱스 번호만 지정하여 값을 변경할 수 있다.
- 배열 요소를 수정하더라도 배열의 요소 개수와 다른 요소의 인덱스 번호는 변경되지 않는다.

```bash
[root@Server-A ~]# echo ${name[@]}
kim lee park ryu hong choi

[root@Server-A ~]# name[3]="jung"

[root@Server-A ~]# echo ${name[@]}
kim lee park jung hong choi		 # 3번 인덱스의 ryu값이  jung으로 수정

[root@Server-A ~]# echo ${!name[@]}
0 1 2 3 4 5
```

### 배열 요소 삭제

- **unset** 명령을 사용하면 배열의 특정 요소 또는 배열 전체를 삭제할 수 있다.
- 특정 배열 요소를 삭제할 때는 배열 이름과 삭제할 인덱스 번호를 지정한다.
- 형식 : `unset '배열이름[인덱스번호]'`
- 배열 요소를 삭제할 때는 대괄호가 셸의 패턴 문자로 해석되는 것을 방지하기 위해 `' '`로 묶어서 사용하는 것이 안전하다.

```bash
[root@Server-A ~]# echo ${name[@]}
kim lee park jung hong choi		# 각 요소의 값
 0     1    2     3     4     5		# 각 요소의 인덱스 번호

[root@Server-A ~]# echo ${!name[@]}
0 1 2 3 4 5

[root@Server-A ~]# unset 'name[2]'

[root@Server-A ~]# echo ${name[@]}
kim lee jung hong choi

[root@Server-A ~]# echo ${!name[@]}
0 1 3 4 5
```

**정리**: 배열은 `배열명=(값들)` 형식으로 선언하며, 인덱스로 요소를 조회/추가/수정하고 `unset`으로 삭제한다. `${arr[@]}`(개별 값), `${arr[*]}`(하나의 문자열), `${!arr[@]}`(인덱스 목록), `${#arr[@]}`(요소 개수)를 상황에 맞게 구분해서 사용하는 것이 핵심이다.

### RANDOM 변수와 배열 실습

- **RANDOM**은 Bash에서 제공하는 특수 변수이다.
- RANDOM 변수의 값을 출력할 때마다 임의의 정수를 생성한다.
- 생성되는 값의 기본 범위 : 0 ~ 32767

**EX1-1) 배열에서 짝수만 출력하기**
- num 배열을 빈 배열로 선언
- for 반복문과 $RANDOM을 사용하여 num 배열에 랜덤한 정수 10개를 저장
- 반복문, 조건문을 사용하여 배열 요소중 짝수인 값만 출력

```bash
[root@Server-A ~]# vi  ./script/arr_example01.sh
#!/bin/bash

num=()

for ((i=0;  i<10;  i++))	# num 배열에 10개의 랜덤값을 넣기위한 반복문
do
    num[i]=$(( RANDOM ))
done

echo "배열안의 짝수 값 : ${num[@]}"
echo

echo "짝수 값 출력"
for  number  in  "${num[@]}"
do
    if (( number % 2 == 0 ))
    then
        echo "$number"
    fi
done
```

```bash
[root@Server-A ~]# chmod  +x  ./script/arr_example01.sh

[root@Server-A ~]# ./script/arr_example01.sh
배열안의 짝수 값 : 376 30682 13595 22303 6761 10565 11563 14092 11636 25255

짝수 값 출력
376
30682
14092
11636
```

**EX1-2) 배열 안의 짝수와 홀수 개수 출력하기**
- num 배열에 랜덤한 정수 100개를 저장하시오
- 반복문과 조건문을 사용하여 배열 안에 저장된 짝수와 홀수의 개수를 각각 계산출력해야 한다.
- 전체 배열 요소 개수, 짝수 개수, 홀수 개수를 다음 형식으로 출력

```
배열 요소 개수 : 100
짝수 개수 : xx
홀수 개수 : xx
```

```bash
[root@Server-A ~]# vi ./script/arr_example01_2.sh
#!/bin/bash

num=()
even=0
odd=0

for (( i=0;  i<100;  i++))
do
    num[i]=$RANDOM
done

for  number  in  "${num[@]}"
do
    if (( number % 2 == 0 ))
    then
        (( even++ ))
    else
        (( odd++ ))
    fi
done

echo "배열안의 정수 : ${#num[@]}개"
echo "짝수 개수 : ${even}개"
echo "홀수 개수 : ${odd}개"
```

```bash
[root@Server-A ~]# chmod  +x  ./script/arr_example01_2.sh

[root@Server-A ~]# ./script/arr_example01_2.sh
배열안의 정수 : 100개
짝수 개수 : 45개
홀수 개수 : 55개

[root@Server-A ~]# ./script/arr_example01_2.sh
배열안의 정수 : 100개
짝수 개수 : 49개
홀수 개수 : 51개
```

**정리**: `RANDOM`으로 배열을 채우고 `for ... in "${arr[@]}"` 순회와 산술 비교(`(( ))`)를 조합하면 짝수/홀수 카운팅 같은 통계 처리를 간단히 구현할 수 있다.

### 점수 배열 실습 (합격/불합격 판정)

- RANDOM은 Bash에서 제공하는 특수 변수이다.
- RANDOM 변수의 값을 출력할 때마다 임의의 정수를 생성한다.
- 생성되는 값의 기본 범위 : 0 ~ 32767

**EX2-0) 랜덤 점수의 합격/불합격 출력하기**
- scores 배열에 0~100 사이의 랜덤한 점수 10개를 저장하시오

```bash
[root@Server-A ~]# vi ./script/arr_example02_0.sh
#!/bin/bash

scores=()

for ((i=0;  i<50;  i++))
do
    scores[i]=$(( RANDOM % 101 ))
    echo  -n "${scores[i]} "
done
echo
```

```bash
[root@Server-A ~]# chmod +x ./script/arr_example02_0.sh

[root@Server-A ~]# ./script/arr_example02_0.sh
53 25 83 32 89 10 25 43 0 18 86 86 50 8 64 38 7 99 1 12 39 54 42 19 86 13 90 61 61 42 65 7 68 64 39 16 82 84 15 22 33 53 49 1 74 0 47 81 100 59

[root@Server-A ~]# ./script/arr_example02_0.sh
61 8 45 25 9 1 96 32 40 72 5 43 37 97 10 16 76 61 42 9 91 91 3 5 92 34 85 23 20 34 70 96 57 30 12 60 97 22 43 24 21 50 37 60 94 52 69 22 56 80
```

**EX2-1) 랜덤 점수의 합격/불합격 출력하기**
- scores 배열에 30~100 사이의 랜덤한 점수 10개를 저장하시오

```bash
[root@Server-A ~]# vi ./script/arr_example02_1.sh
#!/bin/bash

scores=()

for ((i=0;  i<50;  i++))
do
    scores[i]=$(( (RANDOM % 71) + 30  ))
    echo  -n "${scores[i]} "
done
echo
```

```bash
[root@Server-A ~]# chmod +x ./script/arr_example02_1.sh

[root@Server-A ~]# ./script/arr_example02_1.sh




```

**EX2-2) 랜덤 점수의 합격/불합격 출력하기**
- scores 배열에 0~100 사이의 랜덤한 점수 10개를 저장하시오
- 반복문을 사용하여 배열의 점수를 하나씩 확인하시오
- 점수가 60점 이상이면 "점수: xx  합격"을 출력하시오
- 점수가 60점 미만이면 "점수: xx  불합격"을 출력하시오

```bash
[root@Server-A ~]# vi ./script/arr_example02_2.sh
#!/bin/bash

scores=()

for ((i=0;  i<10;  i++))
do
    scores[i]=$(( RANDOM % 101 ))
done


for  score  in  "${scores[@]}"
do
    if (( score >= 60 ))
    then
        echo "점수 ${score}점 : 합격"
    else
        echo "점수 ${score}점 : 불합격"
    fi
done
```

```bash
[root@Server-A ~]# chmod  +x  ./script/arr_example02_2.sh

[root@Server-A ~]# ./script/arr_example02_2.sh
점수 67점 : 합격
점수 80점 : 합격
점수 59점 : 불합격
점수 81점 : 합격
점수 28점 : 불합격
점수 13점 : 불합격
점수 21점 : 불합격
점수 74점 : 합격
점수 45점 : 불합격
점수 3점 : 불합격
```

**EX2-2) 학생별 3과목 평균과 합격 여부 출력하기**
- 국어, 영어, 수학 점수를 각각 배열로 관리하려고 한다.
- 각 과목 배열에 30~100 사이의 랜덤한 점수를 10개씩 저장
- 같은 인덱스의 국어, 영어, 수학 점수는 같은 학생의 점수로 처리
- 반복문을 사용하여 1번 학생부터 10번 학생까지 각 학생의 국어, 영어, 수학 점수와 평균을 출력
- 3과목의 평균이 60점 이상이면 합격, 60점 미만이면 불합격을 출력

출력 형식:

```
1번 학생
국어 : xx
영어 : xx
수학 : xx
총점 : xxx
평균 : xx
결과 : 합격 또는 불합격
    ~
10번 학생
국어 : xx
영어 : xx
수학 : xx
총점 : xxx
평균 : xx
결과 : 합격 또는 불합격
```

- 합격 : XX명
- 불합격 : XX명

```bash
[root@Server-A ~]# vi ./script/arr_example02_3.sh
#!/bin/bash

kor=()
eng=()
math=()

pass=0
fail=0

for (( i=0;  i<10;  i++))
do
    kor[i]=$(( (RANDOM % 71) + 30 ))
    eng[i]=$(( (RANDOM % 71) + 30 ))
    math[i]=$(( (RANDOM % 71) + 30 ))
done


echo "-----------------------------"
for (( i=0;  i<10;  i++))
do
    total=$(( kor[i] + eng[i] + math[i] ))
    everage=$(( total / 3 ))
    
    echo "$(( i + 1 ))번 학생"
    echo "국어 점수 : ${kor[i]}점"
    echo "영어 점수 : ${eng[i]}점"
    echo "수학 점수 : ${math[i]}점"
    echo "총점 점수 : $total점"
    echo "평균 점수 : $everage점"

    if ((  (everage >= 60)  &&  (kor[i] < 40 || eng[i] < 40 || math[i] < 40)  ))
    then
        echo "평균 ${everage}점 : 과락 불합격"
        (( fail++ ))
    elif (( everage >= 60 ))
    then
        echo "평균 ${everage}점 : 합격"
        (( pass++ ))
    else
        echo "평균 ${everage}점 : 불합격"
        (( fail++ ))
    fi
    echo "-----------------------------"
done


echo "합격 : ${pass}명"
echo "불합격 : ${fail}명"
```

```bash
[root@Server-A ~]# chmod  +x  ./script/arr_example02_3.sh

[root@Server-A ~]# ./script/arr_example02_3.sh
1번 학생
국어 점수 : 44점
영어 점수 : 39점
수학 점수 : 84점
총점 점수 : 167점
평균 점수 : 55점
평균 55점 : 불합격
-----------------------------

~~~~~~ 중간 생략 ~~~~~~

10번 학생
국어 점수 : 81점
영어 점수 : 77점
수학 점수 : 95점
총점 점수 : 253점
평균 점수 : 84점
평균 84점 : 합격
-----------------------------
합격 : 7명
불합격 : 3명
```

**정리**: 여러 배열을 같은 인덱스로 짝지어(국어/영어/수학) 관리하면 학생 단위의 복합 데이터를 표현할 수 있으며, 평균 계산과 과락 조건(`&&`, `||` 결합)을 조합해 합격/불합격 로직을 세분화할 수 있다.

---

## 위치 매개변수 (Positional Parameters)

**위치 매개변수(Positional Parameters)**는 쉘 스크립트를 실행할 때 명령어 뒤에 전달한 값을 저장하는 특수 변수이다.

- 전달된 값은 입력된 순서에 따라 **$1**, **$2**, **$3** 등의 변수에 자동으로 저장된다.
- 위치 매개변수를 사용하면 read 명령어로 값을 입력받지 않고 스크립트를 실행하면서 필요한 값을 전달할 수 있다.

**EX)** `./script/test.sh   apple  banana  cherry`
- 이렇게 실행하면 스크립트 내부에서 다음과 같이 자동으로 저장된다.
- $1 --> apple
- $2 --> banana
- $3 --> cherry
- 즉, 사용자가 입력한 값(인자)을 스크립트 내부에서 활용할 수 있게 하는 기능이다.

**중요 특징**
- `$`로 시작한다.
- 값은 문자열이든 숫자든 상관없이 그대로 들어간다.
- 값을 공백으로 구분하여 순서대로 $1, $2, $3…${10}번에 저장된다.

**기본 위치 매개변수 목록**
- **$0** : 스크립트 이름을 의미한다. (자기 자신)
- **$1, $2, $3 …** : 사용자가 입력한 값들을 순서대로 담는다.
- **$#** : 입력된 매개변수의 개수
- **$@** : 입력된 모든 매개변수를 각각 따로 출력한다. (예: "apple" "banana" "cherry")
- **$\*** : 입력된 모든 매개변수를 하나의 문자열처럼 출력한다. (예: "apple banana cherry")

```bash
[root@Server-A ~]# vi  ./script/test.sh
#!/bin/bash

echo "첫번째 값 : $1"
echo "첫번째 값 : $2"
echo "첫번째 값 : $3"
echo "매개변수 개수 : $#"
echo "매개변수 개별 값 : $@"
echo "매개변수 전체 값 : $*"

:wq
```

```bash
[root@Server-A ~]# ./script/test.sh		# 매개변수를 설정하지 않았기때문에 값이 출력되지 않는다.
첫번째 값 :
첫번째 값 :
첫번째 값 :
매개변수 개수 : 0
매개변수 개별 값 :
매개변수 전체 값 :

[root@Server-A ~]# chmod  +x  ./script/test.sh

[root@Server-A ~]# ./script/test.sh  apple  banana  cherry
첫번째 값 : apple
첫번째 값 : banana
첫번째 값 : cherry
매개변수 개수 : 3
매개변수 개별 값 : apple banana cherry
매개변수 전체 값 : apple banana cherry
```

```bash
[root@Server-A ~]#  cp    /etc/passwd    /temp/backup/passwd
                             ---  ------------   --------------------
                             $0          $1                       $2

[root@Server-A ~]# echo $PATH
/root/.local/bin:/root/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin

[root@Server-A ~]# ls  -l  /usr/bin/cp
-rwxr-xr-x. 1 root root 144400  6월 25 02:37 /usr/bin/cp
```

**정리**: `$0`은 스크립트 자신, `$1~$n`은 입력 인자, `$#`은 인자 개수, `$@`/`$*`는 전체 인자를 각각 개별/하나의 문자열로 다루는 특수 변수이다. 인자가 없으면 해당 변수들은 빈 값 또는 0으로 처리된다.

### 위치 매개변수 실습 예제

**EX1) 2개의 정수를 비교하여 두 숫자 중 더 큰 값 출력하기**
- 첫 번째 인자($1)와 두 번째 인자($2)를 숫자로 받아서, 둘 중 더 큰 값을 출력하시오.
- 인자가 2개가 아니면 사용법을 안내하고 종료한다.

```bash
[root@Server-A ~]# vi  ./script/positional_example01.sh
#!/bin/bash

if  [ $# -ne  2 ]
then
    echo "Usage : $0 number1  number2"		# 메세지 출력
    exit 1					# 스크립트 종료
fi

num1=$1
num2=$2

if [ "$num1" -gt "$num2" ]
then
    echo "큰 수 : $num1"
else
    echo "큰 수 : $num2"
fi
```

```bash
[root@Server-A ~]# chmod  +x   ./script/positional_example01.sh

[root@Server-A ~]# ./script/positional_example01.sh  100
Usage : ./script/positional_example01.sh number1  number2

[root@Server-A ~]# ./script/positional_example01.sh  100 200
큰 수 : 200
```

**EX2) 스크립트 실행시 인자로 받은 모든 숫자 합을 구해야 한다. (인자가 하나도 없으면 0을 출력)**

```bash
[root@Server-A ~]# vi  ./script/positional_example02.sh
#!/bin/bash

if ! [[ "$1" =~ ^[0-9]+$ ]] || ! [[ "$2" =~ ^[0-9]+$ ]]
then 
    echo "Usage : $0  int1  int2"
    exit 1
fi

echo "$1"
echo "$2"

:wq
```

```bash
[root@Server-A ~]# ./script/positional_example02.sh A B
Usage : ./script/positional_example02.sh  int1  int2

[root@Server-A ~]# ./script/positional_example02.sh 10 20
10
20
```

```bash
[root@Server-A ~]# vi  ./script/positional_example02.sh
#!/bin/bash

if ! [[ "$1" =~ ^[0-9]+$ ]] || ! [[ "$2" =~ ^[0-9]+$ ]]
then 
    echo "Usage : $0  int1  int2"
    exit 1
fi

sum=0

for  num  in  "$@"
do
    sum=$((sum + num))
done

echo "인자 개수 : $#"
echo "인자 총합 : $sum"

:wq
```

```bash
[root@Server-A ~]# ./script/positional_example02.sh 1 2 3 4 5	# 1 ~ 5 까지의 합
인자 개수 : 5
인자 총합 : 15

[root@Server-A ~]# ./script/positional_example02.sh {1..10}	# 1 ~ 10 까지의 합
인자 개수 : 10
인자 총합 : 55

[root@Server-A ~]# ./script/positional_example02.sh A B C		# 정수가 아니므로 메세지 출력
Usage : ./script/positional_example02.sh  int1  int2

[root@Server-A ~]# ./script/positional_example02.sh 10  20  A	# 첫번째 두번째 매개변수만 정수인지 확인하는 설정만 있다.
인자 개수 : 3
인자 총합 : 30
```

```bash
[root@Server-A ~]# vi  ./script/positional_example02.sh
#!/bin/bash

if (( $# == 0 ))			# 매개 변수가 없으면
then
        echo "Usage : $0 int1  int2"	# 메세지 출력
        exit 1			# 스크립트 출력
fi

for  num  in  "$@"
do
    if  ! [[  "$num" =~ ^[0-9]+$ ]]	# 매개 변수가 정수가 아니면
    then
        echo "Usage : $0 int1  int2"	# 메세지 출력
        echo "정수가 아닌값 : $num"	# 정수가 아닌 매개변수 출력
        exit 1			# 스크립트 출력
    fi
done

sum=0

for  num  in  "$@"
do
    sum=$((sum + num))
done

echo "인자 개수 : $#"
echo "인자 총합 : $sum"

:wq
```

```bash
[root@Server-A ~]# ./script/positional_example02.sh 10  20  A	# 정수가 아닌값을 매개변수로 사용시
Usage : ./script/positional_example02.sh int1  int2
정수가 아닌값 : A

[root@Server-A ~]# ./script/positional_example02.sh 10  20  30	# 모든 매개변수를 정수로 사용시
인자 개수 : 3
인자 총합 : 60
```

**EX3) 서비스 설치 및 제어 메뉴**
- 첫 번째 인자($1)는 install, start, stop, restart, status 중 하나를 입력받아야 한다.
- 두 번째 인자($2)는 제어할 서비스 이름을 입력받아야 한다.
- install을 입력하면 해당 서비스 패키지를 설치해야 한다.
- start, stop, restart, status를 입력하면 해당 서비스에 대해 systemctl 명령을 실행해야 한다.
- 서비스 이름으로 sshd가 입력되면 openssh-server 패키지를 설치해야 한다.
- 인자가 2개가 아니면 사용법을 출력하고 종료해야 한다.
- 잘못된 동작을 입력하면 오류 메시지를 출력하고 종료해야 한다

```bash
[root@Server-A ~]# vi ./script/positional_example03.sh
#!/bin/bash

if  [ $#  -ne  2 ]
then
    echo "Usage : $0 {install | start | stop | restart | status} service-name"
    exit 1
fi

action="$1"
service="$2"

case "$action" in
    install)
        if [ "$service" = "sshd" ]
        then
            package="openssh-server"
        else
            package="$service"
        fi

        echo "$package Package를 설치합니다."
        dnf  install  -y  $package
#      dnf  install  -y  $package  >  /dev/null 2>&1		# 설치과정 생략시 (에러도 생략)

        if [ $? -eq 0 ]
        then
            echo "$package Package가 설치 성공"
        else
            echo "$package Package가 설치 실패"
        fi
        ;;

    start)
        echo "$service 서비스 시작"
        systemctl  start  "$service"
        ;;

    stop)
        echo "$service 서비스 중지"
        systemctl  stop  "$service"
        ;;

    restart)
        echo "$service 서비스 재시작"
        systemctl  restart  "$service"
        ;;

    status)
        echo "$service 서비스 상태"
        systemctl  status  "$service"
        ;;

    *)
        echo "Unknown Action: $action"
        echo "Usage : $0 {install | start | stop | restart | status} service-name"
        exit 1
        ;;
esac
```

```bash
[root@Server-A ~]# ./script/positional_example03.sh		# 매개변수가 없으면 메세지 출력
Usage : ./script/positional_example03.sh {install | start | stop | restart | status} service-name


[root@Server-A ~]# ./script/positional_example03.sh  A B C	# 매개변수가 2개를 초과해도 메세지 출력
Usage : ./script/positional_example03.sh {install | start | stop | restart | status} service-name


[root@Server-A ~]# ./script/positional_example03.sh  run  httpd	# 잘못된 매개변수가 입력되어도 메세지 출력
Unknown Action: run
Usage : ./script/positional_example03.sh {install | start | stop | restart | status} service-name


[root@Server-A ~]# rpm  -qa  | grep httpd	# httpd 설치 확인
httpd-tools-2.4.62-13.el9_8.5.x86_64
rocky-logos-httpd-90.17-1.el9.noarch
httpd-filesystem-2.4.62-13.el9_8.5.noarch
httpd-core-2.4.62-13.el9_8.5.x86_64
httpd-2.4.62-13.el9_8.5.x86_64


[root@Server-A ~]# dnf  remove  -y  httpd	# httpd가 설치되어있으면 삭제


[root@Server-A ~]# ./script/positional_example03.sh  install  http	# httpd  install
httpd Package를 설치합니다.
마지막 메타자료 만료확인(2:12:32 이전): 2026년 07월 29일 (수) 오후 01시 52분 00초.
종속성이 해결되었습니다.
=======================================================================================
 꾸러미                  구조         버전                       저장소           크기
=======================================================================================
설치 중:
 httpd 

	~~~~~~~~~~ 중간 생략 ~~~~~~~~~~

설치되었습니다:
  apr-1.7.0-12.el9_3.x86_64                    apr-util-1.6.1-23.el9.x86_64
  apr-util-bdb-1.6.1-23.el9.x86_64             apr-util-openssl-1.6.1-23.el9.x86_64
  httpd-2.4.62-13.el9_8.5.x86_64               httpd-core-2.4.62-13.el9_8.5.x86_64
  httpd-filesystem-2.4.62-13.el9_8.5.noarch    httpd-tools-2.4.62-13.el9_8.5.x86_64
  mod_http2-2.0.26-6.el9_8.1.x86_64            mod_lua-2.4.62-13.el9_8.5.x86_64
  rocky-logos-httpd-90.17-1.el9.noarch

완료되었습니다!
httpd Package가 설치 성공


[root@Server-A ~]# ./script/positional_example03.sh  status  httpd		# httpd 상태 확인
httpd 서비스 상태
○ httpd.service - The Apache HTTP Server
     Loaded: loaded (/usr/lib/systemd/system/httpd.service; disabled; preset: disabled)
     Active: inactive (dead)
       Docs: man:httpd.service(8)


[root@Server-A ~]# ./script/positional_example03.sh  start  httpd		# httpd 서버 실행
httpd 서비스 시작


[root@Server-A ~]# ./script/positional_example03.sh  status  httpd		# httpd 상태 확인
httpd 서비스 상태
● httpd.service - The Apache HTTP Server
     Loaded: loaded (/usr/lib/systemd/system/httpd.service; disabled; preset: disabled)
     Active: active (running) since Wed 2026-07-29 16:05:44 KST; 13s ago
       Docs: man:httpd.service(8)
   Main PID: 3100 (httpd)
     Status: "Total requests: 0; Idle/Busy workers 100/0;Requests/sec: 0; Bytes served>
      Tasks: 177 (limit: 10322)
     Memory: 14.0M (peak: 14.3M)
        CPU: 62ms
     CGroup: /system.slice/httpd.service
             ├─3100 /usr/sbin/httpd -DFOREGROUND
             ├─3101 /usr/sbin/httpd -DFOREGROUND
             ├─3102 /usr/sbin/httpd -DFOREGROUND
             ├─3103 /usr/sbin/httpd -DFOREGROUND
             └─3104 /usr/sbin/httpd -DFOREGROUND

 7월 29 16:05:43 Server-A systemd[1]: Starting The Apache HTTP Server...
 7월 29 16:05:43 Server-A httpd[3100]: AH00558: httpd: Could not reliably determine th>
 7월 29 16:05:44 Server-A httpd[3100]: Server configured, listening on: port 80
 7월 29 16:05:44 Server-A systemd[1]: Started The Apache HTTP Server.


[root@Server-A ~]# ./script/positional_example03.sh  stop  httpd		# httpd 서버 중지
httpd 서비스 중지


[root@Server-A ~]# ./script/positional_example03.sh  status  httpd		# httpd 상태 확인
httpd 서비스 상태
○ httpd.service - The Apache HTTP Server
     Loaded: loaded (/usr/lib/systemd/system/httpd.service; disabled; preset: disabled)
     Active: inactive (dead)
       Docs: man:httpd.service(8)
```

**EX4) 백업 스크립트**
- 첫 번째 인자($1): 백업 대상 디렉터리
- 두 번째 인자($2): 백업 파일 저장 디렉터리
- 대상 디렉터리가 없으면 에러 출력 후 종료
- 백업 파일 이름은 디렉터리명_YYYYMMDD.tar.gz 형식으로 만든다.
- tar로 압축 백업을 수행한다.

```bash
[root@Server-A ~]# vi  ./script/positional_example04.sh
#!/bin/bash

if [ $# -ne 2 ]
then
        echo "Usage : $0 source_dir  destination_dir"
        exit 1
fi

src="$1"
dest="$2"

if [ ! -d "$src" ]
then
        echo "백업 대상 디렉터리가 없습니다. : $src"
        exit 1
fi

if [ ! -d "$dest" ]
then
        echo "백업 저장 디렉터리가 없습니다. : $dest"
        exit 1
fi

base=$(basename "$src")			# /home/guest -->  guest만 추출
today=$(date +%F_%T)			# 현재 날짜 및 시간을 변수에 저장
backup_file="${dest}/${base}_${today}.tar.gz"	# 백업할 경로와 파일명을 생성

echo "백업합니다. : $backup_file"

tar czf "$backup_file" "$src"	
# c	: 새로운 압축파일 생성
# z	: gzip으로 압축
# f	: 생성할 압축파일명 지정
```

```bash
[root@Server-A ~]# chmod +x  ./script/positional_example04.sh

[root@Server-A ~]# mkdir -p  /temp/sqlDB		# 백업 대상 디렉터리

[root@Server-A ~]# mkdir -p  /backup/mysqlDB_backup	# 백업 데이터를 저장할 디렉터리

[root@Server-A ~]# cp -r /etc/a*  /temp/sqlDB/


[root@Server-A ~]# ls  -l  /temp/sqlDB/
합계 36
drwxr-xr-x 3 root root   28  7월 29 17:08 accountsservice
-rw-r--r-- 1 root root   16  7월 29 17:08 adjtime
-rw-r--r-- 1 root root 1529  7월 29 17:08 aliases
drwxr-xr-x 3 root root   65  7월 29 17:08 alsa
drwxr-xr-x 2 root root 4096  7월 29 17:08 alternatives
-rw-r--r-- 1 root root  541  7월 29 17:08 anacrontab
-rw-r--r-- 1 root root  269  7월 29 17:08 anthy-unicode.conf
-rw-r--r-- 1 root root  833  7월 29 17:08 appstream.conf
-rw-r--r-- 1 root root   55  7월 29 17:08 asound.conf
-rw-r--r-- 1 root root    1  7월 29 17:08 at.deny
drwxr-x--- 4 root root  100  7월 29 17:08 audit
drwxr-xr-x 3 root root 4096  7월 29 17:08 authselect
drwxr-xr-x 4 root root   71  7월 29 17:08 avahi


[root@Server-A ~]# bash  ./script/positional_example04.sh   /temp/sqlDB/  /backup/mysqlDB_backup/  


[root@Server-A ~]# ./script/positional_example04.sh   /temp/sqlDB/  /backup/mysqlDB_backup/  


[root@Server-A ~]# ls -l /backup/mysqlDB_backup/
합계 24
-rw-r--r-- 1 root root 8437  7월 29 17:13 sqlDB_2026-07-29_17:13:39.tar.gz
-rw-r--r-- 1 root root 8437  7월 29 17:14 sqlDB_2026-07-29_17:14:47.tar.gz
```

**정리**: 위치 매개변수는 `$#`으로 개수를 검증하고, `[[ =~ ]]` 정규식으로 형식을 검증한 뒤, `"$@"`로 순회하며 값을 처리하는 패턴이 실무 스크립트의 기본 골격이 된다. `case`문과 결합하면 install/start/stop/restart/status 같은 서비스 제어 메뉴도 손쉽게 구현할 수 있고, `basename`/`date`와 조합하면 백업 파일명 자동 생성 같은 작업도 가능하다.

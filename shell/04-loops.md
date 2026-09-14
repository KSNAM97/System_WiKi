# Shell Script 반복문 (Loops)

## Looping

쉘 스크립트에서 반복문은 같은 코드를 여러 번 되풀이해서 실행하는 문법이다. **for**, **while**, **until** 세 가지가 대표적이며, 여러 서버에 같은 명령을 실행하거나 디렉터리 안의 파일을 일괄 처리하는 등 반복이 필요한 자동화 작업 대부분을 이 세 문법으로 처리할 수 있다.

### 반복문이란?

- 프로그램을 위에서 아래로 한 번만 실행하는 게 아니라, 같은 코드를 여러 번 반복해서 실행하게 만드는 문법
- 예를 들어 1부터 10까지 숫자를 순서대로 출력
- 특정 디렉터리 안의 파일들을 하나씩 검사
- 사용자가 특정 값을 입력할 때까지 계속 물어보기
- 같은 작업을 할 때 반복문을 사용한다.

**반복문이 왜 필요하냐?**

반복문이 없으면 이런 식으로 써야 한다.

```bash
echo 1
echo 2
echo 3
echo 4
echo 5
```

```bash
num=1

while [ $num -le 5 ]
do
    echo $num
    num=$((num + 1))
done
```

- 코드 길이가 짧고 수정/확장(예: 100까지)도 쉽고 사람이 실수할 가능성도 줄어든다.
- 쉘 스크립트에서 특정 작업을 반복적으로 수행하도록 만드는 제어문이다.
  - 1부터 10까지 숫자 출력
  - 파일 목록을 하나씩 처리
  - 사용자 계정 여러 개 생성
  - 로그 파일 여러 개를 순회
  - 백업 폴더의 파일을 반복 처리
  - 즉, 반복해야 하는 작업은 거의 모두 for문으로 구현 가능하다.

- Bash에서 자주 쓰는 반복문은 크게 3가지다. (**for**문, **while**문, **until**문)

- **for** 문
  - 정해진 목록을 하나씩 꺼내면서 반복
  - 예: 파일 목록, 숫자 리스트 등

- **while** 문
  - 조건이 참인 동안 계속 반복

- **until** 문
  - 조건이 참이 될 때까지 계속 반복
  - 즉, 조건이 거짓인 동안 반복

- **while** 과 **until** 은 조건식만 반대로 생각하면 된다.

**정리**: 반복문은 동일 작업을 여러 번 실행할 때 코드를 짧고 안전하게 만들어준다. for/while/until의 기본 성격 차이(목록 순회 vs 조건 반복 vs 조건 반전)를 이해하는 것이 이후 실습의 출발점이다.

## for 문

**for** 문은 정해진 목록의 값을 하나씩 꺼내면서 반복하는 문법이다.

- 숫자 목록, 문자열 목록, 파일 목록처럼 반복할 대상이 정해져 있을 때 사용한다.
- **for** 문은 목록의 첫 번째 값부터 마지막 값까지 순서대로 변수에 저장한 후 명령을 반복 실행한다.

### 기본 형식

```bash
for 변수 in 값1 값2 값3
do
반복할 명령
done
```

- **변수** : 목록에서 꺼낸 값이 임시로 저장된다.
- **in** : 반복할 값의 목록을 지정한다.
- **do** : 반복 실행할 명령의 시작을 의미한다.
- **done** : for 문의 종료를 의미한다.
- for 반복문의 기본 구조(형식)

Example)

```bash
[root@Server-A ~]# for item  in  A B C D
> do
>     echo $item
> done
A
B
C
D
```

- for문은 다음과 같은 상황에서 사용한다.
  - 정해진 목록을 하나씩 처리할 때
  - 특정 디렉터리 파일 리스트 반복
  - 사용자 계정 여러 개 생성
  - 여러 패키지를 반복 설치
  - 일정 범위를 순회할 때
  - 1~100까지 숫자 출력
  - 반복 체크(모니터링 도구 등)
  - 명령 결과를 반복 처리할 때
  - 즉, 반복이 필요한 거의 모든 자동화는 for문으로 처리한다.

### (1) 기본 목록 반복

```bash
[root@Server-A ~]# for  num  in 1 2 3 4 5
> do
>   echo "출력 : $num"
> done
출력 : 1
출력 : 2
출력 : 3
출력 : 4
출력 : 5
```

### (2) 범위(range) 표현 : {1..10}

```bash
[root@Server-A ~]# for  num  in {1..10}
> do
> echo "출력 : $num"
> done
출력 : 1
출력 : 2
출력 : 3
출력 : 4
출력 : 5
출력 : 6
출력 : 7
출력 : 8
출력 : 9
출력 : 10
```

### (3) 일정 간격(step) 반복 : {1..10..2}

```bash
[root@Server-A ~]# for num  in {1..10..2}
> do
>     echo "출력 : $num"
> done
출력 : 1
출력 : 3
출력 : 5
출력 : 7
출력 : 9
```

### (4) C언어 스타일 for 문

Bash는 C언어 같은 형태도 지원한다.

```bash
[root@Server-A ~]# for  ((i=1;  i<=5;  i++))	# i=1; = 초기식 ,   i<=5; = 조건식  i++ = 증감식
> do
>     echo "출력 : $i"
> done
출력 : 1
출력 : 2
출력 : 3
출력 : 4
출력 : 5
```

### (5) 배열 반복

```bash
[root@Server-A ~]# arr=("apple" "banana" "cherry")

[root@Server-A ~]# for  item  in "${arr[@]}"		# 배열 안의 값을 순차적으로 적용
> do
>     echo $item
> done
apple
banana
cherry


[root@Server-A ~]# for  item  in "${arr[*]}"		# 배열 안의 값을 한번에 적용
> do
>     echo $item
> done
apple banana cherry
```

### (6) 명령 결과 반복

```bash
[root@Server-A ~]# for  file  in  /etc/a*
> do
>   echo $file
> done
/etc/accountsservice
/etc/adjtime
/etc/aliases
/etc/alsa
/etc/alternatives
/etc/anacrontab
/etc/anthy-unicode.conf
/etc/appstream.conf
/etc/asound.conf
/etc/at.deny
/etc/audit
/etc/authselect
/etc/avahi
```

**정리**: for문은 기본 목록 반복, `{시작..끝}`/`{시작..끝..증분}` 범위 표현, C-style 반복, 배열 순회(`"${arr[@]}"` vs `"${arr[*]}"`), 명령 결과(글롭) 순회까지 다양한 형태를 지원한다. 상황에 맞는 목록 표현만 고르면 나머지 구조는 동일하다.

## 실습

### EX1) /etc 디렉터리 a로 시작하는 파일/디렉터리 삭제 및 출력

EX1) 아래의 조건에 맞게 구성해야 한다.
- /etc/ 디렉터리에서 파일 이름이 'a'로 시작하는 모든 파일 및 디렉터리를 삭제 해야 한다.
- 파일 및 디렉터리 생성시 삭제된 파일명이 출력되어야 한다.

```bash
[root@Server-A ~]# rm  -rf  /backup/*


[root@Server-A ~]# cp  -pr  /etc/a*  /backup


[root@Server-A ~]# ls  -l  /backup
합계 36
drwxr-xr-x 3 root root   28  7월  2 12:52 accountsservice
-rw-r--r-- 1 root root   16  7월  2 12:55 adjtime
-rw-r--r-- 1 root root 1529  6월 23  2020 aliases
drwxr-xr-x 3 root root   65  7월  2 15:07 alsa
drwxr-xr-x 2 root root 4096  7월  2 15:06 alternatives
-rw-r--r-- 1 root root  541 12월 30  2025 anacrontab
-rw-r--r-- 1 root root  269 10월 30  2022 anthy-unicode.conf
-rw-r--r-- 1 root root  833  2월 11  2023 appstream.conf
-rw-r--r-- 1 root root   55  1월 25  2026 asound.conf
-rw-r--r-- 1 root root    1 11월 12  2025 at.deny
drwxr-x--- 4 root root  100  7월  2 15:06 audit
drwxr-xr-x 3 root root 4096  7월  2 15:08 authselect
drwxr-xr-x 4 root root   71  7월  2 15:06 avahi


[root@Server-A ~]# for  filerm  in  /etc/a*
> do
>      echo "${filerm} 삭제"
>      rm  -rf  $filerm
> done
/etc/accountsservice 삭제
/etc/adjtime 삭제
/etc/aliases 삭제
/etc/alsa 삭제
/etc/alternatives 삭제
/etc/anacrontab 삭제
/etc/anthy-unicode.conf 삭제
/etc/appstream.conf 삭제
/etc/asound.conf 삭제
/etc/at.deny 삭제
/etc/audit 삭제
/etc/authselect 삭제
/etc/avahi 삭제


[root@Server-A ~]# cp -pr  /backup/*  /etc/


[root@Server-A ~]# ls  -l  /backup
합계 36
drwxr-xr-x 3 root root   28  7월  2 12:52 accountsservice
-rw-r--r-- 1 root root   16  7월  2 12:55 adjtime
-rw-r--r-- 1 root root 1529  6월 23  2020 aliases
drwxr-xr-x 3 root root   65  7월  2 15:07 alsa
drwxr-xr-x 2 root root 4096  7월  2 15:06 alternatives
-rw-r--r-- 1 root root  541 12월 30  2025 anacrontab
-rw-r--r-- 1 root root  269 10월 30  2022 anthy-unicode.conf
-rw-r--r-- 1 root root  833  2월 11  2023 appstream.conf
-rw-r--r-- 1 root root   55  1월 25  2026 asound.conf
-rw-r--r-- 1 root root    1 11월 12  2025 at.deny
drwxr-x--- 4 root root  100  7월  2 15:06 audit
drwxr-xr-x 3 root root 4096  7월  2 15:08 authselect
drwxr-xr-x 4 root root   71  7월  2 15:06 avahi
```

### EX2) 1부터 10까지의 합

EX2) for문을 사용하여 1부터 10까지의 합을 구하는 스크립트를 작성하시오

**방법 1**

```bash
[root@Server-A ~]# vi ./script/for_example.sh
#!/bin/bash

sum=0

for  num  in {1..10}
do
  echo "$sum + $num = $(( sum + num ))"
  sum=$(( sum + num ))
done

echo "1부터 10까지의 총 합 : ${sum}"

:wq

[root@Server-A ~]# chmod  +x  ./script/for_example.sh


[root@Server-A ~]# ./script/for_example.sh
0 + 1 = 1
1 + 2 = 3
3 + 3 = 6
6 + 4 = 10
10 + 5 = 15
15 + 6 = 21
21 + 7 = 28
28 + 8 = 36
36 + 9 = 45
45 + 10 = 55
1부터 10까지의 총 합 : 55
```

**방법 2**

```bash
[root@Server-A ~]# vi ./script/for_example2.sh
#!/bin/bash

sum=0

for  (( num=1;  num<=10;  num++))
do
  echo "$sum + $num = $(( sum + num ))"
  sum=$(( sum + num ))
done

echo "1부터 10까지의 총 합 : ${sum}"

:wq

[root@Server-A ~]# chmod  +x  ./script/for_example2.sh


[root@Server-A ~]# ./script/for_example2.sh
0 + 1 = 1
1 + 2 = 3
3 + 3 = 6
6 + 4 = 10
10 + 5 = 15
15 + 6 = 21
21 + 7 = 28
28 + 8 = 36
36 + 9 = 45
45 + 10 = 55
1부터 10까지의 총 합 : 55
```

### EX2-2) 입력받은 두 정수 사이의 합

EX2-2) 아래의 조건에 맞게 스크립트를 작성하시오
- 2개의 정수를 사용자로부터 입력받아 첫번째 정수부터 2번째 정수까지의 합을 출력해야 한다.
- 단 첫번째 정수보다 두번째 정수가 커야하며 첫번째 정수가 클경우 "두번째 정수가 더 커야합니다." 메세지가 출력되고 스크립트는 중단되어야 한다.

```bash
[root@Server-A ~]# vi  ./script/input_sum.sh
#!/bin/bash

read -p "첫 번째 정수 입력 : " num1
read -p "두 번째 정수 입력 : " num2

if [ "$num1" -gt "$num2" ]
then
    echo "두번째 정수가 더 커야합니다."
    exit 1
fi

sum=0

for ((i=num1; i<=num2; i++))		# {1..10}같은 중괄호 확장은 변수를 사용할 수 없다. | for  i  in  $(seq "num1"  "num2")
do
    sum=$((sum + i))
done

echo "$num1부터 $num2까지의 합 : $sum"

:wq


[root@Server-A ~]# chmod  +x  ./script/input_sum.sh


[root@Server-A ~]# ./script/input_sum.sh
첫 번째 정수 입력 : 10
두 번째 정수 입력 : 5
두번째 정수가 더 커야합니다.


[root@Server-A ~]# ./script/input_sum.sh
첫 번째 정수 입력 : 1
두 번째 정수 입력 : 15
1부터 15까지의 합 : 120
```

### EX4-1) 구구단 한 단 출력

EX4-1) 사용자로부터 정수를 입력받아 입력받은 정수의 단수를 출력하시오 (구구단)

```bash
[root@Server-A ~]# vi  ./script/for_example04_1.sh
#!/bin/bash

read  -p  "정수를 입력하세요(1~9) : " dan

echo "==============  ${dan} 단  =============="

for  i  in  {1..9}
do
    echo "$dan x  $i = $(( dan * i  ))"
done

: wq



[root@Server-A ~]# chmod +x ./script/for_example04_1.sh


[root@Server-A ~]# ./script/for_example04_1.sh
정수 1~9를 입력하세요(구구단) : 7
========= 7단 =========
7 X 1 = 7
7 X 2 = 14
7 X 3 = 21
7 X 4 = 28
7 X 5 = 35
7 X 6 = 42
7 X 7 = 49
7 X 8 = 56
7 X 9 = 63
```

### EX4-2) 2~9단 전체 구구단

EX4-2) 2 ~ 9까지의 구구단을 출력하시오

```bash
[root@Server-A ~]# vi  ./script/for_example04_2.sh
#!/bin/bash

for  num  in  {2..9}
do
    echo "======  ${num} 단  ======"
    
    for  i  in  {1..9}
    do
        echo "$num x $i = $(( $num * $i ))"
    done

    echo
done

:wq


[root@Server-A ~]# chmod  +x  ./script/for_example04_2.sh


[root@Server-A ~]# ./script/for_example04_2.sh
======  2 단  ======
2 x 1 = 2
2 x 2 = 4
2 x 3 = 6
2 x 4 = 8
2 x 5 = 10
2 x 6 = 12
2 x 7 = 14
2 x 8 = 16
2 x 9 = 18

~~~ 중간 생략 ~~~

======  9 단  ======
9 x 1 = 9
9 x 2 = 18
9 x 3 = 27
9 x 4 = 36
9 x 5 = 45
9 x 6 = 54
9 x 7 = 63
9 x 8 = 72
9 x 9 = 81
```

### EX5) 짝수/홀수 판별

EX5) 2개의 정수를 입력받아 스크립트를 구성하시오
- 2개의 정수를 사용자로부터 입력받아 첫번째 정수부터 2번째 정수까지 각 숫자가 짝수인지 홀수인지 if문으로 판단하여 다음과 같이 출력하는 스크립트를 작성하시오
- 단 첫번째 정수보다 두번째 정수가 커야하며 첫번째 정수가 클경우 "두번째 정수가 더 커야합니다." 메세지가 출력되고 스크립트는 중단되어야 한다.

```bash
[root@Server-A abc]# vi ./script/for_example05.sh
#!/bin/bash

read -p "첫 번째 정수 입력 : " num1
read -p "두 번째 정수 입력 : " num2

if [ "$num1" -gt "$num2" ]
then
    echo "두번째 정수가 더 커야합니다."
    exit 1
fi

for ((i=num1; i<=num2; i++))
do
    if (( i % 2 == 0 ))
    then
        echo "$i : 짝수"
    else
        echo "$i : 홀수"
    fi
done

:wq

[root@Server-A abc]# chmod +x ./script/for_example05.sh


[root@Server-A abc]# ./script/for_example05.sh
1은(는) 홀수 입니다.
2은(는) 짝수 입니다.
3은(는) 홀수 입니다.
4은(는) 짝수 입니다.
5은(는) 홀수 입니다.
6은(는) 짝수 입니다.
7은(는) 홀수 입니다.
8은(는) 짝수 입니다.
9은(는) 홀수 입니다.
10은(는) 짝수 입니다.
```

**정리**: EX1~EX5는 for문으로 파일 삭제/백업, 합계 계산, 사용자 입력 검증, 구구단(중첩 for), 짝수/홀수 판별까지 실무에서 자주 쓰는 패턴을 다룬다. C-style for(`((i=num1; i<=num2; i++))`)는 변수 범위를 다뤄야 할 때 `{시작..끝}` 중괄호 확장 대신 사용한다.

## break, continue

**break**, **continue**는 반복문의 흐름을 바꾸는 명령어이다.

- **break**
  : for, while, until과 같은 반복문은 break를 만나게되면 반복문을 종료한다.
    (스크립트를 종료하는것이 아닌 반복문을 종료한다.)

- **continue**
  : for, while, until과 같은 반복문은 continue를 만나게되면 continue아래의 명령은 실행하지 않고 다시 반복문의 시작점으로 돌아간다.

### EX6) break로 반복 종료

EX6) for문을 사용하여 1부터 10까지 숫자를 출력하되, 숫자가 5가 되면 그 이후 숫자는 출력하지 않고 반복문을 종료하는 스크립트를 작성

```bash
[root@Server-A ~]# vi  ./script/for_example06.sh
#!/bin/bash

for  num  in {1..10}
do
        if [ $num -eq 5 ]
        then
                echo "숫자 5를 만나서 반복문을 종료합니다."
                break
        fi
        echo $num
done

:wq


[root@Server-A ~]# chmod  +x  ./script/for_example06.sh

[root@Server-A ~]# ./script/for_example06.sh
1
2
3
4
숫자 5를 만나서 반복문을 종료합니다.
```

### EX7) 배열 + break로 passwd 파일 검색

EX7) 루트 디렉터리의 항목을 배열에 저장한 후 passwd 파일을 검색하는 스크립트를 작성하시오.
- / 디렉터리 바로 아래에 있는 모든 파일과 디렉터리 정보를 배열에 저장해야 한다.
- 배열에 저장된 항목을 for 문으로 하나씩 확인해야 한다.
- 각 항목이 디렉터리이면 해당 디렉터리 바로 아래에서 passwd 파일을 검색해야 한다.
- passwd 파일을 발견하면 파일의 전체 경로를 출력해야 한다.
- passwd 파일을 발견한 후에는 break 문으로 반복문을 종료해야 한다.

**특정 경로안의 파일 또는 디렉터리 정보를 배열에 저장**

```bash
[root@Server-A ~]# arr=(/etc/a*)

[root@Server-A ~]# echo ${arr[@]}
/etc/accountsservice /etc/adjtime /etc/aliases /etc/alsa /etc/alternatives /etc/anacrontab /etc/anthy-unicode.conf 
/etc/appstream.conf /etc/asound.conf /etc/at.deny /etc/audit /etc/authselect /etc/avahi


[root@Server-A ~]# vi  ./script/for_example07.sh
#!/bin/bash

items=(/*)

for  item  in "${items[@]}"
do
    if [ -d "$item" ]					# 해당 값이 디렉터리인지 파일인지 확인 (passwd파일은 디렉터리에만 존재 가능)
    then
        if [ -f "$item/passwd" ]				# 해당 디렉터리안에 passwd 파일이 있는지 확인
        then
            echo "passwd 파일 검색 완료 : $item/paswd"	# passwd 파일이 있으면 경로/파일명 출력
            break					# 반복문 종료
        fi
    fi
done

:wq


[root@Server-A ~]# chmod  +x  ./script/for_example07.sh


[root@Server-A ~]# ./script/for_example07.sh
passwd 파일 검색 완료 : /bin/paswd
```

### EX8) continue로 3의 배수 건너뛰기

EX8) for문으로 1부터 15까지 숫자를 출력하되, 3의 배수는 출력하지 않고 건너뛰는 스크립트를 작성

```bash
[root@Server-A ~]# vi  ./script/for_example08.sh
#!/bin/bash

for  num  in  {1..15}
do
    if  [ $(( num % 3 )) -eq 0 ]
    then
        echo "3의 배수인 $num 점프"
        continue
    fi
    echo  $num
done

:wq


[root@Server-A ~]# chmod  +x  ./script/for_example07.sh
1
2
3의 배수인 3 점프
4
5
3의 배수인 6 점프
7
8
3의 배수인 9 점프
10
11
3의 배수인 12 점프
13
14
3의 배수인 15 점프
```

### EX9) 로그 파일 용량 점검

EX9) /var/log 디렉터리의 여러 로그 파일 크기를 확인하는 스크립트를 작성하시오.
- /var/log/messages, /var/log/secure, /var/log/cron 파일을 for 문으로 하나씩 확인해야 한다.
- 파일이 존재하지 않으면 "파일 없음"을 출력해야 한다.
- 파일 크기가 10MB 이상이면 "용량 경고"를 출력해야 한다.
- 파일 크기가 10MB 미만이면 "정상"을 출력해야 한다.
- 파일 크기는 du -m 명령을 사용하여 확인해야 한다.

- 실제는 MB 또는 GB를 사용하지만 실습환경은 많은 용량을 사용하지 않기때만에 KB를 사용해 실습

```bash
[root@Server-A ~]# du -h /var/log/messages	# -h	:  사람이 보기 편한 방식으로 출력
188K    /var/log/messages


[root@Server-A ~]# du -k /var/log/messages	# -k	: KB단위로 출력
188     /var/log/messages



[root@Server-A ~]# du -m /var/log/messages	# -m	: MB단위로 출력
1       /var/log/messages



[root@Server-A ~]# du -k /var/log/messages | awk '{print $1}'	# 첫번째 필드만 출력
188



[root@Server-A ~]# vi  ./script/for_example09.sh
#!/bin/bash

for  file  in  /var/log/messages  /var/log/secure    /var/log/soldesk   /var/log/cron
do
    if  [ ! -f  "$file" ]
    then  
        echo "$file 파일 또는 디렉터리가 없습니다."
        continue
    fi

    size=$(du  -k  "$file"  |  awk  '{print $1}')

    if  [ "$size" -ge  10 ]
    then
        echo "$file : 용량 초과 (${size}KB)"
    else 
        echo "$file : 용량 정상 (${size}KB)"
    fi
done

:wq


[root@Server-A ~]# chmod  +x  ./script/for_example09.sh


[root@Server-A ~]# ./script/for_example09.sh
/var/log/messages : 용량 초과 (188KB)
/var/log/secure : 용량 정상 (4KB)
/var/log/soldesk 파일 또는 디렉터리가 없습니다.
/var/log/cron : 용량 정상 (4KB)
```

### EX10) 사용자 홈 디렉터리 소유권/권한 점검 및 수정

EX10) 여러 사용자 홈 디렉터리의 소유권과 권한을 검사하고 수정하는 스크립트를 작성하시오.
- user1 ~ user10 계정을 for 문으로 하나씩 확인해야 한다.
- 계정이 존재하지 않으면 "계정 없음"을 출력해야 한다.
- 계정의 홈 디렉터리가 없으면 생성해야 한다.
- 홈 디렉터리 소유권이 사용자 계정과 다르면 올바르게 변경해야 한다.
- 홈 디렉터리 권한이 700이 아니면 700으로 변경해야 한다.

**사전 설정**

```text
# user1	: 정상 계정, 홈 디렉터리 권한 700
# user2	: 정상 계정, 홈 디렉터리 권한 755
# user3	: 계정 없음
# user4	: 홈 디렉터리 소유자가 root
# user5	: 계정 없음
# user6	: 계정은 존재하지만 홈 디렉터리 없음
# user7	: 계정 없음
# user8	: 홈 디렉터리 권한 777
# user9	: 계정 없음
# user10	: 소유자가 root이고 권한이 755
```

```bash
[root@Server-A ~]# useradd user1
[root@Server-A ~]# chmod 700 /home/user1

[root@Server-A ~]# useradd user2
[root@Server-A ~]# chmod 755 /home/user2

[root@Server-A ~]# useradd user4
[root@Server-A ~]# chown root:root /home/user4
[root@Server-A ~]# chmod 700 /home/user4

[root@Server-A ~]# useradd user8
[root@Server-A ~]# chmod 777 /home/user8

[root@Server-A ~]# useradd user10
[root@Server-A ~]# chown root:root /home/user10
[root@Server-A ~]# chmod 755 /home/user10


[root@Server-A ~]# ls -ld  /home/user*
drwx------ 3 user1 user1 92  7월 28 14:32 /home/user1
drwxr-xr-x  3 root  root  92  7월 28 14:33 /home/user10
drwxr-xr-x  3 user2 user2 92  7월 28 14:33 /home/user2
drwx------  3 root  root  92  7월 28 14:33 /home/user4
drwxrwxrwx 3 user8 user8 92  7월 28 14:33 /home/user8




[root@Server-A ~]# vi  ./script/for_example10.sh
#!/bin/bash

for  user  in  user{1..10}
do
    if  ! id "$user"  >  /dev/null  2>&1	# id 명령어로 user1 ~ user10으로 검색시 계정이 없으면
    then
        useradd  "$user"		# useradd 명령어로 계정 생성

        if [ $? -eq 0 ]
        then
            echo "$user 계정 생성 완료"
        else
            echo "$user 계정 생성 실패"
            continue
        fi
    fi

    home="/home/$user"

    if  [ ! -d  "$home" ]
    then
        mkdir -p "$home"

        if [ $? -eq 0 ]
        then
            echo "$user 홈 디렉터리 생성 완료 ($home)"
        else
            echo "$user 홈 디렉터리 생성 실패 ($home)"
            continue
        fi
    fi

    owner=$(stat -c "%U"  "$home")

# -c 	: 출력 형식을 지정
# %U 	: 소유자 이름       	예: user1
# %G 	: 소유 그룹 이름    	예: user1
# %a 	: 숫자 허가권       	예: 700
# %n	: 파일 또는 경로명	예: /home/user1

    if [ "$owner" != "$user" ]
    then
        chown  "$user:$user"  "$home"

        if [ $? -eq 0 ]
        then
            echo "$user 홈 디렉터리 소유권 변경 완료"
        else
            echo "$user 홈 디렉터리 소유권 변경 실패"
            continue
        fi
    fi


    permission=$(stat -c "%a"  "$home")		# rwx rwx rwx확인 EX) 700

    if [ "$permission" -ne 700 ]		# -eq = equal, -ne = net equal
    then
        chmod  700  "$home"

        if [ $? -eq 0 ]
        then
            echo "$user 홈 디렉터리 허가권 변경 완료"
        else
            echo "$user 홈 디렉터리 허가권 변경 실패"
        fi
    fi

done




[root@Server-A ~]# ls -ld  /home/user*
drwx------  3 user1 user1 92	7월 28 14:32 /home/user1
drwxr-xr-x  3 root  root  92	7월 28 14:33 /home/user10
drwxr-xr-x  3 user2 user2 92	7월 28 14:33 /home/user2
drwx------  3 root  root  92 	7월 28 14:33 /home/user4
drwxrwxrwx 3 user8 user8 92	7월 28 14:33 /home/user8


[root@Server-A ~]# chmod  +x  ./script/for_example10.sh


[root@Server-A ~]#   ./script/for_example10.sh
user2 홈 디렉터리 허가권 변경 완료
user3 계정 생성 완료
user4 홈 디렉터리 소유권 변경 완료
user5 계정 생성 완료
user6 계정 생성 완료
user7 계정 생성 완료
user8 홈 디렉터리 허가권 변경 완료
user9 계정 생성 완료
user10 홈 디렉터리 소유권 변경 완료
user10 홈 디렉터리 허가권 변경 완료


[root@Server-A ~]# ls  -ld  /home/user*
drwx------ 3 user1   user1  92  7월 28 14:32 /home/user1		# 허가권 : 700, 소유권:소유그릅 (모두 해당 계정으로 확인)
drwx------ 3 user10 user10 92  7월 28 14:33 /home/user10		# 허가권 : 700, 소유권:소유그릅 (모두 해당 계정으로 확인)
drwx------ 3 user2   user2  92  7월 28 14:33 /home/user2		# 허가권 : 700, 소유권:소유그릅 (모두 해당 계정으로 확인)
drwx------ 3 user 3  user3  92  7월 28 15:21 /home/user3		# 허가권 : 700, 소유권:소유그릅 (모두 해당 계정으로 확인)
drwx------ 3 user4   user4  92  7월 28 14:33 /home/user4		# 허가권 : 700, 소유권:소유그릅 (모두 해당 계정으로 확인)
drwx------ 3 user5   user5  92  7월 28 15:21 /home/user5		# 허가권 : 700, 소유권:소유그릅 (모두 해당 계정으로 확인)
drwx------ 3 user6   user6  92  7월 28 15:21 /home/user6		# 허가권 : 700, 소유권:소유그릅 (모두 해당 계정으로 확인)
drwx------ 3 user7   user7  92  7월 28 15:21 /home/user7		# 허가권 : 700, 소유권:소유그릅 (모두 해당 계정으로 확인)
drwx------ 3 user8   user8  92  7월 28 14:33 /home/user8		# 허가권 : 700, 소유권:소유그릅 (모두 해당 계정으로 확인)
drwx------ 3 user9   user9  92  7월 28 15:21 /home/user9		# 허가권 : 700, 소유권:소유그릅 (모두 해당 계정으로 확인)
```

### EX3) 아이디/비밀번호 3회 확인

EX3) 아이디 비밀번호를 최대 3번까지 확인하는 스크립트
- 아이디, 비밀번호을 맞출 때까지 사용자에게 비밀번호를 입력받는 스크립트를 작성하시오
- 단, 시도 횟수는 최대 3번까지만 허용하며, 3번 모두 틀리면 "접속 실패"를 출력하고 스크립트가 종료해야 한다.

**방법 1**

```bash
[root@Server-A ~]# vi ./script/while_example03.sh
#!/bin/bash

id="user1" 
pw="admin1234" 
count=1

while [ "$count"  -le  3 ]
do

    read  -p  "아이디 : " input_id
    read -s -p  "비빌번호 : " input_pw
    echo

    if [ "$input_id" = "$id" ] && [ "$input_pw" = "$pw" ]
    then
        echo "로그인 성공"
        break
    else 
        echo "아이디 또는 비밀번호를 확인하세요"
        if [ "$count"  -eq  3 ]
        then
            echo "아이디 또는 비밀번호를 3회 실패하여 종료합니다."
            exit 1
        fi
        ((count++))
    fi
done

echo "${id}님이 로그인 하셨습니다."

:wq
```

**방법 2**

```bash
[root@Server-A ~]# vi ./script/while_example03.sh

#!/bin/bash

id="user1"
pw="1234"
count=1

while [ $count -le 3 ]
do
        read -p "아이디 :" input_id
        read -sp "패스워드 :" input_pass

        if [ "$input_id" == "$id" ] && [ "$input_pass" == "$pw" ]
        then
                echo "로그인 성공"
                exit 0
        else
                echo "로그인 실패 ($count/3)"
        fi

        count=$((count + 1))

done
exit 1

:wq

[root@Server-A ~]# chmod  +x  ./script/while_example03.sh

[root@Server-A ~]# ./script/while_example03.sh
아이디 : user2
비빌번호 : 

아이디 또는 비밀번호를 확인하세요
아이디 : user2
비빌번호 : 

아이디 또는 비밀번호를 확인하세요
아이디 : user2
비빌번호 : 

아이디 또는 비밀번호를 확인하세요
아이디 또는 비밀번호를 3회 실패하여 종료합니다.




[root@Server-A ~]# ./script/while_example03.sh
아이디 : user1
비빌번호 :
로그인 성공
user1님이 로그인 하셨습니다.
```

### EX4) 중복 시 번호를 붙여 파일/디렉터리 생성

EX4) 파일 또는 디렉터리를 생성하는 스크립트 생성
- 생성할 데이터가 파일인지 디렉터리인지 입력받아야 한다.
- 생성할 파일 또는 디렉터리의 경로를 입력받아야 한다. (없으면 경로가 생성되어야 한다.)
- 생성할 파일 또는 디렉터리가 존재하게되면 "파일명 또는 디렉터리명+숫자" 형태로 생성되어야한다. (중복시 숫자는 1씩 증가해야 한다.)

- **basename**은 전체 경로에서 마지막 파일명 또는 디렉터리명만 출력하는 명령어

```bash
[root@Server-A ~]# echo $(basename  /etc/ssh/sshd_config)
sshd_config
```

- **dirname**은 전체 경로에서 마지막 이름을 제외하고 상위 디렉터리 경로만 출력하는 명령어

```bash
[root@Server-A ~]# echo $(dirname  /etc/ssh/sshd_config)
/etc/ssh
```

```bash
[root@Server-A ~]# vi ./script/while_example04.sh
#!/bin/bash

read  -p  "생성할 파일(f) 또는 디렉터리(d) 입력 : " type
read  -p  "생성할 전체 경로 입력 : " path

num=1

if [ -e "$path" ]			# EX) /etc/ssh/sshd_config
then
    while [ -e "$path$num" ]		# EX) /etc/ssh/sshd_config1  /etc/ssh/sshd_config2  /etc/ssh/sshd_config3
    do
        ((num++))
    done
    path="${path}${num}"
fi

if [ "$type" = "f" ] 
then
    mkdir  -p  "$(dirname $path)"
    touch  "$path"
    echo "파일 생성 완료 ($path)"

elif [ "$type" = "d" ] 
then
    mkdir  -p  "$path"
    echo "디렉터리 생성 완료 ($path)"
else
    echo "f또는 d만 입력해야합니다."
    exit 1
fi

echo "파일 또는 디렉터리 생성 작업이 완료되었습니다."


[root@Server-A ~]# chmod  +x  ./script/while_example04.sh



[root@Server-A ~]# ./script/while_example04.sh
생성할 파일(f) 또는 디렉터리(d) 입력 : f
생성할 전체 경로 입력 : /etc/ssh/sshnwe_file
파일 생성 완료 (/etc/ssh/sshnwe_file)		# sshnwe_file 생성
파일 또는 디렉터리 생성 작업이 완료되었습니다.


[root@Server-A ~]# ./script/while_example04.sh
생성할 파일(f) 또는 디렉터리(d) 입력 : f
생성할 전체 경로 입력 : /etc/ssh/sshnwe_file
파일 생성 완료 (/etc/ssh/sshnwe_file1)		# sshnwe_file1 생성
파일 또는 디렉터리 생성 작업이 완료되었습니다.


[root@Server-A ~]# ./script/while_example04.sh
생성할 파일(f) 또는 디렉터리(d) 입력 : f
생성할 전체 경로 입력 : /etc/ssh/sshnwe_file
파일 생성 완료 (/etc/ssh/sshnwe_file2)		# sshnwe_file2 생성
파일 또는 디렉터리 생성 작업이 완료되었습니다.
```

**정리**: break는 반복문 전체를 즉시 종료할 때, continue는 특정 조건에서 현재 순환만 건너뛰고 다음으로 넘어갈 때 사용한다. EX6~EX10, 그리고 while 기반의 EX3·EX4는 로그인 재시도, 파일명 중복 처리처럼 실무형 조건 분기와 조합된 반복 제어 패턴을 보여준다.

## until 반복문

**until** 의 개념은 **while** 과 반대다.

- 조건이 참이 될 때까지 반복한다.
- 즉, 조건이 거짓인 동안 계속 반복 조건이 참이 되는 순간 반복 종료

**기본 형식(문법)**

```bash
until [ 조건식 ]
do
    반복해서 실행할 명령들
done
```

- [ 조건식 ] 참이면 반복 끝
- [ 조건식 ] 거짓이면 블록 실행 후 다시 조건 확인

### EX1) 10부터 1까지 카운트다운

EX1) 10부터 1까지 내려가며 출력하기
- 변수 num을 10으로 시작해서, num이 0보다 작아질 때까지 10, 9, 8, …, 1 을 출력하는 스크립트를 작성하시오.

**until 문**

```bash
[root@Server-A ~]# vi ./script/util_example01.sh
#!/bin/bash

num=10

until  [ "$num" -lt 0 ]
do
    echo "카운트다운 : $num"
    num=$(( num - 1))
    sleep 1
done

echo "카운트다운 종료"

:wq


[root@Server-A ~]# chmod  +x  ./script/util_example01.sh


[root@Server-A ~]# ./script/util_example01.sh
카운트다운 : 10
카운트다운 : 9
카운트다운 : 8
카운트다운 : 7
카운트다운 : 6
카운트다운 : 5
카운트다운 : 4
카운트다운 : 3
카운트다운 : 2
카운트다운 : 1
카운트다운 : 0
카운트다운 종료
```

**while 문**

```bash
[root@Server-A ~]# vi ./script/util_example01.sh
#!/bin/bash

num=10

while  [ "$num" -ge 0 ]
do
    echo "카운트다운 : $num"
    num=$(( num - 1))
    sleep 1
done

echo "카운트다운 종료"

:wq
```

### EX2) 특정 파일이 생길 때까지 대기

EX2) 특정 파일이 생길 때까지 대기하기
- /temp/ready.flag 라는 파일이 생성될 때까지 3초 간격으로 계속 확인한다.
- 파일이 없으면 "파일이 아직 없습니다. 대기 중..." 이라고 출력하고 후 다시 검사해야 한다.
- 파일이 생기면 "파일이 감지되었습니다. 작업을 종료합니다." 를 출력하고 종료한다.

```bash
[root@Server-A ~]# vi ./script/util_example02.sh
#!/bin/bash

file="/temp/ready.flag"

echo "$file 파일이 생성될때까지 감지합니다."

until  [ -f  "$file" ]
do
    echo "$file 파일이 아직 없습니다.  대기중...."
    sleep 3
done

echo "${file} 파일이 생성되었습니다."

:wq



[root@Server-A ~]# chmod  +x  ./script/util_example02.sh


[root@Server-A ~]# ./script/util_example02.sh
/temp/ready.flag 파일이 생성될때까지 감지합니다.
/temp/ready.flag 파일이 아직 없습니다.  대기중....
/temp/ready.flag 파일이 아직 없습니다.  대기중....
/temp/ready.flag 파일이 아직 없습니다.  대기중....



[root@Server-A ~]# touch /temp/ready.flag	# 다른 세션으로 접속 후 파일  또는 디렉터리 생성


[root@Server-A ~]# ./script/util_example02.sh
/temp/ready.flag 파일이 생성될때까지 감지합니다.
/temp/ready.flag 파일이 아직 없습니다.  대기중....
/temp/ready.flag 파일이 아직 없습니다.  대기중....
/temp/ready.flag 파일이 아직 없습니다.  대기중....
/temp/ready.flag 파일이 아직 없습니다.  대기중....
/temp/ready.flag 파일이 아직 없습니다.  대기중....
/temp/ready.flag 파일이 생성되었습니다.
```

**정리**: until은 "조건이 거짓인 동안" 반복한다는 점에서 while과 반대다. 카운트다운처럼 while로도 표현 가능한 로직을 until로 뒤집어 쓸 수도 있고, 파일 생성 대기처럼 "조건이 참이 될 때까지 기다리는" 상황에는 until이 더 직관적이다.

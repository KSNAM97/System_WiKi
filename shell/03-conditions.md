# Shell Script 조건문 (Conditions)

## expr, let (산술 연산)

### expr (Expression evaluator)

- `expr`는 리눅스에서 간단한 연산을 수행하는 명령어이다.
- 이름 그대로 expression evaluator(표현식 계산기) 역할을 한다.
- 터미널에서 정수 계산, 비교 연산, 문자열 처리 같은 간단한 연산을 할 때 사용된다.
- 쉘에서 `*`, `>`, `<` 같은 기호는 메타문자이므로 반드시 `\`로 escape 해야 한다.
- `expr`은 표현식을 평가하는 도구로 쉘에서 가장 오래된 외부 명령어 기반 산술연산기이다.

형식: `expr 값1 연산자 값2`

```bash
expr 3 + 4
expr 10 \* 2
expr 7 / 2
```

**expr 특징**
- 띄어쓰기 필수 (`expr 3+7`은 안 됨)
- 연산자는 문자열로 인식되므로 반드시 공백 필요
- 곱하기(`*`)는 쉘 메타문자이므로 escape 해야 한다
- `expr`은 연산 결과를 표준출력으로 출력할 뿐, 변수에는 넣지 않는다.
- 사용자는 `$( )`로 감싸서 변수에 저장해야 한다.

```bash
sum=$(expr 10 + 20)
echo $sum
```

**expr 예제**

두 변수의 합 구하기
```bash
# a=7
# b=5
# expr $a + $b
12
```

곱셈
```bash
[root@Server-A ~]# expr $a \* $b
35
```

연산 결과 변수에 저장
```bash
[root@Server-A ~]# total=$(expr $a + $b)
[root@Server-A ~]# echo "결과: $total"
결과: 12
```

**expr 실습 문제**

- EX1) 변수 a=7, b=13일 때, 두 값을 더해서 결과를 화면에 출력하시오. (expr 사용)
- EX2) 변수 x=30, y=12일 때, x에서 y를 뺀 값을 expr로 계산해서 출력하시오.
- EX3) 변수 a=4, b=6일 때, 두 값을 곱한 결과를 expr로 출력하시오. (곱셈 연산자 `*`는 쉘 메타문자이므로 escape 주의)
- EX4) 변수 total=50, count=4일 때, 평균값(정수 기준)을 expr로 계산해 출력하시오.
- EX5) 변수 num=29, divider=5일 때, 나머지를 expr로 구해 출력하시오.
- EX6) 변수 a=10, b=25가 있을 때, a와 b의 합을 expr로 계산하고 그 결과를 sum이라는 변수에 저장한 후 sum을 echo로 출력하시오.
- EX7) 변수 i=7이라고 할 때, expr을 사용하여 i 값을 1 증가시키고, 증가된 값을 다시 i에 저장한 뒤 i를 출력하시오.
- EX8) 변수 a=5, b=3, c=2일 때, `(a + b) * c`를 expr로 계산하여 결과를 출력하시오.
- EX9) 변수 x=10, y=10일 때, x와 y가 같은지 expr로 비교하고 결과(0 또는 1)를 출력하시오. (같으면 1, 다르면 0)
- EX10) 변수 x=8, y=3일 때, x가 y보다 큰지 expr로 비교하고 결과(1 또는 0)를 출력하시오.
- EX11) 변수 x=5, y=5일 때, x가 y보다 크거나 같은지 expr로 비교하고 결과를 출력하시오.

### let

- `let`은 Bash에서 정수 산술 계산을 수행하는 명령어이다.
- 숫자 계산을 할 때 사용하며 변수 값을 변경하거나 추가 계산을 쉽게 수행한다.

```bash
let x=3+5
let x++
let y=x*2
```

**let의 특징**
- 정수(Integer) 계산만 가능, 소수점 연산 불가 (EX. `5/2` → `2`, 소수점 버림)
- 변수 앞에 `$` 필요 없음 (EX. `let x=5+3`, `let y=x*2`)
- 공백이 있으면 반드시 따옴표 필요
  - 잘못된 예: `let x = 5 + 3` → 오류
  - 올바른 예: `let "x = 5 + 3"`
- 계산만 수행하고 출력은 하지 않음 — 값을 확인하려면 `echo $x` 사용
- `let 5 + 3`처럼 사용하면 가능은 하지만 어디에도 저장하지 않고 출력도 하지 않은 채 종료 코드 0만 남긴다.

**let 실습 문제**

- EX1) 변수 x에 10을 저장하고 5를 더한 결과 출력
- EX2) 변수 a=7, 3을 빼서 새로운 값으로 만들기
- EX3) 변수 n을 1 증가시키기 (`++` 사용)
- EX4) 변수 count를 1 감소시키기 (`--` 사용)
- EX5) 변수 a=6, b=3 두 값을 곱해 c에 저장
- EX6) 변수 x=15을 2로 나눈 몫으로 갱신
- EX7) 변수 y=17, 4로 나눈 나머지를 r에 저장
- EX8) 변수 m=8, m에 3을 곱한 값을 다시 m에 저장
- EX9) 변수 x=10, y=20 비교 (x > y)
- EX10) base=5, base²(제곱) 계산하여 갱신

**정리**: `expr`과 `let`으로 쉘에서 정수 산술 연산을 수행하는 방법을 살펴봤다. 이어서 명령의 종료 코드(exit)와 조건 판별(test)을 다룬다.

---

## exit 코드 & test 명령

### exit (종료 상태 전달)

리눅스/유닉스에서는 명령이 끝날 때 항상 숫자 하나를 남기고 사라진다. 이 숫자를 **종료 코드**(exit status) 또는 리턴 코드라고 부른다. 조건문은 이 종료 코드나 파일 존재 여부, 숫자·문자열 비교 결과를 바탕으로 이후 실행 흐름을 분기할 때 사용된다.

- 리턴 코드는 명령이 성공했는지, 실패했는지, 왜 실패했는지를 알려주는 중요한 정보다.
- 셸(Bash)은 마지막 명령의 종료 코드를 자동으로 **`$?`** 라는 변수에 저장해 둔다.

### exit 코드 범위와 의미

- **exit = 0**
  - 성공(success)을 의미
  - 프로그램 또는 명령이 정상적으로 실행됨
- **exit = 1 ~ 255**
  - 실패(failure)를 의미
  - 각 숫자에 따라 실패 원인이 달라질 수 있다.

대표적인 종료 코드들은 다음과 같다.

| 종료 코드 | 의미 |
|---|---|
| 0 | 성공 (Success) |
| 1 | 일반적인 에러 (모호한 오류) |
| 2 | Syntax error (사용법/옵션/문법 오류) |
| 126 | 명령을 실행할 수 없음 (Permission 문제 등) |
| 127 | 명령(파일)이 존재하지 않음 (command not found) |
| 128 + N | 신호(Signal)에 의해 종료됨 |

- **Ctrl + C** : SIGINT (신호 번호 2) → 종료 코드 = 128 + 2 = 130
- **kill -9 PID** : SIGKILL (신호 번호 9) → 종료 코드 = 128 + 9 = 137

**`$?` 동작 방식**
- 셸에서는 바로 직전에 실행된 명령의 종료 코드를 `$?`에 저장한다.
- 다음 명령을 실행하면 `$?` 값은 그 명령 기준으로 덮어쓴다.

**`$?` (종료 상태 조회)**
- 명령을 실행한 직후 `echo $?` 를 입력하면 직전에 실행한 명령의 exit status가 출력된다.
- 성공하면 0, 실패하면 1~255 중 하나

**Example1 (예제 1)**

```bash
# cp file1 file2
# echo $?
```

file1 이 존재하고 정상적으로 복사되면 0 출력.
file1이 없으면 cp 실패 : `$?` 는 1 또는 2 등 비정상 코드 반환

**Example2 (여기서 사용자가 Ctrl + C 누르면 강제 종료)**

```bash
echo $?
```

- Ctrl+C 는 SIGINT(신호 번호 2)를 보낸다.
- exit code = 128 + 2 = 130
- 따라서 `$?` 는 130 출력된다.

**실습 예제 (명령 성공/실패 확인)**

```bash
touch test.txt
echo $?         # 성공 = 0


rm 없는파일.txt
echo $?         # 실패 = 1


asdfasdf        # 존재하지 않는 명령 실행
echo $?         # 127 (command not found)


sleep 50
Ctrl + C        # 강제 종료 실습
echo $?         # 130 (SIGINT)
```

- **왜 exit 코드가 중요할까?**
  - 자동화 스크립트에서 가장 중요한 것은 이전 명령이 성공했는지 실패했는지 판단하는 것
  - exit 코드를 이해해야 에러 처리, 조건문, 자동화 판단 로직이 모두 가능하다.

**EX1) 존재하지 않는 명령 abc123 을 실행한 후 종료 코드를 확인하시오.**

```bash
[root@Server-A ~]# abc123
bash: abc123: 명령을 찾을 수 없습니다...

[root@Server-A ~]# echo $?
127
```

**EX2) 파일 fileA 가 존재한다고 가정할 때, cp fileA fileB 실행 후 성공했는지 $? 로 확인하는 명령을 작성**

```bash
[root@Server-A ~]# touch fileA && echo HELLO > fileA  && cp fileA fileB


[root@Server-A ~]# echo $?
0


[root@Server-A ~]# ls  -l  file*
-rw-r--r-- 1 root root 6  7월 24 10:11 fileA
-rw-r--r-- 1 root root 6  7월 24 10:11 fileB


[root@Server-A ~]# echo $?
0

[root@Server-A ~]# cat fileA
HELLO


[root@Server-A ~]# echo $?
0
```

### test 명령어

**test**는 파일, 디렉터리, 문자열, 정수 등의 상태를 검사하여 참 또는 거짓을 종료 코드로 반환하는 Bash 명령어이다.

- 조건이 참이면 종료 코드 0, 거짓이면 종료 코드 1을 반환한다.
- **test**는 결과를 화면에 직접 출력하지 않으므로 `$?`, `&&`, `||`, `if` 문과 함께 사용한다.

```bash
EX) test  $x  -gt  $y

EX) [ $x  -gt  $y ]
```

- `[ 조건 ]` 형태에서 `[ A 조건 B ]` 양쪽에 반드시 공백 필수
- 조건을 검사하는 위치는 `if`, `while`, `until` 등의 조건문과 같이 사용된다.

**test 기본 형식**
- `test 조건식`
- `[ 조건식 ]`
- `[ 조건 ]` 형태에서 `[ A 조건 B ]` 양쪽에 반드시 공백 필수
- EX) `[ $x -eq $y ]`
- EX) `[ "$str" = "$str2" ]`

**숫자 비교 옵션**

| 표현 | 의미 |
|---|---|
| `[ $x -eq $y ]` | x와 y가 같으면 참 |
| `[ $x -ne $y ]` | x와 y가 다르면 참 |
| `[ $x -gt $y ]` | x가 y보다 크면 참 |
| `[ $x -lt $y ]` | x가 y보다 작으면 참 |
| `[ $x -ge $y ]` | x가 y보다 크거나 같으면 참 |
| `[ $x -le $y ]` | x가 y보다 작거나 같으면 참 |

**문자열 비교 옵션**

| 표현 | 의미 |
|---|---|
| `[ "$x" = "$y" ]` | 문자열 x와 y가 같으면 참 |
| `[ "$x" != "$y" ]` | 문자열 x와 y가 다르면 참 |
| `[ -z "$x" ]` | x가 빈 문자열이면 참 |
| `[ -n "$x" ]` | x가 비어 있지 않으면 참 |

- 문자열 비교 시 double-quote로 묶지 않으면 공백 문제 발생 가능

**파일 테스트 옵션**

1) 파일 존재 여부
```bash
[ -e $x ] 파일 또는 디렉터리가 존재
```

2) 파일 종류 확인
```bash
[ -f $x ] 일반 파일이면 참
[ -d $x ] 디렉터리면 참
[ -L $x ] 심볼릭 링크면 참
[ -b $x ] 블록 디바이스
[ -c $x ] 문자 디바이스
```

3) 파일 권한
```bash
[ -r $x ] 읽기 가능
[ -w $x ] 쓰기 가능
[ -x $x ] 실행 가능
```

4) 파일 속성
```bash
[ -s $x ] 파일 크기가 0보다 크면 참
[ $x -nt $y ] x가 y보다 최신 파일이면 참
[ $x -ot $y ] x가 y보다 오래된 파일이면 참
[ $x -ef $y ] inode가 같으면 참(하드링크 관계 등)
```

예
```bash
x="/etc/passwd"
[ -f $x ] 	: 참
```

**논리 연산 옵션**

test( 구식 논리연산자 )
```bash
조건1 -a 조건2 	두 조건 모두 참
조건1 -o 조건2 	두 조건 중 하나만 참
```

bash 논리식
```bash
[ 조건1 ] && [ 조건2 ] 둘 다 참일 때 참
[ 조건1 ] || [ 조건2 ] 둘 중 하나만 참이면 참
```

**숫자 비교 옵션**

| 표현 | 의미 |
|---|---|
| `[ $x -eq $y ]` | x와 y가 같으면 참 |
| `[ $x -ne $y ]` | x와 y가 다르면 참 |
| `[ $x -gt $y ]` | x가 y보다 크면 참 |
| `[ $x -lt $y ]` | x가 y보다 작으면 참 |
| `[ $x -ge $y ]` | x가 y보다 크거나 같으면 참 |
| `[ $x -le $y ]` | x가 y보다 작거나 같으면 참 |

**EX1) 변수 x=10, y=20 일 때 x가 y보다 작은지 검사하시오**

```bash
[root@Server-A ~]# a=10
[root@Server-A ~]# b=20

[root@Server-A ~]# [ $a -lt $b ]

[root@Server-A ~]# echo $?
0				# x값이 더 작기때문에 True



[root@Server-A ~]# [ $a -gt $b ]

[root@Server-A ~]# echo $?
1				# x값이 더 작기때문에 Flase
```

**문자열 비교 옵션**

| 표현 | 의미 |
|---|---|
| `[ "$x" = "$y" ]` | 문자열 x와 y가 같으면 참 |
| `[ "$x" != "$y" ]` | 문자열 x와 y가 다르면 참 |
| `[ -z "$x" ]` | x가 빈 문자열이면 참 |
| `[ -n "$x" ]` | x가 비어 있지 않으면 참 |

- 문자열을 비교시 `""`로 감싸야하며 비교는 `=` 를 사용해야 한다.

**EX2) 변수 x="park", y="park" 일 때 문자열이 같은지 검사하시오**

```bash
[root@Server-A ~]# x="park"
[root@Server-A ~]# y="park"

[root@Server-A ~]# [ "$x" = "$y" ]

[root@Server-A ~]# echo $?
0				# x와 y안의 문자가 같기때문에 True



[root@Server-A ~]# [ "$x" != "$y" ]

[root@Server-A ~]# echo $?
1
```

**문자열 비교 옵션**

| 표현 | 의미 |
|---|---|
| `[ "$x" = "$y" ]` | 문자열 x와 y가 같으면 참 |
| `[ "$x" != "$y" ]` | 문자열 x와 y가 다르면 참 |
| `[ -z "$x" ]` | x가 빈 문자열이면 참 |
| `[ -n "$x" ]` | x가 비어 있지 않으면 참 |

**EX3) 변수 x="" 일 때 x가 빈 문자열인지 검사하시오**

```bash
[root@Server-A ~]# x=""

[root@Server-A ~]# [ -z "$x" ]

[root@Server-A ~]# echo $?
0
```

- x변수는 빈 문자열이므로 참 (0)

**파일 테스트 옵션**

1) 파일 존재 여부
```bash
[ -e $x ] 파일 또는 디렉터리가 존재
```

2) 파일 종류 확인
```bash
[ -f $x ] 일반 파일이면 참
[ -d $x ] 디렉터리면 참
[ -L $x ] 심볼릭 링크면 참
[ -b $x ] 블록 디바이스
[ -c $x ] 문자 디바이스
```

**EX4) 파일 /etc/passwd 가 일반 파일인지 검사하시오**

```bash
[root@Server-A ~]# [ -f /etc/passwd ]

[root@Server-A ~]# echo $?
0				# /etc/passwd는 파일이므로 True



[root@Server-A ~]# [ -d /etc/passwd ]

[root@Server-A ~]# echo $?
1				# /etc/passwd는 파일이므로 Flase
```

**파일 테스트 옵션**

1) 파일 존재 여부
```bash
[ -e $x ] 파일 또는 디렉터리가 존재
```

2) 파일 종류 확인
```bash
[ -f $x ] 일반 파일이면 참
[ -d $x ] 디렉터리면 참
[ -L $x ] 심볼릭 링크면 참
[ -b $x ] 블록 디바이스
[ -c $x ] 문자 디바이스
```

**EX5) /var/log 파일 또는 디렉터리가 존재하는지 검사하시오**

```bash
[root@Server-A ~]# [ -e /var/log ]

[root@Server-A ~]# echo $?
0				# /var/log가 있으므로 True


[root@Server-A ~]# [ -d /var/log ]

[root@Server-A ~]# echo $?
0				# /var/log가 디렉터리이므로 True
```

**권한**
```bash
[ -r $x ] 읽기 가능
[ -w $x ] 쓰기 가능
[ -x $x ] 실행 가능
```

**EX6) 변수 x="/etc/hosts" 일 때 읽기 가능한 파일인지 검사하시오**

```bash
[root@Server-A ~]# [ -r /etc/hosts ]


[root@Server-A ~]# echo $?
0

[root@Server-A ~]# ls  -l  /etc/hosts
-rw-r--r--. 1 root root 158  6월 23  2020 /etc/hosts
```

**권한**
```bash
[ -r $x ] 읽기 가능
[ -w $x ] 쓰기 가능
[ -x $x ] 실행 가능
```

**EX7-1) 변수 x="/bin/bash" 가 실행 가능한 파일인지 검사하시오**

```bash
[root@Server-A ~]# [ -x /bin/bash ]


[root@Server-A ~]# echo $?
0

[root@Server-A ~]# ls  -l  /bin/bash
-rwxr-xr-x. 1 root root 1389024  4월 30  2024 /bin/bash	# x가 있으므로 실행파일
```

**EX7-2) 변수 x="/etc/hosts" 가 실행 가능한 파일인지 검사하시오**

```bash
[root@Server-A ~]# [ -x /etc/hosts ]


[root@Server-A ~]# echo $?
1



[root@Server-A ~]# ls -l /etc/hosts
-rw-r--r--. 1 root root 158  6월 23  2020 /etc/hosts	# x가 없으므로 실행파일이 아님
```

**숫자 비교 옵션**

| 표현 | 의미 |
|---|---|
| `[ $x -eq $y ]` | x와 y가 같으면 참 |
| `[ $x -ne $y ]` | x와 y가 다르면 참 |
| `[ $x -gt $y ]` | x가 y보다 크면 참 |
| `[ $x -lt $y ]` | x가 y보다 작으면 참 |
| `[ $x -ge $y ]` | x가 y보다 크거나 같으면 참 |
| `[ $x -le $y ]` | x가 y보다 작거나 같으면 참 |

**EX8) 변수 x=15, y=10 일 때 x가 y보다 크고 동시에 x가 20보다 작은가?**

```bash
[root@Server-A ~]# x=15
[root@Server-A ~]# y=10


[root@Server-A ~]# [ $x -gt $y ] && [ $x -lt 20 ]

[root@Server-A ~]# echo $?
0

# [ $x -gt $y ]	= True
# [ $x -lt 20 ]	= True
```

**숫자 비교 옵션**

| 표현 | 의미 |
|---|---|
| `[ $x -eq $y ]` | x와 y가 같으면 참 |
| `[ $x -ne $y ]` | x와 y가 다르면 참 |
| `[ $x -gt $y ]` | x가 y보다 크면 참 |
| `[ $x -lt $y ]` | x가 y보다 작으면 참 |
| `[ $x -ge $y ]` | x가 y보다 크거나 같으면 참 |
| `[ $x -le $y ]` | x가 y보다 작거나 같으면 참 |

**EX9) 변수 x=5 일 때 x가 1보다 작거나 10보다 큰가?**

```bash
 x=5

[root@Server-A ~]# [ $x -lt 1 ] || [ $x -gt 20 ]

[root@Server-A ~]# echo $?
1


# [ $x -lt 1 ] 	= Flase
# [ $x -gt 20 ] 	= Flase
```

**파일 테스트 옵션**

1) 파일 존재 여부
```bash
[ -e $x ] 파일 또는 디렉터리가 존재
```

2) 파일 종류 확인
```bash
[ -f $x ] 일반 파일이면 참
[ -d $x ] 디렉터리면 참
[ -L $x ] 심볼릭 링크면 참
[ -b $x ] 블록 디바이스
[ -c $x ] 문자 디바이스
```

**EX10-1) 파일 /temp/test.log 가 존재하지 않음을 검사한 후 있다면 파일인지 디렉터리인지 확인하시오**

```bash
[root@Server-A ~]# [ -e /temp/test.log ]


[root@Server-A ~]# echo $?
1


[root@Server-A ~]# ls -l /temp/test*
-rw-r--r-- 1 root root 0  7월 23 16:12 /temp/test.txt
```

**EX10-2) 파일 /etc/ssh/ssh_config.d 가 존재하지 않음을 검사한 후 있다면 파일인지 디렉터리인지 확인하시오**

```bash
[root@Server-A ~]# [ -e /etc/ssh/ssh_config.d ]


[root@Server-A ~]# echo $?			# /etc/ssh/ssh_config.d가 있음
0



[root@Server-A ~]# [ -d /etc/ssh/ssh_config.d ]


[root@Server-A ~]# echo $?			# 디렉터리 O
0




[root@Server-A ~]# [ -f /etc/ssh/ssh_config.d ]


[root@Server-A ~]# echo $?			# 파일 X
1
```

4) 파일 속성
```bash
[ -s $x ] 파일 크기가 0보다 크면 참
[ $x -nt $y ] x가 y보다 최신 파일이면 참
[ $x -ot $y ] x가 y보다 오래된 파일이면 참
[ $x -ef $y ] inode가 같으면 참(하드링크 관계 등)
```

**EX11) 파일 x="/etc/passwd" 와 y="/etc/aliases" 중 passwd 파일이 더 최신 파일인지 검사하시오**

```bash
[root@Server-A ~]# x="/etc/passwd"
[root@Server-A ~]# y="/etc/aliases"


[root@Server-A ~]# [ "$x" -nt "$y" ]


[root@Server-A ~]# echo $?
0				# /etc/passwd가 더 최근에 만들어지거나 수정된 파일이므로 True


[root@Server-A ~]# ls  -l  /etc/passwd
-rw-r--r-- 1 root root 2124  7월 21 16:38 /etc/passwd


[root@Server-A ~]# ls  -l  /etc/aliases
-rw-r--r--. 1 root root 1529  6월 23  2020 /etc/aliases
```

4) 파일 속성
```bash
[ -s $x ] 파일 크기가 0보다 크면 참
[ $x -nt $y ] x가 y보다 최신 파일이면 참
[ $x -ot $y ] x가 y보다 오래된 파일이면 참
[ $x -ef $y ] inode가 같으면 참(하드링크 관계 등)
```

**EX11) 파일 x="/etc/passwd" 의 크기가 0보다 큰지 검사하시오**

```bash
[root@Server-A ~]# x="/etc/passwd"


[root@Server-A ~]# [ -s "$x" ]


[root@Server-A ~]# echo $?
0				# 파일의 크기가 0보다 크므로 True


[root@Server-A ~]# ls  -l  /etc/passwd
-rw-r--r-- 1 root root 2124  7월 21 16:38 /etc/passwd


[root@Server-A ~]# [ ! -s "$x" ]

[root@Server-A ~]# echo $?
1
```

**문자열 비교 옵션**

| 표현 | 의미 |
|---|---|
| `[ "$x" = "$y" ]` | 문자열 x와 y가 같으면 참 |
| `[ "$x" != "$y" ]` | 문자열 x와 y가 다르면 참 |
| `[ -z "$x" ]` | x가 빈 문자열이면 참 |
| `[ -n "$x" ]` | x가 비어 있지 않으면 참 |

**EX13) 변수 x="abc", y="" 일 때 y가 비어 있고 x는 비어 있지 않음을 검사하시오**

```bash
[root@Server-A ~]# [ -n "$x" ] && [ -z "$y" ]

[root@Server-A ~]# echo $?
0
```

**문자열 비교 옵션**

| 표현 | 의미 |
|---|---|
| `[ "$x" = "$y" ]` | 문자열 x와 y가 같으면 참 |
| `[ "$x" != "$y" ]` | 문자열 x와 y가 다르면 참 |
| `[ -z "$x" ]` | x가 빈 문자열이면 참 |
| `[ -n "$x" ]` | x가 비어 있지 않으면 참 |

**EX14) 변수 x="soldesk", y="linux" 일 때 두 문자열이 다름을 검사하시오**

```bash
[root@Server-A ~]# x="soldesk"
[root@Server-A ~]# y="linux"


[root@Server-A ~]# [ "$x" = "y" ]
[root@Server-A ~]# echo $?
1


[root@Server-A ~]# [ "$x" != "y" ]

[root@Server-A ~]# echo $?
0
```

**정리**: exit 코드는 명령의 성공/실패를 숫자로 나타내며 `$?`로 확인할 수 있고, **test**(`[ ]`)는 이 원리를 이용해 숫자·문자열·파일 상태를 검사해 종료 코드(참/거짓)로 돌려주는 도구다.

---

## 조건문 (if/elif/else, case)

조건문은 특정 조건의 참과 거짓을 판단하여 실행할 명령어를 선택하는 문법이다.

- 쉘스크립트에서는 조건식의 결과를 TRUE, FALSE라는 값으로 직접 판단하지 않고, 명령어의 종료 코드를 기준으로 판단한다.
  - 종료 코드 0 : 참(true), 성공
  - 종료 코드 0 이외 : 거짓(false), 실패
- 예
  - 파일이 있으면 복사한다.
  - 숫자가 크면 메시지를 출력한다.
  - 네트워크가 연결되면 서비스를 재시작한다.
  - 이렇게 조건에 따라 흐름을 바꾸는 것을 조건문이라고 한다.

### if–then–fi의 기본 개념

Bash 조건문의 가장 기본적인 형태는 **if–then–fi** 구조이다.

- 조건식의 결과가 참이면 then 블록의 명령어를 실행하고, 거짓이면 실행하지 않고 fi 이후로 넘어간다.
- 쉘스크립트에서는 조건식의 결과를 종료 코드로 판단한다.
  - 종료 코드 0 : 참(True), 성공
  - 종료 코드 0 이외 : 거짓(False), 실패

형식
```bash
if [ 조건식 ]; then
    실행할 명령1
    실행할 명령2
    실행할 명령3
    .....
    실행할 명령10
fi
```

- 조건식이 참(true)이면 then 블록이 실행되고,
- 조건식이 거짓(false)이면 아무것도 실행하지 않고 fi 이후로 넘어간다.

**조건문의 기본 형식**

형식 1) 단순 if
```bash
if [ 조건 ]; 
then
  		명령
fi
```

형식 2) if–else
```bash
if [ 조건 ]; then
  		명령1
else
  		명령2
fi
```

형식 3) if–elif–else
```bash
if [ 조건1 ]; 
then
  		명령1
elif [ 조건2 ]; 
then
  		명령2
else
  		명령3
fi
```

- 여기서 `fi` 는 if 의 끝을 의미한다. (Bash는 블록을 `fi`, `done` 같은 키워드로 닫는다.)
- 조건식 작성 방법
  - 조건식은 `[ 표현식 ]` 형태로 작성하며
  - 실제로는 **test** 명령어와 같은 의미를 가진다.

예
```bash
[ 5 -gt 3 ]
  : 5가 3보다 크면 참
```

조건식의 예시는 다음과 같다.

- 숫자 비교
```bash
[ a -gt b ] a > b
[ a -lt b ] a < b
[ a -eq b ] a == b
[ a -ne b ] a != b
```

- 문자열 비교
```bash
[ "$x" = "$y" ]
[ "$x" != "$y" ]
[ -z "$x" ] 	문자열 길이가 0인가?
[ -n "$x" ] 	문자열 길이가 0이 아닌가?
```

- 파일 조건 비교
```bash
[ -e 파일 ] 파일이 존재하는가?
[ -f 파일 ] 일반 파일인가?
[ -d 디렉터리 ] 디렉터리인가?
[ -r 파일 ] 읽기 가능한가?
[ -w 파일 ] 쓰기 가능한가?
[ -x 파일 ] 실행 가능한가?
```

- 논리 연산
```bash
[ 조건1 ] && [ 조건2 ] (둘 다 참)
[ 조건1 ] || [ 조건2 ] (둘 중 하나 참)
```

**EX1) 변수 x=10 일 때, x 가 5보다 크면 "big" 을 출력하시오**

```bash
[root@Server-A ~]# x=10

[root@Server-A ~]# if [ $x -gt 5 ]
> then
> echo big
> fi
big
```

**EX2) 변수 name="root" 일 때, name 이 root 이면 "admin user" 출력하시오**

```bash
[root@Server-A ~]# name="root"


[root@Server-A ~]# if [ "$name" = "root" ]
> then
> echo "admin user"
> fi
admin user

[root@Server-A ~]# if [ "$name" = "root" ]; then echo "admin user"; fi
```

**EX3) /home 디렉터리가 읽기 가능(-r)이면 "read ok" 출력하시오.**

```bash
[root@Server-A ~]# if [ -r /home ]
> then
> echo "read OK"
> fi
read OK
```

**EX4) 변수 x=3 일 때, x 가 0보다 크면 "positive" 를 출력하시오. (숫자 비교: -gt 사용)**

```bash
[root@Server-A ~]# if [ $x -gt 0 ]
> then
> echo "positive"
> fi
positive



[root@Server-A ~]# if [ $x -gt 0 ]; then echo "positive"; fi
positive




[root@Server-A ~]# if [ $x -gt 0 ]
> then
> echo "positive1"
> echo "positive2"
> echo "positive3"
> fi


[root@Server-A ~]# if [ $x -gt 0 ]; then echo "positive1";  echo "positive2";  echo "positive3";  fi
positive1
positive2
positive3
```

**EX5) 변수 num의 값이 정수 5 이상일 경우에만 "OK" 를 출력하시오 (숫자 비교 : -ge 사용)**

```bash
[root@Server-A ~]# if [ $num -ge 5 ];  then echo "OK";  fi
OK
```

**EX6) 변수를 생성 후 변수 a의 값이 변수 b의 값에 /2한 값과 같으면 "half" 를 출력하시오 (산술 비교는 (( )) 사용)**

```bash
[root@Server-A ~]# a=7
[root@Server-A ~]# b=14

[root@Server-A ~]# if (( a == b /2)); then echo "half";  fi
half
```

**EX7) 파일 /etc/hosts 가 존재하면 "hosts exists" 를 출력하시오.**
(파일 존재 여부: -e)

```bash
[root@Server-A ~]# if [ -e "/etc/hosts" ]; then echo "hosts exists"; fi
hosts exists
```

**EX8) 변수 x="/etc/passwd", y="/etc/aliases"일 때, x 파일이 y 파일보다 최신이면 "x is newer"를 출력하시오.**

```bash
[root@Server-A ~]# x="/etc/passwd"
[root@Server-A ~]# y="/etc/aliases"

[root@Server-A ~]# if [ "$x" -nt "$y" ];  then  echo "x is newer";  fi
x is newer
```

**EX9) /backup/shellscript 파일을 생성하고 변수 file="/backup/shellscript"를 생성한 후, 변수 file이 가리키는 파일의 크기가 0Byte이면 "New Data Insert" 문자열을 파일안에 추가하고, cat 명령어로 전체 내용을 출력하시오**

```bash
[root@Server-A ~]# mkdir  /backup		# 없으면 생성

[root@Server-A ~]# touch  /backup/shellscript

[root@Server-A ~]# file="/backup/shellscript"

[root@Server-A ~]# if [ ! -s  "$file" ]
> then
> echo "New Data Insert" > "$file"
> cat  "$file"
> fi
New Data Insert



[root@Server-A ~]# if [ ! -s  "$file" ];   then echo "New Data Insert" > "$file";  cat  "$file";  fi
New Data Insert
```

### if–then–else 기본 형식

```bash
if [ 조건식 ] 또는  command
then
    		실행문1         	# 조건이 참(true)일 때 실행
else
    		실행문2         	# 조건이 거짓(false)일 때 실행
fi                  	# if 문 종료
```

**EX1) '/var/log/messages 파일이 존재하면 "found", 아니면 "not found" 출력되어야 한다.**

```bash
[root@Server-A ~]# if  [ -e "/var/log/messages" ]
> then
>     echo "found"
> else
>     echo "not found"
> fi
found
```

**EX2) 변수 num에 정수를 입력 후 num의 정수값이 짝수면 "even", 홀수면 "odd" 출력하시오**

```bash
[root@Server-A ~]# num=90348

[root@Server-A ~]# if (( $num % 2 == 0 ))
> then
>   echo "even"
> else
>   echo "odd"
> fi
even



[root@Server-A ~]# num=90348

[root@Server-A ~]# if (( $num % 2 != 1 ))
> then

>   echo "even"
> else
>   echo "odd"
> fi
even



[root@Server-A ~]# num=90231

[root@Server-A ~]# if [ $((num % 2)) -eq 0 ]
> then
>   echo "even"
> else
>   echo "odd"
> fi
odd
```

**EX3) '/usr/bin/top' 파일이 실행 가능 파일이면 "run ok", 아니면 "no exec"가 출력되어야 한다.**

```bash
[root@Server-A ~]# if [ -x /usr/bin/top ]
> then  
>    echo "run OK"
> else 
>    echo "no exec"
> fi
run OK



[root@Server-A ~]# if [ -x /usr/bin/top ]; then  echo "run OK";  else  echo "no exec"; fi
run OK
```

**EX4-1) "ls /root" 명령이 성공하면 "ls OK", 실패하면 "ls Fail"이 출력되어야 한다.**

```bash
[root@Server-A ~]# if  ls  /root
> then
> echo "ls OK"
> else
> echo "ls Fail"
> fi
Solbangul        	data3.txt    	file15.txt	meta_lab           	바탕화면
Soldesk          	example.txt	file3.txt 	meta_lab_setup.sh	비디오
Solnamu          	file1.txt    	file7.txt 	semi.txt           	사진
anaconda-ks.cfg  	file11.txt   	fileA    	test1.txt          	서식
backup           	file12.txt   	fileB     	공개               	음악
data1.txt        	file13.txt   	hosts     	다운로드
data2.txt        	file14.txt   	messages	문서
ls OK
```

**EX4-2) 아래의 조건에 맞게 조건문을 작성하시오**
- "ls /root" 명령이 성공하면 "OK", 실패하면 "FAIL"이 출력되어야 한다.
- "ls /root"의 결과는 모니터에 출력되지 않아야 한다.

```bash
[root@Server-A ~]# if  ls  /root  >  /dev/null   2>&1
> then
>    echo "ls OK"
> else
>    echo "ls Fail"
> fi
ls OK


  > /dev/null      	: 정상 출력 숨김
  2> /dev/null     	: 오류 출력 숨김
  > /dev/null 2>&1	: 정상 출력과 오류 출력 모두 숨김
```

**EX5) 현재 접속한 사용자가 root이면 /root 를 ls -l, 아니면 pwd 를 출력하시오**

```bash
[root@Server-A ~]# user=$(whoami)


[root@Server-A ~]# if [ "$user" = "root" ]
> then
>    ls  -l
> else
>   pwd
> fi
합계 2072
-rw-r--r--  1 root root       0  7월 22 15:11 Solbangul
-rw-r--r--  1 root root       0  7월 22 15:11 Soldesk
-rw-r--r--  1 root root       0  7월 22 15:11 Solnamu
-rw-------. 1 root root    1027  7월  2 12:55 anaconda-ks.cfg
-rw-r--r--  1 root root       0  7월 23 15:55 backup
-rw-r--r--  1 root root       0  7월 23 17:20 data1.txt
-rw-r--r--  1 root root       0  7월 23 17:20 data2.txt
-rw-r--r--  1 root root       0  7월 23 17:20 data3.txt
-rw-r--r--  1 root root       0  7월 23 17:08 example.txt
-rw-r--r--  1 root root       0  7월 22 15:55 file1.txt
-rw-r--r--  1 root root       0  7월 22 15:56 file11.txt
-rw-r--r--  1 root root       0  7월 22 15:56 file12.txt
-rw-r--r--  1 root root       0  7월 22 15:56 file13.txt
-rw-r--r--  1 root root       0  7월 22 15:56 file14.txt
-rw-r--r--  1 root root       0  7월 22 15:56 file15.txt
-rw-r--r--  1 root root       0  7월 22 15:55 file3.txt
-rw-r--r--  1 root root       0  7월 22 15:55 file7.txt
-rw-r--r--  1 root root       6  7월 24 10:11 fileA
-rw-r--r--  1 root root       6  7월 24 10:11 fileB
--wxr--r--  1 root root     158  7월 24 10:51 hosts
-rw-------  1 root root 2099476  7월 23 17:17 messages
drwxr-xr-x  6 root root      74  7월 22 12:46 meta_lab
-rwxr-xr-x  1 root root    3773  7월 22 12:41 meta_lab_setup.sh
-rw-r--r--  1 root root       0  7월 23 15:13 semi.txt
-rw-r--r--  1 root root       0  7월 24 10:05 test1.txt
drwxr-xr-x. 2 root root       6  7월  2 14:46 공개
drwxr-xr-x. 2 root root       6  7월  2 14:46 다운로드
drwxr-xr-x. 2 root root       6  7월  2 14:46 문서
drwxr-xr-x. 2 root root       6  7월  2 14:46 바탕화면
drwxr-xr-x. 2 root root       6  7월  2 14:46 비디오
drwxr-xr-x. 2 root root       6  7월  2 14:46 사진
drwxr-xr-x. 2 root root       6  7월  2 14:46 서식
drwxr-xr-x. 2 root root       6  7월  2 14:46 음악
```

**EX6) pwd를 사용하여 현재 위치가 "/root"면 상위 디렉터리로 이동하는 명령어 실행되어야 하지만 현재 위치가 "/root"가 아니면 자신의 홈 디렉터리로 이동**

```bash
	# /root에서 테스트
[root@Server-A ~]# if [ "$(pwd)" = "/root" ]
> then
>    echo "====================================="
>    echo "현재 위치는 $(pwd) 입니다."
>    cd ..
> else
>    echo "====================================="
>    echo "현재 위치는 $(pwd) 입니다."
>    echo "홈디렉터리인 $(pwd)로 이동합니다."
>    cd "$HOME"
> fi
[root@Server-A /]# pwd
/


	# /etc/ssh에서 테스트
[root@Server-A etc]# cd /etc/ssh

[root@Server-A ssh]# if [ "$(pwd)" = "/root" ]
> then
>    echo "====================================="
>    echo "현재 위치는 $(pwd) 입니다."
>    cd ..
> else
>    echo "====================================="
>    echo "현재 위치는 $(pwd) 입니다."
>    echo "홈디렉터리인 ${HOME}로 이동합니다."
> cd "$HOME"
> fi
=====================================
현재 위치는 /etc/ssh 입니다.
홈디렉터리인 /root로 이동합니다.
[root@Server-A ~]# pwd
/root
```

**EX7) 변수 "/backup/soldesk" 해당 경로에 파일 또는 디렉터리가 존재하는지 확인하시오**
- 존재하면 일반 파일인지 디렉터리인지 구분하여 출력해야 한다.
- 일반 파일이면 "/backup/soldesk는 파일입니다."를 출력해야 한다.
- 디렉터리이면 "/backup/soldesk는 디렉터리입니다."를 출력해야 한다.
- "/backup/soldesk"가 존재하지 않으면 "/backup/soldesk 파일 또는 디렉터리가 없습니다."를 출력해야 한다.

```bash
	# 1) /backup/soldesk 가 없는 상태에서 테스트

[root@Server-A ~]# path="/backup/soldesk"

[root@Server-A ~]# if [ -e "$path" ]
 then
     if [ -f "$path" ]
     then
         echo "$path는 파일입니다."
     else
         echo "$path는 디렉터리입니다."
     fi
 else
     echo "$path 파일 또는 디렉터리가 없습니다."
 fi

/backup/soldesk 파일 또는 디렉터리가 없습니다.



	# 2) /backup/soldesk 디렉터리인 환경에서 테스트

[root@Server-A ~]# mkdir  -p  /backup/soldesk

[root@Server-A ~]# path="/backup/soldesk"

[root@Server-A ~]# if [ -e "$path" ]
 then
     if [ -f "$path" ]
     then
         echo "$path는 파일입니다."
     else
         echo "$path는 디렉터리입니다."
     fi
 else
     echo "$path 파일 또는 디렉터리가 없습니다."
 fi

/backup/soldesk는 디렉터리입니다.



	# 3) /backup/soldesk 파일인 환경에서 테스트

[root@Server-A ~]# rm  -rf  /backup/soldesk

[root@Server-A ~]# touch  /backup/soldesk

[root@Server-A ~]# path="/backup/soldesk"

[root@Server-A ~]# if [ -e "$path" ]
 then
     if [ -f "$path" ]
     then
         echo "$path는 파일입니다."
     else
         echo "$path는 디렉터리입니다."
     fi
 else
     echo "$path 파일 또는 디렉터리가 없습니다."
 fi

/backup/soldesk는 파일입니다.
```

### if–elif–else 기본 형식

```bash
if [ 조건식1 ] 또는  command1
then
    실행문1         	# 조건이 참(true)일 때 실행
elif [ 조건식2 ] 또는  command2
then
    실행문2         	# 조건이 참(true)일 때 실행
elif [ 조건식3 ] 또는  command3
then
    실행문3         	# 조건이 참(true)일 때 실행
else
    실행문4         	# 조건이 매치되지 않을때 실행
fi                  	# if 문 종료
```

**정리**: **if–then–fi**, **if–then–else**, **if–elif–else**는 모두 명령/조건식의 종료 코드(0 = 참)를 기준으로 분기하며, 조건식은 `[ ]`(test) 또는 산술 비교 `(( ))`, 명령 자체(예: `if ls /root`)로 작성할 수 있다.

## 조건문을 스크립트 파일로 작성하기

쉘 스크립트(Shell Script)는 여러 개의 Linux 명령어를 하나의 파일에 작성하여 순서대로 실행하는 프로그램이다.

- 반복적으로 수행하는 작업을 자동화할 때 사용한다.
- 예를 들어 다음 작업을 하나의 스크립트로 만들 수 있다.
  - 사용자 계정 생성
  - 디렉터리와 파일 생성
  - 파일 백업
  - 서비스 상태 확인
  - 디스크 사용량 점검
  - 네트워크 연결 확인
  - 로그 파일 분석

**1) 반복 실행이 가능하다**

터미널에서 직접 if 문을 입력하면 매번 조건문을 타이핑해야 한다.

```bash
EX) if [ -e /etc/hosts ]; then echo OK; fi
```

이걸 하루에 수십번에서 수백번을 사용하려면 불필요한 시간이 많아진다. 파일로 만들면 실행만 하면 된다.

```bash
EX)   ./check_hosts.sh
```

**2) 반복 실행이 필요한 작업일수록 스크립트가 필수다.**
- 자동화가 가능하다 (cron과 연동)
- 실무에서는 서비스 상태 확인, 로그 정리, 백업 자동화, 디렉터리 감시 같은 작업을 주기적으로 자동 실행해야 한다.
- 이런 자동화는 모두 스크립트 파일을 기반으로 한다.
- 터미널에서 입력하는 즉시 if 문이 동작하기때문에 자동화가 불가능하다. (그래서 파일로 만들어서 관리해야 한다.)

**3) 유지보수와 수정이 쉽다**
- 스크립트 파일은 텍스트 파일이므로 수정, 개선, 주석 달기, 기능 추가가 모두 편리하다.
- 즉석으로 작성한 조건문은 기록도 없고, 재사용도 어렵다.

**4) 다른 사람과 공유할 수 있다**
- 실무 팀에서는 같은 스크립트를 여러 사람이 사용한다.
  - 서버 점검 스크립트
  - 배포 스크립트
  - 로그 모니터링 스크립트
- 파일로 존재해야 git, 이메일, 메신저 등을 통해 쉽게 공유할 수 있다.

**5) 실수 방지 (재현성 확보)**
- 복잡한 조건문을 터미널에 직접 입력하면 오타가 나기 쉽고 명령이 길어질수록 실수 확률이 올라간다.
- 스크립트 파일로 만들어두면 항상 동일하게, 안정적으로 실행할 수 있다.

**EX1) 특정 파일 존재 여부 확인 후 메시지 출력**
- /etc/hosts 파일이 존재하면 "hosts OK" , 아니면 "hosts NOT FOUND" 출력해야 한다.

```bash
	# 수동 설정

[root@Server-A ~]# ls -l  /etc/hosts
-rw-r--r--. 1 root root 158  6월 23  2020 /etc/hosts



	# 조건문 테스트

[root@Server-A ~]# if [ -e /etc/hosts ]
> then
>   echo "hosts OK"
> else
>   echo "hosts NOT FOUND"
> fi
hosts OL



	# 스크립트 파일을 생성하여 관리

[root@Server-A ~]# mkdir  script

[root@Server-A ~]# vi ./script/check_hosts.sh
#!/bin/bash

if [ -e /etc/hosts ]
then
   echo "hosts OK"
else
   echo "hosts NOT FOUND"
fi

:wq



[root@Server-A ~]# cat  ./script/check_hosts.sh
#!/bin/bash

if [ -e /etc/hosts ]
then
   echo "hosts OK"
else
   echo "hosts NOT FOUND"
fi



[root@Server-A ~]#  ./script/check_hosts.sh			# 실행파일이 아니므로 해당 파일을 실행할 수 없다.
-bash: ./script/check_hosts.sh: 허가 거부



[root@Server-A ~]# ls  -l ./script/check_hosts.sh
-rw-r--r-- 1 root root 92  7월 24 16:13 ./script/check_hosts.sh	# rw- r-- r--  (x 권한이 없다.)



[root@Server-A ~]# chmod  +x  ./script/check_hosts.sh

[root@Server-A ~]# ls  -l ./script/check_hosts.sh
-rwxr-xr-x 1 root root 92  7월 24 16:13 ./script/check_hosts.sh	# rwx r-x r-x




[root@Server-A ~]#  ./script/check_hosts.sh
hosts OK




	# 스크립트 파일 수정
[root@Server-A ~]# cat  ./script/check_hosts.sh
#!/bin/bash

start="조건문 시작"
end="조건문 종료"

echo start

if [ -e /etc/hosts ]
then
   echo "hosts OK"
else
   echo "hosts NOT FOUND"
fi

echo end



[root@Server-A ~]#  ./script/check_hosts.sh
조건문 시작
hosts OK
조건문 종료
```

**EX2) /var/log/messages 로그 파일 크기가 50MB 이상이면 경고메세지가 출력되어야 한다.**

```bash
# du	: Disk Usage의 약자로 해당 파일 또는 디렉터리의 크기 및 용량을 표시한다.
# -h	: 사람이 읽기 편한 방식으로 출력 (K, M, G 단위로 표기)


[root@Server-A ~]# du  -h  /var/log/messages
2.2M    /var/log/messages


[root@Server-A ~]# du  -m  /var/log/messages
3       /var/log/messages



[root@Server-A ~]# du  -m  /var/log/messages | awk '{print $1}'
3

# $0 	: 현재 줄 전체
# $1 	: 첫 번째 필드
# $2 	: 두 번째 필드
# $NF	: 마지막 필드




	# 방법 1
[root@Server-A ~]# vi  ./script/disk_check.sh
#!/bin/bash

size=$(du  -m  /var/log/messages | awk '{print $1}')

if (( size >= 50 ))
then
   echo "LOG SIZE WARNING : ${size}MB"
else
   echo "LOG SIZE OK : ${size}MB"
fi

:wq


[root@Server-A ~]# chmod  +x ./script/disk_check.sh


[root@Server-A ~]# ./script/disk_check.sh
LOG SIZE OK : 3MB




	# 방법 2
[root@Server-A ~]# vi  ./script/disk_check.sh
#!/bin/bash

size=$(du -m /var/log/messages | awk '{print $1}')

if test "$size" -ge 50
then
    echo "LOG SIZE WARNING : ${size}MB"
else
    echo "LOG SIZE OK : ${size}MB"
fi

:wq


[root@Server-A ~]# chmod  +x ./script/disk_check.sh


[root@Server-A ~]# ./script/disk_check.sh
LOG SIZE OK : 3MB
```

**EX3) /tmp 안의 파일 개수가 100개 이상이면 경고 메세지가 출력되어야 한다.**

```bash
[root@Server-A ~]# ls -l  /tmp | wc -l
12


[root@Server-A ~]# vi ./script/tmp_count.sh
#!/bin/bash

count=$(ls -l  /tmp | wc -l)

if (( count >= 100))
then
  echo "경고 : 파일개수 ${count}개 (정리필요)"
else
  echo "TMP Count OK : 파일 개수 : ${count}개"
fi

:wq


[root@Server-A ~]# chmod +x ./script/tmp_count.sh


[root@Server-A ~]# ./script/tmp_count.sh
TMP Count OK : 파일 개수 : 12개
```

**EX4) SSHD의 상태가 active이면 'SSHD STAUS ACTIVE' 메세지를 출력하고 상태가 active가 아니면 'SSHD INACTIVE'메세지를 출력하고 vsFTP를 재시작 해야 한다.**

```bash
[root@Server-A ~]# systemctl is-active sshd
active					# 현재 ssh 서버가 동작하는 상태


[root@Server-A ~]# systemctl is-active sshd
inactive					# 현재 ssh 서버가 동작하지 않는 상태



[root@Server-A ~]# vi ./script/ssh_check.sh
#!/bin/bash

status=$(systemctl is-active sshd)

if  test  "$status" = "active"
then
  echo "SSHD STAUS ACTIVE" 
else
  echo "SSHD INACTIVE" 
fi

: wq


[root@Server-A ~]# chmod  +x  ./script/ssh_check.sh


[root@Server-A ~]# ./script/ssh_check.sh
SSHD STAUS AC


[root@Server-A ~]# ls  -l  script/
합계 16
-rwxr-xr-x 1 root root 142  7월 24 16:26 check_hosts.sh
-rwxr-xr-x 1 root root 173  7월 24 16:54 disk_check.sh
-rwxr-xr-x 1 root root 145  7월 24 17:41 ssh_check.sh
-rwxr-xr-x 1 root root 187  7월 24 17:11 tmp_count.sh
```

**EX5) "/soltest/test/passwd" 해당 경로에 파일이 있는지 확인해야 한다.**
- "/soltest/test/passwd"파일이 있으면 "/soltest/test/passwd 파일이 있습니다" 메세지가 출력되어야 한다.
- "/soltest/test/passwd"파일이 없으면 디렉터리와 파일을 생성하고 "/etc/passwd"파일 내용이 동일하게 작성되어야 하며 "/soltest/test/passwd 파일이 생성되었습니다." 메세지가 출려되어야 한다.

```bash
	# 실습1)  /soltest/test/passwd 파일이 있는경우

[root@Server-A ~]# vi  ./script/passwd_check.sh
#!/bin/bash

file="/soltest/test/passwd"

if test -f  "$file"
then  
    echo "/soltest/test/passwd 파일이 있습니다"
else
    mkdir  -p  /soltest/test/
    cat  /etc/passwd  >  "$file"

    if  test  -f  "$file"
    then
        echo  "$file 파일이 생성되었습니다."
    else
        echo  "$file 파일생성에 실패했습니다."
    fi
fi

:wq


[root@Server-A ~]# mkdir -p  /soltest/test
[root@Server-A ~]# cp  /etc/passwd  /soltest/test/


[root@Server-A ~]# chmod  +x  ./script/passwd_check.sh


[root@Server-A ~]# ./script/passwd_check.sh
/soltest/test/passwd 파일이 있습니다




	# 실습2)  /soltest/test/passwd 파일이 없는경우

[root@Server-A ~]# rm  -rf  /soltest


[root@Server-A ~]# ./script/passwd_check.sh
/soltest/test/passwd 파일이 생성되었습니다.
```

**EX6) 아래의 조건에 맞게 루트 파일 시스템의 디스크 사용량을 모니터링하시오.**
- df 명령어를 사용하여 "/" 파일 시스템의 디스크 사용량을 확인해야 한다.
- 디스크 사용률이 60% 이상이면 "DISK WARNING: 사용률% 사용 중" 메세지를 출력해야 한다.
- 디스크 사용률이 60% 미만이면 "DISK OK: 사용률%" 메시지를 출력해야 한다.

```bash
[root@Server-A ~]# df  -h
Filesystem	Size   Used   Avail   Use% 	Mounted on
devtmpfs        	807M     	0   807M      0% 	/dev
tmpfs           	838M     	0   838M      0% 	/dev/shm
tmpfs           	335M  5.2M   330M      2% 	/run
/dev/sda2        	16G    5.6G    11G     35% 	/
tmpfs           	168M    40K  168M      1% 	/run/user/0



[root@Server-A ~]# df  -h  / | grep /
/dev/sda2        16G  5.6G   11G  35% /			# / 디렉터리 사용량 확인  (이중 사용량만 따로 검색해야 한다.)



[root@Server-A ~]# df  -h  / | grep / | awk '{print $5}'	# 5번째 필드만 확인  (정수 비교를 해야하므로 %는 삭제해야 한다.)
35%


[root@Server-A ~]# df  -h  / | grep / | awk '{print $5}' | tr '%'  '!'	# tr = 치환
35!


[root@Server-A ~]# df  -h  / | grep / | awk '{print $5}' | tr  -d '%'	# %를 삭제
35




[root@Server-A ~]# vi  ./script/check_disk.sh
#!/bin/bash

use=$(df  -h  / | grep / | awk '{print $5}' | tr  -d '%')

if  test "$use"  -ge  60; then	# 방법1
if  ((use > 60)); then		# 방법2
    echo "DISK WARNING: ${use}% 사용 중"
else
    echo "DISK OK: ${use}%"
fi

:wq

[root@Server-A ~]# chmod  +x  ./script/check_disk.sh

[root@Server-A ~]# ./script/check_disk.sh
DISK OK: 35%




	# / 디렉터리의 디스크 사용량을 60% 이상으로 만들기위해 대용량 파일 생성

[root@Server-A ~]# mkdir  /disk_test
[root@Server-A ~]# dd if=/dev/zero of=/disk_test/bigfile bs=1G count=5 status=progress



[root@Server-A ~]# df  -h  / | grep / | awk '{print $5}' | tr -d '%'
66



[root@Server-A ~]# ./script/check_disk.sh
DISK WARNING: 66% 사용 중



	# 확인 후 /disk_test 디렉터리 삭제

[root@Server-A ~]# rm  -rf  /disk_test


[root@Server-A ~]# ./script/check_disk.sh
DISK OK: 35%
```

**정리**: 조건문을 실행 가능한 `.sh` 파일로 만들어 두면 반복 실행, 자동화(cron 연동), 유지보수, 공유, 재현성 확보가 쉬워지며, 실무에서는 디스크/로그/서비스 상태 점검 스크립트로 자주 활용된다.

## read 명령어

**read** 명령은 사용자로부터 키보드 입력을 받아 변수에 저장하는 쉘 내장(Built-in) 명령어이다.

- 쉘 스크립트에서 실행 중 사용자의 이름, 나이, 비밀번호 등 다양한 정보를 입력받을 때 사용한다.
- 기본적으로 read는 표준 입력(STDIN) 인 키보드에서 값을 읽어 지정한 변수에 저장한다.

### read의 동작 원리

사용자가 키보드로 값을 입력하면 read 명령이 입력을 읽어 변수에 저장한다.

형식 : `read 변수1 변수2 변수3`

예제)
```bash
[root@Server-A ~]# read  name  age
sol 33


[root@Server-A ~]# echo $name	# name 변수의 값 확인
sol

[root@Server-A ~]# echo $age	# age 변수의 값 확인
33
```

### read -p

**-p** 옵션은 사용자에게 입력 안내(Prompt) 메시지를 출력한 후 입력을 받는다.

형식 : `read -p "메시지" 변수`

예제)
```bash
[root@Server-A ~]# read -p "이름을 입력하세요 : " name
이름을 입력하세요 : hong


[root@Server-A ~]# echo $name	# age 변수의 값 확인
hong
```

### read -s

**-s** 옵션은 입력 내용을 화면에 표시하지 않는다.

예제1)
```bash
#!/bin/bash
read -s -p "비밀번호 : " passwd
echo
echo "비밀번호 : $passwd"
```

예제2)
```bash
[root@Server-A ~]# read -s -p "비밀번호 : " passwd
비밀번호 : [root@Server-A ~]#
[root@Server-A ~]# echo "비밀 번호 : $passwd"
비밀 번호 : sol1234
```

### read -t

**-t** 옵션은 입력 대기 시간을 초 단위로 지정한다.

- 시간 내에 입력하지 않으면 종료된다.

```bash
[root@Server-A ~]# read -s -t 5 -p "비밀번호 : " passwd		# 5초간 입력이 없으면 입력이 자동 종료된다.
비밀번호 : 
[root@Server-A ~]#
```

### read를 사용한 쉘 스크립트

**EX1) 사용자로부터 이름과 나이를 입력받아 화면에 출력하는 쉘 스크립트를 작성하시오.**

```bash
[root@Server-A ~]# vi ./script/name.sh
#!/bin/bash

read -p "이름 입력 : " name
read -p "나이 입력 : " age

echo "입력한 이름은 ${name}입니다."
echo "입력한 나이는 ${age}살입니다."

:wq


[root@Server-A ~]# chmod  +x  ./script/name.sh


[root@Server-A ~]# ./script/name.sh
이름 입력 : 홍길동
나이 입력 : 27
입력한 이름은 홍길동입니다.
입력한 나이는 27살입니다
```

**EX2) 아이디, 비밀번호를 입력받는 스크립트를 작성하시오**
- 입력한 비밀번호는 화면에 표시되지 않아야 하며, 입력 제한 시간은 10초로 설정

```bash
[root@Server-A ~]# vi  ./script/password.sh
#!/bin/bash

read -p "아이디 : " uid
read -s -t 10 -p "비밀번호 : " passwd
echo

echo "입력한 아이디 : $uid"
echo "입력한 비밀번호 : $passwd"

:wq


	# 10초간 입력 중지
[root@Server-A ~]# ./script/password.sh
아이디 : root
비밀번호 :			# 10초를 초과하면 스크립트가 종료
입력한 아이디 : root
입력한 비밀번호 :


	# 비밀번호 설정시 화면에 출력되지 않는다.
[root@Server-A ~]# ./script/password.sh
아이디 : root
비밀번호 :			# 설정한 비밀번호가 출력되지 않는다.
입력한 아이디 : root
입력한 비밀번호 : admin1234
```

**EX3) 아래의 조건에 맞게 스크립트를 작성하시오.**
- 복사할 파일의 경로를 입력받아야 한다.
- 복사할 대상 디렉터리의 경로를 입력받아야 한다.
- 입력받은 파일을 대상 디렉터리로 복사하여야 한다.
- 파일이 복사되면 "XXX 파일이 복사 되었습니다." (XXX = 입력받은 파일명)

```bash
[root@Server-A ~]# path="/temp/soldesk/test/passwd.txt"

[root@Server-A ~]# echo $(basename $path)
passwd.txt



	# 방법1
[root@Server-A ~]# vi ./script/file_copy.sh
#!/bin/bash

read -p "복사할 파일의 경로 : " src
read -p "복사할 파일의 대상 경로 : " dest

cp  -r  "$src"  "$dest"

if [ -f "$dest/$(basename $src)" ]
then
    echo "$(basename ${src})" 파일이 복사되었습니다.
fi

:wq


[root@Server-A ~]# ./script/file_copy.sh
복사할 파일의 경로 : /etc/passwd
복사할 파일의 대상 경로 : /temp
passwd 파일이 복사되었습니다.



	# 방법2
[root@Server-A ~]# vi ./script/file_copy2.sh
#!/bin/bash

read -p "복사할 파일의 경로 : " src
read -p "복사할 파일의 대상 경로 : " dest
filename=$(basename $src)

cp  -r  "$src"  "$dest"

if [ -f "$dest/${filename}" ]
then
    echo "${filename}" 파일이 복사되었습니다.
fi

:wq



[root@Server-A ~]# chmod +x ./script/file_copy2.sh

[root@Server-A ~]# rm -rf /temp/*

[root@Server-A ~]# ./script/file_copy2.sh
복사할 파일의 경로 : /etc/passwd
복사할 파일의 대상 경로 : /temp
passwd 파일이 복사되었습니다.




	# 방법3
[root@Server-A ~]# vi ./script/file_copy3.sh
#!/bin/bash

read -p "복사할 파일의 경로 : " src
read -p "복사할 파일의 대상 경로 : " dest
filename=$(basename $src)
target="$dest/filename"

if [ -d "$target" ]
then
    rm -rf "$target"
fi

cp  -r  "$src"  "$dest"


if [ -f "$targe" ]
then
    echo "${filename}" 파일이 복사되었습니다."
else
    echo "${filename}" 파일복사에 실패했습니다."
fi

:wq
```

**EX4) 아래의 조건에 맞게 스크립트를 작성하시오**
- 사용자로부터 경로를 입력 받아야 한다.
- 사용자로부터 파일 또는 디렉터리명을 입력받아야 한다.
- 사용자로부터 입력받은 경로와 파일 또는 디렉터리명을 사용하여 파일이면 "파일 입니다."가 출력되고 디렉터리이면 "디렉터리 입니다."가 출력되어야 한다.
- 파일 또는 디렉터리가 없으면 "존재하지 않는 파일 또는 디렉터리입니다."가 출력되어야 한다.

```bash
[root@Server-A ~]# vi  ./script/check_file_dir.sh
#!/bin/bash

read  -p  "경로 입력 : " path
read  -p  "파일 또는 디렉터리명 입력 : " name

# 파일인 경우
if [ -f "$path/$name" ]
then
   echo "${name}는 파일입니다."

# 디렉터리인 경우
elif [ -d "$path/$name" ]
then
   echo "${name}는 디렉터리입니다."

# 파일 또는 디렉터리가 없는 경우
else 
   echo "${name}는 존재하지 않는 파일 또는 디렉터리 입니다."
fi

: wq


[root@Server-A ~]# chmod  +x  ./script/check_file_dir.sh


[root@Server-A ~]# ./script/check_file_dir.sh
경로 입력 :/etc/ssh/
파일 또는 디렉터리명 입력 : sshd_config
sshd_config는 파일입니다			# 파일 확인


[root@Server-A ~]# ./script/check_file_dir.sh
경로 입력 :/etc/ssh/
파일 또는 디렉터리명 입력 : sshd_config.d
sshd_config.d는 디렉터리입니다.		# 디렉터리 확인



[root@Server-A ~]# ./script/check_file_dir.sh
경로 입력 :/etc/ssh/
파일 또는 디렉터리명 입력 : abc
abc는 존재하지 않는 파일 또는 디렉터리 입니다.
```

**EX5) 아래의 조건에 맞게 스크립트를 작성하시오.**
- 압축할 디렉터리의 경로를 입력받아야 한다.
- 압축파일을 저장할 디렉터리를 입력받아야 한다.
- 입력받은 디렉터리를 tar.gz 형식으로 압축하여야 한다.
- 압축이 완료되면 "XXX.tar.gz 파일이 생성 되었습니다."를 출력하여야 한다.
- (XXX = 디렉터리명)

```bash
[root@Server-A ~]# vi compress.sh
#!/bin/bash

read -p "압축할 디렉터리 : " src
read -p "저장할 디렉터리 : " dest

name=$(basename "$src")

tar -zcf "${dest}/${name}.tar.gz" -C "$(dirname "$src")" "$name"

if [ -f "${dest}/${name}.tar.gz" ]
then
        echo "${name}.tar.gz 파일이 생성 되었습니다."
fi
```

**EX6) 아래의 조건에 맞게 스크립트를 작성하시오.**
- 검색대상인 파일 또는 디렉터리인지 입력받아야 한다.
- 파일 또는 디렉터리명을 입력받아야 한다.
- 입력받은 조건으로 해당 파일 또는 디렉터리의 경로와 파일 또는 디렉터리명이 출력되어야 한다.

```bash
[root@Server-A ~]# find  / -type f  -name  sshd_config
/etc/ssh/sshd_config


[root@Server-A ~]# find  / -type d  -name  sshd_config.d
/etc/ssh/sshd_config.d



	# 방법 1
[root@Server-A ~]# vi  ./script/search.sh
#!/bin/bash

read  -p  "검색대상 [파일(f) / 디렉터리(d)] : " type
read  -p  "파일명 또는 디렉터리명 : " name

if [ "$type" = "f" ]
then
     result=$(find  /  -type  f  -name "$name"  2>/dev/null)

elif [ "$type" = "d" ]
then
     result=$(find  /  -type  d  -name "$name"  2>/dev/null)

else
     echo "잘못된 입력입니다."
fi

if [ -n "$result" ]
then
    echo "$result"

else
    echo "검색 결과가 없습니다."
fi

:wq



[root@Server-A ~]# chmod  +x  ./script/search.sh


[root@Server-A ~]# bash  -n  ./script/search.sh		# 스크립트 파일 에러 체크


[root@Server-A ~]# ./script/search.sh
검색대상 [파일(f) / 디렉터리(d)] : f
파일명 또는 디렉터리명 : sshd_config
/etc/ssh/sshd_config



[root@Server-A ~]# ./script/search.sh
검색대상 [파일(f) / 디렉터리(d)] : d
파일명 또는 디렉터리명 : sshd_config.d
/etc/ssh/sshd_config.d



[root@Server-A ~]# ./script/search.sh
검색대상 [파일(f) / 디렉터리(d)] : f
검색 결과가 없습니다.




	# 방법 2
[root@Server-A ~]# vi  ./script/search2.sh
#!/bin/bash

read  -p  "검색대상 [파일(f) / 디렉터리(d)] : " type

if [ "$type" != "f" ] && [ "$type" != "d" ]
then
     echo "잘못된 입력입니다."
     exit 1				# 스크립트를 중단하고 exit code를 1로 반환
fi

read  -p  "파일명 또는 디렉터리명 : " name

if [ "$type" = "f" ]
then
     result=$(find  /  -type  f  -name "$name"  2>/dev/null)

else [ "$type" = "d" ]
     result=$(find  /  -type  d  -name "$name"  2>/dev/null)

fi

if [ -n "$result" ]
then
    echo "$result"

else
    echo "검색 결과가 없습니다."
fi

:wq



[root@Server-A ~]# ./script/search2.sh
검색대상 [파일(f) / 디렉터리(d)] : s		# f 또는 d가 아니면 스크립트 종료
잘못된 입력입니다.



[root@Server-A ~]# ./script/search.sh
검색대상 [파일(f) / 디렉터리(d)] : f
파일명 또는 디렉터리명 : sshd_config
/etc/ssh/sshd_config



[root@Server-A ~]# ./script/search.sh
검색대상 [파일(f) / 디렉터리(d)] : d
파일명 또는 디렉터리명 : sshd_config.d
/etc/ssh/sshd_config.d



[root@Server-A ~]# ./script/search.sh
검색대상 [파일(f) / 디렉터리(d)] : f
검색 결과가 없습니다.
```

**EX7-1) 아래의 조건에 맞게 스크립트를 작성하시오**
- 생성할 파일의 경로를 입력받아야 한다.
- 생성할 파일 이름을 입력받아야 한다.
- 입력받은 경로와 파일명을 사용하여 해당경로에 파일이 생성어야 한다.
- 파일이 생성되면 "XXX 파일이 생성 되었습니다." (XXX = 입력받은 파일명)

```bash
[root@Server-A ~]# vi ./script/create_file.sh
#!/bin/bash

read -p "생성할 파일의 경로 : " path
read -p "생성할 파일 이름 : " file

touch "$path/${file}"

if [ -f "${path}/${file}" ]
then
        echo "${file} 파일이 생성 되었습니다."
fi

:wq


[root@Server-A ~]# chmod  +x  ./script/create_file.sh


[root@Server-A ~]# ./script/create_file.sh
생성할 파일의 경로 : /temp
생성할 파일 이름 : soldABC
soldABC 파일이 생성 되었습니다.
```

**EX7-2) 아래의 조건에 맞게 스크립트를 작성하시오**
- 생성할 데이터가 파일인지 디렉터리인지 입력받아야 한다.
- 생성할 파일/디렉터리의 경로를 입력받아야 한다.
- 생성할 파일/디렉터리 이름을 입력받아야 한다.
- 입력받은 경로와 파일/디렉터리명을 사용하여 해당경로에 파일/디렉터리가 생성어야 한다.
- 파일이 생성되면 "XXX 파일이 생성 되었습니다."  (XXX = 입력받은 파일명)
- 디렉터리가 생성되면 "XXX 디렉터리가 생성 되었습니다."  (XXX = 입력받은 디렉터리명)

```bash
[root@Server-A ~]# vi ./script/create_file2.sh
#!/bin/bash

read -p "파일(f) / 디렉터리(d) : " type

if [ "$type" != "f" ] && [ "$type" != "d" ]
then
     echo "잘못된 입력입니다."
     exit 1				# 스크립트를 중단하고 exit code를 1로 반환
fi

read -p "생성할 파일의 경로 : " path
read -p "생성할 파일 이름 : " file

if [ "$type" = "f" ]
then
    touch "${path}/${file}"

    if [ -f "${path}/${file}" ]
    then
	echo "${path}/${file} 파일이 생성되었습니다."
    else
	echo "${path}/${file} 파일 생성에 실패했습니다."
    fi
else
    mkdir "${path}/${file}"

    if [ -d "${path}/${file}" ]
    then
	echo "${path}/${file} 디렉터리가 생성되었습니다."
    else
	echo "${path}/${file} 디렉터리 생성에 실패했습니다."
    fi
fi


[root@Server-A ~]# ./script/create_file2.sh
파일(f) / 디렉터리(d) : s			# f , d 이외의 값 입력시 스크립트 종료
잘못된 입력입니다.


[root@Server-A ~]# echo $?
1				# exit code 1 확인


[root@Server-A ~]# ./script/create_file2.sh
파일(f) / 디렉터리(d) : f
생성할 파일의 경로 : /temp/
생성할 파일 이름 : soldpasswd
/temp//soldpasswd 파일이 생성되었습니다.
```

**EX8) 사용자 로그인 확인 스크립트를 다음 조건에 작성하시오**

- 총 3개의 아이디와 비밀번호를 각각 변수에 저장해야 한다.

  | 아이디 | 비밀번호 |
  |---|---|
  | user1 | passwd11 |
  | user2 | passwd22 |
  | user3 | passwd33 |

- 사용자로부터 아이디와 비밀번호를 입력받아야 한다. (비밀번호 입력 시 입력 내용이 화면에 표시되지 않아야 한다.)
- 입력한 아이디와 비밀번호가 저장된 계정 정보 중 하나와 모두 일치하면 "로그인 성공" 메시지를 출력해야 한다.
- 아이디가 존재하지 않거나 비밀번호가 일치하지 않으면 "아이디 또는 비밀번호를 확인하세요" 메시지를 출력해야 한다.

```bash
[root@Server-A ~]# vi ./script/login.sh
#!/bin/bash

id1="user1"
pw1="pw11"

id2="user2"
pw2="pw22"

id3="user3"
pw3="pw33"

read  -p  "아이디 : " input_id
read  -s  -p  "비밀번호 : " input_pw
echo

if [ -z "$input_id" ] || [ -z "$input_pw" ]
then
    echo "아이디와 비밀번호를 모두 입력하세요"
fi

if [ "$input_id" = "$id1" ] && [ "$input_pw" = "$pw1" ]
then
    echo "로그인 성공"
    echo "${input_id}님이 로그인 하셨습니다."

elif [ "$input_id" = "$id2" ] && [ "$input_pw" = "$pw2" ] 
then
    echo "로그인 성공"
    echo "${input_id}님이 로그인 하셨습니다."

elif [ "$input_id" = "$id3" ] && [ "$input_pw" = "$pw3" ] 
then
    echo "로그인 성공"
    echo "${input_id}님이 로그인 하셨습니다."

else
    echo "아이디 또는 비밀번호를 확인하세요"
fi

:wq


[root@Server-A ~]# chmod  +x  ./script/login.sh



[root@Server-A ~]# bash -n  ./script/login.sh


[root@Server-A ~]# ./script/login.sh
아이디 : user1		# 아이디 : user1
비밀번호 :			# 비밀번호 :  pw11
로그인 성공
user1님이 로그인 하셨습니다.


[root@Server-A ~]# ./script/login.sh
아이디 : user1		# 아이디 : user1
비밀번호 :			# 비밀번호 :  pw22
아이디 또는 비밀번호를 확인하세요
```

**정리**: **read**는 표준 입력을 변수에 저장하는 명령으로, `-p`(프롬프트), `-s`(입력 숨김), `-t`(제한 시간) 옵션을 조건문과 결합하면 로그인, 파일 생성/복사, 검색 등 대화형 스크립트를 만들 수 있다.

## case 문

**case** 문은 하나의 변수(또는 값)에 대해 여러 가지 패턴을 비교하여 해당 패턴에 맞는 코드를 실행하는 흐름 제어문이다.

- **if** 문은 조건이 많아질수록 코드가 길어지고 복잡해지지만 **case** 문은 조건이 많을수록 더 깔끔하고 가독성이 좋아진다.

**사용 목적**
- 메뉴 선택
- 파일 확장자 처리
- 서비스 상태 값에 따른 분기 처리
- 사용자 입력 처리 (y/n, 1/2/3 선택 등)
- 문자열 비교를 많이 해야 하는 스크립트에서 사용

**case 문 기본 형식**

```bash
case 변수 in
    패턴1)
        명령어1 ;;
    패턴2)
        명령어2  ;;
    패턴3 | 패턴4)
        명령어3  ;;
    *)
        기본 실행문 ;;
esac
```

- `case 변수 in` : 검사할 대상(문자열 또는 숫자) 지정
- `패턴)` : 값이 패턴과 일치하면 아래 명령어 실행
- `;;` : 명령어 종료 표시
- `*)` : 어떠한 패턴에도 해당하지 않을 때 실행(else 역할)
- `esac` : case 문 종료 표시 (case를 거꾸로 읽으면 esac)

**EX1) 번호 입력에 따라 서로 다른 작업을 수행하는 메뉴 스크립트 작성**

```bash
[root@Server-A ~]# read  cmd
1


[root@Server-A ~]# echo $cmd
1


[root@Server-A ~]# vi  ./script/number_select.sh
#!/bin/bash

read -p "정수입력(1/2/3) : " num

case  $num  in
 1)
    echo "1번 선택) cmd 1 running";;
 2)
    echo "2번 선택) cmd 2 running";;
 3)
    echo "3번 선택) cmd 3 running";;
 *)
    echo "잘못된 입력입니다.";;

esac

:wq


[root@Server-A ~]# chmod  +x  ./script/number_select.sh


[root@Server-A ~]# ls  -l  ./script/number_select.sh
-rwxr-xr-x 1 root root 248  7월 27 14:56 ./script/number_select.sh


[root@Server-A ~]# bash  -n  ./script/number_select.sh


[root@Server-A ~]# ./script/number_select.sh
정수입력(1/2/3) : 1
1번 선택) cmd 1 running


[root@Server-A ~]# ./script/number_select.sh
정수입력(1/2/3) : 3
3번 선택) cmd 3 running


[root@Server-A ~]# ./script/number_select.sh
정수입력(1/2/3) : 5
잘못된 입력입니다.
```

**EX2) 아래의 조건에 맞게 쉘 스크립트를 작성해야 한다.**
- "계속 진행하시겠습니까? (y/n):" 문자열이 출력되어야 한다.
- 사용자가 y 또는 Y 를 입력하면 "YES 선택 - 게임을 계속 진행합니다." 가 출력되어야 한다.
- 사용자가 n 또는 N 를 입력하면 "NO 선택 - 게임을 작업을 종료합니다." 가 출력되어야 한다.
- 이외의 문자를 입력하면 "잘못된 입력입니다. y 또는 n을 입력하세요." 가 출력되어야 한다.

```bash
	# 방법 1
[root@Server-A ~]# vi  ./script/yesno.sh
#!/bin/bash

read -p "게임을 계속 진행하시겠습니까? (y/n) : " answer

case "$answer" in
        y|Y)
                echo "YES 선택 - 계속 진행합니다.";;
        n|N)
                echo "NO 선택 - 작업을 종료합니다.";;
        *)
                echo "잘못 입력했습니다. y 또는 n를 입력하세요";;
esac

:wq


	# 방법 2
[root@Server-A ~]# vi  ./script/yesno.sh
#!/bin/bash

read -p "게임을 계속 진행하시겠습니까? (y/n) : " answer

case "$answer" in
        y|Y|yes|YES)
                echo "YES 선택 - 계속 진행합니다.";;
        n|N|no|NO)
                echo "NO 선택 - 작업을 종료합니다.";;
        *)
                echo "잘못 입력했습니다. y 또는 n를 입력하세요";;
esac

:wq



	# 방법 3
[root@Server-A ~]# vi  ./script/yesno.sh
#!/bin/bash

shopt -s  nocasematch	# 대/소문자를 구분하지 않는다.

read -p "게임을 계속 진행하시겠습니까? (y/n) : " answer

case "$answer" in
        y|yes)
                echo "YES 선택 - 게임을 계속 진행합니다.";;
        n|no)
                echo "NO 선택 - 게임을 종료합니다.";;
        *)
                echo "잘못 입력했습니다. y 또는 n를 입력하세요";;
esac

:wq


[root@Server-A ~]# ./script/yesno.sh
게임을 계속 진행하시겠습니까? (y/n) : yes
YES 선택 - 게임을 계속 진행합니다.


[root@Server-A ~]# ./script/yesno.sh
게임을 계속 진행하시겠습니까? (y/n) : no
NO 선택 - 게임을 종료합니다.


[root@Server-A ~]# ./script/yesno.sh
게임을 계속 진행하시겠습니까? (y/n) : ye
잘못 입력했습니다. y 또는 n를 입력하세요
```

**EX3-1) 다음 조건에 맞게 vsftpd_service.sh 쉘 스크립트를 작성하시오.**
- 스크립트를 실행하면 install / start / stop / restart / status 중 하나를 입력받아야 한다.
- install 입력 시 dnf 명령으로 vsftpd 패키지를 설치해야 한다.
- start, stop, restart 입력 시 systemctl 명령으로 vsftpd 서비스를 제어해야 한다.
- status 입력 시 vsftpd 서비스의 Active 상태를 출력해야 한다.
- 각 작업 완료 후 결과 메시지와 서비스의 Active 상태를 출력해야 한다.
- 지정된 값 이외의 값을 입력하면 "잘못 입력하셨습니다."를 출력해야 한다.
- 여러 입력값을 구분하기 위해 case 문을 사용해야 한다.

```bash
[root@Server-A ~]# vi ./script/vsftpd_service.sh
#!/bin/bash

read  -p  "install/ start / stop / restart /status 중 하나를 입력하세요:" cmd

case "$cmd" in
        install)
                    dnf  install -y  vsftpd
                    echo "vsftpd 설치 완료"
                    echo "상태 :  $(systemctl is-active vsftpd)";;
        start)
                    systemctl start vsftpd
                    echo "vsftpd Start"
                    echo "상태 :  $(systemctl is-active vsftpd)";;
        stop)
                    systemctl stop vsftpd
                    echo "vsftpd Stop"
                    echo "상태 :  $(systemctl is-active vsftpd)";;
        restart)
                    systemctl restart vsftpd
                    echo "vsftpd Restart"
                    echo "상태 :  $(systemctl is-active vsftpd)";;
        *)
                    echo "잘못 입력하셨습니다.";;
esac

:wq



[root@Server-A ~]# ./script/vsftpd_service.sh
install/ start / stop / restart /status 중 하나를 입력하세요:install
마지막 메타자료 만료확인(2:00:36 이전): 2026년 07월 27일 (월) 오후 01시 45분 13초.
종속성이 해결되었습니다.
=========================================================================================
 꾸러미            구조              버전                     저장소                크기
=========================================================================================
설치 중:
 vsftpd            x86_64            3.0.5-8.el9              appstream            157 k

연결 요약
=========================================================================================
설치  1 꾸러미

전체 내려받기 크기: 157 k
설치된 크기 : 347 k
꾸러미 내려받기 중:
vsftpd-3.0.5-8.el9.x86_64.rpm                            2.2 MB/s | 157 kB     00:00
-----------------------------------------------------------------------------------------
합계                                                     225 kB/s | 157 kB     00:00
연결 확인 실행 중
연결 확인에 성공했습니다.
연결 시험 실행 중
연결 시험에 성공했습니다.
연결 실행 중
  준비 중     :                                                                      1/1
  설치 중     : vsftpd-3.0.5-8.el9.x86_64                                            1/1
  구현 중     : vsftpd-3.0.5-8.el9.x86_64                                            1/1
  확인 중     : vsftpd-3.0.5-8.el9.x86_64                                            1/1

설치되었습니다:
  vsftpd-3.0.5-8.el9.x86_64

완료되었습니다!
vsftpd 설치 완료
inactive



[root@Server-A ~]# ./script/vsftpd_service.sh
install/ start / stop / restart /status 중 하나를 입력하세요:start
vsftpd Start
상태 :  active


[root@Server-A ~]# ./script/vsftpd_service.sh
install/ start / stop / restart /status 중 하나를 입력하세요:stop
vsftpd Stop
상태 :  inactive
```

**EX3-2) 다음 조건에 맞게 server_service.sh 쉘 스크립트를 작성하시오.**
- 설정할 서버 이름으로 sshd, vsftpd, httpd 중 하나를 입력받아야 한다.
- install / start / stop / restart / status 중 하나를 입력받아야 한다.
- install 입력 시 선택한 서버의 패키지를 설치해야 한다.
- start, stop, restart 입력 시 선택한 서비스를 제어해야 한다.
- status 입력 시 선택한 서비스의 Active 상태를 출력해야 한다.
- 각 작업 완료 후 서버 이름과 작업 결과를 출력해야 한다.
- 지정된 값 이외의 값을 입력하면 "잘못 입력하셨습니다."를 출력해야 한다.
- 여러 입력값을 구분하기 위해 case 문을 사용해야 한다.

**EX3-2) 다음 조건에 맞게 server_service.sh 쉘 스크립트를 작성하시오.**
- 설정할 서버 이름으로 sshd, vsftpd, httpd 중 하나를 입력받아야 한다.
- install / start / stop / restart / status 중 하나를 입력받아야 한다.
- install 입력 시 선택한 서버의 패키지를 설치해야 한다.
- start, stop, restart 입력 시 선택한 서비스를 제어해야 한다.
- status 입력 시 선택한 서비스의 Active 상태를 출력해야 한다.
- 각 작업 완료 후 서버 이름과 작업 결과를 출력해야 한다.
- 지정된 값 이외의 값을 입력하면 "잘못 입력하셨습니다."를 출력해야 한다.
- 여러 입력값을 구분하기 위해 case 문을 사용해야 한다.

```bash
[root@Server-A abc]# vi  ./script/service_script.sh
#!/bin/bash

read  -p  "설정할 Server(sshd, vsftp, httpd)이름을 입력하세요 : " service
read  -p  "install/ start / stop / restart /status 중 하나를 입력하세요 : " cmd

case "$cmd" in
        install)
                    if [ "$service" = "sshd" ]
                    then
                        package="openssh-server"
                    else
                        package="$service"

                    dnf  install -y  "$package"  >  /dev/null  2>&1		# 설치과정을 생략시 : /dev/null  2>&1	
                    echo ""$service" 설치 완료"
                    echo "상태 : $(systemctl is-active "$service")"
                    ;;
        start)
                    systemctl start "$service"
                    echo ""$service" Start"
                    echo "상태 : $(systemctl is-active "$service")"
                    ;;
        stop)
                    systemctl stop "$service"
                    echo ""$service" Stop"
                    echo "상태 : $(systemctl is-active "$service")"
                    ;;
        restart)
                    systemctl restart "$service"
                    echo ""$service" Restart"
                    echo "상태 : $(systemctl is-active "$service")"
                    ;;
        status)
                    echo "상태 : $(systemctl is-active vsftpd)"
                    ;;
        *)
                    echo "잘못 입력하셨습니다.";;
esac


[root@Server-A ~]# chmod  +x  ./script/service_script.sh




[root@Server-A ~]#  ./script/service_script.sh		# httpd 설치
설정할 Server(sshd, vsftp, httpd)이름을 입력하세요 : httpd
install/ start / stop / restart /status 중 하나를 입력하세요 : install
httpd 설치 완료
상태 : inactive



[root@Server-A ~]#  ./script/service_script.sh		# httpd 서비스 시작
설정할 Server(sshd, vsftp, httpd)이름을 입력하세요 : httpd
install/ start / stop / restart /status 중 하나를 입력하세요 : start
httpd Start
상태 : active


[root@Server-A ~]#  ./script/service_script.sh		# httpd 서비스 종료
설정할 Server(sshd, vsftp, httpd)이름을 입력하세요 : httpd
install/ start / stop / restart /status 중 하나를 입력하세요 : stop
httpd Stop
상태 : inactive
```

**EX4) 사용자가 서비스 이름과 동작(add/remove)을 입력하면, 해당 서비스에 대해 방화벽을 영구적으로 추가하거나 삭제하는 스크립트를 작성하시오.**
- "서비스 이름" 을 먼저 입력받는다. (예: ssh, http, https, ftp, dns등)
- "add 또는 remove" 중 하나를 입력받는다.
- add : `firewall-cmd --permanent --add-service=서비스명`
- remove : `firewall-cmd --permanent --remove-service=서비스명`
- 작업 후 자동으로 `firewall-cmd --reload` 실행 및 서비스 리스트 출력
- 그 외 입력은 오류 메시지 출력 ("지원하지 않는 명령입니다. add 또는 remove 를 입력하세요.")

```bash
	# 방법 1
[root@Server-A ~]# vi  ./script/firewall_add_remove.sh
#!/bin/bash

read -p "방화벽에 적용할 서비스명을 입력하세요 (예: http, https, ftp): " servicename

read -p "서비스를 방화벽에 추가(add) 또는 삭제(remove)하세요: " action

case "$action" in
    add)
        firewall-cmd --permanent --add-service="$servicename"
        firewall-cmd --reload
        firewall-cmd --list-service
        echo "$servicename 서비스 방화벽 추가 완료"
        ;;

    remove)
        firewall-cmd --permanent --remove-service="$servicename"
        firewall-cmd --reload
        firewall-cmd --list-service
        echo "$servicename 서비스 방화벽 삭제 완료"
        ;;
    *)

        echo "지원하지 않는 명령입니다. add 또는 remove 를 입력하세요."
        ;;
esac




	# 방법 2
[root@Server-A ~]# vi  ./script/firewall_add_remove.sh
#!/bin/bash

read -p "방화벽에 적용할 서비스명을 입력하세요 (예: http, https, ftp): " servicename

read -p "서비스를 방화벽에 추가(add) 또는 삭제(remove)하세요: " action

case "$action" in
    add)
        if firewall-cmd --permanent --add-service="$servicename"  >  /dev/null  2>&1	# add 성공시 exit code=0 : then 실행, 실패시 exit code=1 else 실행
        then
            firewall-cmd --reload  >  /dev/null  2>&1
            firewall-cmd --list-service  >  /dev/null  2>&1
            echo "$servicename 서비스 방화벽 추가 완료"
        else
                echo "$servicename 서비스 방화벽 추가 실패"
        fi
        ;;

    remove)

        if firewall-cmd --permanent --remove-service="$servicename"  >  /dev/null  2>&1	# remove 성공시 exit code=0 : then 실행, 실패시 exit code=1 else 실행
        then
                firewall-cmd --reload  >  /dev/null  2>&1
                firewall-cmd --list-service  >  /dev/null  2>&1
                echo "$servicename 서비스 방화벽 삭제 완료"
        else
                echo "$servicename 서비스 방화벽 삭제 실패"
        fi
        ;;

    *)
        echo "지원하지 않는 명령입니다. add 또는 remove 를 입력하세요."
        ;;
esac
```

**정리**: **case** 문은 하나의 변수 값을 여러 패턴(`패턴1)`, `패턴2|패턴3)`, `*)`)과 비교해 분기하는 문법으로, 메뉴 선택이나 서비스 제어(install/start/stop/restart/status), add/remove 같은 다중 분기 스크립트에서 `if-elif`보다 가독성 좋게 사용할 수 있다.

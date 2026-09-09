# Shell Script 패턴 매칭 (Pattern Matching / Globbing)

`case`문의 기본 문법 자체는 `03-conditions.md`에 정리되어 있다. 이 문서는 `case`, `[[ ]]`, 파일명 확장(globbing)에서 공통으로 쓰이는 **패턴 매칭 기호의 의미**에 집중한다.

## 기본 글롭 패턴 (\* · ? · [...])

| 패턴 | 의미 |
|---|---|
| `*` | 길이 0을 포함한 모든 문자열과 매치 |
| `?` | 임의의 문자 정확히 1개와 매치 |
| `[abc]` | 대괄호 안의 문자 중 하나와 매치 (a, b, c 중 하나) |
| `[a-z]` | 범위 지정, a부터 z 사이의 문자 하나와 매치 |
| `[!abc]` 또는 `[^abc]` | 대괄호 안의 문자를 **제외한** 문자 하나와 매치 |

```bash
[root@Server-A ~]# vi ./script/pattern_case.sh
#!/bin/bash

read -p "파일 확장자를 입력하세요 : " file

case "$file" in
    *.txt)
        echo "텍스트 파일입니다."
        ;;
    *.tar.gz | *.tgz)
        echo "압축 파일입니다."
        ;;
    data???.log)
        echo "data 뒤에 정확히 3글자 + .log 형식입니다."
        ;;
    [0-9]*)
        echo "숫자로 시작하는 이름입니다."
        ;;
    *)
        echo "알 수 없는 형식입니다."
        ;;
esac

:wq


[root@Server-A ~]# chmod +x ./script/pattern_case.sh

[root@Server-A ~]# ./script/pattern_case.sh
파일 확장자를 입력하세요 : backup.tar.gz
압축 파일입니다.

[root@Server-A ~]# ./script/pattern_case.sh
파일 확장자를 입력하세요 : data123.log
data 뒤에 정확히 3글자 + .log 형식입니다.

[root@Server-A ~]# ./script/pattern_case.sh
파일 확장자를 입력하세요 : 2026report.pdf
숫자로 시작하는 이름입니다.
```

- 같은 패턴 문법은 `[[ 문자열 == 패턴 ]]`에서도 그대로 쓸 수 있다. (`[[ ]]`에서 `==`의 오른쪽은 문자열이 아니라 패턴으로 해석된다.)

```bash
[root@Server-A ~]# filename="report_2026.txt"

[root@Server-A ~]# [[ "$filename" == *.txt ]] && echo "txt 파일 맞음"
txt 파일 맞음

[root@Server-A ~]# [[ "$filename" == report_????.* ]] && echo "형식 일치"
형식 일치
```

- 파일명 확장(globbing)에서도 동일한 기호가 쓰인다. `*.log`처럼 쉘이 명령 실행 전에 패턴에 맞는 실제 파일 목록으로 치환하는 것을 말한다.

```bash
[root@Server-A ~]# ls /var/log/*.log
/var/log/boot.log  /var/log/dnf.log
```

**정리**: `*`(0개 이상 임의 문자), `?`(문자 1개), `[...]`/`[!...]`(문자 집합)는 `case`, `[[ == ]]`, 파일명 확장에서 공통으로 쓰이는 Bash의 기본 패턴 매칭 문법이다.

## POSIX 문자 클래스 ([[:class:]])

- 대괄호 패턴 안에서 `a-z`, `0-9` 같은 범위 대신 **의미가 정해진 문자 클래스**를 사용할 수 있다.

| 클래스 | 의미 |
|---|---|
| `[[:digit:]]` | 숫자 0-9 |
| `[[:alpha:]]` | 알파벳 문자 |
| `[[:alnum:]]` | 알파벳 + 숫자 |
| `[[:space:]]` | 공백류 문자(스페이스, 탭 등) |
| `[[:upper:]]` / `[[:lower:]]` | 대문자 / 소문자 |
| `[[:punct:]]` | 특수 기호(구두점) |

```bash
[root@Server-A ~]# code="A1234"

[root@Server-A ~]# [[ "$code" == [[:upper:]][[:digit:]][[:digit:]][[:digit:]][[:digit:]] ]] && echo "형식 일치"
형식 일치
```

**정리**: POSIX 문자 클래스는 `[a-zA-Z0-9]`처럼 범위를 여러 번 나열하지 않고, `[[:alnum:]]`처럼 의미 단위로 문자 집합을 지정할 수 있게 해준다.

## extglob 확장 패턴

- 기본 글롭만으로는 "이 패턴이 없는 것", "패턴이 여러 번 반복되는 것" 같은 표현이 어렵다. Bash는 `shopt -s extglob`으로 **확장 패턴 매칭**을 켤 수 있다.

```bash
[root@Server-A ~]# shopt -s extglob		# extglob 옵션 활성화
```

| 패턴 | 의미 |
|---|---|
| `?(패턴)` | 패턴이 0번 또는 1번 |
| `*(패턴)` | 패턴이 0번 이상 |
| `+(패턴)` | 패턴이 1번 이상 |
| `@(패턴)` | 패턴 중 정확히 1개와 일치 (여러 패턴을 `|`로 나열 가능) |
| `!(패턴)` | 패턴과 일치하지 **않는** 것 |

```bash
[root@Server-A ~]# shopt -s extglob

[root@Server-A ~]# vi ./script/extglob_test.sh
#!/bin/bash
shopt -s extglob

read -p "값을 입력하세요 : " val

case "$val" in
    +([0-9]))				# 숫자 1개 이상으로만 이루어져 있는지
        echo "숫자로만 이루어져 있습니다."
        ;;
    @(yes|no|y|n))			# yes/no/y/n 중 하나인지
        echo "Yes/No 응답입니다."
        ;;
    !(*.tmp))				# .tmp로 끝나지 않는 모든 것
        echo ".tmp 파일이 아닙니다."
        ;;
    *)
        echo "그 외 형식입니다."
        ;;
esac

:wq


[root@Server-A ~]# chmod +x ./script/extglob_test.sh

[root@Server-A ~]# ./script/extglob_test.sh
값을 입력하세요 : 20260910
숫자로만 이루어져 있습니다.

[root@Server-A ~]# ./script/extglob_test.sh
값을 입력하세요 : yes
Yes/No 응답입니다.

[root@Server-A ~]# ./script/extglob_test.sh
값을 입력하세요 : report.log
.tmp 파일이 아닙니다.
```

- 파일 삭제/이동에서도 유용하다. 예를 들어 "`.log`가 아닌 파일만 삭제"처럼 제외 조건을 표현할 때 `!(패턴)`이 자주 쓰인다.

```bash
[root@Server-A ~]# shopt -s extglob

[root@Server-A ~]# ls /tmp/work/
a.log  b.log  c.tmp  d.bak

[root@Server-A ~]# rm /tmp/work/!(*.log)		# .log가 아닌 파일만 삭제

[root@Server-A ~]# ls /tmp/work/
a.log  b.log
```

**정리**: `extglob`은 `shopt -s extglob`으로 활성화하며, `?()`, `*()`, `+()`, `@()`, `!()`로 "0/1회", "0회 이상", "1회 이상", "다중 후보 중 하나", "제외" 같은 정규식 수준의 표현을 `case`나 파일명 패턴에서 사용할 수 있게 해준다.

## 패턴 매칭이 실제로 쓰이는 곳

| 위치 | 사용 예 |
|---|---|
| `case` 문 | `case "$1" in start\|stop) ... ;; esac` |
| `[[ == ]]` / `[[ != ]]` | `[[ "$file" == *.sh ]]` |
| 파라미터 확장 `#`/`##`/`%`/`%%` | `${path##*/}` (08-syntax-master.md 참고, 여기서도 같은 글롭 패턴 문법을 사용) |
| 파일명 확장(globbing) | `ls *.conf`, `rm !(*.log)` |

**정리**: 글롭 패턴은 `case`, `[[ ]]`, 파라미터 확장, 파일명 확장까지 Bash 전반에서 공통으로 쓰이는 하나의 문법 체계이며, `extglob`을 켜면 정규식과 비슷한 수준의 표현력까지 확보할 수 있다.

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

## ${var/패턴/대체} 치환에서의 글롭 패턴

- `${var/패턴/대체}`(첫 번째 매치만 치환), `${var//패턴/대체}`(전체 매치를 모두 치환)에서 사용하는 "패턴"은 **정규식이 아니라 이 문서에서 다룬 글롭 패턴**이다. `*`, `?`, `[...]`가 여기서도 똑같이 동작한다.

```bash
[root@Server-A ~]# msg="hello world, hello bash"

[root@Server-A ~]# echo "${msg/hello/HI}"		# 첫 번째 hello만 치환
HI world, hello bash

[root@Server-A ~]# echo "${msg//hello/HI}"		# 전체 hello를 모두 치환
HI world, HI bash

[root@Server-A ~]# path="/var/log/messages.log"
[root@Server-A ~]# echo "${path//[0-9]/#}"		# 글롭 문자 집합도 그대로 사용 가능
/var/log/messages.log
```

- 패턴을 앞/뒤 위치에 고정하고 싶을 때는 `#`(맨 앞부터), `%`(맨 뒤부터)를 패턴 앞에 붙인다. 이 문법은 `${path##*/}` 같은 잘라내기 확장과 헷갈리기 쉬우니 구분해서 알아둔다.

```bash
[root@Server-A ~]# file="backup_2026.tar.gz"

[root@Server-A ~]# echo "${file/#backup/archive}"	# 문자열 맨 앞이 backup으로 시작할 때만 치환
archive_2026.tar.gz

[root@Server-A ~]# echo "${file/%.gz/.zip}"		# 문자열 맨 뒤가 .gz로 끝날 때만 치환
backup_2026.tar.zip
```

- extglob을 켜면 치환에서도 `+()`, `!()` 같은 확장 패턴을 그대로 사용할 수 있다.

```bash
[root@Server-A ~]# shopt -s extglob
[root@Server-A ~]# num="a1b22c333"

[root@Server-A ~]# echo "${num//+([0-9])/N}"		# 연속된 숫자 뭉치를 하나의 N으로 치환
aNbNcN
```

**정리**: `${var/패턴/대체}`, `${var//패턴/대체}`의 "패턴"은 정규식이 아니라 `*`, `?`, `[...]` 같은 글롭 패턴이며, `#`/`%`를 붙여 맨 앞/맨 뒤 고정 매치를 지정할 수 있고 `extglob`을 켜면 `+()`, `!()` 같은 확장 패턴도 치환에 그대로 활용할 수 있다.

## 글롭 패턴 vs 정규식, 이름은 비슷해도 문법은 다르다

- `case`, `[[ == ]]`, 파일명 확장에서 쓰는 **글롭(glob) 패턴**과 `[[ =~ ]]`, `grep -E`, `sed -E`에서 쓰는 **정규식(regex)**은 `*`, `?`, `[...]` 같은 기호를 공유하지만 **의미가 서로 다르다.** 이 둘을 섞어 쓰면 의도와 다른 결과가 나오는 대표적인 실수 포인트다.

| 기호 | 글롭에서의 의미 | 정규식(ERE)에서의 의미 |
|---|---|---|
| `*` | 직전 위치와 무관하게 "모든 문자열"(단독으로도 전체 매치) | 직전 문자가 0번 이상 반복(단독으로 못 씀, 항상 앞 문자에 붙음) |
| `?` | 임의의 문자 1개 | 직전 문자가 0번 또는 1번 |
| `.` | 그냥 점 문자 그대로 | 임의의 문자 1개 |
| `[...]` | 문자 집합(둘 다 거의 동일) | 문자 집합(거의 동일) |
| 전체 매치 기준 | 패턴이 문자열 전체와 일치해야 함(암묵적으로 `^`, `$`가 있는 것처럼 동작) | 패턴이 문자열 일부에만 있어도 매치(부분 일치 허용) |

```bash
[root@Server-A ~]# name="report2026.txt"

[root@Server-A ~]# [[ "$name" == *2026* ]] && echo "글롭 매치"	# 글롭: *는 임의 문자열
글롭 매치

[root@Server-A ~]# [[ "$name" =~ 2026 ]] && echo "정규식 매치"	# 정규식: 부분 문자열만 있어도 매치
정규식 매치

[root@Server-A ~]# [[ "$name" =~ ^report.*txt$ ]] && echo "정규식 매치(명시적 앵커)"
정규식 매치(명시적 앵커)

[root@Server-A ~]# [[ "$name" == report.*txt ]] && echo "글롭 매치" || echo "글롭 불일치"
글롭 불일치				# 글롭에서 '.'은 그냥 점 문자라서 report.*txt와 report2026.txt는 불일치
```

- `grep`도 기본 정규식(BRE)과 확장 정규식(ERE, `-E`)이 다르다. `+`, `?`, `|`, `()`를 메타문자로 쓰려면 `grep -E`(또는 `egrep`)를 쓰거나 `grep`에서 `\+`, `\?`처럼 이스케이프해야 한다.

```bash
[root@Server-A ~]# echo "aaa123" | grep -E '[0-9]+'		# ERE: +가 메타문자로 바로 동작
aaa123

[root@Server-A ~]# echo "aaa123" | grep '[0-9]\+'		# BRE: +를 쓰려면 이스케이프 필요
aaa123
```

**정리**: 글롭과 정규식은 `*`, `?` 같은 기호가 겹치지만 동작 방식이 근본적으로 다르다 — 글롭의 `*`는 "모든 문자열", 정규식의 `*`는 "직전 문자의 반복"이고, 글롭은 암묵적으로 전체 문자열과 매치되어야 하지만 정규식은 부분 일치도 허용한다. `case`/`[[ == ]]`/파일명 확장에는 글롭을, `[[ =~ ]]`/`grep -E`에는 정규식을 쓴다는 것을 명확히 구분해야 한다.

## POSIX 대괄호 표현식의 로케일 민감성

- `[a-zA-Z]`처럼 문자 범위를 직접 나열하는 방식과 `[[:alpha:]]`처럼 POSIX 문자 클래스를 쓰는 방식은 평소에는 같아 보이지만, **로케일(locale) 설정에 따라 결과가 달라질 수 있다**는 차이가 있다.
- `[a-z]` 같은 범위 표현은 시스템의 문자 정렬 순서(collation order)에 의존하는데, 일부 로케일에서는 대문자와 소문자가 섞인 순서로 정렬되어 있어 `[a-z]`가 예상치 못한 문자(숫자나 대문자 일부)까지 포함할 수 있다. 반면 `[[:lower:]]`는 로케일과 무관하게 "소문자"라는 의미 자체로 판단하므로 더 예측 가능하다.

```bash
[root@Server-A ~]# echo $LANG
ko_KR.UTF-8

[root@Server-A ~]# [[ "K" == [a-z] ]] && echo "매치" || echo "불일치"
불일치					# C 로케일 기준으로는 예상대로 대문자 K는 [a-z]에 안 걸림

[root@Server-A ~]# LC_ALL=C [[ "K" == [a-z] ]] && echo "매치" || echo "불일치"
불일치

[root@Server-A ~]# [[ "K" == [[:lower:]] ]] && echo "매치" || echo "불일치"
불일치					# [[:lower:]]는 로케일에 상관없이 "소문자" 의미 그대로 판단
```

- 실무에서는 스크립트의 동작을 항상 일관되게 유지하기 위해, 로케일에 영향받는 스크립트 맨 앞에서 `LC_ALL=C`(또는 `LC_COLLATE=C`)로 고정해두는 경우가 많다. 여러 서버에 배포되는 스크립트가 서버마다 다른 로케일 설정 때문에 다르게 동작하는 사고를 막기 위해서다.

```bash
[root@Server-A ~]# vi ./script/locale_safe.sh
#!/bin/bash
export LC_ALL=C		# 로케일 차이로 패턴 매칭 결과가 달라지는 것을 방지

value="Report"
if [[ "$value" == [A-Z]* ]]; then
    echo "대문자로 시작합니다."
fi

:wq
```

**정리**: `[a-z]`처럼 문자 범위를 직접 나열하는 방식은 로케일의 정렬 순서에 영향을 받을 수 있어 서버마다 결과가 달라질 위험이 있는 반면, `[[:alpha:]]` 같은 POSIX 문자 클래스는 로케일과 무관하게 의미 그대로 동작한다. 여러 환경에 배포되는 스크립트라면 `LC_ALL=C`로 로케일을 고정하거나 POSIX 문자 클래스를 우선 사용하는 것이 안전하다.

## 실습 예제 (EX1~EX5)

**EX1. 파일 확장자별로 case 패턴 매칭해서 분류하는 스크립트**

요구사항 : 디렉터리 안의 파일들을 순회하면서 확장자별로 이미지/문서/압축/기타로 분류해 출력한다.

```bash
[root@Server-A ~]# vi ./script/classify_files.sh
#!/bin/bash
for f in /data/files/*; do
    case "$f" in
        *.jpg | *.png | *.gif)
            echo "$f : 이미지 파일"
            ;;
        *.doc | *.docx | *.pdf)
            echo "$f : 문서 파일"
            ;;
        *.zip | *.tar.gz | *.tgz)
            echo "$f : 압축 파일"
            ;;
        *)
            echo "$f : 기타 파일"
            ;;
    esac
done

:wq


[root@Server-A ~]# chmod +x ./script/classify_files.sh
[root@Server-A ~]# ls /data/files/
photo.jpg  report.pdf  backup.tar.gz  notes.txt

[root@Server-A ~]# ./script/classify_files.sh
/data/files/photo.jpg : 이미지 파일
/data/files/report.pdf : 문서 파일
/data/files/backup.tar.gz : 압축 파일
/data/files/notes.txt : 기타 파일
```

**EX2. extglob으로 특정 패턴의 파일만 일괄 삭제하는 스크립트**

요구사항 : `.tmp`, `.bak`로 끝나는 파일만 골라 삭제하고, 그 외 파일은 건드리지 않는 스크립트를 작성한다.

```bash
[root@Server-A ~]# shopt -s extglob

[root@Server-A ~]# vi ./script/clean_tmp.sh
#!/bin/bash
shopt -s extglob

target_dir="/tmp/work"
echo "삭제 전 : $(ls $target_dir)"

rm -f "$target_dir"/*.@(tmp|bak)

echo "삭제 후 : $(ls $target_dir)"

:wq


[root@Server-A ~]# chmod +x ./script/clean_tmp.sh
[root@Server-A ~]# ls /tmp/work
a.log  b.tmp  c.bak  d.conf

[root@Server-A ~]# ./script/clean_tmp.sh
삭제 전 : a.log b.bak b.tmp d.conf
삭제 후 : a.log d.conf
```

**EX3. ${var//패턴/대체}로 로그 문자열 마스킹하는 스크립트**

요구사항 : 로그 문자열 안의 숫자로만 이루어진 부분(예: 주민번호, 카드번호 흉내)을 `*`로 마스킹한다.

```bash
[root@Server-A ~]# shopt -s extglob

[root@Server-A ~]# vi ./script/mask_log.sh
#!/bin/bash
shopt -s extglob

line="사용자 카드번호 1234567812345678 결제 완료"
masked="${line//+([0-9])/****************}"

echo "원본 : $line"
echo "마스킹 : $masked"

:wq


[root@Server-A ~]# chmod +x ./script/mask_log.sh
[root@Server-A ~]# ./script/mask_log.sh
원본 : 사용자 카드번호 1234567812345678 결제 완료
마스킹 : 사용자 카드번호 **************** 결제 완료
```

**EX4. 글롭과 정규식으로 동일한 파일명을 각각 검사해 차이 확인**

요구사항 : 같은 파일명을 `[[ == ]]`(글롭)과 `[[ =~ ]]`(정규식)로 각각 검사해 판정 방식의 차이를 직접 비교한다.

```bash
[root@Server-A ~]# vi ./script/glob_vs_regex.sh
#!/bin/bash
name="access.log.2026"

[[ "$name" == *.log* ]] && echo "글롭 매치 : 예" || echo "글롭 매치 : 아니오"
[[ "$name" =~ \.log ]] && echo "정규식 매치 : 예" || echo "정규식 매치 : 아니오"
[[ "$name" == access.log ]] && echo "글롭 전체일치 : 예" || echo "글롭 전체일치 : 아니오"
[[ "$name" =~ access.log ]] && echo "정규식 부분일치 : 예" || echo "정규식 부분일치 : 아니오"

:wq


[root@Server-A ~]# chmod +x ./script/glob_vs_regex.sh
[root@Server-A ~]# ./script/glob_vs_regex.sh
글롭 매치 : 예
정규식 매치 : 예
글롭 전체일치 : 아니오
정규식 부분일치 : 예
```

**EX5. POSIX 문자 클래스로 사번 형식을 검증하는 스크립트**

요구사항 : "대문자 1개 + 숫자 6개" 형식의 사번인지 `[[:upper:]]`, `[[:digit:]]`로 검증한다.

```bash
[root@Server-A ~]# vi ./script/id_format.sh
#!/bin/bash
read -p "사번을 입력하세요 : " id

if [[ "$id" == [[:upper:]][[:digit:]][[:digit:]][[:digit:]][[:digit:]][[:digit:]][[:digit:]] ]]; then
    echo "형식이 올바른 사번입니다."
else
    echo "형식이 올바르지 않습니다."
fi

:wq


[root@Server-A ~]# chmod +x ./script/id_format.sh
[root@Server-A ~]# ./script/id_format.sh
사번을 입력하세요 : A123456
형식이 올바른 사번입니다.

[root@Server-A ~]# ./script/id_format.sh
사번을 입력하세요 : a12345
형식이 올바르지 않습니다.
```

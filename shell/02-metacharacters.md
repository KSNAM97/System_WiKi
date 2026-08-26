# Shell Script 메타문자 (Metacharacters)

## 목차

1. [글롭(glob)](#1-글롭glob)
2. [중괄호 확장 (brace expansion)](#2-중괄호-확장-brace-expansion)
3. [변수, 치환 관련 메타 문자](#3-변수-치환-관련-메타-문자)
4. [Bash 산술 연산](#4-bash-산술-연산)
5. [리다이렉션(Redirection)](#5-리다이렉션redirection)
6. [파이프(Pipe)](#6-파이프pipe)
7. [명령 연결 / 제어 메타 문자](#7-명령-연결--제어-메타-문자)
8. [틸드( ~ ) 의 개념과 동작 원리](#8-틸드--의-개념과-동작-원리)

## Metacharacters (메타문자)

**메타문자(Metacharacters)**는 쉘에서 일반 문자와 다르게 특별한 의미로 해석되는 문자이다. 파일 목록을 한 번에 지정하거나(glob), 명령어의 출력을 파일로 저장·연결하거나(리다이렉션, 파이프), 명령의 성공 여부에 따라 다음 동작을 결정하는(`&&`, `||`) 등 쉘 스크립트 작성 전반에서 폭넓게 사용된다.

- 메타문자는 파일명을 검색하거나, 변수를 사용하거나, 명령어의 출력을 저장하고, 여러 명령어를 연결할 때 사용한다.
- 쉘은 명령어를 실행하기 전에 메타문자를 먼저 해석한 후 실제 명령어로 확장하거나 처리한다.

예) `ls *.txt`

- 위 명령어에서 `*`는 일반 문자가 아니라 모든 문자열을 의미하는 메타문자이다.
  따라서 현재 디렉터리에서 `.txt`로 끝나는 모든 파일을 출력한다.
- 이 때 특별한 기능을 가진 문자들이 바로 Metacharacters이다.
- 메타 문자는 대부분 작은따옴표(`'`), 큰따옴표(`"`), 백슬래시(`\`) 없이 썼을 때만 특별한 의미를 갖는다.
- 작은따옴표(`'`), 큰따옴표(`"`), 백슬래시(`\`) 로 의미를 없애고 그냥 문자로 만들 수 있다.
- 셸 스크립트, 리눅스 명령어, 패턴 매칭, 파일 검색, 리다이렉션에서 매우 자주 사용된다.

### Metacharacters 종류

1) **글롭(glob)** / 패턴 매칭용
   `*`, `?`, `[]`, `[! ]`, `[a-z]` 등

2) **중괄호 확장(Brace Expansion)**
   `{a,b}`, `{1..5}`, `{a..z}` 등

3) **변수, 치환 관련**
   `$`, `${}`, `$( )`, `` ` ` ``, `$(( ))`, `~`

4) **리다이렉션 / 파이프**
   `>`, `>>`, `<`, `<<`, `<<<`, `|`, `2>`, `&>`, `2>&1` 등

5) **명령 연결 / 제어**
   `;`, `&&`, `||`, `&`, `()`, `{}`, `\`

6) **주석**
   `#`

## 1) 글롭(glob)

### 패턴 매칭 메타 문자

#### `*` (asterisk)

- 의미: 0글자 이상 아무 문자나 모두 매치
- 주로 파일 이름 패턴에서 사용
- EX) `rm -rf /backup/a*`

#### `?` (question mark)

- 정확히 한 글자만 아무거나 매치
- EX) `file?.txt` = `file1.txt`, `fileA.txt`, `filex.txt` 등

### 대괄호 `[]` (문자 집합, Character Class)

**대괄호**는 글롭(glob) 패턴에서 사용되는 메타 문자로, 대괄호 안에 있는 문자 중 하나와 매칭하는 기능을 가진다.

- 대괄호 하나가 문자 1개를 대표한다.

```
file[abc].txt
 : filea.txt, fileb.txt, filec.txt 와 매치 (대괄호 안에 있는 문자 중 하나가 들어가야 함)
```

- 대괄호는 "문자 집합"을 의미하며, 순서는 어떠해도 동일하게 처리된다.

**기본 형태**
- `[abc]`
- 문자 a, b, c 중 하나와 매치한다.
- `[]` 안의 문자는 순서와 상관없다.
- `[abc]`, `[cba]`, `[bca]` 모두 동일한 의미이다.

**범위 지정 (Range)**

대괄호 안에서 하이픈(`-`)을 사용하면 연속된 문자 범위를 지정할 수 있다.

```
[a-z]	: 소문자 a~z 중 하나
[A-Z] 	: 대문자 A~Z 중 하나
[0-9]	: 숫자 0~9 중 하나

file[0-9].txt
 : file0.txt ~ file9.txt 와 매치
```

- 범위 표현은 단순히 "연속된 문자의 집합"을 의미하며 문자 비교 순서와는 관계없다.

**여러 범위를 조합**

대괄호 안에 여러 범위 또는 여러 단일 문자를 함께 쓸 수 있다.

```
[0-9a-zA-Z]
 : 숫자 또는 영문자

[abc0-5]
 : a, b, c, 0, 1, 2, 3, 4, 5
```

**부정 패턴 (Negation)**

대괄호 안에서 첫 문자로 느낌표(`!`) 또는 캐럿(`^`)을 사용하면 "해당 문자 집합이 아닌 것"을 뜻한다.

- 두 방식 모두 동일하게 동작한다.

```
[!abc] 또는 [^abc]
 :  a, b, c 를 제외한 어떤 문자 1글자와 매치

[!0-9]
 : 숫자를 제외한 문자 1개

예시 파일 매칭:
ls file[^0-9].txt
 : filex.txt, fileA.txt 등 숫자가 아닌 문자 하나를 가진 파일만 매치
```

- 패턴에서 정확히 문자 1개만 매칭한다.
- 대괄호는 항상 "문자 1개" 전용이다.

```
file[abc].txt
 : a, b, c 중 하나만 매치함
 : fileac.txt (두 글자) 는 매치되지 않음

[0-9]
 : 숫자 하나

[0-9][0-9]
 : 두 자리 숫자
```

**와일드카드 `*` 과 비교**

대괄호는 정확히 1글자만 매치하는 반면 `*` 은 0글자 이상 모든 문자열을 매치한다.

```
file*.txt
 : file.txt, file1.txt, fileABC.txt 등 전부 매치

file[0-9].txt
 : file0.txt ~ file9.txt 만 매치
 : file123.txt 는 매치되지 않음
```

**물음표 `?` 와 `[]` 비교**
- `?` 는 "아무 문자 1개"
- `[]` 는 "지정된 문자 중 1개"

### 사전 준비

1) "2-2) Shell Script - Metacharacters - 실습 Pre-config" 파일을 복사 붙여넣기

```
[root@Server-A ~]# ls  -l
합계 8
-rw-------. 1 root root 1027  7월  2 12:55 anaconda-ks.cfg
-rwxr-xr-x  1 root root 3773  7월 22 12:41 meta_lab_setup.sh		# 실행 파일 확인
drwxr-xr-x. 2 root root    6  7월  2 14:46 공개
drwxr-xr-x. 2 root root    6  7월  2 14:46 다운로드
drwxr-xr-x. 2 root root    6  7월  2 14:46 문서
drwxr-xr-x. 2 root root    6  7월  2 14:46 바탕화면
drwxr-xr-x. 2 root root    6  7월  2 14:46 비디오
drwxr-xr-x. 2 root root    6  7월  2 14:46 사진
drwxr-xr-x. 2 root root    6  7월  2 14:46 서식
drwxr-xr-x. 2 root root    6  7월  2 14:46 음악
```

2) 스크립트파일을 실행

```
[root@Server-A ~]# ./meta_lab_setup.sh
기준 디렉터리: /root/meta_lab
[glob_lab] 테스트 파일 생성 중...
[glob_lab] 생성 완료.
[input_lab] 테스트 파일 생성 중...
[input_lab] 생성 완료.
[brace_lab] 테스트 파일 생성 중...
[brace_lab] 생성 완료.
[and_or_lab] 기본 구조 생성 완료.

==== 준비 완료 ====
글롭 연습  : /root/meta_lab/glob_lab
리다이렉션 : /root/meta_lab/input_lab
브레이스   : /root/meta_lab/brace_lab
AND/OR     : /root/meta_lab/and_or_lab



[root@Server-A ~]# ls -l
합계 8
-rw-------. 1 root root 1027  7월  2 12:55 anaconda-ks.cfg
drwxr-xr-x  6 root root    74  7월 22 12:46 meta_lab		# 디렉터리 생성 확인
-rwxr-xr-x  1 root root 3773  7월 22 12:41 meta_lab_setup.sh
drwxr-xr-x. 2 root root     6  7월   2 14:46 공개
drwxr-xr-x. 2 root root     6  7월   2 14:46 다운로드
drwxr-xr-x. 2 root root     6  7월   2 14:46 문서
drwxr-xr-x. 2 root root     6  7월   2 14:46 바탕화면
drwxr-xr-x. 2 root root     6  7월   2 14:46 비디오
drwxr-xr-x. 2 root root     6  7월   2 14:46 사진
drwxr-xr-x. 2 root root     6  7월   2 14:46 서식
drwxr-xr-x. 2 root root     6  7월   2 14:46 음악



[root@Server-A ~]# cd  ./meta_lab


[root@Server-A meta_lab]# ls  -l
합계 16
drwxr-xr-x 4 root root    72  7월 22 12:46 and_or_lab
drwxr-xr-x 2 root root 4096  7월 22 12:46 brace_lab
drwxr-xr-x 2 root root 8192  7월 22 12:46 glob_lab		# glob 실습용 디렉터리
drwxr-xr-x 2 root root    79  7월 22 12:46 input_lab



[root@Server-A meta_lab]# cd  ./glob_lab/		# glob 실습용 디렉터리로 이동



[root@Server-A glob_lab]# pwd
/root/meta_lab/glob_lab



[root@Server-A meta_lab]# ls  -l
```

EX1) filea.txt, fileb.txt, filec.txt만 출력하시오

```
[root@Server-A glob_lab]# ls  -l  ./filea.txt  ./fileb.txt  ./filec.txt
-rw-r--r-- 1 root root 0  7월 22 12:46 ./filea.txt
-rw-r--r-- 1 root root 0  7월 22 12:46 ./fileb.txt
-rw-r--r-- 1 root root 0  7월 22 12:46 ./filec.txt



[root@Server-A glob_lab]# ls  -l  file[abc].txt
-rw-r--r-- 1 root root 0  7월 22 12:46 filea.txt
-rw-r--r-- 1 root root 0  7월 22 12:46 fileb.txt
-rw-r--r-- 1 root root 0  7월 22 12:46 filec.txt
```

EX2) file0.txt ~ file9.txt만 출력하시오

```
[root@Server-A glob_lab]# ls  -l  file[0-9].txt
-rw-r--r-- 1 root root 0  7월 22 12:46 file0.txt
-rw-r--r-- 1 root root 0  7월 22 12:46 file1.txt
-rw-r--r-- 1 root root 0  7월 22 12:46 file2.txt
-rw-r--r-- 1 root root 0  7월 22 12:46 file3.txt
-rw-r--r-- 1 root root 0  7월 22 12:46 file4.txt
-rw-r--r-- 1 root root 0  7월 22 12:46 file5.txt
-rw-r--r-- 1 root root 0  7월 22 12:46 file6.txt
-rw-r--r-- 1 root root 0  7월 22 12:46 file7.txt
-rw-r--r-- 1 root root 0  7월 22 12:46 file8.txt
-rw-r--r-- 1 root root 0  7월 22 12:46 file9.txt
```

EX3) backupA, backupB, backupC만 출력하시오

```
[root@Server-A glob_lab]# ls  -l  backup[ABC]
-rw-r--r-- 1 root root 0  7월 22 12:46 backupA
-rw-r--r-- 1 root root 0  7월 22 12:46 backupB
-rw-r--r-- 1 root root 0  7월 22 12:46 backupC


[root@Server-A glob_lab]# ls  -l  backup?
-rw-r--r-- 1 root root 0  7월 22 12:57 backupA
-rw-r--r-- 1 root root 0  7월 22 12:57 backupB
-rw-r--r-- 1 root root 0  7월 22 12:57 backupC
-rw-r--r-- 1 root root 0  7월 22 12:57 backupD
-rw-r--r-- 1 root root 0  7월 22 12:57 backupE


[root@Server-A glob_lab]# ls  -l  backup*
-rw-r--r-- 1 root root 0  7월 22 12:58 backupA
-rw-r--r-- 1 root root 0  7월 22 12:58 backupA11
-rw-r--r-- 1 root root 0  7월 22 12:58 backupAa
-rw-r--r-- 1 root root 0  7월 22 12:58 backupB
-rw-r--r-- 1 root root 0  7월 22 12:58 backupB22
-rw-r--r-- 1 root root 0  7월 22 12:58 backupBb
-rw-r--r-- 1 root root 0  7월 22 12:58 backupC
-rw-r--r-- 1 root root 0  7월 22 12:58 backupC33
-rw-r--r-- 1 root root 0  7월 22 12:58 backupCc
-rw-r--r-- 1 root root 0  7월 22 12:58 backupD
-rw-r--r-- 1 root root 0  7월 22 12:58 backupD44
-rw-r--r-- 1 root root 0  7월 22 12:58 backupDd
-rw-r--r-- 1 root root 0  7월 22 12:58 backupE
-rw-r--r-- 1 root root 0  7월 22 12:58 backupE55
-rw-r--r-- 1 root root 0  7월 22 12:58 backupEe
```

EX4) 확장자가 .png 또는 .jpg인 파일만 출력하시오

```
[root@Server-A glob_lab]# ls  -l  *.[pj][np]g
-rw-r--r-- 1 root root 0  7월 22 12:46 icon.png
-rw-r--r-- 1 root root 0  7월 22 12:46 photo.jpg
-rw-r--r-- 1 root root 0  7월 22 12:46 picture.png
```

EX5-1) a~c 중 하나로 시작하는 파일만 출력하시오

```
[root@Server-A glob_lab]# ls  [a-c]*
a1file 	aqz      	backupA    	backupC    	backupE    	bpb		cpc
a_start  	axfile   	backupA11  	backupC33  	backupE55  	c9hello
aa.txt   	azc      	backupAa   	backupCc   	backupEe   	c_start
ab.txt   	b3data  	backupB     	backupD    	bafilE     		cdworld
ac.txt   	b_start  	backupB22  	backupD44  	bb.txt     		config.cfg
apa      	ba.txt   	backupBb   	backupDd		bc.txt     		config3
```

EX5-2) a~c 중 하나로 시작하고 파일 이름이 6글자인 파일만 출력하시오

```
[root@Server-A glob_lab]# ls  [a-c]?????
a1file  aa.txt  ab.txt  ac.txt  axfile  b3data  ba.txt  bafilE  bb.txt  bc.txt
```

EX5-3) 다음 조건을 모두 만족하는 파일만 출력하시오.
- 파일 이름의 첫 번째 문자는 a부터 c 중 하나여야 한다.
- 확장자는 .txt여야 한다.
- 점(.)을 포함한 전체 파일 이름의 길이는 6글자여야 한다.

```
[root@Server-A glob_lab]# ls  [a-c]?.txt
aa.txt  ab.txt  ac.txt  ba.txt  bb.txt  bc.txt
```

EX6) 숫자 두 자리(00~99) 이름을 가진 파일만 출력하시오
- file00, file01, file02, …, file99 형태(두 자리 숫자)의 파일만 매칭되도록 출력하시오.

```
[root@Server-A glob_lab]# ls  -l  file[0-9][0-9]
-rw-r--r-- 1 root root 0  7월 22 12:46 file00
-rw-r--r-- 1 root root 0  7월 22 12:46 file01
-rw-r--r-- 1 root root 0  7월 22 12:46 file02
-rw-r--r-- 1 root root 0  7월 22 12:46 file03
    ~~~~~~~ 중간 생략 ~~~~~~~
-rw-r--r-- 1 root root 0  7월 22 12:46 file94
-rw-r--r-- 1 root root 0  7월 22 12:46 file95
-rw-r--r-- 1 root root 0  7월 22 12:46 file96
-rw-r--r-- 1 root root 0  7월 22 12:46 file97
-rw-r--r-- 1 root root 0  7월 22 12:46 file98
-rw-r--r-- 1 root root 0  7월 22 12:46 file99



[root@Server-A glob_lab]# ls  -l  file[09][09]
-rw-r--r-- 1 root root 0  7월 22 12:46 file00
-rw-r--r-- 1 root root 0  7월 22 12:46 file09
-rw-r--r-- 1 root root 0  7월 22 12:46 file90
-rw-r--r-- 1 root root 0  7월 22 12:46 file99
```

EX7) 파일 이름의 두 번째 글자가 a, b, 또는 c인 파일만 출력하시오
- ab.txt, ac.txt, ax.txt, ba.txt, ca.txt...등 두 번째 글자가 a/b/c 중 하나일 때만 출력하시오.

```
[root@Server-A glob_lab]# ls  ?[abc]*
Abc.txt	backupA  		backupBb   	backupD44  	bafilE  	fb.txt    	   mc.txt
aa.txt   	backupA11	backupC    	backupDd   	bb.txt  	fc.txt    	   sample.log
ab.txt   	backupAa   	backupC33  	backupE    	bc.txt  	icon.png
ac.txt   	backupB    	backupCc   	backupE55	data0   	ma.txt
ba.txt   	backupB22  	backupD    	backupEe   	fa.txt  	mb.txt
```

EX8) 대문자로 시작하는 파일만 출력하시오 (A~Z)
- Apple.txt, Zebra.log, MyFile.conf 등이 있을 때 첫 글자가 대문자인 파일만 출력하시오.

```
[root@Server-A glob_lab]# ls  -l  [A-Z]*
-rw-r--r-- 1 root root 0  7월 22 12:46 Abc.txt
-rw-r--r-- 1 root root 0  7월 22 12:46 Apple.txt
-rw-r--r-- 1 root root 0  7월 22 12:46 MyFile.conf
-rw-r--r-- 1 root root 0  7월 22 12:46 Zebra.log
-rw-r--r-- 1 root root 0  7월 22 12:46 Zzz.txt
```

EX9) 확장자가 .log 또는 .cfg 또는 .txt 인 파일만 출력하시오
- sample.log, config.cfg, note.txt 등만 매칭되도록 하시오.

```
[root@Server-A glob_lab]# ls  *.[lct][ofx][gt]
Abc.txt  	ab.txt  	config.cfg	  file1.txt	  file6.txt	  fileb.txt	  md.txt
Apple.txt	ac.txt  	fa.txt      	  file2.txt	  file7.txt	  filec.txt	  me.txt
Zebra.log	ba.txt  	fb.txt      	  file3.txt	  file8.txt	  ma.txt 	  mf.txt
Zzz.txt    	bb.txt  	fc.txt      	  file4.txt	  file9.txt	  mb.txt 	  note.txt
aa.txt     	bc.txt  	file0.txt	  file5.txt	  filea.txt	  mc.txt	  sample.log
```

EX10) 이름 끝이 숫자인 파일만 출력하시오
- test1, config3, item9 등 뒤에 숫자 1개가 있는 파일만 출력하시오.

```
[root@Server-A glob_lab]# ls *[a-zA-Z][0-9]
config3  data0  file1  file7  item9  log5  test1
```

EX11) 이름 두 번째 글자가 숫자가 아닌 파일만 출력하시오
- a1file 	--> 제외
- axfile 	--> 포함
- b3data 	--> 제외
- bafile 	--> 포함

```
[root@Server-A glob_lab]# ls  ?[^0-9]*
Abc.txt  	eqe        	file21     	file48     	file73     	fileA
Apple.txt	fa.txt     	file22     	file49     	file74     	fileB
~~~~~~~~~ 중간 생략 ~~~~~~~~~
data0   	file2.txt	file46     	file71     	file98
dqd      	file20     	file47     	file72     	file99
```

EX12) 이름이 fileA, fileB, fileC가 아니라 다른 file*만 출력하시오
- fileA, fileB, fileC 를 제외한
- fileD, fileX, file1, file.txt 등만 출력하시오.

```
[root@Server-A glob_lab]# ls  file[^ABC]*
file0.txt  file14     file3.txt  file45     file60     file75     file90
file00     file15     file30     file46     file61     file76     file91
file01     file16     file31     file47     file62     file77     file92
file02     file17     file32     file48     file63     file78     file93
file03     file18     file33     file49     file64     file79     file94
file04     file19     file34     file5.txt  file65     file8.txt  file95
file05     file2.txt  file35     file50     file66     file80     file96
file06     file20     file36     file51     file67     file81     file97
file07     file21     file37     file52     file68     file82     file98
file08     file22     file38     file53     file69     file83     file99
file09     file23     file39     file54     file7      file84     fileD
file1      file24     file4.txt  file55     file7.txt  file85     fileX
file1.txt  file25     file40     file56     file70     file86     fileZanything
file10     file26     file41     file57     file71     file87     filea.txt
file11     file27     file42     file58     file72     file88     fileb.txt
file12     file28     file43     file59     file73     file89     filec.txt
file13     file29     file44     file6.txt  file74     file9.txt
```

EX13) 확장자가 .txt 이면서 두 번째 글자가 a~f 에 해당하는 파일만 출력하시오
- fa.txt 	--> X (첫 글자 f, 두 번째 a)
- ma.txt 	--> O (m + a + .txt)
- bb.txt 	--> X (두 번째 b는 a~f 이지만 파일명 조건 확인 필요)

```
[root@Server-A glob_lab]# ls   ?[a-f]*.txt
Abc.txt  ab.txt  ba.txt  bc.txt  fb.txt  ma.txt  mc.txt  me.txt
aa.txt   ac.txt  bb.txt  fa.txt  fc.txt  mb.txt  md.txt  mf.txt
```

EX14) 점(.)으로 시작하는 숨김파일 중, 두 번째 문자가 숫자인 파일만 출력하시오
- .1file 		--> 출력
- .9test 		--> 출력
- .ahello 	--> 제외
- .xx 		--> 제외

```
[root@Server-A glob_lab]# ls  .[0-9]*
.1file  .9test
```

EX15) 세 글자 이름의 파일 중 가운데 문자가 p 또는 q 인 것만 출력하시오

문제 예시:
- apa --> 출력
- bqb --> 출력
- aqz --> 출력
- azc --> 제외 (가운데 글자가 p/q 아님)

```
[root@Server-A glob_lab]# ls  -l  ?[pq]?
-rw-r--r-- 1 root root 0  7월 22 12:46 apa
-rw-r--r-- 1 root root 0  7월 22 12:46 aqz
-rw-r--r-- 1 root root 0  7월 22 12:46 bpb
-rw-r--r-- 1 root root 0  7월 22 12:46 cpc
-rw-r--r-- 1 root root 0  7월 22 12:46 dqd
-rw-r--r-- 1 root root 0  7월 22 12:46 eqe
```

**정리**: 글롭(glob)은 `*`, `?`, `[]` 조합으로 파일 이름 패턴을 매칭하는 기능이며, `*`는 0글자 이상, `?`는 정확히 1글자, `[]`는 문자 집합(범위·부정 포함) 1글자를 의미한다는 점을 구분해서 기억해야 한다.

## 2) 중괄호 확장 (brace expansion)

### 개념

- **중괄호 확장**은 글롭(glob)과 다르게, 문자열을 여러 개로 자동 생성하여 확장하는 기능이다.
- 셸이 중괄호 안의 내용을 펼쳐서 여러 개의 단어로 확장한다.
- 중괄호 확장은 파일의 존재 여부와 상관없이 단순히 문자열 생성 기능을 한다.

```
[root@Server-A ~]# echo file{1,3,5}.txt
file1.txt   file3.txt   file5.txt
```

- 중괄호는 쉘이 문자열을 조립하는 기능이지 파일 매칭(globbing)이 아니다.

### 기본 형태

`{a,b,c}`

- 중괄호 안의 항목을 각각 하나의 값으로 확장한다.

```
[root@Server-A ~]# echo test{A,B,C}.log
testA.log   testB.log   testC.log
```

- 확장은 쉘이 만드는 문자열이지, 파일이 없어도 오류가 아니다.

### 숫자 또는 문자 범위 확장

`{1..5}`, `{a..f}`

- 연속된 범위를 모두 생성한다.

```
[root@Server-A ~]# echo  Soldesk{1..5}
Soldesk1   Soldesk2   Soldesk3   Soldesk4   Soldesk5


[root@Server-A ~]# echo  Soldesk{01..10}
Soldesk01   Soldesk02   Soldesk03   Soldesk04   Soldesk05   Soldesk06   Soldesk07   Soldesk08   Soldesk09   Soldesk10


[root@Server-A ~]# echo  Soldesk{A..D}
SoldeskA   SoldeskB   SoldeskC   SoldeskD


[root@Server-A ~]# echo  Soldesk{a..f}
Soldeska   Soldeskb   Soldeskc   Soldeskd   Soldeske   Soldeskf
```

### 여러 중괄호 조합도 가능

중괄호는 여러 번 중첩하여 조합할 수 있다.

```
[root@Server-A ~]# echo  {A,B}{1,2}.txt
A1.txt   A2.txt   B1.txt   B2.txt


[root@Server-A ~]# echo  {A,B}{1,2,3}.txt
A1.txt  A2.txt   A3.txt   B1.txt   B2.txt   B3.txt
```

- 확장 순서: 첫 번째 중괄호를 펼치고, 다음 중괄호를 조합한다.

### 경로 생성에도 많이 사용된다

중괄호 확장은 디렉터리를 한 번에 여러 개 만드는 데 매우 편리하다.

```
mkdir /data/{log,backup,temp}
 /data/log
 /data/backup
 /data/temp
```

- 3개가 한 번에 생성된다.
- 여러 하위 디렉터리 조합도 가능하다.

```
mkdir  -p  project/{src,bin,conf}/{dev,prod}

[root@Server-A glob_lab]# echo project/{src,bin,conf}/{dev,prod}
project/src/dev 
project/src/prod 
project/bin/dev 
project/bin/prod 
project/conf/dev 
project/conf/prod
```

### 공백이 있으면 확장이 안 된다

중괄호는 다음 규칙이 있다.
- 중괄호 내부 값 앞뒤에 공백이 있으면 확장 불가
- 콤마로 구분해야 한다.

```
[root@Server-A ~]# echo  File{A,B,C}		# 공백이 없으면 확장
FileA FileB FileC


[root@Server-A ~]# echo  File{A, B, C}	# 공백이 있으면 그대로 사용
File{A, B, C}
```

### Brace Expansion은 Quote(따옴표) 안에서도 동작한다

글롭과 달리, 중괄호 확장은 작은따옴표 안이 아닌 이상 대부분 작동한다.

예1) 따옴표 없이 사용

```
[root@Server-A ~]# echo file{1,2,3}.txt
file1.txt file2.txt file3.txt
```

예2) 중괄호 앞뒤만 큰따옴표로 사용

```
[root@Server-A ~]# echo "file"{1,2,3}".txt"
file1.txt file2.txt file3.txt
```

예3) 중괄호 앞뒤만 작은따옴표로 사용

```
[root@Server-A ~]# echo 'file'{1,2,3}'.txt'
file1.txt file2.txt file3.txt
```

- 위의 예제들은 모두 `{1,2,3}` 부분이 따옴표 밖에 있으므로 중괄호 확장이 수행된다.
- 중괄호 표현 자체를 큰따옴표 또는 작은따옴표 안에 넣으면 중괄호 확장이 수행되지 않는다.

예4) 전체를 큰따옴표로 감싼 경우

```
[root@Server-A ~]# echo "file{1,2,3}.txt"
file{1,2,3}.txt
```

예5) 전체를 작은따옴표로 감싼 경우

```
[root@Server-A ~]# echo 'file{1,2,3}.txt'
file{1,2,3}.txt
```

- 큰따옴표와 작은따옴표 모두 중괄호 확장을 막는다.

### 실습 문제

EX1-1) file1.txt file3.txt file7.txt 를 한 번에 출력하시오

```
[root@Server-A ~]# echo file{1,3,7}.txt

file1.txt file3.txt file7.txt
```

EX1-2) file1.txt file3.txt file7.txt 파일을 한 번에 생성하시오

```
[root@Server-A glob_lab]# touch ~/file{1,3,7}.txt


[root@Server-A glob_lab]# ls  -l ~/file*
-rw-r--r-- 1 root root 0  7월 22 15:55 /root/file1.txt
-rw-r--r-- 1 root root 0  7월 22 15:55 /root/file3.txt
-rw-r--r-- 1 root root 0  7월 22 15:55 /root/file7.txt
```

EX2-1) file1.txt ~ file5.txt 를 한 번에 출력하시오

```
[root@Server-A ~]# echo file{1..5}.txt
file1.txt  file2.txt  file3.txt  file4.txt  file5.txt
```

EX2-1) file11.txt ~ file15.txt 파일을 한 번에 생성하시오

```
[root@Server-A glob_lab]# touch ~/file{11..15}.txt


[root@Server-A glob_lab]# ls  -l ~/file1*
-rw-r--r-- 1 root root 0  7월 22 15:55 /root/file1.txt
-rw-r--r-- 1 root root 0  7월 22 15:56 /root/file11.txt
-rw-r--r-- 1 root root 0  7월 22 15:56 /root/file12.txt
-rw-r--r-- 1 root root 0  7월 22 15:56 /root/file13.txt
-rw-r--r-- 1 root root 0  7월 22 15:56 /root/file14.txt
-rw-r--r-- 1 root root 0  7월 22 15:56 /root/file15.txt
```

EX3) /data/log /data/backup /data/temp 디렉터리를 한 번에 생성하시오

정답: `mkdir /data/{log,backup,temp}`

```
[root@Server-A glob_lab]# ls  -lR  /data
/data:
합계 0
drwxr-xr-x 2 root root 6  7월 22 12:46 backup
drwxr-xr-x 2 root root 6  7월 22 12:46 log
drwxr-xr-x 2 root root 6  7월 22 12:46 temp

/data/backup:
합계 0

/data/log:
합계 0

/data/temp:
합계 0
```

EX4) user01 ~ user10 이름을 화면에 출력하시오

```
[root@Server-A ~]# echo user{01..10}

user01 user02 user03 user04 user05 user06 user07 user08 user09 user10
```

EX5) /bin, /sbin, /usr/bin 경로를 한 번에 표시하시오

```
[root@Server-A ~]# echo  /{bin,sbin,usr/bin}
/bin /sbin /usr/bin


[root@Server-A ~]# ls  /{bin,sbin,usr/bin}
```

EX6) testA, testB, testC 파일 이름을 한 번에 출력하시오

```
[root@Server-A ~]# echo test{A,B,C}
testA  testB  testC
```

EX7) 연도 2020, 2021, 2022를 한 번에 출력하시오

```
[root@Server-A ~]# echo 202{0..2}
2020 2021 2022
```

EX8) app/dev, app/prod 두 개의 경로를 한 번에 출력하시오

```
[root@Server-A ~]# echo app/{dev,prod}
app/dev  app/prod
```

EX9) file-A1.txt, file-A2.txt, file-B1.txt, file-B2.txt 네 개를 출력하시오

```
[root@Server-A ~]# echo file-{A,B}{1,2}.txt
file-A1.txt  file-A2.txt  file-B1.txt  file-B2.txt
```

EX10) img001.png ~ img010.png 파일 이름을 한 번에 출력하시오

```
[root@Server-A ~]# echo img{001..010}.png
img001.png  img002.png  img003.png  img004.png  img005.png  img006.png  img007.png  img008.png  img009.png  img010.png
```

EX11) conf-dev.yml, conf-test.yml, conf-prod.yml 이름을 출력하시오

```
[root@Server-A ~]# echo conf-{dev,test,prod}.yml
conf-dev.yml  conf-test.yml  conf-prod.yml
```

EX12) A-1, A-2, A-3, B-1, B-2, B-3 을 한 번에 출력하시오

```
[root@Server-A ~]# echo {A,B}-{1..3}
A-1  A-2  A-3  B-1  B-2  B-3
```

EX13) log_2023_01 ~ log_2023_12 까지 생성되는 이름을 출력하시오

```
[root@Server-A ~]# echo log_2023_{01..12}
log_2023_01  log_2023_02 log_2023_03  log_2023_04  log_2023_05  log_2023_06 
log_2023_07  log_2023_08 log_2023_09  log_2023_10  log_2023_11  log_2023_12
```

EX14) web-dev-1, web-dev-2, web-prod-1, web-prod-2 를 한 번에 출력하시오

```
[root@Server-A ~]# echo web-{dev,prod}-{1,2}
web-dev-1  web-dev-2  web-prod-1  web-prod-2
```

EX15) tmp-a.conf, tmp-b.conf, tmp-c.conf 를 출력하시오

```
[root@Server-A ~]# echo tmp-{a,b,c}.conf
tmp-a.conf  tmp-b.conf  tmp-c.conf
```

**정리**: 중괄호 확장(brace expansion)은 파일 존재 여부와 무관하게 셸이 문자열 자체를 조합·생성하는 기능이며, 콤마 목록(`{a,b,c}`)과 범위(`{1..5}`)를 중첩해서 여러 디렉터리/파일 이름을 한 번에 만들 때 특히 유용하다.

## 3) 변수, 치환 관련 메타 문자

### `$` (변수 참조)

- `$`는 셸에서 변수의 값을 읽어오는 기능을 한다.
- 변수 이름 앞에 `$`를 붙이면, 변수명이 아니라 그 변수에 저장된 실제 값을 의미하게 된다.
- `$`는 어디까지나 값을 가져오는 것이며, 변수 선언 시에는 절대 사용하지 않는다.

### 큰따옴표와 작은따옴표의 차이

#### 큰따옴표

- 큰따옴표(`""`) 안에서는 변수 확장 O
- 큰따옴표 안에서는 문자열의 공백을 그대로 유지하면서 변수 확장을 수행한다.
  따라서 `$변수명`은 변수에 저장된 실제 값으로 바뀐다.
- 큰따옴표는 변수 확장을 막는 것이 아니라, 변수의 값을 하나의 문자열 안에서 안전하게 사용할 수 있도록 한다.

```
[root@Server-A ~]# name=Park

[root@Server-A ~]# echo "Hello $name"
Hello Park
```

#### 작은따옴표

- 작은따옴표(`''`) 안에서는 변수 확장 X
- 작은따옴표 안에서는 모든 문자를 일반 문자로 처리한다.
  따라서 `$`, `*`, `?`, `$( )` 등의 메타문자가 특별한 기능을 수행하지 않는다.
- 변수 확장, 명령 치환, 산술 확장 등이 수행되지 않고 입력한 문자열이 그대로 출력된다.

```
[root@Server-A ~]# name="Park"

[root@Server-A ~]# echo 'Hello $name'
Hello $name
```

- `Hello $name` 그대로 출력됨
- 이 차이는 쉘 스크립트에서 매우 중요하다.

#### 왜 처리 방식이 다른가?

- 큰따옴표는 문자열을 하나로 묶으면서도 변수나 명령의 실행 결과를 문자열 안에 포함하기 위해 사용한다.
- 즉, 문자열의 형태는 유지하지만 필요한 확장은 허용한다.

**큰따옴표**
- 문자열을 하나로 묶음
- 변수 확장 허용	: `$변수명`
- 명령 치환 허용	: `$(명령어)`
- 산술 확장 허용	: `$((산술식))`

- 작은따옴표는 문자열의 내용을 셸이 해석하지 못하도록 완전히 보호하기 위해 사용한다.
- 즉, 메타문자를 실행하거나 확장하지 않고 작성한 내용을 그대로 전달한다.

**작은따옴표**
- 문자열을 하나로 묶음
- 변수 확장 차단
- 명령 치환 차단
- 산술 확장 차단
- 대부분의 메타문자를 일반 문자로 처리

### `${변수}` 형태도 가능

변수명을 명확히 구분해야 할 때는 중괄호로 감싸는 형식을 사용한다.

```
[root@Server-A ~]# var=10

[root@Server-A ~]# echo "$varfile"		# varfile 변수가 없기때문에 아무것도 출력되지 않는다.

[root@Server-A ~]# echo "${var}file"		# 10file 출력
10file
```

- 즉, `${변수}`는 변수명과 뒤 문자 구분을 명확히 할 때 사용된다.

### 환경변수(Environment Variables) 참조

`$`는 일반 변수뿐 아니라 환경변수를 읽을 때도 동일하게 사용한다.

```
echo $USER
echo $HOME
echo $PATH
echo $SHELL
```

- 각각 로그인 사용자 이름, 홈 디렉터리, PATH 경로, 기본 쉘 등이 출력된다.
- 환경변수는 `export` 로 등록된 전역 변수처럼 동작한다.

### 변수, 치환 관련 메타문자 실습 문제

**사전 준비**

```
[root@Server-A ~]# name="Park"
[root@Server-A ~]# message="Hello Linux"
[root@Server-A ~]# var=10
[root@Server-A ~]# filename="report"
[root@Server-A ~]# count=5


[root@Server-A ~]# echo "$name"
Park

[root@Server-A ~]# echo "$message"
Hello Linux

[root@Server-A ~]# echo "$var"
10

[root@Server-A ~]# echo "$filename"
report

[root@Server-A ~]# echo "$count"
5
```

EX1) 변수 값 출력
- name 변수에 저장된 값을 출력하시오.

```
[root@Server-A ~]# echo $name
Park
```

EX2) 변수 선언과 참조 구분
- city 변수에 Seoul을 저장한 후 변수의 값을 출력하시오.

```
[root@Server-A ~]# city="Seoul"     	# 변수에 값 저장

[root@Server-A ~]# echo "$city"     	# 변수의 값 참조
Seoul
```

EX3) 큰따옴표 안에서 변수 확장
- Hello Park가 출력되도록 명령어를 작성하시오.

```
[root@Server-A ~]# echo "Hello $name"
Hello Park
```

EX4) 작은따옴표 안에서 변수 사용
- 다음 문자열을 변수 확장 없이 그대로 출력하시오.
- 문자열 : Hello $name

```
[root@Server-A ~]# echo 'Hello $name'
Hello $name
```

- 작은따옴표 안에서는 `$`를 포함한 메타문자를 일반 문자로 처리하기때문에 `$name`이 Park으로 바뀌지 않는다.

EX5) 공백이 포함된 변수 안전하게 출력
- message 변수에 저장된 Hello Linux를 하나의 문자열로 출력하시오.

```
[root@Server-A ~]# echo "$message"
Hello Linux
```

- 첫번째 인수 : Hello Linux

```
[root@Server-A ~]# echo $message
Hello Linux
```

- 첫번째 인수 : Hello
- 두번째 인수 : Linux

EX6) 변수명과 뒤 문자열 구분
- var 변수의 값 뒤에 file 문자열을 연결하여 다음과 같이 출력하시오.
- 출력 문자열 : 10file

```
[root@Server-A ~]# var=10

[root@Server-A ~]# echo "$varfile"		# varfile 변수가 없기때문에 아무것도 출력되지 않는다.

[root@Server-A ~]# echo "${var}file"		# 10file 출력
10file
```

EX9) `$varfile`과 `${var}file` 비교

```
[root@Server-A ~]# var=10

[root@Server-A ~]# echo "$varfile"		# varfile 변수가 없기때문에 아무것도 출력되지 않는다.

[root@Server-A ~]# echo "${var}file"		# 10file 출력
10file
```

- 즉, `${변수}`는 변수명과 뒤 문자 구분을 명확히 할 때 사용된다.

EX10) 작은따옴표와 큰따옴표 혼합
- name 변수의 값을 사용하여 다음과 같이 출력하시오.
- name="Park"
- 출력 내용 : $name변수의 실제 값은 Park입니다.

```
[root@Server-A ~]# echo '$name'"변수의 실제 값은 Park입니다."
$name변수의 실제 값은 Park입니다
```

### `$( )` (명령 치환, command substitution)

**명령 치환**은 명령의 실행 결과(출력값)를 문자열로 변환하여 그 자리에 삽입하는 기능이다.

- 형식 : `$(command)`
- 셸은 "명령을 먼저 실행"하고 그 출력 결과를 해당 위치에 넣어서 처리한다.

예) `echo "현재 시간: $(date)"`
- date 명령이 먼저 실행되고 출력값이 echo 안으로 들어간다.

- 명령의 결과를 변수에 넣거나, 명령의 결과를 문자열, 경로에 포함해야 할 때 반드시 필요하다.

```
[root@Server-A ~]# today=$(date +%F)
[root@Server-A ~]# echo "오늘 날짜는 $today 입니다."
 : 오늘 날짜는 2023-10-21 입니다.
```

**기본 형식**
- 옛날방식 (` 백틱`)	: `` `date` ``
- 현재 방식	: `$(명령어)`

```
[root@Server-A ~]# d=`date`

[root@Server-A ~]# echo $d
2026. 07. 22. (수) 16:53:51 KST


[root@Server-A ~]# today=$(date)

[root@Server-A ~]# echo $today
2026. 07. 22. (수) 16:54:16 KST
```

- 명령 치환 `$()` 안에는 다른 명령 치환 `$()`를 다시 넣을 수 있다.
- 이처럼 명령 치환 안에 또 다른 명령 치환을 사용하는 것을 중첩 명령 치환이라고 한다.
- 동작 순서는 안쪽 명령어부터 바깥쪽 명령어 순서이다.

예) `echo "현재 쉘 위치: $(basename $(pwd))"`

- 명령 치환은 출력을 가져오는 것이라는 점이 핵심이다.
- 명령 치환은 "stdout(표준 출력)"을 가져오는 기능이다.

```
files=$(ls /var/log)		# ls의 출력이 그대로 변수에 저장된다.
echo "$files"		# 확인
```

- 따옴표와 조합해서 사용 가능
- 대부분의 문자열 상황에서 `$( )`는 안전하게 작동한다.

```
echo "현재 사용자: $(whoami)"
echo "파일 개수: $(ls | wc -l)"		# wc = word count
```

#### 실습

EX1) 현재 날짜를 명령 치환으로 출력하시오

```
[root@Server-A ]# today=$(date)
[root@Server-A ]# echo "오늘 날짜는 $today 입니다."
오늘 날짜는 2025. 12. 07. (일) 18:05:53 KST 입니다.


[root@Server-A ]# today=$(date +%F)		# %F = YYYY-MM-DD 형식
[root@Server-A ]# echo "오늘 날짜는 $today 입니다."
오늘 날짜는 2025-12-07 입니다.
```

**date 주요 옵션 정리**

- 날짜(Date) 관련
```
%Y 	: 연도(4자리), 예: 2025
%y 	: 연도(2자리), 예: 25
%m 	: 월(01~12)
%d 	: 일(01~31)
%D 	: MM/DD/YY 형식 날짜 (미국식)
%F 	: YYYY-MM-DD (표준 ISO 형식)
```

- 시간(Time) 관련
```
%H  	: 시(00~23)
%I 	: 시(01~12), 12시간제
%M  	: 분(00~59)
%S  	: 초(00~59)
%p  	: AM 또는 PM
```

- 전체 날짜, 시간
```
%T 	: HH:MM:SS
%R  	: HH:MM
```

```
[root@Server-A ~]# echo $(date +%D)
07/22/26

[root@Server-A ~]# echo $(date +%F)
2026-07-22



[root@Server-A ~]# echo $(date +%T)
16:58:17

[root@Server-A ~]# echo $(date +%R)
16:58
```

EX2) 현재 로그인한 사용자 이름을 명령 치환으로 가져와 출력하시오
- whoami 명령을 사용하여 "사용자: 이름" 형태로 출력하시오.

```
[root@Server-A ~]# echo "사용자 : $(whoami)"
사용자 : root
```

EX3) 현재 사용 중인 쉘 종류를 출력하시오
- which bash 명령을 명령 치환으로 사용하여 경로를 출력하시오.

```
[root@Server-A ~]# echo "$(which bash)"
/usr/bin/bash
```

EX4) 현재 디렉터리 이름(폴더명)만 출력하시오
- pwd		: 전체 경로
- basename	: 경로의 마지막 이름
- 이 둘을 함께 사용하여 현재 디렉터리 이름만 출력하시오.

```
[root@Server-A ~]# cd /etc/NetworkManager/system-connections/


[root@Server-A system-connections]# pwd
/etc/NetworkManager/system-connections	# pwd는 전체 경로를 출력



[root@Server-A system-connections]# echo "현재위치 : $(basename $(pwd))"
현재위치 : system-connections
```

- 명령 치환 `$()` 안에는 다른 명령 치환 `$()`를 다시 넣을 수 있다.
- 이처럼 명령 치환 안에 또 다른 명령 치환을 사용하는 것을 중첩 명령 치환이라고 한다.
- 동작 순서는 안쪽 명령어부터 바깥쪽 명령어 순서이다.

EX5) /var/log 디렉터리의 파일 개수를 출력하시오
- `ls /var/log | wc -l` 결과를 명령 치환으로 출력하시오.

```
[root@Server-A ~]# echo "파일 개수 : $(ls -l | wc -l)"
파일 개수 : 23
```

EX6) 현재 시스템의 IP 주소를 명령 치환으로 출력하시오
- hostname -I 명령을 사용해서
- "Server-A IP: 주소" 형태로 출력하시오

```
[root@Server-A ~]# hostname
Server-A


[root@Server-A ~]# hostname -I
192.168.10.100


[root@Server-A ~]# echo "Server-A IP 주소 : $(hostname -I)"
Server-A IP 주소 : 192.168.10.100
```

EX7) 시스템의 현재 시간(HH:MM:SS)을 명령 치환으로 출력하시오
- 현재 시간을 "현재 시각: HH:MM:SS" 형태로 출력하시오

```
[root@Server-A ~]# echo "현재 시각 : $(date +%T)"
현재 시각 : 17:14:12
```

EX8) 현재 접속한 사용자 세션 수를 출력하시오
- who 명령으로 로그인 세션 수를 계산한 뒤 명령 치환으로 출력하시오.

```
[root@Server-A ~]# who
root	tty1     	2026-07-22 17:15
root     	pts/0      	2026-07-22 09:28 (192.168.10.1)
root     	pts/1    	2026-07-22 09:29 (192.168.10.1)
guest    	pts/2  	2026-07-22 09:54 (192.168.10.1)
```

- tty는 Teletypewriter의 약자입로 리눅스에서는 보통 로컬 가상 터미널을 의미
- pts는 Pseudo Terminal Slave의 약자로 SSH, Telnet, 터미널 프로그램, GUI 터미널 등을 통해 접속하면 주로 pts가 생성

```
[root@Server-A ~]# who | wc -l
4


[root@Server-A ~]# echo "세션 개수 : $(who | wc -l)"
세션 개수 : 4
```

**정리**: `$`, `${}`는 변수의 값을 읽고, `$()`는 명령 실행 결과를 문자열로 치환하며, 큰따옴표는 변수/명령 치환을 허용하되 공백을 보존하고 작은따옴표는 이 모든 확장을 차단한다는 점이 이 섹션의 핵심이다.

## 4) Bash 산술 연산

- Bash에서는 정수 계산을 수행할 때 `(( ))` 또는 `$(( ))`를 사용할 수 있다.
- 두 형식 모두 C 언어와 유사한 산술 연산자를 지원하지만 사용 목적이 다르다.

```
(( 산술식 )) 	: 산술식을 계산하거나 조건을 판단하는 산술 평가
$(( 산술식 ))	: 산술식을 계산한 결과를 값으로 반환하는 산술 확장
```

### 4-1) `$(( ))` 산술 확장(Arithmetic Expansion)

- **산술 확장**은 괄호 안의 산술식을 정수 연산으로 계산하고, 그 계산 결과를 하나의 값으로 반환하는 기능이다.
- 계산 결과를 화면에 출력하거나 변수에 저장하거나 다른 명령어의 인수로 전달할 때 사용한다.
- 기본 형식	: `$(( 산술식 ))`

예1) 계산 결과 출력

```
[root@Server-A ~]# echo $((10 + 20))
30
```

예2) 변수 값을 사용한 계산 (산술식 안에서는 변수명 앞에 `$` 생략 가능)

```
[root@Server-A ~]# a=10
[root@Server-A ~]# b=20
[root@Server-A ~]# c=$((a + b))

[root@Server-A ~]# echo $c
30
[root@Ser
```

예3) 문자열 안에서 사용

```
[root@Server-A ~]# cnt=10
[root@Server-A ~]# echo "최종 카운트 개수는 $((cnt))입니다."
최종 카운트 개수는 10입니다.
```

### 4-2) `(( ))` 산술 평가(Arithmetic Evaluation)

- `(( ))`는 산술식을 계산하거나 변수의 값을 변경하고, 산술식의 결과를 참 또는 거짓으로 판단할 때 사용한다.
- `(( ))` 자체는 계산 결과를 화면에 출력하지 않는다.
- 기본 형식	: `(( 산술식 ))`

예1) 변수에 계산 결과 대입

```
[root@Server-A ~]# a=10
[root@Server-A ~]# b=20
[root@Server-A ~]# ((c = a* 20))
[root@Server-A ~]# echo $c
200
```

예2) 변수 값 증가

```
[root@Server-A ~]# num=10
[root@Server-A ~]# ((++num))
[root@Server-A ~]# echo $num
11


[root@Server-A ~]# num=10
[root@Server-A ~]# ((num+=5))
[root@Server-A ~]# echo $num
15

[root@Server-A ~]# num=10
[root@Server-A ~]# ((num-=5))
[root@Server-A ~]# echo $num
5
```

예3) 조건 판단

```
[root@Server-A ~]# x=10
[root@Server-A ~]# ((x > 5)) && echo "True"
True

[root@Server-A ~]# echo $?
0			# 정상 처리



[root@Server-A ~]# ((x < 5)) && echo "True"

[root@Server-A ~]# echo $?
1			# 비정상 처리
```

### 4-3) 기본 산술 연산자

```
  +       	: 더하기
  -       	: 빼기
  *       	: 곱하기
  /       	: 나누기 (정수 나눗셈이므로 소수점 이하는 버린다.)
  %       	: 나머지(Modulo)
  **   	: 거듭제곱
```

```
[root@Server-A ~]# echo $((10 + 3))
13

[root@Server-A ~]# echo $((10 - 3))
7

[root@Server-A ~]# echo $((10 * 3))
30

[root@Server-A ~]# echo $((10 / 3))	# 산술연산은 기본적으로 정수만 계산한다.
3

[root@Server-A ~]# echo $((10 % 3))
1

[root@Server-A ~]# echo $((10 ** 3))
1000



[root@Server-A ~]# echo $(( 10 + 5 * 5 ))
35


[root@Server-A ~]# echo $(( (10 + 5) * 5 ))
75
```

### 4-4) 증감 연산자

- C 언어와 동일하게 전위(Prefix)와 후위(Postfix) 증감 연산자를 사용할 수 있다.
- `++변수`    : 값을 먼저 1 증가시킨 후 사용
- `변수++`    : 현재 값을 먼저 사용한 후 1 증가
- `--변수`    : 값을 먼저 1 감소시킨 후 사용
- `변수--`    : 현재 값을 먼저 사용한 후 1 감소

```
[root@Server-A ~]# num=10
[root@Server-A ~]# echo $((num += 1))		# 1 증가
11


[root@Server-A ~]# num=10
[root@Server-A ~]# echo $((++num))		# 1 증가
11


[root@Server-A ~]# num=10
[root@Server-A ~]# echo $((num++))
10
[root@Server-A ~]# echo $num
11



[root@Server-A ~]# num=10
[root@Server-A ~]# echo $((++num))
11
[root@Server-A ~]# echo $((num++))
11
[root@Server-A ~]# echo $((++num))
13
```

### 4-5) 복합 대입 연산자

- **복합 대입 연산자**는 변수의 기존 값에 산술 연산을 수행한 후, 그 결과를 다시 같은 변수에 저장하는 연산자이다.
- 즉, 계산과 대입을 한 번에 처리한다.

```
  =       : 값을 대입
  +=      : 더한 후 대입
  -=      : 뺀 후 대입
  *=      : 곱한 후 대입
  /=      : 나눈 후 대입
  %=      : 나머지를 계산한 후 대입
```

```
[root@Server-A ~]# num=10
[root@Server-A ~]# echo $((num+=5))
15


[root@Server-A ~]# echo $((num*=2))
30
```

### 4-6) 비교 연산자

- **비교 연산자**는 두 숫자의 크기나 같은지 여부를 비교할 때 사용한다.
- 비교 결과는 참(True) 또는 거짓(False)으로 판단된다.
- 산술 확장 `$(( ))` 안에서 비교하면 결과값으로 1 또는 0이 반환된다.
- 참(True)     	: 1 (0이 아닌 나머지는 참이된다.)
- 거짓(False)	: 0

```
  <       	: 왼쪽 값이 작음
  >       	: 왼쪽 값이 큼
  <=      	: 왼쪽 값이 작거나 같음
  >=      	: 왼쪽 값이 크거나 같음
  ==      	: 두 값이 같음
  != 	: 두 값이 다름
```

- 산술 평가 `(( ))` 안에서 비교하면 조건의 참/거짓에 따라 종료 상태(exit code)가 반환된다.
- 조건이 참  	: 종료 상태 0
- 조건이 거짓	: 종료 상태 0이아닌 나머지

```
[root@Server-A ~]# a=10
[root@Server-A ~]# b=20
[root@Server-A ~]# echo $((a < b))
1

[root@Server-A ~]# echo $?
0



[root@Server-A ~]# a=10
[root@Server-A ~]# b=20
[root@Server-A ~]# ((a > b))

[root@Server-A ~]# echo $?
1



[root@Server-A ~]# a=10
[root@Server-A ~]# b=20
[root@Server-A ~]# (( a < b)) && c=true

[root@Server-A ~]# echo $c
true
```

### 4-7) 논리 연산자

- **논리 연산자**는 하나 이상의 조건식을 연결하거나, 조건식의 참/거짓을 반대로 변경할 때 사용한다.
- Bash의 산술식에서는 다음과 같이 판단한다.

```
  &&      	: 논리 AND (두 조건이 모두 참이면 참)
  ||      	: 논리 OR (하나 이상의 조건이 참이면 참)
  !       	: 논리 NOT (참과 거짓을 반대로 변경)
```

예1) &&(AND) 연산자를 사용한 true/flase 확인

```
[root@Server-A ~]# echo $(( 10 > 5 &&  20 < 30 ))
1

# 10 > 5 = true , 20 < 30 = true 	: true



[root@Server-A ~]# echo $(( 10 > 5 &&  20 > 30 ))
0

# 10 > 5 = true , 20 < 30 = flase 	: flase
```

예2) ||(OR) 연산자를 사용한 true/flase 확인

```
[root@Server-A ~]# echo $(( 10 > 5 ||  20 < 30 ))
1

# 10 > 5 = true , 20 < 30 = true 	: true


[root@Server-A ~]# echo $(( 10 > 5 ||  20 > 30 ))
1
# 10 > 5 = true , 20 < 30 = flase 	: true
```

예3) !(NOT) 연산자를 사용한 true/flase 확인

```
[root@Server-A ~]# echo $(( (10 < 20) ))
1


[root@Server-A ~]# echo $(( !(10 < 20) ))
0
```

### 실습 문제

EX1) 7 + 3 의 결과를 `$(( ))` 로 출력하시오

```
[root@Server-A glob_lab]# echo $((7 + 3))
10
```

EX2) 변수 a=10, b=4 일 때 a*b 결과를 출력하시오

```
[root@Server-A glob_lab]# a=10; b=20
[root@Server-A glob_lab]# echo $((a * b))
200
```

EX3) 15 / 4 의 몫을 출력하시오

```
[root@Server-A glob_lab]# echo $((15 / 4))
3
```

EX4) 27 % 5 의 나머지를 출력하시오

```
[root@Server-A glob_lab]# echo $((27 % 5))
2
```

EX5) num=7 일 때 num + 10 의 결과를 출력하시오

```
[root@Server-A glob_lab]# num=7

[root@Server-A glob_lab]# echo $((num + 10))
17
```

EX6) x=3, y=4 일 때 x² + y² 의 값을 계산하시오

```
[root@Server-A glob_lab]# x=3;  y=4

[root@Server-A glob_lab]# echo $((x**2 + y**2))
25
```

EX7) 현재 디렉터리 파일 개수에 100을 더해서 출력하시오

```
[root@Server-A glob_lab]# ls -l | wc -l
181


[root@Server-A glob_lab]# echo $(ls -l | wc -l)
181


[root@Server-A glob_lab]# echo $(( $(ls -l | wc -l) + 100 ))
281
```

EX8) 3 + 2 × 8  의 결과를 출력하시오

```
[root@Server-A glob_lab]# echo $((3 + 2 * 8))
19
```

EX9) (50 - 20) × 2 계산 결과를 출력하시오

```
[root@Server-A glob_lab]# echo $(( (50 - 20) * 2 ))
60
```

EX10) 파일 크기 합계 계산
- /tmp 디렉터리의 파일 개수 × 2 를 출력하시오

```
[root@Server-A ~]# ls  -l | wc -l
23


[root@Server-A ~]# echo $(( $(ls  -l | wc -l) * 2  ))
46
```

EX11) x=5 일 때 x > 3 이 참이면 OK 출력

```
[root@Server-A ~]# x=5


[root@Server-A ~]# (( x > 3 )) && echo "OK"
OK
```

EX12) y=2 일 때 y == 5 가 거짓이면 FAIL 출력

```
[root@Server-A ~]# y=2


[root@Server-A ~]# (( y == 5)) || echo "FAIL"		# && = True일때 오른쪽을 실행 , || = Flase일때 오른쪽을 실행
FAIL
```

EX13) num=10 이 5~20 범위 안에 있으면 IN 출력

```
[root@Server-A ~]# num=10


[root@Server-A ~]# (( num >= 5 && num <= 20 )) && echo "IN"
IN
```

EX14) a=3, b=7 일 때 두 값이 다르면 "DIFF" 출력

```
[root@Server-A ~]# a=3
[root@Server-A ~]# b=7

[root@Server-A ~]# (( a == b )) || echo "DIFF"
DIFF


[root@Server-A ~]# (( a != b )) && echo "DIFF"
DIFF
```

EX15) x=0 일 때 x가 0이면 ZERO 출력

```
[root@Server-A ~]# x=0

[root@Server-A ~]# (( x == 0 )) && echo "ZERO"
ZERO
```

EX16) 파일 개수가 10개 이상이면 MANY 출력

```
[root@Server-A ~]# cnt=$(ls -l | wc -l)


[root@Server-A ~]# echo $cnt
23


[root@Server-A ~]# (( cnt >= 10 )) && echo "MANY"
MANY



[root@Server-A ~]# cnt=$(ls -l /temp | wc -l)


[root@Server-A ~]# (( cnt >= 10 )) && echo "MANY"
```

EX17) 두 숫자가 같으면 SAME, 아니면 DIFFERENT 출력

```
[root@Server-A ~]# a=10
[root@Server-A ~]# b=20


[root@Server-A ~]# (( a == b )) && echo "SAME" || echo "DIFFE"
DIFFE


[root@Server-A ~]# b=10


[root@Server-A ~]# (( a == b )) && echo "SAME" || echo "DIFFE"
SAME
```

```
[root@Server-A ~]# set +H

[root@Server-A ~]# power=0

[root@Server-A ~]# (( power=!power)) && echo "전원 ON" || echo "전원 OFF"
전원 ON


[root@Server-A ~]# (( power=!power)) && echo "전원 ON" || echo "전원 OFF"
전원 OFF


[root@Server-A ~]# (( power=!power)) && echo "전원 ON" || echo "전원 OFF"
전원 ON


[root@Server-A ~]# (( power=!power)) && echo "전원 ON" || echo "전원 OFF"
전원 OFF
```

**정리**: `$(( ))`는 산술 결과를 값으로 반환(산술 확장)하고 `(( ))`는 계산·대입·조건 판단을 수행하되 값을 출력하지 않는(산술 평가) 차이가 핵심이며, 비교/논리 연산의 결과는 참=1(exit 0), 거짓=0(exit 1)로 대응된다.

## 5) 리다이렉션(Redirection)

- **리다이렉션**은 명령어의 입력과 출력 방향을 변경하는 기능이다.
- Linux 명령어는 기본적으로 키보드로부터 데이터를 입력받고, 처리 결과를 화면에 출력한다.
- 리다이렉션을 사용하면 키보드 대신 파일에서 데이터를 입력받거나, 화면에 출력되는 결과를 파일에 저장할 수 있다.
- 리다이렉션은 명령어의 실행 결과만 변경하는 것이 아니라 명령어가 사용하는 입력/출력 장치를 변경하는 기능이다.

```
키보드  --입력-->  프로그램  --출력-->  화면
```
- 입력은 키보드에서 받는다.
- 출력은 화면(터미널)으로 보낸다.
- 리다이렉션을 사용하면 입력과 출력의 방향을 자유롭게 변경할 수 있다.

### 표준 입출력

Linux의 명령어와 프로세스는 기본적으로 다음 세 가지 입출력 통로를 사용한다.
- 표준 입력   : Standard Input
- 표준 출력   : Standard Output
- 표준 오류   : Standard Error

각 표준 입출력에는 파일 디스크립터 번호가 지정되어 있다.
- 0 : 표준 입력  stdin
- 1 : 표준 출력  stdout
- 2 : 표준 오류  stderr

| Communication channels | Redirection characters | 의미 |
|---|---|---|
| STDIN | `0<`, `0<<` | 입력을 키보드가 아닌 파일을 통해 받음 |
| STDOUT | `1>`, `1>>` | 표준 출력을 터미널이 아닌 파일로 출력 |
| STDERR | `2>`, `2>>` | 표준 에러 출력을 터미널이 아닌 파일로 출력 |

### 표준 입력(Standard Input)

- 명령어가 데이터를 입력받는 기본 통로이다.
- 기본 입력 장치는 키보드이다.
  - 파일 디스크립터 번호 : 0
  - 기본 장치                : 키보드

### 표준 출력(Standard Output)

- 명령어가 정상 처리 결과를 출력하는 기본 통로이다.
- 기본 출력 장치는 화면이다.
- 파일 디스크립터 번호 : 1
- 기본 장치           	   : 모니터

### 표준 오류(Standard Error)

- 명령어 실행 중 발생한 오류 메시지를 출력하는 통로이다.
- 기본 출력 장치는 화면이다.
- 파일 디스크립터 번호 : 2
- 기본 장치                : 모니터
- 표준 출력과 표준 오류는 모두 기본적으로 화면에 출력되지만 서로 다른 통로를 사용한다.

### 입력 리다이렉션(Input Redirection)

- 셸(Shell)의 기본 입력은 키보드이다.
  하지만 명령이 받을 입력을 파일, 텍스트 블록, 문자열로 바꾸고 싶을 때 입력 리다이렉션을 사용한다.

#### `<` (입력 리다이렉션: 파일 --> 명령)

- 형식 : `명령 < 파일명`
- 파일 내용을 명령의 표준 입력(stdin) 으로 전달한다.
- 즉, 키보드 대신 파일을 읽어라는 의미

#### `<<` (Here-document)

형식:
```
명령 << 종료문자
내용내용내용
내용내용내용
종료문자
```

- 스크립트 안에서 여러 줄 입력을 직접 제공하고 싶을 때 사용
- 종료문자(EOF, END 등)까지 입력 그대로 전달된다
- 주로 셸 스크립트에서 사용된다.

#### 실습

EX1) names.txt 파일의 내용을 sort 명령에 입력으로 보내 오름차순 정렬하시오

```
[root@Server-A ~]# cd  meta_lab/


[root@Server-A meta_lab]# ls -l
합계 16
drwxr-xr-x 4 root root    72  7월 22 12:46 and_or_lab
drwxr-xr-x 2 root root 4096  7월 22 12:46 brace_lab
drwxr-xr-x 2 root root 8192  7월 22 18:21 glob_lab
drwxr-xr-x 2 root root    79  7월 22 12:46 input_lab


[root@Server-A meta_lab]# cd input_lab/


[root@Server-A input_lab]# ls -l ./name*
-rw-r--r-- 1 root root 23  7월 22 12:46 ./names.txt



[root@Server-A input_lab]# cat names.txt
kim
lee
park
choi
jung



[root@Server-A input_lab]# sort < names.txt		# 알파벳 순으로 정렬
choi
jung
kim
lee
park


[root@Server-A input_lab]# sort -r  < names.txt		# 역순으로 정렬
park
lee
kim
jung
choi
```

EX2) log.txt 파일의 줄 수를 wc 명령으로 계산하시오

```
[root@Server-A input_lab]# cat log.txt
[INFO] service started
[INFO] user login
[WARN] low disk space
[ERROR] connection timeout
[INFO] service ended


[root@Server-A input_lab]# cat  log.txt | wc -l
5



[root@Server-A input_lab]# wc -l < log.txt
5
```

EX3) Here-document(<<)를 이용하여 cat 명령으로 세 줄의 텍스트를 출력하시오

```
[root@Server-A input_lab]# cat  <<  EOF
> line1
> line2
> line3
> EOF
line1
line2
line3
```

### 출력 리다이렉션(Output Redirection)

#### 단일 덮어쓰기 출력: `>`

- 출력 리다이렉션은 화면 대신 파일로 결과를 저장하는 기능이다.
- 기본적으로 프로그램의 실행 결과는 화면으로 출력된다.
- 형식 : `명령 > 파일명`
- 명령의 표준 출력(stdout)을 파일로 보낸다.
- 파일이 없으면 새로 생성되고 있으면 해당 파일안에 출력결과가 저장된다.
- 파일안에 이미 있으면 내용을 완전히 덮어쓴다(overwrite)

```
[root@Server-A input_lab]# echo  Soldesk  >  test.txt	# echo는 기본적으로 모니터로 출력되지만 출력 경로를 파일로 변경했기때문에 화면에 출력 X

[root@Server-A input_lab]# cat  test.txt
Soldesk



[root@Server-A input_lab]# echo  Linux stdIN  > test.txt	# 파일안의 모든 내용이 삭제되고 덮어쓰기된다.
[root@Server-A input_lab]# cat  test.txt
Linux stdIN


[root@Server-A input_lab]# echo CISCO stdIN 1> test.txt
[root@Server-A input_lab]# cat  test.txt
CISCO stdIN
```

#### 추가(append) 출력: `>>`

- 형식 : `명령 >> 파일명`
- 파일이 이미 있으면 기존 내용 뒤에 이어붙인다(append)
- 파일이 없으면 새로 생성
- 기존 내용을 지우지 않는다

```
[root@Server-A input_lab]# echo line1 > test2.txt


[root@Server-A input_lab]# cat test2.txt
line1


[root@Server-A input_lab]# echo line2 >> test2.txt
[

[root@Server-A input_lab]# cat test2.txt
line1
line2
```

EX1) hello 라는 문자열을 msg.txt에 저장하시오

```
[root@Server-A input_lab]# echo  hello > msg.txt

[root@Server-A input_lab]# cat msg.txt
hello
```

EX2) 파일 덮어쓰기 기본 동작 확인
- 문자 A 를 data1.txt 에 저장하시오.
- 그 후 문자 B 를 같은 파일에 덮어쓰시오.

```
[root@Server-A ~]# echo A  >  data1.txt
[root@Server-A ~]# echo B  >  data1.txt
[root@Server-A ~]# cat data1.txt
B
```

EX3) date 결과를 now.txt에 저장하시오

```
[root@Server-A input_lab]# date > now.txt


[root@Server-A input_lab]# cat now.txt
2026. 07. 23. (목) 12:59:39 KST
```

EX4) 파일에 줄 2개를 순서대로 추가하기
- data2.txt 에 hello, world 두 줄을 추가(append)하시오.

```
[root@Server-A ~]# echo hello  >  data2.txt
[root@Server-A ~]# echo world  >>  data2.txt

[root@Server-A ~]# cat data2.txt
hello
world
```

EX5) 다음 두 명령을 이용해 multi.txt 파일을 만들어라
- 첫 줄	: one (>)
- 둘째 줄: twoe (>>)

```
[root@Server-A ~]# echo  one  >  multi.txt
[root@Server-A ~]# echo  twoe  >>  multi.txt

[root@Server-A ~]# cat multi.txt
one
twoe
```

EX6) ls 결과를 파일로 저장하기
- 현재 디렉터리 목록을 list.txt 파일로 저장하시오.

```
[root@Server-A input_lab]# ls  1> list.txt


[root@Server-A input_lab]# cat  list.txt
aaa.txt
data.txt
list.txt
log.txt
messages_demo.log
msg.txt
names.txt
now.txt
output.txt
sort_names.txt
test2.txt
```

EX7) cat 출력 결과를 다른 파일로 복사하기
- source.txt 파일을 만든 뒤, 그 내용을 result.txt 로 복사하시오.
- (source.txt 안에는 ABC 가 들어 있어야 함)

```
[root@Server-A ~]# echo ABC  >  source.txt

[root@Server-A ~]# cat  source.txt  >  result.txt

[root@Server-A ~]# cat result.txt
ABC
```

### 에러 리다이렉션(Error Redirection)

- 프로그램 실행 중 발생하는 오류 메시지는 표준 에러(STDERR)를 통해 출력된다.
- 기본적으로 표준 에러도 표준 출력과 마찬가지로 화면에 출력된다.
- 표준 출력과 표준 에러는 모두 화면에 출력되지만 서로 다른 통로를 사용한다.
  - 표준 출력(STDOUT) : 명령어의 정상적인 실행 결과
  - 표준 에러(STDERR) : 명령어 실행 중 발생한 오류 메시지
- 표준 출력의 파일 디스크립터 번호는 1이고, 표준 에러의 파일 디스크립터 번호는 2이다.
  - 0 : 표준 입력  STDIN
  - 1 : 표준 출력  STDOUT
  - 2 : 표준 에러  STDERR
- 출력 리다이렉션 기호 `>`만 사용하면 표준 출력만 파일로 전송된다.
  - 명령어 > 파일
- 오류 메시지를 파일로 전송하려면 표준 에러의 파일 디스크립터 번호인 2를 사용해야 한다.
  - 명령어 2> 파일    (2>는 명령어의 오류 메시지를 지정한 파일에 저장)
- 대상 파일이 없으면 새로 생성한다.
- 대상 파일이 존재하면 기존 내용을 삭제하고 새로운 오류 메시지로 덮어쓴다.
  - `ls /없는디렉터리 2> error.txt`
- 위 명령은 /없는디렉터리를 찾을 수 없다는 오류 메시지를 error.txt 파일에 저장한다.
- 오류 메시지가 파일로 전송되므로 화면에는 출력되지 않는다.

EX1) 존재하지 않는 파일 test_err.txt를 조회할 때 발생하는 에러 메시지를 error_log.txt 파일에 저장하시오.

```
[root@Server-A input_lab]# ls  -l  test_err.txt
ls: cannot access 'test_err.txt': 그런 파일이나 디렉터리가 없습니다


[root@Server-A input_lab]# ls  -l  test_err.txt  2>  error_log.txt


[root@Server-A input_lab]# cat  error_log.txt
ls: cannot access 'test_err.txt': 그런 파일이나 디렉터리가 없습니다


[root@Server-A input_lab]# vi  error_log.txt
ls: cannot access 'test_err.txt': 그런 파일이나 디렉터리가 없습니다
```

EX2) 존재하지 않는 파일 sample.txt를 cat 명령으로 열어 에러를 error_log2.txt에 저장하시오.

```
[root@Server-A input_lab]# cat  sample.txt
cat: sample.txt: 그런 파일이나 디렉터리가 없습니다



[root@Server-A input_lab]# cat  sample.txt  2>  error_log2.txt



[root@Server-A input_lab]# cat  error_log2.txt
cat: sample.txt: 그런 파일이나 디렉터리가 없습니다
```

EX3) 존재하지 않는 사용자 user1의 홈 디렉터리를 조회하여 발생하는 에러를 err.txt에 저장하시오.

```
[root@Server-A input_lab]# ls  -l  /home/user1
ls: cannot access '/home/user1': 그런 파일이나 디렉터리가 없습니다


[root@Server-A input_lab]# ls  -l  /home/user1  2>  error_log3.txt


[root@Server-A input_lab]# cat  error_log3.txt
ls: cannot access '/home/user1': 그런 파일이나 디렉터리가 없습니다
```

EX4) 존재하지 않는 파일 data2.txt를 삭제하면서 발생하는 에러를 error.log에 저장하시오.

```
[root@Server-A input_lab]# rm  data2.txt
rm: cannot remove 'data2.txt': 그런 파일이나 디렉터리가 없습니다



[root@Server-A input_lab]# rm  data2.txt  2>  error.log



[root@Server-A input_lab]# cat  error.log
rm: cannot remove 'data2.txt': 그런 파일이나 디렉터리가 없습니다
```

EX5) 존재하지 않는 파일 sample.txt를 두 번 조회하여 2>와 2>>의 차이를 확인하시오.

```
[root@Server-A input_lab]# ls  -l  sample.txt  2>  error2.log


[root@Server-A input_lab]# cat  error2.log
ls: cannot access 'sample.txt': 그런 파일이나 디렉터리가 없습니다	# 파일 생성 후 에러 메세지를 생성한 파일에 저장한다.


[root@Server-A input_lab]# ls  -l  sample.txt  2>  error2.log


[root@Server-A input_lab]# cat  error2.log
ls: cannot access 'sample.txt': 그런 파일이나 디렉터리가 없습니다	# >는 덮어쓰기 기능이므로 1줄만 확인된다.




[root@Server-A input_lab]# ls  -l  sample.txt  2>>  error2.log	# >> 는 기존 데이터의 가자 마지막줄에 추가된다.
[root@Server-A input_lab]# ls  -l  sample.txt  2>>  error2.log	# >> 는 기존 데이터의 가자 마지막줄에 추가된다.


[root@Server-A input_lab]# cat  error2.log
ls: cannot access 'sample.txt': 그런 파일이나 디렉터리가 없습니다
ls: cannot access 'sample.txt': 그런 파일이나 디렉터리가 없습니다
ls: cannot access 'sample.txt': 그런 파일이나 디렉터리가 없습니다
```

**정리**: 리다이렉션은 표준 입력(0)/출력(1)/에러(2) 통로를 파일로 바꿔주는 기능으로, `>`는 덮어쓰기, `>>`는 추가, `2>`는 에러만 별도 파일로 분리 저장한다는 점을 실습을 통해 확인했다.

## 6) 파이프(Pipe)

- **파이프(|)**는 왼쪽 명령의 출력(stdout)을 오른쪽 명령의 입력(stdin)으로 전달하는 기능이다.
- 형식 : `명령어1 | 명령어2`
- 명령1의 결과를 화면에 출력하지 말고 그 결과를 명령2의 입력으로 보내라
  그래서 파이프는 여러 명령을 체인(chain) 형태로 연결할 수 있다.

```
# ls -l | grep ".log"
1) ls -l : 현재 디렉터리 목록 출력
2) grep ".log" : 출력된 목록에서 .log 포함된 줄만 필터링
```

- 파이프는 언제 사용하는가?
  - 파일이나 프로세스 목록에서 특정 조건만 골라낼 때
  - 여러 명령을 조합해서 강력한 기능을 만들 때
  - 데이터를 변환, 정렬, 필터링 할 때

- EX) `ls -l | grep ".log"`	: ls -l의 출력 중에서 .log가 포함된 줄만 필터링해서 출력.
- EX) `cat scores.txt | sort -n`	: 파일 내용을 화면에 출력(cat)하고 바로 숫자 정렬(sort -n)하여 출력
- EX) `ps aux | grep sshd`	: 실행 중인 프로세스 목록 중 sshd 관련된 것만 출력
- EX) `ifconfig | grep inet | cut -d' ' -f10`	: 네트워크 설정에서 inet가 포함된 줄만 골라서 10번째 필드만 출력(예: IP 주소)

EX1) /var/log/ 파일에서 vmware 단어 포함된 라인만 출력하시오

```
[root@Server-A input_lab]# ls  -l  /var/log/ | grep vmware
-rw-------  1 root   root        256  7월 22 18:21 vmware-network.1.log
-rw-------  1 root   root        197  7월 22 09:27 vmware-network.2.log
-rw-------  1 root   root        310  7월 21 15:58 vmware-network.3.log
-rw-r--r--. 1 root   root         197  7월  2 12:56 vmware-network.4.log
-rw-------  1 root   root        197  7월 23 09:33 vmware-network.log
-rw-------. 1 root   root     14312  7월 23 09:33 vmware-vgauthsvc.log.0
-rw-------. 1 root   root     34443  7월 23 09:33 vmware-vmsvc-root.log
-rw-------. 1 root   root       7815  7월 23 09:33 vmware-vmtoolsd-root.log
-rw-------. 1 root   root     10065  7월 21 15:59 vmware-vmusr-root.log
```

EX2) 프로세스 목록 중 sshd 문자열 포함된 항목만 출력하시오

```
[root@Server-A input_lab]# ps aux | grep sshd
root          834  0.0	  0.5  14412  8764 ?        	Ss   09:33	   0:00 sshd: /usr/sbin/sshd -D [listener] 0 of 10-100 startups
root        1862  0.0	  0.7  18404 12432 ?        	Ss   09:35	   0:00 sshd-session: root [priv]
root        1877  0.0	  0.4  18664  7584 ?        	S    09:35	   0:01 sshd-session: root@pts/0	# putty 접속 1
root        1911  0.0	  0.7  18404 12608 ?        	Ss   09:35	   0:00 sshd-session: root [priv]
root        1915  0.0	  0.4  18664  7552 ?        	S    09:35	   0:00 sshd-session: root@pts/1	# putty 접속 2
root        2533  0.0	  0.1 221696  2516 pts/0	S+   14:56	   0:00 grep --color=auto sshd
```

```
a : 모든 사용자의 프로세스 출력
u : 사용자 중심 형식으로 상세 출력
x : 터미널에 연결되지 않은 프로세스도 출력

[root@Server-A input_lab]# kill 1877		# PID를 사용하여 특정 프로세스를 지울수 있다.ux
```

EX3) 현재 디스크 사용량을 정렬해서 출력하시오

```
[root@Server-A ~]# df -h
Filesystem     	Size   Used     Avail    Use%	Mounted on
devtmpfs        	807M	0     807M       0% 	/dev
tmpfs           	838M    	0     838M       0% 	/dev/shm
tmpfs           	335M  5.2M     330M       2% 	/run
/dev/sda2        	16G    5.6G      11G      35% 	/
tmpfs    		168M    40K    168M       1% 	/run/user/0



[root@Server-A ~]# df -h | sort  -k 5			# k5 = 5번째 컬럼을 기준으로 정렬
devtmpfs        	807M     	0    807M     0% 	/dev
tmpfs           	838M     	0    838M     0%	/dev/shm
tmpfs           	168M    40K   168M     1% 	/run/user/0
tmpfs           	335M  5.2M    330M     2% 	/run
/dev/sda2        	16G    5.6G     11G    35%	/
Filesystem      	Size  Used   Avail    Use%	Mounted on


[root@Server-A ~]# df -h | sort  -k 5  -r		# k5 = 5번째 컬럼을 기준으로 역순으로 정렬
Filesystem      	Size    Used     Avail    Use% 	Mounted on
/dev/sda2        	16G    5.6G     11G       35% 	/
tmpfs           	335M  5.2M     330M      2% 	/run
tmpfs           	168M    40K    168M      1% 	/run/user/0
tmpfs           	838M     	0     838M      0% 	/dev/shm
devtmpfs        	807M     	0     807M      0%	/dev
```

EX4) 현재 네트워크 정보에서 inet 포함된 줄만 출력하시오

```
[root@Server-A ~]# ifconfig | grep inet
        inet 192.168.10.100  netmask 255.255.255.0  broadcast 192.168.10.255
        inet6 fe80::20c:29ff:fe66:2f9c  prefixlen 64  scopeid 0x20<link>
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
```

**정리**: 파이프(`|`)는 앞 명령의 표준 출력을 뒷 명령의 표준 입력으로 그대로 연결하여, `grep`/`sort`/`cut`/`wc` 등과 조합해 필터링·정렬·가공을 한 줄로 처리할 수 있게 한다.

## 7) 명령 연결 / 제어 메타 문자

### 세미콜론( ; )

- **세미콜론**은 여러 명령을 한 줄에서 순서대로 실행할 때 사용하는 셸의 명령 구분자(command separator)이다.
  즉, 줄바꿈 없이 명령 여러 개를 이어서 실행하고 싶을 때 사용한다.

EX) `command1; command2; command3`

- 동작
  - command1 실행  -->  완료되면
  - command2 실행  -->  완료되면
  - command3 실행

- 특징
  - 왼쪽 명령 성공 여부와 상관없이 오른쪽 명령 실행
  - `ls /no/such/path; echo "Done"`
  - ls 실패해도 echo "Done" 은 실행됨
  - 이는 `&&`, `||` 처럼 성공/실패 조건을 보지 않는다는 점에서 다르다.
  - 줄바꿈과 동일한 효과

아래 두 명령은 완전히 동일한 동작이다:

```
mkdir test; cd test

mkdir test
cd test
```

- 명령 사이의 공백은 있어도 되고 없어도 된다.

```
date;whoami;pwd

또는

date;  whoami;  pwd
```

- 스크립트에서도 동일하게 사용 가능
- 한 줄로 여러 명령을 처리할 때 유용하다.

언제 사용하나?
- 간단한 작업 여러 개를 빠르게 실행할 때
  - `cd /var/log; ls; pwd`
- 스크립트에서 초기 설정 작업을 한 줄에 정리할 때
  - `export LANG=C; export PATH=/usr/bin:$PATH`
- 테스트나 확인용 명령을 연속으로 실행할 때
  - `echo "Start"; date; uptime; echo "End"`

```
[root@Server-A ~]# date;  whoami;  pwd		# 시간 출력  -->  사용자 출력  -->  현재 경로 출력
2026. 07. 23. (목) 15:12:15 KST
root
/root



[root@Server-A ~]# touch  semi.txt;  echo "파일 생성 완료";  ls -l ./semi.txt;  echo "파일 생성 확인"
파일 생성 완료
-rw-r--r-- 1 root root 0  7월 23 15:13 ./semi.txt
파일 생성 확인



[root@Server-A ~]# ping -c 2  www.google.com;  ping -c 2  www.spring.org
PING www.google.com (142.251.155.119) 56(84) bytes of data.
64 bytes from 142.251.155.119 (142.251.155.119): icmp_seq=1 ttl=128 time=31.0 ms
64 bytes from 142.251.155.119 (142.251.155.119): icmp_seq=2 ttl=128 time=32.2 ms

--- www.google.com ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1003ms
rtt min/avg/max/mdev = 30.996/31.603/32.210/0.607 ms
PING www.spring.org (216.40.34.37) 56(84) bytes of data.
64 bytes from 216.40.34.37 (216.40.34.37): icmp_seq=1 ttl=128 time=190 ms
64 bytes from 216.40.34.37 (216.40.34.37): icmp_seq=2 ttl=128 time=191 ms

--- www.spring.org ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1002ms
rtt min/avg/max/mdev = 190.225/190.581/190.938/0.356 ms
```

EX1) /tmp으로 이동한 뒤 ls를 실행하시오

```
[root@Server-A ~]# cd /var/log; ls; cat messages
```

EX2) /home으로 이동하고 pwd를 출력하시오

```
[root@Server-A ~]#  cd /home; pwd
/home
[root@Server-A home]#
```

EX3) /var/log로 이동 후 ls와 cat messages까지 실행하시오

```
[root@Server-A ~]# cd /var/log; ls; cat messages
```

EX4) mkdir test 만들고 ls 실행하시오

```
[root@Server-A log]# mkdir test; ls -l ./test
합계 0
```

EX5) date 실행 후 who 실행하시오

```
[root@Server-A log]# date; who
2025. 12. 10. (수) 10:58:35 KST
guest    pts/0        2025-12-10 10:41 (192.168.111.1)
guest    pts/1        2025-12-10 10:54 (192.168.111.1)
```

**정리**: 세미콜론(`;`)은 앞 명령의 성공/실패와 무관하게 다음 명령을 무조건 순차 실행하는 단순 구분자이다.

### && (AND) – 명령 연결 제어

- **`&&`** 는 앞 명령이 성공(exit code 0) 했을 때만 다음 명령을 실행하는 명령 연결 연산자이다.
  즉, "성공했을 때만 이어서 실행"하는 조건부 실행 방식이다.

- 사용 목적
  - 특정 작업이 정상적으로 끝났을 때만 다음 작업을 이어가도록 만들 때
  - 실패 시 자동 중단되게 하여 스크립트의 안정성을 높일 때
  - if문을 간단히 대체하는 용도로 자주 사용됨

- 동작 구조
  - 명령1 && 명령2
  - 명령1 성공  -->  명령2 실행
  - 명령1 실패  -->  명령2 실행하지 않음

- 종료 코드에 따른 동작
  - exit 0(성공)  -->  true  -->  AND 다음 명령 수행
  - exit 1(실패)  -->  false  -->  전체 실행 종료

- 주요 특징
  - if문 없이 짧게 조건 실행 가능
  - `&&` 왼쪽 명령이 실패하면 즉시 중단
  - `&&` 는 조건 기반 실행 AND 개념

- 앞 명령이 성공했을 때만 다음 명령 실행
- EX) `mkdir test && cd test`	: test 디렉터리를 만드는데 성공하면 cd test로 이동

EX1) test 디렉터리를 만들고 성공하면 그 디렉터리로 이동하시오

```
[root@Server-A ~]# mkdir  /test  &&  cd  /test


[root@Server-A test]# pwd		# /test 디렉터리가 생성되었기때문에  cd 명령어를 사용하여 /test 디렉터리로 이동된다.
/test



[root@Server-A test]# cd ~
[root@Server-A ~]# pwd
/root



[root@Server-A ~]# mkdir  /test  &&  cd  /test
mkdir: `/test' 디렉토리를 만들 수 없습니다: 파일이 있습니다


[root@Server-A ~]# pwd		# /test 디렉터리가 이미 생성되어있기 때문에 cd 명령어로 이동하지 못한다. 
/root
```

EX2) /backup 이름의 디렉터리가 존재하면 /backup 디렉터리의 정보를 ls 명령어로 확인 후 cd 명령어를 사용하여 이동

```
[root@Server-A ~]# ls -l /backup				# backup 디렉터리가 확인되지 않는다.
ls: cannot access '/backup': 그런 파일이나 디렉터리가 없습니다




[root@Server-A ~]# [ -d  /backup ]		# 1) 해당 경로에 /backup이 있는지 확인,  2) 있다면 디렉터리인지 확인

[root@Server-A ~]# echo $?		
1					# 디렉터리가 없기때문에 exit code가 1로 확인




[root@Server-A ~]# touch  /backup		# /backup 파일 생성

[root@Server-A ~]# echo $?			# /backup  해당 경로의 이름이 파일이므로 exit code가 1이된다.
1


[root@Server-A ~]# rm  -rf  /backup		# 파일 삭제


[root@Server-A ~]# mkdir  /backup


[root@Server-A ~]# echo $?			# /backup이 디렉터리이므로  exit code가 0으로 확인된다.
0



[root@Server-A ~]# cp  -r  /etc/a*  /backup


[root@Server-A ~]# [ -d  /backup ] &&  ls -l  /backup  && cd  /backup
합계 36
drwxr-xr-x 3 root root    28  7월 23 15:58 accountsservice
-rw-r--r-- 1 root root    16  7월 23 15:58 adjtime
-rw-r--r-- 1 root root 1529  7월 23 15:58 aliases
drwxr-xr-x 3 root root    65  7월 23 15:58 alsa
drwxr-xr-x 2 root root 4096  7월 23 15:58 alternatives
-rw-r--r-- 1 root root   541  7월 23 15:58 anacrontab
-rw-r--r-- 1 root root   269  7월 23 15:58 anthy-unicode.conf
-rw-r--r-- 1 root root   833  7월 23 15:58 appstream.conf
-rw-r--r-- 1 root root    55  7월 23 15:58 asound.conf
-rw-r--r-- 1 root root      1  7월 23 15:58 at.deny
drwxr-x--- 4 root root  100  7월 23 15:58 audit
drwxr-xr-x 3 root root 4096  7월 23 15:58 authselect
drwxr-xr-x 4 root root    71  7월 23 15:58 avahi


[root@Server-A backup]# pwd
/backup
```

EX3) userlist.txt 생성이 성공하면 echo "File Create OK" 출력하고 /etc/passwd안의 내용을 userlist.txt로 출력해야 한다..

```
[root@Server-A backup]# touch userlist.txt  &&  echo  "File Create OK"  &&  cat  /etc/passwd  >  userlist.txt
File Create OK



[root@Server-A backup]# cat userlist.txt
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
~~~~~~~~~~ 중간 생략 ~~~~~~~~~~
chrony:x:986:981::/var/lib/chrony:/sbin/nologin
dnsmasq:x:985:980:Dnsmasq DHCP and DNS server:/var/lib/dnsmasq:/sbin/nologin
tcpdump:x:72:72::/:/sbin/nologin
systemd-oom:x:978:978:systemd Userspace OOM Killer:/:/usr/sbin/nologin
guest:x:1000:1000:guest:/home/guest:/bin/bash
```

EX4) /var 아래 log 디렉터리가 있으면 이동한 후 ssh_err.log 파일을 생성해야 한다.

```
[root@Server-A ~]# [ -d  /var/log ] && cd  /var/log/  &&  touch  ssh_err.log


[root@Server-A log]# pwd
/var/log


[root@Server-A log]# ls  -l  /var/log/ssh_err.log
-rw-r--r-- 1 root root 0  7월 23 16:10 /var/log/ssh_err.log
```

EX5) /temp/test.txt 생성이 성공하면 test.txt 파일에 HELLO 문자열을 저장, 저장이 성공하면 cat으로 출력하시오

```
[root@Server-A log]# touch  /temp/test.txt &&  echo  HELLO >  /tmp/test.txt &&  cat  /tmp/test.txt
HELLO
```

**정리**: `&&`는 앞 명령이 성공(exit 0)했을 때만 뒤 명령을 실행하는 AND 연결로, if문 없이 조건부 실행 체인을 짧게 구성할 때 사용한다.

### || (OR) – 명령 연결 제어

- **`||`** 는 앞 명령이 실패(exit 1)했을 때만 뒤 명령을 실행하는 제어 메타문자이다.
  즉, OR 조건처럼 동작한다.

형식: `cmd1 || cmd2`

- 동작
  - cmd1 성공(exit 0) --> cmd2 실행하지 않음
  - cmd1 실패(exit 1) --> cmd2 실행됨

- OR 논리는 다음과 같은 의미를 가진다.
  - 조건1이 참이면 전체가 참
  - 조건1이 거짓이면 조건2를 확인한다
  - 쉘에서는 조건 확인 대신 "다음 명령 실행 여부"로 바뀜
  - cmd1 성공  -->  성공이므로 cmd2 불필요
  - cmd1 실패  -->  실패를 보완하기 위해 cmd2 실행

- 종료 코드 기반 동작
  - exit 0	: 성공(true)
  - exit 1 	: 실패(false)

- `||` 는 false 를 만나면 다음 명령 실행, 반대로 true 를 만나면 이어지는 OR 명령은 무시한다.

`cmd1 || cmd2 || cmd3`

- 동작 흐름 : cmd1 || cmd2 || cmd3
  - cmd1 성공  -->  cmd2, cmd3 모두 실행 안 됨
  - cmd1 실패  -->  cmd2 실행
  - cmd2 성공  -->  cmd3 실행 안 됨
  - cmd2 실패  -->  cmd3 실행됨

- 앞 명령이 실패했을 때만 다음 명령 실행
- EX) `cd abc || echo "디렉터리 생성 실패"`	: cd abc 실패하면 "디렉터리 생성 실패" 출력

EX1) 없는 디렉터리 abc로 이동 시도하고 실패하면 메시지 출력

```
[root@Server-A log]# cd  abc  ||  echo "cd command fail"
-bash: cd: abc: 그런 파일이나 디렉터리가 없습니다
cd command fail					# 메세지 출력
```

EX2) /backup/log.txt 존재하지 않으면 메시지 출력

```
[root@Server-A log]# [ -f /backup/log ] ||  echo "/backup/log 파일 없음"
/backup/log 파일 없음
```

EX3) 일반 파일이면 삭제 후 디렉터리 생성
- /test라는 이름이 일반 파일이면 삭제한 후 같은 이름의 디렉터리를 생성
- 이미 디렉터리이면 아무 작업도 하지 않아야 한다.

```
[root@Server-A log]# rm -rf /test		# /test 디렉터리 삭제

[root@Server-A log]# [ -d /test ]		# /test 디렉터리 확인

[root@Server-A log]# echo $?
1



[root@Server-A log]# [ -d /test ] || { [-f /test ] && rm -rf /test; mkdir /test; }
```

/test가 디렉터리일 경우
- 첫 번째 조건 성공
- 뒤의 명령 그룹 실행 안 함 

/test가 일반 파일일 경우
- 디렉터리 검사 실패
- 일반 파일 검사 성공
- 파일 삭제
- mkdir /test 실행 

/test가 존재하지 않음
- 디렉터리 검사 실패
- 일반 파일 검사 실패
- rm 실행 안 함
- mkdir /test 실행

```
[root@Server-A log]# ls -ld /test
drwxr-xr-x 2 root root 6  7월 23 16:48 /test
```

EX4) 디렉터리가 없으면 만들고 권한 설정
- /share 디렉터리가 없으면 생성하고, 생성 또는 기존 디렉터리 확인에 성공하면 권한을 770으로 설정
- 권한 설정에 실패하면 Permission Change Fail을 출력

```
[root@Server-A log]# { [ -d /share ] || mkdir /share; } && chmod 770  /share ||  echo "Permission Change Fail"


[root@Server-A log]# ls  -ld  /share
drwxrwx--- 2 root root 6  7월 23 16:58 /share
```

```
# { [ -d /share ] || mkdir /share; }	# or 조건이므로 둘 중 1개만 True여도 뒤의 명령어가 실행된다.
```

/share가 존재
- mkdir 실행 안 함
- chmod 770 /share 실행

/share가 없음
- mkdir /share 실행
- 생성 성공 시 chmod 실행

디렉터리 생성 또는 권한 변경 실패
- Permission Change Fail 출력

```
[root@Server-A log]# { [ -d /share ] || mkdir /share; } && chmod 779  /share ||  echo "Permission Change Fail"
chmod: invalid mode: `779'
Try 'chmod --help' for more information.
Permission Change Fail
```

- 허가권 779는 없기때문에 허가권 부여가 실패하므로 "Permission Change Fail" 메세지가 출력된다.

**정리**: `||`는 앞 명령이 실패(exit 1)했을 때만 뒤 명령을 실행하는 OR 연결로, `&&`와 조합하면 성공/실패에 따라 서로 다른 후속 동작을 한 줄로 표현할 수 있다.

### 세미콜론 vs && vs ||

| 구분 | 의미 | 사용 상황 |
|---|---|---|
| `;` | 무조건 다음 명령 실행 | 단순한 순차 실행 |
| `&&` | 앞 명령이 성공(exit 0)하면 실행 | 조건 기반 AND |
| `\|\|` | 앞 명령이 실패(exit 1) 하면 다음 명령 실행 | 조건 기반 OR(실패 시 실행) |

- `command1; command2`	# command1이 실패해도 command2 실행됨
- `command1 && command2`	# command1이 성공해야 command2 실행됨
- `command1 || command2`	# command1이 실패해야 command2 실행됨

**정리**: 세 연산자는 각각 무조건 실행(`;`), 성공 시만 실행(`&&`), 실패 시만 실행(`||`)로 구분되며, 이를 조합하면 조건문 없이도 성공/실패 분기 로직을 셸 한 줄로 표현할 수 있다.

## 8) 틸드( ~ ) 의 개념과 동작 원리

- **틸드( ~ )**
  - bash에서 현재 사용자(Home Directory) 를 의미하는 특수 기호
  - 홈 디렉터리 전체 경로를 자동으로 치환(expansion)해 준다.

EX) root 계정	: `~` 	= `/root`
EX) 사용자 sol 계정	: `~` 	= `/home/sol`

- 즉, 사용자가 달라도 항상 본인의 홈 디렉터리를 가리킨다.

**틸드 확장(Tilde Expansion)의 특징**
- 경로의 맨 앞에서만 동작한다.
- EX) `cd ~`
- EX) `ls ~/Downloads`
- EX) `touch  ~/file.txt`
- EX) `cd ~root`      	# /root
- EX) `cd ~guest`		# /home/guest

- 쉘이 명령을 실행하기 전에 미리 경로로 치환한다.
- 즉 `ls ~/Downloads` 는 실제로 = `ls /home/sol/Downloads`

```
[root@Server-A log]# cd /etc/NetworkManager/system-connections/


[root@Server-A system-connections]# pwd
/etc/NetworkManager/system-connections


[root@Server-A system-connections]# cd ~


[root@Server-A ~]# pwd
/root
```

EX1) 현재 사용자 홈 디렉터리로 이동하시오.

```
[root@Server-A system-connections]# cd ~


[root@Server-A ~]# pwd
/root
```

EX2) 홈 디렉터리에 example.txt 파일을 생성하시오.

```
[root@Server-A ~]# touch  ~/example.txt


root@Server-A ~]# ls  -l  ~/example.txt
-rw-r--r-- 1 root root 0  7월 23 17:08 /root/example.txt
```

EX3) 홈 디렉터리의 숨김 파일까지 모두 출력하시오.

```
[root@Server-A ~]# ls  -la  ~/
합계 68
dr-xr-x---. 15 root root  4096  7월 23 17:08 .
dr-xr-xr-x. 23 root root  4096  7월 23 16:58 ..
-rw-------   1 root root 16497  7월 23 14:58 .bash_history
-rw-r--r--.  1 root root    18  5월 11  2022 .bash_logout
-rw-r--r--.  1 root root   141  5월 11  2022 .bash_profile
-rw-r--r--   1 root root   449  7월 22 12:10 .bashrc
drwx------.  8 root root   108  7월  2 15:10 .cache
drwx------. 10 root root  4096  7월  2 15:10 .config
-rw-r--r--.  1 root root   100  5월 11  2022 .cshrc
-rw-------   1 root root   168  7월 22 11:41 .history
-rw-------   1 root root    20  7월 22 17:31 .lesshst
drwx------.  3 root root    19  7월  2 14:46 .local
drwx------.  2 root root     6  7월  2 15:08 .ssh
-rw-r--r--.  1 root root   129  5월 11  2022 .tcshrc
-rw-r--r--   1 root root     0  7월 22 15:11 Solbangul
-rw-r--r--   1 root root     0  7월 22 15:11 Soldesk
-rw-r--r--   1 root root     0  7월 22 15:11 Solnamu
-rw-------.  1 root root  1027  7월  2 12:55 anaconda-ks.cfg
-rw-r--r--   1 root root     0  7월 23 15:55 backup
-rw-r--r--   1 root root     0  7월 23 17:08 example.txt
-rw-r--r--   1 root root     0  7월 22 15:55 file1.txt
-rw-r--r--   1 root root     0  7월 22 15:56 file11.txt
-rw-r--r--   1 root root     0  7월 22 15:56 file12.txt
-rw-r--r--   1 root root     0  7월 22 15:56 file13.txt
-rw-r--r--   1 root root     0  7월 22 15:56 file14.txt
-rw-r--r--   1 root root     0  7월 22 15:56 file15.txt
-rw-r--r--   1 root root     0  7월 22 15:55 file3.txt
-rw-r--r--   1 root root     0  7월 22 15:55 file7.txt
drwxr-xr-x   6 root root    74  7월 22 12:46 meta_lab
-rwxr-xr-x   1 root root  3773  7월 22 12:41 meta_lab_setup.sh
-rw-r--r--   1 root root     0  7월 23 15:13 semi.txt
drwxr-xr-x.  2 root root     6  7월  2 14:46 공개
drwxr-xr-x.  2 root root     6  7월  2 14:46 다운로드
drwxr-xr-x.  2 root root     6  7월  2 14:46 문서
drwxr-xr-x.  2 root root     6  7월  2 14:46 바탕화면
drwxr-xr-x.  2 root root     6  7월  2 14:46 비디오
drwxr-xr-x.  2 root root     6  7월  2 14:46 사진
drwxr-xr-x.  2 root root     6  7월  2 14:46 서식
drwxr-xr-x.  2 root root     6  7월  2 14:46 음악
```

EX4) 홈 디렉터리 안에 backup 폴더를 생성하시오.

```
[root@Server-A ~]# mkdir  ~/backup
mkdir: `/root/backup' 디렉토리를 만들 수 없습니다: 파일이 있습니다


[root@Server-A ~]# ls  -ld  /backup
drwxr-xr-x 8 root root 4096  7월 23 16:06 /backup
```

EX5) /var/log/messages 파일을 홈 디렉터리로 복사하시오

```
[root@Server-A ~]# [ -f /var/log/messages ] &&  cp  /var/log/messages ~/  &&  ls  -l  ~/messages
-rw------- 1 root root 2099476  7월 23 17:17 /root/messages
```

EX6) 홈 디렉터리에 data1.txt, data2.txt, data3.txt 를 한번에 생성하시오.

```
[root@Server-A ~]# touch  ~/data{1..3}.txt  &&  ls -l ~/data*
-rw-r--r-- 1 root root 0  7월 23 17:20 /root/data1.txt
-rw-r--r-- 1 root root 0  7월 23 17:20 /root/data2.txt
-rw-r--r-- 1 root root 0  7월 23 17:20 /root/data3.txt
```

**정리**: 틸드(`~`)는 경로의 맨 앞에서만 홈 디렉터리로 자동 치환되는 확장 기능으로, `~`는 현재 사용자, `~계정명`은 해당 계정의 홈 디렉터리를 가리킨다.

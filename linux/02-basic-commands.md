# Linux-02 기본 명령어

## 목차

1. [리눅스 기초 명령어 (cd)](#리눅스-기초-명령어-cd)
2. [Linux 기본 명령어 (ls)](#linux-기본-명령어-ls)
3. [Linux 기본 명령어 (cp)](#linux-기본-명령어-cp)
4. [Linux 기본 명령어 (mv)](#linux-기본-명령어-mv)
5. [Linux 기본 명령어 (mkdir / rmdir)](#linux-기본-명령어-mkdir--rmdir)
6. [Linux 기본 명령어 (rm)](#linux-기본-명령어-rm)
7. [종합 실습 (mkdir / ls / cp / mv / rm)](#종합-실습-mkdir--ls--cp--mv--rm)
8. [Linux 기본 명령어 (출력 명령어)](#linux-기본-명령어-출력-명령어)
9. [Linux 기본 명령어 (cat 명령어 연산자)](#linux-기본-명령어-cat-명령어-연산자)
10. [Linux 기본 명령어 (find)](#linux-기본-명령어-find)

## 리눅스 기초 명령어 (cd)

- 관리자 계정 또는 사용자 계정을 사용하여 리눅스 서버로 접속시 기본적으로 자신의 홈 디렉터리로 접속된다. `cd`, `pwd`를 비롯한 기본 명령어들은 디렉터리 이동, 파일 확인, 로그 점검 등 서버 관리 현장에서 매일같이 사용된다.

- **pwd** : 현재 자신의 위치를 출력

- **cd**
  - 디렉터리를 이동하는 명령어
  - 형식: `cd (경로)`
  - 이동 방식은 절대 경로방식과 상대 경로 방식으로 사용하여 이동이 가능하다.

**절대경로**
- 최상위 디렉터리부터 이동할 경로까지의 경로를 순대로 입력하여 이동하는 방식

**상대경로**
- 현재 자신의 디렉터리로부터 목적지까지 입력하여 이동하는 방식
- `.` : 현재 자신의 디렉터리
- `..` : 현재 위치에서 자신의 부모 디렉터리

---

### 실습 환경 준비

```bash
# 서버 이름 변경
hostnamectl set-hostname Server-A
exec bash

# 폴더 생성
mkdir -p /soldesk/linux/rocky/version9 /sk/sktel/sales/1team /lg/uplus/display

# 디렉터리 확인
[root@Server-A /]# ls -l /
합계 28
dr-xr-xr-x.   2 root root    6 11월  3  2024 afs
lrwxrwxrwx.   1 root root    7 11월  3  2024 bin -> usr/bin
dr-xr-xr-x.   5 root root 4096  7월  2 19:41 boot
drwxr-xr-x   18 root root 3240  7월  3 10:42 dev
drwxr-xr-x. 132 root root 8192  7월  3 11:31 etc
drwxr-xr-x.   3 root root   19 11월  3  2024 home
drwxr-xr-x    3 root root   19  7월  3 11:35 lg
lrwxrwxrwx.   1 root root    7 11월  3  2024 lib -> usr/lib
lrwxrwxrwx.   1 root root    9 11월  3  2024 lib64 -> usr/lib64
drwxr-xr-x.   2 root root    6 11월  3  2024 media
drwxr-xr-x.   3 root root   18 11월  3  2024 mnt
drwxr-xr-x.   2 root root    6 11월  3  2024 opt
dr-xr-xr-x  367 root root    0  7월  3 10:42 proc
dr-xr-x---.  15 root root 4096  7월  3 11:33 root
drwxr-xr-x   45 root root 1280  7월  3 10:47 run
lrwxrwxrwx.   1 root root    8 11월  3  2024 sbin -> usr/sbin
drwxr-xr-x    3 root root   19  7월  3 11:35 sk
drwxr-xr-x    3 root root   19  7월  3 11:35 soldesk
drwxr-xr-x.   2 root root    6 11월  3  2024 srv
dr-xr-xr-x   13 root root    0  7월  3 10:42 sys
drwxrwxrwt.  24 root root 4096  7월  3 11:33 tmp
drwxr-xr-x.  12 root root  144  7월  2 19:35 usr
drwxr-xr-x.  20 root root 4096  7월  2 19:41 var
```

```bash
[root@Server-A version9]# ls -R /lg
/lg:
uplus

/lg/uplus:
display

/lg/uplus/display:

[root@Server-A version9]# ls -R /sk
/sk:
sktel

/sk/sktel:
sales

/sk/sktel/sales:
1team

/sk/sktel/sales/1team:

[root@Server-A version9]# ls -R /soldesk
/soldesk:
linux

/soldesk/linux:
rocky

/soldesk/linux/rocky:
version9

/soldesk/linux/rocky/version9:


[root@Server-A rocky]# tree /soldesk/
/soldesk/
└── linux
    └── rocky
        └── version9

3 directories, 0 files

[root@Server-A rocky]# tree /lg/
/lg/
└── uplus
    └── display

2 directories, 0 files

[root@Server-A rocky]# tree /sk/
/sk/
└── sktel
    └── sales
        └── 1team

3 directories, 0 files
```

---

### EX1-1) Root계정을 사용하여 최상위 디렉터리의 'home' 디렉터리로 이동

```bash
[root@localhost ~]# cd  /home

[root@localhost home]# pwd
/home           <---- 최상위 디렉터리 하위의 'home' 디렉터리로 이동
```

### EX1-2) Root계정을 사용하여 'guest' 계정의 홈 디렉터리 안으로 이동해야한다.

```bash
[root@localhost home]# cd  /home/guest/

[root@localhost guest]# pwd
/home/guest         <---- '/home/guest' 디렉터리로 이동
```

```
      cd           /        home       /
    -----        ----     -------     ---
   경로이동   최상위 디렉터리  이동 경로    경로 (두번째부터의 '/'는 디렉터리를 의미한다.)
```

### EX2-1) 절대 경로를 사용해서 version9 디렉터리로 이동

```bash
[root@Server-A ~]# pwd
/root

cd /soldesk/linux/rocky/version9

[root@Server-A version9]# pwd
/soldesk/linux/rocky/version9
```

### EX2-2) 절대경로를 사용하여 1team 디렉터리로 이동

```bash
cd /sk/sktel/sales/1team
```

### EX3-1) 절대경로를 사용하여 rocky 디렉터리로 이동 후, 상대경로로 이동

```bash
# 절대경로를 사용하여 rocky 디렉터리로 이동
cd /soldesk/linux/rocky/

# 상대경로를 사용하여 soldesk 디렉터리로
[root@Server-A rocky]# ls ../../
linux
[root@Server-A rocky]# cd ../../
[root@Server-A soldesk]# pwd
/soldesk

# 상대경로를 사용하여 version9 디렉터리로 이동
[root@Server-A soldesk]# cd ./linux/rocky/
[root@Server-A rocky]# pwd
/soldesk/linux/rocky
```

### EX3-2) 상대 경로를 사용해서 디렉터리로 이동

```bash
# 절대 경로를 사용해서 display 디렉터리로 이동
[root@Server-A ~]# cd /lg/uplus/display
[root@Server-A display]# pwd
/lg/uplus/display

# 상대 경로를 사용해서 linux 디렉터리로 이동
[root@Server-A display]# cd ../../../soldesk/linux/
[root@Server-A linux]# pwd
/soldesk/linux

# 상대 경로를 사용해서 1team 디렉터리로 이동
[root@Server-A linux]# ls  ../../../sk/
sktel
[root@Server-A linux]# ls  ../../../sk/sktel/
sales
[root@Server-A linux]# ls  ../../../sk/sktel/sales/
1team
[root@Server-A linux]# cd  ../../../sk/sktel/sales/
[root@Server-A sales]# pwd
/sk/sktel/sales
[root@Server-A sales]# cd  ../../../sk/sktel/sales/1team/
[root@Server-A 1team]# pwd
/sk/sktel/sales/1team
```

**정리**: `cd`는 절대경로와 상대경로 두 방식 모두로 디렉터리를 이동할 수 있으며, `.`과 `..`을 활용한 상대경로 이동을 능숙히 다루는 것이 실무에서 핵심이다.

---

## Linux 기본 명령어 (ls)

- **ls (list)**
  - 지정한 디렉터리의 목록(파일, 디렉터리)을 확인하는 명령어
  - `ls` : 현재 위치 안의 파일 또는 디렉터리를 출력
  - `ls -l` : 현재 위치 안의 파일 또는 디렉터리를 자세히 출력
  - `ls -a` : 현재 위치 안의 파일 또는 디렉터리를 출력(숨김폴더 포함)
  - `ls -h` : 현재 위치 안의 파일 또는 디렉터리를 출력시 사람이 보기편하도록 출력
  - `ls -S` : 현재 위치 안의 파일 또는 디렉터리를 출력시 크기 순으로 출력
  - `ls -s` : 현재 위치 안의 파일 또는 디렉터리를 출력시 a~z순 출력
  - `ls -r` : 현재 위치 안의 파일 또는 디렉터리를 출력시 역순 출력
  - `ls -t` : 현재 위치 안의 파일 또는 디렉터리를 생성 수정이 최신 순으로 정렬
  - `ls -R` : 현재 위치 안의 파일 또는 디렉터리를 하위 디렉터리 포함 출력

```bash
[root@Server-A ~]# ls
anaconda-ks.cfg  공개  다운로드  문서  바탕화면  비디오  사진  서식  음악

[root@Server-A ~]# ls -l
합계 4
-rw-------. 1 root root 1027  7월  2 19:11 anaconda-ks.cfg
drwxr-xr-x. 2 root root    6  7월  2 19:15 공개
drwxr-xr-x. 2 root root    6  7월  2 19:15 다운로드
drwxr-xr-x. 2 root root    6  7월  2 19:15 문서
drwxr-xr-x. 2 root root    6  7월  2 19:15 바탕화면
drwxr-xr-x. 2 root root    6  7월  2 19:15 비디오
drwxr-xr-x. 2 root root    6  7월  2 19:15 사진
drwxr-xr-x. 2 root root    6  7월  2 19:15 서식
drwxr-xr-x. 2 root root    6  7월  2 19:15 음악
rw-------.1   root root   1027   7월  2 19:11     anaconda-ks.cfg
   허가권        소유권     크기   생성, 수정,시간      파일 디렉터리명

[root@Server-A ~]# ls -a
.   .Xauthority    .bash_logout   .bashrc  .config  .local    .ssh     .viminfo         공개      문서      비디오  서식
..  .bash_history  .bash_profile  .cache   .cshrc   .mozilla  .tcshrc  anaconda-ks.cfg  다운로드  바탕화면  사진    음악

[root@Server-A ~]# ls -h
anaconda-ks.cfg  공개  다운로드  문서  바탕화면  비디오  사진  서식  음악
```

### ls 명령어 실습 예제

```bash
# root 계정을 사용하여 /home/guest 디렉터리로 상대경로를 사용하여 이동하시오
# 해당 디렉터리안의 정보와 모든 디렉터리 및 파일 정보를 확인해야 한다

[root@Server-A ~]# cd ../home/guest
[root@Server-A guest]# ls -alh
합계 24K
drwx------. 14 guest guest 4.0K  7월  3 09:48 .
drwxr-xr-x.  3 root  root    19 11월  3  2024 ..
-rw-------   1 guest guest    6  7월  3 09:48 .bash_history
-rw-r--r--.  1 guest guest   18  5월 16  2022 .bash_logout
-rw-r--r--.  1 guest guest  141  5월 16  2022 .bash_profile
-rw-r--r--.  1 guest guest  492  5월 16  2022 .bashrc
drwx------. 10 guest guest 4.0K  7월  2 19:14 .cache
drwxr-xr-x.  9 guest guest  190  7월  2 19:15 .config
drwx------.  4 guest guest   32  7월  2 19:14 .local
drwxr-xr-x.  6 guest guest   81  7월  2 19:14 .mozilla
drwxr-xr-x.  2 guest guest    6  7월  2 19:14 공개
drwxr-xr-x.  2 guest guest    6  7월  2 19:14 다운로드
drwxr-xr-x.  2 guest guest    6  7월  2 19:14 문서
drwxr-xr-x.  2 guest guest    6  7월  2 19:14 바탕화면
drwxr-xr-x.  2 guest guest    6  7월  2 19:14 비디오
drwxr-xr-x.  2 guest guest    6  7월  2 19:14 사진
drwxr-xr-x.  2 guest guest    6  7월  2 19:14 서식
drwxr-xr-x.  2 guest guest    6  7월  2 19:14 음악
```

```bash
# root의 홈 디렉터리에서 /sk/sktel/sales 디렉터리 안의 정보와 그 하위의 모든 디렉터리 및 파일정보를 확인해야 한다
[root@Server-A guest]# ll -al  ../../sk/sktel/sales/
합계 0
drwxr-xr-x 3 root root 19  7월  3 11:35 .
drwxr-xr-x 3 root root 19  7월  3 11:35 ..
drwxr-xr-x 2 root root  6  7월  3 11:35 1team

# root의 홈 디렉터리에서 /soldesk/linux/rocky 디렉터리 안의 정보와 그 하위의 모든 디렉터리 및 파일정보를 확인해야 한다
[root@Server-A ~]# ll -al /soldesk/linux/rocky/
합계 0
drwxr-xr-x 3 root root 22  7월  3 11:35 .
drwxr-xr-x 3 root root 19  7월  3 11:35 ..
drwxr-xr-x 2 root root  6  7월  3 11:35 version9

# root의 /soldesk/linux/rocky에서 /sk/sktel/sales 하위 정보 확인
[root@Server-A rocky]# ll -ahR  ../../../sk/sktel/sales/
../../../sk/sktel/sales/:
합계 0
drwxr-xr-x 3 root root 19  7월  3 11:35 .
drwxr-xr-x 3 root root 19  7월  3 11:35 ..
drwxr-xr-x 2 root root  6  7월  3 11:35 1team

../../../sk/sktel/sales/1team:
합계 0
drwxr-xr-x 2 root root  6  7월  3 11:35 .
drwxr-xr-x 3 root root 19  7월  3 11:35 ..
```

**정리**: **ls**의 각종 옵션(`-l`, `-a`, `-h`, `-S`, `-R` 등)을 조합하면 파일 크기, 권한, 숨김 파일, 하위 디렉터리 구조까지 원하는 형태로 목록을 확인할 수 있다.

---

## Linux 기본 명령어 (cp)

- **cp (copy)**
  - 파일 또는 디렉터리를 복사하는 명령어
  - 형식: `cp [옵션] [원본 경로] [원본 파일 또는 디렉터리명] [복사될 경로] [복사될 파일 또는 디렉터리명]`

```bash
[root@Server-A rocky]# mkdir  /backup

[root@Server-A rocky]# cp  /etc/resolv.conf  /backup
[root@Server-A rocky]# cp  /etc/login.defs  /backup
[root@Server-A rocky]# cp  /etc/passwd  /backup
[root@Server-A rocky]# cp  /etc/inittab  /backup
[root@Server-A rocky]# cp  /etc/group  /backup


[root@Server-A rocky]# ls  -l  /backup
합계 24
-rw-r--r-- 1 root root  815  7월  3 14:34 group
-rw-r--r-- 1 root root  490  7월  3 14:34 inittab
-rw-r--r-- 1 root root 7779  7월  3 14:34 login.defs
-rw-r--r-- 1 root root 2124  7월  3 14:34 passwd
-rw-r--r-- 1 root root   54  7월  3 14:34 resolv.conf
```

### 1) 원본 파일명을 그대로 사용해서 복사

```bash
# EX1) '/backup' 디렉터리의 'login.defs'파일을  '/home/guest' 디렉터리로 복사해야한다.

[guest@Server-A ~]$ cp  /backup/login.defs   ./login.defs          # 상대경로

[guest@Server-A ~]$ cp  /backup/login.defs   /home/guest/login.defs    # 절대경로

[guest@Server-A ~]$ ls  -l
합계 8
-rw-r--r--  1 guest guest 7779  7월  3 14:39 login.defs
drwxr-xr-x. 2 guest guest     6  7월  2 14:38 공개
drwxr-xr-x. 2 guest guest     6  7월  2 14:38 다운로드
drwxr-xr-x. 2 guest guest     6  7월  2 14:38 문서
drwxr-xr-x. 2 guest guest     6  7월  2 14:38 바탕화면
drwxr-xr-x. 2 guest guest     6  7월  2 14:38 비디오
drwxr-xr-x. 2 guest guest     6  7월  2 14:38 사진
drwxr-xr-x. 2 guest guest     6  7월  2 14:38 서식
drwxr-xr-x. 2 guest guest     6  7월  2 14:38 음악
```

### 2) 원본 파일명을 생략해서 복사

```bash
# EX2) '/backup' 디렉터리의 'inittab'파일을  '/home/guest' 디렉터리로 복사해야한다.

[guest@Server-A ~]$ cp  /backup/inittab   ./

[guest@Server-A ~]$ ll
합계 12
-rw-r--r--  1 guest guest  490  7월  3 14:42 inittab
-rw-r--r--  1 guest guest 7779  7월  3 14:39 login.defs
drwxr-xr-x. 2 guest guest     6  7월  2 14:38 공개
```

### 3) 파일 이름을 변경하여 복사

```bash
# EX3) '/backup' 디렉터리의 'passwd' 파일을 '/home/guest' 디렉터리로 'passwd2' 이름으로 복사해야한다.

[guest@Server-A ~]$ cp  /backup/passwd   ./passwd2

[guest@Server-A ~]$ ls  -l
합계 20
-rw-r--r--  1 guest guest  490  7월  3 14:42 inittab
-rw-r--r--  1 guest guest 7779  7월  3 14:39 login.defs
-rw-r--r--  1 guest guest 2124  7월  3 14:44 passwd2
```

### 4) 원본 속성(권한/소유자/시간) 유지하여 복사 (-p 옵션)

```bash
# EX4) '/etc/resolv.conf' 파일을 '/home/guest' 디렉터리로 복사해야한다.
#       단, 파일의 원본 속성(접근시간, 수정시간, 소유자, 허가권)이 변경되지 않아야 한다.

[guest@Server-A ~]$ cp  -p   /backup/resolv.conf  ./

[guest@Server-A ~]$ ls  -l  /backup/resolv.conf  ./resolv.conf
-rw-r--r--  1 guest guest 54  7월  3 14:34 ./resolv.conf   # 소유주가 guest이나 시간과 권한이 동일
-rw-r--r--. 1 root  root  54  7월  3 14:34 /backup/resolv.conf
```

### 5) 복사 대상이 원본보다 최신이면 덮어쓰지 않음 (-u 옵션)

```bash
# -u : 원본 파일이 복사될 파일보다 최신인 경우에만 덮어쓴다.

[guest@Server-A ~]$ cp  -u   /backup/passwd  ./passwd2
```

### 6) 복사 전 덮어쓰기 확인 (-i 옵션)

```bash
# -i : 이미 존재하는 파일을 덮어쓸 때 확인 메시지를 출력한다.

[guest@Server-A ~]$ cp  -i  /backup/passwd  ./passwd2
cp: `./passwd2'를 overwrite 하시겠습니까? y
```

### 7) 디렉터리를 포함하여 복사 (-r 옵션)

```bash
# EX7) '/backup' 디렉터리를 '/home/guest' 디렉터리로 복사해야한다.
#       '/backup' 디렉터리 안에 있는 모든 파일 및 디렉터리를 포함하여 복사해야한다.

[root@Server-A ~]# cp  -r   /backup  /home/guest

[root@Server-A ~]# ls  -l  /home/guest
합계 16
drwxr-xr-x  2 root  root  4096  7월  3 14:58 backup    # 복사된 backup 디렉터리
-rw-r--r--  1 guest guest  490  7월  3 14:42 inittab
-rw-r--r--  1 guest guest 7779  7월  3 14:39 login.defs
-rw-r--r--  1 guest guest 2124  7월  3 14:44 passwd2
-rw-r--r--  1 guest guest   54  7월  3 14:34 resolv.conf
```

### 8) 와일드카드를 사용한 복사

```bash
# EX8-1) '/backup' 디렉터리에서 'g'로 시작하는 파일들을 '/sol' 디렉터리로 복사
[root@Server-A ~]# cp  /backup/g*   /sol

# EX8-2) '/backup' 디렉터리에서 'a'로 시작하는 파일들을 'a1' 디렉터리로 복사
[root@Server-A ~]# cp  /backup/a*   /sol/A-class/a1

# EX8-3) '/etc' 디렉터리에서 확장자가 .conf이면서 파일 이름이 정확히 6글자인 파일만 복사
[root@Server-A ~]# cp  /etc/??????.conf  /home/guest/work/c
```

**정리**: **cp**는 `-p`(속성 유지), `-u`(최신 파일만 복사), `-i`(덮어쓰기 확인), `-r`(디렉터리 포함) 옵션과 와일드카드를 조합해 다양한 복사 시나리오를 처리한다.

---

## Linux 기본 명령어 (mv)

- **mv (move)**
  - 파일 또는 디렉터리를 이동하거나, 이름을 변경하는 명령어
  - 형식: `mv [옵션] [원본 경로/파일명] [이동될 경로/파일명]`

```bash
# EX1) '/backup' 디렉터리의 'resolv.conf' 파일을 '/home/guest' 디렉터리로 이동해야한다.

[guest@Server-A ~]$ mv  /backup/resolv.conf  /home/guest

[guest@Server-A ~]$ ls  -l  /backup
합계 20
-rw-r--r-- 1 root root  815  7월  3 14:34 group
-rw-r--r-- 1 root root  490  7월  3 14:34 inittab
-rw-r--r-- 1 root root 7779  7월  3 14:34 login.defs
-rw-r--r-- 1 root root 2124  7월  3 14:34 passwd
# resolv.conf 파일이 /backup 디렉터리에서 삭제됨

[guest@Server-A ~]$ ls  -l  /home/guest
합계 20
-rw-r--r-- 1 guest guest  490  7월  3 14:42 inittab
-rw-r--r-- 1 guest guest 7779  7월  3 14:39 login.defs
-rw-r--r-- 1 guest guest 2124  7월  3 14:44 passwd2
-rw-r--r-- 1 root  root    54  7월  3 14:34 resolv.conf
```

```bash
# EX2) 파일 이름을 변경 (같은 디렉터리 내에서 이동)

[guest@Server-A ~]$ mv  ./passwd2  ./passwd-change

[guest@Server-A ~]$ ls  -l
합계 20
-rw-r--r--  1 guest guest  490  7월  3 14:42 inittab
-rw-r--r--  1 guest guest 7779  7월  3 14:39 login.defs
-rw-r--r--  1 guest guest 2124  7월  3 14:44 passwd-change    # 이름 변경
-rw-r--r--  1 root  root    54  7월  3 14:34 resolv.conf
```

```bash
# EX3) 디렉터리 이동

[root@Server-A ~]# mv  /home/guest/backup  /home/guest/bak

[root@Server-A ~]# ls  -l  /home/guest
합계 16
drwxr-xr-x  2 root  root  4096  7월  3 14:58 bak    # 이름 변경
```

### mv 옵션

```bash
# -i : 이미 존재하는 파일을 덮어쓸 때 확인 메시지를 출력
[guest@Server-A ~]$ mv  -i  ./resolv.conf  ./inittab

# -u : 원본 파일이 이동될 파일보다 최신인 경우에만 이동
[guest@Server-A ~]$ mv  -u  /backup/group  ./
```

**정리**: **mv**는 이동과 이름 변경을 동시에 수행하는 명령어이며, `-i`/`-u` 옵션으로 안전하게 덮어쓰기를 제어할 수 있다.

---

## Linux 기본 명령어 (mkdir / rmdir)

- **mkdir**
  - 디렉터리를 생성하는 명령어
  - 형식: `mkdir [옵션] [경로/디렉터리명]`

- **rmdir**
  - 디렉터리를 삭제하는 명령어
  - 형식: `rmdir [옵션] [경로/디렉터리명]`
  - rmdir은 디렉터리안에 파일 및 디렉터리가 있으면 해당 디렉터리를 삭제할수 없다. (일반적으로 사용하지 않는다.)
  - 일반적으로 rm 명령어를 사용해서 삭제한다.

```bash
[root@Server-A ~]# rmdir  /sk
rmdir: failed to remove '/sk': 디렉터리가 비어있지 않음

[root@Server-A ~]# rm  -rf  /sk
```

### EX1-1) 최상위 디렉터리에 'sk' 디렉터리를 생성하시오

```bash
mkdir /sk/
```

### EX1-2) sk 하위 디렉터리 생성

```bash
# 방법 1
[root@Server-A ~]# mkdir  /sk/sk-energy
[root@Server-A ~]# mkdir  /sk/sk-telecom
[root@Server-A ~]# mkdir  /sk/sk-networks

# 방법 2
[root@Server-A ~]# mkdir  /sk/sk-energy  /sk/sk-telecom  /sk/sk-networks

# 방법 3
[root@Server-A ~]# cd  /sk
[root@Server-A sk]# mkdir  sk-energy  sk-telecom  sk-networks


[root@Server-A ~]# ls  -lR  /sk
/sk:
합계 0
drwxr-xr-x 2 root root 6  7월  6 09:49 sk-energy
drwxr-xr-x 2 root root 6  7월  6 09:49 sk-networks
drwxr-xr-x 2 root root 6  7월  6 09:49 sk-telecom

/sk/sk-energy:
합계 0

/sk/sk-networks:
합계 0

/sk/sk-telecom:
합계 0
```

### EX2-1) sk-energy 하위에 sk-e-a1 생성

```bash
[root@Server-A ~]# mkdir  /sk/sk-energy/sk-e-a1

[root@Server-A ~]# ls  -lR  /sk
/sk:
합계 0
drwxr-xr-x 3 root root 21  7월  6 09:52 sk-energy
drwxr-xr-x 2 root root  6  7월  6 09:49 sk-networks
drwxr-xr-x 2 root root  6  7월  6 09:49 sk-telecom

/sk/sk-energy:
합계 0
drwxr-xr-x 2 root root 6  7월  6 09:52 sk-e-a1

/sk/sk-energy/sk-e-a1:
합계 0

/sk/sk-networks:
합계 0

/sk/sk-telecom:
합계 0
```

### EX2-2) -p 옵션으로 중간 디렉터리 포함 생성

```bash
[root@Server-A ~]# mkdir  /sk/sk-networks/sk-net-a1/sk-net-b1
mkdir: `/sk/sk-networks/sk-net-a1/sk-net-b1' 디렉토리를 만들 수 없습니다: 그런 파일이나 디렉터리가 없습니다

# mkdir 명령어는 가장 마지막 디렉터리를 생성한다. (sk-net-b1을 생성한다.)
# 하지만 경로상에 sk-net-a1 디렉터리가 없기때문에 sk-net-b1을 생성할 수 없다.

# 방법 1
[root@Server-A ~]# mkdir  /sk/sk-networks/sk-net-a1
[root@Server-A ~]# mkdir  /sk/sk-networks/sk-net-a1/sk-net-b1

# 방법 2
[root@Server-A ~]# mkdir  -p  /sk/sk-networks/sk-net-a1/sk-net-b1
```

**정리**: **mkdir -p**는 중간 경로 디렉터리까지 한 번에 생성해주며, **rmdir**은 빈 디렉터리만 삭제 가능하므로 실무에서는 주로 `rm -r` 계열을 사용한다.

---

## Linux 기본 명령어 (rm)

- **rm (remove)**
  - 파일 또는 디렉터리를 삭제하는 명령어
  - Linux는 삭제된 파일이 휴지통으로 가지 않고 바로 삭제되기 때문에 복구가 어렵다.
    따라서 삭제 시 매우 신중해야 하며, 어떤 파일을 삭제하는지 어디 위치에서 삭제하는지를 항상 확인해야 한다.
  - 파일 삭제시 삭제 유/무를 확인하지 않고 바로 삭제하기위해서는 `-f` 옵션을 사용해야한다.
  - 형식: `rm [option] [경로/파일 또는 디렉터리명]`

### EX1) 종합 실습

```bash
# EX1) 아래의 조건에 맞게 설정하시오
# # 최상위 디렉터리에 'backup' 디렉터리를 생성하시오
# # '/etc' 디렉터리의 모든 파일을 '/backup' 디렉터리로 복사해야한다.
# # 최상위 디렉터리에 '/sol' 디렉터리를 생성하시오
# # '/sol' 디렉터리에 'A-class' 디렉터리를 생성하시오
# # '/sol' 디렉터리에 'B-class' 디렉터리를 생성하시오
# # 'A-class' 디렉터리안에 'a1/a2/a3/a4' 디렉터리를 생성하시오
# # 'B-class' 디렉터리안에 'b1/b2/b3/b4' 디렉터리를 생성하시오
# # '/backup' 디렉터리에서 'g'문자열로 시작하는 파일들을 '/sol' 디렉터리로 복사해야한다.
# # '/backup' 디렉터리에서 'a'문자열로 시작하는 파일들을 'a1' 디렉터리로 복사해야한다.
# # '/backup' 디렉터리에서 'b'문자열로 시작하는 파일들을 'a2' 디렉터리로 복사해야한다.
# # '/backup' 디렉터리에서 'c'문자열로 시작하는 파일들을 'a3' 디렉터리로 복사해야한다.

[root@Server-A ~]# rm  -rf  /backup

[root@Server-A ~]# mkdir  /backup

[root@Server-A ~]# cp  /etc/*   /backup

[root@Server-A ~]# mkdir  -p   /sol/A-class/a1/a2/a3/a4   /sol/B-class/b1/b2/b3/b4

[root@Server-A ~]# cp  /backup/g*   /sol

[root@Server-A ~]# ls  -l  /sol
합계 24
drwxr-xr-x 3 root root   16   7월  6 10:33 A-class
drwxr-xr-x 3 root root   16   7월  6 10:33 B-class
-rw-r--r-- 1 root root  815   7월  6 10:36 group
-rw-r--r-- 1 root root  841   7월  6 10:36 group-
-rw------- 1 root root 7009  7월  6 10:36 grub2.cfg
---------- 1 root root  654   7월  6 10:36 gshadow
---------- 1 root root  677   7월  6 10:36 gshadow-

[root@Server-A ~]# cp  /backup/a*   /sol/A-class/a1

[root@Server-A ~]# ls  -l  /sol/A-class/a1
합계 28
drwxr-xr-x 3 root root   16  7월  6 10:33 a2
-rw-r--r-- 1 root root   16  7월  6 10:37 adjtime
-rw-r--r-- 1 root root 1529  7월  6 10:37 aliases
-rw-r--r-- 1 root root  541  7월  6 10:37 anacrontab
-rw-r--r-- 1 root root  269  7월  6 10:37 anthy-unicode.conf
-rw-r--r-- 1 root root  833  7월  6 10:37 appstream.conf
-rw-r--r-- 1 root root   55  7월  6 10:37 asound.conf
-rw-r--r-- 1 root root    1  7월  6 10:37 at.deny

[root@Server-A ~]# cp  /backup/b*   /sol/A-class/a1/a2

[root@Server-A ~]# cp  /backup/c*   /sol/A-class/a1/a2/a3
```

### EX1-2) 파일 삭제

```bash
[root@Server-A ~]#  rm  /sol/group
rm: remove 일반 파일 '/sol/group'? y

[root@Server-A ~]# ls  -l /sol
합계 20
drwxr-xr-x 3 root root   16   7월  6 10:33 A-class
drwxr-xr-x 3 root root   16   7월  6 10:33 B-class
-rw-r--r-- 1 root root  841   7월  6 10:36 group-
-rw------- 1 root root 7009  7월  6 10:36 grub2.cfg
---------- 1 root root  654   7월  6 10:36 gshadow
---------- 1 root root  677   7월  6 10:36 gshadow-
```

### EX1-3) 여러 파일 동시 삭제

```bash
[root@Server-A ~]# rm  /sol/grub2.cfg  /sol/gshadow
rm: remove 일반 파일 '/sol/grub2.cfg'? y
rm: remove 일반 파일 '/sol/gshadow'? y
```

### EX1-4) -f 옵션 (확인 없이 강제 삭제)

```bash
[root@Server-A ~]# rm  -f  /sol/group-

# -f (force)
#  파일을 삭제할지 유/무를 묻지 않고 삭제한다.
#  에러/경고 메세지를 무시하고 강제로 삭제한다.
```

### EX1-5) 존재하지 않는 파일 삭제 시도

```bash
[root@Server-A ~]# rm  -f  /sol/ABCD
# -f 옵션 사용시 에러 메세지 없이 종료

[root@Server-A ~]# rm   /sol/ABCD
rm: cannot remove '/sol/ABCD': 그런 파일이나 디렉터리가 없습니다
```

### EX2-1) 디렉터리 삭제 (-r 옵션)

```bash
[root@Server-A ~]# rm  /sol/B-class/b1/b2/b3/b4
rm: cannot remove '/sol/B-class/b1/b2/b3/b4': 디렉터리입니다    # rm 명령어는 기본적으로 파일만 삭제한다.

[root@Server-A ~]# rm  -r  /sol/B-class/b1/b2/b3/b4     # 디렉터리 삭제시 -r 옵션을 사용해야 한다.
rm: remove 디렉토리 '/sol/B-class/b1/b2/b3/b4'? y
```

### EX2-2) 여러 디렉터리 삭제

```bash
# 방법 1
[root@Server-A ~]# rm  -r  /sol/B-class/b1/b2/b3
[root@Server-A ~]# rm  -r  /sol/B-class/b1/b2
[root@Server-A ~]# rm  -r  /sol/B-class/b1

# 방법 2
[root@Server-A ~]# rm  -r  /sol/B-class/b1
rm: descend into directory '/sol/B-class/b1'? y
rm: descend into directory '/sol/B-class/b1/b2'? y
rm: remove 디렉토리 '/sol/B-class/b1/b2/b3'? y
rm: remove 디렉토리 '/sol/B-class/b1/b2'? y
rm: remove 디렉토리 '/sol/B-class/b1'? y

# 방법 3
[root@Server-A ~]# rm  -rf  /sol/B-class/b1
```

### EX2-4) 디렉터리 안의 파일만 삭제 (와일드카드)

```bash
[root@Server-A ~]# rm  /sol/A-class/a1/a2/*
rm: cannot remove '/sol/A-class/a1/a2/a3': 디렉터리입니다
rm: remove 일반 파일 '/sol/A-class/a1/a2/bashrc'? y
rm: remove 일반 파일 '/sol/A-class/a1/a2/bindresvport.blacklist'? y
rm: remove 일반 파일 '/sol/A-class/a1/a2/brlapi.key'? y
rm: remove 일반 파일 '/sol/A-class/a1/a2/brltty.conf'? y
```

### EX2-5) -f 옵션과 와일드카드로 디렉터리 안 파일 일괄 삭제

```bash
# 파일 및 디렉터리 삭제시 삭제의 확인 유/무를 묻지않고 삭제해야한다.
[root@Server-A ~]# rm  -f  /sol/A-class/a1/a2/a3/*
```

**정리**: **rm**은 삭제 후 복구가 불가능하므로 `-i`로 확인하며 삭제하는 습관이 중요하고, 디렉터리 삭제 시에는 반드시 `-r` 옵션이 필요하다.

---

## 종합 실습 (mkdir / ls / cp / mv / rm)

### 실습 환경 준비

```bash
# root 계정

# 연습용 상위 디렉터리 생성
mkdir -p /lab/linux/projectA/src
mkdir -p /lab/linux/projectA/conf
mkdir -p /lab/linux/projectB/logs
mkdir -p /lab/backup
mkdir -p /lab/users/guest1

# 실제 시스템 파일을 연습용으로 복사
cp /etc/passwd      /lab/backup/passwd.orig
cp /etc/group       /lab/backup/group.orig
cp /etc/login.defs  /lab/backup/login.defs.orig
cp /etc/resolv.conf /lab/backup/resolv.conf.orig

# guest 계정 홈 디렉터리에 연습용 디렉터리
mkdir -p /home/guest/work/a
mkdir -p /home/guest/work/b
mkdir -p /home/guest/work/c/sub1
```

### 문제 1) 절대 경로로 디렉터리 이동

```bash
[root@Server-A linux]# cd /home/guest/
[root@Server-A guest]# cd /lab/linux/projectA/src
```

### 문제 2) 상대 경로로 이동

```bash
cd /lab/linux/projectA/src
cd ../../projectB/logs
```

### 문제 3) 상대 경로로 이동 (긴 경로)

```bash
cd /lab/linux/projectB/logs
cd ../../../home/guest/work/c/sub1
# 또는
cd ~/work/c/sub1
```

### 문제 4) 현재 디렉터리와 상위 디렉터리 동시 확인

```bash
cd /home/guest/work
ls -l . ..
```

```bash
[root@Server-A work]# ll -a
합계 4
drwxr-xr-x   5 root  root    33  7월  6 11:25 .
drwx------. 17 guest guest 4096  7월  6 11:25 ..
drwxr-xr-x   2 root  root     6  7월  6 11:25 a
drwxr-xr-x   2 root  root     6  7월  6 11:25 b
drwxr-xr-x   3 root  root    18  7월  6 11:25 c
```

### 문제 5) 하위 디렉터리 전체 구조 확인

```bash
ls -lR /lab
```

### 문제 6) 한 번의 명령으로 중첩 디렉터리 생성

```bash
cd ~
mkdir -p app/logs/2024/jan
```

### 문제 7) 빈 디렉터리 생성 후 삭제

```bash
mkdir ~/app/logs/2024/jan/tmp
rmdir ~/app/logs/2024/jan/tmp
```

### 문제 8) 상대 경로만 사용하여 디렉터리 생성

```bash
cd /home/guest/work/a
mkdir -p ../archive/2024/feb
```

### 문제 9) 전체 디렉터리 삭제

```bash
rm -rf ~/app
```

### 문제 10) 파일 복사 (절대경로 원본, 상대경로 목적지)

```bash
cp /lab/backup/passwd.orig  ./
```

```bash
[root@Server-A guest]# ll
합계 8
drwxr-xr-x  2 root  root     6  7월  6 11:49 app
-rw-r--r--  1 root  root  2124  7월  6 11:51 passwd.orig
drwxr-xr-x  2 guest guest   38  7월  3 16:44 sol
drwxr-xr-x  2 guest guest 4096  7월  3 16:42 test
drwxr-xr-x  6 root  root    48  7월  6 11:47 work
drwxr-xr-x. 2 guest guest    6  7월  2 19:14 공개
```

### 문제 11) 같은 이름으로 한 번 더 복사

```bash
cp ./passwd.orig  ./passwd.copy
```

### 문제 12) 원본 속성 유지 복사

```bash
cp -p /lab/backup/group.orig  /home/guest/work/a/
```

### 문제 13) 최신인 경우에만 덮어쓰기 + 속성 유지

```bash
cp -pu /lab/backup/login.defs.orig  ./login.defs
```

### 문제 14) 두 디렉터리에 동시 복사

```bash
cp /lab/backup/resolv.conf.orig ./a/  &&  cp /lab/backup/resolv.conf.orig ./b/
```

### 문제 15) 최신인 경우에만 덮어쓰기

```bash
cd /home/guest/work/a
cp -u /etc/passwd ./passwd.lab
```

```bash
[root@Server-A a]# ll
합계 8
-rw-r--r-- 1 root root  815  7월  6 11:28 group.orig
-rw-r--r-- 1 root root 2124  7월  6 12:00 passwd.lab
```

### 문제 16) 와일드카드로 여러 파일 한 번에 복사

```bash
cd /home/guest/work/c
cp /lab/backup/*.orig  ./
```

```bash
[root@Server-A c]# ll
합계 20
-rw-r--r-- 1 root root  815  7월  6 12:04 group.orig
-rw-r--r-- 1 root root 7779  7월  6 12:04 login.defs.orig
-rw-r--r-- 1 root root 2124  7월  6 12:04 passwd.orig
-rw-r--r-- 1 root root   54  7월  6 12:04 resolv.conf.orig
drwxr-xr-x 2 root root    6  7월  6 11:25 sub1
```

### 문제 17) 특정 패턴으로 시작하는 파일 복사

```bash
cp /etc/hosts* /home/guest/work/b
```

### 문제 18) 정확히 6글자 파일명 + .conf 확장자 복사

```bash
cp /etc/??????.conf /home/guest/work/c
```

```bash
[root@Server-A c]# ll /home/guest/work/c
합계 100
-rw-r--r-- 1 root root    55  7월  6 12:06 asound.conf
-rw-r--r-- 1 root root 28974  7월  6 12:06 brltty.conf
-rw-r--r-- 1 root root  1370  7월  6 12:06 chrony.conf
-rw-r--r-- 1 root root   117  7월  6 12:06 dracut.conf
-rw-r--r-- 1 root root   815  7월  6 12:04 group.orig
-rw-r--r-- 1 root root  5799  7월  6 12:06 idmapd.conf
-rw-r--r-- 1 root root    19  7월  6 12:06 locale.conf
-rw-r--r-- 1 root root  7779  7월  6 12:04 login.defs.orig
-rw-r--r-- 1 root root  5235  7월  6 12:06 man_db.conf
-rw-r--r-- 1 root root  1208  7월  6 12:06 mke2fs.conf
-rw-r--r-- 1 root root  2124  7월  6 12:04 passwd.orig
-rw-r--r-- 1 root root    54  7월  6 12:06 resolv.conf
-rw-r--r-- 1 root root    54  7월  6 12:04 resolv.conf.orig
-rw-r--r-- 1 root root   458  7월  6 12:06 rsyncd.conf
drwxr-xr-x 2 root root     6  7월  6 11:25 sub1
-rw-r--r-- 1 root root   449  7월  6 12:06 sysctl.conf
```

### 문제 19) .orig 파일만 특정 디렉터리로 복사

```bash
cp /lab/backup/*.orig  ./c/sub1/
```

### 문제 20) 파일 이름 변경 (위치 그대로)

```bash
cd /home/guest/work/a
mv passwd.lab passwd.final
```

```bash
[root@Server-A a]# ll
합계 8
-rw-r--r-- 1 root root  815  7월  6 11:28 group.orig
-rw-r--r-- 1 root root 2124  7월  6 12:00 passwd.final
```

### 문제 21) 파일을 다른 디렉터리로 이동

```bash
mv group.orig  ../b
```

### 문제 22) 와일드카드로 파일 일괄 이동

```bash
cd /home/guest/work/c/sub1
mv *.orig ../
```

### 문제 23) 디렉터리 이동 (old/a 구조 만들기)

```bash
mkdir -p /home/guest/work/old
mv /home/guest/work/a /home/guest/work/old/
```

```bash
[root@Server-A ~]# ll /home/guest/work/old
합계 0
drwxr-xr-x 2 root root 26  7월  6 12:14 a
```

### 문제 24) 디렉터리 이동 및 이름 변경

```bash
mv /lab/users/guest1 /home/guest/work/user-lab/
```

```bash
[root@Server-A ~]# ll -R /home/guest/work/user-lab/
/home/guest/work/user-lab/:
합계 0
drwxr-xr-x 2 root root 6  7월  6 12:35 guest1

/home/guest/work/user-lab/guest1:
합계 0
```

**정리**: 이 종합 실습은 `mkdir`, `ls`, `cp`, `mv`, `rm`을 절대/상대 경로와 와일드카드까지 함께 사용하여 실무 디렉터리 관리 흐름을 반복 훈련한다.

---

## Linux 기본 명령어 (출력 명령어)

- 문서 파일의 내용을 변경하지 않고 파일 안의 내용을 출력 및 확인하는 기능 (문서 파일을 출력시 사용)
  - `cat`
  - `head`
  - `tail`
  - `more`
  - `less`
  - `nl`

### cat

- 문서 파일 안의 내용을 한번에 출력하는 기능
- 파일만 적용되며 디렉터리는 적용할수 없다.
- 스크롤 없이 한번에 모든 내용이 출력되기때문에 작은 문서파일을 확인시 사용한다.
- 형식: `cat [경로/파일명]`
- EX) `cat /etc/NetworkManager/system-connections/ens160.nmconnection`
- EX) `cat /etc/passwd`
- EX) `cat /etc/group`
- EX) `cat /etc/shadow`

### head

- 문서 파일안의 내용을 첫번째 line부터 지정한 line까지 출력하는 기능
- 별도의 옵션을 설정하지 않으면 기본값으로 10개의 line을 출력한다. (첫번째 line부터 10번째 line까지 출력)
- 파일만 적용되며 디렉터리는 적용할수 없다.
- 형식: `head [옵션] [경로/파일명]`
- EX) `head /etc/passwd` → 기본값 첫번째 line부터 10줄 출력
- EX) `head -5 /etc/passwd` → 첫번째 line부터 5줄 출력

### tail

- 문서 파일안의 내용을 마지막 line부터 지정한 line까지 출력하는 기능
- 별도의 옵션을 설정하지 않으면 기본값으로 10개의 line을 출력한다. (마지막 line부터 위로 10개의 line을 출력)
- 파일만 적용되며 디렉터리는 적용할수 없다.
- 형식: `tail [옵션] [경로/파일명]`
- EX) `tail /etc/passwd` → 기본값 마지막 10줄 출력
- EX) `tail -5 /etc/passwd` → 마지막 5줄 출력

### more

- 문서 파일 안의 내용을 실행창 단위로 분할하여 출력하는 기능
- 접속한 실행창(putty, crt)의 한 화면을 page 단위로 분할하여 출력한다.
- 출력한 화면을 위, 아래 방향으로 이동하면서 확인할 수 있다.
- 일반적으로는 문서 파일에 적용하지만 디렉터리에도 적용은 가능하다.
- ls 명령어와 연동해서 사용할 수 있다.
- 마지막 line이 출력되면 more 기능은 종료된다.
- 형식: `more [옵션] [경로/파일명]`
- EX) `more /etc/ssh/sshd_config`
  - `enter` : 한 line을 밑으로 이동
  - `space` : 한 page를 밑으로 이동
  - `b` : 한 page를 위로 이동
  - `q` : more기능 종료

### less

- more의 업그레이드 버전으로 문서 파일 안의 내용을 실행창 단위로 분할하여 출력하는 기능
- 접속한 실행창(putty, crt)의 한 화면을 page 단위로 분할하여 출력한다.
- 출력한 화면을 위, 아래 방향으로 이동하면서 확인할 수 있다.
- 일반적으로는 문서 파일에 적용하지만 디렉터리에도 적용은 가능하다.
- ls 명령어와 연동해서 사용할 수 있다.
- 마지막 line이 출력되어도 less기능은 종료되지 않는다.
  - `j` : 아래 방향으로 한 line 이동
  - `k` : 윗 방향으로 한 line 이동
  - `gg` : 첫 line으로 이동
  - `G` : 마지막 line으로 이동
  - `space` : 한 page 밑으로 이동
  - `b` : 한 page 위로 이동
  - `q` : less기능 종료
- 형식: `less [옵션] [경로/파일명]`
- EX) `less /etc/ssh/sshd_config`
- less는 검색 기능을 지원한다.
  - `/문자열` (위에서부터 검색) → `n` : 아랫 방향으로 검색, `N` : 윗방향으로 검색
  - `?문자열` (밑에서부터 검색) → `n` : 윗방향으로 검색, `N` : 아랫 방향으로 검색

### 출력 명령어 활용 예시

```bash
[root@Server-A ~]# ls -l  /etc | nl        : 각 line에 number를 붙여서 출력
[root@Server-A ~]# cat  /etc/passwd | nl   : 각 line에 number를 붙여서 출력
[root@Server-A ~]# cat  -n  /etc/passwd    : 각 line에 number를 붙여서 출력

[root@Server-A ~]# ls -l /etc | more       : ls 명령어 사용시 more기능을 사용해서 page 단위로 출력 (|을 사용시 위로 이동 X)

[root@Server-A ~]# ls -l /etc | less       : ls 명령어 사용시 less기능을 사용해서 page 단위로 출력 (|을 사용해도 위로 이동 O)

[root@Server-A ~]# ls -l /etc | head -20   : ls 명령어 사용시 head기능을 사용해서 첫번째 line부터 지정한 line까지 출력

[root@Server-A ~]# ls -l /etc | tail -20   : ls 명령어 사용시 tail기능을 사용해서 마지막 line부터 지정한 line까지 출력

[root@Server-A ~]# ls -l  /etc | nl | more : ls 명령어 사용시 more기능을 사용해서 page 단위로 출력 (각 line에 번호를 붙여서 출력)

[root@Server-A ~]# ls -l  /etc | nl | less : ls 명령어 사용시 less기능을 사용해서 page 단위로 출력 (각 line에 번호를 붙여서 출력)
```

**정리**: `cat`은 전체 출력, `head`/`tail`은 앞/뒤 일부 출력, `more`/`less`는 페이지 단위 출력에 사용하며, 파이프(`|`)로 `ls`와 조합해 결과를 다양하게 가공할 수 있다.

---

## Linux 기본 명령어 (cat 명령어 연산자)

- cat명령어는 파일 내용을 출력할수도 있지만 연산자를 사용해서 파일 생성, 파일 복사, 파일 덮어쓰기등 다양한 기능을 사용할 수 있다.
- 가장 많이 사용하는 기능은 연산자를 사용한 파일 조작이다.
  - `>` : 파일 덮어쓰기, 파일 병합
  - `>>` : 추가 쓰기

### > 연산자

- `>` 연산자를 사용하면 새 파일이 생성되며 원본 파일의 내용이 새로운 파일에 작성된다.
- 새로운 파일이 이미 존재한다면 이전의 내용이 모두 삭제되고 새로 덮어쓰기 되기때문에 주의해야 한다.
- 형식: `cat [원본 파일경로/원본 파일명] > [새로운 파일 경로/새로운 파일명]`

```bash
# 빈 파일을 생성 후 원본 데이터의 내용을 빈 파일에 복사
# EX) '/home/guest' 디렉터리에 'newfile' 이름의 빈 파일을 생성한 후
#     '/etc/passwd' 파일안의 설정 내용을 'newfile'안으로 복사해야한다.

[guest@Server-A ~]$ touch  newfile

[guest@Server-A ~]$ cat  /etc/passwd  >  ./newfile

[guest@Server-A ~]$ ls  -l  /etc/passwd  ./newfile
-rw-r--r--  1 guest guest 2124  7월  6 13:20 ./newfile   <--- 파일의 크기가 동일
-rw-r--r--. 1 root  root  2124  7월  2 15:07 /etc/passwd <--- 파일의 크기가 동일
```

```bash
# 빈 파일을 생성 후 새로운 내용을 작성 후 저장

[guest@Server-A ~]$ cat  >  /home/guest/newfile2
hello world~~!!     # 내용 작성
hello soldesk~~!!   # 내용 작성
                    # Ctrl + d (저장 후 종료)

[guest@Server-A ~]$ ls  -l
합계 12
-rw-r--r--  1 guest guest 2124  7월  6 14:33 newfile
-rw-r--r--  1 guest guest   34  7월  6 14:35 newfile2

[guest@Server-A ~]$ cat ./newfile2
hello world~~!!
hello soldesk~~!!
```

```bash
# > 연산자로 덮어쓰기

[guest@Server-A ~]$ cat  >  /home/guest/newfile2
>연산자를 사용합니다.

[guest@Server-A ~]$ cat ./newfile2
>연산자를 사용합니다.   # 기존 내용이 삭제되고 덮어쓰기된다.
```

### >> 연산자

- `>>` 연산자는 기존의 파일을 덮어쓰지 않고 마지막 line 다음 line에 새로운 내용을 추가한다. (append)
- 형식: `cat >> [경로/파일명]`

```bash
[guest@Server-A ~]$ cat  >  ./newfile3
이것은 newfile3의 원본 내용입니다.

[guest@Server-A ~]$ cat  newfile3
이것은 newfile3의 원본 내용입니다.

[guest@Server-A ~]$ cat  >>  ./newfile3
새로운 내용을 추가 합니다.
 이 내용은 추가되는 내용입니다.

[guest@Server-A ~]$ cat  newfile3
이것은 newfile3의 원본 내용입니다.
새로운 내용을 추합니다.        # 덮어쓰기되지 않고 새로운 line에 추가된다.
이 내용은 추가되는내용입니다.  # 덮어쓰기되지 않고 새로운 line에 추가된다.
```

### 여러 파일을 하나의 파일에 병합

```bash
# 형식: cat [원본1] [원본2] > [새로운 파일]

[guest@Server-A ~]$ cat  >  a
동해물과 백두산이 마르고 닳도록
하느님이 보우하사 우리 나라 만세

[guest@Server-A ~]$ cat  >  b
무궁화 삼천리 화려강산
대한 사람 대한으로 길이 보전하세

[guest@Server-A ~]$ cat a b
동해물과 백두산이 마르고 닳도록
하느님이보우하사 우리 나라 만세
무궁화 삼천리 화려강산
대한 사람 대한으로 길이 보전하세

[guest@Server-A ~]$ cat a b > c

[guest@Server-A ~]$ ll
합계 28
-rw-r--r--  1 guest guest   94  7월  6 14:48 a
-rw-r--r--  1 guest guest   82  7월  6 14:48 b
-rw-r--r--  1 guest guest  176  7월  6 14:50 c
-rw-r--r--  1 guest guest 2124  7월  6 13:19 newfile
-rw-r--r--  1 guest guest    0  7월  6 14:38 newfile2
-rw-r--r--  1 guest guest  126  7월  6 14:42 newfile3

[guest@Server-A ~]$ cat c
동배물과 백두산이 마르고 닳도록
하느님이 보우하사 우리 나라 만세
무궁화 삼선치 화려강산
대한 사람 대한으로 길이 보전하세
```

### EOF (End Of File)

- EOF는 입력의 끝을 나타내는 신호 (파일 입력이 끝났음을 나타낸다.)
- 유닉스나 리눅스에서 `cat > file`을 사용할때 키보드의 입력이 끝났음을 알려주기 위해서 `Ctrl + D`를 사용하는데 이 동작이 EOF이다.
  - `Ctrl + D` = 입력 종료 (EOF)
- 파일을 읽을 때 파일의 마지막에 도착하면 이상 읽을 내용이 없다고 판단하게되는데 이 상태를 EOF라고 한다.

- `cat > file`에서 EOF가 필요한 이유:
  - 예를 들어 `cat > newfile` 명령어를 입력하면, 키보드로 입력하는 내용을 newfile 파일에 저장하겠다는 것이다.
  - 이때 터미널은 사용자가 입력하는 내용을 계속 기다린다.
  - 컴퓨터 입장에서는 사용자가 입력을 끝냈는지 모르기때문에 사용자가 입력을 다 끝냈다는 신호를 보내야 한다.
  - 그 신호가 바로 `Ctrl + D`이다.

```bash
[guest@Server-A ~]$ cat  << EOF  >>  testfile
> 새로운 내용을 추가합니다.
> EOF에서 >>를 사용하게되면 내용이 추가됩니다.
> EOF

[guest@Server-A ~]$ cat testfile
지금은 EOF에 대해서 공부합니다.
EOF는 여러줄을 입력하는 기능입니다.
이것은 세번째 라인입니다.
새로운 내용을 추가합니다.
EOF에서 >>를 사용하게되면 내용이 추가됩니다.
```

**정리**: `>` 연산자는 파일을 새로 만들거나 덮어쓰고, `>>`는 기존 내용 뒤에 추가하며, `<< EOF`는 여러 줄을 한 번에 입력할 때 사용한다.

---

## Linux 기본 명령어 (find)

- **find**
  - 특정 조건을 사용하여 파일 또는 디렉터리의 경로를 찾는 기능
  - 형식: `find [경로] [옵션] [파일 또는 디렉터리명]`
  - `-name` : 파일 또는 디렉터리 이름을 사용하여 검색하는 기능

### EX1-1) 최상위 디렉터리에서 파일명이 'passwd'인 경로 검색

```bash
[root@Server-A ~]# find  /  -name  passwd
/etc/pam.d/passwd
/etc/passwd
/usr/bin/passwd
/usr/share/licenses/passwd
/usr/share/doc/passwd
/usr/share/bash-completion/completions/passwd
/home/guest/test/passwd
/soldesk/linux/rocky/version9/passwd
/lg/uplus/display/passwd
/backup2/pam.d/passwd
/backup/passwd
```

### EX1-2) '/etc' 디렉터리에서 파일명이 'passwd'인 경로 검색

```bash
[root@Server-A ~]# find  /etc  -name  passwd
/etc/pam.d/passwd
/etc/passwd
```

### EX1-3) 최상위 디렉터리에서 파일명이 'resolv.conf'인 경로 검색

```bash
[root@Server-A ~]# find  /  -name  resolv.conf
/run/NetworkManager/resolv.conf
/etc/resolv.conf
/usr/lib/systemd/resolv.conf
/home/guest/test/resolv.conf
```

### EX1-4) '/etc' 디렉터리에서 확장자가 '_config'인 경로 검색

```bash
[root@Server-A ~]# find  /etc/  -name  *_config
/etc/ssh/ssh_config
/etc/ssh/sshd_config
```

### EX1-5) 파일만 또는 디렉터리만 검색 (-type 옵션)

```bash
# 파일만 검색 (-type f)
[root@Server-A ~]# find  /etc/  -name  b*  -type f
/etc/logrotate.d/btmp
/etc/logrotate.d/bootlog
/etc/bash_completion.d/bpftool
/etc/bindresvport.blacklist
/etc/bashrc
/etc/profile.d/bash_completion.sh
/etc/selinux/targeted/booleans.subs_dist
/etc/sane.d/bh.conf
/etc/libibverbs.d/bnxt_re.driver
/etc/brltty/Input/al/bc-etouch.kti
/etc/brltty/Input/al/bc-smartpad.kti
~~~~~~~~~~~~  중간 생략 ~~~~~~~~~~~~

# 디렉터리만 검색 (-type d)
[root@Server-A ~]# find  /etc/  -name  b*  -type d
/etc/crypto-policies/back-ends
/etc/bash_completion.d
/etc/pki/ca-trust/source/blocklist
/etc/lvm/backup
/etc/systemd/system/bluetooth.target.wants
/etc/systemd/system/basic.target.wants
/etc/systemd/user/basic.target.wants
/etc/binfmt.d
/etc/bluetooth
/etc/wireplumber/bluetooth.lua.d
/etc/brltty
/etc/brltty/Input/ba
/etc/brltty/Input/bd
```

### EX1-6) IP 설정 파일 검색

```bash
[root@Server-A ~]# find  /etc  -name ens160.nmconnection
/etc/NetworkManager/system-connections/ens160.nmconnection

[root@Server-A ~]# find  /etc  -name ens160*
/etc/NetworkManager/system-connections/ens160.nmconnection
```

### EX2-1) 특정 파일보다 최신인 파일 검색 (-newer 옵션)

```bash
# '/etc' 디렉터리내에서 'aliases' 파일이 생성된 이후에 만들어진 파일 또는 디렉터리를 검색
[root@Server-A ~]# find  /etc/  -newer  /etc/aliases

# 파일만 검색
[root@Server-A ~]# find  /etc/ -type f  -newer  /etc/aliases

# 디렉터리만 검색
[root@Server-A ~]# find  /etc/ -type d  -newer  /etc/aliases
```

### EX2-4) 특정 날짜 이후 생성된 파일 검색 (-newermt 옵션)

```bash
[root@Server-A ~]# mkdir /backup2/rootDir
[root@Server-A ~]# touch /backup2/rootFile

[root@Server-A ~]# ls -l /backup2
합계 24
drwxr-xr-x 4 root root   31  7월  3 17:32 dconf
drwxr-xr-x 3 root root   38  7월  3 17:32 fonts
-rw-r--r-- 1 root root  521  7월  3 17:32 fstab
drwxr-xr-x 4 root root   64  7월  3 17:32 fwupd
-rw-r--r-- 1 root root  158  7월  3 17:32 hosts
drwxr-xr-x 2 root root   52  7월  3 17:32 iscsi
-rw-r--r-- 1 root root   20  7월  3 17:32 issue
drwxr-xr-x 4 root root   33  7월  3 17:32 kdump
drwxr-xr-x 2 root root   35  7월  3 17:32 libnl
-rw-r--r-- 1 root root  111  7월  3 17:32 magic
drwxr-xr-x 2 root root 4096  7월  3 17:32 pam.d
drwxr-xr-x 2 root root   25  7월  3 17:32 pulse
drwxr-xr-x 2 root root    6  7월  6 15:35 rootDir
-rw-r--r-- 1 root root    0  7월  6 15:35 rootFile
-rw-r--r-- 1 root root 4017  7월  3 17:32 vimrc

[root@Server-A ~]# find  /backup2/  -newermt "2026-07-05"
/backup2/
/backup2/rootDir
/backup2/rootFile
```

### EX2-5) 특정 파일 이후에 만들어진 파일 또는 디렉터리 검색

```bash
[root@Server-A ~]# find  /home/guest/  -newer /home/guest/newfile3
/home/guest/
/home/guest/a
/home/guest/b
/home/guest/c
/home/guest/testfile
```

**정리**: **find**는 `-name`, `-type`, `-newer`, `-newermt` 등의 조건을 조합하여 파일 시스템 전체에서 원하는 파일이나 디렉터리를 정확히 찾아낼 수 있는 강력한 검색 명령어이다.

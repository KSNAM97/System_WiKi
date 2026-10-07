# Linux-03 VI 편집기

## VI (Visual Editor) Editor 개요

- VI는 Linux, Unix 계열에서 가장 많이 사용되는 텍스트 편집기이며, GUI가 없는 서버 환경에서 설정 파일이나 쉘 스크립트를 직접 편집할 때 널리 쓰인다.
- 줄 단위가 아닌 화면 단위로 편집하는 Visual Editor(화면 기반 편집기) 라는 의미에서 VI라고 부른다.
- 대부분의 리눅스 서버에는 기본적으로 vi가 설치되어 있으며, GUI가 없는 환경에서도 파일 편집이 가능하다.

**정리**: **vi**는 GUI 없이도 사용 가능한 화면 기반(Visual) 편집기로, 대부분의 리눅스 서버에 기본 설치되어 있어 서버 관리의 필수 도구이다.

---

## VI 3가지 Mode (모드)

VI는 세 가지 Mode(모드)로 동작한다.

```
                    i , a , o , O
vi 실행  ------------>  [ 명령 Mode  ]  ------------>  [입력 Mode]
         [ 명령 Mode  ]  <------------  [입력 Mode]
                             ESC
            |    ^
            |    |
            |    |
         :  |    | ESC
            |    |
            V    |
         [ 실행 Mode  ]
```

### 명령 Mode (Command Mode)

- VI 실행시 기본적으로 적용되는 Mode
- 파일의 내용을 직접 입력하는 모드가 아니라 커서를 이동하거나, 삭제, 복사, 붙여넣기 같은 편집 작업을 수행하는 모드
- 명령어 단축키를 이용해 빠르게 편집 가능
- 명령 Mode에서 `i`, `a`, `o`, `O` 등의 명령어를 사용하여 입력 Mode로 전환할 수 있다.
- 명령 Mode에서 `:` 을 사용하여 실행 Mode로 전환할 수 있다.
- 입력 Mode, 실행 Mode에서 ESC를 사용하여 명령 Mode로 전환이 가능하다.

### 입력 Mode (Insert Mode)

- 실제로 텍스트를 입력할 수 있는 모드
- 명령 mode에서 `i`, `a`, `o`, `O`등을 사용하여 입력 mode로 전환할 수 있다. (ESC를 사용하여 명령 mode로 전환 가능)
- vi editor에 직접 설정이 가능한 mode이다. (명령 Mode에서도 일부 설정은 가능하다.)
- ESC를 누르면 명령 Mode로 복귀함

### 실행 Mode (Execute Mode)

- 명령 Mode에서 `:` 을 사용하여 실행 Mode로 전환할 수 있다.
- 설정 및 수정한 내용을 저장하거나 파일명 변경, 확장명 변경, 치환등이 가능한 Mode이다.
- 명령어는 화면 하단(콜론 라인)에 입력
- 명령 Mode로 돌아가려면 ESC 입력

**정리**: vi는 **명령 Mode**, **입력 Mode**, **실행 Mode** 세 가지 모드를 오가며 동작하며, `i`/`a`/`o`/`O`로 입력 모드 진입, `:`로 실행 모드 진입, `ESC`로 명령 모드 복귀가 기본 흐름이다.

---

## VI 실행 형식

```bash
vi [옵션] [경로/파일명]
```

- 설정한 파일명이 있으면 해당 파일이 open되지만 해당 파일명이 없으면 새로운 파일이 생성된다.

```bash
[root@Server-A ~]# vi  /backup/passwd            # /backup 디렉터리안의 passwd 파일을 vi 편집기로 open

[root@Server-A ~]# vi  +10  /backup/passwd       # 파일을 open 후 10번째 line으로 이동

[root@Server-A ~]# vi  +  /backup/passwd         # 파일을 open 후 마지막 line으로 이동
```

**정리**: `vi [옵션] [경로/파일명]` 형식으로 파일을 열며, 파일이 없으면 새로 생성되고 `+`/`+숫자` 옵션으로 열자마자 특정 line으로 이동할 수 있다.

---

## 명령 Mode 단축키

### 방향 이동

```
j    : 아래 방향으로 1 line이동
k    : 윗 방향으로 1 line이동
l    : 오른쪽으로 1칸 이동
h    : 왼쪽으로 1칸 이동

5j   : 아래 방향으로 5 line이동
5k   : 윗 방향으로 5 line이동
5l   : 오른쪽으로 5칸 이동
5h   : 왼쪽으로 5칸 이동
```

### word 단위 이동

```
w    : 다음 단어의 첫번째 문자열로 이동 (단어, 공백, 특수문자 기준)
W    : 다음 단어의 첫번째 문자열로 이동 (공백 기준)

e    : 다음 단어의 마지막 문자열로 이동 (단어, 공백, 특수문자 기준)
E    : 다음 단어의 마지막 문자열로 이동 (공백 기준)

b    : 이전 단어의 첫번째 문자열로 이동 (단어, 공백, 특수문자 기준)
B    : 이전 단어의 첫번째 문자열로 이동 (공백 기준)
```

### 행 단위 이동

```
0    : 해당 line의 첫번째 문자열로 이동
$    : 해당 line의 마지막 문자열로 이동
```

### 화면 단위 이동

```
H    : 출력된 화면의 첫번째 line으로 이동
M    : 출력된 화면의 가운데 line으로 이동
L    : 출력된 화면의 마지막 line으로 이동
```

### 문서 단위 이동

```
gg   : 전체 문서의 첫번째 line으로 이동
G    : 전체 문서의 마지막 line으로 이동
```

### 문서 편집

```
x          : 커서를 기준으로 오른쪽 문자열 1개를 삭제 (delete)
X          : 커서를 기준으로 왼쪽 문자열 1개를 삭제  (backspace)

dw         : 커서의 오른쪽 단어를 삭제 (공백, 특수문자 기준 | 만약 커서가 단어 가운데 있다면 커서의 오른쪽만 삭제된다.)
dW         : 커서의 오른쪽 단어를 삭제 (공백 기준 | 만약 커서가 단어 가운데 있다면 커서의 오른쪽만 삭제된다.)

db         : 커서의 왼쪽 단어를 삭제 (공백, 특수문자 기준 | 만약 커서가 단어 가운데 있다면 커서의 왼쪽만 삭제된다.)
dB         : 커서의 왼쪽 단어를 삭제 (공백 기준 | 만약 커서가 단어 가운데 있다면 커서의 왼쪽만 삭제된다.)

dd         : 커서가 위치한 1개 line을 삭제
5dd        : 커서가 위치한 line을 기준으로 밑의 5개 line 삭제

yy         : 커서가 위치한 1개 line을 복사
5yy        : 커서를 기준으로 아래 5개 line을 복사
yw         : 커서가 위치한 1개의 단어를 복사 (공백 특수문자 기준 | 만약 커서가 단어 가운데 있다면 커서의 오른쪽만 복사)
yW         : 커서가 위치한 1개의 단어를 복사 (공백 기준 | 만약 커서가 단어 가운데 있다면 커서의 오른쪽만 복사)

p          : 커서를 기준으로 아랫 line에 붙여넣기 (단어를 복사하면 커서의 위치에서 붙여넣기)
P          : 커서를 기준으로 윗 line에 붙여넣기

r          : 1개의 문자열을 치환

u          : 실행취소 (앞으로)
Ctrl + r   : 실행취소 (뒤로)
```

**정리**: 방향(`h`/`j`/`k`/`l`), 단어(`w`/`e`/`b`), 행/화면/문서 단위 이동과 `dd`/`yy`/`p`/`u` 같은 삭제·복사·붙여넣기·실행취소 단축키를 조합하면 마우스 없이도 빠르게 문서를 편집할 수 있다.

---

## 입력 Mode 전환 키

```
i    : 현재 커서의 위치에서 입력 모드로 전환
a    : 현재 커서의 다음 문자열 위치에서 입력 모드로 전환
o    : 현재 커서의 아랫 line에 공백처리하고 입력 모드로 전환
O    : 현재 커서의 윗 line에 공백처리하고 입력 모드로 전환
```

**정리**: `i`/`a`는 현재/다음 위치에서, `o`/`O`는 아래/위에 새 line을 만들어 입력 모드로 전환하는 키이다.

---

## 실행 Mode 명령어

```
:set number     = 문서의 각 line에 숫자를 순서대로 붙여서 출력

:q              = vi Editor 종료 (quit)
:w              = vi Editor를 사용해서 설정 추가, 수정, 삭제한 내용을 저장
:wq             = vi Editor를 사용해서 설정 추가, 수정, 삭제한 내용을 저장 후 종료
:!              = 강제 실행 (force)
:q!             = 강제 종료 (read only 파일에서 변경한 정보를 저장하지 않고 강제 종료)
:w!             = 강제 저장

:3              = 해당 문서의 3번째 line으로 이동
:25             = 해당 문서의 25번째 line으로 이동
:$              = 해당 문서의 마지막 line으로 이동

:.              = 현재 커서의 위치
:.d             = 현재 커서가 있는 line을 삭제
:10d            = 10번째 line을 삭제
:3,10d          = 3번째 line부터 10번째 line까지 삭제
:.,20d          = 커서가있는 line부터 20번째 line까지 삭제
:.,+3d          = 현재 커서부터 밑으로 3줄을 삭제 (총 4줄 삭제)
:.,-3d          = 현재 커서부터 위로 3줄을 삭제 (총 4줄 삭제)

:.y             = 현재 커서가 있는 line을 복사
:10y            = 10번째 line을 복사
:3,10y          = 3번째 line부터 10번째 line까지 복사
:.,20y          = 커서가있는 line부터 20번째 line까지 복사
:.,+3y          = 현재 커서부터 밑으로 3줄을 복사 (총 4줄 복사)
:.,-3y          = 현재 커서부터 위로 3줄을 복사 (총 4줄 복사)
:.,$            = 현재 커서부터 마지막 line 까지 복사
:20,$y          = 20번 라인부터 마지막 line 까지 복사
:%y             = 문서 전체 복사
```

**정리**: 실행 Mode의 `:w`/`:q`/`:wq`로 저장·종료를 제어하고, `:숫자`, `:.`, `:,`, `%` 등 범위 지정 문법으로 line 단위 삭제·복사를 정교하게 수행할 수 있다.

---

## .swp (Swap File)

- vi가 파일을 열때 자동으로 생성되는 임시 백업 파일
- vi를 사용해서 파일을 open 하게되면 리눅스는 `.swp` 파일을 생성한다.
  - EX) `vi passwd` → `.passwd.swp`
- vi 편집기를 사용해서 설정을 추가, 수정, 삭제하게되면 원본 파일에 바로 적용되지 않고 `.swp` 파일에 저장된다.
- vi 편집기를 정상적으로 종료하면 `.swp` 파일이 삭제되지만 비정상적으로 종료되면 `.swp` 파일은 삭제되지 않는다.
- 남아있는 `.swp` 파일을 사용해서 데이터를 복구할 수 있다.
- `.swp` 파일이 남아있는 상태에서 해당 파일을 vi로 열게되면 `.swp`파일이 있다고 메세지를 띄워준다.

```bash
[root@Server-A ~]# cp  /etc/passwd  /backup3

[root@Server-A ~]# ls -l /backup3
합계 4
-rw-r--r-- 1 root root 2124  7월  6 16:56 passwd

[root@Server-A ~]# vi  /backup3/passwd       # putty 1로 접속 후 vi 실행

[root@Server-A ~]# ls  -la  /backup3         # putty 2로 접속 후 확인
합계 20
drwxr-xr-x   2 root root     39  7월  6 16:57 .
dr-xr-xr-x. 25 root root   4096  7월  6 16:55 ..
-rw-r--r--   1 root root  12288  7월  6 16:58 .passwd.swp    # swp 파일 생성됨
-rw-r--r--   1 root root   2124  7월  6 16:56 passwd
```

**정리**: **.swp** 파일은 vi가 편집 중 생성하는 임시 백업 파일로, 정상 종료 시 삭제되지만 비정상 종료 시 남아 있으면 이를 이용해 데이터를 복구할 수 있다.

---

## VI Editor Shell Command (실행 Mode에서 쉘 명령어)

- vi는 텍스트 편집기지만, 편집 도중에 리눅스 Shell 명령어를 직접 실행할 수 있는 매우 강력한 기능을 제공
  - vi 파일 수정 도중 디렉터리 목록 확인
  - 특정 파일 내용 확인
  - grep, cat, tail 등 사용
  - vi 내부에서 컴파일/실행
  - vi 상태를 유지한 채 외부 명령어 수행

### 실습 환경 준비

```bash
[root@Server-A ~]# cp  /etc/NetworkManager/system-connections/ens160.nmconnection  /backup/
[root@Server-A ~]# cp  /etc/passwd  /backup/
[root@Server-A ~]# cp  /etc/login.defs  /backup/

[root@Server-A ~]# ls -l  /backup
합계 16
-rw------- 1 root root  288  7월  7 09:44 ens160.nmconnection
-rw-r--r-- 1 root root 7779  7월  7 09:44 login.defs
-rw-r--r-- 1 root root 2124  7월  7 09:44 passwd
```

### EX1-3) passwd 파일 편집 후 line 삭제

```bash
[root@Server-A ~]# cp  /backup/passwd  /home/guest

[root@Server-A ~]# cd ~guest

[root@Server-A guest]# vi  passwd
:set number
:11,35d
```

### EX1-4) vi 편집기 실행 상태에서 외부 명령어 확인

```bash
# vi 내부에서 쉘 명령어 실행 (출력만, vi 문서에 영향 없음)
:!ls -l /root

[No write since last change]
합계 4
-rw-r--r--  1 root root    0  7월  6 13:11 123
-rw-------. 1 root root 1027  7월  2 12:55 anaconda-ks.cfg
drwxr-xr-x. 2 root root    6  7월  2 14:46 공개
drwxr-xr-x. 2 root root    6  7월  2 14:46 다운로드
drwxr-xr-x. 2 root root    6  7월  2 14:46 문서
drwxr-xr-x. 2 root root    6  7월  2 14:46 바탕화면
drwxr-xr-x. 2 root root    6  7월  2 14:46 비디오
drwxr-xr-x. 2 root root    6  7월  2 14:46 사진
drwxr-xr-x. 2 root root    6  7월  2 14:46 서식
drwxr-xr-x. 2 root root    6  7월  2 14:46 음악

Press ENTER or type command to continue
```

### EX1-5) 명령 결과를 vi 문서의 마지막 line에 삽입

```bash
# 방법 1 : :.! 명령어 (현재 line을 쉘 명령어 결과로 교체)
G          # 마지막 line으로 이동
o          # 아래에 새 line 생성
ESC
:.!  ls  -l  /root

# 방법 2 : :r ! 명령어 (쉘 명령어 결과를 현재 커서 아래에 삽입)
G
o
ESC
:r !  ls  -l  /root

     1 root:x:0:0:root:/root:/bin/bash
     2 bin:x:1:1:bin:/bin:/sbin/nologin
     ...
    13 guest:x:1000:1000:guest:/home/guest:/bin/bash
    14 합계 4
    15 -rw-r--r--  1 root root    0  7월  6 13:11 123
    16 -rw-------. 1 root root 1027  7월  2 12:55 anaconda-ks.cfg
    17 drwxr-xr-x. 2 root root    6  7월  2 14:46 공개
```

### EX1-6) 10번째 line에 명령 결과 삽입

```bash
# 10번째 line의 내용을 ls -l /root 결과로 교체
:10!  ls -l  /root

     3 daemon:x:2:2:daemon:/sbin:/sbin/nologin
     4 adm:x:3:4:adm:/var/adm:/sbin/nologin
     ...
     9 mail:x:8:12:mail:/var/spool/mail:/sbin/nologin
    10 합계 4
    11 -rw-r--r--  1 root root    0  7월  6 13:11 123
    12 -rw-------. 1 root root 1027  7월  2 12:55 anaconda-ks.cfg
```

### EX1-7) 파일 내용 초기화 후 /etc/passwd 내용으로 채우기

```bash
:%d
:r ! cat  /etc/passwd
```

### EX1-8) 두 파일 비교 (diff)

```bash
:! diff  /etc/passwd  /home/guest/passwd

Press ENTER or type command to continue
7c7
< shutdown:x:6:0:shutdown:/sbin:/sbin/shutdown
---
> shutdown:x:6:0:shutdown:/sbin:/sbin/shutdown now

shell returned 1

Press ENTER or type command to continue
```

### EX1-9) 다른 이름으로 백업 저장 (vi는 원본 파일 유지)

```bash
:w  /home/passwd.bak

:! ls  -l  /home/guest

합계 40
...
-rw-r--r--  1 root  root  2128  7월  7 10:13 passwd
-rw-r--r--  1 root  root  2128  7월  7 10:32 passwd.bak
```

**정리**: vi 실행 Mode에서 `:!명령어`, `:r !명령어`, `:.!명령어`를 사용하면 편집 도중 쉘 명령어 실행 결과를 확인하거나 문서에 바로 삽입할 수 있어 매우 강력하다.

---

## VI 편집기 치환 (Substitute)

- VI 편집기에서는 특정 문자열을 다른 문자열로 변경하는 치환(Substitute) 기능을 제공한다.
- 이 기능은 실행 Mode(콜론 모드)에서 사용되며, 간단한 검색, 변경부터 파일 전체 치환까지 가능하다.

### 치환 형식

```
:s/원본문자열/변환문자열
```

- 현재 커서가 위치한 line의 문자열만 치환한다.
- 해당 line의 첫번째 문자열을 치환한다. (같은 line에 같은 단어가 여러개가 있어도 첫번째 문자열만 치환한다.)

### 실습 파일 생성 (파일명 = OS)

```bash
[root@Server-A ~]# vi OS
```

```
Operating System
OS = unixos , linuxos , windowsos
unix = cisco IOS , linux , android
Windows = windows xp , windows 7 , windows 8 , windows 10
cisco IOS 12.4 version
Red Hat Linux = Fedora linux , CentOS linux , RedHat linux
Debian Linux =  Ubuntu linux , kali linux
Linux cenos 7 = Linux Redtat 7
selinux = only centOS
selinux = off
linux download = linuxOS
windows10 enterprise downloads
ens-32
IPADDR="192.168.1.251"
NETMASK="255.255.255.0"
GATEWAY="192.168.1.2"
DNS1="192.168.1.2"
```

### EX2) 6번째 line의 첫번째 linux를 soldesk로 변경

```
:6
:s/linux/soldesk
```

```
     1 Operating System
     2 OS = unixos , linuxos , windowsos
     3 unix = cisco IOS , linux , android
     4 Windows = windows xp , windows 7 , windows 8 , windows 10
     5 cisco IOS 12.4 version
     6 Red Hat Linux = Fedora soldesk , CentOS linux , RedHat linux   # 첫번째 linux만 치환
     7 Debian Linux =  Ubuntu linux , kali linux
     8 Linux cenos 7 = Linux Redtat 7
     9 selinux = only centOS
    10 selinux = off
    11 linux download = linuxOS
    12 windows10 enterprise downloads
    13 ens-32
    14 IPADDR="192.168.1.251"
    15 NETMASK="255.255.255.0"
    16 GATEWAY="192.168.1.2"
    17 DNS1="192.168.1.2"
```

### EX3) line 번호와 치환 동시에

```
:6s/linux/soldesk
```

### EX4) 2번째 line부터 7번째 line의 첫번째 linux를 soldesk로 변경

```
:2,7s/linux/soldesk
```

```
     1 Operating System
     2 OS = unixos , soldeskos , windowsos        # 첫번째 linux만 치환
     3 unix = cisco IOS , soldesk , android       # 첫번째 linux만 치환
     4 Windows = windows xp , windows 7 , windows 8 , windows 10
     5 cisco IOS 12.4 version
     6 Red Hat Linux = Fedora soldesk , CentOS linux , RedHat linux   # 첫번째 linux만 치환
     7 Debian Linux =  Ubuntu soldesk , kali linux   # 첫번째 linux만 치환
     8 Linux cenos 7 = Linux Redtat 7
     9 selinux = only centOS
    10 selinux = off
    11 linux download = linuxOS
```

### EX5) 문서 전체의 linux를 WIN으로 치환 (각 line의 첫번째만)

```
:%s/linux/WIN
```

```
     2 OS = unixos , WINos , windowsos
     3 unix = cisco IOS , WIN , android                      # 첫번째만
     6 Red Hat Linux = Fedora WIN , CentOS linux , RedHat linux   # 첫번째만
     9 seWIN = only centOS
    10 seWIN = off
    11 WIN download = linuxOS
```

### EX6) 6번째 line의 모든 linux를 CISCO로 치환 (/g 플래그)

```
:6s/linux/CISCO/g
```

- `-g` : 해당 line에서 match되는 모든 문자열을 치환한다.

```
     6 Red Hat Linux = Fedora CISCO , CentOS CISCO , RedHat CISCO   # 모든 linux 치환
```

### EX7) 2번째 ~ 7번째 line의 모든 linux를 soldesk로 치환

```
:2,7s/linux/soldesk/g
```

```
     2 OS = unixos , soldeskos , windowsos
     3 unix = cisco IOS , soldesk , android
     6 Red Hat Linux = Fedora soldesk , CentOS soldesk , RedHat soldesk
     7 Debian Linux =  Ubuntu soldesk , kali soldesk
```

### EX8) 대/소문자 구분 없이 전체 치환 (/gi 플래그)

```
:%s/linux/HELLO/gi
```

- `-i` : 대/소문자를 구분하지 않고 치환

```
     6 Red Hat HELLO = Fedora soldesk , CentOS soldesk , RedHat soldesk
     7 Debian HELLO =  Ubuntu soldesk , kali soldesk
     8 HELLO cenos 7 = HELLO Redtat 7     # Linux도 HELLO로 치환됨
     9 seHELLO = only centOS
    10 seHELLO = off
    11 HELLO download = HELLOOS
```

### EX9) line의 시작 문자열이 'linux'인 경우만 치환 (^ 앵커)

```
:%s/^linux/soldesk
```

```
    11 soldesk download = linuxOS   # 해당 line의 시작 문자열이 linux인 경우에만 치환
```

### EX9-1) line의 마지막 문자열이 'linux'인 경우만 치환 ($ 앵커)

```
:%s/linux$/soldesk
```

```
     6 Red Hat Linux = Fedora linux , CentOS linux , RedHat soldesk   # 마지막 linux만 치환
     7 Debian Linux =  Ubuntu linux , kali soldesk                     # 마지막 linux만 치환
```

### EX10) IP 주소 일괄 치환

```
:%s/192.168.1/172.16.100/g
```

```
    14 IPADDR="172.16.100.251"
    15 NETMASK="255.255.255.0"
    16 GATEWAY="172.16.100.2"
    17 DNS1="172.16.100.2"
```

### EX11) 정확히 'linux' 단어만 치환 (단어 경계 \< \>)

- `\<` : 단어의 시작
- `\>` : 단어의 끝

```
:%s/\<linux\>/WindowS/g
```

- `selinux`, `linuxOS` 같은 문자열은 치환되지 않고, 정확히 `linux`라는 단어만 치환된다.

**정리**: `:s/원본/변환/` 형식에 범위(`:2,7`, `:%`), `/g`(전체 치환), `/i`(대소문자 무시), `^`/`$`(라인 앵커), `\<`/`\>`(단어 경계) 등을 조합하면 대용량 파일에서도 원하는 문자열만 정밀하게 일괄 치환할 수 있다.

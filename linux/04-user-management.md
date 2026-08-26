# Linux-04 사용자 및 그룹 관리

## 목차

1. [Linux 계정 개요](#linux-계정-개요)
2. [사용자 계정 관련 파일](#사용자-계정-관련-파일)
3. [사용자 계정 생성 (useradd)](#사용자-계정-생성-useradd)
4. [사용자 계정 수정 (usermod)](#사용자-계정-수정-usermod)
5. [사용자 계정 삭제 (userdel)](#사용자-계정-삭제-userdel)
6. [사용자 계정 Password 관리 (passwd)](#사용자-계정-password-관리-passwd)
7. [su 명령어 (사용자 계정 전환)](#su-명령어-사용자-계정-전환)
8. [그룹 관리 (groupadd / groupmod / groupdel)](#그룹-관리-groupadd--groupmod--groupdel)
9. [SUDO (Substitute User and Do)](#sudo-substitute-user-and-do)

## Linux 계정 개요

- Linux는 관리자 계정인 root 계정과 일반 사용자 계정으로 나누어 관리하며, 신입 직원의 계정 생성부터 퇴직자의 계정 삭제까지 계정과 그룹을 관리하는 명령어들이 실무에서 꾸준히 쓰인다.
- 사용자 계정은 다시 login이 가능한 사용자 계정과 login없이 시스템에 의해서 만들어지는 시스템 사용자 계정으로 나누어 관리한다.
- root 계정은 시스템의 모든 권한을 갖는 사용자이므로 '권한이 있는 사용자 (Privilege User)' 또는 '슈퍼 유져 (Super User)'라고 한다.
- 일반 사용자 계정은 시스템의 권한이 없거나 또는 제한적이기때문에 '권한이 없는 사용자 (Unprivilege User)' 또는 "Normal User"라고 한다.
- 실제 시스템은 ID형식이 아닌 UID라고하는 숫자 형식을 사용하여 계정을 인식한다.
- UID값이 '0'인 계정이 관리자 계정으로 동작한다.
- RedHat 계열 (RHEL 7 이상, Rocky Linux, AlmaLinux): 시스템 계정 UID 0~999, 일반 사용자 UID 1000~
- 하나의 계정은 반드시 하나 이상의 Group에 소속되어야 한다.

**정리**: Linux는 UID 0인 **root** 계정과 그 외 일반 사용자 계정으로 구분되며, RedHat 계열은 시스템 계정(0~999)과 일반 사용자(1000~)로 UID 범위를 나눈다.

---

## 사용자 계정 관련 파일

| 파일 | 설명 |
|------|------|
| `/etc/passwd` | 관리자 및 일반 사용자 계정 정보 (계정ID, UID, GID, 홈 디렉터리, 쉘) |
| `/etc/shadow` | 암호화된 Password 정보, 비밀번호 만료일 등 보안 정보 (root만 읽기 가능) |
| `/etc/group` | 시스템의 그룹 정보 (그룹 이름, GID, 보조 그룹 구성원) |
| `/etc/default/useradd` | useradd 명령으로 계정 생성 시 적용되는 기본 설정 |
| `/etc/login.defs` | UID, GID 최소/최대값, 비밀번호 정책 등 시스템 전역 설정 |
| `/etc/skel/` | 신규 사용자 계정 생성 시 기본으로 복사될 파일 디렉터리 |
| `/var/spool/mail/` | 계정 생성 시 자동 생성되는 메일박스 파일 |

```bash
[root@Server-A ~]# cat  /etc/passwd | grep guest
guest:x:1000:1000:guest:/home/guest:/bin/bash

# 필드 설명
# 1) guest    : 계정명 (계정ID)
# 2) x        : password를 의미 (실제 password는 /etc/shadow 파일에 저장된다.)
# 3) 1000     : User-UID (시스템 계정 = 0 ~ 999, 사용자 계정 = 1000 ~)
# 4) 1000     : Group-ID(GID) : 해당 계정이 소속된 Group
# 5) guest    : Comment (닉네임, 생략시 계정 ID와 같은 값)
# 6) /home/guest : 해당 계정의 홈 디렉터리 경로
# 7) /bin/bash   : 해당 계정이 사용할 쉘 정보

[root@Server-A ~]# cat  /etc/shadow    # 암호화된 패스워드 확인
[root@Server-A ~]# cat  /etc/group     # 그룹 정보 확인
[root@Server-A ~]# cat  /etc/default/useradd    # 계정 생성시 기본값 확인
[root@Server-A ~]# ls -la /etc/skel    # skel 디렉터리 확인
[root@Server-A ~]# ls -l  /var/spool/mail    # 메일 계정 확인
```

**정리**: **/etc/passwd**, **/etc/shadow**, **/etc/group**은 각각 계정 정보, 암호화된 비밀번호, 그룹 정보를 담고 있으며, `/etc/skel`은 신규 계정 생성 시 기본으로 복사되는 템플릿 디렉터리이다.

---

## 사용자 계정 생성 (useradd)

```bash
# /etc/default/useradd 기본값 확인
[root@Server-A ~]# cat  /etc/default/useradd
# useradd defaults file
GROUP=100           # GID 기본값
HOME=/home          # 홈 디렉터리 경로
INACTIVE=-1         # Password 만료일 비활성화
EXPIRE=             # Password 만료일 설정 (기본값 = 비활성화)
SHELL=/bin/bash     # 계정 생성시 기본 shell
SKEL=/etc/skel      # skel 디렉터리 경로
CREATE_MAIL_SPOOL=yes   # 계정 생성시 Mail 자동 생성 유/무

[root@Server-A ~]# useradd -D    # 동일한 내용 확인
```

### EX2) user1 계정 생성 기본 예시

```bash
[root@Server-A ~]# useradd  user1

[root@Server-A ~]# ls  -l  /home
합계 4
drwx------. 16 guest guest 4096  7월  7 10:34 guest
drwx------   3 user1 user1   78   7월  7 12:31 user1    # user1의 홈 디렉터리 생성 확인

[root@Server-A ~]# ls  -la  /home/user1
합계 12
drwx------  3 user1 user1  78  7월  7 12:31 .
drwxr-xr-x. 4 root  root    32  7월  7 12:31 ..
-rw-r--r--  1 user1 user1  18  4월 30  2024 .bash_logout    # /etc/skel 디렉터리의 내용이 복사
-rw-r--r--  1 user1 user1 141  4월 30  2024 .bash_profile   # /etc/skel 디렉터리의 내용이 복사
-rw-r--r--  1 user1 user1 492  4월 30  2024 .bashrc         # /etc/skel 디렉터리의 내용이 복사
drwxr-xr-x  4 user1 user1  39 10월 30  2022 .mozilla        # /etc/skel 디렉터리의 내용이 복사

[root@Server-A ~]# tail  -2  /etc/passwd
guest:x:1000:1000:guest:/home/guest:/bin/bash
user1:x:1001:1001::/home/user1:/bin/bash    # user1 계정 정보 확인

[root@Server-A ~]# tail  -2  /etc/shadow
guest:$6$UjralX4dEzs/5RAr$...:20636:0:99999:7:::
user1:!!:20641:0:99999:7:::

[root@Server-A ~]# ls  -l  /var/spool/mail/
합계 0
-rw-rw----. 1 guest mail 0  7월  2 14:38 guest
-rw-rw----  1 user1 mail 0  7월  7 12:31 user1

# user1 계정에는 비밀번호가 설정되지 않았기때문에 로그인할 수 없다.
[root@Server-A ~]# passwd user1
user1 사용자의 비밀 번호 변경 중
새 암호:1234
잘못된 암호: 암호는 8 개의 문자 보다 짧습니다
새 암호 재입력:1234
passwd: 모든 인증 토큰이 성공적으로 업데이트 되었습니다.
```

### EX3) -c 옵션 (comment/닉네임 설정)

```bash
[root@Server-A ~]# useradd  -c  soldesk  user2

[root@Server-A ~]# tail -3  /etc/passwd
guest:x:1000:1000:guest:/home/guest:/bin/bash
user1:x:1001:1001::/home/user1:/bin/bash
user2:x:1002:1002:soldesk:/home/user2:/bin/bash    # commant = soldesk
```

### EX4) -s 옵션 (사용할 shell 지정)

```bash
# 필요한 shell 설치
[root@Server-A ~]# dnf  install  -y  csh  tcsh  zsh

# 현재 사용가능한 쉘 확인
[root@Server-A ~]# cat  /etc/shells
/bin/sh
/bin/bash
/usr/bin/sh
/usr/bin/bash
/bin/csh
/bin/tcsh
/usr/bin/csh
/usr/bin/tcsh
/usr/bin/zsh
/bin/zsh

[root@Server-A ~]# useradd  -c  chulsu  -s  /bin/tcsh  user3

[root@Server-A ~]# tail  -4  /etc/passwd
guest:x:1000:1000:guest:/home/guest:/bin/bash
user1:x:1001:1001::/home/user1:/bin/bash
user2:x:1002:1002:soldesk:/home/user2:/bin/bash
user3:x:1003:1003:chulsu:/home/user3:/bin/tcsh    # tcsh 사용 확인

# 쉘 변경 (현재 로그인 계정의 쉘 변경)
[root@Server-A ~]# chsh  -s  /bin/tcsh
```

### EX5) skel 디렉터리 커스터마이징 후 계정 생성

```bash
# skel에 신규 파일 추가
[root@Server-A ~]# vi /etc/skel/manual.txt
# ... 내용 작성 ...
:wq

[root@Server-A ~]# ls  -la  /etc/skel
합계 32
drwxr-xr-x.   3 root root   110  7월  7 14:37 .
...
-rw-r--r--    1 root root  2644  7월  7 14:37 manual.txt

# csh shell, minsu 닉네임으로 user4 생성 (manual.txt가 자동 복사됨)
[root@Server-A ~]# useradd  -c minsu  -s /bin/csh  user4

[root@Server-A ~]# ls  -la  /home/user4
합계 20
drwx------  3 user4 user4  110  7월  7 14:40 .
...
-rw-r--r--  1 user4 user4 2644  7월  7 14:37 manual.txt    # skel에서 복사됨
```

### EX6) -d 옵션 (홈 디렉터리 경로 지정)

```bash
# user5: /solhome 디렉터리에 홈 디렉터리 생성
[root@Server-A ~]# useradd  -c  mansurr  -s /bin/tcsh  -d  /solhome/user5  user5

[root@Server-A ~]# ls  -l  /solhome
합계 0
drwx------ 3 user5 user5 110  7월  7 14:50 user5

[root@Server-A ~]# tail  -6  /etc/passwd
...
user5:x:1005:1005:mansurr:/solhome/user5:/bin/tcsh
```

### EX7) 사용자 지정 skel 디렉터리 (-k 옵션)

```bash
# skel 디렉터리 생성 및 복사
[root@Server-A ~]# mkdir  /etc/skel-eng
[root@Server-A ~]# mkdir  /etc/skel-insa
[root@Server-A ~]# mkdir  /etc/skel-sales

[root@Server-A ~]# cp  -a  /etc/skel/.  /etc/skel-eng
[root@Server-A ~]# cp  -a  /etc/skel/.  /etc/skel-insa
[root@Server-A ~]# cp  -a  /etc/skel/.  /etc/skel-sales

# -a (archive mode) : 파일, 디렉터리, 숨김 파일을 허가권, 소유권, 생성시간 원본 그대로 복사

# 각 skel에 맞는 파일/디렉터리 추가
[root@Server-A ~]# mkdir  /etc/skel-eng/Engineer
[root@Server-A ~]# touch  /etc/skel-eng/eng-manual

[root@Server-A ~]# mkdir  /etc/skel-insa/Insabu
[root@Server-A ~]# touch  /etc/skel-insa/insa-manual

[root@Server-A ~]# mkdir  /etc/skel-sales/Sales
[root@Server-A ~]# touch  /etc/skel-sales/sales-manual

# -k 옵션으로 다른 skel 디렉터리 사용
# user6: skel-eng 사용, /solhome 홈 디렉터리
[root@Server-A ~]# useradd  -c  mango  -s  /bin/tcsh  -m -k  /etc/skel-eng  -d  /solhome/user6  user6

# -k : 기본 skel(/etc/skel)이 아닌 다른 skel 디렉터리 지정 옵션 (-m 옵션과 함께 사용해야 한다.)
# -m : 홈 디렉터리 자동 생성 + /etc/skel 모든 파일 복사

[root@Server-A ~]# ls  -la  /solhome/user6
합계 20
drwx------ 4 user6 user6  144  7월  7 15:14 .
...
drwxr-xr-x 2 user6 user6    6  7월  7 15:08 Engineer    # skel-eng에서 복사됨
-rw-r--r-- 1 user6 user6    0  7월  7 15:08 eng-manual  # skel-eng에서 복사됨
-rw-r--r-- 1 user6 user6 2644  7월  7 14:37 manual.txt
```

### EX9-EX10) 다른 skel로 계정 생성

```bash
# user7: skel-insa, /home 디렉터리
[root@Server-A ~]# useradd  -c  minji  -mk  /etc/skel-insa  user7

# user8: skel-sales, /saleshome 디렉터리
[root@Server-A ~]# useradd  -c saja  -mk  /etc/skel-sales/  -d  /saleshome/user8  user8
```

### EX12) /etc/default/useradd 기본값 변경

```bash
[root@Server-A ~]# vi  /etc/default/useradd
# useradd defaults file
GROUP=100
HOME=/solhome        # 기본 디렉터리 변경
INACTIVE=-1
EXPIRE=
SHELL=/bin/tcsh      # 기본 쉘 변경
SKEL=/etc/skel
CREATE_MAIL_SPOOL=yes

[root@Server-A ~]# useradd  guest2    # 변경된 기본값 적용되어 생성

[root@Server-A ~]# ls  -l  /solhome/
합계 0
drwx------ 3 guest2 guest2 110  7월  7 15:52 guest2    # /solhome에 생성 확인
```

### useradd -D 옵션으로 기본값 변경

```bash
[root@Server-A ~]# useradd  -D  -b  /solhome  -s  /bin/tcsh

# -D    : useradd 명령어의 기본값을 조회 또는 변경하는 옵션
# -D -b : 기본 홈 디렉터리의 상위 디렉터리 경로 (base directory)를 변경
# -D -s : 계정 생성시 기본 shell 변경
# skel 정보는 vi 편집기로 직접 수정해야 한다.

[root@Server-A ~]# useradd -D
GROUP=100
HOME=/solhome
INACTIVE=-1
EXPIRE=
SHELL=/bin/tcsh
SKEL=/etc/skel-sales
CREATE_MAIL_SPOOL=yes
```

**정리**: **useradd**는 `-c`(comment), `-s`(shell), `-d`(홈 경로), `-k`/`-m`(skel 지정), `-D`(기본값 변경) 등의 옵션으로 계정 생성 방식을 세밀하게 제어할 수 있다.

---

## 사용자 계정 수정 (usermod)

- usermod는 이미 생성된 사용자 계정의 속성을 수정하는 명령어이다.
- 형식: `usermod [옵션] [옵션값] [계정명]`

### EX1) 쉘 변경 (-s 옵션)

```bash
[root@Server-A ~]# usermod  -s  /bin/tcsh  user1

[root@Server-A ~]# grep user1  /etc/passwd
user1:x:1001:1001::/home/user1:/bin/tcsh    # tcsh 사용으로 변경됨
```

### EX2) 홈 디렉터리 변경 (-d 옵션)

```bash
# -d : /etc/passwd 파일안의 경로만 변경 (실제 홈 디렉터리는 변경되지 않음)
[root@Server-A ~]# usermod  -d  /home/user6  user6

[root@Server-A ~]# cat  /etc/passwd | grep user6
user6:x:1006:1006:mango:/home/user6:/bin/tcsh    # 경로 변경됨

# 하지만 실제 디렉터리는 /solhome/user6에 있기 때문에 로그인 후 "No such file or directory" 오류 발생
# 수동으로 디렉터리 이동 필요
[root@Server-A ~]# mv  /solhome/user6  /home
```

### EX3) -md 옵션 (실제 홈 디렉터리까지 이동)

```bash
# -md 사용 조건: /etc/passwd 경로와 실제 홈 디렉터리 경로가 일치해야 한다.
# (user1: 경로 일치 → -md 사용 가능)
[root@Server-A ~]# usermod  -md  /saleshome/user1  user1

[root@Server-A ~]# cat  /etc/passwd | grep user1
user1:x:1001:1001::/saleshome/user1:/bin/tcsh    # 경로 변경

[root@Server-A ~]# ls  -l  /saleshome/
합계 4
drwx------ 14 user1 user1 4096  7월  7 17:11 user1    # 실제 디렉터리도 이동됨

# (user2: 경로 불일치 → -md 사용 불가, 오류 발생)
[root@Server-A ~]# usermod  -md  /saleshome/user2  user2
usermod: The previous home directory (/home/user2) does not exist or is inaccessible. Move cannot be completed.
```

**정리**: **usermod**의 `-d`는 `/etc/passwd`의 경로만 바꾸고, `-md`는 실제 홈 디렉터리까지 함께 이동하지만 기존 경로가 실제로 존재해야 사용할 수 있다.

---

## 사용자 계정 삭제 (userdel)

```bash
# userdel : /etc/passwd, /etc/group, /etc/shadow 파일안의 정보만 삭제
# 홈 디렉터리와 메일 정보는 삭제되지 않기 때문에 수동으로 삭제해야 한다.
[root@Server-A ~]# userdel  user8

[root@Server-A ~]# find / -name  user8
/var/spool/mail/user8    # 이메일 정보 남아있음
/home/user8              # 홈 디렉터리 남아있음

[root@Server-A ~]# rm -rf  /home/user8
[root@Server-A ~]# rm -rf  /var/spool/mail/user8

# -r 옵션: 계정에 관련된 모든 파일 및 디렉터리를 삭제 (홈 디렉터리 안의 데이터가 삭제되므로 사용 시 주의)
[root@Server-A ~]# userdel  -r  user6
```

**정리**: **userdel**은 기본적으로 계정 정보만 삭제하고 홈 디렉터리/메일은 남기므로, 관련 파일까지 한 번에 정리하려면 `-r` 옵션을 사용해야 한다.

---

## 사용자 계정 Password 관리 (passwd)

```bash
# passwd 옵션
# -S : Password의 기본 정보 확인
# -l : Password에 Lock을 걸어 login을 차단
# -u : Password에 걸려있는 Lock을 해제
# -d : 설정된 Password를 삭제

[root@Server-A ~]# passwd -S guest
guest PS 2026-07-02 0 99999 7 -1 (비밀번호가 설정되어있습니다, SHA512 암호화.  )

# guest    : 확인한 사용자 계정
# PS       : Password Set, 비밀번호가 설정된 상태
# 2026-07-02 : 비밀번호를 마지막으로 변경한 날짜
# 0        : 비밀번호 변경 후 다시 변경하기까지 최소 일수 (0 = 언제든 변경 가능)
# 99999    : 비밀번호 만료까지의 기간 (99999 = 사실상 만료되지 않음)
# 7        : 비밀번호 만료 7일 전부터 경고 메시지 출력
# -1       : 비밀번호 만료 후 계정 비활성화까지의 기간 (-1 = 미사용)

[root@Server-A ~]# passwd  -l  guest    # guest 계정 password 잠금
guest 사용자의 비밀 번호를 잠급니다
passwd: 성공

[root@Server-A ~]# passwd -S guest
guest LK 2026-07-02 0 99999 7 -1 (비밀 번호가 잠겨있습니다.)    # LK = 잠금 상태

[root@Server-A ~]# passwd  -u  guest    # guest 계정 잠금 해제
guest 사용자의 비밀 번호 잠금 해제 중
passwd: 성공

[root@Server-A ~]# passwd -S guest
guest PS 2026-07-02 0 99999 7 -1 (비밀번호가 설정되어있습니다, SHA512 암호화.  )

[root@Server-A ~]# passwd  -d  guest    # guest계정의 비밀번호 삭제
guest 사용자의 비밀 번호 삭제 중
passwd: 성공

[root@Server-A ~]# passwd -S guest
guest NP 2026-07-08 0 99999 7 -1 (비밀번호를 입력해 주세요.)
```

**정리**: **passwd**의 `-S`(상태 확인), `-l`(잠금), `-u`(잠금 해제), `-d`(비밀번호 삭제) 옵션으로 계정의 로그인 가능 여부와 비밀번호 상태를 관리할 수 있다.

---

## su 명령어 (사용자 계정 전환)

```bash
# su : 현재 사용자는 root로 변경되었지만, 작업 위치와 일부 환경 설정은 기존 사용자 환경을 유지
[guest@Server-A ~]$ su
암호:
[root@Server-A guest]# pwd
/home/guest

# su - : root로 직접 로그인한 것과 동일한 환경 (권장)
[guest@Server-A ~]$ su -
암호:
[root@Server-A ~]# pwd
/root
```

| 구분 | root 계정으로 바로 접속 시 위험 | 일반 계정에서 root 전환 시 장점 |
|------|----------------------------|-----------------------------|
| 접근 범위 | 시스템 전체 접근 가능 | 제한된 환경에서만 root 권한 사용 |
| 실수 위험 | 중요 파일 삭제 시 복구 어려움 | sudo로 명령 단위 실행 |
| 보안 위험 | root 비밀번호 유출 시 전체 시스템 제어 가능 | 일반 계정마다 별도 암호로 접근 추적 가능 |
| 감사/로그 | root 직접 로그인 시 사용자 추적 어려움 | sudo 사용 시 로그에 사용자 이름 남음 |

### SSH root 직접 접속 차단

```bash
[root@Server-A ~]# vi  /etc/ssh/sshd_config
     40 PermitRootLogin no    # no로 변경하여 root의 직접 접속 차단
:wq

[root@Server-A ~]# systemctl  restart  sshd

login as: root
root@192.168.10.100's password:
Access denied    # root 계정 접속 차단됨
```

**정리**: `su`는 작업 위치를 유지한 채 계정만 전환하고 `su -`는 완전한 root 로그인 환경을 제공하며, 보안을 위해 `sshd_config`의 `PermitRootLogin no` 설정으로 root 직접 SSH 접속을 차단하는 것이 권장된다.

---

## 그룹 관리 (groupadd / groupmod / groupdel)

- 리눅스에서 그룹(Group)은 여러 사용자를 하나의 권한 단위로 묶기 위해 사용하는 기능이다.
- Red Hat 계열은 계정을 생성할 때 사용자명과 동일한 이름의 그룹이 자동 생성된다. (User Private Group, UPG 방식)
- 형식: `groupadd [옵션] [group명]` / `groupdel [옵션] [group명]`

```bash
# 홈 디렉터리 생성
[root@Server-A ~]# mkdir  /homeA
[root@Server-A ~]# mkdir  /homeB
[root@Server-A ~]# mkdir  /homeC

# 계정 생성
[root@Server-A ~]# useradd  -md  /homeA/userA1  userA1
[root@Server-A ~]# useradd  -md  /homeA/userA2  userA2
[root@Server-A ~]# useradd  -md  /homeA/userA3  userA3

[root@Server-A ~]# tail  -9  /etc/passwd
userA1:x:1013:1013::/homeA/userA1:/bin/bash
userA2:x:1014:1014::/homeA/userA2:/bin/bash
...
userC3:x:1021:1021::/homeC/userC3:/bin/bash
```

### EX2-1) 그룹 생성

```bash
[root@Server-A ~]# groupadd GroupA
[root@Server-A ~]# groupadd GroupB
[root@Server-A ~]# groupadd GroupC

[root@Server-A ~]# tail -3 /etc/group
GroupA:x:1022:
GroupB:x:1023:
GroupC:x:1024:
```

### EX2-2) 특정 GID로 그룹 생성

```bash
[root@Server-A ~]# groupadd  -g 2040 GroupD

[root@Server-A ~]# tail -1 /etc/group
GroupD:x:2040:
```

### EX2-3) 그룹 삭제

```bash
[root@Server-A ~]# groupdel  GroupD    # 소속 계정이 없는 그룹은 삭제 가능

[root@Server-A ~]# groupdel  userA1
groupdel: 'userA1' 사용자의 주요 그룹을 제거할 수 없습니다
# 해당 그룹에 계정이 소속되어있으면 삭제 불가
```

### EX2-4) 계정을 기본 그룹으로 변경 (-g 옵션)

```bash
# -g : 기본 그룹을 변경 (기존 그룹 대체)
[root@Server-A ~]# usermod  -g  1022  userA1

[root@Server-A ~]# id  userA1
uid=1013(userA1) gid=1022(GroupA) groups=1022(GroupA)

[root@Server-A ~]# usermod  -g  1022  userA2
[root@Server-A ~]# usermod  -g  1022  userA3

# GroupB에 소속 변경
[root@Server-A ~]# usermod  -g  GroupB  userB1
[root@Server-A ~]# usermod  -g  GroupB  userB2
[root@Server-A ~]# usermod  -g  GroupB  userB3

[root@Server-A ~]# id userB1
uid=1016(userB1) gid=1023(GroupB) groups=1023(GroupB)
```

### EX2-9) 기존 그룹 유지하며 보조 그룹 추가 (-aG 옵션)

```bash
# -g  : 기본 그룹을 변경
# -G  : 기본 그룹이 아닌 보조 그룹 목록 지정 (기존 그룹을 유지하면서 새로운 그룹을 추가)
# -a  : append로 새로운 그룹에 추가를 의미

[root@Server-A ~]# usermod  -aG  GroupD  userD2

[root@Server-A ~]# id userD2
uid=1024(userD2) gid=1027(userD2) groups=1027(userD2),1029(GroupD)
```

### EX2-10) 여러 보조 그룹 동시 추가

```bash
[root@Server-A ~]# usermod  -aG  GroupB,GroupD  userD3

[root@Server-A ~]# id userD3
uid=1025(userD3) gid=1028(userD3) groups=1028(userD3),1023(GroupB),1029(GroupD)
```

### 그룹에서 계정 제거

```bash
[root@Server-A ~]# gpasswd  -d  subroot  wheel
사용자 subroot을(를) 그룹 wheel에서 제거하는 중
```

**정리**: **groupadd**/**groupdel**로 그룹을 관리하고, **usermod**의 `-g`(기본 그룹 변경)와 `-aG`(보조 그룹 추가)를 구분해서 사용하는 것이 계정을 여러 그룹에 소속시키는 핵심이다.

---

## SUDO (Substitute User and Do)

- sudo는 특정 사용자가 root(또는 다른 계정)의 권한으로 명령을 실행할 수 있게 해주는 기능이다.
- root로 직접 로그인시키지 않고, 허용된 명령에 대해서만 일시적으로 root 권한을 부여한다.
- 자기 자신의 비밀번호로 인증한다.
- sudo 사용 시 로그에 사용자 이름이 남는다.
- RHEL/CentOS/Rocky 계열: wheel 그룹을 통해 권한 위임

### 방법 1) wheel 그룹에 계정 추가

```bash
# wheel 그룹: root 권한을 사용할 수 있는 사용자를 묶어놓은 관리자 전용 그룹
[root@Server-A ~]# id  guest
uid=1000(guest) gid=1000(guest) groups=1000(guest),10(wheel)

[guest@Server-A ~]$ cat  /etc/shadow
cat: /etc/shadow: 허가 거부

[guest@Server-A ~]$ sudo head -5  /etc/shadow
# ... 첫 번째 실행시 경고 메시지 출력 ...
[sudo] guest 암호:1234

root:$6$dvArAR/...
bin:*:19123:0:99999:7:::
daemon:*:19123:0:99999:7:::
adm:*:19123:0:99999:7:::
lp:*:19123:0:99999:7:::

# subroot 계정에 sudo 권한 부여
[root@Server-A ~]# useradd  subroot
[root@Server-A ~]# usermod  -aG  wheel  subroot

[root@Server-A ~]# id  subroot
uid=1026(subroot) gid=1031(subroot) groups=1031(subroot),10(wheel)

[subroot@Server-A ~]$ sudo head  -5  /etc/shadow
[sudo] subroot 암호:
root:$6$dvArAR/...
bin:*:19123:0:99999:7:::
```

### 방법 2) sudoers 파일에 직접 계정 추가

```bash
[root@Server-A ~]# useradd  rootsub

[root@Server-A ~]# vi  /etc/sudoers
     99 ## Allow root to run any commands anywhere
    100 root      ALL=(ALL)       ALL
    101 rootsub   ALL=(ALL)       ALL    # rootsub 계정에 관리자 권한 설정

:wq!    # sudoers 파일은 강제 저장 필요

[root@Server-A ~]# cat  /etc/sudoers | grep  root
root    ALL=(ALL)       ALL
rootsub ALL=(ALL)       ALL

[rootsub@Server-A ~]$ sudo tail  -5  /etc/shadow
[sudo] rootsub 암호:
userD2:!!:20642:0:99999:7:::
...
rootsub:$6$rounds=100000$...:20642:0:99999:7:::
```

### UID 0 계정 (관리자 계정 원리)

```bash
# 시스템은 계정 이름이 아닌 UID 값으로 계정을 구분한다.
# UID=0인 계정이 관리자로 동작한다.

[root@Server-A ~]# useradd  sol

[root@Server-A ~]# vi  /etc/passwd
# sol 계정의 UID, GID를 0으로 변경
sol:x:0:0::/home/sol:/bin/bash

# sol 계정으로 로그인하면 root 권한으로 동작
login as: sol
sol@192.168.10.100's password:
[root@Server-A ~]#    # root 프롬프트로 로그인됨
```

**정리**: **sudo**는 `wheel` 그룹 소속 또는 `/etc/sudoers` 직접 등록을 통해 일반 계정에 임시로 root 권한을 위임하며, 시스템은 계정 이름이 아니라 UID 값(0 = root)으로 관리자 권한을 판단한다.

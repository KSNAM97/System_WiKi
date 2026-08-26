# Linux 허가권 (Permission) / 소유권 (Ownership)

## 목차

1. [허가권 (Permission)](#허가권-permission)
2. [소유권 (Ownership)](#소유권-ownership)
3. [특수 권한 Umask](#특수-권한-umask)
4. [특수 권한 (Set-UID / Set-GID / Sticky-bit)](#특수-권한-set-uid--set-gid--sticky-bit)
5. [Set-GID](#set-gid)

## 허가권 (Permission)

Linux는 여러 사용자가 동시에 접속해 같은 파일 시스템을 공유하는 멀티유저 OS이며, 허가권은 이 과정에서 다른 사용자에게 정보가 노출되거나 시스템 파일이 임의로 변경되는 것을 막기 위해 파일/디렉터리 접근을 세밀하게 제어하는 기능이다.
- 허가권은 특정 파일 또는 디렉터리에 대해 누가 어떤 작업을 할 수 있는지를 결정하는 기능이다.
- Linux는 허가권을 세 가지 사용자 범위에 대해 각각 부여한다.
  - Owner : 파일의 소유자
  - Group : 파일이 속한 그룹의 사용자
  - Other : 그 외 모든 사용자
- 각 사용자 범위에 대해 r, w, x 권한을 개별적으로 설정한다.
  - r (read)
  - w (write)
  - x (execute)

### 권한 변경

- 특정 파일, 디렉터리의 권한을 변경하는 기능
- 형식: `chmod  [권한]  [경로/파일명 또는 디렉터리명]`

```
rwxr-xr-x = 755
처음3개문자 = user의 권한
중간3개문자 = group의 권한
마지막3개문자 = other의 권한
r은 파일 읽기 = 4, w는 파일 쓰기 = 2, x는 파일 실행 = 1로, 3개문자씩 수를 더해서 쓴다.
```

```
=================================================================
파일 종류	|  Owner (소유주)	|     Group (그룹)	|    Other (그 외)	|
=================================================================
 d -	|     r     w     x	|     r     w     x	|     r     w     x	|
-----------------------------------------------------------------
 d -	|     -     -     -	|     -     -     -	|     -     -     -	|
-----------------------------------------------------------------


=================================================================
파일 종류	|  Owner (소유주)	|     Group (그룹)	|    Other (그 외)	|
=================================================================
 d -	|     r     w     x	|     r     w     -	|     r     -     x	|
-----------------------------------------------------------------
 d -	|     1     1     1	|     1     1     0	|     1     0     1	|
-----------------------------------------------------------------
 d -	|     	7	|     	6	|     	5	|
-----------------------------------------------------------------
```

- d = Directory
- \- = File
- l = Link file (바로가기)

**r (read)**
- 파일 = 파일 안의 데이터를 확인할 수 있는 권한을 의미 (cat, vi)
- 디렉터리 = 해당 디렉터리안의 파일 및 디렉터리 볼수있는 일부의 권한 (ls -l)

**w (write)**
- 파일 = 해당 파일안의 데이터를 수정 또는 변경할 수 있는 권한
- 디렉터리 = 해당 디렉터리와 그 하위의 디렉터리 및 파일의 생성 및 변경, 삭제할 수 있는 권한

**x (execute)**
- 파일 = 파일에 'x' 권한 없이면 일반 문서 파일을 의미하며 'x' 권한이 있으면 실행파일을 의미한다. (실행파일 or 스크립트여야 함)
  root 사용자, 일반사용자가 파일을 생성하게되면 기본적으로 'x' 권한이 없는 문서 파일로 생성되며 해당 파일을 실행 파일로 변경하기위해는 별도의 설정을 통해서 수동으로 변경해야한다.
- 디렉터리 = 해당 디렉터리로 접근 권한을 의미한다. (해당 디렉터리 안으로 들어갈수있는 권한 cd)

- 특정 디렉터리의 권한을 777로 부여하게되면 공유 폴더의 개념으로 사용이 가능하다.

### root/일반 사용자 기본 권한

- root 계정의 기본 권한
  - root는 시스템 전체를 관리하는 계정이므로 보다 안전한 기본 권한이 적용된다.
  - 디렉터리 생성: 기본 권한 755 (rwx r-x r-x)
  - 파일 생성: 기본 권한 644 (rw- r-- r--)
  - 소유주(root)는 읽기·쓰기 가능
  - 그룹과 기타 사용자는 읽기만 가능
  - 새 파일에는 실행(x) 권한이 자동 부여되지 않음

- 일반 사용자 계정의 기본 권한
  - 일반 사용자 계정들은 협업을 고려한 비교적 완화된 기본 권한을 가진다.
  - 디렉터리 생성: 기본 권한 755 (rwx r-x r-x)
  - 파일 생성: 기본 권한 644 (rw- r-- r--)
  - 소유주와 동일 그룹 사용자까지 읽기/쓰기 가능
  - 기타 사용자는 읽기만 가능
  - 새 파일에는 마찬가지로 실행(x) 권한 없음

- 사용자 홈 디렉터리 기본 권한
  - useradd 명령어로 홈 디렉터리를 생성할 시 일반 파일/디렉터리와 다르게 보안을 위해 강제로 700이 부여된다.
  - 홈 디렉터리: 기본 권한 700 (rwx --- ---)
  - 이유: 다른 사용자가 홈 디렉터리 내부를 조회하면 개인정보, 설정 파일이 노출될 위험기때문이다.

```
[root@Server-A ~]# rm  -rf  /backup/*

[root@Server-A ~]# cp  /etc/passwd  /backup/
[root@Server-A ~]# cp  /etc/group  /backup/

[root@Server-A ~]# mkdir  /backup/testD
[root@Server-A ~]# touch  /backup/testF

[root@Server-A ~]# ls  -l  /backup
합계 8
-rw-r--r-- 1 root root 1262  7월  8 16:56 group
-rw-r--r-- 1 root root 3244  7월  8 16:56 passwd
drwxr-xr-x 2 root root     6  7월  8 16:56 testD		# 디렉터리 허가권 	: 755 (rwx r-x r-x)
-rw-r--r-- 1 root root     0  7월  8 16:56 testF		# 파일 허가권 	: 644 (rw- r-- r--)
```

### 실습 EX1 ~ EX4

### EX1-1) 아래의 조건에 맞게 설정하시오
- 'root' 계정을 사용하여 Server-A로 접속
- 'guest' 계정을 사용하여 Server-A로 접속
- 'root' 계정을 사용하여 최상위 디렉터리에 'net' 디렉터리를 생성
- '/net' 디렉터리의 권한을 777로 변경하시오

```
[root@Server-A ~]# mkdir  /net

[root@Server-A ~]# ls  -l  / | grep net
drwxr-xr-x    2 root root    6  7월  8 16:59 net

[root@Server-A ~]# chmod  777  /net

[root@Server-A ~]# ls  -l  / | grep net
drwxrwxrwx    2 root root    6  7월  8 16:59 net	# 777 (rwx rwx rwx)
```

### EX1-2) 아래의 조건에 맞게 설정하시오
- 'guest' 계정을 사용하여 '/net' 안을 ls 명령어로 확인
- 'guest' 계정을 사용하여 '/net' 디렉터리로 이동
- 'guest' 계정을 사용하여 '/net' 디렉터리에 'guestD' 디렉터리를 생성하시오
- 'guest' 계정을 사용하여 '/net' 디렉터리에 'guestF' 파일을 생성하시오

```
[guest@Server-A ~]$ ls  -l  /net
합계 0

[guest@Server-A ~]$ cd  /net

[guest@Server-A net]$ pwd
/net

[guest@Server-A net]$ mkdir guestD
[guest@Server-A net]$ touch guestF

[guest@Server-A net]$ ls  -l  /net
합계 0
drwxr-xr-x 2 guest guest 6  7월  8 17:05 guestD	# 사용자 계정으로 생성한 디렉터리 (755)
-rw-r--r-- 1 guest guest 0  7월  8 17:05 guestF	# 사용자 계정으로 생성한 파일 (644)
```

### EX2-1) root 계정을 사용하여 /backup 디렉터리의 'passwd' 파일을 '/net' 디렉터리로 복사

```
[root@Server-A ~]# cp  /backup/passwd  /net

[root@Server-A ~]# ls  -l  /net
합계 4
drwxr-xr-x 2 guest guest    6  7월  8 17:05 guestD
-rw-r--r-- 1 guest guest    0  7월  8 17:05 guestF
-rw-r--r-- 1 root  root  3244  7월  8 17:07 passwd
```

### EX2-2) 'guest' 계정은 '/net' 디렉터리에 대해서 어떠한 권한이 적용되나?
- 'guest' 계정은 '/net' 디렉터리에 대해서 other 권한이 적용 (rwx rwx rwx)
- 'guest' 계정은 '/net' 디렉터리에 대해서 x 권한이 있기때문에 해당 디렉터리로 이동이 가능하다.
- 'guest' 계정은 '/net' 디렉터리에 대해서 w 권한이 있기때문에 해당 디렉터리안에서 파일 및 디렉터리를 생성, 수정, 삭제할 수 있다.
- 'guest' 계정은 '/net' 디렉터리에 대해서 r 권한이 있기때문에 해당 디렉터리안의 정보를 확인할 수 있다.

```
[guest@Server-A ~]$ ls  -l  /net
합계 0

[guest@Server-A ~]$ cd  /net

[guest@Server-A net]$ pwd
/net

[guest@Server-A net]$ ls  -l  /net
합계 0
drwxr-xr-x 2 guest guest 6      7월  8 17:05 guestD		# 디렉터리 생성 확인
-rw-r--r-- 1 guest guest 0      7월  8 17:05 guestF		# 파일 생성 확인
-rw-r--r-- 1 root  root  3244  7월  8 17:07 passwd
```

### EX2-3) 'guest' 계정은 '/net' 디렉터리의 passwd 파일을 읽을수 있나?

```
[guest@Server-A net]$ vi  /net/passwd
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
adm:x:3:4:adm:/var/adm:/sbin/nologin
lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
sync:x:5:0:sync:/sbin:/bin/sync
shutdown:x:6:0:shutdown:/sbin:/sbin/shutdown
~~~~~~~~ 중간 생략 ~~~~~~~~
```

- guest 계정은 /net/passwd 파일에 대해서 r 권한이 있기때문에 읽을 수 있다.

### EX2-4) 'guest' 계정은 '/net' 디렉터리의 passwd 파일 내용을 수정할 수 있나?

```
[guest@Server-A net]$ vi  /net/passwd
~~~~~~~~ 중간 생략 ~~~~~~~~
polkitd:x:998:996:User for polkitd:/:/sbin/nologin
avahi:x:70:70:Avahi mDNS/DNS-SD Stack:/var/run/avahi-daemon:/sbin/nologin
rtkit:x:172:172:RealtimeKit:/proc:/sbin/nologin
sssd:x:997:993:User for sssd:/:/sbin/nologin
pipewire:x:996:992:PipeWire System Daemon:/var/run/pipewire:/sbin/nologin
libstoragemgmt:x:995:991:daemon account for libstoragemgmt:/var/run/lsm:/sbin/nologin
tss:x:59:59:Account used for TPM access:/dev/null:/sbin/nologin
"/net/passwd" [readonly] 63L, 3244B		<--- w 권한이 없기때문에 readonly로 확인
```

### EX2-5) 'guest' 계정은 '/net' 디렉터리의 passwd 파일명을 변경할 수 있나?

```
[root@Server-A ~]# ls -l / | grep net
drwxrwxrwx    3 root root   48  7월  8 17:07 net		# guest 계정은 net 디렉터리에 대해서 other 권한 (rwx rwx rwx)

[guest@Server-A net]$ mv  ./passwd ./password

[guest@Server-A net]$ ls -l
합계 4
drwxr-xr-x 2 guest guest     6  7월  8 17:05 guestD
-rw-r--r-- 1 guest guest     0  7월  8 17:05 guestF
-rw-r--r-- 1 root  root  3244  7월  8 17:07 password	# 파일명 수정 확인
```

---

### EX3-1) 아래의 조건에 맞게 설정하시오
- 'root' 계정을 사용하여 '/net' 디렉터리안의 모든 파일 및 디렉터리를 삭제
- 'root' 계정을 사용하여 '/backup' 디렉터리안의 모든 파일을 '/net' 디렉터리로 복사

```
[root@Server-A ~]# rm  -rf  /net/*

[root@Server-A ~]# cp -r  /backup/*  /net

[root@Server-A ~]# ls  -l  /net
합계 8
-rw-r--r-- 1 root root 1262  7월  8 17:36 group
-rw-r--r-- 1 root root 3244  7월  8 17:36 passwd
drwxr-xr-x 2 root root     6  7월  8 17:36 testD
-rw-r--r-- 1 root root     0  7월  8 17:36 testF
```

### EX3-2) '/net' 디렉터리의 허가원을 아래의 조건에 맞게 변경하시오
- Ower 권한 = 읽기(r), 쓰기(w), 실행(x) 권한이 부여되어야 한다.
- Group 권한 = 읽기(r), 쓰기(w), 실행(x) 권한이 부여되어야 한다.
- Other 권한 = 쓰기(w), 실행(x) 권한이 부여되어야 한다.

```
[root@Server-A ~]# chmod  773  /net

[root@Server-A ~]# ls  -l  / | grep net
drwxrwx-wx    3 root root   59  7월  8 17:36 net
```

### EX3-3) 'guest' 계정은 '/net' 디렉터리안에 파일 및 디렉터리를 생성할 수 있는가?
- 'guest' 계정을 사용하여 '/net' 디렉터리로 이동
- 'guest' 계정을 사용하여 'net' 디렉터리에 'GuestD' 디렉터리 생성
- 'guest' 계정을 사용하여 'net' 디렉터리에 'GuestFile' 파일 생성
- 'root' 계정을 사용하여 파일 및 디렉터리의 생성 유/무 확인

```
[guest@Server-A net]$ cd ~		# 홈 디렉터리로 이동

[guest@Server-A ~]$ cd  /net		# guest계정은  /net에 대해서 x 권한이 있기때문에 해당 디렉터리로 이동 가능

[guest@Server-A net]$ pwd
/net

[guest@Server-A net]$ mkdir  GuestD
[guest@Server-A net]$ touch GuestF

[root@Server-A ~]# ls  -l  /net	# root 계정
합계 8
drwxr-xr-x 2 guest guest    6  7월  8 17:44 GuestD
-rw-r--r-- 1 guest guest    0  7월  8 17:44 GuestF
-rw-r--r-- 1 root  root  1262  7월  8 17:36 group
-rw-r--r-- 1 root  root  3244  7월  8 17:36 passwd
drwxr-xr-x 2 root  root     6  7월  8 17:36 testD
-rw-r--r-- 1 root  root     0  7월  8 17:36 testF
```

### EX3-4) 'guest' 계정은 '/net' 디렉터리안에 파일 및 디렉터리 정보를 확인할 수 있는가?
- 'guest' 계정을 사용하여 'net' 디렉터리안의 정보를 'ls  -l  /net' 명령어를 사용하여 확인

```
[guest@Server-A net]$ ls  -l  /net		# guest 계정
ls: cannot open directory '/net': 허가 거부

[root@Server-A ~]# ls  -l  /net		# root 계정
합계 8
drwxr-xr-x 2 guest guest     6  7월  8 17:44 GuestD		# 디렉터리 생성 확인
-rw-r--r-- 1 guest guest     0  7월  8 17:44 GuestF		# 파일 생성 확인
-rw-r--r-- 1 root  root  1262  7월  8 17:36 group
-rw-r--r-- 1 root  root  3244  7월  8 17:36 passwd
drwxr-xr-x 2 root  root      6  7월  8 17:36 testD
-rw-r--r-- 1 root  root      0  7월  8 17:36 testF
```

- guest 계정은 /net 디렉터리에 대해서 other (-wx) 권한을 갖기때문에 해당 디렉터리의 정보를 확인할 수 없다. (r 권한이 없음)

### EX3-5) 'guest' 계정은 '/net' 디렉터리안에 파일 및 디렉터리 정보를 삭제할 수 있는가?
- 'guest' 계정을 사용하여 'net' 디렉터리에 'GuestD' 디렉터리 삭제
- 'guest' 계정을 사용하여 'net' 디렉터리에 'GuestFile' 파일 삭제

```
[guest@Server-A net]$ rm  -rf  /net/GuestD
[guest@Server-A net]$ rm  -rf  /net/GuestF

[guest@Server-A net]$ ls  -l  /net		# guest 계정으로 확인 X
ls: cannot open directory '/net': 허가 거부

[root@Server-A ~]# ls  -l  /net		# root 계정으로 확인
합계 8
-rw-r--r-- 1 root root 1262  7월  8 17:36 group
-rw-r--r-- 1 root root 3244  7월  8 17:36 passwd
drwxr-xr-x 2 root root    6  7월  8 17:36 testD
-rw-r--r-- 1 root root     0  7월  8 17:36 testF
```

---

### EX4-1) '/net' 디렉터리의 허가권을 아래의 조건에 맞게 변경하시오
- Ower 권한 = 읽기(r), 쓰기(w), 실행(x) 권한이 부여되어야 한다.
- Group 권한 = 읽기(r), 쓰기(w), 실행(x) 권한이 부여되어야 한다.
- Other 권한 = 읽기(r), 실행(x) 권한이 부여되어야 한다.

```
[root@Server-A ~]# chmod  775  /net

[root@Server-A ~]# ls  -l  / | grep net
drwxrwxr-x    3 root root   59  7월  8 17:47 net
```

### EX4-2) 'guest' 계정은 '/net' 디렉터리안에 파일 및 디렉터리를 생성할 수 있는가?
- 'guest' 계정을 사용하여 '/net' 디렉터리로 이동
- 'guest' 계정을 사용하여 '/net' 디렉터리에 'GuestD' 디렉터리 생성
- 'guest' 계정을 사용하여 '/net' 디렉터리에 'GuestFile' 파일 생성
- 'guest' 계정을 사용하여 파일 및 디렉터리의 생성 유/무 확인
- 'root' 계정을 사용하여 파일 및 디렉터리의 생성 유/무 확인

```
[guest@Server-A ~]$ cd  /net

[guest@Server-A net]$ pwd
/net

[guest@Server-A net]$ mkdir  GuestD
mkdir: `GuestD' 디렉토리를 만들 수 없습니다: 허가 거부

[guest@Server-A net]$ touch GuestF
touch: cannot touch 'GuestF': 허가 거부
```

- 'guest' 계정은 '/net' 디렉터리에 대해서 other의 권한을 갖는다. (rwx rwx r-x)
- '/net' 디렉터리의 허가권이 775(rwx rwx r-x)이므로 guest 계정은 w 권한이 없기때문에 파일 및 디렉터리를 생성, 삭제할 수 없다.

```
[guest@Server-A net]$ ls  -l  /net		# r-x 권한이므로 '/net' 디렉터리안의 정보를 확인할 수 있다.
합계 8
-rw-r--r-- 1 root root 1262  7월  8 17:36 group
-rw-r--r-- 1 root root 3244  7월  8 17:36 passwd
drwxr-xr-x 2 root root     6  7월  8 17:36 testD
-rw-r--r-- 1 root root     0  7월  8 17:36 testF
```

- '/net' 디렉터리의 허가권이 775(rwx rwx r-x)이므로 guest 계정은 r 권한이 있기때문에 해당 디렉터리의 정보를 확인할 수 있다.

### 추가 예제 (776 허가권)

```
root@Server-A:~# chmod 776 /512
root@Server-A:~# ll / | grep 512
drwxrwxrw-    2 root root    6  7월  8 18:03 512
root@Server-A:~# mkdir /512/aaa
root@Server-A:~# touch /512/bbb


[guest@Server-A net]$ ll /512
합계 0
[guest@Server-A net]$ mkdir /512/aaa
mkdir: `/512/aaa' 디렉토리를 만들 수 없습니다: 허가 거부
[guest@Server-A net]$ ll /512
ls: cannot access '/512/aaa': 허가 거부
ls: cannot access '/512/bbb': 허가 거부
합계 0
d????????? ? ? ? ?             ? aaa
-????????? ? ? ? ?             ? bbb
[guest@Server-A net]$ ll /512/aaa
ls: cannot access '/512/aaa': 허가 거부
[guest@Server-A net]$ ll /512/bbb
ls: cannot access '/512/bbb': 허가 거부
```

**정리**: **허가권**은 Owner/Group/Other 세 범위에 각각 r(4)/w(2)/x(1) 값을 조합해 부여하며, `chmod` 명령으로 숫자 또는 기호 표기를 사용해 변경할 수 있다.

---

## 소유권 (Ownership)

- Linux에서 파일과 디렉터리는 반드시 소유권을 가진다.
- 소유권은 두 가지로 구성된다.
  - 사용자 소유권(Owner)
  - 그룹 소유권(Group)
- 파일 또는 디렉터리의 소유자는 해당 자원에 대한 지배 권한을 가지며 읽기, 쓰기, 실행 권한을 설정하거나 변경할 수 있다.
- 소유권 변경은 관리자가 특정 파일 또는 디렉터리의 사용자(owner) 또는 그룹(group)을 다른 사용자나 그룹으로 바꿀 때 사용한다.
- 형식: `chown  [UID:GID]  [경로/파일명 또는 디렉터리명]`
- chown 명령은 소유자, 그룹을 개별 또는 동시에 변경할 수 있다.

### EX1-1) 'root' 계정을 사용하여 아래의 조건에 맞게 설정하시오
- root 계정을 사용하여 'auser1' 홈 디렉터리안에 'owner' 디렉터리를 생성하시오
- '/backup' 디렉터리안의 모든 파일 및 디렉터리를 'owenr' 디렉터리안에 복사해야한다.

```
[root@Server-A ~]# mkdir /gitA/auser1/owner

[root@Server-A ~]# cp  -r  /backup/*  /gitA/auser1/owner

[root@Server-A ~]# ls -l /gitA/auser1
합계 4
-rw-r--r-- 1 auser1 gitA     0  7월  9 09:47 GIT-A-Manual
-rw-r--r-- 1 auser1 gitA 2644  7월  7 14:37 manual.txt
drwxr-xr-x 3 root   root    59  7월  9 10:12 owner

[root@Server-A ~]# ls -l /gitA/auser1/owner/
합계 8
-rw-r--r-- 1 root root 1262  7월  9 10:12 group
-rw-r--r-- 1 root root 3244  7월  9 10:12 passwd
drwxr-xr-x 2 root root     6  7월  9 10:12 testD
-rw-r--r-- 1 root root     0  7월  9 10:12 testF
```

### EX1-2) 'auser1' 계정을 사용하여 'owner' 디렉터리를 삭제

```
[auser1@Server-A ~]$ rm -rf ./owner/		# auser1은 owner 디렉터리에 대해서 w권한이 없기때문에 삭제할 수 없다.
rm: cannot remove './owner/group': 허가 거부
rm: cannot remove './owner/passwd': 허가 거부
rm: cannot remove './owner/testD': 허가 거부
rm: cannot remove './owner/testF': 허가 거부
```

### EX1-3) '/gitA/auser1/owner' 디렉터리안의 'passwd' 파일의 소유주를 'auser1'으로 변경해야한다.

```
[root@Server-A ~]# ls -l /gitA/auser1/owner/
합계 8
-rw-r--r-- 1 root root 1262  7월  9 10:12 group
-rw-r--r-- 1 root root 3244  7월  9 10:12 passwd		# 소유주 = root, 소유 그룹 = root
drwxr-xr-x 2 root root     6  7월  9 10:12 testD
-rw-r--r-- 1 root root     0  7월  9 10:12 testF

[root@Server-A ~]# chown  auser1  /gitA/auser1/owner/passwd

 [root@Server-A ~]# ls -l /gitA/auser1/owner/
합계 8
-rw-r--r-- 1 root   root  1262  7월  9 10:12 group
-rw-r--r-- 1 auser1 root 3244  7월  9 10:12 passwd	# 소유주 = auser1 , 소유 그룹 = root
drwxr-xr-x 2 root   root      6  7월  9 10:12 testD
-rw-r--r-- 1 root   root      0  7월  9 10:12 testF

	#현재 pc 기준

root@Server-A:~# ll /gitA/gitA_User1/owner/
합계 8
-rw-r--r-- 1 root root 1255  7월  9 10:12 group
-rw-r--r-- 1 root root 3230  7월  9 10:12 passwd
drwxr-xr-x 2 root root    6  7월  9 10:12 testD
-rw-r--r-- 1 root root    0  7월  9 10:12 testF

root@Server-A:~# chown auser1 /gitA/gitA_User1/owner/passwd 

root@Server-A:~# ll /gitA/gitA_User1/owner/
합계 8
-rw-r--r-- 1 root   root 1255  7월  9 10:12 group
-rw-r--r-- 1 auser1 root 3230  7월  9 10:12 passwd
drwxr-xr-x 2 root   root    6  7월  9 10:12 testD
-rw-r--r-- 1 root   root    0  7월  9 10:12 testF
```

### EX1-4) '/gitA/auser1/owner' 디렉터리안의 'group' 파일의 소유주를 'auser2'로 변경해야한다.

```
[root@Server-A ~]# chown  auser2  /gitA/auser1/owner/group

[root@Server-A ~]# ls -l /gitA/auser1/owner/
합계 8
-rw-r--r-- 1 auser2 root 1262  7월  9 10:12 group		# 소유주 = auser2 , 소유 그룹 = root
-rw-r--r-- 1 auser1 root 3244  7월  9 10:12 passwd
drwxr-xr-x 2 root   root    6  7월  9 10:12 testD
-rw-r--r-- 1 root   root    0  7월  9 10:12 testF
```

---

### EX2-1) 아래의 조건에 맞게 설정하시오
- '/home'디렉터리에 'git' 계정을 생성하시오 (passwod = 1234)
- '/home'디렉터리에 'global' 계정을 생성하시오 (passwod = 1234)
- '/home'디렉터리에 'it' 계정을 생성하시오 (passwod = 1234)

```
[root@Server-A ~]# useradd -D
GROUP=100
HOME=/gitHQ
INACTIVE=-1
EXPIRE=
SHELL=/bin/bash
SKEL=/etc/skel
CREATE_MAIL_SPOOL=yes

[root@Server-A ~]# vi  /etc/default/useradd
GROUP=100
HOME=/home	# /gitHQ를  /home으로 변경
INACTIVE=-1
EXPIRE=
SHELL=/bin/bash
SKEL=/etc/skel
CREATE_MAIL_SPOOL=yes

[root@Server-A ~]# useradd git
[root@Server-A ~]# useradd global
[root@Server-A ~]# useradd it

[root@Server-A ~]# ls  -lt  /home
합계 4
drwx------   3 it         it          131  7월  9 10:42 it	# 계정 생성 확인
drwx------   3 global   global    131  7월  9 10:42 global	# 계정 생성 확인
drwx------   3 git        git        131  7월  9 10:42 git	# 계정 생성 확인
drwx------   3 rootsub rootsub  131  7월  8 15:33 rootsub
drwx------   3 subroot subroot  131  7월  8 14:40 subroot
drwx------   3 subroot sol       131  7월  8 13:04 sol
drwx------   3 soluser soluser  110  7월  8 11:16 soluser
drwx------   4 user9   user9    143  7월  7 15:59 user9
drwx------   3 guest1  guest1   110  7월  7 15:44 guest1
drwx------   3 user5   user5    110  7월  7 14:50 user5
drwx------   3 user4   user4    110  7월  7 14:40 user4
drwx------. 16 guest   guest   4096  7월  7 10:34 guest

[root@Server-A ~]# tail  -3  /etc/passwd
git:x:1303:1303::/home/git:/bin/bash
global:x:1304:1304::/home/global:/bin/bash
it:x:1305:1305::/home/it:/bin/bash
```

### EX2-2) 아래의 조건에 맞게 root 계정을 사용해서 설정하시오
- '/home' 디렉터리안에 'imsi' 디렉터리를 생성하시오
- 'imsi' 디렉터리의 허가권을 777로 변경하시오
- '/etc/a*' 파일을 'imsi' 디렉터리로 복사하시오
- '/etc/b*' 파일을 'imsi' 디렉터리로 복사하시오

```
[root@Server-A ~]# mkdir  /home/imsi

[root@Server-A ~]# chmod 777  /home/imsi

[root@Server-A ~]# ls  -l  /home  | grep imsi
drwxrwxrwx   2 root    root    4096  7월  9 10:48 imsi

[root@Server-A ~]# cp  /etc/a*  /etc/b*  /home/imsi

[root@Server-A ~]# ls  -lt  /home
합계 8
drwxrwxrwx   2 root    root     4096  7월  9 10:48 imsi	# 허가권 : 777 (rwx rwx rwx)
drwx------   3 it         it          131  7월  9 10:42 it
drwx------   3 global   global    131  7월  9 10:42 global
drwx------   3 git       git         131  7월  9 10:42 git
drwx------   3 rootsub rootsub  131  7월  8 15:33 rootsub
drwx------   3 subroot subroot  131  7월  8 14:40 subroot
drwx------   3 subroot sol        131  7월  8 13:04 sol
drwx------   3 soluser soluser   110  7월  8 11:16 soluser
drwx------   4 user9   user9      143  7월  7 15:59 user9
drwx------   3 guest1  guest1    110  7월  7 15:44 guest1
drwx------   3 user5   user5      110  7월  7 14:50 user5
drwx------   3 user4   user4      110  7월  7 14:40 user4
drwx------. 16 guest   guest     4096  7월  7 10:34 guest

[root@Server-A ~]# ls  -l  /home/imsi/
합계 72
-rw-r--r-- 1 root root    16  7월  9 10:48 adjtime
-rw-r--r-- 1 root root  1529  7월  9 10:48 aliases
-rw-r--r-- 1 root root   541  7월  9 10:48 anacrontab
-rw-r--r-- 1 root root   269  7월  9 10:48 anthy-unicode.conf
-rw-r--r-- 1 root root   833  7월  9 10:48 appstream.conf
-rw-r--r-- 1 root root    55  7월  9 10:48 asound.conf
-rw-r--r-- 1 root root     1  7월  9 10:48 at.deny
-rw-r--r-- 1 root root  2658  7월  9 10:48 bashrc
-rw-r--r-- 1 root root   535  7월  9 10:48 bindresvport.blacklist
-rw-r----- 1 root root    33  7월  9 10:48 brlapi.key
-rw-r--r-- 1 root root 28974  7월  9 10:48 brltty.conf
```

### EX2-3) root 계정을 사용해서 아래의 조건에 맞게 설정하시오
- 'imsi' 디렉터리안의 'aliases' 파일의 소유주를 'guest'로 소유 그룹을 'global'로 변경해야한다.
- 'aliases' 파일의 허가권을 '770'으로 변경해야한다.

```
[root@Server-A ~]# chown  guest:global  /home/imsi/aliases	# 소유주 = guest , 소유 그룹 = global

[root@Server-A ~]# chmod  770  /home/imsi/aliases		# 허가권 770 (rwx rwx ---)

[root@Server-A ~]# ls  -l  /home/imsi/
합계 72
-rw-r--r-- 1 root  root      16  7월  9 10:48 adjtime
-rwxrwx--- 1 guest global  1529  7월  9 10:48 aliases		# 허가권 770 (rwx rwx ---) , 소유주 = guest , 소유 그룹 = global
-rw-r--r-- 1 root  root      541  7월  9 10:48 anacrontab
-rw-r--r-- 1 root  root      269  7월  9 10:48 anthy-unicode.conf
-rw-r--r-- 1 root  root      833  7월  9 10:48 appstream.conf
-rw-r--r-- 1 root  root       55  7월  9 10:48 asound.conf
-rw-r--r-- 1 root  root         1  7월  9 10:48 at.deny
-rw-r--r-- 1 root  root    2658  7월  9 10:48 bashrc
-rw-r--r-- 1 root  root      535  7월  9 10:48 bindresvport.blacklist
-rw-r----- 1 root  root       33  7월  9 10:48 brlapi.key
-rw-r--r-- 1 root  root   28974  7월  9 10:48 brltty.conf
```

### EX2-4) 아래의 조건에 맞게 설정하시오
- 'guest' 계정 사용자는 '/home/imsi' 디렉터리안의 'aliases' 파일의 내용을 변경할수 있는가?
- 'global' 계정 사용자는 '/home/imsi' 디렉터리안의 'aliases' 파일의 내용을 변경할수 있는가?
- 'auser1' 계정 사용자는 '/home/imsi' 디렉터리안의 'aliases' 파일의 내용을 변경할수 있는가?

```
[root@Server-A ~]# ls  -l  /home/imsi/aliases
-rwxrwx--- 1 guest global 1529  7월  9 10:48 /home/imsi/aliases
```

- guest 계정 사용자는 aliases 파일에 대해서 소유주이므로 해당 파일을 변경할 수 있다. (허가권: rwx rwx ---)
- global 계정 사용자는 aliases 파일에 대해서 소유주는 아니지만 소유 그룹에 포함되기때문에 해당 파일을 변경할 수 있다. (허가권: rwx rwx ---)
- auser1: 소유주도 아니고 소유 그룹에도 속하지 않아 other 권한(---)이 적용되어 변경할 수 없다.

```
	# guest 계정을 사용하여 파일안의 내용 변경 후 저장 가능
[guest@Server-A ~]$ vi  /home/imsi/aliases

	# global 계정을 사용하여 파일안의 내용 변경 후 저장 가능
[global@Server-A ~]$ vi  /home/imsi/aliases

~
~
~
~
"/home/imsi/aliases" [Permission Denied]
```

### EX2-5) 아래의 조건에 맞게 설정하시오
- 2-5.1) 'root' 계정을 사용해서 '/etc/networks/' 파일을 'auser1' 계정의 홈 디렉터리로 복사
- 2-5.2) 'root' 계정을 사용해서 'networks'의 소유주를 'auser1'으로 소유 그룹을 'gitHQ'로 변경해야 한다.
- 2-5.3) 'auser1' 계정을 사용해서 'networks' 파일의 소유 그룹을 'gitA'로 변경해야 한다.
- 2-5.4) 'auser1' 계정을 사용해서 'GIT-A-Manual' 파일의 소유 그룹을 'gitB'로 변경해야 한다.

```
[root@Server-A ~]# cp  /etc/networks  /gitA/auser1/

[root@Server-A ~]# ls  -lt  /gitA/auser1/
합계 8
-rw-r--r-- 1 root   root    58  7월  9 11:05 networks
drwxr-xr-x 3 root   root    59  7월  9 10:12 owner
-rw-r--r-- 1 auser1 gitA     0  7월  9 09:47 GIT-A-Manual
-rw-r--r-- 1 auser1 gitA 2644  7월  7 14:37 manual.txt

[root@Server-A ~]# chown  auser1:gitHQ  /gitA/auser1/networks

[root@Server-A ~]# ls  -l  /gitA/auser1 | grep networks
-rw-r--r-- 1 auser1 gitHQ   58  7월  9 11:05 networks		# 소유쥬 = auser1 , 소유 그룹 = gitHQ
```

- 2-5.3) 'auser1' 계정을 사용해서 'networks' 파일의 소유 그룹을 'gitA'로 변경해야 한다.
  - 소유주는 root 계정으로만 변경이 가능하다.
  - 소유 그룹은 root 계정과 일반 사용자 계정에서 변경이 가능하다.
  - 일반 사용자 계정에서 소유 그룹을 변경시에는 조건이 필요한다.
    - 해당 파일의 소유주여야 한다.
    - 변경하려는 소유 그룹에 자신이 포함되어있어야 한다.

```
[root@Server-A ~]# id  auser1
uid=1200(auser1) gid=1220(gitA) groups=1220(gitA),1333(groupB)

[root@Server-A ~]# ls  -l  /gitA/auser1 | grep networks
-rw-r--r-- 1 auser1 gitHQ   58  7월  9 11:05 networks		# 소유쥬 = auser1 , 소유 그룹 = gitHQ	

[root@Server-A ~]# chown  auser1:gitA  /gitA/auser1/networks
		~~~~ OR ~~~~
[root@Server-A ~]# chown  :gitA  /gitA/auser1/networks

[root@Server-A ~]# ls  -l  /gitA/auser1 | grep networks
-rw-r--r-- 1 auser1 gitA   58  7월  9 11:05 networks		# 소유쥬 = auser1 , 소유 그룹 = gitA
```

- 2-5.4) 'auser1' 계정을 사용해서 'GIT-A-Manual' 파일의 소유 그룹을 'groupB'로 변경해야 한다.

```
[root@Server-A ~]# id  auser1
uid=1200(auser1) gid=1220(gitA) groups=1220(gitA),1333(groupB)

[auser1@Server-A ~]$ ls  -l
합계 8
-rw-r--r-- 1 auser1 gitA     0  7월  9 09:47 GIT-A-Manual
-rw-r--r-- 1 auser1 gitA 2644  7월  7 14:37 manual.txt
-rw-r--r-- 1 auser1 gitA    58  7월  9 11:05 networks
drwxr-xr-x 3 root   root    59  7월  9 10:12 owner

[auser1@Server-A ~]$ chown  :groupB  /gitA/auser1/GIT-A-Manual

root@Server-A:~# ll /gitA/gitA_User1/
합계 4
drwxr-xr-x 2 auser1 gitA    26  7월  8 15:13 GIT-A
-rw-r--r-- 1 root   groupB   0  7월  9 11:41 GIT-A-Maunal
-rw-r--r-- 1 auser1 gitA    58  7월  9 11:06 networks
drwxr-xr-x 3 root   root    59  7월  9 10:12 owner
drwxr-xr-x 3 auser1 gitA   110  7월  8 15:09 skel
```

### EX2-6) 아래의 조건에 맞게 설정하시오
- '/home/imsi' 디렉터리안의 'bashrc' 파일의 소유주는 'guest' , 소유그룹을 'global'로 변경해야한다.
- '/home/imsi' 디렉터리안의 'adjtime' 파일의 소유주를 'git'로 변경해야한다.
- '/home/imsi' 디렉터리안의 'aliases' 파일의 소유그룹을 'it'로 변경해야한다.
- '/home/imsi' 디렉터리안의 'at.deny' 파일의 소유주, 소유그룹을 'git'로 변경해야한다.

```
[root@Server-A ~]# chown  guest:global  /home/imsi/bashrc	# 1) GID:UID로 설정시 UID와 GID가 모두 변경된다.

[root@Server-A ~]# chown  git  /home/imsi/adjtime		# 2) UID만 설정시 UID만 변경 (:는 생략)

[root@Server-A ~]# chown  :it  /home/imsi/aliases		# 3) :GID만 설정시 GID만 변경된다.

[root@Server-A ~]# chown  git: /home/imsi/at.deny		# 4) UID: 로 설정시 설정된 UID로 변경되면 소유 그룹은 기본 그룹으로 변경된다.

[root@Server-A ~]# ls  -l  /home/imsi/
합계 72
-rw-r--r-- 1 git   root         16  7월  9 10:48 adjtime		# 2) 소유주로 설정시 소유주만 변경된다.
-rwxrwx--- 1 guest it       1385  7월  9 11:01 aliases		# 3) :소유그룹으로 설정시 소유 그룹만 변경된다.
-rw-r--r-- 1 root  root      541  7월  9 10:48 anacrontab
-rw-r--r-- 1 root  root      269  7월  9 10:48 anthy-unicode.conf
-rw-r--r-- 1 root  root      833  7월  9 10:48 appstream.conf
-rw-r--r-- 1 root  root       55  7월  9 10:48 asound.conf
-rw-r--r-- 1 git    git           1  7월  9 10:48 at.deny		# 4) 소유주:  로 설정시 소유주가 변경되며 소유 그룹은 해당 UID의 기본 그룹에 소속된다.
-rw-r--r-- 1 guest global  2658  7월  9 10:48 bashrc		# 1) 소유주:소유그룹으로 설정시 소유주와 소유 그룹이 모두 변경된다.
-rw-r--r-- 1 root  root      535  7월  9 10:48 bindresvport.blacklist
-rw-r----- 1 root  root       33  7월  9 10:48 brlapi.key
-rw-r--r-- 1 root  root   28974  7월  9 10:48 brltty.conf
```

**정리**: **소유권**은 소유자(Owner)와 소유 그룹(Group)으로 구성되며, `chown [UID]:[GID]`로 둘 중 하나만 또는 둘 다 변경할 수 있고, 일반 사용자가 소유 그룹을 바꾸려면 자신이 파일 소유주이면서 대상 그룹에 속해 있어야 한다.

---

## 특수 권한 Umask

- u : user
- mask : 가리는 값 (권한을 제한하는 값)
- 관리자 계정 또는 일반 사용자 계정을 사용하여 파일 및 디렉터리를 생성시 적용되는 허가권의 기본값을 의미한다.
- 리눅스에서 새 파일을 만들면, 모든 사용자에게 무조건 풀 권한(모든 접근)을 주면 위험하다. 그래서 시스템은 "기본적으로 허용할 권한"을 먼저 정하고, 그중 일부를 자동으로 차단(mask) 하는 게 바로 umask (User Mask)이다.
- 파일 또는 디렉터리의 허가권값은 '000 ~ 777' 범위내에서 수동으롤 변경이 가능하다. (--------- ~ rwxrwxrwx)
- 디렉터리의 허가권 = 000 ~ 777 (--------- ~ rwxrwxrwx)
  - 디렉터리는 디렉터리 내부로 이동하는 권한이 'x' 권한이 필요하기때문에 7까지 허가권이 가능하다.
- 파일의 허가권 = 000 ~ 666 (--------- ~ rw-rw-rw-)
  - 파일은 기본적으로 문서파일로 생성되지 않가 때문에 'x' 권한이 부여되지 않기때문에 허가권이 6까지만 가능하다. (실행 파일인 경우 수동으로 허가권을 7로 변경해야한다.)
- 기본 권한 - umask = 최종 권한
- 디렉터리의 기본 권한: 777 (rwxrwxrwx)
- 파일의 기본 권한: 666 (rw-rw-rw-)

- 관리자 계정으로
  - 디렉터리 생성시 허가권: 755
  - 파일 생성시 허가권: 644

```
[root@localhost ~]# mkdir ABC
[root@localhost ~]# touch 1234
[root@localhost ~]# ls -l
합계 4
-rw-r--r-- 1 root root    0  4월 20 13:45 1234	: 파일 = 644
drwxr-xr-x 2 root root 4096  4월 20 13:45 ABC	: 디렉터리 = 755
```

- 일반 사용자 계정
  - 디렉터리 생성시 허가권: 755
  - 파일 생성시 허가권: 644

```
[guest@localhost ~]$ mkdir ABC
[guest@localhost ~]$ touch 1234
[guest@localhost ~]$ ls -l
합계 4
-rw-rw-r-- 1 guest guest      0  4월 20 13:49 1234		: 파일 = 664
drwxrwxr-x 2 guest guest 4096  4월 20 13:49 ABC		: 디렉터리 = 775
```

- 관리자 계정 umask = 0022
- 사용자 계정 umask = 0022

```
[root@localhost ~]# umask	   <---- 관리자 계정 umask
0022
```

- 디렉터리의 허가권은 최대값이 777이므로 umask를 적용하게되면 허가권이 755가된다.
- 파일의 허가권은 최대값이 666이므로 umask를 적용하게되면 허가권이 644가된다.

```
[guest@localhost ~]$ umask	   <---- 사용자 계정 umask
0022
```

- 디렉터리의 허가권은 최대값이 777이므로 umask를 적용하게되면 허가권이 755가된다.
- 파일의 허가권은 최대값이 666이므로 umask를 적용하게되면 허가권이 644가된다.

### umask EX1 ~ EX4

### EX1) 관리자 권한의 umask값을 0002로 변경시
- 관리자 권한의 umask값을 0002로 변경 후 디렉터리의 생성시 기본 허가권은?
- 관리자 권한의 umask값을 0002로 변경 후 파일 생성시 기본 허가권은?

```
[root@Server-A ~]# umask 0002

[root@Server-A ~]# umask
0002

[root@Server-A ~]# mkdir 0002D
[root@Server-A ~]# touch 0002F

[root@Server-A ~]# ls -ld 0002*
drwxrwxr-x 2 root root 6  7월  9 12:13 0002D	# rwx rwx r-x = 775
-rw-rw-r-- 1 root root 0  7월  9 12:13 0002F	# rw- rw- r-- = 664
```

### EX2) 관리자 권한의 umask값을 0022로 변경시
- 관리자 권한의 umask값을 0022로 변경 후 디렉터리의 생성시 기본 허가권은? → rwx r-x r-x
- 관리자 권한의 umask값을 0022로 변경 후 파일 생성시 기본 허가권은? → rw- r-- r--

```
=================================================
				umask 0022						|
=================================================
			 d = 	rwx	rwx	rwx					|
			 umask	---	-w-	-w-					|
				-------------------				|
					rwx	r-x	r-x	= 755			|
												|
=================================================
												|
			 f = 	rw-	rw-	rw-					|
			 umask	---	-w-	-w-					|
				-----------------------			|
					rw-	r--	r--	= 644			|
												|
=================================================
```

```
[root@Server-A ~]# umask  0022	# umask 변경

[root@Server-A ~]# umask		# umask 확인

[root@Server-A ~]# mkdir 0022D
[root@Server-A ~]# touch 0022F

[root@Server-A ~]# ls  -ld  ./0022*
drwxr-xr-x 2 root root 6  7월  9 12:34 ./0022D	# 디렉터리 허가권	: rwx r-x r-x = 755
-rw-r--r-- 1 root root 0  7월  9 12:34 ./0022F	# 파일 허가권	: rw- r--r -- = 644
[root@Server-A ~]#
```

### EX3) 관리자 권한의 umask값을 0011로 변경시
- 관리자 권한의 umask값을 0011로 변경 후 디렉터리의 생성시 기본 허가권은? → rwx rw- rw-
- 관리자 권한의 umask값을 0011로 변경 후 파일 생성시 기본 허가권은? → -wx rw- rw-

```
=================================================
					umask 0011					|
=================================================
			 d = 	rwx	rwx	rwx					|
			 umask	---	--x	--x					|
				-----------------------			|
					rwx	rw-	rw-	= 766			|
												|
=================================================
												|
			 f = 	rw-	rw-	rw-					|
			 umask	---	--x	--x					|
				-----------------------			|
					rw-	r--	r--	= 644			|
												|
=================================================
```

```
root@Server-A:~# umask 0011

root@Server-A:~# umask 
0011

root@Server-A:~# mkdir 0011D
root@Server-A:~# touch 0011F

root@Server-A:~# ll -d ./0011*
drwxrw-rw- 2 root root 6  7월  9 12:39 0011D	# 디렉터리 허가권	: rwx rw- rw- = 766
-rw-rw-rw- 1 root root 0  7월  9 12:39 0011F	# 파일 허가권	: rw- rw- rw- = 666
```

### EX4) 관리자 권한의 umask값을 0033로 변경시
- 관리자 권한의 umask값을 0033로 변경 후 디렉터리의 생성시 기본 허가권은? → rwx r-- r--
- 관리자 권한의 umask값을 0033로 변경 후 파일 생성시 기본 허가권은? → -wx r-- r--

```
=================================================
				umask 0033					  	|
=================================================
				 d = 	rwx	rwx	rwx				|
				 umask	---	-wx	-wx				|
					-----------------------		|
						rwx	r--	r--	= 744		|
												|
=================================================
												|
				 f = 	rw-	rw-	rw-				|
				 umask	---	-wx	-wx				|
					-----------------------		|
						rw-	r--	r--	= 644		|
												|
=================================================
```

```
root@Server-A:~# umask 0033

root@Server-A:~# umask
0033

root@Server-A:~# mkdir 0033D
root@Server-A:~# touch 0033F

root@Server-A:~# ll -d ./0033*
drwxr--r-- 2 root root 6  7월  9 12:40 ./0033D  # 디렉터리 허가권	: rwx r-- r-- = 744
-rw-r--r-- 1 root root 0  7월  9 12:41 ./0033F  # 디렉터리 허가권	: -wx r-- r-- = 644
```

**정리**: **umask**는 디렉터리 최대값 777, 파일 최대값 666에서 값을 차감하여 신규 생성 시의 기본 허가권을 결정하며, 기본값 `0022`는 디렉터리 755/파일 644를 만들어낸다.

---

## 특수 권한 (Set-UID / Set-GID / Sticky-bit)

- Linux의 권한 체계는 3계층(소유주, 그룹, 그외 사용자)으로 구성되며 각각의 계층마다 3가지 권한(읽기, 쓰기, 실행)을 부여하는 방식으로 구성된다. 하지만 정해진 시스템으로인해 시스템 운영에서 문제가 발생하는 경우가 존재하기때문에 정해진 체계이외의 특수한 환경에서 사용하도록 만들어진 기능이 특수 권한이다.
- root의 권한이 필요한 기능이지만 관리자가 비밀번호를 알려줄수 없으며 권한을 부여해서도 안된다. root의 일부 권한을 부여하거나 또는 한시적으로 권한을 부여하는 기능을 특수 권한이라고 한다.
- 특수 권한은 Set-UID, Set-GID, Sticky-bit로 구성된다.

- **Set-UID (4777)**
  - 일반적으로 실행 파일에 적용되며 Set-UID가 적용된 파일을 실행시 해당 파일을 실행시킨 사용자에게 해당 파일이 실행되는 동안 한시적으로 소유주의 권한이 부여된다.
  - 실행 파일이 종료시 권한도 함께 중단된다.
  - 허가권의 표기는 소유주의 허가권 자리에 'x'가 's'로 표기된다. (rws rwx rwx)

- **Set-GID (2777, 3777)**
  - Set-GID도 Set-UID처럼 파일에 적용된 경우 해당 파일의 소유 그룹으로 인식된다.
  - Set-GID는 일반적으로 디렉터리에 적용되며 Set-GID가 설정된 디렉터리에 사용자가 파일이나 디렉터리를 생성하게되면 사용자가 속한 그룹과 관계없이 해당 디렉터리의 소유 그룹으로 만들어진다.
  - 허가권의 표기는 그룹의 허가권 자리에 'x'가 's'로 표기된다. (rwx rws rwx)

- **Sticky-bit (1777)**

### Set-UID 실습

### EX1) 아래의 조건에 맞게 설정하시오
- '/backup' 디렉터리내의 모든 파일을 삭제하시오
- '/usr/bin/passwd'파일을 '/backup' 디렉터리로 복사해야한다. (파일 복사시 모든 속성은 변경되지 않아야한다.)

```
[root@Server-A ~]# ls  -l  /usr/bin/passwd
-rwsr-xr-x. 1 root root 32656  5월 15  2022 /usr/bin/passwd

[root@Server-A ~]# rm -rf /backup/*

[root@Server-A ~]# cp  -p  /usr/bin/passwd  /backup

[root@Server-A ~]# ls  -l  /backup
합계 32
-rwsr-xr-x 1 root root 32656  5월 15  2022 passwd

[guest@Server-A ~]$ passwd
guest 사용자의 비밀 번호 변경 중
Current password:1234
새 암호:soldesk1234
새 암호 재입력:soldesk1234
passwd: 모든 인증 토큰이 성공적으로 업데이트 되었습니다.
[guest@Server-A ~]$
```

```
==============================================================================================================================
        chmod	|        허가권	|            passwd 실행		 |		passwd  변경												|
==============================================================================================================================
				|				|							| guest 계정은 /usr/bin/passwd 파일에 대해서 other 권한이 		|
 # chmod 750	|  rwx rwx ---	|   해당 파일을 실행 X		| 적용되기때문에  x 권한이 없다.									|
				|				|							| x 권한이 없기때문에 해당 파일을 실행할 수 없다.					|
==============================================================================================================================
				|				|							| /usr/bin/passwd 파일에 대해서 other 권한이 적용되어				|
 # chmod 755	|  rwx r-x r-x	|  해당 파일을 실행 	O		| x 권한으로 실행은 가능하지만 w 권한이 없기때문에 password를		|
				|				|							| 변경할 수는 없다.												|
==============================================================================================================================
		|		|				|							| /usr/bin/passwd 파일에 대해서 other 권한이 적용되어				|
 # chmod 4755	|	rws r-x r-x	|	해당 파일을 실행 	O		| 특수권한으로 파일이 실행되는동안 임시 관리권한을 수행할 수 있다.	|
		|		|				|							|																|
==============================================================================================================================
```

```
	# 허가권 750

[root@Server-A ~]# chmod 750  /usr/bin/passwd

[root@Server-A ~]# ls  -l  /usr/bin/passwd
-rwxr-x---. 1 root root 32656  5월 15  2022 /usr/bin/passwd		# 허가권 = rwx r-x ---

[guest@Server-A ~]$ passwd		# passwd를 실행할 수 없기때문에 비밀번호를 수정할 수 없다.
-bash: /usr/bin/passwd: 허가 거부


	# 허가권 755

[root@Server-A ~]# chmod 755  /usr/bin/passwd

[root@Server-A ~]# ls  -l  /usr/bin/passwd
-rwxr-xr-x. 1 root root 32656  5월 15  2022 /usr/bin/passw		# 허가권 = rwx r-x r-x

[guest@Server-A ~]$ passwd
guest 사용자의 비밀 번호 변경 중
Current password:soldesk1234
새 암호:studydesk1234
새 암호 재입력:studydesk1234
passwd: 인증 토근 수정 오류


	# 허가권 4755 (특수권한 4XXX)

[root@Server-A ~]# chmod 4755  /usr/bin/passwd

[root@Server-A ~]# ls  -l  /usr/bin/passwd
-rwsr-xr-x. 1 root root 32656  5월 15  2022 /usr/bin/passwd		# 허가권 = rws r-x r-x

[guest@Server-A ~]$ passwd
guest 사용자의 비밀 번호 변경 중
Current password:soldesk1234
새 암호:dlatlqlalfqjsgh
새 암호 재입력:dlatlqlalfqjsgh
passwd: 모든 인증 토큰이 성공적으로 업데이트 되었습니다.
```

- /usr/bin/passwd 파일에 Set-UID 권한이 설정되어 있으면 일반 사용자가 passwd 명령어를 실행할 때 해당 프로그램은 파일 소유주인 root의 권한으로 실행된다. 따라서 guest 사용자는 /etc/shadow 파일을 직접 수정할 수 없지만 passwd 프로그램이 실행되는 동안에는 root 권한으로 동작하므로 자신의 비밀번호를 변경할 수 있다.
- Set-UID는 일반 사용자에게 /usr/bin/passwd 파일의 w 권한을 실제 부여하는 것이 아니라, 실행 중인 프로세스의 유효 사용자 권한을 파일 소유주 권한으로 변경하는 기능이다.

---

### Sticky-bit

- 리눅스에서 특정 디렉터리를 여러 사용자가 함께 사용하는 "공유 디렉터리"로 설정할 때, 보통 허가권을 777로 설정한다. (rwxrwxrwx) 이 경우 모든 사용자가 해당 디렉터리 안에서 파일과 디렉터리를 자유롭게 생성, 수정, 삭제할 수 있다. 그러나 777 상태에서는 문제점은 다른 사용자가 만든 파일이라도, 누구나 해당 파일을 삭제하거나 이름 변경할 수 있다. (예: A 사용자가 만든 파일을 B 사용자가 마음대로 삭제 가능) 이러한 보안 문제를 해결하기 위해 사용하는 것이 Sticky-bit이다.

- Sticky-bit를 설정한 디렉터리의 동작
  1. A 사용자는 Sticky-bit가 설정된 디렉터리 안에서 자신의 파일과 디렉터리를 생성, 수정, 삭제할 수 있다.
  2. B 사용자도 동일하게 자신의 파일을 생성, 수정, 삭제할 수 있다.
     - 단, A 사용자는 B 사용자의 파일을 읽기/복사 여부는 파일 퍼미션(r 권한)에 따라 다를수 있다.
     - 삭제/이름 변경(rename): 절대 불가능
     - (B도 A의 파일을 삭제하거나 이름 변경할 수 없다.)
  3. Sticky-bit는 공유 디렉터리에서 남의 파일 삭제를 막는 기능이다. 파일 내용 수정 여부는 파일 자체의 권한(r, w)에 의해 결정되며 Sticky-bit와는 무관하다.

### EX1) 아래의 조건에 맞게 설정하시오
- 'root' 계정을 사용하여 최상위 디렉터리에 공유 폴더 개념의 '/share' 디렉터리 생성해야한다.
- 'root' 계정을 사용하여 '/share' 디렉터리의 허가권을 777로 변경해야한다.

```
[root@Server-A ~]# mkdir  /share

[root@Server-A ~]# chmod  777  /share/

[root@Server-A ~]# ls  -l  / | grep share
drwxrwxrwx    2 root root    6  7월  9 15:32 share # 허가권 777 (rwx rwx rwx)

 # guest 계정
 # sol 계정
```

```
======================================================================================================
    chmod 777	|    자신이 소유주인 파일 및 디렉터리		|  	other 권한의 파일 및 디렉터리			|
======================================================================================================
				|										|										|
 # guest		| guest 계정으로 파일, 디렉터리 생성 O	| nam 계정으로 guest 소유주의 파일 읽기 O	|
				| guest 계정으로 파일, 디렉터리 수정 O	| nam 계정으로 guest 소유주의 파일 복사 O	|
				| guest 계정으로 파일, 디렉터리 삭제 O	| nam 계정으로 guest 소유주의 파일 수정 O	|
				|										| nam 계정으로 guest 소유주의 파일 삭제 O	|
				|										|										|
======================================================================================================
```

```
    # guest 계정을 사용하여 파일 및 디렉터리 생성
[guest@Server-A ~]$ mkdir /share/guestD
[guest@Server-A ~]$ touch /share/guestF

[guest@Server-A ~]$ ls -ld /share/guest*
drwxr-xr-x 2 guest guest 6  7월  9 15:43 /share/guestD
-rw-r--r-- 1 guest guest 0  7월  9 15:43 /share/guestF

[nam@Server-A ~]$ rm -rf /share/guestD	# ryu 계정으로 디렉터리 삭제
[nam@Server-A ~]$ rm -rf /share/guestF	# ryu 계정으로 파일 삭제

[nam@Server-A ~]$ ls  -l  /share/
합계 0

   # guest 계정을 사용하여 파일 및 디렉터리 생성
[guest@Server-A ~]$ mkdir /share/guestD
[guest@Server-A ~]$ touch /share/guestF

[nam@Server-A ~]$ mv  /share/guestD  /share/ryuD
[nam@Server-A ~]$ mv  /share/guestF  /share/ryuF

[nam@Server-A ~]$ ls  -l  /share/
합계 0
drwxr-xr-x 2 guest guest 6  7월  9 15:46 ryuD
-rw-r--r-- 1 guest guest 0  7월  9 15:46 ryuF
```

```
======================================================================================================
    chmod 1777	|    자신이 소유주인 파일 및 디렉터리		|  	other 권한의 파일 및 디렉터리				|
======================================================================================================
				|										|											|
 # guest		| guest 계정으로 파일, 디렉터리 생성 O	| nam 계정으로 guest 소유주의 파일 읽기 O		|
				| guest 계정으로 파일, 디렉터리 수정 O	| nam 계정으로 guest 소유주의 파일 복사 O		|
				| guest 계정으로 파일, 디렉터리 삭제 O	| nam 계정으로 guest 소유주의 파일 이름 수정 X	|
				|										| nam 계정으로 guest 소유주의 파일 삭제 X		|
				|										|											|
======================================================================================================
```

```
[root@Server-A ~]# chmod  1777  /share/

[root@Server-A ~]# ls  -l  /  | grep share
drwxrwxrwt    3 root root   30  7월  9 15:46 share
```

### EX1) 아래의 조건에 맞게 설정해야 한다.
- 'guest' 계정을 사용하여 '/share' 디렉터리에 guestF 파일과 guestD 디렉터리를 생성해야 한다.
- 'nam' 계정을 사용하여 '/share' 디렉터리의 guestF 파일의 내용을 확인
- 'nam' 계정을 사용하여 '/share' 디렉터리의 guestF 파일을 자신의 홈 디렉터리로 복사
- 'nam' 계정을 사용하여 '/share' 디렉터리의 guestF 파일의 파일명을 'solF'로 변경
- 'nam' 계정을 사용하여 '/share' 디렉터리의 guestF 파일을 삭제

```
[guest@Server-A ~]$ mkdir /share/guestD
[guest@Server-A ~]$ touch /share/guestF

[nam@Server-A ~]$ vi  /share/guestF

[nam@Server-A ~]$ cp  /share/guestF  ./

[nam@Server-A ~]$ ls  -l
합계 4
-rw-r--r-- 1 ryu ryu    0  7월  9 09:46 GIT-HQ-Manual
-rw-r--r-- 1 ryu ryu    0  7월  9 16:00 guestF
-rw-r--r-- 1 ryu ryu 2644  7월  7 14:37 manual.txt

[nam@Server-A ~]$ mv /share/guestF /share/solF
mv: cannot move '/share/guestF' to '/share/solF': 명령을 허용하지 않음

[nam@Server-A ~]$ rm -r /share/guestF
rm: remove write-protected 일반 빈 파일 '/share/guestF'? y
rm: cannot remove '/share/guestF': 명령을 허용하지 않음
```

### 문제1) 아래의 조건에 맞게 설정하시오
- root 계정을 사용하여 'solUser1', 'solUser2', 'solUser3' 계정을 생성해야한다. (홈 디렉터리 '/solhome')
- 'solUser1', 'solUser2', 'solUser3' 계정은 'soldesk' 그룹에 추가 포함되어야 한다. ('soldesk' 그룹이 없으면 생성해야 한다.)
- root 계정을 사용하여 '/solhome/sol_tmp' 디렉터리를 생성하시오
- '/sol_tmp' 디렉터리는 other 권한 사용자는 읽기(r), 쓰기(w), 실행(x) 권한이 없어야한다.
- '/sol_tmp' 디렉터리는 'solUser1', 'solUser2', 'solUser3' 사용자는 '/sol_tmp'디렉터리에 자신의 파일 및 디렉터리를 생성, 수정, 삭제가 가능해야한다.
- 단 자신이 소유주가 아닌 파일 및 디렉터리를 이름 변경, 삭제는 불가능해야한다. (읽기 및 복사는 가능해야한다.)

설정 완료 후 확인 사항:
```
# 'solUser1' 계정으로 '/sol_tmp' 디렉터리에 파일 및 디렉터리 생성
# 'solUser1' 계정으로 '/sol_tmp' 디렉터리에 파일 및 디렉터리 수정
# 'solUser1' 계정으로 '/sol_tmp' 디렉터리에 파일 및 디렉터리 삭제

# 'solUser1' 계정으로 '/sol_tmp' 디렉터리안의 'solUser1' 소유주의 파일 읽기
# 'solUser1' 계정으로 '/sol_tmp' 디렉터리안의 'solUser1' 소유주의 파일 복사
# 'solUser1' 계정으로 '/sol_tmp' 디렉터리안의 'solUser1' 소유주의 파일 수정
# 'solUser1' 계정으로 '/sol_tmp' 디렉터리안의 'solUser1' 소유주의 파일 삭제

# 'solUser2' 계정으로 '/sol_tmp' 디렉터리안의 'solUser1' 소유주의 파일 읽기
# 'solUser2' 계정으로 '/sol_tmp' 디렉터리안의 'solUser1' 소유주의 파일 복사
# 'solUser2' 계정으로 '/sol_tmp' 디렉터리안의 'solUser1' 소유주의 파일 수정
# 'solUser2' 계정으로 '/sol_tmp' 디렉터리안의 'solUser1' 소유주의 파일 삭제

# 'guest' 계정으로 '/sol_tmp' 디렉터리에 파일 및 디렉터리 생성
# 'guest' 계정으로 '/sol_tmp' 디렉터리안의 'solUser1' 소유주의 파일 읽기
# 'guest' 계정으로 '/sol_tmp' 디렉터리안의 'solUser1' 소유주의 파일 복사
# 'guest' 계정으로 '/sol_tmp' 디렉터리안의 'solUser1' 소유주의 파일 수정
# 'guest' 계정으로 '/sol_tmp' 디렉터리안의 'solUser1' 소유주의 파일 삭제
```

```
[root@Server-A ~]#  useradd  -d  /solhome/solUser1  solUser1	# 계정 생성
[root@Server-A ~]#  useradd  -d  /solhome/solUser2  solUser2	# 계정 생성
[root@Server-A ~]#  useradd  -d  /solhome/solUser3  solUser3	# 계정 생성

[root@Server-A ~]#  groupadd soldesk		# 그룹 생성

[root@Server-A ~]#  usermod  -aG  soldesk  solUser1	# solUser1 계정을 soldesk 그룹에 추가 포함
[root@Server-A ~]#  usermod  -aG  soldesk  solUser2	# solUser2 계정을 soldesk 그룹에 추가 포함
[root@Server-A ~]#  usermod  -aG  soldesk  solUser3	# solUser3 계정을 soldesk 그룹에 추가 포함

[root@Server-A ~]# id  solUser1
uid=1307(solUser1) gid=1307(solUser1) groups=1307(solUser1),1334(soldesk)

[root@Server-A ~]# id  solUser2
uid=1308(solUser2) gid=1308(solUser2) groups=1308(solUser2),1334(soldesk)

[root@Server-A ~]# id  solUser3
uid=1309(solUser3) gid=1309(solUser3) groups=1309(solUser3),1334(soldesk)

[root@Server-A ~]# ls  -ld  /solhome/sol*
drwx------ 3 solUser1 solUser1 131  7월  9 16:56 /solhome/solUser1
drwx------ 3 solUser2 solUser2 131  7월  9 16:56 /solhome/solUser2
drwx------ 3 solUser3 solUser3 131  7월  9 16:56 /solhome/solUser3
drwxr-xr-x 2 root     root       6  7월  9 16:57 /solhome/sol_tmp

[root@Server-A ~]# chown root:soldesk /solhome/sol_tmp	# 소유 그룹을 soldesk로 변경

[root@Server-A ~]# ls  -ld  /solhome/sol_tmp/
drwxrwx--T 2 root soldesk 6  7월  9 16:57 /solhome/sol_tmp/	# 소유주 = root , 소유 그룹 = soldesk 

[root@Server-A ~]# chmod 1770 /solhome/sol_tmp	# 공유 디렉터리로 변경 (Sticky-bit)
```

```
# gitUser1 (owner)
[gitUser1@Server-A ~]$ cd /solhome/sol_tmp		O (group = soldesk, x 있음)

[solUser1@Server-A sol_tmp]$ touch file1 file2		O (group에 w,x 있음)
[solUser1@Server-A sol_tmp]$ mkdir dir1 dir2		O (group에 w,x 있음)

[solUser1@Server-A sol_tmp]$ vi file1 으로 내용 수정	O (owner = gitUser1, 파일 퍼미션 644   owner w 가능)

[solUser1@Server-A sol_tmp]$ mv  ./file1  ./file10	O (owner이므로 파일 및 디렉터리 이름 변경 가능)

[solUser1@Server-A sol_tmp]$ rm file1, rm -rf dir1	O (sticky지만 owner이므로 삭제 가능)


# gitUser2 (같은 soldesk 그룹, owner는 아님)
[gitUser2@Server-A ~]$ cd /solhome/sol_tmp		O

[solUser2@Server-A sol_tmp]$ touch file11 file21		O (group에 w,x 있음)
[solUser2@Server-A sol_tmp]$ mkdir dir12 dir22		O (group에 w,x 있음)

[solUser2@Server-A sol_tmp]$ cat file2		 	(group r 권한)

[solUser2@Server-A sol_tmp]$ cp file2  ~/file1.copy	O (읽기 되면 복사 가능)

[solUser2@Server-A sol_tmp]$ vi file2 으로 내용 수정	X (파일 퍼미션 644 → group r-- 이므로 쓰기 불가)

[solUser2@Server-A sol_tmp]$ mv ./file2  ./file200	X (sticky 때문에 owner 아니면 이름 변경 불가)

[solUser2@Server-A sol_tmp]$ rm file2			X (sticky 때문에 owner 아니면 삭제 불가)


# guest (other)
[guest@Server-A ~]$ cd /solhome/sol_tmp 	X (other = ---)
-bash: cd: /solhome/sol_tmp: 허가 거부

[guest@Server-A ~]$ ls  -l  /solhome/sol_tmp/
ls: cannot open directory '/solhome/sol_tmp/': 허가 거부

# 내부 파일 읽기/복사/삭제 : 전부 허가 거부
```

**정리**: **Set-UID**(4xxx)는 실행 중 소유주 권한을 부여하고, **Sticky-bit**(1xxx)는 공유 디렉터리에서 자신이 소유하지 않은 파일의 삭제/이름변경을 막아 `/tmp` 같은 공용 공간을 안전하게 만든다.

---

## Set-GID

- Set-GID (2XXX, 3XXX)
  - Set-GID는 Linux의 특수 권한 중 하나로, 파일이나 디렉터리에 적용할 수 있다.
  - Set-GID도 Set-UID처럼 파일에 적용된 경우 해당 파일의 소유 그룹으로 인식된다.
  - Set-GID는 일반적으로 디렉터리에 적용되며 Set-GID가 설정된 디렉터리에 사용자가 파일이나 디렉터리를 생성하게되면 사용자가 속한 그룹과 관계없이 해당 디렉터리의 소유 그룹으로 만들어진다.
  - 허가권의 표기는 그룹의 허가권 자리에 'x'가 's'로 표기된다. (rwx rws rwx)
- Set-GID는 숫자로 표현할 때 앞자리에 2를 붙여 설정한다.
  - 일반 권한: 775
  - Set-GID: 2775
  - 기존 권한 앞에 2, 3가 붙으면 Set-GID가 된다.

### Set-GID의 기본 개념

- Set-GID는 Set Group ID의 의미를 가진다.
- 일반적으로 사용자가 파일을 생성하면, 새 파일의 소유 그룹은 파일을 만든 사용자의 기본 그룹으로 설정된다.
  - 예를 들어 user1의 기본 그룹이 user1이라면:
  - user1이 파일 생성
    - 파일 소유자: user1
    - 파일 소유그룹: user1
  - 하지만 Set-GID가 설정된 디렉터리 안에서 파일이나 디렉터리를 생성하면, 사용자의 기본 그룹과 관계없이 상위 디렉터리의 소유 그룹을 따라간다.
    - Set-GID 디렉터리 안에서 파일 생성
    - 파일 소유자: 파일을 만든 사용자
    - 파일 소유그룹: 상위 디렉터리의 소유 그룹
- Set-GID가 실행 파일에 적용되면, 해당 파일을 실행하는 사용자는 일시적으로 파일의 소유 그룹 권한으로 실행하게 된다.
  - 파일을 실행한 사용자의 기본 그룹이 아니라, 실행 파일의 소유 그룹 권한으로 동작한다.
  - Set-GID가 설정된 실행 파일 실행: 실행 중에는 파일의 소유 그룹 권한을 사용
- Set-GID가 디렉터리에 적용된 경우
  - Set-GID는 디렉터리에 적용할 때 가장 많이 사용된다.
  - Set-GID가 설정된 디렉터리 안에서 어떤 사용자가 파일이나 디렉터리를 만들더라도, 새로 생성되는 파일과 디렉터리의 소유 그룹은 상위 디렉터리의 소유 그룹으로 자동 설정된다.
  - 예를 들어 /project 디렉터리의 소유 그룹이 teamA이고 Set-GID가 설정되어 있다면:
    - /project 디렉터리 소유 그룹: teamA
    - /project 디렉터리에 Set-GID 설정
    - user1이 파일 생성: 소유그룹 teamA
    - user2가 파일 생성: 소유그룹 teamA
    - user3이 디렉터리 생성: 소유그룹 teamA
  - 즉, 파일을 만든 사용자가 누구든지 소유 그룹은 teamA로 고정된다.

- Set-GID가 필요한 이유
  - 공유 디렉터리나 협업 디렉터리에서는 여러 사용자가 같은 공간에서 파일을 생성한다.
  - Set-GID가 없으면 각 사용자가 만든 파일의 소유 그룹이 서로 다르게 생성될 수 있다.
  - 예)
    - user1이 파일 생성: user1:user1
    - user2가 파일 생성: user2:user2
    - user3이 파일 생성: user3:user3
    - 이렇게 되면 같은 팀 사용자들이 파일을 함께 관리하기 어렵다.
  - 하지만 Set-GID를 설정하면
    - user1이 파일 생성: user1:teamA
    - user2가 파일 생성: user2:teamA
    - user3이 파일 생성: user3:teamA
    - 처럼 소유 그룹이 통일된다.
  - 따라서 팀 프로젝트, 공동 작업 폴더, 부서별 공유 디렉터리에서 사용할 수 있다.

### Set-GID EX1

### EX1) 아래의 조건에 맞게 설정하시오.
- solGroup 그룹을 생성해야 한다.
- userSol1, userSol2, userSol3 계정을 생성해야 한다.
- userSol1, userSol2, userSol3 계정을 solGroup 그룹에 추가 포함시켜야 한다.
- /homesol 디렉터리 안에 sol_tmp1 디렉터리를 생성해야 한다.
- sol_tmp1 디렉터리는 solGroup 그룹에 포함된 계정들이 공동으로 사용해야 한다.
- sol_tmp1 디렉터리 안에서 생성되는 파일 및 디렉터리의 소유 그룹은 모두 solGroup로 생성되어야 한다.
- sol_tmp1 디렉터리 안에서 생성된 파일 및 디렉터리는 solGroup 그룹에 포함된 사용자들이 삭제할 수 있어야 한다.

```
[root@Server-A ~]# groupadd solGroup		# 공용으로 사용할 그룹 생성

[root@Server-A ~]# useradd  -md  /homesol/userSol1  userSol1
[root@Server-A ~]# useradd  -md  /homesol/userSol2  userSol2
[root@Server-A ~]# useradd  -md  /homesol/userSol3  userSol3

[root@Server-A ~]# ls  -l  /homesol/
합계 0
drwx------ 3 userSol1 userSol1 131  7월  9 17:30 userSol1		# 계정 생성 확인
drwx------ 3 userSol2 userSol2 131  7월  9 17:30 userSol2		# 계정 생성 확인
drwx------ 3 userSol3 userSol3 131  7월  9 17:30 userSol3		# 계정 생성 확인

[root@Server-A ~]# tail  -3  /etc/passwd
userSol1:x:1310:1310::/homesol/userSol1:/bin/bash		# 계정 생성 확인
userSol2:x:1311:1311::/homesol/userSol2:/bin/bash		# 계정 생성 확인
userSol3:x:1312:1312::/homesol/userSol3:/bin/bash		# 계정 생성 확인

[root@Server-A ~]# usermod  -aG  solGroup  userSol1	# userSol1 계정을 solGroup 그룹에 추가
[root@Server-A ~]# usermod  -aG  solGroup  userSol2	# userSol1 계정을 solGroup 그룹에 추가
[root@Server-A ~]# usermod  -aG  solGroup  userSol3	# userSol1 계정을 solGroup 그룹에 추가

[root@Server-A ~]# id  userSol1
uid=1310(userSol1) gid=1310(userSol1) groups=1310(userSol1),1335(solGroup)

[root@Server-A ~]# id  userSol2
uid=1311(userSol2) gid=1311(userSol2) groups=1311(userSol2),1335(solGroup)

[root@Server-A ~]# id  userSol3
uid=1312(userSol3) gid=1312(userSol3) groups=1312(userSol3),1335(solGroup)

    # 공용 디렉터리 생성
[root@Server-A ~]# mkdir  /homesol/sol_tmp1

    # 공용 디렉터리의 소유 그룹을 solGroup으로 변경
[root@Server-A ~]# chown  root:solGroup  /homesol/sol_tmp1

[userSol1@Server-A ~]$ mkdir  /homesol/sol_tmp2/aaa
mkdir: `/homesol/sol_tmp2/aaa' 디렉토리를 만들 수 없습니다: 허가 거부		# 디렉터리가 생성되지 않는다.

[userSol1@Server-A ~]$ touch  /homesol/sol_tmp2/bbb			# 파일이 생성되지 않는
touch: cannot touch '/homesol/sol_tmp2/bbb': 허가 거부

[userSol1@Server-A ~]$ ls  -ld  /homesol/sol_tmp2
drwxr--r-- 2 root solGroup 6  7월  9 17:52 /homesol/sol_tmp2		# group에 w 권한이 없기때문에 파일 및 디렉터리를 생성할 수 없다.

[root@Server-A ~]# ls  -l /homesol  | grep sol_tmp1
drwxrws--- 2 root     solGroup   6  7월  9 17:34 sol_tmp1

[root@Server-A ~]# passwd userSol1		# 비밀번호 설정
[root@Server-A ~]# passwd userSol2		# 비밀번호 설정
[root@Server-A ~]# passwd userSol3		# 비밀번호 설정

[root@Server-A ~]# chmod  2770  /homesol/sol_tmp1/

[root@Server-A ~]# ls  -l /homesol  | grep sol_tmp1
drwxrws--- 2 root     solGroup   6  7월  9 17:34 sol_tmp1	# Set-GID = rwx rws ---
```

- chmod 2770 /homesol/sol_tmp1 명령어는 sol_tmp1 디렉터리에 Set-GID를 설정하게되면 2777에서 앞자리 2는 Set-GID를 의미하고, 777은 소유주, 그룹, 기타 사용자 모두에게 rwx 권한을 부여한다.
- Set-GID가 설정되었기 때문에 그룹 권한 부분이 rws로 표시된다.

### EX2-1) userSol1 계정을 사용해서 /homesol/sol_tmp1 디렉터리 안에 파일과 디렉터리를 생성한 후 소유 그룹을 확인하시오.

```
[userSol1@Server-A ~]$ mkdir  /homesol/sol_tmp1/userSol1D
[userSol1@Server-A ~]$ touch  /homesol/sol_tmp1/userSol1F

[userSol1@Server-A ~]$ ls  -l  /homesol/sol_tmp1/
합계 0
drwxr-sr-x 2 userSol1 solGroup 6  7월  9 17:43 userSol1D
-rw-r--r-- 1 userSol1 solGroup 0  7월  9 17:44 userSol1F
```

### EX2-2) userSol2 계정을 사용해서 userSol1이 생성한 파일을 복사, 수정, 삭제 확인

```
	# userSol1이 생성한 파일을 userSol2의 홈 디렉터리로 복사

[userSol2@Server-A ~]$ cp  /homesol/sol_tmp1/userSol1F  ./

[userSol2@Server-A ~]$ ls  -l
합계 4
-rw-r--r-- 1 userSol2 userSol2    0  7월  9 09:46 GIT-HQ-Manual
-rw-r--r-- 1 userSol2 userSol2 2644  7월  7 14:37 manual.txt
-rw-r--r-- 1 userSol2 userSol2    0  7월  9 17:57 userSol1F


	# userSol1이 생성한 파일을 userSol2가 이름 수정

[userSol2@Server-A ~]$ mv /homesol/sol_tmp1/userSol1F  /homesol/sol_tmp1/usF

[userSol2@Server-A ~]$ ls  -l  /homesol/sol_tmp1/
합계 0
-rw-r--r-- 1 userSol1 solGroup 0  7월  9 17:44 usF
drwxr-sr-x 2 userSol1 solGroup 6  7월  9 17:43 userSol1D

	# userSol1이 생성한 파일을 userSol2가 삭제

[userSol2@Server-A ~]$ rm -rf  /homesol/sol_tmp1/usF

[userSol2@Server-A ~]$ ls  -l  /homesol/sol_tmp1/
합계 0
drwxr-sr-x 2 userSol1 solGroup 6  7월  9 17:43 userSol1D
```

- 이름 변경 및 파일, 디렉터리 삭제가 불가는한 기능은 Sticky-bit이다. (Set-GID는 소유 그룹만 통일하며, 삭제/이름변경 제한은 Sticky-bit의 역할이다.)

**정리**: **Set-GID**를 디렉터리에 설정하면(`chmod 2xxx`) 그 안에서 생성되는 모든 파일/디렉터리의 소유 그룹이 사용자와 무관하게 상위 디렉터리 그룹으로 통일되어, 팀 공유 폴더에서 소유 그룹 일관성을 유지하는 데 사용된다.

# Linux 10 — 원격 접속: SSH · SCP · vsFTP · SFTP

## SSH / Telnet / 프로세스 / 데몬 (10-1)

#### SSH / Telnet / 프로세스 / 데몬 정리

원격 서버에 안전하게 접속하고 파일을 주고받기 위해 SSH, SCP, vsFTP, SFTP 같은 원격 접속·전송 기술이 서버 관리와 배포 자동화 현장에서 두루 사용된다.

- Telnet
  - TCP 23번 포트를 사용하는 원격 접속 프로토콜이다.
  - GUI 기능은 없으며 텍스트 기반 원격 터미널 접속만 가능하다.
  - 평문(암호화 없음) 으로 데이터를 송수신하기 때문에 패킷을 스니핑하면 계정, 비밀번호, 명령어가 그대로 노출된다.
  - 보안이 취약하기 때문에 보안 환경에서는 절대 사용 금지한다.

- SSH (Secure Shell)
  - TCP 22번 포트를 사용하는 보안 원격 접속 프로토콜이다.
  - Telnet과 기능은 거의 동일하지만,
  - SSH는 다음 기능을 추가 제공한다.
   * 암호화된 통신
   * 사용자 인증 강화
   * 키 기반 인증(Public/Private Key)
  - 동일한 텍스트 기반 원격 터미널 접속이지만, 보안성이 매우 높기 때문에 현재 모든 리눅스/서버 관리에서 표준으로 사용된다.

#### Program vs Process

- Program (프로그램)
  - 파일 시스템(하드디스크)에 저장되어 있는 실행 가능한 파일이다.
  - 예: /usr/bin/ls, /usr/bin/nginx, /usr/bin/python

- Process (프로세스)
  - 프로그램을 실행하면, 그 프로그램의 명령어가 메모리(RAM)에 로딩되고 CPU를 사용하게 된다.
   이렇게 실행 중인 프로그램의 상태를 프로세스라고 한다.

  - 하나의 프로그램으로 여러 개의 프로세스가 생성될 수도 있다. (예: Apache, Nginx, Chrome 등)

#### Foreground Process vs Background Process

- Foreground Process
  - 사용자가 화면에서 직접 확인하면서 실행되는 프로세스
  - 예 : 터미널에서 실행 중인 vi

- Background Process
  - 실행 중이지만 사용자 화면에 나타나지 않는 프로세스
  - 시스템이 필요에 따라 계속 동작한다.
  - 예 : 웹 서버(Nginx, Apache) , 데이터베이스(MySQL, MariaDB)
  - Cron 서비스, SSH 데몬 등

#### Daemon (데몬)

- 백그라운드에서 지속적으로 실행되며, 특정 요청을 처리하기 위해 대기하는 프로세스이다.
- 특정 조건이 발생하면 바로 동작하도록 메모리에 상주하며 기다리는 역할을 한다.
- 일반 프로세스와의 차이
  - 일반 프로세스는
  - 실행  --> 작업 수행  --> 종료순으로 동작
  - 데몬 프로세스는 계속 살아 있으며 필요한 순간에 다시 동작한다.

- 특징
  - 서버 역할을 하는 대부분의 서비스가 데몬 형태로 동작한다.
  - 일반적으로 프로그램 이름 뒤에 d가 붙는다.
  - sshd 	: SSH 서버 데몬
  - httpd 	: Apache 웹서버 데몬
  - crond	: 스케줄러 데몬

- 설정 변경 시 재시작 필요한 이유

  - 데몬은 메모리에 상주하며 설정 파일을 캐시처럼 사용한다.
   따라서 설정 파일을 수정해도 이미 실행 중인 데몬이 새 설정을 자동으로 읽지 않는다.

- 데몬 재시작 (systemctl restart sshd 등)

---

```bash
EX1) 현재 시스템으로 root 계정으로 접속이 가능해야 한다.

[root@Server-A ~]# vi  /etc/ssh/sshd_config
~~~~~~~~~~ 중간 생략 ~~~~~~~~~~
     39 #LoginGraceTime 2m
     40 PermitRootLogin yes		# PermitRootLogin yes = root로  ssh 접속 허용
     41 #StrictModes yes
     42 #MaxAuthTries 6
     43 #MaxSessions 10
~~~~~~~~~~ 중간 생략 ~~~~~~~~~~

-Linux에서 Deamon은 설정을 바로 인식하지 못하기때문에 System을 재부팅하거나 deamon을 재시작해야한다.
```

```bash
[root@Server-A ~]# systemctl  restart  sshd
```

```bash
[root@Server-A ~]# systemctl  start  sshd	# Package 설치후 실행
[root@Server-A ~]# systemctl  restart  sshd	# Deamon 재실행 (설정 변경시 시스템을 재부팅 하거나 데몬을 재시작 해야한다.)
[root@Server-A ~]# systemctl  enable  sshd	# 시스템이 재부팅되었을때 sshd를 자동 실행
[root@Server-A ~]# systemctl  status  sshd	# Process동작 상태 확인
```

---

**EX2)** 현재 시스템은 root 계정으로 접속할 수 없어야 한다.

```bash
[root@Server-A ~]# vi /etc/ssh/sshd_config
~~~~~~~~ 중간 생략 ~~~~~~~~ 
     39 #LoginGraceTime 2m
     40 PermitRootLogin no	# no 로 변경
     41 #StrictModes yes
     42 #MaxAuthTries 6
     43 #MaxSessions 10
~~~~~~~~ 중간 생략 ~~~~~~~~ 
```

```bash
[root@Server-A ~]# systemctl  restart  sshd	# Deamon 재실행 (설정 변경시 시스템을 재부팅 하거나 데몬을 재시작 해야한다.)
```

```bash
login as: root
root@192.168.10.100's password:
Access denied
root@192.168.10.100's password:
Access denied
```

---

```bash
EX3) SSH 접속시 Banner기능을 사용하여 해당 시스템 접속시 주의사항 또는 경고문이 출력되도록 설정해야 한다.

[root@Server-A ~]# vi  /etc/ssh/ssh-banner
#######################################################################
해당 시스템에 무단으로 접속시 법적 제재를 받을수 있습니다.
admin	: ryu changwan
phone   	: 010-5555-1234
mail    	: admin@soldesk.com
fax     	: 02-555-1234
#######################################################################

:wq
```

```bash
[root@Server-A ~]# vi  /etc/ssh/sshd_config
~~~~~~~~~~~~~ 중간 생략 ~~~~~~~~~~~~~
    112 #UseDNS no
    113 #PidFile /var/run/sshd.pid
    114 #MaxStartups 10:30:100
    115 #PermitTunnel no
    116 #ChrootDirectory none
    117 #VersionAddendum none
    118
    119 # no default banner path
    120 #Banner none
    121 Banner /etc/ssh/ssh-banner	# Banner의 경로 및 파일명 설정
    122
    123 # override default of no subsystems
    124 Subsystem       sftp    /usr/libexec/openssh/sftp-server
    125
    126 # Example of overriding settings on a per-user basis
    127 #Match User anoncvs
    128 #       X11Forwarding no
    129 #       AllowTcpForwarding no
    130 #       PermitTTY no
```

```bash
Using username "guest".		# 배너가 적용되지 않는다.
guest@192.168.10.100's password:
Last login: Wed Dec  3 09:43:37 2025 from 192.168.10.1
[guest@Server-A ~]$
```

```bash
[root@Server-A ~]# systemctl  restart  sshd	# Deamon 재실행 (설정 변경시 시스템을 재부팅 하거나 데몬을 재시작 해야한다.)
```

```bash
login as: guest
Pre-authentication banner message from server:
| #######################################################################
| 해당 시스템에 무단으로 접속시 법적 제재를 받을수 있습니다.
| admin: ryu changwan
| phone: 010-5555-1234
| mail: admin@soldesk.com
| fax: 02-555-1234
| #######################################################################
End of banner message from server
guest@192.168.10.100's password:
Activate the web console with: systemctl enable --now cockpit.socket

Last login: Thu Jul  9 14:40:36 2026 from 192.168.10.1
```

---

**EX4)** SSH 접속시 TCP 22번이 아닌 TCP 2002번을 사용하여 접속되어야 한다.

```bash
[root@Server-A ~]# vi  /etc/ssh/sshd_config
~~~~~~~~~~~~~ 중간 생략 ~~~~~~~~~~~~~
     21 #Port 22
     22 Port 2002		# TCP 2002번 설정
     23 #AddressFamily any
     24 #ListenAddress 0.0.0.0
     25 #ListenAddress :
~~~~~~~~~~~~~ 중간 생략 ~~~~~~~~~~~~~
```

- putty를 사용해서 TCP 2002번으로 접속 X

```bash
login as: guest	#  TCP 22번으로 접속 O
Pre-authentication banner message from server:
| #######################################################################
| 해당 시스템에 무단으로 접속시 법적 제재를 받을수 있습니다.
| admin: ryu changwan
| phone: 010-5555-1234
| mail: admin@soldesk.com
| fax: 02-555-1234
| #######################################################################
End of banner message from server
guest@192.168.10.100's password:
Activate the web console with: systemctl enable --now cockpit.socket

Last login: Thu Jul 16 11:03:54 2026 from 192.168.10.1
[guest@Server-A ~]$
```

```bash
[root@Server-A ~]# systemctl  restart sshd	# Deamon 재실행 (설정 변경시 시스템을 재부팅 하거나 데몬을 재시작 해야한다.)
```

- putty를 사용해서 TCP 22번으로 접속 X
- putty를 사용해서 TCP 2002번으로 접속 X

- httpd 데몬을 재 시작해도 TCP 2002번으로 접속되지 않는다.

- TCP 2002번으로 Server-A에 접속하지만 방화벽에의해 차단된

```bash
[root@Server-A ~]# firewall-cmd  --permanent  --add-port=2002/tcp
success
```

```bash
[root@Server-A ~]# firewall-cmd  --reload
success
```

```bash
[root@Server-A ~]# firewall-cmd  --list-port
111/tcp 139/tcp 445/tcp 2002/tcp 2049/tcp 111/udp 137/udp 138/udp
```

- putty로 접속
  - IP	: 192.168.10.100
  - Port	: TCP 2002

```bash
login as: guest
Pre-authentication banner message from server:
| #######################################################################
| 해당 시스템에 무단으로 접속시 법적 제재를 받을수 있습니다.
| admin: ryu changwan
| phone: 010-5555-1234
| mail: admin@soldesk.com
| fax: 02-555-1234
| #######################################################################
End of banner message from server
guest@192.168.10.100's password:
Activate the web console with: systemctl enable --now cockpit.socket

Last login: Thu Jul 16 11:07:48 2026 from 192.168.10.1
[guest@Server-A ~]$
```

---

**EX5-1)** SSH 기본 기능으로 비밀번호 3회 실패시 세션을 종료 해야 한다.

```bash
[root@Server-A ~]# vi  /etc/ssh/sshd_config
~~~~~~~~~~~~~ 중간 생략 ~~~~~~~~~~~~~
     40 #LoginGraceTime 2m
     41 PermitRootLogin yes
     42 #StrictModes yes
     43 #MaxAuthTries 6
     44 MaxAuthTries 3	# 주석 제거후 3으로 변경
     45 #MaxSessions 10
~~~~~~~~~~~~~ 중간 생략 ~~~~~~~~~~~~~
 : wq
```

```bash
[root@Server-A ~]# systemctl  restart sshd
```

```bash
login as: guest
Pre-authentication banner message from server:
| #######################################################################
| 해당 시스템에 무단으로 접속시 법적 제재를 받을수 있습니다.
| admin: ryu changwan
| phone: 010-5555-1234
| mail: admin@soldesk.com
| fax: 02-555-1234
| #######################################################################
End of banner message from server
guest@192.168.10.100's password:
Access denied
guest@192.168.10.100's password:
Access denied
guest@192.168.10.100's password:
```

**EX5-2)** SSH 기본 기능으로 접속 후 30초안에 인증을 성공하지 못하면 세션이 종료되어야 한다.

```bash
[root@Server-A ~]# vi  /etc/ssh/sshd_config
~~~~~~~~~~~~~ 중간 생략 ~~~~~~~~~~~~~
     40 #LoginGraceTime 2m
     41 LoginGraceTime 30		# 30s , 30 = 30초
     42 PermitRootLogin yes
     43 #StrictModes yes
     44 #MaxAuthTries 6
~~~~~~~~~~~~~ 중간 생략 ~~~~~~~~~~~~~
```

```bash
[root@Server-A ~]# systemctl  restart sshd
```

```bash
-30초간 로그인하지 않고 대기하게되면 SSH Session이 종료된다.

login as: guest
Pre-authentication banner message from server:
| #######################################################################
| 해당 시스템에 무단으로 접속시 법적 제재를 받을수 있습니다.
| admin: ryu changwan
| phone: 010-5555-1234
| mail: admin@soldesk.com
| fax: 02-555-1234
| #######################################################################
End of banner message from server
guest@192.168.10.100's password:
```

**EX5-3)** SSH 기본 기능으로 'guest1', 'guest2' 계정만 SSH로 접속이 가능해야 한다.

```bash
[root@Server-A ~]# vi  /etc/ssh/sshd_config
~~~~~~~~~~~~~ 중간 생략 ~~~~~~~~~~~~~
    128 # Example of overriding settings on a per-user basis
    129 #Match User anoncvs
    130 #       X11Forwarding no
    131 #       AllowTcpForwarding no
    132 #       PermitTTY no
    133 #       ForceCommand cvs server
    134 AllowUsers  guest1 guest2
```

```bash
[root@Server-A ~]# cat /etc/passwd | grep guest
guest:x:1000:1000:guest:/home/guest:/bin/bash
guest1:x:1009:1009:yaja:/home/guest1:/bin/bash
guest2:x:1010:1010::/solhome/guest2:/bin/tcsh
```

```bash
[root@Server-A ~]# systemctl  restart sshd
```

```bash
[root@Server-A ~]# passwd  guest1
guest1 사용자의 비밀 번호 변경 중
새 암호:
잘못된 암호: 암호는 8 개의 문자 보다 짧습니다
새 암호 재입력:
passwd: 모든 인증 토큰이 성공적으로 업데이트 되었습니다.
```

```bash
[root@Server-A ~]# passwd  guest2
guest1 사용자의 비밀 번호 변경 중
새 암호:
잘못된 암호: 암호는 8 개의 문자 보다 짧습니다
새 암호 재입력:
passwd: 모든 인증 토큰이 성공적으로 업데이트 되었습니다.
```

```bash
login as: guest1	# guest1 계정 로그인 O
Pre-authentication banner message from server:
| #######################################################################
| 해당 시스템에 무단으로 접속시 법적 제재를 받을수 있습니다.
| admin: ryu changwan
| phone: 010-5555-1234
| mail: admin@soldesk.com
| fax: 02-555-1234
| #######################################################################
End of banner message from server
guest1@192.168.10.100's password:
[guest1@Server-A ~]$
```

```bash
login as: guest	# guest 계정은 로그인 X
Pre-authentication banner message from server:
| #######################################################################
| 해당 시스템에 무단으로 접속시 법적 제재를 받을수 있습니다.
| admin: ryu changwan
| phone: 010-5555-1234
| mail: admin@soldesk.com
| fax: 02-555-1234
| #######################################################################
End of banner message from server
guest@192.168.10.100's password:
Access denied
guest@192.168.10.100's password:
```

#### 정보 확인 후 삭제

```bash
[root@Server-A ~]# vi  /etc/ssh/sshd_config
~~~~~~~~~~~~~ 중간 생략 ~~~~~~~~~~~~~
#Match User anoncvs
#       X11Forwarding no
#       AllowTcpForwarding no
#       PermitTTY no
#       ForceCommand cvs server
#AllowUsers  guest1 guest2		# 삭제 또는 주석 처리
```

**EX5-4)** SSH 기본 기능으로 192.168.112.0/24 네트워크 대역만 SSH로 접속이 가능해야 한다.

```bash
[root@Server-A ~]# vi  /etc/ssh/sshd_config
~~~~~~~~~~~~~ 중간 생략 ~~~~~~~~~~~~~
#Match User anoncvs
#       X11Forwarding no
#       AllowTcpForwarding no
#       PermitTTY no
#       ForceCommand cvs server
#AllowUsers  guest1 guest2
AllowUsers  @192.168.112.		# 192.168.112.0/24 네트워크 대역만 허용

:wq
```

```bash
[root@Server-A ~]# systemctl  restart sshd
```

```bash
	# 현재 네트워크 대역이 192.168.10.0/24이므로 접속할 수 없다.

login as: guest
Pre-authentication banner message from server:
| #######################################################################
| 해당 시스템에 무단으로 접속시 법적 제재를 받을수 있습니다.
| admin: ryu changwan
| phone: 010-5555-1234
| mail: admin@soldesk.com
| fax: 02-555-1234
| #######################################################################
End of banner message from server
guest@192.168.10.100's password:
Access denied
guest@192.168.10.100's password:
```

```bash
	# 정보 확인 후 삭제

[root@Server-A ~]# vi  /etc/ssh/sshd_config
~~~~~~~~~~~~~ 중간 생략 ~~~~~~~~~~~~~
#Match User anoncvs
#       X11Forwarding no
#       AllowTcpForwarding no
#       PermitTTY no
#       ForceCommand cvs server
#AllowUsers  guest1 guest2
#AllowUsers  @192.168.112.		# 삭제 또는 주석 처리

:wq
```

**EX5-5)** SSH 기본 기능으로 'sshGroup' 그룹에 포함된 계정만 SSH로 접속이 가능해야 한다.

```bash
[root@Server-A ~]# vi  /etc/ssh/sshd_config
~~~~~~~~~~~~~ 중간 생략 ~~~~~~~~~~~~~
    128 # Example of overriding settings on a per-user basis
    129 #Match User anoncvs
    130 #       X11Forwarding no
    131 #       AllowTcpForwarding no
    132 #       PermitTTY no
    133 #       ForceCommand cvs server
    134 #AllowUsers  guest1 guest2
    135 #AllowUsers  @192.168.112.
    136 AllowGroups  sshGroup
:wq
```

```bash
[root@Server-A ~]# systemctl  restart sshd
```

- 'sshGroup' 그룹 생성
- 'sshUser1' 계정 생성
- 'sshUser1' 계정을 'sshGroup' 그룹에 추가
- 'sshUser1' 계정은 Server-A로 접속 확인
- 'guest' 계정은 Server-A로 접속 불가 확인

```bash
[root@Server-A ~]# groupadd sshGroup			# sshGroup 그룹 생성

[root@Server-A ~]# useradd  sshUser1			# sshUser1계정 생성

[root@Server-A ~]# usermod  -aG  sshGroup  sshUser1	# sshUser1 계정을 sshGroup 그룹에 포함1

[root@Server-A ~]# id sshUser1
uid=1326(sshUser1) gid=1326(sshUser1) groups=1326(sshUser1),1338(sshGroup)
```

```bash
[root@Server-A ~]# passwd  sshUser1
sshUser1 사용자의 비밀 번호 변경 중
새 암호:
잘못된 암호: 암호는 8 개의 문자 보다 짧습니다
새 암호 재입력:
passwd: 모든 인증 토큰이 성공적으로 업데이트 되었습니다.
```

```bash
	# guest 계정 SSH 접속 X
login as: guest
Pre-authentication banner message from server:
| #######################################################################
| 해당 시스템에 무단으로 접속시 법적 제재를 받을수 있습니다.
| admin: ryu changwan
| phone: 010-5555-1234
| mail: admin@soldesk.com
| fax: 02-555-1234
| #######################################################################
End of banner message from server
guest@192.168.10.100's password:
Access denied
guest@192.168.10.100's password:
```

```bash
	# sshUser1 계정 SSH 접속 O

login as: sshUser1
Pre-authentication banner message from server:
| #######################################################################
| 해당 시스템에 무단으로 접속시 법적 제재를 받을수 있습니다.
| admin: ryu changwan
| phone: 010-5555-1234
| mail: admin@soldesk.com
| fax: 02-555-1234
| #######################################################################
End of banner message from server
sshUser1@192.168.10.100's password:
[sshUser1@Server-A ~]$
```

```bash
	# 정보 확인 후 삭제

[root@Server-A ~]# vi  /etc/ssh/sshd_config
~~~~~~~~~~~~~ 중간 생략 ~~~~~~~~~~~~~
    129 #Match User anoncvs
    130 #       X11Forwarding no
    131 #       AllowTcpForwarding no
    132 #       PermitTTY no
    133 #       ForceCommand cvs server
    134 #AllowUsers  guest1 guest2
    135 #AllowUsers  @192.168.112.
    136 #AllowGroups  sshGroup		# 삭제 또는 주석 처리
```

---

**정리**: SSH / Telnet / 프로세스 / 데몬 (10-1) 섹션에서 다룬 명령어와 실습 예제를 통해 관련 기능의 구성 방법과 동작 원리를 확인했다.

## SCP (Secure Copy) (10-2)

### SCP  (Secure Copy)

- SCP는 Secure Copy의 약자로, SSH 프로토콜을 기반으로 원격 서버와 파일을 송/수신하는 보안 전송 방식이다.
- SSH에서 사용하는 TCP 22번 포트를 그대로 사용하며, 모든 데이터가 암호화(Encrypted) 되어 전송된다.

- FTP처럼 별도의 데몬을 설치할 필요도 없고, 암호화되지 않아 보안 취약한 FTP와 달리
  SSH만 켜져 있으면 바로 사용할 수 있는 안전한 파일 전송 방식이다.

- SCP의 특징
  - SSH 기반 동작하기때문에 별도의 서비스 데몬이 필요 없다.
  - SSH 포트(TCP 22)만 열려 있으면 사용 가능
  - 로그인 인증 방식도 SSH와 동일(계정/비밀번호, 또는 공개키 인증)
  - 파일 내용뿐 아니라 사용자 인증 정보, 명령어까지 모두 암호화되어 안전하다.
  - 서버 간 파일 복사도 가능
  - 단순히 파일/디렉터리를 복사하는 목적에 최적화
  - 주로 간단한 백업 또는 단일 파일 전송에 많이 사용

---

```bash
[root@Server-A ~]# rpm  -qa  | grep openssh
openssh-9.9p1-7.el9_8.rocky.0.1.x86_64
openssh-clients-9.9p1-7.el9_8.rocky.0.1.x86_64
openssh-server-9.9p1-7.el9_8.rocky.0.1.x86_64
```

```bash
[root@Server-A ~]# firewall-cmd  --permanent  --add-port=22/tcp
success

root@Server-A ~]# firewall-cmd  --permanent  --add-service=ssh
success
```

```bash
[root@Server-A ~]# firewall-cmd  --reload
success
```

```bash
[root@Server-A ~]# firewall-cmd  --list-port
22/tcp 111/tcp 139/tcp 445/tcp 2002/tcp 2049/tcp 111/udp 137/udp 138/udp

[root@Server-A ~]# firewall-cmd  --list-services
cockpit dhcpv6-client nfs rpc-bind samba ssh
```

---

```bash
			# SSH 접속

	# 1) root 계정으로 접속

[root@Client-L ~]# ssh  A.B.C.D
```

```bash
[root@Client-L ~]# ssh  192.168.10.100
The authenticity of host '192.168.10.100 (192.168.10.100)' can't be established.
ED25519 key fingerprint is SHA256:hSo7vPq/6MKH+FikOCDeY8V3GgrVvdPE8e+NHDAJV6g.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '192.168.10.100' (ED25519) to the list of known hosts.
root@192.168.10.100's password:
Activate the web console with: systemctl enable --now cockpit.socket

Last login: Thu Jul 16 10:49:12 2026 from 192.168.10.1
[root@Server-A ~]#
[root@Server-A ~]#	# Server-A로 root 권한으로 SSH 접속
```

#### 2-1) 특정 계정으로 접속

```bash
[root@Client-L ~]# ssh  -l  [계정명]  A.B.C.D
```

```bash
[root@Client-L ~]# ssh  -l  guest  192.168.10.100
guest@192.168.10.100's password:
Activate the web console with: systemctl enable --now cockpit.socket

Last failed login: Thu Jul 16 12:00:02 KST 2026 from 192.168.10.1 on ssh:notty
There were 4 failed login attempts since the last successful login.
Last login: Thu Jul 16 11:47:56 2026 from 192.168.10.1
[guest@Server-A ~]$
[guest@Server-A ~]$		# Server-A로 guest 계정을 사용하여 SSH 접속
```

```bash
	# 2-2) 특정 계정으로 접속

[root@Client-L ~]# ssh  [계정명]@A.B.C.D
```

```bash
[root@Client-L ~]# ssh  guest@192.168.10.100
guest@192.168.10.100's password:
Activate the web console with: systemctl enable --now cockpit.socket

Last login: Thu Jul 16 12:15:36 2026 from 192.168.10.130
```

---

```bash
                       ### SCP를 사용한 Download 실습

EX1) 아래의 조건에 맞게 Server-A , Client-L을 설정하시오
 # Server-A에서 '/temp' 디렉터리안의 모든 파일 및 디렉터리를 삭제해야한다.
 # Server-A에서 '/etc'디렉터리안의 파일중 'a'로 시작하는 모든 파일을  '/temp' 디렉터리로 복사해야한다.
 # Server-A에서 '/etc'디렉터리안의 파일중 'b'로 시작하는 모든 파일을  '/temp' 디렉터리로 복사해야한다.
 # Server-A에서 '/etc'디렉터리안의 파일중 'd'로 시작하는 모든 파일을  '/temp' 디렉터리로 복사해야한다.
 # Server-A의  '/temp '디렉터리안에 'sol-a1 ~ sol-a3'
 # Server-A의  '/temp' 디렉터리안에 'sol-b4 ~ sol-b6'
 # Server-A의  '/temp' 디렉터리안에 'sol-c7 ~ sol-c9'
 # Client-L에서 '/client' 디렉터리를 생성하시오

[root@Server-A ~]# rm -rf /temp/*

[root@Server-A ~]# cp -r /etc/a*  /temp/

[root@Server-A ~]# cp -r /etc/b*  /temp/

[root@Server-A ~]# cp -r /etc/d*  /temp/

[root@Server-A ~]# touch  /temp/sol-a1  /temp/sol-a2  /temp/sol-a3
[root@Server-A ~]# touch  /temp/sol-b4  /temp/sol-b5  /temp/sol-b6
[root@Server-A ~]# touch  /temp/sol-c7  /temp/sol-c8  /temp/sol-c9
```

```bash
[root@Client-L ~]# mkdir /client
```

```bash
[root@Server-A ~]# ls  -l  /temp
```

---

```bash
[root@Client-L ~]# scp  [계정명]@A.B.C.D:[Source경로/파일명]   [Destination경로]	<---- guest계정을 사용하여 Server의 파일을 받아오는 기능
									         (download , Server-A guest계정의 Password를 알고 있어야 한다.)

[root@Client-L ~]# scp  [Source경로/파일명]  [계정명]@A.B.C.D:[Destination/경로]	<---- guest계정을 사용하여 Server의 파일을 전송하는 기능
									         (upload , Server-A  guest계정의 Password를 알고 있어야 한다.)
```

---

**EX2)** Client-L에서 Server-A의 'guest' 계정을 사용하여 파일을 Download 해야한다.
  - Server-A의  '/temp' 디렉터리안의 'aliases'파일을 Client-L의 '/client' 디렉터리로 복사해야한다.
  - Server-A의  '/temp' 디렉터리안의 'bashrc'파일을 Client-L의 '/client' 디렉터리로 복사해야한다.
  - Server-A의  '/temp' 디렉터리안의 'dnsmasq.conf'파일을 Client-L의 '/client' 디렉터리로 복사해야한다.
  - Server-A의  '/temp' 디렉터리안의 'sol-a1' , sol-a2' , sol-a3' 파일을 Client-L의 '/client'디렉터리로 복사해야한다.
  - Server-A의  '/temp' 디렉터리안의 'dnf' 디렉터리를 Client-L의 '/client'디렉터리로 복사해야한다.

```bash
[root@Client-L ~]# scp  guest@192.168.10.100:/temp/aliases  /client
guest@192.168.10.100's password:1234
aliases                                       100% 1529     1.0MB/s   00:00
```

```bash
[root@Client-L ~]# ls  -l  /client
합계 4
-rw-r--r-- 1 root root 1529  7월 16 12:47 aliases
```

```bash
[root@Client-L ~]# scp  guest@192.168.10.100:/temp/bashrc  /client
guest@192.168.10.100's password:1234
bashrc                                        100% 2658     1.3MB/s   00:00
```

```bash
[root@Client-L ~]# ls  -l  /client
합계 8
-rw-r--r-- 1 root root 1529  7월 16 12:47 aliases
-rw-r--r-- 1 root root 2658  7월 16 12:49 bashrc
```

```bash
[root@Client-L ~]# scp  guest@192.168.10.100:/temp/dnsmasq.conf  /client
guest@192.168.10.100's password:
dnsmasq.conf                                  100%   27KB  13.7MB/s   00:00
```

```bash
[root@Client-L ~]# ls  -l  /client
합계 36
-rw-r--r-- 1 root root  1529  7월 16 12:47 aliases
-rw-r--r-- 1 root root  2658  7월 16 12:49 bashrc
-rw-r--r-- 1 root root 27839  7월 16 12:51 dnsmasq.conf
```

```bash
[root@Client-L ~]# scp  guest@192.168.10.100:/temp/sol-a*  /client
guest@192.168.10.100's password:
```

```bash
[root@Client-L ~]# ls  -l  /client
합계 36
-rw-r--r-- 1 root root  1529  7월 16 12:47 aliases
-rw-r--r-- 1 root root  2658  7월 16 12:49 bashrc
-rw-r--r-- 1 root root 27839  7월 16 12:51 dnsmasq.conf
-rw-r--r-- 1 root root     0  7월 16 12:52 sol-a1
-rw-r--r-- 1 root root     0  7월 16 12:52 sol-a2
-rw-r--r-- 1 root root     0  7월 16 12:52 sol-a3
```

```bash
[root@Client-L ~]# scp  guest@192.168.10.100:/temp/dnf  /client		# scp는 기본적으로 파일만 복사한다.
guest@192.168.10.100's password:
scp: download /temp/dnf/: not a regular file
```

```bash
[root@Client-L ~]# scp  -r  guest@192.168.10.100:/temp/dnf  /client
guest@192.168.10.100's password:
copr.conf                                	100%  351   312.0KB/s   00:00
debuginfo-install.conf                        	100%   30    45.6KB/s   00:00
kpatch.conf                                   	100%   41    54.9KB/s   00:00
dnf.conf                                      	100%    4     5.4KB/s   00:00
setup.conf                                    	100%    6     6.4KB/s   00:00
systemd.conf                                  	100%   21    21.1KB/s   00:00
grub2-tools-minimal.conf                	100%   20    33.7KB/s   00:00
sudo.conf                                     	100%    5     8.1KB/s   00:00
grub2-pc.conf                                 	100%    9    10.5KB/s   00:00
yum.conf                                      	100%    4     6.1KB/s   00:00
contentdir                                    	100%   10    15.9KB/s   00:00
rltype                                        	100%    1     1.3KB/s   00:00
sigcontentdir                                 	100%    8    11.0KB/s   00:00
stream                                        	100%    9    13.1KB/s   00:00
dnf.conf                                      	100%  108   163.7KB/s   00:00
```

---

**EX3)** 아래의 조건에 맞게 설정하시오
  - Client-L에서 'clientb' 계정을 생성하시오 (password = 개별 설정)
  - Client-L에서 'clientb' 계정을 사용하여 접속 후 아래의 조건에 맞게 파일을 다운로드해야한다.
  - Server-A의 '/temp/sol-b4 ~ sol-b6' 파일을 Client-L의 '/client' 디렉터리로 파일을 다운로드해야한다.

```bash
[root@Client-L ~]# useradd  clientb
```

```bash
[root@Client-L ~]# passwd  clientb
clientb 사용자의 비밀 번호 변경 중
새 암호:
잘못된 암호: 암호는 8 개의 문자 보다 짧습니다
새 암호 재입력:
passwd: 모든 인증 토큰이 성공적으로 업데이트 되었습니다.
```

```bash
[clientb@Client-L ~]$ scp guest@192.168.10.100:/temp/sol-b*  /client
The authenticity of host '192.168.10.100 (192.168.10.100)' can't be established.
ED25519 key fingerprint is SHA256:hSo7vPq/6MKH+FikOCDeY8V3GgrVvdPE8e+NHDAJV6g.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '192.168.10.100' (ED25519) to the list of known hosts.
guest@192.168.10.100's password:
scp: open local "/client/sol-b4": Permission denied
scp: open local "/client/sol-b5": Permission denied
scp: open local "/client/sol-b6": Permission denied
```

```bash
[clientb@Client-L ~]$ ls  -l  / | grep client
drwxr-xr-x    3 root root  108  7월 16 12:54 client		# rwx  r-x  r-x : clientb는 other의 권한이 적용되므로 해당 디렉터리의 내용을 변경할 수 없다.
```

```bash
	# clientb
[clientb@Client-L ~]$ chmod  777  /client
chmod: changing permissions of '/client': 명령을 허용하지 않음
```

```bash
	# root
[root@Client-L ~]# chmod  777  /client
```

```bash
	# root
[root@Client-L ~]# ls  -l  / | grep client
drwxrwxrwx    3 root root  108  7월 16 12:54 client
```

```bash
	# clientb
[clientb@Client-L ~]$ scp guest@192.168.10.100:/temp/sol-b*  /client
guest@192.168.10.100's password:
```

```bash
[clientb@Client-L ~]$ ls -ld  /client/sol-b*
-rw-r--r-- 1 clientb clientb 0  7월 16 13:06 /client/sol-b4
-rw-r--r-- 1 clientb clientb 0  7월 16 13:06 /client/sol-b5
-rw-r--r-- 1 clientb clientb 0  7월 16 13:06 /client/sol-b6
```

---

```bash
EX4-1) 아래의 조건에 맞게 설정하시오
 # Server-A에  '/SHARE' 디렉터리를 생성하시오
 # Server-A에서 '/etc'디렉터리안의 파일중 'r'로 시작하는 모든 파일을  '/SHARE' 디렉터리로 복사해야한다.
 # Server-A에서 '/etc'디렉터리안의 파일중 's'로 시작하는 모든 파일을  '/SHARE' 디렉터리로 복사해야한다.

[root@Server-A ~]# rm  -rf  /SHARE

[root@Server-A ~]# mkdir /SHARE

[root@Server-A ~]# cp  -r  /etc/r*  /SHARE/
[root@Server-A ~]# cp  -r  /etc/s*  /SHARE/

[root@Server-A ~]# ls -l /
```

**EX4-2)** 아래의 조건에 맞게 설정하시오
  - Client-L에서 'clientb' 계정을 사용하여 접속 후 아래의 조건에 맞게 파일을 다운로드해야한다.
  - Server-A의 '/SHARE/rpc' 파일을 Client-L의 '/client' 디렉터리로 파일을 다운로드해야한다.
  - Server-A의 '/SHARE/rsyncd.conf' 파일을 Client-L의 '/client' 디렉터리로 파일을 다운로드해야한다.
  - Server-A의 '/SHARE/subgid' 파일을 Client-L의 '/client' 디렉터리로 파일을 다운로드해야한다.
  - Server-A의 '/SHARE/subuid' 파일을 Client-L의 '/client' 디렉터리로 파일을 다운로드해야한다.

```bash
[clientb@client-l ~]$ scp  guest@192.168.10.100:/SHARE/rpc  /client
guest@192.168.10.100's password:
rpc                                         100%  986   255.8KB/s   00:00
```

```bash
[clientb@client-l ~]$ scp  guest@192.168.10.100:/SHARE/rsyncd.conf  /client
guest@192.168.10.100's password:
rsyncd.conf                             100%  986   255.8KB/s   00:00
```

```bash
[clientb@client-l ~]$ scp  guest@192.168.10.100:/SHARE/subgid  /client
guest@192.168.10.100's password:
subgid                                         100%  986   255.8KB/s   00:00
```

```bash
[clientb@client-l ~]$ scp  guest@192.168.10.100:/SHARE/subuid  /client
guest@192.168.10.100's password:
subuid                                         100%  986   255.8KB/s   00:00
```

```bash
[clientb@Client-L ~]$ ls  -l  /client/
합계 52
-rw-r--r-- 1 root    root     1529  7월 16 12:47 aliases
-rw-r--r-- 1 root    root     2658  7월 16 12:49 bashrc
drwxr-xr-x 9 root    root      163  7월 16 12:54 dnf
-rw-r--r-- 1 root    root    27839  7월 16 12:51 dnsmasq.conf
-rw-r--r-- 1 clientb clientb  1634  7월 16 14:43 rpc
-rw-r--r-- 1 clientb clientb   458  7월 16 14:45 rsyncd.conf
-rw-r--r-- 1 root    root        0  7월 16 12:52 sol-a1
-rw-r--r-- 1 root    root        0  7월 16 12:52 sol-a2
-rw-r--r-- 1 root    root        0  7월 16 12:52 sol-a3
-rw-r--r-- 1 clientb clientb     0  7월 16 13:06 sol-b4
-rw-r--r-- 1 clientb clientb     0  7월 16 13:06 sol-b5
-rw-r--r-- 1 clientb clientb     0  7월 16 13:06 sol-b6
-rw-r--r-- 1 clientb clientb  1223  7월 16 14:45 subgid
-rw-r--r-- 1 clientb clientb  1223  7월 16 14:45 subuid
```

---

```bash
[root@Client-L ~]# scp  [계정명]@A.B.C.D:[Source경로/파일명]   [Destination경로]	<---- guest계정을 사용하여 Server의 파일을 받아오는 기능
									         (download , Server-A guest계정의 Password를 알고 있어야 한다.)

[root@Client-L ~]# scp  [Source경로/파일명]  [계정명]@A.B.C.D:[Destination/경로]	<---- guest계정을 사용하여 Server의 파일을 전송하는 기능
									         (upload , Server-A  guest계정의 Password를 알고 있어야 한다.)
```

---

### Upload 실습

**EX1)** 아래의 조건에 맞게 Server-A , Client-L을 설정하시오
  - Server-A에서 '/scpS' 디렉터리를 생성하시오
  - Client-L에서 '/scpC' 디렉터리를 생성하시오
  - Client-L에서 '/etc' 디렉터리안의 파일중 'e'로 시작하는 모든 파일을  '/scpC' 디렉터리로 복사해야한다.
  - Client-L에서 '/etc' 디렉터리안의 파일중 'f'로 시작하는 모든 파일을  '/scpC' 디렉터리로 복사해야한다.
  - Client-L의  '/scpC'  디렉터리안에 'CL-A1 ~ CL-A3' 파일 생성
  - Client-L의  '/scpC'  디렉터리안에 'CL-B4 ~ CL-A7' 파일 생성

```bash
[root@Server-A ~]# mkdir /scpS

[root@Client-L ~]# mkdir /scpC

[root@Client-L ~]# cp -r /etc/e*  /scpC/
[root@Client-L ~]# cp -r /etc/f*  /scpC/
```

```bash
[root@Client-L ~]# touch  /scpC/CL-A1  /scpC/CL-A2  /scpC/CL-A3  
[root@Client-L ~]# touch  /scpC/CL-B4  /scpC/CL-B5  /scpC/CL-B6 /scpC/CL-B7
```

```bash
[root@Client-L ~]# ls  -l  /scpC/
```

**EX2)** Client-L에서 Server-A의 'guest' 계정을 사용하여 파일을 Upload 해야한다.
  - Client-L의 'clientb'계정을 사용하여 '/scpC/exports' 파일을 Server-A의 '/temp' 디렉터리로 업로드 해야한다.
  - Client-L의 'clientb'계정을 사용하여 '/scpC/fstab' 파일을 Server-A의 '/temp' 디렉터리로 업로드 해야한다.
  - Client-L의 'clientb'계정을 사용하여 '/scpC/CL-A1' 파일을 Server-A의 '/temp' 디렉터리로 업로드 해야한다.
  - Client-L의 'clientb'계정을 사용하여 '/scpC/CL-A2' 파일을 Server-A의 '/temp' 디렉터리로 업로드 해야한다.
  - Client-L의 'clientb'계정을 사용하여 '/scpC/CL-A3' 파일을 Server-A의 '/temp' 디렉터리로 업로드 해야한다.

```bash
[clientb@Client-L ~]$ scp  /scpC/exports  guest@192.168.10.100:/temp
guest@192.168.10.100's password:
exports                                       100%    0     0.0KB/s   00:00
```

```bash
[clientb@Client-L ~]$ scp  /scpC/fstab  guest@192.168.10.100:/temp
guest@192.168.10.100's password:
fstab                                         100%  649   558.6KB/s   00:00
```

```bash
[clientb@Client-L ~]$ scp  /scpC/CL-A*  guest@192.168.10.100:/temp
guest@192.168.10.100's password:
CL-A1                                         100%    0     0.0KB/s   00:00
CL-A2                                         100%    0     0.0KB/s   00:00
CL-A3                                         100%    0     0.0KB/s   00:00
```

```bash
[root@Server-A ~]# ls -l  /temp/exports
-rw-r--r-- 1 guest guest 0  7월 16 14:53 /temp/exports
```

```bash
[root@Server-A ~]# ls -l  /temp/fstab
-rw-r--r-- 1 guest guest 649  7월 16 14:54 /temp/fstab
```

```bash
[root@Server-A ~]# ls -ld  /temp/CL-A*
-rw-r--r-- 1 guest guest 0  7월 16 14:54 /temp/CL-A1
-rw-r--r-- 1 guest guest 0  7월 16 14:54 /temp/CL-A2
-rw-r--r-- 1 guest guest 0  7월 16 14:54 /temp/CL-A3
```

```bash
[root@Server-A ~]# ls  -ld  /scpS/
drwxr-xr-x 2 root root 39  7월 16 14:58 /scpS/
```

```bash
[root@Server-A ~]# touch  /scpS/aaa
[root@Server-A ~]# touch  /scpS/bbb
[root@Server-A ~]# touch  /scpS/ccc
```

```bash
[root@Server-A ~]# ls  -l  /scpS/
합계 0
-rw-r--r-- 1 root root 0  7월 16 14:58 aaa
-rw-r--r-- 1 root root 0  7월 16 14:58 bbb
-rw-r--r-- 1 root root 0  7월 16 14:58 ccc
```

```bash
[clientb@Client-L ~]$ scp  guest@192.168.10.100:/scpS/aaa  /client
```

```bash
[clientb@Client-L ~]$ ls  -l  /client | grep aaa
-rw-r--r-- 1 clientb clientb     0  7월 16 14:59 aaa

-cp는 원본 디렉터리의 변경이 없기때문에 w권한이 없어도 복사가 가능하다.
```

```bash
[clientb@Client-L ~]$ scp  /client/clientAAA  guest@192.168.10.100:/scpS
guest@192.168.10.100's password:
scp: dest open "/scpS/clientAAA": Permission denied
scp: failed to upload file /client/clientAAA to /scpS
```

- cp는 원본 디렉터리의 변경이 없기때문에 w권한이 없어도 복사가 가능하다.
 즉 cp는 source는 w 권한의 영향을 받지 않지만 destination에는 새로운 파일 또는 디렉터리가 생성되기때문에  w 권한이 있어야 한다.

#### 도메인을 사용한 scp

```bash
[root@Server-A ~]# vi  /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
192.168.10.100  Server-A	# 추가 설정

:wq
```

```bash
[root@Client-L ~]# ping -c 3  Server-A
PING Server-A (192.168.10.100) 56(84) bytes of data.
64 bytes from Server-A (192.168.10.100): icmp_seq=1 ttl=64 time=0.482 ms
64 bytes from Server-A (192.168.10.100): icmp_seq=2 ttl=64 time=0.836 ms
64 bytes from Server-A (192.168.10.100): icmp_seq=3 ttl=64 time=0.322 ms

--- Server-A ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2060ms
rtt min/avg/max/mdev = 0.322/0.546/0.836/0.214 ms
```

```bash
[root@Client-L ~]# scp  guest@Server-A:/temp/brltty.conf  /client
guest@server-a's password:
brltty.conf                                   100%   28KB  11.0MB/s   00:00
```

**EX3-1)** 아래의 조건에 맞게 설정하시오
  - Server-A에  '/SHARE' 디렉터리를 생성하시오
  - Server-A에서 '/etc'디렉터리안의 파일중 'r'로 시작하는 모든 파일을  '/SHARE' 디렉터리로 복사해야한다.
  - Server-A에서 '/etc'디렉터리안의 파일중 's'로 시작하는 모든 파일을  '/SHARE' 디렉터리로 복사해야한다.

```bash
[root@Server-A ~]# mkdir   /SHARE		# /SHARE가 없을 경우

             OR

[root@Server-A ~]# rm  -rf   /SHARE/*	# /SHARE가 있을 경우
```

```bash
# /etc 디렉터리 안의 r로 시작하는 모든 파일을 /SHARE로 복사
[root@Server-A ~]# cp  -r  /etc/r*  /SHARE/

# /etc 디렉터리 안의 s로 시작하는 모든 파일을 /SHARE로 복사
[root@Server-A ~]# cp  -r /etc/s*  /SHARE/
```

**EX3-2)** 아래의 조건에 맞게 설정하시오
  - Client-L에서 'clientb' 계정을 사용하여 접속 후 아래의 조건에 맞게 파일을 다운로드해야한다.
  - Server-A의 '/SHARE/rpc' 파일을 Client-L의 '/client' 디렉터리로 파일을 다운로드해야한다.
  - Server-A의 '/SHARE/resolv.conf' 파일을 Client-L의 '/client' 디렉터리로 파일을 다운로드해야한다.
  - Server-A의 '/SHARE/subgid' 파일을 Client-L의 '/client' 디렉터리로 파일을 다운로드해야한다.
  - Server-A의 '/SHARE/subuid' 파일을 Client-L의 '/client' 디렉터리로 파일을 다운로드해야한다.

```bash
	# 방법 1
[root@Client-L ~]# scp guest@192.168.10.100:/SHARE/rpc     /client/

[root@Client-L ~]# scp guest@192.168.10.100:/SHARE/resolv.conf   /client/

[root@Client-L ~]# scp guest@192.168.10.100:/SHARE/subgid  /client/

[root@Client-L ~]# scp guest@192.168.10.100:/SHARE/subuid  /client/
```

```bash
	# 방법 2
[root@Client-L ~]# scp  guest@192.168.10.100:/SHARE/rpc  \
 guest@192.168.10.100:/SHARE/resolv.conf  \
 guest@192.168.10.100:/SHARE/subgid  \
 guest@192.168.10.100:/SHARE/subuid  \
 /client/

guest@192.168.10.100's password:1234
rpc                                           		100% 1634     1.8MB/s   00:00
guest@192.168.10.100's password:1234
resolv.conf                                   		100%   54    50.3KB/s   00:00
guest@192.168.10.100's password:1234
subgid                                        		100% 1223     1.1MB/s   00:00
guest@192.168.10.100's password:1234
subuid  
```

```bash
	# 방법 3
[root@Client-L ~]# scp  guest@192.168.10.100:/SHARE/{rpc,resolv.conf,subgid,subuid}  /client
guest@192.168.10.100's password:
rpc                                           		100% 1634     1.3MB/s   00:00
guest@192.168.10.100's password:
resolv.conf                                   		100%   54    56.0KB/s   00:00
guest@192.168.10.100's password:
subgid                                        		100% 1223     1.7MB/s   00:00
guest@192.168.10.100's password:
subuid 
```

---

```
			## Windows  --->  Linux Server
```

**EX1)** Wind-C의 파일 C:\data\a.txt 를 Rocky Linux 9 서버의 /temp 디렉터리로 업로드하시오.
  - 리눅스 계정: guest, 서버 IP: 192.168.10.100

- 리눅스에서 경로를 설정시 '/'를 사용하지만 윈도우는 '\'를 사용한다.

```bash
	# cmd 관리자 권한으로 실행

C:\Windows\system32> scp  C:\data\a.txt  guest@192.168.10.100:/temp
The authenticity of host '192.168.10.100 (192.168.10.100)' can't be established.
ED25519 key fingerprint is SHA256:hSo7vPq/6MKH+FikOCDeY8V3GgrVvdPE8e+NHDAJV6g.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '192.168.10.100' (ED25519) to the list of known hosts.
guest@192.168.10.100's password:
a.txt                                                                                        100%   58     0.1KB/s   00:00
```

```bash
[root@Server-A ~]# ls  -ld  /temp/a.txt
-rw-r--r-- 1 guest guest 58  7월 16 15:45 /temp/a.txt
```

```bash
[root@Server-A ~]# cat  /temp/a.txt
scp를 사용하여 윈도우에서 리눅스로 업로드
```

**EX1-2)** Wind-C의 파일 C:\data\b.txt 를 Rocky Linux 9 서버의 /temp 디렉터리로 업로드하시오.
  - 리눅스 계정: guest, 서버 IP: 192.168.10.100

```bash
	# Windows PowerShell

Copyright (C) Microsoft Corporation. All rights reserved.

새로운 크로스 플랫폼 PowerShell 사용 https://aka.ms/pscore6

PS C:\Users\aaa> scp  C:\data\b.txt  guest@192.168.10.100:/temp
guest@192.168.10.100's password:
b.txt                                                                                                          100%   58    56.6KB/s   00:00
PS C:\Users\aaa>
```

```bash
[root@Server-A ~]# ls  -ld  /temp/*.txt
-rw-r--r-- 1 guest guest 58  7월 16 15:45 /temp/a.txt
-rw-r--r-- 1 guest guest 58  7월 16 15:49 /temp/b.txt
```

---

#### Windows  <---  Linux Server

**EX2)** 리눅스 서버 /etc/hosts 파일을 Windows 바탕화면으로 다운로드해야한다.

```bash
PS C:\Users\aaa> scp  guest@192.168.10.100:/etc/hosts  C:\Users\aaa\Desktop
guest@192.168.10.100's password:
hosts
```

PS C:\Users\aaa> cd C:\Users\aaa\Desktop

PS C:\Users\aaa\Desktop> dir

    디렉터리: C:\Users\aaa\Desktop

Mode                 	LastWriteTime      		Length Name
- ---                 	-------------     		------ ----
- a----      	2026-07-16   오후 3:56	158     hosts
- a----      	2026-07-02   오후 5:51  	2328    Microsoft Edge.lnk

---

#### Windows  --->  Linux Server

**EX3)** 아래의 조건에 맞게 SCP를 사용하여 파일을 업/다운로드 해야 한다. 
  - 윈도우 C:\data 폴더 안에 'project' 폴더를 생성 후 'project' 폴더안에 파일 및 디렉터리를 생성 (임의로 생성)
  - 'project' 폴더안의 모든 파일 및 디렉터리들을 리눅스 /home/guest/ 디렉터리로 업로드 해야 한다.

- C:\data\project\linux.txt		# 파일 생성
- C:\data\project\cisco.txt       	# 파일 생성
- C:\data\project\soldesk.txt         	# 파일 생성
- C:\data\project\system.txt     	# 파일 생성

```bash
PS C:\Users\ryu>  scp   -r   C:\data\project\*  guest@192.168.10.100:/home/guest/
guest@192.168.10.100's password:
linux.txt                                                         	100%   15     0.0KB/s   00:00
cisco.txt                                                           	100%   15     0.0KB/s   00:00
soldesk.txt                                                          	100%   15     0.0KB/s   00:00
system.txt
```

```bash
[root@Server-A ~]# ls  -l  /home/guest/*.txt
-rw-r--r-- 1 guest guest 58  7월 16 16:18 /home/guest/cisco.txt
-rw-r--r-- 1 guest guest 58  7월 16 16:18 /home/guest/linux.txt
-rw-r--r-- 1 guest guest 58  7월 16 16:18 /home/guest/soldesk.txt
-rw-r--r-- 1 guest guest 58  7월 16 16:18 /home/guest/system.txt
```

---

#### Windows  <---  Linux Server

**EX3)** 아래의 조건에 맞게 SCP를 사용하여 파일을 업/다운로드 해야 한다. 
  - 윈도우 C:\data 폴더 안에 'Data Folder' 폴더 생성
  - 리눅스 /home/guest/ 디렉터리의 모든 파일 및 디렉터리를  'Data Folder' 폴더로 다운로드 해야 한다.

```bash
PS C:\Users\aaa> scp  -r  guest@192.168.10.100:/home/guest/*  C:\data\"Data Folder"
guest@192.168.10.100's password:
ALL.tar.bz2                                                                 	100%  194KB 194.0KB/s   00:00
brltty.conf                                                                           	100%   28KB   1.8MB/s   00:00
cisco.txt                                                                             	100%   58     3.5KB/s   00:00
dnsmasq.conf                                                                          	100%   27KB  27.2KB/s   00:00
idmapd.conf                                                                           	100% 5799     5.7KB/s   00:00
kdump.conf                                                                            	100% 9077     8.9KB/s   00:00
ld.so.cache                                                                           	100%   42KB  42.4KB/s   00:00
linux.txt                                                                             	100%   58     3.5KB/s   00:00
login.defs                                                                            	100% 7779     7.6KB/s   00:00
man_db.conf                                                                           	100% 5235     5.1KB/s   00:00
mime.types                                                                            	100%   66KB  65.9KB/s   00:00
nanorc                                                                                	100%   10KB 633.1KB/s   00:00
pnm2ppa.conf                                                                          	100% 6300   384.5KB/s   00:00
protocols                                                                             	100% 6568     6.4KB/s   00:00
services                                                                              	100%  676KB  44.0MB/s   00:00
soldesk.txt                                                                           	100%   58     0.1KB/s   00:00
system.txt                                                                            	100%   58     0.1KB/s   00:00
```

PS C:\Users\aaa> cd C:\data\"Data Folder"

```
PS C:\data\Data Folder> dir

    디렉터리: C:\data\Data Folder

Mode     	           LastWriteTime   	Length 	Name
----     	           -------------   	------	----
-a---- 	2026-07-16    오후 4:19 	198676	ALL.tar.bz2
-a---- 	2026-07-16    오후 4:19      	28974 	brltty.conf
-a----  	2026-07-16    오후 4:19       	58 	cisco.txt
-a----  	2026-07-16    오후 4:19    	27839 	dnsmasq.conf
-a----	2026-07-16    오후 4:19     	5799 	idmapd.conf
-a----  	2026-07-16    오후 4:19   	9077 	kdump.conf
-a----   	2026-07-16    오후 4:19      	43431 	ld.so.cache
-a----  	2026-07-16    오후 4:19      	58 	linux.txt
-a----   	2026-07-16    오후 4:19       	7779 	login.defs
-a----  	2026-07-16    오후 4:19      	5235 	man_db.conf
-a----  	2026-07-16    오후 4:19     	67454 	mime.types
-a----  	2026-07-16    오후 4:19      	10373 	nanorc
-a---- 	2026-07-16    오후 4:19      	6300 	pnm2ppa.conf
-a----  	2026-07-16    오후 4:19      	6568 	protocols
-a----    	2026-07-16    오후 4:19     	692252 	services
-a----  	2026-07-16    오후 4:19     	58 	soldesk.txt
-a----   	2026-07-16    오후 4:19       	58 	system.txt
```

---

**정리**: SCP (Secure Copy) (10-2) 섹션에서 다룬 명령어와 실습 예제를 통해 관련 기능의 구성 방법과 동작 원리를 확인했다.

## vsFTP (10-3)

### vsFTP (Very Scurity FTP)

  - FTP  (File Transfer Protocol)

- FTP는 네트워크를 통해 파일을 송수신(업로드/다운로드) 하기 위한 TCP/IP 기반의 표준 프로토콜
- 사용자가 원격 서버에 접속해서 파일을 주고받는 전용 통신 규칙
- FTP는 TCP Port number 20 , 21을 사용한다.
- TCP 21 (FTP)	: FTP Server    <------    FTP Client  (제어 연결)
- TCP 20 (FTP Data)	: FTP Server    ------>    FTP Client  (데이터 전송)
- 인증 방식은 계정(ID, Password) 기반 로그인방식이며 평문 전송(암호화 없음, 보안 약함)

#### vsFTP (Very Secure FTP)

- vsFTP(Very Secure FTP Daemon)는 Linux에서 가장 널리 사용되는 FTP 서버 프로그램
- 이름 그대로 매우 안전한 FTP 서버 를 목표로 설계된 오픈소스 소프트웨어다.
- 보안과 속도, 안정성이 뛰어나서 Red Hat, Rocky, CentOS, Ubuntu 등 대부분 배포판에서 기본 FTP 서버로 제공된다.

---

**EX1)** Server-A에서 vsftp 설치 확인 및 vsFTP 설치

```
root@Server-A ~]# rpm  -qa | grep  ftp
```

```bash
[root@Server-A ~]# dnf  install  -y  vsftpd
```

```bash
[root@Server-A ~]# rpm  -qa | grep  ftp
vsftpd-3.0.5-8.el9.x86_64
```

```bash
[root@Server-A ~]# systemctl  status  vsftpd
○ vsftpd.service - Vsftpd ftp daemon
     Loaded: loaded (/usr/lib/systemd/system/vsftpd.service; disabled; preset: disabled)
     Active: inactive (dead)
```

```bash
[root@Server-A ~]# systemctl  start  vsftpd	# vsftp 동작
```

```bash
[root@Server-A ~]# systemctl  enable  vsftpd	# 서버 재부팅시에도 자동으로 vsftp를 실행
Created symlink /etc/systemd/system/multi-user.target.wants/vsftpd.service → /usr/lib/systemd/system/vsftpd.service.
```

```bash
[root@Server-A ~]# systemctl  status  vsftpd	# vsftp 상태 확인
● vsftpd.service - Vsftpd ftp daemon
     Loaded: loaded (/usr/lib/systemd/system/vsftpd.service; enabled; preset: disabled)
     Active: active (running) since Thu 2026-07-16 16:44:06 KST; 7s ago
   Main PID: 6942 (vsftpd)
      Tasks: 1 (limit: 10321)
     Memory: 864.0K (peak: 1.2M)
        CPU: 3ms
     CGroup: /system.slice/vsftpd.service
             └─6942 /usr/sbin/vsftpd /etc/vsftpd/vsftpd.conf

 7월 16 16:44:06 Server-A systemd[1]: Starting Vsftpd ftp daemon...
 7월 16 16:44:06 Server-A systemd[1]: Started Vsftpd ftp daemon.
```

---

**EX2-1)** Windows PC에서 Server-A로 FTP 접속

C:\Users\aaa> ftp  192.168.10.100
> ftp: connect :연결 시간 초과
ftp>

```bash
[root@Server-A ~]# firewall-cmd  --permanent  --add-service=ftp
success
```

```bash
[root@Server-A ~]# firewall-cmd  --permanent  --add-port=20/tcp
success
```

```bash
[root@Server-A ~]# firewall-cmd  --permanent  --add-port=21/tcp
success
```

```bash
[root@Server-A ~]# firewall-cmd  --reload
success
```

```bash
[root@Server-A ~]# firewall-cmd  --list-port
20/tcp 21/tcp 22/tcp 111/tcp 139/tcp 445/tcp 2002/tcp 2049/tcp 111/udp 137/udp 138/udp
```

```bash
[root@Server-A ~]# firewall-cmd  --list-services
cockpit dhcpv6-client ftp nfs rpc-bind samba ssh
```

```bash
[root@Server-A ~]# firewall-cmd  --list-all
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: ens160
  sources:
  services: cockpit dhcpv6-client ftp nfs rpc-bind samba ssh
  ports: 111/tcp 111/udp 2049/tcp 445/tcp 139/tcp 137/udp 138/udp 2002/tcp 22/tcp 20/tcp 21/tcp
  protocols:
  forward: yes
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:
```

**EX2-3)** Windows PC에서 Server-A로 FTP 접속

```
	# 1) PC에서 cmd를 사용하여 ftp 접속

C:\Users\aaa> ftp  192.168.10.100
192.168.10.100에 연결되었습니다.
220 (vsFTPd 3.0.5)
200 Always in UTF8 mode.
사용자(192.168.10.100:(none)): guest
331 Please specify the password.
암호: 1234
230 Login successful.
ftp>
ftp> dir
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
-rw-r--r--    1 0          0          198676 Jul 10 03:13 ALL.tar.bz2
-rw-r--r--    1 0          0           28974 Jul 10 00:57 brltty.conf
-rw-r--r--    1 1000     1000           58 Jul 16 07:18 cisco.txt
-rw-r--r--    1 0          0           27839 Jul 10 00:57 dnsmasq.conf
-rw-r--r--    1 0          0            5799 Jul 10 00:57 idmapd.conf
-rw-r--r--    1 0          0            9077 Jul 10 00:57 kdump.conf
-rw-r--r--    1 0          0           43431 Jul 10 00:57 ld.so.cache
-rw-r--r--    1 1000     1000           58 Jul 16 07:18 linux.txt
-rw-r--r--    1 0          0            7779 Jul 10 00:57 login.defs
-rw-r--r--    1 0          0            5235 Jul 10 00:57 man_db.conf
-rw-r--r--    1 0          0           67454 Jul 10 00:57 mime.types
-rw-r--r--    1 0          0           10373 Jul 10 00:57 nanorc
-rw-r--r--    1 0          0            6300 Jul 10 00:57 pnm2ppa.conf
-rw-r--r--    1 0          0            6568 Jul 10 00:57 protocols
-rw-r--r--    1 0          0          692252 Jul 10 00:57 services
-rw-r--r--    1 1000     1000           58 Jul 16 07:18 soldesk.txt
-rw-r--r--    1 1000     1000           58 Jul 16 07:18 system.txt
226 Directory send OK.
ftp: 0.09초 12.34KB/초
ftp>
```

  - 2) FTP 툴(알 FTP, 다 FTP, filezilla)을 사용하여 FTP Server로 접속

#### 3) 웹브라우저 접속 (현재는 대부분의 브라우저에서 접속을 차단)

---

```bash
	# FTP 접속 제한

[root@Server-A ~]# rm -rf  /backup/*
```

```bash
[root@Server-A ~]# cp  /etc/vsftpd/vsftpd.conf  /backup
```

```bash
[root@Server-A ~]# ls  -l  /backup
합계 8
-rw------- 1 root root 5039  7월 16 17:01 vsftpd.conf
```

```bash
[root@Server-A ~]# vi  /etc/vsftpd/vsftpd.conf
~~~~~~~~~ 중간 생략 ~~~~~~~~~
     11 # Allow anonymous FTP? (Beware - allowed by default if you comment this out).
     12 anonymous_enable=NO		# 익명 사용자 차단
     13 #
     14 # Uncomment this to allow local users to log in.
     15 local_enable=YES		# 일반 사용자
     16 #
~~~~~~~~~ 중간 생략 ~~~~~~~~~
```

```bash
	# 익명 사용자 허용

[root@Server-A ~]# vi  /etc/vsftpd/vsftpd.conf
~~~~~~~~~ 중간 생략 ~~~~~~~~~

     11 # Allow anonymous FTP? (Beware - allowed by default if you comment this out).
     12 #anonymous_enable=NO
     13 anonymous_enable=YES	# 익명사용자 허용
     14 #
     15 # Uncomment this to allow local users to log in.
     16 local_enable=YES
     17 #

~~~~~~~~~ 중간 생략 ~~~~~~~~~
```

```bash
[root@Server-A ~]# systemctl  restart  vsftpd
```

C:\Users\soldesk> ftp 192.168.10.100
192.168.10.100에 연결되었습니다.
220 (vsFTPd 3.0.5)
200 Always in UTF8 mode.
사용자(192.168.10.100:(none)): anonymous
331 Please specify the password.
암호:
230 Login successful.
ftp>
ftp> pwd
257 "/" is the current directory
ftp> quit
221 Goodbye.

```bash
	# 익명 사용자 차단

[root@Server-A ~]# vi  /etc/vsftpd/vsftpd.conf
~~~~~~~~~ 중간 생략 ~~~~~~~~~
     11 # Allow anonymous FTP? (Beware - allowed by default if you comment this out).
     12 anonymous_enable=NO		# 익명 사용자 차단
     13 #
     14 # Uncomment this to allow local users to log in.
     15 local_enable=YES		# 일반 사용자
     16 #
~~~~~~~~~ 중간 생략 ~~~~~~~~~
```

```bash
[root@Server-A ~]# systemctl  restart  vsftpd
```

C:\Users\soldesk> ftp 192.168.10.100
192.168.10.100에 연결되었습니다.
220 (vsFTPd 3.0.5)
200 Always in UTF8 mode.
사용자(192.168.10.100:(none)): anonymous	# 익명으로 접속 X
331 Please specify the password.
암호:
530 Login incorrect.
로그인하지 못했습니다.
ftp>

C:\Users\soldesk> ftp 192.168.10.100
192.168.10.100에 연결되었습니다.
220 (vsFTPd 3.0.5)
200 Always in UTF8 mode.
사용자(192.168.10.100:(none)): guest		# 사용자 계정으로 접속 O
331 Please specify the password.
암호:
230 Login successful.
ftp>

**EX3-2)** 아래의 조건에 맞게 파일을 생성 및 복사해야한다.
  - '/etc/a*' 파일과 '/etc/b*' 파일을 guest 계정의 홈 디렉터리로 복사
  - Windows PC의 C드라이브에 'abc' 폴더를 생성한 후 'abc.txt' , 'def.txt' , '1234.txt' , '5678.txt'파일을 생성하시오

```bash
[root@Server-A ~]# cp  -rf  /etc/a*  /etc/b*  /home/guest
```

```bash
[root@Server-A ~]# \cp  -rf  /etc/a*  /etc/b*  /home/guest	# alias 옵션 무시

[root@Server-A ~]# ls  -l  /home/guest
합계 1176
-rw-r--r-- 1 root  root  198676  7월 10 12:13 ALL.tar.bz2
drwxr-xr-x 3 root  root      28  7월 16 17:19 accountsservice
-rw-r--r-- 1 root  root      16  7월 16 17:19 adjtime
-rw-r--r-- 1 root  root    1529  7월 16 17:19 aliases
drwxr-xr-x 3 root  root      65  7월 16 17:19 alsa
drwxr-xr-x 2 root  root    4096  7월 16 17:19 alternatives
-rw-r--r-- 1 root  root     541  7월 16 17:19 anacrontab
-rw-r--r-- 1 root  root     269  7월 16 17:19 anthy-unicode.conf
-rw-r--r-- 1 root  root     833  7월 16 17:19 appstream.conf
-rw-r--r-- 1 root  root      55  7월 16 17:19 asound.conf
-rw-r--r-- 1 root  root       1  7월 16 17:19 at.deny
drwxr-x--- 4 root  root     100  7월 16 17:19 audit
drwxr-xr-x 3 root  root    4096  7월 16 17:19 authselect
drwxr-xr-x 4 root  root      71  7월 16 17:19 avahi
drwxr-xr-x 2 root  root     124  7월 16 17:17 bash_completion.d
-rw-r--r-- 1 root  root    2658  7월 16 17:19 bashrc
-rw-r--r-- 1 root  root     535  7월 16 17:19 bindresvport.blacklist
drwxr-xr-x 2 root  root       6  7월 16 17:17 binfmt.d
dr-xr-xr-x 2 root  root      61  7월 16 17:17 bluetooth
-rw-r----- 1 root  root      33  7월 16 17:19 brlapi.key
drwxr-xr-x 7 root  root      84  7월 16 17:17 brltty
-rw-r--r-- 1 root  root   28974  7월 16 17:19 brltty.conf
-rw-r--r-- 1 guest guest     58  7월 16 16:18 cisco.txt
-rw-r--r-- 1 root  root   27839  7월 10 09:57 dnsmasq.conf
-rw-r--r-- 1 root  root    5799  7월 10 09:57 idmapd.conf
-rw-r--r-- 1 root  root    9077  7월 10 09:57 kdump.conf
-rw-r--r-- 1 root  root   43431  7월 10 09:57 ld.so.cache
-rw-r--r-- 1 guest guest     58  7월 16 16:18 linux.txt
-rw-r--r-- 1 root  root    7779  7월 10 09:57 login.defs
-rw-r--r-- 1 root  root    5235  7월 10 09:57 man_db.conf
-rw-r--r-- 1 root  root   67454  7월 10 09:57 mime.types
-rw-r--r-- 1 root  root   10373  7월 10 09:57 nanorc
-rw-r--r-- 1 root  root    6300  7월 10 09:57 pnm2ppa.conf
-rw-r--r-- 1 root  root    6568  7월 10 09:57 protocols
-rw-r--r-- 1 root  root  692252  7월 10 09:57 services
-rw-r--r-- 1 guest guest     58  7월 16 16:18 soldesk.txt
-rw-r--r-- 1 guest guest     58  7월 16 16:18 system.txt
```

C:\Users\aaa> ftp  192.168.10.100
192.168.10.100에 연결되었습니다.
220 (vsFTPd 3.0.5)
200 Always in UTF8 mode.
사용자(192.168.10.100:(none)): guest
331 Please specify the password.
암호:
230 Login successful.
ftp>
ftp> pwd
257 "/home/guest" is the current directory
ftp>
ftp> put  abc.txt
abc.txt: 파일을 찾을 수 없습니다.

- FTP로 파일을 업로드하기위해서는 해당 디렉터리로 이동해야 한다.

C:\Users\aaa> dir
 C 드라이브의 볼륨에는 이름이 없습니다.
 볼륨 일련 번호: EC56-B080

 C:\Users\aaa 디렉터리

2026-07-16  오후 03:45    <DIR>          .
2026-07-16  오후 03:45    <DIR>          ..
2026-07-16  오후 03:45    <DIR>          .ssh
2026-07-02  오후 05:50    <DIR>          3D Objects
2026-07-02  오후 05:50    <DIR>          Contacts
2026-07-16  오후 03:56    <DIR>          Desktop
2026-07-02  오후 05:50    <DIR>          Documents
2026-07-16  오후 04:52    <DIR>          Downloads
2026-07-02  오후 05:50    <DIR>          Favorites
2026-07-02  오후 05:50    <DIR>          Links
2026-07-02  오후 05:50    <DIR>          Music
2026-07-02  오후 05:51    <DIR>          OneDrive
2026-07-02  오후 05:51    <DIR>          Pictures
2026-07-02  오후 05:50    <DIR>          Saved Games
2026-07-02  오후 05:51    <DIR>          Searches
2026-07-02  오후 05:50    <DIR>          Videos
               0개 파일                   0 바이트
              16개 디렉터리  38,271,254,528 바이트 남음

C:\Users\aaa> cd C:\abc		# 업로드할 파일이 있는 디렉터리로 이동

C:\abc>dir
 C 드라이브의 볼륨에는 이름이 없습니다.
 볼륨 일련 번호: EC56-B080

 C:\abc 디렉터리

2026-07-16  오후 05:47    <DIR>          .
2026-07-16  오후 05:47    <DIR>          ..
2026-07-16  오후 03:43                58 1234.txt
2026-07-16  오후 03:43                58 5678.txt
2026-07-16  오후 03:43                58 a.txt
2026-07-16  오후 03:43                58 abc.txt
2026-07-16  오후 03:43                58 def.txt
               5개 파일                 290 바이트
               2개 디렉터리  38,270,685,184 바이트 남음

#### 파일 업로드

C:\abc> ftp  192.168.10.100		# FTP 접속 후 업로드
192.168.10.100에 연결되었습니다.
220 (vsFTPd 3.0.5)
200 Always in UTF8 mode.
사용자(192.168.10.100:(none)): guest
331 Please specify the password.
암호:
230 Login successful.
ftp>
ftp> put  abc.txt
200 PORT command successful. Consider using PASV.
150 Ok to send data.
226 Transfer complete.
ftp: 0.01초 3.87KB/초
ftp>

```bash
[root@Server-A ~]# ls  -l  /home/guest | grep abc
-rw-r--r-- 1 guest guest     58  7월 16 17:48 abc.txt	# 파일 업로드 확인
```

```bash
[root@Server-A ~]# ls  -l  /home/guest | grep .txt
-rw-r--r-- 1 guest guest     58  7월 16 17:48 abc.txt	# 파일 업로드 확인
-rw-r--r-- 1 guest guest     58  7월 16 16:18 cisco.txt
-rw-r--r-- 1 guest guest     58  7월 16 16:18 linux.txt
-rw-r--r-- 1 guest guest     58  7월 16 16:18 soldesk.txt
-rw-r--r-- 1 guest guest     58  7월 16 16:18 system.txt
```

C:\abc> cd c:\data

c:\data> dir
 C 드라이브의 볼륨에는 이름이 없습니다.
 볼륨 일련 번호: EC56-B080
 c:\data 디렉터리
2026-07-16  오후 04:17    <DIR>          .
2026-07-16  오후 04:17    <DIR>          ..
2026-07-16  오후 03:43                58 a.txt
2026-07-16  오후 03:43                58 b.txt
2026-07-16  오후 04:19    <DIR>          Data Folder
2026-07-16  오후 04:17    <DIR>          project
               2개 파일                 116 바이트
               4개 디렉터리  38,270,205,952 바이트 남음

c:\data> ftp  192.168.10.100
192.168.10.100에 연결되었습니다.
220 (vsFTPd 3.0.5)
200 Always in UTF8 mode.
사용자(192.168.10.100:(none)): guest
331 Please specify the password.
암호:
230 Login successful.
ftp>
ftp> put a.txt
200 PORT command successful. Consider using PASV.
150 Ok to send data.
226 Transfer complete.
ftp: 0.00초 58000.00KB/초
ftp>

```bash
[root@Server-A ~]# ls  -l  /home/guest | grep .txt
-rw-r--r-- 1 guest guest     58  7월 16 17:53 a.txt		# 파일 업로드 확인
-rw-r--r-- 1 guest guest     58  7월 16 17:48 abc.txt
-rw-r--r-- 1 guest guest     58  7월 16 16:18 cisco.txt
-rw-r--r-- 1 guest guest     58  7월 16 16:18 linux.txt
-rw-r--r-- 1 guest guest     58  7월 16 16:18 soldesk.txt
-rw-r--r-- 1 guest guest     58  7월 16 16:18 system.txt
```

#### 파일 다운로드

c:\data> ftp  192.168.10.100
192.168.10.100에 연결되었습니다.
220 (vsFTPd 3.0.5)
200 Always in UTF8 mode.
사용자(192.168.10.100:(none)): guest
331 Please specify the password.
암호:
230 Login successful.
ftp> get adjtime
200 PORT command successful. Consider using PASV.
150 Opening BINARY mode data connection for adjtime (16 bytes).
226 Transfer complete.
ftp: 0.00초 16000.00KB/초
ftp>

C:\data> dir
 C 드라이브의 볼륨에는 이름이 없습니다.
 볼륨 일련 번호: EC56-B080

 C:\data 디렉터리

2026-07-16  오후 05:56    <DIR>          .
2026-07-16  오후 05:56    <DIR>          ..
2026-07-16  오후 03:43                  58 a.txt
2026-07-16  오후 05:56                  16 adjtime		# 파일 다운로드 확인
2026-07-16  오후 03:43                  58 b.txt
2026-07-16  오후 04:19    <DIR>          Data Folder
2026-07-16  오후 04:17    <DIR>          project
               3개 파일                 132 바이트
               4개 디렉터리  38,268,448,768 바이트 남음

```bash
[root@Server-A ~]# vi  /etc/vsftpd/vsftpd.conf
~~~~~~~~~~ 중간 생략 ~~~~~~~~~~
     14 # Uncomment this to allow local users to log in.
     15 local_enable=YES
     16 #
     17 # Uncomment this to enable any form of FTP write command.
     18 write_enable=NO	# write_enable을 NO로 변경
     19 #
     20 # Default umask for local users is 077. You may wish to change this to 022,
     21 # if your users expect that (022 is used by most other ftpd's)
     22 local_umask=022
     23 #
~~~~~~~~~~ 중간 생략 ~~~~~~~~~~
```

```bash
[root@Server-A ~]# systemctl  restart  vsftpd
```

C:\data> dir
 C 드라이브의 볼륨에는 이름이 없습니다.
 볼륨 일련 번호: EC56-B080

 C:\data 디렉터리

2026-07-16  오후 05:56    <DIR>          .
2026-07-16  오후 05:56    <DIR>          ..
2026-07-16  오후 03:43                  58 a.txt
2026-07-16  오후 05:56                  16 adjtime
2026-07-16  오후 03:43                  58 b.txt
2026-07-16  오후 04:19    <DIR>          Data Folder
2026-07-16  오후 04:17    <DIR>          project
               3개 파일                 132 바이트
               4개 디렉터리  38,268,448,768 바이트 남음

c:\data> ftp  192.168.10.100
192.168.10.100에 연결되었습니다.
220 (vsFTPd 3.0.5)
200 Always in UTF8 mode.
사용자(192.168.10.100:(none)): guest
331 Please specify the password.
암호:
230 Login successful.
ftp>
ftp> put  b.txt		# 파일 업로드가 제한된다.
200 PORT command successful. Consider using PASV.
550 Permission denied.
ftp>
ftp> get  aliases		# 다운로드는 가능
200 PORT command successful. Consider using PASV.
150 Opening BINARY mode data connection for aliases (1529 bytes).
226 Transfer complete.
ftp: 0.00초 1529000.00KB/초
ftp>

#### FTP 파일 다운로드 로그 확인

```bash
[root@Server-A ~]# vi  /etc/vsftpd/vsftpd.conf
~~~~~~~~~~ 중간 생략 ~~~~~~~~~~
     34 # Activate directory messages - messages given to remote users when they
     35 # go into a certain directory.
     36 dirmessage_enable=YES
     37 #
     38 # Activate logging of uploads/downloads.
     39 xferlog_enable=YES		# FTP 업/다운로드 로그
     40 #
     41 # Make sure PORT transfer connections originate from port 20 (ftp-data).
     42 connect_from_port_20=YES
     43 #
~~~~~~~~~~ 중간 생략 ~~~~~~~~~~

-로그 정보는 기본적으로 /var/log 안에 생성된다.
```

```bash
[root@Server-A ~]# cat  /var/log/xferlog
Thu Jul 16 17:48:15 2026 1 ::ffff:192.168.10.131 58    /home/guest/abc.txt a _ i r guest ftp 0 * c
Thu Jul 16 17:53:11 2026 1 ::ffff:192.168.10.131 58    /home/guest/a.txt a    _ i r guest ftp 0 * c
Thu Jul 16 17:56:05 2026 1 ::ffff:192.168.10.131 16    /home/guest/adjtime a _ o r guest ftp 0 * c
Thu Jul 16 18:01:23 2026 1 ::ffff:192.168.10.131 1529 /home/guest/aliases a  _ o r guest ftp 0 * c

# Thu Jul 16 17:48:15 2026	: 전송시간
# 1			: 파일 전송에 걸린 시간
# 192.168.10.131		: 파일을 업/다운로드한 IP 주소
# /home/guest/abc.txt	: 업/다운로드한 경로/파일명
# 58			: 전송한 파일의 크기
# i 			: incomming (업로드)
# o			: outgoing (다운로드)
# r			: 일반 사용자 (a = 익명사용자)
# guest			: 사용자 계정
# c			: c = complete 정상 전송 , i = incomplete (미완료 또는 비정상)
```

---

**EX4)** 특정 사용자만 FTP 접속 허용(userlist_enable)

- 혀용 또는 차단할 사용자를 userlist에 설정하여 지정된 사용자만 접속이 가능하게 설정이가능하다.

```bash
[root@Server-A ~]# vi  /etc/vsftpd/user_list
root
bin
daemon
adm
lp
sync
shutdown
halt
mail
news
uucp
operator
games
nobody
guest		# 차단할 계정 추가 
```

```bash
[root@Server-A ~]# vi  /etc/vsftpd/vsftpd.conf
    121 # files.
    122 # Make sure, that one of the listen options is commented !!
    123 listen_ipv6=YES
    124
    125 pam_service_name=vsftpd
    126 userlist_enable=YES
         userlist_deny=YES	<--- 설정파일에 확인되지 않는다. (기본값)
```

- userlist_deny=YES	: userlist에 등록된 사용자의 접근을 차단 (기본값)
- userlist_deny=NO		: userlist에 등록된 사용자의 접근을 허용

C:\Users\soldesk> ftp  192.168.10.100
192.168.10.100에 연결되었습니다.
220 (vsFTPd 3.0.5)
200 Always in UTF8 mode.
사용자(192.168.10.100:(none)): user1	# guest 계정이 아닌 나머지 계정은 vsFTP로 접속 가능
331 Please specify the password.
암호:
230 Login successful.
ftp>

C:\Users\soldesk> ftp  192.168.10.100
192.168.10.100에 연결되었습니다.
220 (vsFTPd 3.0.5)
200 Always in UTF8 mode.
사용자(192.168.10.100:(none)): guest	# guest 계정은 password도 확인하지 않는다.
530 Permission denied.
로그인하지 못했습니다.
ftp>

---

**EX5)** 지정된 사용자만 vsFTP로 업로드가 가능해야 한다.

```bash
[root@Server-A ~]# vi  /etc/vsftpd/vsftpd.conf
~~~~~~~~~~~ 중간 생략 ~~~~~~~~~~~
     14 # Uncomment this to allow local users to log in.
     15 local_enable=YES
     16 #
     17 # Uncomment this to enable any form of FTP write command.
     18 write_enable=NO		# 모든 계정의 파일 업로드 차단
     19 #
     20 # Default umask for local users is 077. You may wish to change this to 0 
     21 # if your users expect that (022 is used by most other ftpd's)
     22 local_umask=022
     23 #
~~~~~~~~~~~ 중간 생략 ~~~~~~~~~~~
```

```bash
[root@Server-A ~]# mkdir  /etc/vsftpd/userconfig	# vsFTP의 계정을 관리할 디렉터리 생성
```

```bash
[root@Server-A ~]# ls  -l  /etc/vsftpd
합계 20
-rw------- 1 root root   125  1월 17  2026 ftpusers
-rw------- 1 root root   361  7월 20 09:52 user_list
drwxr-xr-x 2 root root      6  7월 20 09:57 userconfig	# 디렉터리 생성 확인
-rw------- 1 root root 5038  7월 16 17:59 vsftpd.conf
-rwxr--r-- 1 root root   352  1월 17  2026 vsftpd_conf_migrate.sh
```

```bash
	# touch 또는 vi를 사용하여 계정명의 파일을 생성

[root@Server-A ~]# vi  /etc/vsftpd/userconfig/guest
write_enable=YES

:wq
```

```bash
[root@Server-A ~]# vi  /etc/vsftpd/vsftpd.conf		# vsftpd.conf 파일에 위에서 설정한 기능을 적용해야 한다.
~~~~~~~~~~~ 중간 생략 ~~~~~~~~~~~
    121 # files.
    122 # Make sure, that one of the listen options is commented !!
    123 listen_ipv6=YES
    124
    125 pam_service_name=vsftpd
    126 userlist_enable=YES
    127 user_config_dir=/etc/vsftpd/userconfig		# 설정 적용
```

```bash
[root@Server-A ~]# systemctl  restart  vsftpd
```

- user1 계정을 사용하여 vfFTP 접속 후 user1.txt 파일 업로드
- guest 계정을 사용하여 vfFTP 접속 후 guest.txt 파일 업로드

c:\abc> ftp  192.168.10.100
192.168.10.100에 연결되었습니다.
220 (vsFTPd 3.0.5)
200 Always in UTF8 mode.
사용자(192.168.10.100:(none)): user1		# user1 계정으로 접속
331 Please specify the password.
암호:
230 Login successful.
ftp>
ftp> put user1.txt				# user1.txt 파일 업로드
200 PORT command successful. Consider using PASV.
550 Permission denied.			# 파일 업로드 제한 확인

c:\abc> ftp  192.168.10.100
192.168.10.100에 연결되었습니다.
220 (vsFTPd 3.0.5)
200 Always in UTF8 mode.
사용자(192.168.10.100:(none)): guest		# guest 계정으로 접속
331 Please specify the password.
암호:
230 Login successful.
ftp> put guest.txt				# guest.txt 파일 업로드
200 PORT command successful. Consider using PASV.
150 Ok to send data.
226 Transfer complete.			# 파일 업로드 가능
ftp: 0.00초 7820.00KB/초

#### 실습을 유지하기위해서 user1도 업로드 가능해야 한다.

```bash
[root@Server-A ~]# vi  /etc/vsftpd/userconfig/user1
write_enable=YES

:wq
```

```bash
[root@Server-A ~]# systemctl  restart  vsftpd
```

c:\abc> ftp  192.168.10.100
192.168.10.100에 연결되었습니다.
220 (vsFTPd 3.0.5)
200 Always in UTF8 mode.
사용자(192.168.10.100:(none)): user1
331 Please specify the password.
암호:
230 Login successful.
ftp>
ftp>
ftp> put  user1.txt			# 파일 업로드
200 PORT command successful. Consider using PASV.
150 Ok to send data.
226 Transfer complete.		# 파일 업로드 확인
ftp: 0.00초 23460000.00KB/초

---

#### chroot (Change Root)

- chroot란 특정 사용자 또는 프로세스를 가짜 루트(/) 안에 가두는 기술이다.

- 원래 모든 Linux 계정은 / (root filesystem)를 기준으로 전체 파일 시스템을 탐색할 수 있다.
  하지만 chroot를 적용하면 사용자가 보게 되는 최상위 디렉터리가 바뀐다.

- 설정 전 접속 :  /home/guest
- 설정 후 접속 :  guest는 /home/guest 를 '/'라고 인식한다.

- 사용자는 자신의 chroot 공간 밖으로 벗어날 수 없다.

- 시스템 전체 구성 파일(/etc/passwd, /var/log 등)에 접근할 수 없기때문에 보안적으로 매우 유리한 격리 환경을 만든다.

- chroot는 다음과 같은 목적에 많이 사용된다.
  - FTP 사용자 격리
  - Web 서버/FTP 서버 보안 강화
  - 악성코드 분석용 샌드박스 환경
  - 테스트용 가상 루트 환경 구성

- vsFTP에서 chroot를 사용하는 이유
  - FTP는 원래 보안 측면에서 취약하다. 특히 로컬 계정(guest, user1 등)을 FTP 접속용으로 사용할 경우
   사용자들이 의도치 않게 시스템 파일을 열어보거나 잘못된 설정으로 다른 사용자의 파일을 열어보는 사고가 발생 가능
   그래서 FTP 계정이 접속하면 자신의 홈 디렉터리 안에만 가두는 것이 매우 중요하다.
  - 예 : guest는 /home/guest 밖으로 절대 나갈 수 없다.
  - cd /etc 같은 명령 실행해도 실패
  - 시스템 상위 디렉토리 접근 차단
  - 즉 chroot는 "사용자 1명 = 하나의 독립된 감옥"을 만들어 보안성을 대폭 올리는 기능이다.

```bash
[root@Server-A ~]# vi  /etc/vsftpd/vsftpd.conf
~~~~~~~~~~~ 중간 생략 ~~~~~~~~~~~
     99 # chroot)
    100 chroot_local_user=YES			# 모든 사용자에게 chroot 적용
    101 allow_writeable_chroot=YES		# chroot 사용자에게 업로드 허용
    102 #chroot_list_enable=YES
    103 # (default follows)
    104 #chroot_list_file=/etc/vsftpd/chroot_list
    105 #
~~~~~~~~~~~ 중간 생략 ~~~~~~~~~~~
```

```bash
[root@Server-A ~]# systemctl  restart  vsftpd
```

c:\abc> ftp  192.168.10.100
192.168.10.100에 연결되었습니다.
220 (vsFTPd 3.0.5)
200 Always in UTF8 mode.
사용자(192.168.10.100:(none)): guest
331 Please specify the password.
암호:
230 Login successful.
ftp>
ftp> pwd
257 "/" is the current directory	# chroot 적용전 = /home/guest
ftp>
ftp> cd  /etc/
550 Failed to change directory.	# 디렉터리를 이동할 수 없다.
ftp>
ftp> ls  -l  /etc/			# ls 명령어로 확인할 수 없다.
200 PORT command successful. Consider using PASV.
로컬 파일 여는 동안 오류가 발생했습니다. /etc/
> /:잘못된 인수입니다.
ftp>

```bash
[root@Server-A ~]# ls  -l  /home/guest	# guest의 홈 디렉터리안의 파일 확인
합계 1120
-rw-r--r-- 1 root  root  198676  7월 10 12:13 ALL.tar.bz2
-rw-r--r-- 1 guest guest     58  7월 16 16:18 cisco.txt
-rw-r--r-- 1 root  root   27839  7월 10 09:57 dnsmasq.conf
-rw-r--r-- 1 guest guest  23460  7월 20 10:07 guest.txt
-rw-r--r-- 1 root  root    5799  7월 10 09:57 idmapd.conf
-rw-r--r-- 1 root  root    9077  7월 10 09:57 kdump.conf
-rw-r--r-- 1 root  root   43431  7월 10 09:57 ld.so.cache
-rw-r--r-- 1 guest guest     58  7월 16 16:18 linux.txt
-rw-r--r-- 1 root  root    7779  7월 10 09:57 login.defs
-rw-r--r-- 1 root  root    5235  7월 10 09:57 man_db.conf
-rw-r--r-- 1 root  root   67454  7월 10 09:57 mime.types
-rw-r--r-- 1 root  root   10373  7월 10 09:57 nanorc
-rw-r--r-- 1 root  root    6300  7월 10 09:57 pnm2ppa.conf
-rw-r--r-- 1 root  root    6568  7월 10 09:57 protocols
-rw-r--r-- 1 root  root  692252  7월 10 09:57 services
-rw-r--r-- 1 guest guest     58  7월 16 16:18 soldesk.txt
-rw-r--r-- 1 guest guest     58  7월 16 16:18 system.txt
```

```
ftp> pwd
257 "/" is the current directory	# 경로는 최상위 디렉터리로 확인 (/)
ftp>
ftp> ls -l				# 데이터 확인시 guest의 홈 디렉터리안의 파일이 확인된다.
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
-rw-r--r--    1 0        0          198676 Jul 10 03:13 ALL.tar.bz2
-rw-r--r--    1 1000     1000           58 Jul 16 07:18 cisco.txt
-rw-r--r--    1 0        0           27839 Jul 10 00:57 dnsmasq.conf
-rw-r--r--    1 1000     1000        23460 Jul 20 01:07 guest.txt
-rw-r--r--    1 0        0            5799 Jul 10 00:57 idmapd.conf
-rw-r--r--    1 0        0            9077 Jul 10 00:57 kdump.conf
-rw-r--r--    1 0        0           43431 Jul 10 00:57 ld.so.cache
-rw-r--r--    1 1000     1000           58 Jul 16 07:18 linux.txt
-rw-r--r--    1 0        0            7779 Jul 10 00:57 login.defs
-rw-r--r--    1 0        0            5235 Jul 10 00:57 man_db.conf
-rw-r--r--    1 0        0           67454 Jul 10 00:57 mime.types
-rw-r--r--    1 0        0           10373 Jul 10 00:57 nanorc
-rw-r--r--    1 0        0            6300 Jul 10 00:57 pnm2ppa.conf
-rw-r--r--    1 0        0            6568 Jul 10 00:57 protocols
-rw-r--r--    1 0        0          692252 Jul 10 00:57 services
-rw-r--r--    1 1000     1000           58 Jul 16 07:18 soldesk.txt
-rw-r--r--    1 1000     1000           58 Jul 16 07:18 system.txt
```

---

**정리**: vsFTP (10-3) 섹션에서 다룬 명령어와 실습 예제를 통해 관련 기능의 구성 방법과 동작 원리를 확인했다.

## SFTP (10-4)

### SFTP (Secure FTP)

- FTP는 TCP/IP 기반으로 동작하며 Client와 Server가 상호 간 파일을 전송하기 위해 사용하는 오래된 파일 전송 프로토콜이다.

- FTP는 암호화되지 않은 평문 전송 방식이기 때문에 보안에 취약하다.

- FTP는 Port number 20, 21을 사용한다.
 .TCP 21 TCP FTP		(FTP Server  <----   FTP Client)
 .TCP 20 TCP FTP-data	(FTP Server  ---->   FTP Client)

- SSH(Secure Shell)
  - SSH는 인터넷을 통한 원격 접속 시 암호화를 제공하여 안전한 통신을 가능하게 하는 보안 프로토콜이다.
  - SSH 통신은 대칭키 암호화 방식으로 데이터 전송을 보호하며,
   사용자 인증은 비대칭키 암호화(공개키/개인키) 또는 비밀번호 기반 인증을 사용한다.
  - 기본 포트는 TCP 22번이다.

- SFTP
  - SFTP는 SSH(22번 포트) 세션 안에서 동작하는 파일 전송 전용 프로토콜이다.
  - 기존 FTP(20/21번 포트)와는 구조적으로 완전히 다르며, FTP 서버 프로그램(vsftpd)과도 아무 관련이 없다.
  - SSH가 설치되어 있으면 SSH 서버가 자동으로 SFTP 기능을 제공한다.
 (예: /usr/libexec/openssh/sftp-server 또는 internal-sftp)
  - 모든 데이터(인증 정보·파일 내용)가 SSH 암호화 터널 안에서 전송되기 때문에 매우 안전하다.
  - 별도의 FTP 포트 개방이 필요 없고, TCP 22번만 개방하면 된다.

---

```bash
[root@localhost ~]# ssh  A.B.C.D		<---- root로 SSH로 접속
[root@localhost ~]# logout

[root@localhost ~]# ssh  -l  solG  A.B.C.D	<---- 특정 계정으로 SSH로 접속

[root@localhost ~]# ssh  sol@A.B.C.D	<---- 특정 계정으로 SSH로 접속
```

- upload 조건
  - upload할 디렉터리나 윈도우는 업로드할 파일이 있는 폴더로 이동해야한다.

- download 조건
  - download는 경로를 입력할 수 있기때문에 어떤 디렉터리에서 해도 관계가 없다.

---

**EX1-1)** Server-A와 Clinet-L에 SSH 접속
  - Server-A에 'guest'계정으로 접속
  - Client-L에 'sol'계정으로 접속

```bash
	# Server-A로 guest 계정을 사용하여 접속

login as: guest
guest@192.168.10.100's password:
Activate the web console with: systemctl enable --now cockpit.socket

Last failed login: Mon Jul 20 10:54:49 KST 2026 from 192.168.10.1 on ssh:notty
There was 1 failed login attempt since the last successful login.
Last login: Mon Jul 20 10:29:47 2026 from 192.168.10.1
[guest@Server-A ~]$
```

```bash
	# Client-L로 guest 계정으로 접속 후 sol 계정 및 비밀번호 설정

login as: guest
guest@192.168.10.130's password:
Activate the web console with: systemctl enable --now cockpit.socket

Last login: Thu Jul 16 12:11:18 2026 from 192.168.10.1
[guest@Client-L ~]$
[guest@Client-L ~]$
[guest@Client-L ~]$
[guest@Client-L ~]$ su -		# guest 계정으로 접속 후 root 권한 획득
암호:
[root@Client-L ~]#
[root@Client-L ~]# useradd  sol	# sol 계정 생성
[root@Client-L ~]# passwd sol	# sol 계정 비밀번호 생성
sol 사용자의 비밀 번호 변경 중
새 암호:
잘못된 암호: 암호는 8 개의 문자 보다 짧습니다
새 암호 재입력:
passwd: 모든 인증 토큰이 성공적으로 업데이트 되었습니다.
```

```bash
login as: sol			# sol 계정으로 접속
sol@192.168.10.130's password:
[sol@Client-L ~]$
```

**EX1-2)** Client-L에서 아래의 조건에 맞게 temp 디렉터리에 파일 생성및 복사
  - root 계정을 사용하여 '/temp' 디렉터리안의 모든 파일 및 디렉터리를 삭제해야한다. (temp 디렉터리가 없는 경우 디렉터리 생성)
  - Client-L의 'sol'계정을 사용하여 /temp 디렉터리에 /etc 디렉터리의 모든 파일을 복사해야한다.
  - touch 명령어를 사용하여 'sol-a1' ~ 'sol-a3' 파일 생성
  - touch 명령어를 사용하여 'sol-b4' ~ 'sol-b6 파일 생성
  - touch 명령어를 사용하여 'sol-b7' ~ 'sol-b9 파일 생성

```bash
[sol@Client-L ~]$ ls  -l  / | grep temp		# temp 디렉터리가 확인되지 않는다.
```

```bash
[sol@Client-L ~]$ su -			# root 권한으로 변경
암호:
```

```bash
[root@Client-L ~]# mkdir  /temp		# temp 디렉터리 생성
```

```bash
[root@Client-L ~]# chown  sol:sol  /temp	# 소유권 변경
[root@Client-L ~]# chmod  777  /temp		# 허가권 변경

[root@Client-L ~]# ls  -l  / | grep temp
]drwxrwxrwx    2 sol  sol  4096  7월 20 10:59 temp
```

```bash
[root@Client-L ~]# exit			# sol 계정으로 접속
로그아웃
[sol@Client-L ~]$
```

```bash
[sol@Client-L ~]$ cp  /etc/*  /temp/
```

```bash
[sol@Client-L ~]$ ls -l /etc/
합계 1280
-rw-r--r-- 1 sol sol   4673  7월 20 11:10 DIR_COLORS
-rw-r--r-- 1 sol sol   4755  7월 20 11:10 DIR_COLORS.lightbgcolor
-rw-r--r-- 1 sol sol     94  7월 20 11:10 GREP_COLORS
-rw-r--r-- 1 sol sol     16  7월 20 11:10 adjtime
-rw-r--r-- 1 sol sol   1529  7월 20 11:10 aliases
-rw-r--r-- 1 sol sol    541  7월 20 11:10 anacrontab
    ~~~~~~~~~~ 중간 생략 ~~~~~~~~~~
-rw-r--r-- 1 sol sol     28  7월 20 11:10 vconsole.conf
-rw-r--r-- 1 sol sol   4017  7월 20 11:10 vimrc
-rw-r--r-- 1 sol sol   1184  7월 20 11:10 virc
-rw-r--r-- 1 sol sol   4925  7월 20 11:10 wgetrc
-rw-r--r-- 1 sol sol    817  7월 20 11:10 xattr.conf
-rw-r--r-- 1 sol sol    108  7월 20 11:10 yum.conf
```

```bash
[sol@client-l ~]$  touch  /temp/sol-a1   /temp/sol-a2   /temp/sol-a3
[sol@client-l ~]$  touch  /temp/sol-b4   /temp/sol-b5   /temp/sol-b6
[sol@client-l ~]$  touch  /temp/sol-c7   /temp/sol-c8   /temp/sol-c9
```

```bash
[sol@Client-L ~]$ ls  -l  /temp/sol*
-rw-r--r-- 1 sol sol 0  7월 20 11:11 /temp/sol-a1
-rw-r--r-- 1 sol sol 0  7월 20 11:11 /temp/sol-a2
-rw-r--r-- 1 sol sol 0  7월 20 11:11 /temp/sol-a3
-rw-r--r-- 1 sol sol 0  7월 20 11:11 /temp/sol-b4
-rw-r--r-- 1 sol sol 0  7월 20 11:11 /temp/sol-b5
-rw-r--r-- 1 sol sol 0  7월 20 11:11 /temp/sol-b6
-rw-r--r-- 1 sol sol 0  7월 20 11:11 /temp/sol-c7
-rw-r--r-- 1 sol sol 0  7월 20 11:11 /temp/sol-c8
-rw-r--r-- 1 sol sol 0  7월 20 11:11 /temp/sol-c9
```

**EX1-3)** Client-L의 'sol' 계정을 사용하여 Server-A의 'guest'계정으로 SFTP접속

```bash
[sol@Client-L ~]$ sftp  guest@192.168.10.100				<---- 'guest'계정을 사용하여 Server-A로 SFTP 접속
The authenticity of host '192.168.10.100 (192.168.10.100)' can't be established.
ED25519 key fingerprint is SHA256:hSo7vPq/6MKH+FikOCDeY8V3GgrVvdPE8e+NHDAJV6g.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes	<---- yes
Warning: Permanently added '192.168.10.100' (ED25519) to the list of known hosts.
guest@192.168.10.100's password:
Connected to 192.168.10.100.
sftp>
sftp> pwd
Remote working directory: /home/guest				# 경로 확인
sftp>
sftp> ls -l							# 해당 디렉터리내의 파일 및 디렉터리 확인
-rw-r--r--    ? root     root       198676 Jul 10 12:13 ALL.tar.bz2
-rw-r--r--    ? guest    guest           58 Jul 16 16:18 cisco.txt
-rw-r--r--    ? root     root        27839 Jul 10 09:57 dnsmasq.conf
-rw-r--r--    ? guest    guest       23460 Jul 20 10:07 guest.txt
-rw-r--r--    ? root     root         5799 Jul 10 09:57 idmapd.conf
-rw-r--r--    ? root     root         9077 Jul 10 09:57 kdump.conf
-rw-r--r--    ? root     root        43431 Jul 10 09:57 ld.so.cache
-rw-r--r--    ? guest    guest          58 Jul 16 16:18 linux.txt
-rw-r--r--    ? root     root         7779 Jul 10 09:57 login.defs
-rw-r--r--    ? root     root         5235 Jul 10 09:57 man_db.conf
-rw-r--r--    ? root     root        67454 Jul 10 09:57 mime.types
-rw-r--r--    ? root     root        10373 Jul 10 09:57 nanorc
-rw-r--r--    ? root     root         6300 Jul 10 09:57 pnm2ppa.conf
-rw-r--r--    ? root     root         6568 Jul 10 09:57 protocols
-rw-r--r--    ? root     root       692252 Jul 10 09:57 services
-rw-r--r--    ? guest    guest           58 Jul 16 16:18 soldesk.txt
-rw-r--r--    ? guest    guest           58 Jul 16 16:18 system.txt
```

---

#### 파일 업로드

**EX2-1)** 아래의 조건에 맞게 Client-L의 'sol' 계정을 사용하여 Server-A에 파일을 업로드 해야한다.
  - 'sol' 사용자는 Client-L의 '/temp/' 디렉터리안에있는 'passwd' 파일을 Server-A의 'guest' 계정을 사용하여  'guest' 계정의 홈 디렉터리에 업로드해야한다.
  - 'sol' 사용자는 Client-L의 '/temp/' 디렉터리안에있는 'issue' 파일을 Server-A의 'guest' 계정을 사용하여  'guest' 계정의 홈 디렉터리에 업로드해야한다.
  - 'sol' 사용자는 Client-L의 '/temp/' 디렉터리안에있는 'sol-a1' ~ 'sol-a3' 파일을 Server-A의 'guest' 계정을 사용하여  'guest' 계정의 홈 디렉터리에 업로드해야한다.

sftp> put  passwd
stat passwd: No such file or direct	# sol 계정의 위치는 sol 계정의 홈 디렉터리이므로 passwd 파일이 검색되지 않는다.
  - 파일 업로드시 해당 파일이 있는 위치에서 sftp 접속해야 한다.

```bash
sftp> exit				# SFTP 종료
[sol@Client-L ~]$
[sol@Client-L ~]$ pwd		# 현재 sol 계정의 위치
/home/sol
```

- upload 조건
  - upload할 디렉터리나 윈도우는 업로드할 파일이 있는 디렉터리에서 SFTP로 접속해야한다.

- download 조건
  - download는 경로를 입력할 수 있기때문에 어떤 디렉터리에서 해도 관계가 없다.

```bash
[sol@Client-L ~]$ cd  /temp			# 디렉터리 이동
```

```bash
[sol@Client-L temp]$ pwd			# 현재 위치 확인
/temp
```

```bash
[sol@Client-L temp]$ ls -l  passwd
-rw-r--r-- 1 sol sol 2206  7월 20 11:10 passwd	# 업로드할 파일 확인
```

sftp> put  passwd				# passwd 파일을 SFTP를 사용하여 업로드
Uploading passwd to /home/guest/passwd
passwd                                        100% 2206   940.5KB/s   00:00

```
sftp> ls  -l  passwd				# sol 계정으로 파일 업로드 확인
-rw-r--r--    ? guest    guest        2206 Jul 20 11:37 passwd
```

```bash
[guest@Server-A ~]$ ls  -l  passwd		# guest 계정을 사용하여 파일 업로드 확인
-rw-r--r-- 1 guest guest 2206  7월 20 11:37 passwd
```

sftp> put  issue				# issue 파일을 SFTP를 사용하여 업로드
Uploading issue to /home/guest/issue
issue                                         100%   20     6.4KB/s   00:00

```
sftp> put  sol-a*
Uploading sol-a1 to /home/guest/sol-a1
sol-a1                                        100%    0     0.0KB/s   00:00
Uploading sol-a2 to /home/guest/sol-a2
sol-a2                                        100%    0     0.0KB/s   00:00
Uploading sol-a3 to /home/guest/sol-a3
sol-a3                                        100%    0     0.0KB/s   00:00
```

```
sftp> ls  -l
-rw-r--r--    ? root     root       198676 Jul 10 12:13 ALL.tar.bz2
-rw-r--r--    ? guest    guest          58 Jul 16 16:18 cisco.txt
-rw-r--r--    ? root     root        27839 Jul 10 09:57 dnsmasq.conf
-rw-r--r--    ? guest    guest       23460 Jul 20 10:07 guest.txt
-rw-r--r--    ? root     root         5799 Jul 10 09:57 idmapd.conf
-rw-r--r--    ? guest    guest          20 Jul 20 11:39 issue		# 파일 업로드 확인
-rw-r--r--    ? root     root         9077 Jul 10 09:57 kdump.conf
-rw-r--r--    ? root     root        43431 Jul 10 09:57 ld.so.cache
-rw-r--r--    ? guest    guest          58 Jul 16 16:18 linux.txt
-rw-r--r--    ? root     root         7779 Jul 10 09:57 login.defs
-rw-r--r--    ? root     root         5235 Jul 10 09:57 man_db.conf
-rw-r--r--    ? root     root        67454 Jul 10 09:57 mime.types
-rw-r--r--    ? root     root        10373 Jul 10 09:57 nanorc
-rw-r--r--    ? guest    guest        2206 Jul 20 11:37 passwd
-rw-r--r--    ? root     root         6300 Jul 10 09:57 pnm2ppa.conf
-rw-r--r--    ? root     root         6568 Jul 10 09:57 protocols
-rw-r--r--    ? root     root       692252 Jul 10 09:57 services
-rw-r--r--    ? guest    guest           0 Jul 20 11:40 sol-a1		# 파일 업로드 확인
-rw-r--r--    ? guest    guest           0 Jul 20 11:40 sol-a2		# 파일 업로드 확인
-rw-r--r--    ? guest    guest           0 Jul 20 11:40 sol-a3		# 파일 업로드 확인
-rw-r--r--    ? guest    guest          58 Jul 16 16:18 soldesk.txt
-rw-r--r--    ? guest    guest          58 Jul 16 16:18 system.txt
```

```bash
EX2-2) 'guest' 계정 사용자는 '/home/guest' 디렉터리에 linux-A , cisco-A 디렉터리를 생성해야한다.

[guest@Server-A ~]$ mkdir  ./linux-A
[guest@Server-A ~]$ mkdir  ./cisco-A
```

```bash
[guest@Server-A ~]$ ls  -ld  ./*-A
drwxr-xr-x 2 guest guest 6  7월 20 11:43 ./cisco-A
drwxr-xr-x 2 guest guest 6  7월 20 11:43 ./linux-A
```

```bash
[guest@Server-A ~]$ ls  -l  ./ | grep ^d
drwxr-xr-x 2 guest guest      6  7월 20 11:43 cisco-A
drwxr-xr-x 2 guest guest      6  7월 20 11:43 linux-A
```

**EX2-3)** 아래의 조건에 맞게 파일을 전송하시오
 .Client-L의 'sol' 사용자는 'sol-c7' ~ 'sol-c9' 파일을 Server-A의 '/home/guest/linux-A'  디렉터리로 복사해야한다.
 .Client-L의 'sol' 사용자는 locale.conf , localtime , login.defs , logrotate.conf 파일을  'linux-A'  디렉터리로 복사해야한다.
 .Client-L의 'sol' 사용자는 'log'가 포함된 모든 파일을  Server-A의 '/home/guest/cisco-A'  디렉터리로 복사해야한다.

#### 방법 1 (업로드할 디렉터리로 이동)

sftp> cd  /home/guest/linux-A				# 업로드할 디렉터리로 이동
sftp>
sftp> pwd						# 경로 확인
Remote working directory: /home/guest/linux-A

```
sftp> put  sol-c*
Uploading sol-c7 to /home/guest/linux-A/sol-c7
sol-c7                                        100%    0     0.0KB/s   00:00
Uploading sol-c8 to /home/guest/linux-A/sol-c8
sol-c8                                        100%    0     0.0KB/s   00:00
Uploading sol-c9 to /home/guest/linux-A/sol-c9
sol-c9                                        100%    0     0.0KB/s   00:00
```

```
sftp> ls  -l
-rw-r--r--    ? guest    guest           0 Jul 20 11:48 sol-c7
-rw-r--r--    ? guest    guest           0 Jul 20 11:48 sol-c8
-rw-r--r--    ? guest    guest           0 Jul 20 11:48 sol-c9
```

#### 방법 2 (업로드할 디렉터리를 설정)

sftp> cd ..				# guest 계정의 홈 디렉터리로 이동
sftp>
sftp>
sftp> pwd
Remote working directory: /home/guest		# guest 계정의 홈 디렉터리로 이동 확인

```
sftp> put  lo*  ./linux-A/
Uploading locale.conf to /home/guest/./linux-A/locale.conf
locale.conf                                   100%   19     9.5KB/s   00:00
Uploading localtime to /home/guest/./linux-A/localtime
localtime                                     100%  617   766.9KB/s   00:00
Uploading login.defs to /home/guest/./linux-A/login.defs
login.defs                                    100% 7779     6.9MB/s   00:00
Uploading logrotate.conf to /home/guest/./linux-A/logrotate.conf
logrotate.conf                                100%  496   694.0KB/s   00:00
```

```
sftp> ls  -l  ./linux-A/lo*
-rw-r--r--    ? guest    guest           19 Jul 20 11:50 ./linux-A/locale.conf
-rw-r--r--    ? guest    guest         617 Jul 20 11:50 ./linux-A/localtime
-rw-r--r--    ? guest    guest        7779 Jul 20 11:50 ./linux-A/login.defs
-rw-r--r--    ? guest    guest         496 Jul 20 11:50 ./linux-A/logrotate.conf
```

```
sftp> put  *log*  ./cisco-A
Uploading csh.login to /home/guest/./cisco-A/csh.login
csh.login                                     100% 1112   677.4KB/s   00:00
Uploading login.defs to /home/guest/./cisco-A/login.defs
login.defs                                    100% 7779     7.8MB/s   00:00
Uploading logrotate.conf to /home/guest/./cisco-A/logrotate.conf
logrotate.conf                                100%  496   897.9KB/s   00:00
Uploading rsyslog.conf to /home/guest/./cisco-A/rsyslog.conf
rsyslog.conf                                  100% 3300     2.6MB/s   00:00
```

---

```bash
			# 파일 다운로드

EX3-1) 아래의 조건에 맞게 파일을 전송하시오
 # Server-A에 관리자 권한으로 '/temp' 디렉터리안의 모든 파일을 삭제해야한다.
 # Server-A의 'guest' 계정을 사용하여 '/etc' 디렉터리의 모든 파일을 '/temp' 디렉터리로 복사해야한다.

[guest@Server-A ~]$ su -			# 관리자 권한으로 변경
암호:
```

```bash
[root@Server-A ~]# rm -rf  /temp/*		# /temp 디렉터리내의 모든 파일 및 디렉터리 삭제
```

```bash
[root@Server-A ~]# exit			# guest 계정으로 변경
로그아웃
```

```bash
[guest@Server-A ~]$ cp  /etc/*  /temp		# 파일 복사
```

```bash
[guest@Server-A ~]$ ls  -l  /temp		# 파일 복사 확인
```

**EX3-2)** 아래의 조건에 맞게 파일을 전송하시오
  - Client-L의 'sol' 사용자는 Server-A의 '/temp/' 디렉터리안의 파일중 'host'로 시작하는 모든 파일을 다운로드 해야한다.
  - Client-L의 'sol' 사용자는 Server-A의 '/temp/' 디렉터리안의 파일중 '.conf'로 끝나는 모든 파일을 다운로드 해야한다.
  - Client-L의 'sol' 사용자는 Server-A의 '/temp/' 디렉터리안의 파일중 'so'가 포함된 모든 파일을 다운로드 해야한다.

```bash
[sol@Client-L temp]$ cd ~		# 홈 디렉터리로 이동
[sol@Client-L ~]$
```

```bash
[sol@Client-L ~]$ ls  -l
```

```bash
[sol@Client-L ~]$ sftp  guest@192.168.10.100
guest@192.168.10.100's password:
Connected to 192.168.10.100.
sftp>
sftp>
sftp> get  /temp/host*		# 파일 다운로드
Fetching /temp/host.conf to host.conf
host.conf                            	100%    9     6.5KB/s   00:00
Fetching /temp/hostname to hostname
hostname                                      	100%    9     7.8KB/s   00:00
Fetching /temp/hosts to hosts
hosts                                           	100%  158   162.1KB/s   00:00
```

```bash
[sol@Client-L ~]$ ls  -l		# sol 계정 (2번째 접속으로 확인)
합계 12
-rw-r--r-- 1 sol sol   9  7월 20 12:01 host.conf
-rw-r--r-- 1 sol sol   9  7월 20 12:01 hostname
-rw-r--r-- 1 sol sol 158  7월 20 12:01 hosts
```

```
sftp> get  /temp/*.conf
Fetching /temp/anthy-unicode.conf to anthy-unicode.conf
anthy-unicode.conf                            100%  269   111.8KB/s   00:00
Fetching /temp/appstream.conf to appstream.conf
appstream.conf                                100%  833   585.7KB/s   00:00
Fetching /temp/asound.conf to asound.conf
asound.conf                                   100%   55    46.9KB/s   00:00
Fetching /temp/brltty.conf to brltty.conf
brltty.conf                                   100%   28KB   9.4MB/s   00:00
	~~~~~~~~~~ 중간 생략 ~~~~~~~~~~/s   00:00
Fetching /temp/usb_modeswitch.conf to usb_modeswitch.conf
usb_modeswitch.conf                           100% 1523     1.3MB/s   00:00
Fetching /temp/vconsole.conf to vconsole.conf
vconsole.conf                                 100%   28    26.4KB/s   00:00
Fetching /temp/xattr.conf to xattr.conf
xattr.conf                                    100%  817   643.0KB/s   00:00
Fetching /temp/yum.conf to yum.conf
yum.conf                                      100%  108    32.3KB/s   00:00
```

```bash
[sol@Client-L ~]$ ls  -l  ./*.conf
-rw-r--r-- 1 sol sol   269  7월 20 12:03 ./anthy-unicode.conf
-rw-r--r-- 1 sol sol   833  7월 20 12:03 ./appstream.conf
-rw-r--r-- 1 sol sol    55  7월 20 12:03 ./asound.conf
-rw-r--r-- 1 sol sol 28974  7월 20 12:03 ./brltty.conf
-rw-r--r-- 1 sol sol  1370  7월 20 12:03 ./chrony.conf
-rw-r--r-- 1 sol sol 27839  7월 20 12:03 ./dnsmasq.conf
	~~~~~~~~~~ 중간 생략 ~~~~~~~~~~
-rw-r--r-- 1 sol sol   449  7월 20 12:03 ./sysctl.conf
-rw-r--r-- 1 sol sol   624  7월 20 12:03 ./updatedb.conf
-rw-r--r-- 1 sol sol  1523  7월 20 12:03 ./usb_modeswitch.conf
-rw-r--r-- 1 sol sol    28  7월 20 12:03 ./vconsole.conf
-rw-r--r-- 1 sol sol   817  7월 20 12:03 ./xattr.conf
-rw-r--r-- 1 sol sol   108  7월 20 12:03 ./yum.conf
```

sftp> get  *so*
Fetching /home/guest/ld.so.cache to ld.so.cache
ld.so.cache                                   100%   42KB  12.8MB/s   00:00
Fetching /home/guest/sol-a1 to sol-a1
Fetching /home/guest/sol-a2 to sol-a2
Fetching /home/guest/sol-a3 to sol-a3
Fetching /home/guest/soldesk.txt to soldesk.txt
soldesk.txt                                   100%   58    41.0KB/s   00:00

```bash
[sol@Client-L ~]$ ls  -l  *so*
-rw-r--r-- 1 sol sol    55  7월 20 12:03 asound.conf
-rw-r--r-- 1 sol sol 43431  7월 20 12:08 ld.so.cache
-rw-r--r-- 1 sol sol    28  7월 20 12:03 ld.so.conf
-rw-r--r-- 1 sol sol    54  7월 20 12:03 resolv.conf
-rw-r--r-- 1 sol sol     0  7월 20 12:08 sol-a1
-rw-r--r-- 1 sol sol     0  7월 20 12:08 sol-a2
-rw-r--r-- 1 sol sol     0  7월 20 12:08 sol-a3
-rw-r--r-- 1 sol sol    58  7월 20 12:08 soldesk.txt
-rw-r--r-- 1 sol sol    28  7월 20 12:03 vconsole.conf
```

**EX3-3)** sol 계정을 사용하여 홈디렉터리 안에 'solLinux-A' , 'solCisco-A' 디렉터리를 생성해야 한다.

```bash
[sol@Client-L ~]$ mkdir  ./solLinux-A
[sol@Client-L ~]$ mkdir  ./solCisco-A
```

```bash
[sol@Client-L ~]$ ls  -ld  ./*-A
drwxr-xr-x 2 sol sol 6  7월 20 12:10 ./solCisco-A
drwxr-xr-x 2 sol sol 6  7월 20 12:10 ./solLinux-A
```

**EX3-4)** 아래의 조건에 맞게 파일을 전송하시오
  - Client-L의 'sol' 사용자는 Server-A의 '/temp/' 디렉터리안의 파일중 'host'로 시작하는 모든 파일을 홈디렉터리 안의 'solLinux-A' 디렉터리로 다운로드 해야한다.
  - Client-L의 'sol' 사용자는 Server-A의 '/temp/' 디렉터리안의 파일중 ''so'가 포함된 모든 파일을 홈디렉터리 안의 'solCisco-A' 디렉터리로 다운로드 해야한다.

```
sftp> get  /temp/host*  ./solLinux-A/
Fetching /temp/host.conf to ./solLinux-A/host.conf
host.conf                                 	100%    9     8.6KB/s   00:00
Fetching /temp/hostname to ./solLinux-A/hostname
hostname                                  	100%    9     9.8KB/s   00:00
Fetching /temp/hosts to ./solLinux-A/hosts
hosts                                    	100%  158   290.7KB/s   00:00
```

```bash
[sol@Client-L ~]$ ls  -l  ./solLinux-A/
합계 12
-rw-r--r-- 1 sol sol   9   7월 20 12:15 host.conf
-rw-r--r-- 1 sol sol   9   7월 20 12:15 hostname
-rw-r--r-- 1 sol sol 158  7월 20 12:15 hosts
```

```
sftp> get  /temp/*so*  ./solCisco-A
Fetching /temp/asound.conf to ./solCisco-A/asound.conf
asound.conf                                   	100%   55    70.2KB/s   00:00
Fetching /temp/ld.so.cache to ./solCisco-A/ld.so.cache
ld.so.cache                                   	100%   43KB  25.0MB/s   00:00
Fetching /temp/ld.so.conf to ./solCisco-A/ld.so.conf
ld.so.conf                                    	100%   28    54.1KB/s   00:00
Fetching /temp/resolv.conf to ./solCisco-A/resolv.conf
resolv.conf                                   	100%   54    80.2KB/s   00:00
Fetching /temp/vconsole.conf to ./solCisco-A/vconsole.conf
vconsole.conf                                 	100%   28    43.1KB/s   00:00
```

```bash
[sol@Client-L ~]$ ls  -l  ./solCisco-A/
합계 60
-rw-r--r-- 1 sol sol     55  7월 20 12:16 asound.conf
-rw-r--r-- 1 sol sol 43759  7월 20 12:16 ld.so.cache
-rw-r--r-- 1 sol sol     28  7월 20 12:16 ld.so.conf
-rw-r--r-- 1 sol sol     54  7월 20 12:16 resolv.conf
-rw-r--r-- 1 sol sol     28  7월 20 12:16 vconsole.conf
```

---

**EX4-1)** 아래의 조건에 맞게 파일을 전송하시오
  - 윈도우 C드라이브에 'sol' 폴더를 생성하시오
  - cmd 창을 사용하여 SFTP 접속 = sftp [계정명]@A.B.C.D
  - 'sol'계정에서 'guest'계정으로 업로드한 모든 파일을 sol 폴더로 다운로드
  - 'guest'계정에서 'sol'계정으로 다운로드한 모든 파일을 abc 폴더로 다운로드

- C드라이브안에 sol 디렉터리 생성

#### CMD창 1

C:\Users\soldesk> cd  c:\sol	# sol 디렉터리로 이동

```bash
c:\sol> sftp  guest@192.168.10.100		# window에서 Server-A로 guest 계정을 사용해서 접속
The authenticity of host '192.168.10.100 (192.168.10.100)' can't be established.
ED25519 key fingerprint is SHA256:hSo7vPq/6MKH+FikOCDeY8V3GgrVvdPE8e+NHDAJV6g.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '192.168.10.100' (ED25519) to the list of known hosts.
guest@192.168.10.100's password:1234
Connected to 192.168.10.100.
sftp>
sftp> get  ./*	# 파일만 다운로드
sftp> get  -r  ./*	# 파일 및 디렉터리 다운로드
```

```bash
	# CMD창 2

C:\Users\soldesk> cd C:\abc

C:\Users\soldesk> sftp  sol@192.168.10.130	# window에서 Client-L로 sol 계정을 사용해서 접속
The authenticity of host '192.168.10.130 (192.168.10.130)' can't be established.
ED25519 key fingerprint is SHA256:c+mGSVaw7kvAIZTNRsJAEl+aHvzAYJ9/WXGFUzrTteo.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])?
Warning: Permanently added '192.168.10.130' (ED25519) to the list of known hosts.
sol@192.168.10.130's password:
Connected to 192.168.10.130.
sftp> 
sftp> get  ./*	# 파일만 다운로드
sftp> get  -r  ./*	# 파일 및 디렉터리 다운로드
```

**정리**: SFTP (10-4) 섹션에서 다룬 명령어와 실습 예제를 통해 관련 기능의 구성 방법과 동작 원리를 확인했다.

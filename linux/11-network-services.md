# Linux 11 — 네트워크 서비스: DHCP · DNS

## 목차

1. [DHCP Server (10-5)](#dhcp-server-10-5)
2. [DNS Caching Server (10-6)](#dns-caching-server-10-6)
3. [Master DNS Server (10-7)](#master-dns-server-10-7)
4. [Master DNS Server 옵션 (10-8)](#master-dns-server-옵션-10-8)

## DHCP Server (10-5)

### DHCP (Dynamic Host Configuration Protocol)

- PC , Server등의 통신장비가 Internet망을 통신하기위해서는 IP 주소 , Subnetmask , Gateway IP 주소 , DNS server주소등이 필요하며, 사내 네트워크에서는 DHCP로 이 정보를 자동 배포하고 DNS 서버로 도메인 이름과 IP 주소를 관리하는 경우가 많다.

- DHCP는 Client에게 네트워크 통신에 필요한 정보를 자동으로 할당하는 Protocol이다.

- DHCP Server가 Client에게 제공하는 주요 정보
  - IP 주소
  - Subnet Mask
  - Default Gateway
  - DNS Server 주소
  - 임대 시간(Lease Time)

- DHCP는 UDP 67번과 UDP 68번을 사용한다.
  - UDP 67 : DHCP Server가 요청을 수신
  - UDP 68 : DHCP Client가 응답을 수신

- DHCP 기본 동작 과정은 DORA라고 한다.
  - Discover	: Client가 DHCP Server를 찾기 위해 Broadcast 전송
  - Offer    	: DHCP Server가 IP 주소와 네트워크 정보를 제안
  - Request  	: Client가 제안받은 주소의 사용을 요청
  - ACK      	: DHCP Server가 주소 사용을 최종 승인

---

#### 실습 환경

- Server-A
  - OS		: Rocky Linux 9
  - Interface	: ens160
  - IP 주소		: 192.168.10.100/24
  - Gateway	: 192.168.10.2
  - 역할		: DHCP Server

- Client-L
  - OS		: Rocky Linux 9
  - Interface	: ens160
  - IP 설정		: DHCP 자동 할당
  - 역할		: DHCP Client

- Win-L
  - IP 설정	: DHCP 자동 할당
  - 역할		: DHCP Client

- VMware Network
  - VMnet8 NAT 사용
  - Network	: 192.168.10.0/24
  - Gateway	: 192.168.10.2
  - VMware DHCP Server는 중지
  - Server-A, Client-L, Win-L은 동일한 VMnet에 연결

---

```bash
	# Server-A

[root@Server-A ~]# ifconfig ens160
ens160: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.10.100  netmask 255.255.255.0  broadcast 192.168.10.255
        inet6 fe80::20c:29ff:fe66:2f9c  prefixlen 64  scopeid 0x20<link>
        ether 00:0c:29:66:2f:9c  txqueuelen 1000  (Ethernet)
        RX packets 25424  bytes 30077387 (28.6 MiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 12335  bytes 2594201 (2.4 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

```bash
	# Client-L

[guest@Client-L ~]$ ifconfig ens160
ens160: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.10.130  netmask 255.255.255.0  broadcast 192.168.10.255
        inet6 fe80::20c:29ff:fea1:c154  prefixlen 64  scopeid 0x20<link>
        ether 00:0c:29:a1:c1:54  txqueuelen 1000  (Ethernet)
        RX packets 26783  bytes 30533606 (29.1 MiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 12891  bytes 1668718 (1.5 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

#### Client-Win

C:\Users\aaa>ipconfig

Windows IP 구성

이더넷 어댑터 Ethernet0:

   연결별 DNS 접미사. . . . : localdomain
   링크-로컬 IPv6 주소 . . . . : fe80::bfa5:7c22:99e:70a7%6
   IPv4 주소 . . . . . . . . . : 192.168.10.131
   서브넷 마스크 . . . . . . . : 255.255.255.0
   기본 게이트웨이 . . . . . . : 192.168.10.2

```bash
[root@Server-A ~]# nmcli  device  status
DEVICE	TYPE   	   STATE          	CONNECTION
ens160	ethernet	   연결됨         	ens160
lo   	loopback	   연결됨 (외부)	lo
```

```bash
[root@Server-A ~]# nmcli  connection  show
NAME 	UUID                                  TYPE   	 DEVICE
ens160	5989e9d3-fb91-3508-90e8-2ecc49b5b5d3	 ethernet  ens160
lo  	d529963a-c68e-4faa-bbe3-8005b1f133bf	 loopback  lo
```

---

```bash
	# Server-A 에서 IP 주소 할당 방법 확인

[root@Server-A ~]# cat  /etc/NetworkManager/system-connections/ens160.nmconnection
[connection]
id=ens160
uuid=5989e9d3-fb91-3508-90e8-2ecc49b5b5d3
type=ethernet
autoconnect-priority=-999
interface-name=ens160
timestamp=1782996029

[ethernet]

[ipv4]
method=manual				# 수동으로 IP 주소할당
address1=192.168.10.100/24,192.168.10.2
dns=192.168.10.2

[ipv6]
addr-gen-mode=eui64
method=auto

[proxy]
```

```bash
	# Client-L 에서 IP 주소 할당 방법 확인

[guest@Client-L ~]$ sudo cat  /etc/NetworkManager/system-connections/ens160.nmconnection
[sudo] guest 암호:
[connection]
id=ens160
uuid=bfa310d2-926a-34ff-85bf-e78988916dad
type=ethernet
autoconnect-priority=-999
interface-name=ens160
timestamp=1783011504

[ethernet]

[ipv4]
method=auto				# DHCP로 IP 주소할당

[ipv6]
addr-gen-mode=eui64
method=auto

[proxy]
```

```
	# Client-Win 에서 IP 주소 할당 방법 확인

C:\Users\aaa> ipconfig /all

Windows IP 구성

   호스트 이름 . . . . . . . . : DESKTOP-FOUO854
   주 DNS 접미사 . . . . . . . :
   노드 유형 . . . . . . . . . : 혼성
   IP 라우팅 사용. . . . . . . : 아니요
   WINS 프록시 사용. . . . . . : 아니요
   DNS 접미사 검색 목록. . . . : localdomain

이더넷 어댑터 Ethernet0:

   연결별 DNS 접미사. . . . : localdomain
   설명. . . . . . . . . . . . : Intel(R) 82574L Gigabit Network Connection
   물리적 주소 . . . . . . . . : 00-0C-29-E6-35-23
   DHCP 사용 . . . . . . . . . : 예				<--- DHCP를 사용해서 IP 주소 할당
   자동 구성 사용. . . . . . . : 예
   링크-로컬 IPv6 주소 . . . . : fe80::bfa5:7c22:99e:70a7%6(기본 설정)
   IPv4 주소 . . . . . . . . . : 192.168.10.131(기본 설정)
   서브넷 마스크 . . . . . . . : 255.255.255.0
   임대 시작 날짜. . . . . . . : 2026년 7월 20일 월요일 오후 12:48:29
   임대 만료 날짜. . . . . . . : 2026년 7월 20일 월요일 오후 1:18:30
   기본 게이트웨이 . . . . . . : 192.168.10.2
   DHCP 서버 . . . . . . . . . : 192.168.10.254
   DHCPv6 IAID . . . . . . . . : 100666409
   DHCPv6 클라이언트 DUID. . . : 00-01-00-01-31-D7-DC-C0-00-0C-29-E6-35-23
   DNS 서버. . . . . . . . . . : 192.168.10.2
   주 WINS 서버. . . . . . . . : 192.168.10.2
   Tcpip를 통한 NetBIOS. . . . : 사용
```

---

**EX1)** VMware에서 제공하는 DHCP Server를 중지해야 한다.

- 한 네트워크에 VMware DHCP Server와 Server-A의 DHCP Server가 동시에 동작하면 ` Client가 어느 DHCP Server에서 주소를 받을지 예측할 수 없다.

- VMware Workstation 설정 경로

 Edit
  ---> Virtual Network Editor
       ---> Change Settings
            ---> VMnet8 선택
                 ---> Use local DHCP service to distribute IP address to VMs 체크 해제
                      ---> Apply

- VMware DHCP Server를 중지하더라도 VMnet8의 NAT 기능과 Gateway 주소는 계속 사용할 수 있다.

- DHCP Server를 아직 구성하지 않았기때문에 Client-L과 Win-L은 IPv4 주소를 받지 못한다.

```bash
	# Client-L에서 기존에 DHCP로부터 받은 IP 주소를 삭제

[guest@Client-L ~]$ sudo nmcli  connection  down ens160		# ens160 인터페이스를 down으로 전환하게되면 기존 ssh 접속은 해제된다.
```

```bash
[root@Client-L ~]# nmcli  connection  up  ens160		# GUI로 접속 후 설정
```

```bash
[root@Client-L ~]# ifconfig ens160				# IPv4 주소가 확인되지 않는다.
ens160: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet6 fe80::20c:29ff:fea1:c154  prefixlen 64  scopeid 0x20<link>
        ether 00:0c:29:a1:c1:54  txqueuelen 1000  (Ethernet)
        RX packets 27170  bytes 30568569 (29.1 MiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 13030  bytes 1687191 (1.6 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

#### Client-win에서 기존에 DHCP로부터 받은 IP 주소를 삭제

C:\Users\aaa> ipconfig  /release	# 기존에 DHCP로부터 받은 IP 주소를 삭제

Windows IP 구성

이더넷 어댑터 Ethernet0:

   연결별 DNS 접미사. . . . :
   링크-로컬 IPv6 주소 . . . . : fe80::bfa5:7c22:99e:70a7%6
   기본 게이트웨이 . . . . . . :

C:\Users\aaa>ipconfig  /renew	# 다시 IP 주소 요청 (DHCP Server가 없기때문에 IPv4 주소를 받을 수 없다.)

---

#### DHCP Package 설치

```bash
[root@Server-A ~]# dnf  install  -y  dhcp-server
```

```bash
[root@Server-A ~]# rpm  -qa  | grep dhcp
dhcp-common-4.4.2-20.b1.el9.rocky.0.1.noarch
dhcp-server-4.4.2-20.b1.el9.rocky.0.1.x86_64
```

```bash
[root@Server-A ~]# ls  -l  /etc/ | grep dhcp
drwxr-x---.  3 root root        61  7월 20 13:11 dhcp	# 설치 시간을 확인하게되면 현재시간으로 확인된다.
```

```bash
[root@Server-A ~]# ls  -l  /etc/dhcp
합계 8
drwxr-xr-x. 2 root root  23  7월  2 15:07 dhclient.d
-rw-r--r--  1 root root 123  1월  9  2026 dhcpd.conf
-rw-r--r--  1 root root 126  1월  9  2026 dhcpd6.conf
```

```bash
[root@Server-A ~]# cat  /etc/dhcp/dhcpd.conf		# 아직 DHCP관련 설정이 확인되지 않는다.
#
# DHCP Server Configuration file.
#   see /usr/share/doc/dhcp-server/dhcpd.conf.example
#   see dhcpd.conf(5) man page
#
```

#### 방화벽에서 DHCP를 허용해야 한다.

```bash
[root@Server-A ~]# firewall-cmd  --permanent  --add-service=dhcp
success
```

```bash
[root@Server-A ~]# firewall-cmd  --permanent  --add-port=67/udp
success

[root@Server-A ~]# firewall-cmd  --permanent  --add-port=68/udp
success
```

```bash
[root@Server-A ~]# firewall-cmd  --reload
success
```

```bash
[root@Server-A ~]# firewall-cmd  --list-port
20/tcp 21/tcp 22/tcp 111/tcp 139/tcp 445/tcp 2002/tcp 2049/tcp 67/udp 68/udp 111/udp 137/udp 138/udp
```

```bash
[root@Server-A ~]# firewall-cmd  --list-service
cockpit dhcp dhcpv6-client ftp nfs rpc-bind samba ssh
```

```bash
[root@Server-A ~]# firewall-cmd  --list-all
```

```bash
	# DHCP Server 설정

[root@Server-A ~]# vi  /etc/dhcp/dhcpd.conf
#
# DHCP Server Configuration file.
#   see /usr/share/doc/dhcp-server/dhcpd.conf.example
#   see dhcpd.conf(5) man page
#
[root@Server-A ~]# vi  /etc/dhcp/dhcpd.conf
#
authoritative;
ddns-update-style none;

subnet 192.168.10.0 netmask 255.255.255.0 {		# DHCP 서비스를 제공할 네트워크 대역

        option routers 192.168.10.2;           		# Client에게 할당할 Default Gateway 주소
        option domain-name-servers 168.126.63.1, 8.8.8.8;	# Client에게 할당할 DNS Server 주소
        option subnet-mask 255.255.255.0;     		# Client에게 할당할 Subnet Mask
        option broadcast-address 192.168.10.255; 		# Client에게 할당할 Broadcast 주소

        range 192.168.10.150 192.168.10.200;    		# Client에게 동적으로 할당할 IP 주소 범위

        default-lease-time 86400;             		# 기본 IP 임대 시간: 86400초 = 1일
        max-lease-time 864000;                		# 최대 IP 임대 시간: 864000초 = 10일
}

:wq
```

```bash
[root@Server-A ~]# systemctl  status  dhcpd	# DHCP Server 상태 확인
○ dhcpd.service - DHCPv4 Server Daemon
     Loaded: loaded (/usr/lib/systemd/system/dhcpd.service; disabled; preset: d>
     Active: inactive (dead)
       Docs: man:dhcpd(8)
             man:dhcpd.conf(5)
lines 1-5/5 (END)
```

```bash
[root@Server-A ~]# systemctl  start  dhcpd
```

```bash
[root@Server-A ~]# systemctl  enable  dhcpd
Created symlink /etc/systemd/system/multi-user.target.wants/dhcpd.service → /usr/lib/systemd/system/dhcpd.service.
```

```bash
[root@Server-A ~]# systemctl  status  dhcpd	# DHCP Server 상태 확인
● dhcpd.service - DHCPv4 Server Daemon
     Loaded: loaded (/usr/lib/systemd/system/dhcpd.service; enabled; preset: di>
     Active: active (running) since Mon 2026-07-20 14:51:29 KST; 20s ago
       Docs: man:dhcpd(8)
             man:dhcpd.conf(5)
   Main PID: 4279 (dhcpd)
     Status: "Dispatching packets..."
      Tasks: 1 (limit: 10321)
     Memory: 4.9M (peak: 5.4M)
        CPU: 16ms
     CGroup: /system.slice/dhcpd.service
             └─4279 /usr/sbin/dhcpd -f -cf /etc/dhcp/dhcpd.conf -user dhcpd -gr>

 7월 20 14:51:29 Server-A dhcpd[4279]: Wrote 0 leases to leases file.
 7월 20 14:51:29 Server-A dhcpd[4279]: Listening on LPF/ens160/00:0c:29:66:2f:9>
 7월 20 14:51:29 Server-A dhcpd[4279]: Sending on   LPF/ens160/00:0c:29:66:2f:9>
 7월 20 14:51:29 Server-A dhcpd[4279]: Sending on   Socket/fallback/fallback-net
 7월 20 14:51:29 Server-A dhcpd[4279]: Server starting service.
 7월 20 14:51:29 Server-A systemd[1]: Started DHCPv4 Server Daemon.
 7월 20 14:51:33 Server-A dhcpd[4279]: DHCPDISCOVER from 00:0c:29:a1:c1:54 via >
 7월 20 14:51:34 Server-A dhcpd[4279]: DHCPOFFER on 192.168.10.150 to 00:0c:29:>
 7월 20 14:51:34 Server-A dhcpd[4279]: DHCPREQUEST for 192.168.10.150 (192.168.>
 7월 20 14:51:34 Server-A dhcpd[4279]: DHCPACK on 192.168.10.150 to 00:0c:29:a1>
```

#### DHCP Client

- Client-L은 현재 IP 주소가 없기때문에 SSH 접속이 불가능하기때문에 콘솔로 접속해야 한다.

```bash
[root@Client-L ~]# ifconfig ens160
ens160: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.10.150  netmask 255.255.255.0  broadcast 192.168.10.255
        inet6 fe80::20c:29ff:fea1:c154  prefixlen 64  scopeid 0x20<link>
        ether 00:0c:29:a1:c1:54  txqueuelen 1000  (Ethernet)
        RX packets 138703  bytes 197726957 (188.5 MiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 37948  bytes 3191173 (3.0 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

```bash
login as: guest
guest@192.168.10.150's password:1234
Activate the web console with: systemctl enable --now cockpit.socket

Last login: Mon Jul 20 12:52:17 2026 from 192.168.10.1
[guest@Client-L ~]$
```

```bash
[guest@Client-L ~]$ sudo  rpm  -qa | grep dhcp		# dhcp-client package가 설치되지 않았다.
```

- Linux rocky9은 network-manager가 기본적인 dhcp-client기능을 갖고 있기때문에 따로 dhcp-client package를 설치하지 않아도된다.

C:\Users\aaa> ipconfig  /renew	# Windows환경에서 DHCP 주소를 새로 받는 명령어

Windows IP 구성

이더넷 어댑터 Ethernet0:

   연결별 DNS 접미사. . . . :
   링크-로컬 IPv6 주소 . . . . : fe80::bfa5:7c22:99e:70a7%6
   IPv4 주소 . . . . . . . . . : 192.168.10.151
   서브넷 마스크 . . . . . . . : 255.255.255.0
   기본 게이트웨이 . . . . . . : 192.168.10.2

```bash
		# DHCP Server에서 할당 IP 주소 확인

[root@Server-A ~]# cat  /var/lib/dhcpd/dhcpd.leases
# The format of this file is documented in the dhcpd.leases(5) manual page.
# This lease file was written by isc-dhcp-4.4.2b1

# authoring-byte-order entry is generated, DO NOT DELETE
authoring-byte-order little-endian;

server-duid "\000\001\000\0011\360s\341\000\014)f/\234";

lease 192.168.10.150 {
  starts 1 2026/07/20 05:51:34;
  ends 2 2026/07/21 05:51:34;
  cltt 1 2026/07/20 05:51:34;
  binding state active;
  next binding state free;
  rewind binding state free;
  hardware ethernet 00:0c:29:a1:c1:54;
  uid "\001\000\014)\241\301T";
  client-hostname "Client-L";
}
lease 192.168.10.151 {
  starts 1 2026/07/20 05:54:26;
  ends 2 2026/07/21 05:54:26;
  cltt 1 2026/07/20 05:54:26;
  binding state active;
  next binding state free;
  rewind binding state free;
  hardware ethernet 00:0c:29:e6:35:23;
  uid "\001\000\014)\3465#";
  set vendor-class-identifier = "MSFT 5.0";
  client-hostname "DESKTOP-FOUO854";
}
```

C:\Users\aaa> hostname		# Client-win에서 확인
DESKTOP-FOUO854

---

**정리**: DHCP Server (10-5) 섹션에서 다룬 명령어와 실습 예제를 통해 관련 기능의 구성 방법과 동작 원리를 확인했다.

## DNS Caching Server (10-6)

### Master Name Server

- Master Name Server는 해당 도메인에 대한 권한을 가진(Authoritative) DNS 서버 중 하나이며, Zone File(영역 파일)을 직접 관리한다.
  즉, example.com 도메인의 IP 주소 매핑 정보(A, MX, NS, CNAME 등)를 직접 보유하며, 
  DNS 질의가 들어오면 해당 도메인에 대한 공식적인 응답 정보를 제공한다.

- 다른 서버(Secondary Name Server)는 이 Master 서버로부터 데이터를 복제(Zone Transfer)받는다.

- 주요 특징
  - 권한		: 해당 도메인에 대한 최종 권한(Authoritative) 을 가짐
  - 데이터 저장 위치	: /var/named/example.com.zone 같은 zone 파일에 직접 저장
  - 데이터 변경	: 관리자가 직접 수정 가능
  - 동기화		: Secondary 서버에 데이터를 전송
  - 역할		: 도메인에 대한 공식 응답을 제공하는 원본 DNS

- 동작 과정
 1) 내부 사용자가 www.soldesk.local에 접속
 2) 로컬 DNS가 Master Name Server에 질의
 3) Master 서버는 zone 파일을 참조해 192.168.10.2 응답
 4) 로컬 DNS는 해당 결과를 캐싱 후 사용자에게 반환
 5) Secondary 서버가 있을 경우, Master 서버로부터 zone 데이터를 복제해 유지

#### Master vs Secondary vs Caching 서버 비교

- --

구분		| Master Name Server	|  Secondary Name Server	| Caching Name Server	|

- --

역할		| 원본 DNS 데이터 관리	| Master에서 데이터 복제	| 외부 질의 결과를 캐시	|

- --

zone 파일 보유	| 있음 (직접 수정 가능)	| 있음 (복제본)		| 없음			|

- --

데이터 변경	| 수동 또는 자동으로 수정	| Master로부터만 복사	| 없음			|

- --

용도		| 내부 도메인 운영		| 부하 분산, 백업		| 클라이언트 성능 향상	|

- --

---

- Server-A (192.168.10.100)
  - Web Server

- Server-B (192.168.10.200)
  - FTP Serve

- www.soldesk.com	 ---->  Web Server
- ftp.soldesk.com	 ---->  FTP Server

#### Server-A를 HTTP Server로 구성

- httpd package를 설치 (가장 많이 사용되는 아파치 웹서버)

```bash
[root@Server-A ~]# rpm  -qa | grep httpd	# httpd 설치 확인
```

```bash
[root@Server-A ~]# dnf  install  -y  httpd	# httpd 설치 
```

```bash
[root@Server-A ~]# systemctl  start  httpd	# httpd 서버 실행
```

```bash
[root@Server-A ~]# systemctl  enable  httpd	# Server-A 재부팅시 httpd를 자동 실행
Created symlink /etc/systemd/system/multi-user.target.wants/httpd.service → /usr/lib/systemd/system/httpd.service.
```

```bash
[root@Server-A ~]# systemctl  status  httpd
● httpd.service - The Apache HTTP Server
     Loaded: loaded (/usr/lib/systemd/system/httpd.service; enabled; preset: di>
     Active: active (running) since Tue 2026-07-21 09:50:18 KST; 40s ago
       Docs: man:httpd.service(8)
   Main PID: 2997 (httpd)
     Status: "Total requests: 0; Idle/Busy workers 100/0;Requests/sec: 0; Bytes>
      Tasks: 177 (limit: 10320)
     Memory: 14.0M (peak: 14.4M)
        CPU: 103ms
     CGroup: /system.slice/httpd.service
             ├─2997 /usr/sbin/httpd -DFOREGROUND
             ├─2998 /usr/sbin/httpd -DFOREGROUND
             ├─2999 /usr/sbin/httpd -DFOREGROUND
             ├─3000 /usr/sbin/httpd -DFOREGROUND
             └─3001 /usr/sbin/httpd -DFOREGROUND

 7월 21 09:50:18 Server-A systemd[1]: Starting The Apache HTTP Server...
 7월 21 09:50:18 Server-A httpd[2997]: AH00558: httpd: Could not reliably deter>
 7월 21 09:50:18 Server-A httpd[2997]: Server configured, listening on: port 80
 7월 21 09:50:18 Server-A systemd[1]: Started The Apache HTTP Server.
lines 1-20/20 (END)
```

```bash
[root@Server-A ~]# firewall-cmd  --permanent  --add-service=http
success
```

```bash
[root@Server-A ~]# firewall-cmd  --permanent  --add-port=80/tcp
success
```

```bash
[root@Server-A ~]# firewall-cmd  --reload
success
```

```bash
[root@Server-A ~]# firewall-cmd  --list-port
20/tcp 21/tcp 22/tcp 53/tcp 80/tcp 111/tcp 139/tcp 445/tcp 2002/tcp 2049/tcp 53/udp 67/udp 68/udp 111/udp 137/udp 138/udp
```

```bash
[root@Server-A ~]# firewall-cmd  --list-service
cockpit dhcp dhcpv6-client dns ftp http nfs rpc-bind samba ssh
```

```bash
[root@Server-A ~]# vi  /var/www/html/index.html
<style>
        h1 {
                text-align: center;
        }
</style>
```

<h1>Soldesk IT Academy</h1>

<p>안녕하세요 솔데스크 학원입니다.</p>
<hr>
<h3>개설 과정 안내</h3>

<ul>
        <li>AWS 데브옵스 과정</li>
        <li>네트워크 전문가 과정</li>
        <li>AI 데이터 분석 과정</li>
        <li>풀스텍 개발 과정</li>
</ul>

- Server-A에서 파이어 폭스를 사용하여 http://192.168.10.100으로 접속 
- Client-L에서 파이어 폭스를 사용하여 http://192.168.10.100으로 접속 

```bash
	# Server-B를 FTP Server로 구성

[root@localhost ~]# hostnamectl  set-hostname  Server-B

[root@Server-B ~]# cat  /etc/hostname
Server-B
```

```bash
[root@Server-B ~]# dnf  install  -y  net-tools  bind-utils  traceroute		# 네트워크 관련 Package 설치
```

```bash
[root@Server-B ~]# ifconfig ens160
ens160: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.10.200  netmask 255.255.255.0  broadcast 192.168.10.255
        inet6 fe80::20c:29ff:fe40:fe0c  prefixlen 64  scopeid 0x20<link>
        ether 00:0c:29:40:fe:0c  txqueuelen 1000  (Ethernet)
        RX packets 20187  bytes 29543668 (28.1 MiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 5778  bytes 363447 (354.9 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

```bash
[root@Server-B ~]# ip addr show
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: ens160: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:0c:29:40:fe:0c brd ff:ff:ff:ff:ff:ff
    altname enp3s0
    inet 192.168.10.200/24 brd 192.168.10.255 scope global noprefixroute ens160
       valid_lft forever preferred_lft forever
    inet6 fe80::20c:29ff:fe40:fe0c/64 scope link noprefixroute
       valid_lft forever preferred_lft forever
```

```bash
[root@Server-B ~]# dnf  install  -y  vsftpd				# vsFTP 설치
```

```bash
[root@Server-B ~]# systemctl  start  vsftpd
```

```bash
[root@Server-B ~]# systemctl  enable  vsftpd
Created symlink /etc/systemd/system/multi-user.target.wants/vsftpd.service → /usr/lib/systemd/system/vsftpd.service.
```

```bash
[root@Server-B ~]# systemctl  status  vsftpd
● vsftpd.service - Vsftpd ftp daemon
     Loaded: loaded (/usr/lib/systemd/system/vsftpd.service; enabled; preset: d>
     Active: active (running) since Tue 2026-07-21 10:14:25 KST; 9s ago
   Main PID: 2128 (vsftpd)
      Tasks: 1 (limit: 10453)
     Memory: 864.0K (peak: 1.3M)
        CPU: 3ms
     CGroup: /system.slice/vsftpd.service
             └─2128 /usr/sbin/vsftpd /etc/vsftpd/vsftpd.conf

 7월 21 10:14:25 Server-B systemd[1]: Starting Vsftpd ftp daemon...
 7월 21 10:14:25 Server-B systemd[1]: Started Vsftpd ftp daemon.
```

```bash
[root@Server-B ~]# rpm -qa | grep firewall
firewalld-filesystem-1.3.4-18.el9_7.noarch
python3-firewall-1.3.4-18.el9_7.noarch
firewalld-1.3.4-18.el9_7.noarch
```

```bash
[root@Server-B ~]# firewall-cmd  --permanent  --add-service=ftp
success
```

```bash
[root@Server-B ~]# firewall-cmd  --permanent  --add-port=20/tcp
success
```

```bash
[root@Server-B ~]# firewall-cmd  --permanent  --add-port=21/tcp
success
```

```bash
[root@Server-B ~]# firewall-cmd  --reload
success
```

```bash
[root@Server-B ~]# firewall-cmd  --list-port
20/tcp 21/tcp
```

```bash
[root@Server-B ~]# firewall-cmd  --list-service
]cockpit dhcpv6-client ftp ssh
```

```bash
[root@Server-B ~]# firewall-cmd  --list-all
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: ens160
  sources:
  services: cockpit dhcpv6-client ftp ssh
  ports: 20/tcp 21/tcp
  protocols:
  forward: yes
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:
```

```bash
[root@Server-B ~]# ls -l /var/ftp
합계 0
drwxr-xr-x 2 root root 6  1월 17  2026 pub	# FTP 공용 디렉터리 (익명 사용자 허용시 pub에만 접근이 가능하다.)
```

```bash
	# banner 파일 생성

[root@Server-B ~]# vi  /var/ftp/welcome.msg

---

	Welcome to Soldesk FTP Server

---

이 서버는 솔데스크 아이티 아카데미에서 운영하는 FTP 서버입니다.
공개 디렉터리는 /pub 디렉터리만 사용 가능합니다.
업로드 문의는 관리자 승인후 가능합니다.
문의 : admin@soldesk.co.kr

:wq
```

```bash
	# 설정한 banner 파일 적용

[root@Server-B ~]# vi  /etc/vsftpd/vsftpd.conf
       ~~~~~~~~~ 중간 생략 ~~~~~~~~~
    121 # files.
    122 # Make sure, that one of the listen options is commented !!
    123 listen_ipv6=YES
    124
    125 pam_service_name=vsftpd
    126 userlist_enable=YES
    127 banner_file=/var/ftp/welcome.msg	# 설정한 banner 적용

:wq
```

- 익명 사용자(anonymous) 관련 설정
                    항목			기본값	설명
  - anonymous_enable=YES		NO	익명 사용자 접속 허용
  - no_anon_password=YES		NO	익명 로그인 시 비밀번호 입력 없이 접속 허용
  - anon_root=/var/ftp		-	익명 사용자의 홈 디렉터리 지정
  - anon_upload_enable=YES		NO	익명 사용자의 파일 업로드 허용
  - anon_mkdir_write_enable=YES	NO	익명 사용자의 디렉터리 생성 허용
  - anon_other_write_enable=YES	NO	익명 사용자의 삭제/이름변경 허용
  - anon_world_readable_only=YES	YES	익명 사용자는 읽기 전용 디렉터리만 접근 가능

- 일반 사용자(local user) 관련 설정
                    항목		기본값	설명
  - local_enable=YES	YES	시스템 계정(일반 사용자) FTP 접속 허용
  - write_enable=YES	NO	파일 쓰기(업로드/삭제/수정) 허용

```bash
[root@Server-B ~]# systemctl  restart  vsftpd
```

```bash
	# FTP 접속을 허용하기위해서 Server-B에서 guest 계정 생성
[root@Server-B ~]# useradd guest
[root@Server-B ~]# passwd guest
guest 사용자의 비밀 번호 변경 중
새 암호:
잘못된 암호: 암호는 8 개의 문자 보다 짧습니다
새 암호 재입력:
passwd: 모든 인증 토큰이 성공적으로 업데이트 되었습니다.
```

```bash
	# FTP 접속을 위해서 Client-L에서 FTP 설치

[guest@Client-L ~]$ sudo  dnf  install  -y  ftp
```

```bash
[guest@Client-L ~]$ ftp 192.168.10.200
Connected to 192.168.10.200 (192.168.10.200).
220-===========================================================
220-    Welcome to Soldesk FTP Server
220-===========================================================
220-이 서버는 솔데스크 아이티 아카데미에서 운영하는 FTP 서버입니다.
220-공개 디렉터리는 /pub 디렉터리만 사용 가능합니다.
220-업로드 문의는 관리자 승인후 가능합니다.
220-문의 : admin@soldesk.co.kr
220
Name (192.168.10.200:guest): guest
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> 
ftp> pwd
257 "/home/guest" is the current directory
```

---

### Server-A를 Master Name Server로 구성

- Master Name Server
  - 특정 도메인의 Zone File(영역 파일)을 직접 생성하고 관리하는 DNS 서버
  - 해당 도메인의 A, NS, MX, CNAME 등의 DNS 레코드를 직접 보유한다.
  - 클라이언트가 해당 도메인에 대해 질의하면 Zone File을 기준으로 응답한다.
  - 다른 DNS 서버가 이 서버의 Zone 정보를 복제하도록 구성할 수도 있다.
  - 현재 BIND에서는 Master/Slave 대신 Primary/Secondary라는 용어도 사용한다. 

- named.conf
  - BIND DNS 서버의 전체 동작 방식을 설정하는 기본 설정 파일
  - BIND의 서비스 프로세스인 named가 시작될 때 읽어 들인다.
  - 어떤 도메인을 관리할 것인지, Zone File이 어디에 있는지, 어떤 IP 주소에서 DNS 요청을 받을지 등을 설정한다.
  - DNS 질의를 허용할 대상, 재귀 질의 허용 여부, 캐싱 및 보안 정책도 설정한다.
  - 기본 설정 파일의 경로는 /etc/named.conf이다. 

- Zone
  - DNS 서버가 관리하는 도메인 영역을 의미한다.
  - 예를 들어 soldesk.com Zone을 등록하면 다음과 같은 이름을 관리할 수 있다. 
      www.soldesk.com 
      mail.soldesk.com 
      ftp.soldesk.com ns.soldesk.com

- Zone File 
  - 특정 도메인의 실제 DNS 레코드를 저장하는 파일
  - A, NS, MX, CNAME, PTR 등의 레코드를 작성한다.
  - 일반적으로 /var/named 디렉터리에 저장된다.
  - named.conf의 file 항목에는 Zone File의 파일명을 지정한다.

```bash
[root@Server-A ~]# vi  /etc/named.conf
~~~~~~~~~ 중간 생략 ~~~~~~~~~
     51
     52 zone "." IN {
     53         type hint;
     54         file "named.ca";
     55 };
     56
     57 include "/etc/named.rfc1912.zones";
     58 include "/etc/named.root.key";
     59
     60 zone "soldesk.com" IN {
     61         type master;
     62         file "soldesk.com.db";
     63 };

:wq
```

```bash
[root@Server-A ~]# named-checkconf		# named.conf 에러 체크
```

---

#### Zone 파일이란?

zone 파일 = DNS 서버가 직접 관리하는 주소록 파일

예: www.example.com 	--> 	192.168.10.100
예: mail.example.com 	--> 	192.168.10.200

- 도메인 이름을 IP 주소로 바꾸는 데이터가 들어 있는 파일이다.

- Domain(도메인)	: 인터넷에서 쓰는 이름 전체 구조
- Zone(존)		: 그 도메인 중에서 해당 DNS 서버가 직접 관리하는 부분만 잘라낸 것

- 즉 도메인 전체 중에서 우리 DNS 서버가 맡은 부분 = Zone
- 그 관리 데이터가 들어있는 파일 = Zone 파일

- DNS 서버 설정은 두 군데에서 이루어진다

/etc/named.conf	: 어떤 도메인을 관리할지 선언하는 곳 (zone 파일 위치도 여기서 지정)

EX)
zone "example.com" {
    type master;
    file "example.com.zone";
};

이 설정은 example.com 도메인의 데이터는 example.com.zone 이라는 파일에 있다.

/var/named/	: 실제 zone 파일이 저장되는 디렉터리

EX)
/var/named/example.com.zone

- Zone의 구성 요소 (Zone은 보통 다음 파일 구조로 구성)
  - /etc/named.conf		: DNS 서버의 전체 설정 파일 (여기서 zone 정의)
  - /var/named/도메인명.db	: 실제 zone 데이터(레코드) 저장 파일

- zone 파일은 기본적으로 '/var/named/' 디렉터리 안에 생성해야한다.
 이유는 '/etc/named.conf' 파일에 zone 파일의 기본 경로가 '/var/named/'로 설정되어있다.

Zone 파일의 내용 예시 (soldesk.com.db)
$TTL 1D
@   IN SOA  ns1.soldesk.com. admin.soldesk.com. (
              	2025110201 ; Serial 번호
              	1H 	; Refresh
              	10M    	; Retry
              	1W     	; Expire
              	1D 	; Minimum TTL

    	IN NS  ns1.soldesk.com.		# soldesk.com의 DNS 서버를 ns1.soldesk.com으로 지정
ns1 	IN A   192.168.1.1			# ns1.soldesk.com의 IPv4 주소를 192.168.1.1로 지정
www 	IN A   192.168.1.1			# www.soldesk.com의 IPv4 주소를 192.168.1.1로 지정
mail 	IN A   192.168.1.2			# mail.soldesk.com의 IPv4 주소를 192.168.1.2로 지정

    구분		이름		설명
프로그램 이름	BIND		DNS 서버 소프트웨어의 이름 (패키지 이름)
데몬(서비스 이름)	named		실제로 실행되는 DNS 서버 프로세스 이름
데이터 디렉터리	/var/named/	BIND(named)가 사용하는 zone 파일 저장 위치

```bash
[root@Server-A ~]# ls  -l  /var/named
합계 16
drwxrwx--- 2 named named    44  7월 20 17:40 data
drwxrwx--- 2 named named    60  7월 21 09:37 dynamic
-rw-r----- 1 root     named 2112  6월 10 12:48 named.ca
-rw-r----- 1 root     named  152  6월 10 12:48 named.empty
-rw-r----- 1 root     named  152  6월 10 12:48 named.localhost
-rw-r----- 1 root     named  168  6월 10 12:48 named.loopback
drwxrwx--- 2 named named     6  6월 10 12:58 slaves
```

```bash
[root@Server-A named]# touch soldesk.com.db 		# '/etc/named.conf'에서 설정한 파일명을 생성 (vi로 생성 가능)
```

```bash
[root@Server-A ~]# ls  -l  /var/named
합계 16
drwxrwx--- 2 named named    44  7월 20 17:40 data
drwxrwx--- 2 named named    60  7월 21 09:37 dynamic
-rw-r----- 1 root     named 2112  6월 10 12:48 named.ca
-rw-r----- 1 root     named  152  6월 10 12:48 named.empty
-rw-r----- 1 root     named  152  6월 10 12:48 named.localhost
-rw-r----- 1 root     named  168  6월 10 12:48 named.loopback
drwxrwx--- 2 named named     6  6월 10 12:58 slaves
-rw-r--r-- 1 root  root           0  7월 21 11:18 soldesk.com.db	# 파일 생성 확인
```

```bash
[root@Server-A ~]# cp  /var/named/named.empty  /var/named/soldesk.com.db		# 예시파일인 named.empty를 soldesk.com.db로 복사
cp: overwrite '/var/named/soldesk.com.db'? y
```

```bash
[root@Server-A ~]# cat  /var/named/soldesk.com.db
$TTL 3H
@       IN SOA  @ rname.invalid. (
                                        0       	; serial
                                        1D      	; refresh
                                        1H      	; retry
                                        1W      	; expire
                                        3H )	; minimum
        NS      @
        A       127.0.0.1
        AAAA    ::1
```

```bash
[root@Server-A ~]# vi  /var/named/soldesk.com.db
$TTL 1D
@       IN SOA  ns.soldesk.com. admin.naver.com. (
                                        20260721	; serial
                                        1D      	; refresh
                                        1H      	; retry
                                        1W      	; expire
                                        3H )    	; minimum
@       	IN      NS      ns.soldesk.com.		# soldesk.com의 DNS 서버 이름
@        	IN      A       192.168.10.100		# soldesk.com의 기본 IP 주소 설정

ns      	IN      A       192.168.10.100		# ns.soldesk.com의 IP 주소
www	IN      A       192.168.10.100		# www.soldesk.com의 IP 주소
ftp     	IN      A       192.168.10.200		# ftp.soldesk.com의 IP 주소

:wq
```

- named-checkzone
  - named-checkzone 명령은 특정 도메인(zone) 에 대한 zone 데이터 파일의 문법과 구조가 올바른지 검사하는 도구
    즉,named.conf 설정이 아니라, 실제 zone 파일(soldesk.com.db) 내부의 A, NS, SOA 등의 구문이 규칙에 맞게 작성되어 있는지를 확인
  - 형식 : named-checkzone <도메인명> <zone파일경로>

```bash
[root@Server-A ~]# named-checkzone  soldesk.com  /var/named/soldesk.com.db
zone soldesk.com/IN: loaded serial 20260721
OK
```

```bash
[root@Server-A ~]# chown  root:named  /var/named/soldesk.com.db
[root@Server-A ~]# chmod  640   /var/named/soldesk.com.db
```

```bash
[root@Server-A ~]# ls  -l  /var/named/
합계 24
drwxrwx--- 2 named  named   44  7월 20 17:40 data
drwxrwx--- 2 named  named   60  7월 21 12:53 dynamic
-rw-r----- 1  root    named 2112  6월 10 12:48 named.ca
-rw-r----- 1  root    named  152  6월 10 12:48 named.empty
-rw-r----- 1  root    named  152  6월 10 12:48 named.localhost
-rw-r----- 1  root    named  168  6월 10 12:48 named.loopback
drwxrwx--- 2 named named    6  6월 10 12:58 slaves
-rw-r----- 1  root    named  271  7월 21 11:45 soldesk.com.db	# other에게는 권한을 주지 않고 같은 그룹만 읽을수 있게 수정
```

```bash
[root@Server-A ~]# systemctl  restart named
```

```bash
[root@Server-A ~]# systemctl  status named
● named.service - Berkeley Internet Name Domain (DNS)
     Loaded: loaded (/usr/lib/systemd/system/named.service; enabled; preset: di>
     Active: active (running) since Tue 2026-07-21 11:53:33 KST; 15s ago
    Process: 5217 ExecStartPre=/bin/bash -c if [ ! "$DISABLE_ZONE_CHECKING" == >
    Process: 5219 ExecStart=/usr/sbin/named -u named -c ${NAMEDCONF} $OPTIONS (>
   Main PID: 5220 (named)
      Tasks: 8 (limit: 10320)
     Memory: 22.2M (peak: 22.6M)
        CPU: 47ms
     CGroup: /system.slice/named.service
             └─5220 /usr/sbin/named -u named -c /etc/named.conf

 7월 21 11:53:33 Server-A named[5220]: network unreachable resolving './NS/IN':>
 7월 21 11:53:33 Server-A named[5220]: network unreachable resolving './DNSKEY/>
 7월 21 11:53:33 Server-A named[5220]: network unreachable resolving './NS/IN':>
 7월 21 11:53:33 Server-A named[5220]: network unreachable resolving './DNSKEY/>
 7월 21 11:53:33 Server-A named[5220]: network unreachable resolving './NS/IN':>
 7월 21 11:53:33 Server-A named[5220]: network unreachable resolving './DNSKEY/>
 7월 21 11:53:33 Server-A named[5220]: network unreachable resolving './NS/IN':>
 7월 21 11:53:33 Server-A named[5220]: resolver priming query complete
 7월 21 11:53:33 Server-A named[5220]: managed-keys-zone: Key 20326 for zone . >
 7월 21 11:53:33 Server-A named[5220]: managed-keys-zone: Key 38696 for zone . >
lines 1-22/22 (END)
```

- Client-L에서 http://www.soldesk.com으로 접속하게되면 실제 솔데스크 학원의 홈페이지로 접속된다.
- 이유 : 현재 /etc/resolv.conf 파일에는 실제 DNS Server의 IP 가 설정되어있기때문에 실제 홈페이지로 이동된다.

```bash
[guest@Client-L ~]$ cat /etc/resolv.conf
# Generated by NetworkManager
search localdomain
nameserver 192.168.10.2		# 실제 DNS Server IP 주소
```

```bash
[guest@Client-L ~]$ vi /etc/resolv.conf
# Generated by NetworkManager
search localdomain
#nameserver 192.168.10.2		# 주석 처리
nameserver 192.168.10.100		# DNS Server의 IP 주소를 Server-A의 IP 주소로 수정

:wq
```

- Client-L에서 http://www.soldesk.com으로 접속하게되면 Server-A의 Web Server로 접속된다.

```bash
[guest@Client-L ~]$ nslookup
>
> server
Default server: 192.168.10.100
Address: 192.168.10.100#53		# 현재 DNS Server IPv4 주소
>
> www.soldesk.com
Server:	192.168.10.100
Address:	192.168.10.100#53

Name:   www.soldesk.com
Address: 192.168.10.100		# www.soldesk.com 도메인의 IPv4 주소
>
>
> ftp.soldesk.com
Server:	192.168.10.100
Address:	192.168.10.100#53

Name:   ftp.soldesk.com
Address: 192.168.10.200		# ftp.soldesk.com 도메인의 IPv4 주소
>
```

```bash
-이전에는 브라우저에서 FTP 접속이 가능했지만 현재는 거의 대부분의 브라우저에서 FTP 기능을 지원하지 않는다.

[guest@Client-L ~]$ ftp  ftp.soldesk.com
Connected to ftp.soldesk.com (192.168.10.200).
220-===========================================================
220-    Welcome to Soldesk FTP Server
220-===========================================================
220-이 서버는 솔데스크 아이티 아카데미에서 운영하는 FTP 서버입니다.
220-공개 디렉터리는 /pub 디렉터리만 사용 가능합니다.
220-업로드 문의는 관리자 승인후 가능합니다.
220-문의 : admin@soldesk.co.kr
220
Name (ftp.soldesk.com:guest): guest
331 Please specify the password.
Password:1234
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp>
```

```bash
-리눅스는 브라우저가 없기때문에 명령어를 사용해서 확인해야 한다.

[guest@Client-L ~]$ curl  192.168.10.100

[guest@Client-L ~]$ curl  http://www.soldesk.com
```

---

#### 역방향 DNS

- DNS (정방향)
  - Domain에 대해서 IP 주소로 변환하는 기능
  - EX) www.soldesk.com  --->  192.168.10.100

- DNS (역방향)
  - IP 주소를 사용하여 Domain주소로 변환하는 기능
  - EX) 192.168.10.100  --->  www.soldesk.com

#### 역방향 DNS를 설정해야 하는 이유

- 역방향 DNS는 특히 메일 서버의 신뢰성을 확인할 때 중요하게 사용된다.
  - 메일을 수신하는 SMTP 서버는 메일을 전송한 서버가 정상적으로 등록된 서버인지 확인하기 위해 역방향 DNS 정보를 검사할 수 있다.
  - 역방향 DNS가 설정되어 있지 않거나 DNS 정보가 서로 일치하지 않으면 해당 메일이 스팸으로 분류되거나 수신이 거부될 수 있다.

- SMTP 서버는 다음을 검사
  - SMTP(Simple Mail Transfer Protocol)는 이메일을 전송할 때 사용하는 표준 프로토콜
  - 1) 메일을 전송한 서버의 IP 주소에 PTR 레코드가 등록되어 있는가?
  - 2) PTR 레코드로 확인된 도메인 이름을 정방향 조회했을 때 원래 메일 서버의 IP 주소가 다시 조회되는가?
  - 이처럼 역방향 조회 결과를 다시 정방향으로 조회하여 원래 IP 주소와 일치하는지 확인하는 방식을 FCrDNS(Forward-confirmed Reverse DNS)라고 한다

- 정방향: mail.soldesk.com  --->  192.168.111.50
- 역방향: 192.168.10.50  --->  mail.soldesk.com

- 만약 역방향 DNS가 없으면 정체불명의 서버로 판단되어  스팸 점수 증가하고 메일이 스팸함으로 이동하거나 아예 수신 거부될수 있다.

- SSH / 웹 / 시스템 로그 분석이 쉬워진다.

- 로그를 보면 대부분 IP만 찍힌다.

EX) Failed password for 192.168.10.200
EX) Connection from 192.168.10.150 accepted

- IP만 가지고는 어떤 서버인지 파악하기 어렵다.
- 하지만 역방향 DNS가 있으면:

192.168.10.200  --->  ftp.soldesk.com
192.168.10.150  --->  Client-L.soldesk.com

- 이름이 보이기 때문에 누가 접속했고 어떤 장비인지 한눈에 알 수 있다.

- 보안 이벤트 분석, 장애 대응 속도가 매우 빨라진다.

```bash
[root@Server-A ~]# nslookup  dns.google
Server:	      192.168.10.2
Address:	      192.168.10.2#53

Non-authoritative answer:
Name:   dns.google
Address: 8.8.8.8
Name:   dns.google
Address: 8.8.4.4
Name:   dns.google
Address: 2001:4860:4860::8888
Name:   dns.google
Address: 2001:4860:4860::8844
```

```bash
[root@Server-A ~]# nslookup  8.8.8.8
8.8.8.8.in-addr.arpa    name = dns.google.

Authoritative answers can be found from:
```

```bash
[root@Server-A ~]# nslookup  168.126.63.1
1.63.126.168.in-addr.arpa       name = kns.kornet.net.

Authoritative answers can be found from:
```

```bash
[root@Server-A ~]# vi  /etc/named.conf
~~~~~~~~~ 중간 생략 ~~~~~~~~~

include "/etc/named.rfc1912.zones";
include "/etc/named.root.key";

zone "soldesk.com" IN {
        type master;
        file "soldesk.com.db";
};

zone "10.168.192.in-addr.arpa" IN {	# 설정
        type master;			# 설정
        file "soldesk.com.rev";		# 설정
};				# 설정
```

```bash
[root@Server-A ~]# cp  /var/named/named.empty  /var/named/soldesk.com.rev
```

```bash
[root@Server-A ~]# ls -l /var/named
합계 24
drwxrwx--- 2 named named    44  7월 20 17:40 data
drwxrwx--- 2 named named    60  7월 21 11:54 dynamic
-rw-r----- 1 root     named 2112  6월 10 12:48 named.ca
-rw-r----- 1 root     named  152  6월 10 12:48 named.empty
-rw-r----- 1 root     named  152  6월 10 12:48 named.localhost
-rw-r----- 1 root     named  168  6월 10 12:48 named.loopback
drwxrwx--- 2 named named     6  6월 10 12:58 slaves
-rw-r--r-- 1 root     root     271  7월 21 11:45 soldesk.com.db
-rw-r----- 1 root     root     152  7월 21 12:38 soldesk.com.rev
```

```bash
[root@Server-A ~]# vi  /var/named/soldesk.com.rev
$TTL 1D
@       IN SOA  ns.soldesk.com. root.soldesk.com. (
                                        2026072101      ; serial
                                        1D              ; refresh
                                        1H              ; retry
                                        1W              ; expire
                                        3H )            ; minimum
@        	IN      	NS      	ns.soldesk.com.
100     	IN      	PTR     	www.soldesk.com.
200	IN      	PTR	ftp.soldesk.com.

:wq
```

- PTR (Pointer) 레코드
  - A 레코드는 Domian요청에 대해서 IP 주소로 변환하는 기능
  - PTR 레코드는 A 레코드와 반대로 IP 주소에 대해서 domain으로 변환하는 기능

```bash
[root@Server-A ~]# named-checkzone 10.168.192.in-addr.arpa /var/named/soldesk.com.rev
zone 10.168.192.in-addr.arpa/IN: loaded serial 2026072101
OK
```

```bash
[root@Server-A ~]# systemctl  restart  named
```

```bash
[root@Server-A ~]# chown  root:named  /var/named/soldesk.com.rev
[root@Server-A ~]# chmod  640   /var/named/soldesk.com.rev
```

```bash
[root@Server-A ~]# ls  -l  /var/named/
합계 24
drwxrwx--- 2 named  named   44  7월 20 17:40 data
drwxrwx--- 2 named  named   60  7월 21 12:53 dynamic
-rw-r----- 1  root    named 2112  6월 10 12:48 named.ca
-rw-r----- 1  root    named  152  6월 10 12:48 named.empty
-rw-r----- 1  root    named  152  6월 10 12:48 named.localhost
-rw-r----- 1  root    named  168  6월 10 12:48 named.loopback
drwxrwx--- 2 named named    6  6월 10 12:58 slaves
-rw-r----- 1  root    named  271  7월 21 11:45 soldesk.com.db
-rw-r----- 1  root    named  233  7월 21 12:43 soldesk.com.rev	# other에게는 권한을 주지 않고 같은 그룹만 읽을수 있게 수정
```

```bash
[root@Server-A ~]# systemctl restart  named
```

```bash
	# Server-A에서 자신의 DNS에게 요청 (동작하는지 테스트)
[root@Server-A ~]# nslookup  192.168.10.100  127.0.0.1
100.10.168.192.in-addr.arpa     name = www.soldesk.com.
```

```bash
	# Server-A에서 자신의 DNS에게 요청 (동작하는지 테스트)
[root@Server-A ~]# nslookup  192.168.10.200  127.0.0.1
200.10.168.192.in-addr.arpa     name = ftp.soldesk.com.
```

```bash
	# 실제 사용자인 Client-L에서 확인
[guest@Client-L ~]$ nslookup  192.168.10.100
100.10.168.192.in-addr.arpa     name = www.soldesk.com.
```

```bash
	# 실제 사용자인 Client-L에서 확인
[guest@Client-L ~]$ nslookup  192.168.10.200
200.10.168.192.in-addr.arpa     name = ftp.soldesk.com.
```

---

- 실무에서 사용하는 구조(named.conf + named.rfc1912.zones + zone 파일 분리) 를 적용한다.

- 실습 환경 구성
구분	호스트명	IP 주소	역할
Server-A	192.168.10.100	DNS Master / HTTP Web Server	
Server-B	192.168.10.200	FTP Server	
Client-L	192.168.10.150	DNS Client (테스트용)	

```bash
               # Server-A를 HTTP Server로 구성

[root@Server-A ~]# rpm  -qa  | grep http	<---- HTTP관련 Package가 설치되지 않음
```

```bash
[root@Server-A ~]# dnf -y install httpd	<---- HTTP관련 Package 설치
```

```bash
[root@Server-A ~]# systemctl start httpd	<---- HTTP Server 실행
```

```bash
[root@Server-A ~]# systemctl enable httpd	<---- Server-A가 재부팅되어도 자동 실행
Created symlink from /etc/systemd/system/multi-user.target.wants/httpd.service to /usr/lib/systemd/system/httpd.service.
```

```bash
[root@Server-A ~]# systemctl status httpd	<---- 서비스 상태 확인
● httpd.service - The Apache HTTP Server
     Loaded: loaded (/usr/lib/systemd/system/httpd.service; enabled; vendor pre>
     Active: active (running) since Thu 2025-10-30 23:09:34 KST; 12min ago
       Docs: man:httpd.service(8)
   Main PID: 2372 (httpd)
     Status: "Total requests: 0; Idle/Busy workers 100/0;Requests/sec: 0; Bytes>
      Tasks: 213 (limit: 22611)
     Memory: 45.0M
        CPU: 698ms
     CGroup: /system.slice/httpd.service
             ├─2372 /usr/sbin/httpd -DFOREGROUND
             ├─2373 /usr/sbin/httpd -DFOREGROUND
             ├─2374 /usr/sbin/httpd -DFOREGROUND
             ├─2375 /usr/sbin/httpd -DFOREGROUND
             └─2376 /usr/sbin/httpd -DFOREGROUND
```

```bash
[root@Server-A ~]# firewall-cmd  --permanent  --add-port=80/tcp
success

[root@Server-A ~]# firewall-cmd  --permanent  --add-service=http
success

[root@Server-A ~]# firewall-cmd  --reload
success
```

```bash
[root@Server-A ~]# firewall-cmd  --list-all
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: ens33
  sources:
  services: dhcpv6-client dns http ssh
  ports: 53/tcp 53/udp 80/tcp
  protocols:
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:
```

```bash
[root@Server-A ~]# vi  /var/www/html/index.html
<style>
        h1 {
            text-align: center;
        }
</style>

<h1>Soldesk IT Academy</h1>
<p>안녕하세요. 솔데스크 학원입니다.</p>
<hr>
<h3>개설 과정 안내</h3>
<ul>
        <li>네트워크 보안 전문가 과정</li>
        <li>클라우드 엔지니어 과정</li>
        <li>AI · 데이터 분석 과정</li>
        <li>웹 개발자 양성 과정</li>
</ul>

:wq
```

```bash
[root@Server-A ~]# cat  /var/www/html/index.html
<h1> Global IT Academy </h1>
안녕하세요 글로벌 아이티 인재개발원입니다.
```

http://192.168.10.100	# Server-A에서 파이어 폭스로 확인
curl 192.168.10.100		# Server-A에서 명령어로 확인

http://192.168.10.100	# Client-L에서 파이어 폭스로 확인
curl 192.168.10.100		# Client-L에서 명령어로 확인

#### Server-B를 FTP Server로 구성

```bash
[root@localhost ~]# hostnamectl  set-hostname  Server-B

[root@Server-B ~]# cat /etc/hostname		<--- 해당 경로의 파일에 저장된다.
Server-B
```

```bash
[root@Server-B ~]# ifconfig		<---- 네트워크 장치 및 IP 주소가 확인되지 않는다. (Server-B는 최소 설치이므로 네트워크 툴을 별도로 설치해야한다.)
-bash: ifconfig: command not found
```

```bash
[root@Server-B ~]# dnf  -y  install  net-tools	<---- 네트워크 관련 Package 설치

[root@Server-B ~]# dnf  -y  install  vsftpd	<---- FTP Package 설치
```

```bash
[root@Server-B ~]# systemctl status vsftpd
○ vsftpd.service - Vsftpd ftp daemon
     Loaded: loaded (/usr/lib/systemd/system/vsftpd.service; disabled; vendor p>
     Active: inactive (dead)			<---- 서버가 동작하지 않는다.
```

```bash
[root@Server-B ~]# systemctl  start  vsftpd

[root@Server-B ~]# systemctl  enable  vsftpd
Created symlink /etc/systemd/system/multi-user.target.wants/vsftpd.service → /usr/lib/systemd/system/vsftpd.service.
```

```bash
[root@Server-B ~]# firewall-cmd  --permanent  --add-port=20/tcp
success

[root@Server-B ~]# firewall-cmd  --permanent  --add-port=21/tcp
success

[root@Server-B ~]# firewall-cmd  --permanent  --add-service=ftp
success

[root@Server-B ~]# firewall-cmd  --reload
success
```

```bash
[root@Server-B ~]# firewall-cmd  --list-all
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: ens32
  sources:
  services: dhcpv6-client ftp ssh
  ports: 20/tcp 21/tcp
  protocols:
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:
```

```bash
[root@Server-B ~]# cd /var/ftp        		# FTP 기본 경로로 이동
```

```bash
[root@Server-B ftp]# ls -l
total 0
drwxr-xr-x 2 root root 6 Nov  7  2024 pub   	# 외부 사용자가 접근 가능한 공개용 디렉터리 (기본 제공)
```

```bash
[root@Server-B ftp]# vi welcome.msg
```

- --

Welcome to Soldesk FTP Server

- --

이 서버는 솔데스크 학원에서 운영하는 FTP 서버입니다.
공개 디렉터리는 /pub 폴더를 이용해 주세요.
업로드는 관리자 승인 후 가능합니다.
문의: admin@soldesk.com

:wq

```bash
[root@Server-B ftp]# vi /etc/vsftpd/vsftpd.conf
     banner_file=/var/ftp/welcome.msg		<--- 첫줄에 설정
       ~~~~~~~~~ 중간 생략 ~~~~~~~~~
     34 # Activate directory messages - messages given to remote users when they
     35 # go into a certain directory.
     36 dirmessage_enable=YES
     37 #
     38 # Activate logging of uploads/downloads.
     39 xferlog_enable=YES
     40 #
       ~~~~~~~~~ 중간 생략 ~~~~~~~~~
```

- 익명 사용자(anonymous) 관련 설정
                    항목			기본값	설명
  - anonymous_enable=YES		NO	익명 사용자 접속 허용
  - no_anon_password=YES		NO	익명 로그인 시 비밀번호 입력 없이 접속 허용
  - anon_root=/var/ftp		-	익명 사용자의 홈 디렉터리 지정
  - anon_upload_enable=YES		NO	익명 사용자의 파일 업로드 허용
  - anon_mkdir_write_enable=YES	NO	익명 사용자의 디렉터리 생성 허용
  - anon_other_write_enable=YES	NO	익명 사용자의 삭제/이름변경 허용
  - anon_world_readable_only=YES	YES	익명 사용자는 읽기 전용 디렉터리만 접근 가능

- 일반 사용자(local user) 관련 설정
                    항목		기본값	설명
  - local_enable=YES	YES	시스템 계정(일반 사용자) FTP 접속 허용
  - write_enable=YES	NO	파일 쓰기(업로드/삭제/수정) 허용

```bash
[root@Server-B ftp]# systemctl restart vsftpd
```

#### Client-L에서 Server-B로 FTP 접속

```bash
[root@Client-L ~]# dnf  -y  install  ftp		# Clinet-L에서 FTP 접속을 위해서 FTP Package 설치
```

```bash
[root@Client-L ~]# ftp  192.168.10.200
Connected to 192.168.10.200 (192.168.10.200).
220-========================================
220-Welcome to Soldesk FTP Server
220-========================================
220-이 서버는 솔데스크 학원에서 운영하는 FTP 서버입니다.
220-공개 디렉터리는 /pub 폴더를 이용해 주세요.
220-업로드는 관리자 승인 후 가능합니다.
220-문의: admin@soldesk.com
220-
220-
220-
220
Name (192.168.10.200:root): guest	<---- 'guest' 계정으로 접속
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp>
ftp> pwd
257 "/home/guest"
ftp>
```

```bash
[root@Server-A ~]# vi /etc/named.conf
  1 //
  2 // named.conf
  3 //
  4 // Provided by Red Hat bind package to configure the ISC BIND named(8) DNS
  5 // server as a caching only nameserver (as a localhost DNS resolver only).
  6 //
  7 // See /usr/share/doc/bind*/sample/ for example named configuration files.
  8 //
  9
 10 options {
 11         listen-on port 53 { any; };		<---- 127.0.0.1 = 자신의 DNS만 처리 | any = 모든 DNS를 처리
 12         listen-on-v6 port 53 { none; };		<---- IPv6 사용 X
 13         directory       "/var/named";
 14         dump-file       "/var/named/data/cache_dump.db";
 15         statistics-file "/var/named/data/named_stats.txt";
 16         memstatistics-file "/var/named/data/named_mem_stats.txt";
 17         secroots-file   "/var/named/data/named.secroots";
 18         recursing-file  "/var/named/data/named.recursing";
 19         allow-query     { any; };		<---- Localhost = 자신의 DNS 요청만 처리 | any = 모든 DNS 요청을 처리
 20
 21         /*
 22          - If you are building an AUTHORITATIVE DNS server, do NOT enable recursion.
 23          - If you are building a RECURSIVE (caching) DNS server, you need to enable
 24            recursion.
 25          - If your recursive DNS server has a public IP address, you MUST enable access
 26            control to limit queries to your legitimate users. Failing to do so will
 27            cause your server to become part of large scale DNS amplification
 28            attacks. Implementing BCP38 within your network would greatly
 29            reduce such attack surface
 30         */
 31         recursion yes;
 32
 33         dnssec-validation no;		<---- 실습용 로컬 네임서버에서는 DNSSEC 검증이 필요 없으므로 yes를 no로 설정
```

- /etc/named.conf에 아래의 설정이 있어야한다. (기본으로 설정되어 있다.)

options {
        directory       "/var/named";
        recursion       yes;
        allow-query     { any; };
};

zone "." IN {
        type hint;
        file "named.ca";
};

include "/etc/named.rfc1912.zones";
include "/etc/named.root.key";

#### 정방향 DNS 등록

- soldesk.com 도메인에 대한 정방향 해석 (도메인 --> IP) 을 처리하는 영역
- type master는 이 서버가 원본(zone data)을 직접 관리함을 의미한다.

```bash
[root@Server-A ~]# vi /etc/named.rfc1912.zones
          ~~~~~~ 중간 생략 ~~~~~~
zone "soldesk.com" IN {
        type master;
        file "soldesk.com.dns";
};

:wq
```

```bash
[root@Server-A ~]# cd /var/named
[root@Server-A named]# vi soldesk.com.dns
$TTL 1D
@   IN 	SOA 	ns.soldesk.com. 	admin.soldesk.com. (
        		2025110201 ; serial
        		1D         ; refresh
        		1H         ; retry
        		1W         ; expire
        		3H )       ; minimum
   	 IN NS   ns.soldesk.com.
   	 IN A    192.168.10.100

ns  	IN A    192.168.10.100
www 	IN A    192.168.10.100
ftp 	IN A    192.168.10.200

:wq
```

```bash
[root@Server-A ~]# named-checkzone soldesk.com /var/named/soldesk.com.dns
zone soldesk.com/IN: loaded serial 2025110201
OK
```

```bash
	# 역방향 DNS 등록

-IP 주소  -->  도메인 이름으로 해석할 수 있게 해준다.
-in-addr.arpa 도메인은 역방향 네임스페이스 전용으로 사용

[root@Server-A ~]# vi /etc/named.rfc1912.zones
          ~~~~~~ 중간 생략 ~~~~~~
zone "soldesk.com" IN {
        type master;
        file "soldesk.com.dns";
};

zone "10.168.192.in-addr.arpa" IN {	<--- 추가
        type master;			<--- 추가
        file "soldesk.com.rev";		<--- 추가
};				<--- 추가

:wq
```

```bash
[root@Server-A named]# vi soldesk.com.rev
$TTL 1D
@   IN 	SOA 	ns.soldesk.com. 	root.soldesk.com. (
        		2025110201 ; serial
        		1D         ; refresh
        		1H         ; retry
        		1W         ; expire
       		3H )       ; minimum
    	IN NS   ns.soldesk.com.

100 	IN PTR www.soldesk.com.
200 	IN PTR ftp.soldesk.com.
:wq
```

```bash
[root@Server-A ~]# tail -10 /etc/named.rfc1912.zones
zone "soldesk.com" IN {
        type master;
        file "soldesk.com.dns";
};

zone "10.168.192.in-addr.arpa" IN {
        type master;
        file "soldesk.com.rev";
};
```

```bash
[root@Server-A ~]# named-checkzone 10.168.192.in-addr.arpa /var/named/soldesk.com.rev
zone 10.168.192.in-addr.arpa/IN: loaded serial 2025110201
OK
```

```bash
[root@Server-A ~]# systemctl restart named
[root@Server-A ~]# systemctl enable named
[root@Server-A ~]# systemctl status named
```

```bash
[root@Client-L ~]# ftp ftp.soldesk.com		<---- Clinet-L Server-B로 Domain을 사용하여 FTP 접속
Connected to ftp.soldesk.com (192.168.10.200).
220-========================================
220-Welcome to Soldesk FTP Server
220-========================================
220-이 서버는 솔데스크 학원에서 운영하는 FTP 서버입니다.
220-공개 디렉터리는 /pub 폴더를 이용해 주세요.
220-업로드는 관리자 승인 후 가능합니다.
220-문의: admin@soldesk.com
220-
220-
220-
220
Name (ftp.soldesk.com:root): guest
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp>
ftp>
ftp> pwd
257 "/home/guest" is the current directory
ftp>
```

#### Client-L에서 브라우저를 사용하여 웹서버 접속

http://www.soldesk.com/

Soldesk IT Academy

안녕하세요. 솔데스크 학원입니다.
개설 과정 안내

    네트워크 보안 전문가 과정
    클라우드 엔지니어 과정
    AI  데이터 분석 과정
    웹 개발자 양성 과정

#### Client-Win에서 브라우저를 사용하여 웹서버 접속이 가능하다.

---

**정리**: DNS Caching Server (10-6) 섹션에서 다룬 명령어와 실습 예제를 통해 관련 기능의 구성 방법과 동작 원리를 확인했다.

## Master DNS Server (10-7)

### Master Name Server

- Master Name Server는 해당 도메인에 대한 권한을 가진(Authoritative) DNS 서버 중 하나이며, Zone File(영역 파일)을 직접 관리한다.
  즉, example.com 도메인의 IP 주소 매핑 정보(A, MX, NS, CNAME 등)를 직접 보유하며, 
  DNS 질의가 들어오면 해당 도메인에 대한 공식적인 응답 정보를 제공한다.

- 다른 서버(Secondary Name Server)는 이 Master 서버로부터 데이터를 복제(Zone Transfer)받는다.

- 주요 특징
  - 권한		: 해당 도메인에 대한 최종 권한(Authoritative) 을 가짐
  - 데이터 저장 위치	: /var/named/example.com.zone 같은 zone 파일에 직접 저장
  - 데이터 변경	: 관리자가 직접 수정 가능
  - 동기화		: Secondary 서버에 데이터를 전송
  - 역할		: 도메인에 대한 공식 응답을 제공하는 원본 DNS

- 동작 과정
 1) 내부 사용자가 www.soldesk.local에 접속
 2) 로컬 DNS가 Master Name Server에 질의
 3) Master 서버는 zone 파일을 참조해 192.168.10.2 응답
 4) 로컬 DNS는 해당 결과를 캐싱 후 사용자에게 반환
 5) Secondary 서버가 있을 경우, Master 서버로부터 zone 데이터를 복제해 유지

#### Master vs Secondary vs Caching 서버 비교

- --

구분		| Master Name Server	|  Secondary Name Server	| Caching Name Server	|

- --

역할		| 원본 DNS 데이터 관리	| Master에서 데이터 복제	| 외부 질의 결과를 캐시	|

- --

zone 파일 보유	| 있음 (직접 수정 가능)	| 있음 (복제본)		| 없음			|

- --

데이터 변경	| 수동 또는 자동으로 수정	| Master로부터만 복사	| 없음			|

- --

용도		| 내부 도메인 운영		| 부하 분산, 백업		| 클라이언트 성능 향상	|

- --

---

- Server-A (192.168.10.100)
  - Web Server

- Server-B (192.168.10.200)
  - FTP Serve

- www.soldesk.com	 ---->  Web Server
- ftp.soldesk.com	 ---->  FTP Server

#### Server-A를 HTTP Server로 구성

- httpd package를 설치 (가장 많이 사용되는 아파치 웹서버)

```bash
[root@Server-A ~]# rpm  -qa | grep httpd	# httpd 설치 확인
```

```bash
[root@Server-A ~]# dnf  install  -y  httpd	# httpd 설치 
```

```bash
[root@Server-A ~]# systemctl  start  httpd	# httpd 서버 실행
```

```bash
[root@Server-A ~]# systemctl  enable  httpd	# Server-A 재부팅시 httpd를 자동 실행
Created symlink /etc/systemd/system/multi-user.target.wants/httpd.service → /usr/lib/systemd/system/httpd.service.
```

```bash
[root@Server-A ~]# systemctl  status  httpd
● httpd.service - The Apache HTTP Server
     Loaded: loaded (/usr/lib/systemd/system/httpd.service; enabled; preset: di>
     Active: active (running) since Tue 2026-07-21 09:50:18 KST; 40s ago
       Docs: man:httpd.service(8)
   Main PID: 2997 (httpd)
     Status: "Total requests: 0; Idle/Busy workers 100/0;Requests/sec: 0; Bytes>
      Tasks: 177 (limit: 10320)
     Memory: 14.0M (peak: 14.4M)
        CPU: 103ms
     CGroup: /system.slice/httpd.service
             ├─2997 /usr/sbin/httpd -DFOREGROUND
             ├─2998 /usr/sbin/httpd -DFOREGROUND
             ├─2999 /usr/sbin/httpd -DFOREGROUND
             ├─3000 /usr/sbin/httpd -DFOREGROUND
             └─3001 /usr/sbin/httpd -DFOREGROUND

 7월 21 09:50:18 Server-A systemd[1]: Starting The Apache HTTP Server...
 7월 21 09:50:18 Server-A httpd[2997]: AH00558: httpd: Could not reliably deter>
 7월 21 09:50:18 Server-A httpd[2997]: Server configured, listening on: port 80
 7월 21 09:50:18 Server-A systemd[1]: Started The Apache HTTP Server.
lines 1-20/20 (END)
```

```bash
[root@Server-A ~]# firewall-cmd  --permanent  --add-service=http
success
```

```bash
[root@Server-A ~]# firewall-cmd  --permanent  --add-port=80/tcp
success
```

```bash
[root@Server-A ~]# firewall-cmd  --reload
success
```

```bash
[root@Server-A ~]# firewall-cmd  --list-port
20/tcp 21/tcp 22/tcp 53/tcp 80/tcp 111/tcp 139/tcp 445/tcp 2002/tcp 2049/tcp 53/udp 67/udp 68/udp 111/udp 137/udp 138/udp
```

```bash
[root@Server-A ~]# firewall-cmd  --list-service
cockpit dhcp dhcpv6-client dns ftp http nfs rpc-bind samba ssh
```

```bash
[root@Server-A ~]# vi  /var/www/html/index.html
<style>
        h1 {
                text-align: center;
        }
</style>
```

<h1>Soldesk IT Academy</h1>

<p>안녕하세요 솔데스크 학원입니다.</p>
<hr>
<h3>개설 과정 안내</h3>

<ul>
        <li>AWS 데브옵스 과정</li>
        <li>네트워크 전문가 과정</li>
        <li>AI 데이터 분석 과정</li>
        <li>풀스텍 개발 과정</li>
</ul>

- Server-A에서 파이어 폭스를 사용하여 http://192.168.10.100으로 접속 
- Client-L에서 파이어 폭스를 사용하여 http://192.168.10.100으로 접속 

```bash
	# Server-B를 FTP Server로 구성

[root@localhost ~]# hostnamectl  set-hostname  Server-B

[root@Server-B ~]# cat  /etc/hostname
Server-B
```

```bash
[root@Server-B ~]# dnf  install  -y  net-tools  bind-utils  traceroute		# 네트워크 관련 Package 설치
```

```bash
[root@Server-B ~]# ifconfig ens160
ens160: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.10.200  netmask 255.255.255.0  broadcast 192.168.10.255
        inet6 fe80::20c:29ff:fe40:fe0c  prefixlen 64  scopeid 0x20<link>
        ether 00:0c:29:40:fe:0c  txqueuelen 1000  (Ethernet)
        RX packets 20187  bytes 29543668 (28.1 MiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 5778  bytes 363447 (354.9 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

```bash
[root@Server-B ~]# ip addr show
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: ens160: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:0c:29:40:fe:0c brd ff:ff:ff:ff:ff:ff
    altname enp3s0
    inet 192.168.10.200/24 brd 192.168.10.255 scope global noprefixroute ens160
       valid_lft forever preferred_lft forever
    inet6 fe80::20c:29ff:fe40:fe0c/64 scope link noprefixroute
       valid_lft forever preferred_lft forever
```

```bash
[root@Server-B ~]# dnf  install  -y  vsftpd				# vsFTP 설치
```

```bash
[root@Server-B ~]# systemctl  start  vsftpd
```

```bash
[root@Server-B ~]# systemctl  enable  vsftpd
Created symlink /etc/systemd/system/multi-user.target.wants/vsftpd.service → /usr/lib/systemd/system/vsftpd.service.
```

```bash
[root@Server-B ~]# systemctl  status  vsftpd
● vsftpd.service - Vsftpd ftp daemon
     Loaded: loaded (/usr/lib/systemd/system/vsftpd.service; enabled; preset: d>
     Active: active (running) since Tue 2026-07-21 10:14:25 KST; 9s ago
   Main PID: 2128 (vsftpd)
      Tasks: 1 (limit: 10453)
     Memory: 864.0K (peak: 1.3M)
        CPU: 3ms
     CGroup: /system.slice/vsftpd.service
             └─2128 /usr/sbin/vsftpd /etc/vsftpd/vsftpd.conf

 7월 21 10:14:25 Server-B systemd[1]: Starting Vsftpd ftp daemon...
 7월 21 10:14:25 Server-B systemd[1]: Started Vsftpd ftp daemon.
```

```bash
[root@Server-B ~]# rpm -qa | grep firewall
firewalld-filesystem-1.3.4-18.el9_7.noarch
python3-firewall-1.3.4-18.el9_7.noarch
firewalld-1.3.4-18.el9_7.noarch
```

```bash
[root@Server-B ~]# firewall-cmd  --permanent  --add-service=ftp
success
```

```bash
[root@Server-B ~]# firewall-cmd  --permanent  --add-port=20/tcp
success
```

```bash
[root@Server-B ~]# firewall-cmd  --permanent  --add-port=21/tcp
success
```

```bash
[root@Server-B ~]# firewall-cmd  --reload
success
```

```bash
[root@Server-B ~]# firewall-cmd  --list-port
20/tcp 21/tcp
```

```bash
[root@Server-B ~]# firewall-cmd  --list-service
]cockpit dhcpv6-client ftp ssh
```

```bash
[root@Server-B ~]# firewall-cmd  --list-all
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: ens160
  sources:
  services: cockpit dhcpv6-client ftp ssh
  ports: 20/tcp 21/tcp
  protocols:
  forward: yes
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:
```

```bash
[root@Server-B ~]# ls -l /var/ftp
합계 0
drwxr-xr-x 2 root root 6  1월 17  2026 pub	# FTP 공용 디렉터리 (익명 사용자 허용시 pub에만 접근이 가능하다.)
```

```bash
	# banner 파일 생성

[root@Server-B ~]# vi  /var/ftp/welcome.msg

---

	Welcome to Soldesk FTP Server

---

이 서버는 솔데스크 아이티 아카데미에서 운영하는 FTP 서버입니다.
공개 디렉터리는 /pub 디렉터리만 사용 가능합니다.
업로드 문의는 관리자 승인후 가능합니다.
문의 : admin@soldesk.co.kr

:wq
```

```bash
	# 설정한 banner 파일 적용

[root@Server-B ~]# vi  /etc/vsftpd/vsftpd.conf
       ~~~~~~~~~ 중간 생략 ~~~~~~~~~
    121 # files.
    122 # Make sure, that one of the listen options is commented !!
    123 listen_ipv6=YES
    124
    125 pam_service_name=vsftpd
    126 userlist_enable=YES
    127 banner_file=/var/ftp/welcome.msg	# 설정한 banner 적용

:wq
```

- 익명 사용자(anonymous) 관련 설정
                    항목			기본값	설명
  - anonymous_enable=YES		NO	익명 사용자 접속 허용
  - no_anon_password=YES		NO	익명 로그인 시 비밀번호 입력 없이 접속 허용
  - anon_root=/var/ftp		-	익명 사용자의 홈 디렉터리 지정
  - anon_upload_enable=YES		NO	익명 사용자의 파일 업로드 허용
  - anon_mkdir_write_enable=YES	NO	익명 사용자의 디렉터리 생성 허용
  - anon_other_write_enable=YES	NO	익명 사용자의 삭제/이름변경 허용
  - anon_world_readable_only=YES	YES	익명 사용자는 읽기 전용 디렉터리만 접근 가능

- 일반 사용자(local user) 관련 설정
                    항목		기본값	설명
  - local_enable=YES	YES	시스템 계정(일반 사용자) FTP 접속 허용
  - write_enable=YES	NO	파일 쓰기(업로드/삭제/수정) 허용

```bash
[root@Server-B ~]# systemctl  restart  vsftpd
```

```bash
	# FTP 접속을 허용하기위해서 Server-B에서 guest 계정 생성
[root@Server-B ~]# useradd guest
[root@Server-B ~]# passwd guest
guest 사용자의 비밀 번호 변경 중
새 암호:
잘못된 암호: 암호는 8 개의 문자 보다 짧습니다
새 암호 재입력:
passwd: 모든 인증 토큰이 성공적으로 업데이트 되었습니다.
```

```bash
	# FTP 접속을 위해서 Client-L에서 FTP 설치

[guest@Client-L ~]$ sudo  dnf  install  -y  ftp
```

```bash
[guest@Client-L ~]$ ftp 192.168.10.200
Connected to 192.168.10.200 (192.168.10.200).
220-===========================================================
220-    Welcome to Soldesk FTP Server
220-===========================================================
220-이 서버는 솔데스크 아이티 아카데미에서 운영하는 FTP 서버입니다.
220-공개 디렉터리는 /pub 디렉터리만 사용 가능합니다.
220-업로드 문의는 관리자 승인후 가능합니다.
220-문의 : admin@soldesk.co.kr
220
Name (192.168.10.200:guest): guest
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> 
ftp> pwd
257 "/home/guest" is the current directory
```

---

### Server-A를 Master Name Server로 구성

- Master Name Server
  - 특정 도메인의 Zone File(영역 파일)을 직접 생성하고 관리하는 DNS 서버
  - 해당 도메인의 A, NS, MX, CNAME 등의 DNS 레코드를 직접 보유한다.
  - 클라이언트가 해당 도메인에 대해 질의하면 Zone File을 기준으로 응답한다.
  - 다른 DNS 서버가 이 서버의 Zone 정보를 복제하도록 구성할 수도 있다.
  - 현재 BIND에서는 Master/Slave 대신 Primary/Secondary라는 용어도 사용한다. 

- named.conf
  - BIND DNS 서버의 전체 동작 방식을 설정하는 기본 설정 파일
  - BIND의 서비스 프로세스인 named가 시작될 때 읽어 들인다.
  - 어떤 도메인을 관리할 것인지, Zone File이 어디에 있는지, 어떤 IP 주소에서 DNS 요청을 받을지 등을 설정한다.
  - DNS 질의를 허용할 대상, 재귀 질의 허용 여부, 캐싱 및 보안 정책도 설정한다.
  - 기본 설정 파일의 경로는 /etc/named.conf이다. 

- Zone
  - DNS 서버가 관리하는 도메인 영역을 의미한다.
  - 예를 들어 soldesk.com Zone을 등록하면 다음과 같은 이름을 관리할 수 있다. 
      www.soldesk.com 
      mail.soldesk.com 
      ftp.soldesk.com ns.soldesk.com

- Zone File 
  - 특정 도메인의 실제 DNS 레코드를 저장하는 파일
  - A, NS, MX, CNAME, PTR 등의 레코드를 작성한다.
  - 일반적으로 /var/named 디렉터리에 저장된다.
  - named.conf의 file 항목에는 Zone File의 파일명을 지정한다.

```bash
[root@Server-A ~]# vi  /etc/named.conf
~~~~~~~~~ 중간 생략 ~~~~~~~~~
     51
     52 zone "." IN {
     53         type hint;
     54         file "named.ca";
     55 };
     56
     57 include "/etc/named.rfc1912.zones";
     58 include "/etc/named.root.key";
     59
     60 zone "soldesk.com" IN {
     61         type master;
     62         file "soldesk.com.db";
     63 };

:wq
```

```bash
[root@Server-A ~]# named-checkconf		# named.conf 에러 체크
```

---

#### Zone 파일이란?

zone 파일 = DNS 서버가 직접 관리하는 주소록 파일

예: www.example.com 	--> 	192.168.10.100
예: mail.example.com 	--> 	192.168.10.200

- 도메인 이름을 IP 주소로 바꾸는 데이터가 들어 있는 파일이다.

- Domain(도메인)	: 인터넷에서 쓰는 이름 전체 구조
- Zone(존)		: 그 도메인 중에서 해당 DNS 서버가 직접 관리하는 부분만 잘라낸 것

- 즉 도메인 전체 중에서 우리 DNS 서버가 맡은 부분 = Zone
- 그 관리 데이터가 들어있는 파일 = Zone 파일

- DNS 서버 설정은 두 군데에서 이루어진다

/etc/named.conf	: 어떤 도메인을 관리할지 선언하는 곳 (zone 파일 위치도 여기서 지정)

EX)
zone "example.com" {
    type master;
    file "example.com.zone";
};

이 설정은 example.com 도메인의 데이터는 example.com.zone 이라는 파일에 있다.

/var/named/	: 실제 zone 파일이 저장되는 디렉터리

EX)
/var/named/example.com.zone

- Zone의 구성 요소 (Zone은 보통 다음 파일 구조로 구성)
  - /etc/named.conf		: DNS 서버의 전체 설정 파일 (여기서 zone 정의)
  - /var/named/도메인명.db	: 실제 zone 데이터(레코드) 저장 파일

- zone 파일은 기본적으로 '/var/named/' 디렉터리 안에 생성해야한다.
 이유는 '/etc/named.conf' 파일에 zone 파일의 기본 경로가 '/var/named/'로 설정되어있다.

Zone 파일의 내용 예시 (soldesk.com.db)
$TTL 1D
@   IN SOA  ns1.soldesk.com. admin.soldesk.com. (
              	2025110201 ; Serial 번호
              	1H 	; Refresh
              	10M    	; Retry
              	1W     	; Expire
              	1D 	; Minimum TTL

    	IN NS  ns1.soldesk.com.		# soldesk.com의 DNS 서버를 ns1.soldesk.com으로 지정
ns1 	IN A   192.168.1.1			# ns1.soldesk.com의 IPv4 주소를 192.168.1.1로 지정
www 	IN A   192.168.1.1			# www.soldesk.com의 IPv4 주소를 192.168.1.1로 지정
mail 	IN A   192.168.1.2			# mail.soldesk.com의 IPv4 주소를 192.168.1.2로 지정

    구분		이름		설명
프로그램 이름	BIND		DNS 서버 소프트웨어의 이름 (패키지 이름)
데몬(서비스 이름)	named		실제로 실행되는 DNS 서버 프로세스 이름
데이터 디렉터리	/var/named/	BIND(named)가 사용하는 zone 파일 저장 위치

```bash
[root@Server-A ~]# ls  -l  /var/named
합계 16
drwxrwx--- 2 named named    44  7월 20 17:40 data
drwxrwx--- 2 named named    60  7월 21 09:37 dynamic
-rw-r----- 1 root     named 2112  6월 10 12:48 named.ca
-rw-r----- 1 root     named  152  6월 10 12:48 named.empty
-rw-r----- 1 root     named  152  6월 10 12:48 named.localhost
-rw-r----- 1 root     named  168  6월 10 12:48 named.loopback
drwxrwx--- 2 named named     6  6월 10 12:58 slaves
```

```bash
[root@Server-A named]# touch soldesk.com.db 		# '/etc/named.conf'에서 설정한 파일명을 생성 (vi로 생성 가능)
```

```bash
[root@Server-A ~]# ls  -l  /var/named
합계 16
drwxrwx--- 2 named named    44  7월 20 17:40 data
drwxrwx--- 2 named named    60  7월 21 09:37 dynamic
-rw-r----- 1 root     named 2112  6월 10 12:48 named.ca
-rw-r----- 1 root     named  152  6월 10 12:48 named.empty
-rw-r----- 1 root     named  152  6월 10 12:48 named.localhost
-rw-r----- 1 root     named  168  6월 10 12:48 named.loopback
drwxrwx--- 2 named named     6  6월 10 12:58 slaves
-rw-r--r-- 1 root  root           0  7월 21 11:18 soldesk.com.db	# 파일 생성 확인
```

```bash
[root@Server-A ~]# cp  /var/named/named.empty  /var/named/soldesk.com.db		# 예시파일인 named.empty를 soldesk.com.db로 복사
cp: overwrite '/var/named/soldesk.com.db'? y
```

```bash
[root@Server-A ~]# cat  /var/named/soldesk.com.db
$TTL 3H
@       IN SOA  @ rname.invalid. (
                                        0       	; serial
                                        1D      	; refresh
                                        1H      	; retry
                                        1W      	; expire
                                        3H )	; minimum
        NS      @
        A       127.0.0.1
        AAAA    ::1
```

```bash
[root@Server-A ~]# vi  /var/named/soldesk.com.db
$TTL 1D
@       IN SOA  ns.soldesk.com. admin.naver.com. (
                                        20260721	; serial
                                        1D      	; refresh
                                        1H      	; retry
                                        1W      	; expire
                                        3H )    	; minimum
@       	IN      NS      ns.soldesk.com.		# soldesk.com의 DNS 서버 이름
@        	IN      A       192.168.10.100		# soldesk.com의 기본 IP 주소 설정

ns      	IN      A       192.168.10.100		# ns.soldesk.com의 IP 주소
www	IN      A       192.168.10.100		# www.soldesk.com의 IP 주소
ftp     	IN      A       192.168.10.200		# ftp.soldesk.com의 IP 주소

:wq
```

- named-checkzone
  - named-checkzone 명령은 특정 도메인(zone) 에 대한 zone 데이터 파일의 문법과 구조가 올바른지 검사하는 도구
    즉,named.conf 설정이 아니라, 실제 zone 파일(soldesk.com.db) 내부의 A, NS, SOA 등의 구문이 규칙에 맞게 작성되어 있는지를 확인
  - 형식 : named-checkzone <도메인명> <zone파일경로>

```bash
[root@Server-A ~]# named-checkzone  soldesk.com  /var/named/soldesk.com.db
zone soldesk.com/IN: loaded serial 20260721
OK
```

```bash
[root@Server-A ~]# chown  root:named  /var/named/soldesk.com.db
[root@Server-A ~]# chmod  640   /var/named/soldesk.com.db
```

```bash
[root@Server-A ~]# ls  -l  /var/named/
합계 24
drwxrwx--- 2 named  named   44  7월 20 17:40 data
drwxrwx--- 2 named  named   60  7월 21 12:53 dynamic
-rw-r----- 1  root    named 2112  6월 10 12:48 named.ca
-rw-r----- 1  root    named  152  6월 10 12:48 named.empty
-rw-r----- 1  root    named  152  6월 10 12:48 named.localhost
-rw-r----- 1  root    named  168  6월 10 12:48 named.loopback
drwxrwx--- 2 named named    6  6월 10 12:58 slaves
-rw-r----- 1  root    named  271  7월 21 11:45 soldesk.com.db	# other에게는 권한을 주지 않고 같은 그룹만 읽을수 있게 수정
```

```bash
[root@Server-A ~]# systemctl  restart named
```

```bash
[root@Server-A ~]# systemctl  status named
● named.service - Berkeley Internet Name Domain (DNS)
     Loaded: loaded (/usr/lib/systemd/system/named.service; enabled; preset: di>
     Active: active (running) since Tue 2026-07-21 11:53:33 KST; 15s ago
    Process: 5217 ExecStartPre=/bin/bash -c if [ ! "$DISABLE_ZONE_CHECKING" == >
    Process: 5219 ExecStart=/usr/sbin/named -u named -c ${NAMEDCONF} $OPTIONS (>
   Main PID: 5220 (named)
      Tasks: 8 (limit: 10320)
     Memory: 22.2M (peak: 22.6M)
        CPU: 47ms
     CGroup: /system.slice/named.service
             └─5220 /usr/sbin/named -u named -c /etc/named.conf

 7월 21 11:53:33 Server-A named[5220]: network unreachable resolving './NS/IN':>
 7월 21 11:53:33 Server-A named[5220]: network unreachable resolving './DNSKEY/>
 7월 21 11:53:33 Server-A named[5220]: network unreachable resolving './NS/IN':>
 7월 21 11:53:33 Server-A named[5220]: network unreachable resolving './DNSKEY/>
 7월 21 11:53:33 Server-A named[5220]: network unreachable resolving './NS/IN':>
 7월 21 11:53:33 Server-A named[5220]: network unreachable resolving './DNSKEY/>
 7월 21 11:53:33 Server-A named[5220]: network unreachable resolving './NS/IN':>
 7월 21 11:53:33 Server-A named[5220]: resolver priming query complete
 7월 21 11:53:33 Server-A named[5220]: managed-keys-zone: Key 20326 for zone . >
 7월 21 11:53:33 Server-A named[5220]: managed-keys-zone: Key 38696 for zone . >
lines 1-22/22 (END)
```

- Client-L에서 http://www.soldesk.com으로 접속하게되면 실제 솔데스크 학원의 홈페이지로 접속된다.
- 이유 : 현재 /etc/resolv.conf 파일에는 실제 DNS Server의 IP 가 설정되어있기때문에 실제 홈페이지로 이동된다.

```bash
[guest@Client-L ~]$ cat /etc/resolv.conf
# Generated by NetworkManager
search localdomain
nameserver 192.168.10.2		# 실제 DNS Server IP 주소
```

```bash
[guest@Client-L ~]$ vi /etc/resolv.conf
# Generated by NetworkManager
search localdomain
#nameserver 192.168.10.2		# 주석 처리
nameserver 192.168.10.100		# DNS Server의 IP 주소를 Server-A의 IP 주소로 수정

:wq
```

- Client-L에서 http://www.soldesk.com으로 접속하게되면 Server-A의 Web Server로 접속된다.

```bash
[guest@Client-L ~]$ nslookup
>
> server
Default server: 192.168.10.100
Address: 192.168.10.100#53		# 현재 DNS Server IPv4 주소
>
> www.soldesk.com
Server:	192.168.10.100
Address:	192.168.10.100#53

Name:   www.soldesk.com
Address: 192.168.10.100		# www.soldesk.com 도메인의 IPv4 주소
>
>
> ftp.soldesk.com
Server:	192.168.10.100
Address:	192.168.10.100#53

Name:   ftp.soldesk.com
Address: 192.168.10.200		# ftp.soldesk.com 도메인의 IPv4 주소
>
```

```bash
-이전에는 브라우저에서 FTP 접속이 가능했지만 현재는 거의 대부분의 브라우저에서 FTP 기능을 지원하지 않는다.

[guest@Client-L ~]$ ftp  ftp.soldesk.com
Connected to ftp.soldesk.com (192.168.10.200).
220-===========================================================
220-    Welcome to Soldesk FTP Server
220-===========================================================
220-이 서버는 솔데스크 아이티 아카데미에서 운영하는 FTP 서버입니다.
220-공개 디렉터리는 /pub 디렉터리만 사용 가능합니다.
220-업로드 문의는 관리자 승인후 가능합니다.
220-문의 : admin@soldesk.co.kr
220
Name (ftp.soldesk.com:guest): guest
331 Please specify the password.
Password:1234
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp>
```

```bash
-리눅스는 브라우저가 없기때문에 명령어를 사용해서 확인해야 한다.

[guest@Client-L ~]$ curl  192.168.10.100

[guest@Client-L ~]$ curl  http://www.soldesk.com
```

---

#### 역방향 DNS

- DNS (정방향)
  - Domain에 대해서 IP 주소로 변환하는 기능
  - EX) www.soldesk.com  --->  192.168.10.100

- DNS (역방향)
  - IP 주소를 사용하여 Domain주소로 변환하는 기능
  - EX) 192.168.10.100  --->  www.soldesk.com

#### 역방향 DNS를 설정해야 하는 이유

- 역방향 DNS는 특히 메일 서버의 신뢰성을 확인할 때 중요하게 사용된다.
  - 메일을 수신하는 SMTP 서버는 메일을 전송한 서버가 정상적으로 등록된 서버인지 확인하기 위해 역방향 DNS 정보를 검사할 수 있다.
  - 역방향 DNS가 설정되어 있지 않거나 DNS 정보가 서로 일치하지 않으면 해당 메일이 스팸으로 분류되거나 수신이 거부될 수 있다.

- SMTP 서버는 다음을 검사
  - SMTP(Simple Mail Transfer Protocol)는 이메일을 전송할 때 사용하는 표준 프로토콜
  - 1) 메일을 전송한 서버의 IP 주소에 PTR 레코드가 등록되어 있는가?
  - 2) PTR 레코드로 확인된 도메인 이름을 정방향 조회했을 때 원래 메일 서버의 IP 주소가 다시 조회되는가?
  - 이처럼 역방향 조회 결과를 다시 정방향으로 조회하여 원래 IP 주소와 일치하는지 확인하는 방식을 FCrDNS(Forward-confirmed Reverse DNS)라고 한다

- 정방향: mail.soldesk.com  --->  192.168.111.50
- 역방향: 192.168.10.50  --->  mail.soldesk.com

- 만약 역방향 DNS가 없으면 정체불명의 서버로 판단되어  스팸 점수 증가하고 메일이 스팸함으로 이동하거나 아예 수신 거부될수 있다.

- SSH / 웹 / 시스템 로그 분석이 쉬워진다.

- 로그를 보면 대부분 IP만 찍힌다.

EX) Failed password for 192.168.10.200
EX) Connection from 192.168.10.150 accepted

- IP만 가지고는 어떤 서버인지 파악하기 어렵다.
- 하지만 역방향 DNS가 있으면:

192.168.10.200  --->  ftp.soldesk.com
192.168.10.150  --->  Client-L.soldesk.com

- 이름이 보이기 때문에 누가 접속했고 어떤 장비인지 한눈에 알 수 있다.

- 보안 이벤트 분석, 장애 대응 속도가 매우 빨라진다.

```bash
[root@Server-A ~]# nslookup  dns.google
Server:	      192.168.10.2
Address:	      192.168.10.2#53

Non-authoritative answer:
Name:   dns.google
Address: 8.8.8.8
Name:   dns.google
Address: 8.8.4.4
Name:   dns.google
Address: 2001:4860:4860::8888
Name:   dns.google
Address: 2001:4860:4860::8844
```

```bash
[root@Server-A ~]# nslookup  8.8.8.8
8.8.8.8.in-addr.arpa    name = dns.google.

Authoritative answers can be found from:
```

```bash
[root@Server-A ~]# nslookup  168.126.63.1
1.63.126.168.in-addr.arpa       name = kns.kornet.net.

Authoritative answers can be found from:
```

```bash
[root@Server-A ~]# vi  /etc/named.conf
~~~~~~~~~ 중간 생략 ~~~~~~~~~

include "/etc/named.rfc1912.zones";
include "/etc/named.root.key";

zone "soldesk.com" IN {
        type master;
        file "soldesk.com.db";
};

zone "10.168.192.in-addr.arpa" IN {	# 설정
        type master;			# 설정
        file "soldesk.com.rev";		# 설정
};				# 설정
```

```bash
[root@Server-A ~]# cp  /var/named/named.empty  /var/named/soldesk.com.rev
```

```bash
[root@Server-A ~]# ls -l /var/named
합계 24
drwxrwx--- 2 named named    44  7월 20 17:40 data
drwxrwx--- 2 named named    60  7월 21 11:54 dynamic
-rw-r----- 1 root     named 2112  6월 10 12:48 named.ca
-rw-r----- 1 root     named  152  6월 10 12:48 named.empty
-rw-r----- 1 root     named  152  6월 10 12:48 named.localhost
-rw-r----- 1 root     named  168  6월 10 12:48 named.loopback
drwxrwx--- 2 named named     6  6월 10 12:58 slaves
-rw-r--r-- 1 root     root     271  7월 21 11:45 soldesk.com.db
-rw-r----- 1 root     root     152  7월 21 12:38 soldesk.com.rev
```

```bash
[root@Server-A ~]# vi  /var/named/soldesk.com.rev
$TTL 1D
@       IN SOA  ns.soldesk.com. root.soldesk.com. (
                                        2026072101      ; serial
                                        1D              ; refresh
                                        1H              ; retry
                                        1W              ; expire
                                        3H )            ; minimum
@        	IN      	NS      	ns.soldesk.com.
100     	IN      	PTR     	www.soldesk.com.
200	IN      	PTR	ftp.soldesk.com.

:wq
```

- PTR (Pointer) 레코드
  - A 레코드는 Domian요청에 대해서 IP 주소로 변환하는 기능
  - PTR 레코드는 A 레코드와 반대로 IP 주소에 대해서 domain으로 변환하는 기능

```bash
[root@Server-A ~]# named-checkzone 10.168.192.in-addr.arpa /var/named/soldesk.com.rev
zone 10.168.192.in-addr.arpa/IN: loaded serial 2026072101
OK
```

```bash
[root@Server-A ~]# systemctl  restart  named
```

```bash
[root@Server-A ~]# chown  root:named  /var/named/soldesk.com.rev
[root@Server-A ~]# chmod  640   /var/named/soldesk.com.rev
```

```bash
[root@Server-A ~]# ls  -l  /var/named/
합계 24
drwxrwx--- 2 named  named   44  7월 20 17:40 data
drwxrwx--- 2 named  named   60  7월 21 12:53 dynamic
-rw-r----- 1  root    named 2112  6월 10 12:48 named.ca
-rw-r----- 1  root    named  152  6월 10 12:48 named.empty
-rw-r----- 1  root    named  152  6월 10 12:48 named.localhost
-rw-r----- 1  root    named  168  6월 10 12:48 named.loopback
drwxrwx--- 2 named named    6  6월 10 12:58 slaves
-rw-r----- 1  root    named  271  7월 21 11:45 soldesk.com.db
-rw-r----- 1  root    named  233  7월 21 12:43 soldesk.com.rev	# other에게는 권한을 주지 않고 같은 그룹만 읽을수 있게 수정
```

```bash
[root@Server-A ~]# systemctl restart  named
```

```bash
	# Server-A에서 자신의 DNS에게 요청 (동작하는지 테스트)
[root@Server-A ~]# nslookup  192.168.10.100  127.0.0.1
100.10.168.192.in-addr.arpa     name = www.soldesk.com.
```

```bash
	# Server-A에서 자신의 DNS에게 요청 (동작하는지 테스트)
[root@Server-A ~]# nslookup  192.168.10.200  127.0.0.1
200.10.168.192.in-addr.arpa     name = ftp.soldesk.com.
```

```bash
	# 실제 사용자인 Client-L에서 확인
[guest@Client-L ~]$ nslookup  192.168.10.100
100.10.168.192.in-addr.arpa     name = www.soldesk.com.
```

```bash
	# 실제 사용자인 Client-L에서 확인
[guest@Client-L ~]$ nslookup  192.168.10.200
200.10.168.192.in-addr.arpa     name = ftp.soldesk.com.
```

---

- 실무에서 사용하는 구조(named.conf + named.rfc1912.zones + zone 파일 분리) 를 적용한다.

- 실습 환경 구성
구분	호스트명	IP 주소	역할
Server-A	192.168.10.100	DNS Master / HTTP Web Server	
Server-B	192.168.10.200	FTP Server	
Client-L	192.168.10.150	DNS Client (테스트용)	

```bash
               # Server-A를 HTTP Server로 구성

[root@Server-A ~]# rpm  -qa  | grep http	<---- HTTP관련 Package가 설치되지 않음
```

```bash
[root@Server-A ~]# dnf -y install httpd	<---- HTTP관련 Package 설치
```

```bash
[root@Server-A ~]# systemctl start httpd	<---- HTTP Server 실행
```

```bash
[root@Server-A ~]# systemctl enable httpd	<---- Server-A가 재부팅되어도 자동 실행
Created symlink from /etc/systemd/system/multi-user.target.wants/httpd.service to /usr/lib/systemd/system/httpd.service.
```

```bash
[root@Server-A ~]# systemctl status httpd	<---- 서비스 상태 확인
● httpd.service - The Apache HTTP Server
     Loaded: loaded (/usr/lib/systemd/system/httpd.service; enabled; vendor pre>
     Active: active (running) since Thu 2025-10-30 23:09:34 KST; 12min ago
       Docs: man:httpd.service(8)
   Main PID: 2372 (httpd)
     Status: "Total requests: 0; Idle/Busy workers 100/0;Requests/sec: 0; Bytes>
      Tasks: 213 (limit: 22611)
     Memory: 45.0M
        CPU: 698ms
     CGroup: /system.slice/httpd.service
             ├─2372 /usr/sbin/httpd -DFOREGROUND
             ├─2373 /usr/sbin/httpd -DFOREGROUND
             ├─2374 /usr/sbin/httpd -DFOREGROUND
             ├─2375 /usr/sbin/httpd -DFOREGROUND
             └─2376 /usr/sbin/httpd -DFOREGROUND
```

```bash
[root@Server-A ~]# firewall-cmd  --permanent  --add-port=80/tcp
success

[root@Server-A ~]# firewall-cmd  --permanent  --add-service=http
success

[root@Server-A ~]# firewall-cmd  --reload
success
```

```bash
[root@Server-A ~]# firewall-cmd  --list-all
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: ens33
  sources:
  services: dhcpv6-client dns http ssh
  ports: 53/tcp 53/udp 80/tcp
  protocols:
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:
```

```bash
[root@Server-A ~]# vi  /var/www/html/index.html
<style>
        h1 {
            text-align: center;
        }
</style>

<h1>Soldesk IT Academy</h1>
<p>안녕하세요. 솔데스크 학원입니다.</p>
<hr>
<h3>개설 과정 안내</h3>
<ul>
        <li>네트워크 보안 전문가 과정</li>
        <li>클라우드 엔지니어 과정</li>
        <li>AI · 데이터 분석 과정</li>
        <li>웹 개발자 양성 과정</li>
</ul>

:wq
```

```bash
[root@Server-A ~]# cat  /var/www/html/index.html
<h1> Global IT Academy </h1>
안녕하세요 글로벌 아이티 인재개발원입니다.
```

http://192.168.10.100	# Server-A에서 파이어 폭스로 확인
curl 192.168.10.100		# Server-A에서 명령어로 확인

http://192.168.10.100	# Client-L에서 파이어 폭스로 확인
curl 192.168.10.100		# Client-L에서 명령어로 확인

#### Server-B를 FTP Server로 구성

```bash
[root@localhost ~]# hostnamectl  set-hostname  Server-B

[root@Server-B ~]# cat /etc/hostname		<--- 해당 경로의 파일에 저장된다.
Server-B
```

```bash
[root@Server-B ~]# ifconfig		<---- 네트워크 장치 및 IP 주소가 확인되지 않는다. (Server-B는 최소 설치이므로 네트워크 툴을 별도로 설치해야한다.)
-bash: ifconfig: command not found
```

```bash
[root@Server-B ~]# dnf  -y  install  net-tools	<---- 네트워크 관련 Package 설치

[root@Server-B ~]# dnf  -y  install  vsftpd	<---- FTP Package 설치
```

```bash
[root@Server-B ~]# systemctl status vsftpd
○ vsftpd.service - Vsftpd ftp daemon
     Loaded: loaded (/usr/lib/systemd/system/vsftpd.service; disabled; vendor p>
     Active: inactive (dead)			<---- 서버가 동작하지 않는다.
```

```bash
[root@Server-B ~]# systemctl  start  vsftpd

[root@Server-B ~]# systemctl  enable  vsftpd
Created symlink /etc/systemd/system/multi-user.target.wants/vsftpd.service → /usr/lib/systemd/system/vsftpd.service.
```

```bash
[root@Server-B ~]# firewall-cmd  --permanent  --add-port=20/tcp
success

[root@Server-B ~]# firewall-cmd  --permanent  --add-port=21/tcp
success

[root@Server-B ~]# firewall-cmd  --permanent  --add-service=ftp
success

[root@Server-B ~]# firewall-cmd  --reload
success
```

```bash
[root@Server-B ~]# firewall-cmd  --list-all
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: ens32
  sources:
  services: dhcpv6-client ftp ssh
  ports: 20/tcp 21/tcp
  protocols:
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:
```

```bash
[root@Server-B ~]# cd /var/ftp        		# FTP 기본 경로로 이동
```

```bash
[root@Server-B ftp]# ls -l
total 0
drwxr-xr-x 2 root root 6 Nov  7  2024 pub   	# 외부 사용자가 접근 가능한 공개용 디렉터리 (기본 제공)
```

```bash
[root@Server-B ftp]# vi welcome.msg
```

- --

Welcome to Soldesk FTP Server

- --

이 서버는 솔데스크 학원에서 운영하는 FTP 서버입니다.
공개 디렉터리는 /pub 폴더를 이용해 주세요.
업로드는 관리자 승인 후 가능합니다.
문의: admin@soldesk.com

:wq

```bash
[root@Server-B ftp]# vi /etc/vsftpd/vsftpd.conf
     banner_file=/var/ftp/welcome.msg		<--- 첫줄에 설정
       ~~~~~~~~~ 중간 생략 ~~~~~~~~~
     34 # Activate directory messages - messages given to remote users when they
     35 # go into a certain directory.
     36 dirmessage_enable=YES
     37 #
     38 # Activate logging of uploads/downloads.
     39 xferlog_enable=YES
     40 #
       ~~~~~~~~~ 중간 생략 ~~~~~~~~~
```

- 익명 사용자(anonymous) 관련 설정
                    항목			기본값	설명
  - anonymous_enable=YES		NO	익명 사용자 접속 허용
  - no_anon_password=YES		NO	익명 로그인 시 비밀번호 입력 없이 접속 허용
  - anon_root=/var/ftp		-	익명 사용자의 홈 디렉터리 지정
  - anon_upload_enable=YES		NO	익명 사용자의 파일 업로드 허용
  - anon_mkdir_write_enable=YES	NO	익명 사용자의 디렉터리 생성 허용
  - anon_other_write_enable=YES	NO	익명 사용자의 삭제/이름변경 허용
  - anon_world_readable_only=YES	YES	익명 사용자는 읽기 전용 디렉터리만 접근 가능

- 일반 사용자(local user) 관련 설정
                    항목		기본값	설명
  - local_enable=YES	YES	시스템 계정(일반 사용자) FTP 접속 허용
  - write_enable=YES	NO	파일 쓰기(업로드/삭제/수정) 허용

```bash
[root@Server-B ftp]# systemctl restart vsftpd
```

#### Client-L에서 Server-B로 FTP 접속

```bash
[root@Client-L ~]# dnf  -y  install  ftp		# Clinet-L에서 FTP 접속을 위해서 FTP Package 설치
```

```bash
[root@Client-L ~]# ftp  192.168.10.200
Connected to 192.168.10.200 (192.168.10.200).
220-========================================
220-Welcome to Soldesk FTP Server
220-========================================
220-이 서버는 솔데스크 학원에서 운영하는 FTP 서버입니다.
220-공개 디렉터리는 /pub 폴더를 이용해 주세요.
220-업로드는 관리자 승인 후 가능합니다.
220-문의: admin@soldesk.com
220-
220-
220-
220
Name (192.168.10.200:root): guest	<---- 'guest' 계정으로 접속
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp>
ftp> pwd
257 "/home/guest"
ftp>
```

```bash
[root@Server-A ~]# vi /etc/named.conf
  1 //
  2 // named.conf
  3 //
  4 // Provided by Red Hat bind package to configure the ISC BIND named(8) DNS
  5 // server as a caching only nameserver (as a localhost DNS resolver only).
  6 //
  7 // See /usr/share/doc/bind*/sample/ for example named configuration files.
  8 //
  9
 10 options {
 11         listen-on port 53 { any; };		<---- 127.0.0.1 = 자신의 DNS만 처리 | any = 모든 DNS를 처리
 12         listen-on-v6 port 53 { none; };		<---- IPv6 사용 X
 13         directory       "/var/named";
 14         dump-file       "/var/named/data/cache_dump.db";
 15         statistics-file "/var/named/data/named_stats.txt";
 16         memstatistics-file "/var/named/data/named_mem_stats.txt";
 17         secroots-file   "/var/named/data/named.secroots";
 18         recursing-file  "/var/named/data/named.recursing";
 19         allow-query     { any; };		<---- Localhost = 자신의 DNS 요청만 처리 | any = 모든 DNS 요청을 처리
 20
 21         /*
 22          - If you are building an AUTHORITATIVE DNS server, do NOT enable recursion.
 23          - If you are building a RECURSIVE (caching) DNS server, you need to enable
 24            recursion.
 25          - If your recursive DNS server has a public IP address, you MUST enable access
 26            control to limit queries to your legitimate users. Failing to do so will
 27            cause your server to become part of large scale DNS amplification
 28            attacks. Implementing BCP38 within your network would greatly
 29            reduce such attack surface
 30         */
 31         recursion yes;
 32
 33         dnssec-validation no;		<---- 실습용 로컬 네임서버에서는 DNSSEC 검증이 필요 없으므로 yes를 no로 설정
```

- /etc/named.conf에 아래의 설정이 있어야한다. (기본으로 설정되어 있다.)

options {
        directory       "/var/named";
        recursion       yes;
        allow-query     { any; };
};

zone "." IN {
        type hint;
        file "named.ca";
};

include "/etc/named.rfc1912.zones";
include "/etc/named.root.key";

#### 정방향 DNS 등록

- soldesk.com 도메인에 대한 정방향 해석 (도메인 --> IP) 을 처리하는 영역
- type master는 이 서버가 원본(zone data)을 직접 관리함을 의미한다.

```bash
[root@Server-A ~]# vi /etc/named.rfc1912.zones
          ~~~~~~ 중간 생략 ~~~~~~
zone "soldesk.com" IN {
        type master;
        file "soldesk.com.dns";
};

:wq
```

```bash
[root@Server-A ~]# cd /var/named
[root@Server-A named]# vi soldesk.com.dns
$TTL 1D
@   IN 	SOA 	ns.soldesk.com. 	admin.soldesk.com. (
        		2025110201 ; serial
        		1D         ; refresh
        		1H         ; retry
        		1W         ; expire
        		3H )       ; minimum
   	 IN NS   ns.soldesk.com.
   	 IN A    192.168.10.100

ns  	IN A    192.168.10.100
www 	IN A    192.168.10.100
ftp 	IN A    192.168.10.200

:wq
```

```bash
[root@Server-A ~]# named-checkzone soldesk.com /var/named/soldesk.com.dns
zone soldesk.com/IN: loaded serial 2025110201
OK
```

```bash
	# 역방향 DNS 등록

-IP 주소  -->  도메인 이름으로 해석할 수 있게 해준다.
-in-addr.arpa 도메인은 역방향 네임스페이스 전용으로 사용

[root@Server-A ~]# vi /etc/named.rfc1912.zones
          ~~~~~~ 중간 생략 ~~~~~~
zone "soldesk.com" IN {
        type master;
        file "soldesk.com.dns";
};

zone "10.168.192.in-addr.arpa" IN {	<--- 추가
        type master;			<--- 추가
        file "soldesk.com.rev";		<--- 추가
};				<--- 추가

:wq
```

```bash
[root@Server-A named]# vi soldesk.com.rev
$TTL 1D
@   IN 	SOA 	ns.soldesk.com. 	root.soldesk.com. (
        		2025110201 ; serial
        		1D         ; refresh
        		1H         ; retry
        		1W         ; expire
       		3H )       ; minimum
    	IN NS   ns.soldesk.com.

100 	IN PTR www.soldesk.com.
200 	IN PTR ftp.soldesk.com.
:wq
```

```bash
[root@Server-A ~]# tail -10 /etc/named.rfc1912.zones
zone "soldesk.com" IN {
        type master;
        file "soldesk.com.dns";
};

zone "10.168.192.in-addr.arpa" IN {
        type master;
        file "soldesk.com.rev";
};
```

```bash
[root@Server-A ~]# named-checkzone 10.168.192.in-addr.arpa /var/named/soldesk.com.rev
zone 10.168.192.in-addr.arpa/IN: loaded serial 2025110201
OK
```

```bash
[root@Server-A ~]# systemctl restart named
[root@Server-A ~]# systemctl enable named
[root@Server-A ~]# systemctl status named
```

```bash
[root@Client-L ~]# ftp ftp.soldesk.com		<---- Clinet-L Server-B로 Domain을 사용하여 FTP 접속
Connected to ftp.soldesk.com (192.168.10.200).
220-========================================
220-Welcome to Soldesk FTP Server
220-========================================
220-이 서버는 솔데스크 학원에서 운영하는 FTP 서버입니다.
220-공개 디렉터리는 /pub 폴더를 이용해 주세요.
220-업로드는 관리자 승인 후 가능합니다.
220-문의: admin@soldesk.com
220-
220-
220-
220
Name (ftp.soldesk.com:root): guest
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp>
ftp>
ftp> pwd
257 "/home/guest" is the current directory
ftp>
```

#### Client-L에서 브라우저를 사용하여 웹서버 접속

http://www.soldesk.com/

Soldesk IT Academy

안녕하세요. 솔데스크 학원입니다.
개설 과정 안내

    네트워크 보안 전문가 과정
    클라우드 엔지니어 과정
    AI  데이터 분석 과정
    웹 개발자 양성 과정

#### Client-Win에서 브라우저를 사용하여 웹서버 접속이 가능하다.

---

**정리**: Master DNS Server (10-7) 섹션에서 다룬 명령어와 실습 예제를 통해 관련 기능의 구성 방법과 동작 원리를 확인했다.

## Master DNS Server 옵션 (10-8)

- TTL (Time-To-Live)
  - 위치: zone 파일의 첫 번째 줄에 작성
  - 의미: 다른 서버가 이 도메인 정보를 조회했을 때,
    그 결과를 자신의 캐시(Cache) 에 얼마 동안 저장할지를 정하는 설정
  - 기본값: TTL 값을 따로 지정하지 않으면 86400초 (24시간) 이 기본값으로 적용
  - W (주)  ,  D (일)  ,  H (시간)  ,  M (분)을 붙여서도 사용이 가능하다.
  - EX1) $TTL 3H    : 3시간 동안 캐시 유지
  - EX2) $TTL 1D    : 1일 동안 캐시 유지

- SOA (Start Of Authority)
  - 권한의 시작이며 zone 파일이 시작된다는 의미
  - @ 기호는 현재 도메인명을 뜻함 (예: soldesk.com)
  - 첫 번째 항목 : 주 DNS 서버(Primary Name Server) 이름
  - 두 번째 항목 : 관리자 이메일 주소 (@ 대신 .으로 표시)

- nameserver
  - Nameserver (NS Record)
  - 역할	: 이 도메인을 관리하는 DNS 서버의 이름을 지정
  - 주의	: 도메인 이름 끝에는 반드시 . 을 붙여야 한다.
  - 예시 	: @   IN  NS  ns.soldesk.com.
  - soldesk.com 도메인의 네임서버는 ns.soldesk.com 이다.

- serial
  - 역할: zone 파일이 변경될 때마다 증가시켜야 하는 버전 번호
  - Slave(보조) DNS 서버가 이 번호를 보고
  - Master(주) DNS의 데이터가 변경되었는지를 판단
  - 형식: 보통 날짜 형식으로 표현 (YYMMDDnn)
  - EX) 2023100201  (2023년 10월 2일 1번째 수정)

- refresh
  - 역할: Slave DNS 서버가 주기적으로 Master 서버에
  - 데이터 바뀌었는지 확인하러 가는 주기”
  - 단위: 시간 단위 (보통 1일)
  - EX) 1D   ; 하루마다 갱신 확인

- retry
  - 역할 : Slave 서버가 Master에 접근 실패 시
  - 얼마 후 다시 시도할지를 정함
  - 단위 : 시간 단위 (보통 1시간~1일)
  - EX) 1H   : 1시간 후 재시도

- expire
  - 역할 : Slave 서버가 Master에 계속 접근 실패할 경우
  - 데이터를 더 이상 신뢰하지 않고 삭제할 때까지의 시간
  - 단위 : 시간 단위 (보통 1주일)
  - EX) 1W : 1주일 지나면 만료

- minimum
  - 역할: DNS 응답에서 TTL이 명시되지 않은 레코드에 적용되는 최소 TTL 값 (캐시가 유지되는 최소 시간)
  - EX) 3H   : 최소 3시간 동안 캐시 유지다.

- A Record (Address Record)
  - 역할 : 도메인 이름 --> IPv4 주소 변환
  - 형식 : 호스트명   IN   A   IPv4주소
  - www  	IN  A  192.168.111.100
  - ftp  	IN  A  192.168.111.200

- AAAA Record
  - 역할: 도메인 이름 --> IPv6 주소 변환
  - IPv4의 A Record와 같은 역할을 IPv6용으로 수행
  - www  IN  AAAA  2001:43A1:9900:D3::871C:671

**정리**: Master DNS Server 옵션 (10-8) 섹션에서 다룬 명령어와 실습 예제를 통해 관련 기능의 구성 방법과 동작 원리를 확인했다.

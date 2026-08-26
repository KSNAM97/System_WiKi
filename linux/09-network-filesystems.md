# Linux 09 — 네트워크 파일시스템: NFS · Samba

## 목차

1. [NFS (Network File System) — 이론 + 실습 (9-1)](#nfs-network-file-system-—-이론-+-실습-9-1)
2. [NFS 연습 문제 (9-2)](#nfs-연습-문제-9-2)
3. [Samba Server (9-3)](#samba-server-9-3)

## NFS (Network File System) — 이론 + 실습 (9-1)

### NFS (Network File System)

- TCP/IP 네트워크를 통해 다른 컴퓨터의 파일 시스템 일부를 원격으로 마운트(mount)하여 로컬 디렉터리처럼 사용할 수 있게 해주는 기술로, Linux 간 파일 공유(NFS)뿐 아니라 Linux와 Windows 간 파일 공유(Samba)에도 널리 쓰인다.

- 개념
  - 서버(Server)의 디스크 일부(특정 디렉터리)를 클라이언트(Client)가 네트워크를 통해 마치 자신의 디렉터리처럼 접근할 수 있다.
  - 실제 데이터는 항상 NFS 서버의 HDD/SSD에 저장되며, 클라이언트는 파일을 원격 접근하는 것뿐이다.
  - 여러 클라이언트가 하나의 서버 디렉터리를 동시에 마운트하면, 모든 클라이언트가 동일한 데이터를 함께 공유해서 사용할 수 있다.
  - 즉, 로컬 디스크가 부족한 클라이언트가 서버의 디스크 공간을 빌려 쓰는 구조이며,
   메모리"를 공유하는 것이 아니라 디스크 공간(스토리지)"을 공유하는 것이다.

- 활용 예시
  - 내부 사용자의 HDD 용량이 부족해서 더 이상 파일 저장이 어려운 경우
   *서버에 큰 디스크를 장착해두고, 필요한 디렉터리를 NFS로 공유
   *클라이언트는 서버에 직접 로그인하지 않고도, 자신의 /project,  /data 같은 디렉터리로 마운트해서 사용 가능
  - 여러 개발 서버에서 같은 소스 코드 디렉터리를 공유하고 싶을 때
  - 백업 서버에 백업용 디렉터리를 만들어 각 클라이언트가 그 디렉터리에 백업 파일을 저장할 때

- 동작 원리 (NFS Server가 Client에게 자신의 디스크 일부를 제공)
  - NFS 기본 포트는 TCP 2049번이다.
  - NFSv3까지는 RPC 기반으로 여러 보조 포트를 사용하며, 이 포트 정보를 rpcbind(portmap)가 관리한다.
  - NFSv4는 주로 TCP 2049 하나로 통신하도록 설계되어 방화벽 설정이 좀 더 단순하다.

- 서버(Server) 측 역할

1) 공유할 디렉터리 설정
  - /etc/exports 파일에 "어떤 디렉터리를", "어떤 클라이언트에게", "어떤 권한으로" 	공유할지 설정한다.  
  - EX) /share/data  192.168.111.0/24(rw,sync)  

2) NFS 관련 서비스 활성화
  - RHEL/Rocky 계열 기준 : nfs-server, rpcbind 등의 서비스를 사용한다.  
   * systemctl enable nfs-server
   * systemctl start nfs-server
   * systemctl enable rpcbind
   * systemctl start rpcbind

3) 클라이언트 접근 권한과 IP 범위 지정
  - /etc/exports에서 특정 IP, 특정 대역, 특정 호스트만 접근하도록 제한할 수 있다.  
  - EX) 192.168.111.10(rw)  192.168.111.0/24(ro) 등  

4) 권한/퍼미션 주의
  - /etc/exports에 rw로 설정했다고 해서 자동으로 파일 시스템 퍼미션이 바뀌는 것은 아니다.  
  - 실제 디렉터리의 Linux 퍼미션(소유자, 그룹, chmod)이 쓰기 가능해야  클라이언트도 정상적으로 쓰기가 가능하다.  
  - exports: rw 옵션 설정 ,디렉터리 자체 퍼미션: 쓰기 가능한 권한 부여 이 두 가지가 모두 맞아야 쓰기가 된다.

- 클라이언트(Client) 측 역할
1) NFS 클라이언트 패키지 설치 (nfs-utils 등)

2) 서버가 공유한 디렉터리를 네트워크를 통해 마운트
  - mount -t nfs 서버IP:/공유디렉터리  /마운트포인트  

3) 마운트 후 사용
  - 마운트된 경로는 로컬 디렉터리처럼 ls, cp, mv, vi 등으로  읽기/쓰기 작업이 가능하다.  
  - 단, 실제 데이터는 모두 서버 디스크에 저장된다.

- RPC (Remote Procedure Call)와 NFS
  - RPC는 NFS 동작의 기반이 되는 통신 방식이다.
  - 원격 시스템(서버)의 함수를 로컬 프로그램이 함수 호출하듯 사용할 수 있게 해주는 기술이다.
  - NFS는 파일 열기, 읽기, 쓰기, 닫기 등의 요청을 RPC 호출 형태로 서버에 전달한다.
  - 다양한 RPC 서비스는 여러 포트를 사용할 수 있는데, 이 포트 정보를 관리하는 서비스가 portmap 또는 rpcbind이다.
  - rpcbind 서비스는 TCP/UDP 111번 포트를 사용하며, 클라이언트는 먼저 rpcbind에 질의해서 
   NFS 관련 서비스가 어느 포트를 사용하는지 확인한 뒤, 해당 포트로 실제 NFS 통신을 수행한다.

---

- Server-A가 NFS Server로 동작하며 Client-L이 NFS Clinet로 동작한다.

- NFS 기능을 사용하기위해서는 NFS와 RPC-bind가 설치되어야 한다.

```bash
[root@server-a ~]# dnf -y install nfs-utils
```

---

```bash
1-1) NFS를 사용하기위해서 방화벽 구성

[root@Server-A ~]# firewall-cmd  --add-port=2049/tcp
success
```

```bash
[root@Server-A ~]# firewall-cmd  --add-service=nfs
success
```

```bash
[root@Server-A ~]# firewall-cmd  --add-port=111/tcp
success
[root@Server-A ~]# firewall-cmd  --add-port=111/udp
success
```

```bash
[root@Server-A ~]# firewall-cmd  --add-service=rpc-bind
success
```

```bash
[root@Server-A ~]# firewall-cmd  --list-port		# 허용한 Port 확인
111/tcp 2049/tcp 111/udp
```

```bash
[root@Server-A ~]# firewall-cmd  --list-service		# 허용한 Service 확인
cockpit dhcpv6-client nfs rpc-bind ssh
```

```bash
[root@Server-A ~]# firewall-cmd  --list-all		# 모두 확인
```

```bash
[root@Server-A ~]# firewall-cmd  --reload		# 설정한 Port 또는 Service를 적용하기위해서는 방화벽을 reload해야 한다.
success
```

```bash
-방화벽 reload 후 다시 Port와 Service를 확인하게되면 위에서 허용한 Port와 Service가 확인되지않는다.

[root@Server-A ~]# firewall-cmd  --list-port

[root@Server-A ~]#
[root@Server-A ~]# firewall-cmd  --list-service
cockpit dhcpv6-client ssh
```

```bash
-Linux 방화벽 구성 후 설정을 적용하기위해서는 방화벽을 재부팅해야한다.
-Linux 방화벽은 재부팅되면 설정한 모든 설정이 초기화된다.
-Linux 방화벽 설정시 재부팅되어도 설정을 유지하기위해서는 '--permanent' 옵션을 사용해야한다.

	# NFS 허용
[root@Server-A ~]# firewall-cmd  --permanent  --add-port=2049/tcp
[root@Server-A ~]# firewall-cmd  --permanent  --add-service=nfs
```

```bash
	# RPC-BIND 허용
[root@Server-A ~]# firewall-cmd  --permanent  --add-port=111/tcp
[root@Server-A ~]# firewall-cmd  --permanent  --add-port=111/udp
[root@Server-A ~]# firewall-cmd  --permanent  --add-service=rpc-bind
```

```bash
	# 기존에 허용한 Port , Server를 삭제
[root@Server-A ~]# firewall-cmd  --permanent  --remove-port=1049/tcp	# 삭제
```

```bash
	# 설정 적용
[root@Server-A ~]# firewall-cmd  --reload
```

```bash
	# 설정 확인
[root@Server-A ~]# firewall-cmd  --list-port
111/tcp 2049/tcp 111/udp
```

```bash
	# 설정 확인
[root@Server-A ~]# firewall-cmd  --list-services
cockpit dhcpv6-client nfs rpc-bind ssh
```

```bash
	# NFS-Server가 동작하지 않는상태 (inactive)
[root@Server-A ~]# systemctl  status  nfs-server
○ nfs-server.service - NFS server and services
     Loaded: loaded (/usr/lib/systemd/system/nfs-server.service; disabled; preset: disabled)
     Active: inactive (dead)
       Docs: man:rpc.nfsd(8)
             man:exportfs(8)
```

```bash
	# NFS-Server 실행
[root@Server-A ~]# systemctl  start  nfs-server
```

```bash
	# NFS-Server가 동작하는상태 (active)
[root@Server-A ~]# systemctl  status  nfs-server
● nfs-server.service - NFS server and services
     Loaded: loaded (/usr/lib/systemd/system/nfs-server.service; disabled; preset: disabled)
     Active: active (exited) since Wed 2026-07-15 13:08:15 KST; 1s ago
       Docs: man:rpc.nfsd(8)
             man:exportfs(8)
    Process: 35024 ExecStartPre=/usr/sbin/exportfs -r (code=exited, status=0/SUCCESS)
    Process: 35025 ExecStart=/usr/sbin/rpc.nfsd (code=exited, status=0/SUCCESS)
    Process: 35046 ExecStart=/bin/sh -c if systemctl -q is-active gssproxy; then systemctl reloa>
   Main PID: 35046 (code=exited, status=0/SUCCESS)
        CPU: 11ms

 7월 15 13:08:15 Server-A systemd[1]: Starting NFS server and services...
 7월 15 13:08:15 Server-A systemd[1]: Finished NFS server and services.
```

```bash
	# 서버 재부팅후에도 해당 서비스를 유지하기위해서는 enable 명령어를 사용해야 한다.
[root@Server-A ~]# systemctl  enable  nfs-server
Created symlink /etc/systemd/system/multi-user.target.wants/nfs-server.service → /usr/lib/systemd/system/nfs-server.service.
```

```bash
[root@Server-A ~]# systemctl  status  nfs-server
● nfs-server.service - NFS server and services
     Loaded: loaded (/usr/lib/systemd/system/nfs-server.service; enabled; preset: disabled)
     Active: active (exited) since Wed 2026-07-15 13:08:15 KST; 2min 51s ago
       Docs: man:rpc.nfsd(8)
             man:exportfs(8)
   Main PID: 35046 (code=exited, status=0/SUCCESS)
        CPU: 11ms

 7월 15 13:08:15 Server-A systemd[1]: Starting NFS server and services...
 7월 15 13:08:15 Server-A systemd[1]: Finished NFS server and services.
```

```bash
EX1-3) 아래의 조건에 맞게 Server-A에서 NFS Server를 구성하시오
 # Server-A의 최상위 디렉터리에 '/NFSS' 디렉터리를 생성한 후 NFS Clinet에게 메모리를 할당하시오

[guest@localhost ~]$ sudo hostnamectl set-hostname Client-L.
[sudo] guest 암호:
```

```bash
[guest@localhost ~]$ exec bash
[guest@Client-L ~]$
```

```bash
	# Server-A의 IP 주소 확인
[root@Server-A ~]# ifconfig | grep 192
        inet 192.168.10.100  netmask 255.255.255.0  broadcast 192.168.10.255
```

```bash
	# Client-L의 IP 주소 확인
[guest@Client-L ~]$ ifconfig | grep 192
        inet 192.168.10.130  netmask 255.255.255.0  broadcast 192.168.10.255
```

```bash
[root@Server-A ~]# ls  -l  /etc/exports
-rw-r--r--. 1 root root 0  6월 23  2020 /etc/exports	# NFS Server에서 어떤 디렉터리를 어떤 클라이언트에게 공유할지를 설정하는 파일
```

```bash
	# NFS 설정
[root@Server-A ~]# vi /etc/exports
NFSS  192.168.10.130(rw,no_root_squash,sync)

:wq
```

ro 		= NFS를 읽기전용으로 적용
rw		= NFS를 읽기 , 쓰기가 가능하도록 적용
root_squash	= NFS Server 접속시 root 사용자를 무시하고 nfsnobaby 적용 (root 사용자를 일반 권한 사용자로 변환)
no_root_squash	= NFS Server 접속시 root 사용자를 root 권한으로 인정
all_squash 	= root를 포함한 모든 사용자를 nfsnobaby 적용 (익명사용자)
no_subtree_chech 	= 하위디렉터리를 검색하지 못함
sync		= 파일시스템 변경시 동기화

```bash
[root@Server-A ~]# exportfs  -v	# NFS 설정을 확인 (아직 적용하지 않았기때문에 아무것도 출력되지 않는다.)
```

```bash
[root@Server-A ~]# exportfs  -a	# /etc/exports에 설정된 내용을 적용
exportfs: Failed to stat /NFSS: No such file or directory
```

```bash
[root@Server-A ~]# mkdir  /NFSS	# 공유할 디렉터리 생성
```

```bash
[root@Server-A ~]# exportfs  -a	# /etc/exports에 설정된 내용을 적용
```

```bash
[root@Server-A ~]# exportfs  -v
/NFSS           192.168.10.130(sync,wdelay,hide,no_subtree_check,sec=sys,rw,secure,no_root_squash,no_all_squash)
```

```bash
[root@Server-A ~]# systemctl  restart  nfs-server	# NFS-Server Deamon 재시작
```

```bash
[root@Server-A ~]# showmount  -e
Export list for Server-A:
/NFSS 192.168.10.130
```

```bash
[root@Server-A ~]# showmount  -e  192.168.10.100
Export list for 192.168.10.100:
/NFSS 192.168.10.130
```

**EX1-4)** NFS Clinet인 Clinet-L은  Server-A로부터 메모리를 할당받아야한

```bash
[guest@Client-L ~]$ mount  -t  nfs  192.168.10.100:/NFSS  /NFSC	# /NFSC 디렉터리가 없기때문에 연결되지 않는다.
mount.nfs: mount point /NFSC does not exist
```

```bash
[guest@Client-L ~]$ mkdir  /NFSC
mkdir: `/NFSC' 디렉토리를 만들 수 없습니다: 허가 거부
```

```bash
[guest@Client-L ~]$ sudo mkdir  /NFSC
[sudo] guest 암호:1234
```

```bash
[guest@Client-L ~]$ sudo mount  -t  nfs  192.168.10.100:/NFSS  /NFSC
```

```bash
[guest@Client-L ~]$ df  -h
Filesystem            		Size      Used Avail Use% 	Mounted on
devtmpfs              		805M         0  805M	    0% 	/dev
tmpfs                 		838M         0  838M	    0% 	/dev/shm
tmpfs                 		335M    6.9M  329M	    3% 	/run
/dev/mapper/rl-root    	17G      5.7G   12G	    34% 	/
/dev/sda1            		1014M  479M  536M	    48% 	/boot
tmpfs                 		168M    108K  168M	    1% 	/run/user/0
tmpfs                 		168M     40K  168M	    1% 	/run/user/1000
192.168.10.100:/NFSS	16G      5.6G   11G	    35% 	/NFSC		# NFS로 마운트되어 Server-A의 디렉터리 /NFSS를 사용할 수 있다.
```

```bash
[guest@Client-L ~]$ mount | grep NFS
192.168.10.100:/NFSS on /NFSC type nfs4 (rw,relatime,vers=4.2,rsize=262144,wsize=262144,namlen=255,hard,proto=tcp,timeo=600,retrans=2,sec=sys,clientaddr=192.168.10.130,local_lock=none,addr=192.168.10.100)
```

```bash
[guest@Client-L ~]$ sudo reboot		# Client-L 재부팅
```

```bash
[guest@Client-L ~]$ mount | grep NFS		# 마운트가 확인되지 않는다.
```

```bash
[guest@Client-L ~]$ df  -h			# 마운트가 확인되지 않는다.
Filesystem            		Size      Used Avail Use% 	Mounted on
devtmpfs              		805M         0  805M	    0% 	/dev
tmpfs                 		838M         0  838M	    0% 	/dev/shm
tmpfs                 		335M    6.9M  329M	    3% 	/run
/dev/mapper/rl-root    	17G      5.7G   12G	    34% 	/
/dev/sda1            		1014M  479M  536M	    48% 	/boot
tmpfs                 		168M    108K  168M	    1% 	/run/user/0
tmpfs                 		168M     40K  168M	    1% 	/run/user/1000
```

```bash
[guest@Client-L ~]$ sudo mount  -t  nfs  192.168.10.100:/NFSS  /NFSC
```

```bash
[guest@Client-L ~]$ mount | grep NFS
192.168.10.100:/NFSS on /NFSC type nfs4 (rw,relatime,vers=4.2,rsize=262144,wsize=262144,namlen=255,hard,proto=tcp,timeo=600,retrans=2,sec=sys,clientaddr=192.168.10.130,local_lock=none,addr=192.168.10.100)
```

```bash
[guest@Client-L ~]$ df  -h
Filesystem            		Size      Used Avail Use% 	Mounted on
devtmpfs              		805M         0  805M	    0% 	/dev
tmpfs                 		838M         0  838M	    0% 	/dev/shm
tmpfs                 		335M    6.9M  329M	    3% 	/run
/dev/mapper/rl-root    	17G      5.7G   12G	    34% 	/
/dev/sda1            		1014M  479M  536M	    48% 	/boot
tmpfs                 		168M    108K  168M	    1% 	/run/user/0
tmpfs                 		168M     40K  168M	    1% 	/run/user/1000
192.168.10.100:/NFSS	16G      5.6G   11G	    35% 	/NFSC		# NFS로 마운트되어 Server-A의 디렉터리 /NFSS를 사용할 수 있다.
```

```bash
[guest@Client-L ~]$ sudo  vi  /etc/fstab
#
# /etc/fstab
# Created by anaconda on Thu Jul  2 08:04:34 2026
#
# Accessible filesystems, by reference, are maintained under '/dev/disk/'.
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info.
#
# After editing this file, run 'systemctl daemon-reload' to update systemd
# units generated from this file.
#
/dev/mapper/rl-root     			/	xfs     	defaults	0 0
UUID=26ef2af1-0f9c-42ff-9c34-3f8d70154096	/boot	xfs     	defaults	0 0
/dev/mapper/rl-swap     			none	swap	defaults	0 0
192.168.10.100:/NFSS    			/NFSC	nfs	defaults	0 0	# 오토 마운트 설정

:wq
```

---

**정리**: NFS (Network File System) — 이론 + 실습 (9-1) 섹션에서 다룬 명령어와 실습 예제를 통해 관련 기능의 구성 방법과 동작 원리를 확인했다.

## NFS 연습 문제 (9-2)

**EX1)** 아래의 조건에 맞게 Server-A를 설정하시오
  - 10G용량의 HDD를 추가해야한다.  (SCSI)
  - 4G의 용량의 Patition을 구성
  - 2G의 용량의 Patition을 구성
  - 2G의 용량의 Patition을 구성
  - 1G의 용량의 Patition을 구성
  - 1G의 용량의 Patition을 구성

```bash
[root@server-a ~]# lsblk
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda   	  8:0	0  100G  	0 disk
├─sda1	  8:1    	0   90G  	0 part  /
└─sda2	  8:2    	0   1G  	0 part [SWAP]
sdb      	  8:16   	0  100G  	0 disk
├─sdb1	  8:17   	0   4G  	0 part
├─sdb2	  8:18   	0   2G  	0 part
├─sdb3	  8:19   	0   2G  	0 part
├─sdb4	  8:2   	0     1K 	0 part
├─sdb5	  8:21  	0   1G  	0 part
└─sdb6	  8:22   	0   1G  	0 part
sdc      	  8:32   	0  100G  	0 disk
sr0      	 11:0    	1  9.5G  	0 rom  /run/media/root/CentOS 7 x86_64
```

```bash
EX2-1) 아래의 조건에 맞게 NFS를 구성하시오
 # 4G 메모리를 사용하기위해서 ext4형식으로 format해야한다.
 # 2G 메모리를 사용하기위해서 ext4형식으로 format해야한다.

[root@Server-A ~]# mkfs.ext4  /dev/sdb1
mke2fs 1.46.5 (30-Dec-2021)
Creating filesystem with 1048576 4k blocks and 262144 inodes
Filesystem UUID: 195d77b3-fe57-4b3b-94bd-8dbf95d779fe
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376, 294912, 819200, 884736

Allocating group tables: done
Writing inode tables: done
Creating journal (16384 blocks): done
Writing superblocks and filesystem accounting information: done
```

```bash
[root@Server-A ~]# mkfs.ext4  /dev/sdb2
mke2fs 1.46.5 (30-Dec-2021)
Creating filesystem with 524288 4k blocks and 131072 inodes
Filesystem UUID: 957bc104-9d4b-4c1e-a735-6395bacf954a
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376, 294912

Allocating group tables: done
Writing inode tables: done
Creating journal (16384 blocks): done
Writing superblocks and filesystem accounting information: done
```

```bash
[root@Server-A ~]# lsblk -f
NAME FSTYPE FSVER LABEL	UUID                                 		FSAVAIL FSUSE% 	MOUNTPOINTS
sda
├─sda1
│    swap   1                     		520bc18c-2b64-4df1-85e0-d126908ba6dd                	[SWAP]
└─sda2
     xfs                          		73bc277c-741d-4122-9c58-59ccd1889709   10.4G    35% 	/
sdb
├─sdb1
│    ext4   1.0                   		195d77b3-fe57-4b3b-94bd-8dbf95d779fe
├─sdb2
│    ext4   1.0                   		957bc104-9d4b-4c1e-a735-6395bacf954a
├─sdb3
│
├─sdb5
│
├─sdb6
│
└─sdb7
```

**EX2-2)** 아래의 조건에 맞게 NFS를 구성하시오
  - NFS , RPC-bind의 설치를 확인하고 미 설치시 해당 Package를 설치해야한다.
  - Server-B는 HDD 용량을 확장하기위해서 Server-A의 메모리 4G를 사용해야한다.
  - Clinet-L은 HDD 용량을 확장하기위해서 Server-A의 메모리 2G를 사용해야한다.

```bash
[root@Server-A ~]# rpm -qa | grep nfs
libnfsidmap-2.5.4-42.el9.x86_64
sssd-nfs-idmap-2.9.8-4.el9_8.x86_64
nfs-utils-2.5.4-42.el9.x86_64
nfsv4-client-utils-2.5.4-42.el9.x86_64
nfs-utils-coreos-2.5.4-42.el9.x86_64
nfs4-acl-tools-0.4.2-3.el9.x86_64
```

```bash
[root@Server-A ~]# rpm -qa | grep rpc
xmlrpc-c-1.51.0-16.el9_0.x86_64
xmlrpc-c-client-1.51.0-16.el9_0.x86_64
libtirpc-1.3.3-9.el9.x86_64
rpcbind-1.2.6-7.el9.x86_64
```

```bash
[root@Server-A ~]# firewall-cmd  --permanent  --add-port=2049/tcp
[root@Server-A ~]# firewall-cmd  --permanent  --add-service=nfs

[root@Server-A ~]# firewall-cmd  --permanent  --add-port=111/tcp
[root@Server-A ~]# firewall-cmd  --permanent  --add-port=111/udp
[root@Server-A ~]# firewall-cmd  --permanent  --add-service=rpc-bind

[root@Server-A ~]# firewall-cmd  --reload
```

```bash
[root@Server-A ~]# firewall-cmd  --list-port
[root@Server-A ~]# firewall-cmd  --list-service

[root@Server-A ~]# firewall-cmd  --list-all
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: ens160
  sources:
  services: cockpit dhcpv6-client nfs rpc-bind ssh
  ports: 111/tcp 111/udp 2049/tcp
  protocols:
  forward: yes
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:
```

```bash
[root@Server-A ~]# mkdir  /NFS_LC		# Linux-C가 사용할 디렉터리

[root@Server-A ~]# mkdir  /NFS_SB		# Server-B가 사용할 디렉터리
```

```bash
[root@Server-A ~]# mount  /dev/sdb1  /NFS_SB/
```

```bash
[root@Server-A ~]# mount  /dev/sdb2  /NFS_LC/
```

```bash
[root@Server-A ~]# df  -h
Filesystem	Size    Used Avail Use%  Mounted on
devtmpfs        	807M       0  807M	 0%  /dev
tmpfs           	838M       0  838M	 0%  /dev/shm
tmpfs           	335M  6.9M  329M	 3%  /run
/dev/sda2        	16G    5.6G   11G   35%  /
tmpfs           	168M    56K  168M	 1%  /run/user/42
tmpfs           	168M    40K  168M	 1%  /run/user/0
/dev/sdb1       	3.9G     24K  3.7G   1%  /NFS_SB
/dev/sdb2      	 2.0G   8.0K  1.8G   1%  /NFS_LC
```

```bash
[root@Server-A ~]# vi  /etc/exports
/NFS_LC 192.168.10.130(rw,no_root_squash,sync)
/NFS_SB 192.168.10.200(rw,no_root_squash,sync)

: wq
```

```bash
[root@Server-A ~]# exportfs  -ra

-r	: exports에 적용한 파일을 다시 읽기
-a	: exports에 적용한 파일을 적용
```

```bash
[root@Server-A ~]# exportfs  -v
/NFS_LC         192.168.10.130(sync,wdelay,hide,no_subtree_check,sec=sys,rw,secure,no_root_squash,no_all_squash)
/NFS_SB         192.168.10.200(sync,wdelay,hide,no_subtree_check,sec=sys,rw,secure,no_root_squash,no_all_squash)
```

```bash
[root@Server-A ~]# systemctl  restart  nfs-server
```

```bash
[root@Server-A ~]# showmount -e
Export list for Server-A:
/NFS_SB 192.168.10.200
/NFS_LC 192.168.10.130
```

---

#### Server-B

**EX3-1)** 아래의 조건에 맞게 설정하시오
  - Server-B는 Server-A로부터 2GB의 메모리를 할당받아야한다.
  - Client-L은 Server-A로부터 4GB의 메모리를 할당받아야한다.

```bash
[root@server-b ~]# dnf  -y  install  net-tools	<---- IP 주소등을 확인하기위한 Package 설치
[root@server-b ~]# ifconfig
```

```bash
EX3-2) 아래의 조건에 맞게 설정하시오
 # Server-B는 Server-A로부터 4GB의 메모리를 할당받은후 Server-A가 재부팅되어도 바로 사용가능해야한다.
 # Client-L은 Server-A로부터 2GB의 메모리를 할당받은후 Server-A가 재부팅되어도 바로 사용가능해야한다.

	# Server-B 설정

[root@localhost ~]# dnf  install  -y  nfs-utils
```

```bash
[root@localhost ~]# mkdir  /NFS_SBC
```

```bash
[root@localhost ~]# mount  -t  nfs  192.168.10.100:/NFS_SB  /NFS_SBC
```

```bash
[root@localhost ~]# df -h
Filesystem              	Size  Used Avail Use% Mounted on
devtmpfs                	817M     0  817M   0% /dev
tmpfs                   		838M     0  838M   0% /dev/shm
tmpfs                   		335M  4.9M  331M   2% /run
/dev/sda2                	16G  2.2G   14G  14% /
tmpfs                   		168M     0  168M   0% /run/user/0
192.168.10.100:/NFS_SB  	3.9G     0  3.7G   0%  /NFS_SBC
```

**EX3-3)** 아래의 조건에 맞게 설정하시오
  - Server-B는 Server-A로부터 2GB의 메모리를 할당받아야하며 재부팅 되어도 mount가 유지되어야한다.
  - Client-L은 Server-A로부터 4GB의 메모리를 할당받아야하며 재부팅 되어도 mount가 유지되어야한다.

```bash
[guest@Client-L ~]$ sudo mkdir  /NFS_CLINET
```

```bash
[guest@Client-L ~]$ sudo mount  -t  nfs  192.168.10.100:/NFS_LC  /NFS_CLINET
```

```bash
[guest@Client-L ~]$ df  -h
Filesystem              	Size  Used Avail Use% Mounted on
devtmpfs                	805M     0  805M   0% /dev
tmpfs                   		838M     0  838M   0% /dev/shm
tmpfs                   		335M  5.4M  330M   2% /run
/dev/mapper/rl-root      	17G  5.5G   12G  33% /
/dev/sda1              		1014M  517M  498M  51% /boot
tmpfs                   		168M   56K  168M   1% /run/user/42
tmpfs                   		168M   40K  168M   1% /run/user/1000
192.168.10.100:/NFS_LC	2.0G     0  1.8G   0% /NFS_CLINET
```

```bash
[guest@Client-L ~]$ sudo cp  -r  /etc/a*  /NFS_CLINET/

[guest@Client-L ~]$ ls  -l  /NFS_CLINET/
합계 52
drwxr-xr-x 3 root root 4096  7월 15 17:12 accountsservice
-rw-r--r-- 1 root root   16  7월 15 17:12 adjtime
-rw-r--r-- 1 root root 1529  7월 15 17:12 aliases
drwxr-xr-x 3 root root 4096  7월 15 17:12 alsa
drwxr-xr-x 2 root root 4096  7월 15 17:12 alternatives
-rw-r--r-- 1 root root  541  7월 15 17:12 anacrontab
-rw-r--r-- 1 root root  269  7월 15 17:12 anthy-unicode.conf
-rw-r--r-- 1 root root  833  7월 15 17:12 appstream.conf
-rw-r--r-- 1 root root   55  7월 15 17:12 asound.conf
-rw-r--r-- 1 root root    1  7월 15 17:12 at.deny
drwxr-x--- 4 root root 4096  7월 15 17:12 audit
drwxr-xr-x 3 root root 4096  7월 15 17:12 authselect
drwxr-xr-x 4 root root 4096  7월 15 17:12 avahi
```

---

**정리**: NFS 연습 문제 (9-2) 섹션에서 다룬 명령어와 실습 예제를 통해 관련 기능의 구성 방법과 동작 원리를 확인했다.

## Samba Server (9-3)

#### Samba

- NFS는 주로 Linux/Unix 시스템 사이에서 파일과 디렉터리를 공유하기 위해 사용하는 네트워크 파일 공유 기술이다.

- Samba는 Linux/Unix 계열 운영체제에서 Microsoft의 SMB 프로토콜을 사용할 수 있도록 구현한 소프트웨어 모음이다.

- Samba를 사용하면 Linux와 Windows가 네트워크를 통해 파일, 디렉터리, 프린터 등의 자원을 공유할 수 있다.

- NFS
  - Linux	<--->  Linux
  - Unix 	<--->  Linux

- Samba/SMB
  - Linux 	<--->  Windows
  - Linux 	<--->  Linux
  - Windows <---> Windows

- Samba는 Windows 전용 프로토콜을 Linux에서 사용할 수 있도록 구현한 서비스이므로, 단순히 Linux와 Windows 사이에서만 사용하는 것은 아니다.

- Linux 시스템끼리도 Samba를 사용하여 파일을 공유할 수 있지만, Linux 환경에서는 일반적으로 NFS를 더 많이 사용한다.

---

#### Windows Samba Server  (SMB)

- Windows는 Samba 데몬(smbd)을 가지고 있지 않기 때문에 Linux처럼 Samba Server를 직접 구성하는 것은 불가능하다.
 그러나 Windows의 공유 폴더는 SMB(Server Message Block) 프로토콜을 사용하기 때문에
 Linux에서는 이 공유 폴더를 Samba 방식으로 접근할 수 있다.
 결국 Windows가 SMB 서버 역할을 수행하게 된다.

- SMB 관련 용어
  - SMB(Server Message Block)   	: Windows의 기본 파일 공유 프로토콜
  - CIFS(Common Internet File System)	: SMB1 기반 확장 명칭. 현재는 SMB2/SMB3가 주로 사용
  - CIFS 이후에는 라우터를 넘어 다양한 네트워크에서도 파일 공유가 가능해졌다.

	1) Windows의 C: 드라이브에 공유 폴더 생성 (일기_쓰기 권한)

	2) Windows에서 사용할 계정 생성

Microsoft Windows [Version 10.0.19045.2006]
(c) Microsoft Corporation. All rights reserved.

C:\Users\aaa> net user  root  1234  /add
시스템 오류 5이(가) 생겼습니다.

액세스가 거부되었습니다.

C:\Users\aaa>

#### 관리자 권한으로 실행

C:\Windows\system32> net user  root  1234  /add

명령을 잘 실행했습니다.

C:\Windows\system32> net user

\\DESKTOP-FOUO854에 대한 사용자 계정

- --

aaa         	Administrator    	DefaultAccount
Guest                	root                   	WDAGUtilityAccount
명령을 잘 실행했습니다.

C:\Windows\system32>ipconfig

Windows IP 구성

이더넷 어댑터 Ethernet0:

   연결별 DNS 접미사. . . . : localdomain
   링크-로컬 IPv6 주소 . . . . : fe80::5ded:f186:69e1:2224%6
   IPv4 주소 . . . . . . . . . : 192.168.10.131		# SAMBA Server IP address
   서브넷 마스크 . . . . . . . : 255.255.255.0
   기본 게이트웨이 . . . . . . : 192.168.10.2

	3) Linux에서 SAMBA Package 설치

```bash
[root@Server-A /]# dnf  install  -y  samba-client  samba-common  samba-winbind
```

```bash
[root@Server-A /]# rpm  -qa  | grep samba
samba-common-4.23.5-10.el9_8.noarch
samba-client-libs-4.23.5-10.el9_8.x86_64
samba-common-libs-4.23.5-10.el9_8.x86_64
samba-libs-4.23.5-10.el9_8.x86_64
samba-dcerpc-4.23.5-10.el9_8.x86_64
samba-winbind-modules-4.23.5-10.el9_8.x86_64
samba-ldb-ldap-modules-4.23.5-10.el9_8.x86_64
samba-common-tools-4.23.5-10.el9_8.x86_64
samba-winbind-4.23.5-10.el9_8.x86_64
samba-client-4.23.5-10.el9_8.x86_64
```

	4) Linux(samba client)에서 Samba Server(Window)로 mount

```bash
[root@Server-A /]# smbclient  -L  192.168.10.131
Password for [SAMBA\root]:1234

        Sharename   	Type      Comment
        ---------	----      -------
        ADMIN$     	Disk      원격 관리
        C$           	Disk      기본 공유
        IPC$        	IPC       원격 IPC
        Users        	Disk
        winShare   	Disk
SMB1 disabled -- no workgroup available
```

```bash
[root@Server-A /]# mount  -t  cifs  //192.168.10.131/winShare  /smbClient
Password for root@//192.168.10.131/winShare:1234
```

```bash
[root@Server-A /]# mount  | grep  smbClient
//192.168.10.131/winShare on /smbClient type cifs 
(rw,relatime,vers=3.1.1,cache=strict,upcall_target=app,username=root,uid=0,noforceuid,gid=0,
noforcegid,addr=192.168.10.131,file_mode=0755,dir_mode=0755,soft,nounix,serverino,mapposix,
reparse=nfs,nativesocket,symlink=native,rsize=4194304,wsize=4194304,
bsize=1048576,echo_interval=60,actimeo=1,closetimeo=1)
```

```bash
[root@Server-A /]# cp -r /etc/a*  /smbClient/
```

```bash
[root@Server-A /]# ls  -l  /smbClient/
합계 11
drwxr-xr-x 2 root root    0  7월 15 17:58 accountsservice
-rwxr-xr-x 1 root root   16  7월 15 17:58 adjtime
-rwxr-xr-x 1 root root 1529  7월 15 17:58 aliases
drwxr-xr-x 2 root root    0  7월 15 17:58 alsa
drwxr-xr-x 2 root root    0  7월 15 17:58 alternatives
-rwxr-xr-x 1 root root  541  7월 15 17:58 anacrontab
-rwxr-xr-x 1 root root  269  7월 15 17:58 anthy-unicode.conf
-rwxr-xr-x 1 root root  833  7월 15 17:58 appstream.conf
-rwxr-xr-x 1 root root   55  7월 15 17:58 asound.conf
-rwxr-xr-x 1 root root    1  7월 15 17:58 at.deny
drwxr-xr-x 2 root root    0  7월 15 17:58 audit
drwxr-xr-x 2 root root    0  7월 15 17:58 authselect
drwxr-xr-x 2 root root    0  7월 15 17:58 avahi
```

- mount는 서버 재부팅시 해제되기때문에 오토마운트를 사용해서 재부팅시에도 마운트가 유지되도록 설정행야 한다.

```bash
[root@Server-A /]# vi /etc/fstab
#
# /etc/fstab
# Created by anaconda on Thu Jul  2 03:51:29 2026
#
# Accessible filesystems, by reference, are maintained under '/dev/disk/'.
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info.
#
# After editing this file, run 'systemctl daemon-reload' to update systemd
# units generated from this file.
#
UUID=73bc277c-741d-4122-9c58-59ccd1889709       /               	xfs     	defaults        0 0
UUID=520bc18c-2b64-4df1-85e0-d126908ba6dd       none            	swap    	defaults        0 0
//192.168.10.131/winShare			/smbClient	cifs	username=root,password=1234,iocharset=utf8,_netdev 0 0

:wq

# username	: Samba Server의 계정
# password	: Samba Server의 비밀번호
# iocharset=utf8	: 한글을 표현하기위한 UTF-8
# _netdev		: 네트워크 디바이스이므로 부팅시 준비
```

```bash
[root@Server-A ~]# reboot
```

---

#### Linux Samba Server

- Samba에서 사용하는 주요 데몬이며 크게 두 가지 데몬으로 동작한다:

1) SMB 데몬 (smbd)
  - SMB 프로토콜을 사용하여 파일/폴더 공유 기능을 수행하는 핵심 데몬
  - 윈도우의 탐색기에서 공유 폴더를 열 때 실제로 응답하는 부분
  - 인증(로그인), 파일 읽기/쓰기, 파일 잠금(lock) 기능 처리

- 사용하는 포트
  - TCP 445 (Direct SMB, 최신 윈도우 기본)
  - TCP 139 (NetBIOS 기반 SMB, 구 버전 호환)

2) NMB 데몬 (nmbd)
  - 윈도우 네트워크 환경에서 컴퓨터 이름을 검색할 수 있게 하는 데몬
  - (예: 네트워크에서 "SERVER-A" 자동 탐색)
  - NetBIOS 기반 이름 해석 기능

- 사용하는 포트
  - UDP 137 (NetBIOS Name Service)
  - UDP 138 (NetBIOS Datagram Service)

```bash
[root@Server-A ~]# dnf  install  -y  samba*

[root@Server-A ~]# mkdir  /SHARE		# 공유 디렉터리
```

```bash
[root@Server-A ~]# useradd  samba		# samba 접속시 인증을 위해서 사용할 계정 (접속용 X)
[root@Server-A ~]# groupadd  SG		# 공유 디렉터리와 그릅을 연동시 사용할 그룹
[root@Server-A ~]# usermod  -G  SG  samba
```

```bash
[root@Server-A ~]# chmod  777  /SHARE

[root@Server-A ~]# ls  -l / | grep  SHARE
drwxrwxrwx    2 root root    6  7월 16 09:47 SHARE
```

```bash
[root@Server-A ~]# passwd  samba		# 사용자 계정 비밀번호 설정

[root@Server-A ~]# smbpasswd  -a  samba	# Samba 서버로 접속시 인증을 위한 비밀번호
New SMB password:1234
Retype new SMB password:1234
Added user samba.

# a- : Linux에 존재하는 samba 사용자를 Samba 사용자 데이터베이스에 추가하고 Samba 접속용 비밀번호를 설정
```

```bash
[root@Server-A ~]# firewall-cmd  --permanent  --add-service=samba
success
```

```bash
[root@Server-A ~]# firewall-cmd  --permanent  --add-service=samba
success

[root@Server-A ~]# firewall-cmd  --permanent  --add-port=445/tcp
success

[root@Server-A ~]# firewall-cmd  --permanent  --add-port=139/tcp
success

[root@Server-A ~]# firewall-cmd  --permanent  --add-port=137/udp
success

[root@Server-A ~]# firewall-cmd  --permanent  --add-port=138/udp
success

[root@Server-A ~]# firewall-cmd  --reload
success
```

```bash
[root@Server-A ~]# firewall-cmd  --list-port
111/tcp 139/tcp 445/tcp 2049/tcp 111/udp 137/udp 138/udp
```

```bash
[root@Server-A ~]# firewall-cmd  --list-services
cockpit dhcpv6-client nfs rpc-bind samba ssh
```

```bash
[root@Server-A ~]# firewall-cmd  --list-all
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: ens160
  sources:
  services: cockpit dhcpv6-client nfs rpc-bind samba ssh
  ports: 111/tcp 111/udp 2049/tcp 445/tcp 139/tcp 137/udp 138/udp
  protocols:
  forward: yes
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:
```

```bash
	# samba 설정

[root@Server-A ~]# ls -l  /etc  | grep  samba
drwxr-xr-x.  2 root root        84  7월 16 09:46 samba
```

```bash
[root@Server-A ~]# vi  /etc/samba/smb.conf
~~~~~~~~ 중간 생략 ~~~~~~~~
     47 [Share]
     48         path = /SHARE     	# 공유 디렉터리 경로(실제 경로)
     49         writable = yes      	# 쓰기 권한
     50         browseable = yes    	# Windows 파일 탐색기에서 공유 폴더가 검색되거나 목록에 표시되도록 설정
     51         guest ok = no       	# 계정과 비밀번호 없이 접속하는 익명 사용자의 접근을 차단
     52         valid users = @SG    	# SG 그룹에 속한 사용자만 해당 공유 폴더에 접속할 수 있도록 제한
     53         force group = SG     	# 해당 디렉터리에서 생성되는 파일,디렉터리는 SG에 포함
     54         create mask = 0666   	# 새 파일 생성시 권한
     55         directory mask = 0777	# 새 디렉터리 생성시 권한
     56 :wq
```

```bash
[root@Server-A ~]# systemctl  start  smb

[root@Server-A ~]# systemctl  enable  smb
Created symlink /etc/systemd/system/multi-user.target.wants/smb.service → /usr/lib/systemd/system/smb.service.

[root@Server-A ~]# systemctl  status  smb
● smb.service - Samba SMB Daemon
     Loaded: loaded (/usr/lib/systemd/system/smb.service; enabled; preset: disabled)
     Active: active (running) since Thu 2026-07-16 10:07:18 KST; 19s ago
       Docs: man:smbd(8)
             man:samba(7)
             man:smb.conf(5)
   Main PID: 3431 (smbd)
     Status: "smbd: ready to serve connections..."
      Tasks: 3 (limit: 10321)
     Memory: 10.0M (peak: 10.3M)
        CPU: 50ms
     CGroup: /system.slice/smb.service
             ├─3431 /usr/sbin/smbd --foreground --no-process-group
             ├─3433 /usr/sbin/smbd --foreground --no-process-group
             └─3434 /usr/sbin/smbd --foreground --no-process-group

 7월 16 10:07:18 Server-A systemd[1]: Starting Samba SMB Daemon...
 7월 16 10:07:18 Server-A systemd[1]: Started Samba SMB Daemon.
```

```bash
[root@Server-A ~]# systemctl  start  nmb

[root@Server-A ~]# systemctl  enable  nmb
Created symlink /etc/systemd/system/multi-user.target.wants/nmb.service → /usr/lib/systemd/system/nmb.service.

[root@Server-A ~]# systemctl  status  nmb
● nmb.service - Samba NMB Daemon
     Loaded: loaded (/usr/lib/systemd/system/nmb.service; enabled; preset: disabled)
     Active: active (running) since Thu 2026-07-16 10:08:24 KST; 8s ago
       Docs: man:nmbd(8)
             man:samba(7)
             man:smb.conf(5)
   Main PID: 3477 (nmbd)
     Status: "nmbd: ready to serve connections..."
      Tasks: 1 (limit: 10321)
     Memory: 3.0M (peak: 3.3M)
        CPU: 23ms
     CGroup: /system.slice/nmb.service
             └─3477 /usr/sbin/nmbd --foreground --no-process-group

 7월 16 10:08:24 Server-A systemd[1]: Starting Samba NMB Daemon...
 7월 16 10:08:24 Server-A systemd[1]: Started Samba NMB Daemon.
```

- Windows 파일탐색기  --->  내PC  --->  컴퓨터  --->  네트워크 드라이브연결  --->  드라이브 선택
폴더 : \\192.168.10.100\SHARE
체크박스 모두 체크
마침

**EX2)** 아래의 조건에 맞게 설정하시오
  - 현재 Linux 사용자가 공유 디렉터리인 '/SHARE' 디렉터리에 파일을 생성한 하게되면
    Windows 사용자가 해당 파일이나 디렉터리를 삭제할 수있다.
  - 공유 디렉터리인 '/SHARE' 디렉터리에 파일을 생성한 소유주만 변경 , 삭제가 가능하도록 설정하시오
  - 단 읽기 및 복사는 가능해야한다.

```bash
[root@Server-A ~]# chmod  1777  /SHARE/
```

```bash
[root@Server-A ~]# ls  -l  / | grep  SHARE
drwxrwxrwt    6 root root 4096  7월 16 10:14 SHARE
```

**정리**: Samba Server (9-3) 섹션에서 다룬 명령어와 실습 예제를 통해 관련 기능의 구성 방법과 동작 원리를 확인했다.

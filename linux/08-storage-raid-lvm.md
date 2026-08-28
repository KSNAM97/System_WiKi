# Linux 08 — 스토리지 심화: Disk Quota · RAID · LVM

## Disk Quota (8-1)

### Disk Quota

디스크 쿼터, RAID, LVM은 사용자별 저장 공간 제한, 디스크 이중화나 성능 향상, 유연한 볼륨 확장 등 스토리지를 안정적으로 운영하기 위해 실무에서 함께 활용되는 기술이다.

- 리눅스 시스템은 기본적으로 사용자의 디스크 사용량(저장 공간)을 제한하지 않는다.
 따라서, 전체 디스크 용량이 1TB라면 특정 사용자가 1TB 전부를 차지할 수도 있다.

- 최상위 디렉터리에있는 홈디렉터리안에 사용자 계정을 생성하게되면
  특정 사용자가 운영체제가 설치된 HDD의 모든 메모리를 사용할수 있게된다.

- 운영체제가 설치된 HDD의 메모리가 부족하게되면 System이 down되거나 특정 Process가 down될 수 있다.
	
- 사용자 계정은 OS가 설치된 Patition에 생성하지 않고 별도의 Patition을 분할하거나  다른 HDD에 생성하는 것을 권장한다.

- Quota기능을 사용하여 개별 사용자 또는 Group별로 메모리를 제한하거나 생성할 파일의 개수를 제한할 수 있다.

- Disk Quota 기능
  - Quota(쿼터) 는 디스크 자원(저장소)을 관리하기 위한 기능이다.
  - Quota를 사용시
    *사용자별 디스크 용량 제한 가능 	(예: 사용자 A는 최대 10GB까지만 사용)
    *그룹별 디스크 용량 제한 가능 	(예: 개발팀 그룹은 100GB까지만 사용)
    *파일 개수 제한 가능		(예: 최대 10,000개의 파일만 생성 가능)

- EX
  - 대학 서버에서 학생 계정마다 2GB 제한을 걸어 다른 학생들의 공간 침해 방지
  - 기업 서버에서 특정 프로젝트 그룹에만 500GB 할당
  - 웹호스팅 서버에서 사용자별 저장 공간을 제한해 서비스 품질 유지

---

**EX1-1)** 아래의 조건에 맞게 Patition을 구성하시오
  - 'sdb' HDD를 50G용량으로 Patition을 구성하시오
  - 'sdb' HDD를 50G용량으로 Patition을 구성하시오 (남은 용량을 모두 할당)

```bash
[root@Server-A ~]# umount /dev/sdb1
[root@Server-A ~]# umount /dev/sdb5
[root@Server-A ~]# umount /dev/sdb6
[root@Server-A ~]# umount /dev/sdb7
[root@Server-A ~]# umount /dev/sdb8
```

```bash
[root@Server-A ~]# fdisk  /dev/sdb

Welcome to fdisk (util-linux 2.37.4).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

This disk is currently in use - repartitioning is probably a bad idea.
It's recommended to umount all file systems, and swapoff all swap
partitions on this disk.
```

```
Command (m for help): d
Partition number (1,2,5-8, default 8):

Partition 8 has been deleted.

Command (m for help): d
Partition number (1,2,5-7, default 7):

Partition 7 has been deleted.

Command (m for help): d
Partition number (1,2,5,6, default 6):

Partition 6 has been deleted.

Command (m for help): d
Partition number (1,2,5, default 5):

Partition 5 has been deleted.

Command (m for help): d
Partition number (1,2, default 2):

Command (m for help): w
```

```bash
[root@Server-A ~]# fdisk  /dev/sdb

Welcome to fdisk (util-linux 2.37.4).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.
```

```
Command (m for help): n
Partition type
   p   primary (0 primary, 0 extended, 4 free)
   e   extended (container for logical partitions)
Select (default p): p
Partition number (1-4, default 1): 1
First sector (2048-209715199, default 2048):
Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-209715199, default 209715199): +50G

Created a new partition 1 of type 'Linux' and of size 50 GiB.
Partition #1 contains a xfs signature.

Do you want to remove the signature? [Y]es/[N]o: y

The signature will be removed by a write command.

Command (m for help): n
Partition type
   p   primary (1 primary, 0 extended, 3 free)
   e   extended (container for logical partitions)
Select (default p): p
Partition number (2-4, default 2): 2
First sector (104859648-209715199, default 104859648):
Last sector, +/-sectors or +/-size{K,M,G,T,P} (104859648-209715199, default 209715199):

Created a new partition 2 of type 'Linux' and of size 50 GiB.

Command (m for help): p
Disk /dev/sdb: 100 GiB, 107374182400 bytes, 209715200 sectors
Disk model: VMware Virtual S
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x266a5e81

Device     Boot     Start       End   Sectors Size Id Type
/dev/sdb1              2048 104859647 104857600  50G 83 Linux
/dev/sdb2       104859648 209715199 104855552  50G 83 Linux

Filesystem/RAID signature on partition 1 will be wiped.

Command (m for help): w

The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.
```

```bash
[root@Server-A ~]# lsblk
NAME   MAJ:MIN RM  SIZE RO TYPE	MOUNTPOINTS
sda      	  8:0    	0   20G  	0 disk
├─sda1	  8:1    	0    4G  	0 part 	[SWAP]
└─sda2	  8:2    	0   16G  	0 part 	/
sdb      	  8:16   	0  100G  	0 disk
├─sdb1	  8:17   	0   50G  	0 part
└─sdb2	  8:18   	0   50G  	0 part
sdc      	  8:32   	0  100G  	0 disk
sdd      	  8:48   	0  100G  	0 disk
sde      	  8:64   	0  100G  	0 disk
```

**EX1-2)** 아래의 조건에 맞게 File system을 구성하시오
  - 첫번째 Patition의 File system을 ext4로 구성
  - 두번째 Patition의 File system을 ext4로 구성

```bash
[root@Server-A ~]# mkfs.ext4  /dev/sdb1
mke2fs 1.46.5 (30-Dec-2021)
Creating filesystem with 13107200 4k blocks and 3276800 inodes
Filesystem UUID: bf15bd87-80c8-42ab-95f8-ddeb7feb7105
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208,
        4096000, 7962624, 11239424

Allocating group tables: done
Writing inode tables: done
Creating journal (65536 blocks): done
Writing superblocks and filesystem accounting information: done
```

```bash
[root@Server-A ~]# mkfs.ext4  /dev/sdb2
mke2fs 1.46.5 (30-Dec-2021)
Creating filesystem with 13106944 4k blocks and 3276800 inodes
Filesystem UUID: a0a16191-1ba6-4895-af93-e4b4ec52b55b
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208,
        4096000, 7962624, 11239424

Allocating group tables: done
Writing inode tables: done
Creating journal (65536 blocks): done
Writing superblocks and filesystem accounting information: done
```

```bash
[root@Server-A ~]# lsblk  -f
NAME FSTYPE FSVER LABEL      	UUID                                 		 FSAVAIL	 FSUSE%	 MOUNTPOINTS
sda
├─sda1
│    swap   1                		520bc18c-2b64-4df1-85e0-d126908ba6dd                	 [SWAP]
└─sda2
       xfs                     		73bc277c-741d-4122-9c58-59ccd1889709	 10.4G	 35% 	 /
sdb
├─sdb1
│    ext4   1.0              		bf15bd87-80c8-42ab-95f8-ddeb7feb7105
└─sdb2
      ext4   1.0              		a0a16191-1ba6-4895-af93-e4b4ec52b55b
sdc
sdd
sde
```

```bash
EX1-3) 아래의 조건에 맞게 설정하시오
 # 최상위 디렉터리에 '/githome'을 생성한 후 'sdb1'을 mout 해야한다.
 # 최상위 디렉터리에 '/winhome'을 생성한 후 'sdb2'을 mout 해야한다.
 # 'sdb1' Patition은 Server-A가 재부팅 되어도 바로 사용이 가능해야한다.
 # 'sdb2' Patition은 Server-A가 재부팅 되어도 바로 사용이 가능해야한다.

[root@Server-A ~]# mkdir  /githome

[root@Server-A ~]# mkdir  /winhome
```

```bash
[root@Server-A ~]# mount  /dev/sdb1  /githome/

[root@Server-A ~]# mount  /dev/sdb2  /winhome/
```

```bash
[root@Server-A ~]# lsblk  -f  /dev/sdb
NAME   FSTYPE FSVER LABEL 	UUID                                 	  	 FSAVAIL FSUSE% MOUNTPOINTS
sdb
├─sdb1 ext4   1.0         	   	bf15bd87-80c8-42ab-95f8-ddeb7feb7105	 46.4G      0% 	/githome
└─sdb2 ext4   1.0         	   	a0a16191-1ba6-4895-af93-e4b4ec52b55b	 46.4G      0% 	/winhome
```

```bash
[root@Server-A ~]# UUID1=$(blkid  -s UUID -o value  /dev/sdb1)

[root@Server-A ~]# UUID2=$(blkid  -s UUID -o value  /dev/sdb2)

# UUID1		: UUID1이라는 Shell 변수를 생성
# $()		: 괄호안의 내용을 먼저 실행한 후 결과를 가져오는 명령어
# blkid		: 디스크나 파티션의 의 UUID , LABEL 정보등을 가져오는 명령어
# -s UUID	: blkid 정보중 UUID의 정보만 선택
# -o value	: UUID 정보중 값만 선택해서 출력
# /dev/sdb1	: 가져올 디스크 또는 파티션 정보
```

```bash
[root@Server-A ~]# echo $UUID1
bf15bd87-80c8-42ab-95f8-ddeb7feb7105

[root@Server-A ~]# echo $UUID2
a0a16191-1ba6-4895-af93-e4b4ec52b55b
```

```bash
[root@Server-A ~]# cat  << EOF  >>  /etc/fstab
> UUID=$UUID1  /githome         	ext4   defaults  0 0
> UUID=$UUID2  /winhome         	ext4   defaults  0 0
> EOF

[root@Server-A ~]# cat /etc/fstab
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
UUID=73bc277c-741d-4122-9c58-59ccd1889709   	/               	xfs     	defaults  0 0
UUID=520bc18c-2b64-4df1-85e0-d126908ba6dd       none            	swap    	defaults  0 0
UUID=bf15bd87-80c8-42ab-95f8-ddeb7feb7105  	/githome     	ext4   	defaults  0 0	# 추가 확인
UUID=a0a16191-1ba6-4895-af93-e4b4ec52b55b  	/winhome		ext4	defaults  0 0	# 추가 확인
```

---

```bash
EX2-1) 아래의 조건에 맞게 계정을 생성하시오
 # '/githome' 디렉터리에 'quser1' , 'quser2' , 'quser3' 계정을 생성해야한다.

[root@Server-A ~]# useradd -md  /githome/quser1  quser1
[root@Server-A ~]# useradd -md  /githome/quser2  quser2
[root@Server-A ~]# useradd -md  /githome/quser3  quser3
```

```bash
[root@Server-A ~]# ls  -l  /githome
합계 28
drwx------ 2 root   root   16384  7월 13 11:48 lost+found
drwx------ 3 quser1 quser1  4096  7월 13 12:50 quser1
drwx------ 3 quser2 quser2  4096  7월 13 12:50 quser2
drwx------ 3 quser3 quser3  4096  7월 13 12:50 quser3
```

```bash
[root@Server-A ~]# grep  quser  /etc/passwd
quser1:x:1319:1319::/githome/quser1:/bin/bash
quser2:x:1320:1320::/githome/quser2:/bin/bash
quser3:x:1321:1321::/githome/quser3:/bin/bash
```

```bash
[root@Server-A ~]# cat  /etc/passwd  | grep quser
quser1:x:1319:1319::/githome/quser1:/bin/bash
quser2:x:1320:1320::/githome/quser2:/bin/bash
quser3:x:1321:1321::/githome/quser3:/bin/bash
```

```bash
	# Quota 설치

[root@Server-A ~]# rpm  -qa | grep quota
quota-nls-4.09-4.el9.noarch
quota-4.09-4.el9.x86_64
```

```bash
EX2-2) '/githome' 디렉터리에 quota를 구성해야한다.

[root@Server-A ~]# vi  /etc/fstab
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
UUID=73bc277c-741d-4122-9c58-59ccd1889709       /               	xfs     	defaults		0 0
UUID=520bc18c-2b64-4df1-85e0-d126908ba6dd       none            	swap    	defaults		0 0
UUID=bf15bd87-80c8-42ab-95f8-ddeb7feb7105  	/githome     	ext4   	defaults,usrquota	0 0	# quota 기능 활성화
UUID=a0a16191-1ba6-4895-af93-e4b4ec52b55b	/winhome     	ext4   	defaults		0 0
:wq
```

```bash
root@Server-A ~]# mount | grep sdb1		# quota 관련 설정이 확인되지 않는다.
/dev/sdb1 on /githome type ext4 (rw,relatime)
```

- quota 관련 설정을 반영하려면 mount를 해제 후 다시 mount 해야한다.

```bash
[root@Server-A ~]# mount  -o  remount  /dev/sdb1		# remount
mount: (hint) your fstab has been modified, but systemd still uses
       the old version; use 'systemctl daemon-reload' to reload.
```

```bash
[root@Server-A ~]# mount | grep sdb1
/dev/sdb1 on /githome type ext4 (rw,relatime,quota,usrquota)		# quota관련 권한이 확인된다.
```

---

#### quotacheck

- quotacheck는 Linux에서 디스크 Quota를 사용할 때, 파일시스템의 실제 사용량을 검사하고 Quota 관리 정보를 생성하거나 갱신하는 명령어이다.

- 파일시스템 전체를 검사하여 각 사용자와 그룹이 실제로 사용하고 있는 파일 수와 디스크 사용량을 계산한다.

- 계산된 정보는 Quota 관리 파일인 aquota.user, aquota.group에 기록된다.
 따라서 quotacheck는 Quota 기능이 사용할 기초 데이터를 만드는 역할과, 실제 사용량과 Quota 정보가 일치하는지 검사하는 역할을 수행한다.

#### quotacheck가 필요한 이유

- Linux에서 파일을 생성하거나 삭제하는 과정 중 서버가 비정상 종료되거나 Quota 관리 파일이 손상되면, Quota 정보와 실제 디스크 사용량이 서로 다를 수 있다.

- 다음과 같은 상황에서 Quota 정보를 다시 검사할 필요가 있다.
  - 서버가 강제로 종료된 경우
  - Quota 관리 파일이 삭제되거나 손상된 경우
  - 디스크를 다른 시스템에 연결하여 사용한 경우
  - Quota를 처음 구성하는 경우
  - 실제 사용량과 Quota 정보가 일치하지 않는 경우

#### quotacheck의 동작

- 파일시스템 전체를 스캔한다.
- 파일과 디렉터리의 소유자를 기준으로 사용자별/그룹별 실제 디스크 사용량을 계산한다.
- Quota 관리 파일인 aquota.user, aquota.group을 생성하거나 갱신한다.
- 기존 Quota 정보와 실제 사용량이 다르면 올바른 값으로 수정한다.
- Quota 시스템이 사용할 데이터베이스를 생성하거나 재구성한다.

### 주요 옵션

- c	: 기존 Quota 파일을 사용하지 않고 새 Quota 관리 파일을 생성	 (create quota)
- u	: 사용자 Quota 정보를 검사				 (user quota)
- g	: 그룹 Quota 정보를 검사				 (group quota)

 /githome	: Quota를 검사할 대상 파일시스템의 마운트 위치

```bash
[root@Server-A ~]# quotacheck  -cu  /githome
quotacheck: Your kernel probably supports ext4 quota feature but you are using external quota files. 
Please switch your filesystem to use ext4 quota feature as external quota files on ext4 are deprecated. 
You can enable the feature by unmounting the file system and running 'tune2fs -O quota <device>'.
```

```bash
[root@Server-A ~]# ls  -l  /githome
합계 36
-rw------- 1 root   root        7168  7월 13 13:06 aquota.user		# quota 적용시 quota.user파일이 생성된다.
drwx------ 2 root   root      16384  7월 13 11:48 lost+found
drwx------ 3 quser1 quser1    4096  7월 13 12:50 quser1
drwx------ 3 quser2 quser2    4096  7월 13 12:50 quser2
drwx------ 3 quser3 quser3    4096  7월 13 12:50 quser3
```

```bash
EX2-4) 'quser1' 계정의 용량을 10MB로 제한하며 파일개수는 제한하지 않는다.

[root@Server-A ~]# quotaoff  /githome		# /githome 디렉터리에 적용된 quota를 off로 전환 후 쿼터관련 설정
```

```bash
[root@Server-A ~]# edquota  quser1
```

```bash
[root@Server-A ~]# edquota  quser1
Disk quotas for user quser1 (uid 1319):
  Filesystem                   blocks       soft       hard     inodes     soft     hard
  /dev/sdb1                         36          0           0           10         0         0

-blocks
 # 현재 사용자가 사용 중인 디스크 용량 (KB 단위)
 # 메모리(RAM)가 아니라 디스크 파일시스템에서 차지하는 크기를 의미함

-soft (blocks)
 # 사용자가 쓸 수 있는 디스크 용량의 경고선
 # 초과하면 유예 기간(grace period, 기본 7일) 동안은 계속 사용 가능 (유예 기간이 지나면 파일 생성/쓰기 차단된다.)
 # 0으로 설정하면 제한 없음

-hard (blocks)
 # 사용자가 절대 넘을 수 없는 디스크 용량의 최대치로 이 값에 도달하면 즉시 쓰기/생성이 차단된다.
 # soft limit처럼 유예 기간은 없다.

-inodes
 # 현재 사용자가 사용 중인 파일(또는 디렉터리) 개수
 # 파일 수량 제한을 관리할 때 사용된다.

-soft (inodes)
 # 사용자가 생성할 수 있는 파일 개수의 경고선
 # 초과 시 grace period 동안은 허용되지만, 유예 기간이 지나면 더 이상 파일 생성 불가
 # 0으로 설정하면 제한 없음

hard (inodes)
 # 절대 초과할 수 없는 파일 개수의 최대치로이 값을 넘는 순간 추가 파일 생성이 즉시 차단된다.
```

```bash
[root@Server-A ~]# edquota  quser1
Disk quotas for user quser1 (uid 1319):
  Filesystem                   blocks       soft       hard     inodes     soft     hard
  /dev/sdb1                         36       10240      15360        10         0         0

:wq
```

```bash
[root@Server-A ~]# quotaon /githome/		# 파일 시스템의 quota를 on으로 전환
quotaon: Your kernel probably supports ext4 quota feature but you are using external quota files. 
Please switch your filesystem to use ext4 quota feature as external quota files on ext4 are deprecated. 
You can enable the feature by unmounting the file system and running 'tune2fs -O quota <device>'.
```

```bash
[root@Server-A ~]# quota  -u  quser1
Disk quotas for user quser1 (uid 1319):
     Filesystem  blocks   quota   limit   grace   files   quota   limit   grace
      /dev/sdb1      36   10240   15360              10          0       0
```

```bash
[root@Server-A ~]# passwd  quser1
quser1 사용자의 비밀 번호 변경 중
새 암호:1234
잘못된 암호: 암호는 8 개의 문자 보다 짧습니다
새 암호 재입력:1234
passwd: 모든 인증 토큰이 성공적으로 업데이트 되었습니다.
```

```bash
login as: quser1
quser1@192.168.10.100's password:
[quser1@Server-A ~]$ pwd
/githome/quser1
```

```bash
[quser1@Server-A ~]$ cp  -r  /etc/*  ./
cp: cannot open '/etc/NetworkManager/system-connections/ens160.nmconnection' for reading: 허가 거부
cp: cannot access '/etc/audit': 허가 거부
cp: cannot open '/etc/brlapi.key' for reading: 허가 거부
cp: cannot open '/etc/chrony.keys' for reading: 허가 거부
cp: cannot open '/etc/crypttab' for reading: 허가 거부
cp: cannot open '/etc/cups/subscriptions.conf.O' for reading: 허가 거부
cp: cannot open '/etc/cups/subscriptions.conf' for reading: 허가 거부
cp: cannot open '/etc/cups/classes.conf' for reading: 허가 거부
cp: cannot open '/etc/cups/cups-files.conf.default' for reading: 허가 거부
~~~~~~~~~~~~~~ 중간 생략 ~~~~~~~~~~~~~~
cp: './ssh/moduli'에 쓰는 도중 오류 발생: 디스크 할당량이 초과됨
cp: './ssh/ssh_config'에 쓰는 도중 오류 발생: 디스크 할당량이 초과됨
cp: cannot open '/etc/ssh/ssh_host_ed25519_key' for reading: 허가 거부
cp: './ssh/ssh_host_ed25519_key.pub'에 쓰는 도중 오류 발생: 디스크 할당량이 초과됨
cp: cannot open '/etc/ssh/ssh_host_ecdsa_key' for reading: 허가 거부
cp: './ssh/ssh_host_ecdsa_key.pub'에 쓰는 도중 오류 발생: 디스크 할당량이 초과됨
cp: cannot open '/etc/ssh/ssh_host_rsa_key' for reading: 허가 거부
~~~~~~~~~~~~~~ 중간 생략 ~~~~~~~~~~~~~~
```

```bash
[root@Server-A ~]# repquota  /githome
*** Report for user quotas on device /dev/sdb1
Block grace time: 7days; Inode grace time: 7days
                              Block limits                            File limits
User              used     soft      hard  grace    	used  soft  hard  grace

---

root      --        20         0	     0              	2      0      0
quser1    +-   15360   10240    15360  6days	1953  0      0
quser2    --       36         0	     0             	10     0      0
quser3    --       36         0	     0             	10     0      0

# -- 	: Block 용량과 File(Inode) 개수 모두 Soft Limit을 초과하지 않은 상태
# +- 	: Block 용량은 Soft Limit을 초과했지만 File(Inode) 개수는 초과하지 않은 상태
# -+ 	: Block 용량은 초과하지 않았지만 File(Inode) 개수는 Soft Limit을 초과한 상태
# ++ 	: Block 용량과 File(Inode) 개수가 모두 Soft Limit을 초과한 상태
# grace 	: Soft Limit을 초과한 후 제한이 적용되기까지 남은 유예 시간
```

**EX2-5)** 아래의 조건에 맞게 quota를 사용하여 용량을 제한하시오
  - 'quser1' 계정의 용량을 10MB로 제한하며 파일개수는 제한하지 않는다.
  - 제한된 용량을 모두 사용시 추가적으로 50%의 메모리를 사용할 수 있어야 한다.

  - 'quser2' 계정의 파일개수는 10,000개로 제한한다.
  - 제한된 개수을 모두 사용시 추가적으로 50%의 추가 파일을 사용할 수 있어야 한다.

  - 'quser3' 계정의 용량을 60MB로 제한하며, 파일개수는 10,000개로 제한한다.
  - 제한된 용량을 모두 사용시 추가적으로 50%의 추가 메모리, 50%의 추가 파일를 사용할 수 있어야 한다.

```bash
[root@server-a ~]# edquota  quser1
Disk quotas for user quser1 (uid 1310):
  Filesystem                   blocks	soft	hard	inodes	soft	hard
  /dev/sdb1                         28          10240        15360             7         0         0
```

```bash
[root@server-a ~]# edquota  quser2
Disk quotas for user quser1 (uid 1311):
  Filesystem                   blocks	soft	hard	inodes	soft	hard
  /dev/sdb1                         28        	0     	0    	7   	10000   	15000
```

```bash
[root@server-a ~]# edquota  quser3
Disk quotas for user quser1 (uid 1312):
  Filesystem                   blocks	soft	hard	inodes	soft	hard
  /dev/sdb1                         28   	61440 	92160  	7 	10000 	15000
```

```bash
[root@Server-A ~]# repquota  /githome
*** Report for user quotas on device /dev/sdb1
Block grace time: 7days; Inode grace time: 7days
                           Block limits                     File limits
User              used     soft    hard  grace    	used  soft    hard  grace

---

root      --        20         0         0              	2         0      0
quser1    +-   15360   10240   15360  6days    	1953     0     0
quser2    --       36         0         0             	10     10000 15000
quser3    --       36   61440   92160             	10     10000 15000
```

---

#### 그룹 쿼터(Group Quota)

- 사용자(user) 단위가 아니라 그룹(group) 단위로 디스크 제한하는 기능

- 리눅스는 다음 3가지 쿼터를 지원한다.
  - User Quota (USRQUOTA)
  - Group Quota (GRPQUOTA)
  - Project Quota (PRJQUOTA)    # 로키 리눅스에서 제외

- Group Quota는 그룹에 속한 모든 사용자의 파일 사용량을 합쳐서 하나의 총량으로 제한한다.

Example) 
팀 이름	: developers
그룹명	: devgrp
구성원	: user1, user2, user3

- devgrp 그룹에 전체 5GB 제한을 걸면 팀원 전체가 합쳐서 최대 5GB까지만 사용할 수 있다.
  즉, user1, user2, user3이 각각 얼마를 쓰든 상관없이 합계가 5GB를 넘으면 추가 파일 생성이나 쓰기가 제한된다.	

```bash
[root@Server-A ~]# mount | grep /dev/sdb
/dev/sdb1 on /githome type ext4 (rw,relatime,quota,usrquota)
/dev/sdb2 on /winhome type ext4 (rw,relatime)
```

```bash
[root@Server-A ~]# vi  /etc/fstab

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
UUID=73bc277c-741d-4122-9c58-59ccd1889709       /         		xfs     	defaults        		0 0
UUID=520bc18c-2b64-4df1-85e0-d126908ba6dd       none            	swap	defaults        		0 0
UUID=bf15bd87-80c8-42ab-95f8-ddeb7feb7105  	/githome     	ext4   	defaults,usrquota  		0 0
UUID=a0a16191-1ba6-4895-af93-e4b4ec52b55b  	/winhome     	ext4   	defaults,usrquota,grpquota  	0 0

:wq
```

```bash
[root@Server-A ~]# mount  -o  remount  /winhome/		# 리마운트
mount: (hint) your fstab has been modified, but systemd still uses
       the old version; use 'systemctl daemon-reload' to reload.
```

```bash
[root@Server-A ~]# mount | grep /dev/sdb			# 마운트 결과 확인
/dev/sdb1 on /githome type ext4 (rw,relatime,quota,usrquota)
/dev/sdb2 on /winhome type ext4 (rw,relatime,quota,usrquota,grpquota)
```

```bash
	# user quota 생성
[root@Server-A ~]# quotacheck  -cu  /winhome/
quotacheck: Your kernel probably supports ext4 quota feature but you are using external quota files. 
Please switch your filesystem to use ext4 quota feature as external quota files on ext4 are deprecated. 
You can enable the feature by unmounting the file system and running 'tune2fs -O quota <device>'.
```

```bash
	# group quota 생성
[root@Server-A ~]# quotacheck  -cg  /winhome/
quotacheck: Your kernel probably supports ext4 quota feature but you are using external quota files. 
Please switch your filesystem to use ext4 quota feature as external quota files on ext4 are deprecated. 
You can enable the feature by unmounting the file system and running 'tune2fs -O quota <device>'.
```

- -c		: 새로운 쿼터 생성 (create)
- -u		: 사용자 쿼터 (user quota)
- -g		: 그룹 쿼터 (group quota)
- /githome	: 대상 마운트 지점

```bash
[root@Server-A ~]# ls  -l  /winhome/
합계 32
-rw------- 1 root root  6144  7월 13 15:09 aquota.group
-rw------- 1 root root  6144  7월 13 15:09 aquota.user
drwx------ 2 root root 16384  7월 13 11:48 lost+found
```

```bash
[root@Server-A ~]# quotaon  -p  /winhome/
group quota on /winhome (/dev/sdb2) is off
user quota on /winhome (/dev/sdb2) is off
project quota on /winhome (/dev/sdb2) is off
```

```bash
EX1-1) 아래의 조건에 맞게 계정을 생성하시오
 # '/winhome 디렉터리에 'guser1' , 'guser2' , 'guser3' 계정을 생성해야한다

[root@Server-A ~]# useradd  -md  /winhome/guser1  guser1
[root@Server-A ~]# useradd  -md  /winhome/guser2  guser2
[root@Server-A ~]# useradd  -md  /winhome/guser3  guser3
```

```bash
[root@Server-A ~]# ls  -l  /winhome/
합계 44
-rw------- 1 root    root      6144  7월 13 15:09 aquota.group
-rw------- 1 root    root      6144  7월 13 15:09 aquota.user
drwx------ 3 guser1 guser 1  4096  7월 13 15:13 guser1
drwx------ 3 guser2 guser2   4096  7월 13 15:13 guser2
drwx------ 3 guser3 guser3   4096  7월 13 15:13 guser3
drwx------ 2 root    root    16384  7월 13 11:48 lost+found
```

```bash
[root@Server-A ~]# cat  /etc/passwd  | grep guser
guser1:x:1322:1322::/winhome/guser1:/bin/bash
guser2:x:1323:1323::/winhome/guser2:/bin/bash
guser3:x:1324:1324::/winhome/guser3:/bin/bash
```

```bash
[root@Server-A ~]# cat  /etc/group  | grep guser
guser1:x:1322:
guser2:x:1323:
guser3:x:1324:
```

```bash
[root@Server-A ~]# ls  -l  /var/spool/mail/  | grep guser
-rw-rw----  1 guser1   mail 0  7월 13 15:13 guser1
-rw-rw----  1 guser2   mail 0  7월 13 15:13 guser2
-rw-rw----  1 guser3   mail 0  7월 13 15:13 guser3
```

**EX1-2)** 아래의 조건에 맞게 그룹을 수정해야 한다.
  - 'winteam'그룹을 생성후 'guser1' , 'guser2' , 'guser3' 계정을 포함 시켜야 한다.

```bash
[root@Server-A ~]# groupadd  winteam

[root@Server-A ~]# usermod  -aG  winteam  guser1
[root@Server-A ~]# usermod  -aG  winteam  guser2
[root@Server-A ~]# usermod  -aG  winteam  guser3
```

```bash
[root@Server-A ~]# id  guser1
uid=1322(guser1) gid=1322(guser1) groups=1322(guser1),1336(winteam)
```

```bash
[root@Server-A ~]# id  guser2
uid=1323(guser2) gid=1323(guser2) groups=1323(guser2),1336(winteam)
```

```bash
[root@Server-A ~]# id  guser3
uid=1324(guser3) gid=1324(guser3) groups=1324(guser3),1336(winteam)
```

```bash
[root@Server-A ~]# edquota -g  winteam
Disk quotas for group winteam (gid 1336):
  Filesystem                   blocks       soft       hard     inodes     soft     hard
  /dev/sdb2                         0       20480      4060            0         0         0

:wq
```

```bash
[root@Server-A ~]# repquota  -g  /winhome/
*** Report for group quotas on device /dev/sdb2
Block grace time: 7days; Inode grace time: 7days
                        Block limits                File limits
Group           used    soft    hard  grace    used  soft  hard  grace

---

root      --      20        0        0                 2      0      0
```

```bash
[root@Server-A ~]# quotaoff  -ug  /winhome

quotaoff: Your kernel probably supports ext4 quota feature but you are using external quota files. 
Please switch your filesystem to use ext4 quota feature as external quota files on ext4 are deprecated. 
You can enable the feature by unmounting the file system and running 'tune2fs -O quota <device>'.
```

```bash
[root@Server-A ~]# chgrp  -R  winteam  /winhome/guser1
[root@Server-A ~]# chgrp  -R  winteam  /winhome/guser2
[root@Server-A ~]# chgrp  -R  winteam  /winhome/guser3
```

```bash
[root@Server-A ~]# ls  -l  /winhome
합계 44
-rw------- 1 root   root        7168  7월 13 15:54 aquota.group
-rw------- 1 root   root        7168  7월 13 15:54 aquota.user
drwx------ 3 guser1 winteam  4096  7월 13 15:13 guser1
drwx------ 3 guser2 winteam  4096  7월 13 15:13 guser2
drwx------ 3 guser3 winteam  4096  7월 13 15:13 guser3
drwx------ 2 root   root       16384  7월 13 11:48 lost+found
```

```bash
[root@Server-A ~]# quotaon  -ug  /winhome
```

```bash
[root@Server-A ~]# repquota  -g  /winhome/
*** Report for group quotas on device /dev/sdb2
Block grace time: 7days; Inode grace time: 7days
                        Block limits                File limits
Group           used    soft    hard  grace    used  soft  hard  grace

---

root        --      20          0        0               2     0     0
winteam   --     108   20480   40690             30     0     0
```

```bash
[root@Server-A ~]# repquota  /winhome/
*** Report for user quotas on device /dev/sdb2
Block grace time: 7days; Inode grace time: 7days
                        Block limits                File limits
User             used    soft    hard  grace    used  soft  hard  grace

---

root      --       20       0       0                2	0       0
guser1    --      36       0       0                10     	0       0		# 개별 계정에는 제한이 설정 X
guser2    --      36       0       0                10     	0       0		# 개별 계정에는 제한이 설정 X
guser3    --      36       0       0                10     	0       0		# 개별 계정에는 제한이 설정 X
```

```bash
[root@Server-A ~]# quotaon -p  /winhome/
group quota on /winhome (/dev/sdb2) is on
user quota on /winhome (/dev/sdb2) is on
project quota on /winhome (/dev/sdb2) is off
```

```bash
[root@Server-A ~]# passwd guser1		# guser1 계정에 비밀번호 설정
[root@Server-A ~]# passwd guser2		# guser2 계정에 비밀번호 설정
[root@Server-A ~]# passwd guser3		# guser3 계정에 비밀번호 설정
```

```bash
[root@Server-A ~]# chmod  2777  /winhome	# Set-GID (guser1,guser2,guser3 계정을 파일 및 디렉터리 생성시 그룹 권한으로 생성)
```

```bash
[root@Server-A ~]# chown  :winteam  /winteam
```

```bash
[root@Server-A ~]# ls  -l  /  | grep winhome
drwxrwsrwx    6 root winteam 4096  7월 13 16:23 winhome
```

```bash
login as: guser1
guser1@192.168.10.100's password:
[guser1@Server-A ~]$ pwd
/winhome/guser1
```

```bash
[guser1@Server-A ~]$ dd  if=/dev/zero  of=/winhome/bigfile.img  bs=1M  count=10	# bigfile.img 파일 생성
10+0 records in
10+0 records out
10485760 bytes (10 MB, 10 MiB) copied, 0.00630694 s, 1.7 GB/s

-dd			: 데이터를 블록 단위로 복사하거나 파일을 생성하는 명령어
-if=/dev/zero		: Input File이며 입력 장치로 /dev/zero를 사용, /dev/zero는 0으로 채워진 데이터를 계속 생성하는 특수 장치
-of=/winhome/bigfile.img	: Output File이며 생성할 출력 파일의 경로와 이름
-bs=1M			: 1MB 크기의 블록을 10번 생성
```

```bash
[guser1@Server-A ~]$ dd  if=/dev/zero  of=/winhome/bigfile2.img  bs=1M  count=10	# bigfile2.img 파일 생성
sdb2: warning, group block quota exceeded.
10+0 records in
10+0 records out
10485760 bytes (10 MB, 10 MiB) copied, 0.00798797 s, 1.3 GB/s
```

```bash
[guser1@Server-A ~]$ dd  if=/dev/zero  of=/winhome/bigfile3.img  bs=1M  count=10	# bigfile3.img 파일 생성
10+0 records in
10+0 records out
10485760 bytes (10 MB, 10 MiB) copied, 0.00650165 s, 1.6 GB/s
```

```bash
[guser1@Server-A ~]$ dd  if=/dev/zero  of=/winhome/bigfile4.img  bs=1M  count=10	# bigfile4.img 파일 생성
sdb2: write failed, group block limit reached.
dd: '/winhome/bigfile4.img'에 쓰는 도중 오류 발생: 디스크 할당량이 초과됨
10+0 records in
9+0 records out
10092544 bytes (10 MB, 9.6 MiB) copied, 0.0073289 s, 1.4 GB/s
```

```bash
[root@Server-A ~]# repquota  -g  /winhome/
*** Report for group quotas on device /dev/sdb2
Block grace time: 7days; Inode grace time: 7days
                        Block limits                            File limits
Group             used       soft    hard  grace    used  soft  hard  grace

---

root      --         16          0        0                  1     0     0
guser1    --          0         0        0                   1     0     0
winteam   +-   20592   20480   40690  6days       35     0     0
```

```bash
[root@Server-A ~]# ls  -l  /winhome/
합계 40620
-rw------- 1 root   root            7168  7월 13 15:59 aquota.group
-rw------- 1 root   root            7168  7월 13 15:59 aquota.user
-rw-r--r-- 1 guser1 winteam 10485760  7월 13 16:28 bigfile.img	# 파일 크기 10485760
-rw-r--r-- 1 guser1 winteam 10485760  7월 13 16:28 bigfile2.img	# 파일 크기 10485760
-rw-r--r-- 1 guser1 winteam 10485760  7월 13 16:29 bigfile3.img	# 파일 크기 10485760
-rw-r--r-- 1 guser1 winteam 10092544  7월 13 16:29 bigfile4.img	# 파일 크기 10092544
drwx------ 3 guser1 winteam      4096  7월 13 16:22 guser1
drwx------ 3 guser2 winteam      4096  7월 13 15:13 guser2
drwx------ 3 guser3 winteam      4096  7월 13 15:13 guser3
drwx------ 2 root   root           16384  7월 13 11:48 lost+found
```

---

**정리**: Disk Quota (8-1) 섹션에서 다룬 명령어와 실습 예제를 통해 관련 기능의 구성 방법과 동작 원리를 확인했다.

## RAID 이론 (8-2)

### RAID (Redunant Array of Inexpensive Disk)

- RAID란 여러 개의 물리적인 HDD(또는 SSD)를 하나의 논리적 디스크처럼 묶어 사용하는 기술이다.
 이렇게 여러개의 디스크를 묶으면 성능 향상, 데이터 보호(중복 저장), 대용량 저장공간 확보가 가능하다.

- RAID는 크게 Hardware RAID와 Software RAID로 나뉜다.
  두 방식 모두 같은 RAID 레벨(RAID 0, 1, 5 등)을 구성할 수 있지만 동작 방식과 성능, 고장 처리 방식이 다르다..

#### Hardware RAID

- 하드웨어 RAID는 RAID 전용 컨트롤러 카드(RAID Card, HBA) 또는 스토리지 장비를 사용해 RAID 연산을 수행한다.
 즉, 디스크 간의 데이터 분배(스트라이핑), 복제(미러링), 패리티 계산 등을 운영체제가 아닌 전용 하드웨어가 수행한다.

- 특징(성능·신뢰성 중심)
  - 디스크 입출력(IO)을 RAID 컨트롤러가 직접 처리
  - 서버 OS와 CPU의 부담이 감소
  - 고성능, 안정적

- 서버 전원을 끄지 않고 Hot Swap 가능
  - RAID 카드가 디스크 장애를 자동 감지하고 장애 디스크만 제거 가능

- 장점
  - 고성능 		: RAID 컨트롤러 칩셋이 대부분의 작업을 대신 처리
  - 높은 안정성	: 기업 서버 및 스토리지에서 일반적으로 사용하는 방식
  - Hot Swap 지원	: 서비스 중단 없이 장애 디스크 교체 가능
  - 대규모 서버 환경에서 필수적

- 단점
  - 비용이 높음 (RAID 카드, 전용 스토리지 장비는 가격이 높음)
  - RAID 카드 고장 시 같은 제조사/같은 모델/같은 펌웨어 버전이 필요할 수 있음

#### Software RAID

- 소프트웨어 RAID는 운영체제(OS)가 RAID 기능을 직접 수행하는 방식이다.
- Linux에서는 보통 mdadm 명령어로 구성하며, Windows에서는 디스크 관리 기능으로 구성한다.
- 즉, 데이터 스트라이핑/미러링/패리티 계산을 CPU가 직접 수행한다.

- 특징(저비용, 간단한 구성 중심)
  - 추가 하드웨어가 필요 없으므로 저렴
  - 서버나 PC에서도 쉽게 구성 가능
  - 클라우드 환경에서도 자주 사용되는 방식

- 장점
  - 비용 절감	: HDD/SSD만 있으면 구성 가능
  - 구성이 간단	: OS 명령어만으로 관리 가능
  - 유연성		: RAID 레벨 변경, 디스크 확장 등이 비교적 쉬움

- 단점
  - CPU 사용량 증가
   * RAID 연산이 CPU에 부담을 줌
   * 고성능이 필요한 환경에서는 부적합
  - Hot Swap 제한적
   * 디스크 교체 시 OS에서 장치를 분리하고 재인식 과정이 필요
  - 대규모 환경 부적합
   * 장애 처리 능력·성능이 Hardware RAID보다 떨어짐

---

#### Software RAID의 종류

- Software RAID는 운영체제(OS)의 소프트웨어 기능을 통해 구현되는 RAID 방식이다.
- 대표적인 유형 : Linear RAID, RAID 0, RAID 1, RAID 5, RAID 6
- 각 RAID 수준(Level)은 성능(Performance), 안정성(Reliability), 공간 효율성(Storage Efficiency) 에서 차이가 있다.

#### Linear RAID

- 최소 2개 이상의 Disk가 연결되어야한다.
- 2개이상의 HDD를 하나의 논리적인 HDD로 사용
- 데이터 저장시 첫번째 Disk부터 차례대로 저장되며 첫번째 디스크의 메모리를 모두 사용시2번째 디스크의 메모리로 저장된다.
- Fault-tolerance(결함허용)를 제공하지 않는다.
- 데이터를 읽고 쓰는 처리능력이 낮다.
 EX1) HDD1 (1 T) , HDD2 (1 T) , HDD3 (1 T) , HDD4 (1 T) 	  	= 총 4TByte의 공간을 사용할 수 있다. 
 EX2) HDD1 (10 T) , HDD2 (10 T) , HDD3 (10 T) , HDD4 (10 T) 	= 총 40TByte의 공간을 사용할 수 있다. 
 EX3) HDD1 (10T) , HDD2 (1T) , HDD3 (1T) , HDD4 (1T)        	= 총 13TByte의 공간을 사용할 수 있다.
 100%의 메모리를 사용할 수 있기때문에 공간효율이 좋다.

#### RAID 0

- 최소 2개 이상의 Disk가 연결되어야한다.
- 데이터 저장시 모든 Disk에 분할되어 저장된다.
- 데이터를 읽고 쓰는 처리능력이 Linear RAID보다 빠르지만 신뢰도가 낮다.
- 연결된 물리적인 디스크중 한개의 디스크가 오류 또는 고장시 모든 데이터가 문제될수 있다.
- Fault-tolerance(결함허용)를 제공하지 않는다.
- 모든 데이터를 잃어도 큰 문제가되지 않는 자료들만 저장하는것을 권장한다.
 .EX1) HDD1 (1 T) , HDD2 (1 T) , HDD3 (1 T) , HDD4 (1 T) 	  	= 총 4Tbyte의 공간을 사용할 수 있다. 
 .EX2) HDD1 (10 T) , HDD2 (10 T) , HDD3 (10 T) , HDD4 (10 T) 	= 총 40Tbyte의 공간을 사용할 수 있다. 
 .EX3) HDD1 (10T) , HDD2 (1T) , HDD3 (1T) , HDD4 (1T)       	 = 총 4Tbyte의 공간을 사용할 수 있다. (가장 작은 메모리기준)
 .모든 HDD의 크기가 동일한 경우 100%의 메모리를 사용할 수 있지만
  HDD의 용량이 다를 경우 가장 작은 HDD의 용량을 기준으로 사용할 수 있다.

#### RAID 1

- 최소 2개 이상의 Disk가 연결되어야한다.
- RAID 1을 Mirroring이라고도 한다.
- RAID 1을 사용시 필요한 Memory의 2배이상을 구성해야한다.
- 두배 이상의 Memory가 필요하기때문에 비용이 증가하며 공간 효율이 낮다.
- 저장 속도는 기존과 동일 하며 신뢰성이 높다.
- 중요한 데이터를 저장하기에 적합하다.
- Fault-tolerance(결함허용)를 제공한다.

#### RAID 5

- 가장 대중적으로 많이 쓰이는 RAID
- RAID 0의 공간 효율성과 RAID 1의 안정성 속도를 모두 사용하기위한 RAID이다.
- 최소 3개 이상의 HDD를 사용해야한다. (2개 사용시 공간 효율성이 50%이다.)
- 기본 동작은 RAID 0 처럼 각 HDD에 분할하여 저장한다.
- 안정성을 위해서 Parity bit를 사용한다. (HDD장애 발생시 Parity bit를 사용하여 데이터를 복구)
- Fault-tolerance(결함 허용)를 제공하면서 공간 효율성이 좋다.
- 공간 효율성 = HDD개수 - 1개
 .1T HDD 4개 사용시 공간 효율성 	= 75%
 .1T HDD 5개 사용시 공간 효율성 	= 80%
 .1T HDD 10개 사용시 공간 효율성 	= 90%
 .1T HDD 100개 사용시 공간 효율성 	= 99%
- 사용하는 HDD의 개수가 많을 수록 공간 효율성이 높다.
- 단 HDD가 동시에 2개가 오류시 데이터를 복구 할 수 없다.
- 일반적으로 HDD 10개이하로 구성시 RAID 5를 사용한다.

	Data = 000 001 010 011

	   	   X
	HDD1	HDD2	HDD3	HDD4  	=	HDD2
	   0	   0   	   0	   P0		   0
	   0	   0  	   P1	   1		   0
	   0	   P1   	   1	   0		   X
	   P0	   0	   1	   1		   0

#### RAID 6

- RAID 6는 RAID 5를 개량형이다.
- 최소 4개 이상의 HDD를 사용해야한다.
- RAID 6는 Parity bit를 2개 사용하기때문에 RAID 5보다 공간 효율은 낮지만 안정성이 높다.
- RAID 5 공간 효율성 = HDD개수 - 1개 (Parity bit를 1개 사용)
- RAID 6 공간 효율성 = HDD개수 - 2개 (Parity bit를 2개 사용)
- 2대의 HDD가 장애 또는 오류 발생시 Parity bit를 사용하여 복구할 수 있다.
- 공간 효율 RAID 5보다 낮음 (HDD 개수 - 2), 성능도 RAID 5보다 다소 떨어진다.

---

```bash
		# MD127 발생시

-RAID를 구성할 때 원래 /dev/md0으로 생성했지만, 재부팅 후 다음과 같이 /dev/md127로 자동 생성되는 경우가 있다.

-이 현상은 디스크 안에 이전 RAID 정보인 RAID 메타데이터(Superblock)가 남아 있기 때문에 발생한다.

-시스템은 부팅할 때 RAID 메타데이터를 확인하여 RAID를 자동으로 조립하는데, 
  기존 RAID 이름이나 설정 정보를 정확하게 찾지 못하면 임시 장치 번호인 /dev/md127을 사용하여 RAID를 생성할 수 있다.

# 자동 조립된 RAID 정지
[root@Server-A ~]# mdadm --stop /dev/md127
```

```bash
# 메타데이터 초기화(superblock 삭제)
[root@Server-A ~]# mdadm --zero-superblock /dev/sdb1
[root@Server-A ~]# mdadm --zero-superblock /dev/sdc1
```

```bash
# RAID 재생성
[root@Server-A ~]# mdadm --create /dev/md0 --level=linear --raid-devices=2 /dev/sdb1 /dev/sdc1
```

```bash
# mdadm.conf (현재 조립되어 있는 RAID 정보를 검색해서 /etc/mdadm.conf에 저장)
[root@Server-A ~]# mdadm --detail --scan > /etc/mdadm.conf
```

```bash
# initramfs 갱신

[root@Server-A ~]# dracut -fv
```

---

**정리**: RAID 이론 (8-2) 섹션에서 다룬 명령어와 실습 예제를 통해 관련 기능의 구성 방법과 동작 원리를 확인했다.

## Linear RAID 실습 (8-2)

#### Linear RAID

- 최소 2개 이상의 Disk가 연결되어야한다.
- 2개이상의 HDD를 하나의 논리적인 HDD로 사용
- 데이터 저장시 첫번째 Disk부터 차례대로 저장되며 첫번째 디스크의 메모리를 모두 사용시2번째 디스크의 메모리로 저장된다.
- Fault-tolerance(결함허용)를 제공하지 않는다.
- 데이터를 읽고 쓰는 처리능력이 낮다.
 EX1) HDD1 (1 T) , HDD2 (1 T) , HDD3 (1 T) , HDD4 (1 T) 	  	= 총 4TByte의 공간을 사용할 수 있다. 
 EX2) HDD1 (10 T) , HDD2 (10 T) , HDD3 (10 T) , HDD4 (10 T) 	= 총 40TByte의 공간을 사용할 수 있다. 
 EX3) HDD1 (10T) , HDD2 (1T) , HDD3 (1T) , HDD4 (1T)        	= 총 13TByte의 공간을 사용할 수 있다.
 100%의 메모리를 사용할 수 있기때문에 공간효율이 좋다.

```bash
EX1-1) Server-A에 10G의 디스크를 5개 장착 후 정보 확인

[root@Server-A ~]# ls  -ld  /dev/sd*
brw-rw---- 1 root disk 8,  0  7월 13 17:21 /dev/sda
brw-rw---- 1 root disk 8,  1  7월 13 17:21 /dev/sda1
brw-rw---- 1 root disk 8,  2  7월 13 17:21 /dev/sda2
brw-rw---- 1 root disk 8, 16  7월 13 17:21 /dev/sdb
brw-rw---- 1 root disk 8, 32  7월 13 17:21 /dev/sdc
brw-rw---- 1 root disk 8, 48  7월 13 17:21 /dev/sdd
brw-rw---- 1 root disk 8, 64  7월 13 17:21 /dev/sde
brw-rw---- 1 root disk 8, 80  7월 13 17:21 /dev/sdf
```

```bash
[root@Server-A ~]# fdisk  -l

Disk /dev/sda: 20 GiB, 21474836480 bytes, 41943040 sectors
Disk model: VMware Virtual S
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x47a034bc

Device     Boot   Start      End        Sectors    Size Id Type
/dev/sda1          2048       8390655  8388608   4G    82 Linux swap / Solaris
/dev/sda2  *      8390656   41943039 33552384  16G  83 Linux
```

---

**EX2)** Server-A에 장착된 HDD에 Patition을 구성
 .추가로 장착한 sdb를 하나의 Patition으로 구성해야한다.
 .추가로 장착한 sdc를 하나의 Patition으로 구성해야한다.
 .단 추가 장착한 두개의 HDD는 하나의 논리적인 HDD로 동작하도록 RAID로 구성해야한다.

```bash
[root@Server-A ~]# fdisk  /dev/sdb

Welcome to fdisk (util-linux 2.37.4).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Device does not contain a recognized partition table.
Created a new DOS disklabel with disk identifier 0xf15777b5.

Command (m for help): n			# 새로운 Patition
Partition type
   p   primary (0 primary, 0 extended, 4 free)
   e   extended (container for logical partitions)
Select (default p): p				# 주 Patition
Partition number (1-4, default 1): 1		# Patition 번호 1
First sector (2048-20971519, default 2048):
Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-20971519, default 20971519):

Created a new partition 1 of type 'Linux' and of size 10 GiB.

Command (m for help): m

Help:

  DOS (MBR)
   a   toggle a bootable flag
   b   edit nested BSD disklabel
   c   toggle the dos compatibility flag

  Generic
   d   delete a partition
   F   list free unpartitioned space
   l   list known partition types
   n   add a new partition
   p   print the partition table
   t   change a partition type		# Patition type 변경
   v   verify the partition table
   i   print information about a partition

  Misc
   m   print this menu
   u   change display/entry units
   x   extra functionality (experts only)

  Script
   I   load disk layout from sfdisk script file
   O   dump disk layout to sfdisk script file

  Save & Exit
   w   write table to disk and exit
   q   quit without saving changes

  Create a new label
   g   create a new empty GPT partition table
   G   create a new empty SGI (IRIX) partition table
   o   create a new empty DOS partition table
   s   create a new empty Sun partition table
```

```
Command (m for help): t
Selected partition 1
Hex code or alias (type L to list all): L

00 Empty            		24 NEC DOS          		81 Minix / old Lin  		f Solaris
01 FAT12            		27 Hidden NTFS Win  	82 Linux swap / So  	c1 DRDOS/sec (FAT-
02 XENIX root       		39 Plan 9           		83 Linux            		c4 DRDOS/sec (FAT-
03 XENIX usr        		3c PartitionMagic   		84 OS/2 hidden or   		c6 DRDOS/sec (FAT-
04 FAT16 <32M       	40 Venix 80286      		85 Linux extended   		c7 Syrinx
05 Extended         		41 PPC PReP Boot    	86 NTFS volume set  	da Non-FS data
06 FAT16            		42 SFS              		87 NTFS volume set  	db CP/M / CTOS / .
07 HPFS/NTFS/exFAT  	4d QNX4.x           		88 Linux plaintext  		de Dell Utility
08 AIX              		4e QNX4.x 2nd part  	8e Linux LVM        		df BootIt
09 AIX bootable     		4f QNX4.x 3rd part  		93 Amoeba           		e1 DOS access
0a OS/2 Boot Manag  	50 OnTrack DM       	94 Amoeba BBT       	e3 DOS R/O
0b W95 FAT32        	51 OnTrack DM6 Aux  	9f BSD/OS           		e4 SpeedStor
0c W95 FAT32 (LBA)  	52 CP/M             		a0 IBM Thinkpad hi  	ea Linux extended
0e W95 FAT16 (LBA)  	53 OnTrack DM6 Aux  	a5 FreeBSD          		eb BeOS fs
0f W95 Ext'd (LBA)  	54 OnTrackDM6       	a6 OpenBSD          		ee GPT
10 OPUS             		55 EZ-Drive         		a7 NeXTSTEP         	ef EFI (FAT-12/16/
11 Hidden FAT12    		56 Golden Bow       	a8 Darwin UFS       		f0 Linux/PA-RISC b
12 Compaq diagnost  	5c Priam Edisk      		a9 NetBSD           		f1 SpeedStor
14 Hidden FAT16 <3  	61 SpeedStor        		ab Darwin boot      		f4 SpeedStor
16 Hidden FAT16     	63 GNU HURD or Sys  	af HFS / HFS+       		f2 DOS secondary
17 Hidden HPFS/NTF	64 Novell Netware   	b7 BSDI fs          		fb VMware VMFS
18 AST SmartSleep   	65 Novell Netware   	b8 BSDI swap        		fc VMware VMKCORE
1b Hidden W95 FAT3  	70 DiskSecure Mult  	bb Boot Wizard hid  	fd Linux raid auto		<---- Linux raid
1c Hidden W95 FAT3  	75 PC/IX            		bc Acronis FAT32 L  	fe LANstep
1e Hidden W95 FAT1  	80 Old Minix        		be Solaris boot     		ff BBT

Aliases:
   linux     	- 83
   swap           	- 82
   extended       	- 05
   uefi          	- EF
   raid           	- FD
   lvm            	- 8E
   linuxex        	- 85
Hex code or alias (type L to list all): fd		<--- fd

Command (m for help): p

Disk /dev/sdb: 100 GiB, 107374182400 bytes, 209715200 sectors
Disk model: VMware Virtual S
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xc25bde4d

Device     Boot Start       End   Sectors  Size Id Type
/dev/sdb1        2048 209715199 209713152  100G fd Linux raid autodetect	<--- Linux raid

Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.
```

- /dev/sdc도 동일하게 Linux RAID로 변경

```bash
[root@Server-A ~]# lsblk
NAME   MAJ:MIN RM  SIZE  RO TYPE 	MOUNTPOINTS
sda      	  8:0    	0   20G  	0 disk
├─sda1	  8:1    	0    4G  	0 part 	[SWAP]
└─sda2	  8:2    	0   16G  	0 part 	/
sdb      	  8:16   	0   10G  	0 disk
└─sdb1	  8:17   	0   10G  	0 part
sdc      	  8:32   	0   10G  	0 disk
└─sdc1	  8:33   	0   10G  	0 part
sdd      	  8:48   	0   10G  	0 disk
sde      	  8:64   	0   10G  	0 disk
sdf      	  8:80   	0   10G  	0 disk
```

---

```
	# MDADM (Multiple Device Admin)
```

- 리눅스에서 소프트웨어 RAID를 생성/관리/복구/모니터링하는 대표 도구가 mdadm이다.

- 하드웨어 RAID가 컨트롤러 장비로 RAID를 구성하는 방식이라면, 소프트웨어 RAID는 운영체제가 디스크를 묶어 RAID 기능을 제공한다.

- mdadm은 리눅스에서 RAID 디스크 어레이(RAID array) 를 만들고 관리하는 명령어이다.

- RAID 0, 1, 5, 6같은 RAID 수준을 지원한다.

- 디스크를 묶어 하나의 가상 디스크(예: /dev/md0)처럼 사용할 수 있다.

- RAID 생성(create), 상세 확인(detail), 복구(assemble), 제거(remove), 모니터링(monitor) 등의 기능을 제공한다.

#### RAID 생성(create)

- 소프트웨어 RAID를 만들 때 가장 많이 사용하는 명령이 create이다.

형식 : mdadm --create --verbose [RAID장치] --level=[레벨] --raid-devices=[디스크 수] [디스크 목록]

예) RAID 1(미러링) 생성 /dev/sdb 와 /dev/sdc 두 개를 묶어서 /dev/md0 생성

- mdadm --create --verbose /dev/md0 --level=1 --raid-devices=2 /dev/sdb /dev/sdc

- /dev/md0 : 새롭게 생성되는 RAID 장치

- --level=1 : RAID 1(미러링)

- --raid-devices=2 : 사용되는 디스크 개수

- /dev/sdb /dev/sdc : 묶을 실제 디스크

- RAID 생성 후에는 /dev/md0 장치를 ext4 등 파일시스템으로 포맷 후 마운트해 사용한다.

#### RAID 장치 정보 확인(detail, examine)

- mdadm  --detail  /dev/md0

- 해당 RAID 장치의 전체 구성 정보를 보여준다.

- 예: 각 디스크 상태, RAID 레벨, Sync 진행률, 오류 여부 등.

- mdadm --examine  /dev/sdb
- 선택한 디스크에 저장된 RAID 관련 메타데이터(슈퍼블록)를 출력한다.
- 어느 RAID array에 속해있는지 확인할 때 필요하다.

#### RAID 모니터링(monitor)

- RAID 상태 변화를 감지하고 장애 발생 시 이메일 발송 및 syslog 기록 등을 수행한다.

형식: mdadm --monitor --scan

- 주요 기능
  - 디스크 장애 감지
  - RAID Sync 진행률 감시
  - 로그 기록 (/var/log/messages 등)

#### RAID 중지 및 해체(stop, remove)

- mdadm --stop /dev/md0
  - RAID 동작을 중지시킨다.
  - 마운트되어 있다면 반드시 먼저 umount해야 한다.

- mdadm --remove /dev/md0
  - RAID 장치를 제거한다.
  - 구성 디스크가 RAID에서 해제된다.

- 단순 remove는 메타데이터만 지우는 것이므로 디스크 자체의 슈퍼블록은 남아 있을 수도 있다.
- 필요하면 아래처럼 슈퍼블록 초기화해야 한다.
  - mdadm --zero-superblock /dev/sdb

- Superblock(슈퍼블록)
  - 슈퍼블록은 파일시스템이나 RAID에 대한 핵심 정보를 저장해 둔 작은 메타데이터 블록
  - 이 디스크(또는 RAID)가 어떤 구조로 구성되어 있고 어떻게 사용해야 하는지를 기록해 둔 일종의 설명서

#### RAID 복구(assemble)

- 시스템 재부팅 후 RAID가 자동으로 조립되지 않을 경우 사용한다.

형식 : mdadm --assemble --scan
  - 디스크의 슈퍼블록 정보를 읽고 자동으로 RAID array를 재구성한다.
  - /etc/mdadm.conf 에 RAID 설정이 등록되어 있을 경우 안정적으로 자동 조립된다.
  - mdadm --assemble /dev/md0 /dev/sdb /dev/sdc 처럼 수동으로 명시해 복구할 수도 있다.

---

**EX3)** Patiton을 구성한 sdb1 , sdc1을 Linear RAID로 구성해야한다.

- mdadm을 사용하기위해서는 mdadm package가 설치되어있는지를 확인해야 한다.

```bash
[root@Server-A ~]# rpm  -qa | grep mdadm
mdadm-4.4-4.el9_7.x86_64
```

```bash
[root@Server-A ~]# dnf  install  -y  mdadm 	# 미설치시
```

```bash
	# Linear RAID 설정

[root@Server-A ~]# mdadm  --create  /dev/md0  --level=linear  --raid-devices=2  /dev/sdb1  /dev/sdc1
mdadm: Defaulting to version 1.2 metadata
mdadm: array /dev/md0 started.
```

```bash
[root@Server-A ~]# mdadm  --detail  --scan
ARRAY /dev/md0 metadata=1.2 UUID=77dca520:4111cbe8:c08a0fe3:ec203cc0
```

```bash
[root@Server-A ~]# mdadm  --detail  /dev/md0
/dev/md0:						# 장치명
           Version 	: 1.2				# 메타데이터 버전 정보
     Creation Time 	: Mon Jul 13 17:47:58 2026		# 생성된 날짜/시간
        Raid Level 	: linear				# RAID 종류
        Array Size 	: 20951040 (19.98 GiB 21.45 GB)	# 디스크 크기
      Raid Devices 	: 2				# RAID를 구성하는 디스크 개수 (--create로 설정된 디스크 개수)
     Total Devices 	: 2				# 실제 연결된 디스크 개수
       Persistence 	: Superblock is persistent

       Update Time 	: Mon Jul 13 17:47:58 2026		# 마지막 수정 시간
             State 	: clean				# clean = 정상 동작
    Active Devices 	: 2				# RAID에 참여하는 디스크 개수
   Working Devices	: 2				# 고장없이 살아있곻 RAID에서 동작하는 디스크 개수
    Failed Devices 	: 0				# 고장난 디스크수
     Spare Devices 	: 0				# 여분 디스크

          Rounding 	: 0K

Consistency Policy 	: none

              Name : Server-A:0  (local to host Server-A)	# RAID 배열 이름
              UUID : 77dca520:4111cbe8:c08a0fe3:ec203cc0	## UUID (장치명)
            Events : 0

    Number   Major   Minor   RaidDevice State
       0         8        17        0              active sync   /dev/sdb1
       1         8        33        1              active sync   /dev/sdc1
```

```bash
[root@Server-A ~]# lsblk  -f
NAME    FSTYPE            FSVER LABEL           	UUID                                 		 FSAVAIL	 FSUSE% MOUNTPOINTS
sda
├─sda1  swap              1                          	520bc18c-2b64-4df1-85e0-d126908ba6dd                	 [SWAP]
└─sda2  xfs                                          	73bc277c-741d-4122-9c58-59ccd1889709	 10.4G	 35% 	 /
sdb
└─sdb1  linux_raid_member 1.2   Server-A:0    	77dca520-4111-cbe8-c08a-0fe3ec203cc0		# UUID가 동일하다.
  └─md0
sdc
└─sdc1  linux_raid_member 1.2   Server-A:0    	77dca520-4111-cbe8-c08a-0fe3ec203cc0		# UUID가 동일하다.
  └─md0
sdd
sde
sdf
```

---

- 물리적인 2개의 HDD를 하나의 논리적인 HDD로 구성했지만 Format형식을 지정하지 않았기때문에 사용할 수 없다.

**EX4-1)** Linear RAID로 구성한 HDD를 EXT4 형식으로 format해야한다.

```bash
[root@Server-A ~]# mkfs.ext4  /dev/md0
mke2fs 1.46.5 (30-Dec-2021)
Creating filesystem with 5237760 4k blocks and 1310720 inodes
Filesystem UUID: f578247c-cf58-44d0-a661-048f9eb32659
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208,
        4096000

Allocating group tables: done
Writing inode tables: done
Creating journal (32768 blocks): done
Writing superblocks and filesystem accounting information: done
```

```bash
[root@Server-A ~]# lsblk  -f
NAME    FSTYPE            FSVER LABEL    	UUID                                 		FSAVAIL	FSUSE% 	MOUNTPOINTS
sda
├─sda1  swap              1                          	520bc18c-2b64-4df1-85e0-d126908ba6dd                	[SWAP]
└─sda2  xfs                                          	73bc277c-741d-4122-9c58-59ccd1889709	10.4G	35% 	/
sdb
└─sdb1  linux_raid_member 1.2   Server-A:0   	77dca520-4111-cbe8-c08a-0fe3ec203cc0		# sdb1, sdb2를 하나의 HDD로 연결시 사용되는 UUID
  └─md0 ext4              1.0                        	f578247c-cf58-44d0-a661-048f9eb32659		# mount시 사용되는 UUID
sdc
└─sdc1  linux_raid_member 1.2   Server-A:0   	77dca520-4111-cbe8-c08a-0fe3ec203cc0		# sdb1, sdb2를 하나의 HDD로 연결시 사용되는 UUID
  └─md0 ext4              1.0                        	f578247c-cf58-44d0-a661-048f9eb32659		# mount시 사용되는 UUID
sdd
sde
sdf
```

---

- Linear RAID구성후 EXT4 형식으로 File system을 구성했지만 mount되지 않았기 때문에 사용할 수 없다.

**EX4-2)** Linear RAID로 구성한 HDD를 '/linear' 디렉터리로 mount 해야한다.

```bash
[root@Server-A ~]# mkdir  /linear
```

```bash
[root@Server-A ~]# mount  /dev/md0  /linear
```

```bash
[root@Server-A ~]# mount | grep md0
/dev/md0 on /linear type ext4 (rw,relatime)
```

```bash
[root@Server-A ~]# lsblk  -f
NAME    FSTYPE            FSVER LABEL    	UUID                                 		FSAVAIL	FSUSE% 	MOUNTPOINTS
sda
├─sda1  swap              1                          	520bc18c-2b64-4df1-85e0-d126908ba6dd                	[SWAP]
└─sda2  xfs                                          	73bc277c-741d-4122-9c58-59ccd1889709	10.4G	35% 	/
sdb
└─sdb1  linux_raid_member 1.2   Server-A:0   	77dca520-4111-cbe8-c08a-0fe3ec203cc0
  └─md0 ext4              1.0                        	f578247c-cf58-44d0-a661-048f9eb32659	18.5G     	0% 	/linear
sdc
└─sdc1  linux_raid_member 1.2   Server-A:0   	77dca520-4111-cbe8-c08a-0fe3ec203cc0
  └─md0 ext4              1.0                        	f578247c-cf58-44d0-a661-048f9eb32659	18.5G     	0% 	/linear
sdd
sde
sdf
```

```bash
[root@Server-A ~]# df  -h
Filesystem      	Size   Used Avail Use% Mounted on
devtmpfs        	807M      0  807M   0% /dev
tmpfs           	838M      0  838M   0% /dev/shm
tmpfs           	335M  6.8M  329M   3% /run
/dev/sda2        	16G    5.6G   11G  35% /
tmpfs           	168M   56K  168M   1% /run/user/42
tmpfs           	168M   40K  168M   1% /run/user/0
/dev/md0         	20G    24K   19G    1% /linear
```

```bash
[root@Server-A ~]# reboot
```

```bash
[root@Server-A ~]# mount | grep md0		# 마운특 확인되지 않는다.
```

---

```bash
-Linear RAID를 구성했지만 Server-A가 재부팅시 mount는 해제된다.

[root@localhost ~]# reboot
```

**EX4-3)** Linear RAID로 구성한 HDD는  Server-A가 재부팅되어도 바로 동작해야한다

```bash
[root@Server-A ~]# lsblk  -f
NAME    FSTYPE            FSVER LABEL    	UUID                                 		FSAVAIL	FSUSE% 	MOUNTPOINTS
sda
├─sda1  swap              1                          	520bc18c-2b64-4df1-85e0-d126908ba6dd                	[SWAP]
└─sda2  xfs                                          	73bc277c-741d-4122-9c58-59ccd1889709	10.4G	35% 	/
sdb
└─sdb1  linux_raid_member 1.2   Server-A:0   	77dca520-4111-cbe8-c08a-0fe3ec203cc0
  └─md0 ext4              1.0                        	f578247c-cf58-44d0-a661-048f9eb32659	18.5G     	0% 	/linear
sdc
└─sdc1  linux_raid_member 1.2   Server-A:0   	77dca520-4111-cbe8-c08a-0fe3ec203cc0
  └─md0 ext4              1.0                        	f578247c-cf58-44d0-a661-048f9eb32659	18.5G     	0% 	/linear	# UUID 비교 f578247c-cf58-44d0-a661-048f9eb32659
sdd
sde
sdf
```

```bash
[root@Server-A ~]# mkdir  /linear
```

```bash
[root@Server-A ~]# UUID1=$(blkid -s UUID -o value /dev/md0)
```

```bash
[root@Server-A ~]# echo $UUID1
f578247c-cf58-44d0-a661-048f9eb32659		# UUID 비교 f578247c-cf58-44d0-a661-048f9eb32659
```

```bash
[root@Server-A ~]# cat << EOF  >> /etc/fstab
> UUID=$UUID1  /linear  ext4  defaults  0 0
> EOF
```

```bash
[root@Server-A ~]# cat  /etc/fstab
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
UUID=73bc277c-741d-4122-9c58-59ccd1889709   	/         	xfs     	defaults	0 0
UUID=520bc18c-2b64-4df1-85e0-d126908ba6dd	none   	swap    	defaults	0 0
UUID=f578247c-cf58-44d0-a661-048f9eb32659  	/linear	ext4	defaults	0 0
```

```bash
[root@Server-A ~]# reboot
```

```bash
[root@Server-A ~]# mount | grep md0
/dev/md0 on /linear type ext4 (rw,relatime)
```

---

**정리**: Linear RAID 실습 (8-2) 섹션에서 다룬 명령어와 실습 예제를 통해 관련 기능의 구성 방법과 동작 원리를 확인했다.

## RAID 0 실습 (8-3)

#### RAID 0

- 최소 2개 이상의 Disk가 연결되어야한다.
- 데이터 저장시 모든 Disk에 분할되어 저장된다.
- 데이터를 읽고 쓰는 처리능력이 Linear RAID보다 빠르지만 신뢰도가 낮다.
- 연결된 물리적인 HDD중 한개의 HDD가 오류 또는 고장시 모든 데이터가 문제될수 있다.
- 모든 데이터를 잃어도 큰 문제가되지 않는 자료들만 저장하는것을 권장한다.
- EX1) HDD1 (1 T) , HDD2 (1 T) , HDD3 (1 T) , HDD4 (1 T) 	  	= 총 4Tbyte의 공간을 사용할 수 있다. 
- EX2) HDD1 (10 T) , HDD2 (10 T) , HDD3 (10 T) , HDD4 (10 T) 	= 총 40Tbyte의 공간을 사용할 수 있다. 
- EX3) HDD1 (10T) , HDD2 (1T) , HDD3 (1T) , HDD4 (1T)        	= 총 4Tbyte의 공간을 사용할 수 있다. (가장 작은 메모리기준)
- 모든 HDD의 크기가 동일한 경우 100%의 메모리를 사용할 수 있지만
  HDD의 크기가 다를 경우 가장 작은 HDD의 용량을 기준으로 사용할 수 있다.

---

**EX1-1)** Server-A에 10G HDD 2대 장착

#### 기존 디스크 초기화

```bash
	# 현재 /dev/md0 RAID 장치가 /linear 디렉터리에 마운트되어 있는지 확인

[root@Server-A ~]# mount | grep md0
/dev/md0 on /linear type ext4 (rw,relatime)
```

```bash
	# /linear에 마운트된 RAID 파일시스템을 마운트 해제

[root@Server-A ~]# umount /linear
```

```bash
	# /dev/md0에 생성된 파일시스템 서명 삭제

[root@Server-A ~]# wipefs  -a  -f  /dev/md0

# wipefs	: 디스크나 파티션에 기록된 파일시스템, RAID, LVM 등의 서명(Signature)을 확인하거나 삭제하는 명령어
# -a	: all의 약자로 대상 장치에서 발견되는 모든 서명을 삭제
# -f	: force의 약자로 장치가 파티션이 아닌 전체 디스크이거나 RAID 같은 특수 블록 장치여도 강제로 서명 삭제
```

```bash
	# 활성화되어 있는 /dev/md0 RAID 장치를 중지  (RAID 구성을 삭제하는 것이 아니라 현재 조립된 RAID의 동작만 중지)

[root@Server-A ~]# mdadm  --stop  /dev/md0
```

```bash
	# /dev/sdb1과 /dev/sdc1에 기록된 RAID 메타데이터(Superblock)를 삭제 (`RAID 레벨, RAID UUID, 디스크 순서, 구성 정보 등을 제거)

[root@Server-A ~]# mdadm --zero-superblock /dev/sdb1
[root@Server-A ~]# mdadm --zero-superblock /dev/sdc1

        OR

[root@Server-A ~]# mdadm --zero-superblock --force /dev/sdb1
[root@Server-A ~]# mdadm --zero-superblock --force /dev/sdc1
```

```bash
	# /dev/sdb와 /dev/sdc 디스크 전체에서 발견되는 모든 서명을 삭제 (디스크 전체에 실행하므로 sdb1, sdc1 파티션 정보까지 삭제)

[root@Server-A ~]# wipefs -a /dev/sdb

[root@Server-A ~]# wipefs -a /dev/sdc
```

```bash
[root@Server-A ~]# lsblk  -f
NAME FSTYPE FSVER LABEL UUID                                                 FSAVAIL FSUSE% MOUNTPOINTS
sda
├─sda1
│    swap   1           	  520bc18c-2b64-4df1-85e0-d126908ba6dd                              [SWAP]
└─sda2
       xfs                	  73bc277c-741d-4122-9c58-59ccd1889709   10.4G      35%          /
sdb
sdc
sdd
sde
sdf
```

```bash
[root@Server-A ~]# cat  /etc/fstab
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
UUID=73bc277c-741d-4122-9c58-59ccd1889709       /          	xfs     	defaults	0 0
UUID=520bc18c-2b64-4df1-85e0-d126908ba6dd       none  	swap    	defaults	0 0
UUD=f578247c-cf58-44d0-a661-048f9eb32659  	/linear   	ext4	defaults	0 0	# 삭제
'
```

---

```bash
EX1-2) Server-A에 장착된 SDD에 Patition을 구성
 # 추가로 장착한 sdb를 하나의 Patition으로 구성해야한다.
 # 추가로 장착한 sdc를 하나의 Patition으로 구성해야한다.
 # /dev/sdb , /dev/sdc Patition 구성시 RAID용으로 구성해야한다.

[root@Server-A ~]# fdisk  /dev/sdb		<---

Welcome to fdisk (util-linux 2.37.4).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Device does not contain a recognized partition table.
Created a new DOS disklabel with disk identifier 0xb0bfc9fe.

Command (m for help): n		<---
Partition type
   p   primary (0 primary, 0 extended, 4 free)
   e   extended (container for logical partitions)
Select (default p): p
Partition number (1-4, default 1): 1		<---
First sector (2048-20971519, default 2048):
Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-20971519, default 20971519):		<---

Created a new partition 1 of type 'Linux' and of size 10 GiB.

Command (m for help): t		<---
Selected partition 1
Hex code or alias (type L to list all): L		<---

00 Empty            24 NEC DOS          81 Minix / old Lin  bf Solaris
01 FAT12            27 Hidden NTFS Win  82 Linux swap / So  c1 DRDOS/sec (FAT-
02 XENIX root       39 Plan 9           83 Linux            c4 DRDOS/sec (FAT-
03 XENIX usr        3c PartitionMagic   84 OS/2 hidden or   c6 DRDOS/sec (FAT-
04 FAT16 <32M       40 Venix 80286      85 Linux extended   c7 Syrinx
05 Extended         41 PPC PReP Boot    86 NTFS volume set  da Non-FS data
06 FAT16            42 SFS              87 NTFS volume set  db CP/M / CTOS / .
07 HPFS/NTFS/exFAT  4d QNX4.x           88 Linux plaintext  de Dell Utility
08 AIX              4e QNX4.x 2nd part  8e Linux LVM        df BootIt
09 AIX bootable     4f QNX4.x 3rd part  93 Amoeba           e1 DOS access
0a OS/2 Boot Manag  50 OnTrack DM       94 Amoeba BBT       e3 DOS R/O
0b W95 FAT32        51 OnTrack DM6 Aux  9f BSD/OS           e4 SpeedStor
0c W95 FAT32 (LBA)  52 CP/M             a0 IBM Thinkpad hi  ea Linux extended
0e W95 FAT16 (LBA)  53 OnTrack DM6 Aux  a5 FreeBSD          eb BeOS fs
0f W95 Ext'd (LBA)  54 OnTrackDM6       a6 OpenBSD          ee GPT
10 OPUS             55 EZ-Drive         a7 NeXTSTEP         ef EFI (FAT-12/16/
11 Hidden FAT12     56 Golden Bow       a8 Darwin UFS       f0 Linux/PA-RISC b
12 Compaq diagnost  5c Priam Edisk      a9 NetBSD           f1 SpeedStor
14 Hidden FAT16 <3  61 SpeedStor        ab Darwin boot      f4 SpeedStor
16 Hidden FAT16     63 GNU HURD or Sys  af HFS / HFS+       f2 DOS secondary
17 Hidden HPFS/NTF  64 Novell Netware   b7 BSDI fs          fb VMware VMFS
18 AST SmartSleep   65 Novell Netware   b8 BSDI swap        fc VMware VMKCORE
1b Hidden W95 FAT3  70 DiskSecure Mult  bb Boot Wizard hid  fd Linux raid auto
1c Hidden W95 FAT3  75 PC/IX            bc Acronis FAT32 L  fe LANstep
1e Hidden W95 FAT1  80 Old Minix        be Solaris boot     ff BBT

Aliases:
   linux          - 83
   swap           - 82
   extended       - 05
   uefi           - EF
   raid           - FD
   lvm            - 8E
   linuxex        - 85
Hex code or alias (type L to list all): fd		<---
Changed type of partition 'Linux' to 'Linux raid autodetect'.

Command (m for help): p		<---
Disk /dev/sdb: 10 GiB, 10737418240 bytes, 20971520 sectors
Disk model: VMware Virtual S
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xb0bfc9fe

Device     Boot Start      End  Sectors Size Id Type
/dev/sdb1        2048 20971519 20969472  10G fd Linux raid autodetect

Command (m for help): w		<---
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.
```

```bash
[root@Server-A ~]# lsblk
NAME   MAJ:MIN RM  SIZE RO TYPE 	MOUNTPOINTS
sda      	  8:0    	0   20G  	0 disk
├─sda1	  8:1    	0    4G  	0 part 	[SWAP]
└─sda2	  8:2    	0   16G  	0 part 	/
sdb      	  8:16   	0   10G  	0 disk
└─sdb1	  8:17   	0   10G  	0 part
sdc      	  8:32   	0   10G  	0 disk
└─sdc1	  8:33   	0   10G  	0 part
sdd      	  8:48   	0   10G  	0 disk
sde      	  8:64   	0   10G  	0 disk
sdf      	  8:80   	0   10G  	0 disk
```

```bash
[root@Server-A ~]# lsblk  -f
NAME FSTYPE FSVER LABEL	UUID                      		 FSAVAIL FSUSE% MOUNTPOINTS
sda
├─sda1
│    swap   1           		520bc18c-2b64-4df1-85e0-d126908ba6dd                	[SWAP]
└─sda2
     xfs                			73bc277c-741d-4122-9c58-59ccd1889709	 10.4G 	35% 	/
sdb
└─sdb1

sdc
└─sdc1

sdd
sde
sdf
```

---

```bash
EX3) Patiton을 구성한 sdb1 , sdc1을 RAID 0으로 구성해야한다.

[root@Server-A ~]# rpm  -qa | grep mdadm
mdadm-4.4-4.el9_7.x86_64
```

```bash
[root@Server-A ~]# mdadm  --create  /dev/md0  --level=0  --raid-devices=2  /dev/sdb1  /dev/sdc1
mdadm: Defaulting to version 1.2 metadata
mdadm: array /dev/md0 started.
```

```bash
[root@Server-A ~]# mdadm  --detail  -scan
ARRAY /dev/md0 metadata=1.2 UUID=d6211e7b:fb047dcf:77a22771:ee1c406c
```

```bash
[root@Server-A ~]# lsblk -f
NAME FSTYPE FSVER LABEL UUID                                 	     FSAVAIL FSUSE% MOUNTPOINTS
sda
├─sda1
│    swap   1           	  520bc18c-2b64-4df1-85e0-d126908ba6dd                              [SWAP]
└─sda2
     xfs                		  73bc277c-741d-4122-9c58-59ccd1889709   10.4G       35%        /
sdb
└─sdb1
     linux_ 1.2   Server-A:0	  d6211e7b-fb04-7dcf-77a2-2771ee1c406c
sdc
└─sdc1
     linux_ 1.2   Server-A:0	  d6211e7b-fb04-7dcf-77a2-2771ee1c406c
sdd
sde
sdf
```

```bash
[root@Server-A ~]# mdadm --detail /dev/md0                   # /dev/md0 RAID 장치의 상세 정보 확인

/dev/md0:                                                   		# 확인 중인 RAID 장치 이름
           Version 	: 1.2                                   	# RAID 메타데이터(Superblock) 버전
     Creation Time 	: Tue Jul 14 10:34:29 2026               	# RAID가 생성된 날짜와 시간
        Raid Level 	: raid0                                  	# RAID 레벨: RAID 0(Striping)
        Array Size 	: 20951040 (19.98 GiB 21.45 GB)          	# RAID 전체 사용 가능 용량
      Raid Devices 	: 2                                      	# RAID 구성에 필요한 디스크 수
     Total Devices	: 2                                      	# 현재 RAID에 연결된 전체 디스크 수
       Persistence 	: Superblock is persistent               	# RAID 정보가 디스크 Superblock에 영구 저장됨

       Update Time 	: Tue Jul 14 10:34:29 2026               	# RAID 상태가 마지막으로 갱신된 시간
             State 	: clean                                  	# RAID 상태가 정상이고 안정된 상태
    Active Devices 	: 2                                      	# 현재 활성 상태로 사용 중인 디스크 수
   Working Devices 	: 2                                      	# 정상적으로 동작 중인 전체 디스크 수
    Failed Devices 	: 0                                      	# 장애가 발생한 디스크 수
     Spare Devices 	: 0                                      	# 대기용 예비 디스크 수

            Layout 	: original                               	# RAID 0의 데이터 배치 방식
        Chunk Size 	: 512K                                   	# 한 번에 각 디스크에 나누어 기록하는 데이터 크기

Consistency Policy	: none                                   	# 별도의 일관성 검사 정책을 사용하지 않음

              Name 	: Server-A:0  (local to host Server-A) 	# RAID 배열 이름
              UUID 	: d6211e7b:fb047dcf:77a22771:ee1c406c	# RAID 배열의 고유 식별자
            Events 	: 0                                      	# RAID 메타데이터 변경 횟수

    Number   Major   Minor   RaidDevice State
       0         8        17          0            active sync /dev/sdb1	# 첫 번째 RAID 구성 디스크이며 정상적으로 활성화된 상태
       1         8        33          1            active sync /dev/sdc1	# 두 번째 RAID 구성 디스크이며 정상적으로 활성화된 상태
```

```bash
	# md127 문제 방지

# /etc/mdadm.conf 파일 생성 후 "mdadm  --detail  --scan" 출력 결과를 저장

[root@Server-A ~]# mdadm  --detail  --scan
ARRAY /dev/md0 metadata=1.2 UUID=d6211e7b:fb047dcf:77a22771:ee1c406c
```

```bash
[root@Server-A ~]# mdadm  --detail  --scan  >  /etc/mdadm.conf
```

initramfs 갱신
- initramfs	: 리눅스가 부팅 초기에 사용하는 임시 루트 파일시스템으로 실제 루트 파일시스템을 마운트하기 전에 
  	   	  필요한 드라이버, RAID, LVM, 파일시스템 모듈 및 부팅 관련 설정을 포함한다.

- dracut	: initramfs 이미지를 생성하거나 다시 갱신하는 프로그램

```bash
[root@Server-A ~]# dracut -fv
# 현재 커널에 맞는 initramfs 이미지를 강제로 다시 생성 (/etc/mdadm.conf 등에 변경된 RAID 설정이 있으면 initramfs에 반영)
# -f : force이며 기존 initramfs 이미지가 존재하더라도 강제로 덮어써서 다시 생성한다.
# -v : verbose이며 initramfs 생성 과정을 화면에 자세하게 출력한다.
```

---

- 물리적인 2개의 HDD를 하나의 논리적인 HDD로 구성했지만 Format형식을 지정하지 않았기때문에 사용할 수 없다.

```bash
EX3-1) RAID 0으로 구성한 HDD를 EXT4 형식으로 format해야한다.

[root@Server-A ~]# mkfs.ext4  /dev/md0
mke2fs 1.46.5 (30-Dec-2021)
Creating filesystem with 5237760 4k blocks and 1310720 inodes
Filesystem UUID: 4efff958-7112-479f-a95a-0421f8f4ce4e
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208,
        4096000

Allocating group tables: done
Writing inode tables: done
Creating journal (32768 blocks): done
Writing superblocks and filesystem accounting information: done
```

```bash
[root@Server-A ~]# lsblk  -f
NAME   FSTYPE FSVER LABEL       	UUID                                 	 	 FSAVAIL FSUSE%	 MOUNTPOINTS
sda
├─sda1 swap   1                         	520bc18c-2b64-4df1-85e0-d126908ba6dd                	 [SWAP]
└─sda2 xfs                              	73bc277c-741d-4122-9c58-59ccd1889709	 10.4G	 35%	 /
sdb
└─sdb1 linux_ 1.2   Server-A:0	d6211e7b-fb04-7dcf-77a2-2771ee1c406c
  └─md0
       ext4   1.0                       	4efff958-7112-479f-a95a-0421f8f4ce4e
sdc
└─sdc1 linux_ 1.2   Server-A:0    	d6211e7b-fb04-7dcf-77a2-2771ee1c406c
  └─md0
       ext4   1.0                       	4efff958-7112-479f-a95a-0421f8f4ce4e
sdd
sde
sdf
```

---

```bash
-RAID 0 구성후 EXT4 형식으로 File system을 구성했지만 mount되지 않았기 때문에 사용할 수 없다.

EX3-2) RAID 0으로 구성한 HDD를 '/RAID0' 디렉터리로 mount 해야한다.

[root@Server-A ~]# mkdir  /RAID0
```

```bash
[root@Server-A ~]# mount  /dev/md0  /RAID0/
```

```bash
[root@Server-A ~]# mount | grep md0
/dev/md0 on /RAID0 type ext4 (rw,relatime,stripe=256)
```

```bash
[root@Server-A ~]# lsblk  -f
NAME   FSTYPE FSVER LABEL       	UUID                                 	 	 FSAVAIL FSUSE%	 MOUNTPOINTS
sda
├─sda1 swap   1                         	520bc18c-2b64-4df1-85e0-d126908ba6dd                	 [SWAP]
└─sda2 xfs                              	73bc277c-741d-4122-9c58-59ccd1889709	 10.4G	 35%	 /
sdb
└─sdb1 linux_ 1.2   Server-A:0	d6211e7b-fb04-7dcf-77a2-2771ee1c406c
  └─md0
       ext4   1.0                       	4efff958-7112-479f-a95a-0421f8f4ce4e			/RAID0
sdc
└─sdc1 linux_ 1.2   Server-A:0    	d6211e7b-fb04-7dcf-77a2-2771ee1c406c
  └─md0
       ext4   1.0                       	4efff958-7112-479f-a95a-0421f8f4ce4e			/RAID0
sdd
sde
sdf
```

---

```bash
-RAID 0을 구성했지만 Server-A가 재부팅시 mount는 해제된다.

[root@Server-A ~]# reboot
```

```bash
EX3-3) RAID 0으로 구성한 HDD는  Server-A가 재부팅되어도 바로 동작해야한다.

[root@Server-A ~]# RAID0=$(blkid -s UUID  -o value  /dev/md0)
```

```bash
[root@Server-A ~]# echo $RAID0
4efff958-7112-479f-a95a-0421f8f4ce4e
```

```bash
[root@Server-A ~]# cat  << EOF  >>  /etc/fstab
> UUID=$RAID0  /RAID0  ext4  defaults  0 0
> EOF
```

```bash
[root@Server-A ~]# cat  /etc/fstab
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
UUID=73bc277c-741d-4122-9c58-59ccd1889709    	/       	xfs   	defaults	0 0
UUID=520bc18c-2b64-4df1-85e0-d126908ba6dd  	none   	swap  	defaults	0 0
UUID=4efff958-7112-479f-a95a-0421f8f4ce4e  	/RAID0	ext4  	defaults	0 0
```

**EX3-4)** '/etc/*' 파일을  '/RAID0' 디렉터리로 복사해야한다.

```bash
[root@Server-A ~]# cp  -r  /etc/*  /RAID0/
```

```bash
[root@Server-A ~]# ls  /RAID0/
DIR_COLORS               	filesystems	magic                     	sasl2
DIR_COLORS.lightbgcolor  	firefox         	mailcap                   	security
GREP_COLORS              	firewalld       	makedumpfile.conf.sample  	selinux
NetworkManager           	flatpak         	man_db.conf               	services
PackageKit               	fonts           	mcelog                    	sestatus.conf
UPower                   	foomatic        	mdadm.conf                	setroubleshoot
X11                      	fprintd.conf    	microcode_ctl             	sgml
accountsservice          	fstab           	mime.types                	shadow
adjtime                  	fuse.conf       	mke2fs.conf               	shadow-
aliases                  	fwupd           	modprobe.d                	shells
	~~~~~~~~~~~~~ 중간 생략 ~~~~~~~~~~~~~
```

```bash
[root@Server-A ~]# shutdown now
```

- Server-A 전원 off  --->  VMware setting  --->  sdb 또는 sdc 제거

- '/dev/sdb' 또는 '/dev/sdc'를 제거한후 VMware를 다시 부팅하게되면 응급모드로 재부팅된다.
 (automount로 설정한 HDD가 인식되지 않기때문에 응급모드로 부팅되며 설정한 automount를 삭제시 정상 복구된다.)

```bash
You are in emergency mode. After logging in, type "journalctl -xb" to view
system logs, "systemctl reboot" to reboot, "systemctl default" or "exit"
to boot into default mode.
Give root password for maintenance
(or press Control-D to continue): admin1234	# root의 비밀번호 설정
```

```bash
[root@Server-A ~]# cat  /etc/fstab

#
# /etc/fstab
# Created by anaconda on Tue Nov 18 03:10:48 2025
#
# Accessible filesystems, by reference, are maintained under '/dev/disk/'.
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info.
#
# After editing this file, run 'systemctl daemon-reload' to update systemd
# units generated from this file.
#
UUID=660fb32e-8a83-417c-83a6-2782fb9deb6f 	/ 	xfs     	defaults	0 0
UUID=69f5bcdb-8929-4ee2-9efb-90ca4c1ed25c 	none	swap    	defaults	0 0
#UUID=1d05444c-e6a9-4996-80ed-b5aaa170e4c6	/RAID0 	ext4	defaults	0 0	# 주석 처리
```

```bash
[root@Server-A ~]# reboot
```

---

#### RAID0 복구

```bash
1) HDD 10G 추가

[root@Server-A ~]# lsblk  -f
NAME FSTYPE FSVER LABEL	UUID                        		FSAVAIL FSUSE%	MOUNTPOINTS
sda
├─sda1
│    swap   1               		520bc18c-2b64-4df1-85e0-d126908ba6dd                	[SWAP]
└─sda2
     xfs                    		73bc277c-741d-4122-9c58-59ccd1889709	10.4G    	35% 	/
sdb
└─sdb1
     linux_ 1.2   Server-A:0		d6211e7b-fb04-7dcf-77a2-2771ee1c406c
  └─md0

sdc	# RAID용으로 생성
sdd
sde
```

2) 추가된 HDD를  RAID용으로 구성

```bash
[root@Server-A ~]# fdisk  /dev/sdc

Welcome to fdisk (util-linux 2.37.4).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Device does not contain a recognized partition table.
Created a new DOS disklabel with disk identifier 0xd088dfb1.

Command (m for help): n
Partition type
   p   primary (0 primary, 0 extended, 4 free)
   e   extended (container for logical partitions)
Select (default p): p
Partition number (1-4, default 1): 1
First sector (2048-20971519, default 2048):
Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-20971519, default 20971519):

Created a new partition 1 of type 'Linux' and of size 10 GiB.

Command (m for help): t
Selected partition 1
Hex code or alias (type L to list all): L

00 Empty            		24 NEC DOS          		81 Minix / old Lin  		bf Solaris
01 FAT12            		27 Hidden NTFS Win	82 Linux swap / So  	c1 DRDOS/sec (FAT-
02 XENIX root       		39 Plan 9           		83 Linux            		c4 DRDOS/sec (FAT-
03 XENIX usr        		3c PartitionMagic   		84 OS/2 hidden or   		c6 DRDOS/sec (FAT-
04 FAT16 <32M       	40 Venix 80286      		85 Linux extended   		c7 Syrinx
05 Extended         		41 PPC PReP Boot    	86 NTFS volume set  	da Non-FS data
06 FAT16            		42 SFS              		87 NTFS volume set  	db CP/M / CTOS / .
07 HPFS/NTFS/exFAT  	4d QNX4.x           		88 Linux plaintext  		de Dell Utility
08 AIX              		4e QNX4.x 2nd part		8e Linux LVM        		df BootIt
09 AIX bootable     		4f QNX4.x 3rd part  		93 Amoeba           		e1 DOS access
0a OS/2 Boot Manag  	50 OnTrack DM       	94 Amoeba BBT       	e3 DOS R/O
0b W95 FAT32        	51 OnTrack DM6 Aux  	9f BSD/OS           		e4 SpeedStor
0c W95 FAT32 (LBA)  	52 CP/M             		a0 IBM Thinkpad hi  	ea Linux extended
0e W95 FAT16 (LBA)  	53 OnTrack DM6 Aux  	a5 FreeBSD          		eb BeOS fs
0f W95 Ext'd (LBA)  	54 OnTrackDM6       	a6 OpenBSD          		ee GPT
10 OPUS             		55 EZ-Drive         		a7 NeXTSTEP         	ef EFI (FAT-12/16/
11 Hidden FAT12     	56 Golden Bow       	a8 Darwin UFS       		f0 Linux/PA-RISC b
12 Compaq diagnost  	5c Priam Edisk      		a9 NetBSD           		f1 SpeedStor
14 Hidden FAT16 <3  	61 SpeedStor        		ab Darwin boot      		f4 SpeedStor
16 Hidden FAT16     	63 GNU HURD or Sys  	af HFS / HFS+       		f2 DOS secondary
17 Hidden HPFS/NTF  	64 Novell Netware   	b7 BSDI fs          		fb VMware VMFS
18 AST SmartSleep   	65 Novell Netware   	b8 BSDI swap       	 	fc VMware VMKCORE
1b Hidden W95 FAT3  	70 DiskSecure Mult  	bb Boot Wizard hid  	fd Linux raid auto
1c Hidden W95 FAT3  	75 PC/IX            		bc Acronis FAT32 L  	fe LANstep
1e Hidden W95 FAT1  	80 Old Minix        		be Solaris boot     		ff BBT

Aliases:
   linux          - 83
   swap           - 82
   extended       - 05
   uefi           - EF
   raid           - FD
   lvm            - 8E
   linuxex        - 85
Hex code or alias (type L to list all): fd
Changed type of partition 'Linux raid autodetect' to 'Linux raid autodetect'.

Command (m for help): p
Disk /dev/sdc: 10 GiB, 10737418240 bytes, 20971520 sectors
Disk model: VMware Virtual S
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xd088dfb1

Device     Boot Start      End  Sectors Size Id Type
/dev/sdc1        2048 20971519 20969472  10G fd Linux raid autodetect

Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.
```

3) 설정된 mdadm의 사용을 중지

```bash
[root@Server-A ~]# mdadm  --stop  /dev/md0
mdadm: stopped /dev/md0
```

4) 디스크에 남아있는 superblock 삭제
- 디스크에 남아 있는 RAID 메타데이터(superblock)를 삭제
- superblock = RAID 구성 정보 (ex: RAID 레벨, UUID, member 번호, 생성 시간 등)
- 디스크에 한번이라도 RAID를 만든 적이 있다면 sdb1, sdc1 같은 파티션에는 RAID superblock이 항상 남아있다.

```bash
[root@Server-A ~]# mdadm  --zero-superblock  /dev/sdb1
[root@Server-A ~]# mdadm  --zero-superblock  /dev/sdc1

 		OR

[root@Server-A ~]# mdadm  --zero-superblock  /dev/{sdb1,sdc1}
```

```bash
5) RAID 0 재설정

[root@Server-A ~]# mdadm  --create  /dev/md0  --level=0  --raid-devices=2  /dev/sdb1  /dev/sdc1
mdadm: Defaulting to version 1.2 metadata
mdadm: array /dev/md0 started.
```

```bash
[root@Server-A ~]# mdadm  --detail  /dev/md0
/dev/md0:
           Version 	: 1.2
     Creation Time 	: Tue Jul 14 11:15:05 2026
        Raid Level 	: raid0
        Array Size 	: 20951040 (19.98 GiB 21.45 GB)
      Raid Devices 	: 2
     Total Devices 	: 2
       Persistence 	: Superblock is persistent

       Update Time	: Tue Jul 14 11:15:05 2026
             State 	: clean
    Active Devices 	: 2
   Working Devices	: 2
    Failed Devices 	: 0
     Spare Devices 	: 0

            Layout 	: original
        Chunk Size 	: 512K

Consistency Policy : none

              Name 	: Server-A:0  (local to host Server-A)
              UUID 	: d40aa2a1:7730caea:636be67a:efc318f2
            Events 	: 0

    Number   Major   Minor   RaidDevice State
       0          8         17        0            active sync   /dev/sdb1
       1          8         33        1            active sync   /dev/sdc1
```

```bash
6) EXT4 형식으로 포맷

[root@Server-A ~]# mkfs.ext4  /dev/md0
mke2fs 1.46.5 (30-Dec-2021)
Creating filesystem with 5237760 4k blocks and 1310720 inodes
Filesystem UUID: 0dde3d6f-b13a-44d6-8deb-6f9eef7893be
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208,
        4096000

Allocating group tables: done
Writing inode tables: done
Creating journal (32768 blocks): done
Writing superblocks and filesystem accounting information: done
```

```bash
[root@Server-A ~]# lsblk  -f
NAME    FSTYPE      FSVER LABEL  	UUID                                 		 FSAVAIL	 FSUSE%	 MOUNTPOINTS
sda
├─sda1  swap        1                      	520bc18c-2b64-4df1-85e0-d126908ba6dd                	 [SWAP]
└─sda2  xfs                                    	73bc277c-741d-4122-9c58-59ccd1889709	 10.4G	 35%	 /
sdb
└─sdb1  linux_raid_ 1.2   Server-A:0	d40aa2a1-7730-caea-636b-e67aefc318f2
  └─md0 ext4        1.0                   	0dde3d6f-b13a-44d6-8deb-6f9eef7893be
sdc
└─sdc1  linux_raid_ 1.2   Server-A:0  	d40aa2a1-7730-caea-636b-e67aefc318f2
  └─md0 ext4        1.0                   	0dde3d6f-b13a-44d6-8deb-6f9eef7893be
sdd
sde
```

```bash
7) 마운트 및 오토마운트 재설정

[root@Server-A ~]# mount  /dev/md0  /RAID0/
```

```bash
[root@Server-A ~]# mount | grep md0
/dev/md0 on /RAID0 type ext4 (rw,relatime,stripe=256)
```

```bash
[root@Server-A ~]# RAID0=$(blkid -s  UUID  -o  value  /dev/md0)
```

```bash
[root@Server-A ~]# echo $RAID0
0dde3d6f-b13a-44d6-8deb-6f9eef7893be
```

```bash
[root@Server-A ~]# cat  << EOF  >>  /etc/fstab
> UUID=$RAID0  /RAID0  ext4  defaults  0 0
> EOF

	~~~~~~~~~~~~~ OR ~~~~~~~~~~~~~
```

```bash
[root@Server-A ~]# vi /etc/fstab
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
UUID=73bc277c-741d-4122-9c58-59ccd1889709       /        	xfs     	defaults	0 0
UUID=520bc18c-2b64-4df1-85e0-d126908ba6dd       none    	swap    	defaults	0 0
UUID=0dde3d6f-b13a-44d6-8deb-6f9eef7893be  	/RAID0	ext4  	defaults	0 0	# UUID 수정
```

```bash
[root@Server-A ~]# ls  -l  /RAID0/
합계 16
drwx------ 2 root root 16384  7월 14 11:17 lost+found
```

---

**정리**: RAID 0 실습 (8-3) 섹션에서 다룬 명령어와 실습 예제를 통해 관련 기능의 구성 방법과 동작 원리를 확인했다.

## RAID 1 실습 (8-4)

#### RAID 5

	
- RAID 중에서 실무에서 많이 사용되는 방식 중 하나이다.

- RAID 0의 공간 효율성과 읽기 성능, RAID 1처럼 디스크 장애 1개까지 버틸 수 있는 안정성을 절충한 구조이다.

- 최소 3개 이상의 HDD를 사용해야 한다.

- 데이터를 블록 단위로 잘라 여러 HDD에 나누어 저장하며(스트라이핑), 각 블록에 대한 패리티(Parity) 정보를 나머지 디스크에 분산 저장한다.

- 안정성을 위해 패리티(Parity) 정보를 사용한다.
  - 특정 HDD가 장애가 나면, 남은 데이터 블록 + 패리티 정보를 이용해 장애 디스크의 데이터를 재구성(복구)할 수 있다.

- Fault-tolerance(결함 허용)를 제공하면서도 공간 효율성이 좋은 편이다.

- 공간 효율성(사용 가능한 용량)
  - 1TB HDD 3개 사용 시 	: 2TB 사용 가능 (약66%)
  - 1TB HDD 4개 사용 시 	: 3TB 사용 가능 (75%)
  - 1TB HDD 5개 사용 시 	: 4TB 사용 가능 (80%)
  - 1TB HDD 10개 사용 시	: 9TB 사용 가능 (90%)
  - 디스크 개수가 많아질수록 공간 효율성은 100%에 가까워진다.

- 제약 및 주의사항
  - HDD가 한 개 고장 나는 것은 패리티로 복구 가능하지만,
   복구(리빌드)가 끝나기 전에 또 다른 HDD가 추가로 고장 나면 데이터를 복구할 수 없다.
  - 패리티를 계산하고 쓰는 과정 때문에, 일반적으로 읽기 성능은 좋지만
   쓰기 성능(특히 랜덤 쓰기)은 RAID 0, RAID 10보다 느릴 수 있다.
  - 소규모, 중규모의 스토리지에서 용량, 성능, 안정성을 적당히 모두 고려할 때 많이 사용된다.

	Data = 000 001 010 011

	   	   X
	HDD1	HDD2	HDD3	HDD4  =	HDD2
	   0	   0 	   0	   P0	   0
	   0	   0	   P1	   1	   0
	   0	   P1	   1	   0	   1
	   P0	   0	   1	   1	   0

---

#### 기존 디스크 초기화

```bash
	# 현재 /dev/md0 RAID 장치가 /linear 디렉터리에 마운트되어 있는지 확인

[root@Server-A ~]# mount | grep md0
/dev/md0 on /linear type ext4 (rw,relatime)
```

```bash
	# /linear에 마운트된 RAID 파일시스템을 마운트 해제

[root@Server-A ~]# umount /RAID10
```

```bash
	# /dev/md0에 생성된 파일시스템 서명 삭제

[root@Server-A ~]# wipefs -a -f /dev/md10
/dev/md10: 2 bytes were erased at offset 0x00000438 (ext4): 53 ef
```

- wipefs	: 디스크나 파티션에 기록된 파일시스템, RAID, LVM 등의 서명(Signature)을 확인하거나 삭제하는 명령어
- -a	: all의 약자로 대상 장치에서 발견되는 모든 서명을 삭제
- -f	: force의 약자로 장치가 파티션이 아닌 전체 디스크이거나 RAID 같은 특수 블록 장치여도 강제로 서명 삭제

```bash
	# 활성화되어 있는 /dev/md0 RAID 장치를 중지  (RAID 구성을 삭제하는 것이 아니라 현재 조립된 RAID의 동작만 중지)

[root@Server-A ~]# mdadm  --stop  /dev/md10
```

```bash
	# /dev/sdb1과 /dev/sdc1에 기록된 RAID 메타데이터(Superblock)를 삭제 (`RAID 레벨, RAID UUID, 디스크 순서, 구성 정보 등을 제거)

[root@Server-A ~]# mdadm --zero-superblock /dev/sdb1
[root@Server-A ~]# mdadm --zero-superblock /dev/sdc1
[root@Server-A ~]# mdadm --zero-superblock /dev/sdd1
[root@Server-A ~]# mdadm --zero-superblock /dev/sde1

        OR

[root@Server-A ~]# mdadm --zero-superblock --force /dev/sdb1
[root@Server-A ~]# mdadm --zero-superblock --force /dev/sdc1
[root@Server-A ~]# mdadm --zero-superblock --force /dev/sdd1
[root@Server-A ~]# mdadm --zero-superblock --force /dev/sde1
```

```bash
	# /dev/sdb와 /dev/sdc 디스크 전체에서 발견되는 모든 서명을 삭제 (디스크 전체에 실행하므로 sdb1, sdc1 파티션 정보까지 삭제)

[root@Server-A ~]# wipefs -a /dev/sdb
/dev/sdb: 2 bytes were erased at offset 0x000001fe (dos): 55 aa
/dev/sdb: calling ioctl to re-read partition table: 성공
```

```bash
[root@Server-A ~]# wipefs -a /dev/sdc
/dev/sdc: 2 bytes were erased at offset 0x000001fe (dos): 55 aa
/dev/sdc: calling ioctl to re-read partition table: 성공
```

```bash
[root@Server-A ~]# wipefs -a /dev/sdd
/dev/sdd: 2 bytes were erased at offset 0x000001fe (dos): 55 aa
/dev/sdd: calling ioctl to re-read partition table: 성공
```

```bash
[root@Server-A ~]# wipefs -a /dev/sde
/dev/sde: 2 bytes were erased at offset 0x000001fe (dos): 55 aa
/dev/sde: calling ioctl to re-read partition table: 성공
```

---

**EX1)** Server-A에 장착된 SDD에 Patition을 구성
  - 추가로 장착한 sdb를 하나의 Patition으로 구성해야한다.
  - 추가로 장착한 sdc를 하나의 Patition으로 구성해야한다.
  - 추가로 장착한 sdd를 하나의 Patition으로 구성해야한다.
  - 추가로 장착한 sde를 하나의 Patition으로 구성해야한다.
  - sdb , sdc , sdd , sde Patition 구성시 RAID용으로 구성해야한다.

```bash
[root@Server-A ~]# ls  -l  /dev/sd*
brw-rw---- 1 root disk 8,  0  7월 14 12:50 /dev/sda
brw-rw---- 1 root disk 8,  1  7월 14 12:50 /dev/sda1
brw-rw---- 1 root disk 8,  2  7월 14 12:50 /dev/sda2
brw-rw---- 1 root disk 8, 16  7월 14 14:38 /dev/sdb
brw-rw---- 1 root disk 8, 17  7월 14 14:38 /dev/sdb1
brw-rw---- 1 root disk 8, 32  7월 14 14:38 /dev/sdc
brw-rw---- 1 root disk 8, 33  7월 14 14:38 /dev/sdc1
brw-rw---- 1 root disk 8, 48  7월 14 14:38 /dev/sdd
brw-rw---- 1 root disk 8, 49  7월 14 14:38 /dev/sdd1
brw-rw---- 1 root disk 8, 64  7월 14 14:38 /dev/sde
brw-rw---- 1 root disk 8, 65  7월 14 14:38 /dev/sde1
```

```bash
[root@Server-A ~]# lsblk
NAME   MAJ:MIN RM  SIZE RO TYPE 	MOUNTPOINTS
sda      	  8:0    	0   20G  	0 disk
├─sda1	  8:1    	0    4G  	0 part 	[SWAP]
└─sda2	  8:2    	0   16G  	0 part 	/
sdb      	  8:16   	0   10G  	0 disk
└─sdb1	  8:17   	0   10G  	0 part
sdc      	  8:32   	0   10G  	0 disk
└─sdc	  8:33   	0   10G  	0 part
sdd      	  8:48   	0   10G  	0 disk
└─sdd1	  8:49   	0   10G  	0 part
sde      	  8:64   	0   10G  	0 disk
└─sde1	  8:65   	0   10G  	0 part
```

---

**EX2)** Patiton을 구성한 sdb1 , sdc1 , sdd1 , sde1을 RAID 5로 구성해야한다.

```bash
[root@Server-A ~]# rpm  -qa  | grep mdadm
mdadm-4.4-4.el9_7.x86_64
```

```bash
[root@Server-A ~]# mdadm  --create  /dev/md5  --level=5  --raid-devices=4  /dev/sdb1  /dev/sdc1  /dev/sdd1  /dev/sde1
To optimalize recovery speed, it is recommended to enable write-indent bitmap, do you want to enable it now? [y/N]? y
mdadm: Defaulting to version 1.2 metadata
mdadm: array /dev/md5 started.
```

```bash
[root@Server-A ~]# mdadm  --detail  --scan
ARRAY /dev/md5 metadata=1.2 spares=1 UUID=d2e3cf93:8543570c:67ee7c11:52a37b4e
```

```bash
[root@Server-A ~]# mdadm  --detail  /dev/md5
/dev/md5:
           Version 	: 1.2
     Creation Time 	: Tue Jul 14 14:44:17 2026
        Raid Level 	: raid5
        Array Size 	: 31426560 (29.97 GiB 32.18 GB)
     Used Dev Size 	: 10475520 (9.99 GiB 10.73 GB)
      Raid Devices 	: 4
     Total Devices 	: 4
       Persistence 	: Superblock is persistent

     Intent Bitmap 	: Internal

       Update Time 	: Tue Jul 14 14:45:09 2026
             State 	: clean
    Active Devices 	: 4
   Working Devices 	: 4
    Failed Devices 	: 0
     Spare Devices 	: 0

            Layout 	: left-symmetric
        Chunk Size 	: 512K

Consistency Policy	: bitmap

              Name 	: Server-A:5  (local to host Server-A)
              UUID 	: d2e3cf93:8543570c:67ee7c11:52a37b4e
            Events 	: 22

    Number   Major   Minor   RaidDevice State
       0       8       17        0      active sync   /dev/sdb1
       1       8       33        1      active sync   /dev/sdc1
       2       8       49        2      active sync   /dev/sdd1
       4       8       65        3      active sync   /dev/sde1
```

```bash
[root@Server-A ~]# lsblk  -f
NAME    FSTYPE          FSVER LABEL          	UUID                             		 FSAVAIL	 FSUSE%	 MOUNTPOINTS
sda
├─sda1  swap            1                          	520bc18c-2b64-4df1-85e0-d126908ba6dd                	 [SWAP]
└─sda2  xfs                                        	73bc277c-741d-4122-9c58-59ccd1889709	 10.4G	 35%	 /
sdb
└─sdb1  linux_raid_memb 1.2   Server-A:5         	d2e3cf93-8543-570c-67ee-7c1152a37b4e
  └─md5
sdc
└─sdc1  linux_raid_memb 1.2   Server-A:5       	d2e3cf93-8543-570c-67ee-7c1152a37b4e
  └─md5
sdd
└─sdd1  linux_raid_memb 1.2   Server-A:5        	d2e3cf93-8543-570c-67ee-7c1152a37b4e
  └─md5
sde
└─sde1  linux_raid_memb 1.2   Server-A:5      	d2e3cf93-8543-570c-67ee-7c1152a37b4e
  └─md5
```

---

- RAID 5 구성후 EXT4 형식으로 File system을 구성했지만 mount되지 않았기 때문에 사용할 수 없다.

**EX4)** RAID 5로 구성한 HDD를 /RAID 5로 mount 해야한다

```bash
[root@Server-A ~]# mkfs.ext4  /dev/md5
mke2fs 1.46.5 (30-Dec-2021)
Creating filesystem with 7856640 4k blocks and 1966080 inodes
Filesystem UUID: 70dcc674-a327-4e90-8064-f12dc3499bd9
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208,
        4096000

Allocating group tables: done
Writing inode tables: done
Creating journal (32768 blocks): done
Writing superblocks and filesystem accounting information: done
```

```bash
[root@Server-A ~]# mkdir  /RAID5
```

```bash
[root@Server-A ~]# mount  /dev/md5  /RAID5
```

```bash
[root@Server-A ~]# lsblk  -f
NAME    FSTYPE          FSVER LABEL                UUID                                 	 FSAVAIL	 FSUSE%	 MOUNTPOINTS
sda
├─sda1  swap            1                          	520bc18c-2b64-4df1-85e0-d126908ba6dd                	 [SWAP]
└─sda2  xfs                                        	73bc277c-741d-4122-9c58-59ccd1889709	 10.4G	 35%	 /
sdb
└─sdb1  linux_raid_memb 1.2   Server-A:5     	d2e3cf93-8543-570c-67ee-7c1152a37b4e
  └─md5 ext4            1.0                        	70dcc674-a327-4e90-8064-f12dc3499bd9
sdc
└─sdc1  linux_raid_memb 1.2   Server-A:5      	d2e3cf93-8543-570c-67ee-7c1152a37b4e
  └─md5 ext4            1.0                        	70dcc674-a327-4e90-8064-f12dc3499bd9
sdd
└─sdd1  linux_raid_memb 1.2   Server-A:5     	d2e3cf93-8543-570c-67ee-7c1152a37b4e
  └─md5 ext4            1.0                        	70dcc674-a327-4e90-8064-f12dc3499bd9
sde
└─sde1  linux_raid_memb 1.2   Server-A:5   	d2e3cf93-8543-570c-67ee-7c1152a37b4e
  └─md5 ext4            1.0                        	70dcc674-a327-4e90-8064-f12dc3499bd9
```

---

- RAID 5 구성후 EXT4 형식으로 File system을 구성했지만 mount되지 않았기 때문에 사용할 수 없다.

**EX4)** RAID 5로 구성한 HDD를 /RAID 5로 mount 해야한다.

```bash
[root@Server-A ~]# UUID1=$(blkid  -s  UUID  -o  value  /dev/md5)
```

```bash
[root@Server-A ~]# echo $UUID1
70dcc674-a327-4e90-8064-f12dc3499bd9
```

```bash
[root@Server-A ~]# cat  <<EOF  >>  /etc/fstab
> UUID=$UUID1  /RAID5  ext4  defaults  0 0
> EOF
```

```bash
[root@Server-A ~]# cat /etc/fstab

#
# /etc/fstab
# Created by anaconda on Tue Nov 18 03:10:48 2025
#
# Accessible filesystems, by reference, are maintained under '/dev/disk/'.
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info.
#
# After editing this file, run 'systemctl daemon-reload' to update systemd
# units generated from this file.
#
UUID=660fb32e-8a83-417c-83a6-2782fb9deb6f 	/	xfs     	defaults	0 0
UUID=69f5bcdb-8929-4ee2-9efb-90ca4c1ed25c 	none	swap    	defaults	0 0
UUID=70dcc674-a327-4e90-8064-f12dc3499bd9	/RAID5	ext4	defaults	0 0	# 오토마운트 설정
```

---

**EX6)** '/etc/' 디렉터리안의 파일중 c로 시작하는 모든 파일을 '/RAID5' 디렉터리로 복사해야한다.

```bash
[root@Server-A ~]# cp  -r  /etc/c*  /RAID5
```

```bash
[root@Server-A ~]# ls  -l  /RAID5
합계 80
drwxr-xr-x 3 root root  4096  7월 14 14:53 chromium
-rw-r--r-- 1 root root  1370  7월 14 14:53 chrony.conf
-rw-r----- 1 root root   540  7월 14 14:53 chrony.keys
drwxr-xr-x 2 root root  4096  7월 14 14:53 cifs-utils
drwxr-xr-x 4 root root  4096  7월 14 14:53 cockpit
drwxr-xr-x 2 root root  4096  7월 14 14:53 cron.d
drwxr-xr-x 2 root root  4096  7월 14 14:53 cron.daily
-rw-r--r-- 1 root root       0  7월 14 14:53 cron.deny
drwxr-xr-x 2 root root  4096  7월 14 14:53 cron.hourly
drwxr-xr-x 2 root root  4096  7월 14 14:53 cron.monthly
drwxr-xr-x 2 root root  4096  7월 14 14:53 cron.weekly
-rw-r--r-- 1 root root    451  7월 14 14:53 crontab
drwxr-xr-x 6 root root  4096  7월 14 14:53 crypto-policies
-rw------- 1 root root      0  7월 14 14:53 crypttab
-rw-r--r-- 1 root root  1401  7월 14 14:53 csh.cshrc
-rw-r--r-- 1 root root  1112  7월 14 14:53 csh.login
drwxr-xr-x 4 root root  4096  7월 14 14:53 cups
drwxr-xr-x 2 root root  4096  7월 14 14:53 cupshelpers
drwx------ 2 root root 16384  7월 14 14:47 lost+found
```

---

- 현재 복사된 파일들은 '/RAID5' 디렉터리에 저장된다.
- '/dev/sdb1' , '/dev/sdc1'  , '/dev/sdd1'  , '/dev/sdd1' 에서 페리티 비트를 사용하기때문에 결함허용을 지원한다.
  즉 어떤 HDD에 장애가 발생해도 파일은 정상적으로 확인된다.

```bash
[root@Server-A ~]# mdadm  --fail  /dev/md5  /dev/sdc1		# 논리적인 장애로 인식

		OR

[root@Server-A ~]# mdadm  --remove  /dev/md5  /dev/sdc1	# 실제 RAID에서 제거
```

```bash
[root@Server-A ~]# mdadm  --detail  /dev/md5
/dev/md5:
           Version 	: 1.2
     Creation Time 	: Tue Jul 14 14:44:17 2026
        Raid Level 	: raid5
        Array Size 	: 31426560 (29.97 GiB 32.18 GB)
     Used Dev Size 	: 10475520 (9.99 GiB 10.73 GB)
      Raid Devices 	: 4
     Total Devices 	: 4
       Persistence 	: Superblock is persistent

     Intent Bitmap 	: Internal

       Update Time 	: Tue Jul 14 14:55:48 2026
             State 	: clean, degraded
    Active Devices 	: 3
   Working Devices 	: 3
    Failed Devices 	: 1
     Spare Devices 	: 0

            Layout 	: left-symmetric
        Chunk Size 	: 512K

Consistency Policy	: bitmap

              Name : Server-A:5  (local to host Server-A)
              UUID : d2e3cf93:8543570c:67ee7c11:52a37b4e
            Events : 24

    Number   Major   Minor   RaidDevice State
       0         8         17      	0      	active sync    /dev/sdb1
       -         0          0      	1      	removed
       2         8         49      	2      	active sync    /dev/sdd1
       4         8         65       	3      	active sync    /dev/sde1
       1         8         33        	-      	faulty	     /dev/sdc1
```

```bash
[root@Server-A ~]# ls  -l  /RAID5
합계 80
drwxr-xr-x 3 root root  4096  7월 14 14:53 chromium
-rw-r--r-- 1 root root  1370  7월 14 14:53 chrony.conf
-rw-r----- 1 root root   540  7월 14 14:53 chrony.keys
drwxr-xr-x 2 root root  4096  7월 14 14:53 cifs-utils
drwxr-xr-x 4 root root  4096  7월 14 14:53 cockpit
drwxr-xr-x 2 root root  4096  7월 14 14:53 cron.d
drwxr-xr-x 2 root root  4096  7월 14 14:53 cron.daily
-rw-r--r-- 1 root root      0  7월 14 14:53 cron.deny
drwxr-xr-x 2 root root  4096  7월 14 14:53 cron.hourly
drwxr-xr-x 2 root root  4096  7월 14 14:53 cron.monthly
drwxr-xr-x 2 root root  4096  7월 14 14:53 cron.weekly
-rw-r--r-- 1 root root   451  7월 14 14:53 crontab
drwxr-xr-x 6 root root  4096  7월 14 14:53 crypto-policies
-rw------- 1 root root      0  7월 14 14:53 crypttab
-rw-r--r-- 1 root root  1401  7월 14 14:53 csh.cshrc
-rw-r--r-- 1 root root  1112  7월 14 14:53 csh.login
drwxr-xr-x 4 root root  4096  7월 14 14:53 cups
drwxr-xr-x 2 root root  4096  7월 14 14:53 cupshelpers
drwx------ 2 root root 16384  7월 14 14:47 lost+found
```

```bash
[root@Server-A ~]# mdadm  --add  /dev/md5   /dev/sdc1	# superblock이 삭제되지 않으면 add 명령어로 다시 추가 가능
```

---

#### Disk 초기화

**EX7)** '/dev/sdb',  '/dev/sdc',  '/dev/sdd',  '/dev/sde',  '/dev/sdf'를 사용해서 RAID 5를 구성해야 한다. (10G HDD 1개 추가 : 총 5개)
       '/dev/sdb',  '/dev/sdc',  '/dev/sdd',  '/dev/sde'는 실제 동작하는 HDD로 사용하고
       '/dev/sde'는 '/dev/sdb',  '/dev/sdc',  '/dev/sdd' HDD에 장애 발생시 바로 RAID 5로 동작해야 한다.

#### 기존 디스크 초기화

```bash
	# 현재 /dev/md0 RAID 장치가 /linear 디렉터리에 마운트되어 있는지 확인

[root@Server-A ~]# mount | grep md0
/dev/md0 on /linear type ext4 (rw,relatime)
```

```bash
	# /linear에 마운트된 RAID 파일시스템을 마운트 해제

[root@Server-A ~]# umount /RAID5
```

```bash
	# /dev/md0에 생성된 파일시스템 서명 삭제

[root@Server-A ~]# wipefs -a -f /dev/md5
/dev/md10: 2 bytes were erased at offset 0x00000438 (ext4): 53 ef
```

- wipefs	: 디스크나 파티션에 기록된 파일시스템, RAID, LVM 등의 서명(Signature)을 확인하거나 삭제하는 명령어
- -a	: all의 약자로 대상 장치에서 발견되는 모든 서명을 삭제
- -f	: force의 약자로 장치가 파티션이 아닌 전체 디스크이거나 RAID 같은 특수 블록 장치여도 강제로 서명 삭제

```bash
	# 활성화되어 있는 /dev/md0 RAID 장치를 중지  (RAID 구성을 삭제하는 것이 아니라 현재 조립된 RAID의 동작만 중지)

[root@Server-A ~]# mdadm  --stop  /dev/md5
```

```bash
	# /dev/sdb1과 /dev/sdc1에 기록된 RAID 메타데이터(Superblock)를 삭제 (`RAID 레벨, RAID UUID, 디스크 순서, 구성 정보 등을 제거)

[root@Server-A ~]# mdadm --zero-superblock /dev/sdb1
[root@Server-A ~]# mdadm --zero-superblock /dev/sdc1
[root@Server-A ~]# mdadm --zero-superblock /dev/sdd1
[root@Server-A ~]# mdadm --zero-superblock /dev/sde1

        OR

[root@Server-A ~]# mdadm --zero-superblock --force /dev/sdb1
[root@Server-A ~]# mdadm --zero-superblock --force /dev/sdc1
[root@Server-A ~]# mdadm --zero-superblock --force /dev/sdd1
[root@Server-A ~]# mdadm --zero-superblock --force /dev/sde1
```

```bash
	# /dev/sdb와 /dev/sdc 디스크 전체에서 발견되는 모든 서명을 삭제 (디스크 전체에 실행하므로 sdb1, sdc1 파티션 정보까지 삭제)

[root@Server-A ~]# wipefs -a /dev/sdb
/dev/sdb: 2 bytes were erased at offset 0x000001fe (dos): 55 aa
/dev/sdb: calling ioctl to re-read partition table: 성공
```

```bash
[root@Server-A ~]# wipefs -a /dev/sdc
/dev/sdc: 2 bytes were erased at offset 0x000001fe (dos): 55 aa
/dev/sdc: calling ioctl to re-read partition table: 성공
```

```bash
[root@Server-A ~]# wipefs -a /dev/sdd
/dev/sdd: 2 bytes were erased at offset 0x000001fe (dos): 55 aa
/dev/sdd: calling ioctl to re-read partition table: 성공
```

```bash
[root@Server-A ~]# wipefs -a /dev/sde
/dev/sde: 2 bytes were erased at offset 0x000001fe (dos): 55 aa
/dev/sde: calling ioctl to re-read partition table: 성공
```

---

```bash
EX7) '/dev/sdb',  '/dev/sdc',  '/dev/sdd',  '/dev/sde',  '/dev/sdf'를 사용해서 RAID 5를 구성해야 한다. (10G HDD 1개 추가 : 총 5개)
       '/dev/sdb',  '/dev/sdc',  '/dev/sdd',  '/dev/sde'는 실제 동작하는 HDD로 사용하고  '/dev/sdf'는 
       '/dev/sdb',  '/dev/sdc',  '/dev/sdd',  '/dev/sde' HDD에 장애 발생시 바로 RAID 5로 동작해야 한다.

[root@Server-A ~]# lsblk
NAME   MAJ:MIN RM  SIZE RO TYPE 	MOUNTPOINTS
sda      	  8:0    	0   20G  	0 disk
├─sda1	  8:1    	0    4G  	0 part 	[SWAP]
└─sda2	  8:2    	0   16G  	0 part 	/
sdb      	  8:16   	0   10G  	0 disk
sdc      	  8:32   	0   10G  	0 disk
sdd      	  8:48   	0   10G  	0 disk
sde      	  8:64   	0   10G  	0 disk
sdf      	  8:80   	0   10G  	0 disk
```

```bash
[root@Server-A ~]# fdisk  /dev/sdb  (/dev/sdc  /dev/sdd/  dev/sde  dev/sdf)
Command (m for help): n
Select (default p):p
Partition number (1-4, default 1):1
First sector (2048-20971519, default 2048):
Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-20971519, default 20971519):
Command (m for help): t
Hex code or alias (type L to list all): fd
Command (m for help): w
```

```bash
[root@Server-A ~]# lsblk
NAME   MAJ:MIN RM  SIZE RO TYPE 	MOUNTPOINTS
sda      	  8:0    	0   20G  	0 disk
├─sda1	  8:1    	0    4G  	0 part 	[SWAP]
└─sda2	  8:2    	0   16G  	0 part 	/
sdb      	  8:16   	0   10G  	0 disk
└─sdb1	  8:17   	0   10G  	0 part
sdc      	  8:32   	0   10G  	0 disk
└─sdc1	  8:33   	0   10G  	0 part
sdd      	  8:48  	0   10G  	0 disk
└─sdd1	  8:49   	0   10G  	0 part
sde      	  8:64   	0   10G  	0 disk
└─sde1	  8:65   	0   10G  	0 part
sdf      	  8:80   	0   10G  	0 disk
└─sdf1	  8:81   	0   10G  	0 part
```

```bash
[root@Server-A ~]# mdadm  --create   /dev/md5  \
  --level=5  --raid-devices=4  \
  /dev/sdb1  /dev/sdc1  /dev/sdd1  /dev/sde1  \
  --spare-devices=1  /dev/sdf1

To optimalize recovery speed, it is recommended to enable write-indent bitmap, do you want to enable it now? [y/N]? y
mdadm: Defaulting to version 1.2 metadata
mdadm: array /dev/md5 started.
```

```bash
[root@Server-A ~]# mdadm --detail /dev/md5

/dev/md5:                                                      	# 확인 대상 RAID 장치 이름
           Version 	 1.2                                       	# RAID 메타데이터(Superblock) 버전
     Creation Time 	: Tue Jul 14 15:33:05 2026                  	# RAID가 생성된 날짜와 시간
        Raid Level 	: raid5                                     	# RAID 레벨: RAID 5
        Array Size 	: 31426560 (29.97 GiB 32.18 GB)       	# RAID 전체에서 실제 사용할 수 있는 논리적 용량
     Used Dev Size 	: 10475520 (9.99 GiB 10.73 GB)         	# 각 구성 디스크 1개에서 RAID가 사용하는 용량
      Raid Devices 	: 4                                         	# RAID 5를 구성하도록 지정된 활성 디스크 슬롯 수
     Total Devices 	: 5                                         	# 현재 RAID에 연결된 전체 디스크 수(Active + Spare)
       Persistence 	: Superblock is persistent                  	# RAID 구성 정보가 각 디스크의 Superblock에 영구 저장됨

     Intent Bitmap 	: Internal                                  	# 변경된 영역을 기록하는 Write-intent bitmap을 RAID 내부에 저장

       Update Time 	: Tue Jul 14 15:33:28 2026                  	# RAID 상태 정보가 마지막으로 갱신된 시간
             State 	: clean, degraded, recovering               	# 데이터는 일관성 있지만 디스크 1개가 부족하여 복구 중
    Active Devices 	: 4                                         	# 현재 정상적으로 RAID 데이터 저장에 참여 중인 디스크 수
   Working Devices 	: 5                                         	# 정상 인식되는 전체 디스크 수(Active + Rebuilding + Spare)
    Failed Devices 	: 0                                         	# Failed 상태로 등록된 디스크 수
     Spare Devices 	: 1                                         	# 현재 Spare 계열 상태인 디스크 수
                                                                		# 복구 중인 sde1과 대기 중인 sdf1이 포함됨

            Layout 	: left-symmetric                            	# RAID 5의 데이터와 패리티 배치 방식
        Chunk Size 	: 512K                                      	# 한 디스크에 연속으로 기록하는 데이터 조각 크기

Consistency Policy	: bitmap                                    	# Bitmap을 사용하여 변경 영역 중심으로 일관성 복구

              Name : Server-A:5 (local to host Server-A)       	# RAID 배열 이름
              UUID : 6795f6ac:fff1db57:3c2d92aa:99de3b4e 
            Events : 10

    Number   Major   Minor   RaidDevice State                   	# 각 디스크의 장치 번호, RAID 슬롯 및 상태

       0       8       17        0      active sync /dev/sdb1  	# RAID 슬롯 0에서 정상 동작 중인 구성 디스크
       1       8       33        1      active sync /dev/sdc1  	# RAID 슬롯 1에서 정상 동작 중인 구성 디스크
       2       8       49        2      active sync /dev/sdd1  	# RAID 슬롯 2에서 정상 동작 중인 구성 디스크
       5       8       65        3      active sync /dev/sde1	# RAID 슬롯 5에서 정상 동작 중인 구성 디스크
       4       8       81        -      spare         /dev/sdf1        	# RAID 슬롯에 아직 배정되지 않은 대기용 Spare 디스크
```

---

- 물리적인 5개의 HDD를 하나의 논리적인 HDD로 구성했지만 Format형식을 지정하지 않았기때문에 사용할 수 없다.

**EX8)** RAID 5로 구성한 HDD를 EXT4 형식으로 format해야한다.

```bash
[root@Server-A ~]# mkfs.ext4  /dev/md5
mke2fs 1.46.5 (30-Dec-2021)
Creating filesystem with 7856640 4k blocks and 1966080 inodes
Filesystem UUID: 1f9e42c0-cf3b-4f85-ad4f-55fd1bbe4c22
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208,
        4096000

Allocating group tables: done
Writing inode tables: done
Creating journal (32768 blocks): done
Writing superblocks and filesystem accounting information: done
```

```bash
[root@Server-A ~]# lsblk  -f
NAME    FSTYPE          FSVER LABEL        	UUID                                 		 FSAVAIL	 FSUSE%	 MOUNTPOINTS
sda
├─sda1  swap            1                          	520bc18c-2b64-4df1-85e0-d126908ba6dd                	 [SWAP]
└─sda2  xfs                                        	73bc277c-741d-4122-9c58-59ccd1889709	 10.4G	 35% 	 /
sdb
└─sdb1  linux_raid_memb 1.2   Server-A:5       	6795f6ac-fff1-db57-3c2d-92aa99de3b4e
  └─md5 ext4            1.0                        	1f9e42c0-cf3b-4f85-ad4f-55fd1bbe4c22
sdc
└─sdc1  linux_raid_memb 1.2   Server-A:5       	6795f6ac-fff1-db57-3c2d-92aa99de3b4e
  └─md5 ext4            1.0                        	1f9e42c0-cf3b-4f85-ad4f-55fd1bbe4c22
sdd
└─sdd1  linux_raid_memb 1.2   Server-A:5      	6795f6ac-fff1-db57-3c2d-92aa99de3b4e
  └─md5 ext4            1.0                        	1f9e42c0-cf3b-4f85-ad4f-55fd1bbe4c22
sde
└─sde1  linux_raid_memb 1.2   Server-A:5        	6795f6ac-fff1-db57-3c2d-92aa99de3b4e
  └─md5 ext4            1.0                        	1f9e42c0-cf3b-4f85-ad4f-55fd1bbe4c22
sdf
└─sdf1  linux_raid_memb 1.2   Server-A:5      	6795f6ac-fff1-db57-3c2d-92aa99de3b4e
  └─md5 ext4            1.0                        	1f9e42c0-cf3b-4f85-ad4f-55fd1bbe4c22
```

---

```bash
-RAID 5 구성후 EXT4 형식으로 File system을 구성했지만 mount되지 않았기 때문에 사용할 수 없다.

EX9) RAID 5로 구성한 HDD를 /RAID55로 mount 해야한다.

[root@Server-A ~]# mkdir  /RAID55
```

```bash
[root@Server-A ~]# mount  /dev/md5  /RAID55
```

```bash
[root@Server-A ~]# mount  | grep  md5
/dev/md5 on /RAID55 type ext4 (rw,relatime,stripe=384)
```

```bash
[root@Server-A ~]# df  -h
Filesystem      	Size   Used Avail Use%  Mounted on
devtmpfs        	807M       0  807M   0%  /dev
tmpfs           	838M       0  838M   0%  /dev/shm
tmpfs           	335M  5.5M  330M   2%   /run
/dev/sda2        	16G    5.6G   11G   35%  /
tmpfs           	168M    56K  168M  1%   /run/user/42
tmpfs           	168M    40K  168M  1%   /run/user/0
/dev/md5         	30G     24K   28G   1%   /RAID55
```

```bash
[root@Server-A ~]# lsblk  -f
NAME    FSTYPE          FSVER LABEL        	UUID                                 		 FSAVAIL	 FSUSE%	 MOUNTPOINTS
sda
├─sda1  swap            1                          	520bc18c-2b64-4df1-85e0-d126908ba6dd                	 [SWAP]
└─sda2  xfs                                        	73bc277c-741d-4122-9c58-59ccd1889709	 10.4G	 35% 	 /
sdb
└─sdb1  linux_raid_memb 1.2   Server-A:5       	6795f6ac-fff1-db57-3c2d-92aa99de3b4e
  └─md5 ext4            1.0                        	1f9e42c0-cf3b-4f85-ad4f-55fd1bbe4c22	 27.8G 	 0% 	 /RAID55
sdc
└─sdc1  linux_raid_memb 1.2   Server-A:5       	6795f6ac-fff1-db57-3c2d-92aa99de3b4e
  └─md5 ext4            1.0                        	1f9e42c0-cf3b-4f85-ad4f-55fd1bbe4c22	 27.8G  	 0% 	 /RAID55
sdd
└─sdd1  linux_raid_memb 1.2   Server-A:5      	6795f6ac-fff1-db57-3c2d-92aa99de3b4e
  └─md5 ext4            1.0                        	1f9e42c0-cf3b-4f85-ad4f-55fd1bbe4c22	 27.8G 	 0% 	 /RAID55
sde
└─sde1  linux_raid_memb 1.2   Server-A:5        	6795f6ac-fff1-db57-3c2d-92aa99de3b4e
  └─md5 ext4            1.0                        	1f9e42c0-cf3b-4f85-ad4f-55fd1bbe4c22	 27.8G 	 0% 	 /RAID55
sdf
└─sdf1  linux_raid_memb 1.2   Server-A:5      	6795f6ac-fff1-db57-3c2d-92aa99de3b4e
  └─md5 ext4            1.0                        	1f9e42c0-cf3b-4f85-ad4f-55fd1bbe4c22	 27.8G  	 0% 	 /RAID55
```

---

- RAID 5 구성후 EXT4 형식으로 File system을 구성했지만 mount되지 않았기 때문에 사용할 수 없다.

**EX10)** RAID 5로 구성한 HDD를 /RAID55로 mount 해야한다.

```bash
[root@Server-A ~]# UUID1=$(blkid  -s  UUID  -o  value  /dev/md5)
```

```bash
[root@Server-A ~]# echo $UUID1
1f9e42c0-cf3b-4f85-ad4f-55fd1bbe4c22
```

```bash
[root@Server-A ~]# cat  <<EOF  >>  /etc/fstab
> UUID=$UUID1  /RAID55  ext4  defaults  0 0
> EOF
```

```bash
[root@Server-A ~]# cat /etc/fstab

#
# /etc/fstab
# Created by anaconda on Tue Nov 18 03:10:48 2025
#
# Accessible filesystems, by reference, are maintained under '/dev/disk/'.
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info.
#
# After editing this file, run 'systemctl daemon-reload' to update systemd
# units generated from this file.
#
UUID=660fb32e-8a83-417c-83a6-2782fb9deb6f 	/	xfs     	defaults	0 0
UUID=69f5bcdb-8929-4ee2-9efb-90ca4c1ed25c 	none	swap    	defaults	0 0
UUID=1f9e42c0-cf3b-4f85-ad4f-55fd1bbe4c22	/RAID55	ext4	defaults	0 0
```

---

**EX11)** '/etc/' 디렉터리안의 파일중 c로 시작하는 모든 파일을 '/RAID55' 디렉터리로 복사해야한다.

```bash
[root@Server-A ~]# cp  -r  /etc/c*  /RAID55
```

```bash
[root@Server-A ~]# ls  -l  /RAID55
합계 80
drwxr-xr-x 3 root root  4096  7월 14 15:46 chromium
-rw-r--r-- 1 root root  1370  7월 14 15:46 chrony.conf
-rw-r----- 1 root root   540  7월 14 15:46 chrony.keys
drwxr-xr-x 2 root root  4096  7월 14 15:46 cifs-utils
drwxr-xr-x 4 root root  4096  7월 14 15:46 cockpit
drwxr-xr-x 2 root root  4096  7월 14 15:46 cron.d
drwxr-xr-x 2 root root  4096  7월 14 15:46 cron.daily
-rw-r--r-- 1 root root     0  7월 14 15:46 cron.deny
drwxr-xr-x 2 root root  4096  7월 14 15:46 cron.hourly
drwxr-xr-x 2 root root  4096  7월 14 15:46 cron.monthly
drwxr-xr-x 2 root root  4096  7월 14 15:46 cron.weekly
-rw-r--r-- 1 root root   451  7월 14 15:46 crontab
drwxr-xr-x 6 root root  4096  7월 14 15:46 crypto-policies
-rw------- 1 root root     0  7월 14 15:46 crypttab
-rw-r--r-- 1 root root  1401  7월 14 15:46 csh.cshrc
-rw-r--r-- 1 root root  1112  7월 14 15:46 csh.login
drwxr-xr-x 4 root root  4096  7월 14 15:46 cups
drwxr-xr-x 2 root root  4096  7월 14 15:46 cupshelpers
drwx------ 2 root root 16384  7월 14 15:39 lost+found
```

  - /dev/sdb1, /dev/sdc1, /dev/sdd1, /dev/sde1 HDD중 1개 장애 발생

```bash
[root@Server-A ~]# mdadm  --fail  /dev/md5  /dev/sdd1		# sdd1 HDD 장애 발생
```

```bash
[root@Server-A ~]# mdadm  --detail  /dev/md5
~~~~~~~~~ 중간 생략 ~~~~~~~~~
Consistency Policy : bitmap

    Rebuild Status : 3% complete

              Name : Server-A:5  (local to host Server-A)
              UUID : 6795f6ac:fff1db57:3c2d92aa:99de3b4e
            Events : 25

    Number   Major   Minor   RaidDevice State
       0       8       17        0      active sync   /dev/sdb1
       1       8       33        1      active sync   /dev/sdc1
       4       8       81        2      spare rebuilding   /dev/sdf1		# spare인 dev/sdf1이 RAID5로 동작
       5       8       65        3      active sync   /dev/sde1

       2       8       49        -      faulty   /dev/sdd1
```

```bash
[root@Server-A ~]# mdadm  --detail  /dev/md5
~~~~~~~~~ 중간 생략 ~~~~~~~~~
Consistency Policy : bitmap

              Name : Server-A:5  (local to host Server-A)
              UUID : 6795f6ac:fff1db57:3c2d92aa:99de3b4e
            Events : 46

    Number   Major   Minor   RaidDevice State
       0       8       17        0      active sync   /dev/sdb1
       1       8       33        1      active sync   /dev/sdc1
       4       8       81        2      active sync   /dev/sdf1	# RAID로 동작
       5       8       65        3      active sync   /dev/sde1

       2       8       49        -      faulty   /dev/sdd1
```

```bash
[root@Server-A ~]# ls  -l  /RAID55
합계 80
drwxr-xr-x 3 root root  4096  7월 14 15:46 chromium
-rw-r--r-- 1 root root  1370  7월 14 15:46 chrony.conf
-rw-r----- 1 root root   540  7월 14 15:46 chrony.keys
drwxr-xr-x 2 root root  4096  7월 14 15:46 cifs-utils
drwxr-xr-x 4 root root  4096  7월 14 15:46 cockpit
drwxr-xr-x 2 root root  4096  7월 14 15:46 cron.d
drwxr-xr-x 2 root root  4096  7월 14 15:46 cron.daily
-rw-r--r-- 1 root root     0  7월 14 15:46 cron.deny
drwxr-xr-x 2 root root  4096  7월 14 15:46 cron.hourly
drwxr-xr-x 2 root root  4096  7월 14 15:46 cron.monthly
drwxr-xr-x 2 root root  4096  7월 14 15:46 cron.weekly
-rw-r--r-- 1 root root   451  7월 14 15:46 crontab
drwxr-xr-x 6 root root  4096  7월 14 15:46 crypto-policies
-rw------- 1 root root     0  7월 14 15:46 crypttab
-rw-r--r-- 1 root root  1401  7월 14 15:46 csh.cshrc
-rw-r--r-- 1 root root  1112  7월 14 15:46 csh.login
drwxr-xr-x 4 root root  4096  7월 14 15:46 cups
drwxr-xr-x 2 root root  4096  7월 14 15:46 cupshelpers
drwx------ 2 root root 16384  7월 14 15:39 lost+found
```

  - /dev/sdb1, /dev/sdc1, /dev/sde1, /dev/sdf1 HDD중 1개 장애 발생

```bash
[root@Server-A ~]# mdadm  --fail  /dev/md5  /dev/sdb1		# 추가 장애 발생
```

```bash
[root@Server-A ~]# mdadm  --detail  /dev/md5
~~~~~~~~~ 중간 생략 ~~~~~~~~~
Consistency Policy : bitmap

              Name : Server-A:5  (local to host Server-A)
              UUID : 6795f6ac:fff1db57:3c2d92aa:99de3b4e
            Events : 48

    Number   Major   Minor   RaidDevice State
       -       0        0        0      removed
       1       8       33        1      active sync   /dev/sdc1
       4       8       81        2      active sync   /dev/sdf1
       5       8       65        3      active sync   /dev/sde1

       0       8       17        -      faulty   /dev/sdb1		# 추가 장애 발생
       2       8       49        -      faulty   /dev/sdd1
```

---

**정리**: RAID 1 실습 (8-4) 섹션에서 다룬 명령어와 실습 예제를 통해 관련 기능의 구성 방법과 동작 원리를 확인했다.

## RAID 5 실습 (8-6)

#### RAID 5

	
- RAID 중에서 실무에서 많이 사용되는 방식 중 하나이다.

- RAID 0의 공간 효율성과 읽기 성능, RAID 1처럼 디스크 장애 1개까지 버틸 수 있는 안정성을 절충한 구조이다.

- 최소 3개 이상의 HDD를 사용해야 한다.

- 데이터를 블록 단위로 잘라 여러 HDD에 나누어 저장하며(스트라이핑), 각 블록에 대한 패리티(Parity) 정보를 나머지 디스크에 분산 저장한다.

- 안정성을 위해 패리티(Parity) 정보를 사용한다.
  - 특정 HDD가 장애가 나면, 남은 데이터 블록 + 패리티 정보를 이용해 장애 디스크의 데이터를 재구성(복구)할 수 있다.

- Fault-tolerance(결함 허용)를 제공하면서도 공간 효율성이 좋은 편이다.

- 공간 효율성(사용 가능한 용량)
  - 1TB HDD 3개 사용 시 	: 2TB 사용 가능 (약66%)
  - 1TB HDD 4개 사용 시 	: 3TB 사용 가능 (75%)
  - 1TB HDD 5개 사용 시 	: 4TB 사용 가능 (80%)
  - 1TB HDD 10개 사용 시	: 9TB 사용 가능 (90%)
  - 디스크 개수가 많아질수록 공간 효율성은 100%에 가까워진다.

- 제약 및 주의사항
  - HDD가 한 개 고장 나는 것은 패리티로 복구 가능하지만,
   복구(리빌드)가 끝나기 전에 또 다른 HDD가 추가로 고장 나면 데이터를 복구할 수 없다.
  - 패리티를 계산하고 쓰는 과정 때문에, 일반적으로 읽기 성능은 좋지만
   쓰기 성능(특히 랜덤 쓰기)은 RAID 0, RAID 10보다 느릴 수 있다.
  - 소규모, 중규모의 스토리지에서 용량, 성능, 안정성을 적당히 모두 고려할 때 많이 사용된다.

	Data = 000 001 010 011

	   	   X
	HDD1	HDD2	HDD3	HDD4  =	HDD2
	   0	   0 	   0	   P0	   0
	   0	   0	   P1	   1	   0
	   0	   P1	   1	   0	   1
	   P0	   0	   1	   1	   0

---

#### 기존 디스크 초기화

```bash
	# 현재 /dev/md0 RAID 장치가 /linear 디렉터리에 마운트되어 있는지 확인

[root@Server-A ~]# mount | grep md0
/dev/md0 on /linear type ext4 (rw,relatime)
```

```bash
	# /linear에 마운트된 RAID 파일시스템을 마운트 해제

[root@Server-A ~]# umount /RAID10
```

```bash
	# /dev/md0에 생성된 파일시스템 서명 삭제

[root@Server-A ~]# wipefs -a -f /dev/md10
/dev/md10: 2 bytes were erased at offset 0x00000438 (ext4): 53 ef
```

- wipefs	: 디스크나 파티션에 기록된 파일시스템, RAID, LVM 등의 서명(Signature)을 확인하거나 삭제하는 명령어
- -a	: all의 약자로 대상 장치에서 발견되는 모든 서명을 삭제
- -f	: force의 약자로 장치가 파티션이 아닌 전체 디스크이거나 RAID 같은 특수 블록 장치여도 강제로 서명 삭제

```bash
	# 활성화되어 있는 /dev/md0 RAID 장치를 중지  (RAID 구성을 삭제하는 것이 아니라 현재 조립된 RAID의 동작만 중지)

[root@Server-A ~]# mdadm  --stop  /dev/md0
```

```bash
	# /dev/sdb1과 /dev/sdc1에 기록된 RAID 메타데이터(Superblock)를 삭제 (`RAID 레벨, RAID UUID, 디스크 순서, 구성 정보 등을 제거)

[root@Server-A ~]# mdadm --zero-superblock /dev/sdb1
[root@Server-A ~]# mdadm --zero-superblock /dev/sdc1
[root@Server-A ~]# mdadm --zero-superblock /dev/sdd1
[root@Server-A ~]# mdadm --zero-superblock /dev/sde1

        OR

[root@Server-A ~]# mdadm --zero-superblock --force /dev/sdb1
[root@Server-A ~]# mdadm --zero-superblock --force /dev/sdc1
[root@Server-A ~]# mdadm --zero-superblock --force /dev/sdd1
[root@Server-A ~]# mdadm --zero-superblock --force /dev/sde1
```

```bash
	# /dev/sdb와 /dev/sdc 디스크 전체에서 발견되는 모든 서명을 삭제 (디스크 전체에 실행하므로 sdb1, sdc1 파티션 정보까지 삭제)

[root@Server-A ~]# wipefs -a /dev/sdb
/dev/sdb: 2 bytes were erased at offset 0x000001fe (dos): 55 aa
/dev/sdb: calling ioctl to re-read partition table: 성공
```

```bash
[root@Server-A ~]# wipefs -a /dev/sdc
/dev/sdc: 2 bytes were erased at offset 0x000001fe (dos): 55 aa
/dev/sdc: calling ioctl to re-read partition table: 성공
```

```bash
[root@Server-A ~]# wipefs -a /dev/sdd
/dev/sdd: 2 bytes were erased at offset 0x000001fe (dos): 55 aa
/dev/sdd: calling ioctl to re-read partition table: 성공
```

```bash
[root@Server-A ~]# wipefs -a /dev/sde
/dev/sde: 2 bytes were erased at offset 0x000001fe (dos): 55 aa
/dev/sde: calling ioctl to re-read partition table: 성공
```

---

**EX1)** Server-A에 장착된 SDD에 Patition을 구성
  - 추가로 장착한 sdb를 하나의 Patition으로 구성해야한다.
  - 추가로 장착한 sdc를 하나의 Patition으로 구성해야한다.
  - 추가로 장착한 sdd를 하나의 Patition으로 구성해야한다.
  - 추가로 장착한 sde를 하나의 Patition으로 구성해야한다.
  - sdb , sdc , sdd , sde Patition 구성시 RAID용으로 구성해야한다.

```bash
[root@Server-A ~]# ls  -l  /dev/sd*
brw-rw---- 1 root disk 8,  0  7월 14 12:50 /dev/sda
brw-rw---- 1 root disk 8,  1  7월 14 12:50 /dev/sda1
brw-rw---- 1 root disk 8,  2  7월 14 12:50 /dev/sda2
brw-rw---- 1 root disk 8, 16  7월 14 14:38 /dev/sdb
brw-rw---- 1 root disk 8, 17  7월 14 14:38 /dev/sdb1
brw-rw---- 1 root disk 8, 32  7월 14 14:38 /dev/sdc
brw-rw---- 1 root disk 8, 33  7월 14 14:38 /dev/sdc1
brw-rw---- 1 root disk 8, 48  7월 14 14:38 /dev/sdd
brw-rw---- 1 root disk 8, 49  7월 14 14:38 /dev/sdd1
brw-rw---- 1 root disk 8, 64  7월 14 14:38 /dev/sde
brw-rw---- 1 root disk 8, 65  7월 14 14:38 /dev/sde1
```

```bash
[root@Server-A ~]# lsblk
NAME   MAJ:MIN RM  SIZE RO TYPE 	MOUNTPOINTS
sda      	  8:0    	0   20G  	0 disk
├─sda1	  8:1    	0    4G  	0 part 	[SWAP]
└─sda2	  8:2    	0   16G  	0 part 	/
sdb      	  8:16   	0   10G  	0 disk
└─sdb1	  8:17   	0   10G  	0 part
sdc      	  8:32   	0   10G  	0 disk
└─sdc	  8:33   	0   10G  	0 part
sdd      	  8:48   	0   10G  	0 disk
└─sdd1	  8:49   	0   10G  	0 part
sde      	  8:64   	0   10G  	0 disk
└─sde1	  8:65   	0   10G  	0 part
```

---

**EX2)** Patiton을 구성한 sdb1 , sdc1 , sdd1 , sde1을 RAID 5로 구성해야한다.

```bash
[root@Server-A ~]# rpm  -qa  | grep mdadm
mdadm-4.4-4.el9_7.x86_64
```

```bash
[root@Server-A ~]# mdadm  --create  /dev/md5  --level=5  --raid-devices=4  /dev/sdb1  /dev/sdc1  /dev/sdd1  /dev/sde1
To optimalize recovery speed, it is recommended to enable write-indent bitmap, do you want to enable it now? [y/N]? y
mdadm: Defaulting to version 1.2 metadata
mdadm: array /dev/md5 started.
```

```bash
[root@Server-A ~]# mdadm  --detail  --scan
ARRAY /dev/md5 metadata=1.2 spares=1 UUID=d2e3cf93:8543570c:67ee7c11:52a37b4e
```

```bash
[root@Server-A ~]# mdadm  --detail  /dev/md5
/dev/md5:
           Version 	: 1.2
     Creation Time 	: Tue Jul 14 14:44:17 2026
        Raid Level 	: raid5
        Array Size 	: 31426560 (29.97 GiB 32.18 GB)
     Used Dev Size 	: 10475520 (9.99 GiB 10.73 GB)
      Raid Devices 	: 4
     Total Devices 	: 4
       Persistence 	: Superblock is persistent

     Intent Bitmap 	: Internal

       Update Time 	: Tue Jul 14 14:45:09 2026
             State 	: clean
    Active Devices 	: 4
   Working Devices 	: 4
    Failed Devices 	: 0
     Spare Devices 	: 0

            Layout 	: left-symmetric
        Chunk Size 	: 512K

Consistency Policy	: bitmap

              Name 	: Server-A:5  (local to host Server-A)
              UUID 	: d2e3cf93:8543570c:67ee7c11:52a37b4e
            Events 	: 22

    Number   Major   Minor   RaidDevice State
       0       8       17        0      active sync   /dev/sdb1
       1       8       33        1      active sync   /dev/sdc1
       2       8       49        2      active sync   /dev/sdd1
       4       8       65        3      active sync   /dev/sde1
```

```bash
[root@Server-A ~]# lsblk  -f
NAME    FSTYPE          FSVER LABEL          	UUID                             		 FSAVAIL	 FSUSE%	 MOUNTPOINTS
sda
├─sda1  swap            1                          	520bc18c-2b64-4df1-85e0-d126908ba6dd                	 [SWAP]
└─sda2  xfs                                        	73bc277c-741d-4122-9c58-59ccd1889709	 10.4G	 35%	 /
sdb
└─sdb1  linux_raid_memb 1.2   Server-A:5         	d2e3cf93-8543-570c-67ee-7c1152a37b4e
  └─md5
sdc
└─sdc1  linux_raid_memb 1.2   Server-A:5       	d2e3cf93-8543-570c-67ee-7c1152a37b4e
  └─md5
sdd
└─sdd1  linux_raid_memb 1.2   Server-A:5        	d2e3cf93-8543-570c-67ee-7c1152a37b4e
  └─md5
sde
└─sde1  linux_raid_memb 1.2   Server-A:5      	d2e3cf93-8543-570c-67ee-7c1152a37b4e
  └─md5
```

---

- RAID 5 구성후 EXT4 형식으로 File system을 구성했지만 mount되지 않았기 때문에 사용할 수 없다.

**EX4)** RAID 5로 구성한 HDD를 /RAID 5로 mount 해야한다

```bash
[root@Server-A ~]# mkfs.ext4  /dev/md5
mke2fs 1.46.5 (30-Dec-2021)
Creating filesystem with 7856640 4k blocks and 1966080 inodes
Filesystem UUID: 70dcc674-a327-4e90-8064-f12dc3499bd9
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208,
        4096000

Allocating group tables: done
Writing inode tables: done
Creating journal (32768 blocks): done
Writing superblocks and filesystem accounting information: done
```

```bash
[root@Server-A ~]# mkdir  /RAID5
```

```bash
[root@Server-A ~]# mount  /dev/md5  /RAID5
```

```bash
[root@Server-A ~]# lsblk  -f
NAME    FSTYPE          FSVER LABEL                UUID                                 	 FSAVAIL	 FSUSE%	 MOUNTPOINTS
sda
├─sda1  swap            1                          	520bc18c-2b64-4df1-85e0-d126908ba6dd                	 [SWAP]
└─sda2  xfs                                        	73bc277c-741d-4122-9c58-59ccd1889709	 10.4G	 35%	 /
sdb
└─sdb1  linux_raid_memb 1.2   Server-A:5     	d2e3cf93-8543-570c-67ee-7c1152a37b4e
  └─md5 ext4            1.0                        	70dcc674-a327-4e90-8064-f12dc3499bd9
sdc
└─sdc1  linux_raid_memb 1.2   Server-A:5      	d2e3cf93-8543-570c-67ee-7c1152a37b4e
  └─md5 ext4            1.0                        	70dcc674-a327-4e90-8064-f12dc3499bd9
sdd
└─sdd1  linux_raid_memb 1.2   Server-A:5     	d2e3cf93-8543-570c-67ee-7c1152a37b4e
  └─md5 ext4            1.0                        	70dcc674-a327-4e90-8064-f12dc3499bd9
sde
└─sde1  linux_raid_memb 1.2   Server-A:5   	d2e3cf93-8543-570c-67ee-7c1152a37b4e
  └─md5 ext4            1.0                        	70dcc674-a327-4e90-8064-f12dc3499bd9
```

---

- RAID 5 구성후 EXT4 형식으로 File system을 구성했지만 mount되지 않았기 때문에 사용할 수 없다.

**EX4)** RAID 5로 구성한 HDD를 /RAID 5로 mount 해야한다.

```bash
[root@Server-A ~]# UUID1=$(blkid  -s  UUID  -o  value  /dev/md5)
```

```bash
[root@Server-A ~]# echo $UUID1
70dcc674-a327-4e90-8064-f12dc3499bd9
				## RAID 5
	
-RAID 중에서 실무에서 많이 사용되는 방식 중 하나이다.

-RAID 0의 공간 효율성과 읽기 성능, RAID 1처럼 디스크 장애 1개까지 버틸 수 있는 안정성을 절충한 구조이다.

-최소 3개 이상의 HDD를 사용해야 한다.

-데이터를 블록 단위로 잘라 여러 HDD에 나누어 저장하며(스트라이핑), 각 블록에 대한 패리티(Parity) 정보를 나머지 디스크에 분산 저장한다.

-안정성을 위해 패리티(Parity) 정보를 사용한다.
 # 특정 HDD가 장애가 나면, 남은 데이터 블록 + 패리티 정보를 이용해 장애 디스크의 데이터를 재구성(복구)할 수 있다.

-Fault-tolerance(결함 허용)를 제공하면서도 공간 효율성이 좋은 편이다.

-공간 효율성(사용 가능한 용량)
 # 1TB HDD 3개 사용 시 	: 2TB 사용 가능 (약66%)
 # 1TB HDD 4개 사용 시 	: 3TB 사용 가능 (75%)
 # 1TB HDD 5개 사용 시 	: 4TB 사용 가능 (80%)
 # 1TB HDD 10개 사용 시	: 9TB 사용 가능 (90%)
 # 디스크 개수가 많아질수록 공간 효율성은 100%에 가까워진다.

-제약 및 주의사항
 # HDD가 한 개 고장 나는 것은 패리티로 복구 가능하지만,
   복구(리빌드)가 끝나기 전에 또 다른 HDD가 추가로 고장 나면 데이터를 복구할 수 없다.
 # 패리티를 계산하고 쓰는 과정 때문에, 일반적으로 읽기 성능은 좋지만
   쓰기 성능(특히 랜덤 쓰기)은 RAID 0, RAID 10보다 느릴 수 있다.
 # 소규모, 중규모의 스토리지에서 용량, 성능, 안정성을 적당히 모두 고려할 때 많이 사용된다.
```

	Data = 000 001 010 011

	   	   X
	HDD1	HDD2	HDD3	HDD4  =	HDD2
	   0	   0 	   0	   P0	   0
	   0	   0	   P1	   1	   0
	   0	   P1	   1	   0	   1
	   P0	   0	   1	   1	   0

---

#### 기존 디스크 초기화

```bash
	# 현재 /dev/md0 RAID 장치가 /linear 디렉터리에 마운트되어 있는지 확인

[root@Server-A ~]# mount | grep md0
/dev/md0 on /linear type ext4 (rw,relatime)
```

```bash
	# /linear에 마운트된 RAID 파일시스템을 마운트 해제

[root@Server-A ~]# umount /RAID10
```

```bash
	# /dev/md0에 생성된 파일시스템 서명 삭제

[root@Server-A ~]# wipefs -a -f /dev/md10
/dev/md10: 2 bytes were erased at offset 0x00000438 (ext4): 53 ef
```

- wipefs	: 디스크나 파티션에 기록된 파일시스템, RAID, LVM 등의 서명(Signature)을 확인하거나 삭제하는 명령어
- -a	: all의 약자로 대상 장치에서 발견되는 모든 서명을 삭제
- -f	: force의 약자로 장치가 파티션이 아닌 전체 디스크이거나 RAID 같은 특수 블록 장치여도 강제로 서명 삭제

```bash
	# 활성화되어 있는 /dev/md0 RAID 장치를 중지  (RAID 구성을 삭제하는 것이 아니라 현재 조립된 RAID의 동작만 중지)

[root@Server-A ~]# mdadm  --stop  /dev/md10
```

```bash
	# /dev/sdb1과 /dev/sdc1에 기록된 RAID 메타데이터(Superblock)를 삭제 (`RAID 레벨, RAID UUID, 디스크 순서, 구성 정보 등을 제거)

[root@Server-A ~]# mdadm --zero-superblock /dev/sdb1
[root@Server-A ~]# mdadm --zero-superblock /dev/sdc1
[root@Server-A ~]# mdadm --zero-superblock /dev/sdd1
[root@Server-A ~]# mdadm --zero-superblock /dev/sde1

        OR

[root@Server-A ~]# mdadm --zero-superblock --force /dev/sdb1
[root@Server-A ~]# mdadm --zero-superblock --force /dev/sdc1
[root@Server-A ~]# mdadm --zero-superblock --force /dev/sdd1
[root@Server-A ~]# mdadm --zero-superblock --force /dev/sde1
```

```bash
	# /dev/sdb와 /dev/sdc 디스크 전체에서 발견되는 모든 서명을 삭제 (디스크 전체에 실행하므로 sdb1, sdc1 파티션 정보까지 삭제)

[root@Server-A ~]# wipefs -a /dev/sdb
/dev/sdb: 2 bytes were erased at offset 0x000001fe (dos): 55 aa
/dev/sdb: calling ioctl to re-read partition table: 성공
```

```bash
[root@Server-A ~]# wipefs -a /dev/sdc
/dev/sdc: 2 bytes were erased at offset 0x000001fe (dos): 55 aa
/dev/sdc: calling ioctl to re-read partition table: 성공
```

```bash
[root@Server-A ~]# wipefs -a /dev/sdd
/dev/sdd: 2 bytes were erased at offset 0x000001fe (dos): 55 aa
/dev/sdd: calling ioctl to re-read partition table: 성공
```

```bash
[root@Server-A ~]# wipefs -a /dev/sde
/dev/sde: 2 bytes were erased at offset 0x000001fe (dos): 55 aa
/dev/sde: calling ioctl to re-read partition table: 성공
```

---

**EX1)** Server-A에 장착된 SDD에 Patition을 구성
  - 추가로 장착한 sdb를 하나의 Patition으로 구성해야한다.
  - 추가로 장착한 sdc를 하나의 Patition으로 구성해야한다.
  - 추가로 장착한 sdd를 하나의 Patition으로 구성해야한다.
  - 추가로 장착한 sde를 하나의 Patition으로 구성해야한다.
  - sdb , sdc , sdd , sde Patition 구성시 RAID용으로 구성해야한다.

```bash
[root@Server-A ~]# ls  -l  /dev/sd*
brw-rw---- 1 root disk 8,  0  7월 14 12:50 /dev/sda
brw-rw---- 1 root disk 8,  1  7월 14 12:50 /dev/sda1
brw-rw---- 1 root disk 8,  2  7월 14 12:50 /dev/sda2
brw-rw---- 1 root disk 8, 16  7월 14 14:38 /dev/sdb
brw-rw---- 1 root disk 8, 17  7월 14 14:38 /dev/sdb1
brw-rw---- 1 root disk 8, 32  7월 14 14:38 /dev/sdc
brw-rw---- 1 root disk 8, 33  7월 14 14:38 /dev/sdc1
brw-rw---- 1 root disk 8, 48  7월 14 14:38 /dev/sdd
brw-rw---- 1 root disk 8, 49  7월 14 14:38 /dev/sdd1
brw-rw---- 1 root disk 8, 64  7월 14 14:38 /dev/sde
brw-rw---- 1 root disk 8, 65  7월 14 14:38 /dev/sde1
```

```bash
[root@Server-A ~]# lsblk
NAME   MAJ:MIN RM  SIZE RO TYPE 	MOUNTPOINTS
sda      	  8:0    	0   20G  	0 disk
├─sda1	  8:1    	0    4G  	0 part 	[SWAP]
└─sda2	  8:2    	0   16G  	0 part 	/
sdb      	  8:16   	0   10G  	0 disk
└─sdb1	  8:17   	0   10G  	0 part
sdc      	  8:32   	0   10G  	0 disk
└─sdc	  8:33   	0   10G  	0 part
sdd      	  8:48   	0   10G  	0 disk
└─sdd1	  8:49   	0   10G  	0 part
sde      	  8:64   	0   10G  	0 disk
└─sde1	  8:65   	0   10G  	0 part
```

---

**EX2)** Patiton을 구성한 sdb1 , sdc1 , sdd1 , sde1을 RAID 5로 구성해야한다.

```bash
[root@Server-A ~]# rpm  -qa  | grep mdadm
mdadm-4.4-4.el9_7.x86_64
```

```bash
[root@Server-A ~]# mdadm  --create  /dev/md5  --level=5  --raid-devices=4  /dev/sdb1  /dev/sdc1  /dev/sdd1  /dev/sde1
To optimalize recovery speed, it is recommended to enable write-indent bitmap, do you want to enable it now? [y/N]? y
mdadm: Defaulting to version 1.2 metadata
mdadm: array /dev/md5 started.
```

```bash
[root@Server-A ~]# mdadm  --detail  --scan
ARRAY /dev/md5 metadata=1.2 spares=1 UUID=d2e3cf93:8543570c:67ee7c11:52a37b4e
```

```bash
[root@Server-A ~]# mdadm  --detail  /dev/md5
/dev/md5:
           Version 	: 1.2
     Creation Time 	: Tue Jul 14 14:44:17 2026
        Raid Level 	: raid5
        Array Size 	: 31426560 (29.97 GiB 32.18 GB)
     Used Dev Size 	: 10475520 (9.99 GiB 10.73 GB)
      Raid Devices 	: 4
     Total Devices 	: 4
       Persistence 	: Superblock is persistent

     Intent Bitmap 	: Internal

       Update Time 	: Tue Jul 14 14:45:09 2026
             State 	: clean
    Active Devices 	: 4
   Working Devices 	: 4
    Failed Devices 	: 0
     Spare Devices 	: 0

            Layout 	: left-symmetric
        Chunk Size 	: 512K

Consistency Policy	: bitmap

              Name 	: Server-A:5  (local to host Server-A)
              UUID 	: d2e3cf93:8543570c:67ee7c11:52a37b4e
            Events 	: 22

    Number   Major   Minor   RaidDevice State
       0       8       17        0      active sync   /dev/sdb1
       1       8       33        1      active sync   /dev/sdc1
       2       8       49        2      active sync   /dev/sdd1
       4       8       65        3      active sync   /dev/sde1
```

```bash
[root@Server-A ~]# lsblk  -f
NAME    FSTYPE          FSVER LABEL          	UUID                             		 FSAVAIL	 FSUSE%	 MOUNTPOINTS
sda
├─sda1  swap            1                          	520bc18c-2b64-4df1-85e0-d126908ba6dd                	 [SWAP]
└─sda2  xfs                                        	73bc277c-741d-4122-9c58-59ccd1889709	 10.4G	 35%	 /
sdb
└─sdb1  linux_raid_memb 1.2   Server-A:5         	d2e3cf93-8543-570c-67ee-7c1152a37b4e
  └─md5
sdc
└─sdc1  linux_raid_memb 1.2   Server-A:5       	d2e3cf93-8543-570c-67ee-7c1152a37b4e
  └─md5
sdd
└─sdd1  linux_raid_memb 1.2   Server-A:5        	d2e3cf93-8543-570c-67ee-7c1152a37b4e
  └─md5
sde
└─sde1  linux_raid_memb 1.2   Server-A:5      	d2e3cf93-8543-570c-67ee-7c1152a37b4e
  └─md5
```

---

- RAID 5 구성후 EXT4 형식으로 File system을 구성했지만 mount되지 않았기 때문에 사용할 수 없다.

**EX4)** RAID 5로 구성한 HDD를 /RAID 5로 mount 해야한다

```bash
[root@Server-A ~]# mkfs.ext4  /dev/md5
mke2fs 1.46.5 (30-Dec-2021)
Creating filesystem with 7856640 4k blocks and 1966080 inodes
Filesystem UUID: 70dcc674-a327-4e90-8064-f12dc3499bd9
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208,
        4096000

Allocating group tables: done
Writing inode tables: done
Creating journal (32768 blocks): done
Writing superblocks and filesystem accounting information: done
```

```bash
[root@Server-A ~]# mkdir  /RAID5
```

```bash
[root@Server-A ~]# mount  /dev/md5  /RAID5
```

```bash
[root@Server-A ~]# lsblk  -f
NAME    FSTYPE          FSVER LABEL                UUID                                 	 FSAVAIL	 FSUSE%	 MOUNTPOINTS
sda
├─sda1  swap            1                          	520bc18c-2b64-4df1-85e0-d126908ba6dd                	 [SWAP]
└─sda2  xfs                                        	73bc277c-741d-4122-9c58-59ccd1889709	 10.4G	 35%	 /
sdb
└─sdb1  linux_raid_memb 1.2   Server-A:5     	d2e3cf93-8543-570c-67ee-7c1152a37b4e
  └─md5 ext4            1.0                        	70dcc674-a327-4e90-8064-f12dc3499bd9
sdc
└─sdc1  linux_raid_memb 1.2   Server-A:5      	d2e3cf93-8543-570c-67ee-7c1152a37b4e
  └─md5 ext4            1.0                        	70dcc674-a327-4e90-8064-f12dc3499bd9
sdd
└─sdd1  linux_raid_memb 1.2   Server-A:5     	d2e3cf93-8543-570c-67ee-7c1152a37b4e
  └─md5 ext4            1.0                        	70dcc674-a327-4e90-8064-f12dc3499bd9
sde
└─sde1  linux_raid_memb 1.2   Server-A:5   	d2e3cf93-8543-570c-67ee-7c1152a37b4e
  └─md5 ext4            1.0                        	70dcc674-a327-4e90-8064-f12dc3499bd9
```

---

- RAID 5 구성후 EXT4 형식으로 File system을 구성했지만 mount되지 않았기 때문에 사용할 수 없다.

**EX4)** RAID 5로 구성한 HDD를 /RAID 5로 mount 해야한다.

```bash
[root@Server-A ~]# UUID1=$(blkid  -s  UUID  -o  value  /dev/md5)
```

```bash
[root@Server-A ~]# echo $UUID1
70dcc674-a327-4e90-8064-f12dc3499bd9
```

```bash
[root@Server-A ~]# cat  <<EOF  >>  /etc/fstab
> UUID=$UUID1  /RAID5  ext4  defaults  0 0
> EOF
```

```bash
[root@Server-A ~]# cat /etc/fstab

#
# /etc/fstab
# Created by anaconda on Tue Nov 18 03:10:48 2025
#
# Accessible filesystems, by reference, are maintained under '/dev/disk/'.
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info.
#
# After editing this file, run 'systemctl daemon-reload' to update systemd
# units generated from this file.
#
UUID=660fb32e-8a83-417c-83a6-2782fb9deb6f 	/	xfs     	defaults	0 0
UUID=69f5bcdb-8929-4ee2-9efb-90ca4c1ed25c 	none	swap    	defaults	0 0
UUID=70dcc674-a327-4e90-8064-f12dc3499bd9	/RAID5	ext4	defaults	0 0	# 오토마운트 설정
```

---

**EX6)** '/etc/' 디렉터리안의 파일중 c로 시작하는 모든 파일을 '/RAID5' 디렉터리로 복사해야한다.

```bash
[root@Server-A ~]# cp  -r  /etc/c*  /RAID5
```

```bash
[root@Server-A ~]# ls  -l  /RAID5
합계 80
drwxr-xr-x 3 root root  4096  7월 14 14:53 chromium
-rw-r--r-- 1 root root  1370  7월 14 14:53 chrony.conf
-rw-r----- 1 root root   540  7월 14 14:53 chrony.keys
drwxr-xr-x 2 root root  4096  7월 14 14:53 cifs-utils
drwxr-xr-x 4 root root  4096  7월 14 14:53 cockpit
drwxr-xr-x 2 root root  4096  7월 14 14:53 cron.d
drwxr-xr-x 2 root root  4096  7월 14 14:53 cron.daily
-rw-r--r-- 1 root root       0  7월 14 14:53 cron.deny
drwxr-xr-x 2 root root  4096  7월 14 14:53 cron.hourly
drwxr-xr-x 2 root root  4096  7월 14 14:53 cron.monthly
drwxr-xr-x 2 root root  4096  7월 14 14:53 cron.weekly
-rw-r--r-- 1 root root    451  7월 14 14:53 crontab
drwxr-xr-x 6 root root  4096  7월 14 14:53 crypto-policies
-rw------- 1 root root      0  7월 14 14:53 crypttab
-rw-r--r-- 1 root root  1401  7월 14 14:53 csh.cshrc
-rw-r--r-- 1 root root  1112  7월 14 14:53 csh.login
drwxr-xr-x 4 root root  4096  7월 14 14:53 cups
drwxr-xr-x 2 root root  4096  7월 14 14:53 cupshelpers
drwx------ 2 root root 16384  7월 14 14:47 lost+found
```

---

- 현재 복사된 파일들은 '/RAID5' 디렉터리에 저장된다.
- '/dev/sdb1' , '/dev/sdc1'  , '/dev/sdd1'  , '/dev/sdd1' 에서 페리티 비트를 사용하기때문에 결함허용을 지원한다.
  즉 어떤 HDD에 장애가 발생해도 파일은 정상적으로 확인된다.

```bash
[root@Server-A ~]# mdadm  --fail  /dev/md5  /dev/sdc1		# 논리적인 장애로 인식

		OR

[root@Server-A ~]# mdadm  --remove  /dev/md5  /dev/sdc1	# 실제 RAID에서 제거
```

```bash
[root@Server-A ~]# mdadm  --detail  /dev/md5
/dev/md5:
           Version 	: 1.2
     Creation Time 	: Tue Jul 14 14:44:17 2026
        Raid Level 	: raid5
        Array Size 	: 31426560 (29.97 GiB 32.18 GB)
     Used Dev Size 	: 10475520 (9.99 GiB 10.73 GB)
      Raid Devices 	: 4
     Total Devices 	: 4
       Persistence 	: Superblock is persistent

     Intent Bitmap 	: Internal

       Update Time 	: Tue Jul 14 14:55:48 2026
             State 	: clean, degraded
    Active Devices 	: 3
   Working Devices 	: 3
    Failed Devices 	: 1
     Spare Devices 	: 0

            Layout 	: left-symmetric
        Chunk Size 	: 512K

Consistency Policy	: bitmap

              Name : Server-A:5  (local to host Server-A)
              UUID : d2e3cf93:8543570c:67ee7c11:52a37b4e
            Events : 24

    Number   Major   Minor   RaidDevice State
       0         8         17      	0      	active sync    /dev/sdb1
       -         0          0      	1      	removed
       2         8         49      	2      	active sync    /dev/sdd1
       4         8         65       	3      	active sync    /dev/sde1
       1         8         33        	-      	faulty	     /dev/sdc1
```

```bash
[root@Server-A ~]# ls  -l  /RAID5
합계 80
drwxr-xr-x 3 root root  4096  7월 14 14:53 chromium
-rw-r--r-- 1 root root  1370  7월 14 14:53 chrony.conf
-rw-r----- 1 root root   540  7월 14 14:53 chrony.keys
drwxr-xr-x 2 root root  4096  7월 14 14:53 cifs-utils
drwxr-xr-x 4 root root  4096  7월 14 14:53 cockpit
drwxr-xr-x 2 root root  4096  7월 14 14:53 cron.d
drwxr-xr-x 2 root root  4096  7월 14 14:53 cron.daily
-rw-r--r-- 1 root root      0  7월 14 14:53 cron.deny
drwxr-xr-x 2 root root  4096  7월 14 14:53 cron.hourly
drwxr-xr-x 2 root root  4096  7월 14 14:53 cron.monthly
drwxr-xr-x 2 root root  4096  7월 14 14:53 cron.weekly
-rw-r--r-- 1 root root   451  7월 14 14:53 crontab
drwxr-xr-x 6 root root  4096  7월 14 14:53 crypto-policies
-rw------- 1 root root      0  7월 14 14:53 crypttab
-rw-r--r-- 1 root root  1401  7월 14 14:53 csh.cshrc
-rw-r--r-- 1 root root  1112  7월 14 14:53 csh.login
drwxr-xr-x 4 root root  4096  7월 14 14:53 cups
drwxr-xr-x 2 root root  4096  7월 14 14:53 cupshelpers
drwx------ 2 root root 16384  7월 14 14:47 lost+found
```

```bash
[root@Server-A ~]# mdadm  --add  /dev/md5   /dev/sdc1	# superblock이 삭제되지 않으면 add 명령어로 다시 추가 가능
```

---

#### Disk 초기화

**EX7)** '/dev/sdb',  '/dev/sdc',  '/dev/sdd',  '/dev/sde',  '/dev/sdf'를 사용해서 RAID 5를 구성해야 한다. (10G HDD 1개 추가 : 총 5개)
       '/dev/sdb',  '/dev/sdc',  '/dev/sdd',  '/dev/sde'는 실제 동작하는 HDD로 사용하고
       '/dev/sde'는 '/dev/sdb',  '/dev/sdc',  '/dev/sdd' HDD에 장애 발생시 바로 RAID 5로 동작해야 한다.

#### 기존 디스크 초기화

```bash
	# 현재 /dev/md0 RAID 장치가 /linear 디렉터리에 마운트되어 있는지 확인

[root@Server-A ~]# mount | grep md0
/dev/md0 on /linear type ext4 (rw,relatime)
```

```bash
	# /linear에 마운트된 RAID 파일시스템을 마운트 해제

[root@Server-A ~]# umount /RAID5
```

```bash
	# /dev/md0에 생성된 파일시스템 서명 삭제

[root@Server-A ~]# wipefs -a -f /dev/md5
/dev/md10: 2 bytes were erased at offset 0x00000438 (ext4): 53 ef
```

- wipefs	: 디스크나 파티션에 기록된 파일시스템, RAID, LVM 등의 서명(Signature)을 확인하거나 삭제하는 명령어
- -a	: all의 약자로 대상 장치에서 발견되는 모든 서명을 삭제
- -f	: force의 약자로 장치가 파티션이 아닌 전체 디스크이거나 RAID 같은 특수 블록 장치여도 강제로 서명 삭제

```bash
	# 활성화되어 있는 /dev/md0 RAID 장치를 중지  (RAID 구성을 삭제하는 것이 아니라 현재 조립된 RAID의 동작만 중지)

[root@Server-A ~]# mdadm  --stop  /dev/md5
```

```bash
	# /dev/sdb1과 /dev/sdc1에 기록된 RAID 메타데이터(Superblock)를 삭제 (`RAID 레벨, RAID UUID, 디스크 순서, 구성 정보 등을 제거)

[root@Server-A ~]# mdadm --zero-superblock /dev/sdb1
[root@Server-A ~]# mdadm --zero-superblock /dev/sdc1
[root@Server-A ~]# mdadm --zero-superblock /dev/sdd1
[root@Server-A ~]# mdadm --zero-superblock /dev/sde1

        OR

[root@Server-A ~]# mdadm --zero-superblock --force /dev/sdb1
[root@Server-A ~]# mdadm --zero-superblock --force /dev/sdc1
[root@Server-A ~]# mdadm --zero-superblock --force /dev/sdd1
[root@Server-A ~]# mdadm --zero-superblock --force /dev/sde1
```

```bash
	# /dev/sdb와 /dev/sdc 디스크 전체에서 발견되는 모든 서명을 삭제 (디스크 전체에 실행하므로 sdb1, sdc1 파티션 정보까지 삭제)

[root@Server-A ~]# wipefs -a /dev/sdb
/dev/sdb: 2 bytes were erased at offset 0x000001fe (dos): 55 aa
/dev/sdb: calling ioctl to re-read partition table: 성공
```

```bash
[root@Server-A ~]# wipefs -a /dev/sdc
/dev/sdc: 2 bytes were erased at offset 0x000001fe (dos): 55 aa
/dev/sdc: calling ioctl to re-read partition table: 성공
```

```bash
[root@Server-A ~]# wipefs -a /dev/sdd
/dev/sdd: 2 bytes were erased at offset 0x000001fe (dos): 55 aa
/dev/sdd: calling ioctl to re-read partition table: 성공
```

```bash
[root@Server-A ~]# wipefs -a /dev/sde
/dev/sde: 2 bytes were erased at offset 0x000001fe (dos): 55 aa
/dev/sde: calling ioctl to re-read partition table: 성공
```

---

```bash
EX7) '/dev/sdb',  '/dev/sdc',  '/dev/sdd',  '/dev/sde',  '/dev/sdf'를 사용해서 RAID 5를 구성해야 한다. (10G HDD 1개 추가 : 총 5개)
       '/dev/sdb',  '/dev/sdc',  '/dev/sdd',  '/dev/sde'는 실제 동작하는 HDD로 사용하고  '/dev/sdf'는 
       '/dev/sdb',  '/dev/sdc',  '/dev/sdd',  '/dev/sde' HDD에 장애 발생시 바로 RAID 5로 동작해야 한다.

[root@Server-A ~]# lsblk
NAME   MAJ:MIN RM  SIZE RO TYPE 	MOUNTPOINTS
sda      	  8:0    	0   20G  	0 disk
├─sda1	  8:1    	0    4G  	0 part 	[SWAP]
└─sda2	  8:2    	0   16G  	0 part 	/
sdb      	  8:16   	0   10G  	0 disk
sdc      	  8:32   	0   10G  	0 disk
sdd      	  8:48   	0   10G  	0 disk
sde      	  8:64   	0   10G  	0 disk
sdf      	  8:80   	0   10G  	0 disk
```

```bash
[root@Server-A ~]# fdisk  /dev/sdb  (/dev/sdc  /dev/sdd/  dev/sde  dev/sdf)
Command (m for help): n
Select (default p):p
Partition number (1-4, default 1):1
First sector (2048-20971519, default 2048):
Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-20971519, default 20971519):
Command (m for help): t
Hex code or alias (type L to list all): fd
Command (m for help): w
```

```bash
[root@Server-A ~]# lsblk
NAME   MAJ:MIN RM  SIZE RO TYPE 	MOUNTPOINTS
sda      	  8:0    	0   20G  	0 disk
├─sda1	  8:1    	0    4G  	0 part 	[SWAP]
└─sda2	  8:2    	0   16G  	0 part 	/
sdb      	  8:16   	0   10G  	0 disk
└─sdb1	  8:17   	0   10G  	0 part
sdc      	  8:32   	0   10G  	0 disk
└─sdc1	  8:33   	0   10G  	0 part
sdd      	  8:48  	0   10G  	0 disk
└─sdd1	  8:49   	0   10G  	0 part
sde      	  8:64   	0   10G  	0 disk
└─sde1	  8:65   	0   10G  	0 part
sdf      	  8:80   	0   10G  	0 disk
└─sdf1	  8:81   	0   10G  	0 part
```

```bash
[root@Server-A ~]# mdadm  --create   /dev/md5  \
  --level=5  --raid-devices=4  \
  /dev/sdb1  /dev/sdc1  /dev/sdd1  /dev/sde1  \
  --spare-devices=1  /dev/sdf1

To optimalize recovery speed, it is recommended to enable write-indent bitmap, do you want to enable it now? [y/N]? y
mdadm: Defaulting to version 1.2 metadata
mdadm: array /dev/md5 started.
```

```bash
[root@Server-A ~]# mdadm --detail /dev/md5

/dev/md5:                                                      	# 확인 대상 RAID 장치 이름
           Version 	 1.2                                       	# RAID 메타데이터(Superblock) 버전
     Creation Time 	: Tue Jul 14 15:33:05 2026                  	# RAID가 생성된 날짜와 시간
        Raid Level 	: raid5                                     	# RAID 레벨: RAID 5
        Array Size 	: 31426560 (29.97 GiB 32.18 GB)       	# RAID 전체에서 실제 사용할 수 있는 논리적 용량
     Used Dev Size 	: 10475520 (9.99 GiB 10.73 GB)         	# 각 구성 디스크 1개에서 RAID가 사용하는 용량
      Raid Devices 	: 4                                         	# RAID 5를 구성하도록 지정된 활성 디스크 슬롯 수
     Total Devices 	: 5                                         	# 현재 RAID에 연결된 전체 디스크 수(Active + Spare)
       Persistence 	: Superblock is persistent                  	# RAID 구성 정보가 각 디스크의 Superblock에 영구 저장됨

     Intent Bitmap 	: Internal                                  	# 변경된 영역을 기록하는 Write-intent bitmap을 RAID 내부에 저장

       Update Time 	: Tue Jul 14 15:33:28 2026                  	# RAID 상태 정보가 마지막으로 갱신된 시간
             State 	: clean, degraded, recovering               	# 데이터는 일관성 있지만 디스크 1개가 부족하여 복구 중
    Active Devices 	: 4                                         	# 현재 정상적으로 RAID 데이터 저장에 참여 중인 디스크 수
   Working Devices 	: 5                                         	# 정상 인식되는 전체 디스크 수(Active + Rebuilding + Spare)
    Failed Devices 	: 0                                         	# Failed 상태로 등록된 디스크 수
     Spare Devices 	: 1                                         	# 현재 Spare 계열 상태인 디스크 수
                                                                		# 복구 중인 sde1과 대기 중인 sdf1이 포함됨

            Layout 	: left-symmetric                            	# RAID 5의 데이터와 패리티 배치 방식
        Chunk Size 	: 512K                                      	# 한 디스크에 연속으로 기록하는 데이터 조각 크기

Consistency Policy	: bitmap                                    	# Bitmap을 사용하여 변경 영역 중심으로 일관성 복구

              Name : Server-A:5 (local to host Server-A)       	# RAID 배열 이름
              UUID : 6795f6ac:fff1db57:3c2d92aa:99de3b4e 
            Events : 10

    Number   Major   Minor   RaidDevice State                   	# 각 디스크의 장치 번호, RAID 슬롯 및 상태

       0       8       17        0      active sync /dev/sdb1  	# RAID 슬롯 0에서 정상 동작 중인 구성 디스크
       1       8       33        1      active sync /dev/sdc1  	# RAID 슬롯 1에서 정상 동작 중인 구성 디스크
       2       8       49        2      active sync /dev/sdd1  	# RAID 슬롯 2에서 정상 동작 중인 구성 디스크
       5       8       65        3      active sync /dev/sde1	# RAID 슬롯 5에서 정상 동작 중인 구성 디스크
       4       8       81        -      spare         /dev/sdf1        	# RAID 슬롯에 아직 배정되지 않은 대기용 Spare 디스크
```

---

- 물리적인 5개의 HDD를 하나의 논리적인 HDD로 구성했지만 Format형식을 지정하지 않았기때문에 사용할 수 없다.

**EX8)** RAID 5로 구성한 HDD를 EXT4 형식으로 format해야한다.

```bash
[root@Server-A ~]# mkfs.ext4  /dev/md5
mke2fs 1.46.5 (30-Dec-2021)
Creating filesystem with 7856640 4k blocks and 1966080 inodes
Filesystem UUID: 1f9e42c0-cf3b-4f85-ad4f-55fd1bbe4c22
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208,
        4096000

Allocating group tables: done
Writing inode tables: done
Creating journal (32768 blocks): done
Writing superblocks and filesystem accounting information: done
```

```bash
[root@Server-A ~]# lsblk  -f
NAME    FSTYPE          FSVER LABEL        	UUID                                 		 FSAVAIL	 FSUSE%	 MOUNTPOINTS
sda
├─sda1  swap            1                          	520bc18c-2b64-4df1-85e0-d126908ba6dd                	 [SWAP]
└─sda2  xfs                                        	73bc277c-741d-4122-9c58-59ccd1889709	 10.4G	 35% 	 /
sdb
└─sdb1  linux_raid_memb 1.2   Server-A:5       	6795f6ac-fff1-db57-3c2d-92aa99de3b4e
  └─md5 ext4            1.0                        	1f9e42c0-cf3b-4f85-ad4f-55fd1bbe4c22
sdc
└─sdc1  linux_raid_memb 1.2   Server-A:5       	6795f6ac-fff1-db57-3c2d-92aa99de3b4e
  └─md5 ext4            1.0                        	1f9e42c0-cf3b-4f85-ad4f-55fd1bbe4c22
sdd
└─sdd1  linux_raid_memb 1.2   Server-A:5      	6795f6ac-fff1-db57-3c2d-92aa99de3b4e
  └─md5 ext4            1.0                        	1f9e42c0-cf3b-4f85-ad4f-55fd1bbe4c22
sde
└─sde1  linux_raid_memb 1.2   Server-A:5        	6795f6ac-fff1-db57-3c2d-92aa99de3b4e
  └─md5 ext4            1.0                        	1f9e42c0-cf3b-4f85-ad4f-55fd1bbe4c22
sdf
└─sdf1  linux_raid_memb 1.2   Server-A:5      	6795f6ac-fff1-db57-3c2d-92aa99de3b4e
  └─md5 ext4            1.0                        	1f9e42c0-cf3b-4f85-ad4f-55fd1bbe4c22
```

---

```bash
-RAID 5 구성후 EXT4 형식으로 File system을 구성했지만 mount되지 않았기 때문에 사용할 수 없다.

EX9) RAID 5로 구성한 HDD를 /RAID55로 mount 해야한다.

[root@Server-A ~]# mkdir  /RAID55
```

```bash
[root@Server-A ~]# mount  /dev/md5  /RAID55
```

```bash
[root@Server-A ~]# mount  | grep  md5
/dev/md5 on /RAID55 type ext4 (rw,relatime,stripe=384)
```

```bash
[root@Server-A ~]# df  -h
Filesystem      	Size   Used Avail Use%  Mounted on
devtmpfs        	807M       0  807M   0%  /dev
tmpfs           	838M       0  838M   0%  /dev/shm
tmpfs           	335M  5.5M  330M   2%   /run
/dev/sda2        	16G    5.6G   11G   35%  /
tmpfs           	168M    56K  168M  1%   /run/user/42
tmpfs           	168M    40K  168M  1%   /run/user/0
/dev/md5         	30G     24K   28G   1%   /RAID55
```

```bash
[root@Server-A ~]# lsblk  -f
NAME    FSTYPE          FSVER LABEL        	UUID                                 		 FSAVAIL	 FSUSE%	 MOUNTPOINTS
sda
├─sda1  swap            1                          	520bc18c-2b64-4df1-85e0-d126908ba6dd                	 [SWAP]
└─sda2  xfs                                        	73bc277c-741d-4122-9c58-59ccd1889709	 10.4G	 35% 	 /
sdb
└─sdb1  linux_raid_memb 1.2   Server-A:5       	6795f6ac-fff1-db57-3c2d-92aa99de3b4e
  └─md5 ext4            1.0                        	1f9e42c0-cf3b-4f85-ad4f-55fd1bbe4c22	 27.8G 	 0% 	 /RAID55
sdc
└─sdc1  linux_raid_memb 1.2   Server-A:5       	6795f6ac-fff1-db57-3c2d-92aa99de3b4e
  └─md5 ext4            1.0                        	1f9e42c0-cf3b-4f85-ad4f-55fd1bbe4c22	 27.8G  	 0% 	 /RAID55
sdd
└─sdd1  linux_raid_memb 1.2   Server-A:5      	6795f6ac-fff1-db57-3c2d-92aa99de3b4e
  └─md5 ext4            1.0                        	1f9e42c0-cf3b-4f85-ad4f-55fd1bbe4c22	 27.8G 	 0% 	 /RAID55
sde
└─sde1  linux_raid_memb 1.2   Server-A:5        	6795f6ac-fff1-db57-3c2d-92aa99de3b4e
  └─md5 ext4            1.0                        	1f9e42c0-cf3b-4f85-ad4f-55fd1bbe4c22	 27.8G 	 0% 	 /RAID55
sdf
└─sdf1  linux_raid_memb 1.2   Server-A:5      	6795f6ac-fff1-db57-3c2d-92aa99de3b4e
  └─md5 ext4            1.0                        	1f9e42c0-cf3b-4f85-ad4f-55fd1bbe4c22	 27.8G  	 0% 	 /RAID55
```

---

- RAID 5 구성후 EXT4 형식으로 File system을 구성했지만 mount되지 않았기 때문에 사용할 수 없다.

**EX10)** RAID 5로 구성한 HDD를 /RAID55로 mount 해야한다.

```bash
[root@Server-A ~]# UUID1=$(blkid  -s  UUID  -o  value  /dev/md5)
```

```bash
[root@Server-A ~]# echo $UUID1
1f9e42c0-cf3b-4f85-ad4f-55fd1bbe4c22
```

```bash
[root@Server-A ~]# cat  <<EOF  >>  /etc/fstab
> UUID=$UUID1  /RAID55  ext4  defaults  0 0
> EOF
```

```bash
[root@Server-A ~]# cat /etc/fstab

#
# /etc/fstab
# Created by anaconda on Tue Nov 18 03:10:48 2025
#
# Accessible filesystems, by reference, are maintained under '/dev/disk/'.
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info.
#
# After editing this file, run 'systemctl daemon-reload' to update systemd
# units generated from this file.
#
UUID=660fb32e-8a83-417c-83a6-2782fb9deb6f 	/	xfs     	defaults	0 0
UUID=69f5bcdb-8929-4ee2-9efb-90ca4c1ed25c 	none	swap    	defaults	0 0
UUID=1f9e42c0-cf3b-4f85-ad4f-55fd1bbe4c22	/RAID55	ext4	defaults	0 0
```

---

**EX11)** '/etc/' 디렉터리안의 파일중 c로 시작하는 모든 파일을 '/RAID55' 디렉터리로 복사해야한다.

```bash
[root@Server-A ~]# cp  -r  /etc/c*  /RAID55
```

```bash
[root@Server-A ~]# ls  -l  /RAID55
합계 80
drwxr-xr-x 3 root root  4096  7월 14 15:46 chromium
-rw-r--r-- 1 root root  1370  7월 14 15:46 chrony.conf
-rw-r----- 1 root root   540  7월 14 15:46 chrony.keys
drwxr-xr-x 2 root root  4096  7월 14 15:46 cifs-utils
drwxr-xr-x 4 root root  4096  7월 14 15:46 cockpit
drwxr-xr-x 2 root root  4096  7월 14 15:46 cron.d
drwxr-xr-x 2 root root  4096  7월 14 15:46 cron.daily
-rw-r--r-- 1 root root     0  7월 14 15:46 cron.deny
drwxr-xr-x 2 root root  4096  7월 14 15:46 cron.hourly
drwxr-xr-x 2 root root  4096  7월 14 15:46 cron.monthly
drwxr-xr-x 2 root root  4096  7월 14 15:46 cron.weekly
-rw-r--r-- 1 root root   451  7월 14 15:46 crontab
drwxr-xr-x 6 root root  4096  7월 14 15:46 crypto-policies
-rw------- 1 root root     0  7월 14 15:46 crypttab
-rw-r--r-- 1 root root  1401  7월 14 15:46 csh.cshrc
-rw-r--r-- 1 root root  1112  7월 14 15:46 csh.login
drwxr-xr-x 4 root root  4096  7월 14 15:46 cups
drwxr-xr-x 2 root root  4096  7월 14 15:46 cupshelpers
drwx------ 2 root root 16384  7월 14 15:39 lost+found
```

  - /dev/sdb1, /dev/sdc1, /dev/sdd1, /dev/sde1 HDD중 1개 장애 발생

```bash
[root@Server-A ~]# mdadm  --fail  /dev/md5  /dev/sdd1		# sdd1 HDD 장애 발생
```

```bash
[root@Server-A ~]# mdadm  --detail  /dev/md5
~~~~~~~~~ 중간 생략 ~~~~~~~~~
Consistency Policy : bitmap

    Rebuild Status : 3% complete

              Name : Server-A:5  (local to host Server-A)
              UUID : 6795f6ac:fff1db57:3c2d92aa:99de3b4e
            Events : 25

    Number   Major   Minor   RaidDevice State
       0       8       17        0      active sync   /dev/sdb1
       1       8       33        1      active sync   /dev/sdc1
       4       8       81        2      spare rebuilding   /dev/sdf1		# spare인 dev/sdf1이 RAID5로 동작
       5       8       65        3      active sync   /dev/sde1

       2       8       49        -      faulty   /dev/sdd1
```

```bash
[root@Server-A ~]# mdadm  --detail  /dev/md5
~~~~~~~~~ 중간 생략 ~~~~~~~~~
Consistency Policy : bitmap

              Name : Server-A:5  (local to host Server-A)
              UUID : 6795f6ac:fff1db57:3c2d92aa:99de3b4e
            Events : 46

    Number   Major   Minor   RaidDevice State
       0       8       17        0      active sync   /dev/sdb1
       1       8       33        1      active sync   /dev/sdc1
       4       8       81        2      active sync   /dev/sdf1	# RAID로 동작
       5       8       65        3      active sync   /dev/sde1

       2       8       49        -      faulty   /dev/sdd1
```

```bash
[root@Server-A ~]# ls  -l  /RAID55
합계 80
drwxr-xr-x 3 root root  4096  7월 14 15:46 chromium
-rw-r--r-- 1 root root  1370  7월 14 15:46 chrony.conf
-rw-r----- 1 root root   540  7월 14 15:46 chrony.keys
drwxr-xr-x 2 root root  4096  7월 14 15:46 cifs-utils
drwxr-xr-x 4 root root  4096  7월 14 15:46 cockpit
drwxr-xr-x 2 root root  4096  7월 14 15:46 cron.d
drwxr-xr-x 2 root root  4096  7월 14 15:46 cron.daily
-rw-r--r-- 1 root root     0  7월 14 15:46 cron.deny
drwxr-xr-x 2 root root  4096  7월 14 15:46 cron.hourly
drwxr-xr-x 2 root root  4096  7월 14 15:46 cron.monthly
drwxr-xr-x 2 root root  4096  7월 14 15:46 cron.weekly
-rw-r--r-- 1 root root   451  7월 14 15:46 crontab
drwxr-xr-x 6 root root  4096  7월 14 15:46 crypto-policies
-rw------- 1 root root     0  7월 14 15:46 crypttab
-rw-r--r-- 1 root root  1401  7월 14 15:46 csh.cshrc
-rw-r--r-- 1 root root  1112  7월 14 15:46 csh.login
drwxr-xr-x 4 root root  4096  7월 14 15:46 cups
drwxr-xr-x 2 root root  4096  7월 14 15:46 cupshelpers
drwx------ 2 root root 16384  7월 14 15:39 lost+found
```

  - /dev/sdb1, /dev/sdc1, /dev/sde1, /dev/sdf1 HDD중 1개 장애 발생

```bash
[root@Server-A ~]# mdadm  --fail  /dev/md5  /dev/sdb1		# 추가 장애 발생
```

```bash
[root@Server-A ~]# mdadm  --detail  /dev/md5
~~~~~~~~~ 중간 생략 ~~~~~~~~~
Consistency Policy : bitmap

              Name : Server-A:5  (local to host Server-A)
              UUID : 6795f6ac:fff1db57:3c2d92aa:99de3b4e
            Events : 48

    Number   Major   Minor   RaidDevice State
       -       0        0        0      removed
       1       8       33        1      active sync   /dev/sdc1
       4       8       81        2      active sync   /dev/sdf1
       5       8       65        3      active sync   /dev/sde1

       0       8       17        -      faulty   /dev/sdb1		# 추가 장애 발생
       2       8       49        -      faulty   /dev/sdd1
```

```bash
[root@Server-A ~]# ls  -l  /RAID55
합계 80
drwxr-xr-x 3 root root  4096  7월 14 15:46 chromium
-rw-r--r-- 1 root root  1370  7월 14 15:46 chrony.conf
-rw-r----- 1 root root   540  7월 14 15:46 chrony.keys
drwxr-xr-x 2 root root  4096  7월 14 15:46 cifs-utils
drwxr-xr-x 4 root root  4096  7월 14 15:46 cockpit
drwxr-xr-x 2 root root  4096  7월 14 15:46 cron.d
drwxr-xr-x 2 root root  4096  7월 14 15:46 cron.daily
-rw-r--r-- 1 root root     0  7월 14 15:46 cron.deny
drwxr-xr-x 2 root root  4096  7월 14 15:46 cron.hourly
drwxr-xr-x 2 root root  4096  7월 14 15:46 cron.monthly
drwxr-xr-x 2 root root  4096  7월 14 15:46 cron.weekly
-rw-r--r-- 1 root root   451  7월 14 15:46 crontab
drwxr-xr-x 6 root root  4096  7월 14 15:46 crypto-policies
-rw------- 1 root root     0  7월 14 15:46 crypttab
-rw-r--r-- 1 root root  1401  7월 14 15:46 csh.cshrc
-rw-r--r-- 1 root root  1112  7월 14 15:46 csh.login
drwxr-xr-x 4 root root  4096  7월 14 15:46 cups
drwxr-xr-x 2 root root  4096  7월 14 15:46 cupshelpers
drwx------ 2 root root 16384  7월 14 15:39 lost+found
```

---

#### RAID 6 이론

- RAID 6은 RAID 5처럼 블록 단위로 데이터를 분산 저장하면서, Parity bit를 2개 저장하는 RAID이다.
- 최소 4개 이상의 HDD가 필요하다.
- 동시에 HDD 2개까지 장애가 발생해도 데이터 복구가 가능하다.
- Parity를 2개 쓰기 때문에 RAID 5보다 쓰기 성능은 조금 떨어지지만 안정성은 더 높다.
- 공간 효율성 = (HDD 개수 - 2) / HDD 개수
  - 1T HDD 4개 사용시 사용 가능 용량 = 2T (50%)
  - 1T HDD 5개 사용시 사용 가능 용량 = 3T (60%)
  - 1T HDD 6개 사용시 사용 가능 용량 = 4T (66%)

예) 1T HDD 4개를 RAID 6으로 구성하면, 2개는 Parity 용으로 사용되므로 실제 사용 가능한 용량은 2T만 사용 가능하다.

- 디스크 용량이 커질수록 RAID6 안정성이 압도적으로 높다.

- 엔터프라이즈 스토리지 대부분 RAID6 또는 RAID10을 사용

- 최근에는
  - 대용량 파일서버
  - DB 백업 서버
  - VM 클러스터용 스토리지
  - 백업/아카이브 서버
  - 이런 환경에서는 거의 RAID6가 표준처럼 사용된다.

```bash
[root@Server-A ~]# cat  <<EOF  >>  /etc/fstab
> UUID=$UUID1  /RAID5  ext4  defaults  0 0
> EOF
```

```bash
[root@Server-A ~]# cat /etc/fstab

#
# /etc/fstab
# Created by anaconda on Tue Nov 18 03:10:48 2025
#
# Accessible filesystems, by reference, are maintained under '/dev/disk/'.
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info.
#
# After editing this file, run 'systemctl daemon-reload' to update systemd
# units generated from this file.
#
UUID=660fb32e-8a83-417c-83a6-2782fb9deb6f 	/	xfs     	defaults	0 0
UUID=69f5bcdb-8929-4ee2-9efb-90ca4c1ed25c 	none	swap    	defaults	0 0
UUID=70dcc674-a327-4e90-8064-f12dc3499bd9	/RAID5	ext4	defaults	0 0	# 오토마운트 설정
```

---

**EX6)** '/etc/' 디렉터리안의 파일중 c로 시작하는 모든 파일을 '/RAID5' 디렉터리로 복사해야한다.

```bash
[root@Server-A ~]# cp  -r  /etc/c*  /RAID5
```

```bash
[root@Server-A ~]# ls  -l  /RAID5
합계 80
drwxr-xr-x 3 root root  4096  7월 14 14:53 chromium
-rw-r--r-- 1 root root  1370  7월 14 14:53 chrony.conf
-rw-r----- 1 root root   540  7월 14 14:53 chrony.keys
drwxr-xr-x 2 root root  4096  7월 14 14:53 cifs-utils
drwxr-xr-x 4 root root  4096  7월 14 14:53 cockpit
drwxr-xr-x 2 root root  4096  7월 14 14:53 cron.d
drwxr-xr-x 2 root root  4096  7월 14 14:53 cron.daily
-rw-r--r-- 1 root root       0  7월 14 14:53 cron.deny
drwxr-xr-x 2 root root  4096  7월 14 14:53 cron.hourly
drwxr-xr-x 2 root root  4096  7월 14 14:53 cron.monthly
drwxr-xr-x 2 root root  4096  7월 14 14:53 cron.weekly
-rw-r--r-- 1 root root    451  7월 14 14:53 crontab
drwxr-xr-x 6 root root  4096  7월 14 14:53 crypto-policies
-rw------- 1 root root      0  7월 14 14:53 crypttab
-rw-r--r-- 1 root root  1401  7월 14 14:53 csh.cshrc
-rw-r--r-- 1 root root  1112  7월 14 14:53 csh.login
drwxr-xr-x 4 root root  4096  7월 14 14:53 cups
drwxr-xr-x 2 root root  4096  7월 14 14:53 cupshelpers
drwx------ 2 root root 16384  7월 14 14:47 lost+found
```

---

- 현재 복사된 파일들은 '/RAID5' 디렉터리에 저장된다.
- '/dev/sdb1' , '/dev/sdc1'  , '/dev/sdd1'  , '/dev/sdd1' 에서 페리티 비트를 사용하기때문에 결함허용을 지원한다.
  즉 어떤 HDD에 장애가 발생해도 파일은 정상적으로 확인된다.

```bash
[root@Server-A ~]# mdadm  --fail  /dev/md5  /dev/sdc1		# 논리적인 장애로 인식

		OR

[root@Server-A ~]# mdadm  --remove  /dev/md5  /dev/sdc1	# 실제 RAID에서 제거
```

```bash
[root@Server-A ~]# mdadm  --detail  /dev/md5
/dev/md5:
           Version 	: 1.2
     Creation Time 	: Tue Jul 14 14:44:17 2026
        Raid Level 	: raid5
        Array Size 	: 31426560 (29.97 GiB 32.18 GB)
     Used Dev Size 	: 10475520 (9.99 GiB 10.73 GB)
      Raid Devices 	: 4
     Total Devices 	: 4
       Persistence 	: Superblock is persistent

     Intent Bitmap 	: Internal

       Update Time 	: Tue Jul 14 14:55:48 2026
             State 	: clean, degraded
    Active Devices 	: 3
   Working Devices 	: 3
    Failed Devices 	: 1
     Spare Devices 	: 0

            Layout 	: left-symmetric
        Chunk Size 	: 512K

Consistency Policy	: bitmap

              Name : Server-A:5  (local to host Server-A)
              UUID : d2e3cf93:8543570c:67ee7c11:52a37b4e
            Events : 24

    Number   Major   Minor   RaidDevice State
       0         8         17      	0      	active sync    /dev/sdb1
       -         0          0      	1      	removed
       2         8         49      	2      	active sync    /dev/sdd1
       4         8         65       	3      	active sync    /dev/sde1
       1         8         33        	-      	faulty	     /dev/sdc1
```

```bash
[root@Server-A ~]# ls  -l  /RAID5
합계 80
drwxr-xr-x 3 root root  4096  7월 14 14:53 chromium
-rw-r--r-- 1 root root  1370  7월 14 14:53 chrony.conf
-rw-r----- 1 root root   540  7월 14 14:53 chrony.keys
drwxr-xr-x 2 root root  4096  7월 14 14:53 cifs-utils
drwxr-xr-x 4 root root  4096  7월 14 14:53 cockpit
drwxr-xr-x 2 root root  4096  7월 14 14:53 cron.d
drwxr-xr-x 2 root root  4096  7월 14 14:53 cron.daily
-rw-r--r-- 1 root root      0  7월 14 14:53 cron.deny
drwxr-xr-x 2 root root  4096  7월 14 14:53 cron.hourly
drwxr-xr-x 2 root root  4096  7월 14 14:53 cron.monthly
drwxr-xr-x 2 root root  4096  7월 14 14:53 cron.weekly
-rw-r--r-- 1 root root   451  7월 14 14:53 crontab
drwxr-xr-x 6 root root  4096  7월 14 14:53 crypto-policies
-rw------- 1 root root      0  7월 14 14:53 crypttab
-rw-r--r-- 1 root root  1401  7월 14 14:53 csh.cshrc
-rw-r--r-- 1 root root  1112  7월 14 14:53 csh.login
drwxr-xr-x 4 root root  4096  7월 14 14:53 cups
drwxr-xr-x 2 root root  4096  7월 14 14:53 cupshelpers
drwx------ 2 root root 16384  7월 14 14:47 lost+found
```

```bash
[root@Server-A ~]# mdadm  --add  /dev/md5   /dev/sdc1	# superblock이 삭제되지 않으면 add 명령어로 다시 추가 가능
```

---

#### Disk 초기화

**EX7)** '/dev/sdb',  '/dev/sdc',  '/dev/sdd',  '/dev/sde'를 사용해서 RAID 5를 구성해야 한다.
  - '/dev/sdb',  '/dev/sdc',  '/dev/sdd'는 실제 동작하는 HDD로 사용하고 '/dev/sde'는 '/dev/sdb',  '/dev/sdc',  '/dev/sdd' HDD에
     장애 발생시 바로 RAID 5로 동작해야 한다.

---

**정리**: RAID 5 실습 (8-6) 섹션에서 다룬 명령어와 실습 예제를 통해 관련 기능의 구성 방법과 동작 원리를 확인했다.

## LVM (8-8)

### LVM (Logical Volume Manager)

- Linux 시스템에 새 HDD를 추가하거나 설치 후 파티션을 나누면, 일반적으로 각 파티션은 고정된 크기로 설정된다.
  그러나 물리적인 디스크의 크기는 고정되어 있기 때문에, 사용 중에 크기를 변경하거나 공간을 유연하게 관리하기가 어렵다.
 이런 문제를 해결하기 위해 리눅스는 LVM(Logical Volume Manager) 기능을 제공한다.

- LVM의 개념
  - LVM은 여러 개의 물리적 디스크(혹은 파티션)를 하나의 큰 저장 공간처럼 묶어 관리할 수 있는 기술이다.
  - 반대로, 하나의 디스크를 논리적으로 여러 개로 나누어 사용할 수도 있다.
  - LVM을 사용하면, 데이터 이동 없이도 볼륨 크기를 유연하게 늘리거나 줄일 수 있다.

- 물리 볼륨 (PV : Physical Volume)
  - 실제 디스크나 디스크의 파티션을 LVM에서 사용할 수 있도록 초기화한 단위
  - 예: /dev/sdb1, /dev/sdc1
  - pvcreate 명령으로 생성한다.
  - LVM에서는 PV를 기본 블록으로 묶어 사용한다.

- 볼륨 그룹 (VG : Volume Group)
  - 여러 개의 물리 볼륨(PV)을 묶어서 하나의 논리적 저장공간으로 만든 단위
  - 하나의 VG는 여러 PV로 구성될 수 있다.
  - 'vgcreate' 명령으로 생성한다.
  - 예: vgcreate vg_data /dev/sdb1 /dev/sdc1

- 논리 볼륨 (LV : Logical Volume)
  - VG 내부에서 관리자가 원하는 크기만큼 나누어 만든 가상 파티션
  - 실제 파일 시스템을 만들어 마운트하는 단위
  - lvcreate 명령으로 생성한다.
  - 예: lvcreate -L 10G -n lv_backup vg_data

- PE (Physical Extent)
  - VM에서 공간 할당의 가장 작은 단위.
  - 일반적으로 기본 크기는 4MB이다.
  - 여러 개의 PE가 모여 하나의 LV를 구성한다.
  - 즉, LV는 여러 개의 PE 묶음으로 만들어진 논리 공간이다.

- LVM의 장점 
  - 디스크를 재파티셔닝하지 않아도 크기를 늘리거나 줄일 수 있다.
    (할당한 메모리를 모두 사용시 남은 메모리에서 용량을 추가할 수 있다.)
  - RAID보다 구조가 단순하고 관리가 쉽다.
  - 다양한 디바이스 조합(PV)을 유연하게 묶을 수 있다.

- LVM의 단점 
  - RAID처럼 데이터 미러링(복제)이나 결함 허용(Fault Tolerance) 기능이 기본적으로 제공되지 않는다.
  - 단일 LV 손상 시 전체 VG에 영향을 줄 수 있다.

- LVM 구성 절차
1) 새 HDD를 LVM 파티션으로 생성 (fdisk 또는 parted 사용, 타입 = 8e)
2) 생성한 파티션을 LVM용 물리 볼륨(PV)으로 초기화 	EX) pvcreate /dev/sdb1
3) 물리 볼륨(PV)들을 묶어 볼륨 그룹(VG) 생성 		EX) vgcreate vg_data /dev/sdb1 /dev/sdc1
4) 볼륨 그룹(VG)에서 원하는 크기로 논리 볼륨(LV) 생성 	EX) lvcreate -L 10G -n lv_backup vg_data
5) 논리 볼륨(LV)을 파일시스템으로 포맷 			EX) mkfs.ext4 /dev/vg_data/lv_backup
6) 디렉터리 생성 후 마운트				EX) mkdir /backup --> mount /dev/vg_data/lv_backup /backup
7) 부팅 시 자동 마운트				EX) /etc/fstab에 등록한다.

#### RAID vs LVM

- --

비교 항목		| 	RAID			| 		LVM				|

- --

목적		| 안정성, 성능, 장애 허용		| 유연한 용량 관리, 디스크 확장			|

- --

데이터 보호	| O (RAID1 / 5 / 6 / 10 등)	 	| X  (LVM 단독은 보호 기능 없음)			|

- --

성능 향상		| RAID0 / 10 가능			| LVM 자체는 성능 향상 기능 거의 없음			|

- --

장애 허용		| RAID5 / 6 / 10 가능		| 없음 (LVM+RAID 조합은 가능)			|

- --

디스크 묶는 방식	| 블록 단위 동기화 / 스트라이핑 / 패리티	| 논리 볼륨 단위로 디스크를 합쳐 하나의 큰 볼륨 제공	|

- --

디스크 추가/확장	| 어려움(종류에 따라 제한)		| 매우 쉬움 (LV 확장/축소 가능)			|

- --

사용 목적		| 서버 안정성 & 성능 중심		| 용량 확장 & 유연성 중심				|

- --

---

#### 기존 디스크 초기화

```bash
	# 현재 /dev/md0 RAID 장치가 /linear 디렉터리에 마운트되어 있는지 확인

[root@Server-A ~]# mount | grep md5
/dev/md0 on /RAID55 type ext4 (rw,relatime)
```

```bash
	# /linear에 마운트된 RAID 파일시스템을 마운트 해제

[root@Server-A ~]# umount /RAID55
```

```bash
	# /dev/md0에 생성된 파일시스템 서명 삭제

[root@Server-A ~]# wipefs -a -f /dev/md5
/dev/md10: 2 bytes were erased at offset 0x00000438 (ext4): 53 ef
```

- wipefs	: 디스크나 파티션에 기록된 파일시스템, RAID, LVM 등의 서명(Signature)을 확인하거나 삭제하는 명령어
- -a	: all의 약자로 대상 장치에서 발견되는 모든 서명을 삭제
- -f	: force의 약자로 장치가 파티션이 아닌 전체 디스크이거나 RAID 같은 특수 블록 장치여도 강제로 서명 삭제

```bash
	# 활성화되어 있는 /dev/md0 RAID 장치를 중지  (RAID 구성을 삭제하는 것이 아니라 현재 조립된 RAID의 동작만 중지)

[root@Server-A ~]# mdadm  --stop  /dev/md5
```

```bash
	# /dev/sdb1과 /dev/sdc1에 기록된 RAID 메타데이터(Superblock)를 삭제 (`RAID 레벨, RAID UUID, 디스크 순서, 구성 정보 등을 제거)

[root@Server-A ~]# mdadm --zero-superblock /dev/sdb1
[root@Server-A ~]# mdadm --zero-superblock /dev/sdc1
[root@Server-A ~]# mdadm --zero-superblock /dev/sdd1
[root@Server-A ~]# mdadm --zero-superblock /dev/sde1
[root@Server-A ~]# mdadm --zero-superblock /dev/sdf1

        OR

[root@Server-A ~]# mdadm --zero-superblock --force /dev/sdb1
[root@Server-A ~]# mdadm --zero-superblock --force /dev/sdc1
[root@Server-A ~]# mdadm --zero-superblock --force /dev/sdd1
[root@Server-A ~]# mdadm --zero-superblock --force /dev/sde1
[root@Server-A ~]# mdadm --zero-superblock --force /dev/sdf1
```

```bash
	# /dev/sdb ~ /dev/sdf 디스크 전체에서 발견되는 모든 서명을 삭제 (디스크 전체에 실행하므로 sdb1, sdc1 파티션 정보까지 삭제)

[root@Server-A ~]# wipefs -a /dev/sdb
/dev/sdb: 2 bytes were erased at offset 0x000001fe (dos): 55 aa
/dev/sdb: calling ioctl to re-read partition table: 성공
```

```bash
[root@Server-A ~]# wipefs -a /dev/sdc
/dev/sdc: 2 bytes were erased at offset 0x000001fe (dos): 55 aa
/dev/sdc: calling ioctl to re-read partition table: 성공
```

```bash
[root@Server-A ~]# wipefs -a /dev/sdd
/dev/sdd: 2 bytes were erased at offset 0x000001fe (dos): 55 aa
/dev/sdd: calling ioctl to re-read partition table: 성공
```

```bash
[root@Server-A ~]# wipefs -a /dev/sde
/dev/sde: 2 bytes were erased at offset 0x000001fe (dos): 55 aa
/dev/sde: calling ioctl to re-read partition table: 성공
```

```bash
[root@Server-A ~]# wipefs -a /dev/sdf
/dev/sdf: 2 bytes were erased at offset 0x000001fe (dos): 55 aa
/dev/sdf: calling ioctl to re-read partition table: 성공
```

```bash
[root@Server-A ~]# vi /etc/fstab		# 오토 마운트 삭제
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
UUID=73bc277c-741d-4122-9c58-59ccd1889709       /       	xfs     	defaults        0 0
UUID=520bc18c-2b64-4df1-85e0-d126908ba6dd       none   	swap	defaults        0 0

:wq
```

```bash
[root@Server-A ~]# lsblk  -f
NAME   FSTYPE  FSVER LABEL	UUID                                 		FSAVAIL  FSUSE% 	MOUNTPOINTS
sda
├─sda1 swap    1                     	520bc18c-2b64-4df1-85e0-d126908ba6dd                	[SWAP]
└─sda2 xfs                            	73bc277c-741d-4122-9c58-59ccd1889709	10.4G	 35% 	/
sdb
sdc
sdd
sde
sdf
```

---

```bash
     ## 1) HDD를 LVM으로 구성하기위해서 VLM 파티션으로 구성

-/dev/sdb , /dev/sdc 사용

[root@Server-A ~]# fdisk  /dev/sdb
Command (m for help): d
Command (m for help): n
Select (default p):p
Command (m for help): t
Hex code or alias (type L to list all): 8e
Command (m for help): p
/dev/sdb1        2048 20971519 20969472  10G 8e Linux LVM
Command (m for help): w
```

```bash
[root@Server-A ~]# fdisk  /dev/sdc
Command (m for help): d
Command (m for help): n
Select (default p):p
Command (m for help): t
Hex code or alias (type L to list all): 8e
Command (m for help): p
/dev/sdc1        2048 20971519 20969472  10G 8e Linux LVM
Command (m for help): w
```

#### 2) Patition한 HDD를 LVM 물리 볼륨 (PV)으로 전환

```bash
[root@Server-A ~]# pvcreate  /dev/sdb1
  Physical volume "/dev/sdb1" successfully created.
  Creating devices file /etc/lvm/devices/system.devices
```

```bash
[root@Server-A ~]# pvcreate  /dev/sdc1
  Physical volume "/dev/sdc1" successfully created.
```

```bash
[root@Server-A ~]# pvdisplay
  "/dev/sdb1" is a new physical volume of "<10.00 GiB"
  --- NEW Physical volume ---
  PV Name        	/dev/sdb1
  VG Name
  PV Size          	<10.00 GiB
  Allocatable    	NO
  PE Size        	0
  Total PE        	0
  Free PE          	0
  Allocated PE 	0
  PV UUID        	ojkyPF-vcl6-XwGh-0r67-8bwy-SSHO-zkK4Db

  "/dev/sdc1" is a new physical volume of "<10.00 GiB"
  --- NEW Physical volume ---
  PV Name         	/dev/sdc1
  VG Name
  PV Size          	<10.00 GiB
  Allocatable     	NO
  PE Size       	0
  Total PE       	0
  Free PE         	0
  Allocated PE   	0
  PV UUID        	edCsNQ-zS63-BrpH-G0Vv-bNyi-XqjX-Xnbhh2
```

```bash
[root@Server-A ~]# pvs
  PV         VG Fmt  Attr  PSize   PFree
 /dev/sdb1      lvm2 ---  <10.00g <10.00g
 /dev/sdc1      lvm2 ---  <10.00g <10.00g
```

- PV	: Physical Volume, LVM에서 사용할 물리 볼륨
- VG	: 해당 PV가 소속된 Volume Group (VG에는 아직 미등록)
- PSize	: PV 전체 용량
- PFree	: 아직 사용되지 않은 PV 여유 공간

#### 3) 물리 볼륨 (PV)들을 하나의 볼륨그룹(VG)으로 구성한다.

```bash
[root@Server-A ~]# vgcreate  SOLLVM  /dev/sdb1  /dev/sdc1
  Volume group "SOLLVM" successfully created
```

```bash
[root@Server-A ~]# pvdisplay
  --- Physical volume ---
  PV Name         	/dev/sdb1				# PV로 초기화된 HDD 또는 Patition
  VG Name         	SOLLVM				# PV들이 포함된 그룹 (VG : SOLLVM)
  PV Size         	<10.00 GiB / not usable 3.00 MiB	# 해당 PV의 크기 (10G) , 이중 메타데이터등으로 3MB를 사용
  Allocatable       	yes				# 해당 PV에 LV 생성 가능
  PE Size         	4.00 MiB				# PE 1개의 크기
  Total PE       	2559				# 해당 PV에 생성 가능한 PE 개수
  Free PE        	2559				# LV에 할당 가능한 PE 개수
  Allocated PE 	0				# LV에 할당된 PE 개수
  PV UUID         	ojkyPF-vcl6-XwGh-0r67-8bwy-SSHO-zkK4Db

  --- Physical volume ---
  PV Name       	/dev/sdc1
  VG Name       	SOLLVM
  PV Size        	<10.00 GiB / not usable 3.00 MiB
  Allocatable      	 yes
  PE Size        	4.00 MiB
  Total PE       	2559
  Free PE        	2559
  Allocated PE  	0
  PV UUID      	edCsNQ-zS63-BrpH-G0Vv-bNyi-XqjX-Xnbhh2
```

```bash
[root@Server-A ~]# vgs
  VG       #PV #LV #SN Attr     VSize   VFree
  SOLLVM    2     0     0 wz--n- 19.99g  19.99g
```

VG  	: 볼륨 그룹 이름
- PV  	: VG에 포함된 PV 개수
- LV   	: 생성된 LV 개수
- SN   	: 스냅샷 개수
Attr 	: VG 속성
VSize 	: VG 전체 크기
VFree  	 : 아직 사용하지 않은 여유 공간

#### 4) 볼륨그룹(VG)을 원하는 용량의 크기로 Patition하여 논리 볼륨(LV)으로 생성

```bash
[root@Server-A ~]# lvcreate   --size  8G  --name 8G_LV1  SOLLVM
  Logical volume "8G_LV1" created.

# lvcreate		: LV(논리 볼륨) 생성
# --size		: LV의 크기
# --name		: LV의 이름
# SOLLVM		: 파티션할 볼륨그룹 명
```

```bash
[root@Server-A ~]# vgs
  VG       #PV #LV #SN Attr     VSize   VFree
  SOLLVM    2     1     0 wz--n- 19.99g  11.99g
```

```bash
[root@Server-A ~]# lvcreate   --size  8G  --name 6G_LV2  SOLLVM
  Logical volume "6G_LV2" created.
```

```bash
[root@Server-A ~]# vgs
  VG       #PV #LV #SN Attr      VSize   VFree
  SOLLVM    2     2     0  wz--n- 19.99g  3.99g
```

```bash
[root@Server-A ~]# lvcreate  --extents  100%FREE  --name 6G_LV3  SOLLVM
  Logical volume "6G_LV3" created.

# --extents : Volume Group안의 남은 모든 용량을 하나의 LV에 할당
```

```bash
[root@Server-A ~]# vgs
  VG       #PV #LV #SN Attr     VSize   VFree
  SOLLVM    2     3      0 wz--n- 19.99g  0
```

```bash
[root@Server-A ~]# lsblk -f
NAME              FSTYPE      FSVER    LABEL	UUID                                   		FSAVAIL FSUSE% MOUNTPOINTS
sda
├─sda1            swap        1                       	520bc18c-2b64-4df1-85e0-d126908ba6dd                  		[SWAP]
└─sda2            xfs                                       	73bc277c-741d-4122-9c58-59ccd1889709     	10.4G    	35% 	/
sdb
└─sdb1            LVM2_member LVM2 001   	ojkyPF-vcl6-XwGh-0r67-8bwy-SSHO-zkK4Db
  ├─SOLLVM-8G_LV1
  └─SOLLVM-6G_LV3
sdc
└─sdc1            LVM2_member LVM2 001      	edCsNQ-zS63-BrpH-G0Vv-bNyi-XqjX-Xnbhh2
  ├─SOLLVM-6G_LV2
  └─SOLLVM-6G_LV3
sdd
sde
sdf
```

#### 5) Patition한 논리 볼륨(LV)을 파일 시스템으로 포맷

```bash
[root@Server-A ~]# ls  -l  /dev/SOL*
합계 0
lrwxrwxrwx 1 root root 7  7월 14 18:09 6G_LV3 -> ../dm-2
lrwxrwxrwx 1 root root 7  7월 14 18:05 8G_LV1 -> ../dm-0
lrwxrwxrwx 1 root root 7  7월 14 18:06 6G_LV2 -> ../dm-1
```

```bash
[root@Server-A ~]# mkfs.ext4  /dev/SOLLVM/8G_LV1
mke2fs 1.46.5 (30-Dec-2021)
Creating filesystem with 2097152 4k blocks and 524288 inodes
Filesystem UUID: ef64e3c4-4f23-46ac-a119-7615389d2258
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632

Allocating group tables: done
Writing inode tables: done
Creating journal (16384 blocks): done
Writing superblocks and filesystem accounting information: done
```

```bash
[root@Server-A ~]# mkfs.ext4  /dev/SOLLVM/6G_LV2
mke2fs 1.46.5 (30-Dec-2021)
Creating filesystem with 2097152 4k blocks and 524288 inodes
Filesystem UUID: 9d1adacb-48b6-4148-ad00-fac93dc9cd52
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632

Allocating group tables: done
Writing inode tables: done
Creating journal (16384 blocks): done
Writing superblocks and filesystem accounting information: done
```

```bash
[root@Server-A ~]# mkfs.ext4  /dev/SOLLVM/6G_LV3
mke2fs 1.46.5 (30-Dec-2021)
Creating filesystem with 1046528 4k blocks and 261632 inodes
Filesystem UUID: 6831db0a-08d7-450b-8d2b-746aedeff293
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376, 294912, 819200, 884736

Allocating group tables: done
Writing inode tables: done
Creating journal (16384 blocks): done
Writing superblocks and filesystem accounting information: done
```

```bash
[root@Server-A ~]# lsblk -f
NAME              FSTYPE      FSVER    LABEL	UUID                                   FSAVAIL FSUSE% MOUNTPOINTS
sda
├─sda1            swap        1                        	520bc18c-2b64-4df1-85e0-d126908ba6dd                  [SWAP]
└─sda2            xfs                                   	73bc277c-741d-4122-9c58-59ccd1889709     10.4G    35% /
sdb
└─sdb1            LVM2_member LVM2 001        	ojkyPF-vcl6-XwGh-0r67-8bwy-SSHO-zkK4Db
  ├─SOLLVM-8G_LV1  ext4        1.0                  	ef64e3c4-4f23-46ac-a119-7615389d2258
  └─SOLLVM-6G_LV3  ext4        1.0              	6831db0a-08d7-450b-8d2b-746aedeff293
sdc
└─sdc1            LVM2_member LVM2 001        	edCsNQ-zS63-BrpH-G0Vv-bNyi-XqjX-Xnbhh2
  ├─SOLLVM-6G_LV2 ext4        1.0             	9d1adacb-48b6-4148-ad00-fac93dc9cd52
  └─SOLLVM-6G_LV3 ext4        1.0               	6831db0a-08d7-450b-8d2b-746aedeff293
sdd
sde
sdf
```

#### 6) 디렉터리 생성 후 mount하여 HDD를 사용

```bash
[root@Server-A ~]# mkdir  /CU
[root@Server-A ~]# mkdir  /GS
[root@Server-A ~]# mkdir  /LG
```

```bash
[root@Server-A ~]# mount  /dev/SOLLVM/8G_LV1  /CU
mount: (hint) your fstab has been modified, but systemd still uses
       the old version; use 'systemctl daemon-reload' to reload.
```

```bash
[root@Server-A ~]# mount  /dev/SOLLVM/6G_LV2  /GS
mount: (hint) your fstab has been modified, but systemd still uses
       the old version; use 'systemctl daemon-reload' to reload.
```

```bash
[root@Server-A ~]# mount  /dev/SOLLVM/6G_LV3  /LG
mount: (hint) your fstab has been modified, but systemd still uses
       the old version; use 'systemctl daemon-reload' to reload.
```

```bash
[root@Server-A ~]# mount | grep LV
/dev/mapper/SOLLVM-8G_LV1 on /CU type ext4 (rw,relatime)
/dev/mapper/SOLLVM-6G_LV2 on /GS type ext4 (rw,relatime)
/dev/mapper/SOLLVM-6G_LV3 on /LG type ext4 (rw,relatime)
```

```bash
[root@Server-A ~]# df -h
Filesystem                 	  Size    Used Avail Use% Mounted on
devtmpfs                   	  807M       0  807M   0%  /dev
tmpfs                      	  838M       0  838M   0%  /dev/shm
tmpfs                      	  335M  6.9M  329M   3%  /run
/dev/sda2                   	  16G    5.6G   11G    35% /
tmpfs                      	  168M    56K  168M   1%  /run/user/42
tmpfs                      	  168M    40K  168M   1%  /run/user/0
/dev/mapper/SOLLVM-8G_LV1	  7.8G     24K  7.4G   1%  /CU
/dev/mapper/SOLLVM-6G_LV2	  7.8G     24K  7.4G   1%  /GS
/dev/mapper/SOLLVM-6G_LV3 	  3.9G     24K  3.7G   1%  /LG
```

```bash
[root@Server-A ~]# UUID_CU=$(blkid -s UUID -o value /dev/mapper/SOLLVM-8G_LV1)
[root@Server-A ~]# UUID_GS=$(blkid -s UUID -o value /dev/mapper/SOLLVM-6G_LV2)
[root@Server-A ~]# UUID_LG=$(blkid -s UUID -o value /dev/mapper/SOLLVM-6G_LV3)
```

```bash
[root@Server-A ~]# echo $UUID_CU
ef64e3c4-4f23-46ac-a119-7615389d2258

[root@Server-A ~]# echo $UUID_GS
9d1adacb-48b6-4148-ad00-fac93dc9cd52

[root@Server-A ~]# echo $UUID_LG
6831db0a-08d7-450b-8d2b-746aedeff293
```

```bash
[root@Server-A ~]# cat <<EOF >> /etc/fstab
> UUID=$UUID_CU     /CU    ext4    defaults    0 0
> UUID=$UUID_GS     /GS     ext4    defaults    0 0
> UUID=$UUID_LG     /LG     ext4    defaults    0 0
> EOF
```

```bash
[root@Server-A ~]# cat  /etc/fstab

UUID=73bc277c-741d-4122-9c58-59ccd1889709	/       	xfs     	defaults    0 0
UUID=520bc18c-2b64-4df1-85e0-d126908ba6dd  	none    	swap    	defaults    0 0
UUID=ef64e3c4-4f23-46ac-a119-7615389d2258  	/CU 	ext4    	defaults    0 0
UUID=9d1adacb-48b6-4148-ad00-fac93dc9cd52	/GS   	ext4    	defaults    0 0
UUID=6831db0a-08d7-450b-8d2b-746aedeff293  	/LG	ext4	defaults    0 0
```

---

```bash
     ## 7) SOLLVM VG에 HDD 추가

[root@Server-A ~]# fdisk  /dev/sdd
Command (m for help): n
Command (m for help): t
Hex code or alias (type L to list all): 8e
Command (m for help): w
```

```bash
[root@Server-A ~]# fdisk  -l | grep LVM
/dev/sdb1        2048 20971519 20969472  10G 8e Linux LVM
/dev/sdc1        2048 20971519 20969472  10G 8e Linux LVM
/dev/sdd1        2048 20971519 20969472  10G 8e Linux LVM
Disk /dev/mapper/SOLLVM-8G_LV1: 8 GiB, 8589934592 bytes, 16777216 sectors
Disk /dev/mapper/SOLLVM-6G_LV2: 6 GiB, 6442450944 bytes, 12582912 sectors
Disk /dev/mapper/SOLLVM-6G_LV3: 5.99 GiB, 6434062336 bytes, 12566528 sectors
```

```bash
[root@Server-A ~]# fdisk  -l | grep 8e
/dev/sdb1        2048 20971519 20969472  10G 8e Linux LVM
/dev/sdc1        2048 20971519 20969472  10G 8e Linux LVM
/dev/sdd1        2048 20971519 20969472  10G 8e Linux LVM
```

```bash
[root@Server-A ~]# pvcreate  /dev/sdd1
  Physical volume "/dev/sdd1" successfully created.
```

```bash
[root@Server-A ~]# vgextend  SOLLVM  /dev/sdd1
  Volume group "SOLLVM" successfully extended
```

```bash
[root@Server-A ~]# vgs
  VG         #PV #LV #SN Attr   VSize   VFree
  SOLLVM     3     3   0 wz--n- <29.99g <10.00g
```

```bash
[root@Server-A ~]# pvdisplay
  --- Physical volume ---
  PV Name         	/dev/sdb1
  VG Name        	SOLLVM
  PV Size        	<10.00 GiB / not usable 3.00 MiB
  Allocatable        	yes (but full)
  PE Size          	 4.00 MiB
  Total PE         	2559
  Free PE         	0
  Allocated PE     	2559
  PV UUID         	ro5WPh-5XlN-hEoS-ZJWT-sjB1-G4Rl-X3q53x

  --- Physical volume ---
  PV Name         	/dev/sdc1
  VG Name          	SOLLVM
  PV Size           	<10.00 GiB / not usable 3.00 MiB
  Allocatable       	yes (but full)
  PE Size          	4.00 MiB
  Total PE          	2559
  Free PE        	0
  Allocated PE      	2559
  PV UUID         	POU1i5-M6Jv-3Rjg-FyMG-F0NY-20g9-I0NikR

  --- Physical volume ---
  PV Name          	/dev/sdd1
  VG Name          	SOLLVM
  PV Size           	<10.00 GiB / not usable 3.00 MiB
  Allocatable       	yes
  PE Size           	4.00 MiB
  Total PE         	2559
  Free PE            	2559
  Allocated PE       	0
  PV UUID            	DkzcQE-3Ice-d2EB-0Ohm-FPje-ZQWy-8Dpa8k
```

---

#### 8) LV(논리 볼륨) 용량 확장

- 확장할 LV : /dev/SOLLVM/8G_LV1
- 현재 용량  : 8G --> 9G로 확장

```bash
[root@Server-A ~]# lvextend  --size  +1G  /dev/SOLLVM/8G_LV1
  Size of logical volume SOLLVM/8G_LV1 changed from 8.00 GiB (2048 extents) to 9.00 GiB (2304 extents).
  Logical volume SOLLVM/8G_LV1 successfully resized.
```

- lvextend		: LVM 내부 논리 블록의 크기를 확장
- --size			: 확장 또는 축소할 크기
- /dev/SOLLVM/8G_LV1	: 확장 또는 축소할 LV(논리 볼륨)

```bash
-LV을 확장시 파일시스템도 같이 확장해야한다. (파일 시스템 확장은 마운트를 해제하지 않고 증가시킬 수 있다.)

[root@Server-A ~]# resize2fs  /dev/SOLLVM/8G_LV1
resize2fs 1.46.5 (30-Dec-2021)
Filesystem at /dev/SOLLVM/8G_LV1 is mounted on /CU; on-line resizing required
old_desc_blocks = 1, new_desc_blocks = 2
The filesystem on /dev/SOLLVM/8G_LV1 is now 2359296 (4k) blocks long.
```

```bash
[root@Server-A ~]# vgs				# 확장 전
  VG         #PV #LV #SN Attr   VSize   VFree
  SOLLVM     3     3   0 wz--n- <29.99g <10.00g		# VFree = 10.00g
```

```bash
[root@Server-A ~]# vgs				# 확장 후
  VG        #PV #LV #SN Attr   VSize    VFree
  SOLLVM    3     3   0 wz--n- <29.99g <9.00g		# VFree = 9.00g
```

```bash
[root@Server-A ~]# df  -h
Filesystem                		 Size  Used Avail Use% Mounted on
devtmpfs                   		807M     0  807M   0% /dev
tmpfs                      		838M     0  838M   0% /dev/shm
tmpfs                      		335M  6.9M  329M   3% /run
/dev/sda2                   		16G  5.6G   11G  35% /
tmpfs                      		168M   56K  168M   1% /run/user/42
tmpfs                      		168M   40K  168M   1% /run/user/0
/dev/mapper/SOLLVM-8G_LV1  	8.8G   24K  8.4G   1% /CU		# 8.8G (9G로 확장)
/dev/mapper/SOLLVM-6G_LV2  	5.9G   24K  5.6G   1% /GS
/dev/mapper/SOLLVM-6G_LV3	5.9G   24K  5.5G   1% /LG
```

#### 9) LV(논리 볼륨) 용량 축소

- 축소할 LV  : /dev/SOLLVM/6G_LV2
- 현재 용량   : 6G --> 5G 로 축소

```bash
[root@Server-A ~]# umount  /GS	# 마운트 해제
```

#### 파일 시스템을 먼지 축소(리사이즈)

```bash
	# LV 축소전 파일 시스템 축소 (resize2fs)
[root@Server-A ~]# resize2fs  /dev/SOLLVM/6G_LV2  5G
resize2fs 1.46.5 (30-Dec-2021)
Please run 'e2fsck -f /dev/SOLLVM/6G_LV2' first.		# 5G의 용량으로 축소 전 e2fsck를 사용하여 파일시스템을 검사
```

```bash
	# 파일 시스템 검사 (데이터 구조가 정상인지 확인)
[root@Server-A ~]# e2fsck -f  /dev/SOLLVM/6G_LV2
e2fsck 1.46.5 (30-Dec-2021)
Pass 1: Checking inodes, blocks, and sizes
Pass 2: Checking directory structure
Pass 3: Checking directory connectivity
Pass 4: Checking reference counts
Pass 5: Checking group summary information
/dev/SOLLVM/6G_LV2: 11/393216 files (0.0% non-contiguous), 47214/1572864 blocks
```

```bash
	# LV 축소전 파일 시스템 축소 (resize2fs)
[root@Server-A ~]# resize2fs  /dev/SOLLVM/6G_LV2  5G
resize2fs 1.46.5 (30-Dec-2021)
Resizing the filesystem on /dev/SOLLVM/6G_LV2 to 1310720 (4k) blocks.
The filesystem on /dev/SOLLVM/6G_LV2 is now 1310720 (4k) blocks long.
```

```bash
	# LV 축소
[root@Server-A ~]# lvreduce  --size 5G  /dev/SOLLVM/6G_LV2
  File system ext4 found on SOLLVM/6G_LV2.
  File system size (5.00 GiB) is equal to the requested size (5.00 GiB).
  File system reduce is not needed, skipping.
  Size of logical volume SOLLVM/6G_LV2 changed from 6.00 GiB (1536 extents) to 5.00 GiB (1280 extents).
  Logical volume SOLLVM/6G_LV2 successfully resized.
```

```bash
[root@Server-A ~]# mount  /dev/SOLLVM/6G_LV2  /GS
```

```bash
[root@Server-A ~]# df  -h
Filesystem                 		Size  Used Avail Use% Mounted on
devtmpfs                   		807M     0  807M   0% /dev
tmpfs                      		838M     0  838M   0% /dev/shm
tmpfs                      		335M  6.9M  329M   3% /run
/dev/sda2                   		16G  5.6G   11G  35% /
tmpfs                      		168M   56K  168M   1% /run/user/42
tmpfs                      		168M   40K  168M   1% /run/user/0
/dev/mapper/SOLLVM-8G_LV1  	8.8G   24K  8.4G   1% /CU
/dev/mapper/SOLLVM-6G_LV3  	5.9G   24K  5.5G   1% /LG
/dev/mapper/SOLLVM-6G_LV2  	4.9G   24K  4.6G   1% /GS		# 6G에서 5G로 LV가 축소
```

```bash
[root@Server-A ~]# lvs
  LV     	VG     	Attr           LSize Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  6G_LV2	SOLLVM	-wi-ao---- 5.00g		# 6G  -->  5G 축소
  6G_LV3 	SOLLVM	-wi-ao---- 5.99g
  8G_LV1 	SOLLVM	-wi-ao---- 9.00g		# 8G  -->  9G 확장
```

```bash
[root@Server-A ~]# pvdisplay

[root@Server-A ~]# vgdisplay

[root@Server-A ~]# lvdisplay
```

- LVM을 삭제하기위해서는  LV  --->  VG  --->  PV순으로 삭제해야한다.

**정리**: LVM (8-8) 섹션에서 다룬 명령어와 실습 예제를 통해 관련 기능의 구성 방법과 동작 원리를 확인했다.

# Linux 07 — 파티션 · 마운트 · Automount

## 목차

1. [Disk Type](#disk-type)
2. [Partition](#partition)
3. [File System](#file-system)
4. [Partition 구성 (fdisk 실습)](#partition-구성-fdisk-실습)
5. [File System 구성 (mkfs)](#file-system-구성-mkfs)
6. [Mount](#mount)
7. [Automount](#automount-7-2)

## Disk Type

새 디스크를 추가해 사용하거나 서버 재부팅 후에도 스토리지를 자동으로 연결해 두려면 디스크 종류를 이해하고 파티션과 마운트 과정을 거쳐야 한다.

### IDE (Integrated Drive Electronics)

- 가장 오래된 규격, 40개의 핀으로 구성된 직사각형 포트로 구성
- 컴퓨터와 디스크 구동 장치 간의 표준 인터페이스로 데이터를 병렬로 전송한다.
- **CPU에서 직접 하드 관리**하기때문에 CPU 소모율이 증가하며 부팅 중 장착 할 수 없다.
- 버전별로 데이터 전송속도가 다르며 최신 규격에서는 초당 133MB의 데이터를 전송
- 초기형 IDE보다 성능이 향상된 **E-IDE**(Enhanced IDE) 규격이지만 통합으로 IDE라고 부른다.
- 장치명: `/dev/hda`, `/dev/hdb`, `/dev/hdc`, ··· (CentOS 7버전이상에서는 `/dev/dsa`, `/dev/sdb`로 출력된다.)

### SATA (Serial Advanced Technologly Attachment)

- IDE를 대체한 직렬 전송 방식의 하드디스크 규격
- 속도: SATA1 = 150MB/s, SATA2 = 300MB/s, SATA3 = 600MB/s
- SATA1에서는 초당 150MB, SATA2에서는 초당 320MB, SATA3에서는 초당 600MB의 전송 속도를 갖는다.
- 전원을 끄지 않고 장치를 연결/제거할 수 있는 **Hot-plug** 기능을 지원한다.
- 장치명: `/dev/sda`, `/dev/sdb`, `/dev/sdc`, ···

### SSD(Solid State Drive)

- 플래시 메모리를 사용하는 반도체 기반 저장 장치로, 기계적 회전 부품이 없어 속도가 매우 빠르다.
- 랜덤 액세스 속도가 HDD에 비해 뛰어나고, 지연 시간(latency)이 낮아 시스템 전체 체감 속도가 크게 향상된다.
- 충격에 강하고 소음이 거의 없으며, 소비 전력도 HDD보다 적다.
- 지연, 소음이 적다.
- 장치명: `/dev/sda`, `/dev/sdb`, `/dev/sdc`, ···

### SCSI (Small Computer System Interface)

- 서버나 워크스테이션 등에 사용되는 고속 디스크/장치 인터페이스 규격이다.
- 안정성이 높고 속도가 빠르지만, 일반 PC용 SATA에 비해 가격이 높은 편이다.
- 전용 SCSI 컨트롤러(확장 카드)를 사용해야 하는 경우가 많아, 구성 비용과 난이도가 상대적으로 높다.
- 내부 SCSI 컨트롤러 칩이 디스크를 직접 관리하므로, IDE 방식보다 CPU 부하가 적은 편이다.
- 전원을 끄지 않고 장치를 연결/제거할 수 있는 Hot-plug 기능을 지원한다.
- 장치명: 최신 리눅스에서는 SATA/SSD와 마찬가지로 `/dev/sda`, `/dev/sdb`, `/dev/sdc`, … 형태로 통합되어 표시된다.

### SAS(Serial Attached SCSI)

- SCSI 규격을 한단계 발전 시킨 직렬 방식 disk로 서버 등 대형 컴퓨터에 주로 사용 (서버의 SSD)
- SATA 하드 디스크를 SAS 장치에 연결 가능하지만 SAS 하드디스크를 SATA interface에 연결할 수 없다.
- 장치명: `/dev/sda`, `/dev/sdb`, `/dev/sdc`, ···
- SCSI 규격을 직렬 방식으로 발전시킨 인터페이스로, 주로 서버 및 스토리지 장비에 사용된다.
- 고성능, 고신뢰성이 요구되는 환경(데이터베이스 서버, 스토리지, 기업용 서버 등)에 적합하다.
- 일반적으로 엔터프라이즈급(서버용) SSD/HDD에 사용되는 규격이다.
- 장치명: `/dev/sda`, `/dev/sdb`, `/dev/sdc`, …

**정리**: IDE → SATA/SCSI → SAS로 이어지는 디스크 인터페이스의 발전 과정이며, 현재는 대부분 **SATA/SSD/SAS** 계열이 `/dev/sdX` 장치명으로 통합되어 표시된다.

---

## Partition

- **Partition**은 하드디스크의 저장 영역을 논리적으로 분할하여 사용하는 기술이다.
- 하나의 물리 디스크를 여러 개의 논리적 디스크처럼 나누어 관리할 수 있게 해 준다.
- 분할된 Partition은 독립적인 파일 시스템을 갖고, 각각을 별도로 마운트하여 운영한다.
- 각 Partition은 독립적인 디스크처럼 취급되며, Block 구조, 파일 시스템 종류(ext4, xfs, btrfs 등), 마운트 정책을 모두 개별적으로 구성할 수 있다.
- 리눅스는 파일 기반(Everything is file) 구조로 되어 있어 하나의 디스크에 모든 데이터를 저장하면 특정 디렉터리의 용량 폭주로 인해 전체 시스템에 영향을 주는 위험이 있기 때문에 Partition 분리가 매우 중요하다.

### Partition의 구성 목적

1) **멀티부팅(Multi-booting)**
   - 여러 개의 운영체제를 한 컴퓨터에서 사용하기 위해 필요하다.
   - 각각의 Partition은 완전히 독립적으로 동작하며 Windows / Linux 등의 이중 부팅 가능
   - 예: `/dev/sda1` : Windows
   - 예: `/dev/sda2` : Linux

2) **독립적인 정책 적용 가능**
   - 각각의 Partition에 따라 독립적인 정책이나 설정이 가능하다. (파일 시스템, 쿼터, 백업 설정, 보안 정책등을 독립적으로 구성가능)
   - `/home` : 사용자 데이터
   - `/var` : 로그, 캐시
   - `/boot` : 부팅 파일
   - `/tmp` : 임시 파일

3) **안정성 확보**
   - 특정 Partition이 손상되어도 나머지 영역의 데이터는 그대로 유지된다.
   - 특히 로그폭주, 특정 디렉터리 오류 등이 전체 시스템 장애로 이어지는 것을 방지한다.

4) **부팅 및 점검 최적화**
   - `/boot`를 별도로 분리하면 부팅 속도 향상, 복구 용이 등 관리에 유리하다.
   - **fsck**(파일 시스템 점검) 시 전체를 검사할 필요 없이 중요한 Partition만 검사할 수 있다.

5) **리소스 관리**
   - 특정 영역의 데이터 증가로인한 시스템 및 프로세스의 중단을 방지할 수 있다. 예를들어 `/var`를 별도로 분할하지 않으면 특정 데이터에 의해 모든 메모리 용량을 사용하게될수 도있다. 이런경우 시스템 또는 프로세스가 용량이 부족하여 다운되는 문제가 발생할 수 있다.
   - 예: `/var/log`에 로그가 폭주 → `/` 전체 용량이 100% → SSH 접속 불가 → 서비스 다운
   - 예: `/var`를 분리해두면 `/var`만 가득 차고 시스템은 다운되지 않는다.

### Partition의 종류

Partition은 **주 파티션(Primary Partition)**, **확장 파티션(Extend Partition)**, **논리 파티션(Logical Partition)**으로 구성된다.

- **주 파티션 (Primary Partition)**
  - 운영체제가 직접 인식하고 부팅 가능한 Partition으로 4개까지만 사용이 가능하다. (주 Partition이 4개가 아닌 주 Partition과 확장 Partition을 총괄하여 4개만 사용가능)
  - 만약 4개 이상의 Partition을 사용할 경우 확장 Partition을 사용해야한다.
  - Partition 구성시 Partition 번호는 1 ~ 4까지 할당된다.

- **확장 파티션 (Extend Partition)**
  - Extend Partition은 실제 데이터를 저장하는 파티션이 아니라, Logical Partition을 담는 컨테이너 역할 (확장 Partition임을 선언하는 Partition)
  - 확장 Partition은 하나의 디스크에 1개만 생성 가능 (확장 Partition안에서 다수의 logical Partition을 구성하기때문에 논리 파티션을 1개만 사용한다는 개념은 아니다.)

- **논리 파티션 (Logical Partition)**
  - Extended Partition 내부에서 생성되는 실제 데이터 저장 파티션
  - 개수에 제한이 없으며, 운영체제가 인식하는 일반 파티션처럼 사용 가능
  - 번호는 항상 5번부터 시작한다.
  - 이유: 1~4번은 Primary/Extended에 할당되기 때문에, Extended 안의 파티션은 자동으로 5번부터 번호가 매겨진다. (주 Partition을 2번까지 사용한 후 확장 Partition을 선언한 후 logical Partition을 생성해도 5번부터 할당된다.)

### 파티션 구성 예시

```
EX1) 2개의 Partiton이 필요한 환경

	|     P	|      P	|


	|     P	|      L	|
		-----E----

	|     L	|      L	|
	---------E---------




EX2) 3개의 Partiton이 필요한 환경


	|     P	|      P	|      P	|


	|     P	|     P	|      L	|
			-----E----

	|     P	|     L	|      L	|
		---------E---------

	|     L	|     L	|      L	|
	--------------E-------------


EX3) 4개의 Partiton이 필요한 환경

	|     P	|     P	|     P	|     P	|


	|     P	|     P	|     L	|     L	|
                           	---------E---------

	|     P	|     L	|     L	|     L	|
                           -------------E--------------





EX4) 5개의 Partiton이 필요한 환경

	|     P	|     P	|     P	|     L	|     L	|
                           		---------E---------

	|     P	|     P	|     L	|     L	|     L	|
                           	-------------E--------------

	|     P	|     L	|     L	|     L	|     L	|
                           ------------------E------------------

	|     P	|     P	|     P	|     P	|     L	|	<----- X (Partiton은 4개까지만 사용가능)
                           			----E----
```

**정리**: Partition은 시스템 안정성·보안·리소스 관리를 위해 분리하며, **Primary(최대 4개) → Extended(1개, 컨테이너) → Logical(5번부터, 무제한)** 구조로 4개 제한을 우회한다.

---

## File System

- 파일 시스템이란 운영체제가 Partition 또는 디스크 장치(하드디스크, SSD, USB, DVD 등)에 데이터를 저장, 삭제, 검색, 읽기, 쓰기 위해 사용하는 데이터 관리 체계를 의미한다.
- 단순히 파일을 담는 것뿐 아니라, 파일 이름, 크기, 권한, 소유자, 생성/수정 시간 등 메타데이터도 함께 관리한다.
- 시스템에 하드 디스크를 추가한 후 Partition을 사용하여 디스크를 논리적으로 분할한 후 Format을 사용하여 File System을 생성 할 수있다.
- 즉 Format을 통해서 File System을 구성할 수있다. 운영체제는 파일을 만들고 이름을 붙인뒤 디렉터리에 저장한다. 이 과정에서 파일의 길이를 제한하거나 확장명을 지정하거나 또는 파일의 크기를 제한하기도한다. (File System에 따라 파일 복구와 같은 기능의 지원 유/무도 달라질 수 있다.)
- 하드 디스크를 새로 추가하거나 또는 남아있는 메모리를 추가적으로 사용할 경우 파일 시스템 절차를 통해서 사용할 수 있다.

### EXT (Extended File System)

- CentOS 6까지 기본적용되는 File System
- 리눅스에서 일반적으로 많이 사용되는 File System
- **ext**: Minix 파일 시스템을 보완하여 만들어진 Linux 전용 파일 시스템
- **ext2**: ext의 다음 버전으로 고용량 디스크 사용등에 대비하여 확장성을 염두하여 만들어진 File System. 16GB의 단일 파일 생성 지원, 최대 4TB의 File System 구성 가능
- **ext3**: ext2의 확장버전. 2TB의 단일 파일 생성 지원, 최대 16TB의 File System 구성 가능. **Journaling File System**을 사용하여 시스템이 비정상적으로 종료시 파일을 복구할 수 있다.
- **ext4**: ext2, ext3과 호환이 가능하며 16TB의 단일 파일 생성 지원, 최대 1EB의 File System 구성 가능 (ext중 가장 효율적인 방식)

### XFS

- CentOS7 이상에서 기본적용되는 File System
- 높은 확장성과 고성능 64비트 파일시스템으로 최대 용량은 16EB를 지원하며 파일당 8EB를 지원한다.
- 파티션 최대 크기는 500TB이다.
- 메타 데이터 저널링 기능으로 장애시 빠른 복구를 지원
- 대규모 파일 시스템용 이기때문에 작은 용량의 환경에서는 성능이 저하된다. (최소 2TB이상에서 사용해야한다.)

### SWAP

- OS 설치시 기본적으로 RAM 용량과 같은 SWAP 파티션이 생성된다. (설치시 옵션을 사용하여 추가 적으로 용량 조절이 가능하다.)
- SWAP 파티션은 디스크 용량 일부를 가상의 메모리 공간으로 할당하며 운영중 실제 RAM 용량이 부족하게되면 SWAP 메모리를 혼용하여 사용한다. (RAM보다는 속도가 느리지만 안정적으로 OS를 운영할 수 있다.)
- **Red Hat 권장 SWAP 용량**

| RAM 용량 | SWAP 메모리 |
|---------|-----------|
| 4G 이하 | 최소 2GB |
| 4G ~ 16G | 최소 4GB |
| 16G ~ 64G | 최소 8GB |
| 64G ~ 256G | 최소 16GB |
| 256G ~ 512G | 최소 32GB |

### Journaling File System

- 파일 시스템 변경 사항을 저널(log)에 먼저 기록한 뒤 실제 데이터 블록에 반영하여 시스템이 갑자기 종료되거나 전원이 나가도, 저널 정보를 통해 빠른 복구 가능

### 파일 시스템 절차

1) **하드 디스크 추가**
   - 새로운 물리적 디스크를 서버에 장착하거나 가상머신에 추가
   - 디스크 종류(IDE, SATA, NVMe 등)에 따라 `/dev/sdX`, `/dev/nvmeXnY` 형태로 자동 인식됨
   - `lsblk`, `fdisk -l` 명령어로 추가된 디스크 확인 가능

2) **Partition 분할**
   - `fdisk` 명령어를 사용하여 Partition을 분할하여 사용한다.
   - 단, 테스트 환경이나 단순 목적이면 Partition 없이 디스크 전체에 파일 시스템을 생성해도 동작한다.

3) **파일 시스템 생성 (Format)**
   - `mkfs` 명령어로 Partition에 파일 시스템 구조를 기록
   - Format을 사용하여 File System 생성한다.
   - File System 생성은 `mkfs` 명령어를 사용하여 ext, xfs등 어떠한 파일 시스템을 사용할지를 정의한다.
     - **ext4**: 범용, 안정적
     - **xfs**: 대용량, 고성능 환경 적합
     - **vfat/ntfs**: 윈도우 호환

4) **Mount (마운트)**
   - 디스크는 단순히 인식만 된 상태이므로 파일 시스템을 특정 디렉터리에 연결해야 사용 가능
   - mount는 일시적 연결이므로 재부팅 후에도 자동 마운트를 원하면 `/etc/fstab`에 등록해야한다.

**정리**: **EXT4**는 범용/안정성, **XFS**는 대용량/고성능에 강점이 있으며, 새 디스크는 "**파티션 → mkfs로 포맷 → mount**" 3단계를 거쳐야 실제로 사용 가능해진다.

---

## Partition 구성 (fdisk 실습)

```bash
[root@localhost ~]# fdisk  -l			<---- 현재 구성된 Partiton 확인

[root@localhost ~]# fdisk  [경로/장치명]	<---- Partiton 구성
```

- **Partition을 나누기전 옵션**
  - `-p` : 설정한 Partiton을 확인하는 옵션
  - `-w` : 설정한 Partiton을 저정하는 옵션
  - `-d` : 설정한 Partiton을 삭제하는 옵션

- **Partition을 구성하는 옵션**
  - `-p` : 설정된 Partiton을 확인하는 옵션
  - `-n` : 새로운 파티션을 생성하는 옵션
  - `-p` : Primary Partiton을 구성하는 옵션 (주 파티션)
  - `-e` : Extend Partiton을 구성하는 옵션 (확장 파티션)
  - `-l` : Logical Partiton을 구성하는 옵션 (논리 파티션)

```bash
[root@localhost ~]# fisk  -l	 : Partiton 정보 확인

# df (disk free)
# 파일시스템 관점에서 "현재 마운트된 것" 위주로 보여줌
[root@localhost ~]# df	: Partiton , Memory , mount경로 정보 확인  (mount까지 설정된 정보만 확인 가능)
[root@localhost ~]# df -h	: 사람이 읽기 쉬운 단위(GB, MB)로 출력
[root@localhost ~]# df -T	: 파일시스템 종류도 함께 표시 (ext4, xfs 등)
[root@localhost ~]# df -Th	: 파일시스템 종류도 함께 표시하면서 사람이 읽기 쉬운 단위로 출력

#lsblk (list block devices)
# 디스크/파티션 관점에서 "블록 장치 전체"를 보여준다. (마운트되지 않은 파티션도 확인된다.)
[root@localhost ~]# lsblk	: Partiton , Memory , mount경로 정보 확인
[root@localhost ~]# lsblk -f	: 파일시스템 정보, LABEL, UUID 표시
```

### 1) 하드 디스크 추가 (남아있는 추가 메모리를 사용)

**EX1-1) VMware에 100G의 HDD를 추가하시오**

- Linux 시스템에 HDD를 추가하게되면 기본 경로는 `/dev`에 추가된다.
- HDD를 추가하게되면 장치명은 `/dev/sda`, `/dev/sdb`, `/dev/sdc`, `/dev/sdd` 순으로 생성된다.

```bash
[root@Server-A ~]# ls  -l  /dev | grep sd
brw-rw----  1 root disk      8,   0  7월 10 13:04 sda
brw-rw----  1 root disk      8,   1  7월 10 13:04 sda1
brw-rw----  1 root disk      8,   2  7월 10 13:04 sda2
brw-rw----  1 root disk      8,  16  7월 10 14:42 sdb
brw-rw----  1 root disk      8,  32  7월 10 13:04 sdc
brw-rw----  1 root disk      8,  48  7월 10 13:04 sdd
brw-rw----  1 root disk      8,  64  7월 10 13:04 sde
```

### 2) 'fdisk' 명령어를 사용하여 Partiton을 분할하여 사용한다.

**EX1-2) '/dev/sdb'의 100G 용량중 주 파티션으로 30G를 할당해야한다.**

```bash
[root@localhost ~]# fdisk  /dev/sdb
Welcome to fdisk (util-linux 2.23.2).

Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Device does not contain a recognized partition table
Building a new DOS disklabel with disk identifier 0x0595bc6e.

Command (m for help): n			<---- 새 파티션(new) 생성
Partition type:
   p   primary (0 primary, 0 extended, 4 free)	<---- 주 파티션
   e   extended				<---- 확장 파티션
Select (default p): p				<---- 주 파티션 선택
Partition number (1-4, default 1): 1		<---- 파티션 번호
First sector (2048-2147483647, default 2048):	<---- 섹터 지정 (운영체제에서 분할하기때문에 기본값 사용)
Using default value 2048
Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-209715199, default 209715199): +30G	<---- 할당할 메모리 용량 (30G 할당)

Created a new partition 1 of type 'Linux' and of size 30 GiB.t

Command (m for help): p			<---- 설정한 Partiton 확인
Disk /dev/sdb: 100 GiB, 107374182400 bytes, 209715200 sectors
Disk model: VMware Virtual S
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x266a5e81

Device     Boot Start       End Sectors    Size Id Type
/dev/sdb1        2048 62916607 62914560  30G 83 Linux

Command (m for help): w			<---- 설정한 Partiton 저장
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.

[root@Server-A ~]#
```

```bash
[root@Server-A ~]# ls  -l  /dev | grep sd
brw-rw----  1 root disk      8,   0  7월 10 13:04 sda
brw-rw----  1 root disk      8,   1  7월 10 13:04 sda1
brw-rw----  1 root disk      8,   2  7월 10 13:04 sda2
brw-rw----  1 root disk      8,  16  7월 10 14:42 sdb
brw-rw----  1 root disk      8,  17  7월 10 14:42 sdb1	<---- sdb Partiton 확인
brw-rw----  1 root disk      8,  32  7월 10 13:04 sdc
brw-rw----  1 root disk      8,  48  7월 10 13:04 sdd
brw-rw----  1 root disk      8,  64  7월 10 13:04 sde
```

```bash
[root@Server-A ~]# lsblk
NAME	MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
sda      	8:0    0    20G  0 disk
├─sda1	8:1    0    4G    0 part [SWAP]
└─sda2 	8:2    0    16G   0 part /
sdb      	8:16   0   100G  0 disk
└─sdb1	8:17   0   30G   0 part
sdc      	8:32   0   100G  0 disk
sdd      	8:48   0   100G  0 disk
sde      	8:64   0   100G  0 disk
sr0     	11:0    1  7.9G  0 rom
```

**EX1-3) '/dev/sdb'의 100G 용량중 주 파티션으로 20G를 할당해야한다.**

```bash
[root@Server-A ~]# fdisk  /dev/sdb

Welcome to fdisk (util-linux 2.37.4).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.


Command (m for help): n			<----
Partition type
   p   primary (1 primary, 0 extended, 3 free)
   e   extended (container for logical partitions)
Select (default p): p
Partition number (2-4, default 2): 2		<----
First sector (62916608-209715199, default 62916608):
Last sector, +/-sectors or +/-size{K,M,G,T,P} (62916608-209715199, default 209715199): +20G

Created a new partition 2 of type 'Linux' and of size 20 GiB.

Command (m for help): p			<----
Disk /dev/sdb: 100 GiB, 107374182400 bytes, 209715200 sectors
Disk model: VMware Virtual S
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x266a5e81

Device     Boot    Start        End  Sectors   Size Id Type
/dev/sdb1           2048  62916607  62914560  30G 83 Linux
/dev/sdb2      62916608 104859647 41943040  20G 83 Linux

Command (m for help): w			<----
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.
```

```bash
[root@Server-A ~]# ls  -l  /dev | grep sd
brw-rw----  1 root disk      8,   0  7월 10 13:04 sda
brw-rw----  1 root disk      8,   1  7월 10 13:04 sda1
brw-rw----  1 root disk      8,   2  7월 10 13:04 sda2
brw-rw----  1 root disk      8,  16  7월 10 14:48 sdb
brw-rw----  1 root disk      8,  17  7월 10 14:48 sdb1	# sdb 파티션 확인
brw-rw----  1 root disk      8,  18  7월 10 14:48 sdb2	# sdb 파티션 확인
brw-rw----  1 root disk      8,  32  7월 10 13:04 sdc
brw-rw----  1 root disk      8,  48  7월 10 13:04 sdd
brw-rw----  1 root disk      8,  64  7월 10 13:04 sde


[root@Server-A ~]# lsblk
NAME   	MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda      	8:0    0   20G  0 disk
├─sda1	8:1    0    4G  0 part [SWAP]
└─sda2 	8:2    0   16G  0 part /
sdb      	8:16   0  100G  0 disk
├─sdb1 	8:17   0   30G  0 part
└─sdb2	8:18   0   20G  0 part
sdc      	8:32   0  100G  0 disk
sdd      	8:48   0  100G  0 disk
sde     	8:64   0  100G  0 disk
sr0    	 11:0    1  7.9G  0 rom
```

**EX1-4) '/dev/sdb'의 100G 용량중 주 파티션으로 할당한 20G Partiton을 삭제해야한다.**

```bash
Welcome to fdisk (util-linux 2.37.4).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.


Command (m for help): d		<---- Partiton 삭제
Partition number (1,2, default 2): 2	<---- 2번 주 파티션

Partition 2 has been deleted.

Command (m for help): p		<---- Partiton 삭제 확인
Disk /dev/sdb: 100 GiB, 107374182400 bytes, 209715200 sectors
Disk model: VMware Virtual S
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x266a5e81

Device     Boot Start      End  Sectors Size Id Type
/dev/sdb1        2048 62916607 62914560  30G 83 Linux

Command (m for help): w		<---- 설정 저장
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.
```

```bash
[root@Server-A ~]# ls  -l  /dev | grep sd
brw-rw----  1 root disk      8,   0  7월 10 13:04 sda
brw-rw----  1 root disk      8,   1  7월 10 13:04 sda1
brw-rw----  1 root disk      8,   2  7월 10 13:04 sda2
brw-rw----  1 root disk      8,  16  7월 10 14:42 sdb
brw-rw----  1 root disk      8,  17  7월 10 14:42 sdb1	<---- sdb Partiton 확인
brw-rw----  1 root disk      8,  32  7월 10 13:04 sdc
brw-rw----  1 root disk      8,  48  7월 10 13:04 sdd
brw-rw----  1 root disk      8,  64  7월 10 13:04 sde


[root@Server-A ~]# lsblk
NAME	MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
sda      	8:0    0    20G  0 disk
├─sda1	8:1    0    4G    0 part [SWAP]
└─sda2 	8:2    0    16G   0 part /
sdb      	8:16   0   100G  0 disk
└─sdb1	8:17   0   30G   0 part
sdc      	8:32   0   100G  0 disk
sdd      	8:48   0   100G  0 disk
sde      	8:64   0   100G  0 disk
sr0     	11:0    1  7.9G  0 rom
```

**EX1-5) '/dev/sdb'의 100G 용량중 주 파티션으로 20G를 할당해야한다.**

```bash
[root@Server-A ~]# fdisk  /dev/sdb

Welcome to fdisk (util-linux 2.37.4).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.


Command (m for help): n		<---
Partition type
   p   primary (1 primary, 0 extended, 3 free)
   e   extended (container for logical partitions)
Select (default p): p			<---
Partition number (2-4, default 2):
First sector (62916608-209715199, default 62916608):
Last sector, +/-sectors or +/-size{K,M,G,T,P} (62916608-209715199, default 209715199): +20G

Created a new partition 2 of type 'Linux' and of size 20 GiB.

Command (m for help): p		<---
Disk /dev/sdb: 100 GiB, 107374182400 bytes, 209715200 sectors
Disk model: VMware Virtual S
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x266a5e81

Device     Boot    Start       End  Sectors Size Id Type
/dev/sdb1           2048  62916607 62914560  30G 83 Linux
/dev/sdb2       62916608 104859647 41943040  20G 83 Linux

Command (m for help): w		<---
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.
```

```bash
[root@Server-A ~]# fdisk -l

[root@Server-A ~]# ls  -l  /dev | grep sd
brw-rw----  1 root disk      8,   0  7월 10 13:04 sda
brw-rw----  1 root disk      8,   1  7월 10 13:04 sda1
brw-rw----  1 root disk      8,   2  7월 10 13:04 sda2
brw-rw----  1 root disk      8,  16  7월 10 14:54 sdb
brw-rw----  1 root disk      8,  17  7월 10 14:54 sdb1
brw-rw----  1 root disk      8,  18  7월 10 14:54 sdb2
brw-rw----  1 root disk      8,  32  7월 10 13:04 sdc
brw-rw----  1 root disk      8,  48  7월 10 13:04 sdd
brw-rw----  1 root disk      8,  64  7월 10 13:04 sde


[root@Server-A ~]# lsblk
NAME   	 MAJ:MIN RM    	SIZE RO	TYPE   	MOUNTPOINTS
sda      	      8:0     0	20G   0 	disk
├─sda1        8:1     0	4G     0 	part	[SWAP]		# 마운트된 디스크
└─sda2        8:2     0   	16G   0 	part 	/		# 마운트된 디스크
sdb      	      8:16   0	100G  0 	disk
├─sdb1        8:17   0 	30G   0 	part			# 주 파티션 1 (mount X)
└─sdb2        8:18   0	20G   0 	part			# 주 파티션 2 (mount X)
sdc      	      8:32   0	100G  0 	disk
sdd      	      8:48   0	100G  0 	disk
sde      	      8:64   0	100G  0 	disk
sr0     	      11:0   1	7.9G   0	rom


[root@Server-A ~]# lsblk  -f
NAME FSTYPE FSVER LABEL	UUID                                	 	FSAVAIL  FSUSE% 	MOUNTPOINTS
sda
├─sda1
│       swap   1                      	520bc18c-2b64-4df1-85e0-d126908ba6dd                	[SWAP]
└─sda2
         xfs                           	73bc277c-741d-4122-9c58-59ccd1889709 	10.4G       35% 	/
sdb
├─sdb1
│
└─sdb2

sdc
sdd
sde
```

**EX1-6) 아래의 조건에 맞게 Partiton을 구성하시오**
- '/dev/sdb'의 100G 용량중 논리 파티션으로 20G를 할당해야한다.
- '/dev/sdb'의 100G 용량중 논리 파티션으로 20G를 할당해야한다.
- '/dev/sdb'의 100G 용량중 논리 파티션으로 10G를 할당해야한다.

```bash
[root@Server-A ~]# fdisk  /dev/sdb

Welcome to fdisk (util-linux 2.37.4).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.


Command (m for help): n			# New Patition
Partition type
   p   primary (2 primary, 0 extended, 2 free)
   e   extended (container for logical partitions)
Select (default p): p				# 주 Patition
Partition number (3,4, default 3): 3		# Patition 번호 3
First sector (104859648-209715199, default 104859648):
Last sector, +/-sectors or +/-size{K,M,G,T,P} (104859648-209715199, default 209715199): +20G	# 20G 할당

Created a new partition 3 of type 'Linux' and of size 20 GiB.

Command (m for help): n			# New Patition
Partition type
   p   primary (3 primary, 0 extended, 1 free)
   e   extended (container for logical partitions)
Select (default e): e				# 확장 Patition

Selected partition 4
First sector (146802688-209715199, default 146802688):
Last sector, +/-sectors or +/-size{K,M,G,T,P} (146802688-209715199, default 209715199):		# 확장 Patition에 나머지 모든 용량 할당

Created a new partition 4 of type 'Extended' and of size 30 GiB.

Command (m for help): n
All primary partitions are in use.
Adding logical partition 5
First sector (146804736-209715199, default 146804736):
Last sector, +/-sectors or +/-size{K,M,G,T,P} (146804736-209715199, default 209715199): +20G	# 논리 Patition 5번에 20G 할당

Created a new partition 5 of type 'Linux' and of size 20 GiB.

Command (m for help): n			# New Patition
All primary partitions are in use.
Adding logical partition 6
First sector (188749824-209715199, default 188749824):
Last sector, +/-sectors or +/-size{K,M,G,T,P} (188749824-209715199, default 209715199):		# 논리 Patition 6번에 나머지 모든 용량 할당 (10G)

Created a new partition 6 of type 'Linux' and of size 10 GiB.

Command (m for help): w			# 설정 저장
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.

[root@Server-A ~]#
```

```bash
[root@Server-A ~]# ls  -l  /dev | grep sd
brw-rw----  1 root disk      8,   0  7월 10 13:04 sda
brw-rw----  1 root disk      8,   1  7월 10 13:04 sda1
brw-rw----  1 root disk      8,   2  7월 10 13:04 sda2
brw-rw----  1 root disk      8,  16  7월 10 15:07 sdb
brw-rw----  1 root disk      8,  17  7월 10 15:07 sdb1
brw-rw----  1 root disk      8,  18  7월 10 15:07 sdb2
brw-rw----  1 root disk      8,  19  7월 10 15:07 sdb3
brw-rw----  1 root disk      8,  20  7월 10 15:07 sdb4
brw-rw----  1 root disk      8,  21  7월 10 15:07 sdb5
brw-rw----  1 root disk      8,  22  7월 10 15:07 sdb6
brw-rw----  1 root disk      8,  32  7월 10 13:04 sdc
brw-rw----  1 root disk      8,  48  7월 10 13:04 sdd
brw-rw----  1 root disk      8,  64  7월 10 13:04 sde


[root@Server-A ~]# lsblk
NAME   MAJ:MIN RM  SIZE	RO 	TYPE	MOUNTPOINTS
sda     	  8:0      0    20G  	0 	disk
├─sda1	  8:1      0    4G  	0 	part 	[SWAP]
└─sda2	  8:2      0    16G  	0 	part 	/
sdb   	  8:16     0   100G  	0 	disk
├─sdb1 	  8:17     0   30G  	0 	part
├─sdb2	  8:18     0   20G  	0 	part
├─sdb3	  8:19     0   20G  	0 	part
├─sdb4	  8:20     0   1K  	0 	part
├─sdb5	  8:21     0   20G  	0 	part
└─sdb6	  8:22     0   10G  	0 	part
sdc      	  8:32     0   100G  	0 	disk
sdd      	  8:48     0   100G  	0 	disk
sde      	  8:64     0   100G  	0 	disk
```

**EX1-7) 설정한 모든 Partiton을 삭제해야한다.**

```bash
[root@Server-A ~]# fdisk  /dev/sdb
Welcome to fdisk (util-linux 2.37.4).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Command (m for help): d
Partition number (1-6, default 6):

Partition 6 has been deleted.

Command (m for help): d
Partition number (1-5, default 5):

Partition 5 has been deleted.

Command (m for help): d
Partition number (1-4, default 4):

Partition 4 has been deleted.

Command (m for help): d
Partition number (1-3, default 3):

Partition 3 has been deleted.

Command (m for help): d
Partition number (1,2, default 2):

Partition 2 has been deleted.

Command (m for help): d
Selected partition 1
Partition 1 has been deleted.

Command (m for help): w

The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.

[root@Server-A ~]#
```

**EX2) 100G의 '/dev/sdb'를 아래의 조건에 맞게 Partiton을 구성하시오**
- 30GB 용량의 Partiton을 구성하시오
- 30GB 용량의 Partiton을 구성하시오
- 10GB 용량의 Partiton을 구성하시오
- 10GB 용량의 Partiton을 구성하시오
- 10GB 용량의 Partiton을 구성하시오
- 10GB 용량의 Partiton을 구성하시오

```bash
[root@Server-A ~]# fdisk  /dev/sdb

Welcome to fdisk (util-linux 2.37.4).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.


Command (m for help): n
Partition type
   p   primary (0 primary, 0 extended, 4 free)
   e   extended (container for logical partitions)
Select (default p): p
Partition number (1-4, default 1): 1
First sector (2048-209715199, default 2048):
Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-209715199, default 209715199): +30G

Created a new partition 1 of type 'Linux' and of size 30 GiB.

Command (m for help): n
Partition type
   p   primary (1 primary, 0 extended, 3 free)
   e   extended (container for logical partitions)
Select (default p): p
Partition number (2-4, default 2): 2
First sector (62916608-209715199, default 62916608):
Last sector, +/-sectors or +/-size{K,M,G,T,P} (62916608-209715199, default 209715199): +30G

Created a new partition 2 of type 'Linux' and of size 30 GiB.

Command (m for help): n
Partition type
   p   primary (2 primary, 0 extended, 2 free)
   e   extended (container for logical partitions)
Select (default p): e
Partition number (3,4, default 3): 3
First sector (125831168-209715199, default 125831168):
Last sector, +/-sectors or +/-size{K,M,G,T,P} (125831168-209715199, default 209715199):

Created a new partition 3 of type 'Extended' and of size 40 GiB.

Command (m for help): n
All space for primary partitions is in use.
Adding logical partition 5
First sector (125833216-209715199, default 125833216):
Last sector, +/-sectors or +/-size{K,M,G,T,P} (125833216-209715199, default 209715199): +10G

Created a new partition 5 of type 'Linux' and of size 10 GiB.

Command (m for help): n
All space for primary partitions is in use.
Adding logical partition 6
First sector (146806784-209715199, default 146806784):
Last sector, +/-sectors or +/-size{K,M,G,T,P} (146806784-209715199, default 209715199): +10G

Created a new partition 6 of type 'Linux' and of size 10 GiB.

Command (m for help): n
All space for primary partitions is in use.
Adding logical partition 7
First sector (167780352-209715199, default 167780352):
Last sector, +/-sectors or +/-size{K,M,G,T,P} (167780352-209715199, default 209715199): +10G

Created a new partition 7 of type 'Linux' and of size 10 GiB.

Command (m for help): n
All space for primary partitions is in use.
Adding logical partition 8
First sector (188753920-209715199, default 188753920):
Last sector, +/-sectors or +/-size{K,M,G,T,P} (188753920-209715199, default 209715199):

Created a new partition 8 of type 'Linux' and of size 10 GiB.

Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.
```

```bash
[root@Server-A ~]# lsblk
NAME   MAJ:MIN RM  SIZE  RO TYPE	MOUNTPOINTS
sda      	  8:0    	0  20G  	0  disk
├─sda1	  8:1    	0  4G  	0  part 	[SWAP]
└─sda2	  8:2    	0  16G  	0  part 	/
sdb      	  8:16   	0  100G  	0  disk
├─sdb1	  8:17   	0  30G  	0  part	<---- Primary Partiton
├─sdb2	  8:18   	0  30G  	0  part	<---- Primary Partiton
├─sdb3	  8:19   	0  1K  	0  part	<---- Extend Partiton
├─sdb5	  8:21   	0  10G  	0  part	<---- Logical Partiton
├─sdb6	  8:22   	0  10G  	0  part	<---- Logical Partiton
├─sdb7	  8:23   	0  10G  	0  part	<---- Logical Partiton
└─sdb8	  8:24   	0  10G  	0  part	<---- Logical Partiton
sdc      	  8:32   	0  100G  	0  disk
sdd      	  8:48   	0  100G  	0  disk
sde      	  8:64   	0  100G	0  disk
```

**정리**: `fdisk`의 `n`(생성)/`p`(확인)/`d`(삭제)/`w`(저장) 명령을 조합해 **Primary → Extended → Logical** 순서로 파티션을 구성하며, `lsblk`/`ls -l /dev`로 매 단계 결과를 검증한다.

---

## File System 구성 (mkfs)

- 새로 HDD를 추가하고 Partition까지 나눈 상태라면, 아직 그 위에 File System(파일 시스템)이 없으므로 데이터를 저장할 수 없다.
- 리눅스에서는 **mkfs**(make file system) 명령어를 사용하여 파티션에 파일 시스템을 생성해야 한다.
- **mkfs** = Make File System (파일 시스템 생성 도구)
- 디스크 파티션 위에 데이터를 기록하고 관리할 수 있는 구조(ext4, xfs 등)를 만드는 과정
- 파티션을 나눴더라도 mkfs를 하지 않으면 OS가 해당 공간을 일반 데이터 저장소로 쓸 수 없다.

```bash
# 형식
mkfs  -t  [ext4 | xfs]  [경로/장치명]

mkfs.ext4  [경로/장치명]
mkfs.xfs  [경로/장치명]
```

- **ext4**: 범용, 안정성 높고 소규모부터 대규모까지 적합
- **xfs**: CentOS 7, Rocky 9 기본, 대용량 처리와 성능에 강점

### 3) Format을 사용하여 File System 생성한다.

**EX3-1) 아래의 조건에 맞게 File system을 구성하시오**
- 30GB의 HDD를 ext4형식으로 format 해야한다.

```bash
[root@Server-A ~]# mkfs  -t  ext4  /dev/sdb1
mke2fs 1.46.5 (30-Dec-2021)
Creating filesystem with 7864320 4k blocks and 1966080 inodes
Filesystem UUID: 1a5a6b71-4b4c-4476-b080-e409a37a59a9
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
NAME	FSTYPE FSVER LABEL     	UUID                                 		FSAVAIL	FSUSE%	MOUNTPOINTS
sda
├─sda1
│   	 swap    1                      	520bc18c-2b64-4df1-85e0-d126908ba6dd                	[SWAP]
└─sda2
     	xfs                           	73bc277c-741d-4122-9c58-59ccd1889709	10.4G	35% 	/
sdb
├─sdb1
│    	ext4   1.0                    	1a5a6b71-4b4c-4476-b080-e409a37a59a9
├─sdb2
│
├─sdb3
│
├─sdb5
│
├─sdb6
│
├─sdb7
│
└─sdb8

sdc
sdd
sde
```

### UUID (Universally Unique Identifier)

- UUID란 시스템에서 장치나 파일시스템을 구분하기 위해 사용하는 고유 식별값이다.
- 리눅스에서는 주로 디스크 파티션에 생성된 파일시스템을 식별할 때 사용한다.
- 예: `UUID=7f03c34a-3fb8-4a43-8c91-cae6fb8d1a28`
- 리눅스의 디스크와 파티션은 다음과 같은 장치 파일 이름을 사용한다: `/dev/sda`, `/dev/sda1`, `/dev/sdb`, `/dev/sdb1`
- 하지만 장치 파일 이름은 디스크 연결 순서나 시스템 환경에 따라 변경될 수 있다.
- 예를 들어 처음에는 다음과 같았던 파티션이 `/dev/sdb1`이었지만 재부팅 후 디스크 인식 순서가 `/dev/sdc1`로 변경될 수 있다.
- 장치 파일 이름이 변경되면 `/dev/sdb1`을 사용하도록 설정한 마운트 작업이 실패할 수 있다.
- 반면 **UUID**는 장치 이름이 바뀌어도 파일시스템이 동일하면 일반적으로 그대로 유지된다.
- 따라서 UUID를 사용하면 디스크 장치 이름이 변경되더라도 동일한 파일시스템을 정확하게 찾을 수 있다.

**EX3-2) 아래의 조건에 맞게 File system을 구성하시오**
- 30GB의 HDD를 xfs 형식으로 format 해야한다.
- 10GB의 HDD를 ext4 형식으로 format 해야한다.
- 10GB의 HDD를 ext4 형식으로 format 해야한다.
- 10GB의 HDD를 ext4 형식으로 format 해야한다.
- 10GB의 HDD를 ext4 형식으로 format 해야한다.

```bash
[root@Server-A ~]# mkfs  -t xfs  /dev/sdb2
meta-data=/dev/sdb2	isize=512    agcount=4, agsize=1966080 blks
	=                       	sectsz=512   attr=2, projid32bit=1
        	=                       	crc=1        finobt=1, sparse=1, rmapbt=0
         	=                       	reflink=1    bigtime=1 inobtcount=1 nrext64=0
data     	=                       	bsize=4096   blocks=7864320, imaxpct=25
         	=                       	sunit=0      swidth=0 blks
naming	=version 2            	bsize=4096   ascii-ci=0, ftype=1
log      	=internal log       	bsize=4096   blocks=16384, version=2
         	=                       	sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   	extsz=4096   blocks=0, rtextents=0



[root@localhost ~]# mkfs  -t ext4  /dev/sdb5
[root@localhost ~]# mkfs  -t ext4  /dev/sdb6
[root@localhost ~]# mkfs  -t ext4  /dev/sdb7
[root@localhost ~]# mkfs  -t ext4  /dev/sdb8
```

```bash
[root@Server-A ~]# lsblk -f
NAME 	FSTYPE FSVER LABEL 	UUID                                 		FSAVAIL	FSUSE%	MOUNTPOINTS
sda
├─sda1
│    	swap   1           		520bc18c-2b64-4df1-85e0-d126908ba6dd                	[SWAP]
└─sda2
     	xfs                		73bc277c-741d-4122-9c58-59ccd1889709	10.4G	35% 	/
sdb
├─sdb1
│    	ext4   1.0         		1a5a6b71-4b4c-4476-b080-e409a37a59a9
├─sdb2
│    	xfs                		86b1fde1-6181-461a-b28b-b92f24d1df82
├─sdb3
│
├─sdb5
│    	ext4   1.0         		dc9166ca-12fa-414f-9cc9-6d81bdee7c13
├─sdb6
│    	ext4   1.0         		1166e2cb-ed95-4d05-95e0-50e215ffb7de
├─sdb7
│    	ext4   1.0         		4299bb57-9f59-4296-8320-4d5262835db2
└─sdb8
     	ext4   1.0         		5445beef-5d7f-4edd-baba-5cc6f500b40f
sdc
sdd
sde
```

**정리**: `mkfs -t <fstype>` 또는 `mkfs.<fstype>` 형태로 파티션에 파일 시스템을 만들며, 생성 직후 부여되는 **UUID**는 장치명이 바뀌어도 유지되므로 이후 마운트/fstab 등록의 기준으로 사용된다.

---

## Mount

1) **신규 HDD는 연결만 하면 인식은 되지만 사용은 불가능하다.**
   - OS는 디스크 자체는 감지하지만 파티션이 없고 파일시스템이 없기 때문에 저장공간으로 사용할 수 없다.

2) **HDD에 파티션을 먼저 생성해야 한다.**
   - `fdisk`, `parted` 등을 사용하여 HDD를 1개 이상 파티션으로 나눈다. (파티션을 나누지 않아도 사용은 가능하지만 일반적으로 파티션은 나누고 사용한다.)
   - 파티션은 데이터를 저장하기 위한 논리적 구역

3) **파티션을 포맷하여 파일시스템을 구성해야 한다.**
   - 파일시스템은 저장 방식을 정의하는 구조(ext4, xfs 등)
   - 포맷 명령(`mkfs.xxxx`)을 실행해야 파일 저장이 가능해짐.

4) **파일시스템을 구성해도 마운트 포인트가 없으면 사용할 수 없다.**
   - 파일시스템이 있어도 Mount 하지 않으면 해당 파티션은 접근 불가능.
   - 마운트 포인트는 디렉터리이다.
   - `mount` 명령을 통해 디스크와 디렉터리를 연결해야 한다.

```bash
# 형식
mount  [경로/장치명]  [경로/디렉터리명]

umount  [경로/장치명]	       <---- mount 해제
umount  [경로/디렉터리명]	       <---- mount 해제
```

시스템이 물리적인 장치는 인식하지만 현재 물리적인 장치를 사용할 수는 없다. 해당 장치를 사용하기위해서는 물리적인 장치를 디렉터리에 **mount**라는 기능을 통해서 연결 후 사용이 가능하다. (windows에서 드라이버가 없는것처럼 사용은 불가능하다.)

**EX4-1) 'CD-Rom'을  '/media/cdrom'에 mount 해야한다.**

```bash
[root@Server-A ~]# mkdir  -p  /media/cdrom
```

```bash
[root@Server-A ~]# ls -lR /media/
/media/:
합계 0
drwxr-xr-x 2 root root 6  7월 10 15:59 cdrom

/media/cdrom:
합계 0
```

```bash
[root@Server-A ~]# ls -l  /dev/sr*
brw-rw----+ 1 root cdrom 11, 0  7월 10 13:04 /dev/sr0
```

```bash
[root@Server-A ~]# mount  /dev/sr0  /media/cdrom/
mount: /media/cdrom: WARNING: source write-protected, mounted read-only.
```

```bash
[root@Server-A ~]# cd  /media/cdrom/
합계 26
drwxr-xr-x 1 root root 2048  7월  5  2022 AppStream
drwxrwxr-x 1 root root 2048  7월  6  2022 BaseOS
-rw-r--r-- 1 root root 5504  7월  5  2022 COMMUNITY-CHARTER
-rw-r--r-- 1 root root 1394  7월  5  2022 Contributors
drwxrwxr-x 1 root root 2048  7월  5  2022 EFI
-rw-r--r-- 1 root root  372  7월  5  2022 EULA
-rw-r--r-- 1 root root 2204  7월  5  2022 LICENSE
-rw-r--r-- 1 root root 1750  7월  5  2022 RPM-GPG-KEY-Rocky-9
-rw-r--r-- 1 root root 3159  7월  5  2022 RPM-GPG-KEY-Rocky-9-Testing
drwxrwxr-x 1 root root 2048  7월  5  2022 images
drwxrwxr-x 1 root root 2048  7월  5  2022 isolinux
-rw-r--r-- 1 root root  102  7월  5  2022 media.repo
```

**EX4-2) 아래의 조건에 맞게 설정하시오**
- 최상위 디렉터리에 'sol' 디렉터리를 생성하시오
- '/sol' 디렉터리는 30GB 용량을 할당받아야한다.

```bash
[root@Server-A cdrom]# rm  -rf  /sol

[root@Server-A cdrom]# mkdir  /sol
```

```bash
[root@Server-A cdrom]# cp  /etc/a*  /sol/
cp: -r not specified; omitting directory '/etc/accountsservice'
cp: -r not specified; omitting directory '/etc/alsa'
cp: -r not specified; omitting directory '/etc/alternatives'
cp: -r not specified; omitting directory '/etc/audit'
cp: -r not specified; omitting directory '/etc/authselect'
cp: -r not specified; omitting directory '/etc/avahi'
```

```bash
[root@Server-A cdrom]# ls -l  /sol
합계 28
-rw-r--r-- 1 root root    16  7월 10 16:04 adjtime
-rw-r--r-- 1 root root 1529  7월 10 16:04 aliases
-rw-r--r-- 1 root root  541  7월 10 16:04 anacrontab
-rw-r--r-- 1 root root  269  7월 10 16:04 anthy-unicode.conf
-rw-r--r-- 1 root root  833  7월 10 16:04 appstream.conf
-rw-r--r-- 1 root root   55  7월 10 16:04 asound.conf
-rw-r--r-- 1 root root     1  7월 10 16:04 at.deny
```

```bash
[root@Server-A cdrom]# mount  /dev/sdb1  /sol
```

```bash
[root@Server-A cdrom]# df -h
Filesystem	Size   Used  Avail Use%  Mounted on
devtmpfs        	807M     	0  807M	 0%  /dev
tmpfs           	838M     	0  838M	 0%  /dev/shm
tmpfs           	335M  5.5M  330M	 2%  /run
/dev/sda2        	16G   5.6G   11G    35%  /
tmpfs           	168M    56K 168M	 1%  /run/user/42
tmpfs           	168M    40K 168M	 1%  /run/user/0
/dev/sr0        	7.9G   7.9G       0  100% /media/cdrom
/dev/sdb1        	30G     24K   28G	 1%  /sol		# mount 확인
```

```bash
[root@Server-A ~]# lsblk -f
NAME   FSTYPE  FSVER LABEL      	UUID                                 		FSAVAIL 	FSUSE% 	MOUNTPOINTS
sda
├─sda1 swap    1                          	520bc18c-2b64-4df1-85e0-d126908ba6dd                	[SWAP]
└─sda2 xfs                                	73bc277c-741d-4122-9c58-59ccd1889709	10.4G    	35% 	/
sdb
├─sdb1 ext4    1.0                        	1a5a6b71-4b4c-4476-b080-e409a37a59a9	27.8G 	0% 	/sol
├─sdb2 xfs                                	86b1fde1-6181-461a-b28b-b92f24d1df82
├─sdb3
├─sdb5 ext4    1.0                        	dc9166ca-12fa-414f-9cc9-6d81bdee7c13
├─sdb6 ext4    1.0                        	1166e2cb-ed95-4d05-95e0-50e215ffb7de
├─sdb7 ext4    1.0                        	4299bb57-9f59-4296-8320-4d5262835db2
└─sdb8 ext4    1.0                        	5445beef-5d7f-4edd-baba-5cc6f500b40f
sdc
sdd
sde
sr0    iso9660       Rocky-9-0-x86_64-dvd 2022-07-05-02-07-53-00           	0   	100% 	/media/cdrom
```

```bash
[root@Server-A ~]# mount | grep sdb1
/dev/sdb1 on /sol type ext4 (rw,relatime)
```

**EX4-2) 아래의 조건에 맞게 설정하시오**
- 최상위 디렉터리에 'cisco' 디렉터리를 생성하시오
- 'cisco' 디렉터리는 10GB 용량을 할당받아야한다.
- 최상위 디렉터리에 'linux' 디렉터리를 생성하시오
- '/linux/user' 디렉터리를 생성하시오
- '/linux/guest' 디렉터리를 생성하시오
- '/linux/nobody' 디렉터리를 생성하시오
- '/linux/user' 디렉터리에 30GB의 용량을 할당하시오
- '/linux/guest' 디렉터리에 10GB의 용량을 할당하시오
- '/linux/nobody' 디렉터리에 10GB의 용량을 할당하시오

```bash
[root@Server-A ~]# rm  -rf  /cisco

[root@Server-A ~]# rm  -rf  /linux


[root@Server-A ~]# mkdir -p  /cisco  /linux/user  /linux/guest  /linux/nobody

[root@Server-A ~]# mount  /dev/sdb5  /cisco
[root@Server-A ~]# mount  /dev/sdb2  /linux/user
[root@Server-A ~]# mount  /dev/sdb6  /linux/guest
[root@Server-A ~]# mount  /dev/sdb7  /linux/nobody
```

```bash
[root@Server-A ~]# df  -h
Filesystem	Size   Used Avail  Use%  Mounted on
devtmpfs        	807M	0  807M   	 0%  /dev
tmpfs           	838M     	0  838M 	 0%  /dev/shm
tmpfs           	335M  5.5M  330M 	 2%  /run
/dev/sda2        	16G    5.6G   11G	35%  /
tmpfs           	168M    56K  168M	 1%  /run/user/42
tmpfs           	168M    40K  168M	 1%  /run/user/0
/dev/sr0        	7.9G   7.9G     0    100% /media/cdrom
/dev/sdb1        	30G     24K   28G	 1%  /sol
/dev/sdb5       	9.8G    24K  9.3G 	 1%  /cisco
/dev/sdb2        	30G   247M   30G 	 1%  /linux/user
/dev/sdb6       	9.8G    24K  9.3G	 1%  /linux/guest
/dev/sdb7       	9.8G    24K  9.3G	 1%  /linux/nobody
```

```bash
[root@Server-A ~]# lsblk
NAME   MAJ:MIN RM  SIZE RO TYPE 	MOUNTPOINTS
sda      	  8:0    	0   20G  	0 disk
├─sda1	  8:1    	0    4G  	0 part 	[SWAP]
└─sda2	  8:2    	0   16G  	0 part 	/
sdb      	  8:16   	0  100G  	0 disk
├─sdb1	  8:17   	0   30G  	0 part 	/sol
├─sdb2	  8:18   	0   30G  	0 part 	/linux/user
├─sdb3	  8:19   	0    1K  	0 part
├─sdb5	  8:21   	0   10G  	0 part 	/cisco
├─sdb6	  8:22   	0   10G  	0 part 	/linux/guest
├─sdb7	  8:23   	0   10G  	0 part 	/linux/nobody
└─sdb8	  8:24   	0   10G  	0 part
sdc      	  8:32   	0  100G 	0 disk
sdd      	  8:48   	0  100G  	0 disk
sde      	  8:64   	0  100G  	0 disk
sr0    	 11:0    	1  7.9G  	0 rom  	/media/cdrom
```

**정리**: **mount**는 파일시스템이 생성된 장치를 특정 디렉터리에 연결해 실제로 접근 가능하게 만드는 마지막 단계이며, `umount`로 해제할 수 있다. 단, mount 자체는 일시적이라 재부팅 시 사라진다는 점이 다음 절의 Automount로 이어진다.

---

## Automount (7-2)

**EX1-1) 아래의 조건에 맞게 '/dev/sdb'에 Patition을 구성하시오**
- 30GB 용량의 HDD를 Patition해야한다.
- 20GB 용량의 HDD를 Patition해야한다.
- 20GB 용량의 HDD를 Patition해야한다.
- 20GB 용량의 HDD를 Patition해야한다.
- 10GB 용량의 HDD를 Patition해야한다. (남은 모든 용량 할당)

```bash
[root@localhost ~]# fdisk  -l
~~~~~~~~~~~~~ 중간 생략 ~~~~~~~~~~~~~
Disk /dev/sdb: 100 GiB, 107374182400 bytes, 209715200 sectors
Disk model: VMware Virtual S
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x266a5e81

Device     Boot     Start       End   Sectors Size Id Type
/dev/sdb1              2048  62916607   62914560  30G 83 Linux
/dev/sdb2        62916608 209715199 146798592  70G  5 Extended
/dev/sdb5        62918656 104861695  41943040  20G 83 Linux
/dev/sdb6       104863744 146806783  41943040  20G 83 Linux
/dev/sdb7       146808832 188751871  41943040  20G 83 Linux
/dev/sdb8       188753920 209715199  20961280  10G 83 Linux
~~~~~~~~~~~~~ 중간 생략 ~~~~~~~~~~~~~
```

```bash
[root@Server-A ~]# lsblk
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda      	  8:0    	0   20G  	0 disk
├─sda1	  8:1    	0    4G  	0 part    [SWAP]
└─sda2	  8:2    	0   16G  	0 part    /
sdb      	  8:16   	0  100G  	0 disk
├─sdb1	  8:17   	0   30G  	0 part
├─sdb2	  8:18   	0    1K  	0 part
├─sdb5	  8:21   	0   20G  	0 part
├─sdb6	  8:22   	0   20G  	0 part
├─sdb7	  8:23   	0   20G  	0 part
└─sdb8	  8:24   	0   10G  	0 part
sdc      	  8:32   	0  100G  	0 disk
sdd      	  8:48   	0  100G  	0 disk
sde      	  8:64   	0  100G  	0 disk
```

**EX1-2) 아래의 조건에 맞게 File system을 구성하시오**
- 30GB 용량의 HDD를 xfs 형식으로 format 해야한다.
- 20GB 용량의 HDD를 ext4 형식으로 format 해야한다.
- 20GB 용량의 HDD를 ext4 형식으로 format 해야한다.
- 20GB 용량의 HDD를 ext4 형식으로 format 해야한다.
- 10GB 용량의 HDD를 ext4 형식으로 format 해야한다.

```bash
[root@localhost ~]# mkfs  -t  xfs  /dev/sdb1
[root@localhost ~]# mkfs  -t  ext4  /dev/sdb5
[root@localhost ~]# mkfs  -t  ext4  /dev/sdb6
[root@localhost ~]# mkfs  -t  ext4  /dev/sdb7
[root@localhost ~]# mkfs  -t  ext4  /dev/sdb8

	OR

	# 기존 파일시스템이 남은경우 강제 덮어쓰기
[root@localhost ~]# mkfs  -t  xfs  -f  /dev/sdb1
[root@localhost ~]# mkfs  -t  ext4  -f  /dev/sdb5
[root@localhost ~]# mkfs  -t  ext4  -f  /dev/sdb6
[root@localhost ~]# mkfs  -t  ext4  -f  /dev/sdb7
[root@localhost ~]# mkfs  -t  ext4  -f  /dev/sdb8


	OR

[root@localhost ~]# mkfs.xfs  /dev/sdb1
[root@localhost ~]# mkfs.ext4  /dev/sdb5
[root@localhost ~]# mkfs.ext4  /dev/sdb6
[root@localhost ~]# mkfs.ext4  /dev/sdb7
[root@localhost ~]# mkfs.ext4  /dev/sdb8
```

```bash
[root@Server-A ~]# lsblk  -f
NAME   FSTYPE  FSVER LABEL     	UUID                                 		 FSAVAIL FSUSE% 	MOUNTPOINTS
sda
├─sda1 swap    1                          	520bc18c-2b64-4df1-85e0-d126908ba6dd                	[SWAP]
└─sda2 xfs                                	73bc277c-741d-4122-9c58-59ccd1889709 	 10.4G    	 35% 	/
sdb
├─sdb1 xfs                                	84c8cce4-41dd-40ca-9cc7-515da1bcd086
├─sdb2
├─sdb5 ext4    1.0                        	d5e90034-1c84-4599-8617-02bb70a1cc7e
├─sdb6 ext4    1.0                        	e1ea5da7-227c-4bf8-8ee8-9b12938d1593
├─sdb7 ext4    1.0                        	c32cb135-eebe-4419-91e4-03a3d6f18c2d
└─sdb8
sdc
sdd
sde
```

**EX1-3) 아래의 조건에 맞게 구성하시오**
- '/GIT' 디렉터리를 생성하시오
- '/homeSK/' 디렉터리를 생성하시오
- '/homeLG' 디렉터리를 생성하시오

```bash
[root@localhost ~]# mkdir  /GIT
[root@localhost ~]# mkdir  /homeSK
[root@localhost ~]# mkdir  /homeLG
```

```bash
[root@Server-A ~]# ls  -ld  /GIT
drwxr-xr-x 2 root root 6  7월 10 16:58 /GIT
```

```bash
[root@Server-A ~]# ls  -ld  /home*
drwxr-xr-x. 17 root root 4096  7월  9 15:41 /home
drwxr-xr-x   5 root root   48  7월  8 10:55 /homeA
drwxr-xr-x   5 root root   48  7월  8 10:55 /homeB
drwxr-xr-x   5 root root   48  7월  8 10:55 /homeC
drwxr-xr-x   5 root root   48  7월  8 11:55 /homeD
drwxr-xr-x   2 root root    6  7월 10 16:58 /homeLG
drwxr-xr-x   2 root root    6  7월 10 16:58 /homeSK
drwxr-xr-x   7 root root   86  7월  9 17:52 /homesol
```

**EX1-4) 아래의 조건에 맞게 구성하시오**
- '/GIT' 디렉터리는 30GB 용량의 메모리를 할당받아야한다.
- '/homeSK/' 디렉터리는 20GB 용량의 메모리를 할당받아야한다.
- '/homeSK/' 디렉터리에 'mGuest1', 'mGuest1', 'mGuest3' 계정을 생성하시오
- '/homeLG/' 디렉터리에 'mmUser1', 'mmUser2', 'mmUser3' 계정을 생성하시오
- '/homeLG/' 디렉터리안의 'mmUser1' 계정은 20GB의 메모리를 할당받아야한다.
- '/homeLG/' 디렉터리안의 'mmUser2' 계정은 20GB의 메모리를 할당받아야한다.
- '/homeLG/' 디렉터리안의 'mmUser3' 계정은 10GB의 메모리를 할당받아야한다.

```bash
[root@Server-A ~]# mount  /dev/sdb1  /GIT
```

```bash
[root@Server-A ~]# lsblk -f
NAME FSTYPE FSVER LABEL 	UUID                                 		FSAVAIL  FSUSE% 	MOUNTPOINTS
sda
├─sda1
│    swap   1           		520bc18c-2b64-4df1-85e0-d126908ba6dd                	[SWAP]
└─sda2
     xfs                			73bc277c-741d-4122-9c58-59ccd1889709	 10.4G 	35% 	/
sdb
├─sdb1
│    xfs                		84c8cce4-41dd-40ca-9cc7-515da1bcd086	 29.7G  	1% 	/GIT
├─sdb2
│
├─sdb5
│    ext4   1.0         		d5e90034-1c84-4599-8617-02bb70a1cc7e
├─sdb6
│    ext4   1.0         		e1ea5da7-227c-4bf8-8ee8-9b12938d1593
├─sdb7
│    ext4   1.0         		c32cb135-eebe-4419-91e4-03a3d6f18c2d
└─sdb8

sdc
sdd
sde
```

```bash
[root@Server-A ~]# mount  /dev/sdb5  /homeSK

[root@Server-A ~]# mUseradd  -md  /homeSK/mGuest1  mGuest1
[root@Server-A ~]# mUseradd  -md  /homeSK/mGuest2  mGuest2
[root@Server-A ~]# mUseradd  -md  /homeSK/mGuest3  mGuest3
```

```bash
[root@Server-A ~]# lsblk  -f
NAME   FSTYPE  FSVER LABEL    	UUID                                 		 FSAVAIL	 FSUSE%	 MOUNTPOINTS
sda
├─sda1 swap    1                          	520bc18c-2b64-4df1-85e0-d126908ba6dd                	 [SWAP]
└─sda2 xfs                                	73bc277c-741d-4122-9c58-59ccd1889709	 10.4G    	 35%	 /
sdb
├─sdb1 xfs                                	84c8cce4-41dd-40ca-9cc7-515da1bcd086	 29.7G  	 1% 	 /GIT	<------ GIT 디렉터리에 30GB의 용량이 할당
├─sdb2
├─sdb5 ext4    1.0                        	d5e90034-1c84-4599-8617-02bb70a1cc7e	 18.5G  	 0% 	 /homeSK
├─sdb6 ext4    1.0                        	e1ea5da7-227c-4bf8-8ee8-9b12938d1593
├─sdb7 ext4    1.0                        	c32cb135-eebe-4419-91e4-03a3d6f18c2d
└─sdb8
sdc
sdd
sde
```

```bash
[root@Server-A ~]# tail  -3  /etc/passwd
mGuest1:x:1313:1313::/homeSK/mGuest1:/bin/bash
mGuest2:x:1314:1314::/homeSK/mGuest2:/bin/bash
mGuest3:x:1315:1315::/homeSK/mGuest3:/bin/bash
```

```bash
[root@Server-A ~]# ls  -l  /homeSK
합계 28
drwx------ 2 root       root      16384  7월 10 16:57 lost+found	<---- EXT 형식으로 File system 구성시 'lost+found' 파일이 확인
drwx------ 3 mGuest1 mGuest1  4096  7월 10 17:32 mGuest1
drwx------ 3 mGuest2 mGuest2  4096  7월 10 17:32 mGuest2
drwx------ 3 mGuest3 mGuest3  4096  7월 10 17:32 mGuest3
```

```bash
[root@Server-A ~]# mUseradd  -md  /homeLG/mmUser1  mmUser1
[root@Server-A ~]# mUseradd  -md  /homeLG/mmUser2  mmUser2
[root@Server-A ~]# mUseradd  -md  /homeLG/mmUser3  mmUser3
```

```bash
[root@Server-A ~]# mount  /dev/sdb6  /homeLG/mmUser1
[root@Server-A ~]# mount  /dev/sdb7  /homeLG/mmUser2
[root@Server-A ~]# mount  /dev/sdb8  /homeLG/mmUser3
```

- 계정 생성 후 mount하게되면 기존의 파일들이 확인되지 않기때문에 디렉터리 생성 후 mount 하고 그다음 계정을 생성해야 한다.
- 만약 계정을 먼저 생성했다면 `/etc/skel`안의 모든 파일을 해당 계정으로 복사해야 한다.

```bash
[root@Server-A ~]# cp -a  /etc/skel/.  /homeLG/mmUser1
[root@Server-A ~]# cp -a  /etc/skel/.  /homeLG/mmUser2
[root@Server-A ~]# cp -a  /etc/skel/.  /homeLG/mmUser3
```

```bash
[root@Server-A ~]# lsblk  -f
NAME   FSTYPE  FSVER LABEL      	UUID                                 		FSAVAIL FSUSE% 	MOUNTPOINTS
sda
├─sda1 swap    1                          	520bc18c-2b64-4df1-85e0-d126908ba6dd                	[SWAP]
└─sda2 xfs                                	73bc277c-741d-4122-9c58-59ccd1889709 	10.4G    	35% 	/
sdb
├─sdb1 xfs                                	84c8cce4-41dd-40ca-9cc7-515da1bcd086	29.7G     	1% 	/GIT
├─sdb2
├─sdb5 ext4    1.0                        	d5e90034-1c84-4599-8617-02bb70a1cc7e	18.5G     	0% 	/homeSK
├─sdb6 ext4    1.0                        	e1ea5da7-227c-4bf8-8ee8-9b12938d1593	18.5G     	0% 	/homeLG/mmUser1
├─sdb7 ext4    1.0                        	c32cb135-eebe-4419-91e4-03a3d6f18c2d	18.5G     	0% 	/homeLG/mmUser2
└─sdb8 ext4    1.0                        	7dcefe4f-abe9-47af-bb25-e1e67031dfa7	9.2G     	0% 	/homeLG/mmUser3
sdc
sdd
sde
```

```bash
[root@Server-A ~]# reboot		# 서버 재부팅
```

```bash
[root@Server-A ~]# lsblk  -f
NAME   FSTYPE  FSVER LABEL   	UUID                                 		 FSAVAIL	 FSUSE%	 MOUNTPOINTS
sda
├─sda1 swap    1                  	520bc18c-2b64-4df1-85e0-d126908ba6dd                	 [SWAP]
└─sda2 xfs                                	73bc277c-741d-4122-9c58-59ccd1889709	 10.4G	 35% 	 /
sdb
├─sdb1 xfs                                	84c8cce4-41dd-40ca-9cc7-515da1bcd086
├─sdb2
├─sdb5 ext4    1.0                        	d5e90034-1c84-4599-8617-02bb70a1cc7e
├─sdb6 ext4    1.0                        	e1ea5da7-227c-4bf8-8ee8-9b12938d1593
├─sdb7 ext4    1.0                        	c32cb135-eebe-4419-91e4-03a3d6f18c2d
└─sdb8 ext4    1.0                        	7dcefe4f-abe9-47af-bb25-e1e67031dfa7
sdc
sdd
sde
```

- HDD연결후 Partition 및 File System을 구성 후 해당 HDD를 mount하게되면 해당 HDD를 사용할 수 있다.
- Server가 재부팅되면 설정한 모든 mount는 해제된다.
- **automount**를 사용하면 Server 재부팅되어도 기존의 mount는 해제되지 않는다.

### fstab을 이용한 Automount 설정

- 관리자가 특정 장치를 사용할 디렉터리에 mount 하게되면 해당 디렉터리를 사용하여 물리적 장치를 사용할 수 있다. 하지만 mount는 Server가 재부팅되면 umount 되어 다시 mount하기전까지 사용할 수 없다.
- **automount**를 설정하게되면 Server가 재부팅되어도 mount는 해제되지 않는다.
- automount의 관련된 설정은 `/etc/fstab`에 저장된다.

```bash
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
UUID=73bc277c-741d-4122-9c58-59ccd1889709	/   	xfs	defaults        0 0
UUID=520bc18c-2b64-4df1-85e0-d126908ba6dd	none	swap    	defaults        0 0
```

**EX2) 아래의 조건에 맞게 구성하시오**
- '/GIT' 디렉터리는 30GB 용량의 메모리를 할당받아야한다.
- '/homeSK/' 디렉터리는 20GB 용량의 메모리를 할당받아야한다.
- '/homeSK/' 디렉터리에 'mGuest1', 'mGuest1', 'mGuest3' 계정을 생성하시오
- '/homeLG/' 디렉터리에 'mUser1', 'mUser2', 'mUser3' 계정을 생성하시오
- '/homeLG/' 디렉터리안의 'mUser1' 계정은 20GB의 메모리를 할당받아야한다.
- '/homeLG/' 디렉터리안의 'mUser2' 계정은 20GB의 메모리를 할당받아야한다.
- '/homeLG/' 디렉터리안의 'mUser3' 계정은 10GB의 메모리를 할당받아야한다.
- 단 Server-A가 재부팅되어도 해당 Mount는 해제되지 않아야한다.

```bash
-sdb1 xfs		84c8cce4-41dd-40ca-9cc7-515da1bcd086
-sdb5 ext4	d5e90034-1c84-4599-8617-02bb70a1cc7e
-sdb6 ext4	e1ea5da7-227c-4bf8-8ee8-9b12938d1593
-sdb7 ext4	c32cb135-eebe-4419-91e4-03a3d6f18c2d
-sdb8 ext4	7dcefe4f-abe9-47af-bb25-e1e67031dfa7
```

```bash
[root@Server-A ~]# blkid
/dev/sda2: UUID="73bc277c-741d-4122-9c58-59ccd1889709" TYPE="xfs" PARTUUID="47a034bc-02"
/dev/sdb7: UUID="c32cb135-eebe-4419-91e4-03a3d6f18c2d" TYPE="ext4" PARTUUID="266a5e81-07"
/dev/sdb5: UUID="d5e90034-1c84-4599-8617-02bb70a1cc7e" TYPE="ext4" PARTUUID="266a5e81-05"
/dev/sdb1: UUID="84c8cce4-41dd-40ca-9cc7-515da1bcd086" TYPE="xfs" PARTUUID="266a5e81-01"
/dev/sdb8: UUID="7dcefe4f-abe9-47af-bb25-e1e67031dfa7" TYPE="ext4" PARTUUID="266a5e81-08"
/dev/sdb6: UUID="e1ea5da7-227c-4bf8-8ee8-9b12938d1593" TYPE="ext4" PARTUUID="266a5e81-06"
/dev/sr0: UUID="2022-07-05-02-07-53-00" LABEL="Rocky-9-0-x86_64-dvd" TYPE="iso9660" PTUUID="d719ec7f" PTTYPE="dos"
/dev/sda1: UUID="520bc18c-2b64-4df1-85e0-d126908ba6dd" TYPE="swap" PARTUUID="47a034bc-01"
```

#### 방법1 — 장치명 기준으로 fstab 등록

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
UUID=73bc277c-741d-4122-9c58-59ccd1889709	/   		xfs	defaults        	0 0
UUID=520bc18c-2b64-4df1-85e0-d126908ba6dd	none		swap    	defaults        	0 0
/dev/sdb1					/GIT		xfs	defaults		0 0
/dev/sdb5					/homeSK		ext4	defaults		0 0
/dev/sdb6					/homeLG/mUser1	ext4	defaults		0 0
/dev/sdb7					/homeLG/mUser2	ext4	defaults		0 0
/dev/sdb8					/homeLG/mUser3	ext4	defaults		0 0
:wq
```

**fstab 옵션 설명**

- **defaults** = 일반적인 HDD로 사용 (거의 대부분 defaults로 사용)
  - `exec` = 파일 실행 허용
  - `noexec` = 파일 실행 허용하지 않음
  - `ro` = 읽기 전용
  - `wr` = 읽기 쓰기 모두 가능
  - `user` = 일반 사용자도 사용 가능
  - `nouser` = 일반 사용자는 사용 불가능
  - `nosuid` = Set-UID, Set-GID사용 불가능
- 첫번째 필드(dump): `0` = dump를 사용하여 파일의 backup 사용 X, `1` = dump를 사용하여 파일을 backup 사용 O
- 두번째 필드(fsck 순서): `0` = 재부팅시 fdisk의 점검 X, `1` = 재부팅시 /root의 파일 시스템 부터 점검, `2` = 재부팅시 /root 이외의 파일 시스템부터 점검

#### 방법2 (권장 방식) — UUID 기준으로 fstab 등록

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
UUID=73bc277c-741d-4122-9c58-59ccd1889709	/   		xfs	defaults        	0 0
UUID=520bc18c-2b64-4df1-85e0-d126908ba6dd	none		swap    	defaults        	0 0
UUID=84c8cce4-41dd-40ca-9cc7-515da1bcd086	/GIT		xfs	defaults		0 0
UUID=d5e90034-1c84-4599-8617-02bb70a1cc7e	/homeSK		ext4	defaults		0 0
UUID=e1ea5da7-227c-4bf8-8ee8-9b12938d1593	/homeLG/mUser1	ext4	defaults		0 0
UUID=c32cb135-eebe-4419-91e4-03a3d6f18c2d	/homeLG/mUser2	ext4	defaults		0 0
UUID=7dcefe4f-abe9-47af-bb25-e1e67031dfa7	/homeLG/mUser3	ext4	defaults		0 0
:wq
```

```bash
-sdb1 xfs		84c8cce4-41dd-40ca-9cc7-515da1bcd086
-sdb5 ext4	d5e90034-1c84-4599-8617-02bb70a1cc7e
-sdb6 ext4	e1ea5da7-227c-4bf8-8ee8-9b12938d1593
-sdb7 ext4	c32cb135-eebe-4419-91e4-03a3d6f18c2d
-sdb8 ext4	7dcefe4f-abe9-47af-bb25-e1e67031dfa7
```

**정리**: `/etc/fstab`에 마운트 정보를 등록하면 재부팅 후에도 mount가 유지되며, 장치명(`/dev/sdbX`) 대신 **UUID**를 사용하는 방법2가 디스크 인식 순서 변경에 영향을 받지 않는 권장 방식이다.

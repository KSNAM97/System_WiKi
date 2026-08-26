# Linux 실습 종합 문제

> 파티션, 사용자 계정, LVM, NFS 통합 실습 문제

## 목차

1. [파티션 & 사용자 계정 통합 문제](#파티션--사용자-계정-통합-문제)
2. [LVM 연습 문제](#lvm-연습-문제)
3. [NFS 설정 문제](#nfs-설정-문제)

## 파티션 & 사용자 계정 통합 문제

### 문제 1: 파티션 생성 및 사용자 홈 디렉터리 설정

```bash
# 1. 새 디스크(/dev/sdb)에 3개의 파티션 생성
#    - /dev/sdb1: 2GB (사용자 홈 디렉터리용)
#    - /dev/sdb2: 3GB (데이터 공유 디렉터리용)
#    - /dev/sdb3: 1GB (백업용)

# 2. 각 파티션에 ext4 파일시스템 생성
mkfs.ext4 /dev/sdb1
mkfs.ext4 /dev/sdb2
mkfs.ext4 /dev/sdb3

# 3. 마운트 포인트 생성 및 마운트
mkdir /home2 /share /backup2
mount /dev/sdb1 /home2
mount /dev/sdb2 /share
mount /dev/sdb3 /backup2

# 4. /etc/fstab에 영구 등록
blkid /dev/sdb1    # UUID 확인
echo "UUID=xxx /home2   ext4 defaults 0 0" >> /etc/fstab
echo "UUID=xxx /share   ext4 defaults 0 0" >> /etc/fstab
echo "UUID=xxx /backup2 ext4 defaults 0 0" >> /etc/fstab

# 5. 사용자 생성 (홈 디렉터리를 /home2에 설정)
useradd -d /home2/user1 user1
useradd -d /home2/user2 user2
passwd user1
passwd user2

# 6. 공유 디렉터리 권한 설정
chmod 1777 /share    # Sticky bit 설정
```

### 문제 2: RAID 1 + LVM 구성

```bash
# 1. 2개의 디스크로 RAID 1 구성
mdadm --create /dev/md1 --level=1 --raid-devices=2 /dev/sdc /dev/sdd

# 2. RAID 상태 확인
mdadm --detail /dev/md1
cat /proc/mdstat

# 3. RAID 위에 LVM 구성
pvcreate /dev/md1
vgcreate vg_raid /dev/md1
lvcreate -L 8G -n lv_data vg_raid

# 4. 파일시스템 생성 및 마운트
mkfs.xfs /dev/vg_raid/lv_data
mkdir /data
mount /dev/vg_raid/lv_data /data

# 5. fstab 등록
echo "/dev/vg_raid/lv_data  /data  xfs  defaults  0  0" >> /etc/fstab
```

**정리**: 파티션 생성부터 사용자 홈 디렉터리 배정, **RAID 1 + LVM** 구성까지 스토리지·계정 관리 전 과정을 한 흐름으로 연습하는 문제이다.

---

## LVM 연습 문제

### 문제: 다음 조건으로 LVM 구성

```
- 물리 디스크: /dev/sdb, /dev/sdc
- 각 디스크에 5GB 파티션 생성
- VG 이름: vg_test
- LV 이름: lv_app (5GB), lv_log (3GB)
- 마운트: /app, /var/log/app
```

#### 풀이

```bash
# 1. 파티션 생성
fdisk /dev/sdb   # n → p → 1 → Enter → +5G → t → 8e → w
fdisk /dev/sdc   # n → p → 1 → Enter → +5G → t → 8e → w
partprobe /dev/sdb /dev/sdc

# 2. PV 생성
pvcreate /dev/sdb1 /dev/sdc1

# 3. VG 생성
vgcreate vg_test /dev/sdb1 /dev/sdc1
vgdisplay vg_test

# 4. LV 생성
lvcreate -L 5G -n lv_app vg_test
lvcreate -L 3G -n lv_log vg_test
lvs

# 5. 파일시스템 생성
mkfs.ext4 /dev/vg_test/lv_app
mkfs.ext4 /dev/vg_test/lv_log

# 6. 마운트
mkdir -p /app /var/log/app
mount /dev/vg_test/lv_app /app
mount /dev/vg_test/lv_log /var/log/app

# 7. 영구 등록
echo "/dev/vg_test/lv_app  /app        ext4 defaults 0 0" >> /etc/fstab
echo "/dev/vg_test/lv_log  /var/log/app ext4 defaults 0 0" >> /etc/fstab

# 8. 나중에 LV 확장 (디스크 공간이 있을 때)
lvextend -L +2G /dev/vg_test/lv_app
resize2fs /dev/vg_test/lv_app
```

**정리**: **PV → VG → LV** 순서로 논리 볼륨을 구성하고 파일시스템 생성, 마운트, `fstab` 등록, 이후 **lvextend**를 통한 확장까지 LVM의 전체 라이프사이클을 다룬다.

---

## NFS 설정 문제

### 문제: 다음 조건으로 NFS 설정

```
서버: 192.168.10.100
클라이언트: 192.168.10.200
공유 디렉터리: /share/nfs
조건: 클라이언트에서 읽기/쓰기 가능
```

#### 서버 설정

```bash
# 1. nfs-utils 설치
dnf install -y nfs-utils

# 2. 공유 디렉터리 생성
mkdir -p /share/nfs
chmod 777 /share/nfs

# 3. /etc/exports 설정
echo "/share/nfs 192.168.10.200(rw,sync,no_root_squash)" >> /etc/exports
exportfs -r

# 4. 서비스 시작
systemctl start nfs-server
systemctl enable nfs-server

# 5. 방화벽
firewall-cmd --permanent --add-service=nfs
firewall-cmd --permanent --add-service=rpc-bind
firewall-cmd --permanent --add-service=mountd
firewall-cmd --reload
```

#### 클라이언트 설정

```bash
# 1. 서버 공유 목록 확인
showmount -e 192.168.10.100

# 2. 마운트 포인트 생성 및 마운트
mkdir /mnt/nfs
mount -t nfs 192.168.10.100:/share/nfs /mnt/nfs

# 3. 영구 마운트
echo "192.168.10.100:/share/nfs  /mnt/nfs  nfs  defaults  0  0" >> /etc/fstab

# 4. 테스트
touch /mnt/nfs/test.txt
ls /mnt/nfs
```

**정리**: 서버에서 `/etc/exports`와 **nfs-server** 서비스, 방화벽을 설정하고 클라이언트에서 `showmount`로 확인 후 마운트·`fstab` 등록까지, NFS 서버/클라이언트 양쪽 설정을 모두 실습한다.

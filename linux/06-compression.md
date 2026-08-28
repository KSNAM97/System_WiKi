# Linux 압축 (Compress) / tar

## 1) gzip

압축은 파일이 사용하는 저장 공간을 줄이는 기능이다. 디스크 공간 절약, 네트워크 전송 시간 단축, 여러 파일을 하나로 묶어 관리, 백업 파일의 효율적 보관을 위해 사용하며, Windows에서는 알집·반디집 같은 GUI 프로그램을 주로 쓰지만 Linux에서는 gzip, bzip2, xz, tar 같은 명령어를 직접 사용하는 경우가 많다.

- 압축속도가 빠른반면 bzip2에 비해 압축률이 낮다.
- 작은 용량의 파일을 압축하는데 유리한다. (bzip2는 파일 압축시 기본 용량이 크기때문에 작은 파일 압축시에는 gzip이 더 효율적이다.)
- 서버 로그 파일 압축에 자주 사용된다.
- gzip은 파일 압축시 원본 파일을 삭제하고 파일명.gz 파일로 대체한다.

- 형식: `gzip  [경로/파일명]` ← 파일 압축
- 형식: `gzip  -d  [경로/파일명.gz]` ← 압축 해제
- 형식: `gunzip  [경로/파일명.gz]` ← 압축 해제

## 2) bzip2

- 압축 속도는 느리지만 압출율이 높다.
- 일반적으로 가장 많이 사용하는 압축 방식 (작은 용량 압축시에는 gzip이 효율적이다.)
- 서버 백업, 프로그램 소스 배포 파일에서 많이 사용

- 형식: `bzip2  [경로/파일명]` ← 파일 압축
- 형식: `bzip2  -d  [경로/파일명.bz2]` ← 압축 해제
- 형식: `bunzip2  [경로/파일명.bz2]` ← 압축 해제

## 3) xz

- 압축률이 제일 높다.
- 압축 속도는 가장 느림 (CPU 많이 사용)
- 대용량 파일 압축시 사용하는 압축방식
- 용량이 큰 ISO, 백업 파일 압축에 적합 (일반 파일 작업에는 잘 사용하지 않는다.)

- 형식: `xz  [경로/파일명]` ← 파일 압축
- 형식: `xz  -d  [경로/파일명.xz]` ← 압축 해제
- 형식: `unxz  [경로/파일명.xz]` ← 압축 해제

**정리**: **gzip**은 속도, **bzip2**는 압축률과 범용성, **xz**는 최고 압축률을 강점으로 하며, 세 명령 모두 파일 단위로 동작하고 압축 시 원본 파일을 삭제한다는 공통점이 있다.

---

## 압축 명령어 실습 (gzip / bzip2 / xz)

### EX1) 최상위 디렉터리에 '/backup' 디렉터리를 생성한 후 /etc/' 디렉터리안에서 크기가 가장큰 10개의 파일을 '/backup' 디렉터리안으로 복사

```
[root@Server-A ~]# rm -rf /backup/*

[root@Server-A ~]# ls  -lS  /etc | head -20
합계 1376
-rw-r--r--.  1 root root    692252  6월 23  2020 services
-rw-r--r--.  1 root root     67454  4월 22  2020 mime.types
-rw-r--r--   1 root root     43431  7월  7 12:58 ld.so.cache
-rw-r--r--.  1 root root     28974  5월 26  2022 brltty.conf
-rw-r--r--.  1 root dnsmasq  27839  5월 20 07:34 dnsmasq.conf
-rw-r--r--.  1 root root     10373  5월  2  2025 nanorc
-rw-r--r--.  1 root root      9077  7월  2 15:07 kdump.conf
-rw-r--r--.  1 root root      7779 12월 29  2025 login.defs
-rw-r--r--.  1 root root      6568  6월 23  2020 protocols
-rw-r--r--.  1 root root      6300  5월 15  2022 pnm2ppa.conf
-rw-r--r--.  1 root root      5799  3월 13 04:12 idmapd.conf
-rw-r--r--.  1 root root      5235 11월  3  2025 man_db.conf
-rw-r--r--.  1 root root      5122  2월 16 20:56 makedumpfile.conf.sample
-rw-r--r--.  1 root root      4925  9월  4  2024 wgetrc
-rw-r--r--.  1 root root      4760  5월 16  2022 enscript.cfg
-rw-r--r--.  1 root root      4755  6월 25 02:35 DIR_COLORS.lightbgcolor
-rw-r--r--.  1 root root      4673  6월 25 02:35 DIR_COLORS
-rw-r-----.  1 root root      4356  5월 20 10:40 sudo.conf

[root@Server-A ~]# cp  /etc/services \
 /etc/mime.types \
 /etc/ld.so.cache \
 /etc/brltty.conf \
 /etc/dnsmasq.conf \
 /etc/nanorc \
 /etc/kdump.conf \
 /etc/login.defs \
 /etc/protocols \
 /etc/pnm2ppa.conf \
 /etc/idmapd.conf \
 /etc/man_db.conf \
 /backup

[root@Server-A ~]# cp  /etc/services /etc/mime.types  /etc/ld.so.cache  /etc/brltty.conf 
```

### EX1-2) '/backup' 디렉터리안의 모든 파일을 '/home/guest' 디렉터리로 복사해야한다.

```
[root@Server-A ~]# rm -rf /home/guest/*

[root@Server-A ~]# cp  /backup/*  /home/guest/

[root@Server-A ~]# ls -l /home/guest
합계 916
-rw-r--r-- 1 root root  28974  7월 10 09:57 brltty.conf
-rw-r--r-- 1 root root  27839  7월 10 09:57 dnsmasq.conf
-rw-r--r-- 1 root root   5799  7월 10 09:57 idmapd.conf
-rw-r--r-- 1 root root   9077  7월 10 09:57 kdump.conf
-rw-r--r-- 1 root root  43431  7월 10 09:57 ld.so.cache
-rw-r--r-- 1 root root   7779  7월 10 09:57 login.defs
-rw-r--r-- 1 root root   5235  7월 10 09:57 man_db.conf
-rw-r--r-- 1 root root  67454  7월 10 09:57 mime.types
-rw-r--r-- 1 root root  10373  7월 10 09:57 nanorc
-rw-r--r-- 1 root root   6300  7월 10 09:57 pnm2ppa.conf
-rw-r--r-- 1 root root   6568  7월 10 09:57 protocols
-rw-r--r-- 1 root root 692252  7월 10 09:57 services
```

---

### EX2-1) '/home/guest' 디렉터리안의 'services' 파일을 gzip을 사용하여 압축하시오

```
[root@Server-A ~]# gzip  /home/guest/services

[root@Server-A ~]# ls -l /home/guest
합계 376
-rw-r--r-- 1 root root  28974  7월 10 09:57 brltty.conf
-rw-r--r-- 1 root root   27839  7월 10 09:57 dnsmasq.conf
-rw-r--r-- 1 root root    5799  7월 10 09:57 idmapd.conf
-rw-r--r-- 1 root root    9077  7월 10 09:57 kdump.conf
-rw-r--r-- 1 root root   43431  7월 10 09:57 ld.so.cache
-rw-r--r-- 1 root root    7779  7월 10 09:57 login.defs
-rw-r--r-- 1 root root    5235  7월 10 09:57 man_db.conf
-rw-r--r-- 1 root root  67454  7월 10 09:57 mime.types
-rw-r--r-- 1 root root   10373  7월 10 09:57 nanorc
-rw-r--r-- 1 root root    6300  7월 10 09:57 pnm2ppa.conf
-rw-r--r-- 1 root root    6568  7월 10 09:57 protocols
-rw-r--r-- 1 root root 142528  7월 10 09:57 services.gz	# 파일 압축 확인 (원본파일인 services는 삭제된다.)
```

### EX2-2) '/home/guest' 디렉터리안의 'services.gz' 파일의 압축을 풀어야 한다.

```
[root@Server-A ~]# gunzip /home/guest/services.gz

[root@Server-A ~]# ls -l /home/guest
합계 916
-rw-r--r-- 1 root root  28974  7월 10 09:57 brltty.conf
-rw-r--r-- 1 root root  27839  7월 10 09:57 dnsmasq.conf
-rw-r--r-- 1 root root   5799  7월 10 09:57 idmapd.conf
-rw-r--r-- 1 root root   9077  7월 10 09:57 kdump.conf
-rw-r--r-- 1 root root  43431  7월 10 09:57 ld.so.cache
-rw-r--r-- 1 root root   7779  7월 10 09:57 login.defs
-rw-r--r-- 1 root root   5235  7월 10 09:57 man_db.conf
-rw-r--r-- 1 root root  67454  7월 10 09:57 mime.types
-rw-r--r-- 1 root root  10373  7월 10 09:57 nanorc
-rw-r--r-- 1 root root   6300  7월 10 09:57 pnm2ppa.conf
-rw-r--r-- 1 root root   6568  7월 10 09:57 protocols
-rw-r--r-- 1 root root 692252  7월 10 09:57 services	# gzip 압축해제
```

### EX2-3) '/home/guest' 디렉터리안의 'services' 파일을 bzip2를 사용하여 압축하시오

```
[root@Server-A ~]# bzip2  /home/guest/services

[root@Server-A ~]# ls -lSh /home/guest
합계 364K
-rw-r--r-- 1 root root 127K  7월 10 09:57 services.bz2	# bzip2로 압축
-rw-r--r-- 1 root root  66K  7월 10 09:57 mime.types
-rw-r--r-- 1 root root  43K  7월 10 09:57 ld.so.cache
-rw-r--r-- 1 root root  29K  7월 10 09:57 brltty.conf
-rw-r--r-- 1 root root  28K  7월 10 09:57 dnsmasq.conf
-rw-r--r-- 1 root root  11K  7월 10 09:57 nanorc
-rw-r--r-- 1 root root 8.9K  7월 10 09:57 kdump.conf
-rw-r--r-- 1 root root 7.6K  7월 10 09:57 login.defs
-rw-r--r-- 1 root root 6.5K  7월 10 09:57 protocols
-rw-r--r-- 1 root root 6.2K  7월 10 09:57 pnm2ppa.conf
-rw-r--r-- 1 root root 5.7K  7월 10 09:57 idmapd.conf
-rw-r--r-- 1 root root 5.2K  7월 10 09:57 man_db.conf3

[root@Server-A ~]# ls -lSh /home/guest
합계 364K
-rw-r--r-- 1 root root 127K  7월 10 09:57 services.bz2	# bzip2로 압축 (127K)
-rw-r--r-- 1 root root  66K  7월 10 09:57 mime.types
-rw-r--r-- 1 root root  43K  7월 10 09:57 ld.so.cache
-rw-r--r-- 1 root root  29K  7월 10 09:57 brltty.conf
-rw-r--r-- 1 root root  28K  7월 10 09:57 dnsmasq.conf
-rw-r--r-- 1 root root  11K  7월 10 09:57 nanorc
-rw-r--r-- 1 root root 8.9K  7월 10 09:57 kdump.conf
-rw-r--r-- 1 root root 7.6K  7월 10 09:57 login.defs
-rw-r--r-- 1 root root 6.5K  7월 10 09:57 protocols
-rw-r--r-- 1 root root 6.2K  7월 10 09:57 pnm2ppa.conf
-rw-r--r-- 1 root root 5.7K  7월 10 09:57 idmapd.conf
-rw-r--r-- 1 root root 5.2K  7월 10 09:57 man_db.conf
```

### EX2-4) '/home/guest' 디렉터리안의 'services' 파일의 압축을 풀어야 한다

```
[root@Server-A ~]# bunzip2  /home/guest/services.bz2

[root@Server-A ~]# ls -lSh /home/guest
합계 916K
-rw-r--r-- 1 root root 677K  7월 10 09:57 services
-rw-r--r-- 1 root root  66K  7월 10 09:57 mime.types
-rw-r--r-- 1 root root  43K  7월 10 09:57 ld.so.cache
-rw-r--r-- 1 root root  29K  7월 10 09:57 brltty.conf
-rw-r--r-- 1 root root  28K  7월 10 09:57 dnsmasq.conf
-rw-r--r-- 1 root root  11K  7월 10 09:57 nanorc
-rw-r--r-- 1 root root 8.9K  7월 10 09:57 kdump.conf
-rw-r--r-- 1 root root 7.6K  7월 10 09:57 login.defs
-rw-r--r-- 1 root root 6.5K  7월 10 09:57 protocols
-rw-r--r-- 1 root root 6.2K  7월 10 09:57 pnm2ppa.conf
-rw-r--r-- 1 root root 5.7K  7월 10 09:57 idmapd.conf
-rw-r--r-- 1 root root 5.2K  7월 10 09:57 man_db.conf
```

---

### EX3-1) 아래의 조건에 맞게 설정하시오
- '/home/guest' 디렉터리안의 'ld.so.cache' 파일을 gzip으로 압축하시오
- '/home/guest' 디렉터리안의 'mime.types' 파일을 bzip2로 압축하시오
- '/home/guest' 디렉터리안의 'brltty.conf' 파일을 xz으로 압축하시오

```
[root@localhost ~]# gzip  /home/guest/ld.so.cache
[root@localhost ~]# bzip2  /home/guest/mime.types
[root@localhost ~]# xz  /home/guest/brltty.conf

[root@Server-A ~]# ls  -lSh  /home/guest
합계 812K
-rw-r--r-- 1 root root 677K  7월 10 09:57 services
-rw-r--r-- 1 root root  28K  7월 10 09:57 dnsmasq.conf
-rw-r--r-- 1 root root  17K  7월 10 09:57 mime.types.bz2	# bzip2 압축
-rw-r--r-- 1 root root  11K  7월 10 09:57 ld.so.cache.gz	# gzip 압축
-rw-r--r-- 1 root root  11K  7월 10 09:57 nanorc
-rw-r--r-- 1 root root 8.9K  7월 10 09:57 kdump.conf
-rw-r--r-- 1 root root 7.7K  7월 10 09:57 brltty.conf.xz	# xz 압축
-rw-r--r-- 1 root root 7.6K  7월 10 09:57 login.defs
-rw-r--r-- 1 root root 6.5K  7월 10 09:57 protocols
-rw-r--r-- 1 root root 6.2K  7월 10 09:57 pnm2ppa.conf
-rw-r--r-- 1 root root 5.7K  7월 10 09:57 idmapd.conf
-rw-r--r-- 1 root root 5.2K  7월 10 09:57 man_db.conf
```

### EX3-2) 아래의 조건에 맞게 설정하시오
- '/home/guest' 디렉터리안에 압축된 파일이 확인되지 않아야한다.
- 기존 파일 10개는 그대록 확인되어야 한다.

```
[root@localhost ~]# gunzip  /home/guest/ld.so.cache.gz
[root@localhost ~]# bunzip2  /home/guest/mime.types.bz2
[root@localhost ~]# unxz  /home/guest/brltty.conf.xz

[root@Server-A ~]# ls -lSh /home/guest
합계 916K
-rw-r--r-- 1 root root 677K  7월 10 09:57 services
-rw-r--r-- 1 root root  66K  7월 10 09:57 mime.types
-rw-r--r-- 1 root root  43K  7월 10 09:57 ld.so.cache
-rw-r--r-- 1 root root  29K  7월 10 09:57 brltty.conf
-rw-r--r-- 1 root root  28K  7월 10 09:57 dnsmasq.conf
-rw-r--r-- 1 root root  11K  7월 10 09:57 nanorc
-rw-r--r-- 1 root root 8.9K  7월 10 09:57 kdump.conf
-rw-r--r-- 1 root root 7.6K  7월 10 09:57 login.defs
-rw-r--r-- 1 root root 6.5K  7월 10 09:57 protocols
-rw-r--r-- 1 root root 6.2K  7월 10 09:57 pnm2ppa.conf
-rw-r--r-- 1 root root 5.7K  7월 10 09:57 idmapd.conf
-rw-r--r-- 1 root root 5.2K  7월 10 09:57 man_db.conf
```

**정리**: 하나의 파일을 gzip/bzip2/xz로 압축·해제하는 흐름을 실습했으며, 압축 시 원본이 사라지고 `.gz`/`.bz2`/`.xz` 확장자가 붙는 것을 확인했다.

---

## tar (tape archive)

- 리눅스에서 gzip, bzip2, xz는 기본적으로 하나의 파일 단위로 압축한다.
  - `gzip file1 file2 file3`
  - 위 명령어는 여러 파일을 한 번에 지정할 수 있지만, 하나의 압축 파일로 생성되는 것이 아니라 각각 별도로 압축된다.
  - `file1.gz`
  - `file2.gz`
  - `file3.gz`
  - 따라서 10개의 파일을 gzip으로 압축하면 10개의 압축 파일이 생성되며, 각각의 압축 파일을 따로 관리해야 한다.

- 여러 파일과 디렉터리를 하나의 파일로 관리하려면 먼저 tar를 사용하여 하나의 아카이브 파일로 묶는다.
- tar는 Tape Archive의 약자로, 여러 개의 파일과 디렉터리를 하나의 파일로 묶어 관리하는 명령어이다.
- tar 자체의 기본 기능은 압축이 아니라 여러 파일을 하나로 묶는 아카이브 기능이다.
- tar에 gzip, bzip2, xz 옵션을 함께 사용하면 여러 파일을 하나로 묶은 후 압축할 수 있다.

- 형식: `tar  [ 옵션 (필수 옵션)  (선택 옵션)]  [저장할 파일명.tar]  [Source File1]  [Source File2]   [Source File3]`

- `-c` : tar을 사용하여 복수개의 파일을 하나의 파일로 묶는 옵션
- `-x` : tar을 사용하여 하나로 묶은 파일을 다시 원본의 낱개 파일로 복구하는 옵션
- `-v` : tar을 사용하여 파일을 묶거나 또는 파일을 낱개로 복구시 과정을 출력하는 옵션
- `-f` : tar을 사용하여 파일을 묶거나 또는 파일을 낱개로 복구시 파일명을 직접 입력하는 옵션
- `-z` : tar를 사용하여 파일을 묶고난후 gzip을 사용하여 tar 파일을 압축
- `-j` : tar를 사용하여 파일을 묶고난후 bzip2를 사용하여 tar 파일을 압축

---

```
[root@Server-A ~]# ls  -lSh  /home/guest
합계 916K
-rw-r--r-- 1 root root 677K  7월 10 09:57 services
-rw-r--r-- 1 root root  66K  7월 10 09:57 mime.types
-rw-r--r-- 1 root root  43K  7월 10 09:57 ld.so.cache
-rw-r--r-- 1 root root  29K  7월 10 09:57 brltty.conf
-rw-r--r-- 1 root root  28K  7월 10 09:57 dnsmasq.conf
-rw-r--r-- 1 root root  11K  7월 10 09:57 nanorc
-rw-r--r-- 1 root root 8.9K  7월 10 09:57 kdump.conf
-rw-r--r-- 1 root root 7.6K  7월 10 09:57 login.defs
-rw-r--r-- 1 root root 6.5K  7월 10 09:57 protocols
-rw-r--r-- 1 root root 6.2K  7월 10 09:57 pnm2ppa.conf
-rw-r--r-- 1 root root 5.7K  7월 10 09:57 idmapd.conf
-rw-r--r-- 1 root root 5.2K  7월 10 09:57 man_db.conf
```

### EX1-1) '/home/guest/' 디렉터리의 'services', 'dnsmasq.conf', 'mime.types'를 하나의 파일로 묶어야한다.

```
[root@Server-A ~]# cd /home/guest

[root@Server-A guest]# pwd
/home/guest

[root@Server-A guest]# tar  -cvf  GUEST  services  dnsmasq.conf  mime.types

[root@Server-A guest]# ls  -lSh
합계 1.7M
-rw-r--r-- 1 root root 780K  7월 10 10:33 GUEST		# tar 파일
-rw-r--r-- 1 root root 677K  7월 10 09:57 services		# 원본 파일
-rw-r--r-- 1 root root  66K  7월 10 09:57 mime.types	# 원본 파일
-rw-r--r-- 1 root root  43K  7월 10 09:57 ld.so.cache
-rw-r--r-- 1 root root  29K  7월 10 09:57 brltty.conf
-rw-r--r-- 1 root root  28K  7월 10 09:57 dnsmasq.conf	# 원본 파일
-rw-r--r-- 1 root root  11K  7월 10 09:57 nanorc
-rw-r--r-- 1 root root 8.9K  7월 10 09:57 kdump.conf
-rw-r--r-- 1 root root 7.6K  7월 10 09:57 login.defs
-rw-r--r-- 1 root root 6.5K  7월 10 09:57 protocols
-rw-r--r-- 1 root root 6.2K  7월 10 09:57 pnm2ppa.conf
-rw-r--r-- 1 root root 5.7K  7월 10 09:57 idmapd.conf
-rw-r--r-- 1 root root 5.2K  7월 10 09:57 man_db.conf
```

- tar를 사용해서 복수개의 파일을 하나의 파일로 묶어도 원본 파일은 삭제되지 않는다.
- tar를 사용시 파일명만 작성하게되면 해당 파일이 일반 파일인지 tar 파일인지 확인할 수 없다.

### EX1-2) 아래의 조건에 맞게 설정하시오
- '/home/guest/' 디렉터리의 'services', 'dnsmasq.conf', 'ld.so.cache'를 하나의 파일로 묶어야한다.
- tar를 사용하여 생성된 파일은 .tar 확장명이 확인되어야한다.
- 파일명은 오늘 날짜 (년월일)

```
[root@Server-A guest]# tar -cvf  20260710.tar  services  dnsmasq.conf  ld.so.cache
services
dnsmasq.conf
ld.so.cache

[root@Server-A guest]# ls  -lSh
합계 2.4M
-rw-r--r-- 1 root root 780K  7월 10 10:33 GUEST
-rw-r--r-- 1 root root 750K  7월 10 10:37 20260710.tar	# tar 파일을 확인할 수 있다.
-rw-r--r-- 1 root root 677K  7월 10 09:57 services
-rw-r--r-- 1 root root  66K  7월 10 09:57 mime.types
-rw-r--r-- 1 root root  43K  7월 10 09:57 ld.so.cache
-rw-r--r-- 1 root root  29K  7월 10 09:57 brltty.conf
-rw-r--r-- 1 root root  28K  7월 10 09:57 dnsmasq.conf
-rw-r--r-- 1 root root  11K  7월 10 09:57 nanorc
-rw-r--r-- 1 root root 8.9K  7월 10 09:57 kdump.conf
-rw-r--r-- 1 root root 7.6K  7월 10 09:57 login.defs
-rw-r--r-- 1 root root 6.5K  7월 10 09:57 protocols
-rw-r--r-- 1 root root 6.2K  7월 10 09:57 pnm2ppa.conf
-rw-r--r-- 1 root root 5.7K  7월 10 09:57 idmapd.conf
-rw-r--r-- 1 root root 5.2K  7월 10 09:57 man_db.conf
```

### EX1-3) '20260710.tar' 파일을 bzip2를 사용하여 압축해야한다.

```
[root@Server-A guest]# bzip2  20260710.tar

[root@Server-A guest]# ls  -lSh
합계 1.9M
-rw-r--r-- 1 root root 780K  7월 10 10:33 GUEST
-rw-r--r-- 1 root root 677K  7월 10 09:57 services
-rw-r--r-- 1 root root 149K  7월 10 10:37 20260710.tar.bz2		# 20260710.tar 파일을 bzip2로 압축
-rw-r--r-- 1 root root  66K  7월 10 09:57 mime.types
-rw-r--r-- 1 root root  43K  7월 10 09:57 ld.so.cache
-rw-r--r-- 1 root root  29K  7월 10 09:57 brltty.conf
-rw-r--r-- 1 root root  28K  7월 10 09:57 dnsmasq.conf
-rw-r--r-- 1 root root  11K  7월 10 09:57 nanorc
-rw-r--r-- 1 root root 8.9K  7월 10 09:57 kdump.conf
-rw-r--r-- 1 root root 7.6K  7월 10 09:57 login.defs
-rw-r--r-- 1 root root 6.5K  7월 10 09:57 protocols
-rw-r--r-- 1 root root 6.2K  7월 10 09:57 pnm2ppa.conf
-rw-r--r-- 1 root root 5.7K  7월 10 09:57 idmapd.conf
-rw-r--r-- 1 root root 5.2K  7월 10 09:57 man_db.conf
```

### EX1-4) 아래의 조건에 맞게 설정하시오
- 'guest' 계정의 홈디렉터리 안에 'temp' 디렉터리를 생성해야한다.
- '20260710.tar.bz2' 파일을 temp 디렉터리로 이동해야한다.
- temp는 모든 사용자가 사용할수 있도록 설정 (단 다른 사용자는 파일 및 디렉터를 삭제할 수 없어야 한다.)
- temp 디렉터리안에서 '20260710.tar.bz2' 파일의 압축을 풀어야한다.
- temp 디렉터리안에서 '20260710.tar'를 사용하여 묶은 각각의 파일을 낱개의 파일로 복원해야한다.

```
[root@Server-A guest]# mkdir  temp

[root@Server-A guest]# chmod  777  temp
[root@Server-A guest]# chmod  +t  temp

[root@Server-A guest]# ls  -l  | grep temp
drwxrwxrwt 2 root root      6  7월 10 10:49 temp

[root@Server-A guest]# mv  20260710.tar.bz2  ./temp/		# 20260710.tar.bz2 파일을 temp 디렉터리로 이동

[root@Server-A guest]# ls  -l  ./temp/
합계 152
-rw-r--r-- 1 root root 152217  7월 10 10:37 20260710.tar.bz2

[root@Server-A guest]# bunzip2  ./temp/20260710.tar.bz2		# bzip2 압축 해제

[root@Server-A guest]# ls  -l  ./temp/
합계 752
-rw-r--r-- 1 root root 768000  7월 10 10:37 20260710.tar		# tar 파일 확인

	# 방법 1
[root@Server-A guest]# cd ./temp/

[root@Server-A temp]# tar  -xvf  20260710.tar
services
dnsmasq.conf
ld.so.cache

[root@Server-A temp]# ls  -l
합계 1504
-rw-r--r-- 1 root root 768000  7월 10 10:37 20260710.tar
-rw-r--r-- 1 root root  27839  7월 10 09:57 dnsmasq.conf
-rw-r--r-- 1 root root  43431  7월 10 09:57 ld.so.cache
-rw-r--r-- 1 root root 692252  7월 10 09:57 services

	# 낱개의 파일 삭제 
[root@Server-A guest]# rm -rf ./temp/dnsmasq.conf
[root@Server-A guest]# rm -rf ./temp/ld.so.cache
[root@Server-A guest]# rm -rf ./temp/services

	# 방법 2

[root@Server-A guest]# tar  -xvf  ./temp/20260710.tar  -C  ./temp/
services
dnsmasq.conf
ld.so.cache

[root@Server-A guest]# ls  -l  ./temp/
합계 1504
-rw-r--r-- 1 root root 768000  7월 10 10:37 20260710.tar
-rw-r--r-- 1 root root  27839  7월 10 09:57 dnsmasq.conf
-rw-r--r-- 1 root root  43431  7월 10 09:57 ld.so.cache
-rw-r--r-- 1 root root 692252  7월 10 09:57 services
```

**정리**: **tar -cvf**로 여러 파일을 하나의 아카이브로 묶고, **tar -xvf**(`-C`로 대상 경로 지정 가능)로 원본 낱개 파일들로 복원하며, tar 자체는 압축이 아니므로 원본 파일이 삭제되지 않는다.

---

### EX2-1) '/home/guest' 디렉터리에 'services', 'mime.types', 'dnsmasq.conf', 'nanorc' 파일을 tar를 사용하여 하나의 파일로 묶어야 한다.

```
[root@Server-A guest]# tar  -cvf  SOLDESK.tar  services  mime.types  dnsmasq.conf  nanorc
services
mime.types
dnsmasq.conf
nanorc

[root@Server-A guest]# ls  -l
합계 2488
-rw-r--r-- 1 root root 798720  7월 10 10:33 GUEST
-rw-r--r-- 1 root root 808960  7월 10 11:11 SOLDESK.tar		# tar 파일 확인
-rw-r--r-- 1 root root  28974  7월 10 09:57 brltty.conf
-rw-r--r-- 1 root root  27839  7월 10 09:57 dnsmasq.conf
-rw-r--r-- 1 root root   5799  7월 10 09:57 idmapd.conf
-rw-r--r-- 1 root root   9077  7월 10 09:57 kdump.conf
-rw-r--r-- 1 root root  43431  7월 10 09:57 ld.so.cache
-rw-r--r-- 1 root root   7779  7월 10 09:57 login.defs
-rw-r--r-- 1 root root   5235  7월 10 09:57 man_db.conf
-rw-r--r-- 1 root root  67454  7월 10 09:57 mime.types
-rw-r--r-- 1 root root  10373  7월 10 09:57 nanorc
-rw-r--r-- 1 root root   6300  7월 10 09:57 pnm2ppa.conf
-rw-r--r-- 1 root root   6568  7월 10 09:57 protocols
-rw-r--r-- 1 root root 692252  7월 10 09:57 services
drwxrwxrwt 2 root root     81  7월 10 11:01 temp
```

### EX2-2) tar을 사용하여 묶은 파일을 gzip을 사용하여 압축해야한다

```
[root@Server-A guest]# bzip2  SOLDESK.tar

[root@Server-A guest]# ls  -lSh
합계 1.9M
-rw-r--r-- 1 root root 780K  7월 10 10:33 GUEST
-rw-r--r-- 1 root root 677K  7월 10 09:57 services
-rw-r--r-- 1 root root 160K  7월 10 11:11 SOLDESK.tar.bz2	# tar파일을 압축
-rw-r--r-- 1 root root  66K  7월 10 09:57 mime.types
-rw-r--r-- 1 root root  43K  7월 10 09:57 ld.so.cache
-rw-r--r-- 1 root root  29K  7월 10 09:57 brltty.conf
-rw-r--r-- 1 root root  28K  7월 10 09:57 dnsmasq.conf
-rw-r--r-- 1 root root  11K  7월 10 09:57 nanorc
-rw-r--r-- 1 root root 8.9K  7월 10 09:57 kdump.conf
-rw-r--r-- 1 root root 7.6K  7월 10 09:57 login.defs
-rw-r--r-- 1 root root 6.5K  7월 10 09:57 protocols
-rw-r--r-- 1 root root 6.2K  7월 10 09:57 pnm2ppa.conf
-rw-r--r-- 1 root root 5.7K  7월 10 09:57 idmapd.conf
-rw-r--r-- 1 root root 5.2K  7월 10 09:57 man_db.conf
drwxrwxrwt 2 root root   81  7월 10 11:01 temp
```

### EX2-3) 아래의 조건에 맞게 설정하시오
- '/home/guest'의 홈디렉터리 안에 'imsi' 디렉터리를 생성해야한다.
- gzip을 사용하여 압축한 파일을 imsi 디렉터리로 이동해야한다.
- imsi 디렉터리안에서 gzip을 사용하여 압축한 파일을 풀어야한다.
- imsi 디렉터리안에서 tar을 사용하여 묶은 각각의 파일을 낱개의 파일로 복원해야한다

```
[root@Server-A guest]# mkdir  imsi

[root@Server-A guest]# mv  SOLDESK.tar.bz2  ./imsi/

[root@Server-A guest]# ls  -l  ./imsi/
합계 160
-rw-r--r-- 1 root root 163790  7월 10 11:11 SOLDESK.tar.bz2

[root@Server-A guest]# bzip2 -d  ./imsi/SOLDESK.tar.bz2		# 압축 해제

[root@Server-A guest]# ls  -l  ./imsi/
합계 792
-rw-r--r-- 1 root root 808960  7월 10 11:11 SOLDESK.tar		# tar 파일 확인

[root@Server-A guest]# tar  -xvf  ./imsi/SOLDESK.tar -C  ./imsi/
services
mime.types
dnsmasq.conf
nanorc

[root@Server-A guest]# ls  -l  ./imsi/
합계 1580
-rw-r--r-- 1 root root 808960  7월 10 11:11 SOLDESK.tar
-rw-r--r-- 1 root root  27839  7월 10 09:57 dnsmasq.conf
-rw-r--r-- 1 root root  67454  7월 10 09:57 mime.types
-rw-r--r-- 1 root root  10373  7월 10 09:57 nanorc
-rw-r--r-- 1 root root 692252  7월 10 09:57 services
```

**정리**: `tar -cvf`로 묶은 뒤 `bzip2`로 별도 압축하면 `.tar.bz2` 확장자가 되어 tar 파일임을 명확히 알 수 있고, `bzip2 -d`로 압축만 풀고 다시 `tar -xvf -C`로 원하는 위치에 낱개 파일을 복원하는 2단계 절차가 표준적인 흐름이다.

---

### EX3-1) 아래의 조건에 맞게 설정하시오
- 'services', 'mime.types', 'ld.so.cache', 'brltty.conf', 'dnsmasq.conf' 파일을 tar를 사용하여 하나의 파일로 묶어야 한다.
- tar을 사용하여 묶은 파일을 bzip2를 사용하여 압축해야한다.

```
   # tar 파일을 bzip2로 압축
[root@Server-A guest]# bzip2  SOL_AWS.tar

[root@Server-A guest]# ls  -lSh
합계 1.9M
-rw-r--r-- 1 root root 780K  7월 10 10:33 GUEST
-rw-r--r-- 1 root root 677K  7월 10 09:57 services
-rw-r--r-- 1 root root 158K  7월 10 11:36 SOL_AWS.tar.bz2
-rw-r--r-- 1 root root  66K  7월 10 09:57 mime.types
-rw-r--r-- 1 root root  43K  7월 10 09:57 ld.so.cache
-rw-r--r-- 1 root root  29K  7월 10 09:57 brltty.conf
-rw-r--r-- 1 root root  28K  7월 10 09:57 dnsmasq.conf
-rw-r--r-- 1 root root  11K  7월 10 09:57 nanorc
-rw-r--r-- 1 root root 8.9K  7월 10 09:57 kdump.conf
-rw-r--r-- 1 root root 7.6K  7월 10 09:57 login.defs
-rw-r--r-- 1 root root 6.5K  7월 10 09:57 protocols
-rw-r--r-- 1 root root 6.2K  7월 10 09:57 pnm2ppa.conf
-rw-r--r-- 1 root root 5.7K  7월 10 09:57 idmapd.conf
-rw-r--r-- 1 root root 5.2K  7월 10 09:57 man_db.conf
drwxr-xr-x 2 root root   93  7월 10 11:17 imsi
drwxrwxrwt 2 root root   81  7월 10 11:01 temp
```

### EX3-2) 아래의 조건에 맞게 설정하시오
- 'services', 'mime.types', 'ld.so.cache', 'brltty.conf', 'dnsmasq.conf' 파일을 tar를 사용하여 하나의 파일로 묶어야 한다.
- tar을 사용하여 묶은 파일을 bzip2를 사용하여 압축해야한다.

```
[root@Server-A guest]# tar  -cjvf  AWS_SOL.tar  services  ld.so.cache  brltty.conf  dnsmasq.conf
services
ld.so.cache
brltty.conf
dnsmasq.conf

[root@Server-A guest]# ls  -lSh
합계 2.0M
-rw-r--r-- 1 root root 780K  7월 10 10:33 GUEST
-rw-r--r-- 1 root root 677K  7월 10 09:57 services
-rw-r--r-- 1 root root 158K  7월 10 11:38 AWS_SOL.tar		# tar로 압축 후 bzip2로 압축 (tar파일인지 압축파일인지확인할 수 없다.)
-rw-r--r-- 1 root root 158K  7월 10 11:36 SOL_AWS.tar.bz2
-rw-r--r-- 1 root root  66K  7월 10 09:57 mime.types
-rw-r--r-- 1 root root  43K  7월 10 09:57 ld.so.cache
-rw-r--r-- 1 root root  29K  7월 10 09:57 brltty.conf
-rw-r--r-- 1 root root  28K  7월 10 09:57 dnsmasq.conf
-rw-r--r-- 1 root root  11K  7월 10 09:57 nanorc
-rw-r--r-- 1 root root 8.9K  7월 10 09:57 kdump.conf
-rw-r--r-- 1 root root 7.6K  7월 10 09:57 login.defs
-rw-r--r-- 1 root root 6.5K  7월 10 09:57 protocols
-rw-r--r-- 1 root root 6.2K  7월 10 09:57 pnm2ppa.conf
-rw-r--r-- 1 root root 5.7K  7월 10 09:57 idmapd.conf
-rw-r--r-- 1 root root 5.2K  7월 10 09:57 man_db.conf
drwxr-xr-x 2 root root   93  7월 10 11:17 imsi
drwxrwxrwt 2 root root   81  7월 10 11:01 temp
```

### EX3-3) 아래의 조건에 맞게 설정하시오
- 'services', 'mime.types', 'ld.so.cache', 'brltty.conf', 'dnsmasq.conf' 파일을 tar를 사용하여 하나의 파일로 묶어야 한다.
- tar을 사용하여 묶은 파일을 bzip2를 사용하여 압축해야한다.

```
[root@Server-A guest]# tar  -cjvf  SOL_LINUX.tar.bz2  services  ld.so.cache  brltty.conf  dnsmasq.conf
services
ld.so.cache
brltty.conf
dnsmasq.conf

[root@Server-A guest]# ls  -lSh
합계 2.2M
-rw-r--r-- 1 root root 780K  7월 10 10:33 GUEST
-rw-r--r-- 1 root root 677K  7월 10 09:57 services
-rw-r--r-- 1 root root 158K  7월 10 11:38 AWS_SOL.tar
-rw-r--r-- 1 root root 158K  7월 10 11:36 SOL_AWS.tar.bz2
-rw-r--r-- 1 root root 158K  7월 10 11:40 SOL_LINUX.tar.bz2	# tar 파일을 bz2로 압축한것으로 확인된다.
-rw-r--r-- 1 root root  66K  7월 10 09:57 mime.types
-rw-r--r-- 1 root root  43K  7월 10 09:57 ld.so.cache
-rw-r--r-- 1 root root  29K  7월 10 09:57 brltty.conf
-rw-r--r-- 1 root root  28K  7월 10 09:57 dnsmasq.conf
-rw-r--r-- 1 root root  11K  7월 10 09:57 nanorc
-rw-r--r-- 1 root root 8.9K  7월 10 09:57 kdump.conf
-rw-r--r-- 1 root root 7.6K  7월 10 09:57 login.defs
-rw-r--r-- 1 root root 6.5K  7월 10 09:57 protocols
-rw-r--r-- 1 root root 6.2K  7월 10 09:57 pnm2ppa.conf
-rw-r--r-- 1 root root 5.7K  7월 10 09:57 idmapd.conf
-rw-r--r-- 1 root root 5.2K  7월 10 09:57 man_db.conf
drwxr-xr-x 2 root root   93  7월 10 11:17 imsi
drwxrwxrwt 2 root root   81  7월 10 11:01 temp
```

### EX3-4) 아래의 조건에 맞게 설정하시오
- guest의 홈디렉터리 안에 '/ABC' 디렉터리를 생성해야한다.
- gzip을 사용하여 압축한 파일을 '/ABC' 디렉터리로 이동해야한다.
- '/ABC' 디렉터리안에서 bzip2를 사용하여 압축한 파일을 풀어야한다.
- '/ABC' 디렉터리안에서 tar을 사용하여 묶은 각각의 파일을 낱개의 파일로 복원해야한다.

```
[root@Server-A guest]# mkdir  ./ABC

[root@Server-A guest]# mv  SOL_LINUX.tar.bz2  ./ABC

[root@Server-A guest]# ls  -l ./ABC
합계 160
-rw-r--r-- 1 root root 160972  7월 10 11:50 SOL_LINUX.tar.bz2

[root@Server-A guest]# cd ./ABC/

[root@Server-A ABC]# tar  -xjvf  SOL_LINUX.tar.bz2
services
ld.so.cache
brltty.conf
dnsmasq.conf
```

**정리**: `tar -cjvf`는 아카이빙(`-c`)과 bzip2 압축(`-j`)을 한 번에 수행하며, 결과 확장자를 `.tar`가 아닌 `.tar.bz2`로 지정하면 파일 형태를 이름만으로도 명확히 구분할 수 있다.

---

### EX4) /home/guest/ 디렉터리안의 tar 및 압축파일을 모두 삭제

```
[root@Server-A guest]# rm -rf  ABC
[root@Server-A guest]# rm -rf  AWS_SOL.tar
[root@Server-A guest]# rm -rf  GUEST
[root@Server-A guest]# rm -rf  SOL_AWS.tar.bz2
[root@Server-A guest]# rm -rf  imsi/
[root@Server-A guest]# rm -rf  temp/
```

### EX4-2) /home/guest/ 디렉터리안의 모든 파일 및 디렉터리를 bzip2로 압축하시오

### EX4-3) 압축한 파일을 root의 홈 디렉터리에서 낱개의 파일로 확인되어야 한다.

**정리**: 실습 종료 단계에서는 생성했던 tar/압축 파일들을 정리(`rm -rf`)하고, 디렉터리 전체를 압축·복원하는 마무리 문제로 gzip·bzip2·xz·tar의 전체 워크플로를 복습한다.

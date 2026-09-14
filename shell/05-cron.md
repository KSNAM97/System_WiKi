# Shell Script Cron (크론 스케줄러)

## CRON

**CRON**은 Linux에서 특정 명령어나 스크립트를 지정한 시간 또는 일정한 간격에 맞춰 자동으로 실행하는 스케줄러이다. 매일 자정 로그를 백업하거나 매분 서비스 상태를 점검해 자동으로 재시작하는 등, 반복적인 운영 작업을 사람이 직접 실행하지 않고 예약해 둘 때 사용한다.

관리자가 명령어를 직접 실행하지 않아도 다음과 같은 작업을 자동으로 수행할 수 있다.

- 매일 새벽에 데이터 백업
- 일정 시간마다 서비스 상태 확인
- 오래된 로그 파일 자동 삭제
- 매주 또는 매월 정기 작업 실행
- 서버의 디스크 사용량 주기적 확인

즉, **CRON**은 Linux의 시간 기반 자동화 기능이다.

**정리**: CRON은 반복 작업을 사람이 직접 실행하지 않아도 지정한 시각에 자동으로 수행해주는 Linux의 시간 기반 스케줄러이다.

## CRON의 구성

**CRON**은 다음 두 가지 요소로 동작한다.

### 1) crond

**crond**는 CRON 작업을 실제로 실행하는 데몬이다.

백그라운드에서 계속 실행되면서 설정된 스케줄을 확인하고, 지정된 시간이 되면 명령어나 스크립트를 자동으로 실행한다.

```
[root@Server-A ~]# systemctl status crond
[root@Server-A ~]# systemctl start crond
[root@Server-A ~]# systemctl enable crond
```

- crond가 중지되어 있으면 등록된 CRON 작업은 실행되지 않는다.
- enable을 설정하면 시스템을 재부팅한 후에도 crond가 자동으로 실행된다.

### 2) crontab

**crontab**은 언제, 어떤 명령어, 스크립트를 실행할 것인지 기록하는 설정 파일이다.

- 사용자별 crontab과 시스템 전체에서 사용하는 `/etc/crontab`이 있다.

```
[root@Server-A ~]# crontab -e	# 현재 사용자의 CRON 작업을 작성하거나 수정한다.

[root@Server-A ~]# crontab -l	# 현재 사용자의 CRON 작업을 확인한다.

[root@Server-A ~]# crontab -r	# 현재 사용자의 CRON 작업을 모두 삭제한다.
```

**정리**: CRON은 실제 실행을 담당하는 **crond** 데몬과, 실행 일정/명령을 기록하는 **crontab** 설정 파일이라는 두 요소로 구성된다.

## CRON 스케줄 형식

```
분  시  일  월  요일  실행할_명령
```

**cron** 스케줄은 다음 6개 필드로 이루어진다.

| 필드 | 의미 | 값 범위 |
|------|------|---------|
| 분 | 분 | 0~59 |
| 시 | 시 | 0~23 |
| 일 | 일 | 1~31 |
| 월 | 월 | 1~12 |
| 요일 | 요일 | 0~7 (0과 7은 일요일) |

```
┌─────────────	분	0~59
│ ┌─────────── 	시          	0~23
│ │ ┌───────── 	일          	1~31
│ │ │ ┌─────── 	월          	1~12
│ │ │ │ ┌───── 	요일        	0~7 	(0과 7은 일요일)
│ │ │ │ │
 *  *  *   *  *
```

**형식 예시**
```
분  시  일  월  요일  실행할_명령
```

**EX1)**
```
0 3 * * * /usr/local/bin/backup.sh		# 매일 새벽 03시 00분에 /usr/local/bin/backup.sh 스크립트를 실행한다.
```

**EX2)**
```
5 * * * *    /usr/bin/systemctl status sshd	# 매시간 5분에 sshd 서비스 상태를 확인한다.
*/5 * * * * /usr/bin/systemctl status sshd	# 매시간 0분부터 5분 간격으로 sshd 서비스 상태를 확인한다.
```

**EX3)**
```
0 1 * * 1	/home/user/clean.sh			# 매주 월요일 새벽 01시 00분에 /home/user/clean.sh 스크립트를 실행한다.
```

**EX4)**
```
30 18 1 * *    echo "월 초 첫날 18:30 실행"	# 매월 1일 오후 18시 30분에 지정된 메시지를 출력한다.
```

**EX5)**
```
0 */2 * * * /usr/local/bin/check_disk.sh		# 매일 0시부터 2시간 간격으로 /usr/local/bin/check_disk.sh 스크립트를 실행한
```

**정리**: cron 스케줄은 분/시/일/월/요일 5개 필드와 명령으로 구성되며, `*`, `*/N`, 특정 숫자 등을 조합해 다양한 반복 주기를 표현할 수 있다.

## 주의 사항

- 경로를 반드시 절대경로로 적는다. 예: echo, ls 같은 명령도 실제 위치를 사용한다.
  ```
  /bin/echo
  /usr/bin/ls
  ```
- 스크립트 실행 시 전체 경로를 작성한다.
  ```
  /home/user/scripts/disk.sh
  ```
- 스크립트는 실행 권한이 있어야 한다.
  ```
  chmod +x disk.sh
  ```
- 표준 출력, 에러 출력은 파일에 저장하는 것이 좋다. (문제 발생 시 로그 확인 가능)
  ```
  0 3 * * * /home/user/backup.sh > /var/log/backup.log 2>&1
  ```
- 환경 변수(PATH)가 제한적이다. crontab은 일반 쉘의 PATH와 다르므로 명령을 찾지 못하는 경우가 많다.
- 자주 사용하는 예약어, 특정 패턴은 예약어로 간단히 표시 가능하다.

| 예약어 | 의미 |
|--------|------|
| `@reboot` | 부팅 후 한 번 실행 |
| `@hourly` | 매 정각 실행 |
| `@daily` | 매일 1번 실행 |
| `@weekly` | 매주 실행 |
| `@monthly` | 매월 실행 |

예:
```
@reboot root /script/startup.sh	# 시스템이 부팅된 후 /script/startup.sh를 한 번 실행한다.
```

평소에 터미널에서 스크립트를 실행하면 화면에 내용이 보인다. 하지만 CRON은 백그라운드에서 실행되기 때문에 화면이 없고, 출력 내용을 보기도 어렵다. 따라서 명령 출력(stdout)과 에러 출력(stderr)을 반드시 파일로 저장하는 습관이 필요하다.

리눅스 명령 실행 시, 출력은 두 종류로 나뉜다.
- 표준 출력 (stdout) : 번호 1
- 표준 에러 (stderr) : 번호 2

### 1) stdout, stderr 각각 저장하기

```
# /home/student/cron_lab/test.sh 실행시 출력은 out.log여기에 저장 에러는 err.log에 저장

/home/student/cron_lab/test.sh  >  /home/student/cron_lab/out.log  2>  /home/student/cron_lab/err.log
```

### 2) stdout + stderr 를 하나의 파일로 저장하기 (가장 많이 사용)

```
# /home/student/cron_lab/test.sh 실행시 출력, 에러를 ecron_master.log에 저장 
/home/student/cron_lab/test1.sh  >>  /home/student/cron_lab/cron_master.log 2>&1
```

**정리**: cron 작업은 절대경로 사용, 실행 권한 부여, 표준출력/에러의 파일 저장이 기본 원칙이며, `>>` 와 `2>&1` 조합으로 stdout/stderr를 한 로그 파일에 누적하는 방식이 가장 널리 쓰인다.

## crontab 시스템 파일 확인

```
[root@Server-A ~]# rpm  -qa | grep chrony		# NTP 시간 동기화 프로그램
chrony-4.8-1.el9.x86_64


[root@Server-A ~]# dnf  install  -y  chrony
```

```
[root@Server-A ~]# ls  -l  /etc/ | grep cron*
-rw-r--r--   1 root root       541 12월 30  2025 anacrontab
drwxr-xr-x.  2 root root        21  7월  2 15:06 cron.d
drwxr-xr-x.  2 root root         6   5월 11  2022 cron.daily		# 일마다
-rw-r--r--.  1 root root         0  12월 30  2025 cron.deny
drwxr-xr-x.  2 root root        22   7월  2 15:06 cron.hourly		# 시간마다
drwxr-xr-x.  2 root root         6   5월 11  2022 cron.monthly		# 월마다
drwxr-xr-x.  2 root root         6   5월 11  2022 cron.weekly		# 주마다
-rw-r--r--.  1 root root       451   5월 11  2022 crontab		# 수동 작업
```

```
[root@Server-A ~]# vi  /etc/crontab

SHELL=/bin/bash
PATH=/sbin:/bin:/usr/sbin:/usr/bin
MAILTO=root

# For details see man 4 crontabs

# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
# *  *  *  *  * user-name  command to be executed

30 *  *  *  *  *	: 매시간 30분마다 실행

30 3  *  *  *  *	: 매일 03시 30분마다 실행

15 3  *  *  *  1	: 매주 월요일 03시 15분마다 실행

30 3  1  *  *  *	: 매월 1일 03시 30분마다 실행

30 3  1  1  *  *	: 매년 1월 1일 03시 30분마다 실행


*/5 *  *  *  *  *	: 매 5분마다 실행 (00분, 05분, 10분, 15분....)
*/10 *  *  *  *  *	: 매 10분마다 실행 (00분, 10분, 20분, 30분....)
```

**정리**: `/etc/` 아래의 cron.d, cron.daily/hourly/monthly/weekly 디렉터리와 `/etc/crontab` 파일 구조를 확인하고, 실제 스케줄 문법이 어떻게 적용되는지 예시로 살펴보았다.

## 실습 EX1 접속 사용자 정보 점검

EX1) 서버에 현재 접속한 사용자 정보를 주기적으로 확인할 수 있도록 다음 조건에 맞게 설정하시오
- 현재 접속한 사용자 정보를 확인 (실행 날짜와 시간도 함께 출력)
- 결과는 /var/log/login_user_check.log에 누적 저장해야 한다.
- /etc/crontab에 등록하여 5분마다 실행되도록 설정해야 한다.
- 스크립트를 수동으로 실행한 후 로그 파일을 확인해야 한다.

```
[root@Server-A ~]# mkdir -p /script/hourly


[root@Server-A ~]# ls  -lR /script
/script:
합계 0
drwxr-xr-x 2 root root 6  7월 30 10:03 hourly

/script/hourly:
합계 0
```

```
[root@Server-A ~]# vi  /script/hourly/login_user_check.sh
#!/bin/bash

LOG="/var/log/login_user_check.log"

echo "=================================================="  >> "$LOG"
echo "확인시간 : $(date '+%F %T')"  >> "$LOG"
who >> "$LOG"

:wq
```

```
[root@Server-A ~]# chmod  +x  /script/hourly/login_user_check.sh


[root@Server-A ~]# /script/hourly/login_user_check.sh


[root@Server-A ~]# ls  -l  /var/log/login_user_check.log
-rw-r--r-- 1 root root 143  7월 30 10:13 /var/log/login_user_check.log


[root@Server-A ~]# cat  /var/log/login_user_check.log
==================================================
확인시간 : 2026-07-30 10:16:13
root     pts/0        2026-07-30 09:27 (192.168.10.1)
root     pts/1        2026-07-30 09:27 (192.168.10.1)
```

```
[root@Server-A ~]# vi /etc/crontab
SHELL=/bin/bash
PATH=/sbin:/bin:/usr/sbin:/usr/bin
MAILTO=root

# For details see man 4 crontabs

# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
# *  *  *  *  * user-name  command to be executed

*/5 * * * *  root  /script/hourly/login_user_check.sh	# 추가 설정
```

```
[root@Server-A ~]# tail  -1 /etc/crontab
*/5 * * * *  root  /script/hourly/login_user_check.sh	# 설정 확인
```

```
[root@Server-A ~]# cat  /var/log/login_user_check.log
==================================================
확인시간 : 2026-07-30 10:16:51
root     pts/0        2026-07-30 09:27 (192.168.10.1)
root     pts/1        2026-07-30 09:27 (192.168.10.1)
==================================================
확인시간 : 2026-07-30 10:20:01			# 로그 추가 시간 확인 : 10시 20분 01초
root     pts/0        2026-07-30 09:27 (192.168.10.1)		# 로그 추가 확인
root     pts/1        2026-07-30 09:27 (192.168.10.1)		# 로그 추가 확인
```

**정리**: 접속 사용자 확인 스크립트를 작성해 실행 권한을 부여하고 `/etc/crontab`에 5분 주기로 등록하여, cron이 실제로 로그를 누적 기록하는지 확인한 실습이다.

## 실습 EX2 평일 업무 종료 후 자동 백업

EX2) 회사 서버의 주요 설정 파일이 저장된 /temp 디렉터리를 평일 업무 종료 후 자동으로 백업하도록 설정하시오
- 백업 스크립트는 /script/hourly/temp_backup.sh로 작성하시오
- /etc/a*의 모든 파일 및 디렉터리를 /temp 디렉터리로 복사 (원본)
- 백업 파일은 /backup/etc 디렉터리에 저장하시오
- 백업 디렉터리가 없으면 자동으로 생성하시오
- 파일명은 temp_YYYY-MM-DD.tar.gz 형식으로 생성하시오
- 백업 결과는 /var/log/etc_backup.log에 누적 기록하시오
- 매주 월요일부터 금요일까지 오후 18시 40분에 실행되도록 설정하시오 (실습에 적용가능한 시간으로 설정)

```
[root@Server-A ~]# cp -r /etc/a*  /temp

[root@Server-A ~]# vi  /script/hourly/temp_backup.sh
#!/bin/bash

SRC="/temp"
DEST="/backup/etc/"
LOG="/var/log/temp_backup.log"
DATE=$(date +%F)
BACKUP_FILE="${DEST}temp_${DATE}.tar.gz"

if [ ! -d "$DEST" ]
then
    mkdir  -p  "$DEST"
fi

tar  czf  "$BACKUP_FILE"  -C  "$SRC"  .
# c 	: 새로운 압축 파일을 생성합니다. (create)
# z 	: gzip 방식으로 압축합니다.
# f 	: 생성할 압축 파일 이름을 지정합니다.
# -C	: 압축하기 전에 작업 기준 디렉터리를  변경 (C = change director)

if [ $? -eq 0 ]
then
    echo "$(date '+%f %T') - 백업 성공 : $BACKUP_FILE" >> "$LOG"
else
    echo "$(date '+%f %T') - 백업 실패" >> "$LOG"
fi

:wq
```

```
[root@Server-A ~]# chmod +x  /script/hourly/temp_backup.sh	# 실행권한


[root@Server-A ~]# /script/hourly/temp_backup.sh		# 스크립트 실행 (파일 백업 확인, 로그 기록 확인)

/script/hourly/temp_backup.sh
```

```
[root@Server-A ~]# cd /backup/etc/


[root@Server-A etc]# ls  -l
합계 20
-rw-r--r-- 1 root root 17620  7월 30 11:20 temp_2026-07-30.tar.gz	# 백업 파일이 확인된다.
```

```
[root@Server-A etc]# tar  tzf  temp_2026-07-30.tar.gz
./
./passwd
./soldABC
./soldpasswd
~~~~~~~ 중간 생략 ~~~~~~~
./avahi/services/
./avahi/avahi-daemon.conf
./avahi/hosts
./avahi/etc/

# x	: 압축파일 해제
# t	: 압축을 풀지 않고 내부 목록을 확인
# x	: gzip 방식의 압축
# f	: 파일명을 직접 설정
```

```
[root@Server-A etc]# vi  /etc/crontab
SHELL=/bin/bash
PATH=/sbin:/bin:/usr/sbin:/usr/bin
MAILTO=root

# For details see man 4 crontabs

# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
# *  *  *  *  * user-name  command to be executed

#*/5 * * * *  root  /script/hourly/login_user_check.sh	# 주석 처리
35 11 * * 1-5 root  /script/hourly/temp_backup.sh
```

```
[root@Server-A etc]# ls -l /var/log/temp_backup.log
-rw-r--r-- 1 root root 130  7월 30 11:35 /var/log/temp_backup.log	# 로그 정보가 확인된다.
```

```
[root@Server-A etc]# cat /var/log/temp_backup.log
%f 11:34:29 - 백업 성공 : /backup/etc/temp_2026-07-30.tar.gz	# 로그 정보가 확인된다. (스크립트 수동 실행)
```

```
[root@Server-A etc]# cat /var/log/temp_backup.log
%f 11:34:29 - 백업 성공 : /backup/etc/temp_2026-07-30.tar.gz	# 로그 정보가 확인된다. (스크립트 수동 백업)
%f 11:35:01 - 백업 성공 : /backup/etc/temp_2026-07-30.tar.gz	# 로그 정보가 확인된다. (cron에 의한 자동 백업)
```

**정리**: `tar`를 이용한 백업 스크립트를 작성하고 요일 범위(`1-5`) 필드를 사용해 평일에만 실행되도록 crontab에 등록, 수동/자동 실행 결과를 로그로 검증한 실습이다.

## 임시 파일 자동 삭제 테스트

```
[root@Server-A etc]# mkdir  /backup/log		# 디렉터리 생성

[root@Server-A etc]# cp -pr  /etc/a*  /backup/log/	# 기존의 속성을 유지해서 복사


[root@Server-A etc]# touch  /backup/log/logFile{1..5}	# logFile1 ~ logFile5 파일 생성

[root@Server-A etc]# ls  -l  /backup/log/
합계 36
drwxr-xr-x 3 root root    28  7월  2 12:52 accountsservice
-rw-r--r-- 1 root root    16  7월  2 12:55 adjtime
-rw-r--r-- 1 root root 1529  6월 23  2020 aliases
drwxr-xr-x 3 root root    65  7월  2 15:07 alsa
drwxr-xr-x 2 root root 4096  7월  2 15:06 alternatives
-rw-r--r-- 1 root root   541 12월 30  2025 anacrontab
-rw-r--r-- 1 root root   269 10월 30  2022 anthy-unicode.conf
-rw-r--r-- 1 root root   833  2월 11  2023 appstream.conf
-rw-r--r-- 1 root root    55  1월 25  2026 asound.conf
-rw-r--r-- 1 root root      1 11월 12  2025 at.deny
drwxr-x--- 4 root root  100  7월  2 15:06 audit
drwxr-xr-x 3 root root 4096  7월  2 15:08 authselect
drwxr-xr-x 4 root root    71  7월  2 15:06 avahi
-rw-r--r-- 1 root root     0  7월 30 11:54 logFile1
-rw-r--r-- 1 root root     0  7월 30 11:54 logFile2
-rw-r--r-- 1 root root     0  7월 30 11:54 logFile3
-rw-r--r-- 1 root root     0  7월 30 11:54 logFile4
-rw-r--r-- 1 root root     0  7월 30 11:54 logFile5
 
```

```
[root@Server-A etc]# find /  -type f  -name passwd	# 파일 이름이 passwd인 모든 경로상의 파일을 검색
/etc/pam.d/passwd
/etc/passwd
/usr/bin/passwd
/usr/share/bash-completion/completions/passwd
/temp/passwd
/soltest/test/passwd
```

```
[root@Server-A etc]# find  /backup/log  -type f			# /backup/log 경로의 type이 file인것만 검색


[root@Server-A etc]# find  /backup/log  -type f  -mtime +7		# /backup/log 경로의 생성된지 7일 이상인 파일만 검색
/backup/log/adjtime
/backup/log/aliases
/backup/log/alsa/alsactl.conf
/backup/log/alsa/state-daemon.conf
/backup/log/alsa/conf.d/99-pipewire-default.conf
/backup/log/alsa/conf.d/50-pipewire.conf
/backup/log/anacrontab
/backup/log/anthy-unicode.conf
/backup/log/appstream.conf
/backup/log/asound.conf
/backup/log/at.deny
/backup/log/audit/auditd.conf
/backup/log/audit/audit-stop.rules
/backup/log/audit/audit.rules
/backup/log/audit/plugins.d/sedispatch.conf
/backup/log/audit/rules.d/audit.rules
/backup/log/authselect/system-auth
/backup/log/authselect/password-auth
/backup/log/authselect/fingerprint-auth
/backup/log/authselect/smartcard-auth
/backup/log/authselect/postlogin
/backup/log/authselect/nsswitch.conf
/backup/log/authselect/dconf-db
/backup/log/authselect/dconf-locks
/backup/log/authselect/authselect.conf
/backup/log/authselect/user-nsswitch.conf
/backup/log/avahi/avahi-daemon.conf
/backup/log/avahi/hosts
```

```
[root@Server-A etc]# find  /backup/log  -type f  -mtime +7	 -delete
```

**정리**: `find ... -mtime +7 -delete`로 7일 이상 지난 파일만 골라 삭제하는 방법을 수동으로 먼저 검증한 사전 테스트이다.

## 실습 EX3 임시 파일 자동 삭제

EX3) 임시 파일 자동 삭제
- /backup/log 디렉터리에 로그 파일이 계속 쌓이고 있다. 다음 조건에 맞게 로그 백업 및 자동 정리 시스템을 구축하시오
- 백업 및 정리 스크립트는 /script/hourly/log_backup.sh로 작성하시오
- /backup/log 디렉터리 안의 모든 파일과 디렉터리를 tar.gz 형식으로 압축하시오
- 압축 백업 파일은 /temp/log 디렉터리에 저장하시오
- /temp/log 디렉터리가 없으면 자동으로 생성하시오
- 백업 파일명은 log_YYYY-MM-DD.tar.gz 형식으로 생성하시오
- 백업이 성공한 경우에만 /backup/log 아래의 일반 파일 중 수정된 지 7일을 초과한 파일을 삭제하시오
- 삭제 대상 파일은 삭제 전에 /var/log/log_cleanup.log에 기록하시오
- 백업 및 정리 결과도 /var/log/log_cleanup.log에 누적 기록하시오
- 매일 밤 23시 30분에 실행되도록 설정하시오  (현재 실습이 가능한 시간으로 설정)
- 실습 가능한 시간으로 변경하여 자동 실행 여부를 확인하시오
- 테스트용 파일을 생성하여 동작 여부를 확인하시오

```
[root@Server-A etc]# mkdir  /backup/log		# 디렉터리 생성


[root@Server-A etc]# touch  /backup/log/oldFile0.log
[root@Server-A etc]# touch -d "1 days ago" /backup/log/oldFile1.log		# 1일전 시간으로 파일 생성
[root@Server-A etc]# touch -d "2 days ago" /backup/log/oldFile2.log
[root@Server-A etc]# touch -d "3 days ago" /backup/log/oldFile3.log
[root@Server-A etc]# touch -d "4 days ago" /backup/log/oldFile4.log
[root@Server-A etc]# touch -d "5 days ago" /backup/log/oldFile5.log
[root@Server-A etc]# touch -d "6 days ago" /backup/log/oldFile6.log
[root@Server-A etc]# touch -d "7 days ago" /backup/log/oldFile7.log
[root@Server-A etc]# touch -d "8 days ago" /backup/log/oldFile8.log
[root@Server-A etc]# touch -d "9 days ago" /backup/log/oldFile9.log
[root@Server-A etc]# touch -d "10 days ago" /backup/log/oldFile10.log
[root@Server-A etc]# touch -d "11 days ago" /backup/log/oldFile11.log	# 11일전 시간으로 파일 생성
```

```
[root@Server-A ~]# ls  -l  /backup/log/
합계 0
-rw-r--r-- 1 root root 0  7월 30 12:07 oldFile0.log
-rw-r--r-- 1 root root 0  7월 29 12:06 oldFile1.log
-rw-r--r-- 1 root root 0  7월 20 12:06 oldFile10.log
-rw-r--r-- 1 root root 0  7월 19 12:06 oldFile11.log
-rw-r--r-- 1 root root 0  7월 28 12:06 oldFile2.log
-rw-r--r-- 1 root root 0  7월 27 12:06 oldFile3.log
-rw-r--r-- 1 root root 0  7월 26 12:06 oldFile4.log
-rw-r--r-- 1 root root 0  7월 25 12:06 oldFile5.log
-rw-r--r-- 1 root root 0  7월 24 12:06 oldFile6.log
-rw-r--r-- 1 root root 0  7월 23 12:06 oldFile7.log
-rw-r--r-- 1 root root 0  7월 22 12:06 oldFile8.log
-rw-r--r-- 1 root root 0  7월 21 12:06 oldFile9.log
[root@Server-A ~]#
```

```
[root@Server-A ~]# vi /script/hourly/log_backup.sh
#!/bin/bash

SRC="/backup/log/"
DEST="/temp/log/"
LOG="/var/log/log_cleanup.log"
DATE="$(date '+%F')"
BACKUP_FILE="${DEST}/log_${DATE}.tar.gz"

if [ ! -d "$DEST" ]
then
    mkdir -p  "$DEST"
fi

echo  "=================================================="  >> "$LOG"
echo  "$(date '+%F %T') - 로그 백업 시작"  >>  "$LOG"

tar  czf  "$BACKUP_FILE" -C "$SRC" .

if [ $? -eq 0 ]
then
    echo "$(date '+%F %T') - 백업 성공 : $BACKUP_FILE" >> "$LOG"
else
    echo "$(date '+%F %T') - 백업 실패" >> "$LOG"
fi

echo "$(date '+%F %T') - 장기 로그 파일 삭제 시작"  >>  "$LOG"

find  "$SRC"  -type  f  -mtime +7  -print  -delete  >>  "$LOG" 

echo "$(date '+%F %T') - 장기 로그 파일 삭제 완료"  >>  "$LOG"

:wq
```

```
[root@Server-A ~]# chmod  +x  /script/hourly/log_backup.sh


[root@Server-A ~]# vi /etc/crontab
SHELL=/bin/bash
PATH=/sbin:/bin:/usr/sbin:/usr/bin
MAILTO=root

# For details see man 4 crontabs

# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
# *  *  *  *  * user-name  command to be executed

#*/5 * * * *  root   /script/hourly/login_user_check.sh
#35 11 * * 1-5 root  /script/hourly/temp_backup.sh
7 13 * * *   root    /script/hourly/log_backup.sh
```

```
[root@Server-A ~]# ls  -l  /temp/log				# 목적지 디렉터리가 확인되지 않는다. (스크립트에서 없으면 자동 생성)
ls: cannot access '/temp/log': 그런 파일이나 디렉터리가 없습니다


[root@Server-A ~]# ls  -l /var/log/log_cleanup.log		# 로그 파일이 확인되지 않는다. (스크립트에서 없으면 자동 생성)
ls: cannot access '/var/log/log_cleanup.log': 그런 파일이나 디렉터리가 없습니다
```

```
[root@Server-A ~]# ls  -l  /temp/log				# cron에의해 스크립트가 자동으로 실행 (스크립트에 의해 디렉터리 생성)
합계 4
-rw-r--r-- 1 root root 304  7월 30 13:07 log_2026-07-30.tar.gz


[root@Server-A ~]# ls  -l /var/log/log_cleanup.log		# cron에의해 스크립트가 자동으로 실행 (스크립트에 의해 로그 파일 생성)
-rw-r--r-- 1 root root 279  7월 30 13:07 /var/log/log_cleanup.log
```

```
[root@Server-A ~]# cat /var/log/log_cleanup.log			# 로그 파일 확인
==================================================
2026-07-30 13:12:01 - 로그 백업 시작
2026-07-30 13:12:01 - 백업 성공 : /temp/log//log_2026-07-30.tar.gz
2026-07-30 13:12:01 - 장기 로그 파일 삭제 시작
/backup/log/oldFile8.log
/backup/log/oldFile9.log
/backup/log/oldFile10.log
/backup/log/oldFile11.log
2026-07-30 13:12:01 - 장기 로그 파일 삭제 완료
```

```
[root@Server-A ~]# ls  -l  /backup/log/
합계 0
-rw-r--r-- 1 root root 0  7월 30 12:07 oldFile0.log
-rw-r--r-- 1 root root 0  7월 29 12:06 oldFile1.log
-rw-r--r-- 1 root root 0  7월 28 12:06 oldFile2.log
-rw-r--r-- 1 root root 0  7월 27 12:06 oldFile3.log
-rw-r--r-- 1 root root 0  7월 26 12:06 oldFile4.log
-rw-r--r-- 1 root root 0  7월 25 12:06 oldFile5.log
-rw-r--r-- 1 root root 0  7월 24 12:06 oldFile6.log
-rw-r--r-- 1 root root 0  7월 23 12:06 oldFile7.log
```

**정리**: 백업 성공 시에만 조건부로 오래된 파일을 삭제하도록 `log_backup.sh`를 작성하고 crontab에 등록하여, 목적지 디렉터리·로그 파일이 스크립트에 의해 자동 생성되고 7일 초과 파일만 정확히 삭제됨을 확인한 실습이다.

## ANACRON

**ANACRON**은 Linux에서 일간, 주간, 월간 단위의 작업을 주기적으로 실행하는 스케줄러이다.

**CRON**은 지정된 정확한 시간에 작업을 실행하지만, 시스템이 꺼져 있으면 해당 작업을 실행하지 못한다.

**ANACRON**은 시스템이 꺼져 있어서 실행하지 못한 작업이 있으면, 시스템이 다시 켜진 후 누락된 작업을 실행한다.

즉, ANACRON은 다음과 같이 이해할 수 있다.
- CRON : 지정된 시간에 실행
- ANACRON : 실행하지 못한 주기 작업을 나중에 실행

### ANACRON이 필요한 이유

다음과 같은 CRON 설정이 있다고 가정한다.

```
0 3 * * * /script/backup.sh
```

매일 새벽 03시에 백업하도록 설정되어 있다.

그러나 새벽 03시에 서버가 꺼져 있으면 해당 백업 작업은 실행되지 않는다.

**ANACRON**을 사용하면 서버가 다시 부팅된 후 실행하지 못했던 일일 작업을 확인하여 실행할 수 있다.

예:
```
# 새벽 03시	: 서버가 꺼져 있어 백업 실패
# 오전 09시	: 서버 부팅
# 부팅 후		: ANACRON이 누락된 백업 작업 실행
```

### CRON과 ANACRON의 차이

**CRON**
- 정확한 시간과 분을 지정하여 실행한다.
- 분 단위 반복 작업이 가능하다.
- 시스템이 꺼져 있으면 작업을 놓칠 수 있다.
- 서버처럼 항상 켜져 있는 시스템에 적합하다.

**ANACRON**
- 정확한 시간보다 실행 주기를 기준으로 동작한다.
- 일, 주, 월 단위 작업에 적합하다.
- 시스템이 꺼져 있어서 놓친 작업을 나중에 실행한다.
- 노트북이나 개인용 PC처럼 자주 꺼지는 시스템에 적합하다.

```
[root@Server-A ~]# ls  -ld  /etc/cron*
drwxr-xr-x. 2 root root  21  7월  2 15:06 /etc/cron.d
drwxr-xr-x. 2 root root  24  7월 30 12:43 /etc/cron.daily
-rw-r--r--. 1 root root   0  12월 30  2025 /etc/cron.deny
drwxr-xr-x. 2 root root  22  7월  2 15:06 /etc/cron.hourly
drwxr-xr-x. 2 root root   6   5월 11  2022 /etc/cron.monthly
drwxr-xr-x. 2 root root   6   5월 11  2022 /etc/cron.weekly
-rw-r--r--  1 root root 610  7월 30 13:10 /etc/crontab
```

```
[root@Server-A ~]# cat /etc/crontab
SHELL=/bin/bash
PATH=/sbin:/bin:/usr/sbin:/usr/bin
MAILTO=root

# For details see man 4 crontabs

# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
# *  *  *  *  * user-name  command to be executed

 0 * * * *  root  run-parts /etc/cron.hourly	# 매 시간 정각에 /etc/cron.hourly 내부 작업 실행
 0 3 * * *  root  run-parts /etc/cron.daily	# 매일 새벽 03시 정각에 /etc/cron.daily 내부 작업 실행


 0 * * * *  root  cd / &&  run-parts  -report  /etc/cron.hourly
 0 3 * * *  root  cd / &&  run-parts  -report  /etc/cron.daily
```

**정리**: ANACRON은 시스템이 꺼져 있어 CRON이 놓친 일/주/월 단위 작업을 재부팅 후 대신 실행해주는 보완 스케줄러이며, CRON은 상시 가동 서버에, ANACRON은 자주 꺼지는 개인용 시스템에 적합하다.

## 실습 Anacron과 Cron 연동

EX) 다음 조건에 맞게 Anacron과 Cron을 연동하시오
- 서버의 /etc/cron.daily 작업을 매일 실행해야 한다.
- /etc/cron.daily에 테스트용 스크립트를 작성하시오
- 테스트 결과는 /var/log/daily_test.log에 기록하시오
- /usr/sbin/anacron에 실행 권한이 있으면 Anacron이 Daily 작업을 실행하도록 설정하시오
- /usr/sbin/anacron에 실행 권한이 없으면 Cron이 /etc/cron.daily를 직접 실행하도록 설정하시오
- Anacron과 Cron 중 어떤 방식으로 실행됐는지 로그에서 확인할 수 있어야 한다
- 실습 가능한 시간으로 Cron 실행 시간을 설정하시오
- 실습 후 Anacron 실행 권한을 원래대로 복원하시오

### Anacron 실행 및 권한 확인

```
[root@Server-A ~]# ls -l  /usr/sbin/anacron
-rw-r--r--. 1 root root 40464 12월 30  2025 /usr/sbin/anacron	# anacron 확인


[root@Server-A ~]# test -x  /usr/sbin/anacron


[root@Server-A ~]# echo $?
0
```

### Daily 테스트 스크립트 작성

```
[root@Server-A ~]# ls  -ld  /etc/cron*
drwxr-xr-x. 2 root root  21  7월  2 15:06 /etc/cron.d
drwxr-xr-x. 2 root root  24  7월 30 12:43 /etc/cron.daily
-rw-r--r--. 1 root root   0 12월 30  2025 /etc/cron.deny
drwxr-xr-x. 2 root root  22  7월  2 15:06 /etc/cron.hourly
drwxr-xr-x. 2 root root   6  5월 11  2022 /etc/cron.monthly
drwxr-xr-x. 2 root root   6  5월 11  2022 /etc/cron.weekly
-rw-r--r--  1 root root 610  7월 30 13:10 /etc/crontab
```

```
[root@Server-A ~]# vi  /etc/cron.daily/daily_test		# 스크립트 생성시 .sh는 생략한다.
#! /bin/bash

LOG="/var/log/daily_test.log"
echo "$(date '+%F %T') - 실행 주체 : ${RUN_BY:-UNKNOWN}"  >>  "$LOG"

:wq

# ${RUN_BY:-UNKNOWN}" : 환경변수 RUN_BY에 값이 있으면 해당 값을 출력하고 없으면 UNKNOWN을 출력
```

```
[root@Server-A ~]# chmod +x  /etc/cron.daily/daily_test
```

### run-parts 실행 대상 확인

- **run-parts** : 지정한 디렉터리 안의 실행 가능한 스크립트를 차례대로 실행하는 명령어
- **--test** : 실제 실행하지는 않고 실행할 파일만 출력하는 옵션. 파일명에 ".sh" 확장자를 사용하면 run-part 실행 대상에서 제외될 수 있으므로 확장자 없이 작성한다.

```
[root@Server-A ~]# run-parts --test  /etc/cron.daily/
/etc/cron.daily/daily_test
```

### Anacron 설정

```
[root@Server-A ~]# vi  /etc/anacrontab
# /etc/anacrontab: configuration file for anacron

# See anacron(8) and anacrontab(5) for details.

SHELL=/bin/sh
PATH=/sbin:/bin:/usr/sbin:/usr/bin
MAILTO=root
# the maximal random delay added to the base delay of the jobs
RANDOM_DELAY=45
# the jobs will be started during the following hours only
START_HOURS_RANGE=3-22

#period in days   	delay in minutes	job-identifier   	command
#1      	 	5		cron.daily        	nice run-parts  /etc/cron.daily				# 둘다 daily이므로 주석처리
7       		25      		cron.weekly        	nice run-parts  /etc/cron.weekly
@monthly 	45     		cron.monthly       	nice run-parts  /etc/cron.monthly
1		1    		cron.daily_test  	env RUN_BY=ANACRON run-parts  /etc/cron.daily		# 추가 설정
```

- 1 : 1일 마다 실행
- 5 : Anacron 실행 후 1분 지연
- cron.daily : Anacron 작업 이름
- env RUN_BY=ANACRON : 환경변수 RUN_BY를 생성 후 ANACRON 값을 적용
- run-parts : 디렉터리안의 스크립트를 일괄 실행
- /etc/cron.daily : 실행할 디렉터리 경로

```
: wq


[root@Server-A ~]# anacron -T	# 오류 메세지가 없으면 문법이 정상임을 의미
```

### 1) Anacron이 실행권한이 있는 경우 Anacron이 Daily 작업을 담당한다. (ANACRON 동작 확인)

```
[root@Server-A ~]# test  -x  /usr/sbin/anacron

[root@Server-A ~]# echo $?
0
```

강제로 Anacron 작업 실행

```
[root@Server-A ~]# cat /var/log/daily_test.log			# 아직 anacron 또는 cron이 동작하지 않았기때문에 로그 파일이 존재하지 않는다.
cat: /var/log/daily_test.log: 그런 파일이나 디렉터리가 없습니다
```

```
[root@Server-A ~]# anacron -n -f		# anacron 강제 실행

 # -n : 지연 시간을 기다리지 않고 즉시 실행
 # -f : 마지막 실행 날짜와 관계없이 강제로 실행
```

```
[root@Server-A ~]# cat /var/log/daily_test.log
2026-07-30 15:27:28 - 실행 주체 : ANACRON
```

동작 구조:
```
# /usr/sbin/anacron 실행 권한 있음  -->  Anacron 실행  -->  /etc/anacrontab 확인  -->  run-parts /etc/cron.daily  -->  daily_test 실행
# 실행 주체 : ANACRON
```

### 2) Anacron이 실행권한이 없는 경우 Cron이 Daily 작업을 담당한다. (CRON 동작 확인)

```
[root@Server-A ~]# ls  -l  /usr/sbin/anacron
-rwxr-xr-x. 1 root root 40464 12월 30  2025 /usr/sbin/anacron	# rwx r-x r-x	<--- 실행 권한이 있다.
```

```
[root@Server-A ~]# chmod -x  /usr/sbin/anacron


[root@Server-A ~]# ls  -l  /usr/sbin/anacron
-rw-r--r--. 1 root root 40464 12월 30  2025 /usr/sbin/anacron	# rw- r-- r--	<--- 실행 권한이 없다.
```

```
[root@Server-A ~]# cat /etc/anacrontab
#period in days   delay in minutes   job-identifier   command
#1      5       cron.daily              nice run-parts /etc/cron.daily
7       25      cron.weekly             nice run-parts /etc/cron.weekly
@monthly 45     cron.monthly            nice run-parts /etc/cron.monthly
1    1    cron.daily_test    env RUN_BY=ANACRON run-parts /etc/cron.daily	# 실행할 수 없다.
```

```
[root@Server-A ~]# test -x /usr/sbin/anacron


[root@Server-A ~]# echo $?
1				# 실행 권한 X
```

### Anacron이 동작하지 못하는경우 CRON 설정을 통해서 스크립트 실행

```
[root@Server-A ~]# vi  /etc/crontab
SHELL=/bin/bash
PATH=/sbin:/bin:/usr/sbin:/usr/bin
MAILTO=root

# For details see man 4 crontabs

# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
# *  *  *  *  * user-name  command to be executed

#*/5 * * * *  root   /script/hourly/login_user_check.sh
#35 11 * * 1-5 root  /script/hourly/temp_backup.sh
#12 13 * * *   root    /script/hourly/log_backup.sh
```

```
[root@Server-A ~]# vi  /etc/crontab

PATH=/sbin:/bin:/usr/sbin:/usr/bin
MAILTO=root

# For details see man 4 crontabs

# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
# *  *  *  *  * user-name  command to be executed

#*/5 * * * *  	root    /script/hourly/login_user_check.sh
#35 11 * * 1-5 	root   /script/hourly/temp_backup.sh
#12 13 * * *   	root   /script/hourly/log_backup.sh
55 15 * * *      	root   test -x  /usr/sbin/anacron || (cd  /  &&  env RUN_BY=CRON  run-parts   /etc/cron.daily)
```

- RUN_BY=ANACRON : ANACRON이 스크립트 실행
- RUN_BY=CRON : CRON이 스크립트 실행

```
[root@Server-A ~]# cat  /var/log/daily_test.log
2026-07-30 15:27:28 - 실행 주체 : ANACRON


[root@Server-A ~]# cat  /var/log/daily_test.log
2026-07-30 15:27:28 - 실행 주체 : ANACRON
2026-07-30 16:11:01 - 실행 주체 : CRON			# ANACRON에 대한 실행권한이 없기때문에 CRON이 스크립트를 실행한다.
```

```
[root@Server-A ~]# vi  /etc/crontab
SHELL=/bin/bash
PATH=/sbin:/bin:/usr/sbin:/usr/bin
MAILTO=root

# For details see man 4 crontabs

# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
# *  *  *  *  * user-name  command to be executed

#*/5 * * * *  	root   /script/hourly/login_user_check.sh
#35 11 * * 1-5 	root   /script/hourly/temp_backup.sh
#12 13 * * *   	root   /script/hourly/log_backup.sh
@reboot        	root   /script/hourly/reboot_check.sh		# 재부팅 후 root 권한으로 /script/hourly/reboot_check.sh 스크립트 실행
```

```
@reboot	: 시스템이 부팅되고 crond가 시작될 때 한 번 실행
@yearly	: 매년 1월 1일 00:00 실행
@monthly	: 매월 1일 00:00 실행
@weekly	: 매주 일요일 00:00 실행
@daily	: 매일 00:00 실행
@hourly	: 매시간 0분에 실행
```

**정리**: `test -x /usr/sbin/anacron` 결과에 따라 Anacron 또는 Cron이 daily 작업을 대신 수행하도록 이중화하는 방법을 실습했고, `RUN_BY` 환경변수로 실제 실행 주체를 로그에서 구분할 수 있음을 확인했다. 마지막으로 `@reboot` 예약어를 이용한 부팅 시 1회 실행 설정과 나머지 특수 예약어(@yearly, @monthly, @weekly, @daily, @hourly)를 정리했다.

# MariaDB 설치 및 데이터/정보 개념 (DB-01)

## 목차

1. [데이터와 정보](#데이터와-정보)
2. [데이터베이스는 왜 필요한가?](#데이터베이스는-왜-필요한가)
3. [MariaDB 설치](#mariadb-설치)

## 데이터와 정보

**데이터(Data)**와 **정보(Information)**는 비슷해 보이지만 전혀 다른 개념이다. 이 둘을 구분하는 것은 데이터베이스가 왜 필요한지, 그리고 MariaDB 같은 DBMS를 설치해 데이터를 체계적으로 저장·관리해야 하는 이유를 이해하는 출발점이 된다.

**데이터(Data)**
- 데이터는 가공되지 않은 사실(raw facts)이다.
- 현실 세계에서 관찰한 값 그 자체이며, 의미가 붙어 있지 않은 순수한 기록이다.

**예**
- 온도 측정값: 29, 30, 31
- 시험 점수: 85
- 출근 시간: 08:57
- 센서가 보낸 숫자: 1023
- 회원 이름: 김철수

**특징**
- 그 자체로는 의미가 없음
- 단순한 값(숫자, 문자, 날짜 등)
- 상황을 설명해주지 않는다.

**정보(Information)**
- 정보는 데이터를 어떤 방식으로 처리하여 의미를 가지는 형태로 만든 것이다.
- 데이터가 가공되면서 가치가 생긴 결과물이다.

**예**

| 데이터 | 정보 |
|---|---|
| 온도 데이터 29, 30, 31 | 오늘 평균 기온 30도 |
| 시험 점수 85 | 반 평균보다 높은 성적 |
| 출근 시간 08:57 | 지각 여부 판단 가능 |
| 센서 값 1023 | 위험 수준 경고 |
| 회원 데이터 | 마케팅 대상 고객군 분석 |

**특징**
- 데이터에 의미가 추가된다.
- 결정을 내릴 수 있는 기반이 된다.
- 사용자, 조직, 시스템에 가치 제공

**MySQL**은 데이터를 저장하는 도구이지, 정보 자체를 만들지는 않는다. 하지만 정보를 만들기 위한 기반을 제공한다.

**데이터베이스의 역할**
- 데이터를 정확하게 저장
- 원하는 데이터를 빠르게 찾도록 구조화
- 여러 사용자가 동시에 안전하게 사용하도록 관리

그리고 우리가 **SELECT**, 조건 검색, 집계 함수를 사용하는 이유는 데이터를 정보로 변환하기 위해서이다.

**예**

| 구분 | 내용 |
|---|---|
| 데이터 | 판매 내역 10,000건 |
| 정보 | 가장 많이 팔린 상품, 지역별 매출, 월별 추세 |

즉, 데이터베이스의 목적은 데이터 저장이 아니라 정보 가치 창출이다.

**정리**: 데이터는 가공되지 않은 원시 사실이고, 정보는 그 데이터를 처리해 의미와 가치를 부여한 결과물이다. 데이터베이스는 이 데이터를 정확히 저장·구조화하여 정보로 변환하는 기반을 제공한다.

## 데이터베이스는 왜 필요한가?

**데이터베이스(Database)**의 가장 큰 목적은 데이터를 중복 없이, 정확하게, 조직의 목적에 맞게, 효율적으로 관리하기 위함이다.

특히 온라인 서비스처럼 데이터가 폭발적으로 증가하는 환경에서는 DB 없이 운영이 불가능하다.

예) 이커머스 쇼핑몰을 운영시 만약 DB 없이 파일이나 엑셀로 관리시 문제점

### 판매 상품이 많고 관리할 데이터가 너무 많다

- 웹사이트에는 수천, 수만, 수십만 개의 상품이 등록된다.
- 상품명, 가격, 재고, 상세 정보, 이미지 등 많은 속성 필요
- 파일 기반 저장으로는 즉시 검색이나 수정이 어려움
- 상품 하나만 잘못 수정해도 전체 데이터가 꼬일 수 있음

**DB 사용 시**
- 구조적으로 데이터를 저장
- 원하는 상품을 빠르게 검색 가능
- 수정, 삭제 시 전체 데이터의 일관성이 유지된다.

### 많은 사용자가 동시에 많은 상품을 구매한다

- 동시 접속 100명, 1,000명, 10,000명일 수도 있다.
- 모두가 주문/결제/조회 작업을 실행한다.

**DB 없이 파일로 관리하면?**
- 파일이 동시에 열려 충돌 발생
- 재고 차감이 정확하지 않음
- 두 사람이 동시에 마지막 1개 남은 상품을 구매하는 오류 발생

**DB 사용 시**
- **트랜잭션(Transaction)** 처리
- 동시 작업에서도 데이터 오염 없이 정확히 처리
- 주문/재고 정보가 일관성을 유지함

### 매입, 매출, 반품, 재고를 정확히 파악해야 한다

쇼핑몰에서는 매일 다음과 같은 질문에 즉시 답해야 한다.
- 오늘 매출은 얼마인가?
- 카테고리별 매출은 얼마인가?
- 반품률은 어떤가?
- 재고는 어느 창고에 몇 개 남았는가?

**DB 없이 처리하면?**
- 엑셀에 수작업
- 실수 발생
- 최신 정보가 아님

**DB 사용 시**
- **SQL** 한 줄로 필요한 값을 빠르게 분석
- 항상 최신 데이터 기반으로 비즈니스 의사결정 가능함

### 시즌, 지역별로 잘 팔리는 상품을 분석해야 한다

**예**
- 여름엔 선풍기 매출 증가
- 겨울엔 히터 판매 급증
- 서울/부산 구매 패턴이 다를 수 있음

DB는 대량 데이터를 기준으로 어떤 패턴이 존재하는지 빠르게 분석 가능하다.

**DB 사용 시**
- 그룹, 정렬, 집계 기능으로 통계 분석
- 마케팅 전략, 재고 전략을 효율적으로 설계 가능

### 특정 고객의 구매 내역이 중요한 데이터가 된다

고객 한 사람의 구매 패턴만 봐도 다음 전략을 세울 수 있다.

**예**
- 어떤 고객이 어떤 카테고리를 자주 구매하는가
- VIP 고객의 1년 구매 총액은 얼마인가
- 고객 이탈 여부를 어떻게 판단할 것인가

**DB 없이 파일로 관리하면**
- 고객별 데이터 연결이 어렵고
- 분석도 거의 불가능

**DB 사용 시**
- 고객 테이블, 주문 테이블, 상품 테이블을 관계로 연결
- **JOIN**으로 쉽게 분석 가능
- 개인화 추천, 마케팅 전략에 활용

**정리**: 상품 수, 동시 사용자, 매출/재고 분석, 시즌별 트렌드, 고객별 구매 이력 등 이커머스의 거의 모든 측면은 파일 기반 관리로는 한계에 부딪히며, DB의 구조화·트랜잭션·집계·관계형 조인 기능이 있어야 정확하고 효율적인 운영이 가능하다.

## MariaDB 설치

MariaDB 패키지 설치부터 서비스 활성화, 초기 보안 설정, 계정 생성, 방화벽 설정, 설정 파일 수정까지의 전체 흐름을 다룬다.

```bash
[root@Server-A ~]# dnf  install  -y  mariadb-server

[root@Server-A ~]# rpm -qa | grep mariadb-server
mariadb-server-utils-10.5.29-3.el9_7.x86_64
mariadb-server-10.5.29-3.el9_7.x86_64
```

```bash
[root@Server-A ~]# systemctl  start  mariadb
```

```bash
[root@Server-A ~]# systemctl  enable  mariadb
Created symlink /etc/systemd/system/mysql.service → /usr/lib/systemd/system/mariadb.service.
Created symlink /etc/systemd/system/mysqld.service → /usr/lib/systemd/system/mariadb.service.
Created symlink /etc/systemd/system/multi-user.target.wants/mariadb.service → /usr/lib/systemd/system/mariadb.service.
```

```bash
[root@Server-A ~]# systemctl  status  mariadb
● mariadb.service - MariaDB 10.5 database server
     Loaded: loaded (/usr/lib/systemd/system/mariadb.service; enabled; preset: disabled)
     Active: active (running) since Thu 2026-07-30 17:09:53 KST; 12s ago
       Docs: man:mariadbd(8)
             https://mariadb.com/kb/en/library/systemd/
   Main PID: 4871 (mariadbd)
     Status: "Taking your SQL requests now..."
      Tasks: 15 (limit: 10322)
     Memory: 71.5M (peak: 102.4M)
        CPU: 491ms
     CGroup: /system.slice/mariadb.service
             └─4871 /usr/libexec/mariadbd --basedir=/usr
```

**mysql_secure_installation** : DB 설치후 기초 보안 관련 설정을 대화형으로 진행하는 스크립트

```bash
[root@Server-A ~]# mysql_secure_installation

NOTE: RUNNING ALL PARTS OF THIS SCRIPT IS RECOMMENDED FOR ALL MariaDB
      SERVERS IN PRODUCTION USE!  PLEASE READ EACH STEP CAREFULLY!

In order to log into MariaDB to secure it, we'll need the current
password for the root user. If you've just installed MariaDB, and
haven't set the root password yet, you should just press enter here.

Enter current password for root (enter for none):		# root 계정의 비밀번호 입력 (방금 설치했기때문에 root비빌번호가 없다. : enter)
OK, successfully used password, moving on...

Setting the root password or using the unix_socket ensures that nobody
can log into the MariaDB root user without the proper authorisation.

You already have your root account protected, so you can safely answer 'n'.

Switch to unix_socket authentication [Y/n] n		# unix_socket 이증방식 사용 유/무 (no)
 ... skipping.

You already have your root account protected, so you can safely answer 'n'.

Change the root password? [Y/n] y			# root 비빌번호 변경 유/무 (처음이면 설정 유무)
New password: admin1234				# 비밀번호 설정
Re-enter new password: admin1234			# 비빌번호 확인
Password updated successfully!
Reloading privilege tables..
 ... Success!


By default, a MariaDB installation has an anonymous user, allowing anyone
to log into MariaDB without having to have a user account created for
them.  This is intended only for testing, and to make the installation
go a bit smoother.  You should remove them before moving into a
production environment.

Remove anonymous users? [Y/n] n			# 익명 사용자의 접속 허용 유/무
 ... skipping.

Normally, root should only be allowed to connect from 'localhost'.  This
ensures that someone cannot guess at the root password from the network.

Disallow root login remotely? [Y/n] n			# root 사용자의 원격 접속 차단 유/무
 ... skipping.

By default, MariaDB comes with a database named 'test' that anyone can
access.  This is also intended only for testing, and should be removed
before moving into a production environment.

Remove test database and access to it? [Y/n]y		# 기본으로 제공되는 테스트용 데이터베이스 삭제 유/무
 - Dropping test database...
 ... Success!
 - Removing privileges on test database...
 ... Success!

Reloading the privilege tables will ensure that all changes made so far
will take effect immediately.

Reload privilege tables now? [Y/n]y			# 지금까지의 설정을 적용하고 재부팅
 ... Success!

Cleaning up...

All done!  If you've completed all of the above steps, your MariaDB
installation should now be secure.

Thanks for using MariaDB!
```

```bash
[root@Server-A ~]# mysql  -u  root -p
Enter password: admin1234
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 17
Server version: 10.5.29-MariaDB MariaDB Server

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]>
```

### 계정 생성

```sql
MariaDB [(none)]> CREATE USER 'user1'@'%' IDENTIFIED BY '1234';
Query OK, 0 rows affected (0.001 sec)
```

- **CREATE USER** : MariaDB에 새로운 사용자 계정을 생성
- **'user1'** : 생성할 MariaDB 사용자 이름
- **'@'** : 사용자 이름과 접속 허용 위치를 구분
- **'%'** : 모든 IP 주소 또는 모든 호스트에서 접속 허용
- **IDENTIFIED BY** : 사용자의 인증 비밀번호를 지정
- **'1234'** : user1 계정의 MariaDB 접속 비밀번호
- **;** : SQL 명령문의 끝

### 생성된 계정에 권한 부여

```sql
MariaDB [(none)]> GRANT ALL PRIVILEGES ON *.* TO 'user1'@'%';
Query OK, 0 rows affected (0.001 sec)
```

- **GRANT** : 사용자에게 권한을 부여하는 명령어
- **ALL PRIVILEGES** : 부여할 수 있는 대부분의 데이터베이스 권한을 모두 부여 (SELECT, INSERT, UPDATE, DELETE, CREATE, DROP 등의 권한 포함)
- **ON** : 권한을 적용할 대상 지정
- **\*.\*** : 모든 데이터베이스의 모든 테이블 (첫 번째 * = 모든 데이터베이스 , 두 번째 * = 해당 데이터베이스 안의 모든 테이블)
- **TO** : 권한을 부여받을 사용자 지정
- **'user1'@'%'** : 모든 호스트에서 접속하는 user1 계정
- **;** : SQL 명령문 종료

### 계정 생성 및 권한 정보를 다시 읽어서 적용

```sql
MariaDB [(none)]> FLUSH PRIVILEGES;	# MariaDB의 사용자 및 권한 정보를 다시 읽어서 변경된 권한을 적용
Query OK, 0 rows affected (0.001 sec)


MariaDB [(none)]> exit
```

방화벽에서 MariaDB가 사용하는 포트/서비스를 허용하고, 설정을 재적용한다.

```bash
[root@Server-A ~]# firewall-cmd  --permanent   --add-port=3306/tcp
success


[root@Server-A ~]# firewall-cmd  --permanent   --add-service=mysql
success


[root@Server-A ~]# firewall-cmd  --reload
success


[root@Server-A ~]# firewall-cmd  --list-port
3306/tcp


[root@Server-A ~]# firewall-cmd  --list-service
cockpit dhcpv6-client mysql ssh
```

```bash
[root@Server-A ~]# ls  -l  /etc/my.cnf.d/mariadb-server.cnf
-rw-r--r-- 1 root root 1458 12월 26  2025 /etc/my.cnf.d/mariadb-server.cnf
[root@Server-A ~]#
```

**MariaDB 설정 파일** 수정: 외부 인터페이스에서 접속을 허용하도록 `bind-address`의 주석을 해제한다.

```bash
[root@Server-A ~]# vi  /etc/my.cnf.d/mariadb-server.cnf	# mariaDB 설정 파일
~~~~~~~~~~~~ 중간 생략 ~~~~~~~~~~~~
     34 #
     35 # Allow server to accept connections on all interfaces.
     36 #
     37 bind-address=0.0.0.0		# 주석 삭제
     38 #
     39 # Optional setting
     40 #wsrep_slave_threads=1
     41 #innodb_flush_log_at_trx_commit=0
     42
~~~~~~~~~~~~ 중간 생략 ~~~~~~~~~~~~
```

```bash
[root@Server-A ~]# systemctl  restart  mariadb
```

### DB에 접속하기위한 workbench 다운로드

```
https://dev.mysql.com/downloads/workbench/
```

**정리**: MariaDB 설치는 패키지 설치 → 서비스 시작/활성화 → `mysql_secure_installation`을 통한 초기 보안 설정 → 계정 생성 및 권한 부여(`FLUSH PRIVILEGES`) → 방화벽 포트/서비스 허용 → 설정 파일(`bind-address`)에서 외부 접속 허용의 순서로 진행되며, 마지막으로 워크벤치 같은 GUI 클라이언트로 접속을 확인한다.

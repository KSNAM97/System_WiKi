# SQL 문법 (DDL / DML / DCL)

## SQL 문법 종류

**SQL** 문법은 크게 3가지 종류로 나눌 수 있다. 테이블을 만들고 구조를 바꾸는 작업, 데이터를 조회·삽입·수정·삭제하는 작업, 사용자 권한을 부여하거나 회수하는 작업처럼 목적에 따라 사용하는 명령의 종류가 달라진다.

- 데이터 정의 언어(Data Define Language, **DDL**)
- 데이터 조작 언어(Data Manipulation Language, **DML**)
- 데이터 제어 언어(Data Control Language, **DCL**)

**정리**: SQL은 구조를 정의하는 DDL, 데이터를 다루는 DML, 권한을 제어하는 DCL로 크게 3분류되며 각각 역할이 명확히 구분된다.

## 데이터 정의 언어(DDL: Data Define Language)

**DDL**은 데이터베이스의 구조를 정의하고 관리하는 언어이다. 즉, 무엇을 저장할 것인가가 아니라 어떻게 저장할 구조를 만들 것인가를 결정하는 명령어들이다.

데이터베이스, 테이블, 컬럼, 인덱스 등 데이터의 틀(구조)을 생성, 변경, 삭제할 때 사용한다.

**DDL의 특징**

- 테이블 구조 자체를 변경한다
  - 데이터를 넣고 빼는 것이 아니라, 저장될 그릇을 만드는 작업이다.
- 실행 즉시 적용된다
  - ROLLBACK(되돌리기) 불가능한 경우가 많다.
  - 즉, DDL은 실행하면 바로 구조가 바뀐다.
- 트랜잭션 영향을 거의 받지 않는다
  - 일반적으로 COMMIT 없이 바로 반영된다.
- 개발, 설계 단계에서 많이 사용되는 명령어

**DDL에 해당하는 대표 명령어**

- **CREATE** : 새로운 데이터베이스, 테이블, 뷰, 인덱스 등을 생성한다.
- **ALTER** : 이미 존재하는 테이블 구조를 변경한다. (컬럼 추가, 삭제, 타입 변경 등)
- **DROP** : 데이터베이스나 테이블을 삭제한다.
- **TRUNCATE** : 테이블의 데이터만 전체 삭제하고 구조는 남긴다. (DELETE 보다 빠르지만 되돌릴 수 없다.)
- **RENAME** : 테이블 또는 데이터베이스 이름을 변경한다.

**정리**: DDL은 CREATE/ALTER/DROP/TRUNCATE/RENAME으로 데이터베이스와 테이블의 '구조'를 다루며, 대부분 즉시 적용되고 되돌리기 어렵다는 점에 유의해야 한다.

## 데이터 조작 언어(DML: Data Manipulation Language)

**DML**은 테이블 안의 실제 데이터(Data)를 다루는 SQL 문장이다. 즉, 저장된 데이터를 추가(INSERT), 조회(SELECT), 수정(UPDATE), 삭제(DELETE) 하는 작업이 모두 포함된다.

DDL이 그릇(구조)을 만드는 언어라면 DML은 그릇 안에 담기는 음식(데이터)을 다루는 언어이다.

**DML의 특징**

- 데이터의 내용을 변경한다
  - 추가, 수정, 삭제된 값이 바로 반영된다.
- 실수 시 되돌리기 가능
  - COMMIT / ROLLBACK 기능을 통해 복구할 수 있다.
  - 반면 DDL은 되돌리기 어렵다.
- 실무에서 가장 많이 사용하는 SQL
  - CRUD(Create, Read, Update, Delete) 모두 DML로 수행된다.

**DML의 종류(4가지)**

1) **SELECT** : 데이터 조회
2) **INSERT** : 새로운 데이터 추가
3) **UPDATE** : 기존 데이터 수정
4) **DELETE** : 데이터 삭제

**정리**: DML은 SELECT/INSERT/UPDATE/DELETE로 실제 데이터를 다루며, COMMIT/ROLLBACK을 통해 실수를 되돌릴 수 있다는 점이 DDL과 가장 큰 차이다.

## 데이터 제어 언어(DCL: Data Control Language)

**DCL**은 데이터베이스에 대한 접근 권한을 관리하는 SQL 문장이다. 즉, 누가(DB 사용자), 어떤 DB나 테이블에, 어떤 작업을 할 수 있는지(접근 권한)를 허용하거나 제거하는 역할을 한다.

데이터 자체를 다루는 것이 아니라, 데이터를 사용할 수 있는 권한을 제어하는 것이 핵심이다.

**DCL의 목적**

1) 보안(Security) : DB는 중요 정보가 많기 때문에 아무나 접근하면 안 된다. (사용 가능한 권한을 부여/제한)
2) 사용자 관리 : 서비스 운영 시 개발자, 관리자, 외부 사용자 등 다양한 사용자가 접근함
3) 데이터 안정성 유지 : 불필요한 권한을 제거하여 실수로 삭제, 수정하는 사고를 방지

**DCL 주요 명령어(2가지)**

- **GRANT** : 권한을 부여하는 명령어
- **REVOKE** : 권한을 회수하는 명령어

**정리**: DCL은 GRANT/REVOKE로 데이터 자체가 아니라 '누가 무엇을 할 수 있는지'의 접근 권한을 통제하여 보안과 데이터 안정성을 지킨다.

## Database 생성 삭제

데이터베이스 자체를 만들고 지우는 기본 명령어이다.

```sql
# 데이터 베이스 생성
CREATE DATABASE dababase이름;

# 데이터 베이스 삭제
DROP DATABASE dababase이름;
```

**정리**: **CREATE DATABASE**와 **DROP DATABASE**로 데이터베이스 단위의 생성/삭제를 수행하며, 이는 모든 테이블 작업의 전제가 되는 최상위 구조 명령이다.

## 테이블 생성

**CREATE TABLE 테이블명;**

- 테이블은 행(row)과 열(column)의 집합이다
  - 행(row) = 하나의 데이터 묶음(레코드, 튜플)
  - 열(column) = 데이터의 속성(이름, 나이, 이메일 등)
- 각 열(column)은 특정 자료형을 가진다
  - INT(숫자)
  - VARCHAR(문자열)
  - DATE(날짜)
  - 자료형을 지정해야 데이터가 정확하게 저장된다.
- 테이블은 반드시 특정 데이터베이스 안에 생성된다
  - 먼저 데이터베이스(schema)를 만든 뒤
  - 그 안에 테이블을 생성해야 한다.
  - MySQL은 DB --> Table --> Row 구조로 데이터를 관리한다.
- 생성 가능한 테이블 개수는 사실상 무제한
  - MySQL/MariaDB 자체는 무제한 생성 가능
  - 다만 OS 파일 개수 제한이나 스토리지 용량의 영향을 받을 수 있다.
- **InnoDB** 스토리지 엔진은 매우 많은 테이블을 지원한다
  - InnoDB는 MySQL/MariaDB의 기본 스토리지 엔진
  - 테이블 수는 약 40억 개 이상까지 이론적으로 지원
  - 즉, "테이블이 부족해서 못 만든다"는 상황은 사실상 없음.
- 테이블은 데이터베이스 설계의 핵심
  - 어떤 정보를 저장할지 결정하는 단위
  - 잘 설계된 테이블은 검색, 수정, 분석이 매우 빠르고 안정적
  - 반대로 테이블 구조가 잘못되면 성능, 데이터 무결성에 큰 문제가 생긴다.

**정리**: 테이블은 행과 열로 구성되며 반드시 특정 데이터베이스 안에서 생성되고, 컬럼별 자료형 지정과 설계 품질이 이후 성능·무결성을 좌우한다.

## 테이블 삭제와 테이블 수정

데이터베이스에서는 "테이블을 삭제"할 수도 있고 "기존 테이블의 구조(컬럼)"를 변경할 수도 있다. 이때 사용하는 명령어가 **DROP**과 **ALTER**이다.

### 테이블 삭제 (DROP TABLE)

**DROP TABLE 테이블명;**

**특징**

- 테이블과 그 안의 모든 데이터가 완전히 삭제된다
- 되돌릴 수 없다 --> 매우 주의해야 한다
- 실무에서는 백업 없이 DROP 사용을 거의 금지한다

**예(삭제)**

```sql
DROP TABLE member;
```

- 테이블 전체 데이터만 삭제 (TRUNCATE TABLE)
- DROP은 구조 + 데이터 모두 삭제
- TRUNCATE는 구조는 남기고 데이터만 삭제

```sql
TRUNCATE TABLE member;
```

**특징**

- DELETE 보다 훨씬 빠르다
- 되돌릴 수 없다
- AUTO_INCREMENT 값도 초기화된다.

**정리**: **DROP TABLE**은 구조와 데이터를 모두 삭제하고, **TRUNCATE TABLE**은 구조는 남긴 채 데이터만 빠르게 비우며 AUTO_INCREMENT도 초기화한다. 둘 다 되돌릴 수 없으므로 신중히 사용해야 한다.

## 테이블 구조 수정 (ALTER TABLE)

**ALTER TABLE**은 다음 작업을 할 때 사용한다.

- 컬럼 추가
- 컬럼 삭제
- 자료형 변경
- 컬럼 이름 변경
- 기본키/제약조건 변경

**1) 컬럼 추가 (ADD)**

```sql
ALTER TABLE member 
ADD address VARCHAR(100);
```

컬럼 여러 개 추가

```sql
ALTER TABLE member
ADD phone VARCHAR(20),
ADD regdate DATETIME;
```

**2) 컬럼 삭제 (DROP COLUMN)**

```sql
ALTER TABLE member
DROP COLUMN address;
```

컬럼 삭제 시 데이터도 함께 삭제되므로 주의해야 한다.

**3) 컬럼 자료형 수정 (MODIFY)**

name 컬럼 길이를 50 --> 100으로 변경

```sql
ALTER TABLE member
MODIFY name VARCHAR(100);
```

자료형 자체 변경도 가능

```sql
ALTER TABLE member
MODIFY age BIGINT;
```

**4) 컬럼 이름 변경 (CHANGE)**

age --> user_age 로 변경하면서 자료형도 재정의

```sql
ALTER TABLE member
CHANGE age user_age INT;
```

**5) 테이블 이름 변경 (RENAME)**

```sql
ALTER TABLE member
RENAME TO customer;
```

**정리**: **ALTER TABLE**은 ADD/DROP COLUMN/MODIFY/CHANGE/RENAME TO를 통해 컬럼 추가·삭제·자료형 변경·이름 변경·테이블명 변경까지 테이블 구조 전반을 수정하는 핵심 DDL 명령이다.

## Table 자료형

### 숫자형(Number Types)

- **INT**
  - 정수 저장 (약 –21억 ~ +21억)
  - 예: 나이, 수량, 주문번호 등
- **BIGINT**
  - 더 큰 정수 (–9경 ~ +9경 수준)
  - 예: 매우 큰 PK, 로그ID, 누적값
- **FLOAT**
  - 소수점 있는 숫자(근사값)
  - 예: 온도, 실수 계산이 크게 중요하지 않은 값
- **DOUBLE**
  - FLOAT보다 높은 정밀도
  - 예: 과학 계산, 정확한 소수 필요 시
- **DECIMAL(정밀도, 소수점)**
  - 정확한 소수 계산 : 돈, 가격 저장에 필수
  - 예: DECIMAL(10,2) : 12345678.90 형태 저장 가능

### 문자형(String Types)

- **CHAR(n)**
  - 고정 길이 문자 : 항상 n바이트 채움
  - 예: 주민등록 앞자리, 국가코드
- **VARCHAR(n)**
  - 가변 길이 문자 : 실무에서 가장 많이 사용
  - 예 : 이름, 이메일, 주소
  - VARCHAR(50) = 최대 50자까지 저장 가능
- **TEXT**
  - 긴 문자열 저장
  - 예: 게시판 본문, 설명글
- **LONGTEXT**
  - 아주 긴 텍스트 저장
  - 예: 로그, 대용량 문서

### 날짜/시간형(Date & Time Types)

- **DATE**
  - 날짜(YYYY-MM-DD)
  - 예: 2024-12-05
- **TIME**
  - 시간(HH:MM:SS)
  - 예: 13:25:40
- **DATETIME**
  - 날짜 + 시간 저장
  - 예: 2024-12-05 13:25:40
  - 실무에서 가장 많이 사용된다.
- **TIMESTAMP**
  - UNIX 시간 기반 날짜+시간
  - 서버 시간대에 영향을 받는다
  - 로그 기록용으로 주로 사용된다.

### 논리형(Boolean)

- **BOOLEAN**
  - TRUE/FALSE 저장

**정리**: 자료형은 숫자형(INT/BIGINT/FLOAT/DOUBLE/DECIMAL), 문자형(CHAR/VARCHAR/TEXT/LONGTEXT), 날짜/시간형(DATE/TIME/DATETIME/TIMESTAMP), 논리형(BOOLEAN)으로 구분되며 데이터 특성에 맞는 자료형 선택이 저장 정확도와 성능에 직결된다.

## 테이블 생성 시 옵션(제약조건, Constraints)

SQL에서 테이블을 만들 때, 각 컬럼에 추가할 수 있는 조건(옵션)을 **제약조건**이라고 한다. 제약조건은 데이터가 잘못 입력되는 것을 미리 방지해주는 중요한 장치다.

아래는 MySQL/MariaDB에서 가장 많이 사용하는 제약조건이다.

- **PRIMARY KEY**
  - 해당 컬럼은 중복 불가, NULL 불가
  - 한 테이블에 오직 하나만 설정 가능
  - 보통 id, 번호, 고유 식별자에 설정
  - 자동 증가(AUTO_INCREMENT)와 자주 함께 사용된다.
  - EX) `id INT PRIMARY KEY`
  - EX) `id INT AUTO_INCREMENT PRIMARY KEY`
  - 테이블에서 하나의 행(row)을 구별하는 기준이 된다.
- **UNIQUE**
  - 해당 컬럼의 값이 중복될 수 없음
  - 하지만 NULL은 허용된다.
  - PRIMARY KEY와 달리 여러 개 컬럼에 적용 가능
  - EX) `email VARCHAR(100) UNIQUE`
  - 회원 이메일, 사용자 이름 등 중복을 막고 싶은 컬럼에 사용
- **NOT NULL**
  - NULL 값 입력 금지
  - 필수 입력 칸을 만들 때 사용
  - PRIMARY KEY도 자동으로 NOT NULL 포함
  - EX) `pw VARCHAR(20) NOT NULL`
  - 반드시 데이터가 있어야 하는 컬럼에 사용 (비밀번호, 이름, 가격 등)
- **DEFAULT**
  - 값을 입력하지 않았을 때 자동으로 들어가는 기본값
  - NULL이 들어가지 않게 하고 싶을 때 유용
  - EX) `age INT DEFAULT 18`
  - EX) `status VARCHAR(10) DEFAULT 'active'`
  - EX) `reg_date DATETIME DEFAULT CURRENT_TIMESTAMP`
  - 회원가입 시 기본 나이, 기본 등급 설정 등에서 활용
- **CHECK (조건식)**
  - 특정 조건을 만족하는 값만 입력되도록 제한
  - MySQL 8.x : 정상적으로 동작
  - MariaDB 10.x : CHECK 구문은 작성되지만 일부 버전에서는 강제되지 않을 수 있음
  - `age INT CHECK (age >= 18 AND age <= 65)`
  - `score INT CHECK (score BETWEEN 0 AND 100)`
  - 실수 방지, 값의 범위 제한
  - 점수, 나이, 수량 등 범위가 정해진 숫자 컬럼에 사용

```sql
CREATE  TABLE  test ( 
	id  VARCHAR(16)  PRIMARY KEY, 
	pw  VARCHAR(20) NOT NULL, 
	age INT DEFAULT 18  CHECK  (age >= 18 AND age <= 65),
	birth DATE
);
```

```sql
INSERT INTO test VALUES('hong', '1234', 20, '2026-07-31');
```

**정리**: 제약조건(PRIMARY KEY, UNIQUE, NOT NULL, DEFAULT, CHECK)은 테이블 설계 단계에서 데이터의 무결성과 유효 범위를 보장하는 핵심 장치이며, 조합하여 사용함으로써 잘못된 데이터 입력을 원천적으로 방지할 수 있다.

## 실습: member 테이블로 배우는 ALTER / INSERT / SELECT / UPDATE / DELETE

아래는 실제 `member` 테이블을 생성하고, **ALTER**, **INSERT**, **SELECT**, **WHERE**, **UPDATE**, **DELETE**를 순서대로 실습하는 예제 모음이다. 각 EX(실습) 뒤에는 실행한 SQL과 그 결과(DESC 출력 또는 SELECT 결과)가 그대로 포함되어 있다.

```sql
CREATE TABLE member (
    member_id  	INT(10) PRIMARY KEY,	   	-- 회원을 구분하는 고유 번호, 기본 키
    username   	VARCHAR(50) NOT NULL,	   	-- 회원 이름 또는 로그인 아이디, NULL 입력 불가
    password   	VARCHAR(100) NOT NULL,	   	-- 비밀번호, NULL 입력 불가
    email      	VARCHAR(100),             		-- 이메일 주소
    phone      	VARCHAR(20),       			-- 전화번호
    birth_date 	DATE,                 			-- 생년월일
    join_date  	DATETIME DEFAULT NOW(),		-- 가입 일시, 입력하지 않으면 현재 날짜와 시간 저장
    status     	CHAR(1) DEFAULT 'A'     		-- 회원 상태, 기본값 A
);
```

### ALTER TABLE 실습 (EX1~EX7)

**EX1) phone 컬럼의 이름을 smartPhone으로 변경**

```sql
ALTER TABLE member CHANGE phone  smartPhone VARCHAR(20);
```

```
MariaDB [MyDB]> DESC member;
+-----------------+------------------+--------+--------+---------------------------+---------+
| Field      	| Type         	| Null	| Key | Default             		| Extra 	|
+-----------------+------------------+--------+--------+---------------------------+---------+
| member_id  	| int(10)      	| NO   	| PRI	| NULL                		|       	|
| username   	| varchar(50)  	| NO   	|     	| NULL                		|       	|
| password   	| varchar(100)	| NO   	|     	| NULL                		|       	|
| email      	| varchar(100) 	| YES  	|     	| NULL                		|       	|
| smartPhone 	| varchar(20)  	| YES  	|     	| NULL                		|       	|
| birth_date 	| date         	| YES  	|     	| NULL                		|       	|
| join_date  	| datetime     	| YES  	|     	| current_timestamp()	|       	|
| status     	| char(1)      	| YES  	|     	| A			|       	|
+-----------------+------------------+--------+--------+---------------------------+---------+
```

**EX2) status 컬럼의 데이터 타입을 CHAR(1)에서 VARCHAR(10)으로 변경**

```sql
ALTER TABLE member MODIFY status VARCHAR(10)
```

```
MariaDB [MyDB]> DESC member;
+-----------------+------------------+--------+--------+---------------------------+---------+
| Field      	| Type         	| Null	| Key | Default             		| Extra 	|
+-----------------+------------------+--------+--------+---------------------------+---------+
| member_id  	| int(10)      	| NO   	| PRI	| NULL                		|       	|
| username   	| varchar(50)  	| NO   	|     	| NULL                		|       	|
| password   	| varchar(100)	| NO   	|     	| NULL                		|       	|
| email      	| varchar(100) 	| YES  	|     	| NULL                		|       	|
| smartPhone 	| varchar(20)  	| YES  	|     	| NULL                		|       	|
| birth_date 	| date         	| YES  	|     	| NULL                		|       	|
| join_date  	| datetime     	| YES  	|     	| current_timestamp()	|       	|
| status     	| varchar(10)      	| YES  	|     	| A			|       	|
+-----------------+------------------+--------+--------+---------------------------+---------+
```

**EX3) email 컬럼이 반드시 입력되도록 NOT NULL 제약 조건을 추가**

```sql
ALTER TABLE member MODIFY email VARCHAR(100) NOT NULL;
```

```
MariaDB [MyDB]> DESC member;
+-----------------+------------------+--------+--------+---------------------------+---------+
| Field      	| Type         	| Null	| Key | Default             		| Extra 	|
+-----------------+------------------+--------+--------+---------------------------+---------+
| member_id  	| int(10)      	|   NO   	| PRI	| NULL                		|       	|
| username   	| varchar(50)  	|   NO   	|     	| NULL                		|       	|
| password   	| varchar(100)	|   NO   	|     	| NULL                		|       	|
| email      	| varchar(100) 	|   NO  	|     	| NULL                		|       	|
| smartPhone 	| varchar(20)  	|   YES  	|     	| NULL                		|       	|
| birth_date 	| date         	|   YES  	|     	| NULL                		|       	|
| join_date  	| datetime     	|   YES  	|     	| current_timestamp()	|       	|
| status     	| varchar(10)      	|   YES  	|     	| A			|       	|
+-----------------+------------------+--------+--------+---------------------------+---------+
```

**EX4) 회원의 마지막 로그인 시간을 저장하는 last_login 컬럼을 추가**

```sql
ALTER TABLE member ADD last_login  DATETIME; 
```

```
MariaDB [MyDB]> DESC member;
+-----------------+------------------+--------+--------+---------------------------+---------+
| Field      	| Type         	| Null	| Key | Default             		| Extra 	|
+-----------------+------------------+--------+--------+---------------------------+---------+
| member_id  	| int(10)      	|   NO   	| PRI	| NULL                		|       	|
| username   	| varchar(50)  	|   NO   	|     	| NULL                		|       	|
| password   	| varchar(100)	|   NO   	|     	| NULL                		|       	|
| email      	| varchar(100) 	|   NO  	|     	| NULL                		|       	|
| smartPhone 	| varchar(20)  	|   YES  	|     	| NULL                		|       	|
| birth_date 	| date         	|   YES  	|     	| NULL                		|       	|
| join_date  	| datetime     	|   YES  	|     	| current_timestamp()	|       	|
| status     	| varchar(10)      	|   YES  	|     	| A			|       	|
| last_login 	| datetime     	| YES  	|     	| NULL                		|       	|
+-----------------+------------------+--------+--------+---------------------------+---------+
```

**EX5) birth_date 컬럼을 삭제**

```sql
ALTER TABLE member DROP COLUMN birth_date;
```

```
MariaDB [MyDB]> DESC member;
+-----------------+------------------+--------+--------+---------------------------+---------+
| Field      	| Type         	| Null	| Key 	| Default             		| Extra 	|
+-----------------+------------------+--------+--------+---------------------------+---------+
| member_id  	| int(10)      	|   NO   	| PRI	| NULL                		|       	|
| username   	| varchar(50)  	|   NO   	|     	| NULL                		|       	|
| password   	| varchar(100)	|   NO   	|     	| NULL                		|       	|
| email      	| varchar(100) 	|   NO  	|     	| NULL                		|       	|
| smartPhone 	| varchar(20)  	|   YES  	|     	| NULL                		|       	|
| join_date  	| datetime     	|   YES  	|     	| current_timestamp()	|       	|
| status     	| varchar(10)      	|   YES  	|     	| A			|       	|
| last_login 	| datetime     	|   YES  	|     	| NULL                		|       	|
+-----------------+------------------+--------+--------+---------------------------+---------+
```

**EX6) 회원의 나이를 저장할 수 있도록 age 컬럼을 3자리 정수로 추가**

```sql
ALTER TABLE member ADD age INT(3);
```

```
MariaDB [MyDB]> DESC member;
+-----------------+------------------+--------+--------+---------------------------+---------+
| Field      	| Type         	| Null	| Key 	| Default             		| Extra 	|
+-----------------+------------------+--------+--------+---------------------------+---------+
| member_id  	| int(10)      	|   NO   	| PRI	| NULL                		|       	|
| username   	| varchar(50)  	|   NO   	|     	| NULL                		|       	|
| password   	| varchar(100)	|   NO   	|     	| NULL                		|       	|
| email      	| varchar(100) 	|   NO  	|     	| NULL                		|       	|
| smartPhone 	| varchar(20)  	|   YES  	|     	| NULL                		|       	|
| join_date  	| datetime     	|   YES  	|     	| current_timestamp()	|       	|
| status     	| varchar(10)      	|   YES  	|     	| A			|       	|
| last_login 	| datetime     	|   YES  	|     	| NULL                		|       	|
| age        	| int(3)       	|   YES  	|     	| NULL                		|       	|
+-----------------+------------------+--------+--------+---------------------------+---------+
```

**EX7) age 컬럼의 기본값을 18로 변경**

```sql
ALTER TABLE member MODIFY age INT(3) DEFAULT 18;
```

```
MariaDB [MyDB]> DESC member;
+-----------------+------------------+--------+--------+---------------------------+---------+
| Field      	| Type         	| Null	| Key 	| Default             		| Extra 	|
+-----------------+------------------+--------+--------+---------------------------+---------+
| member_id  	| int(10)      	|   NO   	| PRI	| NULL                		|       	|
| username   	| varchar(50)  	|   NO   	|     	| NULL                		|       	|
| password   	| varchar(100)	|   NO   	|     	| NULL                		|       	|
| email      	| varchar(100) 	|   NO  	|     	| NULL                		|       	|
| smartPhone 	| varchar(20)  	|   YES  	|     	| NULL                		|       	|
| join_date  	| datetime     	|   YES  	|     	| current_timestamp()	|       	|
| status     	| varchar(10)      	|   YES  	|     	| A			|       	|
| last_login 	| datetime     	|   YES  	|     	| NULL                		|       	|
| age        	| int(3)       	|   YES  	|     	| 18                		|       	|
+-----------------+------------------+--------+--------+---------------------------+---------+
```

### INSERT / SELECT 기본 실습 (EX8~EX10)

**EX8) 새 회원 1명 추가**

```sql
INSERT INTO member 
(member_id, username, password, email) 
VALUES(1, 'kim', 'passwd1234', 'kim@example.com');
```

**EX9) member 테이블의 모든 회원을 조회**

```sql
SELECT * FROM member;
```

**EX10) 모든 회원의 username과 email만 조회**

```sql
SELECT username, email FROM member;
```

다수 회원을 한 번에 채워 이후 실습(WHERE, UPDATE, DELETE)에 사용할 샘플 데이터를 아래와 같이 추가로 **INSERT**한다.

```sql
INSERT INTO member
(member_id, username, password, email, smartPhone, status, last_login, age)
VALUES
(1, 'user01', 'pass1234', 'user01@test.com', '010-1001-1001', 'ACTIVE', NOW(), 20);

INSERT INTO member
(member_id, username, password, email, smartPhone, status, last_login, age)
VALUES
(2, 'user02', 'pass1234', 'user02@test.com', '010-1002-1002', 'ACTIVE', NOW(), 21);

INSERT INTO member
(member_id, username, password, email, smartPhone, status, last_login, age)
VALUES
(3, 'user03', 'pass1234', 'user03@test.com', '010-1003-1003', 'ACTIVE', NOW(), 22);

INSERT INTO member
(member_id, username, password, email, smartPhone, status, last_login, age)
VALUES
(4, 'user04', 'pass1234', 'user04@test.com', '010-1004-1004', 'ACTIVE', NOW(), 23);

INSERT INTO member
(member_id, username, password, email, smartPhone, status, last_login, age)
VALUES
(5, 'user05', 'pass1234', 'user05@test.com', '010-1005-1005', 'ACTIVE', NOW(), 24);

INSERT INTO member
(member_id, username, password, email, smartPhone, status, last_login, age)
VALUES
(6, 'user06', 'pass1234', 'user06@test.com', '010-1006-1006', 'ACTIVE', NOW(), 25);

INSERT INTO member
(member_id, username, password, email, smartPhone, status, last_login, age)
VALUES
(7, 'user07', 'pass1234', 'user07@test.com', '010-1007-1007', 'ACTIVE', NOW(), 26);

INSERT INTO member
(member_id, username, password, email, smartPhone, status, last_login, age)
VALUES
(8, 'user08', 'pass1234', 'user08@test.com', '010-1008-1008', 'ACTIVE', NOW(), 27);

INSERT INTO member
(member_id, username, password, email, smartPhone, status, last_login, age)
VALUES
(9, 'user09', 'pass1234', 'user09@test.com', '010-1009-1009', 'ACTIVE', NOW(), 28);

INSERT INTO member
(member_id, username, password, email, smartPhone, status, last_login, age)
VALUES
(10, 'user10', 'pass1234', 'user10@test.com', '010-1010-1010', 'ACTIVE', NOW(), 29);

INSERT INTO member
(member_id, username, password, email, smartPhone, status, last_login, age)
VALUES
(11, 'user11', 'pass5678', 'user11@test.com', '010-1011-1011', 'ACTIVE', NOW(), 30);

INSERT INTO member
(member_id, username, password, email, smartPhone, status, last_login, age)
VALUES
(12, 'user12', 'pass5678', 'user12@test.com', '010-1012-1012', 'ACTIVE', NOW(), 31);

INSERT INTO member
(member_id, username, password, email, smartPhone, status, last_login, age)
VALUES
(13, 'user13', 'pass5678', 'user13@test.com', '010-1013-1013', 'INACTIVE', NULL, 32);

INSERT INTO member
(member_id, username, password, email, smartPhone, status, last_login, age)
VALUES
(14, 'user14', 'pass5678', 'user14@test.com', '010-1014-1014', 'ACTIVE', NOW(), 33);

INSERT INTO member
(member_id, username, password, email, smartPhone, status, last_login, age)
VALUES
(15, 'user15', 'pass5678', 'user15@test.com', '010-1015-1015', 'INACTIVE', NULL, 34);

INSERT INTO member
(member_id, username, password, email, smartPhone, status, last_login, age)
VALUES
(16, 'user16', 'linux1234', 'user16@test.com', '010-1016-1016', 'ACTIVE', NOW(), 35);

INSERT INTO member
(member_id, username, password, email, smartPhone, status, last_login, age)
VALUES
(17, 'user17', 'linux1234', 'user17@test.com', '010-1017-1017', 'ACTIVE', NOW(), 36);

INSERT INTO member
(member_id, username, password, email, smartPhone, status, last_login, age)
VALUES
(18, 'user18', 'linux1234', 'user18@test.com', '010-1018-1018', 'INACTIVE', NULL, 37);

INSERT INTO member
(member_id, username, password, email, smartPhone, status, last_login, age)
VALUES
(19, 'user19', 'maria1234', 'user19@test.com', '010-1019-1019', 'ACTIVE', NOW(), 38);

INSERT INTO member
(member_id, username, password, email, smartPhone, status, last_login, age)
VALUES
(20, 'user20', 'maria1234', 'user20@test.com', '010-1020-1020', 'ACTIVE', NOW(), 39);
```

```
MariaDB [MyDB]> SELECT * FROM member;
+-----------+----------+------------+-----------------+---------------+---------------------+----------+---------------------+------+
| member_id | username | password   | email           | smartPhone    | join_date           | status   | last_login          | age  |
+-----------+----------+------------+-----------------+---------------+---------------------+----------+---------------------+------+
|         1     | kim      | passwd1234 | kim@example.com | NULL          | 2026-07-31 12:14:00 | NULL     | NULL                |   18 |
|         2    | user02   | pass1234   | user02@test.com | 010-1002-1002 | 2026-07-31 12:20:42 | ACTIVE   | 2026-07-31 12:20:42 |   21 |
|         3    | user03   | pass1234   | user03@test.com | 010-1003-1003 | 2026-07-31 12:20:42 | ACTIVE   | 2026-07-31 12:20:42 |   22 |
|         4    | user04   | pass1234   | user04@test.com | 010-1004-1004 | 2026-07-31 12:20:42 | ACTIVE   | 2026-07-31 12:20:42 |   23 |
|         5    | user05   | pass1234   | user05@test.com | 010-1005-1005 | 2026-07-31 12:20:42 | ACTIVE   | 2026-07-31 12:20:42 |   24 |
|         6    | user06   | pass1234   | user06@test.com | 010-1006-1006 | 2026-07-31 12:20:42 | ACTIVE   | 2026-07-31 12:20:42 |   25 |
|         7    | user07   | pass1234   | user07@test.com | 010-1007-1007 | 2026-07-31 12:20:42 | ACTIVE   | 2026-07-31 12:20:42 |   26 |
|         8    | user08   | pass1234   | user08@test.com | 010-1008-1008 | 2026-07-31 12:20:42 | ACTIVE   | 2026-07-31 12:20:42 |   27 |
|         9    | user09   | pass1234   | user09@test.com | 010-1009-1009 | 2026-07-31 12:20:42 | ACTIVE   | 2026-07-31 12:20:42 |   28 |
|        10    | user10   | pass1234   | user10@test.com | 010-1010-1010 | 2026-07-31 12:20:42 | ACTIVE   | 2026-07-31 12:20:42 |   29 |
|        11    | user11   | pass5678   | user11@test.com | 010-1011-1011 | 2026-07-31 12:20:42 | ACTIVE   | 2026-07-31 12:20:42 |   30 |
|        12    | user12   | pass5678   | user12@test.com | 010-1012-1012 | 2026-07-31 12:20:42 | ACTIVE   | 2026-07-31 12:20:42 |   31 |
|        13    | user13   | pass5678   | user13@test.com | 010-1013-1013 | 2026-07-31 12:20:42 | INACTIVE | NULL                   |   32 |
|        14    | user14   | pass5678   | user14@test.com | 010-1014-1014 | 2026-07-31 12:20:42 | ACTIVE   | 2026-07-31 12:20:42 |   33 |
|        15    | user15   | pass5678   | user15@test.com | 010-1015-1015 | 2026-07-31 12:20:42 | INACTIVE | NULL                   |   34 |
|        16    | user16   | linux1234  | user16@test.com | 010-1016-1016 | 2026-07-31 12:20:42 | ACTIVE   | 2026-07-31 12:20:42 |   35 |
|        17    | user17   | linux1234  | user17@test.com | 010-1017-1017 | 2026-07-31 12:20:42 | ACTIVE   | 2026-07-31 12:20:42 |   36 |
|        18    | user18   | linux1234  | user18@test.com | 010-1018-1018 | 2026-07-31 12:20:42 | INACTIVE | NULL                   |   37 |
|        19    | user19   | maria1234  | user19@test.com | 010-1019-1019 | 2026-07-31 12:20:42 | ACTIVE   | 2026-07-31 12:20:42 |   38 |
|        20    | user20   | maria1234  | user20@test.com | 010-1020-1020 | 2026-07-31 12:20:42 | ACTIVE   | 2026-07-31 12:20:42 |   39 |
+-----------+----------+------------+-----------------+---------------+---------------------+----------+---------------------+------+
```

### WHERE 조건 검색 실습 (EX11~EX17)

**EX11) 특정 회원 1명만 조회 (PK 사용)**
- member_id가 2인 회원의 모든 정보를 조회

```
member_id	username	password	email		smartPhone	join_date			status	last_login	age
2		user02	pass1234	user02@test.com	010-1002-1002	2026-07-31 12:20:42		ACTIVE	2026-07-31 12:20:42	21
```

**EX12) status가 'ACTIVE' 인 회원만 조회**

```sql
SELECT * FROM member WHERE status='ACTIVE';
```

```
# member_id	username	password	email		smartPhone	join_date		status	last_login		 age
2		user02	pass1234	user02@test.com	010-1002-1002	2026-07-31 12:20:42	ACTIVE	2026-07-31 12:20:42	 21
3		user03	pass1234	user03@test.com	010-1003-1003	2026-07-31 12:20:42	ACTIVE	2026-07-31 12:20:42	 22
4		user04	pass1234	user04@test.com	010-1004-1004	2026-07-31 12:20:42	ACTIVE	2026-07-31 12:20:42	 23
5		user05	pass1234	user05@test.com	010-1005-1005	2026-07-31 12:20:42	ACTIVE	2026-07-31 12:20:42	 24
6		user06	pass1234	user06@test.com	010-1006-1006	2026-07-31 12:20:42	ACTIVE	2026-07-31 12:20:42	 25
7		user07	pass1234	user07@test.com	010-1007-1007	2026-07-31 12:20:42	ACTIVE	2026-07-31 12:20:42	 26
8		user08	pass1234	user08@test.com	010-1008-1008	2026-07-31 12:20:42	ACTIVE	2026-07-31 12:20:42	 27
9		user09	pass1234	user09@test.com	010-1009-1009	2026-07-31 12:20:42	ACTIVE	2026-07-31 12:20:42	 28
10		user10	pass1234	user10@test.com	010-1010-1010	2026-07-31 12:20:42	ACTIVE	2026-07-31 12:20:42	 29
11		user11	pass5678	user11@test.com	010-1011-1011	2026-07-31 12:20:42	ACTIVE	2026-07-31 12:20:42	 30
12		user12	pass5678	user12@test.com	010-1012-1012	2026-07-31 12:20:42	ACTIVE	2026-07-31 12:20:42	 31
14		user14	pass5678	user14@test.com	010-1014-1014	2026-07-31 12:20:42	ACTIVE	2026-07-31 12:20:42	 33
16		user16	linux1234	user16@test.com	010-1016-1016	2026-07-31 12:20:42	ACTIVE	2026-07-31 12:20:42	 35
17		user17	linux1234	user17@test.com	010-1017-1017	2026-07-31 12:20:42	ACTIVE	2026-07-31 12:20:42	 36
19		user19	maria1234	user19@test.com	010-1019-1019	2026-07-31 12:20:42	ACTIVE	2026-07-31 12:20:42	 38
20		user20	maria1234	user20@test.com	010-1020-1020	2026-07-31 12:20:42	ACTIVE	2026-07-31 12:20:42	 39
```

**EX13) age가 35 이상인 회원을 조회**

```sql
SELECT * FROM member WHERE age >= 35;
```

```
member_id	username	password	email		smartPhone	join_date		status	last_login		 age
16		user16	linux1234	user16@test.com	010-1016-1016	2026-07-31 12:20:42	ACTIVE	2026-07-31 12:20:42	 35
17		user17	linux1234	user17@test.com	010-1017-1017	2026-07-31 12:20:42	ACTIVE	2026-07-31 12:20:42	 36
18		user18	linux1234	user18@test.com	010-1018-1018	2026-07-31 12:20:42	INACTIVE		 37
19		user19	maria1234	user19@test.com	010-1019-1019	2026-07-31 12:20:42	ACTIVE	2026-07-31 12:20:42	 38
20		user20	maria1234	user20@test.com	010-1020-1020	2026-07-31 12:20:42	ACTIVE	2026-07-31 12:20:42	 39
```

**EX14) age가 30 미만인 회원을 조회**

```sql
SELECT * FROM member WHERE age < 30;
```

```
# member_id	username	password	email	smartPhone	join_date	status		last_login		 age
1	kim	passwd1234	kim@example.com		2026-07-31 12:14:00				 18
2	user02	pass1234	user02@test.com	010-1002-1002	2026-07-31 12:20:42	ACTIVE	2026-07-31 12:20:42	 21
3	user03	pass1234	user03@test.com	010-1003-1003	2026-07-31 12:20:42	ACTIVE	2026-07-31 12:20:42	 22
4	user04	pass1234	user04@test.com	010-1004-1004	2026-07-31 12:20:42	ACTIVE	2026-07-31 12:20:42	 23
5	user05	pass1234	user05@test.com	010-1005-1005	2026-07-31 12:20:42	ACTIVE	2026-07-31 12:20:42	 24
6	user06	pass1234	user06@test.com	010-1006-1006	2026-07-31 12:20:42	ACTIVE	2026-07-31 12:20:42	 25
7	user07	pass1234	user07@test.com	010-1007-1007	2026-07-31 12:20:42	ACTIVE	2026-07-31 12:20:42	 26
8	user08	pass1234	user08@test.com	010-1008-1008	2026-07-31 12:20:42	ACTIVE	2026-07-31 12:20:42	 27
9	user09	pass1234	user09@test.com	010-1009-1009	2026-07-31 12:20:42	ACTIVE	2026-07-31 12:20:42	 28
10	user10	pass1234	user10@test.com	010-1010-1010	2026-07-31 12:20:42	ACTIVE	2026-07-31 12:20:42	 29
```

**EX15) 두 조건을 AND로 묶기**
- status가 'ACTIVE' 이고, age가 35 이상인 회원만 조회

```sql
SELECT * FROM member WHERE status='ACTIVE' AND age >= 35;
```

```
# member_id	username	password	email		smartPhone	join_date	status		last_login		 age
16		user16	linux1234	user16@test.com	010-1016-1016	2026-07-31 12:20:42	ACTIVE	2026-07-31 12:20:42	 35
17		user17	linux1234	user17@test.com	010-1017-1017	2026-07-31 12:20:42	ACTIVE	2026-07-31 12:20:42	 36
19		user19	maria1234	user19@test.com	010-1019-1019	2026-07-31 12:20:42	ACTIVE	2026-07-31 12:20:42	 38
20		user20	maria1234	user20@test.com	010-1020-1020	2026-07-31 12:20:42	ACTIVE	2026-07-31 12:20:42	 39
```

**EX16) OR 조건 사용하기**
- status가 'ACTIVE' 이거나 age가 35 이상인 회원을 조회

```sql
SELECT * FROM member
WHERE status='ACTIVE' OR age >= 35;
```

```
# member_id	username	password	email		smartPhone	join_date	status		last_login		 age
2		user02	pass1234	user02@test.com	010-1002-1002	2026-07-31 12:20:42	ACTIVE	2026-07-31 12:20:42	 21
3		user03	pass1234	user03@test.com	010-1003-1003	2026-07-31 12:20:42	ACTIVE	2026-07-31 12:20:42	 22
4		user04	pass1234	user04@test.com	010-1004-1004	2026-07-31 12:20:42	ACTIVE	2026-07-31 12:20:42	 23
5		user05	pass1234	user05@test.com	010-1005-1005	2026-07-31 12:20:42	ACTIVE	2026-07-31 12:20:42	 24
6		user06	pass1234	user06@test.com	010-1006-1006	2026-07-31 12:20:42	ACTIVE	2026-07-31 12:20:42	 25
7		user07	pass1234	user07@test.com	010-1007-1007	2026-07-31 12:20:42	ACTIVE	2026-07-31 12:20:42	 26
8		user08	pass1234	user08@test.com	010-1008-1008	2026-07-31 12:20:42	ACTIVE	2026-07-31 12:20:42	 27
9		user09	pass1234	user09@test.com	010-1009-1009	2026-07-31 12:20:42	ACTIVE	2026-07-31 12:20:42	 28
10		user10	pass1234	user10@test.com	010-1010-1010	2026-07-31 12:20:42	ACTIVE	2026-07-31 12:20:42	 29
11		user11	pass5678	user11@test.com	010-1011-1011	2026-07-31 12:20:42	ACTIVE	2026-07-31 12:20:42	 30
12		user12	pass5678	user12@test.com	010-1012-1012	2026-07-31 12:20:42	ACTIVE	2026-07-31 12:20:42	 31
14		user14	pass5678	user14@test.com	010-1014-1014	2026-07-31 12:20:42	ACTIVE	2026-07-31 12:20:42	 33
16		user16	linux1234	user16@test.com	010-1016-1016	2026-07-31 12:20:42	ACTIVE	2026-07-31 12:20:42	 35
17		user17	linux1234	user17@test.com	010-1017-1017	2026-07-31 12:20:42	ACTIVE	2026-07-31 12:20:42	 36
18		user18	linux1234	user18@test.com	010-1018-1018	2026-07-31 12:20:42	INACTIVE		 37
19		user19	maria1234	user19@test.com	010-1019-1019	2026-07-31 12:20:42	ACTIVE	2026-07-31 12:20:42	 38
20		user20	maria1234	user20@test.com	010-1020-1020	2026-07-31 12:20:42	ACTIVE	2026-07-31 12:20:42	 39
```

**EX17) 나이가 30세 이상이고 35세 미만인 회원이거나 회원 상태가 INDACTIVE인 회원 조회**

```sql
SELECT * FROM member
WHERE (age >= 30 AND age < 35) OR status='INACTIVE';
```

```
# member_id	username	password	email		smartPhone	join_date		status	last_login		 age
11		user11	pass5678	user11@test.com	010-1011-1011	2026-07-31 12:20:42	ACTIVE	2026-07-31 12:20:42	 30
12		user12	pass5678	user12@test.com	010-1012-1012	2026-07-31 12:20:42	ACTIVE	2026-07-31 12:20:42	 31
13		user13	pass5678	user13@test.com	010-1013-1013	2026-07-31 12:20:42	INACTIVE		 32
14		user14	pass5678	user14@test.com	010-1014-1014	2026-07-31 12:20:42	ACTIVE	2026-07-31 12:20:42	 33
15		user15	pass5678	user15@test.com	010-1015-1015	2026-07-31 12:20:42	INACTIVE		 34
18		user18	linux1234	user18@test.com	010-1018-1018	2026-07-31 12:20:42	INACTIVE		 37
```

**정리**: WHERE 절은 비교 연산자(`=`, `>=`, `<`), 논리 연산자(**AND**, **OR**), 괄호를 조합해 원하는 조건의 행만 정밀하게 조회할 수 있게 해준다.

### UPDATE 실습 (EX18~EX20)

**EX18) 특정 회원의 전화번호 수정**
- member_id가 3인 회원의 smartPhone을 '010-2222-3333' 으로 수정

```sql
UPDATE member
SET smartPhone = '010-2222-3333'
WHERE member_id=3;
```

```sql
SELECT * FROM member WHERE member_id=3;
```

```
# member_id	username	password	email		smartPhone	join_date		status	last_login		  age
3		user03	pass1234	user03@test.com	010-2222-3333	2026-07-31 12:20:42	ACTIVE	2026-07-31 12:20:42	  22
```

**EX19) 특정 회원의 상태값 변경**
- username이 'user10'인 회원의 status를 'INACTIVE' 로 변경

```sql
UPDATE member
SET status='INACTIVE'
WHERE username='user10';
```

```sql
SELECT * FROM member WHERE username='user10';
```

```
# member_id	username	password	email		smartPhone	join_date		status		last_login		  age
10		user10	pass1234	user10@test.com	010-1010-1010	2026-07-31 12:20:42	INACTIVE	2026-07-31 12:20:42	  29
```

**EX20) 나이가 35 미만인 회원의 상태를 전부 'YOUNG' 으로 변경**
- age가 35 미만인 모든 회원의 status를 'YOUNG' 으로 변경

```sql
UPDATE member
SET status = 'YOUNG'
WHERE age < 35;
```

```sql
SELECT * FROM member;
```

```
# member_id	username	password	email	smartPhone	join_date	status		last_login		 age
1	kim	pass1234	kim@example.com			2026-07-31 12:14:00	YOUNG			 18
2	user02	pass1234	user02@test.com	010-1002-1002	2026-07-31 12:20:42	YOUNG	2026-07-31 12:20:42	  21
3	user03	pass1234	user03@test.com	010-2222-3333	2026-07-31 12:20:42	YOUNG	2026-07-31 12:20:42	 22
4	user04	pass1234	user04@test.com	010-1004-1004	2026-07-31 12:20:42	YOUNG	2026-07-31 12:20:42	 23
5	user05	pass1234	user05@test.com	010-1005-1005	2026-07-31 12:20:42	YOUNG	2026-07-31 12:20:42	 24
6	user06	pass1234	user06@test.com	010-1006-1006	2026-07-31 12:20:42	YOUNG	2026-07-31 12:20:42	 25
7	user07	pass1234	user07@test.com	010-1007-1007	2026-07-31 12:20:42	YOUNG	2026-07-31 12:20:42	 26
8	user08	pass1234	user08@test.com	010-1008-1008	2026-07-31 12:20:42	YOUNG	2026-07-31 12:20:42	 27
9	user09	pass1234	user09@test.com	010-1009-1009	2026-07-31 12:20:42	YOUNG	2026-07-31 12:20:42	 28
10	user10	pass1234	user10@test.com	010-1010-1010	2026-07-31 12:20:42	YOUNG	2026-07-31 12:20:42	 29
11	user11	pass5678	user11@test.com	010-1011-1011	2026-07-31 12:20:42	YOUNG	2026-07-31 12:20:42	 30
12	user12	pass5678	user12@test.com	010-1012-1012	2026-07-31 12:20:42	YOUNG	2026-07-31 12:20:42	 31
13	user13	pass5678	user13@test.com	010-1013-1013	2026-07-31 12:20:42	YOUNG			 32
14	user14	pass5678	user14@test.com	010-1014-1014	2026-07-31 12:20:42	YOUNG	2026-07-31 12:20:42	 33
15	user15	pass5678	user15@test.com	010-1015-1015	2026-07-31 12:20:42	YOUNG			 34
16	user16	linux1234	user16@test.com	010-1016-1016	2026-07-31 12:20:42	ACTIVE	2026-07-31 12:20:42	 35
17	user17	linux1234	user17@test.com	010-1017-1017	2026-07-31 12:20:42	ACTIVE	2026-07-31 12:20:42	 36
18	user18	linux1234	user18@test.com	010-1018-1018	2026-07-31 12:20:42	INACTIVE		 37
19	user19	maria1234	user19@test.com	010-1019-1019	2026-07-31 12:20:42	ACTIVE	2026-07-31 12:20:42	 38
20	user20	maria1234	user20@test.com	010-1020-1020	2026-07-31 12:20:42	ACTIVE	2026-07-31 12:20:42	 39
```

**정리**: **UPDATE ... SET ... WHERE**는 조건에 맞는 행의 특정 컬럼 값을 변경하며, WHERE 절 조건에 따라 단일 행 또는 다수 행을 한 번에 갱신할 수 있다.

### DELETE 실습 (EX21~EX22)

**EX21) 특정 회원 1명 삭제**
- member_id가 1인 회원을 삭제

```sql
DELETE FROM member WHERE member_id=1;
```

```sql
SELECT * FROM member;

SELECT * FROM member WHERE member_id=1;
```

**EX22) 상태가 INACTIVE인 회원 모두 삭제**
- status가 'INACTIVE' 인 회원을 모두 삭제

```sql
DELETE FROM member WHERE status='INACTIVE';
```

```sql
SELECT * FROM member;
```

```
# member_id	username	password	email		smartPhone	join_date	status		last_login		 age
2		user02	pass1234	user02@test.com	010-1002-1002	2026-07-31 12:20:42	YOUNG	2026-07-31 12:20:42	 21
3		user03	pass1234	user03@test.com	010-2222-3333	2026-07-31 12:20:42	YOUNG	2026-07-31 12:20:42	 22
4		user04	pass1234	user04@test.com	010-1004-1004	2026-07-31 12:20:42	YOUNG	2026-07-31 12:20:42	 23
5		user05	pass1234	user05@test.com	010-1005-1005	2026-07-31 12:20:42	YOUNG	2026-07-31 12:20:42	 24
6		user06	pass1234	user06@test.com	010-1006-1006	2026-07-31 12:20:42	YOUNG	2026-07-31 12:20:42	 25
7		user07	pass1234	user07@test.com	010-1007-1007	2026-07-31 12:20:42	YOUNG	2026-07-31 12:20:42	 26
8		user08	pass1234	user08@test.com	010-1008-1008	2026-07-31 12:20:42	YOUNG	2026-07-31 12:20:42	 27
9		user09	pass1234	user09@test.com	010-1009-1009	2026-07-31 12:20:42	YOUNG	2026-07-31 12:20:42	 28
10		user10	pass1234	user10@test.com	010-1010-1010	2026-07-31 12:20:42	YOUNG	2026-07-31 12:20:42	 29
11		user11	pass5678	user11@test.com	010-1011-1011	2026-07-31 12:20:42	YOUNG	2026-07-31 12:20:42	 30
12		user12	pass5678	user12@test.com	010-1012-1012	2026-07-31 12:20:42	YOUNG	2026-07-31 12:20:42	 31
13		user13	pass5678	user13@test.com	010-1013-1013	2026-07-31 12:20:42	YOUNG			 32
14		user14	pass5678	user14@test.com	010-1014-1014	2026-07-31 12:20:42	YOUNG	2026-07-31 12:20:42	 33
15		user15	pass5678	user15@test.com	010-1015-1015	2026-07-31 12:20:42	YOUNG			 34
16		user16	linux1234	user16@test.com	010-1016-1016	2026-07-31 12:20:42	ACTIVE	2026-07-31 12:20:42	 35
17		user17	linux1234	user17@test.com	010-1017-1017	2026-07-31 12:20:42	ACTIVE	2026-07-31 12:20:42	 36
19		user19	maria1234	user19@test.com	010-1019-1019	2026-07-31 12:20:42	ACTIVE	2026-07-31 12:20:42	 38
20		user20	maria1234	user20@test.com	010-1020-1020	2026-07-31 12:20:42	ACTIVE	2026-07-31 12:20:42	 39
```

**정리**: **DELETE FROM ... WHERE**는 조건에 맞는 행을 완전히 제거하며, WHERE 절을 생략하면 테이블의 모든 행이 삭제되므로 조건 지정에 각별히 주의해야 한다.

### 비교 연산자 추가 실습 (EX23~EX24)

**EX23) age가 30인 회원을 모두 조회**

```sql
SELECT * FROM member WHERE age=30;
```

```
# member_id	username	password	email		smartPhone	join_date	status	l	ast_login		 age
11		user11	pass5678	user11@test.com	010-1011-1011	2026-07-31 12:20:42	YOUNG	2026-07-31 12:20:42	 30
```

**EX24) age가 18이 아닌 회원만 조회**

```sql
SELECT * FROM member WHERE age!=30;

	OR

SELECT * FROM member WHERE age<>30;
```

```
# member_id	username	password	email		smartPhone	join_date	status		last_login		age
2		user02	pass1234	user02@test.com	010-1002-1002	2026-07-31 12:20:42	YOUNG	2026-07-31 12:20:42	21
3		user03	pass1234	user03@test.com	010-2222-3333	2026-07-31 12:20:42	YOUNG	2026-07-31 12:20:42	22
4		user04	pass1234	user04@test.com	010-1004-1004	2026-07-31 12:20:42	YOUNG	2026-07-31 12:20:42	23
5		user05	pass1234	user05@test.com	010-1005-1005	2026-07-31 12:20:42	YOUNG	2026-07-31 12:20:42	24
6		user06	pass1234	user06@test.com	010-1006-1006	2026-07-31 12:20:42	YOUNG	2026-07-31 12:20:42	25
7		user07	pass1234	user07@test.com	010-1007-1007	2026-07-31 12:20:42	YOUNG	2026-07-31 12:20:42	26
8		user08	pass1234	user08@test.com	010-1008-1008	2026-07-31 12:20:42	YOUNG	2026-07-31 12:20:42	27
9		user09	pass1234	user09@test.com	010-1009-1009	2026-07-31 12:20:42	YOUNG	2026-07-31 12:20:42	28
10		user10	pass1234	user10@test.com	010-1010-1010	2026-07-31 12:20:42	YOUNG	2026-07-31 12:20:42	29
12		user12	pass5678	user12@test.com	010-1012-1012	2026-07-31 12:20:42	YOUNG	2026-07-31 12:20:42	31
13		user13	pass5678	user13@test.com	010-1013-1013	2026-07-31 12:20:42	YOUNG			32
14		user14	pass5678	user14@test.com	010-1014-1014	2026-07-31 12:20:42	YOUNG	2026-07-31 12:20:42	33
15		user15	pass5678	user15@test.com	010-1015-1015	2026-07-31 12:20:42	YOUNG			34
16		user16	linux1234	user16@test.com	010-1016-1016	2026-07-31 12:20:42	ACTIVE	2026-07-31 12:20:42	35
17		user17	linux1234	user17@test.com	010-1017-1017	2026-07-31 12:20:42	ACTIVE	2026-07-31 12:20:42	36
19		user19	maria1234	user19@test.com	010-1019-1019	2026-07-31 12:20:42	ACTIVE	2026-07-31 12:20:42	38
20		user20	maria1234	user20@test.com	010-1020-1020	2026-07-31 12:20:42	ACTIVE	2026-07-31 12:20:42	39
```

**정리**: `!=`와 `<>`는 동일하게 "같지 않음"을 뜻하는 비교 연산자로, WHERE 절에서 특정 값을 제외한 나머지 행을 조회할 때 사용한다.

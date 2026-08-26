# EMP / DEPT 테이블 실습

## 목차

1. [테이블 스키마 (CREATE TABLE)](#테이블-스키마-create-table)
2. [실습 내용](#실습-내용)
3. [WHERE 절](#where-절)
4. [ORDER BY 정렬 기능](#order-by-정렬-기능)
5. [LIKE 검색 기능](#like-검색-기능)
6. [GROUP BY 개념](#group-by-개념)
7. [HAVING](#having)

## 테이블 스키마 (CREATE TABLE)

**EMP**(사원)와 **DEPT**(부서) 두 테이블을 생성하고 기본 샘플 데이터를 적재하는 스크립트이다. Oracle에서 전통적으로 쓰이던 emp/dept 구조를 그대로 옮겨온 것으로, JOIN·집계 함수·서브쿼리 같은 SQL 문법을 실습하는 표준 예제로 널리 쓰인다. **PRIMARY KEY**와 **FOREIGN KEY** 제약을 통해 두 테이블 간의 관계를 정의한다.

```sql
CREATE TABLE dept (
    deptno INT(2) NOT NULL,
    dname  VARCHAR(14),
    loc    VARCHAR(13),
    PRIMARY KEY (deptno)
);

CREATE TABLE emp (
    empno    INT(4) NOT NULL,
    ename    VARCHAR(10),
    job      VARCHAR(9),
    mgr      INT(4),
    hiredate DATE,
    sal      DECIMAL(7,2),
    comm     DECIMAL(7,2),
    deptno   INT(2),
    PRIMARY KEY (empno),
    FOREIGN KEY (deptno) REFERENCES dept(deptno)
);

INSERT INTO dept (deptno, dname, loc) VALUES (10, 'ACCOUNTING', 'NEW YORK');
INSERT INTO dept VALUES (20, 'RESEARCH', 'DALLAS');
INSERT INTO dept VALUES (30, 'SALES', 'CHICAGO');
INSERT INTO dept VALUES (40, 'OPERATIONS', 'BOSTON');

INSERT INTO emp VALUES
(7839, 'KING', 'PRESIDENT', NULL, '1981-11-17', 5000, NULL, 10);

INSERT INTO emp VALUES
(7698, 'BLAKE', 'MANAGER', 7839, '1981-05-01', 2850, NULL, 30);

INSERT INTO emp VALUES
(7782, 'CLARK', 'MANAGER', 7839, '1981-06-09', 2450, NULL, 10);

INSERT INTO emp VALUES
(7566, 'JONES', 'MANAGER', 7839, '1981-04-02', 2975, NULL, 20);

INSERT INTO emp VALUES
(7788, 'SCOTT', 'ANALYST', 7566,
 DATE_SUB('1987-07-13', INTERVAL 85 DAY),
 3000, NULL, 20);

INSERT INTO emp VALUES
(7902, 'FORD', 'ANALYST', 7566, '1981-12-03', 3000, NULL, 20);

INSERT INTO emp VALUES
(7369, 'SMITH', 'CLERK', 7902, '1980-12-17', 800, NULL, 20);

INSERT INTO emp VALUES
(7499, 'ALLEN', 'SALESMAN', 7698, '1981-02-20', 1600, 300, 30);

INSERT INTO emp VALUES
(7521, 'WARD', 'SALESMAN', 7698, '1981-02-22', 1250, 500, 30);

INSERT INTO emp VALUES
(7654, 'MARTIN', 'SALESMAN', 7698, '1981-09-28', 1250, 1400, 30);

INSERT INTO emp VALUES
(7844, 'TURNER', 'SALESMAN', 7698, '1981-09-08', 1500, 0, 30);

INSERT INTO emp VALUES
(7876, 'ADAMS', 'CLERK', 7788,
 DATE_SUB('1987-07-13', INTERVAL 51 DAY),
 1100, NULL, 20);

INSERT INTO emp VALUES
(7900, 'JAMES', 'CLERK', 7698, '1981-12-03', 950, NULL, 30);

INSERT INTO emp VALUES
(7934, 'MILLER', 'CLERK', 7782, '1982-01-23', 1300, NULL, 10);
```

---

## 실습 내용

**emp**(사원)와 **dept**(부서) 테이블을 실제로 생성하고, 기본 조회부터 연산, 별칭(AS), 문자열 연결까지 실습한다.

### 테이블 생성과 컬럼 설명

```sql
/* 부서 정보를 저장하는 테이블 */
CREATE TABLE dept (
    deptno	INT(2) NOT NULL,   			-- 부서 번호(정수 2자리)
    dname	VARCHAR(14),        			-- 부서명 (최대 14글자)
    loc    	VARCHAR(13),        			-- 부서 위치 (최대 13글자)
    CONSTRAINT pk_dept PRIMARY KEY (deptno)  	-- 기본키 설정(부서 번호)
);


/* 사원 정보를 저장하는 테이블 */
CREATE TABLE emp (
    empno		INT(4) NOT NULL,   		-- 사원 번호(4자리 정수)
    ename    	VARCHAR(10),          		-- 사원 이름(최대 10글자)
    job      	VARCHAR(9),           		-- 직무(최대 9글자)
    mgr     	INT(4),             			-- 직속 상관(empno 참조)
    hiredate	DATE,                 			-- 입사일
    sal      	DECIMAL(7,2),           		-- 급여(전체 7자리, 소수점 2자리)
    comm     	DECIMAL(7,2),        		-- 보너스(전체 7자리, 소수점 2자리)
    deptno   	INT(2),                 			-- 소속 부서 번호

    CONSTRAINT pk_emp PRIMARY KEY (empno), 	-- 기본키 설정
    CONSTRAINT fk_deptno FOREIGN KEY (deptno)  	-- 외래키 설정
        REFERENCES dept(deptno)                  		-- dept 테이블 deptno 참조
);
```

**emp** 테이블에 적재된 전체 데이터는 다음과 같다.

```
# empno	ename	job		mgr	hiredate		sal	comm	deptno
7369	SMITH	CLERK		7902	1980-12-17	800.00		20
7499	ALLEN	SALESMAN	7698	1981-02-20	1600.00	300.00	30
7521	WARD	SALESMAN	7698	1981-02-22	1250.00	500.00	30
7566	JONES	MANAGER	7839	1981-04-02	2975.00		20
7654	MARTIN	SALESMAN	7698	1981-09-28	1250.00	1400.00	30
7698	BLAKE	MANAGER	7839	1981-05-01	2850.00		30
7782	CLARK	MANAGER	7839	1981-06-09	2450.00		10
7788	SCOTT	ANALYST	7566	1987-04-19	3000.00		20
7839	KING	PRESIDENT		1981-11-17	5000.00		10
7844	TURNER	SALESMAN	7698	1981-09-08	1500.00	0.00	30
7876	ADAMS	CLERK		7788	1987-05-23	1100.00		20
7900	JAMES	CLERK		7698	1981-12-03	950.00		30
7902	FORD	ANALYST	7566	1981-12-03	3000.00		20
7934	MILLER	CLERK		7782	1982-01-23	1300.00		10
```

**dept** 테이블에 적재된 전체 데이터는 다음과 같다.

```
# deptno	dname		loc
10	ACCOUNTING	NEW YORK
20	RESEARCH	DALLAS
30	SALES		CHICAGO
40	OPERATIONS	BOSTON
```

### 기본 SELECT 조회

```sql
-- 테이블의 모든 컬럼을 검색
SELECT * FROM emp;


-- 테이블의 특정 컬럼을 검색
SELECT empno, ename, sal FROM emp;
```

```
# empno	ename	sal
7369	SMITH	800.00
7499	ALLEN	1600.00
7521	WARD	1250.00
7566	JONES	2975.00
7654	MARTIN	1250.00
7698	BLAKE	2850.00
7782	CLARK	2450.00
7788	SCOTT	3000.00
7839	KING	5000.00
7844	TURNER	1500.00
7876	ADAMS	1100.00
7900	JAMES	950.00
7902	FORD	3000.00
7934	MILLER	1300.00
```

### 연산 및 별칭(AS) 사용

**sal**과 같은 숫자 컬럼은 조회 시 사칙연산을 적용할 수 있다. 이때 테이블 안의 실제 값이 바뀌는 것은 아니며, 조회 결과에만 반영된다.

```sql
SELECT empno, ename, sal, sal * 12 FROM emp;
```

```
# empno	ename	sal	sal * 12
7369	SMITH	800.00	9600.00
7499	ALLEN	1600.00	19200.00
7521	WARD	1250.00	15000.00
7566	JONES	2975.00	35700.00
7654	MARTIN	1250.00	15000.00
7698	BLAKE	2850.00	34200.00
7782	CLARK	2450.00	29400.00
7788	SCOTT	3000.00	36000.00
7839	KING	5000.00	60000.00
7844	TURNER	1500.00	18000.00
7876	ADAMS	1100.00	13200.00
7900	JAMES	950.00	11400.00
7902	FORD	3000.00	36000.00
7934	MILLER	1300.00	15600.00
```

```sql
SELECT empno, ename, sal, (sal * 12) * 0.7 FROM emp;
```

```
# empno	ename	sal	(sal * 12) * 0.7
7369	SMITH	800.00	6720.000
7499	ALLEN	1600.00	13440.000
7521	WARD	1250.00	10500.000
7566	JONES	2975.00	24990.000
7654	MARTIN	1250.00	10500.000
7698	BLAKE	2850.00	23940.000
7782	CLARK	2450.00	20580.000
7788	SCOTT	3000.00	25200.000
7839	KING	5000.00	42000.000
7844	TURNER	1500.00	12600.000
7876	ADAMS	1100.00	9240.000
7900	JAMES	950.00	7980.000
7902	FORD	3000.00	25200.000
7934	MILLER	1300.00	10920.000
```

- 테이블의 정수 컬럼의 경우 연산이 가능하다.
- 테이블 안의 값이 변경되지는 않는다.(검색시에만 출력된다.)

**AS**를 사용하면 조회 결과 컬럼에 별칭(alias)을 지정할 수 있다.

```sql
SELECT 
    sal AS '월급', 
    sal * 0.7 AS '실수령액',
    sal * 12 AS '연봉' 
FROM emp;
```

```
# 월급	실수령액	연봉
800.00	560.000	9600.00
1600.00	1120.000	19200.00
1250.00	875.000	15000.00
2975.00	2082.500	35700.00
1250.00	875.000	15000.00
2850.00	1995.000	34200.00
2450.00	1715.000	29400.00
3000.00	2100.000	36000.00
5000.00	3500.000	60000.00
1500.00	1050.000	18000.00
1100.00	770.000	13200.00
950.00	665.000	11400.00
3000.00	2100.000	36000.00
1300.00	910.000	15600.00
```

### CONCAT 함수로 문자열 연결

문자 연결(MySQL/MariaDB에서는 **CONCAT** 함수 사용)

- **CONCAT 함수**: 여러 개의 문자열이나 컬럼 값을 하나의 문자열로 연결하는 함수. CONCAT은 concatenate의 약자로, 뜻은 문자열을 이어 붙인다는 의미이다.
- **기본 형식**: `CONCAT(값1, 값2, 값3, ...)` — 괄호 안에 작성한 값을 왼쪽부터 순서대로 연결한다.

```sql
SELECT CONCAT('Hellol', 'MariaDB');
```

```
# CONCAT('Hellol', 'MariaDB')
HellolMariaDB
```

```sql
SELECT ename, CONCAT(ename, '_USA') FROM emp;
```

```
# ename	CONCAT(ename, '_USA')
SMITH	SMITH_USA
ALLEN	ALLEN_USA
WARD	WARD_USA
JONES	JONES_USA
MARTIN	MARTIN_USA
BLAKE	BLAKE_USA
CLARK	CLARK_USA
SCOTT	SCOTT_USA
KING	KING_USA
TURNER	TURNER_USA
ADAMS	ADAMS_USA
JAMES	JAMES_USA
FORD	FORD_USA
MILLER	MILLER_USA
```

```sql
SELECT ename, CONCAT(ename, ' 사원') FROM emp;
```

```
# ename	CONCAT(ename, ' 사원')
SMITH	SMITH 사원
ALLEN	ALLEN 사원
WARD	WARD 사원
JONES	JONES 사원
MARTIN	MARTIN 사원
BLAKE	BLAKE 사원
CLARK	CLARK 사원
SCOTT	SCOTT 사원
KING	KING 사원
TURNER	TURNER 사원
ADAMS	ADAMS 사원
JAMES	JAMES 사원
FORD	FORD 사원
MILLER	MILLER 사원
```

```sql
-- 부서번호가 10번인 직원만 조회하고, ename 뒤에 '사원' 문자열을 붙여서 출력한다.

SELECT 
   ename, CONCAT(ename, '사원') , deptno
   FROM emp 
WHERE deptno = 10;
```

```
# ename	  CONCAT(ename, ' 사원')	deptno
CLARK	  CLARK 사원		10
KING	  KING 사원		10
MILLER	  MILLER 사원		10
```

```sql
-- 부서번호가 10 또는 20인 직원들을 조회하고, ename 뒤에 '사원' 를 붙여 출력한다.

SELECT ename, CONCAT(ename, '사원') 
FROM emp 
WHERE deptno = 10;
```

```
# ename	CONCAT(ename, ' 사원')
CLARK	CLARK 사원
KING	KING 사원
MILLER	MILLER 사원
```

```sql
-- 부서번호가 10이거나, 급여가 2500인 직원들을 조회하고 ename 뒤에 '사원' 를 붙여 출력한다.

SELECT ename, CONCAT(ename, '사원') 
FROM emp
WHERE deptno = 10 OR sal = 2500;
```

```
# ename	CONCAT(ename, ' 사원')
CLARK	CLARK 사원
KING	KING 사원
MILLER	MILLER 사원
```

```sql
-- 부서번호가 20이면서 급여가 2500보다 큰 직원만 조회한다.
-- sal은 그대로 출력하고 ename 뒤에 '사원' 를 붙여 출력한다.

SELECT sal, CONCAT(ename, '사원')
FROM emp
WHERE deptno = 20 AND sal > 2500;
```

```
# sal	CONCAT(ename, '사원')
2975.00	JONES사원
3000.00	SCOTT사원
3000.00	FORD사원
```

**정리**: emp/dept 테이블을 생성하고 기본 SELECT, 산술 연산, **AS** 별칭, **CONCAT** 문자열 연결 함수를 실습했다. 이 예제들은 이후 WHERE, ORDER BY, GROUP BY 실습의 기반이 된다.

---

## WHERE 절

**WHERE** 절은 테이블에서 조건에 맞는 행만 선택하여 조회, 수정, 삭제할 때 사용하는 조건절이다.

- WHERE는 SELECT, UPDATE, DELETE 명령어와 함께 사용할 수 있다.

기본 형식:

```sql
SELECT 컬럼명
FROM 테이블명
WHERE 조건;
```

예:

```sql
SELECT *
FROM emp
WHERE sal >= 3000;
```

- 의미: emp 테이블에서 급여가 3000 이상인 사원의 모든 정보를 조회한다.

```sql
-- empno 컬럼의 모든 레코드를 검색하여 7782가 있는 레코드를 출력한다. (매칭 안 되면 아무것도 출력되지 않는다.)

SELECT * FROM emp WHERE empno=7782;
```

```
# empno	ename	job		mgr	hiredate		sal	comm	deptno
7782	CLARK	MANAGER	7839	1981-06-09	2450.00		10
```

```sql
SELECT empno, ename, sal  FROM emp WHERE empno=7782;
```

```
# empno	ename	sal
7782	CLARK	2450.00
```

```sql
-- 대/소 비교도 가능하다.
SELECT * FROM emp WHERE sal >= 3000;		-- 월급이 3000이상
```

```
# empno	ename	job		mgr	hiredate		sal	comm	deptno
7788	SCOTT	ANALYST	7566	1987-04-19	3000.00		20
7839	KING	PRESIDENT		1981-11-17	5000.00		10
7902	FORD	ANALYST	7566	1981-12-03	3000.00		20
```

```sql
SELECT * FROM emp
WHERE sal >= 2000 AND sal < 3000;		-- 월급이 2천만원대인 사원 정보 출력
```

```
# empno	ename	job		mgr	hiredate		sal	comm	deptno
7566	JONES	MANAGER	7839	1981-04-02	2975.00		20
7698	BLAKE	MANAGER	7839	1981-05-01	2850.00		30
7782	CLARK	MANAGER	7839	1981-06-09	2450.00		10
```

```sql
SELECT * FROM emp
WHERE NOT (sal >= 2000 AND sal < 3000);	-- 월급이 2천만원대인 사원을 제외한 나머지 사원 정보만 출력
```

```
# empno	ename	job		mgr	hiredate		sal	comm	deptno
7369	SMITH	CLERK		7902	1980-12-17	800.00		20
7499	ALLEN	SALESMAN	7698	1981-02-20	1600.00	300.00	30
7521	WARD	SALESMAN	7698	1981-02-22	1250.00	500.00	30
7654	MARTIN	SALESMAN	7698	1981-09-28	1250.00	1400.00	30
7788	SCOTT	ANALYST	7566	1987-04-19	3000.00		20
7839	KING	PRESIDENT		1981-11-17	5000.00		10
7844	TURNER	SALESMAN	7698	1981-09-08	1500.00	0.00	30
7876	ADAMS	CLERK		7788	1987-05-23	1100.00		20
7900	JAMES	CLERK		7698	1981-12-03	950.00		30
7902	FORD	ANALYST	7566	1981-12-03	3000.00		20
7934	MILLER	CLERK		7782	1982-01-23	1300.00		10
```

```sql
SELECT * FROM emp
WHERE job!='SALESMAN';                   	-- job이 SALESMAN이 아닌 사원정보만 출력
```

```
# empno	ename	job		mgr	hiredate		sal	comm	deptno
7369	SMITH	CLERK		7902	1980-12-17	800.00		20
7566	JONES	MANAGER	7839	1981-04-02	2975.00		20
7698	BLAKE	MANAGER	7839	1981-05-01	2850.00		30
7782	CLARK	MANAGER	7839	1981-06-09	2450.00		10
7788	SCOTT	ANALYST	7566	1987-04-19	3000.00		20
7839	KING	PRESIDENT		1981-11-17	5000.00		10
7876	ADAMS	CLERK		7788	1987-05-23	1100.00		20
7900	JAMES	CLERK		7698	1981-12-03	950.00		30
7902	FORD	ANALYST	7566	1981-12-03	3000.00		20
7934	MILLER	CLERK		7782	1982-01-23	1300.00		10
```

```sql
-- 81년 12월 31일 이후 입사 (MySQL에서는 DATE 리터럴 그대로 사용)

SELECT * FROM emp
WHERE hiredate > '1981-12-31'
```

```
# empno	ename	job		mgr	hiredate		sal	comm	deptno
7788	SCOTT	ANALYST	7566	1987-04-19	3000.00		20
7876	ADAMS	CLERK		7788	1987-05-23	1100.00		20
7934	MILLER	CLERK		7782	1982-01-23	1300.00		10
```

**정리**: **WHERE** 절은 행 단위 조건 필터링에 사용하며, 비교 연산자(`>=`, `!=`), 논리 연산자(AND, OR, NOT), 날짜 비교 등을 조합해 원하는 행만 선택할 수 있다.

---

## ORDER BY 정렬 기능

**ORDER BY**는 조회된 데이터를 특정 컬럼을 기준으로 오름차순 또는 내림차순으로 정렬하는 절이다.

- ORDER BY는 일반적으로 SQL문의 마지막 부분에 작성한다.

```sql
SELECT 컬럼명
FROM 테이블명
ORDER BY 정렬기준;
```

### 오름차순 정렬: ASC

**ASC**는 Ascending의 약자이며 작은 값부터 큰 값 순서로 정렬한다.

```sql
SELECT *
FROM emp
ORDER BY sal ASC;
```

- 급여가 낮은 사원부터 높은 사원 순서로 출력
- ASC는 기본값이므로 생략할 수 있다.

### 내림차순 정렬: DESC

**DESC**는 Descending의 약자이며 큰 값부터 작은 값 순서로 정렬한다.

```sql
SELECT *
FROM emp
ORDER BY sal DESC;
```

- 급여가 높은 사원부터 낮은 사원 순서로 출력

### 실습 예제

EX1) emp 테이블의 모든 데이터를 사원 번호가 낮은 순서로 조회

```sql
SELECT * FROM emp ORDER BY empno ASC;
```

```
# empno	ename	job		mgr	hiredate		sal	comm	deptno
7369	SMITH	CLERK		7902	1980-12-17	800.00		20
7499	ALLEN	SALESMAN	7698	1981-02-20	1600.00	300.00	30
7521	WARD	SALESMAN	7698	1981-02-22	1250.00	500.00	30
7566	JONES	MANAGER	7839	1981-04-02	2975.00		20
7654	MARTIN	SALESMAN	7698	1981-09-28	1250.00	1400.00	30
7698	BLAKE	MANAGER	7839	1981-05-01	2850.00		30
7782	CLARK	MANAGER	7839	1981-06-09	2450.00		10
7788	SCOTT	ANALYST	7566	1987-04-19	3000.00		20
7839	KING	PRESIDENT		1981-11-17	5000.00		10
7844	TURNER	SALESMAN	7698	1981-09-08	1500.00	0.00	30
7876	ADAMS	CLERK		7788	1987-05-23	1100.00		20
7900	JAMES	CLERK		7698	1981-12-03	950.00		30
7902	FORD	ANALYST	7566	1981-12-03	3000.00		20
7934	MILLER	CLERK		7782	1982-01-23	1300.00		10
```

EX2) emp 테이블의 모든 데이터를 사원 번호가 높은 순서로 조회

```sql
SELECT * FROM emp ORDER BY empno DESC;
```

```
# empno	ename	job		mgr	hiredate		sal	comm	deptno
7934	MILLER	CLERK		7782	1982-01-23	1300.00		10
7902	FORD	ANALYST	7566	1981-12-03	3000.00		20
7900	JAMES	CLERK		7698	1981-12-03	950.00		30
7876	ADAMS	CLERK		7788	1987-05-23	1100.00		20
7844	TURNER	SALESMAN	7698	1981-09-08	1500.00	0.00	30
7839	KING	PRESIDENT		1981-11-17	5000.00		10
7788	SCOTT	ANALYST	7566	1987-04-19	3000.00		20
7782	CLARK	MANAGER	7839	1981-06-09	2450.00		10
7698	BLAKE	MANAGER	7839	1981-05-01	2850.00		30
7654	MARTIN	SALESMAN	7698	1981-09-28	1250.00	1400.00	30
7566	JONES	MANAGER	7839	1981-04-02	2975.00		20
7521	WARD	SALESMAN	7698	1981-02-22	1250.00	500.00	30
7499	ALLEN	SALESMAN	7698	1981-02-20	1600.00	300.00	30
7369	SMITH	CLERK		7902	1980-12-17	800.00		20
```

EX3) 사원 이름과 직무를 사원 이름의 알파벳 오름차순으로 조회하시오

```sql
SELECT ename, job FROM emp ORDER BY ename ASC;
```

```
# ename	job
ADAMS	CLERK
ALLEN	SALESMAN
BLAKE	MANAGER
CLARK	MANAGER
FORD	ANALYST
JAMES	CLERK
JONES	MANAGER
KING	PRESIDENT
MARTIN	SALESMAN
MILLER	CLERK
SCOTT	ANALYST
SMITH	CLERK
TURNER	SALESMAN
WARD	SALESMAN
```

EX4) 사원 이름과 급여를 사원 이름의 알파벳 내림차순으로 조회하시오

```sql
SELECT ename, sal FROM emp ORDER BY ename DESC;
```

```
# ename	sal
WARD	1250.00
TURNER	1500.00
SMITH	800.00
SCOTT	3000.00
MILLER	1300.00
MARTIN	1250.00
KING	5000.00
JONES	2975.00
JAMES	950.00
FORD	3000.00
CLARK	2450.00
BLAKE	2850.00
ALLEN	1600.00
ADAMS	1100.00
```

EX5) 사원 이름, 부서 번호, 급여를 부서 번호가 낮은 순서로 정렬하고, 같은 부서에서는 급여가 높은 순서로 조회

```sql
SELECT ename, deptno, sal 
FROM emp 
ORDER BY deptno ASC, sal DESC;
```

```
# ename	deptno	sal
KING	10	5000.00
CLARK	10	2450.00
MILLER	10	1300.00
FORD	20	3000.00
SCOTT	20	3000.00
JONES	20	2975.00
ADAMS	20	1100.00
SMITH	20	800.00
BLAKE	30	2850.00
ALLEN	30	1600.00
TURNER	30	1500.00
MARTIN	30	1250.00
WARD	30	1250.00
JAMES	30	950.00
```

EX6) 급여가 2000 미만인 사원의 사원 번호, 이름, 급여를 급여가 높은 순서로 조회

```sql
SELECT empno, ename, sal
FROM emp
WHERE sal < 2000
ORDER BY sal DESC;
```

```
# empno	ename	sal
7499	ALLEN	1600.00
7844	TURNER	1500.00
7934	MILLER	1300.00
7521	WARD	1250.00
7654	MARTIN	1250.00
7876	ADAMS	1100.00
7900	JAMES	950.00
7369	SMITH	800.00
```

EX7) 사원 이름, 직무, 월급, 연봉, 연봉 실수령액을 출력하고 실수령액이 낮은 순서로 조회 (sal이 월급이라고 간주, 실수령액은 70%)

```sql
SELECT 
   ename, 
   job, 
   sal AS salary,
   sal * 12 AS annual_salary_total,
   (sal * 12) * 0.7 AS annual_salary
FROM emp
ORDER BY annual_salary ASC;
```

```
# ename	job		salary	annual_salary_total	annual_salary
SMITH	CLERK		800.00	9600.00		6720.000
JAMES	CLERK		950.00	11400.00		7980.000
ADAMS	CLERK		1100.00	13200.00		9240.000
WARD	SALESMAN	1250.00	15000.00		10500.000
MARTIN	SALESMAN	1250.00	15000.00		10500.000
MILLER	CLERK		1300.00	15600.00		10920.000
TURNER	SALESMAN	1500.00	18000.00		12600.000
ALLEN	SALESMAN	1600.00	19200.00		13440.000
CLARK	MANAGER	2450.00	29400.00		20580.000
BLAKE	MANAGER	2850.00	34200.00		23940.000
JONES	MANAGER	2975.00	35700.00		24990.000
FORD	ANALYST	3000.00	36000.00		25200.000
SCOTT	ANALYST	3000.00	36000.00		25200.000
KING	PRESIDENT	5000.00	60000.00		42000.000
```

**정리**: **ORDER BY**는 **ASC**(오름차순, 기본값)와 **DESC**(내림차순)를 지원하며, 여러 컬럼을 콤마로 나열하면 다중 정렬 기준(1차, 2차 정렬)을 적용할 수 있다.

---

## LIKE 검색 기능

**LIKE**는 문자열 데이터에서 특정 문자나 문자열이 포함된 값을 검색할 때 사용하는 연산자이다.

- 일반적으로 **WHERE** 절과 함께 사용한다.

```sql
SELECT 컬럼명
FROM 테이블명
WHERE 컬럼명 LIKE '검색패턴';
```

```sql
SELECT * FROM emp;
SELECT * FROM emp WHERE ename LIKE 'S%';			-- S로 시작
SELECT * FROM emp WHERE ename LIKE '%S';			-- S로 로 끝나는
SELECT * FROM emp WHERE ename LIKE '%S%';		-- S가 포함된
SELECT * FROM emp WHERE ename LIKE '%SC%';		-- SC가 포함된
SELECT * FROM emp WHERE ename LIKE '%S%C%';		-- S, C가 포함된

SELECT * FROM emp WHERE ename LIKE '_C';			-- 두번째 문자가 C인 (2글자)
SELECT * FROM emp WHERE ename LIKE '_C%';			-- 두번째 문자가 C인 (글자수 상관 X)
SELECT * FROM emp WHERE ename LIKE '__C%';		-- 세번째 문자가 C인 (글자수 상관 X)

SELECT * FROM emp WHERE ename NOT LIKE 'S%';		-- S로 시작하지 않는
SELECT * FROM emp WHERE ename NOT LIKE '%S%';		-- S가 포함되지 않은

SELECT * FROM emp 
WHERE ename LIKE 'A%' OR ename LIKE '%N';			-- A로 시작하거나 N으로 끝나는

SELECT * FROM emp WHERE LOWER(ename) LIKE 'S%';		-- 대/소문자 상관없이 s로 시작하는
```

- `%`는 임의 길이의 문자열(0개 이상)을 의미하고, `_`는 정확히 한 글자를 의미한다.

### LIKE 실습용 테이블 생성

**product_catalog**라는 실습 전용 테이블을 만들어 다양한 **LIKE** 패턴 매칭을 연습한다.

```sql
CREATE TABLE product_catalog (
    product_id   	INT PRIMARY KEY,             	-- 상품 번호
    product_code 	VARCHAR(20) NOT NULL,	-- 상품 코드
    product_name 	VARCHAR(100) NOT NULL,	-- 상품명
    brand        	VARCHAR(50) NOT NULL,    	-- 브랜드
    category     	VARCHAR(30) NOT NULL,  	-- 상품 분류
    model_name   	VARCHAR(50),                 	-- 모델명
    color        	VARCHAR(30),                 	-- 색상
    description  	VARCHAR(200)                 	-- 상품 설명
);

INSERT INTO product_catalog VALUES
(1, 'NB-SAM-001', 'Samsung Galaxy Book', 'Samsung',
 'Notebook', 'GalaxyBook4', 'Silver', 'Lightweight office notebook');

INSERT INTO product_catalog VALUES
(2, 'NB-LEN-002', 'Lenovo ThinkPad X1', 'Lenovo',
 'Notebook', 'ThinkPad_X1', 'Black', 'Business notebook with strong security');

INSERT INTO product_catalog VALUES
(3, 'NB-APP-003', 'Apple MacBook Air', 'Apple',
 'Notebook', 'MacBookAir_M3', 'Midnight', 'Slim notebook for students');

INSERT INTO product_catalog VALUES
(4, 'NB-ASU-004', 'ASUS ROG Gaming Laptop', 'ASUS',
 'Notebook', 'ROG_Strix_G16', 'Gray', 'High performance gaming notebook');

INSERT INTO product_catalog VALUES
(5, 'PH-SAM-101', 'Samsung Galaxy S25', 'Samsung',
 'SmartPhone', 'Galaxy_S25', 'Blue', 'Latest Galaxy smart phone');

INSERT INTO product_catalog VALUES
(6, 'PH-APP-102', 'Apple iPhone 17 Pro', 'Apple',
 'SmartPhone', 'iPhone17_Pro', 'Black', 'Premium smart phone with pro camera');

INSERT INTO product_catalog VALUES
(7, 'PH-GOO-103', 'Google Pixel Phone', 'Google',
 'SmartPhone', 'Pixel_10', 'White', 'AI camera smart phone');

INSERT INTO product_catalog VALUES
(8, 'PH-XIA-104', 'Xiaomi Redmi Note', 'Xiaomi',
 'SmartPhone', 'Redmi_Note_15', 'Green', 'Affordable smart phone');

INSERT INTO product_catalog VALUES
(9, 'MN-LG-201', 'LG UltraWide Monitor', 'LG',
 'Monitor', 'UltraWide_34', 'Black', 'Wide screen monitor for office');

INSERT INTO product_catalog VALUES
(10, 'MN-SAM-202', 'Samsung Odyssey Monitor', 'Samsung',
 'Monitor', 'Odyssey_G7', 'Black', 'Curved gaming monitor');

INSERT INTO product_catalog VALUES
(11, 'MN-DEL-203', 'Dell UltraSharp Display', 'Dell',
 'Monitor', 'UltraSharp_U27', 'Silver', 'Professional color display');

INSERT INTO product_catalog VALUES
(12, 'KB-LOG-301', 'Logitech Wireless Keyboard', 'Logitech',
 'Keyboard', 'MX_Keys', 'Graphite', 'Quiet wireless keyboard');

INSERT INTO product_catalog VALUES
(13, 'KB-RAZ-302', 'Razer Mechanical Keyboard', 'Razer',
 'Keyboard', 'BlackWidow_V4', 'Black', 'RGB mechanical gaming keyboard');

INSERT INTO product_catalog VALUES
(14, 'KB-APP-303', 'Apple Magic Keyboard', 'Apple',
 'Keyboard', 'Magic_Keyboard', 'White', 'Compact wireless keyboard');

INSERT INTO product_catalog VALUES
(15, 'MS-LOG-401', 'Logitech Silent Mouse', 'Logitech',
 'Mouse', 'Silent_M650', 'White', 'Silent wireless office mouse');

INSERT INTO product_catalog VALUES
(16, 'MS-RAZ-402', 'Razer Gaming Mouse', 'Razer',
 'Mouse', 'DeathAdder_V3', 'Black', 'Fast RGB gaming mouse');

INSERT INTO product_catalog VALUES
(17, 'HD-SEA-501', 'Seagate External Hard Drive', 'Seagate',
 'Storage', 'Backup_Plus_5TB', 'Black', 'Portable backup storage');

INSERT INTO product_catalog VALUES
(18, 'SD-SAM-502', 'Samsung Portable SSD', 'Samsung',
 'Storage', 'T7_Shield', 'Blue', 'Fast portable solid state drive');

INSERT INTO product_catalog VALUES
(19, 'NW-CIS-601', 'Cisco Wireless Router', 'Cisco',
 'Network', 'Cisco_Router_AX', 'Black', 'Secure wireless network router');

INSERT INTO product_catalog VALUES
(20, 'NW-TPL-602', 'TP-Link WiFi Router', 'TP-Link',
 'Network', 'Archer_AX80', 'Black', 'High speed home WiFi router');

INSERT INTO product_catalog VALUES
(21, 'AC-USB-701', 'USB-C Multi Hub', 'Baseus',
 'Accessory', 'USB_C_HUB', 'Gray', 'USB hub with HDMI and LAN ports');

INSERT INTO product_catalog VALUES
(22, 'AC-CAB-702', 'Premium HDMI Cable', 'Ugreen',
 'Accessory', 'HDMI_2_1', 'Black', 'Supports 8K 120Hz display');

INSERT INTO product_catalog VALUES
(23, 'SP-JBL-801', 'JBL Bluetooth Speaker', 'JBL',
 'Speaker', 'Flip_7', 'Red', 'Portable waterproof speaker');

INSERT INTO product_catalog VALUES
(24, 'SP-SON-802', 'Sony Smart Speaker', 'Sony',
 'Speaker', 'Smart_Speaker_X', 'White', 'Voice controlled home speaker');

INSERT INTO product_catalog VALUES
(25, 'EV-SAL-901', 'Summer 30% Sale Package', 'EventShop',
 'Event', 'SALE_30_PERCENT', 'Mixed', 'Special 30% discount product');

INSERT INTO product_catalog VALUES
(26, 'EV-SET-902', 'Office_Set Package', 'EventShop',
 'Event', 'OFFICE_SET', 'Mixed', 'Notebook mouse and keyboard package');
```

### LIKE 실습 문제 (EX1 ~ EX18)

EX1) product_catalog 테이블에서 상품명이 Samsung으로 시작하는 상품의 모든 정보를 조회

```sql
SELECT *
FROM product_catalog
WHERE product_name LIKE 'Samsung%';
```

EX2) 상품명이 Keyboard로 끝나는 상품의 상품 번호, 상품명, 브랜드를 조회

```sql
SELECT product_id, product_name, brand
FROM product_catalog
WHERE product_name LIKE '%Keyboard';
```

EX3) 상품명에 Gaming이 포함된 상품의 상품명, 카테고리, 설명을 조회

```sql
SELECT product_name, category, description
FROM product_catalog
WHERE product_name LIKE '%Gaming%';
```

EX4) 상품 설명에 wireless가 포함된 상품을 조회

```sql
SELECT product_name, description
FROM product_catalog
WHERE description LIKE '%wireless%';
```

EX5) 상품 코드가 NB-로 시작하는 상품의 상품 코드, 상품명, 브랜드를 조회

```sql
SELECT product_code, product_name, brand
FROM product_catalog
WHERE product_code LIKE 'NB-%';
```

EX6) 상품 코드가 02를 포함하는 상품을 조회

```sql
SELECT product_code, product_name
FROM product_catalog
WHERE product_code LIKE '%02%';
```

EX7) 브랜드명이 S로 시작하는 상품을 조회

```sql
SELECT product_name, brand
FROM product_catalog
WHERE brand LIKE 'S%';
```

EX8) 브랜드명이 g로 끝나는 상품을 조회

```sql
SELECT product_name, brand
FROM product_catalog
WHERE brand LIKE '%g';
```

EX9) 브랜드명에 o가 포함된 상품을 조회

```sql
SELECT product_name, brand
FROM product_catalog
WHERE brand LIKE '%o%';
```

EX10) 카테고리가 Note로 시작하는 상품의 상품명과 모델명을 조회

```sql
SELECT product_name, model_name
FROM product_catalog
WHERE category LIKE 'Note%';
```

EX11) 상품명에 Smart가 포함되거나 상품 설명에 smart가 포함된 상품을 조회

```sql
SELECT product_name, description
FROM product_catalog
WHERE product_name LIKE '%Smart%'
OR description LIKE '%smart%';
```

EX12) 상품명이 Apple로 시작하면서 Keyboard로 끝나는 상품을 조회

```sql
SELECT product_name, brand, category
FROM product_catalog
WHERE product_name LIKE 'Apple%'
AND product_name LIKE '%Keyboard';
```

EX13) 상품명에 Samsung이 포함되지 않은 상품을 조회

```sql
SELECT product_id, product_name, brand
FROM product_catalog
WHERE product_name NOT LIKE '%Samsung%';
```

EX14) 상품 설명이 Portable로 시작하지 않는 상품을 조회

```sql
SELECT product_name, description
FROM product_catalog
WHERE description NOT LIKE 'Portable%';
```

EX15) 상품 코드에서 세 번째 문자가 -인 상품을 조회

```sql
SELECT product_code, product_name
FROM product_catalog
WHERE product_code LIKE '__-%';
```

EX16) 브랜드명이 정확히 세 글자인 상품을 조회

```sql
SELECT product_name, brand
FROM product_catalog
WHERE brand LIKE '___';
```

EX17) 브랜드명의 두 번째 문자가 p인 상품을 조회

```sql
SELECT product_name, brand
FROM product_catalog
WHERE brand LIKE '_p%';
```

EX18) 다음 조건을 모두 만족하는 상품의 상품명, 브랜드, 카테고리, 설명을 조회
 - 상품 설명에 office 또는 business가 포함
 - 상품명이 Apple로 시작하지 않음
 - 검색 결과를 상품명 오름차순으로 정렬

```sql
SELECT product_name, brand, category, description
FROM product_catalog
WHERE (
description LIKE '%office%'
OR description LIKE '%business%'
)
AND product_name NOT LIKE 'Apple%'
ORDER BY product_name ASC;
```

**정리**: **LIKE**는 `%`(임의 길이 와일드카드)와 `_`(단일 문자 와일드카드)를 조합해 부분 문자열 검색을 수행하며, **NOT LIKE**, **AND/OR**와 결합해 복합 패턴 조건도 구성할 수 있다.

---

## GROUP BY 개념

**GROUP BY**는 테이블의 데이터를 특정 컬럼 값에 따라 묶어서 각 그룹별로 집계 결과(합계, 평균, 최대, 최소, 개수 등)를 계산하는 기능이다. 즉, 동일한 값을 가진 행들을 하나의 그룹으로 만들어 그 그룹 단위로 통계 계산을 할 때 사용한다.

- 부서별 평균 급여
- 직무별 최대 급여
- 연도별 매출 합계

**GROUP BY가 필요한 이유**
- 많은 데이터를 그룹 단위로 분석하고 비교하기 위해
- 집계 함수(SUM, AVG, COUNT, MAX, MIN)와 함께 사용하기 위해
- 테이블 전체가 아닌 특정 기준별 통계를 얻기 위해
- 데이터의 패턴이나 요약 정보를 얻기 위해

**GROUP BY 처리 순서**
1. FROM : 테이블 읽기
2. WHERE : 조건에 맞지 않는 행 제거
3. GROUP BY : 조건을 통과한 행들을 그룹으로 묶음
4. HAVING : 그룹 결과를 필터링
5. SELECT : 필요한 컬럼만 출력
6. ORDER BY : 정렬

**집계 함수와 함께 사용하는 방식**
- `SUM()` 합계
- `AVG()` 평균
- `MAX()` 최대값
- `MIN()` 최소값
- `COUNT()` 개수

### 제약 사항

- SELECT 절에는 두 종류의 값만 올 수 있다
  - GROUP BY에 적은 컬럼
  - 집계함수(SUM, AVG, COUNT, MAX, MIN)
- WHERE에는 집계함수를 사용할 수 없다. 즉, 집계 결과에 조건을 걸고 싶으면 **HAVING**을 사용해야 한다.

### GROUP BY 실습 예제

EX1) 부서별(deptno) 사원 수를 구하시오.

```sql
SELECT deptno, count(*)
FROM emp
GROUP BY deptno;
```

```
# deptno	count(*)
10	3
20	5
30	6
```

- deptno 기준으로 그룹을 만들고 해당 그룹의 행 개수만큼 COUNT 한다.

EX2) 직무(job)별 최고 급여(MAX(sal))를 구하시오.

```sql
SELECT job, MAX(sal)
FROM emp
GROUP BY job;
```

```
# job		MAX(sal)
ANALYST	3000.00
CLERK		1300.00
MANAGER	2975.00
PRESIDENT	5000.00
SALESMAN	1600.00
```

- job 기준으로 그룹을 만들고 sal 컬럼 중 가장 큰 값을 구한다.

EX3) 직무(job)별 평균 급여를 구하시오.

```sql
SELECT job, AVG(sal)
FROM emp
GROUP BY job;
```

```
# job		AVG(sal)
ANALYST	3000.000000
CLERK		1037.500000
MANAGER	2758.333333
PRESIDENT	5000.000000
SALESMAN	1400.000000
```

EX4) 부서별 급여 총합(SUM(sal))을 구하고 부서번호 기준으로 정렬하시오.

```sql
SELECT deptno, SUM(sal)
FROM emp
GROUP BY deptno
ORDER BY deptno;
```

```
# deptno	SUM(sal)
10	8750.00
20	10875.00
30	9400.00
```

```sql
SELECT deptno, COUNT(*), SUM(sal), AVG(sal)
FROM emp
GROUP BY deptno
ORDER BY deptno;
```

```
# deptno	COUNT(*)	SUM(sal)	AVG(sal)
10	3		8750.00	2916.666667
20	5		10875.00	2175.000000
30	6		9400.00	1566.666667
```

EX6) 부서별(deptno) 평균 급여를 구하되, 급여 1000 이상인 사원만 대상으로 계산하시오

```sql
SELECT deptno, AVG(sal) AS sal_avg
FROM emp
WHERE sal >= 1000
GROUP BY deptno;
```

```
# deptno	sal_avg
10	2916.666667
20	2518.750000
30	1690.000000
```

**정리**: **GROUP BY**는 지정한 컬럼 값이 같은 행들을 그룹으로 묶고, SUM/AVG/MAX/MIN/COUNT 같은 집계 함수와 함께 사용해 그룹별 통계를 계산한다. SELECT 절에는 GROUP BY 컬럼과 집계 함수만 올 수 있으며, 집계 결과에 조건을 걸려면 WHERE가 아닌 HAVING을 사용해야 한다.

---

## HAVING

EX1) 부서별(deptno) 평균 급여가 2000 이상인 부서만 조회

```sql
SELECT deptno,  AVG(sal)
FROM emp
WHERE AVG(sal) >= 2000
GROUP BY deptno;
```

- 오류 이유: **AVG(sal)**은 그룹이 만들어진 후에 계산되는데 **WHERE**는 그룹 이전 단계에서 실행되므로 사용 불가능하다.

**HAVING**은 그룹(GROUP BY 결과)에 조건을 거는 절이다. **WHERE**는 그룹 만들기 전에 개별 행(Row)을 필터링하지만, **HAVING**은 그룹이 만들어진 후 그룹 자체에 조건을 건다.

**차이점**
- `WHERE` : 행 조건
- `HAVING` : 그룹 조건(집계 결과 조건)

- 특히 집계함수(SUM, AVG, MAX, MIN, COUNT)는 WHERE 조건에서 사용할 수 없다.
- 집계함수는 그룹이 만들어진 뒤에야 계산되기 때문이다. 그래서 집계함수 조건은 반드시 HAVING 에서 사용해야 한다.

### HAVING 실습 예제

EX1) 부서별(deptno) 평균 급여가 2000 이상인 부서만 조회

```sql
SELECT deptno,  AVG(sal)
FROM emp
GROUP BY deptno
HAVING AVG(sal) >= 2000;
```

```
# deptno	AVG(sal)
10	2916.666667
20	2175.000000
```

EX2) 부서별(deptno) 급여 합계(SUM)가 9000 이상인 부서를 조회

```sql
SELECT deptno,  SUM(sal) AS total_sal
FROM emp
GROUP BY deptno
HAVING SUM(sal) >= 9000;
```

```
# deptno	total_sal
20	10875.00
30	9400.00
```

EX3) 직무(job)별 최대 급여(MAX)가 3000 이상인 직무를 조회

```sql
SELECT job, MAX(sal) AS max_sal
FROM emp
GROUP BY job
HAVING MAX(sal) >= 3000;
```

```
# job	max_sal
ANALYST	3000.00
PRESIDENT	5000.00
```

EX4) 직무(job)별 평균 급여(AVG(sal))가 1500 이상인 직무를 조회하고 평균 급여의 내림차순 정렬 하시오.

```sql
SELECT job, AVG(sal) AS avg_sal
FROM emp
GROUP BY job
HAVING AVG(sal) >= 1500
ORDER BY avg_sal DESC;
```

```
# job		avg_sal
PRESIDENT	5000.000000
ANALYST	3000.000000
MANAGER	2758.333333
```

EX5) 사원(emp) 테이블에서 부서(deptno)별 사원 수(COUNT)를 계산하고, 사원 수가 4명 이상인 부서만 조회

```sql
SELECT deptno, COUNT(*) AS deptno_cnt
FROM emp
GROUP BY deptno
HAVING COUNT(*) >= 4;
```

```
# deptno	deptno_cnt
20	5
30	6
```

EX6) 사원(emp) 테이블에서 직무(job)별 최저 급여(MIN)를 계산하고, 최저 급여가 1000 이상인 직무만 조회

```sql
SELECT job, MIN(sal) AS  min_sal
FROM emp
GROUP BY job
HAVING MIN(sal) >= 1000;
```

```
# job		min_sal
ANALYST	3000.00
MANAGER	2450.00
PRESIDENT	5000.00
SALESMAN	1250.00
```

EX7) 사원(emp) 테이블에서 직무(job)별 급여 합계(SUM)와 사원 수(COUNT)를 계산하고, 급여 합계가 5000 이상이면서 사원 수가 2명 이상인 직무만 조회

```sql
SELECT job, 
    SUM(sal) AS total_sal,
    COUNT(*) AS job_cnt
FROM emp
GROUP BY job
HAVING SUM(sal) >= 5000
    AND COUNT(*) >= 2;
```

```
# job		total_sal	job_cnt
ANALYST	6000.00	2
MANAGER	8275.00	3
SALESMAN	5600.00	4
```

EX8) 급여(sal)가 1500 이상인 사원만 대상으로 부서(deptno)별 평균 급여(AVG)를 계산하고, 평균 급여가 2500 이상인 부서를 조회

```sql
SELECT deptno ,  AVG(sal) AS avg_sal
FROM emp
WHERE sal >= 1500
GROUP BY deptno
HAVING AVG(sal) >= 2500;
```

```
# deptno	avg_sal
10	3725.000000
20	2991.666667
```

EX9) 사원(emp) 테이블에서 부서(deptno)별 평균 급여(AVG)를 계산하고, 전체 사원의 평균 급여보다 높은 부서만 조회

```sql
SELECT deptno ,  AVG(sal) AS avg_sal
FROM emp
GROUP BY deptno
HAVING AVG(sal) > (
	SELECT AVG(sal) FROM emp
);
```

```
# deptno	avg_sal
10	2916.666667
20	2175.000000
```

처리 순서:
1. 서브쿼리 실행 : `SELECT AVG(sal) FROM emp` (전체 사원의 평균 급여 계산)
2. 메인 쿼리의 FROM 실행 : `FROM emp`
3. 부서(deptno)별 그룹 생성 : `GROUP BY deptno`
4. 부서별 평균 급여 계산 : `AVG(sal)`
5. HAVING 조건 비교 : 부서별 평균 급여 > 전체 평균 급여
6. 조건을 만족하는 부서만 선택
7. SELECT 결과 출력 : `deptno, AVG(sal) AS avg_sal`

```sql
SELECT deptno ,  AVG(sal) AS avg_sal ,
    (SELECT AVG(sal) FROM emp) AS all_total_sal
FROM emp
GROUP BY deptno
HAVING AVG(sal) > (
	SELECT AVG(sal) FROM emp
);
```

```
# deptno	avg_sal		all_total_sal
10	2916.666667	2073.214286
20	2175.000000	2073.214286
```

**정리**: **HAVING**은 **GROUP BY**로 만들어진 그룹 자체에 조건을 거는 절로, 집계 함수 조건은 WHERE가 아닌 HAVING에서만 사용할 수 있다. 서브쿼리를 HAVING 조건에 결합하면 "전체 평균보다 높은 그룹"과 같은 동적인 비교도 가능하다.

# 데이터베이스 실습 문제

> SELECT, WHERE, JOIN, GROUP BY, 서브쿼리 실습 문제

## emp/dept 테이블 기반 실습

### SELECT & WHERE 실습

```sql
-- EX1) 모든 직원의 이름, 직무, 급여를 조회
SELECT ename, job, sal FROM emp;

-- EX2) 급여가 3000 이상인 직원 조회
SELECT * FROM emp WHERE sal >= 3000;

-- EX3) 부서 30번 직원 중 급여가 1500 이상인 직원
SELECT ename, deptno, sal FROM emp
WHERE deptno = 30 AND sal >= 1500;

-- EX4) 직무가 MANAGER 또는 ANALYST인 직원
SELECT ename, job FROM emp
WHERE job = 'MANAGER' OR job = 'ANALYST';
-- 또는
SELECT ename, job FROM emp WHERE job IN ('MANAGER', 'ANALYST');

-- EX5) 1981년에 입사한 직원
SELECT ename, hiredate FROM emp
WHERE hiredate BETWEEN '1981-01-01' AND '1981-12-31';

-- EX6) comm(보너스)이 NULL이 아닌 직원
SELECT ename, sal, comm FROM emp WHERE comm IS NOT NULL;

-- EX7) 이름이 S로 시작하는 직원
SELECT ename FROM emp WHERE ename LIKE 'S%';

-- EX8) 이름의 세번째 글자가 I인 직원
SELECT ename FROM emp WHERE ename LIKE '__I%';
```

### ORDER BY 실습

```sql
-- EX1) 급여가 낮은 사원부터 높은 순으로 이름과 급여 조회
SELECT ename, sal FROM emp ORDER BY sal ASC;

-- EX2) 부서번호 오름차순, 같은 부서 내 급여 내림차순
SELECT ename, deptno, sal FROM emp
ORDER BY deptno ASC, sal DESC;

-- EX3) 이름의 알파벳 역순으로 정렬
SELECT ename FROM emp ORDER BY ename DESC;
```

### 집계함수 + GROUP BY 실습

```sql
-- EX1) 전체 사원 수
SELECT COUNT(*) FROM emp;                    -- 14명

-- EX2) 전체 평균 급여
SELECT AVG(sal) FROM emp;                    -- 2073.21

-- EX3) 최고 급여와 최저 급여
SELECT MAX(sal), MIN(sal) FROM emp;          -- 5000, 800

-- EX4) 부서별 사원 수
SELECT deptno, COUNT(*) AS 사원수
FROM emp GROUP BY deptno;

-- EX5) 직무별 급여 합계 (내림차순)
SELECT job, SUM(sal) AS 급여합계
FROM emp GROUP BY job
ORDER BY 급여합계 DESC;

-- EX6) 부서별 최고/최저/평균 급여
SELECT deptno,
       MAX(sal) AS 최고,
       MIN(sal) AS 최저,
       AVG(sal) AS 평균
FROM emp
GROUP BY deptno;
```

### HAVING 실습

```sql
-- EX1) 평균 급여가 2000 이상인 부서
SELECT deptno, AVG(sal) AS avg_sal
FROM emp GROUP BY deptno
HAVING AVG(sal) >= 2000;

-- EX2) 사원 수가 4명 이상인 부서
SELECT deptno, COUNT(*) AS cnt
FROM emp GROUP BY deptno
HAVING COUNT(*) >= 4;

-- EX3) 급여 합계가 가장 큰 부서
SELECT deptno, SUM(sal) AS total
FROM emp GROUP BY deptno
ORDER BY total DESC LIMIT 1;
```

### INNER JOIN 실습

```sql
-- EX1) 모든 사원의 이름, 부서명, 부서 위치
SELECT e.ename, d.dname, d.loc
FROM emp e
INNER JOIN dept d ON e.deptno = d.deptno;

-- EX2) DALLAS 부서 소속 사원
SELECT e.ename, d.dname, d.loc
FROM emp e
INNER JOIN dept d ON e.deptno = d.deptno
WHERE d.loc = 'DALLAS';

-- EX3) 부서별 사원 수와 부서명 (JOIN + GROUP BY)
SELECT d.dname, COUNT(e.empno) AS 사원수
FROM dept d
LEFT JOIN emp e ON d.deptno = e.deptno
GROUP BY d.deptno, d.dname;

-- EX4) 급여가 평균보다 높은 사원과 부서명
SELECT e.ename, d.dname, e.sal
FROM emp e
INNER JOIN dept d ON e.deptno = d.deptno
WHERE e.sal > (SELECT AVG(sal) FROM emp)
ORDER BY e.sal DESC;
```

---

## LIKE 실습 (product_catalog 테이블)

```sql
-- EX1) Samsung으로 시작하는 상품
SELECT * FROM product_catalog
WHERE product_name LIKE 'Samsung%';

-- EX2) Keyboard로 끝나는 상품의 번호, 이름, 브랜드
SELECT product_id, product_name, brand
FROM product_catalog
WHERE product_name LIKE '%Keyboard';

-- EX3) Gaming이 포함된 상품
SELECT product_name, category, description
FROM product_catalog
WHERE product_name LIKE '%Gaming%';

-- EX4) NB-로 시작하는 상품 코드
SELECT product_code, product_name, brand
FROM product_catalog
WHERE product_code LIKE 'NB-%';

-- EX5) Apple로 시작하면서 Keyboard로 끝나는 상품
SELECT product_name, brand
FROM product_catalog
WHERE product_name LIKE 'Apple%'
AND product_name LIKE '%Keyboard';

-- EX6) 브랜드명이 정확히 3글자인 상품
SELECT product_name, brand
FROM product_catalog
WHERE brand LIKE '___';
```

---

## 종합 실습 문제

### 문제 1: 부서별 통계

emp/dept 테이블을 이용하여:
- 각 부서의 부서명, 위치, 사원 수, 평균 급여를 조회
- 평균 급여 내림차순으로 정렬

```sql
SELECT d.dname, d.loc,
       COUNT(e.empno) AS 사원수,
       ROUND(AVG(e.sal), 2) AS 평균급여
FROM dept d
LEFT JOIN emp e ON d.deptno = e.deptno
GROUP BY d.deptno, d.dname, d.loc
ORDER BY 평균급여 DESC;
```

### 문제 2: 직무별 급여 분석

```sql
-- 직무별 최고/최저/평균 급여 및 사원 수
-- 평균 급여가 전체 평균보다 높은 직무만 출력
SELECT job,
       COUNT(*) AS 사원수,
       MAX(sal) AS 최고급여,
       MIN(sal) AS 최저급여,
       ROUND(AVG(sal), 2) AS 평균급여
FROM emp
GROUP BY job
HAVING AVG(sal) > (SELECT AVG(sal) FROM emp)
ORDER BY 평균급여 DESC;

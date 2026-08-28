# DB 05 — INNER JOIN

## 목차

1. [JOIN](#join)
2. [INNER JOIN](#inner-join)
3. [INNER JOIN이 필요한 이유](#inner-join이-필요한-이유)
4. [INNER JOIN 기본 문법](#inner-join-기본-문법)
5. [실습용 테이블 - customer](#실습용-테이블---customer)
6. [실습용 테이블 - orders](#실습용-테이블---orders)
7. [INNER JOIN 실습 (EX1~EX12)](#inner-join-실습-ex1ex12)

## JOIN

- 두 개 이상 테이블을 연결해서 필요한 데이터를 조회하는 기능이다.
- 예: 회원 테이블(member) + 주문 테이블(order)을 합쳐서 회원 이름 + 주문 상품명을 동시에 보여줄 때 JOIN을 사용한다.
- JOIN을 사용할 때 가장 중요한 것은 두 테이블을 서로 연결할 기준(컬럼)을 지정하는 것이다.
  - 예: `member.member_id = orders.member_id`

**정리**: JOIN은 여러 테이블에 나뉘어 저장된 데이터를 연결 기준 컬럼으로 묶어 함께 조회하는 기능이다. 이어서 JOIN의 대표적인 방식인 INNER JOIN을 살펴본다.

---

## INNER JOIN

- INNER JOIN은 두 개 이상의 테이블을 연결하여, 조건이 일치하는 행만 조회하는 JOIN 방식이다.
- 두 테이블을 합칠 때 둘 다 공통된 값을 가진 행을 사용한다.

예: member 테이블과 orders 테이블을 INNER JOIN
- 주문한 회원 → 결과에 표시됨
- 주문하지 않은 회원 → 결과에서 제외됨

즉, INNER JOIN은 양쪽 테이블에 모두 존재하는 데이터만 보여주는 필터이다.

**정리**: INNER JOIN은 두 테이블 모두에 공통으로 존재하는 값을 가진 행만 결과로 보여주는 JOIN 방식이다. 이어서 INNER JOIN이 왜 필요한지 살펴본다.

---

## INNER JOIN이 필요한 이유

- 데이터는 보통 여러 테이블에 나뉘어 저장된다.
  - 예: 쇼핑몰 DB
    - 회원 정보는 member 테이블
    - 주문 정보는 orders 테이블
    - 상품 정보는 product 테이블
  - 이 데이터를 "한 화면"에 보여주기 위해 JOIN이 필요하다.

EX) "회원 이름 + 주문 날짜 + 주문 상품명"
- 이건 어떤 한 테이블만으로는 절대 조회할 수 없다.
- 필요한 정보가 여러 테이블에 흩어져 있기 때문이다.
- 그래서 JOIN이 반드시 필요하다.

INNER JOIN은 다음 조건에 따라 결과가 결정된다.
- A. 두 테이블을 연결할 기준(일치 조건)이 필요하다.
- B. 기준이 일치하는 행만 결과에 나온다.
- C. 기준 값이 한쪽에만 있으면 결과에서 제거된다.

예: `ON member.member_id = orders.member_id`

- member에 있고 orders에도 있는 member_id만 결과에 표시됨
- orders에는 있지만 member에 없다? → 결과 없음
- member에는 있지만 주문한 적이 없다? → 결과 없음

즉, 두 테이블 모두 "참여해야" 결과가 나온다.

**정리**: 여러 테이블에 흩어진 정보를 한 화면에 보여주려면 JOIN이 반드시 필요하며, INNER JOIN은 연결 기준이 양쪽 모두에 존재하는 행만 결과로 남긴다. 이어서 INNER JOIN의 기본 문법을 살펴본다.

---

## INNER JOIN 기본 문법

INNER JOIN 기본 문법 (MySQL)

형식:

```sql
SELECT   컬럼들
FROM 테이블A AS A
INNER JOIN 테이블B AS B
    ON A.공통컬럼 = B.공통컬럼;
```

- AS는 생략 가능.
- JOIN 앞에는 어떤 조건도 쓸 수 없음(조건은 반드시 ON 절에서 작성)

예제) 회원 이름 + 주문 상품 조회

```sql
SELECT m.username, o.product_name
FROM member AS m
INNER JOIN orders AS o
    ON m.member_id = o.member_id;
```

- member_id가 서로 일치하는 행만 출력
- 주문한 회원만 표시됨
- 주문 기록이 없는 회원은 결과에서 제외됨

예제 2) 여러 조건으로 JOIN

```sql
SELECT m.username, o.product_name, o.order_date
FROM member m
INNER JOIN orders o
    ON m.member_id = o.member_id
    AND o.status = 'PAID';
```

- 회원 정보 + 결제 완료(PAID) 주문만 조회
- ORDER 조건도 JOIN 조건 안에 넣을 수 있음

**정리**: INNER JOIN은 `FROM 테이블A INNER JOIN 테이블B ON 연결조건` 형태로 작성하며, JOIN 조건은 반드시 ON 절에 작성하고 AND로 여러 조건을 추가할 수 있다. 이어서 실습에 사용할 customer, orders 테이블을 만든다.

---

## 실습용 테이블 - customer

customer 테이블 (고객 기본 정보 저장)
- `customer_id` : 로그인 시 사용하는 실제 사용자 ID
- `name` : 고객 실명
- `gender` : 성별(M/F)
- `age` : 나이
- `phone` : 휴대폰 번호
- `city` : 거주 도시
- `join_date` : 회원 가입일자

```sql
CREATE TABLE customer (
    customer_id VARCHAR(30) PRIMARY KEY,  -- 로그인용 사용자 ID
    name VARCHAR(50),           -- 고객 실명
    gender ENUM('M','F'),       -- 성별(M/F)
    age INT,                    -- 나이
    phone VARCHAR(20),          -- 휴대폰 번호
    city VARCHAR(50),           -- 거주 도시
    join_date DATE              -- 가입일자
);
```

```sql
INSERT INTO customer VALUES
('minsu25', '김민수', 'M', 25, '010-2001-0001', '서울', '2023-01-05'),
('seoyeon88', '이서연', 'F', 31, '010-2001-0002', '부산', '2023-01-10'),
('jihoon29', '박지훈', 'M', 29, '010-2001-0003', '대구', '2023-02-12'),
('yurichoi', '최유리', 'F', 40, '010-2001-0004', '인천', '2023-03-01'),
('dohyun22', '정도현', 'M', 22, '010-2001-0005', '서울', '2023-03-03'),
('yerin_h', '한예린', 'F', 35, '010-2001-0006', '광주', '2023-04-22'),
('seojoon7', '윤서준', 'M', 28, '010-2001-0007', '부산', '2023-05-20'),
('hanul33', '오하늘', 'F', 33, '010-2001-0008', '대전', '2023-06-25'),
('minho_k', '강민호', 'M', 27, '010-2001-0009', '서울', '2023-07-15'),
('jieun24', '이지은', 'F', 24, '010-2001-0010', '인천', '2023-08-18'),
('jun_h42', '서준혁', 'M', 42, '010-2001-0011', '부산', '2023-09-23'),
('gayoung31', '문가영', 'F', 31, '010-2001-0012', '서울', '2023-10-01'),
('dohyun26', '김도현', 'M', 26, '010-2001-0013', '대구', '2023-10-07'),
('yuna_l', '이유나', 'F', 38, '010-2001-0014', '광주', '2023-10-15'),
('junwoo15', '박준우', 'M', 29, '010-2001-0015', '부산', '2023-11-01'),
('seulgi23', '강슬기', 'F', 23, '010-2001-0016', '서울', '2023-11-03'),
('jinwoo45', '김진우', 'M', 45, '010-2001-0017', '대전', '2023-11-07'),
('arinlee', '이아린', 'F', 33, '010-2001-0018', '인천', '2023-11-12'),
('nakhyun', '박낙현', 'M', 36, '010-2001-0019', '부산', '2023-11-18'),
('hayeon30', '최하연', 'F', 30, '010-2001-0020', '광주', '2023-11-20'),
('jiwon28', '김지원', 'M', 28, '010-2001-0021', '서울', '2023-11-25'),
('soyoung41', '정소영', 'F', 41, '010-2001-0022', '부산', '2023-11-27'),
('seok33', '오석진', 'M', 34, '010-2001-0023', '대구', '2023-11-29'),
('hyemi21', '박혜미', 'F', 21, '010-2001-0024', '서울', '2023-12-01'),
('taemin32', '이태민', 'M', 32, '010-2001-0025', '광주', '2023-12-05'),
('yeseo29', '문예서', 'F', 29, '010-2001-0026', '부산', '2023-12-07'),
('yoonsik37', '조윤식', 'M', 37, '010-2001-0027', '대전', '2023-12-09'),
('jiwoo43', '서지우', 'F', 43, '010-2001-0028', '서울', '2023-12-11'),
('geon22', '최건우', 'M', 22, '010-2001-0029', '부산', '2023-12-15'),
('serin39', '홍세린', 'F', 39, '010-2001-0030', '광주', '2023-12-20');
```

```sql
SELECT * FROM customer;
```

**정리**: customer 테이블은 customer_id를 기본키로 하는 고객 30명의 기본 정보(이름/성별/나이/전화번호/거주 도시/가입일)를 담는다. 이어서 이 고객들의 주문 정보를 담는 orders 테이블을 만든다.

---

## 실습용 테이블 - orders

orders 테이블 (고객 주문 정보 저장)
- `order_id` : 주문 번호(PK)
- `customer_id` : 주문한 고객의 ID (customer 테이블의 PK)
- `product` : 구매한 상품명
- `category` : 상품 카테고리(IT기기, 생활용품 등)
- `amount` : 결제 금액
- `order_status` : 주문 상태 (READY(결제 대기) / PAID(결제 완료) / CANCEL(주문 취소))
- `order_date` : 주문 날짜

```sql
CREATE TABLE orders (
    order_id INT PRIMARY KEY,                      -- 주문 번호
    customer_id VARCHAR(30),                       -- 고객 ID
    product VARCHAR(50),                           -- 상품명
    category VARCHAR(50),                          -- 카테고리
    amount INT,                                    -- 결제 금액
    order_status ENUM('READY','PAID','CANCEL'),    -- 주문 상태
    order_date DATE,                               -- 주문 날짜
    FOREIGN KEY (customer_id) REFERENCES customer(customer_id)
);
```

```sql
INSERT INTO orders VALUES
(1, 'minsu25', '키보드', 'IT기기', 45000, 'PAID', '2024-01-05'),
(2, 'minsu25', '마우스', 'IT기기', 25000, 'PAID', '2024-01-10'),
(3, 'seoyeon88', '모니터', 'IT기기', 210000, 'PAID', '2024-02-12'),
(4, 'jihoon29', 'USB허브', 'IT기기', 15000, 'READY', '2024-03-01'),
(5, 'jihoon29', '외장하드', 'IT기기', 98000, 'PAID', '2024-03-03'),
(6, 'yurichoi', '노트북', 'IT기기', 1250000, 'PAID', '2024-01-22'),
(7, 'dohyun22', '키보드', 'IT기기', 45000, 'PAID', '2024-02-20'),
(8, 'yerin_h', '마우스패드', 'IT기기', 12000, 'READY', '2024-02-25'),
(9, 'yerin_h', 'USB케이블', 'IT기기', 8000, 'PAID', '2024-03-10'),
(10, 'seojoon7', '스피커', 'IT기기', 67000, 'PAID', '2024-01-15'),
(11, 'hanul33', '헤드셋', 'IT기기', 99000, 'PAID', '2024-02-17'),
(12, 'hanul33', '키보드', 'IT기기', 45000, 'PAID', '2024-03-22'),
(13, 'minho_k', '웹캠', 'IT기기', 55000, 'PAID', '2024-03-25'),
(14, 'jieun24', '노트북', 'IT기기', 1320000, 'PAID', '2024-02-14'),
(15, 'jieun24', '마우스', 'IT기기', 25000, 'PAID', '2024-02-15'),
(16, 'jun_h42', '스탠드', '생활용품', 30000, 'READY', '2024-03-11'),
(17, 'gayoung31', '키보드', 'IT기기', 45000, 'PAID', '2024-02-01'),
(18, 'gayoung31', '마우스', 'IT기기', 25000, 'PAID', '2024-02-02'),
(19, 'dohyun26', '블루투스 이어폰', 'IT기기', 89000, 'PAID', '2024-01-07'),
(20, 'dohyun26', '스피커', 'IT기기', 67000, 'PAID', '2024-01-09'),
(21, 'yuna_l', 'USB허브', 'IT기기', 15000, 'READY', '2024-03-15'),
(22, 'junwoo15', '모니터', 'IT기기', 210000, 'PAID', '2024-02-10'),
(23, 'seulgi23', '키보드', 'IT기기', 45000, 'PAID', '2024-01-18'),
(24, 'jinwoo45', '헤드셋', 'IT기기', 99000, 'PAID', '2024-03-18'),
(25, 'arinlee', 'USB케이블', 'IT기기', 8000, 'READY', '2024-03-21'),
(26, 'arinlee', '마우스패드', 'IT기기', 12000, 'PAID', '2024-03-22'),
(27, 'nakhyun', '웹캠', 'IT기기', 55000, 'PAID', '2024-02-11'),
(28, 'hayeon30', '스피커', 'IT기기', 67000, 'PAID', '2024-01-12'),
(29, 'hayeon30', '키보드', 'IT기기', 45000, 'PAID', '2024-01-13'),
(30, 'jiwon28', '노트북', 'IT기기', 1290000, 'READY', '2024-03-01'),
(31, 'soyoung41', '모니터암', 'IT기기', 89000, 'PAID', '2024-03-04'),
(32, 'seok33', '책상조명', '생활용품', 35000, 'PAID', '2024-03-05'),
(33, 'hyemi21', 'USB허브', 'IT기기', 15000, 'READY', '2024-02-20'),
(34, 'taemin32', '키보드', 'IT기기', 45000, 'PAID', '2024-03-09'),
(35, 'taemin32', '마우스', 'IT기기', 25000, 'PAID', '2024-03-10'),
(36, 'taemin32', '노트북받침대', '생활용품', 27000, 'PAID', '2024-03-11'),
(37, 'yeseo29', '스탠드', '생활용품', 30000, 'PAID', '2024-02-18'),
(38, 'yoonsik37', '블루투스 스피커', 'IT기기', 56000, 'READY', '2024-03-12'),
(39, 'jiwoo43', '가습기', '생활용품', 42000, 'PAID', '2024-01-25'),
(40, 'geon22', '헤드셋', 'IT기기', 99000, 'PAID', '2024-02-03'),
(41, 'serin39', '분무기', '생활용품', 7000, 'PAID', '2024-03-05'),
(42, 'jieun24', 'USB C타입 젠더', 'IT기기', 9000, 'PAID', '2024-03-08'),
(43, 'minho_k', '장패드', '생활용품', 16000, 'CANCEL', '2024-02-22'),
(44, 'junwoo15', '소형 선풍기', '생활용품', 19000, 'READY', '2024-03-10'),
(45, 'minsu25', '마우스패드', 'IT기기', 12000, 'PAID', '2024-02-14'),
(46, 'jihoon29', '스마트폰 거치대', '생활용품', 11000, 'READY', '2024-02-21'),
(47, 'seoyeon88', '노트북 받침대', '생활용품', 27000, 'PAID', '2024-02-28'),
(48, 'nakhyun', '키보드 루프', 'IT기기', 9000, 'PAID', '2024-03-02'),
(49, 'geon22', '웹캠 가림막', '기타', 5000, 'READY', '2024-03-15'),
(50, 'gayoung31', '스마트워치 충전기', 'IT기기', 29000, 'PAID', '2024-03-18');
```

```sql
SELECT * FROM orders;
```

**정리**: orders 테이블은 order_id를 기본키로 하고, customer_id로 customer 테이블을 참조하는 외래키를 가진 주문 50건을 담는다. 이어서 이 두 테이블을 INNER JOIN으로 조회하는 실습(EX1~EX12)을 진행한다.

---

## INNER JOIN 실습 (EX1~EX12)

실습에서 다시 사용하는 테이블 정의:

```sql
CREATE TABLE customer (
    customer_id VARCHAR(30) PRIMARY KEY,  -- 로그인용 사용자 ID
    name VARCHAR(50),           -- 고객 실명
    gender ENUM('M','F'),       -- 성별(M/F)
    age INT,                    -- 나이
    phone VARCHAR(20),          -- 휴대폰 번호
    city VARCHAR(50),           -- 거주 도시
    join_date DATE              -- 가입일자
);

CREATE TABLE orders (
    order_id INT PRIMARY KEY,                      -- 주문 번호
    customer_id VARCHAR(30),                       -- 고객 ID
    product VARCHAR(50),                           -- 상품명
    category VARCHAR(50),                          -- 카테고리
    amount INT,                                    -- 결제 금액
    order_status ENUM('READY','PAID','CANCEL'),    -- 주문 상태
    order_date DATE,                               -- 주문 날짜
    FOREIGN KEY (customer_id) REFERENCES customer(customer_id)
);
```

### EX1) 고객 이름과 주문 상품명 조회

고객 이름(name)과 주문 상품명(product)을 INNER JOIN으로 조회하시오.

```sql
SELECT c.name, o.product
FROM customer c
INNER JOIN orders o
    ON  c.customer_id = o.customer_id;
```

| name | product |
|------|---------|
| 김민수 | 키보드 |
| 김민수 | 마우스 |
| 이서연 | 모니터 |
| 박지훈 | USB허브 |
| 박지훈 | 외장하드 |
| 최유리 | 노트북 |
| 정도현 | 키보드 |
| 한예린 | 마우스패드 |
| 한예린 | USB케이블 |
| 윤서준 | 스피커 |
| ~~~~~ | 중간 생략 |
| 박준우 | 소형 선풍기 |
| 김민수 | 마우스패드 |
| 박지훈 | 스마트폰 거치대 |
| 이서연 | 노트북 받침대 |
| 박낙현 | 키보드 루프 |
| 최건우 | 웹캠 가림막 |
| 문가영 | 스마트워치 충전기 |

### EX2) 고객 이름, 상품명, 주문 날짜 조회

고객 이름(name), 상품명(product), 주문 날짜(order_date)를 INNER JOIN으로 조회

```sql
SELECT c.name, o.product, o.order_date
FROM customer c
INNER JOIN orders o
    ON c.customer_id = o.customer_id;
```

| name | product | order_date |
|------|---------|------------|
| 김민수 | 키보드 | 2024-01-05 |
| 김민수 | 마우스 | 2024-01-10 |
| 이서연 | 모니터 | 2024-02-12 |
| 박지훈 | USB허브 | 2024-03-01 |
| 박지훈 | 외장하드 | 2024-03-03 |
| 최유리 | 노트북 | 2024-01-22 |
| 정도현 | 키보드 | 2024-02-20 |
| ~~~~~ | 중간 생략 | |
| 이지은 | USB C타입 젠더 | 2024-03-08 |
| 강민호 | 장패드 | 2024-02-22 |
| 박준우 | 소형 선풍기 | 2024-03-10 |
| 김민수 | 마우스패드 | 2024-02-14 |
| 박지훈 | 스마트폰 거치대 | 2024-02-21 |
| 이서연 | 노트북 받침대 | 2024-02-28 |
| 박낙현 | 키보드 루프 | 2024-03-02 |
| 최건우 | 웹캠 가림막 | 2024-03-15 |
| 문가영 | 스마트워치 충전기 | 2024-03-18 |

### EX3) 서울 거주 고객의 주문 조회

고객(customer) 테이블의 거주 도시(customer.city)가 서울인 고객의 이름(customer.name), 주문 상품명(orders.product), 주문일(orders.order_date)을 조회

```sql
SELECT c.name,
       o.product,
       o.order_date
FROM customer c
INNER JOIN orders o
    ON c.customer_id = o.customer_id
WHERE c.city = '서울';
```

| name | product | order_date |
|------|---------|------------|
| 김민수 | 키보드 | 2024-01-05 |
| 김민수 | 마우스 | 2024-01-10 |
| 정도현 | 키보드 | 2024-02-20 |
| 강민호 | 웹캠 | 2024-03-25 |
| 문가영 | 키보드 | 2024-02-01 |
| 문가영 | 마우스 | 2024-02-02 |
| 강슬기 | 키보드 | 2024-01-18 |
| 김지원 | 노트북 | 2024-03-01 |
| 박혜미 | USB허브 | 2024-02-20 |
| 서지우 | 가습기 | 2024-01-25 |
| 강민호 | 장패드 | 2024-02-22 |
| 김민수 | 마우스패드 | 2024-02-14 |
| 문가영 | 스마트워치 충전기 | 2024-03-18 |

```sql
SELECT c.name,
       c.city,
       o.product,
       o.order_date
FROM customer c
INNER JOIN orders o
    ON c.customer_id = o.customer_id
WHERE c.city = '서울';
```

| name | city | product | order_date |
|------|------|---------|------------|
| 김민수 | 서울 | 키보드 | 2024-01-05 |
| 김민수 | 서울 | 마우스 | 2024-01-10 |
| 정도현 | 서울 | 키보드 | 2024-02-20 |
| 강민호 | 서울 | 웹캠 | 2024-03-25 |
| 문가영 | 서울 | 키보드 | 2024-02-01 |
| 문가영 | 서울 | 마우스 | 2024-02-02 |
| 강슬기 | 서울 | 키보드 | 2024-01-18 |
| 김지원 | 서울 | 노트북 | 2024-03-01 |
| 박혜미 | 서울 | USB허브 | 2024-02-20 |
| 서지우 | 서울 | 가습기 | 2024-01-25 |
| 강민호 | 서울 | 장패드 | 2024-02-22 |
| 김민수 | 서울 | 마우스패드 | 2024-02-14 |
| 문가영 | 서울 | 스마트워치 충전기 | 2024-03-18 |

### EX4) 결제 완료(PAID) 주문 조회

주문(orders) 테이블의 주문 상태(orders.order_status)가 'PAID'인 주문에 대해 고객 이름(customer.name), 주문 상품명(orders.product), 결제 금액(orders.amount)을 조회

```sql
SELECT c.name,
       o.product,
       o.amount
FROM customer c
INNER JOIN orders o
    ON c.customer_id = o.customer_id
WHERE o.order_status = 'PAID';
```

| name | product | amount |
|------|---------|--------|
| 김민수 | 키보드 | 45000 |
| 김민수 | 마우스 | 25000 |
| 이서연 | 모니터 | 210000 |
| 박지훈 | 외장하드 | 98000 |
| 최유리 | 노트북 | 1250000 |
| 정도현 | 키보드 | 45000 |
| ~~~~~ | 중간 생략 | |
| 서지우 | 가습기 | 42000 |
| 최건우 | 헤드셋 | 99000 |
| 홍세린 | 분무기 | 7000 |
| 이지은 | USB C타입 젠더 | 9000 |
| 김민수 | 마우스패드 | 12000 |
| 이서연 | 노트북 받침대 | 27000 |
| 박낙현 | 키보드 루프 | 9000 |
| 문가영 | 스마트워치 충전기 | 29000 |

- 'PAID'는 결제가 완료된 주문 상태이다.
- 개별 주문 상태에 조건을 지정하므로 WHERE를 사용한다.

### EX5) 취소되지 않은 주문 조회

주문(orders) 테이블의 주문 상태(orders.order_status)가 CANCEL이 아닌 주문에 대해 고객 이름(customer.name), 주문 상품명(orders.product), 주문 상태(orders.order_status)를 조회

```sql
SELECT c.name,
       o.product,
       o.amount
FROM customer c
INNER JOIN orders o
    ON c.customer_id = o.customer_id
WHERE o.order_status <> 'CANCEL';
```

| name | product | amount |
|------|---------|--------|
| 김민수 | 키보드 | 45000 |
| 김민수 | 마우스 | 25000 |
| 이서연 | 모니터 | 210000 |
| 박지훈 | USB허브 | 15000 |
| 박지훈 | 외장하드 | 98000 |
| 최유리 | 노트북 | 1250000 |
| 정도현 | 키보드 | 45000 |
| ~~~~~ | 중간 생략 | |
| 홍세린 | 분무기 | 7000 |
| 이지은 | USB C타입 젠더 | 9000 |
| 박준우 | 소형 선풍기 | 19000 |
| 김민수 | 마우스패드 | 12000 |
| 박지훈 | 스마트폰 거치대 | 11000 |
| 이서연 | 노트북 받침대 | 27000 |
| 박낙현 | 키보드 루프 | 9000 |
| 최건우 | 웹캠 가림막 | 5000 |
| 문가영 | 스마트워치 충전기 | 29000 |

```sql
SELECT c.name,
       o.product,
       o.amount,
       o.order_status
FROM customer c
INNER JOIN orders o
    ON c.customer_id = o.customer_id
WHERE o.order_status <> 'CANCEL';
```

| name | product | amount | order_status |
|------|---------|--------|---------------|
| 김민수 | 키보드 | 45000 | PAID |
| 김민수 | 마우스 | 25000 | PAID |
| 이서연 | 모니터 | 210000 | PAID |
| 박지훈 | USB허브 | 15000 | READY |
| 박지훈 | 외장하드 | 98000 | PAID |
| 최유리 | 노트북 | 1250000 | PAID |
| 정도현 | 키보드 | 45000 | PAID |
| ~~~~~ | 중간 생략 | | |
| 이지은 | USB C타입 젠더 | 9000 | PAID |
| 박준우 | 소형 선풍기 | 19000 | READY |
| 김민수 | 마우스패드 | 12000 | PAID |
| 박지훈 | 스마트폰 거치대 | 11000 | READY |
| 이서연 | 노트북 받침대 | 27000 | PAID |
| 박낙현 | 키보드 루프 | 9000 | PAID |
| 최건우 | 웹캠 가림막 | 5000 | READY |
| 문가영 | 스마트워치 충전기 | 29000 | PAID |

### EX6) 고액 주문을 금액순으로 조회

주문(orders) 테이블의 결제 금액(orders.amount)이 100,000 이상인 주문에 대해 고객 이름(customer.name), 주문 상품명(orders.product), 결제 금액(orders.amount)을 조회하고 결제 금액의 내림차순으로 정렬

```sql
SELECT c.name,
       o.product,
       o.amount
FROM customer c
INNER JOIN orders o
    ON c.customer_id = o.customer_id
WHERE o.amount >= 100000
ORDER BY o.amount DESC;
```

| name | product | amount |
|------|---------|--------|
| 이지은 | 노트북 | 1320000 |
| 김지원 | 노트북 | 1290000 |
| 최유리 | 노트북 | 1250000 |
| 이서연 | 모니터 | 210000 |
| 박준우 | 모니터 | 210000 |

- `WHERE o.amount >= 100000` : 100,000 이상 주문만 선택
- `ORDER BY o.amount DESC` : 금액이 큰 주문부터 정렬

### EX7) 특정 기간의 주문 조회

주문(orders) 테이블의 주문일(orders.order_date)이 2024년 2월 1일부터 2024년 2월 29일까지인 주문에 대해 고객 이름(customer.name), 주문 상품명(orders.product), 결제 금액(orders.amount), 주문일(orders.order_date)을 조회. 주문일을 기준으로 오름차순 정렬해야한다.

```sql
SELECT c.name,
       o.product,
       o.amount,
       o.order_date
FROM customer c
INNER JOIN orders o
    ON c.customer_id = o.customer_id
WHERE o.order_date >= '2024-02-01'
       AND o.order_date <= '2024-02-29'
ORDER BY o.order_date ASC;
```

OR

```sql
SELECT c.name,
       o.product,
       o.amount,
       o.order_date
FROM customer c
INNER JOIN orders o
    ON c.customer_id = o.customer_id
WHERE o.order_date
       BETWEEN '2024-02-01' AND '2024-02-29'
ORDER BY o.order_date ASC;      -- BETWEEN 사용시 작성한 시작일과 종료일을 포함한다.
```

| name | product | amount | order_date |
|------|---------|--------|------------|
| 문가영 | 키보드 | 45000 | 2024-02-01 |
| 문가영 | 마우스 | 25000 | 2024-02-02 |
| 최건우 | 헤드셋 | 99000 | 2024-02-03 |
| 박준우 | 모니터 | 210000 | 2024-02-10 |
| 박낙현 | 웹캠 | 55000 | 2024-02-11 |
| 이서연 | 모니터 | 210000 | 2024-02-12 |
| 이지은 | 노트북 | 1320000 | 2024-02-14 |
| 김민수 | 마우스패드 | 12000 | 2024-02-14 |
| 이지은 | 마우스 | 25000 | 2024-02-15 |
| 오하늘 | 헤드셋 | 99000 | 2024-02-17 |
| 문예서 | 스탠드 | 30000 | 2024-02-18 |
| 정도현 | 키보드 | 45000 | 2024-02-20 |
| 박혜미 | USB허브 | 15000 | 2024-02-20 |
| 박지훈 | 스마트폰 거치대 | 11000 | 2024-02-21 |
| 강민호 | 장패드 | 16000 | 2024-02-22 |
| 한예린 | 마우스패드 | 12000 | 2024-02-25 |
| 이서연 | 노트북 받침대 | 27000 | 2024-02-28 |

### EX8) 상품명에 특정 단어가 포함된 주문 조회

주문(orders) 테이블의 상품명(orders.product)에 '노트북'이라는 단어가 포함된 주문에 대해 고객 이름(customer.name), 상품명(orders.product), 결제 금액(orders.amount), 주문 상태(orders.order_status)를 조회

```sql
SELECT c.name,
       o.product,
       o.amount,
       o.order_status
FROM customer c
INNER JOIN orders o
    ON c.customer_id = o.customer_id
WHERE o.product LIKE '%노트북%';
```

| name | product | amount | order_status |
|------|---------|--------|---------------|
| 최유리 | 노트북 | 1250000 | PAID |
| 이지은 | 노트북 | 1320000 | PAID |
| 김지원 | 노트북 | 1290000 | READY |
| 이태민 | 노트북받침대 | 27000 | PAID |
| 이서연 | 노트북 받침대 | 27000 | PAID |

### EX9) 여러 도시의 고객 주문 조회

고객(customer) 테이블의 거주 도시(customer.city)가 부산 또는 대구인 고객의 이름(customer.name), 도시(customer.city), 주문 상품명(orders.product), 주문일(orders.order_date)을 조회. 도시(customer.city)는 오름차순, 주문일(orders.order_date)은 내림차순으로 정렬

```sql
SELECT c.name,
       c.city,
       o.product,
       o.order_date
FROM customer c
INNER JOIN orders o
    ON c.customer_id = o.customer_id
WHERE c.city='부산' OR c.city='대구'
ORDER BY c.city ASC, o.order_date DESC
```

OR

```sql
SELECT c.name,
       c.city,
       o.product,
       o.order_date
FROM customer c
INNER JOIN orders o
    ON c.customer_id = o.customer_id
WHERE c.city IN ('부산' , '대구')
ORDER BY c.city ASC, o.order_date DESC
```

| name | city | product | order_date |
|------|------|---------|------------|
| 오석진 | 대구 | 책상조명 | 2024-03-05 |
| 박지훈 | 대구 | 외장하드 | 2024-03-03 |
| 박지훈 | 대구 | USB허브 | 2024-03-01 |
| 박지훈 | 대구 | 스마트폰 거치대 | 2024-02-21 |
| 김도현 | 대구 | 스피커 | 2024-01-09 |
| 김도현 | 대구 | 블루투스 이어폰 | 2024-01-07 |
| 최건우 | 부산 | 웹캠 가림막 | 2024-03-15 |
| 서준혁 | 부산 | 스탠드 | 2024-03-11 |
| 박준우 | 부산 | 소형 선풍기 | 2024-03-10 |
| 정소영 | 부산 | 모니터암 | 2024-03-04 |
| 박낙현 | 부산 | 키보드 루프 | 2024-03-02 |
| 이서연 | 부산 | 노트북 받침대 | 2024-02-28 |
| 문예서 | 부산 | 스탠드 | 2024-02-18 |
| 이서연 | 부산 | 모니터 | 2024-02-12 |
| 박낙현 | 부산 | 웹캠 | 2024-02-11 |
| 박준우 | 부산 | 모니터 | 2024-02-10 |
| 최건우 | 부산 | 헤드셋 | 2024-02-03 |
| 윤서준 | 부산 | 스피커 | 2024-01-15 |

### EX10) 특정 연령대 고객의 주문 조회

고객(customer) 테이블의 나이(customer.age)가 30세 이상 39세 이하인 고객의 이름(customer.name), 나이(customer.age), 주문 상품명(orders.product), 결제 금액(orders.amount)을 조회

```sql
SELECT c.name,
       c.age,
       o.product,
       o.amount
FROM customer c
INNER JOIN orders o
    ON c.customer_id = o.customer_id
WHERE c.age >= 30 AND c.age <= 39;
```

OR

```sql
SELECT c.name,
       c.age,
       o.product,
       o.amount
FROM customer c
INNER JOIN orders o
    ON c.customer_id = o.customer_id
WHERE c.age BETWEEN 30 AND 39;
```

| name | age | product | amount |
|------|-----|---------|--------|
| 이서연 | 31 | 모니터 | 210000 |
| 한예린 | 35 | 마우스패드 | 12000 |
| 한예린 | 35 | USB케이블 | 8000 |
| 오하늘 | 33 | 헤드셋 | 99000 |
| 오하늘 | 33 | 키보드 | 45000 |
| 문가영 | 31 | 키보드 | 45000 |
| 문가영 | 31 | 마우스 | 25000 |
| 이유나 | 38 | USB허브 | 15000 |
| 이아린 | 33 | USB케이블 | 8000 |
| 이아린 | 33 | 마우스패드 | 12000 |
| 박낙현 | 36 | 웹캠 | 55000 |
| 최하연 | 30 | 스피커 | 67000 |
| 최하연 | 30 | 키보드 | 45000 |
| 오석진 | 34 | 책상조명 | 35000 |
| 이태민 | 32 | 키보드 | 45000 |
| 이태민 | 32 | 마우스 | 25000 |
| 이태민 | 32 | 노트북받침대 | 27000 |
| 조윤식 | 37 | 블루투스 스피커 | 56000 |
| 홍세린 | 39 | 분무기 | 7000 |
| 이서연 | 31 | 노트북 받침대 | 27000 |
| 박낙현 | 36 | 키보드 루프 | 9000 |
| 문가영 | 31 | 스마트워치 충전기 | 29000 |

### EX11) 도시별 주문 통계 조회

고객(customer) 테이블의 거주 도시(customer.city)별로 주문(orders) 테이블의 주문 번호(orders.order_id) 개수와 결제 금액(orders.amount) 합계를 계산. 도시(customer.city), 주문 건수(order_cnt), 총 주문 금액(order_sum)을 조회. 주문 건수(order_cnt)로 내림차순 정렬 후 주문 금액(order_sum)순으로 내림차순 정렬해야 한다.

```sql
SELECT c.city,
       COUNT(o.order_id) AS order_cnt,
       SUM(o.amount) AS order_sum
FROM customer c
INNER JOIN orders o
    ON c.customer_id = o.customer_id
GROUP BY c.city
ORDER BY order_cnt DESC, order_sum DESC;
```

| city | order_cnt | order_sum |
|------|-----------|-----------|
| 서울 | 13 | 1689000 |
| 부산 | 12 | 850000 |
| 광주 | 9 | 251000 |
| 인천 | 6 | 2624000 |
| 대구 | 6 | 315000 |
| 대전 | 4 | 299000 |

### EX12) 주문 건수가 일정 건수 이상인 고객 조회

고객(customer) 테이블의 고객 ID(customer.customer_id), 고객 이름(customer.name), 거주 도시(customer.city)를 기준으로 주문(orders) 테이블의 주문 건수를 계산. 주문 건수가 3건 이상인 고객만 고객 이름(customer.name), 도시(customer.city), 주문 건수(order_count)를 조회하고 주문 건수의 내림차순으로 정렬

```sql
SELECT c.customer_id,
       c.name,
       c.city,
       COUNT(o.order_id) AS order_count
FROM customer c
INNER JOIN orders o
    ON c.customer_id = o.customer_id
GROUP BY c.customer_id,
       c.name,
       c.city
HAVING  COUNT(o.order_id) >=2
ORDER BY order_count DESC;
```

| customer_id | name | city | order_count |
|-------------|------|------|--------------|
| jieun24 | 이지은 | 인천 | 3 |
| minsu25 | 김민수 | 서울 | 3 |
| taemin32 | 이태민 | 광주 | 3 |
| jihoon29 | 박지훈 | 대구 | 3 |
| gayoung31 | 문가영 | 서울 | 3 |
| arinlee | 이아린 | 인천 | 2 |
| nakhyun | 박낙현 | 부산 | 2 |
| yerin_h | 한예린 | 광주 | 2 |
| dohyun26 | 김도현 | 대구 | 2 |
| seoyeon88 | 이서연 | 부산 | 2 |
| geon22 | 최건우 | 부산 | 2 |
| hanul33 | 오하늘 | 대전 | 2 |
| hayeon30 | 최하연 | 광주 | 2 |
| junwoo15 | 박준우 | 부산 | 2 |
| minho_k | 강민호 | 서울 | 2 |

```sql
SELECT c.customer_id,
       c.name,
       c.city,
       COUNT(o.order_id) AS order_count,
       SUM(o.amount) AS order_sum
FROM customer c
INNER JOIN orders o
    ON c.customer_id = o.customer_id
GROUP BY c.customer_id,
       c.name,
       c.city
HAVING  COUNT(o.order_id) >=2
ORDER BY order_count DESC, order_sum DESC;
```

| customer_id | name | city | order_count | order_sum |
|-------------|------|------|--------------|-----------|
| jieun24 | 이지은 | 인천 | 3 | 1354000 |
| jihoon29 | 박지훈 | 대구 | 3 | 124000 |
| gayoung31 | 문가영 | 서울 | 3 | 99000 |
| taemin32 | 이태민 | 광주 | 3 | 97000 |
| minsu25 | 김민수 | 서울 | 3 | 82000 |
| seoyeon88 | 이서연 | 부산 | 2 | 237000 |
| junwoo15 | 박준우 | 부산 | 2 | 229000 |
| dohyun26 | 김도현 | 대구 | 2 | 156000 |
| hanul33 | 오하늘 | 대전 | 2 | 144000 |
| hayeon30 | 최하연 | 광주 | 2 | 112000 |
| geon22 | 최건우 | 부산 | 2 | 104000 |
| minho_k | 강민호 | 서울 | 2 | 71000 |
| nakhyun | 박낙현 | 부산 | 2 | 64000 |
| yerin_h | 한예린 | 광주 | 2 | 20000 |
| arinlee | 이아린 | 인천 | 2 | 20000 |

**정리**: EX1~EX12를 통해 customer와 orders를 INNER JOIN으로 연결한 뒤 WHERE(도시/상태/금액/기간/LIKE 조건), ORDER BY(단일·복합 정렬), GROUP BY + COUNT/SUM(도시별 통계), HAVING(그룹 조건 필터링)까지 실무에서 자주 쓰이는 조회 패턴을 모두 실습했다.

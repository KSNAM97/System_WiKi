# Docker 07 — YAML 문법 & Docker Compose

## 목차

1. [YAML 문법](#yaml-문법)
   1. [YAML의 문법](#yaml의-문법)
   2. [YAML에서 쓰이는 데이터 타입](#yaml에서-쓰이는-데이터-타입)
   3. [줄바꿈과 여러 줄 문자열 쓰기](#줄바꿈과-여러-줄-문자열-쓰기)
   4. [주석(comment)](#주석comment)
   5. [앵커(&)와 별칭(*)](#앵커와-별칭)
2. [Docker Compose](#docker-compose)
   1. [Docker Compose 개요](#docker-compose-개요)
   2. [step1: nginx 단일 컨테이너](#step1-nginx-단일-컨테이너)
   3. [step1: httpd 단일 컨테이너](#step1-httpd-단일-컨테이너)
   4. [step1: MySQL Pull](#step1-mysql-pull)
   5. [step1: nginx with Dockerfile + build](#step1-nginx-with-dockerfile--build)
   6. [step2: nginx 2개 컨테이너](#step2-nginx-2개-컨테이너)
   7. [step2: MySQL + phpMyAdmin 2-container](#step2-mysql--phpmyadmin-2-container)
   8. [step3-1: Web Server 2대 + Reverse Proxy](#step3-1-web-server-2대--reverse-proxy)

# YAML 문법

## YAML의 문법

- **YAML** = YAML Ain't Markup Language
- 사람 눈에 읽기 쉽게 만든 데이터 표현 형식(데이터를 적는 규칙)이며, Docker Compose에서는 이 YAML 문법으로 여러 컨테이너의 구성을 하나의 파일에 정의해 `docker compose up` 한 번으로 전체 스택을 실행하는 데 쓰인다.

어디에 쓰이는가
- Docker Compose : `docker-compose.yml`
- Kubernetes 설정 : `deployment.yaml`, `service.yaml` 등
- Ansible 플레이북 : `site.yml`
- GitHub Actions : `.github/workflows/*.yml`
- 여러 웹/백엔드 프레임워크 설정 파일

### YAML의 가장 중요한 3가지 규칙

**1) 들여쓰기로 구조를 표현한다.**
- 들여쓰기 = 상하관계(부모-자식 구조)를 의미한다.
- 같은 깊이는 같은 칸 수로 들여쓴다.
- 일반적으로 공백 2칸 또는 4칸을 많이 사용 (혼합하면 안 됨)

```yaml
server:
  host: localhost
  port: 8080
```

- `server` : 최상위 키
- `host`, `port`는 server의 자식 (들여쓰기 되어 있다.)

**2) 탭(tab) 사용 금지, 공백(space)만 사용**
- YAML은 탭 대신 공백으로 들여쓰기 해야 한다.
- 탭이 섞여 있으면 파서가 에러 내는 경우가 많다.

**3) key: value 형식으로 작성**
- 기본 형태는 "키: 값"의 형태이다.
- key를 사용해서 value를 호출한다.
- YAML의 핵심 구조(key: value)는 Python의 dict, Java의 Map과 거의 동일하다.

**정리**: 들여쓰기 기반 구조, 공백 전용 들여쓰기, key: value 형식이라는 YAML의 3대 규칙을 살펴봤다. 이어서 YAML이 다루는 구체적인 데이터 타입들을 살펴본다.

---

## YAML에서 쓰이는 데이터 타입

- YAML도 결국 데이터 표현이기 때문에 데이터 종류(타입)를 이해해야 한다.
- 크게 3가지로 보면 된다.
  - 스칼라(scalar) : 한 줄짜리 값
  - 맵(map, 딕셔너리) : key: value 묶음
  - 리스트(list, 배열) : 여러 값을 순서대로 나열

### 스칼라(scalar) 값

- YAML에서 값 하나로 표현되는 데이터를 말한다.
- 리스트도 아니고, 여러 개의 키:값이 묶인 맵(map)도 아니라 단일 값을 의미한다.
- 대표적인 스칼라 타입은 다음 네 가지이다. : 문자열(string), 숫자(number), 불리언(boolean), null

**문자열(string)**
- 문자 그대로 텍스트(text) 형태의 값이다.
- 문자, 기호, 공백 등을 모두 포함한다.
- 이름, 경로, 문장, 설정값 등이 문자열일 때 사용한다.

```yaml
message: hello world
user: guest
path: /var/www/html
```

**숫자(number)**
- 정수 또는 실수 형태의 값이다.
- 따옴표가 없으면 YAML은 숫자로 인식한다.

```yaml
port: 8080
max_retries: 3
pi_approx: 3.14
```

**불리언(boolean)**
- `true` 또는 `false` 값을 의미한다.
- 기능 on/off, 조건이 필요한 설정 등에 사용된다.

```yaml
debug: true
enabled: false
```

**null 값**
- 값이 '없다'라는 의미이다.
- YAML에서는 `null`, `~`, 비워둔 값 모두 null로 해석된다.

```yaml
description: null
another: ~
```

- null과 ~ 둘 다 값이 없음을 의미한다.

문자열에서 따옴표는 생략해도 되지만 특수 문자(공백이 앞에 온다든지, `:`, `#`가 포함된다든지)가 애매하면 따옴표로 감싸는 게 안전하다.

```yaml
title: "YAML: beginner guide"
note: "앞 공백은 '  이렇게 ' 표현"
```

---

### 맵(map, 딕셔너리)

- 맵은 키-값 쌍을 모아놓은 것이다.
- JSON의 객체(Object)와 같은 개념이다.

```yaml
server:
  host: localhost
  port: 8080
  debug: true
```

- 최상위에 `server`라는 키가 있고
- `server`의 값은 다시 맵이다.
- 그 맵 안에 `host`, `port`, `debug`라는 키가 있다.
- 한 줄에 쓰는 방법도 있다 : `server: { host: localhost, port: 8080, debug: true }`
- 하지만 실무에서는 거의 사용하지 않는다. (여러 줄 + 들여쓰기 방식 사용)

---

### 리스트(list, 배열)

- 리스트는 순서가 있는 값들의 모음이다.
- YAML에서는 하이픈(`-`), `[]`으로 표현한다.

```yaml
fruits:
  - apple
  - banana
  - orange
```

- `fruits`라는 키가 있고
- 그 값은 리스트이고 리스트 안에 apple, banana, orange라는 3개의 요소가 있다.
- 한 줄에 쓰는 방법 : `fruits: [apple, banana, orange]`

**정리**: 스칼라, 맵, 리스트라는 YAML의 3가지 기본 데이터 타입을 살펴봤다. 다음으로는 여러 줄 문자열을 다루는 리터럴/접힌 스타일 문법을 살펴본다.

---

## 줄바꿈과 여러 줄 문자열 쓰기

- 설명 문장이나 SQL, 스크립트, HTML 등 여러 줄 문자열을 YAML 안에 넣어야 할 때가 많다.
- 이때 자주 쓰는 문법이 두 가지 있다.

### 리터럴 스타일(literal style, "|")

- `|` 문자를 사용하면 줄바꿈을 그대로 유지한 문자열을 만든다.
- 즉, YAML에서 보이는 줄바꿈이 값에도 그대로 포함된다.

```yaml
description: |
이 서비스는 테스트용 웹 서버입니다.
두 번째 줄입니다.
세 번째 줄입니다.
```

이 구조는 프로그램에서 읽으면 아래와 같은 실제 문자열이 된다.
```
이 서비스는 테스트용 웹 서버입니다.
두 번째 줄입니다.
세 번째 줄입니다.
```

### 접힌 스타일(folded style, ">")

- 이 값은 프로그램에서 읽으면 줄바꿈이 공백으로 변환된 하나의 긴 문자열이 된다.
- 즉, YAML에서는 여러 줄로 작성돼도 실제로는 한 줄 문자열처럼 연결된다.

```yaml
message: >
  이 문장은 YAML 파일에서
  여러 줄로 써 있지만,
  실제로는 한 줄 문자열처럼
  합쳐집니다.
```

이 값은 프로그램에서 읽으면
```
이 문장은 YAML 파일에서 여러 줄로 써 있지만, 실제로는 한 줄 문자열처럼 합쳐집니다.
```
이런 식으로 중간 줄바꿈이 공백으로 바뀐다.

**정리**: 줄바꿈을 그대로 유지하는 리터럴 스타일(`|`)과 공백으로 합쳐지는 접힌 스타일(`>`)의 차이를 살펴봤다. 다음은 YAML의 주석 문법이다.

---

## 주석(comment)

- 주석은 `#` 기호로 시작한다.
- 한 줄 전체를 주석으로 써도 되고 값 뒤에 붙여도 된다.
- 이후의 내용은 파서가 무시한다.

```yaml
server:
  host: localhost    # 개발 환경에서는 localhost 사용
  port: 8080         # 기본 포트
```

**정리**: `#`으로 시작하는 YAML 주석 문법을 살펴봤다. 이어서 반복되는 설정을 재사용하는 앵커와 별칭 문법을 살펴본다.

---

## 앵커(&)와 별칭(*)

- YAML에서 반복되는 설정을 다시 쓰지 않고 재사용하기 위한 기능이다.
- Docker Compose, Kubernetes에서 자주 사용된다.

| 기호 | 의미 |
|------|------|
| `&name` | 앵커(anchor) — 특정 블록에 이름을 붙이는 것 |
| `*name` | 별칭(alias) — 앵커로 붙여둔 내용을 다시 가져오는 것 |
| `<<: *name` | 앵커로 저장해둔 맵(map) 내용을 그대로 가져와 현재 위치에 합치는 것 |

### 예제1)

```yaml
default-env: &common-env
  DB_HOST: db.example.com
  DB_PORT: 3306

dev:
  environment:
    <<: *common-env
    MODE: dev

prod:
  environment:
    <<: *common-env
    MODE: prod
```

YAML이 실제로 해석하는 최종 결과
```yaml
default-env:
  DB_HOST: db.example.com
  DB_PORT: 3306

dev:
  environment:
    DB_HOST: db.example.com
    DB_PORT: 3306
    MODE: dev

prod:
  environment:
    DB_HOST: db.example.com
    DB_PORT: 3306
    MODE: prod
```

### 예제2)

```yaml
default-env: &default-env
  SPRING_PROFILES_ACTIVE: dev
  LOG_LEVEL: INFO

app1:
  environment:
    <<: *default-env
    APP_NAME: app1

app2:
  environment:
    <<: *default-env
    APP_NAME: app2
```

YAML이 실제로 해석하는 최종 결과
```yaml
app1:
  environment:
    SPRING_PROFILES_ACTIVE: dev
    LOG_LEVEL: INFO
    APP_NAME: app1

app2:
  environment:
    SPRING_PROFILES_ACTIVE: dev
    LOG_LEVEL: INFO
    APP_NAME: app2
```

**정리**: `&`(앵커), `*`(별칭), `<<: *name`(병합)을 이용해 반복되는 YAML 설정을 재사용하는 방법을 두 가지 예제로 살펴봤다. 이제 이러한 YAML 문법을 기반으로 여러 컨테이너를 정의하고 관리하는 Docker Compose를 살펴본다.

---

# Docker Compose

## Docker Compose 개요

- **Docker Compose**는 여러 개의 컨테이너를 한 파일(`docker-compose.yaml`)로 정의하고 한 번에 실행·중지·관리하는 도구이다.
- `docker run` 명령을 여러 번 실행하는 대신 하나의 YAML 파일로 전체 스택을 정의한다.

### 주요 옵션 (docker-compose.yaml)

| 옵션 | 설명 |
|------|------|
| `version` | Compose 파일 포맷 버전 |
| `services` | 실행할 컨테이너(서비스)들을 정의하는 블록 |
| `build` | 이미지를 빌드할 Dockerfile 위치 지정 |
| `image` | 사용할 도커 이미지 이름 |
| `command` | 컨테이너 실행 시 기본 CMD를 덮어쓸 명령어 |
| `ports` | 호스트:컨테이너 포트 바인딩 |
| `expose` | 컨테이너 내부에서만 사용하는 포트 (외부 노출 X) |
| `volumes` | 볼륨 또는 바인드 마운트 설정 |
| `environment` | 환경 변수 설정 |
| `restart` | 재시작 정책 (no, always, on-failure, unless-stopped) |
| `depends_on` | 서비스 시작 순서 의존성 |
| `networks` | 사용자 정의 네트워크 연결 |

### Compose 명령어

| 명령어 | 설명 |
|--------|------|
| `docker compose up -d` | 컨테이너 백그라운드 실행 |
| `docker compose down` | 컨테이너 중지 및 제거 |
| `docker compose ps` | 실행 중인 서비스 목록 |
| `docker compose logs` | 서비스 로그 확인 |
| `docker compose config` | 최종 설정 내용 확인 |
| `docker compose build` | 이미지 빌드 |
| `docker compose ls` | 전체 Compose 프로젝트 목록 |

### 설치

```bash
sudo dnf install -y docker-compose-plugin
```

**정리**: Docker Compose의 주요 옵션(services, build, ports, volumes 등)과 명령어(up, down, ps, logs 등)를 정리했다. 이어서 단일 컨테이너부터 단계별로 Compose 실습을 진행한다.

---

## step1: nginx 단일 컨테이너

```bash
[guest@Server-A ~]$ mkdir -p /compose-lab/step1-nginx
[guest@Server-A ~]$ cd /compose-lab/step1-nginx


[guest@Server-A step1-nginx]$ vi docker-compose.yaml
services:
  web:
    image: nginx:latest
    container_name: step1-nginx
    ports:
      - "80:80"

:wq


[guest@Server-A step1-nginx]$ docker  compose  up  -d
[+] Running 2/2
 ✔ Network step1-nginx_default      Created   0.1s
 ✔ Container step1-nginx            Started   0.3s


[guest@Server-A step1-nginx]$ docker  compose  ps
NAME          IMAGE          COMMAND                   SERVICE   CREATED          STATUS          PORTS
step1-nginx   nginx:latest   "/docker-entrypoint.…"   web       11 seconds ago   Up 10 seconds   0.0.0.0:80->80/tcp, [::]:80->80/tcp


[guest@Server-A step1-nginx]$ docker  compose  down
[+] Running 2/2
 ✔ Container step1-nginx            Removed   0.3s
 ✔ Network step1-nginx_default      Removed   0.1s
```

## step1: httpd 단일 컨테이너

```bash
[guest@Server-A ~]$ mkdir -p /compose-lab/step1-httpd
[guest@Server-A ~]$ cd /compose-lab/step1-httpd


[guest@Server-A step1-httpd]$ vi docker-compose.yaml
services:
  web:
    image: httpd:latest
    container_name: step1-httpd
    ports:
      - "8080:80"

:wq


[guest@Server-A step1-httpd]$ docker  compose  up  -d
[+] Running 2/2
 ✔ Network step1-httpd_default      Created   0.1s
 ✔ Container step1-httpd            Started   0.3s


[guest@Server-A step1-httpd]$ docker  compose  ps
NAME          IMAGE          COMMAND              SERVICE   CREATED         STATUS         PORTS
step1-httpd   httpd:latest   "httpd-foreground"   web       5 seconds ago   Up 5 seconds   0.0.0.0:8080->80/tcp, [::]:8080->80/tcp


[guest@Server-A step1-httpd]$ docker  compose  down
```

## step1: MySQL Pull

```bash
[guest@Server-A ~]$ mkdir -p /compose-lab/step1-mysql-pull
[guest@Server-A ~]$ cd /compose-lab/step1-mysql-pull


[guest@Server-A step1-mysql-pull]$ vi docker-compose.yaml
services:
  db:
    image: mysql:8.0
    container_name: step1-mysql
    environment:
      MYSQL_ROOT_PASSWORD: admin123
      MYSQL_DATABASE: testdb
    ports:
      - "3306:3306"
    volumes:
      - mydata:/var/lib/mysql

volumes:
  mydata:

:wq


[guest@Server-A step1-mysql-pull]$ docker  compose  up  -d
[+] Running 3/3
 ✔ Network step1-mysql-pull_default   Created   0.1s
 ✔ Volume "step1-mysql-pull_mydata"   Created   0.0s
 ✔ Container step1-mysql              Started   0.6s


[guest@Server-A step1-mysql-pull]$ docker  compose  ps
NAME          IMAGE       COMMAND                   SERVICE   CREATED         STATUS         PORTS
step1-mysql   mysql:8.0   "docker-entrypoint.s…"   db        5 seconds ago   Up 4 seconds   0.0.0.0:3306->3306/tcp, 33060/tcp


[guest@Server-A step1-mysql-pull]$ docker  compose  down
```

## step1: nginx with Dockerfile + build

```bash
[guest@Server-A ~]$ mkdir -p /compose-lab/step1-nginx-build
[guest@Server-A ~]$ cd /compose-lab/step1-nginx-build


# Dockerfile 작성
[guest@Server-A step1-nginx-build]$ vi Dockerfile
FROM nginx:latest
COPY index.html /usr/share/nginx/html/index.html

:wq


# index.html 작성
[guest@Server-A step1-nginx-build]$ vi index.html
<h1>Hello Docker Compose!</h1>
<p>Step1 with Custom Dockerfile</p>

:wq


# docker-compose.yaml 작성
[guest@Server-A step1-nginx-build]$ vi docker-compose.yaml
services:
  web:
    build:
      context: .
      dockerfile: Dockerfile
    image: step1-custom-nginx:latest
    container_name: step1-build-nginx
    ports:
      - "80:80"

:wq


[guest@Server-A step1-nginx-build]$ docker  compose  up  -d
[+] Building 0.1s (7/7) FINISHED
 => [web 1/2] FROM docker.io/library/nginx:latest
 => [web 2/2] COPY index.html /usr/share/nginx/html/index.html
 => exporting to image
 ✔ Network step1-nginx-build_default   Created
 ✔ Container step1-build-nginx          Started


[guest@Server-A step1-nginx-build]$ docker  compose  ps
NAME                IMAGE                      COMMAND                   SERVICE   CREATED         STATUS         PORTS
step1-build-nginx   step1-custom-nginx:latest  "/docker-entrypoint.…"   web       4 seconds ago   Up 3 seconds   0.0.0.0:80->80/tcp


[guest@Server-A step1-nginx-build]$ docker  compose  down
```

**정리**: nginx, httpd, MySQL(pull 이미지) 단일 컨테이너 실행과, Dockerfile을 빌드해서 실행하는 방식까지 step1 실습을 마쳤다. 이어서 여러 컨테이너를 함께 정의하는 step2 실습(nginx 2개, MySQL+phpMyAdmin)을 진행한다.

---

## step2: nginx 2개 컨테이너

```bash
[guest@Server-A ~]$ mkdir -p /compose-lab/step2-2nginx
[guest@Server-A ~]$ cd /compose-lab/step2-2nginx


[guest@Server-A step2-2nginx]$ vi docker-compose.yaml
services:
  web1:
    image: nginx:latest
    container_name: step2-nginx1
    ports:
      - "8081:80"

  web2:
    image: nginx:latest
    container_name: step2-nginx2
    ports:
      - "8082:80"

:wq


[guest@Server-A step2-2nginx]$ docker  compose  up  -d
[+] Running 3/3
 ✔ Network step2-2nginx_default   Created   0.1s
 ✔ Container step2-nginx1          Started   0.3s
 ✔ Container step2-nginx2          Started   0.3s


[guest@Server-A step2-2nginx]$ docker  compose  ps
NAME           IMAGE          COMMAND                   SERVICE   CREATED         STATUS         PORTS
step2-nginx1   nginx:latest   "/docker-entrypoint.…"   web1      4 seconds ago   Up 4 seconds   0.0.0.0:8081->80/tcp
step2-nginx2   nginx:latest   "/docker-entrypoint.…"   web2      4 seconds ago   Up 4 seconds   0.0.0.0:8082->80/tcp


[guest@Server-A step2-2nginx]$ docker  compose  down
```

---

## step2: MySQL + phpMyAdmin 2-container

```bash
[guest@Server-A ~]$ mkdir -p /compose-lab/mysql-pma
[guest@Server-A ~]$ cd /compose-lab/mysql-pma


[guest@Server-A mysql-pma]$ vi docker-compose.yaml
services:
  db:
    image: mysql:8.0
    container_name: step2-mysql-db
    environment:
      MYSQL_ROOT_PASSWORD: admin123
      MYSQL_DATABASE: sampledb
    volumes:
      - mydbdata:/var/lib/mysql
    ports:
      - "3308:3306"

  pma:
    image: phpmyadmin/phpmyadmin:latest
    container_name: step2-mysql-pma
    environment:
      PMA_HOST: db
      PMA_PORT: 3306
    depends_on:
      - db
    ports:
      - "8086:80"

volumes:
  mydbdata:

:wq


[guest@Server-A mysql-pma]$ docker  compose  config
name: mysql-pma
services:
  db:
    container_name: step2-mysql-db
    environment:
      MYSQL_DATABASE: sampledb
      MYSQL_ROOT_PASSWORD: admin123
    image: mysql:8.0
    networks:
      default: null
    ports:
      - mode: ingress
        target: 3306
        published: "3308"
        protocol: tcp
    volumes:
      - type: volume
        source: mydbdata
        target: /var/lib/mysql
  pma:
    container_name: step2-mysql-pma
    depends_on:
      db:
        condition: service_started
        required: true
    environment:
      PMA_HOST: db
      PMA_PORT: 3306
    image: phpmyadmin/phpmyadmin:latest
    networks:
      default: null
    ports:
      - mode: ingress
        target: 80
        published: "8086"
        protocol: tcp
networks:
  default:
    name: mysql-pma_default
volumes:
  mydbdata:
    name: mysql-pma_mydbdata


[guest@Server-A mysql-pma]$ docker  compose  up  -d
[+] up 26/26
 ✔ Image phpmyadmin/phpmyadmin:latest Pulled                            13.7s
 ✔ Network mysql-pma_default          Created                           0.1s
 ✔ Volume "mysql-pma_mydbdata"        Created                           0.0s
 ✔ Container step2-mysql-db           Started                           0.2s
 ✔ Container step2-mysql-pma          Started                           0.3s


[guest@Server-A mysql-pma]$ docker  compose  ps
NAME              IMAGE                         COMMAND                   SERVICE   CREATED         STATUS        PORTS
step2-mysql-db    mysql:8.0                     "docker-entrypoint.s…"   db        3 seconds ago   Up 3 seconds  33060/tcp, 0.0.0.0:3308->3306/tcp
step2-mysql-pma   phpmyadmin/phpmyadmin:latest  "/docker-entrypoint.…"   pma       3 seconds ago   Up 3 seconds  0.0.0.0:8086->80/tcp


[guest@Server-A mysql-pma]$ docker  exec -it  step2-mysql-db  /bin/bash
bash-5.1#

bash-5.1# mysql -u root -p
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 86
Server version: 8.0.46 MySQL Community Server - GPL

mysql>

mysql> SHOW DATABASES;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sampledb           |
| sys                |
+--------------------+
5 rows in set (0.00 sec)

mysql>

mysql> USE sampledb;
Reading table information for completion of table and column names
Database changed

mysql> SHOW TABLES;
+--------------------+
| Tables_in_sampledb |
+--------------------+
| test_table         |
+--------------------+
1 row in set (0.00 sec)

mysql> select * from test_table;
+----+----------------------+
| id | msg                  |
+----+----------------------+
|  1 | hello  phpmysqladmin |
+----+----------------------+
1 row in set (0.00 sec)


[guest@Server-A mysql-pma]$ docker  compose  down
[+] down 3/3
 ✔ Container step2-mysql-pma Removed   1.2s
 ✔ Container step2-mysql-db  Removed   1.5s
 ✔ Network mysql-pma_default Removed   0.1s
```

**정리**: `depends_on`으로 DB가 먼저 뜬 뒤 phpMyAdmin이 연결되는 2-container 구성과 `docker compose config`로 최종 설정을 확인하는 방법을 실습했다. 마지막으로 리버스 프록시와 로드밸런싱까지 포함한 3-container 구성을 실습한다.

---

## step3-1: Web Server 2대 + Reverse Proxy

### Reverse Proxy 개념

- **리버스 프록시**는 클라이언트(사용자) 대신 서버(Nginx)가 백엔드 서버로 요청을 대신 보내주는 구조이다.
- 사용자 → Proxy Nginx → Nginx(app1)/Nginx(app2) 구조
- 여기서 사용자는 app1, app2의 IP, 포트, 위치를 전혀 모른다. (모든 요청은 Proxy Nginx에서 처리)

Reverse Proxy가 필요한 이유
- 여러 서버를 한 도메인 뒤로 숨길 수 있다.
  - `/app1` → app1 컨테이너
  - `/app2` → app2 컨테이너
  - `/api` → API 서버
  - `/static` → 정적 서버
  - 사용자는 이것들이 전부 한 서버로 보인다.
- 보안 강화: 백엔드 서버(app1/app2)의 IP가 외부에 노출되지 않는다.
- 로드밸런싱 가능: app1, app2를 라운드로빈으로 분배할 수 있다.

### 디렉터리 구성

```bash
[guest@Server-A ~]$ sudo mkdir -p /compose-lab/step3-1/app1/html
[guest@Server-A ~]$ sudo mkdir -p /compose-lab/step3-1/app2/html
[guest@Server-A ~]$ sudo mkdir -p /compose-lab/step3-1/proxy

[guest@Server-A ~]$ cd /compose-lab/step3-1/

[guest@Server-A step3-1]$ ls -l
합계 0
drwxr-xr-x 3 root root 18  8월 10 16:47 app1
drwxr-xr-x 3 root root 18  8월 10 16:47 app2
drwxr-xr-x 2 root root  6  8월 10 16:47 proxy
```

파일 위치
```
app1
  /compose-lab/step3-1/app1/html/index.html
  /compose-lab/step3-1/app1/dockerfile

app2
  /compose-lab/step3-1/app2/html/index.html
  /compose-lab/step3-1/app2/dockerfile

proxy
  /compose-lab/step3-1/proxy/nginx.conf
```

### app1 HTML + Dockerfile

```bash
[guest@Server-A step3-1]$ sudo vi /compose-lab/step3-1/app1/html/index.html
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <title>APP1</title>
</head>
<body>
    <h1>THIS IS APP1 SERVER</h1>
    <p>이 페이지는 app1 컨테이너 이미지 안에 포함된 HTML입니다.</p>
</body>
</html>


[guest@Server-A step3-1]$ sudo vi /compose-lab/step3-1/app1/dockerfile
FROM nginx:latest
COPY ./html  /usr/share/nginx/html
```

### app2 HTML + Dockerfile

```bash
[guest@Server-A step3-1]$ sudo vi /compose-lab/step3-1/app2/html/index.html
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <title>APP2</title>
</head>
<body>
    <h1>THIS IS APP2 SERVER</h1>
    <p>이 페이지는 app2 컨테이너 이미지 안에 포함된 HTML입니다.</p>
</body>
</html>


[guest@Server-A step3-1]$ sudo vi /compose-lab/step3-1/app2/dockerfile
FROM nginx:latest
COPY ./html  /usr/share/nginx/html
```

### reverse proxy 설정 (경로 기반)

```bash
[root@rocky step3-1]# sudo vi /compose-lab/step3-1/proxy/nginx.conf
worker_processes 1;    # NGINX가 사용할 워커 프로세스 개수 (일반적으로 CPU 코어 개수와 동일하게 설정)

events {
    worker_connections 1024;    # 하나의 워커 프로세스가 처리할 최대 연결 개수
}

http {
    include  /etc/nginx/mime.types;    # MIME 타입을 정의 (.html --> text/html | .jpg --> image/jpeg)
    default_type  application/octet-stream;
    charset utf-8;

    sendfile  on;    # 파일을 빠르게 전송하기위한 sendfile 기능 활성화

    server {
        listen 80;    # 외부 클라이언트로부터 수신할 포트번호

        location /app1/ {    # EX) 192.168.10.100/app1
            proxy_pass http://app1/;
        }
        location /app2/ {    # EX) 192.168.10.100/app2
            proxy_pass http://app2/;
        }
        location / {
            return 200 'Use /app1 or /app2';
        }
    }
}


[guest@Server-A step3-1]$ sudo vi /compose-lab/step3-1/proxy/dockerfile
FROM  nginx:latest
COPY  ./nginx.conf  /etc/nginx/nginx.conf
```

### docker-compose.yaml 만들기

```bash
[root@rocky step3-1]# sudo vi /compose-lab/step3-1/docker-compose.yaml
services:
  app1:
    build:
      context: ./app1
      dockerfile: dockerfile
    image: step3-1-app1
    container_name: step3-1-app1

  app2:
    build:
      context: ./app2
      dockerfile: dockerfile
    image: step3-1-app2
    container_name: step3-1-app2

  proxy:
    build:
      context: ./proxy
      dockerfile: dockerfile
    image: step3-1-proxy
    container_name: step3-1-proxy
    ports:
      - "80:80"
    depends_on:
      - app1
      - app2

:wq


[guest@Server-A step3-1]$ docker  compose  config
name: step3-1
services:
  app1:
    build:
      context: /compose-lab/step3-1/app1
      dockerfile: Dockerfile
    container_name: step3-1-app1
    image: step3-1-app1
    networks:
      default: null
  app2:
    build:
      context: /compose-lab/step3-1/app2
      dockerfile: Dockerfile
    container_name: step3-1-app2
    image: step3-1-app2
    networks:
      default: null
  proxy:
    build:
      context: /compose-lab/step3-1/proxy
      dockerfile: Dockerfile
    container_name: step3-1-proxy
    depends_on:
      app1:
        condition: service_started
        required: true
      app2:
        condition: service_started
        required: true
    image: step3-1-proxy
    networks:
      default: null
    ports:
      - mode: ingress
        target: 80
        published: "80"
        protocol: tcp
networks:
  default:
    name: step3-1_default


[guest@Server-A step3-1]$ docker  image  prune -a -f
# prune : 사용하지 않는 Docker 이미지를 정리하는 명령어
# -a    : 컨테이너에서 사용하지 않는 모든 이미지 삭제
# -f    : 삭제 확인 질문 없이 바로 실행


[guest@Server-A step3-1]$ docker compose up -d
 ✔ Network step3-1_default    Created   0.3s
 ✔ Container step3-1-app2     Started   0.8s
 ✔ Container step3-1-app1     Started   0.8s
 ✔ Container step3-1-proxy    Started   1.3s


http://192.168.10.100/app1/    # 접속
http://192.168.10.100/app2/    # 접속


guest@Server-B:~$ curl http://localhost/app1/
<h1>THIS IS APP1 SERVER</h1>
<p>이 페이지는 app1 컨테이너 이미지 안에 포함된 HTML입니다.</p>

guest@Server-B:~$ curl http://localhost/app2/
<h1>THIS IS APP2 SERVER</h1>
<p>이 페이지는 app2 컨테이너 이미지 안에 포함된 HTML입니다.</p>


[guest@Server-A step3-1]$ docker compose down
[+] down 4/4
 ✔ Container step3-1-proxy Removed   0.2s
 ✔ Container step3-1-app2  Removed   0.1s
 ✔ Container step3-1-app1  Removed   0.1s
 ✔ Network step3-1_default Removed
```

---

### proxy 서버 수정 — 라운드 로빈 로드밸런싱

- app1으로 접속시 = `http://192.168.10.100/app1/`
- app2으로 접속시 = `http://192.168.10.100/app2/`
- 각각의 web server로 접속시 uri를 서로 다르게 작성하게 되면 proxy 서버를 사용하는 의미가 없다.
- proxy 서버 구성시 **라운드 로빈** 방식을 사용하게 되면 proxy 서버가 각각의 서버로 번갈아 가면서 연결한다.

```bash
[root@rocky step3-1]# sudo vi /compose-lab/step3-1/proxy/nginx.conf
worker_processes 1;

events {
    worker_connections 1024;
}

http {
    include  /etc/nginx/mime.types;
    default_type  application/octet-stream;
    charset utf-8;
    sendfile  on;

    access_log  /var/log/nginx/access.log;
    error_log  /var/log/nginx/error.log;

    upstream backend {
        server app1;
        server app2;
    }

    server {
        listen 80;

        location / {
            proxy_pass http://backend;
        }
    }
}


:wq
```

upstream backend 옵션
```
# server web1;                              # 기본 서버 등록
# server web1 weight=2;                    # 가중치(트래픽 2배 받음)
# server web1 max_fails=3 fail_timeout=30s; # 3번 실패하면 30초 동안 제외
#============================================================
# 로드밸런싱 알고리즘:
# - 기본: round-robin (순차)
# - least_conn;    # 연결 수가 가장 적은 서버로 전달
# - ip_hash;       # 같은 클라이언트 IP는 항상 같은 서버로 보냄(세션 유지)
```

```bash
[guest@Server-A step3-1]$ docker  image  prune -a -f

[guest@Server-A step3-1]$ docker  compose  up  -d

[guest@Server-A step3-1]$ docker  compose  ps
NAME              IMAGE           COMMAND                   SERVICE   CREATED         STATUS        PORTS
step3-1-app1      step3-1-app1    "/docker-entrypoint.…"   app1      8 seconds ago   Up 8 seconds  80/tcp
step3-1-app2      step3-1-app2    "/docker-entrypoint.…"   app2      8 seconds ago   Up 8 seconds  80/tcp
step3-1-proxy     step3-1-proxy   "/docker-entrypoint.…"   proxy     8 seconds ago   Up 8 seconds  0.0.0.0:80->80/tcp, [::]:80->80/tcp
```

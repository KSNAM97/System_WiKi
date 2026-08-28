# Docker 07 — YAML 문법 & Docker Compose

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

- 서비스: `web` (nginx:latest)
- 기능: `docker run` 대신 `docker compose`로 Nginx 1개 띄우기
- 작업 폴더: `/compose-lab/step1`
- 파일: `docker-compose.yaml`
- 서비스 이름: `web`
- 이미지: `nginx:latest`

```bash
[guest@Server-A ~]$ sudo mkdir  -p  /compose-lab/step1

[guest@Server-A ~]$ cd  /compose-lab/step1/

[guest@Server-A step1]$ pwd
/compose-lab/step1


[guest@Server-A step1]$ vi docker-compose.yaml
version: "3.9"

services:
  web:
    image: nginx:latest
    container_name: step1-nginx
    ports:
      - "8080:80"


[guest@Server-A step1]$ ls  -l
합계 4
-rw-r--r-- 1 root root 119  8월 10 10:45 docker-compose.yaml


[guest@Server-A step1]$ docker compose config
WARN[0000] /compose-lab/step1/docker-compose.yaml: the attribute `version` is obsolete, it will be ignored, please remove it to avoid potential confusion
name: step1
services:
  web:
    container_name: step1-nginx
    image: nginx:latest
    networks:
      default: null
    ports:
      - mode: ingress
        target: 80
        published: "8080"
        protocol: tcp
networks:
  default:
    name: step1_default


# version 줄을 주석 처리한 뒤 다시 확인
[guest@Server-A step1]$ docker compose config
#name: step1		# 주석처리
services:
  web:
    container_name: step1-nginx
    image: nginx:latest
    networks:
      default: null
    ports:
      - mode: ingress
        target: 80
        published: "8080"
        protocol: tcp
networks:
  default:
    name: step1_default


# Docker Compose로 컨테이너 생성
[guest@Server-A step1]$ docker  compose up  -d
[+] up 2/2
 ✔ Network step1_default Created                                   0.1s
 ✔ Container step1-nginx Started                               0.2s


[guest@Server-A step1]$ docker compose ps
NAME          IMAGE          COMMAND                   SERVICE   CREATED          STATUS          PORTS
step1-nginx   nginx:latest   "/docker-entrypoint.…"    web          39 seconds ago    Up 38 seconds   0.0.0.0:8080->80/tcp, [::]:8080->80/tcp


[guest@Server-A step1]$ docker  compose  ls
NAME           	STATUS            	CONFIG FILES
step1               	running(1)      	/compose-lab/step1/docker-compose.yaml
```

- `docker compose ls` : Docker Compose로 실행 중이거나 관리되는 Compose 프로젝트 목록을 확인
  - `NAME` : Compose 프로젝트명(compose 파일이 있는 디렉터리 이름이 출력된다)
  - `STATUS` : 프로젝트 상태(현재 해당 프로젝트로 실행중인 컨테이너 개수가 출력된다)
  - `CONFIG FILES` : 설정파일의 경로 및 파일명 출력

```bash
# 웹 브라우저를 사용하여 접속
http://192.168.10.100:8080/


# curl 명령어를 사용하여 접속 확인
[guest@Server-A ~]$ curl 192.168.10.100:8080
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, nginx is successfully installed and working.
Further configuration is required for the web server, reverse proxy,
API gateway, load balancer, content cache, or other features.</p>

<p>For online documentation and support please refer to
<a href="https://nginx.org/">nginx.org</a>.<br/>
To engage with the community please visit
<a href="https://community.nginx.org/">community.nginx.org</a>.<br/>
For enterprise grade support, professional services, additional
security features and capabilities please refer to
<a href="https://f5.com/nginx">f5.com/nginx</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>


[guest@Server-A step1]$ docker compose  logs web
step1-nginx  | /docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
step1-nginx  | /docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
step1-nginx  | /docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
step1-nginx  | 10-listen-on-ipv6-by-default.sh: info: Getting the checksum of /etc/nginx/conf.d/default.conf
step1-nginx  | 10-listen-on-ipv6-by-default.sh: info: Enabled listen on IPv6 in /etc/nginx/conf.d/default.conf
step1-nginx  | /docker-entrypoint.sh: Sourcing /docker-entrypoint.d/15-local-resolvers.envsh
step1-nginx  | /docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
step1-nginx  | /docker-entrypoint.sh: Launching /docker-entrypoint.d/30-tune-worker-processes.sh
step1-nginx  | /docker-entrypoint.sh: Configuration complete; ready for start up
step1-nginx  | 2026/08/10 01:52:31 [notice] 1#1: using the "epoll" event method
step1-nginx  | 2026/08/10 01:52:31 [notice] 1#1: nginx/1.31.3
step1-nginx  | 2026/08/10 01:52:31 [notice] 1#1: built by gcc 14.2.0 (Debian 14.2.0-19)
step1-nginx  | 2026/08/10 01:52:31 [notice] 1#1: OS: Linux 5.14.0-687.17.1.el9_8.x86_64
step1-nginx  | 2026/08/10 01:52:31 [notice] 1#1: getrlimit(RLIMIT_NOFILE): 1024:524288
step1-nginx  | 2026/08/10 01:52:31 [notice] 1#1: start worker processes
step1-nginx  | 2026/08/10 01:52:31 [notice] 1#1: start worker process 29
step1-nginx  | 2026/08/10 01:52:31 [notice] 1#1: start worker process 30
step1-nginx  | 2026/08/10 01:52:31 [notice] 1#1: start worker process 31
step1-nginx  | 2026/08/10 01:52:31 [notice] 1#1: start worker process 32
step1-nginx  | 192.168.10.1 - - [10/Aug/2026:01:58:58 +0000] "GET / HTTP/1.1" 200 896 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/151.0.0.0 Safari/537.36" "-"
step1-nginx  | 2026/08/10 01:58:58 [error] 29#29: *1 open() "/usr/share/nginx/html/favicon.ico" failed (2: No such file or directory), client: 192.168.10.1, server: localhost, request: "GET /favicon.ico HTTP/1.1", host: "192.168.10.100:8080", referrer: "http://192.168.10.100:8080/"
step1-nginx  | 192.168.10.1 - - [10/Aug/2026:01:58:58 +0000] "GET /favicon.ico HTTP/1.1" 404 555 "http://192.168.10.100:8080/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/151.0.0.0 Safari/537.36" "-"
step1-nginx  | 192.168.10.100 - - [10/Aug/2026:01:59:36 +0000] "GET / HTTP/1.1" 200 896 "-" "curl/7.76.1" "-"
step1-nginx  | 192.168.10.1 - - [10/Aug/2026:02:02:05 +0000] "GET / HTTP/1.1" 200 896 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/151.0.0.0 Safari/537.36 Edg/151.0.0.0" "-"
step1-nginx  | 192.168.10.1 - - [10/Aug/2026:02:02:05 +0000] "GET /favicon.ico HTTP/1.1" 404 555 "http://192.168.10.100:8080/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/151.0.0.0 Safari/537.36 Edg/151.0.0.0" "-"


[guest@Server-A step1]$ docker  compose  down
[+] down 2/2
 ✔ Container step1-nginx Removed                                                       0.2s
 ✔ Network step1_default Removed                                                       0.1s
```

### 연습문제 (step1 응용)

**연습 1** — `docker-compose.yaml`에서 `ports` 부분을 `"8080:80"` → `"8088:80"`으로 바꾸고 다시 `docker compose up -d` 실행 후 `http://서버IP:8088`로 접속해 Nginx 화면이 나오는지 확인한다.

**연습 2** — 이미지 버전을 `latest` 대신 특정 버전으로 바꿔본다. 예: `image: nginx:1.25`. `docker compose down` 후 다시 `up -d`하고 `docker ps`로 어떤 이미지로 올라왔는지 확인한다.

**연습 3** — `container_name` 라인을 지우고 `docker compose down` → `up -d` 다시 한 뒤 `docker ps`로 컨테이너 이름이 어떻게 바뀌는지 확인해본다.

---

## step1: httpd 단일 컨테이너

- `httpd:latest` 이미지를 사용해서 Docker Compose로 웹 서버 컨테이너를 1개 실행한다.
- 호스트에서 index.html 파일을 작성한다.
- `volumes`를 사용하여 호스트의 index.html을 컨테이너의 Apache 기본 웹 페이지 위치에 연결한다.
- 브라우저에서 작성한 웹 페이지가 출력되는지 확인한다.

구조
- 작업 폴더: `/compose-lab/step1-httpd`
- 파일: `docker-compose-httpd.yaml`
- 서비스 이름: `web`
- 이미지: `httpd:latest`
- 포트: 호스트 8083 → 컨테이너 80 (구성 문서 기준. 실습에서는 아래처럼 8080을 재사용)

기본 파일명 (compose가 자동으로 찾는 이름)
1. `compose.yaml`
2. `compose.yml`
3. `docker-compose.yaml`
4. `docker-compose.yml`

```bash
[guest@Server-A step1]$ sudo mkdir -p /compose-lab/step1-httpd

[guest@Server-A step1]$ cd /compose-lab/step1-httpd/

[guest@Server-A step1-httpd]$ pwd
/compose-lab/step1-httpd


[guest@Server-A step1-httpd]$ sudo vi  docker-compose-httpd.yaml
services:
  web-httpd:
    image: httpd:latest
    container_name: myweb_httpd
    ports:
      - "8080:80"
    volumes:
      - ./index.html:/usr/local/apache2/htdocs/index.html:ro


# 컨테이너에서 사용할 index.html 파일 생성
[guest@Server-A step1-httpd]$ cat  index.html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>Docker Compose HTTPD</title>
</head>
<body>
    <h1>Docker Compose HTTPD Web Server</h1>
    <h2>Apache Container Test</h2>
</body>
</html>


[guest@Server-A step1-httpd]$ docker compose config	# 지정된 파일명이 없기때문에 찾지 못한다.
no configuration file provided: not found


# 기본 파일명이 아닌 경우 -f 옵션을 사용해서 파일명을 지정해야 한다.
[guest@Server-A step1-httpd]$ docker compose -f  docker-compose-httpd.yaml  config
name: step1-httpd
services:
  web-httpd:
    container_name: myweb_httpd
    image: httpd:latest
    networks:
      default: null
    ports:
      - mode: ingress
        target: 80
        published: "8080"
        protocol: tcp
    volumes:
      - type: bind
        source: /compose-lab/step1-httpd/index.html
        target: /usr/local/apache2/htdocs/index.html
        read_only: true
        bind: {}
networks:
  default:
    name: step1-httpd_default


[guest@Server-A step1-httpd]$ docker  compose -f  docker-compose-httpd.yaml  up  -d
[+] up 2/2
 ✔ Network step1-httpd_default Created                                                 0.1s
 ✔ Container myweb_httpd       Started                                                  0.2s


[guest@Server-A step1-httpd]$ docker compose ps
no configuration file provided: not found


[guest@Server-A step1-httpd]$ docker compose -f  docker-compose-httpd.yaml  ps
NAME            IMAGE          COMMAND              SERVICE     CREATED          STATUS          PORTS
myweb_httpd   httpd:latest    "httpd-foreground"     web-httpd     35 seconds ago    Up 34 seconds   0.0.0.0:8080->80/tcp, [::]:8080->80/tcp


[guest@Server-A step1-httpd]$ docker compose -f  docker-compose-httpd.yaml  ls
NAME                STATUS              CONFIG FILES
step1-httpd         running(1)             /compose-lab/step1-httpd/docker-compose-httpd.yaml


# 웹 브라우저로 접속
http://192.168.10.100:8080/

Docker Compose HTTPD Web Server
Apache Container Test


# curl 명령어를 사용하여 확인
[guest@Server-A ~]$ curl 192.168.10.100:8080
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>Docker Compose HTTPD</title>
</head>
<body>
    <h1>Docker Compose HTTPD Web Server</h1>
    <h2>Apache Container Test</h2>
</body>
</html>


[guest@Server-A step1-httpd]$ docker  compose -f  docker-compose-httpd.yaml  logs  web-httpd
myweb_httpd  | AH00558: httpd: Could not reliably determine the server's fully qualified domain name, using 172.19.0.2. Set the 'ServerName' directive globally to suppress this message
myweb_httpd  | AH00558: httpd: Could not reliably determine the server's fully qualified domain name, using 172.19.0.2. Set the 'ServerName' directive globally to suppress this message
myweb_httpd  | [Mon Aug 10 02:50:43.746757 2026] [mpm_event:notice] [pid 1:tid 1] AH00489: Apache/2.4.68 (Unix) configured -- resuming normal operations
myweb_httpd  | [Mon Aug 10 02:50:43.746841 2026] [core:notice] [pid 1:tid 1] AH00094: Command line: 'httpd -D FOREGROUND'
myweb_httpd  | 192.168.10.1 - - [10/Aug/2026:02:52:42 +0000] "GET / HTTP/1.1" 200 208
myweb_httpd  | 192.168.10.100 - - [10/Aug/2026:02:53:09 +0000] "GET / HTTP/1.1" 200 208
myweb_httpd  | 192.168.10.1 - - [10/Aug/2026:02:53:33 +0000] "-" 408 -
myweb_httpd  | 192.168.10.200 - - [10/Aug/2026:03:00:12 +0000] "GET / HTTP/1.1" 200 208


[guest@Server-A step1-httpd]$ docker  compose -f  docker-compose-httpd.yaml  down
[+] down 2/2
 ✔ Container myweb_httpd        Removed                                                 1.2s
 ✔ Network step1-httpd_default Removed                                                 0.1s
```

---

## step1: MySQL Pull

- Docker Hub의 `mysql:8.0` 이미지를 pull 해서 바로 실행
- `docker-compose.yaml` 파일만 사용
- MySQL root 비밀번호, DB 자동 생성
- 컨테이너 접속 및 DB 확인까지

구성
- 작업 디렉터리: `/compose-lab/step1-mysql-pull`
- 파일: `docker-compose.yaml`
- 컨테이너 이름: `mysql_db`
- 이미지: `mysql:8.0`
- 호스트 포트: 3307 → 3306

```bash
[guest@Server-A step1-httpd]$ sudo mkdir -p /compose-lab/step1-mysql-pull

[guest@Server-A step1-httpd]$ cd  /compose-lab/step1-mysql-pull/

[guest@Server-A step1-mysql-pull]$ pwd
/compose-lab/step1-mysql-pull


[guest@Server-A step1-mysql-pull]$ sudo  vi  docker-compose.yaml
services:
  web:
    image: mysql:8.0
    container_name: mysql_db
    ports:
      - "3307:3306"
    environment:
      MYSQL_ROOT_PASSWORD: "1234"
      MYSQL_DATABASE: "sampledb"
      MYSQL_USER: "testuser"
      MYSQL_PASSWORD: "test1234"

    volumes:
      - dbdata:/var/lib/mysql

volumes:
  dbdata:

:wq


[guest@Server-A step1-mysql-pull]$ docker  compose  config
name: step1-mysql-pull
services:
  web:
    container_name: mysql_db
    environment:
      MYSQL_DATABASE: sampledb
      MYSQL_PASSWORD: test1234
      MYSQL_ROOT_PASSWORD: "1234"
      MYSQL_USER: testuser
    image: mysql:8.0
    networks:
      default: null
    ports:
      - mode: ingress
        target: 3306
        published: "3307"
        protocol: tcp
    volumes:
      - type: volume
        source: dbdata
        target: /var/lib/mysql
        volume: {}
networks:
  default:
    name: step1-mysql-pull_default
volumes:
  dbdata:
    name: step1-mysql-pull_dbdata


[guest@Server-A step1-mysql-pull]$ docker  compose up -d
[+] up 17/17
 ✔ Image mysql:8.0                  	Pulled                              	18.8s
 ✔ Network step1-mysql-pull_default 	Created                         	0.1s
 ✔ Volume step1-mysql-pull_dbdata   	Created                            	0.0s
 ✔ Container mysql_db               	Started                         	0.2s


[guest@Server-A step1-mysql-pull]$ docker  compose  ps
NAME       IMAGE       COMMAND                   SERVICE   CREATED          STATUS          PORTS
mysql_db   mysql:8.0   "docker-entrypoint.s…"   web           50 seconds ago    Up 49 seconds   33060/tcp, 0.0.0.0:3307->3306/tcp, [::]:3307->3306/tcp


# Docker Volume 확인
[guest@Server-A step1-mysql-pull]$ sudo ls  -l  /var/lib/docker/volumes
[sudo] guest 암호:
합계 32
drwx-----x 3 root root    19  8월  5 17:28 05c327209a0b1b524155426377884ee6340f520e1fe4bc52651aba261f1e325f
drwx-----x 3 root root    19  8월  5 17:33 5fd6c809cfee738b4437578754e3d2881e734456ffdcbbf50891ac5631da5956
drwx-----x 3 root root    19  8월  6 09:44 76aa33cb9f2ba20e2b76e1819db7b7ad0905d1ddae2b8793b49ddbbcd871d99d
drwx-----x 3 root root    19  8월  5 18:06 7f63586331ca555e5bf22a5a3d0fe228c64de01728e0b374d5211c7a90248977
drwx-----x 3 root root    19  8월  5 17:36 82eb4304c3463432ed9742b36ed7881a0da53e925d4eacb562ed6c6360f50e1d
drwx-----x 3 root root    19  8월  6 09:42 95c91f4626c5ec38c7875e945fdb31ef78db03d1e1a7b51dcc98098c9ab8027a
drwx-----x 3 root root    19  8월  7 10:43 9b05843001c68acc15abcd3aae8e9cdd2c3a373628656255995caee4d54f919d
drwx-----x 3 root root    19  8월  6 09:43 a2cb0b42cf7938ea4643c48a00963bb3605c0bd643e96931be379f1b7f35006f
brw------- 1 root root  8, 2  8월 10 10:35 backingFsBlockDev
drwx-----x 3 root root    19  8월  6 10:54 bc3286b6d271c1483af8d5cc83848d1a090263d27aa885a8bcdd8c8eab5263c8
drwx-----x 3 root root    19  8월  6 09:43 e419e26161ef3f699ed60bc4778f0076b47afa271289d9c6f10276fe9550248f
drwx-----x 3 root root    19  8월  5 18:02 e555c275c0a33a6c7e2eb448ea07593746d8023fc9f9d5826cf7943643db85ab
drwx-----x 3 root root    19  8월  7 10:59 fc2e7a728e9ee0bd2ad1307b3020bb476f079ace3019edcb1e48d79750c35585
-rw------- 1 root root 65536  8월 10 12:17 metadata.db
drwx-----x 3 root root    19  8월 10 12:17 step1-mysql-pull_dbdata


[guest@Server-A step1-mysql-pull]$ docker volume ls
DRIVER    VOLUME NAME
local     5fd6c809cfee738b4437578754e3d2881e734456ffdcbbf50891ac5631da5956
local     05c327209a0b1b524155426377884ee6340f520e1fe4bc52651aba261f1e325f
local     7f63586331ca555e5bf22a5a3d0fe228c64de01728e0b374d5211c7a90248977
local     9b05843001c68acc15abcd3aae8e9cdd2c3a373628656255995caee4d54f919d
local     76aa33cb9f2ba20e2b76e1819db7b7ad0905d1ddae2b8793b49ddbbcd871d99d
local     82eb4304c3463432ed9742b36ed7881a0da53e925d4eacb562ed6c6360f50e1d
local     95c91f4626c5ec38c7875e945fdb31ef78db03d1e1a7b51dcc98098c9ab8027a
local     a2cb0b42cf7938ea4643c48a00963bb3605c0bd643e96931be379f1b7f35006f
local     bc3286b6d271c1483af8d5cc83848d1a090263d27aa885a8bcdd8c8eab5263c8
local     e419e26161ef3f699ed60bc4778f0076b47afa271289d9c6f10276fe9550248f
local     e555c275c0a33a6c7e2eb448ea07593746d8023fc9f9d5826cf7943643db85ab
local     fc2e7a728e9ee0bd2ad1307b3020bb476f079ace3019edcb1e48d79750c35585
local     step1-mysql-pull_dbdata
```

- Docker Compose의 Named Volume 기본 이름: `프로젝트이름_볼륨이름`
  - 프로젝트 이름: `step1-mysql-pull`
  - 볼륨 이름: `dbdata`
  - 볼륨명: `step1-mysql-pull_dbdata`

```bash
# 컨테이너 접속
[guest@Server-A step1-mysql-pull]$ docker  exec  -it  mysql_db  /bin/bash
bash-5.1#
bash-5.1# mysql -u root  -p
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 8
Server version: 8.0.46 MySQL Community Server - GPL

Copyright (c) 2000, 2026, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>


mysql> SHOW DATABASES;
+--------------------+
| Database           	   |
+--------------------+
| information_schema  |
| mysql              	   |
| performance_schema |
| sampledb           	   |	<--- YAML에서 만든 DATABASE
| sys                	   |
+--------------------+
5 rows in set (0.00 sec)

mysql>


mysql> USE sampledb;
Database changed


mysql> CREATE TABLE test (
    ->    id INT AUTO_INCREMENT PRIMARY KEY,
    ->    msg VARCHAR(50)
    ->    );
Query OK, 0 rows affected (0.01 sec)


[guest@Server-A step1-mysql-pull]$ sudo ls  /var/lib/docker/volumes/step1-mysql-pull_dbdata/_data
'#ib_16384_0.dblwr'	  binlog.000002  	ib_buffer_pool   	performance_schema	sys
'#ib_16384_1.dblwr'	  binlog.index      	ibdata1          	private_key.pem      	undo_001
'#innodb_redo'     	  ca-key.pem        	ibtmp1           	public_key.pem       	undo_002
'#innodb_temp'   	  ca.pem            	mysql            	sampledb
 auto.cnf           	  client-cert.pem   	mysql.ibd        	server-cert.pem
 binlog.000001     	  client-key.pem    	mysql.sock	server-key.pem


[guest@Server-A step1-mysql-pull]$ docker compose down
[+] down 2/2
 ✔ Container mysql_db                    Removed                                                      3.0s
 ✔ Network step1-mysql-pull_default Removed                                                       0.1s
[guest@Server-A step1-mysql-pull]$
```

---

## step1: nginx with Dockerfile + build

- Dockerfile로 Nginx 이미지(`mynginx:v1.0`) 생성
- `docker-compose.yaml`에서 `build` 설정으로 이미지를 빌드
- 같은 compose로 컨테이너까지 띄운다.
- 브라우저에서 HTML 페이지가 보이는지 확인

구조
- 디렉터리: `/compose-lab/step1-nginx`
- 파일: `Dockerfile`, `docker-compose.yaml`, `html/index.html`

```bash
[guest@Server-A ~]$ sudo  mkdir  -p  /compose-lab/step1-nginx/html

[guest@Server-A ~]$ cd /compose-lab/step1-nginx

[guest@Server-A step1-nginx]$ pwd
/compose-lab/step1-nginx


[guest@Server-A step1-nginx]$ sudo vi  ./html/index.html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <title>Docker Compose Step1 - My Nginx</title>
</head>
<body>
    <h1>Step1: My Custom Nginx Image</h1>
    <p>이 페이지는 내가 만든 Docker 이미지에서 제공되는 페이지입니다.</p>
</body>
</html>


[guest@Server-A step1-nginx]$ ls  -l  ./html/
합계 4
-rw-r--r-- 1 root root 279  8월 10 12:53 index.html


# Dockerfile 생성
[guest@Server-A step1-nginx]$ sudo vi dockerfile
FROM  nginx:latest			# nginx 공식 이미지를 기반으로 이미지 생성

ENV  LANG=C.UTF-8		# 한글 사용을 위한 환경변수
ENV  LC_ALL=C.UTF-8		# 한글 사용을 위한 환경변수

COPY  ./html  /usr/share/nginx/html	# 호스트의 index.html을 컨테이너의 "/usr/share/nginx/html" 디렉터리로 복사

: wq


-/compose-lab/step1-nginx		: dockerfile
-/compose-lab/step1-nginx/html	: index.html


[guest@Server-A step1-nginx]$ ls -lR
.:
합계 4
-rw-r--r-- 1 root root 327  8월 10 13:02 dockerfile
drwxr-xr-x 2 root root  24  8월 10 12:53 html

./html:
합계 4
-rw-r--r-- 1 root root 279  8월 10 12:53 index.html


# Docker-compose 생성
[guest@Server-A step1-nginx]$ sudo vi docker-compose.yaml
services:
  NginxWeb:
    build:
      context:  .
      dockerfile: dockerfile
    image: mynginx:v1.0
    container_name: nginx_build
    ports:
      - "8080:80"

: wq


[guest@Server-A step1-nginx]$ docker  compose  config
name: step1-nginx
services:
  NginxWeb:
    build:
      context: /compose-lab/step1-nginx
      dockerfile: dockerfile
    container_name: nginx_build
    image: mynginx:v1.0
    networks:
      default: null
    ports:
      - mode: ingress
        target: 80
        published: "8080"
        protocol: tcp
networks:
  default:
    name: step1-nginx_default


[guest@Server-A step1-nginx]$ docker  compose  build	# docker compose로 이미지 생성
[guest@Server-A step1-nginx]$ docker  compose  up  -d	# docker compose 파일안의 이미지로 컨테이너 생성

[guest@Server-A step1-nginx]$ docker  compose  up  -d  --build	# 이미지 build 후 컨테이너 생성


[guest@Server-A step1-nginx]$ docker  compose  up  -d  --build
[+] Building 0.2s (9/9) FINISHED
 => [internal] load local bake definitions                                                 	0.0s
 => => reading from stdin 508B                                                                	0.0s
 => [internal] load build definition from dockerfile                                      	0.0s
 => => transferring dockerfile: 132B                                                          	0.0s
 => [internal] load metadata for docker.io/library/nginx:latest                     	0.0s
 => [internal] load .dockerignore                                                                0.0s
 => => transferring context: 2B                                                                  0.0s
 => CACHED [1/2] FROM docker.io/library/nginx:latest@sha256:8541484afb	0.0s
 => => resolve docker.io/library/nginx:latest@sha256:8541484afbc9c8a5a8a9	0.0s
 => [internal] load build context                                                                	0.0s
 => => transferring context: 351B                                                              	0.0s
 => [2/2] COPY  ./html  /usr/share/nginx/html                                       	0.0s
 => exporting to image                                                                           	0.1s
 => => exporting layers                                                                          	0.0s
 => => exporting manifest sha256:647160bf17a76c635081fe9447a04e89aa77380e	 0.0s
 => => exporting config sha256:7aa9377b9e35452c2c88f70252470465950db90e84	0.0s
 => => exporting attestation manifest sha256:af4839f464160e5a2eb136cea06d0e	0.0s
 => => exporting manifest list sha256:4eb3e8067024150638684ad0e60bcfb61d1a	0.0s
 => => naming to docker.io/library/mynginx:v1.0                                       	0.0s
 => => unpacking to docker.io/library/mynginx:v1.0                                       	0.0s
 => resolving provenance for metadata file                                                	0.0s
[+] up 3/3
 ✔ Image mynginx:v1.0          Built                                                     	0.2s
 ✔ Network step1-nginx_default Created                                                	0.1s
 ✔ Container nginx_build       Started                                                    	0.2s
[guest@Server-A step1-nginx]$


[guest@Server-A step1-nginx]$ docker images
IMAGE             	ID             	DISK USAGE   	CONTENT SIZE   	EXTRA
httpd:latest         	2920ed858727        	175MB         	47.6MB
memtest:latest    	4f93aad3d9ae        	530MB          	136MB
mynginx:v1.0         	4eb3e8067024        	235MB         	63.1MB    	U
mysql:8.0            	7dcddc01f13b        	1.1GB          	249MB
mysql:latest         	66aec17cd21a       	1.29GB          	290MB
nginx:latest         	8541484afbc9        	238MB           	66MB
registry:3           	1be55279f18a       	83.3MB         	20.4MB
webp:1.0             	371d663bf0bf        	235MB         	63.1MB


[guest@Server-A step1-nginx]$ docker compose ps
NAME          IMAGE           COMMAND                   SERVICE    CREATED         STATUS         PORTS
nginx_build    mynginx:v1.0   "/docker-entrypoint.…"   NginxWeb   2 minutes ago    Up 2 minutes    0.0.0.0:8080->80/tcp, [::]:8080->80/tcp


# 웹 브라우저를 사용하여 확인
http://192.168.10.100:8080/
Step1: My Custom Nginx Image
이 페이지는 내가 만든 Docker 이미지에서 제공되는 페이지입니다.


# curl 명령어를 사용하여 확인
guest@Server-B:~$ curl 192.168.10.100:8080
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <title>Docker Compose Step1 - My Nginx</title>
</head>
<body>
    <h1>Step1: My Custom Nginx Image</h1>
    <p>이 페이지는 내가 만든 Docker 이미지에서 제공되는 페이지입니다.</p>
</body>
</html>


[guest@Server-A step1-nginx]$ docker compose down
[+] down 2/2
 ✔ Container nginx_build       Removed                                      	0.2s
 ✔ Network step1-nginx_default Removed                                	0.1s
```

**정리**: nginx, httpd, MySQL(pull 이미지) 단일 컨테이너 실행과, Dockerfile을 빌드해서 실행하는 방식까지 step1 실습을 마쳤다. 이어서 여러 컨테이너를 함께 정의하는 step2 실습(nginx 2개, MySQL+phpMyAdmin)을 진행한다.

---

## step2: nginx 2개 컨테이너

- `docker-compose.yaml` 안에 서비스 2개(web1, web2)를 정의한다.
- `docker compose up -d` 명령 한 번으로 컨테이너 2개를 동시에 띄운다.
- 브라우저에서 8081, 8082로 각각 접속해 확인한다.
- `docker compose down`으로 한 번에 정리한다.

구조
- 작업 폴더: `/compose-lab/step2/nginx2`
- 파일: `docker-compose.yaml`
- 서비스 이름: web1, web2
- 이미지: web1 `nginx:latest`, web2 `nginx:1.31`
- 포트: web1 `8081:80`, `8083:80` / web2 `8082:80`, `8084:80`

```bash
[guest@Server-A ~]$ sudo mkdir  -p  /compose-lab/step2/nginx2

[guest@Server-A ~]$ ls -l  /compose-lab/
합계 0
drwxr-xr-x 2 root root 33  8월 10 10:49 step1
drwxr-xr-x 2 root root 57  8월 10 11:49 step1-httpd
drwxr-xr-x 2 root root 33  8월 10 12:16 step1-mysql-pull
drwxr-xr-x 3 root root 63  8월 10 13:10 step1-nginx
drwxr-xr-x 2 root root  6  8월 10 12:50 step1-nginx-dockerfile
drwxr-xr-x 3 root root 20  8월 10 14:35 step2


[guest@Server-A ~]$ cd  /compose-lab/step2/nginx2/

[guest@Server-A nginx2]$ pwd
/compose-lab/step2/nginx2


[guest@Server-A nginx2]$ sudo vi docker-compose.yaml
services:
  web1:
    image: nginx:latest
    container_name: step2-nginx-web1
    ports:
      - "8081:80"
      - "8083:80"

  web2:
    image: nginx:1.31
    container_name: step2-nginx-web2
    ports:
      - "8082:80"
      - "8084:80"


name: nginx2
services:
  web1:
    container_name: step2-nginx-web1
    image: nginx-latest
    networks:
      default: null
    ports:
      - mode: ingress
        target: 80
        published: "8081"
        protocol: tcp
      - mode: ingress
        target: 80
        published: "8083"
        protocol: tcp
  web2:
    container_name: step2-nginx-web2
    image: nginx-1.31
    networks:
      default: null
    ports:
      - mode: ingress
        target: 80
        published: "8082"
        protocol: tcp
      - mode: ingress
        target: 80
        published: "8084"
        protocol: tcp
networks:
  default:
    name: nginx2_default


[guest@Server-A nginx2]$ docker  compose  up  -d
[+] up 4/4
 ✔ Image nginx:1.31           		Pulled                               	3.9s
 ✔ Network nginx2_default     	Created                             	0.1s
 ✔ Container step2-nginx-web2 	Started                         	0.4s
 ✔ Container step2-nginx-web1 	Started                           	0.3s
[guest@Server-A nginx2]$


[guest@Server-A nginx2]$ docker  compose  ps
NAME                  IMAGE          COMMAND                   SERVICE   CREATED          STATUS          PORTS
step2-nginx-web1   nginx:latest    "/docker-entrypoint.…"   web1         41 seconds ago    Up 40 seconds   0.0.0.0:8081->80/tcp, 0.0.0.0:8083->80/tcp
step2-nginx-web2   nginx:1.31      "/docker-entrypoint.…"   web2         41 seconds ago    Up 40 seconds   0.0.0.0:8082->80/tcp, 0.0.0.0:8084->80/tcp


[guest@Server-A nginx2]$ docker  compose  logs  web1		# 브라우저 접속 전
step2-nginx-web1  | /docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
step2-nginx-web1  | /docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
step2-nginx-web1  | /docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
step2-nginx-web1  | 10-listen-on-ipv6-by-default.sh: info: Getting the checksum of /etc/nginx/conf.d/default.conf
step2-nginx-web1  | 10-listen-on-ipv6-by-default.sh: info: Enabled listen on IPv6 in /etc/nginx/conf.d/default.conf
step2-nginx-web1  | /docker-entrypoint.sh: Sourcing /docker-entrypoint.d/15-local-resolvers.envsh
step2-nginx-web1  | /docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
step2-nginx-web1  | /docker-entrypoint.sh: Launching /docker-entrypoint.d/30-tune-worker-processes.sh
step2-nginx-web1  | /docker-entrypoint.sh: Configuration complete; ready for start up
step2-nginx-web1  | 2026/08/10 05:43:29 [notice] 1#1: using the "epoll" event method
step2-nginx-web1  | 2026/08/10 05:43:29 [notice] 1#1: nginx/1.31.3
step2-nginx-web1  | 2026/08/10 05:43:29 [notice] 1#1: built by gcc 14.2.0 (Debian 14.2.0-19)
step2-nginx-web1  | 2026/08/10 05:43:29 [notice] 1#1: OS: Linux 5.14.0-687.17.1.el9_8.x86_64
step2-nginx-web1  | 2026/08/10 05:43:29 [notice] 1#1: getrlimit(RLIMIT_NOFILE): 1024:524288
step2-nginx-web1  | 2026/08/10 05:43:29 [notice] 1#1: start worker processes
step2-nginx-web1  | 2026/08/10 05:43:29 [notice] 1#1: start worker process 29
step2-nginx-web1  | 2026/08/10 05:43:29 [notice] 1#1: start worker process 30
step2-nginx-web1  | 2026/08/10 05:43:29 [notice] 1#1: start worker process 31
step2-nginx-web1  | 2026/08/10 05:43:29 [notice] 1#1: start worker process 32


[guest@Server-A nginx2]$ docker  compose  logs  web1		# 브라우저 접속 후
...(생략)...
step2-nginx-web1  | 192.168.10.1 - - [10/Aug/2026:05:46:15 +0000] "GET / HTTP/1.1" 200 896 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/151.0.0.0 Safari/537.36" "-"
step2-nginx-web1  | 2026/08/10 05:46:15 [error] 29#29: *1 open() "/usr/share/nginx/html/favicon.ico" failed (2: No such file or directory), client: 192.168.10.1, server: localhost, request: "GET /favicon.ico HTTP/1.1", host: "192.168.10.100:8081", referrer: "http://192.168.10.100:8081/"
step2-nginx-web1  | 192.168.10.1 - - [10/Aug/2026:05:46:15 +0000] "GET /favicon.ico HTTP/1.1" 404 555 "http://192.168.10.100:8081/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/151.0.0.0 Safari/537.36" "-"


[guest@Server-A nginx2]$ docker  compose  logs  web2		# 브라우저 접속 전
...(web1과 동일한 시작 로그)...

[guest@Server-A nginx2]$ docker  compose  logs  web2		# 브라우저 접속 후
step2-nginx-web2  | 192.168.10.1 - - [10/Aug/2026:05:49:46 +0000] "GET / HTTP/1.1" 200 896 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/151.0.0.0 Safari/537.36" "-"
step2-nginx-web2  | 192.168.10.1 - - [10/Aug/2026:05:49:46 +0000] "GET /favicon.ico HTTP/1.1" 404 555 "http://192.168.10.100:8082/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/151.0.0.0 Safari/537.36" "-"
step2-nginx-web2  | 2026/08/10 05:49:46 [error] 29#29: *1 open() "/usr/share/nginx/html/favicon.ico" failed (2: No such file or directory), client: 192.168.10.1, server: localhost, request: "GET /favicon.ico HTTP/1.1", host: "192.168.10.100:8082", referrer: "http://192.168.10.100:8082/"


# curl을 사용하여 확인
curl http://192.168.10.100:8081
curl http://192.168.10.100:8083

curl http://192.168.10.100:8082
curl http://192.168.10.100:8084


[guest@Server-A nginx2]$ docker  compose  down
[+] down 3/3
 ✔ Container step2-nginx-web1 Removed                                                                                                     0.2s
 ✔ Container step2-nginx-web2 Removed                                                                                                     0.3s
 ✔ Network nginx2_default     Removed
```

### 서로 다른 웹서버 테스트를 위해서 web1, web2의 HTML 코드 변경

```bash
# web1
[guest@Server-A nginx2]$ docker  exec  -it  step2-nginx-web1  /bin/bash
root@bbc4d4c7c317:/#

root@261c47aed0c5:/# echo "<h1> Step2-Nginx-web1 WEB Server 1 Test </h1>"  >  /usr/share/nginx/html/index.html

root@261c47aed0c5:/# cat  /usr/share/nginx/html/index.html
<h1> Step2-Nginx-web1 WEB Server 1 Test </h1>


# web2
[guest@Server-A nginx2]$ docker  exec  -it  step2-nginx-web2  /bin/bash
root@bbc4d4c7c317:/#

root@10e1248d5497:/# echo "<h1> Step2-Nginx-web2 WEB Server 2 Test </h1>"  >  /usr/share/nginx/html/index.html

root@10e1248d5497:/#  cat  /usr/share/nginx/html/index.html
<h1> Step2-Nginx-web2 WEB Server 2 Test </h1>


http://192.168.10.100:8081
http://192.168.10.100:8083

guest@Server-B:~$ curl  http://192.168.10.100:8081
<h1> step2-nginx-web2 WEB1 TEST </h1>

guest@Server-B:~$ curl  http://192.168.10.100:8083
<h1> step2-nginx-web2 WEB1 TEST </h1>


http://192.168.10.100:8082
http://192.168.10.100:8084

guest@Server-B:~$ curl  http://192.168.10.100:8082
<h1> step2-nginx-web2 WEB2 TEST </h1>

guest@Server-B:~$ curl  http://192.168.10.100:8084
<h1> step2-nginx-web2 WEB2 TEST </h1>
```

---

## step2: nginx 2개 컨테이너 (Dockerfile build)

- Dockerfile로 `mynginx:step2` 이미지를 만든다.
- `docker-compose.yaml`에서 web1, web2 두 서비스를 정의한다.
- 같은 이미지를 쓰는 컨테이너 2개를 포트만 다르게 동시에 실행한다.
- 필요하면 특정 컨테이너만 삭제/재생성도 테스트한다.

구조
- 작업 디렉터리: `/compose-lab/step2/web2`
- 파일: `dockerfile`, `docker-compose.yaml`, `html/index.html`
- 이미지 이름: `mynginx:v12.1`
- 컨테이너 이름: `my_nginx_web1`, `my_nginx_web2`
- 포트: web1 `8088:80`, web2 `8808:80`

```bash
# 작업 디렉터리 생성
[guest@Server-A ~]$ sudo mkdir /compose-lab/step2/web2
mkdir: `/compose-lab/step2/web2' 디렉토리를 만들 수 없습니다: 파일이 있습니다

[guest@Server-A ~]$ cd  /compose-lab/step2/web2

[guest@Server-A web2]$ pwd
/compose-lab/step2/web2


[guest@Server-A web2]$ sudo mkdir html	# html을 저장하는 디렉터리 생성

[guest@Server-A web2]$ ls  -lR
.:
합계 0
drwxr-xr-x 2 root root 6  8월 10 15:07 html

./html:
합계 0


[guest@Server-A web2]$ sudo vi  html/index.html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <title>Docker Compose Step2 - My Nginx</title>
</head>
<body>
    <h1>Step2: My Custom Nginx Image (web1 / web2)</h1>
    <p>이 페이지는 mynginx:step2 이미지를 사용하는 컨테이너에서 제공됩니다.</p>
</body>
</html>


[guest@Server-A web2]$ sudo  vi dockerfile
FROM nginx:latest

ENV LANG=C.UTF-8
ENV LC_ALL=C.UTF-8

COPY  ./html  /usr/share/nginx/html

:wq


-/compose-lab/step2/web2		# dockerfile
-/compose-lab/step2/web2/html	# index.html


[guest@Server-A web2]$ ls -l
합계 4
-rw-r--r-- 1 root root 90  8월 10 15:11 dockerfile
drwxr-xr-x 2 root root 24  8월 10 15:08 html


# YAML 파일 작성
[guest@Server-A web2]$ vi  docker-compose.yaml
services:
  web1:
    build:
      context: .
      dockerfile: dockerfile

    image: mynginx:v12.1
    container_name: my_nginx_web1

    ports:
      - "8088:80"


  web2:
    image: mynginx:v12.1
    container_name: my_nginx_web2

    ports:
      - "8808:80"

:wq


[guest@Server-A web2]$ docker  compose  config
name: web2
services:
  web1:
    build:
      context: /compose-lab/step2/web2
      dockerfile: dockerfile
    container_name: my_nginx_web1
    image: mynginx:v12.1
    networks:
      default: null
    ports:
      - mode: ingress
        target: 80
        published: "8088"
        protocol: tcp
  web2:
    container_name: my_nginx_web2
    image: mynginx:v12.1
    networks:
      default: null
    ports:
      - mode: ingress
        target: 80
        published: "8808"
        protocol: tcp
networks:
  default:
    name: web2_default


[guest@Server-A web2]$ docker  compose  up  -d  --build
[+] up 0/1
 ⠼ Image mynginx:v12.1 Pulling                                                    		1.5s
[+] Building 0.2s (9/9) FINISHED
 => [internal] load local bake definitions                                       		0.0s
 => => reading from stdin 488B                                                   		0.0s
 => [internal] load build definition from dockerfile                             	0.0s
 => => transferring dockerfile: 127B                                             		0.0s
 => [internal] load metadata for docker.io/library/nginx:latest                  	0.0s
 => [internal] load .dockerignore                                                		0.0s
 => => transferring context: 2B                                                  		0.0s
 => [internal] load build context                                                		0.0s
 => => transferring context: 371B                                                		0.0s
 => CACHED [1/2] FROM docker.io/library/nginx:latest@sha256:8541484a	0.0s
 => => resolve docker.io/library/nginx:latest@sha256:8541484afbc9c8a5a8a	0.0s
 => [2/2] COPY ./html /usr/share/nginx/html                                      	0.0s
 => exporting to image                                                           		0.1s
 => => exporting layers                                                          		0.0s
 => => exporting manifest sha256:6044fdc6197c928468e17f173c7e37c5c917e	0.0s
 => => exporting config sha256:d3ee6f291328f9fc62e5088c4072b7aee59399cf	0.0s
 => => exporting attestation manifest sha256:61035ccd4ce62b8213fc372f5d7	0.0s
 => => exporting manifest list sha256:b9610f925c66d047938e469bfaeccc618	0.0s
 => => naming to docker.io/library/mynginx:v12.1                                 	0.0s
[+] up 4/4acking to docker.io/library/mynginx:v12.1                              	0.0s
 ✔ Image mynginx:v12.1     Built                                                  		1.9s
 ✔ Network web2_default    Created                                                	0.1s
 ✔ Container my_nginx_web2 Started                                                	0.3s
 ✔ Container my_nginx_web1 Started                                                	0.3s


[guest@Server-A web2]$ docker  compose  ps
NAME               IMAGE             COMMAND                  SERVICE   CREATED              STATUS                PORTS
my_nginx_web1   mynginx:v12.1   "/docker-entrypoint.…"   web1        About a minute ago   Up About a minute   0.0.0.0:8088->80/tcp, [::]:8088->80/tcp
my_nginx_web2   mynginx:v12.1   "/docker-entrypoint.…"   web2        About a minute ago   Up About a minute   0.0.0.0:8808->80/tcp, [::]:8808->80/tcp


[guest@Server-A web2]$ docker  compose  down
[+] down 3/3
 ✔ Container my_nginx_web1 Removed                                                0.1s
 ✔ Container my_nginx_web2 Removed                                                0.2s
 ✔ Network web2_default    Removed                                                0.1s
```

---

## step2: MySQL + phpMyAdmin 2-container

- `docker-compose.yaml` 하나로 MySQL 컨테이너 1개 + phpMyAdmin 컨테이너 1개 총 2개를 동시에 띄운다.
- phpMyAdmin은 웹 브라우저에서 MySQL 또는 MariaDB를 관리할 수 있게 해주는 웹 기반 관리 도구
- phpMyAdmin 웹 화면에서 MySQL에 접속해서 DB/테이블을 확인한다.

phpMyAdmin에서 할 수 있는 작업
- 데이터베이스 생성/삭제
- 테이블 생성/수정/삭제
- 데이터 조회, 추가, 수정, 삭제
- SQL문 직접 실행
- 사용자 및 권한 관리
- 테이블 구조 확인

- 작업 디렉터리: `/compose-lab/step2/mysql-pma`
- 파일: `docker-compose.yaml`
- 컨테이너: db → MySQL(`step2-mysql-db`), pma → phpMyAdmin(`step2-mysql-pma`)
- 포트: MySQL 호스트 3308 → 컨테이너 3306, phpMyAdmin 호스트 8086 → 컨테이너 80 (포트는 겹치지 않게 3308, 8086으로 사용)

```bash
# 디렉터리 준비
[root@rocky ~]# sudo mkdir  -p  /compose-lab/step2/mysql-pma

[root@rocky ~]# cd  /compose-lab/step2/mysql-pma

[guest@Server-A mysql-pma]$ pwd
/compose-lab/step2/mysql-pma


# docker-compose.yaml 작성
[guest@Server-A mysql-pma]$ sudo  vi  docker-compose.yaml
services:
  db:
    image: mysql:8.0
    container_name: step2-mysql-db

    environment:
      MYSQL_ROOT_PASSWORD: "admin1234"
      MYSQL_DATABASE: "sampledb"
      MYSQL_USER: "testuser"
      MYSQL_PASSWORD: "1234"

    ports:
      - "3308:3306"

    volumes:
      - "mydbdata:/var/lib/mysql"

  pma:
    image: phpmyadmin/phpmyadmin:latest
    container_name: step2-mysql-pma
    environment:
      PMA_HOST: db
      PMA_PORT: 3306
      PMA_USER: root
      PMA_PASSWORD: "admin1234"

    ports:
      - "8086:80"
    depends_on:
      - db

volumes:
  mydbdata:

: wq
```

- `mydbdata`의 실제 경로 = `/var/lib/docker/volumes/mydbdata/_data`

```bash
[guest@Server-A mysql-pma]$ docker  compose  config
name: mysql-pma
services:
  db:
    container_name: step2-mysql-db
    environment:
      MYSQL_DATABASE: sampledb
      MYSQL_PASSWORD: "1234"
      MYSQL_ROOT_PASSWORD: admin1234
      MYSQL_USER: testuser
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
        volume: {}
  pma:
    container_name: step2-mysql-pma
    depends_on:
      db:
        condition: service_started
        required: true
    environment:
      PMA_HOST: db
      PMA_PASSWORD: admin1234
      PMA_PORT: "3306"
      PMA_USER: root
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
 ✔ Image phpmyadmin/phpmyadmin:latest Pulled                                 	13.7s
 ✔ Network mysql-pma_default 	Created                                 	0.1s
 ✔ Volume mysql-pma_mydbdata         	Created                                  	0.0s
 ✔ Container step2-mysql-db           	Started                                  	0.2s
 ✔ Container step2-mysql-pma          	Started                                 	0.3s
[guest@Server-A mysql-pma]$


[guest@Server-A mysql-pma]$ docker  compose  ps
NAME                  IMAGE                                  COMMAND                   SERVICE   CREATED         STATUS         PORTS
step2-mysql-db      mysql:8.0                              "docker-entrypoint.s…"   db              3 seconds ago    Up 3 seconds   33060/tcp, 0.0.0.0:3308->3306/tcp
step2-mysql-pma   phpmyadmin/phpmyadmin:latest   "/docker-entrypoint.…"   pma           3 seconds ago    Up 3 seconds   0.0.0.0:8086->80/tcp


[guest@Server-A mysql-pma]$ docker  exec -it  step2-mysql-db  /bin/bash
bash-5.1#

bash-5.1# mysql -u root -p
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 86
Server version: 8.0.46 MySQL Community Server - GPL

Copyright (c) 2000, 2026, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

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
You can turn off this feature to get a quicker startup with -A

Database changed
mysql>
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
 ✔ Container step2-mysql-pma Removed                                              1.2s
 ✔ Container step2-mysql-db  Removed                                              1.5s
 ✔ Network mysql-pma_default Removed                                              0.1s
[guest@Server-A mysql-pma]$
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

- 디렉터리: `/compose-lab/step3-1`
- 구성: `proxy/`(Dockerfile, nginx.conf), `app1/`(Dockerfile, html/index.html), `app2/`(Dockerfile, html/index.html), `docker-compose.yaml`

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

[guest@Server-A step3-1]$ ls -l  app1
합계 0
drwxr-xr-x 2 root root 6  8월 10 16:47 html

[guest@Server-A step3-1]$ ls -l  app2
합계 0
drwxr-xr-x 2 root root 6  8월 10 16:47 html

[guest@Server-A step3-1]$ ls -l  proxy/
합계 0
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


[guest@Server-A step3-1]$ ls  -lR ./app1
./app1:
합계 4
-rw-r--r-- 1 root root 53  8월 10 16:53 dockerfile
drwxr-xr-x 2 root root 24  8월 10 16:52 html

./app1/html:
합계 4
-rw-r--r-- 1 root root 243  8월 10 16:52 index.html
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


[guest@Server-A step3-1]$ ls  -lR ./app2
./app2:
합계 4
-rw-r--r-- 1 root root 53  8월 10 16:57 dockerfile
drwxr-xr-x 2 root root 24  8월 10 16:57 html

./app2/html:
합계 4
-rw-r--r-- 1 root root 243  8월 10 16:57 index.html
```

### reverse proxy 설정 (경로 기반)

```bash
[root@rocky step3-1]# sudo vi /compose-lab/step3-1/proxy/nginx.conf
worker_processes 1;			# NGINX가 사용할 워커 프로세스 개수 (일반적으로 CPU 코어 개수와 동일하게 설정)

events {
    worker_connections 1024;		# 하나의 워커 프로세스가 처리할 최대 연결 개수
}

http {
    include  /etc/nginx/mime.types;	# MIME 타입을 정의 (.html --> text/html | .jpg --> image/jpeg | .png --> image/png)
    default_type  application/octet-stream;
    charset utf-8;

    sendfile  on;			# 파일을 빠르게 전송하기위한 sendfile 기능 활성화

    server {
        listen 80;			# 외부 클라이언트로부터 수신할 포트번호

        location /app1/ {		# EX) 192.168.10.100/app1
            proxy_pass http://app1/;
        }
        location /app2/ { 		# EX) 192.168.10.100/app2
            proxy_pass http://app2/;
        }
        location / {
            return 200 'Use /app1 or /app2';
        }
    }
}

: wq


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

# prune	:  사용하지 않는 Docker 이미지를 정리하는 명령어
# -a 	: 컨테이너에서 사용하지 않는 모든 이미지 삭제
# -f 	: 삭제 확인 질문 없이 바로 실행


[guest@Server-A step3-1]$ docker compose up -d
 ✔ Network step3-1_default	Created                           	0.3s
 ✔ Container step3-1-app2 	Started                           	0.8s
 ✔ Container step3-1-app1  	Started                             	0.8s
 ✔ Container step3-1-proxy 	Started                             	1.3s


http://192.168.10.100/app1/	# 접속

http://192.168.10.100/app2/	# 접속


guest@Server-B:~$ curl http://localhost/app1/
<h1>THIS IS APP1 SERVER</h1>
<p>이 페이지는 app1 컨테이너 이미지 안에 포함된 HTML입니다.</p>


guest@Server-B:~$ curl http://localhost/app2/
<h1>THIS IS APP2 SERVER</h1>
<p>이 페이지는 app2 컨테이너 이미지 안에 포함된 HTML입니다.</p>


[guest@Server-A step3-1]$ docker compose down
[+] down 4/4
 ✔ Container step3-1-proxy Removed                                                0.2s
 ✔ Container step3-1-app2  Removed                                                0.1s
 ✔ Container step3-1-app1  Removed                                                0.1s
 ✔ Network step3-1_default Removed
```

- app1으로 접속시 = `http://192.168.10.100/app1/`
- app2으로 접속시 = `http://192.168.10.100/app2/`
- 각각의 web server로 접속시 uri를 서로 다르게 작성하게 되면 proxy 서버를 사용하는 의미가 없다.
- proxy 서버 구성시 라운드 로빈 방식을 사용하게 되면 proxy 서버가 각각의 서버로 번갈아 가면서 연결한다.

### proxy 서버 수정 — 라운드 로빈 로드밸런싱

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

- `upstream backend` : web1, web2 두 웹 서버를 하나의 그룹으로 묶어 로드밸런싱
- 주요 옵션
  - `server web1;` : 기본 서버 등록
  - `server web1 weight=2;` : 가중치(트래픽 2배 받음)
  - `server web1 max_fails=3 fail_timeout=30s;` : 3번 실패하면 30초 동안 제외
- 로드밸런싱 알고리즘
  - 기본: round-robin (순차)
  - `least_conn;` : 연결 수가 가장 적은 서버로 전달
  - `ip_hash;` : 같은 클라이언트 IP는 항상 같은 서버로 보냄(세션 유지)

```bash
[guest@Server-A step3-1]$ docker  image  prune -a -f

# prune	:  사용하지 않는 Docker 이미지를 정리하는 명령어
# -a 	: 컨테이너에서 사용하지 않는 모든 이미지 삭제
# -f 	: 삭제 확인 질문 없이 바로 실행


[guest@Server-A step3-1]$ docker  compose  up  -d


[guest@Server-A step3-1]$ docker  compose  ps
NAME            	IMAGE            COMMAND                   SERVICE   CREATED         STATUS         PORTS
step3-1-app1	step3-1-app1     "/docker-entrypoint.…"   app1         8 seconds ago    Up 8 seconds   80/tcp
step3-1-app2    	step3-1-app2     "/docker-entrypoint.…"   app2         8 seconds ago    Up 8 seconds   80/tcp
step3-1-proxy   	step3-1-proxy   "/docker-entrypoint.…"    proxy       8 seconds ago   Up 8 seconds   0.0.0.0:80->80/tcp, [::]:80->80/tcp
```


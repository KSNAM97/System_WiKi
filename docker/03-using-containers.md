# Docker 03 — 컨테이너 사용하기

## Container Lifecycle

- **컨테이너 라이프사이클**이란 도커 이미지로부터 컨테이너가 생성되고 실행되고 종료되기까지의 전체 흐름을 의미하며, `docker create`/`docker start`로 생성과 실행 단계를 나누거나 `docker exec`로 실행 중인 컨테이너 내부에 접속하는 등 각 단계마다 별도의 명령으로 다룰 수 있다.
- 즉, 컨테이너가 생성되고, 실행되고, 사라지는 과정이다.

**1단계 이미지 준비 단계 (Image Pull / Local Image)**
- 컨테이너는 이미지를 기반으로 만들어진다.
- 이미지는 미리 만들어진 프로그램 패키지(운영체제, 실행파일, 설정 등)를 포함한다.

```bash
docker pull nginx
docker pull ubuntu:20.04
docker images
```

**2단계 컨테이너 생성 단계 (Create)**
- 컨테이너는 run을 하기 전 '생성(create)' 단계가 존재한다.
- 다만 `docker run`은 생성(create) + 실행(start)을 동시에 진행한다.
- 컨테이너 생성만 하고 실행은 하지 않을 때:

```bash
docker create --name test nginx
docker ps -a    # 컨테이너 목록 확인(종료 포함)
```

**3단계 컨테이너 실행 단계 (Start)**
- 생성된 컨테이너 시작
- `start`는 컴퓨터에서 전원 버튼 누르는 것과 같다.

```bash
docker start test
# 신규 컨테이너를 바로 실행
docker run -d -p 80:80 --name web nginx
```

**4단계 일시중지(Pause) / 재개(Unpause)**
- 프로세스를 잠시 멈출 수도 있다.
- Pause는 CPU 사용을 멈추는 개념이며 네트워크, 메모리는 유지된다.

```bash
docker pause web
docker unpause web
```

**5단계 컨테이너 내부 작업 단계 (Exec / Attach)**
- 컨테이너 내부로 들어가 명령을 실행한다.

```bash
docker exec -it web /bin/bash    # 컨테이너 내부 쉘 접속
docker exec -it web /bin/sh
docker logs web                  # 컨테이너 로그 확인
```

**6단계 정지(Stop) / 강제 종료(Kill)**
```bash
docker stop web    # 정상 종료 (10초 정도 기다린 후 종료)
docker kill web    # 강제 종료 (기다리지 않고 바로 종료)
```

**7단계 삭제(Remove)**
```bash
docker rm web      # 컨테이너 삭제 (실행 중에는 먼저 stop 해야 함)
docker rmi nginx   # 이미지 삭제 (해당 이미지를 사용하는 모든 컨테이너가 삭제되어야 함)
```

**정리**: 컨테이너는 이미지 준비 → 생성 → 실행 → 일시중지/재개 → 내부 작업 → 정지/강제종료 → 삭제의 라이프사이클을 거친다. 이어지는 절에서는 create/start를 나누어 실행하는 실습을 진행한다.

---

## 컨테이너 생성 및 실행 (Create, Start)

```bash
[guest@Server-A ~]$ docker  create  --name httpd_web  -p 80:80  httpd:latest
5b4cfeb95032d267a3f2ceefed78f3dc5b0d1f0014bc87107e0b57b02f9cdcff


[guest@Server-A ~]$ docker ps -a
CONTAINER ID   IMAGE          COMMAND              CREATED          STATUS    PORTS     NAMES
5b4cfeb95032   httpd:latest   "httpd-foreground"   10 seconds ago   Created             httpd_web
```

```bash
[guest@Server-A ~]$ docker start  httpd_web
httpd_web


[guest@Server-A ~]$ docker ps -a
CONTAINER ID   IMAGE          COMMAND              CREATED              STATUS          PORTS                               NAMES
5b4cfeb95032   httpd:latest   "httpd-foreground"   About a minute ago   Up 13 seconds   0.0.0.0:80->80/tcp, [::]:80->80/tcp   httpd_web
```

```bash
[guest@Server-A ~]$ docker  inspect  --format '{{.NetworkSettings.Networks.bridge.IPAddress}}' httpd_web
172.17.0.2


[guest@Server-A ~]$ docker  inspect  --format '{{.NetworkSettings.Networks.bridge.Gateway}}' httpd_web
172.17.0.1
```

```bash
[guest@Server-A ~]$ docker  stop  httpd_web
httpd_web

[guest@Server-A ~]$ docker  rm  httpd_web
```

**정리**: `docker create`로 컨테이너를 만들고 `docker start`로 실행하는 2단계 방식과 IP 확인(`docker inspect`)을 실습했다. 다음은 `docker run`과 `docker exec`를 함께 사용하는 방식이다.

---

## 컨테이너 생성 및 실행 (run + exec)

```bash
# 컨테이너 생성 및 실행
[guest@Server-A ~]$ docker  run  -d  --name nginx_web  -p 80:80   nginx:latest
5e4cf32df303e06aa8f52b177aa6c0fcf1c827e6b27d08fb00df7569358c87ca


# 컨테이너 접속
[guest@Server-A ~]$ docker  exec  -it  nginx_web  /bin/bash
root@5e4cf32df303:/#
```

옵션 설명
- `docker exec` : 이미 실행 중인 컨테이너 안에서 명령어를 실행하는 기능
- `-i` (interactive) : 표준 입력(stdin)을 열어서 명령어를 직접 입력할 수 있게 만든다.
- `-t` (tty) : 터미널 환경을 만들어준다. (ls, cd, cat 등을 사용할 수 있다.)
- `nginx_web` : 접속할 컨테이너 이름 (컨테이너 이름 대신 ID도 가능)
- `/bin/bash` : 접속했을 때 실행할 쉘 (bash 쉘 사용)

```bash
guest@Server-B:~$ curl http://192.168.10.100
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
...
</body>
</html>
```

**정리**: `docker run`으로 컨테이너를 즉시 생성/실행하고 `docker exec -it`로 내부에 접속하는 방법을 확인했다. 이제 컨테이너 내부에서 실행 중인 프로세스를 확인하는 `docker top`을 살펴본다.

---

## docker top — 프로세스 확인

```bash
[guest@Server-A ~]$ docker top nginx_web
UID    PID    PPID   C   STIME   TTY   TIME    CMD
root   6371   6348   0   12:58   ?     00:00:00   nginx: master process nginx -g daemon off;
101    6447   6371   0   12:58   ?     00:00:00   nginx: worker process
101    6448   6371   0   12:58   ?     00:00:00   nginx: worker process
```

- **master process** = 총괄 관리자
  - nginx를 시작할 때 맨 처음 시작되는 프로세스가 master
  - 설정 파일(nginx.conf) 읽기 및 적용
  - worker 프로세스 생성, 종료 관리
  - 포트 바인딩(80, 443 등 시스템 포트 열기)
  - 실제 요청은 처리하지 않음; 설정이 바뀌면 worker에게 전달만 한다.

- **worker process** = 실제 일하는 직원
  - 브라우저 요청 처리 (HTML, CSS, JS, 이미지 등 전달)
  - 클라이언트 연결 유지 및 처리
  - 여러 worker가 생성될 수 있음(고성능 구조)
  - 각 worker는 독립적으로 실행되어 병렬 처리 가능
  - master가 설정한 내용을 기반으로 동작

- **UID** : 프로세스를 실행한 사용자 ID (root = 시스템 관리자, 101 = 일반 사용자(nginx worker 보안용))
- **PID** : 현재 컨테이너 내부에서 실행 중인 프로세스 ID
- **C** : CPU 사용률 (CPU usage %)
- **STIME** : Start Time으로 프로세스가 시작된 시간 (24시간제 또는 날짜 형식)
- **CMD** : 실행된 명령어(프로세스 이름)

**정리**: `docker top`으로 master/worker process 구조와 UID, PID, CPU 사용률 등을 확인했다. 다음으로 컨테이너 내부에 네트워크 도구가 기본으로 없다는 점과 이를 설치하는 방법을 살펴본다.

---

## 컨테이너 내부 네트워크 도구 설치

```bash
[guest@Server-A ~]$ docker  exec  -it  nginx_web  /bin/bash
root@5e4cf32df303:/#
root@5e4cf32df303:/# ip addr
bash: ip: command not found
root@5e4cf32df303:/# ifconfig
bash: ifconfig: command not found    # 네트워크 관련 패키지가 없기때문에 확인할 수 없다.


root@5e4cf32df303:/# apt-get  install  -y  iproute2
root@5e4cf32df303:/# apt-get  install  -y  net-tools


root@5e4cf32df303:/# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0@if18: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether 9a:5d:18:31:3e:e7 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.17.0.2/16 brd 172.17.255.255 scope global eth0
       valid_lft forever preferred_lft forever
```

**정리**: 컨테이너에는 기본적으로 iproute2, net-tools 같은 네트워크 도구가 없어 필요 시 직접 설치해야 한다는 점을 확인했다. 하지만 이렇게 컨테이너 내부에서 직접 설정하는 방식에는 근본적인 문제가 있는데, 다음 절에서 그 이유를 다룬다.

---

## 컨테이너 내부에서 직접 설정하면 안 되는 이유

- 실행 중인 컨테이너에 직접 접속하여 패키지를 설치하거나 설정을 변경하는 방식은 실습이나 임시 확인에는 사용할 수 있지만, 운영 방식으로는 권장되지 않는다.

### 1) 재현할 수 없다

- 컨테이너 내부에서 직접 설치한 패키지와 변경한 설정은 Dockerfile에 기록되지 않는다.

```
Dockerfile     : 원래 이미지 생성 과정만 기록된다.
실행 중 컨테이너 : 사용자가 직접 설치한 내용만 존재
```

- 따라서 다른 사람이 동일한 이미지로 컨테이너를 만들어도 같은 환경이 생성되지 않는다.
  - 내 컨테이너          : iproute2, net-tools 설치됨
  - 다른 사람이 생성한 컨테이너 : iproute2, net-tools 설치되지 않음
- 결과적으로 다음과 같은 문제가 발생: "내 환경에서는 정상인데 다른 환경에서는 동작하지 않는다."

### 2) 컨테이너 재생성 시 설정이 사라진다.

- 컨테이너 내부에 직접 설치한 패키지는 해당 컨테이너의 쓰기 레이어에 저장된다.

```
이미지  -->  컨테이너 생성
             컨테이너 쓰기 레이어
             └── iproute2 설치
             └── net-tools 설치
```

- 컨테이너를 삭제하면 쓰기 레이어도 함께 삭제된다.

```bash
docker run -d --name nginx_web nginx:latest    # 컨테이너를 실행

ip addr
bash: ip: command not found
```

### 3) 여러 컨테이너의 설정이 달라질 수 있다

- 같은 이미지로 여러 컨테이너를 생성한 후 일부 컨테이너에만 패키지를 설치하면 컨테이너마다 환경이 달라진다.
  - nginx_web1 : iproute2 설치됨
  - nginx_web2 : net-tools만 설치됨
  - nginx_web3 : 아무것도 설치되지 않음

- 모두 같은 이미지로 생성했지만 실제 내부 환경은 서로 달라진다.
- 이 상태에서는 일부 컨테이너에서만 오류가 발생할 수 있으며, 문제 원인을 찾기가 어려워진다.

```
같은 이미지
 → 서로 다른 컨테이너 설정
 → 동작 결과가 달라짐
 → 장애 원인 추적 어려움
```

**정리**: 컨테이너 내부에서 직접 설정하면 재현 불가능, 재생성 시 소실, 컨테이너 간 불일치라는 세 가지 문제가 발생한다. 따라서 다음 절에서는 Dockerfile을 통해 환경변수와 패키지를 이미지 자체에 고정하는 방법을 실습한다.

---

## Dockerfile를 사용하여 환경변수 설정 및 패키지 설치

```bash
[guest@Server-A ~]$ mkdir  nginx_con

[guest@Server-A ~]$ cd nginx_con/

[guest@Server-A nginx_con]$ vi dockerfile
FROM  nginx:latest

ENV  LANG=C.UTF-8
ENV  LC_ALL=C.UTF-8

RUN  apt-get  update  &&  apt-get  install  -y  iproute2  net-tools  iputils-ping   vim

:wq
```

환경변수 설명
- **LANG=C.UTF-8**
  - `LANG` : 시스템의 기본 언어, 문자 인코딩, 정렬 방식 등에 사용하는 환경변수
  - `C` : 특정 국가 언어를 적용하지 않는 기본 POSIX 환경
  - `UTF-8` : 한글, 영어, 일본어 등 다양한 문자를 표현할 수 있는 문자 인코딩

- **LC_ALL=C.UTF-8 적용**
  - `LC_CTYPE`   : 문자 인식과 인코딩
  - `LC_TIME`    : 날짜와 시간 표시
  - `LC_NUMERIC` : 숫자 표시 형식
  - `LC_COLLATE` : 문자열 정렬 방식
  - `LC_MESSAGES`: 프로그램 메시지 언어
  - `LC_ALL`     : 모든 LC를 의미하며 다른 LC_* 환경변수보다 우선순위가 높다.

```bash
[guest@Server-A nginx_con]$ docker  build  -t  nginx-with-package:1.1  .
[+] Building 6.7s (6/6) FINISHED                                        docker:default
 => [internal] load build definition from dockerfile                              0.0s
 => => transferring dockerfile: 186B                                              0.0s
 => [internal] load metadata for docker.io/library/nginx:latest                  0.0s
 => [internal] load .dockerignore                                                 0.0s
 => => transferring context: 2B                                                   0.0s
 => CACHED [1/2] FROM docker.io/library/nginx:latest@sha256:8541484afbc9c8a5a8a9 0.0s
 => [2/2] RUN  apt-get  update  &&  apt-get  install  -y  iproute2  net-tools  i 4.0s
 => exporting to image                                                            2.6s
 => => exporting layers                                                           2.0s
 => => naming to docker.io/library/nginx-with-package:1.1                        0.0s
 => => unpacking to docker.io/library/nginx-with-package:1.1                     0.6s
```

```bash
[guest@Server-A nginx_con]$ docker  run \
  -d  \
  --name nginx_package_web    \
  -p 8080:80  \
  nginx-with-package:1.1


[guest@Server-A nginx_con]$ docker ps -a
CONTAINER ID   IMAGE                      COMMAND                CREATED       STATUS         PORTS                      NAMES
e4b09cb18ed2   nginx-with-package:1.1   "/docker-entrypoint.…"  3 seconds ago  Up 3 seconds   0.0.0.0:8080->80/tcp   nginx_package_web
5e4cf32df303   nginx:latest              "/docker-entrypoint.…"  2 hours ago    Up 2 hours     0.0.0.0:80->80/tcp     nginx_web
```

**정리**: Dockerfile에 `ENV LANG`/`LC_ALL`과 `RUN apt-get install`을 작성해 환경변수와 패키지가 이미지에 고정된 새 이미지를 빌드하고 실행했다. 마지막으로 이 이미지 내부에 실제로 패키지와 UTF-8 설정이 반영되었는지 검증한다.

---

## 패키지 포함 이미지 내부 검증

```bash
[guest@Server-A nginx_con]$ docker exec  -it  nginx_package_web  /bin/bash
root@e4b09cb18ed2:/#


root@e4b09cb18ed2:/# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0@if25: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether 9e:38:90:a7:34:cd brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.17.0.3/16 brd 172.17.255.255 scope global eth0
       valid_lft forever preferred_lft forever


root@e4b09cb18ed2:/# ifconfig
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.17.0.3  netmask 255.255.0.0  broadcast 172.17.255.255
        ether 9e:38:90:a7:34:cd  txqueuelen 0  (Ethernet)
        RX packets 18  bytes 2244 (2.1 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 3  bytes 126 (126.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0


root@e4b09cb18ed2:/# ping  -c  3  www.google.com
PING www.google.com (142.251.157.119) 56(84) bytes of data.
64 bytes from 142.251.157.119: icmp_seq=1 ttl=127 time=31.8 ms
64 bytes from 142.251.157.119: icmp_seq=2 ttl=127 time=32.0 ms
64 bytes from 142.251.157.119: icmp_seq=3 ttl=127 time=31.2 ms

--- www.google.com ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2004ms
rtt min/avg/max/mdev = 31.220/31.675/31.963/0.325 ms


root@e4b09cb18ed2:/# vi  /usr/share/nginx/html/index.html    # 한글이 깨지지 않고 출력되는것을 확인
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <title>Nginx Web Server</title>
</head>
<body>
    <h1>Nginx 웹 서버</h1>
    <h2>Docker 컨테이너에서 실행 중입니다.</h2>
    <p>index.html 파일 수정이 정상적으로 적용되었습니다.</p>
</body>
</html>
```

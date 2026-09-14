# Docker 06 — 컨테이너간 통신 (네트워크)

## Container Network Model

- 도커는 컨테이너를 생성할 때 네트워크를 자동으로 구성하며, 컨테이너 간 통신은 가상의 네트워크 구조를 통해 이뤄진다. 외부에서 컨테이너에 접근하려면 `-p`로 포트를 바인딩해야 하고, 컨테이너끼리 이름으로 통신하려면 기본 docker0 bridge 대신 user-defined bridge 네트워크를 만들어 연결해야 한다.

```bash
[guest@Server-A web]$ ip addr
1: lo: ...
2: ens160: ...
    inet 192.168.10.100/24 brd 192.168.10.255 scope global noprefixroute ens160
    ...
3: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default
    link/ether ce:c0:8f:0f:28:99 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
    ...
36: br-f9f4654680bf: ...
    inet 172.18.0.1/16 brd 172.18.255.255 scope global br-f9f4654680bf
    ...
43: veth258c9b6@if2: ...
46: veth0bd3034@if2: ...


[guest@Server-A web]$ ip addr show docker0
3: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default
    link/ether ce:c0:8f:0f:28:99 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
```

### docker0 브릿지

- `docker0`는 도커가 설치되면 자동으로 생성되는 가상 브릿지(virtual ethernet bridge)이다.
- `docker0`의 네트워크 대역은 기본적으로 `172.17.0.0/16`이다.
- L2 기반 통신하며 MAC 주소 기반으로 프레임을 전달 (스위치처럼 동작)
- 컨테이너 eth0에 MAC 주소가 존재하며 브릿지가 이를 학습한다.
- 모든 컨테이너는 docker0 브릿지에 연결되는 방식으로 IP를 부여받는다.

### veth pair (Sandbox)

- 컨테이너를 실행하면 `veth` 인터페이스 쌍(veth pair)이 자동으로 생성됨
- veth pair는 가상 LAN 케이블 두 가닥이라고 보면 된다.
- 두 개가 한 쌍으로 만들어지며, 한쪽으로 들어간 패킷은 반드시 다른 쪽으로 나온다.
- 두 개의 가상 네트워크 인터페이스가 튜브로 연결된 구조이다.

왜 veth pair를 사용할까
- 컨테이너 내부 네트워크 공간은 호스트 OS와 별도로 격리되어 있다.
- 그래서 컨테이너 내부의 eth0와 호스트의 네트워크를 물리적으로 연결할 케이블이 필요하다.

도커의 네트워크 구조에서 veth pair의 역할
1. veth pair 두 개 생성 : veth1234, vethABCD
2. 한쪽은 컨테이너 내부 eth0에 연결 : 컨테이너 안에서는 eth0 인터페이스로 보이며 실제로는 veth의 한쪽 끝이다.
3. 반대쪽은 호스트의 docker0 브릿지에 연결 : 호스트에서는 veth1234 같은 이름으로 보이고 docker0 스위치에 접속된 포트처럼 동작한다.

구조
```
컨테이너 eth0 ── veth ─ethaaa───┐
컨테이너2 eth0 ── veth ─ethbbb───┤
컨테이너3 eth0 ── veth ─ethccc───┼── docker0 ── 호스트 네트워크 ── eth0/ens160
컨테이너4 eth0 ── veth ─ethddd───┘
컨테이너5 eth0 ── veth ─etheee───┘
```

외부 통신 방식
- 모든 컨테이너는 기본적으로 docker0 브릿지를 통해 외부와 통신한다.
- 브릿지가 L2 스위치처럼 동작하기 때문에 컨테이너끼리도 통신이 가능하다.
- 외부(인터넷, 호스트 등)도 docker0을 통해 연결된다.

컨테이너의 기본 IP 주소
- 컨테이너가 실행되면 다음과 같은 형식의 IP를 자동으로 받는다: `172.17.X.Y`
- EX) container1 : 172.17.0.2
- EX) container2 : 172.17.0.3
- 컨테이너끼리는 `ping 172.17.0.3` 처럼 IP로도 통신 가능하다.

**정리**: docker0 브릿지와 veth pair를 통해 컨테이너가 가상 네트워크에 연결되는 구조를 살펴봤다. 다음으로는 이 내부 네트워크를 외부와 연결하는 포트 포워딩의 개념을 다룬다.

---

## 포트 포워딩(port-forwarding)의 개념

- 컨테이너 내부의 포트를 호스트 포트와 연결(bind)시켜 외부(브라우저, 다른 PC, 스마트폰 등)에서 컨테이너 서비스에 접속할 수 있도록 하는 기능이다.
- 도커는 이 포트 포워딩을 위해 내부적으로 iptables(NAT 규칙)를 자동으로 생성하여 호스트로 들어온 요청을 컨테이너로 전달한다.
- 즉, 외부 요청 → 호스트 포트 → 도커가 만든 NAT → 컨테이너 내부 포트 흐름으로 동작한다.

### 포트 바인딩 옵션 정리

```bash
형식: docker run -p 호스트포트:컨테이너포트 이미지명
EX)  docker run -d -p 8080:80 nginx
# 외부에서 http://localhost:8080 으로 접속 가능
# 컨테이너 내부 Nginx(80번 포트)가 보인다.
```

| 옵션 | 설명 |
|------|------|
| `-p hostPort:containerPort` | 호스트의 특정 포트와 컨테이너 포트를 직접 연결 (호스트 포트를 직접 지정). 예: `-p 8080:80` |
| `-p containerPort` | 호스트의 임의 포트를 자동 배정 (도커가 랜덤으로 지정, 예: 32768 → 80) |
| `-P` | Dockerfile의 EXPOSE에 선언된 모든 포트를 자동 매핑 |

### 포트를 바인딩하지 않으면?

- 컨테이너 내부에서는 서비스가 정상적으로 동작하지만 호스트 외부에서는 절대 접근할 수 없다.
- 이유: `docker0`는 컨테이너 내부 네트워크(172.17.x.x)이며 외부 PC에서는 이 네트워크로 직접 접근할 수 없다.
- 외부 접속을 위해서는 NAT 규칙을 통한 포트 바인딩이 필수이다.
- 즉, `docker run nginx` 이렇게 실행하면 외부에서 접근할 방법이 없다.
- 반면 `docker run -p 8080:80 nginx` 이렇게 해야 외부에서 `http://localhost:8080`으로 접근 가능하다.

### 전체 동작 흐름

```
1) 컨테이너 Nginx가 내부 포트 80에서 대기
2) docker run -p 8080:80 으로 실행
3) 도커가 iptables NAT 규칙 자동 생성 (DNAT)
4) 외부 요청이 호스트 8080으로 들어옴
5) 도커가 NAT 변환하여 컨테이너 80 포트로 전달
6) 컨테이너가 응답 --> 도커 NAT --> 외부로 반환
```

즉, 포트 바인딩은 컨테이너 서비스를 외부로 공개하는 공식 절차이다.

**정리**: 포트 포워딩은 iptables NAT 규칙을 통해 호스트 포트와 컨테이너 포트를 연결하는 절차이며, 바인딩 없이는 외부 접근이 불가능하다는 점을 확인했다. 이어서 실제로 컨테이너를 생성해 네트워크 상태를 직접 확인해본다.

---

## 컨테이너 생성 및 네트워크 확인

### web_test1 컨테이너 생성

```bash
[guest@Server-A web]$ docker  run  -d  --name  web_test1  nginx:latest
c84bd02c4a93cf4aaa981a2486a45ff266d3e534edcc79a9a06911877a2dcc3f


[guest@Server-A web]$ docker  exec  -it  web_test1  /bin/bash

root@c84bd02c4a93:/# ip addr
bash: ip: command not found

root@c84bd02c4a93:/# apt-get update
root@c84bd02c4a93:/# apt-get install  -y  iproute2

root@c84bd02c4a93:/# ip addr
1: lo: ...
2: eth0@if47: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether fe:e7:c6:66:fb:84 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.17.0.2/16 brd 172.17.255.255 scope global eth0
```

### web_test2 컨테이너 생성

```bash
[guest@Server-A web]$ docker  run  -d  --name  web_test2  nginx:latest
9cde2030211ea7ee4e005e22e1f7a7fe46a1209ad1c7aac9ddab4cc9cedfcb7b


[guest@Server-A web]$ docker  exec  -it  web_test2  /bin/bash

root@9cde2030211e:/# apt-get update
root@9cde2030211e:/# apt-get install  -y  iproute2

root@9cde2030211e:/# ip addr
2: eth0@if50: ...
    inet 172.17.0.3/16 brd 172.17.255.255 scope global eth0
```

### 컨테이너 간 ICMP 통신

```bash
root@9cde2030211e:/# apt-get  install  -y  iputils-ping

root@9cde2030211e:/# ping  -c  3  172.17.0.1    # web_test2 컨테이너 ---> Gateway
PING 172.17.0.1 (172.17.0.1) 56(84) bytes of data.
64 bytes from 172.17.0.1: icmp_seq=1 ttl=64 time=1.72 ms
64 bytes from 172.17.0.1: icmp_seq=2 ttl=64 time=0.131 ms
64 bytes from 172.17.0.1: icmp_seq=3 ttl=64 time=0.261 ms

--- 172.17.0.1 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2040ms


root@9cde2030211e:/# ping  -c  3  172.17.0.2    # web_test2 컨테이너 ---> web_test1 컨테이너
PING 172.17.0.2 (172.17.0.2) 56(84) bytes of data.
64 bytes from 172.17.0.2: icmp_seq=1 ttl=64 time=2.08 ms
64 bytes from 172.17.0.2: icmp_seq=2 ttl=64 time=0.094 ms
64 bytes from 172.17.0.2: icmp_seq=3 ttl=64 time=0.053 ms

--- 172.17.0.2 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2010ms


[guest@Server-A web]$ docker rm -f  web_test1
[guest@Server-A web]$ docker rm -f  web_test2
```

### brctl show — 브릿지 확인

```bash
[guest@Server-A web]$ brctl  show
bash: brctl: 명령을 찾을 수 없습니다...

[guest@Server-A web]$ dnf  install  -y  bridge-utils


[guest@Server-A web]$ brctl  show
bridge name         bridge id              STP enabled   interfaces
br-f9f4654680bf     8000.163887ebfce9      no            veth0bd3034
docker0             8000.cec08f0f2899      no            veth258c9b6
```

- **docker0** : 도커 설치 시 자동 생성되는 기본 브릿지. 컨테이너를 생성하면 기본적으로 여기에 연결된다. 기본 IP: 172.17.0.1/16
- **br-f9f4654680bf** : `docker network create` 명령으로 만든 user-defined bridge 네트워크의 실제 장치 이름 (mynet 같은 이름의 네트워크가 내부적으로 `br-랜덤ID` 형태로 구현)
- **bridge id** : 브릿지를 식별하는 고유 ID. MAC address 기반으로 만들어진 값
- **STP enabled** : STP(Spanning Tree Protocol) 켜짐 여부. 도커 네트워크에서는 루프 구조가 발생하지 않기 때문에 `no`
- **interfaces** : 현재 브릿지에 연결된 veth 인터페이스(컨테이너 포트) 목록

**정리**: 두 컨테이너를 생성해 각자의 IP를 확인하고 ICMP로 컨테이너 간 통신을, `brctl show`로 브릿지 구조를 직접 확인했다. 이어서 이러한 컨테이너의 포트를 외부로 노출하는 다양한 방법(`-p`, `-P`)을 실습한다.

---

## 컨테이너 포트 외부로 노출하기

### -p (hostPort:containerPort)

```bash
[guest@Server-A web]$ docker  run  -d  --name  web_port1 -p 80:80  nginx:latest
4b36e08724e0cde5fa2814fbd2553843a89f2a7b983241d93d4750ba4623c16c


[guest@Server-A web]$ docker ps
CONTAINER ID   IMAGE          COMMAND                   CREATED         STATUS         PORTS                    NAMES
4b36e08724e0   nginx:latest   "/docker-entrypoint.…"   8 seconds ago   Up 8 seconds   0.0.0.0:80->80/tcp       web_port1

# 웹 브라우저로 접속
http://192.168.10.100/

# curl로 테스트
guest@Server-B:~$ curl  http://192.168.10.100
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
...
</html>
```

### -p (containerPort) — 랜덤 포트 자동 배정

```bash
[guest@Server-A web]$ docker  run  -d  --name  web_port2 -p 80  nginx:latest
2420dcec82e800f7c2137b17dfc369f7d8e2c4d3cc8dab177eaf8c4adfbcfba9


[guest@Server-A web]$ docker ps
CONTAINER ID   IMAGE          COMMAND                   CREATED        STATUS        PORTS                                      NAMES
2420dcec82e8   nginx:latest   "/docker-entrypoint.…"   9 seconds ago  Up 8 seconds  0.0.0.0:32768->80/tcp, [::]:32768->80/tcp  web_port2
4b36e08724e0   nginx:latest   "/docker-entrypoint.…"   2 minutes ago  Up 2 minutes  0.0.0.0:80->80/tcp, [::]:80->80/tcp        web_port1

# 웹 브라우저로 접속
http://192.168.10.100:32768

# curl로 테스트
guest@Server-B:~$ curl  http://192.168.10.100:32768
```

### -P 옵션 (Dockerfile의 EXPOSE 사용)

```bash
[guest@Server-A ~]$ mkdir  webport
[guest@Server-A ~]$ cd  webport/

[guest@Server-A webport]$ vi dockerfile
FROM  nginx:latest

COPY  index.html   /usr/share/nginx/html/index.html

EXPOSE 80

: wq


[guest@Server-A webport]$ vi index.html
<h1> Docker Port number EXPOSE Test </h1>

:wq


[guest@Server-A webport]$ docker  build  -t  webp:1.0  .


[guest@Server-A webport]$ docker  images  webp
IMAGE     ID             DISK USAGE   CONTENT SIZE
webp:1.0  371d663bf0bf   235MB        63.1MB


[guest@Server-A webport]$ docker  run  -d  --name web_port3  -P  webp:1.0
d23a09c17c498cff31a2ae043f207bb2046f0a6ad258a6321f57f9cbeee0bc4c


[guest@Server-A webport]$ docker  ps
CONTAINER ID   IMAGE        COMMAND                   CREATED         STATUS        PORTS                                      NAMES
d23a09c17c49   webp:1.0     "/docker-entrypoint.…"   8 seconds ago   Up 7 seconds  0.0.0.0:32769->80/tcp, [::]:32769->80/tcp  web_port3
2420dcec82e8   nginx:latest "/docker-entrypoint.…"   7 minutes ago   Up 7 minutes  0.0.0.0:32768->80/tcp, [::]:32768->80/tcp  web_port2
4b36e08724e0   nginx:latest "/docker-entrypoint.…"   10 minutes ago  Up 10 minutes 0.0.0.0:80->80/tcp, [::]:80->80/tcp        web_port1

# 웹 브라우저로 접속
http://192.168.10.100:32769

# curl로 테스트
guest@Server-B:~$ curl  http://192.168.10.100:32769
```

**정리**: `-p hostPort:containerPort`, `-p containerPort`(랜덤 배정), `-P`(EXPOSE 기반 자동 매핑) 세 가지 포트 노출 방식을 실습했다. 마지막으로 컨테이너 이름 기반 통신을 지원하는 user-defined bridge network를 살펴본다.

---

## user-defined bridge network

- **user-defined bridge network**는 도커가 기본 제공하는 docker0 말고 사용자가 직접 만드는 별도의 가상 스위치이다.

장점
- **컨테이너끼리 서비스명으로 통신(DNS 지원)** : 기본 bridge는 `app1 → app2` 같은 이름 접근이 안 된다. user-defined bridge는 자동 DNS 제공한다.
- **IP 고정할 필요가 없음** : Docker가 내부 DNS로 서비스명을 자동 매칭해주기 때문에 컨테이너가 재기동되어 IP가 바뀌어도 문제 없다.
- **복잡한 구조(Reverse Proxy, 멀티 웹서버, DB 연결 등)에 필수** : nginx reverse proxy → app1, app2 라우팅, 웹 서버 여러 대 → DB 1대 연결

### 네트워크 생성

```bash
# 기본형
docker network create webnet

# 고급형
docker network create --driver bridge  --subnet 192.168.100.0/24  --gateway 192.168.100.254  mynet
# --driver bridge : 브릿지 네트워크 생성
# --subnet        : 네트워크 주소 범위
# --gateway       : 게이트웨이 IP 지정
# mynet           : 네트워크 이름

# 확인
docker network ls
docker network inspect mynet
```

### 생성한 네트워크에 컨테이너 포함하여 실행

```bash
# 기본 형태
docker run  -d  --name web1  --network webnet nginx

# 웹 서버 nginx가 webnet 네트워크에 연결되며 webnet 내부 DNS 서버가 web1 --> containerIP 로 매핑한다.

# 고급 형태(예시)
docker run -d --name appjs  --net mynet  --ip 192.168.100.100  -p 8080:8080  node-app
# --net mynet            : mynet 스위치에 연결
# --ip 192.168.100.100   : 컨테이너 IP를 고정 부여
# -p 8080:8080           : 외부 공개 포트

curl localhost:8080    # 정상 동작 확인
```

### 실행 중인 컨테이너에 추가 네트워크 연결

- 한 컨테이너가 여러 네트워크에 연결될 수 있다 (스위치 여러 개에 PC를 꽂는 것과 동일)

```bash
docker network connect webnet web1

# 이제 web1은 두 개의 스위치에 동시 연결된 상태가 된다.

# 확인
docker inspect web1 | grep -i network

# 결과
"Networks":
"bridge": { ... }
"webnet": { ... }

# 즉 web1은 docker0 + webnet 두 네트워크 모두에 연결된다.
```

### 네트워크 삭제

```bash
docker network rm webnet

# 컨테이너가 연결되어 있으면 삭제 불가
# 반드시 네트워크에 연결된 컨테이너를 먼저 정리해야 한다.
# 에러 메시지 예: Error: resource is in use

# 해결
docker network disconnect webnet web1
docker network rm webnet
```

### 네트워크 구조 비교

| 구조 | 설명 |
|------|------|
| **docker0** (기본 브릿지) | 172.17.0.0/16 자동할당. 기본 컨테이너는 모두 여기에 붙음. 컨테이너 이름 DNS 미지원. |
| **사용자 정의 네트워크** (mynet, webnet) | 사용자가 만든 가상 스위치. 서브넷/게이트웨이 직접 설정. 컨테이너 이름 통신 가능. 서비스 분리 가능. |

```
구조: container eth0  <-->  veth pair  <-->  webnet (가상 스위치)  <-->  host(eth0)  <-->  인터넷, 외부 PC
```

### 네트워크를 추가해서 사용하는 이유

- **서비스 분리** : web 서버는 webnet, db 서버는 dbnet. 같이 구성하고, 필요할 경우에만 서로 연결한다.
- **보안 및 트래픽 분리** : 특정 네트워크끼리만 통신하도록 구성 가능하다.

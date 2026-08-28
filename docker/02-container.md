# 도커 컨테이너

# 컨테이너란(Container)?

-**컨테이너**는 애플리케이션과 그 실행 환경을 하나로 묶어 배포할 수 있는 단위다. 이미지 하나로 여러 서버에 동일한 실행 환경을 재현하거나, 서비스별로 컨테이너를 나누어 독립적으로 개발·배포·확장할 수 있어 MSA 구조에서 특히 널리 활용된다.

-컨테이너는 애플리케이션이 실행되는 독립된 공간이다.
 하나의 운영체제 위에서 여러 개의 컨테이너를 동시에 실행할 수 있고, 각 컨테이너는 서로 완전히 분리된 환경에서 동작한다.
 즉, 컨테이너는 가상머신처럼 보이지만 실제로는 훨씬 가볍다.
 OS 전체를 복제하지 않고, 호스트의 커널을 공유하면서도 격리된 공간을 만든다.

-예전에는 애플리케이션을 실행하려면 OS마다 다른 설정과 라이브러리를 일일이 맞춰야 했지만,
 컨테이너는 이 모든 환경을 패키징해서 어디서나 동일하게 실행되도록 해준다.

| 비교 항목 | 기존 서버 환경 | 컨테이너 환경 |
|---|---|---|
| 환경 구성 | 직접 설치 (복잡) | 이미지 한 번으로 즉시 실행 |
| 실행 속도 | OS 부팅 필요 | 몇 초 내 실행 |
| 자원 사용 | OS별 중복 점유 | 커널 공유로 가벼움 |
| 배포 | 수동 설정 필요 | 이미지 기반 자동 배포 |

## 컨테이너와 이미지의 차이

| 구분 | 이미지(Image) | 컨테이너(Container) |
|---|---|---|
| 정의 | 실행 가능한 템플릿 | 이미지를 실행한 결과물 |
| 상태 | 정적(읽기 전용) | 동적(읽기, 쓰기 가능) |
| 역할 | 애플리케이션 설치본 | 실제 실행 환경 |
| 예시 | Ubuntu 22.04 이미지 | Ubuntu 22.04를 실행 중인 컨테이너 |

즉, 이미지(Image)는 설계도, 컨테이너(Container)는 그 설계도로 만들어진 건물이라고 보면 된다.

## 이미지(Image)

-**이미지**는 건물의 설계도. 읽기 전용이다. 설계도를 바꾸지 않는 한 언제 어디서 그 설계대로 똑같이 찍어낼 수 있다.

**무엇을 담나**
- 실행 파일과 라이브러리
- 설정파일시스템 레이어들(읽기 전용 레이어들의 묶음)
- 특징값, 기본 명령(CMD/ENTRYPOINT)

**특징**
- 불변성 : 한 번 빌드된 이미지는 바뀌지 않는다.
- 이식성 : 어디서 실행해도 동일한 동작을 보장한다.
- 계층 구조 : 공통 레이어를 여러 컨테이너가 공유하므로 디스크를 절약한다.

## 컨테이너(Container)

-설계도로부터 실제로 지은 건물. 살고, 전기 켜고, 물도 쓰는 바로 그 집이 컨테이너다.

**무엇인가**
- 이미지를 기반으로 생성된 실행 인스턴스
- 이미지의 읽기 전용 레이어 위에 쓰기 가능한 얇은 레이어가 한 장 더 얹힌다.

-컨테이너는 여러 개의 파일시스템 레이어(layer)가 합쳐져서 하나의 완전한 OS처럼 보이게 된다.
이것을 Union File System(유니언 파일 시스템)이라고 한다.

**파일시스템 개념도**
```
[Writable Layer] 	<---- 컨테이너에서 바뀌는 내용이 이 레이어에만 기록
[Image Layers] 	<---- 읽기 전용, 여러 컨테이너가 함께 사용
[Host Kernel]
```

### Host Kernel
- 호스트 운영체제의 커널(Linux Kernel)
- 컨테이너는 자체 커널을 가지고 있지 않고, 호스트의 커널을 공유하기 때문에 컨테이너는 가볍고 빠르다.
  (가상머신은 커널까지 따로 있어서 무겁다.)

### Image Layers
- 컨테이너의 기본 뼈대
- 읽기 전용(read-only) 레이어들로, 여러 컨테이너가 동시에 공유할 수 있다.
- 이미지 하나가 여러 개의 레이어로 구성되어 있고, 각 레이어는 Dockerfile의 한 줄(FROM, RUN, COPY 등)마다 쌓이는 구조

### Writable Layer
- 컨테이너가 실행될 때 새로 만들어지는 "쓰기 가능" 공간
- 컨테이너 안에서 무언가 바꾸거나(예: 파일 생성, 삭제, 수정) 새로운 로그가 생기면 모두 여기에 저장된다.
- 원본 이미지는 그대로 두고 변경사항은 이 위에 별도로 기록되는 구조

### 분리해서 사용하는 이유

| 이유 | 설명 |
|---|---|
| 효율성 | 이미지는 여러 컨테이너가 공유 (디스크 절약) |
| 성능 | 새로운 컨테이너 실행 시 이미지 복사하지 않고 바로 실행 (원본을 사용하기 때문에 복사하지 않는다.) |
| 안전성 | 원본 이미지가 변경되지 않는다. (버전 관리 용이) |
| 일회성 | 컨테이너 삭제하면 쓰기 레이어도 함께 삭제된다. |

---

## 컨테이너의 동작 방식

-컨테이너는 리눅스 커널의 기능을 이용해서 격리된 환경을 만든다.

**1) 네임스페이스 (Namespaces)**
- 프로세스가 서로의 PID, 네트워크, 파일 시스템을 볼 수 없게 분리한다.
- 마치 각각의 컨테이너가 독립된 시스템처럼 보이게 해준다.

**2) Cgroups (Control Groups)**
- CPU, 메모리, 네트워크 등 시스템 자원 사용량을 제한한다.
- 하나의 컨테이너가 전체 자원을 독점하지 못하게 제어한다.

**3) UnionFS (Union File System)**
- 여러 계층의 파일 시스템을 합쳐서 하나의 이미지처럼 보이게 하는 기술
- 컨테이너는 이미지 레이어(읽기 전용) 위에 쓰기 가능한 레이어를 하나 추가해서 동작한다.

구조를 단순히 도식으로 표현하면:
```
[Container Layer] (Writable)
----------------------------
[Image Layer 3]
[Image Layer 2]
[Image Layer 1]
----------------------------
[Host OS + Kernel]
```

### 이미지(Image) 기준

```
run node app.js	: Run Layer (실행 명령)
app.js		: Application Source Layer (애플리케이션 코드)
nodejs		: Base Image Layer (기초 환경)
```

### 컨테이너(Container) 기준

```
[Writable Layer]    	: 실행 중 바뀌는 파일(로그, 캐시, 데이터 등)
run node app.js     	: 실행 명령
app.js              	: 애플리케이션 코드
nodejs             	: 실행 환경
[Host Kernel]       	: 리눅스 커널 (공유)
```

**Base Image Layer**
- 컨테이너의 기반이 되는 환경
- 도커에서 `FROM node:latest` 같은 구문으로 불러오는 부분
- Node.js 런타임과 필수 OS 구성 요소(리눅스 배포판 등)를 포함
- 컨테이너가 실행될 최소한의 "운영 환경"을 제공

**Application Source Layer (애플리케이션 코드)**
- 프로그램 코드가 들어있는 레이어
- 서비스의 실제 코드, 설정 파일, 정적 리소스(HTML/CSS/이미지 등)를 포함
- COPY 명령어를 통해 이미지에 복사된다.

**run node app.js → Run Layer (실행 명령)**
- 도커 컨테이너를 실행할 때 실제 수행되는 명령
- 즉, 이 이미지가 실행될 때 "무엇을 할 것인지"를 정하는 시작점(엔트리포인트) 역할
- 컨테이너가 시작될 때 실행할 명령어 지정

### 이미지(Image) 동작

```
run node app.js		: UUID로 디스크에 저장 (8f2b3c4e-91d7-4dca-82c5-f0a9a3e67d51, 실행 명령 레이어)
app.js			: UUID로 디스크에 저장 (3a7d9c2f-5b18-4a0e-9e1a-48c3c76b2a64, 애플리케이션 코드 레이어)
nodejs			: UUID로 디스크에 저장 (a4e19b68-9f1e-4b3c-b8d4-1329cf96c278, Node.js 실행 환경)
```

      UUID를 하나의 이미지로 저장한다.
```
--------------------------------------------
|  8f2b3c4e-91d7-4dca-82c5-f0a9a3e67d51	|
|  3a7d9c2f-5b18-4a0e-9e1a-48c3c76b2a64	|
|  a4e19b68-9f1e-4b3c-b8d4-1329cf96c278	|
--------------------------------------------
```

-도커를 실행하게되면 이미지를 메모리로 로딩하여 하나의 application process로 만들며 running 상태가 된다.
 이렇게 메모리에서 동작하는 상태를 컨테이너(Container)라고 한다.

---

## Docker Hub 실습

https://hub.docker.com/

```
[guest@Server-A ~]$ docker  search  httpd

[guest@Server-A ~]$ docker  search  nginx

[guest@Server-A ~]$ docker  search  mysql
```

```
[guest@Server-A ~]$ docker  pull  nginx
Using default tag: latest
latest: Pulling from library/nginx
d26f27cc8c41: Pull complete
3c7ab7949321: Pull complete
062e450697fa: Pull complete
82454cdbf456: Pull complete
b6698f04e005: Pull complete
cacfcdd01f30: Pull complete
2bedaf25031a: Pull complete
6c496f5b5050: Download complete
ea1d76ccc2c6: Download complete
Digest: sha256:5a88c9c45479443d7be2eadc894b4ed0a9801bae03d97a5760ae13b5c2005942
Status: Downloaded newer image for nginx:latest
docker.io/library/nginx:latest
```

```
[guest@Server-A ~]$ docker images
IMAGE                	ID            	DISK USAGE   	CONTENT SIZE   EXTRA
hello-world:latest	7f4da0fc94bc       	21.8kB         	9.49kB
nginx:latest         	5a88c9c45479        	238MB           	66MB
```

```
[guest@Server-A ~]$ docker inspect nginx
[
    {
        "Id": "sha256:5a88c9c45479443d7be2eadc894b4ed0a9801bae03d97a5760ae13b5c2005942",
        "RepoTags": [
            "nginx:latest"
        ],
        "RepoDigests": [
            "nginx@sha256:5a88c9c45479443d7be2eadc894b4ed0a9801bae03d97a5760ae13b5c2005942"
        ],
	~~~~~~~~~~~~~ 중간 생략 ~~~~~~~~~~~~~
        "Architecture": "amd64",
        "Os": "linux",
        "Size": 63132183,
        "RootFS": {
            "Type": "layers",
            "Layers": [
                "sha256:f2ec4de84f559f5c7be4233b589cdbdbb5507807e05621b77320edd55a1f2a0f",		# Layer 로 구성된다.
                "sha256:228e620990b5c3d9e3f214a69e229b4f448ff84a506224a00941910c9983ff65",		# Layer 로 구성된다.
                "sha256:1c53941aba0260ff1127e6b3bfecdccca19be41697b53907582d977c01e574f0",	# Layer 로 구성된다.
                "sha256:32b1ccafaf99cc8e384384e81bfe821ee85641a9547d49a0028a2e68dc26f615",	# Layer 로 구성된다.
                "sha256:d04b91e45bfae9b6f8c681f3a8df71b7dc0f189e273fd6e572e8654dd9e0dce6",		# Layer 로 구성된다.
                "sha256:ee27e7233604155bf0fe221d34807f9143b0daadfce8e7367eb7864b6cfe64b5",		# Layer 로 구성된다.
                "sha256:fb28b3cca92af0364e8a369d9715ebbf0e8922c743be5e71c09c4d3a38bc8992"		# Layer 로 구성된다.
            ]
        },

	~~~~~~~~~~~~~ 중간 생략 ~~~~~~~~~~~~~
```

```
[guest@Server-A ~]$ docker  run  -d  --name web  -p 80:80  nginx
ac41f644d411f67e62363ea0bd5d26257c6185ff1f0996ae3fb031f310a5d731
```

```
[guest@Server-A ~]$ docker images
IMAGE                	ID            	DISK USAGE   	CONTENT SIZE   EXTRA
hello-world:latest	7f4da0fc94bc       	21.8kB         	9.49kB
nginx:latest         	5a88c9c45479        	238MB           	66MB
```

**옵션 : 의미**
- `-d` : 백그라운드(daemon)로 실행 (터미널 안 막힘)
- `--name web` : 컨테이너 이름을 mynginx로 지정
- `-p 80:80` : 호스트의 80포트를 컨테이너의 80포트와 연결 (`-p <호스트포트>:<컨테이너포트>`)
- `nginx` : 실행할 이미지 이름

```
[guest@Server-A ~]$ docker ps
CONTAINER ID	IMAGE     COMMAND                   CREATED              STATUS                PORTS                                        NAMES
ac41f644d411   	nginx     "/docker-entrypoint.…"     About a minute ago   Up About a minute   0.0.0.0:80->80/tcp, [::]:80->80/tcp   web
```

```
[guest@Server-A ~]$ sudo ls  -l  /var/lib/docker/containers
합계 4
drwx--x--- 4 root root 4096  8월  5 11:02 ac41f644d411f67e62363ea0bd5d26257c6185ff1f0996ae3fb031f310a5d731
```

### 웹브라우저를 사용하여 컨테이너로 만든 웹서버 접속

http://192.168.10.100

```
Welcome to nginx!
If you see this page, nginx is successfully installed and working. 
Further configuration is required for the web server, reverse proxy, 
API gateway, load balancer, content cache, or other features.

For online documentation and support please refer to nginx.org.
To engage with the community please visit community.nginx.org.
For enterprise grade support, professional services, 
additional security features and capabilities please refer to f5.com/nginx.

Thank you for using nginx.
```

### GUI환경이 없는 LINUX에서 CURL을 사용하여 확인

```
[guest@Server-A ~]$ curl  http://192.168.10.100
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
```

### 이미지 및 컨테이너 삭제

```
[guest@Server-A ~]$ docker images
IMAGE                	ID            	DISK USAGE   	CONTENT SIZE   EXTRA
hello-world:latest	7f4da0fc94bc       	21.8kB         	9.49kB
nginx:latest         	5a88c9c45479        	238MB           	66MB
```

```
[guest@Server-A ~]$ docker ps
CONTAINER ID	IMAGE     COMMAND                   CREATED              STATUS                PORTS                                        NAMES
ac41f644d411   	nginx     "/docker-entrypoint.…"     About a minute ago   Up About a minute   0.0.0.0:80->80/tcp, [::]:80->80/tcp   web
```

```
[guest@Server-A ~]$ docker  rm  web		# 컨테이너 삭제 (컨테이너가 동작중이므로 삭제할 수 없다.)
Error response from daemon: cannot remove container "web": container is running: stop the container before removing or force remove
```

```
[guest@Server-A ~]$ docker  stop  web		# 동작중인 컨테이너 중지
web
```

```
[guest@Server-A ~]$ docker  ps		# 동작중인 컨테이너만 확인
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```

```
[guest@Server-A ~]$ docker  ps  -a		# 모든 컨테이너 확인
CONTAINER ID	IMAGE     COMMAND                   CREATED          STATUS                      	PORTS     NAMES
ac41f644d411   	nginx     "/docker-entrypoint.…"     14 minutes ago    Exited (0) 36 seconds ago                 web
```

```
[guest@Server-A ~]$ docker  rm  web		# 컨테이너 삭제
web
```

```
[guest@Server-A ~]$ docker  rm  image  nginx

		OR

[guest@Server-A ~]$ docker  rmi  nginx
Untagged: nginx:latest
Deleted: sha256:5a88c9c45479443d7be2eadc894b4ed0a9801bae03d97a5760ae13b5c2005942
```

```
[guest@Server-A ~]$ docker images
IMAGE                	ID            	DISK USAGE   	CONTENT SIZE   EXTRA
hello-world:latest	7f4da0fc94bc       	21.8kB         	9.49kB
```

**정리**: 이미지/컨테이너의 개념, 레이어 구조, Docker Hub를 통한 pull/run/rm 실습까지 컨테이너의 기본기를 다뤘다. 이어서 여러 컨테이너로 하나의 서비스를 구성하는 MSA 관점의 "컨테이너 만들기"를 살펴본다.

---

# 컨테이너 만들기

## MSA (Micro Services Architecture)

-하나의 큰 프로그램을 여러 개의 작은 프로그램(서비스)으로 나누어 개발하는 방식
-각 서비스는 독립적인 팀이 따로 만들고 따로 배포할 수 있는 하나의 작은 애플리케이션

### 쇼핑몰 시스템 (기존의 Monolithic 방식)
- 예전에는 모든 기능(로그인, 장바구니, 결제, 배송 등)이 한 프로그램 안에 다 들어 있었다.
- 로그인 → 장바구니 → 결제 → 배송 → 상품관리
- 결제 기능을 하나 수정하려면 전체를 다시 빌드하고 배포해야 하고, 결제 쪽 오류가 나면 전체 서비스가 중단될 수도 있다.

### 쇼핑몰 시스템 (MSA 구조)
- Auth Service (회원 로그인, 토큰 관리) - DB: Redis
- Cart Service (장바구니 관리) - DB: Redis (Key-Value)
- Order Service (주문 생성·조회) - DB: MongoDB (Document)
- Payment Service (결제 처리, PG 연동) - 외부 결제 서버(PSP) + MySQL
- Shipping Service (배송 관리) - DB: MySQL

### 특징

| 항목 | 설명 |
|---|---|
| 독립성 | 각 서비스는 개별 배포·개발·테스트 가능 |
| 확장성 | 트래픽 많은 서비스만 따로 scale-out 가능 |
| 유연한 언어 선택 | 서비스별로 다른 언어·프레임워크 사용 가능 |
| 장애 격리 | 특정 서비스 장애가 전체에 영향 최소화 |

## Polyglot

-Polyglot : 여러 언어를 할 줄 아는 사람
-개발에서는 여러 언어와 여러 데이터 기술을 함께 사용하는 시스템을 의미
 즉, 모든 걸 하나로 통일하지 말고, 각자에게 맞는 언어를 사용하는 개념

| 서비스 | 언어 | 데이터베이스 |
|---|---|---|
| cart-service | Java(Spring Boot) | Redis (Key-Value) |
| order-service | PHP(Laravel) | MongoDB (Document) |
| inventory-service | Node.js(Express) | MySQL (RDBMS) |
| social-service | Python(FastAPI) | Neo4j (Graph DB) |

### MSA와 폴리글랏의 관계
- MSA가 폴리글랏을 가능하게 한다.
- 단일(monolithic) 구조에서는 언어, DB를 하나로 통일해야 하지만, MSA에서는 서비스별로 기술을 독립적으로 선택할 수 있다.

### 전자상거래 플랫폼 예시 구조

```
                      			      전자상거래 플랫폼 (e-commerce platform)
                               		/           		|                     		/
		         	/                  		|                              		/
		/           	            			|					/
	장바구니(Shopping Cart)		완료된 주문(Completed Orders)	    	재고 및 가격(Inventory & Item Price) 
                	|                                		|                            			| 
	키-값 저장소(Key-Value Store)		문서 저장소(Document Store)         	관계형 데이터베이스(RDBMS, 기존 시스템)
```

| 번호 | 구성요소 | 역할 | 컨테이너 필요 여부 |
|---|---|---|---|
| 1 | e-commerce platform | 전체 서비스의 중심, 사용자 요청 관리 | 컨테이너 1개 |
| 2 | Shopping cart | 장바구니 서비스 | 컨테이너 1개 |
| 3 | Completed Orders | 주문 완료 처리 서비스 | 컨테이너 1개 |
| 4 | Inventory & Item Price | 재고 및 가격 관리 서비스 | 컨테이너 1개 |
| 5 | Key-Value Store | 데이터 저장소 (예: Redis) | 컨테이너 1개 |
| 6 | Document Store | 데이터 저장소 (예: MongoDB) | 컨테이너 1개 |
| 7 | RDBMS (Legacy DB) | 관계형 데이터베이스 (예: MySQL) | 컨테이너 1개 |

## DockerFile

-이미지(Image) = 조리된 결과물(요리) 언제든 똑같이 꺼내서 먹을 수 있다.
-컨테이너(Container) = 이미지를 실제로 실행한 것(접시에 담아 손님에게 내놓은 요리)
-Dockerfile = 이미지를 만드는 레시피(재료와 만드는 순서가 적힌 문서)

### Dockerfile 개요
- Dockerfile이란 컨테이너를 만들 수 있는 명령들의 집합을 의미한다.
- 도커는 이 파일을 위에서 아래로(Top-Down) 한 줄씩 읽어서 이미지를 만든다.
- 한 줄 한 줄이 도커 고유의 지시어(Instruction)이며, 각각이 이미지에 어떤 작업을 추가할지를 명령하는 역할
- 지시어는 대/소문자를 구분하지 않지만 관례적으로 명령어는 모두 대문자로 써서 가독성을 높인다.

-Dockerfile 작성 → docker build로 이미지 생성 → docker run으로 컨테이너 실행

**정리**: MSA, Polyglot, Dockerfile의 개념을 정리했다. 다음 절에서는 Dockerfile을 구성하는 각 지시어(FROM, COPY, RUN 등)의 문법을 하나씩 살펴본다.

---

# Dockerfile 문법

-Dockerfile은 Docker Image를 만들기 위한 설정과 명령을 순서대로 작성하는 파일이다.

-Dockerfile에 Base Image, 파일 복사, Package 설치, 실행할 프로그램 등의 내용을 작성하고
 docker build 명령을 실행하면 해당 내용을 기반으로 Docker Image가 생성된다.

-Dockerfile은 일반적으로 확장자 없이 "Dockerfile"이라는 이름으로 작성한다.

```
[guest@Server-A ~]$ mkdir build
[guest@Server-A ~]$ cd build
[guest@Server-A build]$ vi dockerfile
FROM node:12
COPY hello.js /
CMD ["node", "/hello.js"]

: wq

[guest@Server-A build]$ docker build -t imagename:tag .
```

## FROM (필수)

-Docker Image를 만들 때 기반으로 사용할 Base Image를 지정한다.
-쉽게 말하면 "어떤 Image에서부터 새로운 Image를 만들 것인가?"를 지정하는 명령이다.
-일반적으로 Dockerfile의 가장 처음에 작성한다.
-Base Image에는 Linux User Space, Library, Language Runtime, Application 등이 포함될 수 있다.
-Container가 별도의 Kernel을 가지는 것은 아니며, 실행될 때 Host OS의 Kernel을 공유한다.

-형식 : `FROM <Image>:<Tag>`

```
EX) FROM ubuntu:24.04		# Ubuntu 24.04 Image를 Base Image로 사용

EX) FROM node:20-alpine		# Node.js 20이 설치된 Alpine Linux 기반 Image를 사용

EX) FROM nginx:alpine		# Alpine Linux 기반의 Nginx Image를 사용

EX) FROM eclipse-temurin:17-jre	# Java 17 Runtime이 포함된 Image를 사용
```

-Tag를 생략하면 일반적으로 latest Tag가 사용되지만, 실습 및 운영 환경에서는 사용할 Version을 명확하게 지정하는 것이 좋다.

## LABEL

-Image에 대한 설명 정보를 Metadata 형태로 추가한다.
-Image의 작성자, Version, 설명, Source 등의 관리 정보를 기록할 때 사용한다.
-LABEL에 기록한 내용은 Application 실행에 직접적인 영향을 주는 설정이 아니라 Image를 관리하기 위한 부가 정보이다.

-형식 : `LABEL <Key>=<Value>`

```
EX) LABEL maintainer="soldesk@example.com"

EX) LABEL version="1.0"

EX) LABEL description="Web Application Image"
```

-여러 개의 정보를 작성할 수도 있다.

```
EX)

LABEL maintainer="soldesk@example.com" \
      version="1.0" \
      description="Web Application Image"
```

## COPY

-Build Context에 있는 파일이나 디렉터리를 Docker Image 내부로 복사한다.
-주로 Application Source Code, 설정 파일, HTML 파일, 실행 파일 등을 Image에 포함시킬 때 사용한다.

-형식 : `COPY <Source> <Destination>`

```
EX) Nginx
COPY index.html  /usr/share/nginx/html/index.html
 : Build Context에 있는 index.html 파일을 Nginx Image 내부의 /usr/share/nginx/html/index.html로 복사

EX) Apache HTTP Server
COPY public/index.html   /usr/local/apache2/htdocs/index.html
 : public/index.html 파일을 Apache의 Web Root에 복사

EX)
WORKDIR /app
COPY <복사할 원본> <복사될 위치>
 # 첫 번째 .	: Build Context의 현재 디렉터리
 # 두 번째 .	: WORKDIR로 지정된 /app 디렉터리

 # 즉, Build Context의 파일을 Image 내부 /app으로 복사
```

-COPY는 Host의 아무 파일이나 가져오는 명령이 아니다.
-Source 파일은 기본적으로 docker build에서 지정한 Build Context 내부에 있어야 한다.

## ADD

-ADD는 COPY와 마찬가지로 파일이나 디렉터리를 Docker Image 내부에 추가하는 명령이다.
-COPY보다 몇 가지 추가 기능을 가지고 있다.
-예를 들어 Local의 tar 압축 파일을 ADD하면 Image 내부에서 자동으로 압축을 해제할 수 있다.

```
EX) ADD app.tar.gz /app/
 # app.tar.gz의 내용이 /app 디렉터리에 압축 해제되어 추가될 수 있다.
```

-일반적인 파일 복사는 COPY를 사용하는 것이 좋고, ADD의 추가 기능이 필요한 경우에 ADD를 사용하는 것이 좋다.

-처음 Dockerfile을 작성할 때
- 단순 파일 복사 : COPY
- ADD의 추가 기능 필요 : ADD

## RUN

-Docker Image를 Build하는 과정에서 명령을 실행한다.
-Package 설치, 파일 생성, 권한 변경, Source Build 등의 작업에 사용한다.
-RUN으로 실행된 작업 결과는 만들어지는 Image에 반영된다.

```
EX) Rocky Linux
RUN dnf install -y curl			# Image를 Build하면서 curl Package를 설치


EX) Ubuntu Linux
RUN apt-get update && apt-get install -y curl	# Package 목록을 갱신한 후 curl Package를 설치
```

-여러 명령을 &&로 연결하여 하나의 RUN에서 실행할 수도 있다.

```
EX)

RUN apt-get update && \
    apt-get install -y curl && \
    rm -rf /var/lib/apt/lists/*
```

-RUN은 Container를 실행할 때 실행되는 명령이 아니다.
- RUN : Image Build 시 실행
- CMD : Container 시작 시 실행
- ENTRYPOINT : Container 시작 시 실행

```
EX) RUN apt-get install -y nginx	 	# Image를 만들면서 nginx 설치

EX) CMD ["nginx", "-g", "daemon off;"]		# 만들어진 Image로 Container를 실행할 때 nginx 실행
```

## WORKDIR

-Dockerfile 내부에서 작업할 기본 디렉터리를 지정한다.
-Linux에서 cd 명령으로 작업 디렉터리를 이동하는 것과 비슷한 역할을 한다.
-이후 RUN, CMD, ENTRYPOINT, COPY, ADD 등의 명령에서 상대 경로를 사용할 경우
 WORKDIR로 지정한 디렉터리를 기준으로 처리된다.
-지정한 디렉터리가 존재하지 않으면 Docker가 생성한다.

```
EX) WORKDIR /app		# WORKDIR을 사용하면 다음처럼 명령마다 cd를 반복해서 작성할 필요가 없다.
```

## ENV

-Image 및 Container에서 사용할 환경 변수(Environment Variable)를 설정한다.
-Application의 실행 환경, Port, 경로 등의 설정값을 지정할 때 사용한다.

-형식 : `ENV <Key>=<Value>`

```
EX)
ENV APP_ENV=production
ENV PORT=8080
ENV APP_HOME=/app
```

-설정한 환경 변수는 이후 Dockerfile에서도 사용할 수 있다.

```
EX)
ENV APP_HOME=/app
WORKDIR $APP_HOME
```

-위의 WORKDIR은 /app으로 설정된다.
-Container를 실행할 때 -e 옵션을 사용하여 환경 변수 값을 변경할 수도 있다.

```
EX) docker run -e APP_ENV=test myapp
```

-Dockerfile에는 production으로 설정되어 있더라도 Container 실행 시 test로 변경하여 사용할 수 있다.

## VOLUME

-Container에서 생성되는 데이터를 Container 자체와 분리하여 저장할 수 있도록 특정 디렉터리를 Volume Mount 지점으로 지정한다.

-형식 : `VOLUME ["<Container Directory>"]`

```
EX) VOLUME ["/data"]
 # 위 명령은 Container 내부의 /data 디렉터리를 Volume을 사용할 수 있는 Mount 지점으로 선언한다.
```

-특히 Database Container에서는 데이터 보존이 중요하다.

```
EX) MySQL
/var/lib/mysql
```

-MySQL은 일반적으로 /var/lib/mysql 디렉터리에 Database Data를 저장한다.

-이 데이터를 Container의 Writable Layer에만 저장하고 Container 자체를 삭제하면
 해당 Container에 저장되어 있던 데이터도 함께 제거될 수 있다.

-Volume을 사용하면 Container와 Data의 생명주기를 분리하여 관리할 수 있다.

```
EX) docker run -v mysql-data:/var/lib/mysql mysql
 # mysql-data	: Docker가 관리하는 Named Volume
 # /var/lib/mysql	: MySQL Container 내부의 데이터 저장 경로
```

-Container를 삭제하더라도 mysql-data Volume을 별도로 삭제하지 않았다면
 새로운 MySQL Container에서 다시 연결하여 데이터를 사용할 수 있다.

-Container를 단순히 stop/start 또는 restart 하는 경우에는
  Container의 Writable Layer 자체가 사라지는 것이 아니다.

-Volume의 중요한 목적은 Container가 삭제되거나 교체되어도
  데이터를 Container와 독립적으로 관리할 수 있도록 하는 것이다.

-Dockerfile의 VOLUME은 특정 Host Directory를 직접 지정하는 명령이 아니다.
  실제 Volume 연결은 docker run의 -v 또는 --mount 옵션 등으로 지정할 수 있다.

## USER

-Container 내부에서 명령이나 Application을 실행할 사용자 계정을 지정한다.
-USER 이후에 실행되는 RUN과 Container 실행 시 CMD, ENTRYPOINT 등에 해당 사용자 설정이 적용된다.
-Docker Container의 기본 사용자는 일반적으로 root이다.
-보안 강화를 위해 Application을 root가 아닌 일반 사용자 권한으로 실행할 때 사용한다.

-형식 : `USER <User>`
    OR
-형식 : `USER <User>:<Group>`

```
EX) USER appuser

EX) USER appuser:appgroup
```

-필요한 Package 설치나 시스템 설정은 root 권한으로 수행한 후
 Application 실행 단계에서 일반 사용자로 변경하는 방법을 사용할 수 있다.

## EXPOSE

-Container에서 Application이 사용할 예정인 Port 정보를 Image의 Metadata에 기록한다.
-쉽게 말하면 "이 Application은 Container 내부에서 이 Port를 사용한다."라는 정보를 표시하는 것이다.

```
EX) EXPOSE 80

EX) EXPOSE 8080
```

-중요한 점은 EXPOSE를 작성한다고 Host와 Container의 Port가 실제로 연결되는 것은 아니라는 것이다.
-실제 Host Port와 Container Port를 연결하려면 docker run의 -p 옵션 등을 사용한다.

```
EX) docker run -p 8080:80 nginx
 # Host의 8080 Port로 들어온 요청을 Container의 80 Port로 전달
 # EXPOSE	: Container가 사용할 Port 정보를 Image에 기록
 # -p		: Host Port와 Container Port를 실제로 연결
```

-EXPOSE는 Firewall Port를 열어주는 명령도 아니다.

## CMD

-Container가 시작될 때 실행할 기본 명령 또는 기본 인자를 지정한다.
-Dockerfile에 CMD를 여러 개 작성할 수는 있지만 실제로는 마지막 CMD만 적용된다.
-일반적으로 JSON 배열 형태의 Exec Form 사용을 권장한다.

```
EX) CMD ["node", "app.js"]
 # Container가 시작될 때 다음 명령을 실행한다.
 # node app.js

EX) CMD ["python", "app.py"]
 # Container가 시작될 때 다음 명령을 실행한다.
 # python app.py
```

-CMD의 중요한 특징은 Container 실행 시 다른 명령으로 쉽게 변경할 수 있다는 것이다.

## ENTRYPOINT

-ENTRYPOINT도 CMD와 마찬가지로 Container가 시작될 때 실행할 명령을 지정한다.
 하지만 CMD보다 Container의 주 실행 프로그램을 명확하게 지정할 때 많이 사용한다.
-CMD와 ENTRYPOINT를 함께 사용하면 ENTRYPOINT는 실행 프로그램, CMD는 기본 인자(argument) 역할을 하도록 구성할 수 있다.

```
EX)
ENTRYPOINT ["python"]
CMD ["main.py"]
```

-Container를 기본 실행 : `docker run myapp`
-실제 실행되는 명령은 : `python main.py`

-Container 실행 시 다른 값을 전달하면 CMD의 기본 인자를 변경할 수 있다.

```
EX) docker run myapp test.py
 # 실제 실행 명령 : python test.py
```

-즉, ENTRYPOINT ["python"] 은 유지되고 CMD ["main.py"] 부분이 test.py로 변경된 것이다.

**CMD**
- Container가 실행될 때 사용할 기본 명령 또는 기본 인자를 지정
- docker run 뒤에 다른 명령을 작성하면 쉽게 대체 가능

**ENTRYPOINT**
- Container의 주 실행 프로그램을 지정
- docker run 뒤의 값은 일반적으로 ENTRYPOINT에 전달되는 인자로 사용
- --entrypoint 옵션을 사용하면 ENTRYPOINT 자체도 변경 가능

## HEALTHCHECK

-Container 내부의 Application이 실제로 정상 동작하고 있는지 주기적으로 검사한다.
-Container Process가 실행 중이라고 해서 Application 서비스까지 정상이라는 의미는 아니다.
-예를 들어 Web Application Process는 살아 있지만 내부 오류로 HTTP 요청에 응답하지 못할 수도 있다.
-HEALTHCHECK를 사용하면 이러한 Application의 상태를 검사할 수 있다.

```
EX)
HEALTHCHECK --interval=30s --timeout=5s --retries=3 \
CMD curl -f http://localhost:8080 || exit 1
```

### Option

`--interval=30s`
- 30초마다 Health Check 실행

`--timeout=5s`
- 검사 명령이 5초 안에 완료되지 않으면 해당 검사를 실패로 처리

`--retries=3`
- 연속 3회 검사에 실패하면 Container를 unhealthy 상태로 표시

`curl -f http://localhost:8080`
- Container 내부에서 localhost:8080으로 HTTP 요청을 보내 정상 응답 여부 확인

`|| exit 1`
- curl 명령이 실패했을 경우 종료 코드 1을 반환

-Health Check 결과에 따라 Container의 상태에 다음 정보가 표시될 수 있다.
- starting : Health Check가 아직 초기 검사 중
- healthy : 정상
- unhealthy : 연속 검사 실패로 비정상 상태

-상태는 docker ps 또는 docker inspect 등을 이용하여 확인할 수 있다.

-HEALTHCHECK에서 curl을 사용하려면 해당 Docker Image 내부에 curl 명령이 설치되어 있어야 한다.
-unhealthy가 되었다고 Docker Engine이 해당 Container를 자동으로 재시작하는 것은 아니다.
  HEALTHCHECK는 기본적으로 Container의 상태를 판단하고 표시하기 위한 기능이다.

## 주요 Dockerfile 명령 정리

- FROM : 새로운 Image의 Base Image 지정
- LABEL : Image에 설명 및 관리용 Metadata 추가
- COPY : Build Context의 파일이나 디렉터리를 Image 내부로 복사
- ADD : COPY와 비슷하지만 압축 해제 등의 추가 기능 제공
- RUN : Image Build 과정에서 명령 실행
- WORKDIR : Dockerfile 내부의 기본 작업 디렉터리 설정
- ENV : Image 및 Container에서 사용할 환경 변수 설정
- VOLUME : Container의 Volume Mount 지점 지정
- USER : 명령 및 Application을 실행할 사용자 지정
- EXPOSE : Container가 사용할 Port 정보를 Image Metadata에 기록
- CMD : Container 시작 시 사용할 기본 명령 또는 기본 인자 지정
- ENTRYPOINT : Container 시작 시 실행할 주 실행 프로그램 지정
- HEALTHCHECK : Container 내부 Application의 정상 동작 여부 검사

## Java Spring Boot 애플리케이션 예제

```dockerfile
FROM openjdk:17-jdk-slim
WORKDIR /app
COPY target/demo-0.0.1-SNAPSHOT.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

- `FROM openjdk:17-jdk-slim` : 베이스 이미지를 자바 17 JDK가 들어있는 슬림 버전
- `WORKDIR /app` : 이후 명령(COPY, RUN, ENTRYPOINT 등)의 기본 작업 디렉터리를 /app으로 설정
- `COPY target/demo-0.0.1-SNAPSHOT.jar app.jar` : 호스트의 target 폴더에 빌드된 JAR 파일을 컨테이너 /app/app.jar 로 복사
- `EXPOSE 8080` : 엔진이 문서로 참고하기 위한 표시지(방화벽을 열거나 통신을 허용하지 않는다.)
- `ENTRYPOINT ["java", "-jar", "app.jar"]` : 컨테이너가 시작될 때 실행할 명령

## React 프론트엔드 빌드 + Nginx 배포 예제

```dockerfile
FROM node:20-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 3000
CMD ["node", "server.js"]
```

- `FROM node:20-alpine` : 베이스 이미지를 Node.js 20이 설치된 경량 Alpine 리눅스
- `WORKDIR /app` : 이후 모든 명령(COPY, RUN, CMD 등)의 기본 작업 디렉터리를 /app으로 설정
- `COPY package*.json ./` : package.json, package-lock.json(또는 npm-shrinkwrap.json)을 먼저 복사
- `RUN npm install` : 의존성 설치. 위 단계의 파일이 변하지 않으면 캐시가 재사용되어 속도가 빠르다.
- `COPY . .` : 나머지 애플리케이션 소스와 설정 파일을 모두 복사
- `EXPOSE 3000` : 컨테이너가 내부적으로 3000포트를 사용한다는 메타데이터를 문서
- `CMD ["node", "server.js"]` : 컨테이너가 시작될 때 node server.js를 실행하라는 기본 실행 명령

**정리**: Dockerfile의 주요 지시어(FROM, LABEL, COPY, ADD, RUN, WORKDIR, ENV, VOLUME, USER, EXPOSE, CMD, ENTRYPOINT, HEALTHCHECK)와 실전 예제를 살펴봤다. 이제 실제로 Node.js 애플리케이션을 이미지로 빌드하고 실행하는 전체 실습 과정을 진행한다.

---

# Docker Image 생성 실습 (Node.js)

```
[guest@Server-A build]$ mkdir  hellojs
```

```
[guest@Server-A build]$ ls -l
합계 4
-rw-r--r-- 1 guest guest 55  8월  5 11:54 dockerfile
drwxr-xr-x 2 guest guest  6  8월  5 12:46 hellojs
```

```
[guest@Server-A build]$ rm -rf dockerfile
```

```
[guest@Server-A build]$ ls -l
합계 0
drwxr-xr-x 2 guest guest 6  8월  5 12:46 hellojs
```

```
[guest@Server-A build]$ touch ./hellojs/hello.js		# js 파일 생성
```

```
[guest@Server-A build]$ ls -l ./hellojs/hello.js
-rw-r--r-- 1 guest guest 0  8월  5 12:48 ./hellojs/hello.js	# js 파일 확인
```

```
[guest@Server-A build]$ vi ./hellojs/hello.js
const http = require('http');
const os = require('os');

console.log("=======================================");
console.log("Node.js Test Server Starting...");
console.log("Hostname:", os.hostname());
console.log("=======================================");

const PORT = process.env.PORT || 8080;

const handler = function (req, res) {
    const clientIP = req.socket.remoteAddress;
    console.log(`[Request] From: ${clientIP} | URL: ${req.url} | Method: ${req.method}`);

    try {
        res.writeHead(200, { "Content-Type": "text/plain" });
        res.end(
            `Container Hostname: ${os.hostname()}\n` +
            `Client IP: ${clientIP}\n` +
            `Server Time: ${new Date().toISOString()}\n`
        );
    } catch (error) {
        console.error("[Error]", error);
        res.writeHead(500, { "Content-Type": "text/plain" });
        res.end("Internal Server Error\n");
    }
};

const server = http.createServer(handler);
server.listen(PORT, () => {
    console.log(`Server is running and listening on port ${PORT}`);
});

:wq
```

```
[guest@Server-A build]$ cat ./hellojs/hello.js
const http = require('http');
const os = require('os');

console.log("=======================================");
console.log("Node.js Test Server Starting...");
console.log("Hostname:", os.hostname());
console.log("=======================================");

const PORT = process.env.PORT || 8080;

const handler = function (req, res) {
    const clientIP = req.socket.remoteAddress;
    console.log(`[Request] From: ${clientIP} | URL: ${req.url} | Method: ${req.method}`);

    try {
        res.writeHead(200, { "Content-Type": "text/plain" });
        res.end(
            `Container Hostname: ${os.hostname()}\n` +
            `Client IP: ${clientIP}\n` +
            `Server Time: ${new Date().toISOString()}\n`
        );
    } catch (error) {
        console.error("[Error]", error);
        res.writeHead(500, { "Content-Type": "text/plain" });
        res.end("Internal Server Error\n");
    }
};

const server = http.createServer(handler);
server.listen(PORT, () => {
    console.log(`Server is running and listening on port ${PORT}`);
});
```

## Dockerfile 생성

```
[guest@Server-A build]$ vi  ./hellojs/dockerfile
FROM node:24
COPY  hello.js  /
CMD ["node", "hello.js"]

:wq
```

```
[guest@Server-A build]$ cat  ./hellojs/dockerfile
FROM node:24
COPY  hello.js  /
CMD ['node', '/hello.js']
```

```
[guest@Server-A build]$ pwd
/home/guest/build			# 현재 위치 확인
```

```
[guest@Server-A build]$ ls  -l	# 현재 위치의 파일 및 디렉터리 확인
합계 0
drwxr-xr-x 2 guest guest 40  8월  5 12:58 hellojs
```

```
[guest@Server-A build]$ cd ./hellojs	# 디렉터리 이동
```

```
[guest@Server-A hellojs]$ pwd	# 현재 디렉터리 확인
/home/guest/build/hellojs
```

```
[guest@Server-A hellojs]$ ls  -l
합계 8
-rw-r--r-- 1 guest guest    56  8월  5 12:58 dockerfile	# dockerfile
-rw-r--r-- 1 guest guest 1050  8월  5 12:50 hello.js	# js 파일
```

```
[guest@Server-A hellojs]$ docker build  -t  hellojs:latest  .
```

- `bulild` : dockerfile을 읽어서 dockerfile안의 설정대로 이미지를 생성
- `-t` : --tag로 해당 이미지 생성시 이미지 이름을 "이미지명:태그명" 형태로 생성
- `hellojs:latest` : 이미지명:태그명

```
[guest@Server-A hellojs]$ docker build  -t  hellojs:latest  .
[+] Building 38.5s (7/7) FINISHED                                   docker:default
 => [internal] load build definition from dockerfile                          			0.0s
 => => transferring dockerfile: 93B                                           			0.0s
 => [internal] load metadata for docker.io/library/node:24                    		5.0s
 => [internal] load .dockerignore                                             			0.0s
 => => transferring context: 2B                                               			0.0s
 => [internal] load build context                                             			0.0s
 => => transferring context: 1.09kB                                           			0.0s
 => [1/2] FROM docker.io/library/node:24@sha256:da4221677e02b54ef6335adfa44  	33.3s
 => => resolve docker.io/library/node:24@sha256:da4221677e02b54ef6335adfa447  	0.0s
 => => sha256:a8a1300f167248399db1a2150207842a8ab704553b7d2d0673 448B / 448B  	0.5s
 => => sha256:b85f93d4f8f64f32a8187318997c1c4d4abd8b9ccf0b 59.12MB / 59.12MB  	2.4s
 => => sha256:82cfaa427f84dea5fdc6767506c4a618dcaa22ac0ee5dd 1.25MB / 1.25MB  	3.2s
 => => sha256:34e7f1cb7a296e8a2f35401d96a0c2fc030894e20 211.63MB / 211.63MB  	26.5s
 => => sha256:87bb985080b3f921cf426b26d8450d84a06b982ba1f0f8 3.33kB / 3.33kB  	2.7s
 => => sha256:2dd2dd4f152bd44fe5b02de3e47483f6cdf32bf3d7ea 64.41MB / 64.41MB  	8.8s
 => => sha256:bd0ec93c9c52acfa7f522ce201898ba8ebdf67a6d01c 24.04MB / 24.04MB  	2.0s
 => => sha256:c4013e1e38341061b49f51c4b44f2e534c25135d7ed 48.50MB / 48.50MB  	19.8s
 => => extracting sha256:c4013e1e38341061b49f51c4b44f2e534c25135d7ed5b75af8b  	2.4s
 => => extracting sha256:bd0ec93c9c52acfa7f522ce201898ba8ebdf67a6d01c6a4ea70  	0.4s
 => => extracting sha256:2dd2dd4f152bd44fe5b02de3e47483f6cdf32bf3d7ea1c7cc70  	1.5s
 => => extracting sha256:34e7f1cb7a296e8a2f35401d96a0c2fc030894e200451ae7443  	4.3s
 => => extracting sha256:87bb985080b3f921cf426b26d8450d84a06b982ba1f0f84e752  	0.0s
 => => extracting sha256:b85f93d4f8f64f32a8187318997c1c4d4abd8b9ccf0b137f808  	1.5s
 => => extracting sha256:82cfaa427f84dea5fdc6767506c4a618dcaa22ac0ee5dd7d785  	0.0s
 => => extracting sha256:a8a1300f167248399db1a2150207842a8ab704553b7d2d0673f  	0.0s
 => [2/2] COPY  hello.js  /                                                   			0.1s
 => exporting to image                                                        			0.1s
 => => exporting layers                                                       			0.0s
 => => exporting manifest sha256:797a7de57f1a64d0723a2055dbbc59c7201bbdc3468  	0.0s
 => => exporting config sha256:f0dc195946be92026989e9b7792c433fd9dde3c9ff0f1  	0.0s
 => => exporting attestation manifest sha256:dd011813bb8e8f8cf2ae56fad10a7de  		0.0s
 => => exporting manifest list sha256:b47042c6a14fb8e63da30eb545d6a3773d3792  		0.0s
 => => naming to docker.io/library/hellojs:latest                             			0.0s
 => => unpacking to docker.io/library/hellojs:latest                          			0.0s

 1 warning found (use docker --debug to expand):
 - JSONArgsRecommended: JSON arguments recommended for CMD to prevent unintended behavior related to OS signals (line 3)
```

```
[guest@Server-A hellojs]$ docker  images
IMAGE                	ID             	DISK USAGE   	CONTENT SIZE   EXTRA
hello-world:latest	7f4da0fc94bc       	21.8kB         	9.49kB
hellojs:latest       	b47042c6a14f       	1.61GB          	409MB
```

### 베이스 이미지 비교

| 태그 | 기반OS | 특징 | 용량(대략) | 비고 |
|---|---|---|---|---|
| node:24 | Debian (full) | 전체 패키지 포함, 개발용 | 약 1.5GB | apt 가능, 완전한 Linux |
| node:24-slim | Debian (slim) | 최소 패키지, 빠른 빌드 | 약 250MB | 실무/배포용으로 가장 자주 사용 |
| node:24-alpine | Alpine Linux | 매우 작고 빠름 | 약 60MB | 일부 패키지 빌드 실패 가능 |

## 컨테이너 생성

```
[guest@Server-A hellojs]$ docker  run  -d  -p 8080:8080  --name hellojs_con  hellojs:latest
cf7acb6f183030de7ab489a3c028d6dc6748c0c7ae0547fd9077ea485a9b70c5
```

**docker**
- Docker CLI 명령어

**run**
- 새로운 컨테이너를 생성(create) 하고
- 즉시 실행(start) 한다
- create + start를 한 번에 수행

**-d (detached mode)**
- 컨테이너를 백그라운드에서 실행
- 터미널을 점유하지 않음
- 웹서버, DB 같은 서비스형 컨테이너에 사용

**-p 8080:8080 (port mapping)**
- 포트 포워딩(포트 매핑) 옵션

```
[guest@Server-A hellojs]$ docker ps
CONTAINER ID   	IMAGE            COMMAND                   CREATED         STATUS         PORTS                                                   NAMES
bebbdf549594   	hellojs:latest    "docker-entrypoint.s…"   4 seconds ago    Up 3 seconds    0.0.0.0:8080->8080/tcp, [::]:8080->8080/tcp   hellojs_con
```

```
[guest@Server-A hellojs]$ docker  logs  -f hellojs_con
=======================================
Node.js Test Server Starting...
Hostname: bebbdf549594
=======================================
Server is running and listening on port 8080

# docker  	: Docker 실행 명령어
# logs  		: 컨테이너 표준 입력, 출력
# -f 		: follow(실시간 모니터링) 상태를 유지
# hellojs_con	: 컨테이너 이름
```

```
http://192.168.10.100:8080/

Container Hostname: bebbdf549594
Client IP: ::ffff:192.168.10.1
Server Time: 2026-08-05T05:35:08.283Z
```

```
[guest@Server-A hellojs]$ docker  logs  -f hellojs_con
=======================================
Node.js Test Server Starting...
Hostname: bebbdf549594
=======================================
Server is running and listening on port 8080
[Request] From: ::ffff:192.168.10.1 | URL: / | Method: GET
[Request] From: ::ffff:192.168.10.1 | URL: /favicon.ico | Method: GET
```

```
[root@Server-A ~]# curl http://192.168.10.100:8080/~
Container Hostname: bebbdf549594
Client IP: ::ffff:192.168.10.100
Server Time: 2026-08-05T05:36:38.289Z
```

### 네트워크 인터페이스 확인

```
[guest@Server-A hellojs]$ ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: ens160: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:0c:29:66:2f:9c brd ff:ff:ff:ff:ff:ff
    altname enp3s0
    inet 192.168.10.100/24 brd 192.168.10.255 scope global noprefixroute ens160
       valid_lft forever preferred_lft forever
    inet6 fe80::20c:29ff:fe66:2f9c/64 scope link noprefixroute
       valid_lft forever preferred_lft forever
3: docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether 56:6f:ca:1a:18:70 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
    inet6 fe80::546f:caff:fe1a:1870/64 scope link
       valid_lft forever preferred_lft forever
14: veth1735398@if2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP group default
    link/ether b2:c2:1a:54:11:5c brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet6 fe80::b0c2:1aff:fe54:115c/64 scope link
       valid_lft forever preferred_lft forever
```

```
[guest@Server-A hellojs]$ ip addr show ens160
2: ens160: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:0c:29:66:2f:9c brd ff:ff:ff:ff:ff:ff
    altname enp3s0
    inet 192.168.10.100/24 brd 192.168.10.255 scope global noprefixroute ens160
       valid_lft forever preferred_lft forever
    inet6 fe80::20c:29ff:fe66:2f9c/64 scope link noprefixroute
       valid_lft forever preferred_lft forever
```

```
[guest@Server-A hellojs]$ ip addr show docker0
3: docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether 56:6f:ca:1a:18:70 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
    inet6 fe80::546f:caff:fe1a:1870/64 scope link
       valid_lft forever preferred_lft forever
```

-172.17.0.0/16	: Docker가 만드는 내부 브리지 네트워크 대역
-172.17.0.1	: docker0 브리지에서 가지는 호스트가 가진 IP이면서, 컨테이너 입장에서는 게이트웨이 IP 주소

### 요청 흐름

1) 브라우저에서 http://192.168.10.100:8080 요청
2) 호스트(리눅스)의 eth0 인터페이스에서 수신
3) 도커가 자동 등록한 DNAT 규칙 172.17.0.2:8080 으로 목적지 주소 변환 (포트 포워딩)
4) 컨테이너 내부의 Node.js 서버가 응답
5) 응답 시 SNAT을 통해 다시 192.168.10.100으로 변환
6) 브라우저에 응답 전달

```
http://192.168.10.100:8080/

Container Hostname: bebbdf549594
Client IP: ::ffff:192.168.10.1
Server Time: 2026-08-05T05:35:08.283Z
```

## 컨테이너 및 이미지 삭제

### 컨테이너 중지

```
[guest@Server-A hellojs]$ docker stop  hellojs_con
hellojs_con
```

### 컨테이너 삭제

```
[guest@Server-A hellojs]$ docker rm  hellojs_con
hellojs_con
```

### 이미지 삭제

```
[guest@Server-A hellojs]$ docker  rmi  hellojs:latest
Untagged: hellojs:latest
Deleted: sha256:d4b2aec782102e1d21daff9b137055d1050a215283ac895fc19fb797768d2fe4
```

**정리**: Node.js 이미지 빌드부터 컨테이너 실행, 네트워크 요청 흐름(DNAT/SNAT), 삭제까지 전체 흐름을 실습했다. 이어서 Rocky Linux 기반으로 Apache Web Server를 컨테이너화하는 실습을 진행한다.

---

# Rocky Linux 기반 Web Server 실습

```
[guest@Server-A ~]$ mkdir  webserver
```

```
[guest@Server-A ~]$ cd webserver/
[guest@Server-A webserver]$
```

```
[guest@Server-A webserver]$ vi dockerfile
FROM rockylinux:9

LABEL maintainer="Soldesk IT Academy<info@soldesk.co.kr>"

RUN dnf update -y  && dnf  install -y  httpd  &&  dnf clean all

RUN echo"<!DOCTYPE html> \
<html lang='en'> \
<head> \
       <meta charset='utf-8'> \
       <title>Rocky Web Server</title> \
</head> \
<body> \
     <h1>Soldesk Academy</h1> \
    <h2>IT 전문 교육기관</h2> \
    <p>Spring Boot, AWS, Python, 보안, 네트워크 등 다양한 실무 과정을 제공합니다.</p> \
    <p>문의: info@soldesk.co.kr</p> \      
</body> \
</html>"   >  /var/www/html/index.html

EXPOSE 80

CMD ["/usr/sbin/httpd", "-D", "FOREGROUND"]

:wq
```

- `CMD` : 컨테이너가 시작될 때 실행할 기본 명령어를 지정한다.
- `/usr/sbin/httpd` : Apache 웹서버 실행 파일의 절대 경로이다.
- `-D` : Define의 약자로, Apache 실행 시 특정 설정을 활성화하는 옵션이다.
- `FOREGROUND` : Apache를 백그라운드로 보내지 않고 포그라운드에서 계속 실행하도록 지정한다.

-실제로 컨테이너 내부에서는 "/usr/sbin/httpd -D FOREGROUND" 명령어가 실행된다.

-Docker 컨테이너는 주 프로세스가 종료되면 함께 종료된다.
 따라서 Apache를 포그라운드에서 계속 실행하여 컨테이너가 종료되지 않도록 한다

### Apache의 일반 실행 방식 (리눅스 서버에서)
- 일반 리눅스 서버에서는 Apache가 데몬(daemon)으로 실행된다.
- 즉, 백그라운드(background)로 실행되어, 시스템이 계속 관리한다.

### 컨테이너의 기본 개념
- Docker 컨테이너는 하나의 프로세스(PID 1)를 기준으로 동작한다.
- PID 1 프로세스가 컨테이너의 대표(메인 프로세스)가 된다.
  - PID 1 : 컨테이너 안에서 맨 처음 실행되는 프로세스
  - PID 역할 : 컨테이너 전체의 생명과 종료를 관리
  - 종료 시 : PID 1이 종료되면 컨테이너도 종료됨

### Docker 환경의 차이점
- Docker에는 systemd 같은 서비스 관리자가 없다.
- 컨테이너의 PID 1 프로세스 하나만이 주인이다.
- 만약 Apache를 백그라운드로 실행하면 다음 현상이 발생한다:
  1. Apache가 데몬화 → 백그라운드로 감
  2. 시작 명령(apachectl)이 종료됨 (PID 1 종료)
  3. Docker가 컨테이너가 끝났네? 하고 컨테이너를 정지시킴

## Dockerfile을 build하여 이미지 생성

```
[guest@Server-A webserver]$ docker  build  -t  rocky_web:1.0  .
[+] Building 2.8s (7/7) FINISHED                                                      			docker:default
 => [internal] load build definition from dockerfile                                            		0.0s
 => => transferring dockerfile: 629B                                                            		0.0s
 => [internal] load metadata for docker.io/library/rockylinux:9                                 		1.3s
 => [internal] load .dockerignore                                                               			0.0s
 => => transferring context: 2B                                                                 			0.0s
 => [1/3] FROM docker.io/library/rockylinux:9@sha256:d7be1c094cc5845ee815d4632fe377514ee6ebc	0.0s
 => => resolve docker.io/library/rockylinux:9@sha256:d7be1c094cc5845ee815d4632fe377514ee6ebcf8	0.0s
 => CACHED [2/3] RUN dnf update -y  && dnf  install -y  httpd  &&  dnf clean all                	0.0s
 => CACHED [3/3] RUN echo "<!DOCTYPE html> <html lang='en'> <head> <meta charset='utf-8  	0.0s
 => exporting to image                                                                          			1.4s
 => => exporting layers                                                                         			0.0s
 => => exporting manifest sha256:8509f0755296679622d6b69346e6ff19b28e4056a764f12ee599acd1592  	0.0s
 => => exporting config sha256:b2498510dbf992fd2bd5e9954d94847d25728db613317af70095a907b197e  	0.0s
 => => exporting attestation manifest sha256:3b216aa8961782b8c91a5dd005f3454dcf1fa88a2b46842fc  	0.0s
 => => exporting manifest list sha256:02123e55d6662418707addb93901e704cac47665d681f30f858c563  	0.0s
 => => naming to docker.io/library/rocky_web:1.0                                                		0.0s
 => => unpacking to docker.io/library/rocky_web:1.0  
```

```
[guest@Server-A webserver]$ docker images
IMAGE               	ID             	DISK USAGE	CONTENT SIZE   EXTRA
hello-world:latest	7f4da0fc94bc    	21.8kB   		9.49kB
rocky_web:1.0   	bb863614acde     	554MB      	142MB
```

## Dockerfile에 의해 만들어진 이미지로 컨테이너 생성

```
[guest@Server-A webserver]$ docker  run  -d  --name rocky_web  -p 80:80  rocky_web:1.0
447fb444dc74129c2442767fc225751129df9a93443a8a9d77afa54aee324808
```

```
[guest@Server-A webserver]$ docker  ps
CONTAINER ID   IMAGE             COMMAND                   CREATED          STATUS          PORTS                                        NAMES
447fb444dc74       rocky_web:1.0   "/usr/sbin/httpd -D …"    10 seconds ago    Up 10 seconds   0.0.0.0:80->80/tcp, [::]:80->80/tcp   rocky_web
```

```
http://192.168.10.100/

Soldesk Academy
IT 전문 교육기관
Spring Boot, AWS, Python, 보안, 네트워크 등 다양한 실무 과정을 제공합니다.

문의: info@soldesk.co.kr
```

```
[guest@Server-A webserver]$ curl http://192.168.10.100/
<!DOCTYPE html> 
<html lang='en'> 
<head>        
<meta charset='utf-8'>        
<title>Rocky Web Server</title> 
</head> 
<body>      
<h1>Soldesk Academy</h1>     
<h2>IT 전문 교육기관</h2>     
<p>Spring Boot, AWS, Python, 보안, 네트워크 등 다양한 실무 과정을 제공합니다.</p>     
<p>문의: info@soldesk.co.kr</p> 
</body> 
</html>
```

**정리**: Rocky Linux 기반으로 Apache를 컨테이너화하며 PID 1과 포그라운드 실행의 관계, systemd 부재 등 컨테이너 프로세스 모델의 핵심을 실습했다. 이어서 Ubuntu Linux 기반 실습에서는 인코딩(UTF-8) 문제와 ENV 설정을 다룬다.

---

# Ubuntu Linux 기반 Web Server 실습

```
guest@Server-B:~$ pwd
/home/guest
```

## 1) 실습 디렉터리 생성

```
guest@Server-B:~$ apt-get  install  -y  vim

guest@Server-B:~$ mkdir  webserver
```

```
guest@Server-B:~$ cd webserver/
```

```
guest@Server-B:~/webserver$ pwd
/home/guest/webserver
```

## 2) Web Page 작성

```
guest@Server-B:~/webserver$ vi index.html
<!DOCTYPE html> 
<html lang='en'> 
<head>        
    <meta charset='utf-8'>        
    <title>Rocky Web Server</title> 
</head> 
<body>
    <h1>Soldesk Academy</h1>
    <h2>IT 전문 교육기관</h2>
    <p>솔데스크는 실무 중심의 IT 교육을 제공합니다.</p>
    <p>Linux Server, Network, Docker 과정을 운영합니다.</p>
    <p>AWS, Python, 보안 과정도 함께 제공합니다.</p>

    <h2>교육 특징</h2>
    <p>실습 중심의 체계적인 교육을 진행합니다.</p>
    <p>전문 강사진이 수업을 담당합니다.</p>
    <p>자격증 취득과 취업 준비를 지원합니다.</p>

    <h2>교육 문의</h2>
    <p>이메일: info@soldesk.co.kr</p>
</body>
</html>

:wq
```

## 3) Dockerfile 작성

### Apache Web Server

```
guest@Server-B:~/webserver$ vi dockerfile
FROM ubuntu:22.04

LABEL maintainer="Soldesk IT Academy<info@soldesk.co.kr>"

RUN  apt-get update  &&  apt-get  install  -y  apache2 && apt-get clean

COPY  index.html  /var/www/html/index.html

EXPOSE 80

CMD ["apache2ctl", "-D", "FOREGROUND"]
```

### NGINX Web Server

```
guest@Server-B:~/webserver$ vi dockerfile
FROM ubuntu:22.04

LABEL maintainer="Soldesk IT Academy<info@soldesk.co.kr>"

RUN  apt-get update  &&  apt-get  install  -y  nginx && apt-get clean

COPY  index.html  /var/www/html/index.html

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

```
guest@Server-B:~/webserver$ docker  build  -t  ubunweb:1.0  .
```

```
guest@Server-B:~/webserver$ docker  images
IMAGE                	ID             	DISK USAGE   	CONTENT SIZE   	EXTRA
ubunweb:1.0         	7c52d642e8ea        	401MB          	117MB
```

## 컨테이너 생성

```
guest@Server-B:~/webserver$ docker  run  -d  --name  ubun_web -p 80:80  ubunweb:1.0
1dbfb735d74c8b6805f5c4909f13df9d2dc4c714a12364d370f6cca43776b261
```

```
guest@Server-B:~/webserver$ docker  ps
CONTAINER ID   IMAGE          COMMAND                     CREATED          STATUS          PORTS                                       NAMES
1dbfb735d74c      ubunweb:1.0   "apache2ctl -D FOREG…"   35 minutes ago    Up 35 minutes   0.0.0.0:80->80/tcp, [::]:80->80/tcp   ubun_web
```

```
guest@Server-B:~/webserver$ curl  http://192.168.10.200
<!DOCTYPE html>
<html lang='en'>
<head>
    <meta charset='utf-8'>
    <title>Rocky Web Server</title>
</head>
<body>
    <h1>Soldesk Academy</h1>
    <h2>IT 전문 교육기관</h2>
    <p>솔데스크는 실무 중심의 IT 교육을 제공합니다.</p>
    <p>Linux Server, Network, Docker 과정을 운영합니다.</p>
    <p>AWS, Python, 보안 과정도 함께 제공합니다.</p>

    <h2>교육 특징</h2>
    <p>실습 중심의 체계적인 교육을 진행합니다.</p>
    <p>전문 강사진이 수업을 담당합니다.</p>
    <p>자격증 취득과 취업 준비를 지원합니다.</p>

    <h2>교육 문의</h2>
    <p>이메일: info@soldesk.co.kr</p>
</body>
</html>
```

## 컨테이너 접속

```
guest@Server-B:~/webserver$ docker  exec  -it  ubun_web  /bin/bash
root@1dbfb735d74c:/#
root@1dbfb735d74c:/#
root@1dbfb735d74c:/# apt-get  install -y  vim		# vim 설치
```

```
root@1dbfb735d74c:/# vi  /var/www/html/index.html	# vi를 사용하여 확인하게되면 한글이 확인되지 않는다.
<!DOCTYPE html>
<html lang='en'>
<head>
    <meta charset='utf-8'>
    <title>Rocky Web Server</title>
</head>
<body>
    <h1>Soldesk Academy</h1>
    <h2>IT ▒| ~D문 ▒~P▒~\▒기▒~@</h2>
    <p>▒~F~T▒~M▒▒~J▒▒~A▒▒~J~T ▒~K▒무 ▒~Q▒~K▒▒~]~X IT ▒~P▒~\▒▒~]~D ▒| ~\공▒~U▒▒~K~H▒~K▒.</p>
    <p>Linux Server, Network, Docker 과▒| ~U▒~]~D ▒~Z▒▒~X~A▒~U▒▒~K~H▒~K▒.</p>
    <p>AWS, Python, 보▒~U~H 과▒| ~U▒~O~D ▒~U▒▒~X ▒| ~\공▒~U▒▒~K~H▒~K▒.</p>

    <h2>▒~P▒~\▒ ▒~J▒▒~U</h2>
    <p>▒~K▒▒~J▒ ▒~Q▒~K▒▒~]~X 체▒~D▒| ~A▒~]▒ ▒~P▒~\▒▒~]~D ▒~D▒~V~I▒~U▒▒~K~H▒~K▒.</p>
    <p>▒| ~D문 ▒~U▒~B▒▒~D▒~]▒ ▒~H~X▒~W~E▒~]~D ▒~K▒▒~K▒▒~U▒▒~K~H▒~K▒.</p>
    <p>▒~^~P격▒~] 취▒~S~]과 취▒~W~E ▒~@▒~D를 ▒~@▒~[~P▒~U▒▒~K~H▒~K▒.</p>

    <h2>▒~P▒~\▒ 문▒~]~X</h2>
    <p>▒~]▒▒~T▒~]▒: info@soldesk.co.kr</p>
</body>
</html>
```

```
root@1dbfb735d74c:/# export LANG=C.UTF-8		# 환경변수에 UTF-8을 적용

root@1dbfb735d74c:/# export LC_ALL=C.UTF-8		# 환경변수에 UTF-8을 적용
```

```
root@1dbfb735d74c:/# vi  /var/www/html/index.html	# 한글 파일이 확인된다.
<!DOCTYPE html>
<html lang='en'>
<head>
    <meta charset='utf-8'>
    <title>Rocky Web Server</title>
</head>
<body>
    <h1>Soldesk Academy</h1>
    <h2>IT 전문 교육기관</h2>
    <p>솔데스크는 실무 중심의 IT 교육을 제공합니다.</p>
    <p>Linux Server, Network, Docker 과정을 운영합니다.</p>
    <p>AWS, Python, 보안 과정도 함께 제공합니다.</p>

    <h2>교육 특징</h2>
    <p>실습 중심의 체계적인 교육을 진행합니다.</p>
    <p>전문 강사진이 수업을 담당합니다.</p>
    <p>자격증 취득과 취업 준비를 지원합니다.</p>

    <h2>교육 문의</h2>
    <p>이메일: info@soldesk.co.kr</p>
</body>
</html>
```

-이 문제를 해결하기 위해서는 dockerfile 생성시 ENV를 사용하여 환경변수를 미리 생성해야 한다.

## Dockerfile 작성시 ENV 적용

### Apache Web Server

```
guest@Server-B:~/webserver$ vi dockerfile
FROM ubuntu:22.04

LABEL maintainer="Soldesk IT Academy<info@soldesk.co.kr>"

ENV  LANG=C.UTF-8		# 환경변수에 UTF-8 적용
ENV  LC_ALL=C.UTF-8		# 환경변수에 UTF-8 적용

RUN apt-get update && apt-get install -y apache2 && apt-get clean && apt-get install -y vim

COPY  index.html  /var/www/html/index.html

EXPOSE 80

CMD ["apache2ctl", "-D", "FOREGROUND"]


:wq
```

```
guest@Server-B:~/webserver$ docker  build  -t ubunweb:1.1 .
```

```
guest@Server-B:~/webserver$ docker  run  -d  --name  ubun_web2  -p 81:81  ubunweb:1.1
76f81b38d8c1bceb7146ddd2569fdcd5d1620a88362f319798221ad931b29f01
```

```
root@76f81b38d8c1:/# apt-get install -y  vim
```

```
root@76f81b38d8c1:/# vi  /var/www/html/index.html
<!DOCTYPE html>
<html lang='en'>
<head>
    <meta charset='utf-8'>
    <title>Rocky Web Server</title>
</head>
<body>
    <h1>Soldesk Academy</h1>
    <h2>IT 전문 교육기관</h2>
    <p>솔데스크는 실무 중심의 IT 교육을 제공합니다.</p>
    <p>Linux Server, Network, Docker 과정을 운영합니다.</p>
    <p>AWS, Python, 보안 과정도 함께 제공합니다.</p>

    <h2>교육 특징</h2>
    <p>실습 중심의 체계적인 교육을 진행합니다.</p>
    <p>전문 강사진이 수업을 담당합니다.</p>
    <p>자격증 취득과 취업 준비를 지원합니다.</p>

    <h2>교육 문의</h2>
    <p>이메일: info@soldesk.co.kr</p>
</body>
</html>
```

**정리**: Ubuntu 기반 웹서버 실습을 통해 한글 인코딩 문제와 Dockerfile ENV 설정의 필요성을 확인했다. 이어서 데이터베이스(MySQL) 이미지를 초기화 SQL과 함께 빌드하는 실습을 진행한다.

---

# Dockerfile을 이용한 MySQL Image 생성

-Dockerfile을 작성하여 MySQL이 설치된 새로운 Docker Image를 생성한다.
-MySQL 공식 Image를 Base Image로 사용하고 다음과 같이 설정한다.

- MySQL root Password : admin1234
- Database 이름 : soldesk
- 초기 Table : student
- MySQL Port : 3306

```
[guest@Server-A webserver]$ cd ~
```

```
guest@Server-A ~]$ mkdir mysql-build
```

```
[guest@Server-A ~]$ cd mysql-build/
```

-init.sql은 데이터베이스를 처음 만들 때 자동으로 실행할 SQL 명령어를 모아 둔 초기화 파일
-Docker에서 init.sql 파일의 위치 = /docker-entrypoint-initdb.d/

```
[guest@Server-A mysql-build]$ vi  init.sql
CREATE TABLE student (
    id INT PRIMARY KEY,
    name VARCHAR(20),
    age INT
);

INSERT INTO student VALUES (1, 'kim', 20);
INSERT INTO student VALUES (2, 'lee', 25);
INSERT INTO student VALUES (3, 'park', 30);

:wq
```

## Dockerfile 생성

```
[guest@Server-A mysql-build]$ vi  dockerfile
FROM  mysql:8.4
ENV  MYSQL_ROOT_PASSWORD=admin1234
ENV  MYSQL_DATABASE=soldesk
COPY  init.sql   /docker-entrypoint-initdb.d/
EXPOSE  3306

:wq 
```

```
[guest@Server-A mysql-build]$ pwd
/home/guest/mysql-build
```

```
[guest@Server-A mysql-build]$ ls  -l
합계 8
-rw-r--r-- 1 guest guest 138  8월  5 17:04 dockerfile
-rw-r--r-- 1 guest guest 215  8월  5 16:58 init.sql
```

```
[guest@Server-A mysql-build]$ docker  build  -t  soldesk_mysql:v1.0  .
```

```
[guest@Server-A mysql-build]$ docker  images
IMAGE                	ID             	DISK USAGE	CONTENT SIZE   	EXTRA
hello-world:latest	7f4da0fc94bc       	21.8kB         	9.49kB
rocky_web:1.0      	f2db180a7ad0        	554MB          	142MB
soldesk_mysql:v1.0	a2bd7a0f53a8        	1.1GB          	239MB
```

```
[guest@Server-A mysql-build]$ docker  run  -d  --name sol-mysql -p 3306:3306  soldesk_mysql:v1.0        6c85bbfe7a8a7868b13be97a08f1ec038efd5a58ffaa54ed4c246db53dadab31
docker: Error response from daemon: failed to set up container networking: driver failed programming external connectivity on endpoint sol-mysql (c091bae77f0098fc843c9a76ff70e2671e6b29403a7ecfba3b8266975e86a1bd): failed to bind host port 0.0.0.0:3306/tcp: address already in use

Run 'docker run --help' for more information
```

-해당 서버의 누군가 TCP 3306번 Port를 사용하고있다.

```
[guest@Server-A mysql-build]$ sudo ss  -lntp | grep :3306
[sudo] guest 암호:
LISTEN 0      80           0.0.0.0:3306      0.0.0.0:*    users:(("mariadbd",pid=931,fd=25))    

# ss는 Socket Statistics의 약자로 Linux에서 현재 열려 있는 포트, 네트워크 연결 상태, 소켓 정보를 확인하는 명령어
# -l : Listen 상태의 포트만 출력
# -n : 포트 번호를 숫자로 출력
# -t : TCP 포트 출력
# -p : 해당 포트를 사용하는 프로세스 정보 출력
```

```
[guest@Server-A mysql-build]$ sudo netstat  -lntp | grep :3306
tcp        0      0 0.0.0.0:3306            0.0.0.0:*               LISTEN      931/mariadbd

# netstat는 Network Statistics의 약자로 네트워크 연결 상태, 라우팅 정보, 포트 사용 상태 등을 확인하는 명령어
# -l : Listen 상태의 포트만 출력
# -n : 서비스 이름 대신 IP 주소와 포트 번호를 숫자로 출력
# -t : TCP 포트만 출력
# -p : 해당 포트를 사용하는 프로세스의 PID와 프로그램 이름 출력
# grep :3306 : 출력 결과 중 3306번 포트가 포함된 내용만 검색
```

```
[guest@Server-A mysql-build]$ sudo systemctl  stop  mariadb.service		# mariadb.service 중지
```

```
[guest@Server-A mysql-build]$ docker  rm  sol-mysql
sol-mysql
```

```
[guest@Server-A mysql-build]$ docker  run  -d  --name sol-mysql -p 3306:3306  soldesk_mysql:v1.0
4b9ec29a91e26b43721b4e36c29bbf4bd0641d8efa1e6a9dd84bf2ba69e3d086
```

```
[guest@Server-A mysql-build]$ docker  ps
CONTAINER ID   IMAGE                    COMMAND                  CREATED          STATUS          PORTS                                      NAMES
4b9ec29a91e2      soldesk_mysql:v1.0   "docker-entrypoint.s…"   34 seconds ago    Up 33 seconds   0.0.0.0:3306->3306/tcp, 33060/tcp   sol-mysql
```

## MySQL 접속

```
[guest@Server-A mysql-build]$ docker  exec  -it  sol-mysql  mysql -u root -p
Enter password: admin1234
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 8
Server version: 8.4.11 MySQL Community Server - GPL

Copyright (c) 2000, 2026, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>
```

- `docker  exec  -it  sol-mysql  mysql  -u root  -p`
  - `docker exec` : 실행 중인 Container 내부에서 명령 실행
  - `-it` : Terminal을 이용하여 대화형으로 명령을 입력할 수 있도록 설정
  - `sol-mysql` : 명령을 실행할 Container 이름
  - `mysql` : Container 내부에서 MySQL Client 실행
  - `-u root` : root 계정으로 접속
  - `-p` : Password를 입력하여 접속

```
mysql> SHOW DATABASES;
+--------------------+
| Database           	    |
+--------------------+
| information_schema  |
| mysql              	    |
| performance_schema |
| soldesk            	    |	<----
| sys                	    |
+--------------------+
5 rows in set (0.01 sec)
```

```
mysql> USE soldesk;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql>
```

```
mysql> SHOW TABLES;
+-------------------+
| Tables_in_soldesk	  |
+-------------------+
| student           	  |
+-------------------+
1 row in set (0.00 sec)
```

```
mysql> SELECT * FROM students;
ERROR 1146 (42S02): Table 'soldesk.students' doesn't exist
mysql> SELECT * FROM student;
+----+------+------+
| id | name  | age	|
+----+------+------+
|  1 | kim   |   20 	|
|  2 | lee    |   25 	|
|  3 | park  |   30 	|
+----+------+------+
3 rows in set (0.00 sec)
```

```
http://192.168.10.100

[guest@Server-A mysql-build]$ docker  run  -d  --name sol-mysql  -p 80:80  soldesk_mysql:v1.0

[guest@Server-A mysql-build]$ docker  run  -d  --name sol-mysql2  -p 81:80  soldesk_mysql:v1.0


http://192.168.10.100:81
```

**정리**: init.sql을 이용한 MySQL Image 빌드, 포트 충돌 해결, 컨테이너 접속과 데이터 확인까지의 흐름을 실습했다. 이제 완성된 이미지를 Docker Hub에 배포(login/tag/push)하는 방법을 살펴본다.

---

# Image 배포하기

```
[guest@Server-A ~]$ docker  login		# hub.docker.com 로그인

USING WEB-BASED LOGIN

i Info → To sign in with credentials on the command line, use 'docker login -u <username>'


Your one-time device confirmation code is: DHJK-RWFV
Press ENTER to open your browser or submit your device code here: https://login.docker.com/activate

Waiting for authentication in the browser…
```

```
[guest@Server-A ~]$ docker  info | grep Username
 Username: konan7979
```

```
[guest@Server-A ~]$ docker  tag  soldesk_mysql:v1.0    konan7979/soldesk_mysql:v1.0
```

```
[guest@Server-A ~]$ docker images
IMAGE                          	  ID             	DISK USAGE   	CONTENT SIZE   EXTRA
konan7979/soldesk_mysql:v1.0	  a2bd7a0f53a8     	1.1GB          	239MB			# ID가 동일하다.
nginx:latest                   	  8541484afbc9      	238MB           	66MB
rocky_web:1.0                 	  f2db180a7ad0      	554MB          	142MB
soldesk_mysql:v1.0             	  a2bd7a0f53a8     	1.1GB          	239MB			# ID가 동일하다.
```

```
[guest@Server-A ~]$ docker  push  konan7979/soldesk_mysql:v1.0
The push refers to repository [docker.io/konan7979/soldesk_mysql]
44136fa355b3: Pushed
a8ca58bc6ea9: Pushed
30627cea5424: Pushed
50064421612b: Pushed
d0f44f87b588: Pushed
860b7bc67210: Pushed
3efae9596a0b: Pushed
a5cddd18da97: Pushed
289dbe2b4aa0: Pushed
01cb8e5472ee: Pushed
e31ea7613c63: Pushed
9bebc71cfb90: Pushed
b47d50f371f4: Pushed
v1.0: digest: sha256:a2bd7a0f53a853a74cc25ede1f28f13ac0d5eb484618bde600268f9b8b8723da size: 856
```

**정리**: docker login/tag/push를 통해 Docker Hub에 이미지를 배포하는 과정을 실습했다. 마지막으로 Registry의 개념과 Public/Private Registry의 차이, 그리고 사설 Registry 구축 실습을 살펴본다.

---

# Docker Registry

-**레지스트리** : 도커 이미지를 보관, 배포하는 서버 프로그램 또는 서비스
-리포지토리 : 같은 이름의 이미지가 태그만 달리해 모여 있는 폴더 같은 단위
-이미지 : 애플리케이션 실행에 필요한 파일계층(레이어) 묶음, 컨테이너의 설계도
-컨테이너 : 이미지를 실제로 실행한 프로세스.

-푸시(push) : 이미지 파일 묶음을 레지스트리에 업로드
-풀(pull) : 레지스트리에서 이미지 파일 묶음을 다운로드
-컨테이너 자체를 올리고 내리는 게 아니라, 이미지를 주고받는다.

### 왜 레지스트리가 필요한가
- 배포 자동화 : CI/CD 파이프라인이 빌드 후 이미지를 레지스트리에 올리고, 서버들은 해당 이미지를 내려받아 배포
- 버전 관리 : 태그로 버전(예: v1.2.3, 2025-11-12, commit 해시)을 명확히 관리
- 표준화 : 모든 환경(개발/스테이징/운영)에서 같은 이미지를 재현성 있게 사용
- 접근 제어 : 사내용 민감 이미지는 사설(private) 레지스트리로 보호

## Public Registry

-공개된 모든 사용자가 접근 가능한 이미지 저장소
-누구나 이미지를 검색, 다운로드(pull)할 수 있고 일부는 업로드(push)도 가능하다.

**대표 예시**
- Docker Hub : 세계 최대 규모, 공식 이미지 'library/' 네임스페이스 보유
- GitHub Container Registry : GitHub와 연동, 오픈소스 프로젝트 공유에 적합

**장점**
- 설정이 간단하고 바로 사용 가능.
- 오픈소스 프로젝트와의 연계가 쉽다.
- 전 세계 CDN 분산 덕분에 빠른 다운로드 속도

**단점**
- 사내 비공개 코드/이미지를 올릴 수 없음(보안 위험)
- 무료 플랜은 일정 기간 미사용 이미지 자동 삭제
- 외부 서비스 장애 시 내부 배포 파이프라인도 영향 받음

## Private Registry

-인증된 사용자만 접근 가능한 폐쇄형 레지스트리
-조직의 내부 이미지 저장소로, 자체 구축하거나 클라우드 서비스(ECR, ACR 등)로 운영 가능

**구축 방식**
- 온프레미스(On-Premise) : 조직 내부 서버에 직접 설치
- 매니지드 서비스(Managed Service) : AWS, Azure, GCP 등 클라우드에서 제공하는 전용 서비스 사용

**장점**
- 접근 제어, 암호화, 로그 감사 등 강력한 보안 정책 가능.
- 네트워크가 폐쇄되어 외부 의존도 낮음.
- CI/CD 파이프라인에 완전 통합 가능 (빌드 → 테스트 → 푸시 → 배포 자동화)

**단점**
- 초기 구축과 유지보수 부담
- 스토리지 관리(GC, 백업), 인증서 관리 필요
- 네트워크/보안 인프라 구성 복잡

## Registry 실습

```
[guest@Server-A ~]$ docker  pull  httpd	# httpd OFFICIAL 이미지 다운로드
Using default tag: latest
latest: Pulling from library/httpd
c4a216db6600: Pull complete
c271d17baba2: Pull complete
4f4fb700ef54: Pull complete
04951bf53bdd: Pull complete
2c2b87df2933: Pull complete
b5b79efb0f54: Download complete
b40cab778c77: Download complete
Digest: sha256:2920ed8587277d6aa8ea785e143e970835057123dc7bf1199d102c60c80a73bb
Status: Downloaded newer image for httpd:latest
docker.io/library/httpd:latest
```

```
[guest@Server-A ~]$ docker images
IMAGE                          	  ID             	DISK USAGE  	CONTENT SIZE   EXTRA
httpd:latest                   	  2920ed858727     	175MB         	47.6MB
konan7979/soldesk_mysql:v1.0	  a2bd7a0f53a8      	1.1GB          	239MB
nginx:latest                   	  8541484afbc9      	238MB           	66MB
rocky_web:1.0                  	  f2db180a7ad0     	554MB         	142MB
soldesk_mysql:v1.0             	  a2bd7a0f53a8   	1.1GB          	239MB
```

```
[guest@Server-A ~]$ docker search  registry	# registry 검색
NAME                    DESCRIPTION                                      	STARS     OFFICIAL
registry                Distribution implementation for storing and …   	4198      [OK]
okteto/registry                                                          		0
aptrust/registry                                                         		0
goharbor/registry                                                        		1
rootpublic/registry                                                      		0
corpusops/registry                                                       		0
deis/registry           Docker image registry for the Deis open sour… 	12
distribution/registry   WARNING: NOT the registry official image!!! …	57
allingeek/registry      A specialization of registry:2 configured fo…   	4
convox/registry                                                          		0
kontena/registry        Kontena Registry                                 	0
zoined/registry         Private Docker registry based on registry:2      	0
mattford63/registry     The officail docker-registry with python-mys…	0
ibmcom/registry         Docker Image for IBM Cloud private-CE 	2
```

```
[guest@Server-A ~]$ docker  pull  registry	# 사설 저장소를 만들기위해서 registry 다운로드
Using default tag: latest
latest: Pulling from library/registry
8f4e1177a675: Pull complete
e6f31ffc071e: Pull complete
47465c9fe4b1: Pull complete
269b60c1a347: Pull complete
f90c3e905677: Pull complete
fb37447a81d2: Download complete
3f6a72b62a6e: Download complete
Digest: sha256:1be55279f18a2fe1a74edf2664cac61c1bea305b7b4642dab412e7affdcb3e33
Status: Downloaded newer image for registry:latest
docker.io/library/registry:latest
```

```
[guest@Server-A ~]$ docker  images
IMAGE                          	  ID             	DISK USAGE   	CONTENT SIZE   EXTRA
httpd:latest                   	  2920ed858727     	175MB         	47.6MB
konan7979/soldesk_mysql:v1.0	  a2bd7a0f53a8     	1.1GB          	239MB
nginx:latest                   	  8541484afbc9      	238MB           	66MB
registry:latest                	  1be55279f18a     	83.3MB         	20.4MB
rocky_web:1.0                  	  f2db180a7ad0      	554MB          	142MB
soldesk_mysql:v1.0          	  a2bd7a0f53a8      	1.1GB          	239MB
```

### 사용할 이미지가 Local에 없으면 자동으로 PULL한 후 컨테이너를 생성한다.

```
[guest@Server-A ~]$ docker  run  -d  --name sol_reg  -p  5000:5000  --restart  always  registry:3
Unable to find image 'registry:3' locally
3: Pulling from library/registry
Digest: sha256:1be55279f18a2fe1a74edf2664cac61c1bea305b7b4642dab412e7affdcb3e33
Status: Downloaded newer image for registry:3
e766df4f2cbcb9ffd24a40674707759bcc7ccbd2baed69b4ef3e85893be3c694
```

```
[guest@Server-A ~]$ docker  ps
CONTAINER ID   	IMAGE  	  COMMAND                   CREATED          STATUS          PORTS                                                   NAMES
e766df4f2cbc   	registry:3	  "/entrypoint.sh /etc…"    29 seconds ago    Up 28 seconds   0.0.0.0:5000->5000/tcp, [::]:5000->5000/tcp   sol_reg
```

### httpd를 사설 Registry에 PUSH

```
[guest@Server-A ~]$ docker  images
IMAGE                          	  ID             	DISK USAGE   	CONTENT SIZE   EXTRA
httpd:latest                   	  2920ed858727     	175MB         	47.6MB
konan7979/soldesk_mysql:v1.0	  a2bd7a0f53a8     	1.1GB          	239MB
nginx:latest                   	  8541484afbc9      	238MB           	66MB
registry:latest                	  1be55279f18a     	83.3MB         	20.4MB
rocky_web:1.0                  	  f2db180a7ad0      	554MB          	142MB
soldesk_mysql:v1.0          	  a2bd7a0f53a8      	1.1GB          	239MB
```

```
[guest@Server-A ~]$ docker  tag  nginx:latest   localhost:5000/httpd:latest

 # localhost:5000	: 저장소 이름
 # httpd		: 이미지 이름
 # latest		: 태그
```

```
[guest@Server-A ~]$ docker  images
IMAGE                          	  ID             	DISK USAGE   	CONTENT SIZE   	EXTRA
httpd:latest                   	  2920ed858727      	175MB         	47.6MB
konan7979/soldesk_mysql:v1.0	  a2bd7a0f53a8     	1.1GB          	239MB
localhost:5000/httpd:latest    	  8541484afbc9      	238MB           	66MB
nginx:latest                   	  8541484afbc9      	238MB           	66MB
registry:3                     	  1be55279f18a     	83.3MB         	20.4MB    	U
registry:latest                	  1be55279f18a     	83.3MB         	20.4MB    	U
rocky_web:1.0                  	  f2db180a7ad0    	554MB          	142MB
soldesk_mysql:v1.0             	  a2bd7a0f53a8     	1.1GB          	239MB
```

```
[guest@Server-A ~]$ docker  images  localhost:5000/httpd:latest
IMAGE                         	ID             	DISK USAGE 	CONTENT SIZE   EXTRA
localhost:5000/httpd:latest 	8541484afbc9        	238MB           	66MB
```

```
[root@Server-A ~]# curl  http://localhost:5000/v2/_catalog
{"repositories":["httpd"]}
```

```
[guest@Server-A ~]$ docker ps
CONTAINER ID   	IMAGE        COMMAND                   CREATED          STATUS          PORTS                                                   NAMES
e766df4f2cbc   	registry:3    "/entrypoint.sh /etc…"    19 minutes ago     Up 19 minutes   0.0.0.0:5000->5000/tcp, [::]:5000->5000/tcp   sol_reg
```

```
[guest@Server-A ~]$ docker  exec  -it  sol_reg  /bin/sh
/ #
/ #
/ # ls  -l  /var/lib/registry/docker/registry/v2/repositories/
total 0
drwxr-xr-x    5 root     root            55 Aug  6 02:03 httpd		# 업로드한 httpd 이미지가 확인된다.
```

```
[guest@Server-A ~]$ docker  tag  nginx:latest  localhost:5000/nginx		# 다른 세션으로 접속 후 이미지 생성
```

```
[guest@Server-A ~]$ docker  images
IMAGE                          	  ID             	DISK USAGE   	CONTENT SIZE   	EXTRA
httpd:latest                   	  2920ed858727      	175MB         	47.6MB
konan7979/soldesk_mysql:v1.0	  a2bd7a0f53a8     	1.1GB          	239MB
localhost:5000/httpd:latest    	  8541484afbc9      	238MB           	66MB
localhost:5000/nginx:latest    	  8541484afbc9     	238MB           	66MB
nginx:latest                   	  8541484afbc9      	238MB           	66MB
registry:3                     	  1be55279f18a     	83.3MB         	20.4MB    	U
registry:latest                	  1be55279f18a     	83.3MB         	20.4MB    	U
rocky_web:1.0                  	  f2db180a7ad0    	554MB          	142MB
soldesk_mysql:v1.0             	  a2bd7a0f53a8     	1.1GB          	239MB
```

```
[guest@Server-A ~]$ docker  push  localhost:5000/nginx:latest		# 다른 세션으로 접속 후 이미지 PUSH
The push refers to repository [localhost:5000/nginx]
f5de6e85ac74: Mounted from httpd
5a4222b844e8: Mounted from httpd
c0df8d325117: Mounted from httpd
26c307b5e35a: Mounted from httpd
3c55dc422a81: Mounted from httpd
d84ae7b21412: Mounted from httpd
b8b80b9bc028: Mounted from httpd
latest: digest: sha256:963cfe6e75d1c292f66589d7e190b137cf89310414c0c1c5b476dfc61a4fcd0d size: 2290

i Info → Not all multiplatform-content is present and only the available single-platform image was pushed
          sha256:8541484afbc9c8a5a8a99b379568ebbc957f658583ec9448fc43104229c03cf8 -> sha256:963cfe6e75d1c292f66589d7e190b137cf89310414c0c1c5b476dfc61a4fcd0d
```

```
~ # ls  -l  /var/lib/registry/docker/registry/v2/repositories/
total 0
drwxr-xr-x    5 root     root            55 Aug  6 02:03 httpd
drwxr-xr-x    4 root     root            39 Aug  6 02:20 nginx
```

### 우분투 리눅스에서 이미지를 PULL

```
[guest@Server-A ~]$ sudo firewall-cmd  --permanent  --add-port=5000/tcp
[sudo] guest 암호:
success
```

```
[guest@Server-A ~]$ sudo firewall-cmd  --reload
success
```

```
[guest@Server-A ~]$ sudo firewall-cmd  --list-ports
3306/tcp 5000/tcp
```

```
guest@Server-B:~$ docker  pull  nginx:latest		# public registry에서 다운로드
latest: Pulling from library/nginx
3c55dc422a81: Pull complete
d84ae7b21412: Pull complete
c0df8d325117: Pull complete
26c307b5e35a: Pull complete
5a4222b844e8: Pull complete
b8b80b9bc028: Pull complete
f5de6e85ac74: Pull complete
0f03cb4db0ef: Download complete
92fcf0fc2ef2: Download complete
Digest: sha256:8541484afbc9c8a5a8a99b379568ebbc957f658583ec9448fc43104229c03cf8
Status: Downloaded newer image for nginx:latest
docker.io/library/nginx:latest
```

### 우분투 리눅스에 localhost:5000/httpd:latest 이미지 PULL

```
guest@Server-B:~$ docker pull 192.168.10.100:5000/httpd:latest
Error response from daemon: failed to resolve reference "192.168.10.100:5000/httpd:latest": failed to do request: Head "https://192.168.10.100:5000/v2/httpd/manifests/latest": http: server gave HTTP response to HTTPS client
```

-Server-B(우분투)의 Docker가 HTTPS로 접속했지만, Registry는 HTTP로 응답하기 때문에 에러가 발생한다.

-daemon.json : 도커 엔진이 동작시 어떻게 동작할지를 설정하는 파일

```
guest@Server-B:~$ sudo vi /etc/docker/daemon.json
{
  "insecure-registries": ["192.168.10.100:5000"]
}
```

```
guest@Server-B:~$ sudo systemctl  daemon-reload
```

```
guest@Server-B:~$ sudo systemctl  restart  docker
```

```
guest@Server-B:~$ docker  info
Client: Docker Engine - Community
  ~~~~~~~~ 중간 생략 ~~~~~~~~ 
 Insecure Registries:
  192.168.10.100:5000
  127.0.0.0/8
  ::1/128
 Live Resto
  ~~~~~~~~ 중간 생략 ~~~~~~~~ 	
```

```
guest@Server-B:~$ docker images
IMAGE          	ID             	DISK USAGE   	CONTENT SIZE   	EXTRA
nginx:latest   	8541484afbc9        	241MB           	66MB
ubunweb:1.0	7c52d642e8ea        	401MB          	117MB    		U
ubunweb:1.1	853834adc81f        	401MB          	117MB    		U
```

```
guest@Server-B:~$ docker pull 192.168.10.100:5000/httpd:latest
latest: Pulling from httpd
Digest: sha256:963cfe6e75d1c292f66589d7e190b137cf89310414c0c1c5b476dfc61a4fcd0d
Status: Downloaded newer image for 192.168.10.100:5000/httpd:latest
192.168.10.100:5000/httpd:latest
```

```
guest@Server-B:~$ docker images
                                                           i Info →   U  In Use
IMAGE                           ID             DISK USAGE   CONTENT SIZE   EXTRA
192.168.10.100:5000/httpd:latest
                                963cfe6e75d1        238MB         63.1MB
nginx:latest                    8541484afbc9        241MB           66MB
ubunweb:1.0                     7c52d642e8ea        401MB          117MB    U
ubunweb:1.1                     853834adc81f        401MB          117MB    U
guest@Server-B:~$
```

```
guest@Server-B:~$ docker  pull  mysql:8
8: Pulling from library/mysql
9bebc71cfb90: Pull complete
860b7bc67210: Pull complete
30627cea5424: Pull complete
a8ca58bc6ea9: Pull complete
d0f44f87b588: Pull complete
3efae9596a0b: Pull complete
289dbe2b4aa0: Pull complete
a5cddd18da97: Pull complete
01cb8e5472ee: Pull complete
e31ea7613c63: Pull complete
32ca1b8d1938: Download complete
80a9fe861429: Download complete
Digest: sha256:b3b90af2a6552ae30c266fdb7d5dd55f3afb72404bb78d37fe8a23eb857fd3fb
Status: Downloaded newer image for mysql:8
docker.io/library/mysql:8
```

```
guest@Server-B:~$ docker images
IMAGE          			ID             	DISK USAGE   	CONTENT SIZE   	EXTRA
192.168.10.100:5000/httpd:latest	963cfe6e75d1        	238MB         	63.1MB
mysql:8                         		b3b90af2a655       	1.12GB          	255MB
nginx:latest   			8541484afbc9        	241MB           	66MB
ubunweb:1.0			7c52d642e8ea        	401MB          	117MB    		U
ubunweb:1.1			853834adc81f        	401MB          	117MB    		U
```

```
guest@Server-B:~$ docker  tag  mysql:8  192.168.10.100:5000/mysql:v8
```

```
guest@Server-B:~$ docker images
IMAGE          			ID             	DISK USAGE   	CONTENT SIZE   	EXTRA
192.168.10.100:5000/httpd:latest	963cfe6e75d1        	238MB         	63.1MB
192.168.10.100:5000/mysql:v8    	b3b90af2a655       	1.12GB          	255MB
mysql:8                         		b3b90af2a655       	1.12GB          	255MB
nginx:latest   			8541484afbc9        	241MB           	66MB
ubunweb:1.0			7c52d642e8ea        	401MB          	117MB    		U
ubunweb:1.1			853834adc81f        	401MB          	117MB    		U
```

```
                                                           i Info →   U  In Use
IMAGE                           ID             DISK USAGE   CONTENT SIZE   EXTRA
192.168.10.100:5000/httpd:latest
                                963cfe6e75d1        238MB         63.1MB
192.168.10.100:5000/mysql:v8    b3b90af2a655       1.12GB          255MB
mysql:8                         b3b90af2a655       1.12GB          255MB
nginx:latest                    8541484afbc9        241MB           66MB
ubunweb:1.0                     7c52d642e8ea        401MB          117MB    U
ubunweb:1.1                     853834adc81f        401MB          117MB    U
```

### PULL로 다운로드 받은 이미지를 사용하여 컨테이너 생성

```
guest@Server-B:~$ docker run  -d  --name nginx_web  -p 80:80  192.168.10.100:5000/nginx:latest
b8e78878a0e431cf5869d79da9a0b7633a19e6d6141ef0e8fcabc4c2730e0570
```

```
guest@Server-B:~$ docker  ps
CONTAINER ID   IMAGE                                  COMMAND                   CREATED          STATUS          PORTS        	NAMES
b8e78878a0e4     192.168.10.100:5000/httpd:latest   "/docker-entrypoint.…"   48 seconds ago   Up 47 seconds   0.0.0.0:80->80	nginx_web
```

```
guest@Server-B:~$ curl 192.168.10.200
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
```

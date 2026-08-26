# K8s-08. Ingress (경로 기반 라우팅 · Ingress Controller · Canary Deployment)

## 목차

1. [Ingress](#ingress)
2. [Ingress와 Ingress Controller의 관계](#ingress와-ingress-controller의-관계)
3. [Ingress 동작과정 (예시)](#ingress-동작과정-예시)
4. [ingress-nginx 설치](#ingress-nginx-설치)
5. [soldeskweb 데모 실습 (webserver-demo)](#soldeskweb-데모-실습-webserver-demo)
6. [강좌 페이지 만들기](#강좌-페이지-만들기)
7. [로그인 / 회원가입 페이지 (auth)](#로그인--회원가입-페이지-auth)
8. [정규표현식 기반 Ingress (ingress-html.yaml)](#정규표현식-기반-ingress-ingress-htmlyaml)
9. [Canary Deployment](#canary-deployment)
10. [Canary Deployment 실습](#canary-deployment-실습)

## Ingress

**Ingress**는 외부에서 들어오는 HTTP/HTTPS 요청을 받아서, 규칙에 따라 내부의 여러 Service로 연결해주는 라우팅 규칙 집합이다. 서비스가 하나둘뿐이라면 NodePort나 LoadBalancer만으로도 충분하지만, 서비스가 여러 개로 늘어나 경로 기반 라우팅이나 HTTPS 인증서의 중앙 관리, 단일 진입점 구성이 필요해지면 Ingress를 사용한다.

Ingress 자체는 규칙 정의이고 실제로 트래픽을 처리하는 주체는 **Ingress Controller**다.
즉, Ingress는 설정서이고 Ingress Controller는 실제 일하는 로드밸런서라고 생각하면 된다.

### 처음에 왜 NodePort만으로는 부족한가

쿠버네티스에서 파드는 외부에서 직접 접근할 수 없기 때문에 ClusterIP(내부 통신용), NodePort, LoadBalancer 같은 방법을 사용해야 한다.

처음에는 NodePort 하나만으로도 충분하지만 서비스가 늘어나면 문제가 생길 수 있다.

예를 들어 웹 서비스가 3개 있다고 가정해보자.

```
web 서비스   : NodePort 30001
api 서비스   : NodePort 30002
admin 서비스 : NodePort 30003
```

이 방식은 다음과 같은 문제가 있다.

- 포트 번호를 사용자가 직접 기억해야 한다.
- URL 기반 분기가 불가능하다 (/api, /admin 같은 경로 처리 불가)
- 서비스가 많아질수록 포트 관리가 어려워진다.
- HTTPS 인증서를 서비스마다 따로 관리해야 한다.

이 문제를 해결하기 위해 등장한 개념이 Ingress다.

**정리**: NodePort만으로는 포트 관리, URL 기반 분기, HTTPS 인증서 관리가 어려워지므로, 이를 해결하기 위해 Ingress가 등장했다.

---

## Ingress와 Ingress Controller의 관계

### Ingress

- **Ingress**는 쿠버네티스 리소스다.
- 어떤 도메인으로 들어오면 어떤 Service로 보낼지, 어떤 경로로 들어오면 어떤 Service로 보낼지를 정의만 한다.
- 외부에서 들어오는 HTTP/HTTPS 요청을 어떤 Service로 전달할지에 대한 라우팅 규칙을 정의한다.
- Ingress는 주로 도메인(Host)과 경로(Path)를 기준으로 라우팅 규칙을 설정한다.

예를 들어 다음과 같이 설정할 수 있다.

```
example.com        --> web-service
example.com/api    --> api-service
example.com/admin  --> admin-service
```

즉, Ingress는

- 어떤 도메인으로 들어왔는가?
- 어떤 경로로 들어왔는가?
- 조건에 맞는 Service는 무엇인가?

를 정의한다.

Ingress는 라우팅 규칙을 정의할 뿐 실제 HTTP/HTTPS 트래픽을 처리하지 않는다.
따라서 Ingress만 만들어서는 실제 트래픽이 라우팅되지 않는다.

### Ingress Controller

**Ingress Controller**는 Ingress에 정의된 규칙을 실제로 실행하여 트래픽을 처리하는 프로그램이다.

```
Ingress            --> 어디로 보낼지 적어놓은 규칙
Ingress Controller --> 그 규칙을 보고 실제로 보내는 주체
```

Ingress Controller의 역할은 다음과 같다.

- 클러스터 외부에서 들어오는 HTTP/HTTPS 요청을 받는다.
- Kubernetes의 Ingress 리소스를 감시한다.
- Ingress에 정의된 Host와 Path 등의 라우팅 규칙을 확인한다.
- 요청 조건에 맞는 Service를 찾는다.
- 해당 Service로 트래픽을 전달한다.
- 필요에 따라 여러 Pod로 트래픽을 분산한다.
- 즉, Ingress Controller가 실제 HTTP/HTTPS 트래픽을 처리하는 주체다.

### Ingress가 해결해주는 문제

**포트가 아닌 URL 기반 접근** — **경로 기반 라우팅**의 핵심

- 사용자는 포트 번호를 몰라도 된다.
- `http://example.com`       --> web 서비스
- `http://example.com/api`   --> api 서비스
- `http://example.com/admin` --> admin 서비스
- 외부에서는 하나의 주소만 보이고 내부에서만 서비스가 분기된다.

**단일 진입 지점 (Single Entry Point)**

- Ingress는 클러스터의 정문 역할을 한다.
- 외부 트래픽은 무조건 Ingress로 들어온다.
- 내부 서비스는 외부에 직접 노출되지 않는다.

**HTTPS 처리 중앙화** (**TLS** 인증서 중앙 관리)

- 모든 서비스에 공통 HTTPS 적용
- 인증서 관리 위치를 한 곳으로 통합
- 서비스 내부는 HTTP만 사용 가능
- 운영 환경에서는 거의 필수 구조다.

### Ingress의 동작 흐름

1. 사용자가 브라우저에서 `http://example.com/api`를 요청한다.
2. 요청은 클러스터 외부 IP 또는 LoadBalancer IP로 들어온다.
3. 이 요청은 Ingress Controller로 전달된다.
4. Ingress Controller는 Ingress 규칙을 확인한다.
5. `/api` 경로에 매칭되는 Service를 찾는다.
6. 해당 Service 뒤에 연결된 Pod 중 하나로 트래픽을 전달한다.

**중요 포인트**

- Ingress는 Pod를 직접 모른다.
- Ingress는 Service까지만 연결한다.
- 실제 Pod 선택은 Service가 담당한다.

### Ingress와 Service의 역할 분리

- Ingress: 외부 요청 --> 어떤 Service로 보낼지 결정
- Service: 내부 요청 --> 어떤 Pod로 보낼지 결정

즉, Ingress는 길 안내, Service는 최종 목적지 선택. 이렇게 역할이 완전히 분리되어 있다.

**Ingress 사용 예**

- 웹 서비스가 여러 개 존재할 때
- URL 기반 라우팅이 필요할 때
- HTTPS를 적용해야 할 때
- 외부 노출 지점을 하나로 통합하고 싶을 때
- 운영 환경(실무)에 가까운 구조를 만들 때

### 웹 주소의 기본 구조

웹 주소는 두 부분으로 나뉜다.

예시: `http://example.com/api`

- 도메인(host): `example.com`
- 경로(path): `/api`

Ingress는 이 두 가지를 보고 판단한다.
Ingress는 같은 웹 주소로 들어온 요청을 주소 뒤의 경로에 따라 서로 다른 Service로 보내는 규칙이다. 도메인(host) 기준으로 나누면 **호스트 기반 라우팅**, 경로(path) 기준으로 나누면 **경로 기반 라우팅**이다.

### Ingress 기본 예제 (경로 기반)

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web-ingress
spec:
  rules:
  - host: example.com
    http:
      paths:
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: api-service
            port:
              number: 80
      - path: /web
        pathType: Prefix
        backend:
          service:
            name: web-service
            port:
              number: 80
```

이 규칙은 `example.com`으로 들어오는 요청에만 적용된다.

- 주소의 경로가 `/api`로 시작하면 `api-service`로 보낸다.
- 주소의 경로가 `/web`로 시작하면 `web-service`로 보낸다.

```
path: /api
pathType: Prefix
```

이 설정은 경로가 `/api`로 시작하면 이 규칙을 적용한다는 뜻이다.
Ingress는 `/api`라는 문자열까지만 판단한다.

### Ingress가 실제로 판단하는 순서

1. 도메인이 example.com 인가?
2. 경로가 `/api`로 시작하는가? 또는 경로가 `/web`로 시작하는가?
3. 조건에 맞는 Service로 전달

Ingress는 이 외의 일은 하지 않는다.

### Ingress와 Service의 역할 차이

- Ingress: 웹 요청을 보고 어느 Service로 보낼지 결정
- Service: 요청을 받아 Pod 중 하나로 전달

Ingress는 앞에서 길을 고르는 역할이고 Service는 뒤에서 나눠 주는 역할이다.

**정리**: Ingress는 도메인(host)과 경로(path)를 기준으로 어떤 Service로 보낼지만 결정하고, 실제 Pod 분산은 Service가 담당한다는 역할 분리가 이번 절의 핵심이다.

---

## Ingress 동작과정 (예시)

실무에서는 페이지 기준이 아니라 업무 책임(도메인) 기준으로 나눈다.

**auth-service**
- 로그인
- 로그아웃
- 회원가입
- 토큰 발급(JWT 발급/갱신)

**user-service**
- 회원 프로필 조회/수정
- 비밀번호 변경
- 주소/연락처 관리

**product-service**
- 상품 목록 조회
- 상품 상세 조회
- 상품 등록/수정(관리자)
- 재고 조회(간단하면 포함)

**order-service**
- 주문 생성
- 주문 조회(내 주문 목록/상세)
- 결제 요청 생성(결제 준비 단계)
- 주문 상태 변경(결제대기/결제완료/취소 등)

**payment-service**
- 외부 PG 연동(카카오페이/토스/이니시스 등)
- 결제 승인/실패 콜백 처리
- 결제 결과를 order-service에 통보

이 단위가 실무에서 흔한 서비스 분리다.

### 각 서비스는 Pod 1개가 아니라 Pod 여러 개다

각 서비스는 하나의 코드(애플리케이션)이고, 그 애플리케이션을 쿠버네티스에서 여러 번 실행한다.

- auth-service Deployment: auth Pod 3개
- user-service Deployment: user Pod 2개
- product-service Deployment: product Pod 4개 (트래픽이 많으면 더 많은 pod를 사용해야 한다.)
- order-service Deployment: order Pod 3개
- payment-service Deployment: payment Pod 2개

로그인 Pod, 로그아웃 Pod 이렇게 API별로 쪼개는 게 아니라 auth-service 하나에 로그인/로그아웃/회원가입/토큰 발급이 다 들어 있고 그 auth-service를 Pod 3개로 확장하는 방식이다.

### Service(ClusterIP)가 Pod들을 묶어서 내부 주소를 만든다

Pod는 IP가 바뀌고(재시작/재배치) 개수가 늘었다 줄었다 한다. 그래서 Pod에 직접 접속하지 않는다.

각 Deployment 앞에 Service를 1개씩 둔다.

```
auth-service Service
type: ClusterIP
selector: app=auth
```

역할: auth Pod 3개로 로드밸런싱 + 고정 내부 DNS 제공

```yaml
# user-service Service
type: ClusterIP
selector:
  app: user
```

```yaml
# product-service Service
type: ClusterIP
selector:
  app: product
```

```yaml
# order-service Service
type: ClusterIP
selector:
  app: order
```

```yaml
# payment-service Service
type: ClusterIP
selector:
  app: payment
```

이 상태가 되면, 클러스터 내부에서는 서비스 이름으로 통신한다.

order-service가 payment-service를 호출할 때 payment-service(ClusterIP)로 호출한다.

### Ingress는 단일 진입점에서 여러 ClusterIP로 분기한다

지금까지 만든 ClusterIP들은 외부에서 직접 접근이 안 된다.
외부 트래픽을 받는 입구가 필요한데 그 입구가 Ingress다.

Ingress는 외부에서 들어오는 HTTP/HTTPS 요청을 한 곳에서 받고 규칙(host/path)에 따라 내부 Service(ClusterIP)로 전달한다.

예시 규칙(개념):

```
example.com/auth/*     ---> auth-service(ClusterIP)
example.com/users/*    ---> user-service(ClusterIP)
example.com/products/* ---> product-service(ClusterIP)
example.com/orders/*   ---> order-service(ClusterIP)
example.com/payments/* ---> payment-service(ClusterIP)
```

```
외부 사용자
-> Ingress Controller(nginx 같은 것)
-> Ingress Rules 확인
-> 해당 Service(ClusterIP)
-> Service가 Pod들 중 하나로 분산
-> Pod
```

Ingress는 Pod로 직접 보내지 않고, 반드시 Service(ClusterIP)로 보낸다.

**정리**: 실무에서는 도메인/서비스 단위(auth/user/product/order/payment)로 구조를 나누고, Ingress가 단일 진입점에서 host/path 규칙에 따라 각 Service(ClusterIP)로 트래픽을 분기하며, Service가 다시 여러 Pod로 로드밸런싱한다.

---

## ingress-nginx 설치

Ingress 리소스가 실제로 동작하려면 클러스터에 **Ingress Controller**를 먼저 설치해야 한다. 여기서는 대표적인 구현체인 `ingress-nginx`를 설치한다.

참고 사이트:
- https://kubernetes.io/
- https://kubernetes.github.io/ingress-nginx/

```bash
# 설치
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.15.1/deploy/static/provider/baremetal/deploy.yaml

# 다운로드
curl  -O  https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.15.1/deploy/static/provider/baremetal/deploy.yaml
```

**정리**: ingress-nginx 설치가 끝나면 이후부터는 Ingress 리소스를 실제로 apply했을 때 트래픽이 처리된다. 아래부터는 이를 이용한 실습이다.

---

## soldeskweb 데모 실습 (webserver-demo)

앞서 설명한 개념을 실제 클러스터에서 확인하기 위해, 메인 페이지/커리큘럼 페이지를 각각 별도 Deployment/Service로 구성하고 Ingress로 경로 기반 라우팅을 연결하는 실습을 진행한다.

### 파일 전송 및 압축 해제

```
C:\WINDOWS\system32> scp  C:\tmp\soldeskweb.zip   guest@192.168.10.100:/home/guest
guest@192.168.10.100's password:
soldeskweb.zip                                                         100%   25KB  24.8MB/s   00:00
```

```
[root@k8s-master ~]# ls  -l  /home/guest/
합계 68
drwxr-xr-x  2 guest guest         6  8월 14 16:57 exec-liveness-lab
-rw-r--r--  1 root  root        246  8월 13 12:15 mynginx.yaml
-rw-r--r--  1 root  root        152  8월 13 12:15 nginx-pod.yaml
-rw-r--r--  1 root  root        514  8월 13 12:15 nginx.yaml
-rw-r--r--  1 root  root        593  8월 13 12:15 rq-step-3-2-request-mem-pod.yaml
-rw-r--r--  1 root  root        114  8월 13 12:15 rq-step2-qouta.yaml
-rw-r--r--  1 root  root        133  8월 13 12:15 rq-step3-2-request-quota.yaml
-rw-r--r--  1 root  root        587  8월 13 12:15 rq-step3-request-pod.yaml
-rw-r--r--  1 root  root        124  8월 13 12:15 rq-step3-requests-qouta.yaml
-rw-r--r--  1 root  root         57  8월 13 12:15 sol-namespace.yaml
-rw-r--r--  1 guest guest   26045  8월 24 10:47 soldeskweb.zip	<-----
-rw-r--r--  1 root  root       517  8월 13 12:15 step1-baseline.yaml
```

```
[root@k8s-master ~]# ls  -l  /home/guest/soldeskweb.zip
-rw-r--r-- 1 guest guest 26045  8월 24 10:47 /home/guest/soldeskweb.zip

[root@k8s-master ~]# cp  /home/guest/soldeskweb.zip  /root/

[root@k8s-master ~]# ls  -l  soldeskweb.zip
-rw-r--r-- 1 root root 26045  8월 24 10:50 soldeskweb.zip

[root@k8s-master ~]# unzip  soldeskweb.zip
Archive:  soldeskweb.zip
   creating: webserver-demo/
   creating: webserver-demo/curriculum/
  inflating: webserver-demo/curriculum/Dockerfile
  inflating: webserver-demo/curriculum/index.html
   creating: webserver-demo/ingress/
  inflating: webserver-demo/ingress/curriculum.yaml
  inflating: webserver-demo/ingress/ingress.yaml
  inflating: webserver-demo/ingress/sol-home.yaml
   creating: webserver-demo/sol-collection/
  inflating: webserver-demo/sol-collection/Dockerfile
   creating: webserver-demo/sol-collection/html/
   creating: webserver-demo/sol-collection/html/images/
  inflating: webserver-demo/sol-collection/html/images/sol_logo.jpg
  inflating: webserver-demo/sol-collection/html/images/soldesk.jpg
  inflating: webserver-demo/sol-collection/html/index.html
```

초기 디렉터리 구조:

```
soldeskweb/
      └─webserver-demo/
	├─ curriculum/
	│  ├─ Dockerfile
	│  └─ index.html
	│
	├─ ingress/
	│      ├─ curriculum.yaml
	│      ├─ ingress.yaml
	│      └─ sol-home.yaml
	│
	└─ sol-collection/
		├─ Dockerfile
		└─ html/
		    ├─ index.html
		    └─ images/
		        ├─ sol_logo.jpg
		        └─ soldesk.jpg
```

```
[root@k8s-master ~]# ls -l webserver-demo/sol-collection/
합계 4
-rw-r--r-- 1 root root 151 12월 19  2025 Dockerfile
drwxr-xr-x 3 root root  38 12월 19  2025 html

[root@k8s-master ~]# ls -l webserver-demo/sol-collection/html/
합계 4
drwxr-xr-x 2 root root 45 12월 19  2025 images
-rw-r--r-- 1 root root 341  1월 13  2026 index.html

[root@k8s-master ~]# ls -l webserver-demo/sol-collection/html/images/
합계 24
-rw-r--r-- 1 root root  5132 12월 19  2025 sol_logo.jpg
-rw-r--r-- 1 root root 15874 12월 19  2025 soldesk.jpg
```

### 웹서비스: sol-collection (메인 페이지)

Dockerfile (`webserver-demo/sol-collection/Dockerfile`):

```dockerfile
FROM nginx:1.29.1
LABEL maintainer="NGINX Front-end container <soldesk@gmail.com>"

COPY html /usr/share/nginx/html

CMD ["nginx", "-g", "daemon off;"]
```

메인 페이지 HTML (초기, `webserver-demo/sol-collection/html/index.html`):

```html
<html>
<head>
  <meta charset="UTF-8">
  <title>soldesk</title>
</head>
<body>
  <center>
    <img src="images/sol_logo.jpg"><br>
    <p style="color:red;">Soldesk Academy/AWS Cloud</p><br>
    <img src="images/soldesk.jpg"><br>
    <button onclick="location.href='/curriculum/index.html'">커리큘럼</button>
  </center>
</body>
</html>
```

### Docker를 사용하여 이미지 생성

```
[root@k8s-master ~]# cd  /root/webserver-demo/sol-collection/

[root@k8s-master sol-collection]# pwd
/root/webserver-demo/sol-collection

[root@k8s-master sol-collection]# ls -l
합계 4
-rw-r--r-- 1 root root 151 12월 19  2025 Dockerfile
drwxr-xr-x 3 root root 38 12월 19  2025 html

[root@k8s-master sol-collection]# docker  build  -t  konan7979/sol-collection:1.0  .
[+] Building 7.4s (8/8) FINISHED                                                    			docker:default
 => [internal] load build definition from Dockerfile                                          		0.0s
 => => transferring dockerfile: 190B                                                          			0.0s
 => [internal] load metadata for docker.io/library/nginx:1.29.1                               		3.0s
 => [auth] library/nginx:pull token for registry-1.docker.io                                  		0.0s
 => [internal] load .dockerignore                                                             			0.0s
 => => transferring context: 2B                                                               			0.0s
 => [internal] load build context                                                             			0.0s
 => => transferring context: 21.56kB                                                          			0.0s
 => [1/2] FROM docker.io/library/nginx:1.29.1@sha256:8adbdcb969e2676478ee2c7ad333956f0c8e0e4  	4.2s
 => => resolve docker.io/library/nginx:1.29.1@sha256:8adbdcb969e2676478ee2c7ad333956f0c8e0e4  	0.0s
 => => sha256:16d05858bb8d98a948d273ef83ff992f7eb4b7b50b9d92dcb186ec02d6cd1089 955B / 955B    	1.1s
 => => sha256:4f4e50e2076584d483a45b2db7718a03e941045e8dcd0023b6d326b743b282 1.40kB / 1.40kB  	0.4s
 => => sha256:08cfef42fd24116711bc1e323e83d40e6145937250f876e0342c2c90426c3bfb 404B / 404B    	0.8s
 => => sha256:3cc5fdd1317a723bde90305759b954dda6335ade70354d860e82c59588df4e 1.21kB / 1.21kB  	1.1s
 => => sha256:5f825f15e2e0140c77e43d026664718f274284e907b3dbfea8af5c3f2e843673 629B / 629B    	0.7s
 => => sha256:375a694db7346a00da49aac62757cec58667d0c90874d4b08edef1814161 44.07MB / 44.07MB	1.7s
 => => sha256:5c32499ab806884c5725c705c2bf528662d034ed99de13d3205309e0d9ef 28.23MB / 28.23MB	1.8s
 => => extracting sha256:5c32499ab806884c5725c705c2bf528662d034ed99de13d3205309e0d9ef0375     	0.6s
 => => extracting sha256:375a694db7346a00da49aac62757cec58667d0c90874d4b08edef1814161f8f2     	0.7s
 => => extracting sha256:5f825f15e2e0140c77e43d026664718f274284e907b3dbfea8af5c3f2e843673     	0.0s
 => => extracting sha256:16d05858bb8d98a948d273ef83ff992f7eb4b7b50b9d92dcb186ec02d6cd1089     	0.0s
 => => extracting sha256:08cfef42fd24116711bc1e323e83d40e6145937250f876e0342c2c90426c3bfb     	0.0s
 => => extracting sha256:3cc5fdd1317a723bde90305759b954dda6335ade70354d860e82c59588df4e4b     	0.0s
 => => extracting sha256:4f4e50e2076584d483a45b2db7718a03e941045e8dcd0023b6d326b743b282a1     	0.0s
 => [2/2] COPY html /usr/share/nginx/html                                                     		0.1s
 => exporting to image                                                                        			0.1s
 => => exporting layers                                                                       			0.0s
 => => exporting manifest sha256:514baf95ea3c7510077620851fdc2919af5fb9db03dbe18cdeae9d759c8  	0.0s
 => => exporting config sha256:aaf41cd0dacc0877cf10c02b814430d2d6624e078e874d24fed8e2a8cbfef  	0.0s
 => => exporting attestation manifest sha256:1c1425037be73e3abe6e9db3bca48adcc89d7da24ce702f  	0.0s
 => => exporting manifest list sha256:e1edbbc89fd853725eae8ff7ce8b7ad94fbec11be2fb76a99b1fc6  	0.0s
 => => naming to docker.io/konan7979/sol-collection:latest                                    		0.0s
 => => unpacking to docker.io/konan7979/sol-collection:latest                                 		0.0s

[root@k8s-master sol-collection]# docker  images
IMAGE                               	ID             	DISK USAGE   CONTENT SIZE   EXTRA
custom-nginx-web:1.31               	02218a3d52d6        	348MB            97MB
konan7979/custom-nginx-web:1.31     	02218a3d52d6        	348MB            97MB
konan7979/nginx-exec-liveness:1.0   	caeb48d1c8b7        	235MB            63.1MB
konan7979/nginx-liveness:1.0        	75edbab6ae2a        	235MB            63.1MB
konan7979/sol-collection:1.0     	e1edbbc89fd8        	276MB            72.3MB
konan7979/ssh-probe:1.0             	6578cbc133e3       	321MB            81.9MB

[root@k8s-master sol-collection]# docker  push  konan7979/sol-collection:1.0
The push refers to repository [docker.io/konan7979/sol-collection]
e8b1cc07af8d: Pushed
44136fa355b3: Mounted from konan7979/ssh-probe
375a694db734: Pushed
5c32499ab806: Pushed
5f825f15e2e0: Pushed
16d05858bb8d: Pushed
08cfef42fd24: Pushed
3cc5fdd1317a: Pushed
4f4e50e20765: Pushed
ba54b4289f64: Pushed
1.0: digest: sha256:571a4a896a75278579d6b575aebaf986e5ccf420ac21beddf10f0793af30af72 size: 856
```

### ingress 매니페스트 확인

```
[root@k8s-master sol-collection]# ls  -l  /root/webserver-demo/ingress/
합계 12
-rw-r--r-- 1 root root 528 12월 19  2025 curriculum.yaml
-rw-r--r-- 1 root root 523 12월 19  2025 ingress.yaml
-rw-r--r-- 1 root root 514 12월 19  2025 sol-home.yaml
```

`sol-home.yaml` (Deployment + Service):

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: sol-home-deploy
spec:
  replicas: 1
  selector:
    matchLabels:
      app: sol-home

  template:
    metadata:
      labels:
        app: sol-home
    spec:
      containers:
        - name: nginx
          image: konan7979/sol-collection:1.0	# 개별 이미지로 수정
          ports:
            - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: sol-home-service
spec:
  type: ClusterIP
  selector:
    app: sol-home
  ports:
    - port: 80
      targetPort: 80
```

`curriculum.yaml` (Deployment + Service):

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: curriculum-deploy
spec:
  replicas: 2
  selector:
    matchLabels:
      app: curriculum
  template:
    metadata:
      labels:
        app: curriculum
    spec:
      containers:
        - name: nginx
          image: konan7979/curriculum-service:1.0
          ports:
            - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: curriculum-service
spec:
  type: ClusterIP
  selector:
    app: curriculum
  ports:
    - port: 80
      targetPort: 80
```

`ingress.yaml` (초기 버전, 주석 포함):

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress                      		# 리소스 종류: Ingress
metadata:
  name: sol-ingress                	# Ingress 이름

spec:
  ingressClassName: nginx		# nginx Ingress Controller 사용
  rules:
#   -host: www.soldesk.com		# Domain이 있는경우 Domain 설정
    - http:                        		# HTTP 요청에 대한 라우팅 규칙 설정
        paths:
          - path: /          		# / 로 들어오는 요청 처리 (: http://도메인/)
            pathType: Prefix       	# / 로 시작하는 모든 경로를 매칭
            backend:
              service:
                name: sol-home-service	# 요청을 전달할 Service 이름
                port:
                  number: 80             	# Service의 80번 포트로 전달

          - path: /curriculum		# /curriculum 으로 들어오는 요청 처리 (예: http://도메인/curriculum)
            pathType: Prefix       	# /curriculum으로 시작하는 경로를 매칭
            backend:
              service:
                name: curriculum-service	# curriculum Service로 전달
                port:
                  number: 80              	# Service의 80번 포트로 전달
```

이 예제의 `pathType: Prefix`는 앞서 설명한 **경로 기반 라우팅**을 그대로 구현한 것이다.

### Deployment, Service 생성

```
	# Deployment , Service 생성
[root@k8s-master ~]# kubectl apply -f /root/webserver-demo/ingress/sol-home.yaml
deployment.apps/sol-home-deploy created
service/sol-home-service created

[root@k8s-master sol-collection]# kubectl  get  deployments
NAME                 READY   UP-TO-DATE   AVAILABLE   AGE
sol-home-deploy   1/1     1            1           13s

[root@k8s-master sol-collection]# kubectl  get  pods
NAME                                         READY   STATUS    RESTARTS   AGE
sol-home-deploy-745b565968-rl9k7   1/1         Running     0                98s

[root@k8s-master sol-collection]# kubectl  get  service  sol-home-service
NAME                 TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
sol-home-service   ClusterIP   10.97.125.105     <none>             80/TCP   

	# sol-home-service 서비스의 endPoint
[root@k8s-master sol-collection]# kubectl  get  endpoints  sol-home-service
Warning: v1 Endpoints is deprecated in v1.33+; use discovery.k8s.io/v1 EndpointSlice
NAME               	ENDPOINTS	AGE
sol-home-service	10.244.2.4:80	9m34s
```

### Ingress 생성

```
	# Ingress 생성
[root@k8s-master ~]# kubectl apply -f /root/webserver-demo/ingress/ingress.yaml
ingress.networking.k8s.io/sol-ingress created

[root@k8s-master sol-collection]# kubectl  get  ingress
NAME          CLASS   HOSTS   ADDRESS          PORTS   AGE
sol-ingress    nginx     *       192.168.10.101   80      16s

[root@k8s-master sol-collection]# kubectl  get  service  --namespace  ingress-nginx
NAME                                 	TYPE    	   CLUSTER-IP   	EXTERNAL-IP   PORT(S)                      	AGE
ingress-nginx-controller             	NodePort	   10.105.3.121     	<none>            80:32181/TCP,443:30366/TCP   	86m
ingress-nginx-controller-admission   	ClusterIP	   10.101.171.143	<none>            443/TCP                      	86m
```

Pod와 Service는 Ingress 뒤에 있는 내부 서비스이고, Ingress(정확히는 Ingress Controller)가 외부와 클러스터를 연결하는 관문이다.

접속 URL:

```
https://192.168.10.100:30366/
```

### curriculum 서비스

```
	# curriculum 서비스
[root@k8s-master ~]# cd  /root/webserver-demo/curriculum/

[root@k8s-master curriculum]# ls  -l
합계 8
-rw-r--r-- 1 root root  125 12월 19  2025 Dockerfile
-rw-r--r-- 1 root root 3777 12월 19  2025 index.html

[root@k8s-master curriculum]# docker  build  -t  konan7979/curriculum-service:1.0  .

[root@k8s-master curriculum]# docker  images
IMAGE                               	ID             	DISK USAGE	CONTENT SIZE   EXTRA
custom-nginx-web:1.31               	02218a3d52d6        	348MB           	97MB
konan7979/curriculum-service:1.0    	2a2b49481bba        	276MB         	72.3MB
konan7979/custom-nginx-web:1.31     	02218a3d52d6        	348MB           	97MB
konan7979/nginx-exec-liveness:1.0   	caeb48d1c8b7        	235MB         	63.1MB
konan7979/nginx-liveness:1.0        	75edbab6ae2a        	235MB         	63.1MB
konan7979/sol-collection:1.0        	571a4a896a75        	276MB         	72.3MB
konan7979/ssh-probe:1.0             	6578cbc133e3        	321MB         	81.9MB

[root@k8s-master curriculum]# docker  push  konan7979/curriculum-service:1.0
The push refers to repository [docker.io/konan7979/curriculum-service]
a3362294a34d: Pushed
44136fa355b3: Mounted from konan7979/ssh-probe
375a694db734: Mounted from konan7979/sol-collection
5c32499ab806: Mounted from konan7979/sol-collection
5f825f15e2e0: Mounted from konan7979/sol-collection
16d05858bb8d: Mounted from konan7979/sol-collection
08cfef42fd24: Mounted from konan7979/sol-collection
3cc5fdd1317a: Mounted from konan7979/sol-collection
4f4e50e20765: Mounted from konan7979/sol-collection
24b091158190: Pushed
5f3c66393266: Pushed
1.0: digest: sha256:2a2b49481bba9e7db30203649a8bfaf804aed75d94697cca3350a33b164afe5f size: 856

[root@k8s-master curriculum]# kubectl  apply  -f  /root/webserver-demo/ingress/curriculum.yaml
deployment.apps/curriculum-deploy created
service/curriculum-service created

[root@k8s-master curriculum]# kubectl  get  deployments  curriculum-deploy
NAME                READY   UP-TO-DATE   AVAILABLE   AGE
curriculum-deploy   2/2     2            2           25s

[root@k8s-master curriculum]# kubectl  get  pods
NAME                                 	READY   STATUS    RESTARTS   AGE
curriculum-deploy-587bbcd4c5-5vct9   	1/1        Running     0                 6s
curriculum-deploy-587bbcd4c5-fqqsm   	1/1        Running     0                 6s
sol-home-deploy-745b565968-rl9k7     	1/1        Running     0                 46m

[root@k8s-master curriculum]# kubectl  get  service
NAME                 	TYPE        CLUSTER-IP      	EXTERNAL-IP   PORT(S)   AGE
curriculum-service	ClusterIP    10.110.9.135	<none>            80/TCP     81s
kubernetes       	ClusterIP    10.96.0.1       	<none>            443/TCP    12d
sol-home-service  	ClusterIP    10.97.125.105   	<none>            80/TCP     47m

[root@k8s-master curriculum]# kubectl  get  pods  -o wide
NAME                                 	READY   STATUS    RESTARTS   AGE    IP            NODE          NOMINATED NODE   READINESS GATES
curriculum-deploy-587bbcd4c5-5vct9	1/1        Running     0                 3m8s   10.244.2.5   k8s-worker2   <none>           <none>
curriculum-deploy-587bbcd4c5-fqqsm   	1/1        Running     0                 3m8s   10.244.1.5   k8s-worker1   <none>           <none>
sol-home-deploy-745b565968-rl9k7     	1/1        Running     0                 49m    10.244.2.4   k8s-worker2   <none>           <none>

[root@k8s-master curriculum]# kubectl  get  endpoints  curriculum-service
Warning: v1 Endpoints is deprecated in v1.33+; use discovery.k8s.io/v1 EndpointSlice
NAME                 	ENDPOINTS                      AGE
curriculum-service	10.244.1.5:80,10.244.2.5:80   2m30s
```

이 시점의 `ingress.yaml` 확인:

```
[root@k8s-master curriculum]# cat /root/webserver-demo/ingress/ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: sol-ingress
spec:
  ingressClassName: nginx
  rules:
    - http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: sol-home-service
                port:
                  number: 80
          - path: /curriculum
            pathType: Prefix
            backend:
              service:
                name: curriculum-service
                port:
                  number: 80
```

**정리**: 메인 페이지(sol-home-service)와 커리큘럼 페이지(curriculum-service)를 각각 별도 Deployment/Service로 만들고, 하나의 Ingress에서 `/`와 `/curriculum` 경로로 분기했다. 다음 절부터는 강좌/로그인 페이지를 같은 방식으로 추가한다.

---

## 강좌 페이지 만들기

```
	# 강좌 페이지 만들기
[root@k8s-master ~]#  mkdir -p  /root/webserver-demo/class/html/ban

	# 로그인 및 회원가입의 html 및 도커 파일을 위한 디렉터리 생성
[root@k8s-master ~]# mkdir -p  /root/webserver-demo/auth/html/login

	# Service YAML 파일 디렉터리 
[root@k8s-master ~]# mkdir -p  /root/webserver-demo/service

	# Deployment YAML 파일 디렉터리 
[root@k8s-master ~]# mkdir -p  /root/webserver-demo/deploy

[root@k8s-master ~]# ls -l  webserver-demo/
합계 0
drwxr-xr-x 3 root root 18   8월 24 13:10 auth
drwxr-xr-x 3 root root 18   8월 24 13:06 class
drwxr-xr-x 2 root root 42 12월 19  2025 curriculum
drwxr-xr-x 2 root root   6  8월 24 13:09 deploy
drwxr-xr-x 2 root root 70 12월 19  2025 ingress
drwxr-xr-x 2 root root   6  8월 24 13:11 service
drwxr-xr-x 3 root root 36 12월 19  2025 sol-collection

[root@k8s-master ~]# tree  /root/webserver-demo/

/root/webserver-demo/
├── auth
│        └── html
│                   └── login
├── class
│        └── html
│                   └── ban
├── curriculum
│        ├── Dockerfile
│        └── index.html
├── deploy
├── ingress
│        ├── curriculum.yaml
│        ├── ingress.yaml
│        └── sol-home.yaml
├── service
└── sol-collection
           ├── Dockerfile
           └── html
                       ├── images
                       │        ├── sol_logo.jpg
                       │        └── soldesk.jpg
                       └── index.html
```

서비스 단위로 소스/도커파일을 분리:

- `services/class`: 강좌 페이지 전용(정적 HTML)
- `services/auth`: 로그인/회원가입 전용(정적 HTML)
- `k8s`: 쿠버네티스 배포 매니페스트(Deployment/Service/Ingress)
- `konan7979/soldesk-main:1.0`: 메인 HTML만 포함
- `konan7979/soldesk-auth:1.0`: 로그인/회원가입 HTML만 포함

장점:

- 메인 화면만 수정해도 main 이미지만 다시 빌드/배포하면 됨(auth는 그대로)
- 트래픽/복제수(replica)를 서비스별로 다르게 운영 가능(메인은 2개, auth는 3개처럼)
- Ingress 경로 기반 라우팅( `/`는 main, `/login`은 auth )이 직관적으로 성립

### 강좌 페이지, 로그인 페이지, 회원가입 페이지 html 파일 생성

**강좌 페이지** (`webserver-demo/class/html/ban/index.html`):

```
[root@k8s-master ~]# vi ./webserver-demo/class/html/ban/index.html
```

```html
<!doctype html>
<html lang="ko">
<head>
  <meta charset="utf-8">
  <title>Soldesk Class</title>
</head>
<body>
  <h1>Soldesk Academy</h1>
  <p>강의 종류</p>
  <hr>

  <ul>
    <li><a href="/class/linux.html">리눅스</a></li>
    <li><a href="/class/network.html">네트워크</a></li>
    <li><a href="/class/docker.html">도커</a></li>
    <li><a href="/class/kubernetes.html">쿠버네티스</a></li>
    <li><a href="/class/ansible.html">앤서블</a></li>
    <li><a href="/class/terraform.html">테라폼</a></li>
    <li><a href="/class/aws.html">AWS</a></li>
    <li><a href="/class/elk.html">ELK</a></li>
    <li><a href="/class/project.html">통합 프로젝트</a></li>
  </ul>

  <hr>
  <button onclick="location.href='/'">메인으로</button>
</body>
</html>
```

```
[root@k8s-master ~]# ls  -l  ./webserver-demo/class/html/ban/
합계 4
-rw-r--r-- 1 root root 779  8월 24 16:36 index.html
```

### 강좌 페이지를 서비스하기 위한 Dockerfile 생성

```
[root@k8s-master ~]# cd  /root/webserver-demo/class/

[root@k8s-master class]# pwd
/root/webserver-demo/class

[root@k8s-master class]# ls  -lR
.:
합계 0
drwxr-xr-x 2 root root 24  8월 24 14:50 html

./html:
합계 4
-rw-r--r-- 1 root root 779  8월 24 14:48 index.html
```

```
   # Dockerfile 생성
[root@k8s-master class]# 	
```

```dockerfile
FROM  nginx:1.29.1
COPY  html/   /usr/share/nginx/html/
EXPOSE 80
```

```
[root@k8s-master class]# ls  -lR
.:
합계 4
-rw-r--r-- 1 root root 66  8월 24 14:53 Dockerfile
drwxr-xr-x 2 root root 24  8월 24 14:50 html

./html:
합계 4
-rw-r--r-- 1 root root 779  8월 24 14:48 index.html

   # 이미지 생성
[root@k8s-master class]# docker  build  -t  konan7979/soldesk-class:1.0  .

   # 이미지 hub.docker.com에 PUSH
[root@k8s-master class]# docker  push  konan7979/soldesk-class:1.0
The push refers to repository [docker.io/konan7979/soldesk-class]
1dd9c66811ae: Pushed
44136fa355b3: Mounted from konan7979/ssh-probe
375a694db734: Mounted from konan7979/sol-collection
5c32499ab806: Mounted from konan7979/sol-collectionektl 
16d05858bb8d: Mounted from konan7979/sol-collection
5f825f15e2e0: Mounted from konan7979/sol-collection
08cfef42fd24: Mounted from konan7979/sol-collection
3cc5fdd1317a: Mounted from konan7979/sol-collection
4f4e50e20765: Mounted from konan7979/sol-collection
dc7cb1c8cf65: Pushed
1.0: digest: sha256:d85e63c1753bfeefb177e3b975c4690b3573a63c49fddf2e569cfd4b23ac1f4b size: 856
```

### 강좌 페이지 이미지를 사용한 Deployment와 Service YAML 파일 생성

```
[root@k8s-master class]# vi  /root/webserver-demo/deploy/class-deploy.yaml
```

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: class-deploy
spec:
  replicas: 2
  selector:
    matchLabels:
      app: class
  template:
    metadata:
      labels:
        app: class
    spec:
      containers:
      - name: nginx
        image: konan7979/soldesk-class:1.0
        ports:
        - containerPort: 80
```

```
	# 강좌 페이지 이미지로 만든 pod들을 위한 service 생성

[root@k8s-master auth]# vi  /root/webserver-demo/deploy/class-svc.yaml
```

```yaml
apiVersion: v1
kind: Service
metadata:
  name: class-service
spec:
  selector:
    app: class
  ports:
  - port: 80
    targetPort: 80
```

```
[root@k8s-master class]# ls  -l  /root/webserver-demo/deploy/
합계 8
-rw-r--r-- 1 root root 327  8월 24 15:06 class-deploy.yaml
-rw-r--r-- 1 root root 135  8월 24 15:06 class-svc.yaml
```

### 강좌 페이지로 이동하기 위한 Path 설정

```
[root@k8s-master class]# vi /root/webserver-demo/ingress/ingress.yaml
```

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: sol-ingress
spec:
  ingressClassName: nginx
  rules:
    - http:
        paths:

          - path: /
            pathType: Prefix
            backend:
              service:
                name: sol-home-service
                port:
                  number: 80

          - path: /curriculum
            pathType: Prefix
            backend:
              service:
                name: curriculum-service
                port:
                  number: 80

          - path: /ban
            pathType: Prefix
            backend:
              service:
                name: class-service
                port:
                  number: 80
```

```
[root@k8s-master class]# ls -l  /root/webserver-demo/deploy/
합계 8
-rw-r--r-- 1 root root 327  8월 24 15:06 class-deploy.yaml		# 실행
-rw-r--r-- 1 root root 135  8월 24 15:06 class-svc.yaml		# 실행

[root@k8s-master class]# ls -l  /root/webserver-demo/ingress/
합계 12
-rw-r--r-- 1 root root 528 12월 19  2025 curriculum.yaml
-rw-r--r-- 1 root root 523 12월 19  2025 ingress.yaml		# 실행
-rw-r--r-- 1 root root 514 12월 19  2025 sol-home.yaml

[root@k8s-master class]# kubectl  apply  -f  /root/webserver-demo/deploy/class-deploy.yaml

[root@k8s-master class]# kubectl  apply  -f  /root/webserver-demo/deploy/class-svc.yaml

[root@k8s-master class]# kubectl  apply  -f  /root/webserver-demo/ingress/ingress.yaml

[root@k8s-master class]# kubectl  get  deployments  class-deploy
NAME           READY   UP-TO-DATE   AVAILABLE   AGE
class-deploy   2/2         2                    2                 70s

[root@k8s-master class]# kubectl  get  pods
NAME                                 	READY   STATUS    RESTARTS   AGE
class-deploy-5567486569-b62q5        	1/1     Running   0          5m36s
class-deploy-5567486569-qp6lf        	1/1     Running   0          5m36s
curriculum-deploy-587bbcd4c5-5vct9   	1/1     Running   0          161m
curriculum-deploy-587bbcd4c5-fqqsm   	1/1     Running   0          161m
sol-home-deploy-745b565968-rl9k7     	1/1     Running   0          3h27m

[root@k8s-master class]# kubectl  get  service  class-service
NAME            TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
class-service   ClusterIP   10.97.151.80   <none>        80/TCP    76s

[root@k8s-master class]# kubectl  get  ingress  sol-ingress
NAME          CLASS   HOSTS   ADDRESS          PORTS   AGE
sol-ingress   nginx   *       192.168.10.101   80      3h18m

[root@k8s-master class]# kubectl  describe  ingress  sol-ingress
Name:       	sol-ingress
Labels:           	<none>
Namespace:        	default
Address:          	192.168.10.101
Ingress Class:    	nginx
Default backend:  	<default>
Rules:
  Host		Path  	Backends
  ----  		----  	--------
  *
         		/             	sol-home-service:80 (10.244.2.4:80)
          		/curriculum	curriculum-service:80 (10.244.1.5:80,10.244.2.5:80)
         		/ban        	class-service:80 (10.244.1.6:80,10.244.2.6:80)
Annotations:  	<none>
Events:
  Type    Reason  Age                    	  From                      	Message
  ----    ------  ----                   	  ----                      	-------
  Normal  Sync    2m30s (x3 over 3h19m)  nginx-ingress-controller  	Scheduled for sync
```

현재 `sol-collection/html/index.html` 코드에는 강좌 페이지로 이동할 방법이 없다.
강좌 페이지로 이동할 코드를 생성해야 이동이 가능하다.

```
	# 강좌로 이동하기위해서  sol-collection안의 html 코드 수정
[root@k8s-master class]# ls -l  /root/webserver-demo/sol-collection/html/
합계 4
drwxr-xr-x 2 root root  45 12월 19  2025 images
-rw-r--r-- 1 root root 341  1월 13  2026 index.html
```

### 강좌 페이지로 이동하기 위해서 html 코드 수정

```
[root@k8s-master class]# vi  /root/webserver-demo/sol-collection/html/index.html
```

```html
<html>
<head>
  <meta charset="UTF-8">
  <title>soldesk</title>
</head>
<body>
  <center>
    <img src="images/sol_logo.jpg"><br>
    <p style="color:red;">Soldesk Academy/AWS Cloud</p><br>
    <img src="images/soldesk.jpg"><br>
    <button onclick="location.href='/curriculum/index.html'">커리큘럼</button>
    <button onclick="location.href='/ban/index.html'">강좌 페이지</button>		# 버튼추가
  </center>
</body>
</html>
```

### 변경사항을 반영하기 위한 롤링 업데이트 (sol-collection:1.0 → 1.1)

```
[root@k8s-master class]# cd  /root/webserver-demo/sol-collection/

[root@k8s-master sol-collection]# pwd
/root/webserver-demo/sol-collection

[root@k8s-master sol-collection]# ls  -l
합계 4
-rw-r--r-- 1 root root 151 12월 19  2025 Dockerfile
drwxr-xr-x 3 root root 38   8월 24 16:21 html

    # sol-collection:1.0  -->  sol-collection:1.1 로 버전업
[root@k8s-master sol-collection]# docker  build  -t  konan7979/sol-collection:1.1  .

[root@k8s-master sol-collection]# docker push konan7979/sol-collection:1.1
The push refers to repository [docker.io/konan7979/sol-collection]
762b6005bb6c: Pushed
44136fa355b3: Already exists
375a694db734: Layer already exists
5c32499ab806: Layer already exists
5f825f15e2e0: Layer already exists
16d05858bb8d: Layer already exists
08cfef42fd24: Layer already exists
3cc5fdd1317a: Layer already exists
4f4e50e20765: Layer already exists
9a6d0ff9649d: Pushed
1.1: digest: sha256:50e9203d47063d65fb5e293c35d665f891aa3293ba52c71d5ef3f7ea7174d366 size: 856

	# 롤링 업데이트
[root@k8s-master sol-collection]# kubectl  set  image  deployments sol-home-deploy  nginx=konan7979/sol-collection:1.1
deployment.apps/sol-home-deploy image updated

	# 버전 관리
[root@k8s-master sol-collection]# kubectl annotate deployment  sol-home-deploy  \
> kubernetes.io/change-cause="rev2: sol-collection:1.0 -> sol-collection:1.1" --overwrite
deployment.apps/sol-home-deploy annotated

[root@k8s-master sol-collection]# kubectl  rollout  history  deployment  sol-home-deploy
deployment.apps/sol-home-deploy
REVISION  CHANGE-CAUSE
1         	  <none>
2              rev2: sol-collection:1.0 -> sol-collection:1.1
```

**정리**: 새 서비스(class-service)를 추가할 때는 이미지 빌드 → Deployment/Service 생성 → Ingress에 경로 추가 → 기존 페이지에서 이동 링크 추가 → **롤링 업데이트**로 반영하는 흐름을 따른다. 로그인/회원가입 페이지도 동일한 흐름으로 추가한다.

---

## 로그인 / 회원가입 페이지 (auth)

### 로그인 페이지

```
[root@k8s-master ~]# vi  ./webserver-demo/auth/html/login/login.html
```

```html
<!doctype html>
<html lang="ko">
<head>
  <title>Login</title>
  <meta charset="utf-8">
</head>
<body>
  <h1>로그인 페이지</h1>

  <form>
    ID: <input type="text"><br><br>
    PW: <input type="password"><br><br>
    <input type="submit" value="로그인">
  </form>

  <hr>
  <a href="/">메인으로</a>
</body>
</html>
```

### 회원 가입 페이지

```
[root@k8s-master ~]# vi  ./webserver-demo/auth/html/login/signup.html
```

```html
<!doctype html>
<html lang="ko">
<head>
  <meta charset="utf-8">
  <title>Signup</title>
</head>
<body>
  <h1>회원가입 페이지</h1>

  <form>
    ID: <input type="text"><br><br>
    PW: <input type="password"><br><br>
    Email: <input type="email"><br><br>
    <input type="submit" value="회원가입">
  </form>

  <hr>
  <a href="/">메인으로</a>
</body>
</html>
```

```
[root@k8s-master class]# ls  -l  /webserver-demo/auth/html/login
합계 8
-rw-r--r-- 1 root root 329  1월 13 23:28 login.html
-rw-r--r-- 1 root root 376  1월 13 23:28 signup.html
```

### 로그인, 회원가입용 이미지 생성

```
[root@k8s-master class]# cd   /root/webserver-demo/auth/

[root@k8s-master auth]# pwd
/root/webserver-demo/auth

[root@k8s-master soldesk-ingress]# vi Dockerfile
```

```dockerfile
FROM nginx:1.29.1
COPY html/ /usr/share/nginx/html/
EXPOSE 80
```

```
[root@k8s-master auth]# ls -lR
.:
합계 4
-rw-r--r-- 1 root root 62  8월 24 17:12 Dockerfile
drwxr-xr-x 3 root root 19  8월 24 13:10 html

./html:
합계 0
drwxr-xr-x 2 root root 43  8월 24 14:39 login

./html/login:
합계 8
-rw-r--r-- 1 root root 329  8월 24 14:38 login.html
-rw-r--r-- 1 root root 376  8월 24 14:39 signup.html
```

### 회원가입, 로그인을 서비스하기 위한 이미지 생성

```
[root@k8s-master auth]# docker  build  -t  konan7979/soldesk-auth:1.0 .

[root@k8s-master auth]# docker images
IMAGE                              		ID             	DISK USAGE   CONTENT SIZE   EXTRA
konan7979/curriculum-service:1.0   	2a2b49481bba        	276MB           72.3MB
konan7979/sol-collection:1.0       	571a4a896a75        	276MB           72.3MB
konan7979/sol-collection:1.1       	50e9203d4706        	276MB           72.3MB
konan7979/soldesk-auth:1.0         	85d142e226fd        	276MB           72.3MB
konan7979/soldesk-class:1.0        	4c3e945db06d        	276MB           72.3MB
konan7979/soldesk-class:1.1        	3191088fda2f       	276MB           72.3MB

[root@k8s-master auth]# docker push konan7979/soldesk-auth:1.0
```

### 회원가입, 로그인을 서비스하기 위한 Deployment 만들기

```
[root@k8s-master auth]# vi  /root/webserver-demo/deploy/auth-deploy.yaml
```

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: auth-deploy
spec:
  replicas: 3
  selector:
    matchLabels:
      app: auth
  template:
    metadata:
      labels:
        app: auth
    spec:
      containers:
      - name: nginx
        image: konan7979/soldesk-auth:1.0
        ports:
        - containerPort: 80
```

```
[root@k8s-master auth]# ls  -l   /root/webserver-demo/deploy/
합계 12
-rw-r--r-- 1 root root 323  8월 24 17:20 auth-deploy.yaml
-rw-r--r-- 1 root root 327  8월 24 17:05 class-deploy.yaml
-rw-r--r-- 1 root root 135  8월 24 15:06 class-svc.yaml
```

### 회원가입, 로그인 이미지로 만든 pod들을 위한 service 생성

```
[root@k8s-master auth]# vi  /root/webserver-demo/service/auth-svc.yaml
```

```yaml
apiVersion: v1
kind: Service
metadata:
  name: auth-service
spec:
  selector:
    app: auth
  ports:
  - port: 80
    targetPort: 80
```

```
[root@k8s-master auth]# ls  -l  /root/webserver-demo/service/
합계 8
-rw-r--r-- 1 root root 133  8월 24 17:22 auth-svc.yaml
-rw-r--r-- 1 root root 135  8월 24 17:21 class-svc.yaml
```

### ingress.yaml에 /login 경로 추가

```
[root@k8s-master auth]# vi  /root/webserver-demo/ingress/ingress.yaml
```

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: sol-ingress
spec:
  ingressClassName: nginx
  rules:
    - http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: sol-home-service
                port:
                  number: 80
          - path: /curriculum
            pathType: Prefix
            backend:
              service:
                name: curriculum-service
                port:
                  number: 80

          - path: /ban
            pathType: Prefix
            backend:
              service:
                name: class-service
                port:
                  number: 80

          - path: /login
            pathType: Prefix
            backend:
              service:
                name: auth-service
                port:
                  number: 80
```

### 배포 실행 및 확인

```
    # 로그인, 회원가입 Deployment 실행
[root@k8s-master auth]# kubectl  apply  -f  /root/webserver-demo/deploy/auth-deploy.yaml
deployment.apps/auth-deploy created

    # 로그인, 회원가입 Service 실행
[root@k8s-master auth]# kubectl  apply  -f  /root/webserver-demo/service/auth-svc.yaml
service/auth-service created

   # ingress적용을 위한 재시작
[root@k8s-master auth]# kubectl  apply  -f  /root/webserver-demo/ingress/ingress.yaml
ingress.networking.k8s.io/sol-ingress configured

[root@k8s-master auth]# kubectl  get  deployments  auth-deploy
NAME          READY   UP-TO-DATE   AVAILABLE   AGE
auth-deploy   3/3     3            3           73s

[root@k8s-master auth]# kubectl  get  pods
NAME                                 READY   STATUS    RESTARTS   AGE
auth-deploy-555dd97c97-2d57n         1/1     Running   0          88s
auth-deploy-555dd97c97-f5cdv         1/1     Running   0          88s
auth-deploy-555dd97c97-vz62h         1/1     Running   0          88s
class-deploy-84cc5b8786-bsw9s        1/1     Running   0          21m
class-deploy-84cc5b8786-scrlm        1/1     Running   0          21m
curriculum-deploy-587bbcd4c5-5vct9   1/1     Running   0          4h48m
curriculum-deploy-587bbcd4c5-fqqsm   1/1     Running   0          4h48m
sol-home-deploy-66bfd9c66c-q6cbx     1/1     Running   0          57m

[root@k8s-master auth]# kubectl  get  service
NAME                 TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
auth-service         ClusterIP   10.107.138.137   <none>        80/TCP    71s
class-service        ClusterIP   10.97.151.80     <none>        80/TCP    132m
curriculum-service   ClusterIP   10.110.9.135     <none>        80/TCP    4h48m
kubernetes           ClusterIP   10.96.0.1        <none>        443/TCP   13d
sol-home-service     ClusterIP   10.102.49.106    <none>        80/TCP    68m

	# 외부포트 확인
[root@k8s-master auth]# kubectl  get  svc  -A | grep ingress
ingress-nginx   ingress-nginx-controller             	   NodePort    10.105.3.121     <none>        80:32181/TCP,443:30366/TCP   6h49m
ingress-nginx   ingress-nginx-controller-admission   ClusterIP    10.101.171.143   <none>        443/TCP                      6h49m

[root@k8s-master auth]# kubectl  describe  ingress
Name:       	sol-ingress
Labels:        	<none>
Namespace:        	default
Address:          	92.168.10.101
Ingress Class:    	nginx
Default backend:  	<default>
Rules:
  Host        Path  Backends
  ----        ----  --------
  *
            	/             	sol-home-service:80 (10.244.1.8:80)
           	/curriculum   	curriculum-service:80 (10.244.1.5:80,10.244.2.5:80)
            	/ban          	class-service:80 (10.244.1.12:80,10.244.2.12:80)
           	/login        	auth-service:80 (10.244.2.13:80,10.244.1.13:80,10.244.2.14:80)
Annotations:  <none>
Events:
  Type    Reason  Age                    From                      Message
  ----    ------  ----                   ----                      -------
  Normal  Sync    4m43s (x5 over 5h32m)  nginx-ingress-controller  Scheduled for sync

[root@k8s-master auth]# kubectl  get  pods  -o  wide
NAME                                 	READY   STATUS    RESTARTS   AGE     IP            NODE          NOMINATED NODE   READINESS GATES
auth-deploy-555dd97c97-2d57n         	1/1     Running   0          6m38s   10.244.2.13   k8s-worker2   <none>           <none>
auth-deploy-555dd97c97-f5cdv         	1/1     Running   0          6m38s   10.244.1.13   k8s-worker1   <none>           <none>
auth-deploy-555dd97c97-vz62h         	1/1     Running   0          6m38s   10.244.2.14   k8s-worker2   <none>           <none>
class-deploy-84cc5b8786-bsw9s        	1/1     Running   0          27m     10.244.2.12   k8s-worker2   <none>           <none>
class-deploy-84cc5b8786-scrlm        	1/1     Running   0          27m     10.244.1.12   k8s-worker1   <none>           <none>
curriculum-deploy-587bbcd4c5-5vct9 	1/1     Running   0          4h53m   10.244.2.5    k8s-worker2   <none>           <none>
curriculum-deploy-587bbcd4c5-fqqsm   	1/1     Running   0          4h53m   10.244.1.5    k8s-worker1   <none>           <none>
sol-home-deploy-66bfd9c66c-q6cbx     	1/1     Running   0          62m     10.244.1.8    k8s-worker1   <none>           <none>
```

로그인, 회원가입이 추가되었으나 main 페이지에서 로그인, 회원가입으로 이동할 기능이 없다.
이동할 기능을 추가하기 위해서 html코드를 수정 후 롤링 업데이트 실시.

### main 페이지에 로그인/회원가입 버튼 추가

```
[root@k8s-master ingress]# vi  /root/webserver-demo/sol-collection/html/index.html
```

```html
<html>
<head>
  <meta charset="UTF-8">
  <title>soldesk</title>
</head>
<body>
  <center>
    <img src="images/sol_logo.jpg"><br>
    <p style="color:red;">Soldesk Academy/AWS Cloud</p><br>
    <img src="images/soldesk.jpg"><br>

    <button onclick="location.href='/curriculum/index.html'">커리큘럼</button>

    <br><br>							# 추가 설정
    <button onclick="location.href='/login/login.html'">로그인</button>		# 추가 설정
    <button onclick="location.href='/login/signup.html'">회원가입</button>	# 추가 설정
  </center>

</body>
</html>
```

### 변경사항을 반영하기 위한 롤링 업데이트 (sol-collection:1.1 → 1.2)

```
	# 변경사항을 반영하기위한 롤링 업데이트
[root@k8s-master ingress]# cd  /root/webserver-demo/sol-collection

[root@k8s-master sol-collection]# ls  -l
합계 4
-rw-r--r-- 1 root root 151 12월 19  2025 Dockerfile
drwxr-xr-x 3 root root  38  8월 24 17:37 html

	# main 페이지 이미지 생성
[root@k8s-master sol-collection]# docker  build -t  konan7979/sol-collection:1.2  .

	# main 페이지 이미지 PUSH
[root@k8s-master sol-collection]# docker  push  konan7979/sol-collection:1.2

	# 롤링 업데이트
[root@k8s-master sol-collection]# kubectl  set  image  deployments  sol-home-deploy  nginx=konan7979/sol-collection:1.2
deployment.apps/sol-home`-deploy image updated

	# 롤링 업데이트 버전 관리
[root@k8s-master sol-collection]# kubectl annotate deployment  sol-home-deploy  \
kubernetes.io/change-cause="rev3: sol-collection:1.1 -> sol-collection:1.2" --overwrite
deployment.apps/sol-home-deploy annotated

	# 버전 확인
[root@k8s-master sol-collection]# kubectl  rollout  history  deployment  sol-home-deploy
deployment.apps/sol-home-deploy
REVISION  CHANGE-CAUSE
1              <none>
2              rev2: sol-collection:1.0 -> sol-collection:1.1
3              rev3: sol-collection:1.1 -> sol-collection:1.2
```

ingress.yaml에 `/login` 경로 주석 추가:

```
[root@k8s-master auth]# vi  /root/webserver-demo/ingress/ingress.yaml
```

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: sol-ingress
spec:
  ingressClassName: nginx
  rules:
    - http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: sol-home-service
                port:
                  number: 80
          - path: /curriculum
            pathType: Prefix
            backend:
              service:
                name: curriculum-service
                port:
                  number: 80

          - path: /ban
            pathType: Prefix
            backend:
              service:
                name: class-service
                port:
                  number: 80

          - path: /login		# /login/login.html
            pathType: Prefix
            backend:
              service:
                name: auth-service
                port:
                  number: 80
```

접속 확인:

```
https://192.168.10.100:30366/login/login.html
```

**정리**: 로그인/회원가입 페이지도 강좌 페이지와 동일한 패턴(이미지 빌드 → Deployment/Service → Ingress 경로 추가)으로 추가했다. 다만 `pathType: Prefix` 방식은 실제 파일명까지 URL에 그대로 붙어야 하는 한계가 있어, 다음 절에서는 이를 정규표현식 방식으로 개선한다.

---

## 정규표현식 기반 Ingress (ingress-html.yaml)

`path: /login`처럼 Prefix 방식이면 `/login/login.html`, `/login/signup.html`처럼 실제 파일명까지 URL에 붙여야 한다.
이를 개선하기 위해 정규표현식과 `rewrite-target`을 사용하는 방식으로 전환한다.

```
[root@k8s-master ~]# vi /root/webserver-demo/ingress/ingress-html.yaml
```

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress

metadata:
  name: sol-ingress-html
  annotations:
    nginx.ingress.kubernetes.io/use-regex: "true"    	# path에서 정규표현식 사용
    nginx.ingress.kubernetes.io/rewrite-target: /$2		# 두 번째 그룹($2)의 경로만 Backend로 전달

spec:
  ingressClassName: nginx                                 # nginx Ingress Controller 사용

  rules:
  - http:
      paths:

      - path: /curriculum(/|$)(.*)                       	# /curriculum 및 하위 경로 매칭
        pathType: ImplementationSpecific         	# Ingress Controller 방식으로 경로 해석
        backend:
          service:
            name: curriculum-service              	# curriculum-service로 전달
            port:
              number: 80                            	# Service의 80번 포트 사용

      - path: /login(/|$)(.*)                      	# /login 및 하위 경로 매칭
        pathType: ImplementationSpecific
        backend:
          service:
            name: auth-service                        	# auth-service로 전달
            port:
              number: 80

      - path: /class(/|$)(.*)                        	# /class 및 하위 경로 매칭
        pathType: ImplementationSpecific
        backend:
          service:
            name: class-service                      	# class-service로 전달
            port:
              number: 80

      - path: /()(.*)                                  	# 위 경로에 해당하지 않는 요청은 메인 서비스로 전달
        pathType: ImplementationSpecific
        backend:
          service:
            name: sol-home-service                  	# sol-home-service로 전달
            port:
              number: 80
```

### 정규표현식 경로 문법 설명

Ingress 정규표현식 경로: `/class(/|$)(.*)` — 무엇을 매칭할지를 작성한다.

`(/|$)`
- `/`: 슬래시(/)
- `$`: 문자열의 끝
- 즉, `/class` 뒤에 슬래시(/)가 오거나 주소가 끝나는 경우를 허용한다.
  - `/class`
  - `/class/`
  - `/class/index.html/`

`(.*)`
- `.`: 아무 문자 1개
- `*`: 앞의 문자가 0개 이상
- `.*`: 아무 문자가 0개 이상
- `(.*)`: 뒤에 오는 모든 문자열을 하나의 그룹으로 묶는다.

### rewrite-target: /$2

`rewrite-target: /$2`는 매칭된 것 중 무엇을 보낼지(치환) 작성한다.

**예1)**
- 요청 = `/curriculum`
- `$2` = `""` (빈 문자열)
- rewrite 결과 = `/`
- 백엔드 nginx: `/usr/share/nginx/html/index.html`

**예2)**
- 요청 = `/curriculum/index.html`
- `$2` = `index.html`
- rewrite 결과 = `/index.html`
- 백엔드 nginx: `/usr/share/nginx/html/index.html`

**예3)**
- 요청 = `/curriculum/css/style.css`
- `$2` = `css/style.css`
- rewrite 결과 = `/css/style.css`
- 백엔드 nginx: `/usr/share/nginx/html/css/style.css`

최종 `ingress-html.yaml`:

```
[root@k8s-master ~]# vi /root/webserver-demo/ingress/ingress-html.yaml
```

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: sol-ingress-html
  annotations:
    nginx.ingress.kubernetes.io/use-regex: "true"
    nginx.ingress.kubernetes.io/rewrite-target: /$2

spec:
  ingressClassName: nginx

  rules:
  - http:
      paths:

      - path: /curriculum(/|$)(.*)
        pathType: ImplementationSpecific
        backend:
          service:
            name: curriculum-service
            port:
              number: 80

      - path: /login(/|$)(.*)
        pathType: ImplementationSpecific
        backend:
          service:
            name: auth-service
            port:
              number: 80

      - path: /class(/|$)(.*)
        pathType: ImplementationSpecific
        backend:
          service:
            name: class-service
            port:
              number: 80

      - path: /()(.*)
        pathType: ImplementationSpecific
        backend:
          service:
            name: sol-home-service
            port:
              number: 80
```

이 시점의 전체 디렉터리 구조:

```
[root@k8s-master ~]# tree  /root/webserver-demo/
/root/webserver-demo/
├── auth
│       ├── Dockerfile
│       └── html
│                 └── login
│                            ├── login.html
│                            └── signup.html
├── class
│       ├── Dockerfile
│       └── html
│                  └── ban
│                            └── index.html
├── curriculum
│       ├── Dockerfile
│       └── index.html
├── deploy
│       ├── auth-deploy.yaml
│       ├── class-deploy.yaml
├── ingress
│       ├── curriculum.yaml
│       ├── ingress-html.yaml
│       ├── ingress.yaml
│       └── sol-home.yaml
├── service
│       ├── auth-svc.yaml
│       └── class-svc.yaml
└── sol-collection
          ├── Dockerfile
          └── html
                     ├── images
                     │   ├── sol_logo.jpg
                     │   └── soldesk.jpg
                     └── index.html
```

### 로그인, 회원가입을 새로운 ingress에 맞게 경로 수정

정규표현식 방식은 `/login/(.*)`이 곧바로 백엔드의 `/`로 치환되므로, auth 컨테이너 안의 html 파일 경로도 `login/` 하위 디렉터리가 아니라 루트로 옮겨야 한다.

```
[root@k8s-master ~]# cd  /root/webserver-demo/auth/

[root@k8s-master auth]# ls -l
합계 4
-rw-r--r-- 1 root root 62  8월 24 17:12 Dockerfile
drwxr-xr-x 3 root root 19  8월 24 13:10 html

[root@k8s-master auth]# cp ./html/login/login.html   ./html/
[root@k8s-master auth]# cp ./html/login/signup.html ./html/

[root@k8s-master auth]# ls -l ./html/
합계 8
drwxr-xr-x 2 root root  43  8월 24 14:39 login
-rw-r--r-- 1 root root 329  8월 25 10:19 login.html
-rw-r--r-- 1 root root 376  8월 25 10:20 signup.html

[root@k8s-master auth]# rm  -rf ./html/login

[root@k8s-master auth]# ls -l ./html/
합계 8
-rw-r--r-- 1 root root 329  8월 25 10:19 login.html
-rw-r--r-- 1 root root 376  8월 25 10:20 signup.html
```

### 강좌 페이지를 새로운 ingress에 맞게 경로 수정

```
[root@k8s-master auth]# cd /root/webserver-demo/class/

[root@k8s-master class]# ls -l
합계 4
-rw-r--r-- 1 root root 66  8월 24 14:53 Dockerfile
drwxr-xr-x 3 root root 17  8월 24 15:58 html

[root@k8s-master class]# cp ./html/ban/index.html   ./html/

[root@k8s-master class]# rm  -rf  ./html/ban/

[root@k8s-master class]# ls -l  ./html/
합계 4
-rw-r--r-- 1 root root 779  8월 25 10:22 index.html
```

```
[root@k8s-master ~]# kubectl  get  pods
NAME                                 	READY   STATUS    RESTARTS   AGE
auth-deploy-555dd97c97-cgp25         	1/1         Running     0                109s		# 로그인 및 회원 가입 Pod
auth-deploy-555dd97c97-zhncw         	1/1         Running     0                109s		# 로그인 및 회원 가입 Pod
auth-deploy-555dd97c97-zjbzp         	1/1         Running     0                109s		# 로그인 및 회원 가입 Pod
class-deploy-84cc5b8786-4rhwv        	1/1         Running     0                100s		# 강좌 Pod
class-deploy-84cc5b8786-b4xsx        	1/1         Running     0                100s		# 강좌 Pod
curriculum-deploy-587bbcd4c5-4gnhd   	1/1         Running     0                2m21s	# 커리큘럼 Pod
curriculum-deploy-587bbcd4c5-gz9cs   	1/1         Running     0                2m21s	# 커리큘럼 Pod
sol-home-deploy-745b565968-b67vc     	1/1         Running     0                2m25s	# 메인 페이지 Pod
```

### 롤링 업데이트전 로그인 및 회원가입 pod의 경로를 확인

```
[root@k8s-master ~]# kubectl  exec  -it  auth-deploy-555dd97c97-cgp25  -- /bin/bash
root@auth-deploy-555dd97c97-cgp25:/#

root@auth-deploy-555dd97c97-cgp25:/# ls  -l  /usr/share/nginx/html/
total 8
-rw-r--r-- 1 root root 497 Aug 13  2025 50x.html
-rw-r--r-- 1 root root 615 Aug 13  2025 index.html
drwxr-xr-x 2 root root  43 Aug 24 05:39 login		# login 디렉터리 확인

root@auth-deploy-555dd97c97-cgp25:/# ls  -l  /usr/share/nginx/html/login/
total 8	
-rw-r--r-- 1 root root 329 Aug 24 05:38 login.html		# 로그인 페이지
-rw-r--r-- 1 root root 376 Aug 24 05:39 signup.html	# 회원 가입 페이지
```

### 로그인, 회원가입 Pod 롤링 업데이트 (soldesk-auth:1.0 → 1.1)

```
[root@k8s-master ~]# cd  /root/webserver-demo/auth/

[root@k8s-master auth]# ls  -l
합계 4
-rw-r--r-- 1 root root 62  8월 24 17:12 Dockerfile
drwxr-xr-x 2 root root 43  8월 25 10:20 html

	# 로그인, 회원가입 이미지 생성
[root@k8s-master auth]# docker  build  -t  konan7979/soldesk-auth:1.1  .

	# 로그인, 회원가입 이미지 확인
[root@k8s-master auth]# docker  images
IMAGE                              		ID             	DISK USAGE   CONTENT SIZE   EXTRA
konan7979/curriculum-service:1.0 	2a2b49481bba        276MB         72.3MB
konan7979/sol-collection:1.0       	571a4a896a75        276MB         72.3MB
konan7979/sol-collection:1.1       	50e9203d4706        276MB         72.3MB
konan7979/sol-collection:1.2       	6f9c8602891e        276MB         72.3MB
konan7979/soldesk-auth:1.0         	85d142e226fd        276MB         72.3MB
konan7979/soldesk-auth:1.1         	c2d9d04b0a87        276MB         72.3MB
konan7979/soldesk-class:1.0        	4c3e945db06d        276MB         72.3MB
konan7979/soldesk-class:1.1        	3191088fda2f        276MB         72.3MB

	# 로그인, 회원가입 이미지 PUSH
[root@k8s-master auth]# docker  push  konan7979/soldesk-auth:1.1
The push refers to repository [docker.io/konan7979/soldesk-auth]
077cf53213bf: Pushed
44136fa355b3: Already exists
375a694db734: Layer already exists
5c32499ab806: Layer already exists
5f825f15e2e0: Layer already exists
16d05858bb8d: Layer already exists
08cfef42fd24: Layer already exists
4f4e50e20765: Layer already exists
3cc5fdd1317a: Layer already exists
3c388c881ddd: Pushed
1.1: digest: sha256:c2d9d04b0a870716404c1da825c46a99735c033c9144389940f2551d0b05bc64 size: 856

	# 로그인, 회원가입 Deployment 롤링 업데이트
[root@k8s-master auth]# kubectl  set  image  deployments  auth-deploy  nginx=konan7979/soldesk-auth:1.1
deployment.apps/auth-deploy image updated

	# 버전 관리
[root@k8s-master auth]# kubectl annotate deployment  auth-deploy  \
 kubernetes.io/change-cause="rev2: soldesk-auth:1.0 -> soldesk-auth:1.1 Version Update" --overwrite
deployment.apps/auth-deploy annotated

[root@k8s-master auth]# kubectl  rollout  history  deployment  auth-deploy
deployment.apps/auth-deploy
REVISION  CHANGE-CAUSE
1         <none>
2         rev2: soldesk-auth:1.0 -> soldesk-auth:1.1 Version Update

	# 새로 만든 Ingerss 실행
[root@k8s-master auth]# kubectl  apply  -f  /root/webserver-demo/ingress/ingress-html.yaml
ingress.networking.k8s.io/sol-ingress-html created
```

접속:

```
https://192.168.10.100:30366/
```

**정리**: 정규표현식(`use-regex`)과 `rewrite-target`을 이용하면 하위 경로를 그대로 백엔드에 전달할 수 있어, `/login`, `/class`, `/curriculum` 등 여러 서비스를 하나의 Ingress로 깔끔하게 묶을 수 있다. 이제부터는 배포 전략인 Canary Deployment를 다룬다.

---

## Canary Deployment

**Canary Deployment**(카나리 배포)는 새로운 버전을 전체 사용자에게 한 번에 적용하지 않고, 일부 Pod 또는 일부 사용자에게만 먼저 적용하여 안정성을 확인한 뒤 점진적으로 확대하는 배포 방식이다.

1. 기존 버전 전체 운영
2. 신규 버전 소수만 배포
3. 문제 없는지 확인
4. 신규 버전 비율 확대
5. 최종적으로 전체 신규 버전 전환

### 왜 Canary라고 부르는가?

과거 광산에서는 유독가스를 감지하기 위해 카나리아 새를 먼저 광산 안으로 데리고 들어갔다.
카나리아가 이상 반응을 보이면 사람이 위험을 미리 알 수 있었다.
소프트웨어 배포도 이와 비슷하게, 신규 버전을 일부에 먼저 적용 후 문제가 있는지 확인하고 문제가 없으면 전체 적용 하기 때문에 Canary Deployment라고 부른다.

### 기존 배포 방식의 문제점

예를 들어 현재 웹 서비스가 1.0 버전으로 운영되고 있다고 가정한다.

현재 운영:

```
web:1.0
web:1.0
web:1.0
```

새로운 2.0 버전을 바로 전체 배포하면:

```
web:2.0
web:2.0
web:2.0
```

만약 2.0에 치명적인 오류가 있다면 전체 사용자에게 동시에 장애가 발생할 수 있다.

Canary Deployment는 이를 방지하기 위해 일부만 먼저 변경한다.

```
web:1.0
web:1.0
web:1.0
web:2.0   <-- Canary
```

이 상태에서 2.0의 동작을 확인한다.

### Canary Deployment의 기본 구조

예를 들어 총 10개의 Pod가 운영 중이라고 가정한다.
- 기존 버전: v1 v1 v1 v1 v1 v1 v1 v1 v1 v1

새 버전 v2를 1개만 먼저 배포한다.
- 새 버전: v1 v1 v1 v1 v1 v1 v1 v1 v1 v2(Canary)

대략 다음과 같은 비율이 된다.

- Canary v1: 90%
- Canary v2: 10%

문제가 없다면 점차 확대한다.

- 1단계: Canary v1 90% / Canary v2 10%
- 2단계: Canary v1 70% / Canary v2 30%
- 3단계: Canary v1 50% / Canary v2 50%
- 최종: Canary v1 0% / Canary v2 100%

### Kubernetes에서 Canary Deployment를 만드는 기본 원리

K8s에서는 일반적으로 서로 다른 버전의 Deployment를 동시에 실행하여 Canary Deployment를 구현할 수 있다.

- Deployment 1: `web-v1`, replicas: 9, image: `web:1.0`
- Deployment 2: `web-v2`, replicas: 1, image: `web:2.0`

두 Deployment의 Pod가 동일한 Service에 포함되도록 Label을 구성한다.

- Deployment 1의 web-v1 Pod: `app=web`, `version=v1`
- Deployment 2의 web-v2 Pod: `app=web`, `version=v2`

Service는 다음 Label만 선택한다.

```yaml
selector:
  app: web
```

그러면 Service 입장에서는 `app=web, version=v1` (×3), `app=web, version=v2` 모두 자신의 Backend Pod가 된다.

### Service와 Canary Deployment 관계

Canary Deployment에서 중요한 부분은 Service Selector이다.

예)

- 기존 Pod: `app=web`, `version=v1`
- Canary Pod: `app=web`, `version=v2`

Service가 다음과 같이 설정되어 있다면:

```yaml
selector:
  app: web
```

version은 확인하지 않기 때문에 v1과 v2 모두 Service에 포함된다.

구조:

```
                         Service
                            |
                  selector: app=web
                            |
           +-----------+------------+
           |                |                 |
       v1 Pod        v1 Pod           v2 Pod
```

따라서 사용자 요청이 기존 버전과 신규 버전으로 분산될 수 있다.

| 구분 | Rolling Update | Canary Deployment |
|------|----------------|--------------------|
| 목적 | 순차적으로 새 버전 교체 | 새 버전을 일부에 먼저 테스트 |
| 기존/신규 버전 동시 운영 | 일시적으로 존재 | 의도적으로 일정 기간 함께 운영 |
| 트래픽 검증 | 주 목적이 아님 | 핵심 목적 |
| 장애 영향 최소화 | 가능 | 더욱 세밀하게 가능 |
| 신규 버전 테스트 | 제한적 | 실제 트래픽으로 검증 가능 |

**정리**: Canary Deployment는 같은 `app` 라벨을 공유하는 서로 다른 버전의 Deployment(v1/v2)를 두고, Service의 selector가 version을 구분하지 않게 하여 일부 트래픽만 신규 버전으로 흘려보내는 방식이다. 아래에서는 이를 직접 구성해본다.

---

## Canary Deployment 실습

- 기존 버전 v1과 신규 버전 v2 Deployment를 각각 생성한다.
- v1과 v2는 서로 다른 HTML 화면을 사용한다.
- 두 Deployment는 version 라벨로 서로 구분한다.
- Service는 공통 라벨 `app=web`만 선택하여 v1과 v2를 모두 서비스에 포함한다.
- 반복 요청을 통해 기존 버전과 Canary 버전으로 요청이 분산되는 것을 확인한다.

다음 조건으로 Canary Deployment 환경을 구성하시오.

**기존 버전**

| 항목 | 값 |
|------|-----|
| Deployment 이름 | web-v1 |
| replicas | 5 |
| image | nginx:1.31 |
| label | app=web, version=v1 |
| HTML | VERSION V1 |

**신규 Canary 버전**

| 항목 | 값 |
|------|-----|
| Deployment 이름 | web-v2 |
| replicas | 1 |
| image | nginx:1.31 |
| label | app=web, version=v2 |
| HTML | VERSION V2 - CANARY |

**Service 조건**

| 항목 | 값 |
|------|-----|
| Service | web-service |
| selector | app=web |
| port | 80 |
| targetPort | 80 |

### STEP 1) v1 HTML, v2 HTML 작성

```
[root@k8s-master ~]# mkdir  -p  /canary/v1

[root@k8s-master ~]# vi  /canary/v1/index.html
```

```html
<!DOCTYPE html>
<html>
<head>
  <title>Canary Test</title>
</head>
<body>
  <h1>Canary Version V1</h1>
  <p>Stable Version</p>
</body>
</html>
```

```
[root@k8s-master ~]# mkdir  -p  /canary/v2

[root@k8s-master ~]# vi  /canary/v2/index.html
```

```html
<!DOCTYPE html>
<html>
<head>
  <title>Canary Test</title>
</head>
<body>
  <h1>Canary Version V2</h1>
  <p>Stable Version</p>
</body>
</html>
```

```
[root@k8s-master ~]# ls  -l  /canary/
합계 0
drwxr-xr-x 2 root root 24  8월 25 12:38 v1
drwxr-xr-x 2 root root 24  8월 25 12:39 v2
[root@k8s-master ~]# ls  -lR  /canary/
/canary/:
합계 0
drwxr-xr-x 2 root root 24  8월 25 12:38 v1
drwxr-xr-x 2 root root 24  8월 25 12:39 v2

/canary/v1:
합계 4
-rw-r--r-- 1 root root 143  8월 25 12:38 index.html

/canary/v2:
합계 4
-rw-r--r-- 1 root root 143  8월 25 12:39 index.html
```

### STEP 2) v1 이미지 생성

```
# Dockerfile 생성
[root@k8s-master ~]# vi  /canary/v1/Dockerfile
```

```dockerfile
FROM nginx:1.31
COPY index.html /usr/share/nginx/html/index.html
```

```
[root@k8s-master ~]# cd  /canary/v1

[root@k8s-master v1]# ls  -l
합계 8
-rw-r--r-- 1 root root  65  8월 25 12:42 Dockerfile
-rw-r--r-- 1 root root 143  8월 25 12:38 index.html

# Canary web1  이미지 생성
[root@k8s-master v1]# docker  build  -t  konan7979/canary-web:v1.0  .

[root@k8s-master v1]# docker  images | grep canary
konan7979/canary-web:v1.0          e098fee9e533        236MB         63.3MB

Docker Hub에 업로드
[root@k8s-master v2]# docker  push  konan7979/canary-web:v1.0
```

### STEP 3) v2 이미지 생성

```
# Dockerfile 생성
[root@k8s-master ~]# vi  /canary/v2/Dockerfile
```

```dockerfile
FROM nginx:1.31
COPY index.html /usr/share/nginx/html/index.html
```

```
[root@k8s-master v1]# cd /canary/v2

# Canary web2  이미지 생성
[root@k8s-master v2]# docker build  -t konan7979/canary-web:v2.0  .

[root@k8s-master v1]# docker  images | grep canary
konan7979/canary-web:v1.0          e098fee9e533        236MB         63.3MB
konan7979/canary-web:v2.0          18d7f06eb446        236MB         63.3MB

Docker Hub에 업로드
[root@k8s-master v2]# docker push konan7979/canary-web:v2.0
```

### STEP 4) Canary v1 Deployment 생성

```
[root@k8s-master ~]# cd ~

[root@k8s-master ~]# vi canary-v1.yaml
```

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-v1
spec:
  replicas: 5
  selector:
    matchLabels:
      app: web
      version: v1

  template:
    metadata:
      labels:
        app: web
        version: v1
    spec:
      containers:
      - name: nginx
        image: knan7979/canary-web:v1.0
        ports:
        - containerPort: 80
```

```
[root@k8s-master ~]# kubectl  apply -f  canary-v1.yaml
deployment.apps/web-v1 created

[root@k8s-master ~]# kubectl  get  deployments  web-v1
NAME     READY   UP-TO-DATE   AVAILABLE   AGE
web-v1    5/5        5                     5                13s

[root@k8s-master ~]# kubectl  get  pods
NAME                      	READY   STATUS    RESTARTS   AGE
web-v1-5dbbd65577-4cw9l   	1/1     Running   0          2s
web-v1-5dbbd65577-7dm44   	1/1     Running   0          2s
web-v1-5dbbd65577-85gcp   	1/1     Running   0          2s
web-v1-5dbbd65577-mk825   	1/1     Running   0          2s
web-v1-5dbbd65577-w22p6   	1/1     Running   0          2s
```

### STEP 5) Canary v2 Deployment 생성

```
[root@k8s-master ~]# vi canary-v2.yaml
```

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-v2
spec:
  replicas: 1

  selector:
    matchLabels:
      app: web
      version: v2

  template:
    metadata:
      labels:
        app: web
        version: v2
    spec:
      containers:
      - name: nginx
        image: konan7979/canary-web:v2.0
        ports:
        - containerPort: 80
```

```
[root@k8s-master ~]# kubectl  apply -f  canary-v2.yaml
deployment.apps/web-v2 created

[root@k8s-master ~]#  kubectl  get  deployments  web-v2
NAME     READY   UP-TO-DATE   AVAILABLE   AGE
web-v2    1/1        1                    1                  12s

[root@k8s-master ~]#  kubectl  get  pods
NAME                      	READY   STATUS    RESTARTS   AGE
web-v1-5dbbd65577-4cw9l   	1/1        Running      0                2m6s
web-v1-5dbbd65577-7dm44   	1/1        Running      0                2m6s
web-v1-5dbbd65577-85gcp   	1/1        Running      0                2m6s
web-v1-5dbbd65577-mk825   	1/1        Running      0                2m6s
web-v1-5dbbd65577-w22p6   	1/1        Running      0                2m6s
web-v2-5d5dd8c4b9-wgcgn   	1/1        Running      0                19s

[root@k8s-master ~]# kubectl  get  pods  --show-labels
NAME                      	READY   STATUS    RESTARTS   AGE     LABELS
web-v1-5dbbd65577-4cw9l   	1/1     Running   0          3m28s   app=web,pod-template-hash=5dbbd65577,version=v1
web-v1-5dbbd65577-7dm44   	1/1     Running   0          3m28s   app=web,pod-template-hash=5dbbd65577,version=v1
web-v1-5dbbd65577-85gcp   	1/1     Running   0          3m28s   app=web,pod-template-hash=5dbbd65577,version=v1
web-v1-5dbbd65577-mk825   	1/1     Running   0          3m28s   app=web,pod-template-hash=5dbbd65577,version=v1
web-v1-5dbbd65577-w22p6   	1/1     Running   0          3m28s   app=web,pod-template-hash=5dbbd65577,version=v1
web-v2-5d5dd8c4b9-wgcgn   	1/1     Running   0          101s    app=web,pod-template-hash=5d5dd8c4b9,version=v2

[root@k8s-master ~]# kubectl get pods -L version,app
NAME                     	 READY   STATUS    RESTARTS   AGE     VERSION   APP
web-v1-5dbbd65577-4cw9l   	1/1          Running     0                6m3s     v1            web
web-v1-5dbbd65577-7dm44   	1/1          Running     0                6m3s     v1            web
web-v1-5dbbd65577-85gcp   	1/1          Running     0                6m3s     v1            web
web-v1-5dbbd65577-mk825   	1/1          Running     0                6m3s     v1            web
web-v1-5dbbd65577-w22p6   	1/1          Running     0                6m3s     v1            web
web-v2-5d5dd8c4b9-wgcgn   	1/1          Running     0                4m16s    v2           web
```

### STEP 6) Service 생성

Service는 version 라벨을 사용하지 않는다. 공통 라벨: `app=web`

```
[root@k8s-master ~]# vi canary-service.yaml
```

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-service

spec:
  selector:
    app: web
  ports:
  - port: 80
    targetPort: 80
```

```
[root@k8s-master ~]# kubectl  apply  -f  canary-service.yaml
service/web-service created

[root@k8s-master ~]# kubectl  get  service  web-service
NAME          TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
web-service   ClusterIP   10.109.198.12   <none>        80/TCP    10s

[root@k8s-master ~]# kubectl get pods  -L app
NAME                      	READY   STATUS    RESTARTS   AGE     APP
web-v1-5dbbd65577-4cw9l   	1/1         Running     0                10m      web
web-v1-5dbbd65577-7dm44   	1/1         Running     0                10m      web
web-v1-5dbbd65577-85gcp   	1/1         Running     0                10m      web
web-v1-5dbbd65577-mk825   	1/1         Running     0                10m      web
web-v1-5dbbd65577-w22p6   	1/1         Running     0                10m      web
web-v2-5d5dd8c4b9-wgcgn   	1/1         Running     0                8m6s    web

[root@k8s-master ~]# kubectl  describe  service  web-service
Name:                     	web-service
Namespace:                	default
Labels:                   	<none>
Annotations:              	<none>
Selector:                 	app=web
Type:                     	ClusterIP
IP Family Policy:         	SingleStack
IP Families:              	IPv4
IP:                       	10.109.198.12
IPs:                      	10.109.198.12
Port:                     	<unset>  80/TCP
TargetPort:               	80/TCP
Endpoints:                	10.244.2.27:80,10.244.2.29:80,10.244.1.23:80 + 3 more...
Session Affinity:         	None
Internal Traffic Policy:  	Cluster
Events:                   	<none>

[root@k8s-master ~]# kubectl  get endpoints  web-service
Warning: v1 Endpoints is deprecated in v1.33+; use discovery.k8s.io/v1 EndpointSlice
NAME           ENDPOINTS                                                           AGE
web-service   10.244.1.22:80,10.244.1.23:80,10.244.1.24:80 + 3 more...   2m56s

[root@k8s-master ~]# kubectl  get  endpointslices
NAME                	ADDRESSTYPE   PORTS   ENDPOINTS                                    	AGE
kubernetes          	IPv4       	6443 	192.168.10.100                                    	13d
web-service-5s6z6	IPv4      	80    	10.244.2.27,10.244.2.29,10.244.1.23 + 3 more...	6m8s

[root@k8s-master ~]# kubectl  get  endpointslices  web-service-5s6z6
NAME                	ADDRESSTYPE   PORTS   ENDPOINTS                                    	AGE
web-service-5s6z6	IPv4      	80    	10.244.2.27,10.244.2.29,10.244.1.23 + 3 more...	6m8s

[root@k8s-master ~]# kubectl  get  endpointslices  -l  kubernetes.io/service-name=web-service
NAME                ADDRESSTYPE   PORTS   ENDPOINTS                                    	AGE
web-service-5s6z6   IPv4          80      10.244.2.27,10.244.2.29,10.244.1.23 + 3 more...	7m

# v1 Pod 확인
root@k8s-master ~]# kubectl  get  pods  -l  version=v1  -o  wide
NAME                      	READY   STATUS    RESTARTS   AGE   IP               NODE          NOMINATED NODE   READINESS GATES
web-v1-5dbbd65577-4cw9l   	1/1         Running     0                87m    10.244.2.28   k8s-worker2   <none>           <none>
web-v1-5dbbd65577-7dm44   	1/1         Running     0                87m    10.244.1.23   k8s-worker1   <none>           <none>
web-v1-5dbbd65577-85gcp   	1/1         Running     0                87m    10.244.2.27   k8s-worker2   <none>           <none>
web-v1-5dbbd65577-mk825   	1/1         Running     0                87m    10.244.2.29   k8s-worker2   <none>           <none>
web-v1-5dbbd65577-w22p6   	1/1         Running     0                87m    10.244.1.22   k8s-worker1   <none>           <none>

[root@k8s-master ~]# curl  http://10.244.2.28
<!DOCTYPE html>
<html>
<head>
  <title>Canary Test</title>
</head>
<body>
  <h1>Canary Version V1</h1>
  <p>Stable Version</p>
</body>
</html>

# v2 Pod 확인
[root@k8s-master ~]# kubectl  get  pods  -l  version=v2  -o  wide
NAME                      	READY   STATUS    RESTARTS   AGE   IP               NODE          NOMINATED NODE   READINESS GATES
web-v2-5d5dd8c4b9-wgcgn     1/1        Running      0                87m    10.244.1.24   k8s-worker1   <none>                  <none>

[root@k8s-master ~]# curl  http://10.244.1.24
<!DOCTYPE html>
<html>
<head>
  <title>Canary Test</title>
</head>
<body>
  <h1>Canary Version V2</h1>
  <p>Stable Version</p>
</body>
</html>
```

### STEP 7) Service를 통한 Canary 확인

```
[root@k8s-master ~]# kubectl  get  svc  web-service
NAME          TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
web-service   ClusterIP   10.109.198.12     <none>              80/TCP    82m

[root@k8s-master ~]# for  i  in  {1..10}
> do
>    curl -s  http://10.109.198.12 | grep "<h1>";
> done
  <h1>Canary Version V1</h1>
  <h1>Canary Version V1</h1>
  <h1>Canary Version V1</h1>
  <h1>Canary Version V1</h1>
  <h1>Canary Version V1</h1>
  <h1>Canary Version V2</h1>	<--- Canary 배포
  <h1>Canary Version V1</h1>
  <h1>Canary Version V1</h1>
  <h1>Canary Version V2</h1>	<--- Canary 배포
  <h1>Canary Version V1</h1>
```

### STEP 8) Canary Pod 비율 변경

Pod 개수: v1 5개, v2 1개

```
                                            web-service
                                                   |
	               +------------------------+-----------------------+
	               |                               	                                |
	   Deployment web-v1                                            Deployment web-v2
	      replicas: 3                                                             replicas: 1
	               |                               	                                 |
	      +-----+-----+-----+-----+                         	      |
	      |       |        |       |       |                          	      |
	     v1      v1      v1     v1     v1                                           v2

	     VERSION V1                                                 VERSION V2 - CANARY
```

```
# Canary Pod 비율 변경
[root@k8s-master ~]# kubectl  scale  deployment  web-v2  --replicas=3
deployment.apps/web-v2 scaled

[root@k8s-master ~]# kubectl  get  pods  -L  version
NAME                      	READY   STATUS    RESTARTS   AGE    VERSION
web-v1-5dbbd65577-4cw9l   	1/1     Running   0          100m   v1
web-v1-5dbbd65577-7dm44   	1/1     Running   0          100m   v1
web-v1-5dbbd65577-85gcp   	1/1     Running   0          100m   v1
web-v1-5dbbd65577-mk825   	1/1     Running   0          100m   v1
web-v1-5dbbd65577-w22p6   	1/1     Running   0          100m   v1
web-v2-5d5dd8c4b9-7wzsp   	1/1     Running   0          40s    v2
web-v2-5d5dd8c4b9-jcklb   	1/1     Running   0          40s    v2
web-v2-5d5dd8c4b9-wgcgn   	1/1     Running   0          98m    v2

[root@k8s-master ~]# for  i  in  {1..10}
> do
>    curl -s  http://10.109.198.12 | grep "<h1>";
> done

[root@k8s-master ~]# for  i  in  {1..10}; do   curl -s  http://10.109.198.12 | grep "<h1>"; done
  <h1>Canary Version V2</h1>	<--- Canary 배포
  <h1>Canary Version V1</h1>
  <h1>Canary Version V2</h1>	<--- Canary 배포
  <h1>Canary Version V1</h1>
  <h1>Canary Version V1</h1>
  <h1>Canary Version V2</h1>	<--- Canary 배포
  <h1>Canary Version V2</h1>	<--- Canary 배포
  <h1>Canary Version V1</h1>
  <h1>Canary Version V1</h1>
  <h1>Canary Version V1</h1>
```

이전보다 "VERSION V2 - CANARY" 응답을 볼 가능성이 증가한다.

### STEP 9) Canary에 문제가 발생한 경우

신규 버전 v2에서 오류가 발생했다고 가정한다. v2를 즉시 0개로 줄인다.

```
# Canary Pod 비율 변경
[root@k8s-master ~]#  kubectl  scale  deployment  web-v2  --replicas=0
deployment.apps/web-v2 scaled

[root@k8s-master ~]# kubectl  get  pods  -L  version
NAME                      	READY   STATUS    RESTARTS   AGE    VERSION
web-v1-5dbbd65577-4cw9l   	1/1        Running     0                 104m    v1
web-v1-5dbbd65577-7dm44   	1/1        Running     0                 104m    v1
web-v1-5dbbd65577-85gcp   	1/1        Running     0                 104m    v1
web-v1-5dbbd65577-mk825   	1/1        Running     0                 104m    v1
web-v1-5dbbd65577-w22p6   	1/1        Running     0                 104m    v1

[root@k8s-master ~]# for  i  in  {1..10}; do   curl -s  http://10.109.198.12 | grep "<h1>"; done
  <h1>Canary Version V1</h1>
  <h1>Canary Version V1</h1>
  <h1>Canary Version V1</h1>
  <h1>Canary Version V1</h1>
  <h1>Canary Version V1</h1>
  <h1>Canary Version V1</h1>
  <h1>Canary Version V1</h1>
  <h1>Canary Version V1</h1>
  <h1>Canary Version V1</h1>
  <h1>Canary Version V1</h1>
```

### web-ingress로 Canary 서비스 외부 노출

```
[root@k8s-master ~]# cat web-ingress.yaml
```

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web-ingress

spec:
  ingressClassName: nginx

  rules:
  - http:
      paths:
      - path: /
        pathType: Prefix

        backend:
          service:
            name: web-service
            port:
              number: 80
```

```
[root@k8s-master ~]# kubectl  get  service  -n ingress-nginx
NAME                                 	TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                              AGE
ingress-nginx-controller             	NodePort    10.105.3.121     	<none>            80:32181/TCP,443:30366/TCP   28h
ingress-nginx-controller-admission	ClusterIP   10.101.171.143	<none>            443/TCP                              28h

# 웹브라우저로 접속
https://192.168.10.100:30366/
```

10접 접속시 모두 v1으로 접속된다 (web-v2가 replicas=0인 상태):

```
  <h1>Canary Version V1</h1>
  <h1>Canary Version V1</h1>
  <h1>Canary Version V1</h1>
  <h1>Canary Version V1</h1>
  <h1>Canary Version V1</h1>
  <h1>Canary Version V1</h1>
  <h1>Canary Version V1</h1>
  <h1>Canary Version V1</h1>
  <h1>Canary Version V1</h1>
  <h1>Canary Version V1</h1>
```

```
# Scale Out
[root@k8s-master ~]#  kubectl  scale  deployment  web-v2  --replicas=3
deployment.apps/web-v2 scaled
```

10접 접속시:

```
  <h1>Canary Version V1</h1>
  <h1>Canary Version V1</h1>
  <h1>Canary Version V2</h1>
  <h1>Canary Version V1</h1>
  <h1>Canary Version V1</h1>
  <h1>Canary Version V2</h1>
  <h1>Canary Version V1</h1>
  <h1>Canary Version V1</h1>
  <h1>Canary Version V2</h1>
  <h1>Canary Version V1</h1>
```

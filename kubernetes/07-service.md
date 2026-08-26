# Kubernetes Service

## 목차

1. [Service란?](#service란)
2. [Service의 핵심 역할](#service의-핵심-역할)
3. [Service가 하는 일 (동작 개념)](#service가-하는-일-동작-개념)
4. [Service가 Pod를 찾는 방법 (Label & Selector)](#service가-pod를-찾는-방법-label--selector)
5. [Kubernetes Service Type](#kubernetes-service-type)
6. [Service 주요 타입 상세](#service-주요-타입-상세)
7. [Service 타입별 실무 활용 가이드](#service-타입별-실무-활용-가이드)
8. [Deployment + Service YAML 기본 예시](#deployment--service-yaml-기본-예시)
9. [ClusterIP 실습](#clusterip-실습)
10. [NodePort 실습](#nodeport-실습)
11. [LoadBalancer 실습](#loadbalancer-실습)
12. [ExternalName 실습](#externalname-실습)
13. [EX) ExternalName Service를 생성하여 naver라는 이름으로 google.com에 접근](#ex-externalname-service를-생성하여-naver라는-이름으로-googlecom에-접근)
14. [Headless Service 실습](#headless-service-실습)
15. [EX) Headless Service와 StatefulSet을 연동해서 Pod 3개를 생성하고, 각 Pod가 고정된 이름으로 DNS 조회되는지 확인](#ex-headless-service와-statefulset을-연동해서-pod-3개를-생성하고-각-pod가-고정된-이름으로-dns-조회되는지-확인)

---

## Service란?

쿠버네티스 **Service**는 계속 바뀌는 Pod들을 대신해서 고정된 단일 진입점(주소와 이름)을 제공해주는 객체이다.

쿠버네티스에서 실제 애플리케이션은 Pod 안에서 실행된다.
하지만 Pod는 고정된 서버처럼 계속 유지되는 리소스가 아니다.

**Pod가 변경되는 이유:**
1. 노드 장애
2. 컨테이너 오류
3. 스케일 조정
4. 업데이트 (Rolling Update)

Pod가 새로 만들어질 때마다 IP 주소가 매번 바뀐다.

즉, 어제 접속하던 Pod IP로 오늘은 접속이 안 되는 상황이 정상적인 동작이다.

**EX) 예제 상황 (웹 서버 Pod 3개 실행 중)**

각각 IP가:
- `10.244.1.10`
- `10.244.1.11`
- `10.244.2.5`

이 중 하나의 Pod가 죽고 다시 생성되면 IP는 전혀 다른 값으로 바뀐다.
이 상태에서 DB, 다른 서비스, 외부 사용자가 Pod IP를 직접 사용한다면 서비스는 바로 끊어진다.
그래서 필요한 것이 Service다.

> **정리**: Pod의 IP는 언제든 바뀔 수 있으므로, 바뀌지 않는 고정 접점 역할을 하는 Service가 필요하다.

---

## Service의 핵심 역할

**Service**는 다음 역할을 한다:
- 여러 Pod를 하나의 서비스로 묶는다.
- 고정된 접근 지점을 제공한다. (단일 진입점 제공)
- Pod가 바뀌어도 사용자는 신경 쓸 필요가 없다.
- Service 내부에서 자동으로 로드밸런싱을 수행한다.

Service는 Pod들이 바뀌어도 항상 동일한 주소와 이름으로 접근할 수 있게 해주는 중간 관리자다.

---

## Service가 하는 일 (동작 개념)

- Service는 Pod를 직접 생성하지 않는다.
- Service는 Pod를 선택만 한다.
- 선택 기준은 **label**(레이블) 이다.

**예시 개념 흐름:**

1. Pod 생성 → `label: app=web`
2. Service 생성 → `selector: app=web`
3. Service는 `app=web` 라벨을 가진 모든 Pod를 자동으로 추적
4. 사용자는 Pod IP가 아니라 Service IP 주소로 접속
5. Service는 내부적으로 여러 개의 Pod 중 하나로 트래픽 전달

Pod가 추가되거나 삭제되면 Service는 자동으로 대상 목록을 갱신한다.

> **정리**: Service는 Pod를 직접 관리하지 않고 label을 기준으로 대상을 자동 추적하며, 실제 트래픽 전달과 대상 갱신은 아래에서 다루는 **selector**와 **Endpoints** 메커니즘을 통해 이뤄진다.

---

## Service가 Pod를 찾는 방법 (Label & Selector)

Service가 Pod를 식별하는 핵심 수단은 **selector**이다. Pod에 붙은 label과 Service의 selector가 일치하면 자동으로 연결된다.

**Pod 예시:**
```yaml
metadata:
  labels:
    app: web
    tier: frontend
```

**Service 예시:**
```yaml
spec:
  selector:
    app: web
```

이 Service는 `app=web` 라벨을 가진 Pod만 대상으로 삼는다.
Service는 Pod 이름, Pod IP를 전혀 몰라도 된다.
Label만 맞으면 자동으로 Service와 연결된다.

> **정리**: selector와 label의 일치 여부만으로 Service-Pod 연결이 결정되며, 이 관계는 쿠버네티스 내부적으로 **Endpoints** 객체에 실시간으로 반영된다.

---

## Kubernetes Service Type

쿠버네티스에서 Pod는 생성되고 삭제될 수 있으며, Pod가 다시 생성되면 IP 주소가 변경될 수 있다.
따라서 클라이언트가 Pod의 IP를 직접 사용하면 문제가 발생한다.

**Service**는 Pod에 접근하기 위한 고정된 네트워크 엔드포인트를 제공하고, 선택된 Pod로 트래픽을 전달하는 논리적 객체다. 쿠버네티스는 용도에 따라 여러 **Service Type**을 제공한다.

**Service Type 목록:**
| 타입 | 설명 |
|------|------|
| ClusterIP | 클러스터 내부에서만 접근 가능 (기본값) |
| NodePort | 각 노드의 포트를 열어 외부 접근 허용 |
| LoadBalancer | 외부 클라우드 로드밸런서를 통한 접근 |
| ExternalName | 외부 DNS 이름을 Service 이름으로 매핑 |
| Headless Service | ClusterIP가 없는 Service (Pod IP 직접 노출) |

---

## Service 주요 타입 상세

### 1) ClusterIP (기본값)

- **ClusterIP**는 Service의 기본 타입이다.
- 클러스터 내부에서만 접근할 수 있는 가상 IP(ClusterIP)를 제공한다.
- 외부에서 직접 접근할 수 없다.
- 주로 클러스터 내부의 마이크로서비스 간 통신에 사용한다.

**사용 예:**
- Web → API
- API → DB
- Frontend → Backend

---

### 2) NodePort

- **NodePort**는 각 Node의 특정 포트를 열어 외부에서 Service에 접근할 수 있도록 한다.
- 일반적으로 `30000~32767` 범위의 포트를 사용한다.

```
Node IP:NodePort  →  Service  →  Pod
```

**예:**
- Node 1 : `192.168.10.101:30080`
- Node 2 : `192.168.10.102:30080`
- Node 3 : `192.168.10.103:30080`

각 Node에서 동일한 NodePort를 통해 Service에 접근할 수 있다.
운영 환경에서는 직접 NodePort를 노출하기보다 LoadBalancer나 Ingress 등을 함께 사용하는 경우가 많다.

**NodePort의 단점:**
- Node의 IP와 포트를 외부에 노출해야 함
- 외부 사용자에게 직접 NodePort를 제공하는 구조는 운영 환경에서 관리가 불편할 수 있음
- 일반적으로 LoadBalancer나 Ingress 등을 함께 사용

---

### 3) LoadBalancer

- **LoadBalancer**는 외부 Load Balancer를 통해 Service에 접근할 수 있도록 하는 타입이다.
- 클라우드 환경에서 주로 사용한다. (AWS, Azure, GCP...)
- 클라우드 환경의 Kubernetes에서 LoadBalancer Service를 생성하면 클라우드 환경에 따라 외부 Load Balancer가 프로비저닝될 수 있다.

```
외부 사용자  →  Cloud Load Balancer  →  Service  →  Pod
```

외부 사용자가 Node의 IP와 NodePort를 직접 사용할 필요를 줄일 수 있다.

---

### 4) ExternalName

- **ExternalName**은 쿠버네티스 외부에 있는 서비스의 DNS 이름을 Service 이름으로 매핑한다.
- 즉, Pod를 연결하는 것이 아니라 "Service 이름 → 외부 DNS 이름"을 연결하는 방식이다.
- 셀렉터(selector)를 사용하지 않는다. (연결할 Pod가 없기 때문에 Pod를 선택하지 않는다.)
- 클러스터 내부의 애플리케이션이 외부 서비스를 쿠버네티스 내부의 Service 이름으로 사용할 수 있도록 하기 위한 목적

**동작 방식:**
1. 클러스터 내부에서 Service 이름으로 DNS 조회
2. ExternalName Service
3. 외부 DNS 이름(CNAME) 반환
4. 외부 서비스로 접근

**예시 YAML (yaml에 pod가 존재하지 않는다):**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: mydb
spec:
  type: ExternalName
  externalName: db.external.com
```

- 외부 DB 주소 : `db.external.com`
- ExternalName Service : `mydb`
- 클러스터 내부에서 `mydb.default.svc.cluster.local` ---- CNAME ---→ `db.external.com`

```
mydb.default.svc.cluster.local
│     │       │      └───────────── 클러스터 내부 DNS 영역
│     │       └───────────────── Service
│     └───────────────────────── Namespace
└─────────────────────────────── Service 이름
```

`mydb.default.svc.cluster.local`는 Kubernetes DNS가 자동으로 만들어주는 이름이다.

**주 사용 사례:**
- 외부 DB
- 외부 API
- 외부 SaaS 서비스
- Kubernetes 외부에서 운영되는 서비스 연동

---

### 5) Headless Service

- **Headless Service**는 ClusterIP에 IP 주소가 없는 Service다.
- `clusterIP: None`으로 설정한다.
- 일반적인 Service처럼 하나의 ClusterIP로 트래픽을 전달하지 않는다.
- DNS 조회 시 Service의 ClusterIP 대신 연결된 Pod들의 IP를 조회할 수 있다.
- 주로 개별 Pod에 직접 접근해야 하는 경우 사용

**사용 예:**
- StatefulSet
- 데이터베이스 클러스터
- 각 Pod를 개별적으로 식별해야 하는 경우

> **정리**: ClusterIP, NodePort, LoadBalancer, ExternalName, Headless Service는 각각 접근 범위와 용도가 다르며, 실무에서는 상황에 맞게 조합해서 사용한다.

---

## Service 타입별 실무 활용 가이드

### ClusterIP — 내부 마이크로서비스 간 통신

**언제 사용하는가:**
- 외부에 노출할 필요 없이 클러스터 내부에서만 통신하는 경우
- 마이크로서비스 아키텍처에서 서비스 간 내부 API 호출

**실무 활용 예시:**
- **Web → API 서버**: 프론트엔드 Pod가 백엔드 API Pod를 ClusterIP Service 이름으로 호출
- **API → DB**: 백엔드 서비스가 데이터베이스 Pod에 내부 Service 이름(예: `mysql-svc`)으로 접속
- **Frontend → Backend**: React/Vue 프론트엔드가 같은 클러스터 내 Express/FastAPI 백엔드를 호출

**특징 요약:**
- `type` 생략 시 자동으로 ClusterIP가 적용됨 (기본값)
- 외부 노출 없이 안전하게 내부 서비스 연결 가능
- 클러스터 내부 DNS를 통해 `서비스명.네임스페이스.svc.cluster.local` 형태로 접근

---

### NodePort — 개발/테스트 환경 외부 접근

**언제 사용하는가:**
- 운영 환경이 아닌 개발/테스트/PoC 환경에서 외부 접근이 필요할 때
- 클라우드 로드밸런서 없이 간단하게 외부 노출이 필요한 경우
- 사내 테스트 서버나 온프레미스 쿠버네티스 환경에서 임시 접근

**실무 활용 예시:**
- **개발 환경 테스트**: 개발자가 로컬 PC에서 클러스터 내 앱을 `192.168.10.101:30080` 형태로 직접 접근
- **CI/CD 파이프라인 검증**: 배포 후 빠른 동작 확인을 위해 NodePort로 임시 접근
- **온프레미스 내부 서비스 노출**: 사내망에서만 사용하는 서비스를 노드 포트로 접근

**주의사항:**
- 운영 환경에서는 NodePort 직접 노출보다 LoadBalancer나 Ingress 사용 권장
- 포트 범위가 `30000~32767`로 제한되어 있어 표준 포트(80, 443) 사용 불가

---

### LoadBalancer — 클라우드 운영 환경의 외부 서비스 노출

**언제 사용하는가:**
- AWS EKS, Azure AKS, Google GKE 등 퍼블릭 클라우드의 Kubernetes에서 외부 트래픽을 받을 때
- 외부 사용자에게 서비스를 안정적으로 제공해야 하는 운영 환경

**실무 활용 예시:**
- **AWS EKS**: LoadBalancer Service 생성 시 AWS ALB/NLB가 자동 프로비저닝되어 외부 IP 할당
- **GCP GKE**: Google Cloud Load Balancer가 자동으로 연결되어 글로벌 트래픽 처리
- **Azure AKS**: Azure Load Balancer가 자동 생성되어 외부 접근 URL 제공

**특징 요약:**
- 클라우드 환경이 아닌 경우 `EXTERNAL-IP`가 `<pending>` 상태로 유지됨
- 일반적으로 Ingress Controller와 함께 사용하여 HTTP/HTTPS 라우팅 처리

---

### ExternalName — 클러스터 내부에서 외부 서비스 접근

**언제 사용하는가:**
- 클러스터 외부에 있는 서비스(DB, API, SaaS)를 클러스터 내부의 Service 이름으로 접근하고 싶을 때
- 나중에 외부 서비스를 내부로 마이그레이션할 계획이 있을 때 (코드 변경 없이 Service 이름만 유지)

**실무 활용 예시:**
- **외부 RDS 접근**: AWS RDS의 DNS 엔드포인트(`mydb.xxxx.rds.amazonaws.com`)를 `db-svc`라는 이름으로 매핑 → 앱 코드에서는 `db-svc`로만 접속
- **외부 API 연동**: `api.third-party.com`을 `payment-api`라는 서비스 이름으로 매핑
- **마이그레이션 전환**: 외부 서비스 → 내부 서비스 전환 시 Service 이름만 유지하고 `externalName` 제거

**특징 요약:**
- Pod(Selector)가 없으므로 Endpoint도 생성되지 않음
- DNS CNAME 방식으로 동작 (`Service 이름 → 외부 DNS`)

---

### Headless Service — StatefulSet, DB 클러스터, 개별 Pod 직접 접근

**언제 사용하는가:**
- 각 Pod를 개별적으로 식별해야 하는 경우 (StatefulSet과 함께)
- 데이터베이스 클러스터에서 Primary/Replica를 구분해서 접근할 때
- 특정 Pod로 반드시 접속해야 하는 Stateful 애플리케이션

**실무 활용 예시:**
- **MySQL Cluster**: `mysql-0`(Primary 쓰기용), `mysql-1`, `mysql-2`(Replica 읽기용)를 각각 고정 DNS 이름으로 접근
- **Kafka/Zookeeper**: 브로커 각각을 `kafka-0.kafka-headless`, `kafka-1.kafka-headless`로 직접 지정
- **Elasticsearch**: 각 데이터 노드를 고정 이름으로 식별하여 클러스터 구성
- **Redis Sentinel**: 마스터/슬레이브를 개별 Pod DNS로 구분하여 접근

**특징 요약:**
- `clusterIP: None` 설정으로 가상 IP 없이 DNS만 관리
- DNS 조회 시 Service IP 대신 Pod IP 목록이 직접 반환됨
- kube-proxy가 개입하지 않아 트래픽 분산 없이 Pod 직접 접근

> **정리**: 실무에서는 내부 통신은 ClusterIP, 임시/개발용 외부 접근은 NodePort, 운영 환경 외부 노출은 LoadBalancer, 외부 서비스 연동은 ExternalName, Stateful 워크로드는 Headless Service를 선택하는 것이 일반적인 기준이다. 이제 실제 YAML 예시를 통해 Deployment와 Service가 어떻게 연결되는지 살펴본다.

---

## Deployment + Service YAML 기본 예시

### Deployment YAML
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deploy-nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: webui
  template:
    metadata:
      name: nginx-pod
      labels:
        app: webui    # 아래 Service의 selector와 연결되는 핵심 기준
    spec:
      containers:
      - name: nginx-container
        image: nginx:1.31
        ports:
        - containerPort: 80
```

### Service YAML
```yaml
apiVersion: v1
kind: Service
metadata:
  name: webui-svc
spec:
  selector:
    app: webui    # app=webui 라벨을 가진 Pod만 이 Service의 대상으로 선택함
  ports:
  - protocol: TCP
    port: 80         # Service가 클라이언트의 요청을 받는 포트
    targetPort: 80   # Service가 선택된 Pod로 트래픽을 전달할 포트
```

---

## ClusterIP 실습

앞서 살펴본 **ClusterIP** 타입을 실제로 생성하고 동작을 확인하는 실습이다.

### 개요
- selector의 label이 동일한 파드들의 그룹으로 묶어 단일 진입점(Virtual IP)을 생성
- 클러스터 내부에서만 사용 가능
- type 생략 시 default 값으로 `10.96.0.0/12` 범위에서 랜덤하게 할당된다.

### Deployment 생성

```bash
[root@k8s-master ~]# vi deploy-nginx.yaml
```
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deploy-web-dep
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web-dep
  template:
    metadata:
      name: nginx-pod
      labels:
        app: web-dep
    spec:
      containers:
      - name: nginx-container
        image: nginx:1.31
```

```bash
[root@k8s-master ~]# kubectl  apply  -f  deploy-nginx.yaml
deployment.apps/deploy-web-dep created
```

```bash
[root@k8s-master ~]# kubectl  get  pods
NAME                              READY   STATUS    RESTARTS   AGE
deploy-web-dep-747497b79d-bptqs   1/1     Running   0          4s
deploy-web-dep-747497b79d-gdrnq   1/1     Running   0          4s
deploy-web-dep-747497b79d-lnw72   1/1     Running   0          4s
```

```bash
[root@k8s-master ~]# kubectl  get  pods  -o  wide
NAME                              READY   STATUS    RESTARTS   AGE   IP              NODE          NOMINATED NODE   READINESS GATES
deploy-web-dep-747497b79d-bptqs   1/1     Running   0          63s   10.244.2.95   k8s-worker2   <none>           <none>
deploy-web-dep-747497b79d-gdrnq   1/1     Running   0          63s   10.244.2.94   k8s-worker2   <none>           <none>
deploy-web-dep-747497b79d-lnw72   1/1     Running   0          63s   10.244.1.18   k8s-worker1   <none>           <none>
```

### ClusterIP Service 생성

```bash
[root@k8s-master ~]# vi clusterip-nginx.yaml
```
```yaml
apiVersion: v1                  # 쿠버네티스 core API 버전 (Service는 v1)
kind: Service                   # 생성할 리소스 종류가 Service임을 의미
metadata:
  name: clusterip-service       # Service 이름 (클러스터 내부 DNS 이름으로 사용됨)
spec:
  type: ClusterIP               # Service 타입 (ClusterIP는 클러스터 내부에서만 접근 가능한 서비스)
  clusterIP: 10.100.100.100     # Service에 할당할 가상 IP(Virtual IP), 명시하지 않으면 쿠버네티스가 자동 할당
  selector:
    app: web-dep                # app=web-dep 라벨을 가진 Pod들을 이 Service의 대상으로 선택
                                # 해당 라벨이 있는 Pod들만 트래픽을 받음
  ports:
  - protocol: TCP               # 통신 프로토콜 (일반적인 웹 서비스는 TCP)
    port: 80                    # Service가 외부(클러스터 내부)에 제공하는 포트
    targetPort: 80              # 실제 Pod(컨테이너)가 리스닝 중인 포트
                                # Service로 들어온 요청이 이 포트로 전달됨
```

```bash
[root@k8s-master ~]# kubectl  apply  -f  clusterip-nginx.yaml  --dry-run=client
service/clusterip-service created (dry run)
```

```bash
[root@k8s-master ~]# kubectl  apply  -f  clusterip-nginx.yaml
service/clusterip-service created
```

### Service 확인

```bash
[root@k8s-master ~]# kubectl  get  service
NAME                TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
clusterip-service   ClusterIP   10.100.100.100  <none>        80/TCP     11s
kubernetes          ClusterIP   10.96.0.1       <none>        443/TCP    9d
```

```bash
[root@k8s-master ~]# kubectl  get  svc
NAME                TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
clusterip-service   ClusterIP   10.100.100.100  <none>        80/TCP     11s
kubernetes          ClusterIP   10.96.0.1       <none>        443/TCP    9d
```

```bash
[root@k8s-master ~]# kubectl  get  svc  clusterip-service
NAME                TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
clusterip-service   ClusterIP   10.100.100.100  <none>        80/TCP     11s
```

```bash
[root@k8s-master ~]# kubectl  describe  svc  clusterip-service
Name:                   clusterip-service
Namespace:              default
Labels:                 <none>
Annotations:            <none>
Selector:               app=web-dep
Type:                   ClusterIP
IP Family Policy:       SingleStack
IP Families:            IPv4
IP:                     10.100.100.100
IPs:                    10.100.100.100
Port:                   <unset>  80/TCP
TargetPort:             80/TCP
Endpoints:              10.244.2.94:80,10.244.1.18:80,10.244.2.95:80
Session Affinity:       None
Internal Traffic Policy: Cluster
Events:                 <none>
```

### Service 접속 확인

```bash
[root@k8s-master ~]# curl  10.100.100.100
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

<p><em>Thank you for using nginx!</em></p>
</body>
</html>
```

- `deploy-web-dep-747497b79d-bptqs`
- `deploy-web-dep-747497b79d-gdrnq`
- `deploy-web-dep-747497b79d-lnw72`

3개의 Pod 중 어떤 Pod에 접속했는지 확인할 수 없다.

---

### 어떤 Pod로 접속했는지 확인할 수 있도록 index.html 코드 수정

#### 첫번째 Pod 수정

```bash
[root@k8s-master ~]# kubectl  exec  deploy-web-dep-747497b79d-bptqs  -it  -- /bin/bash
root@deploy-web-dep-747497b79d-bptqs:/#

root@deploy-web-dep-747497b79d-bptqs:/# ls  -l  /usr/share/nginx/html/
total 8
-rw-r--r-- 1 root root 497 Jul 15 16:03 50x.html
-rw-r--r-- 1 root root 896 Jul 15 16:03 index.html

root@deploy-web-dep-747497b79d-bptqs:/# cd  /usr/share/nginx/html/

root@deploy-web-dep-747497b79d-bptqs:/usr/share/nginx/html#  echo "<h1> ClusterIP Service Web-dep-1 </h1>"  >  index.html

root@deploy-web-dep-747497b79d-bptqs:/usr/share/nginx/html# cat index.html
<h1> ClusterIP Service Web-dep-1 </h1>
```

#### 두번째 Pod 수정

```bash
[root@k8s-master ~]# kubectl  exec  deploy-web-dep-747497b79d-gdrnq  -it  -- /bin/bash
root@deploy-web-dep-747497b79d-gdrnq:/#

root@deploy-web-dep-747497b79d-gdrnq:/# cd  /usr/share/nginx/html/

root@deploy-web-dep-747497b79d-gdrnq:/usr/share/nginx/html#  echo "<h1> ClusterIP Service Web-dep-2 </h1>"  >  index.html

root@deploy-web-dep-747497b79d-gdrnq:/usr/share/nginx/html# cat index.html
<h1> ClusterIP Service Web-dep-2 </h1>
```

#### 세번째 Pod 수정

```bash
[root@k8s-master ~]# kubectl  exec  deploy-web-dep-747497b79d-lnw72  -it  -- /bin/bash
root@deploy-web-dep-747497b79d-lnw72:/#

root@deploy-web-dep-747497b79d-lnw72:/# cd  /usr/share/nginx/html/

root@deploy-web-dep-747497b79d-lnw72:/usr/share/nginx/html#  echo "<h1> ClusterIP Service Web-dep-3 </h1>"  >  index.html

root@deploy-web-dep-747497b79d-lnw72:/usr/share/nginx/html# cat index.html
<h1> ClusterIP Service Web-dep-3 </h1>
```

#### 로드밸런싱 확인

```bash
[root@k8s-master ~]# curl  http://10.100.100.100
<h1> ClusterIP Service Web-dep-3 </h1>

[root@k8s-master ~]# curl  http://10.100.100.100
<h1> ClusterIP Service Web-dep-2 </h1>

[root@k8s-master ~]# curl  http://10.100.100.100
<h1> ClusterIP Service Web-dep-3 </h1>

[root@k8s-master ~]# curl  http://10.100.100.100
<h1> ClusterIP Service Web-dep-1 </h1>

[root@k8s-master ~]# curl  http://10.100.100.100
<h1> ClusterIP Service Web-dep-2 </h1>
```

### Pod 스케일 업 후 Endpoint 자동 갱신 확인

```bash
[root@k8s-master ~]# kubectl  scale  deployment  deploy-web-dep  --replicas=4
deployment.apps/deploy-web-dep scaled
```

```bash
[root@k8s-master ~]# kubectl  describe  service  clusterip-service
Name:                     clusterip-service
Namespace:                default
Labels:                   <none>
Annotations:              <none>
Selector:                 app=web-dep
Type:                     ClusterIP
IP Family Policy:         SingleStack
IP Families:              IPv4
IP:                       10.100.100.100
IPs:                      10.100.100.100
Port:                     <unset>  80/TCP
TargetPort:               80/TCP
Endpoints:                10.244.2.94:80,10.244.1.18:80,10.244.2.95:80 + 1 more...
Session Affinity:         None
Internal Traffic Policy:  Cluster
Events:                   <none>
```

```bash
[root@k8s-master ~]# kubectl  get  pods
NAME                              READY   STATUS    RESTARTS   AGE
deploy-web-dep-747497b79d-bptqs   1/1     Running   0          44m
deploy-web-dep-747497b79d-gdrnq   1/1     Running   0          44m
deploy-web-dep-747497b79d-lnw72   1/1     Running   0          44m
deploy-web-dep-747497b79d-ndr9b   1/1     Running   0          7s
```

```bash
[root@k8s-master ~]# curl  http://10.100.100.100
<h1> ClusterIP Service Web-dep-1 </h1>

[root@k8s-master ~]# curl  http://10.100.100.100
<h1> ClusterIP Service Web-dep-2 </h1>

[root@k8s-master ~]# curl  http://10.100.100.100
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
...
</html>

[root@k8s-master ~]# curl  http://10.100.100.100
<h1> ClusterIP Service Web-dep-3 </h1>
```

### 멀티 Label을 이용한 Selector 예시

```bash
[root@k8s-master ~]# vi deploy-nginx.yaml
```
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deploy-web-dep
spec:
  replicas: 2
  selector:
    matchLabels:
      app: web-dep
  template:
    metadata:
      name: nginx-pod
      labels:
        app: web-dep
        service: nginx
    spec:
      containers:
      - name: nginx-container
        image: nginx:1.31
```

```bash
[root@k8s-master ~]# vi clusterip-nginx.yaml
```
```yaml
apiVersion: v1
kind: Service
metadata:
  name: clusterip-service
spec:
  type: ClusterIP
  clusterIP: 10.100.100.100
  selector:
    service: nginx
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
```

```bash
[root@k8s-master ~]# kubectl  apply  -f  deploy-nginx.yaml
deployment.apps/deploy-web-dep created

[root@k8s-master ~]# kubectl  apply  -f  clusterip-nginx.yaml
service/clusterip-service created

[root@k8s-master ~]# kubectl  get  deployments  deploy-web-dep
NAME             READY   UP-TO-DATE   AVAILABLE   AGE
deploy-web-dep   2/2     2            2           5m30s

[root@k8s-master ~]# kubectl  get  pods
NAME                              READY   STATUS    RESTARTS   AGE
deploy-web-dep-578859465c-rfghn   1/1     Running   0          3m20s
deploy-web-dep-578859465c-vfblj   1/1     Running   0          3m20s

[root@k8s-master ~]# kubectl  describe  service  clusterip-service
Name:                     clusterip-service
Namespace:                default
Labels:                   <none>
Annotations:              <none>
Selector:                 service=nginx
Type:                     ClusterIP
IP Family Policy:         SingleStack
IP Families:              IPv4
IP:                       10.100.100.100
IPs:                      10.100.100.100
Port:                     <unset>  80/TCP
TargetPort:               80/TCP
Endpoints:                10.244.2.96:80,10.244.1.21:80
Session Affinity:         None
Internal Traffic Policy:  Cluster
Events:                   <none>
```

### Service에 포함되는 새로운 Pod 생성

```bash
[root@k8s-master ~]# vi svc-in-pod.yaml
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: svc-in-pod
  labels:
    service: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.31
    ports:
    - containerPort: 80
```

```bash
[root@k8s-master ~]# kubectl  apply  -f   svc-in-pod.yaml
pod/svc-in-pod created
```

```bash
# Deployment에 의해 관리되는 Pod는 2개
[root@k8s-master ~]# kubectl  get  deployments  deploy-web-dep
NAME             READY   UP-TO-DATE   AVAILABLE   AGE
deploy-web-dep   2/2     2            2           8m11s

# 총 Pod는 3개
[root@k8s-master ~]# kubectl  get  pods
NAME                              READY   STATUS    RESTARTS   AGE
deploy-web-dep-578859465c-rfghn   1/1     Running   0          8m6s
deploy-web-dep-578859465c-vfblj   1/1     Running   0          8m6s
svc-in-pod                        1/1     Running   0          8s

# ClusterIP Service에 의해 서비스되는 Pod는 3개
[root@k8s-master ~]# kubectl  describe  service  clusterip-service
Name:                     clusterip-service
Namespace:                default
Labels:                   <none>
Annotations:              <none>
Selector:                 service=nginx
Type:                     ClusterIP
IP Family Policy:         SingleStack
IP Families:              IPv4
IP:                       10.100.100.100
IPs:                      10.100.100.100
Port:                     <unset>  80/TCP
TargetPort:               80/TCP
Endpoints:                10.244.2.96:80,10.244.1.21:80,10.244.2.97:80
Session Affinity:         None
Internal Traffic Policy:  Cluster
Events:                   <none>
```

### Service에 포함되지 않는 새로운 Pod 생성

```bash
[root@k8s-master ~]# vi svc-out-pod.yaml
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: svc-out-pod
  labels:
    service: apache
spec:
  containers:
  - name: nginx
    image: nginx:1.31
    ports:
    - containerPort: 80
```

```bash
[root@k8s-master ~]# kubectl apply  -f  svc-out-pod.yaml
pod/svc-out-pod created

[root@k8s-master ~]# kubectl  get  deployments  deploy-web-dep
NAME             READY   UP-TO-DATE   AVAILABLE   AGE
deploy-web-dep   2/2     2            2           35m

[root@k8s-master ~]# kubectl  get  pods
NAME                              READY   STATUS    RESTARTS   AGE
deploy-web-dep-578859465c-rfghn   1/1     Running   0          35m
deploy-web-dep-578859465c-vfblj   1/1     Running   0          35m
svc-in-pod                        1/1     Running   0          27m
svc-out-pod                       1/1     Running   0          10s

[root@k8s-master ~]# kubectl  describe  service  clusterip-service
Name:                     clusterip-service
...
Endpoints:                10.244.2.96:80,10.244.1.21:80,10.244.2.97:80
...
```

svc-out-pod는 `service: apache` 라벨이므로 Endpoint에 포함되지 않는다.

### 특정 Pod에 Label 추가하여 Service에 포함시키기

```bash
[root@k8s-master ~]# kubectl  label  pod  svc-out-pod  service=nginx  --overwrite
pod/svc-out-pod labeled

[root@k8s-master ~]# kubectl  get  pods  --show-labels
NAME                              READY   STATUS    RESTARTS   AGE    LABELS
deploy-web-dep-578859465c-rfghn   1/1     Running   0          42m    app=web-dep,pod-template-hash=578859465c,service=nginx
deploy-web-dep-578859465c-vfblj   1/1     Running   0          42m    app=web-dep,pod-template-hash=578859465c,service=nginx
svc-in-pod                        1/1     Running   0          35m    service=nginx
svc-out-pod                       1/1     Running   0          8m7s   service=nginx

[root@k8s-master ~]# kubectl  describe  service  clusterip-service
Name:                     clusterip-service
...
Endpoints:                10.244.2.96:80,10.244.1.21:80,10.244.2.97:80 + 1 more...
...
```

### 리소스 정리

```bash
[root@k8s-master ~]# kubectl  delete  deployments  deploy-web-dep
deployment.apps "deploy-web-dep" deleted from default namespace

[root@k8s-master ~]# kubectl  delete  service  clusterip-service
service "clusterip-service" deleted from default namespace

[root@k8s-master ~]# kubectl  delete  pod svc-in-pod
pod "svc-in-pod" deleted from default namespace

[root@k8s-master ~]# kubectl  delete  pod svc-out-pod
pod "svc-out-pod" deleted from default namespace
```

> **정리**: ClusterIP 실습을 통해 selector와 label 일치 여부에 따라 Endpoints가 자동으로 갱신되는 것을 확인했다. 이어서 외부 접근이 가능한 **NodePort** 타입을 실습한다.

---

## NodePort 실습

### 개요
- 모든 노드를 대상으로 외부 접속 가능한 포트를 예약
- Default NodePort 범위 : `30000–32767`
- ClusterIP를 생성 후 NodePort를 예약
- 외부에서 port번호를 사용해서 접속 가능

### Deployment 생성

```bash
[root@k8s-master ~]# vi deploy-nginx.yaml
```
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deploy-web-dep
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web-dep
  template:
    metadata:
      name: nginx-pod
      labels:
        app: web-dep
    spec:
      containers:
      - name: nginx-container
        image: nginx:1.31
```

### NodePort Service YAML

```bash
[root@k8s-master ~]# vi nodeport-nginx.yaml
```
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nodeport-service
spec:
  type: NodePort
  clusterIP: 10.100.100.200
  selector:
    app: web-dep
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
    nodePort: 30100
```

```bash
[root@k8s-master ~]# kubectl  apply  -f  deploy-nginx.yaml
deployment.apps/deploy-web-dep created

[root@k8s-master ~]# kubectl  apply  -f  nodeport-nginx.yaml
service/nodeport-service created

[root@k8s-master ~]# kubectl  get  deployments deploy-web-dep
NAME             READY   UP-TO-DATE   AVAILABLE   AGE
deploy-web-dep   3/3     3            3           75s

[root@k8s-master ~]# kubectl  get  pods
NAME                              READY   STATUS    RESTARTS   AGE
deploy-web-dep-578859465c-h88cn   1/1     Running   0          97s
deploy-web-dep-578859465c-ppkbj   1/1     Running   0          25s
deploy-web-dep-578859465c-xfbtl   1/1     Running   0          97s

[root@k8s-master ~]# kubectl  get  pods  -o  wide
NAME                              READY   STATUS    RESTARTS   AGE     IP             NODE           NOMINATED NODE   READINESS GATES
deploy-web-dep-578859465c-h88cn   1/1     Running   0          5m6s    10.244.2.2   k8s-worker2   <none>            <none>
deploy-web-dep-578859465c-ppkbj   1/1     Running   0          3m4s    10.244.1.3   k8s-worker1   <none>            <none>
deploy-web-dep-578859465c-xfbtl   1/1     Running   0          5m6s    10.244.1.2   k8s-worker1   <none>            <none>

[root@k8s-master ~]# kubectl  get  service
NAME               TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
kubernetes         ClusterIP   10.96.0.1        <none>        443/TCP          9d
nodeport-service   NodePort    10.100.100.200   <none>        80:30100/TCP     2m6s

[root@k8s-master ~]# kubectl  describe   service  nodeport-service
Name:                   nodeport-service
Namespace:              default
Labels:                 <none>
Annotations:            <none>
Selector:               app=web-dep
Type:                   NodePort
IP Family Policy:       SingleStack
IP Families:            IPv4
IP:                     10.100.100.200
IPs:                    10.100.100.200
Port:                   <unset>  80/TCP
TargetPort:             80/TCP
NodePort:               <unset>  30100/TCP
Endpoints:              10.244.1.2:80,10.244.2.2:80,10.244.1.3:80
Session Affinity:       None
External Traffic Policy: Cluster
Internal Traffic Policy: Cluster
Events:                 <none>

[root@k8s-master ~]# kubectl  get  nodes -o  wide
NAME          STATUS   ROLES         AGE   VERSION   INTERNAL-IP       EXTERNAL-IP   OS-IMAGE                       KERNEL-VERSION
k8s-master    Ready    control-plane 9d    v1.35.7   192.168.10.100    <none>        Rocky Linux 9.8 (Blue Onyx)   5.14.0-687.36.1.el9_8.x86_64
k8s-worker1   Ready    <none>        9d    v1.35.7   192.168.10.101    <none>        Rocky Linux 9.8 (Blue Onyx)   5.14.0-687.36.1.el9_8.x86_64
k8s-worker2   Ready    <none>        9d    v1.35.7   192.168.10.102    <none>        Rocky Linux 9.8 (Blue Onyx)   5.14.0-687.36.1.el9_8.x86_64
```

> **정리**: NodePort는 각 노드에 동일한 포트를 열어 외부 접근을 허용한다. 다음은 클라우드 환경에서 사용하는 **LoadBalancer** 타입 실습이다.

---

## LoadBalancer 실습

### 개요
- Public 클라우드(AWS, Azure, GCP 등)에서 운영 가능
- LoadBalancer를 자동으로 구성 요청
- NodePort를 예약 후 해당 nodeport로 외부 접근을 허용
- 현재는 구성해도 사용할 수는 없다.
- AWS, Azure, GCP 등과 연계해서 사용해야 한다.

### LoadBalancer Service YAML

```bash
[root@k8s-master ~]# vi loadbalancer-nginx.yaml
```
```yaml
apiVersion: v1
kind: Service
metadata:
  name: loadbalancer-service
spec:
  type: LoadBalancer
  selector:
    app: web-dep
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
```

```bash
[root@k8s-master ~]# kubectl  apply  -f  loadbalancer-nginx.yaml
service/loadbalancer-service created

[root@k8s-master ~]# kubectl  get  service  loadbalancer-service
NAME                   TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
loadbalancer-service   LoadBalancer   10.111.152.137   <pending>     80:31378/TCP   25s
```

- `EXTERNAL-IP`가 `<pending>` 상태 : 클라우드 환경이 아니면 외부 IP가 할당되지 않는다.

```bash
[root@k8s-master ~]# kubectl  delete   service  loadbalancer-service
service "loadbalancer-service" deleted from default namespace
```

> **정리**: LoadBalancer는 클라우드 환경이 아니면 `EXTERNAL-IP`가 `<pending>` 상태로 유지된다. 다음은 클러스터 외부 서비스를 내부 이름으로 매핑하는 **ExternalName** 실습이다.

---

## ExternalName 실습

### 개요
- 외부 DNS 이름을 Service 이름으로 매핑한다
- 셀렉터(selector)를 사용하지 않는다 (Pod를 선택하지 않음)
- 클러스터 내부에서 외부 서비스를 내부 서비스처럼 사용하기 위한 목적

### ExternalName Service + 테스트 Pod YAML

```bash
[root@k8s-master ~]# vi extname-service.yaml
```
```yaml
apiVersion: v1
kind: Service
metadata:
  name: web
spec:
  type: ExternalName
  externalName: example.com

---
apiVersion: v1
kind: Pod
metadata:
  name: tester
spec:
  containers:
  - name: test
    image: busybox:1.36
    command: ["sh","-c","sleep 3600"]
```

- service 이름인 `web`이 쿠버네티스 Core DNS에 저장 = `web.default.svc.cluster.local`

```bash
[root@k8s-master ~]# kubectl  apply  -f  extname-service.yaml  --dry-run=client
service/web created (dry run)
pod/tester created (dry run)
```

### DNS 조회 테스트

```bash
[root@k8s-master ~]# kubectl  exec  -it  tester  -- nslookup web
Server:   10.96.0.10
Address:  10.96.0.10:53

** server can't find web.cluster.local: NXDOMAIN

** server can't find web.cluster.local: NXDOMAIN

** server can't find web.svc.cluster.local: NXDOMAIN

** server can't find web.svc.cluster.local: NXDOMAIN

web.default.svc.cluster.local   canonical name = example.com
Name:   example.com
Address: 104.20.23.154
Name:   example.com
Address: 172.66.147.243

web.default.svc.cluster.local   canonical name = example.com
Name:   example.com
Address: 2606:4700:10::6814:179a
Name:   example.com
Address: 2606:4700:10::ac42:93f3

command terminated with exit code 1
```

> **정리**: `nslookup` 결과 Service 이름이 `CNAME`으로 외부 DNS에 연결됨을 확인했다. 아래는 동일한 개념을 다른 이름으로 적용해보는 예제이다.

---

## EX) ExternalName Service를 생성하여 naver라는 이름으로 google.com에 접근

- Service 이름 : `naver`
- Service Type : `ExternalName`
- 외부 DNS : `google.com`
- 테스트 Pod에서 DNS 및 접속 확인

```bash
[root@k8s-master ~]# vi externalname-google.yaml
```
```yaml
apiVersion: v1
kind: Service
metadata:
  name: naver              # Kubernetes 내부에서 사용할 Service 이름

spec:
  type: ExternalName       # 외부 DNS 이름과 연결하는 Service
  externalName: google.com # 실제로 연결할 외부 DNS 이름
```

```bash
[root@k8s-master ~]# kubectl apply  -f  externalname-google.yaml  --dry-run=client
service/naver created (dry run)

[root@k8s-master ~]# kubectl apply  -f  externalname-google.yaml
service/naver created

[root@k8s-master ~]# kubectl  get  service
NAME        TYPE          CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes  ClusterIP     10.96.0.1    <none>        443/TCP   9d
naver       ExternalName  <none>       google.com    <none>    16s

[root@k8s-master ~]# kubectl  run  dnspod  --image=busybox:1.36  --command  -- sleep 3600

[root@k8s-master ~]# kubectl   get  pods
NAME    READY   STATUS    RESTARTS   AGE
dnspod  1/1     Running   0          93s
```

### dnspod로 접속하여 DNS 확인

```bash
[root@k8s-master ~]# kubectl  exec  -it  dnspod  -- sh
/ #
/ # nslookup  naver
Server:    10.96.0.10
Address:   10.96.0.10:53

** server can't find naver.cluster.local: NXDOMAIN

** server can't find naver.svc.cluster.local: NXDOMAIN

** server can't find naver.cluster.local: NXDOMAIN

** server can't find naver.svc.cluster.local: NXDOMAIN

naver.default.svc.cluster.local canonical name = google.com
Name:   google.com
Address: 2404:6800:400b:c00c::64
Name:   google.com
Address: 2404:6800:400b:c00c::65
Name:   google.com
Address: 2404:6800:400b:c00c::8b
Name:   google.com
Address: 2404:6800:400b:c00c::71

naver.default.svc.cluster.local canonical name = google.com
Name:   google.com
Address: 142.250.21.101
Name:   google.com
Address: 142.250.21.138
Name:   google.com
Address: 142.250.21.102
Name:   google.com
Address: 142.250.21.113
Name:   google.com
Address: 142.250.21.139
Name:   google.com
Address: 142.250.21.100
```

### curl이 되는 이미지로 접속 확인

```bash
[root@k8s-master ~]# kubectl  run curlpod  --image=curlimages/curl:latest  --rm  -it  --restart=Never  -- curl  http://naver
<!DOCTYPE html>
<html lang=en>
  <meta charset=utf-8>
  <meta name=viewport content="initial-scale=1, minimum-scale=1, width=device-width">
  <title>Error 404 (Not Found)!!1</title>
  <style>
    *{margin:0;padding:0}html,code{font:15px/22px arial,sans-serif}...
  </style>
  <p><b>404.</b> <ins>That's an error.</ins>
  <p>The requested URL <code>/</code> was not found on this server.  <ins>That's all we know.</ins>
pod "curlpod" deleted from default namespace
```

```bash
# 정해진 기능을 수행 후 --rm 명령어에 의해 pod 삭제
[root@k8s-master ~]# kubectl  get pods
No resources found in default namespace.
```

> **정리**: ExternalName Service는 Pod를 선택하지 않고 DNS CNAME만 연결하므로, 클러스터 내부 코드가 외부 서비스 주소 변경에 영향받지 않도록 하는 데 유용하다. 다음은 개별 Pod IP를 직접 노출하는 **Headless Service** 실습이다.

---

## Headless Service 실습

### 개요

- **Headless Service**는 서비스 IP(ClusterIP)를 아예 만들지 않는 Service다.
- 일반 Service는 하나의 가상 IP(ClusterIP)를 만들고 그 IP 뒤에서 여러 Pod로 트래픽을 분산한다.
- Headless Service는 ClusterIP를 만들지 않는다. 대신 Pod 각각의 IP를 그대로 노출한다.
- 즉, 로드밸런서를 없애고 Pod 하나하나를 직접 보이게 만드는 Service이다.

### 일반 Service vs Headless Service 차이

| 항목 | 일반 Service (ClusterIP) | Headless Service |
|------|--------------------------|------------------|
| Service IP | 있음 (예: 10.96.10.50) | 없음 (clusterIP: None) |
| DNS 조회 결과 | Service IP 1개 | Pod IP 목록 여러 개 |
| kube-proxy | 트래픽을 Pod로 분산 | 개입하지 않음 |
| 클라이언트 | Pod 존재를 모름 | Pod를 직접 선택 |

### Headless Service YAML 구조

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-headless
spec:
  clusterIP: None
  selector:
    app: db
  ports:
  - port: 3306
    targetPort: 3306
```

이렇게 설정하면 Service IP는 생성되지 않고 DNS만 관리하는 Service가 된다.

### Headless Service의 DNS 동작 방식

**일반 Service DNS 조회:**
```
myservice.default.svc.cluster.local
→ 10.96.10.50
```

**Headless Service DNS 조회:**
```
my-headless.default.svc.cluster.local
→ 10.244.1.10
→ 10.244.2.15
→ 10.244.3.8
```

즉, 하나의 IP가 아니라 Pod IP 리스트가 그대로 반환된다.
이 리스트를 애플리케이션이 직접 사용하거나 클라이언트가 직접 선택한다.

### Headless Service가 필요한 이유

일반적인 웹 서비스에서는 어느 Pod로 가든 상관없다.

하지만 이런 경우는 다르다:
- DB 클러스터 → Primary / Replica 구분 필요 → 특정 Pod로 반드시 접속해야 함
- Stateful 서비스 → Pod마다 고유 ID 필요 → Pod 순서와 이름이 중요

이런 시스템은 "쿠버네티스가 분산하지 말고 내가 직접 관리할게"라는 전제에서 동작한다.

- **DB 읽기/쓰기용** : Primary, db-1, db-2는 읽기 전용 Replica로 구분해서 사용할 때 각 DB Pod를 고정된 이름으로 직접 지정할 수 있다.
- **분산 저장소 구성** : 각 저장소 노드가 서로 다른 데이터를 담당하거나 고유한 역할을 가지는 경우, 특정 노드를 정확하게 지정해서 접근할 수 있다.

### 실습 - ClusterIP + Headless 비교

```bash
[root@k8s-master ~]# vi deploy-nginx.yaml
```
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-dep
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web-dep
  template:
    metadata:
      name: nginx-pod
      labels:
        app: web-dep
    spec:
      containers:
      - name: nginx-container
        image: nginx:1.31
```

```bash
[root@k8s-master ~]# kubectl  apply  -f  deploy-nginx.yaml
deployment.apps/deploy-web-dep created

[root@k8s-master ~]# kubectl  get  deployments  deploy-web-dep
NAME             READY   UP-TO-DATE   AVAILABLE   AGE
deploy-web-dep   3/3     3            3           23s

[root@k8s-master ~]# kubectl  get  pods
NAME                              READY   STATUS    RESTARTS   AGE
deploy-web-dep-578859465c-chqzs   1/1     Running   0          26s
deploy-web-dep-578859465c-qj479   1/1     Running   0          45s
deploy-web-dep-578859465c-wgtj2   1/1     Running   0          45s

[root@k8s-master ~]# kubectl  get  pods  -o  wide
NAME                              READY   STATUS    RESTARTS   AGE   IP             NODE           NOMINATED NODE   READINESS GATES
deploy-web-dep-578859465c-chqzs   1/1     Running   0          41s   10.244.1.7   k8s-worker1   <none>           <none>
deploy-web-dep-578859465c-qj479   1/1     Running   0          60s   10.244.1.6   k8s-worker1   <none>           <none>
deploy-web-dep-578859465c-wgtj2   1/1     Running   0          60s   10.244.2.6   k8s-worker2   <none>           <none>
```

#### Service (ClusterIP)

```bash
[root@k8s-master ~]# vi clusterip-nginx.yaml
```
```yaml
apiVersion: v1
kind: Service
metadata:
  name: clusterip-service
spec:
  type: ClusterIP
  clusterIP: 10.100.100.100
  selector:
    app: web-dep
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
```

#### Service (Headless)

```bash
[root@k8s-master ~]# vi headless-nginx.yaml
```
```yaml
apiVersion: v1
kind: Service
metadata:
  name: headless-service
spec:
  type: ClusterIP
  clusterIP: None
  selector:
    app: web-dep
  ports:
  - protocol: TCP
    port: 80
```

```bash
[root@k8s-master ~]# kubectl  apply  -f   clusterip-nginx.yaml
service/clusterip-service created

[root@k8s-master ~]# kubectl  apply  -f   headless-nginx.yaml
service/headless-service created

[root@k8s-master ~]# kubectl  get  service
NAME               TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
clusterip-service  ClusterIP   10.100.100.100  <none>        80/TCP     23s
headless-service   ClusterIP   None            <none>        80/TCP     20s
kubernetes         ClusterIP   10.96.0.1       <none>        443/TCP    9d
```

Headless Service는 CLUSTER-IP가 할당되지 않는다.

```bash
[root@k8s-master ~]# kubectl  describe  service  headless-service
Name:                     headless-service
Namespace:                default
Labels:                   <none>
Annotations:              <none>
Selector:                 app=web-dep
Type:                     ClusterIP
IP Family Policy:         SingleStack
IP Families:              IPv4
IP:                       None
IPs:                      None
Port:                     <unset>  80/TCP
TargetPort:               80/TCP
Endpoints:                10.244.2.6:80,10.244.1.7:80,10.244.1.6:80
Session Affinity:         None
Internal Traffic Policy:  Cluster
Events:                   <none>
```

### kube-system 네임스페이스 DNS 확인

```bash
[root@k8s-master ~]# kubectl  get  pods  --all-namespaces   -o wide
NAMESPACE      NAME                                  READY   STATUS    RESTARTS       AGE     IP               NODE          NOMINATED NODE   READINESS GATES
default        deploy-web-dep-578859465c-chqzs       1/1     Running   0              19m     10.244.1.7       k8s-worker1   <none>           <none>
default        deploy-web-dep-578859465c-qj479       1/1     Running   0              19m     10.244.1.6       k8s-worker1   <none>           <none>
default        deploy-web-dep-578859465c-wgtj2       1/1     Running   0              19m     10.244.2.6       k8s-worker2   <none>           <none>
default        testcentos                            1/1     Running   0              4m52s   10.244.2.7       k8s-worker2   <none>           <none>
kube-flannel   kube-flannel-ds-7hlrp                 1/1     Running   8 (153m ago)   9d      192.168.10.100   k8s-master    <none>           <none>
kube-flannel   kube-flannel-ds-f9wcz                 1/1     Running   7 (153m ago)   9d      192.168.10.101   k8s-worker1   <none>           <none>
kube-flannel   kube-flannel-ds-z4fbv                 1/1     Running   8 (153m ago)   9d      192.168.10.102   k8s-worker2   <none>           <none>
kube-system    coredns-7d764666f9-2dwdd              1/1     Running   8 (153m ago)   9d      10.244.0.19      k8s-master    <none>           <none>
kube-system    coredns-7d764666f9-klkhq              1/1     Running   8 (153m ago)   9d      10.244.0.18      k8s-master    <none>           <none>
kube-system    etcd-k8s-master                       1/1     Running   9 (153m ago)   9d      192.168.10.100   k8s-master    <none>           <none>
kube-system    kube-apiserver-k8s-master             1/1     Running   9 (153m ago)   9d      192.168.10.100   k8s-master    <none>           <none>
kube-system    kube-controller-manager-k8s-master    1/1     Running   9 (153m ago)   9d      192.168.10.100   k8s-master    <none>           <none>
kube-system    kube-proxy-bxjvw                      1/1     Running   7 (153m ago)   9d      192.168.10.102   k8s-worker2   <none>           <none>
kube-system    kube-proxy-gbr4f                      1/1     Running   7 (153m ago)   9d      192.168.10.101   k8s-worker1   <none>           <none>
kube-system    kube-proxy-rwhkc                      1/1     Running   8 (153m ago)   9d      192.168.10.100   k8s-master    <none>           <none>
kube-system    kube-scheduler-k8s-master             1/1     Running   9 (153m ago)   9d      192.168.10.100   k8s-master    <none>           <none>

[root@k8s-master ~]# kubectl get svc -n kube-system
NAME       TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)                  AGE
kube-dns   ClusterIP   10.96.0.10    <none>        53/UDP,53/TCP,9153/TCP   9d
```

### Pod 내부 resolv.conf 확인

```bash
# centos pod를 생성하고 바로 컨테이너로 접속
[root@k8s-master ~]# kubectl  run  testcentos  -it  --image=centos:8  /bin/bash
If you don't see a command prompt, try pressing enter.
[root@testcentos /]#

[root@testcentos /]# cat  /etc/resolv.conf
search default.svc.cluster.local svc.cluster.local cluster.local
nameserver 10.96.0.10
options ndots:5
```

### DNS 테스트 Pod로 Headless Service 조회

```bash
# dns를 테스트하기 위한 pod 생성 (자동 삭제)
[root@k8s-master ~]# kubectl  run  dns-test  --image=busybox:1.36  -it  --rm  --restart=Never  -- sh
/ #
/ # nslookup  headless-service
Server:          10.96.0.10
Address:        10.96.0.10:53

** server can't find headless-service.cluster.local: NXDOMAIN
** server can't find headless-service.cluster.local: NXDOMAIN
** server can't find headless-service.svc.cluster.local: NXDOMAIN

Name:   headless-service.default.svc.cluster.local
Address: 10.244.2.6
Name:   headless-service.default.svc.cluster.local
Address: 10.244.1.6
Name:   headless-service.default.svc.cluster.local
Address: 10.244.1.7

** server can't find headless-service.svc.cluster.local: NXDOMAIN
```

```bash
[root@k8s-master ~]# kubectl  get  pods  -o  wide
NAME                              READY   STATUS    RESTARTS   AGE   IP             NODE           NOMINATED NODE   READINESS GATES
deploy-web-dep-578859465c-chqzs   1/1     Running   0          41s   10.244.1.7   k8s-worker1   <none>           <none>
deploy-web-dep-578859465c-qj479   1/1     Running   0          60s   10.244.1.6   k8s-worker1   <none>           <none>
deploy-web-dep-578859465c-wgtj2   1/1     Running   0          60s   10.244.2.6   k8s-worker2   <none>           <none>
```

### 리소스 정리

```bash
[root@k8s-master ~]# kubectl  delete  deployments  deploy-web-dep
deployment.apps "deploy-web-dep" deleted from default namespace

[root@k8s-master ~]# kubectl  delete service  clusterip-service
service "clusterip-service" deleted from default namespace

[root@k8s-master ~]# kubectl  delete service  headless-service
service "headless-service" deleted from default namespace
```

> **정리**: Headless Service는 DNS 조회 시 ClusterIP 대신 Pod IP 목록을 그대로 반환한다는 것을 확인했다. 마지막으로 **StatefulSet**과 결합해 각 Pod가 고정된 이름으로 조회되는지 실습한다.

---

## EX) Headless Service와 StatefulSet을 연동해서 Pod 3개를 생성하고, 각 Pod가 고정된 이름으로 DNS 조회되는지 확인

- StatefulSet 이름 : `web-sts`
- Pod 개수 : 3개
- 이미지 : `nginx:1.31`
- Service 이름 : `web-headless`

각 Pod는 다음 이름으로 생성되어야 한다:
- `web-sts-0`
- `web-sts-1`
- `web-sts-2`

각 Pod를 다음 DNS 이름으로 직접 조회할 수 있어야 한다:
- `web-sts-0.web-headless`
- `web-sts-1.web-headless`
- `web-sts-2.web-headless`

### STEP 1) StatefulSet 생성

```bash
[root@k8s-master ~]# vi statefulset-pod.yaml
```
```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web-sts
spec:
  serviceName: web-headless
  replicas: 3
  selector:
    matchLabels:
      app: web-sts
  template:
    metadata:
      labels:
        app: web-sts
    spec:
      containers:
      - name: nginx
        image: nginx:1.31
        ports:
        - containerPort: 80
```

```bash
[root@k8s-master ~]# kubectl apply -f statefulset-pod.yaml
statefulset.apps/web-sts created
```

### STEP 2) Headless Service 생성

```bash
[root@k8s-master ~]# vi headless-sts.yaml
```
```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-headless
spec:
  clusterIP: None
  selector:
    app: web-sts
  ports:
  - port: 80
    targetPort: 80
```

```bash
[root@k8s-master ~]# kubectl apply -f headless-sts.yaml
service/web-headless created
```

### STEP 3) Pod, Headless Service 확인

```bash
[root@k8s-master ~]# kubectl  get  statefulsets  web-sts
NAME      READY   AGE
web-sts   3/3     2m16s

[root@k8s-master ~]# kubectl  get  pods
NAME        READY   STATUS    RESTARTS   AGE
web-sts-0   1/1     Running   0          54s
web-sts-1   1/1     Running   0          52s
web-sts-2   1/1     Running   0          51s

[root@k8s-master ~]# kubectl  get  service  web-headless
NAME           TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
web-headless   ClusterIP   None         <none>        80/TCP    70s

[root@k8s-master ~]# kubectl  describe  service web-headless
Name:                     web-headless
Namespace:                default
Labels:                   <none>
Annotations:              <none>
Selector:                 app=web-sts
Type:                     ClusterIP
IP Family Policy:         SingleStack
IP Families:              IPv4
IP:                       None
IPs:                      None
Port:                     <unset>  80/TCP
TargetPort:               80/TCP
Endpoints:                10.244.1.9:80,10.244.2.9:80,10.244.1.10:80
Session Affinity:         None
Internal Traffic Policy:  Cluster
Events:                   <none>

[root@k8s-master ~]# kubectl  get  pods  -o  wide
NAME        READY   STATUS    RESTARTS   AGE    IP              NODE          NOMINATED NODE   READINESS GATES
web-sts-0   1/1     Running   0          4m7s   10.244.1.9    k8s-worker1   <none>            <none>
web-sts-1   1/1     Running   0          4m5s   10.244.2.9    k8s-worker2   <none>            <none>
web-sts-2   1/1     Running   0          4m4s   10.244.1.10   k8s-worker1   <none>            <none>
```

### STEP 4) Service 이름으로 DNS 조회

```bash
[root@k8s-master ~]# kubectl  run  dnspod  --image=busybox:1.36  --rm  -it  --restart=Never  -- nslookup  web-headless
Server:        10.96.0.10
Address:       10.96.0.10:53

** server can't find web-headless.cluster.local: NXDOMAIN

** server can't find web-headless.cluster.local: NXDOMAIN

** server can't find web-headless.svc.cluster.local: NXDOMAIN

Name:   web-headless.default.svc.cluster.local
Address: 10.244.2.9
Name:   web-headless.default.svc.cluster.local
Address: 10.244.1.9
Name:   web-headless.default.svc.cluster.local
Address: 10.244.1.10

** server can't find web-headless.svc.cluster.local: NXDOMAIN


pod "dnspod" deleted from default namespace
```

Headless Service 이름으로 해당 Service에 포함된 모든 Pod의 IP 주소를 리턴받을 수 있다.

### STEP 5) 특정 Pod를 이름으로 직접 조회

> busybox는 몇 가지 명령어만 실행할 수 있는 작은 이미지이므로 DNS 기능의 한계로 전체 Domain 주소를 입력해야 한다.

```bash
[root@k8s-master ~]# kubectl  run  dnspod  --image=busybox:1.36  --rm  -it  --restart=Never  -- nslookup  web-sts-0.web-headless.default.svc.cluster.local
Server:         10.96.0.10
Address:        10.96.0.10:53

Name:   web-sts-0.web-headless.default.svc.cluster.local
Address: 10.244.1.9
```

```bash
[root@k8s-master ~]# kubectl  run  dnspod  --image=busybox:1.36  --rm  -it  --restart=Never  -- nslookup  web-sts-1.web-headless.default.svc.cluster.local
Server:         10.96.0.10
Address:        10.96.0.10:53

Name:   web-sts-1.web-headless.default.svc.cluster.local
Address: 10.244.2.9
```

```bash
[root@k8s-master ~]# kubectl  run  dnspod  --image=busybox:1.36  --rm  -it  --restart=Never  -- nslookup  web-sts-2.web-headless.default.svc.cluster.local
Server:         10.96.0.10
Address:        10.96.0.10:53

Name:   web-sts-2.web-headless.default.svc.cluster.local
Address: 10.244.1.10
```

### 현재 StatefulSet으로 만들어진 Pod 내에서 DNS 확인

```bash
[root@k8s-master ~]# kubectl  get  pods
NAME        READY   STATUS    RESTARTS   AGE
web-sts-0   1/1     Running   0          54s
web-sts-1   1/1     Running   0          52s
web-sts-2   1/1     Running   0          51s

[root@k8s-master ~]# kubectl  exec  -it  web-sts-0  -- /bin/bash
root@web-sts-0:/#

root@web-sts-0:/# getent  hosts  web-sts-1.web-headless
10.244.2.9      web-sts-1.web-headless.default.svc.cluster.local

root@web-sts-0:/# getent  hosts  web-sts-2.web-headless
10.244.1.10     web-sts-2.web-headless.default.svc.cluster.local

[root@k8s-master ~]# kubectl  get  pods  -o  wide
NAME        READY   STATUS    RESTARTS   AGE    IP              NODE          NOMINATED NODE   READINESS GATES
web-sts-0   1/1     Running   0          4m7s   10.244.1.9    k8s-worker1   <none>            <none>
web-sts-1   1/1     Running   0          4m5s   10.244.2.9    k8s-worker2   <none>            <none>
web-sts-2   1/1     Running   0          4m4s   10.244.1.10   k8s-worker1   <none>            <none>
```

> **정리**: StatefulSet과 Headless Service를 함께 사용하면 각 Pod가 `<pod-name>.<service-name>` 형태의 고정된 DNS 이름을 가지게 되어, Pod가 재생성되어도 동일한 이름으로 식별 및 접근이 가능하다.

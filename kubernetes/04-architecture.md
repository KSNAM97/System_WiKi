# K8s-04 쿠버네티스 아키텍처

## 목차

1. [쿠버네티스 전체 구조 개요](#쿠버네티스-전체-구조-개요)
2. [마스터(Control Plane) 컴포넌트](#마스터control-plane-컴포넌트)
3. [워커 노드(Node) 컴포넌트](#워커-노드node-컴포넌트)
4. [전체 구성요소](#전체-구성요소)
5. [전체 배포 흐름 (0~6단계)](#전체-배포-흐름-06단계)
6. [namespace](#namespace)
7. [클러스터 노드 및 네임스페이스 확인](#클러스터-노드-및-네임스페이스-확인)
8. [YAML로 Pod 생성](#yaml로-pod-생성)
9. [- 와 -- 옵션 차이](#--와---옵션-차이)
10. [namespace에 pod 생성](#namespace에-pod-생성)
11. [Base namespace 변경 (Context 관리)](#base-namespace-변경-context-관리)
12. [커스텀 Nginx 이미지 생성 및 Kubernetes 배포 실습](#커스텀-nginx-이미지-생성-및-kubernetes-배포-실습)
13. [왜 자원 제한이 필요한가](#왜-자원-제한이-필요한가)
14. [ResourceQuota](#resourcequota)
15. [LimitRange](#limitrange)
16. [EX1) 베이스라인 만들기](#ex1-베이스라인-만들기)
17. [EX2) 파드 개수 제한](#ex2-파드-개수-제한)
18. [EX3-1) requests.cpu 총합 제한](#ex3-1-requestscpu-총합-제한)
19. [EX3-2) requests.memory 총합 제한](#ex3-2-requestsmemory-총합-제한)
20. [Limits-EX1) limits.cpu / limits.memory 총합 제한](#limits-ex1-limitscpu--limitsmemory-총합-제한)
21. [Limits-EX2) resources 없는 파드 생성 금지 확인](#limits-ex2-resources-없는-파드-생성-금지-확인)
22. [EX1) LimitRange 최소값 제한](#ex1-limitrange-최소값-제한)
23. [EX2) LimitRange min/max 범위 강제](#ex2-limitrange-minmax-범위-강제)
24. [Limit-TEST) CPU / Memory Limit 초과 동작 확인](#limit-test-cpu--memory-limit-초과-동작-확인)

---

## 쿠버네티스 전체 구조 개요

쿠버네티스 클러스터의 전체 골격을 먼저 살펴본다. 클러스터 구조를 이해하면 API Server부터 kubelet까지 이어지는 배포 흐름을 추적할 수 있고, Namespace로 dev/prod 등 환경을 분리하거나 ResourceQuota·LimitRange로 자원 사용을 통제하는 데도 이 구조에 대한 이해가 바탕이 된다.

-쿠버네티스 클러스터는 크게 **Control Plane**(컨트롤 플레인) 과 **Worker Node**(워커 노드) 로 나뉜다.

1) 마스터(Control Plane) 컴포넌트
  - 클러스터 전체를 관리하고 무엇을, 어떻게 실행할지 결정하는 두뇌 역할
  - 클러스터의 원하는 상태(Desired State)를 관리하고 실제 상태(Current State)가 원하는 상태와 일치하도록 조정한다.

2) 워커 노드(Node) 컴포넌트
 : 실제 컨테이너(Pod)를 실행하는 현장 역할

-개발자는 kubectl이나 API 요청을 통해 원하는 상태만 선언하고, 쿠버네티스는 이 상태를 계속 유지하도록 자동으로 동작한다.

**정리:** Control Plane은 두뇌, Worker Node는 현장 역할을 맡으며 개발자는 원하는 상태만 선언하면 된다. 이어서 각 Control Plane 컴포넌트를 하나씩 살펴본다.

---

## 마스터(Control Plane) 컴포넌트

Control Plane을 구성하는 핵심 컴포넌트들을 순서대로 설명한다.

### 1) kube-api server
**API Server**는 쿠버네티스의 모든 요청이 반드시 거치는 중앙 관문이다.
-쿠버네티스의 모든 요청이 반드시 거치는 중앙 관문
-kubectl 요청 수신
-REST API 형태로 요청 처리
-인증(Authentication)
-권한 확인(Authorization)
-요청 유효성 검사(Validation)
-최종적으로 **etcd**에 상태 저장
-다른 컴포넌트들은 직접 통신하지 않고 API Server를 통해서만 통신 (쿠버네티스의 단일 진입점)

### 2) etcd
**etcd**는 클러스터의 모든 상태 정보를 저장하는 저장소이다.
-Pod 정보
-Deployment 정보
-ReplicaSet 정보
-Service 정보
-ConfigMap 정보
-Secret 정보
-Node 정보
-Namespace 정보
-각 리소스의 현재 상태(Current State)
-사용자가 선언한 원하는 상태(Desired State)

### 3) kube-scheduler

**kube-scheduler**는 새로 생성될 Pod를 어떤 Worker Node에 배치할지 결정하는 컴포넌트다.

-새로 생성될 Pod를 어떤 Worker Node에 배치할지 결정하는 컴포넌트
-Pod를 직접 실행하지 않는다.
-적절한 Node를 선택한 후 해당 정보를 API Server에 반영한다.

-판단 기준
  - Node의 CPU / Memory 여유
  - Pod의 resource request/limit
  - nodeSelector, affinity/Anti-Affinity, taint/toleration
  - 이미 실행 중인 Pod 분포

-kube-scheduler는 실제로 Pod를 실행하지는 않는다.
-Pod는 Node A 또는 Node B에서 실행해라라고 결정만 한다.

EX) 회사에서 업무를 직원에게 배정하는 관리자 역할

### 4) kube-controller-manager

**kube-controller-manager**는 원하는 상태와 실제 상태를 비교해 자동으로 조정하는 컴포넌트다.

-클러스터의 현재 상태(Current State)를 지속적으로 감시하고
 사용자가 선언한 원하는 상태(Desired State) 와 비교 후 두 상태가 다르면 원하는 상태가 되도록 자동으로 조정한다.

-대표적인 컨트롤러들
  - Deployment Controller
  - ReplicaSet Controller
  - DaemonSet Controller
  - Node Controller
  - Job Controller / CronJob Controller

-동작 개념
  - 현재 상태(current state) 확인
  - 원하는 상태(desired state)와 비교
  - 다르면 자동으로 조치

예시
  - Pod 3개가 필요하다고 선언
  - 실제 Pod가 2개만 실행 중
  - 현재 상태와 원하는 상태를 비교해서 새 Pod 1개 생성

**정리:** Control Plane의 4대 컴포넌트(API Server, etcd, Scheduler, Controller Manager)는 각각 관문/저장소/배치결정/상태조정 역할을 나누어 맡는다. 이어서 실제 실행을 담당하는 Worker Node 컴포넌트를 살펴본다.

---

## 워커 노드(Node) 컴포넌트

실제 Pod와 컨테이너가 실행되는 작업 Node로 **Control Plane**이 결정한 내용을 실제로 수행하는 역할

### 1) kubelet

**kubelet**은 각 Worker Node에서 실행되는 Kubernetes 에이전트다.

-각 Worker Node에서 실행되는 Kubernetes 에이전트
-각 Worker Node마다 하나씩 실행되며, 백그라운드에서 지속적으로 동작하는 데몬 형태의 프로세스

-주요 역할
  - API Server와 통신하여 자신의 Node에 배정된 Pod 정보를 확인
  - API Server의 리소스 변경 사항을 Watch 방식으로 감시
  (Watch : 변경사항이 생기면 나한테 알려줘"라고 API Server에 등록해 놓는 방식)
  - 자신에게 할당된 Pod가 정상적으로 실행되고 있는지 관리
  - Container Runtime에 컨테이너의 생성·실행·중지 등의 작업을 요청
  - 컨테이너와 Pod의 상태를 지속적으로 확인
  - Pod 및 Node의 상태 정보를 API Server에 보고

-Pod를 실제로 실행하는 것은 kubelet 자체가 아니라 Container Runtime이다.

### 2) kube-proxy

**kube-proxy**는 Kubernetes의 Service 네트워크 기능을 구현하는 데 관여한다.

-Kubernetes의 Service 네트워크 기능을 구현하는 데 관여한다.
-Service로 들어온 트래픽을 적절한 Pod로 전달할 수 있도록 Node의 네트워크 규칙을 관리한다.
  - Service 동작 구현
  - iptables / ipvs 규칙 설정
  - Pod 간 통신, 로드밸런싱 지원

예시) Service IP로 요청 : kube-proxy가 규칙에 따라 여러 Pod 중 하나로 트래픽 전달

핵심 포인트
  - L4 레벨 트래픽 제어
  - 쿠버네티스 내부 네트워크의 핵심

### 3) Container Runtime

**Container Runtime**은 실제로 컨테이너를 실행하는 엔진이다.

-실제로 컨테이너를 실행하는 엔진
-대표 종류
  - containerd	: 현재 쿠버네티스에서 가장 널리 쓰이는 컨테이너 런타임
  - CRI-O		: Docker 기능 없이 쿠버네티스 실행에 필요한 최소 기능만 제공
  - (과거) Docker	: 예전 쿠버네티스에서 사용되던 컨테이너 런타임

**정리:** Worker Node는 kubelet(에이전트) - kube-proxy(네트워크) - Container Runtime(실행 엔진) 3요소로 구성된다.

---

## 전체 구성요소

-개발자 PC
  - Docker(또는 빌드 도구)	: 이미지 생성
  - kubectl			: 배포 요청 전송

-이미지 저장소(Registry)
  - Docker Hub / GitHub Container Registry / ECR
  - 이미지 보관 창고

-쿠버네티스 Control Plane
  - API Server		: 접수/조회/권한/상태 관리
  - etcd			: 상태 저장소
  - Scheduler		: 어느 워커에 배치할지 결정
  - Controller Manager	: 지금 상태가 사용자가 원한 상태와 다르면 자동으로 계속 수정하는 역할

-쿠버네티스 Worker Node
  - kubelet	: 현장 에이전트(명령 감시, 실행 지시, 상태 보고)
  - containerd	: 컨테이너 런타임(이미지 pull, 컨테이너 실행)
  - CNI(네트워크 플러그인): Pod IP/통신 구성

**정리:** 개발자 PC, 이미지 레지스트리, Control Plane, Worker Node가 각각의 역할로 연결되어 하나의 배포 파이프라인을 이룬다. 다음 절에서 이 흐름을 0~6단계로 나눠 자세히 추적한다.

---

## 전체 배포 흐름 (0~6단계)

### 0단계) 개발자 로컬 환경 (컨테이너 이미지 생성)

-실행 파일 + 라이브러리 + 설정을 하나로 묶어서 이미지로 만든다.
  - 쿠버네티스는 소스코드를 실행하지 않고 이미지를 실행한다.
  - 즉, 쿠버네티스에서 파드를 실행하려면 반드시 이미지 형태여야 한다.

```bash
docker build -t hub.example.com/nginx .
docker tag myapp:1.0 hub.example.com/myapp:1.0
docker push hub.example.com/myapp:1.0
```

중요 포인트
  - 쿠버네티스는 이미지를 직접 만드는 도구가 아니다.
  - 이미지는 반드시 Docker 같은 도구로 미리 만들어져 있어야 한다.

### 1단계) 사용자가 원하는 상태를 선언

-쿠버네티스의 시작은 이 애플리케이션을 항상 이런 상태로 유지해줘라고 요구사항만 선언한다.
-예를 들어 nginx 컨테이너를 항상 3개 실행하고 싶다면 replicas: 3 이라고만 적는다.
-이 단계에서 중요한 점은 사용자는 실행 위치, 실행 순서, 실행 방법을 전혀 신경 쓰지 않는다.

### 2단계) API Server가 요청을 접수하고 상태로 저장

-사용자가 kubectl을 통해 요청을 보내면, 그 요청은 가장 먼저 API Server로 들어온다.

-API Server는 이 사용자가 권한이 있는지 확인하고 설정 파일(yaml)이 문법적으로 맞는지 검사하고 실행 가능한 요청인지 검증한다.

-문제가 없으면 nginx Pod 3개를 유지해야 한다.는 정보를 클러스터의 상태 저장소(etcd)에 저장한다.

-이 순간부터 쿠버네티스 클러스터는 이 상태를 반드시 유지해야 한다.는 목표를 가지게 된다.

### 3단계) Controller Manager가 상태를 계속 감시

-Controller Manager는 쿠버네티스에서 감시와 조정을 담당하는 관리자다.

-이 컴포넌트는 API Server에 저장된 정보를 계속 지켜보면서 다음 질문을 반복한다.
  - 지금 클러스터 상태가 사용자가 원한 상태와 같은가?
  - 예를 들어 Pod가 3개여야 하는데 실제로는 2개만 실행 중이라면 Controller Manager는 상태가 어긋났다고 판단한다.

-중요한 점은 이 감시는 한 번만 이루어지는 것이 아니라 클러스터가 살아 있는 동안 계속 반복된다.

### 4단계) 상태가 다르면 Pod를 만들거나 줄이도록 요청

-Controller Manager는 컨테이너를 직접 실행하지 않는다.
-대신 API Server에게 Pod를 하나 더 만들어라 또는 Pod를 하나 줄여라라고 요청한다.
-이 요청으로 인해 새로운 Pod 객체가 생성된다.
-이 Pod는 아직 어느 워커 노드에서 실행될지 정해지지 않았기 때문에 잠시 Pending 상태로 보일 수 있다.
-이 단계는 무엇을 실행해야 하는지를 확정하는 단계라고 보면 된다.

### 5단계) Scheduler가 Pod를 실행할 워커 노드를 결정

-Scheduler는 새로 만들어질 Pod를 보고 이 Pod를 어느 워커 노드에서 실행할지를 결정한다.
 이때 Scheduler는 각 워커의 CPU와 메모리 여유, 라벨 조건, 분산 배치 여부 등을 고려한다.

-결정을 마치면 Pod 정보에 "이 Pod는 worker1에서 실행한다."라는 정보가 기록된다. (실행 장소가 확정)

### 6단계) 워커 노드에서 실제 실행과 유지

-Pod가 특정 워커 노드에 배정되면, 그 노드에서 실행 중인 kubelet이 이를 감지한다.
-kubelet은 이 Pod는 내가 실행해야 한다.는 사실을 확인한 뒤 컨테이너 런타임(containerd 등)에게 컨테이너 실행을 지시
-컨테이너 런타임은 이미지를 확인하고, 필요하면 Registry에서 이미지를 내려받은 뒤 컨테이너를 실제로 실행한다.
-컨테이너가 정상적으로 실행되면 그 상태가 kubelet을 통해 API Server로 보고되고, Pod 상태는 Running이 된다.
-이후에도 컨테이너가 종료되거나 문제가 생기면 이 상태 변화는 다시 감지되어 앞 단계들이 반복되면서 자동 복구가 이루어진다.

### 전체 흐름 요약

```
1) 개발자 Docker로 이미지 생성
2) Registry에 push
3) kubectl로 배포 요청
4) API Server가 접수
5) Scheduler가 노드 선택
6) kubelet이 명령 수신
7) containerd가 이미지 pull
8) 컨테이너 실행
9) Pod 생성 완료
```

**정리:** 이미지 생성부터 Pod 실행까지의 0~6단계는 선언(사용자) → 저장(API Server) → 감시/조정(Controller Manager) → 배치(Scheduler) → 실행(kubelet/Container Runtime)의 순환 구조로 요약된다. 이제 리소스를 논리적으로 분리하는 namespace 개념을 살펴본다.

---

## namespace

**namespace**는 한 개의 쿠버네티스 클러스터를 여러 개의 가상의 공간으로 나눠서 리소스를 서로 섞이지 않게 관리하는 기능이다.

-namespace는 한 개의 쿠버네티스 클러스터를 여러 개의 가상의 공간으로 나눠서 리소스를 서로 섞이지 않게 관리하는 기능

-대표적인 리소스 :  Pod , Deployment, Service , ConfigMap, Secret를 의미

-쿠버네티스를 수업에서 하나의 큰 교실(클러스터)이라고 치면 namespace는 조별로 나눠진 각 조의 책상 구역

예:
```
1조 namespace	: web Deployment
2조 namespace	: web Deployment
이게 동시에 가능하다(서로 다른 공간이므로 같은 이름을 사용할 수 있다.)
```

### namespace 특징 3가지

-분리(격리)
  - 팀/서비스/환경별로 리소스를 분리해서 관리
  - dev, test, prod를 namespace로 분리

-이름 충돌 방지
  - 같은 이름을 서로 다른 namespace에 만들 수 있음
  - 예: dev/web, prod/web

-권한과 자원 제한 적용
  - 특정 namespace에만 접근 가능하게 권한을 줄 수 있다.(RBAC)
  - 특정 namespace는 CPU/메모리를 얼마까지 쓰게 제한할 수 있다.(ResourceQuota)

-namespace에 속하는 리소스(대부분)
  - Pod
  - Deployment
  - ReplicaSet
  - Service
  - ConfigMap
  - Secret
  - Ingress(설치 방식에 따라 다를 수 있음)
  - Job, CronJob 등

-namespace에 속하지 않는 리소스(클러스터 전체 공용)
  - Node
  - PersistentVolume(PV)
  - StorageClass
  - Namespace 자체
  - ClusterRole / ClusterRoleBinding(권한 관련)

**정리:** namespace는 격리, 이름 충돌 방지, 권한/자원 제한이라는 세 가지 특징으로 클러스터를 논리적으로 나눈다.

### namespace 생성 방법

```bash
# CLI 방식
kubectl create namespace soldesk

# 확인
kubectl get namespaces
kubectl get ns

# YAML 방식 (dry-run으로 yaml 생성)
kubectl create namespace soldesk --dry-run=client -o yaml > soldesk-ns.yaml
```

---

## 클러스터 노드 및 네임스페이스 확인

실제 클러스터에서 노드와 네임스페이스, kube-system 내부 Pod들을 명령어로 확인해본다.

```bash
[root@k8s-master ~]# kubectl  get  nodes
NAME          STATUS   ROLES           AGE   VERSION
k8s-master    Ready    control-plane   20h   v1.35.7
k8s-worker1   Ready    <none>          20h   v1.35.7
k8s-worker2   Ready    <none>          20h   v1.35.7

[root@k8s-master ~]# kubectl  get  namespaces
NAME              STATUS   AGE
default           Active   20h
kube-flannel      Active   20h
kube-node-lease   Active   20h
kube-public       Active   20h
kube-system       Active   20h
```

```bash
[root@k8s-master ~]# kubectl  get  pod  --namespace kube-system
NAME                                      READY   STATUS    RESTARTS       AGE
coredns-7d764666f9-2dwdd                  1/1     Running   1 (137m ago)   20h
coredns-7d764666f9-klkhq                  1/1     Running   1 (137m ago)   20h
etcd-k8s-master                           1/1     Running   2 (137m ago)   20h
kube-apiserver-k8s-master                 1/1     Running   2 (137m ago)   20h
kube-controller-manager-k8s-master        1/1     Running   2 (137m ago)   20h
kube-proxy-bxjvw                          1/1     Running   1 (157m ago)   20h
kube-proxy-gbr4f                          1/1     Running   1 (157m ago)   20h
kube-proxy-rwhkc                          1/1     Running   1 (137m ago)   20h
kube-scheduler-k8s-master                 1/1     Running   2 (137m ago)   20h
```

컴포넌트 설명:
-coredns-7d764666f9-2dwdd
  - 클러스터 내부 DNS 역할
  - Service 이름을 IP로 해석해 주는 핵심 컴포넌트
  - Pod 간 서비스 이름 기반 통신에 사용

-coredns-7d764666f9-klkhq
  - CoreDNS 이중화용 Pod
  - 하나의 CoreDNS Pod에 문제가 생겨도 DNS 서비스가 유지되도록 복수 개 실행

-etcd-k8s-master
  - Kubernetes 클러스터의 모든 상태 정보를 저장하는 Key-Value 저장소
  - Pod, Deployment, Service, ConfigMap 등의 상태 정보 저장

-kube-apiserver-k8s-master
  - Kubernetes API의 중심
  - kubectl, Controller, Scheduler 등의 요청을 처리하는 API Server
  - Kubernetes의 모든 제어 요청이 이 컴포넌트를 통해 전달됨

-kube-controller-manager-k8s-master
  - 원하는 상태(Desired State)를 유지하도록 감시하는 Controller 모음
  - Pod 개수 유지, Node 상태 감시, 자동 복구 등을 담당

-kube-scheduler-k8s-master
  - 새로 생성되는 Pod를 어떤 Node에 배치할지 결정
  - 실제 Pod 실행은 하지 않고 배치할 Node만 결정

-kube-proxy-bxjvw / kube-proxy-gbr4f / kube-proxy-rwhkc
  - 각 Node에서 네트워크 통신을 관리하는 컴포넌트
  - Service로 들어온 트래픽을 실제 Pod로 전달
  - iptables 또는 nftables 등의 네트워크 규칙을 관리

```bash
[root@k8s-master ~]# kubectl  get  pod  -n kube-flannel
NAME                      READY   STATUS    RESTARTS       AGE
kube-flannel-ds-7hlrp     1/1     Running   1 (138m ago)   20h
kube-flannel-ds-f9wcz     1/1     Running   1 (158m ago)   20h
kube-flannel-ds-z4fbv     1/1     Running   2 (149m ago)   20h
```

**정리:** kube-system 네임스페이스 안에는 CoreDNS, etcd, API Server, Controller Manager, Scheduler, kube-proxy 등 Control Plane/Worker Node의 실체가 Pod 형태로 실행되고 있음을 확인할 수 있다.

---

## YAML로 Pod 생성

YAML 매니페스트로 Pod를 직접 생성하고, `create`와 `apply`의 차이를 실습으로 확인한다.

```bash
[root@k8s-master ~]# vi  nginx.yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod

spec:
  containers:
    - name: mynginx
      image: nginx:1.31
      ports:
        - containerPort: 80
```

```yaml
apiVersion: v1              # 쿠버네티스 API 버전 (Pod는 v1 사용)
kind: Pod                   # 생성할 리소스 종류 (Pod)
metadata:                   # 리소스의 메타데이터 영역
  name: mypod               # Pod이름(네임스페이스 내에서 유일해야 함)

spec:                       # Pod의 실제 동작 정의 영역
  containers:               # 이 Pod에서 실행될 컨테이너 목록
    - name: nginx           # 컨테이너 이름
      image: nginx:1.31     # 사용할 컨테이너 이미지
      ports:                # 컨테이너가 사용하는 포트 정보
        - containerPort: 80 # 컨테이너 내부에서 사용하는 HTTP 포트
```

```bash
# create vs apply 차이
# create로 다시 실행하게되면 이미 mypod 이름의 파드가 있기때문에 에러가 발생한다.
[root@k8s-master ~]# kubectl  create  -f  nginx.yaml
Error from server (AlreadyExists): error when creating "nginx.yaml": pods "mypod" already exists

[root@k8s-master ~]# kubectl  delete  -f  nginx.yaml
pod "mypod" deleted from default namespace

# apply로 실행 (이미 있으면 변경사항만 적용)
[root@k8s-master ~]# kubectl  apply  -f  nginx.yaml
pod/mypod created

[root@k8s-master ~]# kubectl  apply  -f  nginx.yaml
pod/mypod unchanged
```

**정리:** `create`는 이미 존재하면 에러를 내지만, `apply`는 변경사항만 반영하므로 반복 실행에 안전하다.

---

## - 와 -- 옵션 차이

리눅스 명령어와 kubectl에서는 옵션을 크게 **짧은 옵션**(short option)과 **긴 옵션**(long option)으로 구분한다.

-리눅스 명령어와 kubectl에서는 옵션을 크게 짧은 옵션(short option)과 긴 옵션(long option)으로 구분한다.

-짧은 옵션 (Short Option)
  - 옵션 앞에 - 를 사용한다.
  - 한 글자로 표현하는 경우가 많다.

```
-n       =  --namespace
-o       =  --output
-f       =  --filename
```

```bash
# EX1) 같은 의미
kubectl get pods -n soldesk
kubectl get pods --namespace soldesk

# EX2) 같은 의미
kubectl get pods -o wide
kubectl get pods --output wide

# EX3) 같은 의미
kubectl apply -f nginx.yaml
kubectl apply --filename nginx.yaml
```

**정리:** 짧은 옵션(-n, -o, -f)과 긴 옵션(--namespace, --output, --filename)은 동일한 의미이며 상황에 맞게 혼용할 수 있다.

---

## namespace에 pod 생성

하나의 YAML 파일 안에 Namespace와 Pod를 함께 정의해 특정 namespace에 Pod를 생성하는 방법을 실습한다.

```yaml
# mynginx.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: studydesk

---
apiVersion: v1
kind: Pod
metadata:
  name: studyweb
  namespace: studydesk

spec:
  containers:
    - name: nginxweb
      image: nginx:1.31
      ports:
        - containerPort: 80
```

```bash
[root@k8s-master ~]# kubectl  create  -f  mynginx.yaml  --dry-run=client
namespace/studydesk created (dry run)
pod/studyweb created (dry run)

[root@k8s-master ~]# kubectl  create  -f  mynginx.yaml
namespace/studydesk created

[root@k8s-master ~]# kubectl  get  namespaces
NAME              STATUS   AGE
default           Active   23h
kube-flannel      Active   23h
kube-node-lease   Active   23h
kube-public       Active   23h
kube-system       Active   23h
studydesk         Active   8s

[root@k8s-master ~]# kubectl  get  pods  -n studydesk
NAME       READY   STATUS    RESTARTS   AGE
studyweb   1/1     Running   0          109s

[root@k8s-master ~]# kubectl  get  pods  --all-namespaces
NAMESPACE      NAME                                  READY   STATUS    RESTARTS        AGE
kube-flannel   kube-flannel-ds-7hlrp                 1/1     Running   1 (4h59m ago)   23h
kube-flannel   kube-flannel-ds-f9wcz                 1/1     Running   1 (5h19m ago)   23h
kube-flannel   kube-flannel-ds-z4fbv                 1/1     Running   2 (5h10m ago)   23h
kube-system    coredns-7d764666f9-2dwdd              1/1     Running   1 (4h59m ago)   23h
kube-system    coredns-7d764666f9-klkhq              1/1     Running   1 (4h59m ago)   23h
kube-system    etcd-k8s-master                       1/1     Running   2 (4h59m ago)   23h
kube-system    kube-apiserver-k8s-master             1/1     Running   2 (4h59m ago)   23h
kube-system    kube-controller-manager-k8s-master    1/1     Running   2 (4h59m ago)   23h
kube-system    kube-proxy-bxjvw                      1/1     Running   1 (5h19m ago)   23h
kube-system    kube-proxy-gbr4f                      1/1     Running   1 (5h19m ago)   23h
kube-system    kube-proxy-rwhkc                      1/1     Running   1 (4h59m ago)   23h
kube-system    kube-scheduler-k8s-master             1/1     Running   2 (4h59m ago)   23h
studydesk      studyweb                              1/1     Running   0               3m17s

[root@k8s-master ~]# kubectl  delete  -f  mynginx.yaml
namespace "studydesk" deleted
pod "studyweb" deleted from studydesk namespace
```

**정리:** 하나의 YAML 파일에 Namespace 리소스와 Pod 리소스를 `---`로 구분해 함께 선언하면, 지정한 namespace 안에 Pod가 생성된다.

---

## Base namespace 변경 (Context 관리)

매번 `-n` 옵션으로 namespace를 지정하는 대신, **Context**를 이용해 기본 namespace 자체를 바꾸는 방법을 다룬다.

-예를 들어 이번주는 계속 soldesk의 namespace에 관련된 작업을 수행해야 하는데
 기본값으로 default namespace로 설정되어 있다.

-작업시 namespace를 변경하기위해서 -n  soldesk (--namespace) 명령어를 계속 사용해야 한다.

-변경 방법: k8s의 config 파일에 namespace를 등록해야 한다.

```bash
[root@k8s-master ~]# kubectl  config  view
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: DATA+OMITTED
    server: https://192.168.10.100:6443
  name: kubernetes
contexts:
- context:
    cluster: kubernetes
    user: kubernetes-admin
  name: kubernetes-admin@kubernetes
current-context: kubernetes-admin@kubernetes
kind: Config
users:
- name: kubernetes-admin
  user:
    client-certificate-data: DATA+OMITTED
    client-key-data: DATA+OMITTED
```

```bash
[root@k8s-master ~]# kubectl  config  \
  set-context  \
  soldesk@kubernetes \
  --cluster=kubernetes \
  --user=kubernetes-admin \
  --namespace=soldesk
Context "soldesk@kubernetes" created.
```

```
# kubectl  config		: kubectl 설정 파일 관리
# set-context		: 새로운 context 생성 또는 수정
# soldesk@kubernetes	: 생성할 context 이름 (여러 context를 구분하기위한 이름)
# --cluster=kubernetes	: 해당 context를 생성할 클러스터를 지정
# --user=kubernetes-admin	: kubenetes를 접속시 사용할 계정
# --namespace=soldesk	: 해당 context를 사용시 사용시 namespace를 soldesk로 지정
```

```bash
# 사용할 context를 변경
[root@k8s-master ~]# kubectl  config  use-context soldesk@kubernetes
Switched to context "soldesk@kubernetes".

[root@k8s-master ~]# kubectl  config  view
...
current-context: soldesk@kubernetes  # kubernetes-admin@kubernetes에서 변경됨

# 기본 context로 복구
[root@k8s-master ~]# kubectl  config  use-context kubernetes-admin@kubernetes
Switched to context "kubernetes-admin@kubernetes".
```

```bash
# soldesk namespace에서 pod 생성 (context 변경 후)
[root@k8s-master ~]# kubectl  run  --image=nginx:1.31  nameweb  --port 80
pod/nameweb created

[root@k8s-master ~]# kubectl get  pods
NAME       READY   STATUS    RESTARTS   AGE
nameweb    1/1     Running   0          31s
```

**정리:** kubectl config로 **Context**를 만들고 `use-context`로 전환하면, 매번 `-n`을 붙이지 않아도 지정한 namespace가 기본값으로 적용된다.

### 여러 namespace에 Pod 동시 생성 (하나의 YAML)

```yaml
# nginx.yaml
apiVersion: v1
kind: Pod
metadata:
  name: web1
spec:
  containers:
    - name: mynginx
      image: nginx:1.31
      ports:
        - containerPort: 80

---
apiVersion: v1
kind: Pod
metadata:
  name: web2
  namespace: studydesk
spec:
  containers:
    - name: mynginx
      image: nginx:1.31
      ports:
        - containerPort: 80

---
apiVersion: v1
kind: Pod
metadata:
  name: web3
  namespace: default
spec:
  containers:
    - name: mynginx
      image: nginx:1.31
      ports:
        - containerPort: 80
```

```bash
[root@k8s-master ~]# kubectl  apply  -f  nginx.yaml --dry-run=client
pod/web1 created (dry run)
pod/web2 created (dry run)
pod/web3 created (dry run)

[root@k8s-master ~]# kubectl  apply  -f  nginx.yaml
pod/web1 created
pod/web2 created
pod/web3 created

[root@k8s-master ~]# kubectl  get  pod  -n  soldesk
NAME      READY   STATUS    RESTARTS   AGE
nameweb   1/1     Running   0          29m
web1      1/1     Running   0          3s

[root@k8s-master ~]# kubectl  get  pod  -n  studydesk
NAME   READY   STATUS    RESTARTS   AGE
web2   1/1     Running   0          42s

[root@k8s-master ~]# kubectl  get  pod  -n  default
NAME   READY   STATUS    RESTARTS   AGE
web3   1/1     Running   0          46s

[root@k8s-master ~]# kubectl  delete  pods  --all		# soldesk namespace의 pods들만 삭제된다.
pod "nameweb" deleted from soldesk namespace
pod "web1" deleted from soldesk namespace
```

**정리:** 하나의 YAML에 서로 다른 `namespace:` 필드를 지정한 여러 Pod를 정의하면 한 번의 apply로 각 namespace에 분산 배치할 수 있고, 현재 Context의 기본 namespace가 지정되지 않은 리소스에 적용된다.

---

## 커스텀 Nginx 이미지 생성 및 Kubernetes 배포 실습

control-plane에서 직접 Nginx 커스텀 이미지를 생성한 후 **Docker Hub**에 Push하고, Worker Node가 이를 Pull하여 Pod/Deployment로 실행하는 전체 파이프라인을 실습한다.

-control-plane에서 직접 Nginx 커스텀 이미지를 생성한 후 Docker Hub에 Push하고,
 Kubernetes의 Worker Node가 해당 이미지를 Pull하여 Pod와 Deployment를 실행하는 전체 과정을 실습

흐름: 이미지 생성 --> 태그 변경 --> Docker Hub Push --> Kubernetes 배포 --> Pod 실행 --> 확인

### 1단계. 디렉터리 생성

```bash
[root@k8s-master ~]# mkdir  -p  /root/nginx-image-lab
[root@k8s-master ~]# cd ./nginx-image-lab/
[root@k8s-master nginx-image-lab]# pwd
/root/nginx-image-lab
```

### 2단계. index.html 작성

```bash
[root@k8s-master nginx-image-lab]# vi index.html
```

```html
<!doctype html>
<html>
<head>
  <meta charset="utf-8">
  <title>custom nginx</title>
</head>
<body>
  <h1>custom nginx image</h1>
  <p>built on control-plane</p>
</body>
</html>
```

### 3단계. Dockerfile 작성

```bash
[root@k8s-master nginx-image-lab]# vi dockerfile
```

```dockerfile
FROM  nginx:1.31

RUN  apt-get update && \
apt-get install -y \
curl \
iputils-ping \
iproute2 \
vim

ENV LANG=C.UTF-8
ENV LC_ALL=C.UTF-8

COPY index.html  /usr/share/nginx/html/index.html
```

### 4단계. 이미지 빌드

-이미지 이름	: custom-nginx-web
-태그		: 1.31

```bash
[root@k8s-master nginx-image-lab]# docker  build  -t  custom-nginx-web:1.31  .
[+] Building 15.6s (8/8) FINISHED
 => [internal] load build definition from dockerfile                              0.0s
 => => transferring dockerfile: 229B                                              0.0s
 => [internal] load metadata for docker.io/library/nginx:1.31                    4.2s
 => [1/3] FROM docker.io/library/nginx:1.31@sha256:8541484...                    4.0s
 => [2/3] RUN  apt-get update && apt-get install -y curl iputils-ping iproute2 vim  4.5s
 => [3/3] COPY index.html  /usr/share/nginx/html/index.html                      0.1s
 => exporting to image                                                            2.7s
 => => naming to docker.io/library/custom-nginx-web:1.31                         0.0s

[root@k8s-master ~]# docker  images
IMAGE                          ID              DISK USAGE   CONTENT SIZE   EXTRA
custom-nginx-web:1.31          02218a3d52d6    348MB        97MB
```

### 5단계. docker tag로 Hub용 이미지 태그 생성

**docker tag**는 이미지를 새로 만드는 게 아니라 기존 이미지에 이름표를 하나 더 붙이는 것이다.

-docker tag는 이미지를 새로 만드는 게 아니라 기존 이미지에 이름표를 하나 더 붙이는 것

```bash
[root@k8s-master ~]# docker tag  custom-nginx-web:1.31  konan7979/custom-nginx-web:1.31

[root@k8s-master ~]# docker  images
IMAGE                             ID              DISK USAGE   CONTENT SIZE   EXTRA
custom-nginx-web:1.31             02218a3d52d6    348MB        97MB
konan7979/custom-nginx-web:1.31   02218a3d52d6    348MB        97MB
```

### 6단계. Docker Hub 로그인

```bash
root@k8s-master nginx-image-lab]# docker login
USING WEB-BASED LOGIN
Your one-time device confirmation code is: DXCW-QFWC
Press ENTER to open your browser or submit your device code here: https://login.docker.com/activate
Login Succeeded
```

### 7단계. Docker Hub로 이미지 push

```bash
[root@k8s-master nginx-image-lab]# docker  push konan7979/custom-nginx-web:1.31
The push refers to repository [docker.io/konan7979/custom-nginx-web]
1f7f5124d211: Pushed
44136fa355b3: Pushed
3c55dc422a81: Pushed
26c307b5e35a: Pushed
c0df8d325117: Pushed
d84ae7b21412: Pushed
b8b80b9bc028: Pushed
5a4222b844e8: Pushed
f5de6e85ac74: Pushed
59b9be530d2f: Pushed
79f88e26950e: Pushed
1.31: digest: sha256:02218a3d52d69c10c071c3d5a426cffb6e56d61a4f72d5373b5319a587e1e358 size: 856

[root@k8s-master ~]# docker  search  konan7979/custom-nginx-web
 NAME                         DESCRIPTION   STARS     OFFICIAL
konan7979/custom-nginx-web                  0
```

### 8단계. Kubernetes Pod 생성 (워커 노드에서 실행됨)

```bash
[root@k8s-master nginx-image-lab]# kubectl  run  custom-nginx-web-pod  --image=konan7979/custom-nginx-web:1.31  --port 80
pod/custom-nginx-web-pod created

[root@k8s-master nginx-image-lab]# kubectl  get  pods
NAME                   READY   STATUS    RESTARTS   AGE
custom-nginx-web-pod   1/1     Running   0          38s

[root@k8s-master nginx-image-lab]# kubectl  get  pods  -o  wide
NAME                   READY   STATUS    RESTARTS   AGE   IP             NODE          NOMINATED NODE   READINESS GATES
custom-nginx-web-pod   1/1     Running   0          57s   10.244.2.16    k8s-worker2   <none>           <none>

[root@k8s-master nginx-image-lab]# curl http://10.244.2.16
<!doctype html>
<html>
<head>
  <meta charset="utf-8">
  <title>custom nginx</title>
</head>
<body>
  <h1>custom nginx image</h1>
  <p>built on control-plane</p>
</body>
</html>

[root@k8s-master nginx-image-lab]# kubectl  delete  pods  custom-nginx-web-pod
pod "custom-nginx-web-pod" deleted from default namespace
```

### YAML로 Pod 생성 및 Deployment 배포

```yaml
# nginx-custom-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-custom-pod
  namespace: soldesk
spec:
  containers:
  - name: nginx-custom-container
    image: konan7979/custom-nginx-web:1.31
    ports:
    - containerPort: 80
```

```bash
[root@k8s-master nginx-image-lab]# kubectl  apply  -f  nginx-custom-pod.yam
pod/nginx-custom-pod created

[root@k8s-master nginx-image-lab]# kubectl  get  pods  -n  soldesk
NAME               READY   STATUS    RESTARTS   AGE
nginx-custom-pod   1/1     Running   0          14s

[root@k8s-master nginx-image-lab]# kubectl  create  deployment  nginx-custom-pod --image=konan7979/custom-nginx-web:1.31 --replicas=3
deployment.apps/nginx-custom-pod created

[root@k8s-master nginx-image-lab]# kubectl  get  pods
NAME                                READY   STATUS    RESTARTS   AGE
nginx-custom-pod-b5c4b9774-6vbl5    1/1     Running   0          17s
nginx-custom-pod-b5c4b9774-s8shp    1/1     Running   0          17s
nginx-custom-pod-b5c4b9774-t258r    1/1     Running   0          17s
```

```bash
# deployments 수정 (edit) - replicas 3→5
[root@k8s-master nginx-image-lab]# kubectl  edit  deployments.apps nginx-custom-pod

[root@k8s-master nginx-image-lab]# kubectl  get  pods
NAME                                READY   STATUS    RESTARTS   AGE
nginx-custom-pod-b5c4b9774-6vbl5    1/1     Running   0          5m54s
nginx-custom-pod-b5c4b9774-mj2gk    1/1     Running   0          10s
nginx-custom-pod-b5c4b9774-nm5c6    1/1     Running   0          10s
nginx-custom-pod-b5c4b9774-s8shp    1/1     Running   0          5m54s
nginx-custom-pod-b5c4b9774-t258r    1/1     Running   0          5m54s

# deployments scale로 수정
[root@k8s-master nginx-image-lab]# kubectl  scale  deployment  nginx-custom-pod  --replicas=3
deployment.apps/nginx-custom-pod scaled

[root@k8s-master nginx-image-lab]# kubectl  delete deployment  nginx-custom-pod
```

### Deployment YAML 예제

```yaml
# custom-nginx.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-custom-deploy
spec:
  replicas: 3

  selector:
    matchLabels:
      app: nginx-custom

  template:
    metadata:
      labels:
        app: nginx-custom
    spec:
      containers:
        - name: nginx
          image: konan7979/custom-nginx-web:1.31
```

```bash
[root@k8s-master nginx-image-lab]# kubectl get pods
NAME                                    READY   STATUS    RESTARTS   AGE
nginx-custom-deploy-6ccb7f6479-82b6k    1/1     Running   0          8s
nginx-custom-deploy-6ccb7f6479-shjxl    1/1     Running   0          8s
nginx-custom-deploy-6ccb7f6479-tcgtx    1/1     Running   0          8s
```

**정리:** 커스텀 이미지는 build → tag → push → (Pod 실행 또는 Deployment 배포) 순서로 Kubernetes 클러스터까지 전달되며, Deployment를 사용하면 replicas 수만큼 Pod가 자동으로 유지된다. 다음으로 여러 파드가 자원을 공유할 때 필요한 제한 정책을 다룬다.

---

## 왜 자원 제한이 필요한가

-쿠버네티스 클러스터는 여러 팀 / 여러 서비스가 같이 쓰는 서버 묶음이다.

-자원 제한이 없으면
  - 한 팀이 실수로 파드를 무한히 생성
  - 어떤 파드가 메모리, CPU를 과도하게 사용
  - 다른 팀 서비스가 갑자기 느려지거나 죽음

-여기서 등장하는 개념이 **ResourceQuota** 와 **LimitRange**다.

---

## ResourceQuota

**ResourceQuota**는 네임스페이스 전체에 걸리는 제한이다. 쉽게 말하면 "이 네임스페이스는 총량이 여기까지다." 라고 말하는 규칙이다.

-ResourceQuota는 네임스페이스 전체에 걸리는 제한이다.
 쉽게 말하면 "이 네임스페이스는 총량이 여기까지다." 라고 말하는 규칙이다.

-ResourceQuota가 제한하는 것들
  - pods
  - requests.cpu / limits.cpu
  - requests.memory / limits.memory
  - services

-pods
  - 해당 네임스페이스에서 생성할 수 있는 파드의 최대 개수를 제한
  - 예: pods: 10 (이 네임스페이스에는 파드를 최대 10개까지만 생성 가능)

-requests.cpu
  - 파드들이 "최소로 보장받고 싶다"고 요청한 CPU의 총합 제한
  - 예: requests.cpu: 2 (이 네임스페이스의 모든 파드 CPU 요청 합계는 2코어를 넘을 수 없음)

-limits.cpu
  - 실제로 파드가 사용할 수 있는 CPU 최대치의 총합 제한
  - 예: limits.cpu: 4 (모든 파드의 CPU limit 합계는 4코어를 초과할 수 없음)

-requests.memory
  - 파드들이 "최소 보장 메모리"로 요청한 값의 총합 제한
  - 예: requests.memory: 2Gi

-limits.memory
  - 파드들이 실제로 사용할 수 있는 메모리 최대치의 총합 제한
  - 예: limits.memory: 4Gi

-services
  - 네임스페이스에서 생성 가능한 Service 객체 개수 제한
  - 예: services: 5

**정리:** ResourceQuota는 pods 개수, requests/limits의 cpu·memory, services 개수 등 네임스페이스 단위의 총량을 통제한다. 다음은 파드 하나하나에 적용되는 LimitRange다.

---

## LimitRange

**LimitRange**는 개별 파드(또는 컨테이너)에 적용되는 규칙이다. 쉽게 말하면 파드 하나당 규칙이다.

-LimitRange는 개별 파드(또는 컨테이너)에 적용되는 규칙이다.
 쉽게 말하면 파드 하나당 규칙이다.

-LimitRange가 하는 핵심 역할
  - 너무 작은 값 방지 (말도 안 되는 파드 생성 방지)
  - 너무 큰 값 방지 (한 파드가 자원 독점하는 것 방지)
  - request / limit 안 적어도 기본값 강제 (아무 생각 없이 만든 파드를 통제)

### LimitRange 주요 설정값

-default
  - Pod/Container에 limits를 작성하지 않았을 때 자동으로 적용되는 기본 Limit 값

-defaultRequest
  - Pod/Container에 requests를 작성하지 않았을 때 자동으로 적용되는 기본 Request 값

-min
  - 사용자가 설정할 수 있는 최소 requests/limits 값
  - 이 값보다 작게 설정하면 Pod 생성이 거부됨

-max
  - 사용자가 설정할 수 있는 최대 requests/limits 값
  - 이 값보다 크게 설정하면 Pod 생성이 거부됨

### ResourceQuota와 LimitRange 차이

-ResourceQuota
  - 범위: 네임스페이스 전체
  - 목적: 전체 자원 총량 통제
  - 예: 파드 최대 10개, CPU 총합 2코어

-LimitRange
  - 범위: 파드/컨테이너 하나
  - 목적: 파드 크기 규격화
  - 예: 파드 메모리 128Mi ~ 512Mi

-LimitRange로 파드 모양을 통제, ResourceQuota로 네임스페이스 전체 규모를 통제하는 조합으로 운영 단위를 만든다.

**정리:** ResourceQuota(네임스페이스 총량)와 LimitRange(파드 단위 규격)는 서로 보완적으로 동작한다. 이제 실습으로 두 개념의 동작을 직접 확인한다.

---

## EX1) 베이스라인 만들기

resource 네임스페이스를 만들고 Quota 없이 파드 3개를 생성해 제한이 없는 상태를 확인

```bash
[root@k8s-master ~]# kubectl  create  namespace resource
namespace/resource created

[root@k8s-master ~]# kubectl  run  base-pod1  --image=nginx:1.31  -n resource
pod/base-pod1 created

[root@k8s-master ~]# kubectl  run  base-pod2  --image=nginx:1.31  -n resource
pod/base-pod2 created

[root@k8s-master ~]# kubectl  run  base-pod3  --image=konan7979/custom-nginx-web:1.31  -n resource
pod/base-pod3 created

[root@k8s-master ~]# kubectl  get  pods   -n  resource
NAME        READY   STATUS    RESTARTS   AGE
base-pod1   1/1     Running   0          4m21s
base-pod2   1/1     Running   0          4m16s
base-pod3   1/1     Running   0          94s
```

```yaml
# step1-baseline.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: resource
---
apiVersion: v1
kind: Pod
metadata:
  name: base-pod1
  namespace: resource
spec:
  containers:
    - name: nginx-web1
      image: nginx:1.31
---
apiVersion: v1
kind: Pod
metadata:
  name: base-pod2
  namespace: resource
spec:
  containers:
    - name: nginx-web2
      image: nginx:1.31
---
apiVersion: v1
kind: Pod
metadata:
  name: base-pod3
  namespace: resource
spec:
  containers:
    - name: nginx-web3
      image: konan7979/custom-nginx-web:1.31
```

**정리:** 제한이 없는 상태에서는 파드 개수나 자원 사용량에 아무 제약 없이 생성이 가능함을 베이스라인으로 확인했다.

---

## EX2) 파드 개수 제한

resource에 pods=2 **ResourceQuota**를 걸고 파드 3개 생성 시 3번째 pod부터 거부되는지 확인

```yaml
# rq-step2-qouta.yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: rq-pod-count
  namespace: resource
spec:
  hard:
    pods: 2
```

```bash
[root@k8s-master ~]# kubectl  apply  -f  rq-step2-qouta.yaml
resourcequota/rq-pod-count created

[root@k8s-master ~]# kubectl  get  resourcequotas  -n  resource
NAME            REQUEST     LIMIT   AGE
rq-pod-count    pods: 0/2           51s

[root@k8s-master ~]# kubectl  describe  resourcequotas  -n  resource
Name:         rq-pod-count
Namespace:    resource
Resource     Used  Hard
--------     ----  ----
pods          0     2

-Hard = 이 네임스페이스에 허용된 최대 pod 한도(제한값)
-Used = 현재 이 네임스페이스에서 실제로 사용 중인 pod

# 첫번째 pod 생성
[root@k8s-master ~]# kubectl  run  base-pod1  --image=nginx:1.31  -n resource
pod/base-pod1 created

[root@k8s-master ~]# kubectl  get  resourcequotas  -n  resource
NAME           REQUEST     LIMIT   AGE
rq-pod-count   pods: 1/2           4m46s

# 두번째 pod 생성
[root@k8s-master ~]# kubectl  run  base-pod2  --image=nginx:1.31  -n resource
pod/base-pod2 created

[root@k8s-master ~]# kubectl  get  resourcequotas  -n  resource
NAME            REQUEST     LIMIT   AGE
rq-pod-count    pods: 2/2           6m4s

# 세번째 pod 생성 -> 실패
[root@k8s-master ~]# kubectl  run  base-pod3  --image=konan7979/custom-nginx-web:1.31  -n resource
Error from server (Forbidden): pods "base-pod3" is forbidden: exceeded quota: rq-pod-count, requested: pods=1, used: pods=2, limited: pods=2
```

**정리:** `pods: 2`로 설정한 ResourceQuota는 Hard(허용 최대치)를 초과하는 3번째 Pod 생성 요청을 거부한다.

---

## EX3-1) requests.cpu 총합 제한

resource에 requests.cpu=1 를 제한하고 요청이 있는 파드로 총합을 채운 뒤 초과 생성이 거부되는지 확인

```yaml
# rq-step3-requests-qouta.yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: rq-req-total
  namespace: resource
spec:
  hard:
    requests.cpu: "1"
```

```bash
[root@k8s-master ~]# kubectl  apply  -f  rq-step3-requests-qouta.yaml
resourcequota/rq-req-total created

[root@k8s-master ~]# kubectl  describe  resourcequotas  -n  resource
Name:           rq-req-total
Namespace:      resource
Resource       Used  Hard
--------       ----  ----
requests.cpu   0       1
```

```yaml
# rq-step3-request-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: req-pod1
  namespace: resource
spec:
  containers:
  - name: nginx-container
    image: nginx:1.31
    resources:
      requests:
        cpu: "500m"
---
apiVersion: v1
kind: Pod
metadata:
  name: req-pod2
  namespace: resource
spec:
  containers:
  - name: nginx-container
    image: nginx:1.31
    resources:
      requests:
        cpu: "400m"
---
apiVersion: v1
kind: Pod
metadata:
  name: req-pod3
  namespace: resource
spec:
  containers:
  - name: nginx-container
    image: nginx:1.31
    resources:
      requests:
        cpu: "400m"
```

```bash
[root@k8s-master ~]# kubectl  apply  -f  rq-step3-request-pod.yaml
pod/req-pod1 created
pod/req-pod2 created
Error from server (Forbidden): error when creating "rq-step3-request-pod.yaml": pods "req-pod3" is forbidden: exceeded quota: rq-req-total, requested: requests.cpu=400m, used: requests.cpu=900m, limited: requests.cpu=1

[root@k8s-master ~]# kubectl  get  resourcequotas  -n resource
NAME           REQUEST                LIMIT   AGE
rq-req-total   requests.cpu: 900m/1           19m
```

**정리:** `requests.cpu: "1"` 제한 하에서 500m+400m=900m까지는 허용되고, 추가 400m 요청은 총합이 1을 초과하므로 거부된다.

---

## EX3-2) requests.memory 총합 제한

resource에 requests.memory=1Gi 를 제한하고 초과 생성이 거부되는지 확인

```yaml
# rq-step3-2-request-quota.yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: rq-req-mem-total
  namespace: resource
spec:
  hard:
    requests.memory: "1Gi"
```

```yaml
# rq-step-3-2-request-mem-pod.yaml
# mem-pod1: 512Mi, mem-pod2: 512Mi, mem-pod3: 256Mi (초과 실패)
apiVersion: v1
kind: Pod
metadata:
  name: mem-pod1
  namespace: resource
spec:
  containers:
  - name: mem-container
    image: nginx:1.31
    resources:
      requests:
        memory: "512Mi"
---
apiVersion: v1
kind: Pod
metadata:
  name: mem-pod2
  namespace: resource
spec:
  containers:
  - name: mem-container
    image: nginx:1.31
    resources:
      requests:
        memory: "512Mi"
---
apiVersion: v1
kind: Pod
metadata:
  name: mem-pod3
  namespace: resource
spec:
  containers:
  - name: mem-container
    image: nginx:1.31
    resources:
      requests:
        memory: "256Mi"
```

```bash
[root@k8s-master ~]#  kubectl  apply  -f   rq-step-3-2-request-mem-pod.yaml
pod/mem-pod1 created
pod/mem-pod2 created
Error from server (Forbidden): error when creating "rq-step-3-2-request-mem-pod.yaml": pods "mem-pod3" is forbidden: exceeded quota: rq-req-mem-total, requested: requests.memory=256Mi, used: requests.memory=1Gi, limited: requests.memory=1Gi

[root@k8s-master ~]# kubectl  describe  resourcequotas  -n  resource
Name:             rq-req-mem-total
Namespace:        resource
Resource          Used  Hard
--------          ----  ----
requests.memory   1Gi   1Gi
```

**정리:** `requests.memory`도 `requests.cpu`와 동일한 방식으로 네임스페이스 총합을 제한하며, 512Mi+512Mi=1Gi를 채운 뒤 추가 요청은 거부된다.

---

## Limits-EX1) limits.cpu / limits.memory 총합 제한

limits.cpu=2, limits.memory=1Gi 를 걸고 초과 생성이 거부되는지 확인

```yaml
# rq-step1-limit-quota.yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: rq-limits-total
  namespace: resource
spec:
  hard:
    limits.cpu: "2"
    limits.memory: "1Gi"
```

```yaml
# rq-step1-limit-quota-pods.yaml
# limit-a, limit-b: cpu 1, mem 512Mi / limit-c: cpu 1, mem 512Mi (초과 실패)
apiVersion: v1
kind: Pod
metadata:
  name: limit-a
  namespace: resource
spec:
  containers:
  - name: limit-nginx
    image: nginx:1.31
    resources:
      limits:
        cpu: "1"
        memory: "512Mi"
---
apiVersion: v1
kind: Pod
metadata:
  name: limit-b
  namespace: resource
spec:
  containers:
  - name: limit-nginx
    image: nginx:1.31
    resources:
      limits:
        cpu: "1"
        memory: "512Mi"
---
apiVersion: v1
kind: Pod
metadata:
  name: limit-c
  namespace: resource
spec:
  containers:
  - name: limit-nginx
    image: nginx:1.31
    resources:
      limits:
        cpu: "1"
        memory: "512Mi"
```

```bash
[root@k8s-master ~]# kubectl  apply  -f  rq-step1-limit-quota-pods.yaml
pod/limit-a created
pod/limit-b created
Error from server (Forbidden): error when creating "rq-step1-limit-quota-pods.yaml": pods "limit-c" is forbidden: exceeded quota: rq-limits-total, requested: limits.cpu=1,limits.memory=512Mi, used: limits.cpu=2,limits.memory=1Gi, limited: limits.cpu=2,limits.memory=1Gi

[root@k8s-master ~]# kubectl  get  resourcequotas  -n  resource
NAME              REQUEST    LIMIT                                      AGE
rq-limits-total              limits.cpu: 2/2, limits.memory: 1Gi/1Gi   19m

# limit를 사용하여 제한하게되면 limit와 request가 모두 적용된다.
[root@k8s-master ~]# kubectl  describe  pods  limit-a  -n  resource
...
    Limits:
      cpu:     1
      memory:  512Mi
    Requests:
      cpu:        1
      memory:     512Mi
```

**정리:** `limits.cpu`/`limits.memory` 제한도 requests와 같은 원리로 동작하며, limit을 지정하면 request도 함께 채워짐을 확인했다.

---

## Limits-EX2) resources 없는 파드 생성 금지 확인

```yaml
# rq-step2-limit-quota.yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: rq-request-limits-total
  namespace: resource
spec:
  hard:
    requests.cpu: "1"
    requests.memory: "512Mi"
    limits.cpu: "2"
    limits.memory: "1Gi"
```

```yaml
# rq-step2-no-limit-quota-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: bad-pod
  namespace: resource
spec:
  containers:
  - name: nginx
    image: nginx
```

```bash
[root@k8s-master ~]# kubectl  apply  -f  rq-step2-no-limit-quota-pod.yaml
Error from server (Forbidden): error when creating "rq-step2-no-limit-quota-pod.yaml": pods "bad-pod" is forbidden: failed quota: rq-request-limits-total: must specify limits.cpu for: nginx; limits.memory for: nginx; requests.cpu for: nginx; requests.memory for: nginx
```

-ResourceQuota가 보는 항목은 4개다
  - requests.cpu
  - requests.memory
  - limits.cpu
  - limits.memory

-파드에 resources가 없으면(= requests/limits 미기재) 쿠버네티스 입장에서는
 이 파드의 requests.cpu는 얼마지 알 수 없다.

-ResourceQuota가 CPU/Memory의 requests와 limits를 관리하는 네임스페이스에서는
  파드도 해당 resources 값을 명시해야 한다.

### resources 있는 파드와 없는 파드 비교 실습

```yaml
# 실습 1: requests 범위 이내 (성공)
# requests.cpu = 400m + 300m + 300m = 1000m
# limits.cpu   = 600m + 600m + 600m = 1800m
apiVersion: v1
kind: Pod
metadata:
  name: pod-a
  namespace: resource
spec:
  containers:
  - name: nginx
    image: nginx:1.31
    resources:
      requests:
        cpu: "400m"
        memory: "256Mi"
      limits:
        cpu: "600m"
        memory: "512Mi"
---
# pod-b: requests.cpu 300m, limits.cpu 600m ...
```

```bash
[root@k8s-master ~]# kubectl apply -f rq-step2-limit-quota-pod.yaml
pod/pod-a created
pod/pod-b created
pod/pod-c created

[root@k8s-master ~]# kubectl  get  resourcequotas -n resource
NAME                       REQUEST                                             LIMIT                                           AGE
rq-request-limits-total    requests.cpu: 1/1, requests.memory: 512Mi/512Mi    limits.cpu: 1800m/2, limits.memory: 1Gi/1Gi     3m29s
```

**정리:** requests/limits의 cpu·memory를 모두 관리하는 ResourceQuota가 있는 네임스페이스에서는, 파드에도 resources 값을 반드시 명시해야 생성이 허용된다. 이제 파드 개별 규격을 강제하는 **LimitRange** 실습으로 넘어간다.

---

## EX1) LimitRange 최소값 제한

limit의 min CPU 200m, min Memory 128Mi를 설정하고 그보다 작은 파드를 생성해 실패를 확인

```yaml
# step1-limitrange.yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: lr-min-max-default
  namespace: resource

spec:
  limits:
  - type: Container
    min:
      cpu: "200m"
      memory: "128Mi"
```

```bash
[root@k8s-master ~]# kubectl  apply  -f  step1-limitrange.yaml
limitrange/lr-min-max-default created

[root@k8s-master ~]# kubectl  describe  limitranges lr-min-max-default  -n  resource
Name:       lr-min-max-default
Namespace:  resource
Type        Resource  Min     Max  Default Request  Default Limit  Max Limit/Request Ratio
----        --------  ---     ---  ---------------  -------------  -----------------------
Container   cpu       200m    -    200m             -              -
Container   memory    128Mi   -    128Mi            -              -
```

```bash
# 최소보다 큰 pod 생성 (성공)
[root@k8s-master ~]# kubectl  apply -f  step1-normal-pod.yaml
pod/normal-pod created

# 최소보다 작은 pod 생성 (실패)
[root@k8s-master ~]# kubectl  apply -f  step1-min-pod.yaml
Error from server (Forbidden): error when creating "step1-min-pod.yaml": pods "min-pod" is forbidden: minimum cpu usage per Container is 200m, but request is 50m
```

**정리:** LimitRange의 `min` 값보다 작은 requests/limits로 파드를 생성하면 거부되어, 지나치게 작은 파드 생성을 막을 수 있다.

---

## EX2) LimitRange min/max 범위 강제

min/max 범위를 벗어난 파드 생성이 실패하는지 확인

```yaml
# step2-limitrange.yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: lr-min-max-default
  namespace: resource

spec:
  limits:
  - type: Container
    min:
      cpu: "150m"
      memory: "128Mi"
    max:
      cpu: "300m"
      memory: "512Mi"
    default:              # limit 미 설정시 적용
      cpu: "250m"
      memory: "256Mi"
    defaultRequest:       # request 미 설정시 적용
      cpu: "200m"
      memory: "128Mi"
```

**정리:** LimitRange의 `min`/`max`로 파드 크기의 하한과 상한을 강제하고, `default`/`defaultRequest`로 값 미기재 시 기본값을 자동 부여할 수 있다.

---

## Limit-TEST) CPU / Memory Limit 초과 동작 확인

resource 네임스페이스에 limit-test-pod를 생성하고 CPU와 Memory Limit의 실제 동작을 확인

```
네임스페이스    : resource
Pod             : limit-test-pod
컨테이너        : stress
CPU Limit       : 500m
Memory Limit    : 100Mi
```

### CPU 부하 테스트

```yaml
# cpu-list-test.yaml
apiVersion: v1
kind: Pod
metadata:
  name: limit-test-pod
  namespace: resource
spec:
  containers:
  - name: limt-test-container
    image: polinux/stress
    resources:
      limits:
        cpu: "500m"
        memory: "100Mi"
    command: ["stress"]
    args: ["--cpu", "1"]
```

-polinux/stress는 CPU, Memory 등에 강제로 부하를 발생시키는 stress 프로그램이 포함된 이미지

```bash
[root@k8s-master ~]# nproc		# CPU 코어 개수 확인
4

[root@k8s-master ~]# kubectl  apply  -f  cpu-list-test.yaml
pod/limit-test-pod created

[root@k8s-master ~]# kubectl  get  pods  -n  resource
NAME             READY   STATUS    RESTARTS   AGE
limit-test-pod   1/1     Running   0          17s
```

```bash
[root@k8s-worker1 ~]# ps -eo pid,ppid,comm,args,%cpu,%mem --sort=-%cpu | head -10
    PID   PPID  COMMAND     COMMAND                        %CPU  %MEM
   8598   8586  stress      stress --cpu 1                 49.9   0.0
   2258      1  kubelet     /usr/bin/kubelet --bootstra     1.1   4.3
    945      1  containerd  /usr/bin/containerd             0.6   3.5
```

-stress가 원하는 CPU : 약 1 CPU  
-Pod CPU Limit       : 0.5 CPU  
-실제로는 **CPU Limit**에 의해 제한된다.

### Memory 부하 테스트 (OOMKilled)

```yaml
# cpu-list-test.yaml (메모리 테스트용)
apiVersion: v1
kind: Pod
metadata:
  name: limit-test-pod
  namespace: resource
spec:
  containers:
  - name: limt-test-container
    image: polinux/stress
    resources:
      limits:
        cpu: "500m"
        memory: "100Mi"
    command: ["stress"]
    args:
    - "--vm"
    - "1"
    - "--vm-bytes"
    - "150M"
    - "--vm-hang"
    - "1"
```

```
"--vm 1"         : stress를 사용해서 메모리 부하를 발생시키는 vm 워커 1개 생성
"--vm-bytes 150M": vm 워커가 150M의 부하를 발생
전체 명령어: stress  --vm 1  --vm-bytes 150M  --vm-hang 1
```

```bash
[root@k8s-master ~]# kubectl  get  pods  -n  resource
NAME             READY   STATUS      RESTARTS      AGE
limit-test-pod   0/1     OOMKilled   2 (23s ago)   27s

[root@k8s-master ~]# kubectl  get  pods  -o  wide  -n  resource
NAME             READY   STATUS      RESTARTS      AGE    IP           NODE          NOMINATED NODE   READINESS GATES
limit-test-pod   0/1     OOMKilled   4 (67s ago)   116s   10.244.1.3   k8s-worker1   <none>           <none>
```

**정리:** CPU Limit을 초과하는 부하는 CPU 사용량이 강제로 제한(throttling)되지만, Memory Limit을 초과하면 컨테이너가 **OOMKilled** 상태로 강제 종료된다는 차이를 확인했다. 이것으로 Control Plane/Worker Node 구조부터 namespace, ResourceQuota, LimitRange까지 쿠버네티스 아키텍처 전반을 살펴보았다.

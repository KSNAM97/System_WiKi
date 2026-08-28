# K8s-03 쿠버네티스 Pod 생성

## 목차

1. [노드 확인](#노드-확인)
2. [Kubernetes Pod 개념](#kubernetes-pod-개념)
3. [Pod 생성](#pod-생성)
4. [Deployment로 Pod 관리](#deployment로-pod-관리)

## 노드 확인

Pod를 생성하기 전에 클러스터를 구성하는 **노드(Node)**의 상태를 먼저 확인한다. Pod는 `kubectl run`으로 즉시 만들어 간단히 확인할 수도 있고, 여러 개를 안정적으로 운영하며 장애 시 자동으로 복구되도록 Deployment로 관리할 수도 있다. `kubectl` 명령으로 노드 목록과 상세 정보를 조회할 수 있다.

```bash
[root@k8s-master ~]# kubectl  api-resources	# 축약 명령어 확인
```

```bash
[root@k8s-master ~]# kubectl  get  nodes
NAME           STATUS   ROLES           AGE   VERSION
k8s-master     Ready    control-plane   81m   v1.35.7
k8s-worker1    Ready    <none>          57m   v1.35.7
k8s-worker2    Ready    <none>          56m   v1.35.7
```

```bash
[root@k8s-master ~]# kubectl  get  nodes  -o  wide
NAME          STATUS   ROLES           AGE   VERSION   INTERNAL-IP      EXTERNAL-IP   OS-IMAGE                        KERNEL-VERSION                 CONTAINER-RUNTIME
k8s-master    Ready    control-plane   82m   v1.35.7   192.168.10.100   <none>        Rocky Linux 9.8 (Blue Onyx)     5.14.0-687.36.1.el9_8.x86_64   containerd://2.3.3
k8s-worker1   Ready    <none>          58m   v1.35.7   192.168.10.101   <none>        Rocky Linux 9.8 (Blue Onyx)     5.14.0-687.36.1.el9_8.x86_64   containerd://2.3.3
k8s-worker2   Ready    <none>          57m   v1.35.7   192.168.10.102   <none>        Rocky Linux 9.8 (Blue Onyx)     5.14.0-687.36.1.el9_8.x86_64   containerd://2.3.3
```

**정리:** 노드 상태가 모두 `Ready`인 것을 확인했으므로, 이제 쿠버네티스의 가장 작은 실행 단위인 **Pod** 개념을 살펴본다.

---

## Kubernetes Pod 개념

**Pod**란 무엇이고 왜 컨테이너를 직접 쓰지 않고 Pod 단위로 관리하는지, 그리고 Pod의 특징과 생명주기를 정리한다.

-Kubernetes에서 컨테이너를 실행하기 위한 가장 작은 단위

-**Pod**는 쿠버네티스의 기본 실행 단위 쿠버네티스에서 실제 애플리케이션이 실행되는 가장 작은 단위가 바로 Pod이다.
  - 컨테이너를 담는 상자 또는 하나의 서비스(웹 서버 등)가 동작하는 기본 단위

-**Docker**에서는 컨테이너가 직접 실행되지만, Kubernetes에서는 컨테이너가 Pod 안에서 실행된다.

-쿠버네티스는 Pod 단위로 관리한다.

-컨테이너를 직접 안 쓰고 Pod를 사용하는 이유
 1) 컨테이너를 하나의 실행 단위로 묶기 위해
  - 예: 웹 서버 + 사이드카 로그 수집기(Fluentd)
  - 이 둘은 반드시 같은 네트워크/스토리지 공유가 필요하다.
   그래서 Pod 하나 안에 넣어서 한 세트로 움직이게 만든다.
  - 쿠버네티스 스케줄링, 관리 단위가 Pod이기 때문에 쿠버네티스는 컨테이너가 아니라 Pod 단위로 배포, 복구, 이동을 관리한다.


-**Pod의 특징 3가지**

1) IP를 가진다
  - Pod는 생성되면 고유한 **Pod IP**가 하나 생긴다.
  - Pod 안의 모든 컨테이너는 이 IP를 공유한다.

2) 네임스페이스(네트워크 + 파일시스템) 공유
  - Pod 내부의 컨테이너들은 같은 네트워크(IP, 포트) 같은 localhost 같은 Volume(스토리지)를 공유한다.
  - 즉 Pod = 여러 컨테이너가 한 컴퓨터처럼 행동하는 구조

3) 사라졌다가 다시 생길 수 있다
  - Pod는 고정된 존재가 아니다.
  - 재시작되면 IP도 달라진다.
  - 장애 나면 자동 복구할 수 있다.
  - 그래서 Pod는 언제든 교체되는 일회성 실행 단위라고 보면 된다.


-Pod는 1개 이상 컨테이너를 포함할 수 있다

-Pod 내부 컨테이너 수
  - 1개 		= 가장 흔한 형태(nginx 한 개 실행)
  - 2개 이상 	= 사이드카 패턴 (로그Agent + 메인앱 등)
  - Pod가 컨테이너 모음이라고 설명하는 이유가 바로 이것이다.
  - 하지만 보통 1컨테이너 = 1개의 Pod 구조로 사용한다.


-Pod는 직접 만들지 않는다.
  - 실무에서는 거의 절대로 **kubectl run** 으로 Pod를 직접 띄우지 않는다.
   왜냐하면 Pod는 장애가 발생하면 자동 복구가 안 되고 수평 확장(Replica 증가/감소), 롤링 업데이트도 불가능하다.
  - 실무에서는 **Deployment**라는 컨트롤러가 Pod를 대신 만든다.
  - 직접 Pod 만들기 (테스트 용)
  - Deployment로 Pod 관리 (실무 방식)


-Pod는 일회성 존재이다

-Pod는 다음과 같은 특징을 가진다.
  - 고정 IP 아님 (재생성되면 IP가 바뀜)
  - 이름도 변경될 수 있음 (ReplicaSet이 새 Pod 생성)
  - 고장 나면 새 Pod가 자동으로 생성됨 (Deployment가 관리할 때)
  - 일정 주기로 재 시작될 수도 있음
  - 그래서 Pod는 고정된 서버라고 보면 안 되고 언제든 교체되는 실행 단위라고 봐야 한다.


-**Pod의 생명주기(Life Cycle)**

-Pod는 아래와 같은 상태를 거친다.

```
Pending --> ContainerCreating --> Running --> Succeeded / Failed / CrashLoopBackOff
```

-Pod가 Running이어도 내부 컨테이너는 계속 재시작될 수 있다.

**정리:** Pod는 IP를 가지고, 네임스페이스를 공유하며, 언제든 사라졌다 다시 생길 수 있는 일회성 실행 단위이다. 이제 실제로 `kubectl run` 명령으로 Pod를 직접 생성해본다.

---

## Pod 생성

**kubectl run** 명령으로 Pod를 직접 생성하고, 상태 조회·상세 정보 확인·응답 테스트·삭제까지의 흐름을 살펴본다.

```bash
[root@k8s-master ~]# kubectl  run  webserver  --image=nginx:latest  --port 80
pod/webserver created
```

```bash
[root@k8s-master ~]# kubectl get pods
NAME        READY   STATUS    RESTARTS   AGE
webserver   1/1     Running   0          19s
```

```bash
[root@k8s-master ~]# kubectl get pods  -o wide
NAME        READY   STATUS    RESTARTS   AGE     IP           NODE          NOMINATED NODE   READINESS GATES
webserver   1/1     Running   0          2m28s   10.244.2.2   k8s-worker2   <none>           <none>
```

```bash
[root@k8s-master ~]# kubectl  describe  pods  webserver
Name:             webserver
Namespace:        default
Priority:         0
Service Account:  default
Node:             k8s-worker2/192.168.10.102
Start Time:       Tue, 11 Aug 2026 16:55:38 +0900
Labels:           run=webserver
Annotations:      <none>
Status:           Running
IP:               10.244.2.2
IPs:
  IP:  10.244.2.2
Containers:
  webserver:
    Container ID:   containerd://ad56aba02ea28c6eb77cea71a14247eaa36c490efbf84e0bd968025553223522
    Image:          nginx:latest
    Image ID:       docker.io/library/nginx@sha256:8541484afbc9c8a5a8a99b379568ebbc957f658583ec9448fc43104229c03cf8
    Port:           80/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Tue, 11 Aug 2026 16:55:47 +0900
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-b89xh (ro)
Conditions:
  Type                        Status
  PodReadyToStartContainers   True
  Initialized                 True
  Ready                       True
  ContainersReady             True
  PodScheduled                True
Volumes:
  kube-api-access-b89xh:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    Optional:                false
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age    From               Message
  ----    ------     ----   ----               -------
  Normal  Scheduled  4m     default-scheduler  Successfully assigned default/webserver to k8s-worker2
  Normal  Pulling    3m59s  kubelet            spec.containers{webserver}: Pulling image "nginx:latest"
  Normal  Pulled     3m51s  kubelet            spec.containers{webserver}: Successfully pulled image "nginx:latest" in 7.815s (7.815s including waiting). Image size: 63135215 bytes.
  Normal  Created    3m51s  kubelet            spec.containers{webserver}: Container created
  Normal  Started    3m51s  kubelet            spec.containers{webserver}: Container started
```

```bash
[root@k8s-master ~]# kubectl get pods
NAME        READY   STATUS    RESTARTS   AGE
webserver   1/1     Running   0          13m

[root@k8s-master ~]# kubectl get pods  webserver
NAME        READY   STATUS    RESTARTS   AGE
webserver   1/1     Running   0          13m
```

```bash
[root@k8s-master ~]# kubectl  run  webhttp  --image=httpd  --port 8080
pod/webhttp created

[root@k8s-master ~]# kubectl get pods
NAME        READY   STATUS              RESTARTS   AGE
webhttp     0/1     ContainerCreating   0          7s
webserver   1/1     Running             0          14m

[root@k8s-master ~]# kubectl get pods
NAME        READY   STATUS    RESTARTS   AGE
webhttp     1/1     Running   0          38s
webserver   1/1     Running   0          15m
```

```bash
[root@k8s-master ~]# kubectl get svc
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   133m

[root@k8s-master ~]# kubectl get pods -o wide
NAME        READY   STATUS    RESTARTS   AGE   IP           NODE          NOMINATED NODE   READINESS GATES
webhttp     1/1     Running   0          25m   10.244.1.2   k8s-worker1   <none>           <none>
webserver   1/1     Running   0          40m   10.244.2.2   k8s-worker2   <none>           <none>
```

### curl 로 Pod 응답 확인

생성된 Pod의 IP로 **curl**을 실행해 nginx 웹 서버가 정상 응답하는지 확인한다.

```bash
[root@k8s-master ~]# curl http://10.244.2.2
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

### elinks 설치 및 사용

CLI 환경에서 웹 페이지 응답을 확인하기 위한 텍스트 기반 웹 브라우저 **elinks**를 설치하고 사용하는 과정이다.

```bash
# EPEL(Extra Packages for Enterprise Linux) 저장소를 설치한다.
# Rocky Linux 기본 저장소에 없는 추가 패키지를 설치할 수 있도록 저장소를 추가한다.
[root@k8s-master ~]# dnf install -y epel-release

# 활성화된 Repository의 패키지 메타데이터를 내려받아 로컬 캐시를 생성/갱신한다.
# 이후 패키지를 검색하거나 설치할 때 저장소 정보를 빠르게 사용할 수 있다.
[root@k8s-master ~]# dnf makecache

# 텍스트 기반 웹 브라우저인 elinks를 설치한다.
# GUI가 없는 CLI 환경에서 웹 페이지에 접속하여 HTTP 서비스의 동작을 확인할 때 사용할 수 있다.
[root@k8s-master ~]# dnf install -y elinks

[root@k8s-master ~]# elinks 10.244.1.2
[root@k8s-master ~]# elinks 10.244.2.2
```

### Pod 삭제

`kubectl delete pods` 명령으로 생성했던 webhttp, webserver Pod를 삭제한다.

```bash
[root@k8s-master ~]# kubectl  get  pods
NAME        READY   STATUS    RESTARTS   AGE
webhttp     1/1     Running   0          33m
webserver   1/1     Running   0          48m

[root@k8s-master ~]# kubectl  delete  pods  webhttp	# webhttp 이름의 파드 삭제
pod "webhttp" deleted from default namespace

[root@k8s-master ~]# kubectl  delete  pods  webserver	# webserver 이름의 파드 삭제
pod "webserver" deleted from default namespace

[root@k8s-master ~]# kubectl  get  pods
No resources found in default namespace.
```

**정리:** `kubectl run`으로 Pod를 직접 생성/삭제하는 방법을 확인했다. 하지만 실무에서는 Pod를 직접 다루지 않고 **Deployment** 컨트롤러로 관리하므로, 다음 섹션에서 이를 다룬다.

---

## Deployment로 Pod 관리

**Deployment**를 사용해 여러 개의 Pod를 자동으로 생성하고, 장애 시 자동 복구되며, replicas 수를 동적으로 조정하는 방법을 다룬다.

```bash
[root@k8s-master ~]# kubectl  create  deployment  sol-deploy  --image=nginx:latest  --replicas=3
deployment.apps/sol-deploy created
```

-**deployment**
  - 리소스 종류(Type)
  - Deployment는 다음 기능을 가진 가장 중요한 쿠버네티스 컨트롤러
  - Pod 자동 생성
  - Pod 개수 유지(replica 관리)
  - Pod 자동 복구
  - 롤링업데이트
  - 롤백
  - 즉, 여러 개의 동일한 파드를 관리하는 상위 관리자

-sol-deploy
  - Deployment의 이름(Name)
  - 리소스를 식별하는 고유 이름
  - kubectl 명령에서 sol-deploy 이라고 부르면 해당 Deployment만 조작
  - EX) kubectl get deployment sol-deploy
  - EX) kubectl delete deployment sol-deploy

--image=httpd:latest
  - Pod 내부에서 실행할 컨테이너 이미지 지정
  - httpd:latest = Apache 웹서버 최신 버전 이미지 사용
  - Deployment는 내부적으로 Pod 템플릿을 만들면서 컨테이너 image를 httpd:latest로 설정
  - Pod 3개가 생성되면 각 Pod 안에서 httpd 컨테이너가 실행됨

--replicas=3
  - Deployment가 생성해야 할 Pod 개수(Replica 수) 지정
  - replicas=3 : 항상 3개의 Pod를 유지하라는 뜻
  - Pod 하나가 죽으면 새로운 Pod 자동 생성
  - 노드가 죽어도 다른 노드에 다시 배치
  - 확장이 필요하면 --replicas 변경 가능

```bash
[root@k8s-master ~]# kubectl get pods
NAME                          READY   STATUS    RESTARTS   AGE
sol-deploy-956c888c6-c9hng    1/1     Running   0          11s
sol-deploy-956c888c6-flnf6    1/1     Running   0          11s
sol-deploy-956c888c6-kcv2w    1/1     Running   0          11s

[root@k8s-master ~]# kubectl get pods  -o wide
NAME                          READY   STATUS    RESTARTS   AGE     IP           NODE          NOMINATED NODE   READINESS GATES
sol-deploy-956c888c6-c9hng    1/1     Running   0          4m22s   10.244.1.4   k8s-worker1   <none>           <none>
sol-deploy-956c888c6-flnf6    1/1     Running   0          5m59s   10.244.2.3   k8s-worker2   <none>           <none>
sol-deploy-956c888c6-kcv2w    1/1     Running   0          5m59s   10.244.1.3   k8s-worker1   <none>           <none>

[root@k8s-master ~]# kubectl  get  deployments
NAME         READY   UP-TO-DATE   AVAILABLE   AGE
sol-deploy   3/3     3            3           8m16s
```

```
NAME		: Deployment(배포)의 이름
READY		: 현재 Running 중인 파드 / 목표 파드 수
UP-TO-DATE	: 최신 버전으로 업데이트된 파드 수
AVAILABLE	: 외부 요청을 받을 준비가 된 파드 수, 건강한(Healthy) 파드의 개수
AGE		: Deployment가 생성된 시간
```

```bash
[root@k8s-master ~]# kubectl get pods  sol-deploy-956c888c6-c9hng
NAME                         READY   STATUS    RESTARTS   AGE
sol-deploy-956c888c6-c9hng   1/1     Running   0          8m28s

[root@k8s-master ~]# kubectl get pods  sol-deploy-956c888c6-c9hng  -o  wide
NAME                         READY   STATUS    RESTARTS   AGE     IP           NODE          NOMINATED NODE   READINESS GATES
sol-deploy-956c888c6-c9hng   1/1     Running   0          8m54s   10.244.1.4   k8s-worker1   <none>           <none>
```

### YAML 파일로 Deployment 생성

위의 `kubectl create deployment` 명령형(imperative) 방식과 동일한 결과를 선언형(declarative) YAML 매니페스트로도 만들 수 있다. 실무에서는 버전 관리(Git)와 재사용성 때문에 YAML 파일 방식을 더 많이 사용한다.

```yaml
# sol-deploy.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: sol-deploy
  labels:
    app: sol-deploy
spec:
  replicas: 3
  selector:
    matchLabels:
      app: sol-deploy
  template:
    metadata:
      labels:
        app: sol-deploy
    spec:
      containers:
      - name: sol-deploy
        image: nginx:latest
        ports:
        - containerPort: 80
```

```bash
[root@k8s-master ~]# kubectl apply -f sol-deploy.yaml
deployment.apps/sol-deploy created

[root@k8s-master ~]# kubectl get deployments
NAME         READY   UP-TO-DATE   AVAILABLE   AGE
sol-deploy   3/3     3            3           8s

[root@k8s-master ~]# kubectl get pods
NAME                          READY   STATUS    RESTARTS   AGE
sol-deploy-956c888c6-c9hng    1/1     Running   0          8s
sol-deploy-956c888c6-flnf6    1/1     Running   0          8s
sol-deploy-956c888c6-kcv2w    1/1     Running   0          8s
```

- `spec.replicas` : 유지할 Pod 개수 (`--replicas` 옵션과 동일)
- `spec.selector.matchLabels` : Deployment가 관리할 Pod를 찾는 라벨 (`spec.template.metadata.labels`와 반드시 일치해야 함)
- `spec.template` : 실제 생성될 Pod의 스펙 (컨테이너 이미지, 포트 등)

### Pod 하나 삭제 → 자동 재생성 확인

Deployment가 관리하는 Pod 중 하나를 삭제했을 때 자동으로 새 Pod가 생성되어 replicas 수가 유지되는지 확인한다.

```bash
[root@k8s-master ~]# kubectl get pods
NAME                          READY   STATUS    RESTARTS   AGE
sol-deploy-956c888c6-c9hng    1/1     Running   0          11s
sol-deploy-956c888c6-flnf6    1/1     Running   0          11s
sol-deploy-956c888c6-kcv2w    1/1     Running   0          11s

[root@k8s-master ~]# kubectl  delete  pods  sol-deploy-956c888c6-c9hng
pod "sol-deploy-956c888c6-c9hng" deleted from default namespace

[root@k8s-master ~]# kubectl get pods
NAME                          READY   STATUS              RESTARTS   AGE
sol-deploy-956c888c6-flnf6    1/1     Running             0          11m
sol-deploy-956c888c6-kcv2w    1/1     Running             0          11m
sol-deploy-956c888c6-pthxd    0/1     ContainerCreating   0          3s
```

Pod를 삭제해도 Deployment가 즉시 새 Pod(pthxd)를 생성하여 replicas=3을 유지한다.

### kubectl edit 으로 replicas 변경 (3 → 5)

**kubectl edit** 명령으로 Deployment 스펙을 직접 열어 replicas 값을 3에서 5로 수정해 스케일 아웃한다.

```bash
[root@k8s-master ~]# kubectl  edit  deployments.apps sol-deploy
```

```yaml
# reopened with the relevant failures.
#
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    deployment.kubernetes.io/revision: "1"
  creationTimestamp: "2026-08-11T08:47:53Z"
  generation: 1
  labels:
    app: sol-deploy
  name: sol-deploy
  namespace: default
  resourceVersion: "14719"
  uid: 83f4e5e4-4792-4176-82ce-f39aa2cd41fe
spec:
  progressDeadlineSeconds: 600
  replicas: 5			# replicas를 3에서 5로 수정
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app: sol-deploy
  strategy:
```

저장(:wq) 후:

```bash
[root@k8s-master ~]# kubectl get pods
NAME                          READY   STATUS              RESTARTS   AGE
sol-deploy-956c888c6-bk8hc    0/1     ContainerCreating   0          2s		# pod 생성
sol-deploy-956c888c6-bvbqc    0/1     ContainerCreating   0          2s		# pod 생성
sol-deploy-956c888c6-flnf6    1/1     Running             0          17m
sol-deploy-956c888c6-kcv2w    1/1     Running             0          17m
sol-deploy-956c888c6-pthxd    1/1     Running             0          5m42s
```

**정리:** replicas 값을 수정하는 것만으로 Deployment가 Pod 개수를 즉시 목표 수치에 맞춰 조정하는 것을 확인했다.

### Deployment 삭제

Deployment를 삭제하면 그것이 관리하던 모든 Pod도 함께 제거된다.

```bash
[root@k8s-master ~]# kubectl  delete  deployments.apps  sol-deploy
deployment.apps "sol-deploy" deleted from default namespace

[root@k8s-master ~]# kubectl get pods
No resources found in default namespace
```

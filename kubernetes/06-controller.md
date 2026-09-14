# Kubernetes Controller

## 파드 3개 요청시 쿠버네티스 동작 순서

상황: 사용자가 파드 3개를 유지해라라고 요청 (Deployment / ReplicaSet 생성). 쿠버네티스에는 이렇게 원하는 상태를 계속 유지하도록 감시·복구하는 여러 컨트롤러가 있으며, Deployment는 무중단 배포와 Rollback을, DaemonSet은 모든 노드에 하나씩 배치되는 것을, StatefulSet은 고유 이름과 전용 디스크가 필요한 인스턴스를, Job과 CronJob은 한 번 또는 주기적으로 실행되고 끝나는 작업을 각각 담당한다.

### 1단계. API Server

- 사용자의 요청(kubectl apply 등)을 API Server가 가장 먼저 받는다.
- 요청 내용을 검증한다.
- 파드 3개가 필요하다는 정보를 etcd에 저장한다.

핵심
- API Server는 명령을 전달하고 기록만 한다.
- 직접 파드를 만들지는 않는다.

### 2단계. Controller

- 컨트롤러는 API Server를 계속 감시하고 있다.
- etcd에 저장된 정보를 보고 파드가 3개 필요하지만 아직 없다는 것을 확인한다. 그래서 API Server에게 Pod 객체 3개를 생성해 달라고 요청한다.

핵심
- 컨트롤러는 pod 개수 유지가 역할이다.
- 노드 배치나 실행은 하지 않는다.

### 3단계. API Server

- 컨트롤러의 요청을 받아 Pod 객체 3개를 생성한다.
- 생성된 Pod 정보는 etcd에 저장된다.
- 이 시점의 파드는 아직 어느 노드에도 배치되지 않았다.

### 4단계. Scheduler

- 스케줄러는 nodeName이 없는 Pod를 발견한다.
- etcd에 있는 노드 상태(리소스, 사용량 등)를 참고한다.

판단 결과
- 파드 2개 : 노드1
- 파드 1개 : 노드2

- Pod 객체에 nodeName을 기록한다 (배치 결정).

핵심
- 스케줄러는 배치만 결정한다.
- 실제 실행은 하지 않는다.

### 5단계. API Server

- 스케줄러가 기록한 배치 결과를 etcd에 저장한다.
- 이제 해당 파드들은 어느 노드에서 실행할지가 확정된 상태다.

### 6단계. kubelet (실제 실행 주체)

- 각 노드의 kubelet은 API Server를 계속 감시한다.
- 자기 노드로 지정된 Pod를 발견한다.
- 노드1 kubelet : 파드 2개 실행
- 노드2 kubelet : 파드 1개 실행
- 컨테이너 런타임에게 요청해서 이미지 다운로드 --> 컨테이너 생성 --> 실행을 수행한다.
- 실제로 파드를 만드는 주체는 kubelet이다.

---

## Kubernetes Controller

- **Controller**는 쿠버네티스에서 현재 상태를 감시하고, 원하는 상태(desired state)로 맞추기 위해 자동으로 조정하는 관리자 역할이다.
- 쿠버네티스는 사람이 직접 하나하나 관리하는 시스템이 아니라 원하는 상태만 선언하면, 나머지는 컨트롤러가 자동으로 맞춰주는 시스템이다.
- 이 자동 조정의 핵심 주체가 바로 Controller이다.

### 왜 Controller가 필요한가?

- 예를 들어 다음과 같은 상황을 생각해보자.
- Pod를 3개 유지하고 싶다 그런데 그 중 1개가 죽었다
- 사람이 직접 확인하고 다시 Pod를 만들어야 할까?
- 쿠버네티스에서는 사람이 개입하지 않아도 Controller가 자동으로 Pod를 생성/수정/삭제를 자동으로 진행한다.

Controller의 역할은 다음과 같다.
- 상태 감시 (watch)
- 차이 발견 (현재 상태 vs 원하는 상태)
- 자동 복구 (create / delete / restart)

### Controller의 기본 동작 개념

Controller는 항상 아래 구조로 동작한다.

1단계) 사용자가 YAML로 원하는 상태를 선언한다. (예: Pod를 3개 유지하겠다)

2단계) Controller가 현재 상태를 계속 감시한다 (예: 지금 Pod가 몇 개인지 확인)

3단계) 현재 상태와 원하는 상태를 비교한다

4단계) 차이가 있으면 자동으로 조정한다
- 부족하면 생성
- 많으면 삭제
- 죽었으면 재생성
- 이 과정을 무한 반복한다.

이 구조를 **Reconciliation Loop**(상태 동기화 루프)라고 부른다.

### Controller가 관리하는 대표적인 대상

Controller는 보통 직접 Pod를 관리하지 않고, Pod를 관리하는 상위 리소스를 관리한다.

대표적인 Controller들
- ReplicationController
- ReplicaSet Controller
- Deployment Controller
- DaemonSet Controller
- StatefulSet Controller
- Job / CronJob Controller

정리: Controller는 사람이 매번 개입하지 않아도 감시-비교-조정을 반복하는 Reconciliation Loop로 원하는 상태를 유지하며, 이 개념을 실제로 구현한 첫 사례가 다음의 ReplicationController다.

---

## ReplicationController (RC)

- **ReplicationController**는 Pod의 개수를 항상 일정하게 유지해주는 Controller이다. 즉, Pod가 죽거나 삭제되면 자동으로 다시 생성해서 지정한 개수(**replicas**)를 보장한다.

### ReplicationController의 역할

ReplicationController는 다음을 보장한다.
- Pod가 지정한 개수만큼 항상 존재
- Pod가 죽으면 자동 재생성
- Pod를 수동으로 삭제해도 다시 생성

중요한 점: ReplicationController는 Pod 자체를 직접 감시하는 것이 아니라 Label을 기준으로 Pod를 관리한다.

### Label 기반 관리 개념

ReplicationController는 아래 두 가지를 항상 확인한다.
- 몇 개를 유지할 것인가? (**replicas**)
- 어떤 Pod를 관리할 것인가? (**selector**)

예시 개념
```
label: app=web
replicas: 3
```

의미
- app=web 라벨이 붙은 Pod를
- 항상 3개 유지하겠다

만약 app=web Pod가 2개 --> 1개 생성
만약 app=web Pod가 4개 --> 1개 삭제

### ReplicationController 동작 흐름

1단계) ReplicationController 생성 / replicas 값 설정 / selector(label) 설정

2단계) Controller가 현재 Pod 개수 확인

3단계) 원하는 개수와 비교

4단계) 부족하면 Pod 생성 / 초과하면 Pod 삭제 — 이 과정을 계속 반복한다.

장애 상황 예시
- replicas = 3
- 현재 Pod = 3

문제 발생: Pod 1개가 노드 장애로 종료됨

ReplicationController 동작
- 현재 Pod 수 = 2
- 원하는 Pod 수 = 3
- Pod 1개 자동 생성

결과: 사용자는 아무것도 하지 않았지만 Pod는 다시 3개가 된다.

### ReplicationController YAML

기본 구성 요소
- replicas : 유지해야 하는 Pod 개수
- selector : ReplicationController가 관리할 Pod를 선택하는 기준 (라벨)
- template : Pod가 부족할 때 새로 생성할 Pod의 설계도

**replicas**
- 항상 몇 개를 켜두고 있을지 목표 개수다.
- ReplicationController는 실제 개수와 replicas를 계속 비교한다.

동작 예시
- replicas: 3 인데 현재 Pod가 2개면 → 1개를 더 만든다.
- replicas: 3 인데 현재 Pod가 4개면 → 1개를 삭제한다.

**selector**
- 이 RC가 어떤 Pod들을 내 관리 대상으로 볼 것인가를 고르는 필터다.
- 라벨(key: value)이 일치하는 Pod를 관리 대상으로 삼는다.

예시
```
replicas: 3

selector:
  app: web
```

ReplicationController는 현재 실행 중인 Pod들 중에서 라벨이 app: web 인 Pod만을 관리 대상으로 삼는다. 이 관리 대상 Pod의 개수를 지속적으로 확인하면서, 개수가 3개보다 적으면 template에 정의된 설정을 사용해 Pod를 추가로 생성하고, 개수가 3개보다 많으면 초과된 Pod를 삭제한다. 이 과정을 반복함으로써 항상 app: web 라벨을 가진 Pod가 3개 유지되도록 동작한다.

중요한 포인트(중요)
- selector 조건이 너무 넓으면 위험하다.
- 예: app=web 인 Pod가 이미 다른 용도로 존재하는데 RC selector가 app=web이면, RC가 그 Pod까지 자기 관리 대상으로 착각할 수 있다.
- 그래서 보통 RC/RS/Deploy에서 라벨을 목적별로 명확히 잡는다.

**template : Pod가 부족할 때 새로 생성할 Pod의 설계도**
- RC가 Pod를 새로 만들어야 할 때 참고하는 Pod 생성 템플릿이다.

template 안에 들어가는 것
- metadata.labels : 새로 만들어질 Pod에 붙일 라벨
- spec.containers : 컨테이너 이름, 이미지, 포트, 환경변수, 볼륨 등 Pod 실행 정보

가장 중요한 규칙
- template.metadata.labels는 selector와 반드시 맞아야 한다.
- 왜냐하면 RC가 Pod를 만들었는데 라벨이 selector와 다르면, RC가 내가 만든 Pod를 자기 Pod로 인식하지 못한다.
- RC는 Pod가 부족하다고 판단해서 계속 새 Pod를 만들 수 있다(무한 생성 상황)

```yaml
apiVersion: v1
kind: ReplicationController

metadata:
  name: <RC 이름>

spec:
  replicas: 3

  selector:
    app: web		# <--- 반드시 같은 값을 갖아야 한다. (값이 다를 경우 pod를 무한 생성할 수 있다.)

  template:
    metadata:
      labels:
        app: web	# <--- 반드시 같은 값을 갖아야 한다. (값이 다를 경우 pod를 무한 생성할 수 있다.)

    spec:
      containers:
      - name: nginx-container
        image: nginx:latest
        ports:
        - containerPort: 80
```

### RC 실습 — 생성/확인

```bash
[root@k8s-master ~]# vi rc-nginx.yaml
```
```yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: rc-nginx

spec:
  replicas: 3
  selector:
    app: webui

  template:
    metadata:
      name: nginx-pod
      labels:
        app: webui

    spec:
      containers:
      - name: nginx-container
        image: nginx:1.31
        ports:
        - containerPort: 80
```

```bash
	# 터미널 2
[root@k8s-master ~]# kubectl  apply  -f  rc-nginx.yaml  --dry-run=server
replicationcontroller/rc-nginx created (server dry run)

	# 터미널 1
[root@k8s-master ~]# kubectl  get  pods  -o  wide  --watch

	# 터미널 2
[root@k8s-master ~]# kubectl  apply  -f  rc-nginx.yaml
replicationcontroller/rc-nginx created

	# 터미널 1
[root@k8s-master ~]# kubectl  get  pods  -o  wide  --watch
NAME              READY   STATUS              RESTARTS   AGE   IP            NODE          NOMINATED NODE   READINESS GATES
rc-nginx-fq9sk    0/1     Pending             0          0s    <none>        <none>        <none>            <none>
rc-nginx-ggjvk    0/1     Pending             0          0s    <none>        <none>        <none>            <none>
rc-nginx-nlp42    0/1     Pending             0          0s    <none>        <none>        <none>            <none>
rc-nginx-fq9sk    0/1     Pending             0          0s    <none>        k8s-worker1   <none>            <none>
rc-nginx-nlp42    0/1     Pending             0          0s    <none>        k8s-worker1   <none>            <none>
rc-nginx-ggjvk    0/1     Pending             0          0s    <none>        k8s-worker2   <none>            <none>
rc-nginx-fq9sk    0/1     ContainerCreating   0          0s    <none>        k8s-worker1   <none>            <none>
rc-nginx-ggjvk    0/1     ContainerCreating   0          0s    <none>        k8s-worker2   <none>            <none>
rc-nginx-nlp42    0/1     ContainerCreating   0          0s    <none>        k8s-worker1   <none>            <none>
rc-nginx-ggjvk    1/1     Running             0          1s    10.244.2.3    k8s-worker2   <none>            <none>
rc-nginx-nlp42    1/1     Running             0          1s    10.244.1.6    k8s-worker1   <none>            <none>
rc-nginx-fq9sk    1/1     Running             0          1s    10.244.1.5    k8s-worker1   <none>            <none>

[root@k8s-master ~]# kubectl  get  pods
NAME             READY   STATUS    RESTARTS   AGE
rc-nginx-fq9sk   1/1     Running   0          3m19s
rc-nginx-ggjvk   1/1     Running   0          3m19s
rc-nginx-nlp42   1/1     Running   0          3m19s

[root@k8s-master ~]# kubectl  get  replicationcontrollers
NAME       DESIRED   CURRENT   READY   AGE
rc-nginx   3         3         3       3m56

[root@k8s-master ~]# kubectl  get  rc
NAME       DESIRED   CURRENT   READY   AGE
rc-nginx   3         3         3       3m5
```

- DESIRED : 사용자가 원하는 Pod 개수 (3)
- CURRENT : 현재 실제로 존재하는 Pod 개수 (3)
- READY : 현재 정상적으로 서비스 가능한 Pod 개수 (3)

```bash
[root@k8s-master ~]# kubectl  describe  rc  rc-nginx
Name:         rc-nginx
Namespace:    default
Selector:     app=webui
Labels:       app=webui
Annotations:  <none>
Replicas:     3 current / 3 desired
Pods Status:  3 Running / 0 Waiting / 0 Succeeded / 0 Failed
Pod Template:
  Labels:  app=webui
  Containers:
   nginx-container:
    Image:          nginx:1.31
    Port:           80/TCP
    Host Port:      0/TCP
    Environment:    <none>
    Mounts:         <none>
  Volumes:          <none>
  Node-Selectors:   <none>
  Tolerations:      <none>
Events:
  Type     Reason              Age     From                     Message
  ----     ------              ----    ----                     -------
  Normal   SuccessfulCreate    6m37s   replication-controller   Created pod: rc-nginx-fq9sk
  Normal   SuccessfulCreate    6m37s   replication-controller   Created pod: rc-nginx-nlp42
  Normal   SuccessfulCreate    6m37s   replication-controller   Created pod: rc-nginx-ggjvk

[root@k8s-master ~]# kubectl  get  pods  --show-labels
NAME             READY   STATUS    RESTARTS   AGE     LABELS
rc-nginx-fq9sk   1/1     Running   0          9m41s   app=webui
rc-nginx-ggjvk   1/1     Running   0          9m41s   app=webui
rc-nginx-nlp42   1/1     Running   0          9m41s   app=webui

[root@k8s-master ~]# kubectl  run  httpd-web  --image=httpd:latest  --labels=app=webui  -o  yaml  >  rc-httpd.yaml

[root@k8s-master ~]# ls  -l  rc-httpd.yaml
-rw-r--r-- 1 root root 1583  8월 18 12:12 rc-httpd.yaml

[root@k8s-master ~]# vi  rc-httpd.yaml  (수정 버전)
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: webui
  name: httpd-web
  namespace: default
spec:
  containers:
  - image: httpd:latest
    name: httpd-web
```

```bash
	# 터미널 2
[root@k8s-master ~]# kubectl  apply  -f  rc-httpd.yaml  --dry-run=server
pod/httpd-web created (server dry run)

	# 터미널 1
[root@k8s-master ~]# kubectl  get  pods  -o  wide  --watch

	# 터미널 2
[root@k8s-master ~]# kubectl  apply  -f  rc-httpd.yaml
pod/httpd-web created

	# 터미널 1
[root@k8s-master ~]# kubectl  get  pods  -o  wide  --watch
NAME              READY   STATUS              RESTARTS   AGE   IP            NODE          NOMINATED NODE   READINESS GATES
rc-nginx-fq9sk    0/1     Pending             0          0s    <none>        <none>        <none>            <none>
rc-nginx-ggjvk    0/1     Pending             0          0s    <none>        <none>        <none>            <none>
rc-nginx-nlp42    0/1     Pending             0          0s    <none>        <none>        <none>            <none>
rc-nginx-fq9sk    0/1     Pending             0          0s    <none>        k8s-worker1   <none>            <none>
rc-nginx-nlp42    0/1     Pending             0          0s    <none>        k8s-worker1   <none>            <none>
rc-nginx-ggjvk    0/1     Pending             0          0s    <none>        k8s-worker2   <none>            <none>
rc-nginx-fq9sk    0/1     ContainerCreating   0          0s    <none>        k8s-worker1   <none>            <none>
rc-nginx-ggjvk    0/1     ContainerCreating   0          0s    <none>        k8s-worker2   <none>            <none>
rc-nginx-nlp42    0/1     ContainerCreating   0          0s    <none>        k8s-worker1   <none>            <none>
rc-nginx-ggjvk    1/1     Running             0          1s    10.244.2.3    k8s-worker2   <none>            <none>
rc-nginx-nlp42    1/1     Running             0          1s    10.244.1.6    k8s-worker1   <none>            <none>
rc-nginx-fq9sk    1/1     Running             0          1s    10.244.1.5    k8s-worker1   <none>            <none>

httpd-web         0/1     Pending             0          0s    <none>        <none>        <none>            <none>
httpd-web         0/1     Pending             0          0s    <none>        k8s-worker2   <none>            <none>
httpd-web         0/1     ContainerCreating   0          0s    <none>        k8s-worker2   <none>            <none>
httpd-web         0/1     Terminating         0          0s    <none>        k8s-worker2   <none>            <none>
httpd-web         0/1     Terminating         0          1s    <none>        k8s-worker2   <none>            <none>
```

- pod를 3개만 유지해야하는데 같은 LABEL로 pod를 생성하게되면 rc-controller에 의해 바로 삭제된다.

### RC pod 개수 수정 (edit)

```bash
[root@k8s-master ~]# kubectl  edit  rc  rc-nginx
  namespace: default
  resourceVersion: "167372"
  uid: 5e20c4e1-df53-4e74-8259-387ed5594e44
spec:
  replicas: 5	# 3에서 5로 수정
  selector:
    app: webui
  template:
    metadata:
      labels:
        app: webui
      name: nginx-pod
    spec:
      containers:
      - image: nginx:1.31
        imagePullPolicy: IfNotPresent
        name: nginx-container
        ports:
        - containerPort: 80
          protocol: TCP
        resources: {}
~~~~~~~~~~~~ 중간 생략 ~~~~~~~~~~~~

	# 터미널 1
[root@k8s-master ~]# kubectl  get  pods  -o  wide  --watch
~~ (RC1~3 실행 완료) ~~

rc-nginx-2sn7n   0/1     Pending             0          0s    <none>        <none>        <none>            <none>
rc-nginx-gr57d   0/1     Pending             0          0s    <none>        <none>        <none>            <none>
rc-nginx-2sn7n   0/1     Pending             0          0s    <none>        k8s-worker2   <none>            <none>
rc-nginx-gr57d   0/1     Pending             0          0s    <none>        k8s-worker1   <none>            <none>
rc-nginx-2sn7n   0/1     ContainerCreating   0          0s    <none>        k8s-worker2   <none>            <none>
rc-nginx-gr57d   0/1     ContainerCreating   0          0s    <none>        k8s-worker1   <none>            <none>
rc-nginx-gr57d   1/1     Running             0          1s    10.244.1.7    k8s-worker1   <none>            <none>
rc-nginx-2sn7n   1/1     Running             0          1s    10.244.2.4    k8s-worker2   <none>            <none>

[root@k8s-master ~]# kubectl  get  pods  --show-labels
NAME             READY   STATUS    RESTARTS   AGE    LABELS
rc-nginx-2sn7n   1/1     Running   0          2m8s    app=webui
rc-nginx-fq9sk   1/1     Running   0          36m     app=webui
rc-nginx-ggjvk   1/1     Running   0          36m     app=webui
rc-nginx-gr57d   1/1     Running   0          2m8s    app=webui
rc-nginx-nlp42   1/1     Running   0          36m     app=webui
```

### RC pod 개수 수정 (scale 명령어)

```bash
[root@k8s-master ~]# kubectl  scale  rc  rc-nginx  --replicas=2

	# 터미널 1
[root@k8s-master ~]# kubectl  get  pods  -o  wide  --watch
~~~~~~~~~~~~ 중간 생략 ~~~~~~~~~~~~
rc-nginx-gr57d   1/1     Terminating   0   4m3s   10.244.1.7   k8s-worker1   <none>   <none>
rc-nginx-2sn7n   1/1     Terminating   0   4m3s   10.244.2.4   k8s-worker2   <none>   <none>
rc-nginx-fq9sk   1/1     Terminating   0   38m    10.244.1.5   k8s-worker1   <none>   <none>
rc-nginx-gr57d   1/1     Terminating   0   4m3s   10.244.1.7   k8s-worker1   <none>   <none>
rc-nginx-2sn7n   1/1     Terminating   0   4m3s   10.244.2.4   k8s-worker2   <none>   <none>
rc-nginx-fq9sk   1/1     Terminating   0   38m    10.244.1.5   k8s-worker1   <none>   <none>
rc-nginx-2sn7n   0/1     Completed     0   4m3s   10.244.2.4   k8s-worker2   <none>   <none>
rc-nginx-gr57d   0/1     Completed     0   4m3s   10.244.1.7   k8s-worker1   <none>   <none>
rc-nginx-fq9sk   0/1     Completed     0   38m    10.244.1.5   k8s-worker1   <none>   <none>
rc-nginx-2sn7n   0/1     Completed     0   4m3s   10.244.2.4   k8s-worker2   <none>   <none>
rc-nginx-2sn7n   0/1     Completed     0   4m3s   10.244.2.4   k8s-worker2   <none>   <none>
rc-nginx-fq9sk   0/1     Completed     0   38m    10.244.1.5   k8s-worker1   <none>   <none>
rc-nginx-fq9sk   0/1     Completed     0   38m    10.244.1.5   k8s-worker1   <none>   <none>
rc-nginx-gr57d   0/1     Completed     0   4m3s   10.244.1.7   k8s-worker1   <none>   <none>
rc-nginx-gr57d   0/1     Completed     0   4m3s   10.244.1.7   k8s-worker1   <none>   <none>

[root@k8s-master ~]# kubectl  get  pods  --show-labels
NAME             READY   STATUS    RESTARTS   AGE    LABELS
rc-nginx-ggjvk   1/1     Running   0          36m    app=webui
rc-nginx-nlp42   1/1     Running   0          36m    app=webui
```

- 특정 시간대에 트래픽이 몰리면 pod를 확장하고 특정 시간대에 트래픽이 줄어들면 축소할 수 있다.

### RC 이미지 버전 변경은 반영되지 않는다

```bash
[root@k8s-master ~]# kubectl  edit  rc  rc-nginx
~~~~~~~~~~~~ 중간 생략 ~~~~~~~~~~~~
spec:
  replicas: 2
  selector:
    app: webui
  template:
    metadata:
      labels:
        app: webui
      name: nginx-pod
    spec:
      containers:
      - image: nginx:1.31		<--- 현재 이미지 = nginx:1.31  -->  nginx:1.31.3으로 수정
        imagePullPolicy: IfNotPresent
        name: nginx-container
        ports:
        - containerPort: 80
          protocol: TCP
        resources: {}
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
~~~~~~~~~~~~ 중간 생략 ~~~~~~~~~~~~
:wq

	# 터미널 1
[root@k8s-master ~]# kubectl  get  pods  -o  wide  --watch
NAME             READY   STATUS    RESTARTS   AGE   IP       NODE     NOMINATED NODE   READINESS GATES
rc-nginx-ggjvk   0/1     Pending   0           0s    <none>   <none>   <none>            <none>
rc-nginx-nlp42   0/1     Pending   0           0s    <none>   <none>   <none>            <none>

[root@k8s-master ~]# kubectl describe pod rc-nginx-ggjvk
Name:             rc-nginx-ggjvk
Namespace:        default
Priority:         0
Service Account:  default
Node:             k8s-worker2/192.168.10.102
Start Time:       Tue, 18 Aug 2026 12:00:23 +0900
Labels:           app=webui
Annotations:      <none>
Status:           Running
IP:               10.244.2.3
IPs:
  IP:           10.244.2.3
Controlled By:  ReplicationController/rc-nginx
Containers:
  nginx-container:
    Container ID:  containerd://5f3d6f6ee1d25afd61e71caf18176f36ac7904f0be62054563e5a00959242e57
    Image:          nginx:1.31
    Image ID:       docker.io/library/nginx@sha256:8541484afbc9c8a5a8a99b379568ebbc957f658583ec9448fc43104229c03cf8
    Port:           80/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Tue, 18 Aug 2026 12:00:24 +0900
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-zd69m (ro)
Conditions:
  Type                        Status
  PodReadyToStartContainers   True
  Initialized                 True
  Ready                       True
  ContainersReady             True
  PodScheduled                True
Volumes:
  kube-api-access-zd69m:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    Optional:                false
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:               <none>
Tolerations:                  node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                               node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  44m   default-scheduler  Successfully assigned default/rc-nginx-ggjvk to k8s-worker2
  Normal  Pulled     44m   kubelet            spec.containers{nginx-container}: Container image "nginx:1.31" already present on machine and can be accessed by the pod
  Normal  Created    44m   kubelet            spec.containers{nginx-container}: Container created
  Normal  Started    44m   kubelet            spec.containers{nginx-container}: Container started
```

- 지금은 image를 1.31버전을 사용하고 있지만 만약 1.31버전을 사용하다가 최신 버전인 1.31.3으로 버전을 변경하게되면 pod에는 변화가 없다
- 왜냐하면 ReplicationController은 selector의 key: value만으로 pod를 관리하기 때문에 버전은 확인하지 않는다.

```bash
	# 터미널 2
[root@k8s-master ~]# kubectl  delete  pods  rc-nginx-ggjvk
pod "rc-nginx-ggjvk" deleted from default namespace

	# 터미널 1
[root@k8s-master ~]# kubectl  get  pods  -o  wide  --watch
NAME             READY   STATUS        RESTARTS   AGE   IP            NODE          NOMINATED NODE   READINESS GATES
rc-nginx-ggjvk   1/1     Running       0          41m   10.244.2.3    k8s-worker2   <none>            <none>
rc-nginx-nlp42   1/1     Running       0          41m   10.244.1.6    k8s-worker1   <none>            <none>
rc-nginx-ggjvk   1/1     Terminating   0          47m   10.244.2.3    k8s-worker2   <none>            <none>
rc-nginx-sd45g   0/1     Pending       0          0s    <none>        <none>        <none>            <none>
rc-nginx-ggjvk   1/1     Terminating   0          47m   10.244.2.3    k8s-worker2   <none>            <none>
rc-nginx-sd45g   0/1     Pending       0          0s    <none>        k8s-worker2   <none>            <none>
rc-nginx-sd45g   0/1     ContainerCreating   0     0s    <none>        k8s-worker2   <none>            <none>
rc-nginx-ggjvk   0/1     Completed     0          47m   10.244.2.3    k8s-worker2   <none>            <none>
rc-nginx-sd45g   1/1     Running       0          3s    10.244.2.5    k8s-worker2   <none>            <none>
```

- ReplicationController에의해 새로 만들어진 pod = rc-nginx-sd45g

```bash
	# 이미지가 1.31에서 1.31.3 버전으로 변경 확인
Image size: 63135215 bytes.
[root@k8s-master ~]# kubectl describe pod rc-nginx-sd45g  | grep -i  image
    Image:          nginx:1.31.3
    Image ID:       docker.io/library/nginx@sha256:8541484afbc9c8a5a8a99b379568ebbc957f658583ec9448fc43104229c03cf8
  Normal  Pulling    3m43s  kubelet            spec.containers{nginx-container}: Pulling image "nginx:1.31.3"
  Normal  Pulled     3m42s  kubelet            spec.containers{nginx-container}: Successfully pulled image "nginx:1.31.3" in 1.618s (1.618s including waiting). Image size: 63135215 bytes.

	# 이미지 그대로 사용
[root@k8s-master ~]# kubectl describe pod rc-nginx-nlp42  | grep -i  image
    Image:          nginx:1.31
    Image ID:       docker.io/library/nginx@sha256:8541484afbc9c8a5a8a99b379568ebbc957f658583ec9448fc43104229c03cf8
  Normal  Pulled     51m   kubelet            spec.containers{nginx-container}: Container image "nginx:1.31" already present on machine and can be accessed by the pod
```

- ReplicationController의 모든 pod를 삭제할때에는 pod가 아니라 Controller를 삭제해야 한다.

```bash
[root@k8s-master ~]# kubectl  delete  rc  rc-nginx
replicationcontroller "rc-nginx" deleted from default namespace

[root@k8s-master ~]# kubectl  get  pods
No resources found in default namespace.
```

### ReplicationController의 한계

- ReplicationController는 개념 이해에는 좋지만 실무에서는 거의 사용하지 않는다.

이유
- Rolling Update 기능이 부족함
- 더 발전된 ReplicaSet / Deployment를 사용

정리: ReplicationController는 selector 기반 Pod 개수 유지라는 핵심 개념을 보여주지만 Rolling Update 등 실무 기능이 부족해 표준 자리를 **ReplicaSet**에 넘겨준다.

---

## ReplicaSet

- **ReplicaSet**은 지정한 개수만큼 Pod가 항상 존재하도록 유지하는 컨트롤러다.
- Pod가 죽거나 삭제되면 자동으로 다시 만들고, Pod가 너무 많으면 줄여서 원하는 개수(**replicas**)를 맞춘다.
- 쿠버네티스에서 Pod 개수 보장의 표준 컨트롤러지만, 실무에서는 ReplicaSet을 직접 쓰기보다 **Deployment**가 ReplicaSet을 만들어서 관리하는 형태가 가장 흔하다.

### ReplicaSet이 필요한 이유

- Pod는 쉽게 죽을 수 있다: 컨테이너 오류, 노드 장애, 사람이 kubectl delete pod 실행 등으로 Pod는 언제든 사라질 수 있다.
- 서비스는 계속 살아 있어야 한다: 웹 서버를 3개 띄워 운영 중이라면, 1개가 죽어도 다시 3개로 복구되어야 한다.
- ReplicaSet이 이 역할을 한다: 항상 Pod 3개 같은 목표를 정해두면, 실제 상태가 바뀌어도 자동으로 원래 상태로 되돌린다.

### ReplicaSet의 핵심 개념 3가지

- **replicas** : 유지해야 하는 Pod 개수(목표치). 예: replicas: 3 이면 항상 3개를 유지하려고 한다
- **selector** : ReplicaSet이 내가 관리할 Pod를 고르는 조건(라벨 조건). selector에 걸리는 Pod를 세어서 replicas에 맞춘다. 매우 중요: selector를 너무 넓게 잡으면 다른 Pod까지 관리 대상으로 잡아버릴 수 있다
- **template** : Pod가 부족할 때 새로 만들 Pod의 설계도. 어떤 이미지, 포트, 환경변수로 Pod를 만들지 정의한다. **template.metadata.labels는 selector와 일치해야 한다.**
  - 예: replicas: 3, selector: app=webui 라고 했을 때 ReplicaSet은 현재 클러스터에서 label이 app=webui인 Pod만 센다.
  - 개수가 3보다 적으면 template으로 Pod를 추가 생성한다.
  - 개수가 3보다 많으면 초과된 Pod를 삭제한다.
  - 이 과정을 계속 반복해서 항상 3개 상태를 유지한다.

### ReplicaSet과 ReplicationController와 차이

- 기능 목적은 거의 동일하다. (Pod 개수 유지)
- 가장 큰 차이 : selector 표현 범위
  - ReplicationController는 보통 equality 기반(=) 중심
  - ReplicaSet은 set-based selector(예: in, notin, exists 등)까지 지원해서 더 유연하다
  - 현재 표준은 ReplicaSet이고, 운영에서는 보통 Deployment를 사용한다

ReplicaSet 예제 YAML

```yaml
apiVersion: apps/v1
kind: ReplicaSet

metadata:
  name: rs-webui

spec:
  replicas: 3

  selector:
    matchLabels:
      app: webui
    matchExpressions:
    - key: version  	# 검사할 라벨의 key 이름 (version 라벨을 기준으로 판단)
      operator: In   	# 조건 연산자: 값이 아래 values 목록에 포함되면 참
      values:
      - "2.1"      	# version 라벨 값이 "2.1" 인 Pod만 선택

  template:
    metadata:
      labels:
        app: webui

    spec:
      containers:
      - name: nginx-container
        image: nginx:1.28
        ports:
        - containerPort: 80
```

### operator 종류와 의미

**1) In**
```yaml
- key: version
  operator: In
  values:
  - "2.1"
  - "2.2"
```
- version 라벨이 있고 그 값이 2.1 또는 2.2 중 하나면 참
- 선택되는 Pod: version=2.1, version=2.2
- 선택 안 됨: version=1.0, version 없음

**2) NotIn**
```yaml
- key: env
  operator: NotIn
  values:
  - dev
```
- env 라벨이 있고 값이 dev가 아니면 참
- 선택되는 Pod: env=prod, env=stage
- 선택 안 됨: env=dev, env 없음

**3) Exists**
```yaml
- key: tier
  operator: Exists
```
- tier 라벨이 존재하기만 하면 참 (값은 상관없음)
- 선택되는 Pod: tier=frontend, tier=backend, tier=anything
- 선택 안 됨: tier 라벨 없음

**4) DoesNotExist**
```yaml
- key: debug
  operator: DoesNotExist
```
- debug 라벨이 없으면 참
- 선택되는 Pod: (debug 라벨 없음)
- 선택 안 됨: debug=true, debug=false

### ReplicaSet 실습 — 생성/확인

```bash
[root@k8s-master ~]# vi  rs-nginx.yaml
```
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: rs-nginx

spec:
  replicas: 3
  selector:
    matchLabels:
      app: webui

  template:
    metadata:
      name: nginx-pod
      labels:
        app: webui

    spec:
      containers:
      - name: nginx-container
        image: nginx:1.31
```

```bash
	# 터미널 2
[root@k8s-master ~]# kubectl  apply  -f  rs-nginx.yaml  --dry-run=client
replicaset.apps/rs-nginx created (dry run)

	# 터미널 1
[root@k8s-master ~]# watch  kubectl  get  pods  -o   wide

	# 터미널 2
[root@k8s-master ~]# kubectl  apply  -f  rs-nginx.yaml
replicaset.apps/rs-nginx created

	# 터미널 1
[root@k8s-master ~]# watch  kubectl  get  pods  -o   wide
Every 2.0s: kubectl get pods -o wide                                                                      k8s-master: Tue Aug 18 13:08:58 2026
NAME             READY   STATUS    RESTARTS   AGE   IP            NODE          NOMINATED NODE   READINESS GATES
rs-nginx-845g8   1/1     Running   0          16s   10.244.2.6    k8s-worker2   <none>            <none>
rs-nginx-9bkmw   1/1     Running   0          16s   10.244.1.9    k8s-worker1   <none>            <none>
rs-nginx-vjpv7   1/1     Running   0          16s   10.244.1.8    k8s-worker1   <none>            <none>

[root@k8s-master ~]# kubectl  get  replicasets.apps  rs-nginx
NAME       DESIRED   CURRENT   READY   AGE
rs-nginx   3         3         3       71s

[root@k8s-master ~]# kubectl  get  rs  rs-nginx
NAME       DESIRED   CURRENT   READY   AGE
rs-nginx   3         3         3       71s

[root@k8s-master ~]# kubectl  get  pods
NAME             READY   STATUS    RESTARTS   AGE
rs-nginx-845g8   1/1     Running   0          2m35s
rs-nginx-9bkmw   1/1     Running   0          2m35s
rs-nginx-vjpv7   1/1     Running   0          2m35s

[root@k8s-master ~]# kubectl  get  pods  --show-labels
NAME             READY   STATUS    RESTARTS   AGE     LABELS
rs-nginx-845g8   1/1     Running   0          3m22s   app=webui
rs-nginx-9bkmw   1/1     Running   0          3m22s   app=webui
rs-nginx-vjpv7   1/1     Running   0          3m22s   app=webui

	# ReplicaSet Controller에 의해 생성된 pod 삭제
[root@k8s-master ~]# kubectl  delete  pods  rs-nginx-vjpv7
pod "rs-nginx-vjpv7" deleted from default namespace

[root@k8s-master ~]# kubectl  get  pods
NAME             READY   STATUS    RESTARTS   AGE
rs-nginx-845g8   1/1     Running   0          2m35s
rs-nginx-9bkmw   1/1     Running   0          2m35s
rs-nginx-h9tf5   1/1     Running   0          42s		# 새로운 pod 생성

[root@k8s-master ~]# kubectl  get  rs  rs-nginx
NAME       DESIRED   CURRENT   READY   AGE
rs-nginx   3         3         3       6m15s
```

### Scale-out (pod 확장)

```bash
[root@k8s-master ~]# kubectl  edit  rs   rs-nginx
~~~~~~~~~~~~ 중간 생략 ~~~~~~~~~~~~
spec:
  replicas: 3		# 3을 5로 변경
  selector:
    matchLabels:
      app: webui
  template:
    metadata:
      labels:
        app: webui
      name: nginx-pod
    spec:
      containers:
      - image: nginx:1.31
        imagePullPolicy: IfNotPresent
        name: nginx-container
~~~~~~~~~~~~ 중간 생략 ~~~~~~~~~~~~

	OR

	# 터미널 2
[root@k8s-master ~]# kubectl  scale  rs  rs-nginx --replicas=5
replicaset.apps/rs-nginx scaled

	# 터미널 1
[root@k8s-master ~]# watch  kubectl  get  pods  -o   wide
Every 2.0s: kubectl get pods -o wide                                                                      k8s-master: Tue Aug 18 14:38:01 2026
NAME             READY   STATUS    RESTARTS   AGE   IP            NODE          NOMINATED NODE   READINESS GATES
rs-nginx-845g8   1/1     Running   0          89m   10.244.2.6    k8s-worker2   <none>            <none>
rs-nginx-9bkmw   1/1     Running   0          89m   10.244.1.9    k8s-worker1   <none>            <none>
rs-nginx-h9tf5   1/1     Running   0          84m   10.244.2.7    k8s-worker2   <none>            <none>
rs-nginx-rrqnk   1/1     Running   0          12s   10.244.1.10   k8s-worker1   <none>            <none>
rs-nginx-zsr9r   1/1     Running   0          12s   10.244.1.11   k8s-worker1   <none>            <none>

	# 터미널 2
[root@k8s-master ~]# kubectl  get  pods
NAME             READY   STATUS    RESTARTS   AGE
rs-nginx-845g8   1/1     Running   0          91m
rs-nginx-9bkmw   1/1     Running   0          91m
rs-nginx-h9tf5   1/1     Running   0          87m
rs-nginx-rrqnk   1/1     Running   0          2m48s
rs-nginx-zsr9r   1/1     Running   0          2m48s
```

### Scale-in (pod 축소)

```bash
[root@k8s-master ~]# kubectl  edit  rs   rs-nginx
~~~~~~~~~~~~ 중간 생략 ~~~~~~~~~~~~
spec:
  replicas: 3		# 5를 2로 변경
  selector:
    matchLabels:
      app: webui
  template:
    metadata:
      labels:
        app: webui
      name: nginx-pod
    spec:
      containers:
      - image: nginx:1.31
        imagePullPolicy: IfNotPresent
        name: nginx-container
~~~~~~~~~~~~ 중간 생략 ~~~~~~~~~~~~

	OR

	# 터미널 2
[root@k8s-master ~]# kubectl  scale  rs  rs-nginx --replicas=2
replicaset.apps/rs-nginx scaled

	# 터미널 1
[root@k8s-master ~]# watch  kubectl  get  pods  -o   wide
Every 2.0s: kubectl get pods -o wide                                                                      k8s-master: Tue Aug 18 14:38:01 2026
NAME             READY   STATUS    RESTARTS   AGE   IP            NODE          NOMINATED NODE   READINESS GATES
rs-nginx-9bkmw   1/1     Running   0          89m   10.244.1.9    k8s-worker1   <none>            <none>
rs-nginx-h9tf5   1/1     Running   0          84m   10.244.2.7    k8s-worker2   <none>            <none>

	# 터미널 2
[root@k8s-master ~]# kubectl  get  pods
NAME             READY   STATUS    RESTARTS   AGE
rs-nginx-9bkmw   1/1     Running   0          91m
rs-nginx-h9tf5   1/1     Running   0          87m
```

### ReplicaSet Controller 삭제

```bash
[root@k8s-master ~]# kubectl  delete  rs  rs-nginx
             OR
[root@k8s-master ~]# kubectl  delete  -f  rs-nginx.yaml

# Controller를 삭제하면 해당 Controller가 관리하는 pod도 같이 삭제된다.
[root@k8s-master ~]# kubectl  get  pods
No resources found in default namespace.
```

### rs controller 재생성 및 cascade 삭제

```bash
[root@k8s-master ~]# kubectl  apply  -f  rs-nginx.yaml
replicaset.apps/rs-nginx created

[root@k8s-master ~]# kubectl  get  pods
NAME             READY   STATUS    RESTARTS   AGE
rs-nginx-dfks9   1/1     Running   0          7s
rs-nginx-hcvv2   1/1     Running   0          7s
rs-nginx-nz7pl   1/1     Running   0          7s
```

	# 해당 Pod는 삭제하지 않고 Controller만 삭제

```bash
[root@k8s-master ~]# kubectl  delete  rs  rs-nginx  --cascade=false		# 현재 비 권장 방식
warning: --cascade=false is deprecated (boolean value) and can be replaced with --cascade=orphan.
replicaset.apps "rs-nginx" deleted from default namespace

	~~~~~~~~~~ OR ~~~~~~~~~~

[root@k8s-master ~]# kubectl  delete  rs  rs-nginx  --cascade=orphan		# 현재 권장 방식
replicaset.apps "rs-nginx" deleted from default namespace

[root@k8s-master ~]# kubectl  get  rs			# 컨트롤러가 확인되지 않는다.
No resources found in default namespa

[root@k8s-master ~]# kubectl  get  rs  rs-nginx		# 컨트롤러가 확인되지 않는다.
Error from server (NotFound): replicasets.apps "rs-nginx" not found

[root@k8s-master ~]# kubectl  get  pods		# Pod는 확인된다.
NAME             READY   STATUS    RESTARTS   AGE
rs-nginx-dfks9   1/1     Running   0          5m10s
rs-nginx-hcvv2   1/1     Running   0          5m10s
rs-nginx-nz7pl   1/1     Running   0          5m10s

[root@k8s-master ~]# kubectl  delete  pods --all
pod "rs-nginx-dfks9" deleted from default namespace
pod "rs-nginx-hcvv2" deleted from default namespace
pod "rs-nginx-nz7pl" deleted from default namespace
```

### EX1) ReplicaSet selector 기본 원리 + 포함/제외 + scale 실습

- selector는 "라벨 조건" 이다.
- ReplicaSet은 selector에 걸리는 Pod 개수만 유지한다.
- 라벨 변경이 관리 대상 포함/제외를 즉시 변경한다.

```bash
[root@k8s-master ~]# vi rs-lab-a.yaml
```
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: rs-lab-a

spec:
  replicas: 3
  selector:
    matchLabels:
      app: web
      env: prod

  template:
    metadata:
      labels:
        app: web
        env: prod
        tier: frontend
    spec:
      containers:
      - name: nginx
        image: nginx:1.31
```

```bash
[root@k8s-master ~]# watch  kubectl  get  pods  -o   wide

[root@k8s-master ~]# kubectl  apply  -f rs-lab-a.yaml  --dry-run=client
replicaset.apps/rs-lab-a created (dry run)

[root@k8s-master ~]# kubectl  apply  -f rs-lab-a.yaml
replicaset.apps/rs-lab-a created

[root@k8s-master ~]# watch  kubectl  get  pods  -o   wide
NAME             READY   STATUS    RESTARTS   AGE   IP            NODE          NOMINATED NODE   READINESS GATES
rs-lab-a-hr6kn   1/1     Running   0          8s    10.244.1.15   k8s-worker1   <none>            <none>
rs-lab-a-jvz5j   1/1     Running   0          8s    10.244.1.14   k8s-worker1   <none>            <none>
rs-lab-a-m8v2j   1/1     Running   0          8s    10.244.2.9    k8s-worker2   <none>            <none>

[root@k8s-master ~]# kubectl  get  rs  rs-lab-a
NAME       DESIRED   CURRENT   READY   AGE
rs-lab-a   3         3         3       57s

[root@k8s-master ~]# kubectl  get  pods
NAME             READY   STATUS    RESTARTS   AGE
rs-lab-a-hr6kn   1/1     Running   0          104s
rs-lab-a-jvz5j   1/1     Running   0          104s
rs-lab-a-m8v2j   1/1     Running   0          104s

[root@k8s-master ~]# kubectl  get  pods  --show-labels
NAME             READY   STATUS    RESTARTS   AGE     LABELS
rs-lab-a-hr6kn   1/1     Running   0          2m19s   app=web,env=prod,tier=frontend
rs-lab-a-jvz5j   1/1     Running   0          2m19s   app=web,env=prod,tier=frontend
rs-lab-a-m8v2j   1/1     Running   0          2m19s   app=web,env=prod,tier=frontend
```

**1. Pod 하나를 selector에서 제외시키기**

EX1-1) rs-lab-a가 만든 Pod 중 하나의 env를 prod에서 dev 로 변경

```bash
[root@k8s-master ~]# kubectl  get  pods  --show-labels
NAME             READY   STATUS    RESTARTS   AGE     LABELS
rs-lab-a-hr6kn   1/1     Running   0          2m19s   app=web,env=prod,tier=frontend
rs-lab-a-jvz5j   1/1     Running   0          2m19s   app=web,env=prod,tier=frontend
rs-lab-a-m8v2j   1/1     Running   0          2m19s   app=web,env=prod,tier=frontend

	# rs-lab-a-hr6kn pod의 label을 변경
[root@k8s-master ~]# kubectl  label  pod  rs-lab-a-hr6kn env=dev  --overwrite

[root@k8s-master ~]# watch  kubectl  get  pods  -o   wide
NAME             READY   STATUS    RESTARTS   AGE     IP            NODE          NOMINATED NODE   READINESS GATES
rs-lab-a-hr6kn   1/1     Running   0          5m31s   10.244.1.15   k8s-worker1   <none>            <none>
rs-lab-a-jvz5j   1/1     Running   0          5m31s   10.244.1.14   k8s-worker1   <none>            <none>
rs-lab-a-m8v2j   1/1     Running   0          5m31s   10.244.2.9    k8s-worker2   <none>            <none>
rs-lab-a-wlj8h   1/1     Running   0          3s      10.244.2.10   k8s-worker2   <none>            <none>		# 새로운 pod 생성

[root@k8s-master ~]# kubectl  get  pods  --show-labels
NAME             READY   STATUS    RESTARTS   AGE     LABELS
rs-lab-a-hr6kn   1/1     Running   0          6m34s   app=web,env=dev,tier=frontend
rs-lab-a-jvz5j   1/1     Running   0          6m34s   app=web,env=prod,tier=frontend
rs-lab-a-m8v2j   1/1     Running   0          6m34s   app=web,env=prod,tier=frontend
rs-lab-a-wlj8h   1/1     Running   0          66s     app=web,env=prod,tier=frontend

[root@k8s-master ~]# kubectl  get  rs rs-lab-a
NAME       DESIRED   CURRENT   READY   AGE
rs-lab-a   3         3         3       8m16s

[root@k8s-master ~]# kubectl  scale  rs  rs-lab-a  --replicas=5
replicaset.apps/rs-lab-a scaled

[root@k8s-master ~]# watch  kubectl  get  pods  -o   wide
NAME             READY   STATUS    RESTARTS   AGE    IP            NODE          NOMINATED NODE   READINESS GATES
rs-lab-a-7xwmd   1/1     Running   0          5s     10.244.1.16   k8s-worker1   <none>           <none>
rs-lab-a-hr6kn   1/1     Running   0          10m    10.244.1.15   k8s-worker1   <none>           <none>
rs-lab-a-jvz5j   1/1     Running   0          10m    10.244.1.14   k8s-worker1   <none>           <none>
rs-lab-a-m8v2j   1/1     Running   0          10m    10.244.2.9    k8s-worker2   <none>           <none>
rs-lab-a-n8ztl   1/1     Running   0          5s     10.244.2.11   k8s-worker2   <none>           <none>
rs-lab-a-wlj8h   1/1     Running   0          5m1s   10.244.2.10   k8s-worker2   <none>           <none>

[root@k8s-master ~]# kubectl  get  pods  --show-labels
NAME             READY   STATUS    RESTARTS   AGE     LABELS
rs-lab-a-7xwmd   1/1     Running   0          43s     app=web,env=prod,tier=frontend
rs-lab-a-hr6kn   1/1     Running   0          11m     app=web,env=dev,tier=frontend		# label match
rs-lab-a-jvz5j   1/1     Running   0          11m     app=web,env=prod,tier=frontend		# label match
rs-lab-a-m8v2j   1/1     Running   0          11m     app=web,env=prod,tier=frontend		# label match
rs-lab-a-n8ztl   1/1     Running   0          43s     app=web,env=prod,tier=frontend		# label match
rs-lab-a-wlj8h   1/1     Running   0          5m39s   app=web,env=prod,tier=frontend		# label match
```

**3. 수동 Pod를 selector에 끼워 넣기**

EX1-3) 수동 Pod를 하나 만들되, ReplicaSet Controller rs-lab-a selector에 매치되어야

```bash
[root@k8s-master ~]# kubectl  run  maunal-pod  --image=nginx:1.31  --labels app=web,env=prod,tier=backend
pod/maunal-pod created

	# tier=backend 라벨이 확인되지 않는다. (ReplicaSet Controller에 의해 삭제)
[root@k8s-master ~]# kubectl  get  pods  --show-labels
NAME             READY   STATUS    RESTARTS   AGE     LABELS
rs-lab-a-7xwmd   1/1     Running   0          43s     app=web,env=prod,tier=frontend
rs-lab-a-hr6kn   1/1     Running   0          11m     app=web,env=dev,tier=frontend
rs-lab-a-jvz5j   1/1     Running   0          11m     app=web,env=prod,tier=frontend
rs-lab-a-m8v2j   1/1     Running   0          11m     app=web,env=prod,tier=frontend
rs-lab-a-n8ztl   1/1     Running   0          43s     app=web,env=prod,tier=frontend
rs-lab-a-wlj8h   1/1     Running   0          5m39s   app=web,env=prod,tier=frontend
```

- ReplicaSet Controller에의해 만들어진 pod가 아니어도 label만 매치되면 관리대상으로 포함된다.

**4. ownerReferences로 RS Pod/수동 Pod 구분**

- ownerReferences
  - 이 리소스는 누가 만든 자식 객체인지 쿠버네티스가 내부적으로 관리하기 위한 연결 정보
  - 부모–자식 관계를 명확하게 기록해 두는 메타데이터
  - 단 단독으로 실행되는 pod에는 owneReferences가 없다.

```bash
[root@k8s-master ~]# kubectl  get  pod  rs-lab-a-7xwmd  -o  yaml | nl
     1  apiVersion: v1
     2  kind: Pod
     3  metadata:
     4    creationTimestamp: "2026-08-18T06:10:36Z"
     5    generateName: rs-lab-a-
     6    generation: 1
     7    labels:
     8      app: web
     9      env: prod
    10      tier: frontend
    11    name: rs-lab-a-7xwmd
    12    namespace: default
    13    ownerReferences:
    14    - apiVersion: apps/v1
    15      blockOwnerDeletion: true
    16      controller: true
    17      kind: ReplicaSet
    18      name: rs-lab-a
    19      uid: f127b14a-eb5a-40bc-80af-bf900ff43f2e
    20    resourceVersion: "183395"
    21    uid: 054c33cb-c1f6-4f2d-b83c-d018befb114f
    22  spec:
    23    containers:
    24    - image: nginx:1.31
    25      imagePullPolicy: IfNotPresent
    26      name: nginx
~~~~~~~~~ 중간 생략 ~~~~~~~~~

---

## ReplicaSet (selector or)

- ReplicaSet matchExpressions(In) 하나로 OR 조건을 구성해 둔 뒤 수동 Pod 투입, 라벨 변경 편입/제외, scale 반응, 실수 상황을 YAML 1개로 실습
  - 쿠버네티스 라벨 셀렉터에 OR 문법이 없고 matchExpressions(In)을 OR 문법으로 확인
  - 라벨 변경으로 관리 대상이 실시간으로 편입/제외 확인
  - scale out/in이 selector에 걸리는 Pod만 기준으로 동작 확인

```bash
[root@k8s-master ~]# vi rs-lab-a-or.yaml
```
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: rs-lab-a-or

spec:
  replicas: 2
  selector:
    matchExpressions:
    - key: env
      operator: In
      values:
      - prod
      - stage

  template:
    metadata:
      labels:
        app: web
        env: prod
        lab: rs-lab-a-or
    spec:
      containers:
      - name: nginx
        image: nginx:1.31

# env=prod OR env=stage
```

```bash
[root@k8s-master ~]# kubectl  apply  -f  rs-lab-a-or.yaml
replicaset.apps/rs-lab-a-or created

[root@k8s-master ~]# kubectl  get  rs  rs-lab-a-or
NAME          DESIRED   CURRENT   READY   AGE
rs-lab-a-or   2         2         2       13s

[root@k8s-master ~]# kubectl  get  pods
NAME                READY   STATUS    RESTARTS   AGE
rs-lab-a-or-56l95   1/1     Running   0          31s
rs-lab-a-or-bsr78   1/1     Running   0          31s

[root@k8s-master ~]# kubectl  get  pods  --show-labels
NAME                READY   STATUS    RESTARTS   AGE   LABELS
rs-lab-a-or-56l95   1/1     Running   0          52s   app=web,env=prod,lab=rs-lab-a-or
rs-lab-a-or-bsr78   1/1     Running   0          52s   app=web,env=prod,lab=rs-lab-a-or

[root@k8s-master ~]# kubectl  describe  rs  rs-lab-a-or
Name:         rs-lab-a-or
Namespace:    default
Selector:     env in (prod,stage)
Labels:       <none>
Annotations:  <none>
Replicas:     2 current / 2 desired
Pods Status:  2 Running / 0 Waiting / 0 Succeeded / 0 Failed
Pod Template:
  Labels:  app=web
           env=prod
           lab=rs-lab-a-or
  Containers:
   nginx:
    Image:          nginx:1.31
    Port:           <none>
    Host Port:      <none>
    Environment:    <none>
    Mounts:         <none>
  Volumes:          <none>
  Node-Selectors:   <none>
  Tolerations:      <none>
Events:
  Type    Reason            Age    From                   Message
  ----    ------            ----   ----                   -------
  Normal  SuccessfulCreate  3m20s  replicaset-controller  Created pod: rs-lab-a-or-56l95
  Normal  SuccessfulCreate  3m20s  replicaset-controller  Created pod: rs-lab-a-or-bsr78

[root@k8s-master ~]# kubectl  get  rs  rs-lab-a-or  -o  wide
NAME          DESIRED   CURRENT   READY   AGE     CONTAINERS   IMAGES       SELECTOR
rs-lab-a-or   2         2         2       5m32s   nginx        nginx:1.31   env in (prod,stage)
```

### STEP 1. OR 조건 확인

```bash
[root@k8s-master ~]# kubectl  get  rs  rs-lab-a-or  -o  wide
NAME          DESIRED   CURRENT   READY   AGE     CONTAINERS   IMAGES       SELECTOR
rs-lab-a-or   2         2         2       5m32s   nginx        nginx:1.31   env in (prod,stage)

[root@k8s-master ~]# kubectl  get  pods --show-labels
NAME                READY   STATUS    RESTARTS   AGE   LABELS
rs-lab-a-or-56l95   1/1     Running   0          52s   app=web,env=prod,lab=rs-lab-a-or
rs-lab-a-or-bsr78   1/1     Running   0          52s   app=web,env=prod,lab=rs-lab-a-or
```

- env가 prod이거나 stage인 Pod를 관리 대상으로 삼는다

### STEP 2. 수동 Pod 5개를 서로 다른 env로 pod 생성 (분류 실습)

```bash
[root@k8s-master ~]# kubectl  get  rs  rs-lab-a-or  -o  wide
NAME          DESIRED   CURRENT   READY   AGE     CONTAINERS   IMAGES       SELECTOR
rs-lab-a-or   2         2         2       5m32s   nginx        nginx:1.31   env in (prod,stage)

[root@k8s-master ~]# kubectl  get  pods  -o  wide  --watch

설정 (전부 YAML 추가 없이 run으로)
[root@k8s-master ~]# kubectl  run  p-prod  --image=nginx:1.31  --labels env=prod,lab=rs-lab-a-or
```

- env=stage는 ReplicaSet의 Selector 조건인 env=prod 또는 env=stage에 해당한다. 따라서 p-stage Pod는 ReplicaSet Controller의 관리 대상이 된다.
- ReplicaSet이 이미 원하는 Pod 개수(replicas)를 유지하고 있다면, 새로 생성된 p-stage까지 포함해 Pod 수가 초과되므로 ReplicaSet이 Pod 하나를 자동 삭제하여 설정된 개수를 유지한다.

```bash
[root@k8s-master ~]# kubectl  run  p-stage  --image=nginx:1.31  --labels env=stage,lab=rs-lab-a-or
```

- env=stage는 ReplicaSet의 Selector 조건인 env=prod 또는 env=stage에 해당한다. 따라서 p-stage Pod는 ReplicaSet Controller의 관리 대상이 된다.
- ReplicaSet이 이미 원하는 Pod 개수(replicas)를 유지하고 있다면, 새로 생성된 p-stage까지 포함해 Pod 수가 초과되므로 ReplicaSet이 Pod 하나를 자동 삭제하여 설정된 개수를 유지한다.

```bash
[root@k8s-master ~]# kubectl  run  p-dev  --image=nginx:1.31  --labels env=dev,lab=rs-lab-a-or
```

- env=dev는 ReplicaSet의 Selector 조건인 env=prod 또는 env=stage에 해당하지 않는다. 따라서 p-dev Pod는 ReplicaSet Controller의 관리 대상이 아니다.
- ReplicaSet의 replicas 개수 계산에도 포함되지 않으며 ReplicaSet에 의해 자동 삭제되지 않고 독립적인 Pod로 계속 실행된다.

```bash
[root@k8s-master ~]# kubectl  run  p-test  --image=nginx:1.31  --labels env=test,lab=rs-lab-a-or
```

- env=test는 ReplicaSet의 Selector 조건인 env=prod 또는 env=stage에 해당하지 않는다. 따라서 p-dev Pod는 ReplicaSet Controller의 관리 대상이 아니다.
- ReplicaSet의 replicas 개수 계산에도 포함되지 않으며 ReplicaSet에 의해 자동 삭제되지 않고 독립적인 Pod로 계속 실행된다

```bash
[root@k8s-master ~]# kubectl get  pods
NAME                READY   STATUS    RESTARTS   AGE
p-dev               1/1     Running   0          16s
p-test              1/1     Running   0          5s
rs-lab-a-or-56l95   1/1     Running   0          16m
rs-lab-a-or-bsr78   1/1     Running   0          16m

[root@k8s-master ~]# kubectl  run  p-none  --image=nginx:1.31  --labels  lab=rs-lab-a-or
```

- p-none Pod에는 lab=rs-lab-a-or 라벨만 설정된다.
- ReplicaSet의 Selector 조건인 env=prod 또는 env=stage에 필요한 env 라벨이 없다. 따라서 p-none Pod는 ReplicaSet Controller의 관리 대상이 아니다.
- ReplicaSet의 replicas 개수에도 포함되지 않기 때문에 자동 삭제되지 않고 독립적인 Pod로 실행된다.

```bash
[root@k8s-master ~]# kubectl get  pods
NAME                READY   STATUS    RESTARTS   AGE
p-dev               1/1     Running   0          3m24s
p-none              1/1     Running   0          29s
p-test              1/1     Running   0          3m13s
rs-lab-a-or-56l95   1/1     Running   0          19m
rs-lab-a-or-bsr78   1/1     Running   0          19m

[root@k8s-master ~]# kubectl get  pods  --show-labels
NAME                READY   STATUS    RESTARTS   AGE     LABELS
p-dev               1/1     Running   0          4m44s   env=dev,lab=rs-lab-a-or
p-none              1/1     Running   0          109s    lab=rs-lab-a-or
p-test              1/1     Running   0          4m33s   env=test,lab=rs-lab-a-or
rs-lab-a-or-56l95   1/1     Running   0          21m     app=web,env=prod,lab=rs-lab-a-or
rs-lab-a-or-bsr78   1/1     Running   0          21m     app=web,env=prod,lab=rs-lab-a-or
```

### STEP 3. 제외된 Pod를 라벨 변경으로 편입시키기

EX) p-dev를 관리 대상에 편입시켜라(env를 stage 또는 prod로 변경)

```bash
[root@k8s-master ~]# kubectl  label  pod  p-dev  env=stage  --overwrite
pod/p-dev labeled

[root@k8s-master ~]# kubectl  get  pods  -o  wide  --watch
NAME                READY   STATUS        RESTARTS   AGE     IP            NODE          NOMINATED NODE   READINESS GATES
p-dev               1/1     Running       0          6m51s   10.244.1.18   k8s-worker1   <none>            <none>
p-none              1/1     Running       0          3m56s   10.244.1.19   k8s-worker1   <none>            <none>
p-test              1/1     Running       0          6m40s   10.244.2.13   k8s-worker2   <none>            <none>
rs-lab-a-or-56l95   1/1     Running       0          23m     10.244.2.12   k8s-worker2   <none>            <none>
rs-lab-a-or-bsr78   1/1     Running       0          23m     10.244.1.17   k8s-worker1   <none>            <none>
p-dev               1/1     Running       0          7m27s   10.244.1.18   k8s-worker1   <none>            <none>
p-dev               1/1     Running       0          7m27s   10.244.1.18   k8s-worker1   <none>            <none>
p-dev               1/1     Terminating   0          7m27s   10.244.1.18   k8s-worker1   <none>            <none>
p-dev               1/1     Terminating   0          7m28s   10.244.1.18   k8s-worker1   <none>            <none>
p-dev               0/1     Completed     0          7m28s   10.244.1.18   k8s-worker1   <none>            <none>
p-dev               0/1     Completed     0          7m29s   10.244.1.18   k8s-worker1   <none>            <none>
p-dev               0/1     Completed     0          7m29s   10.244.1.18   k8s-worker1   <none>            <none>

	# p-dev가 확인되지 않는다.
[root@k8s-master ~]# kubectl get pods --show-labels
NAME                READY   STATUS    RESTARTS   AGE     LABELS
p-none              1/1     Running   0          6m10s   lab=rs-lab-a-or
p-test              1/1     Running   0          8m54s   env=test,lab=rs-lab-a-or
rs-lab-a-or-56l95   1/1     Running   0          25m     app=web,env=prod,lab=rs-lab-a-or
rs-lab-a-or-bsr78   1/1     Running   0          25m     app=web,env=prod,lab=rs-lab-a-or

[root@k8s-master ~]# kubectl get rs
NAME          DESIRED   CURRENT   READY   AGE
rs-lab-a-or   2         2         2       26m
```

- 라벨을 변경하게되면 ReplicaSet Controller의 selector에 매치되어 관리 대상이 된다.

### STEP 4. 포함된 Pod를 라벨 변경으로 제외시키기(제외 실습)

EX) p-prod를 관리 대상에서 제외시켜라(정답: env를 dev 등으로 바꿈)

```bash
[root@k8s-master ~]# kubectl get pods --show-labels
NAME                READY   STATUS    RESTARTS   AGE   LABELS
p-none              1/1     Running   0          10m   lab=rs-lab-a-or
p-test              1/1     Running   0          13m   env=test,lab=rs-lab-a-or
rs-lab-a-or-56l95   1/1     Running   0          30m   app=web,env=prod,lab=rs-lab-a-or
rs-lab-a-or-bsr78   1/1     Running   0          30m   app=web,env=prod,lab=rs-lab-a-or

[root@k8s-master ~]# kubectl  label  pod  rs-lab-a-or-56l95  env=test  --overwrite
pod/rs-lab-a-or-56l95 labeled

[root@k8s-master ~]# kubectl  get  pods  -o  wide  --watch
NAME                READY   STATUS              RESTARTS   AGE   IP            NODE          NOMINATED NODE   READINESS GATES
p-none              1/1     Running             0          11m   10.244.1.19   k8s-worker1   <none>            <none>
p-test              1/1     Running             0          14m   10.244.2.13   k8s-worker2   <none>            <none>
rs-lab-a-or-56l95   1/1     Running             0          30m   10.244.2.12   k8s-worker2   <none>            <none>
rs-lab-a-or-bsr78   1/1     Running             0          30m   10.244.1.17   k8s-worker1   <none>            <none>
rs-lab-a-or-56l95   1/1     Running             0          30m   10.244.2.12   k8s-worker2   <none>            <none>
rs-lab-a-or-56l95   1/1     Running             0          30m   10.244.2.12   k8s-worker2   <none>            <none>
rs-lab-a-or-rwm5l   0/1     Pending             0          0s    <none>        <none>        <none>            <none>
rs-lab-a-or-rwm5l   0/1     Pending             0          0s    <none>        k8s-worker2   <none>            <none>
rs-lab-a-or-rwm5l   0/1     ContainerCreating   0          0s    <none>        k8s-worker2   <none>            <none>
rs-lab-a-or-rwm5l   1/1     Running             0          0s    10.244.2.14   k8s-worker2   <none>            <none>

[root@k8s-master ~]# kubectl  get  rs  rs-lab-a-or
NAME          DESIRED   CURRENT   READY   AGE
rs-lab-a-or   2         2         2       32m

[root@k8s-master ~]# kubectl get pods --show-labels
NAME                READY   STATUS    RESTARTS   AGE     LABELS
p-none              1/1     Running   0          14m     lab=rs-lab-a-or
p-test              1/1     Running   0          16m     env=test,lab=rs-lab-a-or
rs-lab-a-or-56l95   1/1     Running   0          33m     app=web,env=test,lab=rs-lab-a-or
rs-lab-a-or-bsr78   1/1     Running   0          33m     app=web,env=prod,lab=rs-lab-a-or
rs-lab-a-or-rwm5l   1/1     Running   0          2m41s   app=web,env=prod,lab=rs-lab-a-or
```

---

## 실습 B — selector 사고 재현과 복구

- ReplicaSet selector를 너무 넓게/너무 좁게 설계했을 때 생기는 대표 사고를 YAML 1개로 만든 뒤 라벨 조작만으로 재현하고 원인 설명과 복구까지 단계별로 수행하는 실습
- selector가 너무 넓으면 다른 용도의 Pod까지 관리 대상으로 보일 수 있다.
- selector와 template.labels가 불일치하거나 selector가 너무 좁으면 Pod가 생성되지 않는 문제가 발생

```bash
	# namespace 생성
[root@k8s-master ~]# kubectl create namespace selector-lab-b

[root@k8s-master ~]# kubectl  get  namespaces

	# 생성한 namespace를 기본 namespace로 변경
[root@k8s-master ~]# kubectl config  set-context  --current  --namespace=selector-lab-b
```

- kubectl config set-context : kubectl Context 설정 변경
- --current : 현재 사용 중인 Context를 수정
- --namespace=selector-lab-b : 기본 Namespace를 selector-lab-b로 설정

```bash
[root@k8s-master ~]# vi rs-lab-b.yaml
```
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: rs-b

spec:
  replicas: 2
  selector:
    matchLabels:
      app: web

  template:
    metadata:
      labels:
        app: web
        team: a
    spec:
      containers:
      - name: nginx
        image: nginx:1.31
```

```bash
[root@k8s-master ~]# kubectl  apply  -f  rs-lab-b.yaml
replicaset.apps/rs-b created

[root@k8s-master ~]# kubectl  get  rs  rs-b
NAME   DESIRED   CURRENT   READY   AGE
rs-b   2         2         2       12s

[root@k8s-master ~]# kubectl  get  pods
NAME         READY   STATUS    RESTARTS   AGE
rs-b-j6dgp   1/1     Running   0          25s
rs-b-wwzdf   1/1     Running   0          25s

[root@k8s-master ~]# kubectl  get  pods  --show-labels
NAME         READY   STATUS    RESTARTS   AGE   LABELS
rs-b-j6dgp   1/1     Running   0          54s   app=web,team=a
rs-b-wwzdf   1/1     Running   0          54s   app=web,team=a
```

### STEP 2. ReplicaSet Controller에 포함되지않는 다른 용도의 Pod를 만들어 selector 사고 재현

```bash
[root@k8s-master ~]# kubectl  run  otherteam  --image=nginx:1.31  --labels app=web,team-b
error: unexpected label spec: team-b

[root@k8s-master ~]# kubectl  get  pods  -o  wide  --watch
NAME         READY   STATUS    RESTARTS   AGE   IP            NODE          NOMINATED NODE   READINESS GATES
rs-b-6p6rl   1/1     Running   0          25s   10.244.1.22   k8s-worker1   <none>            <none>
rs-b-fhcwp   1/1     Running   0          25s   10.244.2.17   k8s-worker2   <none>            <none>

[root@k8s-master ~]# kubectl  get  pods  --show-labels
NAME         READY   STATUS    RESTARTS   AGE    LABELS
rs-b-6p6rl   1/1     Running   0          3m5s   app=web,team=a
rs-b-fhcwp   1/1     Running   0          3m5s   app=web,team=a

[root@k8s-master ~]# kubectl  get  rs  rs-b
NAME   DESIRED   CURRENT   READY   AGE
rs-b   2         2         2       8m44s
```

- otherteam Pod는 ReplicaSet Controller인 rs-b가 만든 Pod가 아니다. 하지만 app=web 라벨 때문에 selector에 매치되기 때문에 라벨 충돌 사고의 출발점이다.

### STEP 3. selector가 너무 넓어서 생기는 문제를 안전한 라벨 설계로 개선

- rs-b selector가 app=web 하나라서 위험하다
- selector를 app=web AND team=a 로 바꾼 안전한 RS로 교체하라
- 실행(교체 방식: 기존 RS 삭제 후 새 RS 생성)
- 기존 RS 삭제

```bash
[root@k8s-master ~]# kubectl  delete  rs  rs-b
```

- 같은 파일을 수정해서 안전 버전으로 재적용

```bash
[root@k8s-master ~]# vi  rs-lab-b.yaml
```
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: rs-b

spec:
  replicas: 2
  selector:
    matchLabels:
      app: web
      team: a	# label 추가

  template:
    metadata:
      labels:
        app: web
        team: a
    spec:
      containers:
      - name: nginx
        image: nginx:1.31
```

```bash
[root@k8s-master ~]# kubectl  apply  -f  rs-lab-b.yaml  --dry-run=client
replicaset.apps/rs-b created (dry run)

[root@k8s-master ~]# kubectl  get  rs  rs-b
NAME   DESIRED   CURRENT   READY   AGE
rs-b   2         2         2       8s

[root@k8s-master ~]# kubectl  get  pods
NAME         READY   STATUS    RESTARTS   AGE
rs-b-k42s2   1/1     Running   0          17s
rs-b-v2p2m   1/1     Running   0          17s

	# pod에 설정된 label 확인
[root@k8s-master ~]# kubectl  get  pods  --show-labels
NAME         READY   STATUS    RESTARTS   AGE    LABELS
rs-b-k42s2   1/1     Running   0          2m4s   app=web,team=a
rs-b-v2p2m   1/1     Running   0          2m4s   app=web,team=a

	# RS가 관리대상으로 포함할 label 확인
[root@k8s-master ~]# kubectl  get  rs  rs-b  -o  wide
NAME   DESIRED   CURRENT   READY   AGE    CONTAINERS   IMAGES       SELECTOR
rs-b   2         2         2       111s   nginx        nginx:1.31   app=web,team=a

	# 수동으로 pod 생성
[root@k8s-master ~]# kubectl  run  otherteam  --image=nginx:1.31  --labels app=web,team=b
error: unexpected label spec: team-b

[root@k8s-master ~]# kubectl  get  pods  -o  wide  --watch
NAME         READY   STATUS              RESTARTS   AGE     IP            NODE          NOMINATED NODE   READINESS GATES
rs-b-k42s2   1/1     Running             0          4m31s   10.244.2.18   k8s-worker2   <none>            <none>
rs-b-v2p2m   1/1     Running             0          4m31s   10.244.1.23   k8s-worker1   <none>            <none>
otherteam    0/1     Pending             0          0s      <none>        <none>        <none>            <none>
otherteam    0/1     Pending             0          0s      <none>        k8s-worker1   <none>            <none>
otherteam    0/1     ContainerCreating   0          0s      <none>        k8s-worker1   <none>            <none>
otherteam    1/1     Running             0          2s      10.244.1.24   k8s-worker1   <none>            <none>

[root@k8s-master ~]# kubectl  delete  rs  rs-b
replicaset.apps "rs-b" deleted from selector-lab-b namespace

[root@k8s-master ~]# kubectl  delete  pods  otherteam
pod "otherteam" deleted from selector-lab-b namespace

[root@k8s-master ~]# kubectl config  set-context  --current  --namespace=default
Context "kubernetes-admin@kubernetes" modified.
```

정리: **selector**를 너무 넓게 잡으면 다른 용도의 Pod까지 관리 대상으로 편입되는 사고가 발생할 수 있으므로, 실무에서는 라벨을 목적별로 세밀하게 조합해 selector를 설계해야 한다. 이제 ReplicaSet 위에서 무중단 배포와 롤백까지 담당하는 **Deployment**를 살펴본다.

---

## Deployment

- **Deployment**는 Pod를 어떻게 만들고, 어떻게 업데이트하고, 문제가 생기면 어떻게 되돌릴지를 쿠버네티스에게 선언적으로 알려주는 컨트롤러다.
- Deployment = Pod 생성 + Pod 개수 유지 + 무중단 업데이트 + 버전 관리 + 롤백

### Deployment를 사용하는 이유

Pod만 사용하면 생기는 문제
- Pod를 직접 생성하게되면 파드가 죽으면 재시작되지 않고 사라진다.
- 재시작, 개수 유지, 업데이트를 직접 해야 한다
- 서비스 중 이미지 버전을 바꾸려면 기존 Pod 삭제 --> 새 Pod 생성 --> 서비스 중단 가능성 발생
- Deployment는 이 문제를 해결한다.

Deployment가 해주는 일
- Pod 개수를 항상 유지
- 새로운 버전으로 천천히 교체
- 문제 발생 시 이전 버전으로 복구
- 그래서 실무에서는 Pod 단독 생성 거의 안 하고 Deployment를 사용한다.

### Deployment의 기본 구조

Deployment는 내부적으로 이런 구조를 가진다.

```
Deployment
└ ReplicaSet
    └ Pod
    └ Pod
    └ Pod
```

- 즉 Deployment는 직접 Pod를 만들지 않는다.
- Deployment가 ReplicaSet을 만든다.
- ReplicaSet이 Pod를 만든다.

### Deployment가 관리하는 핵심 기능

Deployment가 하는 핵심 역할은 5가지

1) Pod 개수 유지
   - replicas 값만큼 항상 유지
   - Pod가 죽으면 자동 재생성

2) 버전 관리
   - 이미지 변경 시 새로운 ReplicaSet 생성
   - 이전 ReplicaSet은 기록으로 남음

3) 무중단 업데이트 (Rolling Update)
   - Pod를 한 번에 다 지우지 않음
   - 하나씩 교체하면서 서비스 유지

4) 롤백 (Rollback)
   - 새 버전에서 문제가 생기면 이전 정상 버전으로 되돌림

5) 선언적 관리
   - 이 상태가 되길 원한다만 작성하면 쿠버네티스가 알아서 맞춘다

### Deployment YAML 기본 예제

```yaml
apiVersion: apps/v1
kind: Deployment		# kind가 Deployment인걸 제외하면 나머지는 ReplicaSet과 설정이 동일하다.
metadata:
  name: nginx-deploy

spec:
  replicas: 3
  selector:
    matchLabels:
      app: web

  template:
    metadata:
      labels:
        app: web

    spec:
      containers:
      - name: nginx
        image: nginx:1.31
        ports:
        - containerPort: 80
```

- metadata.name : Deployment 이름
- spec.replicas : 유지할 Pod 개수
- spec.selector.matchLabels : 어떤 Pod를 관리할지 기준 라벨
- spec.template : 실제 Pod 설계도
- template.metadata.labels : selector와 반드시 일치해야 함
- template.spec.containers : Pod 안에 들어갈 컨테이너 정의

### template이 중요한 이유

- Deployment에서 가장 중요한 부분은 spec.template 이다.
- 이 부분은 Pod 설계도이다.
- Deployment는 template을 보고 Pod를 만든다.
- template이 바뀌면 새 버전으로 인식한다.
- 즉 이미지 변경, 환경변수 변경, 포트 변경 = 전부 새로운 ReplicaSet 생성

### Deployment 업데이트 흐름

이미지를 변경하면 내부에서 일어나는 일
- 기존 ReplicaSet (nginx:1.28.4) 유지
- 새로운 ReplicaSet (nginx:1.29.04) 생성
- 새 Pod 1개 생성
- 기존 Pod 1개 종료
- 이 과정을 반복
- 최종적으로 새 버전만 남게된다.
- 이게 바로 Rolling Update다.

중요 포인트
- 서비스 중단 없이 교체
- Pod를 한 번에 다 없애지 않는다.

### Rollback이 가능한 이유

- Deployment는 이전 상태를 기록으로 남긴다. 그래서 최신 버전 문제 발생 시 이전 안정 버전으로 즉시 복구 가능
- 이전 버전의 ReplicaSet이 남아 있기 때문에 가능한 구조다.

### Deployment와 ReplicaSet 차이

- ReplicaSet
  - Pod 개수 유지 전용
  - 업데이트, 롤백 기능 없음

- Deployment
  - ReplicaSet을 관리
  - 업데이트 전략 제공
  - 롤백 가능
  - 운영용 표준 방식

정리
- ReplicaSet은 엔진
- Deployment는 운전자

### 언제 Deployment를 쓰는가

Deployment를 사용하는 경우
- 웹 서버
- API 서버
- 백엔드 서비스
- 프론트엔드
- 대부분의 일반 서비스

Deployment를 쓰지 않는 경우
- DB (StatefulSet 사용)
- 노드마다 1개씩 필요한 경우 (DaemonSet)

정리: Deployment는 ReplicaSet을 감싸서 버전 관리, 롤백, 무중단 업데이트를 자동화하는 운영 표준 컨트롤러이며, 그 무중단 업데이트의 실제 동작 방식이 바로 **Rolling Update**다.

---

## Rolling Update

- **Rolling Update**(롤링 업데이트)는 서비스를 멈추지 않고 기존 Pod를 새 Pod로 하나씩 교체하는 업데이트 방식이다. 즉, 전체 Pod를 한 번에 삭제하지 않는다.
- 기존 Pod를 유지한 상태에서 새 Pod를 조금씩 늘리고 오래된 Pod를 조금씩 줄인다. 그래서 무중단 업데이트라고 부른다.

### 왜 롤링 업데이트가 필요한가?

- 롤링 업데이트가 없으면 --> 모든 Pod 삭제 --> 새 Pod 생성 --> 그 사이 서비스 중단 발생
- 운영 환경에서는 몇 초의 중단도 큰 장애가 된다.
- 롤링 업데이트를 사용하면 사용자는 서비스 중단을 느끼지 못한다. (서버를 켠 상태에서 엔진만 교체)

### 롤링 업데이트는 누가 수행하는가

- Pod가 직접 하지 않는다.
- 사용자가 직접 제어하지도 않는다.
- Deployment가 자동으로 수행
- Deployment는 새 ReplicaSet 생성하고 기존 ReplicaSet은 줄이고 새 ReplicaSet은 늘린다

### 롤링 업데이트 기본 동작 흐름

Pod가 3개 있는 Deployment에서 이미지를 변경시 내부에 동작
1) 기존 Pod 3개 유지
2) 새 Pod 1개 생성
3) 기존 Pod 1개 종료
4) 새 Pod 2개
5) 기존 Pod 2개
6) 반복
7) 최종적으로 새 Pod 3개만 남음

- 항상 최소 1개 이상의 Pod는 살아 있음
- 서비스는 계속 응답 가능

롤링 업데이트를 그림처럼 이해하기

```
업데이트 전
[ OLD ][ OLD ][ OLD ]

업데이트 중
[ NEW ][ OLD ][ OLD ]
[ NEW ][ NEW ][ OLD ]

업데이트 완료
[ NEW ][ NEW ][ NEW ]
```

- 이 흐름을 자동으로 관리하는 게 Deployment다.

### 롤링 업데이트가 발생하는 조건

다음 중 하나라도 바뀌면 롤링 업데이트가 발생한다.
- 컨테이너 이미지 변경
- 환경변수 변경
- 포트 변경
- 커맨드 변경
- 볼륨 설정 변경

공통점
- spec.template 안의 내용이 변경됨
- 즉, Deployment는 template이 바뀌면 새 버전으로 인식한다.

### 롤링 업데이트 전략 옵션

- Deployment에는 롤링 업데이트 속도와 안정성을 조절하는 옵션이 있다.
- 대표 옵션 2가지: **maxSurge**, **maxUnavailable**
- 이 옵션들은 얼마나 여유 있게 업데이트할 것인가를 정한다.

**maxSurge**

- maxSurge는 기존 Pod 개수보다 초과해서 만들 수 있는 Pod 수다.

예시
- replicas: 3
- maxSurge: 1

의미
- 최대 4개 Pod까지 동시에 존재 가능
- 새 Pod를 먼저 하나 만들고
- 기존 Pod를 나중에 하나 줄임
- 즉, 안전하지만 자원을 조금 더 사용한다.

**maxUnavailable**

- maxUnavailable은 업데이트 중 사용할 수 없게 되어도 되는 Pod 수다.

예시
- replicas: 3
- maxUnavailable: 1

의미
- 동시에 최대 1개 Pod까지는 내려가도 허용
- 즉, 자원은 적게 쓰지만 조금 더 위험할 수 있다.

RollingUpdate 설정 예제

```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxSurge: 1
    maxUnavailable: 1
```

- Pod를 하나 더 만들 수 있고
- 동시에 하나까지는 내려도 된다
- 가장 일반적인 기본 설정

### 롤링 업데이트 vs 재생성 방식

- Recreate 방식
  - 기존 Pod 전부 삭제
  - 새 Pod 전부 생성
  - 서비스 중단 발생

- RollingUpdate 방식
  - Pod를 하나씩 교체
  - 서비스 유지
  - 운영 환경에 적합

- Deployment 기본값은 RollingUpdate다.

**1단계: 새 파드 생성 (maxSurge)**
```
old-pod-1 (v1)  서비스 중
old-pod-2 (v1)  서비스 중
old-pod-3 (v1)  서비스 중
new-pod-1 (v2)  생성됨
```
- 총 파드: 4
- 여기까지는 모든 pod가 동작한다.
- 서비스 파드 수: 4 (readiness 통과 기준)

**2단계: 기존 파드 1개 종료 (maxUnavailable)**
```
old-pod-2 (v1)  서비스 중
old-pod-3 (v1)  서비스 중
new-pod-1 (v2)  서비스 중
```
- old-pod-1이 제거됨
- 총 파드 : 3
- 서비스 파드 수 : 3
- pod-1이 업데이트 된 게 아니라 old-pod-1이 삭제되고 new-pod-1이 서비스한다.

**3단계: 다시 새 파드 생성**
```
old-pod-2 (v1)  서비스 중
old-pod-3 (v1)  서비스 중
new-pod-1 (v2)  서비스 중
new-pod-2 (v2)  생성됨
```
- 총 파드: 4

**4단계: 또 기존 파드 1개 종료**
```
old-pod-3 (v1)  서비스 중
new-pod-1 (v2)  서비스 중
new-pod-2 (v2)  서비스 중
```

**5단계: 반복**
```
new-pod-1 (v2)  서비스 중
new-pod-2 (v2)  서비스 중
new-pod-3 (v2)  서비스 중
```

### Deployment 실습 — 생성/확인/삭제

```bash
[root@k8s-master ~]# vi deploy-nginx.yaml
```
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deploy-nginx

spec:
  replicas: 3
  selector:
    matchLabels:
      app: web

  template:
    metadata:
      name: nginx-pod
      labels:
        app: web
    spec:
      containers:
      - name: nginx-container
        image: nginx:1.31
```

```bash
[root@k8s-master ~]# kubectl  apply  -f  deploy-nginx.yaml  --dry-run=client
deployment.apps/deploy-nginx created (dry run)

[root@k8s-master ~]# kubectl  apply  -f  deploy-nginx.yaml
deployment.apps/deploy-nginx created

[root@k8s-master ~]# kubectl  get  deployments.apps
NAME           READY   UP-TO-DATE   AVAILABLE   AGE
deploy-nginx   3/3     3            3           32s
```

- READY : 3/3
  - 정상 준비된 Pod 수 / 원하는 Pod 수
  - 현재 3개를 원하고, 3개 모두 정상 준비 완료된 상태
- UP-TO-DATE : 3
  - 현재 Deployment에 설정된 최신 설정(template)으로 실행 중인 Pod가 3개라는 의미
  - 예를 들어 이미지를 변경해서 롤링 업데이트 중이면 일시적으로 READY와 숫자가 다를 수 있다.
- AVAILABLE : 3
  - 실제로 서비스 가능한 상태의 Pod가 3개라는 의미
  - 즉, 트래픽을 받을 수 있는 Pod가 3개

```bash
[root@k8s-master ~]# kubectl  get  deployments.apps  -o  wide
NAME           READY   UP-TO-DATE   AVAILABLE   AGE     CONTAINERS        IMAGES       SELECTOR
deploy-nginx   3/3     3            3           5m25s   nginx-container   nginx:1.31   app=web

[root@k8s-master ~]# kubectl  get  pods
NAME                            READY   STATUS    RESTARTS   AGE
deploy-nginx-5796ddf486-7cgpx   1/1     Running   0          3m31s
deploy-nginx-5796ddf486-fz9mg   1/1     Running   0          3m31s
deploy-nginx-5796ddf486-xlb2m   1/1     Running   0          3m31s
```

- deploy-nginx : Deployment
- deploy-nginx-5796ddf486 : ReplicaSet
- deploy-nginx-5796ddf486-7cgpx / -fz9mg / -xlb2m : pod

```bash
[root@k8s-master ~]# kubectl  delete  deployments.apps deploy-nginx
deployment.apps "deploy-nginx" deleted from default namespace

[root@k8s-master ~]# kubectl  get  pods
No resources found in default namespace.
```

정리: Rolling Update는 **maxSurge**(초과 생성 허용치)와 **maxUnavailable**(서비스 불가 허용치)로 교체 속도와 안정성을 조절하며, 이어지는 실습에서는 실제 `kubectl` 명령으로 이 업데이트와 **롤백(Rollback)** 을 직접 수행해 본다.

---

## Rolling Update & Rollback (실습)

### Rolling Update 명령

- Deployment에서 실행 중인 컨테이너 이미지를 새로운 버전으로 변경한다.
- 형식 : `kubectl set image deployment <deploy_name> <container_name>=<new_version_image> --record`
- 이 명령을 실행하면 Deployment가 이를 감지하고 자동으로 롤링 업데이트(Rolling Update)를 수행한다.
  - `<deploy_name>` : Deployment 이름
  - `<container_name>` : Deployment 안에 정의된 컨테이너 이름
  - `<new_version_image>` : 변경할 새 컨테이너 이미지 (예: nginx:1.29.0)

예시: `kubectl set image deployment deploy-nginx nginx-container=nginx:1.29.0 --record`

### RollBack 명령

- 해당 Deployment의 배포 이력을 확인한다.
- 과거에 어떤 이미지와 설정으로 배포되었는지 버전 목록을 보여준다.
- 형식 : `kubectl rollout undo deployment <deploy_name>` : 이전 정상 버전으로 복구
- 형식 : `kubectl rollout history deployment <deploy_name>` : 배포 이력 확인
- 가장 최근 이전 버전으로 Deployment를 되돌린다.
- 롤링 업데이트 도중 장애가 발생했을 때 사용하는 명령어다.

예시: `kubectl rollout history deployment deploy-nginx`

### deploy-nginx.yaml 파일 수정 및 첫 배포

```bash
[root@k8s-master ~]# vi deploy-nginx.yaml
```
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deploy-update	# 수정

spec:
  replicas: 3
  selector:
    matchLabels:
      app: web

  template:
    metadata:
      name: nginx-pod
      labels:
        app: web
    spec:
      containers:
      - name: nginx-web	# 수정
        image: nginx:1.31
```

```bash
[root@k8s-master ~]# kubectl  apply  -f  deploy-nginx.yaml
deployment.apps/deploy-update created

[root@k8s-master ~]# kubectl  get  pods
NAME                              READY   STATUS    RESTARTS   AGE
deploy-update-fdd46fc97-rsvt2     1/1     Running   0          34s
deploy-update-fdd46fc97-x6qw9     1/1     Running   0          34s
deploy-update-fdd46fc97-zxpws     1/1     Running   0          34s

[root@k8s-master ~]# kubectl  describe  deployments.apps  deploy-update | grep Image
    Image:         nginx:1.31

[root@k8s-master ~]# kubectl  rollout  history  deployment  deploy-update
deployment.apps/deploy-update
REVISION  CHANGE-CAUSE
1         <none>
```

- 아직 버전 기록이 없다.

```bash
[root@k8s-master ~]# kubectl  delete  deployments.apps  deploy-update
deployment.apps "deploy-update" deleted from default namespace

[root@k8s-master ~]# kubectl  get  pods
No resources found in default namespace.
```

### 방법 1 — `--record` 옵션 (deprecated, 비권장)

```bash
[root@k8s-master ~]# kubectl  create  -f  deploy-nginx.yaml  --record=true
Flag --record has been deprecated, --record will be removed in the future
deployment.apps/deploy-update created

[root@k8s-master ~]# kubectl  get deployments.apps
NAME            READY   UP-TO-DATE   AVAILABLE   AGE
deploy-update   3/3     3            3           75

[root@k8s-master ~]# kubectl  get pods
NAME                              READY   STATUS    RESTARTS   AGE
deploy-update-fdd46fc97-fc9ht     1/1     Running   0          54s
deploy-update-fdd46fc97-npm9q     1/1     Running   0          54s
deploy-update-fdd46fc97-rtvl4     1/1     Running   0          54s

[root@k8s-master ~]# kubectl  rollout  history  deployment  deploy-update
deployment.apps/deploy-update
REVISION  CHANGE-CAUSE
1         kubectl create --filename=deploy-nginx.yaml --record=true

[root@k8s-master ~]# kubectl  rollout  history  deployment  deploy-update  --revision=1
deployment.apps/deploy-update with revision #1
Pod Template:
  Labels:       app=web
        pod-template-hash=fdd46fc97
  Annotations:  kubernetes.io/change-cause: kubectl create --filename=deploy-nginx.yaml --record=true
  Containers:
   nginx-web:
    Image:      nginx:1.31
    Port:       <none>
    Host Port:  <none>
    Environment: <none>
    Mounts:     <none>
  Volumes:      <none>
  Node-Selectors: <none>
  Tolerations:  <none>

[root@k8s-master ~]# kubectl  delete  deployments.apps  deploy-update
deployment.apps "deploy-update" deleted from default namespace
```

### 방법 2 — annotation 방식 (권장)

```bash
[root@k8s-master ~]# kubectl  create  -f  deploy-nginx.yaml
Flag --record has been deprecated, --record will be removed in the future
deployment.apps/deploy-update created

[root@k8s-master ~]# kubectl  get  deployments.apps  deploy-update
NAME            READY   UP-TO-DATE   AVAILABLE   AGE
deploy-update   3/3     3            3           19s

[root@k8s-master ~]# kubectl  get  pods
NAME                              READY   STATUS    RESTARTS   AGE
deploy-update-fdd46fc97-59p2g     1/1     Running   0          24s
deploy-update-fdd46fc97-k67fr     1/1     Running   0          24s
deploy-update-fdd46fc97-rkdx6     1/1     Running   0          24s

[root@k8s-master ~]# kubectl  annotate  deployments.apps  deploy-update \
> kubernetes.io/change-cause="nginx 1.31 이미지 최초 배포" --overwrite
deployment.apps/deploy-update annotated

[root@k8s-master ~]# kubectl  rollout history deployment  deploy-update
deployment.apps/deploy-update
REVISION  CHANGE-CAUSE
1         nginx 1.31 이미지 최초 배포

[root@k8s-master ~]# kubectl  rollout history deployment  deploy-update  --revision=1
deployment.apps/deploy-update with revision #1
Pod Template:
  Labels:       app=web
        pod-template-hash=fdd46fc97
  Annotations:  kubernetes.io/change-cause: nginx 1.31 이미지 최초 배포
  Containers:
   nginx-web:
    Image:      nginx:1.31
    Port:       <none>
    Host Port:  <none>
    Environment: <none>
    Mounts:     <none>
  Volumes:      <none>
  Node-Selectors: <none>
  Tolerations:  <none>
```

### Image Version up (실습)

```bash
	# 터미널 1
[root@k8s-master ~]# kubectl  get  pods  -o  wide  --watch
```

- 형식 : `kubectl set image deployment <deploy_name> <container_name>=<new_version_image>`

```bash
	# 터미널 2

# 위의 방법 1
[root@k8s-master ~]# kubectl  set image  deployments  deploy-update  nginx-web=nginx:1.31.1  --record

# 위의 방법 2 (실습 진행)
[root@k8s-master ~]# kubectl  set image  deployments  deploy-update  nginx-web=nginx:1.31.1

[root@k8s-master ~]# kubectl  get  pods  -o  wide  --watch
NAME                              READY   STATUS              RESTARTS   AGE   IP            NODE          NOMINATED NODE   READINESS GATES
deploy-update-fdd46fc97-84882     1/1     Running             0          4s    10.244.1.11   k8s-worker1   <none>            <none>
deploy-update-fdd46fc97-lssnv     1/1     Running             0          4s    10.244.1.12   k8s-worker1   <none>            <none>
deploy-update-fdd46fc97-thtfw     1/1     Running             0          4s    10.244.2.8    k8s-worker2   <none>            <none>
deploy-update-64f9b77664-cznwv    0/1     Pending             0          0s    <none>        <none>        <none>            <none>
deploy-update-64f9b77664-cznwv    0/1     Pending             0          0s    <none>        k8s-worker2   <none>            <none>
deploy-update-64f9b77664-cznwv    0/1     ContainerCreating   0          0s    <none>        k8s-worker2   <none>            <none>
deploy-update-64f9b77664-cznwv    1/1     Running             0          11s   10.244.2.9    k8s-worker2   <none>            <none>
deploy-update-fdd46fc97-thtfw     1/1     Terminating         0          77s   10.244.2.8    k8s-worker2   <none>            <none>
deploy-update-64f9b77664-tcr24    0/1     Pending             0          0s    <none>        <none>        <none>            <none>
deploy-update-fdd46fc97-thtfw     1/1     Terminating         0          77s   10.244.2.8    k8s-worker2   <none>            <none>
deploy-update-64f9b77664-tcr24    0/1     Pending             0          0s    <none>        k8s-worker1   <none>            <none>
deploy-update-64f9b77664-tcr24    0/1     ContainerCreating   0          0s    <none>        k8s-worker1   <none>            <none>
deploy-update-fdd46fc97-thtfw     0/1     Completed           0          77s   10.244.2.8    k8s-worker2   <none>            <none>
deploy-update-fdd46fc97-thtfw     0/1     Completed           0          78s   10.244.2.8    k8s-worker2   <none>            <none>
deploy-update-fdd46fc97-thtfw     0/1     Completed           0          78s   10.244.2.8    k8s-worker2   <none>            <none>
deploy-update-64f9b77664-tcr24    1/1     Running             0          10s   10.244.1.13   k8s-worker1   <none>            <none>
deploy-update-fdd46fc97-84882     1/1     Terminating         0          87s   10.244.1.11   k8s-worker1   <none>            <none>
deploy-update-64f9b77664-sqhmb    0/1     Pending             0          0s    <none>        <none>        <none>            <none>
deploy-update-fdd46fc97-84882     1/1     Terminating         0          87s   10.244.1.11   k8s-worker1   <none>            <none>
deploy-update-64f9b77664-sqhmb    0/1     Pending             0          0s    <none>        k8s-worker2   <none>            <none>
deploy-update-64f9b77664-sqhmb    0/1     ContainerCreating   0          0s    <none>        k8s-worker2   <none>            <none>
deploy-update-fdd46fc97-84882     0/1     Completed           0          87s   10.244.1.11   k8s-worker1   <none>            <none>
deploy-update-64f9b77664-sqhmb    1/1     Running             0          1s    10.244.2.10   k8s-worker2   <none>            <none>
deploy-update-fdd46fc97-lssnv     1/1     Terminating         0          88s   10.244.1.12   k8s-worker1   <none>            <none>
deploy-update-fdd46fc97-lssnv     1/1     Terminating         0          88s   10.244.1.12   k8s-worker1   <none>            <none>
deploy-update-fdd46fc97-lssnv     0/1     Completed           0          88s   10.244.1.12   k8s-worker1   <none>            <none>
deploy-update-fdd46fc97-lssnv     0/1     Completed           0          88s   10.244.1.12   k8s-worker1   <none>            <none>
deploy-update-fdd46fc97-lssnv     0/1     Completed           0          88s   10.244.1.12   k8s-worker1   <none>            <none>
deploy-update-fdd46fc97-84882     0/1     Completed           0          88s   10.244.1.11   k8s-worker1   <none>            <none>
deploy-update-fdd46fc97-84882     0/1     Completed           0          88s   10.244.1.11   k8s-worker1   <none>            <none>

[root@k8s-master ~]# kubectl  annotate  deployments.apps  deploy-update \
> kubernetes.io/change-cause="2026-0819 10:54  nginx 1.31.1 이미저 버전업" --overwrite
deployment.apps/deploy-update annotated

[root@k8s-master ~]# kubectl  rollout  history deployment  deploy-update
deployment.apps/deploy-update
REVISION  CHANGE-CAUSE
1         nginx 1.31 이미지 최초 배포
2         2026-0819 10:54  nginx 1.31.1 이미저 버전업

[root@k8s-master ~]# kubectl  rollout  history deployment  deploy-update  --revision=2
deployment.apps/deploy-update with revision #2
Pod Template:
  Labels:       app=web
        pod-template-hash=64f9b77664
  Annotations:  kubernetes.io/change-cause: 2026-0819 10:54  nginx 1.31.1 이미저 버전업
  Containers:
   nginx-web:
    Image:      nginx:1.31.1
    Port:       <none>
    Host Port:  <none>
    Environment: <none>
    Mounts:     <none>
  Volumes:      <none>
  Node-Selectors: <none>
  Tolerations:  <none>

[root@k8s-master ~]# kubectl  describe  deployments.apps
Name:                   deploy-update
Namespace:              default
CreationTimestamp:      Wed, 19 Aug 2026 10:53:52 +0900
Labels:                 <none>
Annotations:            deployment.kubernetes.io/revision: 2
                        kubernetes.io/change-cause: 2026-0819 10:54  nginx 1.31.1 이미저 버전업
Selector:               app=web
Replicas:               3 desired | 3 updated | 3 total | 3 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=web
  Containers:
   nginx-web:
    Image:         nginx:1.31.1
    Port:          <none>
    Host Port:     <none>
    Environment:   <none>
    Mounts:        <none>
  Volumes:         <none>
  Node-Selectors:  <none>
  Tolerations:     <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  deploy-update-fdd46fc97 (0/0 replicas created)
NewReplicaSet:   deploy-update-64f9b77664 (3/3 replicas created)
Events:
  Type    Reason              Age    From                    Message
  ----    ------              ----   ----                    -------
  Normal  ScalingReplicaSet   4m45s  deployment-controller   Scaled up replica set deploy-update-fdd46fc97 from 0 to 3
  Normal  ScalingReplicaSet   4m24s  deployment-controller   Scaled up replica set deploy-update-64f9b77664 from 0 to 1
  Normal  ScalingReplicaSet   4m23s  deployment-controller   Scaled down replica set deploy-update-fdd46fc97 from 3 to 2
  Normal  ScalingReplicaSet   4m23s  deployment-controller   Scaled up replica set deploy-update-64f9b77664 from 1 to 2
  Normal  ScalingReplicaSet   4m22s  deployment-controller   Scaled down replica set deploy-update-fdd46fc97 from 2 to 1
  Normal  ScalingReplicaSet   4m22s  deployment-controller   Scaled up replica set deploy-update-64f9b77664 from 2 to 3
  Normal  ScalingReplicaSet   4m21s  deployment-controller   Scaled down replica set deploy-update-fdd46fc97 from 1 to 0

[root@k8s-master ~]# kubectl  get pods
NAME                              READY   STATUS    RESTARTS   AGE
deploy-update-64f9b77664-9kp96    1/1     Running   0          10m
deploy-update-64f9b77664-c4tgn    1/1     Running   0          10m
deploy-update-64f9b77664-trnd9    1/1     Running   0          10m

[root@k8s-master ~]# kubectl  describe  pods  deploy-update-64f9b77664-9kp96

[root@k8s-master ~]# kubectl  delete  deployments.apps  deploy-update
deployment.apps "deploy-update" deleted from default namespace
```

### Rollback 실습

- 형식 : `kubectl rollout undo deployment <deploy_name>` : 이전 정상 버전으로 복구
- 형식 : `kubectl rollout history deployment <deploy_name>` : 배포 이력 확인

```bash
	# 배포 이력 확인
[root@k8s-master ~]# kubectl  rollout  history deployment  deploy-update
deployment.apps/deploy-update
REVISION  CHANGE-CAUSE
1         nginx 1.31 이미지 최초 배포
2         2026-0819 10:54  nginx 1.31.1 이미저 버전업

[root@k8s-master ~]# kubectl  rollout  history deployment  deploy-update  --revision=1
[root@k8s-master ~]# kubectl  rollout  history deployment  deploy-update  --revision=2

	# Rollback
[root@k8s-master ~]# kubectl rollout undo deployment  deploy-update
deployment.apps/deploy-update rolled back

[root@k8s-master ~]# kubectl  rollout  history deployment  deploy-update
deployment.apps/deploy-update
REVISION  CHANGE-CAUSE
2         2026-0819 10:54  nginx 1.31.1 이미지 버전업
3         nginx 1.31 이미지 최초 배포
```

```
1 = nginx:1.31
2 = nginx:1.31.1
3 = nginx:1.31 (Revision 1의 설정으로 Rollback하면서 동일한 설정이 새로운 Revision 3으로 기록되고,
                 Revision 1은 history 목록에서 사라진다.)
```

```bash
[root@k8s-master ~]# kubectl  describe  deployments.apps  deploy-update | grep Image
    Image:         nginx:1.31

[root@k8s-master ~]# kubectl annotate deployment deploy-update \
> kubernetes.io/change-cause="Revision 2에서 nginx 1.31.1 버전을 nginx 1.31로 Rollback" \
> --overwrite

[root@k8s-master ~]# kubectl  rollout  history deployment  deploy-update
deployment.apps/deploy-update
REVISION  CHANGE-CAUSE
2         2026-0819 10:54  nginx 1.31.1 이미저 버전업
3         Revision 2에서 nginx 1.31.1 버전을 nginx 1.31로 Rollback

	# deployments 삭제
[root@k8s-master ~]# kubectl delete deployments.apps deploy-update
deployment.apps "deploy-update" deleted
```

### Deployment 롤링업데이트 + 히스토리(change-cause) 기록 + 롤백 (Step2)

```bash
[root@k8s-master ~]# vi deploy-update-step2.yaml
```
```yaml
apiVersion: apps/v1    	# Deployment에서 사용하는 API 버전
kind: Deployment         	# 생성할 리소스 종류
metadata:
  name: deploy-update 	# Deployment 이름

spec:
  replicas: 3                 	# 유지할 Pod 개수
  revisionHistoryLimit: 5   	# 이전 ReplicaSet(Revision) 이력을 최대 5개까지 보관
  strategy:
    type: RollingUpdate     	# 배포 전략을 RollingUpdate 방식으로 설정
    rollingUpdate:
      maxSurge: 1        	# 업데이트 중 원하는 Pod 수보다 최대 1개까지 추가 생성 가능
      maxUnavailable: 1      	# 업데이트 중 최대 1개의 Pod까지 사용 불가 상태 허용
  selector:
    matchLabels:
      app: web              	# app=web 라벨을 가진 Pod를 관리 대상으로 선택

  template:
    metadata:
      labels:
        app: web       		# Deployment selector와 반드시 일치해야 하는 라벨
        track: stable
    spec:
      containers:
      - name: nginx-container 		# 컨테이너 이름
        image: nginx:1.31     		# 사용할 nginx 이미지 버전
        ports:
        - containerPort: 80    		# 컨테이너가 사용하는 포트 정보
```

```bash
[root@k8s-master ~]# kubectl  apply  -f  deploy-update-step2.yaml  --dry-run=client
deployment.apps/deploy-update created (dry run)

[root@k8s-master ~]# kubectl  apply  -f  deploy-update-step2.yaml
deployment.apps/deploy-update created

[root@k8s-master ~]# kubectl  get  deployments.apps  deploy-update
NAME            READY   UP-TO-DATE   AVAILABLE   AGE
deploy-update   3/3     3            3           6s

[root@k8s-master ~]# kubectl  get  pods
NAME                              READY   STATUS    RESTARTS   AGE
deploy-update-6747747c6f-mf2mh    1/1     Running   0          17s
deploy-update-6747747c6f-q5pvz    1/1     Running   0          17s
deploy-update-6747747c6f-zpl4k    1/1     Running   0          17s

	# 초기 배포 revision에 대한 변경 사유 기록
[root@k8s-master ~]# kubectl  annotate deployment deploy-update \
kubernetes.io/change-cause="revision 1 : initial deploy nginx 1.31" --overwrite
deployment.apps/deploy-update annotated

	# 롤링 업데이트 버전 확인
[root@k8s-master ~]# kubectl  rollout  history  deployment  deploy-update
deployment.apps/deploy-update
REVISION  CHANGE-CAUSE
1         revision 1 : initial deploy nginx 1.31

[root@k8s-master ~]# kubectl  rollout  history  deployment  deploy-update  --revision=1
deployment.apps/deploy-update with revision #1
Pod Template:
  Labels: 	app=web
        pod-template-hash=6747747c6f
        track=stable
  Annotations:  kubernetes.io/change-cause: revision 1 : initial deploy nginx 1.31
  Containers:
   nginx-container:
    Image:      nginx:1.31
    Port:       80/TCP
    Host Port:  0/TCP
    Environment: <none>
    Mounts:     <none>
  Volumes:      <none>
  Node-Selectors: <none>
  Tolerations:  <none>

[root@k8s-master ~]# kubectl  get  pods
NAME                              READY   STATUS    RESTARTS   AGE
deploy-update-6747747c6f-mf2mh    1/1     Running   0          17s
deploy-update-6747747c6f-q5pvz    1/1     Running   0          17s
deploy-update-6747747c6f-zpl4k    1/1     Running   0          17s
```

**이미지 버전 업 (Rolling Update)**

```bash
[root@k8s-master ~]# kubectl  set  image  deployments  deploy-update  nginx-container=nginx:1.13.1
deployment.apps/deploy-update image updated

	# 변경 이력(change-cause) 기록
[root@k8s-master ~]# kubectl  annotate  deployments  deploy-update  \
> kubernetes.io/change-cause="revision 2 : nginx:1.31  -->  nginx:1.13.1 version update" \
> --overwrite
deployment.apps/deploy-update annotated

	# Deployment의 롤링 업데이트 진행 상태를 확인
[root@k8s-master ~]# kubectl  rollout  status  deployment  deploy-update
deployment "deploy-update" successfully rolled out

	# 롤아웃 상태 및 히스토리 확인
[root@k8s-master ~]# kubectl  rollout  history  deployment  deploy-update
deployment.apps/deploy-update
REVISION  CHANGE-CAUSE
1         revision 1 : initial deploy nginx 1.31
2         revision 2 : nginx:1.31  -->  nginx:1.13.1 version update

[root@k8s-master ~]# kubectl  rollout  history  deployment  deploy-update  --revision=2
deployment.apps/deploy-update with revision #2
Pod Template:
  Labels:       app=web
        pod-template-hash=59c675765b
        track=stable
  Annotations:  kubernetes.io/change-cause: revision 2 : nginx:1.31  -->  nginx:1.13.1 version update
  Containers:
   nginx-container:
    Image:      nginx:1.13.1
    Port:       80/TCP
    Host Port:  0/TCP
    Environment: <none>
    Mounts:     <none>
  Volumes:      <none>
  Node-Selectors: <none>
  Tolerations:  <none>

[root@k8s-master ~]# kubectl  describe  deployments.apps  deploy-update  | grep Image
    Image:         nginx:1.13.1
```

**롤백**

```bash
# 이전 revision으로 롤백
[root@k8s-master ~]# kubectl  rollout  undo  deployment  deploy-update
deployment.apps/deploy-update rolled back

# 특정 revision으로 롤백
[root@k8s-master ~]# kubectl  rollout  undo  deployment  deploy-update  --to-revision=1
deployment.apps/deploy-update rolled back

	# 히스토리 확인
[root@k8s-master ~]# kubectl  rollout  history  deployment  deploy-update
deployment.apps/deploy-update
REVISION  CHANGE-CAUSE
2         revision 2 : nginx:1.31  -->  nginx:1.13.1 version update--overwrite
3         revision 1 : initial deploy nginx 1.31

	# 히스토리 수정
[root@k8s-master ~]# kubectl annotate deployment deploy-update \
kubernetes.io/change-cause="revision 3 : rollback to nginx 1.31"  --overwrite
deployment.apps/deploy-update annotated

	# 히스토리 확인
[root@k8s-master ~]# kubectl  rollout  history  deployment  deploy-update
deployment.apps/deploy-update
REVISION  CHANGE-CAUSE
2         revision 2 : nginx:1.31  -->  nginx:1.13.1 version update--overwrite
3         revision 3 : rollback to nginx 1.31

	# 이미지 변경 확인
[root@k8s-master ~]# kubectl  describe  deployments.apps  deploy-update  | grep Image
    Image:         nginx:1.31

[root@k8s-master ~]# kubectl  get  pods
NAME                              READY   STATUS    RESTARTS   AGE
deploy-update-6747747c6f-b9t8x    1/1     Running   0          4m49s
deploy-update-6747747c6f-c9ld5    1/1     Running   0          4m49s
deploy-update-6747747c6f-z4klk    1/1     Running   0          4m49s
```

**다시 최신 버전으로 롤링 업데이트**

```bash
[root@k8s-master ~]# kubectl  rollout  undo  deployment  deploy-update --to-revision=2
deployment.apps/deploy-update rolled back

	# Revision 1 (nginx:1.31)
[root@k8s-master ~]# kubectl  get  pods
NAME                              READY   STATUS    RESTARTS   AGE
deploy-update-6747747c6f-b9t8x    1/1     Running   0          4m49s
deploy-update-6747747c6f-c9ld5    1/1     Running   0          4m49s
deploy-update-6747747c6f-z4klk    1/1     Running   0          4m49s

	# Revision 2 (nginx:1.31.1)
[root@k8s-master ~]# kubectl  get  pods
NAME                              READY   STATUS    RESTARTS   AGE
deploy-update-59c675765b-5snbr    1/1     Running   0          11s
deploy-update-59c675765b-l72sc    1/1     Running   0          11s
deploy-update-59c675765b-p5xlm    1/1     Running   0          11s

[root@k8s-master ~]# kubectl  describe  deployments.apps  deploy-update  | grep Image
    Image:         nginx:1.13.1

	# 히스토리 수정
[root@k8s-master ~]# kubectl annotate deployment deploy-update \
kubernetes.io/change-cause="revision 4 : rollback to revision 2 nginx 1.31.1" \
--overwrite

	# 히스토리 확인
[root@k8s-master ~]# kubectl  rollout  history  deployment  deploy-update
deployment.apps/deploy-update
REVISION  CHANGE-CAUSE
3         revision 3 : rollback to nginx 1.31
4         revision 4 : rollback to revision 2 nginx 1.31.1
```

### revision이 증가하지 않는 변경

- 모든 Deployment 변경이 Revision을 증가시키는 것은 아니다.
- Deployment 자체의 metadata.annotation만 변경하면 Pod Template(spec.template)은 변경되지 않는다.
- Pod Template이 변경되지 않으면 새로운 ReplicaSet이 생성되지 않고 Revision도 증가하지 않는다.
- 이미지, 환경변수, Pod Template의 label/annotation 등이 변경되면 새로운 ReplicaSet이 생성되고 Revision이 증가한다.

```bash
	# 아무 annotation 설정
[root@k8s-master ~]# kubectl  annotate  deployments.apps  deploy-update \
> test-annotation=hello  --overwrite
deployment.apps/deploy-update annotated

[root@k8s-master ~]# kubectl  get  deployments.apps  deploy-update  -o  yaml | grep ann
  annotations:
      {"apiVersion":"apps/v1","kind":"Deployment","metadata":{"annotations":{},"name":"deploy-update","namespace":"default"},"spec":{"replicas":3,"revisionHistoryLimit":5,"selector":{"matchLabels":{"app":"web"}},"strategy":{"rollingUpdate":{"maxSurge":1,"maxUnavailable":1},"type":"RollingUpdate"},"template":{"metadata":{"labels":{"app":"web","track":"stable"}},"spec":{"containers":[{"image":"nginx:1.31","name":"nginx-container","ports":[{"containerPort":80}]}]}}}}
    test-annotation: hello

	# 아무 annotation을 사용하면 Revision이 변경되지 않는다.
[root@k8s-master ~]# kubectl  rollout  history  deployment  deploy-update
deployment.apps/deploy-update
REVISION  CHANGE-CAUSE
3         revision 3 : rollback to nginx 1.31
4         revision 4 : rollback to revision 2 nginx 1.31.1

	# kubernetes.io/change-cause 설정
[root@k8s-master ~]# kubectl  annotate  deployments  deploy-update \
kubernetes.io/change-cause="revision 4 nginx:1.31.1  -->  nginx:1.31.2로 수정" --overwrite
deployment.apps/deploy-update annotated

[root@k8s-master ~]#  kubectl  rollout  history  deployment  deploy-update
deployment.apps/deploy-update
REVISION  CHANGE-CAUSE
3         revision 3 : rollback to nginx 1.31
4         revision 4 nginx:1.31.1  -->  nginx:1.31.2로 수정
```

- 이미지나 환경변수 변경 없이 annotation을 작성하면 새로운 버전이 저장되는 것이 아니라 덮어쓰기된다.
- 즉 새로운 replicaset이 생성되지 않으면 덮어쓰기된다. (새 ReplicaSet 생성 --> revision 증가)

### change-cause를 남겼을 때 vs 안 남겼을 때

- Deployment의 Revision 자체는 Pod Template이 변경되면 자동으로 증가한다.
- 하지만 CHANGE-CAUSE는 자동으로 작성되는 값이 아니다.
- kubernetes.io/change-cause annotation을 별도로 작성하지 않으면 rollout history의 CHANGE-CAUSE 값은 `<none>`으로 표시될 수 있다.
- 따라서 운영 환경에서는 어떤 변경을 했는지 알 수 있도록 change-cause를 남겨두는 것이 좋다.

```bash
	# 롤링 업데이트 (change-cause 없이 업데이트)
[root@k8s-master ~]# kubectl  set  image  deployments  deploy-update  nginx-container=nginx:1.31.3
deployment.apps/deploy-update image updated

[root@k8s-master ~]# kubectl  get  pods
NAME                              READY   STATUS    RESTARTS   AGE
deploy-update-7c85497f86-8fq57    1/1     Running   0          4s
deploy-update-7c85497f86-fwsk9    1/1     Running   0          3s
deploy-update-7c85497f86-vdv7d    1/1     Running   0          4s

[root@k8s-master ~]#  kubectl  rollout  history  deployment  deploy-update
deployment.apps/deploy-update
REVISION  CHANGE-CAUSE
3         revision 3 : rollback to nginx 1.31
4         revision 4 nginx:1.31.1  -->  nginx:1.31.2로 수정
5         revision 4 nginx:1.31.1  -->  nginx:1.31.2로 수정

	# 롤링 업데이트 (change-cause 없이 업데이트)
[root@k8s-master ~]# kubectl  set  image  deployments  deploy-update  nginx-container=nginx:1.30
deployment.apps/deploy-update image updated
```

- 롤링업데이트 후 CHANGE-CAUSE를 설정하지 않으면 관리가 어려워질 수 있다.
- 롤링 업데이트인지 롤백인지 확인 할 수 없다.

```bash
[root@k8s-master ~]#  kubectl  rollout  history  deployment  deploy-update
deployment.apps/deploy-update
REVISION  CHANGE-CAUSE
3         revision 3 : rollback to nginx 1.31
4         revision 4 nginx:1.31.1  -->  nginx:1.31.2로 수정
5         revision 4 nginx:1.31.1  -->  nginx:1.31.2로 수정
6         revision 4 nginx:1.31.1  -->  nginx:1.31.2로 수정

	# 변경 이력(change-cause) 기록
[root@k8s-master ~]# kubectl annotate deployment deploy-update \
kubernetes.io/change-cause="revision 6: nginx:1.31.3  -->  nginx:1.30로 수정" --overwrite

[root@k8s-master ~]#  kubectl  rollout  history  deployment  deploy-update
deployment.apps/deploy-update
REVISION  CHANGE-CAUSE
3         revision 3 : rollback to nginx 1.31
4         revision 4 : nginx:1.31.1  -->  nginx:1.31.2로 수정
5         revision 4 : nginx:1.31.1  -->  nginx:1.31.2로 수정
6         revision 6 : nginx 1.31.3  -->  nginx:1.30로 수정
```

### rollout status가 멈추는 상황 만들기

- 잘못된 이미지로 Rolling Update를 수행하여 Deployment가 정상적으로 완료되지 않는 상태를 확인한다.
- rollout status 명령어가 무엇을 기다리는지 확인한다.
- 새 ReplicaSet은 생성되지만, 새 Pod가 정상적으로 Ready 상태가 되지 않으면 Rolling Update는 완료되지 않는다.

```bash
	# 1) 잘못된 이미지로 업데이트
[root@k8s-master ~]# kubectl  set  image  deployments  deploy-update  nginx-container=nginx:not-exist

	# 2) 롤링 업데이트 상태 확인
[root@k8s-master ~]# kubectl  rollout  status  deployment  deploy-update
Waiting for deployment "deploy-update" rollout to finish: 2 out of 3 new replicas have been updated...

[root@k8s-master ~]# kubectl  get  pods
NAME                              READY   STATUS         RESTARTS   AGE
deploy-update-5b7ddfdd86-cnf26    1/1     Running        0          13m
deploy-update-5b7ddfdd86-mrbzj    1/1     Running        0          13m
deploy-update-c95df6845-7cm6z     0/1     ErrImagePull   0          47s		# image Pull 실패
deploy-update-c95df6845-rxqft     0/1     ErrImagePull   0          47s		# image Pull 실패
```

- 잘못된 이미지로 Rolling Update 할 때 2개의 Pod가 ErrImagePull 상태가 될 수 있는 이유
  - 기존 Pod가 3개 실행 중인 상태에서 Rolling Update를 시작한다.
  - 첫 번째 새 Pod가 생성되고 잘못된 이미지를 Pull하려고 시도한다.
  - 이미지가 존재하지 않으면 첫 번째 새 Pod는 ErrImagePull 또는 ImagePullBackOff 상태가 된다.
  - 이 과정에서 기존 Pod 1개가 Terminating 상태로 들어갈 수 있다.
  - 그러면 Deployment Controller는 원하는 새 버전 Pod 개수를 맞추기 위해 새 Pod를 하나 더 생성할 수 있다.
  - 두 번째 새 Pod 역시 같은 잘못된 이미지를 사용하므로 ErrImagePull 상태가 된다.

**3) 롤백으로 복구**

```bash
# 이전 Revision으로 롤백
[root@k8s-master ~]# kubectl  rollout  undo  deployment  deploy-update
deployment.apps/deploy-update rolled back

# 특정 Revision으로 롤백
[root@k8s-master ~]# kubectl  rollout  undo  deployment  deploy-update  --to-revision=4
deployment.apps/deploy-update rolled back
```

```bash
	# 4) rollout 상태 확인
[root@k8s-master ~]# kubectl  rollout  status  deployment  deploy-update
Waiting for deployment "deploy-update" rollout to finish: 2 out of 3 new replicas have been updated...
Waiting for deployment spec update to be observed...
Waiting for deployment spec update to be observed...
Waiting for deployment "deploy-update" rollout to finish: 0 out of 3 new replicas have been updated...
Waiting for deployment "deploy-update" rollout to finish: 0 out of 3 new replicas have been updated...
Waiting for deployment "deploy-update" rollout to finish: 2 out of 3 new replicas have been updated...
Waiting for deployment "deploy-update" rollout to finish: 2 out of 3 new replicas have been updated...
Waiting for deployment "deploy-update" rollout to finish: 2 out of 3 new replicas have been updated...
Waiting for deployment "deploy-update" rollout to finish: 2 out of 3 new replicas have been updated...
Waiting for deployment "deploy-update" rollout to finish: 2 out of 3 new replicas have been updated...
Waiting for deployment "deploy-update" rollout to finish: 2 out of 3 new replicas have been updated...
Waiting for deployment "deploy-update" rollout to finish: 1 old replicas are pending termination...
Waiting for deployment "deploy-update" rollout to finish: 1 old replicas are pending termination...
Waiting for deployment "deploy-update" rollout to finish: 1 old replicas are pending termination...
Waiting for deployment "deploy-update" rollout to finish: 1 old replicas are pending termination...
deployment "deploy-update" successfully rolled out

[root@k8s-master ~]# kubectl  get  pods
NAME                              READY   STATUS    RESTARTS   AGE
deploy-update-59c675765b-fn7bl    1/1     Running   0          14s
deploy-update-59c675765b-mmnrv    1/1     Running   0          15s
deploy-update-59c675765b-wkt9k    1/1     Running   0          15s

[root@k8s-master ~]# kubectl  describe  deployments  deploy-update | grep Image
    Image:         nginx:1.13.1

[root@k8s-master ~]# cat  deploy-update-step2.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deploy-update
spec:
  replicas: 3
  revisionHistoryLimit: 5	# ReplicaSet 5개까지 기록
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
~~~~~~ 중간 생략 ~~~~~~

[root@k8s-master ~]# kubectl  get  rs
NAME                       DESIRED   CURRENT   READY   AGE
deploy-update-59c675765b   3         3         3       155m
deploy-update-5b7ddfdd86   0         0         0       96m
deploy-update-6747747c6f   0         0         0       163m
deploy-update-7c85497f86   0         0         0       101m
deploy-update-c95df6845    0         0         0       83m

[root@k8s-master ~]# kubectl  get  pods
NAME                              READY   STATUS    RESTARTS   AGE
deploy-update-59c675765b-fn7bl    1/1     Running   0          80m
deploy-update-59c675765b-mmnrv    1/1     Running   0          80m
deploy-update-59c675765b-wkt9k    1/1     Running   0          80m
```

- 오래된 ReplicaSet에 파드가 생성되어있다.
- 잘못된 이미지명으로 롤링업데이트 중 롤백했기 때문에 오래된 ReplicaSet에 파드가 생성되어있다.

### rollout pause / resume

- rollout pause와 rollout resume은 Deployment의 Rolling Update를 일시 중지했다가 다시 진행할 때 사용하는 명령어
- 일반적인 Rolling Update는 Deployment의 이미지나 환경변수 등 Pod Template이 변경되면 즉시 새로운 ReplicaSet을 생성하고 새로운 Pod를 배포한다.

Deployment 설정 변경 (Pod Template)
- 새 ReplicaSet 생성 --> 새 Pod 생성 --> 새 Pod Ready 확인 --> 기존 Pod 제거 --> Rolling Update 완료

**rollout pause**

- 형식 : `kubectl rollout pause deployment <deployment_name>`
- Deployment의 Rolling Update를 일시 중지한다.
- 중요한 점은 pause가 현재 실행 중인 Pod를 정지시키는 명령어가 아니다.
  - 기존 Pod 실행 O
  - 서비스 계속 O
  - 새 ReplicaSet 생성 X
  - 새 Pod 생성 X
  - 기존 Pod 교체 X
- pause 상태에서도 Deployment의 설정은 변경할 수 있다.
- 이미지 설정 자체는 변경되지만, pause 상태이므로 새로운 ReplicaSet을 생성하거나 Pod를 교체하지 않는다.
  - pause --> 이미지 변경 --> Deployment 설정 변경 O --> 실제 Rolling Update X

**rollout resume**

- 형식 : `kubectl rollout resume deployment <deployment_name>`
- pause 상태를 해제하고 중지되어 있던 Rolling Update를 다시 진행한다.
  - resume --> pause 중 변경했던 Pod Template 확인 --> 새 ReplicaSet 생성 --> 새 Pod 생성 --> 새 Pod Ready 확인 --> 기존 Pod 제거 --> Rolling Update 완료

**일반 Rolling Update와 pause/resume의 차이**

- 일반 Rolling Update
  - 이미지 변경 --> 즉시 Rolling Update 시작 --> 새 ReplicaSet 생성 --> 새 Pod 생성 --> 기존 Pod 교체
- pause / resume 사용
  - pause --> 이미지, 환경변수 등 변경 --> 실제 Pod 교체는 아직 하지 않음 --> resume --> 변경된 Rolling Update 시작

**pause / resume은 언제 사용하는가?**

- 여러 설정을 한꺼번에 변경한 뒤 한 번의 Rolling Update로 적용하고 싶을 때 사용할 수 있다.
- 예를 들어 다음 세 가지를 변경한다고 가정한다.
  1. nginx 이미지 변경
  2. 환경변수 변경
  3. CPU/Memory 설정 변경
- pause 없이 각각 수정하면 변경할 때마다 rollout이 발생할 수 있다.
- pause --> 이미지, 환경변수, 리소스 설정 변경 --> 설정 확인 --> resume --> 한 번의 Rolling Update
- 운영 중 변경 사항을 먼저 작성한 뒤 검토가 끝난 후 실제 배포하고 싶을 때도 유용하다.

**5-1) 현재 Deployment와 Pod 확인**

```bash
[root@k8s-master ~]# kubectl  get  deployments  deploy-update
NAME            READY   UP-TO-DATE   AVAILABLE   AGE
deploy-update   3/3     3            3           175m

[root@k8s-master ~]# kubectl  get  pods
NAME                              READY   STATUS    RESTARTS   AGE
deploy-update-59c675765b-fn7bl    1/1     Running   0          80m
deploy-update-59c675765b-mmnrv    1/1     Running   0          80m
deploy-update-59c675765b-wkt9k    1/1     Running   0          80m
```

**5-2) Deployment rollout 일시 중지**

```bash
[root@k8s-master ~]# kubectl  rollout  pause  deployment  deploy-update
deployment.apps/deploy-update paused
```

**5-3) pause 상태에서 이미지 변경**

```bash
[root@k8s-master ~]# kubectl  describe  deployments  deploy-update | grep Image
    Image:         nginx:1.13.1

[root@k8s-master ~]# kubectl  get  pods  --watch
NAME                              READY   STATUS    RESTARTS   AGE
deploy-update-59c675765b-fn7bl    1/1     Running   0          91m
deploy-update-59c675765b-mmnrv    1/1     Running   0          92m
deploy-update-59c675765b-wkt9k    1/1     Running   0          92m

[root@k8s-master ~]# kubectl  set  image  deployments  deploy-update  nginx-container=nginx:1.31.3
deployment.apps/deploy-update image updated

# 롤링업데이트되지 않는다.
[root@k8s-master ~]# kubectl  get  pods  --watch
NAME                              READY   STATUS    RESTARTS   AGE
deploy-update-59c675765b-fn7bl    1/1     Running   0          91m
deploy-update-59c675765b-mmnrv    1/1     Running   0          92m
deploy-update-59c675765b-wkt9k    1/1     Running   0          92m
```

**5-4) Deployment에 설정된 이미지 확인**

```bash
[root@k8s-master ~]# kubectl  describe  deployments  deploy-update  | grep Image
    Image:         nginx:1.31.3
```

- 현재 이미지가 nginx:1.31.1에서 nginx:1.31.3으로 변경되어있다. 하지만 롤링 업데이트는 발생하지 않는다.

**5-5) 현재 Pod 및 ReplicaSet 확인**

```bash
[root@k8s-master ~]# kubectl  get  pods
NAME                              READY   STATUS    RESTARTS   AGE
deploy-update-59c675765b-fn7bl    1/1     Running   0          91m
deploy-update-59c675765b-mmnrv    1/1     Running   0          92m
deploy-update-59c675765b-wkt9k    1/1     Running   0          92m

[root@k8s-master ~]# kubectl  describe  pods  deploy-update-59c675765b-fn7bl | grep  Image
Image
    Image:      nginx:1.13.1
    Image ID:   docker.io/library/nginx@sha256:72c7191585e9b79cde433c89955547685db00f3a8595a750339549f6acef7702

[root@k8s-master ~]# kubectl  get  rs
NAME                       DESIRED   CURRENT   READY   AGE
deploy-update-59c675765b   3         3         3       174m
deploy-update-5b7ddfdd86   0         0         0       114m
deploy-update-6747747c6f   0         0         0       3h2m
deploy-update-7c85497f86   0         0         0       120m
deploy-update-c95df6845    0         0         0       102m
```

**5-6) rollout 재개**

```bash
[root@k8s-master ~]# kubectl  rollout  resume  deployment  deploy-update
deployment.apps/deploy-update resumed

[root@k8s-master ~]# kubectl  get  pods  --watch
NAME                              READY   STATUS              RESTARTS   AGE
deploy-update-59c675765b-fn7bl    1/1     Running             0          97m
deploy-update-59c675765b-mmnrv    1/1     Running             0          97m
deploy-update-59c675765b-wkt9k    1/1     Running             0          97m
deploy-update-59c675765b-mmnrv    1/1     Terminating         0          98m
deploy-update-7c85497f86-svlqn    0/1     Pending             0          0s
deploy-update-7c85497f86-svlqn    0/1     Pending             0          0s
deploy-update-59c675765b-mmnrv    1/1     Terminating         0          98m
deploy-update-7c85497f86-svlqn    0/1     ContainerCreating   0          0s
deploy-update-7c85497f86-hhqvh    0/1     Pending             0          0s
deploy-update-7c85497f86-hhqvh    0/1     Pending             0          0s
deploy-update-7c85497f86-hhqvh    0/1     ContainerCreating   0          0s
deploy-update-59c675765b-mmnrv    0/1     Completed           0          98m
deploy-update-7c85497f86-svlqn    1/1     Running             0          0s
deploy-update-59c675765b-wkt9k    1/1     Terminating         0          98m
deploy-update-7c85497f86-sncvt    0/1     Pending             0          0s
deploy-update-59c675765b-wkt9k    1/1     Terminating         0          98m
deploy-update-7c85497f86-sncvt    0/1     Pending             0          0s
deploy-update-7c85497f86-sncvt    0/1     ContainerCreating   0          0s
deploy-update-59c675765b-wkt9k    0/1     Completed           0          98m
deploy-update-7c85497f86-hhqvh    1/1     Running             0          0s
deploy-update-59c675765b-mmnrv    0/1     Completed           0          98m
deploy-update-59c675765b-mmnrv    0/1     Completed           0          98m
deploy-update-59c675765b-fn7bl    1/1     Terminating         0          98m
deploy-update-59c675765b-fn7bl    1/1     Terminating         0          98m
deploy-update-59c675765b-fn7bl    0/1     Completed           0          98m
deploy-update-59c675765b-wkt9k    0/1     Completed           0          98m
deploy-update-59c675765b-wkt9k    0/1     Completed           0          98m
deploy-update-7c85497f86-sncvt    1/1     Running             0          1s
deploy-update-59c675765b-fn7bl    0/1     Completed           0          98m
deploy-update-59c675765b-fn7bl    0/1     Completed           0          98m
```

**4-7) ReplicaSet 다시 확인**

```bash
[root@k8s-master ~]# kubectl  get  rs
NAME                       DESIRED   CURRENT   READY   AGE
deploy-update-59c675765b   0         0         0       179m
deploy-update-5b7ddfdd86   0         0         0       120m
deploy-update-6747747c6f   0         0         0       3h7m
deploy-update-7c85497f86   3         3         3       125m
deploy-update-c95df6845    0         0         0       108m
```

- 예전 어느 시점
  - nginx:1.31.3 설정으로 ReplicaSet 생성
  - deploy-update-7c85497f86 생성
  - AGE는 그때부터 계속 누적
- 그 이후 다른 이미지로 업데이트
  - 7c85497f86은 replicas 0으로 내려감
  - ReplicaSet 자체는 revisionHistoryLimit 때문에 남아 있음
- 이번에 pause 상태에서 다시 nginx:1.31.3 설정
  - resume
  - Deployment Controller가 확인
  - 이 Pod Template과 동일한 기존 ReplicaSet이 있네? 판단
  - 새 ReplicaSet 생성 안 함
  - 기존 deploy-update-7c85497f86을 다시 scale up
  - replicas 3

```bash
	# deployment 삭제
[root@k8s-master ~]# kubectl  delete  deployments.apps  deploy-update
deployment.apps "deploy-update" deleted from default namespace
```

### EX) maxSurge/maxUnavailable 8개 Pod 실습

EX) Deployment의 Pod를 8개로 실행하고, Rolling Update 시 maxSurge: 2, maxUnavailable: 2를 설정하여 새 버전 Pod가 최대 2개 추가되고 기존 Pod도 최대 2개까지 사용할 수 없는 상태를 확인

```bash
[root@k8s-master ~]# vi deploy-update-step2.yaml
```
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deploy-update
spec:
  replicas: 6		# 3에서 6로 수정
  revisionHistoryLimit: 5
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 2		# 2로 수정
      maxUnavailable: 2	# 2로 수정
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
        track: stable
    spec:
      containers:
      - name: nginx-container
        image: nginx:1.31
        ports:
        - containerPort: 80
```

- maxSurge는 업데이트 중 원래 replicas 수보다 몇 개까지 Pod를 더 만들 수 있는가를 의미
- 예를 들어: replicas: 3, maxSurge: 1
  - 기본 Pod 수 = 3개
  - 추가 생성 가능 = 1개
  - 최대 Pod 수 = 4개
  - 즉 새 버전 Pod를 먼저 하나 더 만들어 놓을 수 있다.

- maxUnavailable은 업데이트 중 원하는 replicas 수에서 몇 개까지 서비스 불가 상태를 허용할 것인가를 의미
- 예를 들어: replicas: 3, maxUnavailable: 1
  - 원하는 Pod = 3개
  - Unavailable 허용 = 1개
  - 최소 Available Pod = 2개
  - 즉 업데이트 도중 최소 2개는 정상 서비스 가능한 상태로 유지해야 한다.

```bash
[root@k8s-master ~]# kubectl  apply  -f  deploy-update-step2.yaml  --dry-run=client
deployment.apps/deploy-update configured (dry run)

[root@k8s-master ~]# kubectl  apply  -f  deploy-update-step2.yaml
deployment.apps/deploy-update created

[root@k8s-master ~]# kubectl  get  pods  --watch
NAME                              READY   STATUS    RESTARTS   AGE
deploy-update-6747747c6f-f9mrz    1/1     Running   0          9s
deploy-update-6747747c6f-k9rcc    1/1     Running   0          9s
deploy-update-6747747c6f-ksm6s    1/1     Running   0          9s
deploy-update-6747747c6f-nbvhc    1/1     Running   0          9s
deploy-update-6747747c6f-q2sdl    1/1     Running   0          9s
deploy-update-6747747c6f-rzd7d    1/1     Running   0          9s
deploy-update-6747747c6f-t9nqj    1/1     Running   0          9s
deploy-update-6747747c6f-whx2k    1/1     Running   0          9s

[root@k8s-master ~]# kubectl  annotate  deployments  deploy-update \
> kubernetes.io/change-cause="Revistion 1 2026-08-19 : nginx:1.31 버전 최초 배포" --overwrite
deployment.apps/deploy-update annotated

[root@k8s-master ~]# kubectl  rollout  history  deployment  deploy-update
deployment.apps/deploy-update
REVISION  CHANGE-CAUSE
1         Revistion 1 2026-08-19 : nginx:1.31 버전 최초 배포

[root@k8s-master ~]# kubectl  set  image  deployments  deploy-update   nginx-container=nginx:1.31.1

[root@k8s-master ~]# kubectl  get  pods  --watch
NAME                              READY   STATUS    RESTARTS   AGE
deploy-update-6747747c6f-f9mrz    1/1     Running   0          9s
deploy-update-6747747c6f-k9rcc    1/1     Running   0          9s
deploy-update-6747747c6f-ksm6s    1/1     Running   0          9s
deploy-update-6747747c6f-nbvhc    1/1     Running   0          9s
deploy-update-6747747c6f-q2sdl    1/1     Running   0          9s
deploy-update-6747747c6f-rzd7d    1/1     Running   0          9s
deploy-update-6747747c6f-t9nqj    1/1     Running   0          9s
deploy-update-6747747c6f-whx2k    1/1     Running   0          9s

deploy-update-6cbccd8b88-rhx9k    0/1     Pending   0          0s
deploy-update-6cbccd8b88-2ncl6    0/1     Pending   0          0s

deploy-update-6cbccd8b88-rhx9k    0/1     Pending   0          0s
deploy-update-6cbccd8b88-2ncl6    0/1     Pending   0          0s

deploy-update-6747747c6f-q2sdl    1/1     Terminating   0          7m15s
deploy-update-6747747c6f-f9mrz    1/1     Terminating   0          7m15s

deploy-update-6cbccd8b88-rhx9k    0/1     ContainerCreating   0          0s
deploy-update-6cbccd8b88-2ncl6    0/1     ContainerCreating   0          0s
~~~~~~~~~~~~~~~~~~ 중간 생략 ~~~~~~~~~~~~~~~~~~

[root@k8s-master ~]# kubectl  annotate  deployments  deploy-update \
 kubernetes.io/change-cause="Revistion 2 2026-08-19 : nginx:1.31 버전  --> nginx:1.31.1 버전  업데이트" --overwrite
deployment.apps/deploy-update annotated

[root@k8s-master ~]# kubectl  rollout  history  deployment  deploy-update
deployment.apps/deploy-update
REVISION  CHANGE-CAUSE
1         Revistion 1 2026-08-19 : nginx:1.31 버전 최초 배포
2         Revistion 2 2026-08-19 : nginx:1.31 버전  --> nginx:1.31.1 버전  업데이트

[root@k8s-master ~]# kubectl  rollout  history  deployment  deploy-update  --revision=2
deployment.apps/deploy-update with revision #2
Pod Template:
  Labels:       app=web
        pod-template-hash=6cbccd8b88
        track=stable
  Annotations:  kubernetes.io/change-cause: Revistion 2 2026-08-19 : nginx:1.31 버전  --> nginx:1.31.1 버전  업데이트
  Containers:
   nginx-container:
    Image:      nginx:1.31.1
    Port:       80/TCP
    Host Port:  0/TCP
    Environment: <none>
    Mounts:     <none>
  Volumes:      <none>
  Node-Selectors: <none>
  Tolerations:  <none>

[root@k8s-master ~]# kubectl  get  rs
NAME                       DESIRED   CURRENT   READY   AGE
deploy-update-6747747c6f   0         0         0       12m
deploy-update-6cbccd8b88   8         8         8       5m35s

[root@k8s-master ~]# kubectl  delete  deployments  deploy-update
deployment.apps "deploy-update" deleted from default namespace
```

정리: `rollout undo`, `rollout pause/resume`, `change-cause` annotation을 조합하면 Rolling Update의 이력 관리와 안전한 되돌리기가 가능하다. 다음은 서비스용 Pod 개수를 관리하는 Deployment와 달리, 노드마다 Pod를 하나씩 배치하는 **DaemonSet**이다.

---

## DaemonSet

- **DaemonSet**은 클러스터의 각 Node마다 특정 Pod를 1개씩(또는 조건에 맞는 노드마다) 자동으로 실행해 주는 컨트롤러
- 즉, Deployment가 서비스용 Pod를 n개 유지라면, DaemonSet은 노드마다 1개씩 유지한다.

### DaemonSet이 필요한 이유

- 모든 노드에서 반드시 돌아야 하는 공통 기능 Pod를 자동 배치/유지한다.

예)
- 로그 수집 에이전트 (각 노드의 로그를 모아서 중앙으로 전송)
- 모니터링 에이전트 (노드 CPU/메모리/디스크/네트워크 수집)
- 네트워크 플러그인 구성 요소 (CNI 관련)
- 노드 보안/감사 에이전트
- 스토리지/CSI 노드 에이전트(환경에 따라)

- DaemonSet은 특정 서비스 트래픽을 처리하는 앱이 아니라, 노드 자체에서 데이터를 가져오거나 시스템 기능을 제공해야 하므로 노드마다 1개씩 있어야 한다.

### DaemonSet의 동작 원리

DaemonSet을 생성하면, 컨트롤러가 다음을 자동으로 한다.
1) 현재 클러스터의 노드 목록을 확인한다.
2) 각 노드마다 DaemonSet용 Pod가 있는지 확인한다.
3) 없으면 그 노드에 DaemonSet용 Pod를 만든다.
4) DaemonSet용 Pod가 죽으면 그 노드에서 다시 만든다.
5) 노드가 새로 추가되면 그 노드에도 자동으로 DaemonSet용 Pod를 만든다.
6) 노드가 삭제되면 그 노드의 DaemonSet용 Pod도 함께 사라진다.

결론
- 노드 수가 늘면 DaemonSet용 Pod도 같이 늘고 노드 수가 줄면 DaemonSet용 Pod도 같이 줄어든다
- 예: 워커 노드가 2개면 DaemonSet용 Pod 2개, 5개면 DaemonSet용 Pod 5개 (조건에 맞는 노드만)

### Deployment와 DaemonSet 차이

- Deployment
  - 목적 : 서비스 트래픽 처리용 Pod를 원하는 개수만큼 유지
  - 스케줄링 : 어떤 노드에 올라갈지는 쿠버네티스가 분산 배치
  - replicas : 사용자가 개수를 지정한다 (예: 3개, 10개)

- DaemonSet
  - 목적 : 노드마다 반드시 필요한 공통 Pod를 유지
  - 스케줄링 : 각 노드에 1개씩이 원칙
  - replicas : 사용자가 숫자를 지정하는 개념이 아니다. (노드 수/조건에 의해 자동 결정)

- Deployment = 서비스 개수 보장
- DaemonSet = 노드당 1개 보장

### 모든 노드가 아니라 특정 노드만 돌리고 싶을 때

- DaemonSet은 기본이 모든 노드(정확히는 스케줄 가능한 노드)이지만, 필요하면 조건을 걸어서 일부 노드에만 배치할 수 있다.

대표 방법
- nodeSelector : 라벨이 붙은 노드에만 배치
- nodeAffinity : 더 복잡한 조건(OR/AND 등)으로 노드 선택
- taints/tolerations : 특정 노드(예: control-plane)에 원래는 안 올라가게 막혀있는데 toleration을 넣으면 올라가게 할 수 있다

예시 상황
- GPU 노드에만 에이전트 설치
- 특정 역할(role=log-node) 라벨이 있는 노드에만 로그 수집기 실행
- control-plane 노드에도 반드시 실행해야 하는 시스템 에이전트 배치

### DaemonSet 업데이트(롤링업데이트)

- DaemonSet도 이미지 버전을 바꾸면 Pod가 노드별로 순차 교체될 수 있다.
- 즉, 각 노드에 있는 Pod를 새 버전으로 하나씩 바꿔치기한다.
- 다만 DaemonSet은 노드당 1개가 중요해서, 업데이트 중에도 각 노드에서 에이전트가 완전히 사라지는 시간을 최소화하려고 전략을 사용한다.
- 설정에 따라 동시에 바꾸는 노드 수를 제한할 수 있다(업데이트 전략)
- Deployment는 replicas 기반으로 바꾸고, DaemonSet은 노드 단위로 바뀐다.

### DaemonSet에서 자주 보게 되는 특징/필드들

1) selector / template : 어떤 Pod가 DaemonSet 소속인지 구분하기 위한 라벨 매칭. template은 실제로 노드에 생성될 Pod 모양(컨테이너/이미지/환경변수 등)
2) nodeSelector / affinity : 어느 노드에 깔지를 결정
3) tolerations : 특정 taint가 있는 노드에도 배치할지 결정. 대표적으로 control-plane 노드(마스터 노드)에 올릴 때 필요할 수 있다
4) hostPath (에이전트에서 자주 사용) : 노드의 파일 시스템을 Pod에 마운트해서 로그/메트릭 파일 등을 읽음. 예: /var/log 를 마운트해서 노드 로그 수집

### 언제 DaemonSet을 쓰면 안 되나?

- 서비스 애플리케이션(웹 서버, API 서버)을 각 노드에 1개씩 깔 필요가 없다면 Deployment/StatefulSet이 보통 맞다.

예)
- 쇼핑몰 웹 서버: DaemonSet로 깔면 노드마다 1개씩 강제라서 운영이 불편해질 수 있음
- 트래픽에 따라 3개에서 10개로 늘리고 싶은 서비스: Deployment가 맞다
- DaemonSet은 노드 공통 기능에 맞는 도구다.

DaemonSet YAML 예제

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: daemonset-nginx

spec:
  replicas: 3		# ReplicaSet에서 replicas를 제외하면 형식은 동일하다. (DaemonSet은 기본 1개로 설정)
  selector:
    matchLabels:
      app: web

  template:
    metadata:
      name: nginx-pod
      labels:
        app: web
    spec:
      containers:
      - name: nginx-container
        image: nginx:1.31
```

### DaemonSet 관련 명령어 정리

```bash
	# DaemonSet 생성 및 확인
# kubectl create -f daemonset-exam.yaml
# kubectl get daemonset
# kubectl get pods

	# DaemonSet으로 동작 중인 Pod를 삭제
# kubectl delete pod daemonset-nginx-XXX

	# DaemonSet Rolling Update (컨테이너 버전 수정)
# kubectl edit ds daemonset-nginx

	# Rolling Back (롤백)
# kubectl rollout undo daemonset daemonset-nginx

	# DaemonSet 삭제
# kubectl delete daemonsets.apps daemonset-nginx

	또는 (축약형)
# kubectl delete ds daemonset-nginx
```

### DaemonSet 실습 — 생성/롤링업데이트/롤백

```bash
[root@k8s-master ~]# vi daemonset-nginx.yaml
```
```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: daemonset-nginx
spec:
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      name: nginx-pod
      labels:
        app: web
    spec:
      containers:
      - name: nginx-container
        image: nginx:1.31
```

```bash
[root@k8s-master ~]# kubectl  create  -f  daemonset-nginx.yaml  --dry-run=client
daemonset.apps/daemonset-nginx created (dry run)

[root@k8s-master ~]# kubectl  create  -f  daemonset-nginx.yaml
daemonset.apps/daemonset-nginx created

[root@k8s-master ~]# kubectl  get  pods  -o  wide
NAME                    READY   STATUS    RESTARTS   AGE   IP            NODE          NOMINATED NODE   READINESS GATES
daemonset-nginx-k4bqk   1/1     Running   0          12s   10.244.1.54   k8s-worker1   <none>            <none>
daemonset-nginx-xc4mg   1/1     Running   0          12s   10.244.2.37   k8s-worker2   <none>            <none>
```

- 이 상태에서 워커노드 1대가 추가되면 해당 워커 노드에도 해당 pod가 자동으로 생성된다.

```bash
	# 터미널 1
[root@k8s-master ~]# kubectl  get  pods  -o  wide  --watch

	# 터미널 2
[root@k8s-master ~]# kubectl  edit  daemonsets  daemonset-nginx
# Please edit the object below. Lines beginning with a '#' will be ignored,
# and an empty file will abort the edit. If an error occurs while saving this file will be
# reopened with the relevant failures.
#
apiVersion: apps/v1
kind: DaemonSet
metadata:
  annotations:
    deprecated.daemonset.template.generation: "1"
  creationTimestamp: "2026-08-19T07:01:16Z"
  generation: 1
  name: daemonset-nginx
  namespace: default
  resourceVersion: "238778"
  uid: 87489b5f-b630-4780-9f27-101ad790acc6

spec:
  revisionHistoryLimit: 10		# revision을 10개까지 보관
  selector:
    matchLabels:
      app: web			# app: web 라벨을 갖은 pod를 관리 대상으로 인식
  template:
    metadata:
      labels:
        app: web			# 해당 pod 생성시 app: web 라벨을 사용
      name: nginx-pod
    spec:
      containers:
      - image: nginx:1.31		# 해당 pod가 사용할 이미지 (image: nginx:1.31  -->  image: nginx:1.31.1)
        imagePullPolicy: IfNotPresent
        name: nginx-container
        resources: {}

~~~~~~~~~~~~~ 중간 생략 ~~~~~~~~~~~~~

[root@k8s-master ~]# kubectl  get  pods  -o  wide  --watch
NAME                    READY   STATUS              RESTARTS   AGE     IP            NODE          NOMINATED NODE   READINESS GATES
daemonset-nginx-k4bqk   1/1     Running             0          2m53s   10.244.1.54   k8s-worker1   <none>            <none>
daemonset-nginx-xc4mg   1/1     Running             0          2m53s   10.244.2.37   k8s-worker2   <none>            <none>

daemonset-nginx-xc4mg   1/1     Terminating         0          9m31s   10.244.2.37   k8s-worker2   <none>            <none>
daemonset-nginx-xc4mg   1/1     Terminating         0          9m31s   10.244.2.37   k8s-worker2   <none>            <none>
daemonset-nginx-xc4mg   0/1     Completed           0          9m31s   10.244.2.37   k8s-worker2   <none>            <none>
daemonset-nginx-4zkth   0/1     Pending             0          0s      <none>        <none>        <none>            <none>
daemonset-nginx-4zkth   0/1     Pending             0          0s      <none>        k8s-worker2   <none>            <none>
daemonset-nginx-4zkth   0/1     ContainerCreating   0          0s      <none>        k8s-worker2   <none>            <none>
daemonset-nginx-xc4mg   0/1     Completed           0          9m32s   10.244.2.37   k8s-worker2   <none>            <none>
daemonset-nginx-xc4mg   0/1     Completed           0          9m32s   10.244.2.37   k8s-worker2   <none>            <none>
daemonset-nginx-4zkth   1/1     Running             0          2s      10.244.2.38   k8s-worker2   <none>            <none>
daemonset-nginx-k4bqk   1/1     Terminating         0          9m33s   10.244.1.54   k8s-worker1   <none>            <none>
daemonset-nginx-k4bqk   1/1     Terminating         0          9m33s   10.244.1.54   k8s-worker1   <none>            <none>
daemonset-nginx-k4bqk   0/1     Completed           0          9m33s   10.244.1.54   k8s-worker1   <none>            <none>
daemonset-nginx-qsft7   0/1     Pending             0          0s      <none>        <none>        <none>            <none>
daemonset-nginx-qsft7   0/1     Pending             0          0s      <none>        k8s-worker1   <none>            <none>
daemonset-nginx-qsft7   0/1     ContainerCreating   0          0s      <none>        k8s-worker1   <none>            <none>
daemonset-nginx-k4bqk   0/1     Completed           0          9m33s   10.244.1.54   k8s-worker1   <none>            <none>
daemonset-nginx-k4bqk   0/1     Completed           0          9m33s   10.244.1.54   k8s-worker1   <none>            <none>
daemonset-nginx-qsft7   1/1     Running             0          1s      10.244.1.55   k8s-worker1   <none>            <none>

[root@k8s-master ~]# kubectl  describe  daemonsets   daemonset-nginx | grep Image
    Image:         nginx:1.31.1

[root@k8s-master ~]# kubectl  rollout  history  daemonset daemonset-nginx
daemonset.apps/daemonset-nginx
REVISION  CHANGE-CAUSE
1         <none>
2         <none>

kubectl annotate daemonset daemonset-nginx \
kubernetes.io/change-cause="nginx 1.31 -> 1.31.1 업데이트" \
--overwrite

# CHANGE-CAUSE 기록이 확인되지 않는다.
[root@k8s-master ~]# kubectl  rollout  history  daemonset daemonset-nginx
daemonset.apps/daemonset-nginx
REVISION  CHANGE-CAUSE
1         <none>
2         <none>
```

- DaemonSet은 Deployment처럼 ReplicaSet으로 이력을 관리하지 않고, ControllerRevision을 사용하여 Revision 이력을 관리한다.
- ControllerRevision은 DaemonSet의 이전 Pod Template 정보를 저장하는 이력용 리소스이다.

```bash
	# Daemonset 삭제
[root@k8s-master ~]# kubectl  delete  daemonsets  daemonset-nginx
daemonset.apps "daemonset-nginx" deleted from default namespace

	# Daemonset 생성
[root@k8s-master ~]# kubectl  apply  -f  daemonset-nginx.yaml

	# change-cause 설정
[root@k8s-master ~]# kubectl annotate daemonset daemonset-nginx \
kubernetes.io/change-cause="nginx 1.31 --> 1.31.1 업데이트" \
--overwrite
daemonset.apps/daemonset-nginx annotated

	# 터미널 2
[root@k8s-master ~]# kubectl  edit  daemonsets  daemonset-nginx
~~~~~~~~~~~~~ (동일한 편집: image nginx:1.31 --> nginx:1.31.1) ~~~~~~~~~~~~~

[root@k8s-master ~]# kubectl  rollout  history  daemonset daemonset-nginx
daemonset.apps/daemonset-nginx
REVISION  CHANGE-CAUSE
1         <none>
2         nginx 1.31 --> 1.31.1 업데이트
```

- 첫번째 REVISION은 daemonset이 없는 상태에서 daemonset 버전을 설정할 수 없기때문에 버전관리를 할수 없다.

```bash
	# Daemonset 삭제
[root@k8s-master ~]# kubectl  delete  daemonsets  daemonset-nginx
daemonset.apps "daemonset-nginx" deleted from default namespace

[root@k8s-master ~]# vi  daemonset-nginx.yaml
```
```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: daemonset-nginx
  annotations:					# 추가 설정
    kubernetes.io/change-cause: "nginx 1.31 첫 배포"	# 추가 설정 (yaml 파일에 버전 관련 설정을 미리 설정)
spec:
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      name: nginx-pod
      labels:
        app: web
    spec:
      containers:
      - name: nginx-container
        image: nginx:1.31
```

```bash
	# Daemonset 생성
[root@k8s-master ~]# kubectl  apply  -f  daemonset-nginx.yaml

	# 버전 관리 확인
[root@k8s-master ~]# kubectl  rollout  history  daemonset daemonset-nginx
daemonset.apps/daemonset-nginx
REVISION  CHANGE-CAUSE
1         nginx 1.31 첫 배포

	# 버전 관리 설정
[root@k8s-master ~]# kubectl annotate daemonset daemonset-nginx \
kubernetes.io/change-cause="nginx 1.31 --> 1.31.1 업데이트" --overwrite
daemonset.apps/daemonset-nginx annotated

	# 터미널 2
[root@k8s-master ~]# kubectl  edit  daemonsets  daemonset-nginx
~~~~~~~~~~~~~ (image nginx:1.31 --> nginx:1.31.1) ~~~~~~~~~~~~~

	# 버전 관리 확인
[root@k8s-master ~]# kubectl  rollout  history  daemonset daemonset-nginx
daemonset.apps/daemonset-nginx
REVISION  CHANGE-CAUSE
1         nginx 1.31 첫 배포
2         nginx 1.31 --> 1.31.1 업데이트

	# 현재 사용하는 이미지 확인
[root@k8s-master ~]# kubectl  describe  daemonsets  daemonset-nginx  | grep Image
    Image:         nginx:1.31.1
```

### DaemonSet Rollback 확인

```bash
	# 터미널 1
[root@k8s-master ~]# kubectl  get  pods  -o  wide  --watch
NAME                    READY   STATUS    RESTARTS   AGE     IP            NODE          NOMINATED NODE   READINESS GATES
daemonset-nginx-79mdq   1/1     Running   0          9m36s   10.244.2.45   k8s-worker2   <none>            <none>
daemonset-nginx-7z8sj   1/1     Running   0          9m38s   10.244.1.62   k8s-worker1   <none>            <none>

	# 터미널 2  (이전 버전의 이미지로 롤백)
[root@k8s-master ~]# kubectl  rollout  undo  daemonset  daemonset-nginx

	# 터미널 1
[root@k8s-master ~]# kubectl  get  pods  -o  wide  --watch
NAME                    READY   STATUS              RESTARTS   AGE     IP            NODE          NOMINATED NODE   READINESS GATES
daemonset-nginx-79mdq   1/1     Running             0          9m36s   10.244.2.45   k8s-worker2   <none>            <none>
daemonset-nginx-7z8sj   1/1     Running             0          9m38s   10.244.1.62   k8s-worker1   <none>            <none>
daemonset-nginx-79mdq   1/1     Terminating         0          11m     10.244.2.45   k8s-worker2   <none>            <none>
daemonset-nginx-79mdq   1/1     Terminating         0          11m     10.244.2.45   k8s-worker2   <none>            <none>
daemonset-nginx-79mdq   0/1     Completed           0          11m     10.244.2.45   k8s-worker2   <none>            <none>
daemonset-nginx-mhfld   0/1     Pending             0          0s      <none>        <none>        <none>            <none>
daemonset-nginx-mhfld   0/1     Pending             0          0s      <none>        k8s-worker2   <none>            <none>
daemonset-nginx-mhfld   0/1     ContainerCreating   0          0s      <none>        k8s-worker2   <none>            <none>
daemonset-nginx-mhfld   1/1     Running             0          0s      10.244.2.46   k8s-worker2   <none>            <none>
daemonset-nginx-7z8sj   1/1     Terminating         0          11m     10.244.1.62   k8s-worker1   <none>            <none>
daemonset-nginx-7z8sj   1/1     Terminating         0          11m     10.244.1.62   k8s-worker1   <none>            <none>
daemonset-nginx-79mdq   0/1     Completed           0          11m     10.244.2.45   k8s-worker2   <none>            <none>
daemonset-nginx-79mdq   0/1     Completed           0          11m     10.244.2.45   k8s-worker2   <none>            <none>
daemonset-nginx-7z8sj   0/1     Completed           0          11m     10.244.1.62   k8s-worker1   <none>            <none>
daemonset-nginx-psrvx   0/1     Pending             0          0s      <none>        <none>        <none>            <none>
daemonset-nginx-psrvx   0/1     Pending             0          0s      <none>        k8s-worker1   <none>            <none>
daemonset-nginx-psrvx   0/1     ContainerCreating   0          0s      <none>        k8s-worker1   <none>            <none>
daemonset-nginx-7z8sj   0/1     Completed           0          11m     10.244.1.62   k8s-worker1   <none>            <none>
daemonset-nginx-7z8sj   0/1     Completed           0          11m     10.244.1.62   k8s-worker1   <none>            <none>
daemonset-nginx-psrvx   1/1     Running             0          1s      10.244.1.63   k8s-worker1   <none>            <none>

	# nginx:1.31.1 에서 nginx:1.31로 롤백 확인
[root@k8s-master ~]# kubectl  describe  daemonsets  daemonset-nginx  | grep Image
    Image:         nginx:1.31
```

정리: DaemonSet은 노드 수에 맞춰 자동으로 늘고 줄며, ReplicaSet 대신 **ControllerRevision**으로 이력을 관리한다. 다음은 이름과 저장소의 정체성을 고정해야 하는 상태 저장(stateful) 워크로드를 위한 **StatefulSet**이다.

---

## StatefulSet

- **StatefulSet**은 상태(State)가 있는 애플리케이션을 안정적으로 운영하기 위한 Kubernetes 컨트롤러이다.
- Deployment와 비슷하게 여러 Pod를 관리하지만, StatefulSet은 각 Pod의 고유한 이름과 순서를 유지한다.
- 상태(state)란 애플리케이션이 실행되면서 계속 기억하거나 유지해야 하는 정보를 말한다.

예:
- 데이터
- 고유한 이름
- Pod의 순서
- 각 인스턴스의 역할
- 재시작 후에도 유지해야 하는 정보

- 예를 들어 데이터베이스에서는 mysql-0, mysql-1, mysql-2 처럼 각각의 인스턴스가 구분될 필요가 있을 수 있다.
- Deployment/ReplicaSet은 Pod가 죽으면 새로 만들 때 이름도 바뀌고 어느 노드에 뜰지도 바뀌고 Pod의 정체성이 계속 바뀌어도 상관없는 무상태(Stateless) 서비스에 최적화되어 있다.
- 반면 StatefulSet은 Pod의 이름이 고정되고 Pod마다 고유한 저장소(PVC)를 붙일 수 있고 Pod를 만들고 지우는 순서가 보장된다. 그래서 DB, 메시지큐, 분산 시스템처럼 각 인스턴스가 구분되어야 하는 서비스에 사용한다.

### Deployment와 StatefulSet 차이

**(1) Pod 이름(정체성)**
- Deployment: pod 이름이 랜덤 해시가 붙음 (예: web-7f9d7c9d7f-abcde). 죽으면 다른 이름으로 다시 생성됨
- StatefulSet: pod 이름이 규칙적으로 고정됨 (예: db-0, db-1, db-2). db-1이 죽으면 db-1이라는 이름으로 다시 생성됨. 즉, 정체성이 유지된다.

**(2) 저장소(Persistent Volume)**
- Deployment: 여러 Pod가 같은 PVC를 쓰면 충돌 가능. 보통은 공유 저장소가 아니면 위험
- StatefulSet: 각 Pod마다 전용 PVC를 자동으로 만들 수 있음 (db-0 전용 PVC, db-1 전용 PVC …). Pod가 재생성되어도 그 Pod 번호의 PVC를 다시 붙인다. (그래서 데이터가 유지된다)

**(3) 생성/삭제 순서 보장**
- Deployment: 동시에 여러 Pod가 막 생성/삭제될 수 있음. 순서 보장 없음
- StatefulSet: 기본적으로 순서가 보장됨. 생성: 0 --> 1 --> 2 순서대로 생성. 삭제: 2 --> 1 --> 0 역순으로 삭제. 앞 Pod가 Ready 상태가 되어야 다음 Pod 생성 같은 흐름이 가능

**(4) 업데이트(롤링업데이트)**
- Deployment: 일반적으로 빠르게 순차 업데이트
- StatefulSet: 업데이트도 순서대로 진행. 데이터가 있는 서비스는 업데이트 시 순서가 중요해서 StatefulSet 방식이 필요할 때가 많다.

### StatefulSet이 꼭 필요한 대표 서비스 예시

(1) 데이터베이스
- mysql, mariadb, postgresql
- DB는 각 인스턴스가 자기 데이터 저장소를 가져야 한다
- 죽었다가 살아나도 내 데이터 디스크를 붙여야 한다

(2) 분산 시스템 / 클러스터형 서비스
- kafka, zookeeper, elasticsearch, redis cluster(일부 구성)
- 이런 것들은 노드마다 역할이 있고(leader/follower) 특정 노드 이름으로 통신하거나 노드 수를 기준으로 클러스터가 구성된다. 그래서 고정된 정체성 + 고정된 스토리지가 필요하다.

### Headless Service (헤드리스 서비스)

- StatefulSet Pod들은 서로를 고정된 DNS 이름으로 찾아야 하는 경우가 많다.
- 예를 들어 db-0이 db-1에게 연결할 때 IP를 쓰면 Pod 재생성 때 IP가 바뀔 수 있다. 그래서 DNS 이름으로 통신해야 안정적이다.
- Headless Service는 LoadBalancer처럼 하나의 IP로 묶어주는 서비스가 아니라 각 Pod의 DNS를 만들어주는 서비스다.

StatefulSet DNS의 전형적인 형태
- `<pod이름>.<서비스이름>.<네임스페이스>.svc.cluster.local`
- 예) `db-0.db-headless.default.svc.cluster.local`
- 예) `db-1.db-headless.default.svc.cluster.local`
- 즉, Pod 개별 주소록을 만들어주는 역할을 한다.

### StatefulSet 구성 요소

StatefulSet을 만들 때 보통 3개가 같이 온다.

1) StatefulSet 오브젝트
- 몇 개 Pod를 유지할지(replicas)
- pod template(컨테이너 정의)
- volumeClaimTemplates(Pod별 PVC 자동 생성 템플릿)

(2) Headless Service
- serviceName으로 StatefulSet과 연결
- Pod DNS 생성

3) PV/PVC
- 각 Pod가 쓰는 저장소
- StatefulSet이 만든 PVC는 Pod가 살아있든 죽었든 남는다. (Pod 삭제해도 데이터가 남는 게 핵심)

가장 중요한 개념 3개
1) 고정된 이름 : db-0, db-1, db-2 처럼 번호 붙은 고정 Pod 이름
2) Pod별 전용 디스크 : 각 Pod마다 PVC가 별도로 생성되어 데이터 유지
3) 순서 보장 : 0부터 만들고, 역순으로 지움. 업데이트도 순서대로 진행

### 실무 상황 예시

- 상황: mysql을 3개로 운영(클러스터 or 복제 구조)
- mysql-0, mysql-1, mysql-2을 서비스 하는 상황에서 mysql-1이 장애로 죽었다.
- Deployment라면 새 Pod가 mysql-xxxxx 이런 이름으로 생기고 어떤 디스크를 붙여야 하는지 불명확해지거나 재구성 과정이 복잡해진다.
- StatefulSet이라면 mysql-1이라는 이름 그대로 다시 살아난다.
- mysql-1 전용 PVC를 다시 붙인다.
  - 클러스터 입장에서 mysql-1이 다시 돌아왔네 처럼 안정적으로 처리 가능

### StatefulSet 쓸 때 주의점

(1) 스토리지가 중요하다
- StatefulSet은 결국 디스크가 핵심이다.
- 따라서 StorageClass, PV 동작을 이해해야 운영이 된다.

(2) Pod를 지워도 PVC는 남는다.
- Pod 삭제는 데이터 삭제가 아니다.
- 재배포/재구성할 때 PVC가 남아 있어서 이전 데이터가 그대로 붙을 수 있다.
- 필요하면 PVC까지 삭제해야 완전 초기화된다.

(3) 무상태 서비스에 쓰면 오히려 불편
- 단순 웹서버 nginx 같은 건 Deployment가 더 적합하다.
- StatefulSet은 고정성 때문에 유연성이 떨어질 수 있다.
- StatefulSet은 상태(state)가 있는 애플리케이션을 쿠버네티스에서 안정적으로 운영하기 위한 컨트롤러다.

### StatefulSet 실습

```bash
[root@k8s-master ~]# vi statefulset-exam.yaml
```
```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: sf-nginx

spec:
  replicas: 3
  serviceName: sf-service
  podManagementPolicy: OrderedReady
# podManagementPolicy: Parallel
  selector:
    matchLabels:
      app: web

  template:
    metadata:
      name: nginx-pod
      labels:
        app: web
    spec:
      containers:
      - name: nginx-container
        image: nginx:1.31
```

- podManagementPolicy: OrderedReady (default)
  - Pod를 순서대로 생성한다.
  - 앞 번호의 Pod가 Running/Ready 상태가 되어야 다음 Pod 생성을 진행한다.
  - Pod 간 순서가 중요할 때 사용한다.
  - 앞 번호의 Pod가 정상 Ready 상태가 된 뒤 다음 번호의 Pod를 생성해야 하는 경우에 적합하다.
  - 생성: pod-0 --> pod-1 --> pod-2. StatefulSet을 축소하거나 삭제할 때는 높은 번호부터 역순으로 처리한다.
  - 삭제: pod-2 --> pod-1 --> pod-0. StatefulSet을 축소하거나 삭제할 때는 높은 번호부터 역순으로 처리한다.

- podManagementPolicy: Parallel
  - Pod를 순서대로 기다리지 않고 병렬로 생성한다.
  - 앞 Pod가 Ready 상태가 될 때까지 기다리지 않고 다른 Pod도 생성한다.
  - Pod 간 생성 순서가 중요하지 않을 때 사용한다.
  - 각 Pod가 서로 독립적으로 실행될 수 있고 빠르게 여러 Pod를 생성해야 하는 경우에 적합하다.
  - 생성: pod-0, pod-1, pod-2 생성 요청이 병렬로 진행. 각 Pod는 독립적으로 Running/Ready 상태가 된다.
  - Scale Down 시에도 OrderedReady처럼 앞 Pod의 상태를 기다리는 순차적인 관리 제약을 적용하지 않고 병렬적으로 처리할 수 있다.

```bash
	# 터미널 2
[root@k8s-master ~]# kubectl  apply  -f  statefulset-exam.yaml  --dry-run=client
statefulset.apps/sf-nginx created (dry run)

	# 터미널 1
[root@k8s-master ~]# kubectl  get  pods  -o  wide  --watch

	# 터미널 2
[root@k8s-master ~]# kubectl  apply  -f  statefulset-exam.yaml
statefulset.apps/sf-nginx created

	# 터미널 1
[root@k8s-master ~]# kubectl  get  pods  -o  wide  --watch
NAME         READY   STATUS              RESTARTS   AGE   IP            NODE          NOMINATED NODE   READINESS GATES
sf-nginx-0   0/1     Pending             0          0s    <none>        <none>        <none>            <none>
sf-nginx-0   0/1     Pending             0          0s    <none>        k8s-worker2   <none>            <none>
sf-nginx-0   0/1     ContainerCreating   0          0s    <none>        k8s-worker2   <none>            <none>
sf-nginx-0   1/1     Running             0          1s    10.244.2.47   k8s-worker2   <none>            <none>

sf-nginx-1   0/1     Pending             0          0s    <none>        <none>        <none>            <none>
sf-nginx-1   0/1     Pending             0          0s    <none>        k8s-worker1   <none>            <none>
sf-nginx-1   0/1     ContainerCreating   0          0s    <none>        k8s-worker1   <none>            <none>
sf-nginx-1   1/1     Running             0          1s    10.244.1.64   k8s-worker1   <none>            <none>

sf-nginx-2   0/1     Pending             0          0s    <none>        <none>        <none>            <none>
sf-nginx-2   0/1     Pending             0          0s    <none>        k8s-worker1   <none>            <none>
sf-nginx-2   0/1     ContainerCreating   0          0s    <none>        k8s-worker1   <none>            <none>
sf-nginx-2   1/1     Running             0          1s    10.244.1.65   k8s-worker1   <none>            <none>
```

### scale-out (확장)

```bash
[root@k8s-master ~]# kubectl  get  pods  -o  wide  --watch
NAME         READY   STATUS    RESTARTS   AGE   IP            NODE          NOMINATED NODE   READINESS GATES
sf-nginx-0   1/1     Running   0          1s    10.244.2.47   k8s-worker2   <none>            <none>
sf-nginx-1   1/1     Running   0          1s    10.244.1.64   k8s-worker1   <none>            <none>
sf-nginx-2   1/1     Running   0          1s    10.244.1.65   k8s-worker1   <none>            <none>

[root@k8s-master ~]# kubectl  scale  statefulset  sf-nginx  --replicas=5
statefulset.apps/sf-nginx scaled

	# 터미널 1
[root@k8s-master ~]# kubectl  get  pods  -o  wide  --watch
NAME         READY   STATUS              RESTARTS   AGE   IP            NODE          NOMINATED NODE   READINESS GATES
sf-nginx-0   1/1     Running             0          1s    10.244.2.47   k8s-worker2   <none>            <none>
sf-nginx-1   1/1     Running             0          1s    10.244.1.64   k8s-worker1   <none>            <none>
sf-nginx-2   1/1     Running             0          1s    10.244.1.65   k8s-worker1   <none>            <none>

sf-nginx-3   0/1     Pending             0          0s    <none>        <none>        <none>            <none>
sf-nginx-3   0/1     Pending             0          0s    <none>        k8s-worker2   <none>            <none>
sf-nginx-3   0/1     ContainerCreating   0          0s    <none>        k8s-worker2   <none>            <none>
sf-nginx-3   1/1     Running             0          1s    10.244.2.48   k8s-worker1   <none>            <none>

sf-nginx-4   0/1     Pending             0          0s    <none>        <none>        <none>            <none>
sf-nginx-4   0/1     Pending             0          0s    <none>        k8s-worker1   <none>            <none>
sf-nginx-4   0/1     ContainerCreating   0          0s    <none>        k8s-worker1   <none>            <none>
sf-nginx-4   1/1     Running             0          1s    10.244.1.66   k8s-worker1   <none>            <none>
```

### scale-in (축소)

```bash
	# 터미널 1
[root@k8s-master ~]# kubectl  get  pods  -o  wide  --watch
NAME         READY   STATUS    RESTARTS   AGE   IP            NODE          NOMINATED NODE   READINESS GATES
sf-nginx-0   1/1     Running   0          1s    10.244.2.47   k8s-worker2   <none>            <none>
sf-nginx-1   1/1     Running   0          1s    10.244.1.64   k8s-worker1   <none>            <none>
sf-nginx-2   1/1     Running   0          1s    10.244.1.65   k8s-worker1   <none>            <none>
sf-nginx-3   1/1     Running   0          1s    10.244.2.48   k8s-worker1   <none>            <none>
sf-nginx-4   1/1     Running   0          1s    10.244.1.66   k8s-worker1   <none>            <none>

[root@k8s-master ~]# kubectl  scale  statefulset  sf-nginx  --replicas=2
statefulset.apps/sf-nginx scaled

[root@k8s-master ~]# kubectl  get  pods  -o  wide  --watch
NAME         READY   STATUS    RESTARTS   AGE   IP            NODE          NOMINATED NODE   READINESS GATES
sf-nginx-0   1/1     Running   0          1s    10.244.2.47   k8s-worker2   <none>            <none>
sf-nginx-1   1/1     Running   0          1s    10.244.1.64   k8s-worker1   <none>            <none>
sf-nginx-2   1/1     Running   0          1s    10.244.1.65   k8s-worker1   <none>            <none>
sf-nginx-3   1/1     Running   0          1s    10.244.2.48   k8s-worker1   <none>            <none>
sf-nginx-4   1/1     Running   0          1s    10.244.1.66   k8s-worker1   <none>            <none>

sf-nginx-4   1/1     Terminating   0          3m1s    10.244.1.66   k8s-worker1   <none>            <none>
sf-nginx-4   1/1     Terminating   0          3m1s    10.244.1.66   k8s-worker1   <none>            <none>
sf-nginx-4   0/1     Completed     0          3m1s    10.244.1.66   k8s-worker1   <none>            <none>
sf-nginx-4   0/1     Completed     0          3m1s    10.244.1.66   k8s-worker1   <none>            <none>
sf-nginx-4   0/1     Completed     0          3m1s    10.244.1.66   k8s-worker1   <none>            <none>

sf-nginx-3   1/1     Terminating   0          3m2s    10.244.2.48   k8s-worker2   <none>            <none>
sf-nginx-3   1/1     Terminating   0          3m2s    10.244.2.48   k8s-worker2   <none>            <none>
sf-nginx-3   0/1     Completed     0          3m2s    10.244.2.48   k8s-worker2   <none>            <none>
sf-nginx-3   0/1     Completed     0          3m3s    10.244.2.48   k8s-worker2   <none>            <none>
sf-nginx-3   0/1     Completed     0          3m3s    10.244.2.48   k8s-worker2   <none>            <none>

sf-nginx-2   1/1     Terminating   0          7m29s   10.244.1.65   k8s-worker1   <none>            <none>
sf-nginx-2   1/1     Terminating   0          7m29s   10.244.1.65   k8s-worker1   <none>            <none>
sf-nginx-2   0/1     Completed     0          7m29s   10.244.1.65   k8s-worker1   <none>            <none>
sf-nginx-2   0/1     Completed     0          7m29s   10.244.1.65   k8s-worker1   <none>            <none>
sf-nginx-2   0/1     Completed     0          7m29s   10.244.1.65   k8s-worker1   <none>            <none>
```

### StatefulSet 롤링 업데이트

```bash
[root@k8s-master ~]# kubectl  edit  statefulsets  sf-nginx
~~~~~~~~~~~~ 중간 생략 ~~~~~~~~~~~~
spec:
  persistentVolumeClaimRetentionPolicy:
    whenDeleted: Retain
    whenScaled: Retain
  podManagementPolicy: OrderedReady
  replicas: 3
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app: web
  serviceName: sf-service
  template:
    metadata:
      labels:
        app: web
      name: nginx-pod
    spec:
      containers:
      - image: nginx:1.31.1 		#  nginx:1.31  -->  nginx:1.31.1
        imagePullPolicy: IfNotPresent
        name: nginx-container
        resources: {}
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
~~~~~~~~~~~~ 중간 생략 ~~~~~~~~~~~~

[root@k8s-master ~]# kubectl  get  pods  -o  wide  --watch
NAME         READY   STATUS    RESTARTS   AGE     IP            NODE          NOMINATED NODE   READINESS GATES
sf-nginx-0   1/1     Running   0          16m     10.244.2.47   k8s-worker2   <none>            <none>
sf-nginx-1   1/1     Running   0          5m45s   10.244.1.68   k8s-worker1   <none>            <none>
sf-nginx-2   1/1     Running   0          6m12s   10.244.1.67   k8s-worker1   <none>            <none>
sf-nginx-2   1/1     Terminating   0          8m21s   10.244.1.67   k8s-worker1   <none>            <none>
sf-nginx-2   1/1     Terminating   0          8m21s   10.244.1.67   k8s-worker1   <none>            <none>
sf-nginx-2   0/1     Completed     0          8m21s   10.244.1.67   k8s-worker1   <none>            <none>
sf-nginx-2   0/1     Completed     0          8m22s   10.244.1.67   k8s-worker1   <none>            <none>
sf-nginx-2   0/1     Completed     0          8m22s   10.244.1.67   k8s-worker1   <none>            <none>
sf-nginx-2   0/1     Pending       0          0s      <none>        <none>        <none>            <none>
sf-nginx-2   0/1     Pending       0          0s      <none>        k8s-worker1   <none>            <none>
sf-nginx-2   0/1     ContainerCreating   0    0s      <none>        k8s-worker1   <none>            <none>
sf-nginx-2   1/1     Running             0    1s      10.244.1.69   k8s-worker1   <none>            <none>
sf-nginx-1   1/1     Terminating         0    7m56s   10.244.1.68   k8s-worker1   <none>            <none>
sf-nginx-1   1/1     Terminating         0    7m56s   10.244.1.68   k8s-worker1   <none>            <none>
sf-nginx-1   0/1     Completed           0    7m56s   10.244.1.68   k8s-worker1   <none>            <none>
sf-nginx-1   0/1     Completed           0    7m57s   10.244.1.68   k8s-worker1   <none>            <none>
sf-nginx-1   0/1     Completed           0    7m57s   10.244.1.68   k8s-worker1   <none>            <none>
sf-nginx-1   0/1     Pending             0    0s      <none>        <none>        <none>            <none>
sf-nginx-1   0/1     Pending             0    0s      <none>        k8s-worker1   <none>            <none>
sf-nginx-1   0/1     ContainerCreating   0    0s      <none>        k8s-worker1   <none>            <none>
sf-nginx-1   1/1     Running             0    1s      10.244.1.70   k8s-worker1   <none>            <none>
sf-nginx-0   1/1     Terminating         0    18m     10.244.2.47   k8s-worker2   <none>            <none>
sf-nginx-0   1/1     Terminating         0    18m     10.244.2.47   k8s-worker2   <none>            <none>
sf-nginx-0   0/1     Completed           0    18m     10.244.2.47   k8s-worker2   <none>            <none>
sf-nginx-0   0/1     Completed           0    18m     10.244.2.47   k8s-worker2   <none>            <none>
sf-nginx-0   0/1     Completed           0    18m     10.244.2.47   k8s-worker2   <none>            <none>
sf-nginx-0   0/1     Pending             0    0s      <none>        <none>        <none>            <none>
sf-nginx-0   0/1     Pending             0    0s      <none>        k8s-worker2   <none>            <none>
sf-nginx-0   0/1     ContainerCreating   0    0s      <none>        k8s-worker2   <none>            <none>
sf-nginx-0   1/1     Running             0    1s      10.244.2.49   k8s-worker2   <none>            <none>
```

- StatefulSet 롤링 업데이트도 순서를 보장한다: 가장 높은 번호(sf-nginx-2)부터 먼저 교체되고, 마지막으로 sf-nginx-0이 교체된다.

정리: StatefulSet은 고정된 이름·전용 PVC·순서 보장이라는 세 축으로 DB 등 상태 저장 서비스를 안정적으로 운영한다. 다음 실습은 관점을 바꿔, 항상 떠 있어야 하는 서비스가 아니라 "한 번 실행하고 끝나는 작업"을 다루는 **Job**의 필요성을 보여준다.

---

## 쿠버네티스는 desired state를 관리한다 (Job 도입부 실습)

```bash
[root@k8s-master ~]# kubectl  get  pods  -o  wide  --watch

[root@k8s-master ~]# kubectl  run  jobpod  --image=centos:7  --command  sleep 5

[root@k8s-master ~]# kubectl  get  pods  -o  wide  --watch
NAME     READY   STATUS              RESTARTS       AGE   IP            NODE          NOMINATED NODE   READINESS GATES
jobpod   0/1     Pending             0              0s    <none>        <none>        <none>            <none>
jobpod   0/1     Pending             0              0s    <none>        k8s-worker2   <none>            <none>
jobpod   0/1     ContainerCreating   0              0s    <none>        k8s-worker2   <none>            <none>
jobpod   1/1     Running             0              1s    10.244.2.3    k8s-worker2   <none>            <none>
jobpod   0/1     Completed           0              6s    10.244.2.3    k8s-worker2   <none>            <none>
jobpod   1/1     Running             1 (2s ago)     7s    10.244.2.3    k8s-worker2   <none>            <none>
jobpod   0/1     Completed           1 (7s ago)     12s   10.244.2.3    k8s-worker2   <none>            <none>
jobpod   0/1     CrashLoopBackOff    1 (12s ago)    23s   10.244.2.3    k8s-worker2   <none>            <none>
jobpod   1/1     Running             2 (12s ago)    23s   10.244.2.3    k8s-worker2   <none>            <none>
jobpod   0/1     Completed           2 (17s ago)    28s   10.244.2.3    k8s-worker2   <none>            <none>
jobpod   0/1     CrashLoopBackOff    2 (26s ago)    54s   10.244.2.3    k8s-worker2   <none>            <none>
jobpod   1/1     Running             3 (26s ago)    54s   10.244.2.3    k8s-worker2   <none>            <none>

[root@k8s-master ~]# kubectl  delete  pods jobpod
pod "jobpod" deleted from default namespace
```

- 이 실습은 "쿠버네티스는 프로세스를 관리하는 게 아니라 원하는 상태(desired state)를 관리한다." 라는 것을 보여주기 위한 예제다.
- 사용자가 쿠버네티스에게 한 말은 딱 하나다.
  - jobpod라는 Pod는 존재해야 한다.
  - 쿠버네티스는 이 말을 이렇게 해석한다.
  - 그럼 jobpod는 항상 살아 있어야겠네

- 왜 Pod가 끝났는데 다시 살아나는가
  - 실행한 명령 : `kubectl run jobpod --image=centos:7 --command sleep 5`
  - 이 명령의 의미는 다음과 같다.
    - jobpod라는 Pod를 하나 만들고 그 Pod 안에서 sleep 5를 실행한다.
    - restartPolicy: Always = 컨테이너가 종료되면 같은 Pod 안에서 다시 실행하라
    - sleep 5는 5초 후 정상 종료(exit 0)한다. 하지만 쿠버네티스 입장에서는 정상 종료든 비정상 종료든 상관없이 컨테이너가 멈췄다라는 사실만 중요하다.
      그래서 kubelet은 이렇게 행동한다. --> 컨테이너가 멈췄네? 그럼 다시 실행해서 원하는 상태를 맞춰야지.
    - Completed --> Running --> Completed을 계속 반복한다.

- 이건 버그가 아니라 쿠버네티스가 충실하게 일하고 있다는 증거다.
- Pod는 존재해야 하고 컨테이너는 계속 실행 상태여야 한다. 그러니 종료되면 다시 실행한다.
- 이게 바로 "쿠버네티스는 작업이 끝났는지를 보지 않는다." 라는 특징이다.
- 쿠버네티스는 기본적으로 서비스를 오래 유지하는 플랫폼이다.

- CrashLoopBackOff는 에러가 발생했다는 뜻이 아니라 너무 짧은 시간 안에 컨테이너가 계속 종료되고 있다. 그래서 kubelet이 스스로 이렇게 판단한다.
- 이건 문제가 있을 수도 있으니 잠깐 쉬었다가 다시 실행하자 — 이것이 CrashLoopBackOff이다.

- 그런데 만약 이 작업을 한 번만 실행하고 끝내고 싶다면 어떻게 해야 할까?

---

## Job Controller

- **Job Controller**는 한 번 실행하고 끝나는 작업을 쿠버네티스에서 실행하기 위한 컨트롤러다.
- 쿠버네티스를 배우다 보면 가장 먼저 접하는 개념이 Deployment다.
- Deployment는 웹 서버처럼 항상 살아 있어야 하는 서비스를 관리하기 위한 컨트롤러다.
- 하지만 실무에서는 항상 떠 있을 필요가 없는 작업도 많다.
  - DB 마이그레이션을 한 번 실행하는 작업
  - 하루의 로그를 모아서 정리하는 작업
  - 통계 데이터를 계산해서 결과 파일을 만드는 작업
  - 백업 스크립트를 한 번 실행하는 작업
- 이 작업들의 공통점은 계속 실행될 필요가 없다는 것이다.
- 성공하면 끝이다. (실패시 다시 시도는 필요)
- 이런 작업을 Deployment로 실행하면 문제가 생긴다.
- Deployment는 Pod가 종료되면 장애라고 판단하고 다시 띄우기 때문이다.
- 그래서 쿠버네티스에는 한 번 실행하고 끝나는 작업을 위한 전용 컨트롤러인 Job Controller가 존재한다.

### Job과 Pod의 역할 관계

- Job을 이해할 때 가장 중요한 관점은 Job은 일을 하지 않는다는 점이다.
- 실제 작업을 수행하는 주체는 Pod이고 Job Controller는 그 Pod를 관리하는 관리자이다.

사람에 비유
- Pod : 현장에서 실제 일을 하는 작업자
- Job : 작업자가 일을 제대로 끝냈는지 확인하는 관리자

1) 작업을 수행할 Pod를 생성
2) Pod가 성공했는지 실패했는지 감시
3) 실패하면 다시 Pod를 생성해서 재시도
4) 성공하면 작업을 종료 처리

- 즉, Job은 작업 실행 책임자이지 작업 실행자는 아니다.

### Job Controller는 성공과 실패를 어떻게 판단하는가

- Job이 성공했는지 실패했는지를 판단하는 기준은 매우 단순하다.
- 컨테이너의 종료 코드(exit code)를 사용한다.
- 리눅스에서 프로그램은 종료될 때 숫자를 하나 남긴다.
  - exit 0 : 정상 종료
  - exit 1 이상 : 비정상 종료
  - Job은 오직 종료 코드(exit code)만 본다.

- 컨테이너가 exit 0으로 종료되면 성공, exit 0이 아니면 실패
  - 로그 내용은 보지 않는다.
  - 에러 메시지가 있었는지도 보지 않는다.
  - 출력이 많았는지 적었는지도 상관없다.
  - 0으로 끝났냐, 아니냐 이것만으로 판단한다.

- 그래서 Job에서 실행되는 스크립트나 프로그램은 실패 시 반드시 exit 1 같은 실패 코드로 종료해야 한다.

### Job의 전체 동작 흐름 (시간 순서로 이해)

Job 하나가 실행될 때 실제 내부에서는 다음 순서로 움직인다.

1단계: 사용자가 Job을 생성한다.
2단계: Job Controller가 이 Job을 수행할 Pod가 필요하다고 판단한다.
3단계: Job Controller가 Pod를 하나 생성한다.
4단계: Pod 안의 컨테이너가 실행된다. (배치 스크립트 실행 / 프로그램 수행 / 데이터 처리 작업 수행)
5단계: 컨테이너가 종료된다.
6단계: Job Controller가 종료 코드를 확인한다.
  - exit 0이면 Job은 성공 상태가 된다.
  - exit 0이 아니면 Job은 아직 성공하지 못했다고 판단하고 새로운 Pod를 생성해 작업을 다시 시도한다.

- 이 과정을 성공할 때까지 반복하는 것이 Job의 핵심 동작이다.

### Job의 주요 개념 옵션

- **completions** : 이 Job이 총 몇 번 성공해야 끝나는가. 기본적인 Job은 completions 1. 여러 번 성공해야 하는 반복 작업도 가능
- **parallelism** : 동시에 몇 개의 Pod를 실행할 것인가. 병렬 처리가 필요한 작업에서 사용
- **backoffLimit** : 실패를 몇 번까지 허용할 것인가. 무한 재시도를 막기 위한 안전장치

- 이 옵션들을 조합하면 단일 실행 Job, 병렬 Job, 반복 성공 Job 같은 다양한 작업 패턴을 만들 수 있다.

### Job이 주로 쓰이는 사례

Job은 다음과 같은 상황에서 거의 필수적으로 사용된다.
- DB 백업 작업
- 데이터 정리 및 배치 처리
- 통계 집계
- 캐시 초기화
- 테스트 자동 실행
- 마이그레이션 스크립트 실행

공통 특징
- 항상 실행될 필요 없음
- 결과가 성공/실패로 명확
- 실패 시 재시도 필요

### Job과 CronJob의 관계

- 여기서 자연스럽게 나오는 개념이 CronJob이다.
  - Job : 한 번 실행
  - CronJob : 정해진 시간마다 Job을 생성
- 즉, CronJob --> Job --> Pod 이 순서로 동작한다.
- CronJob은 직접 일을 하지 않고 Job을 정기적으로 만들어주는 역할만 한다.

### Job을 쓰면 안 되는 경우

다음과 같은 서비스에는 Job을 쓰면 안 된다.
- 웹 서버
- API 서버
- 지속적으로 요청을 받아야 하는 서비스는 Deployment를 사용해야 한다.
- Job은 반드시 끝이 있는 작업에만 사용해야 한다.

### Job 실습 — completions 기본값(1)

```bash
[root@k8s-master ~]# vi job-exam.yaml
```
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: centos-job
spec:
#  completions: 5
#  parallelism: 2
#  activeDeadlineSeconds: 5
  template:
    spec:
      containers:
      - name: centos-container
        image: centos:7
        command: ["bash"]
        args:
        - "-c"
        - "date; echo 'Hello World'; sleep 20; date; echo 'Bye'"
      restartPolicy: Never
#      restartPolicy: OnFailure
#  backoffLimit: 3
```

- completions : 이 Job이 총 몇 번 성공해야 끝나는가. 기본적인 Job은 completions 1. 여러 번 성공해야 하는 반복 작업도 가능
- parallelism : 동시에 몇 개의 Pod를 실행할 것인가. 병렬 처리가 필요한 작업에서 사용
- backoffLimit : 실패를 몇 번까지 허용할 것인가. 무한 재시도를 막기 위한 안전장치

```bash
[root@k8s-master ~]# kubectl  apply  -f  job-exam.yaml
job.batch/centos-job created

[root@k8s-master ~]# kubectl  get  pods  -o  wide  --watch
NAME               READY   STATUS              RESTARTS   AGE   IP           NODE          NOMINATED NODE   READINESS GATES
centos-job-vssh5   0/1     Pending             0          0s    <none>       <none>        <none>            <none>
centos-job-vssh5   0/1     Pending             0          0s    <none>       k8s-worker2   <none>            <none>
centos-job-vssh5   0/1     ContainerCreating   0          0s    <none>       k8s-worker2   <none>            <none>
centos-job-vssh5   1/1     Running             0          1s    10.244.2.5   k8s-worker2   <none>            <none>
centos-job-vssh5   0/1     Completed           0          21s   10.244.2.5   k8s-worker2   <none>            <none>
centos-job-vssh5   0/1     Completed           0          22s   10.244.2.5   k8s-worker2   <none>            <none>
centos-job-vssh5   0/1     Completed           0          23s   10.244.2.5   k8s-worker2   <none>            <none>

[root@k8s-master ~]# kubectl  logs  job/centos-job
Thu Aug 20 01:45:00 UTC 2026
Hello World
Bye
```

### Job 재실행 — sleep 시간 늘려서 pod 강제 삭제 반응 확인

```bash
[root@k8s-master ~]# vi job-exam.yaml
```
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: centos-job
spec:
#  completions: 5
#  parallelism: 2
#  activeDeadlineSeconds: 5
  template:
    spec:
      containers:
      - name: centos-container
        image: centos:7
        command: ["bash"]
        args:
        - "-c"
        - "date; echo 'Hello World'; sleep 60; date; echo 'Bye'"		# 60초로 수정
      restartPolicy: Never
#      restartPolicy: OnFailure
#  backoffLimit: 3
```

```bash
[root@k8s-master ~]# kubectl  apply  -f  job-exam.yaml
job.batch/centos-job created

[root@k8s-master ~]# kubectl  get  pods  -o  wide  --watch
NAME               READY   STATUS              RESTARTS   AGE   IP           NODE          NOMINATED NODE   READINESS GATES
centos-job-ttw2p   0/1     Pending             0          0s    <none>       <none>        <none>            <none>
centos-job-ttw2p   0/1     Pending             0          0s    <none>       k8s-worker2   <none>            <none>
centos-job-ttw2p   0/1     ContainerCreating   0          0s    <none>       k8s-worker2   <none>            <none>
centos-job-ttw2p   1/1     Running             0          1s    10.244.2.5   k8s-worker2   <none>            <none>

[root@k8s-master ~]# kubectl   delete   pods  centos-job-ttw2p
pod "centos-job-ttw2p" deleted from default namespace

NAME               READY   STATUS              RESTARTS   AGE   IP           NODE          NOMINATED NODE   READINESS GATES
centos-job-ttw2p   0/1     Pending             0          0s    <none>       <none>        <none>            <none>
centos-job-ttw2p   0/1     Pending             0          0s    <none>       k8s-worker2   <none>            <none>
centos-job-ttw2p   0/1     ContainerCreating   0          0s    <none>       k8s-worker2   <none>            <none>
centos-job-ttw2p   1/1     Running             0          1s    10.244.2.5   k8s-worker2   <none>            <none>

centos-job-ttw2p   1/1     Terminating         0          17s   10.244.2.7   k8s-worker2   <none>            <none>
centos-job-ttw2p   1/1     Terminating         0          17s   10.244.2.7   k8s-worker2   <none>            <none>
centos-job-ttw2p   1/1     Terminating         0          18s   10.244.2.7   k8s-worker2   <none>            <none>
centos-job-x8jxz   0/1     Pending             0          0s    <none>       <none>        <none>            <none>
centos-job-x8jxz   0/1     Pending             0          0s    <none>       k8s-worker1   <none>            <none>
centos-job-x8jxz   0/1     ContainerCreating   0          0s    <none>       k8s-worker1   <none>            <none>
centos-job-x8jxz   1/1     Running             0          1s    10.244.1.4   k8s-worker1   <none>            <none>
centos-job-x8jxz   0/1     Completed           0          60s   10.244.1.4   k8s-worker1   <none>            <none>
centos-job-x8jxz   0/1     Completed           0          61s   10.244.1.4   k8s-worker1   <none>            <none>
centos-job-x8jxz   0/1     Completed           0          62s   10.244.1.4   k8s-worker1   <none>            <none>

[root@k8s-master ~]# kubectl  get  pods
NAME               READY   STATUS      RESTARTS   AGE
centos-job-x8jxz   0/1     Completed   0          6m20s

[root@k8s-master ~]# kubectl  get  job
NAME         STATUS     COMPLETIONS   DURATION   AGE
centos-job   Complete   1/1           89s        7m10s
```

- Complete : job이 정상적으로 작업을 완료한 상태
- 1/1 : 목표 작업 1개 완료 작업 1개
- DURATION : job이 생성되어 작업을 완료하는데 걸린 시간

```bash
[root@k8s-master ~]# kubectl  delete  pods centos-job-x8jxz
pod "centos-job-x8jxz" deleted from default namespace

[root@k8s-master ~]# kubectl  get  pods
No resources found in default namespace.
```

- Job은 성공 횟수(= completions)만 채우면 역할이 끝난다.
- Job의 completions의 default값은 1이기 때문에 성공 Pod 1개가 exit 0으로 끝난다.
- Job 상태가 Complete(1/1) 되고 그 순간부터 Job 컨트롤러는 더 만들 이유가 없다고 판단한다.

```bash
	# pod만 삭제시 job controller는 그대로 동작한다.
[root@k8s-master ~]# kubectl  get  job
NAME         STATUS     COMPLETIONS   DURATION   AGE
centos-job   Complete   1/1           89s        12m

[root@k8s-master ~]# kubectl  delete jobs.batch centos-job
job.batch "centos-job" deleted from default namespace
```

### Job 실습 — completions: 3

```bash
[root@k8s-master ~]# vi job-exam.yaml
```
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: centos-job
spec:
  completions: 3			# 주석 삭제 후 3으로 수정
#  parallelism: 2
#  activeDeadlineSeconds: 5
  template:
    spec:
      containers:
      - name: centos-container
        image: centos:7
        command: ["bash"]
        args:
        - "-c"
        - "date; echo 'Hello World'; sleep 20; date; echo 'Bye'"
#      restartPolicy: Never		# 주석 처리
      restartPolicy: OnFailure		# 주석 삭제
  backoffLimit: 3			# 주석 삭제
```

```bash
[root@k8s-master ~]# kubectl  apply  -f  job-exam.yaml  --dry-run=client
job.batch/centos-job created (dry run)
```

- completions: 5 (default = 1) : job 전체 기준으로 성공한 pod 수가 5개가 되면 job의 상태를 complete로 전환
- parallelism: 2 (default = 1) : job을 동시에 실행할 pod의 개수
- restartPolicy: Never : 컨테이너가 작업 실패시 pod안에서 컨테이너를 재시작하지 않는다. 필요시 Job Controller가 pod를 다시 생성해서 실행
- restartPolicy: OnFailure : 컨테이너가 작업 실패시 pod안에서 컨테이너를 재시작. Pod는 그대로 유지
- backoffLimit: 3 (default = 6) : Job이 작업을 실패시 재시도 횟수를 작성

```bash
[root@k8s-master ~]# kubectl  apply  -f  job-exam.yaml
job.batch/centos-job created

	# 해당 작업을 성공한 pod가 3개이면 작업 성공으로 판단
[root@k8s-master ~]# kubectl  get  pods  -o  wide  --watch
NAME               READY   STATUS              RESTARTS   AGE   IP            NODE          NOMINATED NODE   READINESS GATES
centos-job-v649p   0/1     Pending             0          0s    <none>        <none>        <none>            <none>
centos-job-v649p   0/1     Pending             0          0s    <none>        k8s-worker2   <none>            <none>
centos-job-v649p   0/1     ContainerCreating   0          0s    <none>        k8s-worker2   <none>            <none>
centos-job-v649p   1/1     Running             0          1s    10.244.2.9    k8s-worker2   <none>            <none>
centos-job-v649p   0/1     Completed           0          20s   10.244.2.9    k8s-worker2   <none>            <none>
centos-job-v649p   0/1     Completed           0          21s   10.244.2.9    k8s-worker2   <none>            <none>

centos-job-5vvlh   0/1     Pending             0          0s    <none>        <none>        <none>            <none>
centos-job-5vvlh   0/1     Pending             0          0s    <none>        k8s-worker2   <none>            <none>
centos-job-v649p   0/1     Completed           0          22s   10.244.2.9    k8s-worker2   <none>            <none>
centos-job-5vvlh   0/1     ContainerCreating   0          0s    <none>        k8s-worker2   <none>            <none>
centos-job-5vvlh   1/1     Running             0          1s    10.244.2.10   k8s-worker2   <none>            <none>
centos-job-5vvlh   0/1     Completed           0          21s   10.244.2.10   k8s-worker2   <none>            <none>
centos-job-5vvlh   0/1     Completed           0          23s   10.244.2.10   k8s-worker2   <none>            <none>

centos-job-djh5p   0/1     Pending             0          0s    <none>        <none>        <none>            <none>
centos-job-djh5p   0/1     Pending             0          0s    <none>        k8s-worker2   <none>            <none>
centos-job-5vvlh   0/1     Completed           0          24s   10.244.2.10   k8s-worker2   <none>            <none>
centos-job-djh5p   0/1     ContainerCreating   0          0s    <none>        k8s-worker2   <none>            <none>
centos-job-djh5p   1/1     Running             0          1s    10.244.2.11   k8s-worker2   <none>            <none>
centos-job-djh5p   0/1     Completed           0          21s   10.244.2.11   k8s-worker2   <none>            <none>
centos-job-djh5p   0/1     Completed           0          22s   10.244.2.11   k8s-worker2   <none>            <none>
centos-job-djh5p   0/1     Completed           0          23s   10.244.2.11   k8s-worker2   <none>            <none>

	# Job Controller 삭제
[root@k8s-master ~]# kubectl  delete jobs.batch centos-job
job.batch "centos-job" deleted from default namespace
```

### Job 실패 시나리오 — 명령어 오타 (bashz)

```bash
[root@k8s-master ~]# vi job-exam.yaml
```
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: centos-job
spec:
  completions: 3			# 주석 삭제 후 3으로 수정
#  parallelism: 2
#  activeDeadlineSeconds: 5
  template:
    spec:
      containers:
      - name: centos-container
        image: centos:7
        command: ["bashz"]		#  <--- bashc 강제 명령어 오타
        args:
        - "-c"
        - "date; echo 'Hello World'; sleep 20; date; echo 'Bye'"
#      restartPolicy: Never
      restartPolicy: OnFailure		#  <--- Test (작업 실패시 컨테이너 재시작)
  backoffLimit: 3			#  <--- Test (3번 연속 실패시 작업 실패로 간주)
```

```bash
[root@k8s-master ~]# kubectl  apply  -f  job-exam.yaml  --dry-run=client
job.batch/centos-job created (dry run)

[root@k8s-master ~]# kubectl  get  pods  -o  wide  --watch
NAME               READY   STATUS              RESTARTS      AGE   IP            NODE          NOMINATED NODE   READINESS GATES
centos-job-msqkl   0/1     Pending             0             0s    <none>        <none>        <none>            <none>
centos-job-msqkl   0/1     Pending             0             0s    <none>        k8s-worker2   <none>            <none>
centos-job-msqkl   0/1     ContainerCreating   0             0s    <none>        k8s-worker2   <none>            <none>
centos-job-msqkl   0/1     RunContainerError   0             1s    10.244.2.16   k8s-worker2   <none>            <none>
centos-job-msqkl   0/1     RunContainerError   1 (1s ago)    2s    10.244.2.16   k8s-worker2   <none>            <none>
centos-job-msqkl   0/1     CrashLoopBackOff    1 (13s ago)   14s   10.244.2.16   k8s-worker2   <none>            <none>
centos-job-msqkl   0/1     RunContainerError   2 (0s ago)    14s   10.244.2.16   k8s-worker2   <none>            <none>
centos-job-msqkl   0/1     CrashLoopBackOff    2 (22s ago)   36s   10.244.2.16   k8s-worker2   <none>            <none>
centos-job-msqkl   0/1     RunContainerError   3 (0s ago)    36s   10.244.2.16   k8s-worker2   <none>            <none>
centos-job-msqkl   0/1     Terminating         3 (1s ago)    37s   10.244.2.16   k8s-worker2   <none>            <none>
centos-job-msqkl   0/1     Terminating         3 (1s ago)    37s   10.244.2.16   k8s-worker2   <none>            <none>
centos-job-msqkl   0/1     Terminating         3             37s   10.244.2.16   k8s-worker2   <none>            <none>
centos-job-msqkl   0/1     StartError          3             37s   10.244.2.16   k8s-worker2   <none>            <none>
centos-job-msqkl   0/1     StartError          3             38s   10.244.2.16   k8s-worker2   <none>            <none>
centos-job-msqkl   0/1     StartError          3             38s   10.244.2.16   k8s-worker2   <none>            <none>

[root@k8s-master ~]# kubectl  get  job  centos-job
NAME         STATUS   COMPLETIONS   DURATION   AGE
centos-job   Failed   0/3           5m3s       5m3s

[root@k8s-master ~]# kubectl describe  jobs.batch  centos-job
Name:             centos-job
Namespace:        default
Selector:         batch.kubernetes.io/controller-uid=31513b28-7dfa-4cb1-aae7-26f5ce3aa214
Labels:           batch.kubernetes.io/controller-uid=31513b28-7dfa-4cb1-aae7-26f5ce3aa214
                  batch.kubernetes.io/job-name=centos-job
                  controller-uid=31513b28-7dfa-4cb1-aae7-26f5ce3aa214
                  job-name=centos-job
Annotations:      <none>
Parallelism:      1
Completions:      3
Completion Mode:  NonIndexed
Suspend:          false
Backoff Limit:    3
Start Time:       Thu, 20 Aug 2026 12:04:00 +0900
Pods Statuses:    0 Active (0 Ready) / 0 Succeeded / 1 Failed
Pod Template:
  Labels: batch.kubernetes.io/controller-uid=31513b28-7dfa-4cb1-aae7-26f5ce3aa214
          batch.kubernetes.io/job-name=centos-job
          controller-uid=31513b28-7dfa-4cb1-aae7-26f5ce3aa214
          job-name=centos-job
  Containers:
   centos-container:
    Image:      centos:7
    Port:       <none>
    Host Port:  <none>
    Command:
      bashz
    Args:
      -c
      date; echo 'Hello World'; sleep 20; echo 'Bye'
    Environment: <none>
    Mounts:      <none>
  Volumes:       <none>
  Node-Selectors: <none>
  Tolerations:    <none>
Events:
  Type     Reason                Age    From             Message
  ----     ------                ----   ----             -------
  Normal   SuccessfulCreate      6m23s  job-controller   Created pod: centos-job-msqkl
  Normal   SuccessfulDelete      5m46s  job-controller   Deleted pod: centos-job-msqkl
  Warning  BackoffLimitExceeded  5m45s  job-controller   Job has reached the specified ba
```

- BackoffLimitExceeded : BackoffLimit에 의해 Job이 Failed됨을 의미 (실패 한도 초과)

### Job 실습 — completions: 5, parallelism: 2

```bash
[root@k8s-master ~]# vi job-exam.yaml
```
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: centos-job
spec:
  completions: 5			# 3에서 5로 수정
  parallelism: 2			# 주석 제거
#  activeDeadlineSeconds: 5
  template:
    spec:
      containers:
      - name: centos-container
        image: centos:7
        command: ["bash"]
        args:
        - "-c"
        - "date; echo 'Hello World'; sleep 20; date; echo 'Bye'"
#      restartPolicy: Never
      restartPolicy: OnFailure
  backoffLimit: 3
```

```bash
[root@k8s-master ~]# kubectl  apply  -f  job-exam.yaml
job.batch/centos-job created
```

- completions: 5 # 5번 작업을 성공해야 completed가 된다.
- parallelism: 2 # 한번에 2개의 pod를 사용하여 작업 수행

```bash
[root@k8s-master ~]# kubectl  get  pods  -o  wide  --watch
NAME               READY   STATUS              RESTARTS   AGE   IP            NODE          NOMINATED NODE   READINESS GATES
centos-job-2rjjn   0/1     Pending             0          0s    <none>        <none>        <none>            <none>
centos-job-2rjjn   0/1     Pending             0          0s    <none>        k8s-worker2   <none>            <none>
centos-job-t2b4t   0/1     Pending             0          0s    <none>        <none>        <none>            <none>
centos-job-t2b4t   0/1     Pending             0          0s    <none>        k8s-worker1   <none>            <none>
centos-job-2rjjn   0/1     ContainerCreating   0          0s    <none>        k8s-worker2   <none>            <none>
centos-job-t2b4t   0/1     ContainerCreating   0          0s    <none>        k8s-worker1   <none>            <none>
centos-job-2rjjn   1/1     Running             0          1s    10.244.2.12   k8s-worker2   <none>            <none>
centos-job-t2b4t   1/1     Running             0          1s    10.244.1.5    k8s-worker1   <none>            <none>
centos-job-t2b4t   0/1     Completed           0          20s   10.244.1.5    k8s-worker1   <none>            <none>
centos-job-2rjjn   0/1     Completed           0          21s   10.244.2.12   k8s-worker2   <none>            <none>
centos-job-t2b4t   0/1     Completed           0          21s   10.244.1.5    k8s-worker1   <none>            <none>
centos-job-2rjjn   0/1     Completed           0          22s   10.244.2.12   k8s-worker2   <none>            <none>

centos-job-cgtjt   0/1     Pending             0          0s    <none>        <none>        <none>            <none>
centos-job-wmnrd   0/1     Pending             0          0s    <none>        <none>        <none>            <none>
centos-job-cgtjt   0/1     Pending             0          0s    <none>        k8s-worker2   <none>            <none>
centos-job-wmnrd   0/1     Pending             0          0s    <none>        k8s-worker1   <none>            <none>
centos-job-2rjjn   0/1     Completed           0          22s   10.244.2.12   k8s-worker2   <none>            <none>
centos-job-cgtjt   0/1     ContainerCreating   0          0s    <none>        k8s-worker2   <none>            <none>
centos-job-t2b4t   0/1     Completed           0          22s   10.244.1.5    k8s-worker1   <none>            <none>
centos-job-wmnrd   0/1     ContainerCreating   0          0s    <none>        k8s-worker1   <none>            <none>
centos-job-wmnrd   1/1     Running             0          1s    10.244.1.6    k8s-worker1   <none>            <none>
centos-job-cgtjt   1/1     Running             0          2s    10.244.2.13   k8s-worker2   <none>            <none>
centos-job-wmnrd   0/1     Completed           0          21s   10.244.1.6    k8s-worker1   <none>            <none>
centos-job-cgtjt   0/1     Completed           0          22s   10.244.2.13   k8s-worker2   <none>            <none>
centos-job-wmnrd   0/1     Completed           0          23s   10.244.1.6    k8s-worker1   <none>            <none>
centos-job-cgtjt   0/1     Completed           0          23s   10.244.2.13   k8s-worker2   <none>            <none>

centos-job-w766f   0/1     Pending             0          0s    <none>        <none>        <none>            <none>
centos-job-w766f   0/1     Pending             0          0s    <none>        k8s-worker2   <none>            <none>
centos-job-cgtjt   0/1     Completed           0          23s   10.244.2.13   k8s-worker2   <none>            <none>
centos-job-wmnrd   0/1     Completed           0          23s   10.244.1.6    k8s-worker1   <none>            <none>
centos-job-w766f   0/1     ContainerCreating   0          0s    <none>        k8s-worker2   <none>            <none>
centos-job-w766f   1/1     Running             0          2s    10.244.2.14   k8s-worker2   <none>            <none>
centos-job-w766f   0/1     Completed           0          22s   10.244.2.14   k8s-worker2   <none>            <none>
centos-job-w766f   0/1     Completed           0          23s   10.244.2.14   k8s-worker2   <none>            <none>
centos-job-w766f   0/1     Completed           0          24s   10.244.2.14   k8s-worker2   <none>            <none>

[root@k8s-master ~]# kubectl  delete jobs.batch centos-job
job.batch "centos-job" deleted from default namespace
```

### Job 실습 — activeDeadlineSeconds

```bash
[root@k8s-master ~]# vi job-exam.yaml
```
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: centos-job
spec:
#  completions: 5			# 주석 처리
#  parallelism: 2			# 주석 처리
  activeDeadlineSeconds: 10		# 주석 삭제
  template:
    spec:
      containers:
      - name: centos-container
        image: centos:7
        command: ["bash"]
        args:
        - "-c"
        - "date; echo 'Hello World'; sleep 20; echo 'Bye'"
#      restartPolicy: Never
      restartPolicy: OnFailure
  backoffLimit: 3
```

- activeDeadlineSeconds: 10 : Job이 실행되는 최대시간을 초단위로 제한하는 옵션. job이 실행된 후 10초가 지나면 실행중인 pod들을 강제 종료

```bash
[root@k8s-master ~]# kubectl  apply  -f  job-exam.yaml
job.batch/centos-job created

[root@k8s-master ~]# kubectl  get  pods  -o  wide  --watch
NAME               READY   STATUS              RESTARTS   AGE   IP            NODE          NOMINATED NODE   READINESS GATES
centos-job-d8q4z   0/1     Pending             0          0s    <none>        <none>        <none>            <none>
centos-job-d8q4z   0/1     Pending             0          0s    <none>        k8s-worker2   <none>            <none>
centos-job-d8q4z   0/1     ContainerCreating   0          0s    <none>        k8s-worker2   <none>            <none>
centos-job-d8q4z   1/1     Running             0          1s    10.244.2.15   k8s-worker2   <none>            <none>
centos-job-d8q4z   1/1     Terminating         0          10s   10.244.2.15   k8s-worker2   <none>            <none>
centos-job-d8q4z   1/1     Terminating         0          10s   10.244.2.15   k8s-worker2   <none>            <none>
centos-job-d8q4z   1/1     Terminating         0          10s   10.244.2.15   k8s-worker2   <none>            <none>
```

정리: Job은 **completions**·**parallelism**·**backoffLimit**·`activeDeadlineSeconds` 조합으로 단발성 배치 작업의 성공/실패와 재시도를 제어한다. 이 Job을 정해진 시간마다 자동으로 만들어주는 것이 다음의 **CronJob**이다.

---

## CronJob

- 실무에서 배치 작업은 대부분 정기적으로 실행된다.
  - 매일 새벽 2시 DB 백업
  - 10분마다 로그 정리
  - 매주 통계 집계
  - 매일 캐시 초기화

- 이 작업들의 공통적인 특징은 다음과 같다.
  - 항상 실행 중일 필요가 없다.
  - 사용자의 요청을 기다리는 서버 프로그램이 아니다.
  - 정해진 시간에 실행된다.
  - 작업을 수행하고 종료된다.

- 리눅스에서는 이러한 정기적인 작업을 cron으로 처리한다.
- 쿠버네티스에서는 cron과 같은 역할을 하는 전용 리소스가 **CronJob**이다.
- CronJob은 지정된 일정(Schedule)에 따라 Job을 자동으로 생성하는 컨트롤러다.
- 중요한 점은 CronJob이 실제 작업을 직접 수행하는 것이 아니라는 것이다.

CronJob은
- 백업 작업을 직접 수행하지 않는다.
- 스크립트를 직접 실행하지 않는다.
- 컨테이너를 직접 실행하지 않는다.
- Pod를 직접 관리하여 작업을 수행하는 역할도 하지 않는다.

- CronJob이 하는 일은 딱 하나다.
  - 지금 시간이 됐으니까 Job 하나 만들어야겠다.

- 실제 작업은 항상 Job이 수행한다. 그래서 실행 구조는 항상 다음과 같다.
- 정해진 시간이 됨 --> Job 생성 --> Job이 Pod 생성 --> Pod에서 컨테이너 실행 --> 작업 수행 --> 작업 종료 --> Job 완료
- 따라서 CronJob은 "언제 실행할 것인가"를 관리하고, Job은 "무슨 작업을 수행할 것인가"를 관리한다.

실행 구조는 다음과 같다.
1) CronJob --> 2) 정해진 시간이 됨 --> 3) Job 생성 --> 4) Pod 생성 --> 5) Container 실행 --> 6) 작업 수행 --> 7) Container 종료 --> 8) Pod 완료 --> 9) Job 완료

### CronJob의 전체 동작 흐름 (매일 새벽 2시에 백업하는 CronJob)

1단계 : 관리자가 CronJob을 생성한다.
2단계 : 쿠버네티스는 이 CronJob을 기억하고 대기한다. (아직 아무 작업도 실행하지 않는다)
3단계 : 시간이 새벽 2시가 된다.
4단계 : CronJob Controller가 판단한다. (실행 시간이 됐다.)
5단계 : CronJob Controller가 새 Job 하나를 생성한다.
6단계 : Job Controller가 Pod를 생성한다.
7단계 : Pod 안 컨테이너에서 백업 스크립트가 실행된다.
8단계 : 작업이 끝나면 exit 0(성공) 또는 exit 1(실패)
9단계 : Job은 성공 또는 실패 상태로 남는다.

- 다음 날 새벽 2시가 되면 또 다른 새로운 Job이 생성된다.
  - CronJob은 매번 같은 Job을 재사용하지 않는다.
  - 항상 새로운 Job을 생성한다.

CronJob과 Job의 차이
- Job : 지금 당장 한 번 실행
- CronJob : 이 Job을 언제 실행할지 예약

- Job은 실행 담당
- CronJob은 시간 담당
- 배치 작업 로직은 Job에 있고, 정기 실행은 CronJob이 관리한다.

### CronJob 스케줄 문법 (cron 표현식)

- CronJob은 리눅스 cron과 완전히 같은 형식을 쓴다.

Cronjob Schedule: `"30 2 1 * *"`
- Minutes (from 0 to 59)
- Hours (from 0 to 23)
- Day of the month (from 1 to 31)
- Month (from 1 to 12)
- Day of the week (from 0 to 6)

기본 형식 : 분 시 일 월 요일

- `0 2 * * *` : 매일 새벽 2시
- `*/5 * * * *` : 5분마다
- `0 0 * * 0` : 매주 일요일 0시

- 쿠버네티스 CronJob은 서버 시간 기준이므로 타임존 설정이 매우 중요하다.

### CronJob 주요 옵션 개념 설명

(1) **schedule** : 언제 실행할지 결정. 예) `schedule: "*/5 * * * *"` : 5분마다 Job 생성

(2) **jobTemplate** : CronJob이 만들 Job의 템플릿. CronJob 안에는 Job 정의가 들어 있다

(3) **successfulJobsHistoryLimit** : 예) `successfulJobsHistoryLimit: 3` : 성공한 Job을 최대 3개까지 보관

(4) **failedJobsHistoryLimit** : 예) `failedJobsHistoryLimit: 2` : 실패한 Job을 최대 2개까지 보관

- 이 옵션이 없으면 Job이 계속 쌓여서 관리가 어려워질 수 있다

Job과 CronJob 문법 비교

```yaml
# apiVersion: batch/v1              |  apiVersion: batch/v1beta1
# kind: Job                         |  kind: CronJob
metadata:                           |  metadata:
  name: centos-job                  |    name: cronjob-definition
spec:                                |  spec:
                                     |    schedule: "0 3 1 * *"
                                     |    jobTemplate:
                                     |      spec:
  template:                         |        template:
    spec:                           |          spec:
      containers:                   |            containers:
      - name: hello-busybox         |            - name: hello-busybox
        image: busybox               |              image: busybox
        args:                        |              args:
        - /bin/sh                   |              - /bin/sh
        - "-c"                      |              - -c
        - "echo Hello; sleep 5; echo Bye"  |        - "echo Hello; sleep 5; echo Bye"
      restartPolicy: Never          |            restartPolicy: Never
```

### CronJob 실습 — concurrencyPolicy: Forbid

```bash
[root@k8s-master ~]# vi cronjob-exam.yaml
```
```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: cronjob-exam

spec:
  schedule: "* * * * *"
  startingDeadlineSeconds: 500
  concurrencyPolicy: Forbid
  successfulJobsHistoryLimit: 3
  failedJobsHistoryLimit: 2

  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: hello-busybox
            image: busybox
            args:
            - /bin/sh
            - -c
            - date; echo Hello; sleep 10; echo Bye; date
          restartPolicy: Never
```

- **concurrencyPolicy: Allow**
  - default로 적용
  - 이전 Job이 아직 실행 중이어도 스케줄 시간이 되면 새 Job을 또 생성
  - 동시에 여러 Job 실행 가능
  - 로그 수집, 통계 집계, 단순 스크립트, 서로 영향을 주지 않는 배치

동작 개념
```
03:00 Job A 시작 (아직 실행 중)
03:01 Job B 시작 (아직 실행 중)
03:02 Job C 시작
```

- **concurrencyPolicy: Forbid**
  - 이전 Job이 아직 실행 중이면 다음 스케줄의 Job은 아예 실행하지 않음
  - 항상 1개 Job만 실행 (중복 실행 완전 차단)
  - 데이터 무결성 보장에 유리, 파일 이동/삭제, 상태 변경 작업

동작 개념
```
03:00 Job A 시작 (실행 중)
03:01 스케줄 도착 : 실행 안 함
03:02 스케줄 도착 : 실행 안 함
03:05 Job A 종료
03:06 다음 스케줄 : Job B 시작
```

```bash
[root@k8s-master ~]# kubectl  apply  -f  cronjob-exam.yaml
cronjob.batch/cronjob-exam created (dry run)

[root@k8s-master ~]# kubectl  get  pods  -o  wide  --watch
NAME                          READY   STATUS              RESTARTS   AGE   IP            NODE          NOMINATED NODE   READINESS GATES
cronjob-exam-29786643-xnhrp   0/1     Pending             0          0s    <none>        <none>        <none>            <none>
cronjob-exam-29786643-xnhrp   0/1     Pending             0          0s    <none>        k8s-worker2   <none>            <none>
cronjob-exam-29786643-xnhrp   0/1     ContainerCreating   0          0s    <none>        k8s-worker2   <none>            <none>
cronjob-exam-29786643-xnhrp   1/1     Running             0          3s    10.244.2.17   k8s-worker2   <none>            <none>
cronjob-exam-29786643-xnhrp   0/1     Completed           0          13s   10.244.2.17   k8s-worker2   <none>            <none>
cronjob-exam-29786643-xnhrp   0/1     Completed           0          14s   10.244.2.17   k8s-worker2   <none>            <none>
cronjob-exam-29786643-xnhrp   0/1     Completed           0          15s   10.244.2.17   k8s-worker2   <none>            <none>
cronjob-exam-29786644-g5dp8   0/1     Pending             0          0s    <none>        <none>        <none>            <none>
cronjob-exam-29786644-g5dp8   0/1     Pending             0          0s    <none>        k8s-worker2   <none>            <none>
cronjob-exam-29786644-g5dp8   0/1     ContainerCreating   0          0s    <none>        k8s-worker2   <none>            <none>
cronjob-exam-29786644-g5dp8   1/1     Running             0          3s    10.244.2.18   k8s-worker2   <none>            <none>
cronjob-exam-29786644-g5dp8   0/1     Completed           0          13s   10.244.2.18   k8s-worker2   <none>            <none>
cronjob-exam-29786644-g5dp8   0/1     Completed           0          14s   10.244.2.18   k8s-worker2   <none>            <none>
cronjob-exam-29786644-g5dp8   0/1     Completed           0          15s   10.244.2.18   k8s-worker2   <none>            <none>
		~~~~~~~~~~~~~~ 중간 생략 ~~~~~~~~~~~~~~

[root@k8s-master ~]# kubectl  get  pods
NAME                          READY   STATUS      RESTARTS   AGE
cronjob-exam-29786646-qfskv   0/1     Completed   0          3m3s
cronjob-exam-29786647-sd54w   0/1     Completed   0          2m3s
cronjob-exam-29786648-zv5w6   0/1     Completed   0          63s
cronjob-exam-29786649-zhkv4   1/1     Running     0          3s

[root@k8s-master ~]# kubectl  get  cronjobs.batch
NAME           SCHEDULE      TIMEZONE   SUSPEND   ACTIVE   LAST SCHEDULE   AGE
cronjob-exam   * * * * *     <none>     False     0        37s             8m25s

	# 성공적으로 완료된 Job을 최대 3개까지 보관한다
[root@k8s-master ~]# kubectl  get  jobs.batch
NAME                    STATUS     COMPLETIONS   DURATION   AGE
cronjob-exam-29786648   Complete   1/1           15s        2m44s
cronjob-exam-29786649   Complete   1/1           15s        104s
cronjob-exam-29786650   Complete   1/1           15s        44s

[root@k8s-master ~]# kubectl get pods
NAME                          READY   STATUS      RESTARTS   AGE
cronjob-exam-29786655-7kxj9   0/1     Completed   0          2m22s
cronjob-exam-29786656-74nxc   0/1     Completed   0          82s
cronjob-exam-29786657-8pzhx   0/1     Completed   0          22s

[root@k8s-master ~]# kubectl  logs cronjob-exam-29786655-7kxj9
Thu Aug 20 04:15:02 UTC 2026
Hello
Bye
Thu Aug 20 04:15:12 UTC 2026

[root@k8s-master ~]# kubectl  delete  cronjobs cronjob-exam
cronjob.batch "cronjob-exam" deleted from default namespace
```

### "concurrencyPolicy: Forbid" 옵션으로 CronJob 확인

- **concurrencyPolicy: Forbid**
  - 이전 Job이 아직 실행 중이면 다음 스케줄의 Job은 실행하지 않음
  - 항상 1개 Job만 실행 (중복 실행 완전 차단)
  - 데이터 무결성 보장에 유리, 파일 이동/삭제, 상태 변경 작업

동작 개념
```
03:00 Job A 시작 (실행 중)
03:01 스케줄 도착 : 실행 안 함
03:02 스케줄 도착 : 실행 안 함
03:05 Job A 종료
03:06 다음 스케줄 : Job B 시작
```

```bash
[root@k8s-master ~]# vi cronjob-exam.yaml
```
```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: cronjob-exam

spec:
  schedule: "* * * * *"
  startingDeadlineSeconds: 500
  concurrencyPolicy: Forbid
  successfulJobsHistoryLimit: 3
  failedJobsHistoryLimit: 2

  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: hello-busybox
            image: busybox
            args:
            - /bin/sh
            - -c
            - date; echo Hello; sleep 100; echo Bye; date
          restartPolicy: Never
```

```bash
[root@k8s-master ~]# kubectl  apply  -f  cronjob-exam.yaml
cronjob.batch/cronjob-exam created

[root@k8s-master ~]# kubectl  get  pods  -o  wide  --watch
NAME                          READY   STATUS              RESTARTS   AGE   IP            NODE          NOMINATED NODE   READINESS GATES
cronjob-exam-29786736-z84jv   0/1     Pending             0          0s    <none>        <none>        <none>            <none>
cronjob-exam-29786736-z84jv   0/1     Pending             0          0s    <none>        k8s-worker2   <none>            <none>
cronjob-exam-29786736-z84jv   0/1     ContainerCreating   0          0s    <none>        k8s-worker2   <none>            <none>
cronjob-exam-29786736-z84jv   1/1     Running             0          2s    10.244.2.31   k8s-worker2   <none>            <none>

	# 아직 Pod가 실행중이므로 상태가 Running으로 확인
[root@k8s-master ~]# kubectl  get  pods
NAME                          READY   STATUS    RESTARTS   AGE
cronjob-exam-29786736-z84jv   1/1     Running   0          31s

[root@k8s-master ~]# kubectl  get  pods  -o  wide  --watch
NAME                          READY   STATUS              RESTARTS   AGE    IP            NODE          NOMINATED NODE   READINESS GATES
cronjob-exam-29786736-z84jv   0/1     Pending             0          0s     <none>        <none>        <none>            <none>
cronjob-exam-29786736-z84jv   0/1     Pending             0          0s     <none>        k8s-worker2   <none>            <none>
cronjob-exam-29786736-z84jv   0/1     ContainerCreating   0          0s     <none>        k8s-worker2   <none>            <none>
cronjob-exam-29786736-z84jv   1/1     Running             0          2s     10.244.2.31   k8s-worker2   <none>            <none>
cronjob-exam-29786736-z84jv   0/1     Completed           0          102s   10.244.2.31   k8s-worker2   <none>            <none>
cronjob-exam-29786736-z84jv   0/1     Completed           0          103s   10.244.2.31   k8s-worker2   <none>            <none>
cronjob-exam-29786736-z84jv   0/1     Completed           0          104s   10.244.2.31   k8s-worker2   <none>            <none>

	# 작업이 완료되었기때문에 Completed 상태가 된다.
[root@k8s-master ~]# kubectl  get  pods
NAME                          READY   STATUS      RESTARTS   AGE
cronjob-exam-29786736-z84jv   0/1     Completed   0          2m41s
cronjob-exam-29786737-gr978   1/1     Running     0          57s

[root@k8s-master ~]# kubectl  logs  cronjob-exam-29786736-z84jv
Thu Aug 20 05:36:02 UTC 2026
Hello
Bye
Thu Aug 20 05:37:42 UTC 2026

[root@k8s-master ~]# kubectl  logs  cronjob-exam-29786737-gr978
Thu Aug 20 05:37:46 UTC 2026
Hello
Bye
Thu Aug 20 05:39:26 UTC 2026
```

### "concurrencyPolicy: Allow" 옵션으로 CronJob 확인

- **concurrencyPolicy: Allow**
  - default로 적용
  - 이전 Job이 아직 실행 중이어도 스케줄 시간이 되면 새 Job을 또 생성
  - 동시에 여러 Job 실행 가능
  - 로그 수집, 통계 집계, 단순 스크립트, 서로 영향을 주지 않는 배치

동작 개념
```
03:00 Job A 시작 (아직 실행 중)
03:01 Job B 시작 (아직 실행 중)
03:02 Job C 시작
```

```bash
[root@k8s-master ~]# vi cronjob-exam.yaml
```
```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: cronjob-exam

spec:
  schedule: "* * * * *"
  startingDeadlineSeconds: 500
#  concurrencyPolicy: Forbid		# 주석 처리
  concurrencyPolicy: Allow		# 추가 설정
  successfulJobsHistoryLimit: 3
  failedJobsHistoryLimit: 2

  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: hello-busybox
            image: busybox
            args:
            - /bin/sh
            - -c
            - date; echo Hello; sleep 100; echo Bye; date
          restartPolicy: Never
```

```bash
[root@k8s-master ~]# kubectl  apply  -f  cronjob-exam.yaml  --dry-run=client
cronjob.batch/cronjob-exam created (dry run)

[root@k8s-master ~]# kubectl  apply  -f  cronjob-exam.yaml
cronjob.batch/cronjob-exam created

	# Allow이기 때문에 첫 번째 Job이 실행중인데도 두 번째 Job도 생성된다.
[root@k8s-master ~]# kubectl get pods
NAME                          READY   STATUS    RESTARTS   AGE
cronjob-exam-29786750-xjmxr   1/1     Running   0          64s	# 첫 번째 Job (Job 실행중)
cronjob-exam-29786751-btsg6   1/1     Running   0          4s	# 두 번째 Job

[root@k8s-master ~]# kubectl  get  pods  -o  wide  --watch
NAME                          READY   STATUS              RESTARTS   AGE   IP            NODE          NOMINATED NODE   READINESS GATES
cronjob-exam-29786750-xjmxr   0/1     Pending             0          0s    <none>        <none>        <none>            <none>
cronjob-exam-29786750-xjmxr   0/1     Pending             0          0s    <none>        k8s-worker2   <none>            <none>
cronjob-exam-29786750-xjmxr   0/1     ContainerCreating   0          0s    <none>        k8s-worker2   <none>            <none>
cronjob-exam-29786750-xjmxr   1/1     Running             0          2s    10.244.2.37   k8s-worker2   <none>            <none>
cronjob-exam-29786751-btsg6   0/1     Pending             0          0s    <none>        <none>        <none>            <none>
cronjob-exam-29786751-btsg6   0/1     Pending             0          0s    <none>        k8s-worker1   <none>            <none>
cronjob-exam-29786751-btsg6   0/1     ContainerCreating   0          0s    <none>        k8s-worker1   <none>            <none>
cronjob-exam-29786751-btsg6   1/1     Running             0          2s    10.244.1.7    k8s-worker1   <none>            <none>
```

- 첫번째 작업이 Running 중인데도 새로운 파드(ContainerCreating)가 생성된다.

### EX) 내부 서비스 HTTP 헬스 체크를 CronJob으로 자동화하고 실패 Job을 근거로 장애 징후를 판단

- CronJob이 만든 Job이 성공/실패로 명확히 갈리는 흐름을 만든다
- 서비스가 정상일 때는 성공, 비정상일 때는 실패가 나는 것을 직접 만든다

**Deployment, Service 생성**

```bash
[root@k8s-master ~]# vi job-deploy-nginx.yaml
```
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hc-nginx-ctr
spec:
  replicas: 3
  selector:
    matchLabels:
      app: hc-nginx
  template:
    metadata:
      labels:
        app: hc-nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.31
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: hc-nginx-svc
spec:
  selector:
    app: hc-nginx		# app: hc-nginx  label을 갖은 Pod를 하나의 Service로 묶어서 단일 진입점을 제공한다.
  ports:
  - port: 80
    targetPort: 80
```

```bash
[root@k8s-master ~]# kubectl  apply  -f  job-deploy-nginx.yaml
deployment.apps/hc-nginx-ctr created
service/hc-nginx-svc created

[root@k8s-master ~]# kubectl  get  pods
NAME                              READY   STATUS    RESTARTS   AGE
hc-nginx-ctr-7699b6f665-9fj9p     1/1     Running   0          8m28s
hc-nginx-ctr-7699b6f665-n4z9z     1/1     Running   0          8m28s
hc-nginx-ctr-7699b6f665-shqws     1/1     Running   0          8m28s
```

```bash
[root@k8s-master ~]# vi cron-http-check.yaml
```
```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: cron-http-check
spec:
  schedule: "* * * * *"
  jobTemplate:
    spec:
      backoffLimit: 0
      template:
        spec:
          containers:
          - name: http-check
            image: curlimages/curl:8.5.0
            command:
            - sh
            - -c
            - |
              curl -sSf http://hc-nginx-svc.default.svc.cluster.local >/dev/null
          restartPolicy: Never
```

- -s(silent) : 출력과 진행 표시를 숨김. 정상일 때 아무 로그도 안 남김
- -S (show-error) : -s와 같이 쓸 때 실패 에러 메시지만 출력
- -f (fail) : HTTP 응답 코드가 4xx / 5xx면 실패 처리. exit code를 0이 아닌 값으로 반환
- `>/dev/null` : 정상 응답 바디(HTML 등)를 버림. 상태만 체크

```bash
[root@k8s-master ~]# kubectl  apply  -f  cron-http-check.yaml
cronjob.batch/cron-http-check created

	# http server로부터 200OK 정보를 받기때문에 Completed로 확인
[root@k8s-master ~]# kubectl  get  pods
NAME                              READY   STATUS      RESTARTS   AGE
cron-http-check-29786775-h27fw    0/1     Completed   0          95s
cron-http-check-29786776-gl8m7    0/1     Completed   0          35s
hc-nginx-ctr-7699b6f665-9fj9p     1/1     Running     0          13m
hc-nginx-ctr-7699b6f665-n4z9z     1/1     Running     0          13m
hc-nginx-ctr-7699b6f665-shqws     1/1     Running     0          13m

	# 강제로 Fail 발생
[root@k8s-master ~]# kubectl  scale  deployment hc-nginx-ctr  --replicas=0

[root@k8s-master ~]# kubectl  get  pods  -o  wide  --watch
NAME                              READY   STATUS              RESTARTS   AGE     IP            NODE          NOMINATED NODE   READINESS GATES
cron-http-check-29786777-kn7k9    0/1     Completed           0          2m24s   10.244.1.14   k8s-worker1   <none>            <none>
cron-http-check-29786778-4zwlw    0/1     Completed           0          84s     10.244.1.15   k8s-worker1   <none>            <none>
cron-http-check-29786779-jgzfz    0/1     Completed           0          24s     10.244.1.16   k8s-worker1   <none>            <none>
cron-http-check-29786780-6m5s2    0/1     Pending             0          0s      <none>        <none>        <none>            <none>
cron-http-check-29786780-6m5s2    0/1     Pending             0          0s      <none>        k8s-worker2   <none>            <none>
cron-http-check-29786780-6m5s2    0/1     ContainerCreating   0          0s      <none>        k8s-worker2   <none>            <none>
cron-http-check-29786780-6m5s2    0/1     Error               0          7s      10.244.2.44   k8s-worker2   <none>            <none>
cron-http-check-29786780-6m5s2    0/1     Error               0          8s      10.244.2.44   k8s-worker2   <none>            <none>
cron-http-check-29786780-6m5s2    0/1     Error               0          9s      10.244.2.44   k8s-worker2   <none>            <none>

[root@k8s-master ~]# kubectl  get  pods
NAME                              READY   STATUS      RESTARTS   AGE
cron-http-check-29786777-kn7k9    0/1     Completed   0          4m
cron-http-check-29786778-4zwlw    0/1     Completed   0          3m
cron-http-check-29786779-jgzfz    0/1     Completed   0          2m
cron-http-check-29786780-6m5s2    0/1     Error       0          60s
```

- 서비스(hc-nginx-ctr)를 replicas=0으로 줄여 백엔드가 없어지자, curl 헬스체크가 실패(Error)로 바뀐다. CronJob이 만든 Job의 성공/실패 이력만 보고도 서비스 장애 시점을 판단할 수 있다.

정리: CronJob은 **schedule**로 정한 시간마다 Job을 새로 만들 뿐 실제 작업은 하지 않으며, **concurrencyPolicy**(Allow/Forbid/Replace)로 동시 실행 여부를 제어한다. 이처럼 ReplicationController부터 CronJob까지 모든 Controller는 "현재 상태를 감시하고 원하는 상태로 맞춘다"는 동일한 Reconciliation Loop 원리 위에서, 각자의 목적(개수 유지, 무중단 배포, 노드당 배치, 상태 저장, 단발성/주기성 작업)에 맞게 특화되어 있다.

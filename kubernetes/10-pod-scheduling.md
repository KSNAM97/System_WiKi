# Pod Scheduling

> Pod를 어느 Node에 배치할지 kube-scheduler가 결정하는 과정과, nodeSelector·Affinity/Anti-Affinity·Taint & Toleration·Cordon/Drain으로 배치를 직접 제어하는 방법을 정리한다.

## 목차

1. [Pod Scheduling 개념](#pod-scheduling-개념)
2. [스케줄러가 Node를 선택하는 방식](#스케줄러가-node를-선택하는-방식)
3. [배치 가능한 Node의 조건](#배치-가능한-node의-조건)
4. [스케줄링을 직접 제어하는 대표 방법 4가지](#스케줄링을-직접-제어하는-대표-방법-4가지)
5. [nodeSelector](#nodeselector)
6. [Affinity / Anti-Affinity란](#affinity--anti-affinity란)
7. [Node Affinity](#node-affinity)
8. [Pod Affinity / Anti-Affinity](#pod-affinity--anti-affinity)
9. [Node Affinity 실습](#node-affinity-실습)
10. [Pod Affinity 실습](#pod-affinity-실습)
11. [Pod Anti-Affinity 실습](#pod-anti-affinity-실습)
12. [Taint & Toleration이란](#taint--toleration이란)
13. [Taint의 기본 구조와 Effect](#taint의-기본-구조와-effect)
14. [Toleration이란](#toleration이란)
15. [Taint와 Node Affinity의 차이](#taint와-node-affinity의-차이)
16. [Taint & Toleration 실습](#taint--toleration-실습)
17. [Cordon & Drain이란](#cordon--drain이란)
18. [Cordon과 Drain의 차이](#cordon과-drain의-차이)
19. [Cordon & Drain 실습](#cordon--drain-실습)

---

## Pod Scheduling 개념

Pod Scheduling은 Pod를 어느 Node(서버)에 올려서 실행할지 결정하는 과정이다. Pod는 그냥 만들어진다고 바로 실행되지 않으며, 반드시 배치될 Node가 결정되어야 한다. 클러스터 운영 중 자원이 부족하거나 특정 조건을 만족하는 Node가 없을 때 이 결정 과정에서 문제가 생길 수 있어, 이때는 이 결정을 담당하는 kube-scheduler(스케줄러)의 동작 방식을 이해해야 한다.

- Pod = 실행할 프로그램 묶음
- Node = Pod가 실제로 올라가서 실행되는 서버(워커 서버)
- Scheduler = Pod를 어느 Node에 올릴지 선택하는 관리자

쿠버네티스에서 Pod가 계속 Pending(대기) 상태로 남는 경우가 있다. Pending은 Pod가 아직 실행 상태가 되지 않았다는 의미다. 스케줄링 대상 Pod라면 배치할 적절한 Node를 찾지 못한 경우가 대표적인 원인이다. 하지만 이미지 다운로드, 볼륨 연결 등 다른 이유로도 Pending 상태가 유지될 수 있으므로 Pending이 항상 스케줄링 실패를 의미하는 것은 아니다.

Scheduling이 실제로 진행되는 흐름은 다음과 같다.

```
Pod 생성 --> API Server에 Pod 정보 저장 --> kube-scheduler가 Node 선택 --> Pod가 해당 Node에 Bind --> 해당 Node의 kubelet이 Pod 실행
```

**정리**: 스케줄링에서 제일 중요한 것은 스케줄러가 Pod를 배치할 적절한 Node를 선택하는 과정이다.

## 스케줄러가 Node를 선택하는 방식

스케줄러는 단순히 랜덤으로 Node를 고르지 않는다.

1. Pod를 올릴 수 없는 Node를 후보에서 제거한다. (Filtering)
2. 남은 Node의 적합성을 평가한다. (Scoring)
3. 점수가 높은 Node를 최종적으로 선택한다. (Binding)

Filtering에서는 Pod의 요구사항을 만족하지 못하는 Node를 후보에서 제외한다. Filtering 결과 배치 가능한 Node가 0개라면 해당 Pod는 스케줄링되지 못하고 Pending 상태가 유지될 수 있다. 배치 가능한 Node가 여러 개라면 Scoring을 통해 어느 Node가 더 적절한지 판단한다.

즉, 스케줄링은 단순히 "Node 하나를 선택하는 과정"이 아니라 배치할 수 없는 Node를 제거하고 남은 Node의 적합도를 비교한 후 최종 Node를 선택하는 과정이다.

## 배치 가능한 Node의 조건

스케줄러가 Node를 후보에서 제거할 때 확인하는 대표적인 조건은 다음과 같다.

**1) 자원이 충분한가 (CPU, 메모리)**
Pod에는 requests(요청 자원)가 설정될 수 있다. 스케줄러는 Pod의 requests를 기준으로 Node에 해당 Pod를 배치할 수 있는지 판단한다. Node에 Pod가 요청한 만큼의 allocatable 자원이 충분하지 않으면 해당 Node는 후보에서 제외된다. requests가 클수록 배치할 수 있는 Node가 줄어들 수 있다.

예시: Pod requests가 `cpu 1, memory 1Gi`이고 Node에 배치 가능한 여유 자원이 `cpu 0.5, memory 2Gi`라면 CPU 부족으로 그 Node는 후보에서 제외된다.

**2) Node 상태가 정상인가**
일반적으로 Node가 Ready 상태가 아니면 정상적인 Pod 배치가 어렵다. Node가 cordon 상태라면 새 Pod를 해당 Node에 배치하지 않는다. cordon은 해당 Node를 Scheduling 대상에서 제외하는 것으로, 기존에 실행 중인 Pod는 그대로 유지될 수 있고 새 Pod는 다른 Node에 배치된다.

**3) Pod가 요구한 노드 조건을 만족하는가**
대표적으로 nodeSelector / node affinity가 있다. nodeSelector는 특정 라벨이 있는 Node에만 배치하고, node affinity는 Node의 라벨을 기준으로 강제 조건 또는 선호 조건으로 배치한다.

**4) Node가 Pod를 거부하고 있지는 않은가 (taint)**
taint는 특정 Node에 아무 Pod나 배치되지 못하도록 제한하는 기능이고, toleration은 특정 Pod가 해당 taint를 허용할 수 있도록 설정하는 기능이다. toleration이 없거나 taint를 만족하지 못하면 해당 Node는 후보에서 제외될 수 있다.

## 스케줄링을 직접 제어하는 대표 방법 4가지

**1) nodeSelector (가장 쉬움)** — 라벨이 있는 노드에만 배치

**2) Node Affinity (조건을 더 세밀하게)**
- required: 조건 불만족이면 절대 배치 안 됨 (Pending 가능)
- preferred: 가능하면 그쪽, 없으면 다른 노드도 가능
- weight: preferred 조건의 선호도를 나타내는 점수

**3) Pod Affinity/Anti-Affinity (분산 배치)**
- 특정 Pod와 같은 Node 또는 가까운 위치에 배치
- 특정 Pod와 같은 Node에 몰리지 않도록 분산 배치
- 고가용성 및 장애 분산에 활용

**4) taint & toleration (특정 노드 보호)**
- GPU Node, 중요한 Node 등 특정 Node를 일반 Pod가 사용하지 못하도록 제한
- 필요한 Pod만 toleration을 사용하여 해당 Node에 배치할 수 있도록 허용

## nodeSelector

nodeSelector는 Pod를 특정 라벨(label)을 가진 Node에만 배치하도록 강제하는 기능이다. Pod가 스케줄러에게 "이 라벨이 붙은 Node가 아니면 나는 실행되지 않아도 된다"라고 요구하는 것과 같다.

사용하는 이유는 모든 Node의 성능이 동일하지 않기 때문이다 (SSD 서버, GPU 서버, 테스트용 서버, 운영 전용 서버 등). 특정 Pod는 아무 Node에서나 실행되면 안 되는 경우가 있으며, nodeSelector는 이런 경우 Pod의 배치 위치를 명확히 제한한다.

기준은 Node의 라벨(label)이다. nodeSelector는 Node에 붙은 label(key=value)을 기준으로 동작하며, 라벨은 미리 Node에 설정되어 있어야 한다 (예: `disktype=ssd`, `env=prod`).

스케줄러가 nodeSelector를 처리하는 방식:

1. 모든 Node를 가져온다.
2. nodeSelector 조건을 확인한다.
3. 조건과 일치하지 않는 Node를 전부 제외한다.
4. 조건을 만족하는 Node만 배치 후보로 남긴다.

결과는 둘 중 하나다: 조건을 만족하는 Node가 있으면 그 Node에 Pod가 배치되고, 없으면 Pod는 Pending 상태로 유지된다. nodeSelector는 선호 조건이 아니라 강제 조건이다.

예시: Node A는 `disktype=hdd`, Node B는 `disktype=ssd`이고 Pod가 `nodeSelector: {disktype: ssd}`라면 Node A는 제외되고 Node B만 후보로 남아 Pod는 Node B에서 실행된다. Node B가 없다면 Pod는 Pending이 된다.

주의할 점: 라벨 key 또는 value가 조금이라도 다르면 매칭되지 않는다. nodeSelector를 사용하면 해당 라벨이 없는 Node에는 절대 배치되지 않는다. 여러 라벨을 쓰면 모두 만족해야 배치된다 (AND 조건).

### nodeSelector 실습

```bash
[root@k8s-master ~]# kubectl get nodes --show-labels
NAME          STATUS   ROLES           AGE   VERSION   LABELS
k8s-master    Ready    control-plane   14d   v1.35.7   ...
k8s-worker1   Ready    <none>          14d   v1.35.7   ...,disktype=ssd,env=prod,...
k8s-worker2   Ready    <none>          14d   v1.35.7   ...,disktype=gpu,env=dev,...

# 기존 Label 삭제
[root@k8s-master ~]# kubectl  label  node  k8s-worker{1,2} disktype-
node/k8s-worker1 unlabeled
node/k8s-worker2 unlabeled

[root@k8s-master ~]# kubectl  label  node  k8s-worker{1,2} env-
node/k8s-worker1 unlabeled
node/k8s-worker2 unlabeled

# 새 Label 설정
[root@k8s-master ~]# kubectl  label  nodes k8s-worker1  hardware=general
node/k8s-worker1 labeled

[root@k8s-master ~]# kubectl  label  nodes k8s-worker2  hardware=gpu
node/k8s-worker2 labeled

[root@k8s-master ~]# kubectl  get  nodes  -L  hardware
NAME          STATUS   ROLES           AGE   VERSION   HARDWARE
k8s-master    Ready    control-plane   14d   v1.35.7
k8s-worker1   Ready    <none>          14d   v1.35.7   general
k8s-worker2   Ready    <none>          14d   v1.35.7   gpu
```

```yaml
# gpu-pod-nodeselector.yaml
apiVersion: v1
kind: Pod
metadata:
  name: gpu-pod
spec:
  nodeSelector:
    hardware: gpu
  containers:
  - name: nginx-container
    image: nginx:1.31
```

```bash
[root@k8s-master ~]# kubectl  apply  -f  gpu-pod-nodeselector.yaml
pod/gpu-pod created

[root@k8s-master ~]# kubectl  get  pods  -o wide
NAME      READY   STATUS    RESTARTS   AGE   IP            NODE          NOMINATED NODE   READINESS GATES
gpu-pod   1/1     Running   0          34s   10.244.2.32   k8s-worker2   <none>            <none>
```

Deployment로 여러 Pod를 생성해도 `nodeSelector: {hardware: gpu}`가 설정되어 있으면 모두 같은 조건을 만족하는 Node(k8s-worker2)에 생성된다.

```yaml
# gpu-deploy-nodeselector.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: gpu-deploy
spec:
  replicas: 3
  selector:
    matchLabels:
      app: gpu-web
  template:
    metadata:
      labels:
        app: gpu-web
    spec:
      nodeSelector:
        hardware: gpu
      containers:
      - name: nginx
        image: nginx:1.31
```

```bash
[root@k8s-master ~]# kubectl  apply  -f  gpu-deploy-nodeselector.yaml
deployment.apps/gpu-deploy created

[root@k8s-master ~]# kubectl  get  pods  -o  wide
NAME                          READY   STATUS    RESTARTS   AGE     IP            NODE          NOMINATED NODE   READINESS GATES
gpu-deploy-8449f8f588-2tnpd   1/1     Running   0           5m5s   10.244.2.35   k8s-worker2   <none>            <none>
gpu-deploy-8449f8f588-djgj9   1/1     Running   0           5m5s   10.244.2.34   k8s-worker2   <none>            <none>
gpu-deploy-8449f8f588-jkg8k   1/1     Running   0           5m5s   10.244.2.33   k8s-worker2   <none>            <none>
gpu-pod                       1/1     Running   0           11m    10.244.2.32   k8s-worker2   <none>            <none>

[root@k8s-master ~]# kubectl  describe  pod  gpu-deploy-8449f8f588-2tnpd | grep Node-
Node-Selectors:              hardware=gpu
```

## Affinity / Anti-Affinity란

Affinity / Anti-Affinity는 Pod를 어떤 Node에 가깝게 또는 떨어지게 배치할지 정하는 규칙이다. Affinity는 특정 조건을 만족하는 대상과 가깝게 배치하도록 하는 개념이고, Anti-Affinity는 특정 조건을 만족하는 대상과 떨어지게 배치하도록 하는 개념이다.

nodeSelector가 특정 Node의 Label을 기준으로 단순하게 배치 위치를 제한한다면, Affinity / Anti-Affinity는 Node 또는 다른 Pod와의 관계를 이용하여 더 세밀하게 배치할 수 있다.

nodeSelector의 한계:
- 복잡한 조건 표현에 제한이 있음
- 선호(prefer) 표현 불가
- 특정 Pod와 같은 위치에 배치하는 조건 표현 불가
- 특정 Pod와 떨어져 배치하는 조건 표현 불가
- Node와 Pod 사이의 관계를 이용한 배치 표현 불가

Affinity의 종류는 크게 두 가지다.

1. **Node Affinity**: Node의 라벨을 기준으로 Pod를 어떤 Node에 배치할지 결정. nodeSelector보다 복잡하고 유연한 조건 설정 가능. required / preferred를 이용하여 강제 또는 선호 조건 설정 가능.
2. **Pod Affinity / Pod Anti-Affinity**: 다른 Pod와의 관계를 기준으로 배치. 특정 Pod와 같은 위치에 배치하거나 떨어뜨려 배치. 고가용성 및 Pod 분산 배치에 활용.

## Node Affinity

Node Affinity는 Pod를 특정 Node에 배치하고 싶은 조건을 더 자세하게 표현할 수 있는 nodeSelector의 확장이다. Node의 Label을 기준으로 어떤 Node에 배치할지 결정한다.

Node Affinity에는 두 가지 방식이 있다.

- **requiredDuringSchedulingIgnoredDuringExecution**: 조건을 반드시 만족해야 배치(강제 조건). 조건을 만족하지 않는 Node에는 배치하지 않음. 만족하는 Node가 없으면 Pod는 Pending 상태가 될 수 있음.
- **preferredDuringSchedulingIgnoredDuringExecution**: 가능하면 조건을 만족하는 Node로 배치(선호 조건). 조건을 만족하는 Node를 우선적으로 선호. 조건을 만족하는 Node가 없어도 다른 조건을 만족하는 Node에 배치 가능.

Node Affinity의 preferred 조건에서는 weight를 사용할 수 있다. weight는 1~100 사이의 값이며, 높을수록 해당 조건을 더 선호한다. weight가 높다고 해당 Node에 반드시 배치되는 것은 아니다.

`nodeSelectorTerms`는 조건 묶음(세트)으로 여러 개를 나열할 수 있고 묶음들 사이는 OR 관계다. `matchExpressions`는 한 묶음(term) 안에 들어가는 실제 조건들로, 같은 term 안에서는 AND 관계다.

```
예제 1) AND만 있는 경우(TERM이 1개)
nodeSelectorTerms:
- matchExpressions:
    A
    B
    C
논리식: A AND B AND C

예제 2) OR만 있는 경우(TERM이 여러 개, 각 term 조건 1개)
nodeSelectorTerms:
- matchExpressions:
  A
- matchExpressions:
  B
- matchExpressions:
  C
논리식: A OR B OR C

예제 3) AND 묶음 OR AND 묶음 (가장 흔한 형태)
nodeSelectorTerms:
- matchExpressions:
    A
    B
- matchExpressions:
    C
    D
논리식: (A AND B) OR (C AND D)
```

## Pod Affinity / Anti-Affinity

Pod Affinity / Anti-Affinity는 Node의 Label이 아니라 다른 Pod와의 관계를 기준으로 배치 위치를 결정한다.

**1) Pod Affinity** — 특정 Pod와 같은 Node 또는 지정한 토폴로지 영역에 같이 배치. 관련 Pod를 가까이 배치하여 네트워크 지연 등을 줄이는 데 활용. 예) 웹 Pod와 캐시 Pod를 같은 Node에 배치해 Pod 간 통신 거리를 줄인다.

**2) Pod Anti-Affinity** — 특정 Pod와 같은 Node 또는 지정한 토폴로지 영역에 배치하지 않도록 설정. 동일한 서비스의 Pod를 서로 다른 Node에 분산 배치할 때 사용해, 하나의 Node에 장애가 발생해도 다른 Node의 Pod가 서비스를 유지할 수 있도록 함. 예) 같은 app Pod를 서로 다른 Node에 분산 배치하여 특정 Node 장애 시 모든 Pod가 동시에 장애나는 상황을 줄인다.

강제 vs 선호 개념은 Node Affinity와 동일하게 적용된다.

1. **required(강제)**: 조건을 반드시 만족해야 함. 만족하는 Node가 없다면 Pod는 Pending 상태가 될 수 있음.
2. **preferred(선호)**: 가능하면 조건을 만족하는 곳에 배치. weight를 이용하여 선호도를 지정할 수 있음.

## Node Affinity 실습

**EX1) required 조건으로 특정 환경(env)을 반드시 만족하는 노드에만 Pod 배치하기**

```bash
[root@k8s-master ~]# kubectl  label  node  k8s-worker1  env=pord  disktype=ssd
node/k8s-worker1 labeled

[root@k8s-master ~]# kubectl  label  node  k8s-worker2  env=dev  disktype=hdd
node/k8s-worker2 labeled
```

```yaml
# step1-required.yaml
apiVersion: v1
kind: Pod
metadata:
  name: step1-required
spec:
  containers:
  - name: nginx
    image: nginx:1.31
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: env
            operator: In
            values: [prod]
```

```bash
[root@k8s-master ~]# kubectl  apply  -f  step1-required.yaml
pod/step1-required created

[root@k8s-master ~]# kubectl  get  pods  -o  wide
NAME             READY   STATUS    RESTARTS   AGE     IP            NODE          NOMINATED NODE   READINESS GATES
step1-required   1/1     Running   0          6m20s   10.244.1.28   k8s-worker1   <none>            <none>

[root@k8s-master ~]# kubectl  delete  pods  step1-required
pod "step1-required" deleted from default namespace
```

Deployment로 3개의 Pod를 생성해도 required 조건에 의해 모두 k8s-worker1에 생성된다.

```yaml
# step1-required-deploy.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: step1-deploy
spec:
  replicas: 3
  selector:
    matchLabels:
      app: webapp
  template:
    metadata:
      labels:
        app: webapp
    spec:
      containers:
      - name: nginx
        image: nginx:1.31
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: env
                operator: In
                values:
                - prod
```

```bash
[root@k8s-master ~]# kubectl  apply  -f  step1-required-deploy.yaml
deployment.apps/step1-deploy created

[root@k8s-master ~]# kubectl  get  pods  -o  wide
NAME                            READY   STATUS    RESTARTS   AGE   IP            NODE          NOMINATED NODE   READINESS GATES
step1-deploy-74979754fb-72w9x   1/1     Running   0          28s   10.244.1.30   k8s-worker1   <none>            <none>
step1-deploy-74979754fb-dczzw   1/1     Running   0          28s   10.244.1.31   k8s-worker1   <none>            <none>
step1-deploy-74979754fb-stvtd   1/1     Running   0          28s   10.244.1.29   k8s-worker1   <none>            <none>
```

**EX2) matchExpressions를 사용하여 여러 Node 조건을 AND 방식으로 동시에 적용해 Pod 배치하기**

```yaml
# step2-and-required.yaml (env=dev AND disktype=hdd)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: step2-deploy
spec:
  replicas: 3
  selector:
    matchLabels:
      app: webapp
  template:
    metadata:
      labels:
        app: webapp
    spec:
      containers:
      - name: nginx
        image: nginx:1.31
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: env
                operator: In
                values:
                - dev
              - key: disktype
                operator: In
                values:
                - hdd   # ssd이면 Pending 상태가 된다.
```

```bash
[root@k8s-master ~]# kubectl  apply  -f  step2-and-required.yaml
deployment.apps/step2-deploy created

[root@k8s-master ~]# kubectl  get  pods   -o  wide
NAME                           READY   STATUS    RESTARTS   AGE   IP            NODE          NOMINATED NODE   READINESS GATES
step2-deploy-864db8984-584g8   1/1     Running   0          19s   10.244.2.37   k8s-worker2   <none>            <none>
step2-deploy-864db8984-pz722   1/1     Running   0          19s   10.244.2.36   k8s-worker2   <none>            <none>
step2-deploy-864db8984-zf7h8   1/1     Running   0          19s   10.244.2.38   k8s-worker2   <none>            <none>
```

조건을 `disktype: ssd`로 바꾸면(k8s-worker2는 hdd이므로 AND 조건을 만족하는 Node가 없음) 3개의 Pod가 모두 Pending 상태가 된다.

```bash
[root@k8s-master ~]# kubectl  apply  -f  step2-and-required.yaml
deployment.apps/step2-deploy created

[root@k8s-master ~]# kubectl  get  pods  -o  wide
NAME                            READY   STATUS    RESTARTS   AGE   IP       NODE     NOMINATED NODE   READINESS GATES
step2-deploy-5d65846458-9k2dd   0/1     Pending   0          76s   <none>   <none>   <none>            <none>
step2-deploy-5d65846458-mdhjf   0/1     Pending   0          76s   <none>   <none>   <none>            <none>
step2-deploy-5d65846458-rdd4b   0/1     Pending   0          76s   <none>   <none>   <none>            <none>
```

**EX3) matchExpressions를 사용하여 여러 Node 조건을 OR 방식으로 동시에 적용해 Pod 배치하기**

```yaml
# step3-or-required.yaml (env=prod OR disktype=hdd)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: step3-deploy
spec:
  replicas: 3
  selector:
    matchLabels:
      app: webapp
  template:
    metadata:
      labels:
        app: webapp
    spec:
      containers:
      - name: nginx
        image: nginx:1.31
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: env
                operator: In
                values:
                - prod
            - matchExpressions:
              - key: disktype
                operator: In
                values:
                - hdd
```

```bash
[root@k8s-master ~]# kubectl  apply  -f  step3-or-required.yaml
deployment.apps/step3-deploy created

[root@k8s-master ~]# kubectl  get  pods  -o  wide
NAME                           READY   STATUS    RESTARTS   AGE   IP            NODE          NOMINATED NODE   READINESS GATES
step3-deploy-c87c87575-7gvw2   1/1     Running   0          30s   10.244.2.40   k8s-worker2   <none>            <none>
step3-deploy-c87c87575-9p79j   1/1     Running   0          30s   10.244.1.32   k8s-worker1   <none>            <none>
step3-deploy-c87c87575-rsjbp   1/1     Running   0          30s   10.244.2.39   k8s-worker2   <none>            <none>
```

OR 조건이므로 env=prod를 만족하는 k8s-worker1과 disktype=hdd를 만족하는 k8s-worker2 모두 후보가 되어 Pod가 두 Node에 나뉘어 배치된다.

**EX4) 강제 조건(required)과 선호 조건(preferred)을 함께 사용하는 스케줄링**

```bash
[root@k8s-master ~]# kubectl  label  nodes  k8s-worker1 env=prod  region=seoul
[root@k8s-master ~]# kubectl  label  nodes  k8s-worker2 env=prod  region=busan
[root@k8s-master ~]# kubectl  label  nodes  k8s-worker3 env=dev  region=seoul
```

```yaml
# step4-1-preferred.yaml
apiVersion: v1
kind: Pod
metadata:
  name: affinity-required-preferred
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: env
            operator: In
            values: [prod]
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 70
        preference:
          matchExpressions:
          - key: region
            operator: In
            values: [seoul]
  containers:
  - name: nginx
    image: nginx
```

| 노드 | required(env=prod) | preferred(region=seoul) | 점수 |
|------|---------------------|--------------------------|------|
| worker1 | 만족 | 만족 | +70 |
| worker2 | 만족 | 불만족 | +0 |
| worker3 | 불만족 | 만족 | 후보 제외 |

```bash
[root@k8s-master ~]# kubectl  apply  -f  step4-1-preferred.yaml
pod/affinity-required-preferred created

[root@k8s-master ~]# kubectl  get  pods  -o  wide
NAME                          READY   STATUS    RESTARTS   AGE   IP           NODE          NOMINATED NODE   READINESS GATES
affinity-required-preferred   1/1    Running   0          71s   10.244.1.4   k8s-worker1   <none>            <none>
```

required와 preferred를 동시에 만족하는 k8s-worker1(80점 아님, 여기선 70점)에 배치된다.

동일한 조건을 Deployment로 replicas 5개까지 확장해도 결과는 같다.

```yaml
# step4-2-preferred.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: affinity-required-preferred
spec:
  replicas: 5
  selector:
    matchLabels:
      app: nginx-affinity
  template:
    metadata:
      labels:
        app: nginx-affinity
    spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: env
                operator: In
                values: [prod]
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 70
            preference:
              matchExpressions:
              - key: region
                operator: In
                values: [seoul]
      containers:
      - name: nginx
        image: nginx:1.29.1
        ports:
        - containerPort: 80
```

```bash
[root@k8s-master ~]# kubectl  apply  -f  step4-2-preferred.yaml
deployment.apps/affinity-required-preferred created

[root@k8s-master ~]# kubectl  get  pods  -o  wide
NAME                                          READY   STATUS    RESTARTS   AGE   IP            NODE          NOMINATED NODE   READINESS GATES
affinity-required-preferred-7c478d5784-4vnjf  1/1     Running   0          3s    10.244.1.5    k8s-worker1   <none>            <none>
affinity-required-preferred-7c478d5784-fstvm  1/1     Running   0          3s    10.244.1.8    k8s-worker1   <none>            <none>
affinity-required-preferred-7c478d5784-vg2xb  1/1     Running   0          3s    10.244.1.7    k8s-worker1   <none>            <none>
affinity-required-preferred-7c478d5784-xqwld  1/1     Running   0          3s    10.244.1.6    k8s-worker1   <none>            <none>
affinity-required-preferred-7c478d5784-zgsd2  1/1     Running   0          3s    10.244.1.9    k8s-worker1   <none>            <none>

[root@k8s-master ~]# kubectl  delete  -f  step4-2-preferred.yaml
```

5개 Pod 모두 required+preferred 조건을 동시에 만족하는 k8s-worker1에 배치된다. 다음으로 여러 preferred 조건에 weight를 함께 부여하는 경우:

```yaml
# step4-3-preferred.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: affinity-multi-weight
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx-affinity-mw
  template:
    metadata:
      labels:
        app: nginx-affinity-mw
    spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: env
                operator: In
                values: [prod]
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 50
            preference:
              matchExpressions:
              - key: region
                operator: In
                values: [seoul]
          - weight: 30
            preference:
              matchExpressions:
              - key: disk
                operator: In
                values: [ssd]
      containers:
      - name: nginx
        image: nginx:1.29.1
        ports:
        - containerPort: 80
```

모든 후보 Node를 대상으로 각 preferred 조건을 검사하고, 조건을 만족할 때마다 weight를 더해 최종 점수가 가장 높은 Node를 선택한다.

- k8s-worker1: env=prod(required 통과) + region=seoul(+50) + disk=ssd(+30) = 총점 80
- k8s-worker2: env=prod(required 통과) + region=busan(+0) + disk=ssd(+30) = 총점 30
- k8s-worker3: env=prod(required 통과) + region=seoul(+50) + disk=hdd(+0) = 총점 50

우선순위는 worker1(80) > worker3(50) > worker2(30)이며, Pod는 가장 높은 점수인 k8s-worker1에 우선적으로 생성된다. worker1의 리소스가 부족해지면 다음으로 점수가 높은 worker3에 생성된다.

**EX5) Control-Plane(Master)을 임시 Worker Node로 활용**

```bash
# Control-Plane(Master)에서 Taint 확인
[root@k8s-master ~]# kubectl  describe  nodes  k8s-master  | grep  Taint
Taints:             node-role.kubernetes.io/control-plane:NoSchedule

# -i 옵션으로 대/소문자 구분 없이 확인
[root@k8s-master ~]# kubectl  describe  nodes  k8s-master  | grep -i  taint
Taints:             node-role.kubernetes.io/control-plane:NoSchedule
```

Control-Plane에는 Taint가 설정되어 있기 때문에 Pod가 생성되지 않는다(`control-plane:NoSchedule`). k8s-worker1/k8s-worker2는 Taint가 없다(`Taints: <none>`).

```bash
# Control-Plane(Master)의 Taint를 삭제
[root@k8s-master ~]# kubectl  taint  node  k8s-master  node-role.kubernetes.io/control-plane-
node/k8s-master untainted

[root@k8s-master ~]# kubectl  describe  nodes  k8s-master  | grep  Taint
Taints:             <none>
```

```yaml
# deploy-nginx-notaint.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deploy-nginx
spec:
  replicas: 5
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
        image: nginx:1.29.1
```

```bash
[root@k8s-master ~]# kubectl  apply  -f  deploy-nginx-notaint.yaml
deployment.apps/deploy-nginx created

[root@k8s-master ~]# kubectl  get  pods  -o  wide
NAME                           READY   STATUS    RESTARTS   AGE   IP            NODE          NOMINATED NODE   READINESS GATES
deploy-nginx-5c68c45fb6-9d8qr  1/1     Running   0          43s   10.244.0.28   k8s-master    <none>            <none>
deploy-nginx-5c68c45fb6-kkch2  1/1     Running   0          43s   10.244.1.13   k8s-worker1   <none>            <none>
deploy-nginx-5c68c45fb6-ps2d8  1/1     Running   0          43s   10.244.1.14   k8s-worker1   <none>            <none>
deploy-nginx-5c68c45fb6-zkvlh  1/1     Running   0          43s   10.244.2.5    k8s-worker2   <none>            <none>
deploy-nginx-5c68c45fb6-zsvkx  1/1     Running   0          43s   10.244.2.4    k8s-worker2   <none>            <none>
```

Taint를 제거하면 Master에도 Pod가 배치된다. 리소스 부족으로 Worker Node를 추가할 수 없는 경우 Master를 임시 Worker Node로 사용할 수 있다.

```bash
# Control-Plane(Master)에 다시 Taint 설정 (Taint : Key=Value:Effect)
[root@k8s-master ~]# kubectl  taint  node  k8s-master  node-role.kubernetes.io/control-plane:NoSchedule
node/k8s-master tainted
```

Taint의 원본 설정은 `node-role.kubernetes.io/control-plane:NoSchedule` (Key:Effect)이며, Taint는 Value를 생략할 수 있다.

```bash
[root@k8s-master ~]# kubectl  describe  nodes  k8s-master  | grep Taint
Taints:             node-role.kubernetes.io/control-plane:NoSchedule

# 다시 Taint를 설정해도 기존에 생성된 Pod는 삭제되거나 이동되지 않는다
[root@k8s-master ~]# kubectl  get  pods  -o  wide
NAME                           READY   STATUS    RESTARTS   AGE   IP            NODE          NOMINATED NODE   READINESS GATES
deploy-nginx-5c68c45fb6-9d8qr  1/1     Running   0          43s   10.244.0.28   k8s-master    <none>            <none>
deploy-nginx-5c68c45fb6-kkch2  1/1     Running   0          43s   10.244.1.13   k8s-worker1   <none>            <none>
deploy-nginx-5c68c45fb6-ps2d8  1/1     Running   0          43s   10.244.1.14   k8s-worker1   <none>            <none>
deploy-nginx-5c68c45fb6-zkvlh  1/1     Running   0          43s   10.244.2.5    k8s-worker2   <none>            <none>
deploy-nginx-5c68c45fb6-zsvkx  1/1     Running   0          43s   10.244.2.4    k8s-worker2   <none>            <none>

[root@k8s-master ~]# kubectl  delete  pods  deploy-nginx-5c68c45fb6-9d8qr
pod "deploy-nginx-5c68c45fb6-9d8qr" deleted from default namespace

[root@k8s-master ~]# kubectl  get  pods  -o  wide
NAME                           READY   STATUS    RESTARTS   AGE     IP            NODE          NOMINATED NODE   READINESS GATES
deploy-nginx-5c68c45fb6-jkn76  1/1     Running   0          2s      10.244.2.6    k8s-worker2   <none>            <none>
deploy-nginx-5c68c45fb6-kkch2  1/1     Running   0          8m58s   10.244.1.13   k8s-worker1   <none>            <none>
deploy-nginx-5c68c45fb6-ps2d8  1/1     Running   0          8m58s   10.244.1.14   k8s-worker1   <none>            <none>
deploy-nginx-5c68c45fb6-zkvlh  1/1     Running   0          8m58s   10.244.2.5    k8s-worker2   <none>            <none>
deploy-nginx-5c68c45fb6-zsvkx  1/1     Running   0          8m58s   10.244.2.4    k8s-worker2   <none>            <none>
```

삭제된 Master의 Pod가 다시 Master로 스케줄되지 않고 k8s-worker2로 대체 생성된다.

```bash
# Deployment로 Pod를 10개로 증가
[root@k8s-master ~]# kubectl  scale  deployment  deploy-nginx  --replicas=10
deployment.apps/deploy-nginx scaled

# Pod를 증가시켜도 Master에는 Pod가 다시 생성되지 않는다
[root@k8s-master ~]# kubectl  get  pods  -o  wide
NAME                           READY   STATUS    RESTARTS   AGE    IP            NODE
deploy-nginx-5c68c45fb6-f2842  1/1     Running   0          14s    10.244.1.17   k8s-worker1
deploy-nginx-5c68c45fb6-gc8lp  1/1     Running   0          14s    10.244.2.7    k8s-worker2
deploy-nginx-5c68c45fb6-jkn76  1/1     Running   0          2m10s  10.244.2.6    k8s-worker2
deploy-nginx-5c68c45fb6-kkch2  1/1     Running   0          11m    10.244.1.13   k8s-worker1
deploy-nginx-5c68c45fb6-kqpmv  1/1     Running   0          14s    10.244.1.16   k8s-worker1
deploy-nginx-5c68c45fb6-n5879  1/1     Running   0          14s    10.244.1.15   k8s-worker1
deploy-nginx-5c68c45fb6-ps2d8  1/1     Running   0          11m    10.244.1.14   k8s-worker1
deploy-nginx-5c68c45fb6-s7lmp  1/1     Running   0          14s    10.244.2.8    k8s-worker2
deploy-nginx-5c68c45fb6-zkvlh  1/1     Running   0          11m    10.244.2.5    k8s-worker2
deploy-nginx-5c68c45fb6-zsvkx  1/1     Running   0          11m    10.244.2.4    k8s-worker2

[root@k8s-master ~]# kubectl  delete  deployments  deploy-nginx
deployment.apps "deploy-nginx" deleted from default namespace
```

재-Taint 이후에는 새로 생성되는 Pod가 Master에 스케줄되지 않는다 — 임시로 Taint를 제거해 Master를 Worker처럼 활용한 뒤, 다시 Taint를 설정하면 정상적인 Control-Plane 상태로 돌아온다.

## Pod Affinity 실습

**EX1) Pod Affinity 미적용 상태 확인**

```yaml
# affinity-step1-cache.yaml
apiVersion: v1
kind: Pod
metadata:
  name: cache
  labels:
    app: cache
spec:
  containers:
  - name: nginx
    image: nginx:1.29.1
```

```bash
[root@k8s-master ~]# kubectl  apply  -f  affinity-step1-cache.yaml
[root@k8s-master ~]# kubectl  apply  -f  affinity-step1-frontend.yaml

[root@k8s-master ~]# kubectl  get  pods  -o  wide
NAME       READY   STATUS    RESTARTS   AGE   IP            NODE          NOMINATED NODE   READINESS GATES
cache      1/1     Running   0          17s   10.244.2.9    k8s-worker2   <none>            <none>
frontend   1/1     Running   0          14s   10.244.1.18   k8s-worker1   <none>            <none>
```

Affinity를 사용하지 않으면 두 개의 서버가 서로 다른 노드에 생성될 수 있다(상황에 따라 같은 노드에 생성될 수도 있다).

**EX2) Pod Affinity 적용 — frontend를 cache와 같은 Node에 배치**

```yaml
# affinity-step2-frontend.yaml
apiVersion: v1
kind: Pod
metadata:
  name: frontend
  labels:
    app: frontend
spec:
  affinity:
    podAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:   # required = 강제
      - labelSelector:
          matchExpressions:
          - key: app
            operator: In
            values: [cache]
        topologyKey: kubernetes.io/hostname   # app: cache Label을 가진 Pod와 Hostname이 같은 노드에 배치
  containers:
  - name: nginx
    image: nginx:1.29.1
```

```bash
[root@k8s-master ~]# kubectl  apply  -f  affinity-step2-frontend.yaml
pod/frontend created

# app: cache Pod가 있는 노드에 frontend Pod가 생성된다.
[root@k8s-master ~]# kubectl  get pods  -o  wide
NAME       READY   STATUS    RESTARTS   AGE   IP            NODE          NOMINATED NODE   READINESS GATES
cache      1/1     Running   0          33m   10.244.2.9    k8s-worker2   <none>            <none>
frontend   1/1     Running   0          22s   10.244.2.10   k8s-worker2   <none>            <none>
```

**EX3) cache Pod가 없는 상태에서 frontend Pod 생성 후 확인**

```bash
[root@k8s-master ~]# kubectl  delete  pods  cache
[root@k8s-master ~]# kubectl  delete  pods  frontend
[root@k8s-master ~]# kubectl  apply  -f  affinity-step2-frontend.yaml
pod/frontend created

[root@k8s-master ~]# kubectl  get pods  -o  wide
NAME       READY   STATUS    RESTARTS   AGE   IP       NODE     NOMINATED NODE   READINESS GATES
frontend   0/1     Pending   0          2s    <none>   <none>   <none>            <none>
```

frontend Pod가 Pending되는 이유는 `app=cache` Label을 가진 Pod가 현재 존재하지 않기 때문이다. `requiredDuringSchedulingIgnoredDuringExecution`은 강제 조건이므로, frontend Pod는 반드시 `app=cache` Pod가 실행 중인 노드와 같은 노드에 배치되어야 하는데, 조건을 만족하는 cache Pod가 없어 스케줄러가 배치할 노드를 찾지 못하고 Pending 상태가 된다.

cache Pod를 다시 생성하면 Pending이던 frontend가 즉시 같은 노드에서 실행된다.

**EX5) Preferred Pod Affinity + weight 실습**

cache Pod의 Label은 `app=cache`이고, frontend Pod는 `app=cache` Pod가 있는 Node를 선호하도록 `preferredDuringSchedulingIgnoredDuringExecution`, `weight: 80`, `topologyKey: kubernetes.io/hostname`을 사용해 2개 생성한다.

```yaml
# affinity-step5-cache.yaml
apiVersion: v1
kind: Pod
metadata:
  name: cache
  labels:
    app: cache
spec:
  containers:
  - name: nginx
    image: nginx:1.29.1
```

```bash
[root@k8s-master ~]# kubectl apply -f affinity-step5-cache.yaml
pod/cache created

[root@k8s-master ~]# kubectl get pod cache -o wide
NAME    READY   STATUS    RESTARTS   AGE   IP            NODE          NOMINATED NODE   READINESS GATES
cache   1/1     Running   0          4s    10.244.2.13   k8s-worker2   <none>            <none>
```

```yaml
# affinity-step5-frontend.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
spec:
  replicas: 2
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
    spec:
      affinity:
        podAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 80
            podAffinityTerm:
              labelSelector:
                matchExpressions:
                - key: app
                  operator: In
                  values: [cache]
              topologyKey: kubernetes.io/hostname
      containers:
      - name: nginx
        image: nginx:1.29.1
```

```bash
[root@k8s-master ~]# kubectl apply -f affinity-step5-frontend.yaml
deployment.apps/frontend created

[root@k8s-master ~]# kubectl get pods -o wide
NAME                        READY   STATUS   RESTARTS   AGE     IP            NODE          NOMINATED NODE   READINESS GATES
cache                       1/1     Running  0          8m45s   10.244.2.13   k8s-worker2   <none>            <none>
frontend-5555496466-gqxz    1/1     Running  0          2m30s   10.244.2.15   k8s-worker2   <none>            <none>
frontend-5555496466-h9j6n   1/1     Running  0          2m30s   10.244.2.14   k8s-worker2   <none>            <none>
```

cache Pod가 k8s-worker2에서 실행 중이므로 frontend Pod 2개도 같은 Node에 배치되는 것을 우선적으로 선호한다. weight 80은 k8s-worker2에 스케줄링 점수 80점을 추가로 부여한다는 의미다. 하지만 preferred는 강제 조건이 아니므로 CPU/Memory 부족 등의 이유로 다른 조건이 더 적합하면 frontend Pod가 다른 Node에 배치될 수도 있다.

Replica를 5개로 늘려도 Pod Affinity가 같이 모이는 것을 선호하기 때문에 cache Pod가 있는 Node로 frontend Pod들이 모이는 방향으로 배치된다.

## Pod Anti-Affinity 실습

복제 Pod가 같은 노드에 같이 있지 못하도록 강제하는 것이 목적이며, 고가용성(HA)에서 가장 기본이 되는 분산 규칙을 만든다.

**EX1) required Pod Anti-Affinity로 2개 Pod를 반드시 다른 노드로 분산시키기**

```yaml
# anti-step1-required.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-antiaffinity
spec:
  replicas: 2
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: app
                operator: In
                values: [web]
            topologyKey: kubernetes.io/hostname
      containers:
      - name: nginx
        image: nginx:1.29.1
```

```bash
[root@k8s-master ~]# kubectl  apply  -f  anti-step1-required.yaml
deployment.apps/web-antiaffinity created

[root@k8s-master ~]# kubectl  get  pods  -o  wide
NAME                                READY   STATUS    RESTARTS   AGE   IP            NODE          NOMINATED NODE   READINESS GATES
web-antiaffinity-6fc4ddbfd7-dxzrv   1/1     Running   0          24s   10.244.2.19   k8s-worker2   <none>            <none>
web-antiaffinity-6fc4ddbfd7-vppn5   1/1     Running   0          24s   10.244.1.19   k8s-worker1   <none>            <none>
```

`app=web` Pod 2개가 서로 다른 Node에 배치된 것을 확인할 수 있다 — required Pod Anti-Affinity 조건이 정상 적용되어 같은 app=web Pod끼리는 같은 Node에 배치될 수 없다.

```bash
[root@k8s-master ~]# kubectl  scale  deployment  web-antiaffinity  --replicas=3

[root@k8s-master ~]# kubectl  get  pods  -o  wide
NAME                                READY   STATUS    RESTARTS   AGE     IP            NODE          NOMINATED NODE   READINESS GATES
web-antiaffinity-6fc4ddbfd7-dxzrv   1/1     Running   0          3m47s   10.244.2.19   k8s-worker2   <none>            <none>
web-antiaffinity-6fc4ddbfd7-vlqqb   0/1     Pending   0          13s     <none>        <none>        <none>            <none>
web-antiaffinity-6fc4ddbfd7-vppn5   1/1     Running   0          3m47s   10.244.1.19   k8s-worker1   <none>            <none>
```

Worker Node가 2개뿐이고 required Pod Anti-Affinity에 의해 같은 `app=web` Pod를 같은 Node에 배치할 수 없으므로, 세 번째 Pod는 배치될 Node가 없어 Pending 상태를 유지한다.

**EX2) preferred Pod Anti-Affinity로 분산 배치 유도**

```yaml
# anti-step2-required.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: anti-aff
spec:
  replicas: 2
  selector:
    matchLabels:
      app: web-step2
  template:
    metadata:
      labels:
        app: web-step2
    spec:
      containers:
      - name: nginx
        image: nginx
      affinity:
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 80
            podAffinityTerm:
              labelSelector:
                matchExpressions:
                - key: app
                  operator: In
                  values: [web-step2]
              topologyKey: kubernetes.io/hostname
```

```bash
[root@k8s-master ~]# kubectl  apply  -f  anti-step2-required.yaml
deployment.apps/web-step2 created

[root@k8s-master ~]# kubectl  scale  deployment  web-step2  --replicas=5

[root@k8s-master ~]# kubectl  get  pods  -o  wide
NAME                       READY   STATUS    RESTARTS   AGE   IP            NODE          NOMINATED NODE   READINESS GATES
anti-aff-9c44899d7-46gd5   1/1     Running   0          5m    10.244.2.20   k8s-worker2   <none>            <none>
anti-aff-9c44899d7-bj8sf   1/1     Running   0          7s    10.244.2.22   k8s-worker2   <none>            <none>
anti-aff-9c44899d7-l2sjp   1/1     Running   0          7s    10.244.1.21   k8s-worker1   <none>            <none>
anti-aff-9c44899d7-llzlr   1/1     Running   0          7s    10.244.2.21   k8s-worker2   <none>            <none>
anti-aff-9c44899d7-qzd8n   1/1     Running   0          5m    10.244.1.20   k8s-worker1   <none>            <none>
```

preferred이므로 required와 달리 Node가 부족해도 Pending되지 않고, 가능한 만큼 분산시키면서 나머지는 같은 Node에도 배치된다.

## Taint & Toleration이란

Taint & Toleration은 특정 Node에 아무 Pod나 배치되지 못하도록 제한하고, 필요한 Pod만 해당 Node에 배치할 수 있도록 하는 기능이다.

- Taint = Node에 출입 제한을 거는 것
- Toleration = Pod가 그 출입 제한을 견딜 수 있도록 허용하는 것

즉, Node가 "아무 Pod나 들어오지 마"라고 막는다면 Pod는 "나는 이 조건을 수용할 수 있어"라고 설정해야 해당 Taint가 있는 Node에 배치될 수 있다.

Taint를 사용하는 이유는 모든 Node를 모든 Pod가 사용하는 것이 항상 좋은 것은 아니기 때문이다. GPU 전용 Node, 데이터베이스 전용 Node, 중요한 시스템 Pod용 Node, 운영 전용 Node, 특정 팀이나 서비스 전용 Node 등을 만들고 싶을 때 해당 Node에 Taint를 설정하면 일반 Pod가 배치되는 것을 막을 수 있다.

```bash
# Node 2에 Taint 설정
kubectl  taint  nodes  k8s-worker2  gpu=true:NoSchedule
```

## Taint의 기본 구조와 Effect

형식: `key=value:effect`

- key = Taint의 이름
- value = Taint의 값
- effect = 해당 Taint가 Pod에게 적용하는 동작

예시: `gpu=true:NoSchedule`은 "gpu=true라는 제한이 걸려 있으므로 이 조건을 허용하지 않는 Pod는 배치하지 마라"라는 의미다.

Taint의 Effect는 대표적으로 3가지가 있다.

- **NoSchedule**: Toleration이 없는 Pod를 새롭게 배치하지 않는다. 기존에 실행 중인 Pod는 그대로 실행될 수 있다. 가장 일반적으로 사용하는 방식이다.
- **PreferNoSchedule**: 가능하면 해당 Taint를 허용하지 않는 Pod를 배치하지 않으려고 하지만, 다른 조건에 따라 해당 Node에 배치될 수도 있다. 강제 차단이 아니라 선호 방식이다.
- **NoExecute**: Toleration이 없는 Pod를 새롭게 배치하지 않을 뿐 아니라, Taint를 허용하지 않는 기존 Pod를 해당 Node에서 제거할 수도 있다.

NoSchedule과 NoExecute의 차이:

- NoSchedule = 새로 들어오는 것을 막음, 기존 Pod에는 직접적인 영향이 없음
- NoExecute = 새로 들어오는 것도 막고 기존 Pod도 내보낼 수 있다

## Toleration이란

Toleration은 Pod가 특정 Taint를 허용할 수 있도록 설정하는 것이다. 중요한 점은 Toleration이 있다고 해서 해당 Node에 반드시 배치되는 것은 아니라는 것이다 — "이 Pod는 이 Taint를 허용할 수 있다"는 뜻일 뿐이다. 해당 Node에 배치되려면 Taint를 허용하고 다른 스케줄링 조건도 만족해야 한다. 특정 Node에 반드시 배치하고 싶다면 nodeSelector나 Node Affinity 등을 함께 사용해야 한다.

```yaml
tolerations:
- key: "gpu"
  operator: "Equal"
  value: "true"
  effect: "NoSchedule"
```

Pod가 생성되면 스케줄러는 다음 순서로 판단한다.

1. Node에 Taint가 있는지 확인
2. Pod에 해당 Taint를 허용하는 Toleration이 있는지 확인
3. 허용하지 못하는 Taint가 있으면 해당 Node를 후보에서 제외
4. 모든 Taint를 허용할 수 있다면 다른 스케줄링 조건을 확인
5. 자원, Affinity 등 다른 조건까지 만족하면 해당 Node가 선택될 수 있음

## Taint와 Node Affinity의 차이

Taint와 Node Affinity는 목적이 다르다.

- Taint: Node 입장에서 Pod를 제한 ("누가 들어올 수 있는가")
- Node Affinity: Pod 입장에서 원하는 Node를 선택 ("어디에 들어갈 것인가")

GPU Node를 일반 Pod가 사용하지 못하게 하려면 Node에 `gpu=true:NoSchedule` Taint를 설정하고, GPU Pod가 그 Node를 사용하도록 하려면 Pod에 Toleration을 설정한다. GPU Pod가 반드시 GPU Node에 배치되기를 원한다면 Toleration에 Node Affinity 또는 nodeSelector를 함께 사용해야 한다.

## Taint & Toleration 실습

**EX1) Toleration 설정 X**

```bash
[root@k8s-master ~]# kubectl  taint  nodes  k8s-worker1  role=db:NoSchedule
node/k8s-worker1 tainted

[root@k8s-master ~]# kubectl  describe  nodes  k8s-worker1 | grep Taint
Taints:             role=db:NoSchedule
```

```yaml
# taint-step1-noschedule.yaml (Toleration 없음)
apiVersion: v1
kind: Pod
metadata:
  name: taint-step1-pod-1
spec:
  containers:
  - name: nginx-container
    image: nginx:1.29.1
---
apiVersion: v1
kind: Pod
metadata:
  name: taint-step1-pod-2
spec:
  containers:
  - name: nginx-container
    image: nginx:1.29.1
```

```bash
[root@k8s-master ~]# kubectl  apply  -f  taint-step1-noschedule.yaml
[root@k8s-master ~]# kubectl  get  pods  -o  wide
NAME                READY   STATUS    RESTARTS   AGE   IP            NODE          NOMINATED NODE   READINESS GATES
taint-step1-pod-1   1/1     Running   0          3s    10.244.2.24   k8s-worker2   <none>            <none>
taint-step1-pod-2   1/1     Running   0          3s    10.244.2.25   k8s-worker2   <none>            <none>
```

Toleration이 없는 Pod는 role=db:NoSchedule Taint가 설정된 k8s-worker1에는 배치되지 못하고, Taint가 없는 k8s-worker2에만 배치된다.

**Toleration 설정 O**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: taint-step1-pod-1
spec:
  tolerations:
  - key: "role"
    operator: "Equal"
    value: "db"
    effect: "NoSchedule"
  containers:
  - name: nginx-container
    image: nginx:1.29.1
```

이 Toleration을 추가하면 k8s-worker1의 Taint를 허용할 수 있게 되어(다만 반드시 그 Node로 가는 것은 아니다) 두 Node 모두 후보가 될 수 있다.

**EX3) 일반 Pod와 전용 Pod를 Taint로 분리**

- k8s-worker1: `role=db:NoSchedule` Taint 설정
- k8s-worker2: Taint 없음

db-pod를 반드시 k8s-worker1에 배치하려면 Toleration만으로는 부족하고, k8s-worker1에 Label을 설정하고 nodeSelector를 함께 사용해야 한다.

```bash
[root@k8s-master ~]# kubectl  label  nodes  k8s-worker1  role=db
node/k8s-worker1 labeled
```

```yaml
# taint-step3-mixed.yaml
apiVersion: v1
kind: Pod
metadata:
  name: normal-pod
spec:
  containers:
  - name: nginx
    image: nginx
---
apiVersion: v1
kind: Pod
metadata:
  name: db-pod
spec:
  nodeSelector:
    role: db
  tolerations:
  - key: "role"
    operator: "Equal"
    value: "db"
    effect: "NoSchedule"
  containers:
  - name: db
    image: mysql:8.0
    env:
    - name: MYSQL_ROOT_PASSWORD
      value: "root"
    ports:
    - containerPort: 3306
```

```bash
[root@k8s-master ~]# kubectl  apply  -f  taint-step3-mixed.yaml
[root@k8s-master ~]# kubectl   get  pods  -o  wide
NAME         READY   STATUS    RESTARTS   AGE   IP            NODE          NOMINATED NODE   READINESS GATES
db-pod       1/1     Running   0          13s   10.244.1.24   k8s-worker1   <none>            <none>
normal-pod   1/1     Running   0          13s   10.244.2.28   k8s-worker2   <none>            <none>
```

**EX4) Node마다 서로 다른 Taint를 설정하고 여러 Pod의 Toleration 동작 확인**

```bash
[root@k8s-master ~]# kubectl taint node k8s-worker1 cpu=true:NoSchedule
[root@k8s-master ~]# kubectl taint node k8s-worker2 gpu=true:NoSchedule
[root@k8s-master ~]# kubectl taint node k8s-master cpu=true:NoSchedule
[root@k8s-master ~]# kubectl taint node k8s-master gpu=true:NoSchedule

[root@k8s-master ~]# kubectl  label  node  k8s-worker1  type=cpu
[root@k8s-master ~]# kubectl  label  node  k8s-worker2  type=gpu
[root@k8s-master ~]# kubectl  label  node  k8s-master  type=cpu-gpu
```

```yaml
# taint-step4-multi-taint-nodeselector.yaml
apiVersion: v1
kind: Pod
metadata:
  name: cpu-pod
spec:
  nodeSelector:
    type: cpu
  tolerations:
  - key: "cpu"
    operator: "Equal"
    value: "true"
    effect: "NoSchedule"
  containers:
  - name: nginx
    image: nginx:1.29.1
---
apiVersion: v1
kind: Pod
metadata:
  name: gpu-pod
spec:
  nodeSelector:
    type: gpu
  tolerations:
  - key: "gpu"
    operator: "Equal"
    value: "true"
    effect: "NoSchedule"
  containers:
  - name: nginx
    image: nginx:1.29.1
---
apiVersion: v1
kind: Pod
metadata:
  name: cpu-gpu-pod
spec:
  nodeSelector:
    type: cpu-gpu
  tolerations:
  - key: "cpu"
    operator: "Equal"
    value: "true"
    effect: "NoSchedule"
  - key: "gpu"
    operator: "Equal"
    value: "true"
    effect: "NoSchedule"
  containers:
  - name: nginx
    image: nginx:1.29.1
```

```bash
[root@k8s-master ~]# kubectl  apply  -f  taint-step4-multi-taint-nodeselector.yaml
[root@k8s-master ~]# kubectl  get  pods  -o  wide
NAME          READY   STATUS    RESTARTS   AGE   IP             NODE
cpu-gpu-pod   1/1     Running   0          15s   10.244.0.29    k8s-master
cpu-pod       1/1     Running   0          15s   10.244.1.25    k8s-worker1
gpu-pod       1/1     Running   0          15s   10.244.2.29    k8s-worker2
```

각 Pod는 nodeSelector로 특정 Node를 선택하고 해당 Node의 Taint를 모두 허용하는 Toleration을 가지고 있어 정상적으로 배치된다.

**EX5) NoSchedule과 PreferNoSchedule의 차이 확인**

```bash
[root@k8s-master ~]# kubectl  taint  node  k8s-worker1  role=db:NoSchedule
[root@k8s-master ~]# kubectl  cordon  k8s-worker2
```

Toleration이 없는 Pod 2개를 생성하면, k8s-worker2는 cordon 상태, k8s-worker1은 NoSchedule Taint, k8s-master는 control-plane Taint가 있어 모두 배치될 곳이 없어 Pending 상태가 된다.

```bash
[root@k8s-master ~]# kubectl  get  pods
NAME                  READY   STATUS    RESTARTS   AGE
taint-step1-pod-1     0/1     Pending   0          33s
taint-step1-pod-2     0/1     Pending   0          33s
```

Taint를 `role=db:PreferNoSchedule`로 바꾸면(강제가 아닌 선호이므로) Pod가 정상적으로 k8s-worker1에 배치된다.

```bash
[root@k8s-master ~]# kubectl taint node  k8s-worker1 role=db:NoSchedule-
[root@k8s-master ~]# kubectl taint node k8s-worker1 role=db:PreferNoSchedule
[root@k8s-master ~]# kubectl  apply  -f  taint-prefernoschedule.yaml

[root@k8s-master ~]# kubectl  get  pods  -o  wide
NAME                READY   STATUS    RESTARTS   AGE   IP            NODE          NOMINATED NODE   READINESS GATES
taint-step1-pod-1   1/1     Running   0          11s   10.244.1.26   k8s-worker1   <none>            <none>
taint-step1-pod-2   1/1     Running   0          11s   10.244.1.27   k8s-worker1   <none>            <none>
```

**정리**: NoSchedule은 조건을 만족 못 하면 절대 배치되지 않아 다른 후보가 없으면 Pending이 되지만, PreferNoSchedule은 되도록 피할 뿐이라 다른 대안이 없으면 그 Node에도 배치된다.

## Cordon & Drain이란

Cordon과 Drain은 특정 Node에 새로운 Pod가 배치되지 않도록 하거나, 기존에 실행 중인 Pod를 다른 Node로 이동시키기 위해 사용하는 Node 관리 기능이다. Node를 비우고 유지보수하거나, OS 업데이트, Kubernetes 업그레이드, 하드웨어 교체 등을 진행하기 전에 이 두 기능이 자주 쓰인다.

- Cordon = 새 Pod를 받지 않도록 Node를 막음
- Drain = Node를 비우기 위해 기존 Pod를 다른 곳으로 이동시킴

### Cordon

Cordon은 특정 Node를 Scheduling 대상에서 제외하는 기능이다. 새로운 Pod만 해당 Node에 배치되지 않고, 기존에 실행 중인 Pod는 그대로 실행된다.

```
kubectl cordon <Node이름>
예: kubectl cordon k8s-worker1
```

```bash
kubectl get nodes
NAME          STATUS                     ROLES
k8s-worker1   Ready,SchedulingDisabled   <none>
k8s-worker2   Ready                      <none>
```

SchedulingDisabled가 표시되면 해당 Node가 Cordon 상태라는 의미다.

Cordon을 해제하면 다시 새로운 Pod를 해당 Node에 배치할 수 있다.

```
kubectl uncordon <Node이름>
예: kubectl uncordon k8s-worker1
```

uncordon은 기존 Pod를 다시 배치하는 명령이 아니라, 단지 해당 Node를 다시 Scheduling 가능 상태로 변경할 뿐이다.

### Drain

Drain은 특정 Node에서 실행 중인 Pod를 정상적으로 종료하고 다른 Node에서 다시 실행될 수 있도록 Node를 비우는 기능이다. 주로 Node를 유지보수하거나 종료하기 전에 사용한다.

```
kubectl drain <Node이름>
예: kubectl drain k8s-worker1
```

```
Drain  -->  새로운 Pod 배치 차단  -->  기존 Pod 종료/퇴출  -->  Controller가 관리하는 Pod라면 다른 Node에 새로운 Pod 생성  -->  Node가 비워짐
```

Drain은 Pod를 단순히 삭제하는 명령어가 아니다. Deployment, ReplicaSet 등의 Controller가 관리하는 Pod라면 기존 Pod를 종료한 뒤 ReplicaSet 등이 원하는 Replica 수를 유지하기 위해 다른 Node에 새로운 Pod를 생성한다. 단, 실제 배치는 자원, Affinity, Taint 등의 조건에 따라 달라질 수 있다.

## Cordon과 Drain의 차이

- **Cordon**: 새로운 Pod 배치 차단, 기존 Pod 유지, Node를 Scheduling에서 제외 ("새로운 Pod만 받지 마")
- **Drain**: 새로운 Pod 배치 차단, 기존 Pod를 종료/퇴출, Controller가 관리하는 Pod는 다른 Node에서 다시 생성될 수 있음 ("새로운 Pod도 받지 말고, 기존 Pod도 다른 곳으로 보내서 이 Node를 비워")

Drain은 Node를 비우는 과정에서 새로운 Pod가 해당 Node로 들어오지 못하도록 먼저 Scheduling을 막는다. 즉 개념적으로는 `Cordon --> 새로운 Pod 배치 차단 --> Drain --> 기존 Pod 퇴출 --> Node 비움` 순서이며, 실제로 `kubectl drain`을 실행하면 이 Scheduling 차단 동작까지 함께 수행한다.

전체 유지보수 흐름:

1. Cordon — `kubectl cordon k8s-worker1` (새로운 Pod 배치 차단, 기존 Pod 계속 실행)
2. Drain — `kubectl drain k8s-worker1 --ignore-daemonsets` (기존 Pod 종료/퇴출, 관리되는 Pod는 다른 Node에서 재생성, Node를 비움)
3. Node 유지보수 (OS 업데이트, Kubernetes 구성 변경, 하드웨어 점검 등)
4. Uncordon — `kubectl uncordon k8s-worker1` (Scheduling 다시 허용)

### Drain에서 자주 사용하는 옵션

- `--ignore-daemonsets`: DaemonSet이 관리하는 Pod는 Drain 과정에서 무시한다. DaemonSet Pod는 각 Node에 존재하는 것이 목적이라 일반적인 Drain으로 다른 Node에 옮기는 대상이 아니므로, DaemonSet Pod가 있는 Node를 Drain할 때 자주 사용한다.
- `--delete-emptydir-data`: emptyDir 볼륨을 사용하는 Pod의 데이터를 삭제할 수 있도록 허용한다. emptyDir 데이터는 Node의 Pod가 삭제되면 유지되지 않으므로, 해당 데이터가 삭제되어도 되는지 확인한 후 사용해야 한다.
- `--force=false`(기본값): 컨트롤러가 관리하지 않는 일반 Pod가 Node에 존재하면 강제로 삭제하지 않는다. 일반 Pod는 삭제되어도 자동으로 재생성되지 않을 수 있으므로, 서비스 중단을 방지하기 위해 drain이 이런 Pod의 강제 삭제를 막는다.
- `--force=true`: Deployment, ReplicaSet 등의 컨트롤러가 관리하지 않는 일반 Pod도 삭제를 허용한다. 삭제 후 자동으로 다시 생성되지 않을 수 있으므로 주의해야 한다.

### PodDisruptionBudget과 Drain

운영 환경에서는 PodDisruptionBudget(PDB)도 함께 고려해야 한다. PDB는 유지보수나 Drain처럼 자발적인 Pod 중단(Voluntary Disruption) 상황에서 동시에 중단되는 Pod 수를 제한하는 기능이다. 예를 들어 동일한 애플리케이션 Pod가 3개 실행 중일 때 최소 2개는 항상 실행되도록 PDB를 설정하면, Node Drain 과정에서 애플리케이션의 가용성을 유지하는 데 도움이 된다.

### Cordon과 Taint의 차이

- **Cordon**: Node 전체를 Scheduling에서 제외한다. 새로운 Pod가 해당 Node에 배치되지 않으며 별도의 Pod 설정이 필요 없다. ("현재 이 Node에는 새로운 Pod를 전부 받지 않음")
- **Taint**: Node에 특정 제한을 설정한다. 해당 Taint를 허용하지 않는 Pod만 배치가 제한되며, Toleration이 있는 Pod는 통과할 수 있다. ("이 조건을 허용하지 않는 Pod는 받지 않음")

## Cordon & Drain 실습

**EX1) cordon으로 신규 Pod 배치 차단**

cordon은 기존 Pod에는 영향을 주지 않고 새로운 Pod만 배치되지 않게 만든다는 것을 확인한다.

```yaml
# cordon-step1.yaml
apiVersion: v1
kind: Pod
metadata:
  name: cordon-test-pod
spec:
  containers:
  - name: nginx
    image: nginx:1.29.1
```

```bash
[root@k8s-master ~]# kubectl  apply  -f  cordon-step1.yaml
pod/cordon-test-pod created

[root@k8s-master ~]# kubectl  cordon  k8s-worker2
node/k8s-worker2 cordoned

[root@k8s-master ~]# kubectl  get  nodes
NAME          STATUS                     ROLES
k8s-worker2   Ready,SchedulingDisabled   <none>

# 새로운 Pod 2개 생성 시도
[root@k8s-master ~]# kubectl  apply  -f  taint-prefernoschedule.yaml
pod/taint-step1-pod-1 created
pod/taint-step1-pod-2 created

# 기존의 Pod는 유지되지만 새로운 Pod는 다른 Node에만 배치된다.
[root@k8s-master ~]# kubectl  get  pods  -o  wide
NAME                READY   STATUS    RESTARTS   AGE   IP            NODE          NOMINATED NODE   READINESS GATES
cordon-test-pod     1/1     Running   0          4m    10.244.2.30   k8s-worker2   <none>            <none>
taint-step1-pod-1   1/1     Running   0          19s   10.244.1.29   k8s-worker1   <none>            <none>
taint-step1-pod-2   1/1     Running   0          19s   10.244.1.28   k8s-worker1   <none>            <none>
```

**EX2) drain으로 기존 Pod를 안전하게 이동**

```yaml
# drain-ex2-deploy.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: drain-test-deploy
spec:
  replicas: 4
  selector:
    matchLabels:
      app: drain-test
  template:
    metadata:
      labels:
        app: drain-test
    spec:
      containers:
      - name: nginx
        image: nginx
```

```bash
[root@k8s-master ~]# kubectl  apply  -f  drain-ex2-deploy.yaml

[root@k8s-master ~]# kubectl  get  pods  -o  wide
NAME                                READY   STATUS    RESTARTS   AGE   IP            NODE
drain-test-deploy-789567f66-2qrs4   1/1     Running   0          6s    10.244.2.31   k8s-worker2
drain-test-deploy-789567f66-bqfpk   1/1     Running   0          6s    10.244.1.30   k8s-worker1
drain-test-deploy-789567f66-h7vch   1/1     Running   0          6s    10.244.2.32   k8s-worker2
drain-test-deploy-789567f66-tjcxr   1/1     Running   0          6s    10.244.1.31   k8s-worker1

[root@k8s-master ~]# kubectl  drain  k8s-worker1  --ignore-daemonsets --delete-emptydir-data
node/k8s-worker1 cordoned
Warning: ignoring DaemonSet-managed Pods: kube-flannel/kube-flannel-ds-f9wcz, kube-system/kube-proxy-gbr4f
evicting pod default/drain-test-deploy-789567f66-bqfpk
evicting pod ingress-nginx/ingress-nginx-controller-f85ff6d7d-wvdpf
evicting pod default/drain-test-deploy-789567f66-tjcxr
pod/drain-test-deploy-789567f66-bqfpk evicted
pod/drain-test-deploy-789567f66-tjcxr evicted
pod/ingress-nginx-controller-f85ff6d7d-wvdpf evicted
node/k8s-worker1 drained

[root@k8s-master ~]# kubectl  get  pods  -o  wide
NAME                                READY   STATUS    RESTARTS   AGE    IP            NODE
drain-test-deploy-789567f66-2qrs4   1/1     Running   0          4m5s   10.244.2.31   k8s-worker2
drain-test-deploy-789567f66-2rqvk   1/1     Running   0          24s    10.244.2.33   k8s-worker2
drain-test-deploy-789567f66-bckdv   1/1     Running   0          24s    10.244.2.35   k8s-worker2
drain-test-deploy-789567f66-h7vch   1/1     Running   0          4m5s   10.244.2.32   k8s-worker2

[root@k8s-master ~]# kubectl  get  nodes
NAME          STATUS                     ROLES
k8s-worker1   Ready,SchedulingDisabled   <none>
k8s-worker2   Ready                      <none>
```

drain을 취소하는 명령어는 별도로 존재하지 않으며 uncordon으로 Scheduling을 다시 허용한다.

```bash
[root@k8s-master ~]# kubectl  uncordon  k8s-worker1
node/k8s-worker1 uncordoned
```

**EX2 유지보수 실습: cordon + drain + uncordon + 파드 재배치 확인 (DaemonSet 포함)**

```yaml
# step2-daemonset.yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: drain-ds
spec:
  selector:
    matchLabels:
      app: drain-ds
  template:
    metadata:
      labels:
        app: drain-ds
    spec:
      containers:
      - name: agent
        image: busybox:1.36
        command: ["/bin/sh","-c"]
        args:
        - |
          while true; do
            echo "$(date) node=$(cat /etc/hostname)" >> /var/log/agent.log
            sleep 10
          done
```

```bash
[root@k8s-master ~]# kubectl  apply  -f  step2-daemonset.yaml
daemonset.apps/drain-ds created

[root@k8s-master ~]# kubectl  get  daemonsets  drain-ds
NAME       DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
drain-ds   2         2         2       2            2           <none>          79s

# k8s-worker2에 Cordon 설정
[root@k8s-master ~]# kubectl  cordon  k8s-worker2

# replicas를 6으로 변경 (Cordon에 의해 새 Pod는 k8s-worker1에만 생성된다.)
[root@k8s-master ~]# kubectl  scale  deployment  drain-test-deploy  --replicas=6

[root@k8s-master ~]# kubectl  drain  k8s-worker2  --ignore-daemonsets  --force=false
node/k8s-worker2 already cordoned
Warning: ignoring DaemonSet-managed Pods: default/drain-ds-7z2qs, kube-flannel/kube-flannel-ds-z4fbv, kube-system/kube-proxy-bxjvw
evicting pod ingress-nginx/ingress-nginx-controller-f85ff6d7d-9lzcj
evicting pod default/drain-test-deploy-789567f66-4f745
evicting pod default/drain-test-deploy-789567f66-tlr4m
pod/drain-test-deploy-789567f66-tlr4m evicted
pod/drain-test-deploy-789567f66-4f745 evicted
pod/ingress-nginx-controller-f85ff6d7d-9lzcj evicted
node/k8s-worker2 drained
```

`--ignore-daemonsets` 덕분에 DaemonSet Pod(`drain-ds`)는 k8s-worker2에 그대로 남고, Deployment가 관리하는 일반 Pod만 evict되어 k8s-worker1로 옮겨간다. Drain은 1회성 동작이므로 완료 후에도 k8s-worker2는 계속 Cordon(SchedulingDisabled) 상태로 남는다.

**정리**: Cordon은 새 Pod 배치만 막고 기존 Pod는 그대로 두는 반면, Drain은 기존 Pod까지 다른 Node로 옮기며, DaemonSet Pod는 `--ignore-daemonsets` 옵션으로 이 과정에서 예외 처리된다.

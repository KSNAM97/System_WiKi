# K8s-05 쿠버네티스 Pod 개념과 동작 원리

## Pod란 무엇인가

**Pod**는 쿠버네티스에서 **컨테이너**를 실행하는 가장 작은 단위이며, 웹서버와 로그수집기처럼 여러 컨테이너를 하나로 묶거나 livenessProbe로 죽은 컨테이너를 자동 재시작하는 등 컨테이너 실행과 관련된 여러 기능이 이 Pod 단위를 중심으로 이루어진다.

쿠버네티스에서는 컨테이너(Docker 컨테이너)를 혼자서 바로 실행하지 않는다.
반드시 Pod라는 그릇 안에 담아서 실행한다.
즉, 컨테이너는 실행 파일이고 Pod는 컨테이너를 담는 실행 단위이다.

쿠버네티스 입장에서 컨테이너 하나 실행해줘가 아니라 Pod 하나 실행해줘라고 요청한다.

### 왜 컨테이너를 바로 실행하지 않고 Pod를 쓰는가

이유는 여러 컨테이너를 하나의 묶음으로 관리하기 위해서이다.
실제 서비스에서는 컨테이너 하나만 단독으로 동작하는 경우보다 
서로 밀접하게 연결된 컨테이너들이 함께 움직이는 경우가 많다.

예를 들어, 웹 서버 컨테이너 + 로그 수집 컨테이너
이 두 개는 항상 같이 실행되고 같이 종료되고 같은 네트워크를 사용해야 한다.

이때 쿠버네티스는 이 컨테이너들을 각각 따로 관리하지 않고 하나의 Pod 안에 넣어서 하나의 덩어리로 관리한다.
그래서 Pod는 컨테이너들을 묶은 단위이다.

### Pod 안에는 무엇이 들어 있는가

Pod 안에는 다음과 같은 것들이 들어 있다.
- 하나 이상의 컨테이너
- 공통 네트워크(IP, Port 공간)
- 공통 스토리지(볼륨)
- 컨테이너 실행 규칙

중요한 포인트는 Pod 안의 모든 컨테이너는 같은 **IP**를 사용한다는 것이다.
그래서 Pod 안의 컨테이너들은 **localhost**로 서로 통신할 수 있다.
이것이 Pod를 쓰는 핵심 이유 중 하나다.

**정리:** Pod는 하나 이상의 컨테이너와 네트워크·스토리지를 함께 묶어 관리하는 쿠버네티스의 최소 실행 단위이며, 같은 Pod 안의 컨테이너들은 같은 IP를 공유해 localhost로 통신한다.

---

## Single Container Pod와 Multi Container Pod

Pod에는 컨테이너가 1개일 수도 있고 여러 개일 수도 있다.

**Single Container Pod**
- Pod 안에 컨테이너 1개
- 가장 기본적인 형태
- 연습, 테스트, 단순 서비스에서 주로 사용
- 예) nginx 컨테이너 1개만 들어 있는 Pod

**Multi Container Pod**
- Pod 안에 컨테이너 여러 개
- 서로 강하게 의존하는 컨테이너들을 묶는다.
- 예) 웹 서버 컨테이너 + 로그 수집 컨테이너
- 예) 웹 서버 컨테이너 + 프록시 컨테이너

### Pod는 어떻게 생성되는가

Pod는 보통 두 가지 방법으로 생성된다.

**명령어로 생성**
- kubectl run 명령을 사용하면 쿠버네티스가 내부적으로 Pod를 만든다.
- EX) kubectl  run  webserver  --image=nginx:latest
- EX) kubectl  get pods

**YAML 파일로 생성**
- Pod의 설정을 YAML 파일로 작성하고 kubectl apply로 생성한다.
- EX) kubectl  create  -f  webserver.nginx.yaml
- EX) kubectl  get pods

쿠버네티스에서 가장 정석적인 방법은 **YAML**이다.
- YAML 안에는 어떤 이미지로 어떤 컨테이너를 어떤 이름의 Pod로 실행할지 작성되어 있다.

### Pod의 생명 주기

- Pod는 영구적인 존재가 아니다.
- Pod는 언제든지 삭제될 수 있고 재생성될 수 있고 다른 노드로 옮겨질 수 있다.
- Pod가 죽으면 같은 Pod가 살아나는 것이 아니라 새 Pod가 만들어진다.
 그래서 Pod의 IP도 바뀐다.
- 이것이 중요한 이유는 Pod 자체를 믿고 서비스하면 안 되기 때문이다.
- 실무에서는 Pod 위에 **Deployment**, **Service** 같은 개념을 얹어서 안정적인 서비스를 만든다.

### Pod와 Node의 관계

Pod는 반드시 **Node**(워커 노드) 위에서 실행된다.
하지만 사용자는 이 Pod를 이 노드에 실행해라라고 직접 지정하지 않는다.

쿠버네티스의 **Scheduler**가 현재 여유 있는 노드를 골라 Pod를 배치한다.
즉, Pod는 실행 단위 Node는 실행 장소 라고 생각하면 된다.

**정리:** Pod는 컨테이너 개수에 따라 Single/Multi로 나뉘고 명령어 또는 YAML로 생성하며, 영구적이지 않고 Scheduler가 배치하는 임시 실행 단위이므로 Deployment·Service 같은 상위 개념으로 안정성을 확보한다.

---

## CLI를 사용한 pod 생성

`kubectl run` 명령으로 **Pod**를 즉시 생성하는 방법이다.

```bash
[root@k8s-master ~]# kubectl  run  web1  --image=nginx:1.31  --port 80
pod/web1 created


[root@k8s-master ~]# kubectl  get  namespace
NAME              	STATUS   AGE
default           	Active      2d19h
kube-flannel      	Active      2d19h
kube-node-lease   	Active      2d19h
kube-public       	Active      2d19h
kube-system       	Active      2d19h
resource          	Active      24h
soldesk           	Active      43h
studydesk         	Active      43h



[root@k8s-master ~]# kubectl  get  pods
NAME   READY   STATUS    RESTARTS   AGE
web1     1/1        Running     0                 3s
```

---

## YAML 파일을 사용한 pod 생성

**YAML** 파일에 Pod 스펙을 정의하고 `kubectl apply`로 생성하는, 쿠버네티스에서 가장 정석적인 방식이다.

```bash
[root@k8s-master ~]# vi pod-nginx.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
  - name: nginx-container
    image: nginx:1.29.1
    ports:
    - containerPort: 80
      protocol: TCP



[root@k8s-master ~]# kubectl  apply  -f   pod-nginx.yaml  --dry-run=server
pod/nginx-pod created (server dry run)


[root@k8s-master ~]# kubectl  get  pods
NAME   	   READY   STATUS    RESTARTS   AGE
nginx-pod	   1/1        Running      0                22s
web1   	   1/1        Running      0                3m25s
```

---

## CLI를 YAML 파일로 생성해서 pod 생성

이미 실행 중인 Pod의 YAML을 `kubectl get pods -o yaml`로 뽑아내어 다른 Pod의 YAML로 재활용하는 방법이다.

```bash
[root@k8s-master ~]# kubectl  get  pods  web1  -o  yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: "2026-08-14T02:00:47Z"
  generation: 1
  labels:
    run: web1
  name: web1
  namespace: default
  resourceVersion: "112221"
  uid: 2c5ecc32-463c-4ba4-89f7-3c844722d26e
spec:
  containers:
  - image: nginx:1.31
    imagePullPolicy: IfNotPresent
    name: web1
    ports:
    - containerPort: 80
      protocol: TCP
~~~~~~~ 중간 생략 ~~~~~~~



[root@k8s-master ~]# vi  web1-copy-nginx.yaml
apiVersion: v1
kind: Pod
metadata:
  name: web1-copy
  namespace: default
spec:
  containers:
  - image: nginx:1.31
    name: web1
    ports:
    - containerPort: 80
      protocol: TCP

: wq



[root@k8s-master ~]# kubectl  apply  -f  web1-copy-nginx.yaml
pod/web1-copy created
```

---

## multi container pod 생성

하나의 Pod 안에 **nginx-container**와 **centos-container** 두 개의 컨테이너를 함께 넣어 **Multi Container Pod**를 만드는 실습이다.

```bash
[root@k8s-master ~]# vi  multi-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: multipod
spec:
  containers:
  - name: nginx-container
    image: nginx:1.29.1
    ports:
    - containerPort: 80
  - name: centos-container
    image: centos:7
    command:
    - sleep
    - "10000"


[root@k8s-master ~]# kubectl  apply  -f  multi-pod.yaml


[root@k8s-master ~]# kubectl  get pods
NAME     		READY   STATUS              	RESTARTS   AGE
multipod    	0/2         ContainerCreating	0                5s
nginx-pod   	1/1         Running             	0                12m
web1        	1/1         Running             	0                15m
web1-copy	1/1         Running             	0                8m



[root@k8s-master ~]# kubectl  get  pods
NAME        	READY   STATUS 	RESTARTS   AGE
multipod    	2/2        Running   	0                 13s
nginx-pod   	1/1        Running 	0                 15m
web1        	1/1        Running 	0                 18m
web1-copy   	1/1        Running 	0                 10m




[root@k8s-master ~]# kubectl  get  pods  -o  wide
NAME        	READY   STATUS 	RESTARTS   AGE   IP              NODE          NOMINATED NODE   READINESS GATES
multipod    	2/2         Running	0                73s     10.244.2.5   k8s-worker2   <none>                   <none>
nginx-pod   	1/1         Running 	0                16m    10.244.2.2   k8s-worker2   <none>                   <none>
web1        	1/1         Running  	0                19m    10.244.1.6   k8s-worker1   <none>                   <none>
web1-copy	1/1         Running 	0                11m    10.244.1.7   k8s-worker1   <none>                   <none>



[root@k8s-master ~]# kubectl  describe  pods
Name:             	multipod
Namespace:    	default
Priority:         	0
Service Account: 	default
Node:             	k8s-worker2/192.168.10.102
Start Time:       	Fri, 14 Aug 2026 11:19:08 +0900
Labels:           	<none>
Annotations:      	<none>
Status:           	Running
IP:               	10.244.2.5
IPs:
  IP:  10.244.2.5
Containers:
  nginx-container:
    Container ID:	containerd://9f2e78cad9c6ae234e6533bc906827b6df03fd6d7d6905653f0b6cd2e499f796
    Image:          	nginx:1.29.1
    Image ID:       	docker.io/library/nginx@sha256:8adbdcb969e2676478ee2c7ad333956f0c8e0e4c5a7463f4611d7a2e7a7ff5dc
    Port:           	80/TCP
    Host Port:     	 0/TCP
    State:          	Running
      Started:      	Fri, 14 Aug 2026 11:19:09 +0900
    Ready:          	True
    Restart Count:  	0
    Environment:    	<none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-26k72 (ro)
  centos-container:
    Container ID:  	containerd://92d426f49ba7bc00d545b37d1b71f216ab22af336117404436dc676eb05627d9
    Image:         	centos:7
    Image ID:      	docker.io/library/centos@sha256:be65f488b7764ad3638f236b7b515b3678369a5124c47b8d32916d6487418ea4
    Port:          	<none>
    Host Port:     	<none>
    Command:
      sleep
      10000
    State:          	Running
      Started:      	Fri, 14 Aug 2026 11:19:17 +0900
    Ready:          	True
    Restart Count:  	0
    Environment:    	<none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-26k72 (ro)
~~~~~~~~~~~~~~~~~~~~~~~~~ 중간 생략 ~~~~~~~~~~~~~~~~~~~~~~~~~
```

---

## 컨테이너로 접속

`kubectl exec -c`로 Multi Container Pod 안의 특정 컨테이너에 접속해 웹 페이지를 수정하고 로그를 확인하는 실습이다.

**정리:** 여기까지 CLI/YAML을 통한 Pod 생성 방법과 Single/Multi Container Pod 구성, 컨테이너 접속 방법을 살펴봤다. 다음은 Pod가 실제로 생성되어 Running 상태가 되기까지의 내부 동작 흐름을 다룬다.

```bash
[root@k8s-master ~]# cat  multi-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: multipod		# pod 명
spec:
  containers:
  - name: nginx-container	# container 명
    image: nginx:1.29.1
    ports:
    - containerPort: 80
  - name: centos-container	# container 명
    image: centos:7
    command:
    - sleep
    - "10000"


[root@k8s-master ~]# kubectl  exec  multipod  -c  nginx-container  -it --  /bin/bash
root@multipod:/#
root@multipod:/# cat  /usr/share/nginx/html/index.html
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
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>




root@multipod:/# apt-get  update

root@multipod:/# apt-get  install  -y  vim

root@multipod:/# vi  /usr/share/nginx/html/index.html
<!DOCTYPE html>
<html>
<head>
    <title>My Web Page</title>
</head>
<body>

    <h1>Hello Kubernetes</h1>
    <p>간단한 테스트 페이지입니다.</p>

</body>
</html>



root@multipod:/# exit



[root@k8s-master ~]# kubectl  get  pods  -o  wide
NAME        	READY   STATUS    RESTARTS   AGE   IP           NODE          NOMINATED NODE   READINESS GATES
multipod    	2/2     Running   0          24m   10.244.2.5   k8s-worker2   <none>           <none>
nginx-pod   	1/1     Running   0          39m   10.244.2.2   k8s-worker2   <none>           <none>
web1        	1/1     Running   0          42m   10.244.1.6   k8s-worker1   <none>           <none>
web1-copy   	1/1     Running   0          34m   10.244.1.7   k8s-worker1   <none>           <none>
[root@k8s-master ~]#



[root@k8s-master ~]# curl  http://10.244.2.5
<!DOCTYPE html>
<html>
<head>
    <title>My Web Page</title>
</head>
<body>

    <h1>Hello Kubernetes</h1>
    <p>간단한 테스트 페이지입니다.</p>

</body>
</html>



[root@k8s-master ~]# kubectl  logs  multipod -c  nginx-container
/docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
/docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
/docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
10-listen-on-ipv6-by-default.sh: info: Getting the checksum of /etc/nginx/conf.d/default.conf
10-listen-on-ipv6-by-default.sh: info: Enabled listen on IPv6 in /etc/nginx/conf.d/default.conf
/docker-entrypoint.sh: Sourcing /docker-entrypoint.d/15-local-resolvers.envsh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/30-tune-worker-processes.sh
/docker-entrypoint.sh: Configuration complete; ready for start up
2026/08/14 02:19:09 [notice] 1#1: using the "epoll" event method
2026/08/14 02:19:09 [notice] 1#1: nginx/1.29.1
2026/08/14 02:19:09 [notice] 1#1: built by gcc 12.2.0 (Debian 12.2.0-14+deb12u1)
2026/08/14 02:19:09 [notice] 1#1: OS: Linux 5.14.0-687.36.1.el9_8.x86_64
2026/08/14 02:19:09 [notice] 1#1: getrlimit(RLIMIT_NOFILE): 1024:524288
2026/08/14 02:19:09 [notice] 1#1: start worker processes
2026/08/14 02:19:09 [notice] 1#1: start worker process 29
2026/08/14 02:19:09 [notice] 1#1: start worker process 30
2026/08/14 02:19:09 [notice] 1#1: start worker process 31
2026/08/14 02:19:09 [notice] 1#1: start worker process 32
10.244.0.0 - - [14/Aug/2026:02:41:38 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.76.1" "-"
10.244.0.0 - - [14/Aug/2026:02:43:38 +0000] "GET / HTTP/1.1" 200 175 "-" "curl/7.76.1" "-"
10.244.0.0 - - [14/Aug/2026:02:48:45 +0000] "GET / HTTP/1.1" 200 175 "-" "curl/7.76.1" "-"
```

---

## Pod 동작 flow

### 1) 사용자가 Pod 생성 요청

```bash
# kubectl apply -f pod.yaml

    또는

# kubectl run nginx --image=nginx
```

사용자가 Pod 만들어줘"라고 **API Server**에게 요청
- kubectl은 Pod를 직접 만들지 않는다.
- kubectl은 그냥 요청 도구이다.

### 2) API Server가 kubectl 요청을 받는다.

**API Server 역할**
- 요청 문법 검사
- YAML 문법 맞는지
- apiVersion, kind, spec 구조 맞는지

**권한 검사**
- 이 사용자가 Pod 만들 권한 있는지

이상 없으면 **etcd**에 저장

이 상태의 Pod는
- 상태 : Pending
- 이유 : 아직 실행할 노드가 정해지지 않음

### 3) Scheduler가 노드를 선택

**Scheduler**는 아직 노드가 정해지지 않은 Pod 있나 계속 확인한다.

노드가 정해지지 않은 Pod있으면 아래 기준으로 노드를 고른다.
- CPU, 메모리 여유
- nodeSelector, affinity 조건
- taint / toleration
- 기존 Pod 분포

위의 기준에 의해 Pod는 worker-node-X 를 선택하고 그 결과를 API Server에 다시 알려준다.

### 4) 선택된 노드의 kubelet이 감지

worker-node-X에는 **kubelet**이 항상 실행 중이다.

kubelet은 내 노드에서 실행해야 할 Pod 있나 계속 감시한다.

API Server에서 worker-node-X 이 Pod 실행해 라는 정보를 받으면 실행 시작

### 5) 컨테이너 생성 준비

**kubelet이 하는 일**
- Pod 스펙 읽기
- 파드가 이렇게 생겼어
- 컨테이너는 몇 개고
- 이미지는 뭐고
- 포트, 볼륨은 이렇게 써를 판단하고 지시하는 프로그램이다.

kubelet은
- 이미지 다운로드
- 컨테이너 프로세스 생성
- 네임스페이스 / cgroup 설정
- 실제 리눅스 프로세스 실행 같은 저수준 리눅스 작업을 직접 하지 않는다.

**컨테이너 런타임**은 컨테이너 만드는 전문 엔진이다.

예시
- containerd
- CRI-O

컨테이너 런타임은
- 이미지 pull
- 컨테이너 생성
- 컨테이너 실행
- 컨테이너 중지 / 삭제같은 작업을 수행한다.

### 6) 이미지 다운로드

컨테이너 런타임 동작
- 이미지가 로컬에 이미지 있으면 바로 사용하고 없으면 레지스트리에서 pull한다.
- 이미지 다운로드 완료 후 컨테이너를 생성하고 실행한다.

### 7) Pod Running 상태

컨테이너가 정상 실행
- kubelet --> API Server에 상태 보고

Pod 상태 변화
- Pending --> Running

결과

```bash
kubectl get pods
STATUS: Running
```

### Pod 실행 이후 지속 동작

Pod가 실행된 이후에도 kubelet은 계속 감시한다.
- 컨테이너 죽었나? (컨테이너 프로세스가 종료되었는지 확인)
- **readiness probe** 실패? (이 컨테이너가 서비스 요청을 받을 준비가 되었는지 확인)
- **liveness probe** 실패? (이 컨테이너가 정상적으로 살아있는지 확인)
   - 프로세스는 살아 있지만 응답이 없는 경우
   - liveness probe 실패 시 kubelet이 컨테이너를 삭제 후 컨테이너를 다시 생성한

상황에 따라
- 컨테이너 재시작
- Pod 재생성 (Deployment라면)

**정리:** Pod 생성은 kubectl 요청 → API Server 검증/etcd 저장 → Scheduler의 노드 선택 → kubelet 감지 → 컨테이너 런타임의 이미지 pull/컨테이너 생성 → Running 상태 순으로 진행되며, 이후에도 kubelet이 readiness/liveness probe로 지속 감시한다.

---

## 쿠버네티스 Pod - livenessProbe

### livenessProbe를 이용해서 Self-healing Pod 만들기

실제 서버에서 가장 무서운 상황은 프로그램이 멈췄는데 서버는 살아 있는 것처럼 보이는 상태다.

예제 상황
- 웹 서버 프로세스는 떠 있다.
- 포트도 열려 있다.
- 그런데 요청을 보내면 응답이 없다.
- 로그도 안 찍힌다.

이걸 사람 손으로 관리하면 왜 안되는지를 찾고 시스템을 재부팅하는 상황이 반복될 수 있다.
쿠버네티스는 이걸 사람이 아니라 시스템이 자동으로 처리하도록 설계되어 있다.
그 핵심이 바로 **livenessProbe**, **Self-healing Pod**이다.

### livenessProbe란 무엇인가

**livenessProbe**를 한 문장으로 말하면 이렇다.
- 컨테이너가 아직 살아있는 상태인가를 쿠버네티스가 직접 확인하는 방법

여기서 중요한 포인트는 프로세스가 떠 있냐가 아니라 정상적으로 살아있냐를 본다는 점이다.
프로세스는 살아 있는데 무한 루프에 빠졌거나 데드락 상태이거나 외부 요청에 전혀 응답하지 못하는 상태
이런 경우는 실제로는 죽은 것과 다름없다.

livenessProbe는 바로 이 상태를 잡아내기 위한 장치다.

### kubelet이 livenessProbe를 사용하는 방식

Pod가 워커 노드에서 실행되면 해당 노드의 **kubelet**은 가만히 있는 게 아니라
kubelet은 정해진 주기마다 컨테이너에게 정상적으로 동작 중인지를 확인한다. 

이 질문을 하는 방식이 바로 livenessProbe다.
Probe 결과는 단순하다.
- 요청 메시지 전송 후 응답 메세지를 수신 성공시  -->  정상
- 요청 메시지 전송 후 응답 메세지를 수신 실패시  -->  비정상

livenessProbe가 실패하면 kubelet은 해당 컨테이너는 살아있는 상태가 아니라고 판단하고
해당 컨테이너를 종료시킨 후 컨테이너를 다시 생성한다.

Pod 전체를 지우는 게 아니라 컨테이너만 재 시작한다.
즉, Pod IP는 유지, Pod는 그대로 유지 컨테이너만 새로 생성한다.

이게 바로 자동 복구의 시작점이다.

### Self-healing Pod란 무엇인가

**Self-healing Pod**는 특정 기능이 아니라 개념이다.
- 문제가 발생했을 때 사람 개입 없이 쿠버네티스가 스스로 정상 상태로 되돌리는 구조
- Self-healing은 한 가지 기능으로 이루어지지 않는다. (여러 요소가 합쳐진 결과)

### Self-healing이 이루어지는 실제 흐름

-하나의 시나리오로 보자.

```
1) Pod 안의 웹 애플리케이션이 멈춘다
2) 컨테이너는 떠 있지만 응답이 없다
3) kubelet이 livenessProbe 체크
4) probe 실패 감지
5) kubelet이 컨테이너 종료
6) 새 컨테이너 생성
7) 서비스 정상화
```

이 과정에서 관리자는 아무것도 하지 않았다.
이게 바로 Self-healing Pod의 핵심이다.

### livenessProbe가 Self-healing의 핵심인 이유

livenessProbe가 없다면
1) 컨테이너는 멈췄다, 하지만 쿠버네티스는 컨테이너가 떠 있으니까 괜찮겠지라고 판단할 수 있다.
2) Pod는 계속 Running
3) 서비스는 장애 상태

즉, 자동 복구 자체가 일어나지 않는다.

### livenessProbe와 readinessProbe의 결정적 차이

**readinessProbe**
- 지금 이 컨테이너가 요청을 받아도 돼?

**livenessProbe**
- 이 컨테이너, 살아있긴 한 거야?

livenessProbe와 readinessProbe는 요청이 다르기 때문에 결과도 다르다.
- readiness 실패	: 트래픽만 차단
- liveness 실패	: 컨테이너 재시작

Self-healing은 livenessProbe가 있어야만 가능하다.

**정리:** livenessProbe는 컨테이너의 생존 여부를 kubelet이 주기적으로 확인해 실패 시 컨테이너를 재생성하는 장치이며, 트래픽만 차단하는 readinessProbe와 달리 재시작까지 이어지는 Self-healing Pod의 핵심 요소다.

---

## livenessProbe 메커니즘

livenessProbe는 컨테이너가 정상적으로 살아있는지를 쿠버네티스가 주기적으로 확인하는 기능이다.
정상으로 판단되지 않으면 해당 컨테이너를 종료하고 다시 시작한다. livenessProbe는 확인 방식에 따라 아래 세 가지로 나뉜다.

### 1) httpGet probe

- 지정한 IP 주소, port, path로 HTTP GET 요청을 보낸다.
- HTTP 응답 코드가 200번대이면 정상으로 판단한다.
- 응답이 없거나 오류 코드가 반환되면 비정상으로 판단하고 컨테이너를 다시 시작한다.
- nginx, apache, spring boot, node.js 같은 웹 애플리케이션은 항상 HTTP 요청을 받아서 응답해야 정상이다.

```yaml
livenessProbe:
  httpGet:
    path: /
    port: 80
```

### 2) tcpSocket probe

- 지정된 port로 TCP 연결을 시도 후 TCP 연결이 성공하면 정상으로 판단한다.
- 연결이 실패하면 비정상으로 판단하고 컨테이너를 다시 시작한다.
- HTTP 응답까지는 필요 없고 서비스가 떠 있는지만 확인하면 되는 경우에 사용한다.
  - MySQL, MariaDB, PostgreSQL
  - Redis
  - SSH 데몬
  - 단순 TCP 서버

```yaml
livenessProbe:
  tcpSocket:
    port: 22
```

### 3) exec probe

- 컨테이너 내부에서 지정한 명령어를 실행한다.
- 명령어 실행 결과의 종료 코드가(exit code) 0이면 정상으로 판단한다.
- 종료 코드가 0이 아니면 비정상으로 판단하고 컨테이너를 다시 시작한다.
- 컨테이너 내부 상태까지 직접 검사해야 할 때 쓰는 방식이다.
  - 특정 파일이 존재해야 정상
  - 특정 프로세스가 떠 있어야 정상
  - 내부 스크립트 결과가 정상이어야 정상

```yaml
livenessProbe:
  exec:
    command:
      - /bin/sh
      - -c
      - cat /data/file
```

**정리:** livenessProbe는 httpGet, tcpSocket, exec 세 방식 중 하나로 컨테이너의 생존 상태를 확인하며, 다음 절부터는 initialDelaySeconds 등 실제 파라미터 설정과 실습 예제를 다룬다.

---

## livenessProbe 기본 설정

```bash
[root@k8s-master ~]# vi pod-nginx-liveness.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod-liveness
spec:
  containers:
  - name: nginx-container
    image: nginx:1.31
    ports:
    - containerPort: 80
      protocol: TCP
    livenessProbe:
      httpGet:
        path: /
        port: 80


[root@k8s-master ~]# kubectl  apply  -f  pod-nginx-liveness.yaml  --dry-run=server
pod/nginx-pod-liveness created (server dry run)




[root@k8s-master ~]# kubectl  get  pods  -o  wide  --watch



[root@k8s-master ~]# kubectl  apply  -f  pod-nginx-liveness.yaml
pod/nginx-pod-liveness created



[root@k8s-master ~]# kubectl  get  pods  -o  wide  --watch
NAME                 	READY   STATUS    	RESTARTS   AGE   IP              NODE           NOMINATED NODE   READINESS GATES
nginx-pod-liveness	0/1         Pending   		0                0s      <none>        <none>         <none>                   <none>
nginx-pod-liveness	0/1         Pending   		0                0s      <none>        k8s-worker1   <none>                  <none>
nginx-pod-liveness 	0/1         ContainerCreating	0                0s      <none>        k8s-worker1   <none>                  <none>
nginx-pod-liveness	1/1         Running             	0                1s      10.244.1.11   k8s-worker1   <none>                  <none>




[root@k8s-master ~]# kubectl  describe  pods  nginx-pod-liveness | grep Liveness
    Liveness:       http-get http://:80/ delay=0s timeout=1s period=10s #success=1 #failure=3


-initialDelaySeconds: 15	# 컨테이너 시작 후 15초 뒤부터 probe 시작	(default = 0sec)
-timeoutSeconds: 1 	 	# 1초 안에 응답 없으면 실패로 판단		(default = 1sec)
-periodSeconds: 20	 	# 20초마다 livenessProbe 실행			(default = 10sec)
-successThreshold: 1 	# 1번 성공하면 정상으로 판단			(default = 1)
-failureThreshold: 3     	# 3번 연속 실패하면 컨테이너 재시작		(default = 3)
```

```yaml
# livenessProbe 파라미터 추가 설정
[root@k8s-master ~]# vi pod-nginx-liveness.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod-liveness
spec:
  containers:
  - name: nginx-container
    image: nginx:1.31
    ports:
    - containerPort: 80
      protocol: TCP
    livenessProbe:
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 10
      periodSeconds: 10
      timeoutSeconds: 1
      failureThreshold: 3
      successThreshold: 1
```

```bash
[root@k8s-master ~]# kubectl  delete  -f  pod-nginx-liveness.yaml
pod "nginx-pod-liveness" deleted from default namespace (server dry run)



[root@k8s-master ~]# kubectl  apply  -f  pod-nginx-liveness.yaml  --dry-run=server
pod/nginx-pod-liveness created (server dry run)



[root@k8s-master ~]# kubectl  apply  -f  pod-nginx-liveness.yaml
pod/nginx-pod-liveness created




[root@k8s-master ~]# kubectl  describe  pods  nginx-pod-liveness | grep Liveness
    Liveness:       http-get http://:80/ delay=10s timeout=1s period=10s #success=1 #failure=3




[root@k8s-master ~]# kubectl  exec  nginx-pod-liveness  -it  -- /bin/bash
root@nginx-pod-liveness:/#
root@nginx-pod-liveness:/# nginx  -s  stop
2026/08/14 04:06:36 [notice] 40#40: signal process started
root@nginx-pod-liveness:/# command terminated with exit code 137



	# 메인 프로세스가 삭제되어 파드가 재시작
[root@k8s-master ~]# kubectl  get  pods  -o  wide  --watch
NAME                	READY   STATUS      RESTARTS   	AGE     IP            	  NODE           NOMINATED NODE   READINESS GATES
nginx-pod-liveness	1/1         Running      0          	73s       10.244.1.12	  k8s-worker1   <none>                   <none>
nginx-pod-liveness 	0/1         Completed   0          	2m32s   10.244.1.12	  k8s-worker1   <none>                   <none>
nginx-pod-liveness 	1/1         Running      1 (1s ago)   	2m33s   10.244.1.12	  k8s-worker1   <none>                   <none>
```

---

## /health 경로를 이용한 livenessProbe 실습

`httpGet` probe의 검사 경로를 `/health`로 지정해, 해당 경로에 파일이 있는지에 따라 probe 성공/실패가 어떻게 갈리는지 확인하는 실습이다.

```yaml
# pod-nginx-liveness.yaml (health 경로 변경)
[root@k8s-master ~]# vi pod-nginx-liveness.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod-liveness
spec:
  containers:
  - name: nginx-container
    image: nginx:1.31
    ports:
    - containerPort: 80
      protocol: TCP
    livenessProbe:
      httpGet:
        path: /usr/share/nginx/html/health		# 경로 변경
        port: 80
      initialDelaySeconds: 10
      periodSeconds: 10
      timeoutSeconds: 1
      failureThreshold: 3
      successThreshold: 1
```

```bash
[root@k8s-master ~]# kubectl  exec  nginx-pod-liveness  -- sh -c "echo OK >  /usr/share/nginx/html/health"


[root@k8s-master ~]# kubectl  delete  -f  pod-nginx-liveness.yaml
pod "nginx-pod-liveness" deleted from default namespace


[root@k8s-master ~]# kubectl  apply  -f  pod-nginx-liveness.yaml
pod/nginx-pod-liveness created



[root@k8s-master ~]# kubectl  exec  nginx-pod-liveness  -it  -- /bin/bash


root@nginx-pod-liveness:/# ls -l  /usr/share/nginx/html/
total 12
-rw-r--r-- 1 root root 497 Jul 15 16:03 50x.html
-rw-r--r-- 1 root root   3 Aug 14 04:12 health
-rw-r--r-- 1 root root 896 Jul 15 16:03 index.html


root@nginx-pod-liveness:/# cat  /usr/share/nginx/html/health
OK

root@nginx-pod-liveness:/# exit

[root@k8s-master ~]# kubectl  exec  nginx-pod-liveness  --  rm  /usr/share/nginx/html/health
```

**정리:** /health 파일이 있으면 200 OK로 probe가 성공하고, 파일을 지우면 probe가 실패해 컨테이너가 재시작되는 것을 실습으로 확인했다. 이어서 이 개념을 **커스텀 이미지**로 직접 빌드해 재현한다.

---

## EX1) 커스텀 이미지 + livenessProbe 실습

### EX1-1) nginx:1.31 이미지를 기반으로 사용자 정의 이미지를 생성하고, 이미지 내부에 /usr/share/nginx/html/health 파일이 기본으로 존재하도록 Dockerfile을 작성하시오

```bash
[root@k8s-master ~]# mkdir liveness-lab


[root@k8s-master ~]# cd liveness-lab


[root@k8s-master liveness-lab]# pwd
/root/liveness-lab



[root@k8s-master liveness-lab]# vi health
health OK



[root@k8s-master liveness-lab]# vi dockerfile
FROM  nginx:1.31

ENV LANG=C.UTF-8
ENV LC_ALL=C.UTF-8

COPY  health  /usr/share/nginx/html/health
```

### EX1-2) EX1-1에서 생성한 이미지를 빌드하고 Docker Hub에 업로드하시오

```bash
[root@k8s-master liveness-lab]# docker  build  -t  konan7979/nginx-liveness:1.0  .
[+] Building 1.9s (8/8) FINISHED                                                                                	docker:default
 => [internal] load build definition from dockerfile                                                                	0.0s
 => => transferring dockerfile: 136B                                                                                   	0.0s
 => [internal] load metadata for docker.io/library/nginx:1.31                                                  	1.8s
 => [auth] library/nginx:pull token for registry-1.docker.io                                                  	0.0s
 => [internal] load .dockerignore                                                                                  	0.0s
 => => transferring context: 2B                                                                                   	0.0s
 => [internal] load build context                                                                                         	0.0s
 => => transferring context: 43B                                                                                     	0.0s
 => CACHED [1/2] FROM docker.io/library/nginx:1.31@sha256:8541484afbc9c8a5a8a99b379568e	0.0s
 => => resolve docker.io/library/nginx:1.31@sha256:8541484afbc9c8a5a8a99b379568ebbc957f6585	0.0s
 => [2/2] COPY  health  /usr/share/nginx/html/health                                                         	0.0s
 => exporting to image                                                                                                  	0.1s
 => => exporting layers                                                                                                	0.0s
 => => exporting manifest sha256:a12719dc7b15eba3da57e90bdc60568c95613bcffa582a07ec31f7cb7	0.0s
 => => exporting config sha256:3a5175eb555a100852c3a22fa2583d6f1c9a0812c23aa5f07eda32b1fcab	0.0s
 => => exporting attestation manifest sha256:6e76d6c1769305225cd3afbe40e095a3567dc37528e72f	0.0s
 => => exporting manifest list sha256:75edbab6ae2a0b81b19ee784deddc4cff5afe7c81fcd789c379751	0.0s
 => => naming to docker.io/konan7979/nginx-liveness:1.0                                                       	0.0s
 => => unpacking to docker.io/konan7979/nginx-liveness:1.0                                                     	0.0s



[root@k8s-master liveness-lab]# docker  images
IMAGE                            	 	ID             	DISK USAGE	CONTENT SIZE   EXTRA
custom-nginx-web:1.31             	02218a3d52d6        	348MB           	97MB
konan7979/custom-nginx-web:1.31	02218a3d52d6        	348MB           	97MB
konan7979/nginx-liveness:1.0      	75edbab6ae2a        	235MB         	63.1MB



[root@k8s-master liveness-lab]# docker login
Authenticating with existing credentials... [Username: konan7979]

i Info → To login with a different account, run 'docker logout' followed by 'docker login'

Login Succeeded




[root@k8s-master liveness-lab]# docker  push konan7979/nginx-liveness:1.0
The push refers to repository [docker.io/konan7979/nginx-liveness]
6f8ee52d8f41: Pushed
44136fa355b3: Mounted from konan7979/ssh-probe
3c55dc422a81: Mounted from konan7979/custom-nginx-web
26c307b5e35a: Mounted from konan7979/custom-nginx-web
d84ae7b21412: Mounted from konan7979/custom-nginx-web
c0df8d325117: Mounted from konan7979/custom-nginx-web
b8b80b9bc028: Mounted from konan7979/custom-nginx-web
f5de6e85ac74: Mounted from konan7979/custom-nginx-web
5a4222b844e8: Mounted from konan7979/custom-nginx-web
df2401a872d9: Pushed
1.0: digest: sha256:75edbab6ae2a0b81b19ee784deddc4cff5afe7c81fcd789c379751a1679bf7a5 size: 856
```

### EX1-3) EX1-2에서 업로드한 이미지를 사용하는 Pod를 생성하고, /health 경로를 검사하는 livenessProbe를 설정하시오

```
Pod 이름      	: nginx-pod-liveness
Container 이름	: nginx
Probe 방식           	: httpGet
Path                 	: /health
Port                 	: 80
초기 대기시간     	: 10초
검사 주기             	: 5초
Timeout              	: 1초
연속 실패 횟수    	: 3회
```

```yaml
[root@k8s-master ~]# vi pod-nginx-liveness.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod-liveness

spec:
  containers:
  - name: nginx
    image: konan7979/nginx-liveness:1.0

    ports:
    - containerPort: 80

    livenessProbe:
      httpGet:
        path: /health
        port: 80
        scheme: HTTP
      initialDelaySeconds: 10
      periodSeconds: 5
      timeoutSeconds: 1
      successThreshold: 1
      failureThreshold: 3
```

```bash
[root@k8s-master liveness-lab]# kubectl  apply  -f  pod-nginx-liveness.yaml  --dry-run=server
pod/nginx-pod-liveness created (server dry run)



[root@k8s-master liveness-lab]# kubectl  apply  -f  pod-nginx-liveness.yaml
pod/nginx-pod-liveness created




[root@k8s-master liveness-lab]# kubectl  get  pods
NAME                 	READY   STATUS                RESTARTS   AGE
nginx-pod-liveness	0/1         ContainerCreating   0                 2s



[root@k8s-master liveness-lab]# kubectl  get  pods
NAME                 READY   STATUS    RESTARTS   AGE
nginx-pod-liveness   1/1      Running     0                 9s



[root@k8s-master liveness-lab]# kubectl  describe  pods | grep Liveness
    Liveness:       http-get http://:80/health delay=10s timeout=1s period=5s #success=1 #failure=3



[root@k8s-master liveness-lab]# kubectl  exec  nginx-pod-liveness   --  curl  http://localhost/health
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100    10  100    10    0     0  23696      0 --:--:-- --:--:-- --:--:-- 10000
health OK
```

### EX1-4) 생성한 Pod가 정상 실행되고 /health 경로의 Probe가 성공하는 것을 확인하시오

### EX1-5) 실행 중인 컨테이너에서 /usr/share/nginx/html/health 파일을 삭제하여 livenessProbe가 실패하도록 하시오

```bash
[root@k8s-master liveness-lab]# watch -n 1 kubectl get pods
Every 1.0s: kubectl get pods                               k8s-master: Fri Aug 14 15:29:19 2026

NAME                    READY   STATUS    RESTARTS   AGE
nginx-pod-liveness   1/1         Running     0                6m48s


        OR

[root@k8s-master liveness-lab]# kubectl get pod nginx-pod-liveness --watch
NAME                    READY   STATUS    RESTARTS   AGE
nginx-pod-liveness   1/1         Running     0                6m48s




	# 컨테이너 접속
[root@k8s-master liveness-lab]# kubectl exec nginx-pod-liveness   -it  --  /bin/bash
root@nginx-pod-liveness:/#
root@nginx-pod-liveness:/# ls  -l /usr/share/nginx/html/
total 12
-rw-r--r-- 1 root root 497 Jul 15 16:03 50x.html
-rw-r--r-- 1 root root  10 Aug 14 06:08 health		<--- 파일 확인
-rw-r--r-- 1 root root 896 Jul 15 16:03 index.html


root@nginx-pod-liveness:/# rm  -rf  /usr/share/nginx/html/health



	# health 파일이 없기때문에 health-check에 실패 (컨테이너 재실행으로 인해 컨테이너에서 자동으로 아웃)
root@nginx-pod-liveness:/# ls  -l /usr/share/nginx/html/
total 12
-rw-r--r-- 1 root root 497 Jul 15 16:03 50x.html
-rw-r--r-- 1 root root 896 Jul 15 16:03 index.html




[root@k8s-master liveness-lab]# kubectl  get  pods
NAME                    READY   STATUS    RESTARTS      AGE
nginx-pod-liveness   1/1         Running     1 (55s ago)      11m

-health-check에 실패했기 때문에 컨테이너를 재실행한다.




[root@k8s-master liveness-lab]# kubectl  describe  pods nginx-pod-liveness | tail -10
  Type     Reason     Age                    From               Message
  ----     ------     ----                   ----               -------
  Normal   Scheduled  13m                    default-scheduler  Successfully assigned default/nginx-pod-liveness to k8s-worker1
  Normal   Pulling    13m                    kubelet            spec.containers{nginx}: Pulling image "konan7979/nginx-liveness:1.0"
  Normal   Pulled     12m                    kubelet            spec.containers{nginx}: Successfully pulled image "konan7979/nginx-liveness:1.0" in 6.638s (6.638s including waiting). Image size: 63126097 bytes.
  Normal   Created    2m13s (x2 over 12m)    kubelet            spec.containers{nginx}: Container created
  Normal   Started    2m13s (x2 over 12m)    kubelet            spec.containers{nginx}: Container started
  Warning  Unhealthy  2m13s (x3 over 2m23s)  kubelet            spec.containers{nginx}: Liveness probe failed: HTTP probe failed with statuscode: 404
  Normal   Killing    2m13s                  kubelet            spec.containers{nginx}: Container nginx failed liveness probe, will be restarted
  Normal   Pulled     2m13s                  kubelet            spec.containers{nginx}: Container image "konan7979/nginx-liveness:1.0" already present on machine and can be accessed by the pod
```

### EX1-6) livenessProbe가 연속 실패한 후 컨테이너가 자동으로 재시작되는 것을 확인하시오

```bash
[root@k8s-master liveness-lab]# kubectl exec nginx-pod-liveness   -it  --  /bin/bash
root@nginx-pod-liveness:/#
root@nginx-pod-liveness:/# ls  -l /usr/share/nginx/html/
total 12
-rw-r--r-- 1 root root 497 Jul 15 16:03 50x.html
-rw-r--r-- 1 root root  10 Aug 14 06:08 health
-rw-r--r-- 1 root root 896 Jul 15 16:03 index.html

-다시 컨테이너에 접속하게되면 이미지로 컨테이너를 다시 만들기 때문에 health 파일이 확인된다.
```

**정리:** 커스텀 이미지에 /health 파일을 심어 httpGet probe를 통과시키고, 파일 삭제 시 kubelet이 컨테이너를 강제로 재시작(exit code 137)하는 전체 흐름을 확인했다.

---

## smlinux/unhealthy 이미지 실습

**smlinux/unhealthy**는 Kubernetes의 livenessProbe 실습용 교육 이미지이다.

- 일반적인 nginx 이미지처럼 계속 정상 응답하는 것이 아니라, 일정 조건 이후 일부러 비정상 응답을 반환하도록 만들어졌다.
- livenessProbe, readinessProbe, self-healing 같은 Kubernetes의 상태 점검과  자동 복구 기능을 설명하기 위한 데모 용도로 사용
- 실제 서비스 운영용 이미지가 아니라, Kubernetes가 컨테이너 장애를 감지하고  재시작하는 과정을 확인하기 위한 실습용 이미지
- HTTP 요청이 5번까지는 200 OK로 정상 응답하지만, 6번째 요청부터는 500 Internal Server Error를 반환하도록 동작
 따라서 livenessProbe가 주기적으로 HTTP 요청을 보내면, 처음에는 성공하다가 
 이후 실패가 누적되고, **failureThreshold** 조건을 만족하면 kubelet이 해당 컨테이너를 재시작

```yaml
[root@k8s-master ~]# vi  pod-nginx-unhealthy.yaml
apiVersion: v1
kind: Pod
metadata:
  name: unhealthy-pod-liveness
spec:
  containers:
  - name: unhealthy-container
    image: smlinux/unhealthy
    ports:
    - containerPort: 80
      protocol: TCP
    livenessProbe:
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 15
      periodSeconds: 10
      timeoutSeconds: 1
      successThreshold: 1
      failureThreshold: 3
```

```bash
[root@k8s-master ~]# kubectl  apply -f  pod-nginx-unhealthy.yaml  --dry-run=server
pod/unhealthy-pod-liveness created (server dry run)



[root@k8s-master ~]# kubectl  apply -f  pod-nginx-unhealthy.yaml
pod/unhealthy-pod-liveness created



[root@k8s-master ~]# kubectl  get  pods
NAME                     	READY   	STATUS              	RESTARTS      AGE
nginx-pod-liveness      	1/1     	Running             	1 (24m ago)      34m
unhealthy-pod-liveness	0/1     	ContainerCreating	0                   14s



[root@k8s-master ~]# kubectl  get  pods
NAME                     	READY   	STATUS      RESTARTS     	AGE
nginx-pod-liveness       	1/1     	Running       1 (24m ago)	35m
unhealthy-pod-liveness  	1/1     	Running       0                   	58s



[root@k8s-master ~]# kubectl  get  pods
NAME                     	READY 	STATUS      RESTARTS      	AGE
nginx-pod-liveness       	1/1     	Running       1 (26m ago)   	36m
unhealthy-pod-liveness   	1/1     	Running       1 (5s ago)    	2m15s



[root@k8s-master ~]# kubectl  describe  pods | tail -10
  Type     Reason     Age                  From               Message
  ----     ------     ----                 ----               -------
  Normal   Scheduled  3m24s                default-scheduler  Successfully assigned default/unhealthy-pod-liveness to k8s-worker2
  Normal   Pulled     2m53s                kubelet  	  spec.containers{unhealthy-container}: Successfully pulled image "smlinux/unhealthy" in 31.181s (31.181s including waiting). Image size: 263841919 bytes.
  Normal   Pulling    74s (x2 over 3m24s)  kubelet	  spec.containers{unhealthy-container}: Pulling image "smlinux/unhealthy"
  Normal   Created    72s (x2 over 2m53s)  kubelet	  spec.containers{unhealthy-container}: Container created
  Normal   Started    72s (x2 over 2m53s)  kubelet	  spec.containers{unhealthy-container}: Container started
  Normal   Pulled     72s                  kubelet     	  spec.containers{unhealthy-container}: Successfully pulled image "smlinux/unhealthy" in 1.515s (1.515s including waiting). Image size: 263841919 bytes.
  Warning  Unhealthy  4s (x6 over 2m24s)   kubelet	  spec.containers{unhealthy-container}: Liveness probe failed: Get "http://10.244.2.8:80/": dial tcp 10.244.2.8:80: connect: connection refused
  Normal   Killing    4s (x2 over 104s)    kubelet	  spec.containers{unhealthy-container}: Container unhealthy-container failed liveness probe, will be restarted



[root@k8s-master ~]# kubectl  get  pods
NAME                     	READY  	STATUS    RESTARTS      AGE
nginx-pod-liveness       	1/1     	Running     1 (28m ago)      39m
unhealthy-pod-liveness	1/1     	Running     2 (46s ago)       4m36s




[root@k8s-master ~]# kubectl  delete  pods nginx-pod-liveness
pod "nginx-pod-liveness" deleted from default namespace


[root@k8s-master ~]# kubectl  delete  unhealthy-pod-liveness
error: the server doesn't have a resource type "unhealthy-pod-liveness"
```

**정리:** smlinux/unhealthy 이미지로 일정 횟수 이후 응답이 실패로 바뀌는 상황을 재현해, failureThreshold가 쌓이면 kubelet이 컨테이너를 재시작(RESTARTS 증가)하는 것을 확인했다. 다음은 httpGet이 아닌 **tcpSocket** 방식의 probe 실습이다.

---

## EX) tcpSocket Probe를 이용한 SSH 서비스 장애 감지 및 자동 복구

- SSH 서버가 실행되는 Pod를 생성한다.
- SSH 서비스는 기본적으로 22번 포트를 사용하며, **tcpSocket** 방식의 livenessProbe를 이용하여 22번 포트가 정상적으로 열려 있는지 확인한다.
- 컨테이너 내부에 접속하여 /etc/ssh/sshd_config 파일의 SSH 포트를 수동으로 22 --> 2222로 변경하고 SSH 서비스를 다시 시작한다.

```
1. SSH가 22번 포트를 사용할 때 tcpSocket Probe 성공
2. SSH 포트를 2222번으로 변경
3. tcpSocket Probe는 계속 22번 포트를 검사
4. Probe 실패
5. Kubernetes가 컨테이너를 재시작
6. 컨테이너가 이미지의 원래 설정인 SSH 22번으로 시작
7. tcpSocket Probe 다시 성공


-Pod 이름		: ssh-probe
-컨테이너 이름	: ssh-server
-SSH 기본 Port	: 22
```

### 작업 디렉터리 생성

```bash
[root@k8s-master ~]# mkdir tcp-probe-lab

[root@k8s-master ~]# cd tcp-probe-lab

[root@k8s-master tcp-probe-lab]# pwd
/root/tcp-probe-lab
```

### Dockerfile 생성

```bash
[root@k8s-master tcp-probe-lab]# vi dockerfile
FROM rockylinux:9

RUN dnf install -y openssh-server iproute procps-ng vim-minimal && \
    dnf clean all && \
    ssh-keygen -A

EXPOSE 22

CMD ["/bin/bash", "-c", "/usr/sbin/sshd && exec tail -f /dev/null"]

: wq


-tail -f /dev/null = 컨테이너를 계속 실행시키는 메인 프로세스 (실습 중 sshd를 중지해도 컨테이너 자체는 바로 종료되지 않는다.)
```

### Image Build 및 hub.docker.com에 이미지 PUSH

```bash
[root@k8s-master tcp-probe-lab]# docker  build  -t  konan7979/ssh-probe:1.0  .
[+] Building 1.9s (7/7) FINISHED                                                 docker:default
 => [internal] load build definition from dockerfile                                       0.0s
 => => transferring dockerfile: 248B                                                       0.0s
 => [internal] load metadata for docker.io/library/rockylinux:9                            1.6s
 => [auth] library/rockylinux:pull token for registry-1.docker.io                          0.0s
 => [internal] load .dockerignore                                                          0.0s
 => => transferring context: 2B                                                            0.0s
 => [1/2] FROM docker.io/library/rockylinux:9@sha256:d7be1c094cc5845ee815d4632fe377514ee6  0.0s
 => => resolve docker.io/library/rockylinux:9@sha256:d7be1c094cc5845ee815d4632fe377514ee6  0.0s
 => CACHED [2/2] RUN dnf install -y openssh-server iproute procps-ng vim-minimal &&     d  0.0s
 => exporting to image                                                                     0.3s
 => => exporting layers                                                                    0.0s
 => => exporting manifest sha256:833686fdd0e0174c5900eaf080cb9502ceada2452682ad4d04f3acad  0.0s
 => => exporting config sha256:23bcbc653110eb6719c37f59268047d99985d845ee4d48c7f9186664b6  0.0s
 => => exporting attestation manifest sha256:ee0a74b1440c8d271aa1df2fc613a93598c07529e4f1  0.0s
 => => exporting manifest list sha256:6578cbc133e376b889836ab3406bed26cad7d5941bd3f792c75  0.0s
 => => naming to docker.io/konan7979/ssh-probe:1.0                                         0.0s
 => => unpacking to docker.io/konan7979/ssh-probe:1.0                                      0.3s



[root@k8s-master tcp-probe-lab]# docker  push  konan7979/ssh-probe:1.0
The push refers to repository [docker.io/konan7979/ssh-probe]
4dfcf8f9dcf3: Pushed
44136fa355b3: Already exists
83781c17e4da: Layer already exists
446f83f14b23: Layer already exists
1.0: digest: sha256:6578cbc133e376b889836ab3406bed26cad7d5941bd3f792c75a2ae220c9a86c size: 855
[root@k8s-master tcp-probe-lab]#
```

### tcpSocket Probe Pod 작성 및 실행

```yaml
[root@k8s-master ~]# vi ssh-probe.yaml
apiVersion: v1
kind: Pod
metadata:
  name: ssh-probe

spec:
  containers:
  - name: ssh-server
    image: konan7979/ssh-probe:1.0
    ports:
    - containerPort: 22

    livenessProbe:
      tcpSocket:
        port: 22
      initialDelaySeconds: 10
      periodSeconds: 5
      failureThreshold: 3

: wq
```

```bash
[root@k8s-master tcp-probe-lab]# kubectl  apply  -f  ssh-probe.yaml  --dry-run=server
pod/ssh-probe created (server dry run)


[root@k8s-master tcp-probe-lab]# kubectl  apply  -f  ssh-probe.yaml
pod/ssh-probe created



[root@k8s-master tcp-probe-lab]# kubectl  get  pods
NAME     	  READY	  STATUS      	  RESTARTS    AGE
ssh-probe	  0/1     	  ContainerCreating	  0                 10s


[root@k8s-master tcp-probe-lab]# kubectl  get  pods
NAME     	  READY	  STATUS      RESTARTS    AGE
ssh-probe	  1/1     	  Running       0                  34s



[root@k8s-master tcp-probe-lab]# kubectl  describe  pods | grep Liveness
    Liveness:       tcp-socket :22 delay=10s timeout=1s period=5s #success=1 #failure=3


	# 컨테이너 접속
[root@k8s-master tcp-probe-lab]# kubectl  exec  ssh-probe -it  -- /bin/bash
[root@ssh-probe /]#


	# 현재 서비스중인 sshd의 port 번호 확인 (TCP 22)
[root@ssh-probe /]# ss  -lntp | grep sshd
LISTEN 0      128          0.0.0.0:22        0.0.0.0:*    users:(("sshd",pid=8,fd=7))
LISTEN 0      128              [::]:22            [::]:*    users:(("sshd",pid=8,fd=8))


-현재 sshd가 TCP 22 port로 서비스 하기때문에 liveness probe에의해 health-check되고 있다.

-이때 sshd의 서비스 port 번호를  TCP 22번이 아닌 다른 port 번호로 변경하게되면 health-check가 실패하고 컨테이너를 재시작한다.




[root@ssh-probe /]# vi  /etc/ssh/sshd_config		# sshd의 서비스 port번호를 변경하기위해서 sshd_config 파일 변경
      1 #       $OpenBSD: sshd_config,v 1.104 2021/07/02 05:11:21 dtucker Exp $
      ...
     21 Port 2222			# 주석 해제 후 port번호 변경

:wq


	# 두번째 터미널에서 확인
[root@k8s-master liveness-lab]# kubectl  get  pods  -o  wide  --watch

NAME        READY   STATUS    RESTARTS   AGE   IP            NODE          NOMINATED NODE   READINESS GATES
ssh-probe   1/1     Running   0          23m   10.244.1.17   k8s-worker1   <none>           <none>




[root@k8s-master tcp-probe-lab]# kubectl  exec  ssh-probe -it  -- /bin/bash
[root@ssh-probe /]#



[root@ssh-probe /]# pkill  sshd		# sshd 프로세스 중지

[root@ssh-probe /]# /usr/sbin/sshd		# sshd를 다시 실행하게되면 위에서 작성한 TCP 2222가 적용된다.



[root@ssh-probe /]#  ss  -lntp | grep sshd
LISTEN 0      128          0.0.0.0:2222      0.0.0.0:*    users:(("sshd",pid=70,fd=7))
LISTEN 0      128              [::]:2222          [::]:*    users:(("sshd",pid=70,fd=8))




	# 두번째 터미널에서 확인
[root@k8s-master liveness-lab]# kubectl  get  pods  -o  wide  --watch
NAME        READY   STATUS    RESTARTS   AGE   IP            NODE          NOMINATED NODE   READINESS GATES
ssh-probe   1/1     Running   0          23m   10.244.1.17   k8s-worker1   <none>           <none>
ssh-probe   1/1     Running   1 (0s ago)   28m   10.244.1.17   k8s-worker1   <none>           <none>



root@ssh-probe /]# command terminated with exit code 137		# 컨테이너가 다시 생성되기때문에 접속한 컨테이너에서 강제로 out된다.
[root@k8s-master tcp-probe-lab]#




	# 다시 컨테이너 접속
[root@k8s-master tcp-probe-lab]# kubectl  exec  ssh-probe -it  -- /bin/bash
[root@ssh-probe /]#


	# 현재 서비스중인 sshd의 port 번호 확인 (TCP 22)
[root@ssh-probe /]# ss  -lntp | grep sshd
LISTEN 0      128          0.0.0.0:22        0.0.0.0:*    users:(("sshd",pid=8,fd=7))
LISTEN 0      128              [::]:22            [::]:*    users:(("sshd",pid=8,fd=8))
```

**정리:** SSH 포트를 22에서 2222로 바꾸면 tcpSocket probe가 22번 포트 연결에 실패해 컨테이너가 재시작되고, 재시작된 컨테이너는 이미지의 기본 설정대로 다시 22번 포트로 뜨는 것을 확인했다. 다음은 **exec** 방식 probe 실습이다.

---

## 실습 3) exec probe - /healthy 파일 존재 여부 검사

**LivenessProbe**의 **exec** 방식을 이용하여 컨테이너 내부의 /usr/share/nginx/html/healthy 파일 존재 여부를 검사하고, 
파일이 없을 경우 Probe 실패로 컨테이너가 자동 재시작되는 것을 확인

- 이후 컨테이너 내부에 healthy 파일을 생성하여 Probe가 정상 상태로 복구되고 
 더 이상 컨테이너가 재시작되지 않는 것을 확인
- 해당 파일을 다시 삭제하여 livenessProbe 실패와 컨테이너 재시작이 다시 발생하는지 확인하시오

```bash
	# 실습 디렉터리 생성

[guest@k8s-master ~]# mkdir exec-liveness-lab

[guest@k8s-master ~]# cd exec-liveness-lab


[guest@k8s-master exec-liveness-lab]# pwd
/home/guest/exec-liveness-lab



[root@k8s-master exec-liveness-lab]# vi healthy
Liveness OK


[root@k8s-master exec-liveness-lab]# ls  -l
합계 4
-rw-r--r-- 1 root root 11  8월 14 16:58 healthy
```

```bash
	# Dockerfile 작성
[root@k8s-master exec-liveness-lab]# vi dockerfile
FROM nginx:1.31

COPY healthy /usr/share/nginx/html/healthy

:wq
```

```bash
	# 이미지 빌드 및 이미지 PUSH
[root@k8s-master exec-liveness-lab]# docker  build  -t  konan7979/nginx-exec-liveness:1.0  .


root@k8s-master exec-liveness-lab]# docker  push  konan7979/nginx-exec-liveness:1.0
The push refers to repository [docker.io/konan7979/nginx-exec-liveness]
3c55dc422a81: Mounted from konan7979/nginx-liveness
26c307b5e35a: Mounted from konan7979/nginx-liveness
51a7089ca9bb: Pushed
44136fa355b3: Mounted from konan7979/ssh-probe
d84ae7b21412: Mounted from konan7979/nginx-liveness
c0df8d325117: Mounted from konan7979/nginx-liveness
b8b80b9bc028: Mounted from konan7979/nginx-liveness
f5de6e85ac74: Mounted from konan7979/nginx-liveness
5a4222b844e8: Mounted from konan7979/nginx-liveness
0af710fe68e2: Pushed
1.0: digest: sha256:caeb48d1c8b7df9b94553b4f5197ac52af38dc0e3f0beac7c6a4b22167146166 size: 856
```

```yaml
	# exec 방식의 livenessProbe Pod 작성

[root@k8s-master exec-liveness-lab]# cd ..



[root@k8s-master ~]# vi exec-liveness.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-exec-liveness

spec:
  containers:
  - name: nginx
    image: konan7979/nginx-exec-liveness:1.0

    livenessProbe:
      exec:
        command:
        - /bin/sh
        - -c
        - 'test -f /usr/share/nginx/html/healthy && echo "$(date) : Liveness OK"  >>  /tmp/liveness.log'
      initialDelaySeconds: 5
      periodSeconds: 5
      failureThreshold: 3

:wq
```

```bash
[root@k8s-master ~]# kubectl  apply  -f  exec-liveness.yaml  --dry-run=server
pod/nginx-exec-liveness created (server dry run)



[root@k8s-master ~]# kubectl  get  pods  -n  soldesk
NAME               	  READY	  STATUS    RESTARTS   AGE
nginx-exec-liveness	  1/1    	  Running     0                 23s



[root@k8s-master ~]# kubectl describe pod nginx-exec-liveness | grep Liveness
    Liveness:       exec [/bin/sh -c test -f /usr/share/nginx/html/healthy && echo "Liveness OK"  >>  /tmp/liveness.log] delay=5s timeout=1s period=5s #success=1 #failure=3




	# 두번째 터미널에서 확인
[root@k8s-master ~]# kubectl  get  pods  -o  wide  -n soldesk  --watch
NAME               	  READY	  STATUS    RESTARTS   AGE      IP             NODE           NOMINATED NODE   READINESS GATES
nginx-exec-liveness	  1/1    	  Running     0                 2m24s   10.244.2.9   k8s-worker2   <none>                   <none>




[root@k8s-master ~]# kubectl  exec  nginx-exec-liveness   -it  --  /bin/bash
root@nginx-exec-liveness:/#



root@nginx-exec-liveness:/# ls -l /tmp
total 4
-rw-r--r-- 1 root root 43 Aug 14 08:25 liveness.log



root@nginx-exec-liveness:/# cat /tmp/liveness.log
Fri Aug 14 08:25:22 UTC 2026 : Liveness OK
Fri Aug 14 08:25:27 UTC 2026 : Liveness OK
Fri Aug 14 08:25:32 UTC 2026 : Liveness OK
Fri Aug 14 08:25:37 UTC 2026 : Liveness OK
Fri Aug 14 08:25:42 UTC 2026 : Liveness OK
Fri Aug 14 08:25:47 UTC 2026 : Liveness OK
Fri Aug 14 08:25:52 UTC 2026 : Liveness OK
Fri Aug 14 08:25:57 UTC 2026 : Liveness OK
Fri Aug 14 08:26:02 UTC 2026 : Liveness OK
Fri Aug 14 08:26:07 UTC 2026 : Liveness OK
Fri Aug 14 08:26:12 UTC 2026 : Liveness OK




	# 두번째 터미널에서 확인
[root@k8s-master ~]#  kubectl  get  pods  -o  wide  --watch
NAME                    READY   STATUS    RESTARTS   AGE     IP               NODE           NOMINATED NODE   READINESS GATES
nginx-exec-liveness   1/1        Running     0                4m12s   10.244.1.22   k8s-worker1   <none>                    <none>




	# exec Probe 실패 확인

[root@k8s-master ~]# kubectl  exec  nginx-exec-liveness   -it  --  /bin/bash
root@nginx-exec-liveness:/#
root@nginx-exec-liveness:/# rm  -rf  /usr/share/nginx/html/healthy
root@nginx-exec-liveness:/#
root@nginx-exec-liveness:/# command terminated with exit code 137		# exit code 137
[root@k8s-master ~]#




	# 두번째 터미널에서 확인
[root@k8s-master ~]#  kubectl  get  pods  -o  wide  --watch
NAME                    READY   STATUS    RESTARTS   AGE     IP               NODE           NOMINATED NODE   READINESS GATES
nginx-exec-liveness   1/1        Running     0                4m12s   10.244.1.22   k8s-worker1   <none>                    <none>
nginx-exec-liveness   1/1        Running     1 (0s ago)     5m21s   10.244.1.22   k8s-worker1   <none>                    <none>



	# 컨테이너 재생성 후 확인
[root@k8s-master ~]# kubectl  exec  nginx-exec-liveness   -it  --  /bin/bash
root@nginx-exec-liveness:/#
root@nginx-exec-liveness:/#


root@nginx-exec-liveness:/# ls  -l  /tmp/
total 4
-rw-r--r-- 1 root root 516 Aug 14 08:31 liveness.log



root@nginx-exec-liveness:/# ls  -l  /usr/share/nginx/html/
total 12
-rw-r--r-- 1 root root 497  Jul  15 16:03 50x.html
-rw-r--r-- 1 root root  12  Aug 14 08:01 healthy
-rw-r--r-- 1 root root 896  Jul  15 16:03 index.html
```

**정리:** exec probe는 컨테이너 내부에서 명령을 실행해 그 종료 코드로 정상 여부를 판단하며, 여기서는 /healthy 파일 존재 여부로 그 동작을 확인했다. httpGet, tcpSocket, exec 세 가지 livenessProbe 방식을 모두 실습했으니, 이제 Pod 내부의 또 다른 구성요소인 Init/Infra Container로 넘어간다.

---

## Pod - init container & infra container

Pod 안에는 실제로 일을 하는 컨테이너 말고도, 보이지 않지만 반드시 존재하거나, 먼저 실행되는 컨테이너들이 있다.

- **Init Container**	: 본 컨테이너가 실행되기 전에 준비 작업을 하는 컨테이너
- **Infra Container** 	: Pod 자체를 유지하기 위해 자동으로 생성되는 기반 컨테이너

### Init Container (초기화 컨테이너)

**Init Container**는 Pod가 시작될 때 메인 컨테이너보다 먼저 실행되는 컨테이너이다.

**특징**
- 메인 컨테이너보다 먼저 실행된다.
- 순차적으로 실행된다. (여러 개 가능)
- Init Container가 모두 정상 종료되어야만 메인Container가 실행된다.
- 한 번 실행되고 종료된다.
- Init Container가 실패하면 Pod 전체가 시작되지 않는다.

메인 컨테이너는 환경이 이미 준비되어 있다는 가정 하에 만들어진 경우가 많다.
그래서 다음과 같은 준비 작업을 미리 수행한다.
- 설정 파일 생성
- DB 연결 가능 여부 확인
- 공용 볼륨에 파일 생성
- 외부 서비스 준비 대기

---

## EX1) Init Container를 이용하여 메인 컨테이너의 실행을 지연시키시오.

- init-test-pod라는 Pod를 생성
- 이 Pod는 nginx 컨테이너가 바로 실행되지 않고, Init Container가 먼저 실행된 후 30초가 지나야 nginx 컨테이너가 실행

```
-Pod 이름: init-test-pod
 # Init Container 이미지: busybox
 # nit Container 실행 시 init container start 메시지를 출력
 # 30초 동안 대기
 # 대기 후 init container end 메시지를 출력하고 종료
 # 메인 컨테이너  이미지: nginx
 # nginx 컨테이너의 포트는 80번으로 설정
 # Init Container가 실행되는 동안 nginx가 실행되지 않는 것을 확인
 # Init Container가 종료된 후 nginx가 실행되는 것을 확인한다.
```

```yaml
[root@k8s-master ~]# vi init-test-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: init-test-pod

spec:
  initContainers:
  - name: init-sleep
    image: busybox
    command:
    - sh
    - -c
    - |
      echo "init container start"
      sleep 30
      echo "init container end"

  containers:
  - name: nginx
    image: nginx
    ports:
    - containerPort: 80


-busybox는 리눅스에서 자주 사용하는 기본 명령어들을 하나의 아주 작은 실행 파일에 모아둔 경량 도구
-쿠버네티스 실습에서는 주로 간단한 테스트용 컨테이너로 많이 사용
```

```bash
	# 1번 터미널
[root@k8s-master ~]# kubectl  get  pods  -o  wide  --watch



	# 2번 터미널
[root@k8s-master ~]# kubectl  create  -f  init-test-pod.yaml
pod/init-test-pod created


	# 1번 터미널
[root@k8s-master ~]# kubectl  get  pods  -o  wide  --watch
NAME            READY   STATUS    	RESTARTS   AGE   IP       	  NODE     	NOMINATED NODE   READINESS GATES
init-test-pod    0/1        Pending	0                 0s     <none> 	  <none>   	<none>             	  <none>
init-test-pod    0/1        Pending   	0                 0s     <none>  	  k8s-worker1   	<none>            	  <none>
init-test-pod    0/1        Init:0/1   	0                 0s     <none>  	  k8s-worker1   	<none>           	  <none>
init-test-pod    0/1        Init:0/1   	0                 6s     10.244.1.2	  k8s-worker1   	<none>           	  <none>
init-test-pod    0/1        PodInitializing	0                 36s   10.244.1.2   k8s-worker1   	<none>           	  <none>
init-test-pod    1/1        Running       	0                 39s   10.244.1.2   k8s-worker1   	<none>           	  <none>


**PodInitializing** : init 컨테이너가 거의 끝나거나 init 컨테이너 종료 이후, 
  메인 컨테이너(nginx)를 올리는 과정(이미지 풀, 네트워크, 볼륨 마운트, 컨테이너 생성 등)에 들어간 상태
```

**정리:** Init Container가 실행되는 30초 동안 Pod 상태가 Init:0/1로 머물다가, Init Container 종료 후 PodInitializing을 거쳐 nginx가 Running으로 올라오는 순서를 확인했다.

---

## EX2) init이 실패하면 main 컨테이너는 실행되지 않는다.

```yaml
[root@k8s-master ~]# vi  init-step2-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: init-test-pod

spec:
  initContainers:
  - name: init-sleep
    image: busybox
    command: ["sh", "-c", "echo init fail now;  exit 1"]

  containers:
  - name: nginx-init-pod
    image: nginx:latest
    ports:
    - containerPort: 80
```

```bash
[root@k8s-master ~]# kubectl  apply -f  init-step2-pod.yaml  --dry-run=server
pod/init-test-pod created (server dry run)


	# 1번 터미널
[root@k8s-master ~]# kubectl  get  pods  -o  wide  --watch



	# 2번 터미널
[root@k8s-master ~]# kubectl  apply -f  init-step2-pod.yaml
pod/init-test-pod created



	# 2번 터미널
[root@k8s-master ~]# kubectl logs  init-test-pod init-sleep
init fail now
```

**정리:** Init Container가 exit 1로 실패하면 메인 컨테이너(nginx)는 아예 시작되지 않으며, Pod는 Init 단계에서 멈춘 채 로그로만 실패 원인을 확인할 수 있다.

---

## EX3) Init Container와 emptyDir 볼륨을 이용하여 nginx 웹 페이지를 생성

```
- Pod 이름: init-test-pod
- Init Container에서 /work/index.html 생성
- emptyDir 볼륨을 Init Container와 nginx가 공유
- Init Container는 파일 생성 후 30초 대기
- nginx는 공유된 index.html을 웹 페이지로 사용
- nginx 컨테이너 포트는 80번으로 설정
```

```yaml
[root@k8s-master ~]# vi init-step1-volume.yaml
apiVersion: v1
kind: Pod
metadata:
  name: init-test-pod

spec:

  volumes:
  - name: shared-data
    emptyDir: {}

  initContainers:
  - name: init-create-index
    image: busybox:1.36
    command:
    - /bin/sh
    - -c
    - |
      echo "Init Container가 만든 페이지입니다" > /work/index.html
      sleep 30
    volumeMounts:
    - name: shared-data
      mountPath: /work

  containers:
  - name: nginx
    image: nginx:1.29.1
    ports:
    - containerPort: 80
    volumeMounts:
    - name: shared-data
      mountPath: /usr/share/nginx/html
```

```
volumes:
- name: shared-data
  emptyDir: {}

-Pod에서 사용할 볼륨을 생성
 # 볼륨 이름은 shared-data
 # emptyDir는 Pod가 생성될 때 만들어지는 임시 공유 공간
 # 같은 Pod 안의 여러 컨테이너가 이 공간을 공유할 수 있다.
 # Pod가 삭제되면 emptyDir의 데이터도 같이 삭제된다.
```

```bash
	# 1번 터미널
[root@k8s-master ~]# kubectl  get  pods  -o  wide  --watch


	# 2번 터미널
[root@k8s-master ~]# kubectl  apply  -f init-step1-volume.yaml
pod/init-test-pod created



	# 1번 터미널
[root@k8s-master ~]# kubectl  get  pods  -o  wide  --watch
NAME            READY   STATUS    	RESTARTS   AGE   IP       	   NODE     	NOMINATED NODE   READINESS GATES
init-test-pod    0/1        Pending	0                 0s    	<none>   	   <none>   	<none>           	  <none>
init-test-pod    0/1        Pending   	0                 0s    	<none>   	   k8s-worker1   	<none>           	  <none>
init-test-pod    0/1        Init:0/1   	0                 0s   	<none>   	   k8s-worker1   	<none>           	  <none>
init-test-pod    0/1        Init:0/1   	0                 8s  	10.244.1.3	   k8s-worker1   	<none>           	  <none>
init-test-pod    0/1        PodInitializing	0                 37s	10.244.1.3   k8s-worker1   	<none>           	  <none>
init-test-pod    1/1        Running      	0                 38s    10.244.1.3   k8s-worker1   	<none>           	  <none>



-파드 안에서 Init Container가 만든 파일을 nginx 컨테이너가 이어받기 위한 공용 공간

1) Init Container	: index.html 파일을 만든다.
2) nginx 컨테이너	: 그 파일을 웹 페이지로 보여준다.




[root@k8s-master ~]# curl  http://10.244.1.3
Init Container가 만든 페이지입니다
```

**정리:** Init Container와 nginx 컨테이너가 **emptyDir** 볼륨(shared-data)을 공유해, Init Container가 만든 index.html을 nginx가 그대로 서비스하는 것을 확인했다. 이제 Pod를 눈에 보이지 않게 떠받치는 Infra Container를 살펴본다.

---

## Infra Container (pause)

**Infra Container**는 Pod를 Pod답게 유지하기 위해 쿠버네티스가 내부적으로 자동 생성하는 컨테이너다.
사용자가 YAML로 만들지 않는다.

**왜 필요한가**
- Pod의 핵심 개념은 여러 컨테이너가 네트워크와 볼륨을 공유하는 하나의 묶음 이다.
- 이 공유 공간을 유지할 주체가 필요한데 그 역할을 하는 것이 Infra Container다.

**Infra Container가 담당하는 것**
- Pod의 IP 주소 유지
- 네트워크 네임스페이스 유지
- 볼륨 마운트 유지
- 컨테이너 간 통신 공간 유지

메인 컨테이너가 재시작되어도 Pod IP가 바뀌지 않는 이유가 바로 이것 때문이다.

**동작 흐름**
- Pod 생성 요청 --> Infra Container 생성 --> 네트워크/IP/볼륨 고정 --> Init Container 실행 --> 메인 컨테이너 실행

**특징**
- 항상 실행 중이다.
- 사용자가 직접 접근하지 않는다.
- 리소스 사용량이 매우 적다.
- Pod가 살아 있는 한 계속 유지된다.

```bash
[root@k8s-master ~]# ssh  guest@k8s-worker1
The authenticity of host 'k8s-worker1 (192.168.10.101)' can't be established.
ED25519 key fingerprint is SHA256:wa8EL0U/8UruYh4dmxNhUFpfE4jF7IKNHyCAtiV8JoE.
This host key is known by the following other names/addresses:
    ~/.ssh/known_hosts:4: 192.168.10.101
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'k8s-worker1' (ED25519) to the list of known hosts.
guest@k8s-worker1's password:
Activate the web console with: systemctl enable --now cockpit.socket

Last login: Fri Aug 14 10:15:51 2026 from 192.168.10.100
[guest@k8s-worker1 ~]$


[guest@k8s-worker1 ~]$ su -
암호:
[root@k8s-worker1 ~]#


[root@k8s-worker1 ~]# dnf  install  -y  cri-tools


[root@k8s-worker1 ~]# crictl pods
WARN[0000] Config "/etc/crictl.yaml" does not exist, trying next: "/usr/bin/crictl.yaml"
WARN[0000] runtime connect using default endpoints: [unix:///run/containerd/containerd.sock unix:///run/crio/crio.sock unix:///var/run/cri-dockerd.sock]. As the default settings are now deprecated, you should set the endpoint instead.
POD ID              	CREATED        	STATE               	NAME                    	NAMESPACE     	ATTEMPT             RUNTIME
29f56d92132aa   	9 hours ago         	Ready               	kube-flannel-ds-f9wcz	kube-flannel	3                          (default)
57ad58a49fbd8       	9 hours ago         	Ready               	kube-proxy-gbr4f        	kube-system         	3                          (default)
f5a1f0ddbc5fb       	32 hours ago        	NotReady            	kube-flannel-ds-f9wcz   	kube-flannel        	2                          (default)
6cc3fe1a5fa93      	 32 hours ago	NotReady    	kube-proxy-gbr4f        	kube-system         	2                          (default)
```

**정리:** Infra Container(pause)는 Pod의 네트워크/IP/볼륨을 계속 유지해주는 보이지 않는 기반 컨테이너로, `crictl pods` 결과에서 실제 kube-flannel/kube-proxy Pod들도 이 구조 위에서 동작함을 확인할 수 있다. 마지막으로 API Server 없이도 동작하는 **Static Pod**를 다룬다.

---

## Static Pod란 무엇인가

**Static Pod**는 kube-api server(쿠버네티스 API 서버)를 거치지 않고, 각 노드의 kubelet이 직접 관리하는 Pod이다.

일반적인 Pod 생성 흐름
- kubectl --> API Server --> Scheduler --> kubelet --> pod 생성

**Static Pod**
- kubelet이 로컬 파일(YAML)을 직접 읽어서 Pod를 생성한다.
- 즉, 클러스터 전체가 아니라, 특정 노드에 고정되어 실행되는 Pod

### 왜 Static Pod가 필요한가

쿠버네티스 클러스터가 처음 올라올 때를 생각해보자.
API Server, Scheduler, Controller Manager 같은 핵심 컴포넌트는 
쿠버네티스가 완전히 동작하기 전에도 먼저 실행되어야 한다.
하지만 이 시점에는 API Server가 아직 실행되지 않았기 때문에, 일반적인 방식으로 Pod 생성을 요청할 수 없다.

즉 API Server가 아직 없는데 API Server에게 kube-apiserver Pod를 생성해달라고 요청할 수는 없다.

그래서 Control Plane의 핵심 컴포넌트들은 API Server를 거치지 않고, 각 노드의 kubelet이 Static Pod 형태로 직접 실행하고 관리한다.

```bash
[root@k8s-master ~]# kubectl get pods -n kube-system
NAME                               	READY   STATUS	RESTARTS     AGE
coredns-55cb58b774-h2gc6           	1/1       Running   	7 (169m ago)   15d
coredns-55cb58b774-pgsfq           	1/1       Running   	7 (169m ago)   15d
etcd-k8s-master                     	1/1       Running   	7 (169m ago)   15d
kube-apiserver-k8s-master           	1/1       Running   	7 (169m ago)   15d
kube-controller-manager-k8s-master 	1/1       Running   	7 (169m ago)   15d
kube-proxy-7ghsg                     	1/1       Running   	7 (169m ago)   15d
kube-proxy-dzxb5                     	1/1       Running   	4 (169m ago)   15d
kube-proxy-nsh2n                     	1/1       Running   	4 (169m ago)   15d
kube-scheduler-k8s-master            	1/1       Running   	7 (169m ago)   15d
```

그래서 등장한 방식이 Static Pod이다.

Static Pod는 쿠버네티스 없이도 kubelet 혼자서 실행 가능한 Pod 이기 때문에
클러스터의 시작을 책임지는 핵심 컴포넌트 실행에 사용된다.

### Static Pod의 동작 구조

kubelet은 특정 디렉터리를 감시한다.
- 예: /etc/kubernetes/manifests

- 그 디렉터리 안에 YAML 파일이 있으면 kubelet이 자동으로 Pod를 생성한다.
- YAML 파일이 수정되면 Pod도 자동으로 재생성된다.
- YAML 파일을 삭제하면 Pod도 자동으로 삭제된다.

**Static Pod**
- API Server에 요청하지 않는다.
- kubelet이 혼자 처리한다.

### Static Pod와 일반 Pod의 차이

**일반 Pod**
- API Server를 통해 생성
- etcd에 정보 저장
- Scheduler가 노드 배치 결정
- kubectl로 관리

**Static Pod**
- kubelet이 직접 생성
- etcd에 직접 저장되지 않음
- 특정 노드에 고정
- kubectl로 생성/삭제 불가

### Static Pod의 위치와 설정

kubelet 설정 파일에 Static Pod 경로가 지정되어 있다.
kubelet이 /etc/kubernetes/manifests 디렉터리를 계속 지켜본다.

그 디렉터리에 다음과 같은 파일이 있으면
kube-apiserver.yaml
kube-scheduler.yaml
kube-controller-manager.yaml 가 자동으로 실행된다.

이 파일들은 kubectl apply로 만든 것이 아니라 노드에 직접 파일로 존재한다.
kubelet의 config.yaml은 control-plane과 모든 worker node가 갖고있는 파일이다.

```bash
[root@k8s-master ~]# kubectl get pods -o  wide -n kube-system
NAME                                 	READY   STATUS    RESTARTS      AGE     IP                   NODE          NOMINATED NODE   READINESS GATES
coredns-7d764666f9-2dwdd     	1/1         Running     5 (81m ago)     6d19h    10.244.0.12       k8s-master    <none>           <none>
coredns-7d764666f9-klkhq             	1/1         Running     5 (81m ago)     6d19h    10.244.0.13       k8s-master    <none>           <none>
etcd-k8s-master                      	1/1         Running     6 (81m ago)     6d19h    192.168.10.100   k8s-master    <none>           <none>
kube-apiserver-k8s-master            	1/1         Running     6 (81m ago)     6d19h    192.168.10.100   k8s-master    <none>           <none>
kube-controller-manager-k8s-master	1/1         Running     6 (81m ago)     6d19h    192.168.10.100   k8s-master    <none>           <none>
kube-proxy-bxjvw                     	1/1         Running     4 (81m ago)     6d19h    192.168.10.102   k8s-worker2   <none>           <none>
kube-proxy-gbr4f                     	1/1         Running     4 (81m ago)     6d19h    192.168.10.101   k8s-worker1   <none>           <none>
kube-proxy-rwhkc                     	1/1         Running     5 (81m ago)     6d19h    192.168.10.100   k8s-master    <none>           <none>
kube-scheduler-k8s-master            	1/1         Running     6 (81m ago)     6d19h    192.168.10.100   k8s-master    <none>           <none>



[root@k8s-master ~]# ls  -l  /etc/kubernetes/manifests
합계 16
-rw------- 1 root root 2612  8월 11 15:20 etcd.yaml
-rw------- 1 root root 3674  8월 11 15:20 kube-apiserver.yaml
-rw------- 1 root root 3168  8월 11 15:20 kube-controller-manager.yaml
-rw------- 1 root root 1726  8월 11 15:20 kube-scheduler.yaml




[root@k8s-master ~]# cat /var/lib/kubelet/config.yaml
apiVersion: kubelet.config.k8s.io/v1beta1
authentication:
  anonymous:
    enabled: false
  webhook:
    cacheTTL: 0s
    enabled: true
  x509:
    clientCAFile: /etc/kubernetes/pki/ca.crt
authorization:
  mode: Webhook
  webhook:
    cacheAuthorizedTTL: 0s
    cacheUnauthorizedTTL: 0s
cgroupDriver: systemd
clusterDNS:
- 10.96.0.10
clusterDomain: cluster.local
containerRuntimeEndpoint: unix:///var/run/containerd/containerd.sock
cpuManagerReconcilePeriod: 0s
crashLoopBackOff: {}
evictionPressureTransitionPeriod: 0s
fileCheckFrequency: 0s
healthzBindAddress: 127.0.0.1
healthzPort: 10248
httpCheckFrequency: 0s
imageMaximumGCAge: 0s
imageMinimumGCAge: 0s
kind: KubeletConfiguration
logging:
  flushFrequency: 0
  options:
    json:
      infoBufferSize: "0"
    text:
      infoBufferSize: "0"
  verbosity: 0
memorySwap: {}
nodeStatusReportFrequency: 0s
nodeStatusUpdateFrequency: 0s
rotateCertificates: true
runtimeRequestTimeout: 0s
shutdownGracePeriod: 0s
shutdownGracePeriodCriticalPods: 0s
staticPodPath: /etc/kubernetes/manifests	# static pod의 경로
streamingConnectionIdleTimeout: 0s
syncFrequency: 0s
volumeStatsAggPeriod: 0s
```

### Static Pod 실습 - k8s-worker1에서 생성

k8s-worker1 노드에 직접 접속해 `/etc/kubernetes/manifests`에 YAML을 복사/삭제하며 **Static Pod**가 자동으로 생성·삭제되는 것을 확인하는 실습이다.

```bash
	# k8s-worker1 노드 접속
[root@k8s-master ~]# ssh  guest@k8s-worker1
guest@k8s-worker1's password:
Activate the web console with: systemctl enable --now cockpit.socket

Last login: Fri Aug 14 17:56:15 2026 from 192.168.10.100
[guest@k8s-worker1 ~]$



[guest@k8s-worker1 ~]$ pwd
/home/guest


	# static pod로 생성할 yaml 파일 생성
[root@k8s-worker1 ~]# vi nginx-static-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-static-pod
spec:
  containers:
  - name: nginx-container
    image: nginx:1.31
    ports:
    - containerPort: 80



[guest@k8s-worker1 ~]$ ls -l
합계 4
-rw-r--r--  1 guest guest 163  8월 18 11:08 nginx-static-pod.yaml



[guest@k8s-worker1 ~]$ ls  -l  /etc/kubernetes/manifests/
합계 0



[guest@k8s-worker1 ~]$ sudo cp  nginx-static-pod.yaml  /etc/kubernetes/manifests/



[root@k8s-master ~]# kubectl get pods
NAME                           	  READY   STATUS    RESTARTS   AGE
nginx-static-pod-k8s-worker1	  1/1         Running     0                 51s




[root@k8s-master ~]# kubectl get pods -o wide
NAME                           	  READY   STATUS    RESTARTS   AGE     IP              NODE           NOMINATED NODE   READINESS GATES
nginx-static-pod-k8s-worker1	  1/1         Running     0                2m50s   10.244.1.4   k8s-worker1   <none>                    <none>




	# yaml 파일을 삭제하게되면 pod도 삭제된다.
[guest@k8s-worker1 ~]$ sudo rm -rf  /etc/kubernetes/manifests/nginx-static-pod.yaml




[root@k8s-master ~]# kubectl get pods
No resources found in default namespace.
```

**정리:** `/etc/kubernetes/manifests`에 YAML을 넣으면 kubelet이 즉시 Pod를 생성하고, 파일을 지우면 즉시 삭제되는 것을 확인했다. 이는 API Server 없이도 Control Plane 핵심 컴포넌트가 부팅될 수 있게 하는 Static Pod의 핵심 동작 원리다.

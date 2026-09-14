# 쿠버네티스 Label

## Label이란?

**Label**은 쿠버네티스의 오브젝트(Pod, Service, Deployment 등)를 분류하고 식별하기 위한 가장 기본적인 메타데이터 개념이며, Service나 Deployment가 관리할 Pod를 찾거나 nodeSelector로 Pod를 특정 Node에 배치할 때 이 Label을 기준으로 삼는다.

- **Label**은 쿠버네티스의 오브젝트(Pod, Service, Deployment 등)에 키(Key)와 값(Value) 형태의 정보를 붙이는 기능이다.
- Label 형식 : Key = Value

```yaml
metadata:
  labels:
    app: web
    env: prod
```

- `app` = Label의 Key
- `web` = Label의 Value
- `env` = Label의 Key
- `prod` = Label의 Value

### Label을 사용하는 이유

쿠버네티스에서는 수많은 Pod가 생성될 수 있기 때문에 Pod를 분류하고 선택하기 위해 Label을 사용한다.

예를 들어 Pod가 3개 있다고 하자.

- Pod1 : app=web
- Pod2 : app=web
- Pod3 : app=db

이때 `kubectl get pod -l app=web` 을 사용하면 Pod1, Pod2만 조회할 수 있다.  
즉, **Label**은 오브젝트를 그룹화하고 원하는 대상을 선택하기 위한 식별 정보이다.

> **정리** : Label은 Key=Value 형태로 오브젝트에 붙이는 분류 정보이며, 이를 통해 원하는 오브젝트만 선택적으로 조회할 수 있다.

---

## Label의 기본 구조

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: web-pod
  labels:
    app: web
    env: prod
    tier: frontend
```

- **Label**은 여러 개를 지정할 수 있다.
  - `app  = web`
  - `env  = prod`
  - `tier = frontend`
- 각 Label은 서로 독립적이다.

---

## Label과 Name의 차이

**Label**과 **Name**은 둘 다 metadata 아래에 위치하지만 용도가 다르다.

### Name

오브젝트 자체의 이름이다.

```yaml
metadata:
  name: web-pod
```

하나의 특정 Pod를 식별한다.

### Label

오브젝트의 분류 정보다.

```yaml
metadata:
  labels:
    app: web
```

여러 Pod에 같은 Label을 지정할 수 있다.

- web-pod-1 : app=web
- web-pod-2 : app=web
- web-pod-3 : app=web

| 구분   | 설명                     |
|--------|--------------------------|
| Name   | 개별 오브젝트의 이름     |
| Label  | 오브젝트를 분류하기 위한 정보 |

> **정리** : Name은 오브젝트 하나를 유일하게 식별하고, Label은 여러 오브젝트를 그룹으로 묶어 분류하는 역할을 한다.

---

## Label 확인하기

전체 Pod와 **Label**을 함께 확인하려면:

```bash
[root@k8s-master ~]# kubectl get pod --show-labels
```

```
NAME   	READY     STATUS    LABELS
web-pod	1/1          Running     app=web,env=prod
db-pod  	1/1          Running     app=db,env=prod
```

---

## 특정 Label을 가진 오브젝트 조회

특정 **Label**을 가진 오브젝트만 조회하려면 `-l` 옵션을 사용한다.

```bash
[root@k8s-master ~]# kubectl get pod -l app=web
```

또는

```bash
[root@k8s-master ~]# kubectl get pod --selector app=web
```

- `-l` = `--selector`
- `kubectl get pod -l app=web` 은 app=web이라는 Label을 가진 Pod만 조회하라는 의미다.

> **정리** : `-l`(`--selector`) 옵션으로 특정 Label을 가진 오브젝트만 골라 조회할 수 있다.

---

## 여러 Label을 이용한 조회

여러 개의 Label 조건을 동시에 지정하여 조회할 수도 있다.

- Pod  : app=web, env=prod
- Pod2 : app=web, env=dev
- Pod3 : app=db,  env=prod

다음 명령어를 실행하면:

```bash
[root@k8s-master ~]# kubectl get pod -l app=web,env=prod
```

두 조건을 모두 만족하는 Pod를 찾는다.

```
결과 = Pod1
```

즉, `app=web AND env=prod` 라고 생각하면 된다.

> **정리** : 쉼표(`,`)로 여러 Label 조건을 나열하면 AND 조건으로 동작하여 모든 조건을 만족하는 오브젝트만 선택된다.

---

## Label 추가, 변경, 삭제

`kubectl label` 명령어를 사용하면 오브젝트의 Label을 추가, 변경, 삭제할 수 있다.

```bash
# app이 없을 때  -->  추가
[root@k8s-master ~]# kubectl label pod web-pod app=web

# app=web이 있을 때  -->  에러
[root@k8s-master ~]# kubectl label pod web-pod app=api

# app=web이 있을 때  -->  변경
[root@k8s-master ~]# kubectl label pod web-pod app=api  --overwrite

# app=web이 있을 때 stage는 없으므로 새로운 Label 추가
[root@k8s-master ~]# kubectl label pod web-pod stage=dev

# Label 삭제 (Label의 Key 뒤에 -를 붙인다)
[root@k8s-master ~]# kubectl label pod web-pod app-
```

> **정리** : Label 추가는 그냥 `key=value`, 변경은 `--overwrite` 옵션, 삭제는 `key-` 형태로 명령어를 실행한다.

---

## Label Selector

**Label Selector**는 Label을 기준으로 특정 오브젝트를 선택하는 기능이다.

```bash
[root@k8s-master ~]# kubectl get pod -l app=web
```

여기서 `app=web` 이 **Label Selector**다.  
쿠버네티스에서 Label Selector는 매우 중요하다.  
특히 Service와 Deployment/ReplicaSet에서 Pod를 선택할 때 핵심적으로 사용된다.

> **정리** : Label Selector는 Label을 조건으로 오브젝트를 골라내는 기능이며, Service와 Deployment 같은 상위 오브젝트가 자신의 관리 대상 Pod를 찾는 핵심 수단이다.

---

## Service에서 Label 사용

```yaml
# Pod
metadata:
  labels:
    app: web

# Service
spec:
  selector:
    app: web
```

Service의 **selector**와 Pod의 label이 일치하면 Service가 해당 Pod를 대상으로 선택한다.

---

## Deployment에서 Label 사용

Deployment에서 흔히 다음과 같이 사용한다.

- **matchLabels**는 Deployment/ReplicaSet이 관리할 Pod를 지정하는 데 사용되는 조건이다.

```yaml
spec:
  selector:
    matchLabels:
      app: web

  template:
    metadata:
      labels:
        app: web
```

Deployment/ReplicaSet은 이 Label을 기준으로 자신이 관리할 Pod를 구분한다.

> **정리** : Service는 `selector`, Deployment는 `matchLabels`로 각각 Pod의 Label을 기준 삼아 대상 Pod를 연결하거나 관리한다.

---

## Label과 Annotation의 차이

둘 다 metadata 아래에 작성하지만 목적이 다르다.

### Label

검색, 분류, 선택에 사용

```yaml
metadata:
  labels:
    app: web
```

### Annotation

**Annotation**은 추가적인 메타데이터를 저장하기 위한 용도이다.  
Annotation은 일반적으로 Selector의 기준으로 사용하지 않는다.

```bash
[root@k8s-master ~]# kubectl get pod -l app=web
```

```yaml
metadata:
  annotations:
    description: "web server"
```

| 구분       | 목적                  |
|------------|-----------------------|
| Label      | 분류 / 검색 / 선택    |
| Annotation | 부가 정보 저장        |

> **정리** : Label은 오브젝트를 검색·분류·선택하는 데 쓰이고, Annotation은 Selector의 기준이 되지 않는 부가 정보를 담는 용도로 사용한다.

---

## 실습: 10개의 Pod 생성

### EX2. 다음 조건에 맞는 label-practice.yaml 파일을 작성

총 10개의 Pod를 생성한다.

| Pod   | app | env  | tier     |
|-------|-----|------|----------|
| web-1 | web | prod | frontend |
| web-2 | web | prod | frontend |
| web-3 | web | dev  | frontend |
| web-4 | web | dev  | frontend |
| api-1 | api | prod | backend  |
| api-2 | api | prod | backend  |
| api-3 | api | dev  | backend  |
| db-1  | db  | prod | database |
| db-2  | db  | dev  | database |
| db-3  | db  | prod | database |

```bash
[root@k8s-master ~]# vi label-test-pod.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: web-1
  labels:
    app: web
    env: prod
    tier: frontend
spec:
  containers:
    - name: nginx
      image: nginx:latest

---
apiVersion: v1
kind: Pod
metadata:
  name: web-2
  labels:
    app: web
    env: prod
    tier: frontend
spec:
  containers:
    - name: nginx
      image: nginx:latest

---
apiVersion: v1
kind: Pod
metadata:
  name: web-3
  labels:
    app: web
    env: dev
    tier: frontend
spec:
  containers:
    - name: nginx
      image: nginx:latest

---
apiVersion: v1
kind: Pod
metadata:
  name: web-4
  labels:
    app: web
    env: dev
    tier: frontend
spec:
  containers:
    - name: nginx
      image: nginx:latest

---
apiVersion: v1
kind: Pod
metadata:
  name: api-1
  labels:
    app: api
    env: prod
    tier: backend
spec:
  containers:
    - name: nginx
      image: nginx:latest

---
apiVersion: v1
kind: Pod
metadata:
  name: api-2
  labels:
    app: api
    env: prod
    tier: backend
spec:
  containers:
    - name: nginx
      image: nginx:latest

---
apiVersion: v1
kind: Pod
metadata:
  name: api-3
  labels:
    app: api
    env: dev
    tier: backend
spec:
  containers:
    - name: nginx
      image: nginx:latest

---
apiVersion: v1
kind: Pod
metadata:
  name: db-1
  labels:
    app: db
    env: prod
    tier: database
spec:
  containers:
    - name: nginx
      image: nginx:latest

---
apiVersion: v1
kind: Pod
metadata:
  name: db-2
  labels:
    app: db
    env: dev
    tier: database
spec:
  containers:
    - name: nginx
      image: nginx:latest

---
apiVersion: v1
kind: Pod
metadata:
  name: db-3
  labels:
    app: db
    env: prod
    tier: database
spec:
  containers:
    - name: nginx
      image: nginx:latest
```

```bash
[root@k8s-master ~]# kubectl  apply  -f  label-test-pod.yaml  --dry-run=client
pod/web-1 created (dry run)
pod/web-2 created (dry run)
pod/web-3 created (dry run)
pod/web-4 created (dry run)
pod/api-1 created (dry run)
pod/api-2 created (dry run)
pod/api-3 created (dry run)
pod/db-1 created (dry run)
pod/db-2 created (dry run)
pod/db-3 created (dry run)
```

```bash
[root@k8s-master ~]# kubectl  apply  -f  label-test-pod.yaml
pod/web-1 created
pod/web-2 created
pod/web-3 created
pod/web-4 created
pod/api-1 created
pod/api-2 created
pod/api-3 created
pod/db-1 created
pod/db-2 created
pod/db-3 created
```

```bash
[root@k8s-master ~]# kubectl  get  pods
NAME    	READY   STATUS    RESTARTS   AGE
api-1   	1/1         Running     0                28s
api-2   	1/1         Running     0                28s
api-3   	1/1         Running     0                28s
db-1    	1/1         Running     0                28s
db-2    	1/1         Running     0                28s
db-3    	1/1         Running     0                28s
web-1   	1/1         Running     0                28s
web-2   	1/1         Running     0                28s
web-3   	1/1         Running     0                28s
web-4   	1/1         Running     0                28s
```

> **정리** : 앞서 정의한 Label 조합대로 10개의 Pod를 생성했다. 이후 실습에서는 이 Pod들을 대상으로 Label 조회, 변경, 삭제를 연습한다.

---

## Label 확인 실습

### EX1) 생성된 10개의 Pod와 각각의 Label을 확인

```bash
[root@k8s-master ~]# kubectl get pod --show-labels
```

```
NAME    READY   STATUS    RESTARTS   AGE    LABELS
api-1   1/1     Running   0          4m3s   app=api,env=prod,tier=backend
api-2   1/1     Running   0          4m3s   app=api,env=prod,tier=backend
api-3   1/1     Running   0          4m3s   app=api,env=dev,tier=backend
db-1    1/1     Running   0          4m3s   app=db,env=prod,tier=database
db-2    1/1     Running   0          4m3s   app=db,env=dev,tier=database
db-3    1/1     Running   0          4m3s   app=db,env=prod,tier=database
web-1   1/1     Running   0          4m3s   app=web,env=prod,tier=frontend
web-2   1/1     Running   0          4m3s   app=web,env=prod,tier=frontend
web-3   1/1     Running   0          4m3s   app=web,env=dev,tier=frontend
web-4   1/1     Running   0          4m3s   app=web,env=dev,tier=frontend
```

### EX2) app=web Label을 가진 Pod만 조회

```bash
[root@k8s-master ~]# kubectl  get  pods  -l app=web
NAME    READY   STATUS    RESTARTS   AGE
web-1    1/1         Running     0                5m12s
web-2    1/1         Running     0                5m12s
web-3    1/1         Running     0                5m12s
web-4    1/1         Running     0                5m12s
```

### EX3) app=api Label을 가진 Pod만 조회

```bash
[root@k8s-master ~]# kubectl  get  pods  -l app=api
NAME    READY   STATUS    RESTARTS   AGE
api-1     1/1         Running     0                 6m5s
api-2     1/1         Running     0                 6m5s
api-3     1/1         Running     0                 6m5s
```

> **정리** : `--show-labels`로 전체 Label을 확인하고, `-l`로 단일 조건 Label을 가진 Pod만 골라 조회했다.

---

## AND 조건 실습

### EX4) app=web이면서 env=prod인 Pod만 조회

```bash
[root@k8s-master ~]# kubectl  get  pods  -l app=web,env=prod
NAME    READY   STATUS    RESTARTS   AGE
web-1    1/1        Running     0                 7m29s
web-2    1/1        Running     0                 7m29s
```

### EX5) app=web이면서 env=dev인 Pod만 조회

```bash
[root@k8s-master ~]# kubectl get pod -l app=web,env=dev
NAME    READY   STATUS    RESTARTS   AGE
web-3   1/1     Running   0          10m
web-4   1/1     Running   0          10m
```

### EX6) app=api이면서 env=prod인 Pod만 조회

```bash
[root@k8s-master ~]# kubectl get pod -l app=api,env=prod
NAME    READY   STATUS    RESTARTS   AGE
api-1     1/1         Running     0                11m
api-2     1/1         Running     0                11m
```

> **정리** : 쉼표로 여러 Label 조건을 연결하면 AND 조건으로 동작하여 모든 조건을 만족하는 Pod만 조회된다.

---

## 번외 OR 검색

`in (값1,값2)` 형식을 사용하면 여러 값 중 하나라도 일치하는 OR 조건으로 조회할 수 있다.

### EX7) env=prod 또는 env=dev인 Pod를 조회

```bash
[root@k8s-master ~]# kubectl  get  pods  -l 'env in (prod,dev)'
NAME	READY   STATUS    RESTARTS   AGE
api-1     	1/1     Running   0          14m
api-2     	1/1     Running   0          14m
api-3   	1/1     Running   0          14m
db-1    	1/1     Running   0          14m
db-2    	1/1     Running   0          14m
db-3    	1/1     Running   0          14m
web-1   	1/1     Running   0          14m
web-2   	1/1     Running   0          14m
web-3   	1/1     Running   0          14m
web-4   	1/1     Running   0          14m
```

### EX8) app=web 또는 app=api인 Pod를 조회하시오.

```bash
[root@k8s-master ~]# kubectl  get  pods  -l 'app in (web,api)'
NAME	READY   STATUS    RESTARTS   AGE
api-1 	1/1     Running   0          15m
api-2 	1/1     Running   0          15m
api-3  	1/1     Running   0          15m
web-1	1/1     Running   0          15m
web-2	1/1     Running   0          15m
web-3	1/1     Running   0          15m
web-4	1/1     Running   0          15m
```

### EX9) app=api 또는 app=db인 Pod를 조회하고 전체 레이블을 함께 출력

```bash
[root@k8s-master ~]# kubectl  get  pods  -l 'app in (db,api)'  --show-labels
NAME 	READY   STATUS    RESTARTS   AGE   LABELS
api-1	1/1     Running   0          16m   app=api,env=prod,tier=backend
api-2	1/1     Running   0          16m   app=api,env=prod,tier=backend
api-3 	1/1     Running   0          16m   app=api,env=dev,tier=backend
db-1 	1/1     Running   0          16m   app=db,env=prod,tier=database
db-2  	1/1     Running   0          16m   app=db,env=dev,tier=database
db-3 	1/1     Running   0          16m   app=db,env=prod,tier=database
```

### EX10) env=prod 또는 env=dev인 Pod를 조회하고 app, env 레이블을 컬럼으로 출력

```bash
[root@k8s-master ~]# kubectl  get  pods -l 'env in (prod,dev)' -L app,env
NAME	READY   STATUS    RESTARTS   AGE   APP   ENV
api-1   1/1     Running   0          36m   api   prod
api-2   1/1     Running   0          36m   api   prod
api-3   1/1     Running   0          36m   api   dev
db-1    1/1     Running   0          36m   db    prod
db-2    1/1     Running   0          36m   db    dev
db-3    1/1     Running   0          36m   db    prod
web-1   1/1     Running   0          36m   web   prod
web-2   1/1     Running   0          36m   web   prod
web-3   1/1     Running   0          36m   web   dev
web-4   1/1     Running   0          36m   web   dev
```

### EX11) app=web 또는 app=db인 Pod를 조회하고 app, env 레이블 값을 함께 확인

```bash
[root@k8s-master ~]# kubectl  get  pods  -l 'app in (web,db)'  -L app,env
NAME    	READY   STATUS    RESTARTS   AGE   APP   ENV
db-1    	1/1     Running   0          38m   db    prod
db-2    	1/1     Running   0          38m   db    dev
db-3    	1/1     Running   0          38m   db    prod
web-1   	1/1     Running   0          38m   web   prod
web-2   	1/1     Running   0          38m   web   prod
web-3   	1/1     Running   0          38m   web   dev
web-4   	1/1     Running   0          38m   web   dev
```

> **정리** : `in (값1,값2)` 문법으로 OR 조건을 표현할 수 있으며, `-L` 옵션을 함께 쓰면 해당 Label 값을 컬럼으로 바로 확인할 수 있다.

---

## Label 변경 실습

기존 Label에 새 값을 추가하거나, `--overwrite` 옵션으로 기존 값을 변경하는 실습이다.

### EX12) web-1에 version=v1 Label을 추가

```bash
[root@k8s-master ~]# kubectl  get  pods  web-1  --show-labels
NAME    READY   STATUS    RESTARTS   AGE   LABELS
web-1    1/1         Running     0                41m    app=web,env=prod,tier=frontend


[root@k8s-master ~]# kubectl  label  pod  web-1  version=v1
pod/web-1 labeled


[root@k8s-master ~]# kubectl  get  pods  web-1  --show-labels
NAME    READY   STATUS    RESTARTS   AGE   LABELS
web-1    1/1         Running     0                41m    app=web,env=prod,tier=frontend,version=v1
```

### EX13) web-2에 version=v1 Label을 추가.

```bash
[root@k8s-master ~]# kubectl  get  pods  web-2  --show-labels
NAME    READY   STATUS    RESTARTS   AGE   LABELS
web-2    1/1      Running     0           45m    app=web,env=prod,tier=frontend


[root@k8s-master ~]# kubectl  label  pod  web-2  version=v1
pod/web-2 labeled


[root@k8s-master ~]# kubectl  get  pods  web-2  --show-labels
NAME    READY   STATUS    RESTARTS   AGE   LABELS
web-2    1/1      Running     0           45m    app=web,env=prod,tier=frontend,version=v1
```

### EX14) web-1의 version=v1을 version=v2로 변경

```bash
[root@k8s-master ~]# kubectl  get  pods  web-2  --show-labels
NAME    READY   STATUS    RESTARTS   AGE   LABELS
web-2    1/1      Running     0           45m    app=web,env=prod,tier=frontend


[root@k8s-master ~]# kubectl  label  pod  web-2  version=v1
pod/web-2 labeled


[root@k8s-master ~]# kubectl  get  pods  web-2  --show-labels
NAME    READY   STATUS    RESTARTS   AGE   LABELS
web-2    1/1      Running     0           45m    app=web,env=prod,tier=frontend,version=v1
```

### EX15) api-1의 기존 env=prod를 env=dev로 변경

```bash
[root@k8s-master ~]# kubectl  get  pods  api-1  --show-labels
NAME    READY   STATUS    RESTARTS   AGE   LABELS
api-1    1/1       Running     0            51m   app=api,env=prod,tier=backend


[root@k8s-master ~]# kubectl label pod api-1 env=dev --overwrite
pod/api-1 labeled


[root@k8s-master ~]# kubectl  get  pods  api-1  --show-labels
NAME    READY   STATUS    RESTARTS   AGE   LABELS
api-1     1/1      Running     0            51m   app=api,env=dev,tier=backend
```

> **정리** : 새 Key를 붙이는 것은 단순 추가지만, 이미 존재하는 Key의 값을 바꾸려면 반드시 `--overwrite` 옵션이 필요하다.

---

## Label 삭제 실습

Label의 Key 뒤에 `-`를 붙이면 해당 Label을 삭제할 수 있다.

### EX16) web-3의 app Label을 삭제

```bash
[root@k8s-master ~]# kubectl  get  pods  web-3  --show-labels
NAME    READY   STATUS    RESTARTS   AGE   LABELS
web- 3   1/1         Running     0                54m    app=web,env=dev,tier=frontend


[root@k8s-master ~]# kubectl  label  pod  web-3  app-
pod/web-3 unlabeled


[root@k8s-master ~]# kubectl  get  pods  web-3  --show-labels
NAME    READY   STATUS    RESTARTS   AGE   LABELS
web- 3   1/1         Running     0                54m    env=dev,tier=frontend
```

### EX17) web-1의 version Label을 삭제

```bash
[root@k8s-master ~]# kubectl  get  pods  web-1  --show-labels
NAME    READY   STATUS    RESTARTS   AGE   LABELS
web-1    1/1        Running      0                55m    app=web,env=prod,tier=frontend,version=v1


[root@k8s-master ~]# kubectl  label  pod  web-1  version-
pod/web-1 unlabeled


[root@k8s-master ~]# kubectl  get  pods  web-1  --show-labels
NAME    READY   STATUS    RESTARTS   AGE   LABELS
web-1    1/1        Running      0                56m    app=web,env=prod,tier=frontend
```

> **정리** : `key-` 형태로 명령어를 실행하면 해당 Key의 Label이 삭제된다.

---

## Label Selector와 Service 실습

**Label Selector**를 실제 Service 오브젝트에 적용해 트래픽 라우팅을 확인하는 실습이다.

### EX18) app=web인 Pod를 대상으로 하는 Service를 작성

- Service 이름  : web-service
- Service Type  : ClusterIP
- app=web Selector
- Service Port  : 80
- Target Port   : 80

```bash
[root@k8s-master ~]# vi web-service.yaml
```

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-service
spec:
  type: ClusterIP
  selector:
    app: web
  ports:
    - port: 80
      targetPort: 80
```

```bash
[root@k8s-master ~]# kubectl apply -f web-service.yaml
```

### EX19) web-service의 Selector를 확인

```bash
[root@k8s-master ~]# kubectl describe service web-service
```

다음 부분을 확인한다.

```
Selector: app=web
```

### EX20) web-service가 선택한 Pod의 Endpoint를 확인

```bash
[root@k8s-master ~]# kubectl get endpoints web-service
```

현재 app=web인 Pod들이 Endpoint로 연결된다.

> **정리** : Service의 selector와 Pod의 Label이 일치하면 해당 Pod가 자동으로 Service의 Endpoint에 포함되어 트래픽을 받을 수 있게 된다. 여기까지가 Pod 단위 Label의 핵심 활용이며, 이어서 Node 단위의 Label을 다룬다.

---

---

# Kubernetes Worker Node Label

---

## Node Label이란?

**Node Label**은 Pod가 아닌 Node에 붙이는 Key=Value 형태의 분류 정보로, Pod Label과 개념은 같지만 대상이 Node라는 점이 다르다.

- Node Label은 Kubernetes Node에 Key=Value 형태의 정보를 지정하는 기능이다.
- Pod에 Label을 붙이는 것과 기본적인 개념은 같다.  
  - Key = Value
- 예를 들어 Worker Node에 다음과 같이 Label을 지정할 수 있다.
  - `kubectl label nodes worker1 disktype=ssd`
- 그러면 worker1 Node에 `disktype=ssd` 라는 Label이 추가된다.

> **정리** : Node Label은 `kubectl label nodes` 명령어로 Node에 Key=Value 정보를 부여하는 기능이며, Pod Label과 사용 방식이 동일하다.

---

## Node Label을 사용하는 이유

Node마다 CPU, 메모리, 디스크, GPU, 지역, 용도 등의 특성이 다를 수 있다.

예를 들어 Worker Node가 3대 있다고 하자.

- worker1 : 일반 서버
- worker2 : SSD 장착
- worker3 : GPU 장착

Node Label을 이용하면 다음과 같이 구분할 수 있다.

- worker1 : type=general
- worker2 : type=ssd
- worker3 : type=gpu

이렇게 설정하면 Pod를 아무 Node에나 배치하는 것이 아니라 특정 Label을 가진 Node에 Pod를 배치할 수 있다.

> **정리** : Node마다 다른 하드웨어/용도 특성을 Label로 표현해두면, 이후 Pod 배치 시 원하는 조건의 Node를 선택적으로 지정할 수 있다.

---

## Node Label의 기본 구조

Node에 Label을 추가하는 명령어는 다음과 같다.

```bash
kubectl label nodes <노드 이름> <레이블 키>=<레이블 값>
```

예 : `kubectl label nodes worker1 disktype=ssd`

- Node 이름   : worker1
- Label Key   : disktype
- Label Value : ssd

### Node Label 확인

```bash
# 전체 Node를 확인
kubectl get nodes

# Node의 Label까지 확인
kubectl  get  nodes  --show-labels
```

```
NAME      STATUS   LABELS
worker1   Ready    ... disktype=ssd
worker2   Ready    ... disktype=hdd
worker3   Ready    ... disktype=ssd
```

> **정리** : `kubectl label nodes <노드 이름> <키>=<값>` 형식으로 Node Label을 추가하며, `--show-labels`로 전체 Label을 확인할 수 있다.

---

## 특정 Label을 가진 Node 조회

Node에도 **Label Selector**를 사용할 수 있다.

```bash
kubectl get nodes -l disktype=ssd
```

그러면 `disktype=ssd` Label을 가진 Node만 조회한다.

> **정리** : Pod와 마찬가지로 `-l` 옵션을 사용하면 원하는 Label 조건을 가진 Node만 골라 조회할 수 있다.

---

## Node Label의 가장 중요한 목적

**Node Label**의 핵심은 Pod를 특정 Node에 배치하기 위한 기준을 만드는 것이다.

예를 들어:

- worker1 : disktype=hdd
- worker2 : disktype=ssd
- worker3 : disktype=hdd

특정 Pod를 SSD Node에서 실행하고 싶다면:

```yaml
# Pod
spec:
  nodeSelector:
    disktype: ssd
```

위처럼 파드를 생성시 **nodeSelector**를 구성하게 되면 Kubernetes Scheduler가 조건에 맞는 Node를 찾아 Pod를 배치한다.

```
Pod
 │
 │ nodeSelector
 │
 ↓
disktype=ssd
 │
 ├── worker1 : hdd X
 ├── worker2 : ssd O
 └── worker3 : hdd X
```

> **정리** : nodeSelector는 Pod가 요구하는 Node Label 조건이며, Scheduler는 이 조건과 일치하는 Node를 찾아 Pod를 배치한다.

---

## Node Label과 Pod Label의 차이

### Pod Label

```yaml
metadata:
  labels:
    app: web
```

Pod를 분류하고 선택하기 위해 사용한다.

- Service           : Pod 선택
- Deployment/ReplicaSet : Pod 관리 대상 구분

### Node Label

```
disktype=ssd
```

Node의 특성을 구분하고 Pod를 특정 Node에 배치하기 위해 사용한다.

```
Pod  -->  nodeSelector  -->  Node Label 확인  -->  조건에 맞는 Node에 배치
```

> **정리** : Pod Label은 Pod 자신을 분류·선택하기 위한 정보이고, Node Label은 Node의 특성을 나타내어 Pod 배치 조건으로 사용된다.

---

## Node Label과 nodeSelector

Node Label만 붙인다고 Pod가 자동으로 해당 Node에 배치되는 것은 아니다.

`kubectl label nodes worker1 disktype=ssd` 를 했다고 해서 모든 Pod가 worker1에 배치되는 것은 아니다.  
Pod에서 어떤 Node를 원하는지 조건을 지정해야 한다.

```yaml
# Pod 설정
spec:
  nodeSelector:
    disktype: ssd
```

```
Node
└── Label
    └── disktype=ssd
             ↑
             │ 조건 확인
             │
Pod
└── nodeSelector
    └── disktype=ssd
```

Node Label과 Pod의 nodeSelector 값이 일치해야 한다.

> **정리** : Node Label을 지정하는 것만으로는 Pod가 자동 배치되지 않으며, Pod 쪽에서 nodeSelector로 원하는 Node 조건을 명시해야 한다.

---

## Node Label을 이용한 배치 과정

```
1) Worker Node에 Label 설정
worker1
disktype=ssd
        │
        ↓
2) Pod에 nodeSelector 설정
nodeSelector:
  disktype: ssd
        │
        ↓
3) Scheduler가 Node 확인
worker1 : disktype=ssd  O
worker2 : disktype=hdd  X
worker3 : disktype=hdd  X
        │
        ↓
4) Pod를 worker1에 배치
```

> **정리** : Node Label 설정 → Pod의 nodeSelector 지정 → Scheduler가 조건에 맞는 Node 확인 → 해당 Node에 Pod 배치, 순서로 동작한다.

---

## Node Label 변경

이미 disktype=ssd가 있는 Node에서 값을 변경하려면 `--overwrite`를 사용한다.

```bash
kubectl label nodes worker1 disktype=hdd --overwrite
```

- 기존 : disktype=ssd
- 변경 : disktype=hdd

새로운 Label 추가:

```bash
kubectl label nodes worker1 zone=seoul
```

기존에 zone이라는 Key가 없다면 새로운 Label이 추가된다.

- disktype=ssd
- zone=seoul

Pod Label과 동일하다.

> **정리** : 기존 Key의 값을 바꿀 때는 `--overwrite`, 새 Key를 추가할 때는 그냥 `key=value` 형태로 명령어를 실행하며, 방식은 Pod Label과 동일하다.

---

## Node Label 삭제

Label의 Key 뒤에 `-`를 붙인다.

```bash
kubectl label nodes worker1 zone-
```

`zone=seoul` 이 삭제된다.

> **정리** : Node Label 삭제도 Pod Label과 동일하게 Key 뒤에 `-`를 붙여서 실행한다.

---

## 여러 Node Label 사용

Node 하나에 여러 Label을 지정할 수 있다.

```bash
kubectl label nodes worker1 disktype=ssd
kubectl label nodes worker1 environment=prod
kubectl label nodes worker1 role=web
```

worker1에 다음과 같이 여러 특성을 표현할 수 있다.

- disktype=ssd
- environment=prod
- role=web

> **정리** : 하나의 Node에도 여러 Key=Value Label을 자유롭게 지정할 수 있어 다양한 특성을 동시에 표현할 수 있다.

---

## Node Label의 대표적인 활용 예

예를 들어 Node의 용도를 구분할 수 있다.

- worker1 :  role=web
- worker2 :  role=api
- worker3 :  role=db

특정 Pod를 DB Node에 배치:

```yaml
spec:
  nodeSelector:
    role: db
```

또는 GPU Node:

- worker1 :  gpu=false
- worker2 :  gpu=true
- worker3 :  gpu=false

GPU를 필요로 하는 Pod:

```yaml
spec:
  nodeSelector:
    gpu: "true"
```

이런 식으로 Node의 특성에 따라 Pod의 배치 위치를 제한할 수 있다.

> **정리** : 역할(role), GPU 여부 등 다양한 기준의 Node Label과 nodeSelector 조합으로 워크로드를 원하는 Node에만 배치할 수 있다. 지금부터는 실제 클러스터에서 Node Label을 다루는 실습을 진행한다.

---

## Node Label 실습

### EX1) 현재 Kubernetes 클러스터에 등록된 Node를 확인

```bash
[root@k8s-master ~]# kubectl  get  nodes
NAME          	STATUS   ROLES           AGE   VERSION
k8s-master    	Ready    control-plane   10d   v1.35.7
k8s-worker1	Ready    <none>          10d   v1.35.7
k8s-worker2  	Ready    <none>          10d   v1.35.7
```

> **정리** : 실습에 사용할 클러스터는 control-plane 1대와 worker 2대로 구성되어 있다.

### EX2) Worker Node에 현재 설정된 Label을 확인

```bash
[root@k8s-master ~]# kubectl  get  nodes  --show-labels
NAME          STATUS   ROLES           AGE   VERSION   LABELS
k8s-master    Ready    control-plane   10d   v1.35.7   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=k8s-master,kubernetes.io/os=linux,node-role.kubernetes.io/control-plane=,node.kubernetes.io/exclude-from-external-load-balancers=
k8s-worker1   Ready    <none>          10d   v1.35.7   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=k8s-worker1,kubernetes.io/os=linux
k8s-worker2   Ready    <none>          10d   v1.35.7   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=k8s-worker2,kubernetes.io/os=linux
```

> **정리** : 기본 설치 상태에서는 시스템이 자동으로 부여한 Label만 존재하며, 사용자 정의 Label은 아직 없는 상태이다.

---

## Worker Node에 Label 추가

Worker Node의 디스크 종류를 Label로 구분

- worker1 : disktype=ssd
- worker2 : disktype=hdd

### EX3) k8s-worker1에 disktype=ssd Label을 추가

```bash
[root@k8s-master ~]# kubectl  label  nodes  k8s-worker1  disktype=ssd
node/k8s-worker1 labeled
```

### EX4) k8s-worker2에 disktype=hdd Label을 추가

```bash
[root@k8s-master ~]# kubectl  label  nodes  k8s-worker2  disktype=hdd
node/k8s-worker2 labeled
```

### EX5) worker1, worker2의 Label을 확인

```bash
[root@k8s-master ~]# kubectl  get  nodes  --show-labels
NAME          STATUS   ROLES           AGE   VERSION   LABELS
k8s-master    Ready    control-plane   10d   v1.35.7   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=k8s-master,kubernetes.io/os=linux,node-role.kubernetes.io/control-plane=,node.kubernetes.io/exclude-from-external-load-balancers=
k8s-worker1   Ready    <none>          10d   v1.35.7   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,disktype=ssd,kubernetes.io/arch=amd64,kubernetes.io/hostname=k8s-worker1,kubernetes.io/os=linux
k8s-worker2   Ready    <none>          10d   v1.35.7   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,disktype=hdd,kubernetes.io/arch=amd64,kubernetes.io/hostname=k8s-worker2,kubernetes.io/os=linux
```

> **정리** : k8s-worker1은 disktype=ssd, k8s-worker2는 disktype=hdd Label을 가지도록 구성했다.

---

## Label을 이용하여 Node 선택

### EX6) disktype=ssd Label을 가진 Node만 조회

```bash
# disktype=ssd 레이블이 있는 Node를 출력
[root@k8s-master ~]# kubectl  get  nodes  -l  disktype=ssd
NAME           STATUS   ROLES    AGE   VERSION
k8s-worker1   Ready      <none>    10d     v1.35.7


# disktype=ssd 레이블이 있는 Node의 모든 label을 출력
[root@k8s-master ~]# kubectl  get  nodes  -l  disktype=ssd  --show-labels
NAME          STATUS   ROLES    AGE   VERSION   LABELS
k8s-worker1   Ready    <none>   10d   v1.35.7   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,disktype=ssd,kubernetes.io/arch=amd64,kubernetes.io/hostname=k8s-worker1,kubernetes.io/os=linux


# disktype=ssd 레이블이 있는 Node를 출력시 해당 label을 출력
[root@k8s-master ~]# kubectl  get  nodes  -l  disktype=ssd  -L disktype
NAME           STATUS   ROLES    AGE   VERSION   DISKTYPE
k8s-worker1   Ready      <none>    10d     v1.35.7      ssd
```

### EX7) disktype=hdd Label을 가진 Node만 조회

```bash
# disktype=hdd 레이블이 있는 Node를 출력
[root@k8s-master ~]# kubectl  get  nodes  -l  disktype=hdd
NAME       	STATUS   ROLES      AGE   VERSION
k8s-worker2	Ready      <none>      10d     v1.35.7


# disktype=hdd 레이블이 있는 Node의 모든 label을 출력
[root@k8s-master ~]# kubectl  get  nodes  -l  disktype=hdd  --show-labels
NAME          STATUS   ROLES    AGE   VERSION   LABELS
k8s-worker2   Ready    <none>   10d   v1.35.7   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,disktype=hdd,kubernetes.io/arch=amd64,kubernetes.io/hostname=k8s-worker2,kubernetes.io/os=linux


# disktype=hdd 레이블이 있는 Node를 출력시 해당 label을 출력
[root@k8s-master ~]# kubectl  get  nodes  -l  disktype=hdd  -L disktype
NAME       	STATUS   ROLES    AGE   VERSION   DISKTYPE
k8s-worker2	Ready      <none>    10d     v1.35.7      hdd
```

> **정리** : `-l`로 조건에 맞는 Node만 필터링하고, `-L`을 함께 쓰면 해당 Label 값을 컬럼으로 바로 확인할 수 있다.

---

## Node에 두 번째 Label 추가

Worker Node의 환경을 구분한다.

- worker1 : env=prod
- worker2 : env=dev

### EX8) k8s-worker1에 env=prod Label을 추가

```bash
[root@k8s-master ~]# kubectl  label  nodes  k8s-worker1  env=prod

[root@k8s-master ~]# kubectl  get  nodes  -l  env=prod  --show-labels

[root@k8s-master ~]# kubectl  get  nodes  -l  env=prod  -L env
```

### EX9) k8s-worker2에 env=dev Label을 추가

```bash
[root@k8s-master ~]# kubectl  label  nodes  k8s-worker2  env=dev

[root@k8s-master ~]# kubectl  get  nodes  -l  env=dev  --show-labels

[root@k8s-master ~]# kubectl  get  nodes  -l  env=dev  -L env
```

```
worker1
    ├── disktype=ssd
    └── env=prod

worker2
    ├── disktype=hdd
    └── env=dev
```

> **정리** : 이제 각 worker Node는 disktype과 env, 두 종류의 Label을 함께 갖게 되었다.

---

## 여러 Label을 이용해 Node 선택

### EX10) disktype=ssd이면서 env=prod인 Node를 조회

```bash
[root@k8s-master ~]# kubectl  get  nodes  -l  disktype=ssd,env=prod
NAME      	STATUS   ROLES    AGE   VERSION
k8s-worker1	Ready      <none>    10d     v1.35.7


[root@k8s-master ~]# kubectl  get  nodes  -l  disktype=ssd,env=prod  -L  disktype,env
NAME      	STATUS   ROLES    AGE   VERSION   DISKTYPE   ENV
k8s-worker1	Ready      <none>    10d     v1.35.7      ssd             prod
```

- 쉼표로 여러 조건을 지정하면 AND 조건이다.
- disktype=ssd AND env=prod
- 두 조건을 모두 만족하는 Node만 선택된다.

> **정리** : Pod Label 조회와 마찬가지로 Node Label도 쉼표로 여러 조건을 나열하면 AND 조건으로 필터링된다.

---

## Node Label을 사용하여 Pod를 특정 Node에 배치

현재 Node:

- k8s-worker1 : disktype=ssd, env=prod
- k8s-worker2 : disktype=hdd, env=dev

### EX11) 다음 조건을 만족하는 Pod YAML 파일을 작성

- Pod 이름  = ssd-pod
- Image     = nginx:1.31
- disktype=ssd인 Node에 배치
- nodeSelector 사용

```bash
[root@k8s-master ~]# kubectl  delete  pods  --all


[root@k8s-master ~]# vi ssd-pod.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: ssd-pod
spec:
  nodeSelector:
    disktype: ssd

  containers:
    - name: nginx-container
      image: nginx:1.31
```

```bash
[root@k8s-master ~]# kubectl  apply  -f  ssd-pod.yaml
pod/ssd-pod created


[root@k8s-master ~]# kubectl  get  pods  -o  wide
NAME      READY   STATUS    RESTARTS   AGE   IP               NODE           NOMINATED NODE   READINESS GATES
ssd-pod    1/1         Running     0                17s     10.244.1.16   k8s-worker1   <none>                   <none>
```

> **정리** : Pod의 nodeSelector가 `disktype: ssd`이므로 Scheduler는 disktype=ssd Label을 가진 k8s-worker1에 Pod를 배치했다.

---

## HDD Node에 Pod 배치

### EX12) 다음 조건을 만족하는 Pod를 생성

- Pod 이름  : hdd-pod
- Image     : nginx:1.31
- disktype=hdd인 Node에 배치

```bash
[root@k8s-master ~]# vi hdd-pod.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: hdd-pod
spec:
  nodeSelector:
    disktype: hdd
  containers:
    - name: nginx
      image: nginx:1.31
```

```bash
[root@k8s-master ~]# kubectl  get  pods  -o  wide
NAME	READY   STATUS    RESTARTS   AGE     IP              NODE            NOMINATED NODE   READINESS GATES
hdd-pod	1/1         Running     0                3s        10.244.2.23   k8s-worker2   <none>           	   <none>
ssd-pod	1/1         Running     0                5m43s   10.244.1.16   k8s-worker1   <none>           	   <none>
```

> **정리** : 동일한 방식으로 disktype=hdd Node에는 hdd-pod가 배치되어, nodeSelector 조건에 따라 각기 다른 Node에 Pod가 분산 배치됨을 확인할 수 있다.

---

## Node Label 변경 실습

### EX13) 현재 다음과 같은 상태이다.

- worker1 : disktype=ssd
- worker2 : disktype=hdd
- worker2의 disktype=hdd를 disktype=ssd로 변경

```bash
[root@k8s-master ~]# kubectl label nodes k8s-worker2 disktype=ssd --overwrite


[root@k8s-master ~]# kubectl  get  nodes  -l  disktype=ssd  -L  disktype,env
NAME          STATUS   ROLES    AGE   VERSION   DISKTYPE   ENV
k8s-worker1   Ready      <none>   10d    v1.35.7      ssd         prod
k8s-worker2   Ready      <none>   10d    v1.35.7      ssd         dev
```

- worker2에는 이미 disktype=hdd 라는 같은 Key가 존재한다.  
  따라서 `kubectl label nodes worker2 disktype=ssd` 만 실행하면 오류가 발생하고  
  `--overwrite` 를 사용해야 값을 변경할 수 있다.

- Node의 Label을 바꿔도 이미 실행 중인 Pod는 자동으로 다른 Node로 이동하지 않는다.
- nodeSelector가 Pod를 처음 스케줄링할 때 사용할 Node를 선택하는 조건이므로  
  이미 배치된 Pod를 다시 검사해서 다른 Node로 이동시키지는 않는다.

> **정리** : Node Label 변경은 향후 스케줄링될 Pod에만 영향을 주며, 이미 실행 중인 Pod는 재배치되지 않는다.

---

### EX14) 현재 Node가 다음과 같다.

- k8s-worker1 : disktype=ssd, env=prod
- k8s-worker2 : disktype=ssd, env=dev
- disktype=gpu label이 있는 k8s-worker에 Pod를 생성

```bash
[root@k8s-master ~]# vi gpu-pod.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: gpu-pod
spec:
  nodeSelector:
    disktype: gpu
  containers:
    - name: nginx-container
      image: nginx:1.31
```

```bash
[root@k8s-master ~]# kubectl  apply  -f  gpu-pod.yaml
pod/gpu-pod created


[root@k8s-master ~]# kubectl  get  pods  -o  wide
NAME  	READY   STATUS	  RESTARTS   AGE     IP               NODE     	NOMINATED NODE   READINESS GATES
gpu-pod	0/1         Pending	  0                 2m14s  <none>        <none>      	<none>                    <none>
hdd-pod	1/1         Running	  0                 12m     10.244.2.23   k8s-worker2	<none>                    <none>
ssd-pod	1/1         Running	  0                 18m     10.244.1.16   k8s-worker1	<none>                    <none>
```

Pod가 Pending 상태가 된다.

현재 Node:

- k8s-worker1 : disktype=ssd, env=prod
- k8s-worker2 : disktype=ssd, env=dev

Pod는 disktype=gpu 인 Node를 요구한다.

- worker1 : ssd
- worker2 : hdd
- 조건을 만족하는 Node가 없기 때문에 Scheduler가 Pod를 배치할 수 없다.

> **정리** : nodeSelector 조건을 만족하는 Node가 클러스터에 하나도 없으면 Pod는 Pending 상태로 남아 스케줄링되지 못한다.

---

## 문제 상황 해결

Pending 상태인 Pod를 배치시키려면 조건에 맞는 Node Label을 추가해주면 된다.

### EX19) 앞의 gpu-pod가 Pending 상태이다.

k8s-worker2에 disktype=gpu Label을 추가하여 gpu-pod가 배치될 수 있어야 한다.

```bash
[root@k8s-master ~]# kubectl  label  nodes  k8s-worker2  disktype=gpu  --overwrite
node/k8s-worker2 labeled


[root@k8s-master ~]# kubectl  get  pods  -o  wide
NAME  	READY   STATUS    RESTARTS   AGE     IP               NODE          NOMINATED NODE   READINESS GATES
gpu-pod	1/1         Running     0                5m28s   10.244.2.24   k8s-worker2   <none>           	  <none>
hdd-pod	1/1         Running     0                15m      10.244.2.23   k8s-worker2   <none>           	  <none>
ssd-pod 	1/1         Running     0                21m      10.244.1.16   k8s-worker1   <none>           	  <none>


[root@k8s-master ~]# kubectl  get  nodes  -L  disktype,env
NAME          	STATUS   ROLES           	AGE   VERSION   DISKTYPE   ENV
k8s-master    	Ready      control-plane	10d     v1.35.7
k8s-worker1  	Ready      <none>          	10d     v1.35.7     ssd              prod
k8s-worker2	Ready      <none>          	10d     v1.35.7     gpu              dev
```

> **정리** : k8s-worker2에 disktype=gpu Label을 추가하자 Pending 상태였던 gpu-pod가 조건을 만족하는 Node를 찾아 정상적으로 배치되었다. 이처럼 Node Label과 nodeSelector 조합은 워크로드를 원하는 Node에 정확히 배치하기 위한 핵심 메커니즘이다.

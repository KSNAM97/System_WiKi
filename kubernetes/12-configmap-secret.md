# ConfigMap · Secret

> 애플리케이션의 설정값과 민감한 데이터를 이미지와 분리해서 관리하는 ConfigMap/Secret 오브젝트, 생성 방법, Pod에서 사용하는 방법(환경변수/파일 Mount)을 실습으로 정리한다.

## 목차

1. [ConfigMap 개념](#configmap-개념)
2. [ConfigMap을 사용하는 이유](#configmap을-사용하는-이유)
3. [ConfigMap에 저장할 수 있는 값](#configmap에-저장할-수-있는-값)
4. [ConfigMap 생성 방법](#configmap-생성-방법)
5. [ConfigMap 확인 명령어](#configmap-확인-명령어)
6. [ConfigMap을 Pod에서 사용하는 방법](#configmap을-pod에서-사용하는-방법)
7. [ConfigMap 실습 1) 환경변수로 개별 전달](#configmap-실습-1-환경변수로-개별-전달)
8. [ConfigMap 실습 2) 전체를 환경변수로 사용](#configmap-실습-2-전체를-환경변수로-사용)
9. [ConfigMap 실습 3) 파일로 Mount](#configmap-실습-3-파일로-mount)
10. [ConfigMap 파일 Mount 종합 실습 (Deployment)](#configmap-파일-mount-종합-실습-deployment)
11. [ConfigMap 수정 실습 (etcd 반영 확인)](#configmap-수정-실습-etcd-반영-확인)
12. [Secret 개념](#secret-개념)
13. [Secret을 사용하는 이유](#secret을-사용하는-이유)
14. [ConfigMap과 Secret의 차이](#configmap과-secret의-차이)
15. [Secret의 데이터 구조](#secret의-데이터-구조)
16. [Secret의 종류](#secret의-종류)
17. [Secret 생성 방법](#secret-생성-방법)
18. [Secret 저장값 확인](#secret-저장값-확인)
19. [Secret을 Pod에서 사용하는 방법](#secret을-pod에서-사용하는-방법)
20. [Secret과 환경변수/Volume의 관계](#secret과-환경변수volume의-관계)
21. [Secret 실습 (Deployment)](#secret-실습-deployment)
22. [Secret + MySQL 실습](#secret--mysql-실습)

---

## ConfigMap 개념

ConfigMap은 애플리케이션에서 사용하는 설정값(Configuration)을 저장하는 Kubernetes 오브젝트이다.

예를 들어 애플리케이션에서 다음과 같은 값을 사용한다고 하자.

- 개발/운영 환경 구분
- DB 주소
- 서버 포트
- 로그 레벨
- 애플리케이션 이름
- 외부 API 주소

이런 민감하지 않은 설정값을 Pod의 이미지에 직접 넣지 않고 ConfigMap에 저장해 사용할 수 있다.

## ConfigMap을 사용하는 이유

**기존 방식**

애플리케이션 이미지 안에 설정값을 넣으면 환경이 변경될 때마다 이미지를 다시 만들어야 한다.

```
Docker Image
 └─ application
          └─ 설정값
```

개발 환경에서 운영 환경으로 변경하려면 이미지 수정이 필요하다.

**ConfigMap 사용**

```
Docker Image
   └─ application

ConfigMap
  └─ 환경별 설정값
```

애플리케이션이 ConfigMap의 설정값을 외부에서 받아 사용하도록 구성하기 때문에 이미지는 그대로 사용하고 ConfigMap만 변경할 수 있다. 즉, 애플리케이션 코드/이미지와 환경 설정을 분리하기 위해 사용한다.

## ConfigMap에 저장할 수 있는 값

ConfigMap에는 일반적인 문자열 형태의 설정값을 저장할 수 있다.

예:

```
APP_NAME=spring-app
APP_ENV=dev
LOG_LEVEL=INFO
SERVER_PORT=8080
DB_HOST=mysql
```

단, 비밀번호, API Key, 인증서 같은 민감한 정보는 ConfigMap이 아니라 Secret을 사용하는 것이 적절하다.

- ConfigMap : 일반적인 설정값
- Secret : 비밀번호, 인증정보 등 민감한 데이터

## ConfigMap 생성 방법

ConfigMap은 여러 방법으로 생성할 수 있다.

**1) 명령어로 생성**

```bash
kubectl create configmap <config-map이름> --from-literal=<키>=<값>
```

예:

```bash
kubectl create configmap app-config \
  --from-literal=APP_NAME=spring-app \	# 생성될 Config-map 이름 (app-config)
  --from-literal=APP_ENV=dev \		# 저장할 Key=Value (APP_ENV=dev)
  --from-literal=LOG_LEVEL=INFO		# 저장할 Key=Value (LOG_LEVEL=INFO)
```

- `app-config` : 생성할 ConfigMap 이름
- `APP_NAME` : Key, `spring-app` : Value
- `APP_ENV` : Key, `dev` : Value
- `LOG_LEVEL` : Key, `INFO` : Value

**2) YAML로 생성**

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config		# 생성될 Config-map 이름 (app-config)

data:
  APP_NAME: spring-app	# 저장할 Key=Value (APP_NAME=spring-app)
  APP_ENV: dev		# 저장할 Key=Value (APP_ENV=dev)
  LOG_LEVEL: INFO		# 저장할 Key=Value (LOG_LEVEL=INFO)
```

- `app-config` : 생성할 ConfigMap 이름
- `APP_NAME` : Key, `spring-app` : Value
- `APP_ENV` : Key, `dev` : Value
- `LOG_LEVEL` : Key, `INFO` : Value

```
        ConfigMap : app-config
========================================================
|	Key		| Value			|
========================================================
|  APP_NAME		| spring-app		|
--------------------------------------------------------
|   APP_ENV		| dev			|
--------------------------------------------------------
|  LOG_LEVEL		| INFO			|
========================================================
```

## ConfigMap 확인 명령어

ConfigMap 목록:

```bash
kubectl get configmap
kubectl get cm
```

특정 ConfigMap 확인:

```bash
kubectl get configmap app-config
kubectl describe configmap app-config
```

YAML 형태로 확인:

```bash
kubectl get configmap app-config -o yaml
```

## ConfigMap을 Pod에서 사용하는 방법

ConfigMap의 값을 Pod에서 사용하는 대표적인 방법은 2가지이다.

**방법 1) 환경변수로 전달**

```
ConfigMap  --->  Environment Variable  --->  Container
```

예:

```yaml
env:
  - name: APP_NAME
    valueFrom:
      configMapKeyRef:
        name: app-config
        key: APP_NAME
```

컨테이너 내부에서는 `echo $APP_NAME` 명령어로 확인이 가능하다.
결과 : `APP_NAME: spring-app`

**방법 2) 파일로 Mount**

```
        ConfigMap : app-config
========================================================
|	Key		| Value			|
========================================================
|  APP_NAME		| spring-app		|
--------------------------------------------------------
|   APP_ENV		| dev			|
--------------------------------------------------------
|  LOG_LEVEL		| INFO			|
========================================================
```

ConfigMap을 Pod 내부의 파일로 사용할 수도 있다.

```
ConfigMap  --->  Volume  --->  Container 내부 파일
```

예:

```yaml
volumes:				# Pod에서 사용할 Volume을 정의
  - name: config-volume
    configMap:			# Volume의 데이터 원본이 ConfigMap이라는 의미
      name: app-config		# 어떤 ConfigMap을 Volume으로 사용할 것인지 지정

volumeMounts:			# Volume을 Container 내부에 연결(Mount)하는 설정
  - name: config-volume		# 위에서 만든 "config-volume" Volume을 지정
    mountPath: /etc/config		# Container 내부에서 Volume을 어느 디렉터리에 연결할 것인지 지정
```

그러면 ConfigMap의 각 key가 파일 형태로 만들어진다.

- `/etc/config/APP_NAME`
- `/etc/config/APP_ENV`
- `/etc/config/LOG_LEVEL`
- APP_NAME 이름의 파일이 만들어지고 파일이름이 key가 되고 안의 값이 value가 된다.
- APP_ENV 이름의 파일이 만들어지고 파일이름이 key가 되고 안의 값이 value가 된다.
- LOG_LEVEL 이름의 파일이 만들어지고 파일이름이 key가 되고 안의 값이 value가 된다.

## ConfigMap 실습 1) 환경변수로 개별 전달

**ConfigMap 생성**

```bash
[root@k8s-master ~]# kubectl create configmap app-config \
  --from-literal=APP_NAME=spring-app \
  --from-literal=APP_ENV=dev \
  --from-literal=LOG_LEVEL=INFO
```

```bash
[root@k8s-master ~]# kubectl  get  configmaps
NAME               	DATA   	AGE
app-config         	3      	20s
kube-root-ca.crt	1      	16d
```

```bash
[root@k8s-master ~]# kubectl  get  configmaps  app-config
NAME         	DATA   	AGE
app-config   	3      	55s
```

**config-map 사용 X**

```bash
[root@k8s-master ~]# vi configmap-pod.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: no-configmap-pod

spec:
  containers:
    - name: nginx-container
      image: nginx:1.31

      env:
        - name: APP_NAME
          value: "spring-app"

        - name: APP_ENV
          value: "dev"

        - name: LOG_LEVEL
          value: "INFO"
```

```
        ConfigMap : app-config
========================================================
|	Key		| Value			|
========================================================
|  APP_NAME		| spring-app		|
--------------------------------------------------------
|   APP_ENV		| dev			|
--------------------------------------------------------
|  LOG_LEVEL		| INFO			|
========================================================
```

**config-map 사용 O**

```bash
[root@k8s-master ~]# vi configmap-pod.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: configmap-pod
spec:
  containers:
    - name: nginx-container
      image: nginx:1.31

      env:
        - name: APP_NAME
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: APP_NAME

        - name: APP_ENV
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: APP_ENV

        - name: LOG_LEVEL
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: LOG_LEVEL
```

```bash
[root@k8s-master ~]# kubectl apply -f configmap-pod.yaml
pod/configmap-pod created
```

```bash
[root@k8s-master ~]# kubectl  get  pods configmap-pod
NAME            	READY   STATUS    RESTARTS   AGE
configmap-pod	1/1         Running     0                 22s
```

```bash
[root@k8s-master ~]# kubectl  exec  -it  configmap-pod  -- /bin/bash
root@configmap-pod:/#

root@configmap-pod:/# echo $APP_NAME
spring-app

root@configmap-pod:/# echo $APP_ENV
dev

root@configmap-pod:/# echo $LOG_LEVEL
INFO
```

```bash
[root@k8s-master ~]# kubectl  exec  configmap-pod  -- env | grep -E 'APP_NAME|APP_ENV|LOG_LEVEL'
LOG_LEVEL=INFO
APP_NAME=spring-app
APP_ENV=dev
```

## ConfigMap 실습 2) 전체를 환경변수로 사용

ConfigMap의 값을 하나씩 지정하는 대신 전체 key-value를 한꺼번에 환경변수로 가져올 수도 있다.

```
        ConfigMap : app-config
========================================================
|	Key		| Value			|
========================================================
|  APP_NAME		| spring-app		|
--------------------------------------------------------
|   APP_ENV		| dev			|
--------------------------------------------------------
|  LOG_LEVEL		| INFO			|
========================================================
```

```bash
[root@k8s-master ~]# vi configmap-all.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: configmap-envfrom-pod

spec:
  containers:
    - name: nginx
      image: nginx:1.31

      envFrom:
        - configMapRef:
            name: app-config
```

```bash
[root@k8s-master ~]# kubectl  apply  -f configmap-all.yaml
pod/configmap-envfrom-pod created
```

```bash
[root@k8s-master ~]# kubectl  exec  configmap-envfrom-pod  -- env | grep -E 'APP_NAME|APP_ENV|LOG_LEVEL'
APP_NAME=spring-app
LOG_LEVEL=INFO
APP_ENV=dev
```

## ConfigMap 실습 3) 파일로 Mount

**3-1) ConfigMap 생성**

```bash
[root@k8s-master ~]# kubectl create configmap file-config \
  --from-literal=APP_NAME=spring-app \
  --from-literal=APP_ENV=dev \
  --from-literal=LOG_LEVEL=INFO
```

```bash
[root@k8s-master ~]# kubectl  get  configmaps
NAME               	DATA   	AGE
app-config         	3      	39m
file-config        	3      	6s
kube-root-ca.crt	1      	16d
```

```bash
[root@k8s-master ~]# vi configmap-volume-pod.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: configmap-volume-pod

spec:
  containers:
    - name: nginx
      image: nginx:1.31

      volumeMounts:
        - name: config-volume
          mountPath: /etc/config

  volumes:
    - name: config-volume
      configMap:
        name: file-config
```

**컨테이너에 접속 후 확인**

```bash
[root@k8s-master ~]# kubectl  apply  -f  configmap-volume-pod.yaml
pod/configmap-volume-pod created
```

```bash
[root@k8s-master ~]# kubectl  exec  -it  configmap-volume-pod  --  /bin/bash
root@configmap-volume-pod:/#

root@configmap-volume-pod:/# ls  -l  /etc/config/
total 0
lrwxrwxrwx 1 root root 14 Aug 28 01:36 APP_ENV -> ..data/APP_ENV
lrwxrwxrwx 1 root root 15 Aug 28 01:36 APP_NAME -> ..data/APP_NAME
lrwxrwxrwx 1 root root 16 Aug 28 01:36 LOG_LEVEL -> ..data/LOG_LEVEL

root@configmap-volume-pod:/# cat  /etc/config/APP_ENV
dev

root@configmap-volume-pod:/# cat  /etc/config/APP_NAME
spring-app

root@configmap-volume-pod:/# cat  /etc/config/LOG_LEVEL
INFO
```

**컨테이너에 접속하지 않고 확인**

```bash
[root@k8s-master ~]# kubectl  exec  configmap-volume-pod  --  ls  -l  /etc/config
total 0
lrwxrwxrwx 1 root root 14 Aug 28 01:36 APP_ENV -> ..data/APP_ENV
lrwxrwxrwx 1 root root 15 Aug 28 01:36 APP_NAME -> ..data/APP_NAME
lrwxrwxrwx 1 root root 16 Aug 28 01:36 LOG_LEVEL -> ..data/LOG_LEVEL

[root@k8s-master ~]# kubectl  exec  configmap-volume-pod  --  cat  /etc/config/APP_NAME
spring-app

[root@k8s-master ~]# kubectl  exec  configmap-volume-pod  --  cat  /etc/config/APP_ENV
dev

[root@k8s-master ~]# kubectl  exec  configmap-volume-pod  --  cat  /etc/config/LOG_LEVEL
INFO
```

## ConfigMap 파일 Mount 종합 실습 (Deployment)

ConfigMap에 File을 저장한 후 해당 파일을 Key=Value 형태로 사용한다.

```
ConfigMap  -->  Deployment  -->  Pod  -->  Volume  -->  index.html
```

**STEP 1) 실습 디렉터리 생성**

```bash
[root@k8s-master ~]# mkdir  configmap-test
[root@k8s-master ~]# cd configmap-test/
[root@k8s-master configmap-test]# pwd
/root/configmap-test
```

**STEP 2) index.html 파일 생성**

```bash
[root@k8s-master configmap-test]# vi index.html
```

```html
<!DOCTYPE html>
<html>
<head>
    <title>ConfigMap 실습</title>
</head>
<body>
    <h1>쿠버네티스 ConfigMap 실습</h1>

    <p>ConfigMap으로 전달된 index.html입니다.</p>
</body>
</html>
```

index.html을 ConfigMap에 저장한다. ConfigMap으로 생성하면 다음과 같은 구조가 된다.

```
ConfigMap : web-page
   Key
    └── index.html
   Value
    └── index.html 파일 내용
```

- Key = `index.html`
- Value = HTML 코드 전체

```
        ConfigMap : web-page
========================================================
|	Key		| Value			|
========================================================
|  index.html		| index.html 파일 내용	|
========================================================
```

```bash
[root@k8s-master configmap-test]# kubectl  create  configmap  web-page  --from-file=index.html
configmap/web-page created
```

```bash
[root@k8s-master configmap-test]# kubectl  get  configmaps
NAME               	DATA   	AGE
app-config         	3      	57m
file-config        	3      	17m
kube-root-ca.crt   	1      	16d
web-page           	1      	15s
```

**STEP 4) ConfigMap 내용 확인**

web-page ConfigMap에 index.html이 정상적으로 저장되었는지 확인:

```
Name:         	web-page
Namespace:    	default
Labels:       	<none>
Annotations:  	<none>

Data
====
index.html:
----
<!DOCTYPE html>
<html>
<head>
    <title>ConfigMap 실습</title>
</head>
<body>
    <h1>쿠버네티스 ConfigMap 실습</h1>

    <p>ConfigMap으로 전달된 index.html입니다.</p>
</body>
</html>

BinaryData
====

Events:  <none>
```

```bash
[root@k8s-master configmap-test]# kubectl  get  configmaps  web-page  -o  yaml
```

```yaml
apiVersion: v1
data:
  index.html: |
    <!DOCTYPE html>
    <html>
    <head>
        <title>ConfigMap 실습</title>
    </head>
    <body>
        <h1>쿠버네티스 ConfigMap 실습</h1>

        <p>ConfigMap으로 전달된 index.html입니다.</p>
    </body>
    </html>
kind: ConfigMap
metadata:
  creationTimestamp: "2026-08-28T01:51:14Z"
  name: web-page
  namespace: default
  resourceVersion: "574377"
  uid: 983de534-262d-4925-816f-cd19af8b187e
```

**STEP 5) local-config ConfigMap 생성**

Container의 문자 환경을 UTF-8로 설정하기 위한 값을 ConfigMap에 저장한다.

Container에서 사용할 LANG과 LC_ALL 환경변수를 ConfigMap으로 생성:

- ConfigMap 이름: `local-config`
- Key = `LANG`, Value = `C.UTF-8`
- Key = `LC_ALL`, Value = `C.UTF-8`

```bash
[root@k8s-master configmap-test]# kubectl create configmap local-config \
  --from-literal=LANG=C.UTF-8 \
  --from-literal=LC_ALL=C.UTF-8
```

```
       ConfigMap : local-config
======================================
| Key      	| Value        	|
======================================
| LANG    	| C.UTF-8      	|
--------------------------------------
| LC_ALL  	| C.UTF-8      	|
======================================
```

```bash
[root@k8s-master configmap-test]# kubectl get configmap
NAME               	DATA	AGE
app-config         	3      	63m
file-config        	3      	24m
kube-root-ca.crt   	1      	16d
local-config       	2      	11s
web-page           	1      	6m38s
```

모든 ConfigMap의 상세 정보 확인:

```bash
[root@k8s-master configmap-test]# kubectl  describe  configmaps
```

특정 ConfigMap의 상세 정보 확인:

```bash
[root@k8s-master configmap-test]# kubectl  describe  configmaps  local-config
Name:         local-config
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
LANG:
----
C.UTF-8

LC_ALL:
----
C.UTF-8

BinaryData
====

Events:  <none>
```

**STEP 6) Deployment YAML 작성**

다음 조건에 맞는 Deployment를 작성한다.

- Deployment 이름: `web-deployment` (Pod 2개)
- Container 이름: `nginx-container`
- Image: `nginx:1.31`
- ConfigMap `web-page`를 Volume으로 사용
- `/usr/share/nginx/html`에 Mount
- ConfigMap `local-config`를 환경변수로 사용

```bash
[root@k8s-master configmap-test]# vi web-deployment.yaml
```

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-deployment
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
      containers:
        - name: nginx-container
          image: nginx:1.31

          envFrom:
            - configMapRef:
                name: local-config

          volumeMounts:
            - name: html-volume
              mountPath: /usr/share/nginx/html

      volumes:
        - name: html-volume
          configMap:
            name: web-page
```

local-config ConfigMap의 모든 Key-Value를 Container의 환경변수로 사용:

```yaml
envFrom:
  - configMapRef:
      name: local-config
```

따라서 Container에는 다음 환경변수가 적용된다.

```
LANG=C.UTF-8
LC_ALL=C.UTF-8
```

web-page ConfigMap을 html-volume이라는 Volume으로 사용:

```yaml
volumes:
  - name: html-volume
    configMap:
      name: web-page
```

html-volume을 Container의 `/usr/share/nginx/html`에 Mount 한다.

```yaml
volumeMounts:
  - name: html-volume
    mountPath: /usr/share/nginx/html
```

전체 흐름:

```
web-page ConfigMap --> html-volume --> /usr/share/nginx/html
local-config ConfigMap --> 환경변수 --> Container
```

**STEP 7) Deployment 생성 및 확인**

작성한 Deployment YAML을 Kubernetes에 적용하고 Deployment와 Pod가 정상적으로 생성되었는지 확인한다.

```bash
[root@k8s-master configmap-test]# kubectl  apply  -f  web-deployment.yaml
deployment.apps/web-deployment created
```

```bash
[root@k8s-master configmap-test]# kubectl  get  deployments  web-deployment
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
web-deployment   2/2        2                    2                  35s
```

```bash
[root@k8s-master configmap-test]# kubectl  get  pods
NAME                              		READY   STATUS    RESTARTS   AGE
web-deployment-6d5df754c5-2hxsn   	1/1         Running     0                115s
web-deployment-6d5df754c5-gqkn9   	1/1         Running     0                115s
```

Deployment가 생성되고 `replicas: 2`에 따라 Pod 2개를 생성한다. Deployment의 Pod Template에 local-config와 web-page 사용 설정이 적용된다.

**STEP 8) Container의 LANG, LC_ALL 확인**

```bash
[root@k8s-master configmap-test]# kubectl exec web-deployment-6d5df754c5-2hxsn -- /bin/bash  -c 'echo $LANG'
C.UTF-8

[root@k8s-master configmap-test]# kubectl exec web-deployment-6d5df754c5-2hxsn -- /bin/bash  -c 'echo $LC_ALL'
C.UTF-8
```

```bash
[root@k8s-master configmap-test]# kubectl exec web-deployment-6d5df754c5-2hxsn -- env |  grep -E 'LANG|LC_ALL'
LANG=C.UTF-8
LC_ALL=C.UTF-8
```

local-config ConfigMap의 `LANG=C.UTF-8`, `LC_ALL=C.UTF-8` 값이 envFrom을 통해 Container의 환경변수로 적용된다.

**STEP 9) Pod 내부의 index.html 확인**

```bash
[root@k8s-master configmap-test]# kubectl exec web-deployment-6d5df754c5-2hxsn  --  ls  -l  /usr/share/nginx/html
total 0
lrwxrwxrwx 1 root root 17 Aug 28 02:13 index.html -> ..data/index.html
```

구조:

```
ConfigMap
   └── index.html
                ↓
            Volume
                ↓
Container
   └── /usr/share/nginx/html/index.html
```

```bash
[root@k8s-master configmap-test]# kubectl exec web-deployment-6d5df754c5-2hxsn -- cat  /usr/share/nginx/html/index.html
<!DOCTYPE html>
<html>
<head>
    <title>ConfigMap 실습</title>
</head>
<body>
    <h1>쿠버네티스 ConfigMap 실습</h1>

    <p>ConfigMap으로 전달된 index.html입니다.</p>
</body>
</html>
```

```bash
[root@k8s-master configmap-test]# kubectl exec  -it  web-deployment-6d5df754c5-2hxsn  --  /bin/bash
root@web-deployment-6d5df754c5-2hxsn:/#

root@web-deployment-6d5df754c5-2hxsn:/# cat  /usr/share/nginx/html/index.html
<!DOCTYPE html>
<html>
<head>
    <title>ConfigMap 실습</title>
</head>
<body>
    <h1>쿠버네티스 ConfigMap 실습</h1>

    <p>ConfigMap으로 전달된 index.html입니다.</p>
</body>
</html>
```

모든 Pod에 동일한 web-page ConfigMap이 Mount된다.

```
                               Deployment
                                    │
        ┌─────────┴─────────┐
        ↓                                                      ↓
      Pod 1                                                Pod 2
        │                                                     │
        ↓                                                      ↓
   index.html                                           index.html
```

```bash
[root@k8s-master configmap-test]# kubectl  get  pods  -o  wide
NAME                              		READY   STATUS    RESTARTS   AGE   IP             NODE            NOMINATED NODE   READINESS GATES
web-deployment-6d5df754c5-2hxsn   	1/1        Running      0               14m     10.244.2.6   k8s-worker2   <none>           <none>
web-deployment-6d5df754c5-gqkn9   	1/1        Running      0               14m     10.244.1.5   k8s-worker1   <none>           <none>
```

```bash
[root@k8s-master configmap-test]# curl  http://10.244.2.6
<!DOCTYPE html>
<html>
<head>
    <title>ConfigMap 실습</title>
</head>
<body>
    <h1>쿠버네티스 ConfigMap 실습</h1>

    <p>ConfigMap으로 전달된 index.html입니다.</p>
</body>
</html>
```

```bash
[root@k8s-master configmap-test]# curl  http://10.244.1.5
<!DOCTYPE html>
<html>
<head>
    <title>ConfigMap 실습</title>
</head>
<body>
    <h1>쿠버네티스 ConfigMap 실습</h1>

    <p>ConfigMap으로 전달된 index.html입니다.</p>
</body>
</html>
```

## ConfigMap 수정 실습 (etcd 반영 확인)

**STEP 10) ConfigMap의 index.html 수정**

ConfigMap은 Kubernetes API 리소스이므로 etcd에 저장된다.

web-page ConfigMap의 index.html에 다음 내용을 추가한다.

```bash
[root@k8s-master configmap-test]# kubectl  edit  configmaps  web-page
```

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">		<!-- 추가 설정 -->
    <title>ConfigMap 실습</title>
</head>
<body>
    <h1>쿠버네티스 ConfigMap 실습</h1>

    <p>ConfigMap으로 전달된 index.html입니다.</p>

    <h2>ConfigMap 수정 테스트</h2>				<!-- 추가 설정 -->
    <p>이 내용은 ConfigMap을 수정한 후 추가되었습니다.</p>		<!-- 추가 설정 -->
    <p>현재 버전: Version 2</p>				<!-- 추가 설정 -->
</body>
</html>
```

ConfigMap은 Kubernetes API 리소스이므로 생성하거나 수정한 내용은 kube-apiserver를 통해 etcd에 저장된다. 즉, `kubectl edit configmap web-page` 명령으로 내용을 수정하면 변경된 ConfigMap 데이터가 Kubernetes의 상태 정보로 etcd에 저장된다.

**컨테이너에 접속 후 변경 확인**

```bash
[root@k8s-master configmap-test]# kubectl  exec  -it web-deployment-6d5df754c5-2hxsn  --  /bin/bash
root@web-deployment-6d5df754c5-2hxsn:/#
root@web-deployment-6d5df754c5-2hxsn:/#

root@web-deployment-6d5df754c5-2hxsn:/# cat /usr/share/nginx/html/index.html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>ConfigMap 실습</title>
</head>
<body>
    <h1>쿠버네티스 ConfigMap 실습</h1>

    <p>ConfigMap으로 전달된 index.html입니다.</p>
    <h2>ConfigMap 수정 테스트</h2>
    <p>이 내용은 ConfigMap을 수정한 후 추가되었습니다.</p>
    <p>현재 버전: Version 2</p>

</body>
</html>
```

```bash
[root@k8s-master configmap-test]# curl  http://10.244.2.6
[root@k8s-master configmap-test]# curl  http://10.244.1.5
```

두 요청 모두 위와 동일하게 `<meta charset="UTF-8">`, "ConfigMap 수정 테스트", "Version 2"가 포함된 갱신된 index.html을 반환한다.

---

## Secret 개념

Secret은 비밀번호, 토큰, 인증키 등 민감한 데이터를 저장하기 위한 Kubernetes 오브젝트이다.

ConfigMap과 마찬가지로 Key-Value 형태로 데이터를 저장한다.

```
Secret
  │
  ├── USERNAME = admin
  ├── PASSWORD = 1234
  └── TOKEN    = abc123
```

## Secret을 사용하는 이유

애플리케이션에서 다음과 같은 값을 Deployment YAML이나 Docker Image에 직접 작성하면 보안상 위험하다.

```
DB_USERNAME=admin
DB_PASSWORD=1234
API_TOKEN=abc123
```

예를 들어 Deployment에 직접 작성하면:

```yaml
env:
  - name: DB_PASSWORD
    value: "1234"
```

설정값이 YAML에 그대로 노출된다.

Secret을 사용하면 민감한 값을 별도의 Kubernetes 오브젝트로 관리할 수 있다.

```
Secret  -->  Pod  -->  Container
```

## ConfigMap과 Secret의 차이

| 구분 | ConfigMap | Secret |
|---|---|---|
| 목적 | 일반 설정값 | 민감한 데이터 |
| 예 | 환경명, 로그 레벨 | 비밀번호, 토큰 |
| 오브젝트 | O | O |
| Key-Value | O | O |
| 환경변수 사용 | O | O |
| 파일 Mount | O | O |
| 기본 표현 | 일반 문자열 | Base64 |

- ConfigMap: 공개되어도 큰 문제가 없는 설정
- Secret: 보호해야 하는 설정

## Secret의 데이터 구조

Secret도 내부적으로 Key-Value 구조이다.

```
	Secret : app-secret
======================================
| Key       	| Value		|
======================================
| USERNAME	| admin      	|
--------------------------------------
| PASSWORD	| 1234      	|
--------------------------------------
| TOKEN 	| abc123   	|
======================================
```

## Secret의 종류

Secret에는 여러 Type이 있다.

- `Opaque`
- `kubernetes.io/tls`
- `kubernetes.io/dockerconfigjson`
- `kubernetes.io/basic-auth`

**Opaque**
- 일반적인 Key-Value 데이터를 저장할 때 사용
- `type: Opaque`
- 우리가 실습에서 사용하는 일반적인 Secret

**TLS Secret**
- TLS 인증서와 개인키를 저장할 때 사용한다.
- `tls.crt`
- `tls.key`

**Docker Registry Secret**
- Private Docker Registry 인증 정보를 저장할 때 사용
- `kubernetes.io/dockerconfigjson`

## Secret 생성 방법

**방법 1 (--from-literal)** — 직접 Key-Value를 입력하는 방식

```bash
kubectl create secret generic app-secret \
  --from-literal=USERNAME=admin \
  --from-literal=PASSWORD=1234
```

**방법 2 (--from-file)** — 파일의 내용을 Secret으로 저장하는 방식

```bash
kubectl create secret generic app-secret  --from-file=password.txt
```

- Key = `password.txt`
- Value = `password.txt` 파일 내용

**방법 3 (YAML)**

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secret

type: Opaque

data:
  USERNAME: YWRtaW4=	# Base64값 직접 입력
  PASSWORD: MTIzNA==	# Base64값 직접 입력
```

Secret의 data에 값을 직접 작성할 때는 일반적으로 Base64로 인코딩된 값을 사용한다.

```bash
echo  -n  "admin" | base64
YWRtaW4=
```

따라서:

```yaml
data:
  USERNAME: YWRtaW4=
```

로 작성한다. (Base64는 암호화가 아니라 인코딩 방식이다.)

## Secret 저장값 확인

- Secret 목록: `kubectl  get secret`
- Secret 정보: `kubectl  describe  secret  app-secret`
- YAML: `kubectl  get secret  app-secret  -o yaml`
- 특정 값 확인: `kubectl  get secret  app-secret  -o jsonpath='{.data.PASSWORD}' | base64 -d`

## Secret을 Pod에서 사용하는 방법

ConfigMap과 마찬가지로 크게 두 가지로 사용 가능하다.

**방법 1) 환경변수 (Key를 사용해서 Secret에 저장된 Value 적용)**

```
Secret  -->  Environment Variable  -->  Container
```

특정 Key만 가져오는 예:

```yaml
env:
  - name: PASSWORD
    valueFrom:
      secretKeyRef:
        name: app-secret
        key: PASSWORD
```

**방법 2) 환경변수 (Secret에 환경변수를 적용)**

```yaml
envFrom:
  - secretRef:
      name: app-secret
```

**방법 3) Secret을 파일로 Mount**

```
Secret -->  Volume -->  Container 내부 파일
```

Pod에서 사용할 Volume을 정의한다. app-secret Secret을 Pod에서 사용할 Volume으로 가져오고, 그 Volume의 이름을 secret-volume으로 정의한다.

```yaml
volumes:
  - name: secret-volume
    secret:
      secretName: app-secret
```

볼륨 적용:

```yaml
volumeMounts:
  - name: secret-volume
    mountPath: /etc/secret
```

- `/etc/secret/USERNAME` 경로 및 파일 생성
- `/etc/secret/PASSWORD` 경로 및 파일 생성

## Secret과 환경변수/Volume의 관계

**Secret의 Key와 파일의 관계**

예를 들어 Secret:

```
USERNAME = admin
PASSWORD = 1234
```

을 `/etc/secret`에 Mount하면:

```
/etc/secret
       ├── USERNAME
       └── PASSWORD
```

파일 내용:
- USERNAME 파일: key, `admin`: value
- PASSWORD 파일: key, `1234`: value

즉 Secret의 Key가 파일 이름이 되고 Value가 파일 내용이 된다.

**Secret과 환경변수의 관계**

```yaml
env:
  - name: DB_PASSWORD
    valueFrom:
      secretKeyRef:
        name: app-secret
        key: PASSWORD
```

Secret: `PASSWORD = 1234` → Container: `DB_PASSWORD=1234`

즉 Secret의 Key 이름과 Container 환경변수 이름은 반드시 같을 필요가 없다.

**env와 envFrom 차이**

`env` : Secret의 특정 Key만 가져온다.

```yaml
env:
  - name: DB_PASSWORD
    valueFrom:
      secretKeyRef:
        name: app-secret
        key: PASSWORD
```

`envFrom` : Secret의 모든 Key를 가져온다.

```yaml
envFrom:
  - secretRef:
      name: app-secret
```

**Secret과 Volume의 관계**

Secret 자체가 Volume은 아니다.

- Secret: 설정 데이터를 저장하는 Kubernetes 오브젝트
- Volume: Container에 저장공간이나 파일을 연결하는 방식

## Secret 실습 (Deployment)

**STEP 1) 실습 디렉터리 생성**

```bash
[root@k8s-master ~]# mkdir secret-practice
[root@k8s-master ~]# cd secret-practice
[root@k8s-master secret-practice]# pwd
/root/secret-practice
```

**STEP 2) Secret 생성**

다음 조건에 맞는 Secret을 생성한다.

- Secret 이름: `app-secret`
- `USERNAME`: `admin`
- `PASSWORD`: `1234`

```bash
[root@k8s-master secret-practice]# kubectl create secret generic app-secret \
  --from-literal=USERNAME=admin \
  --from-literal=PASSWORD=1234
secret/app-secret created
```

```bash
[root@k8s-master secret-practice]# kubectl  get  secrets
NAME                                    	TYPE                 	DATA	AGE
app-secret                              	Opaque               	2      	28s
sh.helm.release.v1.nfs-provisioner.v1	helm.sh/release.v1	1      	20h
```

```bash
[root@k8s-master secret-practice]# kubectl  get  secrets  app-secret
NAME         TYPE     DATA   AGE
app-secret   Opaque    2          73s
```

```
	Secret : app-secret
========================================================
| 	Key		| 	Value		|
========================================================
| USERNAME		| admin        		|
--------------------------------------------------------
| PASSWORD		| 1234         		|
========================================================
```

**STEP 3) Secret에 저장된 값 확인**

```bash
[root@k8s-master secret-practice]# kubectl  get  secrets  app-secret  -o  yaml
```

```yaml
apiVersion: v1
data:
  PASSWORD: MTIzNA==
  USERNAME: YWRtaW4=
kind: Secret
metadata:
  creationTimestamp: "2026-08-28T03:14:34Z"
  name: app-secret
  namespace: default
  resourceVersion: "582606"
  uid: 809838ba-4332-4c8c-bb3f-ba7e89110d4e
type: Opaque
```

Secret의 값이 Base64 형태로 표시된다.

- `USERNAME=admin` → `USERNAME=YWRtaW4=`
- `PASSWORD=1234` → `PASSWORD=MTIzNA==`

Base64는 암호화가 아니라 인코딩이므로 Base64 값만 알고 있어도 쉽게 원래 값을 확인할 수 있다.

Secret의 Base64 값을 원래 값으로 변환:

```bash
# text --> base64
[root@k8s-master secret-practice]# echo "admin" | base64
YWRtaW4K

# text <-- base64
[root@k8s-master secret-practice]# echo "YWRtaW4K" | base64 -d
admin

# text --> base64
[root@k8s-master secret-practice]# echo "1234" | base64
MTIzNAo=

# text <-- base64
[root@k8s-master secret-practice]# echo "MTIzNAo=" | base64 -d
1234
```

**STEP 4) Deployment YAML 작성**

다음 조건에 맞는 Deployment를 작성한다.

- Deployment 이름: `secret-deployment`
- Pod 2개
- Container 이름: `nginx`
- Image: `nginx:1.31`
- Secret `app-secret` 사용
- Secret의 모든 Key-Value를 환경변수로 사용

```bash
[root@k8s-master secret-practice]# vi secret-deployment.yaml
```

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: secret-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: secret

  template:
    metadata:
      labels:
        app: secret
    spec:
      containers:
        - name: nginx
          image: nginx:1.31
          envFrom:
            - secretRef:
                name: app-secret
```

```yaml
envFrom:
  - secretRef:
      name: app-secret
```

app-secret Secret의 모든 Key-Value를 Container의 환경변수로 전달한다.

```
Secret: app-secret
USERNAME=admin
PASSWORD=1234
          ↓
      envFrom
          ↓
Container
USERNAME=admin
PASSWORD=1234
```

**STEP 5) Deployment 생성 및 확인**

작성한 Deployment YAML을 Kubernetes에 적용하고 Pod가 정상적으로 생성되었는지 확인한다.

```bash
[root@k8s-master secret-practice]# kubectl apply -f secret-deployment.yaml
deployment.apps/secret-deployment created
```

```bash
[root@k8s-master secret-practice]# kubectl  get  deployments  secret-deployment
NAME                READY   UP-TO-DATE   AVAILABLE   AGE
secret-deployment   2/2     2            2           18s
```

```bash
[root@k8s-master secret-practice]# kubectl  get  pods
NAME                                         READY   STATUS    RESTARTS   AGE
secret-deployment-68f778b75-6ddhh   1/1       Running     0                 4s
secret-deployment-68f778b75-qhhd7   1/1       Running     0                 4s
```

**Secret 환경변수 확인**

Pod에 Secret의 USERNAME과 PASSWORD가 환경변수로 전달되었는지 확인한다.

```bash
# 첫 번째 Pod에서 확인
[root@k8s-master secret-practice]# kubectl  exec  secret-deployment-68f778b75-6ddhh  --  /bin/bash -c 'echo $USERNAME'
admin

# 첫 번째 Pod에서 확인
[root@k8s-master secret-practice]# kubectl  exec  secret-deployment-68f778b75-6ddhh  --  /bin/bash -c 'echo $PASSWORD'
1234
```

두 번째 Pod에서도 동일한 방식으로 확인 가능하다.

## Secret + MySQL 실습

```
Secret  -->  Deployment  -->  Pod  -->  MySQL
```

Secret에 저장된 DB 접속 정보를 MySQL Container에 환경변수로 전달하여 실제 DB가 생성되는 것을 확인한다.

**STEP 1) 실습 디렉터리 생성**

Secret + MySQL 실습을 위한 디렉터리를 생성하고 이동한다.

```bash
[root@k8s-master secret-practice]# cd
[root@k8s-master ~]# mkdir secret-mysql-practice
[root@k8s-master ~]# cd secret-mysql-practice
[root@k8s-master secret-mysql-practice]# pwd
/root/secret-mysql-practice
```

**STEP 2) MySQL Secret 생성**

MySQL에서 사용할 민감한 정보를 Secret으로 생성한다.

- Secret 이름: `mysql-secret`
- `MYSQL_ROOT_PASSWORD`: `root1234`
- `MYSQL_DATABASE`: `testdb`
- `MYSQL_USER`: `appuser`
- `MYSQL_PASSWORD`: `app1234`

```bash
[root@k8s-master secret-mysql-practice]# kubectl create secret generic mysql-secret \
  --from-literal=MYSQL_ROOT_PASSWORD=root1234 \
  --from-literal=MYSQL_DATABASE=testdb \
  --from-literal=MYSQL_USER=appuser \
  --from-literal=MYSQL_PASSWORD=app1234
```

```
	Secret : mysql-secret
========================================================
| 		Key      	| 	Value         	|
========================================================
| MYSQL_ROOT_PASSWORD	| root1234      		|
--------------------------------------------------------
| MYSQL_DATABASE     	| testdb        		|
--------------------------------------------------------
| MYSQL_USER         	| appuser       		|
--------------------------------------------------------
| MYSQL_PASSWORD    	| app1234       		|
========================================================
```

MYSQL_ROOT_PASSWORD, MYSQL_PASSWORD는 민감한 정보이므로 Secret으로 관리한다.

```bash
[root@k8s-master configmap-test]# kubectl  get  secrets
NAME                                    	TYPE                 	DATA   	AGE
app-secret                              	Opaque               	2      	35m
mysql-secret                            	Opaque               	4      	4s
sh.helm.release.v1.nfs-provisioner.v1	helm.sh/release.v1	1      	20h
```

```bash
[root@k8s-master configmap-test]# kubectl  get  secrets  mysql-secret
NAME           TYPE     DATA   AGE
mysql-secret   Opaque   4         11s
```

```bash
[root@k8s-master configmap-test]# kubectl  describe  secrets  mysql-secret
Name:     	mysql-secret
Namespace: 	default
Labels:     	<none>
Annotations:	<none>

Type:  Opaque

Data
====
MYSQL_DATABASE:	6 bytes
MYSQL_PASSWORD:       	7 bytes
MYSQL_ROOT_PASSWORD:	8 bytes
MYSQL_USER:           	7 bytes
```

```bash
[root@k8s-master secret-mysql-practice]# kubectl get secret mysql-secret -o yaml
```

```yaml
apiVersion: v1
data:
  MYSQL_DATABASE: dGVzdGRi
  MYSQL_PASSWORD: YXBwMTIzNA==
  MYSQL_ROOT_PASSWORD: cm9vdDEyMzQ=
  MYSQL_USER: YXBwdXNlcg==
kind: Secret
metadata:
  creationTimestamp: "2026-08-28T03:50:13Z"
  name: mysql-secret
  namespace: default
  resourceVersion: "586138"
  uid: 40bfc666-96c8-413e-aff4-cb8aaaf04f8c
type: Opaque
```

**STEP 3) MySQL Deployment YAML 작성**

다음 조건에 맞는 MySQL Deployment를 작성한다.

- Deployment 이름: `mysql-deployment` (Pod 1개 생성)
- Container 이름: `mysql`
- Image: `mysql:8.0`
- Secret `mysql-secret` 사용
- Secret의 모든 Key-Value를 환경변수로 사용

```bash
[root@k8s-master secret-mysql-practice]# vi mysql-deployment.yaml
```

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mysql

  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
        - name: mysql
          image: mysql:8.0
          envFrom:
            - secretRef:
                name: mysql-secret
```

envFrom을 사용하여 mysql-secret의 모든 Key-Value를 MySQL Container의 환경변수로 전달한다.

```
	Secret : mysql-secret
========================================================
| 		Key      	| 	Value         	|
========================================================
| MYSQL_ROOT_PASSWORD	| root1234      		|
--------------------------------------------------------
| MYSQL_DATABASE     	| testdb        		|
--------------------------------------------------------
| MYSQL_USER         	| appuser       		|
--------------------------------------------------------
| MYSQL_PASSWORD    	| app1234       		|
========================================================
```

```
mysql-secret
 MYSQL_ROOT_PASSWORD
 MYSQL_DATABASE
 MYSQL_USER
 MYSQL_PASSWORD
          ↓
     envFrom
          ↓
MySQL Container
 MYSQL_ROOT_PASSWORD=root1234
 MYSQL_DATABASE=testdb
 MYSQL_USER=appuser
 MYSQL_PASSWORD=app1234
```

**STEP 5) Deployment 생성**

작성한 MySQL Deployment를 Kubernetes에 적용한다.

```bash
[root@k8s-master secret-mysql-practice]# kubectl apply -f mysql-deployment.yaml
deployment.apps/mysql-deployment created
```

```bash
[root@k8s-master secret-mysql-practice]# kubectl get deployment
NAME           	READY   UP-TO-DATE   AVAILABLE   AGE
mysql-deployment	0/1        1                     0                 9s
```

```bash
[root@k8s-master secret-mysql-practice]#  kubectl get pods
NAME                                	READY   STATUS    RESTARTS   AGE
mysql-deployment-595b4d775c-97w6l   	1/1         Running     0                26s
```

Deployment가 MySQL Pod 1개를 생성한다.

**STEP 6) Pod에 Secret 환경변수가 전달되었는지 확인**

```bash
[root@k8s-master secret-mysql-practice]#  kubectl get pods
NAME                                	READY   STATUS    RESTARTS   AGE
mysql-deployment-595b4d775c-97w6l   	1/1         Running     0                26s
```

```bash
[root@k8s-master secret-mysql-practice]# kubectl  exec  mysql-deployment-595b4d775c-97w6l  --  env | grep MYSQL
MYSQL_MAJOR=8.0
MYSQL_VERSION=8.0.46-1.el9
MYSQL_SHELL_VERSION=8.0.46-1.el9
MYSQL_PASSWORD=app1234
MYSQL_ROOT_PASSWORD=root1234
MYSQL_USER=appuser
MYSQL_DATABASE=testdb
```

Secret에 저장된 값이 MySQL Container의 환경변수로 전달된 것을 확인한다.

```
Secret : mysql-secret
 MYSQL_ROOT_PASSWORD
 MYSQL_DATABASE
 MYSQL_USER
 MYSQL_PASSWORD
        ↓
    envFrom
        ↓
MySQL Container
 MYSQL_ROOT_PASSWORD
 MYSQL_DATABASE
 MYSQL_USER
 MYSQL_PASSWORD
```

Secret 자체가 MySQL에 직접 연결되는 것이 아니라 Secret의 값을 환경변수로 전달하고, MySQL이 그 환경변수를 사용하는 구조이다.

**STEP 8) MySQL에 접속**

MySQL Container에 접속하여 실제로 testdb 데이터베이스가 생성되었는지 확인한다.

```bash
# 컨테이너 접속
[root@k8s-master secret-mysql-practice]# kubectl  exec  -it  mysql-deployment-595b4d775c-97w6l  --  /bin/bash
bash-5.1#

# MySQL 로그인
bash-5.1# mysql  -u  appuser  -p
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 9
Server version: 8.0.46 MySQL Community Server - GPL

Copyright (c) 2000, 2026, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>
```

```sql
mysql> SHOW DATABASES;
+---------------------------+
| Database           		|
+---------------------------+
| information_schema 	|
| performance_schema 	|
| testdb             	   	|
+---------------------------+
3 rows in set (0.01 sec)
```

```sql
mysql> SELECT USER();
+-------------------+
| USER()            	  |
+-------------------+
| appuser@localhost	  |
+-------------------+
1 row in set (0.00 sec)
```

```sql
mysql> USE testdb;
Database changed
```

```sql
-- 테이블 생성
mysql> CREATE TABLE member (
    id INT PRIMARY KEY,
    name VARCHAR(50)
);
```

```sql
mysql> SHOW TABLES;
+------------------+
| Tables_in_testdb	|
+------------------+
| member           	|
+------------------+
1 row in set (0.00 sec)
```

```sql
mysql> INSERT INTO  member  VALUES(1, 'Kubernetes');
Query OK, 1 row affected (0.01 sec)
```

```sql
mysql> SELECT * FROM  member;
+----+------------+
| id | name       	|
+----+------------+
|  1 | Kubernetes 	|
+----+------------+
1 row in set (0.00 sec)
```

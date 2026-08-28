# Storage

> Pod와 데이터를 분리해야 하는 이유, Storage 3단계 구조, Volume/PV/PVC/StorageClass 핵심 개념, emptyDir·hostPath·NFS 기반 PV/PVC 실습, StorageClass를 이용한 Dynamic Provisioning까지 정리한다.

## Pod와 데이터 분리

Pod는 원래 언제든지 사라질 수 있는 존재다.

- Deployment는 Pod를 필요하면 새로 만들고, 문제가 있으면 재생성한다.
- 노드를 drain 하면 Pod가 다른 노드로 이동(재생성)한다.
- 스케줄러는 Pod를 다른 노드에 새로 올릴 수 있다.

컨테이너 내부 저장은 기본적으로 휘발성이다. 컨테이너 안에 파일을 저장해도 Pod가 삭제되면 같이 사라진다. 그래서 DB, 업로드 파일, 로그 같은 데이터는 컨테이너 내부에만 두면 안 된다.

Pod(애플리케이션)와 데이터(저장소)는 반드시 분리해야 한다.

- Pod는 언제든지 삭제, 재생성, 이동될 수 있는 존재다.
- 드레인, 장애 복구 과정에서 Pod는 쉽게 바뀐다.
- 따라서 Pod 안에 데이터를 두면 Pod가 사라질 때 데이터도 함께 사라진다.

Pod는 갈아끼우고, 데이터는 그대로 유지하는 것이 목표다.

- Pod는 소모품처럼 교체된다.
- 데이터는 PersistentVolume(PV), PersistentVolumeClaim(PVC)로 외부에 유지된다.
- Pod가 바뀌어도 동일한 PVC를 다시 마운트하면 기존 데이터를 그대로 사용한다.

## Storage 전체 구조

쿠버네티스 스토리지는 3단계로 나눠서 이해한다.

**1) Pod 내부 관점 — 어디 경로에 마운트해서 쓰나?**
- `volumeMounts`: 컨테이너 경로(`/data` 등)에 붙여야 한다.

**2) 쿠버네티스 리소스 관점 — 저장소를 어떻게 요청/할당하나?**
- PV: 실제 저장공간(리소스)
- PVC: Pod가 요청하는 저장공간(주문서)
- StorageClass: 동적 생성 규칙(자동으로 PV 만들어주는 규칙)

**3) 실제 인프라 관점 — 진짜 디스크가 어디 있나**
- NFS, iSCSI, Ceph, GlusterFS, 클라우드 디스크(예: AWS EBS), 로컬 디스크 등

## 핵심 개념 4가지

**Volume**
- Pod가 사용하는 저장공간 인터페이스
- Pod YAML에 `volumes`와 `volumeMounts`로 정의한다
- Pod에 붙는 개념이라 Pod가 없어지면 연결도 끊긴다(데이터는 종류에 따라 유지/삭제)

**PV (PersistentVolume)**
- 클러스터에 존재하는 실제 저장공간
- Pod와 분리되어 존재한다
- 예: NFS 서버의 특정 경로, Ceph 볼륨, AWS EBS 디스크, Google Persistent Disk, Azure Disk 등

**PVC (PersistentVolumeClaim)**
- Pod가 PV를 요청하는 주문서
- 필요한 용량(예: 10Gi), 접근 방식(예: ReadWriteOnce)을 적어서 요청한다.
- 쿠버네티스는 조건이 맞는 PV를 찾아 PVC와 연결(binding)한다

**StorageClass**
- PVC가 만들어질 때 PV를 자동으로 생성해주는 규칙
- 이게 있으면 관리자가 PV를 미리 만들어두지 않아도 된다(동적 프로비저닝)

## Volume 종류

**1) emptyDir**
- 특징: Pod가 살아있는 동안만 존재하는 임시 공간
- 사용처: 임시 캐시, 컨테이너 간 임시 파일 공유
- Pod가 삭제되면 데이터도 삭제된다.

**2) hostPath**
- 특징: 노드의 특정 디렉터리를 Pod에 마운트
- 장점: 쉬움, 로컬 파일 접근 가능
- 단점
  - Pod가 다른 노드로 이동하면 데이터가 안 보일 수 있다
  - 운영환경에서는 보안/이식성 문제가 커서 제한적으로만 사용

**3) configMap / secret (설정/키 주입)**
- 파일로 설정값 또는 비밀값을 컨테이너에 제공하는 용도
- DB 데이터 저장용이 아니다
- secret은 base64로 저장되며, 민감정보 관리에 사용한다.

**4) PV/PVC 기반 영구 저장소**
- Pod가 없어져도 데이터 유지
- DB, 업로드 파일, 중요 로그 저장에 사용

## emptyDir 실습 (2개 Pod 비교)

emptyDir 임시 저장소를 마운트한다. nginx가 사용하는 웹 루트 디렉터리를 Pod 전용 임시 공간으로 바꿔서 쓰는 구조이며, nginx는 컨테이너 이미지에 들어 있던 기본 html이 아니라 emptyDir 볼륨에 있는 파일을 읽게 된다.

```yaml
# storage_test.yaml
apiVersion: v1
kind: Pod
metadata:
  name: web-empt
spec:
  containers:
  - name: nginx-empt
    image: nginx:1.31

    volumeMounts:
    - name: html
      mountPath: /usr/share/nginx/html

  volumes:
  - name: html
    emptyDir: {}

---
apiVersion: v1
kind: Pod
metadata:
  name: web-noempt
spec:
  containers:
  - name: nginx-container
    image: nginx:1.31
    ports:
    - containerPort: 80
      protocol: TCP
```

```bash
[root@k8s-master ~]# kubectl  apply  -f  storage_test.yaml
pod/web-empt created
pod/web-noempt created

[root@k8s-master ~]# kubectl  get  pods
NAME         	READY   STATUS    RESTARTS   AGE
web-empt     	1/1     Running   0          2m10s
web-noempt	    1/1     Running   0          2m10s

[root@k8s-master ~]# kubectl  get  pods  -o  wide
NAME         	READY   STATUS    RESTARTS   AGE     IP            NODE          NOMINATED NODE   READINESS GATES
web-empt     	1/1     Running   0          2m33s   10.244.1.6   k8s-worker1   <none>            <none>
web-noempt	    1/1     Running   0          2m33s   10.244.1.7   k8s-worker1   <none>            <none>

   # web-noemp
[root@k8s-master ~]# curl  http://10.244.1.7
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

   # web-emp
[root@k8s-master ~]# curl  http://10.244.1.6
<html>
<head><title>403 Forbidden</title></head>
<body>
<center><h1>403 Forbidden</h1></center>
<hr><center>nginx/1.31.3</center>
</body>
</html>
```

임시 디렉터리에 아무런 파일이 없기 때문에 결과가 출력되지 않는다.

```bash
   # web-noempt 컨테이너 접속
[root@k8s-master ~]# kubectl  exec  web-noempt  -it  -- /bin/bash
root@web-noempt:/#
root@web-noempt:/# ls  -l  /usr/share/nginx/html/
total 8
-rw-r--r-- 1 root root 497 Jul 15 16:03 50x.html
-rw-r--r-- 1 root root 896 Jul 15 16:03 index.html		<---

   # web-empt 컨테이너 접속
[root@k8s-master ~]# kubectl  exec  web-empt  -it  -- /bin/bash
root@web-empt:/#
root@web-empt:/# ls  -l  /usr/share/nginx/html/
total 0						# index.html 파일이 확인되지 않는다.

root@web-empt:/# mount | grep  /usr
/dev/mapper/rl-root on /usr/share/nginx/html type xfs (rw,relatime,attr2,inode64,logbufs=8,logbsize=32k,noquota)

root@web-empt:/# echo "<h1> Storage EmptDir Test </h1>"  > /usr/share/nginx/html/index.html

[root@k8s-master ~]# curl  http://10.244.1.6
<h1> Storage EmptDir Test </h1>

[root@k8s-master ~]# kubectl  delete pods web-empt
pod "web-empt" deleted from default namespace

[root@k8s-master ~]# kubectl  delete pods web-noempt
pod "web-noempt" deleted from default namespace
```

## emptyDir 실습 (컨테이너 2개 간 공유)

Pod 1개 안에서 nginx 2개를 생성한 후 emptyDir로 index.html을 공유하는 실습이다.

```yaml
# pod-emptydir-nginx2.yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-emptydir-nginx2
spec:
  containers:
  - name: nginx-a
    image: nginx:1.31
    ports:
    - containerPort: 80
    volumeMounts:
    - name: html
      mountPath: /usr/share/nginx/html

  - name: nginx-b
    image: nginx:1.31
    ports:
    - containerPort: 8080
    volumeMounts:
    - name: html
      mountPath: /usr/share/nginx/html

    command: ["/bin/sh", "-c"]
    args:
    - |
      cat > /etc/nginx/conf.d/default.conf <<'CONF'
      server {
        listen 8080;
        server_name _;
        root /usr/share/nginx/html;
        index index.html;
        location / {
          try_files $uri $uri/ =404;
        }
      }
      CONF
      nginx -g 'daemon off;'

  volumes:
  - name: html
    emptyDir: {}
```

```bash
[root@k8s-master ~]# kubectl  apply  -f  pod-emptydir-nginx2.yaml
pod/pod-emptydir-nginx2 created

[root@k8s-master ~]# kubectl  get  pods
NAME                  		READY  STATUS    RESTARTS   AGE
pod-emptydir-nginx2	    2/2    Running   0          61s

[root@k8s-master ~]# kubectl  get  pods  -o  wide
NAME                  		READY   STATUS    RESTARTS   AGE    IP             NODE          NOMINATED NODE   READINESS GATES
pod-emptydir-nginx2   	2/2     Running   0          100s   10.244.1.8   k8s-worker1   <none>            <none>

# pod-emptydir-nginx2 Pod안의 nginx-a 컨테이너로 통신
[root@k8s-master ~]# curl  http://10.244.1.8:80
<html>
<head><title>403 Forbidden</title></head>
<body>
<center><h1>403 Forbidden</h1></center>
<hr><center>nginx/1.31.3</center>
</body>
</html>

# pod-emptydir-nginx2 Pod안의 nginx-b 컨테이너로 통신
[root@k8s-master ~]# curl  http://10.244.1.8:8080
<html>
<head><title>403 Forbidden</title></head>
<body>
<center><h1>403 Forbidden</h1></center>
<hr><center>nginx/1.31.3</center>
</body>
</html>

   # pod-emptydir-nginx2 Pod안의 nginx-a 컨테이너로 접속
[root@k8s-master ~]# kubectl  exec  -it  pod-emptydir-nginx2 -c nginx-a  --  /bin/bash
root@pod-emptydir-nginx2:/#

   # html 디렉터리에 index.html 파일이 없다.
root@pod-emptydir-nginx2:/# ls  -l  /usr/share/nginx/html/
total 0

   # /usr/share/nginx/html/ 경로에 index.html 파일 생성
root@pod-emptydir-nginx2:/# echo  "<h1> emptDir volumeMount Test NGINX html </h1>"  >  /usr/share/nginx/html/index.html

root@pod-emptydir-nginx2:/# cat  /usr/share/nginx/html/index.html
<h1> emptDir volumeMount Test NGINX html </h1>

   # pod-emptydir-nginx2 Pod안의 nginx-b 컨테이너로 접속
[root@k8s-master ~]# kubectl  exec  -it  pod-emptydir-nginx2 -c nginx-b  --  /bin/bash
root@pod-emptydir-nginx2:/#

root@pod-emptydir-nginx2:/# cat  /usr/share/nginx/html/index.html
<h1> emptDir volumeMount Test NGINX html </h1>

[root@k8s-master ~]# curl  http://10.244.1.8:80
<h1> emptDir volumeMount Test NGINX html </h1>

[root@k8s-master ~]# curl  http://10.244.1.8:8080
<h1> emptDir volumeMount Test NGINX html </h1>

   # 실습 완료 후 Pod 삭제
[root@k8s-master ~]# kubectl  delete  pods  pod-emptydir-nginx2
pod "pod-emptydir-nginx2" deleted from default namespace
```

## hostPath 실습

hostPath는 Pod가 실행되는 노드의 특정 디렉터리를 컨테이너 안에 그대로 마운트해서 사용하는 방식이다. Pod가 다른 노드로 이동할 수 있으므로, 모든 노드에 동일한 디렉터리가 존재해야 데이터 불일치 위험을 줄일 수 있다. 특정 Node의 데이터를 사용해야 한다면 nodeSelector나 nodeAffinity를 이용하여 Pod가 특정 Node에서 실행되도록 제한할 수 있다. 이 구조는 노드 종속적이어서 운영 환경에서는 권장되지 않는다.

```bash
[root@k8s-master ~]# ssh  guest@k8s-worker1
guest@k8s-worker1's password:
Activate the web console with: systemctl enable --now cockpit.socket

Last login: Tue Aug 18 11:06:41 2026 from 192.168.10.100

[guest@k8s-worker1 ~]$ sudo  mkdir  /webdata

[root@k8s-worker1 ~]# sudo  vi  /webdata/index.html
```

```html
<!doctype html>
<html lang="ko">
<head>
  <meta charset="utf-8" />
  <title>스토리지 테스트 페이지</title>
</head>
<body>
  <h1>Hello, Kubernetes!</h1>
  <p>스토리지 테스트 페이지입니다. 워커노드1</p>
</body>
</html>
```

```bash
[root@k8s-master ~]# ssh  guest@k8s-worker2
guest@k8s-worker1's password:
Activate the web console with: systemctl enable --now cockpit.socket

Last login: Tue Aug 18 11:06:41 2026 from 192.168.10.100

[guest@k8s-worker2 ~]$ sudo  mkdir  /webdata

[root@k8s-worker2 ~]# sudo  vi  /webdata/index.html
```

```html
<!doctype html>
<html lang="ko">
<head>
  <meta charset="utf-8" />
  <title>스토리지 테스트 페이지</title>
</head>
<body>
  <h1>Hello, Kubernetes!</h1>
  <p>스토리지 테스트 페이지입니다. 워커노드2</p>
</body>
</html>
```

```yaml
# storage_hostpath.yaml
apiVersion: v1
kind: Pod
metadata:
  name: web-hostpath
spec:
  containers:
  - name: nginx
    image: nginx:1.29.1

    volumeMounts:
    - name: html
      mountPath: /usr/share/nginx/html

  volumes:
  - name: html
    hostPath:
      path: /webdata
```

```bash
[root@k8s-master ~]# kubectl  apply  -f  storage_hostpath.yaml
pod/web-hostpath created

[root@k8s-master ~]# kubectl  get  pods  -o  wide
NAME           	READY   STATUS    RESTARTS   AGE   IP            NODE          NOMINATED NODE   READINESS GATES
web-hostpath	1/1     Running   0          22s   10.244.1.9   k8s-worker1   <none>            <none>

[root@k8s-master ~]# curl  http://10.244.1.9
<!doctype html>
<html lang="ko">
<head>
  <meta charset="utf-8" />
  <title>스토리지 테스트 페이지</title>
</head>
<body>
  <h1>Hello, Kubernetes!</h1>
  <p>스토리지 테스트 페이지입니다. 워커노드1</p>
</body>
</html>

[root@k8s-master ~]# kubectl  exec  -it  web-hostpath  --  /bin/bash
root@web-hostpath:/#

root@web-hostpath:/# apt-get  update

root@web-hostpath:/# apt-get  install  -y  vim

root@web-hostpath:/# export LANG=C.UTF-8

root@web-hostpath:/# vim  /usr/share/nginx/html/index.html
```

```html
<!doctype html>
<html lang="ko">
<head>
  <meta charset="utf-8" />
  <title>Storage Test Page</title>
</head>
<body>
  <h1>Hello, Kubernetes!</h1>
  <p>Storage Test Page k8s-worker1</p>
</body>
</html>
```

```bash
[root@k8s-master ~]# ssh  guest@k8s-worker1
guest@k8s-worker1's password:
Activate the web console with: systemctl enable --now cockpit.socket

Last login: Thu Aug 27 11:00:21 2026 from 192.168.10.100

[guest@k8s-worker1 ~]$ cat /webdata/index.html
<!doctype html>
<html lang="ko">
<head>
  <meta charset="utf-8" />
  <title>Storage Test Page</title>
</head>
<body>
  <h1>Hello, Kubernetes!</h1>
  <p>Storage Test Page k8s-worker1</p>
</body>
</html>
```

nginx 컨테이너 안에서 vim으로 직접 수정한 index.html이 워커노드의 실제 `/webdata/index.html`에도 그대로 반영된 것을 확인할 수 있다 — hostPath는 컨테이너 경로와 노드 경로가 같은 파일을 가리킨다.

## nodeSelector + hostPath 실습

hostPath는 Pod가 실행되는 Node의 로컬 디렉터리를 사용한다. 따라서 특정 Node의 디렉터리를 사용하려면 nodeSelector로 Pod를 그 Node에 고정할 수 있다. 아래 실습은 k8s-worker1의 `/webdata`를 Pod의 `/usr/share/nginx/html`에 마운트한다.

**STEP 1) worker1에 Label 추가**

```bash
# k8s-worker1에 식별용 Label을 추가
[root@k8s-master ~]# kubectl label node k8s-worker1 storage=hostpath

[root@k8s-master ~]# kubectl get nodes -L storage
NAME           STATUS  	ROLES           AGE   VERSION   STORAGE
k8s-master     Ready	control-plane   15d   v1.35.7
k8s-worker1   Ready	<none>          15d   v1.35.7   hostpath
k8s-worker2   Ready	<none>          15d   v1.35.7
```

**STEP 2) worker1에 hostPath 디렉터리 생성 (위에서 생성한 디렉터리 재사용)**

```bash
[root@k8s-master ~]# ssh  guest@k8s-worker1
guest@k8s-worker1's password:
Activate the web console with: systemctl enable --now cockpit.socket

Last login: Tue Aug 18 11:06:41 2026 from 192.168.10.100

[guest@k8s-worker1 ~]$ sudo  mkdir  /webdata

[root@k8s-worker1 ~]# sudo  vi  /webdata/index.html
```

```html
<!doctype html>
<html lang="ko">
<head>
  <meta charset="utf-8" />
  <title>스토리지 테스트 페이지</title>
</head>
<body>
  <h1>Hello, Kubernetes!</h1>
  <p>스토리지 테스트 페이지입니다. 워커노드1</p>
</body>
</html>
```

**STEP 3) nodeSelector + hostPath Deployment 생성**

```yaml
# hostpath-nodeselector.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-hostpath-deploy

spec:
  replicas: 3
  selector:
    matchLabels:
      app: web-hostpath

  template:
    metadata:
      labels:
        app: web-hostpath
    spec:
      nodeSelector:
        storage: hostpath
      containers:
      - name: nginx
        image: nginx:1.31
        ports:
        - containerPort: 80

        volumeMounts:
        - name: html
          mountPath: /usr/share/nginx/html

      volumes:
      - name: html
        hostPath:
          path: /webdata
          type: Directory
```

```bash
[root@k8s-master ~]# kubectl apply -f hostpath-nodeselector.yaml
deployment.apps/web-hostpath-deploy created

[root@k8s-master ~]# kubectl  get  pods  -o  wide
NAME                                   	  READY   STATUS    RESTARTS   AGE   IP               NODE           NOMINATED NODE   READINESS GATES
web-hostpath-deploy-67957494b6-66gh9	  1/1     Running   0          44s   10.244.1.11   k8s-worker1   <none>            <none>
web-hostpath-deploy-67957494b6-fwxtl	  1/1     Running   0          44s   10.244.1.13   k8s-worker1   <none>            <none>
web-hostpath-deploy-67957494b6-lx26j	  1/1     Running   0          44s   10.244.1.12   k8s-worker1   <none>            <none>
```

**STEP 4) Pod 내부에서 확인**

```bash
   # 컨테이너 접속
[root@k8s-master ~]# kubectl exec  -it  web-hostpath-deploy-67957494b6-66gh9  --  /bin/bash
root@web-hostpath-deploy-67957494b6-66gh9:/#
root@web-hostpath-deploy-67957494b6-66gh9:/# cat /usr/share/nginx/html/index.html
<!doctype html>
<html lang="ko">
<head>
  <meta charset="utf-8" />
  <title>Storage Test Page</title>
</head>
<body>
  <h1>Hello, Kubernetes!</h1>
  <p>Storage Test Page k8s-worker1</p>
</body>
</html>

   # 컨테이너 접속
[root@k8s-master ~]# kubectl exec  -it  web-hostpath-deploy-67957494b6-fwxtl  --  /bin/bash
root@web-hostpath-deploy-67957494b6-fwxtl:/#
root@web-hostpath-deploy-67957494b6-fwxtl:/# cat /usr/share/nginx/html/index.html
<!doctype html>
<html lang="ko">
<head>
  <meta charset="utf-8" />
  <title>Storage Test Page</title>
</head>
<body>
  <h1>Hello, Kubernetes!</h1>
  <p>Storage Test Page k8s-worker1</p>
</body>
</html>

[root@k8s-master ~]# kubectl  get  pods  -o  wide
NAME                                   	  READY   STATUS    RESTARTS   AGE   IP               NODE           NOMINATED NODE   READINESS GATES
web-hostpath-deploy-67957494b6-66gh9	  1/1     Running   0          44s   10.244.1.11   k8s-worker1   <none>            <none>
web-hostpath-deploy-67957494b6-fwxtl	  1/1     Running   0          44s   10.244.1.13   k8s-worker1   <none>            <none>
web-hostpath-deploy-67957494b6-lx26j	  1/1     Running   0          44s   10.244.1.12   k8s-worker1   <none>            <none>

[root@k8s-master ~]# curl  http://10.244.1.11
<!doctype html>
<html lang="ko">
<head>
  <meta charset="utf-8" />
  <title>Storage Test Page</title>
</head>
<body>
  <h1>Hello, Kubernetes!</h1>
  <p>Storage Test Page k8s-worker1</p>
</body>
</html>

[root@k8s-master ~]# curl  http://10.244.1.12
[root@k8s-master ~]# curl  http://10.244.1.13

[root@k8s-master ~]# kubectl   delete  deployments  web-hostpath-deploy
deployment.apps "web-hostpath-deploy" deleted from default namespace
```

**정리**: nodeSelector로 3개의 Replica가 모두 `storage=hostpath` Label을 가진 k8s-worker1에만 배치되어 동일한 hostPath 디렉터리(`/webdata`)를 공유하는 것을 확인할 수 있다.

## hostPath type 필드 옵션

**DirectoryOrCreate**
- 지정한 경로에 디렉터리가 있으면 그대로 사용한다.
- 디렉터리가 없으면 kubelet이 자동으로 빈 디렉터리를 생성한다.
- 생성되는 디렉터리의 기본 권한은 일반적으로 0755이다.

```yaml
hostPath:
  path: /webdata
  type: DirectoryOrCreate
```

**Directory**
- 지정한 경로에 디렉터리가 반드시 존재해야 한다.
- 디렉터리가 없으면 Pod 생성에 실패한다.
- 디렉터리를 자동 생성하지 않는다.

```yaml
hostPath:
  path: /webdata
  type: Directory
```

**FileOrCreate**
- 지정한 경로에 파일이 있으면 그대로 사용한다.
- 파일이 없으면 kubelet이 빈 파일을 자동 생성한다.
- 단, 상위 디렉터리는 자동 생성하지 않는다.
- 생성되는 파일의 기본 권한은 일반적으로 0644이다.

```yaml
hostPath:
  path: /webdata/index.html
  type: FileOrCreate
```

**File**
- 지정한 경로에 파일이 반드시 존재해야 한다.
- 파일이 없으면 Pod 생성에 실패한다.
- 파일을 자동 생성하지 않는다.

```yaml
hostPath:
  path: /webdata/index.html
  type: File
```

### DirectoryOrCreate 실습

```yaml
# storage_directoryorcreate.yaml
apiVersion: v1
kind: Pod
metadata:
  name: web-directory
spec:
  containers:
  - image: nginx:1.31
    name: nginx

    volumeMounts:
    - name: html
      mountPath: /usr/share/nginx/html

  volumes:
  - name: html
    hostPath:
      path: /webdata2
      type: DirectoryOrCreate
```

```bash
[root@k8s-master ~]# kubectl  apply  -f  storage_directoryorcreate.yaml
pod/web-directory created

[root@k8s-master ~]# kubectl  get  pods  -o  wide
NAME            	READY   STATUS    RESTARTS   AGE   IP            NODE          NOMINATED NODE   READINESS GATES
web-directory	1/1     Running   0          25s   10.244.2.3   k8s-worker2   <none>            <none>

# 디렉터리는 만들어졌지만 index.html이 없기 때문에 403에러
[root@k8s-master ~]# curl http://10.244.2.3
<html>
<head><title>403 Forbidden</title></head>
<body>
<center><h1>403 Forbidden</h1></center>
<hr><center>nginx/1.31.3</center>
</body>
</html>

[root@k8s-master ~]# ssh  guest@k8s-worker2
guest@k8s-worker2's password:
Activate the web console with: systemctl enable --now cockpit.socket

Last login: Thu Aug 27 11:02:53 2026 from 192.168.10.100
[guest@k8s-worker2 ~]$

[guest@k8s-worker2 ~]$ ls  -ld  /webdata*
drwxr-xr-x 2 root root 24  8월 27 11:03 /webdata
drwxr-xr-x 2 root root  6  8월 27 12:06 /webdata2

   # /webdata/index.html 파일을   /webdata2로 복사
[guest@k8s-worker2 ~]$ sudo  cp  /webdata/index.html   /webdata2/index.html
[guest@k8s-worker2 ~]$ exit

[root@k8s-master ~]# curl http://10.244.2.3
<!doctype html>
<html lang="ko">
<head>
  <meta charset="utf-8" />
  <title>스토리지 테스트 페이지</title>
</head>
<body>
  <h1>Hello, Kubernetes!</h1>
  <p>스토리지 테스트 페이지입니다. 워커노드2</p>
</body>
</html>
```

DirectoryOrCreate 타입이라 `/webdata2` 디렉터리가 없었음에도 kubelet이 자동으로 생성했고, 그 안에 index.html을 넣어주자 정상적으로 페이지가 표시된다.

## PV / PVC 개념

Pod는 삭제/재생성/이동이 자주 발생하고, 컨테이너 안에 저장한 데이터는 Pod가 없어지면 같이 사라질 수 있다. 그래서 필요한 것이 Pod와 데이터 저장소의 분리(Pod는 교체 가능, 데이터는 유지)이며, PV/PVC는 이를 위한 표준 방식이다.

**PV와 PVC 이해**
- PV(PersistentVolume) = 실제 하드디스크(또는 NAS 공간)
- PVC(PersistentVolumeClaim) = "10GB짜리 디스크 빌려주세요" 신청서
- Pod = 빌린 디스크를 `/data` 같은 경로에 붙여서 사용

**PV (PersistentVolume)**
- 클러스터에 미리 준비된 실제 영구 저장소 리소스
- Pod와 독립적으로 존재한다

PV가 담고 있는 정보:
- 용량: 10Gi, 100Gi 등
- accessModes: RWO/RWX 등
- 실제 저장소 종류: NFS, EBS, Ceph, iSCSI 등
- ReclaimPolicy: PVC 삭제 시 데이터를 어떻게 할지(Delete/Retain)

**PVC (PersistentVolumeClaim)**
- Pod(또는 개발자)가 PV를 요청하는 요청서
- "이 정도 용량, 이 방식으로 쓸 수 있는 저장소가 필요해요"라고 요청한다.

PVC가 담는 정보:
- 필요한 용량: 10Gi
- accessModes: RWO, RWX
- storageClassName(선택): 어떤 종류의 디스크를 자동으로 만들지

PVC는 요구사항이고, 쿠버네티스가 그 요구사항에 맞는 PV를 찾아서 연결해준다.

**PV <--> PVC 연결(Binding)이 일어나는 과정**

1. PV가 존재한다 (또는 StorageClass로 자동 생성될 준비가 되어 있다.)
2. PVC를 생성한다 (예: 10Gi 필요합니다.)
3. 쿠버네티스가 조건이 맞는 PV를 찾는다.
   - 용량 >= 요청 용량
   - accessModes 일치
   - StorageClass 일치(있는 경우)
4. 조건이 맞으면 PV와 PVC가 연결된다 (Bound)
5. Pod는 PVC를 참조해서 마운트한다.

## Access Mode

**RWO (ReadWriteOnce)**
- 하나의 PV를 한 시점에 하나의 Node에서만 읽기/쓰기 가능
- worker1, worker2, worker3 모두 사용할 수는 있지만 동시에 여러 Node에서 사용할 수는 없음
- 같은 Node에 있는 여러 Pod가 같은 볼륨을 사용하는 것은 가능하다.
- DB나 일반적인 애플리케이션 데이터 저장에 많이 사용
- 대표적인 스토리지: AWS EBS, Azure Disk, Google Persistent Disk

**RWX (ReadWriteMany)**
- 하나의 PV를 여러 Node에서 동시에 읽기/쓰기 가능
- 여러 Pod가 같은 데이터를 공유해야 할 때 사용
- 보통 NFS, CephFS 같은 공유 파일시스템 사용

**ROX (ReadOnlyMany)**
- 하나의 PV를 여러 Node에서 동시에 사용 가능
- 단, 읽기만 가능
- 쓰기는 불가능
- 여러 Pod가 동일한 공용 데이터를 읽어야 할 때 사용

## volumeMode

volumeMode는 PV가 어떤 형태로 파드에 제공될지를 결정하는 옵션이다. 파일처럼 쓰게 할 건지, 디스크 덩어리 그대로 넘겨줄 건지를 정하는 설정이다.

**volumeMode: Filesystem**
- PV를 이미 포맷된 파일시스템 형태로 파드에 제공한다.
- 파드 입장에서는 `/data`, `/usr/share/nginx/html` 같은 일반 디렉터리로 보인다.

특징:
- 일반적으로 가장 많이 사용하며 거의 모든 실무 예제에서 사용
- 파일 저장, 로그 저장, 웹 컨텐츠 저장에 적합
- 이 PV를 파드에 마운트하면 디스크를 마운트한 폴더처럼 사용하게 된다.

**volumeMode: Block**
- 파일시스템 없이 디스크 블록 자체를 컨테이너에 전달
- 데이터베이스가 직접 디스크를 제어해야 할 때
- 특수한 스토리지/DB 튜닝 환경(일반 실무에서는 거의 사용하지 않는다.)

초반엔 Filesystem만 기억하면 된다.

**요약**
- Filesystem: 디렉터리처럼 쓰는 디스크
- Block: 가공 안 된 원판 디스크

## PersistentVolumeReclaimPolicy

PVC가 삭제되면 그 PV와 실제 데이터는 어떻게 할지를 결정해야 한다. 즉, 사용자가 볼륨을 안 쓰겠다고 했을 때 데이터는 버릴지 남길지를 정하는 정책이다.

**Retain (실습에서 사용한 값)**
- PVC가 삭제되어도 PV와 실제 데이터는 그대로 유지: `persistentVolumeReclaimPolicy: Retain`

상태 변화:
- PVC 삭제
- PV 상태: Released
- 데이터: 삭제되지 않음

특징:
- 데이터 보호에 가장 안전
- 관리자가 수동으로 처리해야 함

사용 예: DB 데이터, 중요한 로그 같은 경우 많이 사용

**Delete**
- PVC 삭제 시 PV + 실제 스토리지까지 함께 삭제: `persistentVolumeReclaimPolicy: Delete`
- 클라우드 스토리지(EBS, GCE PD)에서 주로 사용
  - 자동 정리 목적
  - hostPath에서는 사실상 의미가 거의 없음
  - 클라우드 스토리지에서 중요

**Recycle (거의 사용되지 않는다.)**
- 데이터만 삭제하고 PV는 재사용
- 상태: deprecated

## NFS 서버 설치 및 설정

### Control-Plane에서 NFS 서버 설치 (Master Node를 NFS Server로 사용)

```bash
[root@k8s-master ~]# dnf  install  -y  nfs-utils

[root@k8s-master ~]# systemctl  start  nfs-server		# NFS Server 실행
[root@k8s-master ~]# systemctl  enable  nfs-server		# 해당 서버가 재부팅되어도 NFS Server를 자동 재실행

[root@k8s-master ~]# systemctl  status  nfs-server
● nfs-server.service - NFS server and services
     Loaded: loaded (/usr/lib/systemd/system/nfs-server.service; enabled; preset: disabled)
     Active: active (exited) since Thu 2026-08-27 12:53:14 KST; 15s ago
       Docs: man:rpc.nfsd(8)
             man:exportfs(8)
    Process: 54195 ExecStartPre=/usr/sbin/exportfs -r (code=exited, status=0/SUCCESS)
    Process: 54197 ExecStart=/usr/sbin/rpc.nfsd (code=exited, status=0/SUCCESS)
    Process: 54217 ExecStart=/bin/sh -c if systemctl -q is-active gssproxy; then systemctl reload gssproxy ; fi (code=exited, sta>
   Main PID: 54217 (code=exited, status=0/SUCCESS)
        CPU: 11ms

 8월 27 12:53:14 k8s-master systemd[1]: Starting NFS server and services...
 8월 27 12:53:14 k8s-master systemd[1]: Finished NFS server and services.

[root@k8s-master ~]# systemctl  enable  --now  nfs-server		# start, enable을 동시에 처리
```

**NFS로 사용할 디렉터리 생성**

```bash
[root@k8s-master ~]# mkdir  -p  /export/k8s

[root@k8s-master ~]# chmod  777  /export/k8s/
```

**NFS expose 설정**

```bash
[root@k8s-master ~]# vi  /etc/exports
/export/k8s 192.168.10.0/24(rw,sync,no_root_squash)
:wq
```

- `/export/k8s`: NFS로 공유할 디렉터리 및 경로
- `192.168.10.0/24`: 접근을 허용할 네트워크 범위
- `rw`: 읽기, 쓰기 권한 허용
- `sync`: 실제 디스크에 데이터가 기록된 후 응답
- `no_root_squash`: NFS Client의 root 사용자 권한을 NFS Server의 root 권한으로 인정

```bash
   # 설정 반영
[root@k8s-master ~]# exportfs  -rv
exporting 192.168.10.0/24:/export/k8s

   # 설정 확인
[root@k8s-master ~]# exportfs  -v
/export/k8s     192.168.10.0/24(sync,wdelay,hide,no_subtree_check,sec=sys,rw,secure,no_root_squash,no_all_squash
```

**NFS Client(Worker Node)에서 NFS Server로 접근 확인**

```bash
[root@k8s-master ~]# ssh  guest@k8s-worker1

[guest@k8s-worker1 ~]$ sudo  dnf install  -y  nfs-utils

   # 접속 가능 목록 확인
[guest@k8s-worker1 ~]$ sudo showmount  -e  192.168.10.100
Export list for 192.168.10.100:
/export/k8s 192.168.10.0/24

[root@k8s-master ~]# ssh  guest@k8s-worker2

[guest@k8s-worker2 ~]$ sudo  dnf install  -y  nfs-utils

   # 접속 가능 목록 확인
[guest@k8s-worker2 ~]$ sudo showmount  -e  192.168.10.100
Export list for 192.168.10.100:
/export/k8s 192.168.10.0/24
```

## PV/PVC(NFS) 생성 및 Pod 마운트 실습

### PersistentVolume 생성

```yaml
# pv-nfs-rwx.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-nfs-rwx

spec:
  capacity:
    storage: 5Gi
  volumeMode: Filesystem

  accessModes:
    - ReadWriteMany

  persistentVolumeReclaimPolicy: Retain
  nfs:
    server: 192.168.10.100
    path: /export/k8s
```

```bash
[root@k8s-master ~]# kubectl  apply  -f  pv-nfs-rwx.yaml  --dry-run=client
persistentvolume/pv-nfs-rwx created (dry run)

# pv를 생성하지 않았기 때문에 pv가 확인되지 않는다.
[root@k8s-master ~]# kubectl  get  persistentvolume
No resources found

# pv를 생성하지 않았기 때문에 pv가 확인되지 않는다.
[root@k8s-master ~]# kubectl  get  pv
No resources found

[root@k8s-master ~]# kubectl  apply  -f  pv-nfs-rwx.yaml

[root@k8s-master ~]# kubectl  get  persistentvolume
NAME          CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   VOLUMEATTRIBUTESCLASS   REASON   AGE
pv-nfs-rwx   5Gi         RWX            Retain           Available                          <unset>                          2s

[root@k8s-master ~]# kubectl  describe  persistentvolume
Name:       	pv-nfs-rwx
Labels:        	<none>
Annotations:   	<none>
Finalizers:    	[kubernetes.io/pv-protection]
StorageClass:
Status:          	Available				#  PV 생성시 : Available , PVC 연결시 : Bound , PVC 삭제시 : Released
Claim:
Reclaim Policy:	Retain
Access Modes:    	RWX
VolumeMode:     	Filesystem
Capacity:        	5Gi
Node Affinity:   	<none>
Message:
Source:
    Type:      	NFS (an NFS mount that lasts the lifetime of a pod)
    Server:    	192.168.10.100
    Path:      	/export/k8s
    ReadOnly:  	false
Events:        	<none>
```

### PVC (디스크 요청) 생성

```yaml
# pvc-nfs-rwx.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-nfs-rwx
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 5Gi
```

```bash
[root@k8s-master ~]# kubectl  apply  -f  pvc-nfs-rwx.yaml
persistentvolumeclaim/pvc-nfs-rwx created

[root@k8s-master ~]# kubectl  get  persistentvolumeclaims  pvc-nfs-rwx
NAME           STATUS   VOLUME       CAPACITY   ACCESS MODES   STORAGECLASS   VOLUMEATTRIBUTESCLASS   AGE
pvc-nfs-rwx   Bound      pv-nfs-rwx   5Gi        RWX                           <unset>                 70s

[root@k8s-master ~]# kubectl  get  pvc  pvc-nfs-rwx
NAME           STATUS   VOLUME       CAPACITY   ACCESS MODES   STORAGECLASS   VOLUMEATTRIBUTESCLASS   AGE
pvc-nfs-rwx   Bound      pv-nfs-rwx   5Gi        RWX                           <unset>                 70s

[root@k8s-master ~]# kubectl describe  pvc  pvc-nfs-rwx
Name:          	pvc-nfs-rwx
Namespace:     	default
StorageClass:
Status:        	Bound
Volume:        	pv-nfs-rwx
Labels:        	<none>
Annotations:   	pv.kubernetes.io/bind-completed: yes
               	pv.kubernetes.io/bound-by-controller: yes
Finalizers:    	[kubernetes.io/pvc-protection]
Capacity:      	5Gi
Access Modes:  	RWX
VolumeMode:    	Filesystem
Used By:       	<none>
Events:        	<none>

[root@k8s-master ~]# kubectl describe  pv  pv-nfs-rwx
Name:            	pv-nfs-rwx
Labels:          	<none>
Annotations:     	pv.kubernetes.io/bound-by-controller: yes
Finalizers:      	[kubernetes.io/pv-protection]
StorageClass:
Status:          	Bound
Claim:           	default/pvc-nfs-rwx
Reclaim Policy: 	Retain
Access Modes:    	RWX
VolumeMode:      	Filesystem
Capacity:        	5Gi
Node Affinity:   	<none>
Message:
Source:
    Type:      	NFS (an NFS mount that lasts the lifetime of a pod)
    Server:    	192.168.10.100
    Path:      	/export/k8s
    ReadOnly:  	false
Events:        	<none>
```

### Pod 하나 만들어서 실제로 NFS에 쓰기

```yaml
# pod-nfs-test.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nfs-pvc-test
spec:
  containers:
  - name: app
    image: busybox:1.36
    command: ["/bin/sh", "-c"]
    args:
      - |
        echo "pvc mount test" > /data/test.txt
        sleep 3600

    volumeMounts:
    - name: nfsvol
      mountPath: /data

  volumes:
  - name: nfsvol
    persistentVolumeClaim:
      claimName: pvc-nfs-rwx
```

Pod가 가져온 볼륨(`nfsvol`)을 이 컨테이너 안에서는 `/data`라는 경로로 연결한다(`/data`는 쿠버네티스가 자동 생성).

```yaml
    volumeMounts:
    - name: nfsvol	      	# 이 컨테이너에서 사용할 볼륨 이름 (아래 volumes 섹션의 name과 반드시 같아야 한다)
      mountPath: /data	    # 컨테이너 내부에서 사용할 경로 (경로에 쓰는 데이터가 PVC --> PV --> NFS로 저장)
```

이 Pod는 `pvc-nfs-rwx`라는 PVC를 `nfsvol`이라는 이름으로 가져다 쓰겠다는 의미다.

```yaml
  volumes:
  - name: nfsvol	      	# 파드에서 사용하는 볼륨의 논리적 이름 (컨테이너의 volumeMounts.name 과 1:1로 매칭)
    persistentVolumeClaim:
      claimName: pvc-nfs-rwx
```

```bash
[root@k8s-master ~]# kubectl  apply  -f  pod-nfs-test.yaml
pod/nfs-pvc-test created

[root@k8s-master ~]# kubectl  get  pods  -o  wide
NAME           READY   STATUS    RESTARTS   AGE   IP              NODE          NOMINATED NODE   READINESS GATES
nfs-pvc-test   1/1     Running   0          44s   10.244.1.15   k8s-worker1   <none>            <none>

   # 컨테이너 접속
[root@k8s-master ~]# kubectl  exec  -it  nfs-pvc-test  -- sh
/ #
/ #
/ # ls  -l
total 20
drwxr-xr-x    2 root     root         12288 May 18  2023 bin
drwxrwxrwx    2 root     root            22 Aug 27 05:44 data
drwxr-xr-x    5 root     root           360 Aug 27 05:44 dev
drwxr-xr-x    1 root     root            54 Aug 27 05:44 etc
drwxr-xr-x    2 nobody   nobody           6 May 18  2023 home
drwxr-xr-x    2 root     root          4096 May 18  2023 lib
lrwxrwxrwx    1 root     root             3 May 18  2023 lib64 -> lib
dr-xr-xr-x  280 root     root             0 Aug 27 05:44 proc
drwx------    1 root     root            26 Aug 27 05:46 root
dr-xr-xr-x   13 root     root             0 Aug 27 05:44 sys
drwxrwxrwt    2 root     root             6 May 18  2023 tmp
drwxr-xr-x    4 root     root            29 May 18  2023 usr
drwxr-xr-x    1 root     root            17 Aug 27 05:44 var

/ # ls  -l  /data
total 4
-rw-r--r--    1 root     root            15 Aug 27 05:44 test.txt

/ # cat  /data/test.txt
pvc mount test

/ # touch  /data/sol-data

/ # ls  -l  /data
total 4
-rw-r--r--    1 root     root             0 Aug 27 05:48 sol-data
-rw-r--r--    1 root     root            15 Aug 27 05:44 test.txt

   # NFS Server (Control-Plane에서 확인)
[root@k8s-master ~]# ls  -l  /export/k8s/
합계 4
-rw-r--r-- 1 root root  0  8월 27 14:48 sol-data
-rw-r--r-- 1 root root 15  8월 27 14:44 test.txt
```

### 다른 Pod에서 같은 PVC 사용

첫 번째 Pod `nfs-pvc-test`는 이미 `pvc-nfs-rwx`를 사용 중이다. 두 번째 Pod도 같은 PVC를 요청해서 두 Pod가 서로 다른 워커노드에 배치되어도 같은 NFS 데이터를 공유하는지 확인한다.

해당 실습을 위해서는 k8s-worker2에 Pod가 생성되어야 한다. 실제 Pod는 스케줄러에 의해 생성되기 때문에 k8s-worker2에 생성된다는 보장이 없으므로, nodeSelector를 사용하여 k8s-worker2에 Pod가 생성되도록 설정한다.

```bash
[root@k8s-master ~]#  kubectl label node k8s-worker2 storage=nfs-test
node/k8s-worker2 labeled
```

```yaml
# pod-nfs-test2.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nfs-pvc-test2

spec:
  nodeSelector:
    storage: nfs-test
  containers:
  - name: app
    image: busybox:1.36
    command: ["/bin/sh", "-c"]
    args:
    - |
      sleep 3600

    volumeMounts:
    - name: nfsvol
      mountPath: /data

  volumes:
  - name: nfsvol
    persistentVolumeClaim:
      claimName: pvc-nfs-rwx
```

```bash
[root@k8s-master ~]# kubectl  get  pods  -o  wide
NAME            	READY   STATUS    RESTARTS   AGE   IP              NODE          NOMINATED NODE   READINESS GATES
nfs-pvc-test    	1/1     Running   0          17m   10.244.1.15   k8s-worker1   <none>            <none>
nfs-pvc-test2	    1/1     Running   0          2s    10.244.2.4    k8s-worker2   <none>            <none>

   # 두 번째 Pod에서 기존 데이터 확인
[root@k8s-master ~]# kubectl exec -it nfs-pvc-test2 -- sh
/ #

/ # ls  -l /data
total 4
-rw-r--r--    1 root     root             0 Aug 27 05:48 sol-data
-rw-r--r--    1 root     root            15 Aug 27 05:44 test.txt

/ # cat  /data/test.txt
pvc mount test

   # 두 번째 Pod에서 새로운 파일 생성
/ # echo "Created New File~~!!!" > /data/pod2File.txt

/ # ls  -l /data
total 8
-rw-r--r--    1 root     root            22 Aug 27 06:06 pod2File.txt
-rw-r--r--    1 root     root             0 Aug 27 05:48 sol-data
-rw-r--r--    1 root     root            15 Aug 27 05:44 test.txt
```

```
	worker1                                                                    worker2
	  │                                                                              │
	Pod1                                                                           Pod2
	  │                                                                              │
	  └─────────── 같은 PVC ────────────┘
	                                         │
	                                  pvc-nfs-rwx
	                                         │
	                                  pv-nfs-rwx
	                                         │
	                                NFS:/export/k8s
	                                         │
	                                실제 데이터 저장
```

```bash
   # 첫 번째 Pod에서 pod2File.txt 확인
[root@k8s-master ~]# kubectl  exec  nfs-pvc-test  -- ls  -l /data
total 8
-rw-r--r--    1 root     root            22 Aug 27 06:06 pod2File.txt
-rw-r--r--    1 root     root             0 Aug 27 05:48 sol-data
-rw-r--r--    1 root     root            15 Aug 27 05:44 test.txt

[root@k8s-master ~]# kubectl  exec  nfs-pvc-test  -- cat /data/pod2File.txt
Created New File~~!!!

   # NFS 서버에서 실제 파일 확인
[root@k8s-master ~]# ls  -l  /export/k8s/
합계 8
-rw-r--r-- 1 root root 22  8월 27 15:06 pod2File.txt
-rw-r--r-- 1 root root  0  8월 27 14:48 sol-data
-rw-r--r-- 1 root root 15  8월 27 14:44 test.txt

[root@k8s-master ~]# cat  /export/k8s/test.txt
pvc mount test

[root@k8s-master ~]# cat  /export/k8s/pod2File.txt
Created New File~~!!!
```

### 첫 번째 Pod를 삭제해도 데이터가 유지되는지 확인

```bash
   # 첫 번째 Pod 삭제
[root@k8s-master ~]# kubectl  delete  pods  nfs-pvc-test
pod "nfs-pvc-test" deleted from default namespace

   # 두 번째 Pod는 계속 실행 중이다.
[root@k8s-master ~]# kubectl  get  pods
NAME             READY   STATUS    RESTARTS   AGE
nfs-pvc-test2   1/1     Running   0          12m

   # 기존 데이터가 그대로 존재한다.
[root@k8s-master ~]# kubectl  exec  nfs-pvc-test2 -- ls  -l  /data
total 8
-rw-r--r--    1 root     root            22 Aug 27 06:06 pod2File.txt
-rw-r--r--    1 root     root             0 Aug 27 05:48 sol-data
-rw-r--r--    1 root     root            15 Aug 27 05:44 test.txt

   # 두 번째 Pod도 삭제
[root@k8s-master ~]# kubectl  delete  pod  nfs-pvc-test2
pod "nfs-pvc-test2" deleted from default namespace

  # Pod가 하나도 없어도 NFS 서버의 파일은 그대로 존재한다.
[root@k8s-master ~]# ls  -l  /export/k8s/
합계 8
-rw-r--r-- 1 root root 22  8월 27 15:06 pod2File.txt
-rw-r--r-- 1 root root  0  8월 27 14:48 sol-data
-rw-r--r-- 1 root root 15  8월 27 14:44 test.txt

   # PVC , PV 삭제
[root@k8s-master ~]# kubectl  delete  pvc  pvc-nfs-rwx
persistentvolumeclaim "pvc-nfs-rwx" deleted from default namespace

[root@k8s-master ~]# kubectl  delete  pv  pv-nfs-rwx
persistentvolume "pv-nfs-rwx" deleted
```

**정리**: Pod를 삭제해도, 심지어 PVC를 사용하는 Pod가 하나도 없어도 PVC/PV가 살아있는 한 NFS 서버의 실제 데이터는 그대로 유지된다 — 이것이 Pod와 데이터를 분리하는 PV/PVC 구조의 핵심이다.

## StorageClass · Dynamic Provisioning 개념

쿠버네티스에서 파드는 기본적으로 휘발성이다. 파드가 삭제되거나 다른 노드로 이동하면, 파드 안에 있던 파일은 같이 사라진다.

- 웹 서버의 index.html
- DB 컨테이너의 데이터 디렉터리
- 업로드한 파일

이런 데이터는 파드가 재시작되어도 유지돼야 한다. 그래서 쿠버네티스는 파드와 데이터를 분리하는 구조를 만들었고, 그 결과가 PV, PVC 개념이다. 하지만 초창기 방식에는 큰 문제가 있었다.

### 기존 방식(Static Provisioning)의 구조와 한계

초기 쿠버네티스의 스토리지 방식:

1. 관리자가 미리 PersistentVolume(PV)를 만든다.
2. 사용자가 PersistentVolumeClaim(PVC)를 만든다.
3. PVC가 조건에 맞는 PV를 찾아서 연결된다.

이 방식을 Static Provisioning이라고 부른다. 문제는 관리자가 미리 만들어야 한다는 점이다.

- 개발팀 A가 10Gi 스토리지를 요청
- 개발팀 B가 50Gi 스토리지를 요청
- QA 팀이 5Gi 스토리지를 요청

운영자는 다음을 미리 고민해야 한다.

- 몇 개의 PV를 만들 것인가
- 용량은 몇 Gi로 나눌 것인가
- NFS인지, 로컬인지, 클라우드 볼륨인지
- 다 쓰고 남은 PV는 어떻게 할 것인가

즉, 스토리지 관리가 사람 손에 의존하게 된다. 클러스터가 커질수록 이 방식은 유지가 거의 불가능해진다. 이 문제를 해결하기 위해 등장한 개념이 StorageClass와 Dynamic Provisioning이다.

### StorageClass

StorageClass는 한 줄로 말하면 스토리지를 만드는 방법에 대한 템플릿(설계도)이다.

- StorageClass는 실제 디스크가 아니다.
- 스토리지 서버도 아니다.
- 용량도 아니다.
- "이런 방식으로 스토리지를 만들어라"라고 쿠버네티스에게 알려주는 규칙이다.

StorageClass의 정보:

- 어떤 스토리지 타입을 쓸 것인가(NFS, AWS EBS, Ceph, iSCSI 등)
- 누가 실제 볼륨을 만들어줄 것인가(provisioner)
- 볼륨을 삭제할 때 같이 지울 것인가, 남길 것인가
- 성능 옵션이나 마운트 방식

즉, StorageClass는 자동 생성 규칙이다.

### Dynamic Provisioning

Dynamic Provisioning은 동작 방식이다. PVC가 생성되는 순간, 필요한 PV를 자동으로 만들어주는 방식이다.

이전 방식:
- 관리자 --> PV 생성
- 사용자 --> PVC 생성
- 쿠버네티스 --> 둘을 매칭

Dynamic Provisioning에서는 순서가 바뀐다.
- 사용자 --> PVC 생성
- 쿠버네티스 --> StorageClass 확인
- 쿠버네티스 --> PV 자동 생성
- PVC <--> PV 자동 연결

PV를 관리자가 미리 만들지 않고 PVC가 필요해질 때 그 순간에 PV가 생성된다. 이 자동 생성이 바로 Dynamic Provisioning이다.

**1단계** — 관리자가 StorageClass를 하나 만들어 둔다. 이 StorageClass는 "PVC가 오면, NFS 서버의 이 경로를 사용해서, 이런 옵션으로, 볼륨을 하나 자동 생성해라"라고 작성한다.

**2단계** — 개발자가 PVC를 만든다. "나는 10Gi가 필요하다, 이 StorageClass를 사용하겠다"라고 작성한다.

**3단계** — 쿠버네티스가 PVC를 보고 판단한다. Static PV를 찾는 게 아니라 StorageClass가 지정돼 있으니 Dynamic Provisioning을 사용한다.

**4단계** — StorageClass에 정의된 provisioner가 실행된다. NFS라면 디렉터리를 만들고, 클라우드라면 디스크를 생성하고, 쿠버네티스 안에는 PV 오브젝트를 자동 생성한다.

**5단계** — 생성된 PV가 PVC와 즉시 연결된다. 사용자는 PV가 언제, 어떻게 만들어졌는지 몰라도 된다. PVC만 쓰면 된다.

## StorageClass + Dynamic Provisioning 실습

Static Provisioning에서는 관리자가 PV를 직접 미리 만들었다. Dynamic Provisioning에서는 관리자가 PV를 미리 만들지 않고, 사용자가 PVC를 생성하면 StorageClass와 Provisioner를 통해 PV가 자동으로 생성된다. PVC 개수만큼 각각의 PV가 자동으로 생성되는 과정과, Pod가 PVC를 실제로 사용해서 NFS 서버에 데이터를 저장하는지 확인한다.

### Step1) NFS Provisioner 설치

NFS 서버만 있다고 해서 Dynamic Provisioning이 자동으로 동작하는 것은 아니다. NFS 서버는 단순히 실제 저장공간을 제공하는 역할을 한다. 하지만 PVC가 생성되었을 때 PVC 요청을 확인하고, NFS 서버에 사용할 디렉터리를 만들고, PV를 자동 생성하고, PVC와 PV를 연결하는 작업을 수행할 구성 요소가 필요하다. 이 역할을 하는 것이 NFS Provisioner이다.

- NFS Server: 실제 데이터를 저장하는 공간 또는 서버
- StorageClass: 어떤 방식으로 저장공간을 만들 것인지 정의하는 규칙
- NFS Provisioner: PVC 요청을 받아 실제 NFS 저장공간과 PV를 자동으로 만들어주는 프로그램
- PVC: 필요한 저장공간 요청
- PV: Provisioner에 의해 자동으로 생성되는 Kubernetes 저장공간 리소스

NFS Provisioner가 없다면 StorageClass만 만들어도 실제로 PV를 자동 생성해 줄 구성 요소가 없기 때문에 Dynamic Provisioning이 동작하지 않는다.

### Step2) NFS 서버 준비

k8s-master를 NFS 서버로 사용한다.

- NFS Server IP: 192.168.10.100
- 공유 디렉터리: /storage/dynamic-multi
- 허용 네트워크: 192.168.10.0/24

```bash
# NFS 패키지 설치
[root@k8s-master ~]# dnf install -y nfs-utils

# NFS 서비스 시작
[root@k8s-master ~]# systemctl start nfs-server

# 부팅 시 자동 시작
[root@k8s-master ~]# systemctl enable nfs-server

# NFS 공유 디렉터리 생성
[root@k8s-master ~]# mkdir -p /storage/dynamic-multi

# 권한 설정
[root@k8s-master ~]# chmod 777 /storage/dynamic-multi

# NFS export 설정
[root@k8s-master ~]# vi /etc/exports
/storage/dynamic-multi 192.168.10.0/24(rw,sync,no_root_squash)

# 설정 적용
[root@k8s-master ~]# exportfs -rv
exporting 192.168.10.0/24:/storage/dynamic-multi

# 설정 확인
[root@k8s-master ~]# exportfs -v
/storage/dynamic-multi
                192.168.10.0/24(sync,wdelay,hide,no_subtree_check,sec=sys,rw,secure,no_root_squash,no_all_squash)
```

`/storage/dynamic-multi`가 Dynamic Provisioning에서 사용할 실제 NFS 저장공간이다. 앞으로 PVC가 생성되면 NFS Provisioner가 해당 디렉터리 안에 PVC별 저장공간을 자동으로 생성한다.

```
/storage/dynamic-multi/
 |
 +-- PVC-1용 디렉터리
 |
 +-- PVC-2용 디렉터리
 |
 +-- PVC-3용 디렉터리
```

### Step3) Worker Node에서 NFS Client 준비

k8s-worker1과 k8s-worker2에서 NFS 볼륨을 사용할 수 있도록 NFS Client 패키지를 설치하고 NFS Server의 공유 디렉터리가 보이는지 확인한다.

```bash
   # worker1
[root@k8s-master ~]# ssh guest@k8s-worker1

[root@k8s-worker1 ~]# dnf install -y nfs-utils

[root@k8s-worker1 ~]# showmount -e 192.168.10.100
Export list for 192.168.10.100:
/storage/dynamic-multi 192.168.10.0/24

   # worker2
[root@k8s-master ~]# ssh guest@k8s-worker2

[root@k8s-worker2 ~]# dnf install -y nfs-utils

[root@k8s-worker2 ~]# showmount -e 192.168.10.100
Export list for 192.168.10.100:
/storage/dynamic-multi 192.168.10.0/24
```

Pod가 어느 Worker Node에 생성될지 모르기 때문에 모든 Worker Node에서 NFS를 사용할 수 있어야 한다. Worker Node의 kubelet이 NFS Volume을 Mount할 때 NFS 관련 Client 프로그램이 필요하다.

### Step4) Helm 설치 확인

NFS Provisioner를 쉽게 설치하기 위해 Helm이 설치되어 있는지 확인한다. Helm은 Kubernetes 애플리케이션을 패키지 형태로 설치하고 관리할 수 있는 도구이다. NFS Provisioner는 여러 Kubernetes 리소스가 필요하기 때문에 Helm을 이용하면 간단하게 설치할 수 있다. Helm이 설치되어 있지 않다면 먼저 Helm을 설치해야 한다.

- Linux: dnf / yum / apt
- Kubernetes: Helm

```bash
   # Helm Version 확인
[root@k8s-master ~]# helm  version
bash: helm: 명령을 찾을 수 없습니다...

 	# HELM 설치
[root@k8s-master ~]# curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 12252  100 12252    0     0   498k      0 --:--:-- --:--:-- --:--:--  520k
Downloading https://get.helm.sh/helm-v3.21.4-linux-amd64.tar.gz
Verifying checksum... Done.
Preparing to install helm into /usr/local/bin
helm installed into /usr/local/bin/helm

   # Helm Version 확인
[root@k8s-master ~]# helm  version
version.BuildInfo{Version:"v3.21.4", GitCommit:"813176c51bb5c181dbbd7901298ddcc104cd3417", GitTreeState:"clean", GoVersion:"go1.26.5"}

 	# NFS Subdir External Provisioner를 설치하기 위한 Helm Repository를 추가
[root@k8s-master ~]# helm repo add nfs-subdir-external-provisioner \
https://kubernetes-sigs.github.io/nfs-subdir-external-provisioner/
"nfs-subdir-external-provisioner" has been added to your repositories

# Repository 정보 갱신
[root@k8s-master ~]# helm repo update
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "nfs-subdir-external-provisioner" chart repository
Update Complete. ⎈Happy Helming!⎈

# Repository 확인
[root@k8s-master ~]# helm repo list
NAME                            	   URL
nfs-subdir-external-provisioner  https://kubernetes-sigs.github.io/nfs-subdir-external-provisioner/
```

`nfs-subdir-external-provisioner`는 이미 존재하는 NFS 서버를 이용해서 Dynamic Provisioning을 가능하게 해주는 Provisioner이다. NFS 서버 자체를 만드는 것이 아니라 이미 구성되어 있는 NFS 서버와 공유 디렉터리를 사용한다.

**NFS Provisioner 설치**

- Release 이름: nfs-provisioner
- NFS Server: 192.168.10.100
- NFS Path: /storage/dynamic-multi
- StorageClass 이름: sc-nfs-multi

```bash
[root@k8s-master ~]# helm install nfs-provisioner \
nfs-subdir-external-provisioner/nfs-subdir-external-provisioner \
--set nfs.server=192.168.10.100 \
--set nfs.path=/storage/dynamic-multi \
--set storageClass.name=sc-nfs-multi
NAME: nfs-provisioner
LAST DEPLOYED: Thu Aug 27 16:11:50 2026
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
```

- `helm install`: Helm Chart를 이용해서 Kubernetes에 프로그램 설치
- `nfs-provisioner`: Helm Release 이름(설치된 패키지를 관리할 때 사용하는 이름)
- `nfs-subdir-external-provisioner/nfs-subdir-external-provisioner`: 설치할 Helm Chart(NFS 서버를 이용해서 Dynamic Provisioning을 수행하는 Provisioner)

옵션:
- `--set nfs.server=192.168.10.100`: 사용할 NFS Server의 IP 주소
- `--set nfs.path=/storage/dynamic-multi`: NFS Server에서 사용할 공유 디렉터리
- `--set storageClass.name=sc-nfs-multi`: Provisioner가 사용할 StorageClass 이름

### Step5) NFS Provisioner Pod 확인

NFS Provisioner가 정상적으로 실행되고 있는지 확인한다.

```bash
   # provisioner가 Pod로 만들어진다.
[root@k8s-master ~]# kubectl  get  pods
NAME                                                              		READY   STATUS    RESTARTS   AGE
nfs-provisioner-nfs-subdir-external-provisioner-854fdbf4b76kld9	1/1     Running   0          3m38s
```

NFS Provisioner도 Kubernetes 안에서는 Pod 형태로 실행된다. STATUS가 "Running"이어야 정상적으로 PVC 요청을 처리할 수 있다. 만약 Provisioner Pod가 실행되지 않으면 PVC 생성, PV 자동 생성 과정이 정상적으로 동작하지 않는다.

### Step6) StorageClass 확인

NFS Provisioner 설치 과정에서 생성된 StorageClass를 확인한다.

```bash
[root@k8s-master ~]# kubectl  get  storageclasses sc-nfs-multi
NAME           PROVISIONER                                                     RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
sc-nfs-multi   cluster.local/nfs-provisioner-nfs-subdir-external-provisioner   Delete          Immediate           true                   5m34s
```

**StorageClass 설정 시 (예시)**

```bash
[root@k8s-master ~]# helm install nfs-provisioner \
nfs-subdir-external-provisioner/nfs-subdir-external-provisioner \
--set nfs.server=192.168.10.100 \
--set nfs.path=/storage/dynamic-multi \
--set storageClass.name=sc-nfs-multi \
--set storageClass.defaultClass=false \
--set storageClass.reclaimPolicy=Delete \
--set storageClass.archiveOnDelete=true \
--set storageClass.accessModes=ReadWriteMany \
--set storageClass.volumeBindingMode=Immediate
```

**StorageClass 수정 시 (예시)**

```bash
[root@k8s-master ~]# helm upgrade nfs-provisioner \
nfs-subdir-external-provisioner/nfs-subdir-external-provisioner \
--set nfs.server=192.168.10.100 \
--set nfs.path=/storage/dynamic-multi \
--set storageClass.name=sc-nfs-multi \
--set storageClass.defaultClass=false \
--set storageClass.reclaimPolicy=Delete \
--set storageClass.archiveOnDelete=true \
--set storageClass.accessModes=ReadWriteMany \
--set storageClass.volumeBindingMode=Immediate
```

StorageClass 옵션 정리:

- `storageClass.reclaimPolicy`: PVC 삭제 후 PV를 어떻게 처리할지 결정
  - Delete: PVC 삭제 시 PV도 삭제
  - Retain: PVC 삭제 후에도 PV 유지
- `storageClass.volumeBindingMode`: PVC와 PV를 언제 Binding할지 결정
  - Immediate: PVC 생성 즉시 PV 생성 및 Binding, NFS 같은 공유 스토리지에서 많이 사용
  - WaitForFirstConsumer: PVC를 사용하는 Pod가 생성될 때까지 기다림, Node/Zone 위치가 중요한 스토리지에서 사용
- `storageClass.defaultClass`: 해당 StorageClass를 기본 StorageClass로 사용할지 결정
  - true: 기본 StorageClass로 지정(생략 가능)
  - false: 기본 StorageClass로 지정하지 않음(직접 지정)
- `storageClass.allowVolumeExpansion`: PVC 용량 확장을 허용할지 결정
  - true: PVC 용량 확장 허용(예: 5Gi --> 10Gi)
  - false: PVC 용량 확장 불가
- `storageClass.archiveOnDelete`: PV가 삭제될 때 NFS 서버에 있는 실제 데이터 디렉터리를 어떻게 할지 결정
  - true: 데이터를 바로 삭제하지 않고 `archived-...` 형태로 보관
  - false: archive 형태로 보관하지 않음
- `storageClass.accessModes`: 자동 생성되는 볼륨의 접근 방식
  - ReadWriteOnce: RWO(한 Node에서 읽기/쓰기)
  - ReadWriteMany: RWX(여러 Node에서 동시에 읽기/쓰기, NFS에서 주로 사용)
  - ReadOnlyMany: ROX(여러 Node에서 동시에 읽기만 가능)

### Step7) PV가 없는 상태 확인

PVC를 생성하기 전에 현재 PV 상태를 확인한다.

```bash
[root@k8s-master ~]# kubectl  get  pv
No resources found
```

StorageClass와 Provisioner는 준비되었지만 아직 PVC 요청이 없기 때문에 PV도 생성되지 않는다.

### Step8) PVC 3개 생성

PVC 3개를 생성한다.

- pvc-log: 용량 1Gi
- pvc-image: 용량 5Gi
- pvc-backup: 용량 10Gi

공통 조건: AccessMode는 ReadWriteMany, StorageClass는 sc-nfs-multi

```yaml
# pvc-multi.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-log
spec:
  accessModes:
  - ReadWriteMany
  storageClassName: sc-nfs-multi
  resources:
    requests:
      storage: 1Gi
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-image
spec:
  accessModes:
  - ReadWriteMany
  storageClassName: sc-nfs-multi
  resources:
    requests:
      storage: 5Gi
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-backup
spec:
  accessModes:
  - ReadWriteMany
  storageClassName: sc-nfs-multi
  resources:
    requests:
      storage: 10Gi
```

```bash
[root@k8s-master ~]# kubectl  apply -f  pvc-multi.yaml  --dry-run=client
persistentvolumeclaim/pvc-log created (dry run)
persistentvolumeclaim/pvc-image created (dry run)
persistentvolumeclaim/pvc-backup created (dry run)

[root@k8s-master ~]# kubectl  apply -f  pvc-multi.yaml
persistentvolumeclaim/pvc-log created
persistentvolumeclaim/pvc-image created
persistentvolumeclaim/pvc-backup created
```

PV YAML 파일을 작성 및 실행하지 않고 PVC만 YAML 파일을 작성 및 실행한다. 각 PVC는 `storageClassName: sc-nfs-multi`를 통해 NFS Provisioner를 사용하게 된다.

### Step9) PVC 상태 확인

```bash
[root@k8s-master ~]# kubectl  get  pvc
NAME         STATUS   VOLUME                                      CAPACITY   ACCESS MODES   STORAGECLASS   VOLUMEATTRIBUTESCLASS   AGE
pvc-backup   Bound    pvc-bbeabf9d-03dc-40a6-97de-d164573815b4    10Gi       RWX            sc-nfs-multi   <unset>                 7s
pvc-image    Bound    pvc-0f2c6eb1-53fe-40ba-888f-d7e76bb53f3b    5Gi        RWX            sc-nfs-multi   <unset>                 7s
pvc-log      Bound    pvc-246d1cf4-e25a-477b-9ec2-3acb494163ca    1Gi        RWX            sc-nfs-multi   <unset>                 7s
```

PVC가 모두 Bound 상태다. PV를 직접 생성하지 않았지만 PVC에 자동으로 PV가 연결되어 있다.

동작 과정: `PVC 생성 --> sc-nfs-multi 확인 --> NFS Provisioner 동작 --> NFS 저장공간 준비 --> PV 자동 생성 --> PVC와 PV Binding`

### Step10) 자동 생성된 PV 확인

```bash
[root@k8s-master ~]# kubectl  get  pv
NAME                                        CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                 STORAGECLASS   VOLUMEATTRIBUTESCLASS   REASON   AGE
pvc-0f2c6eb1-53fe-40ba-888f-d7e76bb53f3b    5Gi        RWX            Delete           Bound    default/pvc-image     sc-nfs-multi   <unset>                          3m40s
pvc-246d1cf4-e25a-477b-9ec2-3acb494163ca    1Gi        RWX            Delete           Bound    default/pvc-log       sc-nfs-multi   <unset>                          3m40s
pvc-bbeabf9d-03dc-40a6-97de-d164573815b4    10Gi       RWX            Delete           Bound    default/pvc-backup    sc-nfs-multi   <unset>                          3m40s
```

PVC 3개를 생성하면 PV도 3개가 자동 생성된다. 이것이 Dynamic Provisioning이다.

기본적인 Binding 구조:
- pvc-log <--> PV-1
- pvc-image <--> PV-2
- pvc-backup <--> PV-3

PV 하나의 남은 용량을 여러 PVC가 나눠 쓰는 구조가 아니다.

### Step11) NFS 서버의 자동 생성 디렉터리 확인

PVC 3개가 생성된 후 NFS 서버의 실제 공유 디렉터리를 확인한다.

```bash
[root@k8s-master ~]# ls  -l  /storage/dynamic-multi/
합계 0
drwxrwxrwx 2 root root 6  8월 27 16:48 default-pvc-backup-pvc-bbeabf9d-03dc-40a6-97de-d164573815b4
drwxrwxrwx 2 root root 6  8월 27 16:48 default-pvc-image-pvc-0f2c6eb1-53fe-40ba-888f-d7e76bb53f3b
drwxrwxrwx 2 root root 6  8월 27 16:48 default-pvc-log-pvc-246d1cf4-e25a-477b-9ec2-3acb494163ca
```

StorageClass가 직접 디렉터리를 만든 것이 아니라 NFS Provisioner가 StorageClass의 요청을 받아 실제 NFS 서버에 디렉터리를 생성한다.

```
/storage/dynamic-multi/
 |
 +-- pvc-log용     저장공간
 |
 +-- pvc-image용  저장공간
 |
 +-- pvc-backup용 저장공간
```

### Step12) 자동 생성된 PV 상세 확인

```bash
   # PVC 정보 확인
[root@k8s-master ~]# kubectl  describe  pvc  pvc-log
Name:          	pvc-log
Namespace:     	default
StorageClass:  	sc-nfs-multi
Status:        	Bound
Volume:        	pvc-246d1cf4-e25a-477b-9ec2-3acb494163ca
Labels:        	<none>
Annotations:   	pv.kubernetes.io/bind-completed: yes
               	pv.kubernetes.io/bound-by-controller: yes
               	volume.beta.kubernetes.io/storage-provisioner: cluster.local/nfs-provisioner-nfs-subdir-external-provisioner
               	volume.kubernetes.io/storage-provisioner: cluster.local/nfs-provisioner-nfs-subdir-external-provisioner
Finalizers:    	[kubernetes.io/pvc-protection]
Capacity:      	1Gi
Access Modes:  	RWX
VolumeMode:    	Filesystem
Used By:       	<none>
Events:
  Type    Reason                 Age    From                                                                                                                                                                Message
  ----    ------                 ----   ----                                                                                                                                                                -------
  Normal  ExternalProvisioning   8m51s  persistentvolume-controller                                                                                                                                         Waiting for a volume to be created either by the external provisioner 'cluster.local/nfs-provisioner-nfs-subdir-external-provisioner' or manually by the system administrator. If volume creation is delayed, please verify that the provisioner is running and correctly registered.
  Normal  Provisioning           8m51s  cluster.local/nfs-provisioner-nfs-subdir-external-provisioner_nfs-provisioner-nfs-subdir-external-provisioner-854fdbf4b76kld9_241ca03a-be47-4f8c-94ed-bb4250435afb  External provisioner is provisioning volume for claim "default/pvc-log"
  Normal  ProvisioningSucceeded  8m51s  cluster.local/nfs-provisioner-nfs-subdir-external-provisioner_nfs-provisioner-nfs-subdir-external-provisioner-854fdbf4b76kld9_241ca03a-be47-4f8c-94ed-bb4250435afb  Successfully provisioned volume pvc-246d1cf4-e25a-477b-9ec2-3acb494163ca

[root@k8s-master ~]# kubectl  get  pvc  pvc-log
NAME      STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   VOLUMEATTRIBUTESCLASS   AGE
pvc-log   Bound    pvc-246d1cf4-e25a-477b-9ec2-3acb494163ca   1Gi        RWX            sc-nfs-multi   <unset>                 14m

   # PV 정보 확인
[root@k8s-master ~]# kubectl  describe  pv  pvc-246d1cf4-e25a-477b-9ec2-3acb494163ca
Name:            	pvc-246d1cf4-e25a-477b-9ec2-3acb494163ca
Labels:          	<none>
Annotations:    	 pv.kubernetes.io/provisioned-by: cluster.local/nfs-provisioner-nfs-subdir-external-provisioner
Finalizers:      	[kubernetes.io/pv-protection]
StorageClass:    	sc-nfs-multi
Status:          	Bound
Claim:           	default/pvc-log
Reclaim Policy:  	Delete
Access Modes:    	RWX
VolumeMode:      	Filesystem
Capacity:        	1Gi
Node Affinity:   	<none>
Message:
Source:
    Type:      	NFS (an NFS mount that lasts the lifetime of a pod)
    Server:    	192.168.10.100
    Path:      	/storage/dynamic-multi/default-pvc-log-pvc-246d1cf4-e25a-477b-9ec2-3acb494163ca
    ReadOnly:  	false
Events:        	<none>
```

`pvc-log`는 PVC 이름이다. PV 이름은 별도로 자동 생성된다. 먼저 실제 PV 이름을 확인한 후 describe해야 한다.

### Step13) PVC를 사용하는 Pod 생성

`pvc-log`를 사용하는 Pod를 생성한다.

- 이름: log-pod
- 이미지: busybox:1.36
- PVC: pvc-log
- Mount Path: /data

```yaml
# pod-dynamic-test.yaml
apiVersion: v1
kind: Pod
metadata:
  name: log-pod
spec:
  containers:
  - name: app-log-container
    image: busybox:1.36
    command:
    - /bin/sh
    - -c
    args:
    - sleep 3600

    volumeMounts:
    - name: log-volume
      mountPath: /data

  volumes:
  - name: log-volume
    persistentVolumeClaim:
      claimName: pvc-log
```

```bash
[root@k8s-master ~]# kubectl  apply  -f   pod-dynamic-test.yaml
pod/log-pod created

[root@k8s-master ~]# kubectl  get  pods
NAME                                                              		READY   STATUS    RESTARTS   AGE
log-pod                                                           		1/1     Running   0          7s
nfs-provisioner-nfs-subdir-external-provisioner-854fdbf4b76kld9   	1/1     Running   0          55m

[root@k8s-master ~]# kubectl  get  pods  -o  wide
NAME                                                                    		READY   STATUS    RESTARTS   AGE	   IP              NODE
log-pod                                                                              	1/1     Running   0          72s 	   10.244.1.16   k8s-worker1
nfs-provisioner-nfs-subdir-external-provisioner-854fdbf4b76kld9   	1/1     Running   0          56m	   10.244.2.5    k8s-worker2
```

Pod는 StorageClass를 직접 사용하지 않는다. Pod는 PV도 직접 지정하지 않는다. Pod는 PVC 이름만 지정한다.

### Step14) Pod에서 파일 생성

`log-pod`의 `/data`에 `log.txt` 파일을 생성한다.

```bash
[root@k8s-master ~]# kubectl  exec  -it  log-pod   -- sh
/ #
/ #

/ # echo "Dynamic Provisioning Test" > /data/log.txt

/ # ls  -l  /data
total 4
-rw-r--r--    1 root     root            26 Aug 27 08:11 log.txt

/ # cat  /data/log.txt
Dynamic Provisioning Test
```

`/data`는 PVC를 통해 NFS Storage와 연결되어 있다. 따라서 `/data`에 저장하는 파일은 Pod의 임시 파일시스템에만 저장되는 것이 아니다.

### Step15) NFS 서버에서 파일 확인

Pod에서 생성한 `log.txt` 파일이 실제 NFS 서버에 저장되었는지 확인한다.

```bash
[root@k8s-master ~]# ls  -l  /storage/dynamic-multi
합계 0
drwxrwxrwx 2 root root  6  8월 27 16:48 default-pvc-backup-pvc-bbeabf9d-03dc-40a6-97de-d164573815b4
drwxrwxrwx 2 root root  6  8월 27 16:48 default-pvc-image-pvc-0f2c6eb1-53fe-40ba-888f-d7e76bb53f3b
drwxrwxrwx 2 root root 21  8월 27 17:11 default-pvc-log-pvc-246d1cf4-e25a-477b-9ec2-3acb494163ca

[root@k8s-master ~]# ls  -l  /storage/dynamic-multi/default-pvc-log-pvc-246d1cf4-e25a-477b-9ec2-3acb494163ca/
합계 4
-rw-r--r-- 1 root root 26  8월 27 17:11 log.txt

[root@k8s-master ~]# cat  /storage/dynamic-multi/default-pvc-log-pvc-246d1cf4-e25a-477b-9ec2-3acb494163ca/log.txt
Dynamic Provisioning Test
```

Pod의 `/data`에 만든 파일이 실제 NFS Server에 저장되었다.

```
Pod
 # /data/log.txt  -->  PVC  -->  PV  -->  NFS Server  -->  /export/dynamic-multi/default-pvc-log-pvc-.../log.txt
```

### Step16) Pod 삭제 후 데이터 유지 확인

`log-pod`를 삭제한 후 PVC와 PV가 그대로 유지되는지 확인한다.

```bash
[root@k8s-master ~]# kubectl  delete  pod  log-pod
pod "log-pod" deleted from default namespace

[root@k8s-master ~]# kubectl  get  pvc
NAME         STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   VOLUMEATTRIBUTESCLASS   AGE
pvc-backup   Bound    pvc-bbeabf9d-03dc-40a6-97de-d164573815b4   10Gi       RWX            sc-nfs-multi   <unset>                 29m
pvc-image    Bound    pvc-0f2c6eb1-53fe-40ba-888f-d7e76bb53f3b   5Gi        RWX            sc-nfs-multi   <unset>                 29m
pvc-log      Bound    pvc-246d1cf4-e25a-477b-9ec2-3acb494163ca   1Gi        RWX            sc-nfs-multi   <unset>                 29m

[root@k8s-master ~]# kubectl  get  pv pvc-246d1cf4-e25a-477b-9ec2-3acb494163ca
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM             STORAGECLASS   VOLUMEATTRIBUTESCLASS   REASON   AGE
pvc-246d1cf4-e25a-477b-9ec2-3acb494163ca   1Gi        RWX            Delete           Bound    default/pvc-log   sc-nfs-multi   <unset>                          30m
```

Pod를 삭제해도 PVC는 삭제되지 않는다. PVC가 존재하므로 PV도 계속 연결되어 있다. NFS 서버에 저장된 데이터도 그대로 유지된다.

### Step17) Pod 재생성 후 기존 데이터 확인

같은 PVC(`pvc-log`)를 사용하는 Pod를 다시 생성하고 기존 `log.txt` 파일을 확인한다.

```bash
   # Pod 생성
[root@k8s-master ~]# kubectl  apply  -f   pod-dynamic-test.yaml
pod/log-pod created

   # 컨테이너 접속 후 log.txt 파일 확인
[root@k8s-master ~]# kubectl  exec  -it  log-pod -- sh
/ #
/ #
/ # ls  -l /data/
total 4
-rw-r--r--    1 root     root            26 Aug 27 08:11 log.txt
/ #
/ #
/ # cat /data/log.txt
Dynamic Provisioning Test

   # NFS Server에 기존의 데이터 확인
[root@k8s-master ~]# ls  -l  /storage/dynamic-multi/default-pvc-log-pvc-246d1cf4-e25a-477b-9ec2-3acb494163ca/
합계 4
-rw-r--r-- 1 root root 26  8월 27 17:11 log.txt

[root@k8s-master ~]# cat  /storage/dynamic-multi/default-pvc-log-pvc-246d1cf4-e25a-477b-9ec2-3acb494163ca/log.txt
Dynamic Provisioning Test
```

새로 생성된 Pod가 동일한 PVC를 사용하기 때문에 기존 NFS 데이터에 다시 접근할 수 있다.

### Step18) PVC 삭제 후 PV 자동 삭제 확인

`log-pod`를 삭제한 후 `pvc-log`를 삭제하고, `pvc-log`와 연결되어 있던 PV도 자동으로 삭제되는지 확인한다.

```bash
[root@k8s-master ~]# kubectl  delete  pods log-pod
pod "log-pod" deleted from default namespace

   # PVC 3개 확인
[root@k8s-master ~]# kubectl  get  pvc
NAME         STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   VOLUMEATTRIBUTESCLASS   AGE
pvc-backup   Bound    pvc-bbeabf9d-03dc-40a6-97de-d164573815b4   10Gi       RWX            sc-nfs-multi   <unset>                 51m
pvc-image    Bound    pvc-0f2c6eb1-53fe-40ba-888f-d7e76bb53f3b   5Gi        RWX            sc-nfs-multi   <unset>                 51m
pvc-log      Bound    pvc-246d1cf4-e25a-477b-9ec2-3acb494163ca   1Gi        RWX            sc-nfs-multi   <unset>                 51m

   # PV 3개 확인
[root@k8s-master ~]# kubectl  get  pv
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                STORAGECLASS   VOLUMEATTRIBUTESCLASS   REASON   AGE
pvc-0f2c6eb1-53fe-40ba-888f-d7e76bb53f3b   5Gi        RWX            Delete           Bound    default/pvc-image    sc-nfs-multi   <unset>                          51m
pvc-246d1cf4-e25a-477b-9ec2-3acb494163ca   1Gi        RWX            Delete           Bound    default/pvc-log      sc-nfs-multi   <unset>                          51m
pvc-bbeabf9d-03dc-40a6-97de-d164573815b4   10Gi       RWX            Delete           Bound    default/pvc-backup   sc-nfs-multi   <unset>                          51m

   # pvc-log PVC 삭제
[root@k8s-master ~]# kubectl  delete  pvc pvc-log
persistentvolumeclaim "pvc-log" deleted from default namespace

   # pvc-log PVC가 삭제되고 PVC 2개가 확인
[root@k8s-master ~]# kubectl  get  pvc
NAME         STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   VOLUMEATTRIBUTESCLASS   AGE
pvc-backup   Bound    pvc-bbeabf9d-03dc-40a6-97de-d164573815b4   10Gi       RWX            sc-nfs-multi   <unset>                 53m
pvc-image    Bound    pvc-0f2c6eb1-53fe-40ba-888f-d7e76bb53f3b   5Gi        RWX            sc-nfs-multi   <unset>                 53m

   # default/pvc-log PV가 삭제되고 PV 2개가 확인
[root@k8s-master ~]# kubectl  get  pv
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                STORAGECLASS   VOLUMEATTRIBUTESCLASS   REASON   AGE
pvc-0f2c6eb1-53fe-40ba-888f-d7e76bb53f3b   5Gi        RWX            Delete           Bound    default/pvc-image    sc-nfs-multi   <unset>                          53m
pvc-bbeabf9d-03dc-40a6-97de-d164573815b4   10Gi       RWX            Delete           Bound    default/pvc-backup   sc-nfs-multi   <unset>                          53m
```

NFS Provisioner가 생성한 StorageClass의 reclaimPolicy가 Delete라면 PVC가 삭제된 후 연결된 PV도 자동으로 삭제된다.

```bash
[root@k8s-master ~]# kubectl get storageclass
NAME           PROVISIONER                                                     RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
sc-nfs-multi   cluster.local/nfs-provisioner-nfs-subdir-external-provisioner   Delete          Immediate           true                   94m

[root@k8s-master ~]# ls  -l  /storage/dynamic-multi/
합계 0
drwxrwxrwx 2 root root 21  8월 27 17:11 archived-default-pvc-log-pvc-246d1cf4-e25a-477b-9ec2-3acb494163ca
drwxrwxrwx 2 root root  6  8월 27 16:48 default-pvc-backup-pvc-bbeabf9d-03dc-40a6-97de-d164573815b4
drwxrwxrwx 2 root root  6  8월 27 16:48 default-pvc-image-pvc-0f2c6eb1-53fe-40ba-888f-d7e76bb53f3b

[root@k8s-master ~]# ls -l /storage/dynamic-multi/archived-default-pvc-log-pvc-246d1cf4-e25a-477b-9ec2-3acb494163ca/
합계 4
-rw-r--r-- 1 root root 26  8월 27 17:11 log.txt

[root@k8s-master ~]# kubectl  get  storageclasses sc-nfs-multi -o yaml
allowVolumeExpansion: true
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  annotations:
    meta.helm.sh/release-name: nfs-provisioner
    meta.helm.sh/release-namespace: default
  creationTimestamp: "2026-08-27T07:11:50Z"
  labels:
    app: nfs-subdir-external-provisioner
    app.kubernetes.io/managed-by: Helm
    chart: nfs-subdir-external-provisioner-4.0.18
    heritage: Helm
    release: nfs-provisioner
  name: sc-nfs-multi
  resourceVersion: "548672"
  uid: 9d88988d-9cfd-4c1e-acf9-39250a78ae4d
parameters:
  archiveOnDelete: "true"
provisioner: cluster.local/nfs-provisioner-nfs-subdir-external-provisioner
reclaimPolicy: Delete
volumeBindingMode: Immediate
```

`storageClass.archiveOnDelete`가 `true`이면 PV가 삭제될 때 NFS 서버에 있는 실제 데이터 디렉터리를 바로 삭제하지 않고 `archived-...` 형태로 보관한다. `archiveOnDelete`가 `true`로 되어있기 때문에 `pvc-log`의 PV는 삭제되지만 NFS Server의 디렉터리는 `archived-default-pvc-log-...` 형태로 보관되는 것을 확인할 수 있다.

**정리**: Dynamic Provisioning에서는 관리자가 PV를 미리 만들 필요 없이 StorageClass + Provisioner가 PVC 생성 시점에 PV를 자동으로 만들어주고, PVC를 삭제하면 reclaimPolicy(Delete/Retain)와 archiveOnDelete 설정에 따라 PV와 실제 데이터의 처리 방식이 결정된다.

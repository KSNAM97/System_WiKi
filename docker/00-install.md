# Docker 00 — Docker 설치

참고: https://docs.docker.com/ , https://docs.docker.com/desktop/setup/install/linux/

## Ubuntu에 Docker 설치

Docker 공식 설치 스크립트를 다운로드받아 관리자 권한으로 실행한다.

```bash
guest@Server-B:~$ sudo apt install -y curl

guest@Server-B:~$ curl -fsSL https://get.docker.com | sudo sh


guest@Server-B:~$ sudo systemctl  start  docker


Executing: /usr/lib/systemd/systemd-sysv-install enable docker


guest@Server-B:~$ sudo systemctl  status  docker
● docker.service - Docker Application Container Engine
     Loaded: loaded (/usr/lib/systemd/system/docker.service; enabled; preset: e>
     Active: active (running) since Wed 2026-08-05 09:46:21 KST; 1min 1s ago
TriggeredBy: ● docker.socket
       Docs: https://docs.docker.com
   Main PID: 2635 (dockerd)
      Tasks: 9
     Memory: 26.8M (peak: 29.1M)
        CPU: 302ms
     CGroup: /system.slice/docker.service
             └─2635 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/cont>


guest@Server-B:~$ sudo usermod  -aG  docker  guest

        OR

guest@Server-B:~$ sudo usermod  -aG  docker  $USER


guest@Server-B:~$ newgrp  docker            # 현재 로그인한 세션에 설정을 바로 적용


guest@Server-B:~$ docker run hello-world
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
4f55086f7dd0: Pull complete
d5e71e642bf5: Download complete
Digest: sha256:7f4da0fc94bcece205a8c0b6f4d11c8196924654ffe5c4d1aa439b7f632048b2
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
```

**정리**: Ubuntu에서는 `get.docker.com` 공식 스크립트로 설치 → `systemctl start docker` → `usermod -aG docker`로 sudo 없이 docker 명령을 쓸 수 있게 그룹을 추가 → `docker run hello-world`로 정상 동작을 확인하는 흐름이다. 이어서 Rocky Linux(RHEL 계열)에서의 설치 방법을 살펴본다.

---

## Rocky Linux에 Docker 설치

### 1) 필수 도구 설치 (yum-utils 포함)

`dnf-plugins-core`는 DNF 확장 기능(플러그인) 모음 패키지이다.

```bash
[root@docker ~]$ sudo dnf install -y yum-utils
```

### 2) Docker 공식 Repository 등록

Rocky Linux는 RHEL 계열의 Linux 배포판이므로 Docker의 RHEL Repository를 추가한다.

- Repository는 Linux에서 설치할 프로그램 패키지와 관련 정보를 보관하는 저장소이다.
- DNF는 시스템에 등록된 Repository를 검색하여 필요한 패키지를 내려받고 설치한다.
- Rocky Linux에는 기본 Repository가 등록되어 있지만, Docker의 공식 패키지인 `docker-ce`는 기본 Repository에서 제공되지 않거나 원하는 최신 버전이 없을 수 있다. 따라서 Docker CE 공식 버전을 설치하려면 Docker에서 제공하는 별도의 Repository를 추가하는 것이 일반적이다.

Docker 공식 Repository를 추가하면 다음과 같은 패키지를 설치할 수 있다.
- `docker-ce`
- `docker-ce-cli`
- `containerd.io`
- `docker-buildx-plugin`
- `docker-compose-plugin`

```bash
[root@docker ~]$ sudo dnf config-manager --add-repo https://download.docker.com/linux/rhel/docker-ce.repo
```

### 3) Docker Engine 설치

```bash
[guest@Server-A ~]$ sudo dnf install -y \
  docker-ce \
  docker-ce-cli \
  containerd.io \
  docker-buildx-plugin \
  docker-compose-plugin
```

- `docker-ce` : Docker Engine(Docker Daemon)을 제공하는 핵심 패키지
- `docker-ce-cli` : docker 명령어를 사용할 수 있도록 Docker CLI를 제공하는 패키지
- `containerd.io` : 컨테이너의 생성 및 실행 등을 담당하는 Container Runtime
- `docker-buildx-plugin` : Docker Image Build 기능을 확장하는 Buildx Plugin
- `docker-compose-plugin` : Docker Compose 명령을 사용할 수 있도록 제공하는 Plugin

### 4) 서비스 시작 및 활성화

```bash
[guest@Server-A ~]$ sudo systemctl  start  docker


[guest@Server-A ~]$ sudo systemctl  enable  docker
Created symlink /etc/systemd/system/multi-user.target.wants/docker.service → /usr/lib/systemd/system/docker.service.


[guest@Server-A ~]$ sudo systemctl  status  docker
● docker.service - Docker Application Container Engine
     Loaded: loaded (/usr/lib/systemd/system/docker.service; enabled; preset: d>
     Active: active (running) since Wed 2026-08-05 10:02:12 KST; 8s ago
TriggeredBy: ● docker.socket
       Docs: https://docs.docker.com
   Main PID: 3624 (dockerd)
      Tasks: 8
     Memory: 30.9M (peak: 31.4M)
        CPU: 169ms
     CGroup: /system.slice/docker.service
             └─3624 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/cont>
```

### 5) 일반 사용자로 docker 명령 사용하기

```bash
[guest@Server-A ~]$ sudo  usermod  -aG  docker  guest

        OR

[guest@Server-A ~]$ sudo  usermod  -aG  docker  $USER
```

### 6) 설치 확인

```bash
[guest@Server-A ~]$ docker  run  hello-world
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
4f55086f7dd0: Pull complete
d5e71e642bf5: Download complete
Digest: sha256:7f4da0fc94bcece205a8c0b6f4d11c8196924654ffe5c4d1aa439b7f632048b2
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
```

```bash
[guest@Server-A ~]$ docker image ls
IMAGE                 ID              DISK USAGE    CONTENT SIZE    EXTRA
hello-world:latest    7f4da0fc94bc    21.8kB        9.49kB          U


[guest@Server-A ~]$ docker  ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES


[guest@Server-A ~]$ docker  ps -a
CONTAINER ID    IMAGE         COMMAND     CREATED        STATUS                       PORTS     NAMES
9fc91aa2a575    hello-world   "/hello"    4 minutes ago  Exited (0) 4 minutes ago               heuristic_ritchie


[guest@Server-A ~]$ docker  rm  heuristic_ritchie
heuristic_ritchie


[guest@Server-A ~]$ docker  ps -a
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```

**정리**: Rocky Linux(RHEL 계열)에서는 `yum-utils` 설치 → Docker 공식 RHEL Repository 등록 → `docker-ce`/`docker-ce-cli`/`containerd.io`/`docker-buildx-plugin`/`docker-compose-plugin` 설치 → 서비스 시작/활성화 → 사용자를 `docker` 그룹에 추가 → `hello-world` 컨테이너로 정상 동작을 확인하고 `docker image ls`/`docker ps -a`/`docker rm`으로 이미지·컨테이너 상태까지 확인하는 순서로 설치를 마친다.

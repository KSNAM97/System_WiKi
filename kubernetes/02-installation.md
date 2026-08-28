# K8s-02 쿠버네티스 설치

## 쿠버네티스 설치 도구 개요

쿠버네티스 클러스터는 학습이나 소규모 환경에서는 kubeadm으로 각 구성 요소를 직접 설치하며 구조를 이해하는 방식으로, 기업의 대규모 환경에서는 kubespray로 HA 클러스터를 자동화 구성하는 방식으로 설치한다. 쿠버네티스 클러스터를 직접 구성할 때 사용하는 대표 도구인 **kubeadm**과 **kubespray**, 그리고 Pod 네트워크를 담당하는 **CNI**를 정리한다.

-쿠버네티스 클러스터를 직접 구성하는 도구 정리

-**kubeadm**
  - 쿠버네티스 프로젝트에서 공식으로 제공하는 클러스터 생성, 설치, 관리 도구
  - 최소한의 구성 요소만 설치해 주기 때문에 구조를 이해하면서 직접 클러스터를 구축하고 싶은 학습자에게 적합
  - 마스터 노드 초기화, 워커 노드 조인, 인증서 관리 등 기본 기능 제공
  - 클라우드 환경, 온프레미스 모두 사용 가능
  - 디버깅과 구조 이해 목적의 실습 환경에서 가장 많이 사용됨


-**kubespray**
  - 쿠버네티스 클러스터를 자동으로 배포해 주는 오픈소스 프로젝트
  - 다양한 OS(Ubuntu, CentOS, Rocky 등)를 지원하며, 설치 옵션을 매우 세밀하게 조정할 수 있음
  - 온프레미스 환경에서 상용 수준의 쿠버네티스 클러스터를 구성하거나
   여러 대의 서버에 복잡한 설정을 반복해야 할 때 유용
  - 고가용성(HA) 구성, 인증서 관리, 네트워크 플러그인(CNI) 설치까지 자동화됨
  - Calico, Flannel, Weave 등 다양한 CNI 플러그인을 선택하여 구성 가능
  - 기업 환경에서 자동화된 쿠버네티스 배포 도구로 많이 활용됨


-**CNI(Container Network Interface)**
  - 컨테이너와 쿠버네티스에서 네트워크를 사용할 수 있도록 만드는 표준 규칙(인터페이스)이다.
  - 쿠버네티스 안의 모든 컨테이너가 서로 통신할 수 있도록 네트워크를 만들어 주는 네트워크 플러그인 규격이다.
  - 쿠버네티스 자체는 네트워크 기능을 직접 제공하지 않는다.
  - 대신 CNI라는 규칙만 제공하고, 실제 네트워크 동작은 CNI 플러그인이 수행한다.


-CNI 종류
1) Calico
-전 세계에서 가장 널리 사용되는 CNI
-BGP 기반 라우팅
-NetworkPolicy 기능이 매우 강력
-대규모, 기업용, 온프레미스·클라우드 어디든 활용 가능

2) Flannel
-가장 기본적이고 단순한 CNI
-학습, 테스트 환경에서 압도적으로 많이 사용
-설정이 매우 쉬워 kubeadm 설치 직후 자주 선택됨
-NetworkPolicy는 기본 미지원

3) Cilium
-eBPF 기반의 최신 고성능 CNI
-최근 빠르게 점유율 증가
-성능, 보안·관찰(Observability)이 뛰어나 현대형 인프라에 적합

4) Weave Net
-설치가 쉬운 편이며, 자동 암호화 기능 제공
-중소규모 클러스터에서 자주 사용
-Pod 간 연결이 안정적이고 구성 단순

5) Canal
-Calico + Flannel 조합
-Flannel의 단순성 + Calico의 정책 기능을 동시에 사용 가능

**정리**: kubeadm/kubespray로 클러스터를 구성하는 방법과, Pod 네트워크를 담당하는 CNI 플러그인의 종류를 살펴보았다. 다음으로는 클러스터를 이루는 핵심 구성 요소(노드, 클러스터, 파드)를 알아본다.

---

## 쿠버네티스 클러스터 구성

### 1) 노드(Node)

-쿠버네티스에서 작업을 실제로 실행하는 컴퓨터(서버)를 의미한다.

-노드는 물리 서버일 수도 있고, VMware 같은 가상머신(VM)일 수도 있으며, AWS EC2와 같은 클라우드 가상 서버일 수도 있다.

-각 노드에는 CPU, Memory, Network, Disk 등의 자원이 존재하며 쿠버네티스는 이러한 자원을 이용하여 컨테이너를 실행한다.

-노드 = 쿠버네티스 클러스터를 구성하는 각각의 서버, 컨테이너(Pod)를 실행할 수 있는 컴퓨터

-쿠버네티스의 노드는 크게 두 종류로 구분한다.
  - **Control Plane Node(Master Node)**	: 전체 클러스터를 관리하는 지휘자
  - **Worker Node** 			: 실제 Pod와 컨테이너를 실행하는 작업 서버


#### Control Plane Node (Master Node)

-쿠버네티스 클러스터 전체를 관리하고 제어하는 노드이다.

-Pod를 어느 Worker Node에서 실행할지 결정하고, 각 노드와 Pod의 상태를 지속적으로 확인한다.

-사용자가 kubectl 명령을 실행하면 대부분의 요청은 Control Plane의 API Server를 통해 처리된다.

-주요 역할
  - 클러스터 전체 상태 관리
  - Worker Node 상태 확인
  - Pod가 실행될 Worker Node 결정(Scheduling)
  - 장애 발생 시 Pod 재생성 및 복구
  - Kubernetes API 요청 처리
  - 클러스터 설정 및 상태 정보 저장


#### Worker Node

-실제 애플리케이션의 Pod와 컨테이너가 실행되는 노드이다.
-Control Plane의 명령을 받아 필요한 Pod를 생성하거나 삭제한다.
-웹서버, WAS, API서버, 데이터처리 프로그램 등 실제 서비스용 애플리케이션은 일반적으로 Worker Node에서 실행된다.

-주요 역할
  - Pod 및 컨테이너 실행
  - CPU, Memory 등의 자원 제공
  - 서비스 트래픽 처리
  - Control Plane의 명령에 따라 Pod 생성/삭제
  - 컨테이너 런타임(containerd 등)을 이용하여 실제 컨테이너 실행


### 2) 클러스터(Cluster)

-여러 개의 Node를 하나의 시스템처럼 묶어서 사용하는 구조를 의미한다.
-즉, 클러스터 = 여러 Node를 하나의 팀으로 구성한 것
-쿠버네티스에서는 Control Plane과 여러 Worker Node를 하나의 Cluster로 구성한다.

-왜 여러 서버를 하나의 클러스터로 구성하는 이유

1. 장애 대응
  - 특정 Worker Node에 장애가 발생해도 다른 정상적인 Node에서 Pod를 다시 실행하도록 할 수 있다.
  - 따라서 서버 한 대의 장애가 전체 서비스 장애로 바로 이어지는 것을 줄일 수 있다.

2. 부하 분산
  - 사용자 요청이나 트래픽이 많을 경우 여러 Pod와 Node에 작업을 분산할 수 있다.
  - 한 대의 서버가 모든 요청을 처리하는 것보다 안정적으로 서비스를 운영할 수 있다.

3. 확장성(Scaling)
  - 서비스 사용량이 증가하면 Pod의 개수를 증가시키거나 Worker Node를 추가할 수 있다.
  - 반대로 사용량이 감소하면 필요에 따라 Pod 수를 줄일 수도 있다.

4. 자동 복구(Self-Healing)
  - 실행 중인 Pod에 장애가 발생하면 쿠버네티스가 이를 감지하고 새로운 Pod를 생성할 수 있다.
  - 사용자가 매번 직접 컨테이너를 다시 실행하지 않아도 된다.

5. 중앙 관리
  - 수십 대 또는 수백 대의 서버에서 각각 컨테이너를 직접 관리하는 것은 매우 어렵다.
  - 쿠버네티스는 여러 Node와 Pod를 하나의 클러스터 단위로 관리할 수 있게 해준다.


### 3) 파드(Pod)

-파드는 쿠버네티스에서 컨테이너를 실행하는 가장 작은 배포 및 실행 단위이다.

-쿠버네티스는 일반적으로 컨테이너를 직접 관리하는 것이 아니라 Pod라는 단위를 기준으로 관리한다.

-하나의 Pod에는 하나의 컨테이너가 들어가는 경우가 가장 일반적이지만, 필요에 따라 여러 개의 컨테이너가 함께 들어갈 수도 있다.

-컨테이너 	= 프로그램이 실행되는 각각의 박스
-Pod 		= 컨테이너를 담고 있는 작은 집

-같은 Pod 안에 있는 컨테이너들은 매우 밀접하게 동작하는 프로그램들로 구성하는 경우가 많다.

-Pod의 주요 특징
  - Pod에는 하나 이상의 컨테이너가 포함될 수 있다.
  - 같은 Pod 안의 컨테이너들은 동일한 네트워크 공간을 공유한다.
  - 같은 Pod의 컨테이너는 하나의 Pod IP를 공유한다.
  - 같은 Pod 안에서는 localhost를 이용하여 서로 통신할 수 있다.
  - 필요에 따라 Volume을 연결하여 데이터를 공유할 수 있다.
  - Pod는 Worker Node 위에서 실행된다.
  - Pod에 장애가 발생하면 Deployment 등의 Controller가 새로운 Pod를 생성할 수 있다.

-쿠버네티스는 Pod 단위로 다음과 같은 작업을 수행한다.
  - Pod 생성
  - Pod 삭제
  - Pod 배치
  - Pod 재시작 및 재생성
  - Pod 개수 확장/축소
  - Pod 네트워크 연결
  - Pod 상태 확인

-Worker Node 안에서 Pod가 실행되고 Pod 안에서 Container가 실행된다.

**정리**: 노드, 클러스터, 파드는 쿠버네티스의 가장 기본이 되는 구성 단위다. 이제 실제 실습 환경(Rocky Linux 9, 마스터 1대 + 워커 2대)에서 노드 기본 설정부터 시작한다.

---

## 0단계. 노드 기본 설정 (마스터, 워커 모두 실행)

### 0-1) hostname 설정

```bash
# k8s-master (192.168.10.100)
[root@localhost ~]# hostnamectl set-hostname k8s-master
[root@localhost ~]# exec bash

# k8s-worker1 (192.168.10.101)
[root@localhost ~]# hostnamectl set-hostname k8s-worker1
[root@localhost ~]# exec bash

# k8s-worker2 (192.168.10.102)
[root@localhost ~]# hostnamectl set-hostname k8s-worker2
[root@localhost ~]# exec bash
```

### 0-2) /etc/hosts 등록 (마스터, 워커 모두 실행)

-쿠버네티스 내부 통신에서 노드 이름 해석이 필요하다
-DNS 서버를 별도로 두지 않는 실습 환경에서는 /etc/hosts가 가장 단순하다.
-hostname  <--->  IP 매핑을 모든 노드에 동일하게 맞춰야 한다

```bash
cat <<EOF >> /etc/hosts
192.168.10.100  k8s-master
192.168.10.101  k8s-worker1
192.168.10.102  k8s-worker2
EOF
```

```
[root@k8s-master ~]# cat  /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
192.168.10.100  k8s-master
192.168.10.101  k8s-worker1
192.168.10.102  k8s-worker2

[root@k8s-worker1 ~]# cat  /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
192.168.10.100  k8s-master
192.168.10.101  k8s-worker1
192.168.10.102  k8s-worker2

[root@k8s-worker2 ~]# cat  /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
192.168.10.100  k8s-master
192.168.10.101  k8s-worker1
192.168.10.102  k8s-worker2
```

### 0-3) 방화벽 및 SELinux 비활성화 (마스터, 워커 모두 실행)

-학습 환경에서는 네트워크 차단이 가장 흔한 장애 원인이다
-CNI, kube-proxy, API Server 통신이 막히면 원인 파악이 어렵다.

```bash
[root@k8s-master ~]# vi  /etc/selinux/config
# SELINUX=disabled  로 변경 후 저장

[root@k8s-master ~]# systemctl stop firewalld
[root@k8s-master ~]# systemctl disable firewalld
[root@k8s-master ~]# reboot

# k8s-worker1, k8s-worker2도 동일하게 실행
```

**정리**: hostname, /etc/hosts, 방화벽/SELinux 설정까지 모든 노드에 공통으로 적용되는 기본 준비를 마쳤다. 아래 Docker 설치 항목은 참고용이며, 본격적인 쿠버네티스 설치는 이후 STEP 1부터 이어진다.

---

## Docker 설치 (선택적 참고)

### 1단계) 기존 Docker 관련 패키지 제거

기존에 설치했거나 기본 패키지가 남아 있을 수 있으므로 정리

```bash
[root@localhost ~]# dnf remove -y  docker  docker-client  docker-client-latest  docker-common  docker-latest  docker-latest-logrotate  docker-logrotate  docker-engine
```

### 2단계) 필수 도구 설치

Docker Repository를 추가하고 스토리지 관련 기능을 사용하기 위해 필요한 패키지를 설치하는 단계  
yum-utils는 Repository 관리 명령을 제공하며, device-mapper-persistent-data와 lvm2는 스토리지 관리에 필요한 기능을 제공한다.

```bash
[root@localhost ~]# dnf install -y  yum-utils  device-mapper-persistent-data  lvm2
```

### 3단계) Docker 공식 Repository 추가

Docker 공식 Repository를 시스템에 추가하는 명령어이다.  
dnf config-manager를 이용해 docker-ce.repo 파일을 등록하며, 이후 Docker CE 관련 패키지를 dnf로 설치할 수 있게 된다.  
등록된 Repository 정보는 일반적으로 /etc/yum.repos.d/ 아래에 저장된다.

```bash
[root@localhost ~]# dnf config-manager --add-repo https://download.docker.com/linux/rhel/docker-ce.repo
```

### 4단계) Docker Engine 설치

Docker Engine과 Docker 명령어 도구, containerd, Buildx, Compose 플러그인을 한 번에 설치하는 명령어

```bash
# dnf install -y  docker-ce  docker-ce-cli  containerd.io  docker-buildx-plugin  docker-compose-plugin

# 서비스 시작
# systemctl  start  docker

# 서버 재부팅시 도커 자동 실행
# systemctl  enable  docker
Created symlink /etc/systemd/system/multi-user.target.wants/docker.service → /usr/lib/systemd/system/docker.service.

# 도커 상태 확인
# systemctl  status  docker
```

### 5단계) Docker 사용 테스트

```bash
[root@localhost ~]# docker version
[root@localhost ~]# docker info
```

정상이라면 Version 정보가 출력된다.

### 6단계) 컨테이너 테스트

```bash
[root@localhost ~]# docker run hello-world

[root@localhost ~]# docker images

[root@localhost ~]# docker ps -a
CONTAINER ID   IMAGE         COMMAND    CREATED         STATUS                   PORTS   NAMES
88fa307d0f50   hello-world   "/hello"   29 seconds ago  Exited (0) 28 seconds            festive_hermann
```

### 7단계) 일반 사용자 권한으로 실행

학생 실습용으로는 docker 그룹 권한을 주면 편하다.

```bash
[root@localhost ~]# usermod -aG docker guest
```

**정리**: Docker 설치는 쿠버네티스 설치에 필수는 아니며 참고용이다. 이제부터는 **kubeadm**을 이용해 Rocky Linux 9 환경에서 실제 쿠버네티스 클러스터를 구축하는 본 과정을 시작한다.

---

## Kubernetes 설치 (Rocky Linux 9 kubeadm)

참고: https://kubernetes.io/ → Documentation → Set up a K8s cluster (Install the kubeadm setup tool)

---

## STEP 1. 모든 노드 공통 사전 준비 (마스터, 워커 모두 실행)

### 1-1) hostname 설정

-쿠버네티스는 노드를 **hostname** 기준으로 식별한다
-로그, 노드 관리, 장애 분석 시 IP보다 hostname이 훨씬 직관적이다.
-클러스터에서 이 노드가 마스터인지 워커인지 바로 구분하기 위함이다.

```bash
# K8s Control-plane
[root@localhost ~]# hostnamectl set-hostname k8s-master
[root@localhost ~]# exec bash

# K8s worker-node1
[root@localhost ~]# hostnamectl set-hostname k8s-worker1
[root@localhost ~]# exec bash

# K8s worker-node2
[root@localhost ~]# hostnamectl set-hostname k8s-worker2
[root@localhost ~]# exec bash
```

### 1-2) /etc/hosts 등록 (마스터, 워커 모두 실행)

-쿠버네티스 내부 통신에서 노드 이름 해석이 필요하다
-DNS 서버를 별도로 두지 않는 실습 환경에서는 /etc/hosts가 가장 단순하다.
-hostname  <--->  IP 매핑을 모든 노드에 동일하게 맞춰야 한다

```bash
cat <<EOF >> /etc/hosts
192.168.10.100  k8s-master
192.168.10.101  k8s-worker1
192.168.10.102  k8s-worker2
EOF
```

```
[root@k8s-master ~]# cat /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
192.168.10.100  k8s-master
192.168.10.201  k8s-worker1
192.168.10.202  k8s-worker2

[root@k8s-worker1 ~]# cat /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
192.168.10.100  k8s-master
192.168.10.201  k8s-worker1
192.168.10.202  k8s-worker2

[root@k8s-worker2 ~]# cat /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
192.168.10.100  k8s-master
192.168.10.201  k8s-worker1
192.168.10.202  k8s-worker2
```

### 1-3. 시스템 업데이트 및 재부팅 (마스터, 워커 모두 실행)

-커널, 네트워크, 컨테이너 관련 패키지가 최신 상태여야 충돌이 없다
-kubeadm 설치 중 의존성 오류를 방지

```bash
dnf update -y
reboot
```

### 1-4) Swap 완전 제거 (마스터, 워커 모두 실행)

-**kubelet**은 기본 설정에서 Swap이 활성화되어 있으면 정상적으로 동작하지 않을 수 있으므로 모든 노드에서 Swap을 비활성화 한다.
-swapoff -a 명령으로 현재 사용 중인 Swap을 즉시 해제하고, /etc/fstab의 Swap 설정도 제거하여 재부팅 후 다시 활성화되지 않도록 한다.
-Kubernetes 클러스터를 안정적으로 구성하기 위한 공통 사전 설정 단계이다.

```bash
# swap 메모리 확인
[root@k8s-master ~]# swapon  --show
NAME      TYPE       SIZE USED PRIO
/dev/sda1  partition   4G    280K  -2

# swap 메모리 임시 비활성화
[root@k8s-master ~]# swapoff  -a

[root@k8s-master ~]# wipefs  -a /dev/sda1
/dev/sda1: 10 bytes were erased at offset 0x00000ff6 (swap): 53 57 41 50 53 50 41 43 45 32

# swap 메모리가 확인되지 않는다.
[root@k8s-master ~]# swapon  --show
```

/etc/fstab에서 swap 항목 주석 처리:

```
#
# /etc/fstab
# Created by anaconda on Thu Jul  2 03:51:29 2026
#
UUID=73bc277c-741d-4122-9c58-59ccd1889709 /               xfs     defaults        0 0
#UUID=520bc18c-2b64-4df1-85e0-d126908ba6dd none     swap    defaults        0 0
```

k8s-worker1:

```bash
[root@k8s-worker1 ~]# swapon --show
NAME      TYPE      SIZE USED PRIO
/dev/dm-1 partition 2.3G   0B   -2

[root@k8s-worker1 ~]# swapoff  -a

[root@k8s-worker1 ~]# wipefs  -a /dev/dm-1
/dev/dm-1: 10 bytes were erased at offset 0x00000ff6 (swap): 53 57 41 50 53 50 41 43 45 32
```

/etc/fstab (worker1):

```
/dev/mapper/rl-root     /                       xfs     defaults        0 0
UUID=f608e67e-5ec6-4715-b5ab-f79831ba1fc4 /boot           xfs     defaults        0 0
/dev/mapper/rl-home     /home                   xfs     defaults        0 0
#/dev/mapper/rl-swap     none                    swap    defaults        0 0
```

k8s-worker2:

```bash
[root@k8s-worker2 ~]# swapon --show
NAME      TYPE      SIZE USED PRIO
/dev/dm-1 partition 2.3G   0B   -2

[root@k8s-worker2 ~]# swapoff -a

[root@k8s-worker2 ~]# wipefs  -a /dev/dm-1
/dev/dm-1: 10 bytes were erased at offset 0x00000ff6 (swap): 53 57 41 50 53 50 41 43 45 32
```

/etc/fstab (worker2):

```
/dev/mapper/rl-root     /                       xfs     defaults        0 0
UUID=bb2182a5-9e31-4f54-848e-5dfb9275bab3 /boot           xfs     defaults        0 0
/dev/mapper/rl-home     /home                   xfs     defaults        0 0
# /dev/mapper/rl-swap     none                    swap    defaults        0 0
```

### 1-5) 방화벽 및 SELinux 비활성화 (마스터, 워커 모두 실행)

-학습 환경에서는 네트워크 차단이 가장 흔한 장애 원인이다
-CNI, kube-proxy, API Server 통신이 막히면 원인 파악이 어렵다

```bash
[root@k8s-master ~]# systemctl stop firewalld
[root@k8s-master ~]# systemctl disable firewalld
Removed "/etc/systemd/system/multi-user.target.wants/firewalld.service".
Removed "/etc/systemd/system/dbus-org.fedoraproject.FirewallD1.service".

[root@k8s-master ~]# setenforce 0
[root@k8s-master ~]# sed -i 's/^SELINUX=enforcing/SELINUX=disabled/' /etc/selinux/config

[root@k8s-master ~]# cat /etc/selinux/config
SELINUX=disabled
# SELINUXTYPE= can take one of these three values:
#     targeted - Targeted processes are protected,
#     minimum - Modification of targeted policy. Only selected processes are protected.
#     mls - Multi Level Security protection.
SELINUXTYPE=targeted

[root@k8s-worker1 ~]# systemctl stop firewalld
[root@k8s-worker1 ~]# systemctl disable firewalld
Removed "/etc/systemd/system/multi-user.target.wants/firewalld.service".
Removed "/etc/systemd/system/dbus-org.fedoraproject.FirewallD1.service".

[root@k8s-worker1 ~]# setenforce 0
[root@k8s-worker1 ~]# sed -i 's/^SELINUX=enforcing/SELINUX=disabled/' /etc/selinux/config

[root@k8s-worker1 ~]# cat /etc/selinux/config
SELINUX=disabled
# SELINUXTYPE= can take one of these three values:
#     targeted - Targeted processes are protected,
#     minimum - Modification of targeted policy. Only selected processes are protected.
#     mls - Multi Level Security protection.
SELINUXTYPE=targeted

# k8s-worker2도 설정
```

### 1-6) 커널 모듈 및 네트워크 설정 (마스터, 워커 모두 실행)

-Kubernetes에서는 Pod들이 서로 통신할 수 있도록 Linux 커널의 네트워크 기능을 사용한다.

-Pod의 네트워크는 CNI 플러그인이 구성, 이때 Bridge를 통과하는 패킷도 Linux의 Netfilter가 검사할 수 있어야 한다.

-**Netfilter**
  - Linux 커널에서 네트워크 패킷을 검사하고 처리하는 기능

-**br_netfilter**
  - Linux Bridge를 통과하는 패킷도 Netfilter가 처리할 수 있도록 해주는 커널 모듈

-**iptables**
  - Netfilter에 패킷 처리 규칙을 설정하는 명령어 도구

```bash
# 리눅스 커널에 bridge network도 검사해달라고 요청
[root@k8s-master ~]# modprobe br_netfilter

# modprobe는 리눅스가 재부팅하게되면 삭제되기 때문에 재부팅후에도 로딩되도록 설정
[root@k8s-master ~]# cat <<EOF | tee /etc/modules-load.d/k8s.conf
br_netfilter
EOF

# 시스템에 정의된 모든 sysctl 설정 파일을 전부 읽어서 커널에 적용
[root@k8s-master ~]# sysctl --system

# k8s-worker1, k8s-worker2에서도 동일하게 설정
```

**정리**: 모든 노드 공통 사전 준비(hostname, /etc/hosts, 시스템 업데이트, swap 제거, 방화벽/SELinux, 커널 모듈)를 마쳤다. 다음은 컨테이너 런타임인 containerd 설치 단계다.

---

## STEP 2. containerd 설치 (모든 노드)

### 2-1) containerd 설치

-Kubernetes는 Pod 안의 컨테이너를 실제로 실행하기 위해 컨테이너 런타임(Container Runtime)이 필요하다.

-Kubernetes 자체가 컨테이너를 직접 생성하거나 실행하는 것은 아니며, kubelet이 **CRI(Container Runtime Interface)**를 이용하여 컨테이너 런타임과 통신한다.

-CRI는 Kubernetes와 컨테이너 런타임 사이의 표준 통신 인터페이스이다.
 즉, kubelet이 특정 런타임에 직접 종속되지 않고 표준 방식으로 컨테이너 실행을 요청할 수 있게 해준다.

-**containerd**는 Kubernetes에서 많이 사용하는 대표적인 컨테이너 런타임이다.

-kubelet은 API Server에서 자신의 Node에 배치된 Pod 정보를 확인한 뒤, CRI를 통해 containerd에 컨테이너 실행을 요청한다.

-containerd는 요청을 받아 다음과 같은 실제 작업을 수행한다.
  - 컨테이너 이미지 다운로드(Pull)
  - 컨테이너 생성
  - 컨테이너 시작 및 종료
  - 컨테이너 상태 관리
  - 이미지 및 컨테이너 관리

### 2-2) Docker repo 추가 (마스터, 워커 모두 실행)

-Rocky Linux 9 기본 저장소에는 containerd 패키지가 없다
-Docker 공식 repo를 사용하면 안정적인 containerd.io 설치 가능

-repo는 프로그램을 보관해 두는 공식 다운로드 창고다.
-리눅스는 인터넷에서 아무 파일이나 받아서 설치하지 않는다.
 대신 믿을 수 있는 창고(repo)에서만 프로그램을 가져온다.

-윈도우 방식
  - 홈페이지 접속
  - exe 다운로드
  - 설치

-리눅스 방식
  - repo에 있는지 확인
  - 있으면 바로 설치
  - 없으면 설치 불가
  - 이유
    * 보안
    * 버전 관리
    * 의존성 자동 해결

-repo 없이 설치하려면 파일 직접 다운로드, 라이브러리 하나씩 수동 설치, 충돌 나면 직접 해결해야 한다.(의존성 문제)

-repo 안의 정보
  - 프로그램 파일
  - 필요한 라이브러리
  - 버전 정보
  - 의존성 정보등이 전부 들어 있다.

그래서 dnf install nginx 이 한 줄로 nginx + 필요한 모든 것 전부 같이 설치된다.

-repo의 위치 (Rocky / RHEL 계열)
-repo 설정 파일 위치 = /etc/yum.repos.d/
-여기에 *.repo 파일이 있으면 dnf가 자동으로 읽는다.

-Rocky 기본 repo에는 최신 containerd 없다.
 그래서 Docker가 직접 관리하는 공식 repo를 추가해야 한다.

#### Docker CE Repository를 수동으로 등록

```bash
[root@k8s-master ~]# cat << 'EOF' > /etc/yum.repos.d/docker-ce.repo
[docker-ce-stable]
name=Docker CE Stable - $basearch
baseurl=https://download.docker.com/linux/centos/9/x86_64/stable
enabled=1
gpgcheck=0
EOF

# k8s-worker1, k8s-worker2에서도 동일하게 설정
```

### 2-3). containerd 설치 및 기본 설정 (마스터, 워커 모두 실행)

-Kubernetes는 Pod 안의 컨테이너를 직접 실행하지 않기 때문에 컨테이너 런타임(Container Runtime)이 필요하다.
-containerd는 Kubernetes에서 사용할 수 있는 대표적인 컨테이너 런타임으로, 실제 컨테이너의 생성, 실행, 종료와 이미지 관리를 담당한다.
-kubelet은 CRI(Container Runtime Interface)를 통해 containerd에 컨테이너 실행을 요청한다.
 따라서 모든 Kubernetes Node에서 containerd가 정상적으로 설치되고 실행되어야 한다.

```bash
# containerd 설정 파일을 저장할 디렉터리 생성
[root@k8s-master ~]# dnf makecache
[root@k8s-master ~]# dnf install -y containerd.io

[root@k8s-master ~]# mkdir -p /etc/containerd

# containerd가 제공하는 기본 설정 예제를 출력해서 설정 파일로 저장
[root@k8s-master ~]# containerd config default > /etc/containerd/config.toml

# k8s-worker1, k8s-worker2에서도 동일하게 설정
```

### 2-4. containerd 핵심 설정 수정 (마스터, 워커 모두 실행)

-이 설정이 틀리면 kubelet이 containerd와 연결되지 않는다
-쿠버네티스 실패 원인 1순위

```bash
[root@k8s-master ~]# vi /etc/containerd/config.toml
```

수정 사항:
- `disabled_plugins` 안에 `"cri"` 제거
  - CRI는 kubelet과 통신하기 위한 인터페이스
  - 비활성화되어 있으면 kubelet이 런타임을 찾지 못한다.

- `SystemdCgroup = true` 설정
  - cgroup: Linux에서 CPU, Memory, Process 등의 자원을 그룹별로 제한하고 관리하는 기능
  - containerd가 cgroup을 직접 관리하지 않고 systemd를 이용해서 관리하도록 설정

```bash
[root@k8s-master ~]# systemctl restart containerd
[root@k8s-master ~]# systemctl enable containerd

# k8s-worker1, k8s-worker2에서도 동일하게 설정
```

**정리**: 모든 노드에 containerd 설치와 CRI/SystemdCgroup 설정까지 완료했다. 다음으로 쿠버네티스 핵심 도구인 kubeadm, kubelet, kubectl을 설치한다.

---

## STEP 3. kubeadm / kubelet / kubectl 설치

### 3-1. Kubernetes repo 추가 (마스터, 워커 모두 실행)

-우리는 지금 쿠버네티스를 직접 설치해야 한다.
 그런데 쿠버네티스 프로그램은 리눅스 기본 저장소에 들어 있지 않다.

dnf install kubeadm / dnf install kubelet / dnf install kubectl을 그대로 실행해도 어디서 가져올지 모르는 상태다.

그래서 먼저 쿠버네티스 공식 프로그램 창고(repo)의 위치를 dnf에게 알려줘야한다.
이 작업이 바로 Kubernetes repo 추가다.

-kubeadm / kubelet / kubectl
  - **kubeadm**	: 쿠버네티스 클러스터를 만드는 도구
  - **kubelet**	: 각 노드에서 항상 실행되는 에이전트
  - **kubectl**	: 쿠버네티스를 조작하는 명령어

-이 3개는 쿠버네티스 공식 구성 요소이므로 공식 repo에서만 설치하는 게 원칙이다.

```bash
[root@k8s-master ~]# cat <<EOF | tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://pkgs.k8s.io/core:/stable:/v1.35/rpm/
enabled=1
gpgcheck=1
gpgkey=https://pkgs.k8s.io/core:/stable:/v1.35/rpm/repodata/repomd.xml.key
EOF

# k8s-worker1, k8s-worker2에서도 동일하게 설정
```

### 3-2. 패키지 설치 및 kubelet 활성화

```bash
[root@k8s-master ~]# dnf install -y kubelet  kubeadm  kubectl

[root@k8s-master ~]# systemctl start kubelet
[root@k8s-master ~]# systemctl enable kubelet

# k8s-worker1, k8s-worker2에서도 동일하게 설정
```

**정리**: 모든 노드에 kubeadm, kubelet, kubectl 설치와 kubelet 활성화를 마쳤다. 이제 마스터 노드에서 클러스터를 초기화하는 단계로 넘어간다.

---

## STEP 4. 마스터 노드 초기화 (마스터만)

### 4-1) 이전 설정 초기화

-kubeadm init 또는 kubeadm join으로 구성했던 해당 노드의 Kubernetes 설정을 초기화한다.
 (kubeadm init 실패 흔적이 남아 있으면 재시도 불가)
-인증서, kubelet 설정 등 kubeadm이 생성한 클러스터 관련 설정을 정리한다.
- -f는 사용자 확인 질문 없이 강제로 reset을 진행한다.

```bash
[root@k8s-master ~]# kubeadm reset -f
```

기존에 설치된 CNI(Container Network Interface)의 네트워크 설정 파일을 삭제한다.  
Flannel, Calico 등의 기존 CNI 설정이 남아서 재설치 시 충돌하는 것을 방지하기 위한 작업이다.

```bash
[root@k8s-master ~]# rm -rf /etc/cni/net.d
```

Kubernetes에서 컨테이너를 실행하는 Container Runtime인 containerd를 재시작한다.

```bash
[root@k8s-master ~]# systemctl restart containerd
```

Kubernetes의 Node Agent인 kubelet을 재시작한다.

```bash
[root@k8s-master ~]# systemctl restart kubelet
```

kubeadm init은 현재 노드를 Kubernetes Control Plane으로 초기화하고 새로운 클러스터를 생성하는 명령이다.  
API Server, Scheduler, Controller Manager, etcd 등 Control Plane의 핵심 구성 요소를 설정한다.  
`--pod-network-cidr=10.244.0.0/16`은 클러스터에서 Pod들이 사용할 Pod 전용 IP 주소 대역을 지정한다.  
10.244.0.0/16은 현재 실습에서 사용하는 Flannel CNI에 맞춘 Pod 네트워크 대역이다.

```bash
[root@k8s-worker1 ~]# kubeadm init --pod-network-cidr=10.244.0.0/16
I0811 15:06:11.431410    9851 version.go:260] remote version is much newer: v1.36.3; falling back to: stable-1.35
[init] Using Kubernetes version: v1.35.7
[preflight] Running pre-flight checks
~~~~~~~~~~~~~~~~~ 중간 생략 ~~~~~~~~~~~~~~~~~
[addons] Applied essential addon: kube-proxy

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.10.101:6443 --token cr55qt.19pkewhugq2lg8ew \
        --discovery-token-ca-cert-hash sha256:51e6c74b0fad20caf6701598f99ea268629b77750abd141ce241cc82299b0b00
```

**정리**: kubeadm init으로 마스터 노드의 Control Plane 초기화를 완료했다. 다음은 이 마스터에서 kubectl을 사용할 수 있도록 kubeconfig를 설정하는 단계다.

---

## STEP 5. kubectl 설정 (마스터만)

-**kubectl**은 쿠버네티스를 직접 조작하는 리모컨이다.

-Kubernetes Master(Control Plane)에서 kubeadm init을 완료한 후, kubectl 명령어로 클러스터를 관리하려면 설정 파일을 준비해야 한다.

-그 연결 정보가 들어 있는 파일이 kubeconfig 파일이다.

-kubectl은 기본적으로 설정 파일을 이 위치에서 찾는다.
-위치 : ~/.kube/config
-이 폴더가 없으면  kubectl이 어디로 접속해야 할지 모른다.

```bash
[root@k8s-master ~]# mkdir -p $HOME/.kube
[root@k8s-master ~]# cp /etc/kubernetes/admin.conf $HOME/.kube/config
```

-cp /etc/kubernetes/admin.conf $HOME/.kube/config
  - kubeadm이 만들어 준 관리자 설정 파일을 kubectl이 기본으로 읽는 위치로 복사
  - kubectl은 기본적으로 $HOME/.kube/config 이 파일을 자동으로 읽는다.

```bash
[root@k8s-master ~]# ls -la .kube/
합계 12
drwxr-xr-x   2 root root   20  8월 11 15:35 .
dr-xr-x---. 15 root root 4096  8월 11 15:34 ..
-rw-------   1 root root 5638  8월 11 15:35 config
```

guest에서도 kubectl을 사용하려면:

```bash
[root@k8s-master ~]# mkdir -p /home/guest/.kube
[root@k8s-master ~]# cp /etc/kubernetes/admin.conf /home/guest/.kube/config
[root@k8s-master ~]# chown -R guest:guest /home/guest/.kube

[root@k8s-master ~]# ls -la /home/guest/.kube/
합계 12
drwxr-xr-x   2 guest guest   20  8월 11 15:35 .
drwx------. 15 guest guest 4096  8월 11 15:35 ..
-rw-------   1 guest guest 5638  8월 11 15:35 config
```

```bash
[root@k8s-master ~]# kubectl get nodes
NAME         STATUS     ROLES          AGE   VERSION
k8s-master   NotReady   control-plane  18m   v1.30.14
```

**정리**: kubeconfig 설정으로 root와 guest 계정 모두 kubectl로 클러스터에 접근할 수 있게 되었다. 노드 상태가 아직 NotReady인 것은 Pod 네트워크(CNI)가 아직 설치되지 않았기 때문이며, 다음 단계에서 이를 해결한다.

---

## STEP 6. CNI(Flannel) 설치 (컨트롤 플레인만)

-쿠버네티스에서 Pod는 각각 자기 IP를 가지고 서로 직접 통신해야 한다.
 그런데 문제는 쿠버네티스는 Pod 네트워크를 직접 만들어주지 않는다.
-Pod끼리 어떻게 연결할지는 각각 알아서 정해라 이게 쿠버네티스 기본 철학이다.
 그래서 필요한 게 CNI다.

-**CNI = Container Network Interface**
  - Pod 네트워크를 실제로 만들어주는 플러그인

-CNI 종류
  - Flannel
  - Calico
  - Weave
  - Cilium

-그중 Flannel은 설정이 가장 단순해서 실습에 많이 사용된다.

```bash
[root@k8s-master ~]# kubectl apply -f https://raw.githubusercontent.com/flannel-io/flannel/master/Documentation/kube-flannel.yml
namespace/kube-flannel created
clusterrole.rbac.authorization.k8s.io/flannel created
clusterrolebinding.rbac.authorization.k8s.io/flannel created
serviceaccount/flannel created
configmap/kube-flannel-cfg created
daemonset.apps/kube-flannel-ds created

[root@k8s-master ~]# kubectl  get  nodes
NAME         STATUS   ROLES           AGE   VERSION
k8s-master   Ready    control-plane   21m   v1.35.7
```

**정리**: Flannel CNI를 설치하고 나서 마스터 노드가 Ready 상태로 바뀌었다. 이제 워커 노드를 클러스터에 조인시키는 마지막 단계가 남았다.

---

## STEP 7. 워커 노드 조인

-마스터(control plane)는 준비 완료 하지만 혼자서만 쿠버네티스가 돌고 있다.

-워커 노드는 Pod를 실제로 실행하는 일꾼 서버다.
-이 일꾼을 마스터가 관리하는 클러스터에 등록시켜야 한다.
-이 등록 과정이 바로 join이다.

```bash
# join 토큰 재발급
[root@k8s-master ~]# kubeadm  token  create  --print-join-command
kubeadm join 192.168.10.100:6443 --token 177z1b.lj5pkgk0aluwxdtd --discovery-token-ca-cert-hash sha256:c0bfb5f54fef86b5a223485afd5772d330953c94c6aabec98740eb9456241bf2

# k8s-worker1 워커 노드를 control plane(master)에 등록
[root@k8s-worker1 ~]# kubeadm join 192.168.10.100:6443 --token 177z1b.lj5pkgk0aluwxdtd --discovery-token-ca-cert-hash sha256:c0bfb5f54fef86b5a223485afd5772d330953c94c6aabec98740eb9456241bf2

# k8s-worker2 워커 노드를 control plane(master)에 등록
[root@k8s-worker2 ~]# kubeadm join 192.168.10.100:6443 --token 177z1b.lj5pkgk0aluwxdtd --discovery-token-ca-cert-hash sha256:c0bfb5f54fef86b5a223485afd5772d330953c94c6aabec98740eb9456241bf2

[root@k8s-master ~]# kubectl  get  nodes
NAME          STATUS   ROLES           AGE   VERSION
k8s-master    Ready    control-plane   25m   v1.35.7
k8s-worker1   Ready    <none>          86s   v1.35.7
k8s-worker2   Ready    <none>          33s   v1.35.7
```

**정리**: kubeadm join으로 워커 노드 2대를 클러스터에 등록하면서 마스터 1대 + 워커 2대 구성의 쿠버네티스 클러스터 구축이 완료되었다. 이하는 kubectl 사용 편의 설정과 명령어 구조에 대한 참고 내용이다.

---

## kubectl bash completion 설정

```bash
source <(kubectl completion bash)
echo "source <(kubectl completion bash)" >> ~/.bashrc
```

참고: https://kubernetes.io/ → search : kubectl cheat sheet

---

## kubectl

-**kubectl**은 쿠버네티스(Kubernetes) 클러스터를 관리하기 위한 명령어 도구(Command Line Tool)이다.

-쉽게 말하면
  - 쿠버네티스에게 명령을 전달하는 리모컨
  - 쿠버네티스 API 서버에 요청을 보내는 통역사
  - 쿠버네티스 리소스를 만들고 관리할 수 있는 도구

-kubectl --> Kubernetes API Server  -->  노드/파드/서비스에 반영

-kubectl이 없으면 Pod 만들기, Deployment 배포, 서비스 연결, 노드 확인, 클러스터 관리 등 아무 것도 할 수 없다.

### kubectl이 동작하는 구조

1) 사용자가 명령 실행  
예: kubectl get pods

2) kubectl은 kubeconfig 파일을 읽는다.  
이 파일에 다음 정보가 들어 있다.
  - 쿠버네티스 API 서버 주소 (https://192.168.10.100:6443)
  - 인증 정보(토큰, 인증서 등)
  - 어떤 클러스터에 연결할지
  - 어떤 사용자 권한으로 실행할지

3) API 서버에 요청 전송  
  - kubectl --> API Server 에 HTTP/HTTPS 요청을 보냄.

4) API 서버가 실행 중인 클러스터 상태를 확인하여 응답

5) kubectl은 결과를 사용자에게 출력

-구조 : 사용자 명령 --> kubectl --> API Server --> 클러스터 상태 확인 --> 결과 반환

### kubeconfig

-**kubeconfig**는 kubectl이 쿠버네티스와 통신하는데 필요한 설정 파일

-위치
  - 리눅스 root 계정: /root/.kube/config
  - 일반 사용자: /home/USER/.kube/config
  - 이 파일이 있어야만 kubectl은 올바르게 동작한다.
  - 없으면 다음 오류가 뜬다 : The connection to the server localhost:8080 was refused
  - 즉 kubectl이 "API 서버 주소를 못 찾아서" localhost로 접속하기 때문이다.

### kubectl 기본 구조(명령 형식)

-kubectl 명령은 기본적으로 다음과 같은 형태를 가진다.

```
kubectl  [Command]  [Type]  [Name]  [Flags]
```

#### Command (명령, 동작: 무엇을 할 것인가?)

Command는 kubectl이 무엇을 해야 하는지를 의미한다.
  - get (조회) 		: EX) kubectl get pods
  - describe (상세 정보)	: EX) kubectl describe node k8s-worker1
  - create (생성)		: EX) kubectl create deployment web --image=nginx
  - delete (삭제)		: EX) kubectl delete pod web-123
  - logs (로그 확인)	: EX) kubectl logs web-pod
  - exec 		: Pod 내부 명령 실행
  - apply (YAML로 생성/수정) EX) kubectl apply -f web.yaml

#### Type (리소스 종류: 무엇을 대상으로 하는가?)

Type은 kubectl이 어떤 쿠버네티스 리소스를 대상으로 작업할지 지정하는 요소다.
즉, Command는 무엇을 할지, Type은 무엇을 대상으로 할지를 의미한다.

-pods
  - 쿠버네티스에서 실행되는 가장 작은 배포 단위
  - 컨테이너 1개 또는 여러 개가 들어있는 실행 단위
  - 예: kubectl get pods

-deployment
  - Pod를 관리하고 자동 복구, 자동 롤링업데이트를 제공하는 상위 컨트롤러
  - 실제 운영 환경에서 가장 많이 사용되는 리소스
  - 예: kubectl get deployment

-node
  - 쿠버네티스 클러스터에 참여하는 물리/가상 서버
  - Pod가 배치되는 실제 워커 머신
  - 예: kubectl get nodes

-namespace (ns)
  - 클러스터 안에서 리소스를 구분하는 논리적인 공간
  - 개발/운영/테스트 등 환경 분리 용도로 사용
  - 예: kubectl get ns

#### Name (대상 이름: 어떤 리소스를 정확히 지정하는가?)

Name은 쿠버네티스 리소스 하나를 특정하기 위한 고유 이름을 의미한다.
즉, Type이 무엇을 대상으로 할지라면 Name은 그 대상 중 어떤 것을 정확히 가리킬지를 의미한다.

```bash
# 예: kubectl describe pod web-6d4ff56789-abcde
# pod라는 Type 중에서도 'web-6d4ff56789-abcde'라는 특정 Pod 1개만 조회

kubectl delete service web-service
# service 중에서 'web-service'라는 Service만 삭제
```

#### Flags (추가 옵션)

Flags는 옵션이다.
어떻게 동작할지 설정하는 추가 옵션

대표적인 Flags:
  - -o wide : 더 많은 정보를 출력
  - EX) kubectl get pods -o wide

  - -n NAMESPACE : 특정 네임스페이스 조회
  - EX) kubectl get pods -n kube-system

  - -f FILE : YAML 파일을 지정
  - EX) kubectl apply -f web.yaml

**정리**: kubectl 명령은 `[Command] [Type] [Name] [Flags]` 구조로 이루어지며, kubeconfig 파일을 통해 API 서버와 통신한다는 점을 이해하면 이후 다양한 리소스 조작 명령을 응용할 수 있다.

---

## 기타 유용한 명령

```bash
# CLI로 즉시 전환
systemctl isolate multi-user.target

# CLI를 기본값으로 변경
systemctl set-default multi-user.target
```

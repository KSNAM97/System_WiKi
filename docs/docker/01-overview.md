# Docker 01 — 도커, 컨테이너의 이해

## 도커 (Docker)

- **도커(Docker)**는 애플리케이션을 **컨테이너(Container)**라는 격리된 환경에서 실행할 수 있도록 도와주는 오픈소스 컨테이너 플랫폼이다. 개발 환경과 운영 환경이 달라 생기는 문제를 줄이고, 여러 서버에 동일한 실행 환경을 빠르게 복제하거나 하나의 서버에서 여러 서비스를 격리해 운영하려는 목적으로 널리 쓰인다.
- 2013년에 Solomon Hykes에 의해 공개되었으며 Go 언어로 개발되었다.
- 도커의 핵심 목적은 애플리케이션과 실행에 필요한 환경을 함께 패키징하여 개발/테스트/운영 환경의 차이를 줄이고 어디서 실행하더라도 동일한 실행 환경을 쉽게 재현할 수 있도록 하는 것이다.

### 도커를 이해하기 위한 해상 컨테이너 비유

- 해상 운송을 생각해보자.
- 모든 화물은 표준 규격의 컨테이너 박스에 담겨 운송된다.
- 컨테이너 안에 TV가 들어 있든, 옷이 들어 있든 운송 방식은 동일하다.

컨테이너의 특징
- 크기와 규격이 표준화되어 있다.
- 운송업자는 내부의 화물에 따라 운송 방식을 바꿀 필요가 없다.
- 선박, 트럭, 기차 등 다양한 운송 환경에서 동일한 방식으로 운반할 수 있다.
- 컨테이너 단위로 싣고, 내리고, 보관할 수 있다.

만약 컨테이너가 없다면
- 물건마다 다른 운송 방법 필요
- 서로 다른 장비와 규칙 필요
- 운송 과정 복잡
- 운송 및 관리 비용 증가

즉, 해상 컨테이너의 핵심은 화물 운송 방식을 표준화하는 것이다.

**정리**: 해상 컨테이너는 화물의 종류와 관계없이 표준 규격 박스로 운송 방식을 통일한 사례이며, 이 비유는 다음 절에서 다룰 도커 컨테이너의 개념을 이해하는 기반이 된다.

---

## 도커의 컨테이너 개념

- 도커의 **컨테이너** 개념도 해상 컨테이너와 비슷하다.
- 프로그램마다 실행하기 위해 필요한 환경은 서로 다르다.

EX)

Java Application
- Java Runtime
- Library
- Application
- 설정 파일

Python Application
- Python Runtime
- Library
- Application
- 설정 파일

- 기존에는 이러한 프로그램 실행 환경을 각각의 서버에 직접 설치하고 설정해야 했다.
- 도커에서는 애플리케이션과 실행에 필요한 파일들을 **이미지(Image)**로 패키징하고, 해당 이미지를 이용하여 컨테이너를 생성한다.

### Docker Container

- **컨테이너**는 애플리케이션을 실행하기 위한 격리된 실행 환경이다.
- 컨테이너에는 애플리케이션 실행에 필요한 다음과 같은 요소가 포함될 수 있다.
  - 프로그램(Application)
  - 언어 실행 환경(Runtime)
  - 라이브러리(Library)
  - 시스템 도구 및 파일
  - 설정 파일
- 중요한 점은 컨테이너마다 완전한 운영체제(OS)가 하나씩 설치되는 것은 아니라는 것이다.
- Linux Container의 경우 Host OS의 Linux Kernel을 공유하면서 애플리케이션 실행에 필요한 사용자 공간(User Space)을 각각 분리하여 사용한다.
- 즉, 컨테이너는 독립된 컴퓨터처럼 보이지만 실제로는 Host의 Kernel을 공유하는 격리된 프로세스 환경이다.

### 해상 컨테이너 vs 도커 컨테이너

해상 컨테이너
- 화물을 표준 규격의 박스에 담아 운송
- 내용물과 관계없이 동일한 방식으로 처리
- 다양한 운송 환경에서 동일한 규격 사용

도커 컨테이너
- 애플리케이션 실행 환경을 표준화하여 패키징
- 애플리케이션 종류와 관계없이 컨테이너 단위로 실행
- 개발/테스트/운영 환경의 차이를 줄일 수 있다.

- 해상 컨테이너 : 화물 운송 방식의 표준화
- Docker Container : 애플리케이션 실행 환경의 표준화

**정리**: 도커 컨테이너는 해상 컨테이너와 마찬가지로 "실행 환경의 표준화"라는 개념을 애플리케이션 영역에 적용한 것이며, 이어서 기존 서버 관리 방식과 비교하며 그 필요성을 살펴본다.

---

## 전통적인 서버 관리 방식

- 새로운 서비스를 개발하고 운영 서버에 배포해야 한다고 가정해보자.
- 개발 환경에서는 프로그램을 실행하기 위해 여러 환경을 구성해야 한다.
  - OS 설정
  - 방화벽 설정
  - 네트워크 설정
  - 프로그래밍 언어 및 Runtime 설치
  - 라이브러리 설치
  - Web Server 설치
  - DB 또는 DB Client 설치
  - 각종 환경 설정
- 개발 서버에 모든 환경을 구성하고 프로그램을 정상적으로 실행했다고 해도 운영 서버에 동일한 환경을 다시 구성해야 한다.

### 문제 발생 과정

```
개발 서버 환경 구성
        ↓
프로그램 개발
        ↓
개발 서버에서는 정상 실행
        ↓
운영 서버 환경 다시 구성
        ↓
Runtime 버전 차이
Library 버전 차이
환경 설정 차이
        ↓
프로그램 오류 발생
```

대표적인 문제는 개발 서버에서는 되는데 운영 서버에서는 안되는 상황이다.

기존 방식의 문제점
- 서버마다 동일한 환경을 반복해서 구성해야 함
- 서버마다 설정 차이가 발생할 수 있다.
- 프로그램 및 라이브러리 버전 관리가 어려움
- 서버를 추가할 때마다 동일한 작업을 반복해야 함
- 개발/테스트/운영 환경의 차이로 오류가 발생할 수 있다.
- 환경 구성에 많은 시간과 인력이 필요함

**정리**: 전통적인 방식은 서버마다 환경을 반복 구성해야 해서 버전 차이와 오류가 발생하기 쉬웠고, 도커는 이 문제를 이미지 기반 배포로 해결한다.

---

## 도커를 사용한 서버 관리 방식

- 도커를 사용하면 애플리케이션과 실행에 필요한 환경을 **Docker Image**로 만들 수 있다.
- 한 번 만들어진 이미지를 여러 서버에서 동일하게 사용할 수 있기 때문에 환경을 반복해서 구성하는 작업을 크게 줄일 수 있다.
- 도커의 기본 흐름
  - `Dockerfile → Build → Docker Image → Push → Registry → Pull → Container`

- Dockerfile에 이미지 생성 방법을 작성한다.
- Dockerfile을 Build하여 Docker Image를 생성한다.
- 생성된 Image를 Registry에 Push하여 저장하고 공유할 수 있다.
- 다른 서버에서는 Registry에 저장된 Image를 Pull한다.
- Pull한 Image를 기반으로 Container를 생성하여 애플리케이션을 실행한다.

도커를 사용한 배포 과정
```
Dockerfile 작성
        ↓
Docker Image Build
        ↓
Registry에 Image Push
        ↓
운영 환경에서 Image Pull
        ↓
Container 생성 및 실행
```

도커의 장점
- 개발/테스트/운영 환경의 차이를 줄일 수 있다.
- 애플리케이션 배포 방법을 표준화할 수 있다.
- 동일한 실행 환경을 쉽게 재현할 수 있다.
- 동일한 이미지를 여러 서버에서 재사용할 수 있다.
- 새로운 서버 또는 컨테이너를 빠르게 추가할 수 있다.
- CI/CD와 같은 자동화 환경과 연계하기 쉬움
- Kubernetes와 같은 컨테이너 오케스트레이션 환경에서 이미지를 사용할 수 있다.

**정리**: 도커의 기본 흐름(`Dockerfile → Build → Docker Image → Push → Registry → Pull → Container`)을 이해했다면, 이제 이 흐름의 첫 단계인 이미지를 만드는 기능(Build)을 자세히 살펴본다.

---

## Docker 이미지를 만드는 기능 (Build)

### 1) Docker Image

- **Docker Image**는 컨테이너를 생성하고 실행하기 위한 읽기 전용(Read Only) 템플릿이다.
- 이미지에는 애플리케이션 실행에 필요한 파일과 환경이 포함될 수 있다.
  - 프로그램(Application)
  - 언어 실행 환경(Runtime)
  - 라이브러리(Library)
  - Web Server
  - 필요한 OS User Space 파일
  - 환경 및 실행 설정
- 중요한 점은 Docker Image 안에 완전한 OS Kernel이 포함되는 것은 아니라는 것이다.
- 컨테이너가 실행되면 Host OS의 Kernel을 공유한다.
- 따라서 VM처럼 각각의 Guest OS 전체를 이미지에 포함할 필요가 없다.

### Image와 Container의 관계

- Docker Image와 Container의 관계는 프로그램과 실행 중인 프로세스의 관계와 비슷하게 이해할 수 있다.

Docker Image
- 컨테이너를 만들기 위한 템플릿
- 실행 전 상태
- 기본적으로 읽기 전용

Docker Container
- Image를 기반으로 생성
- 실제 애플리케이션이 실행되는 환경
- 실행 중 변경되는 데이터를 저장할 수 있는 쓰기 가능한 영역이 추가됨
- `Image → docker run → Container`

- 하나의 Image를 이용하여 여러 개의 Container를 생성할 수도 있다.

EX)
```
                    nginx:latest
                         │
            ─────────┼─────────
            │                 │                 │
        Container1        Container2        Container3
```
- 세 개의 Container는 동일한 Image를 기반으로 생성되지만 각각 독립된 실행 환경으로 동작한다.

**정리**: 이미지는 컨테이너를 만들기 위한 읽기 전용 템플릿이고, 컨테이너는 그 이미지를 기반으로 실행되는 실체라는 관계를 이해했다면, 이제 이미지가 내부적으로 어떻게 구성되는지(Layer 구조)를 살펴본다.

---

## Docker Image Layer

- Docker Image는 일반적인 하나의 파일처럼 구성되는 것이 아니라 여러 개의 읽기 전용 **Layer**가 쌓여 만들어진다.
- Dockerfile의 명령에 따라 이미지 Layer가 만들어질 수 있으며 Docker는 이러한 Layer를 이용하여 이미지를 효율적으로 저장하고 관리한다.

EX)
```dockerfile
FROM ubuntu:24.04
RUN apt-get update
RUN apt-get install -y nginx
COPY index.html /var/www/html/index.html
```

개념적으로 다음과 같이 여러 Layer가 쌓여 하나의 Image를 구성한다.

```
[ COPY index.html ]
[ RUN install nginx ]
[ RUN apt update ]
[ Ubuntu Base Image ]

          ↓

     Docker Image
```

- 이미지의 Layer는 기본적으로 읽기 전용이다.
- 컨테이너를 실행하면 이미지 Layer 위에 쓰기 가능한 Container Layer가 추가된다.

```
[ Container Layer ]    # Read / Write
[ Image Layer ]
[ Image Layer ]
[ Base Image ]         # Read Only
```

- 컨테이너에서 파일을 생성하거나 수정하면 이러한 변경 사항은 기본 이미지 자체를 직접 수정하는 것이 아니라 컨테이너의 쓰기 가능한 영역에 기록된다.
- 따라서 동일한 Image를 이용해 여러 Container를 생성할 수 있다.

Layer 방식의 장점
- 동일한 Layer를 여러 이미지에서 재사용 가능
- 이미지 저장 공간 절약
- 변경되지 않은 Layer는 Build Cache로 재사용 가능
- 이미지 Build 및 전송을 효율적으로 처리 가능

**정리**: Layer 구조 덕분에 이미지는 효율적으로 저장·재사용될 수 있으며, 다음으로는 이러한 이미지를 실제로 생성하는 방식(수동 vs Dockerfile)을 비교한다.

---

## Docker 이미지 생성 방식

Docker 이미지를 생성하는 대표적인 방법에는 다음과 같은 방식이 있다.

### 수동 방식 (docker commit)

- 실행 중인 컨테이너 내부에서 프로그램이나 파일을 변경한 뒤 해당 컨테이너의 변경 상태를 새로운 이미지로 저장하는 방식이다.

```bash
docker commit <컨테이너명> <이미지명>
```

장점
- 간단한 테스트에 사용할 수 있다.

단점
- 어떤 과정을 거쳐 이미지가 만들어졌는지 확인하기 어려움
- 동일한 이미지를 다시 재현하기 어려움
- 변경 이력 관리가 어려움
- 자동화에 적합하지 않음

따라서 일반적인 이미지 제작에서는 Dockerfile을 사용하는 방법이 권장된다.

### 자동 방식 (Dockerfile) — 권장

- **Dockerfile**은 Docker Image를 어떻게 만들 것인지 명령어 형태로 작성한 파일이다.

EX)
```dockerfile
FROM httpd
COPY index.html /usr/local/apache2/htdocs/
```

Dockerfile을 이용하여 이미지를 Build한다.

```bash
docker build -t myweb:1.0 .
```

장점
- 이미지 생성 과정을 코드로 관리할 수 있다.
- 동일한 Dockerfile을 이용하여 이미지를 다시 생성할 수 있다.
- 변경 이력을 관리하기 쉬움
- Git과 같은 형상관리 시스템으로 관리 가능
- CI/CD 자동화와 연계하기 쉬움

따라서 Dockerfile을 이용한 이미지 생성 방식이 일반적으로 사용된다.

### Docker Image 이름과 Tag

- Docker Image에는 이름과 Tag를 지정할 수 있다.

EX)
```
nginx:1.27
mysql:8.4
myapp:1.0
```

Tag는 이미지의 버전이나 용도를 구분하는 데 사용한다.

EX)
```
myapp:1.0
myapp:2.0
myapp:dev
myapp:test
```

- Tag를 지정하지 않으면 일반적으로 latest Tag가 사용된다.

EX) `docker pull nginx` ---> `docker pull nginx:latest`

- latest라는 이름이 반드시 가장 최신 버전이라는 것을 보장하는 것은 아니다.
- latest 역시 이미지에 붙이는 Tag 이름 중 하나이다.
- 따라서 운영 환경에서는 명확한 버전 Tag를 사용하는 것이 이미지 버전 관리에 유리하다.

### Docker 이미지 설계 원칙

- 컨테이너는 하나의 주요 역할을 중심으로 구성하는 것이 일반적이다.

EX)
```
nginx Container     ---> Web Server
mysql Container     ---> Database
redis Container     ---> Cache
spring-app Container---> Application
```

- 여러 서비스를 하나의 컨테이너에 모두 넣기보다는 역할에 따라 컨테이너를 분리하면 관리와 확장이 쉬워진다.

EX)
```
[ Web Container ]
        +
[ Application Container ]
        +
[ DB Container ]
        +
[ Cache Container ]
          ↓
      하나의 서비스
```

- 이러한 구조는 각 서비스를 독립적으로 배포하거나 확장하기 쉽다는 장점이 있다.

**정리**: 이미지 생성 방식과 설계 원칙(단일 역할 컨테이너)을 이해했다면, 이제 만들어진 이미지를 다른 서버와 공유하는 기능(Ship / Share)을 살펴본다.

---

## Docker 이미지를 공유하는 기능 (Ship / Share)

### 1) Docker Registry

- **Docker Registry**는 Docker Image를 저장하고 공유하기 위한 이미지 저장소이다.
- 개발 환경에서 생성한 이미지를 Registry에 저장한 후 다른 서버에서 다운로드하여 사용할 수 있다.
- 대표적인 Public Registry 서비스로 Docker Hub가 있다.

Registry의 기본 흐름
```
개발 서버 → Image Build → Push → Docker Registry → Pull → 운영 서버
```

Registry의 주요 기능
- Docker Image 저장
- Docker Image 다운로드(Pull)
- Docker Image 업로드(Push)
- 이미지 Tag 관리
- 이미지 버전 관리
- Public / Private Repository 관리

### Repository

- Registry 내부에서는 이미지를 Repository 단위로 관리한다.

EX)
```
nginx
mysql
redis
mycompany/myapp
```

- 하나의 Repository에는 여러 Tag의 이미지를 저장할 수 있다.

EX)
```
myapp:1.0
myapp:1.1
myapp:2.0
```

### Pull

- Registry에서 Image를 다운로드하는 작업이다.

```bash
docker pull nginx:latest
```

### Push

- 로컬에서 생성한 Image를 Registry에 업로드하는 작업이다.

```bash
docker push myaccount/myapp:1.0
```

따라서 Docker Image를 이용하면 개발 서버에서 만든 실행 환경을 Registry를 통해 다른 서버로 전달할 수 있다.

**정리**: Registry를 통해 이미지를 Push/Pull하는 방법을 이해했다면, 마지막으로 이 이미지를 실제로 실행하는 기능(Run)과 컨테이너 격리 원리를 살펴본다.

---

## Docker 이미지를 동작시키는 기능 (Run)

### 1) Container 실행 개념

- Docker Image 자체는 실행 중인 프로그램이 아니다.
- Image를 기반으로 Container를 생성하고 실행해야 실제 애플리케이션이 동작한다.

```bash
docker run nginx
```

개념
```
Docker Image → docker run → Docker Container → Application 실행
```

- 하나의 Image에서 여러 개의 Container를 생성할 수 있다.
- 각 Container는 같은 Image를 사용하더라도 서로 분리된 실행 환경을 가진다.

### 2) Container는 어떻게 격리되는가?

- Linux Container는 Host OS의 Linux Kernel을 공유한다.
- VM처럼 컨테이너마다 완전한 Guest OS를 새로 실행하는 방식이 아니다.
- 대신 Linux Kernel이 제공하는 여러 기능을 이용하여 프로세스와 자원을 격리하고 관리한다.

#### Namespace

- **Namespace**는 프로세스가 볼 수 있는 시스템 자원을 분리하는 기능이다.
- 컨테이너마다 자신만의 독립적인 시스템 환경이 존재하는 것처럼 보이게 한다.
- Namespace를 이용하여 다음과 같은 요소를 분리할 수 있다.
  - Process ID
  - Network
  - Hostname
  - Mount
  - User
  - IPC 등

- 따라서 Container A에서 실행 중인 프로세스와 Container B에서 실행 중인 프로세스가 서로 분리되어 보인다.

#### cgroups (Control Groups)

- **cgroups**는 프로세스가 사용할 수 있는 시스템 자원을 관리하고 제한하는 기능이다.

EX)
- CPU
- Memory
- I/O 등

- 특정 Container가 Host의 CPU나 Memory를 과도하게 사용하는 것을 제한할 수 있다.

#### Container File System

- 컨테이너는 자신의 Root File System을 가진 것처럼 동작한다.
- Linux의 Mount Namespace 등의 기능과 컨테이너 파일 시스템 기술을 이용하여 각 컨테이너의 파일 시스템 환경을 분리한다.

정리
```
Namespace    : 프로세스가 볼 수 있는 환경과 자원을 격리
cgroups      : CPU, Memory 등의 자원 사용을 관리/제한
Container FS : 컨테이너별 파일 시스템 환경 제공
```

이러한 Linux Kernel 기능들을 기반으로 Container가 독립적인 실행 환경처럼 동작한다.

**정리**: Namespace, cgroups, Container File System은 컨테이너가 격리된 것처럼 보이게 하는 핵심 Linux Kernel 기능이며, 이제 이러한 컨테이너 방식을 Bare Metal, VM과 비교하여 정리한다.

---

## Bare Metal vs Virtual Machine vs Container

- 애플리케이션을 실행하는 환경은 크게 다음과 같이 비교할 수 있다.
  - Bare Metal
  - Virtual Machine
  - Container

### 1) Bare Metal

- **Bare Metal**은 물리적인 서버 하드웨어에 운영체제를 직접 설치하여 사용하는 방식이다.

구조
```
[ Application ]
[     WAS     ]
[     OS      ]
[ Bare Metal  ]
```

- Bare Metal은 CPU, RAM, Disk, Network Interface 등 실제 물리적인 서버 장비를 의미한다.

특징
- 물리 하드웨어에 OS를 직접 설치
- 가상화 계층이 없어 직접적인 하드웨어 사용 가능
- 높은 성능이 필요한 환경에서 사용할 수 있다.
- 하나의 물리 서버를 여러 독립 OS 환경으로 나누어 사용하려면 별도의 가상화 기술이 필요함

### 2) Virtual Machine (VM)

- **Virtual Machine**은 하나의 물리 서버에서 여러 개의 독립적인 가상 컴퓨터를 실행하는 방식이다.
- VM 환경에서는 **Hypervisor**가 물리적인 CPU, Memory, Disk 등의 자원을 가상화하여 각각의 VM에 제공한다.

#### Hypervisor

- Hypervisor는 Virtual Machine을 생성하고 실행하며 물리적인 자원을 각 VM에 할당하고 관리하는 가상화 계층이다.
- 대표적인 Hypervisor 또는 가상화 제품으로 다음과 같은 것들이 있다.
  - VMware
  - KVM
  - Hyper-V
  - VirtualBox

- Hypervisor는 크게 Type 1과 Type 2 방식으로 구분할 수 있다.
- 물리 하드웨어에서 직접 동작하는 방식: VMware ESXi, Microsoft Hyper-V, KVM 기반 가상화 환경 등

#### Virtual Machine 구조 예시

```
[Application]  [Application]  [Application]
[   WAS    ]   [   WAS    ]   [   WAS    ]
[ Guest OS ]   [ Guest OS ]   [ Guest OS ]
[    VM    ]   [    VM    ]   [    VM    ]
[              Hypervisor                ]
[               Hardware                 ]
```

#### Guest OS

- 각 VM 내부에 설치되는 독립적인 운영체제이다.

EX)
- Ubuntu
- Rocky Linux
- Windows Server

- 각 VM은 독립적인 Kernel을 가진 Guest OS를 실행한다.

VM의 특징
- 하나의 물리 서버에서 여러 운영체제 실행 가능
- VM마다 독립적인 Guest OS와 Kernel 사용
- 서로 다른 종류의 OS를 실행할 수 있다.
- VM 단위의 높은 격리성 제공
- 각각 Guest OS를 실행하므로 Container보다 일반적으로 더 많은 자원을 사용
- Guest OS를 부팅해야 하므로 Container보다 시작 시간이 긴 편

### 3) Container

- Container는 VM과 다르게 일반적으로 각각의 Guest OS 전체를 실행하지 않는다.
- Linux Container는 Host OS의 Linux Kernel을 공유하면서 프로세스를 격리한다.

구조
```
[Application]  [Application]  [Application]
[Container ]   [Container ]   [Container ]
         [ Container Engine ]
         [    Host OS       ]
         [    Hardware      ]
```

#### Container Engine

- Container를 생성하고 실행하고 관리하는 소프트웨어 계층이다.

EX)
- Docker Engine
- containerd

Container의 특징
- Host Kernel 공유
- Guest OS 전체를 각각 실행하지 않음
- 프로세스 단위의 격리 환경 제공
- VM보다 일반적으로 적은 자원을 사용
- 빠른 생성 및 실행
- 동일한 Image를 이용하여 실행 환경을 쉽게 재현
- 여러 Container를 빠르게 생성하거나 제거 가능

### Bare Metal / VM / Container 핵심 비교

| 항목 | Bare Metal | Virtual Machine | Container |
|------|-----------|-----------------|-----------|
| 구조 | Hardware에 OS 직접 설치 | Hypervisor로 Hardware 가상화 | Host OS Kernel 공유 |
| Guest OS | 없음 | VM마다 독립 Guest OS | Guest OS 전체 실행 안 함 |
| Kernel | Host OS Kernel | VM마다 독립 Kernel | Host OS Kernel 공유 |
| 격리 | 없음 | VM 단위 높은 격리 | 프로세스 격리 |
| 자원 사용 | 직접 | VM마다 Guest OS 자원 필요 | 상대적으로 적음 |
| 시작 시간 | OS 부팅 시간 | Guest OS 부팅 필요 | 빠름 |

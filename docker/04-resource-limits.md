# Docker 04 — 컨테이너 리소스 제한

## 목차

1. [컨테이너 리소스 관리](#컨테이너-리소스-관리)
2. [메모리 리소스 제한](#메모리-리소스-제한)
3. [컨테이너 사용 리소스를 확인하는 모니터링 도구](#컨테이너-사용-리소스를-확인하는-모니터링-도구)
4. [stress — 부하 테스트 도구](#stress--부하-테스트-도구)
5. [컨테이너 메모리 리소스 제한 실습](#컨테이너-메모리-리소스-제한-실습)
6. [CPU 리소스 제한](#cpu-리소스-제한)
7. [CPU 리소스 제한 실습 1 (cpuset-cpus)](#cpu-리소스-제한-실습-1-cpuset-cpus)
8. [CPU 리소스 제한 실습 2 (cpu-shares)](#cpu-리소스-제한-실습-2-cpu-shares)

## 컨테이너 리소스 관리

- 컨테이너는 기본적으로 **리소스 제한**을 받지 않으며, 메모리 누수가 있는 컨테이너가 호스트 자원을 독점하는 것을 막거나 CPU 코어를 지정해 성능을 통제하려면 `-m`, `--cpuset-cpus`, `--cpu-shares` 같은 옵션으로 직접 제한을 걸어야 한다.
- 도커 컨테이너는 기본 설정으로 실행하면 CPU, RAM, 디스크 등의 사용량을 제한하지 않는다.
- 즉, 하나의 컨테이너는 운영시 필요한 만큼 호스트 서버의 자원을 자유롭게 가져다 쓸 수 있다.

제한하지 않는 이유
- 단순 애플리케이션 테스트나 개발 환경에서는 리소스 제한이 필요 없는 경우가 많다.
- 도커는 기본적으로 가볍고 빠르게 실행하는 것을 목표로 하기 때문에 처음부터 제한을 걸지 않는다.
- 필요한 경우에만 사용자가 직접 옵션을 지정해서 제한을 건다.

리소스를 제한하지 않으면 생길 수 있는 문제
- 하나의 컨테이너가 메모리를 너무 많이 사용해버리면 호스트 서버 전체가 느려지거나, 다른 컨테이너에 영향을 줄 수 있다.
- 예를 들어, Node.js 서버가 메모리 누수로 계속 증가하면 호스트 OS 자체가 다운될 수도 있다.

- 컨테이너는 호스트의 CPU, RAM, 디스크를 빌려 쓰는 구조다.
- 하나의 서버(Docker HOST) 위에서 여러 컨테이너가 동시에 실행되기 때문에 필요하면 각각의 컨테이너에게 "얼마까지 사용 가능"이라고 규칙을 줄 수 있다.

```
Docker Host
├── CPU
├── RAM
├── Swap
└── 디스크

Docker Host
├── Web 컨테이너
├── DB 컨테이너
└── WAS 컨테이너
```

**정리**: 컨테이너는 기본적으로 리소스 제한이 없어 자원을 독점할 위험이 있다. 다음 절부터 메모리, CPU 각각에 대한 구체적인 제한 옵션을 살펴본다.

---

## 메모리 리소스 제한

- 도커 컨테이너는 기본적으로 메모리 제한이 없기 때문에 애플리케이션에 따라 메모리가 계속 증가하면 서버 전체가 죽을 수 있다.
- 따라서 서비스 운영 시에는 반드시 메모리 제한 옵션을 사용한다.
- 제한 단위는 b, k, m, g 로 할당한다.
- 옵션을 넘어서 메모리를 사용하려 하면 컨테이너는 OOM(Out Of Memory) 오류를 내고 종료된다.

| 옵션 | 설명 |
|------|------|
| `--memory`, `-m` | 컨테이너가 사용할 최대 메모리 양을 지정 |
| `--memory-swap` | 컨테이너가 사용할 스왑 메모리 설정, 메모리 용량과 스왑의 합을 의미 (생략시 메모리의 2배) |
| `--memory-reservation` | memory 값보다 작은 값으로 구성하는 소프트 제한 값 설정 |
| `--oom-kill-disable` | OOM Killer가 프로세스를 kill 하지 못하도록 보호 |

### --memory, -m

- 컨테이너가 사용할 수 있는 메모리의 최대량을 지정한다.
- m 단위 또는 g 단위로 많이 사용한다. EX) `-m 512m`, `-m 2g`
- 이 옵션을 넘어서 메모리를 사용하려 하면 컨테이너는 OOM(Out Of Memory) 오류를 내고 종료된다.

### --memory-swap

- 컨테이너가 사용할 수 있는 "메모리 + 스왑"의 총합을 설정 (스왑 용량 자체를 지정하는 옵션은 아니다.)
- 예를 들어 메모리를 200m로 설정하고 `--memory-swap`을 300m로 설정하면 컨테이너가 사용할 수 있는 물리 메모리는 200m이고, 실제 사용할 수 있는 스왑 공간은 100m가 된다.
- 메모리 문제를 테스트할 때 유용한 옵션이다.

### --memory-reservation

- 소프트 제한값으로 메모리 사용이 이 값보다 넘어가면 시스템은 메모리 회수를 시도한다.
- 연성 제한이라고 부르며, 실제 강제 제한은 아니다. 하지만 자원이 부족한 환경에서는 이 값이 의미 있게 동작한다.
  - EX) `--memory 1g`
  - EX) `--memory-reservation 500m`
  - 최대는 1g 사용하지만, 가능하면 500m 이하로 유지하려는 정책이다.

### --oom-kill-disable

- 컨테이너 내부에서 메모리를 과다 사용해도 리눅스의 OOM Killer가 컨테이너 프로세스를 강제로 종료시키지 못하게 한다.
- 일반적으로 프로덕션 환경에서는 위험하므로 거의 사용하지 않는다.
- 서비스가 계속 다운되지 않고 버티게 하려는 실험 환경에서만 사용한다.

### 메모리 제한 예시

```bash
[guest@docker ~]$ docker run  -d  --name  ex-memory  -m 300m  nginx:1.14
# 컨테이너가 사용할 수 있는 최대 메모리 = 300MB
# 300MB 초과시 OOM 발생 (컨테이너가 강제 종료(Killed) 될 수 있음)


[guest@docker ~]$ docker run  -d  --name ex-swap  -m 200m  --memory-swap  300m  nginx:1.14
# 실제 메모리(RAM) : 최대 200MB
# Swap             : RAM + Swap 합  (swap 사용 가능량 = 100MB)
# RAM은 200MB까지만 쓰게 하고, 부족하면 swap까지 포함해서 총 300MB까지만 사용 가능


[guest@docker ~]$ docker run  -d  --name  ex-reservation  -m 1g  --memory-reservation 500m  nginx:1.14
# 평상시   : 500MB 기준으로 관리
# 시스템 메모리 부족 시 이 컨테이너가 먼저 압박 대상
# reservation은 soft limit으로 강제 종료 기준은 아니다.


[guest@docker ~]$ docker run -d --name ex-oom -m 150m --oom-kill-disable nginx:1.14
# 메모리 제한: 150MB
# OOM Killer로 컨테이너를 죽이지 않음 (메모리 부족 발생 시 컨테이너가 멈추거나 응답 불가상태가 될수 있다.)
```

**정리**: `--memory`, `--memory-swap`, `--memory-reservation`, `--oom-kill-disable` 옵션으로 메모리 제한을 거는 방법을 살펴봤다. 다음으로는 이러한 제한이 실제로 어떻게 동작하는지 확인할 모니터링 도구(`docker stats`, `docker events`)를 살펴본다.

---

## 컨테이너 사용 리소스를 확인하는 모니터링 도구

### docker stats

- `docker stats`는 컨테이너의 실시간 런타임(Resource) 사용량을 확인하는 명령어이다.
- CPU 사용률, 메모리 사용량, 네트워크 입출력, 블록 I/O 등을 실시간으로 표시한다.

```bash
docker stats
docker stats web
docker stats 컨테이너1 컨테이너2
```

| 항목 | 설명 |
|------|------|
| **CPU%** | 호스트 CPU 전체 대비 현재 컨테이너가 쓰는 CPU 사용률. 예) CPU가 4코어인데 25%라면, 한 코어의 25% 또는 여러 코어 합쳐 25%를 사용 중 |
| **MEM USAGE / LIMIT** | 컨테이너 메모리 사용량 / 설정된 최대 메모리 제한. 예) 100MiB / 500MiB --> 현재 100MiB 사용 중, 최대 500MiB까지 사용 가능 |
| **MEM %** | 컨테이너 메모리 사용률(%). 메모리 사용량 ÷ 메모리 제한 × 100 |
| **NET I/O** | 컨테이너가 네트워크로 주고받은 데이터 총량. 예) 2.1MB / 1.2MB --> 받은 데이터 / 보낸 데이터 |
| **BLOCK I/O** | 컨테이너 내부에서 파일을 읽고 쓴 총량. 예) 50MB / 10MB --> 디스크에서 읽은 양 / 쓴 양 |
| **PIDS** | 컨테이너 안에서 실행 중인 프로세스(Process) 개수. 예) Apache 웹서버 컨테이너는 보통 2~4개 프로세스가 실행됨 |

### docker events

- 도커 호스트에서 발생하는 각종 이벤트를 실시간으로 출력한다.
- 컨테이너 생성, 시작, 중지, 삭제, 이미지 Pull, Push 등 다양한 이벤트를 확인할 수 있다.

```bash
# 기본 사용
[root@localhost ~]# docker events

# 특정 컨테이너 필터링
[root@localhost ~]# docker events  -f  container=web
[root@localhost ~]# docker events  -f  image=nginx
```

- `-docker events` : Docker에서 발생하는 이벤트를 실시간으로 확인
- `-f` : filter의 약자로 조건에 맞는 이벤트만 필터링
- `container=web` : 이름 또는 ID가 web인 컨테이너의 이벤트만 확인

### 컨테이너 종료 event 확인

```bash
[guest@Server-A ~]$ docker events


[guest@Server-A ~]$ docker stop  nginx_web    # 다른 SSH 세션으로 접속 후 컨테이너 실행 중지


[guest@Server-A ~]$ docker events
2026-08-06T16:04:56.364594221+09:00 container kill 5e4cf32df303e06aa8f52b177aa6c0fcf1c827e6b27d08fb00df7569358c87ca (image=nginx:latest, name=nginx_web, signal=3)
2026-08-06T16:04:56.486829151+09:00 network disconnect b1e0f3cbc5f90028861449a493113250962958392c331ae899d179fa2ce58ec6 (container=5e4cf32df303..., name=bridge, type=bridge)
2026-08-06T16:04:56.488636391+09:00 container stop 5e4cf32df303e06aa8f52b177aa6c0fcf1c827e6b27d08fb00df7569358c87ca (image=nginx:latest, name=nginx_web)
2026-08-06T16:04:56.489243908+09:00 container die 5e4cf32df303e06aa8f52b177aa6c0fcf1c827e6b27d08fb00df7569358c87ca (exitCode=0, image=nginx:latest, name=nginx_web)
```

- `signal=15` : SIGTERM — 정상적으로 종료해 달라는 요청, 프로그램이 정리 작업을 수행할 수 있음
- `signal=3`  : SIGQUIT — 종료하면서 디버깅용 코어 덤프를 남길 수 있음
- `signal=9`  : SIGKILL — 즉시 강제 종료, 프로그램이 거부하거나 처리할 수 없음

### 컨테이너 실행 event 확인

```bash
[guest@Server-A ~]$ docker events


[guest@Server-A ~]$ docker start  nginx_web    # 다른 SSH 세션으로 접속 후 컨테이너 실행


[guest@Server-A ~]$ docker events
2026-08-06T16:09:10.238807774+09:00 network connect b1e0f3cbc5f90028861449a493113250962958392c331ae899d179fa2ce58ec6 (container=5e4cf32df303..., name=bridge, type=bridge)
2026-08-06T16:09:10.248533995+09:00 container start 5e4cf32df303e06aa8f52b177aa6c0fcf1c827e6b27d08fb00df7569358c87ca (image=nginx:latest, name=nginx_web)
```

**정리**: `docker stats`로 실시간 자원 사용량을, `docker events`로 컨테이너의 생명주기 이벤트를 확인하는 방법을 살펴봤다. 이제 이러한 제한을 실제로 테스트하기 위한 부하 발생 도구 `stress`를 설치하고 사용해본다.

---

## stress — 부하 테스트 도구

- `stress`는 CPU, 메모리, 디스크 입출력(I/O) 등에 의도적으로 부하를 발생시키는 테스트 도구이다.
- 컨테이너의 CPU 및 메모리 제한이 정상적으로 적용되는지 확인하거나, 서버가 부하 상황에서 어떻게 동작하는지 테스트할 때 사용한다.
- 운영 환경에서 발생할 수 있는 다양한 부하 상황을 인위적으로 재현할 수 있다.

stress가 제공하는 부하 종류
- CPU 부하       : `-c` 옵션
- 메모리 부하    : `-m` 옵션
- 디스크 I/O 부하 : `-i` 옵션
- 프로세스 생성 부하 : `-d` 옵션
- 실행 시간 제한 : `-t` 옵션

### EPEL 저장소 추가

```bash
[root@rocky ~]# sudo dnf install -y epel-release
```

- **EPEL (Extra Packages for Enterprise Linux)**
  - 엔터프라이즈 리눅스용 추가 패키지 저장소
  - 공식 저장소에 없는 다양한 오픈소스 패키지를 설치할 수 있게 해주는 저장소
  - 서버 OS(RHEL, Rocky, CentOS)는 안정성이 중요하기 때문에 기본 패키지가 적다.
  - EPEL을 활성화하면 수천 개 이상의 패키지를 더 설치할 수 있다.

### stress 설치

```bash
[guest@Server-A ~]$ sudo  dnf install -y stress


[guest@Server-A ~]$ rpm  -qa  | grep stress
stress-1.0.4-29.el9.x86_64


[guest@Server-A ~]$ nproc    # CPU 개수
2
```

### stress CPU 부하 테스트

```bash
[guest@Server-A ~]$ top
top - 16:18:42 up  6:45,  2 users,  load average: 0.02, 0.02, 0.00
Tasks: 187 total,   2 running, 185 sleeping,   0 stopped,   0 zombie
%Cpu(s):  2.9 us,  2.9 sy,  0.0 ni, 94.1 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
MiB Mem :   1674.8 total,    669.6 free,    649.1 used,    532.9 buff/cache
MiB Swap:   4096.0 total,   4046.3 free,     49.7 used.   1025.7 avail Mem


[guest@Server-A ~]$ stress  -c  1    # CPU 1개만큼의 부하를 발생
stress: info: [9138] dispatching hogs: 1 cpu, 0 io, 0 vm, 0 hdd


[guest@Server-A ~]$ top
%Cpu(s): 18.5 us,  0.2 sy,  0.0 ni, 81.0 id,  ...    # 18.5%


[guest@Server-A ~]$ top
%Cpu(s): 49.7 us,  0.0 sy,  0.0 ni, 50.2 id,  ...    # 49.7%


[guest@Server-A ~]$ top
%Cpu(s): 49.9 us,  0.0 sy,  0.0 ni, 50.0 id,  ...    # 49.9%
```

### stress 메모리 부하 테스트

```bash
[guest@Server-A ~]$ free  -m
         total   used   free   shared  buff/cache   available
Mem:      1674    610    707        2         533       1064
Swap:     4095     49   4046


[guest@Server-A ~]$ stress  --vm=2  --vm-bytes  1G -t 20
```

옵션 설명
- `--vm 2` : 메모리 부하(vm worker)를 2개 생성. worker 하나당 메모리 할당을 반복하며 CPU와 메모리에 부하를 준다.
- `--vm-bytes 1G` : 각 vm worker가 사용할 메모리 용량을 지정 (worker 하나당 1GB). 2개 worker → 1GB + 1GB = 총 2GB 사용.
- `-t 20` : 부하를 주는 시간을 지정 (20초 동안 메모리를 계속 할당·접근하며 테스트)

```bash
[guest@Server-A ~]$ stress  --vm 2  --vm-bytes  1G -t 20
stress: info: [9151] dispatching hogs: 0 cpu, 0 io, 2 vm, 0 hdd
stress: info: [9151] successful run completed in 20s


[guest@Server-A ~]$ free  -m
          total    used   free   shared  buff/cache   available
Mem:       1674    1618     79        1          99          56
Swap:      4095     870   3225


[guest@Server-A ~]$ free  -m
           total    used   free   shared  buff/cache   available
Mem:        1674    1650     69        0          54          24
Swap:       4095    1513   2582
```

**정리**: 호스트에서 `stress`로 CPU/메모리 부하를 발생시키는 방법을 실습했다. 이제 이 도구를 컨테이너 안에 넣어 실제 메모리 제한 옵션이 어떻게 동작하는지 직접 검증한다.

---

## 컨테이너 메모리 리소스 제한 실습

### Dockerfile 작성

```bash
[guest@Server-A ~]$ mkdir  membuild

[guest@Server-A ~]$ cd  membuild/

[guest@Server-A membuild]$ vi dockerfile
FROM rockylinux:9

RUN  dnf update -y &&  \
 dnf  install  -y  epel-release  && \
 dnf  install  stress  -y  && \
 dnf  clean all

CMD ["/bin/sh", "-c", "stress -c 2"]
```

옵션 설명
- `/bin/sh`   : /bin/sh 쉘을 사용
- `-c`        : 뒤에 오는 문자를 명령어로 처리
- `stress -c 2` : CPU 2개 만큼의 부하를 발생

### 실습 1 — 90m 메모리 사용 (제한 100m, swap=100m)

```bash
[guest@Server-A membuild]$ docker  run  -m 100m  --memory-swap=100m   stress_img:latest  stress  --vm 1  --vm-bytes 90m  -t  5
stress: info: [1] dispatching hogs: 0 cpu, 0 io, 1 vm, 0 hdd
stress: info: [1] successful run completed in 5s
```

- 컨테이너의 최대 메모리 사용량은 100MB로 제한되어 있다.
- stress 명령은 메모리 부하 작업 1개를 생성하여 90MB를 사용한다.
- 요청한 메모리 90MB가 제한값 100MB보다 작으므로 OOM이 발생하지 않기 때문에 stress 작업은 5초 동안 정상적으로 수행된다.
- 5초가 지나면 stress 명령이 정상 종료되고, 컨테이너도 함께 종료된다.

### 실습 2 — 150m 메모리 사용 (제한 100m, swap=100m) → OOM Kill 발생

```bash
[guest@Server-A membuild]$ docker  run  -m 100m  --memory-swap=100m   stress_img:latest  stress  --vm 1  --vm-bytes 150m  -t  5
stress: info: [1] dispatching hogs: 0 cpu, 0 io, 1 vm, 0 hdd
stress: FAIL: [1] (415) <-- worker 7 got signal 9
stress: WARN: [1] (417) now reaping child worker processes
stress: FAIL: [1] (421) kill error: No such process
stress: FAIL: [1] (451) failed run completed in 1s
```

- `worker 7`   : 메모리 테스트를 수행하던 작업(프로세스) 번호
- `got signal 9` : 이 프로세스가 시그널 9를 받았다는 의미 (SIGKILL - 강제 종료)
- 즉 컨테이너 안에서 제한된 메모리 이상을 사용하기 때문에 커널이 강제로 컨테이너를 kill한다.

### 실습 3 — 150m 메모리 사용 (제한 100m, swap 미설정) → swap 2배 자동 할당으로 성공

```bash
[guest@Server-A membuild]$ docker  run  -m 100m  stress_img:latest  stress  --vm 1  --vm-bytes 150m  -t  5
stress: info: [1] dispatching hogs: 0 cpu, 0 io, 1 vm, 0 hdd
stress: info: [1] successful run completed in 5s
```

- swap 메모리를 할당하지 않았기 때문에 할당된 메모리의 2배를 자동으로 할당한다.
- swap 메모리의 사용으로 인해 컨테이너는 kill되지 않는다.

### Dockerfile 실습 2 (CMD에 메모리 120MB 테스트 내장)

```bash
[guest@Server-A membuild]$ vi dockerfile
FROM rockylinux:9

RUN  dnf update -y &&  \
 dnf  install  -y  epel-release  && \
 dnf  install  stress  -y  && \
 dnf  clean all

CMD ["/bin/sh", "-c", "stress --vm 1  --vm-bytes 120m -t 5"]

:wq


[guest@Server-A membuild]$ docker  build -t  memtest  .


[guest@Server-A membuild]$ docker  images
IMAGE                            ID             DISK USAGE    CONTENT SIZE
httpd:latest                     2920ed858727   175MB         47.6MB
memtest:latest                   4f93aad3d9ae   530MB         136MB
nginx-with-package:1.1           659fa47f6fde   349MB         97.3MB
nginx:latest                     8541484afbc9   238MB         66MB
registry:3                       1be55279f18a   83.3MB        20.4MB
stress_img:latest                3f8392f560c6   530MB         136MB
```

```bash
# --rm 옵션: 실행 끝나자마자 자동 정리
[guest@Server-A membuild]$ docker  run  --name  mem_test  --rm  -m 100m  memtest:latest
stress: info: [1] dispatching hogs: 0 cpu, 0 io, 1 vm, 0 hdd
stress: info: [1] successful run completed in 5s
```

- CMD에서 메모리를 120MB 사용하도록 되어 있다.
- RAM 제한은 100MB, swap을 설정하지 않았기 때문에 2배로 설정되므로 스왑 메모리가 200MB가 된다.
- 따라서 120MB 사용이 성공한다.

```bash
# -m 100m : 컨테이너가 사용할 수 있는 RAM 최대 100MB
# --memory-swap 100m : RAM + SWAP 총합 100MB로 제한
[root@rocky ~]# docker run  --name mem_test  --rm -m 100m --memory-swap 100m  memtest:latest
stress: info: [1] dispatching hogs: 0 cpu, 0 io, 1 vm, 0 hdd
stress: FAIL: [1] (415) <-- worker 7 got signal 9
stress: WARN: [1] (417) now reaping child worker processes
stress: FAIL: [1] (421) kill error: No such process
stress: FAIL: [1] (451) failed run completed in 0s
```

### MemoryReservation 테스트 (soft limit)

- MemoryReservation은 "소프트 제한"으로, 실제 강제 제한은 아니지만, 컨테이너가 과도한 메모리를 쓰지 않도록 우선순위를 낮춘다.

```bash
[root@rocky ~]# docker  run  --name  mem_test  --rm  --memory=200m  --memory-reservation=50m  memtest:latest
stress: info: [1] dispatching hogs: 0 cpu, 0 io, 1 vm, 0 hdd
stress: info: [1] successful run completed in 5s
```

- `reservation`은 "이 메모리 값을 넘어가면 압박을 준다." 하지만 강제로 죽이지는 않는다.
- 200MB까지는 kill당하지 않는다. (200MB 초과 시 즉시 강제 종료)
- 50MB까지는 최소한 보장해주려고 노력한다.(50MB 넘게 써도 됨)

**정리**: `stress`를 이용해 메모리 제한(-m), swap 조합, MemoryReservation의 실제 동작(OOM Kill 발생 여부)을 직접 확인했다. 다음으로는 CPU 리소스를 제한하는 옵션들을 살펴본다.

---

## CPU 리소스 제한

| 옵션 | 설명 |
|------|------|
| `--cpus` | 컨테이너가 사용할 수 있는 CPU 사용량의 상한을 코어 개수 기준으로 지정 |
| `--cpuset-cpus` | 컨테이너가 사용할 수 있는 CPU 코어 번호를 고정(pinning)하는 옵션 |
| `--cpu-shares` | 컨테이너가 "다른 컨테이너와 함께 CPU를 나눠 쓸 때"의 상대적인 비중(가중치)을 지정 |

### --cpus

- 컨테이너가 사용할 수 있는 CPU 사용량의 상한을 코어 개수 기준으로 지정한다.
- 물리 코어를 직접 할당하는 것이 아니라, 스케줄러가 주는 CPU 점유 시간을 최대 몇 개의 코어에 해당하는 수준까지 허용할지 정하는 옵션이다.
- "이 컨테이너는 최대 CPU 1.5개가 동시에 일하는 것과 비슷한 양의 CPU 스케줄링 시간만 사용할 수 있다."

### --cpuset-cpus

- 컨테이너가 사용할 수 있는 CPU 코어 번호를 고정(pinning)하는 옵션
- 지정된 코어 범위 안에서만 CPU를 사용하게 강제한다.
- EX) `--cpuset-cpus="0-2"` : 0번, 1번, 2번 코어에서만 실행 (총 3개 코어만 사용)
- EX) `--cpuset-cpus="0,3"` : 0번, 3번 코어에서만 실행
- 주의) 호스트가 4코어(0~3)인데 "0-4"라고 쓰면 4번 코어는 존재하지 않으므로 잘못된 설정이다.

### --cpu-shares

- 컨테이너가 "다른 컨테이너와 함께 CPU를 나눠 쓸 때"의 상대적인 비중(가중치)을 지정한다.
- 절대적인 CPU 상한을 제한하는 것이 아니라, 경쟁 상황에서 우선순위를 정하는 옵션이다.
- 기본값은 1024 (아무 설정도 안 한 컨테이너의 기준값)
- EX) `--cpu-shares=2048` : 기본값(1024)을 가진 컨테이너보다 2배 정도 더 많은 CPU 점유 기회를 갖는다.
- 주의) 시스템에 여유가 많을 때는 두 컨테이너 모두 충분히 CPU를 쓸 수 있으므로 shares 값 차이가 체감되지 않을 수 있다. (경쟁 상황에서만 상대적 비중이 의미를 가짐)

### CPU 제한 예시

```bash
[guest@docker ~]$ docker run -d --cpus=".5" rockylinux:9
# 이 컨테이너는 CPU 0.5개만 사용 가능
# 전체 CPU 시간의 50%까지만 사용

[guest@docker ~]$ docker run -d --cpu-shares 2048 rockylinux:9
# CPU 사용 비중(weight) 설정
# 기본값은 1024
# 2048이면 기본 컨테이너보다 2배 우선권

[guest@docker ~]$ docker run -d --cpuset-cpus 0-3 rockylinux:9
# 컨테이너가 CPU 코어 0,1,2,3만 사용
# 다른 코어는 절대 사용 불가
```

**정리**: `--cpus`, `--cpuset-cpus`, `--cpu-shares` 세 가지 CPU 제한 옵션의 개념을 살펴봤다. 이어서 각 옵션을 실제로 적용해보는 실습을 진행한다.

---

## CPU 리소스 제한 실습 1 (cpuset-cpus)

```bash
[guest@Server-A ~]$ docker  run  -d  --name CPUset  --cpuset-cpus 0 stress_img:latest  stress  --cpu 1
6aa81e58c339864e7c0a6c2fc42df091d980916000d0e75fd334bb50fc53a897

# --cpuset-cpus 0 : 해당 컨테이너가 사용할 CPU를 고정하는 옵션


# htop 패키지 설치
[guest@Server-A ~]$ sudo dnf install -y htop

[guest@Server-A ~]$ htop


[guest@Server-A ~]$ docker rm -f CPUset    # 컨테이너 강제 삭제
CPUset


# 사용할 CPU를 0 - 1까지로 고정
[guest@Server-A ~]$ docker  run  -d  --name CPUset  --cpuset-cpus 0-1 stress_img:latest  stress  --cpu 1
```

**정리**: `--cpuset-cpus`로 컨테이너가 사용할 CPU 코어를 고정하는 실습을 진행했다. 마지막으로 `--cpu-shares`(가중치 기반 배분)를 여러 컨테이너에 적용해 CPU 점유 비율 차이를 확인한다.

---

## CPU 리소스 제한 실습 2 (cpu-shares)

```bash
# 가중치 = 기본값 1024
[guest@Server-A ~]$ docker run  -d  --name share1  stress_img:latest

# 가중치 = 1024 (2배)
[guest@Server-A ~]$ docker run  -d  --name share2  -c 2048  stress_img:latest

# 가중치 = 512
[guest@Server-A ~]$ docker run  -d  --name share3  -c 512  stress_img:latest
```

- Docker의 `-c` 옵션(CpuShares)는 "CPU를 나눠 가질 때 누구에게 더 많이 줄 것인가?"를 결정하는 비율(weight) 값이다.
- 절대적인 CPU 제한이 아니다.
- 컨테이너끼리 CPU를 나눌 때 "누가 더 많이 가져가는지" 결정하는 배분 규칙이다.

```
share1 : 1024
share2 : 2048
share3 : 512

1024 + 2048 + 512 = 3584

share1 = 1024 / 3584 ≈ 28%
share2 = 2048 / 3584 ≈ 57%
share3 = 512 / 3584  ≈ 14%
```

```bash
[guest@Server-A ~]$ docker ps
CONTAINER ID   IMAGE                COMMAND                CREATED          STATUS          NAMES
4c8f316f6d17   stress_img:latest   "/bin/sh -c 'stress …"  2 seconds ago    Up 2 seconds    share3
b56aa5a5551d   stress_img:latest   "/bin/sh -c 'stress …"  6 seconds ago    Up 5 seconds    share2
fbff03257e30   stress_img:latest   "/bin/sh -c 'stress …"  14 seconds ago   Up 14 seconds   share1


CONTAINER ID   NAME     CPU %    MEM USAGE / LIMIT       MEM %    NET I/O        BLOCK I/O    PIDS
4c8f316f6d17   share3   73.80%   912KiB / 1.636GiB       0.05%    2.17kB / 126B   0B / 0B     3
b56aa5a5551d   share2   198.79%  904KiB / 1.636GiB       0.05%    2.3kB / 126B    0B / 0B     3
fbff03257e30   share1   125.16%  2.555MiB / 1.636GiB     0.15%    2.69kB / 126B   1.81MB / 0B 3


[guest@Server-A ~]$ docker rm -f share1
[guest@Server-A ~]$ docker rm -f share2
[guest@Server-A ~]$ docker rm -f share3
```

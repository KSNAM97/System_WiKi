# 리눅스 개요 (Linux Overview)

> GNU/Linux 개요, 커널·쉘 구조, 배포판 종류 및 특징

## 목차

1. [리눅스란?](#리눅스란)
2. [GNU/Linux 철학](#gnulinux-철학)
3. [가상머신 (Virtual Machine)](#가상머신-virtual-machine)
4. [커널(Kernel)](#커널kernel)
5. [쉘(Shell)](#쉘shell)
6. [리눅스 배포판(Distribution)](#리눅스-배포판distribution)
7. [리눅스 디렉터리 구조 (FHS)](#리눅스-디렉터리-구조-fhs)
8. [패키지 관리 (dnf)](#패키지-관리-dnf)
9. [기본 시스템 명령어](#기본-시스템-명령어)
10. [GNU 프로젝트 상세](#gnu-프로젝트-상세)
11. [커널 버전 역사](#커널-버전-역사)
12. [쉘 동작 흐름 (ls -l 입력 시)](#쉘-동작-흐름-ls--l-입력-시)
13. [Red Hat 계열 계보](#red-hat-계열-계보)
14. [VMware Workstation Pro vs Player](#vmware-workstation-pro-vs-player)
15. [실습용 VM 구성표](#실습용-vm-구성표)

## 리눅스란?

- **운영체제(OS)**: 하드웨어를 제어하고 응용 프로그램이 동작하는 환경을 제공하는 소프트웨어
- **리눅스**: Unix 기반 오픈소스 운영체제 커널
- 1991년 **리누스 토르발스(Linus Torvalds)** 가 개발

**정리**: 리눅스는 리누스 토르발스가 개발한 Unix 기반 오픈소스 커널이며, 하드웨어와 응용 프로그램 사이의 실행 환경을 제공하는 운영체제의 핵심이다.

---

## GNU/Linux 철학

| 원칙 | 내용 |
|------|------|
| Free Software | 소프트웨어를 자유롭게 사용, 복사, 수정, 배포 가능 |
| Open Source | 소스 코드가 공개됨 |
| GPL | GNU 일반 공중 사용 허가서 - 배포 시 소스 공개 의무 |

**GNU**: "GNU is Not Unix" - 유닉스 호환이지만 자유로운 소프트웨어 프로젝트

**정리**: GNU/Linux는 자유 소프트웨어·오픈소스·GPL이라는 세 가지 원칙 위에서 만들어진 생태계이다.

---

## 가상머신 (Virtual Machine)

하나의 물리 컴퓨터에서 여러 운영체제를 동시에 실행하는 기술

```bash
물리 컴퓨터
├── Windows 10 (호스트 OS)
│   └── VMware / VirtualBox
│       ├── Rocky Linux 9 (Server-A)
│       ├── Rocky Linux 9 (Server-B)
│       └── Rocky Linux 9 (Client-L)
```

| 소프트웨어 | 특징 |
|---------|------|
| VMware Workstation | 기업용, 안정적 |
| VirtualBox | 무료, 개인용 |

**정리**: 가상머신은 하나의 물리 장비 위에서 여러 게스트 OS를 동시에 띄워 실습·서버 환경을 구성할 수 있게 해준다.

---

## 커널(Kernel)

운영체제의 핵심으로 하드웨어와 소프트웨어 사이의 중간 역할

```bash
사용자 프로그램
    ↓
  쉘(Shell)     ← 명령 해석기
    ↓
커널(Kernel)    ← 하드웨어 제어
    ↓
하드웨어 (CPU, Memory, Disk, Network)
```

### 커널 버전 확인

```bash
uname -r        # 커널 버전
uname -a        # 전체 시스템 정보
```

**정리**: **커널**은 하드웨어를 직접 제어하며, `uname` 명령으로 현재 커널 버전과 시스템 정보를 확인할 수 있다.

---

## 쉘(Shell)

사용자의 명령을 커널에게 전달하는 **명령 해석기**

```bash
사용자 → [쉘 입력] → 커널 → 하드웨어 → 결과 → 쉘 화면 출력
```

| 쉘 종류 | 특징 |
|---------|------|
| sh | 전통적인 Bourne Shell, POSIX 표준 |
| bash | Rocky Linux 기본 쉘, 자동완성, 히스토리 |
| zsh | 강력한 자동완성, oh-my-zsh 플러그인 |
| tcsh, ksh | 상업용 Unix 계열 |

```bash
# 현재 쉘 확인
echo $SHELL

# 사용 가능한 쉘 목록
cat /etc/shells

# 쉘 변경
chsh -s /bin/zsh
```

**정리**: **쉘**은 사용자와 커널 사이의 명령 해석기이며, `bash`가 Rocky Linux의 기본 쉘이다. `echo $SHELL`, `chsh` 등으로 확인·변경할 수 있다.

---

## 리눅스 배포판(Distribution)

리눅스 커널 + GNU 도구 + 패키지 관리자 + 설정 도구 묶음

| 계열 | 배포판 | 특징 |
|------|--------|------|
| Red Hat 계열 | RHEL, Rocky Linux, AlmaLinux, CentOS | 기업용, dnf 패키지 관리 |
| Debian 계열 | Ubuntu, Debian, Kali Linux | 개인/서버, apt 패키지 관리 |
| SUSE 계열 | openSUSE, SLES | 유럽 기업용 |
| Arch 계열 | Arch, Manjaro | 롤링 릴리즈, 최신 패키지 |

### Rocky Linux

- RHEL(Red Hat Enterprise Linux)의 완전한 호환 배포판
- CentOS 8 단종 이후 대안으로 등장
- 기업 환경에서 가장 많이 사용

```bash
# Rocky Linux 버전 확인
cat /etc/rocky-release
cat /etc/os-release
```

**정리**: **배포판**은 커널에 GNU 도구와 패키지 관리자를 묶은 것으로, **Rocky Linux**는 CentOS의 대안으로 자리 잡은 RHEL 호환 배포판이다.

---

## 리눅스 디렉터리 구조 (FHS)

Filesystem Hierarchy Standard - 표준 디렉터리 구조

| 디렉터리 | 설명 |
|---------|------|
| `/` | 루트(최상위) 디렉터리 |
| `/root` | root 사용자 홈 디렉터리 |
| `/home` | 일반 사용자 홈 디렉터리 |
| `/etc` | 시스템 설정 파일 |
| `/bin`, `/usr/bin` | 사용자 명령어 |
| `/sbin`, `/usr/sbin` | 시스템 관리 명령어 |
| `/var` | 로그, 스풀 등 가변 데이터 |
| `/tmp` | 임시 파일 (재부팅 시 삭제) |
| `/dev` | 장치 파일 |
| `/proc` | 프로세스/커널 가상 파일시스템 |
| `/sys` | 커널 하드웨어 정보 |
| `/boot` | 커널, 부트로더 파일 |
| `/lib`, `/usr/lib` | 라이브러리 |
| `/opt` | 추가 소프트웨어 |
| `/mnt`, `/media` | 임시 마운트 포인트 |

**정리**: **FHS**는 리눅스의 표준 디렉터리 구조로, `/etc`(설정), `/var`(가변 데이터), `/proc`·`/sys`(커널 정보) 등 각 디렉터리의 역할이 명확히 구분되어 있다.

---

## 패키지 관리 (dnf)

```bash
# 패키지 설치
dnf install -y httpd

# 패키지 삭제
dnf remove -y httpd

# 패키지 업데이트
dnf update -y httpd
dnf update -y          # 전체 업데이트

# 패키지 검색
dnf search httpd

# 패키지 정보
dnf info httpd

# 설치된 패키지 목록
rpm -qa
rpm -qa | grep httpd

# 패키지 파일 확인
rpm -ql httpd
```

**정리**: **dnf**는 Red Hat 계열의 패키지 관리자로 설치·삭제·업데이트·검색을 담당하며, `rpm`으로 설치된 패키지 상세 정보를 조회할 수 있다.

---

## 기본 시스템 명령어

```bash
# 시스템 정보
uname -a            # 전체 시스템 정보
hostname            # 호스트명 확인
uptime              # 가동 시간 및 로드
who                 # 현재 로그인 사용자
w                   # 로그인 사용자 상세
last                # 최근 로그인 기록

# 프로세스
ps -ef              # 전체 프로세스
ps aux              # BSD 스타일
top                 # 실시간 프로세스 모니터링
htop                # top의 개선 버전
kill PID            # 프로세스 종료
kill -9 PID         # 강제 종료

# 네트워크
ip addr             # IP 주소 확인
ss -tuln            # 포트 확인
ping IP             # 네트워크 연결 테스트
netstat -tuln       # 네트워크 상태

# 디스크/메모리
df -h               # 디스크 사용량
du -sh 경로         # 디렉터리 크기
free -h             # 메모리 사용량
lsblk               # 블록 장치 목록
```

**정리**: 시스템 정보, 프로세스, 네트워크, 디스크/메모리 확인을 위한 기본 명령어들을 카테고리별로 정리했다.

---

## GNU 프로젝트 상세

| 항목 | 내용 |
|------|------|
| 시작 | 1984년 리처드 스톨만(Richard Stallman) |
| FSF | 1985년 자유 소프트웨어 재단 설립 |
| 라이선스 | GPL (General Public License) |
| 핵심 | 카피레프트(Copyleft) — 재배포 시에도 GPL 유지 |

- **프리웨어(Freeware)**: 단순히 무료 소프트웨어
- **자유 소프트웨어(Free SW)**: 실행·연구·수정·재배포의 자유를 보장
- 유료 판매도 가능하지만 판매자는 소스 코드를 공개해야 하며, 구매자는 다시 무료 배포 가능

**정리**: **GNU 프로젝트**는 리처드 스톨만이 시작했으며, **카피레프트** 원칙에 따라 GPL 라이선스로 소스 공개 의무를 유지한다.

---

## 커널 버전 역사

| 연도 | 버전 | 의미 |
|------|------|------|
| 1991 | 0.01 | 리누스 토르발스 최초 공개 |
| 1994 | 1.0 | 정식 릴리스 |
| 이후 | 2.x → 3.x → 4.x → 5.x → 6.x | 세대별 발전 |

버전 체계 예시 `5.15.83`:
- 주 버전(Major) `5` — 큰 변화, 아키텍처 변경
- 부 버전(Minor) `15` — 기능 개선, 하드웨어 지원 추가
- 패치 버전(Patch) `83` — 버그 수정, 보안 패치

**정리**: 커널 버전은 `주.부.패치` 형식으로 표기되며, 1991년 첫 공개 이후 세대별로 발전해왔다.

---

## 쉘 동작 흐름 (ls -l 입력 시)

```bash
1) 쉘이 명령어 문법 분석
2) 커널에게 "이 디렉터리의 파일 목록 읽어라" 요청
3) 커널이 디스크에서 정보를 읽어 쉘에 전달
4) 쉘이 보기 좋은 형태로 화면 출력
```

**정리**: 명령 실행은 쉘의 문법 분석 → 커널 요청 → 디스크 접근 → 결과 출력의 순서로 이루어진다.

---

## Red Hat 계열 계보

| 배포판 | 특징 |
|--------|------|
| Fedora | 실험용/테스트베드 → RHEL의 베타 성격 (6개월 릴리스) |
| RHEL | 상용 배포판(유료) → 기업 표준 (2003년 이후 재편) |
| CentOS | 무료 RHEL 클론 → 2021년 지원 종료, CentOS Stream으로 전환 |
| CentOS Stream | RHEL 베타 단계 → 무료지만 기업용은 불안정 |
| Rocky Linux | 무료 RHEL 클론(현재) → CentOS 철학 계승, Bug-for-Bug Compatible |
| AlmaLinux | 무료 RHEL 클론(현재) → 비영리 재단 관리 |

**정리**: Fedora → RHEL → CentOS(종료) → Rocky Linux/AlmaLinux로 이어지는 Red Hat 계열의 계보를 보여준다.

---

## VMware Workstation Pro vs Player

| 항목 | Pro | Player |
|------|-----|--------|
| 라이선스 | 유료 | 유료 또는 무료 |
| 가상머신 생성 | O | O |
| 스냅숏 기능 | O | X |
| 가상 네트워크 설정 | O | X |

**정리**: **Pro**는 스냅숏과 가상 네트워크 설정을 지원하는 반면 **Player**는 기본 가상머신 생성 기능만 제공한다.

---

## 실습용 VM 구성표

| 항목 | Server-A | Server-B | Client | WinClient |
|------|----------|----------|--------|-----------|
| 게스트OS | RHEL9 64 | RHEL9 64 | RHEL9 64 | Win10 64 |
| ISO | Rocky9 | Rocky9 | Rocky9 | Win10평가 |
| 하드용량 | 80GB | 40GB | 40GB | 60GB |
| 하드타입 | SCSI | SCSI | SCSI/NVMe | SCSI/NVMe |
| 메모리 | 2GB | 2GB | 2GB | 2GB |
| 네트워크 | NAT | NAT | NAT | NAT |
| Audio/USB/Printer | X | X | X | X |

**정리**: 실습 환경은 Server-A/B, Client, WinClient 4대의 VM으로 구성되며 모두 NAT 네트워크를 사용한다.

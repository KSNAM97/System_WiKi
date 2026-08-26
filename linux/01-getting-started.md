# Linux-01 리눅스 시작

## 목차

1. [리눅스의 개요](#리눅스의-개요)
2. [GNU 프로젝트 ("GNU"s Not Unix Project)](#gnu-프로젝트-gnus-not-unix-project)
3. [가상머신과 가상머신 소프트웨어의 개념](#가상머신과-가상머신-소프트웨어의-개념)
4. [VMware 특징](#vmware-특징)
5. [커널(Kernel)](#커널kernel)
6. [쉘(Shell)](#쉘shell)
7. [Red Hat 계열 리눅스](#red-hat-계열-리눅스)
8. [가상머신 종류와 VMware 설치](#가상머신-종류와-vmware-설치)
9. [리눅스 기본 세팅](#리눅스-기본-세팅)
10. [SSH 권한 허용 변경 (root 계정으로 SSH 접속)](#ssh-권한-허용-변경-root-계정으로-ssh-접속)
11. [가상 콘솔 접속](#가상-콘솔-접속)
12. [서버 종료](#서버-종료)
13. [서버 예약 종료](#서버-예약-종료)
14. [서버 예약 종료 취소](#서버-예약-종료-취소)
15. [서버 종료 메세지 전송](#서버-종료-메세지-전송)
16. [지정된 시간에 서버 종료](#지정된-시간에-서버-종료)

## 리눅스의 개요

리눅스는 서버 환경 구축, 네트워크 설정, 원격 접속, 시스템 점검 등 다양한 실습과 운영 현장에서 널리 쓰이는 운영체제이다.

- **리눅스(Linux)** = 무료 유닉스(Unix-like OS)
  - 기존의 상용 유닉스 운영체제를 모방하여 누구나 무료로 사용할 수 있도록 만든 운영체제
  - 오픈소스(Open Source) 기반이어서 전 세계 개발자들이 자유롭게 개선하고 배포 가능

- **1991년 리누스 토르발스(Linus Torvalds)**
  - 헬싱키 대학교 학생이던 리누스 토르발스가 처음 공개한 버전은 Linux 0.01 (학습·연구용)
  - 같은 해 0.02 버전을 공개하면서 본격적으로 개발자 커뮤니티에 확산됨

- **리눅스의 구성**
  - 리누스 토르발스는 커널(Kernel)만 개발
  - 커널은 하드웨어와 소프트웨어를 중재하는 핵심 부분임
  - 이후 GNU 프로젝트에서 개발된 각종 도구(컴파일러, 쉘, 라이브러리 등)가 결합 → 오늘날의 완전한 리눅스 운영체제 배포판 탄생

- **리눅스 배포판**
  - 커널 + GNU 도구 + 추가 유틸리티를 조합하여 다양한 배포판이 만들어짐
  - 예: Red Hat, CentOS, Rocky Linux, Ubuntu, Debian, Fedora, SUSE 등

**정리**: **리눅스**는 1991년 리누스 토르발스가 개발을 시작한 오픈소스 커널이며, GNU 도구와 결합해 오늘날의 다양한 배포판으로 발전했다.

---

## GNU 프로젝트 ("GNU"s Not Unix Project)

- **시작 배경**
  - 1984년 리처드 스톨만(Richard Stallman)에 의해 시작
  - 목표: 모두가 공유할 수 있는 자유 소프트웨어 개발
  - 당시 상용 소프트웨어는 소스 코드 접근이 제한되어 있었다. (사용자가 프로그램을 고치거나 공유할 자유가 없었다.)

- **자유 소프트웨어 재단 (FSF, Free Software Foundation)**
  - 1985년 스톨만이 설립
  - 목적: 자유롭게 사용할 수 있는 소프트웨어 개발과 배포 지원
  - 철학: 소프트웨어는 단순히 상품이 아니라 인류가 공유해야 할 지식이라는 가치관

- **GNU 프로젝트의 핵심 목표**
  - GNU 프로젝트로 제작된 소프트웨어를 활용해, 컴퓨터 프로그램의 복제, 변경, 소스 코드 사용에 대한 제한을 철폐하는 것
  - 즉, 소프트웨어를 누구나 실행·연구·수정·배포할 수 있도록 보장

- **GPL (General Public License)**
  - GNU 프로젝트 소프트웨어는 GPL 라이선스를 따름
  - 특징:
    - 프로그램의 수정과 공유의 자유 보장
    - 단, 수정 후 재배포할 때도 반드시 동일한 GPL 라이선스를 유지해야 함 (카피레프트 Copyleft)
    - 이를 통해 자유 소프트웨어가 계속 자유롭게 유지될 수 있도록 한다.

- **프리웨어와의 차이**
  - 프리웨어(Freeware) = 무료 소프트웨어
  - 자유 소프트웨어(Free Software) = 자유(Freedom)를 중시 (실행, 소스 코드 접근, 수정, 재배포에 대한 자유)
  - 따라서 자유 소프트웨어는 단순히 무료가 아니라 사용자 권리 중심의 개념

- **자유 소프트웨어의 판매**
  - 자유 소프트웨어는 무료로 배포 가능하지만, 동시에 유료로 판매도 가능
  - 단, 판매자가 소스 코드를 반드시 공개해야 하며, 구매자는 다시 무료로 배포할 수 있다.
  - 이 구조 덕분에 Red Hat 같은 기업도 GNU/Linux를 상용으로 패키징해 서비스 제공 가능

**정리**: **GNU 프로젝트**는 리처드 스톨만이 주도한 자유 소프트웨어 운동이며, **GPL**과 **카피레프트** 원칙으로 소스 공개와 재배포의 자유를 보장한다.

---

## 가상머신과 가상머신 소프트웨어의 개념

### 가상머신(Virtual Machine, VM)의 기본 개념

- 가상머신은 물리적 컴퓨터(호스트) 안에서 가상의 컴퓨터(게스트)를 만들어 실행하는 기술
- 하드웨어 자원을 가상화하여, 독립된 컴퓨터처럼 사용할 수 있게 해준다
- 호스트 운영체제(Host OS) 위에 가상머신 소프트웨어(VMware, VirtualBox, Hyper-V 등)를 설치하고, 그 안에서 게스트 운영체제(Guest OS)를 구동
- 하나의 PC 안에서 여러 운영체제를 동시에 사용할 수 있게 해준다
- 각 가상머신은 독립된 CPU, 메모리, 디스크를 할당받은 것처럼 동작함
- 테스트, 실습, 개발, 보안 격리 등의 목적에 매우 유리함

### 호스트 OS와 게스트 OS

- **호스트 OS (Host Operating System)**
  - 실제 컴퓨터에 설치되어 사용자가 기본적으로 사용하는 운영체제
  - 가상머신 소프트웨어가 설치되어 있으며, 이를 통해 게스트 OS 실행 가능
  - 예: Windows 11, macOS, Linux 등

- **게스트 OS (Guest Operating System)**
  - 가상머신에 설치되어 가상 환경에서 실행되는 운영체제
  - 물리적 하드웨어 없이도 OS 설치 및 운영이 가능
  - 예: Ubuntu, Rocky Linux, Windows Server 등
  - 호스트 OS와 완전히 독립적으로 동작
  - 서로 다른 게스트 OS를 여러 개 동시에 실행할 수 있음

### 멀티부팅(Multi-Booting)과의 차이

- **멀티부팅**
  - PC 전원을 켤 때 어떤 운영체제를 실행할지 사용자가 직접 선택
  - 한 번에 하나의 운영체제만 실행 가능
  - 운영체제 전환 시 재부팅 필요
  - 각 운영체제는 하드디스크의 별도 파티션에 설치됨

- **가상머신**
  - 호스트 OS 위에서 여러 게스트 OS를 동시에 실행 가능
  - 운영체제 간 전환이 매우 빠름 (창 전환만으로 가능)
  - 게스트 OS를 실험·학습·테스트 용도로 빠르게 실행 가능
  - 네트워크, 저장소, CPU 자원도 개별로 제어 가능

### 하이퍼바이저 (Hypervisor)

- 가상머신을 생성하고 실행·관리하는 가상화 소프트웨어
- 호스트 OS의 하드웨어(CPU, RAM, Disk)를 가상으로 나누어 게스트 OS에 제공하는 핵심 역할
- 호스트 OS와 게스트 OS를 연결하는 가상화 엔진

- 예:
  - VMware Workstation / VMware Player
  - VirtualBox
  - Hyper-V

- **주요 기능**
  - 가상머신 생성 및 관리
  - CPU, 메모리, 디스크 등 자원 할당
  - 네트워크(브리지, NAT, 호스트온리) 구성
  - 스냅샷, 클론, 일시 정지(Suspend) 기능 제공

**정리**: **가상머신**은 하이퍼바이저를 통해 하나의 호스트 위에서 여러 게스트 OS를 동시에 실행하며, 재부팅이 필요한 멀티부팅과 달리 빠른 전환과 독립적 자원 제어가 가능하다.

---

## VMware 특징

- **실무와 유사한 환경 구성 가능**
  - 한 대의 실제 컴퓨터만으로 여러 대의 가상머신(VM)을 만들 수 있다.
  - 만든 VM들을 가상 네트워크로 연결하면 실제 기업 환경과 거의 동일한 구조를 만들 수 있다.
  - 서버용 VM + 클라이언트용 VM + 방화벽 역할 VM 등 다양한 조합을 구성할 수 있다.
  - 개발, 보안, 네트워크, 서버 실습에 매우 적합

- **스냅숏(Snapshot) 기능 제공**
  - 운영체제의 특정 시점 전체를 그대로 저장하는 기능이다.
  - 시스템 오류, 설정 실수, 패키지 충돌 등이 발생했을 때 저장한 지점으로 즉시 복구 가능하다.
  - 여러 개의 스냅숏을 단계별(설치 전 → 설치 후 → 설정 완료 등)로 관리할 수 있다.
  - "되돌리기" 기능 덕분에 실험적인 작업도 안전하게 시도할 수 있다.

- **여러 하드웨어 장착 및 테스트 가능**
  - 실제 PC처럼 CPU, 메모리, 디스크, 네트워크 어댑터를 추가하거나 제거할 수 있다.
  - 고급 실습(예: LVM, RAID, NAT/브리지/호스트온리 네트워크 구조 구성)이 가상환경만으로도 가능하다.
  - 기업에서 사용하는 서버 환경(다중 NIC, 다중 디스크, 라우팅 테스트 등)을 그대로 재현할 수 있다.
  - 별도 장비 구매 없이 다양한 하드웨어 구성을 실습할 수 있다는 점이 큰 장점이다.

- **Suspend 기능 (일시 중단 및 재개)**
  - 가상머신의 현재 상태(메모리, 실행 중인 프로그램, 열려 있는 파일)를 통째로 저장하는 기능이다.
  - 다음 실행 시 부팅 과정 없이 즉시 이전 작업 상태로 복원된다.
  - 컴퓨터를 껐다 켜더라도 VM은 "중단했던 그 순간" 그대로 돌아온다.
  - 강의·실습 중 작업을 잠시 멈추고 싶을 때 매우 유용하다.

- **호스트와 독립적인 환경 유지**
  - 게스트 OS는 호스트 OS에 영향을 거의 받지 않는 독립된 컴퓨터처럼 동작한다.
  - 실수로 게스트 OS를 망가뜨려도 호스트 OS에는 영향이 없다.
  - 실험, 해킹 방어 실습, 서버 설정 실습 등 위험한 작업도 안전하게 진행할 수 있다.

**정리**: VMware는 **스냅숏**, **Suspend**, 독립적인 게스트 환경을 통해 실무와 유사한 실습 환경을 안전하게 구성할 수 있게 해준다.

---

## 커널(Kernel)

- **커널(Kernel)**
  - 운영체제의 핵심(심장부)로, 하드웨어와 응용 프로그램 사이를 중재하는 소프트웨어
  - CPU, 메모리, 디스크, 네트워크 장치를 직접 관리하고, 사용자 프로그램이 안전하게 실행되도록 지원
  - 리눅스 커널은 전 세계 개발자들이 함께 개발하는 오픈소스 프로젝트

- **주요 기능**
  - 프로세스 관리: 프로그램 실행·종료, CPU 자원 분배
  - 메모리 관리: 메모리 할당과 회수, 가상 메모리 관리
  - 파일 시스템 관리: 파일 읽기·쓰기, 디스크 관리
  - 입출력 장치 관리: 키보드, 마우스, 네트워크 장치 제어

- **예시**
  - 게임 실행 시: CPU, 그래픽카드, 메모리 할당 등을 커널이 다 조율한다.
  - 여러 프로그램이 동시에 실행될 때 충돌 없이 자원을 나눠 쓰도록 제어

- **커널 버전 체계 (예: 5.15.83)**
  - 주 버전 (Major Version) : 5 → 큰 변화, 아키텍처 변경, 새로운 기능 추가
  - 부 버전 (Minor Version) : 15 → 기능 개선, 하드웨어 지원 추가
  - 패치 버전 (Patch Version) : 83 → 버그 수정, 보안 패치, 작은 안정성 개선

- **커널 버전 역사**
  - 1991년 리눅스 커널 0.01 공개 (리누스 토르발스에 의해 개발 시작)
  - 1994년 1.0 정식 버전 릴리즈
  - 이후 꾸준히 발전: 2.x → 3.x → 4.x → 5.x → 현재는 6.x 세대
  - 장기 지원(LTS) 버전과 최신 개발 버전이 함께 제공됨

- **배포판과 커널**
  - 배포판마다 기본 탑재 커널 버전이 다르다.
  - 예: Rocky Linux 9는 커널 5.14를 기본 포함
  - 하지만 사용자가 필요하면 `dnf update` 또는 직접 컴파일을 통해 최신 커널로 업그레이드 가능
  - 단, 기업 환경에서는 안정성과 호환성을 위해 배포판 기본 커널을 유지하는 경우가 많다.

**정리**: **커널**은 프로세스·메모리·파일시스템·입출력을 관리하는 운영체제의 핵심이며, 버전은 `주.부.패치` 체계로 관리되고 배포판마다 기본 탑재 버전이 다르다.

---

## 쉘(Shell)

- 사용자와 커널 사이의 인터페이스(중간 통역사)
  - 사용자가 입력한 명령을 해석하여 커널에게 전달하고
  - 커널이 처리한 결과를 다시 사용자에게 표시한다.
  - 즉, 사람 ↔ 쉘 ↔ 커널 구조

- **쉘의 역할**
  - 명령어 해석기(Command Interpreter)
  - 프로그램 실행 도우미
  - 스크립트 실행(자동화 기능)
  - 시스템 환경 관리

- **쉘 종류**
  - CLI 쉘 (Command Line Interface 쉘)
    - 예: bash, sh, zsh, ksh
    - 사용자 입력(명령어)을 텍스트로 해석 후 커널로 전달
    - 서버 환경에서 가장 많이 사용됨
  - GUI 쉘 (Graphical Shell)
    - 아이콘 클릭·마우스 조작을 명령으로 번역하는 쉘
    - GNOME Shell, KDE Plasma 등이 대표적

- **쉘 동작 예시**
  - 사용자가 "ls -l" 입력
    1. 쉘이 명령어 문법 분석
    2. 커널에게 "이 디렉터리의 파일 목록 읽어라" 요청
    3. 커널이 디스크에서 정보를 읽어 쉘에 전달
    4. 쉘이 화면에 보기 좋은 형태로 출력

**정리**: **쉘**은 사용자와 커널을 잇는 명령어 해석기로, CLI 쉘(bash 등)과 GUI 쉘로 나뉘며 명령 입력 → 커널 요청 → 결과 출력의 흐름으로 동작한다.

---

## Red Hat 계열 리눅스

### Red Hat Linux

- Red Hat 사에서 배포한 리눅스로, 가장 널리 알려진 상용 리눅스 배포판 중 하나
- Red Hat에서 파생된 운영체제들을 통틀어 "Red Hat 계열"이라고 부른다.
- 대표 계열: Red Hat Enterprise Linux(RHEL), CentOS, Fedora, Rocky Linux, AlmaLinux
- 2003년 이후 Red Hat Linux는 상용 모델인 RHEL 중심으로 재편
- Red Hat 계열의 특징
  - 기업 환경에서 가장 안정적으로 사용됨
  - 보안 패치, 커널 업데이트, 품질 검증이 빠르고 체계적임
  - 서버·클라우드·기업 인프라에서 사실상 표준(Standard)
  - SELinux, 시스템 관리 도구, 패키지 시스템(YUM/DNF)이 잘 정비되어 있음

### CentOS (Community Enterprise Operating System)

- RHEL 소스를 기반으로 만들어진 무료 배포판(RHEL Clone)
- Red Hat의 상표·로고만 제거하고 재컴파일한 형태

- 장점:
  - RHEL과 완전 100% 호환
  - 비용이 없어 기업·교육기관에서 폭넓게 사용됨

- 단점:
  - Red Hat의 상용 기술 지원이 없음
  - 보안 패치 시점이 RHEL보다 다소 늦음

- 과거에는 무료 리눅스 서버의 대표 선택지였으나
- 2021년 Red Hat이 CentOS 지원 종료 → CentOS Stream으로 정책 변경
- 기존 CentOS 사용자는 Rocky Linux, AlmaLinux로 대거 이동

### Rocky Linux

- CentOS 지원 종료 후, 이를 대체하기 위해 등장한 완전 무료 RHEL 기반 배포판
- CentOS 공동 창립자 Gregory Kurtzer가 직접 개발 주도
- RHEL과 "버그 레벨까지 동일한 완전한 호환(Bug-for-Bug Compatible)"을 목표로 제작
- 특징:
  - 무료·오픈소스
  - RHEL과 기능·커널·패키지 구성까지 1:1 호환
  - 서버 운영 환경에서 CentOS를 대체하는 대표적인 선택지
- 기업, 기관, 클라우드 환경에서 활발히 사용됨
- 안정성·호환성·패치 정책이 CentOS 시절과 매우 유사하여 인기가 높음

### AlmaLinux

- Rocky Linux와 함께 CentOS 종료 후 등장한 또 다른 무료 RHEL 클론 배포판
- 장기적 생태계를 위해 비영리 재단(AlmaLinux Foundation)에서 관리
- Rocky Linux와 함께 현재 RHEL 무료 대안의 양대 축

### Fedora

- Red Hat에서 지원하는 커뮤니티 기반 배포판
- 최신 기능과 기술을 가장 먼저 시험하는 "테스트베드"
- Fedora → 안정화 → RHEL → CentOS Stream → Rocky/Alma 구조로 이어짐
- 특징:
  - 최신 기능 실험, 신기술 테스트용
  - 릴리스 주기 6개월(가장 빠름)
  - 개발자·연구용으로 많이 활용
  - 데스크탑 환경 구성도 매우 우수함

### 배포판 비교표

| 배포판 | 성격 | 특징 | 현재 위치 |
|--------|------|------|-----------|
| Fedora | 실험용, 테스트베드 | 최신 기능 먼저 반영 | RHEL의 베타 성격 |
| RHEL | 상용 배포판 | 안정성·지원 보장, 유료 | 기업 표준 |
| CentOS | 무료 RHEL 클론 (과거) | RHEL 8까지 대표적 무료 대안 | CentOS Stream으로 변경 |
| CentOS Stream | RHEL 베타 단계 | 정식 RHEL 전 단계 | 무료지만 기업용으로 불안정 |
| Rocky Linux | 무료 RHEL 클론 (현재) | CentOS의 철학 계승, 완전 호환 | 기업용 무료 대안 |

**정리**: Red Hat 계열은 Fedora(실험) → RHEL(상용) → CentOS(종료) → **Rocky Linux**/**AlmaLinux**(현재 무료 대안)로 이어지는 계보를 가진다.

---

## 가상머신 종류와 VMware 설치

### VMware Workstation Pro vs Player

| 항목 | VMware Workstation Pro | VMware Workstation Player |
|------|------------------------|---------------------------|
| 호스트 OS | Windows 10 이상 64bit | Windows 10 이상 64bit |
| 게스트 OS | 대부분의 32bit, 64bit Windows / 대부분의 리눅스 | 대부분의 32bit, 64bit Windows / 대부분의 리눅스 |
| 라이선스 | 유료 | 유료 또는 무료 |
| 라이선스 키 | 구매 필요 | 무료 라이선스의 경우 키 필요 없음 |
| 가상머신 생성 기능 | O | O |
| 스냅숏 기능 | O | X |
| 가상 네트워크 사용자 설정 기능 | O | X |

### 실습 VM 구성 계획

| 항목 | Server-A | Server-B | Client | WinClient |
|------|----------|----------|--------|-----------|
| 주 용도 | 서버 전용 | 서버 전용 (테스트 용도) | 클라이언트 전용 | Windows 클라이언트 전용 |
| 게스트 OS 종류 | Red Hat Enterprise Linux 9 64-bit | Red Hat Enterprise Linux 9 64-bit | Red Hat Enterprise Linux 9 64-bit | Windows 10 64bit |
| 설치할 ISO | Rocky Linux 9 | Rocky Linux 9 | Rocky Linux 9 | Windows 10 평가판 (64bit) |
| 가상머신 이름 | Server-A | Server-B | Client | WinClient |
| 저장 폴더 | C:\Rocky9\Server-A | C:\Rocky9\Server-B | C:\Rocky9\Client | C:\Rocky9\WinClient |
| 하드 용량 | 80GB | 40GB | 40GB | 60GB |
| 하드 타입 | SCSI | SCSI | SCSI or NVMe | SCSI or NVMe |
| 메모리 할당 | 2GB | 2GB | 2GB | 2GB |
| 네트워크 타입 | NAT | NAT | NAT | NAT |
| CD/DVD 장치 | O | O | O | O |
| Audio 장치 | X | X | X | X |
| USB 장치 | X | X | X | X |
| Printer 장치 | X | X | X | X |

**정리**: **Pro**와 **Player**의 기능 차이(스냅숏, 가상 네트워크 설정)를 확인하고, Server-A/B/Client/WinClient 4대의 VM 구성 계획표로 실습 환경을 준비한다.

---

## 리눅스 기본 세팅

### Server-A 기본 설정

**root 패스워드**: admin1234

#### IP 주소 확인

```bash
[root@localhost ~]# ifconfig
ens160: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.10.128  netmask 255.255.255.0  broadcast 192.168.10.255
        inet6 fe80::20c:29ff:fe66:2f9c  prefixlen 64  scopeid 0x20<link>
        ether 00:0c:29:66:2f:9c  txqueuelen 1000  (Ethernet)
        RX packets 1359637  bytes 2033121351 (1.8 GiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 405815  bytes 22053872 (21.0 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 37  bytes 3398 (3.3 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 37  bytes 3398 (3.3 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

```bash
[root@localhost ~]# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: ens160: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:0c:29:66:2f:9c brd ff:ff:ff:ff:ff:ff
    altname enp3s0
    inet 192.168.10.128/24 brd 192.168.10.255 scope global dynamic noprefixroute ens160
       valid_lft 1705sec preferred_lft 1705sec
    inet6 fe80::20c:29ff:fe66:2f9c/64 scope link noprefixroute
       valid_lft forever preferred_lft forever
```

```bash
[root@localhost ~]# ip addr show ens160
2: ens160: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:0c:29:66:2f:9c brd ff:ff:ff:ff:ff:ff
    altname enp3s0
    inet 192.168.10.128/24 brd 192.168.10.255 scope global dynamic noprefixroute ens160
       valid_lft 1674sec preferred_lft 1674sec
    inet6 fe80::20c:29ff:fe66:2f9c/64 scope link noprefixroute
       valid_lft forever preferred_lft forever
```

#### 고정 IP 설정 (nmconnection 파일 편집)

```bash
[root@localhost ~]# cd /etc/NetworkManager/system-connections/
[root@localhost system-connections]# gedit ens160.nmconnection
```

```ini
[connection]
id=ens160
uuid=5989e9d3-fb91-3508-90e8-2ecc49b5b5d3
type=ethernet
autoconnect-priority=-999
interface-name=ens160
timestamp=1782996029

[ethernet]

[ipv4]
method=manual
address1=192.168.10.100/24,192.168.10.2
dns=192.168.10.2

[ipv6]
addr-gen-mode=eui64
method=auto

[proxy]
```

```bash
[root@localhost system-connections]# nmcli  connection  down  ens160
'ens160' 연결이 성공적으로 비활성화되었습니다 (D-Bus 활성 경로: /org/freedesktop/NetworkManager/ActiveConnection/1)

[root@localhost system-connections]# nmcli  connection  up  ens160
연결이 성공적으로 활성화되었습니다 (D-버스 활성 경로: /org/freedesktop/NetworkManager/ActiveConnection/2)

[root@localhost system-connections]# reboot    <--- IP 주소 변경 안되면 재부팅
```

```bash
[root@localhost ~]# ifconfig
ens160: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.10.100  netmask 255.255.255.0  broadcast 192.168.10.255
        inet6 fe80::20c:29ff:fe66:2f9c  prefixlen 64  scopeid 0x20<link>
        ether 00:0c:29:66:2f:9c  txqueuelen 1000  (Ethernet)
        RX packets 32  bytes 4309 (4.2 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 50  bytes 5304 (5.1 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

#### 패키지 업데이트

```bash
[root@localhost ~]# dnf  -y  update
```

#### SELinux 상태 확인

```bash
[root@localhost ~]# sestatus
SELinux status:                     enabled
SELinuxfs mount:                /sys/fs/selinux
SELinux root directory:         /etc/selinux
Loaded policy name:             targeted
Current mode:                       enforcing
Mode from config file:              enforcing
Policy MLS status:                  enabled
Policy deny_unknown status:     allowed
Memory protection checking:     actual (secure)
Max kernel policy version:      33
```

#### SELinux 비활성화

**방법 1 — grubby 사용**

```bash
[root@localhost ~]# grubby  --update-kernel ALL --args selinux=0
```

**방법 2 — config 파일 편집**

```bash
[root@localhost ~]# gedit /etc/selinux/config
```

```bash
# This file controls the state of SELinux on the system.
# SELINUX= can take one of these three values:
#     enforcing - SELinux security policy is enforced.
#     permissive - SELinux prints warnings instead of enforcing.
#     disabled - No SELinux policy is loaded.
# See also:
# https://docs.fedoraproject.org/en-US/quick-docs/getting-started-with-selinux/#getting-started-with-selinux-selinux-states-and-modes
#
# NOTE: In earlier Fedora kernel builds, SELINUX=disabled would also
# fully disable SELinux during boot. If you need a system with SELinux
# fully disabled instead of SELinux running with no policy loaded, you
# need to pass selinux=0 to the kernel command line. You can use grubby
# to persistently set the bootloader to boot with selinux=0:
#
#    grubby --update-kernel ALL --args selinux=0
#
# To revert back to SELinux enabled:
#
#    grubby --update-kernel ALL --remove-args selinux
#
SELINUX=disabled        <--- enforcing를 disabled로 변경
# SELINUXTYPE= can take one of these three values:
#     targeted - Targeted processes are protected,
#     minimum - Modification of targeted policy. Only selected processes are protected.
#     mls - Multi Level Security protection.
SELINUXTYPE=targeted
```

```bash
[root@localhost ~]# reboot    : 재부팅

[root@localhost ~]# sestatus
SELinux status:                 disabled    <--- SELINUX 해제 확인

[root@localhost ~]# getenforce
Disabled                        <--- SELINUX 해제 확인
```

```bash
[root@localhost ~]# dnf  update  -y
[root@localhost ~]# reboot    : 재부팅
```

**정리**: **Server-A**는 `ifconfig`/`ip addr`로 IP를 확인하고, nmconnection 파일 수정으로 고정 IP를 설정한 뒤 grubby 또는 config 파일 편집으로 SELinux를 비활성화한다.

---

### Server-B 기본 설정

```bash
localhost login: root
Password: admin1234

[root@localhost ~]# dnf  install  -y  bind-utils  net-tools

[root@localhost ~]# ifconfig

[root@localhost ~]# vi  /etc/selinux/config
```

방법 1 또는 방법 2로 SELinux 비활성화 후:

```bash
[root@localhost ~]# dnf  update  -y

[root@localhost ~]# reboot    : 재부팅

[root@localhost ~]# getenforce
Disabled                <--- SELINUX 해제 확인
```

**정리**: **Server-B**도 동일하게 SELinux 비활성화 후 패키지 업데이트와 재부팅으로 설정을 완료한다.

---

### Client-L 기본 설정

```bash
[root@localhost ~]# grubby  --update-kernel ALL --args selinux=0
```

또는 `/etc/selinux/config` 편집으로 `SELINUX=disabled` 설정 후:

```bash
[root@localhost ~]# dnf  update  -y

[root@localhost ~]# reboot    : 재부팅

[root@localhost ~]# getenforce
Disabled                <--- SELINUX 해제 확인
```

**정리**: **Client-L**도 grubby 또는 config 편집 방식으로 동일하게 SELinux를 비활성화한다.

---

## SSH 권한 허용 변경 (root 계정으로 SSH 접속)

```bash
[root@localhost ~]# gedit /etc/ssh/sshd_config
```

```bash
# Authentication:

#LoginGraceTime 2m
PermitRootLogin yes    <--- 주석 제거 (root SSH 로그인 허용)
#StrictModes yes
#MaxAuthTries 6
#MaxSessions 10
```

```bash
[root@localhost ~]# systemctl restart sshd
```

**정리**: `sshd_config`에서 **PermitRootLogin yes**로 변경하고 `systemctl restart sshd`로 적용하면 root 계정으로 SSH 직접 접속이 가능해진다.

---

## 가상 콘솔 접속

- Rocky Linux(및 대부분의 리눅스)는 텍스트 콘솔과 그래픽(데스크탑)을 분리해서 쓸 수 있다.
- 키보드 단축키로 여러 가상 콘솔(tty)을 전환해 시스템 점검/복구/서버 관리 등을 할 수 있다.

- **사용법**
  - 그래픽(GUI)으로 돌아가기: 보통 `Ctrl + Alt + F1`
  - 텍스트 콘솔(tty2~tty6)로 전환: `Ctrl + Alt + F2` ~ `Ctrl + Alt + F6`
  - 콘솔에선 로그인 프롬프트가 나오며 계정(예: root)과 비밀번호로 로그인

**정리**: **가상 콘솔(tty)**은 `Ctrl+Alt+F2~F6`으로 전환하며, GUI 없이도 별도의 로그인 세션으로 시스템을 점검할 수 있다.

---

## 서버 종료

```bash
[root@localhost ~]# shutdown now

[root@localhost ~]# halt -p
```

**정리**: `shutdown now` 또는 `halt -p`로 서버를 즉시 종료할 수 있다.

---

## 서버 예약 종료

```bash
[root@localhost ~]# shutdown -P +10    <-- 10분후 시스템 종료
```

접속한 사용자들에게 메세지 자동 전송:
```bash
Broadcast message from root@localhost on pts/1 (Fri 2026-07-03 10:39:32 KST):

The system will power off at Fri 2026-07-03 10:49:32 KST!
```

**정리**: `shutdown -P +분` 형태로 지정한 시간 뒤 전원을 끄도록 예약할 수 있으며, 접속 사용자에게 자동으로 안내 메시지가 전송된다.

---

## 서버 예약 종료 취소

```bash
[root@localhost ~]# shutdown -c    <-- 예약 종료 취소
```

접속한 사용자들에게 메세지 자동 전송:
```bash
Broadcast message from root@localhost on pts/1 (Fri 2026-07-03 10:40:41 KST):

System shutdown has been cancelled
```

**정리**: `shutdown -c`로 예약된 종료를 취소할 수 있으며, 취소 사실도 사용자에게 자동 안내된다.

---

## 서버 종료 메세지 전송

- 사용자들에게 서버 종료 메세지 전송 (실제 서버가 종료되지는 않고 메세지만 전송)

```bash
[root@localhost ~]# shutdown -k +30
Shutdown scheduled for Fri 2026-07-03 11:14:19 KST, use 'shutdown -c' to cancel.
```

**정리**: `shutdown -k +분`은 실제 종료 없이 경고 메시지만 전송하는 테스트용 옵션이다.

---

## 지정된 시간에 서버 종료

```bash
[root@localhost ~]# shutdown -r 23:00
Reboot scheduled for Fri 2026-07-03 23:00:00 KST, use 'shutdown -c' to cancel.
```

**정리**: `shutdown -r HH:MM` 형식으로 특정 시각에 재부팅을 예약할 수 있다.

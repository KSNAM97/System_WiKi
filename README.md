# System Wiki

> Rocky Linux 9 기반 시스템 관리, 쉘 스크립트, 데이터베이스, HTML, Docker, Kubernetes, AWS 기술 문서 모음

이 문서는 리눅스 시스템 관리부터 컨테이너 오케스트레이션, 클라우드까지, 실무에서 바로 참고할 수 있도록 정리한 시스템 엔지니어링 문서이다. 총 7개 카테고리, 53개 문서로 구성되어 있다.

## 구성

### 🐧 Linux (14)

Rocky Linux 9 기반 리눅스 시스템 관리 핵심 정리

- [개요 · GNU · 커널 · 쉘](linux/00-overview.md)
- [리눅스 시작 · 기본 세팅](linux/01-getting-started.md) — IP 설정, SELinux, SSH 초기 설정
- [기본 명령어](linux/02-basic-commands.md) — cd · ls · cp · mv · rm · grep
- [VI 편집기](linux/03-vi-editor.md)
- [사용자 계정 · Group · SUDO](linux/04-user-management.md)
- [허가권 · 소유권 · 특수권한](linux/05-permissions.md) — chmod · chown · umask · SUID/SGID/Sticky
- [압축 (gzip · tar)](linux/06-compression.md)
- [파티션 · 마운트](linux/07-partition-mount.md) — fdisk · mkfs · /etc/fstab
- [RAID · LVM · Disk Quota](linux/08-storage-raid-lvm.md)
- [NFS · Samba](linux/09-network-filesystems.md)
- [SSH · SCP · FTP · SFTP](linux/10-remote-access.md)
- [DHCP · DNS](linux/11-network-services.md)
- [Rocky Linux 9 vs Rocky Linux 10](linux/rocky9-vs-rocky10.md)
- [실습 문제](linux/practice.md)

### 💻 Shell Script (7)

Bash 쉘 스크립트 문법과 실전 활용

- [변수 · 환경변수](shell-script/01-variables.md)
- [Metacharacters](shell-script/02-metacharacters.md) — 글롭 · 중괄호 확장 · 명령치환
- [조건문 (if · case)](shell-script/03-conditions.md)
- [반복문 (for · while)](shell-script/04-loops.md)
- [cron · crond](shell-script/05-cron.md)
- [배열 · 위치 매개변수](shell-script/06-arrays-parameters.md)
- [실습 문제](shell/07-example-scripts.md)

### 🗄️ Database — MariaDB (6)

MariaDB/MySQL 설치부터 JOIN 실습까지

- [설치 · 계정 · 권한](database-mariadb/01-setup.md)
- [SQL 문법 · DDL · DML](database-mariadb/02-sql-syntax.md)
- [emp · dept 실습](database-mariadb/03-emp-dept.md)
- [제약조건 (PK · Unique · FK)](database-mariadb/04-constraint.md)
- [INNER JOIN 실습](database-mariadb/05-inner-join.md)
- [실습 문제](database-mariadb/practice.md)

### 🌐 HTML (1)

- [HTML 기초 · 태그 정리](html/basics.md)

### 🐳 Docker (8)

컨테이너 가상화부터 Compose까지

- [Docker 설치](docker/00-install.md)
- [도커 개요 · VM vs Container](docker/01-overview.md)
- [Dockerfile · 이미지 빌드](docker/02-container.md)
- [컨테이너 생명주기 · exec](docker/03-using-containers.md)
- [메모리 · CPU 자원 제한](docker/04-resource-limits.md)
- [Volume · Bind Mount](docker/05-storage.md)
- [docker0 · 포트포워딩 · 네트워크](docker/06-network.md)
- [YAML 문법 · Docker Compose](docker/07-yaml-compose.md)

### ☸️ Kubernetes (14)

클러스터 구성부터 AutoScaling까지 실무 오케스트레이션 전 과정

- [소개 · VM vs Container · K8s란](kubernetes/01-overview.md)
- [설치 (Docker · kubeadm · CNI)](kubernetes/02-installation.md)
- [Pod · Deployment 생성 및 관리](kubernetes/03-pod-creation.md)
- [아키텍처 · Namespace · ResourceQuota](kubernetes/04-architecture.md)
- [Pod 개념 · livenessProbe](kubernetes/05-pod-concepts.md)
- [Controller (RC·RS·Deploy·DS·SS·Job·CronJob)](kubernetes/06-controller.md)
- [Service (ClusterIP · NodePort · LoadBalancer · ExternalName · Headless)](kubernetes/07-service.md)
- [Readiness Probe](kubernetes/07-2-readiness-probe.md)
- [Ingress (경로 기반 라우팅 · Ingress Controller)](kubernetes/08-ingress.md)
- [Label · Label Selector · Node Label · nodeSelector](kubernetes/09-label.md)
- [Pod Scheduling (nodeSelector · Affinity · Taint&Toleration · Cordon/Drain)](kubernetes/10-pod-scheduling.md)
- [Storage (Volume · PV/PVC · StorageClass · NFS · Dynamic Provisioning)](kubernetes/11-storage.md)
- [ConfigMap · Secret](kubernetes/12-configmap-secret.md)
- [AutoScaling (HPA · VPA · Cluster Autoscaler)](kubernetes/13-autoscaling.md)

### ☁️ AWS (3)

Amazon Web Services 클라우드 기초 개념 및 핵심 서비스

- [클라우드 기초 개념](aws/01-cloud-basics.md) — EC2·IAM·VPC·S3·Route 53·RDS, IaaS/PaaS/SaaS, 고가용성, 리전·가용 영역
- [EC2 배포](aws/02-ec2-deployment.md) — 인스턴스·EBS·AMI·요금 모델, 보안 그룹, 접속 방법, 생명주기, User Data/Meta Data, IAM 역할, 수직/수평 확장, Auto Scaling, ELB·대상 그룹·리스너
- [로드 밸런서·HTTPS](aws/03-load-balancer-https.md) — ALB 생성 실습, 대상 그룹·헬스 체크, Route 53 도메인 연결, ACM 인증서 발급, HTTPS 리스너, HTTP→HTTPS 리다이렉트

왼쪽 목차(SUMMARY)를 따라 순서대로 읽거나, 필요한 주제로 바로 이동하여 참고할 수 있다.

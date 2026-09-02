# Table of contents

* [소개](README.md)

## 🐧 Linux

* [개요 · GNU · 커널 · 쉘](linux/00-overview.md)
* [리눅스 시작 · 기본 세팅](linux/01-getting-started.md)
* [기본 명령어 (cd · ls · cp · mv · rm)](linux/02-basic-commands.md)
* [VI 편집기](linux/03-vi-editor.md)
* [사용자 계정 · Group · SUDO](linux/04-user-management.md)
* [허가권 · 소유권 · 특수권한](linux/05-permissions.md)
* [압축 (gzip · tar)](linux/06-compression.md)
* [파티션 · 마운트](linux/07-partition-mount.md)
* [RAID · LVM · Disk Quota](linux/08-storage-raid-lvm.md)
* [NFS · Samba](linux/09-network-filesystems.md)
* [SSH · SCP · FTP · SFTP](linux/10-remote-access.md)
* [DHCP · DNS](linux/11-network-services.md)
* [Rocky Linux 9 vs Rocky Linux 10](linux/rocky9-vs-rocky10.md)
* [실습 문제](linux/practice.md)

## 💻 Shell Script

* [변수 · 환경변수](shell-script/01-variables.md)
* [Metacharacters](shell-script/02-metacharacters.md)
* [조건문 (if · case)](shell-script/03-conditions.md)
* [반복문 (for · while)](shell-script/04-loops.md)
* [cron · crond](shell-script/05-cron.md)
* [배열 · 위치 매개변수](shell-script/06-arrays-parameters.md)
* [실습 문제](shell/07-example-scripts.md)

## 🗄️ Database (MariaDB)

* [설치 · 계정 · 권한](database-mariadb/01-setup.md)
* [SQL 문법 · DDL · DML](database-mariadb/02-sql-syntax.md)
* [emp · dept 실습](database-mariadb/03-emp-dept.md)
* [제약조건 (PK · Unique · FK)](database-mariadb/04-constraint.md)
* [INNER JOIN 실습](database-mariadb/05-inner-join.md)
* [실습 문제](database-mariadb/practice.md)

## 🌐 HTML

* [HTML 기초 · 태그 정리](html/basics.md)

## 🐳 Docker

* [Docker 설치](docker/00-install.md)
* [도커 개요 · VM vs Container](docker/01-overview.md)
* [Dockerfile · 이미지 빌드](docker/02-container.md)
* [컨테이너 생명주기 · exec](docker/03-using-containers.md)
* [메모리 · CPU 자원 제한](docker/04-resource-limits.md)
* [Volume · Bind Mount](docker/05-storage.md)
* [docker0 · 포트포워딩 · 네트워크](docker/06-network.md)
* [YAML 문법 · Docker Compose](docker/07-yaml-compose.md)

## ☸️ Kubernetes

* [소개 · VM vs Container · K8s란](kubernetes/01-overview.md)
* [설치 (Docker · kubeadm · CNI)](kubernetes/02-installation.md)
* [Pod · Deployment 생성 및 관리](kubernetes/03-pod-creation.md)
* [아키텍처 · Namespace · ResourceQuota](kubernetes/04-architecture.md)
* [Pod 개념 · livenessProbe](kubernetes/05-pod-concepts.md)
* [Controller (RC·RS·Deploy·DS·SS·Job·CronJob)](kubernetes/06-controller.md)
* [Service (ClusterIP · NodePort · LoadBalancer · ExternalName · Headless)](kubernetes/07-service.md)
* [Readiness Probe](kubernetes/07-2-readiness-probe.md)
* [Ingress (경로 기반 라우팅 · Ingress Controller)](kubernetes/08-ingress.md)
* [Label · Label Selector · Node Label · nodeSelector](kubernetes/09-label.md)
* [Pod Scheduling (nodeSelector · Affinity · Taint&Toleration · Cordon/Drain)](kubernetes/10-pod-scheduling.md)
* [Storage (Volume · PV/PVC · StorageClass · NFS · Dynamic Provisioning)](kubernetes/11-storage.md)
* [ConfigMap · Secret](kubernetes/12-configmap-secret.md)
* [AutoScaling (HPA · VPA · Cluster Autoscaler)](kubernetes/13-autoscaling.md)

## ☁️ AWS

* [클라우드 기초 개념](aws/01-cloud-basics.md)
* [EC2 배포](aws/02-ec2-deployment.md)
* [로드 밸런서·HTTPS](aws/03-load-balancer-https.md)
* [탄력적 IP](aws/04-elastic-ip.md)

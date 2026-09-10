# System Wiki

> Rocky Linux 9 기반 시스템 관리, 쉘 스크립트, 데이터베이스, HTML, Docker, Kubernetes, AWS, Python 기술 문서 모음

이 문서는 리눅스 시스템 관리부터 컨테이너 오케스트레이션, 클라우드, 파이썬까지, 실무에서 바로 참고할 수 있도록 정리한 시스템 엔지니어링 문서이다. 총 7개 카테고리, 78개 문서로 구성되어 있다.

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

### 💻 Shell Script (14)

Bash 쉘 스크립트 문법과 실전 활용

- [변수 · 환경변수](shell-script/01-variables.md)
- [Metacharacters](shell-script/02-metacharacters.md) — 글롭 · 중괄호 확장 · 명령치환
- [조건문 (if · case)](shell-script/03-conditions.md)
- [반복문 (for · while)](shell-script/04-loops.md)
- [cron · crond](shell-script/05-cron.md)
- [배열 · 위치 매개변수](shell-script/06-arrays-parameters.md) — 슬라이싱(offset:length) 포함
- [함수 (Function)](shell-script/07-functions.md) — join 함수 구현 예제 포함
- [문법 총정리 (Syntax Master)](shell-script/08-syntax-master.md) — 콜론 유무 매개변수 확장 차이 포함
- [스크립트 실행 방법](shell-script/09-script-execution.md) — ./script.sh · bash/sh · source(.) 비교
- [대화형/비대화형 · 로그인/비로그인 쉘](shell-script/10-shell-types.md)
- [종료 상태 코드 심화](shell-script/11-exit-status-advanced.md) — 예약 코드, $PIPESTATUS, pipefail
- [test · [ · [[ 명령 심화](shell-script/12-test-command.md)
- [패턴 매칭 (Globbing)](shell-script/13-pattern-matching.md) — extglob 포함
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

### ☁️ AWS (15)

Amazon Web Services 클라우드 기초 개념 및 핵심 서비스

**이론**

- [클라우드 기초 개념](aws/t01-cloud-basics.md) — EC2·IAM·VPC·S3·Route 53·RDS, IaaS/PaaS/SaaS, 고가용성, 리전·가용 영역
- [EC2 배포](aws/t02-ec2-deployment.md) — 인스턴스·EBS·AMI·요금 모델, 보안 그룹, 접속 방법, 생명주기, User Data/Meta Data, IAM 역할, 수직/수평 확장, Auto Scaling, ELB·대상 그룹·리스너, EC2 모니터링(CloudWatch), Auto Scaling 정책·기타 기능, EFS, T 타입, EC2 사이즈 변경
- [VPC](aws/t03-vpc.md) — 사설망·NAT, CIDR·서브넷, VPC·라우트 테이블, 퍼블릭/프라이빗 서브넷·인터넷 게이트웨이, 기본/커스텀 VPC, Bastion Host·NAT Gateway, 보안 그룹·Stateful·Source·Prefix List, NACL(Stateless·규칙 번호·Deny), VPC Endpoint(Interface·Gateway), EICE, VPC Peering, Transit Gateway, Direct Connect
- [S3](aws/t04-s3.md) — 객체 스토리지 개념, 버킷·객체 구성 요소, S3 비용·스토리지 클래스(Standard·IA·Glacier), S3 권한(IAM·버킷 정책·ACL), 버전 관리·객체 잠금(WORM), 수명주기(Lifecycle), 정적 웹 호스팅, 액세스 로깅·이벤트 알림(Lambda 이미지 리사이징)
- [RDS](aws/t05-rds.md) — RDB 개념, RDS 개요·EC2 연동, Multi-AZ 고가용성·Read Replica, RDS 접속·인증(Username/Password·IAM DB 인증), Amazon Aurora(분산 스토리지·Quorum·Self-Healing·Cluster Failover), Aurora Global Database(RPO/RTO), 백업·Clone·Backtrack
- [모니터링 (CloudWatch · CloudTrail · KMS)](aws/t06-monitoring.md) — CloudWatch 지표·로그·경보·Composite Alarm, CloudTrail(Trail·Management/Data/Insight Event), AWS KMS 개요

**가이드**

- [AWS 가입하기](aws/g01-signup.md) — 계정 가입 실습, 프리 티어 정책(무료/유료 플랜), 가입 절차, 가입 후 체크리스트(IAM·예산 알림·MFA·리전)
- [IAM MFA 설정](aws/g02-iam-mfa.md) — IAM 개념·기능, IAM 사용자 추가하기(세부 정보·권한·검토·암호 확인), 가상 MFA 디바이스 할당(QR·Google OTP), MFA 로그인 강제 IAM 정책(JSON), 사용자 그룹 연결, MFA 적용 IAM 사용자 생성
- [EC2 설정](aws/g03-ec2-setup.md) — 리전, 보안 그룹 생성, IP·Port 개념, 인스턴스 시작(AMI·유형·키 페어·스토리지), 인스턴스 세부 내용, 인스턴스 종료
- [EC2 접속하기](aws/g04-ec2-connect.md) — AWS 콘솔(EC2 Instance Connect) 접속, 터미널 SSH 접속, 키 페어 권한(chmod) 오류 해결
- [탄력적 IP](aws/g05-elastic-ip.md) — Elastic IP 개념, 퍼블릭 IP 고정 문제, 탄력적 IP 할당·연결 실습
- [ALB · Auto Scaling](aws/g06-load-balancer-autoscaling.md) — VPC 3-tier 보안 그룹(sg-alb·sg-web·sg-rds) 구성, ALB 생성·대상 그룹·헬스 체크, Auto Scaling(시작 템플릿·대상 추적 정책) 그룹 생성
- [EC2와 S3 연동](aws/g07-ec2-s3.md) — S3 버킷 생성, S3 접근 권한 IAM 역할 생성·EC2 연결, AWS CLI(cp·sync)로 파일 주고받기, User Data로 S3 콘텐츠 자동 배포
- [3-Tier 워드프레스 클러스터](aws/g08-wordpress-3tier.md) — IAM 역할·VPC·RDS·EFS·S3 준비, User Data 자동 배포, AMI·ASG·ALB 구성, ALB 환경 정적 리소스 CORS 트러블슈팅, 보안 그룹 강화, 리소스 삭제
- [CloudWatch 모니터링 실습](aws/g09-cloudwatch-monitoring.md) — EC2 웹 서버 로그 → CloudWatch Agent 수집 → Metric Filter로 404 지표화 → Alarm → SNS 연동 실습, CloudTrail로 Management/Data Event 추적 실습

### 🐍 Python (17)

- [Python 설치](python/00-install.md) — 온라인 컴파일러, Anaconda 설치(Windows), VS Code + Python 확장 설치, REPL/스크립트 실행으로 설치 확인
- [파이썬의 기본](python/01-basics.md) — 순차 실행, print() 함수(sep·end 옵션), 주석(#, 여러 줄 문자열)
- [변수와 자료형](python/02-variables.md) — 변수 선언/재할당, 값 복사 vs 참조, int/float/str/bool/NoneType과 type(), 산술 연산자·우선순위, 형변환, 복합 대입 연산자, 변수명 규칙
- [함수](python/03-functions.md) — def 문법, 매개변수·타입 힌트, 기본값 매개변수, 반환값, 지역/전역 변수 스코프, 변수 가림(shadowing), 중첩 함수, 상호 재귀
- [리스트와 딕셔너리](python/04-lists-dictionaries.md) — 리스트 인덱싱·2차원 리스트·슬라이싱·sum/min/max/len, append/extend/del/remove/pop, 딕셔너리 keys()/values()
- [조건문과 반복문](python/05-conditions-loops.md) — if/elif/else, 비교·불리언 연산자, while/for(range·enumerate·zip), 중첩 반복문과 break, 홀수 필터링·약수·최대공약수 실전 예제
- [클래스](python/06-classes.md) — 절차 지향 vs 객체 지향, __init__ 생성자, self, 상속, super(), 메서드 오버라이딩, 학생 출석·성적 관리 실전 예제
- [모듈과 라이브러리](python/07-modules-libraries.md) — import/from import, 패키지·점 표기법, 와일드카드 임포트, math·os·shutil, pip으로 matplotlib 설치
- [에러 처리](python/08-error-handling.md) — 에러 메시지·스택 트레이스 읽는 법, try/except/else/finally, VS Code 디버거, 흔한 실수, 특정 버전 pip 설치
- [제어 흐름 심화](python/09-control-flow-advanced.md) — match 문(구조적 패턴 매칭), *args/**kwargs, 인자 목록 언패킹, 위치 전용·키워드 전용 매개변수, lambda, 함수 애너테이션, PEP 8 네이밍
- [자료구조 심화](python/10-data-structures-advanced.md) — 스택/큐(collections.deque), 리스트·중첩·딕셔너리 컴프리헨션, 튜플 언패킹, 집합(합·교·차집합), sorted()/reversed(), 시퀀스 사전식 비교
- [입력과 출력](python/11-io-files.md) — f-string 포맷 스펙(정렬·소수점·진법), str.format(), open()/with 파일 입출력, read/readline/readlines/write, json 모듈
- [예외 처리 심화](python/12-exceptions-advanced.md) — raise 직접 발생, raise ... from ...(예외 연쇄), 사용자 정의 예외와 예외 계층, finally·with(컨텍스트 매니저), except (튜플)/except*(예외 그룹), add_note()
- [클래스 심화](python/13-classes-advanced.md) — LEGB 스코프 규칙, 클래스 변수 vs 인스턴스 변수, 비공개 변수(_·__와 name mangling), 이터레이터(__iter__/__next__), 제너레이터(yield), 제너레이터 표현식
- [표준 라이브러리 살펴보기](python/14-stdlib-tour.md) — sys.argv/sys.exit(), glob 와일드카드, re 정규표현식(match/search/findall/sub), datetime/timedelta, random, time 실행 시간 측정, collections(Counter/defaultdict)
- [가상 환경 심화와 부동소수점](python/15-venv-precision.md) — venv/pip와 requirements.txt 협업 워크플로, 부동소수점 표현 오차와 round()의 한계, decimal.Decimal, REPL 팁(탭 완성·히스토리·_), PY-00~PY-15 학습 로드맵 정리
- [표준 라이브러리 심화와 테스트](python/16-stdlib-advanced.md) — logging 모듈(레벨·getLogger), threading 기초(Thread·join·GIL·Lock), urllib(urlopen·URLError/HTTPError), unittest·doctest로 테스트 작성

왼쪽 목차(SUMMARY)를 따라 순서대로 읽거나, 필요한 주제로 바로 이동하여 참고할 수 있다.

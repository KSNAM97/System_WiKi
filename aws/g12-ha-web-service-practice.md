# AWS 고가용성 웹 서비스 구축 실습 (VPC + S3 + EC2 + ALB + Auto Scaling + Route 53)

## 목차
1. [개요](#1-개요)
2. [실습 1: VPC 구성](#2-실습-1-vpc-구성)
3. [실습 2: Internet Gateway 구성](#3-실습-2-internet-gateway-구성)
4. [실습 3: NAT Gateway 구성](#4-실습-3-nat-gateway-구성)
5. [실습 4: S3 버킷 구성](#5-실습-4-s3-버킷-구성)
6. [실습 5: IAM Role 구성](#6-실습-5-iam-role-구성)
7. [실습 6: 보안 그룹 구성](#7-실습-6-보안-그룹-구성)
8. [실습 7: Launch Template 생성](#8-실습-7-launch-template-생성)
9. [실습 8: Target Group 생성](#9-실습-8-target-group-생성)
10. [실습 9: Application Load Balancer 생성](#10-실습-9-application-load-balancer-생성)
11. [실습 10: Auto Scaling Group 생성](#11-실습-10-auto-scaling-group-생성)
12. [실습 11: 서비스 동작 확인](#12-실습-11-서비스-동작-확인)
13. [실습 12: HTTP Access Log 확인](#13-실습-12-http-access-log-확인)
14. [실습 13: Route 53 도메인 연결](#14-실습-13-route-53-도메인-연결)
15. [실습 14: 전체 트래픽 흐름 정리](#15-실습-14-전체-트래픽-흐름-정리)
16. [실습 15: 장애 테스트](#16-실습-15-장애-테스트)

---

## 1. 개요

이 실습은 VPC, S3, EC2, ALB, Auto Scaling, Route 53을 하나의 아키텍처로 통합해 고가용성(HA) 웹 서비스를 직접 구축해보는 종합 실습이다. 사용자가 도메인으로 접속하면 Route 53이 요청을 ALB로 보내고, ALB는 여러 가용 영역(AZ)에 분산된 EC2 인스턴스로 트래픽을 나누어 전달하며, EC2는 S3에 저장된 정적 웹 리소스를 읽어와 서비스한다. EC2 인스턴스 수는 Auto Scaling Group이 자동으로 관리한다.

이 실습에서 사용하는 개념은 [Amazon VPC](t03-vpc.md) · [Amazon S3](t04-s3.md) · [Amazon Route 53](t09-route53.md) 문서를 참고한다.

**전체 아키텍처**

```
사용자(브라우저)
   │ HTTPS 요청
   ▼
Amazon Route 53 (도메인 이름 → ALB Alias)
   │
   ▼
Application Load Balancer (Public Subnet, 2개 AZ)
   │
   ▼
Target Group → Auto Scaling Group (Private Subnet, 2개 AZ)
   │
   ▼
EC2 인스턴스 (Apache) ── S3 API ──▶ Amazon S3 (index.html)
```

VPC(`10.0.0.0/16`) 안에 `ap-northeast-2a`, `ap-northeast-2c` 두 개의 가용 영역을 두고, 각 AZ마다 Public Subnet(ALB, NAT Gateway 배치)과 Private Subnet(EC2 배치)을 구성한다. Public Subnet은 Internet Gateway를 통해 인터넷과 직접 통신하고, Private Subnet은 NAT Gateway를 거쳐야만 외부로 나갈 수 있다.

이 구조의 특징은 다음과 같다.

- 멀티 AZ 기반의 고가용성 구조
- Auto Scaling Group을 통한 유연한 확장/축소
- ALB를 통한 트래픽 분산
- S3에 정적 웹 리소스를 저장해 EC2와 분리 관리
- 보안 그룹을 통한 안전한 접근 제어

기업 웹 서비스 인프라, 온라인 쇼핑몰, 정적 파일과 동적 처리가 함께 필요한 콘텐츠 서비스 등에서 흔히 쓰이는 패턴이며, 클라우드 아키텍처를 처음 익힐 때도 좋은 학습 예제가 된다.

## 2. 실습 1: VPC 구성

콘솔 왼쪽 메뉴에서 서비스 검색창에 `VPC`를 입력해 **VPC** 콘솔로 이동한다. 왼쪽의 [Your VPCs]에서 [Create VPC]를 클릭한다.

- **Name tag**: `my-web-vpc`
- **IPv4 CIDR block**: `10.0.0.0/16`

VPC를 생성한 뒤에는 왼쪽의 [Subnets] 메뉴에서 [Create subnet]을 눌러 아래 4개의 서브넷을 각각 만든다. VPC ID는 방금 생성한 `my-web-vpc`를 선택한다.

| 서브넷 이름 | CIDR | 가용 영역 | 용도 |
|---|---|---|---|
| `public-subnet-a` | `10.0.1.0/24` | `ap-northeast-2a` | ALB 배치 |
| `public-subnet-b` | `10.0.2.0/24` | `ap-northeast-2c` | ALB 배치 |
| `private-subnet-a` | `10.0.3.0/24` | `ap-northeast-2a` | Auto Scaling EC2 배치 |
| `private-subnet-b` | `10.0.4.0/24` | `ap-northeast-2c` | Auto Scaling EC2 배치 |

Public Subnet 2개는 서로 다른 가용 영역에, Private Subnet 2개도 서로 다른 가용 영역에 배치해야 한다. ALB는 Public Subnet에, Auto Scaling으로 생성되는 EC2는 Private Subnet에 위치한다 — 이렇게 나누면 EC2가 인터넷에 직접 노출되지 않으면서도 ALB를 통해서만 트래픽을 받을 수 있다.

콘솔 절차는 [Amazon VPC 시작하기(AWS 공식 문서)](https://docs.aws.amazon.com/ko_kr/vpc/latest/userguide/vpc-getting-started.html)를 참고한다.

## 3. 실습 2: Internet Gateway 구성

**VPC** 콘솔 왼쪽의 [Internet Gateways]로 이동해 [Create internet gateway]를 클릭한다. 이름은 `my-web-igw`처럼 지정하고 생성한다. 생성된 Internet Gateway를 선택한 뒤 [Actions] → [Attach to VPC]를 눌러 `my-web-vpc`에 연결한다.

Public Subnet에서 인터넷 통신이 가능하도록 라우트 테이블을 구성한다. 왼쪽의 [Route Tables]에서 [Create route table]을 클릭한다.

- **Name**: `public-route-table`
- **VPC**: `my-web-vpc`

생성한 라우트 테이블을 선택하고 [Routes] 탭 → [Edit routes]에서 경로를 추가한다.

- **Destination**: `0.0.0.0/0`
- **Target**: 앞서 만든 Internet Gateway(`my-web-igw`)

마지막으로 [Subnet associations] 탭 → [Edit subnet associations]에서 `public-subnet-a`, `public-subnet-b` 두 서브넷을 이 라우트 테이블에 연결한다.

## 4. 실습 3: NAT Gateway 구성

Private Subnet의 EC2가 소프트웨어 설치나 S3 접근을 위해 인터넷으로 나갈 수 있도록 NAT Gateway를 구성한다. **VPC** 콘솔 왼쪽의 [NAT Gateways]에서 [Create NAT gateway]를 클릭한다.

- **Name**: `my-web-nat`
- **Subnet**: `public-subnet-a` (NAT Gateway는 Public Subnet에 위치해야 한다)
- **Connectivity type**: `Public`
- **Elastic IP allocation ID**: [Allocate Elastic IP]를 클릭해 새로 할당

NAT Gateway 생성이 끝나면, Private Subnet 전용 라우트 테이블을 새로 만든다. [Route Tables] → [Create route table]에서 이름을 `private-route-table`로, VPC는 `my-web-vpc`로 지정한다.

- **Destination**: `0.0.0.0/0`
- **Target**: 방금 생성한 NAT Gateway(`my-web-nat`)

[Subnet associations]에서 `private-subnet-a`, `private-subnet-b`를 이 라우트 테이블에 연결한다.

## 5. 실습 4: S3 버킷 구성

**S3** 콘솔로 이동해 [Create bucket]을 클릭한다.

- **Bucket name**: 전역에서 고유한 이름(예: `my-web-service-assets-<임의문자열>`)
- **Block Public Access settings for this bucket**: 기본값인 **모두 차단(활성화)** 상태를 그대로 유지한다. 이 버킷은 외부에 공개하지 않고, EC2가 IAM Role을 통해서만 읽도록 구성할 것이다.

버킷 생성 후 [Upload]를 눌러 미리 준비한 `index.html` 파일을 업로드한다.

```html
<!DOCTYPE html>
<html lang="ko">
<head>
  <meta charset="UTF-8">
  <title>My Web Service</title>
</head>
<body>
  <h1>My Web Service</h1>
  <hr>
</body>
</html>
```

이 `index.html`은 실습 7의 User Data 스크립트가 EC2 부팅 시 자동으로 다운로드해서 사용한다.

## 6. 실습 5: IAM Role 구성

EC2가 S3의 `index.html`을 읽어올 수 있도록 IAM Role을 생성한다. **IAM** 콘솔 왼쪽의 [Roles]에서 [Create role]을 클릭한다.

- **Trusted entity type**: `AWS service`
- **Use case**: `EC2`
- [Next]를 눌러 권한 정책에서 `AmazonS3ReadOnlyAccess`를 검색해 체크
- **Role name**: `EC2-S3-Role`

생성을 완료하면 이 Role은 실습 7의 Launch Template에서 EC2 인스턴스에 연결한다. 콘솔 절차는 [Amazon EC2용 IAM 역할 생성(AWS 공식 문서)](https://docs.aws.amazon.com/ko_kr/IAM/latest/UserGuide/id_roles_create_for-service.html)를 참고한다.

## 7. 실습 6: 보안 그룹 구성

**EC2** 콘솔 왼쪽의 [Security Groups]에서 [Create security group]을 클릭해 아래 두 개를 각각 생성한다.

**ALB-SG** (ALB용 보안 그룹)

- **VPC**: `my-web-vpc`
- **Inbound rules**: `HTTP`, 포트 `80`, Source `0.0.0.0/0` (모든 사용자의 웹 접속 허용)

**EC2-SG** (EC2용 보안 그룹)

- **VPC**: `my-web-vpc`
- **Inbound rules**: `HTTP`, 포트 `80`, Source에 `ALB-SG`를 지정 (ALB에서 오는 트래픽만 허용, 외부에서 EC2로 직접 접근 불가)

이렇게 구성하면 EC2는 ALB를 거친 트래픽만 받을 수 있고, ALB는 누구에게나 열려 있는 구조가 된다.

## 8. 실습 7: Launch Template 생성

**EC2** 콘솔 왼쪽의 [Launch Templates]에서 [Create launch template]을 클릭한다.

- **Launch template name**: `my-web-launch-template`
- **AMI**: `Amazon Linux 2023`
- **Instance type**: `t3.micro`
- **Key pair**: 필요 시 기존 키 페어 선택
- **Security groups**: 실습 6에서 만든 `EC2-SG`
- [Advanced details] → **IAM instance profile**: 실습 5에서 만든 `EC2-S3-Role`

**Advanced details** 맨 아래 **User data** 칸에 아래 스크립트를 입력한다. `<S3_BUCKET_NAME>`은 실습 4에서 만든 버킷 이름으로 바꿔야 한다.

```bash
#!/bin/bash
# 1. httpd(Apache) 설치
yum install -y httpd

# 2. httpd 서비스 시작
systemctl start httpd

# 3. httpd 부팅 시 자동 시작 설정
systemctl enable httpd

# 4. S3에서 index.html 다운로드
aws s3 cp s3://<S3_BUCKET_NAME>/index.html /var/www/html/index.html

# 5. 자신의 EC2 Instance ID 조회 (IMDSv2 토큰 발급 후 조회)
TOKEN=$(curl -s -X PUT "http://169.254.169.254/latest/api/token" \
  -H "X-aws-ec2-metadata-token-ttl-seconds: 21600")
INSTANCE_ID=$(curl -s -H "X-aws-ec2-metadata-token: $TOKEN" \
  http://169.254.169.254/latest/meta-data/instance-id)

# 6. index.html 하단에 Instance ID 추가
echo "<h3>Instance ID: ${INSTANCE_ID}</h3>" >> /var/www/html/index.html
```

이 스크립트가 정상적으로 동작하면, EC2가 처음 부팅될 때 자동으로 httpd를 설치·실행하고 S3에서 받아온 `index.html`에 자신의 Instance ID를 덧붙인다. 웹 브라우저로 접속하면 아래와 같은 형태로 출력된다.

```
My Web Service
--------------
Instance ID: i-xxxxxxxxxxxxxxxxx
```

- IMDSv2를 사용하는 이유: IMDSv1은 토큰 없이 메타데이터 엔드포인트에 접근할 수 있어 SSRF 공격에 취약하다는 지적이 있었다. 토큰을 먼저 발급받아 헤더에 실어 요청하는 IMDSv2 방식이 더 안전하다.
- `aws s3 cp`가 동작하려면 EC2에 `EC2-S3-Role`이 연결되어 있어야 하며, Amazon Linux 2023 AMI에는 AWS CLI가 기본 설치되어 있어 별도 설치 과정이 필요 없다.

## 9. 실습 8: Target Group 생성

**EC2** 콘솔 왼쪽의 [Target Groups]에서 [Create target group]을 클릭한다.

- **Target type**: `Instances`
- **Target group name**: `my-web-tg`
- **Protocol / Port**: `HTTP` / `80`
- **IP address type**: `IPv4`
- **VPC**: `my-web-vpc`
- **Health checks** → **Health check path**: `/index.html`

이 단계에서는 아직 개별 인스턴스를 등록하지 않고 [Next] → [Create target group]으로 빈 Target Group만 만들어 둔다. 인스턴스 등록은 실습 10의 Auto Scaling Group이 자동으로 처리한다.

## 10. 실습 9: Application Load Balancer 생성

**EC2** 콘솔 왼쪽의 [Load Balancers]에서 [Create load balancer] → **Application Load Balancer**의 [Create]를 클릭한다.

- **Load balancer name**: `my-web-alb`
- **Scheme**: `Internet-facing`
- **VPC**: `my-web-vpc`
- **Mappings**: `public-subnet-a`, `public-subnet-b` 두 서브넷 모두 체크
- **Security groups**: 실습 6의 `ALB-SG`
- **Listeners**: `HTTP` `80`, **Default action**: 실습 8에서 만든 `my-web-tg`로 전달

[Create load balancer]를 눌러 생성한다. 생성이 끝나면 ALB 상세 페이지에서 **DNS name**을 확인할 수 있다 — 이 값은 실습 13에서 Route 53 레코드의 Alias 대상으로 사용한다.

## 11. 실습 10: Auto Scaling Group 생성

**EC2** 콘솔 왼쪽의 [Auto Scaling Groups]에서 [Create Auto Scaling group]을 클릭한다.

- **Auto Scaling group name**: `my-web-asg`
- **Launch template**: 실습 7의 `my-web-launch-template`
- **VPC**: `my-web-vpc`
- **Subnets**: `private-subnet-a`, `private-subnet-b`
- [Attach to an existing load balancer] → 실습 8의 Target Group `my-web-tg` 선택
- **Health check type**: `ELB` (ALB의 Health Check 결과를 함께 반영)

**Group size** 설정은 다음과 같이 지정한다.

- **Desired capacity**: `2`
- **Minimum capacity**: `2`
- **Maximum capacity**: `5`

> **주의**: Auto Scaling Group은 Desired Capacity가 항상 Minimum과 Maximum 사이(Min ≤ Desired ≤ Max)에 있어야 정상 동작한다. Minimum을 Desired보다 크게 설정하면(예: Desired 2, Minimum 3) 그룹이 즉시 스케일 아웃을 시도하거나 설정 자체가 거부될 수 있으므로, Minimum Capacity는 반드시 Desired Capacity 이하로 맞춰야 한다. 이번 실습에서는 평소 2대를 유지하다가 트래픽이 늘어나면 최대 5대까지 늘어나는 구조를 의도했으므로 Minimum과 Desired를 동일하게 2로 설정한다.

## 12. 실습 11: 서비스 동작 확인

Auto Scaling Group이 EC2 인스턴스를 생성하고 Target Group에 등록해 Health Check를 통과할 때까지 몇 분 정도 기다린다. **EC2** 콘솔의 [Load Balancers]에서 `my-web-alb`를 선택해 **DNS name**을 복사한 뒤, 웹 브라우저 주소창에 `http://<ALB DNS name>` 형태로 접속한다.

페이지에 `My Web Service`와 함께 `Instance ID: i-xxxxxxxxxxxxxxxxx`가 출력되는지 확인한다. 브라우저를 여러 번 새로고침하면서 Instance ID 값이 바뀌는지 관찰한다 — ALB가 기본적으로 라운드 로빈 방식으로 Target Group 안의 여러 EC2에 요청을 고르게 분산하기 때문에, 새로고침할 때마다 다른 인스턴스가 응답하는 것을 확인할 수 있다.

## 13. 실습 12: HTTP Access Log 확인

Auto Scaling Group이 생성한 각 EC2 인스턴스에 SSH 또는 EC2 Instance Connect로 접속한 뒤, Apache의 Access Log를 실시간으로 확인한다.

```bash
tail -f /var/log/httpd/access_log
```

이 상태에서 ALB의 DNS 주소로 웹 브라우저를 여러 번 새로고침하면서 접속하면, 요청이 어느 EC2로 분산되는지 각 인스턴스의 로그에서 직접 확인할 수 있다. 특정 인스턴스의 로그에만 요청이 쌓이거나, 두 인스턴스에 번갈아 쌓이는 패턴을 관찰하면 ALB의 트래픽 분산 동작을 눈으로 검증할 수 있다.

## 14. 실습 13: Route 53 도메인 연결

**Route 53** 콘솔로 이동해 왼쪽의 [Hosted zones]에서 사용할 도메인의 Hosted Zone을 선택한다 (Hosted Zone이 없다면 [Amazon Route 53](t09-route53.md) 문서의 "Route 53 사용 과정 3단계"를 참고해 먼저 생성한다). [Create record]를 클릭한다.

- **Record name**: `web` (예: `web.example.com`으로 접속하도록 구성)
- **Record type**: `A`
- **Alias**: 토글을 `On`으로 활성화
- **Route traffic to**: `Alias to Application Load Balancer and Classic Load Balancer`
- **Region**: ALB가 위치한 리전 (`ap-northeast-2`)
- **Load balancer**: 실습 9에서 생성한 `my-web-alb` 선택

[Create records]로 저장한 뒤, 웹 브라우저에서 `http://web.example.com`(실제로 사용한 도메인)으로 접속해 실습 11에서 확인한 것과 동일하게 웹 페이지와 Instance ID가 정상적으로 출력되는지 확인한다.

## 15. 실습 14: 전체 트래픽 흐름 정리

이번 실습으로 구성한 아키텍처에서 사용자가 `http://web.example.com`에 접속했을 때의 요청 처리 흐름은 다음과 같다.

```
사용자 요청 (http://web.example.com)
   │
   ▼
Route 53          ─ Alias A 레코드가 도메인을 ALB로 매핑
   │
   ▼
ALB               ─ Listener(80)가 요청을 받아 Target Group으로 전달
   │
   ▼
Target Group      ─ Health Check를 통과한 EC2 중 하나를 선택
   │
   ▼
Auto Scaling Group ─ 선택된 EC2가 실제로 요청을 처리
   │
   ▼
EC2 (Apache)       ─ /var/www/html/index.html 응답
   │
   ▼
사용자에게 웹 페이지 반환
```

Route 53은 도메인을 ALB로 연결하는 진입점 역할만 하고, 실제 트래픽 분산과 인스턴스 상태 관리는 ALB와 Target Group, Auto Scaling Group이 담당한다는 점이 이 아키텍처의 핵심이다.

## 16. 실습 15: 장애 테스트

Auto Scaling Group의 자동 복구 동작을 직접 확인하기 위해 장애 상황을 인위적으로 만들어본다. **EC2** 콘솔의 [Instances]에서 `my-web-asg`가 실행 중인 인스턴스 중 하나를 선택해 [Instance state] → [Terminate instance]로 강제 종료한다.

종료 직후부터 다음 항목들을 순서대로 확인한다.

1. **Target Group 상태 변화**: [Target Groups] → `my-web-tg`의 [Targets] 탭에서 종료된 인스턴스가 `unhealthy` 또는 목록에서 제거되는 상태로 바뀌는지 확인한다.
2. **새 EC2 자동 생성 여부**: [Auto Scaling Groups] → `my-web-asg`의 [Activity] 탭에서 새 인스턴스가 시작되는 활동 로그가 기록되는지 확인한다.
3. **Desired Capacity 유지**: 인스턴스가 1대 종료되어도 Auto Scaling Group이 곧바로 새 인스턴스를 띄워 Desired Capacity `2`를 다시 맞추는지 확인한다.
4. **새 EC2의 Target Group 자동 등록**: 새로 생성된 인스턴스가 별도 조작 없이 `my-web-tg`에 자동으로 등록되는지 확인한다.
5. **Health Check 통과 후 트래픽 전달**: 새 인스턴스가 Health Check를 통과하면 ALB가 이 인스턴스로도 트래픽을 전달하기 시작하는지, 실습 12와 같은 방식으로 Access Log를 확인해 검증한다.

이 과정을 통해 Auto Scaling Group이 장애가 발생한 인스턴스를 감지해 자동으로 대체하고, ALB와 Target Group이 새 인스턴스를 자연스럽게 서비스에 편입시키는 것이 고가용성 아키텍처의 핵심 동작이라는 점을 확인할 수 있다.

# Route 53 Failover 라우팅 실습 (Health Check + Failover)

## 1. 실습 개요

이 실습은 Primary EC2와 Secondary EC2를 각각 준비하고, Route 53의 Health Check와 Failover 라우팅 정책을 이용해 Primary 장애 시 Secondary로 트래픽이 자동 전환되는 것을 직접 확인하는 실습이다. Health Check와 8가지 라우팅 정책의 개념은 [AWS-T10. Route 53 Health Check와 라우팅 정책](t10-route53-healthcheck-routingpolicy.md) 문서를 먼저 참고하면 이해가 쉽다.

**실습 흐름**

```
사용자
  │ 도메인 접속
  ▼
Route 53 (Failover 레코드 + Health Check)
  │
  ├─ Health Check: Healthy → Primary EC2로 라우팅
  └─ Health Check: Unhealthy → Secondary EC2로 자동 전환
```

이번 실습에서는 EC2 2대(Primary, Secondary)를 준비하고, Primary에 장애를 발생시켜 Health Check가 이를 감지한 뒤 Secondary로 트래픽이 넘어가는 전체 과정을 확인한다.

**실습 시 참고**: 아래 절차에 등장하는 IP(`<EC2_A_PUBLIC_IP>`, `<EC2_B_PUBLIC_IP>`)와 도메인(`my-web-service.com`)은 예시 값이다. 실제로는 자신이 준비한 EC2의 퍼블릭 IP와 자신의 도메인으로 치환해서 진행한다.

## 2. 실습 1: Primary/Secondary EC2 준비

Primary EC2와 Secondary EC2를 준비하는 방법은 두 가지가 있다. 실무에서는 목적에 따라 두 방식을 섞어서 쓰기도 하므로, 자동화 방식과 수동 배포 방식의 차이를 이해하고 진행한다.

### 방법 A: Launch Template의 User Data로 자동 구성

**EC2** 콘솔 왼쪽의 [Launch Templates]에서 [Create launch template]을 클릭해 Primary용 템플릿을 만든다.

- **Launch template name**: `failover-primary-template`
- **AMI**: Amazon Linux 계열 (`dnf` 패키지 매니저를 사용하는 최신 Amazon Linux)
- **Instance type**: `t3.micro`
- **Security groups**: HTTP(80) 인바운드를 허용하는 보안 그룹

[Advanced details] 맨 아래 **User data** 칸에 아래 스크립트를 입력한다. EC2가 부팅될 때 자동으로 httpd를 설치하고, 자신의 Instance ID를 페이지에 표시하도록 구성한다.

```bash
#!/bin/bash
sudo -s
dnf install httpd -y
service httpd start
chkconfig httpd on
TOKEN=$(curl -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 21600")
INSTANCE_ID=$(curl -s -H "X-aws-ec2-metadata-token: $TOKEN" http://169.254.169.254/latest/meta-data/instance-id)
echo "<h1>$INSTANCE_ID</h1>" > /var/www/html/index.html
echo "<h1>hello My Web Service</h1>" >> /var/www/html/index.html
```

이 템플릿으로 EC2를 하나 실행하면 Primary EC2가 되고, Security Group과 AMI를 동일하게 맞춘 또 다른 Launch Template(또는 같은 템플릿으로 인스턴스만 추가 실행)으로 Secondary EC2를 준비한다.

### 방법 B: 로컬에서 index.html을 scp로 직접 배포

Launch Template의 User Data 대신, 이미 실행 중인 EC2에 로컬에서 준비한 `index.html`을 직접 업로드하는 방식이다. 리눅스/맥 환경에서는 `scp` 명령을 사용한다.

```bash
scp -i My-EC2-KeyPair.pem ./index.html ec2-user@<EC2_A_PUBLIC_IP>:~/index.html
scp -i My-EC2-KeyPair.pem ./index.html ec2-user@<EC2_B_PUBLIC_IP>:~/index.html
```

여기서 `<EC2_A_PUBLIC_IP>`는 Primary EC2의 퍼블릭 IP, `<EC2_B_PUBLIC_IP>`는 Secondary EC2의 퍼블릭 IP다. 파일을 전송한 뒤에는 각 인스턴스에 SSH로 접속해서 `/var/www/html/index.html` 위치로 파일을 옮기고 httpd를 재시작하면 된다.

### 두 방식의 차이

| 구분 | 방법 A (User Data 자동화) | 방법 B (scp 수동 배포) |
|---|---|---|
| 실행 시점 | EC2 최초 부팅 시 자동 실행 | 관리자가 필요할 때 수동 실행 |
| 반복 가능성 | Auto Scaling 등으로 동일한 인스턴스를 여러 대 찍어낼 때 유리 | 인스턴스 1~2대를 직접 다루는 소규모 환경에 적합 |
| 변경 반영 | 새 인스턴스를 다시 띄워야 반영됨(재부팅만으로는 User Data 재실행 안 됨) | 파일만 다시 전송하면 즉시 반영 |
| 실무 활용 | 표준화된 배포 파이프라인, Auto Scaling 연동 환경 | 테스트, 긴급 수정, 소규모 실습 환경 |

이번 실습에서는 Primary EC2는 방법 A로, Secondary EC2는 방법 B로 구성해 보면서 두 방식을 모두 경험해 보는 것을 권장한다.

## 3. 실습 2: S3 정적 웹 호스팅 버킷 준비 (선택)

EC2 장애 시 정적 페이지로도 대응할 수 있도록, 선택적으로 S3 정적 웹 호스팅 버킷을 백업 대상으로 함께 준비할 수 있다. EC2 Primary/Secondary 구성만으로 실습을 진행해도 무방하며, 이 단계는 건너뛰어도 된다.

**S3** 콘솔에서 [Create bucket]을 클릭해 버킷을 생성하고, [Properties] 탭에서 [Static website hosting]을 활성화한다. 버킷을 퍼블릭으로 열어야 하므로 [Permissions] 탭의 [Block public access]를 해제하고, 아래와 같은 버킷 정책을 등록해 모든 사용자가 객체를 다운로드(읽기)할 수 있도록 허용한다.

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "Statement1",
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::my-web-service.com/*"
        }
    ]
}
```

`Resource`에 지정한 버킷 이름(`my-web-service.com`)은 실제로 생성한 버킷 이름으로 바꿔서 등록해야 한다. 이 버킷을 Failover의 Secondary 대상으로 활용하면, EC2 인스턴스 자체가 완전히 사라지는 상황에도 최소한의 정적 안내 페이지를 계속 서비스할 수 있다.

## 4. 실습 3: Route 53 Health Check 생성

**Route 53** 콘솔 왼쪽의 [Health checks]에서 [Create health check]를 클릭한다.

- **Name**: `primary-ec2-healthcheck`
- **What to monitor**: `Endpoint` 선택
- **Specify endpoint by**: `IP address` 또는 `Domain name` 중 하나를 선택한다.
  - IP 기반으로 진행한다면 Primary EC2의 퍼블릭 IP(`<EC2_A_PUBLIC_IP>`)를 입력한다.
  - 도메인 기반으로 진행한다면 Primary EC2를 가리키는 A 레코드가 이미 있어야 하며, 해당 도메인을 입력한다.
- **Protocol**: `HTTP`
- **Port**: `80`
- **Path**: `/` 또는 `/index.html`
- **Request interval**: `Standard(30 seconds)` 또는 `Fast(10 seconds)` 중 선택한다. Fast는 추가 요금이 발생하므로, 이번 실습에서는 `Standard(30 seconds)`로 진행한다.
- **Failure threshold**: 기본값(3)을 그대로 사용한다.

[Create health check]로 생성을 완료하면, Health Check 목록에서 상태가 처음에는 `Unknown`으로 표시되다가 몇 분 내로 `Healthy`로 바뀌는 것을 확인할 수 있다.

## 5. 실습 4: Failover 레코드 생성

**Route 53** 콘솔 왼쪽의 [Hosted zones]에서 사용할 도메인의 Hosted Zone으로 이동한다. Primary 레코드와 Secondary 레코드를 각각 따로 생성한다.

### Primary 레코드 생성

[Create record]를 클릭한다.

- **Record name**: `failover` (예: `failover.my-web-service.com`으로 접속)
- **Record type**: `A`
- **Value**: `<EC2_A_PUBLIC_IP>` (Primary EC2의 퍼블릭 IP)
- **Routing policy**: `Failover`
- **Failover record type**: `Primary`
- **Health check ID**: 실습 3에서 생성한 `primary-ec2-healthcheck`
- **Record ID**: `primary-record` (Failover 레코드는 같은 이름/타입이라도 서로 구분할 Record ID가 필요하다)

### Secondary 레코드 생성

같은 방식으로 [Create record]를 한 번 더 클릭한다.

- **Record name**: `failover` (Primary와 동일한 이름으로 지정)
- **Record type**: `A`
- **Value**: `<EC2_B_PUBLIC_IP>` (Secondary EC2의 퍼블릭 IP)
- **Routing policy**: `Failover`
- **Failover record type**: `Secondary`
- **Health check ID**: 선택 사항이다. Secondary에도 별도의 Health Check를 연결해 두면, Primary와 Secondary가 동시에 장애 상태일 때도 상황을 구분해서 확인할 수 있다.
- **Record ID**: `secondary-record`

두 레코드 모두 저장하고 나면, Route 53은 Health Check 결과에 따라 `failover.my-web-service.com`에 대한 응답을 Primary IP 또는 Secondary IP 중 하나로 자동 결정한다.

## 6. 실습 5: 장애 조치 테스트

Primary EC2에 장애를 일으켜 Health Check가 Unhealthy로 바뀌는 것과, 이후 도메인 응답이 Secondary로 전환되는 것을 확인한다.

1. Primary EC2에 SSH로 접속한 뒤, httpd 서비스를 중지한다.

```bash
sudo service httpd stop
```

2. **Route 53** 콘솔의 [Health checks]로 이동해 `primary-ec2-healthcheck`의 상태를 확인한다. 몇 차례의 연속 실패(Failure threshold에 도달) 후 상태가 `Healthy`에서 `Unhealthy`로 바뀌는 것을 확인한다.
3. 상태가 `Unhealthy`로 바뀐 뒤, 웹 브라우저에서 `http://failover.my-web-service.com`으로 재접속한다. 응답 내용이 Primary EC2가 아니라 Secondary EC2의 페이지(다른 Instance ID 또는 다른 문구)로 바뀌어 있는지 확인한다.
4. 확인이 끝나면 Primary EC2에서 httpd를 다시 시작해 원상 복구한다.

```bash
sudo service httpd start
```

5. 다시 Health Check 상태가 `Healthy`로 돌아오고, 도메인 응답도 Primary EC2로 복귀하는지 확인한다.

더 강하게 테스트하고 싶다면 httpd를 중지하는 대신 Primary EC2 인스턴스 자체를 [Instance state] → [Stop instance] 또는 [Terminate instance]로 종료해 완전한 장애 상황을 재현할 수도 있다.

## 7. 실습 6: DNS 전파 확인

Failover 전환이 실제로 전 세계 여러 위치에서 어떻게 보이는지 확인하려면 [https://www.whatsmydns.net](https://www.whatsmydns.net) 을 활용한다. 이 사이트는 전 세계 여러 위치에서 특정 도메인의 DNS 레코드가 어떻게 전파되었는지 한눈에 보여준다.

주소창에 `failover.my-web-service.com`을 입력하고 레코드 타입을 `A`로 선택해 조회하면, 지역별로 어떤 IP가 응답되고 있는지 확인할 수 있다. 장애 전환 직후에는 일부 지역에서 아직 이전 캐시(Primary IP)가 남아 있을 수 있고, 시간이 지나면서 점점 Secondary IP로 통일되는 과정을 관찰할 수 있다.

이 현상은 [AWS-T09. Route 53](t09-route53.md) 문서에서 다룬 TTL 개념과 직접 연결된다. 레코드의 TTL 값이 짧을수록 리졸버의 캐시가 더 빨리 만료되어 새로운 값(Secondary IP)이 더 빠르게 반영된다. 반대로 TTL을 길게 잡아둔 상태였다면, Health Check가 Unhealthy로 바뀌어도 일부 사용자에게는 한동안 Primary IP가 계속 캐시되어 보일 수 있다. Failover 라우팅을 쓰는 레코드는 TTL을 평소보다 짧게(예: 60초 이내) 설정해 두는 것이 실무에서 흔한 패턴이다.

## 8. 실습 7 (선택): Health Check 조합 활용

여러 리전에 걸쳐 서비스를 운영한다면, 리전별 Health Check 결과를 조합해서 더 정교하게 장애를 판단할 수 있다. [AWS-T10. Route 53 Health Check와 라우팅 정책](t10-route53-healthcheck-routingpolicy.md) 문서의 "여러 Health Check 조합" 절에서 다룬 AND/OR/Quorum 방식을 실제 콘솔의 [Create health check] → `Calculated Health Check` 옵션에서 구성할 수 있다.

CloudWatch 관점에서는 리전별 점검 결과를 최소(Minimum)/최대(Maximum)/평균(Average) 연산으로 조합해 지표를 집계할 수도 있다. 예를 들어 도쿄·싱가포르·유럽 리전은 정상이고 미국 리전만 비정상인 상황에서, 최소 연산을 쓰면 전체를 즉시 비정상으로 판단해 민감하게 대응할 수 있고, 평균 연산을 쓰면 부분 장애율(예: 0.75)을 근거로 알림 수준을 조절할 수 있다. 이때 CloudWatch가 지표를 얼마나 자주 집계할지 결정하는 **기간(Period)** 값을 1분(빠른 감지) 또는 5분(안정적 감지) 중에서 선택해 알림 민감도를 조정한다.

이 실습에서 만든 단일 Primary/Secondary 구성을 확장해서, 리전별로 Health Check를 추가하고 Calculated Health Check로 묶어보면 더 실무에 가까운 장애 조치 구성을 경험할 수 있다.

# AWS ALB · Auto Scaling

## 1. ELB (Elastic Load Balancer) 소개

ELB는 인터넷 트래픽(부하)를 여러 대의 서버(일반적으로 EC2 인스턴스)에 분산시켜 처리한다. ELB의 주요 목적은 애플리케이션의 가용성과 내구성을 높이는 것이다. ELB는 요청을 여러 서버에 분산함으로써 단일 서버에 발생할 수 있는 과부하를 방지하고, 서버 중 하나가 실패하더라도 자동으로 트래픽을 건강한 서버로 리디렉션하여 서비스 중단 시간을 최소화한다.

**AWS ELB의 세 가지 유형**

- **Application Load Balancer (ALB)**: HTTP와 HTTPS 트래픽에 최적화되어 있으며, 고급 라우팅 기능(경로 기반 라우팅, 호스트 기반 라우팅 등)을 제공한다. 애플리케이션의 특정 URL 또는 호스트 이름에 따라 트래픽을 다른 타겟 그룹으로 라우팅할 수 있다.
- **Network Load Balancer (NLB)**: TCP, UDP, TLS 트래픽에 최적화되어 있으며, 초고성능과 정적 IP 주소 할당을 필요로 하는 애플리케이션에 적합하다. 밀리초 단위의 지연 시간과 매우 높은 처리량을 제공한다.
- **Classic Load Balancer (CLB)**: 초기 ELB 서비스로, 단순한 로드 밸런싱 기능을 제공한다. 애플리케이션 계층과 네트워크 계층 모두에서 로드 밸런싱을 지원하지만, AWS는 새 애플리케이션에는 ALB나 NLB 사용을 권장한다.

이 문서에서는 **Application Load Balancer(ALB) + 대상 그룹 + Auto Scaling 그룹을 하나의 아키텍처로 구성하는 방법**을 실습 중심으로 다룬다. ALB가 사용자 트래픽을 받아 대상 그룹으로 전달하고, 대상 그룹의 실제 EC2 인스턴스는 Auto Scaling 그룹이 부하에 따라 자동으로 늘리고 줄인다. 이 구조에서 각 계층 간 접근은 보안 그룹의 Source 지정으로 통제한다.

트래픽과 구성 요소의 흐름을 정리하면 `사용자 → ALB(sg-alb) → 대상 그룹 → EC2(sg-web, Auto Scaling 그룹이 관리)`이며, Auto Scaling 그룹은 시작 템플릿을 기반으로 EC2 인스턴스를 생성해 대상 그룹에 자동으로 등록·해제한다.

## 2. VPC 보안 그룹 구성 (3-tier)

로드 밸런서와 Auto Scaling 그룹을 만들기 전에, 계층별 접근을 통제할 보안 그룹부터 구성한다. `sg-alb`(외부 → ALB) → `sg-web`(ALB → EC2) → `sg-rds`(EC2 → DB) 순으로 이어지는 3-tier 패턴을 사용한다. 보안 그룹 생성과 인바운드 규칙 추가 절차는 [Amazon EC2 보안 그룹(AWS 공식 문서)](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-security-groups.html)을 참고한다.

### 1) sg-alb — 외부 사용자 요청 수신

VPC 콘솔 또는 EC2 콘솔의 [보안 그룹] 메뉴에서 [보안 그룹 생성]을 클릭한다.

- **보안 그룹 이름**: `sg-alb`
- **설명**: 예) `ALB inbound from internet`
- **VPC**: 사용할 VPC 선택

**인바운드 규칙**

| 유형 | 포트 | Source |
|---|---|---|
| HTTP | 80 | `0.0.0.0/0` |
| HTTPS | 443 | `0.0.0.0/0` |

모든 사용자의 웹 접속을 허용하는 규칙이다. 아웃바운드 규칙은 기본값(모든 트래픽 허용)을 그대로 둔다.

### 2) sg-web — ALB로부터 요청만 처리

같은 방식으로 보안 그룹을 하나 더 생성한다.

- **보안 그룹 이름**: `sg-web`
- **설명**: 예) `EC2 inbound from ALB only`
- **VPC**: 동일 VPC 선택

**인바운드 규칙**

| 유형 | 포트 | Source |
|---|---|---|
| 사용자 지정 TCP | 3000 (또는 애플리케이션 포트) | `sg-alb` |

Source에 CIDR 대신 앞서 만든 `sg-alb`를 직접 지정한다. 이렇게 하면 `sg-alb`에 속한 리소스(ALB)로부터 오는 트래픽만 허용되며, ALB를 거치지 않은 직접 접근은 차단된다. Auto Scaling으로 EC2 인스턴스가 늘거나 줄어도, 각 인스턴스가 `sg-web`을 사용하는 한 규칙을 다시 설정할 필요가 없다.

### 3) sg-rds — 웹 서버에서만 DB 접속 허용

DB(RDS)를 함께 운영하는 경우 하나 더 생성한다.

- **보안 그룹 이름**: `sg-rds`
- **설명**: 예) `RDS inbound from EC2 only`
- **VPC**: 동일 VPC 선택

**인바운드 규칙**

| 유형 | 포트 | Source |
|---|---|---|
| MySQL/Aurora (또는 사용 DB 엔진) | 3306 | `sg-web` |

Source를 `sg-web`으로 지정하면, `sg-web`을 사용하는 EC2에서 오는 연결만 DB에 접근할 수 있다. 외부나 다른 보안 그룹의 리소스는 차단된다.

정리하면 트래픽 흐름은 `사용자 → (sg-alb) ALB → (sg-web) EC2 → (sg-rds) RDS` 순으로 흐르며, 각 구간은 이전 구간의 보안 그룹만을 Source로 허용하는 방식으로 통제된다.

## 3. 로드 밸런서 생성

콘솔 절차는 [Create an Application Load Balancer(AWS 공식 문서)](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/create-application-load-balancer.html)를 참고한다.

### 3.1 로드 밸런서 생성 시작

EC2 콘솔의 [로드 밸런서(Load Balancers)] 메뉴 → [로드 밸런서 생성] 클릭 (리전이 서울로 설정되어 있는지 확인) → **Application Load Balancer** 유형에서 [생성] 클릭.

**기본 구성(Basic configuration)**

- **로드 밸런서 이름**: 자유롭게 작성 (영숫자와 하이픈만 허용, 생성 후 변경 불가)
- **체계(Scheme)**: 인터넷을 통해 트래픽을 받을 것이므로 **Internet-facing** 선택
- **로드 밸런서 IP 주소 유형**: 기본값 **IPv4** 유지

### 3.2 네트워크 매핑

**네트워크 매핑(Network mapping)** 단계에서 사용할 VPC를 선택한다. Internet-facing 로드 밸런서는 인터넷 게이트웨이가 연결된 VPC만 선택할 수 있다. **가용 영역 및 서브넷(Availability Zones and subnets)**에서는 최소 2개 이상의 가용 영역(AZ)에 속한 서브넷을 선택한다 — 고가용성을 위해 전부 체크한다.

### 3.3 보안 그룹

**보안 그룹(Security groups)** 단계에서는 VPC의 기본 보안 그룹이 자동으로 선택되어 있다. 이를 제거하고 앞서 생성한 `sg-alb`를 선택한다.

### 3.4 리스너 및 라우팅 - 대상 그룹 생성

**리스너 및 라우팅(Listeners and routing)**은 ELB로 들어온 요청을 어떤 대상 그룹으로 전달할지 지정하는 항목이다. 기본 리스너는 `HTTP : 80`이며 그대로 사용한다. **기본 작업(Default action)**에서 대상 그룹을 선택해야 하는데, 아직 없으므로 [대상 그룹 생성(Create target group)]을 클릭한다.

**대상 그룹 세부 정보**: 고객이 로드 밸런서에 접속 시 EC2의 특정 인스턴스에 전달해야 하므로 대상 유형은 **인스턴스**로 지정한다. 대상 그룹 이름은 자유롭게 작성하고, 프로토콜:포트는 `HTTP : 80`으로 둔다.

나머지 IP 주소 유형·프로토콜 버전 등은 기본값(IPv4, HTTP1)으로 둔다. 이 항목들은 ELB가 사용자로부터 트래픽을 받아 대상 그룹에게 어떤 방식으로 전달할지 설정하는 부분이다.

### 3.5 상태 검사(Health Check)와 헬스 체크 API

ELB의 부가 기능으로 **상태 검사(Health Check)**가 있다. 특정 인스턴스의 서버가 예상치 못한 오류가 발생했을 때, ELB 입장에서는 해당 서버에 요청(트래픽)을 보내는 것이 비효율적이다. 이러한 상황을 방지하기 위해 ELB는 주기적으로 대상 그룹에 속해 있는 인스턴스에게 요청을 보낸다. 그 요청 상태가 200으로 전달되면 서버에 문제가 없다고 판단하며, 응답이 오지 않는다면 ELB는 해당 인스턴스에 요청을 보내지 않는다.

인스턴스에 상태 검사용 `/health` API가 필요하다. [EC2 배포](t02-ec2-deployment.md)의 [23. 실습: 웹 서버 배포 · 메타데이터 조회 · IAM 자격증명](t02-ec2-deployment.md#23-실습-웹-서버-배포-메타데이터-조회-iam-자격증명)에서 사용한 **`demo-aws-credential`** Node.js 프로젝트를 그대로 확장해서 순수 Node.js(`http` 모듈)로 헬스 체크 API를 추가한다.

```js
// demo-aws-credential/src/health.js
const http = require("http");

const server = http.createServer((req, res) => {
    if (req.url === "/health") {
        res.writeHead(200, { "Content-Type": "application/json" });
        res.end(JSON.stringify({ status: "ok" }));
        return;
    }
    res.writeHead(404);
    res.end();
});

server.listen(3000, () => console.log("Health check server running on port 3000"));
```

EC2 인스턴스에 코드를 배포한 후, 빌드 없이 바로 pm2로 데몬 실행/재시작한다.

```bash
pm2 start src/health.js --name demo-aws-credential
# 코드 수정 후 재시작
pm2 reload demo-aws-credential
```

브라우저 또는 `curl http://<EC2_PUBLIC_IP>:3000/health`로 호출하면 `{"status":"ok"}` 응답이 오는 것을 확인할 수 있다. 대상 그룹의 상태 검사 경로를 `/health`, 포트를 `3000`으로 지정하면 이 엔드포인트를 통해 ELB가 인스턴스 상태를 주기적으로 확인한다.

### 3.6 대상 등록과 생성 완료

이 시점에는 아직 Auto Scaling 그룹이 없으므로, 대상 등록 단계는 건너뛰고 [대상 그룹 생성]으로 완료한다. 리스너 및 라우팅 화면으로 돌아와 새로 고침한 다음 방금 만든 대상 그룹을 기본 작업으로 선택한다. 이어지는 **요약(Summary)** 단계에서 설정을 확인하고 [로드 밸런서 생성]을 클릭한다.

로드 밸런서 상태가 "프로비저닝 중"에서 "활성"으로 바뀌면 상세 페이지에서 DNS 이름을 확인할 수 있다. 대상 그룹에 실제 EC2 인스턴스가 등록되는 것은 다음 단계의 Auto Scaling 그룹 생성 과정에서 자동으로 이루어진다.

## 4. Auto Scaling 그룹 생성

Auto Scaling 그룹을 만들면 대상 그룹에 연결된 EC2 인스턴스를 지정한 정책에 따라 자동으로 늘리고 줄일 수 있다. 콘솔 절차는 [Creating an Auto Scaling group using a launch template(AWS 공식 문서)](https://docs.aws.amazon.com/autoscaling/ec2/userguide/create-asg.html)를 참고한다.

전체 흐름은 다음과 같다. ① 시작 템플릿을 기반으로 ASG가 EC2 인스턴스를 생성하고, ② 각 인스턴스가 대상 그룹의 헬스 체크(`GET /health`)를 통과하면 자동으로 대상 그룹에 등록되어 트래픽을 받기 시작한다. ③ CloudWatch가 인스턴스들의 평균 CPU 사용률을 지속적으로 감시하고, ④ 이 값이 스케일링 정책의 목표치를 초과하거나 미달하면 ASG가 인스턴스 수를 자동으로 늘리거나(Scale Out) 줄인다(Scale In). 아직 헬스 체크를 통과하지 못했거나 Scale In 대상으로 지정된 인스턴스는 대상 그룹에서 제외되어 트래픽을 받지 않는다.

### 4.1 시작 템플릿(Launch Template) 생성

EC2 콘솔의 [시작 템플릿] 메뉴에서 [시작 템플릿 생성]을 클릭한다.

- **AMI**: 배포할 애플리케이션과 동일한 AMI(또는 Amazon Linux 등 기본 이미지) 선택
- **인스턴스 유형**: 워크로드에 맞는 유형 선택 (프리 티어라면 `t2.micro`/`t3.micro`)
- **키 페어**: 기존에 생성한 키 페어 선택
- **보안 그룹**: 앞서 만든 `sg-web` 선택
- **고급 세부 정보 - 사용자 데이터(User Data)**: 인스턴스가 처음 부팅될 때 자동으로 애플리케이션을 설치·실행하도록 스크립트를 등록한다 (예: `dnf install -y httpd`, `pm2 start src/health.js` 등)

### 4.2 Auto Scaling 그룹 생성

EC2 콘솔의 [Auto Scaling 그룹] 메뉴에서 [Auto Scaling 그룹 생성]을 클릭한다. 콘솔 절차는 [Create an Auto Scaling group using a launch template(AWS 공식 문서)](https://docs.aws.amazon.com/autoscaling/ec2/userguide/create-asg.html)를 참고한다.

1. **시작 템플릿 또는 구성 선택(Choose launch template or configuration)**: Auto Scaling 그룹 이름을 입력하고, 앞서 만든 시작 템플릿과 버전(기본값은 **Latest**)을 선택
2. **인스턴스 시작 옵션 선택(Choose instance launch options)**: 시작 템플릿에 인스턴스 유형이 지정돼 있다면 기본값 그대로 두고, **네트워크(Network)**에서 VPC와 가용 영역별 서브넷(2개 이상)을 선택
3. **다른 서비스와 통합(Integrate with other services)**: **로드 밸런싱(Load balancing)**에서 [기존 로드 밸런서에 연결]을 선택하고 앞서 만든 대상 그룹을 지정 — 이 설정으로 Auto Scaling이 생성하는 인스턴스가 자동으로 대상 그룹에 등록된다. **상태 검사(Health checks)**에서 상태 검사 유예 기간(Health check grace period)을 애플리케이션 부팅 시간에 맞게 설정(예: 90초)
4. **그룹 크기 및 조정 구성(Configure group size and scaling)**: **그룹 크기(Group size)**에서 Desired capacity를 입력하고, **조정 한도(Scaling limits)**에서 Min/Max desired capacity를 입력(예: Min 2 / Desired 2 / Max 4). **자동 조정(Automatic scaling)**에서 대상 추적 조정 정책(Target tracking scaling policy)을 선택 후 지표를 `평균 CPU 사용률`로 지정, 목표 값을 입력(예: 50%) — CPU 사용률이 목표치를 넘으면 자동으로 인스턴스를 추가(Scale Out)하고, 낮아지면 축소(Scale In)한다
5. **알림 추가·태그 추가**: 필요 없다면 건너뛰고 **검토(Review)** 단계에서 설정을 확인 후 [Auto Scaling 그룹 생성] 클릭

### 4.3 생성 확인

Auto Scaling 그룹 생성이 완료되면, 지정한 Desired capacity만큼 EC2 인스턴스가 자동으로 시작되고 대상 그룹에 등록된다. 대상 그룹의 [대상] 탭에서 인스턴스가 "정상(healthy)" 상태로 표시되는지 확인한다. 로드 밸런서의 DNS 이름으로 접속하면 Auto Scaling 그룹의 인스턴스 중 하나가 요청을 처리한 응답을 받을 수 있다.

이후 인위적으로 CPU 부하를 발생시키면(예: `stress` 명령 등) 대상 추적 정책의 임계값을 넘긴 시점에 CloudWatch 경보가 트리거되어 인스턴스가 자동으로 추가되는 것을 Auto Scaling 그룹의 [활동] 탭에서 확인할 수 있다.

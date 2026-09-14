# Amazon CloudFront

[AWS-T04. S3](t04-s3.md) 문서의 정적 웹 호스팅 개념, [AWS-T09. Route 53](t09-route53.md) 문서의 Alias Record 개념과 함께 보면 CloudFront+S3+Route53 조합을 이해하기 쉽다.

## 목차
1. [CloudFront 개요](#1-cloudfront-개요)
2. [엣지 로케이션](#2-엣지-로케이션)
3. [정적 콘텐츠 vs 동적 콘텐츠](#3-정적-콘텐츠-vs-동적-콘텐츠)
4. [Origin 개요](#4-origin-개요)
5. [S3 Origin 상세](#5-s3-origin-상세)
6. [Custom Origin 상세](#6-custom-origin-상세)
7. [Origin Group](#7-origin-group)
8. [Origin Custom Header](#8-origin-custom-header)
9. [Behavior](#9-behavior)
10. [Viewer 설정](#10-viewer-설정)
11. [Policy 설정](#11-policy-설정)
12. [CloudFront와 API 연동](#12-cloudfront와-api-연동)
13. [ALB를 함께 쓰는 이유](#13-alb를-함께-쓰는-이유)
14. [OAC (Origin Access Control)](#14-oac-origin-access-control)
15. [캐시 파일 관리: Invalidation vs 버저닝](#15-캐시-파일-관리-invalidation-vs-버저닝)
16. [CloudFront 설계 시 자주 놓치는 실무 포인트](#16-cloudfront-설계-시-자주-놓치는-실무-포인트)

---

## 1. CloudFront 개요

**Amazon CloudFront**는 AWS가 제공하는 글로벌 CDN(Content Delivery Network) 서비스이다. 웹페이지, 이미지, 동영상, API 응답 같은 콘텐츠를 원본 서버(Origin)에서 가져와 전 세계에 분산된 **엣지 로케이션(Edge Location)**에 캐싱해두고, 사용자는 자신과 물리적으로 가까운 엣지 로케이션에서 콘텐츠를 전달받는다.

**동작 방식**

사용자가 콘텐츠를 요청하면 CloudFront는 가까운 엣지 로케이션의 캐시를 먼저 확인한다. 캐시가 있으면 Origin까지 가지 않고 엣지 로케이션에서 즉시 응답하고, 캐시가 없으면 Origin에서 콘텐츠를 가져와 사용자에게 전달하면서 동시에 엣지 로케이션에 저장해 둔다. 이후 동일한 콘텐츠를 다른 사용자가 요청하면 이미 저장된 캐시로 바로 응답할 수 있다.

**주요 특징**

- **빠른 전송**: Origin과 사용자의 물리적 거리가 멀어도 가까운 엣지 로케이션에서 응답을 받기 때문에 지연 시간(Latency)이 줄어든다. Origin이 미국에 있어도 한국 사용자는 서울이나 도쿄 엣지 로케이션에서 응답을 받는 식이다.
- **일관된 콘텐츠 제공과 Origin 부하 감소**: 엣지 로케이션의 캐시는 설정된 만료 시간(Expiry Time)까지 유지된다. Origin의 원본이 바뀌어도 캐시가 갱신되기 전까지는 이전 콘텐츠가 계속 제공될 수 있는 대신, Origin으로 가는 요청 수와 서버 부하가 크게 줄어든다.
- **글로벌 서비스**: 전 세계에 분산된 다수의 엣지 로케이션을 이용하므로 전 세계 사용자를 대상으로 하는 서비스에 유리하다.
- **정적/동적 콘텐츠 모두 지원**: HTML/CSS/JS/이미지 같은 정적 콘텐츠뿐 아니라 API 응답 같은 동적 콘텐츠도 함께 전달할 수 있다. 자세한 차이는 [3. 정적 콘텐츠 vs 동적 콘텐츠](#3-정적-콘텐츠-vs-동적-콘텐츠)에서 다룬다.
- **보안 기능**: HTTPS로 사용자-CloudFront 구간 통신을 암호화하고, AWS WAF와 연동해 웹 공격을 방어하며, AWS Shield Standard가 기본으로 적용되어 DDoS 공격에 대응한다.

**정리**: CloudFront는 Origin의 콘텐츠를 엣지 로케이션에 캐싱해 사용자에게 더 빠르고, 더 적은 Origin 부하로 전달하는 CDN 서비스다. 캐시가 있으면 엣지에서 즉시 응답하고, 없으면 Origin에서 가져와 캐싱한 뒤 응답한다는 기본 동작을 이해하는 것이 이후 모든 개념의 출발점이다.

## 2. 엣지 로케이션

**엣지 로케이션(Edge Location)**은 CloudFront 같은 AWS 글로벌 서비스가 사용자와 가까운 위치에서 콘텐츠를 전달하기 위해 전 세계에 분산 배치한 네트워크 거점이다. 서울, 도쿄, 홍콩, 미국, 유럽 등 세계 각지에 위치하며, Origin까지 직접 가지 않고 엣지 로케이션에 캐싱된 콘텐츠를 전달함으로써 전송 속도를 높이고 Origin 부하를 줄인다.

**Region/AZ와의 차이**

엣지 로케이션은 EC2, RDS 같은 리소스를 생성하고 운영하는 Region/AZ와는 다른 개념이다. Region/AZ가 실제 컴퓨팅 자원이 배치되는 위치라면, 엣지 로케이션은 콘텐츠를 사용자에게 빠르게 전달하기 위한 캐싱·전송용 네트워크 거점이며 Region/AZ보다 훨씬 촘촘하게 전 세계에 분산되어 있다.

**Global Accelerator와의 관계**

엣지 로케이션은 AWS의 글로벌 네트워크를 이용해 연결 속도와 안정성을 높이는 별도 서비스인 **Global Accelerator**와 연계해서 활용할 수도 있다. CloudFront가 콘텐츠 캐싱에 초점을 맞춘 서비스라면, Global Accelerator는 AWS 백본 네트워크를 통해 사용자의 트래픽을 가장 가까운 AWS 엔드포인트로 라우팅하는 데 초점을 맞춘다.

**활용 사례**: 정적 파일 제공, 이미지/동영상 전송, 동영상 스트리밍, API 응답 속도 향상, 글로벌 온라인 쇼핑몰의 콘텐츠 전달 등에 널리 쓰인다.

**정리**: 엣지 로케이션은 Region/AZ와 목적이 다른 네트워크 거점으로, CloudFront가 콘텐츠를 사용자와 가까운 곳에서 전달할 수 있게 하는 핵심 인프라다. Global Accelerator와는 상호 보완적으로 함께 사용할 수 있다.

## 3. 정적 콘텐츠 vs 동적 콘텐츠

CloudFront가 다루는 콘텐츠는 크게 정적 콘텐츠와 동적 콘텐츠로 나뉘며, 이 구분은 이후 Origin 선택과 Behavior 설계의 기준이 된다.

**정적 콘텐츠(Static Contents)**: 서버에 저장된 파일이 모든 사용자에게 동일하게 전달되는 콘텐츠다. 사용자에 따라 내용이 달라지지 않고, 내용이 자주 바뀌지 않아 캐싱하기 좋다. 서버가 매번 새로 계산할 필요가 없어 응답이 빠르고 서버 부하도 적다. HTML/CSS/JS/이미지 등으로 구성되며 이미지, 글, 뉴스 기사, 정적 웹페이지가 대표적인 예다.

**동적 콘텐츠(Dynamic Contents)**: 시간, 사용자, 입력값 등에 따라 내용이 달라지는 콘텐츠다. 요청이 들어올 때마다 서버가 처리해서 새로운 결과를 생성하므로 사용자별로 다른 결과를 제공할 수 있지만, 서버 연산이 많이 필요해 상대적으로 느리고 부하가 커질 수 있다. PHP/JSP/ASP.NET/Node.js/Python 같은 서버사이드 기술로 구성되며 로그인 사용자 정보, 게시판, 댓글, 장바구니, 결제 페이지가 대표적인 예다.

**비교표**

| 구분 | 정적 콘텐츠 | 동적 콘텐츠 |
|---|---|---|
| 내용 변화 | 모든 사용자에게 동일 | 사용자/시점/입력값에 따라 다름 |
| 처리 방식 | 미리 만들어진 파일을 그대로 전달 | 요청마다 서버가 연산해서 생성 |
| 구성 기술 | HTML, CSS, JS, 이미지 | PHP, JSP, ASP.NET, Node.js, Python 등 |
| 응답 속도 | 빠름 | 상대적으로 느릴 수 있음 |
| 서버 부하 | 적음 | 요청마다 부하 증가 가능 |
| 장점 | 응답 속도가 빠르고 CDN과 궁합이 좋음 | 개인화된 정보 제공, 로그인/주문/결제 등 구현 가능 |
| 단점 | 개인화 서비스에 부적합, 수정 시 파일을 직접 교체 필요 | 서버 부하와 개발·관리 복잡도가 높음 |

**정리**: 정적 콘텐츠는 캐싱에 최적화되어 있어 S3 Origin과 궁합이 좋고, 동적 콘텐츠는 매 요청을 서버가 처리해야 하므로 EC2/ALB 같은 Custom Origin이 필요하다. 이 구분이 뒤에서 다룰 Origin 종류 선택과 Behavior 라우팅 설계의 기본 축이 된다.

## 4. Origin 개요

**Origin**은 CloudFront가 사용자에게 전달할 콘텐츠를 실제로 가지고 있는 원본 서버 또는 저장소다. CloudFront는 엣지 로케이션에 캐시가 없을 때만 Origin에서 콘텐츠를 가져온다.

**Origin 종류**

- **Amazon S3 Origin**: S3 버킷을 Origin으로 사용한다. 이미지, HTML, CSS, JS, 동영상 등 정적 콘텐츠 제공에 주로 쓰이며 별도의 웹 서버 운영이 필요 없다. S3 버킷 정책으로 접근 권한을 제어할 수 있다.
- **Custom Origin**: S3 Origin을 제외한 나머지 서버(EC2, ALB, 온프레미스 서버, 외부 웹 서버 등)를 Origin으로 사용한다. 주로 웹 애플리케이션이나 동적 콘텐츠를 제공할 때 쓰인다.

**Origin 제한 사항**: 하나의 CloudFront Distribution에는 여러 Origin을 등록할 수 있다. 각 Origin은 [9. Behavior](#9-behavior)와 연결되어, 어떤 URL 경로의 요청을 어느 Origin으로 보낼지 지정할 수 있다.

**활용 예시**: 정적 콘텐츠는 S3, 동적 콘텐츠는 EC2/ALB로 나누어 등록하면, CloudFront 하나로 여러 Origin의 콘텐츠를 하나의 도메인 아래에서 서비스할 수 있다. 구체적인 라우팅 방법은 [9. Behavior](#9-behavior)와 [12. CloudFront와 API 연동](#12-cloudfront와-api-연동)에서 다룬다.

**정리**: Origin은 CloudFront가 콘텐츠를 가져오는 원천이며, S3 Origin과 Custom Origin 중 무엇을 쓸지는 정적/동적 콘텐츠 구분에 따라 결정된다. 하나의 Distribution에 여러 Origin을 섞어 등록하는 것이 CloudFront 설계의 기본 패턴이다.

## 5. S3 Origin 상세

**도메인 형식**

S3 Origin의 기본 도메인 형식은 `{bucketname}.s3.{region}.amazonaws.com`이다. `s3.amazonaws.com/{bucketname}` 같은 경로 스타일 형식은 Origin 도메인으로 사용하지 않는다.

S3 정적 웹사이트 호스팅 기능을 사용하는 경우에는 도메인 형식이 `http://{bucketname}.s3-website-{region}.amazonaws.com`으로 달라지며, 이 경우 CloudFront에서는 일반 S3 Origin이 아니라 **Custom Origin**으로 등록해서 처리한다.

**S3 전용 보안 기능: OAI vs OAC**

| 구분 | OAI (Origin Access Identity) | OAC (Origin Access Control) |
|---|---|---|
| 성격 | 레거시 방식 | 현재 권장되는 방식 |
| 기능 | 사용자가 S3에 직접 접근하지 못하게 하고 CloudFront를 통해서만 접근하도록 제한 | CloudFront가 S3에 안전하게 접근하도록 제어 |
| 권한 제어 | 상대적으로 단순 | 더 세분화된 권한 제어와 서명 기능 제공 |

자세한 동작 구조와 활용 방법은 [14. OAC](#14-oac-origin-access-control)에서 다룬다.

**HTTP Method**: 기본적으로 GET으로 콘텐츠를 조회하며, 설정에 따라 PUT/POST/DELETE 같은 메서드도 허용할 수 있다.

**사용 사례**: 정적 웹사이트 배포, HTML/CSS/JS/이미지 제공, 동영상·음악·PDF 같은 대용량 파일 전송, 다운로드 파일 제공에 주로 활용한다.

**정리**: S3 Origin은 올바른 도메인 형식(`{bucket}.s3.{region}.amazonaws.com`)을 사용해야 정상 연결되며, S3를 Private으로 유지하면서 CloudFront 전용 접근만 허용하려면 OAI보다 최신 방식인 OAC를 사용하는 것이 표준이다.

## 6. Custom Origin 상세

**대표적인 Custom Origin 유형**

- **MediaStore**: 미디어 스트리밍 콘텐츠 제공에 사용한다.
- **S3 Static Hosting**: S3 정적 웹사이트 호스팅 주소를 Origin으로 사용하는 경우로, 일반 S3 Origin이 아닌 Custom Origin으로 처리된다.
- **Lambda Function URL**: Lambda 함수가 직접 반환하는 응답을 CloudFront로 제공한다.
- **Application Load Balancer(ALB)**: 여러 EC2에 트래픽을 분산하면서 CloudFront의 Origin으로 사용한다.
- **EC2 또는 기타 HTTP 서버**: Apache, Nginx, Spring, Node.js 등 직접 운영하는 서버를 Origin으로 사용한다.

**접속 방식 제약**: Custom Origin은 HTTP/HTTPS로 접속하며, Origin은 반드시 **도메인 이름**으로 지정해야 한다. IP 주소는 Origin으로 사용할 수 없다.

**Origin Group과의 관계**: 여러 Origin을 Primary/Secondary로 묶어 장애 시 자동 전환하는 Origin Group 기능을 Custom Origin에 적용해 고가용성을 확보할 수 있다. 자세한 내용은 [7. Origin Group](#7-origin-group)에서 다룬다.

**활용 예시**: EC2에서 API나 동적 콘텐츠를 제공하는 경우, ALB로 여러 EC2에 트래픽을 분산하는 경우, 온프레미스 서버나 다른 클라우드의 웹 서버를 연결하는 경우 등에 Custom Origin을 사용한다.

**정리**: Custom Origin은 S3를 제외한 모든 서버를 Origin으로 등록할 때 쓰는 개념이며, 도메인 이름으로만 지정 가능하다는 제약을 지켜야 한다. 동적 콘텐츠나 API 서버를 CloudFront에 연결할 때 사실상 필수적인 구성 방식이다.

## 7. Origin Group

**Origin Group**은 여러 Origin을 Primary/Secondary로 묶어 Failover를 구성하는 기능이다. 평상시에는 Primary Origin에서 콘텐츠를 가져오다가, Primary에 장애가 발생하면 자동으로 Secondary Origin으로 전환한다.

**기본 구조**

```
사용자 → CloudFront → Origin Group → (Primary Origin / Secondary Origin)
```

**Failover 동작 조건**: 다음 중 하나라도 해당하면 Secondary로 전환한다.

- Primary에서 지정된 HTTP 오류 코드가 발생한 경우
- Primary와 네트워크 연결이 불가능한 경우
- 요청 시간이 초과된 경우
- 설정된 재시도 이후에도 정상 응답이 없는 경우

**적용 가능한 요청**: GET, HEAD, OPTIONS처럼 읽기 중심의 요청에만 Origin Group Failover가 적용된다.

**추가 기능**: Primary와 Secondary가 모두 실패하면 사용자 정의 에러 페이지(예: "현재 서버 점검 중입니다.")를 표시하도록 구성할 수 있다.

**활용 예시**: Primary EC2 장애 시 Secondary EC2로 전환하는 경우, Primary 웹 서버 장애 시 S3에 미리 준비해 둔 정적 장애 안내 페이지로 전환하는 경우, 특정 리전 장애 시 다른 리전의 Origin으로 전환하는 경우 등이 있다. 서비스 장애 시 자동으로 대체 Origin을 사용하게 함으로써 서비스 중단을 최소화하는 것이 핵심 목적이다.

**정리**: Origin Group은 Primary Origin에 문제가 생겼을 때 CloudFront가 자동으로 Secondary Origin으로 전환하는 고가용성 기능이다. GET/HEAD/OPTIONS 같은 읽기 요청에만 적용되며, 두 Origin 모두 실패했을 때를 대비한 사용자 정의 에러 페이지까지 함께 준비해두는 것이 실무 패턴이다.

## 8. Origin Custom Header

**Origin Custom Header**는 CloudFront가 Origin으로 요청을 보낼 때, 사용자가 지정한 추가 Header를 함께 전달하는 기능이다. 클라이언트가 같은 이름의 Header를 보내더라도, CloudFront에서 설정한 값으로 덮어써서 Origin에 전달할 수 있다.

**활용 예시**

- **보안 키 전달**: CloudFront만 아는 Header 값을 Origin에 전달하고, Origin은 이 값이 포함된 요청만 허용하도록 구성해 사용자가 Origin에 직접 접근하는 것을 제한할 수 있다.
- **API Key 전달**: Origin이 요구하는 API Key를 CloudFront 단에서 삽입해 전달한다.
- **테스트/버전 구분용 Header**: 배포 버전이나 테스트 환경을 구분하는 값을 전달한다.
- **접근 제어**: 특정 Header 값을 가진 요청만 Origin이 허용하도록 구성한다.

**정리**: Origin Custom Header는 CloudFront와 Origin 사이의 통신에만 존재하는 값을 추가해, 사용자가 Origin을 우회해서 직접 접근하는 것을 막거나 Origin 쪽 인증·구분 로직을 단순화하는 데 활용된다.

## 9. Behavior

**Behavior(동작)**는 CloudFront가 요청 경로와 특성에 따라 어느 Origin으로 보낼지, 어떻게 캐시할지, 어떤 정책을 적용할지를 정의하는 규칙 집합이다.

**매칭 방식**: 각 Behavior는 경로 패턴(Path Pattern)을 가진다(예: `*` 기본, `/images/*`, `*.png`, `/api/*`). 목록의 위에서부터 첫 번째로 매칭되는 규칙이 적용되므로, 가장 범위가 넓은 기본 규칙(`*`)은 보통 맨 아래에 두고, `/api/*`나 `/static/*`처럼 구체적인 규칙을 위쪽에 배치한다.

**Behavior와 Origin(또는 Origin Group) 연결**: Behavior마다 연결할 Origin을 지정한다. 예를 들어 `/static/*`는 S3로, `/api/*`는 ALB/EC2로, 기본값 `*`는 기본 Origin으로 연결하는 식이다. 장애 조치가 필요하다면 [7. Origin Group](#7-origin-group)과 함께 사용한다.

**Cache Behavior 주요 구성 요소**

1. **Origin(원본)**: Behavior 규칙마다 서로 다른 Origin을 연결할 수 있다.
2. **뷰어 설정(Viewer Settings)**: 사용자가 CloudFront에 요청을 보낼 때 어떻게 처리할지 정한다. 자세한 내용은 [10. Viewer 설정](#10-viewer-설정)에서 다룬다.
3. **추가 정책 연결**: Cache Policy(캐시 구분 기준과 TTL), Response Headers Policy(응답 헤더 정책), Lambda@Edge(CloudFront 경계 지점에서 실행되는 코드) 등을 연결할 수 있다. 자세한 내용은 [11. Policy 설정](#11-policy-설정)에서 다룬다.

**정리**: Behavior는 "어떤 경로 요청을 어느 Origin으로, 어떤 정책으로 처리할지"를 결정하는 CloudFront의 핵심 라우팅 규칙이다. 경로 패턴의 매칭 순서(구체적인 규칙을 위로)를 지키는 것이 의도한 대로 라우팅되게 하는 핵심 포인트다.

## 10. Viewer 설정

CloudFront에서 콘텐츠를 요청하는 클라이언트(사용자 브라우저나 앱)를 **Viewer**라고 한다.

**1) Viewer 프로토콜**

| 옵션 | 설명 |
|---|---|
| HTTP and HTTPS | 둘 다 허용. 실습 초반엔 편하지만 보안상 비권장 |
| Redirect HTTP to HTTPS | HTTP 접속 시 자동으로 HTTPS로 리다이렉트. 보안과 편의를 함께 잡을 수 있어 가장 많이 쓰는 옵션 |
| HTTPS only | HTTPS만 허용. 가장 강력한 보안 옵션 |

**2) HTTP Method**: 정적 콘텐츠는 GET/HEAD 정도면 충분하다. API 요청처럼 동적 처리가 필요한 경우에는 GET/HEAD/OPTIONS/PUT/POST/PATCH/DELETE까지 허용할 수 있다. OPTIONS는 보통 CORS(교차 출처 요청) 확인용으로 필요하다.

**3) 뷰어 액세스 제한(콘텐츠 접근 제어)**: Presigned URL이나 Presigned Cookie를 가진 사용자만 접근하도록 제한할 수 있다. 결제한 사용자만 접근 가능한 영상 스트리밍 서비스, 프리사인드 URL을 발급받은 사람만 다운로드할 수 있는 S3의 개인 파일 등이 대표적인 활용 예다.

**정리**: Viewer 설정은 사용자와 CloudFront 사이의 프로토콜, 허용 메서드, 접근 제한을 정하는 설정이다. 실무에서는 대부분 "Redirect HTTP to HTTPS"를 기본으로 사용하고, 민감한 콘텐츠에는 Presigned URL/Cookie 기반 접근 제한을 추가한다.

## 11. Policy 설정

**Policy**란 CloudFront에서 어떤 규칙대로 요청과 응답을 다룰지 정리해 둔 설정 묶음이다. 캐시·요청·응답 처리 방식을 표준화해 여러 Behavior에서 재사용할 수 있게 한다. 예를 들어 "모든 정적 파일은 1시간 캐시하고 특정 헤더만 전달한다"는 규칙을 Policy로 만들어두면 다른 Behavior에도 그대로 재사용할 수 있다.

**1) Cache Policy(캐시 정책)**

무엇을 기준으로 캐시를 다르게 저장할지 결정한다.

- HTTP Header: 브라우저, 언어, 인증 정보 등에 따라 캐시를 구분한다.
- 쿠키: 로그인 상태나 세션 값에 따라 캐시를 구분한다.
- 쿼리스트링: URL 파라미터 값(예: `?id=1`)에 따라 캐시를 구분한다.

TTL(최소 TTL/기본 TTL/최대 TTL)로 얼마 동안 캐시할지 정하며, 브라우저가 gzip/brotli를 지원하면 자동으로 압축하는 설정도 포함한다.

**2) Origin Request Policy(원본 요청 정책)**: CloudFront가 Origin(EC2, S3 등)으로 요청을 보낼 때 어떤 헤더, 쿠키, 쿼리스트링을 함께 전달할지 결정한다. 캐시 정책과는 별개로 Origin에 전달되는 내용을 세밀하게 조정할 수 있다.

**3) Response Headers Policy(응답 헤더 정책)**: Origin이 응답을 보낸 뒤, CloudFront가 최종적으로 Viewer에게 돌려줄 때 헤더를 추가·수정·삭제할 수 있다. 예를 들어 email, phone, user_id 같은 민감한 정보가 담긴 헤더를 응답에서 제거하고, `service:prodapp` 같은 커스텀 헤더를 추가하는 식으로 활용한다.

**정리**: Cache Policy는 "무엇을 기준으로 캐시할지", Origin Request Policy는 "Origin에 무엇을 전달할지", Response Headers Policy는 "Viewer에게 무엇을 돌려줄지"를 각각 담당한다. 세 정책은 서로 독립적으로 조합 가능하며, Behavior 단위로 재사용할 수 있다.

## 12. CloudFront와 API 연동

**API(Application Programming Interface)**는 프로그램끼리 서로 정보를 주고받기 위한 약속된 통신 방법이다. 웹에서는 주로 HTTP/HTTPS로 API를 요청하며, `/api/hello`, `/api/login`, `/api/products`, `/api/order` 같은 주소가 대표적인 API 엔드포인트 형태다.

**일반 파일과 API의 차이**

정적 파일(`index.html`, `style.css`, `app.js`, `logo.png` 등)은 미리 만들어진 파일을 그대로 전달하는 것이라 서버 계산이 필요 없고, 주로 S3에서 제공하기 좋다. 반면 API 요청(로그인, 회원가입, 상품 조회, 게시글 작성, 주문 처리 등)은 서버 프로그램이 요청을 받아 직접 필요한 작업을 수행해야 한다.

| 구분 | 정적 파일 요청 | API 요청 |
|---|---|---|
| 흐름 | 사용자 → CloudFront → S3 → HTML/CSS/JS/이미지 전달 | 사용자 → CloudFront → ALB → EC2 → 서버 프로그램 처리 → 결과 반환 |
| 서버 계산 | 불필요 | 필요 |

**CloudFront에서 API를 사용하는 이유**: CloudFront는 하나의 배포에 여러 Origin을 연결할 수 있으므로, 정적 파일은 S3로, API 요청은 ALB/EC2로 나눌 수 있다. [9. Behavior](#9-behavior)로 요청 경로별 Origin을 구분한다.

예를 들어 `https://example.com/index.html`은 `/api/`로 시작하지 않으므로 기본 Behavior(`*`)가 적용되어 S3에서 파일을 전달받는다. 반면 `https://example.com/api/hello`는 `/api/`로 시작하므로 `/api/*` Behavior가 적용되어 ALB로 전달되고, ALB가 EC2로 전달한 뒤 EC2가 API 요청을 처리해 결과를 반환한다.

**정리**: 정적 파일과 API 요청은 처리 방식이 근본적으로 다르므로, CloudFront의 Behavior 기능을 활용해 경로별로 서로 다른 Origin(S3 vs ALB/EC2)으로 분기시키는 것이 표준적인 설계다.

## 13. ALB를 함께 쓰는 이유

**ALB(Application Load Balancer)**는 여러 EC2 인스턴스에 요청을 분산하는 역할을 한다. API 서버가 EC2 한 대뿐이라면 해당 인스턴스에 장애가 생겼을 때 서비스 전체가 중단될 수 있다.

여러 EC2와 ALB를 함께 구성하면 요청이 여러 서버로 분산되고, ALB는 정상적으로 동작 중인 EC2에만 요청을 전달한다. 이 구성을 CloudFront의 Custom Origin으로 등록하면, CloudFront는 ALB 하나만 바라보면서도 그 뒤에서 여러 EC2가 트래픽을 분산 처리하는 고가용성 구조를 얻을 수 있다.

**정리**: CloudFront가 캐싱과 전 세계 전달을 담당한다면, ALB는 그 뒤에서 여러 EC2 사이의 트래픽 분산과 장애 격리를 담당한다. 동적 API를 안정적으로 서비스하려면 EC2 단독보다 ALB+EC2 조합을 Custom Origin으로 쓰는 편이 실무에서 일반적이다.

## 14. OAC (Origin Access Control)

**OAC(Origin Access Control)**는 CloudFront가 S3 같은 Origin에 안전하게 접근하도록 권한을 제어하는 기능이다. S3 버킷을 Public으로 공개하지 않고 Private 상태로 유지하면서도, CloudFront를 통해 들어오는 요청만 S3에서 허용하도록 만들 수 있다. 즉 S3 콘텐츠를 인터넷에 직접 노출하지 않고 CloudFront를 통해서만 제공하도록 하는 보안 기능이다.

**동작 구조**

```
사용자 → CloudFront → OAC를 이용한 인증된 요청 → Private S3 Bucket
```

**장점**

- S3 버킷을 Public으로 열 필요가 없다.
- 사용자가 S3 주소로 직접 접근하는 것을 차단한다.
- CloudFront를 통해서만 콘텐츠가 제공되므로 보안성이 향상된다.

OAC는 [5. S3 Origin 상세](#5-s3-origin-상세)에서 다룬 OAI(Origin Access Identity)의 후속 기능으로, 더 세분화된 권한 제어와 서명 기능을 제공하는 현재의 권장 방식이다.

**정리**: OAC는 S3를 Private으로 유지하면서 CloudFront 전용 접근만 허용하는 표준 방식이다. S3 버킷을 Public Access Block 상태로 두고 OAC를 설정하면, S3 직접 URL로는 접근이 거부되고 CloudFront URL로만 정상 응답을 받을 수 있다.

## 15. 캐시 파일 관리: Invalidation vs 버저닝

CloudFront는 한 번 가져온 파일을 엣지 로케이션에 캐싱해서 전달한다. 원본 파일이 수정되어도 CloudFront에 예전 파일이 캐시되어 있으면, 사용자는 일정 시간 동안 예전 파일을 계속 받을 수 있다. 이 문제를 다루는 방법은 크게 두 가지다.

**1) 싱글 파일 관리 방식**

처음 요청 시 Origin에서 가져와 캐싱한 뒤 전달한다. 이후 Origin의 파일이 수정되어도 CloudFront에는 이전 캐시가 남아있을 수 있어, 사용자가 다시 요청하면 기존 캐시 파일을 받을 수 있다. 최신 파일을 바로 적용하려면 **Invalidation(캐시 무효화)**을 사용해야 한다.

- 장점: 파일 이름을 계속 동일하게 사용할 수 있어 HTML/JS에서 경로를 변경할 필요가 없다.
- 단점: 파일을 수정해도 캐시 때문에 바로 최신 파일이 보이지 않을 수 있고, 즉시 적용하려면 Invalidation 작업이 필요하다.

**Invalidation 상세**

CloudFront 캐시에 저장된 파일을 강제로 무효화해서 Origin에서 최신 파일을 다시 가져오게 만드는 과정이다. 파일 이름을 동일하게 유지하며 내용만 바꾼 경우(버저닝 방식이 아닐 때) 캐시된 예전 버전이 계속 전달될 수 있으므로, Invalidation으로 캐시를 지워야 최신 파일을 받을 수 있다.

경로(Path) 기반으로 무효화 범위를 지정한다.

| 경로 패턴 | 효과 |
|---|---|
| `/img/img1.png` | 단일 파일만 무효화 |
| `/img/*` | 폴더 안 모든 파일 무효화 |
| `/img/img*` | 특정 접두사로 시작하는 파일들 무효화 |

**제한과 비용**: 한 번 요청으로 최대 3,000개 파일까지 무효화할 수 있다(예: 100개씩 30번, 또는 1,000개씩 3번 요청). 한 달에 1,000개 경로(Path)까지는 무료이며(모든 Distribution 합산 기준), 무료 횟수를 초과하면 경로당 약 $0.005의 비용이 발생한다.

**2) 버저닝 관리 방식**

파일을 수정할 때 파일 이름에 버전 정보를 붙여 새 파일로 만드는 방식이다(예: `style_v1.css` → `style_v2.css`). CloudFront는 파일 이름이 다르면 서로 다른 파일로 인식하므로, `style_v1.css`와 `style_v2.css`는 완전히 별개의 파일로 처리된다. 새로운 `style_v2.css`를 요청하면 기존 캐시와 무관하게 Origin에서 새 파일을 가져와 캐싱한다.

동작 순서: 새 버전 파일 생성 → HTML에서 파일 경로 변경 → CloudFront가 새 파일로 인식 → Origin에서 새 파일을 가져와 캐싱.

- 장점: 기존 캐시를 삭제할 필요가 없고, Invalidation 없이 새 파일을 바로 배포할 수 있다.
- 단점: 파일 이름이 바뀌므로 HTML/JS에서 사용하는 파일 경로도 새 버전으로 함께 수정해야 한다.

**두 방식 비교**

| 구분 | 싱글 파일 관리(Invalidation) | 버저닝 관리 |
|---|---|---|
| 파일 이름 | 동일하게 유지 | 버전마다 변경 |
| 즉시 반영 | Invalidation 실행 필요 | 파일명이 곧 새 캐시 키이므로 즉시 반영 |
| 추가 비용 | 무료 한도(월 1,000경로) 초과 시 경로당 약 $0.005 | 없음 |
| 참조 코드 수정 | 불필요 | 필요(HTML/JS의 파일 경로 변경) |
| 실무 활용 | 파일명을 고정해야 하는 환경, 긴급 수정 | 정적 자산 배포 파이프라인(빌드 시 해시 자동 부여) |

실무에서는 `app-v1.abc123.js` → `app-v2.abc456.js`처럼 콘텐츠 해시를 파일명에 붙이는 방식으로 캐시 무효화 없이 버저닝을 관리하는 경우가 많다. 빌드 도구가 파일 내용을 기준으로 해시를 자동 생성해주므로, 내용이 바뀌지 않은 파일은 캐시가 계속 유지되고 실제로 바뀐 파일만 새 캐시 키를 갖게 된다.

**정리**: 싱글 파일 관리 방식은 파일명을 고정할 수 있지만 변경 반영을 위해 Invalidation을 실행해야 하고, 버저닝 관리 방식은 Invalidation 없이 즉시 반영되지만 참조 경로를 함께 수정해야 한다. 실무에서는 정적 자산 배포에는 버저닝(해시 기반 파일명)을, 급한 수정이나 파일명을 바꿀 수 없는 상황에는 Invalidation을 활용하는 식으로 병행하는 경우가 많다.

## 16. CloudFront 설계 시 자주 놓치는 실무 포인트

| 흔한 실수 | 원인 | 방지 방법 |
|---|---|---|
| S3 Origin 도메인 형식을 잘못 입력(`s3.amazonaws.com/{bucket}`)해서 연결 실패 | 경로 스타일 주소를 Origin 도메인으로 착각 | 반드시 `{bucketname}.s3.{region}.amazonaws.com` 형식을 사용한다 |
| Custom Origin에 IP 주소를 입력해서 등록 실패 | Custom Origin을 EC2 IP로 바로 연결하려 함 | Custom Origin은 도메인 이름으로만 지정 가능하다는 제약을 지킨다 |
| Behavior 경로 패턴 순서를 잘못 배치해 의도한 라우팅이 안 됨 | 범위가 넓은 `*` 규칙을 구체적인 `/api/*` 규칙보다 위에 둠 | 구체적인 경로 패턴을 목록 위쪽에, 가장 넓은 기본 규칙(`*`)을 맨 아래에 배치한다 |
| Invalidation 없이 싱글 파일 방식으로 배포해 캐시가 갱신되지 않음 | 파일 이름을 그대로 두고 내용만 교체 | 즉시 반영이 필요하면 Invalidation을 실행하거나, 처음부터 버저닝(해시 기반 파일명) 방식으로 배포한다 |
| OAC 설정 없이 S3를 Public으로 열어버리는 보안 실수 | CloudFront 연결이 안 되는 문제를 S3를 공개해서 해결하려 함 | S3는 Private(Block Public Access)으로 유지하고, OAC로 CloudFront 전용 접근만 허용한다 |

**정리**: CloudFront 설계에서 반복되는 실수는 대부분 "Origin 도메인/주소 형식을 잘못 지정하는 경우", "Behavior 우선순위를 잘못 배치하는 경우", "캐시 갱신 전략 없이 배포하는 경우", "보안을 위해 만든 기능(OAC)을 쓰지 않고 우회하는 경우" 네 가지로 요약된다. Origin·Behavior·캐시 관리·OAC 각각의 규칙을 정확히 이해해두면 대부분 예방할 수 있다.

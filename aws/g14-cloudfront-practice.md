# CloudFront 실습 (OAC 접근 제어 · Origin Group Failover · Behavior 라우팅)

## 목차
1. [실습 개요](#1-실습-개요)
2. [실습 1: S3 Origin + OAC로 CloudFront 전용 접근 구성](#2-실습-1-s3-origin--oac로-cloudfront-전용-접근-구성)
3. [실습 2: Origin Group으로 Primary/Secondary EC2 장애 조치](#3-실습-2-origin-group으로-primarysecondary-ec2-장애-조치)
4. [실습 3: Behavior로 정적 콘텐츠와 API 요청 분기](#4-실습-3-behavior로-정적-콘텐츠와-api-요청-분기)
5. [실습 4 (선택): 캐시 무효화 vs 버저닝 실습 팁](#5-실습-4-선택-캐시-무효화-vs-버저닝-실습-팁)

---

## 1. 실습 개요

이 실습은 [AWS-T11. CloudFront](t11-cloudfront.md) 문서에서 다룬 개념을 실제 콘솔에서 구성해보는 실습이다. 먼저 해당 문서를 훑어보고 오면 이해가 쉽다.

이번 실습에서는 세 가지 시나리오를 순서대로 다룬다.

1. **OAC 접근 제어**: S3 버킷을 Private으로 유지한 채, CloudFront를 통해서만 접근되도록 구성한다.
2. **Origin Group Failover**: Primary/Secondary EC2를 준비하고, Primary 장애 시 Secondary로 자동 전환되는 것을 확인한다.
3. **Behavior 라우팅**: 하나의 Distribution 안에서 정적 콘텐츠(S3)와 API 요청(ALB→EC2)을 경로별로 분기한다.

**실습 시 참고**: 아래 절차에 등장하는 계정 ID, CloudFront 도메인, EC2 퍼블릭/프라이빗 IP, ALB DNS는 모두 예시로 마스킹된 값이다. 실제로는 자신의 콘솔에서 발급받은 값으로 치환해서 진행한다.

## 2. 실습 1: S3 Origin + OAC로 CloudFront 전용 접근 구성

### 2-1. S3 버킷 생성 및 이미지 업로드

**S3** 콘솔에서 [Create bucket]을 클릭해 Origin으로 사용할 버킷을 만든다.

- **Bucket name**: `my-test-origin-bucket-<ACCOUNT_ID>-ap-northeast-2-an` (버킷 이름에 계정 ID가 포함되는 것은 이름 중복을 피하기 위한 관례다)
- **Block Public Access settings**: 기본값(전체 차단) 그대로 유지한다. 이 실습의 핵심은 S3를 Public으로 열지 않고도 CloudFront로만 접근시키는 것이다.

버킷이 생성되면 [Upload]로 `cat.jpg`, `dog.jpg` 같은 이미지 파일을 업로드한다.

### 2-2. CloudFront Distribution 생성 및 OAC 설정

**CloudFront** 콘솔에서 [Create distribution]을 클릭한다.

- **Origin domain**: 방금 만든 S3 버킷을 선택한다 (`my-test-origin-bucket-<ACCOUNT_ID>-ap-northeast-2-an.s3.ap-northeast-2.amazonaws.com` 형식으로 자동 채워진다)
- **Origin access**: [Origin access control settings (recommended)]를 선택한다.
- [Create control setting]을 클릭해 새 OAC를 생성한다.
  - **Name**: 기본값 그대로 사용해도 무방하다.
  - **Signing behavior**: `Sign requests (recommended)`를 선택한다.
- OAC를 생성하고 나면 콘솔 화면에 "이 서명 설정이 동작하려면 S3 버킷 정책을 업데이트해야 한다"는 안내 문구와 함께 정책 예시가 표시된다. [Copy policy]로 복사해 둔다.
- **Viewer protocol policy**: `Redirect HTTP to HTTPS`를 선택한다.
- 나머지 옵션은 기본값으로 두고 [Create distribution]으로 생성을 완료한다.

### 2-3. S3 버킷 정책에 OAC 정책 반영

**S3** 콘솔에서 방금 만든 버킷의 [Permissions] 탭 → [Bucket policy] → [Edit]로 이동해, CloudFront 콘솔에서 복사해 둔 정책을 붙여넣고 저장한다. 이 정책은 "해당 CloudFront Distribution을 통해 들어오는 요청만 `s3:GetObject`를 허용한다"는 내용으로, S3를 Public으로 열지 않고도 CloudFront 경유 접근만 허용하게 만든다.

### 2-4. 검증: S3 직접 접근 차단 vs CloudFront 경유 접근

CloudFront Distribution이 `Deployed` 상태가 될 때까지 기다린 뒤 두 가지 URL로 접속해 비교한다.

**S3 버킷에 직접 접속(차단되어야 정상)**

```
https://my-test-origin-bucket-<ACCOUNT_ID>-ap-northeast-2-an.s3.ap-northeast-2.amazonaws.com/cat.jpg
https://my-test-origin-bucket-<ACCOUNT_ID>-ap-northeast-2-an.s3.ap-northeast-2.amazonaws.com/dog.jpg
```

아래와 같은 `AccessDenied` XML 에러가 반환되면 S3가 Private 상태로 잘 유지되고 있는 것이다.

```xml
This XML file does not appear to have any style information associated with it. The document tree is shown below.
<Error>
<Code>AccessDenied</Code>
<Message>Access Denied</Message>
<RequestId><REQUEST_ID></RequestId>
<HostId><HOST_ID></HostId>
</Error>
```

**CloudFront(엣지 로케이션)를 통한 접속(정상 응답되어야 함)**

```
https://d111111abcdef8.cloudfront.net/cat.jpg
https://d111111abcdef8.cloudfront.net/dog.jpg
```

두 이미지 모두 정상적으로 로드되면, S3는 Private 상태를 유지하면서도 CloudFront를 통한 요청만 OAC로 인증되어 허용되고 있음이 확인된 것이다.

## 3. 실습 2: Origin Group으로 Primary/Secondary EC2 장애 조치

### 3-1. Primary/Secondary EC2 준비

EC2 2대를 실행하면서, Launch Template의 **User Data**에 아래 스크립트를 등록해 정상 페이지(`page.html`)와 장애 안내용 백업 페이지(`backup.html`)를 자동으로 생성하도록 구성한다.

```bash
#!/bin/bash
sudo -s
sudo yum install -y httpd
systemctl start httpd
chkconfig httpd on
TOKEN=$(curl -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 21600")
INSTANCE_ID=$(curl -s -H "X-aws-ec2-metadata-token: $TOKEN" http://169.254.169.254/latest/meta-data/instance-id)

echo "<h1>hello,world!</h1>" >> /var/www/html/page.html
echo "<h1>$INSTANCE_ID</h1>" >> /var/www/html/page.html

echo "<h1>OMG its 404</h1>" >> /var/www/html/backup.html
echo "<h1>$INSTANCE_ID</h1>" >> /var/www/html/backup.html
```

이 템플릿으로 EC2를 두 대 실행해 각각 Primary, Secondary로 사용한다.

### 3-2. 두 EC2에 동일한 장애 안내 이미지 배포

로컬(또는 CloudShell)에서 준비한 `404-error.jpg`를 `scp`로 두 EC2에 각각 전송한다.

```bash
scp -i My-EC2-KeyPair.pem ./404-error.jpg ec2-user@<EC2_PRIMARY_PUBLIC_IP>:/home/ec2-user/
scp -i My-EC2-KeyPair.pem ./404-error.jpg ec2-user@<EC2_SECONDARY_PUBLIC_IP>:/home/ec2-user/
```

**Primary EC2에 SSH 접속 후 이미지를 웹 루트로 옮기고 `backup.html`을 수정한다.**

```bash
# 전송된 이미지 확인
[root@<PRIMARY_PRIVATE_IP> ec2-user]# ls -l
total 56
-rw-r--r--. 1 ec2-user ec2-user 56914 Sep 14 03:57 404-error.jpg

# 이미지를 html 디렉터리로 복사
[root@<PRIMARY_PRIVATE_IP> ec2-user]# cp /home/ec2-user/404-error.jpg /var/www/html/

# 복사 확인
[root@<PRIMARY_PRIVATE_IP> ec2-user]# ls -l /var/www/html/
total 60
-rw-r--r--. 1 root root 56914 Sep 14 04:02 404-error.jpg
-rw-r--r--. 1 root root    12 Sep 14 03:29 backup.html

# 404 에러 이미지를 출력하도록 backup.html 수정
[root@<PRIMARY_PRIVATE_IP> ec2-user]# vi /var/www/html/backup.html
<img src="/404-error.jpg" alt="404 Error Image" style="max-width:600px;">
```

**Secondary EC2에도 동일한 작업을 반복한다.**

```bash
[root@<SECONDARY_PRIVATE_IP> ec2-user]# ls -l
total 56
-rw-r--r--. 1 ec2-user ec2-user 56914 Sep 14 03:58 404-error.jpg
[root@<SECONDARY_PRIVATE_IP> ec2-user]# cp /home/ec2-user/404-error.jpg /var/www/html/
[root@<SECONDARY_PRIVATE_IP> ec2-user]# ls -l /var/www/html/
total 60
-rw-r--r--. 1 root root 56914 Sep 14 04:02 404-error.jpg
-rw-r--r--. 1 root root    12 Sep 14 03:29 backup.html
[root@<SECONDARY_PRIVATE_IP> ec2-user]# vi /var/www/html/backup.html
<img src="/404-error.jpg" alt="404 Error Image" style="max-width:600px;">
```

이렇게 하면 Primary와 Secondary 모두 `page.html`(정상 페이지)과 `backup.html`(장애 안내 페이지)을 동시에 가진 상태가 된다.

### 3-3. CloudFront에서 Origin Group 구성

**CloudFront** 콘솔에서 대상 Distribution의 [Origins] 탭으로 이동해 [Create origin]을 두 번 클릭해 Primary EC2, Secondary EC2를 각각 Origin으로 등록한다(Origin domain에 각 EC2의 퍼블릭 DNS를 입력).

이어서 [Origin groups] 탭 → [Create origin group]을 클릭한다.

- **Origin group members**: 앞서 등록한 Primary EC2 Origin을 `Primary`로, Secondary EC2 Origin을 `Secondary`로 지정한다.
- **Failover criteria**: 장애로 판단할 HTTP 상태 코드(예: 500, 502, 503, 504)를 체크한다.

생성한 Origin Group을 [Behaviors]의 기본 Behavior(`*`)가 바라보는 Origin으로 지정해 저장한다.

### 3-4. 검증: Primary 장애 시 Secondary로 자동 전환

Primary EC2에 SSH로 접속해 httpd를 강제로 중지시켜 장애 상황을 만든다.

```bash
sudo systemctl stop httpd
```

이후 웹 브라우저에서 CloudFront 도메인으로 `backup.html`에 접속한다.

```
https://d111111abcdef8.cloudfront.net/backup.html
```

Primary가 정상일 때는 Primary의 Instance ID가 표시되던 페이지가, Primary 장애 이후에는 Origin Group의 Failover 조건에 걸려 Secondary EC2가 응답한 `backup.html`(같은 형식의 안내 이미지와 Secondary의 Instance ID)로 바뀌어 보이는지 확인한다. 확인이 끝나면 Primary EC2에서 `sudo systemctl start httpd`로 원상 복구한다.

## 4. 실습 3: Behavior로 정적 콘텐츠와 API 요청 분기

### 4-1. API 서버(EC2)와 S3(정적 콘텐츠) 준비

API 역할을 할 EC2의 Launch Template User Data에 아래 스크립트를 등록한다.

```bash
#!/bin/bash
yum install -y httpd
systemctl enable httpd
systemctl start httpd
mkdir -p /var/www/html/api
echo "<h1>Hello from EC2 API</h1>" > /var/www/html/api/hello
echo "<h1>EC2 API Origin</h1>" > /var/www/html/index.html
```

이 EC2를 ALB 뒤에 연결해 Target Group으로 등록해 둔다(ALB 생성 절차는 [AWS-G12. HA Web Service Practice](g12-ha-web-service-practice.md) 문서를 참고한다). 정적 콘텐츠용 S3 버킷에는 별도의 `index.html`을 업로드해 둔다.

### 4-2. Distribution에 두 Origin 등록 및 Behavior 추가

**CloudFront** 콘솔에서 [Origins] 탭에 S3 버킷과 ALB를 각각 Origin으로 등록한다.

- 기본 Behavior(`*`)는 S3 Origin을 그대로 유지한다.
- [Behaviors] 탭 → [Create behavior]를 클릭해 새 규칙을 추가한다.
  - **Path pattern**: `/api/*`
  - **Origin**: 앞서 등록한 ALB Origin을 선택한다.
  - **Viewer protocol policy**: `Redirect HTTP to HTTPS`
  - **Allowed HTTP methods**: API 요청을 고려해 `GET, HEAD, OPTIONS` 이상을 허용한다.

저장 후 [Behaviors] 목록에서 `/api/*` 규칙이 기본 규칙 `*`보다 **위쪽**에 위치하는지 확인한다. 순서가 뒤바뀌면 `/api/*` 요청도 기본 규칙(`*`)에 먼저 매칭되어 S3로 가버리므로, 목록의 우선순위(Precedence)를 반드시 확인해야 한다.

### 4-3. 검증: 경로별로 서로 다른 Origin이 응답하는지 확인

```
https://d222222abcdef8.cloudfront.net/index.html    # 기본 Behavior(*)가 처리 → S3에서 정적 페이지 응답
https://d222222abcdef8.cloudfront.net/api/hello      # /api/* Behavior가 처리 → ALB→EC2로 전달되어 응답
```

`/index.html`은 S3에 업로드해 둔 정적 페이지가, `/api/hello`는 EC2의 `Hello from EC2 API` 응답이 각각 반환되는지 확인한다.

### 4-4. 보충 검증: EC2에 페이지를 직접 작성해 ALB DNS로 바로 접속

API용 EC2 콘솔에 접속해 `/var/www/html/api/main.html`을 직접 작성한다.

```bash
[ec2-user@<API_EC2_PRIVATE_IP> ~]$ sudo -s
[root@<API_EC2_PRIVATE_IP> ec2-user]# vi /var/www/html/api/main.html
```

```html
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <title>My Web Service</title>
</head>
<body>
    <h1>Hello, My Web Service!</h1>
    <p>AWS CloudFront 테스트 페이지입니다.</p>
    <hr>
    <h2>서비스 정보</h2>
    <p>정적 웹 페이지 테스트</p>
</body>
</html>
```

CloudFront를 거치지 않고 ALB DNS로 직접 접속해도 동일한 페이지가 응답하는지 확인한다.

```
http://my-api-alb-XXXXXXXXXX.ap-northeast-2.elb.amazonaws.com/api/main.html
```

`Hello, My Web Service!`, `AWS CloudFront 테스트 페이지입니다.`, `서비스 정보`, `정적 웹 페이지 테스트` 내용이 정상적으로 출력되면, `/api/*` 경로가 CloudFront를 거치든 ALB에 직접 접속하든 동일하게 EC2 API 서버로 연결되고 있음을 확인한 것이다.

## 5. 실습 4 (선택): 캐시 무효화 vs 버저닝 실습 팁

실습 2, 3에서 만든 정적 파일(`page.html`, `index.html` 등)의 내용을 수정한 뒤, CloudFront에 곧바로 재접속해보면 변경 사항이 반영되지 않고 이전 캐시가 그대로 보이는 경우가 있다. [AWS-T11. CloudFront](t11-cloudfront.md) 문서의 "캐시 파일 관리" 섹션에서 다룬 두 가지 방식을 직접 비교해본다.

**Invalidation으로 즉시 반영하기**

**CloudFront** 콘솔에서 대상 Distribution의 [Invalidations] 탭 → [Create invalidation]을 클릭하고, 무효화할 경로를 입력한다.

- `/index.html`만 무효화하려면 해당 경로만 입력한다.
- 전체를 무효화하려면 `/*`를 입력한다.

[Create invalidation]을 실행하면 수 분 내로 해당 경로의 캐시가 삭제되고, 이후 요청부터는 Origin에서 최신 파일을 다시 가져와 캐싱하는 것을 확인할 수 있다.

**버저닝 방식 맛보기**

실무에서는 `app-v1.abc123.js` → `app-v2.abc456.js`처럼 파일 내용이 바뀔 때마다 파일명에 해시를 붙여 새 파일로 배포하는 방식도 널리 쓰인다. 이 방식은 Invalidation 없이도 파일명 자체가 새로운 캐시 키가 되기 때문에 즉시 반영되지만, HTML/JS에서 참조하는 파일 경로도 함께 새 버전으로 바꿔줘야 한다는 차이가 있다. 간단히 실습해보려면 S3에 `page_v2.html`처럼 이름을 바꿔 업로드한 뒤, 이를 참조하는 링크를 수정해서 Invalidation 없이 새 콘텐츠가 바로 조회되는지 비교해보면 된다.

**정리**: Invalidation은 파일명을 그대로 둔 채 캐시만 강제로 비우는 방식이고, 버저닝은 파일명 자체를 바꿔 자연스럽게 새 캐시를 만드는 방식이다. 둘 중 무엇이 더 나은지는 상황에 따라 다르므로, 실습을 통해 두 방식의 반영 속도와 관리 방식 차이를 직접 비교해보는 것이 좋다.

# AWS SNS 실습 (S3 업로드 알림 · EventBridge · SQS FIFO 소비자)


## 1. 개요

SNS(Simple Notification Service)의 기본 개념 — Topic, 구독, Publish/Subscribe 모델, 메시지 포맷 — 은 [SNS](t08-sns.md)에 정리되어 있다. 이 문서의 실습 3·4에서는 SQS FIFO 큐를 SNS 구독자로 사용하므로, SQS의 큐/FIFO/Long Polling 개념을 먼저 정리한 [SQS](t07-decoupling-sqs.md)를 함께 읽어두는 것을 권장한다.

이 문서에서는 그 개념들을 실제로 손으로 구성해본다. S3에 파일이 업로드되면 Lambda가 이를 감지해 SNS로 알림을 발행하는 가장 기본적인 파이프라인(실습 1), EventBridge의 이벤트 패턴으로 어떤 S3 이벤트만 골라 받을지 필터링하는 방법(실습 2), EC2 위에 SQS FIFO 큐를 폴링하는 소비자를 직접 배포하는 방법(실습 3), 그리고 실제로 발행된 SNS 알림이 SQS를 통해 어떤 형태로 도착하는지 필드 단위로 확인하는 실습(실습 4)까지 이어서 다룬다.

## 2. 실습 1: S3 업로드를 Lambda로 감지해 SNS 알림 보내기

**목표**: S3 버킷의 `uploads/` 경로에 파일이 올라오면 Lambda가 이를 감지해 SNS Topic으로 알림 메시지를 발행하도록 구성한다.

**1) IAM 정책 — Lambda에 SNS 발행 권한 부여**

Lambda가 SNS Topic에 메시지를 발행하려면 `sns:Publish` 권한이 필요하다. 이 권한은 계정 내 모든 Topic이 아니라 실제로 사용할 Topic의 ARN 하나로 범위를 좁혀서 부여하는 것이 안전하다 — 최소 권한 원칙에 따라, Lambda 실행 역할이 다른 Topic까지 건드릴 수 없도록 `Resource`를 특정 ARN으로 고정한다.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "sns:Publish",
      "Resource": "arn:aws:sns:ap-northeast-2:<ACCOUNT_ID>:s3-upload-sms-email"
    }
  ]
}
```

이 정책을 Lambda 실행 역할에 연결해두면, 함수 코드 안에서 `sns.publish()`를 호출할 때 별도 자격 증명 없이 이 역할의 권한으로 동작한다.

**2) Lambda 함수 — S3 이벤트를 받아 SNS로 발행**

S3 버킷에 `ObjectCreated` 이벤트가 발생하면 아래 Lambda 함수가 트리거된다.

```python
import urllib.parse
import boto3

# AWS SNS 서비스를 사용하기 위한 클라이언트 생성
# 서울 리전(ap-northeast-2)의 SNS를 사용
sns = boto3.client("sns", region_name="ap-northeast-2")

# 메시지를 전송할 SNS Topic의 ARN
TOPIC_ARN = "arn:aws:sns:ap-northeast-2:<ACCOUNT_ID>:s3-upload-sms-email"


def lambda_handler(event, context):

    # S3 이벤트에서 첫 번째 이벤트 정보를 가져옴
    record = event["Records"][0]

    # 파일이 업로드된 S3 버킷 이름 추출
    bucket = record["s3"]["bucket"]["name"]

    # 업로드된 파일의 경로 및 파일명 추출
    # S3 이벤트의 Object Key는 URL 인코딩되어 있을 수 있으므로 디코딩
    key = urllib.parse.unquote_plus(
        record["s3"]["object"]["key"]
    )

    # 업로드된 파일이 uploads/ 경로가 아니면 SNS 알림을 보내지 않음
    if not key.startswith("uploads/"):
        return {
            "status": "ignored",
            "key": key
        }

    # SNS로 전송할 알림 메시지 작성
    message = (
        f"S3 파일 업로드 알림\n"
        f"버킷: {bucket}\n"
        f"파일: {key}"
    )

    # 지정한 SNS Topic으로 메시지 발행
    # SNS Topic에 연결된 Email, SMS 등의 구독자에게 알림 전송
    sns.publish(
        TopicArn=TOPIC_ARN,
        Subject="S3 Upload Alert",
        Message=message
    )

    # Lambda 함수 정상 처리 결과 반환
    return {
        "status": "ok",
        "bucket": bucket,
        "key": key
    }
```

함수 구조를 단계별로 보면 다음과 같다.

- `urllib.parse.unquote_plus(...)`: S3 이벤트의 Object Key는 공백이나 한글 등이 URL 인코딩된 채로 전달될 수 있다(예: 공백이 `+`로 치환). 이를 디코딩하지 않으면 실제 파일 경로와 다른 문자열을 다루게 되므로, 로그·메시지에 정확한 경로를 남기려면 반드시 디코딩을 거쳐야 한다.
- `uploads/` 접두사 검사: S3 버킷 전체에 대해 이벤트를 받되, 실제로 알림이 필요한 것은 `uploads/` 경로로 올라온 파일뿐이라면 이런 코드 레벨 필터가 필요하다. 이 조건에 걸리지 않는 업로드(예: 임시 파일, 다른 경로)는 조용히 무시하고 종료한다.
- 메시지 구성 및 발행: 버킷 이름과 파일 경로를 담은 텍스트 메시지를 만들어 `sns.publish()`로 지정한 Topic에 발행한다. Topic에 Email, SMS 등 구독자가 연결되어 있다면 이 한 번의 `publish` 호출로 모든 구독자에게 알림이 전달된다.

**정리**: S3 이벤트 → Lambda → SNS로 이어지는 가장 단순한 알림 파이프라인을 구성했다. 핵심은 IAM 정책을 Topic 단위로 최소화하는 것과, Lambda 안에서 Object Key 디코딩·경로 필터링을 빠뜨리지 않는 것이다. 경로 필터링을 Lambda 트리거 설정(S3 이벤트 알림의 Prefix 필터)에서 미리 걸 수도 있지만, 이 실습에서는 코드 레벨에서도 한 번 더 확인하는 방식을 택했다.

## 3. 실습 2: EventBridge로 S3 이벤트 필터링하기

**목표**: S3 이벤트를 Lambda가 아니라 EventBridge를 거쳐 받을 때, 이벤트 패턴(Event Pattern)으로 어떤 이벤트만 규칙에 매칭시킬지 좁혀본다.

EventBridge 규칙의 이벤트 패턴은 기본적으로 `source`(이벤트를 발생시킨 서비스), `detail-type`(이벤트 종류), `detail`(이벤트의 세부 필드) 세 부분으로 구성된다. S3 업로드 이벤트라면 `source`는 `aws.s3`, `detail-type`은 `Object Created`가 되고, `detail.bucket.name`으로 버킷을 좁히거나 `detail.object.key`에 `prefix` 조건을 걸어 특정 폴더 아래로 들어온 파일만 매칭시킬 수 있다.

```json
{
  "source": ["aws.s3"],
  "detail-type": ["Object Created"],
  "detail": {
    "bucket": {
      "name": ["버킷이름"]
    },
    "object": {
      "key": [
        { "prefix": "폴더명/" }
      ]
    }
  }
}
```

이 구조를 바탕으로 실습에서는 필터 범위를 좁은 것부터 넓은 것까지 세 단계로 나누어 테스트했다.

**1) 버킷 + 접두사 필터 (가장 좁은 범위)**

```json
{
  "source": ["aws.s3"],
  "detail-type": ["Object Created"],
  "detail": {
    "bucket": {
      "name": ["my-s3-eb-sns-email-sms-<ACCOUNT_ID>-ap-northeast-2-an"]
    },
    "object": {
      "key": [
        { "prefix": "uploads/" }
      ]
    }
  }
}
```

특정 버킷의 `uploads/` 경로로 올라온 파일만 매칭된다. 실습 1의 Lambda 코드 레벨 필터와 동일한 효과를 EventBridge 규칙 단계에서 미리 걸어두는 방식으로, 불필요한 이벤트가 아예 Lambda까지 전달되지 않아 호출 비용과 처리 로직을 줄일 수 있다.

**2) 버킷만 필터 (접두사 조건 제거)**

```json
{
  "source": ["aws.s3"],
  "detail-type": ["Object Created"],
  "detail": {
    "bucket": {
      "name": ["my-s3-eb-sns-email-sms-<ACCOUNT_ID>-ap-northeast-2-an"]
    }
  }
}
```

버킷 경로와 무관하게 해당 버킷에 생성되는 모든 객체 이벤트를 받는다. 서울 리전에서 이 패턴으로 테스트해 특정 버킷의 업로드를 폴더 구분 없이 전부 수신하는 동작을 확인했다.

**3) 필터 없음 (가장 넓은 범위)**

```json
{
  "source": ["aws.s3"],
  "detail-type": ["Object Created"]
}
```

`detail` 조건이 전혀 없어 계정 내 모든 S3 버킷의 Object Created 이벤트가 매칭된다. 도쿄 리전에서 이 패턴으로 테스트해, 버킷·경로를 특정하지 않았을 때 얼마나 넓게 이벤트가 잡히는지를 좁은 패턴과 비교했다.

세 패턴을 나란히 두고 보면, 실제 운영 환경에서는 (1)처럼 버킷과 경로까지 좁힌 패턴을 쓰는 것이 일반적이다. (2), (3)처럼 범위를 넓히는 것은 이번처럼 "필터가 정확히 어디까지 걸러주는지" 동작을 검증하거나 여러 리전에서 이벤트 발생 패턴을 비교해볼 때 유용하다.

**정리**: EventBridge 이벤트 패턴은 `source`/`detail-type`/`detail`의 조합으로 좁게도, 넓게도 구성할 수 있다. 필터를 규칙 단계에서 미리 걸어두면 뒤에 연결된 Lambda나 SNS로 불필요한 이벤트가 흘러가는 것을 막을 수 있으므로, 코드 레벨 필터링과 EventBridge 패턴 필터링을 함께 쓰는 것이 실무에서 흔한 조합이다.

## 4. 실습 3: EC2에 SQS FIFO 소비자 배포하기

**목표**: EC2 인스턴스가 부팅되면서 자동으로 SQS FIFO 큐를 폴링하는 Python 소비자를 설치·기동하도록 user-data 스크립트를 구성한다.

**1) user-data 스크립트**

```bash
#!/bin/bash
yum update -y
yum install -y python3 python3-pip
pip3 install boto3

cd /home/ec2-user

cat << 'EOF' > sqs_consumer.py
import boto3, time

sqs = boto3.client("sqs", region_name="ap-northeast-2")

QUEUE_NAME = "my-test-queue.fifo"

queue_url = sqs.get_queue_url(
    QueueName=QUEUE_NAME
)["QueueUrl"]

while True:
    resp = sqs.receive_message(
        QueueUrl=queue_url,
        MaxNumberOfMessages=1,
        WaitTimeSeconds=10,
        VisibilityTimeout=30
    )

    if "Messages" in resp:
        for msg in resp["Messages"]:
            print("받은 메시지:", msg["Body"])

            with open("/home/ec2-user/messages.log", "a") as f:
                f.write(msg["Body"] + "\n")

            sqs.delete_message(
                QueueUrl=queue_url,
                ReceiptHandle=msg["ReceiptHandle"]
            )
    else:
        print("메시지 없음...")
        time.sleep(1)
EOF

chown ec2-user:ec2-user /home/ec2-user/sqs_consumer.py

nohup python3 /home/ec2-user/sqs_consumer.py > /home/ec2-user/consumer.log 2>&1 &
```

스크립트를 단계별로 보면 다음과 같다.

- `yum update -y` / `yum install -y python3 python3-pip` / `pip3 install boto3`: 인스턴스 부팅 시점에 Python 실행 환경과 AWS SDK(boto3)를 준비한다. 이 세 줄이 없으면 이후 Python 스크립트가 동작할 수 없다.
- `cat << 'EOF' > sqs_consumer.py ... EOF`: 여기 문서(Here Document) 문법으로 Python 소스 코드 전체를 파일로 생성한다. 별도의 파일을 EC2에 미리 올려둘 필요 없이, user-data 스크립트 하나에 설치 과정과 실행할 코드를 동시에 담을 수 있다. `'EOF'`처럼 따옴표로 감싸면 내부의 `$`, 백틱 등을 쉘이 치환하지 않고 그대로 파일에 써준다.
- 폴링 루프(`sqs.receive_message`): `WaitTimeSeconds=10`은 Long Polling을 의미한다 — 큐가 비어 있어도 즉시 빈 응답을 반환하지 않고 최대 10초까지 메시지 도착을 기다렸다가 응답하므로, `WaitTimeSeconds=0`(Short Polling)으로 짧은 간격에 계속 요청을 날리는 것보다 API 호출 횟수를 크게 줄일 수 있다. `VisibilityTimeout=30`은 메시지를 받아간 뒤 30초 동안은 다른 소비자에게 같은 메시지가 보이지 않도록 숨기는 시간으로, 그 사이에 처리와 삭제가 끝나야 중복 처리를 막을 수 있다.
- 처리 후 삭제(`sqs.delete_message`): 메시지를 로그 파일에 기록한 뒤 `ReceiptHandle`로 큐에서 명시적으로 삭제한다. SQS는 메시지를 받았다고 자동으로 지워주지 않으므로, 처리가 끝난 메시지를 직접 삭제해야 같은 메시지가 다시 수신되지 않는다.
- `nohup ... &`: 인스턴스가 부팅되는 동안 실행된 이 스크립트가 세션 종료 후에도 백그라운드에서 계속 동작하도록 한다.

**2) 실제 기동 확인**

user-data로 자동 기동되는지와 별개로, 콘솔 세션에서 직접 스크립트를 실행하고 프로세스가 떠 있는지 확인했다.

```text
[ec2-user@ip-172-31-24-44 ~]# python3 /home/ec2-user/sqs_consumer.py

[ec2-user@ip-172-31-24-44 ~]# ps -ef | grep sqs_consumer
root        3268    3249  0 06:42 pts/4    00:00:00 python3 /home/ec2-user/sqs_consumer.py
ec2-user    3270    2724  0 06:43 pts/3    00:00:00 grep --color=auto sqs_consumer
```

`ps -ef | grep sqs_consumer` 결과에 `python3 /home/ec2-user/sqs_consumer.py` 프로세스가 살아있는 것이 보이면, 소비자가 정상적으로 백그라운드에서 폴링을 이어가고 있다는 뜻이다.

**정리**: user-data 스크립트로 패키지 설치·코드 생성·백그라운드 실행까지 한 번에 자동화할 수 있다. 폴링 루프에서는 Long Polling(`WaitTimeSeconds`)으로 호출 횟수를 줄이고, `VisibilityTimeout` 동안 처리를 끝낸 뒤 명시적으로 `delete_message`를 호출하는 삭제 패턴을 지키는 것이 SQS 소비자 구현의 기본이다.

## 5. 실습 4: SNS 알림 메시지 수신 확인하기

**목표**: SNS Topic에 발행한 메시지가 SQS FIFO 큐를 통해 실제로 어떤 형태로 도착하는지 확인하고, 필드별 의미를 정리한다.

**1) 테스트 메시지 발행**

아래와 같은 간단한 JSON을 원본 메시지로 SNS Topic에 발행했다.

```json
{"orderId":"1001", "status":"new"}
```

**2) 소비자가 실제로 수신한 로그**

실습 3에서 띄워둔 SQS FIFO 소비자가 이 메시지를 수신한 결과는 다음과 같다.

```text
[root@ip-172-31-33-40 ec2-user]# python3 /home/ec2-user/sqs_consumer.py
메시지 없음...
메시지 없음...
받은 메시지: {
  "Type" : "Notification",
  "MessageId" : "05ebceec-d15b-5059-98ad-af12051ad22e",
  "SequenceNumber" : "10000000000000003000",
  "TopicArn" : "arn:aws:sns:ap-northeast-1:<ACCOUNT_ID>:my-test-topic.fifo",
  "Subject" : "order-alarm",
  "Message" : "{\"orderId\":\"1001\" , \"status\":\"NEW\"}",
  "Timestamp" : "2026-02-05T17:23:13.176Z",
  "UnsubscribeURL" : "https://sns.ap-northeast-1.amazonaws.com/?Action=Unsubscribe&SubscriptionArn=arn:aws:sns:ap-northeast-1:<ACCOUNT_ID>:my-test-topic.fifo:c5dd8602-0a19-40c5-86dc-d67f42f5693c"
}
```

처음 두 번의 "메시지 없음..." 출력은 실습 3에서 구성한 Long Polling 루프가 메시지가 도착할 때까지 `WaitTimeSeconds=10` 만큼 대기와 재시도를 반복하고 있다는 뜻이며, 이후 실제 메시지가 도착하자 SNS가 감싼 형태 그대로 로그에 출력됐다.

**3) 필드별 의미**

- `Type`: `"Notification"` — 이 메시지가 SNS에서 발행된 "알림(Notification)"이라는 것을 나타낸다.
- `MessageId`: SNS가 이 메시지에 부여한 고유 ID.
- `SequenceNumber`: FIFO Topic에서 메시지의 순서를 추적하기 위한 번호.
- `TopicArn`: 메시지가 발행된 SNS Topic의 ARN(고유 식별자).
- `Subject`: 발행 시 지정한 "제목" 값 — 여기서는 `sns.publish()` 호출 시 넘긴 `Subject`와 동일한 역할이다.
- `Message`: 실제로 전달하려던 본문. 원본 JSON(`{"orderId":"1001" , "status":"NEW"}`)이 SNS 봉투 안에 이스케이프 처리된 문자열로 한 번 더 감싸져 들어있는 것을 볼 수 있다.
- `Timestamp`: 메시지가 SNS에서 발행된 시각(UTC 기준).
- `UnsubscribeURL`: 이 링크를 호출하면 해당 구독을 취소할 수 있다.

이렇게 SQS가 SNS 메시지를 원본 그대로가 아니라 `Type`, `MessageId`, `TopicArn` 등의 메타데이터로 한 번 감싼 "봉투(envelope)" 형태로 전달하는 것이 SNS→SQS 구독의 기본 동작이다. [SNS](t08-sns.md)에서 다룬 SNS 메시지 래핑 개념이 여기서 실제 로그로 확인된 셈이다. 만약 이 봉투 없이 원본 `Message` 내용만 그대로 받고 싶다면 구독 설정에서 **Raw Message Delivery**를 활성화하면 되는데, 이 경우 소비자 코드에서 `msg["Body"]`를 더 이상 SNS 포맷으로 파싱할 필요 없이 원본 JSON으로 바로 다룰 수 있게 된다.

**정리**: SNS로 발행한 메시지는 구독자(SQS)에게 그대로 전달되는 것이 아니라 `Type`/`MessageId`/`TopicArn` 등을 포함한 봉투 형태로 감싸져 도착한다는 점이 이번 실습의 핵심이다. 소비자 코드를 작성할 때는 `msg["Body"]`가 SNS 봉투인지, 원본 메시지인지(Raw Message Delivery 여부)를 먼저 확인하고 그에 맞게 파싱해야 한다는 점을 기억해둘 만하다.

# AWS SNS FIFO 실습 (SQS FIFO 소비자 배포)


## 1. 개요

SNS FIFO와 SQS FIFO의 순서 보장·중복 제거 개념(Message Group ID, Deduplication ID 등)은 [SNS](t08-sns.md)에, SQS의 큐/FIFO/Long Polling 기본 개념은 [SQS](t07-decoupling-sqs.md)에 정리되어 있다. S3 업로드를 Lambda로 감지해 SNS로 알림을 보내는 기본 파이프라인 실습은 [S3 업로드 알림 실습](g10-sns-practice.md)에서 별도로 다룬다.

이 문서에서는 SNS FIFO Topic에 발행된 메시지가 SQS FIFO 큐를 통해 실제로 어떻게 도착하는지 직접 확인한다. EC2 위에 SQS FIFO 큐를 폴링하는 Python 소비자를 배포하고(실습 1), 실제로 발행된 SNS FIFO 알림이 소비자에게 어떤 형태로 도착하는지 필드 단위로 확인한다(실습 2).

## 2. 실습 1: EC2에 SQS FIFO 소비자 배포하기

**목표**: EC2 인스턴스가 부팅되면서 자동으로 SQS FIFO 큐를 폴링하는 Python 소비자를 설치·기동하도록 user-data 스크립트를 구성한다.

**1) SQS FIFO 큐 생성**

**SQS** 콘솔로 이동해 오른쪽 상단의 [Create queue]를 클릭한다.

- **Type**: `FIFO`를 선택한다. FIFO를 선택하면 이름 입력란에 `.fifo` 접미사가 자동으로 붙는다.
- **Name**: `my-test-queue`를 입력하면 `my-test-queue.fifo`로 저장된다(뒤에서 다룰 소비자 스크립트의 `QUEUE_NAME`과 반드시 일치해야 한다).
- **Configuration**의 `Visibility timeout`은 기본 `30`초로 두고, 나머지 옵션도 기본값으로 둔 채 [Create queue]를 클릭한다.
- 큐가 생성되면 [Access policy] 탭에서 뒤에서 만들 SNS FIFO Topic이 이 큐로 메시지를 보낼 수 있도록 권한을 허용해야 하는데, SNS Topic 쪽에서 [Create subscription]으로 이 큐를 구독자로 등록하면 콘솔이 필요한 권한을 자동으로 큐의 Access policy에 추가해준다.

**2) EC2 인스턴스에 user-data 스크립트 등록**

**EC2** 콘솔에서 [Launch instance]를 클릭하고, 이름·AMI(Amazon Linux 2023 권장)·인스턴스 유형(`t2.micro`)·키 페어·보안 그룹을 [EC2 설정](g03-ec2-setup.md)과 동일한 방식으로 설정한다. 화면 아래쪽 [Advanced details]를 펼치면 맨 아래에 **User data** 입력란이 있는데, 여기에 아래 user-data 스크립트 전체를 붙여넣는다.

이 인스턴스가 SQS 큐에 접근하려면 `sqs:ReceiveMessage`·`sqs:DeleteMessage`·`sqs:GetQueueUrl` 권한이 필요하므로, [IAM] → [Roles]에서 이 권한을 담은 역할을 만들어 인스턴스 생성 화면의 **IAM instance profile** 항목에 연결해야 한다(리소스를 특정 큐 ARN으로 좁혀 최소 권한 원칙을 지키는 것은 [S3 업로드 알림 실습](g10-sns-practice.md)의 Lambda IAM 정책과 같은 맥락이다).

**3) user-data 스크립트 내용**

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

**4) 실제 기동 확인**

인스턴스가 시작되면 **EC2** 콘솔에서 해당 인스턴스를 선택하고 [Connect] 버튼으로 EC2 Instance Connect 또는 SSH 세션에 접속한다. user-data로 자동 기동되는지와 별개로, 콘솔 세션에서 직접 스크립트를 실행하고 프로세스가 떠 있는지 확인했다.

```text
[ec2-user@ip-172-31-24-44 ~]# python3 /home/ec2-user/sqs_consumer.py

[ec2-user@ip-172-31-24-44 ~]# ps -ef | grep sqs_consumer
root        3268    3249  0 06:42 pts/4    00:00:00 python3 /home/ec2-user/sqs_consumer.py
ec2-user    3270    2724  0 06:43 pts/3    00:00:00 grep --color=auto sqs_consumer
```

`ps -ef | grep sqs_consumer` 결과에 `python3 /home/ec2-user/sqs_consumer.py` 프로세스가 살아있는 것이 보이면, 소비자가 정상적으로 백그라운드에서 폴링을 이어가고 있다는 뜻이다.

**정리**: user-data 스크립트로 패키지 설치·코드 생성·백그라운드 실행까지 한 번에 자동화할 수 있다. 폴링 루프에서는 Long Polling(`WaitTimeSeconds`)으로 호출 횟수를 줄이고, `VisibilityTimeout` 동안 처리를 끝낸 뒤 명시적으로 `delete_message`를 호출하는 삭제 패턴을 지키는 것이 SQS 소비자 구현의 기본이다.

## 3. 실습 2: SNS 알림 메시지 수신 확인하기

**목표**: SNS FIFO Topic에 발행한 메시지가 SQS FIFO 큐를 통해 실제로 어떤 형태로 도착하는지 확인하고, 필드별 의미를 정리한다.

**1) SNS FIFO Topic 생성과 SQS FIFO 구독 연결**

**SNS** 콘솔의 [Topics] → [Create topic]에서 **Type**을 `FIFO`로 선택한다. **Name**에 `my-test-topic`을 입력하면 `my-test-topic.fifo`로 생성된다. `Content-based message deduplication`을 활성화하면 퍼블리셔 쪽에서 별도의 Deduplication ID를 지정하는 코드 없이도 본문 해시 기준으로 중복이 제거된다.

Topic이 생성되면 [Create subscription]을 클릭하고, **Protocol**을 `Amazon SQS`로, **Endpoint**를 앞서 만든 `my-test-queue.fifo`의 ARN으로 지정해 구독을 등록한다. SNS FIFO는 SQS FIFO·SQS Standard 큐만 구독자로 허용하므로(본문에서 다룬 연동 제한), 이메일이나 Lambda를 구독자로 선택하는 항목 자체가 나타나지 않는다.

**2) 테스트 메시지 발행**

Topic 상세 페이지에서 [Publish message] 버튼을 클릭하면 메시지를 직접 발행해볼 수 있다. **Subject**에 `order-alarm`을, **Message body**에 아래 JSON을 입력하고, FIFO Topic이므로 추가로 나타나는 **Message group ID** 입력란에는 `order`처럼 임의의 그룹 이름을 채운 뒤 [Publish message]를 누른다.

```json
{"orderId":"1001", "status":"new"}
```

**3) 소비자가 실제로 수신한 로그**

실습 1에서 띄워둔 SQS FIFO 소비자가 이 메시지를 수신한 결과는 다음과 같다.

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

처음 두 번의 "메시지 없음..." 출력은 실습 1에서 구성한 Long Polling 루프가 메시지가 도착할 때까지 `WaitTimeSeconds=10` 만큼 대기와 재시도를 반복하고 있다는 뜻이며, 이후 실제 메시지가 도착하자 SNS가 감싼 형태 그대로 로그에 출력됐다. Topic 이름이 `my-test-topic.fifo`로 `.fifo` 접미사를 갖고 있다는 점에서 이 Topic이 SNS FIFO로 생성되었음을 확인할 수 있다.

**4) 필드별 의미**

- `Type`: `"Notification"` — 이 메시지가 SNS에서 발행된 "알림(Notification)"이라는 것을 나타낸다.
- `MessageId`: SNS가 이 메시지에 부여한 고유 ID.
- `SequenceNumber`: FIFO Topic에서 메시지의 순서를 추적하기 위한 번호로, [SNS](t08-sns.md)에서 다룬 Message Sequence Number가 실제로 이 필드에 담겨 온다.
- `TopicArn`: 메시지가 발행된 SNS Topic의 ARN(고유 식별자).
- `Subject`: 발행 시 지정한 "제목" 값 — 여기서는 `sns.publish()` 호출 시 넘긴 `Subject`와 동일한 역할이다.
- `Message`: 실제로 전달하려던 본문. 원본 JSON(`{"orderId":"1001" , "status":"NEW"}`)이 SNS 봉투 안에 이스케이프 처리된 문자열로 한 번 더 감싸져 들어있는 것을 볼 수 있다.
- `Timestamp`: 메시지가 SNS에서 발행된 시각(UTC 기준).
- `UnsubscribeURL`: 이 링크를 호출하면 해당 구독을 취소할 수 있다.

이렇게 SQS가 SNS 메시지를 원본 그대로가 아니라 `Type`, `MessageId`, `TopicArn` 등의 메타데이터로 한 번 감싼 "봉투(envelope)" 형태로 전달하는 것이 SNS→SQS 구독의 기본 동작이다. [SNS](t08-sns.md)에서 다룬 SNS 메시지 래핑 개념이 여기서 실제 로그로 확인된 셈이다. 만약 이 봉투 없이 원본 `Message` 내용만 그대로 받고 싶다면 구독 설정에서 **Raw Message Delivery**를 활성화하면 되는데, 이 경우 소비자 코드에서 `msg["Body"]`를 더 이상 SNS 포맷으로 파싱할 필요 없이 원본 JSON으로 바로 다룰 수 있게 된다.

**정리**: SNS FIFO Topic으로 발행한 메시지는 구독자(SQS FIFO)에게 그대로 전달되는 것이 아니라 `Type`/`MessageId`/`SequenceNumber`/`TopicArn` 등을 포함한 봉투 형태로 감싸져 도착하며, `SequenceNumber` 필드를 통해 FIFO 특유의 순서 추적이 실제로 동작함을 확인할 수 있었다. 소비자 코드를 작성할 때는 `msg["Body"]`가 SNS 봉투인지, 원본 메시지인지(Raw Message Delivery 여부)를 먼저 확인하고 그에 맞게 파싱해야 한다는 점을 기억해둘 만하다.


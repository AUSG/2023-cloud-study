# 7.1 스트리밍 데이터 수집을 위한 Kinesis Stream 사용

> _애플리케이션의 스트리밍 데이터를 수집하기 위해 스트림 생성 후 레코드를 삽입한다_

#### Kinesis Stream을 생성하고 `ACTIVE` 상태로 변경된 것을 확인

- 샤드(shard) : 데이터베이스나 웹 검색 엔진 데이터의 수평 분할
  - 읽기에 대해 초당 최대 5개의 트랜잭션을 지원하고, 최대 초당 2MB의 읽기 속도를 지원한다.
  - 쓰기에 대해 초당 최대 1,000개의 레코드를 지원하고, 파티션 키를 포함하여 초당 1MB의 쓰기 속도를 지원한다.
  - 더 많은 데이터를 처리해야 할 경우, 언제든지 스트림을 다시 샤딩할 수 있다.

```bash
aws kinesis create-stream --stream-name AWSCookbook701 --shard-count 1

aws kinesis describe-stream-summary --stream-name AWSCookbook701
```

```json
>> output
{
    "StreamDescriptionSummary": {
        "StreamName": "AWSCookbook701",
        "StreamARN": "arn:aws:kinesis:ap-northeast-2:944371408142:stream/AWSCookbook701",
        "StreamStatus": "ACTIVE",
        "StreamModeDetails": {
            "StreamMode": "PROVISIONED"
        },
        "RetentionPeriodHours": 24,
        "StreamCreationTimestamp": "2023-03-26T14:16:21+09:00",
        "EnhancedMonitoring": [
            {
                "ShardLevelMetrics": []
            }
        ],
        "EncryptionType": "NONE",
        "OpenShardCount": 1,
        "ConsumerCount": 0
    }
}
```

---

🥕 **유효성 검사**

- 샤드 반복자 : 특정 샤드 내의 데이터 레코드 시퀀스의 특정 위치를 나타내며, 이 반복자를 이용하여 스트림에서 데이터 레코드를 읽을 수 있다.

```bash
# kinesis stream에 레코드 삽입
aws kinesis put-record --stream-name AWSCookbook701 \
	--partition-key 111 \
	--cli-binary-format raw-in-base64-out \
	--data={\"Data\":\"1\"}
```

```json
>> output
{
    "ShardId": "shardId-000000000000",
    "SequenceNumber": "49639232751543449786256520802190680815174790430526013442"
}
```

```bash
# 샤드 반복자를 가져와 getrecords 명령 실행
SHARD_ITERATOR=$(aws kinesis get-shard-iterator \
	--shard-id shardId-000000000000 \
	--shard-iterator-type TRIM_HORIZON \
	--stream-name AWSCookbook701 \
	--query 'ShardIterator' \
	--output text)

aws kinesis get-records --shard-iterator $SHARD_ITERATOR \
  --query 'Records[0].Data' --output text | base64 --decode
```

```json
>> output
{"Data":"1"}
```

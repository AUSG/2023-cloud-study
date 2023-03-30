# 1. 스트리밍 데이터 수집을 위한 Kinesis 스트림 사용

<br>
<br>

### 실습

---

스트리밍 데이터를 수집하기 위해 스트림을 생성하고 스트림에 레코드를 삽입한다.

1. Kinesis 스트림 생성

   ```bash
   aws kinesis create-stream --stream-name AWSCookbook701 --shard-count 1
   ```

   → 샤드는 읽기에 대한 초당 최대 5개의 트랜잭션을 지원할 수 있으며, 최대 초당 2MB의 읽기 속도를 지원

   → 쓰기의 경우 초당 최대 1,000개의 레코드를 지원할 수 있으며 파티션 키를 포함해 초당 1MB의 쓰기 속도를 지원

   → 더 많은 데이터를 처리해야하는 경우 언제든지 스트림을 다시 샤딩할 수 있다.

   ![image](https://user-images.githubusercontent.com/49095587/227703550-ccd1e401-9984-4616-a1d7-097023637e8b.png)

2. 스트림이 ACTIVE 상태로 변경된 것을 확인

   ```bash
   aws kinesis describe-stream-summary --stream-name AWSCookbook701
   ```

3. Kinesis 스트림에 레코드 삽입

   ```bash
   aws kinesis put-record --stream-name AWSCookbook701 --partition-key 111 --cli-binary-format raw-in-base64-out --data={\"Data\":\"1\"}
   ```

   `—data` 플래그 : 스트림에 삽입할 데이터 지정, JSON 형식의 데이터를 사용

   → 위 명령어는 **`AWSCookbook701`**이라는 이름의 Amazon Kinesis 스트림에 **`111`**이라는 파티션 키를 가진 데이터 레코드를 삽입하고, 데이터 필드에는 **`1`**이라는 값을 가진 JSON 데이터를 사용

   ![image](https://user-images.githubusercontent.com/49095587/227703573-6da01c0e-cf24-4114-9501-5b0cd76c5eef.png)

4. Kinesis 스트림에서 Shard Iterator

   ```bash
   SHARD_ITERATOR=$(aws kinesis get-shard-iterator --stream-name AWSCookbook701 --shard-id shardId-000000000000 --shard-iterator-type TRIM_HORIZON --query 'ShardIterator')
   ```

5. Kinesis 스트림에서 레코드를 가져온다. 샤드 반복자를 가져와 getrecords 명령을 실행한다.
   `bash
 SHARD_ITERATOR=$(aws kinesis get-shard-iterator --shard-id shardId-000000000000 --shard-iterator-type TRIM_HORIZON --stream-name AWSCookbook701 --query 'SHARDIterator' --output text) aws kinesis get-records --shard-iterator $SHARD_ITERATOR --query 'Records[0].Data' --output text | base64 --decode
 `
   <br>
   <br>

### 정리

---

`Kinesis Data Stream` : 대규모 실시간 데이터 스트리밍 서비스

생산자 : 스트림에 기록을 삽입하는 소스

소비자 : 스트림에서 레코드를 가져오는 개체

예시 : 실시간 금융 데이터, IoT 및 써 데이터, 웹 및 모바일 애플리케이션의 최종 사용자 클릭 스트림

사용 예시 : 스트림에서 직접 람다함수를 호출할 수 있다. 추가적으로 Kinesis Data Analytics를 사용할 수 있다.

`Shard` : 대규모 데이터베이스 시스템에서 데이터를 분산 저장하는 방법, 데이터를 작은 조각으로 분할하고, 각 조각을 별도의 서버에 저장

`Shard Iterator` : Kinesis 스트림에서 레코드를 가져오기 위한 포인터

`파티션 키` : NoSQL 데이터베이스에서 데이터를 분산 저장하고 검색하기 위해 사용되는 중요한 개념

<br>

**Kinesis Data Stream을 생성할 때, 온디맨드와 프로비저닝 차이**

---

온디맨드 모드는 스트림에 데이터를 쓰거나 읽는 데 필요한 샤드의 수가 예측하기 어려울 때, 프로비저닝 모드는 데이터 처리량이 일정하고 예측 가능할 때 사용하는 것이 좋다.

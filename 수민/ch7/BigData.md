# Big Data
# 7.1 스트리밍 데이터 수집을  위한 Kinesis 스트림 사용
``` bash
aws Kinesis Data streams에서 producer와 consumer 개념이 존재. producer는 스트림 data를 input를 넣는 개념이고 스트림에 로그 데이터를 보내는 웹서버가 producer가 될 수 있고 consumer는 그걸 처리하는 kinesis data stream 어플리케이션이 될 수 있음. 데이터 레코드는 저장되는 데이터의 단위.
샤드는 스트림에서 고유하게 식별되는 데이터 레코드 시퀀스이며 스트림은 하나 이상의 샤드로 구성되며 각 샤드는 고정된 용량 단위를 제공. 각 샤드는 읽기에 대해 초당 최대 5개의 트랜잭션, 초당 최대 2MB의 데이터 읽기 속도, 초당 최대 1,000개의 레코드까지, 초당 최대 1MB의 총 데이터 쓰기 속도 (파티션 키 포함) 를 지원 => 참고(https://docs.aws.amazon.com/ko_kr/streams/latest/dev/key-concepts.html)

파티션 키는 스트림 내에서 샤드별로 그룹화하기 위해서 사용되고 시퀀스 넘버는 샤드 내에 파티션의 키마다 고유.

aws kinesis 콘솔에서 생산자로는 Amazon Kinesis 에이전트, AWS SDK,Amazon Kinesis Producer Library(KPL)가 있고
consumer로는 Amazon Kinesis Data Analytics,Amazon Kinesis Data Firehose, Amazon Kinesis Client Library(KCL)가 있음.

먼저 aws kinesis stream을 하나 생성 -> 레코드 삽입
aws kinesis put-record --stream-name aws701 --partition-key 111 --cli-binary-format raw-in-base64-out --data={\"Data\":\"1\"}

그 다음 스트림의 데이터 뷰어에 가서 내가 생성한 shardId로 trim Horizon으로 레코드를 검색하면 내가 삽입했던 레코드가 들어간걸 확인할 수 있음.

Kinesis 데이터를 처리하는 람다 함수를 만들어서 자동으로 트리거하도록 구성할수도 있음!
``` 

# 7.2 Amazon Kinesis Data Firehose를 사용한 Amazon S3로 데이터 스트리밍
``` bash
이번 챕터는 kinesis 스트림 consumer를 aws Kinesis Data Firehose로 이용해서 스트림된 데이터를 s3로 저장하는 챕터임.

보통 스트림 데이터는 실시간으로 사용하기 위해서 스트림으로 데이터 프로듀싱하고 컨슈밍하지만 데이터를 저장하거나 한번에 처리하는 경우도 있음. 그럴때는 s3가 아니어도 다른 엔드포인ㄴ트에 데이터를 전달해서 저장할 수 있음. 그리고 만약 스트림 대상이 여러 개인 경우 방금 만든 전송 스트림을 여러 개 만들어서 단일 producer 스트림에 연결해서 데이터를 전달할 수도 있음. 그리고 또한 데이터 변환도 가능 한대. 스트리밍 데이터를 변환하고 싶을 때는 람다 훔스를 호출할 수 있도록 활성화하면 됨.
전체 과정은 블로그 참고!

```
[블로그 글](https://suminn0.tistory.com/148)

# 7.3 AWS Glue 크롤러를 사용한 메타데이터 검색 자동화
``` bash
aws glue 크롤러를 이용하면 s3에 저장되어 있는 csv같이 스토리지에 있는 다양한 데이터에 대한 메타데이터를 검색할 수 있음. aws glue crawlers는 데이터 소스(여기선 s3)에 연결하고 소스의 객체를 스캔하고, 데이터 스키마와 연결된 테이블로 glue 데이터 카탈로그 데이터베이스를 채움.

과정은 간단한데 먼저 aws glue 콘솔로 들어가서 database를 하나 생성하고 data catalog tables에서 add table using crawler를 눌러서 이름을 입력하고 data source를 s3로 하나 추가해주면 됨.
s3 말고도 JDBC나 DynamoDB 테이블도 스캔할 수 있으며 JDBC의 경우에는 네트워크 연결이 필요해서 따로 연결을 정의해야함. 역할 같은 경우엔 하나 새로 생성해서 사용하면 되고 만들어진 크롤러를 s3에 csv 파일을 넣어놓고 crawler를 run해보면 데이터베이스 테이블에 하나 추가 되는 것을 확인할 수 있음.
```

# 7.4 Amazon Athena를 사용한 S3 내의 파일 쿼리
``` bash
Athena 서비스는 표준 sql을 이용해서 s3에 있는 데이터를 query를 실행해서 쉽게 분석할 수 있는 대화형 쿼리 서비스. 정말 간단하게 결과 s3를 설정해주고 편집기에서 쿼리로 데이터베이스와 테이블을 만든다음 조회하면 끝!
```

# 7.5 AWS Glue DataBrew를 사용한 데이터 변
``` bash
프로젝트를 생성해서 데이터의 변환을 가능하게 하는 서비스 유형. 데이터 처리 결과를 미리 확인할 수 있고 데이터 워크플로우를 자동화할 수 있음. 시각적 인터페이스를 제공해줌.
```
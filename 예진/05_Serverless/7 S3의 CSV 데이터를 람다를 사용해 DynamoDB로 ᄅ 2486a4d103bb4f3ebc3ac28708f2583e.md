# 7. S3의 CSV 데이터를 람다를 사용해 DynamoDB로 로드

<br>

### 실습

---

CSV 파일을 S3에 업로드하고 S3에서 DynamoDB로 데이터를 로드해보자. → S3에서 DynamoDB로 데이터를 업로드 하는 일을 람다가 처리

S3 데이터를 DynamoDB로 로드하는 람다 함수를 생성 → `S3:PutObject`이벤트가 람다 함수를 트리거 하는 S3 알림을 구성

1. DynamoDB 테이블 생성
2. S3 버킷 이름에 사용할 고유 접미사를 설정
3. S3 버킷 생성
4. S3 및 DynamoDB 사용을 허용하는 람다 함수에 대한 역할을 생성
5. `AmazonS3ReadOnlyAccess`, `AmazonDynamoDBFullAccess`, `AWSLambdaBasicExecutionRole`에 대한 IAM 관리형 정책을 IAM 역할에 연결
6. 함수 코드를 압축 → 람다 함수 생성
7. S3 서비스에 람다 함수에 대한 호출 권한을 부여 aws lambda add-permission
8. 파일을 참고해 파일이 업로드될 때 람다 함수를 자동으로 트리거하기 위한 이벤트 생성
9. 람다 함수를 트리거하는 S3 버킷 알림 설정을 구성 `aws s3api put-bucket-notification-configuration`
10. 파일을 S3에 업로드

<br>

### 정리

---

해당 실습에서는 notification.json 파일을 생성할 때와 람다 함수, S3 버킷 그리고 객체를 버킷에 넣을 때 람다 함수를 트리거하는 키 패턴을 지정했다.

이벤트 기반 아키텍처를 사용하면 필요할 때만 함수를 실행하기 때문에 앺ㄹ리케이션 실행과 관련된 비용과 복잡성을 최소화할 수 있다.

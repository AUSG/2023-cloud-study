# 0. Serverless 들어가며

<br>

### 서버리스 서비스

---

- 컴퓨팅 : AWS Lambda, Amazon Fargate
- 애플리케이션 : Amazon EventBridge, Amazon SNS, Amazon SQS, Amazon API Gateway
- 데이터스토어 : Amazon S3, DynamoDB, Aurora Serverless

<br>

**AWS 서버리스의 주요 이점**

---

- 비용 절감
- 확장성 : 필요한 만큼 확장 / 축소
- 적은 관리 : 배포할 서버나 관리할 시스템이 없다
- 유연성 : 다양한 프로그래밍 언어

서버리스의 람다를 사용하기 위해선, `AWSLambdaBasicExecutionRole`

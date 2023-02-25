## 1.9 S3 버킷에 대한 퍼블릭 액세스 차단

### 문제 설명

S3 버킷의 퍼블릭 액세스를 차단해야 한다.

### 준비 단계

- cookbook 저장소 참고

### 해결 방법

Amazon S3 퍼블릭 액세스 차단 기능을 버킷에 적용하고, Access Analyzer로 상태를 확인

1. 접근 유효성 검사에 사용할 Access Analyzer 생성 (accessDeniedException 😭)

```bash
ANALYZER_ARN=$(aws accessanalyzer create-analyzer --analyzer-name awscookbook109 --type ACCOUNT --output text --query arn)
```

2. Access Analyzer로 S3 버킷 스캔

```bash
aws accessanalyzer start-resource-scan \
--analyzer-arn $ANALYZER_ARN \
--resource-arn arn:aws:s3:::awscookbook109-$RANDOM_STRING
```

3. Access Analyzer 스캔 결과를 확인한다.

```bash
aws accessanalyzer get-analyzed-resource \
--analyzer-arn $ANALYZER_ARN \
--resource-arn arn:aws:s3:::awscookbook109-$RANDOM_STRING
```

4. 버킷에 대한 퍼블릭 액세스 차단을 설정한다.

```bash
aws s3api put-public-access-block \
--bucket awscookbook109-$RANDOM_STRING \
--public-access-block-configuration \
"BlockPublicAcls=true,IgnorePubllicAcls=true,BlockPublicPolicy=true,RestrctPublic Buckets=true"
```

### 유효성 검사

1. Accesss Analyzer S3 버킷 스캔 시작

```bash
aws accessanalyzer start-resource-scan \
--analyzer-arn $ANALYZER_ARN \
--resource-arn arn:aws:s3:::awscookbook109-$RANDOM_STRING
```

2. Accesss Analyzer 스캔 결과 확인

```bash
aws accessanalyzer get-analyzed-resource \
--analyzer-arn $ANALYZER_ARN \
--resource-arn arn:aws:s3:::awscookbook109-$RANDOM_STRING
```

### 참고

aws 계정의 데이터 보안을 위해 항상 데이터에 올바른 보안을 적용하고 있는지 확인해야 한다. (S3 버킷의 객체를 퍼블릭으로 설정하면, 인터넷의 모든 사용자가 객체에 액세스 가능)

- 버킷에 대해 BlockPublicAccess를 활성화하는 습관을 가지자.

버킷을 비공개로 유지하더라도, HTTP 및 HTTPS를 통해 인터넷에 S3 콘텐츠를 제공할 수 있다.
CloudFront와 같은 CDN(콘텐츠 전송 네트워크)를 사용하고 정적 웹사이트 호스팅을 통해 S3의 객체를 보다 안전하고 효율적인 방법으로 콘텐츠 서빙이 가능하다.

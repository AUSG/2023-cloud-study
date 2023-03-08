## 3.9 DataSync를 활용한 EFS와 S3 간의 데이터 복제

### 🎯 목표

- S3에서 EFS로 파일을 복제해야 한다.

### 🤔 해결 방법

- S3에서 EFS를 대상으로 AWS DataSync를 구성한다.
- DataSync 작업을 생성하고 복제 작업을 시작한다.

### 👉 작업 방법

1. IAM role 생성

```bash
S3_ROLE_ARN=$(aws iam create-role --role-name AWSCookbookS3LocationRole \
   --assume-role-policy-document file://assume-role-policy.json \
   --output text --query Role.Arn)
```

2. 위 IAM role에 AmazonS3ReadOnlyAccess IAM 관리형 정책 연결

```bash
aws iam attatch-role-policy --role-name AWSCookbookS3LocationRole \
    --policy-arn arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess
```

3. DataSync S3 위치 생성

```bash
S3_LOCATION_ARN=$(aws datasync create-location-s3 \
    --s3-bucket-arn $BUCKET_ARN \
    --s3-config BucketAccessRoleArn=$S3_ROLE_ARN \
    --output text --query LocationArn)
```

4. `assume-role-policy.json` 기반으로 IAM role 생성

### 🤗 참고

- AWS DataSync를 사용해 다양한 AWS 서비스의 온디맨드 또는 지속적/자동화한 파일 동기화 작업을 할 수 있다.

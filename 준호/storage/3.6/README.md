## 3.6 AWS KMS를 사용한 Amazon S3 버킷의 객체 암호화

### 🎯 목표

S3 객체를 저장할 때 비용 효율적인 방식으로 암호화하고자 KMS 고객 관리형 키(CMK)를 사용해야 한다.

### 🤔 해결 방법

- KMS CMK를 생성하고
- 이를 참고하는 S3 버킷 키를 사용하도록 S3 버킷 구성
- S3:PutObject 작업에 KMS를 사용해야 하는 S3 버킷 정책 구성

### 👉 작업 방법

1. S3 버킷에 사용할 KMS 키 생성, 키 ID를 환경 변수에 저장

```bash
KEY_ID=$(aws kms create-key \
    --tags TagKey=Name,TagValue=AWSCookbook306Key \
    --description "AWSCookbook S3 CMK" \
    --query KeyMetadata.KeyId \
    --output text)
```

2. 키를 참고할 별칭 생성

```bash
aws kms create-alias \
    --alias-name alias/awscookbook306 \
    --target-key-id $KEY_ID
```

3. KMS 키 ID를 S3 버킷 키로 사용하도록 S3 버킷 구성

```bash
aws s3api put-bucket-encryption \
    --bucket awscookbook306-$RANDOM_STRING \
    --server-side-encryption-configuration '{
        "Rules": [
            {
                "ApplyServerSideEncrytionByDefault": {
                    "SSEAlgorithm": "aws:kms",
                    "KMSMasterKeyID": "${KEY_ID}"
                },
                "BucketKeyEnabled": true
            }
        ]
    }
```

4. 버킷에 모든 객체를 암호화하는 버킷 정책 파일 생성(bucket-policy-template.json)
5. 4 참조하여 생성

```bash
sed -e "s|BUCKET_NAME|awscookbook306-${RANDOM_STRING}|g" \
    bucket-policy-template.json > bucket-policy.json
```

6. 버킷 정책을 적용해 모든 업로드에 암호화 적용

```bash
aws s3api put-bucket-policy --bucket awscookbook306-$RANDOM_STRING \
    --policy file://bucket-policy.json
```

7. 유효성 검사

```bash
aws s3 cp ./book_cover s3://awscookbook-$RANDOM_STRING \
    --sse aws:kms --sse-kms-key-id $KEY_ID
```

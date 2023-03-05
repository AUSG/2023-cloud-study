## 3.3 복구 시점 목표 달성을 위한 S3 버킷 복제 구성

### 🎯 목표

데이터 보안 정책에 따라 동일한 리전 내에서 객체를 복제해 15분의 복구 시점 목표를 달성해야 한다.

### 🤔 해결 방법

1. 소스 및 대상 S3 버킷을 생성하고 버전 관리를 활성화한다.
2. IAM 역할을 생성해 S3가 원본에서 대상 버킷으로 객체를 복사하도록 허용하는 IAM 정책을 연결한다.
3. IAM 역할을 참고하는 S3 복제 정책을 생성하고, 해당 정책을 소스 버킷에 적용한다.

### 👉 작업 방법

1. 대상으로 사용할 S3 버킷 생성

```bash
aws s3api create-bucket --bucket awscookbook303-dst-$RANDOM_STRING
```

2. 대상 S3 버킷에 버전 관리를 활성화

```bash
   aws s3api put-bucket-versioning \
    --bucket awscookbook303-dst-$RANDOM_STRING
    --versioning-configuration Status=Enabled
```

3. `s3-assume-role-policy.json` 생성
4. `s3-assume-role-policy.json` 이용하여 IAM 역할 생성

```bash
ROLE_ARN=$(aws iam create-role --role-name AWSCookbook303S3Role \
    --assume-role-policy-document file://s3-assume-role-policy.json \
    --output text --query Role.ROLE_ARN)
```

5. S3 복제 시 소스 및 대상 버킷에 액세스할 수 있도록 `s3-perms-policy-template.json` 파일 생성
6. s3-perms-policy.json 생성

```bash
sed -d "s/DSTBUCKET/awscookbook303-dst-${RANDOM_STRING}/g" \
    -e "s|SRCBUCKET|awscookbook303-src-${RANDOM_STRING}|g" \
    s3-perms-policy-template.json > s3-perms-policy.json
```

7. 생성한 역할에 정책 연결

```bash
aws iam put-role-policy \
    --role-name AWSCookbook303S3Role \
    --policy-document file://s3-perms-policy.json \
    --policy-name S3ReplicationPolicy
```

8. `s3-replictaion-template.json`을 생성해 복제 시간을 15분으로 구성
9. 8을 기반으로 한 `s3-replication.json` 생성

```bash
sed -e "|ROLEARN|${ROLE_ARN}|g" \
    -e "s|DSTBUCKET|awscookbook303-dst-${RANDOM_STRING}|g" \
    s3-replication-template.json > s3-replication.json
```

10. 소스 S3 버킷 정책에 대한 복제 정책을 구성한다.

```bash
aws s3api put-bucket-replictaion \
    --replication-configuration file://s3-replication.json \
    --bucket awscookbook303-src-${RANDOM_STRING}
```

11. 유효성 검사

```bash
aws s3api get-bucket-replication \
    --bucket awscookbook303-src-${RANDOM_STRING}
```

### 🤗 참고

- SRR(Same-Region Replictaion)과 CRR(Cross-Region Replictaion)이라는 두 가지 유형의 복제를 제공
- 같은 리전 복제(SRR)은 프로덕션 환경과 테스트 환경 간의 데이터 복제 등에 이용
- 크로스 리전 복제(CRR)은 지역적으로 더 가까운 데이터셋에 액세스해 대기 시간 감소 등에 이용

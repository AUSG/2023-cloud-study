## 1.10 CloudFront를 사용해 S3에서 안전하게 웹 콘텐츠 제공

### 문제 설명

CloudFront를 구성해 S3의 프라이빗 웹 콘텐츠를 제공하고자 함.

### 해결 방법

- CloudFront 배포를 생성하고 오리진을 S3 버킷으로 설정한다.
- CloudFront 에서만 버킷에 액세스할 수 있도록 원본 액세스ID(OAI)를 구성한다.

### 준비 사항

- 정적 웹 콘텐츠를 가진 S3 버킷

### 준비 단계

S3 버켓을 생성하고, 파일을 업로드하며, 퍼블릭 액세스 차단까지 설정(cookbook 저장소 참고)

#### Create a S3 bucket:

```
aws s3api create-bucket --bucket awscookbook110-$RANDOM_STRING
```

#### Copy the previously created file to the bucket:

```
aws s3 cp index.html s3://awscookbook110-$RANDOM_STRING/
```

#### Set the public access block for your bucket:

```
aws s3api put-public-access-block \
     --bucket awscookbook110-$RANDOM_STRING \
     --public-access-block-configuration "BlockPublicAcls=true,IgnorePublicAcls=true,BlockPublicPolicy=true,RestrictPublicBuckets=true"
```

### 작업 방법

1. S3 버킷 정책이 참고할 CloudFront OAI를 생성 (CF로만 접근할 수 있도록)

```bash
OAI=$(aws cloudfront create-cloud-front-origin-access-identity \
--cloud-front-origin-access-identity-config \
CallerReference="awscookbook",Comment="AWSCookbook OAI" \
--query CloudFrontOriginAccessIdentity.Id --output text)
```

2. distribution-template.json을 CF OAI 및 S3 버킷 이름으로 변경

```bash
sed -e "s/CLOUDFRONT_OAI/${OAI}/g" \
-e "s|S3_BUCKET_NAME|awscookbook110-$RANDOM_STRING|g" \
distribution-template.json > distribution.json
```

3. 배포 구성을 참고해 CF 배포를 생성

```bash
DISTRIBUTION_ID=$(aws cloudfront create-distribution \
--distribution-config file://distribution.json \
--query Distribution.Id --output text)
```

4. 배포

```bash
aws cloudfront get-distribution --id $DISTRIBUTION_ID \
--output text --query Distribution.Status
```

5. bucket-policy-template.json을 참고하여 CF의 요청만 허용하도록 S3 버킷 정책 구성

6. 내 CF OAI와 S3 버킷 이름으로 변경

```bash
sed -e "s/CLOUDFRONT_OAI/${OAI}/g" \
-e "s|S3_BUCKET_NAME|awscookbook110-$RANDOM_STRING|g" \
bucket-policy-template.json > bucket-policy.json
```

7. S3 버킷에 버킷 정책 적용

```bash
aws s3api put-bucket-policy --bucket awscookbook110-$RANDOM_STRING \
--policy file://bucket-policy.json
```

8. 생성한 배포의 DOMAIN_NAME 확인

```bash
DOMAIN_NAME=$(aws cloudfront get-distribution --id $DISTRIBUTION_ID \
--query Distribution.DomainName --output text)
```

### 참고

S3 버킷을 비공개로 유지하면서, CF 배포만 버킷의 객체에 접근이 가능하도록 정의할 수 있다.

- CDN은 디도스(분산 서비스 거부 공격)으로부터 보호
- 최종 사용자에게 가장 짧은 지연 시간으로 콘텐츠를 전달하는 기능 제공
- 가격적인 측면으로도 저렴함
- 기본적으로 HTTPS 인증서 함께 제공

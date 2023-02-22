# 1.10 CloudFront를 사용해 S3에서 안전하게 웹 콘텐츠 제공

> _CloudFront로 S3의 프라이빗 웹 콘텐츠 제공_

#### ① S3 버킷 정책이 참고할 CloudFront OAI 생성

```bash
OAI=$(aws cloudfront create-cloud-front-origin-access-identity \
  --cloud-front-origin-access-identity-config \
  CallerReference="awscookbook",Comment="AWSCookbook OAI" \
  --query CloudFrontOriginAccessIdentity.Id --output text)
```

#### ② sed 명령어로 `distribution-template.json` 파일의 값을 CloudFront OAI 및 S3 버킷 이름으로 변경

```bash
sed -e "s/CLOUDFRONT_OAI/${OAI}/g" \
  -e "s|S3_BUCKET_NAME|awscookbook110-$RANDOM_STRING|g" \
  distribution-template.json > distribution.json
```

#### ③ CloudFront 배포 생성

```bash
DISTRIBUTION_ID=$(aws cloudfront create-distribution \
  --distribution-config file://distribution.json \
  --query Distribution.Id --output text)
```

#### ④ 배포하는 데에 일정 시간 소요되므로 ‘Deployed’ 상태가 될 때까지 대기

```bash
aws cloudfront get-distribution ==id $DISTRIBUTION_ID \
  --output text --query Distribution.Status
```

#### ⑤ CloudFront 요청만 허용하도록 S3 버킷 정책 구성 ; `bucket-policy-template.json`

#### ⑥ sed 명령으로 위 파일의 값을 CloudFront OAI 및 S3 버킷 이름으로 교체

```bash
sed -e "s/CLOUDFRONT_OAI/${OAI}/g" \
  -e "s|S3_BUCKET_NAME|awscookbook110-$RANDOM_STRING|g" \
  bucket-policy-template.json > bucket-policy.json
```

#### ⑦ 정적 웹 콘텐츠가 있는 S3 버킷에 버킷 정책 적용

```bash
aws s3api put-bucket-policy --bucket awscookbook110-$RANDOM_STRING \
  --policy file://bucket-policy.json
```

#### ⑧ 생성한 배포의 `DOMAIN_NAME` 확인

```bash
DOMAIN_NAME=$(aws cloudfront get-distribution --id $DISTRIBUTION_ID \
  --query Distribution.DomainName --output text)
```

---

🥕 **유효성 검사** : 직접 액세스 vs CloudFront 요청으로 액세스

```bash
# HTTPS로 S3 버킷에 직접 액세스 ⇒ 403 AccessDenied Error
curl https://awscookbook110-$RANDOM_STRING.s3.$AWS_REGION.amazonaws.com/index.html

# CloudFront를 통해 콘텐츠 제공 ⇒ index.html 제공
curl $DOMAIN_NAME

```

🥕 **참고**

⍢ **CDN** : DDoS 공격으로부터 보호하고 최종 사용자에게 가장 짧은 지연 시간으로 콘텐츠 전달

- S3에서 직접 요청 처리하는 것보다 비용 절감

⍢ **CloudFront** : 트래픽 보호를 위해 해당 배포의 기본 호스트네임에 대한 HTTPS 인증서를 함께 제공

- 고유한 사용자 지정 도메인 이름 설정, ACM에서 사용자 지정 인증서 연결 후 HTTP → HTTPS 리디렉션하거나 HTTPS만 강제할 수 있음

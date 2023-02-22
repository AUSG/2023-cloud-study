# 1.9 S3 버킷에 대한 퍼블릭 액세스 차단

> _S3 버킷의 퍼블릭 액세스 차단 후, Access Analyzer로 상태 확인_

#### ① 접근 유효성 검사에 사용할 Access Analyzer 생성

```bash
ANALYZER_ARN=$(aws accessanalyzer create-analyzer \
  --analyzer-name awscookbook109 \
  --type ACCOUNT \
  --output text --query arn)
```

#### ② Access Analyzer로 S3 버킷 스캔

```bash
aws accessanalyzer start-resource-scan \
  --analyzer-arn $ANALYZER_ARN \
  --resource-arn arn:aws:s3:::awscookbook109-$RANDOM_STRING
```

#### ③ 스캔 결과 확인

```bash
aws accessanalyzer get-analyzed-resource \
  --analyzer-arn $ANALYZER_ARN \
  --resource-arn arn:aws:s3:::awscookbook109-$RANDOM_STRING
```

- output 중 `isPublic` 값 확인 ⇒ true: 퍼블릭 액세스 허용 상태

#### ④ 버킷에 대한 퍼블릭 액세스 차단 설정

```bash
aws s3api put-public-access-block \
  --bucket awscookbook109-$RANDOM_STRING \
  --public-access-block-configuration \
"BlockPublicAcls=true,IgnorePublicAcls=true,BlockPublicPolicy=true,RestrictPublicBuckets=true"
```

---

**🥕 유효성 검사** : ③번 과정 수행 후 `isPublic:false` 확인

**🥕 참고**

⍢ S3 버킷의 객체 퍼블릭 공개는 흔한 보안 오류 중 하나

- 계정 수준에서 퍼블릭 차단 설정

  ```bash
  aws s3control put-public-access-block \
    --public-access-block-configuration \

  BlockPublicAcls=true,IgnorePublicAcls=true,BlockPublicPolicy=true,RestrictPublicBuckets=true \
    --account-id $AWS_ACCOUNT_ID
  ```

⍢ 버킷을 비공개로 유지해도 HTTP 및 HTTPS를 통해 인터넷에 S3 콘텐츠를 제공할 수 있음

- CloudFront와 같은 CDN 사용 (1.10에서 등장)
- 정적 웹사이트 호스팅을 통해 보다 안전하고 비용 효율적으로 콘텐츠 제공

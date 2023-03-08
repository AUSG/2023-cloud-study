# 3.1 S3 수명 주기 정책을 사용한 스토리지 비용 절감

> _자주 사용하지 않는 객체에는 S3 IA로 전환하는 S3 수명 주기 정책을 생성하여 적용한다_

① `lifecycle-rule.json` 참고하여 S3 버킷에 적용할 수명 주기 정책 생성

```json
{
  "Rules": [
    {
      "ID": "Move all objects to Standard Infrequently Access",
      "Prefix": "",
      "Status": "Enabled",
      "Transitions": [
        {
          "Days": 30,
          "StorageClass": "STANDARD_IA"
        }
      ]
    }
  ]
}
```

② 수명 주기 정책을 버킷에 적용

```bash
aws s3api put-bucket-lifecycle-configuration \
	--bucket awscookbook301-$RANDOM_STRING \
	--lifecycle-configuration file://lifecycle-rule.json
```

🥕 **참고**

⍢ **S3 Infrequent Access** : 자주 액세스하지 않는 개체의 데이터 비용을 절감할 수 있는 S3 스토리지 클래스

⍢ 데이터 액세스 패턴을 예측할 수 없지만 비용, 성능, 복원력을 최적화하려면 **S3 Intelligent-Tiering**을 활용!

# 3.2 S3 Intelligent-Tiering 아카이브 정책을 사용한 S3 객체 자동 아카이브

> _90일 이상 된 객체에 대한 액세스 패턴을 기반으로 객체를 S3 Glacier 아카이브로 자동화하는 정책 적용_

① `tiering.json` 생성

```json
{
  "Id": "awscookbook302",
  "Status": "Enabled",
  "Tierings": [
    {
      "Days": 90,
      "AccessTier": "ARCHIVE_ACCESS"
    }
  ]
}
```

② Intelligent-Tiering 구성 적용

```bash
aws s3api put-bucket-intelligent-tiering-configuration \
	--bucket awscookbook302-$RANDOM_STRING \
	--id awscookbook302 \
	--intelligent-tiering-configuration "$(cat tiering.json)"
```

🥕 **참고**

⍢ **S3 Intelligent-Tiering** : 자주 액세스하지 않는 객체를 S3 Glacier Archive로 전환하는 자동 메커니즘 제공

- 객체가 아카이브로 전환하는 시간을 정할 수 있음 (90-730일)

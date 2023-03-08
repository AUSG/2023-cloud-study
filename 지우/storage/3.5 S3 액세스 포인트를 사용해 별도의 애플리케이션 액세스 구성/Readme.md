# 3.5 S3 액세스 포인트를 사용해 별도의 애플리케이션 액세스 구성

> _S3 액세스 포인트에 상이한 정책을 부여함으로써 액세스 권한을 다르게 부여한다_

① VPC에서 두 애플리케이션에 대한 액세스 포인트 각각 생성

```bash
aws s3control create-access-point --name cookbook305-app-1 \
	--account-id $AWS_ACCOUNT_ID \
	--bucket $BUCKET_NAME --vpc-configuration VpcId=$VPC_ID

aws s3control create-access-point --name cookbook305-app-2 \
	--account-id $AWS_ACCOUNT_ID \
	--bucket $BUCKET_NAME --vpc-configuration VpcId=$VPC_ID
```

② sed 명령으로 `app-policy-template.json` 값을 첫 번째 애플리케이션의 `EC2_INSTANCE_PROFILE`, `AWS_REGION`, `AWS_ACCOUNT_ID`, `ACCESS_POINT_NAME` 및 `ACTIONS` 값으로 치환

- 첫 번째 애플리케이션은 `s3:GetObject`, `s3:PutObject` → R/W 권한을 부여

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "EC2_INSTANCE_PROFILE"
      },
      "Action": ["ACTIONS"],
      "Resource": "arn:aws:s3:AWS_REGION:AWS_ACCOUNT_ID:accesspoint/ACCESS_POINT_NAME/object/*"
    }
  ]
}
```

```bash
sed -e "s/AWS_REGION/${AWS_REGION}/g" \
	-e "s|EC2_INSTANCE_PROFILE|${INSTANCE_ROLE_1}|g" \
	-e "s|AWS_ACCOUNT_ID|${AWS_ACCOUNT_ID}|g" \
	-e "s|ACCESS_POINT_NAME|${ACCESS_POINT_NAME}|g" \
	-e "s|ACTIONS|\"s3:GetObject\",\"s3:PutObject\"|g" \
	app-policy-template.json > app-1-policy.json
```

③ 이 정책을 첫 애플리케이션의 액세스 포인트에 배치

```bash
aws s3control put-access-point-policy --account-id $AWS_ACCOUNT_ID  \
	--name cookbook305-app-1 --policy file://app-1-policy.json
```

④ 두 번째 애플리케이션도 ②, ③번 과정 수행

- 두 번째 애플리케이션은 `s3:GetObject` → R 권한만 부여

```bash
sed -e "s/AWS_REGION/${AWS_REGION}/g" \
	-e "s|EC2_INSTANCE_PROFILE|${INSTANCE_ROLE_1}|g" \
	-e "s|AWS_ACCOUNT_ID|${AWS_ACCOUNT_ID}|g" \
	-e "s|ACCESS_POINT_NAME|${ACCESS_POINT_NAME}|g" \
	-e "s|ACTIONS|\"s3:GetObject\"|g" \
	app-policy-template.json > app-2-policy.json

aws s3control put-access-point-policy --account-id $AWS_ACCOUNT_ID  \
	--name cookbook305-app-1 --policy file://app-2-policy.json
```

⑤ 버킷 정책을 수정해 액세스 포인트에 제어를 위임 (공식문서 참고)

---

**🥕 참고**

⍢ S3 액세스 포인트를 왜 사용하는가?

- 특정 보안 주체에 부여할 수 있는 액세스 권한을 세분화할 수 있다는 점에서 S3 버킷 정책 관리보다 용이

⍢ S3 액세스 포인트에 대한 별도 비용은 없다

- 액세스 포인트의 퍼블릭 액세스 차단도 구성할 수 있다

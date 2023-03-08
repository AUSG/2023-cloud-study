# 3.6 AWS KMS를 사용한 Amazon S3 버킷의 객체 암호화

> _S3 객체를 비용 효율적인 방식으로 암호하기 위해 KMS CMK를 사용한다_

- KMS CMK를 참고하는 S3 버킷 키를 사용하도록 S3 버킷 구성

<br>

① S3 버킷용 KMS 키 생성 및 키 ID를 환경변수에 저장

```bash
KEY_ID=$(aws kms create-key \
	--tags TagKey=Name,TagValue=AWSCookbook306Key \
	--description "AWSCookbook S3 CMK" \
	--query KeyMetadata.KeyId \
	--output text)
```

② 키를 참고할 별칭 생성

```bash
aws kms create-alias \
	--alias-name alias/awscookbook306 \
	--target-key-id $KEY_ID
```

③ KMS 키 ID를 S3 버킷 키로 사용하도록 S3 버킷 구성

```bash
aws s3api put-bucket-encryption \
	--bucket awscookbook306-$RANDOM_STRING \
	--server-side-encryption-configuration '{
	"Rules": [
		{
			"ApplyServerSideEncryptionByDefault": {
				"SSEAlgorithm": "aws:kms",
				"KMSMasterKeyID": "${KEY_ID}"
			},
			"BucketKeyEnabled": true
		}
	]
}'
```

④ 버킷에 모든 객체를 암호화하는 버킷 정책 템플릿 파일을 생성하고, sed 명령으로 `BUCKET_NAME` 치환

```json
{
  "Version": "2012-10-17",
  "Id": "PutObjectPolicy",
  "Statement": [
    {
      "Sid": "DenyUnEncryptedObjectUploads",
      "Effect": "Deny",
      "Principal": "*",
      "Action": "s3:PutObject",
      "Resource": "arn:aws:s3:::BUCKET_NAME/*",
      "Condition": {
        "StringNotEquals": {
          "s3:x-amz-server-side-encryption": "aws:kms"
        }
      }
    }
  ]
}
```

```bash
sed -e "s|BUCKET_NAME|awscookbook306-${RANDOM_STRING}|g" \
	bucket-policy-template.json > bucket-policy.json
```

⑤ 버킷 정책을 적용하여 모든 업로드에 암호화 적용

```bash
aws s3api put-bucket-policy --bucket awscookbook306-$RANDOM_STRING \
	--policy file://bucket-policy.json
```

---

**🥕 유효성 검사** : 암호화 없이 객체 업로드하면 `KMS.NotFoundException` 오류를 반환

```bash
# 암호화를 사용하여 객체 업로드
aws s3 cp ./argo.png s3://awscookbook306-$RANDOM_STRING \
	--sse aws:kms --sse-kms-key-id $KEY_ID

# 암호화 없이 객체 업로드 → 오류 반환
aws s3 cp ./argo.png s3://awscookbook306-$RANDOM_STRING
```

<br>

🥕 **참고**

⍢ **AWS CMK**

- S3가 AWS 계정에서 생성하고 자동으로 관리하는 AWS 관리형 CMK
  - 고객 관리형 CMK, AWS 관리형 CMK
- AWS 계정 및 리전에 따라 고유한 값을 지님
- S3는 사용자 대신에 CMK를 사용할 수 있는 권한을 갖게 됨

<br>

**🥕 Demo**

**⍢ KMS 키 생성**

- Key Tag: `Name:S3Key`
- alias: `s3key`

<img src="https://user-images.githubusercontent.com/70079416/223654783-2ebff96a-569b-4934-a10a-682bbfbba9dc.png">

<br>
**⍢ Validation Check**

암호 없이 객체 업로드 시도했을 때 뜨는 오류

```bash

aws s3 cp ./docker.png s3://src-bucket-ziwoo

>> upload failed: ./docker.png to s3://src-bucket-ziwoo/docker.png An error occurred (KMS.NotFoundException) when calling the PutObject operation: Invalid keyId ${KEY_ID}
```

<br>
암호와 함께 객체 업로드 후, 콘솔에서 업로드된 것 확인

```bash
aws s3 cp ./docker.png s3://src-bucket-ziwoo --sse aws:kms --sse-kms-key-id $KEY_ID

>> upload: ./docker.png to s3://src-bucket-ziwoo/docker.png
```

<img src="https://user-images.githubusercontent.com/70079416/223654570-b2c1f677-557c-42da-887d-37b606600687.png">

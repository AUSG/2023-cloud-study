# 5.7 S3의 CSV 데이터를 람다를 사용해 DynamoDB에 로드

> _S3 데이터를 DynamoDB에 로드하는 람다 함수를 생성하고 S3:PutObject 이벤트가 람다 함수를 트리거하는 S3 알림을 구성한다_

① DynamoDB 테이블과 S3 버킷 생성

- S3 버킷 이름에 사용할 고유 접미사 설정 ⇒ 버킷 이름 중복으로 인한 오류 방지

```bash
# DynamoDB 테이블 생성
aws dynamodb create-table \
	--table-name 'AWSCookbook507' \
	--attribute-definitions 'AttributeName=UserID,AttributeType=S' \
	--key-schema 'AttributeName=UserID,KeyType=HASH' \
	--sse-specification 'Enabled=true,SSEType=KMS' \
	--provisioned-throughput \
	'ReadCapacityUnits=5,WriteCapacityUnits=5'

# S3 고유 접미사 설정
RANDOM_STRING=$(aws secretsmanager get-random-password \
	--exclued-punctuation --exclude-uppercase \
	--password-length 6 --require-each-included-type \
	--output text \
	--query RandomPassword)

# S3 버킷 생성
aws s3api create-bucket --bucket awscookbook507-$RANDOM_STRING
```

② S3와 DynamoDB 사용을 허용하는 람다 함수에 대한 역할 생성하고, `AmazonS3ReadOnlyAccess`, `AmazonDynamoDBFullAccess`, `AWSLambdaBasicExecutionRole` IAM 관리형 정책을 역할에 연결

```bash
# 역할 생성
aws iam create-role --role-name AWSCookbook507Lambda \
	--assume-role-policy-document file://assume-role-policy.json

# 정책 연결
aws iam attach-role-policy --role-name AWSCookbook507Lambda \
	--policy-arn arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess

aws iam attach-role-policy --role-name AWSCookbook507Lambda \
	--policy-arn arn:aws:iam::aws:policy/AmazonDynamoDBFullAccess

aws iam attach-role-policy --role-name AWSCookbook507Lambda \
	--policy-arn arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole
```

③ 함수 코드를 압축하여 람다 함수 생성

```bash
zip lambda_function.zip lambda_function.py

LAMBDA_ARN=$(aws lambda create-function \
	--function-name AWSCookbook507Lambda \
	--runtime python3.8 \
	--package-type "Zip" \
	--zip-file fileb://lambda_function.zip \
	--handler lambda_function.lambda_handler --publish \
	--role \
	arn:aws:iam::$AWS_ACCOUNT_ID:role/AWSCookbook507Lambda \
	--output text --query FunctionArn)
```

④ S3 서비스에 람다 함수에 대한 호출 권한 부여

```bash
aws lambda add-permission --function-name $LAMBDA_ARN \ --action
lambda:InvokeFunction --statement-id s3invoke \ --principal s3.amazonaws.com
```

⑤ 파일이 업로드될 때마다 람다 함수를 자동으로 트리거하는 이벤트 생성하고, sed 명령으로 값 치환

```json
{
  "LambdaFunctionConfigurations": [
    {
      "Id": "awscookbook507event",
      "LambdaFunctionArn": "LAMBDA_ARN",
      "Events": ["s3:ObjectCreated:*"],
      "Filter": {
        "Key": {
          "FilterRules": [
            {
              "Name": "prefix",
              "Value": "sample_data.csv"
            }
          ]
        }
      }
    }
  ]
}
```

```bash
sed -e "s/LAMBDA_ARN/${LAMBDA_ARN}/g" \
	notification-template.json > notification.json
```

⑥ 람다 함수를 트리거하는 S3 버킷 알림 설정을 구성

```bash
aws s3api put-bucket-notification-configuration \
	--bucket awscookbook507-$RANDOM_STRING \
	--notification-configuration file://notification.json
```

⑦ S3에 파일 업로드

```bash
aws s3 cp ./smaple_data.csv s3://awscookbook507-$RANDOM_STRING
```

---

**🥕 유효성 검사** : CLI 혹은 콘솔에서 DynamoDB 테이블 스캔하여 S3 객체 로드된 것 확인

```bash
aws dynamodb scan --table-name AWSCookbook507
```

🥕 **참고**

⍢ Lambda + DynamoDB ⇒ 운영 오버헤드를 최소화하고 데이터베이스의 지속성을 크게 확장할 수 있는 애플리케이션 구축에 유용하다.

- 람다 함수는 15분의 제한 시간을 갖는다!

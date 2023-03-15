# 4.5 RDS 데이터베이스의 암호 교체 자동화

> _암호 교체 시간 간격과 암호 교체를 수행하는 람다 함수를 구성하여 자동 비밀번호 교체를 구현한다_

① AWS Secrets Manager로 RDS 요구사항을 충족하는 암호 생성하고 RDS 데이터베이스의 관리자 암호 변경

- Secrets Manager의 `GetRandomPassword API` 메서드를 사용하면 임의의 문자열을 생성할 수 있다.

```bash
# 암호 생성
RDS_ADMIN_PASSWD=$(aws secretsmanager get-random-password \
	--exclude-punctuation \
	--password-length 41 --require-each-included-type \
	--output text --query RandomPassword)

# 데이터베이스 관리자 암호 변경
aws rds modify-db-instance \
	--db-instance-identifier $RDS_ID \
	--master-user-password $RDS_ADMIN_PASSWD \
	--apply-immediately
```

② sed 명령으로 `rdscreds-template.json` 값을 치환하여 `rdscreds.json` 생성

```json
{
  "username": "admin",
  "password": "PASSWORD",
  "engine": "mysql",
  "host": "HOST",
  "port": 3306,
  "dbname": "DBNAME",
  "dbInstanceIdentifier": "DBIDENTIFIER"
}
```

```bash
sed -e "s/AWS_ACCOUNT_ID/${AWS_ACCOUNT_ID}/g" \
	-e "s|PASSWORD|${RDS_ADMIN_PASSWD}|g" \
	-e "s|HOST|${RdsEndpoint}|g" \
	-e "s|DBNAME|${DbName}|g" \
	-e "s|DBIDENTIFIER|${RdsDatabaseId}|g" \
	rdscreds-template.json > redscreds.json
```

③ [AWS 샘플 깃헙](https://github.com/aws-samples/aws-secrets-manager-rotation-lambdas/blob/master/SecretsManagerRDSMySQLRotationSingleUser/lambda_function.py)에서 Rotation 람다 함수 코드를 받아 압축

```bash
wget https://raw.githubusercontent.com/aws/aws-samples/aws-secrets-manager-rotation-lambdas/master/SecretsManagerRDSMySQLRotationSingleUser/lambda_function.py
zip lambda_function.zip lambda_funcion.py
>> adding: lambda_function.py
```

④ 람다 함수가 사용할 새 보안 그룹 생성하고 RDS 보안 그룹에 TCP 3306 포트에 대한 인바운드 규칙 추가

```bash
LAMBDA_SG_ID=$(aws ec2 create-security-group \
	--group-name AWSCookbook405LambdaSG \
	--description "Lambda Security Group" --vpc-id $VPC_ID \
	--output text --query GroupId)

aws ec2 authorize-security-group-ingress \
	--protocol tcp --port 3306 \
	--source-group $LAMBDA_SG_ID \
	--group-id $RDS_SG
```

⑤ `assume-role-policy.json` 파일로 IAM 역할을 생성하고 `AWSLambdaVPCAccess`과 `SecretsManagerReadWrite` IAM 관리형 정책 연결

- prod 환경에선 람다 함수 권한을 제한하기 위해 `SecretsManagerReadWrite` 연결 지양

```bash
aws iam create-role --role-name AWSCookbook405Lambda \
	--assume-role-policy-document file://assume-role-policy.json

aws iam attach-role-policy --role-name AWSCookbook405Lambda \
	--policy-arn arn:aws:iam::aws:policy/service-role/AWSLambdaVPCAccessExecutionRole

aws iam attach-role-policy --role-name AWSCookbook405Lambda \
	--policy-arn arn:aws:iam::aws:policy/SecretsManagerReadWrite
```

⑥ 암호 교체 기능을 수행하는 람다 함수 생성하고 Secrets Manager가 람다 함수를 호출할 수 있도록 권한 추가

```bash
# 람다 함수 생성
LAMBDA_ROTATE_ARN=$(aws lambda create-function \
	--function-name AWSCookbook405Lambda \
	--runtime python3.8 \
	--package-type "Zip" \
	--zip-file file://lambda_function.zip
	--handler lambda_function.lambda_handler --publish \
	--environment Variables={SECRETS_MANAGER_ENDPOINT=https://secretsmanager.$AWS_REGION.amazonaws.com} \
	--layers $PyMysqlLambdaLayerArn \
	--role \
	arn:aws:iam::$AWS_ACCOUNT_ID:role/AWSCookbook405Lambda \
	--output text --query FunctionArn \
	--vpc-config SubnetIds=${ISOLATED_SUBNETS},SecurityGroupIds=$LAMBDA_SG_ID)

# 람다 함수 활성화될 때까지 대기
aws lambda get-function --function-name $LAMBDA_ROTATE_ARN \
	--output text --query Configuration.State

# Secrets Manager가 람다 함수를 호출할 수 있도록 권한 추가
aws lambda add-permission --function-name $LAMBDA_ROTATE_ARN \
	--action lambda:InvokeFunction --statement-id secretsmanager \
	--principal secretsmanager.amazonaws.com
```

_(optional) 자동 비밀번호 교체를 위해 암호에 사용할 고유 접미사 설정 가능_

```bash
AWSCookbook405SecretName=AWSCookbook405Secret-$(aws secretsmanager \
get-random-password \
--exclude-punctuation \
--password-length 6 --require-each-included-type \
--output text --query RandomPassword)
```

⑦ Secrets Manager에서 암호를 저장할 시크릿 생성

```bash
aws secretsmanager create-secret --name $AWSCookbook405SecretName \
	--description "My Database Secret created with the CLI" \
	--secret-string file://rdscred.json
```

⑧ 30일마다 자동 교체를 설정하고, 보안 암호를 교체하도록 람다 함수 지정

- rotate-secret 명령은 암호 교체를 바로 trigger

```bash
aws secretsmanager rotate-secret \
	--secret-id $AWSCookbook405SecretName \
	--rotation-rules AutomaticallyAfterDays=30 \
	--rotation-lambda-arn $LAMBDA_ROTATION_ARN
```

⑨ 암호 로테이션을 한 번 더 수행하고, ⑧과 유사하지만 `Version ID`값이 다른 것 확인

```bash
aws secretsmanager rotate-secret --secret-id $AWSCookbook405SecretName
```

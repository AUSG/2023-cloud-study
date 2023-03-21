# 5.4 람다 함수에서 EFS 파일 시스템 사용

> _기존 서버가 사용하는 네트워크 파일 시스템에 람다가 접근할 수 있어야 한다_

① 람다 함수가 사용할 새 보안 그룹 생성

```bash
LAMBDA_SG=$(aws ec2 create-security-group \
	--group-name AWSCookbook504LambdaSG \
	--description "Lambda Security Group" --vpc-id $VPC_ID \
	--output text --query GroupId)
```

② EFS 파일 시스템의 보안 그룹에 람다 함수 보안 그룹의 TCP 2049 포트에 대한 인바운드 규칙 생성

```bash
aws ec2 authorize-security-group-ingress \
	--protocol tcp --port 2049 \
	--source-group $LAMBDA_SG \
	--group-id $EFS_SG
```

③ `assume-rule-policy.json`으로 IAM 역할을 생성하고 `AWSLambdaVPCAccessExecutionRole` IAM 관리형 정책을 IAM 역할에 연결

```bash
aws iam create-role --role-name AWSCookbook504Role \
	--assume-role-policy-document file://assume-role-policy.json

aws iam attach-role-policy --role-name AWSCookbook504Role \
	--policy-arn arn:aws:iam::aws:policy/service-role/AWSLambdaVPCAccessExecutionRole
```

④ `lambda_function.py` 압축하고, EFS 파일 시스템의 `ACCESS_POINT_ARN`으로 람다 함수 생성

```bash
zip lambda_function.zip lambda_function.py

LAMBDA_ARN=$(aws lambda create-function \
	--function-name AWSCookbook504Lambda \
	--runtime python3.8 \
	--package-type "Zip" \
	--zip-file fileb://lambda_function.zip \
	--handler lambda_function.lambda_handler --publish \
	--role \
	arn:aws:iam::$AWS_ACCOUNT_ID:role/AWSCookbook504Role \
	--file-system-configs Arn="$ACCESS_POINT_ARN",LocalMountPath="/mnt/efs"
	--output text --query FunctionArn \
	--vpc-config SubnetIds=${ISOLATED_SUBNETS},SecurityGroupIds=${LAMBDA_SG}
```

⑤ 람다 함수가 활성화된 것 확인

```bash
aws lambda get-function --function-name $LAMBDA_ARN \
	--output text --query Configuration.State
```

---

**🥕 유효성 검사** : 람다 함수를 실행하여 파일 확인

```bash
aws lambda invoke \
	--function-name $LAMBDA_ARN \
	response.json && cat response.json
```

**🥕 참고**

⍢ 서버리스 컴퓨팅 + 서버리스 파일 시스템, 데이터베이스 ⇒ 컴퓨팅 및 스토리지 운영 오버헤드를 크게 줄임

⍢ Amazon EFS + AWS Lambda 조합으로 구현 가능한 것 :

- 애플리케이션을 위한 영구 스토리지
- 유지 보수
- 이벤트 기반 알림
- 이벤트 기반 파일 처리

# 4.3 RDS proxy를 사용한 람다와 RDS 연결

> _서버리스 함수가 관계형 DB에 연결할 때 connection 수를 최소화하고 성능을 향상시키기 위해 connection pooling을 고려 → RDS proxy로 DB 연결을 관리한다_

① `assume-role-policy.json`으로 RDS proxy에 대한 IAM 역할 생성

```bash
aws iam create-role --assume-role-policy-document \
	file://assume-role-policy.json --role-name AWSCookbook403RDSProxy
```

② RDS proxy에서 사용할 보안 그룹을 생성하고 RDS proxy 생성

```bash
# 보안 그룹 생성
RDS_PROXY_SG_ID=$(aws ec2 create-security-group \
	--group-name AWSCookbook403RDSProxySG \
	--description "Lambda Security Group" --vpc-id $VPC_ID \
	--output text --query GroupId)

# RDS proxy 생성
RDS_PROXY_ENDPOINT_ARN=$(aws rds create-db-proxy \
	--db-proxy-name $DB_NAME \
	--engine-family MYSQL \
	--auth '{
				"AuthScheme": "SECRETS",
				"SecretArn": "'"$RDS_SECRET_ARN"'",
				"IAMAuth": "REQUIRED"
			}' \
	--role-arn arn:aws:iam::$AWS_ACCOUNT_ID:role/AWSCookbook403RDSProxy \
	--vpc-subnet-ids $ISTOLATED_SUBNETS \
	--vpc-security-group-ids $RDS_PROXY_SG_ID \
	--require-tls --output text \
	--query DBProxy.DBProxyArn)

# 사용 가능할 때까지 대기
aws rds describe-db-proxies \
	--db-proxy-name $DB_NAME \
	--query DBProxies[0].Status \
	--output text

# RDS_PROXY_ENDPOINT 저장
RDS_PROXY_ENDPOINT=$(aws rds describe-db-proxies \
	--db-proxy-name $DB_NAME \
	--query DBProxies[0].Endpoint \
	--output text)

# RDS proxy 엔드포인트 ARN에서, IAM 정책을 구성할 proxy ID 분리
RDS_PROXY_ID=$(echo $RDS_PROXY_ENDPOINT_ARN | awk -F: '{ print $7 }')
```

③ 람다 함수가 IAM 인증 토큰을 생성할 수 있는 권한을 가진 IAM 정책 생성하고, sed 명령으로 `policy-template.json ` 값 치환

- 그리고 이 파일로 IAM 정책을 생성한다

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["rds-db:connect"],
      "Resource": [
        "arn:aws:rds-db:AWS_REGION:AWS_ACCOUNT_ID:dbuser:RDSProxyID/*"
      ]
    }
  ]
}
```

```bash
# AWS_ACCOUNT_ID, AWS_REGION, RDSProxyID 값 치환
sed -e "s/AWS_ACCOUNT_ID/${AWS_ACCOUNT_ID}/g" \
	-e "s|AWS_REGION|${AWS_REGION}|g" \
	-e "s|RDSProxyID|${RDS_PROXY_ID}|g" \
	policy-template.json > policy.json

# IAM 정책 생성
aws iam create-policy --policy-name AWSCookbook403RdsIamPolicy \
	--policy-document file://policy.json
```

④ 정책을 `DBAppFunction` 람다 함수 역할에 연결

```bash
aws iam attach-role-policy --role-name $DB_APP_FUNCTION_ROLE_NAME \
	--policy-arn arn:aws:iam::$AWS_ACCOUNT_ID:policy/AWSCookbook403RdsIamPolicy

# 프록시가 사용 가능한 상태인지 확인
aws rds describe-db-proxies --db-proxy-name $DB_NAME \
	--query DBProxies[0].Status \
	--output text
```

⑤ `SecretsManagerReadWrite` 정책을 RDS proxy 역할에 연결

- prod 환경에서는 R/W 보다는 최소한의 리소스로 권한의 범위를 지정할 것을 권장

```bash
aws iam attach-role-policy --role-name AWSCookbook403RDSProxy \
	--policy-arn arn:aws:iam::aws:policy/SecretsManagerReadWrite
```

⑥ RDS proxy의 보안 그룹이 RDS 인스턴스의 보안 그룹의 TCP 3306 포트에 대한 인바운드 규칙 추가

```bash
aws ec2 authorize-security-group-ingress \
	--protocol tcp --port 3306 \
	--source-group $RDS_PROXY_SG_ID \
	--group-id $RDS_SG
```

⑦ RDS proxy에 대상을 등록

```bash
aws rds register-db-proxy-targets \
	--db-proxy-name $DB_NAME \
	--db-instance-identifiers $RDS_ID
```

⑧ 람다 함수의 보안 그룹이 RDS proxy 보안 그룹의 TCP 3306 포트에 대한 인바운드 규칙 추가

```bash
aws ec2 authorize-security-group-ingress \
	--protocol tcp --port 3306 \
	--source-group $DB_APP_FUNCTION_SG_ID \
	--group-id $RDS_PROXY_SG_ID
```

⑨ 데이터베이스에 직접 연결 X, RDS proxy endpoint를 `DB_HOST`로 사용하도록 람다 함수 수정

```bash
aws lambda update-function-configuration \
	--function-name $DB_APP_FUNCTION_NAME \
	--environment Variables={DB_HOST=$RDS_PROXY_ENDPOINT}
```

---

**🥕 유효성 검사** : 람다 함수가 RDS proxy를 통해서 RDS에 연결 가능한지 test

```bash
aws lambda invoke \
	--function-name $DB_APP_FUNCTION_NAME \
	response.json && cat response.json
```

🥕 참고

⍢ RDS와 lambda를 사용할 땐, connection pooling 고려!

- 높은 동시성과 큰 빈도로 실행되는 애플리케이션의 경우, db 연결 수가 증가하여 성능에 영향을 미친다.

⍢ RDS proxy를 사용하면 실제 db 연결을 적게 유지하여 성능과 효율성을 높일 수 있다.

- RDS proxy는 연결을 pool로 관리 → 동시성이 증가해도 db에 대한 연결을 필요한 만큼만 늘리고, TCP 오버헤드를 RDS proxy로 offload한다.

⍢ 전송 중 SSL 암호화와 MySQL 및 PostgreSQL RDS 데이터베이스를 지원

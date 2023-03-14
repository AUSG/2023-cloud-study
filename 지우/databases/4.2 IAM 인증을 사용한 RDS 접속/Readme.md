# 4.2 IAM 인증을 사용한 RDS 접속

> _데이터베이스 연결 시, 암호 대신 교체 임시 자격 증명을 사용한다_

① RDS 데이터베이스 인스턴스의 IAM 데이터베이스 인증 활성화

```bash
aws rds modify-db-instance \
	--db-instance-identifier $RDS_DATABASE_ID \
	--enable-iam-database-authentication \
	--apply-immediately
```

② RDS 데이터베이스 인스턴스의 리소스 ID 저장

```bash
DB_RESOURCE_ID=$(aws rds describe-db-instances \
	--query \
	'DBInstances[?DBName==`AWSCookbookRecipe402`].DBResourceId' \
	--output text)
```

③ sed 명령으로 `policy-template.json`의 값을 환경 변수로 치환하여 저장

```bash
sed -e "s/AWS_ACCOUNT_ID/${AWS_ACCOUNT_ID}/g" \
	-e "s|AWS_REGION|${AWS_REGION}|g" \
	-e "s|DBResourceId|${DB_RESOURCE_ID|g" \
	policy-template.json > policy.json
```

④ 앞서 생성한 파일로 IAM 정책 생성하고 EC2가 사용 중인 IAM 역할에 연결

```bash
# IAM policy
aws iam create-policy --policy-name AWSCookbook402RDSPolicy \
	--policy-document file://policy.json

# IAM role attachment
aws iam attach-role-policy --role-name $INSTANCE_ROLE_NAME \
	--policy-arn arn:aws:iam::$AWS_ACCOUNT_ID:policy/AWSCookbook402EC2RDSPolicy
```

⑤ Secrets Manager에서 RDS 관리자 암호 확인하고 RDS 클러스터의 엔드포인트와 암호 출력

```bash
# RDS admin password 환경 변수 저장
RDS_ADMIN_PASSWD=$(aws secretsmanager get-secret-value \
	--secret-id $RDS_SECRET_ARN \
	--query SecretString | jq -r | jq .password | tr -d '"')

# RDS cluster endpoint
echo $RDS_ENDPOINT
>> awscookbookrecipe402.<string>.<region>.rds.amazonaws.com

# RDS cluster password 출력
echo $RDS_ADMIN_PASSWD
```

⑥ 인스턴스 연결 후 mysql 접속

```bash
aws ssm start-session --target $INSTANCE_ID

# mysql 설치
sudo yum -y install mysql

# password, host name으로 db 연결
mysql -u admin -p$DB_ADMIN_PASSWD -h $RDS_ENDPOINT

# IAM 인증 시 사용할 db user 생성
CREATE USER db_user@'%' IDENTIFIED WITH AWSAuthenticationPlugin as 'RDS';
GRANT SELECT ON *.* TO 'db_user'@'%';
>> Query OK, 0 rows affected

quit
```

# 4.8 RDS 데이터 API를 사용해 Aurora Serverless에 대한 REST 액세스 활성화

> _데이터베이스에 대한 데이터 API를 활성화하여 Aurora Serverless 데이터베이스에 연결한다_

① Aurora Serverless 클러스터에서 데이터 API 활성화

```bash
aws rds modify-db-cluster \
	--db-cluster-identifier $CLUSTER_IDENTIFIER \
	--enable-http-endpoint --apply-immediately
```

② `HttpEndpointEnabled`가 `true`로 설정되어 있는지 확인

```bash
aws rds describe-db-clusters \
	--db-cluster-identifier $CLUSTER_IDENTIFIER \
	--query DBClusters[0].HttpEndpointEnabled
```

③ CLI에서 명령 테스트

```bash
aws rds-data execute-statement \
	--secret-arn "$SECRET_ARN" \
	--resource-arn "$CLUSTER_ARN" \
	--database "$DATABASE_NAME" \
	--sql "select * from pg_user" \
	--output json
```

```bash
# 값 저장
echo $SECRET_ARN
echo $DATABASE_NAME
```

④ 관리자 권한으로 콘솔 로그인 → RDS 콘솔 → **[ 쿼리 편집기 ]** 선택 후 값 입력 → **[ 데이터베이스에 연결 ]**

그리고 동일한 쿼리를 실행하고 결과 확인 : `SELECT * from pg_user`

⑤ 데이터베이스 클러스터의 data API를 사용하도록 EC2 인스턴스를 구성하고 sed 명령으로 `policy-template.json` 파일의 secret 값을 환경 변수로 치환

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Action": [
        "rds-data:BatchExecuteStatement",
        "rds-data:BeginTransaction",
        "rds-data:CommitTransaction",
        "rds-data:ExecuteStatement",
        "rds-data:RollbackTransaction"
      ],
      "Resource": "*",
      "Effect": "Allow"
    },
    {
      "Action": [
        "secretsmanager:GetSecretValue",
        "secretsmanager:DescribeSecret"
      ],
      "Resource": "SecretArn",
      "Effect": "Allow"
    }
  ]
}
```

```bash
sed -e "s/SecretArn/${SECRET_ARN}/g" \
	policy-template.json > policy.json
```

⑥ `policy.json` 파일로 IAM 정책을 생성하고 `AWSCookbook408RDSDataPolicy` IAM 정책을 EC2 인스턴스의 IAM 역할에 연결

```bash
aws iam attach-role-policy --role-name $INSTANCE_ROLE_NAME \
	--policy-arn arn:aws:iam::$AWS_ACCOUNT_ID:policy/AWSCookbook408RDSDataPolicy
```

---

🥕 **참고**

⍢ 데이터 API를 사용하면 기존 TCP 데이터베이스 연결을 사용하는 대신 Aurora의 HTTPS 엔드포인트를 노출하고, IAM 인증을 사용해 SQL문을 실행할 수 있다

⍢ 데이터 API로 수행하는 모든 활동은 CloudTrail에 기록

⍢ 애플리케이션의 역할과 연결된 IAM 정책을 사용해 데이터 API 엔드포인트에 대한 액세스를 제어 및 위임

# 5.9 람다에서 VPC 내의 리소스 접근

> _람다 함수로 VPC 내의 ElastiCache 클러스터에 접근할 수 있어야 한다_

① `AWSLambdaVPCAccess` IAM 관리형 정책을 람다 함수에 연결한 역할에 연결

```bash
aws iam attach-role-policy --role-name AWSCookbookLambdaRole \
	--policy-arn arn:aws:iam::aws:policy/service-role/AWSLambdaVPCAccessExecutionRole
```

② 현재 디렉토리에 Redis Python 패키지 설치

```bash
pip install redis -t .
```

③ 람다에 대한 보안 그룹 생성

```bash
LAMBDA_SG=$(aws ec2 create-security-group \
	--group-name Cookbook509LambdaSG \
	--description "Lambda Security Group" --vpc-id $VPC_ID \
	--output text --query GroupId)
```

④ 함수 코드 압축하고, HTTP 요청에 응답할 람다 함수 생성

```bash
zip lambda_function.zip lambda_function.py redis *

LAMBDA_ARN=$(aws lambda create-function \
	--function-name AWSCookbook509Lambda \
	--runtime python3.8 \
	--package-type "Zip" \
	--zip-file fileb://lambda_function.zip \
	--handler lambda_function.lambda_handler --publish \
	--role \
	arn:aws:iam::$AWS_ACCOUNT_ID:role/AWSCookbookLambdaRole \
	--output text --query FunctionArn \
	--vpc-config SubnetIds=${TRIMMED_ISOLATED_SUBNETS},SecurityGroupIds=${LAMBDA_SG})
```

⑤ ElastiCache 서브넷 그룹 생성

```bash
aws elasticache create-cache-subnet-group \
	--cache-subnet-group-name "AWSCookbook509CacheSG" \
	--cache-subnet-group-description "AWSCookbook509CacheSG" \
	--subnet-ids $ISOLATED_SUBNETS
```

⑥ 하나의 노드를 가진 ElastiCache Redis 클러스터 생성

```bash
aws elasticache create-cache-cluster \
	--cache-cluster-id "AWSCookbook509CacheCluster" \
	--cache-subnet-group-name AWSCookbook509CacheSG \
	--engine redis \
	--cache-node-type cache.t3.micro \
	--num-cache-nodes 1
```

⑦ `HOSTNAME`을 클러스터의 엔드포인트를 사용하도록 함수 호출

```bash
aws lambda invoke \
	--cli-binary-format raw-in-base64-out \
	--function-name $LAMBDA_ARN \
	--payload '{"hostname":"HOSTNAME"}' \
	response.json && cat response.json
```

---

🥕 참고

⍢ (defualt) 람다 함수는 AWS 환경에서 프로비저닝한 VPC에 액세스할 수 없다.

**→ VPC에서 ENI를 프로비저닝하면 람다를 VPC에 연결할 수 있다!**

⍢ 함수의 메모리는 실행 환경이 끝나고 다시 시작할 때까지 유지되지 않는다.

- 메모리 지속성에 대한 액세스가 필요하면, Amazon ElastiCache의 redis나 memcached를 사용하여 세션 스토리지 및 키/값 스토리지를 구현할 수 있다.

⇒ 빠른 R/W를 위한 인 메모리 캐시 → 메모리 지속성 + 애플리케이션 수평확장

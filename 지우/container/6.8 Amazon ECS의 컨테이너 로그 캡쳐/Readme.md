# 6.8 Amazon ECS의 컨테이너 로그 캡쳐

> CloudWatch로 로그를 보내기 위해 컨테이너의 ECS 작업 정의에 awslogs 드라이버를 구성하고, 컨테이너가 CloudWatch 로그를 작성할 수 있도록 IAM 역할을 제공하여 컨테이너 로그를 스트리밍한다

① `task-execution-assume-role.json`으로 IAM 역할 생성하고 ECS 작업 실행을 위한 관리형 IAM 정책 연결

```bash
aws iam create-role --role-name AWSCookbook608ECS \
	--assume-role-policy-document file://task-execution-assume-role.json

aws iam attach-role-policy --role-name AWSCookbook608ECS
	--policy-arn arn:aws:iam::aws:policy/servicerole/AmazonECSTaskExecutionRolePolicy
```

② CloudWatch에서 로그 그룹을 생성

```bash
aws logs create-log-group --log-group-name AWSCookbook608ECS
```

③ IAM 역할과 ECS 작업 정의를 구성한 `taskdef.json`으로 ECS 작업을 생성하고 ①에서 생성한 IAM 역할에 연결

```bash
aws ecs register-task-definition --execution-role-arn \
	"arn:aws:iam::$AWS_ACCOUNT_ID:role/AWSCookbook608ECS" \
	--cli-input-json file://taskdef.json
```

④ CDK를 사용해 생성한 ECS 클러스터에서 ECS 작업을 실행

```bash
aws ecs run-task --cluster $ECS_CLUSTER_NAME \
	--launch-type FARGATE --network-configuration "awsvpcConfiguration={subnets=[$VPC_PUBLIC_SUBNETS],securityGroups=[$VPC_DEFAULT_SECURITY_GROUP],assignPublicIp=ENABLED}" \
	--task-definition awscookbook608
```

---

🥕 **참고**

⍢ ECS 작업이 CloudWatch에 로그를 생성할 수 있도록 awslogs 드라이버와 IAM 역할을 구성

⇒ AWS에서 컨테이너로 작업할 때 애플리케이션의 문제 해결 및 디버깅을 위해 일반적으로 사용하는 패턴

- Copilot으로 자동 구성이 가능하지만, ECS는 개발자가 직접 구성해야 한다

⍢ awslogs 드라이버 → /dev/stdout 및 /dev/stderr에 대한 PID 1 프로세스 출력을 캡처

⍢ IAM 역할 → 대부분의 AWS 서비스가 서로 통신하기 위해선 통신에 필요한 수준의 권한을 허용하는 역할 할당

- awslogs logConfiguration 드라이버를 통해 CloudWatchLogs 작업을 허용하는 역할이 연결되어 있어야 한다.
  `taskdef.json`에서 확인 가능
  ```json
  ...
  "logConfiguration": {
    "logDriver": "awslogs",
    "options": {
      "awslogs-group": "AWSCookbook608ECS",
      "awslogs-region": "us-east-1",
      "awslogs-stream-prefix": "LogStream"
    }
  },
  ...
  ```

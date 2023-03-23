# 5.1 ALB에서 람다 함수를 호출하도록 구성

> _서버리스 함수를 사용하는 애플리케이션은 로드밸런서를 통해 인터넷과 통신해야 한다._

- 사전 조건 : ALB(80 포트를 허용하는 보안 그룹과 이에 대한 리스너 규칙), 람다 함수 실행의 IAM 역할

① `lambda_function.py` 압축

```bash
zip lambda_function.zip lambda_function.py
```

② HTTP 요청에 응답할 람다 함수 생성

```bash
LAMBDA_ARN=$(aws lambda create-function \
	--function-name AWSCookbook501Lambda \
	--runtime python3.8 \
	--package-type "Zip" \
	--zip-file fileb://lambda_function.zip \
	--handler lambda_function.lambda_handler --publish \
	--role \
	arn:aws:iam::$AWS_ACCOUNT_ID:role/AWSCookbookLambdaRole \
	--output text --query FunctionArn)
```

③ 대상 유형을 lambda로 설정한 ALB 대상 그룹 생성

```bash
TARGET_GROUP_ARN=$(aws elbv2 create-target-group \
	--name awscookbook501tg \
	--target-type lambda --output text \
	--query TargetGroups[0].TargetGroupArn)
```

④ `add-permission` 명령으로 로드밸런서가 람다 함수를 호출할 수 있는 권한 부여

```bash
aws lambda add-permission \
	--function-name $LAMBDA_ARN \
	--statement-id load-balancer \
	--principal elasticloadbalancing.amazonaws.com \
	--action lambda:InvokeFunction \
	--source-arn $TARGET_GROUP_ARN
```

⑤ `register-targets` 명령으로 람다 함수를 대상으로 등록

```bash
aws elbv2 register-targets \
	--target-group-arn $TARGET_GROUP_ARN \
	--targets Id=$LAMBDA_ARN
```

⑥ 포트 80에 대한 ALB 리스너 수정하고, /function 경로로 향하는 트래픽을 대상 그룹으로 전달하는 규칙 생성

```bash
RULE_ARN=$(aws elbv2 create-rule \
	--listener-arn $LISTENER_ARN --priority 10 \
	--conditions Field=path-pattern,Values='/function' \
	--actions Type=forward,TargetGroupArn=$TARGET_GROUP_ARN \
	--output text --query Rules[0].RuleArn)
```

---

🥕 **유효성 검사** : /function 경로를 호출해서 람다 함수가 호출되는 것 확인

```bash
curl -v $LOAD_BALANCER_DNS/function
```

**🥕 참고**

⍢ ALB의 특정 URL 경로를 요청하면, ALB는 해당 요청을 람다 함수에 전달하여 응답 처리

- ALB는 단일 로드밸런서에서 여러 경로와 대상을 가질 수 있다
- 트래픽의 일부를 컨테이너, 인스턴스 등 특정 대상으로 보낼 수 있다

⍢ 단일 ALB는 HTTP/HTTPS를 통한 모든 트래픽을 처리할 수 있는 유연성을 제공한다

# 4.6 DynamoDB 테이블의 프로비저닝 용량 Auto Scaling

> _애플리케이션 로드가 가변적인 경우, Auto Scaling으로 DynamoDB 테이블의 R/W 용량에 대한 조정대상과 조정정책을 설정하여 구성한다_

① DynamoDB 테이블의 `ReadCapacityUnits`과 `WriteCapacityUnits` 조정 대상 등록

```bash
aws application-autoscaling register-scalable-target \
	--service-namespace dynamodb \
	--resource-id "table/AWSCookbook406" \
	--scalable-dimension "dynamodb:table:ReadCapacityUnits" \
	--min-capacity 5 \
	--max-capacity 10

aws application-autoscaling register-scalable-target \
	--service-namespace dynamodb \
	--resource-id "table/AWSCookbook406" \
	--scalable-dimension "dynamodb:table:WriteCapacityUnits" \
	--min-capacity 5 \
	--max-capacity 10
```

② `read-policy.json`을 참고하여 읽기 용량 조정을 위한 조정 정책과 `write-policy.json`을 참고하여 쓰기 용량 조정을 위한 조정 정책 파일을 생성하고 적용

```bash
aws application-autoscaling put-scaling-policy \
	--service-namespace dynamodb \
	--resource-id "table/AWSCookbook406" \
	--scalable-dimension "dynamodb:table:ReadCapacityUnits" \
	--policy-name "AWSCookbookReadScaling" \
	--policy-type "TargetTrackingScaling" \
	--target-tracking-scaling-policy-configuration \
	file://read-policy.json

aws application-autoscaling put-scaling-policy \
	--service-namespace dynamodb \
	--resource-id "table/AWSCookbook406" \
	--scalable-dimension "dynamodb:table:WriteCapacityUnits" \
	--policy-name "AWSCookbookWriteScaling" \
	--policy-type "TargetTrackingScaling" \
	--target-tracking-scaling-policy-configuration \
	file://write-policy.json
```

---

🥕 **참고**

⍢ DynamoDB는 용량 단위(capacity unit)로 테이블의 읽기 및 쓰기 용량과, 현재 사용량을 기준으로 확장 및 축소 시기를 정의할 수 있다

**⍢ DynamoDB의 두 가지 용량 모드**

- **프로비저닝된 용량 모드** : 초당 데이터 R/W 수를 선택하고 지정한 용량 단위에 따라 요금 부과
  - 용량을 너무 낮게 설정하면 DB 성능이 느려지고 애플리케이션의 오류 및 대기 상태를 경험할 수 있다
  - 용량을 너무 높게 설정하면 불필요한 용량에 대해 비용을 지불할 수 있다
    ⇒ 애플리케이션의 사용 패턴을 이해해야 한다. AutoScaling을 활성화하면 크기 조정 대상을 설정해서 비용과 성능을 모두 최적화할 수 있다!
- **온디맨드 용량 모드** : 애플리케이션이 테이블에서 수행하는 데이터 R/W에 대해 요청당 비용을 지불
  - 트랜잭션이 많으면 온디맨드 모드가 비용이 더 많이 들 수 있다

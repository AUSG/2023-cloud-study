# 6.6 Amazon ECS의 컨테이너 워크로드 자동 확장

> _CPU 부하 증가 시, 더 많은 컨테이너를 추가하도록 ECS 서비스에 대한 CloudWatch 경보 및 조정 정책을 구성한다_

<br>

① curl 명령으로 ECS 서비스 URL에 액세스해 배포되어 있는 것을 확인

```bash
curl -v -m 3 $LOAD_BALANCER_DNS
```

② verbose(-v) 및 3초 제한 시간(-m 3) 플래그로 전체 연결을 확인하고 제한 시간이 설정됐는지 확인

```bash
curl -v -m 3 $LOAD_BALANCER_DNS:80/
```

③ 자동 크기 조정 트리거가 사용할 수 있는 역할을 생성하고 자동 확장을 위한 관리형 정책 연결

```bash
aws iam create-role --role-name AWSCookbook606ECS \
	--assume-role-policy-document file://task-execution-assume-role.json

aws iam attach-role-policy --role-name AWSCookbook606ECS \
	--policy-arn arn:aws:iam::aws:policy/service-role/AmazonEC2ContainerServiceAutoScaleRole
```

④ 자동 크기 조정 대상을 등록하고 CPU 사용량이 평균 50% 이상되면 정책을 실행하도록 설정

```bash
aws application-autoscaling register-scalable-target \
	--service-namespace ecs \
	--scalable-dimension ecs:service:DesiredCount \
	--resource-id service/$ECS_CLUSTER_NAME/AWSCookbook606 \
	--min-capacity 2 \
	--max-capacity 4

aws application-autoscaling put-scaling-policy --service-namespace ecs \
	--scalable-dimension ecs:service:DesiredCount \
	--resource-id service/$ECS_CLUSTER_NAME/AWSCookbook606 \
	--policy-name cpu50-awscookbook-606 --policy-type TargetTrickingScaling \
	--target-tracking-scaling-policy-configuration file://scaling-policy.json
```

⑤ curl 명령으로 높은 CPU 로드를 시뮬레이션

```bash
curl -v -m 3 $LOAD_BALANCER_DNS/cpu
```

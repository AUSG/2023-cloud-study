# 6.7 이벤트를 통해 Fargate 컨테이너 작업 시작

> _Amazon EventBridge를 사용해 파일이 S3에 업로드되면 Fargate에서 ECS 컨테이너 작업 시작을 트리거하도록 구성한다_

<br>

① CloudTrail이 S3 버킷의 이벤트를 기록하도록 구성

```bash
aws cloudtrail put-event-selectors --trail-name $CLOUD_TRAIL_ARN \
	--eventselectors "[{ \"ReadWriteType\":\"WriteOnly\", \"IncludeManagementEvents\":false, \"DataResources\":[{\"Type\":\"AWS::S3::Object\",\"Values\":[\"arn:aws:s3:::$BUCKET_NAME/input/\"] }], \"ExcludeManagementEventSources\":[] }]"
```

② 역할을 생성하고 EventBridge에 대한 IAM 정책(`policy1.json`)을 연결

```bash
aws iam create-role --role-name AWSCookbook607RuleRole \
	--assume-role-policy-document file://policy1.json
```

③ IAM과 ECS task에 대한 권한을 포함한 정책(`policy2.json`)을 ②번에서 생성한 역할에 연결

```bash
aws iam put-role-policy --role-name AWSCookbook607RuleRole \
	--policy-name ECSRunTaskPermissionsForEvents \
	--policy-document file://policy2.json
```

④ S3 버킷의 파일 업로드를 모니터링하는 EventBridge 규칙을 생성

```bash
aws events put-rule --name "AWSCookbookRule" --role-arn "arn:aws:iam::$AWS_ACCOUNT_ID:role/AWSCookbook607RuleRole" \
	--event-pattern "{\"source\":[\"aws.s3\"],\"detail-type\":[\"AWS API Call via CloudTrail\"],\"detail\":{\"eventSource\":[\"s3.amazonaws.com\"],\"eventName\":[\"CopyObject\",\"PutObject\",\"CompleteMultipartUpload\"],\"requestParameters\":{\"bucketName\":[\"$BUCKET_NAME\"]}}}"
```

⑤ `targets-template.json` 값을 치환하여 `targets.json` 생성

```bash
sed -e "s|AWS_ACCOUNT_ID|${AWS_ACCOUNT_ID}|g" \
	-e "s|AWS_REGION|${AWS_REGION}|g" \
	-e "s|ECSClusterARN|${ECS_CLUSTER_ARN}|g" \
	-e "s|TaskDefinitionARN|${TASK_DEFINITION_ARN}|g" \
	-e "s|VPCPrivateSubnets|${VPC_PRIVATE_SUBNETS}|g" \
	-e "s|VPCDefualtSecurityGroup|${VPC_DEFAULT_SECURITY_GROUP}|g" \
	targets-template.json > targets.json
```

⑥ ECS cluster, ECS task 정의, IAM 역할, 네트워킹 매개 변수를 사용하는 규칙 대상을 생성한 규칙이 트리거할 대상을 지정 (Fargate에서 컨테이너 시작)

```bash
aws events put-targets --rule AWSCookbookRule \
	--targets file://targets.json
```

⑦ S3 버킷이 비어 있는지 확인하고 이미지 파일을 버킷에 복사

```bash
aws s3 ls s3://$BUCKET_NAME

aws s3 cp docker.png s3://$BUCKET_NAME/input/docker.png
```

⑧ 이미지 파일 처리를 위한 ECS 작업을 트리거하고 S3 버킷에 생성된 출력 디렉토리 확인

```bash
aws ecs list-tasks --cluster $ECS_CLUSTER_ARN

aws s3 ls s3://$BUCKET_NAME/output/
```

---

🥕 **참고**

⍢ 서버리스 이벤트 기반 아키텍처 → 워크로드를 쉽게 확장하고 아키텍처의 탄력성을 높일 수 있다

- 보통 이벤트 기반 아키텍처는 S3와 람다 함수를 사용
- 그러나 **장기 실행 데이터 처리 및 계산 작업**은 제한 시간이 없는 Fargate가 더 적합하다 (람다는 15분의 제한 시간이 있다)

⍢ ECS → 작업과 서비스를 실행할 수 있다

- 서비스는 작업으로 구성, 서비스가 특정 작업 집합을 실행한다는 점에서 장기 실행이 가능하다

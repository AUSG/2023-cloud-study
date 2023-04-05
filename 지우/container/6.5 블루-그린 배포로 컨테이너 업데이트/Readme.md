# 6.5 블루/그린 배포로 컨테이너 업데이트

> _AWS CodeDeploy를 통해 블루/그린 배포를 적용하여 애플리케이션을 최신 버전으로 배포하고 배포를 실패하는 경우에는 쉽게 롤백할 수 있도록 한다_

- `codedeploy.json` → CodeDeploy 배포그룹을 정의하는 json 파일
- `appsepc.yaml` → CodeDeploy 배포 스펙을 관리하는 yaml 파일
- `deployment.json` → 실제로 배포할 정보를 정의한 json 파일

<br>

① CDK 스택을 배포한 후, 웹 브라우저를 열어 CDK 출력의 `LOAD_BALANCER_DNS` 주소를 확인

```bash
git clone https://github.com/AWSCookbook/Containers.git
cd Containers/605-Updating-Containers-With-BlueGreen/cdk-AWS-Cookbook-605/
test -d .venv || python3 -m venv .venv
source .venv/bin/activate
pip install --upgrade pip
pip install -r requirements.txt
cdk deploy
```

- ‘블루’ 애플리케이션을 확인한다

```bash
open http://$LOAD_BALANCER_DNS:8080
```

② `assume-role-policy.json`으로 IAM 역할 생성하고 `CodeDeployRoleforECS`에 대한 IAM 관리형 정책을 역할에 연결

```bash
aws iam create-role --role-name ecsCodeDeployRole \
	--assume-role-policy-document file://assume-role-policy.json

aws iam attach-role-policy --role-name ecsCodeDeployRole \
	--policy-arn arn:aws:iam::aws:policy/AWSCodeDeployRoleforECS
```

③ CodeDeploy의 ‘그린’ 대상 그룹으로 사용할 새 ALB 대상 그룹 생성

```bash
aws elbv2 create-target-group --name "GreenTG" --port 80 \
	--protocol HTTP --vpc-id $VPC_ID --target-type ip
```

④ CodeDeploy 애플리케이션 생성

```bash
aws deploy create-application --application-name awscookbook-605 \
	--compute-platform ECS
```

⑤ sed 명령으로 `codedeploy-template.json`의 환경변수 값 치환하고 배포 그룹 생성

```bash
sed -e "s/AWS_ACCOUNT_ID/${AWS_ACCOUNT_ID}/g" \
	-e "s|PROD_LISTENER_ARN|${PROD_LISTENER_ARN}|g" \
	-e "s|TEST_LISTENER_ARN|${TEST_LISTENER_ARN}|g" \
	codedeploy-template.json > codedeploy.json
```

```bash
aws deploy create-deployment-group --cli-input-json file://codedeploy.json
```

⑥ 업데이트할 애플리케이션 정보를 포함한 `appspec-template.yaml` 내용을 환경변수의 값으로 치환

```bash
sed -e "s|FargateTaskGreenArn|${FARGATE_TASK_GREEN_ARN}|g" \
	appspec-template.yaml > appspec.yaml
```

⑦ CDK 배포를 통해 생성한 S3 버킷에 `appspec.yaml`을 복사하고 `deployment-template.json` 파일의 S3 버킷 이름을 치환하여 배포에 대한 구성 파일 생성

```bash
aws s3 cp ./appspec.yaml s3://$BUCKET_NAME

sed -e "s|S3BucketName|${BUCKET_NAME}|g" \
	deployment-template.json > deployment.json

aws deploy create-deployment --cli-input-json file://deployment.json
```

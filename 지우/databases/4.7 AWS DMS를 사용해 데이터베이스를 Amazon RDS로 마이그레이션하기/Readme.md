# 4.7 AWS DMS를 사용해 데이터베이스를 Amazon RDS로 마이그레이션하기

> _DMS가 데이터베이스에 연결할 수 있도록 VPC 보안 그룹 및 IAM 권한을 구성하고, 원본 및 대상 DB에 대한 DMS 엔드포인트를 구성하여 데이터를 이동할 수 있다_

① 복제 인스턴스에 대한 보안 그룹 생성

```bash
DMS_SG_ID=$(aws ec2 create-security-group \
	--group-name AWSCookbook407DMSSG \
	--description "DMS Security Group" --vpc-id $VPC_ID \
	--output text --query GroupId)
```

② DMS 보안 그룹에 원본 및 대상 데이터베이스의 TCP 3306 포트에 대한 인바운드 규칙 생성

```bash
aws ec2 authorize-security-group-ingress \
	--protocol tcp --port 3306 \
	--source-group $DMS_SG_ID \
	--group-id $SOURCE_RDS_SG
aws ec2 authorize-security-group-ingress \
	--protocol tcp --port 3306 \
	--source-group $DMS_SG_ID \
	--group-id $TARGET_RDS_SG
```

③ `assume-role-policy.json`을 참고하여 DMS에 대한 역할을 생성하고, 관리형 DMS 정책을 역할에 연결

- DMS는 특정 이름과 특정 정책을 가진 IAM 역할을 요구하므로 DMS를 기존에 사용했다면 이미 역할이 계정에 존재하므로 오류를 반환할 수 있다

```bash
# DMS 역할 생성
aws iam create-role --role-name dms-vpc-role \
	--assume-role-policy-document file://assume-role-policy.json

# 관리형 DMS 정책 연결
aws iam attach-role-policy --role-name dms-vpc-role --policy-arn \
	arn:aws:iam::aws:policy/service-role/AmazonDMSVPCManagementRole
```

④ 복제 인스턴스에 대한 복제 서브넷 그룹과 인스턴스를 생성하고 ARN 환경 변수에 저장

- DMS는 테이블을 병렬로 전송 ⇒ 더 많은 양의 데이터에 대해 더 큰 인스턴스 크기가 필요!

```bash
# 복제 인스턴스에 대한 복제 서브넷 그룹 생성
REP_SUBNET_GROUP=$(aws dms create-replication-subnet-group \
	--replication-subnet-group-identifier awscookbook407 \
	--replication-subnet-group-description "AWSCookbook407" \
	--subnet-ids $ISOLATED_SUBNETS \
	--query ReplicationSubnetGroup.ReplicationSubnetGroupIdentifier \
	--output text)

# 복제 인스턴스 생성하고 ARN 저장
REP_INSTANCE_ARN=$(aws dms create-replication-instance \
	--replication-instance-identifier awscookbook407 \
	--no-publicly-accessible \
	--replication-instance-class dms.t2.medium \
	--vpc-security-group-ids $DMS_SG \
	--replication-subnet-group-identifier $REP_SUBNET_GROUP \
	--allocated-storage 8 \
	--query ReplicationInstance.ReplicationInstanceArn \
	--output text)

# ReplicationInstanceStatus가 가용 상태가 될 때까지 대기
aws dms describe-replication-instances \
	--filter=Name=replication-instance-id,Values=awscookbook407 \
	--query ReplicationInstances[0].ReplicationInstanceStatus
```

⑤ Secrets Manager에서 원본 및 대상의 DB 관리자 암호를 확인하고 환경 변수에 저장

```bash
RDS_SOURCE_PASSWD=$(aws secretsmanager get-secret-value --secret-id \
	$RDS_SOURCE_SECRET_NAME --query SecretString --output text | jq .password | tr -d '"')

RDS_TARGET_PASSWD=$(aws secretsmanager get-secret-value --secret-id \
	$RDS_TARGET_SECRET_NAME --query SecretString --output text | jq .password | tr -d '"')
```

⑥ DMS에 대한 원본 및 대상 엔드포인트를 생성하고 ARN 변수에 저장

```bash
SOURCE_ENDPOINT_ARN=$(aws dms create-endpoint \
	--endpoint-identifier awscookbook407source \
	--endpoint-type source --engine-name mysql \
	--username admin --password $RDS_SOURCE_PASSWD \
	--server-name $SOURCE_RDS_ENDPOINT --port 3306 \
	--query Endpoint.EndpointArn --output text)

TARGET_ENDPOINT_ARN=$(aws dms create-endpoint \
	--endpoint-identifier awscookbook407target \
	--endpoint-type target --engine-name mysql \
	--username admin --password $RDS_TARGET_PASSWD \
	--server-name $TARGET_RDS_ENDPOINT --port 3306 \
	--query Endpoint.EndpointArn --output text)
```

⑦ 복제 작업을 생성하고 ready 상태가 될 때까지 대기 후 복제 작업 시작

```bash
# 복제 작업 생성
REPLICATION_TASK_ARN=$(aws dms create-replication-task \
	--replication-task-identifier awscookbook-task \
	--source-endpoint-arn $SOURCE_ENDPOINT_ARN \
	--target-endpoint-arn $TARGET_ENDPOINT_ARN \
	--replication-instance-arn $REP_INSTANCE_ARN \
	--migration-type full-load \
	--table-mappings file://table-mapping-all.json \
	--query ReplicationTask.ReplicationTaskArn --output text)

# status: ready
aws dms describe-replication-tasks \
	--filters "Name=replication-task-arn,Values=$REPLICATION_TASK_ARN" \
	--query "ReplicationTasks[0].Status"

# 복제 작업 시작
aws dms start-replication-task \
	--replication-task-arn $REPLICATION_TASK_ARN \
	--start-replication-task-type start-replication
```

---

🥕 **유효성 검사** : 마이그레이션 성공 여부 확인

```bash
# 복제 작업 진행 상황 모니터링
aws dms describe-replication-tasks

# 테이블의 마이그레이션 여부 확인
aws dms describe-replication-tasks \
	--query ReplicationTasks[0].ReplicationTaskStatus
```

🥕 **참고**

⍢ DMS는 복제 인스턴스의 원본 및 대상 엔드포인트를 테스트할 수 있는 기능을 제공

```bash
# 연결 테스트 수행
aws dms test-connection \
	--replication-instance-arn $REP_INSTANCE_ARN \
	--endpoint-arn $SOURCE_ENDPOINT_ARN

aws dms test-connection \
	--replication-instance-arn $REP_INSTANCE_ARN \
	--endpoint-arn $TARGET_ENDPOINT_ARN

# 연결 작업 테스트 상태와 결과 확인
aws dms describe-connections --filter \
	"Name=endpoint-arn,Values=$SOURCE_ENDPOINT_ARN,$TARGET_ENDPOINT_ARN"
```

⍢ 같은 VPC, 다른 AWS 계정, 또는 AWS 환경이 아닌 다른 곳의 여러 유형의 원본 및 대상 데이터베이스 지원

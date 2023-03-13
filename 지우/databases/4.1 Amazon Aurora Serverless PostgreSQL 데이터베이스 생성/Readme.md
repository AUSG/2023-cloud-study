# 4.1 Amazon Aurora Serverless PostgreSQL 데이터베이스 생성

> _변동하는 애플리케이션 사용량에 따라 확장 가능하고 비용 효율적인 데이터베이스 솔루션을 구축한다_

- 기존 PostgreSQL과 호환 가능하고 운영 오버헤드가 낮은 Aurora Serverless 데이터베이스 선정

① AWS Secrets Manager로 복잡한 암호 생성

```bash
ADMIN_PASSWD=$(aws secretsmanager get-random-password \
	--exclude-punctuation \
	--password-length 41 --require-each-included-type \
	--output text --query RandomPassword)
```

② VPC 서브넷을 지정하여 데이터베이스 서브넷 그룹을 생성

- 데이터베이스 subnet group → RDS ENI 배치 단순화

```bash
aws rds create-db-subnet-group \
	--db-subnet-group-name awscookbook401subnetgroup \
	--db-subnet-group-description "AWSCookbook401 subnet group" \
	--subnet-ids $SUBNET1_ID $SUBNET2_ID
```

③ 데이터베이스에 대한 VPC 보안 그룹 생성

```bash
DB_SG_ID=$(aws ec2 create-security-group \
	--group-name AWSCookbook401sg \
	--description "Aurora Serverless Security Group" \
	--vpc-id $VPC_ID --output text --query GroupId)
```

④ engine-mode는 `serverless`로 지정하여 데이터베이스 클러스터 생성

- 설정내용: cluster name, engine, engine mode, engine version, username&passwd, subnet group name, vpc security group name

```bash
aws rds create-db-cluster \
	--db-cluster-identifier awscookbook401dbcluster \
	--engine aurora-postgresql \
	--engine-mode serverless \
	--engine-version 10.14 \
	--master-username dbadmin \
	--master-user-password $ADMIN_PASSWORD \
	--db-subnet-group-name awscookbook401subnetgroup \
	--vpc-security-group-ids $DB_SG_ID
```

```bash
# 사용 가능 상태가 될 때까지 대기 (몇 분 소요)
aws rds describe-db-clusters \
	--db-cluster-identifier awscookbook401dbcluster \
	--output text --query DBClusters[0].Status
```

⑤ 새로운 Auto Scaling 용량 목표(8분, max 16개)로 자동 확장하도록 수정

- 용량: 최소 8, 최대 16
- 5분 동안 사용하지 않으면 일시 중지

```bash
aws rds modify-db-cluster \
	--db-cluster-identifier awscookbook401dbcluster --scaling-configuration \
	MinCapacity=8,MaxCapacity=16,SecondsUntilAutoPause=300,TimeoutAction='ForceApplyCapacityChange',AutoPause=true
```

```bash
# 5분 이상 뒤에 용량이 0으로 축소되는 것 확인
aws rds describe-db-clusters \
	--db-cluster-identifier awscookbook401dbcluster \
	--output text --query DBClusters[0].Capacity
```

⑥ EC2 인스턴스 보안 그룹에 PostgreSQL 포트에 대한 액세스 권한 부여

```bash
aws ec2 authorize-security-group-ingress \
	--protocol tcp --port 5432 \
	--source-group $INSTANCE_SG \
	--group-id $DB_SG_ID
```

---

🥕 **참고**

⍢ RDS 클러스터 삭제 시, 기본적으로 최종 스냅샷을 생성

⍢ `MaxCapacity`, `SecondsUntilAutoPause` 설정으로 사용량 급증에 의한 비용 방지

⍢ Aurora Serverless는 3개의 AZ에 걸쳐 데이터베이스의 기본 스토리지 복제

- 강한 복원력을 갖지만, 작동 오류를 방지하기 위한 자동 백업을 지속적으로 사용해야 한다
- Aurora Serverless는 기본적으로 자동 백업이 활성화되어 있다
- 최대 백업 보존 기간은 35일

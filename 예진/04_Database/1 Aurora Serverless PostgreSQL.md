# 1. Amazon Aurora Serverless PostgreSQL

### 실습

---

변동하는 애플리케이션 사용량에 따라 확장할 수 있는 비용 효율적인 데이터베이스 솔루션이 필요하고 운영오버헤드가 낮으며 PostgreSQL을 사용하는 기존 애플리케이션과 호환되는 솔루션을 구축해보자

→ AWS Secrets Manager의 암호를 사용하는 Aurora Serverless 데이터베이스 클러스터를 구성하고 생성한다. 그 후, **Autoscale을 적용하고 자동 일시 중지**를 활성화한다.

1. AWS Secrets Manager로 복잡한 암호 생성

   ```bash
   ADMIN_PASSWORD=$(aws secretsmanager get-random-password --exclude-punctuation --password-length 41 --require-each-included-type --output text --query RandomPassword)
   ```

2. VPC 서브넷을 지정해 클러스터에 사용할 데이터베이스 서브넷 그룹을 생성

   1. 데이터베이스 서브넷 그룹을 사용하여 RDS 인스턴스를 실행할 서브넷을 선택하고, ENI가 배치될 위치를 명확하게 지정할 수 있다

   ```bash
   aws rds create-db-subnet-group --db-subnet-group-name awscookbook401subnetgroup --db-subnet-group-description "AWSCookbook401 subnet group" --subnet-ids $SUBNET_ID_1 $SUBNET_ID_
   ```

3. 데이터베이스에 대한 VPC 보안 그룹 생성

   ```bash
   DB_SECURITY_GROUP_ID=$(aws ec2 create-security-group --group-name AWSCookbook401sg --description "Aurora Serverless Security Group" --vpc-id $VPC_ID --output text --query GroupId)
   ```

4. `engine-mode`를 `serverless`로 지정해 데이터베이스 클러스터를 생성한다.

   →`serverless` 엔진 모드를 사용하면, RDS 인스턴스를 비용 효율적으로 운영하고, 자동으로 확장할 수 있도록 하며, 유연하게 운영할 수 있도록 할 수 있다.

   ```bash
   aws rds create-db-cluster --db-cluster-identifier awscookbook401dbcluster --engine aurora-postgresql --engine-mode serverless --engine-version 10.14 --master-username dbadmin --master-user-password $ADMIN_PASSWORD --db-subnet-group-name awscookbook401subnetgroup --vpc-security-group-ids $DB_SECURITY_GROUP_ID
   ```

5. 사용가능 상태가 될때까지 기다린다.

   ```bash
   aws rds describe-db-clusters --db-cluster-identifier awscookbook401dbcluster --output text --query 'DBClusters[0].Status'
   ```

   → `available` 확인

6. 새로운 AutoScaling 용량 목표(8분, 최대 16개)로 자동확장하도록 데이터베이스를 수정하고 5분 동안 사용하지 않으면 자동 일시 중지를 하도록 설정한다.

   ```bash
   aws rds modify-db-cluster --db-cluster-identifier awscookbook401dbcluster --scaling-configuration MinCapacity=8,MaxCapacity=16,SecondsUntilAutoPause=300,TimeoutAction='ForceApplyCapacityChange',AutoPause=true
   ```

   → 5분 후, 데이터베이스 용량이 0으로 축소되는 것을 확인할 수 있다.

   ```bash
   aws rds describe-db-clusters --db-cluster-identifier awscookbook401dbcluster --output text --query 'DBClusters[0].Capacity'
   ```

7. EC2 인스턴스의 보안그룹에 PostgreSQL 포트에 대한 액세스 권한을 부여한다.

   ```bash
   aws ec2 authorize-security-group-ingress --protocol tcp --port 5432 --source-group $INSTANCE_SG --group-id $DB_SECURITY_GROUP_ID
   ```

<br>
<br>

### 유효성 검사

---

1. RDS 클러스터의 엔드포인트 확인 (클러스터에 애플리케이션을 연결하려면 이 엔드포인트를 사용하여 애플리케이션과 RDS 클러스터를 연결해야 한다.)
2. psql 명령으로 데이터베이스에 연결할 수 있도록 PostgreSQL 패키지를 설치
3. psql 명령으로 데이터베이스에 연결 (데이터베이스 용량이 0에서 확장하여 시간이 걸릴 수 있다.)
4. psql 종료
5. 클러스터 용량을 다시 확인하면 데이터베이스가 최솟값으로 확장됐는지 확인할 수 있다.

<br>
<br>

### 코드 리뷰

---

클러스터는 사용량 요구 사항에 따라 용량을 자동으로 확장한다. `MaxCapacity=16`으로 상한선을 설정하면 최대 용량이 제한돼 사용량이 급증하더라도 예상치 못한 비용을 막을 수 있다.

클러스터는 `SecondsUntilAutoPause` 설정을 참고해 연결이나 활동이 없으며 용량을 0으로 설정

### Amazon Aurora

---

- Aurora는 MySQL과 PostgreSQL의 장점을 결합하여, 엔터프라이즈급 성능, 확장성, 내구성 및 보안을 제공
- 빠른 처리 속도와 함께 6개 이상의 가용 영역에 걸쳐 복제본을 자동으로 배치하여 데이터 손실 및 시스템 장애에 대한 내구성을 보장
- Autoscale 기능
- Aurora 서버리스 확장은 클러스터에 예약된 컴퓨팅 및 메모리에 해당하는 용량 단위로 측정한다.

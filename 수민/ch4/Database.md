# 4.1 Amazon Aurora Serverless PostgreSQL
Amazon Aurora는 Aurora Serverless v2, 메모리 최적화 및 버스트 가능 성능이라는 3가지 DB 인스턴스 클래스 유형을 제공

db.serverless - Aurora Serverless v2에서 사용하는 특수 DB 인스턴스 클래스. Aurora는 워크로드의 변화에 따라 컴퓨팅, 메모리 및 네트워크 리소스를 동적으로 조정.

``` bash
# secretmanager 이용해서 암호 생성
ADMIN_PASSWORD=$(aws secretsmanager get-random-password --exclude-punctuation --password-length 41 --require-each-included-type --output text --query RandomPassword)

# 데이터베이스 서브넷 그룹 생성(RDS ENI 그룹을 단순화하기 위함)
aws rds create-db-subnet-group --db-subnet-group-name aws401subnet --db-subnet-group-description "aws401 subnet group" --subnet-ids subnet-1 subnet-2

# 데이터베이스 vpc 보안 그룹 생성
DB_SECURITY_GROUP_ID=$(aws ec2 create-security-group --group-name aws401sg --description "aurora serverless security group" --vpc-id vpc-id --output text --query GroupId)

# engine-mode를 serverless로 데이터베이스 클러스터 생성
aws rds create-db-cluster --db-cluster-identifier aws401dbcluster --engine aurora-postgresql --engine-mode serverless --engine-version 10.14 --master-username dbadmin --master-user-password $ADMIN_PASSWORD --db-subnet-group-name aws401subnet --vpc-security-group-ids $DB_SECURITY_GROUP_ID

# Autoscaling 용량 목표 설정
aws rds modify-db-cluster --db-cluster-identifier aws401dbcluster --scaling-configuration MinCapacity=8,MaxCapacity=16,SecondsUntilAutoPause=300,TimeoutAction='ForceApplyCapacityChange',AutoPause=true
```

# 4.2 IAM 인증을 사용한 RDS 접속
aws를 이용한 db 접근 방법 중 IAM을 이용한 방법!
```
코드 대신 과정 나열.(콘솔로 진행)
먼저 RDS 데이터베이스의 인스턴스의 IAM 데이터베이스 인증을 활성화(처음 생성시 설정 가능) -> ec2 인스턴스 역할에 rds에 접근할 수 있는 정책 추가 -> 그 다음 IAM 인증 시 사용할 새로운 데이터베이스 사용자 생성 -> RDS 인증 토큰을 생성하고 ec2에서 mysql의 명령어로 접근할 때 토큰을 password로 쓰고 aws에서 제공하는 rds CA파일을 다운로드해서 접근
```
IAM은 임시 토큰을 15분 동안 유지하고 권한 부여는 데이터베이스 사용자로 부여!

# 4.3 RDS 프록시를 사용한 람다와 RDS 연결
이번 챕터도 cli 대신 콘솔로 진행
```
들어가기 앞서 커넥션 풀링을 사용한다고 했는데 커넥션 풀링이란 데이터베이스와 연결된 커넥션을 미리 만들어 놓고 이를 pool로 관리하는 방식. 이렇게 커넥션 풀링 방식을 이용하면 연결 수를 최소화할 수 있고 성능을 향상시킬 수 있음. RDS프록시는 연결을 pool로 관리함.

mysql rds를 생성하고 기다린 다음 proxy를 생성하면됨. IAM은 rds 관련으로 선택! 그 다음 RDS 프록시를 만들면 기본적으로 endpoint가 만들어지고 대상그룹을 지정할 수 있는데 db 인스턴스로 지정. RDS 프록시를 이용하는 vpc은 보안그룹에 db로 향하는 3306포트에 대해 허용하고 람다함수도 간단하게 db proxy를 지정할 수 있는 설정이 구성칸에 존재. (추가적인 과정이 필요하지만 초기에 db 인스턴스 버전을 잘못 선택해서 proxy로 대상 선택이 안돼서 여기까지는 구성완료!) 추후에 필요한 과정이 단순 보안그룹 설정과 연결 확인 과정

rds에 직접 접근하는 것보다 proxy로 관리하면 커넥션도 관리할 수 있고 성능 향상을 기대할 수 있음.
```

# 4.4 기존 Amazon RDS for MySQL 데이터베이스의 스토리지 암호화
```
# db 저장소 암호화 확인
aws rds describe-db-instances --db-instance-identifier database-1 --query DBInstances

# kms키 생성
KEY_ID=$(aws kms create-key \
    --tags TagKey=Name,TagValue=aws404key \
    --description "AWSCookbook rds key" \
    --query KeyMetadata.KeyId \
    --output text)

# 별칭 생성
aws kms create-alias \
    --alias-name alias/aws404 \
    --target-key-id $KEY_ID

# 기존 데이터베이스 읽기 전용 복제본
aws rds create-db-instance-read-replica --db-instance-identifier aws404db-replica --source-db-instance-identifier database-1 --max-allocated-storage 23

# 복제본 스냅샷
aws rds create-db-snapshot --db-instance-identifier aws404db-replica --db-snapshot-identifier aws404-snapshot

# kms키를 이용해 스냅샷 복사
aws rds copy-db-snapshot --copy-tags --source-db-snapshot-identifier aws404-snapshot --target-db-snapshot-identifier aws4040-snapshot-enc --kms-key-id alias/aws404

# 스냅샷으로 새 rds 인스턴스 복원
aws rds restore-db-instance-from-db-snapshot --db-subnet-group-name aws401subnet --db-instance-identifier aws404db-enc --db-snapshot-identifier aws4040-snapshot-enc

# 이후 스토리지 암호화 확인
```

# 4.5 RDS 데이터베이스의 암호 교체 자동화
```
rds 데이터베이스의 관리자의 암호를 교체하는데 람다 함수를 이용해서 암호를 교체!
secrets manager를 이용해서 rds의 요구사항을 만족하는 randonpwd 생성 -> 람다함수를 만들어서 30일마다 교체를 하도록 rotate 람다 함수를 만듬-> 이때 secrets manager가 람다 함수를 호출할 수 있는 권한을 줘야하고 rds 보안그룹에 람다의 보안 그룹의 3306에 대한 액세스를 허용하는 규칙이 필요 
***
aws Secrets Manager는 자체 인프라를 운영하기 위한 사전 투자 및 지속적인 유지 관리 비용 없이 애플리케이션, 서비스 및 IT 리소스에 대한 액세스를 보호할 수 있음.
규제 및 규정 준수 요구 사항을 준수해야 하는 책임이 있는 보안 관리자는 애플리케이션에 영향을 미칠 위험 부담 없이 Secrets Manager를 사용하여 보안 정보를 모니터링하고 교체할 수 있고 애플리케이션에 하드코딩된 보안 정보를 교체하려는 개발자는 Secrets Manager에서 프로그래밍 방식으로 보안 정보를 검색할 수 있음
***

```

# 4.6 DynamoDB 테이블의 프로비저닝 용량 Auto Scaling
```
먼저 autoscaling에 조정 대상을 등록하는 작업이 필요(read,write 따로)
그 다음 읽기 용량 조정 정책과 쓰기 용량 조정 정책을 json 파일로 만들어서 autoscaling에 조정 정책을 적용.

dynamodb는 프로비저닝과 온디맨드라는 두가지 용량 모드 제공. 프로비저닝 모드는 초당 데이터 읽기 및 쓰기 수를 선택할 수 있으며 요금도 지정한 용량 단위로 부과. 반대로 온디맨드 모드는 테이블에서 수행하는 데이터 읽기 및 쓰기에 대해 요청당 비용을 지불. 즉, 트랜잭션이 빈번한 경우에는 온디맨드 모드가 비용이 높게 될 수도 있음.
```

# 4.7 AWS DMS를 사용해 데이터베이스를 Amazon RDS로 마이그레이션하기
```
AWS Database Migration Service(AWS DMS) 는 관계형 데이터베이스, 데이터 웨어하우스, NoSQL 데이터베이스 및 기타 유형의 데이터 저장소를 마이그레이션할 수 있는 클라우드 서비스
두 데이터베이스가 서로 다른 스키마를 가지고 있다면 mapping파일을 추가해서 데이터를 변환할 수 있고 AWS SCT로 필요한 변환을 식별할 수 있음.
```

# 4.8 RDS 데이터를 API를 사용해 Aurora Serverless에 대한 REST 액세스 활성화
```
aws cli를 통해 db에 쿼리를 날리기 위해서는  db의 serverless 클러스터에서 데이터 api를 활성화해야하고 rds의 data api를 사용할 수 있도록 EC2인스턴스의 IAM 역할에 정책을 추가해줘야함.

{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Action": [
                "rds-data:BatchExecuteStatement",
                "rds-data:BeginTransaction",
                "rds-data:CommitTransaction",
                "rds-data:ExecuteStatement",
                "rds-data:RollbackTransaction"
            ],
            "Resource": "*",
            "Effect": "Allow"
        },
        {
            "Action": [
                "secretsmanager:GetSecretValue",
                "secretsmanager:DescribeSecret"
            ],
            "Resource": "SecretArn",
            "Effect": "Allow"
        }
    ]
}
하지만 ec2인스턴스에서 query를 날리기 위해선 secret arn이나 cluster arn, db name이 필요함.(secrets manager가 암호를 관리하도록 db를 설정해야함.)
그리고 추가적으로 aurora serverless v2에 대한 http-endpoint를 enable하는 방법이 없음.
https://docs.aws.amazon.com/ko_kr/AmazonRDS/latest/AuroraUserGuide/data-api.html#data-api.enabling
안내문서에도 severless v1에 대한 설명만 존재! cli를 입력해도 false에서 true로 바뀌지 않음. 그리고 추가적으로 serverless 에서만 쿼리 편집기를 이용할 수 있고 데이터 api가 활성화되어야지만 그때서야 이용할 수 있음.
```
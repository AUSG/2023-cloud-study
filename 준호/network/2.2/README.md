## 2.2 서브넷과 라우팅 테이블을 포함한 네트워크 티어 생성

### 문제 설명

리소스의 분할 및 중복을 위해 개별 IP 공간으로 구성한 VPC 네트워크를 생성해야 한다.

### 해결 방법

1. VPC 내에 라우팅 테이블을 생성한다.
2. VPC의 다른 AZ에 서브넷 2개를 생성한다.
3. 라우팅 테이블을 서브넷과 연결한다.

### 준비 단계

```bash
# create VPC
VPC_ID=$(aws ec2 create-vpc --cidr-block 10.10.0.0/23 \
--tag-specifications \
'ResourceType=vpc,Tags=[{Key=Name,Value=AWSCookbook202}]' \
--output text --query Vpc.VpcId)
```

### 작업 방법

1. 라우팅 테이블을 생성한다.

```bash
ROUTE_TABLE_ID=$(aws ec2 create-route-table --vpc-id $VPC_ID \
--tag-specifications \
'ResourceType=route-table,Tags=[{Key=Name,Value=AWSCookbook202}]' \
--output text --query RouteTable.RouteTable.Id)
```

2. 각 AZ에 2개의 서브넷을 생성한다.

```bash
SUBNET_ID_1=$(aws ec2 create-subnet --vpc-id $VPC_ID \
--cidr-block 10.10.0.0/24 --availability-zone ${AWS_REGION}a \
--tag-specifications \
'ResourceType=subnet,Tags=[{Key=Name,Value=AWSCookbook202a}]' \
--output text --query Subnet.SubnetId)

SUBNET_ID_2=$(aws ec2 create-subnet --vpc-id $VPC_ID \
--cidr-block 10.10.1.0/24 --availability-zone ${AWS_REGION}b \
--tag-specifications \
'ResourceType=subnet,Tags=[{Key=Name,Value=AWSCookbook202a}]' \
--output text --query Subnet.SubnetId)
```

3. 라우팅 테이블을 2개의 서브넷과 연결한다.

```bash
aws ec2 associate-route-table \
--route-table-id $ROUTE_TABLE_ID --subnet-id $SUBNET_ID_1

aws ec2 associate-route-table \
--route-table-id $ROUTE_TABLE_ID --subnet-id $SUBNET_ID_2
```

4. 각 describe-subnets 명령에서 다음과 같은 유사한 출력을 확인할 수 있다.

```bash
aws ec2 describe-subnets --subnet-ids $SUBNET_ID_1
aws ec2 describe-subnets --subnet-ids $SUBNET_ID_2
```

5. 두 서브넷이 라우팅 테이블과 연결돼 있는지 확인한다.

```bash
aws ec2 describe-route-tables --route-table-ids $ROUTE_TABLE_ID
```

### 참고

서브넷 설계할 때 현재 요구사항에 맞는 서브넷 크기를 선택하고 애플리케이션의 확장을 고려해야 한다.
AWS는 리소스에 대한 탄력적 네트워크 인터페이스 배치를 위해 서브넷을 사용한다. (이는 특정 ENI는 단일 AZ내에 존재한다는 것을 의미한다.)
AWS는 모든 서브넷 CIDR 블록의 처음 4개와 마지막 IP주소를 특별한 기능을 위해 사용한다.

서브넷은 하나의 라우팅 테이블을 가진다.
라우팅 테이블은 하나 이상의 서브넷과 연결될 수 있으며 선택한 대상으로 트래픽을 보낼 수 있다.
라우팅 테이블 내의 항목을 경로라고 하며, 대상과 대상 쌍으로 정의한다.

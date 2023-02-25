# 2.2 서브넷과 라우팅 테이블을 포함한 네트워크 티어 생성

> _VPC 내에 라우팅 테이블 생성하고 서브넷 연결_

#### ① 라우팅 테이블 생성

```bash
ROUTE_TABLE_ID=$(aws ec2 create-route-table --vpc-id $VPC_ID \
  --tag-specifications \
	'ResourceType=route-table,Tags=[{Key=Name,Value=AWSCookbook202}]' \
	--output text --query RouteTable.RouteTableId)
```

#### ② 각 AZ에 2개의 서브넷 생성

```bash
# ap-northeast-2a
SUBNET_ID_1=$(aws ec2 create-subnet --vpc-id $VPC_ID \
	--cidr-block 10.10.0.0/24 --availability-zone ${AWS_REGION}a \
	--tag-specifications \
	'ResourceType=subnet,Tags=[{Key=Name,Value=AWSCookbook202a}]' \
	--output text --query Subnet.SubnetId)

# ap-northeast-2c
SUBNET_ID_2=$(aws ec2 create-subnet --vpc-id $VPC_ID \
	--cidr-block 10.10.1.0/24 --availability-zone ${AWS_REGION}c \
	--tag-specifications \
	'ResourceType=subnet,Tags=[{Key=Name,Value=AWSCookbook202c}]' \
	--output text --query Subnet.SubnetId)

# AZ ID 찾는 명령어
aws ec2 describe-availability-zones --region $AWS_REGION

# availabilityZone에 대해 InvalidParameterValue 에러 출력 시, region 다시 지정
export AWS_REGION=ap-northeast-2
```

#### ③ 라우팅 테이블과 2개의 서브넷 연결

```bash
aws ec2 associate-route-table \
	--route-table-id $ROUTE_TABLE_ID --subnet-id $SUBNET_ID_1

aws ec2 associate-route-table \
	--route-table-id $ROUTE_TABLE_ID --subnet-id $SUBNET_ID_2
```

---

**🥕 유효성 검사** : 동일 VPC의 두 서브넷이 다른 AZ에 위치하는지 확인

```bash
aws ec2 describe-subnets --subnet-ids $SUBNET_ID_1
aws ec2 describe-subnets --subnet-ids $SUBNET_ID_2
```

**🥕 참고**

⍢ ENI 배치를 위해 서브넷 사용

- 특정 ENI는 단일 AZ 내에 존재함을 의미

⍢ 라우팅 테이블은 하나 이상의 서브넷과 연결하여 선택 대상으로 트래픽 전송

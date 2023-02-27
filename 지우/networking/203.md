# 2.3 인터넷 게이트웨이를 사용해 VPC를 인터넷에 연결

> _EC2 인스턴스의 서브넷에서 인터넷 게이트웨이로 트래픽을 보내는 경로 추가; EIP 생성_

#### ① 키페어와 EC2 인스턴스 생성

- EC2 인스턴스는 하나의 서브넷에 하나만 생성 (test)

```bash
# key pair
KEY_PAIR=$(aws ec2 create-key-pair \
	--key-name AWSCookbookKey --query 'AWSCookbookKey' \
	--output text > AWSCookbookKey.pem)

# 가장 최신 ami id 불러오는 명령어
aws ec2 describe-images \
    --owners amazon \
    --filters "Name=name,Values=amzn2-ami-hvm-2.0.*-x86_64-gp2" "Name=state,Values=available" \
    --query "reverse(sort_by(Images, &CreationDate))[:1].ImageId" \
    --output text

# ec2 인스턴스 생성
aws ec2 run-instances \
    --image-id ami-0e6496235b65306a3 \
    --instance-type t2.micro \
    --key-name AWSCookbookKey \
    --subnet-id $SUBNET_ID_1 \
    --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=AWSCookbookEC2}]'
```

#### ② 인터넷 게이트웨이(IGW) 생성

```bash
IGW_ID=$(aws ec2 create-internet-gateway \
	--tag-specifications \
	'ResourceType=internet-gateway,Tags=[{Key=Name,Value=AWSCookbook203}]' \
	--output text --query InternetGateway.InternetGatewayId)
```

#### ③ VPC에 IGW 연결

```bash
aws ec2 attach-internet-gateway \
	--internet-gateway-id $IGW_ID --vpc-id $VPC_ID
```

④ 각 라우팅 테이블의 기본 경로 대상을 IGW로 설정

```bash
aws ec2 create-route --route-table-id $ROUTE_TABLE_ID \
	--destination-cidr-block 0.0.0.0/0 --gateway-id $IGW_ID
```

#### ⑤ EIP 생성

```bash
ALLOCATION_ID=$(aws ec2 allocate-address --domain vpc \
	--output text --query AllocationId)
```

#### ⑥ EIP를 기존 EC2 인스턴스와 연결

```bash
# EC2 Instance ID 불러오는 명령어
aws ec2 describe-instances \
	--query 'Reservations[].Instances[].InstanceId' \
	--output text
```

```bash
# 'xx’을 실제 instance id값으로 대체
aws ec2 associate-address \
	--instance-id 'xx' --allocation-id $ALLOCATION_ID
```

---

**🥕 유효성 검사** : SSM Session Manager로 EC2 인스턴스에 연결

- IAM 역할 정의 및 연결 후 수행 가능 (1.6 참고)

```bash
aws ssm start-session --target $INSTANCE_ID

# 인터넷 연결 테스트
ping -c 4 homestarrunner.com

exit
```

**🥕 참고**

⍢ EIP를 생성함으로써 인스턴스와의 상호작용 필요없이 인터넷과 통신 가능

- 인스턴스에 EIP 연결하면 인스턴스 재부팅 후에도 IP는 변경되지 않음 → 고정 IP 역할

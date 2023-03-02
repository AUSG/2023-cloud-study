## 2.3 인터넷 게이트웨이를 사용해 VPC를 인터넷에 연결

### 문제 설명

VPC의 서브넷에서 기존에 실행 중인 EC2 인스턴스가 인터넷을 통해 클라이언트와 통신해야 한다.

### 해결 방법

1. 인터넷 게이트웨이를 생성해 VPC에 연결한다.
2. EC2 인스턴스의 서브넷에서 인터넷 게이트웨이로 트래픽을 보내는 경로를 추가한다.
3. 탄력적 IP주소(EIP)를 생성해 인스턴스와 연결한다.

### 준비 사항

- VPC에서 2개의 AZ에 배포된 서브넷과 라우팅 테이블
- 기존에 실행중인 EC2 인스턴스. 테스트를 위해 연결할 수 있어야 한다.

### 작업 방법

1. 인터넷 게이트웨이를 생성한다.

```bash
INET_GATEWAY_ID=$(aws ec2 create-internet-gateway \
--tag-specifications \
'ResourceType=internet-gateway,Tags=[{Key=Name,Value=AWSCookbook202}]' \
--output text --query InternetGateway.InternetGatewayId)
```

2. 인터넷 게이트웨이를 기존 VPC에 연결한다.

```bash
aws ec2 attach-internet-gateway \
--internet-gateway-id $INET_GATEWAY_ID --vpc-id $VPC_ID
```

3. VPC의 각 라우팅 테이블에서 기본 경로 대상을 인터넷 게이트웨이로 설정하는 경로를 생성한다.

```bash
aws ec2 create-route --route-table-id $ROUTE_TABLE_ID_1 \
--destination-cidr-block 0.0.0.0/0 --gateway-id $INET_GATEWAY_ID

aws ec2 create-route --route-table-id $ROUTE_TABLE_ID_2 \
--deestination-cidr-block 0.0.0.0/0 --gateway-id $INET_GATEWAY_ID
```

4. EIP 생성

```bash
ALLOCATION_ID=$(aws ec2 allocate-address --domain vpc \
--output text --query AllocationId)
```

5. EIP를 기존 EC2 인스턴스와 연결한다.

```bash
aws ec2 associate-address \
--instance-id $INSTANCE_ID --allocation-id $ALLOCATION_ID
```

### 유효성 검사

- SSM을 사용해 EC2 인스턴스에 연결한다.

```bash
aws ssm start-session --target $INSTANCE_ID
```

- 인터넷 연결 테스트

```bash
ping -c 4 homestarrunner.com
```

- Session Manager 세선 종료

```bash
exit
```

### 참고

레시피에서 생성한 경로를 통해 모든 외부 트래픽을 VPC의 인터넷 연결을 제공하는 IGW로 보낸다.
이미 실행중인 인스턴스로 작업했기 때문에, EIP를 생성하고 연결했다.
=> 인스턴스와 상호작용 없이 인터넷 통신 가능하도록 설정

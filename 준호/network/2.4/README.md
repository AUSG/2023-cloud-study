## 2.4 NAT 게이트웨이 사용한 프라이빗 서브넷의 외부 인터넷 접근

### 문제 설명

IGW에 대한 경로를 가진 퍼블릭 서브넷을 활용해 프라이빗 서브넷의 인스턴스에 외부 인터넷 접근을 허용해야 한다.

### 해결 방법

1. 퍼블릭 서브넷 중 하나에 NAT 게이트웨이를 생성한다.
2. EIP 주소를 생성하고 NAT 게이트웨이와 연결한다.
3. 프라이빗 서브넷과 연결한 라우팅 테이블에 NAT 게이트웨이를 대상으로 하는 인터넷 트래픽에 대한 경로를 추가한다.

### 작업 방법

1. NAT 게이트웨이에 사용할 EIP 주소를 생성한다.

```bash
ALLOCATION_ID=$(aws ec2 allocate-address --domain vpc \
--output text --query AllocationId)
```

2. AZ1의 퍼블릭 서브넷 내에 NAT 게이트웨이를 생성한다.

```bash
NAT_GATEWAY_ID=$(aws ec2 create-nat-gateway \
--subnet-id $VPC_PUBLIC_SUBNET_1 \
--allocation-id $ALLOCATION_ID \
--output text --query NatGateway.NatGatewayId)
```

3. NAT 게이트웨이를 사용할 수 있게 되려면 몇 분 정도 소요된다. (아래 명령어로 상태 확인)

```bash
aws ec2 describe-nat-gateways \
--nat-gateway-ids $NAT_GATEWAY_ID \
--output text --query NatGateWays[0].State
```

4. NAT 게이트웨이의 0.0.0.0/0에 대한 기본 경로를 프라이빗 티어의 두 라우팅 테이블에 추가한다. (기본 경로: 모든 트래픽을 지정된 대상으로 보냄)

```bash
aws ec2 create-route --route-table-id $PRIVATE_RT_ID_1 \
--destination-cidr-block 0.0.0.0/0 \
--nat-gateway-id $NAT_GATEWAY_ID

aws ec2 create-route --route-table-id $PRIVATE_RT_ID_2 \
--destination-cidr-block 0.0.0.0/0 \
--nat-gateway-id $NAT_GATEWAY_ID
```

### 유효성 검사

- SSM을 이용해 EC2 인스턴스 1에 연결한다.

```bash
aws ssm start-session --target $INSTANCE_ID_1
```

- ping 날리기

### 참고

여기선 외부 인터넷의 아웃바운드 액세스를 허용하면서 내부의 액세스에 대한 인바운드 액세스는 허용하지 않는 서브넷 계층의 네트워크 아키텍처를 구성한다.

프라이빗 서브넷의 리소스에서 실행하는 서비스에 인터넷 인바운드 액세스를 허용하려면 퍼블릭 서브넷의 로드 밸런서를 사용해야 한다.

NAT 게이트웨이를 통과하는 모든 통신은 연결한 EIP 주소를 가진다.
예를 들어 외부 공급업체의 허용 목록에 IP를 추가해야 하는 경우 NAT 게이트웨이의 EIP를 화이트리스트로 등록할 수 있다.

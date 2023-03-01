## 2.11 VPC 간 네트워크 통신을 위한 VPC 피어링 적용

### 문제 설명

서로 다른 VPC에 있는 2개의 인스턴스를 간단하고 비용 효율적인 방식으로 통신하는 방법을 구성해야 한다.

### 해결 방법

두 VPC 간의 피어링을 요청하고 연결을 수락한다.
각 VPC 서브넷에 대한 라우팅 테이블을 업데이트하고, 한 인스턴스에서 다른 인스턴스로의 연결을 테스트 한다.

### 작업 방법

1. VPC1을 VPC2에 연결하는 VPC 피어링 연결 생성

```bash
VPC_PEERING_CONNECTION_ID=$(aws ec2 create-vpc-peering-connection \
--vpc-id $vpc_id_1 --peer-vpc-id $vpc_id_2 --output text \
--query VpcPeeringConnection.VpcPeeringConnectionId)
```

2. 피어링 연결 수락

```bash
aws ec2 accept-vpc-peering-connection --vpc-peering-connection-id $VPC_PEERING_CONNECTION_ID
# 다른 계정 간에도 피어링 연결이 가능함
```

3. 서브넷과 연결된 라우팅 테이블에 피어링된 VPC의 CIDR 범위로 향하는 트래픽을 VPC_PEERING_CONNECTION_ID로 보내는 경로를 추가한다.

```bash
aws ec2 create-route --route-table-id $vpc_subnet_rt_1 --destination-cidr-block 10.11.0.0/16 --vpc-peering-connection-id $VPC_PEERING_CONNECTION_ID

aws ec2 create-route --route-table-id $vpc_subnet_rt_2 --destination-cidr-block 10.10.0.0/16 --vpc-peering-connection-id $VPC_PEERING_CONNECTION_ID
```

4. 인스턴스 1의 보안 그룹의 ICMPv4 액세스를 허용하는 수신 규칙을 인스턴스 2의 보안 그룹에 추가한다.

```bash
aws ec2 authorize-security-group-ingress \
--protocol icmp --port -1 \
--source-group $INSTANCE_SG_1 \
--group-id $INSTANCE_SG_2
```

5. 인스턴스1과 인스턴스2이 연결되었는지 핑 테스트 한다.

```bash
ping -c 4 <INSTANCE_IP_2>
```

### 참고

VPC 피어링 연결은 전이되지 않으므로 서론 다른 VPC 간의 통신을 위해서는 모든 VPC와 피어링을 설정해야 한다.

VPC1 VPC2를 간의 연결은 피어링으로도 가능하지만(목적지 경로와 반환경로가 서로에게 존재해야 함) 세 번째 VPC를 추가하라면 구조가 더 복잡해지기 때문에 이땐 트랜짓 게이트웨이 아키텍처가 더 유요하다.

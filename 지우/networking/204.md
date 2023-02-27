# 2.4 NAT 게이트웨이를 사용한 프라이빗 서브넷의 외부 인터넷 접근

> _퍼블릭 서브넷에 NAT 게이트웨이 생성하고 EIP 연결 및 라우팅 테이블에 경로 추가_

- 프라이빗 서브넷과 그에 연결된 라우팅 테이블, 그리고 프라이빗 서브넷에 배포된 EC2 인스턴스 필요
  - NAT 게이트웨이가 존재하는 퍼블릭 서브넷: `PUBLIC_SBN_ID`
  - 프라이빗 서브넷: `PRIVATE_SBN_ID`
  - 라우팅 테이블: `PRIVATE_RT_ID`
  - 프라이빗 서브넷에 배포된 EC2 인스턴스: `PRIVATE_INSTANCE_ID`

#### ① NAT 게이트웨이를 위한 EIP 생성

```bash
ALLOCATION_ID_NAT=$(aws ec2 allocate-address --domain vpc \
	--output text --query AllocationId)
```

#### ② ap-northeast-2a AZ에 배포한 퍼블릭 서브넷에 NAT 게이트웨이 생성

```bash
NGW_ID=$(aws ec2 create-nat-gateway \
	--subnet-id $PUBLIC_SBN_ID \
	--allocation-id $ALLOCATION_ID_NAT \
	--output text --query NatGateway.NatGatewayId)

# NGW 상태 확인 (생성되는 데에 일정 시간 소요)
aws ec2 describe-nat-gateways \
	--nat-gateway-ids $NGW_ID \
	--output text
```

#### ③ 라우팅 테이블에 NGW 경로 추가

```bash
aws ec2 create-route --route-table-id $PRIVATE_RT_ID \
	--destination-cidr-block 0.0.0.0/0 \
	--nat-gateway-id $NGW_ID
```

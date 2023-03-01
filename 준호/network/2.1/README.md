## 2.1 AmazonVPC를 사용해 프라이빗 가상 네트워크 생성

### 문제 설명

클라우드 리소스를 호스팅할 네트워크를 구축해야 한다.

### 해결 방법

Amazon VPC를 생성하고, 이에 대한 CIDR블록을 구성한다.

#### 작업 방법

1. IPv4 CIDR 블록을 가진 VPC를 생성한다. 주소 범위로 10.10.0.0/16을 사용한다.
   주소 범위는 필요에 따라 다른 범위로 수정할 수 있다.

```bash
VPC_ID=$(aws ec2 create-vpc --cidr-block 10.10.0.0/16 \
--tag-specifications 'ResourceType=vpc,Tags=[{Key=Name,Value=AWSCookbook201}]' \
--output text --query Vpc.VpcId)
```

- 유효성 검사

```bash
aws ec2 describe-vpcs --vpc-ids $VPC_ID
```

### 참고

`aws ec2 Associate-vpc-cidrblock` 명령을 사용해 VPC에 IPv4 공간을 추가할 수 있다.
처음부터 큰 블록을 할당할 필요가 없다. 다음 명령어로 추가적인 IPv4 CIDR 블록을 VPC에 연결할 수 있다.

```bash
aws ec2 associate-vpc-cidr-block \
--cidr-block 10.11.0.0/16 \
--vpc-id $VPC_ID
```

VPC는 AWS의 각 리전에 귀속되는 서비스다.
리전은 지리적 영역이고 가용 영역은 지역 내에 있는 물리적 데이터 센터다.
리전은 격리된 물리적 데이터 센터의 그룹인 모든 가용 영역(AZ)에 걸쳐져 있다.
리전당 AZ수는 다양하지만 모든 리전은 최소 3개의 AZ를 가진다.

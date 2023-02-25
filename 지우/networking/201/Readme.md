# 2.1 Amazon VPC를 사용해 프라이빗 가상 네트워크 생성

> _VPC와 CIDR block 구성_

#### ① `10.10.0.0/16` CIDR을 가지는 VPC 생성

- VPC IPv4 CIDR의 가장 큰 블록 크기는 /16, 가장 작은 블록 크기는 /28 넷마스크

```bash
# 방법1. 태그와 함께 VPC 생성
VPC_ID=$(aws ec2 create-vpc --cidr-block 10.10.0.0/16 \
	--tag-specifications 'ResourceType=vpc,Tags=[{Key=Name,Value=AWSCookbook201}]' \
	--output text --query Vpc.VpcId)

# 방법2. VPC 생성 후 태그 추가
VPC_ID=$(aws ec2 create-vpc --cidr-block 10.10.0.0/16 \
  --output text --query Vpc.VpcId)

aws ec2 create-tags --resources $VPC_ID --tags Key=Name,Value=AWSCookbook201
```

---

**🥕 유효성 검사** : VPC 상태 확인

```bash
aws ec2 describe-vpcs --vpc-ids $VPC_ID
```

**🥕 참고**

⍢ VPC에 연결한 CIDR block은 확장 가능하지만, 수정은 불가능

- 추가적인 IPv4 CIDR 블록 연결 명령어
  ```bash
  aws ec2 associate-vpc-cidr-block \
    --cidr-block 10.11.0.0/16 \
    --vpc-id $VPC_ID
  ```

⍢ VPC 피어링 또는 게이트웨이를 통해 다른 네트워크에 연결할 시, IP 범위가 겹치면 문제 발생

⍢ 모든 리전은 최소 3개의 AZ를 가짐

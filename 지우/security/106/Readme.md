# 1.6 AWS SSM Session Manager를 사용해 EC2 인스턴스에 연결

> _ssh를 사용하지 않고 private subnet에 배포된 EC2 인스턴스에 연결_

<img src="https://user-images.githubusercontent.com/70079416/220594073-5848b059-b06a-4ae7-93f7-942ba1875534.png" width="60%" height="60%" />

#### ① `assign-role-policy.json` 생성

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "ec2.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

#### ② 위 파일로 IAM 역할 생성

```bash
ROLE_ARN=$(aws iam create-role --role-name AWSCookbook106SSMRole \
  --assume-role-policy-document file://assume-role-policy.json \
  --output text --query Role.Arn
```

#### ③ `AmazonSSMManagedInstanceCore` 정책 연결

```bash
aws iam attach-role-policy --role-name AWSCookbook106SSMRole \
  --policy-arn arn:aws:iam::aws:policy/AmazonSSMManagedInstanceCore
```

#### ④ 인스턴스 프로필 생성

```bash
aws iam create-instance-profile \
  --instance-profile-name AWSCookbook106InstanceProfile
```

#### ⑤ 앞서 생성한 역할을 위 프로필에 추가

```bash
aws iam add-role-to-instance-profile \
  --role-name AWSCookbook106SSMRole \
  --instance-profile-name AWSCookbook106InstanceProfile
```

#### ⑥ AWS SSM에서 해당 리전에서 사용 가능한 최신 Amazon Linux2 AMI ID를 찾아 환경변수로 저장

```bash
AMI_ID=$(aws ssm get-parameters --names \
  /aws/service/ami-amazon-linux-latest/amzn2-ami-hvm-x86_64-gp2 \
  --query 'Parameters[0].[Value]' --output text)
```

#### ⑦ 서브넷 중 하나에서 생성한 인스턴스 프로필을 참고하는 인스턴스 시작

- 식별을 위한 Name 태그 지정

```bash
INSTANCE_ID=$(aws ec2 run-instances --image-id $AMI_ID \
     --count 1 \
     --instance-type t3.nano \
     --iam-instance-profile Name=AWSCookbook106InstanceProfile \
     --subnet-id $SUBNET_1 \
     --security-group-ids $INSTANCE_SG \
     --metadata-options \
HttpTokens=required,HttpPutResponseHopLimit=64,HttpEndpoint=enabled \
     --tag-specifications \
     'ResourceType=instance,Tags=[{Key=Name,Value=AWSCookbook106}]' \
     'ResourceType=volume,Tags=[{Key=Name,Value=AWSCookbook106}]' \
     --query Instances[0].InstanceId \
     --output text)
```

---

**🥕 유효성 검사** : EC2 인스턴스가 SSM에 등록됐는지 확인

```bash
aws ssm describe-instance-information \
  --filters Key=ResourceType, Values=EC2Instance \
  --query "InstanceInformationList[].InstanceId" --output text
```

⍢ SSM Session Manager로 EC2 인스턴스에 연결

```bash
aws ssm start-session --target $INSTANCE_ID
```

- 연결하면 bash prompt가 뜬다. 여기서 IMDSv2 토큰을 받고 이 토큰으로 인스턴스와 연결된 인스턴스 프로필에 대한 메타데이터를 쿼리하여 EC2 인스턴스에 연결됐는지 확인한다.

```bash
TOKEN=`curl -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 21600"`
curl -H "X-aws-ec2-metadata-token: $TOKEN" http://169.254.169.254/latest/meta-data/iam/info
```

**🥕 참고**

⍢ SSM을 사용하면 SSH 연결이 필요 없음
⍢ SSM은 세션의 모든 명령과 해당 출력 로깅

- 민감데이터는 로깅을 중지하도록 설정 가능
  ```bash
  stty -echo; read passwd; stty echo;
  ```

⍢ HTTPS를 통해 사용 중인 리전 내 SSM API 엔드포인트와 통신

- 인스턴스 부팅 시, 에이전트를 통해 SSM 서비스에 등록
- 인바운드 보안 그룹 규칙 수정 필요 없음
- VPC 엔드포인트로 NGW 비용과 인터넷 트래픽을 피할 수 있음

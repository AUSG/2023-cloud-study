## 1.6 AWS SSM Session Manager를 사용해 EC2 인스턴스에 연결

ssh를 사용하지 않고 배포된 EC2 인스턴스에 연결해요

### 단계

0. 준비단계

```bash
cd 106-Connecting-to-EC2-Instances-Using-Session-Manager/cdk-AWS-Cookbook-106/
test -d .venv || python3 -m venv .venv
source .venv/bin/activate
pip install --upgrade pip
pip install -r requirements.txt
cdk deploy # aws cdk npm으로 설치
```

1. assert-role-policy.json을 사용해 IAM 역할 생성

```bash
ROLE_ARN=$(aws iam create-role --role-name AWSCookbook106SSMRole --assume-role-policy-document file://assume-role-policy.json --output text --query Role.Arn)
```

2. 생성한 역할에 AmazonSSMManagedInstanceCore 관리형 정책 연결

```bash
aws iam attach-role-policy --role-name AWSCookbook106SSMRole --policy-arn arn:aws:iam::aws:policy/AmazonSSMManagedInstanceCore
```

3. 인스턴스 프로필 생성

```bash
aws iam create-instance-profile --instance-profile-name AWSCookbook106InstanceProfile
```

4. 생성한 역할을 인스턴스 프로필에 추가
   인스턴스 프로필은 사용자가 생성한 역할을 포함해요.
   인스턴스와 인스턴스 프로필 연결을 통해 "내가 누구인지"를 정의하고, 역할은 "내가 할 수 있는 일" 을 정의해요.

```bash
aws iam add-role-to-instance-profile --role-name AWSCookbook106SSMRole --instance-profile-name AWSCookbook106InstanceProfile

# 인스턴스 리스트 조회
aws iam list-instance-profiles
```

5.

```bash
AMI_ID=$(aws ssm get-parameters --names /aws/service/ami-amazon-linux-latest/amzn2-ami-hvm-x86_64-gp2 --query 'Parameters[0].[Value]' --output text)
```

# 1.4 IAM 정책 시뮬레이터를 사용해서 IAM 정책 테스트

> _IAM 정책 시뮬레이터로 IAM 정책의 영향 범위 확인_

<img src="https://user-images.githubusercontent.com/70079416/220561917-1d8ec0b6-2bdd-4098-9baa-3ce240ee7bdf.png" width="60%" height="60%" />

#### ① `assign-role-policy.json` 파일 생성

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

#### ② 앞서 생성한 정책 파일로 IAM 역할 생성

```bash
aws iam create-role --assume-role-policy-document \
  file://assume-role-policy.json --role-name AWSCookbook104IamRole
```

#### ③ 앞서 생성한 IAM 역할에 관리형 정책인 `AmazonEC2ReadOnlyAccess` 연결

```bash
aws iam attach-role-policy --role-name AWSCookbook104IamRole \
  --policy-arn arn:aws:iam::aws:policy/AmazonEC2ReadOnlyAccess
```

---

**🥕 유효성 검사** : IAM 정책 효과 시뮬레이션 후 EC2 서비스에 대한 여러 작업 실행

⍢ `ec2:CreateInternetGateway` 작업은 거부, `ec2:DescribeInstances` 작업은 허용해야 함

```bash
aws iam simulate-principal-policy \
  --policy-source-arn arn:aws:iam::$AWS_ACCOUNT_ARN:role/AWSCookbook104IamRole \
  --action-names ec2:CreateInternetGateway

# EvalDecision output
{
  "EvaluationResults": [
    {
      ...
      "EvalDecision": "implicitDeny",
      ...
    }
  ]
}
```

```bash
aws iam simulate-principal-policy \
  --policy-source-arn arn:aws:iam::$AWS_ACCOUNT_ARN:role/AWSCookbook104IamRole \
  --action-names ec2:DescribeInstances

# EvalDecision output
{
  "EvaluationResults": [
    {
      ...
      "EvalDecision": "allowed",
      ...
    }
  ]
}
```

**🥕 참고**

⍢ **IAM 정책 시뮬레이터**는 최소 권한 액세스에 대한 자체 IAM 정책을 설계하고 관리할 때 매우 유용

- ID 기반 정책, IAM 권한 경계, AWS Organizations 서비스 제어 정책(SCP), 리소스 기반 정책

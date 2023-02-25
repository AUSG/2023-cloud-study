# 1.8 Secrets Manager를 사용해 암호 저장, 암호화, 액세스

> _EC2 인스턴스가 데이터베이스 암호를 안전하게 저장하고 가져올 수 있도록 설정_

- Secrets Manager에 암호를 저장
- 이 암호에 대한 액세스 권한을 가진 IAM 정책 생성
- EC2 인스턴스 프로필에 암호에 대한 액세스 권한 부여
- 사전준비; VPC, Subnet, Routing Table, (이미 배포된) EC2 인스턴스

#### ① 보안 암호 생성

```bash
RANDOM_STRING=$(aws secretsmanager get-random-password \
  --password-length 32 --require-each-included-type \
  --output text \
  --query RandomPassword)
```

#### ② Secrets Manager에 새 암호로 저장

```bash
SECRET_ARN=$(aws secretsmanager \
  create-secret --name AWSCookbook108/Secret1 \
  --description "AWSCookbook108 Secret 1" \
  --secret-string $RANDOM_STRING \
  --output text \
  --query ARN)
```

#### ③ 생성한 암호를 참고하여 `secret-access-policy-template.json` 생성

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "secretsmanager:GetResourcePolicy",
        "secretsmanager:GetSecretValue",
        "secretsmanager:DescribeSecret",
        "secretsmanager:ListSecretVersionIds"
      ],
      "Resource": ["SECRET_ARN"]
    },
    {
      "Effect": "Allow",
      "Action": "secretsmanager:ListSecrets",
      "Resource": "*"
    }
  ]
}
```

#### ④ sed 명령어로 `SECRET_ARN`을 교체한 `secret-access-policy.json` 생성

```bash
sed -e "s|SECRET_ARN|$SECRET_ARN|g" \
  secret-access-policy-template.json > secret-access-policy.json
```

#### ⑤ 보안 암호 액세스를 위한 IAM 정책 생성

```bash
aws iam create-policy --policy-name AWSCookbook108SecretAccess \
  --policy-document file://secret-access-policy.json
```

#### ⑥ 앞서 생성한 정책을 EC2 인스턴스 프로필의 IAM 역할에 추가하여 EC2 인스턴스가 보안 암호에 액세스할 수 있는 권한을 부여

```bash
aws iam attach-role-policy --policy-arn \
  arn:aws:iam::$AWS_ACCOUNT_ID:policy/AWSCookbook108SecretAccess \
  --role-name $ROLE_NAME
```

---

**🥕 유효성 검사**

```bash
# EC2 인스턴스에 연결
aws ssm start-session --target $INSTANCE_ID

# 기본 리전 설정
export AWS_DEFAULT_REGION=ap-northeast-2

# secrets manager에서 암호 가져오기
aws secretsmanager get-secret-value --secret-id AWSCookbook108/Secret1

# 종료
exit
```

**🥕 참고**

⍢ EC2 인스턴스를 보안 암호에 액세스하기 위한 자격증명을 저장할 필요 없음!

- 인스턴스 프로필이 있기 때문
- 인스턴스 프로필에 연결된 IAM 정책을 통해 액세스 권한 부여받음

⍢ Secrets Manager 장점

- 고객 관리형 KMS 키로 암호화
- CloudTrail로 보안 암호 액세스 감사
- 람다로 보안 암호 교체 자동화
- 다른 사용자나 역할, EC2, 람다 서비스에 대한 액세스 권한 부여
- 고가용성 및 DR을 위한 타 리전 비밀 복제 지원

# 1.1 개발자 접근을 위한 IAM 역할 생성과 수임

> _IAM 정책을 사용해 수임 가능한 역할을 생성하고 관리형 PowerUserAccess IAM 정책을 역할에 연결_

#### ① `assume-role-policy-template.json` 생성

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "PRINCIPAL_ARN"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

#### ② 사용자의 ARN을 환경 변수로 설정

```bash
PRINCIPAL_ARN=$(aws sts get-caller-identity --query Arn --output text)
```

#### ③ sed 명령어로 위 파일의 `PRINCIPAL_ARN` 변수를 치환하여 `assume-role-policy.json` 파일 생성

```bash
sed -e "s|PRINCIPAL_ARN|${PRINCIPAL_ARN}|g" \
assume-role-policy-template.json > assume-role-policy.json
```

#### ④ 위 정책 파일로 `AWSCookbook101Role` 역할 생성

```bash
ROLE_ARN=$(aws iam create-role --role-name AWSCookbook101Role \
--assume-role-policy-document file://assume-role-policy.json \
--output text --query Role.Arn
```

#### ⑤ `PowerUserAccess` 정책을 역할에 연결

```bash
aws iam attach-iam-policy --role-name AWSCookbook101Role \
--policy-arn arn:aws:iam::aws:policy/PowerUserAccess
```

---

**🥕 유효성 검사** : 앞서 생성한 역할 수임

```bash
aws sts assume-role --role-arn $ROLE_ARN \
--role-session-name AWSCookbook101
```

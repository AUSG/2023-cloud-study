# 1.5 권한 경계를 사용한 IAM 관리 기능 위임

> _람다 함수가 사용할 수 있는 IAM 역할을 생성할 수 있도록 권한 부여_

- 권한 경계 정책을 설정하여 람다 개발자를 위한 IAM 역할을 생성
- 경계 정책을 지정하는 IAM 정책 생성 후 역할에 정책 연결
- ⓪ 과정은 **1.1**과 동일

#### ⓪-1. `assign-role-policy-template.json` 생성

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

#### ⓪-2. 사용자의 ARN을 환경 변수로 설정

```bash
PRINCIPAL_ARN=$(aws sts get-caller-identity --query Arn --output text)
```

#### ⓪-3. sed 명령어로 위 파일의 `PRINCIPAL_ARN` 변수를 치환하여 `assume-role-policy.json` 파일 생성

```bash
# sed 명령어 사용
sed -e "s|PRINCIPAL_ARN|${PRINCIPAL_ARN}|g" \
  assume-role-policy-template.json > assume-role-policy.json
```

#### ⓪-4. 앞서 생성한 역할 수임 정책을 사용할 새 역할 생성하고 ARN을 환경 변수로 저장

```bash
ROLE_ARN=$(aws iam create-role --role-name AWSCookbook105Role \
  --assume-role-policy-document file://assume-role-policy.json \
  --output text --query Role.Arn
```

#### ① DynamoDB, S3, CloudWatch Logs 작업을 허용하는 `boundary-policy-template.json` 파일 생성

#### ② sed 명령어로 위 파일의 `PRINCIPAL_ARN` 교체하고 `boundary-policy.json` 파일 생성

```bash
sed -e "s|AWS_ACCOUNT_ID|${AWS_ACCOUNT_ID}|g" \
  boundary-policy-template.json > boundary-policy.json
```

#### ③ 권한 경계 정책 생성

```bash
aws iam create-policy --policy-name AWSCookbook105PB \
  --policy-document file://boundary-policy.json
```

#### ④ `policy-template.json` 파일 생성

- `DenyPB`, `IAMRead`, `ServerlessFullAccess` 등의 권한 포함

#### ⑤ sed 명령어로 위 json 파일의 `AWS_ACCOUNT_ID` 변수를 치환하여 `policy.json` 파일 생성

```bash
sed -e "s|AWS_ACCOUNT_ID|${AWS_ACCOUNT_ID}|g" \
  policy-template.json > policy.json
```

#### ⑥ 개발자를 위한 정책 생성

```bash
aws iam create-policy --policy-name AWSCookbook105Policy \
  --policy-document file://policy.json
```

#### ⑦ 역할에 정책 연결

```bash
aws iam attach-role-policy --policy-arn \
  arn:aws:iam::$AWS_ACCOUNT_ID:policy/AWSCookbook105Policy \
  --role-name AWSCookbook105Role
```

---

**🥕 유효성 검사**

⍢ 람다 함수에 대한 IAM 역할 생성을 시도하고 역할 수임 정책 생성 ; `lambda-assume-role-policy.json`

⍢ 정책이 지정하는 명명 표준을 준수하는 이름으로 권한 경계를 지정하여 역할 생성

```bash
TEST_ROLE_1=$(aws iam create-role --role-name AWSCookbook105test1 \
  --assume-role-policy-document \ file://lambda-assume-role-policy.json \
  --permissions-boundary \ arn:aws:iam::$AWS_ACCOUNT_ID:policy/AWSCookbook105PB \
  --output text --query Role.Arn)
```

⍢ 관리형 정책 `AmazonDynamoDBFullAccess`, `CloudWatchFullAccess` 연결

- 이 권한은 앞서 생성한 권한 경계 명령문에 의해 제한

**⇒ 역할 정책에서 정의되지 않은 권한은 경계 정책에 정의돼 있어도 유효하지 않다‼️**

```bash
aws iam attach-role-policy --role-name AWSCookbook105test1 \
  --policy-arn arn:aws:iam::aws:policy/AmazonDynamoDBFullAccess

aws iam attach-role-policy --role-name AWSCookbook105test1 \
  --policy-arn arn:aws:iam::aws:policy/CloudWatchFullAccess
```

⍢ IAM 정책 시뮬레이터로 테스트

```bash
aws iam simulate-principal-policy \
  --policy-source-arn arn:aws:iam::$AWS_ACCOUNT_ARN:role/AWSCookbook105test1 \
  --action-names {policy_to_test}
```

**🥕 참고**

⍢ 권한 경계는 생성된 역할이 수행할 수 있는 작업을 정의해 위임된 관리자가 생성한 IAM 보안 주체의 최대 유효 권한을 제한

- 폭넓은 액세스 권한을 부여할 필요 없고, 권한 상승을 방지
- 팀 구성원이 애플리케이션에 대한 최소 권한 역할을 빠르고 반복적으로 생성
- AWSCookbook\*과 같은 명명 규칙을 설정하여 이 규칙을 준수하는 역할만 서비스에 전달할 수 있도록 설정

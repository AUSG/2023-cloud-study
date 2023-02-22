## 1.5 권한 경계를 사용한 IAM 관리 기능 위임

### 문제 설명

팀원에게 IAM 역할은 람다 함수가 필요한 작업만 허용하는 IAM 역할을 생성할 수 있도록 권한을 부여해요.

### 해결 방법

- 권한 경계 정책을 생성 > 람다 개발자를 위한 IAM 역할 생성
- 경계 정책을 지정하는 IAM 정책 생성 > 생성한 역할에 정책 연결

1. 사용자의 ARN 환경변수로 설정

```bash
PRINCIPAL_ARN=$(aws sts get-caller-identity --query Arn --output text)
```

2. assume-role-policy.json 생성

```bash
sed -e "s|PRINCIPAL_ARN|${PRINCIPAL_ARN}|g" assume-role-policy-template.json > assume-role-policy.json
```

3. 새 역할 생성한 뒤 ARN을 환경 변수로 저장

```bash
ROLE_ARN=$(aws iam create-role --role-name AWSCookbook105Role --assume-role-policy-document  file://assume-role-policy.json --output text --query Role.Arn)
```

4. 권한 경계를 지정하는 파일 생성 (레포의 레시는 DynamoDB, S3, CloudWatch Logs 작업 허용)

5. boundary-policy-template.json 생성, 환경변수 설정

```bash
sed -e "s|AWS_ACCOUNT_ID|${AWS_ACCOUNT_ID}|g" \ boundary-policy-template.json > boundary-policy.json
```

6. cli로 권한 경계 정책을 생성

```bash
aws iam create-policy --policy-name AWSCookbook105PB --policy-document file://boundary-policy.json
```

7. 저장소의 template을 이용해 policy.json 파일 생성

```bash
sed -e "s|AWS_ACCOUNT_ID|${AWS_ACCOUNT_ID}|g" policy-template.json > policy.json
```

8. 개발자를 위한 정책 생성

```bash
aws iam create-policy --policy-name AWSCookbook105Policy --policy-document file://policy.json
```

9. 위에서 생성한 역할에 정책을 연결

```bash
aws iam attach-role-policy --policy-arn arn:aws:iam::$AWS_ACCOUNT_ID:policy/AWSCookbook105Policy --role-name AWSCookbook105Role
```

### 유효성 검사

```bash
creds=$(aws --output text sts assume-role --role-arn $ROLE_ARN --role-session-name "AWSCookbook105" | grep CREDENTIALS | cut -d " " -f2,4,5)
export AWS_ACCESS_KEY_ID=$(echo $creds | cut -d " " -f2)
export AWS_SECRET_ACCESS_KEY=$(echo $creds | cut -d " " -f4)
export AWS_SESSION_TOKEN=$(echo $creds | cut -d " " -f5)
```

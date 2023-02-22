### 로컬 환경변수 설정

자주 사용되는 환경 변수들을 미리 설정하는 과정이 필요해요!

1. echo 명령어: 문자열을 출력하는 데 사용돼요.

   ```
   문법: echo [문자열]
   예시: echo "Hello, world!"
   ```

2. export 명령어: 쉘 변수를 환경 변수로 설정하는 데 사용돼요.

   ```
   문법: export [변수 이름]=[값]
   예시: export MY_VAR="hello"
   ```

3. env 명령어: 현재 쉘에서 사용 가능한 모든 환경 변수를 나열합니다.
   ```
   문법: env
   예시: env
   ```

```bash
PRINCIPAL_ARN=$(aws sts get-caller-identity --query Arn --output text --profile ausg-jun)
AWS_ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text --profile ausg-jun)
```

로컬 환경변수를 확인해요

```bash
echo $PRINCIPAL_ARN
# arn:aws:iam::515586596370:root
```

### 프로필 설정

회사 sso 로그인을 위한 프로필이 이미 세팅되어 있어서, 아래와 같이 프로필을 추가했어요.

```bash
aws configure --profile <profile_name>
```

결과

```bash
# ~/.aws/config
[profile junho]
region = us-east-1
output = json
```

### 기본 프로필 변경

```bash
export AWS_DEFAULT_PROFILE=<myprofile>
```

## 1.1 IAM 역할 생성과 사용자 수임

assume-role-policy-template.json 파일을, 내 arn의 정보로 교체하는 cli

```bash
sed -e "s|PRINCIPAL_ARN|${PRINCIPAL_ARN}|g" assume-role-policy-template.json > assume-role-policy.json
```

`assume-role-policy.json` 이라는 파일에 적어둔 arn을 불러와서 환경변수로 저장

```bash
ROLE_ARN=$(aws iam create-role --role-name AWSCookbook101Role --assume-role-policy-document  file://assume-role-policy.json --output text --query Role.Arn --profile junho)
```

```bash
aws iam attach-role-policy --role-name AWSCookbook101Role --policy-arn arn:aws:iam::aws:policy/PowerUserAccess --profile junho
```

## 1.2 액세스 패턴을 기반으로 최소 권한 IAM 권한 생성

IAM Access Analyzer를 사용해 AWS 계정의 Cloud Trail 활동을 기반으로 정책 생성

1. CloudTrail 서비스 접속, 추적 생성
2. IAM 사용자 > 페이지 최하단 **정책 생성** 버튼 클릭
3. 리전을 선택하고 최종 생성

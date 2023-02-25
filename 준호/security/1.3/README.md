## 1.3 AWS 계정의 IAM 사용자 암호 정책 시행

AWS 계정 내 사용자에 대해 암호 정책을 시행해야 할 경우, IAM 사용자에 대한 암호 정책을 설정해야 해요.

1. 소문자, 대문자, 기호, 숫자를 요구하도록(최소 32자, 최대 사용기간 90일, 비밀번호 재사용 방지 포함) IAM 암호 정책을 설정

```bash
aws iam update-account-password-policy --minimum-password-length 32 --require-symbols --require-numbers --require-uppercase-characters --require-lowercase-characters --allow-users-to-change-password --max-password-age 90 --password-reuse-prevention true
```

2. IAM 그룹을 생성해요.

```bash
aws iam create-group --group-name AWSCookbook103Group
```

3. 그룹에 정책을 연결해요.

```bash
aws iam attach-group-policy --group-name AWSCookbook103Group --policy-arn arn:aws:iam::aws:policy/AWSBillingReadOnlyAccess
```

사용자가 늘어날 수록 사용자에게 직접 권한을 따로 부여하는 것 보다, 그룹에 권한을 위임하여 사용자의 역할에 따라 맞는 그룹에 배정하는 것이 좋음

4. IAM 사용자를 생성해요.

```bash
aws iam create-user --user-name awscookbook103user
```

5. 암호를 생성해요.

```bash
RANDOM_STRING=$(aws secretsmanager get-random-password --password-length 32 --require-each-included-type --output text --query RandomPassword)
```

6. 생성한 암호를 바탕으로 로그인 프로필 생성

```bash
aws iam create-login-profile --user-name awscookbook103user --password $RANDOM_STRING
```

7. 정책을 연결한 IAM 그룹에 사용자를 추가해요.

```bash
aws iam add-user-to-group --group-name AWSCookbook103Group --user-name awscookbook103user
```

다음의 명령어로 비밀번호 정책 확인 가능

```bash
aws iam get-account-password-policy
# iam-user-password-policy.json에 결과물 저장해뒀음
```

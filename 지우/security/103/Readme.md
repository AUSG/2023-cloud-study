# 1.3 AWS 계정의 IAM 사용자 암호 정책 시행

> _AWS 계정의 IAM 사용자에 대한 암호 정책 설정_

- 이미 존재하는 사용자 디렉터리를 가진 조직은, ID 연합 활용이 가능한 AWS SSO 사용 추천을 권장
  - ID 연합으로 IdP(ID 공급자)의 사용자 및 그룹 사용 가능

#### ① IAM 암호 정책 설정

> 최소 길이, 최대 비밀번호 사용기간, 비밀번호 재사용 방지 등의 조건 포함

```bash
aws iam update-account-password-policy \
  --minimum-password-length 32 \
  --require-symbols \
  --require-numbers \
  --require-uppercase-characters \
  --require-lowercase-characters \
  --allow-users-to-change-password \
  --max-password-age 90 \
  --password-reuse-prevention true
```

#### ② IAM 그룹 생성

```bash
aws iam create-group --group-name AWSCookbook103Group
```

#### ③ 그룹에 `AWSBillingReadOnlyAccess` 정책 연결

- 정책은 사용자보다 그룹에 연결하는 것이 바람직함

```bash
aws iam attach-group-policy --group-name AWSCookbook103Group \
  --policy-arn arn:aws:iam::aws:policy/AWSBillingReadOnlyAccess
```

#### ④ IAM 사용자 생성

```bash
aws iam create-user --user-name awscookbook103user
```

#### ⑤ Secrets Manager로 암호 정책을 준수하는 암호 생성

```bash
RANDOM_STRING=$(aws secretsmanager get-random-password \
  --password-length 32 --require-each-included-type \
  --output text \
  --query RandomPassword)
```

#### ⑥ 방금 생성한 암호를 사용하는 로그인 프로필 생성

```bash
aws iam create-login-profile --user-name awscookbook103user \
  --password $RANDOM_STRING
```

#### ⑦ `AWSBillingReadOnlyAccess` 정책 연결한 그룹에 사용자 추가

```bash
aws iam add-user-to-group --group-name AWSCookbook103Group \
  --user-name awscookbook103user
```

---

**🥕 유효성 검사** : 암호 정책이 활성화되어 있는지 확인

```bash
aws iam get-account-password-policy
```

- 암호 정책을 위반하는 암호 생성 시, 로그인 프로필 생성 과정에서 `PasswordPolicyViolation` 에러 출력

**🥕 참고**

- MFA 인증 사용을 권장

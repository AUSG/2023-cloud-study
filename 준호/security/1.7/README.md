## 1.7 KMS 키를 사용해 EBS 볼륨 암호화

EC2에 연결된 EBS 볼륨 암호화, 암호화 키를 자동으로 교체해야 해요.

### 해결 방법

1. 고객 관리형 KMS키를 생성하고, 키의 ARN을 로컬 변수로 저장(accessDeniedException 😭)

```
# An error occurred (AccessDeniedException) when calling the CreateKey operation: User: arn:aws:iam::515586596370:user/ausg-jun is not authorized to perform: kms:CreateKey on resource: * because no identity-based policy allows the kms:CreateKey action
```

```bash
KMS_KEY_ID=$(aws kms create-key --description "AWSCookbook107Key" --output text --query KeyMetadata.KeyId)
```

2. (키 별칭 생성)

3. 대칭 키 구성 요소의 자동 교체를 365일로 활성화

```bash
aws kms enable-key-rotation --key-id $KMS_KEY_ID
```

4. 현재 리전 내의 EC2 서비스의 EBS 암호화 활성화

```bash
aws ec2 enable-ebs-encryption-by-default
```

### (참고)

`ebs-encryption-by-default` 옵션을 키면 새로 생성하는 모든 EBS 볼륨을 암호화 할 수 있음.

KMS는 다양한 데이터 암호화 전략을 유연하게 구현할 수 있는 서비스며, 키에 대한 액세스 권한을 가진 사용자를 제어하고자 키에 대한 정책을 지원함.(기존 IAM 정책과 함께 사용 가능)

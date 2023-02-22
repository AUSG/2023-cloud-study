# 1.7 KMS 키를 사용해 EBS 볼륨 암호화

> _EC2 인스턴스에 연결된 EBS 볼륨을 암호화하고, 365일 주기로 자동 교체해야 한다_

- 고객 관리형 KMS 키(CMK)를 생성하고 연간 교체 활성화
- EBS 볼륨에 대한 암호화 활성화하고 CMK를 KMS 키로 설정

#### ① CMK 생성하고 그 키의 ARN을 로컬 변수로 저장

```bash
KMS_KEY_ID=$(aws kms create-key --description "AWSCookbook107Key" \
  --output text --query KeyMetadata.KeyId)
```

#### ② 키에 대한 참고를 쉽게 할 수 있도록 키 별칭 생성

```bash
aws kms create-alias --alias-name alias/AWSCookbook107Key \
  --target-key-id $KMS_KEY_ID
```

#### ③ 대칭 키 구성 요소의 자동교체를 365일로 활성화

```bash
aws kms enable-key-rotation --key-id $KMS_KEY_ID
```

#### ④ 현재 리전 내 EC2 서비스의 EBS 암호화 활성화

```bash
aws ec2 enable-ebs-encryption-by-default
```

#### ⑤ 앞서 생성한 CMK를 EBS 암호화에 사용하는 KMS 키로 설정

```bash
aws ec2 modify-ebs-default-kms-key-id \
  --kms-key-id alias/AWSCookbook107Key
```

---

**🥕 유효성 검사**

```bash
# EC2 서비스에 대한 기본 EBS 암호화 상태 확인
aws ec2 get-ebs-encryption-by-default

# 기본 암호화에 사용하는 KMS 키 ID 검색
aws ec2 get-ebs-default-kms-key-id

# 생성한 키의 자동 교체 상태 확인
aws kms get-key-rotation-status --key-id $KMS_KEY_ID
```

🥕 참조

⍢ **KMS**: 다양한 데이터 암호화 전략을 유연하게 구현할 수 있는 서비스

- 기존 IAM 정책과 함께 사용 가능
- S3, EC2 EBS Volume, RDS, DynamoDB, EFS Volume 등의 저장 데이터 암호화 가능

## 1.4 IAM 정책 시뮬레이터를 사용해서 IAM 정책 테스트

**문제 설명**
IAM 정책을 실제로 사용하기 전에 영향 범위를 확인하고자 해요.

**해결 방법**
IAM 정책을 IAM 역할에 연결하고 IAM 정책 시뮬레이터로 작업할 수 있어요.

1. assume-role 정책이 적힌 json 파일 다운로드

2. 위 파일의 정책을 이용해서 IAM 역할을 생성해요.

```bash
aws iam create-role --assume-role-policy-document file://assume-role-policy.json --role-name AWSCookbook104IamRole
```

3. IAM 관리형 정책인 AmazonEC2ReadOnlyAcces를 생성한 IAM 역할에 연결해요.

```bash
aws iam attach-role-policy --role-name AWSCookbook104IamRole --policy-arn arn:aws:iam::aws:policy/AmazonEC2ReadOnlyAccess
```

**유효성 검사**

- 아래 명령어로 ec2:CreateInternetGateway 작업 시뮬레이션을 할 수 있어요.

```bash
aws iam simulate-principal-policy --policy-source-arn arn:aws:iam:::role/AWSCookbook104IamRole --action-names ec2:CreateInternetGateway
```

ReadOnly는 해당 작업 수행 불가능 하므로 implicitDeny

- 아래 명령어로 ec2:DescribeInstances 작업을 수행해볼 수 있어요.

```bash
aws iam simulate-principal-policy --policy-source-arn arn:aws:iam:::role/AWSCookbook104IamRole --action-names ec2:describeInstances
```

ReadOnly는 해당 작업 수행 가능하므로 allowed

### 정리

- `aws iam simulate` 로 생성된 iam 역할에 부여된 정책에 따라 영향 범위를 확인하고 테스트 할 수 있어요.

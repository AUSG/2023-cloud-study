# 3.9 DataSync를 활용한 EFS와 S3 간의 데이터 복제

> _S3에서 EFS를 대상으로 AWS DataSync를 구성하여 S3→EFS로의 파일 복제를 수행한다_

- 사전 준비 : S3 버킷, EFS 파일 시스템, EFS에 연결한 EC2 인스턴스

<br>

① `assert-role-policy.json`을 참고하여 S3에 대한 IAM 역할 생성

```bash
S3_ROLE_ARN=$(aws iam create-role --role-name AWSCookbookS3LocationRole \
	--assume-role-policy-document file://assume-role-policy.json \
	--output text --query Role.Arn)
```

② `AmazonS3ReadOnlyAccess` IAM 관리형 정책을 위 역할에 연결

```bash
aws iam attach-role-policy --role-name AWSCookbookS3LocationRole \
	--policy-arn arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess
```

③ DataSync S3 위치 생성

```bash
S3_LOCATION_ARN=$(aws datasync create-location-s3 \
	--s3-bucket-arn $BUCKET_ARN \
	--s3-config BucketAccessRoleArn=$S3_ROLE_ARN \
	--output text --query LocationArn)
```

④ ①번과 마찬가지로 EFS에 대한 IAM 역할을 생성하고, `AmazonElasticFileSystemClientReadWriteAccess` IAM 관리형 정책 연결

```bash
EFS_ROLE_ARN=$(aws iam create-role --role-name AWSCookbookEFSLocationRole \
	--assume-role-policy-document file://assume-role-policy.json \
	--output text --query Role.Arn)

aws iam attach-role-policy --role-name AWSCookbookEFSLocationRole \
	--policy-arn arn:aws:iam::aws:policy/AmazonElasticFileSystemClientReadWriteAccess
```

⑤ EFS 파일 시스템의 ARN, 서브넷의 ARN, 보안 그룹의 ARN 저장

```bash
# EFS ARN
EFS_FS_ARN=$(aws efs describe-file-systems \
	--file-system-id $EFS_ID \
	--output text --query FileSystems[0].FileSystemArn)

# 서브넷 ARN
SUBNET_ARN=$(aws ec2 describe-subnets \
	--subnet-ids $PRIVATE_SUBNET1 \
	--output text --query Subnets[0].SubnetArn)

# 보안 그룹 ARN
SG_ARN=arn:aws:ec2:$AWS_REGION:$AWS_ACCOUNT_ID:security-group/$EFS_SG
```

⑥ DataSync EFS 위치 생성

```bash
EFS_LOCATION_ARN=$(aws datasync create-location-efs \
	--efs-filesystem-arn $EFS_FS_ARN \
	--ec2-config SubnetArn=$SUBNET_ARN,SecurityGroupArns=[$SG_ARN] \
	--output text)
```

⑦ DataSync 작업을 생성 및 실행

```bash
# datasync 생성
TASK_ARN=$(aws datasync create-task \
	--source-location-arn $S3_LOCATION_ARN \
	--destination-location-arn $EFS_LOCATION_ARN \
	--output text --query TaskArn)

# datasync 실행
aws datasync start-task-execution \
	--task-arn $TASK_ARN

# 작업 status 확인
aws datasync list-task-executions \
	--task-arn $TASK_ARN
```

---

**🥕 참고**

⍢ **AWS DataSync**

- 복사 항목의 메타데이터는 보존
- 동기화 작업 중에 파일 무결성을 검사하여 필요 시 재시도 수행

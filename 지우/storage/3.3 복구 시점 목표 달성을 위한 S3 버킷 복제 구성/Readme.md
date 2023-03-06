# 3.3 복구 시점 목표 달성을 위한 S3 버킷 복제 구성

> _동일 리전에서 객체를 복제해 15분의 RPO를 달성하도록 한다_

- 사전 조건 : 버전 관리를 활성화한 소스 S3 버킷

① 대상 S3 버킷 생성 및 버전 관리 활성화

```bash
aws s3api create-bucket --bucket awscookbook303-dst-$RANDOM_STRING

aws s3api put-bucket-versioning \
	--bucket awscookbook303-dst-$RANDOM_STRING \
	--versioning-configuration Status-Enabled
```

② `s3-assume-role-policy.json` 생성 및 IAM 역할 생성

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "s3.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

```bash
# IAM role
ROLE_ARN=$(aws iam create-role --role-name AWSCookbook303S3Role \
	--assume-role-policy-document file://s3-assume-role-policy.json \
	--output text --query Role.Arn)
```

③ S3 복제 시 소스 및 대상 버킷에 액세스할 수 있도록 `s3-perms-policy-template.json` 생성

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObjectVersionForReplication",
        "s3:GetObjectVersionAcl",
        "s3:GetObjectVersionTagging"
      ],
      "Resource": ["arn:aws:s3:::SRCBUCKET/*"]
    },
    {
      "Effect": "Allow",
      "Action": ["s3:ListBucket", "s3:GetReplicationConfiguration"],
      "Resource": ["arn:aws:s3:::SRCBUCKET"]
    },
    {
      "Effect": "Allow",
      "Action": [
        "s3:ReplicateObject",
        "s3:ReplicateDelete",
        "s3:ReplicateTags"
      ],
      "Resource": "arn:aws:s3:::DSTBUCKET/*"
    }
  ]
}
```

④ 위 파일의 `DSTBUCKET` 및 `SRCBUCKET` 값을 치환한 `s3-perms-policy.json` 생성

```bash
sed -e "s/DSTBUCKET/awscookbook303-dst-${RANDOM_STRING}/g" \
	-e "s|SRCBUCKET|awscookbook303-src-${RANDOM_STRING}|g" \
	s3-perms-policy-template.json > s3-perms-policy.json
```

⑤ ②번에서 생성한 IAM 역할에 ④번에서 생성한 정책 연결

```bash
aws iam put-role-policy \
	--role-name AWSCookbook303S3Role \
	--policy-document file://s3-perms-policy.json \
	--policy-name S3ReplicationPolicy
```

⑥ `s3-replication-template.json`을 생성하여 대상 버킷에 대한 복제 시간을 15분으로 구성

```json
{
  "Rules": [
    {
      "Status": "Enabled",
      "Filter": {
        "Prefix": ""
      },
      "Destination": {
        "Bucket": "arn:aws:s3:::DSTBUCKET",
        "Metrics": {
          "Status": "Enabled",
          "EventThreshold": {
            "Minutes": 15
          }
        },
        "ReplicationTime": {
          "Status": "Enabled",
          "Time": {
            "Minutes": 15
          }
        }
      },
      "DeleteMarkerReplication": {
        "Status": "Disabled"
      },
      "Priority": 1
    }
  ],
  "Role": "ROLEARN"
}
```

⑦ 위 파일의 `DSTBUCKET` 및 `ROLEARN` 값을 치환하여 `s3-replication.json`로 저장

```bash
sed -e "s|ROLEARN|${ROLE_ARN}|g" \
	-e "s|DSTBUCKET|$awscookbook303-dst-${RANDOM_STRING}|g" \
	s3-replication-template.json > s3-replication.json
```

⑧ 소스 S3 버킷에 대한 복제 정책 구성

```bash
aws s3api put-bucket-replication \
	--replication-configuration file://s3-replication.json \
	--bucket awscookbook303-src-${RANDOM_STRING}
```

🥕 **참고**

⍢ S3 복제의 두 가지 유형

- SRR(Same-Region Replication) : IAM 역할, 소스 및 대상 버킷, 역할 및 버킷을 참고하는 복제 구성
  - 인덱싱을 위한 중앙 버킷에 로그 집계
  - prod 환경과 test 환경 간의 데이터 복제
  - 객체 메타데이터를 유지하며 데이터 중복성 유지
  - 데이터 주권 및 규정 준수 요구 사항에 대한 이중화 설계
  - 백업 및 보관 목적
- CRR(Cross-Region Replication) : SRR과 동일한 복제 구성 사용, 추가적으로 아래 기능 구성 가능
  - 지역 간 데이터 저장 및 아카이브 요구 사항 충족
  - 지역적으로 더 가까운 데이터셋에 액세스해 대기 시간 감소

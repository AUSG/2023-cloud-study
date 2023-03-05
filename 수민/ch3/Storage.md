
# 3.1 S3 수명 주기 정책을 사용한 스토리지 비용 절감
S3 스토리지 클래스에는 알 수 없거나 액세스 패턴이 변경되는 데이터에 대한 자동 비용 절감을 위한 S3 Intelligent-Tiering, 자주 액세스하는 데이터를 위한 S3 Standard, 자주 액세스하지 않는 데이터를 위한 S3 Standard-Infrequent Access(S3 Standard-IA) 및 S3 One Zone-Infrequent Access(S3 One Zone-IA), 즉각적인 액세스가 필요한 아카이브 데이터를 위한 S3 Glacier Instant Retrieval, 즉각적인 액세스가 필요하지 않고 거의 액세스하지 않는 장기 데이터를 위한 S3 Glacier Flexible Retrieval(이전 S3 Glacier), 클라우드에서 가장 저렴한 스토리지로 몇 시간 만에 검색 가능한 장기간 아카이브 및 디지털 보존을 위한 Amazon S3 Glacier Deep Archive(S3 Glacier Deep Archive)가 포함. <br>
<br>
수명 주기 정책을 사용해서 S3 standard-IA로 클래스를 변경시킬 수도 있음.

# 3.2 S3 Intelligent-Tiering 아카이브 정책을 사용한 S3 객체 자동 아카이브
S3 Intelligent-Tiering 아카이브는 자주 액세스하지 않는 객체를 S3 Glacier 아카이브로 전환하는 자동 메커니즘을 제공. S3계층은 객체별로 적용되며 Intelligent-Tiering 아카이브는 버킷별로 적용. 스토리지 계층의 경우 객체에 있는 객체 작업을 통해서 변경할 수 있으며 아카이브는 버킷의 속성을 통해서 구성할 수 있음.

# 3.3 복구 시점 목표 달성을 위한 S3버킷 복제 구성
``` bash
# 대상으로 사용할 버킷과 src 버킷 생성하고 버전 관리를 활성화
aws s3api create-bucket --bucket aws303eeap --create-bucket-configuration LocationConstraint=us-west-2
aws s3api create-bucket --bucket aws303-dst-eeap --create-bucket-configuration LocationConstraint=us-west-2
aws s3api put-bucket-versioning --bucket aws303eeap --versioning-configuration Status=Enabled
aws s3api put-bucket-versioning --bucket aws303-dst-eeap --versioning-configuration Status=Enabled

# IAM 역할 생성
ROLE_ARN=$(aws iam create-role --role-name aws303role --assume-role-policy-document file://s3-assume-role-policy.json --output text --query Role.Arn)

# s3 관련 정책 추가
aws iam put-role-policy --role-name aws303role --policy-document file://s3-perms-policy.json --policy-name S3ReplicationPolicy

# src s3버킷에 대한 복제 정책 구성
aws s3api put-bucket-replication --replication-configuration file://s3-replication.json --bucket aws303eeap

# 버킷에 객체 복사
aws s3 cp ./test.jpg s3://aws303eeap

# 파일의 복제 상태 확인(15분후 pending에서 completed로 변경)
aws s3api head-object --bucket aws303eeap --key test.jpg

```
s3는 같은 리전 복제(SRR)와 크로스 리전 복제(CRR)라는 두 가지 유형의 복제를 제공. 복제 시간은 s3 복제 시간 제어(RTC)의 ㅣ매개 변수 중 하나이며 서비스 수준 계약(SLA)에 따르면 15분이라는 복구 시점 목표를 지원.<br>
SRR 사용 사례
- 인덱싱을 위한 중앙 버킷에 로그 집계
- 프로덕션 환경과 테스트 환경 간의 데이터 복제
- 객체 메타데이터를 유지하면서 데이터 중복성 유지
- 데이터 주권 및 규정 준수 요구 사항에 대한 이중활 설계
- 백업 및 보관 목적

# 3.4 Storage Lens를 사용해 S3의 스토리지 및 액세스 지표 확인
S3 storage Lens는 aws 계정에 대한 s3 사용량을 볼 수 있는 기능을 제공. 버킷 사용량 분석, 스토리지 비용 관찰, 이상 항목 탐색 등 여러 가지 사용 사례 지원. s3 버킷 콘솔창에서 대시보드 생성 가능.

# 3.5 S3 액세스 포인트를 사용해 별도의 애플리케이션 액세스 구성
만약 s3버킷을 사용하는 애플리케이션이 두개가 존재하고 두개의 접근 권한을 다르게 부여할 필요가 있다면 s3의 액세스 포인트를 사용해 설정할 수 있음.<br>
먼저 vpc에 대한 s3 액세스 포인트를 s3 콘솔에서 만들고 거기서 정책을 인스턴스 롤에 대해서 권한을 따로 부여해주는 정책을 부여해주면 됨.
``` json
{
    "Version":"2012-10-17",
    "Statement": [
    {
        "Effect": "Allow",
        "Principal": {
            "AWS": "EC2_INSTANCE_PROFILE"
        },
        "Action": [ACTIONS],
        "Resource": "arn:aws:s3:AWS_REGION:AWS_ACCOUNT_ID:accesspoint/ACCESS_POINT_NAME/object/*"
    }]
}
```

# 3.6 AWS KMS를 사용한 Amazon S3 버킷의 객체 암호화
s3객체를 저장할 때 암호화 방식을 이용해서 객체를 저장하는 방법이 존재. aws KMS의 CMK를 이용하면 되는데 키를 생성한 후에 aws s3 해당 버킷에 들어가서 속성 부분에서 기본 암호화에 들어가서 만든 key로 수정해주면 된다. 그리고 버킷에 모든 객체를 암호화하는 버킷 정책을 추가해주면 됨.
``` json
{
    "Version":"2012-10-17",
    "Id":"PutObjectPolicy",
    "Statement":[{
          "Sid":"DenyUnEncryptedObjectUploads",
          "Effect":"Deny",
          "Principal":"*",
          "Action":"s3:PutObject",
          "Resource":"arn:aws:s3:::BUCKET_NAME/*",
          "Condition":{
             "StringNotEquals":{
                "s3:x-amz-server-side-encryption":"aws:kms"
             }
          }
       }
    ]
 }
 

```
해당 정책을 버킷에 추가하면 객체를 업로드할 때 kms 키를 이용해서 업로드해야함.

# 3.7 aws Backup을 사용해 다른 리전에 EC2 백업 생성
ec2인스턴스의 온디맨드 백업을 생성하고 aws console의 백업 볼트로부터 인스턴스를 복원. 먼저 온디맨드 백업 생성을 완료한 후에 이 만들어진 이미지를 다른 리전으로 복사한 후 대상 리전을 선택한 후 복원이 가능.

# 3.8 EBS 스냅샷 내의 파일 복원
EBS 스냅샷을 사용하면 스냅샷이 생성된 시점으로 인스턴스를 복원할 수 있음. 또한 스냅샷에서 EBS 볼륨을 생성하고 실행 중인 인스턴스에 연결이 가능.

# 3.9 DataSync를 활용한 EFS와 S3간의 데이터 복제
DataSync 서비스는 aws 서비스의 온디맨드 또는 지속적/자동화한 파일 동기화 작업을 수행. DataSync는 복사하는 항목의 메타데이터를 보존하고 동기화 작업 중에 파일 무결성을 검사해 필요한 경우 재시도를 수행. 따라서 인프라를 따로 프로비저닝하거나 스크립트 파일을 작성할 필요가 없음.<br>
aws datasync 서비스에서 위치를 각각 src와 dst을 생성하고 task를 생성해서 작업을 실행할 수 있음.
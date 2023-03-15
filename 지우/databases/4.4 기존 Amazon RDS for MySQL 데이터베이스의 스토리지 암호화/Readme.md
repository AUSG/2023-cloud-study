# 4.4 기존 Amazon RDS for MySQL 데이터베이스의 스토리지 암호화

> _기존 DB의 읽기 전용 복제본과 그 스냅샷을 생성하고, 스냅샷을 암호화된 스냅샷으로 복사 및 새 DB로 복원하여 기존 DB 스토리지를 암호환한다_

① DB 저장소가 암호화되지 않은 것 확인

```bash
# false를 반환하면 암호화되지 않았다는 의미
aws rds describe-db-instances \
	--db-instance-identifier $RDS_ID \
	--query DBInstances[0].StorageEncrypted
```

② DB 스냅샷을 암호화하기 위한 KMS 키를 생성하고 환경 변수에 저장

```bash
KEY_ID=$(aws kms create-key \
	--tags TagKey=Name,TagValue=AWSCookbook404RDS \
	--description "AWSCookbook RDS key" \
	--query KeyMetadata.KeyId \
	--output text)
```

③ 생성한 키를 쉽게 참고하기 위한 별칭 생성

```bash
aws kms create-alias \
	--alias-name alias/awscookbook404 \
	--target-key-id $KEY_ID
```

④ 암호화되지 않은 기존 DB의 읽기 전용 복제본과 그 스냅샷을 생성

```bash
# 기존 DB의 읽기 전용 복제본 생성
aws rds create-db-instance-read-replica \
	--db-instance-identifier awscookbook404db-rep \
	--source-db-instance-identifier $RDS_ID \
	--max-allocated-storage 10

# DBInstanceStatus가 available 될 때까지 대기
aws rds describe-db-instances \
	--db-instance-identifier awscookbook404db-rep \
	--output text --query DBInstances[0].DBInstanceStatus
```

```bash
# 읽기 전용 복제본의 스냅샷 생성 → 암호화되어 있지 않음
aws rds create-db-snapshot \
	--db-instance-identifier awscookbookdb-rep \
	--db-snapshot-identifier awscookbook404-snapshot

# 스냅샷이 가용 상태가 될 때까지 대기
aws rds describe-db-snapshots \
	--db-snapshot-identifier awscookbook404-snapshot \
	--output text --query DBSnapshots[0].Status
```

⑤ KMS 키로 암호화되지 않은 스냅샷을 새 스냅샷으로 복사

```bash
aws rds copy-db-snapshot \
	--copy-tags \
	--source-db-snapshot-identifier awscookbook404-snapshot \
	--target-db-snapshot-identifier awscookbook404-snapshot-enc \
	--kms-key-id alias/awscookbook404

# 암호화된 스냅샷이 가용 상태가 될 때까지 대기
aws rds describe-db-snapshots \
	--db-snapshot-identifier awscookbook404-snapshot-enc \
	--output text --query DBSnapshots[0].Status
```

⑥ 암호화된 새 스냅샷을 새 RDS 인스턴스로 복원

```bash
aws rds restore-db-instance-from-db-snapshot \
	--db-subnet-group-name $RDS_SUBNET_GROUP \
	--db-instance-identifier awscookbook404db-enc \
	--db-snapshot-identifier awscookbook404-snapshot-enc
```

---

🥕 **유효성 검사** : 스토리지 암호화 여부 확인

```bash
# DBInstanceStatus를 사용할 수 있을 때까지 대기
aws rds describe-db-instances \
	--db-instance-identifier awscookbook404db-enc \
	--output text --query DBInstances[0].DBInstanceStatus

# 스토리지 암호화 여부 확인 → true
aws rds describe-db-instances \
	--db-instance-identifier awscookbook404db-enc \
	--query DBInstances[0].StorageEncrypted
```

**🥕 참고**

⍢ **저장 시 암호화(encryption at rest)** : AWS 공통 책임 모델 중 최종 사용자가 관리해야 하는 보안 접근 방식

⍢ **암호화된 스냅샷** : 자동으로 다른 리전으로 복사하거나 보관/백업을 위해 S3로 export 가능

⍢ 새 DB 생성 후에 새로운 엔드포인트를 사용해야 하고, 가동 중지 시간을 최소화하기 위해 그 엔드포인트를 가리키는 Route53 DNS 레코드를 구성할 수 있다.

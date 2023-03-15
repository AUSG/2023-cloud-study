# 2. IAM 인증을 사용한 RDS 접속

### 실습

---

데이터베이스에 연결할 때 **암호를 사용하는 대신 교체 임시 자격 증명을 사용**하고자 한다.

→ **데이터베이스 IAM 인증을 활성화** 한다. 그 후, EC2 인스턴스가 사용할 IAM 권한을 구성한다. 마지막으로, 데이터베이스에 새 사용자를 생성하고 **IAM 인증 토큰**을 사용해 연결한다.

1. RDS 데이터베이스 인스턴스의 IAM 데이터베이스 인증을 활성화

   ```bash
   aws rds modify-db-instance --db-instance-identifier $RDS_DATABASE_ID --enable-iam-database-authentication --apply-immediately
   ```

2. RDS 데이터베이스 인스턴스의 리소스 ID 저장

   ```bash
   DB_RESOURCE_ID=$(aws rds describe-db-instances --query 'DBInstances[?DBName==`AWSCookbook402`].DbiResourceId' --output text)
   ```

3. 저장소를 참고해 다음 내용을 포함하는 policy-template.json 템플릿 파일의 값을 환경변수의 값으로 치환

   ```bash
   sed -e "s|AWS_ACCOUNT_ID|${AWS_ACCOUNT_ID}|g" -e "s|AWS_REGION|${AWS_REGION}|g" -e "s|DBResourceId|${DB_RESOURCE_ID}|g" policy-template.json > policy.json
   ```

4. 방금 생성한 파일을 사용해 IAM 정책을 생성한다.

   ```bash
   aws iam create-policy --policy-name AWSCookbook402EC2RDSPolicy --policy-document file://policy.json
   ```

5. 방금 생성한 IAM 정책 AWSCookbook402EC2RDSPolicy를 EC2가 사용 중인 IAM 역할에 연결

   ```bash
   aws iam attach-role-policy --role-name $INSTANCE_ROLE_NAME --policy-arn arn:aws:iam::$AWS_ACCOUNT_ID:policy/AWSCookbook402EC2RDSPolicy
   ```

6. Secrets Manager(사전 생성)에서 RDS 관리자 암호를 확인

   ```bash
   RDS_ADMIN_PASSWORD=$(aws secretsmanager get-secret-value --secret-id $RDS_SECRET_ARN --query SecretString | jq -r | jq .password |tr -d '"')
   ```

7. EC2 인스턴스에서 데이터베이스 연결할 때 사용할 수 있는 RDS 클러스터의 엔드포인트 확인
8. RDS 클러스터 암호 확인
9. EC2 인스턴스 연결 후, MySQL 설치
10. 데이터베이스 연결하고 welcome 출력 확인
11. IAM 인증 시 사용할 데이터베이스 사용자를 생성

<br>
<br>

### EC2 인스턴스의 IAM 역할과 연결된 토큰을 사용해 MySQL 연결하기 (암호문자열 대신)

---

- IAM은 임시 토큰을 15분 동안 유지한다.
- EC2 인스턴스에 설치된 애플리케이션은 AWS SDK를 통해 해당 토큰을 정기적으로 새로 발급받을 수 있다.
- IAM은 특정 사용자에 대한 db-connect 작업을 제어하고 인증 토큰을 취득한다.
- EC2 인스턴스에서 데이터베이스를 연결할 때 SSL 인증서 번들을 사용해 전송 중 암호화를 활성화
  - 애플리케이션과 데이터베이스 간의 연결을 암호화하는 것은 좋은 보안 관행이며 규정 준수 표준의 요구 사항 중 하나다.
  - 데이터베이스에 IAM 인증 토큰으로 연결할 때 SSL 인증서를 연결 매개변수 (`—ssl-ca`)로 사용한다.

## 3.1 S3 수명 주기 정책을 사용한 스토리지 비용 절감

### 🎯 목표

자주 사용하지 않는 객체를 운영 오버헤드를 최소화하면서 지능적이고 확장할 수 있으며 안전한 시스템을 구축할 수 있도록 여러 가지 서비스를 사용하는 레시피를 살펴본다.

### 🤔 해결 방법

- 30일 이후에 S3 IA(Infrequent Access) 스토리지 클래스로 전환하는 S3 수명 주기 정책을 생성하고 S3 버킷에 적용한다.

### 👉 작업 방법

1. S3 버킷에 적용할 수명 주기 정책을 생성
2. 수명 주기 규칙을 버킷에 적용

```bash
aws s3api put-bucket-lifecycle-configuration \
    --bucket awscookbook301-$RANDOM_STRING \
    --lifecycle-configuration file://lifecycle-rule.json
```

3. 유효성 검사
   (1) 수명 주기 규칙 확인

```bash
aws s3api get-bucket-lifecycle-configuration \
    --bucket awscookbook301-$RANDOOM_STRING
```

### 참고

- S3 버킷에 객체를 업로드 할 때 스토리지 클래스를 지정하지 않으면, 기본 클래스가 적용됨
- 애플리케이션이 객체를 업로드 할 때 스토리지 계층을 지정할 수 없다면 수명 주기 규칙을 통해 원하는 스토리지 클래스로의 전환을 자동화 할 수 있다.
- 수명 주기 규칙 필터를 통해 버킷 내의 일부 객체 또는 모든 객체에 적용 가능

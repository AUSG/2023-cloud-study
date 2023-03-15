# 5. RDS 데이터베이스의 암호 교체 자동화

### 실습

---

목표 : 데이터베이스 사용자에 대한 자동 비밀번호 교체 구현

해결방법 : 암호를 생성해 **AWS Secrets Manager**에 저장하고 암호 교체 시간의 간격을 설정 → **암호 교체를 수행하는** **람다 함수**를 구성

- RDS 인스턴스 보안 그룹에 람다의 보안 그룹의 TCP 포트 3306에 대한 액세스를 허용하는 수신 규칙을 추가

- AWSLambdaVPCAccess IAM 관리형 정책을 IAM 역할에 연결

- SecretsManagerReadWrite IAM 관리형 정책을 IAM 역할에 연결

<Br>
<br>

### 정리

---

- AWS에서 제공하는 **람다 함수를 사용해 Secrets Manager의 암호를 교체**할 수 있다.

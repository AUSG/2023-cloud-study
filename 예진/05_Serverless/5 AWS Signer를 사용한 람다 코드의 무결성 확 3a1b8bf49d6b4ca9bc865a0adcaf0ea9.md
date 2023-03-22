# 5. AWS Signer를 사용한 람다 코드의 무결성 확인

<br>

### 실습

---

서명 프로필을 생성한 다음 AWS Signer를 사용해 코드에 서명을 활성화하고 코드 서명 구성을 참고해 서명된 코드를 사용하는 람다 함수를 배포해보자.

→ 람다 함수 코드의 무결성을 확인하고 코드가 서명된 후 수정되지 않았음을 확인할 수 있어야 한다.

준비사항

- 버전관리를 활성화한 S3 버킷 `505bucket` ← 미리 실행할 함수 zip 파일 넣어두기
- AWS Signer가 대상으로 사용할 S3 버킷 `505bucket-target`
- 람다 함수 실행을 위한 IAM 역할 : `AWSLambdaBasicExecutionRole`

1. 람다함수에서 사용할 압축된 코드의 `S3의 객체 버전`을 가져와 환경 변수에 저장 : `aws s3api list-object-versions`
2. 서명 프로필을 생성 : `aws signer put-signing-profile`
3. 코드 서명 구성을 생성 (람다 함수에 서명 프로필을 참조하게끔 할) : `aws lambda create-code-signing-config`
4. 서명 작업 시작 : `aws signer start-signing-job`
   ![image](https://user-images.githubusercontent.com/49095587/226856959-1ccfa896-cdc1-4fe3-8811-fc6b5ff7655d.png)
   signer가 대상으로 사용하는 S3 버킷에 서명 작업이 성공하였다.
5. 서명된 코드의 S3 객체 키를 확인 : `aws s3api list-objects-v2`
6. 서명된 코드를 사용하는 람다 함수 생성 : `aws lambda create-function`
7. 다음 명령어로 람다 함수 활성화 확인 : `aws lambda get-function`
   <br>

**유효성 검사**

![image](https://user-images.githubusercontent.com/49095587/226857015-46d48f56-d0ca-452b-b0bd-456fd058c82c.png)

<br>

### 정리

---

보안 책임자와 애플리케이션 개발자는 AWS Signer를 사용해 주어진 환경에 신뢰할 수 있는 코드만 배포하도록 허용하는 규칙을 적용하는 DevSecOps 전략을 구현할 수 있다.

<br>

### AWS Signer

---

- 코드 또는 애플리케이션에 디지털 서명을 추가하여 신뢰성과 보안성을 높이는 서비스

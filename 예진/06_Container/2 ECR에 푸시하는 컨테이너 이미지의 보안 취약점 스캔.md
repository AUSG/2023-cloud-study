# 2. ECR에 푸시하는 컨테이너 이미지의 보안 취약점 스캔

### 실습

---

- ECR 리포지토리로 컨테이너 이미지를 푸시할 때마다 보안 취약점을 자동으로 스캔하자.
- ECR의 리포지토리에서 자동 이미지 스캔을 활성화한 뒤 이미지를 푸시하고 스캔 결과를 확인

1. 이미 배포돼있는 nginx 컨테이너 이미지 사용

   ```bash
   docker pull nginx:1.14.1
   ```

2. **미리 생성한 리포지토리(`cookbook602`)에 스캔 구성을 적용**

   ```bash
   REPO=cookbook602 && aws ecr put-image-scanning-configuration --repository-name $REPO --image-scanning-configuration scanOnPush=true
   ```

3. Docker 로그인 정보를 설정

   ```bash
   aws ecr get-login-password | docker login --username AWS --password-stdin $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com
   ```

4. ECR 리포지토리에 푸시할 수 있도록 이미지에 태그를 적용

   ```bash
   docker tag nginx:1.14.1 $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/cookbook602:old
   ```

5. 이미지 푸시

   ```bash
   docker push $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/cookbook602:old
   ```

6. 푸시를 완료한 뒤 JSON 형식으로 스캔 결과를 확인

   ```bash
   aws ecr describe-image-scan-findings --repository-name cookbook602 --image-id imageTag=old
   ```

   → 언제든지 이미지에 대한 마지막 스캔 결과를 검색할 수 있다.

   ```bash
   # 출력 결과
   {
       "imageScanFindings": {
           "findings": [
               {
                   "name": "CVE-2019-3462",
                   "description": "Incorrect sanitation of the 302 redirect field in HTTP trans
   port method of apt versions 1.4.8 and earlier can lead to content injection by a MITM attack
   er, potentially leading to remote code execution on the target machine.",
                   "uri": "https://security-tracker.debian.org/tracker/CVE-2019-3462",
                   "severity": "CRITICAL",
                   "attributes": [
                       {
                           "key": "package_version",
                           "value": "1.4.8"
                       },
                       {
                           "key": "package_name",
                           "value": "apt"
                       },
                       {
                           "key": "CVSS2_VECTOR",
                           "value": "AV:N/AC:M/Au:N/C:C/I:C/A:C"
                       },
                       {
                           "key": "CVSS2_SCORE",
                           "value": "9.3"
                       }
                   ]
               },
               {
                   "name": "CVE-2021-45960",
   ```

<br>

### 정리

---

- ECR의 취약점 스캐닝은 오픈소스 Clair 프로젝트의 CVE 데이터 베이스를 사용하며 취약점의 심각함을 나타내고자 CVSS 점수를 사용한다.
  - CVSS 점수 : CVSS(공통 취약점 평가 시스템)는 취약점의 심각성을 측정하는 데 사용되는 척도입니다. 이 척도는 취약점의 중요성을 0.0에서 10.0까지의 점수로 평가합니다. 높은 점수는 취약점이 심각하다는 것을 나타냅니다. CVSS 점수는 다양한 측면을 고려하여 계산되며, 이는 취약점의 기반 요소, 영향 범위 및 기술적 세부 정보를 고려합니다.
  - CVE 데이터베이스 : 인터넷상의 다양한 소프트웨어 및 하드웨어 제품에서 발견된 취약점과 보안 문제를 추적하기 위한 전 세계적인 공개 데이터베이스
- 이를 사용해 컨테이너 이미지의 취약점을 감지하고 수정할 수 있다.
- EventBridge나 SNS를 연동해 이미지에서 새로 발견한 취약점에 대한 알림을 구성할 수 있다.

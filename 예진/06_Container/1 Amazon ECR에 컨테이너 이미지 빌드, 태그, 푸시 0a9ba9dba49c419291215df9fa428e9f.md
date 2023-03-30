# 1. Amazon ECR에 컨테이너 이미지 빌드, 태그, 푸시

### 실습

---

도커 파일을 만들고 도커 컨테이너 이미지를 빌드한 후, 컨테이너 이미지를 저장할 저장소인 Amazon ECR 에 푸시해보자

![image](https://user-images.githubusercontent.com/49095587/228862448-86f3d087-5d27-4b01-8334-82eaa15f163b.png)

1. 콘솔에서 ECR에 들어가 리포지토리 생성
   ![image](https://user-images.githubusercontent.com/49095587/228862491-6c3bf516-f90a-40dc-947e-c2851f574856.png)

1. Dockerfile 생성

   ```bash
   echo FROM nginx:latest > Dockerfile
   ```

   → nginx:latest 이미지를 기본 이미지로 사용하는 Dockerfile을 생성하는 코드이다.

1. 이미지를 빌드한 뒤 태그. (도커 파일을 만든 위치에서)

   1. 이미지 레이어를 다운로드하고 결합하는데 몇분 정도가 필요하다.

   - 이미지 레이어?
     컨테이너 이미지는 여러 레이어로 구성되는데 각 레이어는 컨테이너 이미지를 구성하는 파일 시스템 변경사항의 스냅샷이다. 버전 관리를 지원하는 것.

   ```bash
   docker build . -t $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/awscookbook601:latest
   ```

1. 태그를 추가

   ```bash
   docker tag $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/awscookbook601:latest $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/awscookbook601:1.0
   ```

1. Docker 로그인 정보 가져오기

   ```bash
   aws ecr get-login-password | docker login --username AWS --password-stdin $AWS_ACCOUNT_ID.dkr.ecr.us-east-1.amazonaws.com
   ```

   → `Login Succeede` 출력 확인

   - AWS ECR에 로그인하기 위한 비밀번호를 가져온 후, Docker CLI를 사용하여 해당 계정에 로그인한다. 이렇게 로그인된 계정은 AWS ECR에서 Docker 이미지를 push 및 pull 할 수 있다.

1. 각 이미지 태그를 Amazon ECR에 푸시한다.

   ```bash
   # 첫번째 이미지 푸시
   docker push $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/awscookbook601:latest

   # 두번째 이미지 푸시
   docker push $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/awscookbook601:1.0
   ```

   → 두번째 이미지 푸시할 때 `Layer already exists`를 확인 ( 첫 푸시에 의한 이미지가 존재하기 때문! )

1. 푸시된 이미지 확인

   ![image](https://user-images.githubusercontent.com/49095587/228862662-7caf5026-f782-4a09-a78c-aacc63066dc6.png)

   <br>

### 정리

---

- 컨테이너 **태깅**을 사용하면 **컨테이너 이미지의 버전을 지정하고 추적**할 수 있다.
  - 예를 들어 개발 환경에서 항상 ‘latest’ 태그된 이미지를 사용하고 프로덕션 환경은 다른 ‘test’라고 지어진 태그된 특정 버전 태그를 사용할 수 있다.
- ECR 리포지토리를 통해 **보안 강화** 또한 가능하다
  - ECR에 따른 다른 AWS 계정, IAM 객체, AWS 서비스에 액세스 권한을 부여할 수 있다.

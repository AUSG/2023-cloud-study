# 4. AWS Copilot을 사용한 컨테이너 배포

### 실습

---

- 기존 Dockerfile을 사용해 네트워크의 모범 사례를 따르는 웹 서비스를 빠르게 배포하고 관리해보자
- AWS Copilot을 사용해 아래와 같은 아키텍처를 사용하는 애플리케이션을 빠르게 배포할 수 있다.

![image](https://user-images.githubusercontent.com/49095587/230257464-3bbdd311-c52c-461e-854c-c0c430e0b3ad.png)

1. AWS Copilot CLI

   ```bash
   brew install aws/tap/copilot-cli
   ```

2. **Copilot은 사용자를 대신해 ECS가 작업을 수행할 수 있도록 하는 ECS 서비스 연결 역할**을 사용한다. 이 역할은 AWS 계정에 이미 생성돼 있을 수 있는데, 다음 명령어로 확인해보자

   ```bash
   aws iam list-roles --path-prefix /aws-service-role/ecs.amazonaws.com/
   ```

   ECS 서비스 연결 역할이 없는 경우 아래 명령어로 역할 생성

   ```bash
   aws iam create-service-linked-role --aws-service-name ecs.amazonaws.com
   ```

3. 원하는 Dockerfile이 있는 디렉토리로 이동
4. AWS Copilot을 사용해 NGINX Dockerfile을 Amazon ECS에 배포

   ```bash
   copilot init --app web --name nginx --type 'Load Balanced Web Service' --dockerfile './Dockerfile' --port 80 --deploy
   ```

5. 배포가 완료되면 다음 명령을 사용해 배포된 서비스에 대한 정보를 확인

   ```bash
   copilot svc show
   ```

   <br>

### AWS Copliot

---

- AWS Copilot은 개발자들이 빠르고 간편하게 컨테이너 기반 애플리케이션을 배포, 운영할 수 있도록 도와준다.
- `copilot init` 명령을 사용해 현재 작업 디렉터리에 copilot 이라는 디렉터리를 생성
  - 여기서, 애플리케이션과 연결된 manifest.yml을 사용해 구성을 확인하고 수정할 수 있다.
- `copilot pipeline` 명령을 사용해 CI/CD 파이프라인을 이용한 자동화된 배포를 구성할 수 있다.

<br>

### Amazon ECS

---

- AWS에서 제공하는 완전관리형 **컨테이너 오케스트레이션 서비스**입니다. 이 서비스를 사용하면 컨테이너 기반 애플리케이션을 쉽게 실행하고 관리할 수 있습니다.
- Amazon ECS를 사용하면 **Docker 컨테이너를 배포하고 실행**할 수 있습니다. ECS는 **EC2 인스턴스 또는 AWS Fargate와 같은 AWS 서비스에서 실행되는 컨테이너를 관리**합니다. ECS는 간단한 API를 제공하며, 이를 사용하여 애플리케이션을 생성, 배포, 수정 및 확장할 수 있습니다.
- Amazon ECS는 **클러스터, 태스크 정의, 서비스, 로드 밸런싱, Auto Scaling 및 무중단 배포와 같은 기능**을 제공합니다. 이 서비스는 또한 Amazon ECR (Elastic Container Registry)와 통합되어 있으며, Docker 이미지를 저장하고 검색할 수 있는 완전관리형 Docker 컨테이너 레지스트리를 제공합니다.
- Amazon ECS는 컨테이너 기반 애플리케이션을 관리하기 위한 다양한 기능과 서비스를 제공하여 개발자 및 운영팀이 애플리케이션을 쉽게 배포하고 관리할 수 있도록 지원합니다.

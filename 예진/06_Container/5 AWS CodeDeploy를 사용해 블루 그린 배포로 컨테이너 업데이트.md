# 5. AWS CodeDeploy를 사용해 블루/그린 배포로 컨테이너 업데이트

- **블루/그린 배포란?**
  블루/그린 배포(Blue/Green Deployment)는 새로운 버전의 소프트웨어를 배포할 때 사용되는 전략 중 하나입니다. 이 배포 전략은 사**용자에게 불편함 없이 새로운 버전의 소프트웨어를 안정적으로 배포하고 롤백할 수 있는 방법을 제공**합니다.
  블루/그린 배포는 두 개의 환경(블루와 그린)을 준비합니다. **현재 운영 중인 환경(블루)과 새로운 버전을 배포할 준비가 된 환경(그린)**입니다. 그리고 새로운 버전의 소프트웨어를 미리 그린 환경에 배포하고 테스트합니다. 그런 다음 트래픽을 블루에서 그린으로 전환하여 사용자가 새로운 버전의 소프트웨어를 사용할 수 있도록 합니다.
  이전 버전과 새 버전의 소프트웨어를 함께 실행하면서 이전 버전과 새 버전의 트래픽을 분산시키기 때문에, 배포 중에 소프트웨어 문제가 발생하면 롤백이 쉽습니다. 롤백 시, 간단한 전환으로 이전 버전을 운영할 수 있습니다. 이를 통해 배포 중단 시간을 최소화하고 사용자 경험을 개선할 수 있습니다.
  블루/그린 배포는 많은 기업들에서 널리 사용되며, AWS에서는 Elastic Beanstalk, CodeDeploy 등 다양한 배포 관리 서비스에서 이 배포 전략을 지원합니다.
    <br>

### 실습

---

- 컨테이너 기반 애플리케이션을 배포할 때 중단 없이 애플리케이션을 최신 버전으로 배포하고 배포를 실패하는 경우에 쉽게 롤백해보자
- AWS CodeDeploy를 사용해 ECS 애플리케이션에 대한 블루/그린 배포를 적용

![image](https://user-images.githubusercontent.com/49095587/230257672-857a4aa2-1403-4427-a248-3ec47b7f73cd.png)

1. CDK 스택을 배포한 후 웹 브라우저를 열고 CDK 출력의 LOAD_BALANCER_DNS 주소를 확인. 실행 중인 ‘블루’ 애플리케이션을 확인한다.

   ```bash
   open http://$LOAD_BALANCER_DNS:8080
   ```

2. assert-role-policy.json과 다음 명령을 사용해 IAM 역할 생성

   ```bash
   aws iam create-role --role-name ecsCodeDeployRole --assume-role-policy-document file://assume-role-policy.json
   ```

3. `CodeDeployRoleForECS`에 대한 IAM 관리형 정책을 IAM 역할에 연결

   ```bash
   aws iam attach-role-policy --role-name ecsCodeDeployRole --policy-arn arn:aws:iam::aws:policy/AWSCodeDeployRoleForECS
   ```

4. CodeDeploy ‘그린’ 대상 그룹으로 사용할 새 ALB 대상 그룹 생성
   - 오류
     ```bash
     An error occurred (ValidationError) when calling the CreateTargetGroup operation: The VPC ID 'vpc-01ad05dbfdd8603d9' is not found
     ```
5. CodeDeploy 애플리케이션을 생성
   1. CodeDeploy를 사용하고자 하는 구성을 포함한 템플릿 파일(codedeploy-template.json) 참고
6. sed 명령어로 `AWS_ACCOUNT_ID`, `PROD_LISTENER_ARN`, `TEST_LISTENER_ARN` 치환
7. 배포 그룹 생성 : `aws deploy create-deployment-group`
8. sed 명령을 사용해 업데이트해야 할 애플리케이션에 대한 정보를 포함한 appspec-template.yaml의 내용을 치환 (`FARGATE_TASK_GREEN_ARN`)
9. CDK 배포를 통해 생성한 S3 버킷에 appspec.yaml을 복사
10. 마지막으로 배포에 대한 구성 파일 생성 : deployment-template.json 파일의 S3 버킷 이름을 치환 : deployment.json으로 재 생성
11. 이제 배포 구성을 사용해 배포를 생성 (deployment.json) : `aws deploy create-deployment`

<br>

### CodeDeploy의 블루/그린 전략

---

- CodeDeploy의 블루/그린 전략 : 모든 트래픽이 새 버전으로 라우팅되는 동안 애플리케이션의 이전 버전을 5분간 실행 상태로 유지한다. 새 버전이 제대로 작동하지 않는다면 별도의 AWS Application Load Balancer 대상그룹에서 이전 버전이 계속 실행 중이기 때문에 트래픽을 원래 버전으로 빠르게 라우팅할 수 있다.

<br>

### 실습 정리 → 한번 더 이해하기

---

- CodeDeploy는 ALB 대상 그룹을 사용해 ‘production’ 애플리케이션을 관리한다. AWS CDK를 사용해 초기 스택을 배포할 때 기존의 블루 컨테이너를 ALB의 포트 8080과 연결된 대상 그룹에 등록했다.
- 새 버전의 배포를 시작한 후 CodeDeploy는 새로운 버전의 ECS 서비스를 시작하고 이를 사용자가 생성한 그린 대상 그룹과 연결한 다음 모든 트래픽을 그린 대상 그룹으로 이동한다.
- 최종적으로 그린 버전의 컨테이너를 ALB의 포트 8080을 통해 제공한다.

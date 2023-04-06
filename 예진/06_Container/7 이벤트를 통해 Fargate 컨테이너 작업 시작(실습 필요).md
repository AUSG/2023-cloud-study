# 7. 이벤트를 통해 Fargate 컨테이너 작업 시작 (실습 필요)

### 실습

---

- 파일에 대한 이벤트를 처리하고자 하는 컨테이너 작업을 트리거하자
- Amazon EventBridge를 사용해 파일이 S3에 업로드 된 후 Fargate에서 컨테이너 작업 시작을 트리거한다.

![image](https://user-images.githubusercontent.com/49095587/230257808-95a7855f-32e6-4d7f-af3b-6250eed8658b.png)

- Amazon EventBridge
  AWS 리소스 및 SaaS(소프트웨어 as a Service) 애플리케이션에서 발생하는 이벤트를 실시간으로 수집, 처리, 라우팅
- Fargate
  AWS Fargate는 서버리스 컨테이너 오케스트레이션 서비스입니다. Fargate를 사용하면 컨테이너를 실행하고 관리하는 서버 인프라를 직접 프로비저닝하지 않아도 됩니다. 이를 통해 컨테이너를 쉽게 배포하고 관리할 수 있으며, 기존 애플리케이션을 빠르게 컨테이너화할 수 있습니다.
  Fargate는 Amazon ECS와 Amazon EKS와 함께 사용할 수 있습니다. ECS는 AWS에서 제공하는 컨테이너 오케스트레이션 서비스이며, EKS는 Kubernetes를 사용하여 컨테이너 오케스트레이션을 제공하는 서비스입니다. Fargate를 사용하면 ECS 또는 EKS에서 컨테이너를 실행하고 관리할 수 있습니다.
- CloudTrail
  AWS 계정 내에서 발생하는 모든 **API 호출과 이벤트**를 기록하고 모니터링하는 서비스
  AWS 계정에서 수행되는 모든 작업에 대한 로그를 수집하고, 검색 및 분석할 수 있습니다. 이러한 로그는 보안, 규정 준수, 운영 분석 등 다양한 목적으로 사용될 수 있습니다.
- EventBridge와 CloudTrail의 차이
  `CloudTrail`은 AWS 계정 내에서 발생하는 이벤트에 대한 로깅 서비스이고, `EventBridge`는 AWS 내부 및 외부 서비스에서 발생하는 이벤트를 모니터링하고 처리하는 서비스입니다. 또한, EventBridge는 이벤트를 실시간으로 처리하고 다른 AWS 서비스와 통합할 수 있는 기능을 제공합니다.

1. CloudTrail이 S3 버킷의 이벤트를 기록하도록 구성 : `aws cloudtrail put-event-selectors`
2. 역할 정책 JSON 문 생성

   ```json
   {
     "Version": "2012-10-17",
     "Statement": [
       {
         "Sid": "",
         "Effect": "Allow",
         "Principal": {
           "Service": "ecs-tasks.amazonaws.com"
         },
         "Action": "sts:AssumeRole"
       }
     ]
   }
   ```

3. 역할 생성 뒤 policy1.json 파일 지정
4. policy2.json 정책 문서 생성

   ```json
   {
     "Version": "2012-10-17",
     "Statement": [
       {
         "Effect": "Allow",
         "Action": ["ecs:RunTask"],
         "Resource": ["arn:aws:ecs:*:*:task-definition/*"]
       },
       {
         "Effect": "Allow",
         "Action": "iam:PassRole",
         "Resource": ["*"],
         "Condition": {
           "StringLike": {
             "iam:PassedToService": "ecs-tasks.amazonaws.com"
           }
         }
       }
     ]
   }
   ```

5. 이제 방금 생성한 IAM 정책 JSON(2)을 IAM 역할에 연결 : `aws iam put-role-policy`
6. S3 버킷의 파일 업로드를 모니터링하는 EventBridge 규칙을 생성
7. 시작할 컨테이너 타깃 파일 내용 수정 (target-template.json → target.json)
8. ECS 클러스터, ECS 작업 정의, IAM 역할, 네트워킹 매개 변수를 사용하는 규칙 대상을 생성해 규칙(EventBridge 규칙)이 트리거할 대상을 지정 → 이 경우 Fargate에서 컨테이너를 시작

   ```bash
   aws events put-targets --rule AWSCookbookRule --targets file://targets.json
   ```

9. S3 버킷이 비어있는지 확인 : `aws s3 ls s3://$BUCKET_NAME/`
10. maze.jpg 파일을 S3 버킷에 복사 : aws s3 cp maze.jpg s3://$BUCKET_NAME/input/maze.jpg
11. 이미지 파일을 처리하기 위한 ECS 작업을 트리거 : `ecs list-tasks` 명령으로 작업 확인
12. 몇 분 후 S3 버킷에 생성된 출력 디렉터리 확인

    ```bash
    aws s3 ls s3://$BUCKET_NAME/output
    ```

<Br>

### 서버리스 이벤트 기반 아키텍처 : Fargate vs lambda

---

- 보통 S3와 람다 함수를 사용하는 것이 일반적이지만, 이와 같은 장기 실행 데이터 처리 작업 및 계산 작업은 제한 시간이 있는 람다 함수에 비해 제**한시간이 없는 Fargate가 적합하다.**
- ECS는 작업과 서비스를 실행할 수 있는데, 서비스는 작업으로 구성되며 일반적으로 서비스가 특정 작업 집합을 계속 실행한다는 점에서 장기 실행이 가능하다. **작업은 컨테이너가 시작하고 일부 데이터를 처리한 다음 완료되면 정상적으로 종료**한다.
  - 해당 레시피에서는 S**3 이벤트에 대한 응답으로 컨테이너가 작업을 시작해 파일을 처리한 뒤 종료**한다.

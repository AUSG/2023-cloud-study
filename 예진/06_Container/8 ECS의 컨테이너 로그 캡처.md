# 8. ECS의 컨테이너 로그 캡처

### 실습

---

- 컨테이너에서 실행 중인 애플리케이션의 로그를 검사하자
- CloudWatch(모니터링 및 관찰 서비스)로 로그를 보내고자 컨테이너의 ECS 작업 정의 내에서 awslogs 드라이버를 구성 → 컨테이너가 CloudWatch 로그를 작성할 수 있도록 IAM 역할을 제공하면 컨테이너 로그를 CloudWatch로 스트리밍 할 수 있다.

![image](https://user-images.githubusercontent.com/49095587/230257954-e0c93433-dcd5-4dcc-82f8-18eaaca1ae7e.png)

1. 정책 연결 : `AmazonECSTaskExecutionRolePolicy`
2. Cloudwath에서 로그 그룹 생성 : `aws logs create-log-group`
3. taskdef.json으로 파일 생성

   ```bash
   {
       "networkMode": "awsvpc",
       "containerDefinitions": [
           {
               "portMappings": [
                   {
                       "hostPort": 80,
                       "containerPort": 80,
                       "protocol": "tcp"
                   }
               ],
               "essential": true,
               "entryPoint": [
                   "sh",
                   "-c"
               ],
               **"logConfiguration": {
                   "logDriver": "awslogs",
                   "options": {
                       "awslogs-group": "AWSCookbook608ECS",
                       "awslogs-region": "us-east-1",
                       "awslogs-stream-prefix": "LogStream"
                   }
               },**
               "name": "awscookbook608",
               "image": "httpd:2.4",
               "command": [
                   "/bin/sh -c \"echo 'Hello AWS Cookbook Reader, this container is running on ECS!'  > /usr/local/apache2/htdocs/index.html && httpd-foreground\""
               ]
           }
       ],
       "family": "awscookbook608",
       "requiresCompatibilities": [
           "FARGATE"
       ],
       "cpu": "256",
       "memory": "512"
    }
   ```

4. IAM 역할과 ECS 작업 정의 구성을 사용해 ECS 작업을 생성하고 IAM 역할 연결 : `aws ecs register-task-definition`
5. CDK를 사용해 생성한 ECS 클러스터에서 ECS 작업을 실행

   ```bash
   aws ecs run-task —cluster <<클러스터 이름>> —launch-type FARGATE --network-configuration "awsvpcConfiguration={subnets=[$VPC_PUBLIC_SUB NETS],secrutiryGroups=[$VPC_DEFAULT_SECURITY_GROUP],assignPublicIp=ENABLED}" --task-definition awscookbook608
   ```

6. describe-tasks 명령의 출력으로 RUNNING 상태 확인
7. 로그 확인 : `aws logs describe-log-streams —log-group-name AWSCookbook608ECS`

<br>

### 정리

---

- 레시피 정리 : ECS 작업이 CloudWatch에 로그를 생성할 수 있도록 awslogs 드라이버와 IAM 역할을 구성했다.
- 대부분의 aws 서비스가 서로 통신하려면 통신에 필요한 수준의 권한을 허용하는 역할을 할당해야한다. : 컨테이너는 awslogs logConfiguration 드라이버를 통해 CloudWatchLogs 작업을 허용하는 역할이 연결되어 있어야 한다.
- CloudWatch 로그는 여러가지 AWS 서비스에 대한 중앙 로깅 솔루션을 제공한다.
  - 여러 컨테이너를 실행할 때 디버깅을 위해 로그를 빠르게 검색할 수 있어야 한다.

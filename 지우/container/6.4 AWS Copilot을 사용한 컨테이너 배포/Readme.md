# 6.4 AWS Copilot을 사용한 컨테이너 배포

> _AWS Copilot으로 애플리케이션을 빠르게 배포할 수 있다_

**⍥ Reference**

[AWS Copilot을 사용하여 Amazon ECS 시작하기](https://docs.aws.amazon.com/ko_kr/AmazonECS/latest/developerguide/getting-started-aws-copilot-cli.html)

<br>

① Copilot CLI 툴 설치

```bash
brew install aws/tap/copilot-cli
```

② Copilot은 사용자 대신 ECS가 작업을 수행할 수 있도록 하는 ECS 서비스 연결 역할(service-linked-role)을 사용

```bash
# 서비스 역할이 생성되어 있는지 확인
aws iam list-roles --path-prefix /aws-service-role/ecs.amazonaws.com

# ECS 서비스 연결 역할이 없는 경우
aws iam create-service-linked-role --aws-service-name ecs.amazonaws.com
```

③ AWS Copilot을 사용해 NGINX Dockerfile을 Amazon ECS에 배포

```bash
copilot init --app web --name nginx --type 'Load Balanced Web Service' \
	--dockerfile './Dockerfile' --port 80 --deploy
```

---

🥕 **유효성 검사** : 배포된 서비스에 대한 정보 확인

```bash
copilot svc show
```

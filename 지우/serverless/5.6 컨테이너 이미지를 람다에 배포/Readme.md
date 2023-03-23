# 5.6 컨테이너 이미지를 람다에 배포

> _기존 컨테이너 기반 개발 프로세스와 도구로 서버리스 함수를 배포한다_

① ECR 정보를 도커에 전달

```bash
aws ecr get-login-password | docker login --username AWS \
	--password-stdin $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com

>> Login Succeeded
```

② 람다에서 실행할 `app.py` 파일의 Dockerfile 생성

```python
import sys
def handler(event, context):
    return 'Hello from the AWS Cookbook ' + sys.version + '!'
```

```docker
FROM public.ecr.aws/lambda/python:3.8

COPY app.py   ./
CMD ["app.handler"]
```

③ 컨테이너 이미지 빌드

```bash
docker build -t aws-cookbook506-image .
```

④ 이미지에 태그 추가하여 ECR에 푸시

```bash
# 이미지 태그
docker tag \
	aws-cookbook506-image:latest \
	$AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/aws-cookbook-506repo:latest

# ECR push
docker push \
	$AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/aws-cookbook-506repo:latest
```

⑤ 도커 이미지의 url을 람다의 ImageUri 값으로 지정하여 함수 생성

- 컨테이너 이미지 함수에서는 `--runtime` 및 `--handler` 매개변수를 사용하지 않는다.

```bash
LAMBDA_ARN=$(aws lambda create-function \
	--function-name AWSCookbook506Lambda \
	--package-type "Image" \
	--code ImageUri=$AWS_ACCONT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/aws-cookbook-506repo:latest
	--role \
	arn:aws:iam::$AWS_ACCOUNT_ID:role/AWSCookbookLambdaRole \
	--output text --query FunctionArn)
```

---

**🥕 유효성 검사** : 콘솔에서 람다 함수가 배포된 것 확인 후 실행 테스트

```bash
aws lambda invoke \
	--function-name $LAMBDA_ARN response.json && cat.response.json
```

**🥕 참고**

⍢ 람다는 컨테이너 이미지에 애플리케이션 코드를 패키징 후 배포할 수 있는 기능을 제공

- 컨테이너 이미지 사용 시, 최대 10GB 크기의 애플리케이션 코드를 패키징할 수 있다.
- 람다 함수와 컨테이너 이미지가 저장된 ECR 레포와 동일 계정이어야 한다.

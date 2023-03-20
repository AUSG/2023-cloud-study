# 5.2 람다 계층을 사용한 라이브러리 패키징

> _외부 라이브러리를 사용하는 람다 함수 배포 시, 패키지를 폴더에 생성 및 압축하여 람다 레이어를 생성한다_

① `lambda_function.py` 압축

```bash
zip lambda_function.zip lambda_function.py
```

② 계층을 사용할 람다 함수 생성

```bash
LMABDA_ARN=$(aws lambda create-function \
	--function-name AWSCookbook502Lambda \
	--runtime python3.8 \
	--package-type "Zip" \
	--zip-file fileb://lambda_function.zip \
	--handler lambda_function.lambda_handler --publish \
	--role \ arn:aws:iam::$AWS_ACCOUNT_ID:role/AWSCookbookLambdaRole \
	--output text --query FunctionArn)
```

③ 계층 내용을 복사할 디렉토리와 requests 모듈 설치 → 디렉토리 압축

```bash
mkdir python && pip install requests --target"./python"

zip -r requests-layer.zip ./python
```

④ 계층을 게시하고 계층 ARN을 환경 변수에 저장

```bash
LAYER_VERSION_ARN=$(aws lambda publish-layer-version \
	--layer-name AWSCookbook502RequestsLayer \
	--description "Requests Layer" \
	--license-info "MIT" \
	--zip-file fileb://requests-layer.zip \
	--compatible-runtimes python3.8 \
	--output text --query LayerVersionArn)
```

⑤ 람다 함수가 앞서 생성한 계층을 사용할 수 있도록 업데이트

```bash
aws lambda update-function-configuration  \
	--function-name AWSCookbook502Lambda \
	--layers $LAYER_VERSION_ARN
```

---

**🥕 유효성 검사** : 함수 테스트

```bash
aws lambda invoke \
	--function-name AWSCookbok502Lambda \
	response.json && cat response.json
```

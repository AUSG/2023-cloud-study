# 5.8 프로비저닝된 동시성을 사용해 람다 시작 시간 단축

> _프로비저닝된 동시성을 설정하여 여러 서버리스 함수를 콜드 스타트 없이 빠르게 호출할 수 있다_

① `lambda_function.py`를 압축하여 람다 함수 생성

```bash
zip lambda_function.zip lambda_function.py

aws lambda create-function \
	--function-name AWSCookbook508Lambda \
	--runtime python3.8 \
	--package-type "Zip" \
	--zip-file fileb://lambda_function.zip \
	--handler lambda_function.lambda_handler --publish \
	--timeout 20 \
	--role \
	arn:aws:iam::$AWS_ACCOUNT_ID:role/AWSCookbookLambdaRole
```

```bash
# 람다 함수가 활성화된 것 확인
aws lambda fet-function --function-name AWSCookbook508Lambda \
	--output text --query Configuration.State
```

② 람다 함수의 프로비저닝된 동시성 설정을 구성

```bash
aws lambda put-provisioned-concurrency-config \
	--function-name AWSCookbook508Lambda \
	--qualifier LATEST \
	--provisioned-concurrent-executions 5
```

---

**🥕 유효성 검사** : 함수를 연속으로 6번 호출

```bash
aws lambda invoke --function-name AWSCookbook508Lambda response.json
```

🥕 **참고**

⍢ 함수가 많은 동시성을 달성해야 하는 경우, 프로비저닝 동시성 기능으로 콜드 스타트를 피하고 복제한 실행 환경을 웜 상태로 유지할 수 있다.

- 콜드 스타트 : 함수 코드 요청 시 프로비저닝으로 인해 실행에 약간의 딜레이가 생기는 현상
- 프로비저닝 동시성 기능으로 함수 호출에 소요되는 시간을 최소화할 수 있다.

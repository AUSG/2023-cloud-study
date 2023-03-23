# 5.5 AWS Signer를 사용한 람다 코드의 무결성 확인

> _람다 함수 코드의 무결성을 확인하고 코드가 서명된 후 수정되지 않았음을 확인하려면 서명 프로필을 생성하고 AWS Signer를 사용해서 코드에 서명을 활성화해야 한다_

① 람다 함수에서 사용할 압축 코드의 S3 객체 버전을 가져와 환경 변수에 저장

```bash
OBJ_VER_ID=$(aws s3api list-object-versions \
	--bucket awscookbook505-src-$RANDOM_STRING \
	--prefix lambda_function.zip \
	--output text --query Versions[0].VersionId)
```

② 서명 프로필 생성

```bash
SIGNING_PROFILE_ARN=$(aws signer put-signing-profile \
	--profile-name AWSCookbook505_$RANDOM_STRING \
	--platform AWSLambda-SHA384-STRING \
	--output text --query arn)

# 사용 가능한 서명 플랫폼 목록
aws signer list-signing-platforms
```

③ 람다 함수에 서명 프로필을 참고하는 코드 서명 구성을 생성

```bash
CODE_SIGNING_CONFIG_ARN=$(aws lambda create-code-signing-config \
	--allowed-publishers SigningProfileVersionArns=$SIGNING_PROFILE_ARN \
	--output text --query CodeSigningConfig.CodeSigningConfigArn)
```

④ 서명 작업을 시작

```bash
SIGNING_JOB_ID=$(aws signer start-signing-job \
	--source 's3={bucketName=awscookbook505-src-'"${RANDOM_STRING}"',key=lambda_function.zip,version='"$OBJ_VER_ID"'}' \
	--destination 's3={bucketName=awscookbook505-dst-'"${RANDOM_STRING}"',prefix=signed-}' \
	--profile-name AWSCookbook505_$RANDOM_STRING \
	--output text --query jobId)

# 서명 작업이 성공했는지 확인
aws signer list-signing-jobs --status Succeeded
```

⑤ 서명된 코드의 S3 객체 키를 확인하고 서명된 코드를 사용하는 람다 함수 생성

```bash
OBJECT_KEY=$(aws s3api list-objects-v2 \
	--bucket awscookbook505-dst-$RANDOM_STRING

LAMBDA_ARN=$(aws lambda create-function \
	--function-name AWSCookbook504Lambda \
	--runtime python3.8 \
	--package-type "Zip" \
	--code S3Bucket=awscookbook505-dst-$RANDOM_STRING,S3Key=$OBJECT_KEY \
	--code-signing-config-arn $CODE_SIGNING_CONFIG_ARN \
	--handler lambda_function.lambda_handler --publish \
	--role \ arn:aws:iam::$AWS_ACCOUNT_ID:role/AWSCookbookLambdaRole \
	--output text --query FunctionArn)
```

⑥ 람다 함수가 활성화된 것 확인

```bash
aws lambda get-function --function-name $LAMBDA_ARN \
	--output text --query Configuration.State
```

---

**🥕 유효성 검사** : 콘솔에서 ‘함수에 서명된 코드가 있어 인라인으로 편집할 수 없습니다’라는 메시지 확인

**🥕 참고**

⍢ AWS Signer에서 생성한 디지털 서명으로 코드를 검증하고 시행 정책을 적용하여 코드 배포 및 실행을 제한할 수 있다. ⇒ 주어진 환경에 신뢰할 수 있는 코드만 배포하도록 하는 리소스

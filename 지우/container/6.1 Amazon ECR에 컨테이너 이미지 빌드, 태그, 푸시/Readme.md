# 6.1 Amazon ECR에 컨테이너 이미지 빌드, 태그, 푸시

> _Dockerfile을 만들고 도커 이미지 빌드하여 ECR 저장소에 푸시한다_

<br>

**⍥ Console**

Elastic Container Registry console → **[ 레포지토리 만들기 ]** → 이름 입력 후 **[ 레포지토리 생성 ]** 클릭

<br>

**⍥ CLI**

① repository 생성

```bash
REPO=aws-cookbook-repo && \
	aws ecr create-repository --repository-name $REPO
```

② Dockerfile 생성

- `COPY`, `ADD` 등의 command 추가 가능

```bash
echo FROM nginx:latest > Dockerfile
```

③ 이미지 빌드 + 태그 추가

```bash
docker build . -t \
	$AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/aws-cookbook-repo:latest

docker tag \
	$AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/aws-cookbook-repo:latest \
	$AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/aws-cookbook-repo:1.0
```

④ 도커 로그인 정보 불러오기

```bash
aws ecr get-login-password | docker login --username AWS \
	--password-stdin $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com

>> Login Succeeded
```

⑤ 각 이미지 태그 ECR에 푸시

```bash
docker push \
	$AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/aws-cookbook-repo:latest

docker push \
	$AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/aws-cookbook-repo:1.0
>> 'Layer already exists'
```

---

**🥕 유효성 검사** : ECR에 푸시한 이미지 확인

```bash
aws ecr list-images --repository-name aws-cookbook-repo
```

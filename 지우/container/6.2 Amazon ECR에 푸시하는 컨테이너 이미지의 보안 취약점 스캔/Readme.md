# 6.2 Amazon ECR에 푸시하는 컨테이너 이미지의 보안 취약점 스캔

> _자동 이미지 스캔을 활성화하여 ECR에 이미지를 푸시할 때마다 이미지의 보안 취약점을 자동으로 스캔한다_

<img src="https://user-images.githubusercontent.com/70079416/228551412-76de6d89-9f42-405d-a6b8-c310ba367cb3.png" width=60% height=60%>

<br>

① 도커 이미지 불러오기

- 앞서 배포했던 nginx 컨테이너 이미지 사용

```bash
docker pull nginx:1.14.1
```

② 미리 생성한 repository에 스캔 구성 적용

- (콘솔에 따르면) 리포지토리 수준의 ScanOnPush 구성은 더 이상 사용되지 않고 레지스트리 수준 스캔 필터로 대체된다고 함

```bash
REPO=aws-cookbook-repo && \
	aws ecr put-image-scanning-configuration \
	--repository-name $REPO \
	--image-scanning-configuration scanOnPush=true
```

③ 도커 로그인 정보 설정

```bash
aws ecr get-login-password | docker login --username AWS \
	--password-stdin $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com
```

④ ECR에 푸시할 수 있도록 이미지에 태그 적용

```bash
docker tag nginx:1.14.1 \
	$AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/aws-cookbook-repo:old
```

⑤ 이미지 푸시

```bash
docker push \
	$AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/aws-cookbook-repo:old
```

---

**🥕 유효성 검사** : JSON 형식의 스캔 결과 확인

```bash
aws ecr describe-image-scan-findings \
	--repository-name aws-cookbook-repo --image-id imageTag=old
```

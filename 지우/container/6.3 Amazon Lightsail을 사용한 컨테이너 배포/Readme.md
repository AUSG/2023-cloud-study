# 6.3 Amazon Lightsail을 사용한 컨테이너 배포

> _애플리케이션을 빠르게 배포할 수 있는 Lightsail을 사용하여 컨테이너를 배포한다_

<br>

① Lightsailctl을 설치한 뒤 새 컨테이너 서비스 생성

```bash
aws lightsail create-container-service \
	--service-name awscookbook --power nano --scale 1
```

② TCP 포트 80을 사용하는 nginx 컨테이너 이미지 불러오기

```bash
docker pull nginx

# 서비스 STATUS: READY인지 확인
aws lightsail get-container-services --service-name awscookbook
```

③ 컨테이너 서비스가 준비되면 컨테이너 이미지를 Lightsail에 푸시

```bash
aws lightsail push-container-image --service-name awscookbook \
	--label awscookbook --image nginx
```

④ 컨테이너 서비스와 푸시한 이미지를 연결한 `lightsail.json` 파일 생성

```json
{
  "serviceName": "awscookbook",
  "containers": {
    "awscookbook": {
      "image": ":awscookbook.awscookbook.1",
      "ports": {
        "80": "HTTP"
      }
    }
  },
  "publicEndpoint": {
    "containerName": "awscookbook",
    "containerPort": 80
  }
}
```

⑤ 배포(deployment) 생성

- 유효성 검사를 위해 엔드포인트 URL을 기록한다

```bash
aws lightsail create-container-service-deployment \
	--service-name awscookbook --cli-input-json file://lightsail.json

# 컨테이너 서비스의 STATUS: ACTIVE인지 확인
aws lightsail get-container-services --service-name awscookbook
```

---

**🥕 유효성 검사** : browser 또는 curl을 사용하여 엔드포인트 URL에 접근

```bash
curl <<URL endpoint>>
```

<br>

🥕 **참고**

⍢ **Lightsail**

- TLS 인증서, 로드밸런서, 컴퓨팅, 스토리지를 직접 관리
- 애플리케이션 상태를 확인하고 응답하지 않으면 새로운 컨테이너를 자동으로 배포한다.
- 사용자 지정 domain alias로 SEO URL 제공

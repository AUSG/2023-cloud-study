# 3. Amazon Lightsail을 사용한 컨테이너 배포

### 실습

---

- 컨테이너 기반 애플리케이션을 신속하게 배포하고 안전하게 접근할 수 있도록 구성하자
- AWS에 애플리케이션을 빠르게 배포할 수 있는 방법을 제공하는 Lightsail를 사용해 nginx 컨테이너를 배포하자

![image](https://user-images.githubusercontent.com/49095587/230257226-1fe13de5-c4d8-4f81-b8c2-8787967ffbd3.png)

1. 사전준비 : AWS CLI용 Lightsail Control 플러그인 설치
2. 새 컨테이너 서비스 생성

   ```bash
   aws lightsail create-container-service --service-name awscookbook --power nano --scale 1
   ```

3. 포트 80/TCP를 사용하는 NGINX 컨테이너 이미지를 가져온다.

   ```bash
   docker pull nginx
   ```

   컨테이너 서비스의 상태가 READY 상태가 됐는지 확인

   ```bash
   aws lightsail get-container-services --service-name awscookbook
   ```

4. 컨테이너 서비스가 준비되면 컨테이너 이미지를 Lightsail로 푸시

   ```bash
   aws lightsail push-container-image --service-name awscookbook --label awscookbook --imagenginx
   ```

   ```bash
   # 출력 결과
   Image "nginx" registered.
   Refer to this image as ":awscookbook.awscookbook.1" in deployments.
   ```

<aside>

✏️ Amazon Lightsail은 **퍼블릭 이미지 리포지토리** 를 사용하거나 **Lightsail 내의 컨테이너 서비스** 에 컨테이너 이미지를 푸시할 수 있다.

</aside>

1. 이제 배포를 위해 생성된 컨테이너 서비스와 푸시한 이미지를 연결 (JSON 파일 생성)

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

2. 배포 생성

   ```bash
   aws lightsail create-container-service-deployment --service-name awscookbook --cli-input-json file://lightsail.json
   ```

   컨테이너 서비스의 ACTIVE 상태 확인

   ```bash
   aws lightsail get-container-services --service-name awscookbook
   ```

   엔드포인트 URL을 기록 : [https://awscookbook.18vljne9h5dfg.us-east-1.cs.amazonlightsail.com/](https://awscookbook.18vljne9h5dfg.us-east-1.cs.amazonlightsail.com/)

3. 이제 브라우저 또는 curl을 사용해 엔드포인트 URL에 접근한다.

   ```bash
   curl <<URL endpoint>>
   ```

   ```html
   <!DOCTYPE html>
   <html>
     <head>
       <title>Welcome to nginx!</title>
       <style>
         html {
           color-scheme: light dark;
         }
         body {
           width: 35em;
           margin: 0 auto;
           font-family: Tahoma, Verdana, Arial, sans-serif;
         }
       </style>
     </head>
     <body>
       <h1>Welcome to nginx!</h1>
       <p>
         If you see this page, the nginx web server is successfully installed
         and working. Further configuration is required.
       </p>

       <p>
         For online documentation and support please refer to
         <a href="http://nginx.org/">nginx.org</a>.<br />
         Commercial support is available at
         <a href="http://nginx.com/">nginx.com</a>.
       </p>

       <p><em>Thank you for using nginx.</em></p>
     </body>
   </html>
   ```

   ![image](https://user-images.githubusercontent.com/49095587/230257253-1819ab4c-3ef2-4ddb-bba7-54ab9d6469c3.png)

   <br>

### Amazon Lightsail

---

- 가상 프라이빗 서버 서비스
- AWS 클라우드에서 호스팅하는 가상 서버 인스턴스를 쉽게 설정하고, 모니터링하고, 확장하고, 백업할 수 있도록 도와준다.
- TLS 인증서, 로드밸런서, 컴퓨팅, 스토리지를 직접 관리한다.
- 애플리케이션이 MySQL, PostgreSQL 데이터베이스를 사용한다면 배포의 일부로 데이터베이스를 관리할 수 있다.
- Lightsail은 애플리케이션의 상태를 확인하고 응답하지 않는 경우 새로운 컨테이너를 자동 배포
- 단기간에 일반적인 컨테이너 기반 애플리케이션을 배포할 숭 ㅣㅅ다.

### 레시피 정리

---

1. lightsail에 컨테이너 서비스를 만들고 컨테이너 이미지를 lightsail에 푸시
2. 컨테이너 서비스와 이미지를 연결
3. 컨테이너 서비스 배포

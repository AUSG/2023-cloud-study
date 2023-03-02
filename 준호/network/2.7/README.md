## 2.7 Application Load Balancer를 사용해 HTTP 트래픽을 HTTPS로 리디렉션

### 문제 설명

프라이빗 서브넷에서 컨테이너화된 애플리케이션을 구동하고 있다.
인터넷 사용자가 애플리케이션에 액세스할 때 HTTP의 모든 요청을 HTTPS로 리디렉션해야 한다.

### 해결 방법

1. ALB를 생성한다.
2. ALB에서 포트 80과 포트 443에 대한 리스너를 생성하고 컨테이너화된 애플리케이션의 대상 그룹 및 리스너 규칙을 작성한다.
3. 그림 2-9와 같이 대상 그룹에 트래픽을 보내도록 리스너 규칙을 구성한다.
4. 요청의 URL을 유지하면서 포트 80(HTTP)의 트래픽을 HTTP 301 응답 코드를 통해 포트 443(HTTPS)으로 리디렉션하는 작업을 구성한다.

### 작업 방법

1. 인증서에 사용할 개인 키 생성
2. openssl cli를 이용해 자체 서명된 인증서 생성
3. 생성된 인증서 IAM에 업로드

```bash
CERT_ARN=$(aws iam upload-server-certificate --server-certificate-name aws207 --certificate-body file://my-certificate.pem --private-key file://my-private-key.pem --query ServerCertificateMetadata.Arn --output text)
```

4. ALB에 사용할 보안 그룹 생성

```bash
ALB_SG_ID=$(aws ec2 create-security-group --group-name aws207sg --description "ALB Security Group" --vpc-id $VPC_ID --output text --query GroupId)
```

5. 보안 그룹에 HTTP 및 HTTPS 트래픽을 허용하는 규칙 추가

```bash
aws ec2 authorize-security-group-ingress \
   --protocol tcp --port 80 \
   --cidr '0.0.0.0/0' \
   --group-id $ALB_SG_ID

aws ec2 authorize-security-group-ingress \
   --protocol tcp --port 443 \
   --cidr '0.0.0.0/0' \
   --group-id $ALB_SG_ID
```

6. ALB에서 들어오는 트래픽을 허용하도록 컨테이너의 보안 그룹에 권한 부여

```bash
# APP_SG_ID가 없음. 컨테이너 필요
aws ec2 authorize-security-group-ingress \
   --protocol tcp --port 80 \
   --source-group $ALB_SG_ID \
   --group-id $APP_SG_ID
```

7. 퍼블릭 서브넷에 ALB를 생성하고 이전에 생성한 보안 그룹을 연결한다.

```bash
LOAD_BALANCER_ARN=$(aws elbv2 create-load-balancer --name aws207alb --subnets $VPC_PUBLIC_SUBNETS --security-groups $ALB_SG_ID --scheme internet-facing --output text --query LoadBalancers.LoadBalancerArn)
```

8. 로드 밸런서에 적용할 대상 그룹을 생성한다.
9. 애플리케이션을 실행하는 컨테이너의 IP를 확인한다.
10. 대상 그룹에 컨테이너를 등록한다.
11. ALB에 앞서 생성한 인증서를 사용하며, 트래픽을 대상 그룹으로 전달하는 리스너를 생성한다.
12. 포트 443의 리스너에 대한 규칙을 추가해 생성한 대상 그룹으로 트래픽을 전달한다.
13. HTTPS 리디렉션에 대한 전체 URL을 유지하면서 브라우저에 301 응답을 보내는 모든 HTTP 트래픽에 리디렉션을 생성한다.
14. 대상의 상태를 확인한다.

# 2.1 Amazon VPC를 사용해 프라이빗 가상 네트워크 생성
해당 리전에 vpc를 만들고 그에 대한 CIDR(classless inter-domain routing) 블록 구성

``` bash
# ipv4 CIDR 블록을 가진 VPC 생성
VPC_ID=$(aws ec2 create-vpc --cidr-block 10.10.0.0/16 --tag-specifications 'ResourceType=vpc, Tags=[{Key=Name, Value=aws201}]' --output text --query Vpc.VpcId)

# VPC 상태 확인
aws ec2 describe-vpcs --vpc-ids $VPC_ID

# CIDR 블록을 한번 VPC와 연결하면 확장할 수는 있지만 수정은 불가능. IPv4 공간 추가
aws ec2 describe-vpcs --vpc-ids $VPC_ID

# VPC를 ipv6으로 만들기
aws ec2 create-vpc --cidr-block 10.11.0.0/16 --amazon-provided-ipv6-cidr-block --tag-specifications 'ResourceType=vpc,Tags=[{Key=Name, Value=aws201-ipv6}]'
```

# 2.2 서브넷과 라우팅 테이블 포함한 네트워크 티어 생성
리소스의 분할 및 중복을 위해 개별 ip 공가능로 구성한 vpc 네트워크 생성 챕터.
 하나의 vpc에 있는 2개의 AZ에 각각 subnet을 하나씩 생성하고 이를 라우팅 테이블을 이용해서 트래픽을 처리하는 방식.
 ``` bash
 # vpc는 이전 챕터에서 만든걸 이용하고 먼저 라우팅 테이블 생성
 ROUTE_TABLE_ID_1=$(aws ec2 create-route-table --vpc-id $VPC_ID --tag-specifications 'ResourceType=route-table, Tags=[{Key=Name,Value=aws202}]' --output text --query RouteTable.RouteTableId)

 # 각 AZ에 서브넷을 생성
 SUBNET_ID_1=$(aws ec2 create-subnet --vpc-id $VPC_ID --cidr-block 10.10.0.0/24 --availability-zone us-west-2a --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=aws202a}]' --output text --query Subnet.SubnetId)

 SUBNET_ID_2=$(aws ec2 create-subnet --vpc-id $VPC_ID --cidr-block 10.10.1.0/24 --availability-zone us-west-2b --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=aws202b}]' --output text --query Subnet.SubnetId)

 # AZ의 매개변수로 리전 뒤에 a나 b를 추가해서 각 서브넷을 프로비저닝할 논리적 AZ를 지정. AWS에서는 AZ 간에 리소스 균형을 맞추고자 계정별로 이름과 가용 영역을 무작위로 지정.

 # 라우팅 테이블을 서브넷과 연결
 aws ec2 associate-route-table --route-table-id $ROUTE_TABLE_ID --subnet-id $SUBNET_ID_1

 aws ec2 associate-route-table --route-table-id $ROUTE_TABLE_ID --subnet-id $SUBNET_ID_2

 # 두 서브넷의 AZ과 라우팅 테이블에 대해 확인하는 코드
 aws ec2 describe-subnets --subnet-ids $SUBNET_ID_1
 aws ec2 describe-subnets --subnet-ids $SUBNET_ID_2
 aws ec2 describe-route-tables --route-table-ids $ROUTE_TABLE_ID

 # 서브넷 설계할 때는 현재 요구사항에 맞는 서브넷 크기를 선택해야함(너무 크게 설정해서도 안되고 너무 적게 설정해서도 안됨. 너무 크면 리소스가 낭비되고 작으면 리소스가 부족하게 됨).
 # 리전에서 VPC를 생성해라 때 해당 네트워크 계층의 AZ에 서브넷을 분산하는 것이 모범사례. 보통 3개 이상의 AZ를 갖고 있고 2개를 쓰는곳도 있다고는 들음. 보통 2-3개가 일반적인 사례인것 같음.
 ```

 # 2.3 인터넷 게이트웨이를 사용해 VPC를 인터넷에 연결

``` bash
# IGW 생성
INET_GATEWAY_ID=$(aws ec2 create-internet-gateway --tag-specifications 'ResourceType=internet-gateway,Tags=[{Key=Name,Value=aws203}]' --output text --query InternetGateway.InternetGatewayId)

# IGW를 기존 VPC에 연결
aws ec2 attach-internet-gateway --internet-gateway-id $INET_GATEWAY_ID --vpc-id $VPC_ID

# 새로운 라우팅 테이블을 만들고 해당 라우팅 테이블을 서브넷에 연결
aws
 ROUTE_TABLE_ID_2=$(aws ec2 create-route-table --vpc-id $VPC_ID --tag-specifications 'ResourceType=route-table, Tags=[{Key=Name,Value=aws203b}]' --output text --query RouteTable.RouteTableId)
# VPC의 각 라우팅 테이블에서 기본 경로 대상을 인터넷 게이트웨이로 설정하는 경로 생성(IGW와 연결된 경로가 0.0.0.0/0인 서브넷은 퍼블릿 서브넷으로 간주)
aws ec2 create-route --route-table-id $ROUTE_TABLE_ID_1 --destination-cidr-block 0.0.0.0/0 --gateway-id $INET_GATEWAY_ID

aws ec2 create-route --route-table-id $ROUTE_TABLE_ID_2 --destination-cidr-block 0.0.0.0/0 --gateway-id $INET_GATEWAY_ID

# EIP(탄력적 ip) 생성
ALLOCATION_ID=$(aws ec2 allocate-address --domain vpc --output text --query AllocationId)

# EIP를 기존 EC2 인스턴스에 연결
aws ec2 associate-address --instance-id $INSTANCE_ID --allocation-id $ALLOCATION_ID

# 그 다음 SSM session manager를 사용해 EC2 인스턴스에 연결해야하는데 또 잘안돼서 그냥 ssh로 들어가서 따라함.

```

# 2.4 NAT 게이트웨이 사용한 프라이빗 세브넷의 외부 인터넷 접근
``` bash
# 먼저 이전에 만들었던 방식으로 프라이빗 서브넷을 생성하고 라우팅 테이블을 서브넷에 연결
# NAT 게이트웨이 사용할 탄력적 ip 주소 생성
ALLOCATION_ID=$(aws ec2 allocate-address --domain vpc --output text --query AllocationId)

# AZ2의 퍼블릭 서브넷 내에 nat 게이트웨이 생성
NAT_GATEWAY_ID=$(aws ec2 create-nat-gateway --subnet-id $SUBNET_ID_2 --allocation-id $ALLOCATION_ID --output text --query NatGateway.NatGatewayId)

# 상태 확인(콘솔로 보는게 나을듯)
aws ec2 describe-nat-gateways --nat-gateway-ids $NAT_GATEWAY_ID --output text --query NatGateways.State

# nat 게이트웨이의 0.0.0.0/0에 대한 기본 경로를 프라이빗 서브네쪽 라우팅 테이블에 추가
aws ec2 create-route --route-table-id $ROUTE_TABLE_ID_3 --destination-cidr-block 0.0.0.0/0 --nat-gateway-id $NAT_GATEWAY_ID

aws ec2 create-route --route-table-id $ROUTE_TABLE_ID_4 --destination-cidr-block 0.0.0.0/0 --nat-gateway-id $NAT_GATEWAY_ID

# 이후 인스턴스에 연결해서 잘되는지 확인
```
이렇게 하나의 AZ에 nat 게이트웨이를 프로비저닝하는 것보다 각 AZ에 프로비저닝하는게 AZ 트리팩 양을 줄이는데 좋고 탄력성 있어짐.
# 2.5 보안 그룹을 참고해 동적으로 접근 권한 부여
``` basg
# 보안 그룹 생성
SG_ID=$(aws ec2 create-security-group --group-name aws205sg --description "Instance Security Group" --vpc-id $VPC_ID --output text --query GroupId)

# 보안 그룹 인스턴스에 연결
aws ec2 modify-instance-attribute --instance-id $INSTANCE_ID_1 --groups $SG_ID
aws ec2 modify-instance-attribute --instance-id $INSTANCE_ID_2 --groups $SG_ID


# tcp 22에 대한 액세스를 허용하는 인바운드 규칙 추가(이렇게 하니 콘솔으로는 ec2인스턴스에 접근이 안됨.. EIP도 주고 했지만 잘안돼서 인바운드 규칙을 0.0.0.0으로 수정하고 진행함. )
aws ec2 authorize-security-group-ingress --protocol tcp --port 22 --source-group $SG_ID --group-id $SG_ID

# 이후 ssm으로 접근해서 다른 인스턴스에 접근하는거지만 ssm 연결이 잘되지도 않고 그래서 EIP를 인스턴스에 연결해서 직접 콘솔로 접근해서 진행.
```

# 2.6 VPC Reachability Analyzer를 활용한 네트워크 경로 확인 및 문제 해결

``` bash
# 시작하기 앞서 사용자 정책에 reachability analyzer 정책을 추가. 안될 경우엔 정책을 인라인으로 추가.

# ec2인스턴스와 tcp 22 지정하는 네트워크 인사이트 경로 생성
INSIGHTS_PATH_ID=$(aws ec2 create-network-insights-path \
    --source $INSTANCE_ID_1 --destination-port 22 \
    --destination $INSTANCE_ID_2 --protocol tcp \
    --output text --query NetworkInsightsPath.NetworkInsightsPathId)

# 두 인스턴스 간의 네트워크 인사이트 분석
ANALYSIS_ID_1=$(aws ec2 start-network-insights-analysis \
    --network-insights-path-id $INSIGHTS_PATH_ID --output text \
    --query NetworkInsightsAnalysis.NetworkInsightsAnalysisId)
aws ec2 describe-network-insights-analyses \
    --network-insights-analysis-ids $ANALYSIS_ID_1
#networkpathFound가 false인것을 확인.

# 인스턴스 2에 연결된 보안 그룹 업데이트. 1의 보안그룹에서 tcp 22로의 액세스를 허용하는 규칙 추가.
aws ec2 authorize-security-group-ingress \
   --protocol tcp --port 22 \
   --source-group $SG_ID \
   --group-id $SG_ID_2

# 인사이트 분석 다시 실행
ANALYSIS_ID_2=$(aws ec2 start-network-insights-analysis \
    --network-insights-path-id $INSIGHTS_PATH_ID --output text \
    --query NetworkInsightsAnalysis.NetworkInsightsAnalysisId)
aws ec2 describe-network-insights-analyses \
    --network-insights-analysis-ids $ANALYSIS_ID_2

# true로 바뀐 것을 확인할 수 있음. 이후엔 전의 방식대로 진행
```
네트워크 연결을 네트워크 인사이트 경로로 정의해 테스트할 수 있음. 처음에는 ssh연결이 허용되지 않았기 때문에 나중에 보안그룹을 업데이트하고 나서 사용할 수 있음. VPC reachability analyzer는 인프라를 프로비저닝할 필요가 없기 때문에 서버리스 방식으로 네트워킁 문제 해결 및 구성의 유효성 검증을 위한 효율적인 도구.

# 2.7 Application Load Balancer를 사용해 HTTP 트래픽을 HTTPS로 리디렉션
``` bash
ALB를 이용해서 HTTP로 오는 요청을 HTTPS로 리다이렉션하는 챕터.

# 인증서에 사용할 새로운 개인 키 생성
openssl genrsa 2048 > my-private-key.pem

# openSSL cli를 사용해 자체 서명된 인증서 생성
openssl req -new -x509 -nodes -sha256 -days 365 -key my-private-key.pem -outform PEM -out my-certificate.pem

# 생성된 인증서를 IAM에 업로드
CERT_ARN=$(aws iam upload-server-certificate --server-certificate-name aws207 --certificate-body file://my-certificate.pem --private-key file://my-private-key.pem --query ServerCertificateMetadata.Arn --output text)

# ALB에 사용할 보안 그룹 생성
ALB_SG_ID=$(aws ec2 create-security-group --group-name aws207sg --description "ALB Security Group" --vpc-id $VPC_ID --output text --query GroupId)

# 보안 그룹에 HTTP 및 HTTPS 트래픽을 허용하는 규칙 추가
aws ec2 authorize-security-group-ingress \
   --protocol tcp --port 80 \
   --cidr '0.0.0.0/0' \
   --group-id $ALB_SG_ID
aws ec2 authorize-security-group-ingress \
   --protocol tcp --port 443 \
   --cidr '0.0.0.0/0' \
   --group-id $ALB_SG_ID

# ALB에서 들어오는 트래픽을 허용하도록 컨테이너의 보안 그룹에 권한 부여
aws ec2 authorize-security-group-ingress \
   --protocol tcp --port 80 \
   --source-group $ALB_SG_ID \
   --group-id $APP_SG_ID #컨테이너 앱의 보안 그룹 id

# 퍼블릭 서브넷에 ALB를 생성하고 이전에 생성한 보안 그룹 연결
LOAD_BALANCER_ARN=$(aws elbv2 create-load-balancer --name aws207alb --subnets $VPC_PUBLIC_SUBNETS --security-groups $ALB_SG_ID --scheme internet-facing --output text --query LoadBalancers.LoadBalancerArn)

# 로드 밸런서에 적용할 대상 그룹 생성
TARGET_GROUP=$(aws elbv2 create-target-group --name aws208tg --vpc-id $VPC_ID --protocol HTTP --port 80 --target-type ip --query 'TargetGroups.TargetGroupArn')

# 컨테이너 ip 확인
TASK_ARN=$(aws ecs list-tasks --cluster $ECS_CLUSTER_NAME --output text --query taskArns)

CONTAINER_IP=$(aws ecs describe-tasks --cluster $ECS_CLUSTER_NAME --task $TAKS_ARN --output text --query tasks[0].attachments[0].details[4 | cut -f 2])

# 대상 그룹에 컨테이너를 등록
aws elbv2 register-targets --targets Id=$CONTAINER_IP --target-group-arn $TARGET_GROUP

# ALB에 인증서를 사용하고 트래픽을 대상 그룹으로 전달하는 리스너를 생성
HTTPS_LISTENER_ARN=$(aws elbv2 create-listener --load-balancer-arn $LOAD_BALANCER_ARN --protocol HTTPS --port 443 certificates CertificateArn=$CERT_ARN --default-actions Type=forward,TargetGroupArn=$TARGET_GROUP --output text --query Listeners[0].ListemerArm)

# 포트 443의 리스너에 대한 규칙을 추가해 생성한 대상 그룹으로 트래픽 전달
aws elbv2 create-rule --listener-arn $HTTPS_LISTENER_ARN --priority 10 --conditions '{"Field":"path-pattern","PathPatternConfig":{"Values":["/*"]}}' --actions Type=forward,TargetGroupArn=$TARGET_GROUP

# 브라우저에 301응답을 보내서 모든 HTTP 트래픽에 대해 리다이렉션 생성
aws elbv2 create-listener --load-balacner-arn $LOAD_BALANCER_ARN --protocol HTTP --port 80 --default-actions "Type=redirect,RedirectConfig={Protocol=HTTPS,Port=443,Host='#{host}',Query='#{query}',Path='/#{path}',StatusCode=HTTP_301}"

# 대상의 상태 확인
aws elbv2 describe-target-health --target-group-arn $TARGET_GROUP --query TargetHealthDescriptions[*].TargetHealth.State

# 로드 밸런서의 URL 확인
LOAD_BALANCER_DNS=$(aws elbv2 describe-load-balancers --name aws207alb --output text --query LoadBalancers[0].DNSName)
echo $LOAD_BALANCER_DNS # 해당 도메인으로 curl 날리면 301코드를 확인할 수 있음.
```
ALB 같은 경우에는 7계층에서 작동하는 로드 밸런서로 연결된 대상 그룹의 구성원에 대해 상태 확인을 지속적으로 수행하고 트래픽을 라우팅할 애플리케이션의 정상적인 구성 요소도 감지. 이오ㅔ에도 네트워크 로드 밸런서, 게이트웨이 로드밸런서가 있음.

# 2.8 접두사 목록을 활용한 보안 그룹의 CIDR 관리
관리형 접두사 목록을 만들어서 보안 그룹에 연결해서 외부에서 접근 가능하도록 만드는 챕터.
``` bash
# 사전 작업
VPC_ID=$(aws ec2 create-vpc --cidr-block 10.10.0.0/16 --tag-specifications 'ResourceType=vpc, Tags=[{Key=Name, Value=aws208}]' --output text --query Vpc.VpcId)
SUBNET_ID_1=$(aws ec2 create-subnet --vpc-id $VPC_ID --cidr-block 10.10.1.0/24 --availability-zone us-west-2a --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=aws202a}]' --output text --query Subnet.SubnetId)
SUBNET_ID_2=$(aws ec2 create-subnet --vpc-id $VPC_ID --cidr-block 10.10.2.0/24 --availability-zone us-west-2b --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=aws202b}]' --output text --query Subnet.SubnetId)
ROUTE_TABLE_ID_1=$(aws ec2 create-route-table --vpc-id $VPC_ID --tag-specifications 'ResourceType=route-table, Tags=[{Key=Name,Value=aws208}]' --output text --query RouteTable.RouteTableId)
ROUTE_TABLE_ID_2=$(aws ec2 create-route-table --vpc-id $VPC_ID --tag-specifications 'ResourceType=route-table, Tags=[{Key=Name,Value=aws208b}]' --output text --query RouteTable.RouteTableId)
aws ec2 associate-route-table --route-table-id $ROUTE_TABLE_ID_1 --subnet-id $SUBNET_ID_1
aws ec2 associate-route-table --route-table-id $ROUTE_TABLE_ID_2 --subnet-id $SUBNET_ID_2
INET_GATEWAY_ID=$(aws ec2 create-internet-gateway --tag-specifications 'ResourceType=internet-gateway,Tags=[{Key=Name,Value=aws208}]' --output text --query InternetGateway.InternetGatewayId)
# 게이트웨이 만들고 서브넷 연결 추가적으로 필요
aws ec2 create-route --route-table-id $ROUTE_TABLE_ID_1 --destination-cidr-block 0.0.0.0/0 --gateway-id $INET_GATEWAY_ID
aws ec2 create-route --route-table-id $ROUTE_TABLE_ID_2 --destination-cidr-block 0.0.0.0/0 --gateway-id $INET_GATEWAY_ID

# aws ip 주소 범위를 가진 json 파일 다운로드

# us-east-1 리전에서 amazon workspaces 게이트웨이 대한 CIDR 범위 목록 생성
jq -r '.prefixes[] | select(.region=="us-east-1") | select(.service=="WORKSPACES_GATEWAYS") | .ip_prefix' < ip-ranges.json

# amazon workspaces에 대한 ip 범위로 관리형 접두사 목록을 생성
PREFIX_LIST_ID=$(aws ec2 create-managed-prefix-list --address-family IPv4 --max-entries 15 --prefix-list-name allowed-us-west-2-cidrs --output text --query "PrefixList.PrefixListId" --entries Cidr=44.234.54.0/23,Description=workspaces-us-east-1-cidr1 Cidr=54.244.46.0/23,Description=workspaces-us-east-1-cidr2)

# 나의 public ipv4주소 확인
MY_ID_4=$(curl myip4.com | tr -d ' ')

# 내 아이피 관리형 접두사 목록에 추가
aws ec2 modify-managed-prefix-list --prefix-list-id $PREFIX_LIST_ID --current-version 1 --add-entries Cidr=${MY_ID_4}/32,Description=my-workstation-ip

# 인스턴스의 보안 그룹에 접두사 목록에 대한 tcp 80 액세스 허용하는 인바운드 규칙 추가
aws ec2 authorize-security-group-ingress \
   --group-id sg-00 --ip-permissions IpProtocol=tcp,FromPort=80,ToPort=80,PrefixListIds="[{Description=http-fromprefix-list,PrefixListId=$PREFIX_LIST_ID}]"

aws ec2 authorize-security-group-ingress \
   --group-id sg-01 --ip-permissions IpProtocol=tcp,FromPort=80,ToPort=80,PrefixListIds="[{Description=http-fromprefix-list,PrefixListId=$PREFIX_LIST_ID}]"

# 관리형 접두사 목록이 사용되는 보안 그룹 확인
aws ec2 get-managed-prefix-list-associations --prefix-list-id $PREFIX_LIST_ID

# 인스턴스에 대한 테스트(안에 nginx 서버 설치해줬음.)
curl -m 2 $INSTANCE_IP
```
보안 그룹을 업데이트하는 것보다 접두사 목록을 업데이트하는게 여러개의 보안 그룹을 관리할 때 효율적. 접두사 목록은 이전 버전으로 돌아가는 롤백도 제공..!

# 2.9 VPC 엔드포인트를 사용한 S3 접근

``` bash
# 먼저 기본적으로 vpc 와 subnet, 라우팅 테이블 생성 및 인스턴스, s3 버킷  생성

# VPC에 게이트웨이 엔드포인트를 생성하고 그 엔드포인트를 라우팅 테이블과 연결
END_POINT_ID=$(aws ec2 create-vpc-endpoint --vpc-id $VPC_ID --service-name com.amazonaws.$AWS_REGION.s3 --route-table-ids $RT_ID_1 $RT_ID_2 --query VpcEndpoint.VpcEndpointId --output text)

# 특정 s3 버킷으로만 접근을 제한하는 엔드포인트 정책 파일 생성
sed -e "s/S3BucketName/${BUCKET_NAME}/g" policy-template.json > policy.json

# vpc의 엔드포인트 정책을 수정
aws ec2 modify-vpc-endpoint --policy-document file://policy.json --vpc-endpoint-id $END_POINT_ID

# 이후는 ec2에 연결해서 인스턴스의 메타데이터 값으로 리전 설정하고 버킷의 이름을 get하고 안의 파일을 복사해서 접근 가능한지 확인.
```

# 2.10 트랜짓 게이트웨이를 사용해 전이 라우팅 연결 활성화
여러 개의 VPC의 라우팅 테이블의 트래픽을 트랜짓 게이트웨이로 보내도록 설정하고 그걸 NAT 게이트웨이와 연결된 vpc에 연결하면 하나의 vpc로만 인터넷에 노출시킬 수 있음.
``` bash
# 기본적으로 vpc3개와 각각에 프라이빗, 퍼블릭 서브넷 필요.

# 트랜짓 게이트웨이 생성(파라미터 문제로 그냥 콘솔로 생성)
TGW_ID=$(aws ec2 create-transit-gateway --description aws210 --options=AmazonSideAsn=65010,AutoAcceptSharedAttachments=enable,DefaultRouteTableAssociation=enable,DefaultRouteTablePropagation=enable,VpcEcmpSupport=enable,DnsSupport=enable --output text --query TransitGateway.TransitGatewayId)

# vpc1에 전송 게이트웨이 연결 생성
TGW_ATTACH_1=$(aws ec2 create-transit-gateway-vpc-attachment --transit-gateway-id $TGW_ID --vpc-id $VPC_ID_1 --subnet-ids $ATTACHMENT_SUBNETS_VPC_1 --query TransitGatewayVpcAttachment.TransitGatewayAttachmentId --output text)
TGW_ATTACH_2=$(aws ec2 create-transit-gateway-vpc-attachment --transit-gateway-id $TGW_ID --vpc-id $VPC_ID_2 --subnet-ids $ATTACHMENT_SUBNETS_VPC_2 --query TransitGatewayVpcAttachment.TransitGatewayAttachmentId --output text)
TGW_ATTACH_3=$(aws ec2 create-transit-gateway-vpc-attachment --transit-gateway-id $TGW_ID --vpc-id $VPC_ID_3 --subnet-ids $ATTACHMENT_SUBNETS_VPC_3 --query TransitGatewayVpcAttachment.TransitGatewayAttachmentId --output text)

# vpc1, 3 의 모든 프라이빗 서브넷에 0.0.0.0/0에 대한 경로를 TGW를 대상으로 추가
aws ec2 create-route --route-table-id $VPC_1_RT_ID_1 --destination-cidr-block 0.0.0.0/0 --transit-gateway-id $TGW_ID
aws ec2 create-route --route-table-id $VPC_1_RT_ID_2 --destination-cidr-block 0.0.0.0/0 --transit-gateway-id $TGW_ID
aws ec2 create-route --route-table-id $VPC_3_RT_ID_1 --destination-cidr-block 0.0.0.0/0 --transit-gateway-id $TGW_ID
aws ec2 create-route --route-table-id $VPC_3_RT_ID_2 --destination-cidr-block 0.0.0.0/0 --transit-gateway-id $TGW_ID

# vpc2의 프라이빗 서브넷의 라우팅 테이블에도 10.10.0.0/24 슈퍼넷에 대한 경로를 추가해 TGW를 가리키도록 설정.
aws ec2 create-route --route-table-id $VPC_2_RT_ID_1 destination-cidr-block 10.10.0.0/24 --transit-gateway-id $TGW_ID
aws ec2 create-route --route-table-id $VPC_2_RT_ID_2 destination-cidr-block 10.10.0.0/24 --transit-gateway-id $TGW_ID

# NAT 게이트웨이 정보 가져오기
NAT_GW_ID_1=$(aws ec2 describe-nat-gateways --filter "Name=subnet-id, Values=$VPC_2_PUBLIC_SUBNET_ID_1" --output text --query NatGateways[*].NatGatewayId)
NAT_GW_ID_2=$(aws ec2 describe-nat-gateways --filter "Name=subnet-id, Values=$VPC_2_PUBLIC_SUBNET_ID_2" --output text --query NatGateways[*].NatGatewayId)

# TGW의 인터넷 트래픽을 NAT 게이트웨이로 보내기 위해 vpc2의 서브넷에 경로 추가
aws ec2 create-route --route-table-id $VPC_2_ATTACH_RT_ID_1 --destination-cidr-block 0.0.0.0/0 --nat-gateway-id $NAT_GW_ID_1
aws ec2 create-route --route-table-id $VPC_2_ATTACH_RT_ID_2 --destination-cidr-block 0.0.0.0/0 --nat-gateway-id $NAT_GW_ID_2

# vpc2의 퍼블릭 서브넷의 라우팅 테이블에 tgw를 추가해서 모든 vpc가 nat 게이트웨이를 공유할 수 있도록 함.
aws ec2 create-route --route-table-id $VPC_2_PUBLIC_RT_ID_1 destination-cidr-block 10.10.0.0/24 --transit-gateway-id $TGW_ID
aws ec2 create-route --route-table-id $VPC_2_PUBLIC_RT_ID_2 destination-cidr-block 10.10.0.0/24 --transit-gateway-id $TGW_ID

# TGW의 라우팅 테이블의 id를 가져와서 vpc2로 경로 추가
TRAN_GW_RT=$(aws ec2 describe-transit-gateways --transit-gateway-ids $TGW_ID --output text --query TransitGateways.Options.AssociationDefaultRouteTableId)

aws ec2 create-transit-gateway-route --destination-cidr-block 0.0.0.0/0 --transit-gateway-route-table-id $TRAN_GW_RT --transit-gateway-attachment-id $TGW_ATTACH_2
```

# 2.11 VPC 간 네트워크 통신을 위한 VPC 피어링 적용
``` bash
# vpc1을 vpc2에 연결하는 vpc 피어링 연결 생성
VPC_PEERING_CONNECTION=$(aws ec2 create-vpc-peering-connection --vpc-id $vpc_id_1 --peer-vpc-id $vpc_id_2 --output text --query VpcPeeringConnection.VpcPeeringConnectionId)

# 피어링 연결 수락
aws ec2 accept-vpc-peering-connection --vpc-peering-connection-id $VPC_PEERING_CONNECTION

# 서브넷과 연결된 기본 라우팅 테이블에 피어링 vpc의 CIDR 범위로 향하는 트래픽을 vpc peering connection id로 보내는 경로로 추가
aws ec2 create-route --route-table-id $vpc_subnet_rt_1 --destination-cidr-block 10.11.0.0/16 --vpc-peering-connection-id $VPC_PEERING_CONNECTION

aws ec2 create-route --route-table-id $vpc_subnet_rt_2 --destination-cidr-block 10.10.0.0/16 --vpc-peering-connection-id $VPC_PEERING_CONNECTION

# 인스턴스 1의 보안그룹의 icpmv4 액세스를 허용하는 규칙을 인스턴스2 보안 그룹에 추가
aws ec2 authorize-security-group-ingress --protocol icmp --port -1 --source-group sg-0a1180cbee092570c --group-id sg-00bbe92fb0fe3f0ec
# 해당 규칙 말고 전체로 해야 가는듯,,

# 이후 인스턴스1에 접속해 인스턴스 2로 다음 명령을 날려서 테스트
ping -c 4 $instance_ip
```
서로 다른 vpc 간의 통신을 위해서는 모든 vpc와 피어링 설정을 해야함. 그런데 vpc를 계속 추가하면 계속 피어링 설정을 해줘야하기 때문에 전이적 vpc 통신에는 앞서 사용했던 트랜짓 게이트웨이 아키텍쳐를 사용.
## 2. Network 용어정리

### AWS VPC(Virtual Private Cloud)

AWS VPC (Virtual Private Cloud)은 AWS (Amazon Web Services)에서 제공하는 클라우드 컴퓨팅 환경에서 사용되는 가상 사설 네트워크(Virtual Private Network)입니다.

AWS VPC는 AWS 클라우드에서 인프라를 배포하고 실행하는 데 사용되며, 사용자는 가상 네트워크를 생성하고, 해당 네트워크 내에서 인스턴스, 서브넷, 라우팅 테이블, 인터넷 게이트웨이 등을 설정할 수 있습니다.

AWS VPC는 네트워크 구성을 완전히 제어할 수 있기 때문에 사용자는 고객의 데이터와 인프라를 보호할 수 있으며, 인터넷과의 연결을 제어하고, 보안 및 액세스 제어 규칙을 구성할 수 있습니다.

또한, AWS VPC는 다른 AWS 서비스와 연동되어 자동화된 네트워크 관리를 지원하며, VPN 연결을 통해 기존 온프레미스 인프라와 연결할 수도 있습니다.

### CIDR(Classless Inter-Domain Routing)

CIDR(Classless Inter-Domain Routing)는 인터넷 주소 체계(IP 주소 체계)에서 IP 주소와 네트워크 마스크를 표현하는 방법 중 하나입니다.

CIDR은 기존의 IP 주소 체계에서 클래스(Class)를 제거하고, 유연하게 IP 주소와 네트워크 마스크를 조합하여 표현할 수 있도록 하는 방식입니다.

CIDR을 사용하면 IP 주소와 네트워크 마스크를 "IP 주소/마스크 비트 수" 형태로 표현할 수 있습니다. 예를 들어, 192.168.0.0/16은 IP 주소가 192.168.x.x인 모든 주소를 나타내며, 마스크는 255.255.0.0입니다.

CIDR은 IP 주소 할당과 라우팅 테이블 구성 등에 사용됩니다. AWS에서는 VPC(Virtual Private Cloud)을 설정할 때, CIDR 블록을 지정하여 가상 네트워크 범위를 설정할 수 있습니다.

### 가용 영역(Availability Zone)

AWS(Amazon Web Services)에서 가용 영역(Availability Zone)은 하나 이상의 데이터 센터를 묶어서 구성된 논리적인 영역을 말합니다. 각 가용 영역은 지리적으로 분리된 위치에 있으며, 별도의 전원 공급, 네트워크 및 연결성을 가지고 있습니다.

가용 영역은 고가용성과 내결함성을 제공하기 위해 사용됩니다. AWS의 다양한 서비스는 가용 영역에 대한 이해 없이 사용될 수 있습니다. 가용 영역을 고려하는 것은 대규모 재해나 데이터 센터의 중단 등과 같은 예상치 못한 사건에 대한 대비책을 마련하기 위함입니다.

가용 영역은 다양한 AWS 서비스에서 사용됩니다. 예를 들어, EC2(Elastic Compute Cloud) 인스턴스를 생성할 때 가용 영역을 선택하여 인스턴스를 배치할 수 있으며, RDS(Relational Database Service) 인스턴스를 생성할 때도 가용 영역을 선택하여 데이터베이스를 배치할 수 있습니다.

### 서브넷(Subnet)

위 그림에서는 VPC를 3개의 서브넷으로 분할하였습니다. 각각의 서브넷은 고유한 CIDR 블록을 가지며, 서브넷 내의 호스트들은 해당 CIDR 블록 내에서 IP 주소를 할당받습니다.

예를 들어, 서브넷 A1은 10.0.1.0/24의 CIDR 블록을 가지며, 이를 할당받은 호스트들은 10.0.1.1부터 10.0.1.254까지의 IP 주소를 사용할 수 있습니다.

각각의 서브넷은 라우팅 테이블을 가지고 있으며, 서브넷 간의 통신은 이 라우팅 테이블을 기반으로 이루어집니다. 서브넷 A1에서 호스트가 서브넷 B1의 호스트와 통신하고자 할 경우, VPC 내부의 라우터를 통해 이루어지며, 이 라우터는 라우팅 테이블을 참조하여 적절한 경로로 패킷을 전달합니다.

이와 같이 서브넷을 구성함으로써, VPC를 보다 효과적으로 관리하고 보안성을 높일 수 있습니다.

### 탄력적 IP

탄력적 IP(Elastic IP)는 Amazon Web Services(AWS)에서 제공하는 고정 IP 주소입니다. 일반적으로 AWS에서 인스턴스를 생성하면 해당 인스턴스는 임시 IP 주소를 할당받게 됩니다. 하지만 이러한 임시 IP 주소는 인스턴스를 중지하거나 재시작할 때마다 변경될 수 있습니다.

이러한 문제를 해결하기 위해 탄력적 IP를 사용할 수 있습니다. 탄력적 IP를 사용하면 인스턴스에 고정된 IP 주소를 할당할 수 있습니다. 따라서 인스턴스를 중지하거나 재시작할 때마다 IP 주소가 변경되지 않아도 됩니다.

또한, 탄력적 IP를 다른 인스턴스로 연결할 수 있습니다. 예를 들어, 기존 인스턴스가 장애가 발생하여 다른 인스턴스로 교체해야 할 경우, 기존 인스턴스에 할당된 탄력적 IP를 새로운 인스턴스로 연결함으로써 서비스 중단 없이 이전할 수 있습니다.

탄력적 IP는 또한 AWS 리소스와 연결할 수 있습니다. 예를 들어, 탄력적 IP를 Amazon S3 버킷이나 Elastic Load Balancer와 연결하여 이러한 리소스에 고정된 IP 주소를 할당할 수 있습니다. 이를 통해 편리한 DNS 이름 매핑이 가능해지며, 서비스의 안정성을 높일 수 있습니다.

### 인터넷 게이트웨이(IGW)

IGW는 인터넷 게이트웨이(Internet Gateway)의 약어입니다.
인터넷 게이트웨이는 Amazon VPC(Virtual Private Cloud)에서 인터넷과 통신하기 위한 게이트웨이로, VPC 내부의 인스턴스나 다른 리소스가 인터넷과 통신할 수 있도록 합니다.

이를 위해 VPC 내부에서 라우팅 테이블(Route Table)을 구성하여 인터넷 게이트웨이를 목적지로 설정해야 합니다. IGW는 VPC당 하나만 생성할 수 있으며, 사용한 데이터 전송량과 연결 시간에 대한 요금이 부과됩니다.

## 2.4장

### NAT(Network Address Translation)

NAT(Network Address Translation)는 사설 네트워크와 공인 네트워크 간의 통신을 위해 사용되는 기술로, 사설 IP 주소와 공인 IP 주소 간의 변환을 수행합니다.

NAT는 사설 IP 주소를 사용하는 네트워크에서 인터넷과 연결할 때 사용됩니다. 이때 NAT 게이트웨이를 사용하여 사설 IP 주소를 공인 IP 주소로 변환하고, 인터넷과 통신을 수행합니다. NAT를 사용하면 한정된 공인 IP 주소를 효율적으로 활용할 수 있으며, 내부 네트워크를 외부에서 보호할 수 있습니다.

NAT 게이트웨이는 두 가지 유형이 있습니다. 첫 번째는 NAT 인스턴스(NAT Instance)로, Amazon EC2 인스턴스를 사용하여 구성할 수 있습니다. 두 번째는 NAT 게이트웨이(NAT Gateway)로, 이는 완전히 관리되는 서비스로써 NAT 인스턴스와 달리 더 높은 처리량 및 가용성을 제공합니다.

NAT 게이트웨이는 Amazon VPC에서 사용됩니다. VPC 내에서 NAT 게이트웨이를 사용하여 VPC 내부의 인스턴스가 인터넷과 통신할 수 있도록 하며, VPC 내부의 인스턴스를 외부에서 보호합니다. 이때 NAT 게이트웨이를 사용하기 위해서는 VPC 내부에서 라우팅 테이블(Route Table)을 구성하여 NAT 게이트웨이를 목적지로 설정해야 합니다.

### 인바운드 / 아웃바운드 액세스

인바운드(Inbound)와 아웃바운드(Outbound) 액세스는 네트워크에서 데이터가 이동하는 방향에 따라 구분됩니다.

인바운드 액세스는 외부에서 내부로 데이터가 이동하는 것을 의미합니다. 즉, 외부에서 내부로 들어오는 데이터를 말합니다. 예를 들어, 인터넷에서 웹 서버로 요청이 들어오는 경우, 이는 인바운드 액세스입니다.

아웃바운드 액세스는 내부에서 외부로 데이터가 이동하는 것을 의미합니다. 즉, 내부에서 외부로 나가는 데이터를 말합니다. 예를 들어, 웹 서버에서 인터넷으로 응답이 나가는 경우, 이는 아웃바운드 액세스입니다.

인바운드와 아웃바운드 액세스는 보안 그룹(Security Group) 등의 네트워크 보안 구성에서 중요한 역할을 합니다. 예를 들어, 웹 서버에 대한 인바운드 액세스를 제한하여 웹 서버를 보호하거나, 웹 서버에서 아웃바운드 액세스를 제한하여 데이터 유출을 방지할 수 있습니다.

## 2.5장

### SSH

SSH는 Secure Shell의 약자로, 네트워크를 통해 다른 컴퓨터에 로그인하거나 원격으로 명령을 실행할 수 있는 프로토콜입니다.

SSH는 암호화 기술을 사용하여 데이터 전송과 인증을 보호합니다. 이를 통해 다른 컴퓨터와 안전하게 통신할 수 있습니다. SSH는 대개 리눅스나 유닉스 기반 서버에서 사용되며, 원격 서버 접속이나 파일 전송 등에 널리 사용됩니다.

일반적으로 SSH를 사용하면 원격 시스템에 로그인하여 명령을 실행하거나, 파일을 전송하거나, 원격 포트 포워딩 등 다양한 작업을 수행할 수 있습니다. SSH는 네트워크 보안에 매우 중요한 역할을 하며, 보안을 위해 많이 사용됩니다.

## 2.7장

### ALB(Application Load Balancer)

ALB는 Application Load Balancer의 약자로, AWS에서 제공하는 로드 밸런서 서비스 중 하나입니다.

ALB는 애플리케이션 수준에서 작동하며, HTTP/HTTPS 트래픽에 특화되어 있습니다. ALB는 여러 대의 EC2 인스턴스 또는 컨테이너 등의 백엔드 서버에 대한 트래픽을 분산하여 처리합니다.

ALB는 다양한 기능을 제공합니다. 예를 들어, HTTP/HTTPS 요청 라우팅, SSL 종료, 애플리케이션 레이어에서의 세션 관리 등이 있습니다. 또한, ALB는 콘텐츠 기반 라우팅, IP 주소 기반 라우팅 등 다양한 라우팅 방식을 지원합니다.

ALB는 또한 Elastic Container Service(ECS)와 같은 AWS 서비스와 통합되어 있습니다. ECS는 ALB와 함께 사용하면 컨테이너화된 애플리케이션을 쉽게 배포하고 관리할 수 있습니다. ALB는 Auto Scaling과 함께 사용하여 트래픽 부하에 따라 EC2 인스턴스의 수를 자동으로 조정할 수 있습니다.

ALB는 다양한 애플리케이션 환경에서 사용할 수 있으며, 안정적인 로드 밸런싱 기능을 제공합니다.

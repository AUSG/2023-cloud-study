## 2. Network

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

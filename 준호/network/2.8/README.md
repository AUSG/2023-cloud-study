## 2.8 접두사 목록을 활용한 보안 그룹의 CIDR 관리

### 문제 설명

퍼블릭 서브넷의 인스턴스에서 특정 액세스 요구 사항을 가진 2개의 애플리케이션을 호스팅하고 있다.
대부분의 작동 시간 중에는 다른 리전의 가상 데스크톱에서 액세스하지만, 테스트 기간에는 집 PC에서 접근해야 한다.

### 해결 방법

1. aws가 제공하는 IP 주소 범위 목록을 사용해 해당 리전의 WorkSpace 게이트웨이의 CIDR 범위 목록을 포함하는 관리형 접두사 목록(managed prefix list)을 각각 만들어 보안 그룹에 연결한다.

2. 테스트를 위해서는 임시로 집의 IP 주소를 접두사 목록에 추가했다가 테스트 종료 시 IP 주소를 삭제한다.

### 작업 방법

1. AWS 주소 범위를 가진 파일 다운로드

```bash
curl -o ip-ranges.json https://ip-ranges.amazonaws.com/ip-ranges.json
```

2. us-west-2 리전에서 Amazon WorkSpace 게이트웨이에 대한 CIDR 범위 목록을 생성한다.

```bash
jq -r '.prefixes[] | select(.region=="us-east-1") | select(.service=="WORKSPACES_GATEWAYS") | .ip_prefix' < ip-ranges.json

```

3. Amazon WorkSpaces에 대한 IP 범위로 관리형 접두사 목록을 생성한다.

```bash
PREFIX_LIST_ID=$(aws ec2 create-managed-prefix-list \
--address-family IPv4 \
--max-entries 15 \
--prefix-list-name allowed-us-west-2-cidrs \
--output text --query "PrefixList.PrefixListId" \
--entries Cidr=44.234.54.0/23,Description=workspaces-us-east-1-cidr1 Cidr=54.244.46.0/23,Description=workspaces-us-east-1-cidr2)
```

4. 사용자의 퍼블릭 IPv4 주소를 확인한다.

```bash
MY_ID_4=$(curl myip4.com | tr -d ' ')
```

5. IPv4 주소 관리형 접두사 목록에 추가

```bash
aws ec2 modify-managed-prefix-list --prefix-list-id $PREFIX_LIST_ID --current-version 1 --add-entries Cidr=${MY_ID_4}/32,Description=my-workstation-ip
```

6. 각 앱의 보안 그룹의 접두사 목록에 TCP 포트 80의 액세스를 허용하는 인바운드 규칙 추가

```bash
aws ec2 authorize-security-group-ingress \
--group-id sg-00 --ip-permissions IpProtocol=tcp,FromPort=80,ToPort=80,PrefixListIds="[{Description=http-fromprefix-list,PrefixListId=$PREFIX_LIST_ID}]"

aws ec2 authorize-security-group-ingress \
--group-id sg-01 --ip-permissions IpProtocol=tcp,FromPort=80,ToPort=80,PrefixListIds="[{Description=http-fromprefix-list,PrefixListId=$PREFIX_LIST_ID}]"
```

### 유효성 검사

- 내 PC에서 해당 인스턴스 접근 되는지 테스트

```bash
curl -m 2 $INSTANCE_IP_1

curl -m 2 $INSTANCE_IP_2
```

### 참고

인스턴스에 대한 인바운드 통신을 허용하는 cidr 블록 목록을 업데이트해야 하는 경우
보안 그룹 대신 접두사 목록을 업데이트하는 것이 다수의 보안 그룹을 관리할 때 오버헤드를 줄일 수 있음.

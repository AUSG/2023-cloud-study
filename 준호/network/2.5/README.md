## 2.5 보안 그룹을 참고해 동적으로 접근 권한 부여

### 문제 설명

2개의 인스턴로 구성한 애플리케이션 그룹 사이에 SSH를 허용해야 한다.
향후 인스턴스 수가 증가해도 이를 손쉽게 적용할 수 있어야 한다.

### 해결 방법

1. 보안 그룹을 생성하고, 두 인스턴스의 ENI에 연결한다.
2. 보안 그룹이 TCP 포트 22에 도달하도록 승인하는 수신 규칙을 생성한다.

### 작업 방법

1. EC2 인스턴스에 대한 새 보안 그룹을 생성한다.

```bash
SG_ID=$(aws ec2 create-security-group \
--group-name AWSCookbook205sg \
--description "Instance Security Group" --vpc-id $VPC_ID \
--output text --query GroupId)
```

2. 보안 그룹을 인스턴스 1과 2에 연결한다.

```bash
aws ec2 modify-instance-attribute --instance-id $INSTANCE_ID_1 \
--groups $SG_ID

aws ec2 modify-instance-attribute --instance-id $INSTANCE_ID_2 \
--groups $SG_ID
```

3. TCP 포트 22에 대한 액세스를 허용하는 인바운드 규칙을 보안 그룹에 추가한다.

```bash
aws ec2 authorize-security-group-ingress \
--protocol tcp --port 22 \
--source-group $SG_ID \
--group-id $SG_ID \
```

### 참고

클라운드의 온디멘드 특성은 탄력성을 제공한다.
이에 따라 네트워크 보안도 보안 그룹 참고와 같은 방법을 사용해 탄력성에 대응해야 한다.
전토적으로 정적 참고 방식으로 CIDR 범위를 조정해 방화벽을 구성했지만, 이런 방식은 인스턴스를 추가하거나 제거할 때 동적으로 변경할 수 없기 때문에 확장에 적합하지 않다.

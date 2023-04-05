# Container

# 6.1 Amazon ECR에 컨테이너 이미지 빌드, 태그 푸시
``` bash
Amazon ECR에서는 dockerhub처럼 private 레포지토리와 public 레포지토리를 제공해줌. 콘솔 창으로 들어가면 쉽게 생성할 수 있고 친절하게 해당 레포지토리에 이미지를 푸쉬하는 방법도 알려줌. 여기서 이미지를 올릴때 인증 토큰을 가져와서 docker login으로 넘겨줘야하고 이미지 같은 경우 올리기 위해선 지정된 태그를 지켜야함.
```

# 6.2 Amazon ECR에 푸시하는 컨테이너 이미지의 보안 취약점 스캔
``` bash
레지스트리에서 레포지토리를 만들 때 scanOnPush 를 이용해서 해당 레포지토리에 푸시하는 컨테이너 이미지의 스캔을 활성화해서 취약점을 스캔할 수 있는데 현재는 콘솔창으로 레포지토리를 만들때 '리포지토리 수준의 ScanOnPush 구성은 더 이상 사용되지 않으며 레지스트리 수준 스캔 필터로 대체' 됐다는 문구가 떴음. 즉, repository 수준은 취약점 스캔은 더 이상 사용되지 않고 private이나 public 레지스트리 수준으로 스캔 필터가 대체됐다는 것을 의미함. 이런 취약점에 대한 알림을 Amazon의 EventBidge나 Amazon SNS 연동을 통해 알림을 구성할 수 있음.
```

# 6.3 Amazon Lightsail을 사용한 컨테이너 배포
``` bash
Amazon Lightsail에서는 컨테이너 뿐만 아니라, 인증서, 로드 밸런서, 컴퓨팅, 스토리지까지 제공을 해줌. 그리고 컨테이너의 경우 default로 된 nginx, redis 같은 배포 설정 템플릿이 있고 커스텀하게도 할 수 있음. 커스텀하게 할 경우 포트나 이런걸 지정해줘야하고 이미지의 경우 퍼블릭 이미지를 사용해야하고 컨테이너 서비스에 컨테이너 이미지를 푸쉬할 수 도 있음. 그리고 컨테이너의 scale을 얼마를 잡을지도 정할 수 있음. scale을 3으로 지정하면 lightsail이 만약 애플리케이션의 상태를 확인하고 응답이 없을 경우 자동으로 새로 배포해줌.
```

# 6.4 AWS Copilot을 사용한 컨테이너 배포
``` bash
aws copilot은 일반적인 애플리케이션 아키텍처 및 인프라 패턴, 사용자 친화적인 운영 워크플로 및 지속적 전달 파이프라인 설정을 제공하여 고객이 AWS의 컨테이너식 애플리케이션을 더 쉽게 구축, 배포 및 운영할 수 있도록 해줌, 모범 사례에 따르면 Amazon ECS에서 컨테이너를 호스팅하는데 필요한 모든 리소스를 구성. 여러 가용 영역에 배포를 한다던지, 서브넷 계층을 사용해서 트래픽을 분할한다던지 KMS를 이용해서 암호화를 한다던지.
copilot을 사용하기 앞서 해당 cli를 mac기준으로 brew를 통해 다운받을 수 있음.
brew install aws/tap/copilot-cli

copilot init을 치면 해당 디렉토리에 copilot이라는 디렉토리를 생성해서 인프라 구성에 대한 파일을(manifest.yaml) 생성.

copilot의 명령어를 이용하면 ci/cd 파이프라인을 이용한 자동화된 배포도 구성할 수 있음.

copilot을 사용하기 위해서는 ECS 작업을 수행할 경우 관련 역할이 필요하고 아래 명령을 치면 nginx dockerfile을 이용해서 amazon ECS에 배포해줌.
copilot init --app web --name nginx --type 'Load Balanced Web Service' --dockerfile './Dockerfile' --port 80 --deploy

위 명령어를 치면 cloudFormation에 관련 stack이 생성되게 됨.
```

# 6.5 블루/그린 배포로 컨테이너 업데이트
``` bash
codeDeploy는 ec2인스턴스나 lambda함수, ECS 서비스로 애플리케이션의 배포를 자동화하는 배포 서비스로서 s3버킷이나 깃헙에 있는 애플리케이션 컨텐츠를 배포할 수 있음. CodeDeploy는 lambda 또는 ECS에서 카나리, AllAtOnce, 블루/그린 등의 여러 가지 배포 전략을 지원. 블루/그린 전략은 모든 트래픽이 새 버전으로 라우팅되는 동안 이전 버전을 5분간 실행 상태로 유지하는데 그 이유는 새 버전이 잘 작동하지 않을 수도 있기 때문에 그럼.
CodeDeploy는 ALB 대상 그룹을 사용해서 애플리케이션을 관리함.
```

# 6.6 Amazon ECS의 컨테이너 워크로드 자동 확장
``` bash
auto scaling 기능은 ecs 서비스에서 원하는 작업 수를 늘리거나 줄이는 기능. 이때 auto scaling을 위해 application auto scaling 서비스를 활용하여 기능 제공. Amazon ECS는 서비스의 평균 cpu 및 메모리 사용량과 함께 cloudWatch 지표를 게시. 그래서 이 지표릴 이용하여 피크 시간에는 서비스를 확장하거나 사용률이 낮은 기간에는 서비스를 축소하여 비용절감.

auto scaling 조정 정책은 대상 추적 조정 정책, 단계 조정 정책, 예약된 조정이 있는데 대상 추적의 경우 특정 지표에 대한 대상값을 기준으로 작업의 수를 조절하고 단계같은 경우 말 그대로 경보 위반의 단계를 조절해서 작업의 수를 조정하는 정책이고 예약된 조정은 날짜 및 시간을 기준으로 작업의 수를 조절하는 정책.

이를 하기 위해서는 IAM 권한이 필요.
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "application-autoscaling:*",
        "ecs:DescribeServices",
        "ecs:UpdateService",
        "cloudwatch:DescribeAlarms",
        "cloudwatch:PutMetricAlarm",
        "cloudwatch:DeleteAlarms",
        "cloudwatch:DescribeAlarmHistory",
        "cloudwatch:DescribeAlarmsForMetric",
        "cloudwatch:GetMetricStatistics",
        "cloudwatch:ListMetrics",
        "cloudwatch:DisableAlarmActions",
        "cloudwatch:EnableAlarmActions",
        "iam:CreateServiceLinkedRole",
        "sns:CreateTopic",
        "sns:Subscribe",
        "sns:Get*",
        "sns:List*"
      ],
      "Resource": [
        "*"
      ]
    }
  ]
}

콘솔에서는 서비스를 만들 때 서비스 자동 크기 조정이 선택사항으로 존재하고 대상 추적으로 태스크 수나 지표값을 정할 수 있음.
```

# 6.7 이벤트를 통해 Fargate 컨테이너 작업 시작
``` bash
이번꺼는 블로그에 올려서 블로그 대체!
https://suminn0.tistory.com/150
```

# 6.8 
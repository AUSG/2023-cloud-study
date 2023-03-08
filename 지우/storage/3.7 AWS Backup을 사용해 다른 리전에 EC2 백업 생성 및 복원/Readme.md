# 3.7 AWS Backup을 사용해 다른 리전에 EC2 백업 생성 및 복원

> _AWS Backup으로 EC2 인스턴스의 온디맨드 백업을 생성하고 다른 리전에서 복원한다_

- 백업 볼트(backup vault)로부터 인스턴스 복원

<br>

**🥕 참고**

⍢ **AWS Backup**

- AWS 서비스 전체의 백업을 한 곳에서 관리하고 모니터링
- EC2 인스턴스 백업 시 선택한 백업 볼트에 백업을 저장

⍢ EBS 스냅샷

- 실행 인스턴스에 보호해야 할 영구 데이터가 있는 경우 필수!
- 수동, 자체적인 자동화 스크립트 또는 Data LifeCycle Manager로 자동화하거나 AWS Backup으로 생성

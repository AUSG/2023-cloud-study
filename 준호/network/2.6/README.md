## 2.6 VPC Reachability Analyzer를 활용한 네트워크 경로 확인 및 문제 해결

### 문제 설명

격리된 서브넷에 SSH 연결이 안되는 2개의 EC2 인스턴스가 배포돼 있다. SSH 연결 문제를 해결해야 한다.

### 해결 방법

1. VPC Reachability Analyzer를 사용해 네트워크 인사이트 경로를 생성하고 분석한다.
2. 결과를 확인하고 인스턴스 1의 보안 그룹에서 SSH 포트를 허용하는 규칙을 인스턴스 2의 보안 그룹에 추가한다.
3. VPC Reachability Analyzer를다시 실행하고 결과를 확인한다.

### 작업 방법

1. EC2 인스턴스와 TCP 포트 22를 지정하는 네트워크 인사이트 경로를 생성한다.

```bash
INSIGHTS_PATH_ID=$(aws ec2 create-network-insights-path \
--source $INSTANCE_ID_1 --destination-port 22 \
--destination $INSTANCE_ID_2 --protocol tcp \
--output text --query NetworkInsightsPath.NetworkInsightsPathId)
```

2. 이전 단계에서 생성한 INSIGHTS_PATH_ID를 사용해 두 인스턴스 간의 네트워크 인사이트 분석을 시작한다.

```bash
ANALYSIS_ID_1=$(aws ec2 start-network-insights-analysis \
--network-insights-path-id $INSIGHTS_PATH_ID --output text \
--query NetworkInsightsAnalysis.NetworkInsightsAnalysisId)
```

3. 분석 실행이 완료될 때까지 기다린 다음 결과를 확인한다.

```bash
aws ec2 describe-network-insights-analyses \
--network-insights-analysis-ids $ANALYSIS_ID_1
```

4. 인스턴스 2에 연결된 보안 그룹을 업데이트한다. 인스턴스 1의 보안 그룹에서 TCP 포트(SSH)로의 액세스를 허용하는 규칙을 추가한다.

```bash
aws ec2 authorize-security-group-ingress \
--protocol tcp --port 22 \
--source-group $INSTANCE_SG_ID_1 \
--group-id $INSTANCE_SG_ID_2
```

5. 네트워크 인사이트 분석을 다시 시작한다.

```bash
ANALYSIS_ID_2=$(aws ec2 start-network-insights-analysis \
--network-insights-path-id $INSIGHTS_PATH_ID --output text \
--query NetworkInsightsAnalysis.NetworkInsightsAnalysisId)
```

6. 새 분석 결과를 확인한다.

```bash
aws ec2 describe-network-insights-analysis \
--network-insights-analysis-ids $ANALYSIS_ID_2
```

### 참고

네트워크 연결을 네트워크 인사이트 경로로 정의해 테스트할 수 있다.
처음에는 대상의 보안 그룹이 액세스를 허용하지 않았기 때문에 인스턴스 간에 SSH 연결을 할 수 없었다.
인스턴스 2의 보안 그룹을 업데이트하고 분석을 다시 실행하면 SSH 연결을 확인할 수 있었다.

VPC Reachability Analyzer는 네트워크 경로 분석 결과를 설명 코드를 반환한다.
이 레시피에서는 보안 그룹이 소스와 대상 간의 트래픽을 허용하지 않음을 나타내는 ENI_SG_RULES_MISMATCH 코드를 확인했다.

# 6. Amazon ECS의 컨테이너 워크로드 자동 확장

### 실습

---

- 자동적으로 확장이 가능한 컨테이너 서비스를 배포하자
- CPU 부하가 증가할 때 서비스에서 더 많은 컨테이너를 추가하도록 ECS 서비스에 대한 CloudWatch 경보 및 조정 정책을 구성

![image](https://user-images.githubusercontent.com/49095587/230257727-e4d84fcf-bf52-4d5c-9502-aa99aa815bd1.png)

1. verbose 및 3초 제한 시간 플래그를 사용해 전체 연결을 확인하고 제한 시간이 설정됐는지 확인

   ```bash
   curl -v -m 3 https://AWSCookbook.us-east-1.elb.amazonaws.com:80/
   ```

   → 이 명령어는 3초 이내에 응답이 오지 않으면 curl이 자동으로 요청을 취소. **`-v`**옵션을 사용하면 verbose 모드로 실행되어 응답에 대한 자세한 정보가 출력.

   - `verbose` : 프로그램이 실행될 때 출력되는 정보의 상세도를 설정하는 옵션

2. 자동 크기 조정 트리거가 사용할 수 있는 역할을 생성

   ```bash
   {
     "Version": "2012-10-17",
     "Statement": [
       {
         "Sid": "",
         "Effect": "Allow",
         "Principal": {
           "Service": "ecs-tasks.amazonaws.com"
         },
         "Action": "sts:AssumeRole"
       }
     ]
   }
   ```

3. 자동 확장을 위한 관리형 정책 연결 : `AmazonEC2ContainerServiceAutoScaleRole`
4. 자동 크기 조정 대상 등록 : `aws application-autoscaling register-scalable-target`
5. CPU 사용량이 평균 50% 이상이 되면 자동 크기 조정 정책을 실행하도록 설정
6. curl 명령을 실행해 높은 CPU 로드를 시뮬

   ```bash
   curl -v -m 3 $LOAD_BALANCER_DNS/cpu
   ```

   → 해당 URL을 호출하면 컨테이너가 CPU에 부하를 주는 프로세스를 실행하기 때문에 3초 후에 시간 초과 오류를 반환

   <Br>

### Auto Scaling

---

- 애플리케이션은 로드가 증가함에 따라 자체 리소스를 프로비저닝할 수 있으며 애플리케이션이 유휴 상태일 때 리소스를 해지함으로써 비용을 절감할 수 있다.
- 위의 레시피에서는 ECS 서비스의 `CPU 사용량` 지표를 모니터링했는데, 이 밖의 `네트워크 I/O`, `메모리 사용량`, `트랜잭션 수`에 따라 자동 크기 조정을 구성할 수 있다.

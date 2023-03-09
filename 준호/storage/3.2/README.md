## 3.2 S3 Intelligent-Tiering 아카이브 정책을 사용한 S3 객체 자동 아카이브

### 🎯 목표

자주 액세스하지 않는 객체를 성능 저하 없이 운영 오버헤드를 추가하지 않고 다른 아카이브 스토리지 클래스로 자동 전환하고자 함

### 🤔 해결 방법

- 90일 이상 된 객체에 대한 액세스 패턴을 기반으로 객체를 S3 Glacier 아카이브로 자동화하는 정책을 생성한 뒤 버킷에 적용한다.

### 👉 작업 방법

1. tiering.json 생성
2. Intelligent-Tiering 구성을 적용한다.

```bash
aws s3api put-bucket-intelligent-tiering-configuration \
    --bucket awscookbook302-$RANDOM_STRING \
    --id awscookbook302 \
    --intelligent-tiering-configuration "$(cat tiering.json)"
```

3. 유효성 검사

```bash
aws s3api get-bucket-intelligent-tiering-configuration \
    --bucket awscookbook302-$RANDOM_STRING \
    --id awscookbook302 \
```

### 🤗 참고

- S3 Intelligent-Tiering 아카이브는 자주 액세스하지 않는 객체를 S3 Glacier 아카이브로 자동 전환하는 자동 매커니즘을 제공하며 전환 시간을 정의할 수 있다.(90-730일 사이로 지정 가능)
- S3 계층(클래스)은 객체별로 적용되고, Intelligent-Tiering 아카이브는 버킷별로 적용된다.

1. Frequent Access
2. Infrequent Access
3. Archive Access
4. Deep Archive Access

**Intelligent-Tiering 아카이브 vs Intelligent-Tiering 클래스**

- Amazon S3에서는 Intelligent-Tiering 클래스와 S3 Glacier 및 S3 Glacier Deep Archive 클래스를 제공합니다. 이 중 Intelligent-Tiering 클래스는 데이터 액세스 패턴에 따라 자동으로 객체를 다른 클래스로 이동시켜 최적화된 비용으로 데이터를 저장하는 기능을 제공합니다.

- S3 Intelligent-Tiering 클래스는 아카이브와 다른점이 있습니다. Intelligent-Tiering 클래스는 데이터 액세스 패턴에 따라 자동으로 객체를 S3 Standard, S3 Standard-IA 및 S3 One Zone-IA 클래스 중 하나로 이동시키거나, S3 Glacier 또는 S3 Glacier Deep Archive 클래스로 이동시킵니다. 이 구성을 사용하면 객체 액세스 및 복원에 대한 요금을 지불할 필요없이 자동으로 데이터 저장소를 최적화할 수 있습니다.

- 반면 S3 Glacier와 S3 Glacier Deep Archive 클래스는 데이터를 오랫동안 저장하는 것을 목적으로 하는 아카이브 클래스입니다. S3 Glacier 클래스의 객체는 일반적으로 몇 시간에서 12시간 안에 복원될 수 있지만, S3 Glacier Deep Archive 클래스의 객체는 최대 12시간까지 복원 시간이 걸릴 수 있습니다. 따라서, S3 Glacier와 S3 Glacier Deep Archive 클래스는 오랫동안 데이터를 저장하고, 자주 액세스하지 않는 경우에 사용합니다.

- 따라서, S3 Intelligent-Tiering 클래스는 데이터 액세스 패턴에 따라 최적화된 클래스로 객체를 자동으로 이동시키는 것이 목적이며, S3 Glacier 및 S3 Glacier Deep Archive 클래스는 데이터를 오랫동안 저장하는 것이 목적입니다.

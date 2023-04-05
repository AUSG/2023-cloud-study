# 7.4 Amazon Athena를 사용한 S3 내의 파일 쿼리

> _S3에 저장된 csv 파일을 인덱싱하지 않고 SQL 쿼리를 실행하려면 Athena_

- 사전 준비 : S3 버킷 (`athena-ziwoo`)

<br>

① Amazon Athena console → **[ 쿼리 편집기 탐색 ]** → **설정** → **관리** → 버킷 선택

- 쿼리 결과까지 암호화 가능

<img src="https://user-images.githubusercontent.com/70079416/227791122-c93186c4-107c-48b0-9d9a-9e09f7e2c4d5.png" />

② **편집기**로 돌아가서 Data Catalog 데이터베이스 생성 후 필요에 맞게 쿼리 실행

```sql
CREATE DATABASE `awscookbook704db`
```

---

🥕 **참고**

⍢ **Athena** : 표준 SQL을 사용하여 데이터 레이크 내의 데이터를 직접 분석할 수 있는 대화형 쿼리 서비스

(AWS Lake Formation → S3, Glue, Athena 등)

- 쿼리를 실행하기 전에 데이터 메타데이터, 스키마, 데이터 위치를 설정해야 한다.
- 테이블, 데이터베이스, Data Catalog 개념의 이해가 필요하다.
- 고유 스키마를 정의하는 대신 Athena의 Glue Data Catalog로 최신 정보를 유지하고 시간을 절약할 수 있다.

⍢ 대규모 데이터셋에 대해 자동으로 확장하여 병렬로 쿼리 실행 → 리소스 프로비저닝 불필요!!

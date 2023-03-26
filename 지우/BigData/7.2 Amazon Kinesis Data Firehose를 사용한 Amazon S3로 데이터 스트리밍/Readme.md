7.2 Amazon Kinesis Data Firehose를 사용한 Amazon S3로 데이터 스트리밍

> _Kinesis Stream 데이터를 S3 버킷으로 전달하도록 Kinesis Data Firehose를 구성한다_

- 사전 준비 : Kinesis Data Stream(`kinesis-ziwoo`), S3 버킷에 업로드된 csv file(`kinesis-bucket-ziwoo`)

① Kinesis Data Firehose console → **[ 전송 스트림 생성 ]** 클릭 → 소스 및 대상 선택

<img src="https://user-images.githubusercontent.com/70079416/227789687-5361bafb-96fc-4c3d-97a7-c36b401971d4.png" />

② 소스 설정 → Kinesis Data Stream 선택

<img src="https://user-images.githubusercontent.com/70079416/227789689-aa3542ac-9f96-411c-9eee-018178e21361.png" />

③ 대상 설정 → S3 버킷 선택

<img src="https://user-images.githubusercontent.com/70079416/227789690-1f6435bb-7d43-400e-8e24-1b0af9babf4e.png" />

④ 고급 설정 → **Create or update IAM role** → Kinesis가 Stream 및 S3 버킷에 액세스하기 위한 역할 생성

---

🥕 **유효성 검사** : 스트림으로의 전송을 테스트

① **Test with demo data** (데모 데이터로 테스트) → **Start sending demo data** (데모 데이터 전송 시작)

<img src="https://user-images.githubusercontent.com/70079416/227789692-9f4f2314-f890-4058-b956-c7a1a0e4f8fc.png" />

② S3 버킷에 전송된 객체 및 내부 데이터 확인

<img src="https://user-images.githubusercontent.com/70079416/227789694-87a069c9-7d9d-4a4b-a8f6-fe6651bd4b48.png" />

🥕 **참고**

⍢ Kinesis Data Firehose를 사용하면 -

- S3, Amazon Redshift, OpenSearch 및 다양한 엔드포인트에 데이터를 전달할 수 있다.
- 여러 전달 스트림을 단일 생산자 스트림에 연결할 수도 있다.

⍢ 데이터 볼륨을 원활히 처리하고자 자동적으로 확장

⍢ 대상에 도달하기 전에 데이터를 변환해야 하는 경우 변환(transform)을 구성할 수 있다.

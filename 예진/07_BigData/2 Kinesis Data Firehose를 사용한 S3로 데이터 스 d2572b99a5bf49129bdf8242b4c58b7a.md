# 2. Kinesis Data Firehose를 사용한 S3로 데이터 스트리밍

<br>

<br>

### 실습

---

- 스트리밍 데이터를 객체 스토리지(S3)로 전달하자.
- S3 버킷을 생성 → Kinesis 스트림 생성 → 스트림 데이터를 S3 버킷으로 전달하도록 Kinesis Data Firehose 구성

1. Kinesis Data Firehose 콘솔 → Create delivery stream 버튼 클릭
2. Amazon kinesis Data Stream을 소스로 선택 , S3를 대상으로 선택
3. 소스 설정에서 (사전)702 Kinesis 스트림 선택
4. 대상 설정에서 (사전)S3 버킷을 찾아 선택 → 동적 분할과 S3 버킷 접두사는 기본값을 사용
5. 고급 설정에서 Create or update IAM role이 선택됐는지 확인

   1. Kinesis가 스트림 및 S3 버킷에 액세스하는 데 사용할 수 있는 IAM 역할을 생성

      ![image](https://user-images.githubusercontent.com/49095587/227703658-e9247056-21b1-4694-bde6-a0e007727f27.png)

6. Kinesis 콘솔 → Delivery streams를 클릭 → 생성한 스트림 선택
7. Test with demo data 섹션에서 Start sending demo data 버튼 클릭

   ![image](https://user-images.githubusercontent.com/49095587/227703665-70406bb4-bc9b-4088-8c74-7eb6b8791ed0.png)

8. S3 버킷에서 Kinesis 스트림의 데이터 확인
   <br>
   <br>

### 정리

---

다양한 소스에서 수집한 스트리밍 데이터를 경우에 따라 저장하고 나중에 확인하거나 처리해야 할 경우, **Kinesis Data Firehose를 사용**하여 S3, Redshift, OpenSearch 및 다양한 엔드포인트에 데이터를 전달할 수 있다.

# 7.3 AWS Glue 크롤러를 사용한 메타데이터 검색 자동화

> _S3의 csv 파일을 추가 분석 및 쿼리 작업에 사용하기 위해 AWS Glue로 S3 버킷 데이터를 스캔하도록 크롤러 구성 후 파일에 대한 스키마 및 메타데이터를 확인한다_

- 사전 준비 : S3 (`glue-bucket-ziwoo`)

<br>

① AWS Glue console → 탐색 메뉴 중 **[ Databases ]** → **[ Add database ]**

<img src="https://user-images.githubusercontent.com/70079416/227789995-a4a66cc3-dc7f-49e0-b7c2-cb9c605266fb.png" />

② 탐색 메뉴 중 **[ Tables ]** → **[ Add tables using crawler ]**

- Data Source : S3, Crawl all sub-folders

<img src="https://user-images.githubusercontent.com/70079416/227789998-6265f1be-70b9-4b89-9abe-615d62540466.png" />
<img src="https://user-images.githubusercontent.com/70079416/227790000-64d7afd1-3d2d-4cd1-98dd-b5168253c670.png" />

- IAM Role 생성

<img src="https://user-images.githubusercontent.com/70079416/227790515-373a4c90-bc91-4d0e-a5ef-46913349dc58.png" />

- ①번에서 생성한 데이터베이스 연결, Crawler schedule: On-demand

<img src="https://user-images.githubusercontent.com/70079416/227790517-475118e0-4ba5-4dc7-bf9b-550270ad194a.png" />

③ 탐색 메뉴 중 **[ Crawlers ]** → **[ Run ]** 클릭하여 크롤러 실행

**🥕 유효성 검사**

⍢ State가 `Running`에서 `Ready`로 바뀌고 Last run이 `Succeeded`임을 확인할 수 있다.

<img src="https://user-images.githubusercontent.com/70079416/227790519-ee6acfb9-37a6-4ae9-8ff8-0e936c686cf9.png" />

_state: running_

<img src="https://user-images.githubusercontent.com/70079416/227790521-1ccde38f-b526-4cae-93c3-cc9707a7b917.png" />

_state: ready_

⍢ CLI에서 다음 명령을 통해 `LastCrawl`의 `Status`가 `SUCCEEDED`임을 확인할 수 있다.

```bash
aws glue get-crawler --name crawler-ziwoo
```

<img src="https://user-images.githubusercontent.com/70079416/227790522-a46d5a44-c996-439c-8e85-690dc14ff31d.png" />

LastCrawl의 Status : SUCCEEDED

⍢ [ **View Log ]\*\*를 선택하면 Amazon CloudWatch에 기록된 로그를 확인할 수 있다.

- 디버깅해야 하는 경우 `/aws-glue/crawlers` 로그 그룹에서 확인 가능
- CLI 통해서도 확인 가능
  ```bash
  aws glue get-table --database-name db-ziwoo --name data
  ```

<img src="https://user-images.githubusercontent.com/70079416/227790523-826b1480-26ea-4b0f-9f3c-79c6b74bc0eb.png" />
<img src="https://user-images.githubusercontent.com/70079416/227790524-aebeb243-ff18-4b76-ab4a-1b8be9cba259.png" />

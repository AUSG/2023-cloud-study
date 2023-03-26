# 7.3 AWS Glue 크롤러를 사용한 메타데이터 검색 자동화

> _S3의 csv 파일을 추가 분석 및 쿼리 작업에 사용하기 위해 AWS Glue로 S3 버킷 데이터를 스캔하도록 크롤러 구성 후 파일에 대한 스키마 및 메타데이터를 확인한다_

- 사전 준비 : S3 (`glue-bucket-ziwoo`)

<br>

① AWS Glue console → 탐색 메뉴 중 **[ Databases ]** → **[ Add database ]**

![Untitled](7-1%20Big%20Data%20d7073282b0c34cb39c29b8da85e69387/Untitled%206.png)

② 탐색 메뉴 중 **[ Tables ]** → **[ Add tables using crawler ]**

- Data Source : S3, Crawl all sub-folders

![Untitled](7-1%20Big%20Data%20d7073282b0c34cb39c29b8da85e69387/Untitled%207.png)

![Untitled](7-1%20Big%20Data%20d7073282b0c34cb39c29b8da85e69387/Untitled%208.png)

- IAM Role 생성

![Untitled](7-1%20Big%20Data%20d7073282b0c34cb39c29b8da85e69387/Untitled%209.png)

- ①번에서 생성한 데이터베이스 연결, Crawler schedule: On-demand

![Untitled](7-1%20Big%20Data%20d7073282b0c34cb39c29b8da85e69387/Untitled%2010.png)

③ 탐색 메뉴 중 **[ Crawlers ]** → **[ Run ]** 클릭하여 크롤러 실행

**🥕 유효성 검사**

⍢ State가 `Running`에서 `Ready`로 바뀌고 Last run이 `Succeeded`임을 확인할 수 있다.

![state: running](7-1%20Big%20Data%20d7073282b0c34cb39c29b8da85e69387/Untitled%2011.png)

state: running

![state: ready](7-1%20Big%20Data%20d7073282b0c34cb39c29b8da85e69387/Untitled%2012.png)

state: ready

⍢ CLI에서 다음 명령을 통해 `LastCrawl`의 `Status`가 `SUCCEEDED`임을 확인할 수 있다.

```bash
aws glue get-crawler --name crawler-ziwoo
```

![LastCrawl의 Status : SUCCEEDED](7-1%20Big%20Data%20d7073282b0c34cb39c29b8da85e69387/Untitled%2013.png)

LastCrawl의 Status : SUCCEEDED

⍢ [ **View Log ]\*\*를 선택하면 Amazon CloudWatch에 기록된 로그를 확인할 수 있다.

- 디버깅해야 하는 경우 `/aws-glue/crawlers` 로그 그룹에서 확인 가능
- CLI 통해서도 확인 가능
  ```bash
  aws glue get-table --database-name db-ziwoo --name data
  ```

![Untitled](7-1%20Big%20Data%20d7073282b0c34cb39c29b8da85e69387/Untitled%2014.png)

![Untitled](7-1%20Big%20Data%20d7073282b0c34cb39c29b8da85e69387/Untitled%2015.png)

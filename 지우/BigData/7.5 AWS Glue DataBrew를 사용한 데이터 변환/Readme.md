# 7.5 AWS Glue DataBrew를 사용한 데이터 변환

> _CSV로 저장된 데이터에 추가적인 처리를 하기 전에 특정 열의 모든 문자를 대문자로 변환하기 위해 Glue DataBrew를 사용한다_

① AWS Glue DataBrew console → **[ 샘플 프로젝트 생성 ]** → **[ 2020년의 인기 아기 이름 ]** & **새 IAM 역할 생성** 선택

- 역할 접미사는 `ziwoo`로 설정한 후 **[ 프로젝트 생성 ]** 클릭

<img src="https://user-images.githubusercontent.com/70079416/227791202-b7ff38d4-a745-4f38-a4a1-bf1c664492a9.png" />

<br>

② 세션 준비되면 메뉴에서 **형식** 클릭하고, 드롭다운 메뉴에서 **대문자로 변경** 선택

<img src="https://user-images.githubusercontent.com/70079416/227791204-b917e035-1d33-4ab1-bd36-1474acf516b3.png" />

<br>

③ 변경된 것 확인 후 **작업** 메뉴에서 **CSV 다운로드** 클릭

<img src="https://user-images.githubusercontent.com/70079416/227791205-586aaefc-9d6a-49c4-9b00-2d76e4d048a9.png" />

---

🥕 **참고**

⍢ DataBrew로 데이터 형식 지정, 정리, 정보 추출 작업을 간소화

- 레시피 작업을 시각적으로 디자인하여 데이터를 처리하고 결과를 미리 확인하며, 데이터 처리 워크플로우를 자동화할 수 있다.

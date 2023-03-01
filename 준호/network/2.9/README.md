## 2.9 VPC 엔드포인트를 사용한 S3 접근

### 문제 설명

VPC 내의 리소스의 대역폭 비용을 낮게 유지하면서 보안을 위해 외부 인터넷을 사용하지 않고 특정 S3 버킷에 접근해야 한다.

### 해결 방법

- S3용 게이트웨이 VPC 엔드포인트를 생성하고, 라우팅 테이블과 연결한 뒤 정책 문서를 업데이트 한다.

### 작업 방법

1. VPC에 게이트웨이 엔드포인트를 생성하고 엔드포인트를 라우팅 테이블과 연결한다.

```bash
END_POINT_ID=$(aws ec2 create-vpc-endpoint --vpc-id $VPC_ID --service-name com.amazonaws.$AWS_REGION.s3 --route-table-ids $RT_ID_1 $RT_ID_2 --query VpcEndpoint.VpcEndpointId --output text)
```

2. 저장소의 코드를 참고해 policy-teplate.json 이라는 엔드포인트 정책 파일을 생성한다.

3. policy.json 파일 생성

```bash
sed -e "s/S3BucketName/${BUCKET_NAME}/g" policy-template.json > policy.json
```

4. 엔드 포인트의 정책 문서 수정한다. 해당 엔드포인트 정책으로 VPC 엔드포인트를 통해 액세스할 수 있는 리소스를 제한할 수 있음

```bash
aws ec2 modify-vpc-endpoint --policy-document file://policy.json --vpc-endpoint-id $END_POINT_ID
```

### 참고

S3 버킷에 대한 액세스를 제한하기 위해 엔드포인트 정책을 사용할 수 있다.
계정이 소유한 버킷뿐만 아니라 AWS의 모든 S3 버킷에도 적용할 수 있다.

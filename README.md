# aws-study-tutorial (Pipeline)

## 파이프라인
![image](https://user-images.githubusercontent.com/79297534/110262905-6d760480-7ff8-11eb-8a44-604301873a89.png)

1. 소스 컨트롤에서 소스 코드를 가져옵니다.
2. 구성 파일을 lint하세요.
3. 코드베이스에서 AWS Lambda 함수에 대한 단위 테스트를 실행하세요.
4. 테스트 파이프라인을 배포하세요.
5. 테스트 파이프라인에 대해 엔드 투 엔드 테스트를 실행하세요.
6. 테스트 상태 머신 및 테스트 인프라를 정리하세요.
7. 승인자에게 승인을 보냅니다.
8. 프로덕션에 배포하세요.


## AWS Pipeline 구축 실습

### Amazon S3 버킷 만들기
- https://console.aws.amazon.com/s3/ 에 접속하여 Create bucket 선택
![image](https://user-images.githubusercontent.com/79297534/110424399-16e6f400-80e6-11eb-976d-0fef9b3a2b13.png)

- bucket명은 awscodepipeline-demobucket-example-0309로 지정
- AWS Region은 본인에 맞는 리전을 선택
![image](https://user-images.githubusercontent.com/79297534/110425395-a0e38c80-80e7-11eb-856b-b082fbb5aff2.png)
- Bucket Versioning을 Enable로 선택해주고 Create bucket 버튼 선택 
![image](https://user-images.githubusercontent.com/79297534/110425910-7d6d1180-80e8-11eb-86b0-d23707793962.png)
- Bucket 생성확인 및 생성한 버킷 선택
![image](https://user-images.githubusercontent.com/79297534/110426077-ca50e800-80e8-11eb-84b3-038175d0ab64.png)
- 아래의 주소에서 SampleApp_Windows.zip 파일 받기 (리눅스는 다른거 받아야함)
https://docs.aws.amazon.com/ko_kr/codepipeline/latest/userguide/samples/SampleApp_Windows.zip
- 다운완료한 파일을 아래 그림과 같은 순서로 추가후 Upload
![image](https://user-images.githubusercontent.com/79297534/110426591-beb1f100-80e9-11eb-97bc-8c6d254167d0.png)
![image](https://user-images.githubusercontent.com/79297534/110426632-ca9db300-80e9-11eb-9fab-c8916b18cbec.png)

### Amazon EC2 Windows instances 생성 및 CodeDeploy agent 설치
- https://console.aws.amazon.com/iam/ 에 접속하여 Roles(역할) 선택후 Create role 선택
![image](https://user-images.githubusercontent.com/79297534/110427061-7c3ce400-80ea-11eb-8431-a15388756aa3.png)
- Select type of trusted entity에서 AWS service 선택/ Choose a use case에서 EC2 선택후 Next:permissions 선택
![image](https://user-images.githubusercontent.com/79297534/110427660-6e3b9300-80eb-11eb-8b85-a57508682538.png)
- AmazonEC2RoleforAWSCodeDeploy를 검색하여 해당 정책의 체크박스에 체크후 Next: Tags 선택 
![image](https://user-images.githubusercontent.com/79297534/110428307-66c8b980-80ec-11eb-82da-621e79770a5f.png)
- Add tags에서 Next:Review로 넘거간후 아래 그림처럼 작성 (Role name:EC2InstanceRole-0309)
![image](https://user-images.githubusercontent.com/79297534/110428803-3e8d8a80-80ed-11eb-97bd-ffb4002becdc.png)


### 인스턴스 실행하기 위한 설정
- https://console.aws.amazon.com/ec2/ 에 접속하여 Launch instance 선택
![image](https://user-images.githubusercontent.com/79297534/110429118-ca9fb200-80ed-11eb-9810-201c94efe760.png)
- Microsoft Windows Server 2019 검색하여 프리티어에 적합한 Microsoft Windows Server 2019 Base를 select
![image](https://user-images.githubusercontent.com/79297534/110429277-0175c800-80ee-11eb-9fc6-2f615ebfdf9e.png)
- t2.micro타입 체크후 Next: Configure Instance Details을 선택
![image](https://user-images.githubusercontent.com/79297534/110429659-9bd60b80-80ee-11eb-9263-1d5f502f99e6.png)
- 아래 그림 같이 변경후 스크롤을 내려 Advanced Details탭으로 이동
![image](https://user-images.githubusercontent.com/79297534/110430103-3898a900-80ef-11eb-945b-6ccb1a69e4b7.png)
- Advanced Details에 User data부분에 As text에 체크한후 아래의 글을 붙여넣기 (bucket-name에는 본인의 리전을 넣어야함)
- 실습에서는 bucket-name에 aws-codedeploy-ap-northeast-2를 기입함
![image](https://user-images.githubusercontent.com/79297534/110431441-2a4b8c80-80f1-11eb-9dbd-8bb41779b18d.png)
![image](https://user-images.githubusercontent.com/79297534/110430504-d55b4680-80ef-11eb-8f09-d2ee26bd3f92.png)
![image](https://user-images.githubusercontent.com/79297534/110430683-15bac480-80f0-11eb-8dec-b2a3ef33b726.png)


## 참고자료
- https://docs.aws.amazon.com/codepipeline/latest/userguide/tutorials-simple-s3.html#s3-create-s3-bucket

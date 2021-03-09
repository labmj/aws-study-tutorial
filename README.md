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


## AWS Pipeline 구축 실습 (BASE)

### CodeCommit repository 생성 및 로컬 연결
- https://console.aws.amazon.com/iam/ 에접속하여 유저 생성 (권한 부여)
- 생성된 유저 선택후 Security credentials탭에서 HTTPS Git credentials for AWS CodeCommit에 Generate로 자격증명 다운받기 (노출금지)
- https://console.aws.amazon.com/codecommit/ 에 접속하여 Create bucket 선택
- 아래 그림과 같이 설정 후 저장소 생성 선택 (Repository name : MyDemoRepo)
![image](https://user-images.githubusercontent.com/79297534/110452218-5292b580-8108-11eb-9773-290b4b8c778b.png)
- 저장소 HTTPS 복제하기
![image](https://user-images.githubusercontent.com/79297534/110458999-c2586e80-810f-11eb-8559-415b41656469.png)
- git SSL 인증서 검증 끄기 : git config --global http.sslVerify false
- 복사한 HTTPS를 git clone 뒤에 붙여서 명령어 실행
- 실행후 자격증명의 계정 정보 입력
- 로컬에 복제 확인

![image](https://user-images.githubusercontent.com/79297534/110459637-8f62aa80-8110-11eb-9d8e-e35560862d7c.png)

- https://docs.aws.amazon.com/ko_kr/codepipeline/latest/userguide/samples/SampleApp_Linux.zip 해당 파일 설치후 C:\tmp\MyDemoRepo 에 압축풀기
- 폴더구조

![image](https://user-images.githubusercontent.com/79297534/110463486-4f51f680-8115-11eb-9ffd-27bea20357e0.png)

- 깃 설정 및 커밋 체크 (cd c:\temp\MyDemoRepo)

![image](https://user-images.githubusercontent.com/79297534/110462300-d605d400-8113-11eb-8cb6-942db5ac0517.png)
![image](https://user-images.githubusercontent.com/79297534/110462368-f33aa280-8113-11eb-860b-5c846629d30d.png)

- (Amazon EC2 Windows instances 생성 및 CodeDeploy agent 설치)에서 만든 EC2InstanceRole-0309 활용 
- https://console.aws.amazon.com/ec2/ 에 접속하여 [Launch instance](인스턴스 실행) 선택
- Amazon Linux 2를 검색하고, 리스트 중 Amazon Linux 2 AMI (HVM), SSD Volume Type 선택
![image](https://user-images.githubusercontent.com/79297534/110467225-1a946e00-811a-11eb-9bd3-4d3530beb750.png)
- t2.micro선택 후 다음 단계 이동
- 아래 그림과 같이 설정후 다음 단계 이동
![image](https://user-images.githubusercontent.com/79297534/110467562-8676d680-811a-11eb-9995-68766918abc4.png)
- 고급 세부 정보 추가후 다음 단계 이동
![image](https://user-images.githubusercontent.com/79297534/110467853-d9e92480-811a-11eb-8ff1-9cde2839530e.png)
- 4. 스토리지 추가 페이지 수정 없이 다음: 태그로 이동
- 태그 추가하여 아래 그림과 같이 채워주고 다음 단계 이동
![image](https://user-images.githubusercontent.com/79297534/110468309-74496800-811b-11eb-8960-c159a43a4c03.png)
- 다음 그림과 같이 설정후 다음 단게 이동
![image](https://user-images.githubusercontent.com/79297534/110469010-6d6f2500-811c-11eb-9914-49ba89ec2d2f.png)
- 시작하기를 선택후 키 페어 없이 계속으로 설정후 인스턴스 시작 선택
![image](https://user-images.githubusercontent.com/79297534/110469086-8677d600-811c-11eb-977e-1d74a1d149f1.png)


--------codecommit 저장소 관련(위)(미완) /  S3 저장소 관련(아래)(완)------------------------------------------------

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
- 실습에서는 붙여넣기한 글에서 bucket-name을 aws-codedeploy-ap-northeast-2로 기입후 Next: Add Storage 선택
![image](https://user-images.githubusercontent.com/79297534/110431441-2a4b8c80-80f1-11eb-9dbd-8bb41779b18d.png)
![image](https://user-images.githubusercontent.com/79297534/110430504-d55b4680-80ef-11eb-8f09-d2ee26bd3f92.png)
![image](https://user-images.githubusercontent.com/79297534/110430683-15bac480-80f0-11eb-8dec-b2a3ef33b726.png)
- Step 4: Add Storage에서는 변경 없이 Next: Add Tags 선택
- Add Tag후 그림과 같이 입력후  Next: Configure Security Group 선택 (Key : Name / Value : MyCodePipelineDemo) 
![image](https://user-images.githubusercontent.com/79297534/110432030-f6bd3200-80f1-11eb-9f4f-c56ba1e1e6c2.png)
- 80 port 허용후 Review and Launch 선택
![image](https://user-images.githubusercontent.com/79297534/110434219-e2c6ff80-80f4-11eb-944f-a7c9eb256cd6.png)
- 확인후 Launch 선택, 아래 화면의 Proceed without a key pair 선택후 Launch Instances 선택
![image](https://user-images.githubusercontent.com/79297534/110434680-77316200-80f5-11eb-86a3-60df995147be.png)

### CodeDeploy에서 어플리케이션 생성
- https://console.aws.amazon.com/codedeploy 에 접속후 Create application 선택
- 어플리케이션명은 MyDemoApplication-0309, 플랫폼은 EC2/On-premises 선택 후 어플리케이션 생성 버튼 선택 
![image](https://user-images.githubusercontent.com/79297534/110435607-8fee4780-80f6-11eb-8587-8c8d907df4e6.png)
- Service role을 위해 새창으로 https://console.aws.amazon.com/iam/ 에 접속후 우측 탭의 Roles 선택
- EC2/온프레미스 배포 - CodeDeploy 선택 / Amazon ECS 배포 - CodeDeploy - ECS 선택 / AWS Lambda 배포 Lambda용 CodeDeploy 선택
- Next: Permissions ->  Nex: Tags -> Next: Review 선택 (실습에서는 CodeDeploy 선택)
![image](https://user-images.githubusercontent.com/79297534/110439387-be6e2180-80fa-11eb-9e62-bfa2b5455e7c.png)
![image](https://user-images.githubusercontent.com/79297534/110439667-12790600-80fb-11eb-83f9-cc347344a6bb.png)
- Role name을 CodeDeployServiceRole-3090로 지정해준 후 Create role 선택
- 만들어진 Role선택후 Trust relationships탭에서 Edit trust relationships 선택 
![image](https://user-images.githubusercontent.com/79297534/110440712-502a5e80-80fc-11eb-8c5c-742d642b9449.png)
![image](https://user-images.githubusercontent.com/79297534/110441225-dd6db300-80fc-11eb-8336-044be1495e7e.png)

- Service부분을 다음과 같이 수정 (일부 엔드포인트에 대한 액세스 권한만 부여하려면 정책 문서 상자의 콘텐츠를 다음 정책)

 "Service": [
                    "codedeploy.us-east-2.amazonaws.com",
                    "codedeploy.us-east-1.amazonaws.com",
                    "codedeploy.us-west-1.amazonaws.com",
                    "codedeploy.us-west-2.amazonaws.com",
                    "codedeploy.eu-west-3.amazonaws.com",
                    "codedeploy.ca-central-1.amazonaws.com",
                    "codedeploy.eu-west-1.amazonaws.com",
                    "codedeploy.eu-west-2.amazonaws.com",
                    "codedeploy.eu-central-1.amazonaws.com",
                    "codedeploy.ap-east-1.amazonaws.com",
                    "codedeploy.ap-northeast-1.amazonaws.com",
                    "codedeploy.ap-northeast-2.amazonaws.com",
                    "codedeploy.ap-southeast-1.amazonaws.com",
                    "codedeploy.ap-southeast-2.amazonaws.com",
                    "codedeploy.ap-south-1.amazonaws.com",
                    "codedeploy.sa-east-1.amazonaws.com"
                ]

![image](https://user-images.githubusercontent.com/79297534/110442466-3722ad00-80fe-11eb-9c6a-7cc57e560188.png)

- 다시 이전 탭으로 돌아가 Create deployment group 선택
![image](https://user-images.githubusercontent.com/79297534/110436942-2707cf00-80f8-11eb-8fba-c4d84f3dbc3c.png)
- Create deployment group선택후 아래의 그림들처럼 기입 (group name:MyDemoDeploymentGroup-0309) 
![image](https://user-images.githubusercontent.com/79297534/110443469-49511b00-80ff-11eb-985c-24d4b6555619.png)
![image](https://user-images.githubusercontent.com/79297534/110443668-84534e80-80ff-11eb-9a51-5dc3a5db164c.png)
![image](https://user-images.githubusercontent.com/79297534/110443773-a351e080-80ff-11eb-8cd5-8886eb3338e3.png)
- 현실습에서는 로드밸런서를 설정할 필요없음 해당 그림들 처럼 수정후  Create deployment group 선택

### CodePipeline 
- http://console.aws.amazon.com/codesuite/codepipeline/home 에 접속후 파이프라인 생성 선택
- 아래 그림과 같이 설정후 다음단계로 이동 (Pipeline name : MyFirstPipeline-0309)
![image](https://user-images.githubusercontent.com/79297534/110446149-33912500-8102-11eb-9946-627172c6cfae.png)
- 아래 그림과 같이 설정후 다음단계로 이동
![image](https://user-images.githubusercontent.com/79297534/110446586-b4e8b780-8102-11eb-9a11-8d653acb8d94.png)
- Skip build stage 선택
- 아래 그림과 같이 설정후 다음단계로 이동 
![image](https://user-images.githubusercontent.com/79297534/110447412-8ddeb580-8103-11eb-9927-70b473208c89.png)
- 파이프라인 생성 선택
- 완료된 화면에서 Deploy쪽에 Details 선택 -> Deployment lifecycle events에 Instance ID탭에 ID 선택 -> Public IPv4 DNS탭에 주소를 복사해서 주소창에 붙여 넣기 -> S3 버킷에 업로드한 샘플 애플리케이션에 대한 인덱스 페이지 출력시 성공완료

## 참고자료
- https://docs.aws.amazon.com/codepipeline/latest/userguide/tutorials-simple-s3.html#s3-create-s3-bucket
- https://docs.aws.amazon.com/codedeploy/latest/userguide/getting-started-create-service-role.html
- https://docs.aws.amazon.com/codepipeline/latest/userguide/tutorials-simple-codecommit.html

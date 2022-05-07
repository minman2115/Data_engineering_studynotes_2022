.

Data_Engineering_TIL(20220416)

[참고자료]

AWS 워크샵 : "AWS 서버리스 애플리케이션의 진화 : 대용량 트래픽도 서버리스로 처리할 수 있습니다!"

URL : https://catalog.us-east-1.prod.workshops.aws/workshops/05e3e1f9-5d5a-4cc5-9899-df114def68e7/ko-KR/

[실습목적]

AWS Lambda 를 중심으로 Amazon API Gateway 를 연동해 간단한 REST API 기반 서버리스 애플리케이션을 구현해본다.

실습 아키텍처는 아래와 같다.

<img width="1344" alt="1" src="https://user-images.githubusercontent.com/41605276/163660589-1f7d7443-806e-4262-9d74-c0e75a7c9c57.png">

[실습요약]

Step 1. 서버리스 애플리케이션에서 사용할 Amazon VPC 생성

Step 2. Amazon RDS 생성

Step 3. AWS Lambda 구성

Step 4. Lambda Layer 구성

Step 5. Lambda 테스트

Step 6. Amazon API Gateway 구성

[실습 상세내용]

### Step 1. 서버리스 애플리케이션에서 사용할 Amazon VPC 생성

이번 실습에서 구성하는 리소스들을 위한 네트워크 환경을 먼저 구성합니다. 데이터베이스를 Amazon DynamoDB 와 같은 리전 서비스를 활용한다면 필요없는 단계겠지만 오늘 실습에는 Amazon RDS 를 활용하기 때문에 이를 위한 VPC 구성이 우선 되어야 합니다. VPC 와 실습에 사용할 서브넷 구성은 CloudFormation 을 통해 수행합니다.

step 1-1) AWS 콘솔에서 AWS CloudFormation 서비스로 이동합니다. 리전은 서울(ap-northeast-2)을 사용합니다.

step 1-2) 화면의 [Create stack] 버튼을 클릭하여 CloudFormation 스택 생성을 시작합니다.

step 1-3) Template source 는 [Upload a template file] 을 선택하고 [Choose file] 을 클릭한 뒤 다음 serverless-app-vpc.yaml 템플릿을 다운받아 업로드합니다. [Next] 를 클릭해 다음으로 진행합니다.

<img width="911" alt="2" src="https://user-images.githubusercontent.com/41605276/163660761-8747bfad-58b7-4eb3-9127-0914767c2e10.png">

참고로 serverless-app-vpc.yaml 스크립트는 아래와 같음.

```yaml
AWSTemplateFormatVersion: "2010-09-09"
Description: 'Cloudformation template to create VPC for workshop (Optimize Serverless Application on AWS)'
Resources:
  VPC:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: 10.0.0.0/16
      EnableDnsSupport: true
      EnableDnsHostnames: true
      Tags:
        - Key: Name
          Value: serverless-app
  
  InternetGateway:
    Type: AWS::EC2::InternetGateway
    Properties:
      Tags:
        - Key: Name
          Value: serverless-app-igw

  InternetGatewayAttachment:
    Type: AWS::EC2::VPCGatewayAttachment
    Properties:
      InternetGatewayId: !Ref InternetGateway
      VpcId: !Ref VPC

  PrivateSubnet1:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref VPC
      AvailabilityZone: ap-northeast-2a
      CidrBlock: 10.0.1.0/24
      MapPublicIpOnLaunch: false
      Tags:
        - Key: Name
          Value: lambda-subnet-a

  PrivateSubnet2:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref VPC
      AvailabilityZone: ap-northeast-2c
      CidrBlock: 10.0.2.0/24
      MapPublicIpOnLaunch: false
      Tags:
        - Key: Name
          Value: lambda-subnet-c

  PrivateSubnet3:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref VPC
      AvailabilityZone: ap-northeast-2a
      CidrBlock: 10.0.10.0/24
      MapPublicIpOnLaunch: false
      Tags:
        - Key: Name
          Value: rds-subnet-a

  PrivateSubnet4:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref VPC
      AvailabilityZone: ap-northeast-2c
      CidrBlock: 10.0.20.0/24
      MapPublicIpOnLaunch: false
      Tags:
        - Key: Name
          Value: rds-subnet-c

  PrivateSubnet5:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref VPC
      AvailabilityZone: ap-northeast-2a
      CidrBlock: 10.0.100.0/24
      MapPublicIpOnLaunch: false
      Tags:
        - Key: Name
          Value: secret-subnet-a

  PrivateSubnet6:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref VPC
      AvailabilityZone: ap-northeast-2c
      CidrBlock: 10.0.200.0/24
      MapPublicIpOnLaunch: false
      Tags:
        - Key: Name
          Value: secret-subnet-c

  PublicSubnet1:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref VPC
      AvailabilityZone: ap-northeast-2a
      CidrBlock: 10.0.0.0/24
      MapPublicIpOnLaunch: true
      Tags:
        - Key: Name
          Value: cloud9-subnet-a

  PublicRouteTable:
    Type: AWS::EC2::RouteTable
    Properties:
      VpcId: !Ref VPC
      Tags:
        - Key: Name
          Value: serverless-app-routes

  DefaultPublicRoute:
    Type: AWS::EC2::Route
    DependsOn: InternetGatewayAttachment
    Properties:
      RouteTableId: !Ref PublicRouteTable
      DestinationCidrBlock: 0.0.0.0/0
      GatewayId: !Ref InternetGateway

  PublicSubnet1RouteTableAssociation:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Properties:
      RouteTableId: !Ref PublicRouteTable
      SubnetId: !Ref PublicSubnet1

Outputs:
  VPC:
    Description: serverless-app-vpc
    Value: !Ref VPC
```

step 1-4) [Stack name] 에는 `serverless-app-vpc`를 입력하고 [Next] 를 클릭합니다. 다른 옵션은 모두 기본값을 유지한채 마지막 리뷰 화면에서 [Create stack] 버튼을 선택해 스택생성을 시작합니다.

step 1-5) 잠시 후 다음과 같이 스택 생성이 완료되면 VPC 와 아래와 같은 7개의 서브넷이 생성되고 기본적인 라우팅도 설정됩니다.

<img width="413" alt="3" src="https://user-images.githubusercontent.com/41605276/163660806-054e5b74-fd27-4bc5-8f38-935e932d1b73.png">

step 1-6) 기본적인 VPC 생성은 완료되었습니다. AWS 콘솔에서 Amazon VPC 서비스로 이동합니다.

step 1-7) RDS 와 Lambda 에서 각각 사용할 보안 그룹을 생성합니다. RDS 의 경우 이 후 생성할 Lambda 와 RDS Proxy 에서 접근이 허용되도록 구성할 것입니다.

step 1-8) 좌측의 [Security Groups] 메뉴로 이동한 뒤 [Create security group] 버튼을 클릭합니다.

step 1-9) 우선 Lambda 에서 사용할 보안 그룹을 생성합니다. [Security group name] 에는 `lambda-sg` 를 입력하고 [Description] 을 적은 뒤 [VPC] 는 CloudFormation 을 통해 생성한 `serverless-app`을 선택합니다. 그리고 인바운드/아웃바운드 룰을 아래와 같이 설정해줍니다. 그런 다음에 하단의 [Create security group] 버튼을 클릭하여 완료합니다.

<img width="870" alt="4" src="https://user-images.githubusercontent.com/41605276/163660839-2404294a-49e1-4491-b54c-6d995fee1bfe.png">

<img width="1368" alt="4-1" src="https://user-images.githubusercontent.com/41605276/163665106-32a66eda-23eb-4e4e-8b9f-2920ca96d05d.png">

<img width="1337" alt="422" src="https://user-images.githubusercontent.com/41605276/167246555-75d5937b-f8cd-4141-9e4c-1c179a3c8f0b.png">

step 1-10) 이어서 RDS 에서 사용할 보안 그룹을 생성합니다. 앞서와 동일하게 보안 그룹 생성 메뉴로 이동한 뒤 [Security group name] 에는 `rds-sg` 를 입력하고 [Description] 을 적은 뒤 [VPC] 는 동일하게 `serverless-app` 을 선택합니다.

step 1-11) 하단의 Inbound rules 에서 [Add rule] 버튼을 클릭하여 인바운드 규칙을 추가합니다. [Type] 은 MYSQL/Aurora 를 선택하고 [Source] 에는 앞서 생성한 `lambda-sg`를 선택합니다. 아래 [Create security group] 을 선택하여 완료합니다. 마찬가지로 아웃바운드 룰을 설정해줍니다.

<img width="931" alt="5" src="https://user-images.githubusercontent.com/41605276/163660869-8045e960-c09e-41da-a877-b55a25723dcb.png">

### Step 2. Amazon RDS 생성

이번 실습은 많은 분들이 사용 중이고 가장 익숙한 RDBMS 를 기준으로 이를 서버리스 애플리케이션에서 활용하는 방법을 살펴봅니다. AWS 에는 사용할 수 있는 다양한 데이터베이스 옵션  이 제공되고 있고 서버리스 데이터 베이스인 Amazon DynamoDB 를 활용하는 사례가 많이 있습니다. 하지만 최근에는 Amazon Aurora Serverless  나 Amazon RDS Proxy  등의 새로운 기능이 추가되어 조금 더 익숙한 데이터베이스를 서버리스 애플리케이션에서도 활용할 수 있게 되었습니다.

이번 단계에서는 이 후 실습에서 사용할 Amazon RDS for MySQL 을 생성하겠습니다.

step 2-1) AWS 콘솔에서 Amazon RDS 서비스로 이동합니다.

step 2-2) 좌측의 [Subnet groups] 메뉴로 이동한 뒤 [Create DB Subnet Group] 을 선택합니다.

step 2-3) [Name] 에는 `rds-subnet-group`을 입력하고 [Description] 을 입력한 뒤 [VPC] 는 `serverless-app`을 선택합니다.

step 2-4) 하단의 Add subnets 의 [Availability Zones] 에는 `ap-northeast-2a` 와 `ap-northeast-2c` 두가지를 선택합니다.

step 2-5) [Subnets] 에는 앞서 데이터베이스용으로 만든 서브넷 두 가지 `10.0.10.0/24` 와 `10.0.20.0/24` 를 선택하고 [Create] 를 클릭하여 완료합니다.

<img width="986" alt="6" src="https://user-images.githubusercontent.com/41605276/163661192-0a891100-27ca-4c9b-8262-e317d259359b.png">

step 2-6) RDS 생성을 시작합니다. 좌측의 [Databases] 옵션으로 이동한 뒤 [Create database] 를 선택합니다.

step 2-7) [Engine options] 에는 [MySQL] 을 선택합니다. [Version] 은 `MySQL 5.7.33` 을 선택합니다. 참고로 RDS Proxy 는 2021년 5월을 기준으로 MySQL 5.6, 5.7 을 지원하고 있습니다.

<img width="862" alt="7" src="https://user-images.githubusercontent.com/41605276/163661245-efd77e75-aa8e-4419-b93c-df3ff7ae9059.png">

step 2-8) 하단 Settings 에서 [DB instance identifier] 에는 `serverless-app-rds` 를 입력합니다. Credentials Settings 에서 [Master username] 은 기본 값인 admin 으로 두고 [Master password] 와 [Confirm password] 에는 기억할 수 있는 임의의 패스워드 를 입력합니다.

step 2-9) [DB instance class] 는 `db.m5.large` 로 변경하고 아래 Storage 옵션에서 [Storage type] 은 General Purpose (SSD) 로 변경합니다. [Allocated storage] 는 `20 GiB` 를 입력하고 아래 [Enable storage autoscaling] 옵션은 비활성화 해줍니다.

<img width="1318" alt="8" src="https://user-images.githubusercontent.com/41605276/163661329-176781b7-8124-4030-9ede-ef078c81972e.png">

step 2-10) 워크샵 환경에서는 Multi-AZ 배포를 할 필요는 없기 때문에 Availability & durability 의 [Multi-AZ deployment] 옵션은 `Do not create a standby instance` 를 체크합니다. 프로덕션 환경에서는 standby instance 옵션을 사용하는 것이 좋습니다.

step 2-11) 아래 Connectivity 옵션에서 [Virtual private cloud (VPC)] 는 `serverless-app`을 선택하고 [Subnet group] 은 `rds-subnet-group` 을 선택합니다.

step 2-12) [VPC security group] 은 Choose existing 을 선택하고 [Existing VPC security groups] 에 앞서 생성한 `rds-sg` 를 추가로 선택해줍니다.

<img width="737" alt="9" src="https://user-images.githubusercontent.com/41605276/163661387-4940b692-8e7f-49bd-ae29-c9b0629fbd92.png">

step 2-13) 다른 옵션은 기본값을 유지한채 [Create database] 를 선택하여 RDS 생성을 완료합니다.

### Step 3. AWS Lambda 구성

이번 단계부터 본격적으로 서버리스 애플리케이션의 핵심이 되는 AWS Lambda 함수를 구성합니다. 이렇게 구성하는 Lambda 함수는 RESTful 한 방식으로 동작하며 앞서 생성한 데이터베이스에 쿼리를 하게 됩니다. 이를 위해 VPC 의 리소스에 액세스 하는 Lambda 함수를 구성하는 작업 등을 수행합니다.

AWS 콘솔에서 AWS Lambda 서비스로 이동합니다.

step 3-1) [Create function] 버튼을 클릭하여 함수 생성을 시작합니다.

step 3-2) [Function name] 에는 `serverless-app-lambda` 를 입력하고 [Runtime] 은 `Python 3.8` 을 선택합니다.

step 3-3) 하단의 [Advanced settings] 의 드롭다운 버튼을 클릭합니다.

step 3-4) Enable VPC 체크박스를 클릭하고 Network 에서 [VPC - optional] 에는 `serverless-app` 을 선택하고 [Subnets] 는 `lambda-subnet-a` 와 `lambda-subnet-c` 를 선택합니다. 

step 3-5) [Security groups] 는 `lambda-sg` 를 선택합니다.

<img width="1386" alt="10" src="https://user-images.githubusercontent.com/41605276/163661728-f3668cad-e3ee-4e12-8228-d37af76d010f.png">

step 3-6) 위와 같이 구성했다면 [Create function] 버튼을 클릭하여 Lambda 함수를 생성을 완료합니다.

step 3-7) 생성한 Lambda 함수 화면으로 자동으로 이동합니다. 화면 중간의 [Code source] 에 해당 Lambda 함수에 포함된 코드를 확인할 수 있습니다. 좌측의 `lambda_function.py` 를 더블 클릭하여 선택합니다.

step 3-8) 기본으로 제공되는 코드를 삭제한 뒤 다음의 Python 코드를 붙여 넣습니다.

```python
import json
import pymysql

def lambda_handler(event, context):
    db = pymysql.connect(
        host='{YOUR RDS ENDPOINT}', 
        user='{YOUR DATABASE MASTER USERNAME}', 
        password='{YOUR MASTER PASSWORD}'
        )

    cursor = db.cursor()
    
    cursor.execute("select now()")
    result = cursor.fetchone()

    db.commit()
    db.close()
    
    return {
        'statusCode': 200,
        'body': json.dumps(result[0].isoformat())
    }
```

step 3-9) 코드 상 5:9 라인의 `pymysql.connect()` 부분의 host, user, password 부분에 대한 업데이트가 필요합니다. 우선 host 에 대한 정보는 앞서 생성한 RDS 에서 확인할 수 있습니다.

<img width="430" alt="11" src="https://user-images.githubusercontent.com/41605276/163661797-0b21195a-bf78-4ea8-abd0-63fc19670642.png">

step 3-10) RDS Endpoint 정보를 확인하여 아래의 내용을 Lambda 함수에 업데이트 합니다.

```text
host : {Your RDS Endpoint}
user : admin
password : {설정한 패스워드}
```

참고로 이번실습에서는 Lambda 에서 RDS 에 연결하기 위해 DB 의 크리덴셜을 직접 코드에 입력하여 접속하였는데 이는 보안상 안전한 방법은 아닙니다. 실제 프로덕션 환경에는 AWS Secrets Manager 등을 활용 해 DB 크리덴셜 정보를 하드코딩하지 않는 방법이 더 안전합니다.

step 3-11) 변경을 완료했다면 [Deploy] 버튼을 클릭해 배포를 완료합니다.

### Step 4. Lambda Layer 구성

앞서 Step 3에서 작성한 Lambda 함수에는 pymysql 이라는 라이브러리가 포함되어 있습니다. 기본적으로 Lambda 에는 AWS SDK for Python (Boto3) 와 각각의 런타임 별 기본 라이브러리는 포함되어 있습니다. 그 외에 다른 라이브러리의 사용을 위해서는 Lambda 배포 패키지  를 사용하여 Lambda 함수 코드를 배포해야 합니다.

여기에서 한단계 더 나아가 Lambda Layer를 활용할 수 있습니다. 이 기능을 활용하여 여러 Lambda 함수 간에 코드를 공유할 수 있고, .zip 배포 패키지 크기를 줄일 수 있으므로 Lambda 를 통한 운영에도 도움이 됩니다. Lambda 함수별로 최대 5개의 Lambda Layer 를 포함할 수 있습니다.

이번 실습에서는 pymysql 라이브러리를 Lambda Layer 로 구성한 뒤 이를 함수가 참조하는 구성을 합니다.

step 4-1) 우선 다음 pypi.org(https://pypi.org/project/PyMySQL/#files)에서 Python 3 용 tar.gz 압축 파일을 다운 받아 zip 으로 새로 압축하거나, https://github.com/aws-samples/the-evolution-of-aws-serverless-applications/blob/main/module2/src/pymysql.zip 링크를 통해 pymysql 을 다운받습니다.

아래와 같이 zip 파일을 다운로드 받자.

```console
$ pwd
/Users/minman/Downloads

$ wget https://github.com/aws-samples/the-evolution-of-aws-serverless-applications/raw/main/module2/src/pymysql.zip
--2022-04-16 14:00:09--  https://github.com/aws-samples/the-evolution-of-aws-serverless-applications/raw/main/module2/src/pymysql.zip
github.com (github.com) 해석 중... xxxxxxxxxxxxxxxx, xxxxxxxxxxxxxxxx
다음으로 연결 중: github.com (github.com)|xxxxxxxxxxxxxxxx... 연결했습니다.
HTTP 요청을 보냈습니다. 응답 기다리는 중... 302 Found
위치: https://raw.githubusercontent.com/aws-samples/the-evolution-of-aws-serverless-applications/main/module2/src/pymysql.zip [따라감]
--2022-04-16 14:00:10--  https://raw.githubusercontent.com/aws-samples/the-evolution-of-aws-serverless-applications/main/module2/src/pymysql.zip
raw.githubusercontent.com (raw.githubusercontent.com) 해석 중... xxxxxxxxxxxxxxxx, ...
다음으로 연결 중: raw.githubusercontent.com (raw.githubusercontent.com)|xxxxxxxxxxxxxxxxx ... 연결했습니다.
HTTP 요청을 보냈습니다. 응답 기다리는 중... 200 OK
길이: 48270 (47K) [application/zip]
저장 위치: `pymysql.zip'

pymysql.zip         100%[===================>]  47.14K   164KB/s    /  0.3s

2022-04-16 14:00:10 (164 KB/s) - `pymysql.zip' 저장함 [48270/48270]
```

step 4-2) 왼쪽 메뉴 탭의 Additional resources 의 [Layers] 메뉴로 이동한 뒤 [Create layer] 버튼을 선택합니다.

step 4-3) [Name] 에는 `pymysql` 을 입력하고 [Upload a .zip file] 을 선택한 뒤 [Upload] 버튼을 클릭해 앞서 다운 받은 `pymysql.zip` 파일을 업로드 합니다. [Compatible runtimes] 에는 `Python 3.8` 을 선택합니다. 그 다음에 [Create] 버튼을 클릭하여 Lambda Layer 생성을 완료합니다.

step 4-4) 앞서 생성한 `serverless-app-lambda` 함수로 이동한 뒤 하단 Layers 메뉴의 [Add a layer] 버튼을 클릭합니다.

step 4-5) Layer source 는 [Custom layers] 를 선택하고 드롭 다운 메뉴에서 `pymysql` 을 선택한 뒤 [Version] 은 `1` 을 선택합니다. 배포를 여러번 했다면 버전은 다르게 보일 수 있습니다. [Add] 버튼을 선택하여 Lambda 에 추가하는 작업을 완료해줍니다.

<img width="1368" alt="12" src="https://user-images.githubusercontent.com/41605276/163662329-8299ac00-8925-4320-804b-532cb6b64af9.png">

step 4-6) Layers 메뉴에 pymysql 이 추가된 것을 확인할 수 있습니다. Lambda 함수 구성이 끝났습니다.

### Step 5. Lambda 테스트

step 5-1) 구성한 Lambda 의 테스를 해봅니다. Lambda 콘솔 중앙의 [Test] 메뉴로 이동합니다.

step 5-2) [Template] 는 `hello-world` 기본값을 사용하고 [Name] 에는 `apptest` 를 입력하고 [Save changes] 를 클릭한 뒤 우측의 [Test] 버튼을 클릭해 테스트를 수행합니다. 다음과 같은 테스트 결과가 표시됩니다.

<img width="1374" alt="13" src="https://user-images.githubusercontent.com/41605276/163662495-8dafca60-bef5-41ba-b25d-8e9dd18a6191.png">

만약 테스트가 실패하면 pymysql.connect 부분에서 `,, ''` 등 syntax 에러가 없는지 확인이 필요합니다. 모든 문자는 `''`로 감싸져야합니다. 또한 host, user 라인의 끝에는 `,` 가 있어야 합니다.

step 5-3) 작성한 Lambda 함수는 단순히 `cursor.execute("select now()")` 를 통해 현재 시간을 반환하는 동작을 합니다. 때문에 아래와 같이 `"statusCode": 200` 와 `"body"` 부분에 RDS 시간이 표시되면 Lambda 와 RDS가 정상적으로 통신하는 것을 확인할 수 있습니다.

```json
{
  "statusCode": 200,
  "body": "\"2021-05-13T06:06:26\""
}
```

한 가지 재밌는 점은 최초 테스트 때의 Duration 을 확인한 뒤 이 후에 다시 테스트를 수행하면 실행 시간이 최초 실행에 비해 줄어드는 것을 확인할 수 있습니다. 이는 Lambda 의 콜드스타트 때문입니다.

### Step 6. Amazon API Gateway 구성

AWS Lambda 는 s3에 파일이 업로드되는 이벤트에 의한 방식으로 호출할 수도 있지만 REST API 작업 등을 위해 Amazon API Gateway 에서 호출하는 방식도 많이 사용되고 있습니다. Amazon API Gateway는 REST 및 WebSocketAPI 등을 생성, 배포, 유지 관리 할 수 있는 AWS 서비스로 모든 규모의 API 를 개발자가 손쉽게 구성할 수 있도록 해줍니다. 이번 실습에서는 Lambda 를 API Gateway 와 함께 사용하여 REST API 호출하는 것을 구성합니다. 

step 6-1) AWS 콘솔에서 Amazon API Gateway 서비스로 이동합니다.

step 6-2) 우측 상단의 [Create API] 를 클릭하고 [REST API] 옵션의 [Build] 버튼을 선택합니다.

<img width="1333" alt="14" src="https://user-images.githubusercontent.com/41605276/163663425-ea4ffb43-2757-4c7c-bfef-943756ff1c97.png">

step 6-3) API 생성 화면에서 Create new API 에는 [New API] 를 선택하고 하단 Settings 의 [API name] 에는 `serverless-app-api` 를 입력합니다. [Endpoint Type] 은 Regional 을 선택합니다. API 트래픽의 오리진에 따라 Edge, Regional, Private 등의 옵션을 제공하고 있습니다. [Create API] 를 클릭하여 API 를 생성합니다.

<img width="1298" alt="15" src="https://user-images.githubusercontent.com/41605276/163663467-ae6e9cd4-68a2-44af-9e27-79f27abdfccc.png">

step 6-4) API 생성이 완료되면 [Resources] 메뉴 상단의 [Actions] 버튼을 드롭 다운 한 뒤 [Create Method] 옵션을 선택합니다. 생성된 빈 드롭 다운 메뉴에서는 [GET] 을 선택한 뒤 체크 버튼을 클릭합니다.

<img width="650" alt="16" src="https://user-images.githubusercontent.com/41605276/163663492-3074da70-6ff5-49f6-a933-d546eccb4ad7.png">

step 6-5) `/ - GET - Setup` 화면이 나타납니다. [Ingegration type] 은 Lambda Function 을 선택하고 [Lambda Region] 은 ap-northeast-2 를 선택합니다. [Lambda Function] 에는 serverless-app-lambda 를 선택합니다. [Save] 를 선택하여 API 메소드 생성을 완료합니다.

<img width="488" alt="17" src="https://user-images.githubusercontent.com/41605276/163663516-298d78d3-de7d-439a-bb29-e82af5e63aad.png">

step 6-6) Add Permission to Lambda Function 팝업이 나타나면 [OK] 를 선택합니다.

step 6-7) 생성한 API 를 배포해줘야 합니다. [Resources] 메뉴 상단의 [Actions] 버튼을 드롭다운 한 뒤 [Deploy API] 를 클릭합니다.

step 6-8) [Deploy stage] 는 [New Stage] 를 선택하고 [Stage name*] 에는 `dev` 를 입력한 뒤 [Deploy] 버튼을 클릭합니다.

<img width="637" alt="18" src="https://user-images.githubusercontent.com/41605276/163663546-2cd12521-86a2-4641-948f-276248edadd2.png">

step 6-9) 각각의 배포 스테이지 별 다양한 설정이 가능하지만 오늘 실습에서 이 부분은 다루지 않겠습니다. 생성한 `dev` 스테이지 상단에 [Invoke URL] 이 표시된 것을 확인할 수 있습니다.

<img width="935" alt="19" src="https://user-images.githubusercontent.com/41605276/163663575-c91f02e7-6f0c-43f9-82f7-54f30840feed.png">


step 6-10) 주소를 복사하여 브라우저에서 연결하거나 혹은 터미널에서 호출해봅니다. Lambda 를 테스트했을 때와 동일한 결과값이 나오는 것을 확인할 수 있습니다.

```console
curl https://39waamw537.execute-api.ap-northeast-2.amazonaws.com/dev
{"statusCode": 200, "body": "\"2022-04-16T07:02:25\""}%
```

step 6-11) Lambda 서비스로 이동하여 `serverless-app-lambda` 함수 콘솔로 접속해보면 트리거에 API Gateway 가 추가된 것을 확인할 수 있습니다.
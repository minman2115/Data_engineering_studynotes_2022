.

Data_Engineering_TIL(20220409)

[참고자료]

AWS 워크샵 : "AWS 서버리스 애플리케이션의 진화 : 대용량 트래픽도 서버리스로 처리할 수 있습니다!"

URL : https://catalog.us-east-1.prod.workshops.aws/workshops/05e3e1f9-5d5a-4cc5-9899-df114def68e7/ko-KR/

[실습목적]

S3 에 파일이 업로드 되는 이벤트가 발생하면 Amazon SNS 를 통해 사용자에게 email 로 알람을 전송하는 간단한 자동화 프로세스를 AWS Lambda 를 통해 구현해본다. 실습 아키텍처는 아래와 같다.

<img width="747" alt="스크린샷 2022-04-09 오후 7 56 33" src="https://user-images.githubusercontent.com/41605276/162569017-adf2871b-9f91-4b43-9a50-cdcdb76a8064.png">

[실습요약]

Step 1. Amazon SNS 구성

Step 2. Amazon S3 구성

Step 3. AWS Lambda 를 구성하여 이벤트 처리

Step 4. 테스트

[실습 상세내용]

### Step 1. Amazon SNS 구성

먼저 SNS 토픽을 하나 생성해준다.

```console
$ aws sns create-topic --name s3-event
{
    "TopicArn": "arn:aws:sns:ap-northeast-2:111111111111:s3-event"
}
```

그런 다음에 아래 명령어와 같이 SNS 서비스 subscribe를 생성해준다.

```console
$ aws sns subscribe --topic-arn arn:aws:sns:ap-northeast-2:111111111111:s3-event --protocol email --notification-endpoint minmantest@gmail.com
{
    "SubscriptionArn": "pending confirmation"
}
```

구독을 해주면 해당 이메일에 아래와 같이 구독이 완료되었다는 이메일이 온다.

<img width="1041" alt="스크린샷 2022-04-09 오후 8 21 27" src="https://user-images.githubusercontent.com/41605276/162569744-5504afa4-bafc-4a4c-a56b-76c5ec67e62b.png">

이때 Confirm subscription 버튼을 누르면 아래 그림과 같이 구독이 완료되었다는 팝업화면이 뜨게 된다.

<img width="589" alt="스크린샷 2022-04-09 오후 8 22 41" src="https://user-images.githubusercontent.com/41605276/162569777-7e9388bd-e075-49cf-bde7-32c45121fa71.png">

### Step 2. Amazon S3 구성

아래 명령어와 같이 s3 버킷을 하나 만들어준다.

```console
$ aws s3api create-bucket --bucket minmantest --region ap-northeast-2 --create-bucket-configuration LocationConstraint=ap-northeast-2
{
    "Location": "http://minmantest.s3.amazonaws.com/"
}
```

### Step 3. AWS Lambda 를 구성하여 이벤트 처리

step 3-1) AWS 콘솔  에서 AWS Lambda 서비스로 이동합니다.

step 3-2) 화면 상단의 [Create Function] 버튼을 클릭합니다.

step 3-3) Lambda 함수를 생성할 수 있는 다양한 옵션이 제공되는 것을 확인할 수 있습니다. 이번 실습은 [Author from scratch] 옵션을 통해 처음부터 함수를 생성합니다.

step 3-4) [Function name] 에는 `s3-email` 을 입력하고, [Runtime] 은 Python 3.8 을 선택합니다. 그 외에도 다양한 프로그래밍 언어를 지원하는 것을 확인할 수 있습니다.

step 3-5) [Change default execution role] 메뉴를 확장한 뒤 [Create a new role from AWS policy templates] 를 선택합니다.

step 3-6) [Role name] 에는 `lambda-sns-pub` 을 입력하고, 아래 [Policy templates - optional] 에는 Amazon SNS publish policy 를 선택합니다.

<img width="821" alt="4" src="https://user-images.githubusercontent.com/41605276/162572707-3be76997-958f-4bcc-bdc0-2a71d3b5102a.png">

step 3-7) 하단의 [Create function] 버튼을 클릭하여 Lambda 함수 생성을 완료합니다.

step 3-8) 코드를 수정하기 전에 잠시 생성된 코드를 살펴보겠습니다. 아래 [Code] 탭의 s3-email 폴더 아래의 lambda_function.py 를 보면 아래와 같이 단순한 Python 코드가 생성된 것을 확인할 수 있습니다.

```python
import json

def lambda_handler(event, context):
    # TODO implement
    return {
        'statusCode': 200,
        'body': json.dumps('Hello from Lambda!')
    }
```

보시는 것처럼 모든 Lambda 함수에는 Handler  가 포함 되어 있습니다. Lambda 함수가 호출되면 실행되는 메소드가 바로 이 Handler 메소드입니다.

이번 실습에 사용되는 함수는 Amazon S3 에 파일이 업로드 되는 이벤트가 발생하면 Amazon SNS 를 통해 사용자에게 이메일을 발송합니다. 이러한 로직은 lambda_handler() 메소드 내부에 구현해주면 됩니다. Lambda 함수의 가장 기본이 되는 구조입니다.

step 3-9) 해당 코드를 모두 삭제한 뒤 아래의 코드를 붙여 넣습니다.

```python
import json
import boto3

sns = boto3.client('sns')

def lambda_handler(event, context):
    response = sns.publish(
        TopicArn = '생성한 SNS Topic 의 ARN',
        Message = event['Records'][0]['s3']['object']['key'] + ' has been ' + event['Records'][0]['eventName'],
        Subject = 'S3 Event',
        )
    # TODO implement
    return {
        'statusCode': 200,
        'body': json.dumps('Hello from Lambda!')
    }
```

step 3-10) 코드의 TopicArn 은 Step 1. Amazon SNS 에서 구성한 Topic 을 참조하여 다음과 같이 변경합니다.

<img width="1382" alt="5" src="https://user-images.githubusercontent.com/41605276/162572831-69769723-b59b-407e-bc29-94e884f633a3.png">

step 3-11) [Deploy] 버튼을 클릭하여 작성한 코드를 저장합니다. 가장 간단하게 AWS 콘솔에서 Lambda 함수를 작성하는 법을 살펴 봤습니다. 

step 3-12) 마지막으로 작성한 Lambda 함수를 호출 할 이벤트를 구성합니다. 상단의 [+ Add trigger] 버튼을 클릭합니다. 

step 3-13) [Select a trigger] 메뉴에는 S3 를 선택합니다. [Bucket] 은 Step 2. Amazon S3 에서 구성한 S3 Bucket 을 선택합니다. 하단의 Recursive invocation 옵션을 체크한 뒤 [Add] 버튼을 클릭하여 트리거 설정을 완료합니다.

<img width="763" alt="6" src="https://user-images.githubusercontent.com/41605276/162572886-5ca03e4d-8590-4836-a0e9-f35864a4fe41.png">

### Step 4. 테스트

아래와 같이 임의의 파일을 생성한 s3 버킷으로 업로드해준다.

```console
$ pwd
/Users/minman/Downloads/test

$ ll
-rw-r--r--  1 minman  staff    10B  4  9 20:56 test.txt

$ cat test.txt
123123123

$ aws s3 cp test.txt s3://minmantest/
upload: ./test.txt to s3://minmantest/test.txt

$ aws s3 ls minmantest
2022-04-09 21:01:09         10 test.txt
```

그러면 아래 그림과 같이 등록된 이메일로 메일이 날라올 것이다.

<img width="1036" alt="7" src="https://user-images.githubusercontent.com/41605276/162573549-41d1f450-5610-4310-b5b4-2b131a402803.png">
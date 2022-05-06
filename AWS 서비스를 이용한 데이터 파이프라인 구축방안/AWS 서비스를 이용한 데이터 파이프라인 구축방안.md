.

Data_Engineering_TIL(20220506)

### [학습자료]

“AWS에서 데이터 레이크에 실시간 배치 데이터 수집하기” 세미나 영상을 보고 공부한 내용입니다.

영상자료 URL : https://youtu.be/S3vdTBbQ2YM

### [아젠다]

- 다양한 데이터 소스로부터 데이터를 S3로 옮기는 방법

어플리케이션 서버에서 S3로 저장하기

RDS에 저장되어 있는 데이터를 S3로 저장하는 다양한 방법 (Glue, DMS)

- 실시간으로 데이터를 저장하고 분석하기

Kinesis Data Streams 및 Kafka

Managed Streaming for Apache Kafka (MSK)

### [데이터 분석에서의 수집, 저장, 분석, 활용]

- 데이터 분석을 위한 파이프라인

![1](https://user-images.githubusercontent.com/41605276/167068822-d64de4e6-4633-4851-a4fd-4e7349075860.png)

데이터 분석의 파이프라인은 위와 같다. 데이터가 있으면 다양한 형태의 데이터를 어떻게 '수집'할지에 대한 부분이 있고, 대용량 데이터를 지속적으로 잘 저장할 수 있는 방법을 고민하는 부분이 '저장' 영역이 있고, 어떤 서비스들로 저장한 데이터를 '분석'할지에 대한 영역이 있고, 잘 처리된 데이터를 시각화 등을 하여 어떻게 '소비'할지 고민하는 부분이 있다. 이런 과정을 통해서 데이터를 갖고 insight를 얻을 수 있다. 결론적으로 데이터를 잘 수집해서 저장을 해야지 뒤에 분석이나 소비를 잘 할 수 있다.

- 데이터 파이프라인 구축에 필요한 AWS 서비스들

![2](https://user-images.githubusercontent.com/41605276/167069303-941d48e8-7f99-4737-8f61-42c6222b5d46.png)

![3](https://user-images.githubusercontent.com/41605276/167069760-68615d34-8f4f-4b31-90a1-62629cd71a01.png)

디비에서 데이터를 가져오기 위한 DMS 서비스가 있고, 스트리밍 데이터를 가져오기 위한 kinesis, Kafka 서비스가 있다. AWS에서는 일반적으로는 데이터를 S3에 저장하고 글루 카탈로그라는 기능을 이용해서 데이터가 어디에 위치해 있고 어떤 스키가인지 관리를 한다. Lake Formation이라는 서비스를 이용해서 데이터 접근제어를 관리하는 거버넌스 기능도 할 수 있다.

Redshift는 DW의 기능을 할 수 있고, 스팍 하둡 클러스터 솔루션을 사용하고 싶다면 EMR을 사용하면 된다. 서버리스 스팍으로 데이터를 처리하고 싶다면 글루를 사용하면 된다. SQL로 데이터를 분석하고 싶다고 한다면 아테나 서비스를 사용하면 되고 실시간 데이터 분석이라던가 검증을 하고자 한다면 OpenSearch Service를 사용하면 된다. 

시각화를 하고자 한다면 퀵사이트를 사용하면 되고, 세이지 메이커라는 서비스를 이용해서 주피터 노트북 환경에서 머신러닝 모델을 개발하고 개발한 모델을 서비스할 수 있다. 추천서비스를 위한 데이터를 잘 가공해 S3에 저장해 두었다면 Personalize 서비스를 이용해서 추천서비스를 인퍼런스 할 수 있다.

- 실시간 데이터 파이프라인 구축을 위한 람다 아키텍처

![4](https://user-images.githubusercontent.com/41605276/167070735-5146a7e3-0ff9-45f9-8591-257f844c3ff6.png)

그러면 만약에 실시간 데이터 파이프라인을 구축한다면 이 다양한 AWS 서비스들을 어떻게 활용해서 구축을 하면 될까. 일반적으로 실시간 데이터 파이프라인을 구축할때 아키텍처는 람다 아키텍처를 많이 채택한다. 데이터 소스에서부터 데이터를 잘 수집해서 메세지 큐 같은곳에 차곡차곡 쌓이면 수초 이내의 데이터들은 스피드 레이어라는 곳에서 리얼타임으로 분석해서 처리하게 된다. 예를 들어서 이상탐지나 알람을 위한 데이터 처리는 이 스피드에서 처리가 되게 된다. 실시간 데이터를 프로세싱하는 엔진들은 대부분 메모리 베이스들이 많아서 테라바이트 이상의 대용량 데이터처리는 실시간으로 하기에는 현실적으로는 어렵다. 그래서 이런 대용량 데이터들 예를들어서 결제 데이터 분석이라던가 사용자 데이터를 조인해서 분석을 한다던가 이런것들은 배치 프로세싱 영역에서 처리를 하게 된다. 

- AWS 서비스를 이용한 람다 아키텍처 구축 방안

![5](https://user-images.githubusercontent.com/41605276/167072572-609d66de-e082-4247-9790-679ae0f89f4c.png)

AWS 서비스를 이용해서 람다 아키텍처를 구축한다면 먼저 스트리밍 데이터를 저장하고 전달할 수 있는 서비스는 키네시스와 카프가가 있을것이다. 스피드 레이어에서 수초내로 SQL 쿼리의 결과값을 추출할 수 있는 키네시스 데이터 어넬리틱스 서비스가 있다. 그리고 엘레스틱서치 솔루션과 거의 동일한 오픈서치 서비스라는게 있는데 여기에 데이터를 넣게 되면 실시간 대시보드를 만들 수 있고, 검색 대시보드도 만들 수 있다. 또한 이상탐지 기능도 내장되어 있어서 이런것들도 구현할 수 있다. 그리고 마이크로 배치로 분단위 통계 등 수초내에 데이터를 처리할 수 있는 스팍 스트리밍 솔루션을 사용할 수 있는 EMR도 있다. 

배치 레이어에서는 S3에 이 스트리밍 데이터를 저장한 후 인메모리 엔진 베이스의 프레스토 기반의 아테나 서비스로 SQL 분석을 할 수 있고, Glue 서비스를 이용해서 서버리스로 스팍엔진을 사용할 수 있다. 또는 EMR이나 DW인 레드시프트에서 처리해서 대용량 데이터를 배치단위로 처리하고 분석할 수 있다. 

![6](https://user-images.githubusercontent.com/41605276/167073545-080c9f25-1f7e-44b1-bf08-43c957502972.png)

람다 아키텍처에서 배치 프로세싱 같은 경우에는 일반적으로 스트리밍 데이터를 S3에 먼저 적재를 하고 그 이후에 아테나나 글루 이런것들을 이용해서 데이터를 처리하고 분석을 하게 된다.

### [다양한 데이터 소스로부터 데이터를 S3에 저장하는 방안]

- 왜 S3로 데이터를 저장해야 하는가

s3는 데이터 레이크를 위한 서비스로 개발되었기 때문에 대용량 데이터를 안전하게 그리고 저렴하게 저장할 수 있다. 그리고 EMR, Glue, Athena 등 다양한 서비스 들과 유기적인 연동이 가능하다. 

- 데이터 사일로 문제

![7](https://user-images.githubusercontent.com/41605276/167074031-02ab8f09-c0e3-47b3-96ed-58281bea8366.png)

간혹가다가 위의 아키텍처와 같이 결제 데이터는 BI 대시보드에서 연결해서 보고 있는데 사용자 행동과 연계해서 뭔가 분석을 하려고 하면 아키텍처상 어려운 문제가 발생한다. 마찬가지로 사용자 행동 데이터를 장바구니 데이터랑 결제 데이터를 혼합해서 뭔가 분석하기에도 어려운 구조이다. 또한 디비에 데이터가 점점 많이 쌓이게 되면 람다 function이 실행되면서 실패할 수 있는 경우도 있을 것이다. 따라서 결론적으로 RDS나 다이나모 디비나 ec2에서 발생하는 디비나 모두 S3에 저장을 해서 분석하는 서비스들이 s3에서 데이터를 가져다가 혼합해서 분석을 할수 있고, 대용량 데이터는 람다 function이 아닌 Glue나 아테나 등 대용량 데이터를 처리할 수 있는 서비스를 사용할 수 있다.

- 아테나를 이용한 SQL 분석

<img width="736" alt="8" src="https://user-images.githubusercontent.com/41605276/167074792-6164aa23-ba07-4b89-9554-c15dd954feb7.png">

예를 들어서 RDS나 다이나모 디비나 ec2에서 발생하는 디비나 모두 S3에 저장을 했다면 아테나 서비스 하나를 이용해서 SQL 로 결제 데이터, 사용자 행동 데이터, 장바구니 데이터 이런 것들을 혼합해서 분석할 수 있다.

- S3에 데이터를 모두 저장한다면 글루, 래드시프트 등 워크로드에 맞는 분석 서비스로 확장이 가능

<img width="1174" alt="9" src="https://user-images.githubusercontent.com/41605276/167075061-565ad0b1-4342-4452-bc2b-8275c1ea5307.png">

s3에 데이터를 file형태로 저장을 했다면 이 데이터가 어떻게 생겼는지에 대한 테이블 스키마를 생성을 먼저 해줘야지 아테나에서 사용이 가능한데 이 스키마 정보를 생성해주는 서비스가 글루 데이터 카탈로그이다. 데이터 카탈로그를 이용해서 데이터 스키마를 만들면 데이터가 어디에 있는지 그리고 이 데이터가 어떻게 생겼는지 파악이 가능하기 때문에 아테나나 글루, EMR spark 등에서 모두 사용이 가능해진다.

위에 그림과 같이 글루라는 ETL 잡을 이용해 글루 카탈로그를 통해 s3에 저장된 데이터를 읽어와서 ETL 처리후 분석을 한 결과 데이터를 다시 s3로 저장할 수 있다. 이렇게 s3에 저장한 데이터를 갖고 Personalize 서비스를 이용해서 추천서비스를 만들거나 아니면 세이지 메이커로 머신러닝을 할 수도 있다. 또한 S3에 저장된 데이터를 레드시프트로 올릴수도 있고, 이렇게 올린 데이터를 퀵사이트를 이용해서 시각화도 할 수 있다. 

결론적으로 S3에 결제데이터 마케팅 데이터, 사용자 행동 데이터 등을 모두 저장해서 통합적으로 분석이 가능해진다.

- 아테나 서비스 관련 세션 영샹

https://youtu.be/MAgd-zeB4QU

- EMR, Glue 관련 세션 영상

https://youtu.be/aavblrrk4Fo

- Redshift 관련 세션 영상

https://youtu.be/4tTMrKwpeuQ

- S3에 저장된 데이터 분석하기 워크샵

<img width="1041" alt="10" src="https://user-images.githubusercontent.com/41605276/167076584-11bc7e9c-a27c-4bc9-ba79-74a9582530d4.png">

https://catalog.us-east-1.prod.workshops.aws/workshops/44c91c21-a6a4-4b56-bd95-56bd443aa449/en-US

### [DB의 데이터를 S3로 저장하는 방안]

디비에 저장된 데이터를 분석하고 싶은 니즈가 상당히 많다. 디비에 데이터가 저장된 경우에는 처음에는 디비에 분석 솔루션이 붙어서 어그리게이션을 하거나 분석을 하게 된다. 디비에 데이터가 적을때는 별문제가 없겠지만 데이터가 점점 디비에 많아지게 되면 디비에 부하가 가게 된다. 이럴 경우에는 서비스에서 실제 사용하는 디비는 그대로 두고 이 디비에 대한 레플리카 디비를 하나 만들어서 이 디비에 있는 데이터를 S3에 먼저 옮겨야 한다. 그러면 이를 어떻게 하면 될까. 

방안 1. RDS에서 S3로 스냅샷 내보내기 기능 이용 (디비가 RDS인 경우에만 가능)

<img width="1024" alt="11" src="https://user-images.githubusercontent.com/41605276/167077654-ee5fa87e-2564-4e0f-8ed5-7e214cb157bf.png">

이 방법은 ad-hoc 한 분석에 적합한 방법이다. 데이터를 디비로부터 지속적으로 덤프해서 s3로 붙는 방법은 아니고 가끔가다가 RDS에 있는 데이터들을 S3로 내려받아서 분석하고 싶은 경우가 있는데 이럴때 사용하기 적합한 방법이다. 어떻게 하면 되냐면 RDS 스냅샷을 S3로 내보내는 기능이 RDS에 있는데 이를 사용하면 된다. RDS에서 S3로 데이터를 내릴때 파케이 포맷으로 내리게 되면 속도는 빠르고 용량은 덜 사용할 수 있는 장점도 있다. 

<img width="849" alt="12" src="https://user-images.githubusercontent.com/41605276/167078180-d3ffe96f-c8b3-4b6a-85c6-69615df002b4.png">

위와 같은 아키텍처로 RDS 스냅샷을 S3로 내리고 아테나로 분석할 수 있다.

URL : https://youtu.be/lyNGeDg6EII

방안 2. EC2에 엠벌크 등을 설치하여 DB에 저장된 최신 데이터를 지속적으로 S3에 업데이트 

방안 1 보다는 사실 디비에 저장된 데이터를 지속적으로 변경분을 업데이트에서 S3에 저장하는 패턴이 일반적이다. 

<img width="881" alt="13" src="https://user-images.githubusercontent.com/41605276/167078777-3888828d-0f57-41fa-8a36-c3314569468d.png">

<img width="863" alt="15" src="https://user-images.githubusercontent.com/41605276/167079534-cb01f161-089e-4906-ae59-1e217c268983.png">

따라서 위와 같은 아키텍처로 EC2 머신위에 스크립트를 짜서 크론텝으로 주기적으로 데이터를 가져오는 프로세스를 구현하거나 아니면 EC2에 엠벌크, 에어플로우 등 오픈소스 솔루션을 설치하여 디비에 있는 데이터를 S3에 주기적으로 저장하는 시스템을 구현할 수도 있다.

embulk 아키텍처는 아래와 같다.

<img width="873" alt="16" src="https://user-images.githubusercontent.com/41605276/167079735-d802b518-e5a8-4071-8475-af0f1c563b62.png">

URL : https://github.sre.pub/ksmin23/embulk-tutorial-cdc-from-mysql-to-s3

아래와 같이 embulk config.yml를 적절하게 설정해서 실행해주면 쉽게 CDC 데이터를 가져와서 S3에 저장할 수 있다.

```yml
in:
  type: mysql
  host: <db_host>
  user: "<db user>"
  password: "<db password>"
  database: test
  table: pet
  select: "name, owner, species, sex, birth"
  where: "c_time >= '2020-06-06 13:23:00' AND c_time < '2020-06-06 13:24:00'"
out:
  type: s3
  path_prefix: logs-json-gzip/out
  file_ext: .json.gz
  bucket: embulk-demo-output-use1
  access_key_id: xxxxxxxxxxxxxxxxxxxxxx
  secret_access_key: yyyyyyyyyyyyyyyyyyyyyyyyyyyyyy
  formatter:
    type: jsonl
    timezone: "UTC"
    date_format: "yyyy-MM-dd'T'HH:mm:ss.SSSZ"
  encoders:
  - type: gzip
    level: 1
```

** 방안 2의 문제점

(1) 디비에 저장된 데이터를 옮기기 위한 코드 개발이나 컨피그 설정이 필요

(2) 프로세스 모니터링 필요

(3) 서버 운영이 필요

(4) 라지 스케일에 대응이 어려움

방안 3. 람다 function을 이용해서 디비에 있는 데이터를 주기적으로 S3에 업데이트

<img width="933" alt="14" src="https://user-images.githubusercontent.com/41605276/167079145-c5eba7b1-24ea-4599-ae0d-bbee9ee70936.png">

EC2 서버를 관리하고 싶지 않으면 서버리스 람다를 사용해도 되지만 일정 수준 이상의 대용량 데이터를 처리하는 것에는 한계가 있다는 점을 유의해야 한다.

방안 4. Glue로 디비에 저장된 데이터를 S3로 저장하기

<img width="873" alt="17" src="https://user-images.githubusercontent.com/41605276/167081230-c57138ee-1cc2-4934-a119-679e663d9697.png">

위에 아키텍처와 같이 글루 서비스를 이용하는 것도 가능하다. 서버리스 스팍이기 때문에 스팍 스크립트만 있으면 되고 라지 스케일도 처리가 가능하다. 그리고 글루잡에 스케쥴링을 걸 수 있어서 주기적으로 데이터 처리도 가능핟. 또한 인프라 관리가 필요없다. 

Glue job source code 예시

<img width="1283" alt="18" src="https://user-images.githubusercontent.com/41605276/167083798-b8001b87-7ce0-46d5-96bb-1d33606551d1.png">

디비에 연결후 북마크를 이용해서 여기에서는 예를들어서 임플로이 넘버라는 것을 이용해서 글루잡이 실행될때마다 어디까지 읽어왔는지 체크하고 이 이후 데이터를 읽어오게 된다. 그리고 지정된 S3 경로에 저장하게 된다. 

```python
# Sample Script
import sys
from awsglue.transforms import *
from awsglue.utils import getResolvedOptions
from pyspark.context import SparkContext
from awsglue.context import GlueContext
from awsglue.job import Job

## @params: [JOB_NAME]
args = getResolvedOptions(sys.argv, ['JOB_NAME'])
sc = SparkContext()
glueContext = GlueContext(sc)
spark = glueContext.spark_session
job = Job(glueContext)
job.init(args['JOB_NAME'], args)
## @type: DataSource
## @args: [database = "database", table_name = "relatedqueries_csv", transformation_ctx = "datasource0"]
## @return: datasource0
## @inputs: []
datasource0 = glueContext.create_dynamic_frame.from_catalog(database = "database", table_name = "relatedqueries_csv", transformation_ctx = "datasource0")
## @type: ApplyMapping
## @args: [mapping = [("col0", "string", "name", "string"), ("col1", "string", "number", "string")], transformation_ctx = "applymapping1"]
## @return: applymapping1
## @inputs: [frame = datasource0]
applymapping1 = ApplyMapping.apply(frame = datasource0, mappings = [("col0", "string", "name", "string"), ("col1", "string", "number", "string")], transformation_ctx = "applymapping1")
## @type: DataSink
## @args: [connection_type = "s3", connection_options = {"path": "s3://input_path"}, format = "json", transformation_ctx = "datasink2"]
## @return: datasink2
## @inputs: [frame = applymapping1]
datasink2 = glueContext.write_dynamic_frame.from_options(frame = applymapping1, connection_type = "s3", connection_options = {"path": "s3://input_path"}, format = "json", transformation_ctx = "datasink2")

job.commit()
```

참고 URL : https://docs.aws.amazon.com/ko_kr/glue/latest/dg/monitor-continuations.html

방안 5. Lake Formation으로 디비에 저장된 데이터를 S3로 저장하기

<img width="845" alt="19" src="https://user-images.githubusercontent.com/41605276/167084219-60c7b7c9-0194-4192-a5a8-9ea97beb5af6.png">

Lake Formation 서비스를 이용해도 되는데 Lake Formation에는 RDS에서 S3로 데이터를 옮겨주는 블루프린트라는 기능이 있다. 그거를 활용하면 된다.

아래 그림과 같이 블루프린트 기능을 이용해서 전체 또는 증분 데이터를 S3로 이동이 가능하다.

<img width="855" alt="20" src="https://user-images.githubusercontent.com/41605276/167084538-edcf1519-5bd8-4ce9-94aa-53adc9add61d.png">

URL : https://aws.amazon.com/ko/blogs/big-data/integrating-aws-lake-formation-with-amazon-rds-for-sql-server/

Lake Formation을 설정하면서 S3 버킷도 설정하고 그 다음에 위와 같이 디비 스냅샷을 할건지 증분 데이터 수집을 할건지 선택을 해서 쉽게 설정이 가능하다.

** 방안 4, 5의 문제점

(1) 서버리스 스팍 (글루)를 알아야지만 사용할 수 있다.

스크립트 개발이 필요하다.

스팍, 글루 환경에 대한 이해가 필요하다.

(2) Lake Formation

비교적 간단하게 설정이 가능하지만 운영중에 문제가 발생하면 글루와 레이크 포메이션에 대해서 둘다 알고 있어야 대응이 가능하다.

방안 6. AWS DMS라는 서비스를 이용하는 방법

<img width="877" alt="21" src="https://user-images.githubusercontent.com/41605276/167085726-c4cac330-4ef0-459e-9bce-46d5df012ba4.png">

Data Migration Service 란

<img width="1334" alt="22" src="https://user-images.githubusercontent.com/41605276/167085823-a462f1c5-b4d0-4e2b-8385-d615f3984095.png">

말 그대로 데이터를 마이그레이션 하기 위한 서비스다. 온프라미스의 디비에 있는 데이터를 AWS의 환경에 RDS에 마이그레이션 하는 용도로 처음에 나온 서비스이다. 데이터를 마이그레이션 하기 위해서는 온프라미스의 데이터를 전체를 복사를 해오고 그런 다음에 증분데이터를 계속 sync를 맞추다가 적정한 수준에 끊어버리고 TO-BE DB로 전환하면 되는 구조다. 이런 구조에서 TO-BE가 DB가 아니라 아래 그림과 같이 S3도 가능하다. 또한 동일기종 디비, 가능한 이기종 디비로 마이그레이션을 할 수 있다.

<img width="1304" alt="23" src="https://user-images.githubusercontent.com/41605276/167086473-a199077f-a77f-4fd4-8160-8363480def6e.png">

아래 그림과 같이 DMS 서비스를 이용하면 데이터 베이스에 저장된 데이터를 목적에 따라서 다양한 곳으로 이동이 가능하다. 아래 그림에는 안나와 있지만 DMS를 이용해서 Redis 같은 일라스틱 캐시로도 데이터를 보낼 수 있다. 또한 데이터 스트림즈와 같이 큐에다가도 보낼 수 있다.

<img width="916" alt="24" src="https://user-images.githubusercontent.com/41605276/167086780-ebb0712c-6997-4e06-a9fd-8b377c2ce09f.png">

DMS에서 지원하는 소스와 타겟은 아래와 같다.

<img width="1260" alt="25" src="https://user-images.githubusercontent.com/41605276/167087271-bc2341f4-a4ef-4241-a83a-41e54cb50591.png">

- DMS를 이용해서 DB to S3로 데이터를 이동시키는 워크샵

<img width="759" alt="26" src="https://user-images.githubusercontent.com/41605276/167087551-5c6a124c-10ad-4aa6-9f1e-b74fe6a2532a.png">

URL : https://catalog.us-east-1.prod.workshops.aws/workshops/976050cc-0606-4b23-b49f-ca7b8ac4b153/en-US/400

- 여기까지 결론

단순한 구현이나 오픈소스의 사용으로도 S3에 데이터를 저장할 수 있지만 개발,설정,모니터링,스케일 등에 대해서 고민할 수 있다.

글루나 레이크 포메이션을 사용하는 옵션도 있지만 이 서비스들이 다양한 기능이 있고 스크립트를 개발해야하기 때문에 트러블 슈팅이 쉽지 않을 수 있다.

DMS를 사용하면 S3를 포함한 다양한 타겟으로 데이터를 이동시킬 수 있다. (이동에 집중)

단 DMS를 사용하면 스키마가 달라지면 데이터 이동에 실패할 수 있고, 원하는 형태로 ETL해서 저장할 수는 없는 단점이 있다.

### [어플리케이션 데이터를 S3로 저장하는 방안]

<img width="1074" alt="27" src="https://user-images.githubusercontent.com/41605276/167088828-35bf149b-917f-4594-8a0a-46d9832fa5b1.png">

어플리케이션 데이터를 S3로 저장하는 방법이 쉽지는 않다. 어플리케이션 데이터를 옮기기 위한 서버가 필요할 것이고, 이 서버들이 죽는 경우를 대비해서 임시로 데이터를 저장할 디비가 필요할 수도 있다. 또한 어플리케이션 사용량이 늘어나면 늘어날 수록 이 데이터를 옮기기 위한 서버의 스펙도 커져야 한다. 그리고 이를 관리해야하는 관리포인트가 발생한다.

이런 고민을 해결할수 있는 서비스가 Kinesis Data Firehose이다. Kinesis data streams에 저장된 데이터를 S3로 저장하기 위한 가장 손쉬운 방법이다. Firehose를 사용하면 Destination이 S3 뿐만 아니라 레드시프트, 일라스틱 서치 등 다양하다. Firehose를 사용해서 S3로 넣게 되면 준실시간(1분 ~ 5분 마다) 정도의 속도로 데이터를 넣을 수 있다. S3에 저장할때는 날짜 형태로 저장할 수도 있고, 기타 원하는 형태로 데이터를 저장할 수 있다. 또한 저장할때 파케이 같이 압축된 형태로 저장할 수 있다.

<img width="1065" alt="28" src="https://user-images.githubusercontent.com/41605276/167089295-6c1af949-96b2-4378-aec8-578f327622c8.png">

아래 그림과 같이 로그나 클릭 스트림을 AWS SDK나 Kinesis Agent를 이용해서 로그를 읽고 쓰고 할 수 있고, 키네시스 데이터 스트림즈로 인제스트도 할 수 있다. 그런 다음에 키네시스 파이어호스를 연결하고 추가적으로 람다를 연결해서 인제스트한 데이터를 전처리하거나 할 수 있다. 그런후에 S3나 레드시프트 등으로 저장할 수 있다.

<img width="975" alt="29" src="https://user-images.githubusercontent.com/41605276/167089901-77992601-9ebc-48a4-8a01-69bf384c06c5.png">

키네시스 에이전트를 이용해서 아래와 같이 인제스트를 할 수 있다. 만약에 EC2를 사용한다고 하면 아래 그림과 같이 예를 들어서 `/tmp/app.log*` 경로로 로그파일을 내릴것이다. 그리고 파이어호스 딜리버리 스트림 이름을 아래와 같이 지정하면 아래와 같은 `/tmp/app.log*` 패턴의 파일들을 파이어호스로 보내게 된다.

<img width="1258" alt="30" src="https://user-images.githubusercontent.com/41605276/167090470-cc292158-53b4-4153-ad8f-f091968c18fe.png">

또는 아래와 같이 SDK를 이용해서 자바나 파이썬을 통해서 데이터를 아래와 같이 set하고 put하면 데이터가 하나씩 파이어 호스로 들어가게 된다.

<img width="1099" alt="31" src="https://user-images.githubusercontent.com/41605276/167090970-ea81cb7e-0fb8-4c80-a380-29a3b6b0fda2.png">

한껀씩 데이터를 보낼수 있는것은 기본이고 아래 그림과 같이 SDK 코딩을해서 리스트 형태로 500건 이상씩 배치로 데이터를 보내는 것도 가능하다.

<img width="1091" alt="32" src="https://user-images.githubusercontent.com/41605276/167091244-b6125cd2-f119-4f14-84f1-20f9b6c4ca2c.png">

파이어호스 설정하는 방법은 쉽다. 아래 그림과 같이 딜리버리 스트림 이름을 정하고, 데스티네이션을 설정해주면 된다.

<img width="1201" alt="33" src="https://user-images.githubusercontent.com/41605276/167091618-7b6efe70-9014-42be-bc56-041c1f84683e.png">

<img width="1058" alt="34" src="https://user-images.githubusercontent.com/41605276/167091718-50473b7d-b28c-4698-95f1-5be1dfe36eff.png">

아래 그림과 같이 EKS 컨테이너를 사용할때도 Firehose를 활용할 수 있다. 컨테이너 레벨에서 아래 그림과 같이 플런트디 어그리게이터를 갖고 파이어호스에 넣는 케이스가 있다.

<img width="883" alt="35" src="https://user-images.githubusercontent.com/41605276/167091881-be8e5191-b8f5-4f8e-b5ee-8289256dce37.png">

ECS 같은 경우에는 아래 그림과 같이 컨테이너에서도 마찬가지로 플런트디 어그리게이터를 갖고 파이어호스에 넣을수도 있다. 또한 ECS 같은 경우에는 파이어 렌즈라는게 있어서 컨피그레이션만 하면 컨테이너 로그들을 파이어호스에 데이터를 저장할수 있다.

<img width="854" alt="36" src="https://user-images.githubusercontent.com/41605276/167092105-e25a2316-9244-4c89-9a10-24dc7884b16f.png">

그래서 아래 그림과 같이 파이어호스를 이용하면 다양한 컴퓨팅 자원에서 나오는 어플리케이션 데이터를 파이어호스의 SDK던 에이전트던 어떤거든 이용을 해서 넣게 되면 S3로 데이터를 저장하는게 용이하다.

<img width="937" alt="37" src="https://user-images.githubusercontent.com/41605276/167092517-c9b98e86-4412-4fff-b9c4-276d5d6f167d.png">

또한 파이어호스에 람다를 연결해서 중간에 데이터를 trnasform도 할수도 있다.

<img width="901" alt="38" src="https://user-images.githubusercontent.com/41605276/167092777-4f4b8a9b-dbe6-4377-b00b-2c53ddee074f.png">

그러면 아래그림과 같이 차피 어플리케이션에 디비로 데이터를 저장할거 파이어호스로도 바로 보낼 수 있다.

<img width="910" alt="39" src="https://user-images.githubusercontent.com/41605276/167092942-cc4291fc-44a1-45dd-a582-eaa3a0f54bda.png">

그러면 아래와 같이 아키텍처를 좀더 간단하게 만들수도 있다는 것이다. 이렇게 하면 디비에 부하를 줄 여지도 없고 분석용 데이터를 편하게 가져올 수 있는 장점도 있다.

<img width="893" alt="40" src="https://user-images.githubusercontent.com/41605276/167093168-18be6fa4-20d8-42e1-b5e3-48cd870cddf3.png">

### [실시간으로 데이터를 저장하고 분석하기]

지금까지는 람다 아키텍처에서 배치영역을 알아본것이고, 이번에는 아래와 같이 스피드 레이어에서 어떻게 데이터를 처리할 것인지에 대해 알아보자.

<img width="926" alt="41" src="https://user-images.githubusercontent.com/41605276/167093563-9e3c64bf-bba6-40c5-a78f-82d709c63853.png">

배치영역에서 실시간 데이터 처리영역으로 가려면 문제가 있다. 예를 들어서 아래 그림과 같이 실시간 데이터를 파이어 호스로 처리해서 S3와 오픈서치 서비스로 보내고 싶은데 문제가 데스티네이션이 하나만 가능하다는 것이다.

<img width="958" alt="42" src="https://user-images.githubusercontent.com/41605276/167093931-7ba85c2c-fe53-4cdb-9b77-93a909ca9269.png">

그러면 아래 그림과 같이 파이어호스를 하나 더 붙여야 하나 라는 고민이 생길것이다. 그러면 EC2나 EKS, ECS에서 딜리버리 스트림즈 2개에다가 데이터를 쏴야 하는데 서버를 관리하는 담당자가 안좋아할 것이다.

<img width="941" alt="43" src="https://user-images.githubusercontent.com/41605276/167094405-c16bd7cf-0f28-42a4-aa0f-5d36c2067393.png">

그러면 아래 그림과 같이 데이터를 한번에 받아주고 여러곳에서 가져갈 수 있는 서비스에 대한 니즈가 생길것이다.

<img width="1133" alt="44" src="https://user-images.githubusercontent.com/41605276/167094661-5e1e707d-afe7-4229-b40b-dd1632770ff1.png">

이런 고민을 해결할 수 있는게 아래 그림과 같이 데이터를 한번에 받아서 다양한 종류의 컨슈머들이 가져갈 수 있는 키네시스 데이터 스트림즈나 카프카를 사용하면 된다.

<img width="1002" alt="45" src="https://user-images.githubusercontent.com/41605276/167094854-57e6b674-ea03-4401-8513-a9ae32eeef72.png">

카프카와 키네시스의 내부구조의 차이는 아래와 같다.

<img width="1363" alt="46" src="https://user-images.githubusercontent.com/41605276/167095193-3c01c5a3-28c4-41bf-a3ca-f9bc05dc6203.png">

키네시스와 MSK 비교 참고 영상 : https://youtu.be/9y-aCX5O3Ms

그래서 아래 그림과 같이 키네시스 스트림즈를 사용하게 되면 ec2나 컨테이너 들에서 나오는 데이터를 키네시스 데이터 스트림즈에 넣고 람다나 파이어호스 등 다양한 컨슈머를 이용해서 데이터를 처리할 수 있다. 아래 그림에서 키네시스 스트림즈 대신에 MSK 가 들어가도 무방하다.

<img width="1125" alt="47" src="https://user-images.githubusercontent.com/41605276/167095952-255b2ed1-19bc-47f3-85b9-e73deb039d41.png">

관련해서 실시간 데이터 처리하는 워크샵 정보는 아래와 같다.

<img width="1053" alt="48" src="https://user-images.githubusercontent.com/41605276/167096401-a1b792b5-af3e-4521-926f-15cb81365d90.png">

아래 그림과 같은 아키텍처를 구현하는 워크샵도 있다. (Kinesis Data Stream - Real time Streaming workshop)

<img width="952" alt="49" src="https://user-images.githubusercontent.com/41605276/167096814-c559dd35-d18d-4d45-bc48-ae569080416e.png">

아래와 그림과 같이 같이 MSK Lab 을 이용해서 실습도 해볼 수 있다.

<img width="733" alt="50" src="https://user-images.githubusercontent.com/41605276/167097311-8c3c73ce-e3ce-47b3-a211-bb041cea17d3.png">

Kinesis Analytics를 활용해서 아래와 같이 아키텍처를 구성해서 이상탐지 시스템을 구현할 수 있다.

<img width="1088" alt="51" src="https://user-images.githubusercontent.com/41605276/167097635-baeebcf9-433d-474c-8a84-ea1000048f36.png">

사용자 클릭 데이터가 API 게이트웨이로 들어오면 이거를 바로 키네시스 스트림즈로 보낼 수 있다 그러면 이 클릭 데이터를 Kinesis Analytics에서 SQL 베이스로 10초 단위로 이상데이터를 랜덤포레스트 모델을 이용해서 분석할 수 있다. 그러면 이상징후가 있는 데이터들은 다시 키네시스 스트림즈에 넣고 람다와 SNS 서비스를 이용해서 알람을 줄 수 있다.

<img width="1146" alt="52" src="https://user-images.githubusercontent.com/41605276/167098471-2e93d796-ac15-45b2-bf5a-e145bc9ea23d.png">

또는 위에 그림과 같이 스트리밍 데이터를 오픈서치 서비스에 밀어넣으면 이상탐지 기능이 있어서 그거를 사용해서 또 이상탐지를 할 수 있다.
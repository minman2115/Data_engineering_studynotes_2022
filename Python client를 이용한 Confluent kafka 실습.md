.

Data_Engineering_TIL(20220712)

- 실습 아키텍처 (메세지 흐름 아키텍처)
 
```text
                         schema registry
                                |
간이 프로듀서(로컬 맥북) --> Confluent kafka 브로커 --> Confluent GCS sink connector --> GCS bucket 

** 매새지 형태 : avro 
```

- 실습 요약

### STEP 1) 로컬 맥북에 파이썬 개발환경 구성

```console
$ mkdir kafka_test

$ cd kafka_test

$ python3 -m venv kafka_env

$ cd kafka_env/bin

# 파이썬 가상환경 활성화
$ source activate

# m1 프로세스의 경우 아래의 라이브러리 및 환경설정을 하지 않게 되면 pip install confluent_kafka에 에러가 발생한다.
$ brew install librdkafka

# 아래에 ibrdkafka/1.9.1 부분은 ibrdkafka 버전에 따라 달라짐
$ C_INCLUDE_PATH=/opt/homebrew/Cellar/librdkafka/1.9.1/include LIBRARY_PATH=/opt/homebrew/Cellar/librdkafka/1.9.1/lib pip install confluent_kafka
```

### STEP 2) Confluent kafka API 크레덴셜 발급

API 크레덴셜 발급목록 : 클러스터 API 크레덴셜, schema registry API 크레덴셜

step 2-1)  클러스터 API 크레덴셜 발급

confluent 웹콘솔 --> Environments 메뉴애서 원하는 환경접속 --> Clusters 메뉴 --> 원하는 클러스터 접속 --> 좌측 메뉴에 Data integration --> API Keys 메뉴에서 API 크레덴셜 발급

step 2-2) schema registry API 크레덴셜 발급

confluent 웹콘솔 --> Environments 메뉴에서 원하는 환경접속 --> Schema Registry 메뉴 --> API credentials 메뉴에서 API 크레덴셜 발급

### STEP 3) Schema Registry에 스키마 등록

confluent 웹콘솔 --> Environments 메뉴에서 원하는 환경접속 --> Schema Registry 메뉴 --> View & manage schemas 메뉴 --> Add schema를 누르고 Avro 양식으로 스키마 등록

### STEP 4) GCS sink Connector 생성

confluent 웹콘솔 --> Environments 메뉴애서 원하는 환경접속 --> Clusters 메뉴 --> 원하는 클러스터 접속 --> 좌측 메뉴에 Data integration --> Connectors에서 Add Connector를 누르고 GCS connector를 생성해준다.

confluent 클러스터 연결설정, GCP credential json 파일을 업로드 및 destination bucket 설정을 하여 생성을 해준다.


### STEP 5) 간이 avro producer 작성

```console
$ vim avro_producer.py
#!/usr/bin/env python

import sys
from random import choice
from argparse import ArgumentParser, FileType
from configparser import ConfigParser
from confluent_kafka import Producer
from confluent_kafka import avro
from confluent_kafka.avro import AvroProducer

if __name__ == '__main__':

    def delivery_report(err, msg):
        """ Called once for each message produced to indicate delivery result.
            Triggered by poll() or flush(). """
        if err is not None:
            print('Message delivery failed: {}'.format(err))
        else:
            print('Message delivered to {} [{}]'.format(msg.topic(), msg.partition()))

    value_schema_str = """
    {"namespace": "student.avro",
        "type": "record",
        "doc": "This is an example of Avro.",
        "name": "Student",
        "fields": [
            {"name": "name", "type": ["null", "string"], "default": null, "doc": "Name of the student"},
            {"name": "class", "type": "int", "default": 1, "doc": "Class of the student"}
        ]
    }
    """
    
    value_schema = avro.loads(value_schema_str)

    avroProducer = AvroProducer({
        'bootstrap.servers':'xxx-xxxx.asia-northeast3.gcp.confluent.cloud:9092',
        'security.protocol':'SASL_SSL',
        'sasl.mechanisms':'PLAIN',
        'sasl.username':'{cluster api key}',
        'sasl.password':'{cluster api secret}',
        'schema.registry.url':'https:/xxxxx-xxxx.australia-southeast1.gcp.confluent.cloud',
        'schema.registry.basic.auth.credentials.source': 'USER_INFO',
        'schema.registry.basic.auth.user.info': '{schema registry api key}:{schema registry api secret}',
        'on_delivery':delivery_report,
    }, default_value_schema=value_schema)

    for class_num in range(0,10000):
        value = {"name": "Peter", "class": class_num} # 전송할 메시지
        avroProducer.produce(topic='minsupark_test', value=value)
    
    avroProducer.flush()
```

### STEP 6) 메세지 프로듀싱 실시

```console
$ chmod +x avro_producer.py

$ python avro_producer.py
Message delivered to minsupark_test [0]
Message delivered to minsupark_test [0]
Message delivered to minsupark_test [0]
Message delivered to minsupark_test [0]
Message delivered to minsupark_test [0]

...

Message delivered to minsupark_test [0]
Message delivered to minsupark_test [0]
Message delivered to minsupark_test [0]
Message delivered to minsupark_test [0]
Message delivered to minsupark_test [0]
```

### STEP 7) GCS sink connector를 통해서 메세지가 avro 파일로 약속된 GCS 버킷에 저장되는지 확인

```console
$ gsutil ls gs://minsu_park_kafka_test/topics/minsupark_test/year=2022/month=07/day=12/hour=00/
gs://minsu_park_kafka_test/topics/minsupark_test/year=2022/month=07/day=12/hour=00/minsupark_test+0+0000000008.avro
gs://minsu_park_kafka_test/topics/minsupark_test/year=2022/month=07/day=12/hour=00/minsupark_test+0+0000001008.avro
gs://minsu_park_kafka_test/topics/minsupark_test/year=2022/month=07/day=12/hour=00/minsupark_test+0+0000002008.avro
gs://minsu_park_kafka_test/topics/minsupark_test/year=2022/month=07/day=12/hour=00/minsupark_test+0+0000003008.avro
gs://minsu_park_kafka_test/topics/minsupark_test/year=2022/month=07/day=12/hour=00/minsupark_test+0+0000004008.avro
gs://minsu_park_kafka_test/topics/minsupark_test/year=2022/month=07/day=12/hour=00/minsupark_test+0+0000005008.avro
gs://minsu_park_kafka_test/topics/minsupark_test/year=2022/month=07/day=12/hour=00/minsupark_test+0+0000006008.avro
gs://minsu_park_kafka_test/topics/minsupark_test/year=2022/month=07/day=12/hour=00/minsupark_test+0+0000007008.avro
gs://minsu_park_kafka_test/topics/minsupark_test/year=2022/month=07/day=12/hour=00/minsupark_test+0+0000008008.avro
gs://minsu_park_kafka_test/topics/minsupark_test/year=2022/month=07/day=12/hour=00/minsupark_test+0+0000009008.avro
gs://minsu_park_kafka_test/topics/minsupark_test/year=2022/month=07/day=12/hour=00/minsupark_test+0+0000010008.avro
gs://minsu_park_kafka_test/topics/minsupark_test/year=2022/month=07/day=12/hour=00/minsupark_test+0+0000011008.avro
gs://minsu_park_kafka_test/topics/minsupark_test/year=2022/month=07/day=12/hour=00/minsupark_test+0+0000012008.avro
gs://minsu_park_kafka_test/topics/minsupark_test/year=2022/month=07/day=12/hour=00/minsupark_test+0+0000013008.avro
gs://minsu_park_kafka_test/topics/minsupark_test/year=2022/month=07/day=12/hour=00/minsupark_test+0+0000014008.avro
gs://minsu_park_kafka_test/topics/minsupark_test/year=2022/month=07/day=12/hour=00/minsupark_test+0+0000015008.avro
gs://minsu_park_kafka_test/topics/minsupark_test/year=2022/month=07/day=12/hour=00/minsupark_test+0+0000016008.avro
gs://minsu_park_kafka_test/topics/minsupark_test/year=2022/month=07/day=12/hour=00/minsupark_test+0+0000017008.avro
gs://minsu_park_kafka_test/topics/minsupark_test/year=2022/month=07/day=12/hour=00/minsupark_test+0+0000018008.avro
```


### ** avro 형태가 아닌 일반적인 메세지일때 간이 컨슈머&프로듀서 예시

```console
# config 파일 작성
$ vim getting_started.ini
[default]
bootstrap.servers=xxx-xxxx.asia-northeast3.gcp.confluent.cloud:9092
security.protocol=SASL_SSL
sasl.mechanisms=PLAIN
sasl.username=xxxxxxxxxxxxxxxxxxxxxxx
sasl.password=yyyyyyyyyyyyyyyyyyyyyyyyyy

[consumer]
group.id=python_example_group_1

# 'auto.offset.reset=earliest' to start reading from the beginning of
# the topic if no committed offsets exist.
auto.offset.reset=earliest

# 간이 프로듀서 작성
$ vim producer.py
#!/usr/bin/env python

import sys
from random import choice
from argparse import ArgumentParser, FileType
from configparser import ConfigParser
from confluent_kafka import Producer

if __name__ == '__main__':
    # Parse the command line.
    parser = ArgumentParser()
    parser.add_argument('config_file', type=FileType('r'))
    args = parser.parse_args()

    # Parse the configuration.
    # See https://github.com/edenhill/librdkafka/blob/master/CONFIGURATION.md
    config_parser = ConfigParser()
    config_parser.read_file(args.config_file)
    config = dict(config_parser['default'])

    # Create Producer instance
    producer = Producer(config)

    # Optional per-message delivery callback (triggered by poll() or flush())
    # when a message has been successfully delivered or permanently
    # failed delivery (after retries).
    def delivery_callback(err, msg):
        if err:
            print('ERROR: Message failed delivery: {}'.format(err))
        else:
            print("Produced event to topic {topic}: key = {key:12} value = {value:12}".format(
                topic=msg.topic(), key=msg.key().decode('utf-8'), value=msg.value().decode('utf-8')))

    # Produce data by selecting random values from these lists.
    topic = "minsupark_test"
    user_ids = ['eabara', 'jsmith', 'sgarcia', 'jbernard', 'htanaka', 'awalther']
    products = ['book', 'alarm clock', 't-shirts', 'gift card', 'batteries']

    count = 0
    for _ in range(10):

        user_id = choice(user_ids)
        product = choice(products)
        producer.produce(topic, product, user_id, callback=delivery_callback)
        count += 1

    # Block until the messages are sent.
    producer.poll(10000)
    producer.flush()

$ chmod u+x producer.py

# 프로듀싱 실시
$ ./producer.py getting_started.ini
Produced event to topic minsupark_test: key = sgarcia      value = alarm clock
Produced event to topic minsupark_test: key = sgarcia      value = batteries
Produced event to topic minsupark_test: key = jsmith       value = book
Produced event to topic minsupark_test: key = htanaka      value = gift card
Produced event to topic minsupark_test: key = sgarcia      value = alarm clock
Produced event to topic minsupark_test: key = htanaka      value = alarm clock
Produced event to topic minsupark_test: key = eabara       value = book
Produced event to topic minsupark_test: key = awalther     value = batteries
Produced event to topic minsupark_test: key = awalther     value = batteries
Produced event to topic minsupark_test: key = eabara       value = gift card

# 간이 컨슈머 생성
$ vim consumer.py
#!/usr/bin/env python

import sys
from argparse import ArgumentParser, FileType
from configparser import ConfigParser
from confluent_kafka import Consumer, OFFSET_BEGINNING

if __name__ == '__main__':
    # Parse the command line.
    parser = ArgumentParser()
    parser.add_argument('config_file', type=FileType('r'))
    parser.add_argument('--reset', action='store_true')
    args = parser.parse_args()

    # Parse the configuration.
    # See https://github.com/edenhill/librdkafka/blob/master/CONFIGURATION.md
    config_parser = ConfigParser()
    config_parser.read_file(args.config_file)
    config = dict(config_parser['default'])
    config.update(config_parser['consumer'])

    # Create Consumer instance
    consumer = Consumer(config)

    # Set up a callback to handle the '--reset' flag.
    def reset_offset(consumer, partitions):
        if args.reset:
            for p in partitions:
                p.offset = OFFSET_BEGINNING
            consumer.assign(partitions)

    # Subscribe to topic
    topic = "minsupark_test"
    consumer.subscribe([topic], on_assign=reset_offset)

    # Poll for new messages from Kafka and print them.
    try:
        while True:
            msg = consumer.poll(1.0)
            if msg is None:
                # Initial message consumption may take up to
                # `session.timeout.ms` for the consumer group to
                # rebalance and start consuming
                print("Waiting...")
            elif msg.error():
                print("ERROR: %s".format(msg.error()))
            else:
                # Extract the (optional) key and value, and print.

                print("Consumed event from topic {topic}: key = {key:12} value = {value:12}".format(
                    topic=msg.topic(), key=msg.key().decode('utf-8'), value=msg.value().decode('utf-8')))
    except KeyboardInterrupt:
        pass
    finally:
        # Leave group and commit final offsets
        consumer.close()

$ chmod u+x consumer.py

# 컨슈밍 실시
$ ./consumer.py getting_started.ini
Consumed event from topic minsupark_test: key = sgarcia      value = alarm clock
Consumed event from topic minsupark_test: key = sgarcia      value = batteries
Consumed event from topic minsupark_test: key = jsmith       value = book
Consumed event from topic minsupark_test: key = htanaka      value = gift card

...

Waiting...
Waiting...
Waiting...
Waiting...
```
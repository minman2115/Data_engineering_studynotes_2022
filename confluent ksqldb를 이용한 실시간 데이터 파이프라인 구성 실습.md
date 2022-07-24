.

Data_Engineering_TIL(20220724)

- 관련 실습자료

Python client를 이용한 Confluent kafka 실습

URL : https://minman2115.github.io/DE_TIL364

confluent 실습자료

URL : https://developer.confluent.io/learn-kafka/ksqldb/intro/

- 실습 아키텍처 (메세지 흐름 아키텍처) 

```text
     ksqldb client     <---------->   confluent ksqldb   <---------->   confluent broker   ---------->   간이 consumer
(docker container in local)                                                                            (python client)
```

- 실습내용

### STEP 1) confluent cli 설치 및 사용자 인증

```console
# https://docs.confluent.io/5.5.4/cli/installing.html
$ curl -sL --http1.1 https://cnfl.io/cli | sh -s -- -l
$ curl -sL --http1.1 https://cnfl.io/cli | sh -s -- 2.22.0
$ curl -sL --http1.1 https://cnfl.io/cli | sh -s -- v2.22.0 -b /Users/minman/confluent/
$ mv bin confluent_cli_bin

$ /Users/minman/confluent/confluent_cli_bin/confluent login
Enter your Confluent Cloud credentials:
Email: minman@test.com
Password: **********
Logged in as "minman@test.com" for organization "xxxxxxxxxx-xxxx-xxx-xxxx-xxxxxxxxxxx" ("minman_test").

# /Users/minman/confluent/confluent_cli_bin/confluent environment use {confluent env id}
$ /Users/minman/confluent/confluent_cli_bin/confluent environment use xxx-yyyy
Now using "xxx-yyyy" as the default (active) environment.

# /Users/minman/confluent/confluent_cli_bin/confluent kafka cluster use {confluent cluster id}
$ /Users/minman/confluent/confluent_cli_bin/confluent kafka cluster use zzz-kkkkk
Set Kafka cluster "zzz-kkkkk" as the active cluster for environment "zzz-kkkkk".

# 컨플루언트 웹 콘솔에서 ksqldb를 생성후에 아래와 같이 명령어를 실행해본다.
$ /Users/minman/confluent/confluent_cli_bin/confluent ksql cluster list
       ID       |         Name          | Topic Prefix |   Kafka    | Storage |                           Endpoint                           | Status | Detailed Processing Log
----------------+-----------------------+--------------+------------+---------+--------------------------------------------------------------+--------+--------------------------
  xxxxxx-yyyyyy | minsupark_test_ksqldb | xxxxxx-yyyyy | kkkk-zzzzz |     125 | https://xxxxxx-yyyyy.asia-northeast3.gcp.confluent.cloud:443 | UP     | true
```

### STEP 2) confluent 웹콘솔 브로커 클러스터 메뉴에서 minsupark_tes2 라는 이름으로 토픽 생성

### STEP 3) confluent 웹콘솔에서 ksqldb 생성

### STEP 4) ksqldb client 생성 및 환경설정

```console
$ cd /Users/minman/confluent

$ vim docker-compose.yml
---
services:
  ksqldb-cli:
    image: confluentinc/ksqldb-cli:0.17.0
    container_name: ksqldb-cli
    entrypoint: /bin/sh
    tty: true

$ docker compose up -d

$ docker ps
CONTAINER ID   IMAGE                            COMMAND     CREATED              STATUS              PORTS     NAMES
xxxxxxxxxxx   confluentinc/ksqldb-cli:0.17.0   "/bin/sh"   About a minute ago   Up About a minute             ksqldb-cli

# ksqldb credential 키 생성
$ /Users/minman/confluent/confluent_cli_bin/confluent api-key create --resource xxxxx-yyyyy
It may take a couple of minutes for the API key to be ready.
Save the API key and secret. The secret is not retrievable later.
+---------+------------------------------------------------------------------+
| API Key | xxxxxxxxxxxxxxxx                                                 |
| Secret  | yyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyy |
+---------+------------------------------------------------------------------+

# ksqldb 클라이언트 실행
# docker exec -it ksqldb-cli ksql -u {ksqldb api key} -p {ksqldb api secret} {ksqldb endpoint}
$ docker exec -it ksqldb-cli ksql -u xxxxxxxxxxxxxxxx -p yyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyy https://xxxxx-xxxxx.asia-northeast3.gcp.confluent.cloud:443
OpenJDK 64-Bit Server VM warning: Option UseConcMarkSweepGC was deprecated in version 9.0 and will likely be removed in a future release.

                  ===========================================
                  =       _              _ ____  ____       =
                  =      | | _____  __ _| |  _ \| __ )      =
                  =      | |/ / __|/ _` | | | | |  _ \      =
                  =      |   <\__ \ (_| | | |_| | |_) |     =
                  =      |_|\_\___/\__, |_|____/|____/      =
                  =                   |_|                   =
                  =  Event Streaming Database purpose-built =
                  =        for stream processing apps       =
                  ===========================================

Copyright 2017-2021 Confluent Inc.

CLI v0.17.0, Server v0.27.1-rc3 located at https://xxxxxx-yyyyy.asia-northeast3.gcp.confluent.cloud:443

WARNING: CLI and server version don't match. This may lead to unexpected errors.
         It is recommended to use a CLI that matches the server version.

Server Status: RUNNING

Having trouble? Type 'help' (case-insensitive) for a rundown of how things work!

ksql>

ksql> show streams;

 Stream Name         | Kafka Topic                 | Key Format | Value Format | Windowed
------------------------------------------------------------------------------------------
 KSQL_PROCESSING_LOG | xxxxxx-ttttt-processing-log | KAFKA      | JSON         | false
------------------------------------------------------------------------------------------
```

### STEP 5) ksqldb client로 데이터 프로듀싱 해보기

```console
# ksqldb를 생성하고 그 ksqldb에서 MINSUPARK_TEST 라는 이름으로 스트림을 생성한다.
# MINSUPARK_TEST 스트림의 토픽은 minsupark_tes2로 설정해준다.
ksql> CREATE STREAM MINSUPARK_TEST (PERSON VARCHAR KEY, LOCATION VARCHAR)
>  WITH (VALUE_FORMAT='JSON', PARTITIONS=1, KAFKA_TOPIC='minsupark_tes2');

 Message
----------------
 Stream created
----------------

ksql> show streams;

 Stream Name         | Kafka Topic                 | Key Format | Value Format | Windowed
------------------------------------------------------------------------------------------
 KSQL_PROCESSING_LOG | xxxxx-tttttt-processing-log | KAFKA      | JSON         | false
 MINSUPARK_TEST      | minsupark_tes2              | KAFKA      | JSON         | false
------------------------------------------------------------------------------------------

# 3개의 데이터 프로듀싱 해보기
ksql> INSERT INTO MINSUPARK_TEST VALUES ('Allison', 'Denver');
>INSERT INTO MINSUPARK_TEST VALUES('Robin', 'Leeds');
>INSERT INTO MINSUPARK_TEST VALUES ('Robin', 'Ilkley');

ksql> SET 'auto.offset.reset' = 'earliest';
Successfully changed local property 'auto.offset.reset' to 'earliest'. Use the UNSET command to revert your change.

# ksqldb에 저장된 데이터 조회하기
ksql> SELECT * FROM MINSUPARK_TEST EMIT CHANGES;
+---------------------------+----------------+
|PERSON                     |LOCATION        |
+---------------------------+----------------+
|Allison                    |Denver          |
|Robin                      |Leeds           |
|Robin                      |Ilkley          |
^CQuery terminated

ksql> 
```

### STEP 6) 간이 consumer로 브로커에 저장된 데이터를 컨슈밍 해보기

터미널을 새로열고 아래와 같이 명령어를 실행한다.

```console
$ cd /Users/minman/confluent/python_client

$ python3 -m venv confluent_env

$ cd confluent_env/bin

$ source activate

$ python --version

# m1 프로세스의 경우 아래의 라이브러리 및 환경설정을 하지 않게 되면 pip install confluent_kafka에 에러가 발생한다.
$ brew install librdkafka

$ C_INCLUDE_PATH=/opt/homebrew/Cellar/librdkafka/1.9.1/include LIBRARY_PATH=/opt/homebrew/Cellar/librdkafka/1.9.1/lib pip install confluent_kafka

$ pip list
Package         Version
--------------- -------
confluent-kafka 1.9.0
pip             22.2
setuptools      58.1.0

$ cd /Users/minman/confluent/python_client

$ vim conf.ini
[default]
bootstrap.servers={confluent broker cluster Bootstrap server}
security.protocol=SASL_SSL
sasl.mechanisms=PLAIN
sasl.username={confluent broker cluster API key}
sasl.password={confluent broker cluster API secret}

[consumer]
group.id=python_example_group_1

# 'auto.offset.reset=earliest' to start reading from the beginning of
# the topic if no committed offsets exist.
auto.offset.reset=earliest

$ vim consumer.py
```

```python
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
    topic = "minsupark_tes2"
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
```

```console
$ chmod +x consumer.py

$ ./consumer.py conf.ini
Waiting...
Consumed event from topic minsupark_tes2: key = Allison      value = {"LOCATION":"Denver"}
Consumed event from topic minsupark_tes2: key = Robin        value = {"LOCATION":"Leeds"}
Consumed event from topic minsupark_tes2: key = Robin        value = {"LOCATION":"Ilkley"}
Waiting...
Waiting...
Waiting...
Waiting...
Waiting...
Waiting...
^C%
```
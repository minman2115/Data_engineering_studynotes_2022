.

Data_Engineering_TIL(20220128)

[참고자료]

패스트캠퍼스 "Kafka 완전 정복, 클러스터 구축부터 MSA 환경 활용까지" 강의내용

URL : https://fastcampus.co.kr/dev_online_kafka

[학습내용]

- 카프카에서 server.properties 설정파일에 대해서 알아보자.

URL : https://github.com/apache/kafka/blob/trunk/config/server.properties

```text
# Licensed to the Apache Software Foundation (ASF) under one or more
# contributor license agreements.  See the NOTICE file distributed with
# this work for additional information regarding copyright ownership.
# The ASF licenses this file to You under the Apache License, Version 2.0
# (the "License"); you may not use this file except in compliance with
# the License.  You may obtain a copy of the License at
#
#    http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

# see kafka.server.KafkaConfig for additional details and defaults

############################# Server Basics #############################

# The id of the broker. This must be set to a unique integer for each broker.
# 브로커 아이디로 각 브로커 인스턴스마다 고유한 값을 가지고 있어야함
broker.id=0

############################# Socket Server Settings #############################

# The address the socket server listens on. It will get the value returned from 
# java.net.InetAddress.getCanonicalHostName() if not configured.
#   FORMAT:
#     listeners = listener_name://host_name:port
#   EXAMPLE:
#     listeners = PLAINTEXT://your.host.name:9092
# listeners는 브로커에서 참조하는 엔드포인트로 이해하면됨
#listeners=PLAINTEXT://:9092

# Hostname and port the broker will advertise to producers and consumers. If not set, 
# it uses the value for "listeners" if configured.  Otherwise, it will use the value
# returned from java.net.InetAddress.getCanonicalHostName().
# advertised.listeners는 컨슈머나 프로듀서가 참조하는 엔드포인트로 이해하면됨
# advertised.listeners를 따로 설정하지 않으면 위에 listeners에서 설정된 기본값이 적용된다.
#advertised.listeners=PLAINTEXT://your.host.name:9092


# 참고사항 : listeners와 advertised.listeners를 나누어 놓은 이유
# 외부 트레픽과 내부 트레픽을 나누어서 구분하기 위함이다.
# 예를들어서 리플리케이션 트레픽은 클라이언트의 트레픽을 방해해서는 안되기 때문에 두가지 트레픽을 나누어서 관리를 하려는 목적이다.
# 그리고 외부트레픽은 프록시나 로드벨런서를 타고 들어오는 반면에 내부트레픽은 성능이나 외부트레픽과의 겹치면 안되기 때문에 
# 직접 브로커와 통신하는 경우가 일반적이다.
# 또한 외부 트레픽과 내부 트레픽은 보안관점에서도 다르게 처리해야하기 때문이다.
# 외부트레픽 같은 경우에는 SSL을 적용할수도 있을테고, 반면에 내부트레픽은 그럴 필요없이 운영할 수 있기 때문이다.



# Maps listener names to security protocols, the default is for them to be the same. See the config documentation for more details
#listener.security.protocol.map=PLAINTEXT:PLAINTEXT,SSL:SSL,SASL_PLAINTEXT:SASL_PLAINTEXT,SASL_SSL:SASL_SSL

# The number of threads that the server uses for receiving requests from the network and sending responses to the network
# 서버가 요청을 받거나 응답을 내보내거나 하는 쓰레드라고 이해하면 됨
num.network.threads=3

# The number of threads that the server uses for processing requests, which may include disk I/O
# 서버가 클라이언트의 요청을 처리할때 그리고 디스크 IO까지 포함한 작업을 처리할때 사용하는 쓰레드로 이해하면 됨
num.io.threads=8

# The send buffer (SO_SNDBUF) used by the socket server
socket.send.buffer.bytes=102400

# The receive buffer (SO_RCVBUF) used by the socket server
socket.receive.buffer.bytes=102400

# The maximum size of a request that the socket server will accept (protection against OOM)
socket.request.max.bytes=104857600


############################# Log Basics #############################

# A comma separated list of directories under which to store log files
# 브로커가 데이터를 저장하는 디렉토리를 설정하는 부분임
# 일반적으로 프로덕션 환경에서는 아래와 같이 tmp 폴더를 사용하지는 않는다.
# 약속된 디렉토리를 지정해서 저장하게 된다.
log.dirs=/tmp/kafka-logs

# The default number of log partitions per topic. More partitions allow greater
# parallelism for consumption, but this will also result in more files across
# the brokers.
num.partitions=1

# The number of threads per data directory to be used for log recovery at startup and flushing at shutdown.
# This value is recommended to be increased for installations with data dirs located in RAID array.
num.recovery.threads.per.data.dir=1

############################# Internal Topic Settings  #############################
# The replication factor for the group metadata internal topics "__consumer_offsets" and "__transaction_state"
# For anything other than development testing, a value greater than 1 is recommended to ensure availability such as 3.
offsets.topic.replication.factor=1
transaction.state.log.replication.factor=1
transaction.state.log.min.isr=1

############################# Log Flush Policy #############################

# 카프카 브로커로 전달된 메세지들은 fsync() 라는 내부함수를 통해서 디스크에 저장된다.
# 아래의 내용은 fsync() 함수를 사용했을때 트레이드 오프에 대한 설명이다.
# Durability : fsync() 실행전에 메모리에 들고 있는 데이터가 만약에 리플리케이션을 사용하지 않는다면 유실될 수 있다
# Latency : flush에 대한 인터벌이 길어서 flush 타이밍에 누적된 메세지들이 많다면 브로커에 부담을 줄수 있다.
# Throughput : flush는 디스크 IO하는 작업이기 때문에 상당히 비싼 작업이다. 그래서 flush 인터벌을 짧게 가져가면
#              성능의 문제를 일으킬수도 있다.


# Messages are immediately written to the filesystem but by default we only fsync() to sync
# the OS cache lazily. The following configurations control the flush of data to disk.
# There are a few important trade-offs here:
#    1. Durability: Unflushed data may be lost if you are not using replication.
#    2. Latency: Very large flush intervals may lead to latency spikes when the flush does occur as there will be a lot of data to flush.
#    3. Throughput: The flush is generally the most expensive operation, and a small flush interval may lead to excessive seeks.
# The settings below allow one to configure the flush policy to flush data after a period of time or
# every N messages (or both). This can be done globally and overridden on a per-topic basis.

# The number of messages to accept before forcing a flush of data to disk
# 데이터를 디스크에 쓰기전에 몇개까지의 메세지를 가지고 있을것이냐를 설정해주는 것
#log.flush.interval.messages=10000

# The maximum amount of time a message can sit in a log before we force a flush
# flush의 인터벌을 시간으로 통제하는 설정값으로 기본값은 1초(1000ms)임
#log.flush.interval.ms=1000

############################# Log Retention Policy #############################

# The following configurations control the disposal of log segments. The policy can
# be set to delete segments after a period of time, or after a given size has accumulated.
# A segment will be deleted whenever *either* of these criteria are met. Deletion always happens
# from the end of the log.

# The minimum age of a log file to be eligible for deletion due to age
log.retention.hours=168

# A size-based retention policy for logs. Segments are pruned from the log unless the remaining
# segments drop below log.retention.bytes. Functions independently of log.retention.hours.
#log.retention.bytes=1073741824

# The maximum size of a log segment file. When this size is reached a new log segment will be created.
log.segment.bytes=1073741824

# The interval at which log segments are checked to see if they can be deleted according
# to the retention policies
# 리텐션 폴리시에 따라 삭제할수 있는지 여부를 확인하기 위해 로그 세그먼트를 확인하는 간격임
# 디폴트 값은 5분임
log.retention.check.interval.ms=300000

############################# Zookeeper #############################

# Zookeeper connection string (see zookeeper docs for details).
# This is a comma separated host:port pairs, each corresponding to a zk
# 카프카가 사용할 주키퍼를 아래와 같이 , 로 지정해주면 된다.
# server. e.g. "127.0.0.1:3000,127.0.0.1:3001,127.0.0.1:3002".
# You can also append an optional chroot string to the urls to specify the
# root directory for all kafka znodes.
zookeeper.connect=localhost:2181

# Timeout in ms for connecting to zookeeper
zookeeper.connection.timeout.ms=18000


############################# Group Coordinator Settings #############################

# The following configuration specifies the time, in milliseconds, that the GroupCoordinator will delay the initial consumer rebalance.
# The rebalance will be further delayed by the value of group.initial.rebalance.delay.ms as new members join the group, up to a maximum of max.poll.interval.ms.
# The default value for this is 3 seconds.
# We override this to 0 here as it makes for a better out-of-the-box experience for development and testing.
# However, in production environments the default value of 3 seconds is more suitable as this will help to avoid unnecessary, and potentially expensive, rebalances during application startup.
group.initial.rebalance.delay.ms=0
```

그 밖에 자주사용하는 설정값들

```text
# 토픽의 오토 creation 기능을 on 할건지에 대한 설정이고 기본값은 True 임
auto.create.topics.enable

# 카프카는 메세지 압축을 지원한다
# gzip, snappy, lz4, zstd 4가지를 지원하고 있다.
# 프로듀서가 메세지를 압축해서 전송하면 브로커가 압축된채로 그대로 디스크에 저장을 하고
# 컨슈머가 pull을 할때도 압축된 형태로 메세지를 가져가게 된다.
# 최종적으로 컨슈머에서 압축이 해제되서 소비되게 된다.
# 메세지를 압축해서 송수신이 가능하기 때문에 밴드위스에서 이점이 있다.
compression.type

# 토픽삭제를 활성화 하는 설정이다.
# 만약에 이 옵션을 False로 주게되면 admin tool이나 shell에서 delete topic 명령을 해도
# 아무 효과가 없다. 기본값은 True 이다.
# 기본값이 True이기 때문에 admin tool이나 shell에서 delete topic 명령이 먹히는 것이다.
delete.topic.enable

# 메세지 페이로드를 브로커에서 허용할 가장 큰 메세지 크기를 지정할 수 있다.
# 만약에 위에 compression.type 설정이 되어 있다면 압축된 바이너리 다시말해서
# after compression 상태로 max.bytes가 체크가 된다.
message.max.bytes

# 리플리카 팔로워가 리더한테 아래에 설정값으로 지정된 시간동안 fetch request를 보내지 않는다면
# 그리고 리더의 로그를 다 소모하지 않았다면 리더는 ISR에서 해당 팔로워를 제거하게 된다.
# 디폴트 값은 30초이다. 
replicas.lag.time.max.ms
```
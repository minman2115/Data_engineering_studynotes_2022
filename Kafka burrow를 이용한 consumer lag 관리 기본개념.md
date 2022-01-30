.

Data_Engineering_TIL(20220130)

[참고자료]

패스트캠퍼스 "Kafka 완전 정복, 클러스터 구축부터 MSA 환경 활용까지" 강의내용

URL : https://fastcampus.co.kr/dev_online_kafka

** "AKHQ에서 카프카 클러스터 컨트롤 해보기" 에 이어서 진행하는 실습내용 입니다.

URL : https://minman2115.github.io/DE_TIL343

[학습내용 - 기본개념]

- consumer lag과 카프카 버로우

프로듀서가 commit한 offset과 컨슈머가 commit한 offset의 차이를 말한다. 컨슈머가 아직 처리하지 못한 프로듀서가 발행한 메세지의 갯수라고 보면된다. 카프카를 운영하면서 consumer lag를 관리하고 대응하는 것은 매우 중요하다. consumer lag를 주기적으로 체크하고 알람을 받기 위해서 카프카 버로우라는 툴을 사용하면 된다.

카프카 버로우는 사용자가 특별하게 Threshold를 지정하지 않아도 자체적으로 컨슈머 그룹과 파티션의 상태를 판정할 수 있는 솔루션이다. 등록된 컨슈머들의 모든 commit된 offset들을 확인하고 최종적으로 컨슈머의 상태가 어떤지를 판정한다. 이 판정된 값을 가져갈수 있도록 HTTP 프로토콜을 지원한다. 이거를 보고 카프카 클러스터의 전체적인 lag를 파악할 수 있다. Email notify나 api 콜 같은 기능도 제공한다.

** 깃헙주소 : https://github.com/linkedin/Burrow

- 카프카 컨슈머의 MaxLag

카프카 컨슈머에는 MaxLag이라는 메트릭이 존재해서 그거를 갖고 lag를 상태를 판단하기는 하지만 현실적으로 이게 진짜 lag 상태인지 판단하기 어려운점이 있다.

현실적으로 MaxLag을 활용할 수 없는 이유 1. MaxLag은 모든 컨슈머에서 모니터링이 되어야 한다. 그래서 MaxLag 메트릭은 모든 컨슈머에서 수집이 필요하다는 니즈가 생긴다. 이러한 메트릭들은 수집이 되고 난 후에 별도로 대조를 하고 각각 해석을 해줘야 한다.

현실적으로 MaxLag을 활용할 수 없는 이유 2. MaxLag은 컨슈머가 살아있을때만 유효하다. 메트릭 자체가 컨슈머에서 넘겨주는 구조이기 때문이다. 따라서 컨슈머가 죽은 이후에는 사용할 수 없다.

현실적으로 MaxLag을 활용할 수 없는 이유 2. MaxLag은 객관적이지 않은 지표다. 컨슈머 자체적으로 리포트하기 때문이다. 그래서 잘못된 값이 리포트 되는 경우가 있다.

현실적으로 MaxLag을 활용할 수 없는 이유 2. MaxLag은 자바 클라이언트에만 제공이 된다.

- 카프카 버로우의 동작방식

** URL : https://github.com/linkedin/Burrow/wiki

Burrow has a modular design that separates out the work needed to multiple subsystems:

Clusters run a Kafka client that periodically updates topic lists and the current HEAD offset (the most recent offset) for every partition.

Consumers fetch information about consumer groups from a repository. This could be a Kafka cluster (consuming the __consumer_offsets topic), or Zookeeper, or some other repository.

The Storage subsystem stores all of this information in Burrow

The Evaluator subsystem retrieves information from the Storage subsystem for a specific consumer group and calculates the status of that group. This follows the consumer lag evaluation rules.

The Notifier subsystem requests status on consumer groups on a configured interval and send out notifications (Email, HTTP, or some other method) for groups that meet configured criteria.

The HTTP Server subsystem provides an API interface to Burrow for fetching information about clusters and consumers. See what HTTP requests are available to use.

버로우는 아래와 같이 여러 멀티 서브시스템들이 있다.

버로우 클러스터는 주기적으로 토픽리스트와 현재 헤드 offset을 업데이트한다.

버로우 컨슈머는 컨슈머 그룹에 대한 정보를 주기적으로 업데이트 한다.

버로우 스토리지는 버로우에서 발생하는 모든정보를 저장한다.

버로우 Evaluator는 특정 컨슈머그룹에 대한 정보들을 가져와서 그 그룹의 상태를 계산한다.

버로우 Notifier는 말그대로 알림을 주게된다.

버로우 HTTP 서버는 API 인터페이스를 제공한다. 버로우가 계산한 정보들을 제공함.

- Consumer Lag Evaluation Rules

** 참고 URL : https://github.com/linkedin/Burrow/wiki/Consumer-Lag-Evaluation-Rules

The status of a consumer group in Burrow is determined based on several rules evaluated against the offsets for each partition the group consumes. By establishing a set of rules that determines what is "good" consumer behavior and what is "bad" behavior, we can check whether or not the consumer is performing well without the need for setting a discrete threshold for the number of messages a consumer is allowed to be behind before alerts go off. By evaluating against every partition the group consumes, we assure that the entire consumer group is healthy, and not just the one or two topics that are being monitored. This is especially important for wildcard consumers, such as Kafka Mirror Maker.

1) 버로우의 evaluation window

The Storage subsystem intervals configuration determines the length of our sliding window. This specifies the number of offsets to store for each partition that a consumer group consumes. The default setting is 10 which, when combined with an offset commit interval (configured on the consumer) of 60 seconds, means we evaluate the status of a consumer group over 10 minutes. This window moves forward with each offset the consumer commits (the oldest offset is removed when the new offset is added).

For each consumer offset, we store the offset itself, the timestamp that the consumer committed it, and the lag at the point Burrow received it. The lag is calculated as difference between the HEAD offset of the broker and the consumer's offset. Because broker offsets are fetched on a fixed interval, it is possible for this to result in a negative number. If this happens, the stored lag value is zero.

2) 버로우의 evaluation rule

rule 1. 윈도우 내에 모든 lag가 0이면 컨슈머의 상태는 'OK'로 판정한다.

rule 2. 컨슈머 offset은 변하지 않는데 lag가 고정되어 있고 증가하면 컨슈머가 에러가 발생했다고 판정한다. 파티션은 STALLED도 마킹된다.

rule 3. 컨슈머의 offset이 증가하고 있지만 lag가 고정되어 있고 증가하고 있으면 컨슈머는 Warning 상태로 판정된다. Warning 상태는 멈춰있는 상태는 아니지만 컨슈머가 느려서 뒤쳐지고 있다는 의미이다.

rule 4. 현재시간과 가장 최근 offset의 시간 차이가 윈도우에서 가장 최근 offset의 시간과 가장 오래된 offset의 시간 차이보다 큰 경우 컨슈머는 Error 상태가 된다. 그리고 파티션은 STOPPED로 마킹이 된다. 하지만 현재 컨슈머 offset과 브로커 offset이 같다면 파티션은 에러로 간주하지 않는다. 이것은 오랜시간 동안 컨슈머가 커밋하지 않는 경우이다.

rule 5. 만약에 lag가 -1이면 이것은 버로우가 각 파티션별로 브로커 offset을 가지고 있지 않은 상황이다. 이런경우는 컨슈머를 'OK' 상태로 간주한다. 

[학습내용 - 실습]

- 카프카 버로우 설치하기

터미널을 하나 더 띄우고 아래와 같이 명령어를 실행하여 설치한다.

```console
[ec2-user@ip-10-10-1-223 ~]$ pwd
/home/ec2-user

# go 언어 설치
[ec2-user@ip-10-10-1-223 ~]$ wget https://storage.googleapis.com/golang/getgo/installer_linux
--2022-01-30 04:19:46--  https://storage.googleapis.com/golang/getgo/installer_linux
Resolving storage.googleapis.com (storage.googleapis.com)... 216.58.220.144, 172.217.175.16, 172.217.175.48, ...
Connecting to storage.googleapis.com (storage.googleapis.com)|216.58.220.144|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 5179246 (4.9M) [application/octet-stream]
Saving to: ‘installer_linux’

100%[================================================================================================================================================================>] 5,179,246   4.62MB/s   in 1.1s

2022-01-30 04:19:47 (4.62 MB/s) - ‘installer_linux’ saved [5179246/5179246]

[ec2-user@ip-10-10-1-223 ~]$ sudo chmod +x ./installer_linux

[ec2-user@ip-10-10-1-223 ~]$ ./installer_linux
Welcome to the Go installer!
Downloading Go version go1.17.6 to /home/ec2-user/.go
This may take a bit of time...
Downloaded!
Setting up GOPATH
GOPATH has been set up!

One more thing! Run `source /home/ec2-user/.bash_profile` to persist the
new environment variables to your current session, or open a
new shell prompt.

[ec2-user@ip-10-10-1-223 ~]$ source ~/.bash_profile

[ec2-user@ip-10-10-1-223 ~]$ go version
go version go1.17.6 linux/amd64

[ec2-user@ip-10-10-1-223 ~]$ rm installer_linux

# 카프카 버로우 소스 다운로드
[ec2-user@ip-10-10-1-223 ~]$ git clone https://github.com/linkedin/Burrow.git
'Burrow'에 복제합니다...
remote: Enumerating objects: 2422, done.
remote: Counting objects: 100% (180/180), done.
remote: Compressing objects: 100% (132/132), done.
remote: Total 2422 (delta 91), reused 98 (delta 48), pack-reused 2242
오브젝트를 받는 중: 100% (2422/2422), 854.92 KiB | 4.97 MiB/s, 완료.
델타를 알아내는 중: 100% (1589/1589), 완료.

[ec2-user@ip-10-10-1-223 ~]$ ll
합계 8
drwxrwxr-x 7 ec2-user ec2-user 4096  1월 30 03:55 Burrow
drwxr-xr-x 7 ec2-user ec2-user   77  6월  5  2021 confluent-6.2.0
drwxrwxr-x 6 ec2-user ec2-user 4096  1월 30 03:38 resources

# Burrow 소스 이동
[ec2-user@ip-10-10-1-223 ~]$ cd Burrow/

[ec2-user@ip-10-10-1-223 Burrow]$ go mod tidy
go: downloading go.uber.org/goleak v1.1.11
go: downloading github.com/BurntSushi/toml v0.3.1
go: downloading github.com/Shopify/toxiproxy/v2 v2.1.6-0.20210914104332-15ea381dcdae
go: downloading github.com/fortytw2/leaktest v1.3.0
go: downloading gopkg.in/alexcesaro/quotedprintable.v3 v3.0.0-20150716171945-2caba252f4dc
go: downloading gopkg.in/check.v1 v1.0.0-20201130134442-10cb98267c6c
go: downloading github.com/benbjohnson/clock v1.1.0
go: downloading github.com/jcmturner/goidentity/v6 v6.0.1
go: downloading github.com/google/go-cmp v0.5.6
go: downloading github.com/frankban/quicktest v1.11.3
go: downloading github.com/kr/pretty v0.2.1
go: downloading golang.org/x/xerrors v0.0.0-20200804184101-5ec99f83aff1
go: downloading github.com/kr/text v0.2.0

# go를 이용해서 버로우 바이너리 설치
[ec2-user@ip-10-10-1-223 Burrow]$ go install

[ec2-user@ip-10-10-1-223 Burrow]$ cd /home/ec2-user/go/bin

# 버로우 바이너리가 설치된 것을 확인할 수 있음
[ec2-user@ip-10-10-1-223 bin]$ ll
합계 21152
-rwxrwxr-x 1 ec2-user ec2-user 21658929  1월 30 04:22 Burrow

[ec2-user@ip-10-10-1-223 bin]$ cd /home/ec2-user/resources

# burrow.toml 파일 확인
# burrow.toml은 버로우의 config 파일로 이해하면 됨
# 아래와 같이 config file을 수정해준다.
[ec2-user@ip-10-10-1-223 resources]$ vim burrow.toml
[general]
pidfile="burrow.pid"
stdout-logfile="burrow.out"

[logging]
filename="logs/burrow.log"
level="info"
maxsize=10
maxbackups=3
use-compression=true

[zookeeper]
servers=["localhost:12181","localhost:22181","localhost:32181"]

[cluster.minman]   <-- cluser.클러스터네임
class-name="kafka"
servers=["localhost:19093","localhost:29093","localhost:39093"]
topic-refresh=60
offset-refresh=10

[consumer.minman]
class-name="kafka"
cluster="minman"
servers=["localhost:19093","localhost:29093","localhost:39093"]
group-denylist=""
group-allowlist=""

[httpserver.listener]
address=":8000"
timeout=300

[storage.inmemory]   <-- 슬라이딩 윈도우를 저장할 스토리지 지정
class-name="inmemory"
min-distance=1
workers=20
intervals=10
expire-group=604800

# 버로우 임시 컨피그 폴더 생성
[ec2-user@ip-10-10-1-223 ~]$ mkdir -p /home/ec2-user/tmp/burrow_config/

# config 파일 복사
[ec2-user@ip-10-10-1-223 ~]$ cp /home/ec2-user/resources/burrow.toml /home/ec2-user/tmp/burrow_config/burrow.toml

# 버로우 실행하기
[ec2-user@ip-10-10-1-223 ~]$ /home/ec2-user/go/bin/Burrow --config-dir /home/ec2-user/tmp/burrow_config/
Reading configuration from /home/ec2-user/tmp/burrow_config/


# 웹브라우저에서 {ec2_public_ip}:8000/burrow/admin 로 접속했을때 GOOD 이라는 화면이 잘 뜨는지도 체크해본다.
```

- 버로우로 테스트해보기

```console
# 터미널을 하나 열고 거기에서 아래와 같이 명령어를 실행한다.
# burrowtest라는 토픽을 하나 만들어보자.
[ec2-user@ip-10-10-1-223 ~]$ cd /home/ec2-user/confluent-6.2.0/bin

[ec2-user@ip-10-10-1-223 bin]$ ./kafka-topics --bootstrap-server localhost:19093 --create --topic burrowtest --partitions 3 --replication-factor 3
Created topic burrowtest.

# burrowtest라는 토픽을 구독할 컨슈머 2개를 띄워보자.
[ec2-user@ip-10-10-1-223 bin]$ ./kafka-console-consumer --bootstrap-server localhost:19093 --topic burrowtest --group burrowgroup

# 터미널을 하나 더 열고 거기에서 똑같이 컨슈머를 하나 띄운다.
[ec2-user@ip-10-10-1-223 bin]$ ./kafka-console-consumer --bootstrap-server localhost:19093 --topic burrowtest --group burrowgroup

# 터미널을 다시 하나 더 열고 거기에서는 프로듀서를 하나 띄워본다.
# 아래와 같이 메세지들을 날리면 위에 컨슈머 두개에 메세지들이 분배되어 컨슈밍이 된다.
[ec2-user@ip-10-10-1-223 bin]$ ./kafka-console-producer --bootstrap-server localhost:19093 --topic burrowtest
>1
>2
>3
>4
>5
>6
>7
>8
>9
>10
>11
>12
>13
>14
>15
>16
```

그러면 burrowtest의 lag를 확인해보자.

https://github.com/linkedin/Burrow/wiki/HTTP-Endpoint 여기에 가면 HTTP 서버의 각종 엔드포인트를 확인해볼 수 있다.

예를 들어서 웹브라우저에서 {ec2_public_ip}:8000/v3/kafka/minman 로 접속하면 우리가 만든 minman 클러스터의 정보를 아래와 같이 제이슨 형태로 확인할 수 있다.

```json
{"error":false,"message":"cluster module detail returned","module":{"class-name":"kafka","servers":["localhost:19093","localhost:29093","localhost:39093"],"client-profile":{"name":"","client-id":"burrow-lagchecker","kafka-version":"2.8.0","tls":null,"sasl":null},"topic-refresh":60,"offset-refresh":10},"request":{"url":"/v3/kafka/minman","host":"ip-10-10-1-223.ap-northeast-2.compute.internal"}}
```


이번에는 웹브라우저에서 아까 만든 burrowgroup의 상태를 한번 체크해보자 {ec2_public_ip}:8000/v3/kafka/minman/consumer/burrowgroup/status 로 접속하면 아래와 같이 정보를 확인할 수 있다.

```json
{"error":false,"message":"consumer status returned","status":{"cluster":"minman","group":"burrowgroup","status":"OK","complete":1,"partitions":[],"partition_count":3,"maxlag":{"topic":"burrowtest","partition":0,"owner":"/172.18.0.1","client_id":"consumer-burrowgroup-1","status":"OK","start":{"offset":10,"timestamp":1643522227149,"observedAt":1643522227000,"lag":0},"end":{"offset":10,"timestamp":1643522272151,"observedAt":1643522272000,"lag":0},"current_lag":0,"complete":1},"totallag":0},"request":{"url":"/v3/kafka/minman/consumer/burrowgroup/status","host":"ip-10-10-1-223.ap-northeast-2.compute.internal"}}
```

이번에는 웹브라우저에서 아까 만든 burrowgroup의 lag를 한번 체크해보자 {ec2_public_ip}:8000/v3/kafka/minman/consumer/burrowgroup/lag 로 접속하면 아래와 같이 정보를 확인할 수 있다.

```json
{"error":false,"message":"consumer status returned","status":{"cluster":"minman","group":"burrowgroup","status":"OK","complete":1,"partitions":[{"topic":"burrowtest","partition":0,"owner":"/172.18.0.1","client_id":"consumer-burrowgroup-1","status":"OK","start":{"offset":10,"timestamp":1643522402155,"observedAt":1643522402000,"lag":0},"end":{"offset":10,"timestamp":1643522447156,"observedAt":1643522447000,"lag":0},"current_lag":0,"complete":1},{"topic":"burrowtest","partition":1,"owner":"/172.18.0.1","client_id":"consumer-burrowgroup-1","status":"OK","start":{"offset":6,"timestamp":1643522402155,"observedAt":1643522402000,"lag":0},"end":{"offset":6,"timestamp":1643522447156,"observedAt":1643522447000,"lag":0},"current_lag":0,"complete":1},{"topic":"burrowtest","partition":2,"owner":"/172.18.0.1","client_id":"consumer-burrowgroup-1","status":"OK","start":{"offset":10,"timestamp":1643522402168,"observedAt":1643522402000,"lag":0},"end":{"offset":10,"timestamp":1643522447168,"observedAt":1643522447000,"lag":0},"current_lag":0,"complete":1}],"partition_count":3,"maxlag":{"topic":"burrowtest","partition":0,"owner":"/172.18.0.1","client_id":"consumer-burrowgroup-1","status":"OK","start":{"offset":10,"timestamp":1643522402155,"observedAt":1643522402000,"lag":0},"end":{"offset":10,"timestamp":1643522447156,"observedAt":1643522447000,"lag":0},"current_lag":0,"complete":1},"totallag":0},"request":{"url":"/v3/kafka/minman/consumer/burrowgroup/lag","host":"ip-10-10-1-223.ap-northeast-2.compute.internal"}}
```

이럴헤 버로우에서 lag 정보를 얻을 수 있는데 이 값들을 잘 가공해서 모니터링 디비에 저장하고 활용할 수 있다.
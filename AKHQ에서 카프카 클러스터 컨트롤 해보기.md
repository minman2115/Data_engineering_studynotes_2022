.

Data_Engineering_TIL(20220129)

[참고자료]

패스트캠퍼스 "Kafka 완전 정복, 클러스터 구축부터 MSA 환경 활용까지" 강의내용

URL : https://fastcampus.co.kr/dev_online_kafka

[학습내용]

- 먼저 아래와 같이 EC2에서 실습환경을 구성하자

```console
$ sudo yum update -y

$ sudo amazon-linux-extras install docker -y

$ sudo service docker start

$ sudo usermod -a -G docker ec2-user

# ec2 로그아웃했다가 재접속

# docker 정상설치 확인
$ docker info

# docker-compose 설치
$ sudo curl -L https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m) -o /usr/local/bin/docker-compose
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   152  100   152    0     0    611      0 --:--:-- --:--:-- --:--:--   610
100   664  100   664    0     0   1302      0 --:--:-- --:--:-- --:--:--  1302
100 23.5M  100 23.5M    0     0  10.0M      0  0:00:02  0:00:02 --:--:-- 18.2M

$ sudo chmod +x /usr/local/bin/docker-compose

$ docker-compose version
Docker Compose version v2.2.3

# 자바 설치
$ sudo yum install java-1.8.0-openjdk.x86_64 -y

$ sudo yum install java-1.8.0-openjdk-devel.x86_64 -y

$ which java
/usr/bin/java

$ readlink -f /usr/bin/java
/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.312.b07-1.amzn2.0.2.x86_64/jre/bin/java

$ sudo vim /etc/profile
# 스크립트 가장 하단에 아래의 내용들 추가
export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk
export PATH=$PATH:$JAVA_HOHE/bin
export CLASSPATH=$JAVA_HOME/jre/lib:$JAVA_HOME/lib/tools.jar

$ source /etc/profile

$ echo $JAVA_HOME
/usr/lib/jvm/java-1.8.0-openjdk

# 깃도 설치해주자.
$ sudo yum install git -y

$ git clone https://github.com/jingene/fastcampus_kafka_handson resources
'resources'에 복제합니다...
remote: Enumerating objects: 86, done.
remote: Counting objects: 100% (86/86), done.
remote: Compressing objects: 100% (80/80), done.
remote: Total 86 (delta 37), reused 12 (delta 2), pack-reused 0
오브젝트를 받는 중: 100% (86/86), 26.90 KiB | 6.72 MiB/s, 완료.
델타를 알아내는 중: 100% (37/37), 완료.

$ cd resources/

$ ls
README.md                           docker-compose-akhq.yml                     docker-compose-confluent-schema-registry-and-rest-proxy.yml  schema_registry_and_rest_proxy_examples.txt
burrow.toml                         docker-compose-confluent-cluster.yml        docker-compose-data-pipeline-application.yml
confluent_kafka_basic_commands.yml  docker-compose-confluent-connect.yml        docker-compose-wordpress.yml
connect_examples.txt                docker-compose-confluent-kafka-and-elk.yml  edm

# confluent kafka 설치
$ cd ~

$ wget https://packages.confluent.io/archive/6.2/confluent-community-6.2.0.tar.gz
--2022-01-29 03:13:43--  https://packages.confluent.io/archive/6.2/confluent-community-6.2.0.tar.gz
Resolving packages.confluent.io (packages.confluent.io)... 54.230.63.118, 54.230.63.75, 54.230.63.110, ...
Connecting to packages.confluent.io (packages.confluent.io)|54.230.63.118|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 356898902 (340M) [application/x-tar]
Saving to: ‘confluent-community-6.2.0.tar.gz’

100%[================================================================================================================================================================>] 356,898,902 38.9MB/s   in 11s

2022-01-29 03:13:55 (30.0 MB/s) - ‘confluent-community-6.2.0.tar.gz’ saved [356898902/356898902]

$ tar -zxvf confluent-community-6.2.0.tar.gz

$ ll
합계 348540
drwxr-xr-x 7 ec2-user ec2-user        77  6월  5  2021 confluent-6.2.0
-rw-rw-r-- 1 ec2-user ec2-user 356898902 12월  9 23:25 confluent-community-6.2.0.tar.gz
drwxrwxr-x 6 ec2-user ec2-user      4096  1월 29 03:03 resources

$ rm confluent-community-6.2.0.tar.gz
```

- 도커 컴포즈로 카프카 클러스터를 구성하고 AKHQ에서 컨트롤 해보기

** AKHQ : https://akhq.io

```console
[ec2-user@ip-10-10-1-35 ~]$ cd /home/ec2-user/resources

# confluent kafka 기본명령어 리마인드
[ec2-user@ip-10-10-1-35 resources]$ cat confluent_kafka_basic_commands.yml
# kafka topic 생성
./kafka-topics --bootstrap-server localhost:19092 --create --topic fastcampus --partitions 20 --replication-factor 3

# kafka에 생성된 토픽 리스트 확인
./kafka-topics --bootstrap-server localhost:19092 --list

# 특정 토픽의 파티션 수, 리플리카 수 등의 상세정보 확인
./kafka-topics --describe --bootstrap-server localhost:19092 --topic fastcampus

# kafka 콘솔 컨슈머 실행
./kafka-console-consumer --bootstrap-server localhost:19092 --topic fastcampus --from-beginning

# kafka 콘솔 프로듀서 실행
./kafka-console-producer --bootstrap-server localhost:19092 --topic fastcampus

# 그런 다음에 아래와 같이 docker yml 파일을 수정해준다.
[ec2-user@ip-10-10-1-35 resources]$ vim docker-compose-akhq.yml
version: '3'
services:
  zookeeper-1:
    hostname: zookeeper1
    image: confluentinc/cp-zookeeper:6.2.0
    environment:
      ZOOKEEPER_SERVER_ID: 1
      ZOOKEEPER_CLIENT_PORT: 12181
      ZOOKEEPER_DATA_DIR: /zookeeper/data
      ZOOKEEPER_SERVERS: zookeeper1:22888:23888;zookeeper2:32888:33888;zookeeper3:42888:43888
    ports:
      - 12181:12181
      - 22888:22888
      - 23888:23888
    volumes:
      - ./zookeeper/data/1:/zookeeper/data

  zookeeper-2:
    hostname: zookeeper2
    image: confluentinc/cp-zookeeper:6.2.0
    environment:
      ZOOKEEPER_SERVER_ID: 2
      ZOOKEEPER_CLIENT_PORT: 22181
      ZOOKEEPER_DATA_DIR: /zookeeper/data
      ZOOKEEPER_SERVERS: zookeeper1:22888:23888;zookeeper2:32888:33888;zookeeper3:42888:43888
    ports:
      - 22181:22181
      - 32888:32888
      - 33888:33888
    volumes:
      - ./zookeeper/data/2:/zookeeper/data

  zookeeper-3:
    hostname: zookeeper3
    image: confluentinc/cp-zookeeper:6.2.0
    environment:
      ZOOKEEPER_SERVER_ID: 3
      ZOOKEEPER_CLIENT_PORT: 32181
      ZOOKEEPER_DATA_DIR: /zookeeper/data
      ZOOKEEPER_SERVERS: zookeeper1:22888:23888;zookeeper2:32888:33888;zookeeper3:42888:43888
    ports:
      - 32181:32181
      - 42888:42888
      - 43888:43888
    volumes:
      - ./zookeeper/data/3:/zookeeper/data

  kafka-1:
    image: confluentinc/cp-kafka:6.2.0
    hostname: kafka1
    depends_on:
      - zookeeper-1
      - zookeeper-2
      - zookeeper-3
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: zookeeper1:12181,zookeeper2:22181,zookeeper3:32181
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka1:19092,PLAINTEXT_HOST://localhost:19093
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT,PLAINTEXT_HOST:PLAINTEXT
      KAFKA_LOG_DIRS: /kafka
    ports:
      - 19092:19092
      - 19093:19093
    volumes:
      - ./kafka/logs/1:/kafka

  kafka-2:
    image: confluentinc/cp-kafka:6.2.0
    hostname: kafka2
    depends_on:
      - zookeeper-1
      - zookeeper-2
      - zookeeper-3
    environment:
      KAFKA_BROKER_ID: 2
      KAFKA_ZOOKEEPER_CONNECT: zookeeper1:12181,zookeeper2:22181,zookeeper3:32181
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka2:29092,PLAINTEXT_HOST://localhost:29093
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT,PLAINTEXT_HOST:PLAINTEXT
      KAFKA_LOG_DIRS: /kafka
    ports:
      - 29092:29092
      - 29093:29093
    volumes:
      - ./kafka/logs/2:/kafka

  kafka-3:
    image: confluentinc/cp-kafka:6.2.0
    hostname: kafka3
    depends_on:
      - zookeeper-1
      - zookeeper-2
      - zookeeper-3
    environment:
      KAFKA_BROKER_ID: 3
      KAFKA_ZOOKEEPER_CONNECT: zookeeper1:12181,zookeeper2:22181,zookeeper3:32181
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka3:39092,PLAINTEXT_HOST://localhost:39093
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT,PLAINTEXT_HOST:PLAINTEXT
      KAFKA_LOG_DIRS: /kafka
    ports:
      - 39092:39092
      - 39093:39093
    volumes:
      - ./kafka/logs/3:/kafka

  akhq:
    image: tchiotludo/akhq:latest
    hostname: akhq
    depends_on:
      - kafka-1
      - kafka-2
      - kafka-3
    environment:
      AKHQ_CONFIGURATION: |
        akhq:
          connections:
            kafka:
              properties:
                bootstrap.servers: kafka1:19092,kafka2:29092,kafka3:39092
    ports:
      - 8080:8080


[ec2-user@ip-10-10-1-35 resources]$ docker-compose -f docker-compose-akhq.yml up

...

# 실행이 정상적으로 완료가 되었으면 터미널을 하나 더 띄워서 거기에서 아래와 같이 docker ps 명령어를 실행해보자.
[ec2-user@ip-10-10-1-35 ~]$ docker ps
CONTAINER ID   IMAGE                             COMMAND                  CREATED          STATUS         PORTS                                                                                                                                                                       NAMES
9c5c7007a53c   tchiotludo/akhq:latest            "docker-entrypoint.s…"   13 minutes ago   Up 2 minutes   0.0.0.0:8080->8080/tcp, :::8080->8080/tcp                                                                                                                                   resources-akhq-1
cc1dc0214e27   confluentinc/cp-kafka:6.2.0       "/etc/confluent/dock…"   13 minutes ago   Up 2 minutes   9092/tcp, 0.0.0.0:19092-19093->19092-19093/tcp, :::19092-19093->19092-19093/tcp                                                                                             resources-kafka-1-1
4c3db4bdd0f1   confluentinc/cp-kafka:6.2.0       "/etc/confluent/dock…"   13 minutes ago   Up 2 minutes   9092/tcp, 0.0.0.0:39092-39093->39092-39093/tcp, :::39092-39093->39092-39093/tcp                                                                                             resources-kafka-3-1
9be0bfe3dcf0   confluentinc/cp-kafka:6.2.0       "/etc/confluent/dock…"   13 minutes ago   Up 2 minutes   9092/tcp, 0.0.0.0:29092-29093->29092-29093/tcp, :::29092-29093->29092-29093/tcp                                                                                             resources-kafka-2-1
d2e797c39056   confluentinc/cp-zookeeper:6.2.0   "/etc/confluent/dock…"   13 minutes ago   Up 2 minutes   2181/tcp, 2888/tcp, 0.0.0.0:32181->32181/tcp, :::32181->32181/tcp, 0.0.0.0:42888->42888/tcp, :::42888->42888/tcp, 3888/tcp, 0.0.0.0:43888->43888/tcp, :::43888->43888/tcp   resources-zookeeper-3-1
a08ac6bb1808   confluentinc/cp-zookeeper:6.2.0   "/etc/confluent/dock…"   13 minutes ago   Up 2 minutes   2181/tcp, 2888/tcp, 0.0.0.0:22181->22181/tcp, :::22181->22181/tcp, 0.0.0.0:32888->32888/tcp, :::32888->32888/tcp, 3888/tcp, 0.0.0.0:33888->33888/tcp, :::33888->33888/tcp   resources-zookeeper-2-1
8595f686219c   confluentinc/cp-zookeeper:6.2.0   "/etc/confluent/dock…"   13 minutes ago   Up 2 minutes   2181/tcp, 2888/tcp, 0.0.0.0:12181->12181/tcp, :::12181->12181/tcp, 0.0.0.0:22888->22888/tcp, :::22888->22888/tcp, 3888/tcp, 0.0.0.0:23888->23888/tcp, :::23888->23888/tcp   resources-zookeeper-1-1


# 만약에 ERROR Disk error while locking directory /kafka 에러가 발생하면 아래와 같이
# 명령어를 실행하고 다시 compose up 명령어를 실행할 것
[ec2-user@ip-10-10-1-35 resources]$ cd /home/ec2-user/resources
[ec2-user@ip-10-10-1-35 resources]$ sudo chown -R ec2-user:ec2-user kafka/
[ec2-user@ip-10-10-1-35 resources]$ sudo chown -R ec2-user:ec2-user zookeeper/



# 그런 다음에 웹브라우저에서 {ec2_public_ip}:8080 으로 접속하면 AKHQ 웹콘솔 화면에 접속하게 된다.
# 웹콘솔 화면에서 토픽과 브로커 노드 현황을 확인할 수 있고, 브로커 서버별로 config도 확인하고 수정할 수 있다.
# 웹콘솔에서 토픽을 생성하고 삭제할 수도 있다.
```

- AKHQ 웹 콘솔에서 토픽 생성해보기

STEP 1) 웹 콘솔 홈화면으로 이동

STEP 2) 우측 하단에 'Create a topic' 단추 클릭

STEP 3) 아래와 같은 정보로 토픽정보를 입력하고 우측 하단에 'Create' 단추 클릭

토픽 이름 : minman

파티션 : 10

리플리케이션 팩터 : 3

Cleanup Policy : delete

리텐션 : 86400000

- 프로듀서에서 생성한 토픽으로 메세지를 보내보자

터미널을 하나 새로 띄우고 거기에서 아래와 같이 명령어를 날려서 메세지를 보내보자.

```console
[ec2-user@ip-10-10-1-35 resources]$ cd /home/ec2-user/confluent-6.2.0/bin

[ec2-user@ip-10-10-1-35 bin]$ ./kafka-console-producer --bootstrap-server localhost:19093 --topic minman
>hello
>minman
>good
>day
>^C   <-- 컨트롤 + c 버튼으로 프로듀서 종료
```

그런 다음에 다시 AKHQ 웹 콘솔 화면의 토픽 화면으로 돌아가서 리프레시를 한번 해주면 Count가 0에서 4로 바뀐것을 확인할 수 있다.

그리고 메세지 사이즈와 마지막 레코드가 들어온 시간까지 확인할 수 있다.

또한 우리가 아까 토픽 만들때 설정했었던 값들도 정상적으로 부여가 되어 있는 것을 확인할 수 있다.

해당 minman 토픽을 더블클릭해서 들어가면 메세지의 값들까지 자세한 정보를 확인할 수 있다.


** 참고사항 : 메세지에 키를 지정해서 프로듀서에서 브로커로 보내게 되면 같은 키끼리 같은 파티션에 저장되게 된다. 만약에 키가 null 이면 디폴트로 파티션에 라운드 로빈형태로 저장되게 된다.


AKHQ에서는 또한 벨류값에 대해서 검색이 가능하다. 예를 들어서 hello라는 벨류값이 들어있는 메세지를 검색하고 싶을때 콘솔에서 검색이 가능하다. 검색을 하게 되면 어떤 파티션에 저장이 되어 있는지도 확인이 가능하다.

- 컨슈머 그룹을 만들어서 컨슈밍을 해보고 AKHQ 웹 콘솔에서 이력을 확인해보자.

터미널을 하나 열고 아래와 같이 명령어를 실행해서 컨슈머 그룹을 만들어서 컨슈밍을 해보자.

```console
[ec2-user@ip-10-10-1-35 bin]$ ./kafka-console-consumer --bootstrap-server localhost:19093 --topic minman --group minman_group --from-beginning
minman
good
hello
day
^CProcessed a total of 4 messages
```

그런 다음에 AKHQ 웹 콘솔에서 minman topic에서 컨슈머 그룹 메뉴로 가면 minman_group 이라는 이름으로 컨슈머 그룹이 하나 생성된 것을 확인할 수 있다.

- 또한 AKHQ 웹 콘솔에 'Live Trail' 이라는 기능이 있어서 콘솔에서 해당 메뉴로 가면 라이브하게 메세지를 모니터링도 할 수 있다. 심지어 웹콘솔에서 프로듀싱 하는 기능도 있다.
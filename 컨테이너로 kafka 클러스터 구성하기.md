.

Data_Engineering_TIL(20220123)

[참고자료]

패스트캠퍼스 "Kafka 완전 정복, 클러스터 구축부터 MSA 환경 활용까지" 강의내용

URL : https://fastcampus.co.kr/dev_online_kafka

[학습내용]

STEP 1) 실습환경 구성

```console
$ sudo yum update -y

$ sudo amazon-linux-extras install docker -y

$ sudo service docker start

$ sudo usermod -a -G docker ec2-user

# ec2 로그아웃했다가 재접속

# docker 정상설치 확인
$ docker info

# 테스트 컨테이너 구동해보기
$ docker run hello-world
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
2db29710123e: Pull complete
Digest: sha256:975f4b14f326b05db86e16de00144f9c12257553bba9484fed41f9b6f2257800
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/

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
```

STEP 2) 테스트 코드 다운로드

```console
# 추가로 깃도 설치해주자.
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
```

STEP 3) docker compose로 간단한 워드프레스 서버 띄워보기

도커 컴포즈는 컨테이너로 여러 서비스를 관리할때 유용한 툴이다.

```console
$ cat docker-compose-wordpress.yml
version: "3"

services:
  mysql_database:
    image: mysql:5.7
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: wordpress_db
      MYSQL_USER: wordpress_user
      MYSQL_PASSWORD: changeme!
    volumes:
      - ./mysql:/var/lib/mysql

  wordpress:
    depends_on:
      - mysql_database
    image: wordpress:5.8.1
    ports:
      - "8080:80"
    environment:
      WORDPRESS_DB_HOST: mysql_database:3306
      WORDPRESS_DB_USER: wordpress_user
      WORDPRESS_DB_PASSWORD: changeme!
      WORDPRESS_DB_NAME: wordpress_db
    volumes:
       - ./wordpress_html:/var/www/html

$ docker-compose -f docker-compose-wordpress.yml up

...

resources-mysql_database-1  | 2022-01-23T03:44:48.933046Z 0 [Note] Server socket created on IP: '::'.
resources-mysql_database-1  | 2022-01-23T03:44:48.934879Z 0 [Warning] Insecure configuration for --pid-file: Location '/var/run/mysqld' in the path is accessible to all OS users. Consider choosing a different directory.
resources-mysql_database-1  | 2022-01-23T03:44:48.943661Z 0 [Note] Event Scheduler: Loaded 0 events
resources-mysql_database-1  | 2022-01-23T03:44:48.943978Z 0 [Note] mysqld: ready for connections.
resources-mysql_database-1  | Version: '5.7.37'  socket: '/var/run/mysqld/mysqld.sock'  port: 3306  MySQL Community Server (GPL)

# 터미널창 하나를 더 띄워서 아래와 같이 컨테이너가 잘 올라왔는지 확인해보자

[ec2-user@ip-10-10-1-30 ~]$ docker ps
CONTAINER ID   IMAGE             COMMAND                  CREATED         STATUS         PORTS                                   NAMES
fd727da457ee   wordpress:5.8.1   "docker-entrypoint.s…"   3 minutes ago   Up 3 minutes   0.0.0.0:8080->80/tcp, :::8080->80/tcp   resources-wordpress-1
091934deed36   mysql:5.7         "docker-entrypoint.s…"   3 minutes ago   Up 3 minutes   3306/tcp, 33060/tcp                     resources-mysql_database-1

# 그런 다음에 웹브라우저에서 {ec2_public_ip}:8080 으로 접속해서 워드프레스에 잘 접속되는지 확인해보자.
# 확인을 다했으면 워드 프레스 도커 컴포즈를 실행한 터미널로 가서 ctrl + c 명령어로 중지를 시켜준다.
```

STEP 4) 컨테이너로 카프카 클러스터 구성하기

```console
# 카프카 클러스터 구성을 위한 docker compose yml 파일을 확인해보자.
# ZOOKEEPER_SERVERS: zookeeper1:22888:23888;zookeeper2:32888:33888;zookeeper3:42888:43888
# 위에 문구에서 22888은 주키퍼 앙상블간에 통신할때 사용하는 포트이고, 23888은 리더 선출할때 사용하는 포트로 이해하면 된다.
# 아래에 ports: 하위에 있는 세개 포트는 뭐냐면 로컬 피시의 포트와 컨테이너 포트를 포워딩 해주는 매핑 정보이다.
# 아래에 depends_on: 하위에 있는거는 무슨의미냐면 컨테이너 간에 디펜던시를 의미한다. 주키퍼 1,2,3번 컨테이너가 전부
# 뜨고 나서 카프카를 컨테이너를 띄우라는 의미로 이해하면 된다.
[ec2-user@ip-10-10-1-30 resources]$ cat docker-compose-confluent-cluster.yml
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
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka1:19092
      KAFKA_LOG_DIRS: /kafka
    ports:
      - 19092:19092
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
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka2:29092
      KAFKA_LOG_DIRS: /kafka
    ports:
      - 29092:29092
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
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka3:39092
      KAFKA_LOG_DIRS: /kafka
    ports:
      - 39092:39092
    volumes:
      - ./kafka/logs/3:/kafka



# 도커 컴포즈로 카프카 클러스터 구동하기
$ docker-compose -f docker-compose-confluent-cluster.yml up

...

# 아마도 ERROR Disk error while locking directory /kafka (kafka.server.LogDirFailureChannel)
# 에러가 발생할 것이다.


# 왜냐하면 카프카 폴더와 주키퍼 폴더가 루트 권한으로 생성되서 아마 권한이 없어서 컨테이너에서 접근이 안될 것이다.
$ pwd
/home/ec2-user/resources

$ ll
합계 60
-rw-rw-r-- 1 ec2-user       ec2-user   93  1월 23 03:30 README.md
-rw-rw-r-- 1 ec2-user       ec2-user  688  1월 23 03:30 burrow.toml
-rw-rw-r-- 1 ec2-user       ec2-user  644  1월 23 03:30 confluent_kafka_basic_commands.yml
-rw-rw-r-- 1 ec2-user       ec2-user 3957  1월 23 03:30 connect_examples.txt
-rw-rw-r-- 1 ec2-user       ec2-user 2964  1월 23 03:30 docker-compose-akhq.yml
-rw-rw-r-- 1 ec2-user       ec2-user 2608  1월 23 04:30 docker-compose-confluent-cluster.yml
-rw-rw-r-- 1 ec2-user       ec2-user 4629  1월 23 03:30 docker-compose-confluent-connect.yml
-rw-rw-r-- 1 ec2-user       ec2-user 4769  1월 23 03:30 docker-compose-confluent-kafka-and-elk.yml
-rw-rw-r-- 1 ec2-user       ec2-user 1707  1월 23 03:30 docker-compose-confluent-schema-registry-and-rest-proxy.yml
-rw-rw-r-- 1 ec2-user       ec2-user  596  1월 23 03:30 docker-compose-data-pipeline-application.yml
-rw-rw-r-- 1 ec2-user       ec2-user  597  1월 23 03:30 docker-compose-wordpress.yml
drwxrwxr-x 2 ec2-user       ec2-user  237  1월 23 03:30 edm
drwxr-xr-x 3 root           root       18  1월 23 04:03 kafka
drwxr-xr-x 6 libstoragemgmt root      334  1월 23 03:52 mysql
-rw-rw-r-- 1 ec2-user       ec2-user 3324  1월 23 03:30 schema_registry_and_rest_proxy_examples.txt
drwxr-xr-x 5             33 tape     4096  1월 23 03:44 wordpress_html
drwxr-xr-x 3 root           root       18  1월 23 04:03 zookeeper

# 그럴때는 아래와 같이 권한을 부여해주자
$ sudo chown -R ec2-user:ec2-user kafka/
$ sudo chown -R ec2-user:ec2-user zookeeper/

# 그런다음에 다시 실행하면 정상적으로 컨테이너들이 뜨는 것을 확인할 수 있다.
$ docker-compose -f docker-compose-confluent-cluster.yml up

...



# 새로 터미널 창을 띄워서 아래와 같이 명령어를 실행해보자.

$ docker ps
CONTAINER ID   IMAGE                             COMMAND                  CREATED          STATUS          PORTS                                                                                                                                                                       NAMES
a6ed0a359153   confluentinc/cp-kafka:6.2.0       "/etc/confluent/dock…"   31 seconds ago   Up 27 seconds   9092/tcp, 0.0.0.0:39092->39092/tcp, :::39092->39092/tcp                                                                                                                     resources-kafka-3-1
29e40ca2323d   confluentinc/cp-kafka:6.2.0       "/etc/confluent/dock…"   31 seconds ago   Up 27 seconds   9092/tcp, 0.0.0.0:29092->29092/tcp, :::29092->29092/tcp                                                                                                                     resources-kafka-2-1
f19286181335   confluentinc/cp-kafka:6.2.0       "/etc/confluent/dock…"   31 seconds ago   Up 27 seconds   9092/tcp, 0.0.0.0:19092->19092/tcp, :::19092->19092/tcp                                                                                                                     resources-kafka-1-1
11f51ff82130   confluentinc/cp-zookeeper:6.2.0   "/etc/confluent/dock…"   31 seconds ago   Up 29 seconds   2181/tcp, 2888/tcp, 0.0.0.0:32181->32181/tcp, :::32181->32181/tcp, 0.0.0.0:42888->42888/tcp, :::42888->42888/tcp, 3888/tcp, 0.0.0.0:43888->43888/tcp, :::43888->43888/tcp   resources-zookeeper-3-1
8b409807ec12   confluentinc/cp-zookeeper:6.2.0   "/etc/confluent/dock…"   31 seconds ago   Up 29 seconds   2181/tcp, 2888/tcp, 0.0.0.0:12181->12181/tcp, :::12181->12181/tcp, 0.0.0.0:22888->22888/tcp, :::22888->22888/tcp, 3888/tcp, 0.0.0.0:23888->23888/tcp, :::23888->23888/tcp   resources-zookeeper-1-1
001ae012e988   confluentinc/cp-zookeeper:6.2.0   "/etc/confluent/dock…"   31 seconds ago   Up 29 seconds   2181/tcp, 2888/tcp, 0.0.0.0:22181->22181/tcp, :::22181->22181/tcp, 0.0.0.0:32888->32888/tcp, :::32888->32888/tcp, 3888/tcp, 0.0.0.0:33888->33888/tcp, :::33888->33888/tcp   resources-zookeeper-2-1
```

STEP 5) Confluent 명령어 툴 다운로드

https://www.confluent.io/previous-versions 로 접속해서 6.2.0 커뮤니티 버전 다운로드 링크를 복사해서 다운로드하자.

```console
$ wget https://packages.confluent.io/archive/6.2/confluent-community-6.2.0.tar.gz
--2022-01-23 04:50:19--  https://packages.confluent.io/archive/6.2/confluent-community-6.2.0.tar.gz
Resolving packages.confluent.io (packages.confluent.io)... 52.84.166.108, 52.84.166.4, 52.84.166.81, ...
Connecting to packages.confluent.io (packages.confluent.io)|52.84.166.108|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 356898902 (340M) [application/x-tar]
Saving to: ‘confluent-community-6.2.0.tar.gz’

100%[================================================================================================================================================================>] 356,898,902  146MB/s   in 2.3s

2022-01-23 04:50:21 (146 MB/s) - ‘confluent-community-6.2.0.tar.gz’ saved [356898902/356898902]

$ ll
합계 348540
-rw-rw-r-- 1 ec2-user ec2-user 356898902 12월  9 23:25 confluent-community-6.2.0.tar.gz
drwxrwxr-x 8 ec2-user ec2-user      4096  1월 23 04:40 resources

$ tar -zxvf confluent-community-6.2.0.tar.gz

$ ll
합계 348540
drwxr-xr-x 7 ec2-user ec2-user        77  6월  5  2021 confluent-6.2.0
-rw-rw-r-- 1 ec2-user ec2-user 356898902 12월  9 23:25 confluent-community-6.2.0.tar.gz
drwxrwxr-x 8 ec2-user ec2-user      4096  1월 23 04:40 resources

$ cd confluent-6.2.0/

$ ls
README  bin  etc  lib  share  src

$ cd bin

# confluent bin 폴더로 이동하면 
$ ll
합계 260
-rwxr-xr-x 1 ec2-user ec2-user  2583  6월  5  2021 connect-distributed
-rwxr-xr-x 1 ec2-user ec2-user  1393  6월  5  2021 connect-mirror-maker
-rwxr-xr-x 1 ec2-user ec2-user  2580  6월  5  2021 connect-standalone
-rwxr-xr-x 1 ec2-user ec2-user   858  6월  5  2021 kafka-acls
-rwxr-xr-x 1 ec2-user ec2-user  1553  6월  5  2021 kafka-avro-console-consumer
-rwxr-xr-x 1 ec2-user ec2-user  1545  6월  5  2021 kafka-avro-console-producer
-rwxr-xr-x 1 ec2-user ec2-user   870  6월  5  2021 kafka-broker-api-versions
-rwxr-xr-x 1 ec2-user ec2-user   857  6월  5  2021 kafka-cluster
-rwxr-xr-x 1 ec2-user ec2-user   861  6월  5  2021 kafka-configs
-rwxr-xr-x 1 ec2-user ec2-user   942  6월  5  2021 kafka-console-consumer
-rwxr-xr-x 1 ec2-user ec2-user   941  6월  5  2021 kafka-console-producer
-rwxr-xr-x 1 ec2-user ec2-user   868  6월  5  2021 kafka-consumer-groups
-rwxr-xr-x 1 ec2-user ec2-user   945  6월  5  2021 kafka-consumer-perf-test
-rwxr-xr-x 1 ec2-user ec2-user   868  6월  5  2021 kafka-delegation-tokens
-rwxr-xr-x 1 ec2-user ec2-user   866  6월  5  2021 kafka-delete-records
-rwxr-xr-x 1 ec2-user ec2-user   863  6월  5  2021 kafka-dump-log
-rwxr-xr-x 1 ec2-user ec2-user   860  6월  5  2021 kafka-features
-rwxr-xr-x 1 ec2-user ec2-user  1576  6월  5  2021 kafka-json-schema-console-consumer
-rwxr-xr-x 1 ec2-user ec2-user  1556  6월  5  2021 kafka-json-schema-console-producer
-rwxr-xr-x 1 ec2-user ec2-user   867  6월  5  2021 kafka-leader-election
-rwxr-xr-x 1 ec2-user ec2-user   860  6월  5  2021 kafka-log-dirs
-rwxr-xr-x 1 ec2-user ec2-user   870  6월  5  2021 kafka-metadata-shell
-rwxr-xr-x 1 ec2-user ec2-user   859  6월  5  2021 kafka-mirror-maker
-rwxr-xr-x 1 ec2-user ec2-user   883  6월  5  2021 kafka-preferred-replica-election
-rwxr-xr-x 1 ec2-user ec2-user   956  6월  5  2021 kafka-producer-perf-test
-rwxr-xr-x 1 ec2-user ec2-user  1578  6월  5  2021 kafka-protobuf-console-consumer
-rwxr-xr-x 1 ec2-user ec2-user  1558  6월  5  2021 kafka-protobuf-console-producer
-rwxr-xr-x 1 ec2-user ec2-user   871  6월  5  2021 kafka-reassign-partitions
-rwxr-xr-x 1 ec2-user ec2-user   871  6월  5  2021 kafka-replica-verification
-rwxr-xr-x 1 ec2-user ec2-user  3500  6월  5  2021 kafka-rest-run-class
-rwxr-xr-x 1 ec2-user ec2-user   668  6월  5  2021 kafka-rest-start
-rwxr-xr-x 1 ec2-user ec2-user   764  6월  5  2021 kafka-rest-stop
-rwxr-xr-x 1 ec2-user ec2-user   956  6월  5  2021 kafka-rest-stop-service
-rwxr-xr-x 1 ec2-user ec2-user 10967  6월  5  2021 kafka-run-class
-rwxr-xr-x 1 ec2-user ec2-user  1867  6월  5  2021 kafka-server-start
-rwxr-xr-x 1 ec2-user ec2-user  1658  6월  5  2021 kafka-server-stop
-rwxr-xr-x 1 ec2-user ec2-user   857  6월  5  2021 kafka-storage
-rwxr-xr-x 1 ec2-user ec2-user   942  6월  5  2021 kafka-streams-application-reset
-rwxr-xr-x 1 ec2-user ec2-user   860  6월  5  2021 kafka-topics
-rwxr-xr-x 1 ec2-user ec2-user   955  6월  5  2021 kafka-verifiable-consumer
-rwxr-xr-x 1 ec2-user ec2-user   955  6월  5  2021 kafka-verifiable-producer
-rwxr-xr-x 1 ec2-user ec2-user   981  6월  5  2021 ksql
-rwxr-xr-x 1 ec2-user ec2-user   500  6월  5  2021 ksql-datagen
-rwxr-xr-x 1 ec2-user ec2-user   798  6월  5  2021 ksql-migrations
-rwxr-xr-x 1 ec2-user ec2-user   957  6월  5  2021 ksql-print-metrics
-rwxr-xr-x 1 ec2-user ec2-user   970  6월  5  2021 ksql-restore-command-topic
-rwxr-xr-x 1 ec2-user ec2-user  5239  6월  5  2021 ksql-run-class
-rwxr-xr-x 1 ec2-user ec2-user   815  6월  5  2021 ksql-server-start
-rwxr-xr-x 1 ec2-user ec2-user   378  6월  5  2021 ksql-server-stop
-rwxr-xr-x 1 ec2-user ec2-user   676  6월  5  2021 ksql-stop
-rwxr-xr-x 1 ec2-user ec2-user   514  6월  5  2021 ksql-test-runner
-rwxr-xr-x 1 ec2-user ec2-user  5912  6월  5  2021 schema-registry-run-class
-rwxr-xr-x 1 ec2-user ec2-user  1144  6월  5  2021 schema-registry-start
-rwxr-xr-x 1 ec2-user ec2-user   848  6월  5  2021 schema-registry-stop
-rwxr-xr-x 1 ec2-user ec2-user   959  6월  5  2021 schema-registry-stop-service
-rwxr-xr-x 1 ec2-user ec2-user  1711  6월  5  2021 trogdor
drwxr-xr-x 2 ec2-user ec2-user  4096  6월  5  2021 windows
-rwxr-xr-x 1 ec2-user ec2-user   864  6월  5  2021 zookeeper-security-migration
-rwxr-xr-x 1 ec2-user ec2-user  1884  6월  5  2021 zookeeper-server-start
-rwxr-xr-x 1 ec2-user ec2-user  1366  6월  5  2021 zookeeper-server-stop
-rwxr-xr-x 1 ec2-user ec2-user  1016  6월  5  2021 zookeeper-shell
```

STEP 6) 각종 카프카 명령어 실행해보기

```console
# 먼저 도커 컴포즈로 카프카를 실행하고 있는 터미널에가서 ctrl + c 명령어로 실행중인 카프카를 종료시킨다.

# 그런 다음에 docker-compose-confluent-cluster.yml를 아래와 같이 수정해준다.
# 로컬 ec2에서 도커 컨테이너의 카프카에 접속을 하기 위해서 카프카 컨테이너들의 각각 로컬호스트의 19093,29093, 39093 포트를 
# 열어주는 설정을 아래와 같이 해줘야 한다. 그렇지 않으면 로컬호스트에서 통신을 할 수가 없다.
$ cat docker-compose-confluent-cluster.yml
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


$ docker-compose -f docker-compose-confluent-cluster.yml up

...



# 새로 터미널 창을 띄워서 아래와 같이 명령어를 실행해보자.

# 그러면 아래와 같이 포트포워딩 경로가 서버별로 하나씩 추가 된 것을 확인할 수 있다.
$ docker ps
CONTAINER ID   IMAGE                             COMMAND                  CREATED              STATUS          PORTS                                                                                                                                                                       NAMES
5fac408853ab   confluentinc/cp-kafka:6.2.0       "/etc/confluent/dock…"   About a minute ago   Up 24 seconds   9092/tcp, 0.0.0.0:19092-19093->19092-19093/tcp, :::19092-19093->19092-19093/tcp                                                                                             resources-kafka-1-1
e0cd03ddfe65   confluentinc/cp-kafka:6.2.0       "/etc/confluent/dock…"   About a minute ago   Up 24 seconds   9092/tcp, 0.0.0.0:29092-29093->29092-29093/tcp, :::29092-29093->29092-29093/tcp                                                                                             resources-kafka-2-1
c74c88749d83   confluentinc/cp-kafka:6.2.0       "/etc/confluent/dock…"   About a minute ago   Up 24 seconds   9092/tcp, 0.0.0.0:39092-39093->39092-39093/tcp, :::39092-39093->39092-39093/tcp                                                                                             resources-kafka-3-1
6543414cc8a3   confluentinc/cp-zookeeper:6.2.0   "/etc/confluent/dock…"   About a minute ago   Up 27 seconds   2181/tcp, 2888/tcp, 0.0.0.0:32181->32181/tcp, :::32181->32181/tcp, 0.0.0.0:42888->42888/tcp, :::42888->42888/tcp, 3888/tcp, 0.0.0.0:43888->43888/tcp, :::43888->43888/tcp   resources-zookeeper-3-1
bcb7f3fd919d   confluentinc/cp-zookeeper:6.2.0   "/etc/confluent/dock…"   About a minute ago   Up 26 seconds   2181/tcp, 2888/tcp, 0.0.0.0:22181->22181/tcp, :::22181->22181/tcp, 0.0.0.0:32888->32888/tcp, :::32888->32888/tcp, 3888/tcp, 0.0.0.0:33888->33888/tcp, :::33888->33888/tcp   resources-zookeeper-2-1
298f1dd8b3ec   confluentinc/cp-zookeeper:6.2.0   "/etc/confluent/dock…"   About a minute ago   Up 26 seconds   2181/tcp, 2888/tcp, 0.0.0.0:12181->12181/tcp, :::12181->12181/tcp, 0.0.0.0:22888->22888/tcp, :::22888->22888/tcp, 3888/tcp, 0.0.0.0:23888->23888/tcp, :::23888->23888/tcp   resources-zookeeper-1-1


# 토픽 생성해보기
# 참고사항 : 한번 늘려놓은 파티션 수는 줄일수 없다.
$ ./kafka-topics --bootstrap-server localhost:19093 --create --topic minman --partitions 2 --replication-factor 3
Created topic minman.

$ pwd
/home/ec2-user/confluent-6.2.0/bin


# 프로듀서와 컨슈머를 띄워보면 아래와 같이 잘 동작할 것이다.


# 프로듀서 띄우기
$ ./kafka-console-producer --bootstrap-server localhost:19093 --topic minman
>1
>2
>3
>hello
>minman
>

# 터미널을 하나 더 띄워서 아래와 같이 컨슈머를 구동해보고 위에 프로듀서에서 메세지를 보내본다. 그러면 아래와 같이 잘 출력될 것이다.
$ ./kafka-console-consumer --bootstrap-server localhost:19093 --topic minman
1
2
3
hello
minman


# 컨슈머와 프로듀서 터미널에서 ctrl + c 로 둘다 중지하고 아래와 같이 카프카 2번서버에 토픽 디스크라이브 명령어를 한번 해보자
$ ./kafka-topics --describe --bootstrap-server localhost:29093 --topic minman
Topic: minman	TopicId: Q8NGEozUSZKBm5j6recXXg	PartitionCount: 2	ReplicationFactor: 3	Configs:
	Topic: minman	Partition: 0	Leader: 1	Replicas: 1,3,2	Isr: 2,3,1
	Topic: minman	Partition: 1	Leader: 2	Replicas: 2,1,3	Isr: 2,1,3


# 2번 서버를 강제로 kill 하고 ISR 변화를 확인해보자.
$ docker kill -s=SIGKILL c79f9a9ea676
c79f9a9ea676

$ docker ps
CONTAINER ID   IMAGE                             COMMAND                  CREATED             STATUS          PORTS                                                                                                                                                                       NAMES
8bc01bcf5d35   confluentinc/cp-kafka:6.2.0       "/etc/confluent/dock…"   20 minutes ago      Up 17 minutes   9092/tcp, 0.0.0.0:39092-39093->39092-39093/tcp, :::39092-39093->39092-39093/tcp                                                                                             resources-kafka-3-1
bd861778ac0c   confluentinc/cp-kafka:6.2.0       "/etc/confluent/dock…"   27 minutes ago      Up 17 minutes   9092/tcp, 0.0.0.0:19092-19093->19092-19093/tcp, :::19092-19093->19092-19093/tcp                                                                                             resources-kafka-1-1
0c2707a915c1   confluentinc/cp-zookeeper:6.2.0   "/etc/confluent/dock…"   About an hour ago   Up 17 minutes   2181/tcp, 2888/tcp, 0.0.0.0:32181->32181/tcp, :::32181->32181/tcp, 0.0.0.0:42888->42888/tcp, :::42888->42888/tcp, 3888/tcp, 0.0.0.0:43888->43888/tcp, :::43888->43888/tcp   resources-zookeeper-3-1
4fe2f7668021   confluentinc/cp-zookeeper:6.2.0   "/etc/confluent/dock…"   About an hour ago   Up 17 minutes   2181/tcp, 2888/tcp, 0.0.0.0:22181->22181/tcp, :::22181->22181/tcp, 0.0.0.0:32888->32888/tcp, :::32888->32888/tcp, 3888/tcp, 0.0.0.0:33888->33888/tcp, :::33888->33888/tcp   resources-zookeeper-2-1
08822bae7e48   confluentinc/cp-zookeeper:6.2.0   "/etc/confluent/dock…"   About an hour ago   Up 17 minutes   2181/tcp, 2888/tcp, 0.0.0.0:12181->12181/tcp, :::12181->12181/tcp, 0.0.0.0:22888->22888/tcp, :::22888->22888/tcp, 3888/tcp, 0.0.0.0:23888->23888/tcp, :::23888->23888/tcp   resources-zookeeper-1-1

# 3번 카프카 서버에서 디스크라이브 토픽을 하니까 아래와 같이 ISR에 2번 서버가 사라진 것을 알 수 있다.
$ ./kafka-topics --describe --bootstrap-server localhost:39093 --topic minman
Topic: minman	TopicId: Q8NGEozUSZKBm5j6recXXg	PartitionCount: 2	ReplicationFactor: 3	Configs:
	Topic: minman	Partition: 0	Leader: 1	Replicas: 1,3,2	Isr: 3,1
	Topic: minman	Partition: 1	Leader: 1	Replicas: 2,1,3	Isr: 1,3
```
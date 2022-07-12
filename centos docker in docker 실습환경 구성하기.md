.

Data_Engineering_TIL(20220712)

맥북 m1 프로세스의 경우 centos7는 도커 run 명령어 실행해 --privileged 옵션을 줘도 적용이 안되는 문제가 있다.

centos 8 이상 도커 컨테이너는 --privileged 옵션을 줘도 적용이 가능하다.


```console
$ docker run -d --privileged --name minman_centos centos /usr/sbin/init

$ docker exec -it minman_centos /bin/bash

# Docker centos8(centos:latest)에서 저장소를 변경시켜줘야함
$ sed -i 's|#baseurl=http://mirror.centos.org|baseurl=http://vault.centos.org|g' /etc/yum.repos.d/CentOS-Linux-*

$ yum install curl git -y

# 도커 설치
$ curl -fsSL https://get.docker.com/ | sh

$ systemctl start docker

$ systemctl status docker

# docker compose 설치
$ curl -L "https://github.com/docker/compose/releases/download/v2.5.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

$ chmod +x /usr/local/bin/docker-compose

$ docker-compose --version
Docker Compose version v2.5.1
```
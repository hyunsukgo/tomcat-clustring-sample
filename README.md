# WAS Container Session Replication Environment

## Introduction
docker-compose를 이용하여 클러스터링 테스트 환경을 구축하는 샘플입니다.    

## Architecture 
2대의 Tomcat 컨테이너를 올리고 앞단에 Nginx로 reverse proxy 합니다.   
2대의 Tomcat은 가장 기본적인 클러스터링 설정을 사용하며 multicast 방식에 의해 세션 공유를 합니다.    

![server structure](https://raw.githubusercontent.com/jistol/docker-compose-nginx-tomcat-clustering-sample/master/img/1.png)     

## Prerequites
로컬에서 웹 서비스 환경을 구성하기 위해 git, docker, docker-compose 가 설치되어야 합니다. 

- git 
- docker-compose 
- docker

## Environment

### Setup AS-IS ( Tomcat session replication )
github에서 webapplication을 구동하기위한 환경을 내려받습니다. 
```$xslt
git clone https://github.com/jingood2/docker-compose-nginx-tomcat-clustering-sample-.git
```
tomcat1/conf/server.xml, tomcat2/conf/server.xml 내용에서 아래 부분 주석을 해제하고 저장합니다. 
```
<!-- server.xml -->
<Cluster className="org.apache.catalina.ha.tcp.SimpleTcpCluster"/>
```

tomcat1/conf/context.xml, tomcat2/conf/context.xml 아래와 같이 주석 처리합니다.  
 ```$xslt
 <!--
 <Manager className="org.redisson.tomcat.RedissonSessionManager"
              configPath="${catalina.base}/conf/redisson.conf"
              readMode="REDIS" updateMode="DEFAULT" broadcastSessionEvents="false"/>
 -->             
 
 ``` 

### Setup Redis Session Manager ( Use Redis for session manager )
session store로 redis를 사용하기 위해 https://github.com/redisson/redisson/tree/master/redisson-tomcat 맨밑에 두개의 파일(redisson-all-3.12.1.jar, redisson-tomcat-8-3.12.1.jar)을 다운받고, tomcat1/lib, tomcat2/lib 에 복사합니다. 

tomcat1/conf/server.xml, tomcat2/conf/server.xml 내용에서 아래 부분을 아래와 같이 주석처리합니다. 
```$xslt
<!-- server.xml -->
<!--
<Cluster className="org.apache.catalina.ha.tcp.SimpleTcpCluster"/>
-->

```
tomcat1/conf/context.xml, tomcat2/conf/context.xml 아래와 같이 주석을 해제합니다. 
```$xslt
<Manager className="org.redisson.tomcat.RedissonSessionManager"
             configPath="${catalina.base}/conf/redisson.conf"
             readMode="REDIS" updateMode="DEFAULT" broadcastSessionEvents="false"/>

```

빌드
---
docker명령어로 일일히 다 올리기 귀찮기 때문에 docker-compose를 이용하여 한방에 올립니다.   
docker-compose에 관한 자세한 사항은 [docker compose doc](https://docs.docker.com/compose/)을 참고하세요.    
```$xslt
$ docker-compose build
```


실행
----
다음과 같이 실행합니다.

```console
$ docker-compose up -d
```

`-d` 옵션을 사용해야 실행후 각 컨테이너 콘솔 화면에서 detach됩니다.   

기본적으로 docker-compose 실행시 이미지를 기존 빌드된 것으로 캐쉬하기 때문에 설정파일이나 배포파일이 바뀔 경우 아래와 같이 실행하여 새로 이미지를 만들도록 합니다.    

```console
$ docker-compose up -d --build
``` 

실행시 아래와 같이 이미지 빌드 로그와 함께 각 컨테이너가 실행되는 것을 볼 수 있습니다.


```console
$ docker-compose up -d --build
Building nginx
Step 1/6 : FROM nginx:latest
 ---> da5939581ac8
Step 2/6 : MAINTAINER jistol <pptwenty@gmail.com>
 ---> Using cache
 ---> 2d9af4961368
Step 3/6 : ARG conf
 ---> Using cache
 ---> ead97bc0569d
Step 4/6 : COPY $conf/nginx.conf /etc/nginx/nginx.conf
 ---> Using cache
 ---> 5ab748ec2a17
Step 5/6 : WORKDIR /usr/local/tomcat/bin
 ---> Using cache
 ---> 3eabdd2a3dd5
Step 6/6 : CMD nginx -g daemon off;
 ---> Using cache
 ---> 7f4f2405e032
Successfully built 7f4f2405e032
Successfully tagged tomcatdocker1_nginx:latest
Building tomcat2
Step 1/9 : FROM tomcat:latest
 ---> 0fbedce2f08c
Step 2/9 : MAINTAINER jistol <pptwenty@gmail.com>
 ---> Using cache
 ---> 13adee263b5d
Step 3/9 : ARG conf
 ---> Using cache
 ---> 57a78bc9e8ce
Step 4/9 : ARG warpath
 ---> Using cache
 ---> 638fca357d24
Step 5/9 : RUN rm -rf /usr/local/tomcat/webapps/*
 ---> Using cache
 ---> 928ef1b94bb2
Step 6/9 : COPY $conf/* /usr/local/tomcat/conf/
 ---> Using cache
 ---> 30f8faae0012
Step 7/9 : COPY $warpath /usr/local/tomcat/webapps/ROOT.war
 ---> 3bd3ddeff2d6
Removing intermediate container 9c0b330ce6f6
Step 8/9 : WORKDIR /usr/local/tomcat/bin
 ---> dc773d206d87
Removing intermediate container 693e8b125384
Step 9/9 : CMD catalina.sh run
 ---> Running in c430115e5460
 ---> 3879504509c0
Removing intermediate container c430115e5460
Successfully built 3879504509c0
Successfully tagged tomcatdocker1_tomcat2:latest
Building tomcat1
Step 1/9 : FROM tomcat:latest
 ---> 0fbedce2f08c
Step 2/9 : MAINTAINER jistol <pptwenty@gmail.com>
 ---> Using cache
 ---> 13adee263b5d
Step 3/9 : ARG conf
 ---> Using cache
 ---> 57a78bc9e8ce
Step 4/9 : ARG warpath
 ---> Using cache
 ---> 638fca357d24
Step 5/9 : RUN rm -rf /usr/local/tomcat/webapps/*
 ---> Using cache
 ---> 26dea2133cee
Step 6/9 : COPY $conf/* /usr/local/tomcat/conf/
 ---> Using cache
 ---> c546b169bf25
Step 7/9 : COPY $warpath /usr/local/tomcat/webapps/ROOT.war
 ---> 4f1edfc42b8d
Removing intermediate container f02a70122c3a
Step 8/9 : WORKDIR /usr/local/tomcat/bin
 ---> 50683e072451
Removing intermediate container e2ca57d27775
Step 9/9 : CMD catalina.sh run
 ---> Running in 8da01a9227c2
 ---> 1e050e465174
Removing intermediate container 8da01a9227c2
Successfully built 1e050e465174
Successfully tagged tomcatdocker1_tomcat1:latest
Creating tomcatdocker1_nginx_1 ... 
Creating tomcatdocker1_tomcat1_1 ... 
Creating tomcatdocker1_tomcat2_1 ... 
Creating tomcatdocker1_nginx_1
Creating tomcatdocker1_tomcat1_1
Creating tomcatdocker1_tomcat1_1 ... done
```

docker 프로세스를 확인해보면 다음과 같이 4개의 컨테이너가 올라간 것을 확인 할 수 있습니다.

```console
$ docker ps -a
CONTAINER ID        IMAGE                                                   COMMAND                  CREATED             STATUS                      PORTS                            NAMES
b9a060f1e47e        docker-compose-nginx-tomcat-clustering-sample_tomcat1   "catalina.sh run"        3 days ago          Up 3 days                   8080/tcp                         docker-compose-nginx-tomcat-clustering-sample_tomcat1_1
a51357cd8f91        docker-compose-nginx-tomcat-clustering-sample_tomcat2   "catalina.sh run"        3 days ago          Up 3 days                   8080/tcp                         docker-compose-nginx-tomcat-clustering-sample_tomcat2_1
fd0116beaa8b        redis:4.0.9                                             "docker-entrypoint.s…"   3 days ago          Up 3 days                   0.0.0.0:6379->6379/tcp           docker-compose-nginx-tomcat-clustering-sample_db_1
2288d98fce8c        docker-compose-nginx-tomcat-clustering-sample_nginx     "nginx -g 'daemon of…"   3 days ago          Up 3 days                   80/tcp, 0.0.0.0:8080->8080/tcp   docker-compose-nginx-tomcat-clustering-sample_nginx_1
```

확인
---
![webapp](https://github.com/jingood2/docker-compose-nginx-tomcat-clustering-sample-/blob/master/img/3.png?raw=true)

중지
----
아래 명령어를 통해 중지할 수 있습니다.

```console
$ docker-compose down
```

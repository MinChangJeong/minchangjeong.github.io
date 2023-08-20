---
layout: post
title: "도커를 써야하는 이유"
date: 2023-08-20
excerpt: "환경 세팅하는 귀찮아서 시작한 도커"
tags: [docker, backend]
comments: true
---

## Docker란?

도커 공식 홈페이지에서 설명하는 '도커'는 다음과 같이 설명한다. 

``` Markdown
Docker is an open platform for developing, shipping, and running applications 

- Docker 공식 홈페이지 - 
```

즉, 애플리케이션을 개발하고 배포하며 실행하기 위한 오픈 플랫폼이다. 도커의 가장 큰 장점인 **Docker를 통해 인프라와 애플리케이션을 동일한 방식으로 관리**는 내가 도커를 사용하게된 가장 큰 계기였던 것 같다. 

----

## Docker를 써야하는 이유

1. 인프라와 애플리케이션을 동일한 방식으로 관리 

주로 백엔드 개발을 하다보니, Application, DB 뿐만아니라 Kafka, zookeeper 등 다양한 어플리케이션을 사용할때, 프론트엔드에서 테스트하기 위한 환경을 세팅하는 시간에 많은 시간을 쏟을 때 마다 여간 귀찮은 작업이 아닐 수 없었다. 

도커를 사용하면 필요한 어플리케이션을 이미지로 생성하고 컨테이너로 실행하는 간단한 과정만으로 동일한  환경을 세팅할 수 있다는 장점이 있다. 

주로 나는 각각의 어플리케이션을 위한 Dockerfile을 작성하지 않고 Docker compose를 사용해 이미지를 생성하고 컨테이너를 실행한다. 

```yml

version: '3'

services:
  mysqldb:
    image: mysql
    ports:
      - "3306:3306"
    environment:
      TZ: Asia/Seoul
      MYSQL_DATABASE: wanted
      MYSQL_ROOT_PASSWORD: wanted
      MYSQL_ROOT_HOST: "%"
    volumes:
      - ./backup:/var/lib/mysql
    networks:
      - default
      - app-tier

  backend:
    build:
      context: .
    restart: always
    depends_on:
      - mysqldb
      - kafka
    ports:
      - "8080:8080"
    environment:
      SPRING_DATASOURCE_URL: 
      SPRING_DATASOURCE_USERNAME: 
      SPRING_DATASOURCE_PASSWORD: 
    networks:
      - default
      - app-tier

  zookeeper:
    image: confluentinc/cp-zookeeper
    container_name: zookeeper
    environment:
      - ZOOKEEPER_CLIENT_PORT=2181
    ports:
      - 2181:2181
    networks:
      - app-tier

  kafka:
    image: confluentinc/cp-kafka
    ports:
      - 9092:9092
    container_name: kafka
    restart: on-failure
    environment:
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka:9092
      KAFKA_LISTENERS: PLAINTEXT://0.0.0.0:9092
      KAFKA_ADVERTISED_HOST_NAME: kafka
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_CREATE_TOPICS: dresses:1:1,ratings:1:1
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    depends_on:
      - zookeeper
    networks:
      - app-tier

  kafdrop:
    image: [image name]
    ports:
      - 9000:9000
    container_name: kafdrop
    restart: on-failure
    environment:
      KAFKA_BROKERCONNECT: kafka:9092
      JVM_OPTS: "-Xms32M -Xmx64M"
      SERVER_SERVLET_CONTEXTPATH: /
    depends_on:
      - kafka
    networks:
      - app-tier
    tty: true

networks:
  app-tier:
    driver: bridge
```

위에 작성된 docker-compose.yml 파일은 실제 프로젝트에서 작성한 docker compose 파일로 mysql, springboot application, zookeeper, kafka, kafdrop 등 다양한 어플리케이션의 이미지를 생성하는 역할을 한다. 

그리고 작성된 위치에서 아래 명령어만 입력해서 실행하면 이미지를 생성하고 컨테이너를 데몬으로 실행할 수 있다. 

```bash
docker compose up --build -d
```

즉, docker를 활용해 동일한 환경을 빠르고 정확히 세팅할 수 있다는 측면이 매우 큰 장점인것 같다. 

2. 빠르고 간편한 배포

간단한 springboot 서비스를 배포하는 방법은 다음과 같이 요약할 수 있다. 

* github에 소스 코드를 커밋한다. 
* 소스코드에는 docker compose.yml 파일을 함께 첨부한다. (사용할 어플리케이션 이미지를 등록하기 위한 파일을 작성한다.)
* aws 인스턴스를 생성하고 ssh 프로토콜을 사용해 원격 접속한다. 
* 다음의 명령어를 순서대로 입력한다. 
    * git clone [git hub repository]
    * [빌드도구가 gradle인 경우] ./gradlew build
    * docker compose up --build -d
* 배포 끝

도커에 조금만 익숙해져도 아래의 배포 과정이 매우 간단하다고 느낄것 같다. 

아직 도커에 대해 모르는 부분이 많지만 앞으로 꾸준히 공부해야겠다. 
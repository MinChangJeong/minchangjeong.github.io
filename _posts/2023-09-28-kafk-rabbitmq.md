---
layout: post
title: "Kafka vs RabbitMq"
date: 2023-09-28
excerpt: "Kafka 와 RabbitMq의 차이를 정리하고 Kafka를 적용한 이유에 대해 정리"
tags: [kafka, rabbitmq]
comments: true
---


## Intro

현재 진행중인 프로젝트에서 실시간 대용량 데이터 처리를 위한 메시지 브로커 선정을 위해 대표적인 메시지 브로커 Kafka와 RabbitMq에 대해 팀원들과 토의 한 결과를 포스팅 하려 한다. 

---

## Kafka의 가장 큰 차이점 RabbitMq

카프카와 RabbitMq의 가장 큰 차이점으 '메시지 순서 보장' 이다. [카프카의 동작방식](https://minchangjeong.github.io/kafka-replication/)이라는 포스팅을 미리 정리해 봤는데 요약하자면 다음과 같다. 

* 카프카 클러스터는 기본적으로 멀티 카프카 브로커로 구성된다. 
* 브로커들은 토픽에 해당되는 파티션을 여러개로 분리 저장한다. 
* 컨슈머는 토픽을 구독하고 파티션에 쌓인 메시지를 전달 받는다. 
* 이때 전달되는 메시지의 순서는 프로듀서가 생산한 메시지의 순서대로 전달되디 않을 수 있다. 

RabbitMq는 카프카와 달리 엔드투엔드 메시지 전달의 우선 순위를 지정하는 범용 메시지 브로커로써 설계되어 있어 메시지의 전송의 순서가 보장되어 있지만 카프카에 비해 처리되는 속도가 늦다는 특징이 있습니다. 

애초에 Kafak는 지속적인 빅 데이터의 실시간 교환을 지원하는 분산 이벤트 스트리밍 플랫폼으로 실시간 데이터를 처리하는 점에서는 RabbitMq에 비해 장점을 갖습니다. 

---

## Kafka 사용 방법

앞서 말했다싶이 현재 진행 중인 프로젝트는 실시간 대용량 데이터 처리를 위한 서버를 설계하는 것이기에 카프카를 쓰는것이 적절하다고 판단했다. 

카프카를 프로젝트에 적용하기 위해 다음과 같은 단계를 거쳤다. 

* Docker 환경 세팅 
    * 카프카 클러스터 및 주키퍼 컨테이너 실행 환경 구축
* 어플리케이션 호나경 세팅
    * 프로듀서/컨슈머 생성
    * 토픽 생성

1. Docker 환경 세팅 

```yml
// docker-compose.yml 

version: '3'

services:
  backend:
    container_name: backend
    build:
      context: .
    restart: always
    depends_on:
      - kafka1
      - kafka2
      - kafka3
    ports:
      - "8080:8080"
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

  kafka1:
    image: confluentinc/cp-kafka
    ports:
      - 9092:9092
    container_name: kafka1
    restart: on-failure
    environment:
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka1:9092
      KAFKA_LISTENERS: PLAINTEXT://0.0.0.0:9092
      KAFKA_ADVERTISED_HOST_NAME: kafka1
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_ALLOW_AUTO_CREATE_TOPICS: 'false'
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    depends_on:
      - zookeeper
    networks:
      - app-tier

  kafka2:
    image: confluentinc/cp-kafka
    ports:
      - 9093:9092
    container_name: kafka2
    restart: on-failure
    environment:
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka2:9092
      KAFKA_LISTENERS: PLAINTEXT://0.0.0.0:9092
      KAFKA_ADVERTISED_HOST_NAME: kafka2
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_ALLOW_AUTO_CREATE_TOPICS: 'false'
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    depends_on:
      - zookeeper
    networks:
      - app-tier

  kafka3:
    image: confluentinc/cp-kafka
    ports:
      - 9094:9092
    container_name: kafka3
    restart: on-failure
    environment:
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka3:9092
      KAFKA_LISTENERS: PLAINTEXT://0.0.0.0:9092
      KAFKA_ADVERTISED_HOST_NAME: kafka3
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_ALLOW_AUTO_CREATE_TOPICS: 'false'
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    depends_on:
      - zookeeper
    networks:
      - app-tier

  kafka-ui:
    image: provectuslabs/kafka-ui
    container_name: kafka-ui
    ports:
      - "8989:8080"
    restart: always
    environment:
      - KAFKA_CLUSTERS_0_NAME=local
      - KAFKA_CLUSTERS_0_BOOTSTRAPSERVERS=kafka1:9092,kafka2:9092,kafka3:9092
      - KAFKA_CLUSTERS_0_ZOOKEEPER=zookeeper:2181
    depends_on:
      - zookeeper
      - kafka1
      - kafka2
      - kafka3
    networks:
      - app-tier

networks:
  app-tier:
    driver: bridge
```

docker compose 파일을 하나하나 뜯어서 보자.

* zookeeper
    * 주키퍼는 분산 어플리케이션 관리를 위한 코디네이션 시스템으로 카프카 클러스터가 안정적인 서비스를 할 수 있도록 분산되어 있는 각 카프카 브로커의 정보를 중아에 집중하고 구성 관리 및 동기화 등의 서비스를 제공한다. 

* kafka1, kafka2, kafka3
    * 카프카 클러스터는 기본적으로 3개의 카프카 브로커로 구성한다. 
    * 각각의 카프카 브로커 컨테이너를 실행함으로써 분산 처리를 구현할 수 있다. 
    * 각각 9091, 9092, 9093번의 포트번호를 할당 받아 실행중이다. 
    * 'KAFKA_LISTENERS: PLAINTEXT://0.0.0.0:9092'에서 PLAINTEXT://0.0.0.0:9092는 텍스트 메시지를 수신받겠다는 의미이다. 

* kafka-ui
    * 카프카 클러스터를 모니터링하기 위한 오픈소스 툴로 토픽이 생성되고 메시지가 컨슈머에 전달되는 과정을 모니터링할 수 있다. 

2. 어플리케이션 환경 세팅

```gradle
dependencies {
    // kafka
    implementation 'org.springframework.kafka:spring-kafka'
    testImplementation 'org.springframework.kafka:spring-kafka-test'
}
```

* 카프카를 위한 종속성을 추가한다. 

```yml
spring:
  kafka:
    consumer:
      bootstrap-servers: kafka1:9092,kafka2:9092,kafka3:9092
      group-id: coin
      auto-offset-reset: earliest
      key-deserializer: org.apache.kafka.common.serialization.StringDeserializer
      value-deserializer: org.springframework.kafka.support.serializer.JsonDeserializer
      properties:
        spring.json.trusted.packages: "*"

    producer:
      bootstrap-servers: kafka1:9092,kafka2:9092,kafka3:9092
      key-serializer: org.apache.kafka.common.serialization.StringSerializer
      value-serializer: org.springframework.kafka.support.serializer.JsonSerializer

    topic:
      upbit: topic-upbit
      bithumb: topic-bithumb
```

* 카프카 컨슈머와 프로듀서에 대해 정의 한다. 
* bootstrap-servers는 카프카 컨슈머와 프로듀서가 참조하는 카프카 브로커에 대한 서버 주소이다. 
* topic-upbit, topic-bithumb이라는 토픽을 생성해서 각 토픽에 코인 거래소별로 데이터를 전송하도록 설계했다. 

```java
import lombok.RequiredArgsConstructor;
import org.springframework.kafka.core.KafkaTemplate;
import org.springframework.kafka.support.KafkaHeaders;
import org.springframework.messaging.Message;
import org.springframework.messaging.support.MessageBuilder;
import org.springframework.stereotype.Service;

import java.util.HashMap;

@Service
@RequiredArgsConstructor
public class KafkaProducer {
    private final KafkaTemplate<String, HashMap<String, Object>> kafkaTemplate;

    public void sendMessage(String topicName, HashMap<String, Object> data){
        Message<HashMap<String, Object>> message = MessageBuilder
                .withPayload(data)
                .setHeader(KafkaHeaders.TOPIC, topicName)
                .build();

        kafkaTemplate.send(message);
    }
}
```

* 카프카 프로듀서 코드로 파라미터로 전달받은 토픽에 따라서 카프카 브로커에 메시지를 전송한다. 

```java
@Service
public class UpbitKafkaConsumer {
    @KafkaListener(topics = "${spring.kafka.topic.upbit}", groupId = "${spring.kafka.consumer.group-id}")
    public void consume(HashMap<String, Object> data){
        System.out.println(data);
    }
}
```

* 토픽에 메시지가 들어오면 해당 그 토픽을 구독하는 컨슈머에서 읽어드린다. 이후 코드에서는 컨슈머가 읽어 드린 메시지를 DB에 세이브 할 예정이다.(이 내용은 코드에 아직 없다.)


앞으로의 포스팅에서 JMeter를 사용해서 카프카를 사용하기 전과 후에 서버 성능을 테스트할 예정이다. 
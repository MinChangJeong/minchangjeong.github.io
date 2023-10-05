---
layout: post
title: "Kafka로 실시간 코인 정보 받아오기"
date: 2023-10-05
excerpt: "SpringBoot + WebSocket + Apache Kafka + kafka UI"
tags: [Spring boot, Kafka, websocket]
comments: true
---

## 개요

Upbit, Bithumb과 같은 코인 거래소에서 제공하는 OpenAPI를 활용하여 실시간 코인 가격 및 거래 데이터를 수집하고 분석하기 위한 프로젝트를 진행중이다. 

---

## 프로젝트 구성도

<img width="855" alt="스크린샷 2023-10-05 오후 12 44 48" src="https://github.com/MinChangJeong/minchangjeong.github.io/assets/65451455/79560ad9-3b1b-4756-8890-8a603c7348b4">

* webSocket을 사용해서 upbit, bithumb의 서버 소켓에 연결한다. springboot 어플리케이션이 클라이언트 소켓으로써 동작한다. 

* 실기간으로 들어오는 코인 정보는 카프카 클러스터에서 처리하고 카프카 컨슈머 그룹에서 구독중인 토픽(토픽은 코인거래소별로 생성됨)에 해당되는 코인 정보를 DB에 save한다. 

* kafka에서 처리하는 동작을 모니터링하기 위해서 Kafka-ui를 사용한다. (UI가 예쁘고 지원하는 기능이 많기 때문이다.)

* mysql, zookeeper, kafka cluster 등 모든 서버는 docker container로 실행한다. 


<img width="1040" alt="스크린샷 2023-10-05 오전 11 10 16" src="https://github.com/MinChangJeong/minchangjeong.github.io/assets/65451455/2ebf1084-2f89-4f6a-a323-c7e44aa92c34">

카프카 클러스터는 다음과 같이 구성한다. 

* 카프카 클러스터 새로운 장애 발생 가능성을 방지하기 위해 3개의 카프카 브로커를 구성함으로써 Replication 구현(카프카 클러스터가 Fail 되는 상황이 아니라면 메시지를 받지 못하는 장애 상황은 발생하기 어려움)

* 프로듀서는 소켓으로부터 받은 코인 데이터를 거래소별로 생성된 토픽에 메시지를 발행한다. 
* 각 토픽을 구독하고 있는 컨슈머는 카프카 브로커에 있는 토픽에 메시지가 들어오면 데이터를 읽고 DB에 저장한다. 
* 카프카에서 발생하는 모든 이벤트는 비동기로 처리되기에 사용자가 어플리케이션에 보내는 다른 요청은 동시에 처리가능하다.

--- 

## 코드 

1. docker-compose.yml

```yml
version: '3'

services:
  backend:
    container_name: backend
    build:
      context: .
    restart: always
    depends_on:
      - mysqldb
      - kafka1
      - kafka2
      - kafka3
    ports:
      - "8080:8080"
    environment:
      SPRING_DATASOURCE_URL: jdbc:mysql://mysqldb:3306/stockcoin?useSSL=false&allowPublicKeyRetrieval=true&serverTimezone=Asia/Seoul&characterEncoding=UTF-8
      SPRING_DATASOURCE_USERNAME: root
      SPRING_DATASOURCE_PASSWORD: stockcoin
    networks:
      - default
      - app-tier

  mysqldb:
    container_name: mysqldb
    image: mysql
    ports:
      - "3306:3306"
    environment:
      TZ: Asia/Seoul
      MYSQL_DATABASE: stockcoin
      MYSQL_ROOT_PASSWORD: stockcoin
      MYSQL_ROOT_HOST: "%"
    volumes:
      - ./backup-mysqldb:/var/lib/mysql
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

* 카프카 클러스터는 미리 언급한 것처럼 3대의 카프카 브로커로 구성한다. 
* 카프카 동작을 모니터링하기 위해 kafka-ui 를 컨테이너로 실행한다. 

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

* 카프카 클러스터에 대한 설정 정보를 등록한다. 
* 각 topic을 구독하는 컨슈머들은 ‘auto-offset-rest: earlist’ 설정됨으로써, 컨슈머가 처음 실행될때 이전에 처리되지 않았던 모든 메시지를 읽도록 함으로써 서버가 중단되더라도 이전에 소켓으로부터 받은 데이터를 누락없이 DB에 저장하도록 구현한다. 

2. kafka producer

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

    /*
      param :
        1. topicName: 거래소별 토픽 정보
        2. data: 소켓으로 부터 전달받은 코인 정보
    */
    public void sendMessage(String topicName, HashMap<String, Object> data){
        Message<HashMap<String, Object>> message = MessageBuilder
                .withPayload(data)
                .setHeader(KafkaHeaders.TOPIC, topicName)
                .build();

        kafkaTemplate.send(message);
    }
}
```

* 카프카 프로듀서는 토픽에 대한 정보와 소켓으로 부터 받은 코인정보를 파라미터로 받고 전달받은 토픽에 코인정보를 발행한다. 


3. kafka consumer(ex. bithumbConsumer)

```java
import com.core.Exchange;
import com.domain.global.CoinManagerIF;
import lombok.RequiredArgsConstructor;
import org.springframework.kafka.annotation.KafkaListener;
import org.springframework.stereotype.Service;

import java.util.HashMap;

@Service
@RequiredArgsConstructor
public class BithumbKafkaConsumer {
    private final CoinManagerIF coinManagerInterface;
    @KafkaListener(topics = "${spring.kafka.topic.bithumb}", groupId = "${spring.kafka.consumer.group-id}")
    public void consume(HashMap<String, Object> data){
        coinManagerInterface.save(Exchange.BITHUMB, data);
    }
}
```

* 카프카 컨슈머는 구독중인 토픽으로 부터 메시지가 들어오면 데이터를 읽고 DB에 저장하는 요청을 보낸다. (coinManagerInterface는 repository에 코인 관련 정보를 save하기 위한 인터페이스 - 해당 프로젝트는 멀티 모듈 프로젝트로 구성되어 있어 카프카관련 모듈에서 JPA 관련 작업을 처리할 수 없기에 Entity 관련 로직을 수행하는 모듈에 처리 요청을 전송한다.)


4. client web socket 생성 및 server socket 연결

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.web.socket.client.WebSocketConnectionManager;
import org.springframework.web.socket.client.standard.StandardWebSocketClient;

@Service
public class UpbitWsClient {
    private final UpbitWsListener upbitWsListener;
    private final String socketPath = "wss://api.upbit.com/websocket/v1";

    @Autowired
    public UpbitWsClient(UpbitWsListener upbitWsListener) {
        this.upbitWsListener = upbitWsListener;
        connect();
    }

    public void connect() {
        StandardWebSocketClient client = new StandardWebSocketClient();
        try {
            client.setTaskExecutor(client.getTaskExecutor());
            WebSocketConnectionManager manager = new WebSocketConnectionManager(
                    client,
                    upbitWsListener,
                    socketPath
            );
            manager.setAutoStartup(true);
            manager.start();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}

```

* Spring framework가 지원하는 소켓 라이브러리를 사용 - StandardWebSocketClient
* 서버 소켓주소와 소켓 연결이 성공적으로 이뤄졌을 경우 이후 작업을 처리할 핸들러 등록 - UpbitWsListener

5. 핸들러 

```java
import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.JsonNode;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.worker.worker.producer.KafkaProducer;
import com.worker.worker.socket.upbit.request.UpbitCodeRequest;
import com.worker.worker.socket.upbit.request.UpbitTicketRequest;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;
import org.springframework.web.socket.CloseStatus;
import org.springframework.web.socket.TextMessage;
import org.springframework.web.socket.WebSocketSession;
import org.springframework.web.socket.handler.TextWebSocketHandler;

import java.util.HashMap;
import java.util.List;
import java.util.UUID;
import java.util.concurrent.CountDownLatch;

@Component
public class UpbitWsListener extends TextWebSocketHandler {

    private final ObjectMapper objectMapper;
    private final CountDownLatch closeLatch = new CountDownLatch(1);
    private final KafkaProducer producer;
    @Value("${spring.kafka.topic.upbit}")
    String topicName;

    @Autowired
    public UpbitWsListener(ObjectMapper objectMapper, KafkaProducer producer) {
        this.objectMapper = objectMapper;
        this.producer = producer;
    }

    @Override
    public void afterConnectionClosed(WebSocketSession session, CloseStatus status) {
        try {
            session.close();
        } catch (Exception e) {
            e.printStackTrace();
        }
        closeLatch.countDown();
    }

    @Override
    public void afterConnectionEstablished(WebSocketSession session) {
        UpbitTicketRequest ticket =
                UpbitTicketRequest.builder()
                        .ticket(UUID.randomUUID().toString())
                        .build();

        UpbitCodeRequest code =
                UpbitCodeRequest.builder()
                        .type("ticker")
                        .codes(List.of("KRW-BTC", "KRW-ETH"))
                        .build();


        try {
            session.sendMessage(new TextMessage(objectMapper.writeValueAsString(List.of(ticket, code))));
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    @Override
    public void handleTextMessage(WebSocketSession session, TextMessage textMessage) throws JsonProcessingException {
        JsonNode jsonNode = objectMapper.readTree(textMessage.getPayload());
        HashMap<String, Object> message = objectMapper.convertValue(jsonNode, HashMap.class);
        producer.sendMessage(topicName, message);
    }
}

```

* afterConnectionClosed 메서드는 소켓 연결이 성공적으로 이뤄졌을 경우 수행하는 메서드로 서버 소켓에 필요한 코인정보를 요청할 Request 객체를 생성하고 WebSocketSession.sendMessage를 사용해 요청을 보낸다. 

* handleTextMessage 메서드는 요청에 대한 응답을 처리하는 메서드로 TextMessage 객체에 응답받은 데이터가 매핑되어 있으며 Json형태의 Response데이터를 JsonNode 클래스를 사용해 Json데이터를 HashMap객체로 변환한다. 

* 이후 카프카 프로듀서에게 소켓으로 부터 받은 코인 정보를 전달한다. 

---

## 결과 

<img width="1086" alt="스크린샷 2023-10-05 오후 1 12 16" src="https://github.com/MinChangJeong/minchangjeong.github.io/assets/65451455/7e01782a-4b7f-4fd8-8535-e48418f0e182">

* 카프카를 통해 처리되는 데이터는 DB에 실시간으로 save된다. 

<img width="1482" alt="스크린샷 2023-10-05 오후 1 14 05" src="https://github.com/MinChangJeong/minchangjeong.github.io/assets/65451455/79210345-3ebd-4a78-beba-cac2a2ccee24">

* 카프카로 동작을 모니터링 할 수 있도록 Kafka-UI 연결(localhost:8989번으로 접속가능)

프로젝트에 대한 모든 코드는 [github](https://github.com/backend-deepdive/backend)에서 확인할 수 있습니다. 
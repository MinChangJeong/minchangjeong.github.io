---
layout: post
title: "Apache Kafka 실습"
date: 2023-08-26
excerpt: "카프카를 직접 로컬에서 실행해보자."
tags: [Apache Kafka, backend]
comments: true
---

모든 내용은 [Apache Kafka 공식 홈페이지](https://kafka.apache.org/quickstart) 에 있으며 카프카의 동작을 조금이나마 이해하기 위해서 raw level에서 카프카를 사용하는 실습을 진행해본다.. (도커 사용 x) 


## Apache Downloads  

[여기서](https://www.apache.org/dyn/closer.cgi?path=/kafka/3.5.0/kafka_2.13-3.5.0.tgz) 제일 위에 있는 .tgz 파일을 다운로드 받아 준다. 

카프카를 테스트할 폴더를 하나 만들고 거기다가 tar 명령어를 사용해 다운로드 받은 zip 파일을 풀어준다. 그리고 풀어준 파일 안으로 cd 해서 들어간다. 

## Start the ZooKeeper And Kafka Broker service
```shell
$ bin/zookeeper-server-start.sh config/zookeeper.properties

$ bin/kafka-server-start.sh config/server.properties
```

zookeeper, kafka broker 실행한다.(여기서는 터미널을 여러개로 띄워서 각각 실행하지만 데몬으로 실행해도 문제 없을것 같다.) 

## 카프카 클러스터와 카프카 브로커의 관계

그 다음 단계를 가기 전에 카프카 클러스터에 대한 언급이 공식문서에도 나와 있기 내용을 정리하고 넘어가려고 한다. 

덧붙여서 ‘카프카 클러스터는 default로 3개의 카프카 브로커로 구성되어 가용성을 제공한다.’ 라는 문장의 의미를 파악것이 핵심이다. 

카프카 클러스터와 카프카 브로커의 관계에 대해 간단히 설명하자면 다음 예시와 같다. 

카프카 클러스터는 마을의 초대형 공간으로, 이 마을은 여러 집과 건물로 이루어져있고 각 집은 중요한 정보나 이야기를 담은 편지들을 받거나 보내는 역할을 한다. (카프카의 프로듀서, 컨슈머의 역할을 비유함)

카프카 브로커(메시지 브로커)는 마을 안의 집마다 있는 우체통으로 사람들은 이 우체통을 통해 편지를 주고 받는다. 

저기 카프카 클러스터라고 보이는 부분에 카프카 브로커 여러대가 모여 있다고 생각하면 될것 같고 특정 카프카 브로커 한대에서 프로듀서가 생성한 토픽에 이벤트나 메시지를 보내면 그 토픽을 구독하고 있는 컨슈머가 토픽에 저장된 이벤트나 메시지를 읽는 형태로 동작한다. 

카프카 클러스터는 default로 3개의 카프카 브로커를 구성하고 있는데 가용성을 확보한다는 측면이라고 하는데 이게 뭔말이냐면 3대중 한대의 브로커가 고장나더라도 나머지 브로커들이 데이터를 계속 처리하고 사용할 수 있도록 장애에 대비하기 위함이라고 한다. Leader-Follower 구조로 동작한다. 

## Kafak Cluster 설정 

클러스터 아이디를 하나 만든다. 이 작업도 작업중인 위치에서 진행한다 

```shell
$ KAFKA_CLUSTER_ID="$(bin/kafka-storage.sh random-uuid)"

$ bin/kafka-storage.sh format -t $KAFKA_CLUSTER_ID -c config/kraft/server.properties
```

## kafka 서버 실행 및 토픽 생성과 프로듀서, 컨슈머 테스트 

```shell
# 카프카 서버 실행
$ bin/kafka-server-start.sh config/kraft/server.properties

# 프로듀셔 - 토픽 생성
$ bin/kafka-topics.sh --create --topic quickstart-events --bootstrap-server localhost:9092
```

아래의 예시처럼 각각 프로듀서, 컨슈머를 테스트할 터미널 두대를 띄우고 메시지를 생성하고 읽는 과정이 잘 되는지 확인한다.  

```shell
# 프로듀서 - 메시지 생성
$ bin/kafka-console-producer.sh --topic quickstart-events --bootstrap-server localhost:9092
> This is my first event
> This is my second event


# 컨슈머 - 메시지를 읽음
$ bin/kafka-console-consumer.sh --topic quickstart-events --from-beginning --bootstrap-server localhost:9092
This is my first event
This is my second event
```


명령어에서 확인할 수 있는 localhost:9092는 '브로커가 로컬 호스트(현재 실행중인 컴퓨터)의 9092포트에서 실행중이라는 의미인데 만약 카프카 브로커가 다른 주소나 포트에서 실행중이라면 해당 정보를 입력해야한다.' 라고 한다. 

**즉 아까 설명했던 카프카 브로커가 복수개 일때 프로듀서와 컨슈머의 작업을 처리할 브로커를 선택하는 명령어라고 생각하면 될것 같다. **

마지막으로 --bootstrap-server는 카프카 클러스터는 여러개의 브로커로 구성되어 있다고 두번 세번 말하지만, 이 브로커들을 연결할 때 사용하는 초기 연결 지점을 부트스트랩 서버라고 부른다고 한다. 

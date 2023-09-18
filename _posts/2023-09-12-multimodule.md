---
layout: post
title: "멀티 모듈"
date: 2023-09-12
excerpt: "우아한테크세미나 멀티모듈 내용 정리"
tags: [multi module, backend]
comments: true
---

## 멀티 모듈

기존 문제 및 해결 과정
* 회원 시스템 (단일 모듈 멀티 프로젝트) -> 사람에게 의존적인 일관성

* 회원 시스템 (단일 모듈 멀티 프로젝트 + 내부 메이븐 저장소) -> 번거로운 개발 사이클
nexus에 올리고 다른 프로젝트에서 받고 개발 프로세스가 너무 복잡함

* **회원 시스템 (멀티 모듈 단일 프로젝트) -> 시스템으로 보장되는 일관성 빠른 개발 사이클**

----

## MSA? 멀티 모듈?

MSA에서 멀티모듈이 부각되는 이유?

역할 의존성의 분리를 통해 시스템의 분리, 통합을 유연하게 만들어 줄 수 있는 좋은 아키텍처를 만들 수 있기 때문이다. 

----

## 실패한 멀티 모듈 프로젝트

1. 스파게티 코드
    * 다양한 비즈니스 코드가 코어 모듈로 모아지기 시작함. 
    * 특정 클래스를 수정하면 서비스 전체에 영향을 주는 일이 발생함. 

2. 의존성 덩어리 
    * 코어에 비즈니스 플로우가 흐르다 보니 비즈니스와 관련된 의존성들이 모두 사용되야 한다는 문제가 발생함. 

3. 공통 설정
    * 코어 모듈안에서 중앙 집중된 설정(모듈간의 차이를 반영하지 못함. - db connection 부족)

* 결론
    * 모듈에 대한 정의가 모호했다!

* 모듈화란? 
    * 어플리케이션 안에서 내가 필요한 의존을 끼워 맞추는 블록화라고 정의

* 무엇을 중심으로 정의 내려야 할까?
    * 역할과 책임 - 모듈화도 역할과 책임이 분명하면 제대로 정의할 수 있지 않을끼?
    * 역할과 책임에 대한 범위는?
        * 계층
            * 계층이 존재하고 역할과 책임이 존재한다. 흐름이 존재한다. 
            * 예시 - 스프링 프레임워크 -> 와닫지 않음
                * 라이브러리(프레임워크)의 목적 - 사용성, 기능 제공
                * 우리가 만드는 어플리케이션의 목적 - 서비스 제공
            * 직접 계층, 역할을 나누어 보자

----

## 멀티 모듈 구성하기 

* 개념 정리 
    * 시스템 - 한 개이상의 서비스와 공유 인프라가 모여 하나의 시스템을 구성한다. 
    * 어플리케이션 비즈니스, 도메인 베즈니스 
        * 서비스의 흐름을 제어 - 어플리케이션 비즈니스
        * 도메인이 생성되고 변경되고 소멸되는 로직 - 도메인 비즈니스

* 레이어 구상 
    * 어플리케이션
    * 내부 모듈 / 도메인 모듈
    * 공통 모듈
    * 독립 모듈

* 레이어 구상 단계
    * 역할이 분명한가? 
        * 아니요 - 어플리케이션 모듈
        * 예 - 어플리케이션 비즈니스를 가지고 있는가? 
            * 예 - 어플리케이션 모듈
            * 아니오 - 도메인 비즈니스를 가지고 있는가? 
                * 예 - 도메인 모듈
                * 아니요 - 시스템을 알고 있는가? 
                    * 예 - 내부 모듈 
                    * 아니요 - 외부 모듈

1. 내부 모듈 
    * 시스템 안에서 의미를 갖는다. 
    * 어플리케이션, 도메인의 비즈니스를 모른다. 

    * 스팩이 중복되는 부분을 분리 할 수 있지 않을까?
    * 외부 시스템 변경에 대한 영향 파악이 어려움. - 외부 통신을 담당하는 모듈 분리 
    
    - client-coduck
        * 환경별 시스템 Host, Header 관리 
        * 요청/응답 스팩 관리 
        * 예외 처리 추상화 수준 통일

    * build.gradle
        ```
        bootJar {enabled = false} // 실행되지 않는 jar
        jar {enabled = true}
        dependencies {
            boot-starter,
            spring-web
        }
        ```
    
    * 무엇을 얻었는가? 
        * 스팩 변경에 대한 단일 변경 포인트 
        * 사용 추적


2. 도메인 모듈 계층
    * 어플리케이션 비즈니스를 모른다. 
    * 하나의 모듈은 최대 하나의 인프라스트럭처에 대한 책임만 갖는다. 
    * 도메인 모듈을 조합한 더 큰 단위의 도메인 모듈이 있을 수 있다. 
    * 멀티 모듈 설계: 역할, 의존성 중심 모듈화 

    * domain-rds
        * infrastructure 연동 관리 
        * 도메인 계층 구현

        * build.gradle 
            ```
            bootJar {enabled = false}
            jar {enabled = true}
            dependencies {
                jpa, 
                querydsl-jpa
                querdsl-apt
                h2
                mysql
                mariadb
            }
            ```

    * 다중 도메인
        인프라스트럭처 연동 관리 
        도에인 계층 구현
    
    * 다중 인프라스트럭처 - 인프라스트럭처를 분리하자   
        * domain-redis
            infrastructure 연동 관리 
            도메인 계층 구현
        * domain-dynamo
            infrastructure 연동 관리 
            도메인 계층 구현
        
    * domain-service-module(domain-redis, domain-dynamo를 의존함)
        * domain-redis
        * domain-dynamo

        * build.gradle
            ```
            bootJar {enabled = false}
            jar {enabled = true}
            dependencies { project :domain-redis, project :domain-dynamo}
            ```

3. 독립 모듈 계층
    * 시스템과 관련 없이 자체로서 독립적인 역할을 갖는다. 

    * data-dynamo-reactive
        * infrastructure 연동 사용성 제공
        * object mapping 기능 지원
        * Reactor 사용성 제공


4. 공통 모듈 계층
    type, util 등을 정의한다. 
    enum class
    순수 자바 클래스만 작성한다고 전제함. 

    * build.gradle
        ```
        bootJar {enabled = false}
        jar {enabled = true}
        dependencies {

        }
        ```

    * root build.gradle
        ```
        cofigure(subprojects.findAll {it.name != 'core'}) {
            dependencies {
                compile project(':core')
            }
        }
        ```
    

* 어플리케이션 모듈 계층
    * app-api
        ```
        dependencies {
            starter-web;
            project :data-service
            project :clients:c-client
            project :clients:d-client
            io-micrometer:micrometer-registry-influx
        }
        ```

    * app-worker(event 기반 처리)
        ```
        dependencies {
            spring-cloud-starter-aws-messaging
            project :data-service
            project :clients:c-client
            project :clients:d-client
            io-micrometer:micrometer-registry-influx
        }
        ```


    * project :client:c-client 
    * project :client:d-client

    이런 것들은 내부 모듈을 여러 모듈로 분할했음을 의미

    * 결과 
    <img width="700" alt="image" src="https://github.com/MinChangJeong/minchangjeong.github.io/assets/65451455/ea211fc6-1d7f-4057-9636-009f26432d30">

---


## 효과

* 명확한 추상화 경계 
    * 각 모듈이 갖는 책임과 역할이 명확하여 리팩토링 기능의 변경의 영향 범위를 파악하기 용이하다. 
    * 경계가 집요함으로써 기능의 제공 정도를 얘측 가능하여 스파게티 발생 가능성 저하
    * 역할과 책임에 대한 애매함이 없어짐으로써 어떤 모듈에서 어느정도까지 개발되어야 할지가 명확해 진다. 

* 최소 의존성
    각 모듈은 역할에 필요한 최소 의존성을 보유함. 
    
    <br/>
    <img width="727" alt="image" src="https://github.com/MinChangJeong/minchangjeong.github.io/assets/65451455/d28ab8d6-8a2f-4ab2-a5fd-7b39e4de99d1">



* 꿀팁(모듈 Scan)

    * before
    ```
    // 패키지 컨밴션
        com
            baemin
                projectName
                    A moduleName
                        AApplication.java
                    B moduleName
                        BApplication.java
    ```


    ```java
        @SpringBootApplication(scanBasePackages = {"com.github.kingbbode.external". "com.github.kingbbode.domain.redis"})
        @EnableJpaRepositories(basePackages ="com.github.kingbbode.domain.rds")
        @EntityScan(basePackages = "com.github.kingbbode.domain.rds")
        @Import({GihubClientConfig.class, CoduckClientConfig.class, EmbeddedReidsConfig, RedisTemplateConfig.class})
        public class ExternalApplication {
            public static void main(String[] args) {
                SpringApplicatino.run(ExternalApplication.class, args);
            }
        }
    ```

    * after
    ```
    // 패키지 컨밴션
        com
            baemin
                projectName
                    A moduleName - component Scan base package
                    B moduleName
                    Application.java
    ```
    Application.java를 모듈들과 같은 위치에 넣는다. 

    ```java
        @SpringBootApplication
        public class ExternalApplication {
            public static void main(String[] args) {
                SpringApplicatino.run(ExternalApplication.class, args);
            }
        }
    ```

    각 모듈의 property 로드
    ```
    resources
        application-{profiles}.yml
    ```       

    application.yml
    ```
    spring:
        profiles:
            include: clienta, clientb, clientc, clientd, dynamo, redis
    ```
    
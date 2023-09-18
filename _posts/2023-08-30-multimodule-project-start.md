---
layout: post
title: "멀티 모듈 프로젝트 설계 - 프로젝트 세팅"
date: 2023-08-30
excerpt: "각 모듈에 대한 역할과 책임을 분명히 하자"
tags: [multi module, backend]
comments: true
---

## 멀티 모듈을 도입한 이유

이제것 개발하던 단일 모듈 프로젝트는 다양한 의존성이 하나의 모듈에 강하게 결합되어 있어 어떤 클래스에서 해당 의존성을 사용하는지 찾기가 어려웠다. 

또한 버그가 생기면 디버깅의 최초 시작지점을 찾는데 시간이 오래 걸렸다. 그러던 중 멀티 모듈이 라는 개념을 알게 되었고 기존에 복잡하게 얽혀 있는 의존성을 역할과 책임에 따라 분리하여 클린 아키텍처를 설계 하고자하는 욕구가 강하게 들었다. 

---

## 멀티 모듈 프로젝트를 설계 규칙 정하기 

멀티 모듈 프로젝트를 설계하기 위해서 가장 먼저 했던 건 '어떤 기준으로 모듈을 분리할 것인가?' 였다. 그 기준을 설정하기 위해 '우아한 테크 세미나'를 참고 했고, 다음과 같은 나름의 질문과 답변을 통해 멀티 모듈 프로젝트를 설계했다. 

```md
Q1. 모듈은 역할과 책임이 분명해야한다고 하는데 역할과 책임의 범위는 어떻게 설정하지? 

A1. 모듈은 계층이 존재한다. 즉, 계층이 존재하고 역할과 책임이 존재한다. 
```

```md
Q2. 계층을 어떻게 나눌것인가? 

A2. 계층을 직접 나눠보자 
    - 어플리케이션 모듈(서비스의 흐름을 제어하는 모듈 - 어플리케이션 시작 지점이라고 정의)
    * 내부 모듈(로그, 예외처리, 요청/응답 스팩 관리 등 AOP 전담 모듈) / 도메인 모듈(도메인이 생성, 수정, 삭제되는 로직을 전담하는 모듈)
    - 공통 모듈(순수 자바 클래스만을 사용하는 모듈로 어떠한 의존성도 없어야 함.)
    - 외부 모듈(특정 기술에 대한 스팩 정의, 언제든 다른 기술로 변경될 수 있고 변경되더라도 다른 모듈에서 변경에 대해 알지 못하도록 관리)

우아한 테크 세미나에서 분리했던 계층 구조는 동일하지만 설명한 내용과 내가 이해한 내용이 조금 다를 수 있다고 생각하지만 내 나름의 기준을 두고 모듈화 했다고 생각했기에 잘못됬다고 생각하지 않는다. (코드에 내 주관이 들어갔다면 그 코드를 작성한 이유에 대해 명확히 설명만 할 수 있고 납득할 수 있다면 그만이라고 생각한다.)
```

```md
Q3. 각 모듈의 네이밍 규칙은 어떻게 할 것인가? 

A3. '계층-역할/책임'으로 모듈의 이름을 작성하자. 예를 들어, 
    - 'application-api' : client로부터 api 요청을 받는 어플리케이션 계층 모듈
    - 'domain-jpa' : jpa를 사용해 도메인의 생명주기를 관리하는 도메인 계층 모듈
```

```java
Q4. 어플리케이션을 어떻게 실행할 수 있지? 

A4. 두 가지 방법이 있다고 생각한다. 
    1. 어플리케이션 모듈들을 실행하는 모듈을 하나 따로 분리하다. 
    2. root 프로젝트위치에서 실행한다. 

    두가지 방법중 2번째 방법을 선택했고 다음과 같이 코드를 작성했다. 

    // root project 에 build.gradle
    dependencies {
        ...
        implementation project(':application-api')
        implementation project(':application-worker')
        ...
    }

    root 프로젝트는 반드시 어플리케이션 계층의 모듈만 볼 수 있다. 
    (내부, 도메인, 외부 모듈은절대 알아서는 안된다가 내가 정의한 기준이었다.)
```

---

## 모듈의 Bean 등록 실패 문제 

어플리케이션은 정상적으로 실행되지만 어플리케이션 계층에 @Component를 상속받는 모든 클래스들의 빈등록이 안되는 문제가 발생했다. 

```gradle
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-web'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'

    // websocket
    implementation group: 'com.squareup.okhttp3', name: 'okhttp', version: '4.9.3'
    implementation group: 'org.apache.httpcomponents', name: 'httpclient', version: '4.5.13'
    implementation 'org.springframework.boot:spring-boot-starter-websocket'

    // apache commons
    implementation group: 'org.apache.commons', name: 'commons-lang3', version: '3.12.0'
    implementation group: 'org.apache.commons', name: 'commons-math3', version: '3.6.1'

    implementation project(':domain-jpa')
}

bootJar {enabled = false}
jar {enabled = true}
```

위의 코드는 application-api 모듈의 build.gradle 코드이다. 

\root 프로젝트 입장에서 모든 서브 모듈은 라이브러리로써 동작한다. 그 이유는 각 build.gradle에 bootJar는 false, jar는 true로 되어 있는 부분에서 확인할 수 있는데, bootJar는 실행가능한 환경을 같이 패키징하고 jar는 실행가능한 환경은 고려하지 않는 순수 자바 클래스 및 yaml 파일만 패키징한다. 

그래서 root 폴더 입장에서 @SpringbootApplication 어노테이션이 붙은 클래스에서 어플리케이션이 실행될 때, 등록된 모든 빈을 조회하는데 라이브러리로써 동작하는 모든 서브 모듈 안에서 빈을 찾을 수 없는것이다. 

이 문제를 해결하는 방법은 생각보다 간단했는데 빈을 찾을 base package를 설정해주면 된다는 것이었다. (물론 어플리케이션 계층만 등록한다. )

```java
@SpringBootApplication(scanBasePackages = {"com.api", "com.worker"})
```





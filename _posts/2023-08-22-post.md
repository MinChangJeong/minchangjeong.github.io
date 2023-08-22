---
layout: post
title: "Jib 도입 - Dockerfile 대체"
date: 2023-08-22
excerpt: "Jib란?"
tags: [Docker, Jib, Springboot, backend]
comments: true
---

## SpringBoot에서 배포하는 하는 방법

보통 Docker를 사용하기 전에는 Java/SpringBoot로 개발된 어플리케이션을 배포하기 위해서는 다음과 같은 프로세스를 거쳤다. 

* 배포할 서버에 Java Runtime 환경을 구성한다. 
* 개발 환경에서 Maven 이나 Gradle을 사용해서 프로젝트를 Build한다. Build 후에는 Jar파일이 생성된다. 
* Jar 파일을 배포 서버에 복사하고 실행시킨다. 

하지만 일반적으로 어플리케이션 하나만을 배포하여 서비스를 운영하는 경우는 드물고, DB나 kafka, fronted server등 기타 다양한 어플리케이션 간의 통신을 하는 경우가 많기 때문에 Docker를 사용해 각각의 어플리케이션을 컨테이너화 해서 배포하는 경우를 사용하고 있었다. 

Docker를 사용해서 SpringBoot 어플리케이션을 컨테이너 이미지화 하기 위해서는 Dockerfile 작성이 필요하다. 아래는 Dockerfile 예시이다. 

```Dockerfile
FROM openjdk:17
ARG JAR_FILE=build/libs/*.jar
COPY ${JAR_FILE} app.jar
ENTRYPOINT ["java", "-jar", "/app.jar"]
```

```shell
docker build -t [새로 생성할 이미지 이름] [Dockerfile 디렉토리 경로]
docker run --name [새로 생성할 컨테이너 이름] - it [새로 생성한 이미지 이름]
```

위의 명령어를 사용해 이미지를 생성하기 위해 build하고 생성된 이미지를 컨테이너로 실행시키는 방법도 있지만 나는 일반적으로 다수의 컨테이너를 한번에 생성하기 위해서 docker compose를 사용한다. 

Docker를 공부해본 분들이라면 Docker Layer Caching을 사용해 변경된 부분만을 빌드함으로써 빌드 시간을 단축하는 것의 중요성에 대해 알고 있을 것으로 생각한다. 

Dockerfile을 작성하면서 Layer cahing을 적용해 base image를 가져오거나 의존성 등, 잘 변하지 않는 부분은 빌드 과정에서 캐싱하는 방법을 사용해 빌드 시간을 단축 할 수 있다. 

이 방법도 물론 매우 훌륭하지만 굳이 단점을 꼽자면 다음과 같다 .

* 불필요한 jar를 생성하고 압축을 해제하고 의존성을 분리하는 작업을 포함해야한다.
* 도커 이미지를 생성하기 위해 Docker 데몬이 실행되어 있어야 한다. 

이와 같은 단점을 해결하기 위해서 오늘 소개할 Jib는 Maven 이나 Gradle의 도움으로 쉽게 Java 어플리케이션을 도커 이미지로 생성할 수 있습니다. 물론 복잡한 Dockerfile의 작성이나 도커 데몬을 실행할 필요도 없죠

----

## Jib란?

Jib는 구글에서 개발한 도구로 도커를 사용하지 않고 자바 어플리케이션을 컨테이너 이미지로 변환할 수 있게 도와줍니다. 

Jib의 작동 방식에 대해 설명하자면, 본래 자바 어플리케이션을 jar파일과 함께 하나의 이미지 레이어로 빌드하는 것과 다릅니다. 

jib 빌드 전략은 자바 어플리케이션을 여러 레이어로 분리하여 더 세분화된 점진적 빌드를 가능하게 합니다. 즉, 변경된 부분에 대해서만 재빌드함으로써 빌드 과정에서 시간을 단축 할 수 있게 되는 겨죠. 

----

## Dockerfile과 Jib의 도커 이미지 생성 비교

<img width="871" alt="스크린샷 2023-08-22 오후 3 30 06" src="https://github.com/MinChangJeong/minchangjeong.github.io/assets/65451455/557c5623-a47a-4230-bc6c-cfe17ce31044">

보통 jib에 대해 구글링을 하면 가장 많이 나오는 이미지 사진이라 저도 이걸 가져와서 두 프로세스의 차이를 설명해보려고 합니다. 

먼저 Dockerfile을 사용한 이미지 빌드 프로세스는 다음과 같습니다. 

* Dockerfile 작성
* Dockerfile 빌드
* Docker 데몬과 통신을 통해 이미지 빌드를 진행함. 

여기서 도커 데몬 초기화 및 컨테이너 이미지 레이어 생성에 시간이 소요됩니다. 

다음은 Jib를 사용한 이미지 빌드 프로세스는 다음과 같습니다. 

* Gradle 또는 Maven을 사용해 jib 플러그인 설정
* jib 빌드 실행
* 이미지 생성 및 푸시

jib의 경우 이미지를 생성하고 푸시하는 과정에서 이미지 레이어를 생성하고 지정한 레지스트리 (도커 허브 default)로 이미지를 푸시합니다. 이 때 도커 데몬은 사용하지 않습니다. 

제가 궁굼했던점은 도커 데몬을 사용하지 않는게 왜 장점일까 하는 부분이었습니다.

그 답은 도커 데몬과 통신을 하는 시간을 절약할 수 있기 때문에 빌드 속도가 빨라진다는 것에 있었습니다. 또한 보안적인 측면에서도 강점을 갖는ㄴ데 jib는 도커데몬을 사용하지 않기 때문에 호스트 시스템이 도커 데몬에 접근할 필요를 없엘수 있습니다. 이로 인해 빌드 프로세스가 보다 격리되고 보안적으로 안전한 환경을 진행할 수 있게 되는 것이었습니다. 

----

## Jib 적용하기 

그럼 어떻게 jib를 적용할 수 있는지 구체적인 예시를 보이겠습니다. 모든 코드는 [Github](https://github.com/MinChangJeong/wanted-pre-onboarding-backend)에 있음으로 참고 부족한 내용은 한번더 확인 해보시는 것을 추천드립니다. 

1. 빌드 도구 선택 - Gradle 사용

저는 Gradle을 사용해 jib 플러그인을 설정했습니다. build.gradle에 플러그인을 추가합니다. 

```gradle
plugins {
    ...
    id 'com.google.cloud.tools.jib' version '3.2.1'
    ...
}


jib {
    from {
        image = "openjdk:17"
    }
    to {
        image = [new docker image name]
        tags = ["version1", "version2"]
    }
    container {
        jvmFlags = ["-Xms128m", "-Xmx128m"]
    }
}
```

2. jib 빌드 

다음은 jib 빌드를 실행합니다. 성공적으로 빌드가 끝나면 docker hub에 이미지가 푸시되는 것을 확인 할 수 있습니다 .

```shell
./gradlew jib
```

끝입니다. 생각보다 단순합니다. 빌드 시간을 단축하기 위해서 갖은 노력을 한다는 점이 개인적으로 흥미로웠습니다. Dockerfile을 사용할 때와 Jib를 사용해 빌드하는 시간을 분석하는 것에 대해서는 이후 포스팅에서 다뤄봐야 할것 같습니다. 

----

출처
* https://medium.com/@gaemi/spring-boot-%EA%B3%BC-docker-with-jib-657d32a6b1f0
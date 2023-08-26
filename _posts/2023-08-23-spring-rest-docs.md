---
layout: post
title: "Spring REST Docs 적용"
date: 2023-08-23
excerpt: "Spring Swagger에서 Spring REST Docs로 전환"
tags: [Spring REST Docs, API, backend]
comments: true
---

## 왜 Spring REST Docs를 사용했는가?

* Spring Swagger
* Spring REST Docs

나는 Spring REST Docs을 알기 전에 API 문서를 작성하기 위해서 Spring Swagger를 사용했다. 

Spring Swagger에서 Spring REST Docs으로 변경한 이유는 다음과 같다. 

1. 비즈니스 코드에 스웨거를 위한 코드를 작성해야하는데 이게 제일 별로 였다. 
2. 문서를 검증할 수 있는 방법이 없었다. 

Spring REST Docs를 사용한 이유는 다음과 같다 .

1. 테스트 코드를 작성하고 테스트를 통과해야만 문서가 작성됨
2. 비즈니스 코드와 API 문서를 위한 코드 완벽 분리 
3. 문서 검증 가능

아래에서 코드를 통해 더 자세히 설명하겠지만 Spring REST Docs는 API 요청을 처리하는 테스트 코드를 작성하고 해당 테스트 코드가 통과되었을 때 문서를 자동으로 작성해 준다. 때문에 API도 테스트 가능 하기 때문에 안전하고 비즈니스 코드와도 완벽하게 분리 될 수 있는 것이다. 

----

## Spring REST Docs 적용하기 

1. build.gradle 의존성 설정

```gradle

dependencies {
    // Spring Rest Docs
    testImplementation 'org.springframework.restdocs:spring-restdocs-mockmvc'
    testImplementation 'org.springframework.restdocs:spring-restdocs-asciidoctor'
}

tasks.named('test') {
    useJUnitPlatform()
}

ext {
    snippetsDir = file('build/generated-snippets')
}

test {
    outputs.dir snippetsDir
    useJUnitPlatform()
}

asciidoctor { // asciidoctor 작업 구성
    dependsOn test // test 작업 이후에 작동하도록 하는 설정
    configurations 'asciidoctorExtensions' // 위에서 작성한 configuration 적용
    inputs.dir snippetsDir // snippetsDir 를 입력으로 구성

    sources{
        include("**/index.adoc")
    }
    baseDirFollowsSourceFile()
}

asciidoctor.doFirst {
    delete file('src/main/resources/static/docs')
}

task copyDocument(type: Copy) {
    dependsOn asciidoctor
    from file("build/docs/asciidoc")
    into file("src/main/resources/static/docs")
}

build {
    dependsOn copyDocument
}

bootJar {
    dependsOn asciidoctor
    from ("${asciidoctor.outputDir}/html5") {
        into 'static/docs'
    }
}
```

asciidoctor 세팅을 보면 'dependsOn test'라는 구문이 있는데, 이것은 테스트 작업 이후에 작동하도록 하는 설정으로 Spring REST Docs가 테스트에 의존하고 있다는 것을 확인할 수 있다. 

snippetsDir = file('build/generated-snippets') 구문은 gradle 빌드 이후에 문서가 생성되는 위치를 지정하는 곳으로 아래 그림처럼 문서 작성을 위한 폴더를 생성하게 된다. 

<img width="294" alt="스크린샷 2023-08-23 오후 6 41 15" src="https://github.com/MinChangJeong/minchangjeong.github.io/assets/65451455/4fb9ea5c-127f-413c-96fb-2a468959c075">

하나만 더 보자면 'sources{ include("**/index.adoc") }' 라는 구문을 확인 할 수 있는데 Spring REST Docs는 asciidoctor의 플러그인을 사용해 문서를 생성한다. 이때 .adoc이라는 파일을 통해 문서를 만들게 되는데 모든 .adoc 파일들을 문서작성을 위해 포함하겠다는 의미이다. 

2. *.adoc 파일 작성

```adoc
-- index.adoc

:doctype: book
:icons: font
:source-highlighter: highlights
:toc: left
:toclevels: 4
:sectlinks:

= API Documentation

include::Member-API.adoc[]

-- Member-API.adoc

:doctype: book
:icons: font
:source-highlighter: highlights
:toc: left
:toclevels: 4

== Member API

=== 로그인
operation::login[snippets='http-request,http-response']

=== 회원 등록
operation::register member[snippets='http-request,http-response']

```

adoc 파일은 위에서 설명한것 처럼 문서를 생성하기 위한 파일로 따로 작성이 필요하다. adoc 파일을 작성하기 위한 문법은 이 글에서는 다루지 않겠다. 

짚고 넘어가야 할부분은 'operation::register member'구문에서 **register member** 부분인데 이 부분의 의미는 build/generated-snippets 폴더 아래 생기게될 폴더의 이름들이다. 아래에서 더 자세하게 설명하겠다. 

그리고 한가지 더, index.adoc 파일에 Member-API.adoc[] 이라는 구문을 확인할 수 있는데 이것은 Member-API.adoc 파일을 포함한다는 내용으로 이렇게 해놓은 이유는 각 도메인 별로 API 문서를 생성할 파일을 분리 하기 위해서 였다. 

3. Test Configuration - Spring REST Docs + MockMvc

MockMvc는 Spring Framework의 일부로 제공되는 클래스로, Spring 기반 애플리케이션의 웹 계층을 테스트하기 위한 모의(Mock) 웹 환경을 제공하는 도구이다. 실제 서버를 실행하지 않고도 컨트롤러의 동작을 시뮬레이션하고 웹 요청과 응답을 테스트할 수 있기 때문에 우리는 이 MockMvc를 사용해서 컨트롤러 레벨에서 테스트코드를 작성하고 문서 생성을 자동화 할 것이다. 

```java
@TestConfiguration
public class RestDocsConfiguration {
    @Bean
    public RestDocsMockMvcConfigurationCustomizer restDocsMockMvcConfigurationCustomizer() {
        return configurer ->
                configurer
                    .operationPreprocessors()
                    .withRequestDefaults(prettyPrint())
                    .withResponseDefaults(prettyPrint());
    }
}
```

4. Support Class - RestDocumentTest.java

```java

@ExtendWith({RestDocumentationExtension.class, SpringExtension.class})
@Import(RestDocsConfiguration.class)
@AutoConfigureRestDocs
@WebMvcTest
public class RestDocumentTest {
    @Autowired
    public ObjectMapper objectMapper;

    protected MockMvc mockMvc;
    @MockBean private JpaMetamodelMappingContext jpaMetamodelMappingContext;

    protected String toRequestBody(Object value) throws JsonProcessingException {
        return objectMapper.writeValueAsString(value);
    }

    @BeforeEach
    public void setupMockMvc(
            WebApplicationContext ctx,
            RestDocumentationContextProvider restDocumentationContextProvider) {
        mockMvc =
                MockMvcBuilders
                        .webAppContextSetup(ctx)
                        .apply(
                                documentationConfiguration(restDocumentationContextProvider)
                                        .uris()
                                        .withScheme("http")
                                        .withHost("localhost")
                                        .withPort(8080))
                        .alwaysDo(print())
                        .build();
    }
```

이 클래스는 Spring REST Docs를 위해 나중에 작성하게될 controller 레벨에서 테스트 코드를 작성하기 위해서 공통적으로 사용하는 부분을 분리한 것이다. 각각의 어노테이션에 대해 보고 넘어가자.

* @ExtendWith({RestDocumentationExtension.class, SpringExtension.class}):
이 어노테이션은 JUnit 5 확장 모델을 사용하여 테스트 환경을 확장

    * RestDocumentationExtension.class는 Spring REST Docs의 테스트 확장을 활성화하며, 
    
    * SpringExtension.class는 Spring Framework 테스트 지원을 확장한다 
    
    따라서 이 두 확장이 함께 사용되어 Spring REST Docs가 Spring 기반 테스트와 통합될 수 있도록 한다. 

* @Import(RestDocsConfiguration.class):
이 어노테이션은 테스트 컨텍스트에 RestDocsConfiguration.class라는 클래스를 가져온다. 이 클래스는 Spring REST Docs를 구성하는 데 사용된다. 주로 API 문서 생성 및 포맷팅과 관련된 설정을 처리합니다.

* @AutoConfigureRestDocs:
이 어노테이션은 Spring REST Docs의 자동 구성을 활성화한다. 이를 통해 테스트 클래스 내에서 자동으로 생성된 문서를 관리하고 제어할 수 있습니다. 예를 들어, API 엔드포인트의 요청과 응답을 캡처하고 문서화하는 데 사용될 수 있습니다.

* @WebMvcTest:
이 어노테이션은 Spring MVC 컨트롤러 레이어에 대한 단위 테스트를 수행하는 데 사용된다. 이 테스트는 웹 관련 빈들만 로드하므로, 전체 애플리케이션 컨텍스트를 로드하는 것보다 가볍고 빠르다. @WebMvcTest를 사용하면 컨트롤러의 동작을 테스트하고 해당 컨트롤러에 대한 API 문서를 생성할 수 있다.

5. Support Class - ApiDocumentUtils

이 클래스는 주석으로도 설명을 적어 놨지만 생성될 문서를 이쁘게 보이게 하는 역할을 수행한다. 

```java
public class ApiDocumentUtils {

    // OperationRequestPreprocessor 는 API 문서화를 위해 요청에 대한 사전처리를 수행
    // prettyPrint() 메서드는 요청/응답을 보기 좋게 만듬.
    public static OperationRequestPreprocessor getDocumentRequest() {
        return preprocessRequest(prettyPrint());
    }

    // OperationResponsePreprocessor 는 API 문서화를 위해 응답에 대한 사전처리를 수행
    public static OperationResponsePreprocessor getDocumentResponse() {
        return preprocessResponse(prettyPrint());
    }
}

```

6. domain controller test

given-when-then 패턴으로 테스트를 작성하였다. 

```java
@WebMvcTest(MemberController.class)
@DisplayName("MemberController 에서")
class MemberControllerTest extends RestDocumentTest {
    @MockBean private MemberService memberService;

    @Test
    @DisplayName("회원을 성공적으로 등록하는가?")
    void successRegisterMember() throws Exception {
        // given
        MemberRegisterRequest request
                = new MemberRegisterRequest(
                "test@naver.com",
                "test12345");

        // when
        when(memberService.registerMember(request))
                .thenReturn(new MemberRegisterResponse());
        
        ResultActions perform =
                mockMvc.perform(
                        post("/member")
                            .contentType(MediaType.APPLICATION_JSON)
                            .content(toRequestBody(request))
                );

        // then
        perform.andExpect(status().isOk());

        // docs
        perform.andDo(print())
                .andDo(document("register member",
                        getDocumentRequest(),
                        getDocumentResponse()));
    }
}
```

controller 레벨의 테스트 코드를 작성하는 방법은 다음과 같다. 

* 어떤 정보가 필요로 하는가? - Post로 요청을 받는 경우 body에 어떤 DTO가 필요한지 알려준다. 
* 무슨 로직을 수행하는가? - controller는 service에 어떤 작업을 요청하는지 명시하고 클라이언트에게 어떤 응답을 주는지 명시한다. 
* 어떻게 요청을 보내는가? - ResultActions perform 객체는 HTTP Method, content type, content등의 내용을 구성해서 어떻게 API 요청을 보낼지 명시한다. 

나는 대게 이단계를 거쳐서 컨트롤러 수준의 테스트 코드를 작성한다. 

그리고 눈에 띄는 부분은 위에서 언급한 adoc 파일에서 보였던 'register member' 문구이다. 

'document("register member")' 구문은 테스트가 통과되었을 때 build 폴더내에 생성될 폴더의 이름을 의미한다. 이 폴더를 가지고 와서 adoc 파일에서 API문서를 만드는 것이다. 

<img width="1504" alt="스크린샷 2023-08-23 오후 7 03 35" src="https://github.com/MinChangJeong/minchangjeong.github.io/assets/65451455/992d2e32-ba63-47bf-9a2f-8e4763f266f2">

위의 사진처럼 성공적으로 테스트를 통과하면 다음과 같은 문서를 생성한다. 


<img width="1321" alt="스크린샷 2023-08-23 오후 7 11 15" src="https://github.com/MinChangJeong/minchangjeong.github.io/assets/65451455/391a0728-2c6d-4dd1-b5ca-dc4937eadf73">


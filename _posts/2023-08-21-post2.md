---
layout: post
title: "Exception Handler 구현 - AOP로 처리"
date: 2023-08-21
excerpt: "Exception을 처리하기 위한 커스텀 프레임워크 구축"
tags: [AOP, Exception, Springboot, backend]
comments: true
---

## Exception 처리를 해야하는 이유

여러 프로젝트를 진행하면서 예외 상황에 대해 사전에 파악하고 처리를 해야함의 필요성을 느낀다. 예외 상황을 분석하면서 사용자가 시스템의 오류에 대해 인지할 수 있게 만들어주고 무엇보다 디버깅하는 시간을 줄일 수 있기 때문이다. 그래서 이번 글은 내가 어떻게 예외처리를 했는지에 대해 정리하려고 한다. 

----

## 사전 지식 - AOP 란?

먼저 예외처리를 위해 내가 도입한 방식은 전역 프로젝트 수준에서 예외를 처리하는 코드를 분리 시켰다는 것이다. 예를 들어 게시글을 작성하는 로직에서 발생하는 예외처리, 회원을 등록하는 예외처리 모두 코드내에 예외를 위한 처리를 try/catch문의 작성이 필요하다고는 누구나 예상할 수 있다. 

예를 들어 이메일 중복을 허용하지 않는 예외를 발생했을 때 요청을 보낸 프론트엔드에 예외가 발생했다는 것을 알려주기위한 후처리가 필요했다. 다음과 같이 말이다. 

```java
    protected ResponseEntity<ErrorResponse> handleEmailDuplicateException(
            MethodArgumentNotValidException e) {
        final ErrorResponse response = new ErrorResponse("SERVER ERROR", e.getMessage());
        return new ResponseEntity<>(response, INTERNAL_SERVER_ERROR);
    }
```

즉, 예외가 발생하면 ErrorResponse에 에러 발생 상황과 이유를 담은 객체를 전달하고자 했다. 이 로직을 모든 서비스에서 발생했을 때 동일하게 처리하기 위해서 AOP라는 개념을 도입했다. 

즉, **AOP란 여러 코드에 작성된 비슷한 관심사(여기서는 예외 발생후 처리)를 한 곳에서 처리하도록 설계하는 기법을 의미한다.** 아래에서 코드를 통해 구체적으로 소개하겠다. 

----

## Exception 처리하기

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler
    protected ResponseEntity<ErrorResponse> handleRuntimeException(BusinessException e) {
        final ErrorCode errorCode = e.getErrorCode();
        final ErrorResponse response =
                ErrorResponse.builder()
                        .httpStatus(errorCode.getStatus())
                        .message(errorCode.getMessage())
                        .build()
        return ResponseEntity.status(errorCode.getStatus()).body(response);
    }
}
```

* @RestControllerAdvice : 이 클래스에 예외 처리에 관한 메서드를 정의하면, 해당 예외들이 발생할 때 Spring은 이 메서드를 호출하여 예외 처리 로직을 수행한다. 이 코드에서 주목할 점은 'BusinessException' 이라는 예외가 발생했을 때 해당 헨들러에서 처리를하도록 구현한 것이다. 


```java

// 예외 상황에 대한 내용을 알려줄 에러 코드 
@Getter
@AllArgsConstructor
public enum ErrorCode {
    EMAIL_DUPLICATION_ERROR(HttpStatus.INTERNAL_SERVER_ERROR, "중복된 이메일이 존재합니다. ");

    private final HttpStatus status;
    private final String message;
}


// 예외 발생시 예외 상황을 전달할 객체
@Getter
@Builder
public class ErrorResponse {
    private HttpStatus httpStatus;
    private String message;

    public ErrorResponse(HttpStatus httpStatus, String message) {
        this.httpStatus = httpStatus;
        this.message = message;
    }

    public ErrorResponse(ErrorCode code) {
        this.httpStatus = code.getStatus();
        this.message = code.getMessage();
    }
}


// 모든 비즈니스 상황에서 발생할 수 있는 예외에 대한 부모 예외 클래스를 생성
@Getter
public class BusinessException extends RuntimeException {

    private final ErrorCode errorCode;

    public BusinessException(ErrorCode errorCode) {
        super(errorCode.getMessage());
        this.errorCode = errorCode;
    }
}

// 예시) 이메일 중복 예외
public class EmailDuplicateException extends BusinessException {

    public EmailDuplicateException() {
        super(ErrorCode.EMAIL_DUPLICATION_ERROR);
    }
}
```

정리하자면, Global Exception Handler에 등록된 BusinessException은 모든 비즈니스 로직을 처리하면서 발생할 예외들이 상속받는 부모 클래스로 GlobalExceptionHandler의 코드를 단순화 시키면서도 예외를 처리할 수 있도록 코드를 작성했다. 

```java
if(memberRepository.findByEmail(request.getEmail()) != null) {
    throw new EmailDuplicateException();
}
```

Service에 작성된 코드 예시로 이런식으로 예외가 발생했을 때 Exception을 발생시켜서 예외를 처리하도록 하기만 하면 된다. 생각보다 간단한 방법으로 똑똑하게 예외를 처리하지 않았나 싶다.

더 자세한 코드는 [Github 코드](https://github.com/MinChangJeong/wanted-pre-onboarding-backend)를 참고하면 좋을 듯하다. 
---
layout: post
title: "멀티 모듈 - Entity & Repository Scan 실패 문제"
date: 2023-09-16
excerpt: "멀티 모듈 프로젝트에서 @Entity / @Repository 빈을 찾지 못하는 이유와 해결 방법"
tags: [multi module, backend]
comments: true
---

## 문제 상황

```log
Description:

Parameter 0 of constructor in com.api.coin.upbit.service.UpbitService required a bean of type 'com.domain.repository.MarketRepository' that could not be found.


Action:

Consider defining a bean of type 'com.domain.repository.MarketRepository' in your configuration.
```

모놀리식 프로젝트에서는 만나지 못해서 매우 당황스러운 문제가 발생했다. @Entity와 @Repository 어노테이션이 붙은 Component를 빈으로 등록하지 못하고 있는 문제가 발생했다. (@Entity 어노테이션이 붙은 클래스를 hibernates가 테이블로 생성하지 못하고 있다.)

## 원인은 SpringBootApplication 

```java
...
@EnableAutoConfiguration
@ComponentScan(excludeFilters = { @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
		@Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication {

	...
	/**
	 * Base packages to scan for annotated components. Use {@link #scanBasePackageClasses}
	 * for a type-safe alternative to String-based package names.
	 * <p>
	 * <strong>Note:</strong> this setting is an alias for
	 * {@link ComponentScan @ComponentScan} only. It has no effect on {@code @Entity}
	 * scanning or Spring Data {@link Repository} scanning. For those you should add
	 * {@link org.springframework.boot.autoconfigure.domain.EntityScan @EntityScan} and
	 * {@code @Enable...Repositories} annotations.
	 * @return base packages to scan
	 * @since 1.3.0
	 */
	@AliasFor(annotation = ComponentScan.class, attribute = "basePackages")
	String[] scanBasePackages() default {};
    ...
```

@SpringBootApplication의 상세 내용을 확인하면 'scanBasePackages'에 다음과 같은 내용을 확인할 수 있다. 

* @Entity 클래스 및 Spring Data Repository 스캔에는 영향을 미치지 않는다고 설명하고 있다. @Entity 클래스를 스캔하려면 @EntityScan 애노테이션을 사용하고, Spring Data Repository 인터페이스를 스캔하려면 @Enable...Repositories 애노테이션을 사용해야 한다. 

이렇게 Entity와 Repository는 스캔에 영향을 미치지 않게 설계해논 이유에 대해 찾아보니 다음과 같았다. 

* Entity 클래스 및 Repository 주로 JPA와 관련이 있으며 JPA는 DB와 상호작용하기 위해 존재한다. 
스프링 컨포넌트 스캔은 어플리케이션의 다양한 컴포넌트를 스캔하고 관리하는데 사용되는데 이러한 두가지 역할을 분리하여 관리하면 어플리케이션을 더 모듈화 하고 유지 관리하기 쉽게 만든다.

---

## 문제 해결

문제는 위에서 알려준데로 해결할 수 있었다. 그리고 "더 모듈화한다" 라는 것도 적용하도록 domain 계층의 모듈화에 Entity와 Repository를 스캔하도록 설계했다. 

```java
@EntityScan("com.domain.entity")
@EnableJpaRepositories("com.domain.repository")
@Configuration
public class ScanComponents {
}
```

[깃 주소](https://github.com/backend-deepdive/backend)도 첨부 하니 참고 하면 좋을듯 하다. 

끗
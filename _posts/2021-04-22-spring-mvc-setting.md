---
layout: post
title: 스프링 MVC 구성 요소를 직접 빈으로 등록하기
categories: [Spring]
tags: [Spring Framework, Spring MVC]
description: HandlerMapping등을 직접 빈으로 등록하는 방법
fullview: false
comments: true
---



```java

@Configuration
@ComponentScan
public class WebConfig {

    @Bean
    public HandlerMapping handlerMapping(){
        RequestMappingHandlerMapping handlerMapping = new RequestMappingHandlerMapping();
        //handlerMapping.setInterceptors();
        handlerMapping.setOrder(Ordered.HIGHEST_PRECEDENCE);
        return handlerMapping;
    }

    @Bean
    public HandlerAdapter handlerAdapter(){
        RequestMappingHandlerAdapter handlerAdapter = new RequestMappingHandlerAdapter();
        handlerAdapter.setArgumentResolvers();
        return handlerAdapter;
    }

    @Bean
    public ViewResolver viewResolver(){
        InternalResourceViewResolver viewResolver = new InternalResourceViewResolver();
        viewResolver.setPrefix("WEB-INF/");
        viewResolver.setSuffix(".jsp");
        return viewResolver;
    }
}

```
위와 같이 `DispathcerServlet`에서 기본전략으로 구현하는 인터페이스를 직접 빈으로 등록해 설정할 수 있다. 기본전략이 아니라 사용자가 커스텀하여 원하는 기능을 추가로 설정하고 싶을 때 사용한다.(하지만 거의 사용하는 방법은 아니라고 한다.)


### 조금 더 편리하게 빈을 등록하는 법 -- @EnableWebMvc
애노테이션 기반 스프링 MVC를 사용할 때 편리한 웹 MVC 기본 설정법은 `@EnableWebMvc` 애노테이션을 사용하는 것이다.

`@EnableWebMvc` 애노테이션을 보면 아래와 같이 정의되어 있다. 

```java
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.TYPE})
@Documented
@Import({DelegatingWebMvcConfiguration.class})
public @interface EnableWebMvc {
}

```
여기서 `DelegatingWebMvcConfiguration`는 `WebMvcConfigurationSupport`를 상속하고 있기에, `WebMvcConfigurationSupport`내부 코드를 살펴본다.

내부적으로 보면 `RequestMappingHandlerMapping`,`Interceptor`등을 셋팅할 수 있는 빈으로 설정된 메소드를 볼 수 있다.

위와 같은 방식으로 설정을 하게 되면, `DispathcerServlet`이 아니라 `WebMvc`가 `Delegate`방식(확장성을 좋게 하기 위해)으로 각각의 인터페이스 기본 전략을 등록한다.

#### @EnableWebMvc가 제공하는 빈을 커스터마이징 하는 방법 -- `WebMvcConfigurer`

`WebMvcConfigurer`는 `@EnableWebMvc`가 제공하는 빈을 커스터마이징 할 수 있는 기능을 제공한다. 인터페이스를 사용하면 `ViewResolver`등을 빈으로 등록하지 않아도, `EnableWebMvc`가 등록하는 인터페이스를 커스터마이징 하며 원하는 기능대로 처리할 수 있다.

```java

@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void configureViewResolvers(ViewResolverRegistry registry) {
        registry.jsp("/WEB-INF/",".jsp");
    }
}
```
위 코드는 `ViewResolver`를 커스터마이징해서 prefix를 `/WEB-INF/`, suffix를 `.jsp`로 추가해주었다. 위와 같은 방법은 Spring Boot없이 스프링 MVC를 설정해주는 방법이다.


***
참고자료

1. [스프링 웹 MVC](https://www.inflearn.com/course/%EC%9B%B9-mvc#)
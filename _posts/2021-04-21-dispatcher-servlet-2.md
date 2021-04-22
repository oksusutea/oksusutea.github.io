---
layout: post
title: DispatcherServlet을 등록하는 여러가지 방법
categories: [Spring]
tags: [Spring Framework, Spring MVC]
description: DispatcherServlet 등록 방법
fullview: false
comments: true
---

스프링 MVC의 구성요소, 정확히는 DispatcherServlet이 사용하는 인터페이스에 대해 알아본다. 

## DispatcherServlet

<p style="text-align:center">

<img src="https://media.vlpt.us/images/gone/post/dad7a078-7de8-45c7-b45f-1ab2d0ecd4c3/img%20(1).png" width="400">
</p>

```java
protected void initStrategies(ApplicationContext context) {
        this.initMultipartResolver(context);
        this.initLocaleResolver(context);
        this.initThemeResolver(context);
        this.initHandlerMappings(context);
        this.initHandlerAdapters(context);
        this.initHandlerExceptionResolvers(context);
        this.initRequestToViewNameTranslator(context);
        this.initViewResolvers(context);
        this.initFlashMapManager(context);
    }
```

`DispatcherServlet`은 처음에 초기화 될때 각각의 인터페이스를 초기화 한다. 전략을 따로 설정하지 않은 경우 기본 전략(`DispatcherServlet.properties)`를 사용하여 초기화를 진행한다. 이제 `DispatcherServlet`에서 사용하는 주요 인터페이스에 대해 알아보도록 하자.


### `MultipartResolver`

* 파일 업로드 요청 처리에 필요한 인터페이스, `MultipartResolver`인터페이스 구현체가 빈으로 등록되어 있어야 `DispatcherServlet`이 그 빈을 사용 할 수 있고, 파일 업로드 요청을 처리 할 수 있다.
* 바이너리 데이터를 부분부분 쪼개서 전송하는데, 이 기능을 진행해주는 것이 `MultipartResolver`이다.
* `HttpServletRequest`을 `MultipartHttpServletRequest`로 변환해주어 요청을 담고 있는 File을 꺼낼 수 있는 API를 제공한다.
* `MultipartResolver`의 구현체로는 `CommonsMultipartResolver`와 `StandareServletMultipartResolver` 두가지가 있다.

### `LocalResolver`
* 클라이언트의 위치(Locale) 정보를 파악하는 인터페이스
* 기본 전략은 HTTP요청의 `accept-language`를 보고 판단한다. 그 외에도 session, cookie등을 기준으로 판단하는 구현체도 있다.
* `MessageSource`에서 받은 값을 토대로 보여지는 언어를 판단한다.

### `ThemeResolver`
* 애플리케이션에 설정된 테마를 파악하고 변경할 수 있는 인터페이스
* 보통 키 값을 토대로 적절한 css를 가져와서 테마를 변경한다.(다크모드/라이트모드)

### `HandlerMapping`
* 요청을 처리할 핸들러를 찾는 인터페이스
* 기본적으로 `BeanNameUrlHandlerMapping`(클래스가 핸들러가 됨), `RequestMappingHandlerMapping`(메소드가 핸들러가 됨) 두가지 핸들러가 등록된다.

### `HandlerAdapter`
* HandlerMapping이 찾아낸 **핸들러**를 처리하는 인터페이스
* 스프링 MVC **확장력**의 핵심이다
* 기본적으로 `HttpRequestHandlerAdapter`(거의 안씀), `SimpleControllerHandlerAdapter`, `RequestMappingHandlerAdapter`가 핸들러어댑터로 등록되어 있다.

### `HandlerExceptionResolver`
* 요청 처리 중, 발생한 에러를 처리하는 인터페이스
* 기본적으로 `ExceptionHandlerExceptionResolver`, `ResponseStatusExceptionResolver`, `DefaultHandlerExceptionResolver`가 등록되어 있다.

### `RequestToViewNameTranslator`
* 핸들러에서 뷰 이름을 명시적으로 리턴하지 않은 경우, 요청을 기반으로 뷰 이름을 판단하는 인터페이스

```java
@GetMapping("/sample")
public void sample() {}
```
위처럼 뷰 이름이 없는 경우, 요청을 가지고 뷰를 판단한다.

### `ViewResolver`
* 뷰 이름(String)에 해당하는 뷰를 찾아내는 인터페이스
* 기본 값으로는 `InternalResourceViewResolver`가 등록되어 있다. 이 리졸버는 JSP를 지원한다.

### `FlashMapManager`
* `FlashMap` 인터페이스를 가져오고 저장하는 인터페이스(스프링 3.1부터 생긴 인터페이스이다)
* 기본적으로 `SessionFlashMapManager`가 등록되어 있다.
* `FlashMap`은 주로 리다이렉션을 사용할 때 요청 매개변수를 사용하지 않고, 데이터를 전달하고 정리할 때 사용한다.(화면에서 refresh)를 할 때 같은 데이터를 또 불러오지 않도록 처리한다.
* 중복 form submit을 방지하기 위해 주로 쓰이며, 리다이렉트를 할 때 데이터를 조금 더 편하게 전달하기 위해서 쓰인다.
* 예시 : `redirect:/events?id=200`으로 콜하지 않고, `redirect:/events`만 전달하여도 데이터를 전달할 수 있다.


***
참고자료

1. [스프링 웹 MVC](https://www.inflearn.com/course/%EC%9B%B9-mvc#)
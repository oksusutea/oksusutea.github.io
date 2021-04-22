---
layout: post
title: 스프링 부트의 스프링 MVC 설정
categories: [Spring]
tags: [Spring Framework, Spring MVC]
description: 스프링 부트는 MVC를 어떻게 설정하는지 알아보자.
fullview: false
comments: true
---

이번 시간에는 스프링 부트가 제공하는 스프링 MVC에 대해 알아본다.

### 디버깅 과정

#### HandlerMapping
앞서 진행했던 방식과 동일하게 스프링 부트에서 `doService`에 디버거를 찍어놓고, 어떤 인터페이스 구현체를 가지고 있는지 확인해보자.

<p style="text-align:center">
<img src="https://user-images.githubusercontent.com/75205849/115664047-53ac3900-a37c-11eb-9898-475441ff32fd.png" img width="400">
</p>

총 5개의 `HandlerMapping`이 등록되어 있다. 여기서 이전에는 보지 않았던 `SimpleUrlHandlerMapping`에 대해 자세히 알아보자.
이 구현체는 `resourceHandlerMapping`을 빈으로 등록하고 있다. 스프링부트가 지원하는 정적 리소스 지원 기능은 이 핸들러를 통해서 구현되었다. `resources/static`등 디렉토리 안에 들어 있는 static 리소스들에다가 `resourceHandlerMapping`을 적용하면 캐싱 관련 정보들이 응답 헤더에 추가된다.


### HandlerAdapter

<p style="text-align:center">
<img src="https://user-images.githubusercontent.com/75205849/115664779-3fb50700-a37d-11eb-8947-35fd95a6f0a9.png" img width="400">
</p>

위와 같이 스프링 부트에서는 총 4개의 핸들러 어댑터를 등록한다.(기존엔 `HandlerFunctionAdapter`가 없었는데, 버전이 업되면서 추가된 것 같다.)

* `RequestMappingHandlerAdapter`
* `HandlerFunctionAdapter`
* `HttpRequestHandlerAdapter`
* `SimpleControllerHandlerAdapter`

### ViewResolver

<p style="text-align:center">
<img src="https://user-images.githubusercontent.com/75205849/115665037-a33f3480-a37d-11eb-8c2d-ef2abf31441f.png" img width="400">
</p>

`ViewResolver`는 총 5개를 등록한다.`ContentNegotiatingViewResolver`는 자기가 직접 어떤 뷰 이름에 해당하는 뷰를 찾아주는 것이 아니라, 다른 `ViewResolver`에게 요청을 위임한다.(내부적으로 보면 다른 4개의 `ViewResolver`를 참조하는 것을 볼 수 있다.) 즉, 나머지 4개의 `ViewResolver`는 `ContentNegotiatingViewResolver`에게 요청을 받아 처리하게 된다.

* `ContentNegotiatingViewResolver`
* `BeanNameViewResolver`
* `ThymeleafViewResolver`
* `ViewResolverComposite`
* `InternalSourceViewResolver`

***

#### 스프링 MVC 커스터마이징 방법 
* `application.properties` : 가장 간단하고 확실하게 구현하는 방법
* `@Configuration` + implements WebMvcConfigurer: 스프링 부트의 스프링 MVC 자동 설정 + 추가 설정 (더 합리적)
* `@Configuration` + `@EnableWebMvc` + implements WebMvcConfigurer : 스프링 부트의 스프링 MVC 자동설정을 사용하지 않음




***
참고자료

1. [스프링 웹 MVC](https://www.inflearn.com/course/%EC%9B%B9-mvc#)
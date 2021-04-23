---
layout: post
title: 핸들러 인터셉터
categories: [Spring]
tags: [Spring Framework, Spring MVC]
description: 핸들러 맵핑에 인터셉터 설정하기
fullview: false
comments: true
---

## 핸들러 인터셉터
* 핸들러 맵핑(=어떠한 요청을 처리할 핸들러를 찾아줌)에 설정 할 수 있는 인터셉터
* 핸들러를 실행하기 전, 후(렌더링 전) 그리고 완료(렌더링 끝난 이후) 시점에 부가 작업을 하고 싶은 경우 사용할 수 있다.
* 여러 핸들러에서 반복적으로 사용하는 코드를 줄이고 싶을 때 사용할 수 있다.
	* 로깅, 인증 체크, Locale 변경 등등..

	
### `boolean preHandle(request, response, handler)`	

* 핸들러를 실행하기 전에 호출된다.
* 서블릿 필터가 진행하는 것과 비슷한데, 거기서 더 구체적으로 처리 할 수 있다. (handler에 대한 정보도 제공하기 때문)
* **핸들러**에 대한 정보를 사용할 수 있기 때문에 서블릿 필터에 비해 보다 세밀한 로직을 구현할 수 있다.
* 리턴 값으로 계속 다음 인터셉터 또는 핸들로러로 요청,응답을 전달할 지(true) 응답 처리가 이곳에서 끝났는지(false) 알린다.
	* false로 마치면 후단 인터셉터인 `afterCompletion`은 안탄다.

### `boolean postHandle(requet, response, modelAndView)`

* 핸들러 실행을 마치고 뷰를 렌더링하기 이전에 호출된다.
* **뷰**에 전달할 추가 정보나 여러 핸들러에 공통적인 모델 정보를 담는데 사용할 수 있다.
* 이 메소드는 인터셉터 역순으로 호출된다.
* 비동기적 요청 처리시에는 호출되지 않는다.
	* 비동기 처리시에는 `AsyncHandlerInterceptor`가 제공하는 다른 메소드가 호출된다.

### `boolean afterCompletion(request, response, handler, ex)`

* 요청 처리가 완전히 끝난 뒤(뷰 렌더링이 끝난 뒤) 호출된다.
* `preHandler`에서 `true`를 리턴한 경우에만 호출된다.
* 이 메소드는 인터셉터 **역순**으로 호출된다.
* 비동기적인 요청 처리시에는 호출되지 않는다.
* `RestController`의 경우 뷰 렌더링 과정이 없기 때문에, `postHandler`가 처리된 후 바로 구동된다.

#### 서블릿 필터와의 비교 
* 서블릿보다 구체적인 처리가 가능하다.(스프링에 특화된 정보를 토대로 작업 가능)
* 서블릿은 보다 일반적인 기능을 구현하는데 초점을 둔다.
* 예시 : XSS 스크립팅(스프링과 관련이 없고 웹 애플리케이션 전반적으로 적용해야 하기 때문에 인터셉터보다는 서블릿 필터에 적용한다.


***

### `HandlerInterceptor`구현 방법

`GreetingInterceptor.java`

```java
package me.cjk.demospringmvc;

import org.springframework.web.servlet.HandlerInterceptor;
import org.springframework.web.servlet.ModelAndView;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

public class GreetingInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        System.out.println("preHandle 1");
        return true; // 다음으로 요청처리 할 수 있도록 true 리턴
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        System.out.println("post Handle 1");
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        System.out.println("afterCompletion 1");
    }
}

```
`AnotherGreetingInterceptor.java`

```java
package me.cjk.demospringmvc;

import org.springframework.web.servlet.HandlerInterceptor;
import org.springframework.web.servlet.ModelAndView;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

public class AnotherInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        System.out.println("preHandle 2");
        return true; // 다음으로 요청처리 할 수 있도록 true 리턴
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        System.out.println("post Handle 2");
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        System.out.println("afterCompletion 2");
    }
}

```

`WebConfig`설정 파일에 `Interceptor`추가

```java
package me.cjk.demospringmvc;

import org.springframework.context.annotation.Configuration;
import org.springframework.format.FormatterRegistry;
import org.springframework.web.servlet.config.annotation.InterceptorRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addInterceptors(InterceptorRegistry registry) {

        registry.addInterceptor(new GreetingInterceptor());
        registry.addInterceptor(new AnotherInterceptor());
    }
}

```
인터셉터는 위와 같이 추가한 순서대로 순서가 결정되고, 만일 순서를 지정하고 싶다면 아래와 같이 작성해주면 된다.

```java
 registry.addInterceptor(new GreetingInterceptor()).order(0);
        registry.addInterceptor(new AnotherInterceptor()).order(-1);
```
order가 낮으면 낮을수록 우선순위가 높은 점 유의하자.


이렇게 작성한 후, 스프링부트 어플리케이션을 실행하여 url로 요청하면 아래와 같이 결과가 출력된다.

```
preHandle 1
preHandle 2
post Handle 2
post Handle 1
afterCompletion 2
afterCompletion 1
```

#### 인터셉터를 특정 패턴에만 적용하는 방법

간단하게 `WebConfig`파일에 `addPathpatterns`만 추가하면 된다.

```java
package me.cjk.demospringmvc;

import org.springframework.context.annotation.Configuration;
import org.springframework.format.FormatterRegistry;
import org.springframework.web.servlet.config.annotation.InterceptorRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addInterceptors(InterceptorRegistry registry) {

        registry.addInterceptor(new GreetingInterceptor()).order(0);
        registry.addInterceptor(new AnotherInterceptor())
                .addPathPatterns("/hi")
                .order(-1);
    }
}
```
위와 같이 작성한 후, `/hello`로 요청처리를 하면 `GreetingInterceptor`만 적용된 것을 확인할 수 있다.


***
참고자료

1. [스프링 웹 MVC](https://inf.run/dJFi)
---
layout: post
title: 로그인 후 세션 정보를 애노테이션으로 관리하기
categories: [dev]
tags: [java, dev]
description: getAttribute 사용을 줄이는 방법
fullview: false
comments: true
---

## 현재 상황

```java
/*
 * 아이 내역 수정
 */
 @PutMapping("/{kidId}")
 public void updateKid(@PathVariable int kidId, @Valid @RequestBody Kid kid,
        HttpSession httpSession) {
    int parentId = getSessionUserId(httpSession);
    kidService.updateKid(kid, parentId);
   }
```
중간중간 수정된 부분도 있긴 하지만, 컨트롤러 단에서 항상 httpSession의 userID attribute가져오는 작업을 진행하고 있다. 중복적인 코드를 작성해야 하는 불필요함 때문에, 해당 부분을 개성해보기로 하였다.

### HandlerMethodArgumentResolver
* 이 인터페이스는 컨트롤러 메소드에서 특정 조건에 맞는 파라미터가 있을 때, 원하는 값을 바인딩해주는 인터페이스이다.
* 스프링에서는 Controller에서 @RequestBody를 통해 body 값을 받아올 때와, @PathVariable을 통해 path parameter를 받아올 때 HandlerMethodArgumentResolver를 사용하여 값을 받아온다.


## 구현 방법

### 1. 어노테이션 작성
우선 어노테이션 기반으로 세션 객체를 바인딩 하기 위해, 세션유저 어노테이션을 만든다.

```java
package com.flab.kidsafer.annotation;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Target(ElementType.PARAMETER)
@Retention(RetentionPolicy.RUNTIME)
public @interface LoginUser {

}
```

* @Target(ElementType.PARAMETER) 
	* 이 어노테이션이 사용될 수 있는 위치를 말한다.
	* 컨트롤러의 파라미터단에 쓰이기 때문에, PARAMETER로 지정해주었다. 즉, 메소드의 파라미터에만 사용 가능하다는 뜻이다.
* @Retention(RetentionPolicy.RUNTIME)
	* 어노테이션의 유지 정책을 말한다.
	* RUNTIME은 바이트 코드 파일까지 어노테이션 정보를 유지할 수 있기 때문에, 런타임시 리플렉션을 이용해 해당 어노테이션의 정보를 가져올 수 있다.

### 2. 세션 객체 작성

```java
package com.flab.kidsafer.domain;

import java.io.Serializable;

public class SessionUser implements Serializable {

    private int id;
    private String email;
    private String type;
    private String status;

    public SessionUser(User user) {
        this.id = user.getUserId();
        this.email = user.getEmail();
        this.type = user.getType();
        this.status = user.getStatus();
    }
}
```
* 세션에 저장하기 위해 serializable을 구현하도록 처리한다.

### 3. HandlerMethodArgumentResolver 상속
HandlerMethodArgumentResolver을 상속하기 위해서는 아래 두가지 메소드를 구현해야 한다.

```java
boolean supportsParameter(MethodParameter parameter);
Object resolveArgument(MethodParameter parameter, @Nullable ModelAndViewContainer mavContainer,
			NativeWebRequest webRequest, @Nullable WebDataBinderFactory binderFactory) throws Exception;
```

* supportsParameter : 파라미터가 Resolver에 의해 수행될 수 있는 타입인지를 판단한다. true시에는 resolveArgument()를 실행한다.
	* 우선 메소드의 파라미터들이 methodParameter에 매핑된다. 그 때 annotation으로 @LoginUser을 사용하였으면 true를 반환한다.
	* @LoginUser 어노테이션이 붙어있는 메소드 파라미터가 SessionUser타입이면 true를 반환한다.
* resolveArgument : 실제로 파라미터와 바인딩할 객체를 리턴한다. NativeWebRequest를 통해 클라이언트 요청이 담긴 파라미터를 컨트롤러보다 먼저 받아 작업을 수행할 수 있다.

```java
package com.flab.kidsafer.config;

import com.flab.kidsafer.annotation.LoginUser;
import com.flab.kidsafer.domain.SessionUser;
import javax.servlet.http.HttpSession;
import org.springframework.core.MethodParameter;
import org.springframework.stereotype.Component;
import org.springframework.web.bind.support.WebDataBinderFactory;
import org.springframework.web.context.request.NativeWebRequest;
import org.springframework.web.method.support.HandlerMethodArgumentResolver;
import org.springframework.web.method.support.ModelAndViewContainer;

@Component
public class LoginUserArgumentResolver implements HandlerMethodArgumentResolver {

    private final HttpSession httpSession;

    public LoginUserArgumentResolver(HttpSession httpSession) {
        this.httpSession = httpSession;
    }

    @Override
    public boolean supportsParameter(MethodParameter parameter) {
        boolean isLoginUserAnnotation = parameter.getParameterAnnotation(LoginUser.class) != null;
        boolean isUserClass = SessionUser.class.equals(parameter.getParameterType());

        return isLoginUserAnnotation && isUserClass;
    }

    @Override
    public Object resolveArgument(MethodParameter parameter, ModelAndViewContainer mavContainer,
        NativeWebRequest webRequest, WebDataBinderFactory binderFactory) throws Exception {
        return httpSession.getAttribute("user");
    }
}
```

### 4. Resolver 등록
구현한 HandlerMethodArgumentResolver를 스프링에 등록하는 작업이 필요하다.

```java
package com.flab.kidsafer.config;

import java.util.List;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.method.support.HandlerMethodArgumentResolver;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

@Configuration
public class WebConfig implements WebMvcConfigurer {

    private final LoginUserArgumentResolver loginUserArgumentResolver;

    public WebConfig(LoginUserArgumentResolver loginUserArgumentResolver) {
        this.loginUserArgumentResolver = loginUserArgumentResolver;
    }

    @Override
    public void addArgumentResolvers(List<HandlerMethodArgumentResolver> argumentResolver) {
        argumentResolver.add(loginUserArgumentResolver);
    }
}
```

* Spring Boot에서 WebMvcConfigurer를 구현하는 방법을 통해서 추가로 스프링에 대한 환경설정 작업을 진행할 수 있다.
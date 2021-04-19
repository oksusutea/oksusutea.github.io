---
layout: post
title: 스프링이 제공하는 서블릿 구현체 DispatcherServlet 사용하기
categories: [Spring]
tags: [Spring Framework, Spring MVC]
description: 서블릿 어플리케이션에 스프링을 연동하는 두번째 방법 - DispatcherServlet 사용하기
fullview: false
comments: true
---

앞서 `ContextLoaderListener`만을 이용하여 스프링 IoC 컨테이너를 연동한 방법은 스프링 MVC를 적용한 것이 아니다. 서블릿자체가 `HttpServlet` 자체를 상속받아 구현하였기 때문에 스프링 MVC는 아니고, 기존 서블릿에 스프링 IoC를 연동하는 방법이라고 볼 수 있다. 이제 스프링 MVC를 이용하여 서블릿 애플리케이션에 스프링을 연동해보자.

### 컨트롤러 등록

```java
package me.cjk;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloController {

    @Autowired
    HelloService helloService;

    @GetMapping("/hello")
    public String hello(){
        return "Hello, " + helloService.getName();
    }
}
```
앞서 서블릿을 통해 작성한 것과 달리, `HelloService`를 주입받아 `@GetMapping`과 같은 어노테이션을 통해 요청을 핸들링 하고 싶다. 이러한 방식으로 요청을 핸들링하려면 스프링 MVC를 사용해야하고, 여기서 스프링 MVC를 쓰려면, 핸들러에게 요청을 위임해줄 수 있는, 그리고 애노테이션을 이해하고 리턴 값을 httpResponse로 만들어 줄 수 있는 `DispatcherServlet`을 사용해야 한다. 

기존 서블릿 등록하듯이 `DispatcherServlet`을 등록한다. 

#### WebApplicaionContext에서 분리하여 등록하기

<p style="text-align:center;">
<img src="https://linked2ev.github.io/assets/img/devlog/201909/spring-context-s1.png" width="400">
</p>

위와 같은 구조에서 볼 수 있듯이 `Root WebApplicationContext`는 `ContextLoaderListener`를 통해서 등록하며, 웹과 무관한 것들(Service, Repository)을 빈으로 등록한다. 그리고, `Servlet WebApplicationContext`에서는 웹과 관련된 것들(Controller, ViewResolver, HandlerMapping)을 빈으로 등록해준다. 이와 같은 과정을 구현하기 위해서, `AppConfig`와 `WebConfig`두가지 환경설정 파일을 만들고, 각각 웹/웹이 아닌 빈들을 등록해주도록 지정해주었다.

`AppConfig`파일 : 

```java
package me.cjk;

import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.stereotype.Controller;

@Configuration
@ComponentScan(excludeFilters = @ComponentScan.Filter(Controller.class))
public class AppConfig {
}

```

`WebConfig`파일 : 

```java
package me.cjk;

import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.stereotype.Controller;

@Configuration
@ComponentScan(useDefaultFilters = false, includeFilters = @ComponentScan.Filter(Controller.class))
public class WebConfig {
}
```

`AppConfig`는 웹과 관련되지 않은 것들을 빈으로 등록하기 위해, 아예 그냥 웹과 관련된 빈(==컨트롤러)를 스캔 범위에서 제거하였다. `WebConfig`는 웹과 관련된 것들만 빈으로 등록하기 위해 `@ComponentScan`에서 컨트롤러만 스캔하도록 지정해주었다.

#### DispatcherServlet 등록

```
<!DOCTYPE web-app PUBLIC
 "-//Sun Microsystems, Inc.//DTD Web Application 2.3//EN"
 "http://java.sun.com/dtd/web-app_2_3.dtd" >

<web-app>
  <display-name>Archetype Created Web Application</display-name>

  <context-param>
    <param-name>contextClass</param-name>
    <param-value>org.springframework.web.context.support.AnnotationConfigWebApplicationContext</param-value>
  </context-param>

  <context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>me.cjk.AppConfig</param-value>
  </context-param>

  <listener>
    <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
  </listener>

  <servlet>
    <servlet-name>app</servlet-name>
    <sevlet-class>org.springframework.web.servlet.DispatcherServlet</sevlet-class>
    <init-param>
      <param-name>contextClass</param-name>
      <param-value>org.springframework.web.context.support.AnnotationConfigWebApplicationContext</param-value>
    </init-param>
    <init-param>
      <param-name>contextConfigLocation</param-name>
      <param-value>me.cjk.WebConfig</param-value>
    </init-param>
  </servlet>


  <servlet-mapping>
    <servlet-name>app</servlet-name>
    <url-pattern>/app/*</url-pattern>
  </servlet-mapping>

</web-app>

```
이렇게 설정하면, `/app/`밑으로 들어오는 모든 요청을 `app` 서블릿, 즉 `DispatcherServlet`이 처리하게 될 것이다. 여기서  우리가 앞서 만든 컨트롤러 빈은, web.xml에서 보면 알 수 있듯이 `DispatcherServlet`이 만들어주는  `AnnotationConfigWebApplicationContext`에 등록이 될 것이다. 또한, `AnnotationConfigWebApplicationContext`을 만들 때`DispatcherServlet`은 현재 서블릿컨텍스트에 들어있는 `ContextLoadListener`가 만들어 준 애플리케이션컨텍스트를 부모로 사용하게 된다. 부모로 사용하는 이 애플리케이션 컨텍스트는 
`AppConfig`를 토대로 만들어지는데, 여기서 보면 Controller 클래스를 제외하고 만들어지기 때문에, Service 클래스가 등록될 것이다.

항상 이렇게 부모구조를 만들 필요는 없으며, 필요에 따라 작성하면 된다. 서블릿을 여러개 등록하지 않고 한 번에 처리하는 방법은 아래와 같다. 

#### Web.xml을 단순화하기

```java
<!DOCTYPE web-app PUBLIC
 "-//Sun Microsystems, Inc.//DTD Web Application 2.3//EN"
 "http://java.sun.com/dtd/web-app_2_3.dtd" >

<web-app>
  <display-name>Archetype Created Web Application</display-name>

  <servlet>
    <servlet-name>app</servlet-name>
    <sevlet-class>org.springframework.web.servlet.DispatcherServlet</sevlet-class>
    <init-param>
      <param-name>contextClass</param-name>
      <param-value>org.springframework.web.context.support.AnnotationConfigWebApplicationContext</param-value>
    </init-param>
    <init-param>
      <param-name>contextConfigLocation</param-name>
      <param-value>me.cjk.WebConfig</param-value>
    </init-param>
  </servlet>


  <servlet-mapping>
    <servlet-name>app</servlet-name>
    <url-pattern>/demo/app/*</url-pattern>
  </servlet-mapping>

</web-app>

```
위와 같이 기존 `ContextLoaderListener`부분을 제거하고, `DispatcherServlet`의 환경설정 부분 `WebConfig`에서 기존에 컨트롤러를 스캐닝 범위에서 제거하였던 부분을 생략하면 된다.

`WebConfig` 파일 : 

```java
package me.cjk;

import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

@Configuration
@ComponentScan
public class WebConfig {
} 
```
이렇게 되면 `Controller`와 `Service`가 `WebConfig`에 의해 빈으로 등록될 것이다. 이렇게 되면 기존 `Root WebApplicationContext`와 `Servlet WebApplicationContext`와 같이 계층구조로 나뉘어져 있던 부분은 사라지고 `Servlet WebApplicationContext`만 존재하게 될 것이다.  
빈 설정파일은 이와 반대로  `ContextLoaderListener`가 만드는 `ApplicationContext`에 모든 빈을 다 등록할 수도 있다. 하지만 이 방법은 구조상의 이유로 권장하는 방법은 아니다.  


### 스프링 부트와의 차이점
앞서 만든 방법은 스프링 부트에서 구현되는 애플리케이션 컨텍스트와는 약간 다르다.  
서블릿 기반의 경우  서블릿 컨테이너가 먼저 뜨고, 서블릿 컨테이너 안에 등록되는 서블릿 애플리케이션에 스프링을 연동하는 방법이었다. `ContextLoaderListener` 혹은 `DispatcherServlet`을 이용해서 말이다.  이에 반해 스프링부트는 스프링부트 어플리케이션이 먼저 뜬다. 그 안에 톰캣이 내장서버로 뜨며, 서블릿을 코드로 등록하게 된다. 톰캣 안에 스프링을 넣었다고 생각하면 된다.


***
참고자료

1. [스프링 웹 MVC](https://inf.run/dJFi)
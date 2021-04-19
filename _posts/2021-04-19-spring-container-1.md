---
layout: post
title: 스프링이 제공하는 IoC 컨테이너를 연동하기
categories: [Spring]
tags: [Spring Framework, Spring MVC]
description: 서블릿 어플리케이션에 스프링을 연동하는 첫번째 방법 - 서블릿에 IoC 컨테이너 연동하기
fullview: false
comments: true
---

### 스프링이 제공하는 IoC 컨테이너를 연동하기

우선 web.xml에 의존성 추가한다.

```
<dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-webmvc</artifactId>
        <version>5.1.3.RELEASE</version>
</dependency>
```
스프링 부트를 사용하지 않기 때문에 버전을 명시해주어야 한다. web.xml에 해당 의존성을 넣고 빌드를 하게되면, 웹MVC를 사용하기 위한 다른 모듈도 추가되는 것을 볼 수 있다.

#### `ContextLoaderListener` 리스너로 등록하기

```
  <listener>
    <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
  </listener>
```
추가로 스프링에서 제공해주는 리스너도 `web.xml`파일에 추가해주자. `ContextLoaderListener`는 스프링 IoC 컨테이너, 즉 애플리케이션 컨텍스트를 서블릿 애플리케이션의 생명주기에 맞추어 바인딩해주는 역할을 한다. 웹애플리케이션에 등록되어 있는 서블릿들을 사용할 수 있도록 애플리케이션 컨텍스트를 만들어서, 서블릿컨텍스트에 등록해준다. 그리고 서블릿이 종료될 시점에 애플리케이션 컨텍스트를 제거해준다. 즉, 서블릿 컨텍스트의 라이프사이클에 맞추어 스프링이 제공해주는 애플리케이션 컨텍스트를 연동해주는 가장 핵심적인 리스너이다.  
정리를 해보자면 `ContextLoaderListener`는 : 

* 서블릿 리스너의 구현체이다.
* `ApplicationContext`를 만들어준다. => 스프링 설정파일이 필요하다.
* `ApplicationContext`를 서블릿 컨텍스트 라이프사이클에 따라 등록하고 소멸시켜주는 역할을 한다.
* 서블릿에서 IoC 컨테이너를 ServletContext를 통해 꺼내서 사용할 수 있도록 한다.

`ServletContextListener`는 서블릿 컨텍스트 생성 시점에 `ApplicationContext`를 만들어준다. `ApplicationContext`를 만들기 위해서는 스프링 설정 파일이 필요하다.

```java
public class ContextLoader {
    public static final String CONTEXT_ID_PARAM = "contextId";
    public static final String CONFIG_LOCATION_PARAM = "contextConfigLocation";
    public static final String CONTEXT_CLASS_PARAM = "contextClass";
    .....
}
```
위와 같이 리스너가 사용하는 파라미터가 있다. 컨텍스트 설정파일의 위치라던가, 설정한 애플리케이션컨텍스트의 타입등을 지정할 수 있다. 기본으로는 xml기반의 애플리케이션 컨텍스트를 사용하지만, 요즘은 자바를 많이 사용하기에 아래와 같이 자바로 설정해보도록 한다.

```
  <context-param>
    <param-name>contextClass</param-name>
    <param-value>org.springframework.web.context.support.AnnotationConfigWebApplicationContext</param-value>
  </context-param>
  <context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>me.cjk.AppConfig</param-value>
  </context-param>  
```

위와 같이 `class`파일로 IoC 컨테이너 환경설정을 등록하겠다고 명시하고, 해당 파일은 `me.cjk.AppConfig`에 등록되어있다고 기재하였다. 이 정보를 활용해서 `ContextLoader`가 `AnnotationConfigWebApplicationContext`를 만들면서 `me.cjk.AppConfig`이 설정파일을 가지고 만든다. 그러면 그 ApplicationContext는 우리가 지정한 빈들이 등록될 것이다. 

#### 추가 설명

`ContextLoaderListener` 클래스 파일 : 

```java
public class ContextLoaderListener extends ContextLoader implements ServletContextListener {
    public ContextLoaderListener() {
    }

    public ContextLoaderListener(WebApplicationContext context) {
        super(context);
    }

    public void contextInitialized(ServletContextEvent event) {
        this.initWebApplicationContext(event.getServletContext());
    }

    public void contextDestroyed(ServletContextEvent event) {
        this.closeWebApplicationContext(event.getServletContext());
        ContextCleanupListener.cleanupAttributes(event.getServletContext());
    }
}

```

`ContextLoaderListener`는 `ServletContext`가 만들어질 시점에 `initWebApplicationContext`를 한다. 즉, `WebApplicationContext`를 만들어서 `ServletContext`에 등록하는 과정이 이 때 일어난다.

```java
public WebApplicationContext initWebApplicationContext(ServletContext servletContext) {
       ...
       servletContext.setAttribute(WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE, this.context);   
}
```

`initWebApplicationContext` 클래스를 타고 들어가다 보면, `servletContext.setAttribute`를 볼 수있다. 즉 우리는 `WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE`로 애플리케이션컨텍스트를 꺼내서 쓸 수 있다. 아래와 같이 말이다.

```java
package me.cjk;

public class HelloServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        ApplicationContext context = (ApplicationContext) getServletContext().getAttribute(WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE);
        HelloService helloService = context.getBean(HelloService.class);
        
    }
}
```
위와 같이 `ApplicationContext`를 가져와서 `getBean()`메소드를 통해서 빈을 가져올 수 있다. 이렇게 되면 우리는 `HelloService`를 직접적으로 `new`해서 사용하는 것이 아니라, 스프링이 제공해주는 IoC 컨테이너에 들어있는 빈을 꺼내서 쓰고 있는 셈이다. 여기서 빈만 교체하면 `HelloService`를 다른 빈으로도 교체할 수 있다.

`AppConfig` 클래스 파일 : 

```
package me.cjk;

import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.stereotype.Controller;

@Configuration
@ComponentScan(excludeFilters = @ComponentScan.Filter(Controller.class))
public class AppConfig {
}

```
빈을 등록하는 애플리케이션 컨텍스트 환경설정 파일은 위와 같이 지정해주었다. 이런 방식으로 톰캣을 켜주게 되면 애플리케이션에서 빈의 정보를 가져와 읽을 수 있는 서블릿이 정상적으로 구동된다.

#### 서블릿에서 스프링 연동시의 문제점
앞서 스프링 IoC 컨테이너를 이용하여 기존 서블릿을 스프링에 연동해주는 작업을 진행하였다. 하지만, 서블릿을 이용해 스프링을 연동하게 되면, 요청 하나당 설정해주어야 할 것들이 너무 많다. 한 개의 요청당 아래의 설정파일만큼 추가된다고 볼 수 있다.

```
<servlet>
    <servlet-name>hello</servlet-name>
    <servlet-class>me.cjk.HelloServlet</servlet-class>
  </servlet>

  <servlet-mapping>
    <servlet-name>hello</servlet-name>
    <url-pattern>/hello</url-pattern>
  </servlet-mapping>
```
이를 고민하다보니, 앞선 개발자들은 여러 서블릿에서 공통적으로 처리하고 싶은데 어떻게 구현할 지에 대해서 고민을 하게 되었고(물론 필터로도 적용이 가능하겠지만), 결과적으로  **Front Controller** 라는 디자인 패턴을 통해 해결하였다.
> 모든 요청을 컨트롤러 하나가 받아서 처리를 하고, 그 컨트롤러가 해당 요청을 처리할 핸들러에게 위임(Dispatch)라고 한다.

스프링이 이런 **Front Controller**역할을 하는 서블릿인 `DispatcherServlet`를 미리 구현해두었다.



***
참고자료

1. [스프링 웹 MVC](https://inf.run/dJFi)
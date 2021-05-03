---
layout: post
title: 스프링 MVC 프레임워크에서의 CORS
categories: [Spring]
tags: [Spring Framework, Spring MVC]
description: 스프링 Web MVC에서 CORS를 처리하는 방법
fullview: false
comments: true
---

Spring Document에서 **스프링 웹 MVC**에서는 다루지 않는 CORS에 대해 알아본다.(정확히는 문서를 번역하며 이해해보자)

## 1.7 CORS
Spring MVC는 CORS(Cross-Origin Resource Sharing)을 지원한다. 이번 섹션에는 어떻게 CORS를 처리하는지 알아볼 것이다.

### 1.7.1 Introduction
보안상의 이유로, 브라우저는 AJAX 호출을 통해 현재 origin영역과 다른 외부의 리소스를 받아오는 것을 허용하지 않는다. 예를 들어, 하나의 탭 안에 사용자의 계좌번호가 있고 `evil.com`이라는 브라우저가 있다고 가정해보자. `evil.com`에서 AJAX 요청을 통해 사용자의 계좌 API를 가져와서는 안된다(사용자의 계좌에서 이체를 하는 등 말이다!)
Cross-Origin Resource Sharing(CORS)는 대부분의 브라우저에서 구현되어있는 W3C 규약이다. CORS는 개발자가 IFRAME 혹은 JSONP와 같은 안전하지 않고 임시방편대책을 사용하지 않고, 어떤 종류(범위)의 cross-domain 요청을 승인할 것인지를 정의해야 한다.

### 1.7.2 Processing
CORS는 prefilight하고, simple하고, actual한 요청을 구분한다. 어떻게 CORS가 동작하는지 이해하기 위해서는 [이 포스트](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS)를 읽어보면 더 자세하게 파악 할 수 있다.  
Spring MVC `HandlerMapping`은 CORS를 위한 기능을 제공한다. 요청에 따른 핸들러를 성공적으로 맵핑한 후에, `HandlerMapping`은 주어진 요청의 CORS 설정을 체크하고 그에 따른 처리를 진행한다. preflight요청은 직접적으로 핸들링되며, simple하고 actual한 요청은 가로채지고(intercept), 검증되며(validated) 응답 헤더 셋에 CORS 요청을 담는다.  
CORS를 가능하게 하기 위해서는(`Origin`헤더가 제공되고 요청쪽 호스트랑 다른 경우), **CORS Configuration**을 명시해주어야 한다. 매칭되는 CORS Configuration이 없을 경우, preflight 요청은 거부당한다. simple하거나 actual한 CORS 요청을 보낼 때 CORS 헤더가 응답에 추가되어 전달되지 않을경우, 브라우저는 이 요청을 거부한다.  
각각의 `HandlerMapping`은 URL 패턴 기반의`CorsConfiguration` 매핑을 통해 독립적으로 설정할 수 있다. 대다수의 상황에서는, MVC 자바 설정이나 XML 네임스페이스로 이러한 매핑을 정의하여 모든 `HandlerMapping` 인스턴스에 설정되도록 정의하곤 한다.  
`HandlerMapping`레벨에서 전역적인 CORS configuration을 결합하여 사용 할 수 있다. 예를 들어, 애노테이션기반의 컨트롤러는 클래스레벨 혹은 메소드레벨의 `@CrossOrigin` 애노테이션을 이용해서 적용할 수 있다.(다른 핸들러는 `CorsConfigurationSource`를 구현하도록 한다).  
전역적인(global) 설정과 지역적인(local) 설정을 결합하는 방법은 additive하다. 보통은 지역적인 설정이 전역적인 설정을 override한다.

### 1.7.3 `@CrossOrigin`
`@CrossOrigin`애노테이션은 컨트롤러 메소드에서 cross-origin 요청을 허용하도록 한다.

```java
@RestController
@RequestMapping("/account")
public class AccountController {

    @CrossOrigin
    @GetMapping("/{id}")
    public Account retrieve(@PathVariable Long id) {
        // ...
    }

    @DeleteMapping("/{id}")
    public void remove(@PathVariable Long id) {
        // ...
    }
}
```

기본적으로 `@CrossOrigin`은 :

* 모든 요청과
* 모든 헤더와
* 컨트롤러와 맵핑된 모든 HTTP 요청
을 허용한다.

추가로 `@CrossOrigin`은 클래스레벨에서 셋팅을 허용하고, 클래스 레벨과 메소드 레벨 동시 작성을 허용한다.


### 1.7.4 Global Configuration
잘 정의된 메소드레벨의 설정 말고도, 우리는 전역적(global)으로 CORS configuration을 만들어 줄 수 있다.URL기반의 `CorsConfiguration`맵핑을 어느 `HandlerMapping`에서나 셋팅해줄 수 있다. 그러나 대다수의 어플리케이션은 MVC 자바 설정이나 XML 네임스페이스를 통해 설정을 한다.  
기본적으로, global configuraion은 아래 내용을 허용한다 :

* 모든 요청(origin)
* 모든 헤더
* GET, HEAD, 그리고 POST 메소드

`allowCredentials`은 기본적으로 허용하지 않으며 추가로 설정을 해주어야만 사용이 가능하다.

`maxAge`는 30분으로 셋팅되어 있다.

#### Java Configuration
MVC Java 설정에서 CORS를 허용하려면, `CorsRegistry` 콜백을 사용하면 된다. 

```java
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addCorsMappings(CorsRegistry registry) {

        registry.addMapping("/api/**")
            .allowedOrigins("https://domain2.com")
            .allowedMethods("PUT", "DELETE")
            .allowedHeaders("header1", "header2", "header3")
            .exposedHeaders("header1", "header2")
            .allowCredentials(true).maxAge(3600);

        // Add more mappings...
    }
}
```

#### XML Configuration
XML 네임스페이스로 CORS를 허용하려면, `<mvc:cors>`요소를 사용해야 한다.

```xml
<mvc:cors>

    <mvc:mapping path="/api/**"
        allowed-origins="https://domain1.com, https://domain2.com"
        allowed-methods="GET, PUT"
        allowed-headers="header1, header2, header3"
        exposed-headers="header1, header2" allow-credentials="true"
        max-age="123" />

    <mvc:mapping path="/resources/**"
        allowed-origins="https://domain1.com" />

</mvc:cors>
```


### 1.7.5 CORS Filter
CORS는 컨트롤러단에서 말고도 `CORSFilter`라는 필터를 통해서도 적용 가능하다.  
필터를 설정하기 위해서는 `CorsConfigurationSource`를 생성자에게 전달해야 한다.

```java
CorsConfiguration config = new CorsConfiguration();

// Possibly...
// config.applyPermitDefaultValues()

config.setAllowCredentials(true);
config.addAllowedOrigin("https://domain1.com");
config.addAllowedHeader("*");
config.addAllowedMethod("*");

UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
source.registerCorsConfiguration("/**", config);

CorsFilter filter = new CorsFilter(source);
```


***
참고자료

1. [1.7 CORS](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-cors)
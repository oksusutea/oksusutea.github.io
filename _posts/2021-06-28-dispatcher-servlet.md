---
layout: post
title: DispatcherServlet의 기본 전략
categories: [spring]
tags: [java, spring]
description: 로드타임 위버 기능
fullview: false
comments: true
---

'21년 6월 28일 기준으로, 스프링 프레임워크는 5.3.8버전까지 나와 있다. 
기존 토비의 스프링을 보며 배웠던 DispatcherServlet의 기본 전략은 3.0버전을 기준으로 작성하였기 때문에, 현 시점과는 일치하지 않는 부분이 일부 있는 것으로 알고 있다.
이에 따라 현재 최신 버전(5.3.8)의 DispatcherServlet의 기본 전략에 대해 정리해보기로 하였다.

## DispatcherServlet
* DispatcherServlet은 클라이언트로부터 받는 모든 HTTP 요청을 프레젠테이션 계층 제일 앞단에 두어 중앙집중식으로 처리할 수 있는 프론트 컨트롤러이다.
* DispatcherServlet을 초기화 할 때, 이와 관련된 전략들을 함께 초기화 해준다. 초기화 해주는 전략들은 아래와 같다 : 


### HandlerMapping
* 핸들러 오브젝트(컨트롤러 메소드)를 찾기 위한 전략이다.
* BeanNameUrlHandlerMapping : HandlerMapping 인터페이스를 구현한 핸들러매핑 오브젝트 빈을 찾을 수 있도록 하는 전략. 요청 URL과 동일한 이름을 가진 컨트롤러 빈을 찾는다.
* RequestMappingHandlerMapping : @RequestMapping 어노테이션을 통해 등록한 핸들러를 찾을 수 있도록 하는 매핑 방식

### HandlerAdapter 
* 핸들러 매핑을 통해 찾은 핸들러 오브젝트를 실행해주는 전략이다.
* HttpRequestHandlerAdapter : HttpRequest 인터페이스를 구현한 핸들러를 실행할 수 있도록 하는 어댑터
* SimpleControllerHandlerAdapter :  Controller 인터페이스를 구현한 핸들러를 실행할 수 있도록 하는 어댑터
* RequestMappingHandlerAdapter : @RequestMapping 어노테이션을 통해 등록한 핸들러를 실행할 수 있도록 하는 어댑터

### HandlerExceptionResolver
* 특정 예외에 에러페이지를 매핑해주는 등 예외 처리를 위한 전략이다.
* ExceptionHandlerExceptionResolver : @ExceptionHandler 어노테이션이 달린 메소드를 예외처리 핸들러 빈으로 등록할 수 있도록 해주는 리졸버
* ResponseStatusExceptionResolver : @ResponseStatue 어노테이션을 통해 특정 HTTP 응답 코드의 예외 처리를 핸들링 할 수 있도록 해주는 리졸버 
* DefaultHandlerExceptionResolver : HandlerExceptionResoulver의 기본적인 구현체, HTTP 응답코드를 그에 상응하는 스프링 MVC 표준 예외로 변환해주는 리졸버

### ViewResolver
* 뷰 이름을 기반으로 실제 뷰 오브젝트를 찾을 수 있도록 하는 전략이다.
* InrernalResourceViewResolver : 컨트롤러가 지정한 이름 앞 뒤로 prefix, suffix를 붙여주고 해당 url에 맞는 뷰를 가져온다. (UrlBasedViewResolver를 상속하였다.)


### RequestToViewNameTranslator
* 뷰 오브젝트나 뷰 이름이 제공되지 않았을 때, 해당 요청을 응당하는 뷰 이름으로 변경해주는 전략이다.
* DefaultRequestToViewNameTranslator : 들어오는 Http 요청 url을 그대로 view 이름으로 변환해주는 translator다. 요청 url의 prefix, suffix를 제외하고 나머지만 뷰 이름으로 치환한다.
	* http://localhost:8080/gamecast/display.html => display
	* http://localhost:8080/gamecast/admin/index.html => admin/index

### MultipartResolver
* 아파치 공용 파일 업로드, 다운로드를 사용하기 위해 쓰는 전략이다.
* 기본 전략으로는 등록되어 있지 않다.

### LocaleResolver
* HTTP 요청 헤더, 쿠키 혹은 세션을 통해 클라이언트의 지역정보를 전달받을 수 있는 전략이다. Multipart 형식으로 데이터가 전송된 경우 이를 적절히 변환해준다.
* AcceptHeaderLocaleResolver : HTTP 요청 헤더의 "accept-language"에 담겨져 있는 primaty localre 값을 받아올 수 있도록 하는 리졸버다.
	* getLocale()만 가능하며, setLocale()은 불가능하다(클라이언트만 브라우저의 http 헤더 요청을 변경할 수 있으므로)

### ThemeResolver
* Theme을 지원하기 위한 전략이다.
* FixedThemeResolver : defaultThemeName 프로퍼티를 통해 뷰의 고정된 theme을 지정해줄 수 있도록 하는 resolver이다.
	* 고정된 테마는 변경되지 않기 때문에,setThemeName은 지원하지 않는다.

***
참고 자료 : 
[Class DispatcherServlet](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/servlet/DispatcherServlet.html)
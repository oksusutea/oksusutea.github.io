---
layout: post
title: DispatcherServlet과 MVC 아키텍쳐
categories: [spring]
tags: [java, spring]
description: Spring MVC
fullview: false
comments: true
---

## DispatcherServlet
* 이 서블릿은 스프링의 웹 기술을 구성하는 다양한 전략을 DI로 구성해서 확장하도록 만들어진 스프링 서블릿/MVC의 엔진과 같은 역할을 한다.
* 프론트 컨트롤러 역할을 한다.
	* 프론트 컨트롤러 패턴 : 중앙 집중형 컨트롤러를 프레젠테이션 계층의 맨 앞에 두어 서버로 들어오는 모든 요청을 먼저 받아서 처리하게 만든다.
	* 클라이언트의 모든 요청을 받아 공통적인 작업을 수행한 후에 적절한 세부 컨트롤러로 작업을 위임해주고, 클라이언트에게 보낼 뷰를 선택해 최종 결과를 생성하는 등의 작업을 수행한다.

## 클라이언트로부터 요청을 받아 결과를 응답하는 과정

### (1) DispatcherServlet의 요청 접수
* 자바 서버의 서블릿 컨테이너는 HTTP 프로토콜을 통해 들어오는 요청이 DispatcherServlet에 할당된 것이라면 요청정보를 DispatcherServlet에게 전달한다.

```xml
<servlet-mapping>
	<servlet-name>Spring MVC Dispatcher Servlet<Servlet-name>
	<url-pattern>/app/*</url-pattern>
</servlet-mapping>
```

* 이와 같이 등록하면 /app으로 시작하는 모든 요청을 스프링의 프론트 컨트롤러인 DispatcherServlet에게 할당한다.
* DispatcherServlet은 모든 요청에대해 공통적으로 진행해야 하는 전처리 작업(로깅, 보안, 파라미터 조작, 인코딩 등)이 있다면 이를 먼저 수행한다.

### (2) DispatcherServlet에서 컨트롤러로 HTTP 요청 위임
* DispatcherServlet은 URL, HTTP 메소드 등을 참고해 어떤 컨트롤러에게 작업을 위임할지 결정한다.
* 이 때 핸들링 매핑 전략(=사용자 요청을 기준으로 어떤 핸들러에게 작업을 위임할지 결정)을 이용한다.
* 어떤 컨트롤러/핸들러가 요청을 처리하게 할지 결정했다면, 해당 컨트롤러 오브젝트의 메소드를 호출해 실제로 웹 요청을 처리하는 작업을 위임한다.
	* 특정 컨트롤러를 호출해야 할 때는 해당 컨트롤러 타입을 지원하는 어댑터를 중간에 껴서 호출하는 어댑터를 이용한다.
	* 이를 통해 DispatcherServlet은 항상 일정한 방식으로 컨트롤러를 호출하고 결과를 받을 수 있다.
	* 어떤 어댑터를 사용할지는 DispatcherServlet 전략의 하나인 핸들러 어댑터 전략을 통해 결정된다.

### (3) 컨트롤러의 모델 생성과 정보 등록
* 컨트롤러의 작업은 먼저 사용자 요청을 해석하는 것, 비즈니스 로직 수행을 위해 서비스 계층 오브젝트에게 작업을 위임하는 것, 결과를 받아 모델을 생성하는 것, 마지막으로 어떤 뷰를 사용할지 결정하는 것 네가지로 분류 할 수 있다.
* 모델을 생성하고 모델에 정보를 넣어주는게 컨트롤러가 해야 할 마지막 중요할 두가지 작업이다.
* 모델은 보통 맵에 담긴 정보라고 생각하면 된다.

### (4) 컨트롤러의 결과 리턴
* 컨트롤러가 뷰 오브젝트를 직접 리턴할 수도 있지만, 보통은 뷰의 논리적인 이름을 리턴해주면 DispatcherServlet의 전략인 뷰 리졸버가 이를 이용해 뷰 오브젝트를 생성해준다.
* 컨트롤러가 리턴해주는 정보는 결국 모델과 뷰 두가지이다. 스프링에는 ModelAndView를 통해 컨트롤러로부터 결과 값을 돌려받는다.
* 모델과 뷰를 넘기는 것으로 컨트롤러의 책임은 끝이다.

### (5) DispatcherServlet의 뷰 호출과 (6) 모델 참조
* 컨트롤러로부터 모델과 뷰를 받은 뒤에, DispatcherServlet는 뷰 오브젝트에게 모델을 전달해주고 클라이언트에게 돌려줄 최종 결과물을 생성해달라고 요청한다.
* 동적으로 생성되도록 표시된 부분은 모델의 내용을 참고해 내용을 채워준다.
* 뷰 작업을 통한 최종 결과물은 HttpServletResponse 오브젝트 안에 담긴다.

### (7) HTTP 응답 돌려주기
* 뷰 생성까지 마친 후, DispatcherServlet은 등록된 빈 후처리기가 있는지 확인하고, 있다면 후속 작업을 진행한 뒤에 뷰가 만들어준 HttpServletResponse의 최종 결과를 서블릿 컨테이너에게 돌려준다.
* 서블릿 컨테이너는 HttpServletResponse에 담긴 정보를 HTTP 응답으로 만들어 사용자의 브라우저나 클라이언트에게 전송하고 작업을 종료한다.

## DispatcherServlet의 DI 가능 전략

### HandlerMapping
* 요청 URL과 요청 정보를 기준으로 어떤 핸들러 오브젝트, 즉 컨트롤러를 사용할 것인지를 결정하는 로직을 담당한다.
* HandlerMapping 인터페이스를 구현하여 만들 수 있고 하나 이상의 핸들러 매핑을 가질 수 있다.
* 기본적으로 BeanNameUrlHandlerMapping과 DefaultAnnotationHandlerMapping 두 가지가 설정되어 있다.

### HandlerAdapter
* 핸들러 매핑으로 선택한 컨트롤러/핸들러를 IspatcherServlet이 호출할 때 사용하는 어댑터이다.
* 컨트롤러 타입에 적합한 어댑터를 가져다가 이를 통해 컨트롤러를 호출한다.
* 디폴트로 등록되어있는 핸들러 어댑터는 HttpReqeustHandlerAdapter, SimpleControllerHandlerAdapter, AnnotationMethodHandlerAdapter가 있다.


### HandlerExceptionResolver
* 예외가 발생했을 때 이를 처리하는 로직을 갖고 있다.
* 예외의 종류에 따라 에러 페이지를 표시하거나, 관리자에게 통보해주는 등의 작업은 개발 컨트롤러가 아니라 프론트 컨트롤러인 DispatcherServlet을 통해 처리돼야 한다.
* 디폴트 전략은 AnnotationMethodHandlerExceptionResolver, ResponseStatusExceptionResolver, DefaultHandlerExceptionResolver가 있다.

### ViewResolver
* 뷰 리졸버는 컨트롤러가 리턴한 뷰 이름을 참고해 적절한 뷰 오브젝트를 찾아주는 로직을 가진 전략 오브젝트다.
* 디폴트 전략은 InternalResourceViewResolver이다.


### LocalResolver
* 지역 정보를 결정해주는 전략이다.
* 디폴트 전략인 AcceptHeaderLocalResolver는 HTTP 헤더의 정보를 보고 지역정보를 설정해준다.
	* 이 전략을 바꾸면 HTTP 헤더 대신 세션이나 URL 파라미터, 쿠키 또는 XML 설정에 직접 지정한 값 등 다양한 방식으로 결정 할 수 있다.

### ThemeResolver
* 테마를 가지고 이를 변경해서 사이트를 구성할 경우 쓸 수 있는 테마 정보를 결정해주는 전략이다.

### RequestToViewNameTranslator
* 컨트롤러에서 뷰 이름이나 뷰 오브젝트를 제공해주지 않았을 경우 URL과 같은 요청정보를 참고해 자동으로 뷰 이름을 생성해주는 전략이다.
* 디폴트 전략은 DefaultRequestToViewNameTranslator이다.


#### 정리
* 지금까지 살펴본 전략은 모두 DispatcherServlet의 동작방식을 확장할 수 있도록 만들어진 확장 포인트라고 볼 수 있다.
* DispatcherServlet은 각 전략의 디폴트 설정을 DispatcherServlet.properties라는 전략 설정파일로부터 가져와 초기화한다.
* DispatcherServlet은 서블릿 컨테이너가 생성하고 관리하는 오브젝트이지, 스프링 컨테이너에서 관리하는 빈 오브젝트가 아니다. 따라서 DispatcherServlet에 직접 DI 설정을 해줄 수 있는 방법은 없다.

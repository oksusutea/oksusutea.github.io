---
layout: post
title: Controller
categories: [spring]
tags: [java, spring]
description: Spring MVC의 Controller
fullview: false
comments: true
---


## 컨트롤러
* 컨트롤러는 MVC의 세가지 컴포넌트 중에서 가장 많은 책임을 지고 있다.
* 비즈니스 로직 처리는 서비스 계층에 위임한다고 하더라고, 컨트롤러가 직접 해야 할 작업이 적지 않다.
* 서블릿이 넘겨주는 HTTP 요청은 HttpServletRequest 오브젝트에 담겨져 있다.
	* 클라이언트 호스트, 포트, URI, 쿼리 스트링, 폼 파라미터, 쿠키, 헤더,세션 및 서블릿 컨테이너가 요청 애트리뷰트로 전달해주는 것까지 매우 다양한 정보를 참고해야 한다.
* 컨트롤러는 다양한 검증조건을 이용해 필수 항목이 빠지지는 않았는지, 바른 형식을 갖췄는지, 심지어는 서비스 계층에 요청해 DB의 정보와 비교하는 검증을 한다.


## 컨트롤러의 종류와 핸들러 어댑터
* 스프링 MVC가 지원하는 컨트롤러는 4가지 종류가 있다.
* 각 컨트롤러를 DispatcherServlet에 연결해주는 핸들러 어댑터가 하나씩 있어야 하므로, 핸들러 어댑터도 4개다.

### Servelt과 SimpleServletHandlerAdapter
* 표준 서블릿이다.
* 서블릿을 컨트롤러로 사용했을 때의 장점은, 서블릿 클래스 코드를 유지하며 스프링 빈으로 등록된다는 점이다.
* 서블릿이 컨트롤러 빈으로 등록된 경우에는 자동으로 init(), destroy()와 같은 생명주기 메소드가 호출되지 않는다.
	* 이를 대비하기 위해 <bean>태그의 init-method 애트리뷰트나, @PostConstruct 애노테이션을 이용해 처리해주어야 한다.
* 서블릿 컨테이너용 핸들러 어댑터는 SimpleServletHandlerAdapter이다.
* Servlet타입의 컨트롤러는 모델과 뷰를 리턴하지 않는다.
	* DispatcherServlet은 컨트롤러가 ModelAndView가 아닌 null을 리턴하면 뷰를 호출하는 과정을 생략하고 작업을 마친다.

### HttpRequestHandler와 HttpRequestHandlerAdapter
* HttpRequestHandler는 서블릿처럼 동작하는 컨트롤러를 만들기 위해 사용한다.
* HttpRequestHandler는 모델과 뷰 개념이 없는 HTTP 기반의 RMI와 같은 로우레벨 서비스를 개발할 때 이용한다.
	* RMI : Remote Method Invocation의 약자로, 원격 메소드 호출을 의미한다.

### Controller와 SimpleControllerHandlerAdapter
* Controller 컨트롤러는 DispatcherServlet이 컨트롤러와 주고받는 정보를 그대로 메소드의 파라미터와 리턴 값으로 갖고 있다.
* 스프링 MVC의 가장 대표적인 컨트롤러 타입이다.

```java
public interface Controller {
	ModelAndVide handleRequest(HttpServletRequest request, HttpServletResponse response) throws Exception;
}
```	

* 실제로는 이 Controller 인터페이스를 직접 구현해 컨트롤러를 만든느 것은 권장되지 않으며, AbstractController를 상속해서 컨트롤러를 만드는 것이 편리하다.
* AbstractControlle는 컨트롤러로서의 필스 기능이 구현되어있다. 해당 클래스에 관련되어있는 프로퍼티는 아래와 같다 : 
	* synchronizeOnSession : HTTP 세션에 대한 동기화 여부 사용자가 자신의 HTTP 세션에 동시에 접근하는 것을 막아준다.
	* supportedMethods:  컨트롤러가 허용하는 HTTP 메소드를 지정할 수 있다.
	* useExpireHeaders, useCacheControlHeader, useCacheControlNoStore, cacheSeconds : HTTP 헤더를 이용해 브라우저의 캐시 정보를 보내줄 것인지를 결정한다.

### AnnotationMethodHandlerAdapter
* 다른 핸들러 어댑터와 가장 다른 특징은 지원하는 컨트롤러의 타입이 정해져 있지 않다는 점이다.
* 다른 핸들러 어댑터는 특정 인터페이스를 구현한 컨트롤러만 지원하지만, 이 어댑터는 컨트롤러 타입에 제한이 없다.
* 그 뿐만 아니라 컨트롤러 하나가 하나 이상의 URL에 맵핑될 수 있다. 이로 인해 URL 매핑을 커컨트롤러 단위가 메소드 단위로 가능하게 했다.
* 이 어댑터는 DefaultAnnotationHandlerMaping 핸들러 매핑과 함께 사용해야 한다.

***

## 핸들러 매핑
* 핸들러 매핑은 HTTP 요청정보를 토대로 이를 처리할 핸들러 오브젝트, 즉 컨트롤러를 찾아주는 기능을 가진 DispatcherServlet의 전략이다.
* 스프링은 기본적으로 다섯가지 핸들러 매핑을 제공한다.

### BeanNameUrlHandlerMapping
* 디폴트 핸들러 매핑의 하나다.
* 빈의 이름에 들어있는 URL을 HTTP 요청의 URL과 비교해 일치하는 빈을 찾아준다.
* 사용하기 간편하지만 컨트롤러의 개수가 많아지면 URL 정보가 XML 빈 선언이나 클래스의 애노테이션 등에 분상되어 나타나 전체적인 매핑구조를 한 눈에 파악하기 힘들다.

### ControllerBeanNameHandlerMapping
* 빈의 아이디나 이름을 이용해 매핑해주는 핸들러 매핑 전략이다.
* XML에 정의하는 경우 <bean>의 id 애트리뷰트에 사용할 수 있는 문자의 제한이 있어 URL의 시작 기호인 /는 사용할 수 없다.
* 특정 클래스를 전략 빈으로 등록할 경우 디폴트 전략은 모두 무시된다는 점을 유의하자.

### ControllerClassNameHandlerMapping
* 빈 이름대신 클래스 이름을 URL에 맵핑해주는 클래스이다.
* 기본적으로 클래스 이름을 모두 URL로 사용하지만, Controller로 끝나면 Controller를 뺀 나머지 이름을 URL에 맵핑해준다.

### SimpleUrlHandlerMapping
* 빈 이름에 매핑정보를 넣어 관리한다.
* URL과 컨트롤러의 매핑정보를 한곳에 모아놓을 수 있는 핸들러 매핑 전략이다.
* 매핑정보가 한 곳에 모여있어 URL을 관리하기 편하다는 장점이 있다. 하지만 xml파일로 관리하게 될 경우 오타등이 발생할 수 있고, 누락될 가능성도 있다.

```xml
<bean class="org.springframework.web.servlet.handler.SimpleUrlHandlerMapping">
	<property name="mapping">
		<props>
			<prop key="/hello">helloController</prop>
			<prop key="/sub/*">myController</prop>
			<prop key="/deep/**/sub">subController</prop>
		</props>
	</property>
</bean>

<bean id="helloController" ...>
<bean id="myController" ...>
<bean id="subController" ...>
```

### DefaultAnnotationHandlerMapping
* @RequetMapping이라는 애노테이션을 컨트롤러 클래스 혹은 메소드에 직접 부여하고 이를 이용해 매핑하는 전략이다.
* 메소드 종류, 심지어 파라미터와 HTTP 헤더 정보도 매핑에 활용할 수 있다.

### 기타 공통 설정정보
* 핸들러 매핑 전략은 매핑방식을 매우 세밀하게 제어할 수 있는 다양한 프로퍼티를 제공한다.
* 공통적으로 사용되는 주요 프로퍼티는 아래와 같다 : 

#### order
* 핸들러 매핑은 한 개 이상을 동시에 사용할 수 있다.
* URL 매핑정보가 중복될 경우 핸들러 매핑의 우선순위를 둘 수 있다.
* 핸들러 매핑은 모두 Ordered 인터페이스를 구현하고 있어 우선순위를 적용 할 수 있다.

#### defaultHandler
* 핸들러 매핑 빈의 defaultHandler를 지정해두면 URL을 매핑할 대상을 찾지 못했을 경우 자동으로 디폴트 핸들러를 선택해준다.
* URL을 매핑할 컨트롤러를 찾지 못하면 404 에러가 발생한다. 404 에러를 돌려주는 대신 안내 메시지를 뿌려주는 것이 좋은 방법이니 해당 프로퍼티를 사용해보도록 하자.

#### alwaysUseFullPath
* URL 매핑은 기본적으로 웹 애플리케이션의 컨텍스트 패스와 서블릿 패스 두 가지를 제외한 나머지만 가지고 비교한다.
* 예를들어 웹 애플리케이션은 /sub에 배포되었고, DispatcherServlet은 /app/*으로 설정하였다고 가정하였을 경우, /hello라는 URL에 접근하기 위해서는 브라우저에 URL을 /sub/app/hello라고 적어야 한다. 여기서는 웹 애플리케이션의 컨텍스트 패스와 서블릿 패스 두가지를 제외한 /hello가 핸들러 매핑의 URL과 비교된다.
* 웹 애플리케이션의 배치 경로나 서블릿 매핑 변경에 영향이 가지 않기 위해 디폴트 값은 상대경로로 지정되어있다.
* 하지만 이렇게 상대경로로 사용하는 것이 아니라, 절대경로로 지정하고 싶을 경우 해당 property를 true로 설정해주면 된다.

#### detectHandlersInAncestorContexts
* 핸들러 매핑 클래스는 기본적으로 현재 컨텍스트, 즉 서블릿 컨텍스트 안에서만 매핑할 컨트롤러를 찾는다. 해당 프로퍼티 값이 false로 선언되어 있기 때문이다.
* 웬만하면 해당 프로퍼티 값은 false로 유지해주자. 웹 환경에 종속적인 컨트롤러 빈은 서블릿 컨텍스트에만 두는 것이 바람직하기 때문이다.

***

## 핸들러 인터셉터

* 핸들러 매핑의 역할은 URL과 요청정보로부터 컨트롤러 빈을 찾아주는 것이다.
* 그 외에도 한가지 중요한 기능이 더 있다. 핸들러 인터셉터를 적용해주는 것이다.
* 핸들러 인터셉터는 DispatcherServlet이 컨트롤러를 호출하기 전과 후에 요청과 응답을 참조하거나 가공할 수 있는 일종의 필터이다.
* 핸들러 인터셉터는 서블릿 필터와 유사하다. 하지만 핸들러 인터셉터는 HttpServletRequest, HttpServletResponse 뿐 아니라 실행될 컨트롤러 빈 오브젝트, 컨트롤러가 돌려주는 ModelAndView, 발생한 예외 등을 제공받을 수 있기 때문에 서블릿 필터보다 더 정교하고 세밀하게 인터셉터를 만들 수 있다. 또한 핸들러 인터셉터 자체가 빈이기 때문에 DI를 통해 다른 빈을 활용할 수 있다.

### HandlerInterceptor
* 핸들러 인터셉터는 HandlerInterceptor를 구현해서 만든다. 이 인터페이스는 세 개의 메소드가 포함되어 있다.

#### boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception
* 메소드 컨트롤러가 호출되기 전에 실행된다.
* handler 파라미터는 핸들러 매핑이 찾아준 컨트롤러 빈 오브젝트다.
* 리턴값이 true면 핸들러 실행 체인의 다음 단계로 넘어가고, false이면 작업을 중단하고 리턴하므로 컨트롤러와 남은 인터셉터는 실행하지 않는다.


#### boolean postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception
* 메소드 컨트롤러를 실행하고 난 후에 호출된다.
* 일종의 후처리 작업을 진행할 수 있다.

#### boolean afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception
* 모든 뷰에서 최종 결과를 생성하는 일 포함 모든 작업이 완료된 후에 실행한다.
* 요청 처리 중에 사용한 리소스를 반환해주기 적당하다.


* 핸들러 인터셉터는 하나 이상 등록할 수 있다.
* preHandle()은 등록된 순서대로 실행되고, 나머지는 등록된 순서의 역순으로 실행된다.




### 핸들러 인터셉터 적용
* 핸들러 인터셉터 적용을 위해 핸들러 매핑 클래스를 빈으로 등록해야 한다.
* interceptors 프로퍼티를 이용해 핸들러 인터셉터 빈의 레퍼런스를 넣어주면 된다.
* 핸들러 인터셉터는 기본적으로 핸들러 매핑 단위로 등록된다.

#### 서플릿 필터 vs 핸들러 인터셉터 vs AOP
* 서플릿 필터는 web.xml에 별도로 등록해주어야 하고 필터 자체는 스프링 빈이 아니다. 하지만 웹 애플리케이션으로 들어오는 모든 요청에 적용된다.
* 핸들러 인터셉터는 적용 대상이 DispathcerServlet의 특정 핸들러 매핑으로 제한되지만, 인터셉터를 빈으로 등록할 수 있고, 컨트롤러 오브젝트에 접근 가능하며, ModelAndView와 같은 컨트롤러가 리턴하는 정보도 활용할 수 있다. 그만큼 스프링 애플리케이션에 특화된 기능을 적용하기에 유리하다.
* AOP를 이용해 컨트롤러에 일괄 적용할 부가기능을 어드바이스로 만들어 적용할 수 있지만 권장하지 않는다.
	* 컨트롤러의 타입이 하나로 정해져 있지 않고,
	* 실행 메소드 또한 제각각이기 때문에 적용할 메소드를 선정하는 포인트컷 작성도 쉽지않다.
	* 또한 파라미터나 리턴 값 또한 일정치 않다.
* 스프링 MVC는 모든 컨트롤러에게 동일한 핸들러 인터셉터를 적용할 수 있기 때문에 컨트롤러에 공통적으로 적용할 부가기능은 핸들러 인터셉터를 이용하는 편이 낫다.

***

## 컨트롤러 확장
* 새로운 컨트롤러를 직접 설계해서 적용하고 싶을 수 있다. 이에 대한 방법을 알아보자.
* 앞서 말한 Controller 인터페이스를 구현해 공통 클래스를 만들 수 있다.
	* 하지만 개별 컨트롤러가 특정 클래스를 상속하도록 강제한다는 단점이 있다.
* 이런 경우에는 핸들러 어댑터를 직접 구현해서 아예 새로운 컨트롤러 타입을 도입하는 방법이 나을 수 있다.

```java
package org.springframework.web.servlet;

public interface HandlerAdapter {
	boolean supports(Object handler);	// 이 핸들러가 지원하는 핸들러(컨트롤러) 타입인지 확인
	ModelAndView handle(HttpServletRequest request, HttpServletResponse respone, Object handler) throws Exception;	// 컨트롤러를 실행해주는 메소드
	long getLastModified(HttpServletRequest request, Obejct handler);	//HttpServlet의 getLastModified()를 지원해주는 메소드
}
```

Adapter 인터페이스 구현 클래스 : 

```java
public class SimpleHandlerAdapter implements HandlerAdapter {
	public boolean supports(Object handler) {
		return ( handler instanceof SimpleController);
	}
	public long getLasModified(HttpServletRequest request, Obejct handler) {
		return -1;	// 캐싱을 적용하지 않으려면 0보다 작은 값을 리턴한다.
	}
	
	public ModelAndView handle(HttpServletRequest request, HttpServletResponse respone, Object handler) throws Exception {
		Method m = ReflectionUtils.findMethod(handler.getClass(),"control", Map.class, Map.class);
		ViewName viewName = AnnotationUtils.getAnnotation(m, ViewName.class);
		RequiredParams requiredParams = AnnotationUtils.getAnnotation(m, RequiredParams.class);
		
		Map<String, String> params = new HashMap<String, String>();
		for( String param : requiredParams.value()) {
			String value = req.getParameter(param);
			if(value == null) throw new IllegalStateException() ;
			param.put(param, value);
		}
		
		Map<String, Object> model = new HashMap<String, Obejct> ();
		((SimpleController)handler).control(params, model);
		
		return new ModelAndView(viewName.value(), model);
	}
}
```
 컨트롤러 인터페이스 : 
 
```java
public interface SimpleController {
	void control(Map<String, String> params, Map<String, Obejct> model);
}
```
구현 컨트롤러 클래스 : 

```java
public HelloController implements SimpleController {
	@ViewName("/WEB-INF/view/hello.jsp")
	@RequiredParams({"name"})
	public void control(Map<String, String> params, Map<String, Object> model) {
		model.put("message", "Hello " + params.get("name"));
	}
}
```
관련 인터페이스 : 

```java
@Retention(RetentionPolicy.RUNTIME)
@Inherited
public @interface ViewName {
	String value();
}

@Retention(RetentionPolicy.RUNTIME)
@Inherited
public @interface RequiredParams {
	String[] value();
}
```


***
참고자료 :  

* [토비의 스프링 3.1 Vol.2 3.3 - 컨트롤러](https://www.aladin.co.kr/shop/wproduct.aspx?ItemId=19505671)





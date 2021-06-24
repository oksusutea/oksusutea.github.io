---
layout: post
title: MVC 네임스페이스
categories: [spring]
tags: [java, spring, 토비의 스프링]
description: 웹서비스 개발시 사용하는 메시지 컨버터
fullview: false
comments: true
---

* 스프링 3.0에서 대거 도입된 @MVC 관련 기능을 제대로 활용하기 위해 디폴트로 등록되는 AnnotationMethodHandlerAdapter의 설정으로는 부족하다.
* 애노테이션 방식의 포맷터를 사용하는 바인딩 기능이나, JSR-303의 빈 검증 기능, 또 메시지 컨버터를 제대로 활용하려면 AnnotationMethodHandlerAdapter를 빈으로 등록하고 여러 가지 프로퍼티 설정을 해줄 필요가 있다.
* 스프링은 최신 @MVC 기능을 손쉽게 등록할 수 있게 해주는 mvc 스키마의 전용 태그를 제공한다.


	
## mvc 스키마 태그 
***

## <mvc:annotation-driven>
* 이 태그는 애노테이션 방식의 컨트롤러 사용시 필요한 DispatcherServlet 전략 빈을 자동으로 등록해준다.
* 그 외에도 최신 @MVC 지원 기능을 제공하는 빈도 함께 등록하고 전략 빈의 프로퍼티로 설정해준다.
* 라이브러리의 존재 여부를 파악해서 자동으로 관련 빈을 추가해주는 기능도 제공한다.
* 매우 빠르게 @MVC의 주요 빈을 설정해주지만, 검증기와 컨버전 서비스를 제외하면 기본적으로 등록되는 빈의 설정을 변경할 수 없다.
* <mvc:annotation-driven> 태그를 통해 자동으로 등록되는 빈 정보는 아래와 같다 : 


### DefaultAnnotationHandlerMapping
* 가장 우선으로 적용되도록 @ReauestMapping을 이용한 핸들러 매핑 전략을 등록한다.
* 다른 디폴트 해들러 매핑 전략은 자동등록되지 않는다.

### AnnotationMethodHandlerAdapter
* 디폴트 핸들러 어댑터다.
* 다른 핸들러 어댑터 전략은 자동등록되지 않는다.

### ConfigurableWebBindingInitializer
* 모든 컨트롤러 메소드에 자동으로 적용되는 WebDataBinder 초기화용 빈을 등록하고 AnnotationMethodHandlerAdapter의 프로퍼티로 연결해준다.

### 메시지 컨버터
* AnnotationMethodHandlerAdapter의 messageConverts 프로퍼티로 메시지 컨버터들이 등록된다.
* 네 개의 디폴트 메시지 컨버터와 함께 Jaxb2RootElementHttpMessageConverter와 MappingJacksonHttpMessageConverter가 추가로 등록된다.(JAXB2, Jackson라이브러리가 클래스패스에 존재하는 경우에만)

### <spring:eval>을 위한 컨버전 서비스 노출용 인터셉터
* <spring:eval> 은 기본적으로 표준 컨버터를 이용해 모델의 프로퍼티 값을 JSP에 출력할 문자열로 변환한다.

### validator
* 자동등록되는 ConfigurableWebBindingInitializer의 validator 프로퍼티에 적용할 Validator 타입의 빈을 지정할 수 있다.
* 모든 컨테이너에 일괄 적용하는 검증기는 디폴트로 추가되는 JSR-303 방식의 LocalValidatorFactoryBean이면 충분하다.

```xml
<mvc:annotation-driven validator="myValidator"    />
<bean id="myValidator" class="MyLocalValidatorFactoryBean">
	// property 설정
</bean>
```

### conversion-service
* ConfigurableWebBindingInitializer의 conversionService 프로퍼티에 설정되는 빈을 직접 지정할 수 있다.
* 직접 개발한 컨버터나 포맷터를 적용하려면 FormattingConversionServiceFactoryBean을 빈으로 직접 등록하고 재구성해줘야 한다.

```xml
<mvc:annotation-driven conversion-service="myConversionService"    />
<bean id="myConversionService" class="FormattingConversionServiceFactoryBean">
	<property name="converters">
	...
	</property>
</bean>
```

## <mvc:interceptors>
* HandlerInterceptor의 적용 방법은 두가지가 있다 : 
	* 핸들러 매핑 빈의 interceptors 프로퍼티를 이용해 등록하는 방법
		* 단점 :
			*  1) 인터셉터 등록을 위해 핸들러 매핑 빈을 명시적으로 빈으로 선언해주어야 한다
			* 2) 핸들러 매핑이 여러개면, 인터셉터를 핸들러 매핑마다 반복해서 설정해주어야 한다.
	* <mvc:interceptors>를 이용하는 방법 
		* 장점 :
			*  1) 모든 핸들러 매핑에 일괄 적용되는 인터셉터를 한 번에 설정할 수 있다
			* 2) URL패턴 지정 가능

URL 지정 안할 경우 인터셉터 등록 방법 : 

```xml
<mvc:interceptors>
	<bean class="...MyInterceptor"	/>
</mvc:interceptors>
```

URL 지정시 인터셉터 등록 방법:


```xml
<mvc:interceptors>
	<mvc:interceptor>
		<mvc:mappint path="/admin/*"/>
		<bean class="...MyInterceptor"	/>
	</mvc:interceptor>
</mvc:interceptors>
```

### <mvc:view-controller>
* 컨트롤러가 모델 없이 뷰를 지정만 할 경우, 이런 기능을 위해 컨트롤러를 만드는 것은 비효율 적이다.
* 이 떄 <mvc:view-controller> 를 사용하면 손쉽게 해결 가능하다.
* 이 태그에 매핑할 URL 패턴과 뷰 이름을 넣어주면 해당 URL을 매핑하고 뷰이름을 리턴하는 컨트롤러가 자동으로 등록된다.

```xml
<mvc:view-controller path="/path" view-name="/index"    />
```

* <mvc:view-controller>를 하나라도 등록하면 SimpleUrlHandlerMapping과 SimpleControllerHandlerAdapter를 자동으로 등록한다.
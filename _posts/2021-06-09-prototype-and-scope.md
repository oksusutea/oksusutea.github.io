---
layout: post
title: 프로토타입과 스코프 
categories: [Spring]
tags: [Spring Framework, 토비의 스프링]
description: 빈의 유효 범위와 싱글톤, 프로토타입
fullview: false
comments: true
---

## 프로토타입 스코프
* 싱글톤 스코프는 컨텍스트당 한 개의 빈 오브젝트만 만들어지게 한다.
* 따라서 하나의 빈을 여러 개의 빈에서 DI하더라도 매번 동일한 오브젝트가 주입된다.
* 프로토타입 스코프는 컨테이너에게 빈을 요청할 때마다 매번 새로운 오브젝트를 생성해준다.

### 프로토타입 빈의 생명주기와 종속성
* IoC의 기본 개념은 애플리케이션을 구성하는 핵심 오브젝트를 코드가 아니라 컨테이너가 관리한다는 것이다.
* 프로토타입 빈은 독특하게 이 IoC의 원칙을 따르지 않는다.
	* 프로토타입 빈 오브젝트는 한번 DL, DI를 통해 컨테이너 밖으로 전달된 후, 더이상 스프링이 관리하지 않게 된다.
	* 이때부터는 DL, DI를 통해 전달받은 빈이 해당 빈 오브젝트를 관리하게 된다. 그렇기 때문에 빈 오브젝트의 관리는 전적으로 DI 받은 오브젝트에 달려있다.
	* 만약 DL방식으로 직접 컨테이너에 프로토타입 빈을 요청했다면, 그 요청한 코드가 유지되는 만큼 빈 오브젝트가 존재할 것이다.

### 프로토타입 빈의 용도
* 프로토타입 빈은 코드에서 new로 오브젝트를 생성하는 것을 대신하기 위해 사용된다.
* 오브젝트에 DI를 적용하려면 컨테이너가 오브젝트를 만들게 해야 한다. 즉 매번 새로운 오브젝트가 필요하면서 DI를 통해 다른 빈을 사용할 수 있어야 한다면 프로토타입 빈이 가장 적절한 선택이다.
* 프로토타입 빈은 @Scope("prototype") 애노테이션을 통해 설정할 수 있다.
* 데이터 중심의 설계와 개발 방식을 선호한다면 굳이 프로토타입 빈을 만들어도 되지 않지만, 좀 더 오브젝트 중심적이고 유연한 확장을 고려한다면 프로토타입 빈을 이용하는 것이 나을 것이다.
* 프로토타입 빈은 DI 될 대상이 여러 군데라면 각기 다른 오브젝트가 생성된다. 하지만 같은 컨트롤러에 매번 요청이 있을 때마다 새롭게 오브젝트가 만들어져야 하는 경우 적합하지 않다.
* 단순히 new키워드를 대신하기 위해 프로토타입 빈으로 사용하는 것은 적합하지 않다.
* 프로토타입 빈이 DI 방식으로 사용되는 경우는 매우 드물다.

### 프로토타입 빈의 DL 전략
* 프로토 타입 빈은 DL 방식을 사용해 매번 컨테이너에 빈을 요청해서 새로운 오브젝트를 받아 사용해야 한다.
* 프로토타입 빈의 DL 방식은 어려가지 방법을 통해 적용할 수 있다.

#### ApplicationContext, BeanFactory
* @Autowired나 @Resource를 이용해 ApplicationContext 또는 BeanFactory를 DI 받은 뒤 getBean() 메소드를 직접 호출해 빈을 가져오는 방법이다.
* 사용하기에는 간단하지만 코드에 스프링 API가 직접 등장한다는 단점이 있다.

#### ObjectFactory, ObjectFactoryCreatingFactoryBean
* 직접 애플리케이션 컨텍스트를 사용하지 않으려면 중간에 컨텍스트에 getBean()을 호출해주는 역할을 맡을 오브젝트를 두면 된다.
* 이 때 팩토리를 사용하는데, 이유는 오브젝트를 요구하면서 오브젝트를 어떻게 생성하거나 가져오는지에 대해서는 신경쓰지 않아도 되기 때문이다.
* 팩토리를 만들기 위해서는 빈으로 등록되어야 하기 때문에 스태틱 팩토리 메소드를 이용하는 것은 적절하지 않다.
* 팩토리 인터페이스를 직접 정의하고 이 인터페이스를 구현한 빈 클래스를 만들어서 해당 빈을 돌려주는 메소드를 직접 구현할 수도 있다.

```java
@Resource //ObjectFactory 타입은 여러 개 있을 수 있어 이름으로 빈을 지정하는 편이 낫다
private ObjectFactory<ServiceRequest> serviceRequestFactory;

public void serviceRequestFormSubmit(HttpServletRequest request) {
	ServiceRequest  serviceRequest = this.serviceRequestFactory.getObject();
	serviceRequest.setCustomerByCustomerNo(request.getParameter("custno"));
...
}
```

```java
@Configuration
public class ObjectFactoryConfig {
	@Bean
	public ObjectFactoryCreatingFactoryBean serviceRequestFactory() {
		ObjectFactoryCreatingFactoryBean factoryBean = new ObjectFactoryCreatingFactoryBean();
		factoryBean.setTargetBeanName("serviceRequest");
		return factoryBean;
	}
}
```

* ObjectFactory는 프로토타입 빈 뿐만 아니라 DL을 이용해 가져와야 하는 모든 케이스에 적용 할 수 있다.

#### ServiceLocateFactoryBean
* ObjectFactory가 단순하고 깔끔하지만 프레임워크의 인터페이스를 애플리케이션 코드에서 사용하는 것이 맘에 들지 않을 경우가 있다. 이 때 ServiceLocateFactoryBean를 사용하면 된다.
* DL방식으로 가져올 빈을 리턴하는 임의의 이름을 가진 메소드가 정의된 인터페이스만 있으면 된다.

```java
public interface ServiceRequestFactory {
	ServiceRequest getServiceFactory();
}
```

* 이렇게 정의한 인터페이스를 이용해 스프링의 ServiceLocateFactoryBean으로 등록해주면 된다.

```java
@Autowired
private ServiceRequestFactory serviceRequestFactory;

public void serviceRequestFormSubmit(HttpServletRequest request) {
	ServiceRequest serviceRequest = this.serviceRequestFactory.getServiceFactory();
	serviceRequest.setCustomerByCustomerNo(request.getParameter("custno"));
...
}
```

* 범용적으로 사용하는 ObjectFactory와 달리 ServiceRequest 전용으로 만든 인터페이스가 이 빈의 타입이 되기 때문에 @Autowired를 사용 할 수 있다.

#### 메소드 주입
* ApplicationContext는 스프링 API에 의존적이고, ObjectFactory나 ServiceLocateFactoryBean는 빈을 새로 추가해야 한다는 번거로움이 있다. 
* 이 두가지 단점을 모두 극복하기 위해 메소드 주입을 이용할 수 있다.
* 메소드 주입은 @Autowired를 메소드에 붙여 메소드 파라미터에 의해 DI하는 방식이 아니라 메소드 코드 자체를 주입하는 것을 말한다.
* 일정한 규칙을 따르는 추상 메소드를 작성해두면 ApplicationContext의 getBean() 메소드를 사용해서 새로운 프로토타입 빈을 가져오는 기능을 담당하는 메소드를 런타임 시에 추가해주는 기술이다.

```java
abstract public ServiceRequest getServiceRequest();

public void serviceRequestFormSubmit(HttpServletRequest request) {
	ServiceRequest serviceRequest = this.getServiceRequest();
	serviceRequest.setCustomerByCustomerNo(request.getParameter("custno"));
...
}
```

* 추상 메서드를 가졌으므로 당연히 클래스도 추상 클래스로 정의되어야 한다.
* 이 추상클래스를 확장해서 getServiceRequest()라는 추상 메소드를 주입해주도록 빈을 정의한다.

```xml
<bean id="serviceRequestController" class="...ServiceRequestController">
	<lookup-method name="getServiceRequest" bean="serviceRequest"    />
</bean>	
```

* lookup-method라는 태그의 name은 스프링이 구현해줄 추상 메소드의 이름이고, bean 애트리뷰트는 메소드에서 getBean()으로 가져올 빈의 이름이다.
* 이렇게 작성하면 스프링은 추상클래스를 상속해서 getServiceRequest() 메소드를 완성하고 상속한 클래스를 빈으로 등록해준다,
* 메소드 주입 방식은 그 자체로 스프링 API에 의존적이지 않아 스프링 외 환경에 가져다 사용할 수 있고, 컨테이너 도움 없이 단위 테스트를 작성 할 수 있다.

#### Provider<T>
* 가장 최근에 소개된 방법이다.
* ObjectFactory와 거의 유사하게 <T> 타입 파라미터와 get()이라는 팩토리 메소드를 가진 인터페이스이다.
* 팩토리 빈을 XML 혹은 @Configuration으로 정의해주지 않아도 ObjectFactory처럼 동작하기 때문에 손쉽게 사용할 수 있다.
* Provider 인터페이스를 @Inject, @Autowired, @Resource중 하나를 이용해 DI 되도록 지정해주면 스프링이 자동으로 Provider를 구현한 오브젝트를 생성해서 주입해준다.(오브젝트 팩토리 주입이라고 보면 된다)

```java

@Inject Provider<ServiceRequest> serviceRequestProvider;

public void serviceRequestFormSubmit(HttpServletRequest request) {
	ServiceRequest serviceRequest = this. serviceRequestProvider.get();
	serviceRequest.setCustomerByCustomerNo(request.getParameter("custno"));
...
}
```

* Provider는 javax.inject 안에 포함된 표준 인터페이스이기 때문에 스프링 API인 ObjectFactory를 사용할 때 보다 호환성이 좋다.
* 다른 프레임워크에서도 같은 방식으로 동작하는 것이 보장된다.


#### 정리
* 지금까지 프로토타입 빈을 DL 방식으로 사용하는 데 쓸 수 있는 전략을 살펴봤다. 가급적 ApplicationContext같은 무거운 스프링 컨테이너 API를 피하고, Provider를 사용하자.
* 그 외 Provider를 사용 할 수 없다면 스프링 API 의존성 여부, 팩토리 인터페이스 존재여부, 추상 클래스 및 단위 테스트 문제를 고려해 ObjectFactory, ServiceLocateFactoryBean, 메소드 주입 중에 적절히 선택하면 된다.
* 해당 방식은 프로토타입 외에도 DL 방식이 코드 내에 필요한 경우라면 적용 가능하다.


***
## 스코프
* 스프링은 싱글톤, 프로토타입 외에 요청, 세션, 글로벌세션, 애플리케이션이라는 네 가지 스코프를 기본적으로 제공한다.
* 모두 웹 환경에서만 의미 있다.
* 네 가지 스코프 중 application 외 나머지 세 가지 스코프는 싱글톤과 다르게 독립적인 상태를 저장해두고 사용하는데 필요하다. 서버환경이기 때문이다.

### 요청 스코프
* 요청 스코프 빈은 하나의 웹 요청 안에서 만들어지고 해당 요청이 끝날 때 제거된다.
* 각 요청별로 독립적인 빈이 만들어지기 때문에 상태 값을 저장해둬도 안전하다.
* 존재 범위와 특징은 하나의 요청이 일어나는 메소드 파라미터로 전달되는 도메인 오브젝트나 DTO와 비슷하다.
* 주로 애플리케이션 코드에서 생성한 정보를 프레임워크 레벨의 서비스나 인터셉트 등에 전달할 때 사용한다. 또는 애플리케이션 코드가 호출되기 전에 프레임워크나 인터셉트 등에서 생성한 정보를 이용할 때에도 사용된다.
	* 보안 프레임 워크에서 현재 요청에 관련된 권한 정보를 요청 스코프 빈에 저장해뒀다가 필요한 빈에서 참조

### 세션 스코프, 글로벌세션 스코프
* HTTP 세션과 같은 존재 범위를 갖는 빈으로 만들어주는 스코프이다.
* HTTP 세션은 사용자별로 만들어지고 브라우저를 닫거나 세션 타임이 종료될 때까지 유지되기 때문에 로그인 정보나 사용자별 선택 옵션등을 저장해두기에 유용하다.
* 글로벌세션 스코프는 포틀릿에만 존재하는 글로벌 세션에 저장되는 빈이다.

### 애플리케이션 스코프
* 서블릿 컨텍스트에 저장되는 빈 스코프이다. 서블릿 컨텍스트는 웹 애플리케이션ㄴ마다 만들어진다.
* 싱글톤 스코프와 다른 점?
	* 웹 애플리케이션과 애플리케이션 컨텍스트 컨텍스트의 존재 범위가 다른 경우가 있기 때문이다.
	* 웹 애플리케이션 밖에서 더 오래 존재하는 컨텍스트도 있고, 더 짧은 시간 동안 존재하는 서블릿 레벨의 컨텍스트도 있다.
* 싱글톤과 마찬가지로 stateless하게 관리하거나, 멀티스레드 환경에서 안전하도록 만들어야 한다.

## 스코프 빈의 사용 방법
* 애플리케이션 스코프 빈 제외, 요청 스코프, 세션 스코프, 글로벌세션 스코프 빈은 매 번 요청시마다 새로운 빈이 생성된다.
* 프로토타입 빈과는 다르게 스프링이 생성부터 초기화, DI, DL, 그리고 제거까지 전 과정을 관리한다.
* 하지만 프로토타입 빈과 마찬가지로 하나 이상의 오브젝트가 만들어져야 하기 때문에 싱글톤에 DI 해주는 방법으로 사용할 수 었다.
* 보통 Provider나 ObjectFactory와 같은 DL 방식을 사용한다.
* 일반 방식으로 DI는 불가능 하지만, 그 대신 스코프 빈에 대한 프록시를 DI 해줌으로써 평범한 싱글톤 빈을 이용하듯이 스코프 빈을 쓸 수 있다.

### DL 방식 

```java
@Scope("session")
public class LoginUser {
	String loginId;
	String name;
	Date loginTime;
	.....
}
```

```java
public class LoginService {
	@Autowired Provider<LoginUser> loginUserProvider; // DL방식으로 접근 가능하도록 Provider로 DI 받는다.
	
	public void login(Login login) {
		LoginUser loginUser = loginUserProvicer.get();	// 같은 사용자의 세션 안에는 매번 같은 오브젝트를 가져온다.
		loginUser.setLoginId(...);
		loginUser.setName(...);
		loginUser.setLoginTime(new Date());
	}
}
```

### DI 방식
* @Scope 애노테이션으로 스코프를 지정했다면 proxyMode 엘리먼트를 이용해서 프록시를 이용한 DI가 되도록 지정할 수 있다.
* 클라이언트는 스코프 프록시 오브젝트를 실제 스코프 빈처럼 사용하고, 프록시에서 현재 스코프에 맞는 실제 빈 오브젝트로 작업을 위임한다.
* 앞서 작성한 방식에서 클라이언트는 싱글톤으로 만든 LoginService 빈이다. 싱글톤 빈이기 때문에 LoginUser같은 세션 스코프 빈을 DI 받더라도 하나의 오브젝트 밖에 할당이 안된다.
* 스코프 프록시를 통해 새로운 오브젝트 빈을 주입 받을 수 있다. 스코프 프록시는 각 요청에 연결된 HTTP 세션 정보를 참고해서 사용자마다 다른 LoginUser 오브젝트를 사용하게 해준다.

<p style="text-align:center">
<img src="https://user-images.githubusercontent.com/75205849/121364186-8634e400-c972-11eb-971a-377d2436dc93.jpeg">
</p>

* LoginService 입장에서는 모두 같은 오브젝트를 사용하는 것처럼 보이지만, 실제로는 그 뒤에 사용자별로 만들어진 여러 개의 LoginUser가 존재한다.
* 스코프 프록시는 실제 LoginUser 오브젝트로 클라이언트 호출을 위임한다.
* 스코프 프록시는 프록시 패턴을 활용하였다. 다만 지연된 로딩, 원격 오브젝트 접속을 위해 사용하는 것이 아닌, 스코프에 따라 다른 오브젝트를 사용하게 해주는 독특한 목적을 위해 사용한다.

```java
@Scope(value="session", proxyMode=ScopedProxyMode.TARGET_CLASS)
public class LoginUser {
.....
```

```java
public class LoginService {
	@Autowired LoginUser loginUser;
	
	public void login(Login login) {
		//로그인 처리
		this.loginUser.setLoginId(...); 	// 세션이 다르면 다른 오브젝트의 메소드가 호출된다.
		
.....
```

* 프록시 방식의 DI를 적용하면 스코프 빈이지만 싱글톤 빈처럼 사용할 수 있다. 하지만 빈의 스코프를 모르면 이해하기 어려울 수 있다.

### 커스텀 스코프와 상태를 저장하는 빈 사용하기
* 스프링은 기본적으로 제공하는 스코프 외에도 임의의 스코프를 만들어 사용할 수 있다.
* 싱글톤 외 스코프를 사용한다는 건 기본적으로 빈에 상태를 저장해두고 사용한다는 의미다.
* 스프링에서는 Scope 인터페이스를 구현해 새로운 스코프를 작성 할 수 있다. 스프링 웹플로우나 제이보스 씸같은 프레임워크를 이용하면 Scope 인터페이스를 구현한 다양한 커스텀 스코프를 제공해준다.
* 빈의 상태를 저장해두는 방식을 선호하지 않는다면, 싱클톤 빈을 사용하고 URL 파라미터, 쿠키, 폼 히든 필드, DB, HTTP 세션 등에 분산해 상태정보를 저장하고 코드로 관리할 수 있다.

***
참고자료 :  
[토비의 스프링 3.1 Vol.2 1.3- 프로토타입과 스코프](https://www.aladin.co.kr/shop/wproduct.aspx?ItemId=19505671)

---
layout: post
title: 기타 빈 설정 메타 정보 
categories: [Spring]
tags: [Spring Framework, 토비의 스프링]
description: 빈 메타 정보 관련 추가 학습
fullview: false
comments: true
---

## 빈 이름

### XML 설정에서의 빈 식별자와 별칭
* 빈 아이디와 빈 이름과 같이 특정 빈을 구분해서 가르키기 위한 것을 빈 식별자라고 한다.
* 빈은 하나 또는 그 이상의 식별자를 가질 수 있으며, 식별자는 해당 애플리케이션 컨텍스트에서 고유해야 한다.
* 빈의 식별자는 XML이라면 <bean> 태그의 id와 name 두 가지 애트리뷰트를 이용해 정의할 수 있다.

#### id
* id는 문서 전체에서 고유해야 하며 아래 규칙을 따라야 한다.
	* 공백 불가
	* 첫 글자는 알파벳, 밑줄(_) 그리고 허용된 일부 언어 문자만 사용 가능
	* 나머지 글자는 알파벳, 밑줄(_) 그리고 숫자와 점(.)을 허용한다. 특수문자는 사용 불가하다.
* id는 생략 가능하다. 스프링 컨테이너가 자동으로 빈의 아이디를 부여해준다.

```xml
<bean id="_hello.Service123" class="...">
<bean id="한글도 됩니다" class="...">
<bean id="userService" class="...UserService"> //관례적인 사용 방법 타입 이름의 첫글자만 소문자 처리
```

#### name
* id와는 달리 name은 특별한 제약이 없다.
* 여러 개의 이름을 지정할 수 있으며, 콤마(,) 또는 세미콜론(;)을 이용해 각 이름을 구분해준다.
* 같은 빈이지만 다른 이름으로 참조하면 편리할 때 name을 여러개로 사용한다.

```xml
<bean name="1234,/hello;헬로우" class="...">	//3 개의 name 값을 가진다.
```

### 애노테이션에서의 빈 이름
* 보통 클래스 이름을 그대로 빈 이름으로 사용하는 방법을 선호한다.

```java
@Component
public class UserService { //	 빈의 이름은 userService

@Component("myUserService")
public class UserService { //	 빈의 이름은 myUserService

@Component
@Named("myUserService")	// JSR-330 방식
public class UserService { //	 빈의 이름은 myUserService
```

```java
@Configuration
public class Config {
	@Bean
	public UserDao userDao() { //	 빈의 이름은 userDao, 메소드 이름
	
@Configuration
public class Config {
	@Bean("name = "myUserDao")
	public UserDao userDao() { //	 빈의 이름은 myUserDao
	
@Configuration
public class Config {
	@Bean("name = {"myUserDao","userDao"})
	public UserDao userDao() { //	 빈의 이름은 myUserDao, userDao  두 개
```

* @Named와 스테레오타입 애노테이션 동시 사용시 한 가지 이름만을 사용해야 한다.
* @Named와 @Component의 이름이 다르다면 예외가 발생할 것이다.

## 빈 생명주기 메소드

### 초기화 메소드
* 빈 오브젝트가 생성되고 DI 작업까지 마친 다음에 실행되는 메소드를 말한다.
* 기본적인 초기화 작업은 생성자에서 진행하지만, DI를 통해 모든 프로퍼티 주입이 완료된 후에 초기화가 필요할 경우에 쓰인다.

#### 초기화 콜백 인터페이스
* InitializingBean 인터페이스를 구현해서 빈을 작성하는 방법이다.
* InitializingBean의 afterPropertiesSet()은 프로퍼티 설정까지 마친 뒤에 호출된다.
* 애플리케이션 빈 코드에 스프링 인터페이스를 노출하기 때문에 권장하는 방법은 아니다.


#### init-method 지정
* XML을 통해 빈을 등록할 경우 init-method 애트리뷰트를 넣어서 초기화 작업을 수행할 메소드 이름을 지정할 수 있다.
* 빈 클래스에 스프링 API가 노출되지 않아 깔끔하지만, 코드만 보고는 초기화 메소드가 호출될지 알 수 없다.

```xml
<bean id="myBean" class="MyBean" init-method="initResource"    />
```

#### @PostConstruct
* 초기화를 담당할 메소드에 @PostConstruct 애노테이션을 붙여주기만 하면 된다.
* 자바의 표준 공통 애노테이션이므로 스프링 콜백 인터페이스를 사용하는 것보다 상대적 부담이 적으며, 코드에서 초기화 메소드가 존재한다는 것을 파악할 수 있다.
* 건장되는 방식이다.

#### @Bean(init-method)
* @Bean 애노테이션을 통해 빈을 정의하는 경우 init-method 엘리먼트를 사용해 초기화 메소드를 지정할 수 있다.

```java
@Bean(init-method="initResource")
public void MyBeean myBean() {
```
***

### 제거 메소드
제거 메소드는 컨테이너가 종료될 때 호출돼서 빈이 사용한 리소스를 반환하거나, 종료 전에 처리해야 할 작업을 수행한다.

#### 제거 콜백 인터페이스
* DisposableBean 인터페이스를 구현해 destroy()를 구현하는 방법이다. 
* 스프링 API에 종속되는코드가 될 수 있다.

#### destroy-method
* <bean> 태그에 destroy-method를 넣어 제거 메소드를 지정할 수 있다.


#### @PreDestroy
* 컨테이너가 종료될 때 실행될 메소드에 @PreDestroy를 붙여주면 된다.

#### @Bean(destroyMethod)
* @Bean 애노테이션의 destroyMethod 엘리먼트를 이용해 제거 메소드를 지정할 수 있다.

***

### 팩토리 빈과 팩토리 메소드
* 생성자 대신 오브젝트를 생성해주는 코드의 도움을 받아 빈 오브젝트를 생성하는 것을 팩토리 빈이라고 부른다.
* 팩토리 빈 자신은 빈 오브젝트로 사용되지 않는다. 다만 빈 오브젝트를 만들어 주는 기능을 제공해줄 뿐이다.

#### FactoryBean 인터페이스
* 다이내믹 프록시를 빈으로 등록할 때 사용했던 방식이다.
* FactoryBean 인터페이스를 구현해서 getObject() 메소드를 구현하고 팩토리 빈으로 등록해 사용할 수 있다.
* 가장 단순하고 자주 사용되는 방법이다.
* 보통 기술 서비스 빈이나 기반 서비스 빈을 활용할 때 주로 사용된다.

#### 스태틱 팩토리 메소드
* 클래스의 스태틱 메소드를 호출해서 인스턴스를 생성하는 방식이다.
* JDK를 비롯해 다양한 기술 API에서 사용된다.
* 전통적인 싱글톤 클래스는 생성자를 직접 호출해서 오브젝트를 만들 수 없다. 오브젝트 생성과 함께 초기화 작업이 필요한 경우라면 스태틱 팩토리 메소드를 사용할 수 있다.

```xml
<bean id="counter" class="GlobalCounter" factory-method="createInstance"    />
```

#### 인스턴스 팩토리 메소드
* FactoryBean 인터페이스를 구현한 팩토리 빈이 바로 팩토리 빈 오브젝트의 메소드를 이용해 빈 오브젝트를 생성하는 대표적인 방법이다.
* 임의의 오브젝트의 메소드를 호출해서 빈을 생셩해야 할 경우, factory-bean과 factory-method를 함께 사용할 수 있다. 이 때는 팩토리 기능을 제공할 빈을 따로 등록해야 한다.

```xml
<bean id="logFactory" class="...LogFactory"    />
<bean id="log" factory-bean="logFactory" factory-method="createLog"   />
```

* logFactory 빈의 createLog 메소드를 호출해 log 빈을 생성하는 xml 파일이다. 이 때는 따로 빈 클래스를 지정하지 않아도 된다.

#### @Bean 메소드
* 일종의 팩토리 메소드이다. 스프링 컨테이너가 @Bean 메소드를 실행해 빈 오브젝트를 가져오는 방식이기 때문이다.
* 자바 코드에 의해 빈의 설정과 DI를 대폭 적용한다면 @Configuration이 붙은 설정 전용 클래스를 사용하는 것이 편리하다. 반면 특정 빈만 팩토리 메소드를 통해 만들고 싶다면 일반 빈 클래스에 @Bean 메소드를 추가하는 방법을 사용하는 편이 낫다.

***
참고자료 :  
[토비의 스프링 3.1 Vol.2 1.4 - 기타 빈 설정 메타정보](https://www.aladin.co.kr/shop/wproduct.aspx?ItemId=19505671)

---
layout: post
title: IoC, DI를 위한 빈 설정 메타정보 작성 
categories: [Spring]
tags: [Spring Framework, 토비의 스프링]
description: DI 설정 메타데이터를 다루는 여러 가지 방법
fullview: false
comments: true
---

 IoC 컨테이너의 가장 기본적인 역할은 코드를 대신해 애플리케이션을 구성하는 오브젝트를 생성하고 관리하는 것이다. 대상은 POJO로 만들어진 애플리케이션 클래스와 서비스 오브젝트이다.   
 이 컨테이너는 빈 설정 메타정보를 통해 빈의 클래스와 이름을 제공받는다.  메타정보는 전용 리더를 통해 읽혀 BeanDefinition 타입의 오브젝트로 변환된다.
 
 <p style="text-align:center">
 <img src="https://user-images.githubusercontent.com/75205849/121119965-9b692000-c857-11eb-89a2-373ed9f0cb8e.png">
 </p>
  
  IoC 컨테이너가 직접 사용하는 BeanDefinition은 순수한 오브젝트로 표현되는 빈 생성 정보다. 따라서 그 정보가 담긴 리소스의 종류와 작성 방식에 독립적이다.
  
### 빈 설정 메타정보
* BeanDefinition에는 IoC 컨테이너가 빈을 만들 때 필요한 핵심 정보가 담겨 있다.
* 몇가지 필수항목을 제외하면 컨테이너에 미리 설정된 디폴트 값이 그대로 적용된다.


#### 빈 설정 메타정보 항목
* BeanDefinition은 여러 개의 빈을 만드는 데 재사용될 수 있다. 설정 메타정보가 같지만 이름이 다른 여러 개의 빈 오브젝트를 만들 수 있기 때문이다.
* 그렇기 때문에 BeanDefinition에는 빈의 이름이나 아이디를 나타내는 정보는 포함되지 않는다. 대신 IoC 컨테이너에 BeanDefinition 정보가 등록될 때 이름을 부여해줄 수 있다.
BeanDefinition에 정의되어있는 빈의 핵심 메타정보는 너무나도 많다. 아래 토비의 스프링에서 제공한 핵심 항목을 살펴보자.

 <p style="text-align:center">
 <img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbUrNMM%2FbtqUuNM22Hx%2FCe2xGOqFiwWSPx3b2k3tbk%2Fimg.png">
 </p>
 
  <p style="text-align:center">
 <img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FyE8Bx%2FbtqUwxXfwQ2%2FaEdmJKin7pv85NDDkIdbt1%2Fimg.png">
 </p>


* 빈 설정 메타정보 항목 중에 가장 중요한 것은 클래스 이름이다. 추상 빈으로 정의하지 않는 한 클래스 정보는 반드시 필요하다.
* 컨테이너에 빈의 메타정보가 등록될 때 꼭 필요한 것은 **클래스 이름**과 **빈의 아이디 또는 이름**이다.


### 빈 등록 방법
* 빈 등록은 빈 메타정보를 작성해 컨테이너에게 건내주면 된다.
* 가장 직접적인 방법은 BeanDefinition 구현 오브젝트를 직접 생성하는 것이다. 하지만 실무에서는 거의 사용되지 않는 방법이다.

스프링에서 자주 사용되는 빈의 등록 방법은 크게 다섯 가지가 있다.

#### XML : <bean> 태그
* <bean> 태그를 사용하는 건 가장 단순하면서도 강력한 설정 방법이다. 

```xml
<bean id="hello" class="spring.learningtest.spring.ioc.bean.Hello>
	...
</bean>
```

* <bean>은 다른 빈의 <property> 태그 안에 정의 할 수 있고, 그런 빈은 내부 빈이라고 부른다.
* 내부 빈은 특정 빈에서만 참조하는 경우에만 사용된다. 아이디가 없으므로 다른 빈에서는 참조할 수 없다. DI이긴 하지만 특정 빈과 강한 결합을 가지고 등록되는 경우에 사용한다.

```xml
<bean id="hello" class="spring.learningtest.spring.ioc.bean.Hello">
	<property name="printer">
		<bean claass="spring.learningtest.spring.ioc.bean.StringPrinter"  />
	</property>
</bean>
```

#### XML : 네임스페이스와 전용 태그
* <bean> 태그 외에도 다양한 스키마에 정의된 전용 태그를 사용해 빈을 등록할 수 있다.
* 스프링의 빈을 분류하자면 크게 애플리케이션의 핵심 코드를 담은 컴포넌트와 서비스 또는 컨테이너 설정을 위한 빈으로 구분 할 수 있다. 보톤 애플리케이션 핵심 코드를 담은 컴포넌트는 <bean>으로 등록하고, 그 외에는 <aop:pointcut>, <jdbc:embedded-database> 등을 이용해서 기술적인 설정을 담당하는 공통 서비스를 선언하는데 사용되는 빈을 손쉽게 확인할 수 있다.
* 그 외에도 개발자가 커스텀 태그를 만들어서 적용할 수 있다.


#### 자동인식을 이용한 빈 등록: 스테레오타입 애노테이션과 빈 스캐너
* 모든 빈을 XML에 일일이 선언하는 것이 귀찮게 느껴질 수도 있다.
* XML 문서와 같이 한곳에 명시적으로 선언하지 않고도 스프링 빈을 등록할 수 있다. **빈으로 사용될 클래스에 특별한 애노테이션을 부여해주면 자동으로 찾아 빈으로 등록할 수 있다**
* 특정 애노테이션이 붙은 클래스를 자동으로 찾아서 빈으로 등록해주는 방식을 빈 스캐닝을 통한 자동인식 빈 등록기능이라 하고, 이런 스캐닝 작업을 담당하는 오브젝트를 빈 스캐너라 한다.
* 스프링 빈 스캐너는 지정된 클래스패스 아래에 있는 모든 패키지의 클래스를 대상으로 필터를 적용해 빈 등록을 위한 클래스를 선별해낸다.
* 빈스캐너에 내장된 디폴트 필터는 @Component 애노테이션 또는 @Component를 메타 애노테이션으로 가진 애노테이션이 부여된 클래스를 선택하도록 되어있다.
* @Component를 포함해 디폴트 필터에 적용되는 애노테이션을 스프링에서는 스테레오타입 애노테이션이라고 부른다.
* 빈 스캐너는 기본적으로 클래스 이름(첫 글자만 소문자로 바꾼 것)을 빈의 아이디로 사용한다.

<br/>

* 어노테이션을 통한 자동 스캐닝의 장점 : 
	* 복잡한 XML 문서 생성과 관리 불필요
	* 개발속도 향상
* 어노테이션을 통한 자동 스캐닝의 단점 : 
	* 애플리케이션에 등록된 빈을 한 눈에 파악하기 어려움
	* XML처럼 상세한 메타정보 지정 불가

#### XML을 이용한 빈 스캐너 등록
* XML 설정파일 안에 context 스키마의 전용 태그를 넣어 빈 스캐너를 등록 할 수 있다.

```xml
<context:component-scan base-package="springbook.learningtest.spring.ioc.bean"   />
```

#### 빈 스캐너를 내장한 애플리케이션 컨텍스트 사용
* XML에 빈 스캐너를 지정하는 대신 아예 빈 스캐너를 내장한 컨텍스트를 사용할 수도 있다.
* 웹에서는 AnnotationConfigWebApplicationContext를 루트 컨텍스트나 서블릿 컨텍스트가 사용하도록 컨텍스트 파라미터를 변경하면 된다.

```xml
<context-param>
	<param-name>contextClass</param-name>
	<param-value>
		org.springframework.web.context.supporx.AnnotationConfigWebApplicationContext
	</param-value>
</context-param>

<context-param>
	<param-name>contextConfigLocation</param-name>
	<param-value>
		pringbook.learningtest.spring.ioc.bean
	</param-value>
</context-param>
```

#### 자바 코드에 의한 빈 등록: @Configuration 클래스의 @Bean 메소드
* 기존에는 의존관계 내에 있지 않은 제 3의 오브젝트를 만들어 의존관계를 가진 오브젝트를 생성하고 메소드 주입을 통해 의존관계를 만들어줬다. 이처럼 오브젝트 생성과 주입을 담당하는 오브젝트를 오브젝트 팩토리라 부른다.
* 오브젝트 팩토리의 기능을 일반화해서 컨테이너로 만든 것이 지금의 스프링 컨테이너, 즉 빈 팩토리이다.
* 자바 코드에서는 @Configuration과 @Bean 애노테이션을 통해 빈 정의를 할 수 있다.
* @Configuration이 붙은 클래스 자체도 빈으로 등록된다.

```java
@Configuration
public class AnnotatedHelloConfig {
	@Bean
	public AnnotatedHello annotatedHello() {	// 메소드 이름이 빈의 이름이다.
		return new AnnotatedHello(); // 디폴트 값은 싱글톤이다.
	}
}
```

* 자바 코드를 이용한 빈 등록은 단순한 빈 스캐닝을 통한 자동인식으로는 등록하기 힘든 기술 서비스 빈의 등록이나 컨테이너 설정용 빈을 XML 없이 등록하려 할 때 유용하게 쓸 수 있다.

```java
@Configuration
public class ServiceConfig {
	@Bean
	public DataSource dataSource() {
		SimpleDriverDataSource dataSource = new SimpleDriverDataSource();
		dataSource.setDriverClass(com.mysql.jdbc.Driver.class);
		dataSource.setUrl("jdbc:mysql://localhost/testdb");
		dataSource.setUsername("sa");
		dataSource.setPassword("book");
		
		return dataSource;
	}
}
```

#### 자바 코드를 통한 빈 등록 방법이 XML보다 유용한 점?
* 컴파일러나 IDE를 통한 타입 검증이 가능하다.(type-safety)
* 자동완성과 같은 IDE 지원 기능을 최대한 이용할 수 있다.
* 이해하기 쉽다.
* 복잡한 빈 설정이나 초기화 작업을 손쉽게 적용할 수 있다.


#### 자바 코드에 의한 빈 등록 : 일반 빈 클래스의 @Bean 메소드
* 앞서 작성한 것에서 @Configuration을 붙이지 않아도 내부 메소드에서 @Bean을 통해 빈으로 등록 할 수 있다.
* 일반적으로 @Bean 메소드를 통해 정의되는 빈이 클래스로 만들어지는 빈과 매우 밀접한 관계가 있는 경우, 특별히 종속적인 빈인 경우에 사용 할 수 있을 것이다.

***
### 빈 등록 메타정보 구성 전략
여러가지 빈을 등록할 수 있는 방법에 대해 알아봤다. 이제 자주 사용되는 설정 방법을 살펴보도록 하자. 

#### XML 단독 사용
* 모든 빈을 명시적으로 XML에 등록하는 방법
* 컨텍스트에서 생성되는 모든 빈을 XML에서 확인할 수 있지만, 반대로 XML 파일을 관리하기 어려울 수 있다.
* <bean>과 스키마에 정의된 전용 태그를 이용하는 것 두가지가 있다.
* 모든 설정정보를 자바 코드에서 분리하고 순수한 POJO 코드를 유지하고 싶다면 XML이 가장 좋은 선택이다.
* XML은 BeanDefinition을 코드에서 직접 만드는 방법을 제외하면 스프링이 제공하는 모든 종류의 빈 설정 메타정모 항목을 지정할 수 있는 유일한 방법이기도 하다.

#### XML과 빈 스캐닝의 혼용
* 애플리케이션의 핵심 로직을 담고 있는 빈 클래스는 애노테이션을 이용하고, 기술 서비스, 기반 서비스, 컨테이너 설정 등의 빈은 XML을 이용하는 방식이다.
* 스캔 대상이 되는 클래스를 위치시킬 패키지를 미리 결정해두어야 한다. 웹 기반의 스프링 애플리케이션에는 보통 두 개의 애플리케이션 컨텍스트가 등록돼서 사용되기 때문이다.
* 특정 빈에 대해 중복으로 스캐닝을 진행하게 되면, AOP등 특정 기능이 적용되어야 하나 적용이 안될 수 있다.
* 웹 애플리케이션의 이중 컨텍스트 계층 구조와 빈 검색 우선순위를 잘 이해하고, 빈 스캐닝 설정을 제공할 때 중복 등록이 발생하지 않도록 주의하자.

#### XML없이 빈 스캐닝 단독 사용
* 스프링 3.0부터 가능한 방법이다.
* 이 방식을 웹 애플리케이션에 적용하려면 루트 컨텍스트와 서블릿 컨텍스트 모두 contextClass 파라미터를 추가해 AnnotationConfigWebApplicationContext로 컨텍스트 클래스를 변경해주어야 한다. (contextLocations 파라미터는 스캔 대상 패키지를 넣어주자)


***

### 빈 의존관계 설정 방법
DI 대상을 선정하는 방법은 여러가지가 있다 :
* 명시적으로 구체적인 빈을 지정하는 방식(DI 할 빈의 아이디를 직접 지정)
* 일정한 규칙에 따라 자동으로 선정하는 방법(주로 타입 비교를 통해 호환되는 타입의 빈을 DI 후보로 삼는 방법, 자동 와이어링이라고 부른다)

이번 챕터에는 총 8가지 빈 의존관계 주입 방법에 대해 알아본다.

#### XML: <property>, <constructor-args>
* <bean>을 이용해 빈을 등룍할 경우 프로퍼티(수정자 메소드)와 생성자(빈 클래스의 생성자) 두 가지 방식으로 DI를 지정할 수 있다.
* <property> : 수정자 주입
	* ref 애트리뷰트 : 빈 이름을 이용해 주입할 빈을 찾을 때 사용
	* value 애트리뷰트 : 단순 값 또는 빈이 아닌 오브젝트 주입시 사용
	* 타입 정보가 나타나지 않기 때문에 입력한 값이 타입과 호환되는지 주의해야 한다.
* <constructor-args> : 생성자를 통한 빈 또는 값의 주입
	* 한 번에 여러 개의 오브젝트 주입 가능
	* 파라미터의 순서나 타입을 명시하는 방법이 필요하다.

#### XML: 자동와이어링
* XML 문서의 양을 대폭 줄여줄 수 있는 방법이다.
* 명시적으로 프로퍼티나 생성자 파라미터를 지정하지 않고 미리 정해진 규칙을 이용해 자동으로 DI 설정을 컨테이너가 추가하도록 만드는 것이다.
* 대표적으로 이름을 사용하는 자동 와이어링과 타입을 사용하는 자동 와이어링 두 가지가 있다.
	* byName: 빈 이름 자동와이어링
		* autowire="byName"을 사용하면 되며, 루트 태그인 <beans>에서 디폴트 자동 와이어링 옵션을 변경해주어도 된다.
		* 보통 디폴트 자동와이어링 옵션을 변경하는 방법을 이용한다. 한두개의 빈만 자동와이어링을 사용할 경우 별 장점이 없기 때문이다.

```xml
<bean id="hello" class="...Hello" autowire="byName">
	<property name="name" value="Spring"	/>		// printer 프로퍼티 명시 생략 
</bean>

<bean id="printer" class=".....StringPrinter"	/>
```

	* byType : 타입에 의한 자동 와이어링
		* autowire="byType"을 <bean> 혹은 <beans>에 넣어주면 된다.
		* 타입이 같은 빈이 두 개 이상 존재하는 경우에 적용되지 못한다.

```xml
<bean id="hello" class="...Hello" autowire="byType">		// 이름이 달라도 주입됨
	<property name="name" value="Spring"	/>		// printer 프로퍼티 명시 생략 
</bean>

<bean id="Mainprinter" class=".....StringPrinter"	/>
```

* 자동 와이어링 방식도 XML만 봐서는 빈 사이의 의존관계를 알기 힘들다.
* 하나의 빈에 대해 한 가지 자동와이어링 방식밖에 지정할 수 없다.


#### XML: 네임스페이스와 전용 태그
* <bean>의 <property> <constructor-args>라는 DI태그처럼 명시해주는 태그가 없어 잘 사용되지 않는다.
* 전용 태그로 만드러지는 빈이 일반 <bean> 태그로 선언된 빈을 DI 할 수도 있으며, 그 반대도 가능하다.

#### 애노테이션: @Resource
* 빈의 아이디로 지정하는 방법이다.
* 자바 클래스의 수정자 뿐 아니라 내부 필드에도 DI 할 수 있다.
* 애노테이션 기반 의존정보를 이용해 DI가 이루어지기 위해서는 아래 세 가지 방법중 하나를 선택해야 한다 : 
	* XML의 <context:annotation-config	/> : @Resource와 같은 애노테이션 의존 관계 정보를 읽어서 메타정보를 추가해주는 기능을 가진 빈 후처리기를 등록해주는 전용 태그다. 빈 후처리기는 이미 등록된 빈의 메타 정보에 프로퍼티 항목을 추가해주는 작업을 한다.
	* XML의 <context:component-scan	/> : 빈 스캐닝 방식이다.
	* AnnotationConfigApplicationCotext 또는 AnnotationConfigWebApplicationContext : 빈 스캐너와 애노테이션 의존관계 정보를 읽는 후처리기를 내장한 애플리케이션 컨텍스트를 사용한다.
* 필드 주입 : @Resource를 필드에 붙여서 원하는 빈을 DI하는 방식. @Resource 애노테이션의 name 엘리먼트를 생략하면 DI할 빈의 이름이 프로퍼티나 필드 이름과 같다고 가정한다.
* @Resource가 붙어 있으면 반드시 참조할 빈이 존재해야 한다. 만약 DI 할 빈을 찾을 수 없다면 예외가 발생한다.
* 만약에 빈 이름으로 참조할 빈을 찾을 수 없는 경우에는 타입을 이용해 다시 한번 빈을 찾기도 한다.

#### 애노테이션: @Autowired/@Inject
이 두가지 애노테이션은 기본적으로 타입에 의한 자동와이어링 방식으로 동작한다.
* @Autowired : 스프링 2.5부터 적용된 스프링 전용 애노테이션
* @Inject : JavaEE6의 표준 스펙인 JSR-330에 정의되어 있는 것으로, 스프링이 아니여도 DI를 적용할 수 있기위해 만든 애노테이션이다.
* @Autowired : 필드, 생성자(@Resource는 불가), 수정자 메소드, 일반 메소드에서 사용 가능하다.
	* 생성자 주입은 일단 오브젝트를 만들고 설정 값을 넣어야 하는 경우에는 적합하지 않다.
	* 수정자 메소드는 DI를 위해 딱 한 번 사용할 수도 있는데 각각 프로퍼티별로 수정자 메소드에 주입해야 하기 때문에 수정자 메소드가 많아질 수 있다.
	* 위 두가지 한계를 극복하기 위해 일반 메소드를 사용해서 DI 하는 것을 지원한다.
	* required=false 속성을 통해 DI를 선택적으로 가능하게 할 수 있다.

* DI 받는 빈이 여러개라면?(같은 타입의 빈이 여러개 등록되어 있을 경우)
	* Collection, Set, List, 배열,Map등으로 필드 주입
	* 의도적으로 타입이 같은 여러 개의 빈을 등록하고 이를 모두 참조하거나, 그 중에서 선별적으로 필요한 빈을 찾을 때 사용하는 것이 좋다.
	* DI 할 빈의 타입이 컬렉션인 경우 @Autowired로 자동 설정이 불가능하다. 빈 자체가 컬렉션인 경우에는 @Resource를 이용해야 한다.
* @Qualifier : 타입 외 정보를 추가해서 자동와이어링을 세밀하게 제어할 수 있는 보조적인 방법이다.
	* 타입만으로는 원하는 빈을 지정하기 어려운 경우 주로 사용된다.
	* 보통 빈의 이름은 환경이나 특정 기술을 따라가는 경우가 많다. 또한 빈 이름은 변경되기 쉽고, 그 자체로 어떤 의미를 부여하기 쉽지 않다.
	* 빈 이름과는 별도로 추가적인 메타정보를 지정해서 의미를 부여해놓고 이를 @Autowired에서 사용할 수 있게 하는 @Qualifier가 훨씬 직관적이다.

```java
@Autowired
@Qualifier("mainDB")
DataSource dataSource;
```

```java
@Component
@Qualifier("mainDB")		// <qualifier value="mainDB"	/>와 동일하다.
public class OracleDataSource {....
```
	* @Qualifier는 빈에 부가적인 속성을 지정해주는 효과가 있다.
	* @Qualifier는 해당 프로퍼티로 등록하지 않아도, 프로퍼티와 같은 이름을 가진 빈이 있는지 확인하고 있다면 해당 빈을 DI 대상으로 선택한다.
	* 필드, 수정자, 파라미터에만 입력 가능하다.

***
참고자료 :  
[토비의 스프링 3.1 Vol.2 1.2- IoC/DI를 위한 빈 설정 메타정보 작성](https://www.aladin.co.kr/shop/wproduct.aspx?ItemId=19505671)

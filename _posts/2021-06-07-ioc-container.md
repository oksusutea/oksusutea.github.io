---
layout: post
title: IoC 컨테이너와 DI
categories: [Spring]
tags: [Spring Framework, 토비의 스프링]
description: DI 설정 메타데이터를 다루는 여러 가지 방법
fullview: false
comments: true
---

### IoC 컨테이너 : 빈 팩토리와 애플리케이션 컨텍스트
스프링 애플리케이션에서는 오브젝트의 생성과 관계설정, 사용, 제거 등의 작업을 코드 대신 독립적인 컨테이너가 담당한다. 이 스프링 컨테이너를 IoC 컨테이너라고 하는데, 스프링에서는 빈 팩토리 혹은 애플리케이션 컨텍스트라고 부른다.  두가지는 아래의 관점으로 주로 이야기 한다. 

* 빈 팩토리 : 오브젝트의 생성과 오브젝트 사이의 런타임 관계 설정 (BeanFactory로 정의)
* 애플리케이션 컨텍스트 : 빈 팩토리에 부가적인 기능 추가 (ApplicationContext로 정의, 해당 인터페이스는 BeanFactory를 상속)

실제 스프링에애플리케이션 내 IoC 컨테이너는 ApplicationContext 인터페이스를 구현한 클래스의 오브젝트이며, 최소한 하나 이상의 IoC 컨테이너를 가지고 있다.

#### IoC 컨테이너를 이용해 애플리케이션 만들기
가장 간단한 방법은 ApplicationContext 구현 클래스의 인스턴스를 만드는 것이다.


여기서 컨테이너만 만들었다고 끝이 아니다. IoC 컨테이너가 제대로 동작하기 위해서는 아래 두가지 정보가 필요하다.  

#### POJO 클래스
*  특정 기술과 스펙에서 독립적일 뿐더러 의존관계에 있는 다른 POJO와 느슨한 결합을 가져야 한다.

#### 메타정보
* 스프링의 설정 메타정보는 BeanDefinition 인터페이스로 표현되는 순수한 추상 정보다. 
* XML이든 소스코드 애노테이션이든 자바 코드이든 프로퍼티 파일이든 상관없이 BeanDefinition으로 정의되는 스프링의 설정 메타정보의 내용을 표현한 것이 있다면 무엇이든 사용가능하다. 
* 원본의 포맷과 구조, 자료의 특성에 맞게 읽어와 BeanDefinition으로 변환해주는 BeanDefinitionReader만 있으면 된다.
* BeanDefinition 인터페이스로 정의되는, IoC 컨테이너가 사용하는 빈 메타정보는 아래와 같다.
	* 빈 아이디, 이름, 별칭 : 빈 오브젝트를 구분할 수 있는 식별자
	* 클래스, 또는 클래스 이름 : 빈으로 만들 POJO 클래스 또는 서비스 정보
	* 스코프 : 싱글톤 / 프로토타입과 같은 빈의 생성 방식과 존재 범위
	* 프로퍼티 값 참조 : DI에 사용할 프로퍼티 이름과 값 또는 참조하는 빈의 이름
	* 생성자 파라미터 값 또는 참조 : DI에 사용할 생성자 파라미터 이름과 값 또는 참조하는 빈의 이름
	* 지연로딩 여부, 우선 빈, 여부, 자동와이어링 여부, 부모 빈 정보, 빈팩토리 이름 등등

스프링 IoC 컨테이너는 이렇게 각 빈에 대한 정보를 담은 설정 메타정보를 읽어들인 후, 이를 참고하여 빈 오브젝트를 생성하고 프로퍼티나 생성자를 통해 의존 오브젝트를 주입해주는 DI작업을 수행한다.

<p style="text-align:center">
<img src="https://user-images.githubusercontent.com/75205849/121006072-6fa05880-c7cb-11eb-9695-832c4d3ddfbd.jpeg" width=400>
</p>

스프링 애플리케이션을 이렇게 POJO 클래스와 설정 매타정보를 통해 IoC 컨테이너가 만들어주는 오브젝트의 조합이라고 할 수 있다. 이 **IoC 컨테이너가 관리하는 빈은 오브젝트 단위이며, 클래스 단위가 아니라는 점을 유의**해야 한다.

애플리케이션을 구성하는 빈 오브젝트를 생성하는 것이 IoC 컨테이너의 핵심기능이다. IoC 컨테이너는 일단 빈 오브젝트가 생성되고 관계가 만들어지면 그 뒤로는 거의 관여하지 않는다. 기본적으로 싱글톤 빈은 애플리케이션 컨텍스트의 초기화 작업 중에 모두 만들어진다.

***

### IoC 컨테이너의 종류와 사용 방법
보통 스프링 애플리케이션에서 직접 코드를 통해 ApplicationContext 오브젝트를 생성한는 경우는 거의 없다. 대부분 간단한 설정을 통해 자동으로 만드어 지는 방법을 사용하기 때문이다. 하지만 어떤 구현 클래스가 있는지 학습차원에서 알아보자.

#### StaticApplicationContext
* 코드를 통해 빈 메타정도를 등록하기 위해 사용
* 스프링 기능에 대한 학습 테스트를 만들 때를 제외하면 실제로 사용되지 않는다
* 테스트 목적으로 코드를 통해 빈을 등록하고, 컨테이너가 어떻게 동작하는지 확인하고 싶을 때를 대비해 이런 컨테이너가 있다는 정도만 기억해둔다.

#### GenericApplicationContext
* 가장 일반적인 애플리케이션 컨텍스트의 구현 클래스
* 실전에서 사용될 수 있는 모든 기능을 갖추고 있다.
* XML파일과 같은 외부 리소스에 있는 빈 설정 메타정보를 리더를 통해 읽어들여 메타정보로 전환해 사용한다. (XmlBeanDefinitionReader, PropertiesBeanDefinitionReader 등등)
* 스프링 IoC 컨테이너가 사용할 수 있은 BeanDefinition 오브젝트로 변환만 될 수 있다면 설정 메타정보는 어떤 포맷으로 만들어져도 상관없다. 스프링에서는 대표적으로 XML, 자바 소스코드 애노테이션, 자바 클래스 세가지 방식으로 빈 설정 메타정보를 등록 할 수 있다.
* GenericApplicationContext은 빈 설정 리더를 여러개 사용해 여러 리소스로부터 설정 메타정보를 읽어오게 할 수도 있다. 모든 메타정보를 가져온 후, refresh()를 호출해 초기화 작업을 수행하게 해주면 된다.
* GenericApplicationContext 또한 직접적으로 이용하진 않는다. 하지만 실제로 자주 사용되는데, JUnit 테스트 내에 해당 애플리케이션 컨텍스트를 활용해 자동으로 만들어준다.

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = "/test-applicationContext.xml")
public class UserServiceTest {
	@Autowired ApplicationContext applicationContext;
}
```

#### GenericXmlApplicationContext
* 코드에서 GenericApplicationContext 를 사용하는 경우에는 번거롭게 XmlBeanDefinitionReader를 직접 만들지 말고, 이 두 개의 클래스가 결합된 GenericXmlApplicationContext를 사용하면 편리하다
* GenericXmlApplicationContext는 XmlBeanDefinitionReader를 내장하고 있기 때문에, XML 파일을 읽어들이고 refresh()를 통해 초기화하는 것까지 한 줄로 끝낼 수 있다.

```java
GenericApplicatonContext ac = new GenericXmlApplicationContext(
    "springbook/learningtest/spring/ioc/genericApplicationContext.xml");
Hello hello = ac.getBean("hello", Hello.class)
```

#### WebApplicationContext
* 스프링 애플리케이션에서 가장 많이 사용되는 애플리케이션 컨텍스트이다.
* 이름 그대로 웹 환경에서 사용할 때 필요한 기능이 추가된 애플리케이션 컨텍스트다.
* 주로 XmlWebApplicationContext와 AnnotationConfigWebApplicationContext를 쓰면 된다.
* 스프링 IoC 컨테이너는 빈 설정 메타정보를 이용해 빈 오브젝트를 만들고 DI 작업을 수행한다. 하지만 그것만으로는 애플리케이션이 동작하지 않는다. 마치 자바 애플리케이션의 main() 메소드처럼 어디에선가 특정 빈 오브젝트의 메소드를 호출함으로써 애플리케이션을 동작시켜야 한다. IoC 컨테이너의 역할은 초기에 빈 오브젝트를 생성하고 DI 한 후에 최초로 애플리케이션을 기동할 빈 하나를 제공해주는 것까지다.
* 웹 환경에서는 main() 메소드 대신 서블릿 컨테이너가 브라우저로부터 오는 HTTP 요청을 받아서 해당 요청에 매핑되어 있는 서블릿을 실행해주는 방식으로 동작한다. 서블릿이 일종의 main() 메소드와 같은 역할을 하는 셈이다.
* 그렇다면 웹 애플리케이션에서 스프링 애플리케이션을 기동시키는 방법은 무엇일까? 일단 main() 메소드 역할을 하는 서블릿을 만들어두고, 미리 애플리케이션 컨텍스트를 생성해둔 다음, 요청이 서블릿으로 들어올 때마다 getBean()으로 필요한 빈을 가져와 정해진 메소드를 실행해주면 된다.

<p style="text-align:center">
<img src="https://user-images.githubusercontent.com/75205849/121010903-f277e200-c7d0-11eb-8e54-1deeeff7af3e.jpeg" width=400>
</p>

* 웹환경에서 스프링 빈으로 이루어진 애플리케이션이 동작하는 구조는 아래와 같다 : 
	* 서블릿 컨테이너는 브라우저와 같은 클라이언트로부터 들어오는 요청을 받아 서블릿을 동작시켜주는 역할을 책임진다.
	* 서블릿은 웹 애플리케이션이 시작될 때 미리 만들어둔 웹 애플리케이션 컨텍스트에게 빈 오브젝트로 구성된 애플리케이션의 기동 역할을 해줄 빈을 요청해 받아둔다.
	* 미리 지정된 메소드를 호출해 스프링 컨테이너가 DI 방식으로 구성해둔 애플리케이션의 기능을 시작한다.
* 스프링은 웹 환경에서 애플리케이션 컨텍스트를 생성하고, 설정 메타정보로 초기화 해주고, 클라이언트로부터 들어오는 요청마다 적절한 빈을 찾아 실행해주는 기능인 DispatcherServlet을 제공한다.
* WebApplicationContext는 자신이 만들어지고 동작하는 환경인 웹 모듈에 대한 정보에 접근 할 수 있다.

****

### IoC 컨테이너 계층구조
모든 애플리케이션 컨텍스트는 부모 애플리케이션 컨텍스트를 가질 수 있다.

<p style="text-align:center">
<img src="https://user-images.githubusercontent.com/75205849/121011719-e50f2780-c7d1-11eb-9963-7e812e17317e.jpeg" width=400>
</p>

* 계층구조 안의 모든 컨텍스트는 각자 독립적인 설정정보를 이용해 빈 오브젝트를 만들고 관리한다. 
* 각자 독립적으로 자신이 관리하는 빈을 갖고 있지만 DI를 위해 빈을 찾을 때는 부모 애플리케이션 컨텍스트의 빈까지 모두 검색한다.
* 자식 및 형제 컨테스트에는 요청을 하지 않는다.
* 검색 순서는 항상 자신이 먼저이고, 그런 다음 직계 부모의 순서다. 따라서 부모 컨텍스트와 같은 이름의 빈을 자신이 정의해서 갖고 있다면 자신이 가진 것이 우선이고, 부모 컨테스트가 정의한 것은 무시된다.
* AOP처럼 컨텍스트 안의 많은 빈에 일괄적으로 적용되는 기능은 대부분 해당 컨텍스트로 제한된다.
* 부모/자식 구조의 계층구조는 스프링 애플리케이션에서 자주 활용되는 방법이지만 절대로 남용해서는 안되고 부모/자식 컨텍스트에 중복해서 빈이 정의되는 일은 가능한 피애햐 한다.

***
### 웹 애플리케이션의 IoC 컨테이너 구성
서버에서 동작하는 애플리케이션에서 스프링 IoC 컨테이너를 사용하는 방법은 크게 세가지로 구분할 수 있다. 두 가지 방법은 웹 모듈 안에 컨테이너를 두는 것이고, 나머지 하나는 엔터프라이즈 애플리케이션 레벨에 두는 방법이다.
#### 웹 애플리케이션 안에 WebApplicationContext 타입의 IoC 컨테이너 설정 방법
웹 애플리케이션 안에서 동작하는 IoC 컨테이너는 두 가지 방법으로 만들어진다. 

* 스프링 애플리케이션의 요청을 처리하는 서블릿 안에서 생성
* 웹 애플리케이션 레벨에서 생성

일반적으로 이 두 가지 방식을 모두 사용해 컨테이너를 만든다. 그래서 스프링 웹 애플리케이션에서는 두 개의 컨테이너, 즉 WebApplicationContext 오브젝트가 만들어진다.

#### 웹 애플리케이션의 컨텍스트 계층 구조
웹 애플리케이션 레벨에 등록되는 컨테이너는 보통 루트 웹 애플리케이션 컨텍스트라고 부른다.

* 해당 컨텍스트는 서블릿 레벨에 등록되는 컨테이너들의 부모 컨테이너가 된다.
* 일반적으로 전체 계층구조 내 가장 최상단에 위치한 루트 컨텍스트가 된다.

<p style="text-align:center">
<img src="https://user-images.githubusercontent.com/75205849/121020796-aed6a580-c7db-11eb-9f01-b501a2f7d503.jpeg" width=400>
</p>
* 웹 애플리케이션에는 하나 이상의 스프링 애플리케이션의 프론트 컨트롤러 역할을 하는 서블릿이 등록될 수 있다.
* 각 서블릿이 공유하게 되는 공통적인 빈들을 웹 애플리케이션 레벨의 컨텍스트에 등록하면, 공통되는 빈들이 서블릿 별로 중복되어 생성되는 것을 방지 할 수 있다.

* 보통 위 그림과 같이 두 개의 서블릿을 두고 사용하는 경우는 많지 않다. 그렇다면 왜 굳이 계층 구조를 띄는 것일까?
	* 전체 애플리케이션에서 웹 기술에 의존적인 부분과 그렇지 않은 부분을 구분하기 위해서이다.

<p style="text-align:center">
<img src="https://user-images.githubusercontent.com/75205849/121021759-acc11680-c7dc-11eb-8522-d8acebd404fe.jpeg" width=400>
</p>

* 위와 같이 프레젠테이션 계층을 분리해 계층구조로 애플리케이션 컨텍스트를 구성해두면, 간단히 웹 기술을 확장하거나 변경, 조합해서 사용할 수 있다.
* 하지만 루트 컨텍스트에 정의된 빈은 이름이 같은 서블릿 컨텍스트의 빈이 존재하면 무시 될 수 있고, AOP 설정 내 다른 컨텍스트의 빈에는 영향을 미치지 않는 다는 점을 주의하자.
* 보통 계층이나 성격에 따라 여러 개의 파일로 분리하여 설정파일을 작성한다.



#### 웹 애플리케이션의 컨텍스트 구성 방법

* 서블릿 컨텍스트와 루트 애플리케이션 컨텍스트 계층구조
	* 가장 많이 사용되는 기본적인 구성 방법
	* 스프링 웹 기술 사용시 웹 관련 빈은 서블릿의 컨텍스트에 두고, 나머지는 루트 애플리케이션 컨텍스트에 등록하는 방식
* 루트 애플리케이션 컨텍스트 단일구조
	* 스프링 웹 기술을 사용하지 않을 경우 쓰는 방식
* 서블릿 컨텍스트 단일구조
	* 스프링 웹 기술을 사용하지만 스프링 외 프레임워크 혹은 서비스 엔진에서 스프링의 빈을 이용하지 않을 경우 사용하는 방식
	* 부모 컨텍스트를 갖지 않아 스스로 루트 컨텍스트가 되는 방식

#### 루트 애플리케이션 컨텍스트 등록
* 웹 애플리케이션 레벨에서 루트 애플리케이션 컨텍스트를 등록하는 가장 간단한 방법은 **서블릿의 이벤트 리스너를 이용하는 것**이다.
* ServletContextListener 인터페이스를 구현한 리스너는 웹 애플리케이션 전체에 적용 가능한 DB 연결 기능 혹은 로깅 같은 서비스를 만드는데 쓰인다.
* ContextLoaderListener는 스프링에서 웹 애플리케이션이 시작될 때 루트 애플리케이션 컨텍스트를 만들어 초기화하고, 웹 애플리케이션 종료시 컨텍스트를 함께 종료하는 기능을 가진 리스너를 제공한다.

* ContextLoaderListener의 디폴트 설정 파일
	* 애플리케이션 컨텍스트 클래스 : XmlWebApplicationContext
	* XML 설정파일 위치 : /WEB-INF/applicationContext.xml

```xml
<listener>
  <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
  <context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>
      /WEB-INF/daoContext.xml
      /WEB-INF/applicationContext.xml
    </param-value>
  </context-param>
  <context-param>
    <param-name>contextClass</param-name>
    <param-value>
      org.springframework.web.context.support.AnnotationConfigWebApplicationContext
    </param-value>
  </context-param>
</listener>
```

XML 파일을 통해 컨텍스트 클래스와 설정파일 위치를 변경할 수 있다.

* contextConfigLocation :
	* 하나 이상의 XML 설정파일을 사용할 경우 여러 줄에 걸쳐 넣어주거나 공백으로 분리하면 된다.
	* 앞에 classpath:와 같은 접두어를 붙여줄 수 있다.
	* 애플리케이션의 규모가 커져 등록해야 할 빈이 많으면 빈 설정을 여러 개의 파일로 쪼개서 관리하는 것이 편하다.
* contextClass :
	* ContextLoaderListener가 자동으로 생성하는 컨텍스트의 클래스는 XmlWebApplicationContext이다.
	* 해당 파라미터를 이용해 다른 애플리케이션 컨텍스트 구현 클래스로 변경 할 수 있다. 
	* 여기에 사용된 컨텍스트는 반드시 WebApplicationContext 인터페이스를 구현해야 한다.
	* 보통 AnnocationConfigWebApplicationContext를 대체로 많이 사용한다. 해당 컨텍스트 클래스 사용지 contextConfigLocation을 반드시 선언해주어야 한다.

#### 서블릿 애플리케이션 컨텍스트 등록
* 스프링의 웹 기능을 지원하는 프론트 컨트롤러 서블릿은 DispatcherServlet이다.해당 서블릿은 web.xml에 등록하여 사용 할 수 있다.
* 서블릿 이름을 다르게 지정해주면 하나의 웹 애플리케이션에 여러 개의 DispatcherServlet을 등록할 수 있다.


```xml
<servlet>
  <servlet-name>spring</servlet-name>
  <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
  <load-on-startup>1</load-on-startup>
</servlet>
```

DispatcherServlet 등록시 아래 두가지 사항을 신경쓰자.
* <servlet-name>
	* DispatcherServlet에 의해 만들어지는 애플리케이션 컨텍스트는 모두 독립적인 네임스페이스를 갖게된다. 네임 스페이스는 <servlet-name>으로 지정한 서블릿 이름에 -servlet을 붙여서 만든다.
	* DispatcherServlet는 사용할 디폴트 XML 설정파일 위치를 네임스페이스를 이용해서 만든다.
	* 이를 통해 여러 개의 DispatcherServlet이 등록되어도 각각 구분 할 수 있고, 자신만의 디폴트 설정파일을 가질 수 있다.
* <load-on-startup>
	* 서블릿 컨테이너가 등록된 서블릿을 언제 만들고 초기화 할지, 또 그 순서는 어떻게 되는지를 지정하는 정수 값이다.
	* 항목 생략 혹은 음의 정수로 넣으면 해당 서블릿은 서블릿 컨테이너가 임의로 정한 시점에 만들어진다.
	* 0 이상의 값을 넣으면 웹 애플리케이션이 시작되는 시점에서 서블릿을 로딩하고 초기화하며, 작은 수를 가진 서블릿이 우선적으로 만들어진다.
	* 보통은 1을 넣는다.


<br/>
루트 애플리케이션 컨텍스트는 여러 계층의 빈을 모두 포함하고, 그 외 각종 기반 서비스와 기술 설정을 갖고 있어 설정 파일을 여러개로 구분해두는 경우가 많다.  
그에 반해 서블릿은 굳이 여러 개로 구분해 분리할 필요가 없어 특별한 경우가 아니라면 서블릿이름 + '-servlet.xml'이라는 디폴트 설정파일 이름을 따르는 것이 간편하다.

<br/>
* DispatcherServlet의 디폴트 설정은 루트 애플리케이션과 동일하게 contextConfigLocation과 contextClass를 지정할 수 있으며, <init-param>을 이용한다.



***
참고자료 :  
[토비의 스프링 3.1 Vol.2 1.1- IoC 컨테이너와 DI](https://www.aladin.co.kr/shop/wproduct.aspx?ItemId=19505671)

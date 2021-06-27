---
layout: post
title: AspectJ와 @Configurable
categories: [spring]
tags: [java, spring, 토비의 스프링]
description: AspectJ AOP 라이브러리 활용 방법과 로딩시점의 바이트코드를 조작하는 방법
fullview: false
comments: true
---

## AspectJ AOP
* AspectJ는 가장 강력한 AOP 프레임워크다.
* AspectJ는 아예 타깃 오브젝트 자체의 코드를 바꿈으로써 애스펙트를 적용한다. 프록시를 사용하지 않고 타깃 오브젝트의 자바 코드에 처음부터 애스펙트가 적용되어 있는 것처럼 클래스 바이트 코드를 변경하는 작업이 필요하다.
* AspectJ의 조인 포인트는 메소드 실행 지점 외에도 필드 읽기와 쓰기, 스태틱 초기화, 인스턴스 생성, 인스턴스 초기화 등도 지원한다.
* 대부분의 엔터프라이즈 애플리케이션에는 스프링의 프록시 방식 AOP로 충분하다. 메소드 실행 지점을 조인포인트로 사용해 부가기능을 충분히 제공할 수 있기 때문이다.


### 빈이 아닌 오브젝트에 DI 적용하기
* DI가 필요없을 경우 굳이 스프링의 빈으로 등록할 필요가 없다. 가장 대표적인 것이 바로 User와 같은 도메인 오브젝트다.
* 하지만 도메인 오브젝트에 DI가 필요한 상황이 발생할 경우 어떻게 해야 할까 ?
	* 스코프를 프로토타입으로 선언하고, Provider등을 이용해 필요할 때마다 새로운 오브젝트를 가져오도록 만들어야 할 것이다.
	* 애플리케이션 코드에서는 가능하지만, 하이버네이트나 iBatis, JPA등 프레임워크다 핸들러 어댑터 내부에서 도메인을 생성할 때는 이런 방식의 적용이 불가능 하다.
* 이 때 AspectJ AOP를 이용해서 특정 오브젝트가 생성되는 시점에 자동으로 DI기능을 가진 어드바이스를 적용 할 수 있다.

#### DI 애스팩트
* 스프링은 AspectJ 기술로 만들어진 DependencyInjectionAspect라는 애스펙트를 제공한다.
* @Configurable이 붙은 도메인 오브젝트가 어디서든 생성될 때마다 이 어드바이스가 적용되어 DI 작업이 자동으로 일어난다.

```java
@Configurable
public class User {
...
}
```

@Configurable이 적용된 클래스의 DI를 설정하는 방법은 세가지가 있다.

* <bean> 설정 : 
	* @Configurable이 붙은 클래스는 스프링의 빈이 아니고, 빈으로 등록될 필요도 없고, 등록할 수도 없다. 그럼에도 DI 설정정보를 제공해주기 위해 <bean>을 이용한다.
	* 빈 오브젝트가 불필요하게 만들어지는 것을 피하기 위해 abstract="true"를 넣어준다.

```xml
<bean class="springbook....User" abstract="true">
	<property name="userPolicyDao" ref="userPolicyDao"    />	<property name="emailService" ref="emailService"    />
</bean>
```

* 자동와이어링 :
	* 수정자 메소드를 준비해놨다면 자동와이어링 방식을 적용할 수도 있다.
	* 모든 수정자 메소드에 대해 DI를 시도한다.

```java
@Configurable(autowire=Autowire.BY_NAME)
public class User {
```

* 어노테이션 의존관계 설정
	* <context:annotation-config	/>와 같은 설정이 필요하다.
	* 수정자 메소드를 생략할 수 있다.

```java
@Configurable
public class User {
	@Autowired private UserPolicyDao userPolicyDao;
	@Autowired private EmailService emailService;
}
```

### 로드타임 위버와 자바 에이전트
* DI 애스펙트를 적용하기 위해 두가지 작업이 필요하다 : 
	* AspectJ AOP가 동작할 수 있는 환경설정 추가
	* DI 애스펙트 자체를 등록해 @Configurable 오브젝트에 어드바이스가 적용되도록 기능 추가

* AspectJ를 사용하려면 클래스를 로딩하는 시점에 바이트코드 조작이 가능하도록 로드타임 위벌르 적용해야 한다.
* 로드타임 위버는 JVM의 javaagent 옵션을 사용해 JVM 레벨에 적용해야 한다.
* AspectJ의 aspectjweaver.jar 파일을 자바에이전트로 지정하고, META-INF 밑에 aop.xml 설정파일도 만들어줘야 한다. (스프링에서는 AspectJ 로드타임 위버 대신 스프링이 직접 제공하는 로드타임 위버 방식을 사용할 수 있으며, 해당 방법을 권장한다.)
* 스프링의 로드타임 위버는 org.springframework.instrument-3.0.7.RELEASE.jar 또는 spring-instrument-3.0.7.RELEASE.jar라는 파일로 되어있다. 이 Jar파일을 클래스패스에 등록해 사용하는 것이 아니라 JVM 옵션으로 지정해주면 된다.

```
-javaagent:lib/org.springramework.instrument-3.0.7.RELEASE.jar
```

* 자바에이전트 등록 후, 스프링의 XML 설정 파일에 아래와 같이 스프링의 로드타임 위버를 사용한다는 설정을 추가한다.

```
<context:load-timew-weaver	aspectj-weaving="on"	/>	//META-INF 밑에 aop.xml 파일을 등록하지 않았다면, aspectj-weaving을 on으로 바꿔준다.
```

* AspectJ 위버를 사용할 수 있는 준비는 끝났다. 다음은 DI 애스펙트를 등록할 차례다. 아래와 같이 간단하게 태그를 추가하면 DI 애스펙트가 등록된다.

```
<context:spring-configured	/>
```

* JVM 옵션을 바르게 적용해 스프링 애플리케이션을 시작하면, @Configurable이 적용된 도메인 오브젝트에 DI가 적용되는 것을 확인할 수 있다. (자바코드는 @EnableAspectJAutoProxy)
---
layout: post
title: 스프링의 기타 기술과 효과적인 학습 방법 (1)
categories: [spring]
tags: [java, spring, 토비의 스프링]
description: 빈으로 등록되는 스프링 클래스와 DI
fullview: false
comments: true
---

* 스프링은 일관된 방식으로 개발된 프레임워크다.
* 모든 코드와 API가 동일한 원리에 기반을 두고 만들어져 있다. 바로 DI다.
* DI는 객체지향 설계 원칙을 충실히 적용했을 때 만들어지는 코드의 특징이다.
* 스프링의 핵심 엔진이라고 할 수 있는 애플리케이션 컨텍스트/컨테이너와 웹 기술의 중심인 DispatcherServlet 모두 DI 원리를 이용하여 확장할 수 있도록 만들어져 있다.

## 빈으로 등록되는 스프링 클래스와 DI
어떤 오브젝트가 빈으로 등록된다는 것은 두 가지 의미가 있다.
	* 다른 빈에 의해 DI 돼서 사용되는 서비스이다, 즉 클라이언틀르 갖는다는 말이다.
	* 다른 빈이나 정보에 의존하고 있다는 의미다. 대부분의 빈은 자신이 클라이언트가 되어 다른 빈 오브젝트를 사용하도록 만들어져 있다.
* 빈이 구현하고 있는 인터페이스는 빈이 제공하는 기능을 정의한 것이다. 또, 해당 빈을 사용하려는 클라이언트와의 연결 채널이다.
* 빈의 프로퍼티는 빈 기능의 확장 포인트다. 이를 이용해 빈의 세부 전략을 재구성 하는 방법도 알 수 있다.

<br/>
이미 살펴봤던 스프링의 기능 중 몇 가지를 이 원리를 이용해 다시 살펴본다.
* 예제 : JdbcTemplate의 DataSourceTransactionManager

### 구현 인터페이스 분석
* 인터페이스를 살펴보는 방법은 두 가지가 있다.
	* 스프링 API 문서에서 DataSourceTransactionManager를 찾아보는 것
	* IDE에서 타입 계층구조를 보는 기능 사용

<p style="text-align:center">
<img src="https://user-images.githubusercontent.com/75205849/123560651-9526ed80-d7de-11eb-8dda-bccf2d2a6e3f.png">
</p>

* 위 내용을 통해 DataSourceTransactionManager가 구현하고 있는 인터페이스 목록을 볼 수 있다. 여기서 PlatformTransactionManager 인터페이스를 보면, DataSourceTransactionManager가 무슨 일을 하는지 파악할 수 있다.
> PlatformTransactionManager는 스프링의 트랜잭션 인프라 스트럭처의 핵심 인터페이스다. 애플리케이션은 이 인터페이스를 직접 사용할 수 있다.
> 하지만 API로 사용되는 것이 주요 용도는 아니다.
> 보통, 애플리케이션에서 TransactionTemplate이나 AOP를 통한 트랜잭션 경계설정 방식을 통해 사용된다.
 
* 이를 통해DataSourceTransactionManager가 어떤 용도로 사용되는지 파악할 수 있다.
* TransactionTemplate과 트랜잭션 AOP에서 참조한다고 했으니, 이 두가지를 지원하는 빈에 DI 된다는 것도 알 수 있다.

### 프로퍼티 분석
* 인터페이스를 통해 DataSourceTransactionManager 빈이 어떤 기능을 제공하는지, 어디서 사용되는지 파악했다.
* 빈의 클래스의 설정 방법은 프로퍼티 분석을 통해 이루어진다.
* 빈 클래스의 활용 방법, 확장 방법, 디폴트와 다른 동작을 가능하게 하는 방법에 관한 지식은 대부분 클래스의 프로퍼티를 살펴보면 알 수 있다.

### DI/확장 포인트 분석
* 빈 클래스 프로퍼티 중 인터페이스 타입의 프로퍼티를 보면 해당 구현 클래스의 확장 포인트로 볼 수 있다.
* 해당 인터페이스를 구현하는 다른 빈을 주입함으로써 기존에는 없던 기능을 확장할 수 있다.


***
## IoC 컨테이너 DI
* DispatcherServlet과 마찬가지로 IoC/DI 컨테이너, 즉 애플리케이션 컨텍스트도 그 자체로 빈은 아니지만 DI를 받는다.
* 직접 프로퍼티를 사용해 설정할 수는 없지만 자신의 확장 포인트 인터페이스를 구현한 빈을 찾아 스스로 DI한다.(엄밀히 말하면 DL이다)

### BeanPostProcessor
* 실제 빈 오브젝트를 생성하는 시점에서 사용된다.

```java
public interface BeanPostProcessor {
	default Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
		return bean;
	}
	default Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
		return bean;
	}
}
```

* BeanPostProcessor 인터페이스는 위와 같이 정의되어 있다.
* postProcessBeforeInitialization() 메소드는 빈 오브젝트가 생성되고, 아직 초기화 메소드가 호출되기 전에 실행된다.
* 만들어진 빈 오브젝트와 이름을 제공해주는데, 원한다면 주어진 빈 오브젝트를 바꿔치기할 수 있다.
* BeanPostProcessor : @Autowired, @Inject등 어노테이션을 이용해 빈 의존관계 적용, AOP의 동작 원리인 자동 프록시 생성기 등도 해당 인터페이스를 구현한 빈이다.

### BeanFactoryPostProcessor
* 빈 설정 메타데이터가 준비된 시점에서 사용된다.
* 빈 오브젝트가 아니라 빈 팩토리에 대한 후처리를 가능하게 하는 확장 포인트 인터페이스이다.

```java
public interface BeanFactoryPostProcessor {
	void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException;

}
```

* 컨테이너는 빈팩토리 후처리기가 있으면 postProcessBeanFactory()에 빈의 모든 메타정보를 담고있는 ConfigurableListableBeanFactory 타입의 오브젝트를 전달해주는데, 바로 컨테이너 자신을 보내준다.
* 이를 이용해 등록된 빈의 메타정보 자ㄹ체를 조작할 수 있다.
* @Configuration 태그가 붙은 클래스를 통해 빈을 등록할 수 있는 이유는 BeanFactoryPostProcessor가 있기 때문이다.
* ConfigurationClassPostProcessor가 @Configuration, @Bean이 붙은 클래스 정보를 이용해 새로운 빈을 추가해주는 기능을 담당한다.
---
layout: post
title: 스프링의 트랜잭션 속성
categories: [Spring]
tags: [Spring Framework, 토비의 스프링]
description: Spring의 트랜잭션
fullview: false
comments: true
---

**트랜잭션은 더 이상 쪼갤 수 없는 최소 단위의 작업**이다. 이번 챕터는 토비의 스프링 3.1의 6.6 트랜잭션 속성을 읽으며 `DefaultTransactionDefinition`인터페이스 내 트랜잭션의 동작방식에 영향을 줄 수 있는 부분에 대해 알아본다.

### 트랜잭션 전파
트랜잭션 전파(transaction propagation)란 트랜잭션의 경계에서 이미 진행 중인 트랜잭션이 있을 때 또는 없을 때 어떻게 동작할 것인가를 결정하는 방식이다. 대표적으로 설정할 수 있는 트랜잭션 전파 속성은 아래와 같다 : 

#### `REQUIRED`   

가장 많이 사용되는 트랜잭션 전파 속성이다. 진행 중인 트랜잭션이 없으면 새로 시작하고, 이미 시작된 트랜잭션이 있으면 이에 참여한다.   
`PROPAGATION_REQUIRED` 트랜잭션 전파 속성을 갖는 코드는 다양한 방식으로 결합해 하나의 트랜잭션으로 구성할 수 있다. `DefaultTranscationDefinition`의 트랜잭션 전파 속성이다.

#### `SUPPORTS`
이미 시작된 트랜잭션이 있으면 참여하고, 그렇지 않을 경우 트랜잭션 없이 작업을 진행한다.  
트랜잭션이 없지만 해당 경계 안에서 Connection 혹은 Session 등을 공유 할 수 있다.

#### `MANDATORY`
이미 시작된 트랜잭션이 있으면 시작하고, 없으면 예외를 발생시킨다.  
혼자서 독립적으로 트랜잭션을 진행하면 안되는 경우에 사용한다.


#### `REQUIRES_NEW`
항상 새로운 트랜잭션을 시작한다. 즉, 앞에서 시작된 트랜잭션이 있든 없든 상관없이 항상 새로운 트랜잭션을 만들어 독자적으로 동작한다. 독립적인 트랜잭션이 보장되어야 하는 코드에 적용할 수 있다.  

#### `NOT_SUPPORTED`
이 속성을 사용하면 트랜잭션 없이 동작하도록 만들 수 있다. 진행중인 트랜잭션이 있어도 무시한다.  
트랜잭션 경계설정은 보통 AOP를 이용해 한 번에 많은 메소드를 동시에 적용하는 방법을 사용한다. 그런데 그 중에서 특별한 메소드만 트랜잭션 적용에서 제외하려면? 물론 포인트컷 표현식에서 해당 메소드만 제외시키면 되지만, 되려 포인트컷이 복잡해 질 수 있다. 그래서 차라리 모든 메소드에 트랜잭션 AOP가 적용되게 하고, 특정 메소드의 트랜잭션 전파 속성만 `PROPAGATION_NOT_SUPPORTED`로 설정해 트랜잭션 없이 동작하게 만드는 편이 낫다.

#### `NEVER`
트랜잭션을 사용하지 않게 강제한다. 이미 진행중인 트랜잭션이 존재하면 안되며, 있을경우 예외를 발생시킨다.

#### `NESTED`
이미 진행중인 트랜잭션이 있으면 중첩 트랜잭션을 시작한다. 중첩 트랜잭션은 트랜잭션 안에 트랜잭션을 만드는 것으로 REQUIRES_NEW와는 다르다.  
부모 트랜잭션의 커밋과 롤백에는 영향을 받지만, 자신의 커밋과 롤백은 부모 트랜잭션에게 영향을 주지 않는다.  
보통 로그 저장시 사용된다.


### 격리수준
모든 DB 트랜잭션은 격리수준(isolation level)을 가지고 있어야 한다. 서버환경에는 여러 개의 트랜잭션이 동시에 진행될 수 있다. 물론 모든 트랜잭션이 순차적으로 진행돼서 다른 트랜잭션의 작업에 독립적인 것이 좋겠지만, 성능이 크게 떨어질 수 밖에 없다. 그렇기 때문에 격리수준을 조정해 가능한 많은 트랜잭션을 동시에 진행시키면서도 문제가 발생하지 않게 하는 제어가 필요하다.   
격리수준은 기본적으로 DB에 설정되어 있지만 JDBC 드라이버 혹은 DataSource에서 재설정 할 수 있으며, 트랜잭션 단위로 격리수준을 조정할 수 있다. `DefaultTranscationDefinition`의 격리수준은 `ISOLATION_DEFAULT`이다. 즉, DataSource에 설정되어있는 디폴트 격리수준을 그대로 따른다는 뜻이다.

### 제한시간
트랜잭션을 수행하는 제한시간(timeout)을 지정할 수 있다. `DefaultTranscationDefinition`의 기본 설정은 제한시간이 없는 것이다. 제한시간은 직접 트랜잭션을 시작할 수 있는 `PROPAGATION_REQUIRED` 혹은 `PROPAGATION_REQUIRES_NEW`와 함께 사용해야만 의미가 있다.

### 읽기전용
읽기전용(read only)로 설정해두면, 트랜잭션 내에서 데이터를 조작하는 시도를 막아줄 수 있다. 또한 데이터 액세스 기술에 따라 성능이 향상될 수도 있다.
`TransactionDefinition` 타입 오브젝트를 사용하면 네가지 속성을 이용해 트랜잭션 동작방식을 제어할 수 있다.


***
## 애노테이션 트랜잭션 속성과 포인트 컷

### 트랜잭션 애노테이션
`@Transactional`애노테이션을 정의한 코드는 아래와 같다. 

```java

@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@Documented
public @interface Transactional {
  @AliasFor("transactionManager")
  String value() default "";

  @AliasFor("value")
  String transactionManager() default "";

  Propagation propagation() default Propagation.REQUIRED;

  Isolation isolation() default Isolation.DEFAULT;

  int timeout() default -1;

  boolean readOnly() default false;

  Class<? extends Throwable>[] rollbackFor() default {};

  String[] rollbackForClassName() default {};

  Class<? extends Throwable>[] noRollbackFor() default {};

  String[] noRollbackForClassName() default {};
}
```

이 애노테이션의 타깃은 메소드와 타입이다. 따라서 메소드, 클래스, 인터페이스에 사용할 수 있다. `@Transactional`애노테이션을 트랜잭션 속성정보로 사용하도록 지정하면, `@Transactional`이 부여된 모든 오브젝트를 자동으로 타깃 오브젝트로 인식한다. 이때 사용되는 포인트컷은 `TransactionAttributeSourcePointcut`이다. 이 애노테이션은 기본적으로 트랜잭션 속성을 정의하지만, 동시에 포인트컷의 자동등록에도 사용된다. 

#### 트랜잭션 속성을 이용하는 포인트 컷
`TransactionInterceptor`는 메소드 이름 패턴을 통해 부여되는 일괄적인 트랜잭션 속성정보 대신, `@Transactional`에노테이션의 엘리먼트에서 트랜잭션 속성을 가져오는 `AnnotationTransactionAttributeSource`를 사용한다. 동시에 포인트 컷도 `@Transactional`을 통한 트랜잭션 속성정보를 참조하도록 만든다. `@Transactional`로 트랜잭션 속성이 부여된 오브젝트라면 포인트컷의 선정 대상이기도 하기 때문이다.  
이 방식을 이용하면 **포인트컷과 트랜잭션 속성을 애노테이션 하나로 지정할 수 있다.**

#### 대체정책
트랜잭션 부가기능 적용 단위는 메소드다. 따라서 메소드마다 `@Transactional`을 부여하고, 속성을 지정할 수 있다. 이렇게 하면 유연하게 속성 제어를 할 수 있겠지만, 코드는 지저분해지고 동일한 속성정보를 가진 애노테이션을 반복적으로 메소드마다 부여해주는 불상사가 발생할 수 있다.  
그래서 스프링은 `@Transactional`을 적용할 때 4단계의 **대체 정책(fallback)**을 이용하게 해준다.  메소드의 속성을 확인할 때,  
1. 타깃 메소드  
2. 타깃 클래스  
3. 선언 메소드  
4. 선언 타입 (클래스, 인터페이스)  
의 순서에 따라 `@Transactional`이 적용되었는지 차례로 확인하고, 가장 먼저 발견된느 속성정보를 사용하게 하는 방법이다.  가장 먼저 타깃의 메소드에 `@Transactional`이 있는지 확인한다. 부여되어 있다면 이를 속성으로 사용하고, 없으면 다음 대체 후보인 타깃 클래스에 부여된 `@Transactional`애노테이션을 찾는다. 타깃 클래스의 메소드 레벨에는 없었지만, 클래스 레벨에 `@Transactional`이 존재한다면 이를 메소드의 트랜잭션 속성으로 사용한다. 이런식으로 메소드가 선언된 타입까지 단계적으로 확인하여 `@Transactional`이 발견되면 적용하고, 끌까지 발견되지 않으면 해당 메소드는 트랜잭션 적용 대상이 아니라고 판단한다.   
`@Transactional`을 사용하면 대체 정책을 잘 활용하여 애노테이션 자체는 최소한으로 사용하면서도 세밀한 제어가 가능하다. 




*** 
참고 자료
1. [토비의 스프링 3.1 6장 -- AOP](http://www.kyobobook.co.kr/product/detailViewKor.laf?mallGb=KOR&ejkGb=KOR&barcode=9788960773431)

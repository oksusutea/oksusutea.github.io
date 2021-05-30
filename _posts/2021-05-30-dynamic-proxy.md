---
layout: post
title: 프록시, 프록시 패턴, 데코레이터 패턴 그리고 다이내믹 프록시
categories: [Spring]
tags: [Spring Framework, 토비의 스프링]
description: 트랜잭션 경계설정 코드를 비즈니스 로직 코드에서 분리할 떄 적용한 기법
fullview: false
comments: true
---

## 부가 기능 구현의 분리

<p style="text-align:center;">
<img src="https://gunju-ko.github.io//assets/img/posts/toby-spring/aop/AOP-3.1.png">
</p>

트랜잭션이라는 기능은 비즈니스 로직과는 성격이 아예 다르기 때문에 적용 사실 자체를 밖으로 분리할 수 있다. 이렇게 부가기능을 담은 클래스는 중요한 특징이 있다. **부가기능 외 나머지 모든 기능은 원래 핵심 기능을 가진 클래스로 위임**해주어야 한다. 핵심기능은 부가기능을 가진 클래스의 존재를 모른다.  
이렇게 구성했더라도, 클라이언트가 핵심기능을 가진 클래스를 직접 사용해버리면 부가기능이 적용될 기회가 없다. 그래서 인터페이스를 통해서만 핵심기능을 사용하도록 하고, 부가기능을 구현한 클래스가 중간에 끼어들어서 요청을 처리하도록 해야 한다.  
이렇게 마치 **자신이 클라이언트가 사용하려고 하는 실제 대상인 것처럼 위장해서 클라이언트의 요청을 받아주는 것을 대리자, 대리인과 같은 역할을 한다고 해서 프록시라고 부른다**. 프록시를 통해 최종 요청을 위임받아 처리하는 오브젝트를 타깃, 또는 실체라고 부른다.  
프록시는 사용 목적에 따라 두가지로 구분 할 수 있다. 

* 클라이언트가 타깃에 접근하는 방법을 제어
* 타깃에 부가적인 기능을 부여

### 데코레이터 패턴
데코레이터 패턴은 타깃에 부가적인 기능을 **런타임**시 다이내믹하게 부여해주기 위해 프록시를 사용하는 패턴을 말한다.   
프록시로서 동작하는 각 데코레이터는 위임하는 대상에도 인터페이스로 접근하기 때문에, 자신이 최종 타깃으로 위임하는지, 혹은 다음 단계의 데코레이터 프록시로 위임하는지 알지 못한다.  

* 자바 내 실제 예시 : 
	* IO패키지의 InputStream, OutputStream 구현 클래스

데코레이터 패턴은 타깃의 코드를 손대지 않고, 클라이언트가 호출하는 방법도 변경하지 않은 채로 새로운 기능을 추가할 때 유용한 방법이다.

### 프록시 패턴
일반적으로 사용하는 프록시라는 용어와 디자인 패턴에서 프록시 패턴은 구분할 필요가 있다. 
* 프록시 : 클라이언트와 사용 대상 사이 대리 역할을 맡은 오브젝트를 두는 방법을 총칭
* 프록시 패턴 : 프록시를 사용하는 방법 중에서 타깃에 대한 접근 방법을 제어하려는 목적을 가진 경우

프록시 패턴의 프록시는 타깃의 기능을 확장하거나 추가하지 않는다. 대신 클라이언트가 타깃에 접근하는 방식을 변경해준다.  타깃 오브젝트를 생성하기 복잡하거나, 당장 필요하지 않은 경우에는 꼭 필요한 시점까지 오브젝트를 생성하지 않는 편이 좋다. 이럴 때 프록시를 이용해 생성을 최대한 늦출 수 있다.  
또한, 원격 오브젝트를 사용하는 경우에도 프록시를 사용하면 편리하다. 원격 오브젝트에 대한 프록시를 만들어두고, 클라이언트는 마치 로걸에 존재하는 오브젝트를 쓰는 것처럼 프록시를 사용하게 할 수 있다. 
추가로, 타깃에 대한 접근권한을 제어하기 위해 프록시 패턴을 사용할 수 있다. Collections의 unmodifiableCollection()을 통해 만들어지는 오브젝트가 전형적인 접근권한 제어용 프록시라고 볼 수 있다.  
이렇게 프록시 패턴은 타깃의 기능 자체에는 관여하지 않으며 접근하는 방법을 제어해주는 프록시를 이용하는 것이다. 다만, 프록시는 코드에서 자신이 만들거나 접근할 타깃 클래스 정보를 알고 있는 경우가 많다 생성을 지연하는 프록시라면 구체적인 생성 방법을 알아야 하기 때문에 타깃 클래스에 대한 직접적인 정보를 알아야 한다.  
<br/>

### 다이내믹 프록시
자바에는 `java.lang.reflect` 패키지 안에 프록시를 손쉽게 만들 수 있도록 지원해주는 클래스들이 있다. 기본적인 아이디어는 목 프레임워크와 비슷하며, 일일이 프록시 클래스를 정의하지 않고도 몇가지 API를 이용해 프록시처럼 동작하는 오브젝트를 다이내믹하게 생성하는 것이다.

#### 프록시의 구성과 프록시 작성의 문제점
* 타깃과 같은 메소드를 구현하고 있다가 메소드가 호출되면 타깃 프로젝트로 위임한다.
* 지정된 요청에 대해 부가 기능 수행

```java
public class UserServiceTx implements UserService {
@Override
    public void add(User user) {
        userService.add(user);
    }

    @Override
    public void upgradeLevels() {

        TransactionStatus status = transactionManager.getTransaction(new DefaultTransactionDefinition());
        try {
            userService.upgradeLevels();
            transactionManager.commit(status);
        } catch(RuntimeException e){
            transactionManager.rollback(status);
            throw e;
        }
    }
}
```
앞서 작성한 코드에서 볼 수 있듯이, 프록시의 역할은 **위임과 부가작업** 두가지로 구분할 수 있다. 여기서 왜 프록시를 만들기 번거로운지 알 수 있다.
* 타깃의 인터페이스를 구현하고, 위임하는 코드를 작성하기 번거롭다. 부가기능이 필요없는 코드도 구현해서 타깃으로 위임하는 코드로 일일이 만들어주어야 한다.
* 부가기능 코드가 중복될 수 있다.

두번째 문제는 중복 메소드를 추출하여 분리할 수도 있을 것 같지만, 첫번째 문제인 인터페이스 메소드의 구현과 위임 문제는 간단해보이지 않는다. 이런 문제를 해결하는데 유용한 것이 JDK의 다이내믹 프록시ㅇ다.

#### 리플렉션
다이내믹 프록시는 리플렉션 기능을 이용해 프록시를 만들어준다. 리플랙션은 자바의 코드 자체를 추상화해서 접근하도록 만든 것이다. 
자바의 모든 클래스는 그 클래스 자체의 구성정보를 담은 `Class`타입의 오브젝트를 하나씩 갖고 있다. '클래스이름.class' 혹은 getClass()메소드를 호출하면 클래스 정보를 담은 Class 타입의 오브젝트를 가져올 수 있다. 이 클래스 오브젝트를 이용하면, 클래스 코드에 대한 메타정보를 가져오거나 오브젝트를 조작할 수 있다.

 * 클래스의 이름은 무엇이고, 어떤 클랙스를 상속하고, 어떤 인터페이스를 구현하였고, 어떤 필드를 갖고 있고, 각각의 타입은 무엇인지, 메소드는 어떤 것을 정의했고, 메소드의 파라미터와 리턴 타입은 무엇인지
 * 더 나아가 오브젝트 필드의 값을 읽고 수정할 수 있고, 원하는 파라미터 값을 이용해 메소드 호출도 가능

메소드에 대한 정보는 아래와 같은 코드로 가져올 수 있다.

```java
	Method lengthMethod = String.class.getMethod("length");
```
스트링이 가진 메소드 중에서 "length"라는 이름을 갖고 있고, 파라미터는 없는 메소드의 정보를 가져오는 것이다. `java.length.reflect.Method` 인터페이스는 메소드에 대한 자세한 정보 뿐 아니라, 이를 이용해 특정 오브젝트의 메소드를 실행할 수 있다. 바로 Method 인터페이스에 정의된 `invoke()`메소드를 이용하면 된다. 이 메소드는 메소드를 실행시킬 대상 오브젝트와, 파라미터 목록을 받아 메소드를 호출한 뒤, 그 결과를 Object 타입으로 돌려준다. lenth() 메소드는 아래와 같이 실행할 수 있다.

```java
	int length = lengthMethod.invoke(name);
```

***

### 프록시 클래스 적용 예시

프록시를 적용할 간단한 타깃 클래스와 인터페이스, 그리고 프록시는 아래와 같다.

```java
package com.example.demo.proxy;

public interface Hello {
  String sayHello(String name);
  String sayHi(String name);
  String sayThankyou(String name);
}
```

타깃 클래스 : 

```java
package com.example.demo.proxy;

public class HelloTarget implements Hello {

  @Override
  public String sayHello(String name) {
    return "Hello " + name;
  }

  @Override
  public String sayHi(String name) {
    return "Hi " + name;
  }

  @Override
  public String sayThankyou(String name) {
    return "Thank You " + name;
  }
}
```

프록시 클래스 : 

```java
package com.example.demo.proxy;

public class HelloUppercase implements Hello{

  Hello hello;

  public HelloUppercase(Hello hello) {
    this.hello = hello;
  }

  @Override
  public String sayHello(String name) {
    return hello.sayHello(name).toUpperCase();
  }

  @Override
  public String sayHi(String name) {
    return hello.sayHi(name).toUpperCase();
  }

  @Override
  public String sayThankyou(String name) {
    return hello.sayThankyou(name).toUpperCase();
  }
}
```
이 프록시는 프록시 적용의 일반적인 문제점 두 가지를 모두 갖고 있다.
* 인터페이스의 모든 메소드를 구현해 위임 처리
* 부가기능인 리턴 값을 대문자로 바꾸는 기능이 모든 메소드에 중복 처리

### 다이내믹 프록시 적용
이제 앞서 만든 프록시 클래스를 다이내믹 프록시를 이용해 만들어보자.

<p style="text-align:center">
<img src="https://gunju-ko.github.io//assets/img/posts/toby-spring/aop/AOP-3.2.png">
</p>

다이내믹 프록시는 **프록시 팩토리에 의해 런타임시 다이내믹하게 만들어지는 오브젝트**다. 타깃의 인터페이스와 같은 타입으로 만들어진다. 클라이언트는 다이내믹 프록시 오브젝트를 타깃 인터페이스를 통해 사용할 수 있다. 이 덕분에 프록시를 만들 때 인터페이스를 모두 구현해가면서 클래스를 정의할 필요가 사라졌다. **프록시 팩토리에게 인터페이스 정보만 제공해주면, 해당 인터페이스를 구현한 클래스의 오브젝트를 자동으로 만들어주기 때문**이다.  

다이내믹 프록시가 인터페이스 구현 클래스의 오브젝트는 만들어주지만, 프록시로서 필요한 부가기능 코드는 직접 작성해야 한다. 부가 기능은 프록시 오브젝트와 독립적으로 InvocationHandler를 구현한 오브젝트에 담는다.  이 인터페이스는 아래 메소드 한 개만 가진 간단한 인터페이스이다.

```java
public Object invoke(Object proxy, Method method, Obejct[] args)
```
이 메소드는 리플렉션의 Method 인터페이스를 파라미터로 받는다. 메소드를 호출할 때 전달되는 파라미터도 args로 받는다. 다이내믹 프록시 오브젝트는 클라이언트의 모든 요청을 리플렉션 정보로 변환해 InvocaionHandler 구현 오브젝트의 Invoke() 메소드로 넘긴다. 타깃 인터페이스의 모든 메소드 요청이 하나의 메소드로 집중되기 때문에, 중복되는 기능을 효과적으로 제공할 수 있다.   
InvocationHandler 구현 오브젝트가 타깃 오브젝트 레퍼런스를 갖고 있다면, 리플렉션을 이용해 간단히 위임 코드를 만들어 낼 수 있다.  
Hello 인터페이스를 제공하면서, 프록시 팩토리에게 다이내믹 프록시를 만들어달라고 요청하면 Hello 인터페이스의 모든 메소드를 구현한 오브젝트를 생성해준다. InvocationHandler 인터페이스를 구현한 오브젝트를 재공해주면, 다이내믹 프록시가 받는 모든 요청을 invoke() 메소드로 보내준다. Hello 인터페이스가 아무리 많더라도 invoke() 메소드 하나로 처리 할 수 있다.

<p style="text-align:center">
<img src="https://gunju-ko.github.io//assets/img/posts/toby-spring/aop/AOP-3.2-2.png">
</p>
 
 이제 다이내믹 프록시를 만들어보자. 다이내믹 프록시로부터 메소드 호출 정보를 받아 처리하는 InvocationHandler를 만든다.
 
 ```java
 public class UppercaseHandler implements InvocationHandler {
  Hello target;

  public UppercaseHandler(Hello target){
    this.target = target;
  }

  @Override
  public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
    String ret = (String) method.invoke(target, args);  //타깃으로 위임. 인터페이스의 모든 메서드 모두 적용된다.
    return ret.toUpperCase(); //부가기능 적용
  }
}
 ```
 다이내믹 프록시로부터 요청을 전달받으려면 InvocationHandler를 구현해야 한다. 메소드는 invoke() 하나 뿐이다. 다이내믹 프록시가 클라이언트로부터 받는 모든 요청은 invoke() 메소드로 전달된다. 그럼 이 Handler를 사용하고, Hello 인터페이스를 구현하는 프록시는 어떻게 만들까? 아래와 같이 작성하면 된다.
 
 ```java
 Hello proxiedHello = (Hello) Proxy.newProxyInstance(
        getClass().getClassLoader(),
        new Class[] {Hello.class},
        new UppercaseHandler(hello));
 ```
 프록시 생성은 Proxy 클래스의 `netProxyInstance()` 스태틱 팩토리 메소드를 이용하면 된다. 이제 각각의 파라미터에 대해 알아보자.
 * 첫번째 파라미터 : 클래서 로더 제공, 다이내믹 프록시가 정의되는 클래스 로더를 지정
 * 두번째 파라미터 : 다이내믹 프록시가 구현해야 할 인터페이스, 하나 이상의 인터페이스도 구현할 수 있어 배열을 사용한다.
 * 세번째 파라미터 : 부가기능과 위임 관련 코드를 담고 있는 InovacationHanlder 구현 오브젝트

 newProxyInstance()에 의해 만들어지는 다이내믹 프록시 오브젝트는 파라미터로 제공한 Hello 인터페이스를 구현한 클래스의 오브젝트이기 때문에, Hello 타입으로 캐스팅이 가능하다.  
 
 * 주의 : 리플렉션 메소드인 Method.invoke()를 이용해 타겟의 오브젝트 메소드를 호출할 때에는, 타겟 오브젝트에서 발생하는 예외가 `InvocationTargetException`으로 한 번 포장돼서 전달된다. 따라서 일단 InvocationTargetException으로 받은 후, getTargetException() 메소드로 중첩되어 있는 예외를 가져와야 한다.
 
#### 다이내믹 프록시의 장점 
  
 타깃 인터페이스의 메소드가 늘어난다면? 기존 방법은 구현 메소드를 추가해주어야 하지만, 다이내믹 프록시는 코드 변경이 없다. 그리고 타깃의 종류에 상관없이 적용이 가능하기 때문에 유연하다.
 
***
### 다이내믹 프록시를 위한 팩토리 빈
다이내믹 프록시 오브젝트는 일반적인 스프링의 빈으로 등록할 수 없다. 스프링의 빈은 기본적으로 클래스 이름과 프로퍼티로 정의된다.     
 스프링은 내부적으로 리플렉션 API를 이용해서 빈 정의에 나오는 클래스 이름을 가지고 빈 오브젝트를 생성한다. 문제는 다이내믹 프록시 오브젝트는 이런 식으로 프록시 오브젝트가 생성되지 않는다는 점이다. 사실 다이내믹 프록시 오브젝트의 클래스가 어떤 것인지 알 수 없다. 클래스 자체도 내부적으로 다이내믹하게 새로 정의해서 사용하기 때문이다.  
 사전에 프록시 오브젝트의 클래스 정보를 미리 알아내 스프링 빈에 정의할 방법이 없다. 다이내믹 프록시는 Proxy 클래스의 newProxyInstance()라는 스태틱 팩토리 메소드를 통해서만 만들 수 있다.
 
#### 팩토리 빈
스프링은 클래스 정보를 가지고 디폴트 생성자를 통해 오브젝트를 만드는 방법 외 빈을 만들 수 있는 여러가지 방법을 제공한다. 대표적으로 팩토리 빈을 이용한 빈 생성 방법이 있다. **팩토리 빈이란 스프링을 대신해 오브젝트의 생성 로직을 담당하도록 만든 특별한 빈**을 말한다.  
팩토리 빈을 만드는 방법도 여러 가지가 있는데, 가장 간단한 방법은 FactoryBean 인터페이스를 구현하는 것이다.

```java
public interface FactoryBean<T> {
    @Nullable
    T getObject() throws Exception;	// 빈 오브젝트를 생성해서 돌려준다.

    @Nullable
    Class<?> getObjectType();		//생성되는 오브젝트의 타입을 알려준다.

    default boolean isSingleton() {		//싱글톤 오브젝트 여부를 알려준다.
        return true;
    }
}
``` 
FactoryBean 인터페이스를 구현한 클래스를 스프링의 빈으로 등록하면 팩토리 빈으로 동작한다. 빈의 클래스로 등록된 팩토리 빈은 빈 오브젝트를 생성하는 과정에서만 사용된다.

* 주의 : 스프링은 private 생성자를 가진 클래스도 빈으로 등록해주면 리플렉션을 이용해 오브젝트로 만들어준다. 리플렉션을 private로 선언되 접근 규약을 위반할 수 있는 강력한 기능이 있기 때문이다.

* 드물지만 팩토리 빈이 만들어주는 빈 오브젝트가 아니라 팩토리 빈 자체를 가져오고 싶을 경우도 이따. 이 때 '&'를 빈 이름 앞에 붙여주면 팩토리 빈 자체를 돌려준다.

```java
@Test
    public void getFactoryBean() {
        Object factory = applicationContext.getBean("&message");
        assertThat(factory instanceof MessageFactoryBean, is(true));
    }
```

#### 다이내믹 프록시를 만들어주는 팩토리 빈
Proxy의 newProxyInstance() 메소드를 통해서만 생성이 가능한 다이내믹 프록시 오브젝트는 일반적인 방법으로는 스프링의빈으로 등록할 수 없다. 대신 팩토리 빈을 사용하면, 다이내믹 프록시 오브젝트를 스프링의 빈으로 만들어줄 수 있다. 팩토리 빈의 getObject() 메소드에 다이내믹 프록시 오브젝트를 만들어주는 코드를 넣으면 되기 때문이다.

***

### 프록시 팩토리 빈 방식의 장점과 한계
* 프록시 팩토리 빈의 재사용:
	* 하나 이상의 팩토리 빈을 동시에 여러개 등록해도 상관없다. 팩토리 빈이기 때문에 각 빈의 타입은 타깃 인터페이스와 일치하다.
	* 프록시 팩토리 빈을 이용하면 프록시 기법을 아주 빠르고 효과적으로 이용할 수 있다.
#### 프록시 팩토리 빈 방식의 장점
* 다이내믹 프록시를 이용하면 타깃 인터페이스를 구현하는 프록시 클래스를 일일이 만들어주지 않아도 된다.
* 하나의 핸들러 메소드로 수많은 메소드에 부가기능을 구현해줄 수 있다.
* 다이내믹 프록시에 팩토리 빈을 이용한 DI 까지 더해주면 번거로운 다이내믹 프록시 생성 코드도 제거 할 수 있다.

#### 프록시 팩토리 빈의 한계
* 프록시를 통해 타깃에 부가기능을 제공하는 것은 메소드 단위로 일어난다. 하나의 클래스 안에 여러 개의 메소드에 부가기능을 한 번에 제공하는건 어렵지 않게 가능했다. 하지만 한 번에 여러 개의 클래스에 공통적인 부가기능을 제공하는 일은 불가능하다.
* 하나의 타깃에 여러 개의 부가기능을 적용할 필요가 있다면 거의 비슷한 프록시 팩토리 빈의 설정이 중복되는 것을 막을 수 없다.

***
참고자료 : 
1. [토비의 스프링 3.1 6장 AOP - 6.1~6.3](http://www.kyobobook.co.kr/product/detailViewKor.laf?mallGb=KOR&ejkGb=KOR&barcode=9788960773431)
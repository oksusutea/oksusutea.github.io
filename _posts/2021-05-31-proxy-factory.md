---
layout: post
title: 스프링의 프록시 팩토리 빈
categories: [Spring]
tags: [Spring Framework, 토비의 스프링]
description: 다양한 타깃 오브젝트에 프록시를 적용하는 방법
fullview: false
comments: true
---

### ProxyFactoryBean
스프링은 프록시 오브젝트를 생성해주는 기술을 추상화한 팩토리 빈을 제공해준다. 앞서 작성했던 팩토리 빈과는 달리, ProxyFactoryBean은 순수하게 프록시를 생성하는 작업만을 담당하고, 프록시를 통해 제공해줄 부가기능은 별도의 빈에 둘 수 있다.  
ProxyFactoryBean이 생성하는 프록시에서 사용할 부가기능은 MethodInterceptor 인터페이스를 구현해서 만든다. 앞서 다이내믹 프록시를 만들었을 때 사용한 InvocationHandler의 invoke()메소드와는 다른 점이  한가지 있는데, invoke()메소드는 타깃 오브젝트에 대한 정보를 제공하지 않아 InvocationHandler를 구현한 클래스가 타깃 오브젝트를 직접 알고 있어야 했었다. 반면, MethodInterceptor의 invoke() 메소드는 ProxyFactoryBean으로부터 타깃 오브젝트에 대한 정보도 함께 제공받는다. 그 덕분에 타깃 오브젝트에 상관없이 **독립적으로 만들어질 수 있다**.  

```java
package com.example.demo.proxy;

import static org.junit.Assert.*;
import static org.hamcrest.Matchers.is;
import java.lang.reflect.Proxy;
import org.aopalliance.intercept.MethodInterceptor;
import org.aopalliance.intercept.MethodInvocation;
import org.junit.Test;
import org.springframework.aop.framework.ProxyFactoryBean;

public class DynamicProxyTest {
    @Test
    public void simpleProxy() {
        Hello proxiedHello = (Hello) Proxy.newProxyInstance(getClass().getClassLoader(),
            new Class[]{Hello.class},
            new UppercaseHandler(new HelloTarget()));
    }

    @Test
    public void proxyFactoryBean() {
        ProxyFactoryBean pfBean = new ProxyFactoryBean();
        pfBean.setTarget(new HelloTarget());
        pfBean.addAdvice(new UpperCaseAdvice());

        Hello proxiedHello = (Hello) pfBean.getObject();
        assertThat(proxiedHello.sayHello("Toby"), is("HELLO TOBY"));
        assertThat(proxiedHello.sayHi("Toby"), is("HI TOBY"));
        assertThat(proxiedHello.sayThankyou("Toby"), is("THANK YOU TOBY"));
    }

    static class UpperCaseAdvice implements MethodInterceptor {

        @Override
        public Object invoke(MethodInvocation invocation) throws Throwable {
            String ret = (String) invocation.proceed();
            return ret.toUpperCase();
        }
    }
    static interface Hello {
        String sayHello(String name);

        String sayHi(String name);

        String sayThankyou(String name);
    }

    static class HelloTarget implements Hello{

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
            return "Thank you " + name;
        }
    }
}
```
ProxyFactoryBean으로 구연한 프록시는 몇가지 눈에 띄는 차이점이 있다.  
InvocationHandler를 구현했을 떄와 달리, MethodInterceptor를 구현한 UppercaseAdvice에는 타깃 오브젝트 정보가 등장하지 않는다. MethodInvocation은 타깃 오브젝트의 메소드를 실행할 수 있는 기능이 있기 때문에, 부가기능을 제공하는 데에만 집중 할 수 있다.  
MethodInvocation은 일종의 콜백 오브젝트로, proceed() 메소드를 실행하면 타깃 오브젝트의 메소드를 내부적으로 실행해주는 기능이 있다. 그렇다면, MethodInvocation 구현 클래스는 일종의 공유 가능한 템플릿처럼 동작하는 것이다. ProxyFactoryBean은 작은 단위의 템플릿/콜백 구조를 응용해 적용했기 때문에 템플릿 역할을 하는 MethodInvocation을 싱글톤으로 두고 사용할 수 있다.  
또한 ProxyFactoryBean은 addAdvice()메소드를 통해 여러 개의 MethodInterceptor를 추가할 수 있다. 해당 빈 하나만으로 여러 개의 부가기능을 제공해주는 프록시를 만들 수 있다는 뜻이다.
* 어드바이스 : 타깃 오브젝트에 적용하는 부가기능을 담은 오브젝트

#### 포인트컷: 부가기능 적용 대상 메소드 선정 방법
기존 InvocationHandler를 직접 구현했을 때는, 메소드의 이름을 토대로 부가기능 적용 메소드를 선정했다. 그렇다면 ProxyFactoryBean과 MethodInterceptor를 사용하는 방식에서도 메소드 선정 기능을 넣을 수 있을까? MethodInterceptor 오브젝트는 여러 프록시가 공유해서 사용할 수 있고, 타깃 정보를 갖고 있지 않도록 만들었기 때문에 기존 방식으로 적용할 수 없다.  
MethodInterceptor는 InvocationHandler와는 다르게 프록시가 클라이언트로부터 받는 요청을 일일이 전달받을 필요가 없다. MethodInterceptor에는 순수하게 부가기능 제공 코드만 남기고, 대신 프록시에 부가기능을 적용하는 메소드를 선택하는 기능을 넣자.  
스프링의 ProxyFactoryBean 은 두가지 확장 기능인 **부가기능(Advice)**와 **메소드 선정 알고리즘(PointCut)**을 활용하는 유용한 구조를 제공한다.


<p style="text-align:center;">
<img src="https://gunju-ko.github.io//assets/img/posts/toby-spring/aop/AOP-3.4.1.png">
</p>

스프링은 부가기능을 제공하는 오브젝트를 어드바이스라고 부르고, 메소드 선정 알고리즘을 담은 오브젝트를 포인트컷이라고 부른다. 어드바이스와 포인트컷은 모두 프록시에 DI로 주입되어 사용된다. 두가지 모두 여러 프록시에 공유가 가능하도록 만들어지기 때문에 싱글톤 빈으로 등록 할 수 있다.  
프록시는 클라이언트로부터 요청을 받으면 먼저 포인트컷에게 부가기능을 부여할 메소드인지 확인해달라고 요청한다. 포인트컷은 Pointcut 인터페이스를 구현해 만들면 된다. 프록시는 포인트컷으로부터 부가기능을 적용할 대상메소드인지 확인 받으면, MethodInterceptor 타입의 어드바이스를 호출한다. 어드바이스는 JDK의 다이내믹 프록시와 다르게 직접 타깃을 호출하지 않으며, 타깃에 의존하지 않도록 템플릿 구조로 설계되어 있다.

```java
	@Test
    public void pointcutAdvisor() {
        ProxyFactoryBean pfBean = new ProxyFactoryBean();
        pfBean.setTarget(new HelloTarget());

        NameMatchMethodPointcut pointcut = new NameMatchMethodPointcut();
        pointcut.setMappedName("sayH*");

        pfBean.addAdvisor(new DefaultPointcutAdvisor(pointcut, new UpperCaseAdvice()));

        Hello proxiedHello = (Hello) pfBean.getObject();
        assertThat(proxiedHello.sayHello("Toby"), is("HELLO TOBY"));
        assertThat(proxiedHello.sayHi("Toby"), is("HI TOBY"));
        assertThat(proxiedHello.sayThankyou("Toby"), is("Thank you Toby"));
    }
```

기존에 포인트 컷이 필요하지 않았을 때에는 addAdvice로 등록했지만, 현재 포인트컷과 함께 등록할 때에는 어드바이스와 포인트컷을 Advisor 타입으로 묶어 addAdvisor() 메소드를 호출해야 한다. 이 개념을 토대로 아래와 같이 정의할 수 있다.  

* 어드바이저 = 포인트컷(메소드 선정 알고리즘) + 어드바이스(부가기능)



***
참고자료 : 
1. [토비의 스프링 3.1 6장 AOP - 6.1~6.3](http://www.kyobobook.co.kr/product/detailViewKor.laf?mallGb=KOR&ejkGb=KOR&barcode=9788960773431)
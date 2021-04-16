---
layout: post
title: Component와 ComponentScan
categories: [Spring]
tags: [Spring Framework, Spring Core]
description: 스프링 프레임워크에서 빈을 등록하는 범위와 방식
fullview: false
comments: true
---

## @ComponentScan
자동주입과 함께 사용되는 기능으로, **스프링이 직접 클래스를 검색하여 빈으로 등록**해주는 기능이다.
**스캔 위치를 지정**하고, 스캔을 제외하는 **Filter**기능을 통해 스캔을 진행한다.


## @Component
컴포넌트 스캔을 하는 기준이다. base-package에서 `@Component`애노테이션을 가지고 있는 클래스를 스캐닝하여 빈으로 등록한다. `@Component`애노테이션 뿐만 아니라 아래 애노테이션도 `@Component` 애노테이션의 일부임을 유의하자.

* `@Repository` : DB와 관련 작업을 할 때 주로 쓰는 애노테이션
* `@Service` : 서비스단
* `@Controller` : 컨트롤러 
* `@Configuration`: 빈 설정파일 정보

### 컴포넌트 스캔의 범위
컴포넌트 스캔은 보통 `@ComponentScan`이 붙어있는 Configuration부터 스캔을 시작한다.  
스프링 부트 프로젝트에서는 `@SpringBootApplication`에 이미 `@ComponentScan`이 붙어있는 것을 확인할 수 있다. 따라서 `@SpringBootApplication`을 기준으로 스캐닝이 시작된다. **하지만 이 패키지 밖에 있는 클래스는 스캔이 되지 않는다.**


<p style="text-align:center">

<img src="https://user-images.githubusercontent.com/75205849/114976166-7cd25280-9ec0-11eb-8ff9-fa65f94ed90c.png" width="400">
</p>

### 스캔 범위 지정

```java
 
 @Configuration
 @ComponentScan(basePackages = {"spring"}, 
 )
 
 // 
 
 @Configuration
 @ComponentScan(
 	basePackageClasses = TestClasses.classes
 )
```
위와 같이 **basePackages** 혹은 **basePackagesClasses**를 이용하여 scan을 시작할 패키지 or 클래스를 지정할 수 있다. 스프링에서는 type-safety를 위해서 **basePackageClasses**를 추천한다. 

### 스캔 항목에서 제외하기
컴포넌트 스캔을 통해 빈을 등록할 때, `excludeFilters`속성을 사용해 특정 빈은 등록되지 않도록 설정할 수 있다. 설정 방법은 다양하지만, 그 중에서 애노테이션을 이용하여 필터링 하는 방법을 정리해보았다.

아래와 같이 `IgnoreScanning` 애노테이션을 정의한다.

```java
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;
 
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface IgnoreScanning {
}
```

> @Target, @Retention 어노테이션이란?  
> 
> * @Target : 어디에 우리가 만든 어노테이션을 적용할 것인가?
> 	* SOURCE : 컴파일시 사라짐
> 	* CLASS : 컴파일러가 클래스를 참조할 때 까지 유효
> 	* RUNTIME : 컴파일 이후에도 VM을 통해 참조 가능
> * @Retention : 어느 시점까지 어노테이션을 남길 것인가?
> 	* TYPE : 클래스, 인터페이스
>  	* FIELD : 필드
>  	* METHOD : 메서드
>  	* PARAMETER : 파라미터
>  	* CONSTURCTOR : 생성자
>  	* LOCAL_VARIABLE : 지역변수
>  	* ANNOTATION_TYPE : 어노테이션 타입
>  	* PACKAGE : 패키지

그리고 빈으로 등록하지 않을 클래스에 `IgnoreScanning`애노테이션을 붙여준다. 

```java
import org.springframework.stereotype.Service;
 
@Service
@IgnoreScanning
public class AnotherBookService {
}
```

이렇게 등록하게 되면, 기본적으로 `Service`애노테이션이 붙어있어 빈으로 등록된다.

```java
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.FilterType;
import org.springframework.context.annotation.ComponentScan.Filter;
 
@Configuration
@ComponentScan(
        basePackageClasses = ApplicationConfig.class,
        excludeFilters = @Filter(
                type = FilterType.ANNOTATION,
                classes = {IgnoreScanning.class}
        )
)
public class ApplicationConfig {
}

```
위와 같이 Config속성파일에 등록된 `ComponentScan`애노테이션에서 `excludeFilters`속성에 @IgnoreScanning 애노테이션이 붙은 컴포넌트 클래스는 빈으로 등록되지 않도록 설정해주었다. 이렇게 되면 `AnotherBookService`는 결국 빈으로 등록되지 못하게 된다.

### 컴포넌트 스캔의 동작 원리
`@ComponentScan`은 [BeanFactoryPostProcessor](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/beans/factory/config/BeanFactoryPostProcessor.html) 인터페이스를 구현한 [ConfigurationClassPostProcessor](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/annotation/ConfigurationClassPostProcessor.html)에 의해 스캐닝이 진행된다.   
`BeanFactoryProcessor`는 다른 Bean들이 등록되기 전에 먼저 컴포넌트 스캐닝이 되어 미리 빈으로 등릭된다. 그리고 미리 빈으로 등록된 `BeanFactoryProcessor`가 `@Component`어노테이션을 스캔하는 작업을 진행한다.

#### BeanFactoryPostProcessor vs BeanPostProcessor
* BeanFactoryPostProcessor : 빈을 만드는 설명서에 해당하는 설정 파일을 조작
* BeanPostProcessor : 빈 객체에 조작을 진행  
BeanFactoryPostProcessor가 먼저 실행된 후에 BeanPostProcessor가 실행된다.


***
참고자료

1. [스프링 프레임워크 핵심 기술](https://www.inflearn.com/course/spring-framework_core#)
2. [[Spring] 컴포넌트 스캔](https://owin2828.github.io/devlog/2019/12/30/spring-5.html)
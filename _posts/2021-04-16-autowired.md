---
layout: post
title: Autowired
categories: [Spring]
tags: [Spring Framework, Spring Core]
description: 스프링 프레임워크에서 객체를 주입받는 방식과 동작 원리
fullview: false
comments: true
---

## @Autowired
필요한 의존 객체의 **타입**에 해당되는 빈을 찾아 주입한다. default 값은 **true**이기에 의존 주입 객체를 찾지 못하면 애플리케이션 구동을 하지 못한다.

사용 가능 위치 :

* 생성자(스프링 4.3 이상부터는 생략 가능)
* 세터
* 필드

### @Autowired의 다양한 케이스 및 해결방법

#### 1. 의존객체 타입의 빈이 없을 때
아래와 같이 `BookRepository`를 주입받는 `BookService` 클래스가 있다.

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
 
@Service
public class BookService {
 
    BookRepository bookRepository;
 
    @Autowired
    public BookService(BookRepository bookRepository) {
        this.bookRepository = bookRepository;
    }
}
 
public interface BookRepository {
}
```
`BookService`는 `@Service` 애노테이션을 등록해 빈으로 등록하였고, `BookRepository`를 주입하도록 설정하였다. 하지만 `BookRepository`는 어떠한 애노테이션도 붙이지 않았기에 빈으로 등록되지 않고, `BookClass` 인스턴스화를 진행할 경우 아래와 같이 오류가 나는 것을 확인할 수 있다.

```
***************************
APPLICATION FAILED TO START
***************************

Description:

Parameter 0 of constructor in com.cjk.demospring51.BookService required a bean of type 'com.cjk.demospring51.BookRepository' that could not be found.


Action:

Consider defining a bean of type 'com.cjk.demospring51.BookRepository' in your configuration.
```

`BookService` 빈을 생성하기 위해서는 `BookRepository `빈이 IoC 컨테이너에 등록되어 있어야 하는데 찾지 못해 오류가 발생했다는 것이다. 그럼 `setter()`로 주입을 받는 경우는 어떨까?

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
 
@Service
public class BookService {
 
    BookRepository bookRepository;
 
    @Autowired
    public void setBookRepository(BookRepository bookRepository) {
        this.bookRepository = bookRepository;
    }
}
 
public interface BookRepository {
}

```
위 코드로 변경하고 실행하면 동일하게 아래와 같이 오류가 난 것을 알 수 있다.


```
***************************
APPLICATION FAILED TO START
***************************

Description:

Parameter 0 of setBookRepository in com.cjk.demospring51.BookService required a bean of type 'com.cjk.demospring51.BookRepository' that could not be found.


Action:

Consider defining a bean of type 'com.cjk.demospring51.BookRepository' in your configuration.
```
생성자로 주입받던 것과 달리 `setter`로 주입 방식을 변경했기 때문에 `BookService`인스턴스를 생성할 수있는게 맞지만, `@Autowired` 애노테이션에 의해 빈 초기화 직후 **주입 작업이 진행**되기 때문에 `BookRepository` 빈을 찾지 못해 에러가 발생한다.  
정리하자면, `@Autowired` 애노테이션을 통해 주입과정을 처리하는 중에 해당하는 빈의 타입을 못찾거나, 의존성 주입이 불가한 경우에는 **에러가 발생하며 어플리케이션 구동이 불가**하다. 에러를 발생하지 않게 하려면 어떻게 해야할까?

```java

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
 
@Service
public class BookService {
 
    BookRepository bookRepository;
 
    @Autowired(required = false)
    public void setBookRepository(BookRepository bookRepository) {
        this.bookRepository = bookRepository;
    }
}
 
public interface BookRepository {
}
```

위와 같이 `@Autowired`에 `required`속성을 추가한다. 이렇게 설정하게 되면 의존성을 `Optional`로 설정하게 된다. 이렇게 되면 `BookService`는 성공적으로 인스턴스화가 가능해지지만, `BookRepository`를 주입받지 않은 상태로 IoC 컨테이너에 등록된다.

**`required`속성은 setter나 필드주입에 사용할 수 있는 점 유의하자.**

#### 2. 의존객체 타입의 빈이 여러개인 경우

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
 
@Service
public class BookService {
 
    @Autowired
    BookRepository bookRepository;
}
  
public interface BookRepository {
}
 
  
@Repository
public class MyBookRepository implements BookRepository {
}
 
 
@Repository
public class AnotherBookRepository implements BookRepository {
}

```

위 코드에서는 `BookService`가 `BookRepository`를 주입받도록 하였다. 여기서 `BookRepository`를 구현한 클래스가 `MyBookRepository`, `AnotherBookRepository` 이 두가지가 존재하며 두가지 다 빈으로 등록이 되어있을 경우, `BookService`는 어떤 클래스를 주입받을까?

```
***************************
APPLICATION FAILED TO START
***************************

Description:

Field BookRepository in com.cjk.demospring51.BookService required a single bean, but 2 were found:
    - myBookRepository: defined by method ...
    - anotherBookRepository: defined by method ....


Action:

Consider marking one of the beans as @Primary, updating the consumer to accept multiple beans, or using @Qualifier to identify the bean that should be consumed
```

스프링에서는 유니크한 빈을 찾기를 원했지만 두 개의 빈을 찾았다며 오류를 발생시킨다. 스프링 입장에서는 어떤 빈을 원하는지 알 수 없기 때문이다.

#### 1) `@Primary`를 이용하여 빈의 우선순위 주기
`@Primary` 애노테이션을 사용하여 우선적으로 주입할 빈을 지정할 수 있다. 아래와 같이 `MyBookRepository`에 `@Primary`를 붙여보자.

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
 
@Service
public class BookService {
 
    @Autowired
    BookRepository bookRepository;
}
  
public interface BookRepository {
}
 
  
@Repository @Primary
public class MyBookRepository implements BookRepository {
}
 
 
@Repository
public class AnotherBookRepository implements BookRepository {
}
```
이렇게 되면 `BookService`는 우선적으로 `MyBookRepository`를 주입하게 된다. 하지만 이마저도 여러 빈에 `@Primary`를 붙이면 무용지물이 될 수 있으니 유의하자.

#### 2) `@Qualifier`

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
 
@Service
public class BookService {
 
    @Autowired @Qualifier("myBookRepository")
    BookRepository bookRepository;
}

public interface BookRepository {
}

import org.springframework.stereotype.Repository;
 
@Repository
public class MyBookRepository implements BookRepository {
}

import org.springframework.stereotype.Repository;
 
@Repository
public class AnotherBookRepository implements BookRepository {
}
```

위와 같이 주입받을 빈의 id(= 소문자로 시작하는 클래스 이름)를 `@Qualifier`의 속성 값으로 입력한다.  
하지만 이렇게 입력할 경우 type-safe하지 않을 수 있기 때문에 빈을 맞게 입력했는지 한 번 더 확인해주자.

#### 3) 해당하는 타입의 빈을 모두 주입받기

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import java.util.List;
 
@Service
public class BookService {
 
    @Autowired @Qualifier("myBookRepository")
    List<BookRepository> bookRepository;
}

public interface BookRepository {
}

import org.springframework.stereotype.Repository;
 
@Repository
public class MyBookRepository implements BookRepository {
}

import org.springframework.stereotype.Repository;
 
@Repository
public class AnotherBookRepository implements BookRepository {
}
```
위와 같이 `BookService`에 주입받는 객체 `BookRepositry`를 `List`방식으로 받는다면, `MyBookRepository`와 `AnotherBookRepository` 모두 주입 받을 수 있다.


#### 4) 빈 id와 변수 명을 동일하게 설정하기
스프링에서 추천하는 방법은 아니다. 하지만 빈을 주입할 때 IoC 컨테이너는 **빈의 타입을 먼저 찾고, 그 후에 이름과 일치하는 빈이 있는지 찾는다.** 이 동작원리를 이용해서 해결할 수는 있지만.. 이 역시 type-safety하지 않기에 권장하는 방법은 아니다.

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
 
@Service
public class BookService {
 
    @Autowired
    BookRepository mybookRepository;
}

public interface BookRepository {
}

import org.springframework.stereotype.Repository;
 
@Repository
public class MyBookRepository implements BookRepository {
}

import org.springframework.stereotype.Repository;
 
@Repository
public class AnotherBookRepository implements BookRepository {
}
```

위와 같이 작성하면 스프링은 먼저 `BookRepository`와 동일한 타입의 빈이 여러개 있다는 것을 발견하고, 그 후에 변수명과 일치한 빈이 있는지 찾는다. 위에서는 `MyBookRepository`라는 빈이 존재하기에 정상적으로 구동된다.

### @Autowired의 동작 원리

그렇다면 `@Autowired`는 어떻게 동작하는 걸까?
결론부터 말하면, [BeanPostProcessor](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/beans/factory/config/BeanPostProcessor.html)라는 라이프 사이클 인터페이스의 구현체인 [AutowiredAnnotationBeanPostProcessor](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/beans/factory/annotation/AutowiredAnnotationBeanPostProcessor.html)에 의해서 의존성 주입이 이루어진다.  
`BeanPostProcessor`는 스프링 컨테이너 안에서 만든 bean 전/후처리 작업(==라이프 사이클 관련)을 할 수 있도록 만든 인터페이스이다. 코드를 들여다보면 아래와 같이 인터페이스를 제공하는데,

```java
public interface BeanPostProcessor {
    @Nullable
    default Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        return bean;
    }

    @Nullable
    default Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        return bean;
    }
}
```
빈 생성 직전에 실행하는 `postProcessBeforeInitialization `메소드와  빈 생성 직후에 실행하는 `postProcessAfterInitialization` 메소드가 있다.

`AutowiredAnnotationBeanPostProcessor`는 `postProcessBeforeInitialization`를 구현하고(내부적으로는 인터페이스를 구현하고 구현하는 인터페이스가 `BeanPostProcessor`를 구현하는...그런 구조이지만) 빈 초기화 라이프사이클 이전에 `@Autowired`가 붙어있는 빈을 찾아 주입해주는 역할을 한다.정리해보자면, 

* BeanPostProcess(before) => 여기서 주입 작업을 진행한다.
* 빈을 초기화
* BeanPostProcess(after)


***
참고 자료 

1. [스프링 프레임워크 핵심 기술](https://www.inflearn.com/course/spring-framework_core#)
---
layout: post
title: Validation 추상화
categories: [Spring]
tags: [Spring Framework, Spring Core]
description: 스프링 코어에서 사용되는 추상화 -- Validation
fullview: false
comments: true
---

## Validation
애플리케이션에서 사용하는 객체 검증용 인터페이스 `Validator`를 제공한다. 이 `Validator`는 주로 스프링 MVC에서 사용하긴 하지만, 웹 계층 전용 개념은 아니며 어떤 계층(웹, 서비스, 데이터)라도 사용할 수 있는 일반적인 인터페이스이다. 그리고 BeanValidation 전용 여러 어노테이션을 사용해 객체의 데이터를 검증할 수 있다.

```java
public interface Validator {
    boolean supports(Class<?> var1);

    void validate(Object var1, Errors var2);
}

```
위와 같이 `Validator`는 `supports` 메소드와 `validate` 메소드로 구성되어 있다.

* `supports` : 인자로 넘어온 인스턴스의 클래스가, 벨리데이터가 검증할 수 있는 클래스인지 구현
* `validate` : 실제 검증 작업 구현

그럼 이제 `Validator`를 실제로 구현하는 코드를 작성해보자.  
아래 `Event`클래스를 만들었다. 여기서 `title`은 빈 값을 허용하지 않는다.

```java
package me.cjkim.demoSpring51;

public class Event {
    Integer id;
    String title;

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getTitle() {
        return title;
    }

    public void setTitle(String title) {
        this.title = title;
    }
}
```

```java
package me.cjkim.demoSpring51;

import org.springframework.validation.Errors;
import org.springframework.validation.ValidationUtils;
import org.springframework.validation.Validator;

public class EventValidator implements Validator {
    @Override
    public boolean supports(Class<?> aClass) {
        return Event.class.isInstance(aClass);
    }

    @Override
    public void validate(Object o, Errors errors) {
        ValidationUtils.rejectIfEmptyOrWhitespace(errors,"title","notempty","Empty title is not allowed.");

    }
```
`EventValidator` 클래스는 `Validator`인터페이스를 구현하였다. `supports`메소드는 매개변수로 넘어온 클래스가 `Event`클래스인지 검증하고, `validate`메소드는 `ValidationUtils.rejectIfEmptyOrWhitespace`를 이용해 title필드가 공백이거나, 존재하지 않을 시 errors를 담는다. 

```java
package me.cjkim.demoSpring51;

import org.springframework.boot.ApplicationArguments;
import org.springframework.boot.ApplicationRunner;
import org.springframework.stereotype.Component;
import org.springframework.validation.BeanPropertyBindingResult;
import org.springframework.validation.Errors;
import java.util.Arrays;

@Component
public class AppRunner implements ApplicationRunner {

    @Override
    public void run(ApplicationArguments args) throws Exception {
       Event event = new Event();
       EventValidator eventValidator = new EventValidator();
       Errors errors = new BeanPropertyBindingResult(event,"event");

       eventValidator.validate(event,errors);
       System.out.println(errors.hasErrors());

       errors.getAllErrors().forEach(e -> {
           System.out.println("===== error code =====");
           Arrays.stream(e.getCodes()).forEach(System.out::println);
           System.out.println(e.getDefaultMessage());
       });

    }
}
```
`Errors errors = new BeanPropertyBindingResult(event,"event");`는 에러정보를 담는 역할을 한다. 에러 인터페이스 구현체 중 하나이다. 현재는 `Validator`에 대해 공부를 하기위해 직접적으로 사용하였지만, 실무적으로는 스프링 MVC에서 자동으로 넘겨주기 때문에 사용할 일은 없다.

```
true
===== error code =====
notempty.event.title
notempty.title
notempty.java.lang.String
notempty
Empty title is not allowed.
```
title 값이 없어 에러가 발생하며, 스프링에서는 개발자가 생성한 에러코드 외 자동으로 에러메세지를 3개 생성해준 것을 확인할 수 있다. 추가로, 스프링 부트 2.0.5 이상 버전 사용시 Validator 인터페이스를 LocalValidatorFactoryBean으로 자동 등록해준다.

```java
public class Event {

    Integer id;
    @NotEmpty
    String title;

    @NotNull @Min(0)
    Integer limit;

    @Email
    String email;
}
```
getter, setter가 모두 있다고 가정할 때 아래와 같이 Bean Valication 어노테이션을 통해 검증을 할 수 있다.

```java
package me.cjkim.demoSpring51;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.ApplicationArguments;
import org.springframework.boot.ApplicationRunner;
import org.springframework.stereotype.Component;
import org.springframework.validation.BeanPropertyBindingResult;
import org.springframework.validation.Errors;
import org.springframework.validation.Validator;
import java.util.Arrays;

@Component
public class AppRunner implements ApplicationRunner {

    @Autowired
    Validator validator;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        System.out.println(validator.getClass());
        Event event = new Event();
        event.setLimit(-1);
        event.setEmail("aaa");
        Errors errors = new BeanPropertyBindingResult(event, "event");

        validator.validate(event, errors);

        System.out.println(errors.hasErrors());

        errors.getAllErrors().forEach(e -> {
            System.out.println("==== error code ====");
            Arrays.stream(e.getCodes()).forEach(System.out::println);
            System.out.println(e.getDefaultMessage());
        });
    }
}
```

```
class org.springframework.validation.beanvalidation.LocalValidatorFactoryBean
true
==== error code ====
Min.event.limit
Min.limit
Min.java.lang.Integer
Min
반드시 0보다 같거나 커야 합니다.
==== error code ====
NotEmpty.event.title
NotEmpty.title
NotEmpty.java.lang.String
NotEmpty
반드시 값이 존재하고 길이 혹은 크기가 0보다 커야 합니다.
==== error code ====
Email.event.email
Email.email
Email.java.lang.String
Email
이메일 주소가 유효하지 않습니다.
```

Bean도 성공적으로 주입이 되었으며, 에러 메시지를 자동으로 만들어주는 것까지 확인할 수 있다.

***
참고 자료 

1. [스프링 프레임워크 핵심 기술](https://www.inflearn.com/course/spring-framework_core#)
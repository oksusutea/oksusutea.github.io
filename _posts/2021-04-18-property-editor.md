---
layout: post
title: 데이터 바인딩 추상화: PropertyEditor
categories: [Spring]
tags: [Spring Framework, Spring Core]
description: 스프링 코어에서 사용되는 데이터 바인딩 추상화 -- PropertyEditor
fullview: false
comments: true
---

## 데이터 바인딩
* 기술적 관점 : 프로퍼티의 값을 타겟 객체에 설정하는 기능
* 사용자 관점 : 사용자가 입력한 값을 애플리케이션 도메인 객체에 동적으로 할당하는 기능

### 데이터 바인딩을 사용해야 하는 이유
사용자가 입력한 값은 주로 **문자열**이다. 객체가 가지고 있는 다양한 프로퍼티 타입, int, long, Boolean등 심지어 도메인 타입으로 변환하기 위해서 데이터 바인딩을 사용한다. 이를 구현하기 위해 **org.springframework.validation.DataBinder** 인터페이스를 사용한다. 

### ProperyEditor
* 스프링 3.0 이전까지 `DataBiner`가 변환 작업하여 사용하던 인터페이스
* Thread-safe하지 않음
* Object와 String간만 변환이 가능해 사용 범위가 제한적임

```
public class Event {
    private Integer id;

    private String title;

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
import java.beans.PropertyEditorSupport;

public class EventEditor extends PropertyEditorSupport {

    @Override
    public String getAsText() {
        Event event = (Event)getValue();
        return event.getId().toString();
    }

    @Override
    public void setAsText(String text) throws IllegalArgumentException {
        setValue(new Event(Integer.parseInt(text)));
    }
}
```
`PropertyEditor`를 구현하려면 불필요한 메소드까지 다 구현해야 하기 때문에, `PropertyEditorSupport`를 상속하여 구현하는 방식을 사용했다.  

* `setAsText`는 사용자가 입력한 데이터를 `Event`객체로 변환해주는 역할을 한다.
* `getAsText`는 프로퍼티가 받은 객체를 `getValue`메소드를 통해 가져오고, `Event`객체를 문자열 정보로 반환하는 역할을 한다. 


```
package me.cjkim.demoSpring51;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class EventController {
    @GetMapping("/event/{event}")
    public String getEvent(@PathVariable Event event){
        System.out.println(event);
        return event.getId().toString();
    }
}
```
위와 같이 컨트롤러를 구현했을 때,  **event/1**과 같이 요청을 보내면 event객체를 출력한다.   
앞서 말했듯이 `PropertyEditor`는 Thread-safe하지 않다. 이를 해결하기 위해서 `@InitBinder`어노테이션을 사용하도록 한다. 


```java
package me.cjkim.demoSpring51;

import org.springframework.web.bind.WebDataBinder;
import org.springframework.web.bind.annotation.InitBinder;

import java.beans.PropertyEditorSupport;

public class EventEditor extends PropertyEditorSupport {

    @InitBinder
    public void init(WebDataBinder webDataBinder){
        webDataBinder.registerCustomEditor(Event.class, new EventEditor());
    }

    @Override
    public String getAsText(){
        Event event = (Event) getValue();
        return event.getId().toString();
    }

    @Override
    public void setAsText(String text) throws IllegalArgumentException {
        setValue(new Event(Integer.parseInt(text)));
    }
}
```


***
참고 자료 

1. [스프링 프레임워크 핵심 기술](https://www.inflearn.com/course/spring-framework_core#)
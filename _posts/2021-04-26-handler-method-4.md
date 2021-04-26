---
layout: post
title: 핸들러 메소드 (4) - 요청 매개변수(단순 타입)
categories: [Spring]
tags: [Spring Framework, Spring MVC]
description: 핸들러 메소드를 처리하는 방법 - @ModelAttribute
fullview: false
comments: true
---


### `@ModelAttribute`
* 여러 곳(URI 패스, 요청 매개변수, 세션 등)에 있는 단순 타입 데이터를 복합 타입 객체로 받아오거나 해당 객체를 새로 만들 때 사용할 수 있다.
* 생략 가능하다.


컨트롤러 파일 : 

```java
 @PostMapping(value = "/events")
    @ResponseBody
    public Event getEvent(@ModelAttribute Event event){
            return event;
    }
```

위와 같이 기존에 String name, Int limit으로 각 Event 객체의 property 값을 받아왔었는데, `@ModelAttribute`를 사용하면 객체 자체를 바로 받아올 수 있다.

테스트 파일 : 

```java
@Test
    public void eventsTest() throws Exception {
        mockMvc.perform(post("/events")
                    .param("name","cjkim")
                    .param("limit","20"))
                .andDo(print())
                .andExpect(status().isOk())
                .andExpect(jsonPath("name").value("cjkim"));
    }
```



#### 값을 바인딩 할 수 없는 경우 ? 
*`BindException` 발생, 400 에러가 뜬다.

```java
@Test
    public void eventsTest() throws Exception {
        mockMvc.perform(post("/events")
                    .param("name","cjkim")
                    .param("limit","cjkim")) //숫자로 입력해야 하는데 문자로 입력할 경우
                .andDo(print())
                .andExpect(status().isOk())
                .andExpect(jsonPath("name").value("cjkim"));
    }
```

테스트 결과 : 

```
java.lang.AssertionError: Status expected:<200> but was:<400>
Expected :200
Actual   :400
```


#### 바인딩 에러를 직접 다루고 싶을 경우
* `BindingResult`타입의 아규먼트를 바로 오른쪽에 추가한다.

```java
@PostMapping(value = "/events")
    @ResponseBody
    public Event getEvent(@ModelAttribute Event event, BindingResult bindingResult){ //bindingResult변수에 바인딩과 관련된 에러가 담겨온다.

        return event;
    }
```
위와 같이 핸들러 아규먼트에 `BindingResult`만 담아줘도 테스트 결과는 성공인 것을 볼 수 있다. 하지만 `limit`값이 Null로 담기기 때문에, 아래와 같이 작성하여 에러를 확인할 수 있도록 하자.

```java
@PostMapping(value = "/events")
    @ResponseBody
    public Event getEvent(@ModelAttribute Event event, BindingResult bindingResult){ //bindingResult변수에 바인딩과 관련된 에러가 담겨온다.

        if(bindingResult.hasErrors()){
            System.out.println("============================");
            bindingResult.getAllErrors().forEach(c->{
                System.out.println(c.toString());
            });
        }

        return event;
    }
```


#### 바인딩 이후 검증작업을 추가로 하고 싶을 경우
* `@Valid`또는 `@Validated`애노테이션을 사용한다.
	* `@Valid`는 그룹을 지정할 수 없으며, `@Validated`만 그룹 클래스를 설정할 수 있다. 즉 `@Validated`는 스프링이 제공하는 애노테이션으로 그룹 클래스(validation group)을 설정할 수 있다.
* 값이 바인딩 될 수는 있지만, `bindingResult`에 에러로 담겨지게 된다.

이벤트 클래스 : 

```java
package me.cjk.demo;

import javax.validation.constraints.Min;
import javax.validation.constraints.NotBlank;

public class Event {

    interface ValidateLimit {}
    interface ValidateName{}


    private Integer id;

    @NotBlank(groups = ValidateName.class)
    private  String name;

    @Min(value = 0, groups = ValidateLimit.class)
    private  Integer limit;

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Integer getLimit() {
        return limit;
    }

    public void setLimit(Integer limit) {
        this.limit = limit;
    }
}

```


컨트롤러 : 

```java
@PostMapping(value = "/events")
    @ResponseBody
    public Event getEvent(@Validated(Event.ValidateLimit.class) @ModelAttribute Event event, BindingResult bindingResult){ //bindingResult변수에 바인딩과 관련된 에러가 담겨온다.

        if(bindingResult.hasErrors()){
            System.out.println("============================");
            bindingResult.getAllErrors().forEach(c->{
                System.out.println(c.toString());
            });
        }

        return event;
    }
```

위와 같이 `@Validated`에 `ValidateLimit.class`만 지정해주었을 경우 `Event`클래스에 `NotBlank` 유효성은 체크되지 않는다.

***
참고자료

1. [스프링 웹 MVC](https://inf.run/dJFi)
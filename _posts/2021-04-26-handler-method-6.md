---
layout: post
title: 핸들러 메소드 (6) - 세션에 모델 정보 담기
categories: [Spring]
tags: [Spring Framework, Spring MVC]
description: 핸들러 메소드를 처리하는 방법 - @SessionAttributes
fullview: false
comments: true
---

### `@SessionAttributes`
* 모델 정보를 HTTP  세션에 저장해주는 애노테이션이다.
* `HttpSession`을 직접 사용할 수도 있지만, 이 애노테이션이 설정한 이름에 해당하는 모델 정보를 자동으로 세션에 넣어준다.
* `@ModelAttribute`는 세션에 있는 데이터도 바인딩한다.
* 여러 화면(또는 요청)에서 사용해야 하는 객체를 공유할 때 사용한다.


#### HTTPSession을 직접 이용하는 방법

```java
@GetMapping("/events/form")
    public String eventsForm(Model model, HttpSession httpSession){
        Event event = new Event();
        event.setLimit(50);
        model.addAttribute("event", event);
        httpSession.setAttribute("event",event);
        return "events/form";
    }
```

테스트 코드

```java
    @Test
    public void eventForm() throws Exception {
        mockMvc.perform(get("/events/form"))
                .andDo(print())
                .andExpect(view().name("events/form"))
                .andExpect(model().attributeExists("event"))
                .andExpect(request().sessionAttribute("event",notNullValue()))
        ;
    }
```

아래와 같이 직접 요청에서 세션의 `event`객체를 꺼내 출력할 수도 있다.

```java
    @Test
    public void eventForm() throws Exception {
        MockHttpServletRequest request = mockMvc.perform(get("/events/form"))
                .andDo(print())
                .andExpect(view().name("events/form"))
                .andExpect(model().attributeExists("event"))
                .andExpect(request().sessionAttribute("event",notNullValue()))
                .andReturn().getRequest()
        ;

        Object event = request.getSession().getAttribute("event");
        System.out.println(event);
    }
```

### `@SessionAttributes` 사용 예시

```java
package me.cjk.demo;

import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.validation.BindingResult;
import org.springframework.validation.annotation.Validated;
import org.springframework.web.bind.annotation.*;

import java.util.List;

@Controller
@SessionAttributes("event")
public class SampleController {

    @GetMapping("/events/form")
    public String eventsForm(Model model){
        Event event = new Event();
        event.setLimit(50);
        model.addAttribute("event", event); //모델에서 설정한 attribute 중에 SessionAttributes에서 설정한 이름과 동일한 것이 있다면 세션 속성으로 넣는다.
        return "events/form";
    
    
}

```

####  세션 처리 완료를 알리는 방법 -- `SessionStatus`
메소드 아규먼트로 `SessionStatus`를 넣어준다.

```java
 @PostMapping(value = "/events")
    public String getEvent(@Validated @ModelAttribute Event event, BindingResult bindingResult, SessionStatus sessionStatus){

        if(bindingResult.hasErrors()){
            return "/events/form";
        }
        sessionStatus.setComplete(); // 폼 처리가 끝나고 세션이 완료되도록, 세션 데이터를 비우는 역할을 한다.
        return "redirect:/events/list";
    }
```
위와 같이 세션을 사용한 폼 처리하는 곳에서 sessionStatus의 `setComplete`를 사용해 데이터를 비울 수 있다.


***
참고자료

1. [스프링 웹 MVC](https://inf.run/dJFi)
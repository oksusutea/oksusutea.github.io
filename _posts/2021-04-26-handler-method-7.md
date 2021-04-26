---
layout: post
title: 핸들러 메소드 (7) - 멀티 폼 서브밋
categories: [Spring]
tags: [Spring Framework, Spring MVC]
description: 여러 폼에 걸쳐 데이터를 나눠 입력받고 저장하기
fullview: false
comments: true
---

앞서 이름과 제한인원을 입력하는 form.html을 이름만 받는 폼, 제한인원을 받는 폼 두개로 나누어 보자.

form-name.html 파일 : 

```html
<!DOCTYPE html>
<html lang="en" xmlns:th = "http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Create New Event</title>
</head>
<body>
<form action="#" th:action="@{/events/form/name}" method="post" th:object="${event}">
    <p th:if="${#fields.hasErrors('name')}" th:errors="*{name}">Incorrect name</p>
   Name :  <input type="text" title="name" th:field="*{name}" />
    <input type="submit" value="Create"/>

</form>

</body>
</html>
```

form-limit.html 파일 : 

```html
<!DOCTYPE html>
<html lang="en" xmlns:th = "http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Create New Event</title>
</head>
<body>
<form action="#" th:action="@{/events/form/limit}" method="post" th:object="${event}">
    <p th:if="${#fields.hasErrors('limit')}" th:errors="*{limit}">Incorrect limit</p>
    Limit : <input type="text" title="limit" th:field="*{limit}" />
    <input type="submit" value="Create"/>

</form>

</body>
</html>
```

컨트롤러 : 

```java
package me.cjk.demo;

import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.validation.BindingResult;
import org.springframework.validation.annotation.Validated;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.bind.support.SessionStatus;

import java.util.ArrayList;
import java.util.List;

@Controller
@SessionAttributes("event")
public class SampleController {

    @GetMapping("/events/form/name")
    public String eventsFormName(Model model){
        model.addAttribute("event", new Event()); //모델에서 설정한 attribute 중에 SessionAttributes에서 설정한 이름과 동일한 것이 있다면 세션 속성으로 넣는다.
        return "events/form-name";
    }

    @PostMapping(value = "/events/form/name")
    public String eventsFormNameSubmit(@Validated @ModelAttribute Event event,
                                 BindingResult bindingResult){

        if(bindingResult.hasErrors()){
            return "/events/form-name"; //이상있을 경우 이름을 입력하는 페이지로 다시 이동
        }
        return "redirect:/events/form/limit"; // 제한인원 입력하는 페이지로 이동
    }

    @GetMapping("/events/form/limit")
    public String eventsFormLimit(@ModelAttribute Event event, Model model){
        model.addAttribute("event", event); //모델에서 설정한 attribute 중에 SessionAttributes에서 설정한 이름과 동일한 것이 있다면 세션 속성으로 넣는다.
        return "events/form-limit";
    }

    @PostMapping(value = "/events/form/limit")
    public String eventsFormLimitSubmit(@Validated @ModelAttribute Event event,
                                       BindingResult bindingResult,
                                       SessionStatus sessionStatus){

        if(bindingResult.hasErrors()){
            return "/events/form-limit"; //이상있을 경우 제한인원을 입력하는 페이지로 다시 이동
        }
        sessionStatus.setComplete(); // limit까지 입력받으면 세션 데이터를 비운다.
        return "redirect:/events/list"; // 제한인원 입력하는 페이지로 이동
    }

    @GetMapping("/events/list")
    public String getEvents(Model model){
        Event event = new Event();
        event.setName("spring");
        event.setLimit(10);
        List<Event> eventList = new ArrayList<>();
        eventList.add(event);
        model.addAttribute(eventList);
        return "/events/list";
    }
}
```

`/events/form/name`을 요청하게 되면 `events/form-name`의 타임리프 템플릿 파일이 렌더링 된다. 여기서 사용자가 이름을 입력한 후, 서브밋을 하게 되면 `/events/form/name`이 포스트 요청으로 처리가 되는데, 여기서 사용자가 입력한 데이터는 `@ModelAttribute`에 의해 `Event`타입으로 받게 된다. 추가로 `@SessionAttributes`와도 이름이 같기 때문에 이 객체는 세션객체로 등록이 된다. 이름만 등록되어 있는 Event 객체가 세션객체로 등록이 된 상태이다. 여기서 `/events/form/limit`으로 넘어가 limit까지 입력을 정상적으로 하면, 최종적으로 list 요청까지 처리한 것을 볼 수 있다.

#### 정리
위와 같이 멀티 폼에 걸쳐 어떤 데이터를 완성해야 하는 경우 `@SessionAttributes`를 사용할 수 있다.

***
참고자료

1. [스프링 웹 MVC](https://inf.run/dJFi)
---
layout: post
title: 핸들러 메소드 (5) - 폼 서브밋 에러 처리
categories: [Spring]
tags: [Spring Framework, Spring MVC]
description: 핸들러 메소드를 처리하는 방법 - 에러 처리 방법
fullview: false
comments: true
---

### 바인딩 에러시 모델에 담기는 정보
* `Event`
* `BindingResult.event`

타임리프 폼파일 : 

```html
<!DOCTYPE html>
<html lang="en" xmlns:th = "http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Create New Event</title>
</head>
<body>
<form action="#" th:action="@{/events}" method="post" th:object="${event}">
    <p th:if="${#fields.hasErrors('limit')}" th:errors="*{limit}">Incorrect limit</p>
    <p th:if="${#fields.hasErrors('name')}" th:errors="*{name}">Incorrect name</p>
    <input type="text" title="name" th:field="*{name}" />
    <input type="text" title="limit" th:field="*{limit}" />
    <input type="submit" value="Create"/>

</form>

</body>
</html>
```
위와 같이 타임리프를 통해 타입의 유효성 체크를 해줄 수 있다.

### POST / REDIRECT / GET
* POST요청 이후 브라우저를 리프래시 하더라도 중복 폼 서브밋이 발생하지 않도록 `REDIRECT`를 사용하자.

```java
@PostMapping(value = "/events")
    public String getEvent(@Validated @ModelAttribute Event event, BindingResult bindingResult, Model model){

        if(bindingResult.hasErrors()){
            return "/events/form";
        }
        List<Event> eventList = new ArrayList<>();
        eventList.add(event);
        model.addAttribute(eventList);
        return "redirect:/events/list";
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
```

위 코드와 같이 `/events`에서 포스트 처리를 하고 `/events/list`로 넘어간 뒤에 리프레쉬를 하게 되면, `post`요청이 중복 처리 되지 않고 get요청으로 `/events/list`요청을 처리할 수 있다. 여기서는 `redirect`를 이용하였다.

***
참고자료

1. [스프링 웹 MVC](https://inf.run/dJFi)
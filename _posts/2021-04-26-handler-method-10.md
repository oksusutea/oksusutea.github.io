---
layout: post
title: 핸들러 메소드 (10) - Flash Attributes
categories: [Spring]
tags: [Spring Framework, Spring MVC]
description: 리다이렉트 할 때 데이터를 전달하는 두번째 방법
fullview: false
comments: true
---

주로 리다이렉트시에 데이터를 전달할 때 사용한다.
* 데이터가 URI에 노출되지 않는다.
* 임의의 객체를 저장할 수 있다.
* 보통 HTTP 세션을 사용한다. 즉, 세션에 데이터가 담겨진다.


리다이렉트 하기 전 데이터를 HTTP세션에 저장하고, 리다이렉트 요청을 처리 한 다음 그 즉시 제거한다.
`RedirectAttributes`를 통해 사용할 수 있다. 

```java
    @PostMapping(value = "/events/form/limit")
    public String eventsFormLimitSubmit(@Validated @ModelAttribute Event event,
                                       BindingResult bindingResult,
                                       SessionStatus sessionStatus,
                                        RedirectAttributes attributes){

        if(bindingResult.hasErrors()){
            return "/events/form-limit"; //이상있을 경우 제한인원을 입력하는 페이지로 다시 이동
        }
        //sessionStatus.setComplete(); // limit까지 입력받으면 세션 데이터를 비운다.
        attributes.addFlashAttribute("newEvent", event);
        return "redirect:/events/list"; // name과 limit가 primitive type이기 때문에 "redirect:/events/list?name="event.getName()"&limit="event.getLimit()"와 동일하게 전달된다.
    }
```
위와 같이 `RedirectAttributes`의 메소드인 `addFlashAttribute`를 사용하는 것을 볼 수 있다. 그럼 넘겨준 값들은 리다이렉트한 쪽에서 어떻게 사용할까?


#### 받는쪽에서 어떻게 처리해야 할까?
물론`@ModelAttribute`를 통해 값을 가져올 수 있지만, 그렇게 하지 않아도 `Model`에 바로 들어온다. 따로 선언해서 받을 필요가 없다.

```java
@GetMapping("/events/list")
    public String getEvents( Model model){
        Event events = new Event();
        events.setName("spring");
        events.setLimit(10);

        Event newEvent = (Event) model.asMap().get("newEvent");

        List<Event> eventList = new ArrayList<>();
        eventList.add(events);
        eventList.add(newEvent);
        model.addAttribute(eventList);
        return "/events/list";
    }
```


### RedirectAttributes vs Flash Attributes

* `RedirectAttributes`는 리다이렉트시 데이터를 쿼리 파라미터로 전달한다. 그렇기 때문에 데이터는 모두 문자열로 변환이 가능해야 한다. 그렇기에 복합적인 객체를 전달하기에는 조금 어렵다.
* `Flash Attributes`는 데이터가 URI에 노출되지 않는다.
* `RedirectAttributes`는 리다이렉트 받는 쪽에서 메소드 아규먼트에 `@RequestParam`등으로 받아 처리해야 하지만, `FlashAttributes`는 `Model`에서 바로 가져와 사용할 수 있다.

***
참고자료

1. [스프링 웹 MVC](https://inf.run/dJFi)
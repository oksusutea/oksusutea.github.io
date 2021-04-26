---
layout: post
title: 핸들러 메소드 (9) - RedirectAttributes
categories: [Spring]
tags: [Spring Framework, Spring MVC]
description: 리다이렉트 할 때 Model에 들어있는 데이터 중 primitive type 데이터를 URI 쿼리 매개변수에 추가하는 방법
fullview: false
comments: true
---

리다이렉트할 때 기본적으로 `Model`에 들어있는 primitive type 데이터는 URI 쿼리 매개변수에 추가된다.

```java
@PostMapping(value = "/events/form/limit")
    public String eventsFormLimitSubmit(@Validated @ModelAttribute Event event,
                                       BindingResult bindingResult,
                                       SessionStatus sessionStatus,
                                        Model model){

        if(bindingResult.hasErrors()){
            return "/events/form-limit"; //이상있을 경우 제한인원을 입력하는 페이지로 다시 이동
        }
        sessionStatus.setComplete(); // limit까지 입력받으면 세션 데이터를 비운다.
        model.addAttribute("name", event.getName());
        model.addAttribute("limit", event.getLimit());
        return "redirect:/events/list"; // name과 limit가 primitive type이기 때문에 "redirect:/events/list?name="event.getName()"&limit="event.getLimit()"와 동일하게 전달된다.
    }
```

### 스프링 MVC와 스프링 부트의 `RedirectAttributes`속성 차이
* 스프링 부트에서는 이 기능이 기본적으로 비활성화 되어있다.
* `Ignore-default-model-on-redirect`프로퍼티를 사용하여 활성화 할 수 있다.

`WebMvcAutoConfiguration.java` (스프링부트 설정파일)

```java
	@Bean
        public RequestMappingHandlerAdapter requestMappingHandlerAdapter(@Qualifier("mvcContentNegotiationManager") ContentNegotiationManager contentNegotiationManager, @Qualifier("mvcConversionService") FormattingConversionService conversionService, @Qualifier("mvcValidator") Validator validator) {
            RequestMappingHandlerAdapter adapter = super.requestMappingHandlerAdapter(contentNegotiationManager, conversionService, validator);
            adapter.setIgnoreDefaultModelOnRedirect(this.mvcProperties == null || this.mvcProperties.isIgnoreDefaultModelOnRedirect());
            return adapter;
        }
```

`ignoreDefaultModelOnRedirect`가 true로 셋팅이 되어 있기 때문에, 리다이렉트를 할 때 primitive type이 전달되지 않는다. 만일 스프링 부트에서 모델의 primitive type 값을 함께 전달하고 싶다면 `application.properties`에 하기 내역을 작성해주면 된다.

```
spring.mvc.ignore-default-model-on-redirect=false
```
이렇게 설정해주면 리다이렉트 받는 쪽에서 Model에 설정한 attribute 데이터를 가져와 사용할 수 있다.


### Model에서 일부 attribute만 전달하고 싶다면?
만일 Model에 있는 모든 attribute가 아니라, 일부만 redirect하는 쪽에 전달하고 싶다면 `RedirectAttributes`를 사용하면 된다.

```java
    @PostMapping(value = "/events/form/limit")
    public String eventsFormLimitSubmit(@Validated @ModelAttribute Event event,
                                       BindingResult bindingResult,
                                       SessionStatus sessionStatus,
                                        RedirectAttributes attributes){

        if(bindingResult.hasErrors()){
            return "/events/form-limit"; //이상있을 경우 제한인원을 입력하는 페이지로 다시 이동
        }
        sessionStatus.setComplete(); // limit까지 입력받으면 세션 데이터를 비운다.
        attributes.addAttribute("name", event.getName());
        attributes.addAttribute("limit", event.getLimit());
        return "redirect:/events/list"; // name과 limit가 primitive type이기 때문에 "redirect:/events/list?name="event.getName()"&limit="event.getLimit()"와 동일하게 전달된다.
    }
```
핸들러 메소드의 아규먼트로 `RedirectAttributes`를 받아와서 redirect시 전달한 attribute를 추가해주면 된다.

#### 받는쪽에서 어떻게 처리해야 할까?
첫번째로는 일일히 `@RequestParam`을 추가해주는 방법이 있다.

```java
    @GetMapping("/events/list")
    public String getEvents(@RequestParam String name, @RequestParam Integer limit, Model model){
        Event event = new Event();
        event.setName(name);
        event.setLimit(limit);
        List<Event> eventList = new ArrayList<>();
        eventList.add(event);
        model.addAttribute(eventList);
        return "/events/list";
    }
```
위와 같이 핸들러 메소드의 아규먼트로 받아올 매개변수를 지정하여 처리할 수 있다.  

두번째로는, `Event`처럼 복합 객체를 `@ModelAttribute`로 받는 방법이 있다.

```java
package me.cjk.demo;

import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.validation.BindingResult;
import org.springframework.validation.annotation.Validated;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.bind.support.SessionStatus;
import org.springframework.web.servlet.mvc.support.RedirectAttributes;

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
                                       SessionStatus sessionStatus,
                                        RedirectAttributes attributes){

        if(bindingResult.hasErrors()){
            return "/events/form-limit"; //이상있을 경우 제한인원을 입력하는 페이지로 다시 이동
        }
        sessionStatus.setComplete(); // limit까지 입력받으면 세션 데이터를 비운다.
        attributes.addAttribute("name", event.getName());
        attributes.addAttribute("limit", event.getLimit());
        return "redirect:/events/list"; // name과 limit가 primitive type이기 때문에 "redirect:/events/list?name="event.getName()"&limit="event.getLimit()"와 동일하게 전달된다.
    }

    @GetMapping("/events/list")
    public String getEvents(@ModelAttribute Event newEvent, Model model){
        Event event = new Event();
        event.setName("spring");
        event.setLimit(10);
        List<Event> eventList = new ArrayList<>();
        eventList.add(event);
        eventList.add(newEvent);
        model.addAttribute(eventList);
        return "/events/list";
    }
}

```


이 케이스의 경우, `@SessionAttributes`에서 지정한 변수 명과 동일하게 셋팅하지는 않았는지 다시 한 번 확인해야 한다. 만일 동일할 경우, 세션에 저장되어 있는 값을 가져온다. 하지만 가져오기 전 이미 세션 데이터를 비운 상태라면 오류가 생길 수 있기 때문에 명칭은 다르게 셋팅해주자. 즉, 세션이 아니라 쿼리파라미터를 통해 가져올 수 있도록 코드를 작성하자.

#### `@RequestParam` vs `@PathVariable`
* `@RequestParam`은 폼 데이터나 쿼리 파라미터를 통해서 가져온다.
* `@PathVariable`은 URI 경로에서 가져온다.


***
참고자료

1. [스프링 웹 MVC](https://inf.run/dJFi)
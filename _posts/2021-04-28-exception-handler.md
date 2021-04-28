---
layout: post
title: 예외 처리 핸들러 : @ExceptionHandler
categories: [Spring]
tags: [Spring Framework, Spring MVC]
description: 특정 예외가 발생한 요청을 처리하는 핸들러 정의
fullview: false
comments: true
---

MVC에서 요청을 처리하다가 예외를 발생시키거나, 자바에서 기본적으로 지원하는 예외가 발생했을 때 우리가 정의한 핸들러로 예외를 처리하는 방법에 대해 알아보자.

### `@ExceptionHandler`
* 특정 예외가 발생한 요청을 처리하는 핸들러를 정의한다
* 지원하는 메소드 아규먼트(해당 예외 객체, 핸들러 객체..)
* 지원하는 리턴 값


컨트롤러 예시 파일 : 

```java
package me.cjk.demo;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.validation.BindingResult;
import org.springframework.validation.Validator;
import org.springframework.validation.annotation.Validated;
import org.springframework.web.bind.WebDataBinder;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.bind.support.SessionStatus;
import org.springframework.web.servlet.mvc.support.RedirectAttributes;

import java.util.ArrayList;
import java.util.List;

@Controller
@SessionAttributes("event")
public class EventController {

    @ExceptionHandler
    public String eventErrorHandler(EventException exception,Model model){
        model.addAttribute("message", "event error");
        return "/events/error";
    }

    @InitBinder
    public void initEventBinder(WebDataBinder webDataBinder){
        webDataBinder.setDisallowedFields("id");
    }

    @GetMapping("/events/form/name")
    public String eventsFormName(Model model){
        throw new EventException(); //예외 발생시 eventErrorHandler 메소드가 실행된다.
        /*model.addAttribute("event", new Event()); //모델에서 설정한 attribute 중에 SessionAttributes에서 설정한 이름과 동일한 것이 있다면 세션 속성으로 넣는다.
        return "events/form-name";*/
    }

    @PostMapping(value = "/events/form/name")
    public String eventsFormNameSubmit(@Validated @ModelAttribute Event event,
                                 BindingResult bindingResult){

        if(bindingResult.hasErrors()){
            return "/events/form-name"; //이상있을 경우 이름을 입력하는 페이지로 다시 이동
        }

        return "redirect:/events/form/limit"; // 제한인원 입력하는 페이지로 이동
    }

}

```

<br/>

* 동일한 에러처리 메소드가 존재할 경우, Exception이 가장 구체적인 메소드를 처리한다.
* 여러 에러를 동시에 처리하고 싶을 경우 아래와 같이 정의해주면 된다.
	* `@ExceptionHandler({EventException.class,RuntimeException.class})`
	* 핸들러 메소드 아규먼트는 위 클래스를 모두 받을 수 있도록 타입을 지정해주어야 한다.
* REST API의 경우 응답 본문에 에러에 대한 정보를 담아주고, 상태 코드를 설정하려면 주로 `ResponseEntity`를 사용한다.

예시 파일 : 

```java
@ExceptionHandler
    public ResponseEntity errorHandler(){
        return ResponseEntity.badRequest().body("can't create event as ...");
    }
```


***
참고자료

1. [스프링 웹 MVC](https://inf.run/dJFi)
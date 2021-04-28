---
layout: post
title: DataBinder :  @InitBinder
categories: [Spring]
tags: [Spring Framework, Spring MVC]
description: 특정 컨트롤러에서 바인딩 또는 검증 설정을 변경하고 싶을 때 사용하는 @InitBinder
fullview: false
comments: true
---

요청 URI path에 있거나 요청 매개변수(쿼리 파라미터, 폼데이터)에서 받아오는 데이터를 바인딩 할 때 우리가 커스텀한 validator, formatter등으로 진행하는 방법에 대해 알아보자.

### `@InitBinder`

`@InitBinder`를 붙인 메소드를 정의하여 설정을 변경할 수 있다. 리턴타입은 반드시 `void`이어야 한다.

모든 요청 전에 `initEventBinder`메소드를 호출하여 커스터마이징 할 수 있다.

* `webDataBinder.setDisallowedFields("id");` : 데이터 바인딩 설정
	* 받고싶지 않은 필드 값을 걸러낼 수 있다. id는 이벤트를 저장할 때 생성하고 싶기 때문에, 요청에서 받아오고 싶지 않을때 이와 같이 설정해준다. 즉 바인딩을 받지 않는다.
* `webDataBinder.setCustomFormatter()` : 포매터 설정
* `webDataBinder.addValidators(new EventValidator());` : Validator 설정
	* 커스텀한 `Validator`를 설정할 수 있다.

`EventValidator` : 

```java
package me.cjk.demo;

import org.springframework.validation.Errors;
import org.springframework.validation.Validator;

public class EventValidator implements Validator {
    @Override
    public boolean supports(Class<?> aClass) {
        return Event.class.isAssignableFrom(aClass);
    }

    @Override
    public void validate(Object o, Errors errors) {
        Event event = (Event) o;
        if(event.getName().equalsIgnoreCase("aaa")){
            errors.rejectValue("name","wrongValue","the value is not allowed");
        }
    }
}
```
`Validator`인터페이스를 상속하여 `Event`객체의 특정 필드 값을 검증할 수 있다.


#### `webDataBinder`를 사용하지 않고 데이터 검증하는 방법
컨트롤러 단에서 `Validator`를 주입받아서, 사용하고자 하는 시점에 `validate()`메소드를 실행한다. 이렇게 하면 명시적으로 원하는 시점에 검증 할 수 있다.

`EventController`

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

    @Autowired
    EventValidator eventValidator; //주입

    @InitBinder
    public void initEventBinder(WebDataBinder webDataBinder){
        webDataBinder.setDisallowedFields("id");
    }

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
        eventValidator.validate(event,bindingResult); //이 시점에 데이터 검증 진행
        return "redirect:/events/form/limit"; // 제한인원 입력하는 페이지로 이동
    }

}
```
이렇게 작성하면 `EventValidator`는 굳이 `Validator`를 구현하지 않아도 된다. 아래와 같이 말이다. 

```java
package me.cjk.demo;

import org.springframework.validation.Errors;

public class EventValidator {

    public void validate(Event event, Errors errors) {
        if(event.getName().equalsIgnoreCase("aaa")){
            errors.rejectValue("name","wrongValue","the value is not allowed");
        }
    }
}
```
#### 특정 모델 객체에만 바인딩 혹은 Validator 설정을 적용하고 싶은 경우
* `@InitBinder("event")` : 애노테이션 뒤에  문자열을 주면, 이 이름의 ModelAttribute를 바인딩 받을 때에만 적용된다.

***
참고자료

1. [스프링 웹 MVC](https://inf.run/dJFi)
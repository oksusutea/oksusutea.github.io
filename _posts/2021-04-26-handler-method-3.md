---
layout: post
title: 핸들러 메소드 (3) - 요청 매개변수(단순 타입)
categories: [Spring]
tags: [Spring Framework, Spring MVC]
description: 핸들러 메소드를 처리하는 방법 - @RequestParam
fullview: false
comments: true
---

### `@RequestParam`
* 요청 매개변수에 들어있는 단순 타입(=다른 타입들을 감싸고 있는 클래스 말고) 데이터를 메소드 아규먼트로 받아올 수 있다.
* 값이 반드시 있어야 한다.
* String이 아닌 값은 타입 컨버전을 지원한다.
* Map<String, String> 또는 MultiValueMap<String, String>에 사용해서 모든 요청 매개변수를 받아 올 수도 있다.
* 이 애노테이션은 생략 가능하다.

```java
package me.cjk.demo;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.*;

@Controller
public class SampleController {

    @GetMapping(value = "/events/{id}")
    @ResponseBody
    public Event getEvent(@RequestParam(required = false, defaultValue = "cjk") String name){
            Event event = new Event();
            event.setName(name);
            return event;
    }
}
```
위 코드에서 `@RequestParam`은 생략 가능하다. 하지만 보다 명시적으로 표현하기 위해 애노테이션을 붙이는 것을 권장한다.(참고로 아규먼트에서 생략 가능한 애노테이션은 `@RequestParam`과 `@ModelAttribute` 두개 뿐이다)

#### URI 패턴 변수와 메소드 아규먼트 매개변수가 다를 때

```java
package me.cjk.demo;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.*;

@Controller
public class SampleController {

    @GetMapping(value = "/events/{id}?name")
    @ResponseBody
    public Event getEvent(@RequestParam(value = "name", required = false, defaultValue = "cjk") String nameValue){
            Event event = new Event();
            event.setName(name);
            return event;
    }
}
```
위와 같이 설정 가능하지만, 보다 간결하고 명확하게 작성하기 위해서 보통은 변수명을 동일하게 설정한다.

#### 요청 매개변수란?
* 쿼리 매개변수

컨트롤러 파일 : 

```java
package me.cjk.demo;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.*;

@Controller
public class SampleController {

    @PostMapping(value = "/events")
    @ResponseBody
    public Event getEvent(@RequestParam String name){
            Event event = new Event();
            event.setName(name);
            return event;
    }
}

```

테스트 코드 : 

```java
package me.cjk.demo;

import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.http.MediaType;
import org.springframework.test.web.servlet.MockMvc;

import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.*;
import static org.springframework.test.web.servlet.result.MockMvcResultHandlers.print;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.jsonPath;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

@WebMvcTest
class SampleControllerTest {

    @Autowired
    MockMvc mockMvc;

    @Test
    public void eventsTest() throws Exception {
        mockMvc.perform(post("/events?name=cjkim"))
                .andDo(print())
                .andExpect(status().isOk())
                .andExpect(jsonPath("name").value("cjkim"));
    }

}
```

위와 같이 쿼리파라미터로 name="cjkim"을 전송하면, 요청한 값이 정상적으로 표기되는 것(테스트 성공!)을 볼 수 있다.

* 폼 데이터

```java
package me.cjk.demo;

import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.http.MediaType;
import org.springframework.test.web.servlet.MockMvc;

import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.*;
import static org.springframework.test.web.servlet.result.MockMvcResultHandlers.print;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.jsonPath;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

@WebMvcTest
class SampleControllerTest {

    @Autowired
    MockMvc mockMvc;

    @Test
    public void eventsTest() throws Exception {
        mockMvc.perform(post("/events?")
                    .param("name","cjkim"))
                .andDo(print())
                .andExpect(status().isOk())
                .andExpect(jsonPath("name").value("cjkim"));
    }

}
```
mockMvc는 모두 requestParam이기 때문에 ,`param`으로 설정하여 보내어도 똑같은 결과를 볼 수 있다.

#### 모든 파라미터를 받아오는 방법

```java
package me.cjk.demo;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.*;

import java.util.Map;

@Controller
public class SampleController {

    @PostMapping(value = "/events")
    @ResponseBody
    public Event getEvent(@RequestParam Map<String,String> params){
            Event event = new Event();
            event.setName(params.get("name"));
            return event;
    }
}
```

`params`안에 모든 요청 매개변수가 들어있어 이렇게 값을 직접적으로 꺼내올 수 있다.

***
참고자료

1. [스프링 웹 MVC](https://inf.run/dJFi)
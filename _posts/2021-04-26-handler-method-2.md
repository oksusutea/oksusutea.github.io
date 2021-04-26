---
layout: post
title: 핸들러 메소드 (2) - URI 패턴
categories: [Spring]
tags: [Spring Framework, Spring MVC]
description: 핸들러 메소드를 처리하는 방법 - @PathVariable, @MatrixVariable
fullview: false
comments: true
---

### `@PathVariable`
* 요청 URI 패턴의 일부를 핸들러 메소드 아규먼트로 받는 방법
* 타입 변환을 지원한다
* (기본) 값이 반드시 있어야 한다.
* Optional을 지원한다.

SampleController : 
```java
package me.cjk.demo;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.*;

@Controller
public class SampleController {

    @GetMapping(value = "/events/{id}")
    @ResponseBody
    public Event getEvent(@PathVariable Integer id){
        Event event = new Event();
        event.setId(id);
        return  event;

    }
}

```

Event 클래스 파일 : 

```java
package me.cjk.demo;

public class Event {
    private Integer id;
    private  String name;

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
        mockMvc.perform(get("/events/1"))
                .andDo(print())
                .andExpect(status().isOk())
                .andExpect(jsonPath("id").value(1));
    }

}
```

테스트는 이상없이 동작하고,  요청으로 들어온 값을 `Event`로 생성하여 응답할 수 있도록 한다. 이 때 HttpMessageConverter를 사용한다.  테스트코드에서 url요청을 할 때 `/events/1`과 같이 **1**을 문자열로 요청하여도 id를 자동으로 Integer로 타입 변환해준다.

#### Optional로 지정하는 방법

```java
package me.cjk.demo;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.*;

import java.util.Optional;

@Controller
public class SampleController {

    @GetMapping(value = "/events/{id}")
    @ResponseBody
    public Event getEvent(@PathVariable Optional<Integer> id){
        if(id.isPresent()) {
            Event event = new Event();
            event.setId(id.get());
            return event;
        }
        return null;
    }
}

```
위와 같이 자바 8부터 `@PathVariable`을 Optional로도 지정할 수 있다.

#### url부분의 변수명과 `@PathVariable` 변수명을 다르게 설정하는 방법

```java
package me.cjk.demo;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.*;

@Controller
public class SampleController {

    @GetMapping(value = "/events/{id}")
    @ResponseBody
    public Event getEvent(@PathVariable("id") Integer idValue, @MatrixVariable String name){
            Event event = new Event();
            event.setId(idValue);
            event.setName(name);
            return event;
    }
}

```
변수명을 같게 할 경우 `@PathVariable`에서 추가로 설정할 필요가 없고, 다를 경우에만 위와 같이 설정해주면 된다.

### `@MatrixVariable`
* 요청 URI 패턴에서 키/값 쌍의 데이터를 아규먼트로 받는 방법
* 타입 변환 지원
* (기본)값이 반드시 있어야 한다.
* Optional 지원
* 이 기능은 기본적으로 비활성화 되어 있어 환경설정을 추가로 진행해주어야 한다.

컨트롤러 파일 : 

```java
package me.cjk.demo;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.*;

@Controller
public class SampleController {

    @GetMapping(value = "/events/{id}")
    @ResponseBody
    public Event getEvent(@PathVariable Integer id, @MatrixVariable String name){
            Event event = new Event();
            event.setId(id);
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
        mockMvc.perform(get("/events/1;name=cjkim"))
                .andDo(print())
                .andExpect(status().isOk())
                .andExpect(jsonPath("id").value(1));
    }

}
```

테스트 결과 : 

```
MockHttpServletResponse:
           Status = 400
    Error message = Required matrix variable 'name' for method parameter type String is not present
          Headers = []
     Content type = null
             Body = 
    Forwarded URL = null
   Redirected URL = null
          Cookies = []

```
위와 같이 Error Message에서 메소드 파라미터로 **name**을 찾을 수 없다고 나온다. `@MatrixVariable`을 사용하기 위해서 아래와 같이 추가 설정이 필요하기 때문이다.

```java
package me.cjk.demo;

import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.PathMatchConfigurer;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;
import org.springframework.web.util.UrlPathHelper;

@Configuration
public class WebConfig implements WebMvcConfigurer {
    @Override
    public void configurePathMatch(PathMatchConfigurer configurer) {
        UrlPathHelper urlPathHelper = new UrlPathHelper();
        urlPathHelper.setRemoveSemicolonContent(false); //세미콜론을 제거하지 않도록 설정, 세미콜론을 없애면 데이터 바인딩이 안된다.
        configurer.setUrlPathHelper(urlPathHelper);     //설정파일에 urlPathHelper 설정
    }
}
```

출력결과 : 

```
MockHttpServletResponse:
           Status = 200
    Error message = null
          Headers = [Content-Type:"application/json"]
     Content type = application/json
             Body = {"id":1,"name":"cjkim"}
    Forwarded URL = null
   Redirected URL = null
          Cookies = []
```
위와 같이 테스트가 성공하는 것을 알 수 있다.

***
참고자료

1. [스프링 웹 MVC](https://inf.run/dJFi)
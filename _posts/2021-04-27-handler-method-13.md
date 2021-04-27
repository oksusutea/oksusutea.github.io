---
layout: post
title: 핸들러 메소드 (13) - @RequestBody와 HttpEntity
categories: [Spring]
tags: [Spring Framework, Spring MVC]
description: 스프링에서 요청에 들어있는 데이터를 받아오는 법
fullview: false
comments: true
---

### `@RequestBody`
* 요청 본문(body)에 들어있는 데이터를 HttpMessageConverter를 통해 변환한 객체로 받아올 수 있다.
* `@Valid` 혹은 `@Validated`를 사용해서 값을 검증할 수 있다.
* `BindingResult` 아규먼트를 사용해 코드로 바인딩 혹은 검증 에러를 확인할 수 있다.

#### 예시 코드 

```java
package me.cjk.demo;

import org.springframework.validation.BindingResult;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import javax.validation.Valid;

@RestController
@RequestMapping("/api/events")
public class EventApi {

    @PostMapping()
    public Event createEvent(@RequestBody @Valid Event event,
                             BindingResult bindingResult){ //요청의 본문에 있는 데이터를 Event로 컨버젼하는데 이 때, HttpMessageConverter로 컨버젼을 진행한다.

        if(bindingResult.hasErrors()){
            bindingResult.getAllErrors().forEach(error->{
                System.out.println(error);
            });
        }
        return event;
    }
}
```
위와 같이 작성해주면, 바인딩에러가 발생하여도 에러(400 Bad request)를 발생하지 않는다. 개발자가 에러에 대해 직접적으로 처리를 해주어야 한다.

테스트 코드 : 

```java
package me.cjk.demo;

import com.fasterxml.jackson.databind.ObjectMapper;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.AutoConfigureMockMvc;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.http.MediaType;
import org.springframework.test.web.servlet.MockMvc;

import static org.junit.jupiter.api.Assertions.*;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.post;
import static org.springframework.test.web.servlet.result.MockMvcResultHandlers.print;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.jsonPath;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

@SpringBootTest
@AutoConfigureMockMvc
class EventApiTest {

    @Autowired
    ObjectMapper objectMapper; //객체를 Json, json을 객체로 변환 가능

    @Autowired
    MockMvc mockMvc;

    @Test
    public void createEvent() throws Exception {
        Event event = new Event();
        event.setName("cjkim");
        event.setId(10);
        event.setLimit(-10);

        String json = objectMapper.writeValueAsString(event); //오브젝트 객체를 json 문자열로 변환해준다.
        mockMvc.perform(post("/api/events")
                .contentType(MediaType.APPLICATION_JSON)
                .content(json))
                .andDo(print())
                .andExpect(status().isOk())
                .andExpect(jsonPath("name").value("cjkim"));
    }


}
```

```
Field error in object 'event' on field 'limit': rejected value [-10]; codes [Min.event.limit,Min.limit,Min.java.lang.Integer,Min]; arguments [org.springframework.context.support.DefaultMessageSourceResolvable: codes [event.limit,limit]; arguments []; default message [limit],0]; default message [must be greater than or equal to 0]

MockHttpServletRequest:
      HTTP Method = POST
      Request URI = /api/events
       Parameters = {}
          Headers = [Content-Type:"application/json;charset=UTF-8", Content-Length:"36"]
             Body = {"id":10,"name":"cjkim","limit":-10}
    Session Attrs = {}

Handler:
             Type = me.cjk.demo.EventApi
           Method = me.cjk.demo.EventApi#createEvent(Event, BindingResult)

Async:
    Async started = false
     Async result = null

Resolved Exception:
             Type = null

ModelAndView:
        View name = null
             View = null
            Model = null

FlashMap:
       Attributes = null

MockHttpServletResponse:
           Status = 200
    Error message = null
          Headers = [Content-Type:"application/json"]
     Content type = application/json
             Body = {"id":10,"name":"cjkim","limit":-10}
    Forwarded URL = null
   Redirected URL = null
          Cookies = []

```

위 테스트의 결과로 에러는 출력되지만, `BindingResult`를 핸들러 메소드의 아규먼트로 받았기 때문에  200번대로 응답코드를 내보낸 것을 볼 수 있다.

### HttpMessageConverter
* 스프링 MVC 설정(WebMvcConfigurer)에서 설정할 수 있다.
* configureMessageConveters: 기본 메시지 컨버터 대체
* extendMessageConverters: 메시지 컨버터에 추가
* 기본 컨버터
	* WebMvcConfigurationSupport.addDefaultHttpMessageConverters
	* 이러한 메시지 컨버터는 **핸들러 어댑터**가 사용해서 아규먼트를 resolving할 때 사용한다.

#### 예시 코드

api 컨트롤러 : 

```java
package me.cjk.demo;

import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/api/events")
public class EventApi {

    @PostMapping()
    public Event createEvent(@RequestBody  Event event){ //요청의 본문에 있는 데이터를 Event로 컨버젼하는데 이 때, HttpMessageConverter로 컨버젼을 진행한다.
        return event;
    }
}
```

테스트 코드 : 

```java
package me.cjk.demo;

import com.fasterxml.jackson.databind.ObjectMapper;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.AutoConfigureMockMvc;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.http.MediaType;
import org.springframework.test.web.servlet.MockMvc;

import static org.junit.jupiter.api.Assertions.*;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.post;
import static org.springframework.test.web.servlet.result.MockMvcResultHandlers.print;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.jsonPath;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

@SpringBootTest
@AutoConfigureMockMvc
class EventApiTest {

    @Autowired
    ObjectMapper objectMapper; //객체를 Json, json을 객체로 변환 가능

    @Autowired
    MockMvc mockMvc;

    @Test
    public void createEvent() throws Exception {
        Event event = new Event();
        event.setName("cjkim");
        event.setId(10);
        event.setLimit(20);

        String json = objectMapper.writeValueAsString(event); //오브젝트 객체를 json 문자열로 변환해준다.
        mockMvc.perform(post("/api/events")
                .contentType(MediaType.APPLICATION_JSON)
                .content(json))
                .andDo(print())
                .andExpect(status().isOk())
                .andExpect(jsonPath("name").value("cjkim"));
    }


}
```



### HttpEntity
* `@RequestBody`와 비슷하지만 추가적으로 요청 헤더 정보를 사용할 수 있다.(`@RequestBody`는 요청 본문에 대한 데이터만 받아올 수 있다)
* 애노테이션을 추가로 지정해주지 않아도 되며, 바디 타입을 `HttpEntity`의 제너릭 타입으로 지정해주어야 한다.

컨트롤러 
```java
package me.cjk.demo;

import org.springframework.http.HttpEntity;
import org.springframework.http.MediaType;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/api/events")
public class EventApi {

    @PostMapping()
    public Event createEvent(HttpEntity<Event> request){ //요청의 본문에 있는 데이터를 Event로 컨버젼하는데 이 때, HttpMessageConverter로 컨버젼을 진행한다.
        MediaType contentType = request.getHeaders().getContentType();
        System.out.println(contentType);
        return request.getBody();
    }
}
```




***
참고자료

1. [스프링 웹 MVC](https://inf.run/dJFi)
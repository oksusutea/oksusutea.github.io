---
layout: post
title: 핸들러 메소드 (13) - @ResponseBody 와 ResponseEntity
categories: [Spring]
tags: [Spring Framework, Spring MVC]
description: 스프링에서 응답 데이터를 처리하는 방법
fullview: false
comments: true
---

### `@ResponseBody`
* 데이터를 HttpMessageConverter를 사용해 본문 메시지로 보낼 수 있다.
* `@RestController` 사용시 자동으로 모든 핸들러 메소드에 적용된다.
	* 불필요하게 메소드마다 `@ResponseBody`를 작성해주지 않아도 된다.

#### 예시 코드 

```java
package me.cjk.demo;

import org.springframework.validation.BindingResult;
import org.springframework.web.bind.annotation.*;

import javax.validation.Valid;

@RequestMapping("/api/events")
public class EventApi {

    @PostMapping()
    @ResponseBody // 이 메소드에서 리턴하는 값을 HTTP 메시지 응답 본문에 담아준다. 이 때 HttpmMessageConverter를 사용한다.
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

`@ResponseBody`는 `@RequestBody`와 동일하게 `HttpMessageConverter`를 사용한다. 그리고 적절한 타입을 선택해서 쓰게 되는데, 여기서 많이 참고하는 값은 요청이 보내오는 `accept-header`이다.

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
                .content(json)
                .accept(MediaType.APPLICATION_JSON))
                .andDo(print())
                .andExpect(status().isOk())
                .andExpect(jsonPath("name").value("cjkim"));
    }


}
```

### ResponseEntity
* 응답 헤더 상태, 코드, 본문등을 직접 다루고 싶은 경우 사용한다.

#### 예시 코드

```java
package me.cjk.demo;

import org.springframework.http.ResponseEntity;
import org.springframework.validation.BindingResult;
import org.springframework.web.bind.annotation.*;

import javax.validation.Valid;

@RequestMapping("/api/events")
public class EventApi {

    @PostMapping
    public ResponseEntity<Event> createEvent(@RequestBody @Valid Event event,
                                      BindingResult bindingResult){ //요청의 본문에 있는 데이터를 Event로 컨버젼하는데 이 때, HttpMessageConverter로 컨버젼을 진행한다.

        if(bindingResult.hasErrors()){
            return ResponseEntity.badRequest().build();
        }

        //ResponseEntity responseEntity = new ResponseEntity(); // 헤더, 상태정보, 바디 등정보를 입력할 수 있다.
        return ResponseEntity.ok(event);
    }
}

```

***
참고자료

1. [스프링 웹 MVC](https://inf.run/dJFi)
---
layout: post
title: 요청 맵핑하기 (5) - HEAD와 OPTIONS
categories: [Spring]
tags: [Spring Framework, Spring MVC]
description: Spring MVC에서 HEAD와 OPTIONS으로 요청을 처리하는 방법
fullview: false
comments: true
---

우리가 직접 만들지 않아도, 스프링 MVC가 제공하는 기능인 HTTP Method 옵션 처리를 해주는 부분에 대해 자세히 알아보자.

### Head
* GET 요청과 동일하지만 **응답 본문을 받아오지 않고 응답 헤더만** 받아온다.

테스트 코드 : 

```java
package me.cjk.demo;

import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.AutoConfigureMockMvc;
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.http.HttpHeaders;
import org.springframework.http.MediaType;
import org.springframework.test.web.servlet.MockMvc;

import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.head;
import static org.springframework.test.web.servlet.result.MockMvcResultHandlers.print;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

@WebMvcTest
class SampleControllerTest {

    @Autowired
    MockMvc mockMvc;

    @Test
    public void helloTest() throws Exception {
        mockMvc.perform(head("/hello")
                    .header(HttpHeaders.FROM,"222"))
                .andDo(print())
                .andExpect(status().isOk());
    }
}
```
출력 결과 : 

```
MockHttpServletResponse:
           Status = 200
    Error message = null
          Headers = [Content-Type:"text/plain;charset=UTF-8", Content-Length:"5"]
     Content type = text/plain;charset=UTF-8
             Body = 
    Forwarded URL = null
   Redirected URL = null
          Cookies = []

```


### OPTIONS
* 사용할 수 있는 HTTP Method를 제공한다.
* 서버 또는 특정 리소스가 제공하는 기능을 확인할 수 있다.
* 서버는 Allow 응답 헤더에 사용할 수 있는 HTTP Method 목록을 제공해야 한다.

예시 코드 : 

```java
package me.cjk.demo;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.ResponseBody;

@Controller
public class SampleController {
    @GetMapping("/hello")
    @ResponseBody
    public String hello(){
        return "hello";
    }

    @PostMapping("/hello")
    @ResponseBody
    public String helloPost(){
        return "hello";
    }
}
```
위 코드에서 `/hello`라는 요청은 GET,POST요청을 받는다. 거기에 스프링 MVC에서 기본적으로 제공하는 HEAD, OPTIONS를 추가로 제공한다.

테스트 코드 : 

```java
package me.cjk.demo;

import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.AutoConfigureMockMvc;
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.http.HttpHeaders;
import org.springframework.http.MediaType;
import org.springframework.test.web.servlet.MockMvc;

import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.options;
import static org.springframework.test.web.servlet.result.MockMvcResultHandlers.print;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

@WebMvcTest
class SampleControllerTest {

    @Autowired
    MockMvc mockMvc;

    @Test
    public void helloTest() throws Exception {
        mockMvc.perform(options("/hello"))
                .andDo(print())
                .andExpect(status().isOk());
    }
}
```

출력 결과 : 

```
MockHttpServletResponse:
           Status = 200
    Error message = null
          Headers = [Allow:"POST,GET,HEAD,OPTIONS", Accept-Patch:""]
     Content type = null
             Body = 
    Forwarded URL = null
   Redirected URL = null
          Cookies = []
```

위와 같이 출력 결과에서 `"POST,GET,HEAD,OPTIONS"`가 나온 것을 볼 수 있다.

***
참고자료

1. [스프링 웹 MVC](https://inf.run/dJFi)
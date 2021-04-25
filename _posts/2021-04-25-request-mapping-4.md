---
layout: post
title: 요청 맵핑하기 (4) - 헤더와 매개변수
categories: [Spring]
tags: [Spring Framework, Spring MVC]
description: Spring MVC에서 헤더와 매개변수의 값을 토대로 요청을 받는 방법
fullview: false
comments: true
---

이전 시간에는 요청시 contents-type을 지정하고, 응답시 accpet-header를 이용하여 특정 요청/응답을 반환할 수 있는 방법에 대해 배웠다. 이번 챕터는 조금 더 일반적인 방법으로, 임의의 http 헤더 값을 토대로 요청을 처리하는 방법을 배운다.


* 특정한 헤더가 있는 요청을 처리하고 싶은 경우
	* `@RequestMapping(headers = "key")`
* 특정한 헤더가 없는 요청을 처리하고 싶은 경우
	* `@RequestMapping(headers = "!key")`
* 특정한 헤더 키/값이 있는 요청을 처리하고 싶은 경우
	* `@RequestMapping(headers = "key=value")`
* 특정한 요청 매개변수 키를 가지고 있는 요청을 처리하고 싶은 경우
	* `@RequestMapping(params = "a")`
* 특정한 요청 매개변수가 없는 요청을 처리하고 싶은 경우
	* `@RequestMapping(params = "!a")`
* 특정한 요청 매개변수 키/값을 가지고 있는 요청을 처리하고 싶은 경우
	* `@RequestMapping(params= "key=value")`

### 예시 및 테스트 코드 : 

컨트롤러 코드 : 

```java
package me.cjk.demo;

import org.springframework.http.HttpHeaders;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.ResponseBody;

@Controller
public class SampleController {
    @GetMapping(value="/hello", headers = HttpHeaders.FROM)
    @ResponseBody
    public String hello(){
        return "hello";
    }
}

```
headers는 문자열로도 표현 가능하지만, 위와 같이 상수로 표현하는 것이 type-safe하다.


```java
@WebMvcTest
class SampleControllerTest {

    @Autowired
    MockMvc mockMvc;

    @Test
    public void helloTest() throws Exception {
        mockMvc.perform(get("/hello")
                    .header(HttpHeaders.FROM,"localhost"))
                .andDo(print())
                .andExpect(status().isOk());
    }
}
```

***
참고자료

1. [스프링 웹 MVC](https://inf.run/dJFi)
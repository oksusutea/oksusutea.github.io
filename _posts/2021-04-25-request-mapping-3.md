---
layout: post
title: 요청 맵핑하기 (3) - 미디어 타입
categories: [Spring]
tags: [Spring Framework, Spring MVC]
description: Spring MVC에서 요청과 응답의 타입을 지정하는 방법
fullview: false
comments: true
---
요청을 처리할 때 컨텐츠 타입 그리고 accpet header를 사용하는 방법에 대해 알아보자.

### Media type이란?
Media type은 인터넷에 전달되는 파일 포맷과 포맷 컨텐츠를 위한 식별자이고, HTTP와 HTML 문서 파일 포맷에 사용된다. 웹의 동작은 request, response로 이루어져 있다. 이 때 request와 response에 Media Type를 지정해서 요청받을 때 사용하는 Media Type와 응답할 때 보내주는 Media Type을 지정, 사전에 필요한 타입만 거를 수 있다.

### 특정한 타입의 데이터를 담고 있는 요청만 처리하는 핸들러
* `@RequestMapping(consumes="application/json")`
* Content-type 헤더로 필터링한다.
* 매치되지 않는 경우 415 Not Supported MediaType 응답

```java
package me.cjk.demo;

import org.springframework.http.MediaType;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;

@Controller
public class SampleController {
    //@RequestMapping(value = "/hello", consumes = "application/json") 이렇게 작성하면 type-safe하지 않기 때문에 아래와 같은 방식으로 작성하는 것을 권장한다.
    @RequestMapping(value="/hello", consumes = MediaType.APPLICATION_JSON_VALUE)
    public String hello(){
        return "hello";
    }
}
```

`MediaType.APPLICATION_JSON_VALUE = "application/json"` 으로 할당되었기 때문에, Content-type의 헤더가 `"application/json"`일 경우에만 `hello()`핸들러로 처리하게 된다.

#### 테스트 코드

```java

@WebMvcTest
class SampleControllerTest {

    @Autowired
    MockMvc mockMvc;

    @Test
    public void helloTest() throws Exception {
        mockMvc.perform(get("/hello"))
                .andDo(print())
                .andExpect(status().isOk());
    }
}
```
위 코드로 테스트 하게 되면, header에는 아무 값도 들어있지 않기 때문에 json요청이 아니라고 판단하고, `hello()`핸들러로 처리하지 않아 테스트는 실패하게 된다.  아래와 같이 `contentType`를 JSON으로 설정하면 테스트가 성공되는 것을 확인할 수 있다.

```java
@Test
    public void helloTest() throws Exception {
        mockMvc.perform(get("/hello")
                    .contentType(MediaType.APPLICATION_JSON))
                .andDo(print())
                .andExpect(status().isOk());
    }
```

### 특정한 타입의 응답을 만드는 핸들러
* `@RequestMapping(produces="application/json")`
* Accept 헤더로 필터링
* 매치되지 않을 경우 406 Not Supported 응답
*  

```java
package me.cjk.demo;

import org.springframework.http.MediaType;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;

@Controller
public class SampleController {
    //@RequestMapping(value = "/hello", produces = "application/json") 이렇게 작성하면 type-safe하지 않기 때문에 아래와 같은 방식으로 작성하는 것을 권장한다.
    @RequestMapping(value="/hello", produces = MediaType.APPLICATION_JSON_VALUE)
    @ResponseBody
    public String hello(){
        return "hello";
    }
}
```

테스트 코드 :

```java
package me.cjk.demo;

import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.AutoConfigureMockMvc;
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.http.MediaType;
import org.springframework.test.web.servlet.MockMvc;

import static org.junit.jupiter.api.Assertions.*;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
import static org.springframework.test.web.servlet.result.MockMvcResultHandlers.print;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

@WebMvcTest
class SampleControllerTest {

    @Autowired
    MockMvc mockMvc;

    @Test
    public void helloTest() throws Exception {
        mockMvc.perform(get("/hello")
                    .contentType(MediaType.APPLICATION_JSON)
                    .accept(MediaType.APPLICATION_JSON))
                .andDo(print())
                .andExpect(status().isOk());
    }
}
```

물론 `produces`를 지정하지 않으면, 보통 accept를 요청한 타입으로 변환을 해주지만, handler에서 `produces`를 지정하였을 경우 해당 타입만 리턴하게 된다.



### 정리
* 요청은 **consums**, 응답은 **produces**로 이용하여 mediatype을 지정할 수 있다.  
* 문자열을 입력하는 대신 MediaType을 사용하면 상수를 자동완성하여 사용할 수 있다.(type-safe하다).
* 클래스에 선언한 `@RequestMapping`에 사용한 것과 조합이 되지 않고 메소드에 사용한 `@RequestMapping`의 설정으로 덮어쓴다(오버라이드). 즉, 메소드 레벨의 미디어 타입이 우선순위를 가진다. `@RequestMapping`과 다르다.


***
참고자료

1. [스프링 웹 MVC](https://inf.run/dJFi)
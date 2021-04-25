---
layout: post
title: 요청 맵핑하기 (6) - Custom Annotation
categories: [Spring]
tags: [Spring Framework, Spring MVC]
description: Spring MVC에서 애노테이션 만들어서 요청을 처리하는 방법
fullview: false
comments: true
---

`@RequestMapping`애노테이션 메타 애노테이션으로 사용하여 `@GetMapping`같이 커스텀한 애노테이션을 만들 수 있다.

#### 메타(Meta) 애노테이션
* 애노테이션에 사용할 수 있는 애노테이션
* 스프링이 제공하는 대부분의 에노테이션은 메타 에노테이션으로 사용할 수 있다.
* 예 : `@Target`, `@Documented`, `@Retention`, `@RequestMapping`

#### 조합(Composed) 애노테이션
* 한 개, 혹은 여러 메타 애노테이션을 조합하여 만든 애노테이션
* 코드를 간결하게 줄일 수 있다.
* 보다 구체적인 의미를 부여 할 수 있다.

### 활용 예시 및 테스트코드

커스텀 애노테이션 파일 : 

```java
package me.cjk.demo;

import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;

@RequestMapping(method = RequestMethod.GET, value = "/hello")
public @interface GetHelloMapping {
}
```

컨트롤러 : 

```java
package me.cjk.demo;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.ResponseBody;

@Controller
public class SampleController {
    @GetHelloMapping
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
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
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
           Status = 404
    Error message = null
          Headers = [Vary:"Origin", "Access-Control-Request-Method", "Access-Control-Request-Headers", Allow:"GET, HEAD, POST, PUT, DELETE, TRACE, OPTIONS, PATCH"]
     Content type = null
             Body = 
    Forwarded URL = null
   Redirected URL = null
          Cookies = []
```

응답에 404에러가 뜬다. 이 이유는 애노테이션의 유지기간을 지정해주지 않았기 때문이다. `@Retention`을 지정해주지 않을 경우, 컴파일까지만 유지되어 런타임시에는 해당 애노테이션 정보는 사라지게 된다.


#### `@Retention`
* 해당 애노테이션을 언제까지 유지할 것인지에 대해 정의
* `Source`: 소스 코드까지만 유지. 즉, 컴파일 하면 해당 애노테이션 정보는 사라진다. (주석처럼 쓰고 싶을 때 이 타입으로 지정한다)
* `Class` : 컴파일한 .class 파일에도 유지. 즉 런타임 시, 클래스를 메모리로 읽어오면 해당 정보는 사라진다.
* `Runtime` : 클래스를 메모리에 읽어왔을 때까지 유지한다. 코드에서 이 정보를 바탕으로 특정 로직을 실행할 수 있다.


#### `@Target`
* 해당 애노테이션을 어디에 사용할 수 있는지 결정한다.
* 여러 타입을 지원하고 싶을 경우 배열로 입력하면 된다.
* **CONSTRUCTOR**, **METHOD**, **TYPE**, **ANNOTATION_TYPE**를 ElementType의 enum으로 받으 수 있다. 여기서 **TYPE**는 클래스 / 인터페이스 / 열거 타입을 뜻한다.


#### `@Documented`
* 해당 애노테이션을 사용한 코드의 문서에 그 애노테이션에 대한 정보를 표기할지 결정한다.
* 예시 : 위 `SampleController`에서 JavaDoc을 만들 때, 해당 애노테이션을 명세에 포함시킬 수 있다.


***
참고자료

1. [스프링 웹 MVC](https://inf.run/dJFi)
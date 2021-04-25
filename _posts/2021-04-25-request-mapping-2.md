---
layout: post
title: 요청 맵핑하기 (2) - URI 패턴
categories: [Spring]
tags: [Spring Framework, Spring MVC]
description: Spring MVC에서 URI 패턴으로 맵핑하는 방법
fullview: false
comments: true
---

### URI, URL, URN
URL과 URN은 URI의 일부이라고 생각하면 된다. URL은 로케이션 기반이며, URN은 네임 기반이다.

### 요청 식별자로 맵핑하기

* `@RequestMapping`은 아래의 패턴을 지원한다.
	* ? : 한 글자("/author/???" -> /author/123)
	* * : 여러 글자("/author/**" -> /author/cjk)
	* ** :  여러 경로("/author/**" -> author/new/cjk)

```java
@RequestMapping(value = {"/hello", "/hi"}) //여러 url을 배열의 형식으로 지정할 수 있다.
    public String hello(){
        return "hello";
    }
@RequestMapping(value = "/hello?") //hello1과 같은 hello뒤 문자 하나가 추가된 요청만 받는다.
    public String hello2(){
        return "hello2";
    }
```

### 클래스에 선언한 `@RequestMapping`과의 조합
* 클래스에 선언한 URI 패턴 뒤에 이어 붙여서 맵핑한다.

```java
@Controller
@RequestMapping("/hello")
public class SampleController {

    @RequestMapping(value = "/hi")
    public String hello2(){
        return "hello2";
    }
}
```
이 경우 /hello/hi로 요청하면 `hello2()`로 요청을 받게 된다.


### 정규 표현식을 통해서 맵핑하기
* `/{name:정규식}`

```java
@Controller
public class SampleController {

    @RequestMapping(value = "/{name:[a-z]+}")
    public String hello(@PathVariable String name){
        return "hello" + name;
    }
}
```
영문자에 해당하는 문자열을 name으로 받아 반환한다.

### 패턴이 중복되는 케이스는?
* 가장 구체적으로 맵핑되는 핸들러를 선택한다.

```java

@Controller
public class SampleController {

    @RequestMapping(value = "cjk")
    public String hello(@PathVariable String name){
        return "hello" + name;
    }

    @RequestMapping(value = "/**")
    public String hello2(){
        return "hello" + name;
    }
}
```

`/cjk`로 요청을 할 경우, 두가지 핸들러에 모두 해당되지만, 가장 구체적으로 선언한 `hello`핸들러를 선택한다.

### URI 확장자 맵핑 지원
* 스프링 MVC에서는 URI에 대한 확장 기능을 지원한다.
* 하지만 스프링 부트에서는 기본적으로 이 기능을 사용하지 않도록 설정해주고, 사용하지 않는 것을 권장한다.
	* 보안이슈(RFD attack)
	* URI 변수, Path 매개변수, URI 인코디 사용시 불명확하다.

```java
@RequestMapping({"/cjk","/cjk.*"}) //이렇게 "cjk"만 작성하여도 "cjk.*"를 추가하여 html, json등 다른 형식의 요청도 처리 가능하도록 설정해주는 것이 스프링 mvc의 특징
    public String hello(@PathVariable String name){
        return "hello" + name;
    }
```





***
참고자료

1. [스프링 웹 MVC](https://inf.run/dJFi)
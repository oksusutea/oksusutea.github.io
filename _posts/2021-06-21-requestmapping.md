---
layout: post
title: @RequestMapping 핸들러 매핑
categories: [spring]
tags: [java, spring, 토비의 스프링]
description: 스프링 @MVC에서 핸들러 매핑 관련 애노테이션
fullview: false
comments: true
---

* @MVC의 가장 큰 특징은 핸들러 매핑과 핸들러 어댑터의 대상이 컨트롤러 오브젝트가 아니라 메소드라는 점이다.
* @MVC 핸들러 매핑을 위해서는 DefaultAnnotationHandlerMapping이 필요하다. 이 전략은 디폴트 핸들러 매핑 전략이므로 다른 핸들러 매핑 빈을 명시적으로 등록하지 않았다면 기본적으로 사용할 수 있다.

## 클래스/메소드 결합 매핑정보
* DefaultAnnotationHandlerMapping의 핵심은 매핑정보로 @RequestMapping 애노테이션을 활용한다는 점이다.
* 기본적인 결합 방법은 타입 레벨의 @RequestMapping 정보를 기준으로 삼고, 메소드 레벨의 @RequestMapping 정보는 타입 레벨의 매핑을 더 세분화 하는데 사용한다.


### @RequestMapping 어노테이션 엘리먼트
#### String[] value() : URL 패턴

```java
@RequestMapping("/hello")
@RequestMapping("/main*")
@RequestMapping("/user/{userId}") // {}위치에 해당하는 내용을 컨트롤러 메소드에서 파라미터로 전달받을 수 있다.
```

* URL 패턴은 디폴트 접미어 패턴이 적용되어 "/hello"로 요청하면 "/hello/", "/hello..*"같은 URL을 적용했을 때와 동일한 결과가 나온다.

####  RequestMethod[] method() : HTTP 요청 메소드
* GET, HEAD, POST, PUT, DELETE, OPTIONS, TRACE 7개의 HTTP 메소드 중, 원하는 요청메소드에 맞게 처리 할 수 있다.

```java
@RequestMapping(value="/hello",method=RequestMethod.GET)
```

* 요청 메소드 외로 요청을 할 경우 **HTTP 405 - Method Not Allowed** 응답을 받는다.

####  String[] params() : 요청 파라미터
* 같은 URL을 사용하더라도 HTTP 요청 파라미터에 따라 별도의 작업을 해주고 싶을 때 사용한다.

```java
@RequestMapping(value="/user/edit",params="type=admin")
@RequestMapping(value="/user/edit",params="type=member")
@RequestMapping(value="/user/edit",params="!type")	// type라는 파라미터가 아예 존재하지 않는 경우에만 매핑
```

* /user/edit?type=admin일 경우 첫번째 매핑이 적용된다.
* 매핑 조건을 만족하는 경우가 여러 개 있을 때는 좀 더 많은 조건을 만족시키는 쪽이 우선된다.

####  String[] headers() : HTTP 헤더

```java
@RequestMapping(value="/view", headers = 'content-type=text/*")
```
* parmas와 비슷하게 '헤더이름=값'의 형식을 사용한다. 위와 같은 맵핑은 context-type이 text/html, text/plain등으로 되어있는 경우에만 매핑해준다.

### 타입 레벨 매핑과 메소드 레벨 매핑의 결합
* 타입(클래스와 인터페이스) 레벨에 붙는 @RequestMapping은 모든 타입 내의 모든 매핑용 메소드의 공통 조건을 지정할 때 사용한다. 그리고 메소드 레벨에서 조건을 세분화해준다.

### 메소드 레벨 단독 매핑
* 메소드 레벨의 매핑조건에 공통점이 없는 경우라면 타입레벨에는 조건을 주지 않고 메소드 레벨에서 독립적으로 매핑정보를 지정할 수도 있다.

```java
@RequestMapping
public class UserController {
	@RequestMapping("/hello") public String hello(...) {}
```

### 타입 레벨 단독 매핑
* @RequestMapping을 타입 레벨에 단독으로 사용해서 다른 타입 컨트롤러에 대한 매핑을 위해 사용할 수 있다.

```java
@RequestMapping("/user/*")
public class UserController {
	@RequestMapping public String add(..) {}
	@RequestMapping public String edit(...) { }
```

***

## 타입 상속과 매핑
* @RequestMapping이 적용된 클래스를 상속해서 컨트롤러로 사용하는 경우 슈퍼클래스의 매핑정보는 무시된다.(재정의 했을 경우에만)
* @RequestMapping은 클래스 말고 인터페이스에도 동일하게 적용된다. 

### 매핑정보 상속의 종류

#### 상위 타입과 메소드의 @RequestMapping 상속
* 슈퍼클래스에만 @RequestMapping을 적용하고 이를 그대로 상속한 서브클래스에는 아무런 @RequestMapping을 사용할 경우는 상위 클래스의 정보를 그대로 상속받는다.

```java
@RequestMapping("/user")
public class Super {
	@RequestMapping("/list")
	public String list() {...}
}

public class Sub extends Super {
}
```

* Sub는 Super의 @RequestMapping 정보를 상속받는다. override를 했더라도, @RequestMapping를 붙이지 않는다면 그대로 상속된다.
* 인터페이스도 동일하다

#### 상위 타입의 @RequestMapping과 하위 타입 메소드의 @RequestMapping 결합

```java
@RequestMapping("/user")
public class Super {
}

public class Sub extends Super {
	@RequestMapping("/list")
	public String list() {...}
}
```
* 위의 케이스의 경우 Sub 클래스를 빈으로 등록한다면 /user/list가 Sub 클래스의 list() 메소드로 매핑된다.
* 인터페이스의 경우에도 마찬가지다.


#### 상위 타입 메소드 @RequestMapping과 하위 타입의 @ReauestMapping 결합
* 슈퍼클래스에는 메소드에만 @RequestMapping이 있고, 서브클래스에는 반대로 클래스 레벨에 @RequestMapping이 부여된 경우다.

```java

public class Super {
	@RequestMapping("/list")
	public String list() {...}
}

@RequestMapping("/user")
public class Sub extends Super {

}
```
* 위와 같은 경우 /user/list는 Sub가 상속받은 list() 메소드에 매핑된다.
* 인터페이스의 경우도 동일한 방식으로 적용된다. 하지만 인터페이스를 구현하는 메소드에 URL이 없는 빈 @RequestMapping을 붙이면 인터페이스 메소드의 매핑정보가 무시된다.

#### 하위 타입과 메소드의 @RequestMapping 재정의
* 슈퍼클래스의 @RequestMapping은 모두 무시되고 새로 정의한 서브 클래스의 @RequestMapping이 적용된다.

#### 서브클래스 메소드의 URL 패턴 없는 @RequestMapping 재정의
* 하위 타입의 @RequestMapping은 항상 상위 타입의 @RequestMapping정보를 대신한다.
* 하지만 URL 조건이 없을 때에는 상위 메소드의 @RequestMapping가 재정의된다.


***
### 제네릭스와 매핑정보 상속을 이용한 컨트롤러 작성

* @RequestMapping의 상속과 구현을 잘 활용하면 반복적인 설정을 피하고 간결한 코드를 얻어 낼 수 있다.

```java
public class UserController {
	UserService service;
	
	public void add(User user) { ... }
	public void update(User user) { ... }
	public User view(Integer id) { ... }
	public void delete(Integer id) { ... }
	public List<User> list() { ... }
}
```

* 대부분의 도메인은 위와 같은 CRUD 메소드를 공통적으로 갖고 있다. 공통적으로 작성하는 @RequestMapping 코드를 줄이기 위해 아래와 같이 제너릭 추상 클래스를 만들어본다.

```java
public abstract class GenericController<T, K, S> {
	S service;
	
	@RequestMapping("/add")
	public void add(T entity) { ... }
	
	@RequestMapping("/update")
	public void update (T entity) { ... }
	
	@RequestMapping("/view")
	public void T view(K id) { ... }
	
	@RequestMapping("/delete")
	public void delete(K id) { ...}
	
	@RequestMapping("/list")
	public List<T> list() { ... }

}
```

* UserController는 GenericController를 상속하여 만들면 메소드에 대한 @RequestMapping을 추가로 설정해주지 않아도 된다.

```java
public class UserController extends GenericController<User, Integer, UserService> {

	@RequestMapping("/login")
	public String login(String userId, String password) { ... }

```

* 상위 타입에선 메소드에, 하위 타입에선 클래스에 @RequestMapping을 붙여서 이를 결합시키는 매핑정보 상속 방법을 응용한 것이다.
* 각 컨트롤러 별로 기본 CRUD 기능 외 추가할 메소드만 넣어주면 되므로 컨트롤러 코드는 매우 간결해지고 개발 생산성은 높아진다!


***
참고자료 :  
[토비의 스프링 3.1 Vol.2 4.1 - @RequestMapping 핸들러 매핑](https://www.aladin.co.kr/shop/wproduct.aspx?ItemId=19505671)


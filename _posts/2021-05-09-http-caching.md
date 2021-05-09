---
layout: post
title: 스프링 MVC 프레임워크에서의 HTTP Caching 전략
categories: [Spring]
tags: [Spring Framework, Spring MVC]
description: 스프링 Web MVC에서 Caching 처리 방법
fullview: false
comments: true
---

Spring Document에서 **스프링 웹 MVC**에서는 다루지 않는 HTTP Caching에 대해 알아본다.(정확히는 문서를 번역하며 이해해보자)


## 1.9 HTTP Caching
HTTP Caching은 웹 애플리케이션의 성능을 명확하게 향상시킬 수 있는 방법중 하나이다. HTTP caching은 `Cache-Control` 응답 헤더를 이용해 적용하고, 요청 헤더의 조건을 붙여 적용할 수도 있다.(`Last-Modified`와 `ETag`등을 이용해서 말이다) `Cache-Control`은  어떻게 캐싱을 할지, 그리고 응답을 재사용할것인지에 대해 (브라우저와 같은)private 캐시와 (프록시같은)public 캐시를 사용하도록 권장한다. `ETag`헤더는 컨텐트의 내용이 변경되지 않았을 경우, 본문의 내용을 전달하지 않고 304(NOT_MODIFIED) 코드를 전달할 수 있도록 하며, 요청에 주로 쓰인다. `ETag`는 `Last-Modified`헤더에 비해 조금 더 자주 쓰인다.
이번 섹션에는 스프링 웹 MVC에서 사용가능한 HTTP 캐싱과 관련된 옵션에 대해서 알아볼 것이다.

> ETag란? HTTP 컨텐츠가 바뀌었는지를 검사할 수 있는 태그이다. 같은 주소의 자원이더라도, 컨텐츠가 달라졌다면 ETag가 달라질 것이다. Last-Modified와 거의 동일하나 시간 대신 Hash 값을 사용한다는 측면에서 다르다.

### 1.9.1 `CacheControl`
`CacheControl`은`Cache-Control`헤더와 관련된 설정 셋팅을 할 수 있도록 하며, 아래 위치에서 아규먼트로 받을 수 있다 : 

* `WebContentInterceptor`
* `WebContentGenerator`
* 컨트롤러
* 스테틱 자원

[RFC 7234](https://datatracker.ietf.org/doc/html/rfc7234#section-5.2.2)에서는 `Cache-Control`응답 헤더에 모든 적용가능한 설정을 기재해두었지만, `CacheControl`타입은 대부분의 상황에서 적용이 가능하도록 케이스 지향적인 방법으로 사용하도록 해두었다.

```java
// Cache for an hour - "Cache-Control: max-age=3600"
CacheControl ccCacheOneHour = CacheControl.maxAge(1, TimeUnit.HOURS);

// Prevent caching - "Cache-Control: no-store"
CacheControl ccNoStore = CacheControl.noStore();

// Cache for ten days in public and private caches,
// public caches should not transform the response
// "Cache-Control: max-age=864000, public, no-transform"
CacheControl ccCustom = CacheControl.maxAge(10, TimeUnit.DAYS).noTransform().cachePublic();
```

`WebContentGenerator`는 간단한 `cachePeriod`속성(초단위로 정의됨)을 지원한다. 

* `-1` : `Cache-Control`응답 헤더를 생성하지 않는다. 즉 캐시와 관련된 헤더를 설정하지 않는다.
* `0` : `Cache-Control: no-store`설정을 사용해 캐시를 방지한다.
* `n > 0` : `Cache-Control : max-age=n`설정과 같이 `n`초동안 응답 캐싱을 지원한다.


### 1.9.2 컨트롤러
컨트롤러에서 HTTP Caching을 하도록 명시할 수 있다. 우리 또한 이런 방식으로 작업하는 것을 권장한다. 리소스에서`lastModified`나 `ETag` 값은 요청 헤더와 비교하기 전에(?) 계산을 먼저 진행해야 하기 때문이다. 컨트롤러에서는 `ETag`헤더나 `Cache-Control`셋팅 값을 `ResponseEntity`에 추가 할 수 있다, 아래와 같이 말이다 : 

```java
@GetMapping("/book/{id}")
public ResponseEntity<Book> showBook(@PathVariable Long id) {

    Book book = findBook(id);
    String version = book.getVersion();

    return ResponseEntity
            .ok()
            .cacheControl(CacheControl.maxAge(30, TimeUnit.DAYS))
            .eTag(version) // lastModified is also available
            .body(book);
}
```
만일 클라이언트가 앞서 작성한 url로 요청을 한 번 더 한 후, 다시 요청을 할 경우 컨텐트는 바뀐 부분이 없기 때문에 서버는 body를 빈 값으로, 그리고 상태 코드는 304(NOT_MODIFIED)로 응답을 보내게 된다. `ETag`와 `Cache-Control`헤더는 응답에 같이 전달된다.   
물론 컨트롤러 단에서 요청 헤더에 조건적으로 캐싱을 적용할 수도 있다 : 

```java
@RequestMapping
public String myHandleMethod(WebRequest request, Model model) {

    long eTag = ...  //1번

    if (request.checkNotModified(eTag)) {
        return null;  //2번
    }

    model.addAttribute(...); 
    return "myViewName"; //3번
}
```

* 1번 : eTag부분은 어플리케이션 레벨에서 eTag를 계산하는 부분이다.
* 2번 : eTag를 체크해 304(NOT_MODIFIED)로 응답을 보낸다, 추가적으로 요청 처리를 하지 않는다.
* 3번 : 요청 처리를 계속 한다.

`eTag`값이나, `lastModified`값이나 혹은 두 값 모두를 체크하여 캐싱을 처리할 수 있다. `GET`과 `HEAD`요청에서는 응답을 304(NOT_MODIFIED)로 전송할 수 있으며, `POST`, `PUT`, `DELETE`에서는 동시 다발적인 수정을 방지하기 위해서 412(PRECONDITION_FAILED)를 응답으로 설정할 수 있다.

### 1.9.3 Static Resources

`Cache-Control`이나 기타 방법을 통해 스태틱 자원을 캐싱하여 성능을 끌어올릴 수 있다. 이 부분에 대해서는 [Static Resources](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-config-static-resources)를 설정하는 챕터 페이지에서 확인할 수 있다.

> 간단하게 정리해보면, `WebMvcConfigurer`를 구현해 `addResourceHandler`에서 `setCacheControl`과 `addResourceLocations`등을 이용해 캐싱하는 정적파일의 위치, 캐싱설정을 할 수 있다.

### 1.9.4 `ETag` Filter
`ShallowEtagHeaderFilter`를 이용하여 **shallow**한 `eTag` 값을 지정할 수 있는데, 이 방법을 통해서 렌더링한 응답을 네트워크로 다시 보내지 않을 수 있어 대역폭(bandwidth)을 줄일 수 있다(CPU는 동일한 요청을 처리해야 하기에 실질적으로 줄일 수 없다)


*** 
참고 자료 : 

1. [1.9 HTTP Caching](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-caching)

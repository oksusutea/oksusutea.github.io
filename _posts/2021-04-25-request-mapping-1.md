---
layout: post
title: 요청 맵핑하기 (1) - HTTP Method
categories: [Spring]
tags: [Spring Framework, Spring MVC]
description: Spring MVC에서 HTTP Method를 요청하는 방법
fullview: false
comments: true
---



### GET 요청
* 클라이언트가 서버의 리소스를 요청할 때 사용한다.
* 캐싱 할 수 있다.(조건적인 GET으로 바뀔 수 있다== isNotModified 같은 헤더 사용)(캐시와 관련된 부분을 헤더에 추가 할 수 있다)
* 브라우저 기록에 남는다.
* 북마크 할 수 있다.
* 민감한 데이터를 보낼 때에는 사용해서는 안된다(URL에 노출된다)
* idemponent(멱등하다).

### POST 요청
* 클라이언트가 서버의 리소스를 수정하거나 새로 만들 때 사용한다.
* 서버에 보내는 데이터를 POST 요청 본문에 담는다.
* 캐시 할 수 없다.
* 브라우저 기록에 남지 않는다.
* 북마크 할 수 없다.
* non-idemponent(멱등하지 않다).
* 데이터 길이에 제한이 없다.


### PUT 요청
* URI에 해당하는 데이터를 새로 만들거나 수정할 때 사용한다.
* **POST**와 다른 점은 **URI**에 대한 의미가 다르다.
	* POST의 URI는 보내는 데이터를 처리할 리소스를 의미한다.
	* PUT의 URI는 보내는 데이터에 해당하는 리소스를 의미한다.

### PATCH 요청
* PUT과 비슷하지만, 기존 엔티티와 새 엔티티의 차이점만 보낸다는 차이가 있다.
* idemponent(멱등하다)

### DELETE 요청
* URI에 해당하는 리소스를 삭제할 때 사용한다.
* idemponent(멱등하다)


### 스프링 MVC에서 HTTP 메소드 요청하는 방법

* `@RequestMapping(method=RequestMethod.GET)`
* `@RequestMapping(method={RequestMethod.GET,RequestMethod.POST})`
* `@GetMapping`
* 메소드 뿐만 아니라, 클래스 단에서도 위 예약어를 기재할 수 있다. 클래스단에서 기재하게 되면, 컨트롤 안에 들어있는 모든 핸들러는 클래스 레벨에 지정한 요청만 받을 수 있다.
	

***
참고자료

1. [스프링 웹 MVC](https://inf.run/dJFi)
---
layout: post
title: 핸들러 메소드 (8) - @SessionAttribute
categories: [Spring]
tags: [Spring Framework, Spring MVC]
description: 세션에 들어있는 값을 참조하는 방법 - @SessionAttribute
fullview: false
comments: true
---

앞서 `@SessionAttributes`라는 애노테이션을 통해 세션기준으로 객체를 셋팅하는 방법을 배웠다. 여기서는 이 애노테이션과 비슷한(생긴건 거의 같은) `@SessionAttribute`에 대해 알아보도록 한다.

### `@SessionAttribute`
* HTTP 세션에 들어있는 값을 참조할 때 사용한다.
* `HttpSession`을 사용할떄에 비해 타입 컨버젼을 자동으로 지원해 조금 더 편리할 수 있다.
* `HTTP`세션에 데이터를 넣고 빼고 싶은 경우에는 `HttpSession`을 사용하자.


### `@SessionAttributes`와의 차이점
* `@SessionAttributes`는 해당 컨트롤러 내에서만 동작한다.
	* 즉, 해당 컨트롤러 안에서 다루는 특정 모댈 객체를 세션에 넣고 공유할 때 사용한다.
* `@SessionAttribute`는 컨트롤러 밖(인터셉터 또는 필터 등)에서 만들어준 세션 데이터에 **접근**할 때 사용한다.

***
참고자료

1. [스프링 웹 MVC](https://inf.run/dJFi)
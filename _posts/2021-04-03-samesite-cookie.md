---
layout: post
title: SameSite 쿠키 정책
categories: [Web]
tags: [Cookie, Web]
description: 크롬에서 추가된 SameSite 정책 관련 
fullview: false
comments: true
---

### SameSite Cookie란?
SameSite Cookie는 기본적으로 CSRF(Cross Site Request Forgery)공격을 막기 위해 추가된 정책이다. **서로 다른 도메인간의 쿠키 전송에 대한 보안을 설정**한다. CSRF란 사이트간 요청 위조라는 뜻의 웹사이트 취약점 공격 방식중 하나로, 사용자의 의지와는 관계없이 공격자가 의도한 행위를 웹사이트에 요청하는 것을 말한다.

### 쿠키 생성 주체에 따른 차이점
사이트 방문시에는 현재 방문한 사이트의 쿠키 뿐 아니라, 다양한 도메인의 쿠키가 존재한다.  현재 사이트의 도메인과 일치하는 쿠키를 `First Party Cookie`라고 한다. 그리고 현재 사이트 외 다른 도메인상의 쿠키를 `Third Party Cookie`라고 한다. 즉, 동일한 쿠키라 하더라도 내가 방문하고 있는 사이트에 따라 쿠키의 속성이 달라진다.

### SameSite의 판단 기준

> A request is "same-site" if its target's URI's origins' registrable domain is an exact match for **the request's client's "site for cookies",** or **if the request has no client**. The request is otherwise "cross-site".

위 말을 정리해보자면,

1) target's URI's origins registrable domain == request's client's "site for cookies"  
2) request가 클라이언트를 갖지 않을 때 
라고 한다. 영어이고 각각 단어도 아는데 무슨 뜻인지는 잘 모르겠다. 각각의 워딩에 대해 이제 알아보도록 하자. 

#### 1. TLD(top level domain)
TLD + 0레벨(public suffix)에 해당하는 도메인은 [ https://publicsuffix.org/list/public_suffix_list.dat ](https://publicsuffix.org/list/public_suffix_list.dat)에서 관리되며, "com", "github.io", "co.kr"등이 있다.
**TLD + 0레벨은 dot(.)기준 가장 오른쪽 문자열이 아니다.**

#### 2. registrable domain은 TLD + 1 레벨의 도메인
예를 들어, https://oksusutea.github.io의 registrable domain은 "oksusutea.github.io"이다. 

#### 3. 브라우저 주소창 URI's origin의 registrable domain을 top-level site라고 한다.
어떤 웹 브라우저에서 `iframe`등을 사용하지 않고 중첩된 브라우저가 없다면 "site for cookies"는 top-level site이다.
만일 중첩된 브라우저가 있다면, 가장 최상위 브라우저의 "site for cookies" 가 top-level site이다.

**결론적으로 Set-Cookie Domain의 registrable domain과 브라우저 주소창 URI의 registrable domain이 정확하게 일치할 경우 SameSite이다**

#### client가 null일때는 무조건 SameSite
보통은 브라우저 주소창에서 직접 타이핑하여 html이 바뀔 경우에 `request`의 `client`가 `null`이 된다.  
RFC 문서에는 아래와 같이 설명하였다. 

```
For a given request ("request"), the following algorithm returns "same-site" or "cross-site":
1. If "request"'s client is "null", return "same-site".
Note that this is the case for navigation triggered by the user directly (e.g. by typing directly into a user agent's address bar).
2. Let "site" be "request"'s client's "site for cookies" (as defined in the following sections).
3. Let "target" be the registrable domain of "request"'s current url.
4. If "site" is an exact match for "target", return "same-site".
5. Return "cross-site".
```

#### Schemeful SameSite
draft 07버전부터 scheme(http, https)까지 같아야 SameSite인 것으로 바뀌었다. 이 것을 통해 SameSite기준이 더 엄격해졌다고 볼 수 있다.
하기 스팩 알고리즘에서 `scheme`, `host`, `post`가 다를 경우 `False`를 반환하는 것을 볼 수 있다.

```
Two origins, A and B, are considered same-site if the following
   algorithm returns true:

   1.  If A and B are both the same globally unique identifier, return
       true.

   2.  If A and B are both scheme/host/port triples:

       1.  If A's scheme does not equal B's scheme, return false.

       2.  Let hostA be A's host, and hostB be B's host.

       3.  If hostA equals hostB and hostA's registrable domain is null,
           return true.

       4.  If hostA's registrable domain equals hostB's registrable
           domain and is non-null, return true.

   3.  Return false.

   Note: The port component of the origins is not considered.

   A request is "same-site" if its target's URI's origin is same-site
   with the request's client's "site for cookies" (which is an origin),
   or if the request has no client.  The request is otherwise "cross-
   site".
```

### SameSite Cookie의 속성

#### Strict

```
Set-Cookie: CookieName=CookieValue; SameSite=Strict;
```
제일 엄격한 설정 값이다. Third-party의 쿠키는 아예 금지한다(같은 웹사이트의 요청에만 쿠키를 전송할 수 있다.) 즉, 서로 다른 도메인에서는 아예 전송이 불가능하다.

#### Lax

```
Set-Cookie: CookieName=CookieValue; SameSite=Strict;
```
대다수의 요청시 Third-Party 쿠키는 전송하지 않는다. 하지만 **안전한 Http Method를 요청** 할 때 Cookie를 전송한다. None과의 차이점은 아래와 같다. 

|요청 유형|예시|None|Lax|
|------|---|---|---|
|a태그1|`<a href="..."></a>`|O|O|
|링크|`<link rel="prerender" href="..."/>`|O|O|
|GET 폼|`<form method="GET" action="...">`|O|O|
|POST 폼|`<form method="POST" action="...">`|O|X|
|iframe|`<iframe src="..."></iframe>`|O|X|
|ajax|`$.ajax("...")`|O|X|
|Image|`<img src="...">`|O|X|

이렇게 `Strix` 혹은 `Lax`로 설정하고 나면, 기본적으로 CSRF 공격은 막을 수 있다고 볼 수 있다.(물론 브라우저에서 `SameSite` 속성을 지원해야 한다)



#### None
```
Set-Cookie: CookieName=CookieValue; SameSite=None;
```
동일 사이트와 크로스 사이트에 모두 쿠키 전송이 가능하다. **크롬 80부터는` SameSite=None`으로 설정할 경우 `Secure`속성을 함께 추가해야만 쿠키가 적용된다.** (Secure 속성이 추가된 쿠키는 HTTPS에서만 전송이 가능하다.)


***
### Java에서의 `SameSite`추가 방법
자바에서 가장 많이 사용하는 `javax.servlet.http.Cookie` 클래스에서는 `SameSite`속성과 관련된 API를 지원하지 않는다. 이 속성을 셋팅하기 위해서는 `HttpServletResponse`의 `response` 객체에 Set-Cookie 헤더를 직접 추가해야 한다. 아래와 같이 말이다.

```java
response.setHeader("Set-Cookie", "Test1=TestCookieValue1; SameSite=Strict"); 
response.addHeader("Set-Cookie", "Test2=TestCookieValue2; Secure; SameSite=Lax");  //Chrome 80 Default 값
response.addHeader("Set-Cookie", "Test3=TestCookieValue3; Secure; SameSite=None");
```

위와 같이 첫 행에 `setHeader`메소드를 통해 쿠키를 추가 후, 다시 `setHeader`를 이용할 경우 동일이름의 헤더가 오버라이드되기 때문에 `addHeader`를 이용해주어야 한다.

***

### Tomcat에서 SameSite 속성 추가 방법
Tomcat WAS에서 지원하는 **Cookie Processor Component**를 이용하여 일괄로 쿠키에 대한 속성을 추가할 수 있다.
context.xml 파일 : 

```
<Context>
	...
	<CookieProcessor sameSiteCookies="Strict" />
</Context>
```


***
참고자료 

1. [Cookie SameSite 설정하기](https://ifuwanna.tistory.com/223)
2. [Cookie的 SameSite属性](http://www.ruanyifeng.com/blog/2019/09/cookie-samesite.html)
3. [Cookie SameSite](https://yangbongsoo.tistory.com/5?category=919814)
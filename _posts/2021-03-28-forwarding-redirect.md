---
layout: post
title: Forward vs Redirect
categories: [JSP]
tags: [Java, JSP]
description: 페이지를 이동하는 두가지 방식의 차이점에 대해 알아보기 
fullview: false
comments: true
---

웹에서는 현재 작업중인 페이지에서 다른 페이지로 이동하기 위해 두가지 방식을 제공한다. 이 두가지 방식의 차이와 사용법에 대해 알아보도록 한다. 

## Forward 방식
WAS의 서블릿 혹은 JSP가 요청을 받은 후, 처리하다가 추가적인 처리를 같은 웹 어플리케이션안에 포함된 다른 서블릿 혹은 JSP에 위임하는 경우가 있다. 이러한 방식을 **포워드(Forward)**라고 한다.   
<p style="center">

<img src="https://media.vlpt.us/images/denmark-choco/post/f763e8eb-234f-4edb-84c0-e18375d11b2f/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202020-07-29%20%E1%84%8B%E1%85%A9%E1%84%8C%E1%85%A5%E1%86%AB%2012.52.01.png">

</p>

<br/> 

주요 동작 과정은 아래와 같다 : 


1. 웹 브라우저에서 Servlet1에 요청을 보냄
2. Servlet1은 요청을 처리한 후, 그 결과를 HttpServletRequest에 저장
3. Servlet1은 결과과 저장된 HttpServletRequest와 HttpServletResponse를 같은 웹 어플리케이션에 있는 Servlet2에게 전송(forward)
4. Servlet2는 Servlet1로부터 전달받은 HttpServletRequest, HttpServletResponse를 이용해 요청을 처리한 뒤 웹 브라우저에게 결과 전송

**주요 특징**: 

 * 웹 컨테이너 안에서 페이지 이동만 하여, 실제 클라이언트는 다른 페이지로 이동을 했는지 알 수 없다. 
 * 클라이언트와 통신을 추가로 하지 않아 리다이렉트보다 더나은 성능을 보여준다.
 * 현재 실행중인 페이지와 새로 실행된 페이지는 `Request`와 `Response`객체를 공유한다.

 
## Redirect 방식
 Web Container로 요청을 받은 후, 처리를 하다가 웹 브라우저에게 다른 페이지로 이동하라고 명령을 내린다. 웹 브라우저는 URL을 바꿔 해당 주소로 이동한다. 다른 웹 컨테이너에 있는 주소로 이동하며 새로운 페이지에는 이전과는 다른 `Request`객체와 `Response`객체가 생성된다.
 
 
<p style="center">

<img src="https://t1.daumcdn.net/cfile/tistory/2516393958845B501C" width="518" height="333">
</p>

<br/>
 
 주요 동작 과정은 아래와 같다 : 

1.웹 브라우저에서 Servlet1에 요청을 보냄
2. Servlet1은 요청을 처리한 후, /url2와 같은 다른 url을 접근하도록 클라이언트에 지시
3. 브라우저는 /url2에 새롭게 접근
4. /url2에 맵핑된 Servlet2가 새로운 요청을 처리
5. 요청 결과를 반환


### Forward와 Redirect의 차이
* forward는 URL의 변경없이 페이지가 이동되지만, redirect는 URL이 변경된다.
* forward는 `Request`, `Response` 객체를 공유하지만, redirect는 새로 요청을 받는 것이기 때문에 공유되지 않는다.  

사용자가 보낸 요청정보를 이용하여 글쓰기 기능을 수행할 경우 어떤 것을 선택해야 할까? 이 때는 redirect를 사용해야 한다. 사용자가 실수 혹은 고의로 새로고침을 누르게 되면, `forward`의 경우 요청정보가 그대로 남아있어 똑같은 글이 여러번 등록될 수 있다. 하지만 `redirect`는 작성할 때 보냈던 요청정보는 존재하지 않고 URL이 달라지기 때문에 글쓰기가 여러번 수행되지 않는다. 
**즉, 시스템(DB, session)에 변화가 생기는 요청(로그인, 회원가입, 글쓰기)은 `redirect`방식으로 응답하는 것이 바람직하며, 시스템에 변화가 생기지 않는 단순 조회(리스트보기, 검색)의 경우 `forward`방식으로 응답하는 것이 바람직하다.**


***
참고 자료 : 
1. [HTTP_Redirect와 Forward의 차이](https://velog.io/@denmark-choco/HTTPRedirect%EC%99%80-Forward%EC%9D%98-%EC%B0%A8%EC%9D%B4)  
2. [Redirect vs Forward(Redirect와 forward의 차이)](https://doublesprogramming.tistory.com/63)


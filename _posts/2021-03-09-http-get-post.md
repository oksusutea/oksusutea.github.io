---
layout: post
title: HTTP내 GET, POST의 차이
categories: [CS]
tags: [network]
description: HTTP 프로토콜을 이용해 서버에 요청할 때 보내는 GET, POST 방식의 차이
fullview: false
comments: true
---


## GET

* 요청하는 데이터가 `Http Request Message`의 Header부분의 URL에 담겨 전송된다.
* URL의 끝에 ?와 함께 데이터가 붙어 request한다.
* 전달할 수 있는 데이터의 크기가 한정적이다.
* URL에 요청데이터가 그대로 전달되어 보안에 취약하다.
* 해당 요청은 브라우저에서 캐싱할 수 있다.

## POST

* `Http Request Message`의 Body부분에 데이터가 담겨져서 들어간다.
* 데이터의 크기에 제한이 없다.
*  요청시 헤더의 Content-Type에 요청 데이터의 타입을 표시해야 한다. 그러지 않을 경우 내용 혹은 URL에 포함된 리소스의 확장자명으로 유추한다.

<br/>
## 그렇다면 근본적인 차이점은?
GET방식은 Idempotent(멱등), POST는 Non-idempotent하게 설계되었다.   
GET으로 서버에게 동일한 요청을 여러 번 전송하더라도 동일한 응답이 돌아와야 하도록 설계(==멱등)되었기 때문에, 주로 조회할 때 사용한다. 반면, POST는 Non-idempotent이기에 서버에게 동일한 요청을 하더라도 응답은 항상 다를 수 있다. 이에 따라 POST는 서버의 상태나 데이터를 변경시킬 때 사용된다.


---
layout: post
title: 메시지 컨버터와 AJAX
categories: [spring]
tags: [java, spring, 토비의 스프링]
description: 웹서비스 개발시 사용하는 메시지 컨버터
fullview: false
comments: true
---

* 메시지 컨버터는 XML이나 JSON을 이용한 AJAX 기능이나 웹 서비스를 개발할 때 사용할 수 있다.
* HTTP 요청 프로퍼티를 모델 오브젝트의 프로퍼티에 개별적으로 바인딩하고, 모델 오브젝트를 다시 뷰를 이용해 클라이언트로 만들 컨텐트로 만드는 대신 HTTP 요청 메시지 본문과 응답 메시지 본문을 통째로 메시지로 다루는 방식이다.
* 메시지 컨버터는 @RequestBody, @ResponseBody를 이용해 쓸 수 있다.
* 메시지 방식의 컨트롤러 사용 방법 : 
	* GET : 요청정보가 URL과 쿼리 스트링으로 제한되므로 @RequestBody 대신 @RequestParam, @ModelAttribute로 요청 파라미터를 전달받는다.
	* POST : HTTP 요청 본문이 그대로 전달되므로 @RequestBody 사용 가능

	
## 메시지 컨버터의 종류
* 사용할 메시지 컨버터는 AnnotationMethodHandlerAdapter를 통해 등록한다. 
* 일반적으로 하나 이상의 메시지 컨버터를 등록해두고 요청 타입이나 오브젝트 타입에 따라 선택되게 한다.
* AnnotationMethodHandlerAdapter를 통해 등록되는 디폴트 메시지 컨버터는 다음과 같다 : 


### ByteArrayHttpMessageConverter
* 지원 오브젝트 타입 : byte[]
* 지원 미디어 타입 : 모든 것
* 모든 종류의 HTTP 요청 메시지 본문을 byte 배열로 가져올 수 있다.
* @ResponseBody로 보낼 경우 컨텐트 타입이 application/octet-stream으로 설정된다.
* 자주 쓰이지 않으며, 클라이언트에게 바이너리 정보를 전달할 필요가 있을 때 주로 사용된다.

### StringHttpMessageConverter
* 지원 오브젝트 : String
* 지원 미디어 타입 : 모든 것
* HTTP 요청의 본문을 그대로 스트링으로 가져올 수 있다.
* 가공하지 않은 본문을 직접 받아 사용하고 싶을 때 사용 가능
* @ResponseBody로 보낼 경우 컨텐트 타입이 text/plain 으로 설정된다.
* 단순 문자열로 응답을 보내고 싶을 때 @ResponseBody와 함께 스트링 리턴 값을 사용하면 된다.

### FormHttpMessageConverter
* 지원 오브젝트 : MultiValueMap<String, String> (맵의 값이 List타입인 맵으로, 하나의 이름을 가진 여러 개의 파라미터가 사용될 수 있는 HTTP 요청 파라미터를 처리하기에 적당하다.)
* 지원 미디어 타입 : application/x-www-form-urlencoded
*  @ResponseBody로 보낼 경우 컨텐트 타입이 application/x-www-form-urlencoded 으로 설정된다.
*  HTTP 요청 폼 정보는 @ModelAttribute를 사용해 바인딩 하는것이 편아므로, FormHttpMessageConverter를 사용할 일은 많지 않을 것 같다.

### SourceHttpMessageConverter
* 지원 미디어 타입 : application/xml, application/*+xml, text/xml
* 지원 오브젝트 타입 : javax.xml.transform.Source 타입인 DOMSource, SAXSource, StreamSource 세 가지를 지원한다.
* XML 문서를 Source 타입의 오브젝트로 전환하고 싶을 때 유용하게 쓸 수 있다.

<br/>

* 위 네가지 컨버터가 디폴트로 등록되지만, 이보다 디폴트로 등록되지 않은 다음 세가지 HttpMessageConverter가 실제로 더 유용하다.
* 이 중 필요한게 있다면 직접 AnnotationMethodHandlerAdapter 빈의 messageConverters 프로퍼티에 등록하고 사용해야 한다.
* 여타 전략과 마찬가지로 전략 프로퍼티를 직접 등록하면, 디폴트 전략은 자동으로 추가되지 않는 점을 유의하자.

### Jaxb2RootElementHttpMessageConverter
* 지원 미디어 타입 : XML 미디어 타입
* JAXB의 @XmlRootElement, @XmlType이 붙은 클래스를 이용해 XML과 오브젝트 사이 메시지 변환을 지원한다.

### MarshallingHttpMessageConverter
* 지원 미디어 타입 : XML 미디어 타입
* 지원 오브젝트 : unmarshaller의 supports() 메소드를 호출해서 판단
* 스프링 OXM 추상화의 Marshaller와 Unmarshaller를 이용해 XML 문서와 오브젝트 사이의 변환을 지원해주는 컨버터.
* MarshallingHttpMessageConverter를 빈으로 등록할 때 프로퍼티에 marshaller, unmarshaller를 설정해줘야 한다.

### MappingJacksonHttpMessageConverter
* 지원 미디어 타입 : application/json
* 지원 자바 오브젝트 타입 : 프로퍼티를 가진 자바빈 스타일 혹은 HashMap을 이용해야 정확한 변환 결과를 얻을 수 있다.
* Jasckson ObjectMapper를 이용해 자바오브젝트와 JSON 문서를 자동변환해주는 메시지 컨버터다.

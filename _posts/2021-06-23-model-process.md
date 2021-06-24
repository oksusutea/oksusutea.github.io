---
layout: post
title: 모델의 전달 과정
categories: [spring]
tags: [java, spring, 토비의 스프링]
description: MVC에서 모델과 관련된 부분의 흐름
fullview: false
comments: true
---

### HTTP 요청으로부터 컨트롤러 메소드까지
* **@ModelAttribute 메소드 파라미터**
	* 컨트롤러 메소드의 모델 파라미터와 @ModelAttribute로부터 모델 이름, 모델 타입 정보를 가져온다.
* **@SessionAttribute 세션 저장 대상 모델 이름**
	* 모델 이름과 동일한 것이 있다면 HTTP 세션에 저장해둔 것이 있는지 확인한다.
	* 있을 경우 모델 오브젝트를 새로 만드는 대신, 세션에 저장된 것을 가져와 사용한다.
* **WebDataBinder에 등록된 프로퍼티 에디터, 컨버전 서비스**
	* WebBindingInitializer나 @InitBinder 메소드를 통해 등록된 변환 기능 오브젝트를 이용해 HTTP 요청 파라미터를 모델의 프로퍼티에 맞도록 변환해서 넣어준다.
	* 커스텀 프로퍼티 에디터 > 컨버전 서비스 > 디폴트 프로퍼티 에디터 순으로 적용된다.
	* 타입 변환에 실패시 BindingResult 오브젝트에 필드 에러가 등록된다.
	* 바인딩할 HTTP 파라미터가 없다면 이 과정은 생략된다.
* **WebDataBinder에 등록된 검증기**
	* 모델 파라미터에 @Valid가 지정되어 있다면 WebBindingInitializer나 @InitBinder 메소드를 통해 등록된 검증기로 모델을 검증한다.
	* 검증 로직을 갖고 있는 Validator 또는 JSR-303의 애노테이션 방식의 검증기를 이용한다.
	* 검증결과는 바인딩시와 동일하게 BindingResult 오브젝트에 등록된다.
	* WebDataBinder에 등록된 검증기가 없거나, @Valid 어노테이션이 없다면 이 과정은 생략된다.
* **ModelAndView의 모델 맵**
	* 모델 오브젝트는 컨트롤러 메소드가 실행되기 전에 임시 모델 맵에 저장된다.
	* 이렇게 저장된 모델 오브젝트는 컨트롤러 메소드의 실행을 마친 뒤에 등록된 모델 오브젝트와 함께 ModelAndView 모델 맵에 담겨 DispatcherServlet으로 전달된다.
* **컨트롤러 메소드와 BindingResult 파라미터**
	* HTTP 요청을 담은 모델 오브젝트가 @ModelAttribute 파라미터로 전달되면서 컨트롤러 메소드가 실행된다.
	* 메소드의 모델 파라미터 다음에 BindingResult 타입 파라미터가 있다면 바인딩과 검증 작업의 결과가 담긴 BindingResult 오브젝트가 제공된다. 이 오브젝트는 ModelAndView의 모델 맵에도 자동으로 추가된다.


* 이 과정은 모델 오브젝트 타입의 메소드 파라미터가 정의되어 있는지 파악하는 것으로 시작해서 메소드 파라미터에 HTTP 요청을 담은 모델 오브젝트가 전달되기까지의 과정이다.

### 컨트롤러 메소드로부터 뷰까지

* **ModelAndView의 모델 맵**
	* 컨트롤러 메소드의 실행을 마치고 최종적으로 DispatcherServlet이 전달받는 결과다.
	* 메소드 안에서 직접 또는 간접적으로 생성된 @ModelAttribute 오브젝트가 ModelAndView의 모델 맵에 담겨 있다.
	* BindingResult 오브젝트도 자동으로 추가된다.
* **WebDataBinder에 기본적으로 등록된 MessageCodeResolver**
	* WebDataBinder에 등록되어 있는 MessageCodeResolver는 바인딩 작업 또는 검증 작업에서 등록된 에러 코드를 확장해서 메시지 코드 후보 목록을 만들어준다.
	* 메시지 코드 후보를 만드는 로직은 WebDataBinder에 디폴트로 등록된 것을 사용한다.
* **빈으로 등록된 MessageSource와 LocalResolver**
	* LocaleResolver에 의해 결정된 지역정보와 MessageCodeResolver가 생성한 메시지 코드 후보 목록을 이용해 MessageSource가 뷰에 출력할 최종 에러 메시지를 결정한다.
	* MessageSource는 기본적으로 messages.properties 파일을 이용하는 ResourceBundleMessageSource를 등록해서 사용한다.
* **@SessionAttribute 세션 저장 대상 모델 이름**
	* 모델 맵에 담긴 모델 중, @SessionAttribute로 지정한 이름과 일치하는 것이 있다면 HTTP 세션에 저장된다.
* **뷰의 EL과 스프링 태그 또는 매크로**
	* 뷰에 의해 최종 콘텐트가 생성될 때 모델 맵으로 전달된 모델 오브젝트는 뷰의 표현식(EL)을 통해 참조돼서 콘텐트에 포함된다.
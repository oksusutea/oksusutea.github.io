﻿---
layout: post
title: JSP와 서블릿, Spring에서 발생할 수 있는 여러 문제점
categories: [java]
tags: [java, spring, 자바 성능 튜닝 이야기]
description: JSP와 Servlet 동작 원리
fullview: false
comments: true
---

* 자바 기반의 시스템 중, WAS에서 병목 현상이 발생할 수 있는 부분을 세밀하게 나눈다고 하면, UI부분(서버에서 수행됨)과 비즈니스 로직 부분으로 나눌 수 있다.
* 자바 기반의 서버단 UI를 구성하는 대부분의 기술은 JSP와 서블릿을 확장하는 기술이기 때문에 그만큼 이 두가지의 기본에 대해 잘 알아야 한다.

## JSP와 Servlet의 기본적인 동작 원리

* JSP와 같은 웹 화면단을 처리하는 부분에서 소요되는 시간은 많지 않다. 가장 처음 호출되는 경우에만 시간이 소요되고, 그 이후의 시간에는 컴파일된 서블릿 클래스가 수행되기 때문이다.
* JSP의 라이프 사이클은 아래와 같다.
	1. JSP URL 호출
	1. 페이지 번역
	1. JSP 페이지 컴파일
	1. 클래스 로드
	1. 인스턴스 생성
	1. jspInit 메서드 호출
	1. _jspService 메서드 호출
	1. jspDestroy 메서드 호출

* 해당 JSP파일이 이미 컴파일 되어있고, 클래스가 로드되어 있으며 JSP파일이 변경되지 않았다면 2~4번 프로세스는 생략된다.

* 서블릿의 라이프 사이클은 아래와 같다. WAS의 JVM이 시작된 이후
	* Servlet 객체가 자동으로 생성되고 초기화 되거나
	* 사용자가 해당 Servlet을 처음으로 호출했을 때 생성되고 초기화 된다.

* 정리하면 생성 -> 초기화 -> 사용가능 -> 파기 단계로 구성되어 있다고 볼 수 있다.
* 서블릿은 JVM에 여러 객체로 생성되지 않기 때문에, 서블릿 클래스의 메서드 내 인스턴스 변수를 선언하지 않도록 조심하자.

### 적절한 include 사용하기
* include 기능 사용시, 하나의 JSP에서 다른 JSP를 호출하여 여러 JSP파일을 혼합해 하나의 JSP로 만들 수 있다.
* JSP에서 사용가능한 include 방식은 아래 두가지가 있다:
	* 정적인 방식 : <%@include file="URL"%>
		* JSP의 라이프 사이클 중, JSP 페이지 번역 및 컴파일 단계에서 필요한 JSP를 읽어 메인 JSP의 자바 소스 밀 클래스에 포함 시키는 방식
	* 동적인 방식 : <jsp:include page="URL" />
		* 페이지가 호출될 때마다 지정된 페이지를 불러들여 수행하도록 하는 방식

* 정적인 방식이 동적인 바익보다 약 30배 더 빠르다.
* 하지만 정적인 방식 사용시 메인 JSP에 추가되는 JSP가 생겨 이름이 같은 JSP를 추가할 경우 오류가 발생한다.

### 자바 빈즈, 상황에 맞게 잘 사용하기
* 자바 빈즈는 UI에서 서버 측 데이터를 담아 처리하기 위한 컴포넌트이다.
* JSP에서 자바 빈즈를 처리하기 위해 소요되는 시간은 상당하므로, 적절하게 TO클래스를 만들고 해당 클래스로 받도록 변환하는 방법이 낫다.

### 태그 라이브러리
* 태그 라이브러리는 JSP에서 공통적으로 반복되는 코드를 클래스로 만들고, 그 클래스를 HTML 태그와 같이 정의된 태그로 사용할 수 있도록 하는 라이브러리이다.
* 태그 라이브러리 클래스를 잘못 작성하거나, 태그 라이브러리 클래스로 전송되는 클래스가 많을 때 성능에 문제가 된다.
*  	태그 라이브러리는 태그 사이에 있는 데이터를 넘겨주어야 하는데, 이 때 문자열 타입으로 넘겨준다. 따라서 데이터가 많을수록 처리를 해야 하는 내용이 많아지고, 태그 라이브러리 클래스에서 처리되는 시간이 많아진다.

### 정리
* 화면의 성능이 좋지 않을 때는 화면을 담당하는 JSP나 서블릿이 원인인 경우가 발생한다.
* include 구문도 정적인 구문을 사용할지, 동적 구문을 사용할지 잘 선택해야 하낟.
* 자바 빈즈를 너무 많이 사용하는 것은 성능에 많은 영향을 줄 수 있다.
* 태그 라이브러리도 상황에 맞게 사용해야 한다.
* 성능에 대한 부분도 중요하지만, 가장 중요한 것은 화면을 구성하는 것이다. 특히 에러 페이지를 어떻게 구성하느냐가 중요하다.



***
참고자료 : 
[자바 성능 튜닝 이야기 11장 - JSP와 서블릿, Spring에서 발생할 수 있는 여러 문제점](http://www.kyobobook.co.kr/product/detailViewKor.laf?mallGb=KOR&ejkGb=KOR&barcode=9788966260928)

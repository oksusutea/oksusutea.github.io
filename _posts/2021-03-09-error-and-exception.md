---
layout: post
title: Throwable, error 그리고 Exception
categories: [Java]
tags: [Java]
description: 자바에서 예외처리를 하는 클래스
fullview: false
comments: true
---

서비스 개발 도중, 혹은 구동 중에 잘못된 무언가로 인해 전체 시스템이 무너지는 결과를 방지하기 위해 자바에서는 **예외처리** 작업을 지원해준다. 예외처리와 관련된 클래스는 **Throwable**인터페이스를 구현하였으며, 상황에 따라 **Error**와 **Exception**으로 구분된다.

<p style = "center">
<img src = "https://media.geeksforgeeks.org/wp-content/uploads/Exception-in-java1.png">
</p>

## Error  
시스템에 비정상적인 상황이 생겼을 때 발생한다. **시스템 레벨**에서 발생하기에 심각한 수준의 오류이며,  시스템에 변화를 주어 문제를 처리해야 하는 경우가 일반적이다. 개발자가 미리 예측하여 처리할 수 없기 때문에 이에 따른 오류 처리를 신경쓰지 않아도 된다.  
* 예시 : JVM, OOM


## Exception
개발자가 구현한 애플리케이션 로직상에서 발생하며, **사용자의 잘못된 동작** 혹은 **개발자의 잘못된 코딩**으로 발생하는 프로그램 오류이다.  
* 예시 : JVM은 정상 동작하나, 배열 조회시 인덱스 범위 초과 등 발생


### Checked Exception

* 컴파일단계에서 확인하기에 실행하기 전에 체크 가능한 예외이다.
* 예외처리 발생시 롤백하지 않는다.
* 반드시 로직을 try/catch로 감싸거나 throw로 던져 처리해야 한다.
* 대표적인 예외 클래스 : 
	* Runtime Exception을 제외한 모든 예외(IO Exception)


### Unchecked Exception( => Runtime Exception)
* 실행단계 시점에 확인하기에 명시적으로 예외처리를 강제하지 않는다.
* 실행 과정 중 어떤 특정 논리에 의해 발견된다.
* 예외처리 발생시 롤백한다.
* 대표적인 예외 클래스 : 
	* Runtime Exception 하위 예외 클래스
	* NullPointerException
	* IndexOutOtBoundException
	* IllegalArgumentException

***
## 스프링에서 Exception을 처리하는 방법

### 통일된 Error Response 객체 정의

* 클라이언트에서 동일한 로직으로 예외처리를 하기 위해 `ResponseEntity<ErrorResponse>` 생성
* Response에는 message, code, status, errors등을 담을 수 있도록 구성

### @ControllerAdvice 어노테이션을 통해 모든 예외 핸들링

### Error Code는 Enum타입으로 한 곳에서 정리
* 에러코드가 전체적으로 흩어져 있을 경우 코드, 메시지의 중복 방지 어려움


### 필수 파라미터가 없거나, 비즈니스 로직 에러일 경우 `RuntimeException`을 상속받는 커스텀 클래스 생성후 사용한다.
* 요구사항에 맞게 개발자가 직접 Exception 발생 (e.x : 쿠폰 만료, 제품 매진 등)

### 컨트롤러 예외 처리
* 컨트롤러에서 모든 요청에 대한 값 검증 진행.

###


###


***
참고 자료  
1. [Excpetions in Java](https://www.geeksforgeeks.org/exceptions-in-java/)
2. [Spring Guide - Exception 전략](https://cheese10yun.github.io/spring-guide-exception/)
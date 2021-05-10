---
layout: post
title: Throwable, error 그리고 Exception
categories: [Java]
tags: [Java]
description: 자바, 스프링에서 예외처리를 하는 클래스
fullview: false
comments: true
---

서비스 개발 도중, 혹은 구동 중에 잘못된 무언가로 인해 전체 시스템이 무너지는 결과를 방지하기 위해 자바에서는 **예외처리** 작업을 지원해준다. 예외처리와 관련된 클래스는 **Throwable**인터페이스를 구현하였으며, 상황에 따라 **Error**와 **Exception**으로 구분된다.

<p style = "center">
<img src = "https://media.geeksforgeeks.org/wp-content/uploads/Exception-in-java1.png">
</p>

## Error  
`java.lang.Error`클래스의 서브 클래스이다. 시스템에 비정상적인 상황이 생겼을 때 발생한다. **시스템 레벨**에서 발생하기에 심각한 수준의 오류이며,  시스템에 변화를 주어 문제를 처리해야 하는 경우가 일반적이다. 개발자가 미리 예측하여 처리할 수 없기 때문에 이에 따른 오류 처리를 신경쓰지 않아도 된다. 
* 예시 : JVM, OOM, ThreadDath


## Exception
개발자가 구현한 애플리케이션 로직상에서 발생하며, **사용자의 잘못된 동작** 혹은 **개발자의 잘못된 코딩**으로 발생하는 프로그램 오류이다.  
* 예시 : JVM은 정상 동작하나, 배열 조회시 인덱스 범위 초과 등 발생


### Checked Exception

* 컴파일단계에서 확인하기에 실행하기 전에 체크 가능한 예외이다.
* 예외처리 발생시 **롤백하지 않는다**.
* 반드시 로직을 try/catch로 감싸거나 throws를 정의해 메소드 밖으로 던져 처리해야 한다.
* 대표적인 예외 클래스 : 
	* Runtime Exception을 제외한 모든 예외(IO Exception, SQLException)


### Unchecked Exception( => Runtime Exception)
* 실행단계 시점에 확인하기에 명시적으로 예외처리를 강제하지 않는다.
* 실행 과정 중 어떤 특정 논리에 의해 발견된다.
* 예외처리 발생시 **롤백한다**.
* 대표적인 예외 클래스 : 
	* Runtime Exception 하위 예외 클래스
	* NullPointerException
	* IndexOutOtBoundException
	* IllegalArgumentException

***
## 예외처리 방법
먼저 예외를 처리하는 일반적인 방법을 살펴보고 나서, 효과적인 예외처리 전략을 생각해보자.

### 예외 복구
예외상황을 파악하고 문제를 해결해 정상 상태로 되돌려 놓는 것이다. 예외로 인해 기본 작업 흐름이 불가능하면 다른 작업 흐름으로 자연스럽게 유도해 주는 것이다. 예외처리 코드를 강제하는 체크 예외들은 이렇게 예외를 어떤 식으로든 복구할 가능성이 있는 경우 사용한다.(API를 사용하는 개발자로 하여금 예외상황이 발생할 수 있음을 인식하도록 도와주고 적절한 처리를 시도하도록 요구할 수 있다)

```java
int maxretry = TRY_NUM;
while(maxretry --> 0){
	try {
		// 예외가 발생할 가능성이 있는 시도
		return; // 작업 성공
	} catch (SomeException e) {
		// 로그 출력. 정해진 시간만큼 대기
	}
	finally {
		// 리소스 반납, 정리 작업 진행
	}
	
	throw new RetryFailedException(); // 최대 재시도 횟수를 넘기면 직접 예외를 발생
}
```
**예외가 발생하더라도 어플리케이션의 로직은 정상적으로 실행이 되게 하도록 처리**하ㅕ였다. 위 예제는 예외가 발생하면 일정 시간동안 대기를 시킨 후 다시 해당 로직을 시도하도록 하였다. 일정 횟수동안 재시도를 진행하여도 정상적인 응답이 오지 않을 경우 오류를 발생시키는 로직이다.

### 예외처리 회피
**예외처리를 자신이 담당하지 않고 자신을 호출한 쪽으로 던져버리는 것**이다. throws문으로 선언해 예외가 발생하면 알아서 던져지게 하거나, catch문으로 일단 예외를 잡은 후 로그를 남기고 다시 예외를 던지는(rethrow) 방법이 있다.

```java
public void add() throws SQLException {
	//JDBC API
}
```

```java
public void add() throws SQLException {
	try {
		// JDBC API
	} catch(SQLException e){
		//로그 출력
		throw e;
	}
}
```
예외를 회피하는 것은, 예외를 복구하는 것처럼 의도가 분명해야 한다. 콜백/템플릿처럼 긴밀한 관계에 있는 다른 오브젝트에게 예외처리 책임을 분명히 지게 하거나, 자신을 사용하는 쪽에서 예외를 다루는 게 최선의 방법이라는 분병한 확신이 있어야 한다.

### 예외 전환
**발생한 예외에 대해 또 다른 예외로 변경하여 던지는 것**을 말한다. 예외 회피와 비슷하게 예외를 복구하여 만들 수 없기 때문에 예외를 메소드 밖으로 던지는 것은 비슷하지만, 예외를 그대로 넘기는 게 아니라 **적절한 예외로 전환해서 던진다는 특징**이 있다.  
예외 전환은 보통 두가지 목적으로 사용된다. 첫번째는 **내부에서 발생한 예외를 그대로 던지는 것이 그 예외상황에 대한 적절한 의미를 부여해주지 못하는 경우, 의미를 분명하게 해줄 수 있는 예외로 바꿔주기 위해서**다.

```java
public void add(User user) throws DuplicateUserIdException, SQLException {
	try {
		// JDBC를 이용해 user 정보를 DB에 추가하는 코드 또는
		// 그런 기능을 가진 다른 SQLException을 던지는 메소드를 호출하는 코드
	} catch(SQLException e) {
		// ErrorCode가 MySQL의 Duplicate Entry일경우 예외 전환
		if (e.getErrorCode() == MysqlErrorNumbers.ER_DUP_ENTRY) 
			throw DuplicateUserIdException();
		else
			throw e; // 그 외는 SQLException 그대로 발생
	}
}
```


보통 전환 예외에 원래 발생한 예외를 담아 *중첩 예외(nested exception)*으로 만드는 것이 좋다고 한다. 중첩 예외에서는 `getCause()`메소드를 통해 처음 발생한 예외가 무엇인지 확인할 수 있다. 해당 방식을 구현하기 위해 아래와 같이 생성자 혹은 `initCause()`메소드로 근본 원인이 되는 예외를 넣어주면 된다.

```java
catch(SQLException e) {
	...
	throw DuplicateUserIdException(e);				// 생성자 방식
	throw DuplicateUserIdException().initCause(e); 		// 메소드방식
```
두번째는 **예외를 처리하기 쉽고 단순하게 만들기 위해 포장하는 것**이다. 주로 예외처리를 강제하는 체크 예외를 언체크 예외인 런타임 예외로 바꾸는 경우 사용된다.  
대부분의 서버 환경에서는 애플리케이션 코드에서 처리하지 않고 전달된 예외들을 일괄적으로 다룰 수 있는 기능을 제공한다. 어차피 복구하지 못할 예외라면 애플리케이션 코드에서는 런타임 예외로 포장해서 로그를 남기고, 관리자에게는 메일을 남기고 사용자에게는 안내 메시지를 보여주는 식으로 처리하는 것이 바람직하다.



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
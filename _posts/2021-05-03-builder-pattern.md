---
layout: post
title: Builder Pattern
categories: [Spring]
tags: [Design Pattern]
description: 빌더 패턴과 롬복에서 빌더 패턴을 사용하는 방법에 대해 알아보기
fullview: false
comments: true
---

롬복을 사용하며 **@Builder**애노테이션을 접하게 되었다. 이 애노테이션은 언제 쓰이는지, 왜 쓰이는지 알아보기 위해 원초적인 개념부터 알아보기로 하였다.

### Effective Java에서 말하는 Builder 패턴

> 규칙 2: 생성자 인자가 많을 때에는 Builder 패턴 적용을 고려하라.  
> 아이템 2. 생성자에 매개변수가 많다면 빌더를 고려하라.

이펙티브 자바에서 말하는 빌더 패턴은 객체 생성을 깔끔하고 유연하게 하기 위한 기법이다. 빌더 객체를 소개하기 앞서 점층적 생성자 패턴과 자바빈 패턴에 대해 알아보자.

#### 점층적 생성자 패턴

* 필수 인자를 받는 **필수 생성자**를 하나 만든다.
* 1개의 선택적 인자를 받는 생성자를 추가한다.
* 2개의 선택적 인자를 받는 생성자를 추가한다.
* .....반복한다.
* 모든 선택적 인자를 다 받는 생성자를 추가한다.

예시 코드 : 

```java
public class Member{
	private final String name; //필수 입력 값
	private final String location;
	private final String hobby;
	
	public Member(String name){
		this(name,null,null);
	}
	public Member(String name, String location){
		this(name,location,null);
	}
	public Member(String name, String location, String hobby){
		this.name = name;
		this.location = location;
		this.hobby = hobby;
	}
}
```
**장점** : 
* 필수 값만 호출하는 코드(`new Member('홍길동',null,null)`)이 많이 발생한다면, `new Member('홍길동')`로 대체할 수 있다.

**단점** : 
* hobby만 작성을 원할 경우, 불가피하게 location은 임의의 값으로 입력하거나 null처리를 해주어야 한다.
* 코드 가독성이 떨어진다.
* 인자가 추가되는 일이 발생하면 코드를 수정하기 어렵다.

#### 자바 빈 패턴
자바빈 패턴은 `setter`메소드를 이용해 생성 코드를 읽기 좋게 만드는 것이다.

```java
Member member = new Member();
member.setName("홍길동");
member.setLocation("서울");
member.setHobby("없음");
```


**장점** : 

* 각 인자의 의미를 파악하기 쉽다.
* 복잡하게 여러 생성자를 만들지 않아도 된다.

**단점** : 

* 객체 일관성이 깨진다.
	* 1회의 호출로 객체 생성이 끝나지 않는다.
* `setter`메소드가 존재하기에 변경 불가능(immutable)한 객체를 만들 수 없다.
	* Thread-safe하기위해 추가적인 작업이 필요하다.


#### 빌더 패턴

```java
public class Member{
	private final String name; //필수 입력 값
	private final String location;
	private final String hobby;
	
	private Member(Builder name){
		this.name = builder.name;
		this.location = builder.location;
		this.hobby = builder.hobby;
	}
	
	public static class Builder { 
		private final name;
		
		private String location = "";
		private String hobby = "";
		
		public Builder(String name){
			this.name = name;
		}
		
		public Builder location(String location){
			this.location = location;
			return this;
		}
		
		public Builder hobby(String hobby){
			this.hobby = hobby;
			return this;
		}
		
		public Member build(){
			return new Member(this);
		}
	}
}
```

스태틱 클래스로 `Builder`를 만들어 주었고, 해당 클래스의 생성자는 필수 입력 값만 입력받아 설정해준다. 그리고 다른 Optional한 인자는 `setter`메소드와 같은 메소드로 설정해주고, 추가로 `build()`메소드로 해당 객체를 반환하는 방식을 사용하였다.  

객체를 생성하는 방법은 아래와 같다.

```java
Member member = new Member
			.Builder("홍길동")
			.location("서울")
			.build();
			
// 다른 방법
Member member2 = new Member.Builder("홍길동");
member2.location("부산");
```

**장점** : 

* 각 인자마다 어떤 의미인지 알기 쉽다.
* `setter`메소드가 없어 변경 불가능한 객체를 만들 수 있다.
* 데이터의 순서에 상관없이 객체 생성을 할 수 있다.
* 불필요한 생성자를 제거 할 수 있다.


#### lombok에서의 @Builder 애노테이션

```java
@Builder
public class Member {
	private String name;
	private String location;
	private String hobby;
}
```

위와같이 `@Builder`애노테이션만 작성해주면 자동으로 빌더패턴으로 작성가능하도록 지원한다. 즉, 아래 방식으로 객체를 선언 하는 것을 허용한다.

```java
Member member = Member.builder().name("홍길동").location("제주").build();
```

`@Builder`애노테이션을 사용하면 아래 7가지가 생성되는 것과 동일하다(거의 빌더패턴과 동일하다) : 

* 빌더라 불리우는, 스태틱 이너 클래스`FooBuilder`
* 타겟 클래스의 인스턴스 변수를 non-static, non-final한 상태로 빌더 안에 담겨진다.
* 아규먼트가 없는 빌더의 빈 생성자
* `setter`메소드와 같은 각 인스턴스 변수의 setter 메소드(여기서 메소드 명은 변수 명과 기본적으로 일치한다)
* 빌더 안에 `build()`메소드를 생성한다.
* 빌더 안에 `toString()`메소드를 생성한다.
* 타겟 클래스에 빌더 인스턴스를 생성하는 `builder()`메소드를 만든다.

그리고 `@Builder`애노테이션에서 변경 가능한 설정 정보는 아래와 같다 : 

* 빌더 클래스의 이름(디폴트 값은 클래스 + 'Builder'이다)
* `build()`메소드의 이름
* `builder()`메소드의 이름
* `toBuilder()`사용여부
* 생성된 element의 접근 레벨(디폴트로는 public)
* 변수 값을 설정하는 메소드의 prefix 값 설정여부 `setterPrefix`

위의 모든 설정정보를 변경하는 애노테이션은 아래와 같이 작성할 수 있다 : 

`@Builder(builderClassName = "HelloWorldBuilder", buildMethodName = "execute", builderMethodName = "helloWorld", toBuilder = true, access = AccessLevel.PRIVATE, setterPrefix = "set")`


***
참고자료

1. [@Builder](https://projectlombok.org/features/Builder)
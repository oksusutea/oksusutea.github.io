---
layout: post
title: final 그리고 static
categories: [Java]
tags: [Java]
description: final과 static, 그리고 final static에 관한 고찰
fullview: false
comments: true
---
잊을만 하면 이따금 다시 궁금해지는 final, static과 final static등등에 관해 다시 한 번 정리해보기로 한다.

***

## final
한 번 선언하면 값을 변경할 수 없을 때 사용한다. 코드를 작성하는 과정에서, 특정한 값이 변경되지 않아야 할 때 이 final 키워드를 사용한다. final은  크게 **변수, 메서드, 클래스** 세가지 경우에 같이 사용할 수 있다. 

### 1. 변수
````Java
final double PI;
PI = 3.14;
PI = 3.15 // 컴파일에러
````    
변수는 할당 후 값 변경이 불가능하다. final 키워드를 사용하는 변수는 되도록 **대문자**로  변수를 명명하여, 해당 변수가 final이라고 인식시켜주는게 좋다.  

### 2. 메소드
````java
class Parent {
	public final String getParent() {
		return "getParent()";
	}
}
class Child extends Parent {
	public String getParent(){
		return "getChild()"; //컴파일 에러
	}
}
````    
메소드에 final 선언시, 해당 메소드가 포함된 클래스를 상속받은 클래스에서는 해당 메소드를 **오버라이딩하여 재정의 할 수 없다**. 내가 만든 클래스를 상속받더라도 오버라이딩하지 않도록 강제성을 부여하고 싶을 경우, 메소드에 final 키워드를 추가하면 된다.  

### 3. 클래스
````java
final class Parent {
	public final String getParent() {
		return "getParent()";
	}
}
class Child extends Parent { //컴파일 에러
	public String getParent(){
		return "getChild()"; 
	}
}
```` 

클래스에 final 선언시, 해당 클래스는 **상속이 불가**하다.  
추가로, *class에는 final과 abstarct가 함께 사용될 수 없는데*, final은 상속이 불가하지만 abstract는 상속을 필요로 하는 배타관계를 띄고 있기 때문이다.

*** 
## static
static으로 선언된 변수는 **메모리 공간에 하나**만 존재하며, **어디서나 접근이 가능**하다. 그렇기 때문에 public으로 선언되어야 한다.  
이 static 변수는 인스턴스가 생성되기 전에 JVM에 의해 클래스로더가 메모리 공간에 적재할 때 초기화가 완료된다. 보통 인스턴스간 **데이터 공유**가 필요할 때 static 변수를 선언한다.

*** 
## static final
static은 클래스 영역에 저장됨을 뜻하고, final은 변할 수 없는 것을 뜻한다. 그러므로 static final은 인스턴스가 아닌 클래스에 존재하는 **변경불가**한 변수임을 뜻한다.

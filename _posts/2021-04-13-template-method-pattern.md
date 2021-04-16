---
layout: post
title: Template Method Pattern
categories: [Design Pattern]
tags: [Design Pattern, OOP]
description: 스프링에서 사용되는 디자인 패턴 (6) 템플릿 메소드 패턴
fullview: false
comments: true
---

## Template Method Pattern
**상위 클래스에 공통 로직을 수행하는 템플릿 메서드와 하위 클래스에 오버라이딩을 강제하는 추상클래스 혹은 선택적으로 오버라이딩 할 수 있는 훅 메소드를 두는 패턴**을 템플릿 메소드 패턴이라고 한다. 다르게 정리해보면, **슈퍼클래스에 기본적인 로직의 흐름을 만들고, 그 기능의 일부를 추상 메소드나 오버라이딩이 가능한 protected 메소드 등으로 만든 뒤 서브클래스에서 이런 메소드를 필요에 맞게 구현해서 사용하도록 하는 방법**을 템플릿 메소드 패턴이라고 한다. 전체적으로 동일하지만 부분적으로 다른 구문으로 구성된 메서드의 코드 중복을 최소화 할 때 유용하다.

템플릿 메소드 패턴의 구조는 아래와 같다. 

<p style="text-align:center;">

<img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FF9zZC%2FbtqwonDtmmJ%2FpiP7Y6uCe1DuJBq5wX6Cd0%2Fimg.png" width="200">

</p>

* AbstractClass : 템플릿 메소드를 정의하는 클래스 
	* 하위 클래스에 공통 알고리즘을 정의하고, 하위 클래스에 구현될 기능을 primitive 메서드 등으로 정의하는 클래스이다.
* ConcreteClass : 물려받은 primitive 메서드 또는 hook 메서드를 구현하는 클래스
	* 상위 클래스에 구현된 템플릿 메서드의 일반적인 알고리즘에서 하위 클레스에 맞게 primitive 메서드등을 오버라이드하는 클래스이다.(primitive 메서드는 추상메서드로 되어있어 필수로 구현해야 하지만, hook 메서드는 선택적으로 오버라이딩 할 수 있다.)



### 활용 예시
아이스 아메리카노와 아이스 라떼를 만드는 클래스

공통적인 뼈대를 만드는 `Coffee` 추상 클래스 : 

```java
package TemplateMethodPattern; 

public abstract class Coffee { 
	final void makeCoffee() { 
		boilWater(); 
		putEspresso();
		putIce(); 
		putExtra(); 
	}
	// SubClass에게 확장/변화가 필요한 코드만 코딩하도록 한다.
	abstract void putExtra(); 
	
	// 공통된 부분은 상위 클래스에서 해결하여 코드 중복을 최소화 시킨다. 
	private void boilWater() { 
		System.out.println("물을 끓인다."); 
	} 
	private void putEspresso() { 
		System.out.println("끓는 물에 에스프레소를 넣는다.");
	} 
	private void putIce() { 
		System.out.println("얼음을 넣는다."); 
	} 
}

```	

`IceAmericano` 클래스 : 

```java
package TemplateMethodPattern;

public class IceAmericano extends Coffee{
	@Override
	void putExtra(){
		System.out.println("시럽을 넣는다.");
	}
}
```


`IceLatte` 클래스 : 

```java
package TemplateMethodPattern;

public class IceLatte extends Coffee{
	@Override
	void putExtra(){
		System.out.println("우유를 넣는다.");
	}
}
```

```java
package TemplateMethodPattern;

public class CoffeeMain {
	public static void main(String[] args){
		IceAmericano americano = new IceAmericano();
		IceLatte latte = new IceLatte();
		
		americano.makeCoffee();
		lattee.makeCoffee();
	}
}
```


위와 같이 `IceAmericano`와 `IceLatte`에서 공통적으로 수행되어야 하는 부분은 상위 클래스에 추상화하여 모델링하고, 각 클래스별로 다르게 구현되어야 할 부분은 추상 메소드를 통해 서브클래스에 구현을 강제하도록 하였다. 

### 정리
템플릿 메소드 패턴은 코드의 중복성을 줄이고, 상속을 통해 유연성을 보장해주는 디자인 패턴이다. 
즉, 추상 메소드와 훅 메소드를 적절히 사용하여 전체적인 알고리즘의 뼈대를 유지하되, 유연하게 기능을 변경할 수 있도록 하고자 할 때 사용하면 유용하다. 


***
참고자료

1. 예제소스 참고 - [템플릿 메서드 패턴(Template Method Pattern)](https://www.crocus.co.kr/1531)
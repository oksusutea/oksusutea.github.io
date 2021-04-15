---
layout: post
title: Strategy Pattern
categories: [Design Pattern]
tags: [Design Pattern, OOP]
description: 스프링에서 사용되는 디자인 패턴 (8) 템플릿 콜백 패턴
fullview: false
comments: true
---

## Template Callback Pattern
템플릿 콜백 패턴은 **전랙패턴**과 매우 유사하다. 기존 전략 패턴에 **익명 내부 클래스를 가미**하여 사용하는 방식이다. 즉, 인터페이스를 상속하는 클래스를 만들지 않고 익명 내부 클래스를 활용하는 방식이다.  
전략 패턴의 컨텍스트(= 전략을 사용하는 객체)를 템플릿이라 부르고, 익명 내부 클래스(=전략)로 만들어지는 오브젝트를 콜백이라 한다. 전략 패턴과 DI의 장점을 익명 내부 클래스 사용 전략과 결합하여 사용한다.  
이 패턴은 주로 **try/catch/finally**블록을 사용하는 코드에 사용된다. 

### 활용 예시

전략 패턴에서 사용하였던 `CarMovable`인터페이스를 그대로 사용하도록 한다.

전략 인터페이스 정의 : 
```java
public interface CarMovable{
	public void action();
}
```


위 전략을 사용하는 `Car`컨텍스트 : 

```java
public class Car {
	private CarMovable carMovable;
	
	public Car(CarMovable carMovable){
		this. carMovable = carMovable;
	}
}
```

클라이언트에서 내부 익명 클래스를 통해 구체 전략 주입:

```java
public class Main {
	public static void main(String[] args){
		Car car1 = new Car(new CarMovable(){
			@Override
			public void action(){
				Systm.out.println("Up");
		});
		car1.move();
		
		Car car2 = new Car(new CarDown());
		car2.move();
		
		Car car3 = new Car(new CarLeft());
		car3.move();
	}
}
```



### 정리
템플릿 콜백 패턴은 전략패턴에서 익명클래스를 가미하여 사용하는 패턴이다. 전략 패턴과 다르게 팩토리 객체 없이 해당 객체를 사용하는 메소드에서 인터페이스의 전략을 선택할 수 있다. 하지만 인터페이스를 사용함에도 불구하고 실제 사용할 클래스를 직접 선언하는 격으로 결합도가 증가한다는 점을 유의하자.

***
참고자료 

1. [[Spring] 템플릿 콜백 패턴](https://withseungryu.tistory.com/89)
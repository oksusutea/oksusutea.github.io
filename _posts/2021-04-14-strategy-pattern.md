---
layout: post
title: Strategy Pattern
categories: [Design Pattern]
tags: [Design Pattern, OOP]
description: 스프링에서 사용되는 디자인 패턴 (7) 전략 패턴
fullview: false
comments: true
---

## Strategy Pattern
**동일 계열의 알고리즘을 정의하고, 각 알고리즘을 캡슐화하며 알고리즘을 상호 교체가 가능하도록 만드는 패턴**이다. 디자인 패턴의 **꽃**이라고 불리기도 하며 가장 자주 쓰이는 패턴이다. 전략패턴의 클래스 다이어그램은 아래와 같다.

<p style="text-align:center;">

<img src="https://mblogthumb-phinf.pstatic.net/20160318_208/2feelus_1458286805546znnWD_PNG/2016-03-18_at_4.47.36_PM.png?type=w800" width="400">

</p>

* Context :
	* 전략을 이용하는 역할을 수행한다(=클라이언트)
	* 필요에 따라 동적으로 구체적인 전략을 바꿀 수 있도록 한다
* Strategy :
	* 인터페이스나 추상클래스로 외부에서 동일한 방식으로 알고리즘을 호출하는 방법을 명시한다
* ConcreateStrategy : 
	* 전략패턴에서 명시한 알고리즘을 실제로 구현한 클래스이다.

### 전략 패턴의 활용 예시

자동차가 움직이는 방향을 컨트롤하는 `CarMovable`인터페이스가 있다.

```java
public interface CarMovable{
	public void action();
}
```

그리고 이 인터페이스를 구현하는 아래 4가지 클래스가 존재한다. 

```java
public class CarUp implements CarMovable{
	@Override
	public void action(){
		System.out.println("Up!");
	}
}

public class CarDown implements CarMovable{
	@Override
	public void action(){
		System.out.println("Down!");
	}
}

public class CarLeft implements CarMovable{
	@Override
	public void action(){
		System.out.println("Left!");
	}
}

public class CarRight implements CarMovable{
	@Override
	public void action(){
		System.out.println("Right!");
	}
}
```

그리고 이러한 차의 움직임 전략을 주입받는 `Car`클래스가 있다.

```java
public class Car {
	private CarMovable carMovable;
	
	public Car(CarMovable carMovable){
		this. carMovable = carMovable;
	}
}
```

`Car`클래스가 외부로부터 전략을 주입받고 구현이 잘 되는지 테스트 해보자.

```java
public class Main {
	public static void main(String[] args){
		Car car1 = new Car(new CarUp());
		car1.move();
		
		Car car2 = new Car(new CarDown());
		car2.move();
		
		Car car3 = new Car(new CarLeft());
		car3.move();
	}
}
```

실행하면 아래와 같이 표기되는 것을 볼 수 있다.

```
Up!
Down!
Left!
```


### JDK에서 사용되는 전략 패턴

#### `java.util.Comparator#compare()`
컬렉션 객체들이 `sort`를 할 때, 객체의 어떤 값을 기준으로 객체간의 비교를 진행할지 정하는 알고리즘이 필요하다.
그래서 객체에 `Comparator`인터페이스를 구현하게 되면, 전략 패턴에 따라 객체간 비교를 가능하게 한다.

#### `javax.servlet,http.HttpServlet#service`
`HttpServletRequest`와 `HttpServletResponse`를 받는 `service()`메소드는 실제 `Servlet`인터페이스의 `service()`메소드 구현체이며 즉 실제 전략이다.

#### `javax.servlet.Filter#doFilter`



### 정리
전략 패턴은 기존의 코드 변경 없이 행위를 자유롭게 바꿀 수 있게 해주는 OCP를 준수한 디자인 패턴이다. 전략패턴을 사용함으로써 상속을 이용하지 않고 자유자재로 구현의 선택을 할 수 있다. 하지만 전략 패턴을 사용하기 위해서는 서로 다른 전략을 이해하고 사용해야 한다는 점을 유의하자.


#### 템플릿 메소드 패턴 vs 전략 패턴

두가지 디자인 패턴 모두 데이터를 은닉화하여 구현될 수 있도록 도와주는 패턴이다.
가장 큰 차이점은, 구체 전략이 선택되는 시점이다.
**템플릿 메소드 패턴**은 런타임 시점에 타입이 선택되며, 추상메소드로 의존성 역전을 구현하였다. 그에 반해 **전략패턴**은 런타임 시점에 합성을 진행하며, 추가 인터페이스로 의존성을 분산시킨다.


***
참고자료 :

1. [[10분 테코톡] 👾베디의 OCP와 전략패턴](https://www.youtube.com/watch?v=90ZDvHl8ROE)
2. [전략 패턴(Strategy Pattern) - 자바 디자인 패턴과 JDK 예제](https://m.blog.naver.com/2feelus/220658663151)
3. [What is the difference between the template method and the strategy patterns?
](https://stackoverflow.com/questions/669271/what-is-the-difference-between-the-template-method-and-the-strategy-patterns)





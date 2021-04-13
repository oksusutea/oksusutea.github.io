---
layout: post
title: Factory Method Pattern
categories: [Design Pattern]
tags: [Design Pattern, OOP]
description: 스프링에서 사용되는 디자인 패턴 (5) 팩토리 메소드 패턴
fullview: false
comments: true
---

## Factory Method Pattern
**객체를 만들어내는 부분을 서브 클래스에 위임하는 패턴**이다. 즉, `new`키워드를 호출하는 부분을 서브 클래스에 위임하는 패턴이다.   
팩토리는 공장을 의미한다. 공장은 물건을 생산하는데, 객체 지향에서 팩토리는 객체를 생성한다. 결국 팩토리 메서드는 객체를 생성하여 반환하는 메소드를 말한다. 

팩토리 메소드를 UML로 표현하자면 아래와 같다. 

<p style="text-align:center;">
<img src="https://gmlwjd9405.github.io/images/design-pattern-factory-method/factory-method-pattern.png" width="400">

</p>

* Product : 팩토리 메서드로 생성될 객체의 공통 인터페이스
* ConcreteProduct : 구체적으로 객체가 생성되는 클래스
* Creator : 팩토리 메서드를 갖는 클래스, 팩토리 메서드를 호출하여 *product* 생성
* ConcreteCreator : 팩토리 메서드를 구현하는 클래스로 ConcreteProduct 객체를 생성

팩토리 메서드 패턴의 적용 방법 : 

* 객체 생성을 전담하는 별도의 **Factory 클래스 사용**
* **상속 이용** : 하위 클래스에서 적합한 클래스의 객체를 생성

### 활용 예시

`Robot`추상 클래스와 그를 구현한 `SuperRobot`, `PowerRobot` 클래스

```java
public abstract class Robot {
	public abstract String getName();
}
```

```java
public class SuperRobot extends Robot {
	@Override
	public String getName(){
		return "SuperRobot";
	}
}
```

```java
public class PowerRobot extends Robot {
	@Override
	public String getName(){
		return "PowerRobot";
	}
}
```

기본 팩토리 클래스와 그를 상속받아 실제로 구현한 `RobotFactory`, `SuperRobotFactory` 클래스

```java
package pattern.factory;

public abstract class RobotFactory {
	abstract Robot createRobot(String name);
}
```

```java
package pattern.factory;

public class SuperRobotFactory extends RobotFactory {
	@Override
	Robot createRobot(String name) {
		switch( name ){
			case "super": return new SuperRobot();
			case "power": return new PowerRobot();
		}
		return null;
	}
}
```

```java
package pattern.factory;

public class ModifiedSuperRobotFactory extends RobotFactory {
	@Override
	Robot createRobot(String name) {
		try {
			Class<?> cls = Class.forName(name);
			Object obj = cls.newInstance();
			return (Robot)obj;
		} catch (Exception e) {
			return null;
		}
	}
}
```

메인 실행 코드 : 

```java
package pattern.factory;

public class FactoryMain {
	public static void main(String[] args) {

		RobotFactory rf = new SuperRobotFactory();
		Robot r = rf.createRobot("super");
		Robot r2 = rf.createRobot("power");

		System.out.println(r.getName());
		System.out.println(r2.getName());

		RobotFactory mrf = new ModifiedSuperRobotFactory();
		Robot r3 =  mrf.createRobot("pattern.factory.SuperRobot");
		Robot r4 =  mrf.createRobot("pattern.factory.PowerRobot");

		System.out.println(r3.getName());
		System.out.println(r4.getName());
	}
}
```

위 코드에서 보면 알 수 있듯이, 직접적으로 `Robot`객체를 생성(`new`)하는 부분은 없다. 객체 생성을 팩토리 클래스에게 위임하였기 때문이다. 그래서 메인 클래스에서는 구체적으로 어떤 객체가 생성되었는지 신경쓰지 않고 단지 반환된 객체만 가져와 사용할 뿐이다. 새로운 로봇이 추가되어도 팩토리 클래스만 변경하고, 실제 로봇 구현 클래스만 작성하면 된다.



### 정리
* 팩토리 메소드 패턴은 개방-폐쇄 원칙과 의존성 역전 원칙에 의거하여 기존 코드를 수정하지 않고, 클래스를 추가하는 방식으로 새로운 객체 생성을 처리할 수 있다. 이를 통해 클래스간 결합도를 낮출 수 있다.
* 새로운 product가 생길때마다 class를 추가하기 때문에, 점점 클래스가 많아진다.


***
참고 자료 : 

1. [팩토리 메소드 패턴(Factory Method Pattern)](https://jdm.kr/blog/180)


---
layout: post
title: Singleton Pattern
categories: [Design Pattern]
tags: [Design Pattern, OOP]
description: 스프링에서 사용되는 디자인 패턴 (4) 싱글톤 패턴 
fullview: false
comments: true
---

## Singleton Pattern
전체 어플리케이션을 통틀어 단 하나의 인스턴스만 생성되도록 하는 디자인 패턴이다. 인스턴스 생성은 외부에서 이루어지지 않고, 접근 제어자를 통해 싱글톤 패턴으로 정의된 클래스의 내부에서만 생성되도록 제한한다.


<p style="text-align:center;">

<img src="https://img-blog.csdn.net/20170802105613589" width="450px">

</p>

싱글톤 패턴은 세가지 특징이 있다. 

* 단 하나의 인스턴스만 존재해야 한다.
* 클래스의 내부에서만 인스턴스를 생성해야 한다.
* 외부에 해당 클래스의 인스턴스를 반환할 수 있도록 해야한다. 


## 싱글톤 패턴을 구현하는 방식

### Eager Initialization (Early Loading)
싱글톤의 가장 기본적인 초기화 방식이다. 클래스 내 전역변수로 instacne 변수를 생성하고, static을 사용해 클래스 로딩 시점에 생성되도록 한다. 동시에 private 접근제어자를 통해 외부에서 Singleton.instance로 접근 불가하도록 작성되었다. 외부에서 Singleton 객체를 생성하여도, 생성자에는 아무것도 없기 때문에 새로운 인스턴스는 생성되지 않는다. 

```java
public class Singleton{
	private static Singleton instance = new Singleton();
	private Singleton(){}
	
	public static Singleton getInstanc() {
		return instance;
	}
}
```
장점 : 

* JVM의 클래스로더에 의해 클래스 최초 로딩시 객체가 생성되어 Thread-safe하다.
* 락을 거는 방식이 아니기 때문에 속도가 빠르다.

단점 :

* 싱글톤 객체 사용유무와 관계없이 클래스가 로딩되는 시점에 싱글톤 객체가 항상 생성된다. 메모리 낭비를 유발 할 수 있다.

<br/>

### Lazy Initialization
앞서 작성한 Early Loading의 단점을 보완한 방식이다. 클래스를 호출할 때에만 객체를 생성하는 방식을 취한다. 

```java
public class Singleton{
	private static Singleton instance;
	private Singleton(){}
	
	public static Singleton getInstance(){
		if(Objects.isNull(instance)){
			instance = new Singleton();
		} 
		return this.instance;
	}
}
```

장점 : 

* 싱글톤 객체가 필요할 때 인스턴스화하기에 메모리 누수를 방지할 수 있다.

단점 : 

* Thread-safe하지 않다. 만일 여러 쓰레드에서 `getInstance()`를 호출할 경우 인스턴스가 여러개 생성될 수 있다.

<br/>

### Thread-safe한 Lazy Initialization
기존 Lazy Initialization에서 Thread-safe하도록 보완한 방법이다. 기존 `getInstance()`메소드에 `synchronized`를 걸어, 해당 메소드는 독립적으로 실행될 수 있도록 한다.

```java
public class Singleton{
	private static Singleton instance;
	private Singleton(){}
	
	public static synchronized Singleton getInstance(){
		if(Objects.isNull(instance)){
			instance = new Singleton();
		} 
		return this.instance;
	}
}
```

장점 : 

* 인스턴스를 필요할 때에만 초기화 하며, thread-safe하다.

단점 : 

* 처음 `instance`생성시에만 동기화 작업이 필요한데, 이 코드는 인스턴스 조회시마다 락을 걸어 성능이 매우 낮다.

<br/>

### Double Checked Locking
앞서 진행한 방식은 `getInstance()`를 콜할때마다 락이 걸린다고 하였다. 하지만, 객체가 생성되는 시점에만 락을 걸면 되므로, 더블체킹하여 성능을 보완하도록 하였다.

```java
public class Singleton{
	private volatile static Singleton instance;
	private Singleton(){}
	
	public static synchronized Singleton getInstance(){
		if(Objects.isNull(instance)){
			synchronized(Singleton.class) {
				if(Objects.isNull(instance)){
					instance = new Singleton();
				}
			}
		} 
		return this.instance;
	}
}
```
위 방식에서 `instance`의 `null` 체크를 두 번 해주었다. `synchronized`전에 한 번하고, 후에 한 번 체크를 하는데, 그 전에 진행하는 `null`체크는 `instance` 존재시 lock을 걸지 않고 빠르게 인스턴스 값을 반환하기 위해서다.  두번째 null체크를 걸어주지 않을 경우, 순차적으로 인스턴스화 하는 과정을 겪기 때문에 추가로 체크를 해주어야 한다.

장점 : 

* Lazy Initialization을 지원하며, thread-safe하다.

단점 : 

* `volatile` 키워드를 붙여야 진정한 `thread-safe`를 실현할 수 있다. JDK 1.5부터 사용할 수 있다.

<br/>

### Bill Pugh Solution
클래스 내부에 클래스를 만드는 방식이다. `Singleton` 클래스 안에 `LazyHolder`라는 static 클래스를 만들었다. `getInstance()`가 호출 될 때, `LazyHolder`가 처음으로 로드되어 `Singleton`객체를 생성하고, 다시 한 번 `getInstance()`가 호출되었을 경우 `final`이기 때문에 기존에 생성한 값으로 반환된다.

```java
public class Singleton{
	private Singleton(){}
	
	public static Singleton getInstance(){
		return LazyHolder.singleton;
	}
	
	public static class LazyHolder{
		private static final Singleton singleton = new Singleton();
	}
}
```


장점 : 

* Lazy Initialization을 지원한다. 클래스가 로드될 때 인스턴스가 생성된다.
* Thread-safe하다. 

<br/>

### Enum Singleton
이펙티브 자바 책에서 추천하는 구현 방식이다. 하지만 실제로 자주 쓰이는 방식은 아니라고 한다.  

```java
public enum Singleton{
	INSTANCE;
	
	public void someMethond(String args){	//메소드 추가 필요시
	}
}
```

장점 : 

* 구현이 쉽다(로지컬한 내용이 전혀 없다)
* `Enum`클래스 자체가 프로그램 구동시 초기에 생성되어 Thread-safe하다.
* 직렬화/역직렬화에 대한 처리가 필요없다.

단점 : 

* Early Loading이다.

### 주의사항
1. 클래스 로더를 2개 이상 사용할 경우, 인스턴스가 2개 이상 생성 될 수 있다. 완벽하게 싱글톤으로 생성하기 위해서는 **클래스 로더를 지정**해야 한다.    
1. 직렬화/역직렬화 구현을 위해서는 모든 필드에 `transient`로 만들어주어 무상태성으로 바꾸어주어야 하고, 구현하고자 하는 클래스에 `readResolve()`메소드를 추가해주어야 한다. 

```java
import java.io.*;

public class ClientSerializedSingleton {
    public static void main(String[] args) 
    throws FileNotFoundException, IOException, ClassNotFoundException {
        SerializedEagerSingleton serializedInstance = SerializedEagerSingleton.getInstance();
        ObjectOutput out = new ObjectOutputStream(new FileOutputStream("output.txt"));
        out.writeObject(serializedInstance);
        out.close();
        
        // 역직렬화
        ObjectInput in = new ObjectInputStream(new FileInputStream("output.txt"));
        SerializedEagerSingleton deSerializedInstance = (SerializedEagerSingleton) in.readObject();
        in.close();
        
        System.out.println(serializedInstance.hashCode());
        System.out.println(deSerializedInstance.hashCode());
    }
}
```
위 코드에서 두개의 인스턴스는 다른 해시코드 값을 출력하는데, 이는 역직렬화가 진행될때 `readObject()`를 호출하며 새로운 인스턴스를 생성하기 때문이다.  아래와 같은 방법을 사용하여 싱글톤 패턴을 보장하자.  

```java
import java.io.Serializable;

public class SerializedEagerSingleton implements Serializable {

    private static final long serialVersionUID = 3368531508195651477L;

    private static SerializedEagerSingleton instance = new SerializedEagerSingleton();

    private SerializedEagerSingleton() {
    }

    public static SerializedEagerSingleton getInstance() {
        return instance;
    }

    // 추가
    private Object readResolve() {
        return getInstance();
    }
}
```

### 자바와 스프링의 싱글톤 차이점
싱글톤 객체의 생명주기가 다르다. 또한 자바에서의 범위는 클래스 로더가 기준이지만, 스프링에서는 어플리케이션 컨텍스트가 기준이다.  
클래스 로더 기준이라는 것은 톰캣이 WAR파일을 만들 때, WAR 파일 하나당 클래스 로더 하나씩 배치가 되고, 다른 WAR파일은 참조가 불가능 하다는 것을 뜻한다.  
반면, 어플리케이션 컨텍스트 기준은 `web.xml`에서 `root context`하나와 `servlet context`여러개를 등록할 수 있는데, 이 각각의 context가 싱글톤의 범위가 된다.


### 정리
싱글톤 패턴을 이용하여 객체를 단일 생성하고 자원을 절약할 수 있다. 또한, 특정 객체를 공유해야 하는 상황이 왔을 때 효율적으로 이용할 수 있는 장점이 있다. 하지만, 동시성 문제와 직렬화/역직렬화를 고려하여 설계해야 하기 때문에 상황에 맞게 구현 방식을 선택하는 것이 좋다.

***
참고자료

1. [设计模式（一）：单例模式（Singleton Pattern）](https://blog.csdn.net/tjiyu/article/details/76572617)
2. [싱글턴 패턴(Singleton Pattern)
](https://webdevtechblog.com/%EC%8B%B1%EA%B8%80%ED%84%B4-%ED%8C%A8%ED%84%B4-singleton-pattern-db75ed29c36)


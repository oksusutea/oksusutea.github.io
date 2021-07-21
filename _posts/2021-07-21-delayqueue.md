---
layout: post
title: DelayQueue
wcategories: [Java]
tags: [Java, Spring Framework]
description: Queue의 서브 클래스
fullview: false
comments: true
---


### DelayQueue의 특징
* DelayQueue는 _java.util.concurrent_ 패키지에 있는 자료구조이다.
* producer-consumer 패턴의 프로그램을 작성할 때 주로 쓰인다.
* blockingQueue를 implement하였으며, 순서를 보장하기 위해 priorityQueue의 기능을 같이 구현하였다(implement는 하지 않았다)

### DelayQueue
* 내부 구성 요소로 Delay 인터페이스의 구현체만 들어갈 수 있다.
	* 모든 요소는 `Delayed`의 `getDelay()` 오버라이드 메서드를 구현해야 한다.

### 사용 예시

#### `Delayed` 인터페이스를 구현한 element class 생성
```java
public class DelayObject implements Delayed { 
	private String data;
	private long startTime;
	
	public DelayObject(String data, long delayInMilliseconds) {
		this.data = data;
		this.startTime = System.currentTimeMillis() + delayInMilliseconds;
	}
	@Override  
	public long getDelay(TimeUnit unit) { // 반환 값이 클수록 우선순위 높음
		long diff = startTime - System.currentTimeMillis(); 
		return unit.convert(diff, TimeUnit.MILLISECONDS); 
	}
	@Override  
	public int compareTo(Delayed o) { 
		return Ints.saturatedCast( this.startTime - ((DelayObject)o).startTime); 
	}
}
```

* 보통 element 내 해당 element가 등록된 시간을 필드(`startTime`)로 지정한다.
* `getDelayed()`를 통해 _현재시간 - 등록시간_계산을 함으로서 가장 일찍 등록된 element가 가장 높은 우선순위를 가질 수 있도록 한다.
* Consumer가 queue로부터 element를 꺼내려 할 때, `DelayQueue`는 `getDelay()` 를 실행해 어떤 element를 반환할지 결정한다.
* `getDelay()`의 리턴 값이 0과 같거나 0보다 작으면 queue에서 element를 찾을 수 없다는 의미이다.
* `DelayQueue`를 만료시간 기준으로 정렬해야 하기 때문에 `compareTo` 메소드도 함께 override 해준다.
* `take,put` 메서드 사용시 _꺼낼 요소가 없거나, 이미 너무 많은 요소가 들어가있어 더이상 넣을 수 없다면_ 해당 메서드를 호출한 스레드는 넣거나 뺄 수 있을 때까지 대기한다.

#### Queue 관련 메소드 차이
* public void put(E e) : delayQueue에 요소를 insert한다. unbounded하기 때문에 영원히 block되지 않는다.
* public void offer(E e, long timeout, TimeUnit unit) : delayQueue에 요소를 insert한다. 위와 같이 Queue를 block하지 않는다.
	* timeout /unit : block하지 않기 때문에 쓰이지 않는 파라미터이다.
* public E take() : Queue의 head element를 반환하고 제거한다. 필요시 expired delay가 생길때 까지 대기한다.
* public E poll() : Queue의 head element를 반환하고 제거한다. 반환할 값이 없으면 null을 리턴한다.

***
참고자료 : 
[Guide to Delay Queue](https://www.baeldung.com/java-delay-queue)


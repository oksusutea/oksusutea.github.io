---
layout: post
title: 자바의 동기화 방법
categories: [Java]
tags: [Java, multi-thread programming]
description: Atomic, volatile, synchroznied의 차이
fullview: false
comments: true
---

Java에서 멀티 스레드 프로그래밍 방식에는 여러 스레드가 동시에 하나의 자원을 접근해 기존 의도와 다르게 데이터가 output 될 수 있다. 여기서 고려해야 할 것이 많은데, 그 원인은 **경쟁 상태(race condition)**과 **변수의 가시성(visibility)**에서 온다. 

#### 경쟁상태
여러 스레드가 같은 시점에 변수를 읽는 상태

#### 변수의 가시성
변수들이 사용될 수 있는 영역의 범위로써 변수의 값이 CPU의 캐시 메모리에 저장되었는지, 메인 메모리에 저장되어있는지 알 수 없다는 것에서부터 온다. 동일한 변수를 동시에 접근했을 때 어떤 스레드는 캐시 메모리에 접근할 수 있고, 어떤 스레드는 메인메모리에 접근할 수도 있기 때문이다. 

### Atomic

* 멀티쓰레드 환경에서 동기화 문제를 별도의 `synchronized`없이 해결하기 위해 고안된 방법이다. 
* Non-blocking하며 동시성 문제를 해결할 수 있다.
* CAS(Compare And Swap) 알고리즘을 통해 원자성을 보장한다.
	* 자신이 읽었던 변수의 값을 기억하고 있다가, 변경을 완료하기 전 읽었던 변수의 값이 그대로인지 확인하고(메인메모리에 변동이 없는지), 아니라면 실행을 무산한다.

#### CAS 알고리즘이란?
멀티 쓰레드 환경에서 각 쓰레드는 CPU Cache영역에서 값을 가져온다. 이 때, 메인 메모리에 저장된 값과 CPU Cache에 저장된 값이 다른 경우가 있다(가시성문제) 이 때 사용되는 것이 CAS 알고리즘이다. **현재 쓰레드에 저장된 값과 메인 메모리의 값을 비교하여, 일치하는 경우 새로운 값으로 교체하고, 일치하지 않는다면 실패하고 재시도를 한다.**

여기서 현재 쓰레드에 저장된 값은 CPU 캐시의 값이고, 메인 메모리의 값과 비교해 일치 하는 경우,
연산으로 작업된 최종 결과 값을 메인메모리에 교체한다는 것을 의미한다.


<br/>
<br/>

### volatile

#### volatile이란?
*  voatile 키워드는 자바 변수를  메인 메모리에 저장하겠다라는 것을 명시하는 것이다.
* 매번 변수의 값을 Read 할 때, 각 쓰레드는 CPU Cache에 저장된 값이 아니라 메인 메모리에서 읽어 가져온다.
* 매번 변수의 값을 Write 할 때, 메인 메모리에 해당 값을 작성한다. 
* 자바 5에서는 volatile 변수만 메인 메모리에 저장되는 것이 아니라, volatile 변수 점근 전 수정한 모든 변수들이 함께 메인 메모리에 저장된다. 그리고 쓰레드가 volatile 변수를 메인 메모리에서 읽을 때, volatile 변수를 수정하며 메인메모리에 저장된 다른 모든 변수들도 메인메모리로부터 읽어온다. 

<br/> 

#### volatile을 사용하는 이유

<p style="center">

<img src="https://nesoy.github.io/assets/posts/20180609/1.png">

</p>

* `volatile`변수를 사용하고 있지 않은 변수는 쓰레드 수행시 성능 향상을 위해 Main Memory에서 읽은 값을 CPU Cache에 저장한다.
* 멀티 쓰레드 프로그래밍 환경에서 각각의 쓰레드가 변수 값을 불러올 때 CPU Cache에 담겨진 값이 다를 수 있어 **변수 값 불일치** 문제가 생긴다.

#### 특징

* 하나의 쓰레드가 아닌 여러 쓰레드가 변수를 write하는 상황에서는 적합하지 않는다.(쓰레드에서 업데이트 한 값이 메인메모리까지 반영이 안될 수 있어 동시성이 보장되지 않는다)
* CPU Cache보다 Main Memory에 접근하는데 비용이 더 크기 때문에 변수 값 일치를 보장해야 하는 경우에만 사용하는 것이 좋다.
* 메인메모리조차 최신의 값이 쓰여지지 않을 수 있기 때문에, 아직 다른 쓰레드에서 사용하는 공유 데이터의 원자성을 보장하지 않는다.

<br/>
<br/>

### synchronized

* 멀티쓰레드 환경에서 동시성 제어를 위해 공유 객체를 동기화 하는 키워드이다. 
* 메소드나, 메소드 안의 특정 블록에만 해당 키워드로 scope를 정해놓고 멀티 쓰레드가 동시에 접근하지 못하도록 blocking하는 방식이다. 
* `synchronized` 블락 진입 전 후 메인 메모리와 CPU 캐시 메모리의 값을 동기화 하여 원자성이 보장된다.

#### synchronized와 static synchronized의 차이점
`synchronized`는 `synchronized(this){} `블럭으로 쓰는 것과 동일하다.   
**즉, 현재 instance에 대해서만 동기화**가 일어나는 것이다. 이 경우 해당 인스턴스에서는 동기화가 일어나지만, 2개 이상의 인스턴스가 있으면 각각의 인스턴스에 대해서 동기화가 발생하므로 각각 method를 호출 할 수 있다. 

<br/>

반면, static synchronized의 경우 **해당 클래스의 클래스 객체에 대해 동기화**가 일어난다. `synchronized(TargetClass.class)`와 동일하다고 볼 수 있다. 해당 클래스를 접근하는 모든 경우는 다 동기화가 발생한다.

***
참고 자료 : 
1. [Java volatile이란?](https://nesoy.github.io/articles/2018-06/Java-volatile)
2. [CAS Compare And Swap](https://www.jianshu.com/p/dcab74dd7ea4)
2. [synchronized와 static synchronized의 차이점](https://ohgyun.com/5)


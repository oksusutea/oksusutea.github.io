---
layout: post
title: 자바의 동시성 프로그래밍 방식
categories: [CS]
tags: [Java]
description: 동시성 프로그래밍시 발생할 수 있는 이슈와, 이를 해결하는 방식에 대해 알아본다.
fullview: false
comments: true
---

앞서 하드웨어적 스레드와 소프트웨어적 스레드의 차이에 대해 알아보았고, 소프트웨어 스레드를 논의하다가 자바의 동시성 프로그래밍시 발생할 수 있는 이슈가 있어 추가적으로 발견한 문제점과 해결 방식을 찾아보았다.

## 자바의 Thread safe

멀티스레드 프로그래밍은  

* 시스템 자원의 사용 (스레드간 공유하는 데이터가 많아 메모리 절약 가능)
* 응답시간 (스레드간 통신은 프로세스간 통신에 비해 간단함)
* Context Switching 횟수 (Cache 초기화, 페이징테이블등 초기화가 필요하지 않아 더 간단)  
를 줄일 수 있다는 장점이 있다. 하지만 데이터의 충돌문제가 발생할 수 있다.  
여러 테스크가 동시에 처리되도록 구현하는 것을 **동시성 프로그래밍**이라고 하고, 데이터 충돌과 같은 동시성 프로그래밍으로 인해 발생되는 이슈를 피하는 방법을 **동시성 보장**이라고 한다.  
이번 포스팅에서는 자바의 동시성 보장 방식중 가장 기본적인 **synchronized, volatile, atomic** 세가지 키워드에 대해 정리해본다.  

## 가시성 문제와 Volatile

<p style="center">
<img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbAz14z%2FbtqRlHC9FZB%2F6mk8kLeYTxo2GyRWK20f30%2Fimg.png">
</p>

위의 그림과 같이 2CPU, 2코어 2스레드가 있다고 가정해보자. `CPU`는 어떤 작업을 처리하기 위해 데이터가 필요할 때, `RAM`의 일부분을 고속 저장 장치인 `CPU Cache Memory`에 저장하고 읽어들인다. 적절한 시점에 `CPU Cache Memory`에서 `RAM`으로 데이터를 쓰고, 반대로 데이터를 불러온다. 이 말은, **각 스레드의 `CPU Cache Memory`와 `Ram`데이터가 불일치 할 수 있다는 것을 의미**한다.  
이 문제를 해결하기 위한 방법은 간단하다.  
**가시성이 보장되어야 하는 변수를 `volatile`로 선언하여 `Cache Memory`에서 읽는 것이 아니라, `RAM`에서만 읽도록 보장하는 것**이다.

<p style="center">
<img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbLWwSW%2FbtqRptqp9RK%2FrL4yGIT5y5iKQvGpk3zXTK%2Fimg.png">
</p>

volatile은 이런 방식으로 동시성 프로그래밍에서 **가시성**을 보장할 수 있도록 한다.

## 동시성 보장의 문제
가시성 보장 외에도 한가지 문제가 더 있다. 바로 **공유자원 접근**이다.

만일 아래와 같은 Thread 두개가 동시에 실행한다고 가정해보자.  

* Thread 1
	* 고객의 나이를 읽는다
	* 읽어온 나이를 기준으로 비용을 계산한다.
	* 비용을 반환한다.
* Thread 2
	* 현재 년도를 지속적으로 읽는다.
	* 해가 바뀌면 고객의 나이를 계산한다.
	* 바뀐 고객의 나이를 저장한다.


만일 대상자의 나이가 27-> 28세로 바뀌는 시점이라면, `Thread1`의 고객의 나이를 읽는 작업이 먼저 시작되고 그 후에 `Thread 2`의 바뀐 고객 나이를 저장하는 로직을 수행한다면, 나이가 잘못 계산되는 문제가 발생한다.

volatile만 쓰면 해결될 줄 알았는데, 가시성 보장은 동시성 보장을 의미하지 않는다. 보통 가시성 보장의 경우, 하나의 스레드는 *쓰기*를 하고, 나머지 스레드는*읽기*(해당 변수와 연관된 연산작업도 안하는 상태)만 하는 상황일 때만 동시성 보장이 가능하다. 

### Blocking과 Synchronized

이렇게 쓰기가 어느정도 사용되는 *공유 데이터*를 사용할 때는 여러 스레드가 동시에 사용할 수 없도록 하면 데이터 충돌을 피할 수 있다.  
`synchronized` 키워드는 메소드 혹은 블록에 붙여, 해당 자원을 사용할 때 다른 스레드가 동시에 사용할 수 없도록 Lock을 걸고, 사용을 마친 후 Lock을 풀 수 있도록 하는 키워드이다.  
`synchronized 메소드`를 사용하면, 해당 메소드를 호출하는 인스턴스 객체 기준으로 동기화가 이루어지고, `synchronized 블록`을 사용하면 블록에 전달받은 객체를 기준으로 동기화가 이루어진다.  
편리하게 키워드 하나만 걸어 사용할 수 있지만, 다른 스레드를 완전히 차단시킨다는 단점이 존재한다.  


### 원자성 보장과 Atomic
앞서 `synchronized`를 개선한 방법이 있다.  
변수를 조회하는 스레드에서, 해당 변수 조회 직전 현재 스레드에서 사용되는 값이 현재 RAM의 값가 같은지 비교하고, 불일치하다면 업데이트한 값을 가져와 계산하는 **CAS 알고리즘**을 이용해 원자성을 보장한다.

이 방법을 이용하면 병렬성을 해치지 않으면서(다른 스레드도 실행중이니까), 동시성을 보장하기 떄문에 더 좋은 성능을 가져올 수 있다.  
대표적으로는 `ConcurrentHashMap`이 이 방법을 적용하였다. `HashMap`을 동시성을 보장하며 사용하고 싶을 때, `HashTable`은 `synchronized`으로 blocking을 사용하지만, `ConcurrentHashMap`은 blocking 없이 동시성을 보장할 수 있어 유용하다.


***
참고 자료 : 
1. [자바의 동시성 #2 - 동시성 프로그래밍에서 발생할 수 있는 문제점과 volatile, synchronized 키워드](https://badcandy.github.io/2019/01/14/concurrency-02/)  
2. [자바 깊이 알기 / 자바의 동기화 방식](https://ecsimsw.tistory.com/entry/%EC%9E%90%EB%B0%94%EC%9D%98-%EB%8F%99%EA%B8%B0%ED%99%94-%EB%B0%A9%EC%8B%9D-%EB%A9%94%EB%AA%A8%EB%A6%AC-%EA%B0%80%EC%8B%9C%EC%84%B1%EC%9D%B4%EB%9E%80-synchronized-volatile-atomic)  


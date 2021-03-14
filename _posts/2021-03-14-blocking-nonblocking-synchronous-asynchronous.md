---
layout: post
title: 동기/비동기, 블로킹/넌블로킹
categories: [CS]
tags: [OS]
description: 블로킹&논블로킹, 그리고 동기&비동기 차이에 대해 알아본다.
fullview: false
comments: true
---

블로킹/넌블로킹, 그리고 동기/비동기에 관해 여러 차례 자료를 찾아가며 익히려고 했지만, 말로 차이점을 풀어나가기가 어려워 정리해보았다.

## Blocking / Non-Blocking
다른 작업을 수행하는 주체를 어떻게 상대하는지에 따라 다르다.  
자신의 작업을 진행하다, 다른 작업 주체가 하는 작업의 시작부터 끝까지 기다렸다가 다시 시작하면, 이 경우는 블로킹이고, 다른 주체의 작업과 관계없이 자신의 작업을 계속 한다면 이는 논블로킹이라고 할 수 있다.  
즉, 블로킹/넌블로킹은 **호출되는 함수가 바로 리턴하느냐 마느냐**를 관심사다.

<p style="center">
<img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FKAhB9%2FbtqDZYxAX5n%2Fm287hwVpNKPgnnjaItsU91%2Fimg.png">
</p>


## Synchronous/Asynchronous
동기 작업은, 수행하는 두 개 이상의 주체가 서로 동시에 수행하거나, 동시에 끝나거나, 끝나는 동시에 시작할 때를 말한다.  
그에 반해 비동기 작업은 두 주체가 서로 시작, 종료시간과는 관계없이 별도의 수행 시작/종료시간을 가지고 있을 때를 뜻한다.  
서로 다른 주체라 하는 작업이 자신의 작업 시작, 종료시간과는 관계가 없을 때 비동기라고 칭한다.  
즉, 동기/비동기는 **호출되는 함수의 작업 완료 여부를 신경쓰냐 마느냐**가 관심사다.

<p style="center">
<img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fdxv5bm%2FbtqDZX6x7aF%2FqoGwd60NkaxZFPsRSFrpK1%2Fimg.png">
</p>

## Nonblocking-Sync
이 케이스는, 호출되는 함수가 바로 리턴하고, 호출하는 함수는 작업 완료 여부를 신경쓴다. 이 경우에, 호출하는 함수는 계속 호출되는 함수에게 완료여부를 물어보게 된다.

<p style="center">
<img src="http://i.imgur.com/a8xZ9No.png">
</p>

## Blocking-Async
이 경우, 앞선 상황과는 정반대의 상태로 호출되는 함수가 바로 리턴하지 않고, 호출하는 함수는 작업완료여부를 신경쓰지 않는다. 이 경우, 어차피 Blocking이 되는 상황이기에 Blocking-Sync 방식과 성능적으로는 거의 차이가 없을 것 같다.

<p style="center">
<img src="http://i.imgur.com/zKF0CgK.png">
</p>


*** 
참고 자료 및 이미지 출처 :

1. [Blocking-NonBlocking-Synchronous-Asynchronous](https://homoefficio.github.io/2017/02/19/Blocking-NonBlocking-Synchronous-Asynchronous/)  
2. [동기/비동기와 블로킹/논블로킹](https://deveric.tistory.com/99)
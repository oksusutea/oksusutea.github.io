---
layout: post
title: Context Switching
categories: [CS]
tags: [OS]
description: 컨텍스트 스위칭
fullview: false
comments: true
---
스레드와 프로세스의 차이에 관해 공부하였을때, 언뜻 context switching이라는 용어가 나왔다. 간략하게는  프로세스 1이 실행중이다가 잠시 멈추고 프로세스 2가 실행될 때 기존 프로세스의 상태/값을 저장하고 새로운 프로세스로 실행하도록 하는 작업이라고 알고 있었는데, 조금 더 상세하게 공부하기 위헤 포스팅에 적어본다.

### Context Switching이란 ? 
멀티 프로세스 환경에서 CPU가 **하나의 task를 실행하고 있는 상태**에서 인터럽트 요청에 의해 **다음 우선 순위의 task가 실행**되어야 할 때, **기존** 프로세스의 상태 또는 레지스터 값(Context)를 저장하고 CPU가 다음 프로세스를 수행하도록 **새로운** 프로세스의 상태 또는 레지스터 값(Context)를 **교체**하는 작업을 Context Switching이라고 한다.

프로세스와 스레드를 처리하는 Conetext Switching은 약간 차이가 있는데, 프로세스는 OS에 의해 스케줄링되는 Process Control Block에서 관련 정보를 저장하고, 스레드는 프로세스 내 TCB(Task Control Block)에서 관리된다.  

> 우선 순위는 해당 OS의 스케줄러가 우선 순위 알고리즘에 의해 정해지고 수행하게 되어있다.  
> 라운드로빈 스케줄링(Round Robin Scheduling)은 시분할 시스템을 위해 설계된 선점형 스케줄링의 하나다.  
> 쉽게 설명하면 시간단위(TIme Quantum)을 CPU에 할당하는 방식이라고 볼 수 있다. 
> 꽤 효율적인 스케줄링 알고리즘이지만, 시간 단위를 작게 설정하면 CPU가 조금 일하고 Context Switching을 반복하여 효율이 떨어진다.

#### 실행 순서
1. 프로세스 A가 실행된다.
2. 프로세스 B가 실행되려 한다!(호출)
3. CPU를 양보하여 Context Switching이 발생한다.
3. 기존 프로세스 A의 실행과정 관련 정보를 저장하기 위해 PCB에 PC, SP에 값을 저장한다.
4. 현재 프로세스의 PC, SP값을 저장한다.


이러한 작업은 기계어 명령어 단위로 이루어져 있으며, CPU개수와 스레드 개수가 같으면 컨텍스트 스위칭이 발생하지 않는다.

### Context Switching이 필요한 이유
* 컨텍스트 스위칭이 없을 경우, 프로세스 하나를 다 수행하고 종료되기 전까지는 다른 프로세스를 수행할 수 없다.
* 반응속도가 매우 느려지며, 사용 속도를 저하합니다.
* 여러 프로세스를 수행하며 빠른 반응속도로 응답하기 위해서는 컨텍스트 스위칭이 필요하다.

### Context란?(PCB)
CPU가 다루는 Task(process, thread)의 정보를 말한다. Context는 프로세스의 PCB(Process Control Block)에 저장된다. 그래서 Context Switching을 할 때에, PCB의 정보를 읽어 CPU가 이전에 진행했던 부분부터 이어서 수행이 가능한 것이다.  
PCB 저장 정보는 아래와 같다 :  
+  프로세스 상태 : 생성, 준비, 수행, 대기, 중지  
+  프로그램 카운터 : 프로세스가 다음 실행할 명령어 주소  
+  레지스터 : 누산기, 스택, 색인 레지스터 
+  프로세스 번호  

![](https://nesoy.github.io/assets/posts/20181113/1.png)


그렇다면 TCB는 어떨까?  
* 프로그램 상태  
* 레지스터 셋 : CPU 정보  
* 포인터 : PCB를 가르킴  

![TCB와 PCB](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FsxO0J%2FbtqEwQ5PbRD%2FkrWKDTE60qcaJpksIFcAy1%2Fimg.jpg)


위 그림을 보면 알 수 있듯이, TCB가 PCB보다 데이터가 작다. 같은 프로세스의 스위칭은 TCB 정보만 저장하며, 다른 프로세스간 스위칭을 할 떄에는 PCB/TCB 모두 저장한다.

### Context Switching 비용
Context Switching은 task를 변경하는 도중 많은 비용을 발생한다.메모리와 레지스터사이의 I/O가 많이 발생하기 때문이다.
* Cache 초기화  
* Memory Mapping 초기화  
* 메모리 접근을 위한 Kernel 실행  
또한, Context Swtiching시 CPU는 유휴상태에 돌입하며, 컨텍스트 스위칭 외 다른 작업을 할 수 없기에 오버헤드가 발생한다.


#### Process 및 Thread의 Context Swithing 비교

* Process의 Context Switching은 OS 운영체제의 스케쥴러에 의해 발생하며, Thread Context Swtiching은 프로세스 내 스레드 라이브러리를 통해 발생한다.
* PCS는 메모리 주소, 페이징테이블, 커널 리소스와 캐시를 날리는 것에 반해 TCS는 커널에 들어가고 나가는 비용만 드는것과 비슷하다.( PC, 레지스터, 스택포인터만 업데이트 한다)
* 일반적으로 PCS의 정보를 변경하는 작업이 TCS의 정보를 변경하는 작업보다 훨씬 더 많은 비용이 소모된다.
process가 thread보다 비용이 더 많이든다.  
* thread는 stack영역을 제외하고 모든 메모리(Code, Data, Heap)를 공유하기 때문에, context switching 발생시 stack영역만 변경을 해주면 되기 때문이다.   
* process Context Swtiching 발생시, 공유하는 데이터가 없으므로 캐쉬가 쌓아놓은 데이터가 무용지물이 되고, 새로운 캐쉬정보를 쌓아야 하기에 process Context Switching에 부담이 된다.  
* 캐쉬는 CPU와 메인 메모리 사이에 위치하며, CPU에서 한 번 이상 읽어들인 메모리의 데이터를 저장하고 있다가, CPU가 다시 그 메모리에 저장된 데이터를 요구할 때, 메인메모리를 통하지 않고 데이터를 전달해준다.


#### Interrupt는 언제 발생하는가?
인터럽트는 CPU가 프로그램을 실행하고 있을 때 싱행중인 프로그램 밖에서 예외 상황이 발생하여 처리가 필요한 경우, CPU에게 알려 예외 상황을 처리할 수 있도록 하는 것을 말한다. 보통 아래의 상황이 도래했을때 인터럽트가 발생한다.  
1. I/O request(입출력 요청) 
2. time slice expirec(CPU 사용시간 만료)  
3. fork a child(자식 프로세스 생성)  
4. wait for an interrupt(인터럽트 처리를 기다림)

주로 Hardware를 통한 I/O요청이나 OS/Driver 레벨의 Timer 기반 schedulling에 의해 발생한다.



***
참고 자료 : 
1. https://slideplayer.com/slide/12698810/  
2. https://nesoy.github.io/articles/2018-11/Context-Switching



---
layout: post
title: Context Switching
categories: [Java]
tags: [Java]
description: 컨텍스트 스위칭
fullview: false
comments: true
---
스레드와 프로세스의 차이에 관해 공부하였을때, 언뜻 context switching이라는 용어가 나왔다. 간략하게는  프로세스 1이 실행중이다가 잠시 멈추고 프로세스 2가 실행될 때 기존 프로세스의 상태/값을 저장하고 새로운 프로세스로 실행하도록 하는 작업이라고 알고 있었는데, 조금 더 상세하게 공부하기 위헤 포스팅에 적어본다.

### Context Switching이란 ? 
멀티 프로세스 환경에서 CPU가 **하나의 프로세스를 실행하고 있는 상태**에서 인터럽트 요청에 의해 **다음 우선 순위의 프로세스가 실행**되어야 할 때, **기존** 프로세스의 상태 또는 레지스터 값(Context)를 저장하고 CPU가 다음 프로세스를 수행하도록 **새로운** 프로세스의 상태 또는 레지스터 값(Context)를 **교체**하는 작업을 Context Switching이라고 한다.

프로세스는 OS에 의해 스케줄링되는 Process Control Block에서 관련 정보를 저장하고, 스레드는 프로세스 내 TCB(Task Control Block)에서 관리된다.  
Context Switchinga시, context switching을 수행하는 CPU는 cache 및 memory mapping 초기화 작업을 거치는 등 아무 작업도 하지 못하므로 잦은 context switching은 성능 저하를 가져온다.  

#### Context가 뭔데?
CPU가 다루는 Task(process, thread)의 정보를 말한다. Context는 프로세스의 PCB(Process Control Block)에 저장된다. 그래서 Context Switching을 할 때에, PCB의 정보를 읽어 CPU가 이전에 끊어졌던 부분부터 이어서 수행이 가능한 것이다.  
PCB 저장 정보는 아래와 같다 :
+  프로세스 상태 : 생성, 준비, 수행, 대기, 중지 
+  프로그램 카운터 : 프로세스가 다음 실행할 명령어 주소  
+  레지스터 : 누산기, 스택, 색인 레지스터 
+  프로세스 번호  


#### Interrupt는 언제 발생하는가?
인터럽트는 CPU가 프로그램을 실행하고 있을 때 싱행중인 프로그램 밖에서 예외 상황이 발생하여 처리가 필요한 경우, CPU에게 알려 예외 상황을 처리할 수 있도록 하는 것을 말한다. 보통 아래의 상황이 도래했을때 인터럽트가 발생한다.  
1. I/O request(입출력 요청) 
2. time slice expirec(CPU 사용시간 만료)  
3. fork a child(자식 프로세스 생성)  
4. wait for an interrupt(인터럽트 처리를 기다림)

주로 Hardware를 통한 I/O요청이나 OS/Driver 레벨의 Timer 기반 schedulling에 의해 발생한다.
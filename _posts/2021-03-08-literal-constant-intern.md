---
layout: post
title: Literal, Constant 그리고 Intern()
categories: [Java]
tags: [Java]
description: 자바에서 말하는 리터럴과 상수(constant)의 차이, 그리고 Intern()메소드에 대해 알아본다
fullview: false
comments: true
---

자바의 신 책을 읽다가 간단하게 literal과 상수에 대해 언급이 되었다. 이전에는 어느정도 정확하게 자리잡은 개념이였는데 자바 개념서를 오랜만에 읽다보니 헷갈리는 부분이 있어 다시 정리해본다.

## Literal 그리고 상수

이 두가지는 공통점이 있다. 바로 **변하지 않는 값(데이터)**를 의미한다.  
하지만 이 두개는 자세히 파고들면 다르다는 것을 알 수 있다.  
* 상수 : 변하지 않는 값을 저장하는 공간(리터럴을 담는 공간), 자바에서는 final 키워드를 붙여 생성한다.  
	* 스택영역에 해당 변수를 저장한다
* 리터럴 : 그 자체로 값을 의미하는 것  
	* 자바 메모리 공간 내 상수풀에 저장된다.
`final int cons = 1;`
위와 같이 상수 선언을 한다면, `cons`는 상수가 되고 우측의 `1`이 리터럴이 된다.   



## String str = "" vs String str = new String("")

결론부터 말하면 좌측 코드는 우측의 값이 *리터럴*로 생성되 str에 대입하고, 우측 코드는 Heap영역에 ""값을 가진 String 객체를 생성하여 좌측에 대입한다.  
보통 참조형 변수들은 new 생성을 통해 객체를 생성하지만, `String`은 우측에 바로 문자열을 대입할 수 있다.   
문자열은 보통 자주 쓰이기 떄문에 *리터럴 재사용, 메모리 절약*을 위해 좌측과 같은 대입식을 지원하는 것 같다.  

## Intern()
new 연산자가 아닌 리터럴("")로 String 객체 생성시, JVM은 우선 String Constant Pool 영역을 방문한다. 거기서 가진 값을 가진 String 객체를 찾으면 그 객체의 주소 값을 반환하여 참조한다. 찾지 못할 경우 String Constant Pool에 해당 값을 가진 객체를 생성하고, 그 주소 값을 반환한다.

```java
String a = "aaa";
String b = "bbb";
if (a == b) {
	System.out.println("true");
} else {
	System.out.println("false");
}
```

이렇게 특수한 String Constant로 인해, 위와 같이 리터럴로 선언한 String 변수 두개는 `true`를 반환하게 된다.


### String Constant Pool 위치
Java 6에서 스트링 풀은 Perm영역에 있었는데, Perm영역은 고정되어있고 Runtime시에도 확장되지 않아 OOM(Out Of Memory)가 자주 발생했다고 한다.  
Java 7 이후 부터는 Heap영역으로 변경되었다고 한다.(즉 GC의 대상이 되었다)  
=> 그럼 일반 상수 풀의 위치랑 스트링 상수풀의 위치랑 다른걸까?   
==> 맞다. 일반적인 상수풀은 Runtime Constant Pool로, JVM이 클래스파일을 로드하며 메소드영역에 넣는다. 그에 반해 String Constant Pool은 Heap영역의 Metaspace영역에 위치하여 GC를 수행할 수 있다.

***
참고 자료 : 
1. [[Java-26] 자바 new & Heap, Constant pool](https://catch-me-java.tistory.com/38)  
2. [Java String Constant Pool의 이해](https://bbchu.tistory.com/13)   
3. [JDK 8 적용 후, 심각한 성능 저하가 발생한다면?](https://brunch.co.kr/@heracul/1)   
4. [String 상수 풀은 어떤 곳일까?](https://devlog-wjdrbs96.tistory.com/248)  
5. [String pool vs Constant pool](https://stackoverflow.com/questions/23252767/string-pool-vs-constant-pool)  
6. [JVM Internal](https://d2.naver.com/helloworld/1230)


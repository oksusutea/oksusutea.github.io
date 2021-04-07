---
layout: post
title: Adapter Pattern
categories: [Design Pattern]
tags: [Design Pattern, OOP]
description: 스프링에서 사용되는 디자인 패턴 (1) 어댑터 패턴 
fullview: false
comments: true
---

## Adapter Pattern
어댑터 패턴은 말그대로 어댑터처럼 사용되는 패턴이다. **한 클래스의 인터페이스를 클라이언트에서 사용하고자 하는 다른 인터페이스로 변환**하는 패턴이다.   
기존에 있는 시스템에 새로운 써드파티 라이브러리가 추가된다던지, 레거시 인터페이스를 새로운 인터페이스로 교체하는 경우 코드의 재사용성을 높일 수 있는 디자인 패턴이다.


<p style="text-align:center;">
<img src="https://s3.ap-northeast-2.amazonaws.com/yaboong-blog-static-resources/diagram/adapter-pattern-1.png" width="400px">
</p>

#### Client 
* 써드파티 라이브러리나 외부 시스템을 사용하려는 쪽이다.

#### Target Interface
* Adapter가 구현하는 인터페이스이다. 클라이언트는 Target Interface를 통해 Adaptee인 써드파티 라이브러리를 사용하게 된다. 

#### Adapter
* Client와 Adaptee 중간에서 호환성이 없는 둘을 연결시켜주는 역할을 담당한다. Target Interface를 구현하며, 클라이언트는 Target  Interface를 통해 어댑터에 요청을 보낸다. 어댑터는 클라이언트의 요청을 Adaptee가 이해할 수 있는 방법으로 전달하고, 처리는 Adaptee에서 진행한다.

#### Adaptee

* 써드파티 라이브러리나 외부 시스템을 의미한다.

### 어댑터 패턴 호출 과정

<p style="text-align:center;">

<img src="https://s3.ap-northeast-2.amazonaws.com/yaboong-blog-static-resources/diagram/adapter-pattern-2.png" width="400">
</p>

클라이언트에서는 Target Interface를 호출하는 것처럼 보인다. 하지만 클라이언트의 요청을 전달받은 (Target Interface를 구현한) Adapter는 자신이 감싸고 있는 Adaptee에게 실질적인 처리를 위임한다. Adapter가 Adaptee를 감싸고 있는 것 때문에 Wrapper 패턴이라고도 불려진다.

###


### Adapter Pattern 사용 이유

* 기존 소스코드를 수정하지 않고 타겟 인터페이스에 맞추어 코드를 작성해 동작을 가능하게 할 수 있다.
* 써드파티 라이브러리가 오픈소스로 제공되지 않고 미리 컴파일된 클래스 바이너리 파일만으로 제공받아도 어댑터를 구현하여 서비스를 가능하게 할 수 있다. 

### JDK에서 사용된 어댑터 패턴
가장 대표적인 예가 자바의 `InputStreamReader`이다.  아래의 예시를 보자. 

```java
BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
```

위 코드중에서도 `System.in`은 `byte stream`으로 데이터를 읽어온다. 그에 반해 `BufferedReader`는 `character stream`으로 데이터를 읽어와야 한다. `System.in`과 `BufferedReader`가 기대하는 값이 다르지만, 여기서도 어댑터 패턴을 이용해 해결했다.

```java
public BufferedReader(Reader in){
	this(in, defaultCharBufferSize);
}
```

```java
/*
 * An InputStreamReader is a bridge from byte streams to character streams.
 */
public InputStreamReader(InputStream in) {
    super(in);
    try {
        sd = StreamDecoder.forInputStreamReader(in, this, (String)null); // ## check lock object
    } catch (UnsupportedEncodingException e) {
        // The default encoding should always be available
        throw new Error(e);
    }
}
```
`InputStreamReader`의 클래스 코드에도 보면 알 수 있듯이, 이 코드는 바이트 스트림을 캐릭터 스트림으로 변환?연결을 해주는 어댑터이다. 이와 관련된 UML은 아래와 같다 : 

<p style="text-align: center">

<img src="https://media.vlpt.us/images/cchloe2311/post/4e3e1162-45dc-41a4-8253-26ecd8091042/image.png" width="400">

</p>

### 결론

* Adaptee를 감싸고 Target Interface만 클라이언트에게 전달한다.
* Target Interface를 구현하여 클라이언트가 예상하는 인터페이스가 되도록 Adaptee의 인터페이스를 간접적으로 변경한다.
* Adaptee가 기대하는 방식으로 클라이언트의 요청을 간접적으로 변경한다.
* 호환되지 않는 우리의 인터페이스와 Adaptee를 함께 사용할 수 있다. 



***

참고자료 :

1.[디자인패턴 - 어댑터 패턴](https://yaboong.github.io/design-pattern/2018/10/15/adapter-pattern/)
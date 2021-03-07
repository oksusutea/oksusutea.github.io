---
layout: post
title: ==연산자와 equals, 그리고 hashcode
categories: [CS]
tags: [Java]
description: 두 객체를 비교하는 여러가지 방법
fullview: false
comments: true
---

자바에서는 여러가지 방법으로 객체의 동등성, 동일성에 대해 판단한다. 
여기서는 가장 자주 사용되는 세가지 비교법에 대해 정리해본다. 

## ==연산자
==연산자는 피연산자가 primitive type(int, double, boolean...)일 때는 값이 같은지 비교하고, 피연산자가 reference type일 때는 가르키는 객체가 같은지 비교한다. 즉, 두 객체가 같은 것을 가르킬 때에만 true를 반환한다.

```java
String str1 = "hello";
String str2 = "hello";
System.out.println(str1 == str2);//true(같은 리터럴 주소를 참조)
String str3 = new String("hello");
String str4 = new String("hello");
String str5 = str4;
System.out.println(str3 == str4);//false (같은 주소 참조)
System.out.println(str4 == str5);//true (각각 객체를 생성했기 때문에 주소값이 다르다)
```

## equals()
객체의 내부 값이 같은지, 동등성을 판단하는 메소드이다.  
기본적으로는 primitive type은 값이 같은지 검사하고, reference type은 객체의 주소가 같은지 검사한다.  
하지만 보통 'equals'는 내부 객체의 값이 같은지를 판단할 때 검사하기 때문에, 개발자가 클래스를 설계할 때 보통 ,equals()를 오버라이드하여 재정의한다.(equals()는 Object클래스의 메서드이기 때문에 오버라이딩 할 수 있다)

```java
String str1 = "hello";
String str2 = "hello";
System.out.println(str1.equals(str2));//true
String str3 = new String("hello");
String str4 = new String("hello");
System.out.println(str3.equals(str4));//true
```

<br/>
Object 객체의 equals() 메소드 :

```java
 public boolean equals(Object obj) {
        return (this == obj);
  }
```
String 클래스는 내부적으로 이미 `equals()`메소드를 오버라이드하였기 때문에, 값만으로도 비교를 할 수 있다.

```java
/**
     * Compares this string to the specified object.  The result is {@code
     * true} if and only if the argument is not {@code null} and is a {@code
     * String} object that represents the same sequence of characters as this
     * object.
     *
     * <p>For finer-grained String comparison, refer to
     * {@link java.text.Collator}.
     *
     * @param  anObject
     *         The object to compare this {@code String} against
     *
     * @return  {@code true} if the given object represents a {@code String}
     *          equivalent to this string, {@code false} otherwise
     *
     * @see  #compareTo(String)
     * @see  #equalsIgnoreCase(String)
     */
    public boolean equals(Object anObject) {
        if (this == anObject) {
            return true;
        }
        if (anObject instanceof String) {
            String aString = (String)anObject;
            if (!COMPACT_STRINGS || this.coder == aString.coder) {
                return StringLatin1.equals(value, aString.value);
            }
        }
        return false;
```

개발자가 본인이 설계한 객체의 값을 비교하기 위해서는, 위와 같이 equals()를 오버라이드 해야한다.  
아래와 같이 Person이라는 클래스를 설계했다고 가정해보자.  

```java
public class Person {
    private String name;
    private int age;
    
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
    public int getAge() {
        return age;
    }
    public void setAge(int age) {
        this.age = age;
    }
    public Person(String name, int age) {
        super();
        this.name = name;
        this.age = age;
    }
    @Override
    public String toString() {
        return "Person [name=" + name + ", age=" + age + "]";
    }
}
```
이럴 때 오버라이딩을 하지 않으면 equals()의 값은 `false`가 나온다.  

```
Person person1 = new Person("oksusu", 27);
Person person2 = new Person("oksusu", 27);
System.out.println(person1 == person2);//false
System.out.println(person1.equals(person2));//false, 오버라이딩을 안했다

```

이런 오류를 방지하기 위해서 보통은 IDE에서 equals()를 오버라이딩 하는 코드를 제공한다. 해당 클래스는 아래와 같이 equals()를 재정의하였다.

```java
@Override
public boolean equals(Object obj) {
    if (this == obj)
        return true;
    if (obj == null)
        return false;
    if (getClass() != obj.getClass())
        return false;
    Person other = (Person) obj;
    if (age != other.age)
        return false;
    if (name == null) {
        if (other.name != null)
            return false;
    } else if (!name.equals(other.name))
        return false;
    return true;
}
```

* 파라미터 객체가 this객체를 참조하면 true를 반환한다.
* 파라미터 객체가 null이면 false를 반환한다.
* 파라미터 객체와 해당 객체의 타입이 다를 경우 false로 반환한다.
* 이제 파라미터로 넘겨받은 객체를 해당 객체의 타입으로 형변환 하여, 해당 타입 내부 매개변수의 값을 비교한다.


## hashCode()
객체 해시코드란 **런타임 중 객체를 식별할 하나의 정수값**을 말한다. Object 클래스의 `hashCode()`는 객체의 메모리 번지를 이용해 해시코드를 만들어 리턴하기 때문에 객체마다 다른 값을 가지고 있다.  보통 `HashSet, HashMap, HashTable`등 해쉬와 연관된 컬렉션 프레임워크에서 hashCode()를 이용하여 두 객체가 동등한지 비교한다.
> HashMap코드를 타고타고 보다보면, HashMap.get() 메소드에서 그리고 hashCode()를 기준으로 key값을 비교한다는 것을 확인할 수 있다. 즉, **인스턴스의 hashCode 메소드 결과과 같다면 동일한 key로 간주**하겠다는 것이다.
> 

객체의 동등성을 판단할 때, 우선 `hashCode()` 메소드를 실행해서 리턴된 해시코드 값이 같은지 비교한다. 해시코드 값이 다를시 다른 객체로 판단하고, 해시코드 값이 같을 경우 `equals()`메소드로 다시 비교한다. 두 메소드가 모두 true를 반환해야 동등 객체로 판단한다.

<p style="center">
<img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FBd6RF%2FbtqBqixoMPq%2Fgauk75M1Fiv2HFG8kUzOA0%2Fimg.png">
</p>

해시코드는 앞서 배웠던 해싱기법처럼, 해당 객체의 메모리 주소를 해싱함수에 넣어 나오는 결과 값을 반환하기에 같은 객체가 아니더라도 값은 동일할 수 있다. 즉, hashCode()와 equals()는 아래와 같이 정리 할 수 있다.  

* equals(Object) 메소드가 true이면 두 객체의 hashCode 값은 같아야 한다.
* equals(Object) 메소드가 false이면 두 객체의 hashCode가 꼭 다를 필요는 없다. 하지만, 서로 다른 hashcode 값이 나오면 해시 테이블의 성능이 향상될 수 있다는 점은 이해하고 있어야 한다.


***
참고 자료 :  
1. [자바 equals(), hashCode(), ==연산자 비교 및 개념 정리](https://jeong-pro.tistory.com/172)  
2. [HashMap과 HashTable](https://devlog-wjdrbs96.tistory.com/88)  
3. [[기초부터자바] hashcode란? hashcode와 equals의 관계(2) {오버라이드, 재정의 문제 포함}
](http://blog.naver.com/PostView.nhn?blogId=travelmaps&logNo=220931531769&parentCategoryNo=&categoryNo=9&viewDate=&isShowPopularPosts=false&from=postView)  
4. [difference between equals() and hashCode()
](https://stackoverflow.com/questions/24446763/difference-between-equals-and-hashcode)
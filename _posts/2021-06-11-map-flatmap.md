---
layout: post
title: .map()과 .flatMap()의 차이
categories: [java]
tags: [java]
description: stream에서 쓰이는 두가지 map의 차이점
fullview: false
comments: true
---

## .map()
* 단일 스트림의 원소를 맵핑시킨 후, 매핑한 값을 다시 스트림으로 변환하는 중간 연산을 담당한다.
* 주로 객체에서 원하는 원소를 추출할 때 사용한다.
* 스트림의 각 요소에 대해 일대일로 맵핑되어 반환한다.

```java
Class Person {
	private String name;
	private int age;
	private String phone;
	....
}
```

```java
List<Person> sample = Arrays.asList(
  new Person(20, "park");
  new Person(35, "kyung");
  new Person(67, "seok");
  new Person(10, "test man");
  new Person(45, "test woman");
);

//List -> Stream<Person> -> map -> Stream<String>
Stream<String> mapStream = sample.stream()		// Stream<Person>
								.map(Person::getName);    // Stream<String>

mapStream.forEach(System.out::println);
```

### .flatMap()
* 중첩구조를 제거하여 연산을 진행한다.
* Array나 Object로 감싸져 있는 모든 원소를 단일 원소 스트림으로 반환한다.
* flatMap은 가장 작은 단위의 단일 스트림으로 반환한다.
* map()과 flatten 과정이 같이 일어난다고 생각할 수 있다.

``` java
List<Integer> list1 = Arrays.asList(1,2,3);
List<Integer> list2 = Arrays.asList(4,5,6);
List<Integer> list3 = Arrays.asList(7,8,9);
 
List<List<Integer>> listOfLists = Arrays.asList(list1, list2, list3);
 
List<Integer> listOfAllIntegers = listOfLists.stream()
                            .flatMap(x -> x.stream())
                            .collect(Collectors.toList());
 
System.out.println(listOfAllIntegers);      //[1, 2, 3, 4, 5, 6, 7, 8, 9]
```
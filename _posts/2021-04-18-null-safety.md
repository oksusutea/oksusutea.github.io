---
layout: post
title: Null Safety
categories: [Spring]
tags: [Spring Framework, Spring Core]
description: Null 관련 어노테이션
fullview: false
comments: true
---

스프링 프레임워크 5에 Null과 관련된 어노테이션이 추가되었다. 이 어노테이션을 사용함으로써 **컴파일 시점에 Null Pointer Exception을 미연에 방지**할 수 있다.

* @NonNull

```java
package me.cjkim.demoSpring51;

import org.springframework.lang.NonNull;

public class EventService {

    @NonNull
    public String createEvent(@NonNull String name){
        return "hello" + name;
    }
}

```

* @Nullable
* @NonNullApi (패키지 레벨 설정)

package-info.java

```java
@NonNullApi
package me.cjkim.demoSpring51;

import org.springframework.lang.NonNullApi;
```

위와 같이 지정하면 패키지 단에서 전체가 다 `NonNull`로 셋팅이 되고, 특정 값만 Null을 허용하고 싶을 경우 해당 값만 `@Nullable`어노테이션을 추가하면 된다.

* @NonNullFields (패키지 레벨 설정)



***
참고 자료 

1. [스프링 프레임워크 핵심 기술](https://www.inflearn.com/course/spring-framework_core#)
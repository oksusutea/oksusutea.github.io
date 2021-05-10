---
layout: post
title: 템플릿/콜백 패턴
categories: [Spring]
tags: [Spring Framework, 토비의 스프링]
description: 
fullview: false
comments: true
---

앞서 반복적으로 사용되는 코드를 템플릿화하여 중복성을 제거하고, 반대로 자주 바뀌는 부분은 동적으로 변경하여 수행될 수 있도록 변경하였다. 여러가지 방법 중, 마지막으로 전략 패턴을 이용해 적용하였었는데 그런 방식을 **템플릿/콜백 패턴**이라고 부른다. 전략 패턴의 컨텍스트를 **템플릿**이라고 부르고, 익명 내부 클래스로 만들어지는 오브젝트(구체 전략)를 **콜백**이라고 한다.

### 템플릿/콜백의 특징
여러 개의 메소드를 가진 일반적인 인터페이스를 사용할 수 있는 전략패턴과 달리, 템플릿/콜백 패턴의 콜백은 보통 **단일 메소드 인터페이스를 사용**한다. 템플릿의 작업 흐름 중 특정 기능을 위해 한 번 호출되는 경우가 일반적이기 때문이다.  
콜백 인터페이스의 메소드는 보통 파라미터가 있다. 이 파라미터는 템플릿의 작업 흐름 중 만들어지는 컨텍스트 정보를 전달받을 때 사용된다. 템플릿/콜백 패턴의 일반적인 작업 흐름은 아래와 같다 : 

1. 클라이언트에서 Callback 생성
2. 클라이언트가 Callback을 전달하며 템플릿을 호출
3. 템플릿의 workflow 시작
4. 템플릿의 참조 정보 생성
5. 템플릿이 참조정보를 전달하며 Callback 호출
6. 콜백은 클라이언트의 final변수와 템플릿이 전달한 정보를 참조하며 작업을 수행하고 작업 결과를 템플릿에 전달
7. 템플릿 작업 마무리하며 클라이언트에게 작업 결과 전달

위의 흐름을 보며 클라이언트, 템플릿의 역할을 확인할 수 있다 : 

* 클라이언트는 템플릿 안에 실행될 로직을 담은 콜백 오브젝트를 만들고, 콜백이 참조할 정보를 제공한다. 만들어진 콜백은 클라이언트가 템플릿의 메소드를 호출할 때 파라미터로 전달된다.
* 템플릿은 정해진 작업 흐름을 따라 진행하다가 내부에서 생성한 참조정보를 가지고 콜백 오브젝트의 메소드를 호출한다. 콜백은 클라이언트 메소드에 있는 정보 + 템플릿이 제공한 참조정보를 이용해 작업을 수행하고, 그 결과를 템플릿에 돌려준다.
* 템플릿은 콜백이 돌려준 정보를 사용해 작업을 마저 수행한다. 경우에 따라 최종 결과를 클라이언트에 다시 돌려주기도 한다.

### 템플릿/콜백 실제 예제
파일을 하나 만들어 모든 라인의 숫자를 더한 합을 돌려주는 코드를 만들어보겠다고 가정해보자. 우선 파일의 값을 만들기 위한 코드를 만들어야 한다.

```java
package com.example.demo.myCalculator;

import java.io.BufferedReader;
import java.io.FileReader;
import java.io.IOException;

public class Calculator {
    public Integer calcSum(String filepath) throws IOException{
        BufferedReader br = null;
        try {
            br = new BufferedReader(new FileReader(filepath));
            Integer sum = 0;
            String line = null;
            while ((line = br.readLine()) != null) {
                sum += Integer.valueOf(line);
            }
        } catch (IOException e){
            System.out.println(e.getMessage());
            throw e;
        } finally {
            if(br != null){
                try{
                    br.close();
                } catch (IOException e){
                    System.out.println(e.getMessage());
                }
            }
        }
    }
}
```

`BufferedReader`를 이용해 한 줄 한 줄 읽어 sum에 더하도록 구현하였다. 하지만 이 코드 역시 마찬가지로 읽는 과정에서 `IOException`이 발생할 수 있기 때문에, 처리하다가 문제가 발생하더라도 정상적으로 리소스를 반환할 수 있도록 try/catch/finally을 적용하였다.   
만일 여기서 파일에 있는 모든 숫자의 곱을 계산하는 기능을 추가해야 한다고 발생했다고 가정해보자. 이 상황에서 앞서 만든 기능을 복사하여 새로 만들 것인가? 그것 보다는 객체지향 설계를 통해 중복을 제거하고, 효율적인 코드를 작성하는 것이 나을 것 같다. 앞서 배운 템플릿/콜백 패턴을 적용해보도록 하자. 템플릿/콜백을 적용할 때는 템플릿과 콜백의 경계를 정하고 템플릿이 콜백에게, 콜백이 템플릿에게 각각 전달하는 내용이 무엇인지 파악하는 것이 중요하다.

```java
package com.example.demo.calculator;

import java.io.BufferedReader;
import java.io.IOException;

public interface LineCallback<T> {
    T doSomethingWithReader(String line, T result) throws IOException;
}
```

```java
private <T>T fileLineReadTemplate(String filepath, LineCallback<T> callback, T initValue) throws IOException {
        BufferedReader br = null;

        try {
            br = new BufferedReader(new FileReader(filepath));
            T result = initValue;
            String line = null;

            while ((line = br.readLine()) != null) {
                result = callback.doSomethingWithReader(line, result);
            }
            return result;
        } catch (IOException e) {
            System.out.println(e.getMessage());
            throw e;
        } finally {
            if (br != null) {
                try {
                    br.close();
                } catch (IOException e) {
                    System.out.println(e.getMessage());
                }
            }
        }
    }
```
위 코드를 통해 곱셈과 덧셈에서 반복적으로 등장하는 코드를 컨텍스트로 잡았고, 각 라인의 값과 현재까지의 결과 값을 파라미터로 전달받는 콜백 메소드를 정의하였다. 

```java
public Integer calcSum(String filepath) throws IOException {
        return fileLineReadTemplate(filepath, new LineCallback<Integer>() {
            @Override
            public Integer doSomethingWithReader(String line, Integer result) throws IOException {
                result += Integer.valueOf(line);
                return result;
            }
        }, 0);
    }

    public Integer calcMultiply(String filepath) throws IOException {
        return fileLineReadTemplate(filepath, new LineCallback<Integer>() {
            @Override
            public Integer doSomethingWithReader(String line, Integer result) throws IOException {
                result *= Integer.valueOf(line);
                return result;
            }
        }, 1);
    }
```
이렇게 범용적으로 제너릭 타입을 이용함으로써 파일을 라인 단위로 처리하는 다양한 기능을 편리하게 만들 수 있다. 

*** 
참고 자료
1. [토비의 스프링 3.1 3장 -- 템플릿](http://www.kyobobook.co.kr/product/detailViewKor.laf?mallGb=KOR&ejkGb=KOR&barcode=9788960773431)
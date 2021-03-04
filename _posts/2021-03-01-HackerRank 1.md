---
layout: post
title: Sales By Match(난이도: 하)
categories: [HackerRank]
tags: [Java, Python]
description: 코딩테스트 연습 
fullview: false
comments: true
---


## 문제
> There is a large pile of socks that must be paired by color. Given an array of integers representing the color of each sock, determine how many pairs of socks with matching colors there are.  
> 번역 : 색깔 별로 짝을 이룰 수 있는 양말들이 놓여져 있다. 양말의 색상으로 정의된 숫자 배열이 주어져있을 때, 총 몇 짝의 양말을 구할 수 있는지 구현해보아라.


## 풀이

### 자바 풀이방식(Java 8)

```java
import java.io.*;
import java.math.*;
import java.security.*;
import java.text.*;
import java.util.*;
import java.util.concurrent.*;
import java.util.regex.*;

public class Solution {

    // Complete the sockMerchant function below.
    static int sockMerchant(int n, int[] ar) {
        Map<Integer, Integer> socks = new HashMap<Integer, Integer>();
        int result = 0;
        
        for(int sock : ar){
            if(socks.containsKey(sock)){
                socks.put(sock,socks.get(sock) + 1);
            } else {
                socks.put(sock,1);
            }
        }

        for( int key : socks.keySet()){
            result = result + (socks.get(key) / 2);
        }

        return result;

    }

}

```

#### 설명 
1. 양말의 색상을 나타내는 key와 count를 value로 잡는 HashMap을 만든다.
2. input으로 주어진 양말을 하나씩 불러온다.
2. 만일 HashMap에 없을 경우, key : input값, value : 1로 HashMap에 넣어준다.
3. 만일 HashMap에 있을경우, 기존 value값에 1을 더해준다.
4. HashMap의 keySet을 기준으로 조회하는데, 각 key의 value에 2를 나눈 값을 result에 더해준다.(남은 값은 버리기)
5. result를 반환한다.

#### 참고코드

```java
    Set<Integer> colors = new HashSet<>();
    int pairs = 0;

    for (int i = 0; i < n; i++) {
        if (!colors.contains(c[i])) {
            colors.add(c[i]);
        } else {
            pairs++;
            colors.remove(c[i]);
        }
    }

    System.out.println(pairs);
```

* 시간복잡도 : O(n)
* 공간복잡도 : O(n)
 
내가 작성한 코드와의 차이점 :   
* 나는 HashMap으로 기준을 잡았는데, 참고 코드는 HashSet을 사용했다. 아마 Key-Value 매칭 방법이 필요 없어서 그런 것 같다.  
* 다른사람은 `ArrayList.contains()`와 `HashSet.contains()`에 대해 비교글을 올렸는데, HashSet의 `contains()`가 훨씬 빠르다고 한다.  
* 내가 작성한 코드의 경우, ar배열을 한 번 조회 하고 그 다음에 hashmap을 조회하지만, 참고 코드는 ar만 조회하고 끝난다.조회하는 횟수를 조금이라도 줄일 수 있는 방법이다! 


### 파이선 풀이 방식

```python 3
def sockMerchant(n, ar):
    pears = 0
    color = set()
    for i in range(len(ar)):
        if ar[i] not in color:
            color.add(ar[i])
        else:
            pears += 1
            color.remove(ar[i])
    return pears

```

위 참고코드를 토대로 python 코드를 작성해보았다. 풀이 방식은 동일하다.

***

* 문제 URL : [Sales by Match](https://www.hackerrank.com/challenges/sock-merchant/problem?h_l=interview&playlist_slugs%5B%5D=interview-preparation-kit&playlist_slugs%5B%5D=warmup)  


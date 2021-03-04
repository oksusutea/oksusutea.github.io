---
layout: post
title: Counting Valleys(난이도: 하)
categories: [HackerRank]
tags: [Java, Python]
description: 코딩테스트 연습 
fullview: false
comments: true
---


## 문제
> Counting Valleys
> 
> Gary is an avid hiker. He tracks his hikes meticulously, paying close attention to small details like topography. 
> During his last hike he took exactly steps. For every step he took, he noted if it was an uphill, , or a downhill, step. 
> Gary's hikes start and end at sea level and each step up or down represents a unit change in altitude.
> 
> We define the following terms:
> 1. A mountain is a sequence of consecutive steps above sea level, starting with a step up from sea level and ending with a step down to sea level. 
> 2. A valley is a sequence of consecutive steps below sea level, starting with a step down from sea level and ending with a step up to sea level.
> 
> Given Gary's sequence of up and down steps during his last hike, find and print the number of valleys he walked through. 
> 
> For example, if Gary's path is , he first enters a valley units deep. Then he climbs out an up onto a mountain units high. Finally, he returns to sea level and ends his hike.
> 
> Sample Input :
> 8
> UDDDUDUU
> 
> Sample Output :
1



## 풀이

### 자바 풀이방식(Java 8)

```java
public static int countingValleys(int steps, String path) {
    // Write your code here
    int levels = 0;
    int result = 0;
    for(int i=0; i<steps; i++){
        int upNdown = 0;
        if(path.charAt(i) == 'U'){
            upNdown = 1;
        } else {
            upNdown = -1;
        }
        levels = levels + upNdown;
        if(upNdown == 1 && levels == 0) result++;
        
    }
    return result;
    }

```

#### 설명 
1. 등산 루트(path)를 한글자씩 읽는다.
2. "U" 혹은 "D" 여부에 따라 현재 레벨에서 더하거나 빼준다.
2. 만일 현재 level이 0이고, 상승하는 단계였다면 valley이므로 result++ 해준다.
5. result를 반환한다.

**두둥!** 쉬운 문제라서 그런지 참고코드와도 동일했다.  
 해당 풀이 방법을 기준으로 python으로 작성해보았다.


### 파이선 풀이 방식

```python 3
def countingValleys(steps, path):
    # Write your code here
    result = 0
    level = 0
    for step in path: 
        upNdown = 0
        if step == "U":
            upNdown += 1
        else: 
            upNdown += -1
        level += upNdown
        
        if level == 0 and upNdown == 1 :
            result += 1
    return result

```


***

* 문제 URL : [Counting Valleys](https://www.hackerrank.com/challenges/counting-valleys/problem?h_l=interview&playlist_slugs%5B%5D=interview-preparation-kit&playlist_slugs%5B%5D=warmup)  


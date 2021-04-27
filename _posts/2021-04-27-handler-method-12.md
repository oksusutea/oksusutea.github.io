---
layout: post
title: 핸들러 메소드 (12) - ResponseEntity
categories: [Spring]
tags: [Spring Framework, Spring MVC]
description: 스프링에서 파일 리소스를 읽어오는 방법
fullview: false
comments: true
---

앞선 파트에서는 `MultipartFile`을 이용해서 파일을 업로드 하는 방법을 배웠다. 이번에는 `ResourceLoader`를 이용하여 파일 리소스를 읽어오는 방법에 대해 알아보자.

#### 1. 파일 리소스를 읽어오는 방법
* 스프링 `ResourceLoader` 사용하기

#### 2. 파일 다운로드 응답 헤더에 설정할 내용
* Content-Disposition : 사용자가 해당 파일을 받을 때 사용할 파일 이름
* Content-Type : 어떤 파일인가
* Content-Length: 얼마나 큰 파일인가.


#### 3. ResponseEntity로 리턴하기
* 응답 상태 코드
* 응답 헤더
* 응답 본문

#### 예시 코드 : 
컨트롤러 : 

```java
    @GetMapping("/file/{filename}")
    public ResponseEntity fileDownload(@PathVariable String filename) throws IOException {
        Resource resource = resourceLoader.getResource("classpath:" + filename);
        File file = resource.getFile();
        return ResponseEntity.ok()
                .header(HttpHeaders.CONTENT_DISPOSITION, "attachement; filename=\"" + resource.getFilename() + "\"")
                .header(HttpHeaders.CONTENT_TYPE, "image/png")
                .header(HttpHeaders.CONTENT_LENGTH, String.valueOf(file.length()))
                .body(resource);
    }
```
응답을 `ResponseEntity`로 하게 되면 클라이언트에게 파일을 제공해 줄 수 있다.

***
참고자료

1. [스프링 웹 MVC](https://inf.run/dJFi)
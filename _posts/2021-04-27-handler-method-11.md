---
layout: post
title: 핸들러 메소드 (11) - MultipartFile
categories: [Spring]
tags: [Spring Framework, Spring MVC]
description: 파일 업로드시 사용하는 메소드 아규먼트
fullview: false
comments: true
---

### MultipartFile
* MultipartResolver 빈이 설정되어 있어야 사용할 수 있다.(스프링 부트 자동 설정이 셋팅해준다)
* POST multipart/form-data 요청에 들어있는 파일을 참조할 수 있다.
* List<MultipartFile>아규먼트로 여러 파일을 참조할 수 있다.

`DispatcherServlet` 파일 : 

```java
private void initMultipartResolver(ApplicationContext context) {
        try {
            this.multipartResolver = (MultipartResolver)context.getBean("multipartResolver", MultipartResolver.class);
            if (this.logger.isTraceEnabled()) {
                this.logger.trace("Detected " + this.multipartResolver);
            } else if (this.logger.isDebugEnabled()) {
                this.logger.debug("Detected " + this.multipartResolver.getClass().getSimpleName());
            }
        } catch (NoSuchBeanDefinitionException var3) {
            this.multipartResolver = null;
            if (this.logger.isTraceEnabled()) {
                this.logger.trace("No MultipartResolver 'multipartResolver' declared");
            }
        }

    }
```
DispatcherServlet은 초기화 할 때 `MultipartResolver`를 빈으로 등록해준다. 이 과정에서 multipartResolver가 없다면 등록해 주지 않고 있는 경우에만 빈으로 등록해준다. 스프링부트에서는 `MultipartAutoConfiguration`에서 `MultipartResolver `을 빈으로 등록해준다.


### MultipartFile 사용방법

파일 업로드 폼은 아래와 같다 : 

```html
<!DOCTYPE html>
<html lang="en" xmlns:th = "http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>File Upload</title>
</head>
<body>
<form method="POST" enctype="multipart/form-data" action="#" th:action="@{/file}">
    File: <input type="file" name="file"/>
    <input type="submit" value="Upload"/>
</form>
</body>
</html>
```
POST요청으로 `multipart/form-data`인코딩 타입으로 요청을 보내면, 그 요청에 있는 파일을 `MultipartFile`이라는 아규먼트로 보낼 수 있다.


컨트롤러 파일 : 

```java
package me.cjk.demo;

import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.multipart.MultipartFile;
import org.springframework.web.servlet.mvc.support.RedirectAttributes;

@Controller
public class FileController {

    @GetMapping("/file")
    public String fileUploadForm(Model model){
        return "/files/indexes";
    }

    @PostMapping("/file")
    public String fileUpload(@RequestParam MultipartFile file,
                             RedirectAttributes attributes){
        //스토리지에 save했다고 가정
        String message = file.getOriginalFilename() + " is uploaded";
        attributes.addFlashAttribute("message", message);
        return "redirect:/file";
    }

}
```
위와 같이 작성하게 되면 폼에서 파일을 업로드 하고, 파일명이 페이지에 노출된다. 

테스트 코드 : 

```java
package me.cjk.demo;

import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.AutoConfigureMockMvc;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.mock.web.MockMultipartFile;
import org.springframework.test.web.servlet.MockMvc;

import static org.junit.jupiter.api.Assertions.*;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.multipart;
import static org.springframework.test.web.servlet.result.MockMvcResultHandlers.print;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

@SpringBootTest
@AutoConfigureMockMvc
class FileControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @Test
    public void fileUploadTest() throws Exception{
        MockMultipartFile file = new MockMultipartFile("file",
                                                "test.txt",
                                                    "text/plain",
                                                                "hello file".getBytes());
        this.mockMvc.perform(multipart("/file").file(file))
                .andDo(print())
                .andExpect(status().is3xxRedirection());
    }
}
```

`MockMultipartFile`은 테스트용으로 파일을 만들어줄 수 있도록 만든 타입이다. mockMvc에서 `multipart()`메소들 파일을 업로드 하여 리다이렉션이 되었는지 테스트를 할 수 있다.

***
참고자료

1. [스프링 웹 MVC](https://inf.run/dJFi)
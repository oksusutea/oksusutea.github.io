---
layout: post
title: 소소소플젝 01. 체크리스트 점검 자동화 - 셀레니움
categories: [Project]
tags: [Java]
description: 업무시 나를 보다 편하게 해주기 위해 진행하는 소소한 프로젝트
fullview: false
comments: true
---



## Selenium을 통해서 점검 페이지까지 이동하기
selenium은 파이썬에서 훨씬 많이 쓰이지만, 자바로도 제공하는 라이브러리이다. 필자는 자바 개발 능력을 키우기 위해서 자바로 개발을 진행하였고, API를 보니 파이썬과 크게 다르지 않았다. 

### Selenium으로 웹 크롤링하기

#### 1. 셀레니움 추가

```java
dependencies {
	...
	// https://mvnrepository.com/artifact/org.seleniumhq.selenium/selenium-java 210303기준
	implementation group: 'org.seleniumhq.selenium', name: 'selenium-java', version: '3.141.59'
}
```
셀레니움을 사용하려면 gradle파일에 셀레니움을 추가해야한다. 위와 같이 추가하고 빌드를 하고 나면, External Library에 셀레니움과 관련된 라이브러리가 추가된 것을 볼 수 있다.

<p style="center">

<img src="https://user-images.githubusercontent.com/75205849/112747775-6ee08e80-8ff2-11eb-8bf3-9a0fb67107ac.png">
</p>

<br/>

#### 2. 셀레니움 드라이버 다운로드
셀레니움은 테스트용 프레임워크이기 때문에, 브라우저를 통해 웹에서 자동화를 구현해준다. 셀레니움 사이트에 들어가보면 여러가지 브라우저를 지원하는 드라이버가 있는데, 필자는 크롬 드라이버를 다운받아 적용하였다. 

<br/>

#### 3. 셀레니움을 통한 인트라넷 사이트 접근

셀레니움을 사용하기 위해서는 하기 프로세스대로 코드를 작성해야 한다. 

1. driver 설정
2. Chrome 드라이버 인스턴스 설정
3. URL 접속

아래 단계에서 해당 작업에 관한 코드까지 함께 기재하였으니 같이 확인해보자.

<br/>

#### 4. 인트라넷 로그인 기능 구현

<p style="center">
<img src="https://user-images.githubusercontent.com/75205849/112747304-77839580-8fef-11eb-8198-4ddbab8bd6ba.png" width="450" >
</p>

`chromeDriver`를 통해 인트라넷 사이트에 접속하면 먼저 로그인 페이지가 로딩된다. 
이 로그인 페이지는 폼 구조에 변동사항이 없고, 인트라넷 아이디와 패스워드만 입력해 로그인만 진행하면 된다.  
나의 ID와 비밀번호를 프로퍼티 파일에 넣어놓고, 해당 파일에서 읽어온 값을 입력하도록 구현하였다. 



```Java
package cjonhrpart.cjkim.dailychecklistforqao.selenium;


import cjonhrpart.cjkim.dailychecklistforqao.exception.CrawlException;
import cjonhrpart.cjkim.dailychecklistforqao.properties.UserProperties;
import lombok.extern.slf4j.Slf4j;
import org.openqa.selenium.By;
import org.openqa.selenium.Keys;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.chrome.ChromeDriver;
import org.openqa.selenium.chrome.ChromeOptions;
import org.openqa.selenium.support.ui.WebDriverWait;
import org.springframework.core.io.ClassPathResource;

import java.io.IOException;
import java.util.Properties;
import java.util.concurrent.ThreadLocalRandom;

@Slf4j
public class SeleniumBrowser {
    private final Properties properties;
    private WebDriver browser;
    private WebDriverWait wait;

    public SeleniumBrowser() {

        properties = new UserProperties().getProperties();

        initDriver();
        login(properties);
    }

    private void login(Properties properties) {
        browser.get(properties.getProperty("base-url"));
        WebElement idElement = browser.findElement(By.id("txtID"));
        WebElement passwordElement = browser.findElement(By.id("txtPWD"));

        sleep(2000);

        typeId(idElement, properties.getProperty("cjw-id"));
        typePassword(passwordElement, properties.getProperty("cjw-pw"));
    }

    private void initDriver()  {
        //System Property SetUp
        var classPathResource = new ClassPathResource("seleniumDriver/chromedriver");
        try {
            System.setProperty(properties.getProperty("WEB_DRIVER_ID"), classPathResource.getURL().getPath());
        } catch (IOException e) {
            e.printStackTrace();
        }

        ChromeOptions options = new ChromeOptions();
        //options.addArguments("--headless"); // 브라우저 키지 않는다
        //options.addArguments("--no-sandbox");
        //options.addArguments("--disable-dev-shm-usage");

        this.browser = new ChromeDriver(options);
        this.wait = new WebDriverWait(this.browser, 20);
    }
    private void typeId(WebElement idElement, String userId) {
        idElement.click();
        humanizeTyping(idElement, userId);
        sleep(300);

        idElement.sendKeys(Keys.TAB);
    }

    private void typePassword(WebElement passwordElement, String userPassword) {
        passwordElement.clear();
        humanizeTyping(passwordElement, userPassword);
        sleep(300);

        passwordElement.sendKeys(Keys.RETURN);
    }

    private void humanizeTyping(WebElement element, String input) {
        for(char c : input.toCharArray()) {
            element.sendKeys(String.valueOf(c));
            sleep((int) (ThreadLocalRandom.current().nextLong(100) + 100));
        }
    }

    private void sleep(int time) {
        try {
            Thread.sleep(time);
        } catch (InterruptedException e) {
            throw new CrawlException("Fail to sleep");
        }
    }

}
```

ID와 PW를 입력하는 과정에서, 너무 빠르게 입력하면 다음 URL의 element 값을 못읽어오는 현상이 있어서, `humanizeTyping()`메소드를 통해 사람이 하나하나 타이핑하는것처럼 시간간격을 주었다. 

<br/>

#### 5. 서비스 데스크로 이동 

<p style= "center">

<img src = "https://user-images.githubusercontent.com/75205849/112750290-ca1a7d00-9002-11eb-801d-a692b121f3e5.png" width="450">

</p>

로그인을 성공적으로 수행한 뒤, 이제는 서비스데스크로 이동해야 한다. 
인트라넷 사이트에서 개발자 도구를 입력해 해당 링크를 분석하고, 이동해서 클릭까지 수행해준다. 

#### 6. 점검구분/항목 비교

팝업창으로 띄워진 서비스데스크에서, 체크해야 할 점검항목까지 확인하기 위해 페이지를 점진적으로 이동해준다. 
이 과정에서 `Selenium test IDE`와 개발자도구를 적극 활용해주었다. 

#### 7. 점검항목 및 미수행자 불러오기

<p style="center">

<img src="https://user-images.githubusercontent.com/75205849/112752276-24204000-900d-11eb-8587-893532042661.png" width="400">
</p>

위 페이지까지 이동했으면 이제 더이상의 페이지 이동은 필요없다. 여기서 각 점검항목 데이터를 가져와 `CheckList` 인스턴스에 넣어주었다. 

*** 


셀레니움에 관해 작업한 항목은 여기까지이므로,
다음 챕터에서는 `Selenium`을 통해 불러온 데이터를 어떻게 `CheckList`의 배열에 넣어뒀는지에 대한 방법과 JFrame 적용 방법에 대해 기재할 예정이다. 


<br/>
<br/>



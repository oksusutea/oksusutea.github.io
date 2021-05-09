---
layout: post
title: HTTP 메시지 컨버터
categories: [Spring]
tags: [Spring Framework, Spring MVC]
description: HTTP 메시지를 변경하는 HTTP 메시지 컨버터
fullview: false
comments: true
---

## HTTP Message Converter
요청 본문에서 메시지를 읽거나(@RequestBody), 응답 본문에 메시지를 작성할 때(@ResponseBody) 사용한다. HTTP 요청에 메시지 본문과 응답 메시지 본문을 통째로 메시지로 다루도록 하는 방식이다.

```java
@GetMapping("/message")
@ResponseBody
public  String message(@RequestBody Person person){
        return "hello person";
    }
```


위와 같이 `@RequestBody`가 들어있으면, 요청 본문에 들어있는 메시지를 HTTP 메시지 컨버터를 사용해서 컨버전을 진행한다. 위에서는 `Person`으로 컨버전을 진행한다.  
`@ResponseBody` 애노테이션은, 리턴 값을 응답의 본문으로 반환하는 역할을 한다.

#### 기본 HTTP 메시지 컨버터
메시지 컨버터는 `AnnotationMethodHandlerAdapter`를 통해 등록한다.

* 바이트 배열 컨버터 -- `byteArrayHttpMessageConverter`
	* Request시 : 요청 메시지를 `byte[]`로 컨버전 할 수 있다.
	* Response시 : `Content-Type:application/octet-stream`으로 설정된다
	* 컨트롤러가 byte배열에 담긴 바이너리 정보를 클라이언트에게 전송할 때 외에는 유용하지 않다.
* 문자열 컨버터  -- `StringHttpMessageConverter`
	* Request시 : 요청 메시지의 본문을 그대로 String으로 가져온다.
	* Response시 : `Content-Type:text/plain`으로 전달된다.
* Resource 컨버터
* Form 컨버터
* (JAXB2 컨버터) - XML용
* (Jackson2 컨버터) - JSON용
* (Jackson 컨버터) - JSON용
* (Gson 컨버터) - JSON용
* .....

### HTTP 메시지 컨버터 적용 방법
* 기본으로 등록해주는 컨버터에 새로운 컨버터 추가하기 : `extendMessageConverters`
* 기본으로 등록해주는 컨버터는 다 무시하고 새로운 컨버터 설정하기 : `configureMessageConverters`
* 의존성 추가로 컨버터 등곧
	* 메이븐 또는 그래들 설정에 의존성을 추가하면 그에 따른 컨버터가 자동으로 등록된다.
	* `WebMvcConfigurationSupport`
	* 스프링 프레임워크의 기능이며, 스프링 부트의 기능이 아니다.
	* `WebMvcConfigurationSupport`에서는 classpath에 해당하는 컨버터가 존재할 경우에만 등록을 해준다.

***

### JSON HTTP Message Converter
* 스프링 부트를 사용하지 않는 경우
	* 사용하고 싶은 JSON 라이브러리를 의존성으로 추가한다.
	* GSON 컨
* 스프링 부트를 사용하는 경우
	* 기본적으로 **JacksonJSON 2**가 의존성으로 들어가있다.
	* 즉, **JSON용 HTTP 메시지 컨버터가 기본**으로 등록되어 있다.


### 활용 예시

컨트롤러 파일 : 
```java
@GetMapping("/jsonMessage")
    public Person jsonMessage(@RequestBody Person person){
        //json으로 요청이 들어올 경우 요청을 Person객체로 변환(컨버팅)하여 반환
        return person;
    }
```
스프링 부트에서는 기본으로 **JSON 컨버터를 제공**하기에 의존성 라이브러리를 추가로 임포트해주지 않아도 된다.

테스트 코드 : 

```java
@Test
    public void jsonMessage() throws Exception{
        Person person = new Person();
        person.setId(2021l);
        person.setName("cjkim");

        String jsonString = objectMapper.writeValueAsString(person);
        this.mockMvc.perform(get("/jsonMessage")
                .contentType(MediaType.APPLICATION_JSON_UTF8) //본문에 JSON을 담아서 보낸다.
                .accept(MediaType.APPLICATION_JSON_UTF8) //JSON으로 응답이 오기를 바란다.
                .content(jsonString))
                .andDo(print())
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.id").value(2021))
                .andExpect(jsonPath("$.name").value("cjkim"));
    }```

결과 :
 
```java
MockHttpServletRequest:
      HTTP Method = GET
      Request URI = /jsonMessage
       Parameters = {}
          Headers = [Content-Type:"application/json;charset=UTF-8", Accept:"application/json;charset=UTF-8", Content-Length:"26"]
             Body = {"id":2021,"name":"cjkim"}
    Session Attrs = {}

Handler:
             Type = me.cjk.demospringmvc.SampleController
           Method = me.cjk.demospringmvc.SampleController#jsonMessage(Person)

MockHttpServletResponse:
           Status = 200
    Error message = null
          Headers = [Content-Type:"application/json;charset=UTF-8"]
     Content type = application/json;charset=UTF-8
             Body = {"id":2021,"name":"cjkim"}
    Forwarded URL = null
   Redirected URL = null
          Cookies = []
```

json을 직접 url로 테스트 하기에는 어려우니, postman을 이용해서 테스트를 진행 할 수 있다.

***

### HTTP Message Converter : XML

* 스프링 부트를 사용하지 않는 경우  
* 스프링 부트를 사용하는 경우 
모두 동일하다, 스프링 부트는 **기본적으로 XML 의존성을 추가해주지 않기 때문**이다. XML HTTP Message Converter를 사용하기 위해서는 OXM(Object-XML Mapper) 라이브러리 중에 스프링이 지원하는 의존성을 추가해야 한다.
* JacksonXML
* JAXB

이전처럼 `WebMvcConfigurer`를 구현해서 거기에 `XMLConverter`를 추가하는 것이 아니라, 의존성만 추가하면 자동으로 변환해준다.


#### 활용 예시

build.gradle에 의존성 추가하기

```
implementation group: 'javax.xml.bind', name: 'jaxb-api', version: '2.4.0-b180830.0359' //인터페이스
implementation group: 'org.glassfish.jaxb', name: 'jaxb-runtime', version: '3.0.0' //구현체
implementation group: 'org.springframework', name: 'spring-oxm'
```
* jaxb-api : 인터페이스
* jaxb-runtime : 인터페이스 구현체
* oxm marshaller : xml <-> 객체 변환

WebConfig에 빈 설정하기 : 

```java
@Bean
    public Jaxb2Marshaller jaxb2Marshaller() {
        Jaxb2Marshaller jaxb2Marshaller = new Jaxb2Marshaller();
        jaxb2Marshaller.setPackagesToScan(Person.class.getPackageName());
        return jaxb2Marshaller;
    }
```
여기서 `Person`클래스는 `@XmlRootElement`라는 jaxb에서 사용하는 애노테이션을 기재해야 한다.

테스트 코드 :

```java
    @Autowired
    Marshaller marshaller;
    
    @Test
    public void xmlMessage() throws Exception {
        Person person = new Person();
        person.setId(2021l);
        person.setName("cjkim");

        StringWriter stringWriter = new StringWriter();
        Result result = new StreamResult(stringWriter);
        marshaller.marshal(person,result);
        String xmlString = stringWriter.toString();

        this.mockMvc.perform(get("/jsonMessage")
                .contentType(MediaType.APPLICATION_XML) //본문에 JSON을 담아서 보낸다.
                .accept(MediaType.APPLICATION_XML) //JSON으로 응답이 오기를 바란다.
                .content(xmlString))
                .andDo(print())
                .andExpect(status().isOk())
                .andExpect(xpath("person/name").string("cjkim"))
                .andExpect(xpath("person/id").string("2021"));

    }
```

출력 결과 : 

```
MockHttpServletRequest:
      HTTP Method = GET
      Request URI = /jsonMessage
       Parameters = {}
          Headers = [Content-Type:"application/xml;charset=UTF-8", Accept:"application/xml", Content-Length:"103"]
             Body = <?xml version="1.0" encoding="UTF-8" standalone="yes"?><person><id>2021</id><name>cjkim</name></person>
    Session Attrs = {}

MockHttpServletResponse:
           Status = 200
    Error message = null
          Headers = [Content-Type:"application/xml"]
     Content type = application/xml
             Body = <?xml version="1.0" encoding="UTF-8" standalone="yes"?><person><id>2021</id><name>cjkim</name></person>
    Forwarded URL = null
   Redirected URL = null
          Cookies = []
```

`xml`은 id가 long이더라도 string으로 검증을 해야한다. 

#### 정리
xml을 이용한 HTTP 메시지 컨버터에 대해 알아보았다. JSON에 비해 훨씬 복잡한 것을 알 수 있다. 의존성 라이브러리를 추가하고, 빈을 등록하고, xml을 테스트 하는 과정중에서도 정확하게 타입 검증을 할 수 없었다. 그래도 xml도 이렇게 커스터마이징하여 HTTP Message Converter를 작성할 수 있다는 것을 알아두자.
 

***
참고자료

1. [스프링 웹 MVC](https://inf.run/dJFi)
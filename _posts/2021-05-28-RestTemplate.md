---
layout: post
title: RESTTemplate
categories: [Spring]
tags: [Spring Framework, Template]
description: 
fullview: false
comments: true
---

## RestTemplate이란?
RestTemplate은 http 통신을 단순화하고 REST 방식으로 API를 호출할 수 있는 내장 클래스이다. JDBCTemplate처럼 기계적이고 반복적인 코드를 줄여주고 필요한 비즈니스 로직에만 집중할 수 있도록 도와준다. 정리해보면 아래 세가지 특징을 갖고 있다고 할 수 있다 : 
* 기계적이고 반복적인 코드를 최대한 줄여줌
* RESTful형식을 맞춤
* json, xml을 쉽게 응답받음

하지만 아래의 단점도 존재한다  : 

* 기본적으로 Connection Pool을 사용하지 않는다.
* Default로 java.net.HttpURLConnection을 사용해 많은 요청을 다루기 힘들다.
* 많은 요청을 하면 TIME_WAIT로 인해 자원이 부족해 질 수 있기 때문에, Connection Pool을 만들어 사용하기를 권장하고 있다.

**현재는 Deprecated 되어 WebClient를 사용하도록 가이드 하고 있다**
> NOTE: As of 5.0 this class is in maintenance mode, with only minor requests for changes and bugs to be accepted going forward. Please, consider using the org.springframework.web.reactive.client.WebClient which has a more modern API and supports sync, async, and streaming scenarios.

## HTTP 서버와 통신하는 방법

### 1) URLConnection
`java.net` 패키지에 있다. URL의 내용을 읽어오거나, URL주소에 GET, POST로 데이터를 전달할 때 주로 사용한다. http 프로토콜 말고도 사용이 가능하다.

1. new URL("http://...")
2. openConnection()
3. URLConnection
4. getInputStream, getOutputStream
5. InputStream, OutputStream 처리

#### 단점 
* 응답코드가 400번대 or 500번대 일 경우 IOException이 발생한다.
* 타임아웃을 설정할 수 없다.
* 쿠키를 제어 할 수 었다.


### 2) HttpClient
`org.apache.http`패키지에 있으며 아래와 같이 사용 가능하다.

1. CloseableHttpClient httpClient = HttpClients.createDefault();
2. new HttpGet("http://...") (메소드에 따라 다르다)
3. CloseableHttpResponse response = httpclient.execute(httpget);
4. HttpEntity entity = response.getEntity();
5. Stream으로 entity.getContent() 처리 등

#### 장점
* `httpResponse.getStatusLine().getStatusCone()`를 이용해 모든 응답코드를 읽을 수 있다
* 타임아웃 설정 가능하다.
* 쿠키를 제어할 수 있다.


#### 단점
* URLConnection보다 코드가 간결하지만, 여전히 반복적이고 코드들이 길다.
* 스트림 처리 로직을 별도로 작성해야 한다.
* 응답의 컨텐츠타입에 따라 별도 로직이 필요하다.

***
## RestTemplate의 동작 원리

<p style="text-align:center">

<img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=http%3A%2F%2Fcfile26.uf.tistory.com%2Fimage%2F99300D335A9400A52C16C1">
</p>

1. 어플리케이션이 RestTemplate을 생성하고, URI, HTTP 메소드등의 헤더를 담아 요청한다.
2. RestTemplate은 HttpMessageConverter를 사용하여 RequestEntity를 요청메시지로 전환한다.
3. RestTemplate는 ClientHttpRequestFactory로부터 ClientHttpRequest를 가져와서 요청을 보낸다.
4. ClientHttpRequest는 요청 메시지를 만들어 HTTP 프로토콜을 통해 서버와 통신한다.
5. RestTemplatesms ResponseErrorHandler로 오류를 확인하고, 있다면 처리 로직을 수행한다.
6. ResponseErrorHandler는 오류가 있다면 ClientHttpResponse에서 응답 데이터를 가져와 처리한다.
7. RestTemplate는 HttpMessageConverter를 이용해 응답 메시지를 java object로 변환한다.
8. 어플리케이션에 반환한다.


### Connection Pool 미적용
RestTemplate은 앞서 이야기 했듯이 기본적으로 connection pool을 사용하지 않는다. 따라서 연결할 때마다, 로컬 포트를 열고 tcp connection을 맺는다. 문제는 close() 이후에 사용된 소켓은 TIME_WAIT 상태가 되는데, 요청량이 많다면 이런 소켓들을 재사용하지 못하고 응답이 지연되는 상황이 발생한다.  
이런 경우 connection pool을 사용해서 해결할 수 있고, DBCP와 같이 소켓의 갯수를 정할 수 있다. 
RESTTemplate에서 Connection Pool을 적용하려면, HttpClient를 만들고 setHttpClient()를 사용하자.

* setMaxConnPerRoute : IP,포트 1쌍에 대해 수행 할 연결 수를 제한한다.
* setMaxConnTotal : 최대 오픈되는 커넥션 수를 제한한다.


** 

## 예제 코드
```java
import org.apache.http.client.HttpClient; 
import org.apache.http.impl.client.HttpClientBuilder; 
import org.springframework.http.client.HttpComponentsClientHttpRequestFactory; 
import org.springframework.web.client.RestTemplate; 
public class RestTemplateEx { 
	public static void main(String[] args) { 
		HttpComponentsClientHttpRequestFactory factory = new HttpComponentsClientHttpRequestFactory(); 
		factory.setReadTimeout(5000); // 읽기시간초과, ms 
		factory.setConnectTimeout(3000); // 연결시간초과, ms 
		HttpClient httpClient = HttpClientBuilder.create() 
							.setMaxConnTotal(100) // connection pool 적용 
							.setMaxConnPerRoute(5) // connection pool 적용
							.setConnectionTimeToLive(5, TImeUnit.SECONDS) // Keep - alive
							.build(); 
		 factory.setHttpClient(httpClient); // 동기실행에 사용될 HttpClient 세팅 
		 RestTemplate restTemplate = new RestTemplate(factory); 
		 String url = "http://testapi.com/search?text=1234"; // 예제
		 Object obj = restTemplate.getForObject("요청할 URI 주소", "응답내용과 자동으로 매핑시킬 java object");
		System.out.println(obj);
 	}
}
```
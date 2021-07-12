---
layout: post
title: DB를 사용하면서 발생 가능한 문제점들
categories: [java]
tags: [java, spring, 자바 성능 튜닝 이야기]
description: DB 사용시 유의해야 할 점
fullview: false
comments: true
---


* 자바 기반 어플리케이션 성능 진단시, 어플리케이션의 응답 속도를 지연시키는 대부분의 요인은 DB 쿼리 수행 시간과, 결과를 처리하는 시간이다.
* 이 챕터는 해당 부분의 실수를 최소화 하기 위해 작성된 내용이다.


## DB Connection과 Connection Pool, DataSource
* 일반적으로 자바에서는 DB에 연결하여 아래와 같이 사용하고 있다.

```java
try {
	Class.forName("oracle.jdbc.driver.OracleDriver");
	Connection con = DriverManaer.getConnection("jdbc:oracle:thin:@ServerIP:1521:SID","ID","Password");
	PreparedStatement ps = con.prepareStatement("SELECT ... where id=?");
	ps.setString(1,id);
	ResultSet rs = ps.executQuery();
	// 중간 데이터 처리 부분 생략
} catch(ClassNotFoundException e) {
	System.out.println("드라이버 load fail");
	throw e;
} catch(SQLException e){
	System.out.println("Connection fail");
	throw e;
} finally {
	rs.close();
	ps.close();
	con.close();
}
```
위 코드에서 수행되는 내용을 정리하자면 아래와 같다 : 
* 드라이버 로드
* getConnection 메서드에 DB 서버의 IP와 PW등을 매개변수로 전달하여 Connection 객체 생성
* Connection으로부터 PreparesStatement 객체를 받는다.
* executeQuery를 수행하여 그결과로 ResultSet 객체를 받아 데이터 처리
* 모든 데이터 처리 완료 후, finally 구문을 이용해 ResultSet, PreparesStatement, Connection 객체를 닫는다.

<br/>

* 위 코드에서 가장 시간이 많이 소요되는 부분은 Connection 객체를 얻는 부분이다. DB와 WAS간 통신을 진행해야 하며, 이 때에는 TCP 통신을 해야하기 때문이다.
* 이렇게 Connection 객체를 생성하는 부분에서 발생하는 대기 시간을 줄이고, 네트워크의 부담을 줄이기 위해 DB Connection Pool을 사용하게 되었다.
* WAS에서 Connection Pool을 제공하고, DataSource를 사용해 JNDI로 호출해 쓸 수 있다.


<br/>

### Statement, PreparedStatement
* 쿼리를 실행할 때 사용하는 객체이며, 아래와 같은 프로세스를 거친다.
	* 쿼리 문장 분석
	* 컴파일
	* 실행

* Statement를 사용하면 쿼리 수행시마다 위 단계를 거치지만, PreparedStatement는 처음 한 번만 세 단계를 거친 후, 캐시에 담아 재사용 한다. 동일한 쿼리를 반복적으로 수행하면 PreparedStatement를 사용하는 것이 효율적이다.

## DB를 사용할 때 닫아야 하는 것들
* 일반적으로 객체를 생성하는 순서는 Connection, Statement, Resultset 순이며, 객체를 닫는 순서는 이의 반대 순서로 적용된다.
* 즉, 먼저 얻은 객체를 가장 마지막에 닫는다.

#### ResultSet 객체가 닫히는 경우
* close() 메서드 호출시
* GC의 대상이 되어 GC되는 경우
* 관련 Statement 객체의 close() 메서드 호출시

<br/>

* Statement 객체가 close시 자동으로 닫히고, GC시에도 자동으로 닫히는데 close()메소드를 호출해야 하는 이유 
	* 자동으로 호출되기 전에 관련된 DB와 JDBC 리소스를 해제하기 위함

#### Statement 객체가 닫히는 경우
* close() 메서드 호출시
* GC의 대상이 되어 GC되는 경우

#### Connection 객체가 닫히는 경우
* close() 메서드 호출시
* GC의 대상이 되어 GC되는 경우
* 치명적인 에러 발생시


## JDK 7에서 등장한 AutoClosable 인터페이스
* AutoClosable 인터페이스에는 리턴 타입이 void인 close() 메서드 단 한개만 선언되어 있다.
* close() 메서드는 아래와 같이 설명되어 있다 : 
	* try-with-resources 문장으로 관리되는 객체에 대해 자동으로 close()처리를 한다.
	* InterruptedException을 던지지 않도록 하는 것을 권장한다.
	* 이 close() 메서드를 두 번 이상 호출할 경우 눈에 보이는 부작용이 나타나도록 개발해야 한다.

```java
public String readFile(String fileName) throws IOException {
	FilrReader reader = new FileReader(new File(fileName));
	BufferedReader br = new BufferedReader(reader);
	String data = null;
	try {
		data = br.readLine();
	} finally {
		if(br!=null) br.close();
	}
	return data;
}
```

* JDK 7 이전에는 finally 블록에 close() 메서드를 호출해주어야 했다.

```java
public String readFile(String fileName) throws IOException {
	FilrReader reader = new FileReader(new File(fileName));
	try(BufferedReader br = new BufferedReader(reader)) {
		return br.readLine();
	}
}
```

* JDK 7부터는 위와 같이 try 블록이 시작될 때 소괄호 안에 close() 메서드를 호출하는 객체를 생성해주면 finally 블록을 작성할 필요가 없다.

***
참고자료 : 
[자바 성능 튜닝 이야기 12장 - DB를 사용하면서 발생가능한 문제점들](http://www.kyobobook.co.kr/product/detailViewKor.laf?mallGb=KOR&ejkGb=KOR&barcode=9788966260928)

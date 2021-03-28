---
layout: post
title: ServletContextListener
categories: [Java]
tags: [Java, JSP]
description: ServletContextListnet의 개념 및 예제
fullview: false
comments: true
---

## `ServletContextListener`란 ? 
ServletContextListener를 이용하여 웹 어플리케이션을 실행할 때 필요한 초기화 작업이나, 웹 어플리케이션이 종료된 후 사용된 자원을 반환하는 등의 작업을 수행할 수 있다. 이 인터페이스는 두 개의 메소드를 정의하고 있다. 

* `public void contextInitialized(ServletContextEvent sce)` : 웹 어플리케이션 초기화시 호출
* `public void contextDestroyed(ServletContextEvent sce)` : 웹 어플리케이션 종료시 호출

웹 어플리케이션이 시작되고, 종료될 때 특정 기능을 수행하기 위해서 아래와 같이 작성하면 된다.  
1. `javax.servlet.ServletContextListener`를 구현한 클래스 작성  
2. `web.xml`파일에 1번에 작성한 클래스를 등록

#### web.xml 예시 파일

```
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee" 
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee 
		http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd"
	version="3.1">

	<listener>
		<listener-class>jdbc.DBCPInitListener</listener-class>
	</listener>

	<context-param>
		<param-name>poolConfig</param-name>
		<param-value>
			jdbcdriver=com.mysql.jdbc.Driver
			jdbcUrl=jdbc:mysql://localhost:3306/guestbook?characterEncoding=utf8
			dbUser=jspexam
			dbPass=jsppw
			poolName=guestbook
		</param-value>
	</context-param>

</web-app>
```

한 개 이상의 `<listener>`태그를 등록할 수 있고, 각 `<listener>`태그는 반드시 한 개의 `<listener-class>`태그를 자식 태그로 가져야 한다. `contextInitialized()`메서드는 등록된 순서되로 실행되며, `contextDestroyed()`는 이와 반대의 순서로 실행된다. 

### 구현 예시

```java
package jdbc;

import java.io.IOException;
import java.io.StringReader;
import java.sql.DriverManager;
import java.util.Properties;

import javax.servlet.ServletContextEvent;
import javax.servlet.ServletContextListener;

import org.apache.commons.dbcp2.ConnectionFactory;
import org.apache.commons.dbcp2.DriverManagerConnectionFactory;
import org.apache.commons.dbcp2.PoolableConnection;
import org.apache.commons.dbcp2.PoolableConnectionFactory;
import org.apache.commons.dbcp2.PoolingDriver;
import org.apache.commons.pool2.impl.GenericObjectPool;
import org.apache.commons.pool2.impl.GenericObjectPoolConfig;

public class DBCPInitListener implements ServletContextListener {

	@Override
	public void contextInitialized(ServletContextEvent sce) {
		String poolConfig = 
				sce.getServletContext().getInitParameter("poolConfig");
		Properties prop = new Properties();
		try {
			prop.load(new StringReader(poolConfig));
		} catch (IOException e) {
			throw new RuntimeException("config load fail", e);
		}
		loadJDBCDriver(prop);
		initConnectionPool(prop);
	}

	private void loadJDBCDriver(Properties prop) {
		String driverClass = prop.getProperty("jdbcdriver");
		try {
			Class.forName(driverClass);
		} catch (ClassNotFoundException ex) {
			throw new RuntimeException("fail to load JDBC Driver", ex);
		}
	}

	private void initConnectionPool(Properties prop) {
		try {
			String jdbcUrl = prop.getProperty("jdbcUrl");
			String username = prop.getProperty("dbUser");
			String pw = prop.getProperty("dbPass");

			ConnectionFactory connFactory = 
					new DriverManagerConnectionFactory(jdbcUrl, username, pw);

			PoolableConnectionFactory poolableConnFactory = 
					new PoolableConnectionFactory(connFactory, null);
			poolableConnFactory.setValidationQuery("select 1");

			GenericObjectPoolConfig poolConfig = new GenericObjectPoolConfig();
			poolConfig.setTimeBetweenEvictionRunsMillis(1000L * 60L * 5L);
			poolConfig.setTestWhileIdle(true);
			poolConfig.setMinIdle(5);
			poolConfig.setMaxTotal(50);

			GenericObjectPool<PoolableConnection> connectionPool = 
					new GenericObjectPool<>(poolableConnFactory, poolConfig);
			poolableConnFactory.setPool(connectionPool);
			
			Class.forName("org.apache.commons.dbcp2.PoolingDriver");
			PoolingDriver driver = (PoolingDriver)
				DriverManager.getDriver("jdbc:apache:commons:dbcp:");
			String poolName = prop.getProperty("poolName");
			driver.registerPool(poolName, connectionPool);
		} catch (Exception e) {
			throw new RuntimeException(e);
		}
	}

	@Override
	public void contextDestroyed(ServletContextEvent sce) {
	}

}

```


### 초기화 파라미터 관련 값 가져오기
앞서 `ServletContextListener`에서 선언한 두가지 메소드는 모두 `ServletContext`를 전달받는다고 하였다. 이 클래스는 웹 어플리케이션 컨텍스트를 구할 수 있는 `getServletContext()` 메소드를 제공한다. 

* `public ServletContext getServletContext()` 

이 메소드에서 리턴하는 `ServletContext` 객체는 `web.xml` 파일에 설정된 컨텍스트 초기화를 구하는데 사용된다. 이 초기화 파라미터는 다음과 같이 `<context-param>` 태그를 사용해 `web.xml` 파일에 설정한다. 

```
<context-param>
		<param-name>poolConfig</param-name>
		<param-value>
			jdbcdriver=com.mysql.jdbc.Driver
			jdbcUrl=jdbc:mysql://localhost:3306/guestbook?characterEncoding=utf8
			dbUser=jspexam
			dbPass=jsppw
			poolName=guestbook
		</param-value>
	</context-param>

</web-app>

```
여기서 초기화 파라미터 값을 구하는데 사용되는 `ServletContext`의 메소드는 아래와 같다 .

* `String getInitParameter(String name)` 
	* 지정한 이름을 갖는 컨텍스트 초기화 파라미터 값을 리턴한다. 존재하지 않는 경우 `null`을 리턴한다. name 파라미터는 `<param-name>`태그로 지정한 이름을 입력한다.
* `java.util.Enumeration<String> getInitParameterNames()`
	* 컨텍스트 초기화 파라미터의 이름 목록을 `Enumeration` 타입으로 가져온다.


### 리스너에서의 익셉션 처리

리스너에서는 정의된 메서드가 `throws` 구문을 이용해 명시적으로 발생시키는 예외가 없다. 이 메서드는 발생시킬 수 있는 `checkedException`을 지정하고 있지 않기 때문에 가장 최상위 예외인 `RuntimeException`이나 그 하위 타입의 예외를 사용해야 한다. 이를 지정하지 않을 경우 웹 어플리케이션 시작 자체가 실패할 수 있다.

```java
@Override
public void contextInitialized(ServletContextEvent sce) {
	String poolConfig = 
			sce.getServletContext().getInitParameter("poolConfig");
	Properties prop = new Properties();
	try {
		prop.load(new StringReader(poolConfig));
	} catch (IOException e) {
		throw new RuntimeException("config load fail", e);
	}
```

***
참고 자료 : 
1.[최범균의 JSP 2.3 웹 프로그래밍: 기초부터 중급까지 CHAPTER 20 : ServletContextListener 구현](http://www.kyobobook.co.kr/product/detailViewKor.laf?mallGb=KOR&ejkGb=KOR&barcode=9788980782802) 


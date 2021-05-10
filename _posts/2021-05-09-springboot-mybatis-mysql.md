---
layout: post
title: spring-boot에서 mybatis로 mysql 연동하는 방법
categories: [dev]
tags: [Spring boot, mybatis, mysql]
description: 스프링 부트 프로젝트에서 mybatis와 mysql 연동하는 방법
fullview: false
comments: true
---

사이드 프로젝트 [kid-safer](https://github.com/f-lab-edu/kid-safer)를 진행하는 도중, mybatis와 mysql db 연동을 위해 의존성 라이브러리를 추가하는 작업을 진행하고 있었다. 실무에서는 이미 누군가가 셋팅해놓은 것을 토대로 따라하고 개발만 진행하다가, 바닥부터 시작하려니 이것저것 찔러보는 부분이 많았다. 차후 같은 실수(?)를 일으키지 않도록 삽질한 내용을 정리해보았다.

## 프로젝트 생성
필자는 IntelliJ의 커뮤니티 버전을 사용하고 있기 때문에, [https://start.spring.io/](https://start.spring.io/)사이트를 통해 spring-boot 프로젝트를 만들어주었다. 아주 기본적인 `spring-boot-starter-web`만 라이브러리로 추가를 해주었었고, gradle로 만들어주었다.  
**mysql**과 **mybatis**를 사용하기 위해서 `build.gradle`의 `dependencies`에 아래의 라이브러리를 추가해주었다. 

```
implementation group: 'mysql', name: 'mysql-connector-java'
implementation group: 'org.mybatis.spring.boot', name: 'mybatis-spring-boot-starter', version: '2.1.4'

```

* mysql-connector-java : 
* mybatis-spring-boot-starter : 


gradle 파일을 빌드해준 후, 이상이 없는지 확인하기 위해 아래와 같은 컨트롤러를 간단하게 작성해본 뒤 서버를 구동해보았다.

```java
package com.flab.kidsafer.controller;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;

@RestController
public class SimpleController {
    @GetMapping("/hello")
    public String hello(){
        return LocalDateTime.now().format(DateTimeFormatter.ISO_LOCAL_DATE_TIME);
    }
}
```

톰캣을 실행해본 뒤, 아래와 같이 서버가 정상적으로 구동되는 것을 확인할 수 있었다.

<p style="text-align:center">

<img src="https://user-images.githubusercontent.com/75205849/117576803-2bfdf480-b122-11eb-8d2b-514d0c8fd383.png" width="300">
</p>

## MySql 연동하기
필자는 mysql과 mybatis를 동시에 연동하려고 하니, 컨트롤러 만들랴, 도메인 엔티티 설계하랴 동시에 많은 코드를 작성했어야 했고, 작성했다 치더라도 에러가 계속 군데군데 발생하니 도대체 어느 곳에서부터 문제가 발생했는지 확인하기가 어려웠다. 그래서 우선 데이터를 연결해주는 ORM인 MyBatis를 셋팅해준 후, MySql을 연동해주는 방식으로 한단계 한단계 설정하여 차근차근 문제를 해결하였다.

우선 driver 연결과 관련된 설정정보를 추가한다.   

#### 설정정보 파일 추가
`application.properties` : 

```
# mysql
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.datasource.url=jdbc:mysql://localhost:3306/localdb?serverTimeZone=UTC&CharacterEncoding=UTF-8
spring.datasource.username=스키마계정
spring.datasource.password=비밀번호
```

#### mapper 파일 추가 

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.flab.kidsafer.mapper.DbMapper">

    <!-- /* select dual */ -->
    <select id="getDual" resultType="java.lang.String">
        SELECT NOW() FROM DUAL
    </select>

</mapper>
```

#### mapper xml과 연동할 인터페이스 추가

```java
package com.flab.kidsafer.service;

import com.flab.kidsafer.mapper.DbMapper;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

@Service
public class DbService {
    @Autowired
    DbMapper dbMapper;

    public String getDual() throws Exception {
        return dbMapper.getDual();
    }
}
```

#### service 추가

```java
package com.flab.kidsafer.service;

import com.flab.kidsafer.mapper.DbMapper;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

@Service
public class DbService {
    @Autowired
    DbMapper dbMapper;

    public String getDual() throws Exception {
        return dbMapper.getDual();
    }
}
```

#### controller 추가

```java
package com.flab.kidsafer.controller;

import com.flab.kidsafer.service.DbService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.ResponseBody;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class SimpleController {
    @Autowired
    DbService  dbService;



    @GetMapping("/now")
    public @ResponseBody String now() throws Exception{
        return dbService.getDual();
    }
}
```

#### mybatis 셋팅과 관련된 환경설정 추가

```java
package com.flab.kidsafer.config;

import org.apache.ibatis.session.SqlSessionFactory;
import org.mybatis.spring.SqlSessionFactoryBean;
import org.mybatis.spring.SqlSessionTemplate;
import org.mybatis.spring.annotation.MapperScan;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.io.support.PathMatchingResourcePatternResolver;

import javax.sql.DataSource;


@Configuration
@MapperScan(basePackages = "com.flab.kidsafer.mapper")
public class DatabaseConfig {

    @Bean
    public SqlSessionFactory sqlSessionFactory(DataSource dataSource) throws Exception {
        SqlSessionFactoryBean sessionFactory = new SqlSessionFactoryBean();
        sessionFactory.setDataSource(dataSource);
        PathMatchingResourcePatternResolver resolver = new PathMatchingResourcePatternResolver();
        sessionFactory.setMapperLocations(resolver.getResources("classpath:mappers/*.xml"));
        return sessionFactory.getObject();
    }

    @Bean
    public SqlSessionTemplate sqlSessionTemplate(SqlSessionFactory sqlSessionFactory){
        return new SqlSessionTemplate(sqlSessionFactory);
    }
}
```
#### 서버 구동 후, 웹브라우저 확인

http://localhost:8080/now를 들어가보면 아래와 같이 현재 시간이 찍히며 mysql db, mybatis가 연동이 잘 된 것을 확인할 수 있다. :)

<p style="text-align:center">

<img src="https://user-images.githubusercontent.com/75205849/117664307-9bd0b580-b1dc-11eb-9ddb-43f0357ad123.png" width="300">
</p>


***
참고 자료 
1. [The server time zone value is unrecognized](https://intellij-support.jetbrains.com/hc/en-us/community/posts/360003472660-The-server-time-zone-value-is-unrecognized)






---
layout: post
title: 02. 쿼리 성능 진단은 최적화의 기초
categories: [Database]
tags: [Database, MySQL 퍼포먼스 최적화]
description: 쿼리 실행계획 플랜 관련 기초 지식
fullview: false
comments: true
---

### 쿼리 실행계획 플랜
* 성능 진단의 가장 첫 걸음은 실행한 SQL이 DB에서 어떻게 처리되는지를 파악하는 것이다.
* DBMS는 SQL 파싱(문법체크 및 실행가능한 형태로 변환) -> 옵티마이징 후, 실제로 데이터를 찾는다.
* 쿼리 실행은 DB가 데이터를 찾아가는 일련의 과정을 사람이 알아보기 쉽게 DB 결과셋으로 보여주는 것이다. 이 실행 계획을 통해 기존의 쿼리를 튜닝하고, 성능 분석과 인덱스 전략 수립등 전반적인 업무를 처리할 수 있다.

```SQL
/* MySQL용 */
EXPLAIN [EXTENDED]
SELECT ...
FROM ..
WHERE ..

--------------
/* Oracle용 */
EXPLAIN PLAN FOR
SELECT ..
FROM ..
WHERE ..
```

* 읽는 순서는 두 가지다 : 
	* 동일한 ID는 위에서 아래 방향으로 향한다.
	* 위에서 아래로 향하는 도중, table 항목에 <drived번호> 항목을 만나면, 해당 번오에 해당되는 ID로 가서 첫번째 단계를 수행한 후, 돌아와서 나머지를 수행한다.

#### 쿼리 실행 계획에서 각 항목이 의미하는 내용
* ID : Select 아이디로 Select를 구분하는 번호
* Select_type : Select에 대한 타입
* Table : 참조하는 테이블
* Type : 조회 혹은 조회 타입
* Possible_keys: 데이터 조회시 DB에서 사용할 수 있는 인덱스 리스트
* Key : 실제로 사용할 인덱스
* Key_len : 실제로 사용할 인덱스 길이
* Ref : Key 안의 인덱스와 비교하는 컬럼(상수)
* Rows : 쿼리 실행시 해당되는 행수
* Extra : 추가 정보

#### Select_type
* SIMPLE : UNION이나 서브쿼리가 없는 단순 SELECT를 의미
* PRIMARY : 서브쿼리가 있을 때, 가장 바깥쪽에 있는 SELECT다
* DERIVED : FROM절 안의 서브쿼리이다.
* DEPENDENT SUBQUERY : 외부 쿼리와 상호 연관된 서브쿼리이다.

#### Type
* system : 테이블에 단 한 개의 데이터만 있는 경우
* const : SELECT에서 PRIMARY Key 혹은 Unique Key를 상수로 조회하는 경우로, 많아야 한 건의 데이터만 있는 경우
* eq_ref : 조인시 PK 혹은 UK로 매칭하는 경우
* ref : 조인시 PK 혹은 UK가 아닌 키로 매칭하는 경우
* ref_or_nul : ref + null
* index_merge : 두 개의 인덱스가 병합되어 검색이 이루어지는 경우
* unique_subquery : IN절 안의 서브쿼리에서 PK가 오는 특수한 경우
* index_subquery : unique_subquery와 비슷하나 PK가 아닌 인덱스를 SELECT할 경우
* range : 특정 범위 내 인덱스를 사용해 원하는 데이터를 추출하는 경우 (데이터가 많지 않고 단순 SELECT시 적합)
* index : 인덱스를 처음부터 끝까지 검색하는 경우, 인덱스 풀스캔이라 부른다
* all : 테이블을 처음부터 끝까지 검색하는 경우, 테이블 풀스캔이라고 부른다

<br/>
성능은 위에서 아래로 내려올수록 좋지 않다. 다만 range의 경우 조회 건수가 많지 않다면 성능상 나쁘지 않다.

#### Extra
MySQL의 쿼리 실행에 대한 부가 정보를 보여준다. 여러 항목이 있지만, 성능과 연관된 항목은 아래 네가지가 있다 : 

* Using Index : 커버링 인덱스라고 하며 인덱스 자료 구조를 이용해 추출한다.
* Using Where : Where 조건으로 데이터를 추출한다. Type이 ALL 혹은 Index타입과 함께 표현된다면 성능이 좋지 않다는 의미이다.
* Using Filesort : 데이터 정렬이 필요한 경우로, 메모리 혹은 디스크상에서의 정렬을 모두 포함한다. 결과 데이터가 많은 경우 성능에 직접적 영향을 미친다.
* Using Temporary : 쿼리 처리시 내부적으로 TP테이블이 사용되는 경우를 의미한다.

<br/>
일반적으로 데이터가 많은 경우, Using Filrsort와 Using Temporary 상태는 좋지 않으며 반드시 쿼리 튜닝이 필요하다.

***
### 쿼리 프로파일링 이해
* 쿼리를 처리할 때 DB 내부적으로는 Open/Close Table, Optimizing, Sending Data등을 비롯해 여러 단계를 거쳐 데이터를 찾는다.
* 쿼리 프로파일리을 통해 실제 쿼리 실행ㅅ 시 병목이 되는 부분을 찾아낼 수 있다.

```SAL
SET PROFILING = 1;
SELECT * FROM tab;
SHOW PROFILE;
```

* 위 쿼리를 실행하면 가장 최근에 실행한 쿼리에 대한 프로파일 정보가 나온다.
* 프로파일링 세션 활성화 이후 실행된 쿼리 리스트를 확인하려면 `SHOW PROFILES;`로 명령어를 실행하고, 리스트를 통해 확인하고 싶은 쿼리 번호를 아래와 같이 실행하면 특정 쿼리에 대한 프로파일링 정보를 알 수 있다 : 
`SHOW PROFILE FOR QUERY 3(Query_ID 값 입력하기);`

***
참고자료 
[MySQL 퍼포먼스 최적화 - 02. 쿼리 성능 진단은 최적화의 기초](https://www.kyobobook.co.kr/product/detailViewKor.laf?mallGb=KOR&ejkGb=KOR&barcode=9788968486289)

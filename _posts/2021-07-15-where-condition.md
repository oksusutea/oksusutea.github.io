---
layout: post
title: 03.WHERE 조건 이해
wcategories: [Database]
tags: [Database, MySQL 퍼포먼스 최적화]
description: 쿼리 속도 향상을 위한 몇가지 방법들
fullview: false
comments: true
---

### 묵시적 형변환의 함정에 빠지지 말자
* 묵시적 형변환 : 조건절의 데이터 타입이 다를 때, 우선순위가 높은 타입으로 타입이 내부적으로 변환되는 것을 말한다. 정수 타입이 문자열 타입보다 우선순위가 높다.
* 쿼리 작성시 이상없이 동작은 하지만, 성능이 안좋을 때가 발생한다.  그 때에는 묵시적 형변환으로 인해 테이블을 풀스캔에 따른 현상을 방지해야 한다.
* 형변환이 일어나는 대상이 인덱스 필드라면, 조건절 처리를 위해 모든 데이터를 묵시적으로 형변환을 하기 때문에 테이블 풀스캔을 야기한다.

#### 테스트 예시


```sql
## 테이블 생성 
CREATE TABLE test (
	i int unsigned NOT NULL auto_increment,
	j int unsigned NOT NULL,
	s varchar(64) NOT NULL,
	d datetime NOT NULL,
	primary key(i)
);

## 인덱스 추가
ALTER TABLE test ADD key(j), ADD key(s), ADD key(d);

## 데이터 추가
INSERT INTO test(j,s,d)
values
( 
	crc32(rand()), 
	crc32(rand())*12345, 
	date_add(now(), interval - crc32(rand())/5 second)
);
INSERT INTO test(j,s,d)	## 여러 번 반복해서 221184건까지 만들었다.
	SELECT 
	crc32(rand()), 
	crc32(rand())*12345, 
	date_add(now(), interval - crc32(rand())/5 second)
FROM test;
```

#### 정수형 컬럼을 문자열 조건으로 검색

```SQL
EXPLAIN
SELECT * FROM test
WHERE j = '2351828558';
```

* 위와 같이 실행시 정수형이 우선순위가 높기 때문에 문자열 조건 값은 묵시적으로 정수형으로 형변환되어 인덱스를 탄다.

#### 문자열 컬럼을 정수 조건으로 검색

```SQL
EXPLAIN
SELECT * FROM test
WHERE s = 2351828558;
```

* 위와 같이 실행할 때, 묵시적 형변환이 발생해 테이블 풀스캔이 진행된다.
* 위와 같은 상황이 발생하지 않도록 주의하자

### 함수 사용을 남발하지 않기 
* 함수는 복잡한 연산을 사람이 알아보기 쉽고 사용하기 편리하지만, 잘못 쓰면 불필요한 시스템 부하를 야기한다.

```SQL
SELECT userid, count(*) AS cnt
FROM user_access_log
WHERE DATE_FORMAT(reg_date,'%Y%m%d') = '20120818'
AND DATE_FORMAT(reg_date, '%H') >= '18'
AND DATE_FORMAT(reg_date, '%H') < '21'
GROUP BY userid;
```

* 위 쿼리는 2012년 8월 18일 오후 6시~9시 사이 접속한 내역을 사용자별로 추출하는 쿼리이다.
* 위와 같이 작성하면 어떤 목적의 쿼리인지 쉽게 알아볼 수 있지만 테이블 풀스캔을 수행할 수 밖에 없다.
	* reg_date 컬럼 검색시 DATE_FORMAT 함수를 사용하면 옵티마이저는 reg_date와 연관된 데이터 분포도를 알 수 없게 된다. 
	* DATE_FORMAT 함수로 인해 변경될 결과값을 옵티마이저가 예상하지 못하기 때문이다.

```SQL
SELECT userid, count(*) AS cnt
FROM user_access_log
WHERE reg_date >='2012-08-18 18:00:00'
  AND reg_date < '2012-08-18 21:00:00'
GROUP BY userid;
```

* 위와 같이 불필요한 함수를 없애고 원하는 시간 사잇값만 가져오도록 쿼리를 수정하였다.
* 이렇게 작성하면 reg_date에 인덱스가 있거나, reg_date에 파티셔닝을 적용하면 빠르게 필요한 데이터에 접근하여 데이터를 가져올 수 있다.
* 함수를 쓰면 머릿속에 있는 내용을 쉽게 쿼리로 풀어쓸 수 있겠지만, 이를 DB가 알아듣기 쉬운 언어로 조금만 변환해준다면 불필요한 시스템 부하는 발생하지 않을 것이다.

### LIKE 검색의 적절한 사용 방법
* LIKE 검색은 '%' 문자 위치에 따라 다르게 수행된다. 그리고 '%'의 위치에 따라 컬럼에 해당 인덱스가 있더라도 의미가 없을 수도 있다. (자료는 인덱스 키 값 순서로 정렬되기 때문에, 중간 또는 뒷부분부터 검색하면 인덱스의 의미가 없어진다)


#### LIKE 'xxx%' 사용 예시

```SQL
EXPLAIN
SELECT * FROM test
WHERE s = '13111%';
```
* 앞서 만든 테스트 테이블에서 위와 같이 쿼리를 수행하면, 인덱스를 활용하는 것을 확인할 수 있다.
* LIKE로 데이터를 검색하더라도 뒷부분에 '%'를 붙이면 인덱스를 적절하게 활용할 수 있다.

```SQL
EXPLAIN
SELECT * FROM test
WHERE s = '1%';
```

* 위 케이스는 '13111%'에서 '1%'로 변경만 했을 뿐인데, 인덱스를 타지 않고 테이블 풀스캔을 한다.
* 대부분의 DBMS는 옵티마이저가 있다. 이 옵티마이저는 데이터의 분포도를 따져서 내부적으로 SQL을 최적의 방식으로 처리한다. 
* 이 예제에서 1로 시작하는 데이터의 비율이 20%로 넘어가기 때문에, 인덱스를 읽고 데이터 파일로 가는 것 보다 처음부터 전체 데이터를 읽고 필요한 데이터를 선별하는 것이 더 빠르다고 옵티마이저가 판단했기 떄문에 풀스캔을 한 것이다.
	* 인덱스는 데이터가 위치한 곳을 지칭한다. 인덱스도 데이터고 지칭하는 데이터로, 찾아가려면 실 데이터에 대한 데이터를 다시 읽어야 하기 때문에 중복된 데이터 처리 비용보다는 테이블 풀스캔으로 접근하는 것이 훨씬 빠르다고 옵티마이저가 판단한 것이다.
	* 인덱스도 데이터라는 사실을 잊지말자

#### LIKE '%xxx%' 사용 예시

```SQL
EXPLAIN
SELECT * FROM test
WHERE s = '%13111%';
```

* 인덱스는 순차적으로 비교되므로 당연히 테이블 풀스캔이 발생한다.

#### LIKE 'xxx%' 사용 예시

```SQL
EXPLAIN
SELECT * FROM test
WHERE s = '13111%';
```

* 위 예시와 동일하게 테이블 풀스캔이 발생한다.
* 물론 상용 DBMS에는 함수 결과를 인덱스로 구성하는 함수 기반 인덱스를 사용해 위와 같은 요구사항을 처리할 수 있다. 함수로 문자열을 거꾸로 만들어서, 그 결과를 인덱스로 구성하는 것이다.
* 하지만 MySQL은 이런 기능이 없다. 이러한 기능을 구현하려면 추가 인덱스 컬럼을 생성하고, 트리거로 관리하면 된다.

#### 정리
* LIKE 조건이 '검색어%'로 쓰인다면 데이터 분포도를 따져서 수행한다.
* LIKE 조건이 '%검색어'와 같은 형태로 반드시 수행되어야 한다면, LIKE 조건 이외의 조건절을 최대한 활용해서 LIKE 처리가 필요한 데이터 범위를 최대한 줄인다.
* 인덱스는 필요한 데이터를 지칭한다는 점에서 데이터에 접근하기 위한 효율적인 요소인 것은 분명하다.
* 하지만 인덱스 또한 메모리를 차지하고, 디스크를 소모하며 CPU 연산이 필요한 데이터이기 때문에, **DB에서 처리하는 데이터 범위를 최대한 줄이는 것**이 성능 최적화의 가장 기본적인 요소라는 것을 명심하자.


***
참고자료 
[MySQL 퍼포먼스 최적화 - 03. WHERE 조건 이해](https://www.kyobobook.co.kr/product/detailViewKor.laf?mallGb=KOR&ejkGb=KOR&barcode=9788968486289)

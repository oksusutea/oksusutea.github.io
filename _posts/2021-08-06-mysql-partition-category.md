---
layout: post
title: 10.3 MySQL 파티션의 종류
categories: [database]
tags: [database, mysql]
description: 레인지 파티션, 리스트 파티션, 해시 파티션, 키 파티션
fullview: false
comments: true
---

## 10.3.1 레인지 파티션
파티션 키의 연속된 범위로 파티션을 정의하는 방법으로, 가장 일반적인 바티션 방법이다. 다른 파티션 방법과 달리, `MAXVALUE` 라는 키워드를 이용해 명시되지 않은 범위의 키 값이 담긴 레코드를 저장하는 파티션을 정의할 수 있다.

### 레인지 파티션 용도
* 날짜를 기반으로 데이터가 누적되고 년도나 월 혹은 일 단위로 분석하고 삭제해야 할 때
* 범위 기반으로 데이터를 여러 파티션에 균등하게 나눌 수 있을 때
* 파티션 키 위주로 검색이 자주 실행될 때

### 레인지 파티션 생성
아래 테이블 생성 쿼리는 레인지 파티션을 이용해 사원의 입사 일자별로 파티션 테이블을 만드는 쿼리이다.

```sql
CREATE TABLE employees(
	id INT NOT NULL,
	first_name VARCHAR(30),
	last_name VARCHAR(30),
	hired DATE NOT NULL DEFAULT '1970-01-01',
	....
) ENGINE=INNODB
PARTITION BY RANGE( YEAR(hired) ) (
	PARTITION p0 VALUES LESS THAN (1991) ENGINE = INNODB,
	PARTITION p1 VALUES LESS THAN (1996) ENGINE = INNODB,
	PARTITION p1 VALUES LESS THAN (2001) ENGINE = INNODB,
	PARTITION p1 VALUES LESS THAN MAXVALUE ENGINE = INNODB
);
```

* PARTITION BY RANGE 키워드로 레인지 파티션 정의
* PARTITION BY RANGE 뒤에 컬럼 또는 내장함수를 이용해 파티션키 명시

### 레인지 파티션의 분리와 병합

#### 단순 파티션 추가

파티션을 추가하는 작업은 아래와 같이 단순하다.

```sql
ALTER TABLE employees ADD PARTITION (PARTITION p4 VALUES LESS THAN (2011));
```

하지만 테이블에 `MAXVALUE` 파티션이 이미 정이되어 있을 경우 테이블에 새로운 파티션을 추가할 수 없어 해당 파티션을 분리하는 방식으로 가야한다.


#### 단순 파티션 삭제
레인지 파티션을 사용하는 테이블에서, 파티션을 삭제하려면 `DROP PARTITION` 키워드에 삭제하려는 파티션의 이름을 지정하면 된다.

```sql
ALTER TABLE employees DROP PARTITION p0;
```

#### 기존 파티션의 분리
하나의 파티션을 두 개 이상의 파티션으로 분리하고자 할 때는 `REORGANIZE PARTITION` 명령을 사용하면 된다.

```sql
ALTER TABLE employees
	REORGANIZE PARTITION p3 INTO (
	PARTITION p3 VALUES LESS THAN (2012),
	PARTITION p4 VALUES LESS THAN MAXVALUE
);
```

이렇게 분리를 하게 되면 기존 MAXVALUE 파티션에 저장되어 있던 데이터가 파티션 키에 의해 p3, p4로 적절히 분리된다.

#### 기존 파티션의 병합
파티션 병합과정도 위와 같은 `REORGANIZE PARTITION` 명령으로 처리할 수 있다.

```sql
ALTER TABLE employees
	REORGANIZE PARTITION p2,p3 INTO (
	PARTITION p23 VALUES LESS THAN (2012)
);
```

### 레인지 파티션 주의사항
레인지 파티션에서 NULL은 어떤 값보다 작은 값으로 간주된다. 하지만 명시적으로 `VALUES LESS THAN (NULL)`은 사용할 수 없는 점 유의하자.
그 외에도, 날짜 컬럼으로 파티션 키를 설정할 경우, 가급적 아래와 같은 함수를 사용하자 : 
* YEAR
* TO_DAYS
위 두 날짜 함수는 MySQL 서버 내부적으로 파티션 프루닝 처리가 최적화되어 있어 성능상 문제가 발생하지 않기 때문이다.


## 10.3.2 리스트 파티션
리스트 파티션은 레인지 파티션과 비슷하지만, 한가지 큰 차이점이 있다. 레인지 파티션은 파티션 키 값의 연속된 값의 범위로 파티션을 구성할 수 있지만, 리스트 파티션은 파티션 키 값 하나하나를 리스트로 나열해야 한다는 점이다. 또한, 리스트 파티션에서는 레인지 파티션처럼 `MAXVALUE`로 파티션을 정의할 수 없다.(레인지가 아니니까?)

### 리스트 파티션의 용도
테이블이 아래와 같은 특성을 지닐 때는 리스트 파티션을 사용하는 것이 좋다.
* 파티션 키 값이 코드 값이거나, 카테고리와 같이 고정적일 때
* 키 값이 연속되지 않고 정렬 순서와 관계없이 파티션을 해야할 때
* 파티션 키 값을 기준으로 레코드 건수가 균일하고, 검색 조건에서 파티션 키가 자주 사용될 때

### 리스트 파티션 테이블 생성

```sql
CREATE TABLE employees(
	id INT NOT NULL,
	first_name VARCHAR(30),
	last_name VARCHAR(30),
	hired DATE NOT NULL DEFAULT '1970-01-01',
	....
) ENGINE=INNODB
PARTITION BY LIST( YEAR(hired) ) (
	PARTITION pappliance VALUES IN (3),
	PARTITION pcomputerVALUES IN (1,9),
	PARTITION psports VALUES IN (2,6,7),
	PARTITION petc VALUES IN (4,5,8,NULL),
);
```
*  PARTITION BY LIST 키워드로 생성할 파티션이 리스트 파티션이라는 것을 명시한다
* PARTITION BY LIST 키워드 뒤에 파티션 키를 정의한다.
* VALUES IN(..) 을 이용해 각 파티션별로 저장할 파티션 키 값의 목록을 나열한다.
* VALUES 내 NULL을 사용할 수 있으며, `MAXVALUE` 파티션은 정의할 수 없다.

### 리스트 파티션의 분리와 병합
파티션을 정의하는 부분에서, `VALUES LESS THAN`이 아닌 `VALUES IN`을 사용한다는 점에서 레인지 파티션의 추가, 삭제, 병합 과정과 완전히 일치한다.


### 리스트 파티션 주의사항
* 리스트 파티션은 `MAXVALUE`를 사용할 수 없다.
* 리스트 파티션은 NULL을 사용할 수 있다.
* 파티션 키는 정수와 문자열 타입(5.5 버전 이상) 을 사용할 수 있다.

## 10.3.3 해시 파티션
해시 파티션은 MySQL에서 정의한 해시 함수에 의해 레코드가 저장될 파티션을 결정하는 방법이다. 해시 파티션의 파티션 키는 항상 정수 타입의 컬럼이거나, 정수를 반환하는 표현식만 사용 가능하다. 해시 파티션의 개수는 레코드를 각 파티션에 할당하는 알고리즘과 연관되기 때문에, 파티션을 추가하거나 삭제하는 작업에는 테이블 전체적으로 레코드를 재분배해야 한다.

### 해시 파티션의 용도 
해시 파티션은 아래의 케이스에 사용하기 적합하다 : 
* 레인지 파티션이나 리스트 파티션으로 데이터를 균등하게 나누는 것이 어려울 때
* 테이블의 모든 레코드가 비슷한 사용 빈도를 보이지만, 테이블이 너무 커서 파티션을 적용해야 할 때

### 해시 파티션 테이블 생성

```sql
-- // 파티션의 개수만 지정
CREATE TABLE employees (
	id INT NOT NULL,
	first_name VARCHAR(30),
	last_name VARCHAR(30),
	hired DATE NOT NULL DEFAULT '1970-01-01',
	....
) ENGINE=INNODB
PARITION BY HASH(id)
PARTITIONS 4;

-- // 파티션의 이름을 별도로 지정
CREATE TABLE employees (
	id INT NOT NULL,
	first_name VARCHAR(30),
	last_name VARCHAR(30),
	hired DATE NOT NULL DEFAULT '1970-01-01',
	....
) ENGINE=INNODB
PARITION BY HASH(id)
PARTITIONS 4 (
	PARTITION p0 ENGINE=INNODB,
	PARTITION p1 ENGINE=INNODB,
	PARTITION p2 ENGINE=INNODB,
	PARTITION p3 ENGINE=INNODB
);
```

해시 파티션으로 테이블을 생성하는 위의 예제를 간단히 살펴보자
* PARTITION BY HASH 키워드로 파티션 종류를 해시 파티션으로 지정한다. 해당 키워드 뒤에는 파티션 키를 명시한다.
* PARTITIONS n으로 몇개의 파티션을 생성할 것인지 명시한다. 1024보다 작으면 된다.
* 두번째 생성 쿼리 예제와 같이 각 파티션의 이름을 명시할 수 있다.

### 해시 파티션의 분리와 병합
해시 파티션의 분리와 병합은 레인지 파티션, 리스트 파티션과 달리 대상 테이블의 모든 파티션에 저장된 레코드를 재분배해야하는 작업이 필요하다.(해시함수의 반환 값이 달라지기 때문이다)

#### 해시 파티션 추가
해시 파티션은 특정 파티션 키 값을 테이블의 파티션 개수로 MOD 연산한 결과 값에 의해 각 레코드가 저장될 파티션을 결정한다. 즉, 해시 파티션은 테이블에 존재하는 파티션의 개수에 의해 파티션 알고리즘이 변하는 것이다.

```sql
-- // 파티션 1개만 추가하면서 파티션 이름을 부여하는 경우
ALTER TABLE employees ADD PARTITION(PARTITION p5 ENGINE=INNODB);
-- // 동시에 6개의 파티션을 별도의 이름 없이 추가하는 경우
ALTER TABLE clients ADD PARTITION PARTITIONS 6;
```

이 작업은 전체 레코드의 파티셔닝을 다시 수행해야 하기 때문에, 부하가 많이 발생한다.

#### 해시 파티션 삭제
해시나 키 파티션은 파티션 단위로 레코드를 삭제하는 방법이 없다. MySQL 서버에서 특정 파티션을 삭제하려고 시도해도 에러를 발생시킨다.

#### 해시 파티션 분할
해시 파티션이나 키 파티션에서 특정 파티션을 두 개 이상의 파티션으로 분할하는 기능은 없으며, 전체적으로 파티션의 개수를 늘리는 것만 가능하다.

#### 해시 파티션 병합
해시나 키 파티션은 2개 이상의 파티션을 하나의 파티션으로 통합하는 기능을 제공하지 않는다.


### 해시 파티션 주의사항
* 특정 파티션만 DROP 하는 것은 불가능 하다.
* 새로운 파티션을 추가하는 작업은 단순히 파티션만 추가하는 것이 아니라, 기존의 모든 데이터의 재배치 작업이 필요하다.
* 해시 파티션은 레인지 파티션이나 리스트 파티션과는 다른 방식으로 관리하기 때문에, 해시 파티션이 용도에 적합한 해결책인지 확인이 필요하다.


## 10.3.4 키 파티션
키 파티션은 해시 파티션과 사용법과 특성이 거의 같다. 해시 파티션은 해시 값을 계산하는 방법을 파티션 키나 표현식에 사용자가 명시한다. 하지만 키 파티션은 해시 값의 계산도 MySQL 서버가 수행한다. 선정된 파티션 키 값을 Md5() 함수를 이용해 해시 값을 계산하고, 그 값을 MOD 연산하여 데이터를 각 파티션에 분배한다.

### 키 파티션의 생성

```sql
-- // PK키가 있는 경우 자동으로 PK키가 파티션 키로 사용됨
-- // PK키가 없는 경우 유니크 키(존재할 경우에만) 파티션 키로 사용됨
CREATE TABLE k1 (
	id INT NOT NULL,
	name VARCHAR(20),
	PRIMARY KEY (id)
)
-- // 괄호의 내용을 비워두면 자동으로 PK키의 모든 컬럼이 파티션 키가 됨
-- // 그렇지 않고 PK키의 일부만 명시할 수도 있음
PARTITION BY KEY()
PARTITIONS 2;

-- // PK키나 유니크 키의 컬럼 일부를 파티션 키로 명시적으로 설정
CREATE TABLE dept_emp (
	emp_no INTEGER NOT NULL,
	dept_no CHAR(4)
...
PRIMARY KEY (dept_no, emp_no)
)
-- // 괄호의 내용을 PK키나 유니크 키를 구성하는 컬럼들 중에서 일부만 선택해 파티션 키로도 설정할 수 있다.
PARTITION BY KEY(dept_no)
PARTITIONS 2;
```

키 파티션을 생성하는 위의 예제를 살펴보자.

* PARTITION BY KEY 키워드로 키 파티션을 정의한다.
* PARTITION BY KEY 키워드 뒤에 파티션 키 컬럼을 명시한다. 아무 컬럼도 명시하지 않으면 MySQL 서버가 자동으로 PK키나 유니크 키의 모든 컬럼을 파티션 키로 선택한다.
* PARTITIONS 키워드로 생성할 파티션 개수를 지정할 수 있다.
* 키 파티션은 MySQL서버가 내부적으로 MD5() 함수를 이용해 파티셔닝하기 때문에 파티션 키가 반드시 정수타입이 아니여도 된다.
* 해시 파티션에 비해 파티션 간 레코드를 더 균등하게 분할할 수 있기 때문에 키 파티션이 더 자주 사용된다.

## 10.3.5 리니어 해시 파티션/리니어 키 파티션
해시 파티션이나 키 파티션은 새로운 파티션을 추가하거나 파티션 개수를 줄일 때 대상 파티션 뿐만 아니라 테이블의 전체 파티션에 저장된 레코드의 재분배 작업이 발생한다. 이런 단점을 최소화하기 위해 **리니어 해시 파티션/리니어 키 파티션 알고리즘이 고안**되었다.

### 리니어 해시 파티션/리니어 키 파티션의 추가 및 통합
리니어 해시 파티션이나 리니어 키 파티션의 경우, 단순히 나머지 연산으로 레코드가 저장될 파티션을 결정하는 것이 아니라 *Power-of-two* 분배 방식을 사용해 파티션의 추가나 통합 과정 중 특정 파티션의 데이터에 대해서만 이동작업을 할 수 있도록 한다. 그래서 나머지 파티션의 데이터는 재분배하지 않아도 된다.

### 리니어 해시 파티션/리니어 키 파티션 관련 주의사항
리니어 파티션은 *Power-of-two* 알고리즘을 사용하기 때문에 작업의 범위를 최소화 할 수 있는 대신 각 파티션이 가지는 레코드의 건수는 일반 해시 파티션/키 파티션에 비해 덜 균등해질 수 있다. 해시 파티션이나 키 파티션으로 파티션된 테이블에 대해 새로운 파티션을 추가/삭제해야 할 요건이 많다면 리니어 해시 파티션/리니어 키 파티션을 적용해보자.

## 10.3.6 서브 파티션
다른 DBMS와 마찬가지로 MySQL 또한 서브 파티션 기능을 제공한다. 기간 단위로 레인지 파티션을 만들고, 각 레인지 파티션 내에 다시 지역별롤 시트스 서브 파티션을 구성하는 형태가 가능하다는 것이다. 하지만 최대로 사용 가능한 파티션의 개수가 1024개 뿐이므로 타 DBMS 대비 제약사항이 많다.

## 10.3.7 파티션 테이블의 실행 계획
파티션 테이블에 쿼리가 실행될 때 테이블의 모든 파티션을 읽는지, 아니면 일부만 읽는지는 성능에 아주 큰 영향을 미친다. 쿼리 실행계획이 수립될 때 불필요한 파티션은 모두 배제하고 꼭 필요한 파티션만 걸러내는 과정을 파티션 프루닝이라고 하며, 쿼리의 성능은 테이블에서 얼마나 많은 파티션을 프루닝 할 수 있는지가 관건이다.

```sql
EXPLAIN PARTITIONS
SELECT * FROM employees WHERE hired = '1995-12-10';
```
 
 위와 같은 쿼리로 쿼리의 실행계획을 확인하면 *partitions* 컬럼을 통해 파티션 프루닝을 실행했는지 확인할 수 있다. 파티션 프루닝을 사용하지 못하는 쿼리는 모든 파티션 이름이 나열될 것ㅇ다.
***
참고자료 : 
[Real MySQL - 10.3 MySQL 파티션의 종류](http://www.yes24.com/Product/Goods/6960931)

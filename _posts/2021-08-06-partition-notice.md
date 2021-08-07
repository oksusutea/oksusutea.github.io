---
layout: post
title: 10.2 주의사항
categories: [database]
tags: [database, mysql]
description: 파티션 관련 주의사항
fullview: false
comments: true
---

MySQL의 파티션은 5.1 버전부터 도입되었지만, 아직 제약사항이 많다. 그 부분에 관해 알아보자.

## 10.2.1 파티션의 제한 사항
* 숫자 값(INTEGER, INTEGER를 반환하는 함수 또는 표현식)에 의해서만 파티션 가능
* 키 파티션은 컬럼 타입 제한 없음
* 최대 1024개의 파티션을 가질 수 있음
* 스토어드 루틴이나 UDF 그리고 사용자 변수를 파티션 함수에 사용할 수 없음
* 파티션 생성 이후 sql_mode 파라미터 변경은 권장하지 않음
* 파티션 테이블에서 외래키 사용 불가
* 파티션 테이블은 전문 검색 인덱스 생성 불가
* 공간 확장 기능에서 제공되는 컬럼 타입은 파티션 테이블에서 사용 불가
* 임시 테이블은 파티션 사용 불가
* MyISAM 파티션 테이블의 경우 키 캐시 사용 불가

MySQL의 인덱스는 로컬 인덱스이며, 같은 테이블에 소속돼있는 모든 파티션은 같은 구조의 인덱스만 가질 수 있다는 점 유의하자.

## 10.2.2 파티션 사용 시 주의사항
파티션 테이블을 사용하면 PK키를 포함한 유니크 키에 대해 제약사항이 많은 것을 볼 수 있다. 파티션은 작업의 범위를 좁히기 위해서 사용하는데, 유니크 인덱스는 중복 레코드의 체크작업을 위해 사용하기 때문에 범위가 좁혀지지 않기 떄문이다.

### 파티션과 유니크 키(PK키 포함)
종류와 관계없이 테이블에 유니크 인덱스가 있으면, 파티션 키는 모든 유니크 인덱스의 일부 또는 모든 컬럼을 포함해야 한다.

```sql
CREATE TABLE tb_partition (
 fd1 INT NOT NULL, fd2 INT NOT NULL, fd3 INT NOT NULL,
 UNIQUE KEY(fd1, fd2)
) PARTITION BY HASH (fd3)
PARTITIONS 4;

CREATE TABLE tb_partition (
 fd1 INT NOT NULL, fd2 INT NOT NULL, fd3 INT NOT NULL,
 UNIQUE KEY(fd1)
 UNIQUE KEY(fd2)
) PARTITION BY HASH (fd1 + fd2)
PARTITIONS 4;

CREATE TABLE tb_partition (
 fd1 INT NOT NULL, fd2 INT NOT NULL, fd3 INT NOT NULL,
 PRIMARY KEY(fd1)
 UNIQUE KEY(fd2, fd3)
) PARTITION BY HASH (fd1 + fd2)
PARTITIONS 4;
```

위 예제는 모두 잘못된 파티션 테이블 생성방식이며, 각각의 오생성 사유는 아래와 같다 : 
* 첫 번째 : 유니크 키와 파티션 키가 전혀 연관이 없다.
* 두 번째 : fd1 유니크키와 fd2 유니크 키 만으로 파티션이 결정되지 않는다.
* 세 번째 : fd1 PK값 만으로는 파티션을 결정할 수 없으며, 유니크 키인 fd2+fd3으로도 파티션 위치를 결정할 수 없다.

이제 파티션 키로 사용할 수 있는 테이블에 대해 알아보자 : 

```sql
CREATE TABLE tb_partition (
 fd1 INT NOT NULL, fd2 INT NOT NULL, fd3 INT NOT NULL,
 UNIQUE KEY(fd1, fd2, fd3)
) PARTITION BY HASH (fd1)
PARTITIONS 4;

CREATE TABLE tb_partition (
 fd1 INT NOT NULL, fd2 INT NOT NULL, fd3 INT NOT NULL,
 UNIQUE KEY(fd1, fd2)
) PARTITION BY HASH (fd1 + fd2)
PARTITIONS 4;

CREATE TABLE tb_partition (
 fd1 INT NOT NULL, fd2 INT NOT NULL, fd3 INT NOT NULL,
 PRIMARY KEY(fd1, fd2, fd3)
 UNIQUE KEY(fd3)
) PARTITION BY HASH (fd3)
PARTITIONS 4;
```

위 예제는 각 유니크 키를 구성하는 컬럼의 값이 결정되면 해당 레코드가 어느 파티션에 저장될지 계산할 수 있다.

### 파티션과 open_files_limit 파라미터
MySQL에서는 일반적으로 테이블을 파일단위로 관리하기 때문에, MySQL 서버에서 동시에 오픈된 파일 개수가 많아질 수 있다. 이를 제한하기 위해 `open_files_limit` 시스템 변수에 동시에 오픈할 수 있는 파일의 개수를 설정할 수 있다.  
파티션도지 않은 일반 테이블은 테이블 1개당 오픈된 파일 개수가 2~3개지만, 파티션 테이블에서는 *파티션의 개수 X 2~3*개가 된다

### 파티션 테이블과 잠금
MySQL에서는 파티션 테이블이 가지는 파티션의 개수가 늘어날수록 성능이 더 떨어질 수 있다. 예를들어, 파티션이 350개 정도인 테이블의 10000건의 레코드를 INSERT하면 오히려 파티션이 없는 테이블의 INSERT가 30%가량 더 빠르다.

MySQL에서는 파티션 테이블의 잠금이 어떻게 처리되길래 위와 같은 결과가 나오는걸까? 쿼리가 실행되면, 우선 테이블을 열고 잠금을 걸어 쿼리의 최적화를 수행한다. 파티션 테이블에 쿼리가 실행되면 MySQL 서버는 테이블의 파티션 개수 관계없이 모든 파티션을 열고 잠금을 건다. 이 때문에 파티션 개수가 많아지면 많아질수록 더 느려지게 되는 것이다.

***
참고자료 : 
[Real MySQL - 10.2 주의사항](http://www.yes24.com/Product/Goods/6960931)

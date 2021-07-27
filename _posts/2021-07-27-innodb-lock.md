---
layout: post
title: 4.4 InnoDB 스토리지 엔진의 잠금
categories: [database]
tags: [database, mysql]
description: 레코드 기반 잠금 방식을 지원하는 InnoDB 스토리지 엔진 잠금
fullview: false
comments: true
---

* InnoDB 스토리지는 MySQL에서 제공하는 잠금과는 별개로, 스토리지 엔진 내부에서 레코드 기반 잠금 방식을 탑재하고 있다. 이로 인해 MyISAM보다는 훨씬 뛰어난 동시성 처리를 제공할 수 있다. 
* MySQL 5.1부터 InnoDB 플러그인 스토리지 엔진이 도입되면서부터 InnoDB의 트랜잭션과 잠금, 그리고 잠금 대기중인 트랜잭션의 목로을 조회할 수 있는 방법이 도입되었다.( `INFORMATION_SCHEMA` 데이터베이스에 `INNODB_TRX`, `INNODB_LOCKS` , `INNODB_LOCK_WAITS` 테이블을 조인해 확인 가능)

## 4.4.1 InnoDB의 잠금 방식
InnoDB의 잠금 방식은 크게 두가지가 있으며, 낙관적 잠금(Optimistic locking)이나 비관적 잠금(Pessimistic locking)이 있다.

#### 비관적 잠금(Pessimistic locking)
* 현 트랜잭션에서 변경하고자 하는 레코드에 잠금을 획득하고, 변경 작업을 처리하는 방식이다.
* 현재 트랜잭션에서 변경하고자 하는 레코드를 다른 트랜잭션에서도 변경할 수 있다고 가정하기 때문에 먼저 잠금을 획득한 후에 변경 작업을 수행한다.
* 일반적으로 높은 동시성 처리에는 비관적 잠금이 유리하다고 알려져있으며, InnoDB는 비관적 잠금 방식을 채택하고 있다.

#### 낙관적 잠금(Optimistic locking)
* 기본적으로 각 트랜잭션이 같은 레코드를 변경할 가능성은 희박하다는 가정하에 진행한다.
* 우선 변경 작업을 수행하고, 마지막에 잠금 충돌이 있는지 확인해 문제가 있을 경우 ROLLBACK 처리한다.

## 4.4.2 InnoDB의 잠금 종류
* InnoDB 스토리지 엔진은 레코드 기반 잠금을 제공하며, 잠금 정보가 타 스토리지 엔진 대비 작은 공간으로 관리된다.
* 그렇기 때문에 레코드 락이 페이지 락 또는 테이블 락으로 레벨업 되는 경우는 없다.
* InnoDB 스토리지 엔진에서는 타 DBMS와는 다르게 레코드 락 뿐만 아니라 레코드와 레코드 사이 간격을 잠그는 GAP락도 존재한다.

<br/>

* 레코드 락 : 레코드 자체를 잠그는 락이며, 다른 DBMDS의 레코드 락과 동일한 역할을 한다. **InnoDB 스토리지 엔진은 레코드 자체가 아니라 인덱스 레코드를 잠근다.**
* 갭 락 : 레코드 그 자체가 아니라 레코드와 바로 인접한 레코드 사이의 간격만을 잠그는 것을 의미한다.(InnoDB에만 있는 락), 레코드와 레코드 사이 간격에 새로운 레코드가 생성(INSERT) 되는 것을 제어하는 역할을 한다.
* 넥스트 키 락 : 레코드락과 갭 락을 합쳐놓은 형태의 잠금을 의미한다.
* 자동 증가 락 : AUTO_INCREMENT 컬럼이 사용된 테이블에 동시에 여러 레코드가 INSERT될 경우, 저장된 순서대로 일련 번호 값을 가지기 위해 제공하는 락(테이블 수준)이다.
	* INSERT, REPLACE같이 새로운 레코드를 저장하는 쿼리에서만 필요하다.
	* AUTO_INCREMENT 락은 명시적으로 획득하고 해제하는 방법은 없으며, 아주 짧은 시간동안만 걸렸다가 해제되기 때문에 대부분의 경우 문제가 되지 않는다.

## 4.4.3 인덱스와 잠금
InnoDB의 잠금은 레코드를 잠그는 것이 아니라 인덱스를 잠그는 방식으로 처리한다. 즉, 변경해야 할 레코드를 찾기 위해 검색한 인덱스의 레코드를 모두 잠가야 한다.

```SQL
-- // employees에는 first_name만 컬럼만 멤버로 담긴 ix_firstname이라는 인덱스가 존재한다.
SELECT COUNT(*) FROM employees WHERE first_name = 'Georgi'; // 253건

SELECT COUNT(*) FROM employees WHERE first_name = 'Georgi ' AND last_name = 'Klassen'; // 1건

UPDATE employees SET hire_date = NOW() WHERE first_name = 'Georgi' AND last_name = 'Klassen';
```

* `ix_firstname` 인덱스만 존재할 때, 위와 같은 UPDATE 쿼리문을 실행하면 253건의 레코드에 락이 걸린다.
* 인덱스가 없을 때는 테이블을 풀 스캔하면서 UPDATE 작업을 하는데, 이 과정에서 테이블에 있는 모든 레코드를 잠그게 된다.

***
참고자료 : 
[Real MySQL - 4.4 InnoDB 스토리지 엔진의 잠금](http://www.yes24.com/Product/Goods/6960931)

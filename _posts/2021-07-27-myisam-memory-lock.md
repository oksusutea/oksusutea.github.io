---
layout: post
title: 4.3 MyISAM과 MEMORY 스토리지 엔진의 잠금
categories: [database]
tags: [database, mysql]
description: MySQL 서버 엔진의 잠금
fullview: false
comments: true
---

MyISAM이나 MEMORY 스토리지 엔진은 자체적으로 잠금을 갖고있지 않고, MySQL 엔진에서 제공하는 테이블 락을 그대로 사용한다.
MyISAM이나 MEMORY 스토리지 엔진에서는 쿼리 단위로 필요한 잠금을 한꺼번에 모두 요청해서 획득하기 때문에 데드락이 발생할 수 있다.

## 4.3.1 잠금 획득
* 읽기 잠금 : 테이블에 쓰기 잠금이 걸려져 있지 않으면 바로 읽기 잠금을 획득하고 읽기 작업을 시작할 수 있다.
* 쓰기 잠금 : 테이블에 아무런 잠금이 걸려 있지 않아야만 쓰기 잠금을 획득할 수 있고, 그렇지 않다면 다른 잠금이 해제될 때까지 대기해야 한다.

## 4.3.2 잠금 튜닝

```sql
SHOW STATUS LIKE 'Table%';
```

* 테이블 락에 대한 작업 상황은 MySQL의 상태 변수를 통해 확인할 수 있다.
* 위 쿼리에서 실행한 결과 중, `Table_locks_immediate`는 다른 잠금이 풀리기를 기다리지 않고 바로 잠금을 획득한 횟수고, `Table_locks_waited`는 다른 잠금이 이미 해당 테이블을 사용하고 있어 기다려야 했던 횟수를 누적해서 저장하고 있다.
* 잠금 대기 쿼리 비율 = `Table_locks_immediate` / (`Table_locks_immediate` + `Table_locks_waited`) * 100 이며, 이 수치가 높을 경우 테이블 잠금으로 인한 경합이 많이 발생한다는 것을 의미한다.
	* 이 경우 InnoDB 스토리지 엔진으로 변환하는 방안으로 고려해보자.


***
참고자료 : 
[Real MySQL - 4.3 MyISAM과 MEMORY 스토리지 엔진의 잠금](http://www.yes24.com/Product/Goods/6960931)
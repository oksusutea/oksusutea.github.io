---
layout: post
title: 트랜잭션의 특징 그리고 격리수준
categories: [Database]
tags: [Database]
description: 트랜잭션 격리 수준 및 격리 수준 선택에 따른 발생 가능 이슈
fullview: false
comments: true
---

## 트랜잭션과 그 특성
트랜잭션은 DB작업시 더이상 나누어지지 않는 최소한의 단위를 말한다. 트랜잭션은 아래 4가지 원칙을 보장해야 한다. 

* 원자성(Atomicity) :
	* 트랜잭션은 더이상 분해가 불가능한 업무의 취소 단위이므로, 전부 처리되거나 아예 하나도 처리되지 않아야 한다.
* 일관성(Consistency) : 
	* 트랜잭션을 완료하고 나면 그 데이터베이스는 여전히 일관적인 상태여야 한다. 
* 격리성(Isolation) : 
	* 실행중인 트랜잭션의 중간 결과를 다른 트랜잭션이 접근할 수 없다. 
* 영속성 (Durability) : 
	* 트랜잭션이 성공적으로 완료되면 그 결과는 데이터베이스에 영속적으로 저장된다. 

	
## 트랜잭션의 격리성
하지만, 실제로 **ACID 원칙은 종종 지켜지지 않는다.** 이를 지킬경우 **동시성이 매우 떨어지기 때문**이다.  
그렇기 때문에 DB엔진은 **트랜잭션의 격리수준**을 선택해 동시성을 얻을 수 있는 방법을 제공한다. Isolation 레벨이 낮을수록 문제가 발생할 가능성은 커지지만 동시성은 더 높아진다. 그리고 Isolation 레벨이 높을수록 더 빡빡하게 lock을 건다. 이제 이 설정 레벨에 대해 알아보도록 하자. 

*** 

## InnoDB의 lock
InnoDB는 트랜잭션의 ACID원칙과 동시성을 최대한 보장하기 위해 다양한 종류의 Lock을 사용한다. 그 중, 트랜잭션의 격리수준을 이해하는데에 필요한 Lock을 알아보자.

### Row-level lock
가장 기본적인 Lock은 **테이블의 row마다 걸리는 row-level lock**이다. 여기에서는 크게 **Shared Lock과 Exclusive Lock**이 있다. 

#### Shared Lock (S lock)
**read에 대한 lock**이다. 일반적인 `SELECT`쿼리는 lock을 사용하지 않고 DB를 읽어온다. 하지만 `SELECT ... FOR SHARE`등 일부 `SELECT`쿼리는 read 작업 수행시 InnoDB가 각 row에 S lock을 건다.

#### Exclusive Lock(X lock)
**write에 대한 lock**이다. `SELECT ... FOR UPDATE`나 `UPDATE`, `DELETE`등의 수정 쿼리를 날릴 때 각 row에 걸리는 Lock이다.

S lock과 X lock에 대한 규칙은 아래와 같다 : 

* 여러 트랜잭션이 동시에 한 row에 S lock을 걸 수 있다. 즉, 여러 트랜잭션이 동시에 한 row를 읽을 수 있다.
* S lock이 걸려있는 row에 다른 트랜잭션이 X lock을 걸 수 없다. 즉, 다른 트랜잭션이 읽고 있는 row를 수정하거나 삭제할 수 없다.
* X lock이 걸려있는 row에는 다른 트랜잭션이 S lock과 X lock을 걸 수 없다. 즉, 다른 트랜잭션이 수정하거나 삭제하는 row는 읽기, 수정, 삭제가 전부 불가능하다. 

요약하자면, **S lock을 사용하는 쿼리끼리는 같은 row에 접근 가능**하다. 반면, **X lock이 걸린 row는 다른 어떠한 쿼리도 접근 불가능하다. 락의 이름인 "Shared"와 "Exclusive"의 의미와 정확히 일치한다.

### Record lock

Record lock은 row가 아니라 **DB의 index Record에 걸리는 lock**이다. 여기도 row-level과 마찬가지로 `S lock`과 `X lock`이 있다.

Record lock의 예시를 들어본다. `c1`이라는 column을 가진 테이블 `t`가 있다고 하자. 이때 한 transaction에서 아래와 같은 쿼리를 실행했다.

```sql
Query 1 in transaction A
SELECT c1 FROM t where c1 = 10 FOR UPDATE;
```
이 경우 `t.c1=10`인 index에 x lock이 걸린다. 이 때, 다른 트랜잭션에서 아래와 같은 쿼리를 실행한다고 가정해보자.

```sql
Query 2 in transaction B
DELETE FROM t where c1 = 10;
```
이 때, query 2는 우선 `t.c1=10`인 index record에 x lock을 걸려고 시도한다. 하지만 이미 해당 index record에는 x lock이 걸려져 있는 상태다. 따라서 query 2는 트랜잭션 A가 끝날때까지 삭제할 수 없다. 

### Gap lock
Gap lock은 **DB index record의 gap에 걸리는 lock이다. 여기서 gap이란 index 중 실제 DB에 record가 없는 부분**이다.
gap lock은 gap에 걸리는 lock이다. 즉, gap lock은 해당 gap에 접근하려는 다른 쿼리의 접근을 막는다. Record lock이 해당 index를 타려는 다른 쿼리의 접근을 막는 것과 동일하다. 둘의 차이점이라면, **record lock은 이미 존재하는 row가 변경되지 않도록 보호**하는 반면, **gap lock은 조건에 해당하는 새로운 row가 추가되는 것을 방지하기 위함**이다.

```sql
Query 1 in transaction A
SELECT c1 FROM t WHERE c1 BETWEEN 10 and 20 FOR UPDATE;
```
t 테이블에는 `c1=13`, `c1=15`라는 값이 있다고 가정해보자. 위 쿼리를 실행하게 되면 `10 <= c1 <=12`, `c1 = 14`, `16 <= c1 <= 20`에 해당하는 gap이 lock이 걸리게 된다.


**Gap은 하나의 index일 수도, 여러 index일 수도, 혹은 아예 아무 값도 없을 수 있다.**

### Lock이 해제되는 타이밍
트랜잭션이 진행되는 동안, InnoDB 엔진은 위에 언급한 것처럼 실행되는 쿼리에 맞는 수많은 lock을 DB에 걸게 된다. 이러한 **lock은 모두 트랜잭션이 commit되거나 rollback 될 때 함께 unlock된다.**


***

### 격리성으로 인해 발생할 수 있는 문제점


#### Dirty Read

<p style="text-align:center">
<img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fbgjwus%2FbtqKbV7Q7go%2FDwXX4Bgjl39Gu1FO78SkJk%2Fimg.png" width="300px">
</p>

**Dirty Read는 트랜잭션 작업이 완료되지 않았는데도 다른 트랜잭션에서 조회가 가능한 현상**을 말한다. 만약 Transaction 1이 정상처리되지 않고 Rollback된다면 Transcation 2는 잘못된 값을 읽은 것이나 마찬가지이다.

<br/>

#### Non-Repeatable Read

<p style="text-align:center">
<img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FOAHDX%2FbtqJ8fZ5yJr%2FAKYMiEi3XQ7m71RasvPXF1%2Fimg.png" width="300px">
</p>

**한 트랜잭션 내에서 같은 Key를 가진 row를 두 번 읽었는데, 그 사이에 값이 변경되거나 삭제되어 결과가 다르게 나타나는 현상**을 말한다.

<br/>

#### Phantom-read

<p style="text-align:center">
<img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FxunmZ%2FbtqJ0pJFsmi%2F2KhKRBkksKwyIOWbhaeVBk%2Fimg.png" width="300px">
</p>

**한 트랜잭션 내 같은 쿼리를 두 번 수행했는데, 첫번째 쿼리에 없던 유령(Phantom) 레코드가 두 번째 쿼리에서 나타나는 현상**을 말한다. 앞서 말한 **Non-Repetable Read는 한 건에 대한 데이터 값이 변경되는 것을 말하며, Phantom Read는 다 건의 데이터 요청시 값이 변경되는 것**을 말한다.


### 트랜잭션의 격리수준
동시에 여러 트랜잭션이 처리될 때, 트랜잭션끼리 얼마나 고립되어 있는지를 나타내는 것이다. 간단하게 말하자면, 특정 트랜잭션이 다른 트랜잭션에 변경한 데이터를 볼 수 있도록 허용할지 말지를 결정하는 것이다.  
격리수준은 크게 4가지로 구분된다.  

* Read Uncommited
* Read Commited
* Repeatable read
* Serializable


아래로 내려갈수록 트랜잭션간 고립 정도가 높아지며, 성능이 떨어지는 것이 일반적이다.  
일반적인 서비스단에는 Read Commited(오라클)나 Repeatable Read(mysql)중 하나를 사용한다.



#### Read Uncommited

* SELECT 문장이 수행되는 동안 해당 데이터에 Shared Lock이 걸리지 않는다. 
* 각 트랜잭션에서의 변경 내용이 `COMMIT`이나 `ROLLBACK`여부에 관계 없이 다른 트랜잭션에서 값을 읽을 수 있다.
* 데이터 수정시 트랜잭션 완료될때 까지 Lock을 사용한다.
* `DIRTY READ` 현상이 발생할 수 있다.
	* 트랜잭션이 완료되지 않았는데도 다른 트랜잭션에서 볼 수 있게 되는 현상

#### Read Commited
* **커밋된 데이터만 조회할 수 있도록 하는 수준의 격리수준을 보장하는 level이다.**
* SELECT 문장이 수행되는 동안 해당 데이터에 Shared Lock이 걸린다. 
* 실제 테이블 값을 가져오는 것이 아니라 **Undo 영역에 백업된 레코드**에서 값을 가져온다. => **read operation때마다 DB snaption을 다시 뜬다.**
* `DIRTY READ`현상은 해결하였지만`NON-REPEATABLE READ` 현상이 발생할 수 있다.
	* 하나의 트랜잭션에서 같은 데이터를 두 번 이상 읽을 때, 값이 다 다른 현상
* 예시 : 여러 트랜잭션에서 입금/출금 처리가 진행되는 트랜잭션이 있고, 잔고 총 합을 보여주는 트랜잭션이 있을 경우 총합을 계산하는 `SELECT`쿼리는 실행될 때마다 다른 값을 가져온다.
* InnoDB에서는 record lock만 사용하고, gap lock은 사용하지 않는다.


#### Repeatable Read
* **반복해서 read operation을 수행하더라도 읽어 들이는 값이 변화하지 않는 정도의 격리 수준을 보장하는 level이다.**
* 트랜잭션이 완료될 때까지 SELECT 문장이 사용하는 모든 데이터에 Shared Lock이 걸린다.
* 트랜잭션 ID를 부여하여 본인 트랜잭션 ID보다 작은 트랜잭션 번호에서 변경한 것만 읽게 된다.(즉 트랜잭션 시작되기 전 커밋된 결과를 참조한다.)
* 데이터 변경시 Undo 공간에 백업해두고 실제 레코드 값을 변경한다.
* `UPDATE`한 데이터에 대해 정합성을 보장하지만, `INSERT`, `DELETE`는 보장되지 않는다. 이로 인해 `Phantom Read`문제가 발생한다.
	* 마치 유령을 보는 것처럼 데이터가 사라지거나, 없던 데이터가 생기는 현상을 말한다.
* 이러한 변경방식은 MVCC(Multi Version Concurrency Control)이라고 한다.
* InnoDB에서는 gap lock을 사용한다.


#### Serializable

* 트랜잭션이 완료될 때까지 SELECT 문장이 사용하는 모든 데이터에 Shared Lock이 걸린다.
* 트랜잭션 내 쿼리를 두 번 이상 수행할 때, 첫번째 쿼리에 있던 레코드가 사라지거나 값이 바뀌지 않음은 물론이고, 새로운 레코드가 나타나지 않도록 하는 설정방식이다.(트랜잭션에서 읽고 쓰는 레코드는 다른 트랜잭션에서 절대 접근 불가)
* 단순 읽기 작업시에도 `공유잠금`을 설정하고, 이러면 동시에 다른 트랜잭션에서 레코드를 변경하지 못하게 된다. 즉 동시성이 낮아지게 된다.

***
참고 자료 및 이미지 출처 : 	

1. [Lock으로 이해하는 Transaction의 Isolation Level](https://suhwan.dev/2019/06/09/transaction-isolation-level-and-lock/)
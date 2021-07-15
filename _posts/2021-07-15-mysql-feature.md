---
layout: post
title: 01. MySQL의 특징
categories: [Database]
tags: [Database, MySQL 퍼포먼스 최적화]
description: MySQL의 구조와 주요 특징
fullview: false
comments: true
---

### MySQL의 전체적인 구조
* MySQL은 **서버 엔진**과 **스로티지 엔진** 두가지로 나뉘어져 있다.

#### 서버 엔진
* 클라이언트의 요청을 받아 SQL을 처리하는 DB자체의 기능적인 역할 담당
* 사용자가 쿼리를 날렸을 때, DB가 SQL을 이해할 수 있도록 쿼리를 재구성하는 **쿼리 파싱**과 디스크나 메모리 같은 물리적인 저장장치와 통신하는 스토리지 엔진에 데이터를 요청하는 업무를 담당한다.
* 스토리지 엔진에서 받아온 데이터를 사용자 요청에 맞게 처리하거나 접근 제어, 쿼리 캐시, 옵티마이저 등의 역할을 수행한다.
* 물리적인 저장장치와 직접적으로 통신하는 역할을 제외하고, 사용자와 MySQL 사이에 발생하는 데이터 처리 프로세스의 대부분을 담당한다.

#### 스토리지 엔진
* 서버 엔진이 필요한 데이터를 물리 장치에서 가져오는 역할
* MySQL은 다른 DBMS와는 다르게 스토리지 엔진이 **플러그인 방식**으로 동작한다.
* MySQL Community 버전 기준 초기 설치시 제공되는 스토리지 엔진은 다음과 같다 : 
	* InnoDB
	* MRG_MYISAM
	* CSV
	* FEDERATED
	* MyISAM
	* BLACKHOLE
	* MEMORY
	* ARCHIVE

### MySQL에서 스토리지 엔진이란 무엇인가?
* MySQL은 다야한 스토리지 엔진을 제공한다.
* 스토리지 엔진별 특성을 이해하고 적재적소에 활용할 수 있다면 강력한 성능 뿐만 아니라, 장비 효율성도 높일 수 있다. 반대로, 서비스 특성에 맞지 않는 스토리지 엔진을 사용하면 예기치 않은 문제가 발생할 수 있다.
* 아래 쿼리를 통해 현재 사용 중인 스토리지 엔진을 확인할 수 있다.
* 
```sql
SHOW ENGINES;
```

#### MyISAM 스토리지 엔진
* MySQL에서 가장 오래된 스토리지 엔진이다.
* 파일 기반 스토리지 엔진이며, 인덱스만 메모리에 올려 처리한다.
* 데이터는 메모리에 적재하지 않고, 디스크에서 바로 접근한다.
* 트랜잭션을 지원하지 않고, 테이블 단위 잠금(Table Level Lock)으로 데이터 변경을 처리하기 때문에, 특정 테이블을 여러 세션에서 변경하면 성능이 저하된다. (특정 세션 데이터 변경이 끝나기 전까지 다른 세션은 대기하는 상태)
* 텍스트 전문으로 검색할 수 있는 풀텍스 인덱싱(Fulltext Indexing)과 지리 정보를 처리할 수 있는 지오메트릭 스파셜 인덱싱(Geometric Spatial Indexing)과 같은 기능도 제공한다. (해당 내용 사용시, 테이블 파티셔닝이 불가능하다)
* 로그 수집 또는 단순 SELECT에는 적합하지만, 동시 데이터 처리에는 한계가 있다.
#### InnoDB 스토리지 엔진
* MySQL에서 유일하게 트랜잭션을 지원하는 스토리지 엔진이다.
* 일반적인 서비스에서 가장 많이 사용된다.
* 다중 버전 동시성 제어 메커니즘(Multiversion Concurrency Controll, MVCC)을 제공하고, 행 단위 잠금(Row Level Lock)으로 데이터 변경 작업을 수행한다.
* 인덱스와 데이터 모두 메모리에 올리기 때문에 메모리 버퍼 크기(InnoDB_Buffer_Pool_Size)가 DB 성능에 큰 영향을 끼친다.
* 프리픽스 인덱스 압축 기법을 사용한다.
* PK는 **클러스터 인덱스(인덱스 순서로 데이터가 저장되는 구조)**로 구성된다. => PK 순서로 디스크에 저장되어 있는 것을 의미
* 만약 PK가 정의되어 있지 않다면, Unique Key가 클러스터 인덱스로 구성되고, PK와 UK가 테이블 내 정의되어 있지 않다면 내부적으로 6바이트 크기의 키를 만들어 PK로 사용한다(사용자 확인 불가)

#### Archive 스토리지 엔진
* 로그 수집에 적합한 스토리지 엔진이다.
* 데이터가 메모리상에서 압축되고, 압축된 상태로 디스크에 저장되어 행 단위 잠금이 가능하다.
* 한 번 INSERT된 데이터는 UPDATE나 DELETE를 할 수 없고 인덱스를 지원하지 않는다.
* 트랜잭션이 크게 중요하지 않은 통계나 로그 분석 서비스의 기초 데이터 수집에 적합한 스토리지 엔진이다.

### MySQL의 데이터 처리 방식

#### MySQL은 모든 SQL을 단일 코어에서 처리한다.
* MySQL은 SQL을 병렬처리하지 않는다.(물론 써드파티 스토리지 엔진을 통해 병렬 처리를 할 수는 있다)
* CPU 코어 개수를 늘리는 Scale-Out 보다는 단위 처리량이 좋은 CPU로 Scale-Up 하는 것이 훨씬 좋다.

#### MySQL은 테이블 조인을 Nested Loop Join 알고리즘으로만 처리한다.

* Nested Loop Join : 선행 테이블에서 만족하는 행을 찾은 뒤, 후행 테이블을 loop로 돌려 조건에 만족하는 행을 찾는 방식, 이중 루프 구문과 비슷하다고 볼 수 있다.

아래 코드는 Nested Loop Join의 sudo 코드이다 :

```
for each row in t1 matching range {
	for each row in t2  matching reference key {
		for each row in t3 {
			if row satisifies join conditions,
			send to client
		}
	}
}
```

* 선행 테이블 t1을 범위 스캔으로 조인할 데이터를 추출한다.
* t2에서 인덱스를 통해 연관된 데이터를 추출한다.
* 적당한 인덱스가 없는 경우 t3을 통해 테이블 풀스캔을 해 모든 데이터를 일일이 체크한다.
* 위 과정에서 적합한 데이터를 가져와서 클라이언트에게 전달한다.

* 이렇게 매번 테이블을 access 하는 방식을 조금이라도 개선하기 위해서, MySQL에서는 Blocked Nested Loop Join(조인 버퍼를 이용)으로 처리한다.
* 테이블 조인 시, 필요한 데이터를 메모리에 일시적으로 저장해 효율적으로 데이터에 접근하는 방식이다.
* 

```
for each row in t1 matching range {
	for each row in t2 matching reference key {
		store used columns FROM t1, t2 injoin buffer
		if buffer is full {
			for each row in t3 {
				for each t1, t2 combination in join buffer {
					if row satisfies join conditions,
					send data to client
				}
			}
			empty buffer;
		}
	}
}
if buffer is not empty {
	for each row in t3 {
		for each t1, t2 combination in join buffer {
			if row satisfies join conditions,
			send data to client
		}
	}
}
```

* 선행 테이블 t1에서 조인할 데이터를 찾는다.
* t1에서 조건에 만족하는 데이터를 찾아 조인 버퍼가 가득찰 때까지 채운다.
* 조인 버퍼가 가득 채워지면 후행 테이블의 데이터를 스캔하며 조인 버퍼에 있는 데이터와 매칭되는지 체크한다.
	* 데이터 일치시, 조인 결과 값으로 내보낸다.
* 조인 버퍼 안의 모든 데이터를 비교하는 첫번째 과정이 완료되면, 조인 버퍼를 비우고 t1에서 조건에 만족하는 데이터를 이어서 채운다. 이 과정은 조인 버퍼에 더이상 데이터를 채울 수 없는 시점, 즉 선행 테이블이 조건에 해당되는 데이터를 모두 처리할 때까지 반복해서 수행된다.
* BNLJ에서 후행 테이블을 스캔하는 횟수는 **조인 버퍼에 데이터가 적재되는 횟수**와 동일하다.


***
### 추가 지식
#### 프리픽스 인덱스 압축 기법
* 인덱스의 첫 번째 값을 통째로 저장하고, 그 이후 변경 값을 차례로 인덱싱 하는 방식이다.
* 프리픽스 인덱스 압축 사용시 키 사이즈는 대폭 줄어들고, 메모리 효율은 높아진다.
* 하지만 키 역순으로 데이터를 찾아야 할 경우에는 성능 이슈가 발생할 수 있다.( order by key desc는 지양하자!)

#### Table Level Lock & Row Level Lock
* 이 둘의 차이는 데이터 변경시 어느 수준까지 락을 걸어 다른 사용자가 데이터를 변경하지 못하게 살 것인지를 의미한다.
* Table Level Lock에서 특정 사용자가 데이터 입력/수정/삭제 작업을 수행하면 해당 테이블 전체에 잠금이 걸려 다른 사용자는 대기해야 한다. 따라서 테이블 변경 작업을 동시에 하면 성능이 상당히 저하된다.
* Roe Level Lock은 변경하고자 하는 Row에만 락을 걸어 해당 Row를 제외한 다른 사용자의 데이터 변경 작업에 영향을 미치지 않는다. Table Level Lock과는 다르게 테이블을 동시에 변경할 수 있다.

***
참고자료 
[MySQL 퍼포먼스 최적화 - 01. MySQL의 특징](https://www.kyobobook.co.kr/product/detailViewKor.laf?mallGb=KOR&ejkGb=KOR&barcode=9788968486289)

---
layout: post
title: 3.3 MyISAM 스토리지 엔진 아키텍처
categories: [database]
tags: [database, mysql]
description: MySQL 서버의 전반적인 구조
fullview: false
comments: true
---

* MyISAM 스토리지 엔진의 성능에 영향을 미치는 요소는 **키 캐시**와 **운영체제의 캐시/버퍼**가 있다.


## 3.3.1 키 캐시
* InnoDB의 버퍼 풀과 비슷한 역할을 하는 것이 MyISAM의 키 캐시이다.
* 인덱스만 대상으로 동작하며, 인덱스의 디스크 쓰기 작업에 대해서만 부분적으로 버퍼링 역할을 한다.
* 키 캐시 히트율 : 100 - (Key_reads / Key_read_requests * 100) 
	* Key_reads  : 인덱스를 디스크에서 읽어들인 횟수
	* Key_read_requests : 키 캐시로부터 인덱스를 읽은 횟수
	* `SHOW GLOBL STATUS LIKE 'Key%'` 를 이용해 확인할 수 있다.
* 일반적으로 키 캐시 히트율을 99% 이상으로 유지하라고 권장한다. 만일 99% 미만일 경우, 키 캐시를 더 크게 설정하는 것이 좋다.
* 키 캐시는 `key_buffer_size` 라는 파라미터를 통해 사이즈를 조절할 수 있다.

```sql
key_buffer_size = 4GB
kbuf_board.key_buffer_size = 2GB
kbuf_comment.key_buffer_size = 2GB
```

위와 같이 설정해두면 디폴트 키캐시 4GB와 *kbuf_board* 키 캐시 2GB, *kbuf_comment* 키 캐시 2GB가 할당된다. 하지만, 특정 테이블의 인덱스가 해당 키 캐시에 저장되도록 지정해주어야 한다. 그렇지 않으면, 계속 디폴트 키 캐시에만 저장된다.

```SQL
CACHE INDEX board IN kbuf_board;
CACHE INDEX comment IN kbuf_comment;
```


## 3.3.2 운영체제의 캐시 및 버퍼
* MyISAM 테이블의 인덱스는 키 캐시를 이용해 디스크를 검색하지 않고도 충분히 빠르게 검색할 수 있다.
* 하지만, MyISAM 테이블의 데이터는 디스크로부터 I/O를 해결해줄만한 캐시나 버퍼링 기능이 없다. 그래서 데이터 읽기, 쓰기 작업은 항상 OS의 디스크 읽기/쓰기 작업으로 요청된다.
* MyISAM이 주로 사용되는 MySQL에서는 일반적으로 키캐시는 최대 물리 메모리의 40% 이상을 넘기지 않게 설정하고, 나머지 공간은 운영체제가 사용하기 위한 공간으로 마련해줄 수 있도록 하자.

***
참고자료 : 
[Real MySQL -3.3 MyISAM 스토리지 엔진 아키텍처](http://www.yes24.com/Product/Goods/6960931)

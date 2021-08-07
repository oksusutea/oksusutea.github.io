---
layout: post
title: 7.10 쿼리 성능 테스트
categories: [database]
tags: [database, mysql]
description: 작성한 쿼리의 효율성을 판단하는 방법
fullview: false
comments: true
---


## 7.10.1 쿼리의 성능에 영향을 미치는 요소
직접 작성한 쿼리를 실행해보고, 성능을 판단할 때 가장 큰 변수는 MySQL 서버가 가지고 있는 여러 종류의 버퍼나 캐시일 것이다. 이 버퍼나 캐시가 영향을 어떻게 미치는지 보고, 최소화하는 방법을 알아보자.

### 운영체제의 캐시
MySQL 서버는 운영체제의 파일 시스템 관련 기능(시스템 콜)을 이용해 데이터 파일을 읽어온다. 그런데 일반적으로 대부분의 운영체제는 한 번 읽은 데이터는 운영체제가 관리하는 별도의 캐시 영역에 보관해 두었다가 다시 해당 데이터가 요청되면 디스크를 읽지 않고 캐시의 내용을 바로 MySQL 서버로 반환한다.
InnoDB 스토리지 엔진은 일반적으로 파일 시스템의 캐시나 버퍼를 거치지 않는 Direct I/O를 사용하므로 운영체제의 캐시가 그다지 큰 영향을 미치지 않는다. 하지만 MyISAM 스토리지 엔진은 운영체제의 캐시에 대한 의존도가 높기 때문에, 운영체제의 캐시에 따라 성능 차이가 큰 편이다.  
운영체제에 포함된 캐시나 버퍼는 프로그램 단위로 관리되기 때문에, 프로그램(MySQL 서버)가 종료되면 해당 프로그램에서 사용되는 캐시는 자동으로 해제된다.

### MySQL 서버의 버퍼 풀(InnoDB 버퍼 풀과 MyISAM의 키 캐시)
운영체제의 버퍼나 캐시와 마찬가지로 MySQL 서버에서도 데이터 파일의 내용을 페이지(또는 블록) 단위로 캐시하는 기능을 제공한다. InnoDB 스토리지 엔진이 관리하는 캐시는 버퍼 풀이라고 하고, MyISAM이 관리하는 캐시는 키 캐시라고 한다. InnoDB의 버퍼 풀은 인덱스 페이지는 물론이고 데이터 페이지까지 캐시하며, 쓰기 작업을 위한 버퍼링 작업까지 겸해서 처리한다. 그와 달리 MyISAM의 키 캐시는 인덱스 데이터에 대해서만 캐시 키능을 제공한다. 그리고 MyISAM의 키 캐시는 주로 읽기를 위한 캐시 역할을 수행한다. 
MySQL이 한 번 기동된 상태에서는 InnoDB의 버퍼 풀과 MyISAM의 키 캐시를 변경하거나 강제 퍼기하는 명령이 없기 때문에, 변경을 하려면 서버를 재기동 해야한다.

### MySQL 쿼리 캐시
쿼리 캐시는 이전에 실행했던 SQL 문장과 임시로 저장해두는 메모리 공간을 의미한다. 한 번 실행한 쿼리는 전체 실행과정을 건너뛰기 때문에 실제 부하와 관계없이 아주 빠르게 처리될 수 있다. 쿼리 캐시에 저장된 데이터를 비우려면 "RESET QUERY CACHE"를 이용하면 된다.
"RESET QUERY CACHE" 명령은 MySQL 섭어 ㅔ포함된 모든 쿼리 캐시 내용을 삭제하며, 삭제 작업이 진행되는 동안 모든 쿼리가 대기상태로 된다. 그렇기 때문에 이 명령을 사용할 때에는 주의하도록 하자.

### 독립된 MySQL 서버
버퍼와 캐시 관련 부분은 아니지만, MySQL 서버가 기동중인 장비에 웹 서버나 다른 배치용 프로그램이 실행되고 있다면, 테스트하려는 쿼리의 성능이 영향을 받는다.  이와 마찬가지로, MySQL 서버 뿐 아니라 테스트 쿼리를 실행하는 클라이언트 프로그램이나 네트워크의 영향요소도 고려해보자.

<br/>
실제 쿼리의 성능테스트는 MySQL 서버의 상태가 워밍업된 상태(캐시 혹은 버퍼가 준비된 상태)에서 진행할지, 혹은 콜드 상태(캐시 혹은 버퍼가 초기화된 상태)에서 진행할지도 고려해야 한다. 일반적으로는 워밍업된 상태를 가정하고 테스트하는 편이다.
운영체제의 캐시나 MySQL의 버퍼 풀, 키 캐시는 그 크기가 제한적이라서 쿼리에서 필요로 하는 데이터나 인덱스 페이지보다 크기가 작으면 플러시 작업과 캐시 작업이 반복되서 발생하므로 쿼리를 1번 실행해서 나온 결과를 그대로 신뢰하기 어렵다. 테스트 하려는 쿼리를 6-7번 실행해서 초기 값은 버리고, 나머지 값들의 평균치를 기준으로 성능을 측정하자.

결국 쿼리의 성능을 비교하는 것은 상대적인 비교이지 절대적인 성능이 아니다. 그래서 그 쿼리가 어떤 서버에서도 그 시간 내에 처리된다고 보장할 수는 없다.(실제 MySQL 서버에서는 그 쿼리말고도 다른 쿼리가 실행중이며, 리소스를 얻기 위해 경합이 발생할 수 있기 떄문이다)

## 7.10.2 쿼리의 성능 테스트
이제 직접 쿼리 문장의 성능을 비교해보자. 

```SQL
SELECT SQL_NO_CACHE STRAIGHT_JOIN
 e.first_name, d.dept_name
FROM employees e, dept_emp de, departments d
WHERE e.emp_no = de.emp_no AND d.dept_no = de.dept_no;

SELECT SQL_NO_CACHE STRAIGHT_JOIN
 e.first_name, d.dept_name
FROM departments d,dept_emp de, employees e
WHERE e.emp_no = de.emp_no AND d.dept_no = de.dept_no;
```

이 쿼리를 실제로 실행하면 30만건의 레코드가 화면에 출력된다. 다 출력될 때까지 기다리면 테스트 해보기도 전에 지쳐버릴 것이다. 아래와 같이 쿼리를 조금 고쳐 실행해보자.

```SQL
SELECT SQL_NO_CACHE COUNT(*) FROM
(SELECT STRAIGHT_JOIN
 e.first_name, d.dept_name
FROM employees e, dept_emp de, departments d
WHERE e.emp_no = de.emp_no AND d.dept_no = de.dept_no
) dt;

SELECT SQL_NO_CACHE COUNT(*) FROM 
(SELECT STRAIGHT_JOIN
 e.first_name, d.dept_name
FROM departments d,dept_emp de, employees e
WHERE e.emp_no = de.emp_no AND d.dept_no = de.dept_no
) dt;
```

위와 같이 기존 쿼리를 서브쿼리로 만들고, 카운팅 값을 계산하도록 변경하였다. 이렇게 변경하면 임시 테이블을 사용하기 때문에 일부 오차가 발생할 수 있다. 이 방법 말고 아래와 같이도 작성할 수 있다.

```SQL
SELECT SAL_CALC_FOUND_ROWS SQL_NO_CACHE STRAIGHT_JOIN
 e.first_name, d.dept_name
FROM employees e, dept_emp de, departments d
WHERE e.emp_no = de.emp_no AND d.dept_no = de.dept_no
LIMIT 0;

SELECT SAL_CALC_FOUND_ROWS SQL_NO_CACHE STRAIGHT_JOIN
 e.first_name, d.dept_name
FROM departments d,dept_emp de, employees e
WHERE e.emp_no = de.emp_no AND d.dept_no = de.dept_no
LIMIT 0;
```

위 쿼리는 `LIMIT 0`과 `SAL_CALC_FOUND_ROWS`를 함께 사용했다. `LIMIT 0` 조건으로 화면에 출력되는 레코드는 하나도 없지만, 실제로 MySQL 서버에서는 `SAL_CALC_FOUND_ROWS` 힌트로 인해 끝까지 처리하므로 쿼리의 전체적인 처리 시간을 확인할 수 이다.


물론 크게 중요한 요소는 아니지만, 위 쿼리들은 모두 결과 데이터를 클라이언트로 가져오지 않으므로 네트워크 통신만큼은 부하가 줄어들 수 있다.

## 7.10.3 쿼리 프로파일링
MySQL에서 쿼리가 처리되는 동안 각 단계별 작업에 시간이 얼마나 걸렸는지 확인할 수 있다면 쿼리의 성능을 예측하거나 개선하는데 많은 도움이 될 것이다. 이를 위해 MySQL은 프로파일링 기능을 제공한다.

```sql
mysql> show variables like  '%profiling%';

+------------------------+-------+

| Variable_name | Value |

+------------------------+-------+

| profiling | OFF |

+------------------------+-------+

mysql> SET PROFILING=1;

mysql> show variables like  '%profiling%';

+------------------------+-------+

| Variable_name | Value |

+------------------------+-------+

| profiling | ON|

+------------------------+-------+
```

위 명령을 실행함으로서 프로파일링 기능이 활성화되었다. 이를 통해 각 쿼리에 대한 프로파일 내용을 확인할 수 있다.


분석된 쿼리의 목록을 확인하려면 `SHOW PROFILES (CPU or MEMORY) (QUERY <쿼리번호>);`를 사용하면 된다.  프로파일링 정보는 모든 쿼리에 대해 저장되는 것이 아니라 최근 15개의 쿼리에 대해서만 저장된다.(이 값은 `profiling_history_size` 시스템 설정 변수를 조정하면 된다)

그리고 각 쿼리의 프로파일링 정보 가운데 CPU나 MEMORY 또는 DISK와 관련된 내용만 구분해서 확인할 수도 있다. 

```sql
mysql> SHOW PROFILE CPU FOR QUERY 2;
```

***
참고자료 : 
[Real MySQL - 7.10 쿼리 성능 테스트](http://www.yes24.com/Product/Goods/6960931)

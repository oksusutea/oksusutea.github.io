---
layout: post
title: 11.2 스토어드 프로그램의 문법
categories: [database]
tags: [database, mysql]
description:  스토어드 프로그램 사용 문법
fullview: false
comments: true
---

프로그래밍 언어에서 사용하는 함수처럼, 스토어드 프로그램도 헤더 부분과 본문 부분으로 나눌 수 있다. 헤더 부분은 정의부라 하며, 스토어드 프로그램의 이름과 입출력 값을 명시하는 부분이다. 그 외 보안이나 스토어드 프로그램의 작동 방식과 관련된 옵션도 명시할 수 있다. 본문 부분은 스토어드 프로그램의 바디라고도 하며, 스토어드 프로그램이 실행하는 내용을 작성하는 부분이다.  
이번 챕터에서는 스토어드 프로그램의 종류별로 헤더의 정의부를 어떻게 작성하는지, 그리고 바디에서 사용할 수 있는 제어문이나 반복문, 그리고 커서와 관련된 기능도 함께 살펴볼 예정이다.

## 11.2.1 예제 테스트시 주의사항
이번 챕터를 통해 예제를 테스트할 때, 스택의 공간이 부족하거나 MySQL의 독특한 문법으로 인해 에러가 발생할 수 있다. 그래서 테스트를 직접 실행하기 전에 설정을 조정해주어야 한다.

* 스토어드 프로그램 실행시 프로시저나 함수의 이름과 파라미터를 입력하는 괄호 사이 공백을 제거하기
* ERROR 1436 발생시 MySQL의 스레드가 사용하는 스택의 크기를 늘려주기
```sql
thread_stack = 512K
```

## 11.2.2 스토어드 프로시저
스토어드 프로시저는 서로 데이터를 주고받아야 하는 여러 쿼리를 하나의 그룹으로 묶어 독립적으로 실행하기 위해 사용하는 것이다. 배치 프로그램에서 첫 번째 쿼리의 결과를 이용해 두 번째 쿼리를 실행해야 할 때를 대표적인 예로 볼 수 있다. 이렇게 각 쿼리가 연관되어 데이터를 주고받으면서 반복적으로 실행되어야 할 때 프로시저를 사용하면 MySQL 서버와 클라이언트 간의 네트워크 전송 작업을 최소화하고 수행 시간을 줄일 수 있다.  
프로시저는 반드시 독립적으로 호출돼야 하며, SELECT나 UPDATE와 같은 SQL문장에서 프로시저를 참조할 수 없다.

### 스토어드 프로시저 생성 및 삭제
프로시저는 `CREATED PROCEDURE` 명령으로 생성할 수 있으며, 아래 형태로 구성된 프로시저 예제를 살펴보자.

```sql
CREATE PROCEDURE sp_sum (IN param1 INTEGER, IN param2 INTEGER, OUT param3 INTEGER)
BEGIN
	SET param3 = param1 + param2;
END;
```

이 프로시저의 이름은 sp_sum이며, param1, param2 그리고 param3이라는 파라미터를 필요로 한다.  
BEGIN부터 END까지는 스토어드 프로시저의 본문에 속하며, param1과 param2를 더한 값은 param3으로 넣는다는 것을 의미한다.
스토어드 프로시저 생성시 아래 사항을 주의하자 : 
* 스토어드 프로시저는 기본 반환값이 없다. 즉, 스토어드 프로시저 내부에서는 값을 반환하는 RETURN 명령을 사용할 수 없다.
* 스토어드 프로시저의 각 파라미터는 아래의 세가지 특성중 하나를 지닌다.
	* IN 타입 : 입력 전용 파라미터를 의미한다. 외부에서 스토어드 프로그램 호출시 프로시저에 값을 저장하는 용도로 사용하고, 값을 반환하는 용도로 사용하지 않는다.
	* OUT 타입 : 출력 전용 파라미터다. 프로시저 외부에서 프로시저를 호출할 때 값을 전달하는 용도로 사용하는 것이 아니라, 프로시저 실행이 완료되면 외부 호출자로 값을 전달하는 용도로 사용한다.
	* INOUT : 입력 및 출력 용도로 모두 사용할 수 있다.

일반적으로 MySQL 쿼리는 `;`가 쿼리의 끝을 의미하지만, 스토어드 프로그램은 본문 내부에 무수히 많은 `;`를 포함하기 때문에  CREATE로 시작한 스토어드 프로그램의 끝을 판별할 수 있는 별도의 문자열을 구분자로 설정해야 한다.  
명령의 끝을 알려주는 종료문자를 변경하는 명령어는 `DELEMITER`이다. 스토어드 프로그램 생성시에는 보통 `;;`, `//`와 같이 연속된 2개의 문자열을 종료문자로 설정한다. 

```sql
-- // 종료문자를 ;;로 변경
DELEMETER ;;
CREATE PROCEDURE sp_sum (IN param1 INTEGER, IN param2 INTEGER, OUT param3 INTEGER)
BEGIN
	SET param3 = param1 + param2;
END;;

-- // 스토어드 프로그램 생성이 완료되면 다시 종료 문자를 기본 문자로 복구
DELEMETER ;
```

보통 종료문자가 변경되면, 스토어드 프로그램 생성 명령 뿐만 아니라 일반적인 SELECT, INSERT와 같은 명령에도 `;;`를 사용해야 한다. 이같은 경우 실수가 자주 발생할 수 있으므로 다시 종료 문자를 원복해주자.  

그 외에도 스토어드 프로시저 변경 시  `ALTER PROCEDURE` 명령을 사용하고, 삭제시에는 `DROP PROCEDURE` 명령을 사용하면 된다. 참고로 스토어드 프로시저의 파라미터나 처리 내용을 변경할 때는 `ALTER`를 사용하지 못하기 때문에, `DROP` 후 프로시저를 재생성 해주어야 한다.

### 스토어드 프로시저 실행
스토어드 프로시저와 스토어드 함수의 가장 큰 차이점은 프로그램을 **실행하는 방법**이다. 스토어드 프로시저는 SELECT쿼리에 실행될 수 없으며, 반드시 `CALL` 명령어로 실행해야 한다.

```sql
SET @result:=0;
SELECT @result; // - 0

CALL sp_sum(1,2,@result);
SELECT @result; // - 3
``` 

프로시저 사용시 첫 번째 파라미터와 두 번째 파라미터는 IN타입이므로 다시 돌려받을 필요가 없어 리터럴 형식으로 전달했지만, 세 번째 파라미터는 OUT 파라미터로 그 결과 값을 넘겨받을 수 있어야 한다. 그래서 INOUT타입이나 OUT타입의 파라미터는 MySQL의 세션변수를 이용해야 한다.

### 스토어드 프로시저의 커서 반환
MySQL 5.5 버전 이하에서는 스토어드 프로그램 내부에서만 커서를 사용할 수 있다. 하지만 스토어드 프로시저 내 커서를 오픈하지 않거나 SELECT 쿼리의 결과 셋을 페치하지 않으면, 해당 쿼리의 결과 셋은 클라이언트에 바로 전달된다.

```sql
CREATE PROCEDURE sp_selectEmployees (IN in_empno INTEGER)
BEGIN
	SELECT * FROM employees WHERE emp_no=in_empno;
END;;

CALL sp_selectEmployees(10001);;
```

위와 같이 실행하면, SELECT 쿼리의 결과 셋을 별도로 반환하는 OUT 변수에 담거나 화면에 출력하는 처리도 하지 않았는데, 쿼리의 결과가 클라이언트로 전송될 것이다.  
보통 스토어드 프로시저에서 쿼리의 결과 셋을 클라이언트로 전송하는 기능은 스토어드 프로시저의 디버깅 용도로 자주 사용된다. MySQL 프로시저는 메시지를 화면에 출력하는 기능을 제공하지 않으며, 별도의 로그 파일에 기록하는 기능도 없다. 그래서 가끔 스토어드 프로시거자 잘못돼 디버깅하려 해도 어느 부분이 잘못됐고 원인이 무엇인지 찾아내기 쉽지 않다. 이 때 스토어드 프로시저 내 단순히 SELECT 쿼리만 사용하면 결과를 화면상에 확인할 수 있어 변수를 트래킹하거나 상태 변화 여부를 확인할 수 있다.

## 11.2.3 스토어드 함수
스토어드 함수는 하나의 SQL문장으로 작성이 불가능한 기능을 하나의 SQL 문장으로 구현해야 할 때 사용한다. SQL 문장과 관계없이 별도로 실행되는 기능이라면 프로시저를 이용하는 것이 좋고, 그게 아닐 경우 함수를 이용해 구현하는 것이 좋다. 함수가 프로시저 대비 제약사항이 더 많기 떄문이다.

### 스토어드 함수 생성 및 삭제
스토어드 함수는 `CREATE FUNCTION` 명령으로 생성할 수 있으며, 모든 파라미터는 읽기 전용이라 형식을 지정할 수 없다. 그리고 스토어드 함수는 반드시 정의부에 RETUNRS 키워드를 통해 값의 타입을 명시해야 한다. 아래 예시를 확인해보자.

```sql
CREATE FUNCTION sf_sum(param1 INTEGER, param2 INTEGER)
	RETURNS INTEGER
BEGIN
	DECLARE param3 INTEGER DEFAULT 0;
	SET param3 = param1 + param2;
	RETURN param3;
END;; 
```

스토어드 함수가 프로시저 대비 다른 점은 아래 두가지다.
* 함수 정의부에 `RETURNS`로 반환되는 값의 타입을 명시해야 한다.
* 함수 본문 마지막에 정의부에 지정된 타입과 동일한 타입의 값을 `RETURNS`명령으로 반환해야 한다.

스토어드 프로시저와 달리 스토어드 프로그램의 본문에서는 아래와 같은 항목을 사용하지 못한다.
* PREPARE, EXECUTE 명령을 이용한 프리페어 스테이먼트 사용 불가
* 명시적 또는 묵시적 ROLLBACK/COMMIT을 유발하는 SQL 문장 사용 불가
* 재귀 호출 사용 불가
* 스토어드 함수 내 프로시저 호출 불가
* 결과 셋을 반환하는 SQL 문장 사용 불가

그리고 프로시저와 마찬가지로 `ALTER FUNCTION` 명령으로 입력 파라미터나 처리 내용을 변경할 수 없고, 스토어드 함수의 특성만 변경할 수 있다. 함수 내부 내용을 변경하려면 `DROP` 후 `CREATE` 해주자.

### 스토어드 함수 실행
스토어드 함수는 프로시저와 달리 CALL 명령으로 실행하지 않고, SELECT 문장을 이용해 실행한다.

```sql
SELECT sf_sum(1,2) as sum;
```

## 11.2.4 트리거
트리거는 테이블의 레코드가 저장되거나, 변경될 때 미리 정의해둔 작업을 자동으로 실행해주는 스토어드 프로그램이다. 이름 그대로 데이터의 변화가 생길 때, 다른 작업을 기동시켜주는 방아쇠라고 볼 수 있다.
MySQL의 트리거는 테이블에 대해서만 생성할 수 있고 특정 테이블에 트리거를 생성하면 해당 테이블에 발생하는 조작에 대해 지정된 시점에 트리거의 루틴을 실행하게 된다. 그리고 테이블당 하나의 이벤트에 대해 두개 이상의 트리거를 등록할 수 없다.

### 트리거 생성
트리거는 `CREATE TRIGGER` 명령으로 생성한다. 스토어드 프로시저와 함수와는 달리 BEFORE나 AFTER 키워드로 트리거가 실행될 이벤트를 명시할 수 있다. 그리고 트리거 정의부 끝에서 `FOR EACH ROW`, `FOR EACH STATEMENT`라는 키워드로 레코드나 SQL 문장 단위로 트리거가 실행되도록 명시해준다.

```sql
CREATE TRIGGER on_delete BEFORE DELETE ON employees
	FOR EACH ROW
BEGIN
	DELETE FROM salaries WHERE emp_no = old.emp_no;
END ;;
```

* 트리거 이름 뒤에는 언제 트리거를 실행할지 명시해준다.(본 예시에서는 `BEFORE DELETE` 로 명시해주었다)
* 테이블 명 뒤에는 트리거가 실행될 단위를 명시해준다. (5버전대 이상부터는 `FOR EACH ROW`만 지원한다)

위 예제 트리거는 employees 테이블의 레코드를 삭제하는 쿼리가 실행되면, 해당 레코드가 삭제되기 전에 on_delete라는 트리거가 실행되고, 트리거가 완료된 이후 테이블의 레코드가 삭제된다.
트리거를 사용하려면 각 SQL 문장이 어떤 이벤트를 발생시키는지 명확하게 알고 있어야 한다.
* INSERT : BEFORE INSERT -> AFTER INSERT
* LOAD DATA : BEFORE INSERT -> AFTER INSERT
* REPLACE : 
	* 중복 없을 경우 : BEFORE INSERT -> AFTER INSERT
	* 중복 있을 경우 : BEFORE DELETE -> AFTER DELETE -> BEFORE INSERT -> AFTER INSERT
* INSERT INTO ON DUPLICATE :
	* 중복 없을 경우 : BEFORE INSERT -> AFTER INSERT
	* 중복 있을 경우 : BEFORE UPDATE -> AFTER UPDATE
* UPDATE : BEFORE UPDATE -> AFTER UPDATE
* DELETE : BEFORE DELETE -> AFTER DELETE
* TRUNCATE : 이벤트 발생 안함
* DROP TABLE : 이벤트 발생 안함

트리거의 BEGIN..END 블록에서는 NEW 또는 OLD라는 특별한 객체를 사용할 수 있다. OLD는 해당 테이블에서 변경이 가해지기 전 레코드를 지칭하는 키워드이며, NEW는 변경이 가해진 이후의 레코드를 지칭할 때 사용한다.

MySQL에서 다른 스토어드 프로그램은 모두 mysql DB의 proc 테이블에 저장되지만, 트리거는 해당 데이터베이스의 데이터 파일이 저장된 디스크의 디렉터리에 *.TRG라는 파일로 생성된다. 만약 물리적인 복사로 데이터 파일을 다른 MySQL서버로 복사하거나 백업할 때에는 이 파일도 같이 복사해주자. 또한 RENAME TABLE 명령으로 데이터베이스 사이 테이블을 이동할 때 트리거가 포함된 테이블은 에러를 발생시키기 때문에, 이 명령을 수행할 때에는 트리거를 삭재하고 명령을 실행해주는 것이 좋다.

### 트리거 실행
트리거는 스토어드 프로시저나 함수처럼 작동을 확인하기 위해 명시적으로 실행해 볼 수 있는 방법이 없다. 트리거가 등록된 테이블에 직접 INSERT, UPDATE, DELETE를 수행해 확인하는 방법밖에 없다.

## 11.2.5 이벤트
주어진 특정한 시간에 스토어드 프로그램을 실행할 수 있는 스케줄러 기능을 이벤트라고 한다. MySQL 5.1.6 버전부터 사용할 수 있으며, 특별히 이벤트의 스케줄링만 담당하는 스레드를 활성해야 사용할 수 있다. 이 스레드를 기동하려면 MySQL 서버의 설정 파일에서 `event_scheduler`라는 시스템 변수를 ON 또는 1로 설정해야 한다.
MySQL의 이벤트는 별도로 실행 이력에 대한 정보를 보관하지 않고 가장 최근에 실행된 정보만 `INFORMATION_SCHEMA.EVENTS` 테이블에 기록해둔다. 실행 이력이 필요한 경우 별도로 테이블을 만들고 이벤트 처리 로직에서 직접 기록하는 것이 좋다.

### 이벤트 생성
이벤트는 반복 실행 여부에 따라 일회성 이벤트와 반복성 이벤트로 나눠볼 수 있다.

* 반복성 이벤트 :  아래 예제는 2011년 1월 1일 밤 12시 정각부터 하루 단위로 2011년 말까지 반복해서 실행하는 daily_ranking 이벤트를 생성하는 예제이다.

```sql
CREATE EVENT daily_ranking
	ON SCHEDULE EVERY 1 DAY STARTS '2011-05-16 01:00:00' ENDS '2011-12-31 12:59:59'
DO
	INSERT INTO daily_rank_log VALUES (NOW(), 'Done')
```

이벤트가 5/18 18시에 생성되었다면 19일부터 매일 똑같은 시간에 반복되서 실행될 것이다. 이벤트 생성 명령은 일 뿐만이 아니라 연, 분기, 월, 시간 등등 여러 반복주기를 생성할 수 있다.

* 일회성 이벤트 : 단 한 번만 실행되는 일회성 이벤트를 등록하려면 `EVERY` 대신 `AT`을 명시하면 된다. 여기서는 정확한 시간을 명시할 수도 있고, 현 시점부터 1시간뒤와 같은 상대 시간도 명시할 수 있다.

```sql
CREATE EVENT etime_jobno
	ON SCHEDULE AT CURRENT_TIMESTAMP + INTERVAL 1 HOUR
DO
	INSERT INTO daily_rank_log VALUES (NOW(), 'Done')
```

위 명령어는 이벤트 생성 시점부터 1시간 뒤에 단 한 번 실행되는 이벤트이다.

반복성이나 일회성이냐 관계 없이 이벤트의 처리 내용을 작성하는 DO절은 여러 가지 방식으로 사용할 수 있다. 단순히 하나의 쿼리나 프로시저를 호출하는 명령을 실행할 수도 있고, BEGIN...END로 구성되는 복합 절을 사용할 수 있다. 그리고 이벤트의 반복성 여부에 관계없이 `ON COMPLETION` 절을 이용해 완전히 종료된 이벤트를 삭제할지, 그대로 유지할지 선택할 수 있다. 기본적으로 완전히 종료된 이벤트(더이상 실행될 필요가 없는 이벤트)는 자동적으로 삭제된다.

### 이벤트 실행 및 결과 확인
트리거와 동일하게 명시적으로 실행할 수 있는 방법은 없다. 그래서 우선 이벤트를 등록하고, 스케줄링 시점을 임의로 설정해 실행되는 내용을 확인해보아야 한다.

참고로 이벤트의 스케줄링 정보나 최종 실행 시간 정보는 mySQL DB의 event 테이블을 통해 조회하거나, INFORMATION_SCHEMA.events라는 테이블을 통해 조회할 수 있는데, 이 두가지 테이블의 타임존이 다르기 때문에 그 부분을 유의해서 보아야 한다.
	* mysql.event : UTC
	* INFROAMTAION_SCHEMA.events : ETZ(이벤트 타임존으로, 해당 이벤트를 생성 또는 변경하는 문장을 실행했던 커넥션의 타임존)

## 11.2.6 스토어드 프로그램 본문(Body) 작성
지금까지 각 스토어드 프로그램을 생성하는 방법과 실행하는 방법을 살표보았다. 위에서 언급한 네가지 스토어드 프로그램(프로시저, 함수, 트리거, 이벤트)는 생성하고 실행하는 방법에 조금씩 차이가 있지만, 각 스토어드 프로그램의 본문 부분의 문법은 동일하다. 이제 본문 부분에 작성될때 사용되는 문법에 대해 알아보도록 한다.


### BEGIN ... END 블록과 트랜잭션
스토어드 프로그램의 본문은 BEGIN으로 시작해서 END로 끝나며, 하나의 BEGIN ... END 블록은 또 다른 여러 개의 BEGIN ... END 블록을 중첩해서 포함할 수 있다.  
BEGIN .. END 블록에서 주의해야 할 것은 트랜잭션 처리이며, 트랜잭션을 시작하는 명령은 아래 두가지가 있다 : 
* BEGIN
* START TRANSACTION
하지만 BEGIN .. END 블록 내에서 사용된 *BEGIN* 명령은 모두 트랜잭션의 시작이 아니라, 바디 부분의 시작으로 해석하기에, 스토어드 프로그램 본문에서 트랜잭션을 시작할 때는 *START TRANSACTION* 명령을 사용해야 한다.

### 프로시저 내부에서 트랜잭션 완료
프로시저 내부에서 COMMIT이나 ROLLBACK 명령으로 트랜잭션을 완료하면, 프로시저 외부에서 COMMIT, ROLLBACK을 해도 의미가 없다.

### 프로시저 외부에서 트랜잭션 완료

```sql
CREATE TABLE tb_hello (name VARCHAR(100), message VARHAR(100)) ENGINE=InnoDB;
CREATE PROCEDURE sp_hello ( IN name VARCHAR(50) )
BEGIN
	INSERT INTO tb_hello VALUES (name, CONCAT('Hello ', name));
END ;;
```
위와 같이 프로시저를 생성하고, 쿼리로 트랜잭션 시작 후에 해당 프로시저를 실행해보자.

```sql
START TRANSACTION;
CALL sp_hello('First');
COMMIT;
SELECT * FROM tb_hello;	// -- Hello First가 INSERT됨

START TRANSACTION;
CALL sp_hello('Second');
ROLLBACK;
SELECT * FROM tb_hello;	// -- Hello First만보임
```

이와 같이 스토어드 프로시저 외부의 트랜잭션 상태에 따라 테이블 레코드 값이 달라진다.

### 변수
스토어드 프로그램의 BEGIN ... END 블록 사이 사용하는 변수는 사용자 변수와는 다르므로 혼동하지 말자. 여기서 언급되는 변수는 블록 안에서만 사용가능하며, 보통 스토어드 프로그램 변수 또는 로컬 변수라고 한다.
로컬 변수는 DECLARE 명령으로 정의되고, 반드시 타입이 함께 명시돼야 한다. 로컬 변수에 값을 할당하는 방법은 SET 명령 또는 SELECT .. INTO .. 문장으로 가능하다.

#### DECLARE : 로컬 변수를 정의하는 명령
`DECLARE v_name VARCHAR(50) DEFAULT 'Lee';`와 같이 선언할 수 있으며, 초기 디폴트 값을 명시하지 않으면 NULL로 초기화된다. 타입도 함께 명시해주자.

#### SET : 변수에 값을 할당하는 명령어
`SET v_name = 'Kim', v_email = 'kim@email.com';`

#### SELECT .... INTO : SELECT한 컬럼 값을 로컬 변수에 할당하는 명령으로, 반드시 1개의 레코드를 반환하는 레코드여야 한다.

```
SELECT emp_no, first_name, last_name INTO v_empno, v_firstname, v_lastname
FROM employees WHERE emp_no = 100001
LIMIT 1;
```

스토어드 프로그램의 바디 블록에서는 입력 파라미터와 로컬변수, 그리고 테이벌 컬럼명이 같은 이름을 가질 수 있다. 세 가지 변수가 같을 때에는 아래와 같은 우선순위를 지닌다 : 
* DECLARE로 정의한 로컬 변수
* 스토어드 프로그램의 입력 파라미터
* 테이블 컬럼

```sql
CREATE PROCEDURE sp_hello ( IN first_name VARCHAR(50) )
BEGIN
	DECLARE first_name VARCHAR(50) DEFAULT 'Kim';
	SELECT CONCAT('Hello ', first_name) FROM employees LIMIT 1;
END ;;
```

```sql
CALL sp_hello('Lee');; // - Hello Kim이라는 결과가 나옴
```

각 변수의 우선순위에 따라 프로시저의 호출 결과는 "Kim"이 된다.

스토어드 프로그램이 복잡해지면 각 변수가 어떤 변수인지 혼동스러워질 때가 많다. 그에 따라 구분을 위해 입력 파라미터(p_) 로컬 변수(v_)로 구분을 할 수 있다.

### 제어문
BEGIN ... END 블록 내에서만 사용할 수 있는 제어문은 아래와 같다 : 

#### IF ... ELSEIF ... ELSE ... END IF
#### CASE WHEN ... THEN ELSE ... END CASE (SWITCH랑 비슷)
#### 반복 루프 (LOOP, REPEAT, WHILE)
* LOOP : 별도의 반복 조건 명시 불가(LEAVE로 BREAK처리)
* REPEAT , WHILE : 반복 조건 명시 가능 (REPEAT은 먼저 본문처리, WHILE은 먼저 반복 조건 체크)

### 핸들러와 컨디션을 이용한 에러 핸들링
안정적이고 견고한 스토어드 프로그램을 작성하기 위해서는 반드시 핸들러를 이용해야 한다. 핸들러가 없는 스토어드 프로그램은 try ~ catch 없이 작성한 프로그램과 같다.
핸들러는 이미 정의한 컨디션 또는 사용자가 정의한 컨디션을 어떻게 처리(핸들링)할지 정의하는 기능이다. 컨디션은 SQL 문장의 처리 상태에 대해 별명을 붙이는 것과 같은 역할을 수행한다. 주로 가독성을 높이기 위해 사용된다. 

#### 핸들러
스토어드 프로그램 또한 다른 프로그래밍 언어와 같이 여러 가지 에러나 예외 상황에 대한 핸들링이 필수적이다. 여기서는 MySQL 스토어드 프로그램에서의 핸들러 처리 중에도 예외에 대한 핸들러를 정의하고 그 핸들러가 어떻게 작동하는지를 살펴보고자 한다.

```sql
DECLARE handler_type HANDLER
	FOR condition_value [, condition_value] ....
	handler_statements
```

* 핸들러 타입이 CONTINUE로 정의되면 handler_statements를 실행하고 스토어드 프로그램의 마지막 실행 지점으로 돌아가 나머지 코드를 실행한다.
* 핸들러 타입이 EXIT으로 정의되었다면 정의된 handler_statements를 실행하고 이 핸들러가 정의된 BEGIN ... END 블록을 벗어난다. handler_statements 부분에서는 반드시 함수의 반환 타입에 맞는 적절한 값을 반환하는 코드가 포함되어 있어야 한다.
* 핸들러 정의 문장에서 condition_value는 아래와 같은 형태를 가진 값이 사용될 수 있다 : 
	* SQLSTATE : 실행도중 어떤 이벤트가 발생했을 때 해당 이벤트의 SQLSTATE 값이 일치할 때 실행되는 핸들러
	* SQLWARNING : 코드 실행 도중 경고가 발생했을 때 실행하는 핸들러
	* NOT FOUND : SELECT 쿼리문의 결과가 1건도 없거나, 커서의 레코드를 마지막까지 읽은 뒤 실행하는 핸들러 정의시 사용
	* SQLEXCEPTION : 경고(SQL Warnin)과 NOT FOUND, 그리고 정상처리된 SQLSTATE외 나머지 모든 케이스를 의미한다.
	* MySQL의 에러코드 

#### 컨디션
MySQL의 핸들러는 특정 조건이 발생했을 때 실행할지를 명시하는 여러가지 방법이 있는데, 그 중 하나가 컨디션이다. 단순히 MySQL의 에러 번호나, SQLSTATE 숫자 값 만으로 어떤 조건을 의미하는지 이해하기 어려우므로 그를 예측할 수 있는 이름으로 만들어주는 것이다.

```sql
DECLARE condition_name CONDITION FOR condition_value
DECLARE dup_key CONDITION FOR 1062;
```

* condition_name : 사용자가 부여하려르 이름을 단순 문자열로 입력하면 된다.
* condition_value : MySQL의 에러번호를 입력하거나, SQLSTATE + SQLSTATE 값을 입력하는 두가지 방식으로 정의할 수 있다.

#### 컨디션을 사용하는 핸들러 정의
사용자가 정의한 CONDITION을 스토ㅓ드 함수에서 어떻게 사용하는지 살펴보자.

```sql
DELIMIETER ;;

CREATE FUNCTION sf_testfunc()
RETUNS BIGINT
BEGIN
	DECLARE dup_key CONDITION FOR 1062;
	DECLARE EXIT HANDLER FOR dup_key
		BEGIN
			RETURN -1;
		END;
	INSERT INTO tb_test VALUES (1);
	RETURN 1;
END ;;
```
위 예제에서 tb_test 테이블은 PK키인 INT형 컬럼 1개를 가지며, 이미 1이란 값이 저장되어 있을 때에는 -1을 반환한다. 스토어드 함수에서 사용되는 EXIT 핸들러에서는 해당 스토어드가 반환해야 하는 타입의 값을 반환하는 RETURN 문장이 포함되어 있어야 한다.

### 시그널(SIGNAL)을 이용한 예외 발생
예외나 에러에 대한 핸들링이 있다면, 반대로 예외를 사용자가 직접 발생시키는 기능이 있어야 한다. MySQL의 스토어드 프로그램에서 사용자가 직접 예외나 에러를 발생시키려면 시그널 명령을 사용해야 한다. 자바와 같은 객체지향 언어에서 핸들러는 CATCH와 같고, 시그널은 THROW와 비슷하다고 볼 수 있다.

시그널은 MySQL 5.5 이상의 버전에서만 지원을 한다.

#### 스토어드 프로그램의 BEGIN ... END 블럭에서의 SIGNAL 사용
DECLARE 구문이 아닌, 스토어드 프로그램의 본문 코드에서 SIGNAL 기능을 사용하는 예제를 살펴보자. 

```sql
DELIMITER ;;
CREATE FUNCTION sf_Devide (p_dividend INT, p_advisor INT)
RETURNS INT
BEGIN
	DECLARE null_divisor CONDITION FOR SQLSTATE '45000';

	IF p_divisor IS NULL THEN
		SIGNAL null_divisor SET MESSAGE_TEXT='Divisor can not be null', MYSQL_ERRNO=9999;
	ELSEIF p_divisor=0 THEN
		SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT='Divisor can not be 0', MYSQL_ERRNO=9998;
	ELSEIF p_dividend IS NULL THEN
		SIGNAL SQLSTATE '01000' SET MESSAGE_TEXT='Dividend is null, so regarding dividend as 0', MYSQL_ERRNO=9997;
		RETURN 0;
	END IF;

	RETURN FLOOR(p_dividend / p_divisor);
END ;;
```

SIGNAL 명령은 직접 SQLSTATE 값을 가질 수도 있으며, 간접적으로 SQLSTATE를 가지는 컨디션을 참조해 에러나 경고를 발생시킬 수도 있다. 중요한 것은 SIGNAL 명령은 SQLSTATE와 직접 또는 간접적으로 연결돼야 한다는 점이다.

### 핸들러 코드에서 SIGNAL 사용
핸들러는 스토어드 프로그램에서 에러나 예외에 대한 처리를 담당한다. 하지만 핸들러 코드에서 SIGNAL 명령을 사용해서 발생된 에러나 예외를 다른 사용자 정의 예외로 변환해서 다시 던지는 것도 가능하다.

### 커서
스토어드 프로그램의 커서는 JDBC 프로그램에서 자주 사용하는 ResultSet으로, PHP상 mysql_query()로 반환되는 결과와 같다. 하지만 스토어드 프로그램에서 사용하는 커서는 JDBC의 ResultSet에 비해 기능이 제약적이다.
(내가 이해했을 때의 커서는, 데이터베이스상 쿼리문에 의해 반환되는 결과 값을 저장하는 공간이라고 생각한다.)
* 스토어드 프로그램의 커서는 전방향(전진) 읽기만 가능하다.
* 스토어드 프로그램에서는 커서의 컬럼을 바로 업데이트 하는 것이 불가능하다.

DBMS의 커서는 인센서티브 커서와, 센서티브 커서로 구분할 수 있다.
* 센서티브 커서 : 일치하는 레코드에 대한 정보를 실제 레코드의 포인터만으로 유지하는 형태. 컬럼의 값이 변경되어 커서를 생성한 SELECT 쿼리 조건에 더는 일치하거나 레코드가 삭제되면 커서에서도 즉시 만영된다.
* 인센서티브 커서 : 일치하는 레코드를 별도의 임시테이블로 복사해서 가지고 있는 형태. SELECT 쿼리에 부합되는 결과를 우선적으로 임시 테이블로 복사해야 하기 때문에 느리다.

센서티브 커서와 인센서티브 커서를 혼용해서 사용하는 방식을 어센서티브 커서라고 하며, MySQL 스토어드 프로그램에서 정의되는 커서는 어센서티브에 속한다.
커서는 일반적인 프로그래밍 언어에서 SELECT 쿼리의 결과를 사용하는 방법과 거의 흡사하다. 스토어드 프로그램에서도 SELECT 쿼리 문장으로 커서를 정의하고, 정의된 커서를 오픈(OPEN)하면 실제로 쿼리가 MySQL 서버에서 실행되고 결과를 가져온다. 이렇게 오픈된 커서는 페치(FETCH) 명령으로 레코드 단위로 읽어 사용할 수 있다. 사용 완료 후, CLOSE 명령으로 커서를 닫어주면 관련 자원이 모두 해제된다. 아래 커서를 사용한 예제를 살펴보자.

```sql
CREATE FUNCTION sf_emp_count(p_dept_no VARCHAR(10))
RETURNS BIGINT
BEGIN
	/* 사원번호가 20000보다 큰 사원 수를 누적하기 위한 변수 */
	DECLARE v_total_count INT DEFAULT 0;
	/* 커서에 더 읽어야 할 레코드가 남아 있는지 여부를 위한 플래그 변수 */
	DECLARE v_no_more_data TINYINT DEFAULT 0;
	/* 커서를 통해 SELECT된 사원번호를 임시로 담아 둘 변수 */
	DECLARE v_emp_no INTEGER;
	/* 커서를 통해 SELECT된 사원의 입사 일자를 임시로 담아 둘 변수 */
	DECLARE v_from_date DATE;

	/* v_emp_list라는 이름으로 커서 정의 */
	DECLARE v_emp_list CURSOR FOR
		SELECT emp_no, form_date, FROM dept_emp WHERE dept_no=p_dept_no;
	/* 커서로부터 더 읽을 데이터가 있는지 여부를 나타내느 플래그 변경을 위한 핸들러 */
	DECLARE CONTINUE HANDLER FOR NOT FOUND SET v_no_more_data = 1;
	
	/* 정의된 v_emp_list 커서를 오픈 */
	OPEN v_emp_list;

	REPEAT
		/* 커서로부터 레코드 한 개씩 읽어서 변수에 저장 */
		FETCH v_emp_list INTO v_emp_no, v_from_date;
		IF v_emp_no > 20000 THEN
			SET v_total_count = v_total_count + 1;
		END IF;
	UNTIL v_no_more_data END REPEAT;

	/* v_emp_list 커서를 닫고 관련 자원 반납 */
	CLOSE v_emp_list;
	RETURN v_total_count;
END ;;
```

위 스토어드 함수는 인자로 전달한 부서의 사원 중, 사원 번호가 20000보다 큰 사원의 수만 employees 테이블에서 카운터해서 반환한다. 커서로부터 더 읽을 데이터가 있는지 여부를 판단하기 위해 HANDLER를 사용했다.

DECLARE 명령으로 CONDITION, HANDLER, CURSOR를 정의하는 순서의 주의해야 한다. 이들의 선언 순서가 정해져 있기 때문이다(아래의 순서대로 정의하자) : 
	* 로컬변수와 CONDITION
	* CURSOR
	* HANDLER


***
참고자료 : 
[Real MySQL - 11.2 스토어드 프로그램의 문법](http://www.yes24.com/Product/Goods/6960931)

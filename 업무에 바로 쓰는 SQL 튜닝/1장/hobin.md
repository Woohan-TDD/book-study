# 1장 MySQL과 MariaDB 개요

## MySQL과 MariaDB 차이

- MySQL 5.5버전 부터 MariaDB 10.1로 버저닝이 나뉜다

## MySQL과 OracleDB 차이

### 구조적 차이

- MySQL
    - MySQL은 물리적인 DB 서버마다 독립적으로 스토리지를 할당하여 구성한다.
    - 보통 Master-Slave 구조로 나뉜다
        - Master에서는 쓰기 트랜잭션(insert, update, delete), Slave에서는 읽기 트랜잭션(select)을 수행한다.
        - 이렇게 쓰기 트랜잭션과 읽기 트랜잭션을 분리해서 DB 처리량을 증가시키는 것을 **쿼리 오프로딩**이라고 한다
    - 그래서 select 쿼리를 마스터에서 쓰면 정상적인 쿼리 결과가 나오지 않는다
- Oracle DB
    - 통합된 스토리지 하나를 공유하여 사용해서 어느 DB 서버에 접속해서 SQL문을 수행하더라도 같은 결과를 출력한다

### 지원 기능 차이

- MySQL
    - nested loop join 방식으로 조인을 사용한다.
    - 스토리지 엔진이라는 개념을 포함한다. 그래서 오픈소스 DBMS를 바로 꽂아서 사용할 수 있다.
    - 메모리 사용률이 낮아서 메모리가 1MB라도 MySQL을 운영할 수 있다.
- OracleDB
    - nested loop join, sort merge join, hash join을 제공한다.
    - 스토리지 엔진이 없다. 그냥 다 자체 형식 데이터로 디스크에 저장한다.
    - 최소 수백 MB 메모리는 있어야 Oracle DB를 운영할 수 있다.

## SQL 구문 차이

Oracle DB 구문은 그때그때 검색해서 사용하면 되므로 MySQL만 적는다.

1. Null을 다른 문자열로 대체 (IFNULL)
    1. `SELECT IFNULL(col1, '대체할 문자열') col1 FROM tab;`
2. 데이터를 일정 부분만 가져올 때 (LIMIT)
    1. `SELECT col FROM tab LIMIT 5;`
3. 현재 날짜
    1. `SELECT NOW() AS date;`
4. 조건문 (IF, CASE WHEN ~ THEN)
    1. `SELECT IF(col1 = 'A', 'apple', '-') AS col1 FROM tab;`
5. 자동 증가 값 (AUTO_INCREMENT)
    1. `CREATE TABLE tab (seq INT NOT NULL AUTO INCREMENT PRIMARY KEY)`
    2. `CREATE SEQUENCE` 로 따로 시퀀스를 추가할 수도 있다. (증감 숫자, 시작 숫자, 최솟값 등...)
6. 문자 결합 (CONCAT)
    1. `SELECT CONCAT('A', 'B') TEXT;`
7. 문자 추출 (SUBSTRING)
    1. `SELECT SUBSTRING('ABCDE, 2, 3) AS sub_string;`

## MySQL과 MariaDB 튜닝의 중요성

- MySQL을 사용하면 제약사항이 있다 (특히 성능적으로 불리함)
    - 대다수의 쿼리가 nested loop join으로 수행
    - 쿼리 결과가 메모리에 적재되는데 데이터가 변경되면 캐시 내용을 모두 삭제돼서 쿼리 작성 및 튜닝이 통하지 않을 수 있음

> 그래서 실행계획(explain)을 항상 확인하는 습관을 길러야 한다!

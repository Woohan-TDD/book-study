# 1장. MySQL과 MariaDB 개요

## 역사

- MySQL은 1995년 오픈소스로 배포된 무료 DBMS입니다. 2010년에 오라클에 인수되고 2021년 6월 현재 MySQL 8.0 버전까지 배포되었으며 사용 버전과 커뮤니티 버전으로 구분합니다.
- Maria DB는 MySQL이 오라클로 인수되고 난 이후 라이선스 및 개발 정책 변화로 MySQL 핵심 개발자들이 주도로 오픈소스 정책을 지향하는 MariaDB를 탄생시켰습니다.
- **SQL을 작성하는 면에서는 차이가 없지만 일부 기능, 수행 메커니즘, 시스템 변수, 시스템 상태 등에서 점점 차이가 나고 있습니다.**

## 현황

- MySQL은 상용 버전과 무료 버전으로 구분되며 무료 버전의 라이선스는 GPL입니다.
- MariaDB는 GPL v2 라이선스를 가진 완전 오픈소스 소프트웨어 입니다.

## 상용 RDBMS와 차이점

### 구조적 차이

- 오라클과 MySQL, MariaDB는 장애 발생 시 가용성을 기대하며 이중화 구조로 구축합니다.
- 오라클은 통합된 스토리지 하나를 공유하여 사용하지만 MySQL은 물리적인 DB 서버마다 독립적으로 스토리지를 할당합니다.
- 즉, 오라클은 DB 서버에 접속하여 SQL을 수행해도 같은 결과를 출력하거나 동일한 구문을 처리할 수 있지만, MySQL은 독립적인 스토리지 할당에 기반을 두는 만큼 이중화를 위한 클러스터나 복제 구성으로 운영하더라도 마스터-슬레이브로 마스터는 쓰기/읽기 처리 모두 수행이 가능하고 슬레이브 노드는 읽기 처리만 가능합니다.
- 애플리케이션을 통해 쿼리 오프로딩이 적용되므로 마스터 노드에는 쓰기를 하고 슬레이브 노드에는 검색을 수행합니다.
- 쿼리 오프로딩이란 DB 서버의 트랜잭션에 쓰기 트랜잭션과 읽기 트랜잭션을 분리하여 DB 처리량을 증가시키는 성능 향상 기법입니다.
- 다만 각 DB 서버의 운영체제 설정, 할당된 스토리지 크기, 시스템 변수, 하드웨어 사양등이 같을 때는 마스터 노드 중심으로 쿼리를 튜닝을 진행해도 무방합니다.

![image](https://user-images.githubusercontent.com/53366407/130351597-2ee6239f-136b-4b70-a9e5-33285ebf2cff.png)
![image](https://user-images.githubusercontent.com/53366407/130351598-6e30661c-ff48-4f80-8d27-2807f74a2628.png)

### 지원 기능 차이

- MySQL과 오라클 DB에서 제공하는 조인 알고리즘의 기능에는 차이가 있습니다.
- MySQL은 대부분 중첩 루프 조인 방식으로 주인을 수행하며, 오라클은 중첩 루프 조인 방식뿐만 아니라 정렬 병합 조인과 해시 조인 방식도 제공합니다.
- MySQL은 오라클과 달리 데이터를 저장하는 스토리지 엔진이라는 개념을 포함하기 떄문에 오픈소스 DBMS를 바로 꽂아서 사용할 수 있는 확장성이 특징입니다. → 플러그 앤 플레이
- MySQL은 오라클 대비 메모리 사용률이 낮아 사양이 낮은 컴퓨팅 환경에서도 설치가 가능합니다.
- 심지어 1MB 메모리 환경에서도 데이터베이스를 운영할 수 있을 만큼 오버헤드가 작지만, 오라클은 최소 수백 MB의 환경이 제공되어야 설치할 수 있습니다.

### SQL 구문 차이

- Null 대체
    - MySQL

    ```jsx
    IFNULL(열명, '대쳇값')
    ```

    - 오라클

    ```jsx
    NVL(열명, '대쳇값')
    ```

- 페이징 처리
    - MySQL은 LIMIT 키워드를 사용하고 오라클은 ROWUM 키워드를 사용합니다.
- 현재 날짜
    - MySQL에서는 NOW()를 사용하고 오라클은 SYSDATE를 사용합니다. MySQL에서도 SYSDATE를 사용가능하지만 함수의 호출 시점을 출력하는 특성으로 다른 값이 나옵니다.
    - MySQL은 FROM 절 없이 SELECT 문만으로 현재 날짜를 출력할 수 있지만 오라클은 가상 테이블을 명시해야 합니다.
- 조건문
    - MySQL은 IF문과 CASE WHEN~THAN문을 사용하며 오라클에서는 DECODE 키워드와 IF문, CASE WHEN~THEN문을 사용합니다.
- 날짜 형식
    - MySQL에서는 DATE_FORMAT() 함수를 사용하며 오라클에서는 TO_CHAR()를 사용합니다.
- 자동 증가값
    - MariaDB와 오라클에서는 각각 스퀀스라는 오브젝트를 활용합니다.
        - 먼저 CREATE SEQUENCE문으로 시퀀스 오브젝트를 생성한 뒤, 해당 시퀀스명으로 함수를 호출하여 신규 숫자를 채번할 수 있습니다.
        - 이때 SELECT 시퀀스명.nextval FROM dual; 구문으로 신규 데이터의 시퀀스 숫자를 가져옵니다.
    - MySQL은 특정 열의 속성으로 자동 증가하는 값을 설정하는 auto_ioncrement를 명시하는 방법과 시퀀스 오브젝트를 생성한 뒤 호출하여 활용하는 방법 두가지를 사용합니다.
        - CREATE SEQUENCE 문으로 시퀀스를 생성한 뒤 SELECT nextval (시퀀스명); 구문으로 신규 순번을 매기는 기능을 활용할 수 있습니다.

    ```jsx
    MySQL/MariaDB
    AUTO_INCREMENT

    MaraiaDB 10.3이상
    CREATE SEQUENCE 시퀀스명
    INCREMENT BY 증감숫자
    START WITH 시작숫자
    NOMINVALUE OR MINVALUE 최솟값
    NOMAXVALUE OR MAXVALUE 최댓값
    CYCLE OR NOCYCEL
    CACHE OR NOCACHE

    다음 값 채번 문법
    SELECT NEXTVAL(시퀀스명);

    오라클
    CREATE SEQUENCE 시퀀스명
    INCREMENT BY 증감숫자
    START WITH 시작숫자
    NOMINVALUE OR MINVALUE 최솟값
    NOMAXVALUE OR MAXVALUE 최댓값
    CYCLE OR NOCYCEL
    CACHE OR NOCACHE

    다음 값 채번 문법
    SELECT 시퀀스명.NEXTVAL FROM dual;
    ```

- 문자 결합
    - MySQL에서는 CONCAT() 함수를 사용하고 오라클에서는 `||` 이나, CONCAT() 함수를 사용 합니다.
- 문자 추출
    - MySQL에서는 SUBSTRING() 함수를 사용하고 오라클에서는 SUBSTR() 함수를 사용합니다.

## MySQL과 MariaDB 튜닝의 중요성

- 클라우드 환경과 접목해 한 대의 기본 노드와 다수의 복제본 노드, 여러 개의 마스터 노드 구조 등 다양한 방식의 안정적인 아키텍처를 서비스에 활용합니다.
- 그러나 기능적인 제약사항이 있습니다. 대다수의 SQL 문이 중첩 루프 조인 알고리즘으로 수행되고, 상용 DBMS와는 다르게 수행된 쿼리 결과가 메모리에 적재되는 캐시 기능에 한계가 있으므로(데이터가 변경되면 캐시된 내용을 모두 삭제) 일반적인 쿼리 작성 및 튜닝이 통하지 않을 수 있습니다.
- 따라서 실행 계획 정보를 분석해 문제점을 인지해 쿼리 튜닝을 해야 합니다.

![image](https://user-images.githubusercontent.com/53366407/130351604-379a2d76-40db-4b5f-8fbf-7c777fdcabe1.png)

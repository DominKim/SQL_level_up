# SQL_level_up

## 1강 DBMS 아키텍처 개요

- 쿼리 평가엔진 : 사용자로부터 입력받은 SQL 구문을 분석하고, 어떤 순서로 기억장치의 데이터에 접근할지를 결정
- 실행 계획(Explain Plan) : 결정되는 계획
- 접근 메서드(access method) : 실행 계획에 기반을 둬서 데이터에 접근하는 방법
- 쿼리(query) : 좁은 의미로는 SELECT 구문을 나타내는 말이며, 큰 의미로는 SQL 구문 전체를 나타냄
- 버퍼 매니저 : DBMS는 버퍼라는 특별한 용도로 사용하는 메모리 영역을 확보해 두는데 이 영역을 관리하는 것
- 디스크 용량 매니저 : 어디에 어떻게 데이터를 저장할지를 관리하며, 데이터의 읽고 쓰기를 제어
- 트랜잭션 매니저와 락 매니저 : 트랜잭션의 정합성을 유지하면서 실행시키고, 필요한 경우 데이터에 락을 걸어 다른 사람의 요청을 대기시키는 것
- 리커버리 매니저 : 데이터를 정기적으로 백업하고, 문제가 일어났을 때 복구 해주는 것
- 기억 비용 : 데이터를 저장하는데 소모되는 비용
- 버퍼(buffer) or 캐시(cache) : 성능 향상을 목적으로 데이터를 저장하는 메모리
- 데이터 캐시 : 디스크에 있는 데이터의 일부를 메모리에 유지하기 위해 사용하는 메모리 영역
- 로그 버퍼 : 갱신 처리와 관련 있음

## 2강 SQL 기초

레코드 (Record) : 레코드란 여러 가지 데이터 타입을 가질 수 있는 복합형 구조의 데이터 타입이며, 하나의 행(Row)에 대응한다. 

1. SELECT 구와 FROM 구
- SELECT : 데이터베이스에서 데이터를 검색ㅎ할 때 반드시 입력해야 하는 부분. 테이블이 갖고 있는 필드라면 쉼표로 연결해서 여러 개 쓸 수 있다.
- FROM [테이블 이름] : 데이터를 선택할 대상 테이블을 지정
``` sql
SELCET name, phone_nbr, address, sex, age 
  FROM Address;
```

2. WHERE 구
- WHERE : SELECT 구문에서 레코드를 선택할 때 추가적인 조건을 지정할 때 사용
``` sql
SELECT name, address
  FROME Address
WHERE address = '인천시';
```
| 연산자 | 의미 |
| ------------- | ------------- |
| = | ~와 같음	|
| <> | ~와 같지 않음	|
| >= | ~ 이상	|
| > | ~ 보다 큼	|
| <= | ~ 이하	|
| < | ~ 보다 작음	|
- AND : 교집합
- OR : 합집합
- IN : OR 조건을 많이 지정해야 할 때
``` sql
SELECT name, address
    FROM Address
 WHERE address IN ('서울시', '부산시', '인천시');
```
- IS NULL : NULL 레코드를 선택할 때 사용하는 특별한 키워드
- IS NOT NULL : IS NULL의 반대
* NULL은 데이터값이 아니므로 데이터값에 적용하는 연산자를 적용할 수 없다.
``` sql
SELECT name, address
    FROM Address
 where phone_nbr IS NULL;
```

3. GROUP BY 구
- GROUP BY : 테으블에서 단순하게 데이터를 선택하는 것뿐만 아리나 합계 또는 평균 등의 집계 연산을 SQL 구문으로 할 수 다.

SQL의 대표적인 집계 함수
| 함수 이름 | 설명 |
| ------------- | ------------- |
| COUNT | 레코드 수를 계산	|
| SUM | 숫자를 더함	|
| AVG | 숫자의 평균을 구함	|
| MAX | 최댓값을 구함	|
| MIN | 최솟값을 구함	|

- GROUP BY 구의 '()'는 키를 지정하지 않는다는 뜻, 일부 DBMS에서는 지원하지 않지만 논리적으로 이해하기는 쉬움
``` sql
SELECT COUNT(*)
  FROM Address
 GROUP BY ();
 
SELECT COUNT(*)
  FROM Address;
```

4. HAVING 구
- HAVING : 집합에 또다시 조건을 걸어 선택하는 기능
* WHERE 구와 차이점은 WHERE 구는 레크드에 조건을 지정하고 HABING 구는 집합에 조건을 지정
``` sql
SELECT address, COUNT(*)
    FROM Address
  GROUP BY address
HAVING COUNT(*) = 1;
```

5. OREDER BY 구
- ORDER BY : 모든 DBMS에서 SELECT 구문의 결과 순서를 보장할 때 명시적은 순서를 지정이 필요한데 그때 사용하는 기능
- DESC : 내림차순, ASC : 오름차순(default)
``` sql
SELECT name, phone_nbr, address, sex, age
    FROM Address
  ORDER BY age DESC, phone_nbr;
```
- 정렬 시 겹치는 레코드가 있으면 정렬 키를 추가해서 정렬 순서를 지정

6. 뷰와 서브쿼리
- VIEW : 자주 사용하는 SELECT 구문을 데이터베이스 안에 저장하는 기능, 데이터를 보유하지 않고 SELECT구문을 저장
- 뷰 생성 : CREATE VIEW [뷰 이름] ([필드 이름1], [필드 이름2] ...) AS
``` sql
CREATE VIEW CountAddress (v_address, cnt)
AS
SELECT address, COUNT(*)
    FROM Address
  GROUP BY address;
```
- 뷰 사용 : SELECT 구문 처럼 사용
``` sql
SELECT v_address, cnt
    FROM CountAddress;
```

- 서브쿼리(subquery) : FROM 구에 직접 지정한 SELECT 구문
* SQL은 서브 쿼리부터 순서대로 실행한다.
``` sql
SELECT name
    FROM Address
 WHERE name IN (SELECT name FROM Address2);
```

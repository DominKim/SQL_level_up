## 2강 SELECT 구

레코드 (Record) : 레코드란 여러 가지 데이터 타입을 가질 수 있는 복합형 구조의 데이터 타입이며, 하나의 행(Row)에 대응한다. 
수치형 데이터 : 오른쪽 정렬
문자열형 데이터 : 왼쪽 정렬
날짜시간형 데이터 : 왼쪽 정렬
DESC : 테이블 참조
리터릴(Literal) : 자료형에 맞게 표기한 상수값 ex) '김도민', '2021-03-12'

1. SELECT 구와 FROM 구
- SELECT : 데이터베이스에서 데이터를 검색할 때 반드시 입력해야 하는 부분. 테이블이 갖고 있는 필드라면 쉼표로 연결해서 여러 개 쓸 수 있다.
  \* : 애스터리스크, 모든 열을 의미하는 메타문자
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
- LIKE : 문자열의 일부분을 부분 검색
  - % : 임의의 문자열, 빈 문자열에도 매치, %를 문자로 검색 하기 위해서 \를 앞에 붙인다 ex) '%\'
  - _ : 임의의 문자 하나
- '''' : 문자열 상수를 검색하기 위해서는 '를 2개 연속해서 기술
- ROWNUM : 행번호를 검색 가능하게 하는 함수 검색 조건에서 몇개의 행을 볼 것인지 정할 때 사용 가능 ex) WHERE ROWNUM <= 3;

3. GROUP BY 구
- GROUP BY : 테이블에서 단순하게 데이터를 선택하는 것뿐만 아니라 합계 또는 평균 등의 집계 연산을 SQL 구문으로 할 수있다.

SQL의 대표적인 집계 함수
| 함수 이름 | 설명 |
| ------------- | ------------- |
| COUNT | 레코드 수를 계산	|
| SUM | 숫자를 더함	|
| AVG | 숫자의 평균을 구함	|
| MAX | 최댓값을 구함	|
| MIN | 최솟값을 구함	|
| ROUND | 반올림 |
| TRUNCATE | 반내림 |

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
- 문자열형 데이터의 대소관계는 사전식 순서에 의해 결정 ex) 알파펫 -> 한글, 자음 -> 모음
- 수치형과 문자열형 데이터는 대소관계의 계산 방법이 다르다. ex) '1' -> '10' -> '2', 1 -> 2-> 10
``` sql
SELECT name, phone_nbr, address, sex, age
    FROM Address
  ORDER BY age DESC, phone_nbr;
```
- 정렬 시 겹치는 레코드가 있으면 정렬 키를 추가해서 정렬 순서를 지정
- NULL 값은 가장 먼저 표시되거나 가장 나중에 표시

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

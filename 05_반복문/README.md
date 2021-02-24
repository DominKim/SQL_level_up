## 05_반복문

## SQL에서는 반복을 어떻게 표현할까?
### 1. 포인트는 CASE 식과 윈도우 함수

- SIGN() : 숫자형 자료형을 매개변술 받아 음수라면 -1, 양수라며 1, 0이라면 0을 리턴하는 함수
- ROWS : 물리적인 ROW 단위로 행 집합을 지정한다.
- RANGE : 논리적인 상대번지로 행 집합을 지정한다.
- BETWEEN ~ AND 절 : 윈도우의 시작과 끝 위치를 지정한다.
- UNBOUNDED PRECEDING : PARTITION의 첫 번째 로우에서 윈도우가 시작한다.
- UNBOUNDED FOLLOWING : PARTITION의 마지막 로우에서 윈도우가 시작한다.
- CURRENT ROW : 윈도우의 시작이나 끝 위치가 현재 로우 이다.
- PRECEDING : 현재 행에서 앞쪽 행을 나타냄
- FOLLOWING : 현재 행에서 뒤쪽 행으 나타냄

윈도우 함수를 사용한 방법
``` sql
INSERT INTO Sales2
SELECT company,
       year,
       sale,
       CASE SIGN(sale - MAX(sale)
                         OVER ( PARTITION BY company
                                    ORDER BY year
                                     ROWS BETWEEN 1 PRECEDING
                                              AND 1 PRECEDING) )
       WHEN 0 THEN '='
       WHEN 1 THEN '+'
       WHEN -1 THEN '-'
       ELSE NULL END AS var
　FROM Sales;
```
윈도우 함수로 '직전 회사명'과 '직전 매상'검색
``` sql
SELECT company,
       year,
       sale,
       MAX(company)
         OVER (PARTITION BY company
                   ORDER BY year
                    ROWS BETWEEN 1 PRECEDING
                             AND 1 PRECEDING) AS pre_company,
       MAX(sale)
         OVER (PARTITION BY company
                   ORDER BY year
                    ROWS BETWEEN 1 PRECEDING
                             AND 1 PRECEDING) AS pre_sale
　FROM Sales;
```

### 2. 최대 반복 횟수가 정해진 경우
- 결국 순위 붙이기 문제
우편번호로 순위를 매기는 커리
``` sql
SELECT pcode,
       district_name,
       CASE WHEN pcode = '4130033' THEN 0
            WHEN pcode LIKE '413003%' THEN 1
            WHEN pcode LIKE '41300%'  THEN 2
            WHEN pcode LIKE '4130%'   THEN 3
            WHEN pcode LIKE '413%'    THEN 4
            WHEN pcode LIKE '41%'     THEN 5
            WHEN pcode LIKE '4%'      THEN 6
            ELSE NULL END AS rank
  FROM PostalCode;
```

가까운 우편번호를 구하는 쿼리
``` sql
SELECT pcode,
       district_name
  FROM PostalCode
 WHERE CASE WHEN pcode = '4130033' THEN 0
            WHEN pcode LIKE '413003%' THEN 1
            WHEN pcode LIKE '41300%'  THEN 2
            WHEN pcode LIKE '4130%'   THEN 3
            WHEN pcode LIKE '413%'    THEN 4
            WHEN pcode LIKE '41%'     THEN 5
            WHEN pcode LIKE '4%'      THEN 6
            ELSE NULL END = 
                (SELECT MIN(CASE WHEN pcode = '4130033' THEN 0
                                 WHEN pcode LIKE '413003%' THEN 1
                                 WHEN pcode LIKE '41300%'  THEN 2
                                 WHEN pcode LIKE '4130%'   THEN 3
                                 WHEN pcode LIKE '413%'    THEN 4
                                 WHEN pcode LIKE '41%'     THEN 5
                                 WHEN pcode LIKE '4%'      THEN 6
                                 ELSE NULL END)
                   FROM PostalCode);
```

- 윈도우 함수를 사용한 스캔 횟수 감소
``` sql
SELECT pcode,
       district_name
  FROM (SELECT pcode,
               district_name,
               CASE WHEN pcode = '4130033' THEN 0
                    WHEN pcode LIKE '413003%' THEN 1
                    WHEN pcode LIKE '41300%'  THEN 2
                    WHEN pcode LIKE '4130%'   THEN 3
                    WHEN pcode LIKE '413%'    THEN 4
                    WHEN pcode LIKE '41%'     THEN 5
                    WHEN pcode LIKE '4%'      THEN 6
                    ELSE NULL END AS hit_code,
               MIN(CASE WHEN pcode = '4130033' THEN 0
                        WHEN pcode LIKE '413003%' THEN 1
                        WHEN pcode LIKE '41300%'  THEN 2
                        WHEN pcode LIKE '4130%'   THEN 3
                        WHEN pcode LIKE '413%'    THEN 4
                        WHEN pcode LIKE '41%'     THEN 5
                        WHEN pcode LIKE '4%'      THEN 6
                        ELSE NULL END) 
                OVER(ORDER BY CASE WHEN pcode = '4130033' THEN 0
                                   WHEN pcode LIKE '413003%' THEN 1
                                   WHEN pcode LIKE '41300%'  THEN 2
                                   WHEN pcode LIKE '4130%'   THEN 3
                                   WHEN pcode LIKE '413%'    THEN 4
                                   WHEN pcode LIKE '41%'     THEN 5
                                   WHEN pcode LIKE '4%'      THEN 6
                                   ELSE NULL END) AS min_code
          FROM PostalCode) Foo
 WHERE hit_code = min_code;
```

### 3. 반복 횟수가 정해지지 않은 경우
- 인접 리슽 모델과 재귀 쿼리
    - 포인터 체인 : 우편번호를 키로 삼아 데이터를 줄줄이 연결하 것
    - 인접 리스 모델 : 포인터 체인을 사용하는 PostalHistory 같은 테이블 형식
- A씨의 현재 주소가 NULL인 필드 부터 풀발햇 포인트 체인을 타고 올 과거으 주소르 모두 찾는다.
가장 오래된 주소 검색
``` sql
WITH Explosion (name, pcode, new_pcode, depth)
AS
(SELECT name, pcode, new_pcode, 1
   FROM PostalHistory
  WHERE name = 'A'
    AND new_pcode IS NULL -- 검색시작
 UNION ALL
 SELECT Child.name, Child.pcode, Child.new_pcode, depth + 1
   FROM Explosion Parent, PostalHistory Child
  WHERE Parent.pcode = Child.new_pcode
    AND Parent.name = Child.name)
-- 메인 SELECT 구문
SELECT name, pcode, new_pcode
  FROM Explosion
 WHERE depth = (SELECT MAX(depth)
                   FROM Explosion);
```

- 중첩 집합 모델 
    - NOT EXISTS() : 함수 안의 조건이 존재하지 않을 때 출력(차집합)
가장 외부에 있는 원 찾기
``` sql
SELECT name, pcode
    FROM PostalHistory2 PH1
 WHERE name = 'A'
    AND NOT EXISTS
        (SELECT *
            FROM PostalHistory2 PH2
         WHERE PH2.name = 'A'
            AND PH1.lft > PH2.lft);
```

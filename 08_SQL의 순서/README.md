## 07_서브쿼리

## 서브쿼리가 일으키는 폐해
### 1. 서브쿼리의 문제점
- 서브쿼리가 실체적인 데이터를 저장하고 있지 않다는 점에서 성능적 문제 야기

- 연산 비용 추가
  - SELECT 구문 실행에 발생하는 비용 추가

- 데이터 I/O 비용 발생
  - 데이터 양이 큰 경우 등에는 DBMS가 저장송 있는 파일에 결과를 쓸 때도 있다.

- 최적화를 받을 수 없음
  - 서브쿼리에는 메타 정보(인덱스 등)가 하나도 존재하지 않다.

### 2. 서브쿼리 의존증
- 서브쿼리를 사용한 방법
``` sql
SELECT R1.cust_id, R1.seq, R1.price
　FROM Receipts R1
         INNER JOIN
           (SELECT cust_id, MIN(seq) AS min_seq
              FROM Receipts
             GROUP BY cust_id) R2
    ON R1.cust_id = R2.cust_id
　 AND R1.seq = R2.min_seq;
```
  - 코드가 복잡해서 읽기 어렵다, 성능의 단점

- 상관 서브쿼리는 답이 될 수 없다.
``` sql
SELECT cust_id, seq, price
　FROM Receipts R1
 WHERE seq = (SELECT MIN(seq)
                FROM Receipts R2
               WHERE R1.cust_id = R2.cust_id);
```

- 윈도우 함수로 결합을 제거
  - ROW_NUMBER() : 그룹 내 순위 결정 함수
``` sql
SELECT cust_id, seq, price
  FROM (SELECT cust_id, seq, price,
            ROW_NUMBER() OVER (PARTITION BY cust_id
                                   ORDER BY seq) AS row_seq
        FROM Receipts) WORK
 WHERE WORK.row_seq = 1;
```

### 3. 장기적 관점에서의 리스크 관리
- 저장소의 I/O 양을 감소시키는 것이 SQL 튜닝의 가장 기본 원칙
- 결합을 사용하는 쿼리의 불안정 요소
  - 1. 결합 알고리즘의 변동 리스크
  - 2. 환경 요인에 의한 지연 리스크(인덱스, 메모리, 매개변수 등)

- 알고리즘 변동 리스크 
  - 테이블의 크기 등을 고려해서 옵티마이저가 자동으로 결정

- 환경 요인에 의하 지여 리스크
  - 실행 계획이 단순할수록 성능이 안정적
  - 엔지니어는 기능(결과)뿐만 아니라 비기능적인 부분(성능)도 보장할 책임이 있다.

### 4.서브쿼리 의존증 - 응용편
- 다시 서브쿼리 의존증
``` sql
SELECT TMP_MIN.cust_id,
       TMP_MIN.price - TMP_MAX.price AS diff
　FROM (SELECT R1.cust_id, R1.seq, R1.price
          FROM Receipts R1
                 INNER JOIN
                  (SELECT cust_id, MIN(seq) AS min_seq
                     FROM Receipts
                    GROUP BY cust_id) R2
            ON R1.cust_id = R2.cust_id
           AND R1.seq = R2.min_seq) TMP_MIN
       INNER JOIN
       (SELECT R3.cust_id, R3.seq, R3.price
          FROM Receipts R3
                 INNER JOIN
                  (SELECT cust_id, MAX(seq) AS min_seq
                     FROM Receipts
                    GROUP BY cust_id) R4
            ON R3.cust_id = R4.cust_id
           AND R3.seq = R4.min_seq) TMP_MAX
    ON TMP_MIN.cust_id = TMP_MAX.cust_id;
```

- 레코드 간 비교에서도 결합은 불필요
``` sql
SELECT cust_id,
       SUM(CASE WHEN min_seq = 1 THEN price ELSE 0 END)
            - SUM(CASE WHEN max_seq = 1 THEN price ELSE 0 END) diff
    FROM (SELECT cust_id, price,
                 ROW_NUMBER() OVER (PARTITION BY cust_id
                                        ORDER BY seq) AS min_seq,
                 ROW_NUMBER() OVER (PARTITION BY cust_id
                                        ORDER BY seq DESC) AS max_seq
                FROM Receipts) WORK
  WHERE WORK.min_seq = 1
     OR WORK.max_seq = 1
 GROUP BY cust_id;
```

### 5. 서브쿼리는 정말 나쁠까?
- 생각의 보조 도구

## 서브쿼리 사용이 더 나은 경우
### 1. 결합과 집약 순서
- 두 가지 방법
결합을 먼저 수행
``` sql
 SELECT C.co_cd, MAX(C.district),
        SUM(emp_nbr) AS  sum_emp
    FROM Companies C
            INNER JOIN
               Shops S
      ON C.co_cd = S.co_cd
 WHERE main_flg = 'Y'
 GROUP BY C.co_cd;
```
집약을 먼저 수행
``` sql
SELECT C.co_cd, C.district, sum_emp
　FROM Companies C
         INNER JOIN
          (SELECT co_cd,
                  SUM(emp_nbr) AS sum_emp
             FROM Shops
            WHERE main_flg = 'Y'
            GROUP BY co_cd) CSUM
    ON C.co_cd = CSUM.co_cd;
```

- 결합 대상 레코드 수
#


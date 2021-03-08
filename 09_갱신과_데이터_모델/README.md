## 08_SQL의 순서

## 레코드에 순번 붙이기
### 1. 기본 키 한 개의 필드일 경우
- 윈도우 함수를 사용
기본 키가 한 개의 필드일 경우(ROW_NUMBER())
``` sql
SELECT student_id,
       ROW_NUMBER() OVER (ORDER BY student_id ) AS seq
    FROM Weights;
```

- 상관 서브쿼리르 사용
기본 키가 한 개의 필드일 경우(상관 서브쿼리)
``` sql
SELECT student_id,
       (SELECT COUNT(*)
          FROM Weights W2
         WHERE W2.student_id <= W1.student_id) AS seq
    FROM Weights W1;
```
  - 재귀 집합을 만들고 요소 수를 COUNT 함수로 센다. 기본 키 student_id를 비교 키로 사용하므로 재귀 집합의 요소가 한 개씩 증가

### 2. 기본 키가 여러 개의 필드로 구성되는 경우
- 윈도우 함수를 사용
기본 키가 여러 개의 필들 구성되는 경우(ROW_NUMBER)
``` sql
SELECT class, student_id,
       ROW_NUMBER() OVER (ORDER BY class, student_id) AS seq
    FROM Weights2;
```

- 상관 서브쿼리를 사용
  - 다중 필드 비교 사용
``` sql
SELECT class, student_id,
       (SELECT COUNT(*)
          FROM Weights2 W2
         WHERE (W2.class, W2.student_id)
                 <= (W1.class, W1.student_id) ) AS seq
　FROM Weights2 W1;
```

### 3. 그룹마다 순번을 붙이는 경우
- 윈도우 함수를 사용
``` sql
SELECT class, student_id,
       ROW_NUMBER() OVER (PARTITION BY class ORDER BY student_id) AS seq
    FROM Weights2;
```

- 상관 서브쿼리를 사용
``` sql
SELECT class, student_id,
       (SELECT COUNT(*)
           FROM Weights2 W2
        WHERE W2.class = W1.class
          AND W2.student_id <= W1.student_id) AS seq
 FROM Weights2 w1;
```

### 4. 순번과 갱신
- UPDATE 구문

- 윈도우 함수를 사용
``` sql
UPDATE Weights3
   SET seq = (SELECT seq
                 FROM (SELECT class, student_id,
                              ROW_NUMBER() OVER (PARTITION BY class
                                                     ORDER BY student_id) AS seq
                         FROM Weights3) SeqTbl
                WHERE Weights3.class = SeqTbl.class
                  AND Weights3.student_id = SeqTbl.student_id);
```

- 상관 서브쿼리를 사용
``` sql
UPDATE Weight3
   SET seq = (SELECT COUNT(*)
                FROM Weights3 W2
              WHERE W2.class = Weights3.class
                AND W2.student_id <= Weights3.student_id);
```

## 레코드에 순번 붙이기 응용
### 1. 중앙값 구하기
- 집합 지향적 방법
``` sql
SELECT AVG(weight)
　FROM (SELECT W1.weight
          FROM Weights W1, Weights W2
         GROUP BY W1.weight
            -- S1(하위 집합)의 조건
        HAVING SUM(CASE WHEN W2.weight >= W1.weight THEN 1 ELSE 0 END)
                  >= COUNT(*) / 2
            -- S2(상위 집합)의 조건
           AND SUM(CASE WHEN W2.weight <= W1.weight THEN 1 ELSE 0 END)
                  >= COUNT(*) / 2 ) TMP;
```

- 절차 지향적 방법 1 : 세계의 중심을 향해
중앙값 구하기(절차 지향형 1) : 양쪽 끝에 레코드 하나씩 세어 중간을 찾음
``` sql
SELECT AVG(weight) AS median
  FROM (SELECT weight,
               ROW_NUMBER() OVER (ORDER BY weight ASC, student_id ASC) AS hi,
               ROW_NUMBER() OVER (ORDER BY weight DESC, student_id DESC) AS lo
            FROM Weights) TMP
  WHERE hi IN (lo, lo + 1, lo - 1);
```
  - RANk or DENSE_RANK는 사용 불가

- 절차 지향적 방법2 : 2 빼기 1은 1
중앙값 구하기(절차 지향형 2) : 반환점 발견
``` sql
SELECT AVG(weight)
　FROM (SELECT weight,
               2 * ROW_NUMBER() OVER(ORDER BY weight)
                   - COUNT(*) OVER() AS diff
          FROM Weights) TMP
```
  - SQL 표준으로 중앙값을 구하는 가장 빠른 쿼리

### 2. 순번을 사용한 테이블 분할
- 단절 구간 찾기

- 집합 지향적 방법 : 집합의 경계선
``` sql
SELECT (N1.num + 1) AS gap_start,
       '~',
       (MIN(N2.num) - 1) AS gap_end
    FROM Numbers N1 INNER JOIN Numbers N2
      ON N2.num > N1.num
 GROUP BY N1.num
HAVING (N1.num + 1) < MIN(N2.num);
```

- 절차 지향적 방법 : '다음 레코드'와 비교
``` sql
SELECT num + 1 AS gap_start,
       '～',
       (num + diff - 1) AS gap_end
　FROM (SELECT num,
               MAX(num)
                 OVER(ORDER BY num
                       ROWS BETWEEN 1 FOLLOWING
                                AND 1 FOLLOWING) - num AS diff
          FROM Numbers) 
 WHERE diff <> 1;
```

### 3. 테이블에 존재하는 시퀀스 구하기
- 집합 지향적 방법 : 다시, 집합의 경계선
``` sql
SELECT MIN(num) AS low,
       '～',
       MAX(num) AS high
　FROM (SELECT N1.num,
               COUNT(N2.num) - N1.num AS gp
          FROM Numbers N1 INNER JOIN Numbers N2
            ON N2.num <= N1.num
         GROUP BY N1.num) 
 GROUP BY gp;
```

- 절차 지향형 방법 : 다시 '다음 레코드 하나'와 비교
``` sql
SELECT low, high
　FROM (SELECT low,
               CASE WHEN high IS NULL
                    THEN MIN(high)
                           OVER (ORDER BY seq
                                  ROWS BETWEEN CURRENT ROW
                                           AND UNBOUNDED FOLLOWING)
                    ELSE high END AS high
          FROM (SELECT CASE WHEN COALESCE(prev_diff, 0) <> 1
                            THEN num ELSE NULL END AS low,
                       CASE WHEN COALESCE(next_diff, 0) <> 1
                            THEN num ELSE NULL END AS high,
                       seq
                  FROM (SELECT num,
                               MAX(num)
                                 OVER(ORDER BY num
                                       ROWS BETWEEN 1 FOLLOWING
                                                AND 1 FOLLOWING) - num AS next_diff,
                               num - MAX(num)
                                       OVER(ORDER BY num
                                             ROWS BETWEEN 1 PRECEDING
                                                      AND 1 PRECEDING) AS prev_diff,
                               ROW_NUMBER() OVER (ORDER BY num) AS seq
                          FROM Numbers) TMP1 ) TMP2) TMP3
 WHERE low IS NOT NULL;
```


#
#

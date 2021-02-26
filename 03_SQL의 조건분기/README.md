## 03_SQL의 조건분기

### UNION을 사용한 쓸데없이 긴 표현
- UNION 사용 여부는 신중히 검토하는 것이 필요!
- Why? UNION 사용은 여러 개의 SELECT 구문으로 인해 테이블 접근횟수가 많아져 I/O 비용이 크게 늘어남 

2001년까지는 세금이 포함되지 않은 가격, 2002년 부터는 세금이 포함된 경우 'price' 필드로 표시
``` sql
SELECT item_name, year, price_tax_ex AS price
    FROM Items
 where year <= 2001
UNION ALL
SELECT item_name, year, price_tax_in AS price
    FROM Items
    where year > 2001;
```

- 정확한 판단 없이 SELECT 구문 전체를 여러 번 사용해서 코드를 길게 만드는 것은 쓸데없는 테이블 접근을 발생시키며 SQL의 성능을 나쁘게 만듬
- WHERE 구에서 조건 분기를 하는 사람은 초보자
``` sql
SELECT item_name, year,
    CASE WHEN year <= 2001 THEN price_tax_ex
         WHEN year >= 2002 THEN price_tax_in END AS price
 FROM Items;
```
- IF 조건문을 CASE로는 어떻게 해결할 수 있는지?에 대해 꾸준히 생각하기 (SQL 마스터 열쇠 중 하나)

### 집계와 조건 분기

- 집계 대상으로 조건 분기
지역 별 남자와 여자 인구 구하기(표측 / 표두 레이아웃 이동 문제)
``` sql
SELECT prefecture,
    SUM(CASE WHEN sex = 1 THEN pop ELSE 0 END) AS pop_men,
    SUM(CASE WHEN sex = 2 THEN pop ELSE 0 ENd) AS pop_wom
 FROM Population
GROUP BY prefecture;
```

- 집약 결과로 조건 분기
- 문자열을 사용할 경우 MAX는 데이터베이스가 해당 열에 정의한 정렬 순서에서 가장 높은 값을 찾습니다.
```sql
SELECT emp_name,
    CASE WHEN COUNT(*) = 1 THEN MAX(team)
         WHEN COUNT(*) = 2 THEN '2개를 겸무'
         WHEN COUNT(*) >= 3 THEN '3개 이상을 겸무'
     END AS team
 FROM Employees
GROUP BY emp_name;
```

### 그래도 UNION이 필요한 경우
- UNION을 사용할 수밖에 없는 경우 : 머지 대상이 되는 테이블이 다른 경우
``` sql
SELECT col_1
    FROM Table_A
 WHERE col_2 = 'A'
UNION ALL
SELECT col_3
    FROM Table_B
 WHERE col_4 = 'B';
```

- UNION을 사용하는 것이 성능적으로 더 좋은 경우
    - 필드 조합에 인덱스를 사용하면 최적의 성능으로 수행

2013-11-1를 값으로 갖고 있고 대칭되는 플래그 필드의 값이 'T'인 레코드 선택

- UNION을 사용한 방법
``` sql
SELECT key, name,
       date_1, flg_1,
       date_2, flg_2,
       date_3, flg_3
    FROM ThreeElements
 WHERE date_1 = '2013-11-01'
  AND flg_1 = 'T'
UNION
SELECT key, name,
       date_1, flg_1,
       date_2, flg_2,
       date_3, flg_3
    FROM ThreeElements
 WHERE date_2 = '2013-11-01'
  AND flg_2 = 'T'
UNION
SELECT key, name,
       date_1, flg_1,
       date_2, flg_2,
       date_3, flg_3
    FROM ThreeElements
 WHERE date_3 = '2013-11-01'
  AND flg_3 = 'T';
```

- OR을 사용한 방법
- WHERE 구문에서 OR을 사용하면 해당 필드에 부여된 인덱스를 사용할 수 없다.
- SO, UNION vs OR = 3회의 인덱스 스캔 vs 1회의 테이블 풀 스캔
- 속도 판단 : 테이블이 크고, WHERE 조건으로 선택되는 레코드의 수가 충분히 작다면 UNION이 더 빠르다.
``` sql
SELECT key, name,
       date_1, flg_1,
       date_2, flg_2,
       date_3, flg_3
    FROM ThreeElements
 WHERE (date_1 = '2013-11-01' AND flg_1 = 'T')
    OR (date_2 = '2013-11-01' AND flg_2 = 'T')
    OR (date_3 = '2013-11-01' AND flg_3 = 'T');
```

-IN을 사용한 방법
- 다중 필드라는 기능을 사용한 방법
``` sql
SELECT key, name,
       date_1, flg_1,
       date_2, flg_2,
       date_3, flg_3
    FROM ThreeElements
 WHERE ('2013-11-01', 'T')
        IN ((date_1, flg_1),
            (date_2, flg_2),
            (date_3, flg_3));
```

- CASE식을 사용한 방법
``` sql
SELECT key, name,
       date_1, flg_1,
       date_2, flg_2,
       date_3, flg_3
　FROM ThreeElements
 WHERE CASE WHEN date_1 = '2013-11-01' THEN flg_1
            WHEN date_2 = '2013-11-01' THEN flg_2
            WHEN date_3 = '2013-11-01' THEN flg_3
       ELSE NULL END = 'T';
```

- UNION과 IN은 짝들을 전부 평가한다.
- CASE 식의 WHEN 구는 단락 평가를 수행 한다.

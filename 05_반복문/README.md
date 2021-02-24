## 04_집약과 자르기

## 집약
### 1. 여러 개의 레코드를 한 개의 레코드로 집약
- 집약 함수(aggregate function)
    - COUNT, SUM, AVG, MAX, MIN


- CASE 식과 GROUP BY 응용
    - GROUP BY 구로 집약 했을 때 SELECT 구에 입력할 수 있는 것 : 상수, GROUP BY 구에서 사용한 집약 키, 집약 함수
    - MAX, MIN을 사용하는 습관을 들인다.
``` sql
SELECT id, 
    MAX(CASE WHEN data_type = 'A' THEN data_1 ELSE NULL END) AS data_1
    MAX(CASE WHEN data_type = 'A' THEN data_2 ELSE NULL END) AS data_2,
    MAX(CASE WHEN data_type = 'B' THEN data_3 ELSE NULL END) AS data_3,
    MAX(CASE WHEN data_type = 'B' THEN data_4 ELSE NULL END) AS data_4,
    MAX(CASE WHEN data_type = 'B' THEN data_5 ELSE NULL END) AS data_5,
    MAX(CASE WHEN data_type = 'C' THEN data_6 ELSE NULL END) AS data_6
 FROM NonAggTbl
GROUP BY id;
```

- 집약, 해시, 정렬

### 2. 합쳐서 하나

여러 개의 레코드로 한 개의 범위를 커버
``` sql
SELECT product_id
    FROM PriceByAge
 GROUP BY product_id
HAVING SUM(high_age - low_age + 1) = 101;
```

여러 개의 레코드에서 운영된 날을 연산
``` sql
SELECT room_nbr,
       SUM(end_date - start_date) AS working_days
    FROM HotelRooms
 GROUP BY room_nbr
HAVING SUM(end_date - start_date) >= 10;
```

## 자르기

### 1. 자르기와 파티션

- substr(필드, start, end) : 필드에 속한 문자를 범위로 자르는 함수

첫 문자 알파벳마다 몇 명의 사람이 존재하는지 계산
``` sql
SELECT SUBSTR(name,1,1) AS label,
    count(*)
 FROM Persons
GROUP BY SUBSTR(name,1,1) ORDER BY label;
```

- 파티션
    - 파티션(partition) : GROUP BY 구로 잘라 만든 하나하나의 부분 집합을 수학적으로 부르는 용어, 서로 중복되는 요소를 가지지 않는 부분 집합
    - 자르기의 기준이 되는 키를 GROUP BY 구와 SELECT 구 모두에 입력하는 것이 키 포인트
    - 집약 함수와 GROUP BY의 실행 계획은 성능적인 측면에서, 해시에 사용되는 워킹 메모리의 용량에 주의 
나이로 자르기
``` sql
SELECT CASE WHEN age < 20 THEN '어린이'
            WHEN age BETWEEN 20 AND 69 THEN '성인'
            WHEN age >= 70 THEN '노인'
            ELSE NULL END AS age_class,
        COUNT(*)
    FROM Persons
GROUP BY CASE WHEN age < 20 THEN '어린이'
            WHEN age BETWEEN 20 AND 69 THEN '성인'
            WHEN age >= 70 THEN '노인'
            ELSE NULL END;
```

    - POWER(필드명, 제곱수) : 필드에 제곱수 만큼 제곱하는 함수
BMI로 자르기
``` sql
SELECT CASE WHEN weight / POWER(height / 100, 2) < 18.5 THEN '저제중'
            WHEN 18.5 <= weight / POWER(height / 100, 2)
                AND weight / POWER(height / 100, 2) < 25 THEN '정상'
            WHEN 25 <= weight / POWER(height / 100, 2) THEN '과체중'
            ELSE NULL END AS bmi,
        COUNT(*)
    FROM Persons
GROUP BY CASE WHEN weight / POWER(height / 100, 2) < 18.5 THEN '저제중'
            WHEN 18.5 <= weight / POWER(height / 100, 2)
                AND weight / POWER(height / 100, 2) < 25 THEN '정상'
            WHEN 25 <= weight / POWER(height / 100, 2) THEN '과체중'
            ELSE NULL END;
```

### 2. PARTITION BY 구를 사용한 자르기
- GROUP BY의 집약 기능을 제외하면 PARTITION BY 구와 기능 차이는 없다

``` sql
SELECT name,
       age,
       CASE WHEN age < 20 THEN '어린이'
            WHEN age BETWEEN 20 AND 69 THEN '성인'
            WHEN age >= 70 THEN '노인'
            ELSE NULL END AS age_class,
        RANK() OVER(PARTITION BY CASE WHEN age < 20 THEN '어린이'
            WHEN age BETWEEN 20 AND 69 THEN '성인'
            WHEN age >= 70 THEN '노인'
            ELSE NULL END
            ORDER BY age) AS age_rank_in_class
    FROM Persons
 ORDER BY age_class, age_rank_in_class;
```

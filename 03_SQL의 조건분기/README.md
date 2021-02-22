## 03_SQL의 조건분기

### UNION을 사용한 쓸데없이 긴 표현
- UNION 사용 여부는 신중히 검토하는 것이 필요!
- Why? UNION 사용은 여러 개의 SELECT 구문으로 인해 테이블 접근횟수가 많아져 I/O 비용이 크게 늘어남 

2001년까지는 세금이 포함되지 않은 가격, 2002년 부터는 세금이 포함된 가겨을'price' 필드로 표시
``` sql
SELECT item_name, year, price_tax_ex AS price
    FROM Items
 where year <= 2001
UNION ALL
SELECT item_name, year, price_tax_in AS price
    FROM Items
    where year > 2001;
```

- 정확한 판단 없이 SELECT 구문 전체를 여러 번 사용해서 코드를 길게 만드느느 것은 쓸데없는 테이블 접근을 발생시키며 SQL의 성능을 나쁘게 만듬
- WHERE 구에서 조건 분기를 하는 사람은 초보자
``` sql
SELECT item_name, year,
    CASE WHEN year <= 2001 THEN price_tax_ex
         WHEN year >= 2002 THEN price_tax_in END AS price
 FROM Items;
```
- IF 조건문을 CASE로는 어떻게 해결할 수 있는지?에 대해 꾸준히 생각하기 (SQL 마스터 열쇠 중 하나)

### 집계와 조건 분기

지역 별 남자와 여자 인구 구하기

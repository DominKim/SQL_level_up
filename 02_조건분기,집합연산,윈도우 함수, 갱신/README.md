# SQL_level_up

## 3강 조건분기, 집합연산, 윈도우 함수, 갱신

1. 조건분기
- CASE 식의 구문
  - 단순 CASE 식
  - 검색 CASE 식 : 단순 CASE 식의 기능을 모두 포함하고 있으므로 검색 CASE식만 기헉해도 충분
``` sql
CASE WHEN [평가식] THEN [식]
     WHEN [평가식] THEN [식]
     WHEN [평가식] THEN [식]
     생략
     ELSE [식]
END
```

- CASE 식의 작동 : 
  - 처음에 있는 WHEN 구의 평가식부터 평가되고 조건이 맞으면 THEN 구에서 지정한 식이 리턴되며 CASE 식 전체가 종료됩니다. 만약 조건이 맞지 않으면
  - 다음 WHEN 구로 이동해 같은 처리를 반복합니다. 마지막 WHEN 구까지 반복 했는데도 조건이 맞는 경우가 없다면 'ELSE'에서 지정한 식이 리턴되며 끝납니다.
``` sql
SELECT name, address,
    CASE WHEN address = '서울시' THEN '경기'
         WHEN address = '인천시' THEN '경기'
         WHEN address = '부산시' THEN '영남'
         WHEN address = '속초시' THEN '관동'
         WHEN address = '서귀포시' THEN '호남'
         ELSE NULL END AS distirict
 FROM Address;
```
  - CASE 식의 강력함 점은 '식'이라는 것 -> 식을 적을 수 있는 곳이라면 어디든지 적을 수 있다.(SELECT, WHERE, GROUP BY, HAVING, ORDER BY)
  - SQL의 성능과도 굉장히 큰 관련이 있다.

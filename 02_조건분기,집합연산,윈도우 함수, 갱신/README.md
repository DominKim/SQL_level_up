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
  - ELSE 생략 시 ELSE NULL이 

2. SQL의 집합연산
- UNION으로 합집합 구하기 : 
  - UNION은 합집합을 구할 때 중복된 레코드를 제거( * 중복을 제외하고 싶지 않다면 UNION ALL 옵션 사용)
``` sql
SELECT *
    FROM Address
UNION
SELECT *
    FROM Address2;
```

- INTERSECT로 교집합 구하기 : 
``` sql
SELECT *
    FROM Address
INTERSECT
SELECT *
    FROM Address2;
```

- EXECPT로 차집합 구하기 :
  - 집합의 뺄셈 연산에서도 교환 밥칙이 성립하지 않기 때문에 주의가 필요하다.(순서가 중요)
  - ORACLE : MINUS
``` sql
SELECT *
    FROM Address
MINUS
SELECT *
    FROM Address2;
```

3. 윈도우 함수
- 집약 기능이 없는 GROUP BY 구
- 집약 함수 뒤에 OVER 구를 작성하고, 내부에 자를 키를 지정하는 PARTITION BY 또는 ORDER BY를 입력하는 것
- RANK() 함수 : 지정된 키로 레코드에 순위를 붙이는 함수
- DENSE_RANK() 함수 : RANK() 함수는 숫자가 같으면 같은 순위로 표시하고 건너 뛰고 표시하는데 건너뛰는 작업 없이 순위를 구할 때 사용
``` sql
SELECT address,
       COUNT(*) OVER (PARTITION BY address)
    FROM Address;
    
SELECT name,
       age,
       RANK() OVER (ORDER BY age DESC) AS rnk
    FROM Address;
    
SELECT name,
       age,
       DENSE_RANK() OVER (ORDER BY age DESC) AS dense_rnk
    FROM Address;
```

4. 트랜잭션과 갱신
- INSERT로 데이터 삽입 :
  - 필드 리스트와 값 리스트는 같은 순서로 대응하게 입력해야 한다.(* 잘못 입력하면 오류 발생)
  - 레코드(행) : 데이터를 등록하는 단위
  - NULL : ''로 입력하면 문자열로 인식하기 때문에 ''없이 입력
- 기본적인 INSERT 구문
``` sql
INSERT INTO [테이블 이름] ([필드1], [필드2], [필드3] ....)
               VALUES([값1], [값2], [값3] ....);

INSERT INTO Address (name, phone_nbr, address, sex, age)
             VALUES ('小川', '080-3333-XXXX', '서울시', '남', 30);
```

- DELETE로 데이터 제거 : 
  - 삭제 대상이 레코드이기 때문에 필드 제거는 오류 발생 (* 기호도 오류)
  - WHERE 구문을 사용하여 일부 데이터 제거 가능
  - DROP TABLE : 테이블 자체를 삭제할 떄 사용
- DELETE 구문의 기본구조
``` sql
DELECT FROM [테이블 이름];

DELETE FROM Address 
    WHERE address = '인천시';
```

- UPDATE로 데이터 갱신 :
  - 테이블의 데이터 갱신 시 사용

- UDPATE 구문의 기본 구조
``` sql
UPDATE [테이블 이름]
  SET [필드 이름] = [식];
  
UPDATE Address
    SET phone_nbr = '080-5848-XXXX',
        age  = 20
    WHERE name = '인성';
```

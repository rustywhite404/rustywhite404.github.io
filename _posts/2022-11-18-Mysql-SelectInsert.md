---
title: SELECT한 내용을 INSERT나 UPDATE시켜보자 
date: 2022-11-18 16:08:00
categories: SQL, MySQL  
published: true 
tags:
- SQL 
- MySQL 
---


같은 테이블 내에서 이미 존재하는 데이터를 한 번 더 삽입(또는 수정)해야 하는 경우가 간혹 생긴다.  
어떻게 해야 할 지 버벅거리지 않기 위해 정리해둔다.    
<br />

### SELECT한 내용을 그대로 다 INSERT 

```SQL 
INSERT INTO 테이블명 SELECT * FROM 테이블명 WHERE 조건;  
```  

보통 PK가 있으므로 원하는 컬럼만 SELECT 해서 INSERT 하게 된다.

---  
### 원하는 컬럼만 SELECT해서 INSERT 

```SQL  
INSERT INTO 테이블명 (컬럼1, 컬럼2, 컬럼3...) SELECT (컬럼1, 컬럼2, 컬럼3) FROM 테이블명 WHERE 조건;
```

---

### SELECT한 내용을 그대로 다 UPDATE  
솔직히 진짜 번거롭다 꼭 UPDATE가 필요한 상황이 아니라면 차라리 SELECT 결과를 한 번 더 INSERT해서 그 값을 수정하는 게 편할지도.  
```SQL  
 UPDATE 테이블명
       SET 수정할컬럼 = (SELECT * FROM(
                    SELECT 가져올값이있는컬럼 FROM 테이블명 WHERE 가져올값이있는컬럼의조건) as A )
       WHERE 수정할컬럼의조건;
``` 
이렇게 설명해놓으니까 오히려 복잡해 보여서 예제도 같이 써 둔다.  
```SQL  
 UPDATE test_tbl
       SET subject = (SELECT * FROM(
                    SELECT subject FROM test_tbl WHERE bno = 1) as A )
       WHERE bno = 101;
```  
test_tbl에서 1번글의 subject를 조회해서, 101번 글의 subject에도 덮어씌우는 예제다.  
SET ``subject =`` 부분에 바로 SELECT문을 넣을 수 없어서 한 번 감싸준다.  



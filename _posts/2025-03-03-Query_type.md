---
title: Native Query, Query Method, QueryDSL, JPQL 차이와 장단점 알아보기                 
date: 2025-03-03 12:08:14
categories: JPA Spring            
published: true 
tags:
- JPA           
- Spring 
---  

MyBatis를 쓸 때는 항상 SQL을 작성해서 사용해야 했지만 JPA를 사용하면 몇 가지 선택지가 생긴다. `Native Query, Query Method, QueryDSL, JPQL` 인데, 이 각각의 방식들이 어떻게 생겼고 장단점은 뭔지, 어떤 경우에 사용하면 좋은지에 대해 정리해보자. 
 

## 📌Native Query  
말 그대로 SQL을 직접 작성해서 실행하는 방식이다.  

```java 
@Query(value = "SELECT * FROM product WHERE price > :price", nativeQuery = true)
List<Product> findExpensiveProducts(@Param("price") int price);
``` 

✔️ **장점** 
- 복잡한 쿼리나 DB 인덱스를 활용 가능하므로 **성능 최적화**를 도모할 수 있다.  
- 따라서 대용량 데이터 처리 시에도 다른 매핑 기술보다 유리하다.  

❌ **단점**  
- DB 종속적이다 → DB마다 SQL 문법이 조금씩 다를 수 있으므로 DB 변경 시 문제가 생길 수 있다.  
- SQL을 직접 작성해야 해서 코드 가독성이 낮아진다. 

--- 

## 📌Query Method   
JPA가 메서드명을 보고 자동으로 SQL을 생성하는 방식이다. 

```java  
List<Product> findByNameAndPriceGreaterThan(String name, int price);
``` 
✔️ **장점** 
- 간단한 CRUD 작업을 할 때 무척 편리하다. 코드가 짧고, 메서드명만 작성하면 된다. 
- JPA에서 자동으로 SQL을 생성하기 때문에 특정 DB 언어에 종속적이지 않다. 

❌ **단점**
- 조건이 많아지면 메서드명이 너ㅡ무 길어지고, 복잡한 조건 처리가 어렵다. 
- 성능 최적화가 어렵다.  

--- 

## 📌JPQL (Java Persistence Query Language)  
SQL과 비슷하지만 엔티티 중심으로 작성하는 JPA 표준 쿼리. 

```java  
@Query("SELECT p FROM Product p WHERE p.price > :price")
List<Product> findExpensiveProducts(@Param("price") int price); 
``` 
✔️ **장점**
- 객체 중심 쿼리 → SQL과 유사해서 이해하기 쉽다.    
- DB 독립적이라 특정 DB에 종속되지 않는다.  

❌ **단점** 
- 문자열 기반이라 오타나 버그를 컴파일 단계에서 잡기 어렵다. 
- 동적 쿼리 불편 → if로 조건 추가하려면 StringBuilder 등 별도 로직이 필요하다.  

---

## 📌QueryDSL  
동적 쿼리를 메서드 체인 방식으로 객체처럼 작성하는 방식이다.  
```java 
QProduct product = QProduct.product;

List<Product> result = queryFactory
    .selectFrom(product)
    .where(product.name.eq("상품명").and(product.price.gt(10000)))
    .fetch();
``` 
✔️ **장점**
- 타입 안전 → 컴파일 시 오류를 잡아준다(JPQL은 문자열이라 런타임 오류 위험이 있다). 
- 동적 쿼리 작성이 편리하다. 조건문을 if로 추가할 수 있어서 가독성이 좋음. 
- 페이징, 정렬을 지원한다. fetchFirst(), orderBy() 등을 간편하게 사용 가능. 

❌ **단점** 
- QueryDSL 라이브러리 추가, Q클래스 생성이 필요하다. 
- JPQL보다 개념이 복잡해서 따로 학습이 필요하다. 

---

## 📌어떨 때 활용하는 게 좋을까?  
| 상황 | 추천방식 |
| --- | --- | 
| 간단한 조회 (예: ID로 찾기, 단순 검색) | 쿼리 메서드 | 
| 복잡한 동적 쿼리 (검색 조건 많음) | QueryDSL | 
| 복잡한 SQL 최적화 필요 (인덱스, JOIN 최적화) | 네이티브 쿼리 | 
| JPA 표준 방식으로 작성하고 싶을 때 | JPQL | 
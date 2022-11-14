---
title: MySQL 쿼리 조회 결과 수직으로 보기      
date: 2022-10-12 12:30:00
categories: MySQL 
published: true 
tags:
- MySQL 
- Linux
---


컬럼이 아주 많은 테이블을 조회하면 이런 식으로 정신이 아득해지는 조회 결과가 나온다.

![Untitled](https://i.imgur.com/9I9sfcp.png)

확인해야 하는 컬럼 수가 적으면 특정 컬럼만 SELECT 하면 되겠지만 전체 컬럼을 보면서 대조해야 하는 경우 눈앞이 깜깜해진다. 그럴 때 유용한 기능이 쿼리 결과를 수직으로 볼 수 있는 옵션이다. 방법은 두 가지가 있다. 

1. 특정 조회 결과만 수직으로 보고 싶을 때 
    
    <aside>
    📌 select * from 조회할 테이블 \G
    
    </aside>
    
    위와 같이 ; 대신 \G를 붙여서 조회하면 아래와 같이 깔끔하게 확인이 가능하다. 
    
    ![Untitled](https://i.imgur.com/uWQyigy.png)
    
2. 모든 조회 결과가 수직으로 나오게 하고 싶을 때 
    
    <aside>
    📌 mysql -E -u아이디 -p 로 로그인
    
    </aside>
---
title: 리눅스 환경에서 MySQL 데이터 백업 및 복구하기     
date: 2022-03-16 13:11:12
categories: SQL, MySQL, Linux 
tags:
- MySQL
- SQL
- Linux
---

## 리눅스 환경에서 MySQL 데이터 백업 및 복구하기     
한 번 DB를 백업하거나 복구 해야 할 일이 있다. 워크벤치 등의 툴에서 편하게 하는 방법도 있지만 리눅스 환경에서만 가능한 상황도 있기 때문에 리눅스 환경에서 백업 및 복구 하는 법을 기록해두려 한다. 

## MySQL 데이터 백업하기   
`스키마`만 백업 하는 방법과 `스키마와 데이터를 함께` 백업 하는 방법, `스키마와 데이터, 프로시저, 트리거 까지` 백업 하는 방법을 정리해보자. 전부 mysql에 접속하지 않고 root에서 입력하면 된다. -p만 써도 되고 -p[패스워드] 까지 바로 써도 되는데, -p[패스워드]를 쓸 때는 -p 뒤에 스페이스 쓰지 말고 패스워드를 붙여 써야 정상적으로 수행 된다(ex. -p1234).  

**1. 스키마(구조)만 백업하는 방법**

```linux   
mysqldump -u[유저이름] -p[패스워드] -d [DB명] > [파일명:ex. db_name_backup.sql]

예시) 
mysqldump -uMYNAME -p -d db_name > db_name_backup.sql 
```

**2. 스키마와 데이터를 함께 백업하는 방법**

```linux 
mysqldump -u[유저이름] -p[패스워드] [DB명] > [파일명:ex. db_name_backup.sql]

예시) 
mysqldump -uMYNAME -p db_name > db_name_backup.sql 
```  
  
**3. 스키마, procedure, function, trigger 포함하여 백업하기**
스키마+데이터를 함께 백업하는 2번 방법만으로는 `트리거나 프로시저가 백업되지 않는다.` 트리거나 프로시저가 포함되어 있는 DB라면 꼭 이 옵션으로 백업하자.  

```linux  
mysqldump --routines  --triggers -u[유저이름] -p[패스워드] [DB명] >  [파일명:ex. dbbackup.sql]  

예시) 
mysqldump --routines --triggers -uMYNAME -p db_name > db_name_backup.sql 
```  

## MySQL 데이터 복구하기   
저장해 둔 sql 또는 txt 파일을 mysql에 복구해보자. 데이터가 들어갈 스키마는 미리 생성을 해 두어야 한다.  
```
mysql -u[유저이름] -p[패스워드] [DB명] < [파일명.sql]


예시)
mysql -uMYNAME -p1234 db_name < db_backup.sql  
```  
데이터 복구는 `mysqldump`가 아니라 `mysql`로 한다는 것과 꺽쇠 방향에 주의할 것 .  


---
title: 자주 쓰이는 날짜 형식 변환 모음
date: 2022-11-15 00:21:00
categories: JAVA  
published: true 
tags:
- JAVA 
---


Date를 String으로 변환하거나 String을 Date로 변환해야 할 때가 많다. 한 번씩 헷갈릴 때가 있어 여러 번 검색하지 않으려고 이 기회에 정리해두기로 했다.  
<br />

### Date형 날짜를 원하는 형식으로 변환하고 싶을 때

```java
Date 오늘 = new Date();
SimpleDateFormat 원하는형식 = new SimpleDateFormat("yyyy년 MM월 dd일");
System.out.println("1. "+원하는형식.format(오늘));
```

---

### String을 원하는 형식의 Date(DateTime)형으로 변환하고 싶을 때

```java
String 입력된날짜 = "2023/12/25 21:03:08";
DateFormat 입력된날짜형식 = new SimpleDateFormat("yyyy/MM/dd HH:mm:ss");
DateFormat 바꿀날짜형식 = new SimpleDateFormat("yyyyMMdd");	
try {
	Date day = 입력된날짜형식.parse(입력된날짜);
	System.out.println("2. "+바꿀날짜형식.format(day));
} catch (ParseException e) {		
}
```

---

### String을 LocalDateTime으로 바꾸고 싶을 때

```java
DateTimeFormatter 정해진형식 = DateTimeFormatter.ofPattern("yyyy/MM/dd HH:mm:ss");
LocalDateTime date = LocalDateTime.parse(입력된날짜,정해진형식);
System.out.println("3. "+date);
```

---

### LocalDateTime을 사용한다면

DateTimeFormatter를 사용해서 미리 설정되어 있는 포맷으로 쉽게 바꿀 수도 있다.

```java
LocalDateTime 지금 = LocalDateTime.now();
		
DateTimeFormatter 포맷1 = DateTimeFormatter.ISO_DATE; // 2022-11-15
DateTimeFormatter 포맷2 = DateTimeFormatter.BASIC_ISO_DATE; // 20221115
DateTimeFormatter 포맷3 = DateTimeFormatter.ISO_LOCAL_TIME; // 00:03:29.7379555
DateTimeFormatter 포맷4 = DateTimeFormatter.ISO_LOCAL_DATE_TIME; // 2022-11-15T00:03:29.7379555

System.out.println("4-1. "+지금.format(포맷1));
System.out.println("4-2. "+지금.format(포맷2));
System.out.println("4-3. "+지금.format(포맷3));
System.out.println("4-4. "+지금.format(포맷4));
```

---

### Date를 String으로 변환하고 싶을 때

```java
Date 오늘 = new Date();
SimpleDateFormat 바꿀형식 = new SimpleDateFormat("yyyy-MM-dd");
String 문자열 = 바꿀형식.format(오늘);
System.out.println("5. "+문자열);
```
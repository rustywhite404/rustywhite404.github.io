---
title: (JAVA) 문자열을 나누는 split(), subString()    
date: 2021-02-25 16:21:00
categories: JAVA
---

<p style="color:#999;">※ 제가 편하게 복습하려고 구어체로 작성했습니다.<br />
※ 초보 개발자의 공부 겸 정리 포스팅이라 틀린 부분이 있을 수 있습니다. </p>

### 문자열을 나눠주는 split()  
```java
String str = "서울,대전,대구,부산";
String arr = str.split(",");
for(String i:arr){
	System.out.println(i);
}

// 결과값 : 
서울
대전
대구
부산
```
split()은 위와 같이 괄호 안의 값을 기준으로 문자열을 나눠 줘.  
```java
String str = "010-1234-5678";
String arr[] = str.split("-");

arr[1] = 010
arr[2] = 1234
arr[3] = 5678
```
---  
### 문자열을 나눠주는 subString()
```java
String num = "12345678";
```
이런 변수에서 특정 부분만 제거하고 문자열을 추출하고 싶을 때는 split을 쓰기 힘들지?  
그럴 때 유용하게 사용할 수 있는 게 subString이야.  
subString(추출할 시작번호 인덱스, 끝번호 인덱스) 로 사용할 수 있어.  
```java
String num2 = num.subString(2,7); // 2번부터 7번까지. 0부터 시작하는 거 기억하기 
```

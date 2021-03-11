---
title: 타임리프Thymeleaf 문법을 알아보자 1    
date: 2021-03-09 22:11:00
categories: Spring, SpringBoot, Thymeleaf
tags:
- Spring 
- Thymeleaf 
--- 
## 기본 사용법  
  
| 종류 | 사용법 | 예시 |  
|:--- | :--- | :--- |  
| 변수 표현 | ${...} | ${session.user} |  
| 선택자 | *{...} | *{firstName} |   
| 메시지 | #{...} | #{message} |  
| 링크 | @{...} | @{abbo.tistory.com} |  
  
--- 
## 자주 사용되는 타임리프 문법  
표현식은 기본적으로 `th:[속성]="전달받은 값 또는 조건식"` 의 형태로 표현됩니다.  
  
| 종류 | 사용법 | 설명 |  
|:--- | :--- | :--- |  
| Text | `<span th:text="${name}"></span>` | 서버에서 받아온 값을 html 텍스트로 렌더링 |  
| Value | th:value=${name} | input이나 select, button등의 태그 값에 서버에서 받은 값을 렌더링 |  
| errors | ` <li th:errors="*{id}" />` | 해당 값에 error가 있는 경우 출력한다. |  
| with | `<div th:with="var1=${user.name}+' is user name'" th:text="${var1}"></div>` | 변수값을 지정해서 사용한다. |   
  
--- 

## if/else  
    
| 종류 | 사용법 | 설명 |  
|:--- | :--- | :--- |  
| if | `<div th:if="${error}">에러발생</div> ` | 조건문처럼 사용한다. 해당 조건이 만족할 때만 출력된다. |  
| else | `<p th:unless="${user.authType}=='facebook'" th:text="'not '+ ${user.authType}"></p>` | else |  
  
---  

## for  
   
| 종류 | 사용법 | 설명 |  
|:--- | :--- | :--- |  
| for | `<p th:each="user : ${users}" th:text="${user.name}"></p>` | for each문. list 안에 있는 객체가 obj로 맵핑된다 |  
  

**사용 예제**  
Table에서 사용시, 아래와 코드와 같이 혼합해서 사용이 가능합니다.  
${list}객체를 user가 받아 user 클래스 안에 있는 프로퍼티스 접근합니다.   

```
<tbody id="docsTr">
     <tr th:if="${size} == '0'">
		<td colspan=2>데이터가 없습니다.</td>
	 </tr>
     <tr th:unless="${size} == '0'" th:each="user : ${list}">
		 <td th:text="${user.index}">1</td>
		 <td th:text="${user.name}">Brian</td>
	</tr>
</tbody>
```
**for문으로 받아온 객체값을 자바스크립트 변수에 넣을 때**  
예) tr 태그 클릭시 화면이동 하고 싶을때, 특정 변수를 파라미터로 받아 값을 넘겨줄때!  
th:data 다음엔 아무이름이나 정해주면 된다. ( *th:data-별칭 ) 그리고 getAttribute로 loadDetailView 함수 파라미터에 값을 넣어준다.  
```    
<tbody id="docsTr">
     <tr th:if="${size} == '0'">
		<td colspan=2>데이터가 없습니다.</td>
	 </tr>
     <tr th:unless="${size} == '0'" th:each="user : ${list}"
     th:data-index="${user.index}"
     th:onclick="loadDetailView(this.getAttribute('data-index'))">
		<td th:text="${user.index}">1</td>
		<td th:text="${user.name}">Brian</td>
	</tr>
</tbody>
```

**요소 하나하나를 반복하면서 데이터를 뷰로 표현할 때**  
```html  
<div th:if="${family_group != null}">
    <div class="group_list_sub" th:each="group : ${family_group}">
        <h2 th:text="#{group.name}"></h2>
        <ul>
            <li>
                <a th:href="@{'/community/member/' + ${group.num}">
                    <img th:src="${group.thumbnail != null ? group.thumbnail : '/images/group_user.svg'}"/>
                </a>
            </li>
        </ul>
    </div>
</div>
```

---  

## 배열/리스트  
```html    
<img th:if="${#lists.size(diary.files) > 0}" th:src="${diary.files[0]}"/>
```  
위 예제처럼 서버로부터 넘어온 diary.files 배열 데이터를 if문을 사용해 사이즈 체크한다. diary.files[0]처럼 특정 인덱스로 접근도 가능하다.   
[그 밖에 사용 가능한 list 메소드](https://www.thymeleaf.org/apidocs/thymeleaf/3.0.0.RELEASE/index.html?org/thymeleaf/expression/Lists.html)

## switch case  
```html  
<div th:switch="${eventFvrDtl.fvrDvsCd}" class="coupon_gift">
    <p th:case="${#foConstants.FVR_DVS_CD_DISCOUNT}" class="cp cptype02">
        <th:block th:switch="${fvrKnd2Cd}">
            <span th:case="${#foConstants.FVR_KND2_CD_FREE_DLV_CPN}" th:text="${fvrKnd2CdNm}" class="tx"></span>
            <span th:case="*" class="tx">
                <th:block th:text="${fvrKnd2CdNm}"></th:block>
            </span>
        </th:block>
    </p>
    <p th:case="${#foConstants.FVR_DVS_CD_ACCUMULATION}" class="cp cptype02">
        <span class="ty" th:text="${fvrKnd2CdNm}"></span>
        <span class="tx" th:text="${#numbers.formatInteger(eventFvrDtl.fvrDtlCnts, 3, 'COMMA')}"></span>
    </p>
    <p th:case="*" class="cp cptype02">
        <span class="tx" th:text="${eventFvrDtl.fvrDtlCnts}"></span>
    </p>
</div>
``` 
위 예제처럼, fvrDvsCd값이 이럴 때 case1, case2로 빠지고 그 외의 값은 th:case=* 로 빠지게 된다.  

## equals 
```HTML 
<img th:if="${thumbnail == null}" th:src="${#strings.equals(gender, 'male') ? '/images/male.png':'/images/female.png'}" width="auto" height="auto"/>
```  
위 예제처럼, #strings.equals(string1, string2) 메소드로 문자열 두 개가 같은지 비교할 수 있다.  
[그 밖에 사용 가능한 strings 메소드](https://www.thymeleaf.org/apidocs/thymeleaf/2.1.4.RELEASE/org/thymeleaf/expression/Strings.html)

## form  
  
| 종류 | 설명 |  
|:--- | :--- |    
| th:action | form 태그에서 해당 경로로 요청을 보낼 때 사용 |  
| th:object | submit을 할 때, form의 데이터가 th:object에 설정해준 객체로 받아진다. |  
| th:field | 각각 필드들을 매핑을 해주는 역할을 한다. 설정해 준 값으로, th:object에 설정해 준 객체의 내부와 매칭해준다. |  
  
```html  
 <form th:action="@{/sign-up}" th:object="${signUpForm}" method="post">
    <div class="form-group">
        <label for="nickname">닉네임</label>
        <input id="nickname" type="text" th:field="*{nickname}" class="form-control">
    </div>
</form>
```

## select  
```html  
<select class="form-control" id="goods" name="goods">
    <option th:each="goodList : ${goodList}" th:value="${goodList.code}" th:text="${goodList.name}"></option>
</select>
```
이런 식으로, each 문법을 사용해 select 옵션을 지정할 수 있다.

## url 관련  
  
| 종류 | 사용법 | 설명 |  
|:--- | :--- | :--- |  
| URL 연결 | `<a th:href="@{/**/**}">` | 서버 내의 특정 위치로 이동 |  
| URL + 파라미터 | `<a th:href="@{/**/**(page=1)}">` | 고정적인 값을 넘길 때 |  
| URL + 파라미터 | `<a th:href="@{/**/**(page=${list.num})">` | 서버에서 받아온 값을 넘길때 |  
  
---  

## Utility  
  
| 종류 | 사용법 |  
|:--- | :--- |  
| 캘린더 객체 사용 | `<td th:text="${#calendars.format(#calendars.createNow(), 'yyyy-MM-dd HH:mm')}"></td>` |  
| listJoin 객체 사용 | `th:value="${#strings.listJoin(tags, ',')}"` |  
  

> 위 문법은 아래의 코드를 구현한 것이다.  

```java    
List<String> tags = List.of("Spring", "JPA");
=> Spring, JPA
```
---
title: 자바스크립트, 제이쿼리를 이용한 체크박스 전체 선택 및 전체 해제    
date: 2022-02-20 13:36:00 
published: true 
categories: Javascript 
tags:
- Javascript 
---

## 체크박스 전체 선택, 전체 해제       
웹 페이지를 만들다 보면 전체 체크 박스를 선택하거나 해제해야 하는 일이 은근히 많다. 상단의 체크박스를 누르면 거기에 맞춰서 전체 선택, 해제 되도록 처리해보자.  
<br />  
<table border="1" style="border-spacing:0;">
  <tr>
    <th><input type="checkbox" onclick="test(this)" /></th>
    <th>글번호</th>
    <th>제목</th>
  </tr>
  <tr>
    <td><input type="checkbox" name="chk" value="1" /></td>
    <td>1</td>
    <td>피카츄</td>
  </tr>
  <tr>
    <td><input type="checkbox" name="chk" value="2" /></td>
    <td>2</td>
    <td>라이츄</td>
  </tr>
  <tr>
    <td><input type="checkbox" name="chk" value="3" /></td>
    <td>3</td>
    <td>파이리</td>
  </tr>
  <tr>
    <td><input type="checkbox" name="chk" value="4" /></td>
    <td>4</td>
    <td>꼬부기</td>
  </tr>		
</table>

```html  
	<table border="1" style="border-spacing:0;">
		<tr>
			<th><input type="checkbox" onclick="test(this)" /></th>
			<th>글번호</th>
			<th>제목</th>
		</tr>
		<tr>
			<td><input type="checkbox" name="chk" value="1" /></td>
			<td>1</td>
			<td>피카츄</td>
		</tr>
		<tr>
			<td><input type="checkbox" name="chk" value="2" /></td>
			<td>2</td>
			<td>라이츄</td>
		</tr>
		<tr>
			<td><input type="checkbox" name="chk" value="3" /></td>
			<td>3</td>
			<td>파이리</td>
		</tr>
		<tr>
			<td><input type="checkbox" name="chk" value="4" /></td>
			<td>4</td>
			<td>꼬부기</td>
		</tr>		
	</table>
```
```javaScript 
<script type="text/javascript">
function test(o) {
	//클릭한 체크박스의 table에서 이름이 chk인 것을 찾고, 
	//현재 요소의 체크 상태를 찾은 대상에 적용
	$(o).closest('table').find('[name=chk]').prop('checked', o.checked);
}
</script>
```

## 버튼으로 처리할 경우  
다음과 같은 조건을 조건해야 한다.  
1. 하나라도 체크가 되어 있지 않다면 전체 체크되도록 해야함
2. 모두 체크되어 있다면 다시 체크가 전부 해제되도록 해야함  
<br />  
<div class="container">
	    <input type="checkbox" />이메일 수신 동의
	    <input type="checkbox" />문자 수신 동의
	    <input type="checkbox" />개인정보 동의
	 </div>
	 <button class="agree-btn">전체동의</button>  
<br />  

```html  
<body>
	<div class="container">
	    <input type="checkbox" />이메일 수신 동의
	    <input type="checkbox" />문자 수신 동의
	    <input type="checkbox" />개인정보 동의
	 </div>
	 <button class="agree-btn">전체동의</button>
</body>
```
```javaScript  
<script>
    const $container = document.querySelector('.container');
    const $inputs = [...$container.children];
    const $agreeBtn = document.querySelector('.agree-btn');

    $agreeBtn.onclick = () => {
      if ($inputs.filter(input => input.checked).length === $inputs.length) {
        $inputs.forEach(input => { input.checked = false; });
      } else {
        $inputs.forEach(input => {
          input.checked = true;
        });
      }
    };
</script>
```
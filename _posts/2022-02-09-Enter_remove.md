---
title: 연속된 줄바꿈(엔터) 한 번으로 바꾸기(자바스크립트)   
date: 2022-02-09 12:39:00
categories: CSS, Javascript 
published: true 
tags:
- CSS  
- Javascript  
---

## 연속된 줄바꿈(엔터) 한 번으로 바꾸기  
사용자가 Textarea에 너무 엔터를 많이 쳐 놓았을 경우나, 글을 복사해서 붙여넣었는데 엔터가 너무 많이 포함되어 있을 경우 등 여러가지 케이스에서 줄바꿈을 한 번으로 변경하고 싶어질 수 있다. 그럴 때 유용하게 사용할 수 있는 방법이다. 
<br/>

```html  
<body>
	<div class="box">
		<textarea rows="8" cols="50" id="book">
      개나리 노란 꽃그늘 아래

      가지런히 놓여있는



      꼬까신




      하나 
  </textarea>
  <button type="button" onclick="change()">엔터 없애기</button>
</div> 
```  

```javascript 
<script type="text/javascript">
		function change() {
			var txt = document.getElementById('book').value;
			// txt에 book 안에 있는 값을 받아옴 
			
			// 정규식을 이용해서 엔터로만 끝나는 줄을 없앰
			// 정규식 m 옵션을 사용하여 엔터를 포함한 여러 줄을 대상으로 한다
			// g : global - 전체를 대사으로 함. 없으면 하나의 일치만 실행
			// m : multi line - 엔터를 포함하여 정규식 실행
			// \n$ - 엔터(줄바꿈)로만 끝나는 것 찾기. m 옵션이 있어서 가능함 
			
			txt = txt.replace(/\n$/gm, '');
			document.getElementById('book').value = txt; 
		}	
</script>
```



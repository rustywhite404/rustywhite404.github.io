---
title: innerHTML, innerText, outerHTML, outerText 은 어떻게 다른가    
date: 2022-01-31 21:28:30
categories: Javascript, Html 
tags:
- Javascript
- Html 
---

## innerHTML, innerText, outerHTML, outerText는 어떻게 다를까    
- innerHTML : 지정한 요소의 태그를 제외한 안쪽 태그만 값을 가져오고 처리(자기 자신은 미포함)  
- innerTEXT : 자기 자신을 미포함 한 텍스트를 변경한다.  
- outerHTML : 지정한 요소의 태그를 포함하여 가져오고 처리(자기 자신 포함)  
- outerTEXT : 자기 자신을 포함한 텍스트를 변경한다.  

※ `<table>, <td>, <div>, <span>` 등에만 사용할 수 있고, `<input>, <img>`는 안 된다. 

---  
<br /> 

```html  
<body>
	<a href="javascript:change()">내용물 바꾸기</a> 
	<div id="box">TOY BOX</div>
</body>
```  
위 코드를 변경해보자. 
<br />
1. innerHTML 
```html  
<script type="text/javascript">
	function change() {
		document.all.box.innerHTML ="<span style='color:#990000'>APPLE BOX</span>";
	}
</script>
```  
결과  
![이미지](https://i.imgur.com/MTQ6i8P.png)  
<br />
2. innerText  
결과  
![이미지](https://i.imgur.com/scGhf9Q.png)   
<br />
3. outerHTML 
```html  
<script type="text/javascript">
	function change() {
		document.all.box.outerHTML ="<span style='color:#990000'>APPLE BOX</span>";
	}
</script>  
```  
결과  
![이미지](https://i.imgur.com/Oq1M1i8.png)   
<br />
4. outerText  
결과  
![이미지](https://i.imgur.com/HMfd6oY.png)  
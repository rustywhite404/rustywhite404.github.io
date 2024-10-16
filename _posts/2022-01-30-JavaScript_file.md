---
title: 자바스크립트로 업로드 파일 확장자 체크하기 + 더 쉬운 방법    
date: 2022-01-30 11:40:00
categories: Javascript Html 
tags:
- Javascript
- Html 
---

## 자바스크립트로 업로드 한 파일 확장자 체크하기   
```html  
<input type="file" name="file1" onchange="checkFile(this)">
```  
위와 같은 input 파일 입력폼에 들어온 확장자를 체크해보자. 

```javascript
<script type="text/javascript">
	function checkFile(f) {
		
		//files로 해당 파일 정보 얻기
		var file = f.files;
		
		// 정규식으로 확장자 체크
		if(!/\.(gif|png|jpg)$/i.test(file[0].name)){
			alert('git, png, jpg만 선택하세요.');
			
			// 체크에 걸리면 선택된 내용을 취소해야 함.
			// 파일선택 폼의 내용은 스크립트로 컨트롤 할 수 없으므로 
			// 폼을 초기화 하면 된다.
			f.outerHTML = f.outerHTML;
		}else{
			// 체크를 통과하면 종료 
			return;
		}		
	}
</script>
```  
위와 같은 방식으로 정규식을 사용하여 체크 할 수 있다.  

---  

## 더 쉽게 업로드 한 파일 확장자 체크하기   
이 파일이 어떤 확장자인지 반드시 확인이 필요한 경우가 아니라, 업로드 시 필터링이 필요한 거라면 `input accept`를 사용하여 훨씬 간단하게도 확장자를 체크 할 수 있다. 
```html  
<input type="file" accept="image/*" id="ex">  
```  
이렇게 accept에서 파일 타입을 미리 정해두면  이미지, 동영상, 음악 파일 등의 형식을 정해서 받을 수 있다. 
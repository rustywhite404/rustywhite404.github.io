---
title: (SPRING)redirect 할 때 parameter 넘기는 법   
date: 2022-02-24 17:54:00
categories: Spring 
published: true 
tags:
- Spring  
---

## 스프링에서 redirect 할 때 parameter 넘기기

스프링 컨트롤러에서 redirect를 해야 하는 경우가 종종 있다. 이 경우에는 평소의 GET/POST 전송을 할 때처럼 model attribute를 사용하기가 힘들다. 어떻게 해야 할까?  
![이미지](https://i.imgur.com/D8PXj6I.png)  
(이상한 짓 하지 말자)  

---

### RedirectAttribute를 사용해보자   
![이미지](https://i.imgur.com/rUHvshZ.png) 
데이터를 넘겨야 하는 쪽에서, 먼저 이런식으로 `RedirectAttributes` 를 사용하여 값을 넘길 준비를 해 준다. `addAttributes`를 사용하면 보통의 GET 방식처럼 URL에 값을 포함하여 날리게 된다. `addFlashAttribute`를 사용하면 POST 방식처럼 URL에 값을 포함하지 않고 전송할 수 있지만 `1회성 전송이기 때문에 한 번 보내고 나면 값은 지워진다`.  나는 그냥 전자를 사용했다.  

![이미지](https://i.imgur.com/tbqUT9D.png)  
받는 쪽에서는 이렇게 `@RequestParam`을 사용해서 전달 된 값을 사용할 수 있다.  

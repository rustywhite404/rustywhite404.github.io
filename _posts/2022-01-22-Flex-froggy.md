---
title: FLEXBOX FROGGY 24번 문제    
date: 2022-01-23 18:46:00
categories: CSS 
tags:
- CSS
---

## CSS FLEX 연습 할 때 유용한 개구리 게임      
![이미지](https://i.imgur.com/yeNVaAC.png)  
FLEX는 다양한 속성들을 가지고 있어서 막 배운 직후에는 헷갈리거나 어디에 어떤 속성을 써야 할 지 생각나지 않는 경우가 있다. 그럴 때 FLEX의 속성들을 연습해 볼 수 있는 미니게임 사이트가 있다. 개구리들을 같은 색 연꽃잎 위에 올려주는 게임이다. 난이도 자체는 어렵지 않고, 여러 번 따라하다 보면 각 속성들의 특징도 쉽게 외울 수 있다. 

[FLEXBOX FROGGY 사이트 바로가기](https://flexboxfroggy.com/#ko)  

<br/>

## 개구리 게임 24번 문제 답   
![이미지](https://i.imgur.com/BnyP4RP.png) 
23번 까지는 별 생각 없이 풀 수 있었는데, 24번은 생각을 좀 해서 풀어야 했으므로 정답을 맞추는 과정과 답을 기록해둔다.  

![이미지](https://i.imgur.com/SfKECyu.png) 
1. 우선 `flex-wrap:wrap-reverse`를 사용해서 한 줄에 비좁게 들어가 있던 개구리들을 두 줄로 만들어준다. wrap이 아니라 wrap-reverse를 쓴 이유는 빨간 개구리의 순서 때문이다. 위 그림에서, 개구리들 전체를 왼쪽으로 90도 회전시키면 우선 색깔 배치는 맞을 것 처럼 보인다. 

![이미지](https://i.imgur.com/4q4QRvT.png) 
2. 우선 `flex-direction:column-reverse`를 사용해서 개구리들을 세로 정렬 해준다. 역시 column이 아니라 column-reverse를 써서 빨간 개구리가 맨 아래로 오도록 해 주어야 한다. 이제 노란 개구리들을 세로축 중앙으로 옮기고, 모든 개구리들이 양 끝 정렬 되도록 바꿔주면 끝난다.  

![이미지](https://i.imgur.com/FKTO5br.png) 
3. `justify-content:center`를 사용해서 노란 개구리들을 중앙 정렬 해 주었다. 

![이미지](https://i.imgur.com/8e4QQki.png)
4. `align-content:space-between`을 사용해서 좌우 개구리들을 양끝 정렬 해주면 끝. 


```css  
#pond{
  display:flex;
  flex-wrap:wrap-reverse;
  flex-direction:column-reverse;
  justify-content:center;
  align-content:space-between;
}
```  
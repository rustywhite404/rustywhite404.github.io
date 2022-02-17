---
title: SVG 이미지로 CSS 라인 애니메이션 만들기    
date: 2022-02-16 15:27:00
published: true
categories: CSS
tags:
- CSS
---

## SVG 이미지로 라인 애니메이션 만들기      

![이미지](https://i.imgur.com/sXEmBGw.jpg)  


이런 SVG 라인 이미지를 사용해서 [이런 효과를 만들 수 있다](https://rustywhite404.github.io/svg_animation.html) 
<br/>

## 일단 SVG 이미지를 준비한다    
재료가 있어야 요리가 되는 법이다. 나는 일러스트레이터를 이용해서 내가 사용하고 싶은 SVG 이미지를 만들었다. 이 때 주의해야 할 건 PATH가 라인으로 된 이미지여야 한다는 거. 면으로 된 이미지(CREATE OUTLINE 등)로는 원하는 라인 애니메이션을 얻을 수 없다. `개별적으로 움직임을 주고 싶은 라인마다 레이어를 분리해서 저장`하고, 가급적 `레이어명`을 알아보기 쉬운 걸로 저장해두는 게 CSS 효과를 줄 때 더 편리해진다.   
SVG파일을 저장한 후에, 메모장이나 노트 패드로 파일을 열어보자. 
<br />
![이미지](https://i.imgur.com/ToSL8T7.png)  
대충 이렇게 생긴 복잡한 코드들이 보인다. `path id=어쩌고"` 하는 부분들이 보이면 제대로 레이어가 분리되어서 만들어 진 것이다. 위 이미지에서 hhh, ggg, eee등으로 보이는 id값들은 내가 일러스트레이터에서 레이어를 구분하기 만든 임의의 레이어명들이다. 레이어명을 따로 지정하지 않는다면 전부 none으로 뜨게 된다.  
위의 DOCTYPE이나 기타등등은 필요없고, 드래그 해 놓은 영역(SVG부터)만 가져다 사용하면 된다.  

## 준비한 SVG 에 라인 애니메이션 입혀보기   

우선 boxBG라는 SVG 이미지의 배경이 될 DIV에 색상을 입혀뒀고, 그 아래로는 각각의 PATH ID에 키프레임을 걸어 애니메이션 효과를 주면 된다. `stroke-dasharray`와 `stroke-dashoffset`는 해당 PATH의 길이에 따라 다르게 주어야 예쁘게 애니메이션이 나오므로, 개발자도구에서 봐가면서 하나하나 조절해주는 게 좋다. 

```css  
#boxBG{
    background-color: cadetblue;
}

#aaa{stroke-dasharray: 1200; stroke-dashoffset: -1200;animation: aaa-anim 2s linear .0s forwards;}
#bbb{stroke-dasharray: 1200; stroke-dashoffset: -1200;animation: bbb-anim 2s linear .0s forwards;}

@keyframes aaa-anim {
   from {stroke-dashoffset : -500px;}
   to {stroke-dashoffset : 0px;}
}

@keyframes bbb-anim {
    from {stroke-dashoffset : -500px;}
    to {stroke-dashoffset : 0px;}
 }

```  
로고 등에 그럴싸한 애니메이션 효과를 주고 싶은데 동영상 편집은 할 줄 모르거나 용량을 아끼고 싶다면 SVG로 애니메이션 효과를 줘 보는 것도 좋은 시도 같다. 
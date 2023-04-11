---
title: JDK 여러개를 편하게 전환하는 방법(자바 버전 변경)
date: 2023-04-11 13:20:00
categories: java   
published: true 
tags:
- java
- tip    
---

진행중인 프로젝트들이 어떤 건 자바8, 어떤 건 자바17을 사용하게 되어 JDK를 변경해야 했다.  
매번 환경 변수를 바꾸는 건 귀찮은 일이라 간편하게 전환 할 수 있는 방법을 찾다가 bat파일을 사용하는 팁을 찾아  따라해보았다. 

### 1. 같은 폴더 안에 JDK들을 설치해둔다.  
![이미지](https://i.imgur.com/6wzL8gY.png)  
나는 자바 1.8과 17을 사용할 예정이므로 각각의 JDK를 위 캡쳐처럼 올려두었다.  

### 2. 환경 변수를 설정한다 
시스템 속성 > 환경 변수로 이동하면 시스템 변수 목록이 보인다.  
기존에 자바를 사용하면서 JAVA_HOME으로 만들어 둔 변수가 이미 존재할 것이다.  

![이미지](https://i.imgur.com/rB3E98w.png)  

이 변수의 값을 우선 사용할 JDK가 들어있는 경로로 변경해준다. 

![이미지](https://i.imgur.com/Wyds0IO.png)  

그런 다음, Path 시스템 변수를 찾아 편집을 눌러준다.  
위 이미지처럼 `%JAVA_HOME%\bin`를 등록해주고 확인 버튼을 눌러 환경 변수 창들을 닫는다.  
아직 환경 변수 설정 단계가 하나 더 남았다.  

![이미지](https://i.imgur.com/bYEJSDw.png)  
JDK들이 들어있는 폴더에 다음 이미지처럼 scripts 폴더를 만든다.  
여기에 자바 버전을 변경해주는 bat파일을 만들 예정이다.  
그런 다음, 시스템 변수 영역의 `Path`에서 다시 편집을 누르고, scripts 폴더의 경로를 추가해준다. 내 경우에는 `C:\Program Files\Java\scripts` 이다. 
![이미지](https://i.imgur.com/U1skwUn.png)  


### 3. bat파일 만들기   
이제 메모장을 켜서 .bat 파일을 만들어야 한다. 사용할 JDK의 갯수만큼 bat파일을 만들게 된다. 나는 두 가지를 사용할 예정이므로 두 개만 만들 거다.  

[java8.bat 파일 내용]
``` 
@echo off
set JAVA_HOME=C:\Program Files\Java\jdk1.8.0_341
set Path=%JAVA_HOME%\bin;%Path%
echo Java 8 activated.
``` 

[java17.bat 파일 내용]
``` 
@echo off
set JAVA_HOME=C:\Program Files\Java\java-17-openjdk-17.0.3.0.6-1.win.x86_64
set Path=%JAVA_HOME%\bin;%Path%
echo Java 17 activated.
``` 

만든 파일들을 `scripts` 폴더 안에 넣고 cmd에서 .bat 파일명을 입력하면 그때그때 버전이 변경되는 것을 확인 할 수 있다. (이 과정을 마친 직후 cmd에서 테스트를 해 보았는데 정상적으로 작동하지 않는다면, cmd를 다시 실행해보거나 재부팅을 추천한다).  

![이미지](https://i.imgur.com/LiIolIF.png)  
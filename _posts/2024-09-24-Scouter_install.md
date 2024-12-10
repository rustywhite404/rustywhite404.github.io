---
title: Scouter 설치(인스톨 중 Java version / Path 문제 해결) 
date: 2024-09-24 18:20:00
categories: etc    
published: true 
tags:
- 성능개선 
- tip    
---

스카우터(Scouter)는 애플리케이션의 성능 모니터링과 진단을 도와주는 위한 오픈 소스 도구다. 서버, 애플리케이션, DB, 시스템 자원을 실시간으로 모니터링하고, 트랜잭션 성능을 분석하여 문제 파악을 도와준다고 한다. 스카우터를 사용해 보기 위해 설치를 진행해보았다.  

### 1. 스카우터 다운로드 및 설치 
다운로드 링크 : https://github.com/scouter-project/scouter/releases 

![이미지](https://i.imgur.com/02iCrtB.png)  
표시한 파일들을 다운로드 받는다. 각 파일의 역할은 아래와 같다. 
- scouter-all-[version].tar.gz : Scouter Collector와 Agent를 포함하는 압축파일
- scouter.client.product-[os].tar.gz : 각 OS별 뷰어 프로그램   

### 2. scouter-all(..).tar.gz 파일 압축 해제 후 실행  
압축을 풀고 난 후 server 디렉토리로 이동 ->  startup.bat를 실행시켜 서버를 구동시킨다.

![이미지](https://i.imgur.com/6Mait5l.png)  

![이미지](https://i.imgur.com/u9nphv9.png)  

### 3. Scouter Client 실행    
scouter.client.product(...).tar.gz의 압축을 풀고 실행시킨다. 
나는 여기서 자바 버전 문제가 발생했다. 컴퓨터에 자바 8과 11이 모두 설치되어 있는데, 환경 변수가 11로 세팅되어 있는데도 스카우터가 8 버전을 인식해서였다. 
그러면 아래 이미지와 같은 에러 메시지가 뜬다. 
![이미지](https://i.imgur.com/qa1YIOE.png)  

**[해결법]**
scouter.client 폴더 내의 scouter.ini 파일을 메모장으로 열고 맨 위에 내 컴퓨터의 javaw.exe 경로를 지정해준다. 
``` 
-vm
C:\Program Files\Java\jdk-11\bin\javaw.exe 
``` 
![이미지](https://i.imgur.com/sqvIfvg.png)  


### 4. Scouter Client 접속     
ID와 PW를 입력하고 접속한다. 기본값은 admin / admin 이다. 
![이미지](https://i.imgur.com/eanaUc5.png)  

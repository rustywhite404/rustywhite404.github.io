---
title: Scouter를 이용한 캐싱 처리 전후 비교 
date: 2024-09-28 21:19:00
categories: etc    
published: true 
tags:
- tip    
---

캐싱으로 조회 성능 개선하기 2 : (https://rustywhite404.github.io/etc/2024/09/27/Data_Cache2/)  
앞선 포스팅에서 캐싱 처리를 통해 데이터 성능 향상을 도모했고 nGrinder를 통해 전후 비교를 해보았다. 이번에는 Scouter를 이용해서 캐싱 처리 전후 비교를 해보려 한다.  

### 1. 스카우터 실행  
1. 스카우터 서버 실행 `scouter\server\startup.bat`  
2. 스카우터 호스트 실행(CPU상태변화를 보기 위함, 사전 conf 설정 필요) `scouter\agent.host\host.bat`  
3. 스카우터 클라이언트 실행 `scouter.client\scouter`  
4. 클라이언트 로그인 (초기 계정 정보 admin/admin)   

### 2. 소스코드 변경 전후 모니터링   
먼저 캐시 처리 부분을 주석 처리 한 후 애플리케이션을 실행시켰다.  
![이미지](https://i.imgur.com/fe8fKPS.png)  
다음은 캐시 처리를 할 수 있도록 주석을 해제 한 후 애플리케이션을 실행시켰다.  
![이미지](https://i.imgur.com/0X3zUen.png)  

### 3. 결과 비교      
- 캐시 처리 전에 비해 후의 CPU 사용량이 소폭 증가했다. => 캐시 처리 덕에 동시에 처리할 수 있는 양이 많아졌기 때문이다.  
`CPU 사용량이 60%를 넘어가면 모니터링이 필요하다고 한다. 너무 CPU 사용량이 많은 것도 좋지 않으니 유의 할 것.`  
- TPS도 캐시 처리 전에 비해 더 높아졌다.  
- 

### 4. 처리 시간이 오래 걸리는 api 확인하기  
![이미지](https://i.imgur.com/Xs9x4P9.png)  
지금은 컴퓨터 성능에 비해 크게 부하가 걸리는 작업으로 테스트 한 게 아니고, 계속 같은 api를 호출했기 때문에 트랜잭션 결과값이 비슷하다. 만일 특정 구간에서 처리 시간이 유독 오래 걸리는 게 확인된다면, 그 부분을 위와 같이 드래그해보자.  
![이미지](https://i.imgur.com/Pw670Lc.png)  
![이미지](https://i.imgur.com/Gwkp4x0.png)  
그러면 위와 같이 어떤 service에서 오랜 시간이 소요되는 지 확인 할 수 있고, 해당 api를 처리하는 과정과 처리시간까지 확인 할 수 있다. 이 정보를 통해 코드 개선이 필요한 api를 찾아낼 수 있을 것 같다.  


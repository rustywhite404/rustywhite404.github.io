---
title: (error)IntelliJ에서 Could not move temporary workspace 빌드 실패 시     
date: 2024-09-22 12:17:18
categories: etc        
published: true 
tags:
- IntelliJ   
- error      
---

아까까지 멀쩡하게 잘 돌아가던 프로젝트들이 갑자기 빌드에 실패하기 시작했다. 신규 프로젝트를 만들어도 그랬다. 검색해보니 아래와 같은 해결법들이 있었고 내 경우에는 이 중에 아무것도 효과가 없었다.  
1. `File`-`Invalidate Caches`에서 캐시 모두 삭제  
2. `C:\Users\user\.gradle\caches` 폴더 날리고 intelliJ 재실행 
3. 자바 버전(JDK) 변경해보기  
4. V3 실시간 검사 종료  

![이미지](https://i.imgur.com/3nE2Gy3.png) 
(문제상황.jpg)  
이전까지 똑같이 사용하던 JDK나 V3 문제는 아닌 것 같고, 에러 로그를 보면 gradle이 문제인 것 같았다. `gradle-wrapper.properties`에서 `distributionUrl`을 변경해주면 다른 그래들 버전으로 빌드를 시도할 수 있다고 해서 진행해보았다. 내 프로젝트들은 gradle 8.10, 8.6버전을 사용하고 있었는데 모두 8.5버전으로 변경했을 때 정상적으로 빌드되었다.  

```
distributionBase=GRADLE_USER_HOME
distributionPath=wrapper/dists
distributionUrl=https\://services.gradle.org/distributions/gradle-8.5-bin.zip
networkTimeout=10000
validateDistributionUrl=true
zipStoreBase=GRADLE_USER_HOME
zipStorePath=wrapper/dists 
```  
문제의 원인은 알 수 없었으나 그래들 버전을 수동으로 변경 할 수 있다는 점을 알게 되었다. 
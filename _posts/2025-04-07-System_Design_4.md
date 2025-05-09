---
title: ［독서기록］ 가상 면접 사례로 배우는 대규모 시스템 설계 기초 4장                 
date: 2025-04-07 16:23:48
categories: Book          
published: true 
tags:
- Book 
- Review          
- Architecture 
---  
 
📖 **제목 :** 가상 면접 사례로 배우는 대규모 시스템 설계 기초 : 4장  
🙋‍♂️ **저자 :** 알렉스 쉬    
4장에서는 `처리율 제한 장치(Rate Limiter) 설계`에 대해 다루고 있다. 
처리율 제한 장치란 말 그대로 특정 시간 동안 서버로 오는 클라이언트의 요청의 수를 제한하는 장치다. 한 번에 대용량 트래픽을 처리해도 모자랄 판에 **요청을 막는 이유**는 뭘까? 이 장에서는 왜 처리율 제한이 필요한지와 함께 대표적인 처리율 제한 알고리즘들을 살펴보고 있다.  

## 📌4장. 왜 처리율 제한이 필요할까?   
예를 들어, 어떤 API에 한 사람이 초당 100번씩 요청을 보내면 어떻게 될까? 다음과 같은 문제가 발생할 수 있다.  

- 서버 자원이 고갈됨 (CPU, 메모리, 네트워크 등)
- 다른 정상 유저에게 피해가 감
- DDoS 같은 공격에 서버가 무너지기도 함
- 비용도 올라감 (트래픽 기반 요금이라면)

👉 그래서 처리율 제한 장치는 공격자 또는 과도하게 사용하는 유저를 제어하고, 전체 시스템의 공정성과 안정성을 지키는 중요한 역할을 한다.  

## 📌처리율 제한의 핵심  

처리율은 `누가, 얼마나 자주, 어떤 리소스에 접근하는지`를 기준으로 제한을 둔다. 책에서 정리한 설계 시의 고려사항은 다음과 같다.   

### 대표적인 제한 기준 
- 호출 제한 위치가 클라이언트 측인지, 서버 측인지 
- 호출 제한 기준은 무엇인가? 사용자 ID, IP, API Key 등등. 
- 초당/분당/시간당 요청 수 제한은 얼마로 할 것인가 
- 분산 환경에서 동작해야 하는가? 
- 리소스당 제한 (예: /api/comments는 분당 10번까지만)이 따로 존재하는가 
- 처리율 제한에 걸렸을 때, 사용자가 그 결과를 알아야 하는가?  

> 👉 한도 초과가 발생한 경우 429번 Too many Request 상태코드로 에러메시지를 리턴할 수도 있다.  

## 📌실무에 적용 시 고려해야 할 점   

1. 요청 기준은 무엇일까? => 사용자 ID, IP, API Key 등... 
2. 어디에서 처리할까? 
    - 클라이언트에서 처리할 경우 : 요청이 위변조 될 가능성이 있고, 모든 클라이언트의 장치에 일괄적인 설정을 하기 어려워서 권장되지 않는다. 
    - 서버 내에서 처리할 경우 : 간단하지만 분산 환경에 약하다. 
    - Redis 같은 중앙 저장소에서  처리 : 분산 환경에 적합하다. 
3. 지속성 필요 여부  
    - 서버 재시작 후에도 카운터를 유지할 것인지  

> 👉 MSA처럼 API Gateway가 있는 경우, 게이트웨이에서 레이트 리미팅을 처리하는 방법도 고려해볼 수 있다. 또는, 직접 개발하지 않고 오픈소스 라이브러리를 사용하는 방법도 있다. 

## 📌대표적인 Rate Limiting 알고리즘 4가지  

### 1. 고정 윈도우(Fixed Window) 알고리즘  
정해진 시간 동안 요청 횟수를 제한하는 방식이다. 예를 들어, 1분에 100번만 요청이 가능하게 하고 매 분마다 카운터를 리셋한다.  

- 😊 **장점** : 구현이 쉽다  
- 😕 **단점** : 임계 시간에 몰리면 문제가 생길 수 있다(ex. 59초에 100번, 다음 1초에 100번 요청이 오면 실제로는 거의 연속 200번 요청됨)  

### 2. 슬라이딩 윈도우(Sliding Window) 알고리즘  
요청 타임스탬프를 저장해서 최근 1분(60초) 동안의 요청 수를 제한한다. 앞에서 본 `고정 윈도우의 단점을 보완`한 알고리즘으로, 실제 요청 시간을 기준으로 판단한다.  

- 😊 **장점** : 고정 윈도우의 약점을 보완했다    
- 😕 **단점** : 타임스탬프를 저장해야 하므로 메모리 사용량이 늘어난다. 

### 3. 토큰 버킷(Token Bucket) 알고리즘  
토큰을 일정 간격으로 채우고, 요청할 때마다 하나씩 꺼내 쓰는 방식이다.  
초당 1개씩 토큰이 생성되고 -> 버킷은 최대 10개까지 토큰을 저장함 -> 요청할 때 토큰이 있으면 OK, 없으면 거절하는 매커니즘이다.  

- 😊 **장점** : 구현이 쉽고, 메모리가 효율적이다. 버스트 상황에서 대처가 가능하다.    
- 😕 **단점** : 내부 시간 동기화가 필요하다. 버킷 수, 토큰 공급률에 대한 적절한 조절이 어렵다.  

> 👉 자바에서는 Bucket4J라는 라이브러리를 통해서 쉽게 처리율 제한이 가능하다고 한다.   


### 4. 누출 버킷(Leaky Bucket) 알고리즘  
통에 물(요청)이 들어오고, 일정한 속도로 물이 빠져나간다고 생각하면 이해하기 편하다. 요청이 버킷에 들어오면 일정 간격으로 하나씩 처리되는데, 한 번에 너무 많은 요청이 들어오면 넘쳐서 버려진다. 

- 😊 **장점** : 처리 속도를 균일하게 조절할 수 있다. 큐의 사이즈가 고정되어 있어 메모리 사용량 측면에서 효율적이다.      
- 😕 **단점** : 처리 속도가 느리면 요청이 유실될 수 있다. 토큰 버킷과 마찬가지로 두 인자를 튜닝하기 어렵다.  

> 👉 Shopify, Nginx에서 해당 알고리즘으로 Rate Limiter를 사용한다고 한다.  

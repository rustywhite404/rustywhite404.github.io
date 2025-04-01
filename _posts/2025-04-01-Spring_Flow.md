---
title: 스프링은 어떻게 동작할까? 요청부터 응답까지 흐름 정리           
date: 2025-04-01 19:20:14
categories: Spring          
published: true 
tags:
- Spring         
---  

## 스프링은 어떻게 동작할까?  
개발 할 때 맨날 쓰고 있지만 스프링이 내부적으로 어떻게 동작하는지에 대해서는 정확히 설명하기 어려웠다. 그래서 요청부터 응답부터 어떤 흐름을 거치는지, 각 단계에서 어떤 동작을 하는지 정리해보기로 했다.  
![스프링흐름도](https://i.imgur.com/AUtKipp.png)  

- **Filter**  
클라이언트 요청이 DispatcherServlet에 도달하기 전에 HTTP 요청/응답을 가로채서 전처리 또는 후처리(인증, CORS 처리 등)를 담당한다.  

- **DispatcherServlet**  
스프링의 핵심 Front Controller로, 요청이 들어오면 어떤 컨트롤러에 전달 할 지 결정하고 응답 결과를 다시 사용자에게 전달한다. (일종의 주소록 역할을 하는 HandlerMapping에서 이 작업을 처리함).   

- **Interceptor**  
DispatcherServlet과 Controller 사이에서 동작한다. 공통 로직 처리를 담당한다(로그인 여부 확인, 로깅, 권한 체크 등).  

- **Controller**  
사용자 요청을 받아서 처리하는 핵심 진입 지점. 비즈니스 로직을 수행하거나 Service에 작업을 위임한다.  

- **Service->Repository->DB**  
실제 비즈니스 로직과 데이터 접근 계층  

- **ViewResolver**  
컨트롤러가 반환한 뷰 이름을 받아서 실제로 보여줄 View를 결정한다.   
---
title: Intelli J 단축키를 알아보자(응용)   
date: 2022-08-09 00:12:00
categories: etc 
published: true 
tags:
- IntelliJ  
---

## 테스트 코드, 리팩토링, 디버깅 모드 단축키     
기본 기능들 외 부가적인 인텔리제이 단축키들도 정리해둔다.  


### 테스트 코드

- 생성 : 테스트 할 클래스에 대고 `Ctrl + Shift + T`
- 선택한 테스트 실행 : `Ctrl + Shift + F10`
- 테스트 정지 : `Ctrl + F2`

### 리팩토링

- 클래스 이동 : 옮기고 싶은 클래스 파일에서 `F6`을 누르면 패키지를 이동시킬 수 있다.
- 타입 변경 : `Ctrl + Shift + F6`
- 시그니처(private / string / name) 부분 변경 : `ctrl + F6`
- 이름 변경 : `Shift + F6`
- 한 번에 같은 단어 모두 선택(해서 변경) : `Ctrl + Shift + Alt + J` / ESC로 빠져나오기
- 리팩토링 목록 보기 : `ctrl + alt + shift + T`

### 디버깅

- 브레이크 포인트 설정 : `Ctrl + F8`
- 브레이크 포인트 보기 : `Ctrl + Shift + F8`
- 디버깅 모드 실행 : `Shift + F9`
- step into(현재 브레이크포인트에서 실행하고 있는 라인으로 이동) `F7`
- step over(현재 브레이크포인트에서 다음 라인으로 이동) `F8`
- Resume Program(다음 브레이트 포인트 or 끝까지 이동) `F9`

### Git

- Git 메뉴들 `alt + ``
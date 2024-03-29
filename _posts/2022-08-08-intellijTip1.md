---
title: Intelli J 단축키를 알아보자(기초)  
date: 2022-08-08 00:12:00
categories: etc 
published: true 
tags:
- IntelliJ  
---

## Intelli J 단축키를 알아보자     
STS와 다소 차이가 있다보니 쓸 때 마다 이게 뭐더라 하고 기억을 더듬게 되어서 정리해 봤다. 


<br/>

### 열기

- 프로젝트 창 열기 `Alt + 1`
- 프로젝트 창에 포커스 있는 상태에서 esc 누르면 에디터로 포커스가 돌아옴
- 프로젝트 창에서는 키보드 위아래로 움직여서 포커스 이동이 가능한데, `space`를 눌러서 내가 원하는 파일이 맞는지 미리 보기가 가능
- 에디터 창 오가기 `ctrl + tab`
- 새 파일 생성 1(에디터에서) : `ctrl + alt + insert`
- 새 파일 생성 2(프로젝트 창에서) : `alt + insert`
- 에디터에서 `alt + insert`를 하면 getter, setter, override 등을 생성
- 최근 파일 열기 `ctrl + E`

### 커서 이동

- 단어 단위로 이동 `ctrl + ← →`
- 라인 시작/끝 이동 `home, end`
- 페이지 위/아래 `page up, page down`

### 확장/축소

- 에디터 창 최대화 `ctrl + shift + F12`
- 선택영역 확장, 축소 : 확장`ctrl + w`, 축소`ctrl + shift + w`

### 주석

- 한 줄 주석 `ctrl + /`
- 블록 주석 `shift + ctrl + /`

### 검색

- 이 코드가 사용되는 위치 찾기 `alt + F7`
- 이 코드가 사용되는 위치 찾기(빠른 이동) `ctrl + B`
- 파일 검색 `ctrl + F`
- 검색해서 찾은 결과로 이동 `F3, shift + F3`
- 이 키워드를 포함하는 파일 검색 `ctrl + shift + F`
- 전체 검색(키워드 포함 파일 뿐만 아니라 인텔리J 내의 기능들까지 전부 검색) `Shift 2번`

### 템플릿

- foreach, main, pst 등 각종 자주 쓰는 단축키 `ctrl + j`
- System.out.println() 단축키 : `sout + tab`
- public static void main(String[] args) 의 단축키 : `psvm + tab`
    
    <aside>
    💡 sout의 경우, 자주 사용하는 코드의 축약형을 제공하는 라이브 템플릿에서 변경 가능하다.
    다른 축약어들 역시 필요하다면 추가로 지정해서 사용할 수 있음. 
    Settings -> Editor -> Live Template
    나는 sout을 sysout+스페이스로 변경해 둔 상태.
    
    </aside>
    

### 픽스

- 빨간줄 뜨는 부분 빠르게 수정(퀵픽스) `alt + enter`
- 프로젝트에 에러가 여러개일 때 하나씩 에러로 이동 `F2, Shift + F2`

### Import 최적화

- 더이상 안 쓰는 import 라이브러리 삭제 `ctrl + alt + O`

### 코드 생성

- 코드 생성(Getter, Setter, toString…) `alt + Ins`
- Override 자동 생성 `ctrl + O`
- implement 자동 생성 `ctrl + I`

### 터미널 창

- 터미널 창 보기 `Alt + F12`

### 대체하기(치환)

- 파일 내 대체 `ctrl + R`
- 경로 내 대체 `ctrl + shift + R`

### 프로젝트 실행/종료

- 실행 `Shift + F10`
- 종료 `ctrl + F2`

### 라인 수정

- 라인 복제 : `CTRL + D`
- 라인 삭제 : `CTRL + Y`

### 기타

- 파라미터 정보 `CTRL + P`
- 자동 들여쓰기 `ctrl + alt + i`
- 인텔리제이 대소문자 가리지 않고 자동완성 하기
settings→ Editor → General → Code Completion의 Match case를 체크해제
- 프로젝트 내 단어 전체 변경 : `CTRL + SHIFT + R`
- Java Doc 작성 /** 하고 enter 치면 알아서 생성
- 한 번에 모든 메소드들 implements 받아오기(오버라이드) : `ALT + ENTER`
- 자바 코드를 람다 형식으로 변경 : `ALT + ENTER`  (이것도 위와 같은 단축키)
- `CTRL + SHIFT + ENTER` : 자동 닫기(구문 자동 완성)
- 코드 정렬 : `CTRL + ALT + L`
- 단축키 찾기(기능 찾기) `shift + ctrl + A`
- 해당 메소드 테스트코드 만들기 : `CTRL + SHIFT + T`
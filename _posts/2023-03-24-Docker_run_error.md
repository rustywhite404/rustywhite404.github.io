---
title: 도커 실행 시 repository does not exist or may require 'docker login' ~만 반복적으로 뜨는 문제 해결  
date: 2023-03-24 11:44:00
categories: Docker   
published: true 
tags:
- Docker  
---

도커에 리눅스 이미지를 올려서 실행해야 하는 상황이었다. 이미지 pull 받고 컨테이너 생성까지 무사히 됐는데 컨테이너 실행이 안 되는 문제가 발생했다. 결과적으로 도커 명령어에 대한 이해가 부족해서 생긴 문제였는데, 에러 메시지나 검색해서 나오는 도커 컨테이너 실행 순서 예제로는 해결이 안 됐었기 떄문에 다시 이런 경우가 발생하지 않도록 기록해둔다. 

### 기본적인 생성 순서  
나는 rocky linux를 사용하려고 컨테이너를 생성한 것이므로 똑같이 rocky linux로 진행해보겠다. 
```linux  
// 1. 버전을 명시해서 image를 내려받는다(나는 8.7을 받았다)
docker pull rockylinux:8.7  

// 2. 내려받은 이미지 확인
docker images 

// 3. 컨테이너 생성(testContainer라는 이름으로 생성했다)
docker run --name testContainer -d rockylinux:8.7 

// 4. 도커 컨테이너 목록 확인(아직 오프라인 상태다)
docker ps -a 

```   

### 문제 상황 
위 4단계 까지는 예상대로 잘 실행이 되었다. 문제가 생긴 건 컨테이너 실행 부분이었다.  
`docker run testContainer`, `docker run rockylinux`, `docker start testContainer`, `docker run --name testContainer2 -d -p 8080:80 rockylinux:8.7` 등등으로 시도해봤을 때, 계속 아래와 같은 메시지가 출력되었다.  

```linux  
// 1. 
Unable to find image 'testContainer:latest' locally
docker: Error response from daemon: pull access denied for testContainer, repository does not exist or may require 'docker login': denied: requested access to the resource is denied.  

// 2. 
Unable to find image 'rockylinux:latest' locally
docker: Error response from daemon: manifest for rockylinux:latest not found: manifest unknown: manifest unknown.
``` 
나는 rockylinux에 대한 권한이 없으니 계속 권한 문제가 발생했던 게 아닌가 싶다. docker login은 이미 되어 있는 상태였으니 백날 다시 해 봐야 결과는 같았다. 

### 해결   
![이미지](https://i.imgur.com/k5d2dSP.png)  
실행에 성공한 명령어는 `docker container run -d -it --name "testContainer" rockylinux:8.7 /bin/bash` 였다. 이 다음에 `docker ps`로 확인해보면 컨테이너가 실행중인 걸 볼 수 있다. 도커에 설치한 rocky linux로 들어가려면 `docker exec -it testContainer /bin/bash` (또는 docker attach 컨테이너ID)로 진입하면 된다.  

### 기타  
- docker container 종료시키기 : `docker stop container아이디`  
- docker container 재시작 : `docker restart container아이디`  
- docker 컨테이너 내부 진입 : `docker attach container아이디`  



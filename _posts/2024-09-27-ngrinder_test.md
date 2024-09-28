---
title: ngrinder를 이용한 캐싱 처리 전후 비교   
date: 2024-09-28 17:48:12
categories: etc    
published: true 
tags:
- tip    
---

캐싱으로 조회 성능 개선하기 2 : https://rustywhite404.github.io/etc/2024/09/27/Data_Cache2/  
앞선 포스팅에서 캐싱 처리를 통해 데이터 성능 향상을 도모했다. 이제 실제로 이 작업이 성능 향상에 도움이 되었는지 ngrinder를 이용하여 확인 해보려 한다. 


### 1. ngrinder란?        
네이버에서 만든 부하테스트 툴로 웹 애플리케이션 Controller와 자바 애플리케이션 Agent로 구성되어 있다.  
다운로드 링크 : https://github.com/naver/ngrinder/releases  
나는 `ngrinder-controller-3.5.9-p1.war` 버전을 사용했다.  

1. controller 실행  
war파일을 다운로드한 디렉토리 경로에서 cmd를 열고 `java -jar {war파일명} --port=8300`을 입력한다. (ex. 'java -jar ngrinder-controller-3.5.9-p1.war --port=8300)  

2. 애플리케이션 접속  
![이미지](https://i.imgur.com/o4FYug2.png)  
컨트롤러를 실행시킨 후 http://localhost:8300/ 로 접속해보면 nGrinder 웹에 접속 할 수 있다. 초기 계정 정보는 admin / admin 이다.  

3. Agent 다운로드  
admin > Download Agent 에서 agent파일을 다운로드 후 압축을 풀어준다. 압축을 푼 후 생성된 `run_agent.bat`을 실행시킨다. 그런 다음 웹으로 돌아와 `Agent Management` 탭을 클릭하면 agent가 정상적으로 올라온 것을 확인 할 수 있다.  
![이미지](https://i.imgur.com/YyDrMgL.png) 


### 2. 부하 테스트를 위한 코드 설정    
![이미지](https://i.imgur.com/FH6DQwS.png)  
캐시 유효기간을 20초로 늘렸다. 
![이미지](https://i.imgur.com/LWpUwZ1.png)  
우선 캐시를 사용하지 않는 경우를 테스트 할 예정이므로 `@Cacheable` 부분은 주석 처리 후 진행한다.  

### 3. ngrinder 스크립트 작성 및 결과 확인   
![이미지](https://i.imgur.com/GXZyZIc.png)  
Script 메뉴의 create 버튼을 눌러 위와 같이 테스트 할 URL을 입력한다. Script Name은 알아 볼 수 있는 이름으로 자유롭게 정하면 된다. 
![이미지](https://i.imgur.com/EULDUpd.png) 
생성을 완료한 후, 정상적으로 동작하는지 Validate를 눌러 검증해본다. 이미지와 같이 200으로 결과가 나오면 정상 작동하는 것이므로 Save한다. 

![이미지](https://i.imgur.com/2Vqfmg8.png) 
이제 Perpomance Test 메뉴로 이동한다. Test Name은 자유롭게 지정해주고, Vuser(가상사용자) 수를 설정해준다. 이번에는 `10명의 가상사용자가 1분 동안 5000건의 공지사항 리스트 전체를 확인` 하는 상황을 가정하고 테스트 해 보았다. 

![이미지](https://i.imgur.com/nVDSN8j.png) 
첫번째 결과(캐시 적용 X)  
![이미지](https://i.imgur.com/DpaQDEg.png) 
두번째 결과(캐시 적용 O)  
=> 두 번 정도 성능이 소폭 저하되는 구간이 있는데, 캐시 생명주기를 20초로 설정해 두어서 새로 캐시를 쌓는 구간에서 성능 저하가 발생한 것 같다.   

***수치 변화 확인***  
- 평균 TPS : 23.2 → 680.6  
- Peek TPS : 25.0 → 757.5  
- Mean Test Time : 426.80ms → 14.72ms  
- Exected Tests : 1,313 → 38,315  
=> 전체적으로 상당히 크게 개선된 것을 알 수 있다.  

### 4. 스크립트를 수정해서 추가 테스트를 해보자  
```java   
// 기존 코드
@Test
	public void test() {
		HTTPResponse response = request.GET("http://127.0.0.1:8080/api/notices", params)

		if (response.statusCode == 301 || response.statusCode == 302) {
			grinder.logger.warn("Warning. The response may not be correct. The response code was {}.", response.statusCode)
		} else {
			assertThat(response.statusCode, is(200))
		}
	}


// 랜덤하게 호출 하도록 수정
@Test
	public void test() {
	// 랜덤한 페이지 값 생성 (1 이상, 10 이하)
    def randomPage = new Random().nextInt(10) + 1

    // API 호출 URL 조합
    def apiUrl = "http://127.0.0.1:8080/api/notices/${randomPage}"

    // API 호출 및 응답 획득
    HTTPResponse response = request.GET(apiUrl, params)

		if (response.statusCode == 301 || response.statusCode == 302) {
			grinder.logger.warn("Warning. The response may not be correct. The response code was {}.", response.statusCode)
		} else {
			assertThat(response.statusCode, is(200))
		}
	}
```  
***결과 확인*** 
![이미지](https://i.imgur.com/nXrtm3n.png)  
(공지사항은 보통 최신 데이터 위주로 보게 되므로 캐싱 처리는 5page 까지만 적용되었다.)  
확실히 페이지를 끊어서 호출하는 게 전체를 조회하는 것보다 성능 부담이 적다.  
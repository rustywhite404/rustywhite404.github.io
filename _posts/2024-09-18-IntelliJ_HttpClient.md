---
title: IntelliJ에서 http Client를 사용해 API 테스트 하기     
date: 2024-09-18 19:21:10
categories: etc       
published: true 
tags:
- Tip     
- IntelliJ      
---

개발하면서 API 테스트를 할 때 포스트맨을 사용하는 경우가 많은데, IntelliJ 내부에서도 API테스트를 할 수 있다는 걸 알게 되었다. 


### 1. 사용을 위한 준비    
![이미지](https://i.imgur.com/FJrPP7n.png)  
원활한 파일 관리를 위해 http 패키지를 만들었다. 그 안에 deb.create.`http`라는 확장자의 파일을 하나 만든다. 파일명은 아무거나 상관없다(ex. test.http, apiTest.http). 
![이미지](https://i.imgur.com/08XPbEs.png)  
생성한 파일을 열어보면 우측 상단에 Examples가 있고, 눌러보면 GET, POST 등 API 테스트를 할 때 활용할 수 있는 예제들을 볼 수 있다.  
![이미지](https://i.imgur.com/36XD00G.png)  
POST 방식으로 API를 호출 할 때 많이 사용하게 되는 json 방식의 예제를 가져와서 테스트를 해 보자.  

```java 
//controller  
@PostMapping("/create-developers")
    public CreateDeveloper.Response getAllDevelopers(@Valid @RequestBody CreateDeveloper.Request request){
        log.info("request:"+request);
        return dMakerService.createDeveloper(request);
    }
```  
```java  
//service  
@Transactional
    public CreateDeveloper.Response createDeveloper(CreateDeveloper.@Valid Request request){

        validateCreateDeveloperRequest(request);

        Developer developer =
                Developer.builder().developerLevel(request.getDeveloperLevel())
                        .developerSkillType(request.getDeveloperSkillType())
                        .experienceYears(request.getExperienceYears())
                        .name(request.getName())
                        .memberId(request.getMemberId())
                        .age(request.getAge()).build();
        developerRepository.save(developer);
        return CreateDeveloper.Response.fromEntity(developer);
    }
```  
이런 식으로 데이터를 입력받아서 저장하는 api가 있다고 쳤을 때, json 데이터를 입력해서 테스트를 진행해보자.  

![이미지](https://i.imgur.com/gFW3rBo.png)  
맨 윗줄의 `###`은 각각의 요청을 구분하는 구분자이다. 주석의 역할을 한다고 생각하면 될 것 같다. 그 아래에 GET/POST 방식을 명시하고 호출할 주소를 작성한 후 데이터를 형식에 맞춰서 넣어준다. 

### 2. 결과 확인해보기  
![이미지](https://i.imgur.com/8fPy5Kb.png)  
![이미지](https://i.imgur.com/jSSyU5A.png) 
프로그램을 실행했을 때의 결과를 볼 수 있다. status 코드 확인이 가능하고, 콘솔에서도 처리 과정을 확인 할 수 있다. 이 예시는 실패했을 때의 결과이다.  
![이미지](https://i.imgur.com/XmW3jpR.png)  
![이미지](https://i.imgur.com/kYArwqM.png)  
성공 했을 때는 이렇게 200 코드가 떨어진다. 

그 밖에도 `{{변수명}}`을 이용하여 변수명을 지정할 수도 있기 때문에 자주 사용하는 몇 가지 변수들을 따로 파일로 빼서 변수를 활용하면 더 사용이 용이할 것 같다.  
> ex. 개발환경과 운영환경의 도메인 주소를 각각 변수로 활용   
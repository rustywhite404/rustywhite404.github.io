---
title: 스프링부트에서 자주 사용하는 어노테이션 정리     
date: 2022-08-15 16:51:00
categories: Spring 
published: true 
tags:
- Spring 
- 어노테이션
- SpringBoot
---


### 이 어노테이션은 무슨 용도일까

스프링부트는 MVC와 기능은 비슷하지만 다른 어노테이션을 쓰는 경우가 있다. 또 비슷한 기능이지만 개발자에 따라 사용하는 어노테이션이 다를 때도 있어서 종종 낯선 어노테이션을 보게 된다. 자주 쓰는 어노테이션을 포함하여 낯설었던 것들까지 함께 정리해둔다.

- **@SpringBootApplication**  
→ Application 클래스에 사용하는 어노테이션으로, 이 어노테이션이 있는 위치에서부터 설정을 읽어가기 때문에 항상 프로젝트의 최상단에 위치한다. 이 어노테이션을 통해 스프링부트가 Bean 읽기와 생성을 모두 자동으로 할 수 있음.
- **@CrossOrigin**  
→ CrossOrigin(CORS)를 지원할 수 있게 허용하는 어노테이션. 다른 서버간의 통신을 지원해준다. 특정 포트만 허용하겠다 또는 특정 메소드(GET, POST…)만 허용하겠다 등의 옵션 설정도 가능하다.
- **@Controller, @RestController**  
→ 둘 다 컨트롤러를 선언해주는 어노테이션이다. RestController는 리턴 값에 자동으로 ResponseBody가 붙게 되어, 어노테이션을 명시하지 않아도 바디에 자바 객체가 매핑되어 전달 된다. Controller는 바디를 자바 객체로 받으려면 @RequestBody 어노테이션을 반드시 명시해야 함.
- **@RequestHeader**  
→ 요청(request)에서 특정 HTTP 헤더 정보를 추출할 때 사용
- **@RequestMapping**  
→ 요청 들어온 URL과 이 어노테이션의 값이 일치하면 해당 클래스나 메소드가 실행됨. Controller 안의 메서드와 클래스에 적용 가능하고, 클래스에 사용하면 하위 메서드에 모두 적용된다.
- **@RequestParam**  
→ URL에 전달되는 파라미터를 메소드의 인자와 매칭시켜준다.
- **@RequestBody**  
→ Body에 전달되는 데이터를 메소드의 인자와 매칭시켜서 데이터를 처리한다. JSON 형태로 값을 전송하면, 이 값을 자바 Object 형태로 변환해주어 편리하다.
- **@SessionAttribute**  
→ 세션상에서 모델의 정보를 유지하고 싶은 경우에 사용
- **@PathVariable**  
→ 현재의 URI에서 {원하는 정보}를 추출할 때 사용하는 어노테이션. REST API에서 값을 호출할 때 많이 사용한다. 
→ REST API 방식에서는 주소를 쿼리 스트링이 아니라 경로 형태로 만들어 주어야 한다(주소이면서 데이터여야 함)
→ 예를 들어, requestParam을 사용하면 주소는 `localhost:8080?name=mjkim&age=21` 처럼 보이게 되지만 PathVariable을 사용하면 [`localhost:8080/mjkim/21`](http://localhost:8080/mjkim/21) 처럼 보이게 된다.
    
    ```java
    	@GetMapping("/info/{id}") 
    	public UserEntity<?> 유저 정보(@PathVariable int id){
    	...  
    	}
    ```
    
- **@PostMapping, @GetMapping, @DeleteMapping, @PatchMapping**  
→ Spring 4.3부터 Spring MVC Controller Method를 위한 어노테이션이 추가되었는데 각각의 어노테이션들은 HttpMethods에 매칭되며, method단에서 사용 가능하다.
    
    ```java
    // 변경전
    @RequestMapping(value="/getList", method={RequestMethod.POST})
    
    // 변경후
    @PostMapping("/getList")
    ```
    
- **@Resource,  @Autowired,  @Inject**  
모두 의존 관계를 자동으로 연결하여 빈을 주입해주는 기능을 가진 어노테이션. 검색 방식이나 범용성 등에서 차이가 있다.
- **@Transactional**  
→ 데이터베이스 트랜잭션을 설정하고 싶은 메서드에 적용하면, 이 메서드 내부에서 일어나는 데이터베이스 로직이 전부 성공하지 않으면 앞에 성공한 로직들도 모두 롤백한다. 즉, 모든 처리가 정상적으로 되었을 때만 DB에 커밋하는 것이다. 대부분 SERVICE 영역에서 관리하게 된다.  `@Transaction(noRollbackFor=Exception.class), @Transaction(timeout = 10)` 등의 옵션이 있다.

```java
@Controller // 컨트롤러 선언 
@RequestMapping(”/user”) // /user 로 들어오는 요청 모두 처리 
public class UserController{

	@GetMapping("/info") //GET 방식으로 들어오는 /user/info 처리 
	public void 유저조회(@RequestParam String id, @RequestParam(name="old") String age){
	String info = id + "," + age; 
	...  
	}
	
	@PostMapping("/create") //POST 방식으로 들어오는 /user/create 처리 
	public void 유저생성(@RequestBody User user){
	String sub_name = user.name;
	String sub_old = user.old; 
	...  
	}
}
```

---

### Entity, Lombok 관련

- **@AllArgsConstructor**  
→ 롬복에서 사용함. 모든 필드 값을 파라미터로 받는 생성자를 추가한다.
- **@NoArgsConstructor**  
→ 롬복에서 사용함. 기본 생성자를 자동으로 추가한다.
- **@Getter, @Setter, @ToString**  
→ 롬복에서 사용함. 보이는 대로의 뜻이며 @ToString(exclude = "원하는 필드")를 하면 특정 필드가 toString 결과에서 제외된다.  
→ Entity 클래스에서는 Setter 사용을 지양한다. 객체에서 메소드를 제공하여 변경하면 변경 의도도 한 번에 알 수 있고, 객체 자신의 값을 객체가 스스로 변경하는 것이 객체 지향의 관점에서도 더 자연스럽다.
    
    ```java
    //레거시 
    Member member = new Member();
    member.setName("이연정");
    member.setAge("19");
    ... 
    //실무에서는 이런 식으로 업데이트 해야 하는 필드가 아주 많은데, 그러면 결국 뭘 위한 수정중인지 알아보기 힘들어지는 문제도 생김
    
    //JPA
    member.changeMemberBasicInfo("이연정","19");  //메소드명을 통해 멤버의 기본 정보를 변경한다는 것을 바로 알 수 있음.
    
    //Member 엔티티 내부는 이렇게 메서드가 있을 것
    public void changeBasicInfo(String name, String tel){
    	this.name = name;
    	this.age = age; 
    } 
    ```
    
- **@Data**  
→ 롬복에서 제공하는 필드와 관련된 모든 코드를 생성한다. ‘모든’ 기능을 허용하므로 위험성이 존재한다. 사용하지 않는 쪽을 권장한다고 함.
- **@Table**   
→ Entity에 매핑할 테이블 정보를 알려준다. 생략하면 클래스명이 테이블명으로 매핑된다.
- **@Entity**  
→ 실제 DB의 테이블과 매칭될 클래스임을 명시한다.
- **@Id**   
→ 이 테이블의 PK필드를 나타낸다.
- **@GeneratedValue**  
→ PK 생성 규칙을 나타낸다. 기본 값은 AUTO_INCTREMANT임.
- **@Column**  
→ 테이블의 컬럼을 나타내며, 굳이 선언하지 않아도 이 클래스의 필드는 모두 컬럼이 된다. 생략할 경우 필드명과 컬럼명이 매핑된다. 그럼에도 굳이 명시하는 이유는 기본값 외에 추가로 변경하고 싶은 옵션이 있을 때 사용한다. (ex. 문자열 사이즈를 늘리고 싶거나, 타입을 변경하고 싶거나 등등)
- **@Null, @NotNull**   
→ 어떤 타입이든 수용하고 null을 허용한다, 허용하지 않는다를 정의한다
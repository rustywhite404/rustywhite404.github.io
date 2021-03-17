---
title: (Spring/Ajax)실시간으로 아이디 중복 체크하기     
date: 2021-03-17 21:25:00
categories: Spring, SpringBoot, Ajax
tags:
- Spring  
- Ajax  
--- 

### 개발 환경  
사용한 파일 및 폴더 구조는 다음과 같다(파란색으로 표시되어 있는 파일들만 이번 단계에서 사용한 파일들이다). Thymeleaf를 사용하여 뷰 페이지를 구현했기 때문에 jsp가 아니라 html로 작성되었지만, 사용하는 문법은 똑같다.    

![이미지](https://i.imgur.com/0nDsj1s.png)  
![이미지2](https://i.imgur.com/RuTJDCX.png)  
  
## join.html에서 id를 입력받는 폼 만들기  
```html
<input type="text" id = "id" name="id" placeholder="your id" autocomplete="username" required oninput = "checkId()" /> 
<span class="id_ok">사용 가능한 아이디입니다.</span>
<span class="id_already">누군가 이 아이디를 사용하고 있어요.</span>
```  
먼저, id값이 "id"인 input 입력란을 만들어주고, oninput요소를 사용할 함수checkId()를 선언해준다.  
> oninput은 HTML5에서 새로 등장한 기능으로, 사용자의 입력을 받으면 실행되는 이벤트다. 즉, 커서를 다른 곳으로 옮기지 않아도 입력 즉시 DB에서 id를 비교할 수 있다.  
 
> placeholder는 입력란에 보일 미리보기 문구, autocomplete는 자동완성 기능 사용 시 브라우저가 이 입력폼에 어떤 값을 넣어줄 지 판단할 수 있도록 해주는 값이다. 이번 예제에서는 꼭 넣지 않아도 상관없다.  
  
그런 다음 id_ok 클래스와 id_already 클래스를 가진 영역을 만들어준다. p, div 등 아무 거나 상관 없고 해당 클래스들만 적용되어 있으면 됨. 내용을 보면 알 수 있듯이 DB에 id가 있는지 확인한 후, 결과에 따라 출력되는 문구들이다.  

---  

## css 수정  
```css  
.id_ok{color:#fff; display: none;}
.id_already{color:#6A82FB; display: none;}
```  
id_ok는 사용 가능한 아이디일 때, id_already는 이미 사용중인 아이디일 때 표시할 클래스들이다. 기본적으로는 모두 display:none 처리를 해 두고, id 값을 입력받았을 때 DB 비교 결과에 따라 둘 중 하나를 display:block 처리하게 된다.    

--- 

## Ajax 처리를 위한 준비  

```html  
<script src="https://code.jquery.com/jquery-3.4.1.min.js"></script>
<script type="text/javascript">
    function checkId(){
        var id = $('#id').val(); //id값이 "id"인 입력란의 값을 저장
        $.ajax({
            url:'/user/idCheck', //Controller에서 인식할 주소
            type:'post', //POST 방식으로 전달
            data:{id:id},
            success:function(){
                console.log("처리 성공 시 변경되는 내용");
            },
            error:function(){
                alert("에러입니다");
            }
        });
    };
</script>
```  
join.html의 스크립트 부분에 위와 같이 제이쿼리 사용 선언을 해준 후 코드를 작성한다.  
id값이 id인 폼에 들어온 내용을 다시 var id에 저장하고, 그 값을 POST 방식으로 /user/idCheck에 보내게 된다. 그러면 Controller → Service → Repository(DAO) → Mapper 에서 입력받은 id값에 해당하는 id가 있는지 DB에서 비교한 후 결과값을 다시 뷰로 리턴받아 사용할 수 있다. <small>잘 이해가 안 되면 일단 따라하자 나도 그랬으니까...^-^)...</small>  

---  

## Controller 영역의 처리  

```java  
@Controller
@RequestMapping("/user")
public class UserController {

    @Autowired
    private UserService userService;

    // 아이디 체크
    @PostMapping("/idCheck")
    @ResponseBody
    public int idCheck(@RequestParam("id") String id){
        logger.info("userIdCheck 진입");
        logger.info("전달받은 id:"+id);
        int cnt = userService.idCheck(id);
        logger.info("확인 결과:"+cnt);
        return cnt;
    }
}
``` 

3번에서 `url:'/user/idCheck'` 라고 선언해두었으니, 컨트롤러에서도 /user/idCheck일 때 아이디 체크를 하도록 매핑해주면 된다. 그런 다음, 전달 받은 id를 가지고 userService의 idCheck로 이동하여 리턴받은 결과를 int cnt에 저장했다. DB에서 이 id의 갯수를 조회했을 때 결과가 1이면 이미 사용되고 있다는 뜻이고, 0이면 사용중이 아니라는 뜻일테니까. 이 cnt 값을 뷰에 넘겨주기로 하고, 이제 Service와 Repository(DAO)를 구현하러 가 보자.  

> logger는 기능이 제대로 실행되고 있는지 확인을 위해 찍어놓은 거라, 작동을 확인하면 지우거나 주석처리 해도 상관없다.  

---  

## Service, Repository 영역의 처리  

Service, Repository에 메소드를 바로 구현하지 않고 인터페이스를 사용하여 구현하였기 때문에, 파일은 총 4개가 된다(Service, ServiceImpl, Repository, RepositoryImpl).  
<br />  

**1. UserService**  
```java  
public interface UserService {
    // 아이디 중복확인
    public int idCheck(String id);
}
```  

**2. UserServiceImpl**  
```java  
@Service
public class UserServiceImpl implements UserService{

    @Autowired
    private UserRepository userRepository;

    // 아이디 중복 체크
    @Override
    public int idCheck(String id) {
        int cnt = userRepository.idCheck(id);
        return cnt;
    }
}

```  

**3. UserRepository**  

```java
public interface UserRepository {
    // 아이디 중복 체크
    public int idCheck(String id);
}

```  
**4. UserRepositoryImpl**  
```java  
@Repository
public class UserRepositoryImpl implements UserRepository {
    @Autowired
    private SqlSession sqlSession;

    private static final String NAMESPACE = "com.mysite.mapper.UserMapper"; 

    // 아이디 중복 체크
    @Override
    public int idCheck(String id) {
        int cnt = sqlSession.selectOne(NAMESPACE+".idCheck", id);
        return cnt;
    }
}
```  
> NAMESPACE의 경우 **꼭 본인이 사용하는 Mapper의 namespace**를 적어야 한다.  

Service와 Repository는 딱히 설명할 부분이 없는 것 같다. 바로 Mapper로 넘어가보자.

--- 

## Mapper의 SQL 구문 처리  

```xml
<?xml version="1.0" encoding="UTF-8"?>

<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.mysite.mapper.UserMapper"> <!--NAMESPACE 확인-->
    <!-- 아이디 중복 체크 -->
    <select id="idCheck" parameterType="String" resultType="int">
        SELECT COUNT(id) FROM USER WHERE id= #{id}
    </select>
</mapper>
```  

SQL 구문을 확인해보면, USER 테이블에서 id가 #{id}(전달받은 id)인 컬럼의 숫자를 센다. 아이디는 중복이 불가능하게 처리해두었으므로 조회 결과는 1(사용중)이나 0(없는 아이디)이 반환될 것이므로 resultType을 int로 받았다.  

---  

## 다시 스크립트로 돌아가서  

```html
<script type="text/javascript">
        function checkId(){
            var id = $('#id').val();
        $.ajax({
            url:'/user/idCheck',
            type:'post',
            data:{id:id},
            success:function(cnt){ //컨트롤러에서 넘어온 cnt값을 받는다 
                if(cnt != 1){ //cnt가 1이 아니면(=0일 경우) -> 사용 가능한 아이디 
                    $('.id_ok').css("display","inline-block"); 
                    $('.id_already').css("display", "none");
                } else { // cnt가 1일 경우 -> 이미 존재하는 아이디
                    $('.id_already').css("display","inline-block");
                    $('.id_ok').css("display", "none");
                }
            },
            error:function(){
                alert("에러입니다");
            }
        });
        };
    </script>
```  
컨트롤러에서 넘어온 cnt 값을 확인한 후 0일 때와 1일 때를 구분하여 id_ok클래스와 id_already클래스를 표시/숨김 처리한다.  

---  

## 결과 확인  
![결과1](https://i.imgur.com/16iFi7o.png)  
cnt가 1일 경우, 이미 해당 id가 존재한다는 뜻이므로 id_already가 활성화 된다.  
![결과2](https://i.imgur.com/GoJgC0h.png)  
cnt가 0일 경우, 기존에 해당 id가 없으므로 id_ok가 활성화 된다.  
---
title: Strategy Pattern(전략패턴)에 대해 쉽게 이해하기(예제)  
date: 2024-09-20 17:12:38
categories: JAVA     
published: true 
tags:
- JAVA
- GoF Pattern    
---

테스트 코드 강의를 듣다가 본 리팩토링 코드 예제가 무척 깔끔하고 확장성이 좋아보였다. 무엇보다 리팩토링 하기 전의 결합도 높은 소스코드가 평소 내가 많이 사용하는 방식이었다. 이참에 제대로 알아두면 좋겠다 싶어 찾아보니 디자인 패턴 중에 하나였다. `Strategy Pattern(전략 패턴)`이라고 하는데 어떤 개념이고 어떻게 사용하는지 공부해 보았다.   

### 1. Strategy Pattern의 정의  
![이미지](https://i.imgur.com/olGB15R.jpeg)  
전략 패턴은 실행(런타임) 중에 전략을 선택하여 객체의 기능이나 동작을 실시간으로 바꿀 수 있는 `행위 디자인 패턴` 이다. 하나의 인터페이스를 각각 다른 클래스에서 구현해두고, 클라이언트의 요청에 맞추어 컨텍스트에서 메서드를 호출해서 사용한다. 
> ***이럴 때 사용하면 좋다 :***  
> - 동작이 실행 중에 실시간으로 교체되어야 할 때 
> - 구현체가 자주 변경 되거나 추가 될 때
> - 테스트 코드를 작성할 때  

> ***이런 점은 주의 :***  
> - 구현체가 많아질수록 관리해야 할 객체의 수가 늘어난다  
> - 구현체가 자주 변경되거나 추가되지 않는다면 굳이 인터페이스를 따로 빼서 복잡하게 만들 필요가 없다. 
> - 소스코드 파악에 좀 더 시간이 걸릴 수 있다.  

### 2. 쉬운 예시와 함께 알아보기     
![이미지](https://i.imgur.com/GoYttLk.jpeg)  
디자인패턴 이론 만으로는 무슨 말인지 완전히 와닿지 않았다. 이해하기 쉽도록 예시를 작성해보니 좀 더 쉽게 이해가 갔다. 

1. 우선 `뭔가를 끓임`이라는 추상화 상태의 인터페이스를 만든다. 
2. 무엇을 끓일 건지 각각의 클래스에서 인터페이스를 구체화 시킨다. 
3. 받은 재료로 스프를 끓일 건지, 소스를 끓일 건지 결정하고 요리하기 위한 `냄비(Context)`가 필요하다.  
4. `요리사(Client)`는 어떤 재료를 사용할 건지 결정해서 `냄비에 전달`한다.  

이 내용을 코드로 작성해보자.  

```java 
//1. 우선 `뭔가를 끓임`이라는 추상화 상태의 인터페이스를 만든다. 
public interface BoiledStrategy {
    String boiled();
}
```  
```java 
//2. 무엇을 끓일 건지 각각의 클래스에서 인터페이스를 구체화 시킨다.  
public class SoupStrategy implements BoiledStrategy{
    @Override
    public String boiled() {
        return "스프를 끓인다";
    }
}
```  
```java 
public class SourceStrategy implements BoiledStrategy{
    @Override
    public String boiled() {
        return "소스를 끓인다";
    }
}
```  
```java  
//3. 받은 재료로 스프를 끓일 건지, 소스를 끓일 건지 결정하고 요리하기 위한 `냄비(Context)`가 필요하다. 
public class Pot {
    public String cook(BoiledStrategy boiledStrategy){
        // 요리사가 넣은 재료에 따라서 어떤 레시피를 실행 할 지 결정됨
        // =클라이언트가 주입한 매개변수에 따라서 어떤 구현체를 실행 할 지 결정됨
        return boiledStrategy.boiled();
    }
}

```  
```java  
//4. `요리사(Client)`는 어떤 재료를 사용할 건지 결정해서 `냄비에 전달`한다.
public class Chef {
    public static void main(String[] args) {
        Pot pot = new Pot();
        //냄비를 준비한다.
        String soup = pot.cook(new SoupStrategy());
        System.out.println(soup);
        //스프 재료를 넣으면 스프를 요리한다
        String source = pot.cook(new SourceStrategy());
        System.out.println(source);
        //소스 재료를 넣으면 소스를 요리한다
    }
}

```  
이렇게 하면 스프를 죽으로 바꿔야 하는 경우가 생겨도 냄비를 건드리지 않고 스프 클래스만 변경하면 된다. 국을 추가해야 하는 경우에도 새 구체화 메서드만 만들어두면 언제든지 셰프가 냄비에 넣을 수 있다.  


### 3. 좀 더 실무에 가까운 예제        
배달 음식 주문 시스템을 상정하고 예제를 만들어보았다.  
1. 음식 정보가 들어갈 도메인 생성  
```java 
public class Item {
    private String menu;
    private int price;
    private int amount;
    private int tip;
    public Item(String menu, int price, int amount, int tip) {
        this.menu = menu;
        this.price = price;
        this.amount = amount;
        this.tip = tip;
    }
    public String getMenu() {
        return menu;
    }

    public int getPrice() {
        return price;
    }

    public int getAmount() {
        return amount;
    }

    public int getTip() {
        return tip;
    }
}
```
2. `주문하기` 라는 인터페이스 만들기  
```java  
public interface OrderStrategy {
    //주문 기능
    void order(int totalPrice);
}
```  
3. `도보배달 주문`과 `택시배달 주문`으로 구체화 하기  
```java  
public class WalkDeliveryStrategy implements OrderStrategy {

    private String name;
    private String payType;
    private boolean useCoupon;
    private double discount;

    public WalkDeliveryStrategy(String name, String payType, boolean useCoupon, double discount){
        this.name = name;
        this.payType = payType;
        this.useCoupon = useCoupon;
        this.discount = discount;
    }
    @Override
    public void order(int totalPrice){
        int lastPrice = totalPrice;
        int discountPrice = (int)(totalPrice*discount);
        if(useCoupon){
            lastPrice = lastPrice-discountPrice;
        }
        System.out.println("도보 배달 시 총 지불금액은 "+lastPrice+"원입니다.");
    }
}
``` 

```java  
public class TaxiDeliveryStrategy implements OrderStrategy {

    private String name;
    private String payType;
    private String message;

    public TaxiDeliveryStrategy(String name, String payType, String message){
        this.name = name;
        this.payType = payType;
        this.message = message;
    }
    @Override
    public void order(int totalPrice) {
        System.out.println("택시 배달 시 총 지불금액은 "+totalPrice+"원입니다.");
    }
}
``` 
4. 배달 방식을 결정하고 그에 따른 정보를 전달할 `장바구니` 만들기  
```java  
public class OrderCart {
    List<Item> items = new ArrayList<>();

    public void addItem(Item item){
        items.add(item);
    }
    public void totalPrice(OrderStrategy orderStrategy){
        int totalPrice = 0;
        for(Item item : items){
            totalPrice += item.getPrice()*item.getAmount()+item.getTip();
        }
        orderStrategy.order(totalPrice);
    }

}

``` 
5. 사용자가 정보를 장바구니에 전달한다  
```java  
public class User {
    public static void main(String[] args) {
        //주문카트 전략 컨텍스트 등록
        OrderCart cart = new OrderCart();

        //주문 메뉴
        Item A = new Item("새우버거",2000,1,2000);
        Item B = new Item("치즈피자", 5000,2,4000);
        cart.addItem(A);
        cart.addItem(B);

        cart.totalPrice(new TaxiDeliveryStrategy("김주문","현장결제","문 앞에 두고가주세요."));
        cart.totalPrice(new WalkDeliveryStrategy("이배달", "카드결제",true,0.2));
    }
}
``` 

### 4. 테스트 코드에서는 어떻게 활용할까   
여러가지 결과값이 나올 수 있는 테스트 상황을 가정해보자. 예를 들어 `패스워드가 8자리 이상 12자리 이하인지 아닌지`를 테스트 해야 한다고 치자. 글자수 1~12자리를 랜덤으로 출력하면 테스트 코드 결과가 어쩔때는 성공하고, 어쩔때는 실패해서 테스트 코드로 결과를 확인하기가 어렵다. 글자수가 무조건 8~12자리인 경우와 무조건 잘못된 경우를 나누어서 만들어 놓고 테스트하면 좀 더 쉽게 결과를 알 수 있을 것이다.  

1. `패스워드 생성` 인터페이스 작성 
```java  
@FunctionalInterface
public interface PasswordGenerator {
    String generatePassword();
}

```     
2. `항상 성공` `항상 실패`하는 메서드 구체화  
```java  
public class CorrectFixedPasswordGenerator implements PasswordGenerator{
    @Override
    public String generatePassword() {
        return "abcdefgh"; //8글자
    }
}
```  
```java  
public class WrongFixedPasswordGenerator implements PasswordGenerator{
    @Override
    public String generatePassword() {
        return "ab"; //2글자
    }
}
```  
3. 패스워드를 입력받고 정보를 전달할 Context 만들기  
```java  
public class User {
    private String password;
    public void initPassword(PasswordGenerator passwordGenerator){        

        String randomPassword = passwordGenerator.generatePassword();
        /*
        비밀번호는 최소 8자 이상, 12자 이하여야 한다.
         */
        if(randomPassword.length()>=8 && randomPassword.length()<=12){
            this.password = randomPassword;
        }
    }

    public String getPassword() {
        return password;
    }
}

``` 

4. 테스트 코드 작성 후 실행  
```java  
class UserTest {

    @DisplayName("패스워드를 초기화한다.")
    @Test
    void passwordTest() {
        //given - User 객체가 주어졌다
        User user = new User();
        //when - 이 메서드를 호출했을 때
        user.initPassword(new CorrectFixedPasswordGenerator());
        //then
        assertThat(user.getPassword()).isNotNull();
    }

    @DisplayName("패스워드가 요구사항에 부합하지 않아 초기화 되지 않는다.")
    @Test
    void passwordTest2() {
        //given - User 객체가 주어졌다
        User user = new User();
        //when - 이 메서드를 호출했을 때
        user.initPassword(new WrongFixedPasswordGenerator());
        //WrongFixedPasswordGenerator는 메서드 하나만 가진 functional Interface기 때문에
        // user.initPassword(()->"ab"));
        // 이렇게 구현해도 동일하다.

        //then
        assertThat(user.getPassword()).isNull();

    }
}
```
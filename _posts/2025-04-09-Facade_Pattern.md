---
title: 파사드(Facade) 패턴과 Service 레이어의 차이에 대하여  알아보자                   
date: 2025-04-09 16:23:48
categories: JAVA     
published: true 
tags:
- JAVA
- GoF Pattern  
---  
 
개발을 하다보면 컨트롤러 내부 메서드 하나에서 여러 서비스나 레포지토리를 호출해야 하는 경우가 종종 생긴다. 예를 들어, 아래와 같이 회원가입 기능을 구현할 때 `유저 정보 저장, 메일 인증번호 발송, 레디스 캐시 저장, 이벤트 쿠폰 발행` 등의 작업이 같이 수행되는 경우다.  
```java  
public void signup(User user) {
    userRepository.save(user);
    emailService.sendVerificationEmail(user);
    redisService.saveUserCache(user);
    eventPublisher.publishSignupEvent(user);
}
```
이런 경우 파사드 패턴을 쓰면 좋다고 하는데, **그냥 서비스 메서드 하나로 처리하는 것**과 크게 다르지 않아 보였다. 의아한 김에 무슨 차이가 있는지를 알아보았다. 


## 📌파사드 패턴이란 뭘까?   
여러 복잡한 기능을 하나로 묶어서, 외부에는 단순한 인터페이스만 제공하는 구조를 말한다. 즉, 내부는 복잡하더라도 컨트롤러에서는 한 줄로 끝나게 만드는 **리모컨 버튼** 같은 역할이다.   

### 👉 그럼 서비스 메서드랑 뭐가 다른 걸까?  
사실 파사드 패턴은 **특정한 코드 구조**로 정해져 있는 패턴이 아니라, `설계 방법론(또는 개념)`에 가까워서 다양한 방식으로 구현 할 수 있다. 예를 들어 전략 패턴은 인터페이스나 상속 구조가 명확하게 요구되지만, 파사드 패턴은 그런 강제성이 없다.  
그래서 우리가 API를 호출할 때 컨트롤러 하나만 실행하면 되는 점도 일종의 파사드 패턴이라고 볼 수 있고, 서비스 메서드에서 여러 로직들을 처리하는 것 역시 서비스가 파사드처럼 동작하고 있다고 볼 수 있다. 이런 이유로 서비스 메서드가 파사드 패턴과 크게 다르지 않아 보이는 것이다.  

## 📌그럼 서비스 메서드와 파사드를 왜 분리하는 걸까?   

이미 서비스가 파사드처럼 동작하고 있는데 왜 굳이 둘을 분리해야 하는 걸까? 그건 설계 의도와 책임을 고려해야 하기 때문이다. 서비스는 보통 `비즈니스 로직`을 담당하는 레이어다. 예를 들어 회원가입을 처리하거나 주문을 생성하는 등 **도메인 중심의 작업**을 처리한다. 아래의 경우 UserService는 도메인(=User)에 관련된 책임을 수행하고 있다.  

```java 
@Service
public class UserService {
    public void registerUser(UserRequest request) {
        validate(request);
        User user = userRepository.save(request.toEntity());
        emailService.sendWelcomeEmail(user);
    }
}
```
그럼 서비스가 파사드의 역할도 같이 하고 있는 경우, 즉 여러 도메인을 조합해서 복잡한 흐름을 담당하고 있는 경우의 코드도 살펴보자.  
```java 
@Service
public class OrderService {
    
    public OrderResponse placeOrder(OrderRequest request) {
        // 1. 상품 재고 확인
        productService.checkStock(request.getProductId(), request.getQuantity());

        // 2. 결제 처리
        paymentService.process(request.getPaymentInfo());

        // 3. 주문 생성
        Order order = orderRepository.save(...);

        // 4. 알림 발송
        notificationService.send(order);

        return new OrderResponse(order);
    }
}
```
이런 구조는 도메인 책임(=Order)도 갖고 있지만, 동시에 **다른 서브시스템(product, payment, notification)**을 조합하고 있다. 이 경우 OrderService는 `도메인 로직 + 파사드 역할`까지 같이 하고 있는 셈이다.  
이렇게 되면 서비스가 점점 커지고, 책임이 많아져서 **거대 서비스**가 될 수 있다. 그래서 파사드의 역할은 별도의 클래스로 분리하는 것이 권장된다.  

```java  
// 서비스: 진짜 핵심 도메인 로직만 담당
@Service
public class OrderService {
    public Order createOrder(...) {
        // 주문 생성만 집중
    }
}

// 파사드: 여러 도메인 서비스 조합
@Component
public class OrderFacade {
    public OrderResponse placeOrder(OrderRequest request) {
        productService.checkStock(...);
        paymentService.process(...);
        Order order = orderService.createOrder(...);
        notificationService.send(...);
        return new OrderResponse(order);
    }
}

// 컨트롤러
@RestController
public class OrderController {
    private final OrderFacade orderFacade;

    @PostMapping("/orders")
    public OrderResponse placeOrder(@RequestBody OrderRequest request) {
        return orderFacade.placeOrder(request);
    }
}

```

---
title: 스프링 Bean과 Component의 역할과 차이 쉽게 이해하기                 
date: 2025-02-01 14:15:20
categories: Spring           
published: true 
tags:
- Spring           
---  

스프링 Bean이란 뭘까? 한 마디로 요약하자면 `스프링이 관리해주는 객체`를 말한다. 자바에서 일반적으로 객체를 생성할 때를 떠올려보자.  
```java 
MyService myService = new MyService(); 
```
이렇게 선언하면 우리가 객체 생성을 직접 관리해야 한다. 그럼 스프링이 객체를 관리해준다면 어떻게 사용할 수 있을까?  
```java 
@Service
public class MyService {
    public String getMessage() {
        return "Hello, Spring!";
    }
}
``` 
이런 식으로 `@Service`(@Service는 @Component에 속한다)를 붙이면 필요한 곳에서 바로 MyService를 가져다 쓸 수 있다. 이게 바로 스프링의 의존성 주입(DI). 객체 관리를 스프링이 해주면 유지보수성도 더 좋아지지만, 성능 상의 이점도 있다. `new` 키워드로 객체를 계속 만들면 메모리를 많이 차지하지만, 스프링 빈은 기본적으로 싱글톤 패턴을 사용해서 **한 번만 만들어지고 계속 재사용이 가능하다**. 

## 📌스프링 빈 사용법 
1. **@Configuration과 @Bean 사용** 
```java 
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class AppConfig {
    @Bean
    public MyComponent myComponent() {
        return new MyComponent();
    }
}
``` 

2. **@Component 사용**  
```java 
import org.springframework.stereotype.Component;

@Component
public class MyComponent {
    public void hello() {
        System.out.println("안녕하세요! 저는 스프링 빈입니다.");
    }
}
```
1번 방식으로 직접 빈을 등록할 수도 있지만, 컴포넌트로 만들 수도 있다. 둘은 무슨 차이가 있을까? 

|  | @Component | @Bean | 
| --- | --- | --- |
| 등록 방법 | 클래스 위에 @Component 직접 붙임 | @Configuration 내부에서 @Bean 메서드로 등록 |
| 사용 사례 | 우리가 직접 만든 클래스 | 외부 라이브러리 클래스를 빈으로 등록할 때 |
| 자동 등록 | O (스프링이 자동으로 빈으로 등록) | X (개발자가 직접 등록) |

> 📌 @Controller, @Service, @Reposiroty, @RestController는 전부 컴포넌트의 일종이다.  
> 역할에 따라 컴포넌트를 확장한 어노테이션 이름을 사용하여 더욱 명확하게 용도를 확인할 수 있도록 해 둔 것. 그래서 컨트롤러나 서비스 클래스에 @Component를 붙여도 동작은 한다.  

## 📌스프링은 어떻게 컴포넌트를 찾을까  
> 스프링은 컴포넌트를 선언하면 자동으로 빈을 등록해준다고 하는데, 어떤 원리로 그렇게 되는 걸까?   

스프링은 `@ComponentScan`을 사용해서 `@Component`가 붙은 클래스를 자동으로 빈으로 등록해준다. 우리가 따로 `@ComponentScan`을 선언하지 않았는데도 `@Component`가 잘 작동하는 이유는, 컴포넌트 스캔이 `@SpringBootApplication`에도 이미 포함되어 있기 때문이다. 그래서 대부분의 경우에는 따로 설정할 필요가 없다.  
수동으로 설정이 필요한 경우에는 이렇게 선언해서 사용할 수 있다.  

```java 
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

@Configuration
@ComponentScan(basePackages = "com.example") // 특정 패키지 스캔
public class AppConfig {
}

```

## 📌@Bean을 활용하는 경우를 실제로 확인해보자  
Bean은 직접 만든 클래스가 아니라 우리가 수정할 수 없는 외부 라이브러리를 사용할 때 많이 활용한다고 했다. 그럼 실제로 Bean을 등록해서 사용하는 사례를 간단하게 알아보자. 

✅**Jackson의 ObjectMapper를 @Bean으로 등록하는 경우**  
Jackson 라이브러리는 JSON 데이터를 다룰 때 가장 많이 사용되는 라이브러리다. 그 중에서도 ObjectMapper는 **Java 객체를 JSON으로 변환하거나, JSON을 Java 객체로 변환 할 때 사용**한다. 하지만 Jackson의 ObjectMapper클래스를 내가 직접 수정할 수 없기 때문에 `@Bean`을 사용해서 스프링 빈으로 등록하는 것이다.  
```java 
import com.fasterxml.jackson.databind.ObjectMapper;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class JacksonConfig {

    @Bean
    public ObjectMapper objectMapper() {
        return new ObjectMapper();
    }
}
``` 
이렇게 하면 ObjectMapper가 스프링 빈으로 등록되고, 이제 생성자를 통해 주입 받아 사용할 수 있다. 

> 📌예전에는 @Autowired를 사용한 필드 주입을 많이 사용했지만 지금은 권장되지 않고, 생성자를 통한 주입이 추천된다.  
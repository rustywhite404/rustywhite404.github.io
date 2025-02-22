---
title: 스택과 힙, 원시타입과 객체는 어디에 저장될까?           
date: 2025-02-21 21:02:58
categories: CS         
published: true 
tags:
- CS         
---  


![스택과 힙](https://i.imgur.com/z6mRPXU.png)  
간단히 이미지로 정리하자면 스택과 힙은 위와 같은 특성을 가지고 있다.  아래에서는 위 특성을 이해하면 부차적으로 생기는 의문점들에 대해 정리해보았다.  

### ❓ static 변수들은 어디에 저장될까?    
객체들의 `생명주기가 메서드 범위를 넘어걸 수 있기 때문에` 힙에 저장된다면, 메서드 범위 밖에 선언되어 있는 **원시 타입 static 변수**들은 어디에 저장되어야 할까?  
> Heap에 저장된다. 정확히는 JAVA 8 이후로 등장한 `Meta Space` 라는 특수한 공간에 저장된다. 주로 클래스 관련 데이터가 여기에 저장되는데, static 변수들은 클래스 레벨 변수이므로 여기에 저장되는 것이다.   

### ❓ 객체의 멤버 변수들은 어디에 저장될까?   
`객체와 인스턴스 변수는 힙에 저장되고, 참조변수는 스택에 저장`된다. 헷갈릴 수 있으므로 코드와 함께 이해해보자.   

```java 
public class Example {
    public static void main(String[] args) {
        Person person = new Person("Alice", 30);
        int localVar = 10;
    }
}

class Person {
    private String name;
    private int age;

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }
}
``` 
#### 1. 힙 메모리에 할당  
- Person 객체 자체 
- 인스턴스 변수 name(String 객체) 
- 인스턴스 변수 age 

#### 2. 스택 메모리에 할당 
- 참조 변수 person(힙에 생성된 Person 객체의 메모리 주소를 저장함) 
- 지역 변수 localVar 

#### 왜 이런 구조로 저장될까? 
- 객체와 인스턴스 변수가 힙에 저장되어 동적 메모리 할당과 GC가 가능해진다.  
- 참조 변수와 지역 변수가 스택에 저장되어 빠른 액세스와 효율적인 메모리 관리가 가능해진다. 
- 객체의 생명주기가 메서드 호출 범위를 넘어설 수 있고, 여러 참조 변수가 동일한 객체를 가리킬 수 있다.  

### ❓ 원시타입 vs. 객체타입 : 어떤 자료형이 더 메모리 효율성이 좋을까? 
- 원시타입은 값을 직접 저장하고, 추가적인 메모리 오버헤드가 발생하지 않는다. 즉, int 자료형이라면 처음에 사용되는 4바이트 이외의 메모리가 소모되지 않는다.  
- 객체타입은 메모리 주소, 추가 메타데이터 등 더 많은 메모리가 사용된다.  
따라서 다양한 값을 독립적으로 사용하는 경우에는 원시타입을 사용하는 게 더 메모리 효율성이 좋다.  
> 다만 동일한 객체를 여러번 재사용 해야하는 상황이라면 원시 타입을 여러 번 생성하는 것보다 한 번 생성한 객체를 재사용 하는 게 더 효율적일 수 있다. 

### 📌 알고 가면 좋을 것 : String은 원시타입일까, 객체일까?  
결론부터 말하자면 Java에서 String은 객체다. 하지만 다른 언어에서는 원시타입일 수도 있다. 일단 자바를 기준으로 설명하고, 다른 언어들에 대해서도 알아보려 한다.  

#### 왜 String은 원시 타입이 아닐까? 
- 자바에서 String은 `java.lang.String` 클래스로 정의된 객체다. 따라서, String을 생성하면 내부적으로 힙에 저장되는 객체가 생성된다.  
- `new String()`으로 생성이 가능하다. 원시 타입은 new 키워드 없이 값을 저장하지만, String은 `String word = new String()` 형태로 생성이 가능함(int, double 등의 원시 타입은 new로 생성이 불가능함). 
- 원시 타입은 메서드를 가질 수 없지만, String은 `length()`, `toUpperCase()` 같은 여러 메서드를 가지고 있다. 

#### 객체인데 왜 String word="단어"; 처럼 사용이 가능한 걸까?  
String은 워낙 많이 사용되는 자료형이기 때문에, 특별하게 `String Constant Pool(문자열 풀)` 이라는 공간을 가지도록 해 준 것이다. 그래서 `String word="단어";` 처럼 리터럴 방식으로 String을 만들면 String Pool에 저장이 되도록 허용되었다. 같은 값의 String을 여러 번 만들면 String Pool에서 기존 값을 재사용하므로 메모리 절약이 가능하다. 하지만 `new String("단어")`로 만들면 힙 영역에 새로운 객체가 생성된다.  

> 📌리터럴(Literal)이란 뭘까?  
> 소스 코드에서 "값 자체"로 표현되는 데이터를 의미한다. 변수에 할당되는 고정된 값을 말한다고 보면 이해하기 쉽다. 
> 예를 들어, int a = 10; 에서 10이 리터럴이다.  

#### 자바에서 String은 객체인데, 다른 언어에서는 어떨까? 
- JavaScript의 String은 원시 타입이지만 객체처럼 동작한다. 하지만 객체 형태의 String 메서드를 사용하면 내부적으로 객체로 변환(Boxing) 됨.  
   ```javascript 
    let str = "Hello"; // 원시 타입 (Primitive)
    console.log(typeof str); // "string"

    let objStr = new String("Hello"); // 객체 타입 (Object)
    console.log(typeof objStr); // "object" 

    //원시타입 문자열도 메서드(length, toUpperCase 등)를 사용하면 내부적으로 객체로 변환됨. 
    let str = "Hello"; 
    console.log(str.length); // 5 (Boxing: 임시 객체 생성)
    console.log(str.toUpperCase()); // "HELLO"
    ```

- Python에서 String은 객체로 관리된다.  
- C에서 String은 원시 타입이 아니라 char 배열로 관리된다(C에는 String 타입이 없음). 문자열을 다루려면 배열을 직접 다루거나 포인터를 사용해야 함. 
- C++에서 String은 객체다. `std::string`을 사용해서 문자열을 객체처럼 다룰 수 있다. 


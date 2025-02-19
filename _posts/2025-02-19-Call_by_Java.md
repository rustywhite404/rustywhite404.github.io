---
title: 자바 시점의 Call by Value, Call by Reference            
date: 2025-02-19 17:52:12
categories: CS         
published: true 
tags:
- CS         
---  


`자바에서 원시 타입은 Call by Value에 가깝고 객체는 Call by Reference처럼 동작하지만, 엄밀히 말해 자바는 항상 Call by Value를 사용한다.`  
Call by Value, Call by Reference에 대해 공부할 때 항상 나오는 말이지만 이것만 보면 뭔 소린가 싶다🤔. 우선 Call by Value vs. Call by Reference 개념을 간단히 정리하고, 자바에서 객체를 어떻게 전달하길래 **Call by Ref처럼 보이지만 사실은 Call by Value** 라고 하는지 알아보자.    


### ❓ Call by Value (값에 의한 호출)이란      
- 함수가 호출될 때 값의 복사본을 전달하는 방식이다.  
- 즉, 원본 변수는 변경되지 않는다. 
- 함수가 호출 될 때 마다 메모리 사용량이 증가한다. 



### ❓ Call by Reference (참조에 의한 호출)이란      
- 함수가 호출될 때 메모리 주소를 전달하는 방식이다.  
- 즉, 원본 데이터가 변경된다. 
- 메모리 효율적이지만, 데이터 무결성에 주의해야 한다.  

### ❓ 자바에서 객체를 전달할 때 Call by Reference처럼 보이는 이유 
```java 
class Person {
    String name;
    
    public Person(String name) {
        this.name = name;
    }

    void changeName(String newName) {
        this.name = newName;
    }
}

public class CallByValueExample {
    public static void main(String[] args) {
        Person p = new Person("Alice");
        modifyPerson(p);
        System.out.println(p.name); // "Bob"
    }

    public static void modifyPerson(Person person) {
        person.changeName("Bob"); // 객체 내부 값 변경
    }
}

```
이 코드를 살펴보면 `modifyPerson(p);` 를 호출할 때, p의 참조값(메모리 주소)이 함수로 전달된다. 그런 다음 `person.changeName("Bob")`이 호출되면서 person이 가리키는 **원래 객체의 필드 값이 변경**된다.  
즉, p.name값이 Bob으로 변경되면서 Call by Reference처럼 원본 객체가 수정되는 것 같이 보인다.  


### ❓ 그럼에도 불구하고 자바가 Call by Value인 이유  
```java  
class Person {
    String name;
    
    public Person(String name) {
        this.name = name;
    }
}

public class CallByValueProof {
    public static void main(String[] args) {
        Person p = new Person("Alice");
        modifyPerson(p);
        System.out.println(p.name); // "Alice" (변경되지 않음)
    }

    public static void modifyPerson(Person person) {
        person = new Person("Bob"); // 새로운 객체 할당 (원본과 무관)
    }
}

``` 
- `modifyPerson(p);`가 호출 될 때, p의 참조값(메모리 주소)이 복사되어 전달된다. 
- `person = new Person("Bob");`으로 넘어와서, **person이라는 원래 객체의 복사본**이 생긴다. person은 p를 복사해와서 새로운 값을 할당한 것이므로 원본 객체 p에는 영향을 주지 않는다. 
- 즉, `p.name`은 여전히 Alice이다. 

Call by Reference 방식으로 동작한다면 p.name 값도 Bob으로 변경되어야 한다. 하지만 변경되지 않았으므로, 자바가 Call by Value 방식으로 동작한다는 것을 알 수 있다.  

### ❓ 그럼 자바에서 객체를 전달 할 때 어떤 일이 일어나는 걸까? 

1. 객체의 참조값(Person p의 p)이 스택에 저장된다. 
2. 함수에 전달 될 때, 이 참조값이 복사되어 전달된다. 
3. 즉, 함수 내부에서 이 복사된 참조값을 통해 객체를 조작 할 수 있다.
4. 하지만, 이 참조값 자체가 변경(새 객체 할당)되면 원본에 영향을 주지 않는다. 
---
title: (JAVA) static변수, static 메서드를 알아보자     
date: 2021-02-26 21:02:00
categories: 
- JAVA
tags:
- JAVA
- 기초문법
---

### 정적(Static)이란?  
스태틱은 '고정된' 이라는 뜻을 가지고 있어. 각 인스턴스마다 따로 생성되는 변수가 아니라, 클래스 전반에서 공통적으로 사용할 수 있는 기준 변수(와 메서드)를 말하지. 얘네는 Static변수(정적 변수), Static 메서드라고 해.

<p style="color:#999;">※ 제가 편하게 복습하려고 구어체로 작성했습니다.<br />
※ 초보 개발자의 공부 겸 정리 포스팅이라 틀린 부분이 있을 수 있습니다. </p>  

---

### Static 멤버의 생성  
![이미지](https://i.imgur.com/1HfRgOr.png) 
Static 키워드를 통해 생성된 정적 멤버들은 Heap 영역이 아닌 Static 영역에 할당돼. Static 영역에 할당된 메모리는 `모든 객체가 공유하게 돼서, 어디에서든지 참조할 수 있는 장점`이 있어. 하지만 Garbage Collector의 관리 영역 밖에 존재하므로 `프로그램이 종료될 때까지 메모리가 할당된 채로 존재`해. 그러니까 Static을 너무 `남발하면 프로그램의 성능에 악영향`을 줄 수 있다는 걸 기억하고 주의해야 해.  

---
### Static 멤버 선언  

그냥 생성하면 자동으로 일반 인스턴스로 생성되고, static을 붙이면 정적 멤버가 돼. 
```java
static int num = 0; // 스태틱 변수
public static void staticMethod(){} // 스태틱 메서드
```
---
```java
Class Number{
	Static int num = 0; // 스태틱 변수 
	int num2 = 0; // 인스턴스 변수 
}

public class Static_ex{
	public static void main(String[] args){
		
		Number number1 = new Number();
		Number number2 = new Number(); 
		
		number1.num++; //number1의 스태틱 변수 num을 1 증가시킴
		number1.num2++; //number1의 인스턴스 변수 num2를 1 증가시킴
		
		System.out.println(number2.num); //number2의 스태틱 변수 num을 출력
		System.out.println(number2.num2); //number2의 인스턴스 변수 num2를 출력

	}
}
// 콘솔에 찍히는 결과
1
0
```

왜 number2.num의 결과값은 1이고, number2.num2의 결과값은 0이 나왔을까?
⇒ 인스턴스 변수는 인스턴스가 생성될 때 마다 생성되므로 인스턴스마다 각기 다른 값을 가지지만, `정적 변수는 모든 인스턴스가 하나의 저장공간을 공유`하기 때문에 `number1.num에서 저장된 값이 그대로 저장공간에 남아있어서` 이런 결과가 나온 거야.

### Static 변수 사용 예시 2  

스태틱 변수를 어디에 활용할 수 있을까? 학번관리, 카운터 등 다양한 활용이 가능할 거야. 
이번에는 방문 시 마다 방문 횟수를 증가시키는 카운터를 한 번 만들어보자. 

```java
public class Counter{
	int count = 0; // 일반 인스턴스 변수로 선언
	Counter(){
		this.count++;
		System.out.println(this.count);
	}
	public static void main(String[] args){
		Counter c1 = new Counter();
		Counter c2 = new Counter();
	}
}
// 콘솔에 찍히는 결과
1
1 
```

이 경우, c1과 c2 객체를 생성할 때 count 값이 1씩 증가해도, c1과 c2가 서로 다른 메모리를 가지고 있기 때문에 원하던 결과(카운트가 연동되어 증가 되는)가 나오지 않은 거야. 이번에는 스태틱 변수를 사용해보자.

```java
public class Counter{
	static int count = 0; // 스태틱 변수로 선언
	Counter(){
		this.count++;
		System.out.println(this.count);
	}
	
	public static void main(String[] args){
		Counter c1 = new Counter();
		Counter c2 = new Counter();
	}
}
// 콘솔에 찍히는 값
1
2
```

스태틱 변수로 선언하니까 count 값이 공유되어서 다음과 같이 방문자 수가 증가한 결과값이 나오게 됐어. 스태틱 변수는 프로그래밍 시 메모리의 효율 때문 보다는, 이렇게 메모리의 공유를 위한 용도로 더 많이 사용돼. 

---

### Static 메서드 사용 예시

```java
Class Name{
	static void print(){ // 스태틱 메서드
		System.out.println("내 이름은 홍길동입니다.");
	}
	void print2(){
		System.out.println("내 이름은 이순신입니다.");
	}
}

Public class StaticEx{
	public static void main(String[] args){
		Name.print(); //스태틱 메서드는 인스턴스를 생성하지 않아도 호출이 가능
		
		Name name = new Name(); //인스턴스 생성
		name.print2(); // 인스턴스를 생성해야만 호출 가능
	}
}
// 콘솔에 찍히는 결과
내 이름은 홍길동입니다.
내 이름은 이순신입니다.
```

정적 메서드는 클래스가 메모리에 올라갈 때 자동으로 생성돼. 그래서 인스턴스를 생성하지 않아도 호출할 수 있는 거야. 스태틱 메서드는 "오늘의 날짜 구하기", "숫자에 콤마 추가하기" 등의 유틸리티성 메소드를 작성할 때 많이 사용돼.

---

### Static 메서드 사용 예시 2

```java
import java.text.SimpleDateFormat;
import java.util.Date;

public class Util{
	public static String getCurrentDate(String fmt){
		SimpleDateFormat sdf = new SimpleDateFormat(fmt);
		return sdf.format(new Date());
	}
	
	public static void main(String[] args){
		System.out.println(Util.getCurrentDate("yyyyMMdd"));
	}
}
```

Util 클래스의 getCurrentDate라는 스태틱 메서드를 이용하여 오늘의 날짜(yyyyMMdd)를 구하는 예제야. 그 날의 날짜는 매일 변하고, 어제 날짜 위에 오늘 날짜가 덮어씌워져야 메서드를 호출했을 때 정상적으로 오늘 날짜가 출력될테니 스태틱 메서드를 사용했겠지?

---
### Static 키워드 관련 문답

Q. Static 키워드에 대해 설명해보세요.  
A. 스태틱 변수와 메서드가 존재하고, 이 정적인 멤버들은 항상 같은 저장공간을 공유하기 때문에 어디에서 호출해도 값을 유지해요. 그리고 스태틱 멤버들은 객체를 생성하지 않고도 변수나 메서드를 사용할 수 있어요.

Q. 왜 Static 키워드를 쓰나요?  
A. 객체를 생성하지 않고 바로 사용할 수 있어서 편리하고, 저장공간을 공유하기 때문에 카운터 구현이나 학번, 일련번호 생성 등에 유용하게 쓸 수 있습니다.

Q. 모든 변수와 메서드에 Static을 붙이는 게 좋을까요?  
A. 필요한 경우에만 붙이는 게 좋습니다. 스태틱 멤버들은 스택 메모리에 저장되어서 가비지 컬렉터의 관리 영역 밖에 있어요. 그래서 프로그램이 종료될 때까지 메모리를 계속 차지하고 있으므로 남발하면 메모리 낭비를 유발합니다. 

Q. 어떤 경우에 static을 붙일 수 있고, 어떤 경우에 붙일 수 없습니까?  
메서드 내에서 인스턴스 변수를 사용하지 않을 경우 static을 붙이는 것을 고려해볼 수 있습니다. 메서드 내에서 인스턴스 변수를 필요로 하면 스태틱을 붙일 수 없기 때문입니다. 반대로, 인스턴스 변수를 필요로 하지 않는다면, 가능하면 static을 붙이는 게 좋습니다. 메서드 호출 시간이 짧아지기 때문에 효율이 높아지기 때문입니다. 

Q. 당신이 새로운 클래스를 작성한다고 칠 때, 어떤 경우에 static 키워드를 사용해야 한다고 생각합니까?  
A. 모든 인스턴스에 공통적으로 사용하게 될 속성이나 기능에 static을 붙여야 합니다.
즉, 클래스의 멤버변수 중 모든 인스턴스에 공통된 값을 유지해야 하는 것이 있는 지 살펴보고, 있을 경우 스태틱을 붙여줍니다. 작성한 메서드 중에서도 인스턴스 변수를 사용하지 않는 메서드가 있다면 스태틱을 붙일 것을 고려해봅니다.
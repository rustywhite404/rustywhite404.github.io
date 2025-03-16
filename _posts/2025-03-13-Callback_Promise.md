---
title: Callback, Promise, async/await 쉽게 이해하기                
date: 2025-03-13 13:09:14
categories: NodeJS Javascript         
           
published: true 
tags:
- NodeJS                
- Javascript 
---  

자바스크립트는 `싱글 스레드 언어`다. 즉, 한 번에 한 가지 작업만 할 수 있다는 뜻이다.  
웹 서비스는 다양한 동작이 필요하다. `DB 읽기, 파일 읽기, API 요청 보내기, 사용자 입력...` 이 작업 하나하나가 끝날 때까지 기다려야 한다면 어떨까? 파일 하나가 다운로드 완료 될 때까지 기다려야 다음 파일을 받을 수 있고, 파일들이 다운로드 되는 동안에는 다른 동작을 아무것도 할 수 없을 것이다. 그래서 자바스크립트에서는 비동기asynchronous로 작업을 처리하는 경우가 많다. 그렇구나~ 하고 넘어가기에는 또 이상한 점이 있다. **비동기로 작업을 하려면 스레드가 두 개 이상 있어야 하는 거 아닌가?** 하는 점이다.  
쉽게 말해서 카페에 직원이 두 명은 있어야 한 명이 주문 받고 한 명은 음료 제조를 할 텐데, **직원이 하나 뿐이라면 어떻게 작업 분담이 가능하냐**는 것이다.  

### 자바스크립트에서 비동기 작업을 처리하는 방식  
자바스크립트가 비동기 작업을 수행할 때는 자바스크립트 엔진 외부의 `Web APIs(브라우저)`나 `libuv(Node.js)`가 대신 비동기 작업을 처리해준다. 메인 스레드는 하나지만 주변 시스템들이 비동기 작업을 수행해주고, 작업이 끝나면 다시 메인 스레드에게 알려주는 구조다. 그래서 싱글 스레드에서도 비동기와 병렬 처리가 가능해진다. 위에서 말한 카페 비유를 다시 사용하자면, 직원은 1명이지만 로봇 커피머신이나 번호표 기계 등이 처리를 도와주고 있는 것이다. 

어떻게 자바스크립트에서 비동기 작업이 가능한지는 이해했으니, 이제 비동기 작업을 어떻게 처리해야 하는지 몇 가지 방법들을 알아보자. 

## 1️⃣ Callback (콜백)  
**Callback**은 "작업이 끝난 뒤 실행할 함수를 파라미터로 넘겨주는 방식"이다. 즉, **"이 작업 끝나면 이 함수 실행해줘"** 라고 알려주는 것.  

### ✅ 장점
- 간단하고 빠르게 사용할 수 있음
- 오래된 환경에서도 사용 가능

### ❌ 단점
- **콜백 지옥** 가능성 (중첩 구조 안의 중첩구조 안의 중첩구조...)이 있다 
- 에러 처리가 복잡함 
- 코드 가독성 떨어짐

---

### ✅ 예제 코드 (상세페이지 데이터 조회)

```javascript
app.get('/detail/:id', function (req, res) {
    const postId = parseInt(req.params.id);

    db.collection('post').findOne({ _id: postId }, function (error, result) {
        if (error) {
            console.error('❗️ 에러 발생:', error);
            return res.status(500).send('서버 에러!');
        }

        if (!result) {
            return res.status(404).send('게시글 없음!');
        }

        res.render('detail.ejs', { data: result });
    });
});
```

---

## 2️⃣ Promise (프로미스)
**Promise**는 `미래에 작업이 끝나면, 결과를 주겠다는 약속`이다. 즉, 지금은 결과가 없지만, **작업이 끝나면 알려주겠다는 객체**라고 할 수 있다.

### ✅ 장점 
- 콜백 지옥을 어느 정도 해결 가능 
- `.then()`, `.catch()` 으로 **성공, 실패 처리 가능**  
- 비동기 작업을 연결(chain) 가능  

### ❌ 단점
- `.then()`이 많아지면 여전히 복잡해질 수 있음
- try-catch로 에러 처리 불편  

---

### ✅ 예제 코드

```javascript
app.get('/detail/:id', (req, res) => {
    const postId = parseInt(req.params.id);

    db.collection('post').findOne({ _id: postId })
        .then(result => {
            if (!result) {
                return res.status(404).send('게시글 없음!');
            }
            res.render('detail.ejs', { data: result });
        })
        .catch(error => {
            console.error('❗️ 에러 발생:', error);
            res.status(500).send('서버 에러!');
        });
});
```
> 📌 **이게 Promise 방식이라는 걸 어떻게 알아볼 수 있을까?**  
> Promise 객체에서 성공 시 실행할 함수는 `.then()`, 실패 시 실행할 함수는 `.catch()`로 등록한다. 콜백 방식에는 이런 메서드가 없기 때문에 구분이 가능하다. 


---

## 3️⃣ async/await  
**async/await**은 Promise를 더 쉽게 사용할 수 있게 만들어주는 문법이다. 

### ✅ 장점
- 코드가 **순차적으로 읽혀서 직관적**
- 에러 처리도 **try-catch로 깔끔**
- 유지보수, 협업에 용이 → **실무에서 가장 많이 사용된다**

### ❌ 단점
- Promise 기반 함수에서만 사용 가능
- await를 빠뜨리면 Promise 객체 그대로 반환(초보자 때 실수 할 수 있음)

---

### ✅ 예제 코드 

```javascript
app.get('/detail/:id', async (req, res) => {
    try {
        const postId = parseInt(req.params.id);
        const result = await db.collection('post').findOne({ _id: postId });

        if (!result) {
            return res.status(404).send('게시글 없음!');
        }

        res.render('detail.ejs', { data: result });
    } catch (error) {
        console.error('❗️ 에러 발생:', error);
        res.status(500).send('서버 에러!');
    }
});
```
> 📌 **async의 역할**  
> 함수 앞에 async를 붙이면 해당 함수가 무조건 Promise를 반환하게 해준다.  
> 즉, "이 함수는 비동기 함수다" 라고 선언하는 것. **내부에서 await를 사용하려면 반드시 함수 앞에 async가 붙어있어야 한다.** 

> 📌 **await의 역할**  
> Promise 객체 앞에 붙인다. 해당 Promise가 끝날 때까지 기다리겠다는 의미. 
> 반드시 async 함수 안에서만 사용 가능하다. 
```javascript 
async function getData() {
    const result = await db.collection('post').findOne({ _id: 1 });
    console.log(result);
}
```  

### ❗️ 자주 하는 실수 ###  
- ❌ async 없이 await 사용 
```javascript 
const result = await db.collection('post').findOne({ _id: 1 }); // 에러 발생 
``` 
- ❌ async를 붙였는데 await 안 쓴 경우(→ await 안 쓰면 Promise 객체 자체가 출력됨)
```javascript 
async function getData() {
    const result = db.collection('post').findOne({ _id: 1 });
    console.log(result); // Promise { <pending> }
} 
```  
### ✅ 올바른 예시  

```javascript
app.get('/detail/:id', async (req, res) => {
    try {
        const result = await db.collection('post').findOne({ _id: 1 });
        res.send(result);
    } catch (error) {
        res.status(500).send('에러 발생!');
    }
});
``` 
---

## 🎯 최종 요약

| 방식 | 정의 | 장점 | 단점 |
|:-:|:-|:-|:-|
| **Callback** | 작업 끝난 뒤 실행할 함수 전달 | 빠르고 간단 | 콜백 지옥, 가독성 낮음 |
| **Promise** | 작업이 끝난 뒤 결과를 약속하는 객체 | then/catch로 연결 가능, 에러 구분 | then 많아지면 복잡 |
| **async/await** | Promise를 쉽게 사용하기 위한 문법 | 코드 흐름 직관적, 에러 처리 쉬움 | await 빠뜨리면 오류 가능 |


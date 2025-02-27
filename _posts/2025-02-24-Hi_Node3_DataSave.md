---
title: Node.js로 데이터 저장하기 3            
date: 2025-02-27 13:50:34
categories: NodeJS         
published: true 
tags:
- NodeJS         
---  


NodeJS로 간단한 GET/POST API 구현을 해 보았으니 이번에는 데이터를 저장하고 가져오는 것 까지 해보자. DB는 MongoDB를 사용했지만 아무거나 상관없다.  
[🙋‍♂️1. Node.js로 5분만에 API 만들기](https://rustywhite404.github.io/nodejs/2025/02/25/Hi_Node/#)  
[🙋‍♂️2. Node.js로 POST API 만들기](https://rustywhite404.github.io/nodejs/2025/02/26/Hi_Node2_Post/#) 

### MongoDB와 서버 연결하기         
MongoDB에 Free tier로 가입해서 데이터베이스를 먼저 생성하자. 그러면 `mongodb+srv://계정명:계정비밀번호@cluster0.qw0l4.mongodb.net/데이터베이스명?retryWrites=true&w=majority&appName=Cluster0` 이런 형태의 접속 URL이 생성되는데 잘 기억해두어야 DB 접속 정보로 사용이 가능하다. 위 예시에서 총 세 군데의 개인정보를 변경해야 한다.   
그런 다음, 터미널에서 아래와 같이 입력하여 MongoDB 접속을 도와주는 라이브러리를 설치하자. 

```sh 
npm install mongodb 
``` 

설치가 끝났다면 server.js 상단에 아래 코드를 추가해주자. 아까 복사해 둔 접속 URL을 활용해서 uri도 만들어둔다. 
```js 
const MongoClient = require('mongodb').MongoClient;
const uri = "mongodb+srv://계정명:계정비밀번호@cluster0.qw0l4.mongodb.net/데이터베이스명?retryWrites=true&w=majority&appName=Cluster0";
```
이제 DB에 연결하는 코드를 작성해보자. 아래와 같이 작성하면 server.js가 실행될 때 `connectDB();`를 통해 DB 연결이 진행된다. 
```js  
async function connectDB() {
    try {
        const client = new MongoClient(uri);
        await client.connect();
        db = client.db('myDB'); // DB 연결 설정
        console.log("✅ MongoDB 연결 성공!");

        // 서버 시작
        app.listen(8080, () => {
            console.log("🚀 서버가 8080 포트에서 실행 중!");
        });

    } catch (error) {
        console.error("❌ MongoDB 연결 실패:", error);
    }
}

// DB 객체를 다른 모듈에서도 사용할 수 있도록 내보내기
module.exports = { app, db };

// 서버 실행
connectDB();
```

### 데이터를 MongoDB에 저장해보자  
`client.db('todoapp').collection('post').insertOne(추가할 자료, 콜백함수)` 이 패턴의 데이터 추가/수정/삭제의 핵심이니까 잘 기억해두자.  
그리고 또 한 가지 중요한 점은 `res.send()` 부분이 항상 존재해야 한다는 점이다. 전송이 **성공했든 실패했든 반드시 어떤 응답을 서버에서 보내주어야 한다**. 메시지를 보내기 싫다면 응답 코드 리턴이나 리다이렉트 같은 방식을 사용할 수도 있다.  
```js 
app.post('/add', async (req, res) => {
    try {
        const data = req.body; // 클라이언트가 보낸 데이터를 받음
        await db.collection('post').insertOne(data);
        console.log('✅ 데이터 저장 완료:', data);
        res.send("데이터 저장 완료!");
    } catch (error) {
        console.error("❌ 데이터 저장 실패:", error);
        res.status(500).send("데이터 저장 실패!");
    }
});
```  

### 데이터를 가져와서 화면에 보여주려면  
2번 포스팅에서는 정적인 HTML파일을 통째로 가져와서 보여주도록 처리했다. 하지만 실제 데이터를 HTML 중간중간에 넣어주어야 하는 경우가 더 많을 것이다. 서버 데이터를 HTML에 쉽게 삽입할 수 있도록 도와주는 랜더링 엔진인 EJS를 사용해서 데이터를 화면에 불러와보자.  
우선 터미널에서 아래와 같이 EJS를 설치한다.   
```sh 
npm install ejs 
```  

그런 다음 server.js 상단에 아래와 같이 작성한다.  
```js  
app.set('view engine', 'ejs');
``` 
이제 EJS를 사용할 준비는 끝났고, 실제로 써 볼 차례다. EJS 파일은 HTML과 크게 다르지 않다. 주의할 점은 `작업 폴더 내에 views라는 폴더를 만든 후 그 안에 list.ejs파일을 생성해야 한다`는 점. 파일의 내부 코드는 아래와 같은 형식으로 작성해보았다. 
```js  
(views/list.ejs)

<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <title>게시물 목록</title>
</head>
<body>
    <h1>게시물 목록</h1>
    <ul>
        <% posts.forEach(function(post) { %>
            <li><strong><%= post.title %></strong> - <%= post.content %></li>
        <% }); %>
    </ul>
</body>
</html>

```

### EJS 파일에 데이터를 넣어보자 
아래는 EJS의 가장 기본적인 문법이다. 자바에서 타임리프를 사용할 때랑 좀 비슷한 기분이다.  
```html 
<%= 서버에서보낸데이터 %> 

<h2><%= user.name %></h2> //예시 
```  
if문이나 for문을 적용하고 싶을 때에는 <% %> 안에 자바스크립트 문법을 담으면 된다.  
```html  
<% if (user) { %>
  <h2><%= user.name %></h2>
<% } %>
```  


### 이제 MongoDB에서 데이터를 꺼내오자  
- db.collection('post').find()   
- db.colleciton('post').findOne()  

이런 식으로 데이터를 가져올 수 있는데, 이번에는 `find().toArray()`를 사용하기로 했다. 
```js 
app.get('/list', async function(req, res) {
    try {
        const result = await db.collection('post').find().toArray(); // 데이터를 가져옴
        console.log("✅ 조회된 데이터:", result);
        res.render('list.ejs', { posts: result }); // 데이터를 EJS로 전달
    } catch (error) {
        console.error("❌ 데이터 조회 실패:", error);
        res.status(500).send("데이터 조회 실패!");
    }
});
``` 
.render()에 파라미터를 이런 식으로 적어주면 list.ejs파일을 렌더링 하면서 {posts: result} 라는 데이터를 함께 보내줄 수 있다. 즉, result라는 데이터를 posts라는 이름으로 ejs에 전달하게 된다. 

#### find().toArray()는 find()랑 뭐가 다른가요?  
`toArray()`를 붙이면 모든 데이터를 한 번에 배열(Array)로 변환해서 반환한다. 그래서 result를 아래와 같이 바로 사용할 수 있음. 1000개 미만의 소규모 데이터를 처리할 때 편리하다.  
```js  
const result = await db.collection('post').find().toArray();
console.log(result);
```  
`find()`는 아래와 같이 사용한다. result가 아니라 cursor 객체를 반환하는데, 데이터가 바로 리스트화 되는 게 아니라 반복(iterator)하면서 하나씩 가져오게 된다. `forEach()` 또는 `next()`를 사용해야 데이터를 읽을 수 있다. 메모리를 적게 사용하기 때문에 대량의 데이터를 처리할 때 효율적이다. 
```js 
const cursor = db.collection('post').find();
cursor.forEach(post => console.log(post)); // 데이터를 하나씩 출력
``` 
만약 페이징을 구현해야 한다면 아래와 같이 특정 개수만 가져올 수 있다. 
```js 
const posts = await db.collection('post').find().limit(10).toArray();
console.log(posts); // 최대 10개의 문서만 가져옴
``` 

### 결과 확인  
이제 데이터가 잘 저장 되었는지 확인해보자. 2번 포스트에서 /write를 통해 아래와 같이 데이터를 MongoDB에 입력했다. 
![MongoDB](https://i.imgur.com/fEqyK2g.png)
/list를 호출해보면 데이터가 정상적으로 입력되는 것을 확인 할 수 있다. 
![Result](https://i.imgur.com/zvhCssT.png) 

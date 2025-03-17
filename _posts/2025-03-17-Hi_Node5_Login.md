---
title: Node.js로 세션 방식 로그인 구현하기 5            
date: 2025-03-17 19:50:24
categories: NodeJS         
published: true 
tags:
- NodeJS         
---  

  
[🙋‍♂️1. Node.js로 5분만에 API 만들기](https://rustywhite404.github.io/nodejs/2025/02/25/Hi_Node/#)  
[🙋‍♂️2. Node.js로 POST API 만들기](https://rustywhite404.github.io/nodejs/2025/02/26/Hi_Node2_Post/#)  
[🙋‍♂️3. Node.js로 데이터 저장하기](https://rustywhite404.github.io/nodejs/2025/02/27/Hi_Node3_DataSave/#)  
[🙋‍♂️4. Node.js에 저장된 데이터에 인덱스 붙이기, 데이터 삭제하기](https://rustywhite404.github.io/nodejs/2025/03/01/Hi_Node4_CountId/#)  
이번에는 로그인 기능을 구현해보자. 제일 기본적인 세션 저장 방식으로 구현하고, 암호화는 고려하지 않는다(원래는 꼭 비밀번호 암호화 해야 함!).  

### 사전 준비 1 : 라이브러리 설치            
```bash 
npm install passport passport-local express-session
```  
터미널에 위와 같이 입력해서 라이브러리들을 설치해준다. 로그인, 로그인 검증, 세션 생성을 도와주는 라이브러리들이다. 실제 서비스 시에는 express-session 말고, MongoDB에서 세션 데이터를 저장해주는 라이브러리를 이용하는 게 좋다.  

### 사전 준비 2 : 설치한 라이브러리들 require  
```javascript 
const passport = require('passport');
const LocalStrategy = require('passport-local').Strategy;
const session = require('express-session');

app.use(session({secret : '비밀코드', resave : true, saveUninitialized: false}));
app.use(passport.initialize());
app.use(passport.session()); 
```  
위에서 설치한 라이브러리들을 사용하려면 server.js 상단에 이렇게 선언을 해주어야 한다. `app.use()`는 **이 미들웨어를 사용하겠다**는 선언이다.  

### 사전 준비 3 : MongoDB에 가상의 사용자 정보 만들기  
회원가입 기능을 구현하지 않았으므로 로그인 시도를 하려면 임의의 계정이 하나 필요하다. MongoDB에 새 컬렉션을 하나 만들고, `Insert Document` 버튼을 눌러 강제로 회원 데이터를 하나 넣어보자.  
![MongoDB](https://i.imgur.com/VbqgEiG.png) 

이제 사전 준비는 모두 끝났고, 로그인 기능을 구현하면 된다.  
어떤 사람이 로그인 시도를 하면 아이디와 비밀번호가 DB에 있는지 검사하고, 결과가 있으면 세션을 하나 생성하며 성공 페이지로 이동시킨다. 로그인에 실패하면 실패 페이지로 이동시키면 끝이다. 

### 개발 1 : 로그인 페이지 만들기 & 라우팅   
/login으로 방문했을 때 보여줄 페이지를 하나 준비해야 한다. 아래와 같이 login.ejs 파일을 생성했다. (바디 부분만 가져왔음)  
```js 
<body>
    <%- include('nav.html') %>
    <div class="container mt-5">
      <form action="/login" method="POST">
        <h3>Login</h3>
        <div class="form-group">  
          <h5 class="mt-4">아이디</h5> 
          <input class="form-control form-control-sm" type="text" placeholder="id" name="id">
        </div>
        <div class="form-group"> 
          <h5 class="mt-4">비밀번호</h5>
          <input class="form-control form-control-sm" type="password" placeholder="password" name="pw">
        </div>
          <button type="submit" class="btn btn-primary" data-toggle="button" aria-pressed="false">
          저장하기
        </button>
      </form>
    </div>

    (...)
  </body>
``` 
그리고 /login이 호출되면 ejs페이지를 보여줄 수 있도록 라우팅도 해주어야 한다.  
```js 
app.get('/login', function(요청, 응답){
    응답.render('login.ejs'); 
}) 
``` 

### 개발 2 : 아이디와 패스워드 검사  
누군가 login폼에서 ID와 PW를 입력하고 전송 버튼을 누르면, 이 회원 정보가 유효한지 검사해야 한다. 아래와 같이 작성하면 `Local 방식으로 아이디/비밀번호를 인증`받을 수 있다. failureRedirect 부분은 인증에 실패했을 때 이동시킬 경로이다. 
> 이 편리한 기능은 위에서 추가한 passport 라이브러리가 제공하는 기능이다.  
```js  
//passport.authenticate를 통해 응답 전에 local 방식으로 아이디/비밀번호를 인증받을 수 있다. 
app.post('/login', passport.authenticate('local', {
    failureRedirect: '/fail'
}), function (req, res) {
    res.redirect('/'); 
});
``` 

### 개발 3 : 검사를 해주는  세부적인 코드  
2번의 코드만 작성한다고 로그인이 되지는 않는다. 아래와 같이 passport 라이브러리가 정상 동작하도록 추가 코드를 입력해주어야 한다. 직접 창조하는 게 아니라 이렇게 사용해야 한다고 라이브러리의 예제 코드에 나와있다.  
```js 
passport.use(new LocalStrategy({
    usernameField: 'id', //사용자가 제출한 ID가 어디에 적혀있는지 
    passwordField: 'pw', //사용자가 제출한 PW가 어디에 적혀있는지(input name값)
    session: true, //세션 생성 여부 
    passReqToCallback: false, //아이디, 패스워드 말고 다른 정보검사가 필요한지 
}, async function (입력한아이디, 입력한비번, done) {
    try {
        const 결과 = await db.collection('login').findOne({ id: 입력한아이디 });
        if (!결과) {
            return done(null, false, { message: '존재하지 않는 아이디입니다.' });
        }

        if (입력한비번 === 결과.pw) {
            return done(null, 결과); // 로그인 성공
        } else {
            return done(null, false, { message: '비밀번호가 틀렸습니다.' });
        }
    } catch (error) {
        return done(error);
    }
}));
```  

### 개발 4 : 로그인이 성공했다면 세션을 만들자  
로그인에 성공하면 세션 데이터를 만들어서 쿠키로 보내주어야 한다. 이것도 라이브러리를 사용할 수 있도록 맨 위에서 미리 추가해두었다. 아래의 코드만 입력하면 유저 id 데이터를 바탕으로 세션 데이터를 생성하고, 생성된 세션 아이디를 이용 할 수 있다. 

```js 
passport.serializeUser(function (user, done) {
    done(null, user.id); // 세션에 사용자 id 저장
});

passport.deserializeUser(async function (아이디, done) {
    try {
        const 결과 = await db.collection('login').findOne({ id: 아이디 });
        if (!결과) {
            return done(null, false);
        }
        done(null, 결과); // 세션에서 사용자 정보 복원
    } catch (error) {
        done(error);
    }
});
```  

### 테스트 : 정상 동작하는지 확인하기  
이제 `localhost:8080`에서 아까 만들어 둔 계정 정보로 로그인을 시도한 후, 쿠키를 확인해보면 정상적으로 session이 생성되어 있는 것을 볼 수 있다. 
![세션](https://i.imgur.com/PK1pM1i.png)
---
title: Node.js로 POST API 만들기 2            
date: 2025-02-25 20:34:34
categories: NodeJS         
published: true 
tags:
- NodeJS         
---  


앞선 포스팅에서 NodeJS 설치와 간단한 GET API를 구현해보았다. 이번에는 POST 방식으로 데이터를 전송하는 방법을 알아보자.  
[🙋‍♂️1. Node.js로 5분만에 API 만들기](https://rustywhite404.github.io/nodejs/2025/02/25/Hi_Node/#) 

  

### POST 요청을 하기 위해 폼을 만들어보자      
`write.html`에 간단한 입력 폼을 만들어보았다. 그리고 write.html을 호출하는 코드도 server.js에 추가해주어야 한다.  
<details><summary>HTML 코드 보기(길어서 접음 )
</summary>  

```html    
<!doctype html>
  <html lang="en">
    <head>
      <!-- Required meta tags -->
      <meta charset="utf-8">
      <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">

      <!-- Bootstrap CSS -->
      <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@4.4.1/dist/css/bootstrap.min.css" integrity="sha384-Vkoo8x4CGsO3+Hhxv8T/Q5PaXtkKtu6ug5TOeNV6gBiFeWPGFN9MuhOf23Q9Ifjh" crossorigin="anonymous">

      <title>Write Page</title>
    </head>
    <body>
      <nav class="navbar navbar-expand-lg navbar-light bg-light">
        <a class="navbar-brand" href="#">Navbar</a>
        <button class="navbar-toggler" type="button" data-toggle="collapse" data-target="#navbarNavAltMarkup" aria-controls="navbarNavAltMarkup" aria-expanded="false" aria-label="Toggle navigation">
          <span class="navbar-toggler-icon"></span>
        </button>
        <div class="collapse navbar-collapse" id="navbarNavAltMarkup">
          <div class="navbar-nav">
            <a class="nav-item nav-link active" href="#">Home <span class="sr-only">(current)</span></a>
            <a class="nav-item nav-link" href="#">Features</a>
            <a class="nav-item nav-link" href="#">Pricing</a>
            <a class="nav-item nav-link disabled" href="#" tabindex="-1" aria-disabled="true">Disabled</a>
          </div>
        </div>
      </nav>
      <div class="container mt-5">
        <form action="/add" method="POST">
          <h3>Things To Do</h3> 
          <h5 class="mt-4">오늘의 할 일</h5> 
          <input class="form-control form-control-sm" type="text" placeholder="오늘의 할 일" name="title">

          <h5 class="mt-4">세부내용</h5>
          <div class="form-group">
            <textarea class="form-control" id="exampleFormControlTextarea1" placeholder="세부내용" rows="3" name="content"></textarea>
          </div>
          <button type="submit" class="btn btn-primary" data-toggle="button" aria-pressed="false">
            저장하기
          </button>
        </form>
      </div>


      <!-- Optional JavaScript -->
      <!-- jQuery first, then Popper.js, then Bootstrap JS -->
      <script src="https://code.jquery.com/jquery-3.4.1.slim.min.js" integrity="sha384-J6qa4849blE2+poT4WnyKhv5vZF5SrPo0iEjwBvKU7imGFAV0wwj1yYfoRSJoZ+n" crossorigin="anonymous"></script>
      <script src="https://cdn.jsdelivr.net/npm/popper.js@1.16.0/dist/umd/popper.min.js" integrity="sha384-Q6E9RHvbIyZFJoft+2mJbHaEWldlvI9IOYy5n3zV9zzTtmI3UksdQRVvoxMfooAo" crossorigin="anonymous"></script>
      <script src="https://cdn.jsdelivr.net/npm/bootstrap@4.4.1/dist/js/bootstrap.min.js" integrity="sha384-wfSDF2E50Y2D1uUdj0O3uMBJnjuUD4Ih7YwaYd1iqfktj0Uod8GCExl3Og8ifwB6" crossorigin="anonymous"></script>
    </body>
  </html>
``` 
</details>  

```JS 
app.get('/write', function(요청, 응답){
    응답.sendFile(__dirname + '/write.html')
})
```  

그리고 server.js 상단에 아래의 코드를 추가해주자. form 데이터를 받기 위한 코드다. 
클라이언트에서 POST 요청이 오면 form에 입력된 데이터를 서버에서 사용할 수 있도록 req.body에 넣어주는 역할을 한다. 
`extended: true`는 객체 안에 중첩된 객체(중첩된 데이터 구조)를 허용할 지 말지에 대한 설정. 
```js  
app.use(express.urlencoded({extended: true})) 
```

### 제대로 전송이 되었는지 확인  
server.js에 아래 코드를 추가해서, POST 요청이 전달 되었을 때 제대로 데이터가 넘어오는지 확인해보면 끝.  
```js  
app.post('/add', function(req, res){
    console.log(res.body);
    res.send('전송 완료');
})
``` 


다음에는 POST로 받은 데이터를 어딘가에 저장해 보는 것까지 해 볼 예정이다. 
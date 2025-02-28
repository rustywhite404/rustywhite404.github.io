---
title: Node.js에 저장된 데이터에 인덱스 붙이기, 데이터 삭제하기기 4            
date: 2025-02-28 21:10:24
categories: NodeJS         
published: true 
tags:
- NodeJS         
---  

  
[🙋‍♂️1. Node.js로 5분만에 API 만들기](https://rustywhite404.github.io/nodejs/2025/02/25/Hi_Node/#)  
[🙋‍♂️2. Node.js로 POST API 만들기](https://rustywhite404.github.io/nodejs/2025/02/26/Hi_Node2_Post/#)  
[🙋‍♂️3. Node.js로 데이터 저장하기](https://rustywhite404.github.io/nodejs/2025/02/27/Hi_Node3_DataSave/#)  
MongoDB에는 id를 Auto Increment 하는 기능이 없다. 게시판의 글번호 같은 기능을 만들려면 따로 구현이 필요한데, 이번에 어떻게 하는지 알아보자. 그리고 게시물을 삭제하는 것까지 해 볼 생각이다(Update, Delete). 

### MongoDB에 collection 하나 더 만들기          
![mongoDB](https://i.imgur.com/GdCfFQS.png) 
우선 기존에 만들어놓은 `post` 컬렉션 외에 `counter` 컬렉션을 하나 더 만들고, 내부에 임의의 데이터를 하나 넣어보자. totalPost 항목은 `int32`로 설정해준다. 이제 totalPost에 지금까지 몇 번 게시물을 발행했는지 저장할 예정이다.  

구현할 내용의 흐름을 정리해보자면 아래와 같다.   
1. 클라이언트에서 게시물 데이터를 요청 
2. `counter`에서 현재 게시물 총 개수 가져오기 
3. 총 개수 + 1로 새로운 게시물의 `_id` 만들기 
4. `post`에 저장 
5. `counter`의 `totalPost`값 +1로 업데이트 
6. 저장 완료 메시지 반환 

### 기존 코드 수정하기  
원래는 이런 식으로 구현 되어 있었다. 
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
아래와 같이 변경해보자. 
```js 

app.post('/add', async (req, res) => {
    const session = db.client.startSession();

    try {
        const data = req.body; // 요청으로 받은 데이터
        await session.withTransaction(async () => {
            // 1. 게시물 갯수 가져오기
            const counterCollection = db.collection('counter');
            const counter = await counterCollection.findOne({ name: '게시물갯수' }, { session });

            if (!counter) {
                throw new Error("카운터 정보가 없습니다.");
            }

            const 총게시물갯수 = counter.totalPost;

            // 2. post 저장 (게시물 번호 = 총게시물갯수 + 1)
            const newPost = {
                _id: 총게시물갯수 + 1,
                title: data.title,
                content: data.content
            };

            await db.collection('post').insertOne(newPost, { session });
            console.log('✅ 새 게시물 저장 완료:', newPost);

            // 3. counter 업데이트 (totalPost + 1)
            await counterCollection.updateOne(
                { name: '게시물갯수' },
                { $inc: { totalPost: 1 } },
                { session }
            );

            console.log('✅ 게시물 갯수 업데이트 완료');
        });

        res.send("데이터 저장 완료!");
    } catch (error) {
        console.error("❌ 데이터 저장 실패:", error);
        res.status(500).send("데이터 저장 실패!");
    } finally {
        await session.endSession();
    }
}); 

``` 

코드가 많이 길어진 것 같지만 더 간결하게만 작성할 수도 있다. 여기서는 게시물 저장과 카운터 업데이트를 한 번에 처리하기 위해 `withTransaction`을 사용해서 안정성을 확보하느라 코드가 좀 더 길어졌다. counter에 데이터가 없을 경우를 대비해서 예외 처리를 하기도 했다. 게시물 저장 중에 서버가 꺼지거나 하는 경우에 게시물 수가 좀 어긋나도 상관없다면 트랜잭션 처리를 하지 않아도 문제는 없다.  

![결과보기](https://i.imgur.com/uBnAkqT.png)  
이제 게시물 번호를 생성하면서 기존 totalPost를 Update 하는 것까지 구현해보았다.  

### 게시물 삭제를 해보자  
HTTP 요청에는 4가지 종류가 있다. `GET, POST, PUT, DELETE`인데 form에서는 GET, POST 요청밖에 할 수 없다. 그럼 삭제 요청은 어떻게 보내야 할까?  
1. AJAX로 DELETE 요청을 보낸다.  
2. POST 요청을 날려서 DELETE를 수행한다.  

사실 두 가지 방법 모두 작동하고, POST로 삭제 요청을 보내도 상관은 없지만! **Restful한 API를 만들기 위해** 조금 더 번거롭더라도 1번 방법을 사용하는 것이다.  

```js 
<script src="https://code.jquery.com/jquery-3.4.1.min.js"></script>
    <script>
        $.ajax({
          method : 'DELETE',
          url : '/delete',
          data : '1번게시물'
        }).done(function(결과){
          AJAX 성공시 실행할 코드는 여기
        }).fail(function(에러){
          실패시 실행할 코드는 여기
        });
      </script>

``` 
AJAX의 기본적인 사용법은 위와 같다. Jquery를 사용하지 않고도 쓸 수 있는데, 우선은 빠르고 편하게 쓰기 위해 jquery 라이브러리를 추가했다. 위 예시를 실제 코드에 적용해보자. 

```html 
<ul class="list-group">
    <% posts.forEach(function(post) { %>
        <li class="list-group-item">
            <h4><%= post.title %></h4> 
            <p><%= post.content %></p>
            <button type="button" data-id="<%= post._id %>" class="btn btn-secondary delete">Delete</button>
        </li>
    <% }); %>
</ul>
```
button에 `data-id`를 붙여서 각 항목의 _id를 찾을 수 있도록 해 둔 다음, 
```js 
<script>
    $('.delete').click(function(){ 
        const button = $(this); // 현재 클릭한 버튼
        const postId = button.data('id'); // 클릭한 버튼의 data-id 값 가져오기

        $.ajax({
            method: 'DELETE',
            url: '/delete',
            data: { _id: postId } // _id에 실제 게시물 ID 전송
        }).done(function(result){
            alert('삭제 완료!');
            button.closest('li').fadeOut(); // 클릭한 버튼이 속한 li 요소 삭제. fadeOut() 대신 remove()도 가능 
        }).fail(function(err) {
            alert('삭제 실패!');
            console.error('❌ 삭제 요청 실패:', err);
        });

    });
</script>

``` 
각 버튼을 눌렀을 때 AJAX가 실행되도록 처리한다.  
```js 
app.delete('/delete', function(req, res){
    req.body._id = parseInt(req.body._id)
    db.collection('post').deleteOne(req.body, function(error, result){
        console.log('삭제완료')
      })
    res.send('삭제완료')
  });
``` 
server.js에도 `/delete`요청이 들어왔을 때 어떻게 해야 할 지 작성이 필요하다. id값을 숫자로 찾을 수 있도록 request에서 받은 _id를 parseInt 하는 과정이 필요하다. 

#### AJAX Delete 요청은 사실  

```js 
$.ajax({
     method: 'DELETE',
     url: '/delete/1',
})
``` 
이런 식으로, data를 전달하지 않고 url에 바로 담아서 전달하는 경우가 더 일반적이다. 위에서는 AJAX사용법에 최대한 맞추어서 작성해보려고 data에 담았다. 


#### 응답 데이터의 종류 알아보기  
```js 
app.get('/경로', function(요청, 응답){
  응답.send('<p>페이지 보기</p>')
  응답.status(404).send('해당 페이지를 찾을 수 없습니다')
  응답.sendFile('/uploads/ok.png')
  응답.render('list.ejs', { ejs에 보낼 데이터 })
  응답.json(json데이터)
});
``` 

- send : 간단한 문자, html 코드를 보낼 수 있다. 
- status : 응답 코드를 보낼 수 있다. 
- sendFile : static 파일을 보낼 수 있다. 
- render : ejs등의 템플릿이 적용된 페이지들을 렌더링 해준다. 
- json : json 데이터를 담아서 보낸다. 
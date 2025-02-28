---
title: Node.js에 저장된 데이터에 인덱스 붙이기(Update 사용해보기기) 4            
date: 2025-02-28 21:10:24
categories: NodeJS         
published: true 
tags:
- NodeJS         
---  

  
[🙋‍♂️1. Node.js로 5분만에 API 만들기](https://rustywhite404.github.io/nodejs/2025/02/25/Hi_Node/#)  
[🙋‍♂️2. Node.js로 POST API 만들기](https://rustywhite404.github.io/nodejs/2025/02/26/Hi_Node2_Post/#)  
[🙋‍♂️3. Node.js로 데이터 저장하기](https://rustywhite404.github.io/nodejs/2025/02/27/Hi_Node3_DataSave/#)  
MongoDB에는 id를 Auto Increment 하는 기능이 없다. 게시판의 글번호 같은 기능을 만들려면 따로 구현이 필요한데, 이번에 어떻게 하는지 알아보자.  

### MongoDB에 collection 하나 더 만들기          
![mongoDB](https://i.imgur.com/GdCfFQS.png) 
기존에 만들어놓은 `post` 컬렉션 외에 `counter` 컬렉션을 하나 더 만들고, 내부에 임의의 데이터를 하나 넣어보자. totalPost 항목은 `int32`로 설정해준다. 이제 totalPost에 지금까지 몇 번 게시물을 발행했는지 저장할 예정이다.  

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


---
title: 필수적으로 알아야 할 Redis 명령어(feat. 레디스 기본 자료형, 유용한 명령어)          
date: 2025-01-15 16:18:31
categories: Redis         
published: true 
tags:
- Redis        
---  

### **필수적으로 알아야 할 Redis 명령어**

Spring Boot 환경에서 MySQL과 함께 캐시 용도로 Redis를 사용할 거라면 레디스 명령어를 전부 알 필요는 없다. 하지만 사용 중인 자료형에 맞는 주요 명령어를 알고 있으면 문제 해결에도 도움이 되고 설계를 할 때에도 유용할 것 같아 중요한 명령어들과 그에 대응하는 Spring Data Redis 메서드를 정리해봤다. 

| **Redis 자료형** | **주요 명령어** | **Spring Data Redis 메서드** | **설명** |
| --- | --- | --- | --- |
| **String** | `SET`, `GET` | `opsForValue().set()`, `opsForValue().get()` | 키-값 쌍으로 데이터를 저장/조회 |
| **List** | `LPUSH`, `RPUSH`, `LPOP`, `RPOP`, `LRANGE` | `opsForList()` | 리스트 데이터 추가/조회 |
| **Hash** | `HSET`, `HGET`, `HDEL`, `HGETALL` | `opsForHash()` | 해시맵 데이터 관리 |
| **Set** | `SADD`, `SREM`, `SMEMBERS` | `opsForSet()` | 집합 데이터 관리 |
| **Sorted Set** | `ZADD`, `ZRANGE`, `ZREM` | `opsForZSet()` | 정렬된 집합 데이터 관리 |

---

### **레디스 기본 자료형 5가지(String, List, Hash, Set, Sorted Set)**

- **String** : 이미지, 실행파일, 숫자값 등 문자열 이외의 데이터도 저장 가능. 
세션 캐시(쿠키 등), 장바구니 내용 등 일시적인 정보를 저장하거나 방문자 수 등 접근 수 카운트가 가능하기 때문에 웹 애플리케이션 캐시를 레디스에 맡기는 일이 많다.
    - `SET`으로 저장하고 `GET`으로 가져온다.
        
        ```bash
        SET USER_X 100 //OK 
        GET USER_X // "100" 반환 
        ```
        
    - `MSET`으로 여러개를 한 번에 저장하고 `MGET`으로 여러개를 한 번에 가져올 수 있다.
        
        ```bash
        MSET USER_Y 100 USER_Z 200 //OK
        MGET USER_Y USER_Z // 1)"100" 2)"200" 반환 
        ```
        
    - `GETDEL` 로 키값을 가져오고 동시에 삭제하기도 가능하다
        
        ```bash
        127.0.0.1:6379> GETDEL zian
        "jaeheeNEW"
        127.0.0.1:6379> GET zian
        (nil)
        ```
        
    - `SETEX` 명령어를 사용하면 키에 값을 설정하면서 동시에 TTL(생존 시간)을 설정할 수 있다. 아래 예시에서는 TTL을 300초로 설정했고, 다시 실행할 때 마다 초가 줄어들고 있다.
        
        ```bash
        127.0.0.1:6379> SETEX user:7 300 "Yamamoto Sinji"
        OK
        127.0.0.1:6379> TTL user:7
        (integer) 294
        127.0.0.1:6379> TTL user:7
        (integer) 291
        
        127.0.0.1:6379> EXPIRE user:7 300 // 이것도 똑같이 작동 
        127.0.0.1:6379> SET user:7 "Yamamoto Sinji" EX 300 //이것도 똑같이 작동 
        //String형을 사용하고 있다면 SET EX나 SETEX, 다른 자료형이면 EXPIRE를 사용하는걸 추천  
        ```
        
- **List :** 값이 삽입한 순서대로 유지되며, `리스트 좌우에서 모두 값을 삽입하거나 삭제` 할 수 있다. 스택이나 큐로 사용할 수 있음. **중간으로 접근하는 것은 느리며** 데이터가 클수록 느려지니 주의해야 한다.
    
    ```bash
    127.0.0.1:6379> LPUSH mylist foo bar baz
    (integer) 3
    127.0.0.1:6379> LRANGE mylist 0 -1
    1) "baz"
    2) "bar"
    3) "foo" 
    // 0은 리스트의 처음부터 값을 가져오는 것을 의미, -1은 리스트의 마지막을 의미한다.
    // 0 -1은 즉 리스트 내 요소 전부를 꺼내오는 것을 뜻한다. 
    ```
    
    - List형 활용 : 인기 콘텐츠 표시. 
    뉴스 사이트에서 최근 열개의 토픽을 표시하거나 트위터 등의 SNS 타임라인을 표시할 때, 범위를 지정하여 값을 가져올 수 있는 List가 적합하다.
    - 리스트 좌우에서 삽입/삭제
        
        ```bash
        127.0.0.1:6379> LPUSH redislist ccc bbb aaa //리스트 왼쪽에서 삽입 
        (integer) 3
        127.0.0.1:6379> LRANGE redislist 0 -1 
        1) "aaa"
        2) "bbb"
        3) "ccc"
        127.0.0.1:6379> LPOP redislist 2 //리스트 왼쪽에서 2개 꺼냄 
        1) "aaa"
        2) "bbb"
        127.0.0.1:6379> LRANGE redislist 0 -1
        1) "ccc"
        127.0.0.1:6379> RPUSH redislist xxx yyy zzz //리스트 오른쪽에서 삽입 
        (integer) 4
        127.0.0.1:6379> RPOP redislist 2 //리스트 오른쪽에서 2개 꺼냄 
        1) "zzz"
        2) "yyy"
        ```
        
    - 타임라인에 게시된 최신 세 개의 트윗을 표시하는 기능을 List로 구현해보자
        
        ```bash
        //timeline:1이라는 키에, 리스트의 왼쪽부터 값을 삽입 
        127.0.0.1:6379> LPUSH timeline:1 "Hello!"
        (integer) 1
        127.0.0.1:6379> LPUSH timeline:1 "Breakfast Time" "Lunch Time" "Dinner Time!"
        (integer) 4
        
        //LRANGE를 이용하여 범위를 지정. 0번부터 2번까지, 즉 세개의 요소를 가져옴. 
        //모든 요소를 가져오고 싶은 경우 0 -1을 하면 된다(음수를 사용하면 뒤에서부터 요소를 셈) 
        127.0.0.1:6379> LRANGE timeline:1 0 2
        1) "Dinner Time!"
        2) "Lunch Time"
        3) "Breakfast Time"
        ```
        
- **Hash :** 하나의 키에 키-값 쌍이 연결되어 있다. 예를 들어 Cart라는 키 안에 <Apple, 4000>, <Bread, 3000>… 이런 식으로 키-값이 들어갈 수 있다.
    
    ```bash
    127.0.0.1:6379> HSET myhash f1 v1 f2 v2 f3 v3
    (integer) 3
    127.0.0.1:6379> HGET myhash f1
    "v1"
    ```
    
    - `HKEYS` 명령어로 특정 키에 포함된 필드명들을 가져 올 수 있지만, *KEYS  계열 명령어의 시간 복잡도는 O(N)으로 큰 편이기 때문에 운영 환경에서는 사용하지 않는 게 좋다. ⇒ **대신 *SCAN 명령어를 사용하기를 권장** 
    `HVALS`와 `HGETALL`도 사용할 수 있다.
        
        ```bash
        127.0.0.1:6379> HSCAN user:1 0
        1) "0"
        2) 1) "name"
           2) "Taro"
           3) "age"
           4) "30"
           5) "sex"
           6) "male"
           7) "email"
           8) "taro@example.com"
        127.0.0.1:6379> HVALS user:1
        1) "Taro"
        2) "30"
        3) "male"
        4) "taro@example.com"
        127.0.0.1:6379> HGETALL user:1
        1) "name"
        2) "Taro"
        3) "age"
        4) "30"
        5) "sex"
        6) "male"
        7) "email"
        8) "taro@example.com"
        ```
        
- **Set :** 레디스 String형의 집합. 여러 값을 `순서와 중복 없이 저장`한다. 같은 값을 여러번 저장해도 하나의 값만 저장된다. 어떤 리소스에 **태그를 부여하거나, 일일 고유 방문자를 계산하는 등의 연산**에 사용 가능. Set끼리의 교집합, 합집합, 차집합 등을 쉽게 구할 수 있고 특정 Set의 데이터를 다른 Set으로 옮길 수도 있음.
- **Sort Set :** `순서가 있는 Set`으로, **레디스의 특징**이라고도 할 수 있는 자료형. 랭킹이나 활동량 계산 등에 사용하기 좋다. 각 키마다 점수를 가진 멤버로 구성된 하나의 집합이 매핑된다. 
⇒ 키 안에 멤버(유니크)와 점수가 있는 건 Hash형과 비슷하지만 Hash형에는 순서가 없다. 
⇒ 집합 내 멤버에 중복이 허용되지 않는 건 Set형과 비슷하지만, Set만으로는 사용자 정보와 점수를 동시에 가질 수 없다.
    
    ```bash
    //myzset 리스트에 멤버와 점수를 입력하고, 전체 조회  
    
    127.0.0.1:6379> ZADD myzset 123 member1
    (integer) 1
    127.0.0.1:6379> ZADD myzset 321 member2
    (integer) 1
    127.0.0.1:6379> ZADD myzset 700 member3
    (integer) 1
    127.0.0.1:6379> ZADD myzset 200 member4
    (integer) 1
    127.0.0.1:6379> ZREVRANGE myzset 0 -1 WITHSCORES
    1) "member3"
    2) "700"
    3) "member2"
    4) "321"
    5) "member4"
    6) "200"
    7) "member1"
    8) "123"
    ```
    
    ```bash
    //어떤 게임의 사용자 랭킹을 구현해본다 
    127.0.0.1:6379> ZADD rank:event:1 324891 Taro
    (integer) 1
    127.0.0.1:6379> ZADD rank:event:1 276302 Jiro
    (integer) 1
    127.0.0.1:6379> ZADD rank:event:1 255547 Saburo
    (integer) 1
    127.0.0.1:6379> ZADD rank:event:1 12547 Shiro
    (integer) 1
    127.0.0.1:6379> ZREVRANGE rank:event:1 0 -1 withscores
    1) "Taro"
    2) "324891"
    3) "Jiro"
    4) "276302"
    5) "Saburo"
    6) "255547"
    7) "Shiro"
    8) "12547"
    127.0.0.1:6379> ZSCORE rank:event:1 Shiro 
    "12547"
    127.0.0.1:6379> ZREVRANK rank:event:1 Shiro //순위는 0부터 시작 
    (integer) 3
    127.0.0.1:6379>
    ```
    

---

### **레디스 유틸리티 명령어(중 자주 사용되는 것들)**

- **KEYS** 
키 목록을 확인한다. 편리하지만 실행이 오래 걸리므로 `운영 서버에서는 사용을 추천하지 않는다`. 
개발, 테스트 단계에서 사용을 권장함. 운영 환경에서 키 목록을 확인하고 싶을 때는 **SCAN/SSCAN/HSCAN/ZSCAN 등 SCAN 계열의 명령어 사용을 추천**한다(자료형 별로 구분)
    
    ```bash
    KEYS * //모든 키 확인
    KEYS <pattern> //pattern으로 검색 대상 제한 
    ```
    
- **EXISTS** 
키 존재 여부를 확인한다. 인수에 해당하는 키가 존재하면 1, 없으면 0을 반환한다. 해당하는 키가 여러개 있는 경우 매칭된 수를 반환한다.
    
    ```bash
    127.0.0.1:6379> SET X valueX 
    OK
    127.0.0.1:6379> EXISTS X
    (integer) 1
    127.0.0.1:6379> EXISTS Y
    (integer) 0
    127.0.0.1:6379> SET Z valueZ
    OK
    127.0.0.1:6379> EXISTS X Y Z
    (integer) 2 //X, Z는 SET을 통해 입력되어 있지만 Y는 존재하지 않으므로 2개를 반환 
    ```
    
- **TYPE** 
자료형과 기능을 확인
    
    ```bash
    127.0.0.1:6379> SET mykey myvalue
    OK
    127.0.0.1:6379> HSET myhash myfield myvalue
    (integer) 1
    127.0.0.1:6379> TYPE mykey
    string
    127.0.0.1:6379> TYPE myhash
    hash 
    ```
    
- **DEL** 
키를 삭제할 때는 모든 자료형에서 DEL을 사용한다. 반환되는 값은 삭제한 키의 개수이다.
    
    ```bash
    127.0.0.1:6379> MSET L 0 M 0 N 0
    OK
    127.0.0.1:6379> EXISTS L M N
    (integer) 3
    127.0.0.1:6379> DEL L
    (integer) 1
    127.0.0.1:6379> EXISTS L M N
    (integer) 2
    127.0.0.1:6379> DEL M N
    (integer) 2
    127.0.0.1:6379> EXISTS L M N
    (integer) 0
    ```
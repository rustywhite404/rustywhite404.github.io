---
title: 동시성 제어를 통해 레이스 컨디션을 해결하는 방법 2 - DB 락 활용       
date: 2025-01-29 14:10:58
categories: Redis        
published: true 
tags:
- Spring 
- Redis      
- 동시성제어   
- 성능개선       
---  


동시성 제어를 통해 레이스 컨디션을 해결하는 방법 1 :  
<https://rustywhite404.github.io/spring/2024/12/10/Concurrency_control1/>  
동시성 제어를 통해 레이스 컨디션을 해결하는 방법 2 :  
<https://rustywhite404.github.io/spring/2024/12/11/Concurrency_control2/>  
앞서 Synchronized와 DB락을 이용한 동시성 제어 방법을 정리해보았고, Lettuce와 Redisson을 사용하는 방법도 알아보기로 했다.   


### 자바 애플리케이션에서 레디스를 사용하려면

레디스를 사용할 때 자바 애플리케이션에서 편하게 접근할 수 있도록 도와주는 라이브러리로 Lettuce와 Redisson이 있다. 우선 각각의 특징과 장점을 간단히 알아보자. 

### 1. Lettuce

Lettuce는 레디스와 자바 애플리케이션 간의 통신을 위한 비동기/동기 클라이언트 라이브러리다. 스프링에서 자주 사용하는 `Spring Data Redis`가 기본적으로 Lettuce를 사용하고 있음. Lettuce의 장점은 다음과 같다. 

- **비동기 프로그래밍을 지원**한다. 
Netty 기반으로 동작하고, 비동기 방식의 API를 제공해 성능이 좋다. 
Future, CompletionStage 같은 자바의 비동기 기술과도 쉽게 연동이 가능하다.
- 싱글 스레드 기반으로, 연결 당 하나의 스레드를 사용해 메모리 사용량이 적다.
- **Redis 클러스터** 구조에서도 잘 동작하도록 설계되어 있다.
- 간단하고 가볍게 사용할 수 있어 **초급 개발자에게 적합하다.**
- Spring Data Redis를 이용하면 별도의 라이브러리 없이도 기본으로 사용할 수 있다.

```java
import io.lettuce.core.RedisClient;
import io.lettuce.core.api.sync.RedisCommands;

public class LettuceExample {
    public static void main(String[] args) {
        // Redis 클라이언트 생성
        RedisClient redisClient = RedisClient.create("redis://localhost:6379");
        
        // 연결하고 동기 명령 인터페이스 가져오기
        var connection = redisClient.connect();
        RedisCommands<String, String> commands = connection.sync();

        // 데이터 저장
        commands.set("key", "value");

        // 데이터 가져오기
        String value = commands.get("key");
        System.out.println("Value: " + value);

        // 연결 닫기
        connection.close();
        redisClient.shutdown();
    }
}
```

---

### 2. Redisson

Redisson은 Redis의 더 높은 수준의 추상화를 제공하는 클라이언트 라이브러리다. Lettuce와 달리 **분산 데이터 구조**와 다양한 유틸리티를 제공한다. 

- 분산 데이터 구조를 지원하므로, 분산 락을 쉽게 구현이 가능하다. 
Map, Set, Queue, Lock 등의 분산 데이터 구조를 사용 가능.
- 동기/비동기 방식 모두 지원하고, 반응형 프로그래밍도 가능하다.
- 분산 캐시, 분산 락, 분산 세마포어, 메시지 큐 등의 기능이 내장되어 있다.
- Redis 클러스터를 쉽게 설정하고 사용이 가능하다.
- pub-sub 방식으로 구현되어 있기 때문에 lettuce와 비교했을 때 redis에 부하가 덜 간다.
- **별도의 라이브러리를 사용해야 하고, 사용법을 공부해야 한다.**

```java
import org.redisson.Redisson;
import org.redisson.api.RLock;
import org.redisson.api.RedissonClient;
import org.redisson.config.Config;

public class RedissonExample {
    public static void main(String[] args) throws InterruptedException {
        // Redisson 클라이언트 설정
        Config config = new Config();
        config.useSingleServer().setAddress("redis://localhost:6379");
        RedissonClient redisson = Redisson.create(config);

        // 분산 락 사용 예제
        RLock lock = redisson.getLock("myLock");

        // 락 획득
        lock.lock();
        try {
            System.out.println("Lock acquired. Performing task...");
            Thread.sleep(2000); // 작업 수행
        } finally {
            // 락 해제
            lock.unlock();
            System.out.println("Lock released.");
        }

        // Redisson 종료
        redisson.shutdown();
    }
}

```

---

## **Lettuce vs. Redisson 주요 차이점**

| **특성** | **Lettuce** | **Redisson** |
| --- | --- | --- |
| **사용 목적** | 단순 레디스 클라이언트 | 고수준의 분산 기능 제공 |
| **비동기 지원** | 지원 | 지원 |
| **추가 기능** | 없음 (클라이언트에 집중) | 분산 데이터 구조, 분산 락, 유틸리티 |
| **클러스터** | 지원 | 지원 |
| **성능** | 가볍고 빠름 | 기능이 많아 상대적으로 무거움 |

### 어떤 것을 쓰는 게 좋을까?

- 재시도가 필요하지 않은 LOCK은 lettuce, 재시도가 필요한 경우에는 redisson을 사용하면 좋다.
- 재시도가 필요한 경우란, 반드시 락을 획득해야 하는 **중요 작업**(재고 감소, 주문 처리 등 중복 방지가 필수인 작업)이다.
- 재시도가 필요하지 않은 경우는 캐시 갱신이나 단순 조회 등이 있다.

---

## Lettuce 사용법

우선 redis-cli에서 아래 내용을 실습해보자. 

```java
C:\Users\user>docker exec -it [내 컨테이너 ID] redis-cli
127.0.0.1:6379> setnx 1 lock //1이라는 lock을 입력 
(integer) 1 //성공 
127.0.0.1:6379> setnx 1 lock ////1이라는 lock을 다시 입력 시도  
(integer) 0 //이미 1이 존재하므로 실패 
127.0.0.1:6379> del 1 //1 삭제 
(integer) 1 //성공 
127.0.0.1:6379> setnx 1 lock //다시 1이라는 lock 입력 
(integer) 1 //성공 

=> Named Lock의 동작 방식과 유사하다. 
```

이제 스프링에서 구현하는 법을 알아보자. 우선 의존성을 추가해준 후, Repository를 하나 만들어 RedisTemplate을 사용할 준비를 해 주자. 

```java
    implementation 'org.springframework.boot:spring-boot-starter-data-redis'
```

```java
@Component
@RequiredArgsConstructor
public class RedisLockRepository {
    private RedisTemplate<String, String> redisTemplate;

    public Boolean lock(Long key){
        return redisTemplate.opsForValue().setIfAbsent(generateKey(key), "lock", Duration.ofMillis(3_000));
    }

    public Boolean unlock(Long key){
        return redisTemplate.delete(generateKey(key));
    }

    private String generateKey(Long key) {
        return key.toString();
    }
}
```

- `RedisTemplate<String, String>`: Redis에서 값을 읽고 쓰기 위한 도구라고 생각하면 된다. 여기에서는 키와 값을 String으로 저장하는 데에 사용했다.
- `redisTemplate.opsForValue().setIfAbsent(...)`: Redis의 `SETNX` 명령어에 해당한다. **특정 키가 존재하지 않을 경우에만 값을 저장**하고, 성공하면 TRUE, 실패하면 FALSE를 반환한다.
- `Duration.ofMillis(3_000)` : 락을 3초간 유지. 즉, 3초 뒤에는 자동으로 만료된다. 
즉, 이 코드는 SETNX와 TTL(Time to Live)을 조합해서 간단한 분산 락을 구현한 예제이다.

```java
@Component
@RequiredArgsConstructor
public class LettuceLockStockFacade {
    private final RedisLockRepository redisLockRepository;
    private final StockService stockService;

    public void decrease(Long id, Long quantity) throws InterruptedException {
        while (!redisLockRepository.lock(id)){
            //레디스에 가는 부하를 줄이기 위해 lock을 획득하지 못했다면 잠시 대기
            Thread.sleep(100);
        }

        //lock 획득에 성공했다면
        try {
            stockService.decrease(id, quantity);
        }finally {
            redisLockRepository.unlock(id);
        }
    }
}
```

서비스 레벨에서 단일 책임 원칙을 지키기 위해 Facade 패턴을 사용해 새 클래스를 만든다. Lettuce는 Spin Lock 방식(Lock 획득 실패 시 레디스로 계속해서 요청을 보냄)이라 부하를 줄이기 위해 lock을 획득하지 못했을 경우 `Thread.*sleep*(100);` 처럼 대기 시간을 꼭 만들어주어야 한다. 


> 📌
**포트 설정은 따로 안 해도 괜찮을까?**  
spring-boot-starter-data-redis에서 Redis 관련 설정을 자동으로 스캔하고 읽어오기 때문에, 따로 명시하지 않으면 기본적으로 localhost:6379을 사용한다. 만약 6380 포트 등으로 변경하고 싶다면 application.yml을 다음과 같이 변경하면 된다. 

```java
spring:
  redis:
    host: 127.0.0.1
    port: 6380
```
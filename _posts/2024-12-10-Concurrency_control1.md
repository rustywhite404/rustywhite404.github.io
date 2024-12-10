---
title: 동시성 제어를 통해 레이스 컨디션을 해결하는 방법 1 - Synchronized        
date: 2024-12-10 12:49:18
categories: Spring        
published: true 
tags:
- Spring     
- 동시성제어   
- 성능개선       
---  

### 동시성 제어란 어떤 상황에서 필요할까

여러 스레드에서 동시에 같은 요청을 보낼 때 데이터 정합성에 문제가 생기는 경우가 있다. 예를 들어 커머스 플랫폼에 선착순 20명만 구매가 가능한 인기상품이 업로드 되고, 동시에 사용자들이 구매를 진행하려 할 때, 동시성 처리가 제대로 되지 않으면 확보한 재고보다 훨씬 많은 수량이 판매되어 문제가 생길 수도 있다. 이런 경우 동시성 제어를 위해 고려해 볼 수 있는 방법들을 고민하고 정리해보았다. 

```java  
@Entity
@NoArgsConstructor
@AllArgsConstructor
@Getter
public class Stock {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private Long productId;

    private Long quantity;

    public Stock(long productId, long quantity) {
        this.productId = productId;
        this.quantity = quantity;
    }

    public void decrease(Long quantity){
        //감소시켰을 때 재고가 0보다 작아지면 exception 발생
        if(this.quantity-quantity<0){
            throw new RuntimeException("재고는 0개 미만이 될 수 없습니다.");
        }
        //0개 미만이 아니라면 재고 감소
        this.quantity -= quantity;
    }

}
```  

```java  
@Service
@AllArgsConstructor
public class StockService {
    private final StockRepository stockRepository;
    //상품 id와 수량을 받아서 재고를 감소시키는 기능
    @Transactional
    public void decrease(Long id, Long quantity){
        //stock 조회
        Stock stock = stockRepository.findById(id).orElseThrow();
        //재고를 감소시킨 뒤
        stock.decrease(quantity);

        //갱신된 값을 저장
        stockRepository.saveAndFlush(stock);
    }
}

```  

위 코드는 구매 요청이 들어오면 상품의 id와 수량을 받아서, 현재 해당 상품의 수량을 확인하고 재고를 감소 시키는 간단한 기능이다. 이제 여러개의 스레드에서 동시에 구매 요청이 발생했을 때 재고가 어떻게 달라지는지 확인해보자.  

>**save와 saveAndFlush는 어떻게 다를까?**  
>`save`메서드는 메서드가 종료될 때 트랜잭션에 의해 모든 작업이 한 번에 커밋된다. > 메서드 중간에 에러가 발생하면 전체 작업이 롤백되어, 데이터베이스는 원래 상태로 돌아간다. 즉, 엔티티를 영속성 컨텍스트에 저장해뒀다가 트랜잭션이 커밋될 때 자동으로 flush한다. 
> 
> 반면, `saveAndFlush`는 엔티티를 저장한 후 즉시 데이터베이스에 반영한다. 
> ⇒ 트랜잭션 롤백이 발생하더라도 이미 커밋된 내용이 자동으로 롤백되지 앟는다. 그래서 > 데이터 정합성이 중요한 기능에는 사용하지 않는 게 좋다. 
> 
> **saveAndFlush를 사용하는 상황 :** 트랜잭션 내에서 데이터의 최신 상태가 필요할 때(트랜잭션 안에 데이터를 즉시 저장한 후, 그 데이터를 기반으로 다른 작업을 해야 할 때), 테스트 환경에서 즉시 반영이 필요할 때 등 
> 
> 이 예제에서는 saveAndFlush로 데이터를 즉시 반영해 실시간성을 유지하려고 했을 때의 결과를 보기 위해서 saveAndFlush를 사용했지만, 일반적으로는 save를 사용하고 다른 동시성 제어 전략을 사용하는 게 좋다. 

```java  
@SpringBootTest
class StockServiceTest {
    @Autowired
    private StockService stockService;
    @Autowired
    private StockRepository stockRepository;

    @BeforeEach
    public void before() {
        //재고가 있어야 테스트가 가능하므로 테스트 전에 재고를 넣어줌
        stockRepository.saveAndFlush(new Stock(1L, 100L));
    }

    @AfterEach
    public void after() {
        stockRepository.deleteAll();
    }

    @Test
    public void 재고감소() {
        stockService.decrease(1L, 1L);
        //100-1 = 99개 가 남아있는지 확인
        Stock stock = stockRepository.findById(1L).orElseThrow();
        assertEquals(499, stock.getQuantity());
    }

    @Test
    public void 동시에_100개의_요청() throws InterruptedException {
        int threadCount = 100;
        ExecutorService executorService = Executors.newFixedThreadPool(32); 
        //32개의 스레드를 가진 스레드풀을 만든다. 
        CountDownLatch latch = new CountDownLatch(threadCount);
        //CountDownLatch: 여러 스레드가 작업을 완료할 때까지 대기하도록 하는 동기화 도구 
        //초기값으로 threadCount(100)를 설정했으니, 
        //100개의 작업이 완료되면 래치가 0이 되어 다음 단계로 넘어간다.
        for (int i = 0; i < threadCount; i++) {
		        //executorService.submit()를 통해 100개의 작업을 스레드풀에 제출 
            executorService.submit(() -> {
                try {
		                //각 스레드는 ID 1L의 재고를 1L씩 감소시킴 
                    stockService.decrease(1L, 1L);
                } finally {
		                //마지막에 호출되어 카운트를 줄인다 
                    latch.countDown();
                }

            });
        }
        //모든 작업이 완료되기 전까지 다음 코드로 진행하지 않고 대기 
        latch.await();
        
        //결과 확인 
        Stock stock = stockRepository.findById(1L).orElseThrow();
        //100 - (1*100) = 0 예상
        assertEquals(0, stock.getQuantity());
        //예상과 다르게 남은 재고는 93개였다(실행 환경마다 오차 있음)        
    }

}
```  

동시에_100개의_요청() 메서드를 실행했을 때의 예상 재고량은 100개의 요청에 따라 100개의 물건이 나갔으므로 0개지만, 실제로 코드를 실행해보면 93개가 남아있었다. 이유는 **레이스 컨디션**(둘 이상의 스레드가 공유 데이터에 액세스 가능한 상황에서, 동시에 변경하려고 할 때 발생하는 문제)이 발생했기 때문이다.

---  

### 동시성 제어를 통해 레이스 컨디션을 해결하는 방법

여러 사람이 동시에 물건을 구매하려 할 때 재고에 문제가 생기지 않도록 동시성 제어를 하려면 어떤 전략을 취해야 할까? 다양한 방법들 중 상황에 맞게 가장 적절한 방법을 골라야 하므로, 여러가지 동시성 제어 방법들을 알아보았다. 

### 1. Synchronized 키워드 사용

메서드에 자바에서 제공하는 synchronized를 붙이면 해당 메서드는 한번에 한개의 스레드만 접근이 가능해진다.  즉, 서비스 레벨(비즈니스 계층)에서 메서드를 한 번에 한 스레드만 접근 가능하도록 동기화 할 수 있다.  

```java  
//@Transactional <- 함께 사용할 수 없음 
public synchronized void decrease(Long id, Long quantity) {
    // 감소 로직(생략) 
}
```  

 synchronized는 간단하게 동시성 처리가 가능하다. 하지만  synchronized가 가진 **한계**가 있다. synchronized는 서버가 한 대일 때는 괜찮은 방법이지만, 서버가 여러대일 경우 각각의 스레드에서 같은 요청을 처리할 때 결국 또 레이스컨디션이 발생할 수 있으므로, **서버가 여러대인 환경에서는 권장되지 않는다.** 그리고 **Transactional어노테이션과 함께 사용할 수 없다**는 점도 기억해두어야 한다. 

>**Transactional어노테이션을 사용하면 synchronized가 정상적으로 작동하지 않는 이유 :** 
> Transactional는 메서드가 실행되기 전에 트랜잭션을 시작하고, 메서드가 끝나면 트랜잭션을 커밋하거나 문제가 생기면 롤백한다. 이 과정에서 원래의 클래스 대신 프록시 객체를 만들어서 트랜잭션을 관리한다. synchronized는 같은 객체를 기준으로 락을 걸기 때문에, 위와 같이 프록시 객체와 원본 객체가 다를 경우 락이 적용되지 않는다. 
> 
> **→ 그렇다면 사용할 수 있는 방법은?**  
> **방법 1 :** @Transactional을 유지하면서 Pessimistic Lock(비관적 락)이나  Optimistic Lock을 사용해 DB 수준에서 동시성 문제를 해결  
> **방법 2 :** 트랜잭션 범위를 최소화 하고, 필요한 부분에만 synchronized를 적용   
> **방법 3 :** @Transactional과 Redis 같은 분산 락을 조합해 사용 


---

아래의 방법으로 동시성 제어를 하는 법은 다음 포스팅에서 정리할 예정이다. 이런 게 있다는 것까지만 정리해둔다. 

### 2. 데이터베이스 레벨의 락 활용

1. Pessimistic Lock(비관적 락) : 데이터에 대해 쓰기 잠금을 설정하여 다른 트랜잭션이 접근하지 못하게 한다. ⇒ 데드락(교착상태) 발생 가능성이 있으므로 주의해서 사용해야 한다. 
2. Optimistic Lock(낙관적 락) : 데이터를 읽고, 업데이트 하는 동안 데이터가 변경되지 않았는지 버전 정보를 통해 확인. 충돌 발생 시 재시도를 통해 처리한다. 
3. Named Lock(이름을 가진 락) : 특정 자원을 식별할 수 있는 고유한 이름을 기반으로 락을 걸고, 이를 통해 동시성을 제어. ⇒ 락이 해제되지 않거나, 특정 스레드가 락을 점유한 상태로 멈추면 데드락이 발생 할 수 있다. 

### 3. Redis와 같은 분산 락 사용

- Redis 같은 **분산 데이터 스토리지**를 사용해 여러 서버 간 동기화를 관리할 수 있다.
- Redis에서 **분산 락**을 구현하면서 Named Lock 방식을 사용할 수도 있다. 이를 위해 **Redisson** 라이브러리를 활용하면 편리하다.

### 4. Message Queue 사용

- RabbitMQ나 Kafka같은 메시지 큐를 사용해 작업 요청을 순차적으로 처리
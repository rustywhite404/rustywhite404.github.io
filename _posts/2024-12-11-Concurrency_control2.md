---
title: 동시성 제어를 통해 레이스 컨디션을 해결하는 방법 2 - DB 락 활용       
date: 2024-12-11 12:20:18
categories: Spring        
published: true 
tags:
- Spring     
- 동시성제어   
- 성능개선       
---  


동시성 제어를 통해 레이스 컨디션을 해결하는 방법 1 : <https://rustywhite404.github.io/spring/2024/12/10/Concurrency_control1/>  
앞서 Synchronized 을 이용한 동시성 제어 방법을 정리해보았고, 여기서는 데이터베이스 레벨의 락을 활용해 볼 예정이다. DB레벨 락은 아래와 같은 방식들이 있다. 하나씩 장단점과 사용법을 알아보자.  


### Pessimistic Lock(비관적 락)

1. **사용법** 
    
    우선 Pessimistic Lock을 사용하기 위해서는 **Repository레벨**에서 `@Lock()` 어노테이션을 달아주어야 한다. 이 잠금은 JPA의 쿼리 실행 시점에 적용되기 때문에, Repository처럼 JPA의 리포지토리 계층에서 실행되는 SQL 쿼리에만 영향을 미친다. 따라서 service 계층 등에서 Lock을 걸려고 하면 제대로 동작하지 않으니 주의해야 한다. 
    
    ```java
    public interface StockRepository extends JpaRepository<Stock, Long> {
    
        @Lock(LockModeType.PESSIMISTIC_WRITE)
        @Query("select s from Stock s where s.id= :id")
        Stock findByIdWithPessimisticLock(Long id);
    }
    ```
    
    ```java
    @Service
    @AllArgsConstructor
    public class PessimisticLockStockService {
        private final StockRepository stockRepository;
    
        @Transactional
        public void decrease(Long id, Long quantity){
            Stock stock = stockRepository.findByIdWithPessimisticLock(id);
            stock.decrease(quantity);
            stockRepository.save(stock);
        }
    }
    ```
    
    ![image.png](https://i.imgur.com/Q2pwFqI.png)  
    
    → 쿼리의 `for update` 부분에서 LOCK이 걸려있음을 확인 할 수 있다. 
    
    1. **Lock의 다양한 종류**  
        - **PESSIMISTIC_WRITE :** 트랜잭션이 완료 될 때까지 다른 트랜잭션에서 쓸 수 없도록 잠금. 읽기와 쓰기가 모두 차단된다. 동시에 여러 트랜잭션이 처리되지 못하고 대기 상태가 되므로, 트랜잭션이 많으면 성능 저하가 발생할 수 있다. ⇒ 동시성 요구가 낮고 데이터 충돌 가능성이 높은 작업에서 주로 사용함.
            
            ```java
            @Lock(LockModeType.PESSIMISTIC_WRITE)
            @Query("SELECT u FROM User u WHERE u.id = :id")
            Optional<User> findByIdWithWriteLock(@Param("id") Long id);
            ```
            
        - **PESSIMISTIC_READ** : 다른 트랜잭션에서 해당 데이터를 읽기만 가능하도록 막아주고, 쓰기 작업은 대기 상태로 잠금
            
            ```java
            @Lock(LockModeType.PESSIMISTIC_READ)
            @Query("SELECT u FROM User u WHERE u.id = :id")
            Optional<User> findByIdWithReadLock(@Param("id") Long id);
            
            ```
            
            <aside>
            📌
            
            PESSIMISTIC_READ와 PESSIMISTIC_WRITE 중 어떤 방식을 사용해야 할까? 아래 특성 중에서 내 서비스에 더 적합한 것을 고민해보아야 한다. 
            
            </aside>
            
            | **기능** | **PESSIMISTIC_READ** | **PESSIMISTIC_WRITE** |
            | --- | --- | --- |
            | **사용 사례** | 재고 조회(읽기 작업이 주요한 시스템) | 계좌 이체(쓰기 작업이 주요한 시스템) |
            | **읽기 작업 가능 여부** | 가능 (다른 트랜잭션에서 읽기 허용) | 불가능 (읽기와 쓰기 모두 차단) |
            | **쓰기 작업 가능 여부** | 불가능 (다른 트랜잭션에서 쓰기 차단) | 불가능 (다른 트랜잭션에서 쓰기 차단) |
            | **목적** | 데이터 읽기 작업의 일관성 보장 | 데이터 수정 시 동시성 문제 방지 |
        - **PESSIMISTIC_FORCE_INCREMENT** : 잠금을 걸면서, 해당 엔티티의 버전을 강제로 증가시켜 충돌 방지 
        이 모드는 다른 모드들과 달리, 해당 엔티티의 VERSION 값을 강제로 증가시킨다. 
        증가한 버전 값은 Optimistic Lock에서 사용하는 @Version 필드와 연관된다. 
        이 모드는 **Optimistic Lock과 Pessimistic Lock을 혼합해서 사용하는 경우**에 유용하다.
            
            ```java
            @Lock(LockModeType.PESSIMISTIC_FORCE_INCREMENT)
            @Query("SELECT u FROM User u WHERE u.id = :id")
            Optional<User> findByIdWithForceIncrement(@Param("id") Long id);
            ```
            
    2. **주의점** 
        - Pessimistic Lock은 **트랜잭션 범위** 안에서만 유효하다. 해당 쿼리를 호출할 서비스 계층에서 `@Transactional`을 반드시 붙여야 한다.
        - DB 설정에 따라 잠금 대기 시간이 너무 길어질 수 있으므로 적절한 설정이 필요하다.
        - 충돌 가능성이 낮은 상황이라면 Optimistic Lock을 사용하는 게 더 효율적일 수 있다. 필요에 따라 `@Version`을 사용하는 것도 고려해 보면 좋다.
            
            <aside>
            📌
            
            **@Version 이란?**
            
            `@Version`은 **Optimistic Lock**에서 사용되는 필드다. JPA는 이 필드를 사용해 데이터 충돌을 감지한다. 엔티티에 `@Version` 필드를 추가하면, 데이터 수정 시 자동으로 **버전 값**이 증가하므로 두 트랜잭션이 같은 데이터를 수정하려고 하면, 버전 값이 서로 달라져 **충돌을 감지**할 수 있다. 
            
            </aside>
            
    3. **장점** 
    - 데이터 정합성이 보장된다.
    - 충돌이 빈번하게 일어난다면 Optimistic Lock보다 Pessimistic Lock의 성능이 더 좋을 수 있다.
    1. **단점**
    - 락을 걸기 때문에 성능 저하가 발생할 수 있다.

---

### Optimistic Lock(낙관적 락)

Pessimistic Lock이 데이터를 잠가 충돌을 막는 방식이라면, Optimistic Lock은 **충돌을 허용**하고, 마지막에 **충돌을 감지**해서 처리하는 방식이다. 주로 JPA의 `@Version`을 사용해서 동작한다. 버전 필드의 값은 데이터가 수정될 때 마다 자동으로 증가하는데, 각 스레드에서 데이터가 수정될 때 현재 버전 값이 저장된 값과 같은지 확인하고 다르면 충돌로 간주한다. 
동작 흐름 : 트랜잭션 A가 데이터를 읽음(버전 값 : 1) → 트랜잭션 B가 동일한 데이터를 읽음(버전 값 : 1) → 트랜잭션 A가 데이터를 수정하고 커밋(버전 값 : 2로 변경) → 트랜잭션 B가 데이터를 수정하려고 시도(버전 값이 다르기 때문에 Exception 발생) 

1. **사용법**
    
    ```java
    @Entity
    public class Stock {
    
        @Id
        @GeneratedValue(strategy = GenerationType.IDENTITY)
        private Long id;
    
        private Integer quantity;
    
        @Version // Optimistic Lock에서 사용
        private Integer version;
    
        // Getter, Setter
    }
    ```
    
    ```java
    public interface StockRepository extends JpaRepository<Stock, Long> {
        @Lock(LockModeType.OPTIMISTIC)
        @Query("select s from Stock s where s.id= :id")
        Stock findByIdWithOptimisticLock(Long id);
    }
    
    ```
    
    Optimistic Lock은 충돌했을 때 재시도를 하게 되므로 **예외처리가 중요**하다. 서비스 로직 안에서 바로 처리할 수도 있지만, 재고감소 클래스에서는 재고 감소 기능만 하고 재시도 기능은 따로 두는 것이 더 좋을 것 같다. **Facade 디자인 패턴**을 이용하여 아래와 같이 클래스를 옮겨서 처리 할 수 있다. 
    
    ```java
    
    @Component
    @AllArgsConstructor
    public class OptimisticLockStockFacade {
        private final OptimisticLockStockService optimisticLockStockService;
        public void decrease(Long id, Long quantity) throws InterruptedException {
            int retryCount = 0;
            while (true){ //재시도 횟수를 제한해 무한루프 방지
                try {
                    optimisticLockStockService.decrease(id, quantity);
                    break; //정상 작동하면 break
                }catch (Exception e){
                    retryCount++;
                    Thread.sleep(50); //실패하면 50ms 후에 재시도
                    if(retryCount==100) throw new RuntimeException("Stock을 감소시키는 데에 실패했습니다.");
    
                }
            }
        }
    }
    
    ```
    
2. **주의점**
    - 충돌 가능성이 낮은 작업, 즉 여러 트랜잭셔이 동시에 접근해도 같은 데이터를 수정할 확률이 낮은 경우 사용하는 게 좋다. (ex. 주문 내역 작성, 댓글 수정)
    - 재시도 로직을 설계할 때 재시도 횟수를 제한하고, 실패 시 사용자에게 알리는 UI&UX를 추가하는 게 좋다.
    - 핫스팟 데이터(인기 상품의 재고)에 여러 트랜잭션이 접근하면 충돌이 잦아질 수 있으므로 적합하지 않다.
3. **장점** 
    - DB 락의 하나라고 하지만, 실질적으로 잠금을 거는 게 아니기 때문에 대기 시간이 없다.
    - 읽기 작업이 많은 시스템에서 성능이 더 우수하다.
    - 잠금이 없으므로 데드락 문제가 원천적으로 차단된다.
4. **단점** 
    - 데이터 수정 충돌 시 예외가 발생하므로, 재시도 로직을 구현해두어야 한다.
    - 데이터 충돌 가능성이 높은 시스템에서는 충돌이 자주 발생해 성능이 떨어질 수 있다.

## **5. Optimistic Lock vs Pessimistic Lock**

| **특징** | **Optimistic Lock** | **Pessimistic Lock** |
| --- | --- | --- |
| **잠금 방식** | 잠금 없음 (충돌 감지) | 데이터 잠금 |
| **성능** | 읽기 작업에 유리 | 쓰기 작업에 유리 |
| **충돌 처리** | 충돌 시 예외 발생 | 충돌 자체를 차단 |
| **사용 사례** | 충돌 가능성이 낮은 경우 | 충돌 가능성이 높은 경우 |
| **데드락 가능성** | 없음 | 있음 |

---

### Named Lock(이름을 가진 락)

특정 이름을 가진 락을 생성하여, 해당 락을 소유한 트랜잭션만 접근을 허용하도록 만드는 방식이다. 비관적 락과 유사하지만, 데이터베이스가 제공하는 이름을 기준으로 락을 관리한다는 점이 다르다. 

1. **사용법** 
    - 락을 설정할 이름을 지정 :
        
        ```java
        public interface LockRepository extends JpaRepository<Stock, Long> {
            @Query(value = "select get_lock(:key, 3000)", nativeQuery = true)
            void getLock(String key);
        
            @Query(value = "select release_lock(:key)", nativeQuery = true)
            void releaseLock(String key);
        }
        ```
        
    - 트랜잭션이 해당 이름을 가진 락을 설정 :
    - 락 해제 : 트랜잭션이 끝나면 락을 해제해야 다른 트랜잭션이 해당 락을 사용할 수 있게 된다.
        
        ```java
        @Component
        @AllArgsConstructor
        public class NamedLockStockFacade {
            private final LockRepository lockRepository;
            private final StockService stockService;
        
            @Transactional
            public void decrease(Long id, Long quantity){
        
                try {
                    lockRepository.getLock(id.toString());
                    stockService.decrease(id, quantity);
                }finally { //앞 과정이 모두 끝나고 나서 락 해제 
                    lockRepository.releaseLock(id.toString());
                }
            }
        }
        ```
        
        ```java
        //위에서 stockService.decrease(id, quantity); 메서드를 사용했는데, 이 때 주의점! 
        //Propagation.REQUIRES_NEW를 해주지 않으면 synchronized 때와 같은 문제가 발생한다.
        @Transactional(propagation = Propagation.REQUIRES_NEW)
            //새로운 트랜잭션을 항상 시작하고, 만약 이미 기존 트랜잭션이 있다면 잠시 중단하고 새 트랜잭션을 시작한 후, 새 트랜잭션이 끝나면 기존 트랜잭션으로 돌아간다.
            
            public void decrease(Long id, Long quantity){
                //stock 조회
                Stock stock = stockRepository.findById(id).orElseThrow();
                //재고를 감소시킨 뒤
                stock.decrease(quantity);
        
                //갱신된 값을 저장
                //saveAndFlush는 엔티티를 영속성 컨텍스트에 저장 -> 트랜잭션이 커밋될 때 자동 flush 하는 게 아니라
                //엔티티를 저장한 후 즉시 데이터베이스에 반영한다.
                stockRepository.saveAndFlush(stock);
            }
        ```
        
2. **장점** 
    - 이름을 통해 명확한 락 관리가 가능하고, 사용이 편하다. 타임아웃을 쉽게 구현 할 수 있다.
    - 분산 락을 구현할 수 있다.
    - 비관적 락과 마찬가지로 잠금 처리가 되어 있는 동안 다른 트랜잭션이 진행되지 않기 때문에 성능 저하가 생길 수 있다.
    - 여러 트랜잭션이 서로 다른 락을 기다리며 데드락 상태에 빠질 수 있다. 이때 트랜잭션을 강제로 종료하거나 롤백하는 등의 처리가 필요하다.
    - 데이터베이스에 의존적인 기능이라 데이터베이스 종류에 따라 다르게 동작할 수 있다.
3. **단점**
    - 락 해제를 잘 해줘야 하고, 실무에서 사용할 때는 구현이 복잡할 수 있다.

---
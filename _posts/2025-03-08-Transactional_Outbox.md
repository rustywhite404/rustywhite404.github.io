---
title: 트랜잭셔널 아웃박스(Transactional Outbox) 패턴 알아보기                
date: 2025-03-08 14:09:14
categories: Spring           
published: true 
tags:
- Spring           
---  

MSA구조의 서비스들이 있다고 생각해보자. 하나의 서비스에서 데이터를 변경하면 다른 서비스들에게도 변경되었음을 알려야 하는 경우가 많다. 예를 들어 주문 서비스에서 주문이 생성되었을 때, 결제 서비스와 재고 서비스가 이 이벤트를 감지해야 결제와 재고 차감을 할 수 있다. 이를 위해 메시지 브로커(Kafka, RabbitMQ)를 사용한다고 치자. 어떤 문제가 발생할 수 있을까?  

1. 🔥 주문 서비스는 DB 반영에 성공했지만, 이벤트(메시지) 전송이 실패한 경우  
주문은 DB에 저장되었지만 결제와 재고 서비스는 이를 알지 못한다. -> 데이터 불일치 발생  
2. 🔥메시지는 저장 되었지만, 데이터베이스가 롤백되면?  
결제 서비스가 이벤트를 감지하고 결제를 진행했지만 이미 주문이 취소된 경우에도 데이터 불일치가 발생한다. 

트랜잭셔널 아웃박스(Transactional Outbox) 패턴은 **트랜잭션과 메시지 브로커의 일관성을 보장하는 방법 중 하나**이다. 

## 📌어떻게 사용해야 할까? 
Transactional Outbox패턴의 구현 방법에 대해 알아보자.  
1. 트랜잭셔널 아웃박스 테이블을 만든다.  
    - `outbox`라는 테이블을 생성하고, 여기에 `전송할 메시지를 저장`한다.  

2. 비즈니스 로직과 메시지 저장을 같은 트랜잭션에서 실행한다.  
    - 주문을 저장하는 트랜잭션 안에서 동시에 `outbox`테이블에도 메시지를 저장한다. 

3. 백그라운드에서 메시지를 메시지 브로커로 전송한다.  
    - 별도의 프로세스(스케줄러, Debezium 등)가 `outbox` 테이블을 조회하여 메시지를 가져와 Kafka 또는 RabbitMQ로 전송한다.  

흐름 자체는 생각보다 간단하다. 구현을 위해 예제 코드도 작성해보자.  

```java 
@Entity
@Table(name = "orders")
public class Order {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String product;
    private int quantity;

    @Enumerated(EnumType.STRING)
    private OrderStatus status;
}
```

```java 
@Entity
@Table(name = "outbox")
public class OutboxMessage {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String topic;
    private String payload;
    private LocalDateTime createdAt;
}
```

```java 
@Service
public class OrderService {
    private final OrderRepository orderRepository;
    private final OutboxRepository outboxRepository;

    @Transactional
    public void createOrder(String product, int quantity) {
        // 1. 주문 생성
        Order order = new Order(product, quantity, OrderStatus.CREATED);
        orderRepository.save(order);

        // 2. 아웃박스에 메시지 저장 (트랜잭션 내부에서 함께 저장됨)
        OutboxMessage message = new OutboxMessage("order-events", "{ \"orderId\": " + order.getId() + " }", LocalDateTime.now());
        outboxRepository.save(message);
    }
}
``` 
`OrderService`에서 주문 생성과 Outbox에 메시지 저장을 하나의 트랜잭션으로 처리한다. 

```java 
@Component
public class OutboxProcessor {
    private final OutboxRepository outboxRepository;
    private final KafkaTemplate<String, String> kafkaTemplate;

    @Scheduled(fixedRate = 5000)
    public void processOutbox() {
        List<OutboxMessage> messages = outboxRepository.findAll();
        for (OutboxMessage message : messages) {
            kafkaTemplate.send(message.getTopic(), message.getPayload());
            outboxRepository.delete(message); // 성공적으로 전송되면 삭제
        }
    }
}
```
`OutboxProcessor`가 주기적으로 Outbox 테이블을 조회하고 메시지를 전송한다.  
메시지 전송이 실패하면 DB에 남아 있으므로 재시도가 가능하다.  

## 📌다른 대안은 없을까?  
트랜잭션이 실패했을 때를 대비한 대안은 여러가지가 있다. 대표적인 몇 가지를 알아보자.  

1. **이중 트랜잭션 (Two-Phase Commit, 2PC)**  
DB와 메시지 브로커에 모두 트랜잭션을 걸어서 처리하는 방법이다. 하지만 분산 트랜잭션이 필요하고 성능이 떨어져서 권장되지 않는다. 

2. **Outbox + Debezium CDC**  
Outbox 테이블을 직접 조회하는 대신, CDC(Change Data Capture) 기술을 이용해서 변경 사항을 감지하고 메시지를 브로커로 전송한다. Debezium 같은 CDC 툴을 활용하면 더욱 안정적인 메시지 전송이 가능하다. 

3. **SAGA 패턴**  
트랜잭션을 여러 개의 **보상 트랜잭션(Compensating Transaction)**으로 분리해서 데이터 일관성을 유지하는 방법이다. 트랜잭션 실패 시, 이전 상태로 되돌리는 롤백 로직이 필요하다. 

요즘 많이 사용하는 SAGA 패턴과 비교했을 때의 장단점도 좀 더 구체적으로 알아보자. 

## ✅ 트랜잭셔널 아웃박스 패턴의 장점 vs. SAGA 패턴

| 비교 항목          | 트랜잭셔널 아웃박스 패턴 | SAGA 패턴 |
|-------------------|-----------------------|-----------|
| **일관성 유지**   | 단일 트랜잭션을 활용하여 강력한 일관성 제공 | 최종적 일관성 (Eventually Consistent) |
| **개발 복잡도**   | 단순함 (DB 테이블과 백그라운드 워커만 필요) | 복잡함 (각 서비스마다 보상 트랜잭션 구현 필요) |
| **에러 처리**     | 실패한 메시지는 Outbox에 남아 재처리 가능 | 롤백이 필요하며 보상 트랜잭션 설계가 중요 |
| **성능**         | 메시지 저장은 빠르지만, Outbox 테이블의 부하가 있음 | 데이터베이스 부하 없음, 대신 서비스 간 요청이 많아짐 |
| **확장성**       | Outbox 테이블이 커질 수 있어 성능 최적화 필요 | 서비스 간 호출이 많아지면서 Latency 증가 |

---

## ✅ 트랜잭셔널 아웃박스 패턴의 단점 vs. SAGA 패턴

| 비교 항목         | 트랜잭셔널 아웃박스 패턴 | SAGA 패턴 |
|------------------|-----------------------|-----------|
| **DB 부하**     | Outbox 테이블에 지속적인 INSERT/DELETE 발생 → 부하 증가 | 분산 트랜잭션이므로 특정 서비스 DB에 부하 없음 |
| **지연 시간**   | 배치 프로세스가 동작할 때까지 메시지 전송 지연 가능 | 서비스 간 직접 호출이 많아 응답 시간이 길어질 수 있음 |
| **데이터 정리 필요** | Outbox 테이블을 주기적으로 정리해야 함 | 추가적인 데이터 정리는 필요 없음 |
| **트랜잭션 롤백 어려움** | 메시지는 트랜잭션에 포함되지만, 이후 전송 실패 시 롤백이 어려움 | 보상 트랜잭션을 활용해 데이터 복구 가능 |

## 📌트랜잭셔널 아웃박스 패턴이 많이 사용되는 경우  
1. 이벤트 기반 아키텍처에서 데이터 일관성이 중요한 경우  
예를 들어, 주문이 성공했을 때 결제 요청을 반드시 전송해야 하는 경우 DB 트랜잭션 내부에서 메시지를 Outbox 테이블에 저장하면 메시지 전송이 보장된다. 
2. Kafka, RabbitMQ 같은 메시지 브로커를 사용하고 있는 경우  
대량의 이벤트를 메시지 브로커로 전달해야 할 때 사용한다. Debezium 같은 CDC(Change Data Capture) 시스템과 함께 사용하면 DB 부하 없이 안정적인 메시지 처리가 가능하다. 
3. 대부분의 금융, 커머스, ERP 시스템에서 선호  
데이터 일관성이 매우 중요한 시스템에서는 트랜잭셔널 아웃박스를 더 선호하는 경향이 있다. 금융 거래, 주문 시스템 등에서는 "돈을 받았는데 주문이 안 되거나, 주문이 됐는데 결제가 안 되는 문제"를 방지해야 하기 때문이다.  

주문 시스템(Order Service) → 결제 서비스(Payment Service) 같은 흐름에서 트랜잭셔널 아웃박스 패턴이 많이 사용된다. 

## 📌사가 패턴이 많이 사용되는 경우  
1. 긴 시간(수초~수분) 동안 수행되는 비즈니스 트랜잭션이 필요한 경우  
예를 들어, 항공권 예약 시스템에서는 좌석을 예약하고, 결제를 승인받고, 호텔 예약을 처리하는 등의 과정이 포함된다. 모든 작업을 한 번에 끝낼 수 없기 때문에 각 단계별 보상 트랜잭션(롤백)을 정의하는 SAGA 패턴이 적합하다.
2. 마이크로서비스 간 트랜잭션이 필요한 경우 
단순한 이벤트 전파가 아니라, 여러 개의 마이크로서비스가 협력해서 작업을 수행해야 할 때. 트랜잭셔널 아웃박스로 해결할 수 없는 복잡한 워크플로우가 있을 때 유용하다.  
3. 고객 요청에 따른 주문을 처리하는 경우
배달 앱 (배민, 쿠팡이츠 등)을 살펴보면 `주문 요청 → 재고 확인 → 결제 승인 → 배달 요청` 등의 과정이 있다. 하나라도 실패하면 이전 작업을 취소해야 할 때 보상 트랜잭션을 많이 사용한다.  

아고다, 에어비앤비 등의 예약 시스템이나 은행 대출 프로세스 (대출 심사 → 신용 평가 → 승인 → 실행)에서 많이 사용된다.  

> 🚀 대규모 시스템에서는 둘을 혼합해서 사용하는 경우가 많다고 한다.  
예를 들어, Netflix는 트랜잭셔널 아웃박스 패턴을 활용해 Kafka로 메시지를 보낸 후,
SAGA 패턴을 활용해 사용자 구독 처리(결제 → 인증 → 콘텐츠 제공)를 수행한다. 
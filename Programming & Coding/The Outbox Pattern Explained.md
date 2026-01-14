# The Outbox Pattern Explained: How Spring Boot Microservices Avoid Data Inconsistency

![Outbox Pattern Header](https://miro.medium.com/v2/resize:fit:720/format:webp/1*B8QvyjuHayk-rc1B0bmHcQ.png)

## 1. What is the Outbox Pattern?
The **Outbox Pattern** guarantees **reliable event publishing** in microservices **without data inconsistency**.

It ensures that:
1. Database update
2. Event/message publish

Either both happen or neither happens.

**Problem it solves:**
“Database updated, but event not sent” (or vice-versa)

---

## 2. Real-World Problem (Why Outbox is Needed)
### Without Outbox (Very Common Bug)
**Order Service**
1. Save order in DB
2. Publish `OrderCreated` event to Kafka
3. **Network issue occurs**

**Result:**
* Order is saved
* Event is **NOT** published
* Inventory / Payment services never get notified
* System becomes inconsistent



---

## 3. Real-Time Use Case
**E-commerce (Amazon-like)**
**Services:**
* Order Service
* Inventory Service
* Payment Service
* Notification Service

**Flow:**
1. User places order
2. Order Service saves order
3. Order Service publishes `ORDER_CREATED` event
4. Inventory reserves stock
5. Payment starts processing

**Outbox ensures step 2 & 3 are atomic.**

---

## 4. Core Idea of Outbox Pattern
Instead of publishing events directly, store them in the database.

**Key Rule:**
* Same database transaction
* One write = business data + event data



---

## 5. How It Works (Internally)
### Step-by-Step Flow
1. **Client** → Order Service
2. **Database Transaction**
   - Save Order
   - Save Event to OUTBOX table
3. **Commit Transaction**
4. **Background Publisher**
   - Reads OUTBOX table
   - Publishes event to Kafka
   - Marks event as SENT

---

## 6. Database Design
**Order Table**
```text
orders
------
id
product_id
quantity
status
```

**Outbox Table**
```text
outbox_events
-------------
id
aggregate_type
aggregate_id
event_type
payload (JSON)
status (NEW/SENT)
```

---

## 7. Spring Boot Implementation

### 7.1 Entity: Order
```java
@Entity
@Table(name = "orders")
public class Order {

    @Id
    @GeneratedValue
    private Long id;

    private Long productId;
    private int quantity;
    private String status;
}
```

### 7.2 Entity: OutboxEvent
```java
@Entity
@Table(name = "outbox_events")
public class OutboxEvent {

    @Id
    @GeneratedValue
    private Long id;

    private String aggregateType;
    private Long aggregateId;
    private String eventType;

    @Lob
    private String payload;

    private String status; // NEW, SENT
}
```

### 7.3 Repositories
```java
public interface OrderRepository extends JpaRepository<Order, Long> {}

public interface OutboxRepository extends JpaRepository<OutboxEvent, Long> {
    List<OutboxEvent> findByStatus(String status);
}
```

### 7.4 Order Service (Important Part)
**Same Transaction Saves Order + Event**
```java
@Service
public class OrderService {

    private final OrderRepository orderRepo;
    private final OutboxRepository outboxRepo;
    private final ObjectMapper objectMapper;

    public OrderService(OrderRepository orderRepo,
                        OutboxRepository outboxRepo,
                        ObjectMapper objectMapper) {
        this.orderRepo = orderRepo;
        this.outboxRepo = outboxRepo;
        this.objectMapper = objectMapper;
    }

    @Transactional
    public void createOrder(Long productId, int qty) throws Exception {

        Order order = new Order();
        order.setProductId(productId);
        order.setQuantity(qty);
        order.setStatus("CREATED");

        orderRepo.save(order);

        OutboxEvent event = new OutboxEvent();
        event.setAggregateType("ORDER");
        event.setAggregateId(order.getId());
        event.setEventType("ORDER_CREATED");
        event.setPayload(objectMapper.writeValueAsString(order));
        event.setStatus("NEW");

        outboxRepo.save(event);
    }
}
```
* If DB fails → nothing saved
* If Kafka is down → event still safe in DB

---

## 8. Event Publisher (Background Worker)
This runs **separately** from business logic.

```java
@Component
public class OutboxPublisher {

    private final OutboxRepository outboxRepo;
    private final KafkaTemplate<String, String> kafkaTemplate;

    public OutboxPublisher(OutboxRepository outboxRepo,
                           KafkaTemplate<String, String> kafkaTemplate) {
        this.outboxRepo = outboxRepo;
        this.kafkaTemplate = kafkaTemplate;
    }

    @Scheduled(fixedDelay = 5000)
    @Transactional
    public void publishEvents() {

        List<OutboxEvent> events = outboxRepo.findByStatus("NEW");

        for (OutboxEvent event : events) {
            kafkaTemplate.send("order-events", event.getPayload());
            event.setStatus("SENT");
            outboxRepo.save(event);
        }
    }
}
```

---

## 9. Inventory Service (Consumer)
```java
@KafkaListener(topics = "order-events")
public void handleOrderCreated(String message) {
    // Deserialize
    // Reserve stock
}
```
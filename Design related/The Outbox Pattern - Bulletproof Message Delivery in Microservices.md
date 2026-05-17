# The Outbox Pattern: Bulletproof Message Delivery in Microservices

> *How to eliminate the dual-write problem and build reliable event-driven systems without distributed transactions*

---

## The Core Problem: You Can't Atomically Write to Two Systems

Imagine a simple order service. When a customer places an order, you need to:
1. Save the order to your database
2. Publish an `OrderCreated` event to a message broker (Kafka, RabbitMQ, etc.)

The instinctive implementation looks like this:

```java
@Service
public class OrderService {
    private final OrderRepository orderRepository;
    private final KafkaTemplate<String, OrderEvent> kafkaTemplate;

    @Transactional
    public Order createOrder(CreateOrderRequest request) {
        Order order = orderRepository.save(new Order(request));
        kafkaTemplate.send("order-events", new OrderCreatedEvent(order));
        return order;
    }
}
```

This code has a fatal flaw. It silently fails in at least three scenarios:

| Failure Scenario | Outcome |
|---|---|
| Kafka is down when `send()` is called | Order saved, event never published — silent data loss |
| Database commits, event sent, but then transaction rolls back | Ghost event in Kafka for an order that doesn't exist |
| App crashes between `send()` and transaction commit | Ambiguous state — did the event go through? |

This is the **dual-write problem**: you're trying to make two independent systems agree atomically, but without a distributed transaction (which is expensive and fragile), you simply can't guarantee it with naive approaches.

The Outbox Pattern is the standard solution. It's elegant because it sidesteps the problem entirely.

---

## The Outbox Pattern: The Key Insight

> **Stop writing to two systems. Write to one, and let a relay handle the second.**

The outbox pattern works in two phases:

**Phase 1 — Atomic write:** Within a single database transaction, save your business entity *and* an event record to an `outbox_events` table. Since both writes go to the same database, they're atomic by default. Either both succeed or both roll back.

**Phase 2 — Relay:** A separate process (the *outbox publisher*) polls the `outbox_events` table, publishes events to the broker, and marks them as processed.

```
┌─────────────────────────────────────────────────────┐
│  Same Database Transaction                          │
│                                                     │
│  ┌──────────────┐       ┌──────────────────────┐   │
│  │  orders      │       │  outbox_events       │   │
│  │  id: 42      │       │  aggregate: Order    │   │
│  │  status: NEW │       │  event: OrderCreated │   │
│  │              │       │  processed_at: NULL  │   │
│  └──────────────┘       └──────────────────────┘   │
└─────────────────────────────────────────────────────┘
                              │
                              │  (Separate process polls)
                              ▼
                    ┌──────────────────┐
                    │   Kafka / MQ     │
                    └──────────────────┘
```

---

## Implementation

### Step 1: The Outbox Table

```sql
CREATE TABLE outbox_events (
    id            UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    aggregate_type VARCHAR(255) NOT NULL,  -- e.g. "Order", "Payment"
    aggregate_id   VARCHAR(255) NOT NULL,  -- the entity's ID
    event_type     VARCHAR(255) NOT NULL,  -- e.g. "OrderCreated"
    payload        TEXT         NOT NULL,  -- JSON-serialized event
    created_at     TIMESTAMP    NOT NULL DEFAULT now(),
    processed_at   TIMESTAMP,              -- NULL = pending
    retry_count    INT          NOT NULL DEFAULT 0,
    last_error     TEXT                    -- last failure message
);

CREATE INDEX idx_outbox_pending ON outbox_events (created_at)
    WHERE processed_at IS NULL;
```

The partial index on `WHERE processed_at IS NULL` keeps polling queries fast even as the table grows.

### Step 2: The JPA Entity

```java
@Entity
@Table(name = "outbox_events")
public class OutboxEvent {

    @Id
    @GeneratedValue(strategy = GenerationType.UUID)
    private UUID id;

    @Column(name = "aggregate_type", nullable = false)
    private String aggregateType;

    @Column(name = "aggregate_id", nullable = false)
    private String aggregateId;

    @Column(name = "event_type", nullable = false)
    private String eventType;

    @Column(name = "payload", columnDefinition = "TEXT", nullable = false)
    private String payload;

    @Column(name = "created_at", nullable = false)
    private Instant createdAt = Instant.now();

    @Column(name = "processed_at")
    private Instant processedAt;

    @Column(name = "retry_count", nullable = false)
    private int retryCount = 0;

    @Column(name = "last_error")
    private String lastError;
}
```

### Step 3: Writing to the Outbox

```java
@Service
@RequiredArgsConstructor
public class OrderService {

    private final OrderRepository orderRepository;
    private final OutboxEventRepository outboxRepository;
    private final ObjectMapper objectMapper;

    @Transactional  // One transaction covers BOTH writes
    public Order createOrder(CreateOrderRequest request) {
        Order order = orderRepository.save(new Order(request));

        OutboxEvent event = OutboxEvent.builder()
            .aggregateType("Order")
            .aggregateId(order.getId().toString())
            .eventType("OrderCreated")
            .payload(objectMapper.writeValueAsString(
                OrderCreatedEvent.from(order)
            ))
            .build();

        outboxRepository.save(event);

        return order;  // Transaction commits here — both rows or neither
    }
}
```

> **Tip:** Encapsulate outbox event creation in a factory or domain event helper to keep your service layer clean. Avoid scattering `objectMapper.writeValueAsString(...)` calls everywhere.

### Step 4: The Outbox Publisher (Polling Approach)

```java
@Component
@RequiredArgsConstructor
@Slf4j
public class OutboxPublisher {

    private final OutboxEventRepository outboxRepository;
    private final KafkaTemplate<String, String> kafkaTemplate;
    private final OutboxPublisherProperties props;

    @Scheduled(fixedDelayString = "${outbox.poll-interval-ms:1000}")
    public void publishPendingEvents() {
        List<OutboxEvent> pending = outboxRepository
            .findAndLockPendingEvents(props.getBatchSize());

        for (OutboxEvent event : pending) {
            publishEvent(event);
        }
    }

    private void publishEvent(OutboxEvent event) {
        try {
            String topic = resolveTopicFor(event.getAggregateType());

            kafkaTemplate.send(topic, event.getAggregateId(), event.getPayload())
                .get(5, TimeUnit.SECONDS);  // Block to confirm delivery

            event.setProcessedAt(Instant.now());
            outboxRepository.save(event);

        } catch (Exception ex) {
            log.error("Failed to publish outbox event {}: {}", event.getId(), ex.getMessage());
            event.setRetryCount(event.getRetryCount() + 1);
            event.setLastError(ex.getMessage());
            outboxRepository.save(event);
            // processedAt stays NULL — will be retried next poll
        }
    }

    private String resolveTopicFor(String aggregateType) {
        return switch (aggregateType) {
            case "Order"   -> "order-events";
            case "Payment" -> "payment-events";
            default        -> throw new IllegalArgumentException(
                "No topic mapped for aggregate: " + aggregateType
            );
        };
    }
}
```

### Step 5: Preventing Concurrent Processing with `SKIP LOCKED`

When you run multiple instances of your service (as you should in production), multiple publisher threads will race to pick up the same unprocessed events. Use `FOR UPDATE SKIP LOCKED` to prevent that:

```java
// In your repository
@Query(value = """
    SELECT * FROM outbox_events
    WHERE processed_at IS NULL
      AND retry_count < :maxRetries
    ORDER BY created_at ASC
    LIMIT :batchSize
    FOR UPDATE SKIP LOCKED
    """, nativeQuery = true)
@Transactional
List<OutboxEvent> findAndLockPendingEvents(
    @Param("batchSize") int batchSize,
    @Param("maxRetries") int maxRetries
);
```

`FOR UPDATE` locks the rows. `SKIP LOCKED` means other threads skip rows already locked by a competing transaction — no blocking, no deadlocks. This is supported in PostgreSQL, MySQL 8+, and Oracle.

---

## Dealing with At-Least-Once Delivery

The outbox pattern gives you **at-least-once delivery**: if the publisher crashes between sending to Kafka and marking the event as processed, the event will be re-sent on the next poll. This means **your consumers must be idempotent**.

### Option A: Deduplicate at the consumer

```java
@Component
@RequiredArgsConstructor
public class OrderEventConsumer {

    private final ProcessedEventRepository processedRepository;
    private final OrderFulfillmentService fulfillmentService;

    @KafkaListener(topics = "order-events", groupId = "fulfillment-service")
    @Transactional
    public void handle(ConsumerRecord<String, String> record, OrderCreatedEvent event) {
        String eventId = record.headers()
            .lastHeader("event-id").value().toString();

        if (processedRepository.existsByEventId(eventId)) {
            log.debug("Skipping duplicate event: {}", eventId);
            return;
        }

        fulfillmentService.fulfillOrder(event);

        // Record idempotency key in same transaction
        processedRepository.save(new ProcessedEvent(eventId, Instant.now()));
    }
}
```

The idempotency check and the business operation commit together in one transaction.

### Option B: Natural idempotency

Design operations that are safe to apply multiple times. `UPDATE orders SET status='CONFIRMED' WHERE id=? AND status='PENDING'` is naturally idempotent — running it twice has the same effect as running it once.

---

## Ordering Guarantees

The outbox pattern preserves **per-aggregate ordering** if you use the `aggregateId` as the Kafka partition key. For order ID `42`, events like `OrderCreated`, `OrderShipped`, and `OrderDelivered` will always land in the same partition and be consumed in order by a single consumer.

Global ordering across all aggregates is not guaranteed — and in practice, rarely needed.

```java
kafkaTemplate.send(topic, event.getAggregateId(), event.getPayload());
//                        ^^^^^^^^^^^^^^^^^^^^^^^^^^
//                        Use aggregateId as key — same key = same partition
```

---

## Beyond Polling: Change Data Capture (CDC)

The polling approach adds latency (up to your poll interval) and puts extra load on the database. For high-throughput systems, **Change Data Capture (CDC)** is a more efficient alternative.

CDC tools like [Debezium](https://debezium.io/) tail your database's binary transaction log (WAL in PostgreSQL, binlog in MySQL) and stream changes in near real-time — without polling.

```
PostgreSQL WAL ──► Debezium Connector ──► Kafka Topic
                   (reads binary log)
```

**Polling vs. CDC — When to choose what:**

| Concern | Polling | CDC (Debezium) |
|---|---|---|
| Latency | Seconds (configurable) | Milliseconds |
| DB load | Regular SELECT queries | Log tailing (minimal) |
| Operational complexity | Low | Higher (requires Kafka Connect) |
| Schema changes | Transparent | May need reconfiguration |
| Suitable for | Most applications | High-throughput, low-latency needs |

Start with polling. Only reach for CDC when you've profiled and confirmed that polling is a bottleneck.

---

## Dead Letter Handling and Poison Messages

Some events will fail repeatedly — maybe the payload is malformed, or the target service has a bug. Without a dead-letter strategy, these events block your entire outbox queue.

Add a `retry_count` column (shown in the schema above) and implement a dead-letter table:

```java
@Scheduled(fixedDelay = 60_000)  // Run every minute
public void moveStalledEventsToDLQ() {
    List<OutboxEvent> stalled = outboxRepository
        .findByRetryCountGreaterThanAndProcessedAtIsNull(props.getMaxRetries());

    for (OutboxEvent event : stalled) {
        deadLetterRepository.save(DeadLetterEvent.from(event));
        outboxRepository.delete(event);
        log.warn("Moved event {} to DLQ after {} retries", event.getId(), event.getRetryCount());
        // Alert via PagerDuty/Slack/etc.
    }
}
```

Review your dead-letter queue regularly. Each entry is a message that could not be delivered and requires human investigation.

---

## Monitoring: The Metrics That Matter

A growing outbox is a silent killer. Set up alerts for:

```yaml
# Prometheus alert examples
- alert: OutboxBacklogTooLarge
  expr: outbox_pending_events > 1000
  for: 5m
  annotations:
    summary: "Outbox backlog exceeds 1000 events"

- alert: OutboxEventTooOld
  expr: outbox_oldest_pending_age_seconds > 300
  for: 1m
  annotations:
    summary: "Outbox has events older than 5 minutes — publisher may be stuck"

- alert: OutboxDeadLetterGrowing
  expr: increase(outbox_dead_letter_events_total[10m]) > 0
  annotations:
    summary: "New events moved to dead-letter queue"
```

Expose these metrics via Micrometer:

```java
@Component
public class OutboxMetrics {

    private final OutboxEventRepository outboxRepository;
    private final MeterRegistry registry;

    @Scheduled(fixedDelay = 15_000)
    public void recordMetrics() {
        long pending = outboxRepository.countByProcessedAtIsNull();
        registry.gauge("outbox.pending_events", pending);

        outboxRepository.findOldestPendingCreatedAt().ifPresent(oldest -> {
            long ageSeconds = Duration.between(oldest, Instant.now()).toSeconds();
            registry.gauge("outbox.oldest_pending_age_seconds", ageSeconds);
        });
    }
}
```

---

## Cleanup: Don't Let the Table Grow Forever

Processed events accumulate. Schedule a periodic cleanup:

```java
@Scheduled(cron = "0 0 3 * * *")  // 3am daily
@Transactional
public void purgeProcessedEvents() {
    Instant cutoff = Instant.now().minus(Duration.ofDays(7));
    int deleted = outboxRepository.deleteByProcessedAtBefore(cutoff);
    log.info("Purged {} processed outbox events older than 7 days", deleted);
}
```

Alternatively, use a time-partitioned table (PostgreSQL `PARTITION BY RANGE`) and drop old partitions wholesale — far more efficient than row-by-row deletes on a large table.

---

## Summary

The Outbox Pattern solves the dual-write problem by collapsing two system writes into one, then using a relay process to bridge the second system asynchronously.

| Property | Guarantee |
|---|---|
| Atomicity | ✅ Same DB transaction — no partial writes |
| Reliability | ✅ At-least-once delivery with retry |
| Ordering | ✅ Per-aggregate ordering via partition key |
| Exactly-once | ❌ Requires idempotent consumers |
| Latency | ⚠️ Adds poll-interval delay (polling approach) |

**The recommended path:**

1. **Start simple** — polling + `SKIP LOCKED` handles the vast majority of use cases.
2. **Add idempotency** to your consumers from day one.
3. **Monitor** pending event count and age aggressively.
4. **Implement a DLQ** before going to production.
5. **Migrate to CDC (Debezium)** only if you hit performance limits with polling.

The beauty of this pattern is that it solves a hard distributed systems problem using only primitives you already have: a relational database and a scheduled task. No distributed transactions, no sagas, no two-phase commit.

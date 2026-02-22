# Implementing Idempotency in Distributed Spring Boot Applications Using MySQL

## Why Idempotency Breaks in Real Systems 

Modern distributed systems expose APIs that trigger state-changing operations such as payments, orders, the account acquisition process, or account updates. In such environments, the chance of duplicate transactions being initiated is quite high and unavoidable due to network retries, a Kafka rebalancer issuing multiple requests, load balancers, and other factors. Without proper safeguards, these duplicate transactions/requests can lead to data inconsistency, financial discrepancies, and variations in business invariants. 

Idempotency is a well-established technique used to ensure that repeated executions of the same request produce a single, consistent outcome. While idempotency can be enforced at the application level using in-memory caches or request deduplication logic, these approaches would fail for a horizontally scaled microservice architecture, where multiple application instances may process requests concurrently and across numerous different regions.

Relational databases like [MySQL](https://dzone.com/refcardz/essential-mysql) (using the InnoDB storage engine) provide transactional guarantees and row-level locking mechanisms that can be leveraged to implement robust, cross-instance idempotency. By persisting an idempotent key and enforcing exclusive access through pessimistic locking, the system can ensure that only one request is allowed to execute the business logic, while the subsequent duplicate request fails gracefully.

## Problem Statement

### Common Approaches to Idempotency

-   **In memory flags/synchronized blocks** – Duplicates still occur under a multi-instance concurrent environment.
-   **Local cache (Ehcache, Caffeine)** – Duplicates still occur under a multi-instance concurrent environment.
-   **"Just checking if it exists" is unsafe** – Duplicates still occur under a multi-instance concurrent environment.
-   **Unique constraint in the database** – Often results in an exception that must be handled and does not protect against partial execution before failure.
-   **Distributed locks (Redis/Zookeeper)** – Adds operational complexity and introduces new failure modes.

Most of the above implementations are insufficient in distributed systems because they do not coordinate state across application instances and fail under crash recovery or redeployments.

Therefore, the problem addressed in this design is implementing a database-backed idempotency check that uses MySQL row-level locking to identify by an idempotency key, ensuring that exactly-once business execution semantics are preserved across distributed [Spring Boot](https://dzone.com/articles/mastering-scalability-in-spring-boot) application instances.

## **Why MySQL Row-Level Locking Works Well**

[Relational databases](https://dzone.com/articles/the-types-of-modern-databases) already provide strong consistency guarantees through transactions and row-level locking.

By leveraging the below semantics:

-   Select `... FOR UPDATE`
-   Transactional boundaries
-   Unique idempotent key

By using this mechanism, we build a:

-   Strong consistency
-   Safe under concurrency
-   Simple to reason
-   Easy to operate
-   Cloud-Native friendly
-   Relies on the database consistency to handle concurrency

This approach works flawlessly in transaction-sensitive domains like payments, wallets, and account acquisition, and many other use cases.

## High-Level Design

![High-level design](https://dz2cdn1.dzone.com/storage/temp/18799750-1765723611664.png)  

The core idea:

-   Each request carries an idempotent key (like a unique UUID).
-   The application stores the key in an idempotent table.
-   Processing happens inside a single database transaction.
-   The idempotency record is row-locked during processing.
-   Duplicate requests detect the existing key and safely exit.

### A Sample Idempotent Key Design

```sql
CREATE TABLE idempotency_key (
       idem_key      VARCHAR(128) NOT NULL,
       status        ENUM('IN_PROGRESS','COMPLETED','FAILED') NOT NULL,
       request_hash  CHAR(64) NULL,
       response_json JSON NULL,
       created_at    TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
       updated_at    TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
       PRIMARY KEY (idem_key)
) ENGINE=InnoDB;
```

![Idempotency key](https://dz2cdn1.dzone.com/storage/temp/18799753-1765724286689.png)  

How this works:

-   The primary key (`idem_key`) guarantees one row per idempotency key.
-   `PESSIMISTIC_WRITE` becomes `SELECT .. FOR UPDATE` in MySQL (InnoDB), blocking concurrent callers on the same key until commit/rollback.
-   Works across threads and across multiple app instances, because the lock is in MySQL.

Important considerations:

-   Use InnoDB.
-   Keep the lock window small: do only the minimal - check + state transitions while holding the lock.
-   Consider setting `innodb_lock_wait_timeout` behavior; decide whether to return 409/429/422 if the request is already `IN_PROGRESS`.

## Transaction Flow

**Step 1**: Begin transaction.

-   All logic will run inside a single transaction.

**Step 2**: Lock or insert the idempotency record.

-   `SELECT * FROM idempotency_keys WHERE key = ? FOR UPDATE`;
    -   If the record exists and is in the "`COMPLETED`" state, return the stored record.
    -   If the record is `IN_PROGRESS`, block or reject based on the policy.
-   If the record does not exist:
    -   Insert a new record with status "`IN_PROGRESS`."

**Step 3**: Execute business logic.

**Step 4**: Mark the record as completed.

-   Update the idempotency record as "`COMPLETED`" and optionally store the response reference in the table

**Step 5**: Commit transactions.

-   At this point:
    -   Locks are released
    -   Data is consistent
    -   Any concurrent duplicate request is prevented and will resume after seeing the "`COMPLETED`" state (if blocked)

Handling concurrent requests safely:

When two identical requests arrive simultaneously:

-   The first request acquires the row lock
-   The second request blocks/rejects on `SELECT FOR UPDATE`
-   Once the first commits, the second sees the updated states (if blocked)
-   Duplicate execution is prevented.

This guarantees exactly-once behavior semantics at the business level.

## Spring Boot Implementation

Strategy:

-   `@Transactional` at the service layer
-   JPA or JDBC repository
-   Explicit locking queries (`FOR UPDATE`)
-   Clear separation of concerns

Typical components to be implemented:

-   `IdempotencyEntity`
-   `IdempotencyRepository`
-   `IdempotencyService` 
-   Business service invoking idempotency checks

This approach integrates naturally with the existing Spring transaction management.

### Spring Application Properties

```java
spring.application.name=IdempotencyCheck

spring.datasource.url=jdbc:mysql://localhost:3306/product?useSSL=false&serverTimezone=UTC&allowPublicKeyRetrieval=true
spring.datasource.username=root
spring.datasource.password=password
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver

spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true
spring.jpa.open-in-view=false

spring.datasource.hikari.auto-commit=false
spring.datasource.hikari.maximum-pool-size=10
spring.datasource.hikari.minimum-idle=2
spring.datasource.hikari.idle-timeout=30000
spring.datasource.hikari.connection-timeout=20000
spring.datasource.hikari.max-lifetime=1800000
```

Two important properties to consider here:

-   open-in-view avoids “lazy loading during web response” (cleaner for REST)
-   auto-commit = false ensures the pool doesn’t auto-commit behind your back (good for `SELECT ... FOR UPDATE` patterns)

### **JPA Entity**

Contains an enum to hold the current state of the table: `IN_PROGRESS`, `COMPLETED`, or `FAILED`, for the caller to take further actions. For this article, we will throw a conflict exception for simplicity.

```java
package repository;

import jakarta.persistence.*;
import lombok.Getter;
import lombok.Setter;

@Entity
@Table(name = "idempotency_key")
@Getter @Setter
public class IdempotencyKeyEntity {

    @Id
    @Column(name="idem_key", length = 128)
    private String key;

    @Enumerated(EnumType.STRING)
    private Status status;

    @Column(name = "request_hash", length = 64)
    private String requestHash;

    @Column(name = "response_json", columnDefinition = "json")
    private String responseJson;

    public enum Status { IN_PROGRESS, COMPLETED, FAILED }

}
```

### Idempotency Key Repository Implementation

This method retrieves an idempotency record by its key while acquiring a pessimistic write lock on the corresponding database row. The lock ensures that only one transaction can read or modify the record at a time, preventing concurrent requests from processing the same idempotency key simultaneously.

```java
import jakarta.persistence.LockModeType;
import org.springframework.data.jpa.repository.Lock;
import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.query.Param;
import org.springframework.stereotype.Repository;

import java.util.Optional;

@Repository
public interface IdempotencyRepositoryImpl {
    
    @Lock(LockModeType.PESSIMISTIC_WRITE)
    @Query("select idem from IdempotencyKeyEntity idem where idem.key = :key")
    Optional<IdempotencyKeyEntity> lockByKey(@Param("key") String key);
    
}
```

### **Service Implementation**

To get a lock from the database based on the state, if the row already has a completed state, return the response from the hash.

```java
@Transactional
public Optional<String> getCompletedResponse(String key, String requestHash){

    //Find the key or else create a record and insert it into the database and return the entity
    repo.findById(key).orElseGet(() -> {

        IdempotencyKeyEntity entity = new IdempotencyKeyEntity();
        entity.setKey(key);
        entity.setStatus(IdempotencyKeyEntity.Status.IN_PROGRESS);
        entity.setRequestHash(requestHash);
        try{
            return repo.saveAndFlush(entity);
        }catch (DataIntegrityViolationException exception){
            return null;
        }
    });

    //Lock the row only one thread at a time
    IdempotencyKeyEntity locked = repo.lockByKey(key)
            .orElseThrow(() -> new IllegalStateException("Row must exist"));

    //If already completed return the cached response
    if (locked.getStatus() == IdempotencyKeyEntity.Status.COMPLETED) {
        if (!Objects.equals(locked.getRequestHash(), requestHash)) {
            throw new ResponseStatusException(HttpStatus.CONFLICT,
                    "Idempotency-Key reuse with different request");
        }
        return Optional.ofNullable(locked.getResponseJson());
    }

    // If in progress and hash differs conflict (Not blocking here)
    if (locked.getRequestHash() != null &&
            !Objects.equals(locked.getRequestHash(), requestHash)) {
        throw new ResponseStatusException(HttpStatus.CONFLICT,
                "Idempotency-Key reuse with different request");
    }
 
    //Not yet completed, the caller should do the work and mark it completed
    return Optional.empty();
}
```

The code block will mark the record as completed:

```java
@Transactional
public void completed(String key, String responseJson) {
    IdempotencyKeyEntity locked = repo.lockByKey(key)
            .orElseThrow(() -> new IllegalStateException("Row must exist"));
    locked.setStatus(IdempotencyKeyEntity.Status.COMPLETED);
    locked.setResponseJson(responseJson);
    repo.save(locked);
}
```

In case the transaction fails, the other thread, if waiting, can do the work.

```java
@Transactional
public void failed(String key) {
    IdempotencyKeyEntity locked = repo.lockByKey(key)
            .orElseThrow(() -> new IllegalStateException("Row must exist"));
    locked.setStatus(IdempotencyKeyEntity.Status.FAILED);
    repo.save(locked);
}
```

### **Controller Implementation**

```java
@RestController
@RequiredArgsConstructor
public class IdempotentController {

    private final IdempotencyService idempotencyService;

    @PostMapping("/payments")
    public ResponseEntity<String> createPayments(@RequestHeader("Idempotency-Key") String idemKey,
                                                 @RequestBody PaymentRequest req){

        //Using Google Guava for 256 hashing
        String hashReq = Hashing.sha256()
                .hashString(req.toString(), StandardCharsets.UTF_8)
                .toString();

        //check for cachedInDB
        Optional<String> cachedInDB = idempotencyService.getCompletedResponse(idemKey, hashReq);
        if(cachedInDB.isPresent()){
            return ResponseEntity.ok(cachedInDB.get());
        }

        //Do the business logic
        try{
            String results = idempotencyService.doWork();
            //Mark the state as completed for the idempotent key
            idempotencyService.completed(idemKey, results);
            
            return ResponseEntity.ok(results);
        }catch (Exception ex){
            //if the transaction fails, mark the idempotent key as failed to be processed later by other threads
            idempotencyService.failed(idemKey);
            throw ex;
        }
    }
}
```

-   This REST controller demonstrates how idempotent request handling is handled at the API layer using an `Idempotency-Key` header. The controller itself remains minimalistic, delegating all concurrency and state-management controls to the `IdempotentService`.
-   When a request is received, the controller first computes a 256-bit hash using the Google Guava library of the request payload. This hash is used to detect whether the same idempotency key is being reused with a different request body, which is a critical safeguard in transaction-sensitive APIs such as payments.
-   Before executing any business logic, the controller checks whether a completed response already exists for the given idempotency key. If a cached response is found, it is immediately returned to the client, ensuring that duplicate requests do not trigger duplicate side effects.
-   If no completed response exists, the controller proceeds with the business operation. Upon successful execution, the idempotency key is marked as `COMPLETED`, and the response is persisted for safe replay on future retries. In case of failure, the key is marked as `FAILED`, allowing subsequent requests to retry the operation safely.

By isolating idempotency enforcement in the service layer and keeping the controller focused on HTTP requests, this design ensures concurrent requests are handled coherently and retried as needed across distributed Spring Boot instances.

### Use Cases

-   Payment processing
-   Wallet and balance management
-   Account onboarding
-   Order creation
-   Financial workflows requiring strong consistency

Note: It might be overkill for simple read-heavy or eventually consistent workloads

## Performance Considerations

-   Row-level locking is lightweight and scoped to a single key
-   No global locks or distributed coordination required
-   Works well under high concurrency
-   Scales horizontally with the database

For extremely high-throughput systems, partitioning strategies or short-lived transactions can help maintain performance

## **Conclusion**

Idempotency is a foundational requirement for reliable distributed systems. By leveraging MySQL row-level locking and transactional guarantees, Spring Boot applications can safely handle retries, duplicates, and concurrent requests without introducing unnecessary complexity.

This pattern helps in a balance between simplicity, correctness, and operational reliability, making it a strong choice for transaction-sensitive cloud-native applications.

# Test Containers with Spring Boot: From Unit Testing to Integration Testing Excellence

![Header Image](https://miro.medium.com/v2/resize:fit:720/format:webp/1*zzvft4tZgmvV729SGgvlNw.png)  

### Introduction
Throughout my years building enterprise applications, I’ve learned that comprehensive testing is the backbone of reliable software. While unit tests give us confidence in individual components, they often miss the intricate dance between our application and external dependencies like databases, message brokers, and caching systems.

I remember a particularly painful production incident where everything worked perfectly in our unit tests, but the moment we deployed to staging, our Kafka integration fell apart.

The issue? Our mocked Kafka broker couldn’t replicate the exact behavior of the real thing. That’s when I discovered Test Containers, and it completely transformed how I approach integration testing.

In this article, I’ll share how Test Containers has become an indispensable tool in my testing toolkit, enabling me to write integration tests that are fast, reliable, and truly representative of production environments.

We’ll build a real-world application that integrates PostgreSQL, Kafka, and Redis — all tested with Test Containers.

### The Problem with Unit Tests Alone
Don’t get me wrong — unit tests are fantastic. They’re fast, isolated, and provide us with immediate feedback on our business logic. I’ve written thousands of them, and they’ve caught countless bugs before they reached production.

#### Why Unit Tests Aren’t Enough

**1. Mocking Hides Integration Issues** When we mock our database, we’re testing against our assumptions of how the database behaves, not how it actually behaves. Real databases have:
* Transaction isolation levels that can cause race conditions
* Unique constraint violations with specific error codes
* Query performance characteristics
* Connection pooling behaviors

**2. Missing SQL Dialect Specifics** I once wrote a query that worked perfectly with H2 in-memory database, but failed in production PostgreSQL because of different date handling. Unit tests with mocked repositories never caught this.

**3. Message Broker Complexity** Kafka has partitioning, consumer groups, offset management, and serialization. A simple mock can’t capture these complexities.

**4. Caching Behavior** Redis eviction policies, TTL expiration, and data type operations need real testing. Mocks can’t replicate these nuances.

**5. Network Failures and Timeouts** Real systems fail in interesting ways. Connection timeouts, network partitions, and transient failures are hard to simulate with mocks.

### Introducing Test Containers
Test Containers ([https://testcontainers.com/](https://testcontainers.com/)) is a Java library that provides lightweight, throwaway instances of common databases, message brokers, and other services that can run in Docker containers. Think of it as “Docker for testing” — each test gets fresh, isolated infrastructure.

#### What Makes Test Containers Special

**1. Real Dependencies, Not Mocks** * Test against actual PostgreSQL, not H2
* Use real Kafka, not a simple queue
* Run genuine Redis, not an in-memory map

**2. Test Isolation** * Each test suite gets its own containers
* No shared state between tests
* Perfect for parallel test execution

**3. Developer-Friendly** * Automatic container lifecycle management
* Waits for services to be ready
* Cleans up automatically after tests

**4. CI/CD Ready** * Works anywhere Docker runs
* No special infrastructure needed
* Reproducible across environments

**5. Spring Boot Integration** * First-class Spring Test support
* @ServiceConnection annotation magic
* Automatic configuration

#### The Problems Test Containers Solve
* **Problem 1: “Works on My Machine” Syndrome.** Every developer has a different local setup. Test Containers ensures everyone tests against identical infrastructure.
* **Problem 2: Expensive Test Infrastructure.** No need for dedicated test databases or Kafka clusters. Containers spin up and down as needed.
* **Problem 3: Data Pollution** Tests can leave data behind, affecting subsequent tests. Fresh containers mean a clean slate every time.
* **Problem 4: Version Testing.** Want to test against PostgreSQL 15 and 16? Just change the version number.
* **Problem 5: Integration Test Confidence.** If the tests pass, your integration will work in production.

### Building Our Example: E-Commerce Order Service
Let’s build a real-world order processing service that demonstrates Test Containers with PostgreSQL, Kafka, and Redis. This service will:
* Store orders in PostgreSQL
* Publish order events to Kafka
* Cache product prices in Redis
* Handle the complete order lifecycle

#### Project Setup
First, let’s create our Spring Boot project with the necessary dependencies.

**pom.xml**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
         http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.2.0</version>
    </parent>
    
    <groupId>com.example</groupId>
    <artifactId>order-service</artifactId>
    <version>1.0.0</version>
    
    <properties>
        <java.version>21</java.version>
        <testcontainers.version>1.19.3</testcontainers.version>
    </properties>
    
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.kafka</groupId>
            <artifactId>spring-kafka</artifactId>
        </dependency>
        
        <dependency>
            <groupId>org.postgresql</groupId>
            <artifactId>postgresql</artifactId>
            <scope>runtime</scope>
        </dependency>
        
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-testcontainers</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.testcontainers</groupId>
            <artifactId>testcontainers</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.testcontainers</groupId>
            <artifactId>junit-jupiter</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.testcontainers</groupId>
            <artifactId>postgresql</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.testcontainers</groupId>
            <artifactId>kafka</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.testcontainers</groupId>
            <artifactId>redis</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.kafka</groupId>
            <artifactId>spring-kafka-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
    
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.testcontainers</groupId>
                <artifactId>testcontainers-bom</artifactId>
                <version>${testcontainers.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
</project>
```

#### Domain Model
Let’s define our core domain entities.

**Order.java**
```java
@Entity
@Table(name = "orders")
@Data
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class Order {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(nullable = false, unique = true)
    private String orderNumber;
    
    @Column(nullable = false)
    private String customerId;
    
    @Column(nullable = false)
    private BigDecimal totalAmount;
    
    @Enumerated(EnumType.STRING)
    @Column(nullable = false)
    private OrderStatus status;
    
    @OneToMany(cascade = CascadeType.ALL, orphanRemoval = true, mappedBy = "order")
    @Builder.Default
    private List<OrderItem> items = new ArrayList<>();
    
    @Column(nullable = false)
    private LocalDateTime createdAt;
    
    private LocalDateTime updatedAt;
    
    @PrePersist
    protected void onCreate() {
        createdAt = LocalDateTime.now();
        updatedAt = LocalDateTime.now();
    }
    
    @PreUpdate
    protected void onUpdate() {
        updatedAt = LocalDateTime.now();
    }
    
    public void addItem(OrderItem item) {
        items.add(item);
        item.setOrder(this);
    }
}
```

**OrderItem.java**
```java
@Entity
@Table(name = "order_items")
@Data
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class OrderItem {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "order_id", nullable = false)
    @ToString.Exclude
    private Order order;
    
    @Column(nullable = false)
    private String productId;
    
    @Column(nullable = false)
    private String productName;
    
    @Column(nullable = false)
    private Integer quantity;
    
    @Column(nullable = false)
    private BigDecimal unitPrice;
    
    @Column(nullable = false)
    private BigDecimal subtotal;
}
```

**OrderStatus.java**
```java
public enum OrderStatus {
    PENDING,
    CONFIRMED,
    PROCESSING,
    SHIPPED,
    DELIVERED,
    CANCELLED
}
```

#### Repository Layer
**OrderRepository.java**
```java
@Repository
public interface OrderRepository extends JpaRepository<Order, Long> {
    
    Optional<Order> findByOrderNumber(String orderNumber);
    
    List<Order> findByCustomerId(String customerId);
    
    List<Order> findByStatus(OrderStatus status);
    
    @Query("SELECT o FROM Order o WHERE o.createdAt >= :startDate AND o.createdAt <= :endDate")
    List<Order> findOrdersBetweenDates(LocalDateTime startDate, LocalDateTime endDate);
    
    boolean existsByOrderNumber(String orderNumber);
}
```

#### Service Layer with Kafka Publishing
**OrderEvent.java**
```java
@Data
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class OrderEvent {
    private String orderNumber;
    private String customerId;
    private BigDecimal totalAmount;
    private OrderStatus status;
    private LocalDateTime timestamp;
    private String eventType; // CREATED, UPDATED, CANCELLED
}
```

**OrderService.java**
```java
@Service
@RequiredArgsConstructor
@Slf4j
public class OrderService {
    
    private final OrderRepository orderRepository;
    private final KafkaTemplate<String, String> kafkaTemplate;
    private final ObjectMapper objectMapper;
    private final PricingService pricingService;
    
    private static final String ORDER_TOPIC = "orders";
    
    @Transactional
    public Order createOrder(Order order) {
        // Generate order number if not present
        if (order.getOrderNumber() == null) {
            order.setOrderNumber("ORD-" + UUID.randomUUID().toString());
        }
        
        // Set initial status
        order.setStatus(OrderStatus.PENDING);
        
        // Save order
        Order savedOrder = orderRepository.save(order);
        
        // Publish event
        publishOrderEvent(savedOrder, "CREATED");
        
        log.info("Order created: {}", savedOrder.getOrderNumber());
        return savedOrder;
    }
    
    @Transactional
    public Order updateOrderStatus(String orderNumber, OrderStatus newStatus) {
        Order order = orderRepository.findByOrderNumber(orderNumber)
                .orElseThrow(() -> new RuntimeException("Order not found: " + orderNumber));
        
        order.setStatus(newStatus);
        Order updatedOrder = orderRepository.save(order);
        
        // Publish event
        publishOrderEvent(updatedOrder, "UPDATED");
        
        log.info("Order {} status updated to {}", orderNumber, newStatus);
        return updatedOrder;
    }
    
    @Transactional
    public void cancelOrder(String orderNumber) {
        Order order = orderRepository.findByOrderNumber(orderNumber)
                .orElseThrow(() -> new RuntimeException("Order not found: " + orderNumber));
        
        order.setStatus(OrderStatus.CANCELLED);
        orderRepository.save(order);
        
        // Publish event
        publishOrderEvent(order, "CANCELLED");
        
        log.info("Order cancelled: {}", orderNumber);
    }
    
    @Transactional(readOnly = true)
    public Optional<Order> findByOrderNumber(String orderNumber) {
        return orderRepository.findByOrderNumber(orderNumber);
    }
    
    @Transactional(readOnly = true)
    public List<Order> findByCustomerId(String customerId) {
        return orderRepository.findByCustomerId(customerId);
    }
    
    private void publishOrderEvent(Order order, String eventType) {
        try {
            OrderEvent event = OrderEvent.builder()
                    .orderNumber(order.getOrderNumber())
                    .customerId(order.getCustomerId())
                    .totalAmount(order.getTotalAmount())
                    .status(order.getStatus())
                    .timestamp(LocalDateTime.now())
                    .eventType(eventType)
                    .build();
            
            String eventJson = objectMapper.writeValueAsString(event);
            kafkaTemplate.send(ORDER_TOPIC, order.getOrderNumber(), eventJson);
            
            log.info("Published {} event for order: {}", eventType, order.getOrderNumber());
        } catch (JsonProcessingException e) {
            log.error("Error publishing order event", e);
            throw new RuntimeException("Failed to publish order event", e);
        }
    }
}
```

#### Redis Cache for Product Pricing
**PricingService.java**
```java
@Service
@RequiredArgsConstructor
@Slf4j
public class PricingService {
    
    private final RedisTemplate<String, String> redisTemplate;
    private static final String PRICE_PREFIX = "product:price:";
    private static final Duration CACHE_TTL = Duration.ofMinutes(30);
    
    public Optional<BigDecimal> getProductPrice(String productId) {
        String cacheKey = PRICE_PREFIX + productId;
        String cachedPrice = redisTemplate.opsForValue().get(cacheKey);
        
        if (cachedPrice != null) {
            log.info("Price for product {} found in cache", productId);
            return Optional.of(new BigDecimal(cachedPrice));
        }
        
        log.info("Price for product {} not in cache", productId);
        return Optional.empty();
    }
    
    public void cacheProductPrice(String productId, BigDecimal price) {
        String cacheKey = PRICE_PREFIX + productId;
        redisTemplate.opsForValue().set(cacheKey, price.toString(), CACHE_TTL);
        log.info("Cached price for product {}: {}", productId, price);
    }
    
    public void invalidateProductPrice(String productId) {
        String cacheKey = PRICE_PREFIX + productId;
        redisTemplate.delete(cacheKey);
        log.info("Invalidated cache for product {}", productId);
    }
    
    public BigDecimal getOrFetchPrice(String productId) {
        return getProductPrice(productId)
                .orElseGet(() -> {
                    // Simulate fetching from external service
                    BigDecimal price = fetchPriceFromExternalService(productId);
                    cacheProductPrice(productId, price);
                    return price;
                });
    }
    
    private BigDecimal fetchPriceFromExternalService(String productId) {
        // Simulate external service call
        log.info("Fetching price from external service for product {}", productId);
        // In real application, this would call an external pricing service
        return new BigDecimal("99.99");
    }
}
```

#### Configuration
**application.yml**
```yaml
spring:
  application:
    name: order-service
  
  datasource:
    url: jdbc:postgresql://localhost:5432/orderdb
    username: postgres
    password: postgres
    driver-class-name: org.postgresql.Driver
  
  jpa:
    hibernate:
      ddl-auto: update
    show-sql: true
    properties:
      hibernate:
        format_sql: true
        dialect: org.hibernate.dialect.PostgreSQLDialect
  
  kafka:
    bootstrap-servers: localhost:9092
    producer:
      key-serializer: org.apache.kafka.common.serialization.StringSerializer
      value-serializer: org.apache.kafka.common.serialization.StringSerializer
      acks: all
    consumer:
      group-id: order-service-group
      key-deserializer: org.apache.kafka.common.serialization.StringDeserializer
      value-deserializer: org.apache.kafka.common.serialization.StringDeserializer
      auto-offset-reset: earliest
  
  data:
    redis:
      host: localhost
      port: 6379
      timeout: 2000ms

logging:
  level:
    com.example.orderservice: DEBUG
```

### Hands-On: Test Containers Implementation
Now comes the exciting part: let’s write integration tests using Test Containers. These tests will spin up real PostgreSQL, Kafka, and Redis containers.

#### Test Configuration Base Class
**TestContainersConfiguration.java**
```java
@TestConfiguration(proxyBeanMethods = false)
public class TestContainersConfiguration {
    
    @Bean
    @ServiceConnection
    PostgreSQLContainer<?> postgresContainer() {
        return new PostgreSQLContainer<>(DockerImageName.parse("postgres:16-alpine"))
                .withDatabaseName("testdb")
                .withUsername("test")
                .withPassword("test")
                .withReuse(true);
    }
    
    @Bean
    @ServiceConnection
    KafkaContainer kafkaContainer() {
        return new KafkaContainer(DockerImageName.parse("confluentinc/cp-kafka:7.5.0"))
                .withReuse(true);
    }
    
    @Bean
    @ServiceConnection(name = "redis")
    GenericContainer<?> redisContainer() {
        return new GenericContainer<>(DockerImageName.parse("redis:7-alpine"))
                .withExposedPorts(6379)
                .withReuse(true);
    }
}
```

#### PostgreSQL Integration Tests
**OrderRepositoryIntegrationTest.java**
```java
@DataJpaTest
@Import(TestContainersConfiguration.class)
@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)
@ActiveProfiles("test")
class OrderRepositoryIntegrationTest {
    
    @Autowired
    private OrderRepository orderRepository;
    
    @BeforeEach
    void setUp() {
        orderRepository.deleteAll();
    }
    
    @Test
    void shouldSaveAndRetrieveOrder() {
        // Given
        Order order = createTestOrder("ORD-001", "CUST-001", OrderStatus.PENDING);
        
        // When
        Order savedOrder = orderRepository.save(order);
        
        // Then
        assertThat(savedOrder.getId()).isNotNull();
        assertThat(savedOrder.getOrderNumber()).isEqualTo("ORD-001");
        assertThat(savedOrder.getCreatedAt()).isNotNull();
    }
    
    @Test
    void shouldFindOrderByOrderNumber() {
        // Given
        Order order = createTestOrder("ORD-002", "CUST-001", OrderStatus.PENDING);
        orderRepository.save(order);
        
        // When
        Optional<Order> found = orderRepository.findByOrderNumber("ORD-002");
        
        // Then
        assertThat(found).isPresent();
        assertThat(found.get().getOrderNumber()).isEqualTo("ORD-002");
    }
    
    @Test
    void shouldFindOrdersByCustomerId() {
        // Given
        orderRepository.save(createTestOrder("ORD-003", "CUST-001", OrderStatus.PENDING));
        orderRepository.save(createTestOrder("ORD-004", "CUST-001", OrderStatus.CONFIRMED));
        orderRepository.save(createTestOrder("ORD-005", "CUST-002", OrderStatus.PENDING));
        
        // When
        List<Order> customerOrders = orderRepository.findByCustomerId("CUST-001");
        
        // Then
        assertThat(customerOrders).hasSize(2);
        assertThat(customerOrders)
                .extracting(Order::getCustomerId)
                .containsOnly("CUST-001");
    }
    
    @Test
    void shouldFindOrdersByStatus() {
        // Given
        orderRepository.save(createTestOrder("ORD-006", "CUST-001", OrderStatus.PENDING));
        orderRepository.save(createTestOrder("ORD-007", "CUST-002", OrderStatus.PENDING));
        orderRepository.save(createTestOrder("ORD-008", "CUST-003", OrderStatus.CONFIRMED));
        
        // When
        List<Order> pendingOrders = orderRepository.findByStatus(OrderStatus.PENDING);
        
        // Then
        assertThat(pendingOrders).hasSize(2);
        assertThat(pendingOrders)
                .extracting(Order::getStatus)
                .containsOnly(OrderStatus.PENDING);
    }
    
    @Test
    void shouldFindOrdersBetweenDates() {
        // Given
        LocalDateTime now = LocalDateTime.now();
        LocalDateTime yesterday = now.minusDays(1);
        LocalDateTime tomorrow = now.plusDays(1);
        
        orderRepository.save(createTestOrder("ORD-009", "CUST-001", OrderStatus.PENDING));
        
        // When
        List<Order> orders = orderRepository.findOrdersBetweenDates(yesterday, tomorrow);
        
        // Then
        assertThat(orders).hasSize(1);
        assertThat(orders.get(0).getOrderNumber()).isEqualTo("ORD-009");
    }
    
    @Test
    void shouldCheckOrderExistence() {
        // Given
        orderRepository.save(createTestOrder("ORD-010", "CUST-001", OrderStatus.PENDING));
        
        // When & Then
        assertThat(orderRepository.existsByOrderNumber("ORD-010")).isTrue();
        assertThat(orderRepository.existsByOrderNumber("ORD-999")).isFalse();
    }
    
    @Test
    void shouldSaveOrderWithItems() {
        // Given
        Order order = createTestOrder("ORD-011", "CUST-001", OrderStatus.PENDING);
        
        OrderItem item1 = OrderItem.builder()
                .productId("PROD-001")
                .productName("Product 1")
                .quantity(2)
                .unitPrice(new BigDecimal("50.00"))
                .subtotal(new BigDecimal("100.00"))
                .build();
        
        OrderItem item2 = OrderItem.builder()
                .productId("PROD-002")
                .productName("Product 2")
                .quantity(1)
                .unitPrice(new BigDecimal("75.00"))
                .subtotal(new BigDecimal("75.00"))
                .build();
        
        order.addItem(item1);
        order.addItem(item2);
        
        // When
        Order savedOrder = orderRepository.save(order);
        
        // Then
        assertThat(savedOrder.getItems()).hasSize(2);
        assertThat(savedOrder.getItems())
                .extracting(OrderItem::getProductId)
                .containsExactlyInAnyOrder("PROD-001", "PROD-002");
    }
    
    private Order createTestOrder(String orderNumber, String customerId, OrderStatus status) {
        return Order.builder()
                .orderNumber(orderNumber)
                .customerId(customerId)
                .totalAmount(new BigDecimal("100.00"))
                .status(status)
                .build();
    }
}
```

#### Kafka Integration Tests
**OrderServiceKafkaIntegrationTest.java**
```java
@SpringBootTest
@Import(TestContainersConfiguration.class)
@ActiveProfiles("test")
class OrderServiceKafkaIntegrationTest {
    
    @Autowired
    private OrderService orderService;
    
    @Autowired
    private OrderRepository orderRepository;
    
    @Autowired
    private ObjectMapper objectMapper;
    
    @Autowired
    private org.springframework.kafka.core.KafkaTemplate<String, String> kafkaTemplate;
    
    private KafkaMessageListenerContainer<String, String> container;
    private BlockingQueue<ConsumerRecord<String, String>> records;
    
    @BeforeEach
    void setUp() {
        orderRepository.deleteAll();
        records = new LinkedBlockingQueue<>();
        
        Map<String, Object> consumerProps = new HashMap<>();
        consumerProps.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, 
                kafkaTemplate.getProducerFactory().getConfigurationProperties()
                        .get("bootstrap.servers"));
        consumerProps.put(ConsumerConfig.GROUP_ID_CONFIG, "test-group");
        consumerProps.put(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, "earliest");
        consumerProps.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
        consumerProps.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
        
        DefaultKafkaConsumerFactory<String, String> consumerFactory = 
                new DefaultKafkaConsumerFactory<>(consumerProps);
        
        ContainerProperties containerProperties = new ContainerProperties("orders");
        container = new KafkaMessageListenerContainer<>(consumerFactory, containerProperties);
        container.setupMessageListener((MessageListener<String, String>) records::add);
        container.start();
        
        ContainerTestUtils.waitForAssignment(container, 1);
    }
    
    @AfterEach
    void tearDown() {
        if (container != null) {
            container.stop();
        }
    }
    
    @Test
    void shouldPublishOrderCreatedEvent() throws Exception {
        // Given
        Order order = Order.builder()
                .orderNumber("ORD-KAFKA-001")
                .customerId("CUST-001")
                .totalAmount(new BigDecimal("150.00"))
                .build();
        
        // When
        orderService.createOrder(order);
        
        // Then
        ConsumerRecord<String, String> record = records.poll(10, TimeUnit.SECONDS);
        assertThat(record).isNotNull();
        assertThat(record.key()).isEqualTo("ORD-KAFKA-001");
        
        OrderEvent event = objectMapper.readValue(record.value(), OrderEvent.class);
        assertThat(event.getOrderNumber()).isEqualTo("ORD-KAFKA-001");
        assertThat(event.getCustomerId()).isEqualTo("CUST-001");
        assertThat(event.getEventType()).isEqualTo("CREATED");
        assertThat(event.getStatus()).isEqualTo(OrderStatus.PENDING);
    }
    
    @Test
    void shouldPublishOrderUpdatedEvent() throws Exception {
        // Given
        Order order = Order.builder()
                .orderNumber("ORD-KAFKA-002")
                .customerId("CUST-002")
                .totalAmount(new BigDecimal("200.00"))
                .build();
        orderService.createOrder(order);
        
        // Clear the created event
        records.poll(5, TimeUnit.SECONDS);
        
        // When
        orderService.updateOrderStatus("ORD-KAFKA-002", OrderStatus.CONFIRMED);
        
        // Then
        ConsumerRecord<String, String> record = records.poll(10, TimeUnit.SECONDS);
        assertThat(record).isNotNull();
        
        OrderEvent event = objectMapper.readValue(record.value(), OrderEvent.class);
        assertThat(event.getOrderNumber()).isEqualTo("ORD-KAFKA-002");
        assertThat(event.getEventType()).isEqualTo("UPDATED");
        assertThat(event.getStatus()).isEqualTo(OrderStatus.CONFIRMED);
    }
    
    @Test
    void shouldPublishOrderCancelledEvent() throws Exception {
        // Given
        Order order = Order.builder()
                .orderNumber("ORD-KAFKA-003")
                .customerId("CUST-003")
                .totalAmount(new BigDecimal("300.00"))
                .build();
        orderService.createOrder(order);
        
        // Clear the created event
        records.poll(5, TimeUnit.SECONDS);
        
        // When
        orderService.cancelOrder("ORD-KAFKA-003");
        
        // Then
        ConsumerRecord<String, String> record = records.poll(10, TimeUnit.SECONDS);
        assertThat(record).isNotNull();
        
        OrderEvent event = objectMapper.readValue(record.value(), OrderEvent.class);
        assertThat(event.getOrderNumber()).isEqualTo("ORD-KAFKA-003");
        assertThat(event.getEventType()).isEqualTo("CANCELLED");
        assertThat(event.getStatus()).isEqualTo(OrderStatus.CANCELLED);
    }
}
```

#### Redis Integration Tests
**PricingServiceIntegrationTest.java**
```java
@SpringBootTest
@Import(TestContainersConfiguration.class)
@ActiveProfiles("test")
class PricingServiceIntegrationTest {
    
    @Autowired
    private PricingService pricingService;
    
    @Autowired
    private RedisTemplate<String, String> redisTemplate;
    
    @BeforeEach
    void setUp() {
        // Clear all Redis keys before each test
        redisTemplate.getConnectionFactory().getConnection().flushAll();
    }
    
    @Test
    void shouldCacheProductPrice() {
        // Given
        String productId = "PROD-001";
        BigDecimal price = new BigDecimal("99.99");
        
        // When
        pricingService.cacheProductPrice(productId, price);
        
        // Then
        Optional<BigDecimal> cachedPrice = pricingService.getProductPrice(productId);
        assertThat(cachedPrice).isPresent();
        assertThat(cachedPrice.get()).isEqualByComparingTo(price);
    }
    
    @Test
    void shouldReturnEmptyWhenPriceNotCached() {
        // Given
        String productId = "PROD-999";
        
        // When
        Optional<BigDecimal> price = pricingService.getProductPrice(productId);
        
        // Then
        assertThat(price).isEmpty();
    }
    
    @Test
    void shouldInvalidateProductPrice() {
        // Given
        String productId = "PROD-002";
        BigDecimal price = new BigDecimal("149.99");
        pricingService.cacheProductPrice(productId, price);
        
        // When
        pricingService.invalidateProductPrice(productId);
        
        // Then
        Optional<BigDecimal> cachedPrice = pricingService.getProductPrice(productId);
        assertThat(cachedPrice).isEmpty();
    }
    
    @Test
    void shouldFetchAndCachePriceWhenNotInCache() {
        // Given
        String productId = "PROD-003";
        
        // When
        BigDecimal price = pricingService.getOrFetchPrice(productId);
        
        // Then
        assertThat(price).isNotNull();
        
        // Verify it's now cached
        Optional<BigDecimal> cachedPrice = pricingService.getProductPrice(productId);
        assertThat(cachedPrice).isPresent();
        assertThat(cachedPrice.get()).isEqualByComparingTo(price);
    }
    
    @Test
    void shouldUseCachedPriceWhenAvailable() {
        // Given
        String productId = "PROD-004";
        BigDecimal cachedPrice = new BigDecimal("199.99");
        pricingService.cacheProductPrice(productId, cachedPrice);
        
        // When
        BigDecimal price = pricingService.getOrFetchPrice(productId);
        
        // Then
        assertThat(price).isEqualByComparingTo(cachedPrice);
    }
    
    @Test
    void shouldHandleMultipleProducts() {
        // Given
        pricingService.cacheProductPrice("PROD-005", new BigDecimal("50.00"));
        pricingService.cacheProductPrice("PROD-006", new BigDecimal("75.00"));
        pricingService.cacheProductPrice("PROD-007", new BigDecimal("100.00"));
        
        // When & Then
        assertThat(pricingService.getProductPrice("PROD-005"))
                .isPresent()
                .get()
                .isEqualByComparingTo(new BigDecimal("50.00"));
        
        assertThat(pricingService.getProductPrice("PROD-006"))
                .isPresent()
                .get()
                .isEqualByComparingTo(new BigDecimal("75.00"));
        
        assertThat(pricingService.getProductPrice("PROD-007"))
                .isPresent()
                .get()
                .isEqualByComparingTo(new BigDecimal("100.00"));
    }
}
```

#### End-to-End Integration Test
**OrderServiceFullIntegrationTest.java**
```java
@SpringBootTest
@Import(TestContainersConfiguration.class)
@ActiveProfiles("test")
class OrderServiceFullIntegrationTest {
    
    @Autowired
    private OrderService orderService;
    
    @Autowired
    private PricingService pricingService;
    
    @Autowired
    private OrderRepository orderRepository;
    
    @Autowired
    private RedisTemplate<String, String> redisTemplate;
    
    @BeforeEach
    void setUp() {
        orderRepository.deleteAll();
        redisTemplate.getConnectionFactory().getConnection().flushAll();
    }
    
    @Test
    void shouldCreateOrderWithCachedPrices() {
        // Given - Cache product prices
        pricingService.cacheProductPrice("PROD-100", new BigDecimal("50.00"));
        pricingService.cacheProductPrice("PROD-200", new BigDecimal("75.00"));
        
        // When - Create order
        Order order = Order.builder()
                .orderNumber("ORD-FULL-001")
                .customerId("CUST-100")
                .totalAmount(new BigDecimal("175.00"))
                .build();
        
        OrderItem item1 = OrderItem.builder()
                .productId("PROD-100")
                .productName("Product 100")
                .quantity(2)
                .unitPrice(pricingService.getOrFetchPrice("PROD-100"))
                .subtotal(new BigDecimal("100.00"))
                .build();
        
        OrderItem item2 = OrderItem.builder()
                .productId("PROD-200")
                .productName("Product 200")
                .quantity(1)
                .unitPrice(pricingService.getOrFetchPrice("PROD-200"))
                .subtotal(new BigDecimal("75.00"))
                .build();
        
        order.addItem(item1);
        order.addItem(item2);
        
        Order createdOrder = orderService.createOrder(order);
        
        // Then - Verify order in database
        Order foundOrder = orderService.findByOrderNumber("ORD-FULL-001").orElseThrow();
        assertThat(foundOrder.getOrderNumber()).isEqualTo("ORD-FULL-001");
        assertThat(foundOrder.getStatus()).isEqualTo(OrderStatus.PENDING);
        assertThat(foundOrder.getItems()).hasSize(2);
        
        // Verify prices are still cached
        assertThat(pricingService.getProductPrice("PROD-100")).isPresent();
        assertThat(pricingService.getProductPrice("PROD-200")).isPresent();
    }
    
    @Test
    void shouldProcessOrderLifecycle() {
        // Given
        Order order = Order.builder()
                .orderNumber("ORD-FULL-002")
                .customerId("CUST-200")
                .totalAmount(new BigDecimal("250.00"))
                .build();
        
        orderService.createOrder(order);
        
        // When - Update through lifecycle
        orderService.updateOrderStatus("ORD-FULL-002", OrderStatus.CONFIRMED);
        orderService.updateOrderStatus("ORD-FULL-002", OrderStatus.PROCESSING);
        orderService.updateOrderStatus("ORD-FULL-002", OrderStatus.SHIPPED);
        
        // Then
        Order finalOrder = orderService.findByOrderNumber("ORD-FULL-002").orElseThrow();
        assertThat(finalOrder.getStatus()).isEqualTo(OrderStatus.SHIPPED);
    }
    
    @Test
    void shouldFindCustomerOrders() {
        // Given
        Order order1 = Order.builder()
                .orderNumber("ORD-FULL-003")
                .customerId("CUST-300")
                .totalAmount(new BigDecimal("100.00"))
                .build();
        
        Order order2 = Order.builder()
                .orderNumber("ORD-FULL-004")
                .customerId("CUST-300")
                .totalAmount(new BigDecimal("200.00"))
                .build();
        
        orderService.createOrder(order1);
        orderService.createOrder(order2);
        
        // When
        List<Order> customerOrders = orderService.findByCustomerId("CUST-300");
        
        // Then
        assertThat(customerOrders).hasSize(2);
        assertThat(customerOrders)
                .extracting(Order::getOrderNumber)
                .containsExactlyInAnyOrder("ORD-FULL-003", "ORD-FULL-004");
    }
}
```

### Best Practices and Lessons Learned
Through extensive use of Test Containers in production projects, I’ve learned several valuable lessons:

**1. Container Reuse for Speed** Enable container reuse during local development to speed up test execution:
```java
@Bean
PostgreSQLContainer<?> postgresContainer() {
    return new PostgreSQLContainer<>(DockerImageName.parse("postgres:16-alpine"))
            .withReuse(true); // Reuse container across test runs
}
```

**2. Use Alpine Images** Alpine-based images are significantly smaller and start faster:
* `postgres:16-alpine` instead of `postgres:16`
* `redis:7-alpine` instead of `redis:7`

**3. Wait Strategies** Ensure containers are fully ready before tests start:
```java
@Bean
GenericContainer<?> customServiceContainer() {
    return new GenericContainer<>("my-service:latest")
            .withExposedPorts(8080)
            .waitingFor(Wait.forHealthcheck()
                    .withStartupTimeout(Duration.ofMinutes(2)));
}
```

**4. Test Isolation** Always clean up state between tests:
```java
@BeforeEach
void setUp() {
    repository.deleteAll();
    redisTemplate.getConnectionFactory().getConnection().flushAll();
}
```

**5. Parallel Test Execution** Test Containers handles port conflicts automatically, enabling parallel execution:
```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-surefire-plugin</artifactId>
    <configuration>
        <parallel>classes</parallel>
        <threadCount>4</threadCount>
    </configuration>
</plugin>
```

**6. CI/CD Optimization** In CI/CD pipelines, use Docker-in-Docker or ensure Docker is available:
```yaml
# GitHub Actions example
jobs:
  test:
    runs-on: ubuntu-latest
    services:
      docker:
        image: docker:dind
    steps:
      - uses: actions/checkout@v2
      - name: Set up JDK
        uses: actions/setup-java@v2
        with:
          java-version: '21'
      - name: Run tests
        run: mvn test
```

**7. Resource Management** Configure resource limits to prevent overwhelming the host:
```java
@Bean
PostgreSQLContainer<?> postgresContainer() {
    return new PostgreSQLContainer<>(DockerImageName.parse("postgres:16-alpine"))
            .withCreateContainerCmdModifier(cmd -> cmd
                    .withMemory(512 * 1024 * 1024L) // 512MB
                    .withMemorySwap(512 * 1024 * 1024L));
}
```

**8. Network Configuration** When testing microservices together, use custom networks:
```java
Network network = Network.newNetwork();

@Bean
PostgreSQLContainer<?> postgres() {
    return new PostgreSQLContainer<>(DockerImageName.parse("postgres:16-alpine"))
            .withNetwork(network)
            .withNetworkAliases("postgres");
}

@Bean
GenericContainer<?> appContainer() {
    return new GenericContainer<>("my-app:latest")
            .withNetwork(network)
            .withEnv("DB_HOST", "postgres");
}
```

### Real-World Impact
Since adopting Test Containers in my projects, I’ve seen:
* **Reduced Production Bugs:** 40% reduction in database-related production issues
* **Faster Debugging:** When integration tests fail, they catch the exact SQL, Kafka configuration, or Redis behavior that’s problematic
* **Confident Refactoring:** Can refactor data access code without fear, knowing tests verify against real databases
* **Better Onboarding:** New team members can run tests immediately without complex local setup
* **Version Upgrades:** Testing PostgreSQL or Kafka upgrades is as simple as changing a version number

### When to Use Test Containers
Use Test Containers when:
* Testing data access layers with JPA/Hibernate
* Verifying Kafka producer/consumer behavior
* Testing Redis caching strategies
* Integration testing across multiple services
* Validating database migrations
* Testing against specific database versions

Don’t use Test Containers when:
* Pure business logic with no external dependencies (use unit tests)
* Very simple CRUD operations (repository tests might be enough)
* Testing UI components
* Performance/load testing (use dedicated tools)

### Alternatives and Comparisons

**Embedded Databases (H2, HSQLDB)**
* **Pros:** Faster startup, no Docker required 
* **Cons:** Different SQL dialect, doesn’t catch production issues

**Mocked Dependencies**
* **Pros:** Fastest execution 
* **Cons:** Tests assumptions, not reality

**Shared Test Database**
* **Pros:** No container overhead 
* **Cons:** State pollution, serial execution, maintenance burden

**Docker Compose for Tests**
* **Pros:** Full control 
* **Cons:** More setup, manual lifecycle management

**Winner:** Test Containers offers the best balance of realism, speed, and developer experience

### Conclusion
Test Containers has fundamentally changed how I approach integration testing. It bridges the gap between unit tests and production environments, giving us the confidence to ship features faster while maintaining quality.

The combination of real dependencies, automatic lifecycle management, and Spring Boot integration makes Test Containers an essential tool in modern Java development. Whether you’re working with PostgreSQL, Kafka, Redis, or any other infrastructure component, Test Containers provides a reliable, fast, and developer-friendly way to test your integrations.

The investment in setting up Test Containers pays off immediately. The first time it catches a database-specific bug that mocks missed, you’ll wonder how you ever lived without it.
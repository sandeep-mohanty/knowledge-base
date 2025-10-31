# Testing Redis Cluster with Spring Boot Application Using Docker

This comprehensive guide will walk you through setting up a Redis Cluster environment on your development machine using Docker, configuring your Spring Boot application to connect to it, and testing the caching functionality.[1][2]

## Prerequisites

Before starting, ensure you have the following installed on your development machine:

- Docker (version 20.10 or later)
- Docker Compose (version 1.29 or later)
- Java 17 or later
- Maven or Gradle
- A Spring Boot application with Redis dependencies

## Understanding Redis Cluster Architecture

A Redis Cluster provides automatic sharding across multiple Redis nodes with high availability. For development purposes, we'll create:[3]

- **6 Redis nodes** (minimum for production-like cluster)
- **3 Master nodes** (for data distribution)
- **3 Replica nodes** (one replica per master for redundancy)

## Method 1: Using Individual Docker Commands

### Step 1: Create Redis Configuration File

First, create a directory structure for your Redis cluster setup:

```bash
mkdir -p ~/redis-cluster/config
cd ~/redis-cluster
```

Create a Redis configuration file at `~/redis-cluster/config/redis.conf`:

```conf
# Enable cluster mode
cluster-enabled yes

# Cluster configuration file
cluster-config-file nodes.conf

# Cluster node timeout (milliseconds)
cluster-node-timeout 5000

# Enable AOF persistence
appendonly yes

# Disable protected mode for local development
protected-mode no
```

### Step 2: Create Docker Network

Create a dedicated Docker network for Redis cluster nodes to communicate:

```bash
docker network create redis-cluster-net
```

### Step 3: Start Redis Nodes

Launch 6 Redis containers (3 masters + 3 replicas):

```bash
# Node 1
docker run -d \
  --name redis-node-1 \
  --net redis-cluster-net \
  -p 7001:7001 \
  -p 17001:17001 \
  -v ~/redis-cluster/config:/usr/local/etc/redis \
  redis:7.2-alpine \
  redis-server /usr/local/etc/redis/redis.conf \
  --port 7001 \
  --cluster-announce-ip 127.0.0.1 \
  --cluster-announce-port 7001 \
  --cluster-announce-bus-port 17001

# Node 2
docker run -d \
  --name redis-node-2 \
  --net redis-cluster-net \
  -p 7002:7002 \
  -p 17002:17002 \
  -v ~/redis-cluster/config:/usr/local/etc/redis \
  redis:7.2-alpine \
  redis-server /usr/local/etc/redis/redis.conf \
  --port 7002 \
  --cluster-announce-ip 127.0.0.1 \
  --cluster-announce-port 7002 \
  --cluster-announce-bus-port 17002

# Node 3
docker run -d \
  --name redis-node-3 \
  --net redis-cluster-net \
  -p 7003:7003 \
  -p 17003:17003 \
  -v ~/redis-cluster/config:/usr/local/etc/redis \
  redis:7.2-alpine \
  redis-server /usr/local/etc/redis/redis.conf \
  --port 7003 \
  --cluster-announce-ip 127.0.0.1 \
  --cluster-announce-port 7003 \
  --cluster-announce-bus-port 17003

# Node 4
docker run -d \
  --name redis-node-4 \
  --net redis-cluster-net \
  -p 7004:7004 \
  -p 17004:17004 \
  -v ~/redis-cluster/config:/usr/local/etc/redis \
  redis:7.2-alpine \
  redis-server /usr/local/etc/redis/redis.conf \
  --port 7004 \
  --cluster-announce-ip 127.0.0.1 \
  --cluster-announce-port 7004 \
  --cluster-announce-bus-port 17004

# Node 5
docker run -d \
  --name redis-node-5 \
  --net redis-cluster-net \
  -p 7005:7005 \
  -p 17005:17005 \
  -v ~/redis-cluster/config:/usr/local/etc/redis \
  redis:7.2-alpine \
  redis-server /usr/local/etc/redis/redis.conf \
  --port 7005 \
  --cluster-announce-ip 127.0.0.1 \
  --cluster-announce-port 7005 \
  --cluster-announce-bus-port 17005

# Node 6
docker run -d \
  --name redis-node-6 \
  --net redis-cluster-net \
  -p 7006:7006 \
  -p 17006:17006 \
  -v ~/redis-cluster/config:/usr/local/etc/redis \
  redis:7.2-alpine \
  redis-server /usr/local/etc/redis/redis.conf \
  --port 7006 \
  --cluster-announce-ip 127.0.0.1 \
  --cluster-announce-port 7006 \
  --cluster-announce-bus-port 17006
```

**Note:** Each node requires two ports - one for client connections and one for cluster bus communication.[1]

### Step 4: Create the Cluster

After all nodes are running, create the cluster configuration:

```bash
docker exec -it redis-node-1 redis-cli --cluster create \
  127.0.0.1:7001 \
  127.0.0.1:7002 \
  127.0.0.1:7003 \
  127.0.0.1:7004 \
  127.0.0.1:7005 \
  127.0.0.1:7006 \
  --cluster-replicas 1 \
  --cluster-yes
```

This command creates 3 masters and 3 replicas (1 replica per master).[1]

### Step 5: Verify Cluster Status

Check the cluster is properly configured:

```bash
# Connect to any node and check cluster info
docker exec -it redis-node-1 redis-cli -c -p 7001 cluster info

# Check cluster nodes
docker exec -it redis-node-1 redis-cli -c -p 7001 cluster nodes
```

Expected output should show:

```
cluster_state:ok
cluster_slots_assigned:16384
cluster_known_nodes:6
```

### Step 6: Test Cluster Connectivity

Test the cluster from your host machine:

```bash
# Install redis-cli if not already installed (macOS)
brew install redis

# Or for Linux
sudo apt-get install redis-tools

# Test connection
redis-cli -c -p 7001 ping
# Expected output: PONG

# Test cluster redirection
redis-cli -c -p 7001
> set mykey "Hello Redis Cluster"
> get mykey
```

### Step 7: Managing Containers

Useful commands for managing your Redis cluster:

```bash
# View running containers
docker ps | grep redis-node

# Stop all nodes
docker stop redis-node-1 redis-node-2 redis-node-3 redis-node-4 redis-node-5 redis-node-6

# Start all nodes
docker start redis-node-1 redis-node-2 redis-node-3 redis-node-4 redis-node-5 redis-node-6

# Remove all nodes
docker rm -f redis-node-1 redis-node-2 redis-node-3 redis-node-4 redis-node-5 redis-node-6

# Remove network
docker network rm redis-cluster-net
```

## Method 2: Using Docker Compose

Docker Compose provides a cleaner, more maintainable approach for managing multi-container applications.[4][1]

### Step 1: Create Project Directory

```bash
mkdir -p ~/redis-cluster-compose/redis
cd ~/redis-cluster-compose
```

### Step 2: Create Redis Configuration File

Create `redis/redis.conf`:

```conf
# Enable cluster mode
cluster-enabled yes

# Cluster configuration file
cluster-config-file nodes.conf

# Cluster node timeout (milliseconds)
cluster-node-timeout 5000

# Enable AOF persistence for data durability
appendonly yes

# Disable protected mode for local development
protected-mode no

# Enable cluster replica validity
cluster-require-full-coverage no
```

### Step 3: Create Docker Compose File

Create `docker-compose.yml`:

```yaml
version: '3.8'

networks:
  redis-cluster-network:
    driver: bridge

services:
  redis-node-1:
    image: redis:7.2-alpine
    container_name: redis-node-1
    command: redis-server /usr/local/etc/redis/redis.conf --port 7001 --cluster-announce-ip 127.0.0.1
    ports:
      - "7001:7001"
      - "17001:17001"
    volumes:
      - redis-data-1:/data
      - ./redis:/usr/local/etc/redis
    networks:
      - redis-cluster-network
    healthcheck:
      test: ["CMD", "redis-cli", "-p", "7001", "ping"]
      interval: 10s
      timeout: 5s
      retries: 3
      start_period: 30s

  redis-node-2:
    image: redis:7.2-alpine
    container_name: redis-node-2
    command: redis-server /usr/local/etc/redis/redis.conf --port 7002 --cluster-announce-ip 127.0.0.1
    ports:
      - "7002:7002"
      - "17002:17002"
    volumes:
      - redis-data-2:/data
      - ./redis:/usr/local/etc/redis
    networks:
      - redis-cluster-network
    healthcheck:
      test: ["CMD", "redis-cli", "-p", "7002", "ping"]
      interval: 10s
      timeout: 5s
      retries: 3
      start_period: 30s

  redis-node-3:
    image: redis:7.2-alpine
    container_name: redis-node-3
    command: redis-server /usr/local/etc/redis/redis.conf --port 7003 --cluster-announce-ip 127.0.0.1
    ports:
      - "7003:7003"
      - "17003:17003"
    volumes:
      - redis-data-3:/data
      - ./redis:/usr/local/etc/redis
    networks:
      - redis-cluster-network
    healthcheck:
      test: ["CMD", "redis-cli", "-p", "7003", "ping"]
      interval: 10s
      timeout: 5s
      retries: 3
      start_period: 30s

  redis-node-4:
    image: redis:7.2-alpine
    container_name: redis-node-4
    command: redis-server /usr/local/etc/redis/redis.conf --port 7004 --cluster-announce-ip 127.0.0.1
    ports:
      - "7004:7004"
      - "17004:17004"
    volumes:
      - redis-data-4:/data
      - ./redis:/usr/local/etc/redis
    networks:
      - redis-cluster-network
    healthcheck:
      test: ["CMD", "redis-cli", "-p", "7004", "ping"]
      interval: 10s
      timeout: 5s
      retries: 3
      start_period: 30s

  redis-node-5:
    image: redis:7.2-alpine
    container_name: redis-node-5
    command: redis-server /usr/local/etc/redis/redis.conf --port 7005 --cluster-announce-ip 127.0.0.1
    ports:
      - "7005:7005"
      - "17005:17005"
    volumes:
      - redis-data-5:/data
      - ./redis:/usr/local/etc/redis
    networks:
      - redis-cluster-network
    healthcheck:
      test: ["CMD", "redis-cli", "-p", "7005", "ping"]
      interval: 10s
      timeout: 5s
      retries: 3
      start_period: 30s

  redis-node-6:
    image: redis:7.2-alpine
    container_name: redis-node-6
    command: redis-server /usr/local/etc/redis/redis.conf --port 7006 --cluster-announce-ip 127.0.0.1
    ports:
      - "7006:7006"
      - "17006:17006"
    volumes:
      - redis-data-6:/data
      - ./redis:/usr/local/etc/redis
    networks:
      - redis-cluster-network
    healthcheck:
      test: ["CMD", "redis-cli", "-p", "7006", "ping"]
      interval: 10s
      timeout: 5s
      retries: 3
      start_period: 30s

  redis-cluster-creator:
    image: redis:7.2-alpine
    container_name: redis-cluster-creator
    command: >
      sh -c "sleep 10 &&
             redis-cli --cluster create
             127.0.0.1:7001
             127.0.0.1:7002
             127.0.0.1:7003
             127.0.0.1:7004
             127.0.0.1:7005
             127.0.0.1:7006
             --cluster-replicas 1
             --cluster-yes"
    network_mode: host
    depends_on:
      redis-node-1:
        condition: service_healthy
      redis-node-2:
        condition: service_healthy
      redis-node-3:
        condition: service_healthy
      redis-node-4:
        condition: service_healthy
      redis-node-5:
        condition: service_healthy
      redis-node-6:
        condition: service_healthy

 # RedisInsight - GUI for Redis
  redisinsight:
    image: redis/redisinsight:latest
    container_name: redisinsight
    restart: unless-stopped
    ports:
      - "5540:5540"
    volumes:
      - redisinsight-data:/data
    networks:
      - redis-cluster-network
    environment:
      - REDISINSIGHT_HOST=0.0.0.0
      - REDISINSIGHT_PORT=5540
    depends_on:
      redis-node-1:
        condition: service_healthy
      redis-node-2:
        condition: service_healthy
      redis-node-3:
        condition: service_healthy
      redis-node-4:
        condition: service_healthy
      redis-node-5:
        condition: service_healthy
      redis-node-6:
        condition: service_healthy
    healthcheck:
      test: ["CMD", "wget", "--quiet", "--tries=1", "--spider", "http://localhost:5540/api/health/"]
      interval: 10s
      timeout: 5s
      retries: 3
      start_period: 30s

volumes:
  redis-data-1:
  redis-data-2:
  redis-data-3:
  redis-data-4:
  redis-data-5:
  redis-data-6:
  redisinsight-data:
```

**Key features of this configuration:**

- **Health checks**: Each node has a health check to ensure it's ready before cluster creation[5][6]
- **Persistent volumes**: Data survives container restarts
- **Network isolation**: All nodes communicate through a dedicated bridge network
- **Automatic cluster creation**: The cluster-creator service automatically initializes the cluster once all nodes are healthy[1]

### Step 4: Start the Cluster

```bash
# Start all services
docker-compose up -d

# View logs to monitor cluster creation
docker-compose logs -f redis-cluster-creator

# Check all services are healthy
docker-compose ps
```

### Step 5: Verify Cluster Status

```bash
# Check cluster info
docker exec -it redis-node-1 redis-cli -c -p 7001 cluster info

# View cluster nodes and slot distribution
docker exec -it redis-node-1 redis-cli -c -p 7001 cluster nodes

# Check health of specific container
docker inspect --format='{{json .State.Health}}' redis-node-1 | jq
```

### Step 6: Test Cluster Operations

```bash
# Interactive cluster testing
docker exec -it redis-node-1 redis-cli -c -p 7001

# Inside redis-cli:
> set user:1001 "John Doe"
> get user:1001
> set user:2002 "Jane Smith"
> keys user:*
> cluster keyslot user:1001
> exit
```

### Step 7: Managing Docker Compose Cluster

```bash
# Stop all services
docker-compose stop

# Start all services
docker-compose start

# Restart all services
docker-compose restart

# Stop and remove all services
docker-compose down

# Stop and remove all services including volumes
docker-compose down -v

# View logs for specific service
docker-compose logs -f redis-node-1

# Scale is not applicable for cluster nodes as each needs unique configuration
```

## Spring Boot Application Configuration

### Step 1: Add Dependencies

Add the required dependencies to your `pom.xml`:

```xml
<dependencies>
    <!-- Spring Boot Starter for Web -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <!-- Spring Boot Starter for Cache -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-cache</artifactId>
    </dependency>

    <!-- Spring Data Redis with Lettuce (default) -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-redis</artifactId>
    </dependency>

    <!-- Optional: Apache Commons Pool for connection pooling -->
    <dependency>
        <groupId>org.apache.commons</groupId>
        <artifactId>commons-pool2</artifactId>
    </dependency>

    <!-- For testing -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
```

### Step 2: Configure Application Properties

Create or update `src/main/resources/application.yml`:

```yaml
spring:
  application:
    name: redis-cluster-demo
  
  # Redis Cluster Configuration
  data:
    redis:
      cluster:
        nodes:
          - 127.0.0.1:7001
          - 127.0.0.1:7002
          - 127.0.0.1:7003
          - 127.0.0.1:7004
          - 127.0.0.1:7005
          - 127.0.0.1:7006
        max-redirects: 3
      
      # Connection timeout
      timeout: 2000ms
      
      # Lettuce pool configuration
      lettuce:
        pool:
          max-active: 8
          max-idle: 8
          min-idle: 0
          max-wait: -1ms
        cluster:
          refresh:
            adaptive: true
            period: 60s
  
  # Cache Configuration
  cache:
    type: redis
    redis:
      time-to-live: 600000  # 10 minutes in milliseconds
      cache-null-values: false
      use-key-prefix: true
      key-prefix: "app:cache:"

# Logging for debugging
logging:
  level:
    io.lettuce.core: DEBUG
    org.springframework.data.redis: DEBUG
```

**Alternative `application.properties` format:**

```properties
spring.application.name=redis-cluster-demo

# Redis Cluster nodes
spring.data.redis.cluster.nodes=127.0.0.1:7001,127.0.0.1:7002,127.0.0.1:7003,127.0.0.1:7004,127.0.0.1:7005,127.0.0.1:7006
spring.data.redis.cluster.max-redirects=3

# Connection settings
spring.data.redis.timeout=2000ms

# Lettuce pool configuration
spring.data.redis.lettuce.pool.max-active=8
spring.data.redis.lettuce.pool.max-idle=8
spring.data.redis.lettuce.pool.min-idle=0
spring.data.redis.lettuce.pool.max-wait=-1ms
spring.data.redis.lettuce.cluster.refresh.adaptive=true
spring.data.redis.lettuce.cluster.refresh.period=60s

# Cache configuration
spring.cache.type=redis
spring.cache.redis.time-to-live=600000
spring.cache.redis.cache-null-values=false
spring.cache.redis.use-key-prefix=true
spring.cache.redis.key-prefix=app:cache:

# Debugging
logging.level.io.lettuce.core=DEBUG
logging.level.org.springframework.data.redis=DEBUG
```

### Step 3: Create Redis Configuration Class

Create `src/main/java/com/example/config/RedisConfig.java`:

```java
package com.example.config;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.cache.annotation.EnableCaching;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.cache.RedisCacheConfiguration;
import org.springframework.data.redis.cache.RedisCacheManager;
import org.springframework.data.redis.connection.RedisClusterConfiguration;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.connection.lettuce.LettuceConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.serializer.GenericJackson2JsonRedisSerializer;
import org.springframework.data.redis.serializer.RedisSerializationContext;
import org.springframework.data.redis.serializer.StringRedisSerializer;

import java.time.Duration;
import java.util.List;

@Configuration
@EnableCaching
public class RedisConfig {

    @Value("${spring.data.redis.cluster.nodes}")
    private List<String> clusterNodes;

    @Bean
    public RedisConnectionFactory redisConnectionFactory() {
        RedisClusterConfiguration clusterConfiguration = 
            new RedisClusterConfiguration(clusterNodes);
        
        // Optional: Set max redirects
        clusterConfiguration.setMaxRedirects(3);
        
        return new LettuceConnectionFactory(clusterConfiguration);
    }

    @Bean
    public RedisTemplate<String, Object> redisTemplate(
            RedisConnectionFactory connectionFactory) {
        RedisTemplate<String, Object> template = new RedisTemplate<>();
        template.setConnectionFactory(connectionFactory);
        
        // Use String serialization for keys
        template.setKeySerializer(new StringRedisSerializer());
        template.setHashKeySerializer(new StringRedisSerializer());
        
        // Use JSON serialization for values
        GenericJackson2JsonRedisSerializer serializer = 
            new GenericJackson2JsonRedisSerializer();
        template.setValueSerializer(serializer);
        template.setHashValueSerializer(serializer);
        
        template.afterPropertiesSet();
        return template;
    }

    @Bean
    public RedisCacheManager cacheManager(
            RedisConnectionFactory connectionFactory) {
        RedisCacheConfiguration config = RedisCacheConfiguration
            .defaultCacheConfig()
            .entryTtl(Duration.ofMinutes(10))
            .serializeKeysWith(
                RedisSerializationContext.SerializationPair
                    .fromSerializer(new StringRedisSerializer()))
            .serializeValuesWith(
                RedisSerializationContext.SerializationPair
                    .fromSerializer(new GenericJackson2JsonRedisSerializer()))
            .disableCachingNullValues();

        return RedisCacheManager.builder(connectionFactory)
            .cacheDefaults(config)
            .build();
    }
}
```

### Step 4: Create a Sample Entity

Create `src/main/java/com/example/model/User.java`:

```java
package com.example.model;

import java.io.Serializable;

public class User implements Serializable {
    private static final long serialVersionUID = 1L;
    
    private Long id;
    private String name;
    private String email;

    // Constructors
    public User() {}

    public User(Long id, String name, String email) {
        this.id = id;
        this.name = name;
        this.email = email;
    }

    // Getters and Setters
    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getEmail() {
        return email;
    }

    public void setEmail(String email) {
        this.email = email;
    }

    @Override
    public String toString() {
        return "User{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", email='" + email + '\'' +
                '}';
    }
}
```

### Step 5: Create a Service with Caching

Create `src/main/java/com/example/service/UserService.java`:

```java
package com.example.service;

import com.example.model.User;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.cache.annotation.CacheEvict;
import org.springframework.cache.annotation.CachePut;
import org.springframework.cache.annotation.Cacheable;
import org.springframework.stereotype.Service;

import java.util.HashMap;
import java.util.Map;
import java.util.Optional;

@Service
public class UserService {
    
    private static final Logger logger = 
        LoggerFactory.getLogger(UserService.class);
    
    // Simulated database
    private final Map<Long, User> userDatabase = new HashMap<>();

    public UserService() {
        // Initialize with sample data
        userDatabase.put(1L, new User(1L, "John Doe", "john@example.com"));
        userDatabase.put(2L, new User(2L, "Jane Smith", "jane@example.com"));
        userDatabase.put(3L, new User(3L, "Bob Johnson", "bob@example.com"));
    }

    @Cacheable(value = "users", key = "#id")
    public Optional<User> getUserById(Long id) {
        logger.info("Fetching user from database for id: {}", id);
        // Simulate slow database query
        simulateSlowService();
        return Optional.ofNullable(userDatabase.get(id));
    }

    @CachePut(value = "users", key = "#user.id")
    public User saveUser(User user) {
        logger.info("Saving user to database: {}", user);
        userDatabase.put(user.getId(), user);
        return user;
    }

    @CacheEvict(value = "users", key = "#id")
    public void deleteUser(Long id) {
        logger.info("Deleting user from database with id: {}", id);
        userDatabase.remove(id);
    }

    @CacheEvict(value = "users", allEntries = true)
    public void clearCache() {
        logger.info("Clearing all user cache");
    }

    private void simulateSlowService() {
        try {
            Thread.sleep(3000); // 3 second delay
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }
}
```

### Step 6: Create a REST Controller

Create `src/main/java/com/example/controller/UserController.java`:

```java
package com.example.controller;

import com.example.model.User;
import com.example.service.UserService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api/users")
public class UserController {

    private final UserService userService;

    @Autowired
    public UserController(UserService userService) {
        this.userService = userService;
    }

    @GetMapping("/{id}")
    public ResponseEntity<User> getUser(@PathVariable Long id) {
        return userService.getUserById(id)
            .map(ResponseEntity::ok)
            .orElse(ResponseEntity.notFound().build());
    }

    @PostMapping
    public ResponseEntity<User> createUser(@RequestBody User user) {
        User savedUser = userService.saveUser(user);
        return ResponseEntity.ok(savedUser);
    }

    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteUser(@PathVariable Long id) {
        userService.deleteUser(id);
        return ResponseEntity.noContent().build();
    }

    @DeleteMapping("/cache/clear")
    public ResponseEntity<String> clearCache() {
        userService.clearCache();
        return ResponseEntity.ok("Cache cleared successfully");
    }
}
```

### Step 7: Create Cluster Health Endpoint

Create `src/main/java/com/example/controller/RedisHealthController.java`:

```java
package com.example.controller;

import io.lettuce.core.cluster.models.partitions.RedisClusterNode;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.redis.connection.RedisClusterConnection;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.HashMap;
import java.util.Map;
import java.util.Properties;

@RestController
@RequestMapping("/api/redis")
public class RedisHealthController {

    @Autowired
    private RedisTemplate<String, Object> redisTemplate;

    @Autowired
    private RedisConnectionFactory connectionFactory;

    @GetMapping("/health")
    public Map<String, Object> checkHealth() {
        Map<String, Object> health = new HashMap<>();
        
        try {
            // Test connection
            String pong = redisTemplate.getConnectionFactory()
                .getConnection()
                .ping();
            health.put("status", "UP");
            health.put("ping", pong);
            
            // Get cluster info
            RedisClusterConnection clusterConnection = 
                connectionFactory.getClusterConnection();
            
            Properties clusterInfo = clusterConnection.clusterGetClusterInfo();
            health.put("clusterState", clusterInfo.getProperty("cluster_state"));
            health.put("clusterSlotsAssigned", 
                clusterInfo.getProperty("cluster_slots_assigned"));
            health.put("clusterKnownNodes", 
                clusterInfo.getProperty("cluster_known_nodes"));
            
            clusterConnection.close();
        } catch (Exception e) {
            health.put("status", "DOWN");
            health.put("error", e.getMessage());
        }
        
        return health;
    }

    @GetMapping("/cluster/nodes")
    public Map<String, Object> getClusterNodes() {
        Map<String, Object> response = new HashMap<>();
        
        try {
            RedisClusterConnection clusterConnection = 
                connectionFactory.getClusterConnection();
            
            Iterable<RedisClusterNode> nodes = clusterConnection.clusterGetNodes();
            int masterCount = 0;
            int replicaCount = 0;
            
            for (RedisClusterNode node : nodes) {
                if (node.isMaster()) {
                    masterCount++;
                } else {
                    replicaCount++;
                }
            }
            
            response.put("totalNodes", masterCount + replicaCount);
            response.put("masterNodes", masterCount);
            response.put("replicaNodes", replicaCount);
            
            clusterConnection.close();
        } catch (Exception e) {
            response.put("error", e.getMessage());
        }
        
        return response;
    }

    @GetMapping("/test")
    public Map<String, Object> testOperations() {
        Map<String, Object> result = new HashMap<>();
        
        try {
            // Test SET operation
            String key = "test:key:" + System.currentTimeMillis();
            String value = "test-value";
            redisTemplate.opsForValue().set(key, value);
            result.put("setOperation", "SUCCESS");
            
            // Test GET operation
            Object retrievedValue = redisTemplate.opsForValue().get(key);
            result.put("getOperation", "SUCCESS");
            result.put("retrievedValue", retrievedValue);
            
            // Test DELETE operation
            Boolean deleted = redisTemplate.delete(key);
            result.put("deleteOperation", deleted ? "SUCCESS" : "FAILED");
            
        } catch (Exception e) {
            result.put("error", e.getMessage());
        }
        
        return result;
    }
}
```

## Testing the Redis Cluster Setup

### Test 1: Verify Cluster Health

```bash
# Test cluster health via Spring Boot endpoint
curl http://localhost:8080/api/redis/health

# Expected response:
# {
#   "status": "UP",
#   "ping": "PONG",
#   "clusterState": "ok",
#   "clusterSlotsAssigned": "16384",
#   "clusterKnownNodes": "6"
# }
```

### Test 2: Verify Cluster Nodes

```bash
# Check cluster nodes
curl http://localhost:8080/api/redis/cluster/nodes

# Expected response:
# {
#   "totalNodes": 6,
#   "masterNodes": 3,
#   "replicaNodes": 3
# }
```

### Test 3: Test Basic Redis Operations

```bash
# Test Redis operations
curl http://localhost:8080/api/redis/test

# Expected response shows successful SET, GET, and DELETE operations
```

### Test 4: Test Caching Functionality

```bash
# First request - should take ~3 seconds (database query)
time curl http://localhost:8080/api/users/1

# Second request - should be instant (cached)
time curl http://localhost:8080/api/users/1

# Create new user
curl -X POST http://localhost:8080/api/users \
  -H "Content-Type: application/json" \
  -d '{"id":4,"name":"Alice Cooper","email":"alice@example.com"}'

# Fetch the new user (should be cached immediately)
curl http://localhost:8080/api/users/4
```

### Test 5: Verify Cache in Redis

```bash
# Connect to Redis cluster
redis-cli -c -p 7001

# Check cached keys
127.0.0.1:7001> keys app:cache:*

# Get specific cached user
127.0.0.1:7001> get "app:cache:users::1"

# Check key distribution across cluster
127.0.0.1:7001> cluster keyslot "app:cache:users::1"
127.0.0.1:7001> cluster nodes
```

### Test 6: Test Cache Eviction

```bash
# Delete user (should clear from cache)
curl -X DELETE http://localhost:8080/api/users/1

# Try to fetch deleted user - should return 404 after checking database
curl http://localhost:8080/api/users/1

# Clear all cache
curl -X DELETE http://localhost:8080/api/users/cache/clear
```

### Test 7: Test Cluster Failover

This test simulates a node failure to verify cluster resilience:

```bash
# Stop one of the master nodes
docker stop redis-node-1

# Application should still work - test an endpoint
curl http://localhost:8080/api/users/2

# Check cluster status (from another node)
docker exec -it redis-node-2 redis-cli -c -p 7002 cluster nodes

# Restart the node
docker start redis-node-1

# Verify it rejoins the cluster
docker exec -it redis-node-1 redis-cli -c -p 7001 cluster nodes
```

### Test 8: Monitor Redis Cluster Performance

Create a load testing script `test-load.sh`:

```bash
#!/bin/bash

echo "Running load test on Redis Cluster..."

for i in {1..100}; do
  USER_ID=$((RANDOM % 10 + 1))
  curl -s "http://localhost:8080/api/users/$USER_ID" > /dev/null
  echo "Request $i completed for user $USER_ID"
done

echo "Load test completed"
```

Make it executable and run:

```bash
chmod +x test-load.sh
./test-load.sh
```

### Test 9: Monitor Redis Cluster Metrics

Connect to Redis and monitor operations in real-time:

```bash
# Monitor all commands
docker exec -it redis-node-1 redis-cli -c -p 7001 monitor

# Get cluster statistics
docker exec -it redis-node-1 redis-cli -c -p 7001 info stats

# Check memory usage
docker exec -it redis-node-1 redis-cli -c -p 7001 info memory

# Check replication status
docker exec -it redis-node-1 redis-cli -c -p 7001 info replication
```

## Troubleshooting Common Issues

### Issue 1: Cluster Creation Fails

**Symptoms:** Cluster creation command hangs or fails

**Solutions:**

```bash
# Check all nodes are running
docker ps | grep redis-node

# Check logs for errors
docker logs redis-node-1

# Verify network connectivity
docker network inspect redis-cluster-network

# Ensure no conflicting Redis instances
lsof -i :7001-7006

# Reset and recreate
docker-compose down -v
docker-compose up -d
```

### Issue 2: Connection Refused Errors

**Symptoms:** Spring Boot cannot connect to Redis cluster

**Solutions:**

```yaml
# Verify application.yml has correct node addresses
spring:
  data:
    redis:
      cluster:
        nodes:
          - localhost:7001  # Not 127.0.0.1 if using Docker Desktop on Mac/Windows
          - localhost:7002
          # ... etc
```

```bash
# Test direct connection
redis-cli -c -h localhost -p 7001 ping

# Check firewall rules
sudo ufw status  # Linux
# Ensure ports 7001-7006 and 17001-17006 are open
```

### Issue 3: MOVED/ASK Redirection Errors

**Symptoms:** Application logs show MOVED or ASK responses

**Solution:** This is normal for Redis Cluster. Ensure your client configuration has cluster support enabled:

```java
// In RedisConfig.java
clusterConfiguration.setMaxRedirects(3);  // Allow up to 3 redirects
```

### Issue 4: Cache Not Working

**Symptoms:** Every request hits the database (no caching)

**Checklist:**

```java
// 1. Ensure @EnableCaching is present
@Configuration
@EnableCaching  // Must be present
public class RedisConfig { ... }

// 2. Verify @Cacheable annotation
@Cacheable(value = "users", key = "#id")
public Optional<User> getUserById(Long id) { ... }

// 3. Check serialization
// Ensure entity implements Serializable
public class User implements Serializable { ... }
```

```bash
# Verify cache entries in Redis
redis-cli -c -p 7001
> keys *
> ttl app:cache:users::1
```

### Issue 5: Memory Issues

**Symptoms:** Redis runs out of memory

**Solutions:**

```conf
# Add to redis.conf
maxmemory 256mb
maxmemory-policy allkeys-lru
```

```bash
# Restart containers with updated config
docker-compose restart
```

## Performance Optimization Tips

### 1. Connection Pooling

Optimize Lettuce connection pool settings:

```yaml
spring:
  data:
    redis:
      lettuce:
        pool:
          max-active: 20      # Increase for high-traffic apps
          max-idle: 10
          min-idle: 5
          max-wait: 2000ms    # Wait time for connection
```

### 2. Topology Refresh

Enable adaptive topology refresh for better failover:

```yaml
spring:
  data:
    redis:
      lettuce:
        cluster:
          refresh:
            adaptive: true    # Auto-refresh on topology changes
            period: 60s       # Periodic refresh interval
```

### 3. Cache TTL Strategy

Implement different TTL for different cache types:

```java
@Bean
public RedisCacheManager cacheManager(RedisConnectionFactory factory) {
    Map<String, RedisCacheConfiguration> cacheConfigurations = new HashMap<>();
    
    // Short-lived cache for frequently updated data
    cacheConfigurations.put("users", RedisCacheConfiguration
        .defaultCacheConfig()
        .entryTtl(Duration.ofMinutes(5)));
    
    // Long-lived cache for static data
    cacheConfigurations.put("config", RedisCacheConfiguration
        .defaultCacheConfig()
        .entryTtl(Duration.ofHours(24)));
    
    return RedisCacheManager.builder(factory)
        .cacheDefaults(RedisCacheConfiguration.defaultCacheConfig()
            .entryTtl(Duration.ofMinutes(10)))
        .withInitialCacheConfigurations(cacheConfigurations)
        .build();
}
```

### 4. Key Naming Strategy

Use consistent, structured key naming:

```java
@Cacheable(value = "users", key = "'user:' + #id + ':profile'")
public User getUserProfile(Long id) { ... }

@Cacheable(value = "users", key = "'user:' + #id + ':permissions'")
public Set<String> getUserPermissions(Long id) { ... }
```

## Cleanup and Maintenance

### Stop and Clean Up Everything

```bash
# Using Docker Compose
cd ~/redis-cluster-compose
docker-compose down -v

# Using individual containers
docker stop redis-node-1 redis-node-2 redis-node-3 redis-node-4 redis-node-5 redis-node-6
docker rm redis-node-1 redis-node-2 redis-node-3 redis-node-4 redis-node-5 redis-node-6
docker network rm redis-cluster-net

# Remove volumes (WARNING: Deletes all data)
docker volume prune
```

### Backup Cluster Data

```bash
# Backup data from all nodes
for i in {1..6}; do
  docker exec redis-node-$i redis-cli -p 700$i --rdb /data/backup-700$i.rdb
done

# Copy backups to host
for i in {1..6}; do
  docker cp redis-node-$i:/data/backup-700$i.rdb ./backups/
done
```

## Production Considerations

When moving to production, consider these enhancements:

1. **Security:**
   - Enable authentication: Add `requirepass` to redis.conf
   - Use TLS/SSL for connections
   - Configure firewall rules
   - Use secrets management for passwords

2. **Monitoring:**
   - Integrate with Prometheus and Grafana
   - Use Redis Enterprise or managed services
   - Set up alerting for cluster health

3. **High Availability:**
   - Use at least 6 nodes (3 masters + 3 replicas)
   - Deploy across multiple availability zones
   - Configure proper cluster-require-full-coverage settings

4. **Resource Limits:**
   - Set memory limits per container
   - Configure CPU limits
   - Implement proper maxmemory policies

5. **Persistence:**
   - Configure AOF persistence for durability
   - Set up regular backups
   - Test disaster recovery procedures

## Additional Resources

- **Redis Cluster Specification:** <https://redis.io/docs/latest/operate/oss_and_stack/management/scaling/>[3]
- **Spring Data Redis Documentation:** <https://docs.spring.io/spring-data/redis/reference/redis/connection-modes.html>[2]
- **Lettuce Documentation:** https://lettuce.io/core/release/reference/
- **Docker Compose Documentation:** <https://docs.docker.com/compose/>[7]

This guide provides a complete setup for testing Redis Cluster with your Spring Boot application in a Docker environment. You now have both manual Docker commands and automated Docker Compose configurations, comprehensive Spring Boot integration, and detailed testing procedures to verify your caching implementation works correctly with clustered Redis.[8][9][10][2][1]

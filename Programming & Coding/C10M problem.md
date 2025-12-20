# We Hit Spring Boot With 1,000,000 Requests/Second… And It Did Something We Didn’t Expect

![text](https://miro.medium.com/v2/resize:fit:720/format:webp/1*LTE0bEMgif_7agCMU7j-PQ.png)

Handling **1,000,000 requests per second (1M RPS)** is NOT an application problem — it is a **systems engineering + distributed architectures + OS/kernel + NIC + GC challenge**.

This scale is part of the **C10M problem**, originally proposed by Dan Kegel, referring to safely managing 10 million concurrent connections.

To achieve this in **Spring Boot 4.0 + Java 21**, the system must completely abandon classical blocking paradigms:

- No Servlets  
- No thread-per-request  
- No JDBC  
- No synchronized lock contention  
- No blocking file I/O  
- No thread pools for reactive pipelines  

Instead, the architecture revolves around:

**Event Loops + Zero-Copy + Async Protocols + Edge Load Shedding + Distributed Caching**

---

## 1.1 The Core Runtime: Spring WebFlux + Reactor Netty

Spring WebFlux is built on **Reactor Netty**, a high-performance network engine using:

- Linux Epoll (edge-triggered)  
- DirectByteBuffer (off-heap zero-copy buffers)  
- Few threads (event loop model)  

At 1M RPS:

- Thread-per-request collapses because context switches dominate CPU time.  
- Reactive loops reuse threads, minimizing scheduling and maximizing L1/L2/L3 cache locality.  

![](https://miro.medium.com/v2/resize:fit:720/format:webp/1*wn4C8cHjHM8WIdTVy3mgbA.jpeg)
---

### 1.1.1 Netty Linux Native Transport

We enable:

- **EPOLL** — Efficient for millions of concurrent FD events.  
- **EPOLL Edge Triggered Mode** — Event-driven; avoids repeated polling.  
- **TCP_FASTOPEN** — Reduces round trips for new connections.  
- **Zero-Copy Buffering** — Uses DirectByteBuffer, avoids moving data into heap → drastically reduces GC pressure.  

---

## 1.2 Data Persistence Layer: R2DBC + Reactive Redis

A reactive frontend is useless if downstream storage is blocking.

- **JDBC** — Blocks the event loop → kills the pipeline.  
- **R2DBC** — Non-blocking, async SQL pipeline. Uses cursor-based streaming with backpressure.  
- **Redis Reactive (Lettuce)**  
  - async I/O  
  - pipelining  
  - connection multiplexing  
  - cluster-aware  

Redis becomes the primary hot-read datastore.  
PostgreSQL becomes a write-durable datastore (via R2DBC).  

![](https://miro.medium.com/v2/resize:fit:720/format:webp/1*C7MCkeiMSB2hszbZ6uwtSg.jpeg)

![](https://miro.medium.com/v2/resize:fit:720/format:webp/1*xKup7VMICwNxGe6nN8NJ6w.jpeg)
---

## Verdict

- Virtual Threads ≈ excellent for “business apps”  
- Reactive ≈ unmatched for raw scale networking  

---

## 2. Spring Boot 4.0 Full High-Performance Codebase

Your provided codebase is correct.  
I refine and expand explanations for each file.

### 2.1 `build.gradle` — Dependency Strategy (Expanded)

Key additions:

- No Servlet dependency — ONLY WebFlux  
- Native Netty Epoll  
- R2DBC + Postgres  
- Reactive Redis  
- Blockhound for blocking detection  
- Prometheus metrics for Kubernetes autoscaling  

### 2.2 Application Entry Point — Advanced Explanation

The main goals:

- Install BlockHound early — Detects blocking calls in event loop threads.  
- Set Reactor Netty tuning BEFORE Spring bootstraps  
- Create custom LoopResources  
- Fix event loop thread count.  

### 2.3 Netty Configuration — Deeper Explanation

Each TCP flag you used is required for 1M RPS:

- **SO_REUSEPORT** — Allow multiple Netty acceptor threads → eliminates accept lock.  
- **TCP_FASTOPEN** — Avoids waiting for 3-way handshake.  
- **SO_BACKLOG** — Queue for pending connections. Default = 128 → catastrophic at 1M RPS.  
- **EPOLL_EDGE_TRIGGERED** — More efficient than level-triggered in extreme concurrency.  

### 2.4 Reactive Controller Explanation

Highlights:

- `Mono<String>` ensures the Event Loop flows the packets instead of blocking threads.  
- No DTO conversions unless necessary → reduces allocations.  
- `@RequestBody Mono<String>` streams data (does not preload).  

### 2.5 RedisConfig — Advanced Tuning

- Use simple String serializers → avoid JSON parser overhead.  
- Lettuce multiplexing → support thousands of concurrent Redis ops on a few connections.  

### 2.6 GlobalError Handler — Deep Explanation

Key performance improvements:

- Avoid stack trace creation (super expensive at scale).  
- JSON responses are lightweight.  
- No blocking logging operations.  

### 2.7 `application.yml` — Deep Explanation

- Redis timeout 200ms → fail fast.  
- R2DBC pool sizing small → connections are async & multiplexed.  
- Actuator exposes P99 latency → for HPA scaling triggers.  

---

## 3. JVM Optimization for 1M RPS

This is critical.

**Why ZGC Generational?**

- Non-generational ZGC scans whole heap → wasteful for short-lived objects.  
- Generational ZGC (Java 21) optimizes short-lived objects → massive performance gain.  

---

## 4. Linux Kernel Tuning — Deep Explanation

At 1M RPS, the JVM is NOT the bottleneck.  
Linux kernel network stack collapses before the app is even reached.  

### 4.1 `sysctl.conf` Critical Parameters — Deep Explanation

- `fs.file-max = 2 million` — Because Netty connections = file descriptors.  
- `somaxconn = 65535` — Allow many pending accepts.  
- `tcp_tw_reuse = 1` — IMPORTANT for outbound connections (DB, Redis). Reduces TIME_WAIT buildup.  
- `rmem/wmem` — High-throughput packet handling buffer.  

### 4.2 NIC Tuning — Deep Explanation

NIC is often the ultimate bottleneck.  

Must enable:

- **RSS (Receive-Side Scaling)** — Distribute NIC interrupts across cores.  
- **RPS (Receive Packet Steering)** — Software fallback if hardware queues are limited.  
- **IRQ Pinning** — Bind NIC interrupts to specific CPU cores → avoids cache thrashing.  

---

## 5. Load Balancing Architecture — Deep Explanation

A single LB cannot handle 1M RPS end-to-end.  

Thus: **L4 → L7 → App Model**

- **AWS NLB** — Handles millions of concurrent TCP connections, does not inspect HTTP, lowest possible latency (~µs).  
- **NGINX / Envoy** — L7 logic: routing, header filtering, maintains keepalive pool to backend, multi-accept + epoll handles huge concurrency.  

---

## 6. Kubernetes Deployment Strategy

- Each Pod ≈ 10k RPS  
- Need ~100 pods for 1M RPS.  

**Anti-affinity** — Spread pods across nodes → avoid NIC bottleneck.  
**Guaranteed QoS** — Prevents CPU throttling.  

---

## 7. Benchmarking Strategy — Deep Explanation

Why distributed load testing?  
A single load generator cannot push >100k RPS due to:

- ephemeral port limit (~65k)  
- NIC saturation  
- client CPU exhaustion  

**Solution:** 20+ agents × 50k RPS each.  

---

## 8. Pitfalls & Best Practices (Expansions)

### 8.1 Blocking in Event Loop (Expanded)

1 blocked event-loop thread = 25k lost RPS.  

### 8.2 Virtual Thread Pinning (Expanded)

Avoid `synchronized`. Causes mounting to carrier thread.  

### 8.3 Thundering Herd (Expanded)

Redis cache expiry triggers DB stampede.  

Fix: request collapsing, probabilistic expiration (jitter).  

### 8.4 Logging Bottlenecks (Expanded)

Never log at INFO. Use sampling.  

### 8.5 GC Warmup and JIT Compilation Effects

At traffic surges:

- JIT compiler still warming → latency spikes  
- ZGC region layout not stabilized  

Solution: Warmup phase with synthetic traffic for 2 minutes after pod start, disable lazy initialization.  

### 8.6 Memory Fragmentation + Off-Heap Pressure

Netty uses DirectByteBuffers, which fragment the off-heap memory space.  

Fixes: Overprovision `MaxDirectMemorySize`, pre-touch pages, use NUMA-binding.  

### 8.7 Tail Latency Amplification (P99.9/P999)

At 1M RPS, even 1ms extra latency → thousands of queued requests.  

Causes: micro-GC pauses, Redis CPU spikes, inter-node network congestion, L7 proxy queue buildup.  

Fixes: run Redis cluster with dedicated CPU, upgrade instance families (Nitro-based AWS ENA cards), fairness queues in load balancer, isolate noisy neighbors via cgroups.  

---

## 9. Complete End-to-End Request Flow

![](https://miro.medium.com/v2/resize:fit:720/format:webp/1*Ejzg5ik5HNI311hJXAwNOw.jpeg)

---

## 11. How Many Pods? How Many Nodes?

Assume:

- Each pod = 10k RPS  
- You need 100 pods  
- Each node (e.g., AWS c6i.4xlarge) can host 8 pods  

Thus:

> 100 pods / 8 pods per node = 13 nodes  

Add 30% buffer
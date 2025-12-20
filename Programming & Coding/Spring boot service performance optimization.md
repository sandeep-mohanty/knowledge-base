# Sprint boot service performance optimization


## Introduction
When Java applications moved from bare-metal servers to Docker containers and Kubernetes, something unexpected happened:

- Apps became slower during startup  
- Services were getting OOMKilled  
- JVM started consuming more memory than allowed  
- CPU throttling caused random latency spikes  
- Developers couldn’t explain why the container failed even though logs looked normal  

**The root cause?**  
The JVM was never originally designed for container environments.  
But today we know exactly how to tune Spring Boot apps to run efficiently inside Docker.

This guide explains — in simple language — how to optimize a Spring Boot application inside Docker, including:

- A production-grade Dockerfile  
- Why it works  
- JVM + OS theory  
- Real Kubernetes issues  
- Best practices from the engineering report you uploaded  

By the end, you’ll understand Docker performance at a level that even senior engineers often miss.

---

## 1. Start With the Right Dockerfile
Here is the optimized Dockerfile:

```dockerfile
# --- Build Stage ---
FROM maven:3.9.4-eclipse-temurin-17 AS build
WORKDIR /app

# Cache dependencies
COPY pom.xml .
RUN mvn -q dependency:go-offline

# Copy sources and build
COPY src ./src
RUN mvn clean package -DskipTests

# --- Runtime Stage ---
FROM eclipse-temurin:17-jre-alpine

# JVM optimized for containers
ENV JAVA_OPTS="-XX:+UseContainerSupport -XX:MaxRAMPercentage=75 -XX:+UseG1GC"

COPY --from=build /app/target/*.jar app.jar

ENTRYPOINT ["sh", "-c", "java $JAVA_OPTS -jar app.jar"]
```

Let’s break it down.

---

## 2. Why Multi-Stage Builds Matter
Multi-stage builds give you:

- Smaller images (900MB → 150MB)  
- No Maven or source code in production image  
- Faster deployments in CI/CD  
- Better security (no build tools to exploit)  

Docker images with unnecessary layers increase attack surface and memory overhead.  
Using a minimal JRE runtime aligns perfectly with cloud-native best practices.

---

## 3. Why Copying pom.xml First Is a HUGE Optimization
This line:

```dockerfile
COPY pom.xml .
RUN mvn -q dependency:go-offline
```

Caches dependencies.  

So when you change only application code, Docker:

- does not download dependencies again  
- builds faster  
- supports incremental caching  

This reduces build time from **2 minutes → 10–15 seconds**.

---

## 4. Why JVM Tuning Is Mandatory in Docker
The JVM traditionally assumes:  
“I have the entire machine for myself.”

But in Kubernetes or ECS, your pod may have:

- 512 MB RAM  
- 1 vCPU  
- CPU limits  
- cgroups v2 enabled  

From your engineering report:  
Older JVMs read host memory, not container memory.  
So a node with 64 GB RAM made the JVM think heap = 16GB → immediate OOMKilled when the container started.

This is why we set:

```
-XX:+UseContainerSupport
```

And limit JVM heap like:

```
-XX:MaxRAMPercentage=75
```

This ensures:

- JVM uses container memory, not host memory  
- You avoid OOMKilled (exit code 137)  
- GC runs efficiently in limited RAM  

---

## 5. CPU Throttling: The Hidden Killer
Most developers focus only on memory.  
But CPU throttling is the number one reason Spring Boot slows down inside Docker.

When a container has 0.2 CPU, but the JVM spawns 10+ threads, the quota is exhausted instantly, and Linux throttles the process for the remaining 98ms of the 100ms period.

This leads to:

- Slow startup  
- Slow GC  
- Random latency spikes  
- Slow response times under load  

**Solution**  
Force JVM to use fewer internal threads:

```
-XX:ActiveProcessorCount=1
```

This prevents the JVM from thinking it has access to 64 host CPUs.

---

## 6. Garbage Collector Optimizations for Containers
### G1GC (Recommended for most microservices)
```
-XX:+UseG1GC
```
**Why?**
- Predictable pause times  
- Good for 512MB–4GB containers  
- Works well under moderate CPU limits  

Your report mentions G1GC issues with humongous allocations and tuning region size if needed.

### SerialGC (Best for <1GB RAM containers)
```
-XX:+UseSerialGC
```
**Why?**
- Lowest memory use  
- Only 1 thread  
- Zero CPU throttling issues  

### ZGC / Generational ZGC (for ultra-low latency)
Java 21 introduced Generational ZGC, which provides:

- Sub-millisecond pause times  
- High throughput  
- Ideal for financial trading, gaming, user-facing APIs  

It needs extra memory headroom, as explained in your report.

---

## 7. Thread Model Tuning: Tomcat, Virtual Threads & Pinning
### Traditional Tomcat Thread Pool
Default = 200 threads.  
200 threads → 200 MB off-heap (1MB stack per thread).  
In low-memory containers, this is fatal.

### Virtual Threads (Project Loom)
Spring Boot 3.2+ supports virtual threads:

```properties
spring.threads.virtual.enabled=true
```

**Benefits:**
- Handles 100,000+ concurrent requests  
- Almost zero memory cost  
- Perfect for I/O heavy microservices  

Your report notes the pinning problem inside synchronized blocks, especially with JDBC pools.  
**Fix:** upgrade JDBC driver + use Java 24 when available.

---

Press enter or click to view image in full size  

---

## 9. Layered JARs for Faster Builds
Spring Boot layered jars allow Docker to cache:

- dependencies  
- spring-boot-loader  
- snapshot-dependencies  
- application code  

Only layer 4 changes frequently → fastest rebuilds.  
Your report emphasized this as a major CI/CD optimization.

---

## 10. Startup Optimization: CDS, CRaC, AOT
- **CDS (Class Data Sharing)**  
  Reduces startup by 30–50%.  

- **CRaC (Checkpoint/Restore)**  
  Starts Spring Boot in ~40 ms.  
  But requires special JDK and CRIU support.  

- **Native Image (AOT)**  
  Starts instantly, uses minimal RAM.  
  Best for serverless.  

---

## 11. Kubernetes Resource Tuning
### CPU Limits
- Setting limits too low = CPU throttling  
- Setting no limit = app may overuse node  

Set CPU requests accurately.  
Set limits 2–4× requests or remove limits if monitored.

### Probes (VERY IMPORTANT)
Misconfigured probes cause endless restarts.  

Use:  
- `startupProbe` — long timeout  
- `readinessProbe` — remove from traffic  
- `livenessProbe` — restart only when hung  

---

## 12. Observability: JFR, eBPF, Async Profiler
- **JFR** for extremely low-overhead profiling  
- **async-profiler** for CPU flamegraphs  
- **ephemeral debug containers** for secure profiling in Kubernetes  

These tools help identify:  
- GC pauses  
- Thread contention  
- CPU starvation  
- Memory leaks  

This is how senior engineers debug real production issues.

# The Invisible OOMKill: A Complete Tutorial on Java + Kubernetes Memory Management

## Introduction

Imagine deploying a Spring Boot microservice that passes every integration test locally — only to watch it crash-loop endlessly in Kubernetes production. No Java exceptions. No stack traces. Just a cryptic message: `OOMKilled`. This tutorial walks you through exactly what happened, why it's so easy to miss, and how to fix it permanently.

---

## The Incident: What Went Wrong

A payment service was deployed to a Kubernetes cluster. Within minutes:

- All pod replicas began restarting every few minutes
- The ingress controller returned `503 Service Unavailable`
- App logs showed nothing — they simply stopped mid-stream
- `kubectl describe pod` revealed: `Reason: OOMKilled`

**The apparent setup looked fine:**
```yaml
resources:
  requests:
    memory: "512Mi"
  limits:
    memory: "1Gi"
```
```bash
# JVM flag set in Dockerfile
-Xmx512m
```

On paper: 512 MB heap + 1 GB limit = 512 MB headroom. Should be plenty. Right?

**Wrong.** Read on.

---

## Understanding JVM Memory: It's More Than Just the Heap

This is the most critical concept to internalize. The `-Xmx` flag only controls one part of JVM memory.

```mermaid
flowchart TD
    A[Total JVM Process Memory] --> B[Heap Memory<br>controlled by -Xmx]
    A --> C[Non-Heap Memory<br>NOT controlled by -Xmx]
    C --> D[Metaspace<br>Class metadata & bytecode]
    C --> E[Thread Stacks<br>~512KB–1MB per thread]
    C --> F[Code Cache<br>JIT-compiled native code]
    C --> G[GC Structures<br>Internal GC bookkeeping]
    C --> H[Direct Buffers<br>NIO off-heap memory]
    C --> I[Native Libraries<br>JNI allocations]

    style A fill:#4a4aaa,color:#fff
    style B fill:#3b8bd4,color:#fff
    style C fill:#d85a30,color:#fff
    style D fill:#f5c4b3,color:#333
    style E fill:#f5c4b3,color:#333
    style F fill:#f5c4b3,color:#333
    style G fill:#f5c4b3,color:#333
    style H fill:#f5c4b3,color:#333
    style I fill:#f5c4b3,color:#333
```

### Memory Component Breakdown

| Memory Region | What It Holds | Typical Size |
|---|---|---|
| **Heap** | Objects, arrays, application data | Controlled by `-Xmx` |
| **Metaspace** | Class definitions, bytecode | 100–500 MB under load |
| **Thread stacks** | Each thread's call stack | 512 KB–1 MB × thread count |
| **Code cache** | JIT-compiled machine code | 100–240 MB |
| **Direct buffers** | NIO off-heap, Netty buffers | Unbounded without limits |
| **GC overhead** | G1GC, ZGC bookkeeping structures | 50–200 MB |

### The Fatal Arithmetic

In the incident:

```
Heap (under load):          ~512 MB   ← controlled, configured
Metaspace:                  ~200 MB   ← not configured
Thread stacks (200 threads): ~200 MB  ← not configured
Code cache:                 ~100 MB   ← not configured
Direct buffers (Netty):     ~100 MB   ← not configured
─────────────────────────────────────
Total process footprint:  ~1,112 MB

Kubernetes memory limit:   1,024 MB
                         ──────────
RESULT:                  OOMKilled 💥
```

---

## Why the JVM Didn't Warn You

Unlike a Java `OutOfMemoryError` (which lets the JVM log and respond), an OOMKill is a **Linux kernel action**. The cgroup memory controller sees the process exceed its memory limit and sends `SIGKILL` — instantly. No warning. No JVM hook. No log message. Just silence.

```mermaid
sequenceDiagram
    participant App as Spring Boot App
    participant JVM as JVM Process
    participant Kernel as Linux Kernel (cgroup)
    participant K8s as Kubernetes

    App->>JVM: Allocates objects (heap stays ≤ 512m)
    JVM->>JVM: Loads classes → Metaspace grows
    JVM->>JVM: Creates threads → Stack memory grows
    JVM->>JVM: Netty/NIO → Direct buffers grow
    Note over JVM: Total RSS creeps past 1GB
    Kernel->>JVM: SIGKILL (no warning, no grace period)
    JVM--xApp: Process dies instantly
    K8s->>K8s: Detects pod exit code 137
    K8s->>K8s: Records OOMKilled, restarts pod
    Note over App,K8s: Loop repeats under load
```

Exit code `137` is the tell: it means `128 + 9` (128 + SIGKILL). If you see it in `kubectl describe pod`, you were OOMKilled.

---

## Container Awareness: Old vs New Java

Older Java versions didn't know they were in a container — they asked the host OS for total RAM and used that to auto-size the heap.

```mermaid
flowchart LR
    subgraph Host["Host Machine (64 GB RAM)"]
        subgraph Container["Container (limit: 1 GB)"]
            JVM["JVM Process"]
        end
    end

    JVM -->|"Java 8u131 and earlier\nasks host OS"| RAM["64 GB RAM reported\n→ auto-sets -Xmx ~16GB\n→ instantly OOMKilled"]
    JVM -->|"Java 8u191+ / Java 11+\nreads cgroup"| CGroup["/sys/fs/cgroup/memory\n→ sees 1 GB limit\n→ auto-sets -Xmx ~256MB"]

    style RAM fill:#f09595,color:#501313
    style CGroup fill:#c0dd97,color:#173404
```

### Minimum Recommended Java Versions for Kubernetes

| Java Version | Container Awareness | Recommendation |
|---|---|---|
| Java 8 < u131 | ❌ None | Upgrade immediately |
| Java 8 u131–u190 | ⚠️ Experimental flag needed | Use `-XX:+UnlockExperimentalVMOptions -XX:+UseCGroupMemoryLimitForHeap` |
| Java 8 ≥ u191 | ✅ Default on | Minimum acceptable |
| Java 11 | ✅ Full support | Good |
| Java 17 / 21 | ✅ Best support + modern GC | **Recommended** |

---

## The Broken Configuration (Before Fix)

```yaml
# deployment.yaml — THE BROKEN VERSION
apiVersion: apps/v1
kind: Deployment
metadata:
  name: payment-service
spec:
  replicas: 3
  template:
    spec:
      containers:
        - name: payment-service
          image: payment-service:1.0.0
          env:
            - name: JAVA_OPTS
              value: "-Xms256m -Xmx512m"  # ❌ Fixed heap, ignores container limit
          resources:
            requests:
              memory: "512Mi"
              cpu: "250m"
            limits:
              memory: "1Gi"              # ❌ No room for non-heap memory
              cpu: "1000m"
```

**Problems:**
1. `-Xmx512m` is a hard-coded assumption — it doesn't adapt when limits change
2. The 1Gi limit assumes all memory is heap — ignores ~400–600 MB of non-heap overhead
3. No metaspace limit, no direct buffer limit, no GC tuning

---

## The Fixed Configuration

### Option 1: Percentage-Based Heap (Recommended)

```yaml
# deployment.yaml — THE FIXED VERSION
apiVersion: apps/v1
kind: Deployment
metadata:
  name: payment-service
spec:
  replicas: 3
  template:
    spec:
      containers:
        - name: payment-service
          image: payment-service:1.0.0
          env:
            - name: JAVA_OPTS
              value: >-
                -XX:InitialRAMPercentage=50.0
                -XX:MaxRAMPercentage=75.0
                -XX:+UseContainerSupport
                -XX:MaxMetaspaceSize=256m
                -XX:ReservedCodeCacheSize=128m
                -Xss512k
                -XX:+UseG1GC
                -XX:+ExitOnOutOfMemoryError
          resources:
            requests:
              memory: "768Mi"
              cpu: "500m"
            limits:
              memory: "1536Mi"           # ✅ Sized for total footprint
              cpu: "1500m"
```

### Memory Budget Math (1536 Mi limit)

```
Heap (75% of 1536 MB):     1,152 MB  ← MaxRAMPercentage=75.0
Metaspace:                   256 MB  ← MaxMetaspaceSize=256m
Code Cache:                  128 MB  ← ReservedCodeCacheSize=128m
Thread stacks (200 × 512k):  100 MB  ← Xss512k
GC overhead + misc:          ~50 MB
────────────────────────────────────
Budgeted total:            1,686 MB
Safety buffer:               ~150 MB
Container limit:           1,536 MB  ← Set limit higher or reduce heap %
```

> **Rule of thumb:** Set your heap to no more than 75% of the container limit. Leave 25% for non-heap overhead.

### Option 2: Explicit Flags (Predictable but less adaptive)

```bash
JAVA_OPTS="-Xms512m -Xmx1024m \
           -XX:MaxMetaspaceSize=256m \
           -XX:ReservedCodeCacheSize=128m \
           -Xss512k \
           -XX:MaxDirectMemorySize=128m"
# Container limit should be at least 1024 + 256 + 128 + 256 + overhead ≈ 2Gi
```

---

## Practical Use Cases

### Use Case 1: High-Concurrency REST API (Spring Boot + Tomcat)

Tomcat creates one thread per request. For 200 concurrent users:

```
200 threads × 512 KB stack = 100 MB thread stack memory
+ 128 MB Tomcat connection pool buffers
+ standard non-heap overhead
= Plan for ~600 MB non-heap
```

**Config:**
```yaml
limits:
  memory: "1.5Gi"
JAVA_OPTS: "-XX:MaxRAMPercentage=60.0 -XX:MaxMetaspaceSize=256m -Xss512k"
```

### Use Case 2: Reactive Microservice (Spring WebFlux + Netty)

Netty uses a small thread pool but heavy off-heap direct buffers:

```
~16 event loop threads × 1 MB = 16 MB thread stacks
+ Netty direct buffer pool: 512 MB (default, often uncapped!)
= Critical to set -XX:MaxDirectMemorySize
```

**Config:**
```yaml
JAVA_OPTS: "-XX:MaxRAMPercentage=70.0 -XX:MaxDirectMemorySize=256m -XX:MaxMetaspaceSize=200m"
limits:
  memory: "1Gi"
```

### Use Case 3: Batch Processing Job (Short-lived pod)

Short-lived pods that process data in bulk need burst memory but can tolerate tight limits:

```yaml
JAVA_OPTS: "-XX:MaxRAMPercentage=80.0 -XX:MaxMetaspaceSize=128m"
limits:
  memory: "512Mi"
requests:
  memory: "256Mi"   # Low request = better scheduling
```

### Use Case 4: Multiple Microservices on a Node

When you have 20+ pods per node, memory stacks up fast. Use `requests` vs `limits` strategically:

```yaml
resources:
  requests:
    memory: "384Mi"   # What Kubernetes uses for scheduling
  limits:
    memory: "768Mi"   # What the pod can burst to
```

---

## Diagnosing an OOMKill: Step-by-Step

```mermaid
flowchart TD
    A["Pod keeps restarting?"] --> B{"kubectl describe pod\nshows OOMKilled?"}
    B -->|Yes| C["Check exit code 137\nin pod events"]
    B -->|No| Z["Different issue\ne.g. liveness probe, crash"]
    C --> D["kubectl top pods\nWatch memory trend"]
    D --> E{"Memory spikes\njust before kill?"}
    E -->|Gradual increase| F["Possible memory leak\nor non-heap growth"]
    E -->|Sudden spike| G["Load-related\nnon-heap burst"]
    F --> H["Enable JVM metrics\nvia JMX or Actuator"]
    G --> H
    H --> I["Check heap vs non-heap\nprometheus jvm_memory_used_bytes"]
    I --> J{"Heap at limit\nor non-heap?"}
    J -->|Heap| K["Tune -Xmx or\nfix memory leak"]
    J -->|Non-heap| L["Set MaxMetaspaceSize\nMaxDirectMemorySize\nXss"]
    K --> M["Increase container\nmemory limit"]
    L --> M
    M --> N["Add 25% buffer\nfor safety headroom"]
    N --> O["Load test in staging\nwith k6 or JMeter"]
    O --> P["Deploy with confidence ✅"]

    style A fill:#4a4aaa,color:#fff
    style P fill:#3b6d11,color:#fff
    style Z fill:#5f5e5a,color:#fff
```

### Key Diagnostic Commands

```bash
# Check pod events and exit reason
kubectl describe pod <pod-name> -n <namespace>

# Watch live memory usage
kubectl top pods -n <namespace> --watch

# Get container memory from metrics-server
kubectl get --raw /apis/metrics.k8s.io/v1beta1/namespaces/<ns>/pods/<pod>

# Check cgroup memory limit from inside the container
cat /sys/fs/cgroup/memory/memory.limit_in_bytes

# View JVM memory breakdown (with Spring Actuator)
curl http://localhost:8080/actuator/metrics/jvm.memory.used
curl http://localhost:8080/actuator/metrics/jvm.memory.max
```

### Reading JVM Memory with Prometheus

Add to your Kubernetes manifest to expose JVM metrics:

```yaml
# Add JMX Prometheus agent to JAVA_OPTS
- name: JAVA_OPTS
  value: >-
    -javaagent:/opt/jmx-exporter/jmx_prometheus_javaagent.jar=9090:/opt/jmx-exporter/config.yaml
    -XX:MaxRAMPercentage=75.0
```

Key Prometheus queries:

```promql
# Total JVM heap used
jvm_memory_used_bytes{area="heap"}

# Metaspace used
jvm_memory_used_bytes{area="nonheap", id="Metaspace"}

# Container memory vs limit (alerts at 80%)
container_memory_usage_bytes / container_spec_memory_limit_bytes > 0.8
```

---

## Liveness Probes: The Hidden Killer

Even with correct memory settings, aggressive liveness probes can kill a healthy pod during GC pause.

```mermaid
flowchart LR
    A["Request hits pod"] --> B["GC pause begins\n(G1GC stop-the-world)"]
    B --> C["App frozen\n200-500ms"]
    C --> D{"Liveness probe\ntimeout?"}
    D -->|"Probe timeout = 1s\nperiod = 5s\nfailure = 1"| E["❌ Pod killed\nas 'unhealthy'"]
    D -->|"Probe timeout = 5s\nperiod = 10s\nfailure = 3\ninitialDelay = 60s"| F["✅ Pod survives\nGC pause"]

    style E fill:#f09595,color:#501313
    style F fill:#c0dd97,color:#173404
```

**Correct probe configuration:**

```yaml
livenessProbe:
  httpGet:
    path: /actuator/health/liveness
    port: 8080
  initialDelaySeconds: 60    # Allow JVM warmup + class loading
  periodSeconds: 10
  timeoutSeconds: 5          # Allow for GC pauses
  failureThreshold: 3        # 3 consecutive failures before kill
  successThreshold: 1

readinessProbe:
  httpGet:
    path: /actuator/health/readiness
    port: 8080
  initialDelaySeconds: 30
  periodSeconds: 5
  timeoutSeconds: 3
  failureThreshold: 3
```

---

## Monitoring and Alerting Setup

```mermaid
flowchart TD
    subgraph K8s["Kubernetes Cluster"]
        Pod["Java Pod\n(JVM Metrics)"]
        Prom["Prometheus\n(Scrapes every 15s)"]
        Graf["Grafana\n(Dashboard)"]
        Alert["AlertManager\n(PagerDuty / Slack)"]
    end

    Pod -->|"JMX Exporter\nport 9090"| Prom
    Prom --> Graf
    Prom -->|"Rule: memory > 80%"| Alert
    Alert -->|"Page on-call"| Oncall["On-call Engineer"]

    style Pod fill:#3b8bd4,color:#fff
    style Alert fill:#d85a30,color:#fff
    style Oncall fill:#1d9e75,color:#fff
```

### Recommended Grafana Alerts

| Alert | Threshold | Urgency |
|---|---|---|
| Container memory approaching limit | > 80% of limit | Warning |
| Container OOMKilled | Event count > 0 | Critical |
| JVM heap saturation | > 90% of max heap | Warning |
| Metaspace growth rate | > 10 MB/min sustained | Warning |
| GC pause time | P99 > 500ms | Warning |

---

## Best Practices Checklist

```mermaid
flowchart TD
    A["Java App for Kubernetes"] --> B["Use Java 11+ or 17+\n(container-aware JVM)"]
    B --> C["Set MaxRAMPercentage=75\nnot fixed -Xmx"]
    C --> D["Cap non-heap memory:\nMetaspace, CodeCache,\nDirectMemory, Xss"]
    D --> E["Size container limit =\ntotal expected RSS + 25%"]
    E --> F["Configure liveness probes\nwith GC pause tolerance"]
    F --> G["Expose JVM metrics\nvia Actuator + Prometheus"]
    G --> H["Load test in staging\nwith k6 / JMeter"]
    H --> I["Alert at 80% memory\nbefore kernel acts"]
    I --> J["Handle SIGTERM gracefully\n(spring.lifecycle.timeout-per-\nshutdown-phase=20s)"]
    J --> K["Run blameless post-mortems\non every OOMKill"]
    K --> L["✅ Production-Ready\nJava on Kubernetes"]

    style A fill:#534ab7,color:#fff
    style L fill:#0f6e56,color:#fff
```

### Graceful Shutdown (Spring Boot)

```yaml
# application.yaml
spring:
  lifecycle:
    timeout-per-shutdown-phase: 20s

server:
  shutdown: graceful  # Drain in-flight requests before exiting
```

```yaml
# Kubernetes gives pods this long to shut down
terminationGracePeriodSeconds: 30
```

When Kubernetes sends `SIGTERM`, Spring Boot will stop accepting new requests and finish in-flight ones — up to 20 seconds. After that, Kubernetes sends `SIGKILL`. Align `timeout-per-shutdown-phase` to always be less than `terminationGracePeriodSeconds`.

---

## The Complete Picture: JVM in a Pod

```mermaid
flowchart TD
    subgraph Node["Kubernetes Node (8 GB RAM)"]
        subgraph Pod["Pod (limit: 1536 Mi)"]
            subgraph Container["Container"]
                subgraph JVM["JVM Process"]
                    Heap["Heap\n75% = ~1.1 GB\n(-XX:MaxRAMPercentage=75)"]
                    Meta["Metaspace\n≤ 256 MB\n(-XX:MaxMetaspaceSize)"]
                    Code["Code Cache\n≤ 128 MB\n(-XX:ReservedCodeCacheSize)"]
                    Stacks["Thread Stacks\n512 KB × threads\n(-Xss512k)"]
                    Direct["Direct Buffers\n≤ 128 MB\n(-XX:MaxDirectMemorySize)"]
                end
                OS["OS + native overhead\n~50–100 MB"]
            end
        end
        CGroup["cgroup memory controller\nEnforces 1536 Mi hard limit\nKills at limit: exit 137"]
    end
    JVM --> CGroup

    style Heap fill:#185fa5,color:#fff
    style Meta fill:#993c1d,color:#fff
    style Code fill:#854f0b,color:#fff
    style Stacks fill:#533489,color:#fff
    style Direct fill:#0f6e56,color:#fff
    style CGroup fill:#a32d2d,color:#fff
```

---

## Summary

The "invisible OOMKill" is invisible only because most developers think of JVM memory as just the heap. The real lesson:

1. **The JVM has a large memory footprint beyond the heap** — metaspace, thread stacks, code cache, direct buffers, and GC structures all count against your cgroup limit.
2. **Use `MaxRAMPercentage` instead of fixed `-Xmx`** — it adapts automatically when you resize your Kubernetes limits.
3. **Always cap non-heap regions explicitly** — `MaxMetaspaceSize`, `ReservedCodeCacheSize`, `MaxDirectMemorySize`, and `Xss` prevent unbounded growth.
4. **The Linux kernel kills with no warning** — exit code 137, no Java exception, no log entry. Watch for it in `kubectl describe pod`.
5. **Load test before production** — OOMKills rarely appear under low load. They surface under concurrency.
6. **Monitor both container RSS and JVM internal metrics** — heap utilization alone won't warn you in time.

> The cgroup memory limit is a hard ceiling enforced by the kernel. The JVM does not know it's about to be killed. Your job is to ensure the total JVM process footprint — every byte of it — fits comfortably within that ceiling before peak traffic arrives.

---

*Happy and safe coding.*
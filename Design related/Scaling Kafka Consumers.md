# Scaling Kafka Consumers: Proxy vs. Client Library for High-Throughput Architectures

Apache Kafka is a powerful foundation for **event-driven architectures**. It enables **true decoupling** between producers and consumers. A single event can be consumed by multiple services, each evolving independently. The **pull-based consumption model** in Kafka is a key enabler of this flexibility, giving consumers full control over how and when to process data.

This architecture works well in most deployments. In fact, in many real-world scenarios, having **multiple independent consumers** per Kafka topic is the **right pattern**. It aligns with microservices best practices and supports autonomous teams building independent applications leveraging domain-driven design (DDD).

However, as data volumes grow and the number of consumer applications increases, organizations start hitting **scalability and operational limits**. Companies like [**Wix**](https://www.wix.engineering/post/from-bottleneck-to-breakthrough-how-wix-cut-kafka-costs-by-30-with-a-push-based-consumer-proxy) and [**Uber**](https://www.uber.com/en-DE/blog/kafka-async-queuing-with-consumer-proxy/) have published excellent insights into the problems they encountered, and how they adapted Kafka consumption for their scale.

This article highlights those challenges, explores emerging solutions, and outlines what data streaming vendors could offer to make large-scale consumption more efficient – without losing the benefits of Kafka’s core architecture.

![Scaling Apache Kafka Consumers for High Throughput with Proxy or Client Library for API and Database Integration](https://www.kai-waehner.de/wp-content/uploads/2025/07/Scaling-Apache-Kafka-Consumers-for-High-Throughput-with-Proxy-or-Client-Library-for-API-and-Database-Integration-1024x770.png)

Join the **data streaming community** and stay informed about new blog posts by [subscribing to my newsletter](https://www.kai-waehner.de/newsletter) and follow me on [LinkedIn](http://linkedin.com/in/kaiwaehner) or [X (former Twitter)](https://twitter.com/kaiwaehner) to stay in touch. And make sure to download my [free book about data streaming use cases](https://www.kai-waehner.de/ebook), including various data Kafka architectures and best practices.

## Apache Kafka Consumer Challenges at Scale

While Kafka brokers scale horizontally with ease, **consumers introduce complexity** at scale. Let’s look at the key challenges.

### Operational Overhead and Cost

Wix observed that spinning up many microservices, each with its own Kafka consumer group, significantly **increased cluster load and costs**. More consumer groups meant more partition assignments, more metadata overhead, and more compute required across the platform.

This model also led to resource fragmentation. Each service needed its own scaling logic, retry handling, and monitoring. The operational burden grew rapidly.

### Head-of-Line Blocking and Poison Pills

Kafka guarantees **message ordering within a partition**, which can create bottlenecks. A **single slow or failing message** – often called a _poison pill –_ can **block all following records** in that partition. This is known as **head-of-line blocking**.

Without parallelism or proper isolation, one bad message can impact SLAs for critical pipelines.

### Complex Error Handling and Dead Letter Queue (DLQ)

Handling failures at scale is difficult. Many companies implement **dead letter queues (DLQs)** to isolate problematic messages. But Kafka doesn’t have a built-in DLQ mechanism. Teams have to build **custom logic to identify failures, forward them to DLQs, monitor the flow, and reprocess data**.

Managing DLQs across hundreds of services becomes an architectural burden and an operational risk.

Handling failures and implementing dead letter queues at scale demand thoughtful solutions. The blog post “[Error Handling via Dead Letter Queue in Apache Kafka](https://www.kai-waehner.de/blog/2022/05/30/error-handling-via-dead-letter-queue-in-apache-kafka/)” outlines common DLQ patterns, trade-offs, and resilient design approaches for Kafka-based systems. It features in-depth case studies from **Uber**, **CrowdStrike**, **Santander Bank**, and **Robinhood**, showcasing how each organization tackled error handling in production environments.

### Limited Backpressure Control

Kafka consumers pull messages at their own pace. This is [ideal for true decoupling using a domain driven design with microservices and data products](https://www.confluent.io/blog/microservices-apache-kafka-domain-driven-design/). It remains the **best approach for many scenarios, regardless of whether the communication protocol is streaming, request-response, or message queue**.

![Domain Driven Design with Event Streaming Using Apache Kafka for True Decoupling Building Microservices and Data Products](https://www.kai-waehner.de/wp-content/uploads/2025/07/Domain-Driven-Design-with-Event-Streaming-Using-Apache-Kafka-for-True-Decoupling-Building-Microservices-and-Data-Products-1024x583.png)

But it creates challenges when **downstream systems are under pressure**. For example, a consumer may pull and process messages faster than a database or external API can handle.

Without explicit rate-limiting or buffering, services may crash or trigger cascading failures.

## Solutions for Kafka Consumers at Scale: Proxy Patterns and Client Libraries

To address these issues, large-scale Kafka users are adopting two main strategies: **centralized proxy layers** and **enhanced consumer libraries**. In some scenarios, **last-mile fan-out** approaches are used instead for mobile, web, or IoT clients that require internet-scale delivery.

### Push-Based Consumer Proxy for Kafka

Kafka’s pull-based model enables flexibility, but some organizations are evolving their consumption architecture for greater efficiency. **Both Uber and Wix introduced a** **push-based consumer proxy** to decouple ingestion from processing. More details:

-   [From Bottleneck to Breakthrough: How Wix Cut Kafka Costs by 30% with a Push-Based Consumer Proxy](https://www.wix.engineering/post/from-bottleneck-to-breakthrough-how-wix-cut-kafka-costs-by-30-with-a-push-based-consumer-proxy)
-   [Enabling Seamless Kafka Async Queuing with Consumer Proxy at Uber](https://www.uber.com/en-DE/blog/kafka-async-queuing-with-consumer-proxy/)

Before I go on with the details: **Don’t (try to) solve a problem that you do NOT have yet!**

This challenge is mainly seen at extreme scale deployments. **Uber processes trillions of messages and multiple petabytes of data per day. Wix runs 7% of the internet’s websites.** This is extreme scale with different challenges compared to what most organizations face.

**How It Works:**

-   A centralized service reads from Kafka topics.
-   It then **pushes** **messages to downstream services** using HTTP, gRPC, or a message queue.
-   These backend services no longer need to implement Kafka clients or manage offsets.

Here is the high level architecture from Uber’s Kafka Consumer Proxy using gRPC and message queues:

![Kafka Async Queuing with Consumer Proxy at Uber](https://www.kai-waehner.de/wp-content/uploads/2025/07/Kafka-Async-Queuing-with-Consumer-Proxy-at-Uber-1024x246.png)

Source: Uber

**Benefits:**

-   **Fewer Kafka clients reduce broker load and network chatter.**
-   Services are isolated from Kafka-specific logic.
-   Retry, error handling, and DLQ logic is centralized.
-   Head-of-line blocking is avoided by queuing messages per service.

**Trade-offs:**

-   **Requires building and maintaining new infrastructure.**
-   Introduces latency and operational complexity between Kafka and services.
-   Error diagnosis becomes harder across multiple layers.

Here is a deeper look into Wix’ push-based Kafka Consumer Proxy using gRPC:

![Wix Push Based Consumer Proxy Architecture for Apache Kafka](https://www.kai-waehner.de/wp-content/uploads/2025/07/Wix-Push-Based-Consumer-Proxy-Architecture-for-Apache-Kafka-1024x521.png)

Source: Wix

This pattern works well for companies with dedicated platform teams and extreme consumption throughput. Wix reported a **30% reduction in Kafka costs**. Uber improved fault isolation and throughput by decoupling processing from Kafka itself.

But this solution isn’t a fit for every organization. **For small and medium Kafka deployments, building a consumer proxy adds unnecessary complexity, overhead, and risk**.

### Client-Side Parallel Processing Consumer Library for Apache Kafka

For many teams, **improving how consumers process messages is more practical than introducing a proxy**. This is where [**Confluent’s Parallel Consumer** **open source library**](https://github.com/confluentinc/parallel-consumer) comes into play.

**What It Does:**

-   Enhances Kafka consumer throughput by enabling **parallel processing per partition**.
-   Maintains **ordering guarantees** when needed (e.g., by key).
-   Supports **asynchronous processing**, retries, and backpressure control.

**Benefits:**

-   **Easy to integrate – just swap the Kafka client for the library.**
-   Ideal for CPU- or I/O-bound processing logic.
-   Reduces head-of-line blocking without infrastructure changes.

**Trade-offs:**

-   **Adds complexity to application code.**
-   Developers need to understand async patterns and concurrency.
-   Not a drop-in solution for all workloads.

Here is the Kafka consumer architecture without the parallel consumer:

![Kafka Consumer Challenges at Scale](https://www.kai-waehner.de/wp-content/uploads/2025/07/Kafka-Consumer-Challenges-at-Scale-1024x568.png)

Source: Confluent

And the solution architecture using the Confluent’s open source Parallel Consumer library for Apache Kafka:

![Parallel Consumer Library for Apache Kafka to Scale Web Service API and Database Requests](https://www.kai-waehner.de/wp-content/uploads/2025/07/Parallel-Consumer-Library-for-Apache-Kafka-to-Scale-Web-Service-API-and-Database-Requests.png)

Source: Confluent

**The Parallel Consumer is a strong fit for teams that want to (and can) stay in the Kafka consumption model but need better performance and flexibility when connecting to slower external services like a database or HTTP request-response API.**

### Last-Mile Fan-Out for Millions of Mobile Apps, Internet Clients and Internet of Things (IoT)

Another important pattern extends beyond internal services: **last-mile fan-out**. Many organizations need to deliver curated Kafka data directly to millions of web browsers, mobile apps, or connected IoT devices. **Specialized push-based proxies** provide this capability for internet clients by maintaining long-lived WebSocket connections and pushing updates only when data changes.

![Last Mile Proxy for Edge Mobile App and WebSockets Integration from Apache Kafka](https://www.kai-waehner.de/wp-content/uploads/2025/09/Last-Mile-Proxy-for-Edge-Mobile-App-and-WebSockets-Integration-from-Apache-Kafka-1024x585.png)

Last-Mile Proxy as Bridge between Kafka and Mobile/Internet Apps (Source: Lightstreamer)

In the IoT space, MQTT brokers serve a similar role, connecting Kafka pipelines to fleets of sensors, vehicles, or machines.

These delivery layers handle critical concerns like backpressure, reconnection, and state snapshots at scale. By offloading internet- and device-scale distribution to dedicated systems, Kafka consumption on the backend remains clean and efficient. This **separation of concerns aligns well with organizational structures**, where dedicated teams often manage mobile or IoT delivery platforms while Kafka continues to power the enterprise event backbone.

## How a Data Streaming Platform Could Help…

Kafka’s architecture is powerful because of its **decoupling** and **pull-based model**. These are not weaknesses – they are strengths in most scenarios and differentiators to traditional message brokers like IBM MQ or RabbitMQ. The key is to choose the **right model** for the **right use case**.

Still, there is room for improvement. Here’s what a data streaming platform could include to avoid the need of external proxies or client libraries:

### Serverless Kafka Consumer Runtime

Imagine a built-in service that provides:

-   **Auto-scaling consumption** across topics and partitions.
-   **Built-in push delivery** to services via HTTP, gRPC, or serverless functions.
-   **Standardized DLQ handling** and poison pill isolation.
-   **Configurable ordering guarantees** and retries.
-   **End-to-end visibility** into lag, failures, and throughput.

This would eliminate the need for most teams to build their own proxies or manage retries. It would combine the **efficiency of a proxy**, the **control of a library**, and the **low TCO of a managed service**.

### Total Cost of Ownership (TCO) Matters

New features are valuable. But enterprises also care deeply about **cost, simplicity, and risk**. Both solutions discussed above – building a proxy or adopting a library – have **hidden costs**:

-   Engineering time and platform support
-   Operational complexity and monitoring
-   Potential lock-in or rigidity if the design doesn’t scale further

A vendor-managed solution could offer **sensible defaults**, simplified configuration, and end-to-end support to reduce not just technical challenges, but also the **TCO of large-scale event streaming**.

Confluent Cloud, as an elastic SaaS, addresses some of these challenges by removing most operational overhead and providing scale and resilience out of the box. Unlike open source Kafka, it is not as sensitive to Consumer Group fan-out and scales orders of magnitude higher thanks to its cloud-native architecture. If needed, a cloud Kafka service can be combined with the patterns explored in this article to scale consumers more efficiently.

## Kafka Proxy: A Different Problem

A central Kafka proxy is a different (very relevant) problem: But it is not about scaling consumers like Uber’s or Wix’s proxies. They solve **compliance, security, and governance** challenges.

A (server-side) Kafka proxy sits between clients and brokers to enforce rules without changing applications or Kafka itself. This is useful for **PCI/GDPR compliance**, **multi-tenant SaaS isolation**, and **protecting against insider threats**.

Solutions like **Kroxylicious** or **Confluent’s built-in proxy features** follow this model. They use filter chains to handle **encryption, audit, policy enforcement, and access control** while passing through traffic with minimal overhead.

The key: keep proxies focused on infrastructure-level concerns, not business logic. Think of them as Kafka’s version of **API management gateways**, but protocol-aware.

## From Simplicity to Scale: The Next Step in Kafka Architecture

Kafka remains the best-in-class solution for **event-driven data architectures**. Its pull-based model and decoupled design are core to its value. **For many use cases, the native pull-based consumer model with multiple consumers per topic is the best approach**.

However, at **large scale**, teams must re-evaluate how they consume Kafka events. Whether through a push-based proxy or smarter client libraries, organizations are evolving their consumer architecture to meet growing demands.

The next step is for a **data streaming platform to** **abstract this complexity** without sacrificing flexibility. Serverless consumption models could bring the same leap forward that managed Kafka clusters brought to event streaming infrastructure.

Choose the right solution for your scale. **Keep your architecture simple – until it isn’t enough**. Then, evolve with purpose.

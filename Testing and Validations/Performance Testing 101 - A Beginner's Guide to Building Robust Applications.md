# Performance Testing 101: A Beginner's Guide to Building Robust Applications

This guide is for anyone who has built an application and wants to ensure it doesn't fall over when real people start using it. We'll walk through the essentials of performance testing without the complicated jargon, focusing on practical steps you can take to make your app robust and reliable.

## Why Is Performance Testing Important?

Imagine you’ve built an application that works fine until thousands of users come in at the same time, and you don't know how to handle this load. You're scaling your app to meet new users, but in parallel, you're seeing a growing bill for the infrastructure cost. At one moment, you'd decide to stop growing and allow users to have a slow, degraded experience.

This is where performance tests jump in to help you avoid this situation by measuring your performance beforehand and finding bottlenecks before real usage.

The article is based on my own experiences and explores how to make life simpler while avoiding unnecessary burdens.

Hereinafter, when I say 'instance,' I mean a single running copy of the application.

## Step 1: Define Your Goal

The most important step is defining the goal of your performance tests: what exactly do you want to get as a result? While you can pursue multiple goals eventually, it's best to start by focusing on just one.

Example of goals:

-   Find resilience limits and determine the Requests Per Second (RPS) an instance can handle before breaking, and define resilience parameters (like circuit breakers or bulkheads) to protect it.
-   Find memory leaks that happen during the handling of RPS.
-   Establish baselines and define the 95th (or 99th) percentile of response time that is important to monitor in production.
-   Verify that scaling happens in time and adjust the scaling policy.

Your goal will determine the type of test you need to run.

## Step 2: Choose the Right Type of Test

There are several types of performance tests, each designed to answer a different question.

### **Load Test**

This simulates the expected user load on an instance.

-   **Objective**: To identify performance bottlenecks and determine the system's normal operating capacity.
-   **Example**: Testing an e-commerce website with the number of concurrent users expected during a Black Friday sale.

### ****Stress Testing****

This pushes the system beyond its normal operational limits to find its breaking point.

-   **Objective**: To determine the system's stability in extreme conditions and its recovery process (e.g., does it crash gracefully or fail catastrophically?).
-   **Example**: Simulating twice the maximum expected user load on a login server to see when it stops responding.

### **Endurance Testing**

Also known as soak testing, this evaluates a system's performance over a prolonged period under a sustained, significant load.

-   **Objective**: To find memory leaks, performance degradation, or connection issues that occur over time.
-   **Example**: Running a test on a server for 24 hours with a continuous, heavy user load to monitor memory usage and response times.

### **Spike Testing**

This checks how a system performs when the user load suddenly and dramatically increases. Unlike a gradual increase in load, this test simulates an abrupt spike in traffic.

-   **Objective**: To determine if the system can handle sudden bursts of activity and how quickly it can recover and return to a normal operational level.
-   **Example**: Testing a news website's performance right after a major story breaks, causing a massive, immediate influx of visitors.

### **Volume Testing**

This focuses on the database. It involves populating a database with a very large amount of data and then testing the application's performance. It's also known as flood testing.

-   **Objective**: To analyze the system's response time and behavior when dealing with a large volume of data. It helps identify issues like slow queries or database connection problems.
-   **Example**: Querying a customer database that has been filled with millions of records to see how long search operations take.

### ****Scalability Testing****

This measures an application's ability to "scale up" or "scale out" to handle an increasing user load. Scaling up involves adding more resources (like CPU or RAM) to an existing server, while scaling out means adding more servers to the system.

-   **Objective**: To determine how effectively an application can handle growth in user traffic by adding more resources.
-   **Example**: Gradually increasing the user load on a web application while simultaneously adding more web servers to see if the performance per user remains consistent.

## Step 3: Isolate Your Application With Stubs

A great performance test _**isolates**_ your application. If your app calls an external service, that service's slowness could make your application look bad. To prevent this, we use **stubs** — mock versions of these external dependencies — to ensure we are only testing our own code.

A fantastic tool for this is **Wiremock**. It's flexible and can mock many protocols like HTTP, SSE, GRPC, and GraphQL. You can also record requests/responses to make stubs on the fly.

But what if that external API is _normally_ slow? We need to simulate that. This is where we can add a dash of chaos engineering to our routine

### **Simulating Delays With Chaos Engineering**

To work it out, you can use several toolsets depending on your environment:

#### **Docker**

-   **`tc` and `netem`**: You use the `tc` command to add `netem` rules to a container's virtual network interface.
-   **Pumba**: Pumba runs as a container itself and interacts with the Docker daemon to manipulate the network of other containers using `tc` and `netem` under the hood. For example, to add a 2-second delay to a container named `my-app`: `pumba netem --duration 5m delay --time 2000 my-app`.

#### **K8s**

-   **Chaos Mesh**: A powerful platform for injecting failures, including network delays, using simple YAML files
-   **LitmusChaos**: Another popular framework with a "ChaosHub" of pre-defined experiments, perfect for CI/CD integration.
-   **Istio/Linkerd (Service Meshes)**: If you're already using a service mesh, they have built-in capabilities to inject network delays and other faults.

## Step 4: Set Up Your Tooling

You'll need tools to generate load and monitor the results.

To actually generate the load and run the performance tests, you'll need an evaluation tool. Popular choices include:

-   Gatling and K6 are code-based and excellent for developers who want to integrate tests into their CI/CD pipelines.
-   Apache JMeter has a user interface and is powerful, making it a good choice for those who prefer less code.
-   Apache Benchmark (ab) is great for a quick and simple test from your command line.

### **Monitoring Tools**

While your tests are running, you need to see what's happening under the hood. That's where monitoring tools come in. In the modern world, we have many tools to provide it:

-   **OpenTelemetry**: A standard to unify logs, metrics, and traces. It's too wide a topic to describe here, but you can learn more at their official docs.
-   **Jaeger**: A popular tool for trace monitoring.
-   **Prometheus**: A time-series database to collect metrics.
-   **Grafana**: This is a visualization tool for creating dashboards from data in Prometheus. You can find many reusable dashboards in official GitHub repositories.

## Step 5: Know Your Metrics and Patterns

### **Key Metrics to Watch**

Of course, these things depend on your case and target. It’s almost impossible to hone them all in one moment.

-   **System metrics**: These are all related to operating system resources—RAM memory, storage, CPU, network bandwidth, disk I/O, etc.
-   **Application metrics**: These are all related to your application. To be more precise, in the Java world, you can monitor JVM memory regions, GC pauses, and HTTP server metrics like the number of successful requests or the state of connection pools.
-   **Business metrics**: These kinds of metrics are related to business activity, for example, the delay of a specific endpoint.

### **Key Resilience Patterns**

Resilience patterns are important because they are proactive strategies designed to prevent system failures, protect resources, and ensure an application remains functional and responsive, even when parts of it are slow, failing, or under extreme load.

Each pattern addresses a specific type of potential failure, making the overall application more robust:

1.  **Circuit breaker**: Acts like an electrical circuit breaker. It monitors calls to an external service, and if failures reach a threshold, the circuit "opens," and subsequent calls fail immediately without being attempted. After a timeout, it moves to a "half-open" state to see if the service has recovered before fully "closing".
    -   **What it prevents**: It prevents an application from wasting resources on a service that is down or struggling, which stops a local failure from causing a cascading failure across the system.
2.  **Bulkhead**: Named after the partitions in a ship's hull, this pattern isolates parts of your application into separate resource pools (like thread pools). If one part fails, it's contained within its own bulkhead and doesn't exhaust resources needed by other parts.
    -   **What it prevents**: It prevents a failure in one part of the system from bringing down the entire application and can help avoid out-of-memory errors. It ensures fault isolation, so even if one feature is down, others can remain available.
3.  **Timeout**: A simple but critical pattern where you set a maximum time limit for waiting on a response from an external call. If the service doesn't respond within that timeframe, the call is automatically abandoned.
    -   **What it prevents**: It prevents system resources (like threads) from being blocked indefinitely waiting for a slow service, which avoids resource exhaustion and keeps your application responsive.
4.  **Retry**: Automatically re-attempts an operation that has failed. It's most effective for transient errors, like a brief network glitch. It's often configured with a delay between retries (e.g., "exponential backoff") to give the service time to recover.
    -   **What it prevents**: It prevents temporary, intermittent failures from being treated as permanent errors, increasing the resilience and success rate of operations.
5.  **Rate Limiter**: Controls the amount of incoming traffic or requests a service accepts in a given period. If the limit is exceeded, the limiter rejects requests, typically with an HTTP 429 "Too Many Requests" status.
    -   **What it prevents**: It protects your services from being overwhelmed by too many requests at once, preventing system overload and helping to mitigate Denial-of-Service (DoS) attacks.
6.  **Cache**: Involves storing a copy of frequently accessed data in a location that is faster to access than the source (e.g., in memory).
    -   **What it prevents**: It prevents unnecessary load on backend systems (like databases) and reduces response times for users. This improves performance and can even keep parts of your application functional if a backend is temporarily down

## Step 6: Record and Analyze Your Results

Depending on the goal, different types of results are collected. It's expected that each test run should be performed on a 'fresh' instance, without the impact of the previous run.

Here is an example of a table you can fill during load testing:

| Run | Baseline | RPS | Multiplier | Duration | Expected/Actual number of requests | Median t | Avg t (ms) | p95 t (ms) | Error rate | other metrics |
| --- | ---| ---| --- | --- | --- | --- | --- | --- | --- | --- |
| 1 | 100 | 3 |
| 2 | 150 | 3 |
| 3 | 200 | 3 |
| 4 | 225 | 3 |
| 5 | 250 | 3 |

### **Column Explanations**

-   **Baseline RPS** – the "normal" requests per second you are testing
-   **Multiplier** – value on which baseline RPS is multiplied during testing. For example, you want to ramp up the load from 100 to 100\***Multiplier****,** from 100 to 300 during a specific period of time.
-   **Median t** – the 50th percentile of response time in milliseconds
-   **Avg t** – the average of response time
-   **Error rate** – how many requests fail during the processing
-   **p95 t (ms)** – the 95th percentile of response time in milliseconds. This means 95% of requests were faster than this value, and it's a key indicator of user experience.
-   **Other metrics** – specific metrics to make an informed decision

## Conclusion

Performance testing can seem daunting, but by starting with a clear goal and taking a structured approach, you can systematically improve your application's reliability. It's not about passing or failing; it's about learning how your system behaves under pressure and making it better. Start small, iterate, and you'll build an application that can stand up to real-world demand.

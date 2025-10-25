# Implement Distributed Tracing with Spring Boot 3

A practical guide to add OpenTelemetry tracing to Spring Boot 3: agent setup, context propagation, messaging, sampling, and exports.


A slow checkout request. A background job stuck waiting on another service. A log message that looks fine — until performance drops. In a Node.js microservices setup, these are the moments that test your observability. You know something's wrong, but tracing the request across dozens of services feels impossible. Distributed tracing changes that. It connects every span in the request's journey, showing exactly where time is spent and where things start to break down.

## What is Distributed Tracing in Modern Microservices

In most modern systems, a single request moves through several services, APIs, and databases before it returns a response. Without a clear way to follow that path, finding the source of latency or errors quickly becomes guesswork. Distributed tracing solves this by tracking each request across services, making it easier to spot where delays occur and what caused them.

Distributed tracing is a methodology and a set of tools designed to monitor and visualize requests as they flow through a distributed system. It captures the end-to-end journey of a request, providing a comprehensive view of its execution path across multiple services, processes, and even different infrastructure components. This holistic perspective is crucial for deciphering complex interactions.

### Why Distributed Tracing Matters for Spring Boot 3 Applications

As [Spring Boot 3](https://spring.io/projects/spring-boot) applications grow into multiple services, keeping track of what happens between them becomes harder. A single request might pass through an API Gateway, an authentication layer, a business service, and several data access components before completing. Without distributed tracing, you're left guessing where things go wrong.

**Debugging slows down:** It's tough to pinpoint which service caused a delay or returned an error.

**Performance issues stay hidden:** You can't easily tell which dependency is responsible for latency.

**Root cause analysis takes longer:** Teams spend hours correlating logs across systems, increasing MTTR.

**Limited visibility into user impact:** You lose sight of how a user request actually moves through your application.

Distributed tracing gives you that missing context. It connects each step of a request — across services, threads, and APIs — into a single trace. You can see where time is spent, which calls fail, and how one slow service affects the rest of the flow. If you're comparing tracing backends for your setup, check out Jaeger vs Zipkin: Choosing the Right Distributed Tracing System.

## Key Concepts: Traces, Spans, and Context Propagation

Before you set up distributed tracing, three building blocks make it work: traces, spans, and context propagation.

**Trace**

A trace is the complete path of a request through your system. It shows every service the request touches — from your API gateway down to the database. You can think of it as a timeline that captures everything that happens from the moment a request starts until it's done.

**Span**

A span is one piece of work within that request. It might be an API call, a database query, or a function execution inside your service. Each span has a name, a start and end time, and attributes like status codes or request URLs. Spans are nested — a parent span might represent an incoming HTTP request, and its child spans capture all the downstream calls triggered by it.

**Context Propagation**

This is how tracing data moves between services. When your service calls another one, it passes along the trace and span IDs — usually through HTTP headers or message metadata. That's what keeps all spans connected as part of the same trace. Without proper context propagation, traces break apart, and you lose the end-to-end view of a request.

## Choose Your Distributed Tracing Solution for Spring Boot 3

When you add distributed tracing to a Spring Boot 3 application, the first decision you face is how. The right choice determines not just how easy it is to instrument your services, but also how well the system will scale, integrate, and stay maintainable over time.

### OpenTelemetry: The Future of Observability

The shift to [OpenTelemetry](https://opentelemetry.io/) (OTel) marks a major evolution in how developers approach observability. It's no longer just about traces — OTel unifies metrics, logs, and traces into one consistent model. Built under the [CNCF](https://www.cncf.io/), it's open-source and vendor-neutral, making it an ideal fit for Spring Boot 3 applications that need both flexibility and longevity.

Here's why most modern Spring Boot teams are moving to OTel:

**Vendor neutrality:** You can switch between backends like [Jaeger](https://www.jaegertracing.io/), [Zipkin](https://zipkin.io/), [Datadog](https://www.datadoghq.com/), or [New Relic](https://newrelic.com/) without rewriting instrumentation.

**Unified observability:** Traces, metrics, and logs follow the same standard — giving you one consistent source of truth.

**Language consistency:** If your ecosystem mixes Java, Go, and Node.js, OTel keeps telemetry data uniform across them.

**Community-driven evolution:** With contributions from Google, Microsoft, and others, OTel moves fast and stays stable.

For Spring Boot 3, OTel isn't just an option — it's the default direction. It replaces multiple fragmented tracing approaches with a single, well-defined standard that scales across teams and environments.

### What About Spring Cloud Sleuth?

If you've been building with Spring for a while, [Spring Cloud Sleuth](https://spring.io/projects/spring-cloud-sleuth) probably feels familiar. It was once the easiest way to get tracing out of a Spring application — auto-instrumenting components and working neatly with Brave and Zipkin.

But Sleuth has reached the end of its lifecycle. It's no longer actively maintained, and most of its capabilities have now been merged into OpenTelemetry. If you're still using Sleuth, the spring-cloud-sleuth-otel bridge offers a transitional path — letting your existing Sleuth-based code emit OpenTelemetry-compatible traces.

For new Spring Boot 3 projects, however, direct OpenTelemetry integration is the better long-term path. It cuts out an extra abstraction layer, gives you more control over instrumentation, and ensures compatibility with future updates in the Spring ecosystem.

### Integrate OpenTelemetry Into Your Stack

Of course, adopting a new tracing solution doesn't happen in isolation — you'll want it to work smoothly with your existing monitoring tools. A few considerations help make that transition clean:

**Check backend support:** Most modern observability platforms — from [Grafana Tempo](https://grafana.com/oss/tempo/) to Datadog and [Last9](https://last9.io/) — already support the OpenTelemetry Protocol (OTLP).

**Avoid data silos:** Correlating traces with metrics and logs is where distributed tracing shines. Keep your telemetry unified to avoid gaps in analysis.

**Watch for agent conflicts:** If your environment already runs APM agents, ensure they don't overlap with the OpenTelemetry Java Agent, especially when it comes to bytecode instrumentation.

## Step-by-Step Process to Implement OpenTelemetry with Spring Boot 3

There are a few ways to add OpenTelemetry to your Spring Boot 3 application — from a plug-and-play setup using the Java Agent to more flexible, code-based configuration. The right approach depends on how much control you need over instrumentation and data flow.

### 1\. Using the OpenTelemetry Java Agent

The [OpenTelemetry Java Agent](https://github.com/open-telemetry/opentelemetry-java-instrumentation) is the quickest way to enable tracing. It doesn't require code changes and automatically instruments most common frameworks — Spring MVC, WebFlux, JDBC, HTTP clients, and more.

**Setup steps:**

Download the agent from the [OpenTelemetry Java Instrumentation releases](https://github.com/open-telemetry/opentelemetry-java-instrumentation/releases).

**Key configurations:**

-   `otel.service.name`: Logical name of your service in the trace data.
-   `otel.exporter.otlp.endpoint`: The collector or backend endpoint to send traces.
-   `otel.traces.exporter=otlp`: Sets the OTLP exporter.
-   `otel.resource.attributes=env=prod,app.version=1.0.0`: Adds metadata to all spans.

Start your Spring Boot app with the agent attached:

```
java -javaagent:/path/to/opentelemetry-javaagent.jar \
     -Dotel.service.name=my-spring-boot-app \
     -Dotel.exporter.otlp.endpoint=http://localhost:4317 \
     -jar app.jar
```

The agent picks up most instrumentation automatically, so you can see traces across controllers, repositories, and downstream HTTP calls without changing code.

💡

Know more on how distributed network monitoring complements tracing in our g[uide](https://last9.io/blog/distributed-network-monitoring-guide/).

### 2\. Configuring the OpenTelemetry SDK

For advanced setups, you can define OpenTelemetry directly in code. This approach is more flexible — you can customize exporters, samplers, and span processors to match your requirements.

**Example:**

```
import io.opentelemetry.api.OpenTelemetry;
import io.opentelemetry.sdk.OpenTelemetrySdk;
import io.opentelemetry.sdk.resources.Resource;
import io.opentelemetry.sdk.trace.SdkTracerProvider;
import io.opentelemetry.sdk.trace.export.BatchSpanProcessor;
import io.opentelemetry.exporter.otlp.trace.OtlpGrpcSpanExporter;
import io.opentelemetry.semconv.ResourceAttributes;

public class OpenTelemetryConfig {
  public static OpenTelemetry initOpenTelemetry() {
    Resource serviceResource = Resource.getDefault()
      .merge(Resource.builder()
        .put(ResourceAttributes.SERVICE_NAME, "my-spring-boot-app")
        .put(ResourceAttributes.SERVICE_VERSION, "1.0.0")
        .put(ResourceAttributes.DEPLOYMENT_ENVIRONMENT, "dev")
        .build());

    OtlpGrpcSpanExporter otlpExporter = OtlpGrpcSpanExporter.builder()
      .setEndpoint("http://localhost:4317")
      .build();

    SdkTracerProvider tracerProvider = SdkTracerProvider.builder()
      .addSpanProcessor(BatchSpanProcessor.builder(otlpExporter).build())
      .setResource(serviceResource)
      .build();

    OpenTelemetrySdk openTelemetry = OpenTelemetrySdk.builder()
      .setTracerProvider(tracerProvider)
      .buildAndRegisterGlobal();

    Runtime.getRuntime().addShutdownHook(new Thread(tracerProvider::close));
    return openTelemetry;
  }
}
```

In Spring Boot, you'd typically expose this as a `@Configuration` class and inject `OpenTelemetry` or `Tracer` beans where needed.

### 3\. Automatic Instrumentation with Micrometer Tracing

Spring Boot 3 includes first-class support for OpenTelemetry through [Micrometer Tracing](https://github.com/micrometer-metrics/tracing). It automatically captures traces for web requests, database queries, and messaging without any explicit agent configuration.

Add these dependencies to your `pom.xml` or `build.gradle`:

```
<dependency>
  <groupId>io.micrometer</groupId>
  <artifactId>micrometer-tracing-bridge-otel</artifactId>
</dependency>
<dependency>
  <groupId>io.micrometer</groupId>
  <artifactId>micrometer-tracing-reporter-otlp</artifactId>
</dependency>
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

This setup uses Micrometer as the bridge and sends traces to your configured OpenTelemetry backend (via OTLP).

### 4\. Manual Instrumentation for Custom Logic

Even with automatic tracing, you'll sometimes need manual spans to capture custom workflows or business logic.

**Example:**

```
import io.opentelemetry.api.trace.Span;
import io.opentelemetry.api.trace.Tracer;
import org.springframework.stereotype.Service;

@Service
public class ProductService {
  private final Tracer tracer;

  public ProductService(Tracer tracer) {
    this.tracer = tracer;
  }

  public Product getProductDetails(String productId) {
    Span span = tracer.spanBuilder("getProductDetails")
        .setAttribute("product.id", productId)
        .startSpan();
    try {
      // Simulate work
      Thread.sleep(50);
      return new Product(productId, "Example Product", 99.99);
    } catch (Exception e) {
      span.recordException(e);
      span.setStatus(io.opentelemetry.api.trace.StatusCode.ERROR, "Failed to fetch product details");
      throw e;
    } finally {
      span.end();
    }
  }
}
```

Manual spans are useful when you want to trace domain-specific operations — such as `checkout.process`, `payment.validate`, or `user.signup` — that automatic instrumentation can't infer.

Each approach — agent-based, SDK, Micrometer, or manual — serves a different need. Start with automatic instrumentation to get full coverage fast, and layer in manual spans or SDK-based configuration as your tracing requirements mature.

## Propagate Trace Context Across Services

A trace loses its value the moment it breaks between services. When trace context isn't passed along, you end up with fragmented spans that tell only part of the story. Maintaining context across services, threads, and queues ensures that every request can be tracked end-to-end.

### W3C Trace Context and B3 Propagation

OpenTelemetry follows open standards to maintain context across service boundaries. The most common is the [W3C Trace Context](https://www.w3.org/TR/trace-context/), which defines two key headers:

-   **traceparent** – carries the trace ID, parent span ID, and trace flags.
-   **tracestate** – includes vendor-specific or custom tracing data.

OpenTelemetry also supports B3 propagation, the format used by Zipkin, which relies on headers like `X-B3-TraceId`, `X-B3-SpanId`, and `X-B3-Sampled`.

Whichever format you choose, consistency matters. All services in your system should use the same propagation standard. For Spring Boot 3, OpenTelemetry defaults to W3C Trace Context — the recommended choice going forward.

### Ensure Context Flow Across REST APIs

In REST-based systems, OpenTelemetry automatically handles trace context for you.

**Incoming requests:** When your Spring Boot service receives an HTTP call, OpenTelemetry extracts the `traceparent` and `tracestate` headers and attaches that data to the current span.

**Outgoing requests:** When your service makes an outbound call using `RestTemplate` or `WebClient`, OpenTelemetry injects the same trace context into the outgoing request headers.

This keeps your trace continuous across multiple services, without manual setup.

### Handle Context in Async and Message-Based Systems

Asynchronous operations and message queues need extra attention because HTTP headers don't apply directly. OpenTelemetry provides ways to propagate context in these scenarios.

**@Async methods:** When you use Spring's `@Async`, the OpenTelemetry integration carries the current trace context within the same JVM automatically.

**Message queues (Kafka, RabbitMQ):** For inter-service messaging, inject trace context into the message headers before publishing and extract it when consuming.

**Send message example (Kafka):**

```
Span currentSpan = Span.current();
Context context = Context.current().with(currentSpan);
OpenTelemetry.getGlobalPropagators().getTextMapPropagator()
  .inject(context, recordHeaders,
    (headers, key, value) -> headers.add(key, value.getBytes()));
```

**Consume message example:**

```
Context extracted = OpenTelemetry.getGlobalPropagators().getTextMapPropagator()
  .extract(Context.current(), headers,
    (hdrs, key) -> {
      Header header = hdrs.lastHeader(key);
      return header != null ? new String(header.value()) : null;
    });

Tracer tracer = OpenTelemetry.getGlobalOpenTelemetry().getTracer("consumer-service");
Span consumerSpan = tracer.spanBuilder("processMessage")
  .setParent(extracted)
  .setSpanKind(SpanKind.CONSUMER)
  .startSpan();

try (Scope scope = consumerSpan.makeCurrent()) {
  // message processing logic
} finally {
  consumerSpan.end();
}
```

Proper propagation ensures your trace remains continuous — from HTTP calls to async jobs and message queues — giving you a full, accurate picture of how your distributed system behaves.

## Integrate OpenTelemetry with Spring Boot 3 Components

OpenTelemetry's real strength shows when it connects the dots between different parts of your Spring Boot 3 application — databases, message brokers, and web clients. Each layer gets its own visibility, forming a continuous picture of how requests move through your system.

### Database Interactions (JPA, JDBC)

The OpenTelemetry Java Agent automatically instruments most standard JDBC drivers. Every time your application runs a query or update, a new span is created for that database call.

Each span includes attributes like:

-   `db.system`: Database type (e.g., mysql, postgresql).
-   `db.connection_string`: Connection details (sanitized).
-   `db.statement`: SQL query text (ensure sensitive data is masked).
-   `db.user`: Database username.
-   `net.peer.name` / `net.peer.port`: Host and port information.

For JPA and [Hibernate](https://hibernate.org/), OpenTelemetry hooks into the underlying JDBC calls, so you get the same visibility without additional setup. This helps spot slow queries, inefficient joins, or N+1 problems early.

### Message Brokers (Kafka, RabbitMQ)

Message-driven systems depend heavily on context propagation, and OpenTelemetry handles that automatically for the most popular brokers.

**Kafka:** The Java Agent instruments the [kafka-clients](https://kafka.apache.org/) library. It injects trace context into `ProducerRecord` headers and extracts it from `ConsumerRecord` headers — creating `PRODUCER` and `CONSUMER` spans that connect message send and receive operations.

**RabbitMQ:** Similar instrumentation is available for [amqp-client](https://www.rabbitmq.com/), propagating trace context through message properties.

This automatic handling lets you trace an entire message lifecycle — from producer to consumer — without modifying business logic.

### Web Clients (RestTemplate, WebClient)

Spring Boot services often call other services over HTTP. OpenTelemetry instruments both `RestTemplate` and `WebClient` by default.

**RestTemplate:** When your app makes an HTTP request, OpenTelemetry injects the trace context headers into the outgoing call, ensuring the next service can continue the trace.

**WebClient:** In reactive applications, OpenTelemetry propagates the context through the reactive chain, maintaining trace continuity even in asynchronous flows.

With this setup, inter-service calls — synchronous or reactive — all appear as part of the same trace.

### Spring Security Context Integration

OpenTelemetry focuses on request and dependency tracing, but you can enrich spans with user context from [Spring Security](https://spring.io/projects/spring-security).

For example, you can add authenticated user information (like user ID or roles) to spans for better debugging and audit visibility.

```
import io.opentelemetry.api.trace.Span;
import org.springframework.security.core.Authentication;
import org.springframework.security.core.context.SecurityContextHolder;
import org.springframework.stereotype.Component;

@Component
public class UserTracingInterceptor {
  public void addUserDetailsToCurrentSpan() {
    Authentication auth = SecurityContextHolder.getContext().getAuthentication();
    if (auth != null && auth.isAuthenticated()) {
      Span.current().setAttribute("user.name", auth.getName());
      // Optionally add roles or user.id
    }
  }
}
```

You can invoke this from an interceptor, filter, or AOP aspect depending on your security flow. Adding security context helps trace how specific users interact with the system — useful for analyzing user impact or debugging access-related issues.

## Export Trace Data to Observability Backends

Once your Spring Boot application starts generating traces, the next step is exporting that data to an observability backend for storage, querying, and visualization. The OpenTelemetry ecosystem supports multiple export options, but most modern systems rely on the OpenTelemetry Protocol (OTLP) for consistent and efficient data transfer.

### Configure OTLP Exporters for Jaeger, Zipkin, or Prometheus

The [OpenTelemetry Protocol (OTLP)](https://opentelemetry.io/docs/specs/otlp/) is the standard for sending telemetry data — including traces, metrics, and logs — to external systems. It works over gRPC (port 4317 by default) or HTTP (port 4318 by default), offering flexibility across environments.

**Common backends:**

**Jaeger:** Run a Jaeger Collector that exposes an OTLP endpoint. The OpenTelemetry Java Agent or SDK can send trace data directly to this collector. Jaeger's UI provides detailed visualization and filtering of spans across services.

**Zipkin:** Zipkin can also receive OTLP data through its collector. It's lightweight and works well for smaller or local tracing setups.

**Prometheus / Tempo:** [Prometheus](https://prometheus.io/) is primarily for metrics, but you can pair it with Grafana Tempo to store and visualize traces. OpenTelemetry can export metrics via the Prometheus exporter and traces via OTLP to Tempo or another trace backend.

**Example configuration (Java Agent):**

```
# OTLP over gRPC
otel.exporter.otlp.endpoint=http://localhost:4317
otel.traces.exporter=otlp

# OTLP over HTTP
otel.exporter.otlp.traces.endpoint=http://localhost:4318/v1/traces
otel.traces.exporter=otlp
```

Using OTLP keeps your setup vendor-neutral and compatible with any backend that supports the standard.

### Use Managed Cloud Tracing Services

If you prefer a managed approach, most cloud providers offer native tracing services that work seamlessly with OpenTelemetry:

**AWS X-Ray:** Accepts OpenTelemetry traces via the [AWS Distro for OpenTelemetry (ADOT)](https://aws.amazon.com/otel/) collector.

**Google Cloud Trace:** Supports OTLP directly and integrates with other [Google Cloud](https://cloud.google.com/trace) observability tools.

**Azure Application Insights:** Offers OTLP ingestion endpoints, simplifying integration for Spring Boot apps deployed on [Azure](https://azure.microsoft.com/en-us/products/monitor).

Using managed services reduces the operational burden of maintaining collectors and storage while providing built-in correlation with other cloud metrics and logs.

### Address Data Security and Privacy

Trace data often contains context that can expose sensitive information. Protecting it should be part of your tracing setup.

Here are a few best practices you can follow:

**Mask sensitive data:** Avoid recording personal identifiers, credentials, or confidential payloads in span attributes. Use custom sanitization logic or filters to remove or redact such fields.

**Control access:** Restrict trace storage access to authorized users only. Configure role-based access controls in your observability backend.

**Encrypt in transit:** Always use HTTPS or TLS for all OTLP endpoints to secure trace data from the application to the backend.

**Define retention policies:** Set clear retention limits for trace data. Keep only what's necessary for troubleshooting and compliance.

## Best Practices for Distributed Tracing in Spring Boot 3

To make your tracing data useful, you need to follow a few key practices that keep it consistent, meaningful, and efficient.

**Consistent Naming Conventions for Spans**

Good span names make traces easier to read and reason about.

**Be descriptive:** Use names that clearly describe what the span represents — for example, `UserService.getUserById`, `OrderRepository.saveOrder`, or `processKafkaMessage`.

**Avoid high cardinality:** Don't include unique values like IDs or timestamps in span names. Use attributes instead.

**Follow semantic conventions:** Stick to [OpenTelemetry's naming patterns](https://opentelemetry.io/docs/specs/semconv/), such as `HTTP GET /users/{id}` or `SELECT FROM users`. These conventions make spans uniform and easier to aggregate or query later.

Consistent naming helps you quickly spot what part of a request is slow or failing without guessing what a span represents.

**Add Relevant Attributes for Context**

Attributes enrich spans with metadata that provides both technical and business context.

**Business context:** Include values like `customer.id`, `order.id`, or `tenant.id` to connect technical events with user impact.

**Technical context:** Capture details such as `db.system`, `http.status_code`, or `message.queue_name`.

**Error details:** Add `error.type`, `error.message`, or `exception.stacktrace` when failures occur.

**Resource attributes:** Keep `service.name`, `service.version`, and `host.name` consistent across all spans.

Well-chosen attributes turn raw traces into something you can actually use during incident analysis.

**Manage Sampling Strategically**

Collecting every single trace can overwhelm your backend in high-throughput environments. Sampling lets you control data volume while still capturing the most valuable traces.

**Head sampling:** Makes the decision when a request starts. If the first span is sampled, all child spans are collected.

**Parent-based sampling:** Inherits decisions from the parent span but can override locally.

**Trace ID ratio-based sampling:** Captures a fixed percentage of traces (e.g., 1–5%).

**Tail sampling:** Decides after the trace completes — often in the OpenTelemetry Collector — based on outcomes like errors or latency.

For most Spring Boot 3 services, a `ParentBasedSampler` combined with a low ratio works well. You can always sample all error traces to ensure failures are captured.

**Use Trace Data for Monitoring and Alerting**

Traces aren't just for debugging; they're a rich source of operational signals.

**Error rate:** Alert when too many traces contain error spans.

**Latency anomalies:** Track response time for key endpoints and trigger alerts when they exceed normal thresholds.

**SLOs:** Define performance objectives (e.g., 99th percentile latency for checkout requests) based on trace data.

**Service dependencies:** Use trace graphs to identify failing downstream services early.

When traces power your monitoring, you can detect slowdowns or dependency failures before they reach users.

**Minimize Performance Overhead**

Tracing inevitably adds some cost, but it can be managed effectively.

**CPU and memory:** The OpenTelemetry Java Agent is lightweight, but complex manual spans can add overhead.

**Network I/O:** Use batching and OTLP gRPC to reduce export traffic.

**Sampling:** The simplest and most effective way to control cost.

**Resource cleanup:** Always close `TracerProvider` or SDK resources on shutdown to prevent leaks.

Monitor your system with tracing enabled, and adjust your sampling ratio or instrumentation depth based on real-world performance.

Applying these practices ensures that tracing in your Spring Boot 3 environment delivers useful, actionable data — without creating noise or unnecessary load.

💡

Fix distributed tracing issues in your Spring Boot applications faster with AI and [Last9 MCP](https://last9.io/mcp/) — bring live production context like traces, metrics, and logs into your IDE to debug and resolve problems instantly.

## Troubleshoot Common Distributed Tracing Issues

Distributed tracing in Spring Boot can fail for specific, diagnosable reasons. The following checks cover the most common technical causes of missing spans, broken context, and tracing overhead.

### Missing Spans or Incomplete Traces

If parts of a request aren't visible in your backend:

**Agent not loaded:** Run the application with `-javaagent:/path/to/opentelemetry-javaagent.jar`. Check the startup logs — if you don't see `otel.javaagent`, it's not attached.

**Unsupported libraries:** Confirm your dependencies appear in the [OpenTelemetry Java Instrumentation supported library list](https://github.com/open-telemetry/opentelemetry-java-instrumentation/blob/main/docs/supported-libraries.md). For unsupported frameworks, use manual instrumentation via `Tracer.spanBuilder()`.

**Async boundaries:** For `@Async` methods or message consumers, wrap tasks using `Context.current().wrap()` or configure `ExecutorService` with `Context.taskWrappingExecutor()`.

**Sampling too low:** In `otel-config.properties`, verify `otel.traces.sampler=parentbased_traceidratio` and `otel.traces.sampler.arg=0.05` (for 5% sampling).

**Exporter not connected:** Check agent logs for lines like `Failed to export spans`. Test OTLP collector reachability:

```
curl -v http://localhost:4318/v1/traces
```

**Context lost:** If spans start new trace IDs instead of continuing the same trace, inspect request headers. Ensure `traceparent` and `tracestate` headers exist on all inbound and outbound requests. Example check using curl:

```
curl -v http://service-b:8080/api --header "traceparent: 00-<trace-id>-<span-id>-01"
```

### Context Propagation Failures

Trace continuity depends on the consistent exchange of context across services and threads.

**Cross-language calls:** Verify the propagation format. Set it explicitly if mixed environments exist:

```
otel.propagators=tracecontext,baggage
```

**Kafka/RabbitMQ:** Ensure producers and consumers use the same propagator (W3C or B3). Example for Kafka headers:

```
OpenTelemetry.getGlobalPropagators().getTextMapPropagator()
  .inject(Context.current(), record.headers(),
    (headers, key, value) -> headers.add(key, value.getBytes(StandardCharsets.UTF_8)));
```

**Thread pools:** Wrap runnables to maintain Context:

```
executor.submit(Context.current().wrap(() -> doWork()));
```

**Custom HTTP clients:** Manually inject and extract headers:

```
propagator.inject(Context.current(), request, (req, key, value) -> req.addHeader(key, value));
```

and

```
Context extracted = propagator.extract(Context.current(), response, (resp, key) -> resp.getHeader(key));
```

Tracing introduces measurable overhead. Monitor and tune using specific metrics and configurations.

**CPU usage:** Profile span creation cost with `otel.javaagent.debug=true`. Look for `SpanProcessor.onStart` time in logs.

**Exporter queue latency:** Check collector queue size (`otel.exporter.otlp.metrics.default.histogram.aggregation.temporality`) and exporter thread pool metrics.

**High-cardinality attributes:** Avoid tags like `user.id` or `request.uuid`. If needed, hash or normalize values before attaching.

**GC pressure:** Excessive short-lived spans can increase GC time. Use asynchronous span processors and batch exporters to minimize allocation churn.

**Tail sampling:** If using collector-based tail sampling, ensure rules don't filter too aggressively. Sample on error or latency attributes:

```
processors:
  tail_sampling:
    policies:
      - name: error-traces
        type: status_code
        status_code: ERROR
```

**Batching:** Adjust batch size to control throughput:

```
otel.bsp.schedule.delay=5000
otel.bsp.max.queue.size=2048
otel.bsp.max.export.batch.size=512
```

_Here's a quick checklist you can perform:_

1.  Confirm agent is attached and logs contain "OpenTelemetry Java Agent started."
2.  Validate `traceparent` headers exist across all HTTP requests.
3.  Run with `-Dotel.logs.level=debug` to inspect exporter output.
4.  Verify OTLP collector reachability and backend ingestion.
5.  Use `ParentBasedSampler` with a ratio that captures key traces without saturating storage.

## Final Thoughts

Start small — instrument the services that define your user journey, export traces to your backend, and observe how the data behaves under load.

Monitor system metrics — CPU, memory, network I/O, and GC activity — alongside tracing volume. As you build confidence in your setup, gradually increase sampling rates or extend instrumentation deeper into your stack.

At [Last9](https://last9.io/), we've seen how teams struggle with trace explosion, storage limits, and cost unpredictability. Our platform is built to handle high-cardinality telemetry without sacrificing performance or visibility. You can ingest and query millions of spans, correlate them with metrics and logs, and still maintain predictable costs.

[Start for free today](https://app.last9.io/?utm_source=springboot_distributed_tracing), or [book some time with us](https://last9.io/schedule-demo/?utm_source=springboot_distributed_tracing) to see how Last9 can make distributed tracing production-ready for your Spring Boot environments.

## FAQs

**How to Create a REST API using Java Spring Boot?**  
Create a Spring Boot project using _Spring Initializr_, add `spring-boot-starter-web`, and annotate your controller methods:

```
@RestController
@RequestMapping("/api/users")
class UserController {
  @GetMapping("/{id}") User get(@PathVariable Long id) { /* ... */ }
  @PostMapping User create(@RequestBody User u) { /* ... */ }
}
```

Run with `@SpringBootApplication` using `mvn spring-boot:run` or `java -jar`.

**How to Write Test Cases in a Java Application using Mockito and JUnit?**  
Use JUnit 5 and Mockito for unit testing:

```
@ExtendWith(MockitoExtension.class)
class UserServiceTest {
  @Mock UserRepo repo;
  @InjectMocks UserService svc;

  @Test void findsById() {
    when(repo.findById(1L)).thenReturn(Optional.of(new User(1L,"A")));
    var u = svc.get(1L);
    assertEquals("A", u.name());
    verify(repo).findById(1L);
  }
}
```

**Is there any GitHub repo link for this?**

-   Spring Boot REST examples: [`spring-projects/spring-boot`](https://github.com/spring-projects/spring-boot)
-   Reference project: [`spring-projects/spring-petclinic`](https://github.com/spring-projects/spring-petclinic)
-   OpenTelemetry instrumentation samples: [`open-telemetry/opentelemetry-java-instrumentation`](https://github.com/open-telemetry/opentelemetry-java-instrumentation/tree/main/examples)

**How to implement distributed tracing with the Java agent and a Spring Boot app?**  
Attach the OpenTelemetry Java agent at runtime:

```
java -javaagent:/path/opentelemetry-javaagent.jar \
     -Dotel.service.name=orders-api \
     -Dotel.exporter.otlp.endpoint=http://otel-collector:4317 \
     -jar app.jar
```

The agent auto-instruments Spring MVC/WebFlux, JDBC, HTTP clients, Kafka, and RabbitMQ, and propagates W3C `traceparent` headers.

**Is there a way to propagate for LDAP or Kafka?**

-   **Kafka:** The agent supports `kafka-clients` out of the box, injecting trace context into `ProducerRecord` headers.
-   **LDAP:** Typically not auto-instrumented. Use manual spans for each LDAP query and include contextual attributes like `ldap.query` or `ldap.endpoint`.

**What is Spring Data JPA?**  
A Spring module that abstracts JPA using repositories like `JpaRepository` or `CrudRepository`. It generates SQL queries based on method names or annotated JPQL.

**How do I set up distributed tracing in a Spring Boot application using OpenTelemetry?**  
Option 1 — Java Agent (no code changes): attach the agent and configure OTLP exporter.  
Option 2 — Micrometer Tracing (code-based setup):

```
<dependency> <groupId>io.micrometer</groupId><artifactId>micrometer-tracing-bridge-otel</artifactId></dependency>
<dependency> <groupId>io.micrometer</groupId><artifactId>micrometer-tracing-reporter-otlp</artifactId></dependency>
<dependency> <groupId>org.springframework.boot</groupId><artifactId>spring-boot-starter-actuator</artifactId></dependency>
```

Set `management.otlp.tracing.endpoint` to your collector.

**How do I enable distributed tracing in a Spring Boot application?**  
Ensure your application sends traces over OTLP (`4317` gRPC or `4318` HTTP). Validate that trace context (`traceparent`, `tracestate`) propagates across services, and uses manual spans for critical operations. Backends like Jaeger, Tempo, Datadog, or Last9 visualize and correlate your traces with metrics and logs for complete observability.

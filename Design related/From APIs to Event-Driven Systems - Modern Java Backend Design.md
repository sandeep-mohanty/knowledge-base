# From APIs to Event-Driven Systems: Modern Java Backend Design

The outage happened during our biggest sales event of the year. Our order processing system ground to a halt. Customers could add items to their carts, but checkout failed repeatedly. The engineering team scrambled to check the logs. We found a chain of synchronous REST API calls that had collapsed under load. Service A called Service B, which called Service C. When Service C slowed down due to database locks, the latency rippled back up the chain. Service A timed out. Service B timed out. The entire order pipeline froze. We were losing revenue by the minute. This incident forced us to rethink our architecture. We realized that synchronous APIs were not suitable for every interaction. We needed to decouple our services. We needed an event-driven system.

In this article, I will share how we migrated from a tightly coupled API architecture to an event-driven design using Java and Kafka. I will explain the specific challenges we faced during the transition. I will detail the code changes required to handle asynchronous communication. This is not a theoretical discussion about microservices. It is a record of the practical steps we took to stabilize our platform. Building resilient backend systems requires more than just choosing the right tools. It requires understanding the trade-offs between consistency and availability.

## **The Synchronous Trap**

Our initial design followed standard [REST principles](https://dzone.com/articles/level-up-api-design-rest-principles). Each microservice exposed endpoints for other services to call. This worked well for simple read operations. It failed for complex workflows involving multiple domains. An order creation process involved inventory management, payment processing, and notification services. Each step depended on the previous one completing successfully.

The problem was latency accumulation. If each service added 50 milliseconds of latency, the total request time grew quickly. Under high load, the network overhead increased. Database connections became scarce. Threads blocked waiting for responses. The thread pools exhausted rapidly. The system entered a death spiral where retries made the congestion worse. We needed to break these dependencies.

## **The Event-Driven Shift**

We decided to introduce [Apache Kafka](https://dzone.com/refcardz/apache-kafka) as our event backbone. Services would no longer call each other directly. Instead, they would publish events when the state changed. Other services would subscribe to these events and react independently. This decoupled the producer from the consumer. The order service could publish an OrderCreated event and return success immediately. The inventory service would consume the event and reserve stock asynchronously. The payment service would consume the event and process charges independently.

This change improved resilience significantly. If the inventory service went down, the order service continued to accept orders. The events were queued in Kafka until the inventory service recovered. We eliminated the cascading failure scenario. The system could absorb spikes in traffic without collapsing.

## **Implementation Details in Java**

We used Spring Boot with Spring Cloud Stream for integration. This abstracts much of the Kafka boilerplate. We defined input and output channels for each service. The code became declarative rather than imperative.

Here is how we structured the event producer in the Order Service.

![How the event producer is structured in the Order Service](https://dz2cdn1.dzone.com/storage/temp/18925533-1772850658114.png)

The consumer logic in the Inventory Service looked like this.

![Consumer logic in the Inventory Service](https://dz2cdn1.dzone.com/storage/temp/18933868-1773258212031.png)

This simple pattern replaced complex REST client code. We removed retry logic from the application layer because Kafka handled redelivery. We removed circuit breakers for inter-service communication because services were no longer directly coupled. The architecture became simpler despite the added infrastructure.

## **Handling Duplicate Events**

[Event-driven systems](https://dzone.com/articles/reliable-event-driven-architecture-patterns) introduce new challenges. At-least-once delivery is the default for Kafka. This means consumers might receive the same event multiple times. Our initial implementation was not idempotent. We processed duplicate events and reserved stock twice. This caused data inconsistencies. Inventory counts became negative.

We fixed this by implementing idempotency checks. Each event carried a unique correlation ID. The consumer stored processed IDs in a database table. Before processing an event, the consumer checked this table. If the ID existed, we skipped the processing.

![Handling duplicate events](https://dz2cdn1.dzone.com/storage/temp/18933869-1773258251584.png)

This ensured each order was processed exactly once from a business logic perspective. The overhead of the database check was minimal compared to the risk of data corruption. We learned that eventual consistency requires careful handling of state.

## **Schema Evolution and Compatibility**

Another challenge was managing event schemas. Services evolved independently. The Order Service might add a new field to the event. The Inventory Service might not expect this field. We used Apache Avro with Schema Registry to manage this. It enforced compatibility rules.

We configured the registry to allow backward-compatible changes. Adding a new optional field was safe. Removing a field required a deprecation period. This prevented breaking changes from reaching production. We treated event contracts as public APIs. Changing them required coordination between teams. This discipline prevented silent failures where consumers ignored new data.

## **Observability in Distributed Flows**

Debugging event-driven systems is harder than debugging REST APIs. A request does not follow a single path. It branches into multiple consumers. Tracing a single order required correlating events across services. We implemented distributed tracing using OpenTelemetry.

We propagated trace IDs in the event headers. Each consumer continued the trace span. This allowed us to visualize the full flow in Grafana Tempo. We could see how long each service took to process the event. We could identify slow consumers that lagged behind. This visibility was crucial for maintaining performance SLAs.

We also monitored consumer lag metrics. Kafka exposes the difference between the latest offset and the committed offset. High lag indicated a slow consumer. We set alerts on this metric. If lag exceeded a threshold, the on-call team received a notification. This allowed us to scale consumers before users noticed delays.

## **When Not to Use Events**

Event-driven architecture is not a silver bullet. We learned this the hard way. We initially tried to use events for user login authentication. This failed because the login requires immediate feedback. The user needs to know instantly if the password is correct. Events introduce latency. They are asynchronous by nature.

We reserved events for background processes and data propagation. Order fulfillment and notification sending were perfect use cases. User authentication and real-time balance checks remained synchronous. We used REST APIs for request-response interactions. We used Kafka for state changes and workflows. Understanding this distinction was key to our success.

## **Lessons Learned and Best Practices**

Our migration taught us several valuable lessons. We incorporated these into our development standards.

1.  **Design for failure**: Assume consumers will fail. Ensure events can be replayed. Store events in a durable log.
2.  **Monitor lag**: Consumer lag is the most important metric. It indicates system health better than CPU usage.
3.  **Version events**: Plan for schema changes from day one. Use a registry to enforce compatibility.
4.  **Test integration**: Unit tests are not enough. Test the full event flow in staging. Verify that consumers handle duplicates correctly.
5.  **Keep events small**: Large events slow down processing. Include only necessary data. Reference large payloads via ID if needed.
6.  **Secure topics**: Restrict access to Kafka topics. Use ACLs to prevent unauthorized publishing or consuming.
7.  **Document flows**: Event flows are invisible. Document which service produces and consumes each event type.

## **Conclusion**

Moving from APIs to event-driven systems was a significant undertaking. It required changes in code and mindset. We stopped thinking in terms of requests and responses. We started thinking in terms of state changes and reactions. The result was a more resilient and scalable platform. Our order processing system now handles peak loads without downtime. Services can fail without bringing down the entire system.

Java provides robust tools for building these systems. Spring Cloud Stream and Kafka integrate seamlessly. The ecosystem is mature and well supported. However, complexity increases with decoupling. Teams must invest in observability and testing. The benefits outweigh the costs for high-scale applications. We continue to refine our architecture. We are exploring event sourcing for critical domains. The journey from synchronous to asynchronous is ongoing. Happy building, and keep your systems decoupled.

Opinions expressed by DZone contributors are their own.
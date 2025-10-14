# Understanding Zanzibar: Google's consistent authorization system

Ever wondered how Google handles millions of authorization (AuthZ) queries per second involving billions of users for its products? Well the answer seems to be [Zanzibar](https://research.google/pubs/zanzibar-googles-consistent-global-authorization-system/) which handles trillions of access control lists (ACLs) while keeping latency in tens of milliseconds and an astounding 99.999% of availability! It also offers interoperability between different Google products to coordinate access control, say for example you are sending a Google document to an user, Zanzibar can warn the recipient that it doesn't have access over it.

Zanzibar was developed internally by Google to handle growing queries and to tackle issues like the "new enemy problem" which seems to have first mentioned in the paper which was published in 2019.

In this article we'll dive into the architecture of Zanzibar including its APIs, aclservers, data management techniques and take a look at its key features including the Zanzibar data model for authorization and [ReBAC](https://en.wikipedia.org/wiki/Relationship-based_access_control) (Relationship-Based Access Control).

## [](#core-architecture)Core Architecture

[![Zanzibar Architecture](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fltlvekz7r4bef9w4iers.png)](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fltlvekz7r4bef9w4iers.png)

Zanzibar’s design prioritizes scalability, consistency, and interoperability. This idea that we start from a globally consistent datastore and then relax consistency for performance is _key_ to understanding Zanzibar.

### [](#clientserver-model)Client-Server Model

Clients (e.g., Gmail, Drive) communicate with Zanzibar’s **ACL servers** via read/write APIs. To optimize performance, frequently accessed ACLs are cached in memory using **consistent hashing**, which distributes load evenly across servers and allows dynamic scaling (adding/removing nodes without downtime). Less frequently used data is stored durably. This ensures fast access for hot data while maintaining durability for cold data, also maintaining low latency at the same time. The client requests for a **zookie** for each request, which contains encoded global timestamp, used by the client for subsequent ACL writes to ensure consistency via a point in time

### [](#data-storage)Data Storage

Zanzibar uses Spanner as a datastore, to store namespaces, relation tuples and appends the change in ACLs to changelog which is utilised in the watchserver for streaming changes for a ACL, and also other cron jobs which we will discuss later on in this article.

* * *

## [](#timestamped-requests)Timestamped Requests

Every request (read or write) includes a timestamp, enabling Zanzibar to enforce **causal consistency**. For example, when a user checks access to a document and then updates its ACLs, the timestamps ensure these operations occur in the correct global order. This prevents scenarios where stale data might incorrectly authorize or deny access. By introducing **zookie** this system also solves the _new enemy problem_.

* * *

## [](#data-model-rebac-and-namespaces)Data Model: ReBAC and Namespaces

Zanzibar uses **Relationship-Based Access Control (ReBAC)**, modeling permissions as a directed graph. In this graph, **nodes** represent entities like users, documents, or groups whereas, **edges** define relationships (e.g., `Doc#edit@team` indicates a team’s edit permission on a document).

To avoid naming collisions, Zanzibar introduces **namespaces** to scope entities. For example:

`Docs:secret_project #edit@org:dev_team`

Here, `Docs` and `org` are namespaces that isolate the document and team identifiers, ensuring uniqueness across services like Drive and Calendar.

* * *

## [](#solving-the-new-enemy-problem)Solving the "New Enemy Problem"

The **"new enemy problem"** occurs when ACL updates are applied out of order, leading to unintended access. Zanzibar solves this with a two-phase mechanism:

1.  **Version Cookies**: When a client checks a permission, Zanzibar returns a "zookie" containing a globally encoded timestamp.
2.  **Write Validation**: The client includes this cookie in subsequent write requests (e.g., updating ACLs). Zanzibar validates the cookie’s timestamp to ensure no conflicting changes occurred between the read and write.

This ensures linearizable consistency—guaranteeing that updates reflect the latest state without sacrificing low latency.

* * *

## [](#data-management-and-performance)Data Management and Performance

### [](#sharding-and-caching)Sharding and Caching

ACL data is sharded by **object ID** (e.g., document ID) because reads vastly outnumber writes. Hot data (e.g., frequently accessed documents) is cached in memory, while cold data resides in Spanner. This design optimizes for read-heavy workloads, which dominate authorization systems.

### [](#watch-server-for-realtime-updates)Watch Server for Real-Time Updates

Zanzibar’s **Watch Server** streams ACL changes to clients in real time. For instance, revoking access to a document triggers immediate notifications to subscribed services, ensuring consistent enforcement across Google’s ecosystem.

### [](#offline-pipelines-and-leopard-indexing)Offline Pipelines and Leopard Indexing

Complex queries (e.g., “share a document with users in Delhi with 10+ years of experience”) require expensive set operations which may spike latency if done in-memory. To avoid latency spikes, Zanzibar uses **offline pipelines** to precompute these relationships, making them ready to serve. **Leopard Indexing system** takes snapshots of the ACL graph, precomputes groups or user sets, and pushes them to ACL servers. The pipeline runs periodically, balancing data freshness with computational overhead in a cron-like behaviour.

Also to mention much about the _Leopard Indexing system_ is left unanswered in the paper, leaving room for speculations.

* * *

## [](#why-zanzibar-matters)Why Zanzibar Matters

Zanzibar’s architecture offers critical lessons for distributed systems:

-   **Consistency vs. Latency**: Global timestamps and version cookies enable strong consistency without compromising speed.
-   **Interoperability**: A unified ACL model allows seamless permission checks across services (e.g., Drive links in Gmail).
-   **Efficiency**: Techniques like sharding, caching, and offline precomputation optimize performance for read-heavy workloads.

While few systems operate at Google’s scale, Zanzibar’s principles—consistent hashing, precomputation, and real-time streaming—are broadly applicable.

* * *

## [](#final-notes)Final Notes

Zanzibar exemplifies Google’s approach to infrastructure challenges: combining distributed databases (Spanner), real-time streaming (Watch Server), and batch processing (offline pipelines) to achieve reliability at scale. For developers, it underscores the importance of thoughtful data modeling and consistency mechanisms in authorization systems.
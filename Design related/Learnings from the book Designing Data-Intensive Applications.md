# Learnings from the book Designing Data-Intensive Applications?

After two decades in software engineering, I thought I had a solid understanding of various topics, including NoSQL, Big Data, transactions, sharding, and more.

However, reading [Martin Kleppmann’s](https://martin.kleppmann.com/) _[Designing Data-Intensive Applications](https://amzn.to/3ZX4uMv)_ (DDIA) was revelatory for me, as I finally understood the concepts behind these technologies and systems.

This (still) popular book (often called the “_Big Ideas Behind Reliable, Scalable, and Maintainable Systems_”) bridges theory and practice to explain **how data systems work and why**.

In this article, we will cover the following:

1.  **Introduction**. Explains why “Designing Data-Intensive Applications” matters and how rereading it clarified its core ideas to me.
    
2.  **The things I liked about the book**. In this section, I show the book’s clear breakdown of reliability, scalability, maintainability, data models, and storage engines, as well as the importance of weighing trade-offs.
    
3.  **The things I didn’t like**. Here we note gaps in the book, such as outdated examples, theory-heavy coverage, and the breadth-over-depth trade-off that can overwhelm readers.
    
4.  **Recommendation**. Identifies who will gain the most (mid-career engineers, architects, tech leads) and who may struggle (new devs, theory-averse readers).
    
5.  **Conclusion.** Here we summarize the mental models and decision frameworks you gained, positioning DDIA as a must-read reference for designing reliable data systems.
    
6.  **Bonus: Key takeaways & principles**. Finally, we made DDIA into a quick-hit list of design rules and trade-offs you can reference during architecture and code reviews.
    

So, let’s dive in.

This is one of the books everyone will say is a great read, but often behind that, there is a wall of silence. I have always wondered whether people really read the book or didn’t understand it well.

I started reading it for the first time in 2018. And I was almost finished, but some parts were tricky to grasp. Then, in 2023. I decided to re-read it properly and take notes. This text is primarily based on the notes I took at the time (check them in the reference section).

DIA is not just another tech book; it’s essentially a **foundational guide to data systems**. Kleppmann begins by reminding us what matters in the world of distributed systems: building applications that are **reliable**, **scalable**, and **maintainable** for the long run.

The book then explores different types of databases, distributed systems, and data processing to help you understand their strengths, weaknesses, and trade-offs.

As I read, I often found myself nodding along and saying, _“Ah, that’s why this design is the way it is!”_ Each chapter presents some significant concepts, from data models and storage engines to replication and stream processing.

By the end, I not only had refreshed knowledge on things I use daily (like **SQL vs NoSQL databases** or **Apache Kafka**), but I also gained a more principled way of thinking about distributed systems.

Each of these subsections highlights what resonated with me the most.

One thing I appreciated immediately was that the book **starts with fundamentals**. It defines three critical concerns for any system: **reliability**, **scalability**, and **maintainability**.

1.  🔒 **Reliability** means your system continues to work correctly even when things go wrong (hardware fails, bugs occur, humans err).
    
2.  **📈 Scalability** refers to a system's ability to handle increased loads efficiently and effectively.
    
3.  **🛠️ Maintainability** refers to the system's ease of _management_ and evolution by engineers over time. All of these are designed from the start.
    

[

![](https://substackcdn.com/image/fetch/$s_!WfMt!,w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fbc784e06-84e5-446b-b583-f6b3c3ad9f76_1417x852.png)

](https://substackcdn.com/image/fetch/$s_!WfMt!,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fbc784e06-84e5-446b-b583-f6b3c3ad9f76_1417x852.png)

Distributed Systems Concerns

Kleppmann also breaks down **maintainability** into the following design principles:

-   **🛠️ Operability**. Make life easier for Ops teams with effective monitoring and automation.
    
-   **✂️ Simplicity**. Tame complexity by avoiding accidental complexity.
    
-   **🌱 Evolvability**. Make it easy to adapt the system to new requirements.
    

This was a great reminder that **“building for change”** is just as crucial as handling today’s traffic.

I also found the discussion on **performance metrics** useful. Instead of focusing on average latency, the book explains why we should care about **percentiles**, such as the median (p50), 95th, and 99th percentile response times, to understand _tail latency_ and worst-case user experience.

For example, if your 99th percentile latency is 2 seconds, 1 in 100 users are waiting _at least_ 2 seconds (even if the average is low). This emphasis on distribution (not just “average”) and techniques like using **rolling percentiles in monitoring** made me reconsider how we measure and talk about performance.

[

![](https://substackcdn.com/image/fetch/$s_!0Gy1!,w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F97ff63b7-80d7-4a37-91ab-11de8131024d_1426x993.png)

](https://substackcdn.com/image/fetch/$s_!0Gy1!,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F97ff63b7-80d7-4a37-91ab-11de8131024d_1426x993.png)

Response times for a sample of 100 requests to a service (approx., based on the book Figure 1-4)

Finally, a minor but essential lesson: **the book constantly highlights** _**trade-offs**_. There’s no free lunch – every design decision (say, a cache for speed or a schema for data quality) has downsides. By keeping reliability, scalability, and maintainability goals in mind, you can reason about these trade-offs more clearly.

> ➡️ _This mindset of **evaluating trade-offs** is probably the most significant meta-learning I gained from the DDIA book._

As someone who’s worked with both SQL and NoSQL databases, I found DDIA’s treatment of data models both a refresher and an eye-opener. It compares the traditional **relational model** with the newer **document** and **graph** models in a very balanced way.

The takeaway? **Use a data model that aligns with your data access patterns.** For example, relational databases are helpful with **complex queries and many-to-many relationships** (thanks to joins and normalized schemas).

If your data is highly interconnected (think social networks), a **graph database** is a natural fit and can simplify those traversals.

On the other hand, if your data is self-contained, primarily consisting of documents (such as user profiles or blog posts with comments), a **document database** might be more convenient.

Document databases offer **schema flexibility and load whole records efficiently**, which can make **reads faster for document-shaped data**. It was interesting to learn that if your app typically loads an entire document (e.g., a user profile with all its nested info) at once, a document store can eliminate the join overhead and be more performant.

An example of one MongoDB document:

Here are the most used data models and their respective database types:

-   **📄 Document databases** (e.g., MongoDB, CouchDB) lack join capabilities, so they _struggle with many-to-many data_, so you might end up doing those joins at the application level (complex).
    
-   **🗄️ Relational databases** have schemas (schema-on-write), which provide consistency, but that rigidity led to the rise of **NoSQL** when developers wanted more agile schemas. DDIA discusses the concept of **impedance mismatch**, which refers to the awkward translation between objects in application code and tables in an SQL database. Many developers, including myself, have felt this pain, and it’s why **Object-Relational Mappers (ORMs)** exist. The document model (storing JSON, etc.) can reduce this mismatch since the stored data resembles in-memory structures more closely. But again, trade-offs: schema flexibility can turn into “schema _chaos_” if you’re not careful with data quality.
    
-   🕸️ The book also explores less common models, such as **Graph databases** (E.g., [Neo4j](https://neo4j.com/) and [Titan](http://espeed.github.io/titandb/)), and explains when they’re helpful (if many-to-many relationships are common). Facebook, for example, maintains a single graph with many different types of vertices and edges. Their vertices represent people, locations, events, check-ins, and comments made by users, while edges indicate which people are friends.
    

In summary, _Designing Data-Intensive Applications_ provided me with proper reasoning about database types: **choose your database not based on hype, but rather on how your application uses the data**.

This means that if you require ACID transactions and numerous complex joins, **relational databases** remain a reliable default. If you need flexible schemas or high write throughput with eventual consistency, a document or **key-value store** may be a more suitable option. If you have complex relationships, a **graph model** could save a lot of code.

Here is the comparison table:

[

![](https://substackcdn.com/image/fetch/$s_!iQia!,w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F6fe372a9-c9b4-4d1c-834a-0e37ca8e81b3_1407x989.png)

](https://substackcdn.com/image/fetch/$s_!iQia!,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F6fe372a9-c9b4-4d1c-834a-0e37ca8e81b3_1407x989.png)

Hearing the pros and cons in one place, with examples, was helpful. (As an aside, the book notes how modern systems are converging: e.g., SQL databases now offer JSON columns, and some NoSQL databases offer SQL-like querying.)

The image below shows the current types of databases:

[

![](https://substackcdn.com/image/fetch/$s_!Qiui!,w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F39f46b3b-d18c-4daf-9900-4ad729ccf037_1014x928.png)

](https://substackcdn.com/image/fetch/$s_!Qiui!,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F39f46b3b-d18c-4daf-9900-4ad729ccf037_1014x928.png)

Types of Databases

One of my favorite learnings was **how databases store and index data internally**. If you’ve ever wondered why _Cassandra_ or _RocksDB_ behave differently from _PostgreSQL_, the book’s explanation of storage engines is gold.

It contrasts the two main approaches to indexing: **B-tree indexes** (used by most relational databases) versus **Log-Structured Merge-trees (LSM-trees)**, which are used by many modern NoSQL databases.

**B-trees** store data in fixed-size blocks (pages) and keep those pages in a sorted tree structure on disk. They are **optimized for read performance**, lookups, and range scans are fast because the tree is balanced and shallow.

Most traditional databases (such as [SQL Server](https://www.red-gate.com/simple-talk/databases/sql-server/database-administration-sql-server/sql-server-storage-internals-101/), Oracle, MySQL/InnoDB, and PostgreSQL) use B-tree indexes for this reason. However, **writes** to B-trees can be slower because inserting a new record may involve multiple disk writes (for the data and updating parent index pages), and small random writes can be I/O-intensive.

> **➡️** _**[SQLite](https://sqlite.org/)**, for example, [includes B-trees for each table and index in the database](https://jvns.ca/blog/2014/10/02/how-does-sqlite-work-part-2-btrees/). For indexes, the key saved on a page is the index's column value, and the value is the row ID where it may be found. For the table B-tree, the key is the row ID, and I believe the value is all the data in that row._

**LSM-trees**, on the other hand, are optimized for **high write throughput**. They buffer writes in memory and always **append** data to disk in bulk, never in-place. Data is stored in sorted order in files ([SSTables](https://www.scylladb.com/glossary/sstable/) \- Sorted String Table format), and background processes **merge** and compact these files as needed.

This sequential write pattern makes **LSM-based storage engines extremely fast for writes** (lower disk seek overhead and high sequential write throughput). The trade-off is that reads can be slower, since a key’s data might be spread across multiple files that haven’t been merged yet. LSM-based systems mitigate this with auxiliary structures like **[Bloom filters](https://en.wikipedia.org/wiki/Bloom_filter)** (to skip files that don’t contain a key quickly).

The book notes a simple rule of thumb: _“B-trees enable faster reads, whereas LSM-trees enable faster writes.”_

The image below illustrates the differences between B-Trees and LSM-Trees, along with the database engines that utilize them.

[

![](https://substackcdn.com/image/fetch/$s_!PCBo!,w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F523aaf9f-2d01-480b-b1a5-dc94bc932e10_1520x1543.png)

](https://substackcdn.com/image/fetch/$s_!PCBo!,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F523aaf9f-2d01-480b-b1a5-dc94bc932e10_1520x1543.png)

**B-Tree vs. LSM-Tree**: B-trees (used in MySQL, PostgreSQL, etc.) favor quick reads with in-place updates, while LSM-trees (used in Cassandra, RocksDB, etc.) favor fast sequential writes and background compaction

This was interesting because it explains why something like **[Apache Cassandra](https://cassandra.apache.org)** chooses an LSM-tree architecture. Cassandra’s storage engine is based on log-structured merges. It writes to an in-memory table and an append-only log, then periodically flushes sorted data to disk and compacts it in the background.

This design achieves excellent write performance on commodity hardware, as Cassandra emphasizes, at the cost of read amplification (reads must check multiple SSTable files); hence, **[Cassandra utilizes Bloom filters](https://cassandra.apache.org/doc/latest/cassandra/architecture/storage-engine.html)** and data summaries to maintain fast reads ([CockroachDB does similar](https://www.cockroachlabs.com/docs/stable/architecture/storage-layer)).

> ➡️ **What are Bloom filters?** _A Bloom filter is a compact, **probabilistic data structure** that allows quickly checking if an element is in a set. Because it stores only bits, it needs far less memory than a full set and answers in constant time (**fast lookup**). Yet, it can have occasional **false positives**._
> 
> [
> 
> ![](https://substackcdn.com/image/fetch/$s_!Zf9t!,w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Ff5a21979-696d-4b04-ac64-041c98a96a00_874x583.png)
> 
> ](https://substackcdn.com/image/fetch/$s_!Zf9t!,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Ff5a21979-696d-4b04-ac64-041c98a96a00_874x583.png)
> 
> Bloom filters

Meanwhile, a traditional RDBMS like [PostgreSQL](https://www.postgresql.org) updates data pages in place on disk (B-tree), which can be slower for a burst of random writes but makes reads simple (each piece of data has one home).

The book also discusses other [indexing structures](https://sqlity.net/en/2445/b-plus-tree/) (hash indexes, secondary indexes, full-text indexes, etc.), but the B-tree vs LSM-tree was the big takeaway for me.

It’s a classic example of trade-offs: **LSM-trees achieve writes faster by turning random writes into sequential writes, at the cost of more complex reads and background compaction work**. B-trees trade off some write performance to make reads as efficient as possible with one-disc seek to find a record.

Now I understand why a database like **[RocksDB](https://rocksdb.org)** (an embeddable key-value store developed by Facebook, based on LSM trees) is favored for write-heavy workloads, or why _Cassandra_ can handle high ingest rates. In contrast, MySQL might struggle unless caching is implemented.

> 📝 _The book also covers **storage engine optimizations** like how some DBs use **copy-on-write B-trees** or **append-only** techniques to improve consistency, and how **compression** and **buffer caches** come into play._
> 
> 📗 _A good further reading on this topic is the book "**[Database Internals](https://amzn.to/4kFTqvV)**" by Alex Petrov. Petrov's book provides the implementation details that Kleppmann omits._

Another aspect I appreciated is the coverage of **data encoding and schema evolution** (from Chapter 4). The book discusses formats such as JSON, XML, and binary protocols (Thrift, Protocol Buffers, Avro), as well as the need for **backward and forward compatibility** when services communicate or when data is stored long-term.

It shows how using explicit schemas and versioning can make applications **forward-compatible** (e.g., new code can still read old messages, and vice versa). I learned the value of **schema registries** and format evolution – for instance, how [Avro’s](https://avro.apache.org) approach, with a writer schema and reader schema, allows data to be interpreted even as the schema evolves, as long as the changes are compatible.

Why is this in a book about data-intensive apps? Because **data outlives code**. If you deploy an update that changes how data is structured, you can’t just invalidate all old data or require everything to update in lockstep.

The table below compares JSON, XML, and Binary formats.

[

![](https://substackcdn.com/image/fetch/$s_!F6tb!,w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F8e62db59-847a-4096-8f06-bb1571c462b5_3133x1968.png)

](https://substackcdn.com/image/fetch/$s_!F6tb!,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F8e62db59-847a-4096-8f06-bb1571c462b5_3133x1968.png)

JSON vs XML vs Binary formats

The middle part of the book (Part II) delves deeply into distributed data systems, a topic that particularly interests me as an architect. It covers **replication, partitioning (also known as sharding), transactions, and consistency models**.

There are many learnings here, as this is the core of the book, so I’ll focus on a few that stood out for me:

DDIA explains the main approaches to replicating data across multiple nodes for fault-tolerance and scaling reads. The classic **leader-follower (single-leader) replication** is described in detail: one node is designated the leader (primary) that handles all writes, and it propagates changes to follower (replica) nodes.

This is used in many systems (PostgreSQL, MySQL, MongoDB, etc.) and ensures a **consistent ordering of writes** (since only one leader serializes them).

I liked how the book discussed the **trade-off between synchronous vs asynchronous replication**: synchronous replication means a leader waits for followers to confirm writes (for stronger consistency at the cost of latency), whereas asynchronous means followers lag, but the leader is more available.

It was a good refresher on _why_ we sometimes get replication lag and stale reads on followers.

[

![](https://substackcdn.com/image/fetch/$s_!fiKB!,w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F2b411645-fc7f-4c7c-9195-7bbfd2b5ddc3_1680x744.png)

](https://substackcdn.com/image/fetch/$s_!fiKB!,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F2b411645-fc7f-4c7c-9195-7bbfd2b5ddc3_1680x744.png)

Leader-based replication (Credits: Author)

The book also covers **multi-master setups** (where multiple nodes can accept writes). This can be useful for geographically distributed databases (each data center has a local leader to reduce latency) or for certain offline-capable apps.

However, it comes with the big headache of **write conflicts**: two leaders might accept conflicting writes concurrently. DDIA outlines conflict resolution strategies (such as last-write-wins and custom merge logic) and makes it clear why a multi-leader approach is **rarely worth it** unless truly necessary.

This helped me understand why systems like PostgreSQL and MongoDB default to single-leader replication, and why multi-leader setups (such as Active-Active configurations) tend to be limited to exceptional cases or carefully designed applications (like collaborative editing in Google Docs, where conflict resolution is application-specific).

At the end of chapter 5, the author also discusses **leaderless replications**. This is the model used by [Cassandra](https://aws.amazon.com/keyspaces/what-is-cassandra/), and [Voldemort](https://github.com/voldemort/voldemort), where there is no single leader, i.e., any replica can accept writes, and they use **quorum for consistency**.

The book describes how **quorum reads/writes** work: e.g., with _N_ replicas, you might require any _W_ of them to acknowledge a write and _R_ of them to respond to a read, such that _W + R > N_ ensures at least one up-to-date copy is read. This yields _**eventual consistency**_, a concept that the book explains in great detail.

[

![](https://substackcdn.com/image/fetch/$s_!rEWt!,w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Faabae476-49cd-40a3-a946-e48169e255f4_640x382.png)

](https://substackcdn.com/image/fetch/$s_!rEWt!,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Faabae476-49cd-40a3-a946-e48169e255f4_640x382.png)

A quorum write (Credits: Author)

I also found the discussion of **sloppy quorums. I hinted at handoffs,** interesting (where writes can be accepted by fewer nodes than the quorum to ensure high availability, at the cost of increased inconsistency risk). Sloppy quorums are particularly useful for increasing write availability.

All in all, it demystified how systems like Cassandra achieve high availability and write throughput by sacrificing strict consistency. The trade-off: you, the developer, now have to consider consistency issues (such as read-repair and tombstones).

[

![CDN media](https://substackcdn.com/image/fetch/$s_!Apms!,w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Ff44f7b97-9c2d-4d82-94c7-7bfb54edecc2_960x928.jpeg "CDN media")

](https://substackcdn.com/image/fetch/$s_!Apms!,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Ff44f7b97-9c2d-4d82-94c7-7bfb54edecc2_960x928.jpeg)

Eventual consistency as a comic book ([Source](https://www.dupuis.com/imbattable/bd/imbattable-tome-1-justice-et-legumes-frais/70978))

The book covers **partitioning** data across nodes to handle large data sets. It details two central partitioning schemes: **range partitioning** (each shard handles a contiguous key range) and **hash partitioning** (keys are hashed to shards).

**Range partitioning** can lead to hotspots if data isn’t uniform (e.g., all recent timestamps go to one shard), whereas hashing usually distributes load more evenly at the cost of losing locality (you can’t easily do range queries without touching many shards).

The image below shows the difference between Range and Hash partitioning.

[

![](https://substackcdn.com/image/fetch/$s_!ULpP!,w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fd2ff93f3-8a46-4965-b090-74c8403817bb_1575x1526.png)

](https://substackcdn.com/image/fetch/$s_!ULpP!,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fd2ff93f3-8a46-4965-b090-74c8403817bb_1575x1526.png)

Range vs Hash partitioning

An “aha” moment for me was the explanation of how _secondary indexes_ work in a sharded database. Either each shard maintains a local index (and a query must scatter to all shards), or you have a distributed index structure that itself must be partitioned. It’s a tricky problem, and it has given me even more respect for systems like [Elasticsearch](https://www.elastic.co/elasticsearch) or [MongoDB,](https://www.mongodb.com/) which provide secondary indexes on sharded data.

The key lesson is that **sharding is essential for scalability. Still, it adds complexity**, from determining the right partition key to rebalancing shards when a node is added, to handling multi-shard queries (scatter/gather).

In distributed systems, concepts like **consistency models,** **linearizability, serializability, snapshot isolation,** and the famous **CAP theorem** often confuse engineers. DDIA did a great job clarifying these.

If you’ve spent significant time building or designing database-backed systems, transactions are likely something you've both loved and hated. Chapter 7 of _Designing Data-Intensive Applications_ addresses the role of transactions in distributed systems.

People often say you must abandon transactions to achieve performance or scalability, but Kleppmann argues that’s not true. While multi-object transactions can be challenging in distributed settings, transactions themselves remain critical for many correctness guarantees.

Transactions are typically discussed around database **ACID properties**:

-   **🧨 Atomicity**: All parts of a transaction either succeed or fail together.
    
-   **🧮 Consistency**: The database is maintained in a “valid state,” although it's typically the application that defines what "valid" means.
    
-   **🔒 Isolation**: Concurrent transactions don't interfere with or see partial results of each other.
    
-   **🪵 Durability**: Once committed, the data remains stored and recoverable.
    

Storage engines almost universally support **single-object atomicity and isolation**, typically through write-ahead logs and locking mechanisms. The real complexity arises with **multi-object transactions**, especially across partitions, which is why many distributed databases avoid them.

[

![](https://substackcdn.com/image/fetch/$s_!73Da!,w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fcd0889e2-d170-496b-b0d1-39725974e662_1026x1328.png)

](https://substackcdn.com/image/fetch/$s_!73Da!,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fcd0889e2-d170-496b-b0d1-39725974e662_1026x1328.png)

What does **ACID** mean?

To boost performance, many databases don’t provide complete isolation by default. Instead, they offer weaker guarantees like **Read Committed** or **Snapshot Isolation**:

-   **🛡️ Read Committed Isolation**: Protects only against fundamental issues, such as dirty reads and dirty writes, but not more subtle problems, like read skew (inconsistent snapshots across different queries within a transaction).
    
-   **📸 Snapshot Isolation**: Provides consistent point-in-time snapshots, thereby reducing issues such as read skew. However, even snapshot isolation isn’t perfect; it can't fully protect against all concurrency anomalies, such as lost updates or write skew.
    

Common **race conditions** Kleppmann highlights include:

-   **🔁 Lost Updates**: When concurrent transactions overwrite each other's updates. Solutions range from atomic increment operations to explicit locks (`SELECT ... FOR UPDATE`), or optimistic concurrency controls, such as compare-and-set.
    
    [
    
    ![](https://substackcdn.com/image/fetch/$s_!UDyQ!,w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F3983155f-b2b3-4d84-8211-22d991676a1d_1670x552.png)
    
    ](https://substackcdn.com/image/fetch/$s_!UDyQ!,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F3983155f-b2b3-4d84-8211-22d991676a1d_1670x552.png)
    
    A race condition between two clients concurrently implementing a counter (Credits: Author)
    
-   **🫥 Write Skew and Phantom Reads**: Subtle issues where concurrent updates cause incorrect business logic outcomes. Serializable isolation is generally needed here.
    

While weaker isolation levels can boost performance, they introduce tricky concurrency bugs that are notoriously hard to detect and debug. Kleppmann argues strongly for **Serializable isolation**, the strongest isolation level, which avoids these issues altogether.

Serializable isolation can be implemented in multiple ways:

-   **🧵 Actual serial execution**: Simply run transactions one by one on a single thread. Surprisingly effective in modern systems with fast in-memory databases and short transactions, but it limits throughput to a single CPU.
    
-   **🛑 Two-Phase Locking (2PL)**: Uses shared and exclusive locks extensively to ensure transaction safety. It’s robust but can significantly degrade performance due to lock contention and deadlocks.
    
-   **📸 Serializable Snapshot Isolation (SSI)**: A newer, optimistic concurrency control technique gaining popularity. Instead of immediately blocking transactions, SSI detects conflicts upon commit, resulting in fewer unnecessary aborts. It was introduced in [Michael Cahill's PhD thesis](https://dl.acm.org/doi/10.1145/1620585.1620587) in 2008.
    

[

![](https://substackcdn.com/image/fetch/$s_!B9ng!,w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F2bf95cfb-d956-4595-bf19-4e4cbf4d68f9_1706x1130.png)

](https://substackcdn.com/image/fetch/$s_!B9ng!,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F2bf95cfb-d956-4595-bf19-4e4cbf4d68f9_1706x1130.png)

Seriazible Snapshot Isolation (Credits: Author)

The image below shows consistency models and isolation levels.

[

![](https://substackcdn.com/image/fetch/$s_!HCM3!,w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F08551a60-04ae-45c4-bf96-5d2923178993_1757x1421.png)

](https://substackcdn.com/image/fetch/$s_!HCM3!,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F08551a60-04ae-45c4-bf96-5d2923178993_1757x1421.png)

Isolation levels (Read more **[here](https://sergeiturukin.com/2017/06/29/eventual-consistency.html)** and **[here](https://jepsen.io/consistency/models)**)

Chapter 9 explains that **linearizability** (usually called “strong consistency”) is essentially the guarantee that every operation appears to execute atomically in some global order - it’s what you’d want for something like “read-after-write” always to return the latest write.

However, achieving linearizable reads across distributed replicas incurs a performance and availability cost (**[CAP theorem](https://en.wikipedia.org/wiki/CAP_theorem)**: you trade availability under partition for linearizability). The book uses CAP to explain why systems like Dynamo prioritize availability and partition tolerance over consistency, whereas systems like ZooKeeper prioritize consistency over availability.

[

![](https://substackcdn.com/image/fetch/$s_!aLCj!,w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F5e86cab9-9a33-46fe-a78c-6a1eb0688c8d_1280x720.png)

](https://substackcdn.com/image/fetch/$s_!aLCj!,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F5e86cab9-9a33-46fe-a78c-6a1eb0688c8d_1280x720.png)

The CAP Theorem

> ℹ️ **What is [CAP Theorem](https://en.wikipedia.org/wiki/CAP_theorem)?** _The CAP theorem is a fundamental concept in distributed systems and databases. It stands for Consistency, Availability, and Partition Tolerance, which are three properties that a distributed system can have. Yet, the theorem states that a distributed system can't simultaneously provide all three of these guarantees. For example, if we design a system where every read receives the most recent write (Consistency) and the system continues to operate despite network failures (Partition Tolerance), we may have to compromise on Availability._
> 
> ➡️ _Check the authors’ critiques of the CAP theorem in [this article](https://arxiv.org/abs/1509.05393)._

It also distinguishes **serializability** (an isolation property for transactions) from linearizability (a consistency property for reads and writes on single objects). A subtle point that many, including myself, weren’t super clear on before.

The treatment of **consensus algorithms** (such as [Raft](https://raft.github.io/) and [Paxos](https://www.scylladb.com/glossary/paxos-consensus-algorithm/)) was also approachable.

By the end, I had a better intuitive sense of how leaders are elected and why distributed systems require consensus for tasks like atomic commits.

One of the chapters I found especially valuable addresses the common problems that can be observed in distributed systems. We know that distributed systems promise scalability, reliability, and high availability; however, anyone who has built one also knows they have many challenges.

Kleppmann calls this out directly: unlike single-node systems (which typically either work entirely or fail), distributed systems can experience **partial failures**, where parts of the system break while the rest continue to work, often unpredictably.

Here are the key insights and lessons from this chapter:

-   **🎲 Faults, Partial Failures, and Nondeterminism**. Distributed systems are fundamentally nondeterministic. Nodes can fail silently, networks can drop messages, and software can behave unpredictably. Partial failures aren't just common, they're the norm. This unpredictability makes building distributed systems inherently more difficult.
    
-   **📡 Networks are unreliable (and always will be)**. The reality of modern networks is that they're asynchronous packet networks. That means messages sent between nodes come with **no delivery guarantees**; packets can be delayed, dropped, or duplicated. Usually, we handle these problems with timeouts and chaos testing (as seen on [Netflix’s Chaos Monkey](https://netflix.github.io/chaosmonkey/)).
    
-   **⏰ Clocks can’t be trusted**. Another subtle yet crucial issue: clocks across different nodes drift out of sync. Kleppmann explains the two main clock types clearly:
    
    -   **🕰️ Time-of-day clocks** (wall-clock time): These can move backward or forward unpredictably due to [NTP synchronization corrections](https://en.wikipedia.org/wiki/Network_Time_Protocol), making them unreliable for measuring elapsed time or sequencing events precisely.
        
    -   **⏱️ Monotonic clocks**: Guaranteed always to move forward, ideal for measuring durations, like request timeouts or response latencies.
        
    
    If precise synchronization is crucial (e.g., ordering transactions globally), tools like **[Google's TrueTime API](https://static.googleusercontent.com/media/research.google.com/en//archive/spanner-osdi2012.pdf)**, used in **[Spanner](https://cloud.google.com/spanner)**, become critical; however, they're also costly and complex. Therefore, it is essential not to trust timestamps across nodes blindly; if your logic relies on precise timing, you're likely to encounter trouble.
    
-   **👑 Leader election**. Many distributed systems rely on electing a "leader" node to coordinate operations. But, there is the challenge. Due to network partitions or delayed messages, sometimes multiple nodes think they’re the leader simultaneously, a dreaded situation known as "**split-brain**." The book recommends using **fencing tokens** to mitigate this. Each time leadership is granted, a unique, increasing token is provided. Operations require the latest token to proceed, effectively invalidating stale leaders.
    
    [
    
    ![](https://substackcdn.com/image/fetch/$s_!qeab!,w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F6a904b9f-1b70-4a22-adf9-33a354c90660_640x254.png)
    
    ](https://substackcdn.com/image/fetch/$s_!qeab!,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F6a904b9f-1b70-4a22-adf9-33a354c90660_640x254.png)
    
    Check-write problem that fencing tokens solve (Credits: Author)
    
-   **🦠 Byzantine faults**. Most practical distributed systems assume nodes behave honestly: they either work correctly or fail. However, Kleppmann discusses a more challenging scenario: "**Byzantine faults**," where nodes intentionally misbehave or send corrupted data. Systems requiring protection against such faults, such as blockchain networks, aerospace software, or military systems, employ specialized algorithms known as **Byzantine Fault Tolerant (BFT)** systems. However, BFT solutions are costly and complex.
    
    > _"A system is Byzantine fault-tolerant if it continues operating correctly even when some nodes lie."_
    
-   **✅ Correctness in distributed algorithms.** Finally, the chapter defines two properties crucial to understanding distributed algorithm correctness:
    
    -   **🛡️ Safety ("nothing bad happens")**: This must always hold. For instance, fencing tokens must be unique.
        
    -   **🌱 Liveness ("something good eventually happens")**: For example, "eventually receiving a response." Liveness may have conditions, e.g., provided a network partition eventually heals.
        
    
    Safety violations are catastrophic and irreversible; liveness violations might be temporary and recoverable. When designing or choosing algorithms, it’s essential to understand these properties clearly, balancing rigor (safety) with pragmatism (liveness).
    

> _This chapter reminds me a lot of the **Fallacies of Distributed Computing**. Read more about it **[here](https://newsletter.techworld-with-milan.com/i/148912953/fallacies-of-distributed-computing)**._

[

![Fallacies of Distributed Systems - by Mahdi Yusuf](https://substackcdn.com/image/fetch/$s_!k4rj!,w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F40e449e3-86c6-417a-ade1-277497182c28_2000x1414.jpeg "Fallacies of Distributed Systems - by Mahdi Yusuf")

](https://substackcdn.com/image/fetch/$s_!k4rj!,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F40e449e3-86c6-417a-ade1-277497182c28_2000x1414.jpeg)

8 Fallacies of Distributed Systems (Credits: Mahdi Yusuf)

The last part of DDIA focuses on **derived data** and data processing pipelines, specifically, **batch processing** (similar to Hadoop) and **stream processing** (similar to Kafka or Spark Streaming). This section was particularly relevant as our industry moves toward real-time data pipelines.

Kleppmann does a great job discussing batch and stream models, saying that, fundamentally, _many data systems boil down to moving data through **logs**_.

-   **🗃️ Batch processing.** The book uses _MapReduce_ and the Unix tool philosophy to explain batch jobs. Batch processing operates on large data sets but doesn’t provide immediate results – it’s about throughput over latency. For example, a nightly job might aggregate log files into a report. We measure batch jobs in **terms of throughput (records per second or total time to process a dataset)**. One superb example in the book is constructing a simple data pipeline with Unix pipes (grep, sort, etc.) and showing how that inspires distributed frameworks like Hadoop’s MapReduce. The key points are that batch jobs **read from a data source, process data in bulk, and output to another** location; these jobs are often scheduled to run periodically. They are great for large-scale analytics where a few minutes or hours of delay is acceptable.
    
-   **⚡Stream processing.** In contrast, stream processing deals with data **event-by-event** in real-time (or near real-time). Instead of processing a million records after the fact, a stream processor handles events _continuously_ as they happen (e.g., processing user actions on a website to update a real-time dashboard or trigger alerts). The benefit is **low latency** – you don’t have to wait for a scheduled job, you get insights or trigger actions immediately. However, stream processing is typically more complex to implement reliably (you deal with issues like exactly-once processing, out-of-order events, etc., which the book does touch on). Note that the presentation of exactly-once semantics in the book is is overly simplified.
    

What I loved is how the book ties stream processing to the earlier concepts. For instance, the log abstraction reappears: **a database’s change log can be viewed as a stream of events**. This is the idea behind **Change Data Capture (CDC)**, where changes in a database are captured and streamed to other systems for processing.

[

![](https://substackcdn.com/image/fetch/$s_!jtP8!,w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F5a2334d6-5941-4caf-b2ed-0ed09489e9d8_640x292.png)

](https://substackcdn.com/image/fetch/$s_!jtP8!,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F5a2334d6-5941-4caf-b2ed-0ed09489e9d8_640x292.png)

Change Data Capture process (Credits: Author)

Kleppmann gives an example: you can stream database updates to a search index or cache, rather than batch-syncing them occasionally. This is essentially how systems like **[Debezium](https://debezium.io/)** or **[LinkedIn’s Databus](https://github.com/linkedin/databus)** work. It blurs the line between “database” and “stream”: the replication log of your DB is feeding a real-time pipeline.

Similarly, the book describes **[Event Sourcing](https://martinfowler.com/eaaDev/EventSourcing.html)** – an architectural pattern where **state changes are logged as immutable events** and the current state is derived by replaying the event log. Many modern systems (especially in fintech and CQRS architectures) use this pattern, and DDIA gives it context: it’s another flavor of the general **idea of treating your data as streams of events**.

The image below shows an example of the Event Sourcing pattern.

[

![](https://substackcdn.com/image/fetch/$s_!5qO-!,w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fa8e03f37-8c99-4b92-b67c-eeb024a23740_2145x1789.png)

](https://substackcdn.com/image/fetch/$s_!5qO-!,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fa8e03f37-8c99-4b92-b67c-eeb024a23740_2145x1789.png)

Event Sourcing

The book also highlights the challenges, such as dealing with out-of-order events in streams (utilizing timestamps and windowing) or handling backpressure when producers are faster than consumers. These were covered conceptually.

It also mentions popular tools, such as **message brokers** (like **[RabbitMQ](https://www.rabbitmq.com/)** [](https://www.rabbitmq.com/)and **[ActiveMQ](https://activemq.apache.org/)**) versus **log-based brokers** (like **[Apache Kafka](https://kafka.apache.org) and** **[Amazon Kinesis](https://aws.amazon.com/kinesis/)**).

> ➡️ **Kafka** _is mentioned as a distributed log that supports high-throughput event streaming. I wish there were more information about stream processing frameworks (the book was published just before Apache Flink and others gained popularity). Still, the concepts it teaches are applicable regardless of the technology._
> 
> 💡 _Fun fact: one of the book’s reviewers is **Jay Kreps (creator of Kafka)**, who praised how it “bridges the gap between theory and practice.”_

[

![](https://substackcdn.com/image/fetch/$s_!gaEy!,w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fc5b33167-d4c9-4c43-b589-2014d1e4ffcd_6619x3678.png)

](https://substackcdn.com/image/fetch/$s_!gaEy!,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fc5b33167-d4c9-4c43-b589-2014d1e4ffcd_6619x3678.png)

Designing Data-Intensive Applications Book Map

No book is perfect. While I highly recommend DDIA, I do have a few critiques regarding its **limitations and shortcomings**:

The first edition of the book was published in 2017; since then, technology has continued to evolve. For example, **[Apache Kafka](https://kafka.apache.org/),** which by now is a cornerstone of many data architectures, is only briefly mentioned in the book. Book examples stop at 2016, which is a large gap of almost a decade in our industry.

Newer trends in cloud data warehouses, serverless computing, stream processing (Flink), or data lakes aren’t covered. **The core ideas in DDIA are timeless**, but some details (e.g., specific technologies or versions) feel a bit dated as of 2025. I understand that the author has published [updates online](https://martin.kleppmann.com/) (and a [second edition is also in preparation](https://www.oreilly.com/library/view/designing-data-intensive-applications/9781098119058/)).

Event mesh architectures and advanced CQRS implementations have become mainstream, with companies adopting "shock absorber" patterns and standardized event versioning strategies that build on DDIA's foundational concepts.

Still, the book itself does not include discussions of topics such as Kubernetes or the latest NewSQL or Vector databases, etc. It occasionally made me wonder, _“What about tool X that came out after 2019?”_

Depending on your learning style, this can be a pro or con. The book leans toward **conceptual explanations** over step-by-step tutorials or code. You won’t find ready-to-run examples or guidance on tuning a specific database.

For instance, it explains how a log-structured storage works in principle, but not how to configure Cassandra’s compaction strategy. I enjoyed the theory, but some readers might be hoping for a “how to build a scalable system” playbook with concrete recipes. DDIA is more like a textbook or reference – it gives you the mental models, not ready-to-use solutions.

**Chapter 9 (on consistency and consensus) is especially overloaded,** representing the book's most significant weakness, as it attempts to cover an entire semester of distributed systems content in a single chapter.

[

![](https://substackcdn.com/image/fetch/$s_!VKGL!,w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F6910a812-5e3a-47f0-b581-66898cbdd0e5_556x344.png)

](https://substackcdn.com/image/fetch/$s_!VKGL!,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F6910a812-5e3a-47f0-b581-66898cbdd0e5_556x344.png)

Chapter 9 (DDIA book)

The book is ambitiously broad, covering everything from low-level storage engines to high-level distributed algorithms. Sometimes I wondered **if the author wants to write about distributed systems or database engine**s, as those are systems on entirely different levels of abstraction.

Also, some topics don’t delve too deeply. Each chapter could probably be a book in its own right (indeed, there are entire books on consensus algorithms or specific databases). For example, the section on **distributed transactions** introduces 2PC but doesn’t delve into newer approaches, such as SAGAs or specific cloud implementations.

I sometimes expected more details on some challenging issues (like exactly-once stream processing mechanisms or deeper performance case studies) or events to point to simple implementations. The flip side is that the book stays focused and doesn’t get bogged down; however, readers expecting a deep dive into any single area might need to supplement with other resources.

This wasn’t a big issue for me, but I’ll note that DDIA is **long (500+ pages)** and dense with information. It’s not light bedtime reading for sure. The writing is clear, but it’s a lot to absorb - I had to read it in chunks and found myself re-reading some sections to understand it correctly (and take notes).

In terms of style, it’s pretty direct and matter-of-fact (it _is_ an engineering book, after all). A bit more narrative or real-world case studies could add some spice.

If you already know a topic well, those parts might feel slow; if it’s new to you, you might need to pause and digest. Some parts I also needed to re-read and understand better.

In short, it’s a **comprehensive reference**, but not exactly a page-turner of a story. Be prepared to invest some effort.

Despite these points, I want to say that none of them are deal-breakers. The “outdated” aspects primarily concern examples (the principles remain solid). And the theoretical nature of the book is by design - it’s actually what makes it stay relevant years later.

[

![](https://substackcdn.com/image/fetch/$s_!53Bj!,w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F815ad30c-03b8-4e74-9014-e6d06969a5c6_4032x2631.jpeg)

](https://substackcdn.com/image/fetch/$s_!53Bj!,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F815ad30c-03b8-4e74-9014-e6d06969a5c6_4032x2631.jpeg)

Part of my bookshelf

The book does not discuss practical migration scenarios, such as **how to perform live migrations, manage acceptable downtime, or handle migrations transparently using middleware layers**. Given that migrations frequently occur in real-world systems, these topics could have their place in the book.

The operational aspects of running distributed systems aren't deeply covered. Readers may miss guidance on **monitoring database telemetry, replication lag, handling production issues such as replication bugs**, and managing schema updates or changes to the replication topology. These topics have a significant impact on real-world systems but are not adequately addressed.

Additionally, I missed information about **backups, restores, RPO/RTO**, and how these factors impact the entire system.

As a summary of this book, I offer the following recommendation.

As a summary of this book, I would **recommend** it to **experienced software engineers, architects, and tech leads** (3-8 years of experience) who build or work with data-intensive systems. If you deal with databases, distributed systems, or large-scale data pipelines in your job, you’ll likely find significant value here.

Even if you have years of experience, DDIA will connect the dots and explain concepts deeply (it certainly did for me). I’d say it’s _essential reading_ if you aspire to design systems at scale – it gives you a vocabulary and framework to make smarter decisions.

I also think it’s a fantastic resource for **prep and self-education**. If you’re preparing for a systems design interview or transitioning into a more architecture-focused role, this book will level up your understanding.

On the other hand, if you’re a **newer developer or student** without much background in distributed systems or databases, parts of this book might be hard to understand. It assumes you already know concepts like SQL vs. NoSQL and have a general understanding of computing systems. A motivated beginner could still gain a lot from it, but be prepared to look up unfamiliar terms or reread sections.

The demanding structure can require readers to "**trace too many references**" (every chapter has 30-50) to fully grasp concepts, making it less accessible to engineers transitioning from other domains.

If you’re looking for **immediate, practical how-tos (e.g., “How do I set up a Kubernetes cluster for Kafka?”)**, You won't find them here. It’s neither a cookbook nor a vendor-specific guide. And, if your work is far removed from data systems (say you’re a pure front-end developer or a data scientist focusing on modeling), you might not need this level of systems detail in your daily work.

Lastly, anyone who dislikes theory or is short on time for reading might struggle – the book requires your full attention.

In summary, DDIA is **not a lightweight overview**; it’s for those who want to _gain a deep understanding of_ data system design. If that’s you, you’ll love it.

Here is a visual overview of **[my notes from the book](https://milan-milanovic.notion.site/Designing-Data-Intensive-Applications-Notes-by-Dr-Milan-Milanovic-1ac22f7b9a5f80eda8a0ebff46919989)**.

In summary, the book gave me a more precise mental map of distributed data system design. It connects the dots between theory and real systems: e.g., how **Kafka’s** design of a replicated log is essentially a leader-based replication under the hood, or how **Cassandra’s** eventual consistency model is an implementation of leaderless quorum replication.

I came away with a deeper understanding of _why_ specific systems make the choices they do. It’s now easier for me to reason about questions like _“Do we need a distributed transaction across services, or can we get away with eventual consistency?”_ or _“Should we prefer a single primary database with failover, or a multi-region multi-master setup?”_ because I can weigh the pros and cons more concretely (latency vs consistency vs complexity, etc.).

Those are some of the key points I carry with me after reading _Designing Data-Intensive Applications_. The book managed to both validate things I’d learned through experience _and_ teach me new ways to think about problems I’d not yet encountered.

**If you’re serious about building systems that handle lots of data, high traffic, or complex distributed workflows, this book is a must-read.** It packs a decade’s worth of hard-earned lessons (and research results) into one volume.

I know I’ll be reaching for it again, whether to double-check something about consistency models or to help decide between technologies for a new project.

For that sake, I created **a cheat sheet** below that you can use.

Here are some key learnings that I noted from the book:

1.  **🔧 Design for failure**. Assume things will fail. Use replication, retries, and graceful degradation. Faults aren't bugs, they're normal. Ensure no single point of failure exists.
    
2.  **⏱️ Measure what matters (latency vs throughput).** Don't rely on averages, watch percentile latencies (p95, p99). Users notice the slowest requests, not averages. Optimize for latency or throughput clearly, based on your goals.
    
3.  **🧩 Choose the right data model.** Match databases to your data:
    
    -   **🗄️ Relational DB** for complex joins and transactions.
        
    -   **📄 Document DB** for flexible schemas and self-contained records (like JSON).
        
    -   **🕸️ Graph DB** for highly interconnected data.
        
4.  **⚙️ Understand your storage engine.** Pick carefully between:
    
    -   **🌳 B-tree databases** (Postgres, MySQL): great for fast reads, slower writes.
        
    -   **📝 LSM-tree databases** (Cassandra, RocksDB): excellent write performance, slower reads.
        
5.  **🧭 Replication**. There are three replication models:
    
    -   **👑 Single-leader:** Simple, consistent, easy failover (standard default).
        
    -   **🌐 Multi-leader:** Complex, useful for multi-region writes, but challenging for conflict resolution.
        
    -   **🛡️ Leaderless:** Flexible, high availability, eventual consistency.
        
          
        Clearly understand consistency-latency tradeoffs and have a failover plan.
        
6.  **🗂️ Partitioning and data distribution:**
    
    -   **#️⃣ Hash partitioning:** Even distribution, fast point lookups, but poor range queries.
        
    -   **📏 Range partitioning** is suitable for range queries, but it risks creating hotspots.
        
          
        Be careful with cross-shard operations. Automate rebalancing and choose partition keys wisely.
        
7.  **🔒 Use transactions wisely**. Transactions (ACID) ensure correctness but add complexity in distributed systems. Avoid using distributed transactions unless necessary; use simpler alternatives, such as sagas, for cross-service workflows.
    
8.  **📩 Embrace Event-Driven architecture (when appropriate)**. Use event logs (e.g., Kafka) to decouple services. Event-driven architectures improve scalability and simplify integration. Be prepared to handle eventual consistency.
    
9.  **🛠️ Maintainability: simplicity and evolvability**. Keep systems as simple as possible. Prioritize observability, good metrics, and clear logs. Utilize schema versioning and implement backward-compatible changes to facilitate easier evolution over time.
    
10.  **⚖️ Always weigh trade-offs**. No single perfect solution exists. Identify what you're optimizing (consistency vs. availability, latency vs. throughput, simplicity vs. performance). Make intentional, context-aware trade-offs rather than defaulting blindly.
     

[

![](https://substackcdn.com/image/fetch/$s_!QJ_o!,w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F242c47ea-6481-4ff9-a3e2-59c02fb2a623_1414x2000.png)

](https://substackcdn.com/image/fetch/$s_!QJ_o!,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F242c47ea-6481-4ff9-a3e2-59c02fb2a623_1414x2000.png)


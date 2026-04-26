# Transactions Across Microservices | Baeldung

## **1\. Introduction**

In this article, we’ll discuss options to implement a transaction across microservices.

We’ll also check out some alternatives to transactions in a distributed microservice scenario.

A distributed transaction is a very complex process with a lot of moving parts that can fail. Also, if these parts run on different machines or even in different data centers, the process of committing a transaction could become very long and unreliable.

This could seriously affect the user experience and overall system bandwidth. So **one of the best ways to solve the problem of distributed transactions is to avoid them completely.**

### **2.1. Example of Architecture Requiring Transactions**

Usually, a microservice is designed in such way as to be independent and useful on its own. It should be able to solve some atomic business task.

**If we could split our system in such microservices, there’s a good chance we wouldn’t need to implement transactions between them at all.**

For example, let’s consider a system of broadcast messaging between users.

The _user_ microservice would be concerned with the user profile (creating a new user, editing profile data etc.) with the following underlying domain class:

```
@Entity
public class User implements Serializable {

    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private long id;

    @Basic
    private String name;

    @Basic
    private String surname;

    @Basic
    private Instant lastMessageTime;
}
```

The _message_ microservice would be concerned with broadcasting. It encapsulates the entity _Message_ and everything around it:

```
@Entity
public class Message implements Serializable {

    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private long id;

    @Basic
    private long userId;

    @Basic
    private String contents;

    @Basic
    private Instant messageTimestamp;

}
```

Each microservice has its own database. Notice that we don’t refer to the entity _User_ from the entity _Message_, as the user classes aren’t accessible from the _message_ microservice. We refer to the user only by id.

Now the _User_ entity contains the _lastMessageTime_ field because we want to show the information about the last user activity time in her profile.

However, to add a new message to the user and update her _lastMessageTime_, we’d now have to implement a transaction across microservices.

### **2.2. Alternative Approach Without Transactions**

We can alter our microservice architecture and remove the field _lastMessageTime_ from the _User_ entity.

Then we could display this time in the user profile by issuing a separate request to the messages microservice and finding the maximum _messageTimestamp_ value for all messages of this user.

Probably, if the _message_ microservice is under high load or even down, we won’t be able to show the time of the last message of the user in her profile.

**But that could be more acceptable than failing to commit a distributed transaction to save a message just because the user microservice didn’t respond in time.**

There are of course more complex scenarios when we have to implement a business process [across multiple microservices](/cs/microservices-cross-cutting-concerns), and we don’t want to allow inconsistency between those microservices.

## **3\. Two-Phase Commit Protocol**

[Two-phase commit protocol](https://en.wikipedia.org/wiki/Two-phase_commit_protocol) (or 2PC) is a mechanism for implementing a transaction across different software components (multiple databases, message queues etc.)

### **3.1. The Architecture of 2PC**

One of the important participants in a distributed transaction is the transaction coordinator. The distributed transaction consists of two steps:

-   Prepare phase — during this phase, all participants of the transaction prepare for commit and notify the coordinator that they are ready to complete the transaction
-   Commit or Rollback phase — during this phase, either a commit or a rollback command is issued by the transaction coordinator to all participants

**The problem with 2PC is that it is quite slow compared to the time for operation of a single microservice.**

**Coordinating the transaction between microservices, even if they are on the same network, can really slow the system down**, so this approach isn’t usually used in a high load scenario.

### **3.2. XA Standard**

The [XA standard](https://en.wikipedia.org/wiki/X/Open_XA) is a specification for conducting the 2PC distributed transactions across the supporting resources. Any JTA-compliant application server (JBoss, GlassFish etc.) supports it out-of-the-box.

The resources participating in a distributed transactions could be, for example, two databases of two different microservices.

**However, to take advantage of this mechanism, the resources have to be deployed to a single JTA platform. This isn’t always feasible for a microservice architecture.**

### **3.3. REST-AT Standard Draft**

Another proposed standard is [REST-AT](https://github.com/jbosstm/documentation/tree/master/rts/docs) which had undergone some development by RedHat but still didn’t get out of the draft stage. It’s however supported by the WildFly application server out-of-the-box.

This standard allows using the application server as a transaction coordinator with a specific REST API for creating and joining the distributed transactions.

The RESTful web services that wish to participate in the two-phase transaction also have to support a specific REST API.

Unfortunately, to bridge a distributed transaction to local resources of the microservice, we’d still have to either deploy these resources to a single JTA platform or solve a non-trivial task of writing this bridge ourselves.

## **4\. Eventual Consistency and Compensation**

**By far, one of the most feasible models of handling consistency across microservices is [eventual consistency](/cs/eventual-consistency-vs-strong-eventual-consistency-vs-strong-consistency).**

This model doesn’t enforce distributed ACID transactions across microservices. Instead, it proposes to use some mechanisms of ensuring that the system would be eventually consistent at some point in the future.

### **4.1. A Case for Eventual Consistency**

For example, suppose we need to solve the following task:

-   register a user profile
-   do some automated background check that the user can actually access the system

The second task is to ensure, for example, that this user wasn’t banned from our servers for some reason.

But it could take time, and we’d like to extract it to a separate microservice. It wouldn’t be reasonable to keep the user waiting for so long just to know that she was registered successfully.

**One way to solve it would be with a message-driven approach including compensation.** Let’s consider the following architecture:

-   the _user_ microservice tasked with registering a user profile
-   the _validation_ microservice tasked with doing a background check
-   the messaging platform that supports persistent queues

The messaging platform could ensure that the messages sent by the microservices are persisted. Then they would be delivered at a later time if the receiver weren’t currently available

### **4.2. Happy Scenario**

In this architecture, a happy scenario would be:

-   the _user_ microservice registers a user, saving information about her in its local database
-   the _user_ microservice marks this user with a flag. It could signify that this user hasn’t yet been validated and doesn’t have access to full system functionality
-   a confirmation of registration is sent to the user with a warning that not all functionality of the system is accessible right away
-   the _user_ microservice sends a message to the _validation_ microservice to do the background check of a user
-   the _validation_ microservice runs the background check and sends a message to the _user_ microservice with the results of the check
    -   if the results are positive, the _user_ microservice unblocks the user
    -   if the results are negative, the _user_ microservice deletes the user account

After we’ve gone through all these steps, the system should be in a consistent state. However, for some period of time, the user entity appeared to be in an incomplete state.

**The last step, when the user microservice removes the invalid account, is a compensation phase**.

### **4.3. Failure Scenarios**

Now let’s consider some failure scenarios:

-   if the _validation_ microservice is not accessible, then the messaging platform with its persistent queue functionality ensures that the _validation_ microservice would receive this message at some later time
-   suppose the messaging platform fails, then the _user_ microservice tries to send the message again at some later time, for example, by scheduled batch-processing of all users that were not yet validated
-   if the _validation_ microservice receives the message, validates the user but can’t send the answer back due to the messaging platform failure, the _validation_ microservice also retries sending the message at some later time
-   if one of the messages got lost, or some other failure happened, the _user_ microservice finds all non-validated users by scheduled batch-processing and sends requests for validation again

Even if some of the messages were issued multiple times, this wouldn’t affect the consistency of the data in the microservices’ databases.

**By carefully considering all possible failure scenarios, we can ensure that our system would satisfy the conditions of eventual consistency. At the same time, we wouldn’t need to deal with the costly distributed transactions.**

But we have to be aware that ensuring eventual consistency is a complex task. It doesn’t have a single solution for all cases.

## **5\. Conclusion**

In this article, we’ve discussed some of the mechanisms for implementing transactions across microservices.

And, we’ve also explored some alternatives to doing this style of transactions in the first place.
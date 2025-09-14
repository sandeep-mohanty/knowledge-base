# Ambassador Design Pattern in Java

In this tutorial, we'll explore the **Ambassador Design Pattern**, a structural pattern used to decouple client applications from remote services or components, while adding additional behavior such as logging, retry logic, or latency simulation.

The Ambassador pattern is part of the **Cloud Design Patterns** and is useful when dealing with unreliable or high-latency services.

## Table of Contents

- [What is the Ambassador Pattern?](#what-is-the-ambassador-pattern)
- [When to Use It](#when-to-use-it)
- [Components of the Pattern](#components-of-the-pattern)
- [Implementing the Ambassador Pattern in Java](#implementing-the-ambassador-pattern-in-java)
  - [1. Remote Service](#1-remote-service)
  - [2. Ambassador](#2-ambassador)
  - [3. Client](#3-client)
- [Enhancements](#enhancements)
- [Conclusion](#conclusion)
- [References](#references)

---

## What is the Ambassador Pattern?

The **Ambassador Pattern** provides a local service that acts as a proxy to a remote service. The ambassador handles responsibilities like:

- Logging
- Retry logic
- Monitoring
- Security
- Latency simulation

This helps decouple the client from the complexities of calling a remote service directly.

---

## When to Use It

Use the Ambassador Pattern when:

- You need to call a remote service that is slow or unreliable.
- You want to simulate or monitor latency.
- You want to abstract infrastructure concerns from the client.
- You're building cloud-native or microservice-based applications.

---

## Components of the Pattern

- **Remote Service**: A service that performs a task but may be prone to failure or latency.
- **Ambassador**: A local object that wraps the remote service and adds additional behavior.
- **Client**: The consumer that interacts only with the ambassador, not the remote service.

---

## Implementing the Ambassador Pattern in Java

Let’s implement a simple version of the Ambassador pattern in Java.

### 1. Remote Service

This simulates an external service that may fail or be slow.

```java
public class RemoteService {

    public int doRemoteFunction(int value) {
        int waitTime = new Random().nextInt(1000);
        try {
            Thread.sleep(waitTime);
        } catch (InterruptedException e) {
            return -1;
        }

        return waitTime <= 200 ? value * 10 : -1;
    }
}
```

- The method simulates latency.
- If the operation takes more than 200ms, it's considered a failure.

---

### 2. Ambassador

This class wraps the remote service and adds retry logic and logging.

```java
public class ServiceAmbassador {

    private final RemoteService service = new RemoteService();

    public int doRemoteFunction(int value) {
        System.out.println("Ambassador: Starting request...");

        for (int i = 0; i < 3; i++) {
            int result = service.doRemoteFunction(value);

            if (result != -1) {
                System.out.println("Ambassador: Success on attempt " + (i + 1));
                return result;
            }

            System.out.println("Ambassador: Attempt " + (i + 1) + " failed, retrying...");
        }

        System.out.println("Ambassador: All attempts failed.");
        return -1;
    }
}
```

- Adds logging
- Retries the operation up to 3 times if it fails
- Shields the client from complexity

---

### 3. Client

The client uses the ambassador instead of calling the remote service directly.

```java
public class Client {

    public static void main(String[] args) {
        ServiceAmbassador ambassador = new ServiceAmbassador();
        int result = ambassador.doRemoteFunction(5);

        if (result != -1) {
            System.out.println("Client: Received result = " + result);
        } else {
            System.out.println("Client: Operation failed.");
        }
    }
}
```

Output may vary due to random wait times, but it demonstrates retry behavior and logging.

---

## Enhancements

You can extend this pattern with:

- **Circuit Breakers** (e.g., Resilience4j, Hystrix)
- **Caching** of results
- **Authentication** and security checks
- **Metrics** collection (e.g., Prometheus, Micrometer)

These can be added inside the Ambassador class without impacting the client logic.

---

## Conclusion

The **Ambassador Pattern** is a powerful pattern for abstracting remote service calls and adding behaviors like retry, logging, and latency simulation. It is especially useful in **distributed systems** and **cloud-native architectures** where remote services may be unreliable.

By using an ambassador, clients remain clean and focused, while the ambassador handles the complexities of remote communication.

---

## References

- Baeldung Article: [Ambassador Design Pattern in Java](https://www.baeldung.com/java-ambassador-design-pattern)
- [Cloud Design Patterns - Microsoft Docs](https://learn.microsoft.com/en-us/azure/architecture/patterns/ambassador)
- [Martin Fowler - Remote Facade](https://martinfowler.com/eaaCatalog/remoteFacade.html)
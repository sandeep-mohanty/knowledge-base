# A Guide to RestTestClient

## 1. Introduction

Spring’s testing ecosystem has evolved from mock-based simulations to full integration with embedded servers. The latest addition, _RestTestClient_ in Spring Framework 7.0, bridges the gap by offering a concise, builder-style interface for HTTP interactions without the ceremony of traditional clients. This makes it a lightweight alternative to _MockMvc_ or _WebTestClient –_ ideal for integration tests that need speed, readability, and flexibility.

In this tutorial, we’ll set up _RestTestClient_ in a Spring Boot project, explore practical examples covering basic requests, error handling, assertions, and more, and highlight best practices to ensure our tests are robust and maintainable.

## 2. Setting up _RestTestClient_

To use _RestTestClient_, we need a [Spring Boot project](/spring-boot) with the appropriate testing dependencies.

Let’s start by including the [Spring Boot Test starter](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-test) in our _pom.xml_:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>
```

**We also must ensure we’re using [Spring Framework 7.0](https://mvnrepository.com/artifact/org.springframework/spring-core/7.0.0) or later, as _RestTestClient_ is a newer addition.**

Next, let’s configure a test class with the [_@SpringBootTest_](/spring-boot-testing) annotation to load the application context:

```java
@SpringBootTest
class RestTestClientUnitTest {

    private RestTestClient restTestClient;

    @BeforeEach
    void beforeEach() {
        // TODO: initialize restTestClient
    }
}
```

This setup allows us to instantiate the _RestTestClient_ any way we like, in the _beforeEach()_ method.

### 2.1. Binding

Before writing tests, we must decide how to create our _RestTestClient_. One of its strengths is the variety of binding options:

1.  Bind to an already initialised [_MockMvc_](/spring-mockmvc-vs-webmvctest) instance to use as the server: _bindTo(MockMvc mockMvc)_
2.  Bind to a live server: _bindToServer(ClientHttpRequestFactory requestFactory)_
3.  Bind to a [_WebApplicationContext_](/spring-webappconfiguration): _bindToApplicationContext(WebApplicationContext context)_
4.  Bind to (multiple) [_RouterFunction_](/spring-5-functional-web#2-routerfunction)(s): _bindToRouterFunction(RouterFunction<?>… routerFunctions)_
5.  Bind to (multiple) [_Controllers_](/spring-controllers): _bindToController(Object… controllers)_

These options bring a high level of flexibility, which makes it an optimal choice testing any Spring Boot project.

### 2.2. Configuration

The final step before testing is client configuration. We can fine-tune _RestTestClient_ via its builder:

```java
restTestClientBuilder
  .baseUrl("/public") // 1
  .defaultHeader("ContentType", "application/json") // 2
  .defaultCookie("JSESSIONID", "abc123def456ghi789") // 3
  .build();
```

The three options in the example are:

1.  Setting up a base url, like a prefix _/public_
2.  Setting default headers, like content type
3.  Setting a default cookie, like a session ID

With setup complete, we’re ready for our first test.

## 3. Practical Examples

Let’s explore several scenarios to highlight _RestTestClient’_s flexibility, including different types of use cases and complex assertions.

The following tests will be bound to and run against our controller:

```java
@RestController("my")
class MyController {

    @GetMapping("/persons/{id}")
    public ResponseEntity<Person> getPersonById(@PathVariable Long id) {
        return id == 1 
          ? ResponseEntity.ok(new Person(1L, "John Doe")) 
          : ResponseEntity.noContent().build();
    }
}
```

The _getPersonById()_ method is returning the entity _Person_:

```java
record Person(Long id, String name) { }
```

### 3.1. Happy Path

Our first test will cover a happy path, by calling a GET request to fetch a person by its id:

```java
@Test
void givenValidPath_whenCalled_thenReturnOk() {
    restTestClient.get() // 1
      .uri("/persons/1") // 2
      .accept(MediaType.APPLICATION_JSON) // 3
      .exchange() // 4
      .expectStatus() // 5
      .isOk() // 6
      .expectBody(Person.class) // 7
      .isEqualTo(new Person(1L, "John Doe")); // 8
}
```

We’re using the API of _RestTestClient_ to (1) initialise an appropriate request to (2) a certain path, with (3) appropriate accept header. We then (4) execute it, and (5) check  for the expected status – OK (6) in our case. Lastly, we (7) convert the body to the return type _Person_ and (8) assert it towards the expected instance.

This looks pretty straight forward, but how would we check for an unsuccessful request?

### 3.2. Simple Error Cases

Our second test covers a client error (wrong HTTP method):

```java
@Test
void givenWrongMethod_whenCalled_thenReturnClientError() {
    restTestClient.post() // <== wrong method
      .uri("/persons/1")
      .accept(MediaType.APPLICATION_JSON)
      .exchange()
      .expectStatus()
      .is4xxClientError();
}
```

And this is how we’d check for _NO_CONTENT_ response (invalid ID):

```java
@Test
void givenWrongId_whenCalled_thenReturnNoContent() {
    restTestClient.get()
      .uri("/persons/0") // <== wrong id
      .accept(MediaType.APPLICATION_JSON)
      .exchange()
      .expectStatus()
      .isNoContent();
}
```

### 3.3. Json Assertions

_RestTestClient_ integrates seamlessly with JSON Path for detailed body assertions:

```java
@Test
void givenValidId_whenGetPerson_thenReturnsCorrectFields() {
    restTestClient.get()
      .uri("/persons/1")
      .exchange()
      .expectStatus()
      .isOk()
      .expectBody()
      .jsonPath("$.id").isEqualTo(1)
      .jsonPath("$.name").isEqualTo("John Doe");
}
```

This avoids full object deserialisation while verifying structure and values precisely.

### 3.4. Custom Assertions

If we prefer custom assertions for more complex scenarios, we can use the _consumeWith()_ method:

```java
@Test
void givenValidRequest_whenGetPerson_thenPassesAllAssertions() {
    restTestClient.get()
      .uri("/persons/1")
      .exchange()
      .expectStatus()
      .isOk()
      .expectBody(Person.class)
      .consumeWith(result -> {
          assertThat(result.getStatus().value()).isEqualTo(200);
          assertThat(result.getResponseBody().name()).isNotNull().isEqualTo("John Doe");
      });
}
```

In the _Consumer<EntityExchangeResult<B>>_ that we provide, we can use any assertions we want, like the [AssertJ](/introduction-to-assertj) library.

### 3.5. Multiple Controllers

The last scenario we’ll be looking at using multiple controllers at once.

_RestTestClient_ makes it possible to bind more than one controller:

```java
restTestClient = RestTestClient.bindToController(myController, anotherController)
  .build();
```

Now we can write a test, asserting the endpoint of the second controller, in the same test class:

```java
@Test
void givenValidQueryToSecondController_whenGetPenguinMono_thenReturnsEmpty() {
    restTestClient.get()
      .uri("/pink/penguin")
      .accept(MediaType.APPLICATION_JSON)
      .exchange()
      .expectStatus()
      .isOk()
      .expectBody(Penguin.class)
      .value(it -> assertThat(it).isNull());
}
```

This approach is useful for testing composite APIs or modular services, ensuring interactions between controllers work as expected without a full server spin-up.

## 4. Best Practices and Pitfalls

Since we’ve covered the basics, let’s go for a deep dive into extended possibilities and pitfalls of the new _RestTestClient._

### 4.1. Choosing the Wrong Binding Setup

**One of the first things to get right when using _RestTestClient_ is** **which binding mode to use**. The API supports a wide range of bindings, as previously mentioned. It’s easy to pick the wrong one.

A common pitfall is using the “mock” binding (controller or context) when we really need full server behaviour. For example, if we bind to a controller, we may not exercise the full HTTP stack (servlet filters, Spring Security, message converters registered globally)!

Wrong bindings can lead to false-negatives in tests (things pass but will fail in production). **We should make sure, the binding method and the arguments provided are valid for the expected use case.**

### 4.2. Thread Safety and Context

_RestTestClient_ instances are immutable once built, making them thread-safe and suitable for parallel test execution. This immutability ensures no shared mutable state, allowing safe reuse across tests without race conditions, which is ideal for speeding up large test suites.

**The _RestTestClient.Builder_, however, is mutable and not thread-safe.** Sharing a single builder instance across tests or threads can lead to unpredictable configurations, like overwritten headers.

**We should either create a fresh builder per test, or use only the built immutable instances (recommended).**

### 4.3. Unnoticed Differences in Behaviour

A tricky risk with _RestTestClient_ is its subtle behavioural differences compared to other test clients like [_WebTestClient_](https://courses.baeldung.com/courses/2463040/lectures/52134950). For example, there is an [open issue](https://github.com/spring-projects/spring-framework/issues/35590) where _RestTestClient_ returns _null_ for _returnResult().getResponseBody()_ when the controller returns no body, whereas _WebTestClient_ returns an empty _byte[]_.

This means that if we write our assertions expecting an empty body and we switch contexts (client vs real server) we might face NPEs or misleading results.

Best practice: **we should explicitly assert _expectBody().isEmpty()_ when no body is expected** rather than relying on _returnResult()_ and then check for _null_ vs array.

## 5. Conclusion

In this article, we explored _RestTestClient_, a modern, fluent addition to Spring Framework 7.0 that simplifies REST integration testing in Spring Boot.

From flexible binding and configuration to expressive assertions on JSON, headers and cookies, it strikes a balance between readability and power – making it an excellent choice over heavier alternatives.

As always, the code is available [over on GitHub](https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-4).

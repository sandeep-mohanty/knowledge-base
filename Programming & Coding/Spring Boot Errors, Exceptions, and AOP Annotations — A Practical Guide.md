# Spring Boot Errors, Exceptions, and AOP Annotations — A Practical Guide

Learn how to centralize error handling, simplify exception management, and use AOP annotations effectively in Spring Boot applications.

In this article, we will explore **Spring Boot errors and AOP annotations** with easy-to-follow examples. These concepts play a crucial role in building robust Spring Boot applications. Let’s dive in and understand how they work together to make your code cleaner and more efficient.

Let’s look at the most commonly used annotations in Spring and Spring Boot for handling errors, exceptions, and AOP, one by one.

## Annotations for Errors & Exceptions in Spring Boot
Spring Boot treats exception handling as a **cross-cutting concern**, and provides several useful annotations to manage it effectively. Some of the most commonly used ones are:

* `@ExceptionHandler`
* `@ControllerAdvice`
* `@ResponseStatus`

### @ResponseStatus
By default, when a runtime exception occurs in a Spring Boot application, the controller returns a generic error response with **HTTP 500 (Internal Server Error)**.

However, sometimes we want to return a more meaningful status code, such as 404 NOT FOUND or 400 BAD REQUEST.

The `@ResponseStatus` annotation allows us to customize the HTTP status for a specific exception. You can apply `@ResponseStatus` in the following places:
* On a **custom exception class**
* On a method with `@ExceptionHandler`
* On a class annotated with `@ControllerAdvice`

**Example: Custom Exception**
```java
@ResponseStatus(code = HttpStatus.NOT_FOUND)
public class NoSuchUserFoundException extends RuntimeException {
    ...
}
```
Now, whenever this exception is thrown, Spring automatically returns **404 NOT FOUND**, which is more meaningful for clients.



### @ExceptionHandler
If you want full control over the **response structure** (message, body, fields, etc.), use the `@ExceptionHandler` annotation. You can place it inside a specific controller or inside a `@ControllerAdvice` class.

**Example: Handling Exception in Controller**
```java
@RestController
@RequestMapping("/user")
public class UserController {

    @Autowired
    private UserService userService;

    @GetMapping("/{id}")
    public Response getUser(@PathVariable String id) {
        return userService.getUser(id);
    }

    @ExceptionHandler(NoSuchUserFoundException.class)
    @ResponseStatus(HttpStatus.NOT_FOUND)
    public ResponseEntity<String> handleNoSuchUserFoundException(NoSuchUserFoundException exception) {
        return ResponseEntity
                .status(HttpStatus.NOT_FOUND)
                .body(exception.getMessage());
    }
}
```
This method will be triggered whenever `NoSuchUserFoundException` occurs in this controller.

### @ControllerAdvice and @RestControllerAdvice
Sometimes, we want to handle exceptions **globally** (for all controllers in the application). This is where `@ControllerAdvice` and `@RestControllerAdvice` come in.

* **@ControllerAdvice:** Works with MVC controllers (you may need `@ResponseBody`)
* **@RestControllerAdvice:** Same as `@ControllerAdvice` but automatically adds `@ResponseBody`

These annotations allow us to define **centralized exception-handling logic**, which is a cross-cutting concern (a concept from AOP).

**Example: Global Exception Handler**
```java
//@RestControllerAdvice
@ControllerAdvice
public class UserExceptionHandler {

    @ExceptionHandler(NoSuchUserFoundException.class)
    @ResponseBody
    public ResponseEntity<Object> handleUserNotFoundException(NoSuchUserFoundException ex) {
        return new ResponseEntity<>("User not found", HttpStatus.NOT_FOUND);
    }
}
```
This handler applies to all controllers in the application, not just `UserController`.

---

## What is Used in Spring AOP?
Spring AOP uses the **AspectJ** library annotations to create Aspects, Pointcuts, and Advices. To enable AOP features in a Spring Boot application, add the following dependency in your `pom.xml`:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```

**@EnableAspectJAutoProxy:** To enable AOP in your Spring application, you must annotate a configuration class with:
```java
@Configuration
@EnableAspectJAutoProxy
public class MyAopConfiguration { }
```
This tells Spring to detect classes marked with `@Aspect` and apply advice to the selected methods.

**@Aspect:** This is used to mark a class as an Aspect, meaning it contains AOP-related logic.
```java
@Aspect
public class InvoiceAspect { }
```

### Pointcut Annotation
**@Pointcut:** This is used to define where an advice should be applied. It selects business methods using an expression, but it does not specify which advice will run.

```java
@Pointcut("execution(public void com.sunil.spring.aop.service.InvoiceBusinessService.saveInvoice())")
public void point1() { }
```
Now all advices can reuse this pointcut by calling `point1()`.



### Types of Advices
**Note:** Every Advice annotation must reference a pointcut (directly or indirectly).

* **@Before:** Executes **before** the business method runs.
```java
@Before("point1()")
public void beginTransaction() {
    System.out.println("Transaction begins!");
}
```
* **@After:** Executes **after** the business method runs, whether it succeeds or fails.
```java
@After("point1()")
public void completeTransaction() {
    System.out.println("Transaction completes!");
}
```
* **@AfterReturning:** Executes **only if the business method executes successfully** (no exceptions).
```java
@AfterReturning("point1()")
public void commitTransaction() {
    System.out.println("Transaction committed!");
}
```
* **@AfterThrowing:** Executes **only when the business method throws an exception.**
```java
@AfterThrowing("point1()")
public void rollbackTransaction() {
    System.out.println("Transaction rolled back!");
}
```
* **@Around:** Executes **both before and after** the business method. This is the most powerful advice type.
```java
@Around("point4()")
public void testAroundAdvice(ProceedingJoinPoint pointj) throws Throwable {
    System.out.println("Executing Before part of business method");
    pointj.proceed(); // calls the business method
    System.out.println("Executing After part of business method");
}
```



Handling errors and cross-cutting concerns properly is key to building reliable Spring Boot applications. Using exception-handling annotations helps you manage errors in one place and keep your code clean. AOP annotations allow you to add features like logging and validation without mixing them into your business logic.

When used together, these tools make your application easier to read, maintain, and scale. By understanding and applying them correctly, you can write cleaner code and build more robust Spring Boot applications with less effort.
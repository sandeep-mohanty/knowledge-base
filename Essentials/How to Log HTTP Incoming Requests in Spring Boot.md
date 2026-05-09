# How to Log HTTP Incoming Requests in Spring Boot

In developing [REST](https://dzone.com/articles/restful-services-1 "REST ") APIs, you often need to log [HTTP](https://dzone.com/articles/http-protocol-obviously-unobvious "HTTP ") incoming requests. You want to see exactly what data your application is receiving and how it is processed. You want a detailed view of the passed data to ease troubleshooting and development. **CommonsRequestLoggingFilter** is a class of [Spring Boot](https://codingstrain.com/category/java/spring/spring-boot/ "Spring Boot") that allows you to log requests with simple configuration steps.

In this article, you'll see how to configure request logging in Spring Boot and inspect request payloads and parameters.

## Why You Need to Log HTTP Incoming Requests?

Sometimes, you need a thorough look at the data that comes into our application. A typical scenario is when you need to solve a subtle bug, and you don't have enough information to understand what's going on. 

Logging HTTP requests allows you to improve control over the APIs' development to find issues in the shortest time possible. You will be able to inspect the request payloads and query parameters and verify the correct integration between services. It also represents a viable way to monitor APIs and catch unexpected behavior in time.

If you don't use specific tools, the best you can do is write logs throughout the application. This is far from ideal, because all those tracings scattered in your code are hard to maintain, and they can contain errors. With **CommonsRequestLoggingFilter** you have a more centralized way of handling this.

## Request Logging Configuration

To configure logging of incoming requests, you should follow some simple steps. First of all, you should create a request logging filter by defining a configuration bean of type **_CommonsRequestLoggingFilter_**:

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.filter.CommonsRequestLoggingFilter;

@Configuration
public class ReqLoggingConfig {

    @Bean
    public CommonsRequestLoggingFilter requestLoggingFilter() {
        CommonsRequestLoggingFilter loggingFilter = new CommonsRequestLoggingFilter();
        
        loggingFilter.setIncludeQueryString(true);
        loggingFilter.setIncludePayload(true);       // Enables request body logging
        loggingFilter.setMaxPayloadLength(10000);   // Limits payload size
        loggingFilter.setIncludeHeaders(false);     // Avoids logging headers
        loggingFilter.setAfterMessagePrefix("REQUEST DATA: ");
        
        return loggingFilter;
    }
}
```

The above filter allows you to log:

-   The query parameters
-   The request body
-   Client information
-   The request URI

An important setting in the above example is the `setMaxPayloadLength()` instruction. It prevents excessive memory consumption by limiting the payload size.

The above is not enough, though. To make logging work, you should take a few additional steps. By default, the filter will not produce output unless the logging level is enabled. Add the following configuration to your **application.properties**:

```bash
logging.level.org.springframework.web.filter.CommonsRequestLoggingFilter=DEBUG
```

This enables debug logging for the filter so that requests will appear in the application logs. If you also want to log request parameters, add this property:

```bash
spring.mvc.publish-request-params=true
```

This allows Spring to expose them when the request is processed inside the controller.

Here is a summary of some important points to remember:

-   Use **CommonsRequestLoggingFilter** to log request payloads and parameters.
-   Enable debug logging for the filter.
-   Limit payload size to prevent memory issues.
-   Avoid logging sensitive information in production.

## Inspect How Spring Processes HTTP Requests

If you want to perform a more detailed inspection, in terms of request parameter resolution, request body conversion, and HTTP message processing, you have to add some extra configuration:

```bash
logging.level.org.springframework.web=DEBUG logging.level.org.springframework.web.servlet.mvc.method.annotation.HttpEntityMethodProcessor=DEBUG
```

Request parameter resolution works by taking the query parameters, path variables, and headers and mapping them to the controller's method parameters. For example, if you have a controller like this:

```java
@PostMapping("/test") 
public String test(@RequestParam boolean active) {
  return "ok"; 
}
```

Spring will map "active=true" to the boolean active parameter. In the log, you will find something like:

```bash
Resolved argument [0] [type=boolean] = true.
```

The request body will also be converted from raw JSON into Java objects. Consider this controller service:

```java
@PostMapping("/test") 
public String test(@RequestBody User user) {    
  return "ok"; 
}
```

Spring will convert an incoming JSON body, like in the following example, into the User object argument:

In the log, you will see:

```bash
Reading [application/json] into [com.example.User]
```

You can also see how Spring processes the request into Java objects and, from Java objects, returns the response as JSON. Spring uses HTTP message converter objects to do this. In the log, you will see something like:

Writing \[application/json\] with MappingJackson2HttpMessageConverter

## Example With a Simple REST Controller

As an example, consider a simple REST service:

```java
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import java.util.Map;

@RestController
@RequestMapping("/api")
public class DemoController {

    @PostMapping("/test")
    public Map<String, Object> test(@RequestBody Map<String, Object> body) {
        return Map.of(
            "message", "Request received",
            "data", body
        );
    }
}
```

The above service accepts a JSON payload and returns it in the response. To test this API endpoint, you can use a curl command:

```bash
curl -X POST http://localhost:8080/api/test \ -H "Content-Type: application/json" \ -d '{"name":"Miriam","age":32}'
```

When the request reaches your application service and is elaborated, something like this will be logged:

```bash
REQUEST DATA: uri=/api/test;payload={"name":"Miriam","age":32}
```

## Further Considerations

**CommonsRequestLoggingFilter** has some limitations:

-   The request body is only logged if it is read by the application.
-   You have to limit large payloads, as they may impact performance.
-   You shouldn't log sensitive data in production.

If you want to log both requests and responses, you should implement a custom filter using **ContentCachingRequestWrapper**.

## Conclusion

Spring Boot provides an easy way to log HTTP requests using **CommonsRequestLoggingFilter**. You need just a few configuration settings. It's an essential tool for diagnosing problems and maintaining the REST APIs. In the context of microservice architectures, this improves the whole observability stack.

You find an example on [GitHub](https://github.com/mcasari/codingstrain/tree/main/spring-request-logging-demo "GitHub repository").
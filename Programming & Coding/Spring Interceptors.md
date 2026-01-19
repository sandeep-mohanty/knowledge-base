# New to Spring Boot? Interceptors Will Save You from Repeating Code

When building Spring Boot APIs, we often end up writing the same code again and again — logging requests, checking tokens, validating headers, measuring execution time. This not only makes controllers messy but also harder to maintain. What if there was a way to run common logic for every request without touching your controllers at all? That’s exactly where **Spring Boot Interceptors** come in — your silent gatekeepers that stand between the client and your controller.

### What is an Interceptor in Spring Boot?
Interceptor = a gatekeeper for your HTTP requests

It allows you to:
* Run some code **BEFORE** controller method is called
* Run some code **AFTER** controller method is executed
* Run some code **AFTER** response is sent to client

So every request that comes to your API can be **checked, modified, logged, or blocked.**

### Real Life Example
Think of **company security at the gate**:
* Employee comes → security checks ID (before entry)
* Employee goes inside → does work
* Employee leaves → security notes exit time (after work)

Security = **Interceptor**
Office = **Controller**

### Where Does Interceptor Sit in Request Flow?
Request flow in Spring Boot:

Client

 ↓

DispatcherServlet

  ↓

Interceptor (preHandle)

  ↓

Controller Method

  ↓

Interceptor (postHandle)

  ↓

View / Response

  ↓

Interceptor (afterCompletion)

  ↓

Client

![](https://miro.medium.com/v2/resize:fit:720/format:webp/1*5eilpMfzIjIeJsliIu8XMw.png)

### How Interceptor Works (3 Important Methods)
We create a class that implements:
**HandlerInterceptor**

It gives 3 methods:

#### 1 preHandle() → Before Controller
**boolean preHandle(request, response, handler)**
Runs **before** controller method.
Used for:
* Token validation
* Reject request
* Start timer

If it returns:
* **true** → request continues
* **false** → request is blocked
Very powerful.

#### 2 postHandle() → After Controller, Before Response
Runs **after** controller logic, but before response is sent.

Used for:
* Modify response model
* Add extra data
(Not much used in REST APIs)

#### 3 afterCompletion() → After Response Sent
Runs after everything is finished.

Used for:
* Logging
* Cleanup
* Stop timer

### Step-by-Step: How to Create Interceptor
Let’s build a **real example: API Execution Time Logger**

**Step 1: Create Interceptor Class**
```java
@Component
public class LoggingInterceptor implements HandlerInterceptor {

    private static final String START_TIME = "startTime";

    @Override
    public boolean preHandle(HttpServletRequest request,
                             HttpServletResponse response,
                             Object handler) {

        request.setAttribute(START_TIME, System.currentTimeMillis());
        System.out.println("Incoming request: " + request.getRequestURI());
        return true; // allow request
    }

    @Override
    public void afterCompletion(HttpServletRequest request,
                                HttpServletResponse response,
                                Object handler,
                                Exception ex) {

        long startTime = (long) request.getAttribute(START_TIME);
        long timeTaken = System.currentTimeMillis() - startTime;

        System.out.println("Request completed: " + request.getRequestURI()
                + " Time Taken: " + timeTaken + " ms");
    }
}
```

**Step 2: Register Interceptor (Very Important)**
Interceptor will not work unless you register it.

Create config class:
```java
@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Autowired
    private LoggingInterceptor loggingInterceptor;

    @Override
    public void addInterceptors(InterceptorRegistry registry) {

        registry.addInterceptor(loggingInterceptor)
                .addPathPatterns("/api/**")   // apply only to APIs
                .excludePathPatterns("/api/login"); // skip login
    }
}
```
Now interceptor will work for:
* /api/products
* /api/orders
But not for:
* /api/login

### Real-Time Use Case #1: JWT Token Validation
**Problem:**
Every API must check token.

**Solution:**
Do it in **Interceptor** instead of every controller.

**preHandle() Example**
```java
@Override
public boolean preHandle(HttpServletRequest request,
                         HttpServletResponse response,
                         Object handler) throws Exception {

    String token = request.getHeader("Authorization");

    if (token == null || !token.startsWith("Bearer ")) {
        response.setStatus(HttpStatus.UNAUTHORIZED.value());
        return false; // block request
    }

    // validate token here
    boolean valid = jwtService.validate(token);

    if (!valid) {
        response.setStatus(HttpStatus.UNAUTHORIZED.value());
        return false;
    }

    return true;
}
```
Now:
✔ Token check happens in **one place**
✔ Controllers remain clean
✔ Security logic centralized

### When to use what?
* **Encoding, CORS** → Filter
* **Auth, logging, metrics** → Interceptor
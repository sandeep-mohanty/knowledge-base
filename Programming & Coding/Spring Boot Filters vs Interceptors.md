# Spring Boot Filters vs Interceptors: Which One Should You Really Be Using?

## The Fundamental Concepts
Before diving into the differences, let’s understand what each component actually is and where it lives in the Spring Boot ecosystem.

### What are Filters?
Filters are part of the Java Servlet specification (`javax.servlet.Filter` or `jakarta.servlet.Filter` in newer versions). They’ve been around since the early days of Java web applications and operate at the servlet container level — before requests even reach the Spring framework.

Think of Filters as security guards at the entrance of a building. They check everyone who comes in, regardless of which office they’re visiting. Filters operate on the raw HTTP request and response objects and have no awareness of Spring’s MVC context.

**Key characteristics of Filters:**
* Part of the Java Servlet API, not Spring-specific
* Execute before the `DispatcherServlet`
* Work with raw `ServletRequest` and `ServletResponse`
* Limited access to Spring context
* Configured using `@Component` or `FilterRegistrationBean`

### What are Interceptors?
Interceptors, on the other hand, are a Spring MVC concept. They operate within the Spring framework after the `DispatcherServlet` has processed the request but before (and after) your controller methods execute.

Using our building analogy, Interceptors are like specialized assistants on each floor who know exactly which office you’re visiting and can provide context-specific help. They have full access to Spring’s ecosystem and understand the MVC workflow.

**Key characteristics of Interceptors:**
* Part of Spring MVC framework
* Execute after `DispatcherServlet`, before controllers
* Work with `HttpServletRequest` and `HttpServletResponse`
* Full access to Spring context and handler information
* Configured through `WebMvcConfigurer`

---

## The Execution Order: A Critical Understanding
One of the most important concepts to grasp is the execution order. Understanding this will help you choose the right tool for your specific use case.



```text
Incoming Request
    ↓
[Filter 1] → preHandle
    ↓
[Filter 2] → preHandle
    ↓
DispatcherServlet
    ↓
[Interceptor 1] → preHandle()
    ↓
[Interceptor 2] → preHandle()
    ↓
Controller Method Execution
    ↓
[Interceptor 2] → postHandle()
    ↓
[Interceptor 1] → postHandle()
    ↓
View Rendering (if applicable)
    ↓
[Interceptor 1] → afterCompletion()
    ↓
[Interceptor 2] → afterCompletion()
    ↓
[Filter 2] → postHandle
    ↓
[Filter 1] → postHandle
    ↓
Response to Client
```

This execution flow has profound implications for how you design your cross-cutting concerns.

---

## Real-World Scenario 1: Request Logging with Filters
Let’s start with a practical example where Filters shine. Imagine you need to log every single HTTP request that hits your application, including static resources and requests that might not even reach Spring MVC.

**When to use Filters for logging:**
* You need to log ALL requests, including static resources
* You want to measure the total time from request arrival to response
* You need to modify the request/response before Spring processes it
* You’re implementing infrastructure-level concerns

**Here’s a production-ready request logging Filter:**

```java
@Component
@Order(1)
public class RequestLoggingFilter implements Filter {
    
    private static final Logger logger = LoggerFactory.getLogger(RequestLoggingFilter.class);
    private static final DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss.SSS");

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
            throws IOException, ServletException {
        
        HttpServletRequest httpRequest = (HttpServletRequest) request;
        HttpServletResponse httpResponse = (HttpServletResponse) response;
        
        // Generate unique request ID for tracing
        String requestId = UUID.randomUUID().toString();
        long startTime = System.currentTimeMillis();
        
        // Log incoming request
        logger.info("→ [{}] {} {} - Started at {}",
            requestId,
            httpRequest.getMethod(),
            httpRequest.getRequestURI(),
            LocalDateTime.now().format(formatter)
        );
        
        try {
            // Continue the filter chain
            chain.doFilter(request, response);
        } finally {
            // Log response - This always executes, even if an exception occurs
            long duration = System.currentTimeMillis() - startTime;
            logger.info("← [{}] {} {} - Completed in {}ms - Status: {}",
                requestId,
                httpRequest.getMethod(),
                httpRequest.getRequestURI(),
                duration,
                httpResponse.getStatus()
            );
        }
    }
}
```

**Why a Filter works well here:**
* Captures the total request lifecycle time
* Logs requests that don’t reach Spring MVC (like 404s for static resources)
* Simple and doesn’t need Spring context
* Executes for every single request

---

## Real-World Scenario 2: Authentication with Interceptors
Now let’s look at a scenario where Interceptors are the superior choice: implementing JWT-based authentication that needs to interact with Spring services.

**When to use Interceptors for authentication:**
* You need to inject Spring beans (services, repositories)
* You want to access controller method information
* You need to modify the `ModelAndView`
* You’re implementing business logic-level concerns

**Here’s a production-grade authentication Interceptor:**

```java
@Component
public class AuthenticationInterceptor implements HandlerInterceptor {
    
    private final JwtTokenService jwtTokenService;
    private final UserService userService;
    private final AuditService auditService;
    
    @Autowired
    public AuthenticationInterceptor(JwtTokenService jwtTokenService,
                                    UserService userService,
                                    AuditService auditService) {
        this.jwtTokenService = jwtTokenService;
        this.userService = userService;
        this.auditService = auditService;
    }

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler)
            throws Exception {
        
        // Only process actual controller methods, not static resources
        if (!(handler instanceof HandlerMethod)) {
            return true;
        }
        
        HandlerMethod handlerMethod = (HandlerMethod) handler;
        Method method = handlerMethod.getMethod();
        
        // Check if method requires authentication
        if (method.isAnnotationPresent(PublicEndpoint.class)) {
            return true; // Public endpoint, no authentication needed
        }
        
        // Extract JWT token from header
        String authHeader = request.getHeader("Authorization");
        if (authHeader == null || !authHeader.startsWith("Bearer ")) {
            response.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
            response.getWriter().write("{\"error\": \"Missing or invalid Authorization header\"}");
            return false;
        }
        
        String token = authHeader.substring(7);
        
        // Validate token using Spring service
        if (!jwtTokenService.validateToken(token)) {
            response.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
            response.getWriter().write("{\"error\": \"Invalid or expired token\"}");
            return false;
        }
        
        // Extract user from token and load full user details
        String userId = jwtTokenService.getUserIdFromToken(token);
        User user = userService.findById(userId)
            .orElseThrow(() -> new RuntimeException("User not found"));
        
        // Store user in request for controller access
        request.setAttribute("authenticatedUser", user);
        
        // Log authentication event
        auditService.logAuthentication(user, request.getRequestURI());
        
        return true;
    }
}
```

**Why an Interceptor works better here:**
* Easy dependency injection with Spring services
* Access to handler method information
* Can check for custom annotations
* Works only on Spring MVC requests, not static resources
* Cleaner separation of concerns

---

## Hands-On: Building a Complete Example
Let me show you a complete, working example that demonstrates both Filters and Interceptors in action. This example implements a realistic API with:
1. Rate limiting (Filter)
2. Authentication (Interceptor)
3. Authorization based on roles (Interceptor)
4. Request/response logging (Filter and Interceptor)
5. Performance metrics (Interceptor)

### Step 1: Setting Up Custom Annotations
First, let’s create some custom annotations for our Interceptors:

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface PublicEndpoint {
}

@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface RequireRole {
    String[] value();
}
```

### Step 2: Creating the Rate Limiting Filter
```java
@Component
@Order(1)
public class RateLimitFilter implements Filter {
    
    private static final int MAX_REQUESTS_PER_MINUTE = 100;
    private final ConcurrentHashMap<String, AtomicInteger> requestCounts = new ConcurrentHashMap<>();
    private final ConcurrentHashMap<String, Long> resetTimes = new ConcurrentHashMap<>();

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
            throws IOException, ServletException {
        
        HttpServletRequest httpRequest = (HttpServletRequest) request;
        HttpServletResponse httpResponse = (HttpServletResponse) response;
        
        String clientIp = getClientIp(httpRequest);
        long currentTime = System.currentTimeMillis();
        
        // Reset counter if minute has passed
        Long resetTime = resetTimes.get(clientIp);
        if (resetTime == null || currentTime > resetTime) {
            requestCounts.put(clientIp, new AtomicInteger(0));
            resetTimes.put(clientIp, currentTime + 60000); // Reset after 1 minute
        }
        
        // Check rate limit
        AtomicInteger count = requestCounts.get(clientIp);
        if (count.incrementAndGet() > MAX_REQUESTS_PER_MINUTE) {
            httpResponse.setStatus(HttpServletResponse.SC_TOO_MANY_REQUESTS);
            httpResponse.setContentType("application/json");
            httpResponse.getWriter().write(
                "{\"error\": \"Rate limit exceeded\", \"maxRequests\": " + MAX_REQUESTS_PER_MINUTE + "}"
            );
            return; // Don't continue the chain
        }
        
        chain.doFilter(request, response);
    }
    
    private String getClientIp(HttpServletRequest request) {
        String ip = request.getHeader("X-Forwarded-For");
        if (ip == null || ip.isEmpty()) {
            ip = request.getRemoteAddr();
        }
        return ip;
    }
}
```

### Step 3: Creating the Authorization Interceptor
```java
@Component
public class AuthorizationInterceptor implements HandlerInterceptor {

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler)
            throws Exception {
        
        if (!(handler instanceof HandlerMethod)) {
            return true;
        }
        
        HandlerMethod handlerMethod = (HandlerMethod) handler;
        RequireRole roleAnnotation = handlerMethod.getMethodAnnotation(RequireRole.class);
        
        if (roleAnnotation == null) {
            return true; // No role requirement
        }
        
        // Get authenticated user from request (set by AuthenticationInterceptor)
        User user = (User) request.getAttribute("authenticatedUser");
        if (user == null) {
            response.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
            response.getWriter().write("{\"error\": \"Authentication required\"}");
            return false;
        }
        
        // Check if user has required role
        String[] requiredRoles = roleAnnotation.value();
        boolean hasRequiredRole = Arrays.stream(requiredRoles)
            .anyMatch(role -> user.getRoles().contains(role));
        
        if (!hasRequiredRole) {
            response.setStatus(HttpServletResponse.SC_FORBIDDEN);
            response.getWriter().write("{\"error\": \"Insufficient permissions\"}");
            return false;
        }
        
        return true;
    }
}
```

### Step 4: Creating the Performance Monitoring Interceptor
```java
@Component
public class PerformanceMonitoringInterceptor implements HandlerInterceptor {
    
    private static final Logger logger = LoggerFactory.getLogger(PerformanceMonitoringInterceptor.class);
    private static final String START_TIME_ATTR = "startTime";

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) {
        request.setAttribute(START_TIME_ATTR, System.currentTimeMillis());
        return true;
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response,
                          Object handler, ModelAndView modelAndView) {
        long startTime = (Long) request.getAttribute(START_TIME_ATTR);
        long controllerTime = System.currentTimeMillis() - startTime;
        request.setAttribute("controllerTime", controllerTime);
        
        logger.debug("Controller execution time: {}ms", controllerTime);
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response,
                               Object handler, Exception ex) {
        long startTime = (Long) request.getAttribute(START_TIME_ATTR);
        long totalTime = System.currentTimeMillis() - startTime;
        Long controllerTime = (Long) request.getAttribute("controllerTime");
        
        if (controllerTime != null) {
            long viewRenderingTime = totalTime - controllerTime;
            logger.info("Request completed - Total: {}ms (Controller: {}ms, View: {}ms)",
                totalTime, controllerTime, viewRenderingTime);
        } else {
            logger.info("Request completed - Total: {}ms", totalTime);
        }
        
        // Log warning for slow requests
        if (totalTime > 1000) {
            logger.warn("SLOW REQUEST: {} {} took {}ms",
                request.getMethod(), request.getRequestURI(), totalTime);
        }
    }
}
```

### Step 5: Registering Interceptors
```java
@Configuration
public class WebMvcConfig implements WebMvcConfigurer {
    
    @Autowired
    private AuthenticationInterceptor authenticationInterceptor;
    
    @Autowired
    private AuthorizationInterceptor authorizationInterceptor;
    
    @Autowired
    private PerformanceMonitoringInterceptor performanceInterceptor;

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        // Performance monitoring runs for all requests
        registry.addInterceptor(performanceInterceptor)
                .addPathPatterns("/**");
        
        // Authentication runs for all API endpoints
        registry.addInterceptor(authenticationInterceptor)
                .addPathPatterns("/api/**");
        
        // Authorization runs after authentication
        registry.addInterceptor(authorizationInterceptor)
                .addPathPatterns("/api/**");
    }
}
```

### Step 6: Creating Sample Controllers
```java
@RestController
@RequestMapping("/api")
public class UserController {
    
    @PublicEndpoint
    @PostMapping("/login")
    public ResponseEntity<LoginResponse> login(@RequestBody LoginRequest request) {
        // Public endpoint - no authentication required
        // Authenticate user and generate token
        String token = jwtTokenService.generateToken(user.getId());
        return ResponseEntity.ok(new LoginResponse(token, user));
    }
    
    @GetMapping("/profile")
    public ResponseEntity<User> getProfile(HttpServletRequest request) {
        // Requires authentication (no @PublicEndpoint)
        User user = (User) request.getAttribute("authenticatedUser");
        return ResponseEntity.ok(user);
    }
    
    @RequireRole({"ADMIN", "MANAGER"})
    @DeleteMapping("/users/{id}")
    public ResponseEntity<Void> deleteUser(@PathVariable Long id) {
        // Requires authentication and ADMIN or MANAGER role
        // Implementation here
        return ResponseEntity.noContent().build();
    }
    
    @RequireRole("ADMIN")
    @PostMapping("/users/{id}/ban")
    public ResponseEntity<Void> banUser(@PathVariable Long id) {
        // Requires authentication and ADMIN role only
        // Implementation here
        return ResponseEntity.noContent().build();
    }
}
```

---

## The Decision Matrix: When to Use What
Based on years of experience building Spring Boot applications, here’s my decision matrix:



### Use Filters When:
1. **Infrastructure-Level Concerns**
    * Character encoding configuration
    * CORS headers (though Spring has better ways now)
    * Request/response compression
    * Request/response body wrapping for multiple reads
2. **Need to Process All Requests**
    * Including static resources
    * Including requests that don’t reach Spring MVC
    * 404s, errors before Spring
3. **Order Matters (Very Early Processing)**
    * Rate limiting at the entry point
    * Security headers that must be set first
    * Request ID generation for distributed tracing

### Use Interceptors When:
1. **Spring-Aware Logic**
    * Authentication with Spring Security integration
    * Authorization based on roles or permissions
    * Accessing Spring beans and services
2. **Controller-Specific Logic**
    * Need to check controller method annotations
    * Need to modify `ModelAndView`
    * Business logic cross-cutting concerns
3. **Granular Path Control**
    * Different behavior for different endpoint patterns
    * Easy inclusion/exclusion of paths
4. **Three-Phase Processing**
    * Before controller (`preHandle`)
    * After controller but before view (`postHandle`)
    * After everything completes (`afterCompletion`)

---

## Performance Considerations
One common question I get is: “Do Filters or Interceptors perform better?”

The truth is, the performance difference is negligible in most real-world applications. However, there are some considerations:

**Filters:**
* Slightly more overhead per request since they run for ALL requests
* Can’t be easily conditionally applied
* Process even requests that don’t reach Spring MVC

**Interceptors:**
* Only run for Spring MVC requests
* Can be conditionally applied to specific paths
* More efficient for business logic concerns

**My recommendation:** Don’t choose based on performance unless you’re operating at massive scale (millions of requests per second). Choose based on functionality and maintainability.

---

## Testing Strategies
Testing Filters and Interceptors requires different approaches. Let me show you how I test each.

### Testing a Filter
```java
@ExtendWith(MockitoExtension.class)
class RateLimitFilterTest {

    @Mock
    private HttpServletRequest request;

    @Mock
    private HttpServletResponse response;

    @Mock
    private FilterChain filterChain;

    @Mock
    private PrintWriter writer;

    @Test
    void shouldAllowRequestsUnderLimit() throws IOException, ServletException {
        RateLimitFilter filter = new RateLimitFilter();
        
        when(request.getRemoteAddr()).thenReturn("192.168.1.1");
        
        // Make 10 requests - all should pass
        for (int i = 0; i < 10; i++) {
            filter.doFilter(request, response, filterChain);
        }
        
        verify(filterChain, times(10)).doFilter(request, response);
        verify(response, never()).setStatus(HttpServletResponse.SC_TOO_MANY_REQUESTS);
    }

    @Test
    void shouldBlockRequestsOverLimit() throws IOException, ServletException {
        RateLimitFilter filter = new RateLimitFilter();
        
        when(request.getRemoteAddr()).thenReturn("192.168.1.1");
        when(response.getWriter()).thenReturn(writer);
        
        // Make 101 requests - last one should be blocked
        for (int i = 0; i < 101; i++) {
            filter.doFilter(request, response, filterChain);
        }
        
        verify(filterChain, times(100)).doFilter(request, response);
        verify(response, times(1)).setStatus(HttpServletResponse.SC_TOO_MANY_REQUESTS);
    }
}
```

### Testing an Interceptor
```java
@ExtendWith(MockitoExtension.class)
class AuthenticationInterceptorTest {

    @Mock
    private JwtTokenService jwtTokenService;

    @Mock
    private UserService userService;

    @Mock
    private AuditService auditService;

    @InjectMocks
    private AuthenticationInterceptor interceptor;

    @Mock
    private HttpServletRequest request;

    @Mock
    private HttpServletResponse response;

    @Mock
    private HandlerMethod handler;

    @Mock
    private PrintWriter writer;

    @Test
    void shouldAllowValidToken() throws Exception {
        String token = "valid-token";
        User user = new User("1", "john@example.com", "John Doe");
        
        when(request.getHeader("Authorization")).thenReturn("Bearer " + token);
        when(jwtTokenService.validateToken(token)).thenReturn(true);
        when(jwtTokenService.getUserIdFromToken(token)).thenReturn("1");
        when(userService.findById("1")).thenReturn(Optional.of(user));
        
        boolean result = interceptor.preHandle(request, response, handler);
        
        assertTrue(result);
        verify(request).setAttribute("authenticatedUser", user);
        verify(auditService).logAuthentication(user, null);
    }

    @Test
    void shouldRejectMissingToken() throws Exception {
        when(request.getHeader("Authorization")).thenReturn(null);
        when(response.getWriter()).thenReturn(writer);
        
        boolean result = interceptor.preHandle(request, response, handler);
        
        assertFalse(result);
        verify(response).setStatus(HttpServletResponse.SC_UNAUTHORIZED);
        verify(jwtTokenService, never()).validateToken(anyString());
    }

    @Test
    void shouldRejectInvalidToken() throws Exception {
        String token = "invalid-token";
        
        when(request.getHeader("Authorization")).thenReturn("Bearer " + token);
        when(jwtTokenService.validateToken(token)).thenReturn(false);
        when(response.getWriter()).thenReturn(writer);
        
        boolean result = interceptor.preHandle(request, response, handler);
        
        assertFalse(result);
        verify(response).setStatus(HttpServletResponse.SC_UNAUTHORIZED);
        verify(userService, never()).findById(anyString());
    }
}
```

---

## Conclusion
Filters and Interceptors are both powerful tools in the Spring Boot toolkit, but they serve different purposes. Filters operate at the servlet level and are ideal for infrastructure concerns, while Interceptors work within the Spring MVC context and excel at business logic cross-cutting concerns.

The key is understanding where each tool fits in the request lifecycle and choosing the right one for your specific use case. Don’t make the mistake I made years ago of using Filters for everything just because they seemed familiar. Embrace Interceptors for your Spring-aware logic, and reserve Filters for true infrastructure concerns.

**Remember: Use Filters for servlet-level concerns, use Interceptors for Spring MVC concerns.**

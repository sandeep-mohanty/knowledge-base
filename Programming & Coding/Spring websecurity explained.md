# @EnableWebSecurity vs @EnableMethodSecurity — Explained Like Never Before

## 1. What is `@EnableWebSecurity`?

`@EnableWebSecurity` is an annotation that:

- Registers the Spring Security filter chain  
- Disables the default auto-configuration (so you can customize security)  
- Allows you to configure authentication and authorization logic  

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {
   // your security config here
}
```

---

## 2. What Happens Internally When You Use `@EnableWebSecurity`?

When Spring Boot starts, the annotation triggers a series of internal mechanisms via Spring’s configuration system:

### 2.1 Internally It Imports
```java
@Import(WebSecurityConfiguration.class)
```
The `@EnableWebSecurity` annotation imports a class called `WebSecurityConfiguration`, which does the heavy lifting behind the scenes.

### 2.2 Registers a Filter Chain
Spring Security is filter-based.  
It creates a `SecurityFilterChain`, which is a chain of servlet filters to inspect every incoming HTTP request.  

The filter chain includes filters like:
- `UsernamePasswordAuthenticationFilter`
- `BasicAuthenticationFilter`
- `CsrfFilter`
- `ExceptionTranslationFilter`
- `SecurityContextPersistenceFilter`

All these filters are ordered and wrapped inside `FilterChainProxy`, which is finally registered in the servlet context.

### 2.3 Looks for a `SecurityFilterChain` Bean
In Spring Security 6+ (and Spring Boot 3+), you define a `SecurityFilterChain` bean manually instead of extending `WebSecurityConfigurerAdapter`.

```java
@Bean
public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
    http
        .authorizeHttpRequests(auth -> auth
            .requestMatchers("/public/**").permitAll()
            .anyRequest().authenticated())
        .httpBasic();
    return http.build();
}
```

---

## 3. How the Filter Chain Works

Here’s what happens step by step when a request comes in:

```
Client -----> DispatcherServlet -----> FilterChainProxy -----> Filters
                                                         |
                       ---------------- SecurityContextPersistenceFilter
                       ---------------- UsernamePasswordAuthenticationFilter
                       ---------------- ExceptionTranslationFilter
                       ---------------- ...
```

- `SecurityContextPersistenceFilter` loads the security context (from session or `SecurityContextHolder`)  
- `UsernamePasswordAuthenticationFilter` processes login (username/password)  
- `AuthorizationFilter` verifies access rules (like roles)  
- If all filters pass, request goes to the Controller  

---

## 5. Why You Need `@EnableWebSecurity`

Without `@EnableWebSecurity`, Spring Boot will:

- Use default security (formLogin with random password)  
- Auto-secure all endpoints  

With it, you gain full control, for example:

- Custom login pages  
- JWT filters  
- Role-based access  
- CSRF control  
- API gateway handling  

### Example with Custom Filter (e.g., JWT)

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .csrf().disable()
            .authorizeHttpRequests(authz -> authz
                .requestMatchers("/api/auth/**").permitAll()
                .anyRequest().authenticated()
            )
            .addFilterBefore(new JwtFilter(), UsernamePasswordAuthenticationFilter.class);

        return http.build();
    }
}
```

---

## What is `@EnableMethodSecurity`?

`@EnableMethodSecurity` enables method-level security in Spring applications, allowing you to use annotations like:

- `@PreAuthorize`  
- `@PostAuthorize`  
- `@Secured`  
- `@RolesAllowed`  

These annotations restrict access to specific methods based on user roles, permissions, or custom logic.

### Example: Why We Use It

```java
@Service
public class BankService {

    @PreAuthorize("hasRole('ADMIN')")
    public void approveLoan() {
        // Only ADMIN can approve
    }

    @PreAuthorize("#username == authentication.name")
    public void viewProfile(String username) {
        // Only the user themselves can view their profile
    }
}
```

Without `@EnableMethodSecurity`, these annotations are ignored!

---

## Internal Working of `@EnableMethodSecurity`

### Step-by-step Breakdown

#### 1. Annotation Usage
```java
@Configuration
@EnableMethodSecurity(prePostEnabled = true, securedEnabled = true)
public class SecurityConfig {
}
```
This tells Spring:  
“Please activate AOP (Aspect-Oriented Programming) to intercept method calls and apply security rules.”

---

#### 2. What it does internally?
It imports the class:
```java
@Import(MethodSecurityConfiguration.class)
```
That class does several important things.

---

#### 3. Registers `MethodSecurityInterceptor`
This is a key class. It’s an AOP interceptor that wraps around method calls.

```java
public class MethodSecurityInterceptor implements MethodInterceptor {
   public Object invoke(MethodInvocation mi) {
      // Get method details, user details
      // Evaluate @PreAuthorize expression
      // Check access
      // If allowed, proceed with method call
   }
}
```

It intercepts method calls and checks:
- What roles the user has  
- What the security annotation says  
- Whether to allow or deny the method call  

---

#### 4. Expression Evaluation with SpEL
If you’re using:
```java
@PreAuthorize("hasRole('ADMIN')")
```

Spring:
- Parses the SpEL expression (`hasRole('ADMIN')`)  
- Injects `Authentication` object (user details)  
- Evaluates it before calling the method  
- If the expression is true, method runs. Otherwise, throws `AccessDeniedException`  

---

## Example Flow Internally

You call a method like:
```java
bankService.approveLoan();
```

- Spring AOP intercepts this method (thanks to `@EnableMethodSecurity`)  
- `MethodSecurityInterceptor` reads `@PreAuthorize("hasRole('ADMIN')")`  
- It gets the current user’s roles from `SecurityContextHolder`  
- It evaluates the expression:  
  - If user has ADMIN → proceed  
  - Else → throw `AccessDeniedException`  

![](https://miro.medium.com/v2/resize:fit:720/format:webp/1*beaaC7_b2f6IJsEmq26ySA.png)
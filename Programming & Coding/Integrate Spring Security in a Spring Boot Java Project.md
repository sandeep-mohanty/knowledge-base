# How to Integrate Spring Security in a Spring Boot Java Project?

Security is the most critical aspect of any app or web development project. **Spring Security** helps keep the application safe from unwanted access. It checks who you are and what you are allowed to do.

With the basics of security in mind, let’s now integrate **Spring Security** into a **Spring Boot application** and explore how Spring Security and its security chain operate.

---

## What Is Spring Security?
**Spring Security** is a powerful framework for handling authentication and authorization in Java applications.

Two concepts are central here:
* **Authentication** is verifying who you are. It’s like showing your ID card at the airport. The system checks if you are really who you claim to be.
* **Authorization** is checking what you’re allowed to do. After confirming you’re a real person, the system checks if you’re allowed to access a specific area or perform a specific action.

Spring Security makes both of these tasks simple by providing pre-built components that handle the most common scenarios.

---

## Setting Up Spring Security

### Step 1: Add Dependencies
The first thing we need to do is tell Spring Boot that we want to use Spring Security. We can do this by adding a dependency to our project.

For Maven projects, add this to your `pom.xml` file:
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```
Once we add this, Spring Boot automatically enables security for our application. By default, all endpoints require authentication. This means we can not access any URL in the application without logging in.

### Step 2: Create a Basic Security Configuration
Now we need to tell Spring Security how we want it to work. The best way to do this is by creating a configuration class.

Here is a simple example of a `Configuration` class:
```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.Customizer;
import org.springframework.security.web.SecurityFilterChain;

@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(authorize -> authorize
                .requestMatchers("/public/**").permitAll()
                .anyRequest().authenticated()
            )
            .formLogin(Customizer.withDefaults())
            .httpBasic(Customizer.withDefaults());
        
        return http.build();
    }
}
```
* `@Configuration` tells Spring that this is a configuration class.
* `@EnableWebSecurity` activates Spring Security for our application.
* `SecurityFilterChain` is a chain of filters that processes every request.

**Understanding the code above:**
1.  URLs starting with `/public/` are accessible to everyone.
2.  All other URLs require the user to be authenticated (logged in).
3.  Users can log in using a form or HTTP Basic Authentication.

---

## Understanding Spring Security Filter Chain
The filter chain is the main concept in **Spring Security**. It is like a list of checkpoints that every request must pass through. Each checkpoint checks one specific thing.

If the check passes, the request moves to the next checkpoint. If it fails, the checkpoint stops the request.



### How the Filter Chain Works?
When a request comes into our application, here’s what happens:
1.  The request enters Spring Security’s `FilterChainProxy`.
2.  `FilterChainProxy` checks multiple security filter chains.
3.  The first filter chain that matches the request pattern is used.
4.  The request passes through each filter in that chain.
5.  Each filter does its job and passes the request to the next filter.
6.  Finally, the request reaches your application code.

### Key Filters in the Chain
Here are the main filters that Spring Security uses:
* **DisableEncodeUrlFilter:** Prevents URL encoding attacks.
* **SecurityContextHolderFilter:** Loads the security context from the session.
* **HeaderWriterFilter:** Adds security headers to the response.
* **CsrfFilter:** Protects against Cross-Site Request Forgery attacks.
* **UsernamePasswordAuthenticationFilter:** Handles form-based login.
* **BasicAuthenticationFilter:** Handles HTTP Basic authentication.
* **AnonymousAuthenticationFilter:** Marks requests from non-authenticated users.
* **ExceptionTranslationFilter:** Catches security exceptions and handles them.
* **AuthorizationFilter:** Checks if the user has permission to access the resource.

```text
Request comes in
    ↓
FilterChainProxy decides which chain to use
    ↓
Filter 1: DisableEncodeUrlFilter (checks for encoding attacks)
    ↓
Filter 2: SecurityContextHolderFilter (loads user info from session)
    ↓
Filter 3: HeaderWriterFilter (adds security headers)
    ↓
Filter 4: CsrfFilter (checks CSRF token)
    ↓
Filter 5: UsernamePasswordAuthenticationFilter (handles login)
    ↓
Filter 6: BasicAuthenticationFilter (handles basic auth)
    ↓
Filter 7: AnonymousAuthenticationFilter (marks anonymous users)
    ↓
Filter 8: ExceptionTranslationFilter (handles exceptions)
    ↓
Filter 9: AuthorizationFilter (checks permissions)
    ↓
Your application code handles the request
    ↓
Response goes back through filters
    ↓
Response sent to client
```

### Why Multiple Filter Chains?
We can have multiple filter chains for different parts of your application. Spring Security **enables the use of multiple filter chains** for different application areas.

For example:
* One chain for API endpoints (which use JWT tokens)
* Another chain for web pages (which use session cookies)

```java
@Configuration
@EnableWebSecurity
public class MultiChainSecurityConfig {

    @Bean
    @Order(1)
    public SecurityFilterChain apiFilterChain(HttpSecurity http) throws Exception {
        http
            .securityMatcher("/api/**")
            .authorizeHttpRequests(authorize -> authorize
                .anyRequest().authenticated()
            )
            .oauth2ResourceServer(oauth2 -> oauth2.jwt(Customizer.withDefaults()));
        
        return http.build();
    }

    @Bean
    @Order(2)
    public SecurityFilterChain webFilterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(authorize -> authorize
                .requestMatchers("/public/**").permitAll()
                .anyRequest().authenticated()
            )
            .formLogin(Customizer.withDefaults());
        
        return http.build();
    }
}
```
The `@Order` annotation tells Spring Security which chain should be checked first. Lower numbers are checked first.

---

## User Authentication Methods

### In-Memory Users
For testing or simple applications, we can define users directly in the code:
```java
@Bean
public UserDetailsService userDetailsService() {
    UserDetails admin = User.builder()
        .username("admin")
        .password("{bcrypt}$2a$10$slYQmyNdGzin7olVN3p5aOYCFVEXYxT1kSVA9nT3n2N5BakE8YzFC")
        .roles("ADMIN")
        .build();

    UserDetails user = User.builder()
        .username("user")
        .password("{bcrypt}$2a$10$G3DjL9E0p7vhR0e7g0F5MeLW.EKhLf/xEqRYqvKkNi5YN3J/L1LJG")
        .roles("USER")
        .build();

    return new InMemoryUserDetailsManager(admin, user);
}

@Bean
public PasswordEncoder passwordEncoder() {
    return PasswordEncoderFactories.createDelegatingPasswordEncoder();
}
```
The encrypted passwords above are examples. In real applications we should never hardcode passwords.

### Database Users
Most real applications store users in a database. Below is an example how to do that.

First, create a **User entity**:
```java
import javax.persistence.*;
import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.core.authority.SimpleGrantedAuthority;
import org.springframework.security.core.userdetails.UserDetails;
import java.util.Collection;
import java.util.Collections;

@Entity
@Table(name = "users")
public class User implements UserDetails {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(unique = true)
    private String username;
    
    private String password;
    
    private String role;
    
    private boolean enabled;

    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        return Collections.singletonList(new SimpleGrantedAuthority("ROLE_" + role));
    }

    @Override
    public boolean isAccountNonExpired() { return true; }

    @Override
    public boolean isAccountNonLocked() { return true; }

    @Override
    public boolean isCredentialsNonExpired() { return true; }

    @Override
    public boolean isEnabled() { return enabled; }

    // Getters and setters
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }
    public String getUsername() { return username; }
    public void setUsername(String username) { this.username = username; }
    public String getPassword() { return password; }
    public void setPassword(String password) { this.password = password; }
    public String getRole() { return role; }
    public void setRole(String role) { this.role = role; }
    public void setEnabled(boolean enabled) { this.enabled = enabled; }
}
```

Next, create a **repository**:
```java
import org.springframework.data.jpa.repository.JpaRepository;
import java.util.Optional;

public interface UserRepository extends JpaRepository<User, Long> {
    Optional<User> findByUsername(String username);
}
```

Then, create a **UserDetailsService**:
```java
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.core.userdetails.UsernameNotFoundException;
import org.springframework.stereotype.Service;

@Service
public class CustomUserDetailsService implements UserDetailsService {
    
    private final UserRepository userRepository;
    
    public CustomUserDetailsService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
    
    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        return userRepository.findByUsername(username)
            .orElseThrow(() -> new UsernameNotFoundException("User not found: " + username));
    }
}
```

Finally, we need to update the **security configuration**:
```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    private final CustomUserDetailsService userDetailsService;
    
    public SecurityConfig(CustomUserDetailsService userDetailsService) {
        this.userDetailsService = userDetailsService;
    }

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(authorize -> authorize
                .requestMatchers("/public/**").permitAll()
                .requestMatchers("/admin/**").hasRole("ADMIN")
                .anyRequest().authenticated()
            )
            .formLogin(Customizer.withDefaults())
            .httpBasic(Customizer.withDefaults());
        
        return http.build();
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }

    @Bean
    public AuthenticationProvider authenticationProvider() {
        DaoAuthenticationProvider provider = new DaoAuthenticationProvider();
        provider.setUserDetailsService(userDetailsService);
        provider.setPasswordEncoder(passwordEncoder());
        return provider;
    }

    @Bean
    public AuthenticationManager authenticationManager(AuthenticationConfiguration config) 
            throws Exception {
        return config.getAuthenticationManager();
    }
}
```

Spring Security provides a robust framework for protecting our spring-boot applications. By understanding the filter chain, authentication methods, and OAuth2/JWT integration, we can build secure applications with confidence.
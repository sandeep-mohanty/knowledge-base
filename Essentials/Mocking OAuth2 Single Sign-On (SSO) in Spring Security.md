# Mocking OAuth2 Single Sign-On (SSO) in Spring Security

In this tutorial, we’ll learn how to **mock an OAuth2 Single Sign-On (SSO)** login in a Spring Boot application using Spring Security. This is especially useful for integration testing without needing a real Identity Provider.

## Table of Contents

- [Overview](#overview)
- [Project Setup](#project-setup)
- [Securing the Application](#securing-the-application)
- [Mocking OAuth2 for Tests](#mocking-oauth2-for-tests)
- [Conclusion](#conclusion)
- [References](#references)

---

## Overview

Spring Security provides excellent support for OAuth2 login. However, during tests, we often want to bypass the actual OAuth2 flow and simulate an authenticated user.

In this tutorial, we’ll use Spring Security’s testing support to mock an OAuth2 login session.

---

## Project Setup

Create a Spring Boot application and add the necessary dependencies for OAuth2 and testing.

### Gradle Dependencies (Groovy DSL)

Add the following to your `build.gradle`:

```groovy
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-oauth2-client'
    implementation 'org.springframework.boot:spring-boot-starter-security'

    testImplementation 'org.springframework.boot:spring-boot-starter-test'
    testImplementation 'org.springframework.security:spring-security-test'
}
```

If you're using the Kotlin DSL (`build.gradle.kts`), let me know and I can provide that as well.

---

## Securing the Application

We'll create a simple controller that returns a welcome message and secure it using Spring Security.

### Sample Controller

```java
@RestController
public class WelcomeController {

    @GetMapping("/welcome")
    public String welcome(@AuthenticationPrincipal OAuth2User principal) {
        return "Welcome, " + principal.getAttribute("name");
    }
}
```

### Security Configuration

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(auth -> auth
                .anyRequest().authenticated()
            )
            .oauth2Login(Customizer.withDefaults());

        return http.build();
    }
}
```

At this point, any request to `/welcome` requires an OAuth2 login.

---

## Mocking OAuth2 for Tests

Instead of relying on a real OAuth2 provider during tests, we can use Spring Security's `SecurityMockMvcRequestPostProcessors` to mock the authentication.

### Sample Integration Test

```java
@SpringBootTest
@AutoConfigureMockMvc
public class WelcomeControllerIntegrationTest {

    @Autowired
    private MockMvc mockMvc;

    @Test
    public void whenUserAuthenticated_thenWelcomeMessageReturned() throws Exception {
        mockMvc.perform(get("/welcome")
                .with(oauth2Login().attributes(attrs -> attrs.put("name", "Baeldung User"))))
            .andExpect(status().isOk())
            .andExpect(content().string("Welcome, Baeldung User"));
    }
}
```

### Explanation

- `oauth2Login()` is provided by `spring-security-test` and mocks an OAuth2 login session.
- `.attributes(...)` allows us to set user attributes like `name`.

This allows us to test secured endpoints without hitting a real OAuth2 provider.

---

## Conclusion

Mocking OAuth2 login in Spring Boot integration tests can dramatically simplify your test setup.

Using Spring Security’s test support, you can:

- Simulate authenticated users
- Skip external OAuth2 provider setup
- Write fast and reliable integration tests

---

## References

- Baeldung Article: [Mock an OAuth2 Single Sign-On in Spring Security](https://www.baeldung.com/spring-oauth2-mock-sso)
- [Spring Security Documentation](https://docs.spring.io/spring-security/site/docs/current/reference/html5/#testing)
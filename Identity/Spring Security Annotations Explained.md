# Spring Security Annotations Explained with Examples (Spring Boot)

![Spring Security Annotations Header](https://miro.medium.com/v2/resize:fit:720/format:webp/1*gPNqBX0y_u9WEcyIB-BgUw.png)

In this article, we will discuss **Spring Security annotations** with simple examples. These annotations are used in **Spring Boot applications** to secure web applications.

Spring Security annotations allow you to define **who can access a method or API** and **what permissions are required**. They provide a simple and powerful way to handle security without writing complex code.

---

### Spring Security Annotations with Examples

First, to use Spring Security annotations in a Spring Boot application, we need to add the **Spring Security starter dependency**. This dependency provides all the required security features such as authentication and authorization.

If you created your project using Spring Initializr or an IDE like IntelliJ or STS, simply select Spring Security. Otherwise, add the following dependency to your `pom.xml` file:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

Now, let’s look at some of the most commonly used **Spring Security annotations**.

---

### 1. @EnableWebSecurity Annotation

`@EnableWebSecurity` is used to enable Spring Security in the application and allows us to define custom security rules.

In Spring Boot 3.x, **we configure security using a SecurityFilterChain bean** instead of extending `WebSecurityConfigurerAdapter`.

#### Spring Boot 2.7 (Old / Deprecated Approach)
```java
@Configuration
@EnableWebSecurity
public class MyWebSecurityConfiguration extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.inMemoryAuthentication()
            .withUser("user")
            .password("password")
            .roles("USER")
            .and()
            .withUser("admin")
            .password("password")
            .roles("USER", "ADMIN");
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
            .authorizeRequests()
                .antMatchers("/public/**").permitAll()
                .anyRequest().hasRole("USER")
            .and()
            .formLogin();
    }
}
```

#### Spring Boot 3.5 (Modern & Recommended Approach)
```java
@Configuration
@EnableWebSecurity
public class MyWebSecurityConfiguration {

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/public/**").permitAll()
                .anyRequest().hasRole("USER")
            )
            .formLogin(Customizer.withDefaults());

        return http.build();
    }

    @Bean
    public UserDetailsService userDetailsService(PasswordEncoder encoder) {
        UserDetails user = User.builder()
                .username("user")
                .password(encoder.encode("password"))
                .roles("USER")
                .build();

        UserDetails admin = User.builder()
                .username("admin")
                .password(encoder.encode("password"))
                .roles("USER", "ADMIN")
                .build();

        return new InMemoryUserDetailsManager(user, admin);
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
```
> **Note:** `@EnableGlobalMethodSecurity` is deprecated and removed in Spring Boot 3.x. Use **@EnableMethodSecurity** for new applications.

---

### 2. @EnableMethodSecurity Annotation

`@EnableMethodSecurity` is used to enable annotation-based method security in Spring Security (Spring Security 5.6+).

* **Replaces** the older `@EnableGlobalMethodSecurity`
* **Enables support for:**
    * `@PreAuthorize` / `@PostAuthorize`
    * `@PreFilter` / `@PostFilter`
* **Optional flags:**
    * `securedEnabled = true` → enables `@Secured`
    * `jsr250Enabled = true` → enables `@RolesAllowed`

This annotation should be placed on a `@Configuration` class.

---

### 3. @Secured Annotation

The `@Secured` annotation is used to restrict access to a method or class based on user roles. Only users with the specified roles are allowed to access the method.

* You can specify one or more roles as a list.
* The role names must start with **ROLE_**.
* `@Secured` **does not support** Spring Expression Language (SpEL).

```java
@Secured({ "ADMIN", "ROLE_SUPER_ADMIN" })
public void createUser(User user) {
    // logic to create a user
}
```

---

### 4. @PreAuthorize Annotation

The `@PreAuthorize` annotation allows you to control access to methods or classes using Spring Expression Language (SpEL).

* The expression must evaluate to **true** for the method to be executed.
* You can use it on **individual methods** or **entire classes**.
* This is more powerful than `@Secured` because it supports complex expressions like checking multiple roles or conditions.

```java
@PreAuthorize("hasRole('ADMIN')")
public void deleteAdminUser(Long userId) {
    // Logic to delete a user
}
```

---

### 5. @PostAuthorize Annotation

The `@PostAuthorize` annotation allows you to control access **after** a method has executed, based on the returned object.

* The expression is written in **Spring Expression Language (SpEL)**.
* This is useful when access depends on the **method result**.

```java
@PostAuthorize("hasRole('ADMIN')")
public List<Course> findAllCourses() {
    // Logic to find all courses
    return courseRepository.findAll();
}
```

---

### 6. @PreFilter Annotation

`@PreFilter` is used to filter the items in a collection parameter before a method is executed.

* It is useful when a method receives a **list of objects** and you want to **allow only certain objects** to be processed based on security rules.
* The expression is written in **Spring Expression Language (SpEL)**.

```java
@PreFilter("hasRole('ADMIN')")
public void deleteCourses(List<Course> courses) {
    // Logic to delete courses
    // Only courses that pass the filter will be processed
}
```

---

### 7. @PostFilter Annotation

`@PostFilter` is used to filter the items in a collection after a method has executed.

* It is useful when a method returns a **list of objects** and you want to **only allow certain items** to be returned based on security rules.
* The expression is written in **Spring Expression Language (SpEL)**.

```java
@PostFilter("hasRole('ADMIN')")
public List<Class> findAllClasses() {
    // Logic to fetch all classes
    return classRepository.findAll();
}
```
---

### 8. @RolesAllowed Annotation

`@RolesAllowed` is a JSR-250 standard annotation used to restrict access to methods or classes based on roles.

* You can specify **one or more roles** that are allowed to access the method.
* Unlike `@Secured`, it is standardized, so other security frameworks (like Apache Shiro) also support it.
* **Does not support SpEL.**

---

### 9. @AuthenticationPrincipal Annotation

`@AuthenticationPrincipal` is used to inject the currently authenticated user directly into a controller or method parameter.

* You can access the user details without manually calling `SecurityContextHolder`.
* Commonly used in **REST controllers** or any Spring Security context.
* The injected object is usually your **UserDetails** or custom user object.

---

### 10. Role Hierarchy in Spring Security

RoleHierarchy allows you to define hierarchical roles, so that higher roles automatically include the privileges of lower roles.

* **Example:** if `ROLE_ADMIN > ROLE_USER`, an admin automatically has all permissions of a user.
* This reduces the number of `@PreAuthorize` or `@RolesAllowed` annotations you need to write.
* You configure **RoleHierarchy** as a Spring bean, not as a method annotation.

---

In summary, Spring Security annotations provide an easy and declarative way to control access to your application’s methods and endpoints. Using annotations like `@PreAuthorize`, `@RolesAllowed`, or `@Secured`, you can secure controllers, services, and REST APIs without writing complex security logic.
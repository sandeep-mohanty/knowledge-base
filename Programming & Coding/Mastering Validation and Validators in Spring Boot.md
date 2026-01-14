# Mastering Validation and Validators in Spring Boot — The Complete Guide..


Hey everyone 👋,
Welcome back to another deep-dive Spring Boot article! Today, we’ll explore one of the most **important and commonly used features in real-world backend applications** — **Validation** and the **Validator mechanism** in Spring Boot.

Whether you’re new to Spring Boot or an experienced developer revising key concepts, this article will help you understand not just **how** validation works, but also **why** it’s essential, **how it works under the hood**, and **how you can extend it for custom use cases**.

### 💡 What is Validation in Spring Boot?
Validation is the process of ensuring that the data coming into your application is correct, consistent, and meets the expected rules or constraints.

In other words:
**Validation ensures that only valid and meaningful data enters your system.**

**Example:**
* A user’s email should be a valid email format.
* Age cannot be negative.
* Password must be at least 8 characters long.

If we don’t validate data, we risk **corrupted data**, **unexpected behavior**, and even **security vulnerabilities**.

### ⚙️ Why Do We Need Validation?
Imagine a Spring Boot REST API that receives a **User** object:

```json
{
  "name": "",
  "email": "invalid-email",
  "age": -10
}
```

**Without validation:**
* You might save this invalid data directly into the database.
* It could lead to runtime exceptions or logic errors later.

**With validation:**
* You catch these errors **before** they reach your service or database layers.
* You provide clear error messages to the client.
* You maintain **data integrity and reliability**.

### 🧩 Spring Boot Validation — Under the Hood
Spring Boot integrates with **Bean Validation API (JSR-380)**, which is implemented by **Hibernate Validator** by default.

You can use annotations like:
* `@NotNull`
* `@NotEmpty`
* `@Email`
* `@Size`
* `@Pattern`
* `@Min`, `@Max`
* `@Past`, `@Future`

and more from the package:
`javax.validation.constraints.*` or `jakarta.validation.constraints.*` (depending on version).

Spring Boot automatically triggers validation when:
1. You annotate your model with these constraint annotations, and
2. Use `@Valid` or `@Validated` in your controller or service methods.

### 🧱 Example 1: Basic Field Validation
Let’s take an example of a **User** entity that we want to validate before saving.

```java
import jakarta.validation.constraints.*;

public class User {
    @NotBlank(message = "Name cannot be blank")
    private String name;
    
    @Email(message = "Email must be valid")
    private String email;
    
    @Min(value = 18, message = "Age must be at least 18")
    private int age;
    
    // getters and setters
}
```

Now, in your Controller:
```java
@RestController
@RequestMapping("/users")
public class UserController {

    @PostMapping
    public ResponseEntity<String> createUser(@Valid @RequestBody User user, BindingResult result) {
        if (result.hasErrors()) {
            String errorMessage = result.getAllErrors()
                    .stream()
                    .map(ObjectError::getDefaultMessage)
                    .collect(Collectors.joining(", "));
            return ResponseEntity.badRequest().body(errorMessage);
        }
        return ResponseEntity.ok("User created successfully!");
    }
}
```

**Explanation:**
* `@Valid` triggers validation on the request body.
* `BindingResult` captures the validation errors.
* If errors exist, you can extract and send a readable message to the user.

### 🔄 Example 2: Using @Validated at Class or Method Level
`@Validated` works similarly to `@Valid`, but it also allows **group-based validation**.

**Example:**
```java
@Validated
@RestController
@RequestMapping("/users")
public class UserController {

    @PostMapping("/create")
    public String addUser(@Valid @RequestBody User user) {
        return "User added successfully!";
    }
}
```
Both `@Valid` and `@Validated` work in most cases — the difference appears when you deal with **validation groups** or **method-level validation**.

### 🧰 Example 3: Custom Validator in Spring Boot
Sometimes, the built-in annotations aren’t enough.

For example, you may want to ensure:
* Username doesn’t contain spaces
* Password follows a complex rule (uppercase, lowercase, digit, special char)

Let’s create a **custom validator** step-by-step.

**Step 1: Create a Custom Annotation**
```java
@Documented
@Constraint(validatedBy = PasswordValidator.class)
@Target({ ElementType.METHOD, ElementType.FIELD })
@Retention(RetentionPolicy.RUNTIME)
public @interface ValidPassword {
    String message() default "Invalid password format!";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
}
```

**Step 2: Create the Validator Class**
```java
public class PasswordValidator implements ConstraintValidator<ValidPassword, String> {

    private static final String PASSWORD_PATTERN =
            "^(?=.*[0-9])(?=.*[a-z])(?=.*[A-Z])(?=.*[@#$%^&+=]).{8,}$";
            
    @Override
    public boolean isValid(String password, ConstraintValidatorContext context) {
        if (password == null) return false;
        return password.matches(PASSWORD_PATTERN);
    }
}
```

**Step 3: Use the Custom Annotation**
```java
public class User {
    @ValidPassword
    private String password;
}
```
Now, whenever you pass an invalid password, the message **"Invalid password format!"** will be returned.

### 🌍 Real-World Example: Validating DTOs in a REST API
Imagine you have a **RegistrationRequest** DTO for user signup.

```java
public class RegistrationRequest {
    @NotBlank(message = "Username cannot be blank")
    private String username;

    @Email(message = "Email is invalid")
    private String email;
    
    @ValidPassword
    private String password;
    
    @Pattern(regexp = "\\d{10}", message = "Mobile number must be 10 digits")
    private String mobile;
}
```

Now in your controller:
```java
@RestController
@RequestMapping("/api/auth")
public class AuthController {

    @PostMapping("/register")
    public ResponseEntity<String> registerUser(@Valid @RequestBody RegistrationRequest request) {
        // save user
        return ResponseEntity.ok("User registered successfully!");
    }
}
```
If the client sends invalid data, Spring Boot automatically responds with a **400 Bad Request** and validation error message.

### ⚙️ Global Exception Handling for Validation
Instead of checking `BindingResult` manually in every controller, you can centralize validation error handling using `@ControllerAdvice`.

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<Map<String, String>> handleValidationErrors(MethodArgumentNotValidException ex) {
        Map<String, String> errors = new HashMap<>();
        ex.getBindingResult().getFieldErrors().forEach(error ->
            errors.put(error.getField(), error.getDefaultMessage()));
        return ResponseEntity.badRequest().body(errors);
    }
}
```
Now your API will always return a neat JSON response like:
```json
{
  "email": "Email is invalid",
  "password": "Invalid password format!"
}
```

### 🧠 Behind the Scenes — How Validation Works
When you annotate your DTO with `@Valid`, Spring Boot internally:

1. Uses Hibernate Validator to validate bean constraints.
2. Throws `MethodArgumentNotValidException` if validation fails.
3. The `@ControllerAdvice` or `BindingResult` captures this.
4. A proper HTTP 400 response is sent back.

This mechanism makes it extremely easy to integrate validation seamlessly with REST APIs.



### ✅ Best Practices for Validation in Spring Boot
* Use **DTOs (Data Transfer Objects)** for validation instead of entities.
* Always provide **meaningful error messages**.
* Keep custom validators reusable.
* Use `@ControllerAdvice` for consistent error handling.
* Avoid putting business logic inside validator classes.
* Prefer `@Validated` when you need method-level or group-based validation.
* Combine validation with **Exception Handling** for clean and maintainable code.

### ❓Common Interview / Practical Questions
**Q1: What’s the difference between @Valid and @Validated?**
✅ `@Valid` is from JSR-303 and focuses on object graph validation.
✅ `@Validated` is a Spring-specific annotation that supports validation groups and method-level validation.

**Q2: What is Hibernate Validator?**
✅ It’s the reference implementation of the Bean Validation API (JSR 380) used by Spring Boot internally.

**Q3: How can you create a custom validator?**
✅ By defining a custom annotation (`@Constraint`) and implementing the `ConstraintValidator` interface.

**Q4: Can you validate method parameters or service-layer inputs?**
✅ Yes. By adding `@Validated` on the service class and validation annotations on method parameters.

**Q5: How do you handle validation errors globally?**
✅ Use `@ControllerAdvice` with an `@ExceptionHandler` for `MethodArgumentNotValidException`.

And that’s it! 🎉
You now understand everything about **Validation and Validator in Spring Boot** — from the basic annotations like `@NotNull` to building **custom validators**, handling **exceptions globally**, and following **best practices** used in real projects.

Validation may seem simple at first glance, but mastering it ensures your API is **robust, secure, and cleanly structured**.
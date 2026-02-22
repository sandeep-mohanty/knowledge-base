# Spring Boot Validation: Everything You Need to Know in One Place
**Master Jakarta Bean Validation and Hibernate extensions to keep your code clean, your data safe, and your frontend developers happy.**

Imagine your application is a secure club. You wouldn’t let just anyone walk in without checking their ID, right? Validation acts as the bouncer for your data, ensuring everything entering your system follows the rules. This protects your database from corruption and maintains data integrity.

---

## Bean Validation (Jakarta Validation) Architecture

Spring Boot doesn’t invent validation from scratch; it uses the standard **Jakarta Validation** (formerly Bean Validation) specification. This standard defines a set of annotations and a processing engine to check them.



### Spring Boot Validation Auto-configuration
When you include the `spring-boot-starter-validation` dependency, Spring automatically configures the validation engine for you.

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```

---

## Built-in Validation Annotations

### Core Jakarta Validation Annotations
These live in the `jakarta.validation.constraints` package:
* **@NotNull**: The value cannot be null.
* **@NotEmpty**: Not null and size > 0 (works for collections/strings).
* **@NotBlank**: Not null and contains at least one non-whitespace character (essential for Strings).
* **@Min / @Max**: Number must be ≥ or ≤ the value.
* **@Size(min, max)**: Checks the length of strings or collections.
* **@Email**: Validates the format of an email address.
* **@Past / @Future**: Validates dates relative to "now".

```java
public class UserDto {
    @NotBlank(message = "Username is required")
    private String username;

    @Email
    private String email;
    
    @Min(18)
    private int age;
}
```

### Hibernate Validator Extensions
Hibernate Validator is the default implementation and provides extra useful annotations:
* **@Range(min, max)**: A convenient combo of Min and Max.
* **@URL**: Checks if a string is a valid URL.
* **@CreditCardNumber**: Validates credit card formats using the Luhn check.

---

## Validation Groups
Sometimes you need different rules for different scenarios (e.g., ID is null on **Create** but required on **Update**). Validation Groups allow you to categorize constraints.

```java
public interface OnCreate {}
public interface OnUpdate {}

public class ProductDto {
    @Null(groups = OnCreate.class)
    @NotNull(groups = OnUpdate.class)
    private Long id;
}
```

---

## Where Validation Can Be Applied

| Layer | Trigger Annotation | Common Use Case |
| :--- | :--- | :--- |
| **REST Controllers** | `@Valid` or `@Validated` | Validating incoming JSON RequestBody. |
| **Service Layer** | `@Validated` | Enforcing rules on internal method parameters. |
| **Kafka Listeners** | `@Valid` on `@Payload` | Preventing bad messages from corrupting downstream systems. |
| **Configuration** | `@Validated` | Validating `application.properties` values at startup. |



---

## How to Handle Validation Errors

By default, Spring Boot throws a `MethodArgumentNotValidException` when validation fails. To take control, use a global exception handler with `@ControllerAdvice`.

```java
@RestControllerAdvice
public class GlobalExceptionHandler {
    
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<Map<String, String>> handleValidationExceptions(
            MethodArgumentNotValidException ex) {
        
        Map<String, String> errors = new HashMap<>();
        ex.getBindingResult().getAllErrors().forEach((error) -> {
              String fieldName = ((FieldError) error).getField();
              String errorMessage = error.getDefaultMessage();
              errors.put(fieldName, errorMessage);
        });

        return ResponseEntity.badRequest().body(errors);
    }
}
```

---

## Creating Custom Validation Annotations

A custom validation consists of two parts: the **Annotation** and the **Validator** logic.

### Step 1: The Annotation
```java
@Target({ ElementType.FIELD })
@Retention(RetentionPolicy.RUNTIME)
@Constraint(validatedBy = UpperCaseValidator.class)
public @interface AllUpperCase {
    String message() default "Must be all upper case";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
}
```

### Step 2: The Validator
```java
public class UpperCaseValidator implements ConstraintValidator<AllUpperCase, String> {
    @Override
    public boolean isValid(String value, ConstraintValidatorContext context) {
        if (value == null) return true; 
        return value.equals(value.toUpperCase());
    }
}
```

---

## Advanced Topics

* **Cross-Field Validation**: Used for logic like "startDate must be before endDate". This is typically handled by a class-level custom annotation.
* **Programmatic Validation**: You can inject the `Validator` interface and call `validator.validate(object)` manually inside your business logic.
* **Performance**: Validation uses reflection. While the cost is negligible for web apps, avoid custom validators that trigger expensive database queries inside loops.

---

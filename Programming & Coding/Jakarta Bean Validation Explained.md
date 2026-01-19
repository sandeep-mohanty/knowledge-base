# Jakarta Bean Validation Explained: A Deep Dive into Mastering Java Bean Validation

When building Java applications, it’s important to make sure that the data coming into your system follows certain rules and limits. This process is called validation. Jakarta Bean Validation 3.0, part of the Jakarta EE platform, provides a powerful and standardized framework for validating JavaBeans. In Spring Boot applications, Bean Validation is commonly used to check inputs for web requests, service methods, and more.

With Bean Validation, you can simply add annotations to your model properties, and the framework will automatically enforce those rules at runtime. It comes with many built-in validation constraints (like @NotNull, @Size, or @Email), and you can even create your own custom ones if needed.

### Getting Started

1. Use an existing Spring Boot 3 project or create a new one.
2. Add the validation starter dependency in your build configuration file:

// For Maven
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```

// For Gradle
```gradle
implementation 'org.springframework.boot:spring-boot-starter-validation'
```

3. Create a model class in your Spring Boot project and use annotations to define validation rules for your data.

```java
package com.sunil.jakartaExample.model;

import java.io.Serializable;
import jakarta.persistence.*;
import jakarta.validation.constraints.*;

import org.springframework.lang.Nullable;

import java.util.Date;
import java.sql.Timestamp;

@Entity
@Table(name = "patient_details")
@NamedQuery(name = "PatientDetail.findAll", query = "SELECT p FROM PatientDetail p")
public class PatientDetail implements Serializable {
    private static final long serialVersionUID = 1L;

    @Id
    @GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "patient_details_id_seq_generator")
    private Long id;

    @NotBlank(message = "Address is required")
    @Size(min = 5, max = 256, message = "Address must be between 5 and 256 characters")
    private String address;

    @NotNull(message = "Age is required")
    @Min(value = 0, message = "Age must be greater than or equal to 0")
    private Integer age;

    @Column(name = "created_date")
    private Timestamp createdDate;

    @Column(name = "email_id")
    @NotBlank(message = "Email is required")
    @Email(message = "Invalid email format")
    private String emailId;

    @Column(name = "gender_id")
    @NotNull(message = "Gender is required")
    private Integer genderId;

    @Column(name = "mobile_number")
    @NotBlank(message = "Mobile Number is required")
    @Pattern(regexp = "^[0-9]{10}$", message = "Invalid mobile number")
    private String mobileNumber;

  
    @NotBlank(message = "Patient Name is required")
    @Size(min = 3, max = 50, message = "Patient Name must be between 3 and 50 characters")
    private String name;

    public PatientDetail() {
    }

    // Getters and Setters
    public Long getId() {
        return this.id;
    }
    public void setId(Long id) {
        this.id = id;
    }

    public String getAddress() {
        return this.address;
    }
    public void setAddress(String address) {
        this.address = address;
    }

    public Integer getAge() {
        return this.age;
    }
    public void setAge(Integer age) {
        this.age = age;
    }

    public Timestamp getCreatedDate() {
        return this.createdDate;
    }
    public void setCreatedDate(Timestamp createdDate) {
        this.createdDate = createdDate;
    }

    public String getEmailId() {
        return this.emailId;
    }
    public void setEmailId(String emailId) {
        this.emailId = emailId;
    }

    public Integer getGenderId() {
        return this.genderId;
    }
    public void setGenderId(Integer genderId) {
        this.genderId = genderId;
    }

    public String getMobileNumber() {
        return this.mobileNumber;
    }
    public void setMobileNumber(String mobileNumber) {
        this.mobileNumber = mobileNumber;
    }

    public String getName() {
        return this.name;
    }
    public void setName(String name) {
        this.name = name;
    }

}
```

4. In your controller class, use the validator by creating an object of your model class and applying validation annotations.

```java
package com.sunil.jakartaExample.controller;

import java.sql.Date;
import java.time.LocalDate;
import java.time.format.DateTimeFormatter;
import java.util.HashMap;
import java.util.List;
import java.util.Map;
import java.util.Set;

import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import jakarta.validation.ConstraintViolation;
import jakarta.validation.Validation;
import jakarta.validation.Validator;
import jakarta.validation.ValidatorFactory;

import org.apache.logging.log4j.LogManager;
import org.apache.logging.log4j.Logger;
import org.json.JSONObject;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.web.bind.annotation.CrossOrigin;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

import com.google.gson.Gson;
import com.google.gson.GsonBuilder;
import com.sunil.jakartaExample.model.PatientDetail;
import com.sunil.jakartaExample.model.User;
import com.sunil.jakartaExample.repository.AuthorityRepository;
import com.sunil.jakartaExample.repository.UserRepository;
import com.sunil.jakartaExample.service.PatientService;
import com.sunil.jakartaExample.service.impl.UserServiceImpl;

/**
 * @author Sandeep
 */

@CrossOrigin(origins = {"http://localhost:4200"})
@RestController
@RequestMapping("/api/patient")
public class PatientController {

    private static final Logger log = LogManager.getLogger(PatientController.class);

    @Autowired
    JdbcTemplate jdbcTemplate;

    @Autowired
    UserRepository userRepo;

    @Autowired
    AuthorityRepository authRepo;

    @Autowired
    private UserServiceImpl userDetailServiceImpl;

    @Autowired
    PatientService patientService;

    @Value("${application.home}")
    private String applicationHome;

    Gson gson = new GsonBuilder()
            .enableComplexMapKeySerialization()
            .serializeNulls()
            .setDateFormat("yyyy-MM-dd HH:mm:ss")
            .setPrettyPrinting()
            .setVersion(1.0)
            .create();

    public <T> ResponseEntity<String> validateEntity(T entity) {
        ValidatorFactory factory = Validation.buildDefaultValidatorFactory();
        Validator validator = factory.getValidator();
        Set<ConstraintViolation<T>> violations = validator.validate(entity);
        if (!violations.isEmpty()) {
            // Construct error response
            StringBuilder errorMessage = new StringBuilder("Validation error(s): ");
            violations.forEach(violation -> {
                errorMessage.append(violation.getMessage()).append(";");
            });
            return ResponseEntity.badRequest().body(errorMessage.toString());
        }
        // No validation errors
        return null;
    }

    @PostMapping("/savePatientPersonalDetails")
    public ResponseEntity<String> savePatientPersonalDetails(
            @RequestParam(value = "personalObj", required = true) String personalObj) {
        try {
            // Convert JSON string to PatientDetail object
            PatientDetail patientDetail = gson.fromJson(personalObj, PatientDetail.class);

            // Validate PatientDetail object
            ResponseEntity<String> validationResponse = validateEntity(patientDetail);
            if (validationResponse != null) {
                return validationResponse;
            }

            Map<String, Object> map = new HashMap<>();
            map = patientService.savePatientPersonalDetail(personalObj, gson, jdbcTemplate);
            return ResponseEntity.ok().body(gson.toJson(map));
        } catch (Exception e) {
            log.error("Exception occurred:", e);
            return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR)
                    .body("An error occurred while processing your request: " + e.getMessage());
        }
    }
}
```

5. The PatientService interface defines a method to save patient personal details into the database.

```java
package com.sunil.jakartaExample.service;

import java.util.Map;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.stereotype.Component;
import org.springframework.web.multipart.MultipartFile;
import com.google.gson.Gson;
/**
 * @author Sandeep
 *
 */
@Component
public interface PatientService {
   Map<String, Object> savePatientPersonalDetail(String personalObj, Gson gson,JdbcTemplate jdbcTemplate) throws Exception; 
}
```

6. The PatientServiceImpl class provides the implementation of PatientService to save patient personal details with transaction management.

```java
package com.sunil.jakartaExample.service.impl;

import java.util.Date;
import java.util.HashMap;
import java.util.Map;
import java.util.Optional;
import java.sql.Timestamp;

import org.apache.logging.log4j.LogManager;
import org.apache.logging.log4j.Logger;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.http.HttpStatus;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.security.core.userdetails.UsernameNotFoundException;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import com.google.gson.Gson;
import com.sunil.jakartaExample.model.PatientDetail;
import com.sunil.jakartaExample.repository.PatientDetailRepository;
import com.sunil.jakartaExample.service.PatientService;

/**
 * @author Sandeep
 */
@Service
public class PatientServiceImpl implements PatientService {

    private static final Logger log = LogManager.getLogger(PatientServiceImpl.class);

    @Value("${application.home}")
    private String applicationHome;

    @Autowired
    PatientDetailRepository patientDetailRepo;

    @Override
    @Transactional(rollbackFor = { UsernameNotFoundException.class, Exception.class })
    public Map<String, Object> savePatientPersonalDetail(String personalObj, Gson gson, JdbcTemplate jdbcTemplate)
            throws Exception {
        long patientId = 0;
        Map<String, Object> map = new HashMap<>();
        String baseFolderPath = applicationHome + "/images/";
        try {
            // Convert JSON string to PatientDetail entity
            PatientDetail patientDetail = gson.fromJson(personalObj, PatientDetail.class);
            // Save entity
            patientDetail = this.patientDetailRepo.save(patientDetail);
            patientId = patientDetail.getId();
            if (patientId > 0) {
                PatientDetail detail = this.patientDetailRepo.findById(patientId)
                        .orElseThrow(() -> new UsernameNotFoundException("Patient not found exception"));

                Optional<PatientDetail> checkUserNull = Optional.ofNullable(detail);
                if (checkUserNull.isPresent()) {
                    try {
                        String folderName = "patient_" + patientId;
                        String folderPath = baseFolderPath + folderName;
                        Util.createFolder(folderPath);
                    } catch (Exception e) {
                        log.error("Error creating patient folder", e);
                    }
                }
                map.put("id", patientId);
                map.put("httpStatus", HttpStatus.CREATED);
                return map;
            } else {
                map.put("httpStatus", HttpStatus.INTERNAL_SERVER_ERROR);
                return map;
            }
        } catch (Exception e) {
            log.error("Error saving Patient Personal Detail", e);
            map.put("httpStatus", HttpStatus.INTERNAL_SERVER_ERROR);
            return map;
        }
    }
}
```

### Constraint Annotations in Spring Boot (Jakarta Validation)
Constraint annotations are used in Spring Boot model classes to apply validation rules on fields. They ensure that user input meets specific requirements before saving to the database or processing further.

**Boolean Constraints**
* **@AssertFalse:** The value must be false.
* **@AssertTrue:** The value must be true.



**Number Constraints**
* **@DecimalMax:** Number must be ≤ (less than or equal to) given maximum.
* **@DecimalMin:** Number must be ≥ (greater than or equal to) given minimum.
* **@Digits:** Number must have specified integer and fraction digits.
* **@Max:** Value must be ≤ (less than or equal to) given maximum.
* **@Min:** Value must be ≥ (greater than or equal to) given minimum.
* **@Negative:** Value must be negative (< 0).
* **@NegativeOrZero:** Value must be negative or zero (≤ 0).
* **@Positive:** Value must be positive (> 0).
* **@PositiveOrZero:** Value must be positive or zero (≥ 0).

**String Constraints**
* **@Email:** Must be a valid email format.
* **@NotBlank:** Value cannot be null, empty, or only spaces.
* **@NotEmpty:** Value cannot be null or empty.
* **@NotNull:** Value must not be null.
* **@Null:** Value must be null.
* **@Pattern:** Value must match a given regex pattern.
* **@Size:** String/Collection must be within a size range.

**Date & Time Constraints**
* **@Future:** Date must be in the future.
* **@FutureOrPresent:** Date must be in the future or today.
* **@Past:** Date must be in the past.
* **@PastOrPresent:** Date must be in the past or today.

### Custom Annotations in Java
Sometimes the built-in validation annotations are not enough. In such cases, you can create your own custom annotation and define the logic for it.



* **ElementType:** java.lang.annotation.ElementType defines where an annotation can be applied.
    * TYPE: class, interface, or enum declaration
    * FIELD: field (including enum constants)
    * METHOD: method declaration
    * PARAMETER: method parameter
    * CONSTRUCTOR: constructor declaration
    * LOCAL_VARIABLE: local variable
    * PACKAGE: package declaration
* **Retention:** java.lang.annotation.RetentionPolicy defines how long an annotation is retained in the program lifecycle.
    * SOURCE: available only in source code, discarded at compile-time
    * CLASS: stored in .class file but not available at runtime
    * RUNTIME: stored in .class file and available at runtime (via reflection)

**Example: Creating a Custom Annotation in Java**

Step 1: Create the Custom Annotation
```java
import jakarta.validation.Constraint;
import jakarta.validation.Payload;
import java.lang.annotation.*;

@Documented
@Constraint(validatedBy = AgeValidator.class)   // Links to validator logic
@Target({ ElementType.FIELD })                  // Can be applied on fields
@Retention(RetentionPolicy.RUNTIME)             // Available at runtime
public @interface ValidAge {

    String message() default "Age must be between 18 and 60";

    Class<?>[] groups() default {};

    Class<? extends Payload>[] payload() default {};
}
```

Step 2: Create the Validator Logic
```java
import jakarta.validation.ConstraintValidator;
import jakarta.validation.ConstraintValidatorContext;

public class AgeValidator implements ConstraintValidator<ValidAge, Integer> {

    @Override
    public boolean isValid(Integer age, ConstraintValidatorContext context) {
        return age != null && age >= 18 && age <= 60;
    }
}
```

Step 3: Use Custom Annotation in a Model Class
```java
public class Patient {
    
    @ValidAge
    private Integer age;

    // getters and setters
}
```

In this article, we explored how to use and integrate Jakarta Bean Validation with Spring Boot 3 applications. We learned how to handle validation errors, apply validation inside a controller, and enforce rules using annotations.
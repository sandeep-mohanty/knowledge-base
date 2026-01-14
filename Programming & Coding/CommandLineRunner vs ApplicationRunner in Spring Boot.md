# CommandLineRunner vs ApplicationRunner in Spring Boot (With Examples)

Spring Boot is a powerful framework that simplifies Java application development by providing many pre-configured features out of the box.

In Spring Boot, **CommandLineRunner** and **ApplicationRunner** are two important interfaces that allow you to execute code **after the application has started**. These interfaces are commonly used to perform tasks such as loading initial data, running startup logic, or executing background processes when the application launches.

In this article, we will learn what **CommandLineRunner** and **ApplicationRunner** are, how they work, and when to use each of them in a Spring Boot application.

---

## ApplicationRunner
**ApplicationRunner** is an interface in Spring Boot that is used to execute code **after the Spring Boot application has started**.



The example below shows how to implement the **ApplicationRunner** interface in the main class.

### Example: ApplicationRunner Implementation
**DemoApplication.java**

```java
package com.sunil.mishra.demo;

import org.springframework.boot.ApplicationArguments;
import org.springframework.boot.ApplicationRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;


@SpringBootApplication
public class DemoApplication implements ApplicationRunner {
    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }

 @Override
 public void run(ApplicationArguments args) throws Exception {
  // TODO Auto-generated method stub
  System.out.println("Hello World from ApplicationRunner");
 }

}
```

### Console Output
When you run the application, the message:

![ApplicationRunner Console Output](https://miro.medium.com/v2/resize:fit:720/format:webp/1*lCPG-KCZHf8Dw5YEhaT6Ng.png)

### Key Features of ApplicationRunner
1. **ApplicationArguments Object**
The `run()` method receives an `ApplicationArguments` object instead of a `String[]`. This object provides easy access to **command-line arguments** passed while starting the application.

2. **Utility Methods**
`ApplicationArguments` provides useful methods such as:
* `getOptionNames()`: returns all option names
* `getOptionValues(String name)`: returns values for a specific option
* `getNonOptionArgs()`: returns arguments without options

These methods make argument handling simple and structured.

### When to Use ApplicationRunner
You should use **ApplicationRunner** when:
* Your application needs **complex command-line argument parsing**
* You want to differentiate between **options and non-option arguments**
* You need a **structured and clean way** to access startup arguments
* You are using flags or named options (e.g., `--profile=dev`)

---

## CommandLineRunner
**CommandLineRunner** is a Spring Boot interface used to execute code **after the application has started**.

The example below demonstrates how to implement the **CommandLineRunner** interface in the main class.

### Example: CommandLineRunner Implementation
**DemoApplication.java**

```java
package com.sunil.mishra.demo;

import org.springframework.boot.ApplicationArguments;
import org.springframework.boot.ApplicationRunner;
import org.springframework.boot.CommandLineRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;


@SpringBootApplication
public class DemoApplication implements CommandLineRunner {
    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }

 @Override
 public void run(String... args) throws Exception {
  // TODO Auto-generated method stub
  System.out.println("Hello World from CommandLineRunner");
 }
    
    
    // implements ApplicationRunner 
 /*
  * @Override public void run(ApplicationArguments args) throws Exception { //
  * TODO Auto-generated method stub
  * System.out.println("Hello World from ApplicationRunner"); }
  */

}
```

### Console Output
When you run the application, the message:

![CommandLineRunner Console Output](https://miro.medium.com/v2/resize:fit:720/format:webp/1*XMZK6GIwZUWiDbZ2gdV2YQ.png)

### Key Features of CommandLineRunner
1. **Execution Timing**
The `run()` method is executed after all Spring beans are created and the application context is fully initialized. This makes it a good place to run **startup logic**.

2. **Command-Line Arguments**
The `run()` method receives a `String[]` array that contains the command-line arguments passed when starting the application.

### When to Use CommandLineRunner
You should use **CommandLineRunner** when:
* You need to **initialize data**, such as inserting default records into a database
* You want to perform **configuration or environment checks** at startup
* You need to **start background tasks** or scheduled jobs when the application launches
* Your argument handling is **simple** and does not require structured parsing

---
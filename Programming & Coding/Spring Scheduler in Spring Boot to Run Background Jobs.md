# How to Use Spring Scheduler in Spring Boot to Run Background Jobs

Spring Boot provides a scheduler feature that lets you run tasks automatically at specific times or intervals, so you don’t have to start them manually. This is helpful for automating repetitive jobs such as sending emails, cleaning up old data, or backing up databases regularly. Setting up a scheduler in Spring Boot is straightforward and efficient.

This article will explain how to implement schedulers in Spring Boot, explore different ways to schedule tasks, and show how to customize the timing of these tasks.

Spring Boot makes scheduling tasks easy with the `@Scheduled` annotation, which lets you run methods automatically at specific times or intervals.

To use scheduling features, you first need to enable scheduling support in your application. This is done by adding the `@EnableScheduling` annotation to your main Spring Boot application class or any configuration class. Once enabled, you can use `@Scheduled` to define when and how often tasks should run.

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.scheduling.annotation.EnableScheduling;

@SpringBootApplication
@EnableScheduling  // Enables scheduling support in Spring Boot
public class SchedulerApplication {
    public static void main(String[] args) {
        SpringApplication.run(SchedulerApplication.class, args);
    }
}
```

In this example, the `@EnableScheduling` annotation activates Spring’s task scheduler. After this, you can create scheduled tasks by annotating methods with `@Scheduled`.

### Scheduling Tasks Using a Cron Expression
Cron expressions allow you to define complex, precise schedules by specifying six time-related fields: seconds, minutes, hours, day of the month, month, and day of the week.

```java
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;

@Component
public class CronScheduler {
    @Scheduled(cron = "0 0/5 * * * ?")
    public void executeTask() {
        System.out.println("Task executed at: " + java.time.LocalTime.now());
    }
}
```

**Explanation:**
- The cron expression `0 0/5 * * * ?` means the task runs every 5 minutes.
- The `@Scheduled` annotation marks the method for scheduling.
- **Output example every 5 minutes:** `Task executed at: HH:MM:SS`

### Scheduling Tasks at a Fixed Rate
Fixed rate scheduling runs the task at a regular interval, regardless of how long the previous execution took.

```java
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;

@Component
public class FixedRateScheduler {
    @Scheduled(fixedRate = 5000)
    public void executeFixedRateTask() {
        System.out.println("Fixed rate task executed at: " + java.time.LocalTime.now());
    }
}
```

**Explanation:**
- `fixedRate = 5000` means the method runs every 5000 milliseconds (5 seconds), measured from the start time of each execution.
- Tasks will continue to run at consistent intervals even if an earlier execution is still running.
- **Output every 5 seconds:** `Fixed rate task executed at: HH:MM:SS`

### Scheduling Tasks with a Fixed Delay
Fixed delay scheduling runs the task a fixed time after the previous execution finishes.

```java
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;

@Component
public class FixedDelayScheduler {
    @Scheduled(fixedDelay = 3000)
    public void executeFixedDelayTask() {
        System.out.println("Fixed delay task executed at: " + java.time.LocalTime.now());
    }
}
```

**Explanation:**
- `fixedDelay = 3000` makes the next execution start 3 seconds after the last one finishes.
- Useful when task execution times vary and you want a fixed gap between executions.
- **Output example:**
  After a 3-second delay from the last task completion, print:
  `Fixed delay task executed at: HH:MM:SS`

[Image comparing Spring @Scheduled fixedRate vs fixedDelay execution timelines]

### Running Tasks in Parallel
By default, Spring Boot runs scheduled tasks sequentially in a single thread. This means a long-running task can delay others from executing on time. To prevent this, you can enable parallel execution by configuring a thread pool that allows multiple tasks to run asynchronously, improving performance and responsiveness.

#### Steps to Run Tasks in Parallel
1. **Enable Async Scheduling:** Use the `@EnableAsync` annotation on a configuration class to enable asynchronous task execution.
2. **Configure a Thread Pool:** Define a thread pool by creating a `TaskExecutor` bean, such as `ThreadPoolTaskExecutor`, to manage concurrent threads.
3. **Annotate Scheduled Methods with @Async:** Add the `@Async` annotation on scheduled methods to run them asynchronously using the configured thread pool.

```java
// ParallelSchedulerConfig.java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.scheduling.annotation.EnableAsync;
import org.springframework.scheduling.concurrent.ThreadPoolTaskExecutor;
import java.util.concurrent.Executor;

@Configuration
@EnableAsync // Enable asynchronous execution
public class ParallelSchedulerConfig {

    @Bean(name = "taskExecutor")
    public Executor taskExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(5);  // Core threads available
        executor.setMaxPoolSize(10);  // Maximum threads allowed
        executor.setQueueCapacity(25); // Queue capacity for tasks
        executor.setThreadNamePrefix("SchedulerThread-"); // Thread name prefix
        executor.initialize();
        return executor;
    }
}
```

```java
// ParallelScheduledTasks.java
import org.springframework.scheduling.annotation.Async;
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;

@Component
public class ParallelScheduledTasks {

    @Async("taskExecutor") // Use configured thread pool
    @Scheduled(fixedRate = 2000) // Run every 2 seconds
    public void taskOne() throws InterruptedException {
        System.out.println("Task One :: Execution Time - " + System.currentTimeMillis()
            + " :: Thread - " + Thread.currentThread().getName());
        Thread.sleep(3000); // Simulate long-running task
    }

    @Async("taskExecutor") // Use configured thread pool
    @Scheduled(fixedRate = 2000) // Run every 2 seconds
    public void taskTwo() throws InterruptedException {
        System.out.println("Task Two :: Execution Time - " + System.currentTimeMillis()
            + " :: Thread - " + Thread.currentThread().getName());
        Thread.sleep(3000); // Simulate long-running task
    }
}
```

**How It Works**
- `ParallelSchedulerConfig` defines a thread pool with core size 5, max size 10, and a queue capacity of 25, enabling async execution with `@EnableAsync`.
- `ParallelScheduledTasks` has two scheduled methods annotated with both `@Scheduled` and `@Async(“taskExecutor”)`, so they’re executed asynchronously on different threads.
- Both tasks run every 2 seconds, but the 3-second sleep simulates long-running tasks. Thanks to the thread pool and async execution, they don’t block each other.

**Key Points to Remember**
- Parallel execution improves application performance, especially when tasks take variable or long time.
- `@Async` annotation combined with a configured thread pool is essential for parallel task execution.
- Proper thread pool configuration (core size, max size, queue capacity) is crucial to prevent resource exhaustion or performance issues.

### Setting Delay or Rate Dynamically at Runtime
Sometimes, you need to adjust the delay or interval of a scheduled task while your Spring Boot application is running — for example, changing an API polling interval based on external conditions. Spring Boot supports this by letting you programmatically register scheduled tasks with dynamic triggers.

#### Steps to Set Delay or Rate Dynamically
1. **Create a Custom Scheduler Configuration:** Implement the `SchedulingConfigurer` interface to define custom scheduling logic.
2. **Use ScheduledTaskRegistrar to Register Tasks:** This registrar class allows tasks to be registered programmatically with custom triggers.
3. **Update Delay or Rate at Runtime:** Provide methods to update scheduling parameters dynamically and apply them in the trigger logic.

```java
import org.springframework.context.annotation.Configuration;
import org.springframework.scheduling.annotation.SchedulingConfigurer;
import org.springframework.scheduling.config.ScheduledTaskRegistrar;
import org.springframework.scheduling.support.PeriodicTrigger;

import java.util.concurrent.TimeUnit;

@Configuration
public class DynamicSchedulerConfig implements SchedulingConfigurer {

    private int initialDelay = 5000; // Initial delay: 5 seconds

    @Override
    public void configureTasks(ScheduledTaskRegistrar taskRegistrar) {
        taskRegistrar.addTriggerTask(
            () -> System.out.println("Dynamic Task :: Execution Time - " + System.currentTimeMillis()),
            triggerContext -> {
                PeriodicTrigger trigger = new PeriodicTrigger(initialDelay, TimeUnit.MILLISECONDS);
                return trigger.nextExecutionTime(triggerContext);
            }
        );
    }

    // Method to update delay dynamically
    public void updateDelay(int newDelay) {
        this.initialDelay = newDelay;
        System.out.println("Delay updated to: " + newDelay + " milliseconds");
    }
}
```

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class DynamicSchedulerController {

    @Autowired
    private DynamicSchedulerConfig dynamicSchedulerConfig;

    @GetMapping("/updateDelay")
    public String updateDelay(@RequestParam int delay) {
        dynamicSchedulerConfig.updateDelay(delay);
        return "Delay updated to: " + delay + " milliseconds";
    }
}
```

**How It Works**
- The task initially runs every 5 seconds.
- Accessing the `/updateDelay?delay=15000` endpoint changes the delay to 15 seconds dynamically.
- The `PeriodicTrigger` fetches the current delay value each time it schedules the next execution, ensuring changes apply immediately.

**Key Points**
- Dynamic scheduling is useful for adapting task intervals during runtime based on changing conditions.
- `ScheduledTaskRegistrar` and `PeriodicTrigger` are essential for flexible, programmatic scheduling.
- This pattern can be extended to update not only delays but also rates and cron expressions dynamically if needed.

### Configuring Scheduled Tasks Using XML
Although Spring Boot commonly uses annotations for scheduling tasks, XML configuration is still supported and useful, especially for legacy projects or those preferring external configuration.

#### Steps to Configure Scheduling with XML
1. **Enable Scheduling in XML:** Use the `<task:annotation-driven />` tag in your Spring configuration XML (like `applicationContext.xml`) to enable scheduling support.
2. **Define the Bean:** Declare your task class as a bean in the XML.
3. **Configure Scheduled Tasks:** Use the `<task:scheduled-tasks>` tag to define scheduled tasks and their timing, specifying methods and intervals like fixed-rate or fixed-delay.

```xml
// XML Configuration (applicationContext.xml)
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:task="http://www.springframework.org/schema/task"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
           http://www.springframework.org/schema/beans/spring-beans.xsd
           http://www.springframework.org/schema/task
           http://www.springframework.org/schema/task/spring-task.xsd">

    <task:annotation-driven/>

    <bean id="scheduledTasks" class="com.example.ScheduledTasks"/>

    <task:scheduled-tasks>
        <task:scheduled ref="scheduledTasks" method="taskWithFixedRate" fixed-rate="5000"/>
        <task:scheduled ref="scheduledTasks" method="taskWithFixedDelay" fixed-delay="5000"/>
    </task:scheduled-tasks>
</beans>
```

```java
// ScheduledTasks.java
package com.example;

public class ScheduledTasks {

    public void taskWithFixedRate() {
        System.out.println("Fixed Rate Task (XML) :: Execution Time - " + System.currentTimeMillis());
    }

    public void taskWithFixedDelay() {
        System.out.println("Fixed Delay Task (XML) :: Execution Time - " + System.currentTimeMillis());
    }
}
```

**Key Points to Remember**
- XML-based scheduling configuration is useful when you prefer to avoid annotations or when working with legacy code.
- It’s a practical option if you want to externalize scheduling configuration outside the application code.
- Spring supports mixing XML-based and annotation-based scheduling configuration within the same application.

### Fixed Rate vs Fixed Delay
- **Task Execution:** **Fixed Rate** runs tasks at a fixed interval measured from the start time of each execution, no matter how long the previous task took. **Fixed Delay** runs tasks with a fixed pause after each task completes, so the next task starts after the delay from the last task’s finish time.
- **Task Overlap:** With **Fixed Rate**, if a task takes longer than the interval, the next execution will start immediately after the current one finishes, which can cause tasks to run back-to-back. With **Fixed Delay**, the next task always starts after the specified delay, preventing overlap and ensuring a cooldown period between executions.
- **Use Cases:** **Fixed Rate** is best when you need strict, consistent intervals, such as in polling or monitoring. **Fixed Delay** is ideal when tasks require a cooldown period or are resource-intensive, like rate-limited API calls or operations where you want

***
Schedulers in Spring Boot provide an efficient way to automate tasks such as sending notifications, generating reports, or running periodic jobs. By learning how to use cron expressions, fixed-rate, and fixed-delay scheduling, you can easily build flexible and reliable background tasks in your applications.
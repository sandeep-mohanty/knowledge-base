# Java Spring Boot Dynamic Task Scheduling

## Table of Contents
- Why Dynamic Scheduling?
- Understanding TaskScheduler Interface
- Implementing Dynamic Scheduling
- Managing Scheduled Tasks
- Best Practices and Common Pitfalls

---

## 1. Why Dynamic Scheduling?

Static scheduling with `@Scheduled` works perfectly for tasks with fixed schedules that never change. However, many real-world applications require flexibility:

**Common Use Cases:**
- User-defined notifications: Users set their own reminder times
- Configurable backups: Administrators adjust backup schedules without deployment
- Business rule changes: Marketing campaigns with varying schedules
- Multi-tenant applications: Different tenants need different schedules
- Dynamic rate limiting: Adjusting task frequency based on system load

Static annotations can’t handle these scenarios because they’re compiled into your code. Dynamic scheduling solves this by managing tasks programmatically at runtime.

---

## 2. Understanding TaskScheduler Interface

Spring provides the `TaskScheduler` interface for programmatic scheduling.

```java
public interface TaskScheduler {
    ScheduledFuture<?> schedule(Runnable task, Trigger trigger);
    ScheduledFuture<?> schedule(Runnable task, Instant startTime);
    ScheduledFuture<?> scheduleAtFixedRate(Runnable task, Duration period);
    ScheduledFuture<?> scheduleAtFixedRate(Runnable task, Instant startTime, Duration period);
    ScheduledFuture<?> scheduleWithFixedDelay(Runnable task, Duration delay);
}
```

The `ScheduledFuture` returned by these methods is crucial—it represents the scheduled task and allows you to cancel it later.

---

## 3. Implementing Dynamic Scheduling

### Step 1: Configuration

```java
@Configuration
@EnableScheduling
public class SchedulerConfig {

    @Bean
    public TaskScheduler taskScheduler() {
        ThreadPoolTaskScheduler scheduler = new ThreadPoolTaskScheduler();
        scheduler.setPoolSize(10);
        scheduler.setThreadNamePrefix("scheduled-task-");
        scheduler.setWaitForTasksToCompleteOnShutdown(true);
        scheduler.setAwaitTerminationSeconds(20);
        scheduler.initialize();
        return scheduler;
    }
}
```

**Key Settings:**
- `setPoolSize(10)`: Allows up to 10 concurrent scheduled tasks
- `setWaitForTasksToCompleteOnShutdown(true)`: Ensures tasks finish gracefully on shutdown
- `setAwaitTerminationSeconds(20)`: Waits up to 20 seconds for tasks to complete

---

### Step 2: Dynamic Scheduler Service

```java
@Service
@RequiredArgsConstructor
public class DynamicSchedulerService {

    private final TaskScheduler taskScheduler;
    private final Map<String, ScheduledFuture<?>> scheduledTasks = new ConcurrentHashMap<>();
   
    public void scheduleTask(String taskId, Runnable task, String cronExpression) {
        cancelTask(taskId);
        CronTrigger trigger = new CronTrigger(cronExpression);
        ScheduledFuture<?> scheduledTask = taskScheduler.schedule(task, trigger);
        scheduledTasks.put(taskId, scheduledTask);
    }

    public void scheduleTaskAtFixedRate(String taskId, Runnable task, long periodInMillis) {
        cancelTask(taskId);
        Duration period = Duration.ofMillis(periodInMillis);
        ScheduledFuture<?> scheduledTask = taskScheduler.scheduleAtFixedRate(task, period);
        scheduledTasks.put(taskId, scheduledTask);
    }

    public void scheduleOneTimeTask(String taskId, Runnable task, Instant startTime) {
        cancelTask(taskId);
        ScheduledFuture<?> scheduledTask = taskScheduler.schedule(task, startTime);
        scheduledTasks.put(taskId, scheduledTask);
    }

    public void cancelTask(String taskId) {
        ScheduledFuture<?> scheduledTask = scheduledTasks.get(taskId);
        if (scheduledTask != null) {
            scheduledTask.cancel(false);
            scheduledTasks.remove(taskId);
        }
    }

    public boolean isTaskScheduled(String taskId) {
        ScheduledFuture<?> task = scheduledTasks.get(taskId);
        return task != null && !task.isCancelled() && !task.isDone();
    }
}
```

---

### Step 3: Practical Usage Example

```java
@Service
@RequiredArgsConstructor
public class UserNotificationService {

    private final DynamicSchedulerService dynamicScheduler;
    private final NotificationRepository notificationRepository;

    public void scheduleUserReminder(Long userId, String reminderMessage, String cronExpression) {
        String taskId = "user-reminder-" + userId;
        Runnable reminderTask = () -> {
            sendNotification(userId, reminderMessage);
            log.info("Reminder sent to user {}: {}", userId, reminderMessage);
        };
        dynamicScheduler.scheduleTask(taskId, reminderTask, cronExpression);
        log.info("Scheduled reminder for user {} with cron: {}", userId, cronExpression);
    }

    public void updateReminderSchedule(Long userId, String newCronExpression) {
        String taskId = "user-reminder-" + userId;
        String reminderMessage = getReminderMessage(userId);
        scheduleUserReminder(userId, reminderMessage, newCronExpression);
        log.info("Updated reminder schedule for user {} to: {}", userId, newCronExpression);
    }

    public void cancelUserReminder(Long userId) {
        String taskId = "user-reminder-" + userId;
        dynamicScheduler.cancelTask(taskId);
        log.info("Cancelled reminder for user {}", userId);
    }

    private void sendNotification(Long userId, String message) {
        // Implementation for sending notification
    }

    private String getReminderMessage(Long userId) {
        return "Your scheduled reminder!";
    }
}
```

---

### Step 4: REST API Controller

```java
@RestController
@RequestMapping("/api/reminders")
@RequiredArgsConstructor
public class ReminderController {

    private final UserNotificationService notificationService;

    @PostMapping
    public ResponseEntity<String> createReminder(
            @RequestParam Long userId,
            @RequestParam String message,
            @RequestParam String cronExpression) {
        try {
            notificationService.scheduleUserReminder(userId, message, cronExpression);
            return ResponseEntity.ok("Reminder scheduled successfully");
        } catch (IllegalArgumentException e) {
            return ResponseEntity.badRequest().body("Invalid cron expression: " + e.getMessage());
        }
    }

    @PutMapping("/{userId}")
    public ResponseEntity<String> updateReminder(
            @PathVariable Long userId,
            @RequestParam String cronExpression) {
        notificationService.updateReminderSchedule(userId, cronExpression);
        return ResponseEntity.ok("Reminder schedule updated");
    }

    @DeleteMapping("/{userId}")
    public ResponseEntity<String> cancelReminder(@PathVariable Long userId) {
        notificationService.cancelUserReminder(userId);
        return ResponseEntity.ok("Reminder cancelled");
    }
}
```

---

## 4. Managing Scheduled Tasks

### Persisting Schedule Information

```java
@Entity
@Table(name = "scheduled_tasks")
public class ScheduledTaskEntity {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String taskId;
    private Long userId;
    private String cronExpression;
    private String taskType;
    private boolean active;

    @Column(columnDefinition = "TEXT")
    private String taskData; // JSON data for task parameters
}
```

### Restoring Tasks on Startup

```java
@Service
@RequiredArgsConstructor
public class TaskRestoreService {

    private final ScheduledTaskRepository taskRepository;
    private final DynamicSchedulerService dynamicScheduler;
    private final UserNotificationService notificationService;

    @EventListener(ApplicationReadyEvent.class)
    public void restoreScheduledTasks() {
        List<ScheduledTaskEntity> activeTasks = taskRepository.findByActiveTrue();
        for (ScheduledTaskEntity taskEntity : activeTasks) {
            try {
                if ("USER_REMINDER".equals(taskEntity.getTaskType())) {
                    notificationService.scheduleUserReminder(
                        taskEntity.getUserId(),
                        extractMessage(taskEntity.getTaskData()),
                        taskEntity.getCronExpression()
                    );
                }
                log.info("Restored task: {}", taskEntity.getTaskId());
            } catch (Exception e) {
                log.error("Failed to restore task: {}", taskEntity.getTaskId(), e);
            }
        }
    }

    private String extractMessage(String taskData) {
        return taskData; // Simplified
    }
}
```

---

## 5. Best Practices and Common Pitfalls

### Best Practices
1. Use Unique Task IDs  
2. Validate Cron Expressions  
3. Proper Resource Management  
4. Error Handling  
5. Monitor Task Execution  

### Common Pitfalls
1. Memory Leaks  
2. Thread Pool Exhaustion  
3. Not Handling Task Failures  
4. Ignoring Time Zones  
5. Concurrent Modification  

---

## Conclusion

Dynamic task scheduling in Spring Boot provides flexibility and runtime control. By using `TaskScheduler` directly instead of `@Scheduled`, you gain:

- Runtime control over task schedules  
- User-customizable timing and frequencies  
- Programmatic management of task lifecycle  
- Scalability for multi-tenant applications  

The key is proper management: unique task IDs, graceful cancellation, persistent storage, and robust error handling.

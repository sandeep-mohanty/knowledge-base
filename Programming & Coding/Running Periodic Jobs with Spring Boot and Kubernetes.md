# Running Periodic Jobs with Spring Boot and Kubernetes: From Traditional Cron to K8s Jobs

### Introduction
If you’ve ever needed to run periodic tasks in production, you know it’s more complicated than it seems. That job that should run every day at 3 AM to clean old data, generate reports, or batch process information can become a nightmare if not properly architected.

I remember when I managed an e-commerce system that processed canceled orders every hour. We used good old Linux cron running on a single machine. It worked… until the day that machine went down. We lost hours of processing, customers didn’t receive automatic refunds, and worse: we only discovered the problem the next day when the phone started ringing.

In this article, I’ll show you how I evolved from that fragile approach to a robust and scalable solution using Spring Boot with Kubernetes Jobs. If you’re struggling with jobs that sometimes run, sometimes don’t, or that duplicate executions when you scale your application, this article is for you.

### The Real Problem: When Periodic Jobs Become Critical
Before diving into the technical solution, let’s understand why periodic jobs are so important and what problems they solve in the real world.

#### Common Use Cases
Throughout my career, I’ve encountered several scenarios where periodic jobs were essential:

* **Temporary Data Cleanup** Systems that maintain authentication tokens, expired sessions, or temporary files need cleanup routines. Without this, the database grows indefinitely, degrading performance and increasing costs.
* **Consolidated Report Generation** Daily financial reports, weekly executive dashboards, or monthly metrics analysis. These processes usually involve heavy aggregations that can’t run in real-time.
* **Batch Processing** Synchronization with legacy systems, importing data from partners, processing recurring payments. Operations that need to happen at specific times, outside peak hours.
* **Cache Maintenance** Cache warm-up, updating frequently accessed data, or scheduled invalidation of outdated information.
* **Compliance and Audit** Regulated companies need to generate audit logs, backup critical data, or send regulatory reports on defined schedules.

#### Traditional Challenges
Here’s what I learned the hard way:

* **Problem 1: Single Point of Failure** If you run your job on a single machine (whether with cron or Spring @Scheduled), that machine becomes a critical point. When it goes down, your jobs stop. Simple as that.
* **Problem 2: Horizontal Scalability** When you scale your Spring Boot application horizontally (say, 3 replicas in Kubernetes), @Scheduled runs on ALL instances. Result? Your email sending job fires 3 times. Your payment processing charges the customer 3 times. Disaster.
* **Problem 3: Configuration Management** Changing a job’s execution time means modifying code, committing, building, deploying. For a simple configuration change, that’s too much bureaucracy.
* **Problem 4: Monitoring and Observability** How do you know if your job ran? If it failed? How long it took? With traditional cron, you depend on scattered logs and hope.
* **Problem 5: Execution History** When was the last successful execution? How many times did it fail this week? This information is critical for troubleshooting but difficult to obtain with traditional approaches.

### The Evolution: Spring Boot with Kubernetes Jobs
Kubernetes offers two powerful resources for handling periodic jobs:

* **Jobs:** One-time executions that ensure a Pod runs until it completes successfully
* **CronJobs:** Scheduled jobs that run at specific times

The key insight? Separate the scheduling responsibility (Kubernetes CronJob) from the business logic (Spring Boot Application).

#### Why This Approach is Superior
* **Isolation:** Each job execution runs in an isolated Pod. If it fails, it doesn’t affect your main application.
* **Automatic Retry:** Kubernetes can automatically retry failed jobs.
* **History:** Kubernetes maintains execution history (successes and failures).
* **Declarative Configuration:** Schedule changes are made via YAML, without modifying code.
* **Scalability:** Your main application can scale freely. Jobs run independently.
* **Observability:** Native Kubernetes metrics, centralized logs, and integration with monitoring tools.

### Real Case: Banking Data Cleanup System
Let me share a real case I recently implemented. We worked on a financial system that processes daily transactions. For compliance and performance reasons, we needed to:
* Archive old transactions (over 90 days) to long-term storage
* Clean temporary audit logs after regulatory retention
* Consolidate daily metrics for executive dashboards
* Send regulatory reports to the Central Bank

Initially, we used Spring’s @Scheduled. It worked… until we scaled to 5 replicas to handle traffic. Suddenly, our jobs ran 5 times. Customers received 5 report emails. The Central Bank received 5 duplicate files.

We implemented the solution I’ll show you now, and the results were:
* Zero duplicate executions
* Complete history of all executions
* Automatic retry when any job failed
* Average execution time reduced by 40% (isolated jobs with dedicated resources)

### Hands-On: Implementing the Solution
Let’s build a complete system, step by step. We’ll use as an example a job that cleans expired records from a database.

#### Part 1: Spring Boot Project Structure
First, create a standard Spring Boot project. Here’s the pom.xml:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
         https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.2.0</version>
        <relativePath/>
    </parent>

    <groupId>com.example</groupId>
    <artifactId>k8s-job-demo</artifactId>
    <version>1.0.0</version>
    <name>Kubernetes Job Demo</name>
    <description>Spring Boot application for Kubernetes Jobs</description>

    <properties>
        <java.version>17</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>

        <dependency>
            <groupId>org.postgresql</groupId>
            <artifactId>postgresql</artifactId>
            <scope>runtime</scope>
        </dependency>

        <dependency>
            <groupId>io.micrometer</groupId>
            <artifactId>micrometer-registry-prometheus</artifactId>
        </dependency>

        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
        <finalName>k8s-job-demo</finalName>
    </build>
</project>
```

#### Part 2: Domain Model
Let’s create a simple entity that represents data that expires:

```java
@Entity
@Table(name = "user_sessions")
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class UserSession {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false, unique = true)
    private String sessionToken;

    @Column(nullable = false)
    private String userId;

    @Column(nullable = false)
    private LocalDateTime createdAt;

    @Column(nullable = false)
    private LocalDateTime expiresAt;

    @Column(nullable = false)
    private boolean active;

    @PrePersist
    protected void onCreate() {
        createdAt = LocalDateTime.now();
        if (expiresAt == null) {
            expiresAt = createdAt.plusHours(24);
        }
    }
}
```

#### Part 3: Repository
```java
@Repository
public interface UserSessionRepository extends JpaRepository<UserSession, Long> {

    @Query("SELECT s FROM UserSession s WHERE s.expiresAt < :expiryTime AND s.active = true")
    List<UserSession> findExpiredSessions(@Param("expiryTime") LocalDateTime expiryTime);

    @Modifying
    @Query("UPDATE UserSession s SET s.active = false WHERE s.expiresAt < :expiryTime")
    int deactivateExpiredSessions(@Param("expiryTime") LocalDateTime expiryTime);

    @Modifying
    @Query("DELETE FROM UserSession s WHERE s.expiresAt < :expiryTime AND s.active = false")
    int deleteInactiveSessions(@Param("expiryTime") LocalDateTime expiryTime);
}
```

#### Part 4: Cleanup Service
Here’s the heart of our solution. This service will be executed by the Kubernetes Job:

```java
@Service
@Slf4j
@RequiredArgsConstructor
public class SessionCleanupService {

    private final UserSessionRepository sessionRepository;

    @Transactional
    public CleanupResult cleanupExpiredSessions() {
        log.info("Starting session cleanup job at {}", LocalDateTime.now());

        LocalDateTime now = LocalDateTime.now();
        CleanupResult result = new CleanupResult();

        try {
            // Step 1: Deactivate expired sessions
            int deactivated = sessionRepository.deactivateExpiredSessions(now);
            result.setDeactivatedCount(deactivated);
            log.info("Deactivated {} expired sessions", deactivated);

            // Step 2: Delete inactive sessions older than 7 days
            LocalDateTime deletionThreshold = now.minusDays(7);
            int deleted = sessionRepository.deleteInactiveSessions(deletionThreshold);
            result.setDeletedCount(deleted);
            log.info("Deleted {} inactive sessions", deleted);

            result.setSuccess(true);
            result.setMessage("Cleanup completed successfully");

        } catch (Exception e) {
            log.error("Error during session cleanup", e);
            result.setSuccess(false);
            result.setMessage("Cleanup failed: " + e.getMessage());
            throw e; // Important: re-throw so Kubernetes knows it failed
        }

        log.info("Session cleanup job completed. Deactivated: {}, Deleted: {}",
                result.getDeactivatedCount(), result.getDeletedCount());

        return result;
    }

    @lombok.Data
    @lombok.Builder
    @lombok.NoArgsConstructor
    @lombok.AllArgsConstructor
    public static class CleanupResult {
        private boolean success;
        private String message;
        private int deactivatedCount;
        private int deletedCount;
    }
}
```

#### Part 5: Job Runner
This is the component that Kubernetes will execute. Important: it runs once and terminates:

```java
@Component
@Slf4j
@RequiredArgsConstructor
public class CleanupJobRunner implements CommandLineRunner {

    private final SessionCleanupService cleanupService;
    private final ApplicationContext context;

    @Override
    public void run(String... args) throws Exception {
        log.info("=================================================");
        log.info("Starting Kubernetes Job: Session Cleanup");
        log.info("=================================================");

        try {
            SessionCleanupService.CleanupResult result = cleanupService.cleanupExpiredSessions();

            if (result.isSuccess()) {
                log.info("Job completed successfully!");
                log.info("Summary: Deactivated={}, Deleted={}",
                        result.getDeactivatedCount(),
                        result.getDeletedCount());
                System.exit(0); // Exit code 0 = success
            } else {
                log.error("Job completed with errors: {}", result.getMessage());
                System.exit(1); // Exit code 1 = failure
            }

        } catch (Exception e) {
            log.error("Job failed with exception", e);
            System.exit(1);
        }
    }
}
```

#### Part 6: Application Configuration
application.yml:

```yaml
spring:
  application:
    name: session-cleanup-job

  datasource:
    url: ${DATABASE_URL:jdbc:postgresql://localhost:5432/sessions_db}
    username: ${DATABASE_USERNAME:postgres}
    password: ${DATABASE_PASSWORD:postgres}
    driver-class-name: org.postgresql.Driver

  jpa:
    hibernate:
      ddl-auto: validate
    show-sql: false
    properties:
      hibernate:
        format_sql: true
        dialect: org.hibernate.dialect.PostgreSQLDialect

logging:
  level:
    com.example.k8sjob: INFO
    org.springframework: WARN
    org.hibernate: WARN
  pattern:
    console: "%d{yyyy-MM-dd HH:mm:ss} - %msg%n"

management:
  endpoints:
    web:
      exposure:
        include: health,metrics,prometheus
  metrics:
    export:
      prometheus:
        enabled: true
```

#### Part 7: Dockerfile
```dockerfile
FROM eclipse-temurin:17-jre-alpine

WORKDIR /app

# Add non-root user for security
RUN addgroup -g 1001 appgroup && \
    adduser -D -u 1001 -G appgroup appuser

# Copy JAR
COPY target/k8s-job-demo.jar app.jar

# Change ownership
RUN chown -R appuser:appgroup /app

USER appuser

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=40s --retries=3 \
  CMD java -cp app.jar org.springframework.boot.loader.JarLauncher --help || exit 1

# Run application
ENTRYPOINT ["java", "-XX:+UseContainerSupport", "-XX:MaxRAMPercentage=75.0", "-jar", "app.jar"]
```

#### Part 8: Kubernetes Job Configuration
Now comes the magic. First, let’s create a simple Job (one-time execution):

k8s/job.yaml
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: session-cleanup-job
  namespace: default
  labels:
    app: session-cleanup
    job-type: maintenance
spec:
  # Number of times to retry on failure
  backoffLimit: 3

  # Keep history of completed jobs
  completions: 1

  # Successful jobs kept in history
  successfulJobsHistoryLimit: 5

  # Failed jobs kept in history
  failedJobsHistoryLimit: 3

  # Maximum execution time (2 hours)
  activeDeadlineSeconds: 7200

  template:
    metadata:
      labels:
        app: session-cleanup
        job-type: maintenance
    spec:
      restartPolicy: OnFailure

      # Service Account with appropriate permissions
      serviceAccountName: job-runner

      containers:
      - name: cleanup-job
        image: your-registry.io/session-cleanup-job:1.0.0
        imagePullPolicy: Always

        env:
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: database-credentials
              key: url

        - name: DATABASE_USERNAME
          valueFrom:
            secretKeyRef:
              name: database-credentials
              key: username

        - name: DATABASE_PASSWORD
          valueFrom:
            secretKeyRef:
              name: database-credentials
              key: password

        - name: SPRING_PROFILES_ACTIVE
          value: "production"

        resources:
          requests:
            memory: "512Mi"
            cpu: "250m"
          limits:
            memory: "1Gi"
            cpu: "500m"

        volumeMounts:
        - name: config
          mountPath: /app/config
          readOnly: true

      volumes:
      - name: config
        configMap:
          name: job-config
```

#### Part 9: Kubernetes CronJob Configuration
Now, the CronJob that will schedule periodic executions:

k8s/cronjob.yaml:
```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: session-cleanup-cronjob
  namespace: default
  labels:
    app: session-cleanup
    job-type: scheduled-maintenance
spec:
  # Run daily at 3 AM (UTC)
  schedule: "0 3 * * *"

  # Timezone (requires Kubernetes 1.25+)
  timeZone: "America/Sao_Paulo"

  # Concurrency policy: don't allow concurrent executions
  concurrencyPolicy: Forbid

  # Deadline to start the job (10 minutes)
  # If it misses the window, it won't execute
  startingDeadlineSeconds: 600

  # Keep history
  successfulJobsHistoryLimit: 5
  failedJobsHistoryLimit: 3

  # Suspend executions (useful for maintenance)
  suspend: false

  jobTemplate:
    metadata:
      labels:
        app: session-cleanup
        job-type: scheduled-maintenance
    spec:
      backoffLimit: 2
      activeDeadlineSeconds: 3600  # 1 hour max

      template:
        metadata:
          labels:
            app: session-cleanup
            job-type: scheduled-maintenance
          annotations:
            prometheus.io/scrape: "true"
            prometheus.io/port: "8080"
            prometheus.io/path: "/actuator/prometheus"

        spec:
          restartPolicy: OnFailure
          serviceAccountName: job-runner

          # Init container to verify dependencies
          initContainers:
          - name: wait-for-db
            image: busybox:1.35
            command: ['sh', '-c', 'until nc -z postgres-service 5432; do echo waiting for db; sleep 2; done']

          containers:
          - name: cleanup-job
            image: your-registry.io/session-cleanup-job:1.0.0
            imagePullPolicy: Always

            env:
            - name: DATABASE_URL
              valueFrom:
                secretKeyRef:
                  name: database-credentials
                  key: url

            - name: DATABASE_USERNAME
              valueFrom:
                secretKeyRef:
                  name: database-credentials
                  key: username

            - name: DATABASE_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: database-credentials
                  key: password

            - name: SPRING_PROFILES_ACTIVE
              value: "production"

            resources:
              requests:
                memory: "512Mi"
                cpu: "250m"
              limits:
                memory: "1Gi"
                cpu: "500m"

            volumeMounts:
            - name: config
              mountPath: /app/config
              readOnly: true

          volumes:
          - name: config
            configMap:
              name: job-config
```

#### Part 10: ConfigMap and Secrets
k8s/configmap.yaml:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: job-config
  namespace: default
data:
  application.yml: |
    spring:
      jpa:
        show-sql: false
    logging:
      level:
        com.example.k8sjob: INFO
```

k8s/secret.yaml:
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: database-credentials
  namespace: default
type: Opaque
stringData:
  url: jdbc:postgresql://postgres-service:5432/sessions_db
  username: app_user
  password: change-me-in-production
```

#### Part 11: Service Account and RBAC
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: job-runner
  namespace: default
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: job-runner-role
  namespace: default
rules:
- apiGroups: [""]
  resources: ["pods", "pods/log"]
  verbs: ["get", "list"]
- apiGroups: ["batch"]
  resources: ["jobs"]
  verbs: ["get", "list", "watch"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: job-runner-binding
  namespace: default
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: job-runner-role
subjects:
- kind: ServiceAccount
  name: job-runner
  namespace: default
```

### Deploy and Execution

#### Step 1: Build and Push Image
```bash
# Build application
mvn clean package -DskipTests

# Build Docker image
docker build -t your-registry.io/session-cleanup-job:1.0.0 .

# Push to registry
docker push your-registry.io/session-cleanup-job:1.0.0
```

#### Step 2: Deploy to Kubernetes
```bash
# Create namespace (optional)
kubectl create namespace jobs

# Apply configurations
kubectl apply -f k8s/secret.yaml
kubectl apply -f k8s/configmap.yaml
kubectl apply -f k8s/service-account.yaml

# Deploy CronJob
kubectl apply -f k8s/cronjob.yaml
```

#### Step 3: Verify Status
```bash
# List CronJobs
kubectl get cronjobs

# Check recent executions
kubectl get jobs

# View logs of most recent job
kubectl logs -l app=session-cleanup --tail=100

# Details of a specific job
kubectl describe job session-cleanup-cronjob-28374650
```

#### Step 4: Run Manually (Test)
```bash
# Create a Job from the CronJob
kubectl create job --from=cronjob/session-cleanup-cronjob manual-cleanup-test

# Follow execution
kubectl logs -f job/manual-cleanup-test
```

### Monitoring and Observability
One of the great advantages of this approach is native observability. Here’s how to configure it:

#### Prometheus Metrics
The Spring Boot application already exposes metrics via Actuator. Configure a ServiceMonitor for Prometheus:

```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: session-cleanup-job-monitor
  namespace: default
spec:
  selector:
    matchLabels:
      app: session-cleanup
  endpoints:
  - port: http
    path: /actuator/prometheus
    interval: 30s
```

#### Alerts
Configure alerts for job failures:

```yaml
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: job-alerts
  namespace: default
spec:
  groups:
  - name: kubernetes-jobs
    interval: 30s
    rules:
    - alert: JobFailed
      expr: kube_job_status_failed{job_name=~"session-cleanup.*"} > 0
      for: 5m
      labels:
        severity: warning
      annotations:
        summary: "Job {{ $labels.job_name }} failed"
        description: "Job {{ $labels.job_name }} has failed. Check logs for details."

    - alert: JobTakingTooLong
      expr: kube_job_status_active{job_name=~"session-cleanup.*"} > 0 and time() - kube_job_status_start_time{job_name=~"session-cleanup.*"} > 3600
      for: 10m
      labels:
        severity: warning
      annotations:
        summary: "Job {{ $labels.job_name }} taking too long"
        description: "Job {{ $labels.job_name }} has been running for more than 1 hour."
```

### Best Practices and Lessons Learned
After implementing dozens of jobs in Kubernetes, here are the practices that really make a difference:

**1. Always Use Correct Exit Codes**
```java
// Good
if (success) {
    System.exit(0); // Success
} else {
    System.exit(1); // Failure
}

// Bad - Kubernetes doesn't know if it worked
// Just returning from the method
```

**2. Implement Idempotency**
Jobs can be retried. Ensure running twice doesn’t cause problems:

```java
@Transactional
public void cleanup() {
    // Idempotent: deleting expired records can run multiple times
    int deleted = repository.deleteExpiredRecords(LocalDateTime.now());

    // Not idempotent: sending email
    // if (deleted > 0) {
    //     emailService.send("Deleted " + deleted + " records");
    // }
}
```

**3. Use Appropriate Resources**
Don’t configure resources too low. Jobs that run out of memory will be killed by Kubernetes:

```yaml
resources:
  requests:
    memory: "512Mi"    # Minimum guaranteed
    cpu: "250m"
  limits:
    memory: "1Gi"      # Maximum allowed
    cpu: "500m"
```

**4. Configure Timeouts**
Always define `activeDeadlineSeconds` to avoid jobs that never finish:

```yaml
spec:
  activeDeadlineSeconds: 3600  # 1 hour max
```

**5. Choose the Correct Concurrency Policy**
* **Forbid:** Don’t allow concurrent executions (default for cleanup jobs)
* **Allow:** Allow concurrent executions (useful for independent jobs)
* **Replace:** Cancel running job and start new one (use with caution)

`concurrencyPolicy: Forbid`

### Comparison: @Scheduled vs Kubernetes Jobs

![Comparison Diagram](https://miro.medium.com/v2/resize:fit:720/format:webp/1*X6E9clsr1MtJL_qflqwJ1A.png)

### Conclusion: Reliable and Scalable Periodic Jobs
Migrating from Spring’s @Scheduled to Kubernetes Jobs was one of the best architectural decisions I made. The combination offers:
* **Reliability:** Automatic retry, execution history, and isolation
* **Scalability:** Your main application can scale freely
* **Observability:** Native metrics, centralized logs, and alerts
* **Flexibility:** Schedule changes without redeployment
* **Maintainability:** Clean code, separation of concerns

If you’re facing problems with duplicate jobs, lack of history, or difficulty monitoring executions, this approach will solve your problems.

The complete code is available in this article’s example repository. Clone, test, and adapt to your use case.

Running periodic jobs in production doesn’t have to be a nightmare. With the right tools and proper architecture, it can be robust, reliable, and easy to maintain.
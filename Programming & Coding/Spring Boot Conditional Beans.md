# You’ve Never Really Understood Conditional Beans

## 1. The Hidden Question Spring Asks Before Every Bean
Before Spring creates **any** bean, it quietly asks:

“Should I actually create this bean in the current environment?”

That’s the magic behind **Conditional Annotations** — a system that controls bean creation dynamically at runtime based on **context, properties, classes, or custom logic**.
Every `@ConditionalXxx` you’ve seen is just a **shortcut for a small decision engine** that runs **before bean registration**.

## 2. The Foundation: `@Conditional`
All conditional annotations — like
`@ConditionalOnProperty`, `@ConditionalOnClass`, `@ConditionalOnMissingBean`, etc.
— are built on the **core meta-annotation**:

```java
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Conditional {
    Class<? extends Condition>[] value();
}
```

Every Spring Boot auto-configuration class is guarded by this.
For example:

```java
@Configuration
@ConditionalOnClass(DataSource.class)
public class DataSourceAutoConfiguration {
   ...
}
```

If **DataSource** isn’t on the classpath, this config won’t even load.

## 3. How It Really Works (The Lifecycle Secret)
When Spring starts:

1. It **scans all configuration classes**.
2. For each bean/config, it finds attached `@Conditional` annotations.
3. It delegates the decision to a `Condition` implementation.
4. If any condition **returns false**, Spring silently skips that bean.

The key interface:

```java
public interface Condition {
    boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata);
}
```

That `matches()` method decides your fate.
You get access to:
* `context.getEnvironment()` → for reading application properties
* `context.getBeanFactory()` → check other beans
* `context.getClassLoader()` → check if classes exist
* `metadata.getAnnotationAttributes()` → read your annotation values



## 4. Hidden Internals: The Conditional Evaluation Report
Spring Boot provides a secret debug treasure:
Run your app with
`--debug`

Then in logs, you’ll see a **Condition Evaluation Report**.
It explains **why** a particular auto-configuration or bean was loaded or not.

Example:
```text
Positive matches:
-----------------
   DataSourceAutoConfiguration matched:
      - found DataSource on classpath

Negative matches:
-----------------
   MailSenderAutoConfiguration:
      - required property spring.mail.host not found
```

Even senior developers rarely check this — but it’s pure gold for debugging mysterious config behavior.

![Evaluation Report Screenshot](https://miro.medium.com/v2/resize:fit:720/format:webp/1*I6bdYZ9mtwdqPeFiUM7yGw.png)

## 6. Real-World Use Case — Building a Smart Feature Toggle
Let’s say you have a **feature flag** system for payment gateways:

```java
@Configuration
@ConditionalOnProperty(prefix = "feature.payment", name = "enabled", havingValue = "true")
public class PaymentFeatureConfiguration {
    
    @Bean
    public PaymentProcessor paymentProcessor() {
        return new PaymentProcessor();
    }
}
```

If `feature.payment.enabled=false` in `application.yml`,
this bean **won’t even be created** — it’s not lazy or disabled — it’s **not registered at all**.
That’s the difference most devs miss.
Conditional beans don’t exist in the context — they’re completely absent.

## 7. Going Beyond Built-in: Write Your Own `@Conditional`
You can even define **your own annotation**:

```java
@Target({ ElementType.TYPE, ElementType.METHOD })
@Retention(RetentionPolicy.RUNTIME)
@Conditional(OnWarDeploymentCondition.class)
public @interface ConditionalOnWarDeployment {
}
```

Then implement:
```java
public class OnWarDeploymentCondition implements Condition {
    @Override
    public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
        return context.getResourceLoader()
                      .getClassLoader()
                      .getResource("WEB-INF") != null;
    }
}
```

Now your bean loads **only when deployed as a WAR**!
This is actually how Spring Boot’s own conditional annotations are built internally.



## 8. Combining Multiple Conditions — The “AND / OR” Magic
Spring combines multiple `@Conditional` annotations using **logical AND** —
**all must pass** for the bean to load.

But you can build **OR** logic manually:
`@Conditional({ OnClassACondition.class, OnClassBCondition.class })`

Each `Condition` can internally decide to pass if **either** class is present.
Or even read another annotation’s attributes dynamically.
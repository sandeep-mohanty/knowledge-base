# Spring Boot at Scale: 7 Patterns From a 400-Module Production Codebase

> *What happens to your Spring Boot architecture when it has to support 400 modules, thousands of deployments, and operators who need to customize nearly everything without touching your source code?*

---

Most Spring Boot tutorials walk you through a controller, a service, and a repository. That's enough to ship a feature. It's not enough to build a platform.

The difference between a Spring Boot *application* and a Spring Boot *platform* is extensibility under pressure — the ability to grow to hundreds of modules without coupling them together, let operators swap components without forking your code, and enable or disable entire subsystems via configuration alone.

[Apereo CAS](https://apereo.github.io/cas), a widely-deployed open-source Single Sign-On platform with over 20 years in production, is one of the best real-world examples of Spring Boot engineering at this scale. It runs on Spring Boot 3.x and Java 21+, spans 400+ modules, and is deployed at thousands of universities, hospitals, and enterprises worldwide.

These are seven patterns from that codebase that every Spring Boot developer should understand — not because they're clever tricks, but because they're consistent solutions to real problems that every growing Spring Boot project eventually hits.

---

## 1. Thin Auto-Configuration Wrappers

### The Problem

Spring Boot's auto-configuration mechanism is powerful, but most teams abuse it by dumping all their bean definitions into one giant `@Configuration` class and registering it as an auto-configuration entry point. When that class has 300 lines of `@Bean` methods and a handful of `@Conditional` annotations scattered throughout, debugging *why* a feature didn't activate becomes archaeology.

### The Pattern

CAS separates two concerns that usually get tangled together:

- **Should this module load?** → Handled by the auto-configuration entry point
- **What loads when it does?** → Handled by dedicated `@Configuration` classes

Every auto-configuration class in CAS looks like this:

```java
@EnableConfigurationProperties(CasConfigurationProperties.class)
@EnableScheduling
@ConditionalOnFeatureEnabled(feature = CasFeatureModule.FeatureCatalog.SimpleMFA)
@AutoConfiguration
@Import({
   CasSimpleMultifactorAuthenticationConfiguration.class,
   CasSimpleMultifactorAuthenticationEventExecutionPlanConfiguration.class,
   CasSimpleMultifactorAuthenticationWebflowConfiguration.class,
   CasSimpleMultifactorAuthenticationTicketCatalogConfiguration.class
   // ...
})
public class CasSimpleMultifactorAuthenticationAutoConfiguration {
   // Intentionally empty
}
```

Empty body. Zero bean definitions. This class exists only to:

1. Declare the activation condition (`@ConditionalOnFeatureEnabled`)
2. Enumerate what loads when the condition passes (`@Import`)

The actual bean definitions live in the imported classes, which can be tested independently, reused across different auto-configuration entry points, and read without hunting for the one annotation that blocked everything.

CAS has 272 of these thin wrappers. Every single one follows this pattern.

### Why It Matters

When a feature fails to activate, you look at exactly one place: the thin wrapper. The conditional logic is isolated from the bean definitions. This is the kind of separation that looks obvious in hindsight but that almost no project actually enforces.

---

## 2. Domain-Specific Conditional Annotations

### The Problem

Spring Boot ships with `@ConditionalOnProperty`, `@ConditionalOnClass`, `@ConditionalOnBean`, and a handful of others. They're useful for infrastructure decisions. They're awkward for domain decisions.

When you find yourself writing `@ConditionalOnProperty("cas.feature.mfa.simple.enabled")` across thirty different configuration classes, you've got magic strings, no type safety, and no central record of what features exist.

### The Pattern

CAS builds a custom annotation on top of Spring's `@Conditional` extension point:

```java
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.TYPE, ElementType.METHOD})
@Documented
@Conditional(CasFeatureEnabledCondition.class)
public @interface ConditionalOnFeatureEnabled {
   CasFeatureModule.FeatureCatalog[] feature();
   String module() default StringUtils.EMPTY;
   boolean enabledByDefault() default true;
}
```

The `CasFeatureEnabledCondition` implements Spring's `SpringBootCondition` and evaluates at startup:

```java
public class CasFeatureEnabledCondition extends SpringBootCondition {

   @Override
   public ConditionOutcome getMatchOutcome(ConditionContext context,
                                           AnnotatedTypeMetadata metadata) {
       var attributes = metadata.getAnnotationAttributes(
           ConditionalOnFeatureEnabled.class.getName()
       );
       var features = (CasFeatureModule.FeatureCatalog[]) attributes.get("feature");
       var module = (String) attributes.get("module");
       var enabledByDefault = (boolean) attributes.get("enabledByDefault");

       return getConditionOutcome(context, features, module, enabledByDefault);
   }

   private static ConditionOutcome getConditionOutcome(
           ConditionContext context,
           CasFeatureModule.FeatureCatalog[] features,
           String module,
           boolean enabledByDefault) {

       for (var feature : features) {
           var property = feature.toProperty(module);

           if (!context.getEnvironment().containsProperty(property) && !enabledByDefault) {
               return ConditionOutcome.noMatch(
                   "Feature " + property + " is disabled by default"
               );
           }

           if ("false".equalsIgnoreCase(context.getEnvironment().getProperty(property))) {
               return ConditionOutcome.noMatch(
                   "Feature " + property + " is explicitly disabled"
               );
           }

           feature.register(module); // Record active features in a runtime registry
       }
       return ConditionOutcome.match("All requested features are enabled");
   }
}
```

Two details worth highlighting:

- **`enabledByDefault`**: Core features are on unless explicitly disabled. Optional integrations are off unless explicitly enabled. This distinction prevents operators from accidentally activating half-built integrations they didn't ask for.
- **`feature.register(module)`**: When a feature activates, it's recorded in a static registry. At runtime, you can query which features are currently active — useful for diagnostics endpoints and support tooling.

### The Broader Lesson

`@Conditional` is an extension point, not just a set of built-in annotations. If your domain has a concept that maps cleanly to "should this component load?" — licensing tiers, deployment profiles, feature flags, regional configurations — encode it as a type-safe annotation instead of scattering property key strings across your codebase.

---

## 3. Every Bean Is Replaceable by Default

### The Problem

You're building a library or platform. A user wants to swap your default in-memory cache for Redis. Their options are usually grim: fork your code, subclass three abstract classes, write a `BeanPostProcessor` that intercepts bean creation, or just maintain a patch against your jar.

### The Pattern

CAS puts `@ConditionalOnMissingBean` on every single `@Bean` method, without exception:

```java
@Bean
@ConditionalOnMissingBean(name = "ticketRegistry")
public TicketRegistry ticketRegistry(
       CasConfigurationProperties casProperties,
       @Qualifier("ticketSerializationManager")
       TicketSerializationManager ticketSerializationManager) {
   return new DefaultTicketRegistry(ticketSerializationManager, casProperties);
}
```

The default in-memory `TicketRegistry` only gets created if no other bean named `ticketRegistry` exists in the context. When you add `cas-server-support-redis-ticket-registry` to your dependencies, its auto-configuration creates a `RedisTicketRegistry` with the same bean name — and the default silently steps aside.

Operators can override nearly any component in CAS by defining a `@Bean` with the right name in their own `@Configuration` class. No subclassing. No hooks. No reflection. Just standard Spring bean resolution.

```java
// In the operator's own configuration class:
@Bean
public TicketRegistry ticketRegistry(...) {
   return new MyCustomTicketRegistry(...); // Default never created
}
```

### What Discipline This Requires

The pattern only works if it's applied *everywhere*, not just where someone remembered. One unguarded `@Bean` method creates a component that users can't replace. CAS enforces this in code reviews — `@ConditionalOnMissingBean` is expected on every bean definition, and its absence is a review flag.

If you're building anything that other teams will extend or deploy, this is the single highest-leverage pattern in this list.

---

## 4. The Execution Plan Configurer Pattern

### The Problem

Multiple independent modules need to contribute to a shared capability. The LDAP module registers an authentication handler. The JDBC module registers a different one. The YubiKey module registers an MFA handler. They don't know about each other. They all need to plug into the same pipeline.

The naive solution — a central class that imports all the modules and wires them together — destroys modularity. The slightly-less-naive solution — `@Autowired List<AuthenticationHandler>` — works but lacks ordering, validation, and a clear registration contract.

### The Pattern

CAS splits the problem into two interfaces: a *plan* (a registry that accepts contributions) and a *configurer* (what each module implements to contribute):

```java
// The plan — a registry that modules write into
public interface AuthenticationEventExecutionPlan {
   boolean registerAuthenticationHandler(AuthenticationHandler handler);
   void registerAuthenticationPostProcessor(AuthenticationPostProcessor processor);
   void registerAuthenticationPolicy(AuthenticationPolicy policy);
   void registerAuthenticationHandlerWithPrincipalResolver(
       AuthenticationHandler handler, PrincipalResolver resolver);
   // ...
}

// The configurer — what each module implements
@FunctionalInterface
public interface AuthenticationEventExecutionPlanConfigurer extends Ordered {
   void configureAuthenticationExecutionPlan(AuthenticationEventExecutionPlan plan)
       throws Exception;

   default int getOrder() { return 0; }
}
```

Each module registers its contributions by returning a configurer bean:

```java
// In the LDAP module's configuration class
@Bean
@ConditionalOnMissingBean(name = "ldapAuthenticationEventExecutionPlanConfigurer")
public AuthenticationEventExecutionPlanConfigurer ldapAuthenticationEventExecutionPlanConfigurer(
       @Qualifier("ldapAuthenticationHandlers")
       List<AuthenticationHandler> ldapHandlers) {
   return plan -> ldapHandlers.forEach(plan::registerAuthenticationHandler);
}
```

At startup, CAS collects every `AuthenticationEventExecutionPlanConfigurer` bean from the context, sorts them by `getOrder()`, and calls each one. The LDAP module contributes its handlers. The JDBC module contributes its. The YubiKey module contributes its MFA handler. None of them imports the others. None of them knows the others exist.

### Where Else This Applies

This same pattern appears across CAS for webflow configurers, audit trail configurers, notification senders, service registry configurers, and more. It's the Strategy pattern applied to Spring's bean lifecycle.

If you have a capability that multiple modules should contribute to — request filters, scheduled jobs, health checks, data exporters, OpenAPI spec extensions — this pattern is far cleaner than maintaining a growing `@Autowired List<Something>` that every module has to know about.

---

## 5. BeanSupplier: Runtime Conditional Beans With No-Op Proxies

### The Problem

`@ConditionalOnMissingBean` handles the case where a bean shouldn't exist. But sometimes a bean *needs* to exist — other beans depend on it at wiring time — yet it should be a no-op when a feature is disabled. Putting `if (featureEnabled)` guards everywhere the bean is used is exactly as bad as it sounds.

### The Pattern

CAS introduces a `BeanSupplier` abstraction that makes runtime conditional bean creation readable:

```java
public interface BeanSupplier<T> extends Supplier<T> {
   static <T> BeanSupplier<T> of(Class<T> clazz);

   BeanSupplier<T> when(Supplier<Boolean> condition);
   BeanSupplier<T> supply(Supplier<T> beanSupplier);
   BeanSupplier<T> otherwiseProxy();
   T get();
}
```

In a configuration class it reads almost like a sentence:

```java
@Bean
@RefreshScope(proxyMode = ScopedProxyMode.DEFAULT)
public CasWebflowExecutionPlanConfigurer myWebflowConfigurer(
       ConfigurableApplicationContext applicationContext,
       CasWebflowConfigurer myFlowConfigurer) {

   return BeanSupplier.of(CasWebflowExecutionPlanConfigurer.class)
       .when(MY_FEATURE_CONDITION.given(applicationContext.getEnvironment()))
       .supply(() -> plan -> plan.registerWebflowConfigurer(myFlowConfigurer))
       .otherwiseProxy()
       .get();
}
```

*"Create a bean of this type. When this condition holds, supply the real implementation. Otherwise, create a proxy that does nothing."*

The `otherwiseProxy()` call creates a JDK dynamic proxy at runtime using `java.lang.reflect.Proxy`. The proxy implementation maps return types to sensible safe defaults: empty collections, `Optional.empty()`, `false` for booleans, zero for numeric primitives. The bean exists in the context, satisfies all injection points, and does nothing when invoked.

### Implementation Sketch

For teams wanting to implement something similar:

```java
public class DefaultBeanSupplier<T> implements BeanSupplier<T> {
   private final Class<T> type;
   private Supplier<Boolean> condition = () -> true;
   private Supplier<T> realSupplier;

   @Override
   public T get() {
       if (condition.get()) {
           return realSupplier.get();
       }
       return createNoOpProxy();
   }

   @SuppressWarnings("unchecked")
   private T createNoOpProxy() {
       return (T) Proxy.newProxyInstance(
           type.getClassLoader(),
           new Class[]{type},
           (proxy, method, args) -> defaultReturnValue(method.getReturnType())
       );
   }

   private Object defaultReturnValue(Class<?> returnType) {
       if (returnType == boolean.class) return false;
       if (returnType == int.class || returnType == long.class) return 0;
       if (returnType == Optional.class) return Optional.empty();
       if (Collection.class.isAssignableFrom(returnType)) return List.of();
       return null;
   }
}
```

The decision happens once at wiring time. Consumers never know whether they got the real implementation or a no-op.

---

## 6. proxyBeanMethods = false and @RefreshScope as Non-Negotiable Defaults

These two settings appear on nearly every class and bean definition in the codebase. They're worth understanding as a pair.

### proxyBeanMethods = false

By default, Spring creates CGLIB subclasses of `@Configuration` classes. This allows one `@Bean` method to call another and get the singleton back rather than a new instance:

```java
@Configuration
class MyConfig {
   @Bean
   public ServiceA serviceA() { return new ServiceA(serviceB()); } // Intercepted — returns singleton

   @Bean
   public ServiceB serviceB() { return new ServiceB(); }
}
```

This interception requires CGLIB proxying, which has real costs: class generation at startup, memory overhead, and slightly slower bean initialization. CAS never calls `@Bean` methods directly — it uses constructor injection exclusively — so the proxy is pure overhead.

```java
// Every configuration class in CAS:
@Configuration(value = "CasCoreWebConfiguration", proxyBeanMethods = false)
class CasCoreWebConfiguration {
   // Constructor injection only — CGLIB proxy not needed
}
```

Across 400+ configuration classes, this meaningfully reduces startup time and memory footprint. Even if you don't reach CAS's scale, `proxyBeanMethods = false` is a good habit for any configuration class that doesn't use internal `@Bean` method calls.

### @RefreshScope

CAS is a production SSO system. Restarting it to change a timeout value or rotate a credential is unacceptable. Any bean that reads from `CasConfigurationProperties` gets `@RefreshScope`:

```java
@Bean
@RefreshScope(proxyMode = ScopedProxyMode.DEFAULT)
@ConditionalOnMissingBean(name = "casMessageSource")
public MessageSource messageSource(CasConfigurationProperties casProperties) {
   var bundle = new CasReloadableMessageBundle();
   bundle.setDefaultEncoding(casProperties.getMessageBundle().getEncoding());
   return bundle;
}
```

When configuration changes — via Spring Cloud Config, an actuator `/refresh` call, or CAS's own property reloading — `@RefreshScope` beans are destroyed and recreated with the new values. No restart required.

The key discipline: `@RefreshScope` only works if applied *consistently*. If five beans are refresh-aware and the sixth isn't, a configuration change produces a partially-updated system that's harder to reason about than a full restart. Apply it to all configuration-reading beans or none.

---

## 7. Application Events as a First-Class Architectural Concept

### The Problem

The audit module needs to know when authentication succeeds. The risk engine needs to react to failed logins. The notification system needs to trigger on ticket creation. If each of these modules imports the authentication module directly, you've built a dependency graph that nobody can untangle in six months.

### The Pattern

CAS publishes `ApplicationEvent` subclasses for everything significant that happens in the system: authentication success and failure, ticket creation and destruction, logout, service access, MFA triggers, and more.

The base event class is minimal:

```java
public abstract class AbstractCasEvent extends ApplicationEvent {
   private final ClientInfo clientInfo; // IP, user agent, geolocation

   protected AbstractCasEvent(Object source, ClientInfo clientInfo) {
       super(source);
       this.clientInfo = clientInfo;
   }
}
```

Listener interfaces declare handlers for specific event types, annotated `@Async` so they never block the main authentication flow:

```java
public interface CasAuthenticationEventListener {

   @EventListener
   @Async
   void handleAuthenticationSuccess(CasAuthenticationSuccessEvent event) throws Throwable;

   @EventListener
   @Async
   void handleAuthenticationFailure(CasAuthenticationFailureEvent event) throws Throwable;

   @EventListener
   @Async
   void handleTicketGrantingTicketCreated(CasTicketGrantingTicketCreatedEvent event) throws Throwable;
}
```

Any module that cares about authentication events implements this interface and registers itself as a Spring bean. The audit module gets its trail. The risk engine gets its signals. The notification system gets its triggers. None of them touches the authentication pipeline. None of them imports the others.

### Practical Advice for Your Own Codebase

Define a hierarchy of events for every meaningful domain action:

```java
// Base event
public abstract class OrderEvent extends ApplicationEvent {
   private final String orderId;
   private final String customerId;
   // ...
}

// Specific events
public class OrderPlacedEvent extends OrderEvent { ... }
public class OrderShippedEvent extends OrderEvent { ... }
public class OrderCancelledEvent extends OrderEvent { ... }
public class PaymentFailedEvent extends OrderEvent { ... }
```

The audit log, the notification service, the fraud detection module, and the warehouse system all subscribe independently. The ordering module publishes and forgets. You can add a new subscriber without touching existing code.

The `@Async` annotation on listeners is critical for domain events — you don't want an audit log write to add latency to a checkout request.

---

## The Compound Effect: Why Consistency Beats Cleverness

None of these patterns requires advanced Spring knowledge. `@ConditionalOnMissingBean` is in the official docs. `proxyBeanMethods = false` is a footnote in the reference guide. `ApplicationEvent` shipped with Spring 1.0.

What makes the CAS codebase remarkable isn't any individual pattern — it's the *discipline* of applying all of them uniformly across 400 modules, enforced through code review, for years.

The thin wrapper pattern works because *every* module follows it — debugging is predictable because the structure is predictable. `@ConditionalOnMissingBean` enables extensibility because it's on *every* bean, not just the ones someone remembered on a good day. The configurer pattern scales because it's the *only* way modules contribute to shared capabilities — there's no parallel ad-hoc wiring to maintain alongside it.

This is a lesson that applies well beyond Spring Boot: **good architecture is mostly just good patterns applied consistently**. One `@ConditionalOnMissingBean` doesn't make a platform. Four hundred of them, applied without exception, does.

The jump from "Spring Boot application" to "Spring Boot platform" isn't a technology change. It's a discipline change.
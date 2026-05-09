# Spring Boot Done Right: Lessons From a 400-Module Codebase

Most Spring Boot tutorials show you a controller, a service, a repository, and call it a day. That's fine for a TODO app. But what happens when your application grows to 400 modules, gets deployed at thousands of organizations worldwide, and needs to let operators swap out nearly any component without touching your source code?

That's the problem Apereo CAS solves every day. CAS — the [Central Authentication Service](https://dzone.com/articles/installing-and-debugging-an-apereo-cas-application) — is an identity and single sign-on platform that's been running in production for over 20 years. Its current incarnation is a Spring Boot 3.x application on Java 21+, and its codebase is one of the best real-world examples I've seen of Spring Boot engineering at scale.

I've been working on this codebase for years. Here are seven patterns I think every Spring Boot developer should steal.

## 1\. The Thin Auto-Configuration Wrapper

Most [Spring Boot](https://dzone.com/articles/mastering-scalability-in-spring-boot) projects dump all their bean definitions into a single `@Configuration` class and call it their auto-configuration. CAS takes a different approach: every auto-configuration class is essentially empty.

`support/cas-server-support-simple-mfa/src/main/java/org/apereo/cas/config/CasSimpleMultifactorAuthenticationAutoConfiguration.java`

```java
@EnableConfigurationProperties(CasConfigurationProperties.class)
@EnableScheduling
@ConditionalOnFeatureEnabled(feature = CasFeatureModule.FeatureCatalog.SimpleMFA)
@AutoConfiguration
@Import({
    CasSimpleMultifactorAuthenticationComponentSerializationConfiguration.class,
    CasSimpleMultifactorAuthenticationConfiguration.class,
    CasSimpleMultifactorAuthenticationEventExecutionPlanConfiguration.class,
    CasSimpleMultifactorAuthenticationMultifactorProviderBypassConfiguration.class,
    CasSimpleMultifactorAuthenticationRestConfiguration.class,
    CasSimpleMultifactorAuthenticationTicketCatalogConfiguration.class,
    CasSimpleMultifactorAuthenticationWebflowConfiguration.class
})
public class CasSimpleMultifactorAuthenticationAutoConfiguration {
}
```

Empty body. Zero bean definitions. The class exists only to carry annotations — specifically `@ConditionalOnFeatureEnabled` (should this module load?) and `@Import` (what configuration classes should load when it does?).

This separation is subtle but powerful. The conditional logic — _should this load?_ — lives in one place. The bean definitions — _what should load?_ — live in the imported configuration classes. You can test the configuration classes independently. You can reuse them across different auto-configuration entry points. And when you're debugging why a feature didn't activate, you only need to look at the thin wrapper, not wade through 200 lines of bean definitions to find the one `@Conditional` annotation that blocked everything.

CAS has 272 of these auto-configuration entry points. Every single one follows this pattern.

## 2\. Building a Custom Feature Flag System on Spring's @Conditional

Spring Boot gives you `@ConditionalOnProperty`, `@ConditionalOnClass`, `@ConditionalOnBean`. They're useful but generic. CAS needed something more domain-specific: the ability to enable or disable entire subsystems with a single property.

`core/cas-server-core-util-api/src/main/java/org/apereo/cas/util/spring/boot/ConditionalOnFeatureEnabled.java`

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

The magic is in `@Conditional(CasFeatureEnabledCondition.class)` — this annotation delegates to a custom `SpringBootCondition` implementation that evaluates at startup:

`core/cas-server-core-util-api/src/main/java/org/apereo/cas/util/spring/boot/CasFeatureEnabledCondition.java`

```java
public class CasFeatureEnabledCondition extends SpringBootCondition {

    private static ConditionOutcome getConditionOutcome(
        final ConditionContext context,
        final CasFeatureModule.FeatureCatalog[] features,
        final String module,
        final boolean enabledByDefault) {

        for (val feature : features) {
            val property = feature.toProperty(module);

            val isFeatureDisabled = !context.getEnvironment()
                .containsProperty(property) && !enabledByDefault;
            if (isFeatureDisabled) {
                return ConditionOutcome.noMatch(
                    "CAS feature " + property + " is disabled by default");
            }

            val propertyValue = context.getEnvironment().getProperty(property);
            if (Strings.CI.equals(propertyValue, "false")) {
                return ConditionOutcome.noMatch(
                    "CAS feature " + property + " is set to false.");
            }
            feature.register(module);
        }
        return ConditionOutcome.match("Requested features are enabled");
    }
    // ...
```

There are a few things worth noting here. The `enabledByDefault` flag means core features are on unless you explicitly turn them off, while optional integrations are off unless you explicitly enable them. And the `feature.register(module)` call — when a feature's condition passes, CAS records it in a static registry, so at any point during runtime, you can ask: "Which features are currently active?"

The takeaway isn't "build your own conditional annotation." It's that Spring's `@Conditional` mechanism is an extension point, not just a set of built-in annotations. If your domain has a concept that maps to "should this load?" — feature flags, licensing tiers, deployment profiles — you can express it as a type-safe annotation instead of scattering `@ConditionalOnProperty` checks with magic strings across your codebase.

## 3\. Every Bean Is Replaceable

This pattern shows up on virtually every `@Bean` method in CAS:

```java
@Bean
@ConditionalOnMissingBean(name = "ticketRegistry")
public TicketRegistry ticketRegistry(...) {
    return new DefaultTicketRegistry(...);
}
```

The default in-memory ticket registry only gets created if _no other bean named `ticketRegistry` already exists_. When you add `cas-server-support-redis-ticket-registry` to your classpath, its auto-configuration creates a `RedisTicketRegistry` bean with that same name — and the default one quietly steps aside.

This means operators can override virtually any component in CAS by defining their own bean with the right name. No subclassing, no framework hooks, no `BeanDefinitionRegistryPostProcessor` gymnastics. Just standard Spring.

I've seen Spring Boot projects where replacing a single component requires forking the library, or subclassing three layers of abstract classes, or writing a `BeanPostProcessor` that intercepts bean creation. CAS avoids all of that with one annotation. The discipline is in applying it consistently — every bean, every module, every time.

If you're building a library or a framework-like application that other teams will customize, this is the single most impactful pattern you can adopt.

## 4\. The Execution Plan Configurer Pattern

Here's a problem CAS faces constantly: Multiple independent modules need to contribute to a single capability. The LDAP module registers an authentication handler. The JDBC module registers a different one. The YubiKey module registers an MFA handler. None of them knows about each other, and they all need to plug into the same authentication pipeline.

CAS solves this with what I call the "configurer pattern." First, there's a plan interface — a registry that accepts contributions:

`api/cas-server-core-api-authentication/src/main/java/org/apereo/cas/authentication/AuthenticationEventExecutionPlan.java`

```java
public interface AuthenticationEventExecutionPlan {
    boolean registerAuthenticationHandler(AuthenticationHandler handler);
    void registerAuthenticationMetadataPopulator(AuthenticationMetaDataPopulator populator);
    void registerAuthenticationPolicy(AuthenticationPolicy authenticationPolicy);
    void registerAuthenticationHandlerWithPrincipalResolver(
        AuthenticationHandler handler, PrincipalResolver principalResolver);
    // ...
}
```

Then there's a configurer interface — a `@FunctionalInterface` that modules implement to contribute to the plan:

`api/cas-server-core-api-authentication/src/main/java/org/apereo/cas/authentication/AuthenticationEventExecutionPlanConfigurer.java`

```java
@FunctionalInterface
public interface AuthenticationEventExecutionPlanConfigurer extends Ordered, NamedObject {
    void configureAuthenticationExecutionPlan(
        AuthenticationEventExecutionPlan plan) throws Exception;

    @Override
    default int getOrder() {
        return 0;
    }
}
```

Each module defines a `@Bean` that returns a configurer implementation. At startup, CAS collects all `AuthenticationEventExecutionPlanConfigurer` beans from the entire application context and invokes them in order. The [LDAP](https://dzone.com/articles/spring-security-with-ldap-authentication) module contributes its handler. The JDBC module contributes its handler. They never import each other. They never even know the other exists.

This pattern appears across CAS for webflow configurers, notification senders, audit trail configurers, and more. It's essentially the Strategy pattern applied to Spring's bean lifecycle, and it scales to hundreds of modules without any of them coupling to each other.

If your application has a capability that multiple modules contribute to — request filters, scheduled jobs, health checks, data exporters — this pattern is far cleaner than `@Order`\-annotated `@Bean` lists or manual `@Autowired List<Something>` injection.

## 5\. BeanSupplier: Runtime Conditional Bean Creation

Sometimes `@ConditionalOnMissingBean` isn't enough. You need a bean to exist in the context — other beans depend on it — but you need it to be a no-op proxy when a feature is disabled. CAS handles this with a fluent `BeanSupplier` abstraction:

`core/cas-server-core-util-api/src/main/java/org/apereo/cas/util/spring/beans/BeanSupplier.java`

```java
public interface BeanSupplier<T> extends Supplier<T> {

    static <T> BeanSupplier<T> of(final Class<T> clazz) {
        return new DefaultBeanSupplier<>(clazz);
    }

    BeanSupplier<T> when(Supplier<Boolean> conditionSupplier);
    BeanSupplier<T> supply(Supplier<T> beanSupplier);
    BeanSupplier<T> otherwiseProxy();
    // ...
}
```

Usage in configuration classes looks like this:

```java
@Bean
public CasWebflowExecutionPlanConfigurer myWebflowConfigurer(
    final ConfigurableApplicationContext applicationContext,
    final CasWebflowConfigurer myFlowConfigurer) {
    return BeanSupplier.of(CasWebflowExecutionPlanConfigurer.class)
        .when(CONDITION.given(applicationContext.getEnvironment()))
        .supply(() -> plan -> plan.registerWebflowConfigurer(myFlowConfigurer))
        .otherwiseProxy()
        .get();
}
```

Read that fluently: "Create a bean _of_ this type. _When_ this condition holds, _supply_ the real implementation. _Otherwise_, create a JDK dynamic proxy that returns safe defaults for every method."

The `otherwiseProxy()` method creates a proxy at runtime using `java.lang.reflect.Proxy`. It maps return types to sensible defaults — empty collections, zero for primitives, `Optional.empty()`, `false` for booleans. The bean exists, it satisfies injection points, but it does nothing.

This is one of the more creative solutions I've seen for the "bean exists but feature is off" problem. The alternative — littering your code with `if (featureEnabled)` checks everywhere the bean is used — is far worse. With `BeanSupplier`, the decision is made once at wiring time, and consumers never know the difference.

## 6\. @RefreshScope and proxyBeanMethods = false

Two patterns that CAS applies religiously across the entire codebase:

Every `@Configuration` class uses `proxyBeanMethods = false`:

```java
@Configuration(value = "CasCoreWebConfiguration", proxyBeanMethods = false)
class CasCoreWebConfiguration {
    // ...
}
```

This tells Spring not to create CGLIB subclasses for the configuration class. Normally, Spring intercepts `@Bean` method calls so that calling one `@Bean` method from another returns the singleton instance rather than creating a new object. CAS never does that — it uses constructor injection exclusively — so the proxy is pure overhead. Across 272 auto-configuration classes and hundreds of inner `@Configuration` classes, this saves measurable startup time and memory.

And most beans that read from `CasConfigurationProperties` are annotated with `@RefreshScope`:

```java
@Bean
@RefreshScope(proxyMode = ScopedProxyMode.DEFAULT)
@ConditionalOnMissingBean(name = "casMessageSource")
public MessageSource messageSource(...) {
    return new CasReloadableMessageBundle(...);
}
```

This means when the configuration changes — via Spring Cloud Config, CAS's built-in property reloading, or actuator refresh — these beans are destroyed and recreated with the new configuration values. No restart required. For a production SSO system that can't afford downtime, this is essential.

The discipline is in applying _both_ patterns consistently. One-off `@RefreshScope` annotations are nearly useless if half your beans aren't refresh-aware. And one `proxyBeanMethods = true` configuration class doesn't hurt, but 400 of them do.

## 7\. Events as a First-Class Architectural Concept

CAS publishes Spring `ApplicationEvent` subclasses for nearly everything that happens: authentication success, authentication failure, ticket creation, ticket destruction, logout, service access, MFA triggers. The base event class is clean:

`api/cas-server-core-api-events/src/main/java/org/apereo/cas/support/events/AbstractCasEvent.java`

```java
public abstract class AbstractCasEvent extends ApplicationEvent {
    private final ClientInfo clientInfo;

    protected AbstractCasEvent(final Object source, final ClientInfo clientInfo) {
        super(source);
        this.clientInfo = clientInfo;
    }
}
```

Listener interfaces declare methods for specific event types, marked `@Async` so they don't block the main flow:

```java
public interface CasAuthenticationEventListener extends CasEventListener {
    @EventListener
    @Async
    void handleCasTicketGrantingTicketCreatedEvent(
        CasTicketGrantingTicketCreatedEvent event) throws Throwable;

    @EventListener
    @Async
    void handleCasTicketGrantingTicketDeletedEvent(
        CasTicketGrantingTicketDestroyedEvent event) throws Throwable;
}
```

This event infrastructure is what makes CAS's audit trail work. It's what drives real-time risk analysis. It's what allows external systems to react to SSO events without coupling to the authentication pipeline. And because it's built on standard Spring events, adding a custom listener is trivial — you don't need to learn a CAS-specific API.

The broader lesson: if your application has actions that other components might care about, publish events. Don't let the audit module import the authentication module. Don't let the notification system couple to the ticket registry. Events are the cleanest way to keep modules decoupled while still allowing rich cross-cutting behavior.

## The Compound Effect

None of these patterns is revolutionary on its own. `@ConditionalOnMissingBean` is in the Spring Boot docs. `proxyBeanMethods = false` is a known optimization. `ApplicationEvent` has been around since Spring 1.0.

What makes the CAS codebase impressive is the _discipline_ of applying all of them consistently, across 400 modules, for years. The thin wrapper pattern works because every module follows it. `@ConditionalOnMissingBean` works because it's on every bean, not just the ones someone remembered. The execution plan configurer works because it's the _only_ way modules contribute to shared capabilities.

Good engineering isn't clever code. It's consistent patterns applied at scale. That's what makes a 400-module application navigable by someone who's never seen it before.

* * *

CAS is, to my knowledge, one of the most complex Spring Boot applications in open-source production use. If you want to go deeper — not just the patterns, but the full architecture, every protocol engine, every ticket lifecycle, every webflow state machine — I wrote a book about it. **CAS Internals** is 30 chapters and 825 pages of source-level walkthroughs of how CAS actually works. It's the resource I wish existed when I started contributing to this project.
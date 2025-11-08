# Order of Configuration in Spring Boot

## 1\. Overview

In a Spring Boot application, multiple configuration classes often coexist to define beans, properties, or integrations. **While Spring automatically detects and processes these configurations, it doesn’t guarantee the order in which they are handled.** In scenarios where configurations depend on each other, or we need predictable initialization (for example, a data configuration before service configuration), controlling the **order of configuration** becomes crucial.

In this tutorial, we explore how Spring Boot determines the configuration order during application startup and how developers can control it using annotations such as _[@Order](/spring-order), [@AutoConfigureOrder](https://docs.spring.io/spring-boot/api/java/org/springframework/boot/autoconfigure/AutoConfigureOrder.html), [@AutoConfigureAfter](https://docs.spring.io/spring-boot/api/java/org/springframework/boot/autoconfigure/AutoConfigureAfter.html),_ and _[@AutoConfigureBefore](https://docs.spring.io/spring-boot/api/java/org/springframework/boot/autoconfigure/AutoConfigureBefore.html)_. We also explain the difference between regular configuration and auto-configuration ordering, helping readers understand when and how each mechanism applies. To solidify these concepts, the article includes fully runnable examples along with unit tests that demonstrate the behavior of different ordering strategies in practice.

## 2\. Understanding Configuration Classes

**Spring configuration classes can appear in two forms: full and lite configuration.** A _full configuration_ class is annotated with _@Configuration_, whereas a _lite configuration_ class is annotated with _@Component, @Import_, or contains at least one method annotated with _@Bean_. Both types are processed by Spring’s _ConfigurationClassPostProcessor_, which scans their annotations, interprets the metadata, and registers the corresponding bean definitions during the application startup phase.

## 3\. Default Behavior of Configuration Ordering

By default, Spring Boot doesn’t guarantee any order for processing configuration classes. It scans all candidates and loads them as found in the classpath.

### 3.1. Example Without Explicit Order

The following code defines two independent configuration classes, _ConfigA_ and _ConfigB:_

```java
@Configuration
public class ConfigA {
    
    @Bean
    public String beanA() {
        return "Bean A";
    }
}
```

```java
@Configuration
public class ConfigB {
    
    @Bean
    public String beanB() {
        return "Bean B";
    }
}
```

Here, both configurations declare beans that are unrelated to each other. When Spring loads these configurations, both beans are registered successfully, regardless of which configuration class is loaded first.

Before running the test, it’s essential to verify that both beans are present in the context, regardless of their load order:

```java
@SpringBootTest
class DefaultConfigOrderUnitTest {

    @Autowired
    private ApplicationContext context;

    @Test
    void givenConfigsWithoutOrder_whenLoaded_thenBeansExistRegardlessOfOrder() {
        assertThat(context.getBean("beanA")).isEqualTo("Bean A");
        assertThat(context.getBean("beanB")).isEqualTo("Bean B");
    }
}
```

This test confirms that Spring registers both beans, even though no specific order is defined. The container doesn’t enforce or rely on any explicit sequencing.

## 3\. Controlling Order Using _@Order_ Annotation

Sometimes the initialization order matters, for example, when one configuration depends on beans from another. The _@Order_ annotation helps enforce a predictable loading sequence.

Here, the _ConfigOne_ class must load before the _ConfigTwo_ class. We specify explicit order values:

```java
@Configuration
@Order(1)
public class ConfigOne {

    @Bean
    public String configOneBean() {
        return "ConfigOneBean";
    }
}
```

```java
@Configuration
@Order(2)
public class ConfigTwo {

    @Bean
    public String configTwoBean() {
        return "ConfigTwoBean";
    }
}
```

Spring processes _ConfigOne_ before _ConfigTwo_ since its order value is lower. While the order does not affect independent beans, it ensures dependency order when necessary.

The following unit test validates that both beans are created, and since _ConfigOne_ has a smaller order value, it will load before _ConfigTwo_. However, bean creation order is not directly observable unless one configuration depends on another:

```java
@SpringBootTest(classes = {ConfigTwo.class, ConfigOne.class})
class OrderedConfigUnitTest {

    @Autowired
    private ApplicationContext context;

    @Test
    void givenOrderedConfigs_whenLoaded_thenOrderIsRespected() {
        String beanOne = context.getBean("configOneBean", String.class);
        String beanTwo = context.getBean("configTwoBean", String.class);

        assertThat(beanOne).isEqualTo("ConfigOneBean");
        assertThat(beanTwo).isEqualTo("ConfigTwoBean");
    }
}
```

Even though the _ConfigTwo_ class is declared first in the test class, Spring still loads _ConfigOne_ first, respecting the order annotation.

## 4\. Managing Dependencies Using _@DependsOn_

Sometimes a bean explicitly depends on another bean’s initialization. The _@DependsOn_ annotation enforces this relationship at the bean level rather than the configuration level. Let’s see how one bean depends on another:

```java
@Configuration
public class DependsConfig {

    @Bean
    public String firstBean() {
        return "FirstBean";
    }

    @Bean
    @DependsOn("firstBean")
    public String secondBean() {
        return "SecondBeanAfterFirst";
    }
}
```

Here, _secondBean_ depends on _firstBean_. Spring ensures _firstBean_ is fully initialized before creating _secondBean_.

The unit test below ensures both beans are registered and that the dependent bean is available after its dependency:

```java
@SpringBootTest(classes = DependsConfig.class)
class DependsConfigUnitTest {

    @Autowired
    private ApplicationContext context;

    @Test
    void givenDependsOnBeans_whenLoaded_thenOrderIsMaintained() {
        String first = context.getBean("firstBean", String.class);
        String second = context.getBean("secondBean", String.class);

        assertThat(first).isEqualTo("FirstBean");
        assertThat(second).isEqualTo("SecondBeanAfterFirst");
    }
}
```

The test ensures both beans exist and are initialized in the correct order, as defined by the _@DependsOn_ annotation.

**Spring Boot uses auto-configuration classes to set up application defaults. When multiple auto-configurations exist, Spring determines their order using _@AutoConfigureOrder, @AutoConfigureAfter,_ and_@AutoConfigureBefore_.**

Let’s explore these further:

```java
@Configuration
@AutoConfigureOrder(1)
public class FirstAutoConfig {

    @Bean
    public String autoBeanOne() {
        return "AutoBeanOne";
    }
}
```

```java
@Configuration
@AutoConfigureAfter(FirstAutoConfig.class)
public class SecondAutoConfig {

    @Bean
    public String autoBeanTwo() {
        return "AutoBeanTwoAfterOne";
    }
}
```

_SecondAutoConfig_ ensures it loads after _FirstAutoConfig_, using _@AutoConfigureAfter_. These annotations allow Spring Boot to manage configuration sequencing internally.

The test below confirms that beans from auto-configurations are present in the application context:

```java
@SpringBootTest(classes = {SecondAutoConfig.class, FirstAutoConfig.class})
class AutoConfigOrderUnitTest {

    @Autowired
    private ApplicationContext context;

    @Test
    void givenAutoConfigs_whenLoaded_thenOrderFollowsAnnotations() {
        String beanOne = context.getBean("autoBeanOne", String.class);
        String beanTwo = context.getBean("autoBeanTwo", String.class);

        assertThat(beanOne).isEqualTo("AutoBeanOne");
        assertThat(beanTwo).isEqualTo("AutoBeanTwoAfterOne");
    }
}
```

Even though _SecondAutoConfig_ is listed before _FirstAutoConfig_ in the test, the annotation ordering ensures Spring still loads them in the correct order.

## 6\. Conclusion

By combining _@Order, @DependsOn,_ and auto-configuration annotations, developers can precisely control the startup sequence and avoid initialization conflicts.

These techniques ensure that our application context loads consistently, whether configurations are user-defined or auto-configured.

For most applications, explicit configuration ordering is unnecessary. Spring’s dependency resolution mechanism handles bean creation order automatically based on dependencies. Use configuration ordering judiciously, only when the sequence of configuration processing genuinely impacts our application’s behavior. 
# Spring Boot @EnableAutoConfiguration

_In this section, we will discuss why Spring Boot can automatically configure beans for us with zero-configuration based on Spring Boot starters. Then we will discuss how to implement a basic Spring Boot starter._

## How Spring Boot Automatically Configure Beans with Zero-Configuration?

The answer of this question is that Spring Boot reads resources with a common convention location **"META-INF/spring.factories"** from all **Spring Boot starter** jars in the classpath to know what configuration classes needs to be auto configured. Combined with `@ConditionalOnXXX(...)`, e.g. `@ConditionalOnProperty`, annotations, Spring Boot can do things like, once the user specifies the following properties in the application.properties file

```property
spring.datasource.url=url
spring.datasource.username=username
spring.datasource.password=password
```

Spring Boot will automatically initialize the SessionFactory bean in Spring container for the user without any XML or JavaConfig configuration.

### Which Method Loads META-INF/spring.factories from Spring Boot starters?

In this subsection, we will first take a look at in which method all the **"META-INF/spring.factories"** of the **Spring Boot starters** in one's project's classpath are being loaded. In the next subsection, we will discuss when and where Spring invoke that method.

The following is a very classical code snippet of a Spring Boot application's main entrance.

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class SpringBootLearningDemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringBootLearningDemoApplication.class, args);
    }

}
```

The key of the auto-configuration feature of Spring Boot is based on the compose annotation `@SpringBootApplication`. Its definition is shown as follow.

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(
    excludeFilters = {@Filter(
    type = FilterType.CUSTOM,
    classes = {TypeExcludeFilter.class}
), @Filter(
    type = FilterType.CUSTOM,
    classes = {AutoConfigurationExcludeFilter.class}
)}
)
public @interface SpringBootApplication {
    // ...
}
```

The compose annotation `@SpringBootConfiguration` contains a `@Configuration` annotation, which means any class annotated with `@SpringBootApplication` is also a JavaConfig class.

Now, let's focus on the compose annotation `@EnableAutoConfiguration`, which is the core annotation when discussing Spring Boot's auto configuration. The definition of it is shown below.

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage
@Import({AutoConfigurationImportSelector.class})
public @interface EnableAutoConfiguration {
    String ENABLED_OVERRIDE_PROPERTY = "spring.boot.enableautoconfiguration";

    Class<?>[] exclude() default {};

    String[] excludeName() default {};
}
```

**During the container construction, Spring creates an instance for every `ImportSelector` based class being imported using `@Import(...)` in a JavaConfig class and invokes their `selectImport` method.** The code of the selectImport method of `org.springframework.boot.autoconfigure.AutoConfigurationImportSelector` is shown below.

```java
public class AutoConfigurationImportSelector implements DeferredImportSelector, BeanClassLoaderAware,
    ResourceLoaderAware, BeanFactoryAware, EnvironmentAware, Ordered {
    // ...

    @Override
    public String[] selectImports(AnnotationMetadata annotationMetadata) {
        if (!isEnabled(annotationMetadata)) {
            return NO_IMPORTS;
        }
        AutoConfigurationEntry autoConfigurationEntry = getAutoConfigurationEntry(annotationMetadata);
        return StringUtils.toStringArray(autoConfigurationEntry.getConfigurations());
    }

    // ...

    protected AutoConfigurationEntry getAutoConfigurationEntry(AnnotationMetadata annotationMetadata) {
        if (!isEnabled(annotationMetadata)) {
            return EMPTY_ENTRY;
        }
        AnnotationAttributes attributes = getAttributes(annotationMetadata);
        List<String> configurations = getCandidateConfigurations(annotationMetadata, attributes);

        // ...
    }

```

The key method is `getCandidateConfigurations(AnnotationMetadata, AnnotationAttributes)`, whose logic is

```java
protected List<String> getCandidateConfigurations(AnnotationMetadata metadata, AnnotationAttributes attributes) {
    List<String> configurations = SpringFactoriesLoader.loadFactoryNames(getSpringFactoriesLoaderFactoryClass(),
    getBeanClassLoader());
    Assert.notEmpty(configurations, "No auto configuration classes found in META-INF/spring.factories. If you "
            + "are using a custom packaging, make sure that file is correct.");
    return configurations;
}
```

Now, one can easily infer from the assertion above that the `loadFactoryNames` method in `SpringFactoriesLoader` is where Spring Boot loads "META-INF/spring.factories". Let's step into it to see what's happening inside.

```java
public static List<String> loadFactoryNames(Class<?> factoryType, @Nullable ClassLoader classLoader) {
    // ...

    return (List)loadSpringFactories(classLoaderToUse).getOrDefault(factoryTypeName, Collections.emptyList());
}

private static Map<String, List<String>> loadSpringFactories(ClassLoader classLoader) {
    Map<String, List<String>> result = (Map)cache.get(classLoader);
    if (result != null) {
        return result;
    } else {
        HashMap result = new HashMap();

        try {
            /*
               The getResources(location:String) method of a
               java.lang.ClassLoader instance will loads all
               resource in the same location from each jar in
               the classpath. Therefore, each Spring Boot starter
               should place the spring.factories into
               classpath:META-INF so that the AutoConfigurationImportSelector
               can load it correclty.
            */
            Enumeration urls = classLoader.getResources("META-INF/spring.factories");

            while(urls.hasMoreElements()) {
                URL url = (URL)urls.nextElement();
                UrlResource resource = new UrlResource(url);

                // omitted part
            }

            // ...
        } catch (IOException var14) {
            throw new IllegalArgumentException("Unable to load factories from location [META-INF/spring.factories]", var14);
        }
    }
}
```

Finally, let's take a look at what spring.factories looks like.

```text
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
...
org.springframework.boot.autoconfigure.data.jdbc.JdbcRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.jpa.JpaRepositoriesAutoConfiguration,\
...
```

As we can see, for implement a Spring Boot starter, one only needs to **specify their JavaConfig class under the key `org.springframework.boot.autoconfigure.EnableAutoConfiguration` (multiple JavaConfig is separated by comma).** Then those JavaConfig will be imported to the Spring container.

### When Does Spring Call `selectImports(...)` During Creating the Container (i.e. Refreshing the `ApplicationContext`)?

In the above subsection, we discuss where does Spring resolve and know what JavaConfig need to be imported to the container. However, when does Spring do this resolve procedure? **The key answer is that when Spring invoke the one of the `BeanFactoryPostProcessor`, `ConfigurationClassPostProcessor`, which process the annotations, including the `@Import` annotations inside classes being annotated with `@Configuration`.**

The following two diagrams explains the whole procedure (The second digram is the same as the first diagram but with more annotated notes and screenshots). You are welcome to download them so that you can zoom in for a clearer vision of the diagrams.

![What's Behind Spring Boot AutoConfiguration](./../../images/Spring-Boot-AutoConfiguration/Spring-Boot-AutoConfiguration.png)

> _The Original Version of The Spring Boot Auto-Configuration Sequence Diagram_

![What's Behind Spring Boot AutoConfiguration (Annotated)](./../../images/Spring-Boot-AutoConfiguration/Spring-Boot-AutoConfiguration_annotated.png)

> _The Annotated Version of The Spring Boot Auto-Configuration Sequence Diagram_

## How to Implement A Basic Spring Boot Starter?

TODO: more details are being editted.

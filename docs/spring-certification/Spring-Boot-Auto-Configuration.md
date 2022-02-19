---

title: Spring Boot Auto Configuration

---

* [How does Spring Boot know what to configure?](#how-does-spring-boot-know-what-to-configure)
* [What does @EnableAutoConfiguration do?](#what-does-enableautoconfiguration-do)
* [What does @SpringBootApplication do?](#what-does-springbootapplication-do)
* [Does Spring Boot do component scanning? Where does it look by default?](#does-spring-boot-do-component-scanning-where-does-it-look-by-default)
* [How are DataSource and JdbcTemplate auto-configured?](#how-are-datasource-and-jdbctemplate-auto-configured)
* [What is spring.factories file for?](#what-is-springfactories-file-for)
* [How do you customize Spring Boot auto configuration?](#how-do-you-customize-spring-boot-auto-configuration)
* [What are the examples of @Conditional annotations? How are they used?](#what-are-the-examples-of-conditional-annotations-how-are-they-used)

## How does Spring Boot know what to configure?

Spring Boot detects the dependencies available on the classpath and configures Spring beans accordingly. There are a number of annotations, examples are **@ConditionalOnClass, @ConditionalOnBean, @ConditionalOnMissingBean and @ConditionalOnMissingClass**, that allows for applying conditions to Spring configuration classes or Spring bean declaration methods in such classes. Examples:
- A Spring bean is to be created only if a certain dependency is available on the classpath. Use @ConditionalOnClass and supply a class contained in the dependency in question.
- A Spring bean is to be created only if there is no bean of a certain type or with a certain name created. Use @ConditionalOnMissingBean and specify name or type of bean to
check.

## What does @EnableAutoConfiguration do?

The @EnableAutoConfiguration annotation enables Spring Boot auto-configuration. As earlier, Spring Boot autoconfiguration attempts to create and configure Spring beans based on the dependencies available on the class-path to allow developers to quickly get started with different technologies in a Spring Boot application and reducing boilerplate code and configuration.

## What does @SpringBootApplication do?

- Turn on autoconfig(@EnableAutoConfiguratio)
- Enable auto-scanning(@ComponentScan)
- Defines a configuration class (@Configuration)

## Does Spring Boot do component scanning? Where does it look by default?

If it’s annotated with @SpringBootApplication. The @SpringBootApplication annotation is often placed on your main class, and it implicitly defines a base “search package” for certain items.

## How are DataSource and JdbcTemplate auto-configured?

DataSource configuration is controlled by external configuration properties in spring.datasource.*. For example, you might declare the following section in application.properties:

```properties
spring.datasource.url=jdbc:mysql://localhost/test 
spring.datasource.username=dbuser 
spring.datasource.password=dbpass 
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
```

Spring’s JdbcTemplate and NamedParameterJdbcTemplate classes are auto-configured, and you can `@Autowire` them directly into your own beans.

```java
@Component 
public class MyBean {

  private final JdbcTemplate jdbcTemplate;
  
  @Autowired 
  public MyBean(JdbcTemplate jdbcTemplate) { 
    this.jdbcTemplate = jdbcTemplate; 
  }
}
```

## What is spring.factories file for?

the `META-INF/spring.factories` defined all the auto-configuration classes that will be used to guess what kind of application you are running. It's the secret behind the auto-configuration.

You need to specify the class that will be picked up by the `EnableAutoConfiguration` class. This class imports the `EnableAutoConfigurationImportSelector` that will inspect the `spring.factories` and loads the class and executes the declaration.

Some events are actually triggered **before** the ApplicationContext is created, so you cannot register a listener on those as a `@Bean`. You can register them with the `SpringApplication.addListeners(…)` method or the `SpringApplicationBuilder.listeners(…)` method.

If you want those listeners to be registered automatically, regardless of the way the application is created, you can add a `META-INF/spring.factories` file to your project and reference your listener(s) by using the `org.springframework.context.ApplicationListener `key.

```properties
org.springframework.context.ApplicationListener=com.example.project.MyListener
```

Spring Boot checks for the presence of a `META-INF/spring.factories` file within your published jar. The file should list your configuration classes under the `EnableAutoConfiguration` key, as shown in the following example:

```properties
org.springframework.boot.autoconfigure.EnableAutoConfiguration=com.mycorp.libx.autoconfigure.LibXAutoConfiguration, com.mycorp.libx.autoconfigure.LibXWebAutoConfiguration
```

## How do you customize Spring Boot auto configuration?

configuration properties are nothing more than properties of beans that have been designated to accept configurations from Spring’s environment abstraction.

To support property injection of configuration properties, Spring Boot provides the @ConfigurationProperties annotation. When placed on any Spring bean, it specifies that the properties of that bean can be injected from properties in the Spring environment.

**Creating a Custom Auto-Configuration**

1. To create a custom auto-configuration, we need to create a class annotated as @Configuration and register it
    ```java
    @Configuration
    public class MySQLAutoconfiguration {}
    ```

2. The next mandatory step is registering the class as an auto-configuration candidate, by adding the name of the class under the key `org.springframework.boot.autoconfigure.EnableAutoConfiguration` in the standard file `resources/META-INF/spring.factories`:
    ```properties
    org.springframework.boot.autoconfigure.EnableAutoConfiguration=com.baeldung.autoconfiguration.MySQLAutoconfiguration
    ```
3. Auto-configuration is designed using classes and beans marked with @Conditional annotations so that the auto-configuration or specific parts of it can be replaced.

Note that the auto-configuration is only in effect if the auto-configured beans are not defined in the application. If you define your bean, then the default one will be overridden.

## What are the examples of @Conditional annotations? How are they used?

The Spring Boot **autoconfiguration** mechanism heavily depends on the `@Conditional` feature.
Using the @Conditional approach, you can register a bean conditionally based on any arbitrary condition.

For example, you may want to register a bean when:

- A specific class is present in the classpath
- A Spring bean of a certain type isn’t already registered in the ApplicationContext
- A specific file exists in a location
- A specific property value is configured in a configuration file
- A specific system property is present/absent

`@ConditionalOnBean`: Matches when the specified bean classes and/or names are already registered.

`@ConditionalOnMissingBean`: Matches when the specified bean classes and/or names are not already registered.

`@ConditionalOnClass`: Matches when the specified classes are on the classpath.

`@ConditionalOnMissingClass`:  Matches when the specified classes are not on the classpath.

`@ConditionalOnProperty`: Matches when the specified properties have a specific value.

`@ConditionalOnResource`: Matches when the specified resources are on the classpath.

`@ConditionalOnWebApplication`: Matches when the application context is a web application context.

## References

1. [MrR0807 Spring certification notes](https://github.com/MrR0807/SpringCertification5.0)
2. [Moss Green Spring certification notes](https://mossgreen.github.io/)
3. [Spring Documentation](https://docs.spring.io/spring-framework/docs/5.3.8/reference/html/)
4. [Spring Boot Documentation](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/)
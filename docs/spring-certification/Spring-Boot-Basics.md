---

title: Spring Boot Basics

---

## What is Spring Boot?

Spring Boot is a set of preconfigured modules that works on top of Spring Framework and simplifies configuring a Spring application.
Some of the more central modules are:
- Spring-boot-dependencies. Contains versions of dependencies
- Spring-boot-starter-parent. Parent pom.xml
- Spring-boot-starters. Parent for all the Spring-Boot starter modules
- Spring-boot-autoconfigure. Contains the autoconfigure modules for the starters.
- Spring-boot-actuator. Allows for monitoring and managing of application created

## What are the advantages of using Spring Boot?

- Automatic configuration of “sensible defaults” reducing boilerplate configuration
- Allows for easy customization when the defaults no longer suffices
- A Spring Boot project can produce an executable stand-alone JAR-file
- Provides a set of managed dependencies that have been verified to work together
- Integrated web containers that allow for easy testing

## What things affect what Spring Boot sets up?

There are a number of condition annotations in Spring Boot each of which can be used to control the creation of Spring beans. The following is a list of the condition annotations in Spring Boot (**there are more**):

| **Conditional Annotation** | **Condition Factor**  |
| ------- | --- |
| @ConditionalOnClass | Presence of class on classpath |
| @ConditionalOnMissingClass | Absence of class on classpath |
| @ConditionalOnBean | Presence of Spring bean or bean type (class) |
| @ConditionalOnMissingBean | Absence of Spring bean or bean type (class) |
| @ConditionalOnProperty | Presence of Spring environment property |
| @ConditionalOnResource | Presence of resource such as file |
| @ConditionalOnWebApplication | If the application is considered to be a web application, that is uses the Spring WebApplicationContext |
| @ConditionalOnNotWebApplication | If the application is not considered to be a web application |


## What is a Spring Boot starter? Why is it useful?

Spring Boot Starters are a set of convenient dependency descriptors that you can include in your application. You get a one-stop-shop for all the Spring and related technology that you need without having to hunt through sample code and copy paste loads of dependency descriptors. For example, if you want to get started using Spring and JPA for database access just include the spring-boot-starter-data-jpa dependency in your project, and you are good to go.
The starters contain a lot of the dependencies that you need to get a project up and running quickly and with a consistent, supported set of managed transitive dependencies.

## Spring Boot supports both properties and YML files. Would you recognize and understand them if you saw them?

Java properties files come in application.properties file; YAML come in application.yml. YML:

```yaml
server:
  port: 8000
---
spring:
  profiles: default
  security:
    user:
    password: weak
```

```properties
environments.dev.url=http://dev.example.com
environments.dev.name=Developer Setup
environments.prod.url=http://another.example.com
environments.prod.name=My Cool App
```

## Can you control logging with Spring Boot? How?

### Controlling Log levels

As per default, messages written with the ERROR, WARN and INFO levels will be output in a Spring Boot application. To enable DEBUG or TRACE logging for the entire application, use the --debug or --trace flags or set the properties debug=true or trace=true in the application.properties file.
Log levels can be controlled at a finer granularity using the application.properties file:

```properties
logging.level.root=WARN
logging.level.org.springframework.web=DEBUG
logging.level.org.hibernate=ERROR
```

### File Output

By default, Spring Boot logs only to the console and does not write log files. If you want to write log files in addition to the console output, you need to set a logging.file or logging.path property (for example, in your application.properties).

### Color-coding of Log levels

In consoles that support ANSI, messages of different log levels can be color-coded to improve readability. This can be accomplished as shown in the following example:

```properties
logging.pattern.console=%clr(%d{yyyy-MM-dd HH:mm:ss}) {yellow}
```

Suppose that you want to write the log entries to a file named BookWorm.log at /var/ logs/. The logging.path and logging.file properties can help with that:

```yaml
logging:
  path: /var/logs/
  file: BookWorm.log
  level:
    root: WARN
    org:
      springframework:
        security: DEBUG
```

## Where does Spring Boot look for application.properties file by default?

src/main/resources

## How do you define profile specific property files?

These have to be named in the format application-{profile}.properties.
For example: application-dev.properties and application-production.properties:

## How do you access the properties defined in the property files?

@Value annotation or

```java
@Autowired
private Environment env;

public void getRootLogLevel() {
    return env.getProperty("logging.level.root")
}
```


or @ConfigurationProperties

## What properties do you have to define in order to configure external MySQL?

1. dependencies: Spring boot,and msql
2. config properties

In your `application.properties` file, add:
```properties
spring.jpa.hibernate.ddl-auto=none
spring.datasource.url=jdbc:mysql://<dbhost>:<dbport>/<db>
spring.datasource.username=<username>
spring.datasource.password=<password>
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
```

## How do you configure default schema and initial data?

Spring Boot uses the `spring.datasource.initialize` property value, which is true by default, to determine whether to initialize the database. If it's set to `true`, Spring Boot will use the `schema.sql` and `data.sql` files in the root classpath to initialize the database.
- `schema.sql` to initialize the schema (tables, views, etc.)
- and a `data.sql` to insert data into the tables.

Spring Boot will load the `schema-${platform}.sql` and `data-${platform}.sql` files if they are available in the root classpath to do database initialization. The value for <platform> is read from the `spring.datasource.platform` property. This allows you to switch to database-specific scripts if necessary.

## What is a fat jar? How is it different from original jar?

**Executable jars**, known as “fat jars”, are archives containing your compiled classes along with all of the jar dependencies that your code needs to run.

In your project target directory, you should see `myproject-0.0.1-SNAPSHOT.jar`. This the far Jar. Inside of it, you would see `myproject-0.0.1-SNAPSHOT.jar.original`. This is the **original jar** file that Maven created before it was repackaged by Spring Boot.

**Spring Boot Executable jars VS uber jars**

1. An **uber jar** packages all the classes from all the application’s dependencies **into one single archive**. The problem with this approach is that
    - it becomes hard to see which libraries are in your application.
    - It can also be problematic if the same filename is used (but with different content) in multiple jars.

2. Spring Boot lets you actually nest jars directly.
    - you can run your application as you would any other
    - You do not need any special IDE plugins or extensions.
    - Executable jars can be used for production deployment. As they are self-contained, they are also ideally suited for cloud-based deployment.
    - Spring Boot’s executable jars are ready-made for most popular cloud PaaS (Platform-as-a-Service) providers. These providers tend to require that you “bring your own container”. They manage application processes (not Java applications specifically), so they need an intermediary layer that adapts your application to the cloud’s notion of a running process.

**To create an executable jar**, we need to have the `spring-boot-maven-plugin` to our `pom.xml`.

```xml
<plugin>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-maven-plugin</artifactId>
</plugin>
```

**Running as a Packaged Application**

1. `java -jar`
    ```bash
    $ java -jar target/myapplication-0.0.1-SNAPSHOT.jar
    ```
2. Maven Plugin
    ```bash
    $ mvn spring-boot:run
    ```
3. Gradle Plugin
    ```bash
    $ gradle bootRun
    ```

## What embedded containers does Spring Boot support?

Spring Boot includes support for embedded Tomcat, Jetty, and Undertow servers.
---

title: Spring Boot Actuator

---

* [What value does Spring Boot Actuator provide?](#what-value-does-spring-boot-actuator-provide)
* [What are the two protocols you can use to access actuator endpoints?](#what-are-the-two-protocols-you-can-use-to-access-actuator-endpoints)
* [What are the actuator endpoints that are provided out of the box?](#what-are-the-actuator-endpoints-that-are-provided-out-of-the-box)
* [What is info endpoint for? How do you supply data?](#what-is-info-endpoint-for-how-do-you-supply-data)
* [How do you change logging level of a package using loggers endpoint?](#how-do-you-change-logging-level-of-a-package-using-loggers-endpoint)
* [How do you access an endpoint using a tag?](#how-do-you-access-an-endpoint-using-a-tag)
* [What is metrics for?](#what-is-metrics-for)
* [How do you create a custom metric?](#how-do-you-create-a-custom-metric)
* [What is Health Indicator?](#what-is-health-indicator)
* [What are the Health Indicators that are provided out of the box?](#what-are-the-health-indicators-that-are-provided-out-of-the-box)
* [What is the Health Indicator status?](#what-is-the-health-indicator-status)
* [What are the Health Indicator statuses that are provided out of the box?](#what-are-the-health-indicator-statuses-that-are-provided-out-of-the-box)
* [How do you change the Health Indicator status severity order?](#how-do-you-change-the-health-indicator-status-severity-order)
* [Why do you want to leverage 3rd-party external monitoring system?](#why-do-you-want-to-leverage-3rd-party-external-monitoring-system)

## What value does Spring Boot Actuator provide?

The Spring Boot Actuator module provides production-ready features such as monitoring, metrics, health checks, etc. The Spring Boot Actuator enables you to monitor the application using HTTP endpoints and JMX.
Spring Boot provides spring-boot-starter-actuator to autoconfigure Actuator.

## What are the two protocols you can use to access actuator endpoints?

HTTP and JMX

## What are the actuator endpoints that are provided out of the box?

1. The `/actuator` endpoint will provide a hypermedia-based discovery page for all the other endpoints, but it will require the Spring HATEOAS in the classpath.

2. `/autoconfig` This endpoint will display the auto-configuration report. It will give you two groups: positiveMatches and negativeMatches.

3. `/beans` This endpoint will display all the Spring beans that are used in your application. R

4. `/configprops` This endpoint will list all the configuration properties that are defined by the @ConfigurationProperties beans,

5. `/docs` This endpoint will show HTML pages with all the documentation for all the Actuator module endpoints. This endpoint can be activated by including the `spring-boot-actuator-docs` dependency in pom.xml

6. `/dump` This endpoint will perform a thread dump of your application.

7. `/env` This endpoint will expose all the properties from the Spring’s ConfigurableEnvironment interface. This will show any active profiles and system environment variables and all application properties, including the Spring Boot properties.

8. `/flyway` This endpoint will provide all the information about your database migration scripts; it’s based on the Flyway project (https://flywaydb.org/). This is very useful when you want to have full control of your database by versioning your schemas. I

9. `/health` This endpoint will show the health of the application. If you are doing a database app like in the previous section (/flyway) you will see the DB status and by default you will see also the diskSpace from your system.

10. `/info` This endpoint will display the public application info. This means that you need to add this information to application.properties. It’s recommended that you add it if you have multiple Spring Boot applications.

11. `/logfile` This endpoint will show the contents of the log file specified by the logging.file property, where you specify the name of the log file (this will be written in the current directory). You can also set the logging.path, where you set the path where the spring.log will be written. By default, Spring Boot writes to the console/standard out, and if you specify any of these properties, it will also write everything from the console to the log file. You can stop your application. Go to src/main/resources/application.properties and add this to the very end:

    ```properties
    logging.file=mylog.log
    ```
12. `/metrics` This endpoint shows the metrics information of the current application, where you can determine the how much memory it’s using, how much memory is free, the uptime of your application, the size of the heap is being used, the number of threads used, and so on.

13. `/mappings` This endpoint shows all the lists of all @RequestMapping paths declared in your application. This is very useful if you want to know more about what mappings are declared.

14. `/shutdown` This endpoint is not enabled by default.

The only endpoints that are **not sensitive** are `/docs`, `/info` and `/health`. So, if you want to disable the sensitive feature, you can configure them in the `application.properties` file.

## What is info endpoint for? How do you supply data?

If you added any information about the application in the `application.properties` file using the `info.app.*` properties, then you can view it at the `http://localhost:8080/application/info` endpoint.

```properties
info.app.name=Spring Boot Web Actuator Application 
info.app.description=This is an example of the Actuator module 
info.app.version=1.0.0
```

f you want to add the build version to the info endpoint, then the application.properties won’t cut it. Also, changing the application.properties on each build takes effort. Spring boot handles this by loading data from META-INF/build-info.properties file if it is present in the JAR file.
To create the build-info.properties file, you should first change the spring-boot-maven-plugin settings as shown below.

```xml
<plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
    <executions>
        <execution>
            <goals>
                <goal>build-info</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

Importantly, You can provide your InfoContributor bean and it will be picked by spring boot.

## How do you change logging level of a package using loggers endpoint?

Spring Boot Actuator includes the ability to **view and configure** the log levels of your application at runtime.

You can view either the entire list or an individual logger’s configuration, which is made up of both the explicitly configured logging level and the effective logging level given to it by the logging framework.

- TRACE
- DEBUG
- INFO
- WARN
- ERROR
- FATAL
- OFF
- null,  indicates that there is no explicit configuration.

To configure a given logger, POST a partial entity to the resource’s URI, as shown in the following example:
```json
{ 
  "configuredLevel": "DEBUG"
}
```

To “reset” the specific level of the logger (and use the default configuration instead), you can pass a value of null as the configuredLevel.

you can use the `/logfile` endpoint to view the log file content. Go to `http://localhost:8080/application/logfile`.

## How do you access an endpoint using a tag?

The `/metrics` endpoint is capable of reporting all manner of metrics produced by a running application, including metrics concerning memory, processor, garbage collection, and HTTP requests.

There are so many metrics covered that it would be impossible to monitor them all. You can narrow down the results further by using the tags listed under availableTags.

For example, you know that there have been 2,103 requests, but what’s unknown is how many of them resulted in an HTTP 200 versus an HTTP 404 or HTTP 500 response status. **Using the status tag**, you can get metrics for all requests resulting in an HTTP 404 status like this:
```bash
$ curl localhost:8081/actuator/metrics/http.server.requests?tag=status:404 
```
```json
{ 
"name": "http.server.requests", 
"measurements": [ 
  { "statistic": "COUNT", "value": 31 }, 
  { "statistic": "TOTAL_TIME", "value": 0.522061212 }, 
  { "statistic": "MAX", "value": 0 } 
], 
"availableTags": [ 
  { "tag": "exception", "values": [ "ResponseStatusException", "none" ] }, 
  { "tag": "method", "values": [ "GET" ] }, 
  { "tag": "uri", "values": [ "/actuator/metrics/{requiredMetricName}", "/**" ] } 
]
}
```

Add any number of `tag=KEY:VALUE` query parameters to the end of the URL to dimensionally drill down on a meter. By specifying the tag name and value with the `tag` request attribute, you now see metrics specifically for requests that resulted in an HTTP 404 response.

To know how many of those HTTP 404 responses were for the /** path?
All you need to do to filter this further is to specify the uri tag in the request, like this:
```bash
$ curl "localhost:8081/actuator/metrics/http.server.requests?tag=status:404&tag=uri:/**"
```
```json
{ 
"name": "http.server.requests", 
"measurements": [ 
  { "statistic": "COUNT", "value": 30 }, 
  { "statistic": "TOTAL_TIME", "value": 0.519791548 }, 
  { "statistic": "MAX", "value": 0 } ], 
"availableTags": [ 
  { "tag": "exception", "values": [ "ResponseStatusException" ] }, 
  { "tag": "method", "values": [ "GET" ] } 
]
}
```

As you refine the request, the available tags are more limited. The tags offered are only those that match the requests captured by the displayed metrics.

**Common tags**
Common tags are generally used for dimensional **drill-down on the operating environment** like host, instance, region, stack, etc. Commons tags are applied to all meters and can be configured as shown in the following example.
```properties
management.metrics.tags.region=us-east-1 
management.metrics.tags.stack=prod
```

## What is metrics for?

This endpoint shows the metrics information of the current application, where you can determine the how much memory it’s using, how much memory is free, the uptime of your application, the size of the heap is being used, the number of threads used, and so on.

One of the important features about this endpoint is that it has some counters and gauges that you can use, even for statistics about how many times your app is being visited or if you have the log file enabled. If you are accessing the /logfile endpoint, you will find some counters like counter.status.304.logfile, which indicates that the /logfile endpoint was accessed but hasn’t change. And of course you can have custom counters.

## How do you create a custom metric?

To register custom metrics, inject MeterRegistry into your component, as shown in the following example:

```java
@Component
public class MyBean {
    private final Dictionary dictionary;
    
    public MyBean(MeterRegistry registry) {
        this.dictionary = Dictionary.load();
        registry.gauge("dictionary.size", Tags.empty(), this.dictionary.getWords().size());
    }
}
```

If your metrics depend on other beans, it is recommended that you use a MeterBinder to register them, as shown in the following example:

```java
public class MyMeterBinderConfiguration {
    @Bean
    public MeterBinder queueSize(Queue queue) {
        return (registry) -> Gauge.builder("queueSize", queue::size).register(registry);
    }
}
```

Using a MeterBinder ensures that the correct dependency relationships are set up and that the bean is available when the metric’s value is retrieved. A MeterBinder implementation can also be useful if you find that you repeatedly instrument a suite of metrics across components or applications.

## What is Health Indicator?

This endpoint will show the health of the application. If you are doing a database app like in the previous section (/flyway) you will see the DB status and by default you will see also the diskSpace from your system. If you are running your app, you can go to `http://localhost:8080/health`.

## What are the Health Indicators that are provided out of the box?

| Health Indicator | Key  | Reports|
| ------- | --- | ---|
|`ApplicationHealthIndicator` | none | Always "UP"|
|`DataSourceHealthIndicator` | db | "UP" and database type if the database can be contacted; "DOWN" status otherwise|
|`DiskSpaceHealthIndicator` | diskSpace |  "UP" and JMS provider name if the message broker can be contacted; "DOWN" otherwise|
|`JmsHealthIndicator` | jms | "UP" and JMS provider name if the message broker can be contacted; "DOWN" otherwise|
|`MailHealthIndicator` | mail | "UP" and the mail server host and port if the mail server can be contacted; "DOWN" otherwise|
|`MongoHealthIndicator` | mongo | "UP" and the MongoDB server version; "DOWN" otherwise |
|`RabbitHealthIndicator` | rabbit | "UP" and the RabbitMQ broker version; "DOWN" otherwise|
|`RedisHealthIndicator` | redis | "UP" and the Redis server version; "DOWN" otherwise |
|`SolrHealthIndicator` | solr | "UP" if the Solr server can be contacted; "DOWN" otherwise |

## What is the Health Indicator status?

It describes health of the system and maps to  HTTP status code.

## What are the Health Indicator statuses that are provided out of the box?

- `OUT_OF_SERVICE` SERVICE_UNAVAILABLE (503)

- `UP` No mapping by default, so http status is 200

- `UNKNOWN` No mapping by default, so http status is 200

## How do you change the Health Indicator status severity order?

- Assume a new Status with code `FATAL` is being used in one of your HealthIndicator implementations.
- You might also want to register custom status mappings if you access the health endpoint over HTTP.

```properties
management.health.status.order=FATAL, DOWN, OUT_OF_SERVICE, UNKNOWN, UP
management.health.status.http-mapping.FATAL=503
```

## Why do you want to leverage 3rd-party external monitoring system?

Spring Boot auto-configures a composite `MeterRegistry` and adds a registry to the composite for each of the supported implementations that it finds on the classpath.

Having a dependency on `micrometer-registry-{system}` in your runtime classpath is enough for Spring Boot to configure the registry.

## References

1. [MrR0807 Spring certification notes](https://github.com/MrR0807/SpringCertification5.0)
2. [Moss Green Spring certification notes](https://mossgreen.github.io/)
3. [Spring Documentation](https://docs.spring.io/spring-framework/docs/5.3.8/reference/html/)
4. [Spring Boot Documentation](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/)
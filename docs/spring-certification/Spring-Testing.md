---

title: Spring Testing

---

* [What type of tests typically use Spring?](#what-type-of-tests-typically-use-spring)
* [How can you create a shared application context in a JUnit integration test?](#how-can-you-create-a-shared-application-context-in-a-junit-integration-test)
* [When and where do you use @Transactional in testing?](#when-and-where-do-you-use-transactional-in-testing)
* [How are mock frameworks such as Mockito or EasyMock used?](#how-are-mock-frameworks-such-as-mockito-or-easymock-used)
* [How is @ContextConfiguration used?](#how-is-contextconfiguration-used)
* [How does Spring Boot simplify writing tests?](#how-does-spring-boot-simplify-writing-tests)
* [What does @SpringBootTest do? How does it interact with @SpringBootApplication and @SpringBootConfiguration?](#what-does-springboottest-do-how-does-it-interact-with-springbootapplication-and-springbootconfiguration)
  * [How does it interact with @SpringBootApplication and @SpringBootConfiguration?](#how-does-it-interact-with-springbootapplication-and-springbootconfiguration)


## What type of tests typically use Spring?

The Spring Framework provides first-class support for integration testing in the spring-test module.

## How can you create a shared application context in a JUnit integration test?

The Spring TestContext Framework provides consistent loading of Spring ApplicationContext instances and WebApplicationContext instances as well as caching of those contexts. Support for the caching of loaded contexts is important, because startup time can become an issue.By default, once loaded, the configured ApplicationContext is reused for each test. Thus, the setup cost is incurred only once per test suite, and subsequent test execution is much faster. 
To create shared application context you have to annotate a JUnit 4 based test class with @RunWith(SpringJUnit4ClassRunner.class) or @RunWith(SpringRunner.class). 
However, @RunWith will try to initialize default @ContextConfiguration:
- Instantiating TestContextBootstrapper for test class DemoApplicationTests from class org.springframework.test.context.support.DefaultTestContextBoots trapper;
- Neither @ContextConfiguration nor @ContextHierarchy found for test class DemoApplicationTests, using DelegatingSmartContextLoader
- org.springframework.test.context.support.AbstractDelegatingSmartContextLoader - Delegating to GenericXmlContextLoader to process context configuration declaringClass = 'DemoApplicationTests', contextLoaderClass = 'org.springframework.test.context.Context Loader']. 
- org.springframework.test.context.support.AbstractContextLoader - Did not detect default resource location for test class DemoApplicationTests: class path resource [com/qualifiertest/demo/DemoApplicationTests-context.xml] does not exist;
- org.springframework.test.context.support.AbstractContextLoader - Could not detect default resource locations for test class DemoApplicationTests: no resource found for suffixes {-context.xml}. 
- org.springframework.test.context.support.AbstractDelegatingSmartContextLoader - Delegating to AnnotationConfigContextLoader to process context configuration declaringClass = 'DemoApplicationTests', contextLoaderClass = 'org.springframework.test.context.ContextLoader' . 
- org.springframework.test.context.support.AnnotationConfigContextLoaderUtils - Could not detect default configuration classes for test class DemoApplicationTests : DemoApplicationTests does not declare any static, non-private, non-final, nested classes annotated with @Configuration.
If it does find – it will fail. That is why you need to use @ContextConfiguration to point @Configuration classes.

## When and where do you use @Transactional in testing?
Annotating a test method with @Transactional causes the test to be run within a transaction that is, by default, automatically rolled back after completion of the test. If a test class is annotated with @Transactional, each test method within that class hierarchy runs within a transaction. Test methods that are not annotated with @Transactional (at the class or method level) are not run within a transaction. Furthermore, tests that are annotated with @Transactional but have the propagation type set to NOT_SUPPORTED are not run within a transaction.Also, you can create custom annotations to include more than one spring test annotation:

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
@Transactional
@Tag("integration-test") // org.junit.jupiter.api.Tag
@Test // org.junit.jupiter.api.Test
public @interface TransactionalIntegrationTest {}
```
## How are mock frameworks such as Mockito or EasyMock used?

Mockito lets you write tests by mocking the external dependencies with the desired behavior. Mock objects have the **advantage over stubs** in that they are created dynamically and only for the specific scenario tested.

Steps of using Mockito:

1. Declare and create the mock
2. Inject the mock
3. Define the behavior of the mock
4. Test
5. Validate the execution

```java
public class SimpleReviewServiceTest {

  private ReviewRepo reviewMockRepo = mock(ReviewRepo.class); // (1)
  private SimpleReviewService simpleReviewService;
  
  @Before
  public void setUp(){
    simpleReviewService = new SimpleReviewService();
    simpleReviewService.setRepo(reviewMockRepo); //(2)
  }
  
  @Test
  public void findByUserPositive() {
    User user = new User();
    Set<Review> reviewSet = new HashSet<>();
    when(reviewMockRepo.findAllForUser(user)).thenReturn(reviewSet);// (3)
    Set<Review> result = simpleReviewService.findAllByUser(user); // (4)
    assertEquals(result.size(), 1); //(5)
  }
}
```

Mockito with Annotations

- `@Mock` : Creates mock instance of the field it annotates

- `@InjectMocks` has a behavior similar to the Spring IoC, because its role is to instantiate testing object instances and to try to inject fields annotated with `@Mock` or `@Spy` into private fields of the testing object.

- Use Mockito
    - Either: `@RunWith(MockitoJUnitRunner.class)` to initialize the mock objects.
    - OR: `MockitoAnnotations.initMocks(this)` in the JUnit `@Before` method.

```java
public class MockPetServiceTest {

  @InjectMocks
  SimplePetService simplePetService;
  
  @Mock
  PetRepo petRepo;
  
  @Before
  public void initMocks() {
    MockitoAnnotations.initMocks(this);
  }
  
  @Test p
  ublic void findByOwnerPositive() {
    Set<Pet> sample = new HashSet<>();
    sample.add(new Pet());
    Mockito
      .when(petRepo.findAllByOwner(owner))
      .thenReturn(sample);

    Set<Pet> result = simplePetService.findAllByOwner(owner);

    assertEquals(result.size(), 1); }
  }
}
```

Mockito in Spring Boot

`@MockBean`

- It is a Spring Boot annotation,
- used to define a new Mockito mock bean or replace a Spring bean with a mock bean and inject that into their dependent beans.
- The annotation can be used directly on test classes, on fields within your test, or on @Configuration classes and fields.
- When used on a field, the instance of the created mock is also injected
- Mock beans will be automatically reset after each test method.
- If your test uses one of Spring Boot’s test annotations (such as`@SpringBootTest`), this feature is automatically enabled.

## How is @ContextConfiguration used?

@ContextConfiguration defines class-level metadata that is used to determine how to load and configure an ApplicationContext for integration tests. Specifically, @ContextConfiguration declares the application context resource locations or the annotated classes used to load the context.
Resource locations are typically XML configuration files or Groovy scripts located in the classpath, while annotated classes are typically @Configuration classes. However, resource locations can also refer to files and scripts in the file system, and annotated classes can be component classes, and so on.

```java
@ContextConfiguration("/test-config.xml")
public class XmlApplicationContextTests {
    // class body...
}
```

```java
@ContextConfiguration(classes = TestConfig.class)
public class ConfigClassApplicationContextTest {
    // class body...
}
```

As an alternative or in addition to declaring resource locations or annotated classes, you can use @ContextConfiguration to declare ApplicationContextInitializer classes. The following example shows such a case:

```java 
@ContextConfiguration(initializers = CustomContextIntializer.class)
public class ContextInitializaerTests {
    // class body...
}
```

You can optionally use @ContextConfiguration to declare the ContextLoader strategy as well. Note, however, that you typically do not need to explicitly configure the loader, since the default loader supports initializers and either resource locations or annotated classes.

```java 
@ContextConfiguration(locations = "/test-context.xml", loader = CustomContextLoader.class)
public class CustomLoaderXmlApplicationContextTests {
    // class body...
}
```

## How does Spring Boot simplify writing tests?

Spring Boot provides a number of utilities and annotations to help when testing your application. Test support is provided by two modules: spring-boot-test contains core items, and spring-boot-test-autoconfigure supports auto-configuration for tests.Most developers use the spring-boot-starter-test “Starter”, which imports both Spring Boot test modules as well as JUnit, AssertJ, Hamcrest, and a number of other useful libraries.
Spring Boot provides a @SpringBootTest annotation, which can be used as an alternative to the standard spring-test @ContextConfiguration annotation when you need Spring Boot features. The annotation works by creating the ApplicationContext used in your tests through SpringApplication.
When testing Spring Boot applications, this is often not required. Spring Boot’s @*Test annotations search for your primary configuration automatically whenever you do not explicitly define one.The search algorithm works up from the package that contains the test until it finds a class annotated with @SpringBootApplication or @SpringBootConfiguration.
By default, @SpringBootTest will not start a server. You can use the webEnvironment attribute of @SpringBootTest to further refine how your tests run:
- MOCK (default)
- RANDOM_PORT: Loads WebServerApplicationContext
- DEFINED_PORT: Loads WebServerApplicationContext
- NONE: Loads and ApplicationContext by using SpringApplication but does not provide any web environment (mock or otherwise).

Additional annotations:
- @AutoConfigureWebTestClient
- @AutoConfigureMockMvc
- @JsonTest
- @WebMvcTest: @WebMvcTest auto-configures the Spring MVC infrastructure and limits scanned beans to @Controller, @ControllerAdvice, @JsonComponent, Converter, GenericConverter, Filter, WebMvcConfigurer, and HandlerMethodArgumentResolver. Regular @Component beans are not scanned when using this annotation.
- @DataJpaTest: By default, data JPA tests are transactional and roll back at the end of each test.
- @JdbcTest

## What does @SpringBootTest do? How does it interact with @SpringBootApplication and @SpringBootConfiguration?

Annotation that can be specified on a test class that runs Spring Boot based tests. Provides the following features over and above the regular Spring TestContext Framework: 
- Uses SpringBootContextLoader as the default ContextLoader when no specific @ContextConfiguration(loader=...) is defined.
- Automatically searches for a @SpringBootConfiguration when nested @Configuration is not used, and no explicit classes are specified. 
- Allows custom Environment properties to be defined using the properties attribute.
- Provides support for different webEnvironment modes, including the ability to start a fully running web server listening on a defined or random port.
- Register a TestRestTemplate and/or WebTestClient bean for use in web tests that are using a fully running web server. 

### How does it interact with @SpringBootApplication and @SpringBootConfiguration?
When testing Spring Boot applications, this is often not required. Spring Boot’s @*Test annotations search for your primary configuration automatically whenever you do not explicitly define one.The search algorithm works up from the package that contains the test until it finds a class annotated with @SpringBootApplication or @SpringBootConfiguration. As long as you structured your code in a sensible way, your main configuration is usually found. 

## References

1. [MrR0807 Spring certification notes](https://github.com/MrR0807/SpringCertification5.0)
2. [Moss Green Spring certification notes](https://mossgreen.github.io/)
3. [Spring Documentation](https://docs.spring.io/spring-framework/docs/5.3.8/reference/html/)
4. [Spring Boot Documentation](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/)
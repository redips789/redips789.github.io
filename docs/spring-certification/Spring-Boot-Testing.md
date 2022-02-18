---

title: Spring Boot Testing

---

* [When do you want to use @SpringBootTest annotation?](#when-do-you-want-to-use-springboottest-annotation)
* [What does @SpringBootTest auto-configure?](#what-does-springboottest-auto-configure)
* [What dependencies does spring-boot-starter-test brings to the classpath?](#what-dependencies-does-spring-boot-starter-test-brings-to-the-classpath)
* [How do you perform integration testing with @SpringBootTest for a web application?](#how-do-you-perform-integration-testing-with-springboottest-for-a-web-application)
* [When do you want to use @WebMvcTest? What does it auto-configure?](#when-do-you-want-to-use-webmvctest-what-does-it-auto-configure)
* [What are the differences between @MockBean and @Mock?](#what-are-the-differences-between-mockbean-and-mock)
* [When do you want @DataJpaTest for? What does it auto-configure?](#when-do-you-want-datajpatest-for-what-does-it-auto-configure)

## When do you want to use @SpringBootTest annotation?

the test context framework will be searching for the class annotated with `@SpringBootApplication` (if no specific configuration is passed) and will use that to actually start the application.

```java
@RunWith(SpringRunner.class) 
@SpringBootTest(classes = CalculatorApplication.class) 
public class CalculatorApplicationTests {

  @Autowired private Calculator calculator;

  @Test(expected = IllegalArgumentException.class) 
  public void doingDivisionShouldFail() { 
    calculator.calculate(12,13, '/'); 
  }
}
```

## What does @SpringBootTest auto-configure?

Spring boot provides the `@SpringBootTest` annotation to configure the `ApplicationContext` for tests that use SpringApplication behind the scenes so that **all the Spring Boot features will be available**.

For `@SpringBootTest`, you can pass
- Spring configuration classes,
- Spring bean definition XML files,
- and more,

In Spring Boot applications, you’ll typically use the entry point class.


## What dependencies does spring-boot-starter-test brings to the classpath?

The Spring Boot Test starter `spring-boot-starter-test` pulls in
- Spring Test
- Spring Boot Test modules
- JSONPath,
- JUnit,
- AssertJ,
- Mockito,
- Hamcrest,
- JSONassert

## How do you perform integration testing with @SpringBootTest for a web application?

To properly test a web application, you need a way to throw actual HTTP requests at it and assert that it processes those requests correctly. Two options:

1. **Spring Mock MVC** — Enables controllers to be tested in a mocked approximation of a servlet container without actually starting an application server. To set up a Mock MVC in your test, you can use MockMvcBuilders.
    - `standaloneSetup()` — Builds a Mock MVC to serve one or more manually created and configured controllers. It expects you to manually instantiate and inject the controllers you want to test, whereas webAppContextSetup() works from an instance of WebApplicationContext, which itself was probably loaded by Spring. The former is slightly more akin to a unit test in that you’ll likely only use it for very focused tests around a single controller.
    - `webAppContextSetup()` — Builds a Mock MVC using a Spring application context, which presumably includes one or more configured controllers. lets Spring load your controllers as well as their dependencies for a full-blown integration test.
```java
@RunWith(SpringJUnit4ClassRunner.class) 
@SpringApplicationConfiguration( classes = ReadingListApplication.class) 
@WebAppConfiguration public class MockMvcWebTests {
  @Autowired 
  private WebApplicationContext webContext;

  private MockMvc mockMvc;

  @Before 
  public void setupMockMvc() {

    mockMvc = MockMvcBuilders
      .webAppContextSetup(webContext)
      .build(); 
  }
  
  @Test 
  public void homePage() throws Exception { 
  mockMvc.perform(MockMvcRequestBuilders.get("/readingList")) 
    .andExpect(MockMvcResultMatchers.status().isOk()) 
    .andExpect(MockMvcResultMatchers.view().name("readingList")) 
    .andExpect(MockMvcResultMatchers.model().attributeExists("books")) 
    .andExpect(MockMvcResultMatchers.model().attribute("books", Matchers.is(Matchers.empty()))); 
  }
}
```

2. **Web integration tests** — Actually starts the application in an embedded servlet container (such as Tomcat or Jetty), enabling tests that exercise the application in a real application server.  
   uses `@WebIntegrationTest` to start the application along with a server and uses Spring’s RestTemplate to perform HTTP requests against the application.
```java
@RunWith(SpringJUnit4ClassRunner.class) 
@SpringApplicationConfiguration( classes=ReadingListApplication.class) 
@WebIntegrationTest 
public class SimpleWebTest {

  @Test(expected=HttpClientErrorException.class) 
  public void pageNotFound() {
  
    try { 
      RestTemplate rest = new RestTemplate(); 
      rest.getForObject( "http://localhost:8080/bogusPage", String.class); 
      fail("Should result in HTTP 404"); 
    } catch (HttpClientErrorException e) { 
      assertEquals(HttpStatus.NOT_FOUND, e.getStatusCode()); 
      throw e; 
    }
  }
}
```

## When do you want to use @WebMvcTest? What does it auto-configure?

Spring Boot provides the `@WebMvcTest` annotation, which will autoconfigure SpringMVC infrastructure components and **load only**
- `@Controller`,
- `@ControllerAdvice`,
- `@JsonComponent`,
- `Filter`,
- `WebMvcConfigurer`, and
- `HandlerMethodArgumentResolver` components.

Other Spring beans (annotated with @Component, @Service, @Repository, etc.) will not be scanned when using this annotation.

In contrast to `@SpringBootTest`, which loads the **entire configuration**.

```java
@RunWith(SpringRunner.class) 
@WebMvcTest(controllers= TodoController.class) 
public class TodoControllerTests {

  @Autowired 
  private MockMvc mvc;
  
  @MockBean 
  private TodoRepository todoRepository;
  
  @Test 
  public void testShowAllTodos() throws Exception {
  
    Todo todo1 = new Todo(1, "Todo1",false); 
    Todo todo2 = new Todo(2, "Todo2",true);
  
    given(this.todoRepository.findAll())
      .willReturn(Arrays.asList(todo1, todo2));
  
    this.mvc.perform(get("/todolist") 
      .accept(MediaType.TEXT_HTML)) 
      .andExpect(status().isOk()) 
      .andExpect(view().name("todos")) 
      .andExpect(model().attribute("todos", hasSize(2))) ;
  
      verify(todoRepository, times(1)).findAll();
  }
}
```

You have annotated the test with `@WebMvcTest(controllers = TodoController.class)` by explicitly specifying which controller you are testing. As `@WebMvcTest` doesn’t load other regular Spring beans and TodoController depends on TodoRepository, **you provided a mock bean using the `@MockBean` annotation**. The `@WebMvcTest` autoconfigures MockMvc, which can be used to test controllers without starting an actual servlet container.

## What are the differences between @MockBean and @Mock?

**`@Mock`**
`@Mock` = `Mockito.mock()`. It's a from Mockito library.

**`@MockBean`**

- This is indeed a Spring Boot class.
- It allows to add Mockito mocks in a Spring ApplicationContext.
- If a bean, compatible with the declared class exists in the context, it replaces it by the mock.
- If it is not the case, it adds the mock in the context as a bean.

## When do you want @DataJpaTest for? What does it auto-configure?

`@DataJpaTest` and `@JdbcTest` annotations to test the Spring beans, which talk to relational databases.

- The `@DataJpaTest` allows you to test the persistence layer components, **doesn’t load other Spring beans** (@Components, @Controller, @Service, and annotated beans) into ApplicationContext.

- Enable transactions by applying Spring's `@Transactional` annotation to the test class Enable caching on the test class, defaulting to a NoOp cache instance

- Autoconfigure an embedded test database in place of a real one Create a **TestEntityManager** bean and add it to the application context

- Regular `@Component` beans are not loaded into the ApplicationContext.

```java
@RunWith(SpringRunner.class) 
@DataJpaTest 
public class UserRepositoryTests {

  @Autowired 
  private UserRepository userRepository;

  @Test 
  public void testFindByEmail() { 
    User user = userRepository.findByEmail("admin@gmail.com"); 
    assertNotNull(user); 
  }
}
```
---
title: Spring Data Management JDBC Transaction
search: true
tags:
- Spring
- Spring Data Management JDBC Transaction
- Spring Professional Certification
  toc: true
  toc_label: "My Table of Contents"
  toc_icon: "cog"
  classes: wide
---

## What is the difference between checked and unchecked exceptions?

Checked exceptions are exceptions that are checked at compile time – that fact enforces developer to handle by try catch or propagated via method to caller. Unchecked exceptions does not require that.

## How do you configure a DataSource in Spring?

Obtaining a DataSource in a Spring application depends on whether the application is deployed to an application or web server, for example GlassFish or Apache Tomcat, or if the application is a standalone application.
- Configuring data source using a JDBC driver for **Stand Alone Application**
- Implementing the object pool design pattern to provide data source objects:
  - Configure data source using JNDI **Web Server**
  - Configure data source using pool connections **Stand Alone Application**

### Configuring data source using JDBC Driver

- **DriverManagerDataSource:** It always creates a new connection for every connection request
- **SimpleDriverDataSource:** It is similar to the DriverManagerDataSource except that it works with the JDBC driver directly
- **SingleConnectionDataSource:**  It returns the same connection for every connection request, but it is not a pooled data source
- **EmbeddedDatabaseBuilder: Very useful in development/test databases.** Used to create an embedded database using Java Config like in the following example 

```java
return new EmbeddedDatabaseBuilder()
        .generateUniqueName(true)
        .setType(H2)
        .setScriptEncoding("UTF-8")
        .ignoreFailedDrops(true)
        .addScript("schema.sql")
        .addScripts("user_data.sql", "country_data.sql")
        .build();
```
## What is the Template design pattern and what is the JDBC template?

This pattern defines the outline or skeleton of an algorithm, and leaves the details to specific implementations later. This pattern hides away large amounts of boilerplate code. Spring provides many template classes, such as JdbcTemplate, JmsTemplate, RestTemplate, and WebServiceTemplate. Spring implements this pattern to access data from a database. In a database, or any other technology, there are some **steps** that are **always common**, such as **establishing a connection to the database, handling transactions, handling exceptions**, and some **clean up** actions which are required for **each data access process**. But there are also some steps which are not fixed, but depend on the application's requirement.

### What is JDBC template?

The Spring JdbcTemplate class is a Spring class that simplifies the use of JDBC by implementing common workflows for querying, updating, statement execution etc. Spring’s JdbcTemplate, in a nutshell, is responsible for the following:
- Acquisition of the connection
- Participation in the transaction
- Execution of the statement
- Processing of the result set
- Handling any exceptions
- Release of the connection

## What is a callback? What are the JdbcTemplate callback interfaces that can be used with queries? What is each used for? (You would not have to remember the interface names in the exam, but you should know what they do if you see them in a code sample).

A callback is code or reference to a piece of code that is passed as an argument to a method that, at some point during the execution of the methods, will call the code passed as an argument. In Java a callback can be a reference to a Java object that implements a certain interface or, starting with Java 8, a lambda expression.

### RowMapper

Spring provides a RowMapper interface for mapping a single row of a ResultSet to an object. It can be used for both single and multiple row queries.

```java
public interface RowMapper<T> {
  T mapRow(ResultSet rs, int rowNum) throws SQLException;
}
```

### RowCallBackHandler
Allows for processing rows in a result set one by one typically accumulating some type of result. Row callback handlers are typically stateful, storing the accumulated result in an instance variable. Note that the processRow method in this interface has a void return type.

```java
public interface RowCallbackHandler {
  void processRow(ResultSet rs) throws SQLException;
}
```

### ResultSetExtractor

Spring provides a ResultSetExtractor interface for processing an entire ResultSet at once. Here, you are responsible for iterating the ResultSet, for example, for mapping the entire ResultSet to a single object.

```java
import java.sql.SQLException;

public interface ResultSetExtractor<T> {
  T extractData(ResultSet rs) throws SQLException, DataAccessException;
}

public class AccountExtractor implements ResultSetExtractor<List<Account>> {
  @Override
  public List<Account> extractData(ResultSet resultSet) throws SQLException, DataAccessException {
      List<Account> extractedAccounts = null;
      Account account = null;
      while (resultSet.next()) {
          if (extractedAccounts == null) {
              extractedAccounts = new ArrayList<>();
              account = new Account(resultSet.getLong("ID"), resultSet,getString("NAME"), ...);
          }
          extractedAccounts.add(account);
      }
      return extractedAccounts;
  }
}
```

## Can you execute a plain SQL statement with the JDBC template?

Plain Sql statements can be executed using the JdbcTemplate class. The following methods accept one or more SQL strings as parameters:
- batchUpdate
- execute
- query
- queryForList
- queryForMap
- queryForObject
- queryForRowSet
- update

## When does the JDBC template acquire (and release) a connection, for every method called or once per  template? Why?

JdbcTemplate acquire and release a database connection for every method called. That is, a connection is acquired immediately before executing the operation at hand and released immediately after the operation has completed, be it successfully or with an exception thrown. The reason for this is to avoid holding on to resources (database connections) longer than necessary and creating as few database connections as possible, since creating connections can be a potentially expensive operation. When database connection pooling is used connections are returned to the pool for others to use.

## How does the JdbcTemplate support queries? How does it return objects and lists/maps of objects?

**queryForObject(..):** This is a query for simple java types (int, long, String, Date …) and for custom domain objects.
**queryForMap(..):** This is used when expecting a single row. JdbcTemplate returns each row of a ResultSet as a Map.
**queryForList(..):** This is used when expecting multiple rows.
The queryForList methods all return a list containing the resulting rows of the query and comes in two flavors:
- **One type that takes an element type as parameter.** This type of queryForList method returns a list containing objects of the type specified by the element type parameter.
- **The other type that do not have any element type parameter.** This type of queryForList method returns a list containing maps with string keys and Object values. Each map contains a row of the query result with the column names as keys and the column values as values in the map.

## What is a transaction? What is the difference between a local and a global transaction?

Transaction is an indivisible unit of work. **Local transactions** are transactions associated with **one single resource**, such as one single database or a queue of a message broker, but not both in one and the same transaction.
**Global transaction** is application server managed and spreads across many components/ technologies. For global transactions consider the case that a record must be persisted in a database ONLY if some message is sent to a queue and processed – if the later fail the transaction must be rolled back. **Global transactions requires a dedicated transaction manager.**

## Is a transaction a cross cutting concern? How is it implemented by Spring?

Yes, transaction management is a cross-cutting concern.

**AOP** is used to decorate beans with transactional behavior. This means that when we annotate classes or methods with `@Transactional`, a proxy bean will be created to provide the transactional behavior, and it is wrapped around the original bean **in an Around advice** that takes care of getting a transaction before calling the method and committing the transaction afterward.

AOP proxies use two infrastructure beans for this:

1. `TransactionInterceptor`

2. An implementation of `PlatformTransactionManager` interface. E.g.,
  1. `DataSourceTransactionManager`
  2. `HibernateTransactionManager`
  3. `JpaTransactionManager`
  4. `JtaTransactionManager`
  5. `WebLogicJtaTransactionManager`

**Under the hood**:

An internal infrastructure Spring-specific bean of type `InfrastructureAdvisorAutoProxyCreator` is registered and acts as a **bean postprocessor** that modifies the service and repository bean to add transaction-specific logic. Basically, this is the bean that creates the transactional AOP proxy.

When an exception is thrown from within the body of the transactional method, Spring checks the exception type in order to decide if the transaction will commit or rollback.

## How are you going to define a transaction in Spring?

Two ways of implementing it:

1. Declarative, use `@transaction`
2. Programmatic
  - Use `TransactionTemplate`
  - Use `PlatformTransactionManager`

### Declarative transaction management

Declarative transaction management is **non-invasive**.

1. **Configure transaction management support**
  - Marked a Spring Configuration class using the `@Configuration`
  - Using Java Configuration, define a `PlatformTransactionManager` bean using `@bean`
  - Enable transaction management by annotate the config file with `@EnableTransactionManagement`

    ```java
    @Configuration
    @EnableTransactionManagement
    public class TestDataConfig {
      @Bean
      public PlatformTransactionManager txManager(){
        return new DataSourceTransactionManager(dataSource());
      }
    }
    ```

2. Declare transactional methods using `@Transactional`

    ```java
    @Service
    public class UserServiceImpl implements UserService {
      @Transactional(propagation = Propagation.REQUIRED, readOnly= true)
      @Override
      public User findById(Long id) {
        return userRepo.findById(id);
      }
    }
    ```

### Programmatic transaction management

It is possible to use both declarative and programmatic transaction models simultaneously.

Programmatic transaction management allows you to control transactions through your codeexplicitly starting, committing, and joining them as you see fit.

Spring Framework provides two ways of implemeting Programmatic Transaction:

1. Using `TransactionTemplate`, which is recommended by Spring;
2. Using `PlatformTransactionManager` directly, which is low level.

**Using `TransactionTemplate`**
It uses a callback approach.

1. can write a `TransactionCallback` implementation, run `execute(..)`

    ```java
    public class SimpleService implements Service {

      // single TransactionTemplate shared amongst all methods in this instance
      private final TransactionTemplate transactionTemplate;

      // use constructor-injection to supply the PlatformTransactionManager
      public SimpleService(PlatformTransactionManager transactionManager) {
        this.transactionTemplate = new TransactionTemplate(transactionManager);
      }

      public Object someServiceMethod() {

        return transactionTemplate.execute(new TransactionCallback() {
          // the code in this method executes in a transactional context
          public Object doInTransaction(TransactionStatus status) {
            updateOperation1();
            return resultOfUpdateOperation2();
          }
        });
      }
    }
    ```

2. If there is no return value, use `TransactionCallbackWithoutResult`

    ```java
    transactionTemplate.execute(new TransactionCallbackWithoutResult() {
      protected void doInTransactionWithoutResult(TransactionStatus status) {
        updateOperation1();
        updateOperation2();
      }
    });
    ```

3. NB. Code within the callback can roll the transaction back by calling the `setRollbackOnly()` method on the supplied `TransactionStatus` object

    ```java
    transactionTemplate.execute(new TransactionCallbackWithoutResult() {

      protected void doInTransactionWithoutResult(TransactionStatus status) {
        try {
          updateOperation1();
          updateOperation2();
        } catch ( SomeBusinessException ex) {
          status.setRollbackOnly();
        }
      }
    });
    ```

**Using `PlatformTransactionManager`**

1. First, pass the implementation of the `PlatformTransactionManager` you use to your bean through a bean reference.
2. Then, by using the `TransactionDefinition` and `TransactionStatus` objects, you can initiate transactions, roll back, and commit.

```java
DefaultTransactionDefinition def = new DefaultTransactionDefinition();
// explicitly setting the transaction name is something that can be done only programmatically
def.setName("SomeTxName");
def.setPropagationBehavior(TransactionDefinition.PROPAGATION_REQUIRED);

TransactionStatus status = txManager.getTransaction(def);
try {
  // execute your business logic here
} catch (MyException ex) {
  txManager.rollback(status);
  throw ex;
}
txManager.commit(status);
```

## Is the JDBC template able to participate in an existing transaction?

Yes, the **JdbcTemplate** is able to participate in existing transactions both when declarative and programmatic transaction management is used. This is accomplished by wrapping the **DataSource** using a **TransactionAwareDataSourceProxy.**

## What is @EnableTransactionManagement for?

Enables Spring's annotation-driven transaction management capability. To be used on **@Configuration** classes.

```java
@configuration
@EnableTransactionManagement
public class AppConfig {}
```

Components registered when the **@EnableTransactionManagement** annotation is used are:
- **TransactionInterceptor**. Intercepts calls to @Transactional methods creating new
  transactions as necessary etc.
- **A JDK Proxy or AspectJ advice.**. This advice intercepts methods annotated with
  @Transactional (or methods that are located in a class annotated with @Transactional).

**@EnableTransactionManagement**  and \<tx:annotation-driven\/> only looks for @Transactional on beans in the same application context they are defined in. This means that, if you put annotation driven configuration in a WebApplicationContext for a DispatcherServlet, it only checks for @Transactional beans in your controllers, and not your services.
## How does transaction propagation work?

Transaction propagation determines the way an existing transaction is used when the method is invoked.
- MANDATORY. There must be an existing transaction when the method is invoked, or an exception will be thrown
- NESTED. Executes in a nested transaction if a transaction exists, otherwise a new transaction will be created. This transaction propagation mode is not implemented in all transaction managers
- NEVER. Method is executed outside of a transaction. Throws exception if a transaction exists
- NOT_SUPPORTED. Method is executed outside of a transaction. Suspends any existing transaction
- REQUIRED (**DEFAULT**). Method will be executed in the current transaction. If no transaction exists, one will be created
- REQUIRES_NEW. Creates a new transaction in which the method will be executed. Suspends any existing transaction

- SUPPORTS. Method will be executed in the current transaction, if one exists, or outside of a transaction if one does not exist

## What happens if one @Transactional annotated method is calling another @Transactional annotated method  inside a same object instance?

Self-invocation of a proxied Spring bean effectively bypasses the proxy and thus also any transaction interceptor managing transactions. Thus the second method, the method being invoked from another method in the bean, will execute in the same transaction context as the first.

## Where can the @Transactional annotation be used? What is a typical usage if you put it at class level?

The **@Transactional** annotation can be used on class and method level both in classes and interfaces. When using Spring AOP proxies, only **@Transactional** annotations on public methods will have any effect – applying the **@Transactional** annotation to protected or private methods or methods with package visibility will not cause errors but will not give the desired transaction management, as above.
**Spring recommends that you only annotate concrete classes (and methods of concrete classes) with the @Transactional annotation, as opposed to annotating interfaces.** You certainly can place the @Transactional annotation on an interface (or an interface method), but **this works only if you are using interface-based proxies.**

## What does declarative transaction management mean?

Declarative transaction management is a model build on AOP. Spring has some transactional aspects that may be used to advise methods for them to work in a transactional manner.

## What is the default rollback policy? How can you override it?

In its default configuration, the Spring Framework’s transaction infrastructure code **only marks a transaction for rollback in the case of runtime, unchecked exceptions** (instance of Runtime Exception. Errors will also – by default - result in a rollback).
The types of exceptions that are to cause a rollback can be configured using the **rollbackFor** element of the @Transactional annotation. In addition, the types of exceptions that not are to cause rollbacks can also be configured using the **noRollbackFor** element.

## What is the default rollback policy in a JUnit test, when you use the @ RunWith(SpringJUnit4ClassRunner.class) in JUnit 4 or @ExtendWith(SpringExtension. class) in JUnit 5, and annotate your @Test annotated method with @Transactional?

If a test-method annotated with @Test is also annotated with @Transactional, then the framework creates and rolls back a transaction for each test.

## Are you able to participate in a given transaction in Spring while working with JPA?

Yes, if you register TransactionManager.
The Spring JpaTransactionManager supports direct DataSource access within one and the same transaction allowing for mixing plain JDBC code that is unaware of JPA with code that use JPA.

## Which PlatformTransactionManager(s) can you use with JPA?

First, any JTA transaction manager can be used with JPA since JTA transactions are global transactions, that is they can span multiple resources such as databases, queues etc. Thus JPA persistence becomes just another of these resources that can be involved in a transaction. When using JPA with one single entity manager factory, the Spring Framework JpaTransactionManager is the recommended choice. This is also the only transaction manager that is JPA entity manager factory aware.
If the application has multiple JPA entity manager factories that are to be transactional, then a JTA transaction manager is required.

## What do you have to configure to use JPA with Spring? How does Spring Boot make this easier?

1. Configure DataSource
2. Configure LocalContainerEntityManagerFactoryBean
3. Configure JpaVendorAdapter
4. Configure PlatformTransactionManager
5. Configure PersistenceExceptionTransaltionPostProcessor (Note that exception
   translation, whether with JPA or Hibernate, isn’t mandatory. If you’d prefer that your repository throw JPA-specific or Hibernate-specific exceptions, you’re welcome to forgo PersistenceExceptionTranslationPostProcessor and let the native exceptions flow freely.
6. Configure PersistenceAnnotationBeanPostProcessor (if you want to use @PersistenceUnit and @PersistenceContext)
   
Spring boot has Autoconfigure class called JpaBaseConfiguration, which create this out of the box.
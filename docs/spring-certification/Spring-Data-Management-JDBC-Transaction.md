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
## What is a transaction? What is the difference between a local and a global transaction?
## Is a transaction a cross cutting concern? How is it implemented by Spring?
## How are you going to define a transaction in Spring?
## Is the JDBC template able to participate in an existing transaction?
## What is @EnableTransactionManagement for?
## How does transaction propagation work?
## What happens if one @Transactional annotated method is calling another @Transactional annotated method  inside a same object instance?
## Where can the @Transactional annotation be used? What is a typical usage if you put it at class level?
## What does declarative transaction management mean?
## What is the default rollback policy? How can you override it?
## What is the default rollback policy in a JUnit test, when you use the @ RunWith(SpringJUnit4ClassRunner.class) in JUnit 4 or @ExtendWith(SpringExtension. class) in JUnit 5, and annotate your @Test annotated method with @Transactional?
## Are you able to participate in a given transaction in Spring while working with JPA?
## Which PlatformTransactionManager(s) can you use with JPA?
## What do you have to configure to use JPA with Spring? How does Spring Boot make this easier?
---

title: Spring Data JPA

---

* [What is a Spring Data Repository interface?](#what-is-a-spring-data-repository-interface)
* [How do you define a Spring Data Repository interface? Why is it an interface not a class?](#how-do-you-define-a-spring-data-repository-interface-why-is-it-an-interface-not-a-class)
* [What is the naming convention for finder methods in a Spring Data Repository interface?](#what-is-the-naming-convention-for-finder-methods-in-a-spring-data-repository-interface)
* [How are Spring Data repositories implemented by Spring at runtime?](#how-are-spring-data-repositories-implemented-by-spring-at-runtime)
* [What is @Query used for?](#what-is-query-used-for)

## What is a Spring Data Repository interface?

An instant repository, also known as a Spring Data repository, is a repository that need no implementation and that supports the basic CRUD (create, read, update and delete) operations. Such a repository is declared as an interface that typically extend the Repository interface or an interface extending the Repository interface. Annotating an interface with **@RepositoryDefinition** will cause the same behaviour as extending Repository.

## How do you define a Spring Data Repository interface? Why is it an interface not a class?

You have 2 options:
- Extend some implementation of repository interface.
- Annotate the future instant repository class with @RepositoryDefinition

Repositories are defined as interfaces in order for Spring Data to be able to use the JDK dynamic proxy mechanism to create the proxy objects that intercept calls to repositories. With the Spring Data repository interfaces in place, **annotate a Spring configuration class with an annotation @EnableJpaRepositories** to enable the discovery and creation of repositories.

## What is the naming convention for finder methods in a Spring Data Repository interface?

The naming convention of these finder methods are:
```java
find(First[count])By[property expression][comparison operator][ordering operator]
```
- If a count is supplied after the “First”, for example “findFirst10”, then the count number of entities first found will be the result
- Optional comparison operator enables creation of finder methods that selects a range of entities. Examples: LessThan, GreaterThan

## How are Spring Data repositories implemented by Spring at runtime?

For a Spring Data repository a JDK dynamic proxy is created which intercepts all calls to the repository. The default behavior is to route calls to the default repository implementation, which in Spring Data JPA is the SimpleJpaRepository class.

## What is @Query used for?

The @Query annotation allows for specifying a query to be used with a Spring Data JPA repository method.

```java
@Query("select s from Spitter s where s.email like '%gmail.com'")
List<Spitter> findAllGmailSpitters();
```

```java
@Query("select u from User u where u.firstname = :firstname or u.lastname = :lastname")
User findByLastnameOrFirstname(@Param("lastname") String lastname,
                               @Param("firstname") String firstname);                   
```

## References

1. [MrR0807 Spring certification notes](https://github.com/MrR0807/SpringCertification5.0)
2. [Moss Green Spring certification notes](https://mossgreen.github.io/)
3. [Spring Documentation](https://docs.spring.io/spring-framework/docs/5.3.8/reference/html/)
4. [Spring Boot Documentation](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/)
---

title: Spring MVC Basics

---

* [What is the @Controller annotation used for?](#what-is-the-controller-annotation-used-for)
* [How is an incoming request mapped to a controller and mapped to a method?](#how-is-an-incoming-request-mapped-to-a-controller-and-mapped-to-a-method)
* [What is the difference between @RequestMapping and @GetMapping?](#what-is-the-difference-between-requestmapping-and-getmapping)
* [What is @RequestParam used for?](#what-is-requestparam-used-for)
* [What are the differences between @RequestParam and @PathVariable?](#what-are-the-differences-between-requestparam-and-pathvariable)
* [What are the ready-to-use argument types you can use in a controller method?](#what-are-the-ready-to-use-argument-types-you-can-use-in-a-controller-method)
* [What are some valid return types of a controller method?](#what-are-some-valid-return-types-of-a-controller-method)

## What is the @Controller annotation used for?

The @Controller annotation is a specialization of the @Component annotation. In a web application, the **controllers work between the web layer and the core application layer.** In the Spring MVC framework, controllers are also more like POJO classes with methods; **these methods are known as handlers**, because these are annotated with the @RequestMapping annotation.
You could also use the **@Component** annotation instead of **@Controller** to create Spring beans in a web application, but in this case, that bean does not have the capability of the Spring MVC framework such as **exception handling at web layer, handler mapping**, and so on.

## How is an incoming request mapped to a controller and mapped to a method?

When a request is issued to the application:
- DispatcherServlet of the application receives the request.
- DispatcherServlet maps the request to a method in a controller.
- DispatcherServlet holds a list of classes implementing the HandlerMapping interface.
- DispatcherServlet dispatches the request to the controller.
- The method in the controller is executed.
  
/articles/new 
Dispatcher servlet: / 
Controller: /articles 
Method: /new

## What is the difference between @RequestMapping and @GetMapping?

@GetMapping is a composed annotation that acts as a shortcut for @RequestMapping(method = RequestMethod.GET).

## What is @RequestParam used for?

```java
@Controller 
public  class  AccountController {
    @GetMapping(value = "/account")
    public String getAccountDetails (ModelMap model, HttpServletRequest request) {
        String accountId = request.getParameter("accountId");
        Account account = accountService.findOne(Long.valueOf(accountId));
        model.put("account", account);
        return "accountDetails";
    } 
}
```

The @RequestParam annotation is used to annotate parameters to handler methods in order to bind request parameters to method parameters.
Assume there is a controller method with the following signature and annotations:

```java
@RequestMapping("/greeting")
public String greeting(@RequestParam(name="name", required=false) String inName) {
        ...
}
```

If then a request is sent to the URL http://localhost:8080/greeting?name=Ivan then the inName method parameter will contain the string “Ivan”.

## What are the differences between @RequestParam and @PathVariable?

Spring MVC allows you to pass parameters in the URI instead of passing them through request parameters. The passed values can be extracted from the request URLs. It is based on URI templates. It allows clean URLs without request parameters. The following is an example:

```java
@Controller
public class AccountController {
    @GetMapping("/accounts/{accountId}")
    public String show(@PathVariable("accountId") long accountId, Model model) {
        Account account = accountService.findOne(accountId);
        model.put("account", account);
        return "accountDetails";
    }
  ...
}
```

**Difference**
The difference between the @RequestParam annotation and the @PathVariable annotation is that they map different parts of request URLs to handler method arguments.

## What are the ready-to-use argument types you can use in a controller method?

- WebRequest
- ServletRequest
- ServletResponse
- HttpSession
- Principle
- HttpMethod
- Locale
- TimeZone
- java.io
- HttpEntity
- Collections, like Map<>
- Errors
- BindingResult
- SessionStatus
- UriComponentsbuilder

###What other annotations might you use on a controller method parameter?
(You can ignore form-handling annotations for this exam)

- @PathVariable
- @MatrixVariable: key-value pair in url
- @RequestParam
- @CookieValue
- @RequestBody
- @RequestHeader
- @RequestPart: “multipart/form-data”
- @ModelAttribute
- @SessionAttribute
- @SessionAttributes
- @RequestAttribute

## What are some valid return types of a controller method?

- HttpEntity
- ResponseEntity
- HttpHeaders
- String
- View
- Map<>
- ModelAndView
- void
- CompletableFuture | CompletionStage: Asynchronous

## References

1. [MrR0807 Spring certification notes](https://github.com/MrR0807/SpringCertification5.0)
2. [Moss Green Spring certification notes](https://mossgreen.github.io/)
3. [Spring Documentation](https://docs.spring.io/spring-framework/docs/5.3.8/reference/html/)
4. [Spring Boot Documentation](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/)

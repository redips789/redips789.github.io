---
title: Spring AOP
search: true
tags:
  - Spring
  - Spring AOP
  - Spring Professional Certification
toc: true
toc_label: "My Table of Contents"
toc_icon: "cog"
classes: wide
---

## What is the concept of AOP? Which problem does it solve? What is a cross cutting concern?
### What is the concept of AOP?
Aspect-Oriented Programming (AOP) enables modularization of cross-cutting concerns. It complements Object-oriented programming (OOP). OOP has class and object as key elements but AOP has aspect as key element. Aspects allow you to modularize some functionality across the application at multiple points. This type of functionality is known as cross-cutting concerns.
### Which problem does it solve?
1. **Code tangling**. Code tangling occurs when there is a mixing of cross-cutting concerns with the application's business logic. It promotes tight coupling between the cross- cutting and business modules.
2. **Code scattering**.  This means that the same concern is spread across modules in the application. Code scattering promotes the duplicity of the concern's code across the application modules

### What is cross cutting concern?
In any application, there is some generic functionality that is needed in many places. But this functionality is not related to the application's business logic. Suppose you perform a role-based security check before every business method in your application. Here security is a cross- cutting concern. Example:
- Logging and tracing 
- Transaction management
- Security
- Caching
- Error handling
- Performance monitoring
- Custom business rules

## What is a pointcut, a join point, an advice, an aspect, weaving?
### Pointcut
A point-cut selects one or more join points out of the set of all join points in an application.
![Pointcut](/docs/assets/images/Pointcut.jpg)
### Join Point
A point during the execution of a program, such as the execution of a method or the handling of an exception. In Spring AOP, a join point always represents a method execution.
![Pointcut](/docs/assets/images/Joint Points.jpg)
### Advice
Action taken by an aspect at a particular join point. Types:
- Before – advice is executed first and then it calls the Target method
- After – advice executed after the target method terminates by throwing any exception
  or normally
- After-returning – advice is executed after the target returns successfully. This advice
  will never execute if target throws any exception in the application
- After-throwing – advice is executed after the target throws an exception. This advice
  will never execute if the target doesn't throw any exception in the application
- Around – executed two times, first time it is executed before the advised method and
  second time it is executed after advised method is invoked.
### Aspect
An aspect is the merger of advice and point-cuts. Taken together, advice and point-cuts define everything there is to know about an aspect – what it does and where and when it does it.
### Weawing
Weaving is the process of applying aspects to a target object to create a new proxied object. The aspects are woven into the target object at the specified join points. The weaving can take place at several points in the target object’s lifetime:
- Compile time
- Class load time
- Runtime

**Spring AOP aspects are woven at Runtime.**
**If the advice throws an exception, target will not be called - this is a valid use of a Before Advice.**
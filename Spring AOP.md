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

##What is the concept of AOP? Which problem does it solve? What is a cross cutting concern?
###What is the concept of AOP?
Aspect-Oriented Programming (AOP) enables modularization of cross-cutting concerns. It complements Object-oriented programming (OOP). OOP has class and object as key elements but AOP has aspect as key element. Aspects allow you to modularize some functionality across the application at multiple points. This type of functionality is known as cross-cutting concerns.
###Which problem does it solve?
1. **Code tangling**. Code tangling occurs when there is a mixing of cross-cutting concerns with the application's business logic. It promotes tight coupling between the cross- cutting and business modules.
2. **Code scattering**.  This means that the same concern is spread across modules in the application. Code scattering promotes the duplicity of the concern's code across the application modules

###What is cross cutting concern?
In any application, there is some generic functionality that is needed in many places. But this functionality is not related to the application's business logic. Suppose you perform a role-based security check before every business method in your application. Here security is a cross- cutting concern. Example:
- Logging and tracing 
- Transaction management
- Security
- Caching
- Error handling
- Performance monitoring
- Custom business rules
# NestJS Architecture & Design Patterns - Top Interview Questions

## Architecture Fundamentals

1. What is the recommended architecture in NestJS (layered architecture)?
2. What are the typical layers (Controller, Service, Repository)?
3. What is Separation of Concerns (SoC)?
4. What is Dependency Injection and why is it important?

## Modular Architecture

5. What is a Module in NestJS?
6. How do you organize code into modules?
7. What is feature-based modularization?
8. When should you create a new module?
9. What is the difference between shared modules and feature modules?

## Design Patterns

10. What is Dependency Injection pattern?
11. What is Repository Pattern and when to use it?
12. What is Factory Pattern and how is it used in NestJS?
13. What is Strategy Pattern (e.g., Passport strategies)?
14. What is Decorator Pattern (used extensively in NestJS)?
15. What is Singleton Pattern (providers)?
16. What is Observer Pattern (event-driven architecture)?

## Domain-Driven Design (DDD)

17. What is Domain-Driven Design?
18. What are Entities, Value Objects, and Aggregates?
19. How do you implement DDD in NestJS?
20. What is the difference between Domain Layer and Application Layer?

## SOLID Principles

21. What are SOLID principles?
22. What is Single Responsibility Principle (SRP)?
23. What is Dependency Inversion Principle (DIP)?
24. How does NestJS enforce SOLID principles?

## Service Layer Patterns

25. What should go in a Service vs Controller?
26. Should business logic be in Controllers or Services?
27. How do you handle complex business logic?
28. What is the difference between Service and Repository?

## Error Handling Patterns

29. How should you structure error handling in large applications?
30. Should you use try-catch in controllers or services?
31. What is the global error handling strategy?

## Code Organization

32. How do you structure a large NestJS project?
33. What is the recommended folder structure?
34. How do you organize shared code?
35. Should you use barrel exports (index.ts)?

## Dependency Injection Scopes

36. What are Provider scopes (DEFAULT, REQUEST, TRANSIENT)?
37. What is REQUEST scope and when to use it?
38. What is TRANSIENT scope?
39. What are the performance implications of REQUEST scope?

## DTOs (Data Transfer Objects)

40. What is a DTO and why use them?
41. Where should DTOs be defined?
42. Should DTOs contain logic?
43. What is the difference between Entity and DTO?

## Event-Driven Architecture

44. What is event-driven architecture?
45. How do you implement event emitters in NestJS?
46. What is CQRS (Command Query Responsibility Segregation)?
47. How do you implement CQRS in NestJS using `@nestjs/cqrs`?
48. What is Event Sourcing?

## Scalability Patterns

49. How do you design for horizontal scalability?
50. What is stateless architecture and why is it important?

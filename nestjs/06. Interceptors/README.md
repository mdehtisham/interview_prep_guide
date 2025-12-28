# NestJS Interceptors - Top Interview Questions

## Interceptor Fundamentals

1. What is an Interceptor in NestJS?
2. When are Interceptors executed in the request lifecycle?
3. What is the `NestInterceptor` interface?
4. What parameters does the `intercept()` method receive?
5. What is RxJS and why is it used in Interceptors?
6. Can Interceptors modify request and response?

## Creating Interceptors

7. How do you create an Interceptor implementing `NestInterceptor`?
8. What is `ExecutionContext` in Interceptors?
9. What is `CallHandler` and what does `handle()` do?

## RxJS Operators

10. What are the most common RxJS operators used in Interceptors?
11. What is `tap()` operator used for?
12. What is `map()` operator used for?
13. What is `catchError()` operator used for?
14. What is the difference between `tap()` and `map()`?

## Applying Interceptors

15. How do you apply an Interceptor using `@UseInterceptors()`?
16. How do you apply an Interceptor globally?
17. What is the order when multiple Interceptors are applied?
18. What is `APP_INTERCEPTOR` token?

## Common Use Cases

19. How do you implement a logging Interceptor to measure execution time?
20. How do you transform responses using `map()` operator?
21. How do you wrap responses in a standard format?
22. How do you implement a timeout Interceptor using `timeout()`?
23. How do you implement error handling using `catchError()`?
24. What is `ClassSerializerInterceptor` used for?
25. How do you exclude sensitive data using `@Exclude()` decorator?

## Cache Interceptor

26. What is the built-in `CacheInterceptor`?
27. How do you cache responses?
28. How do you use `@CacheKey()` and `@CacheTTL()` decorators?

## Interceptors vs Other Features

29. What is the difference between Interceptors and Middleware?
30. What is the difference between Interceptors and Guards?
31. When should you use Interceptors vs Middleware vs Guards?

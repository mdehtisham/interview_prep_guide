# NestJS Exception Filters - Top Interview Questions

## Exception Filter Fundamentals

1. What is an Exception Filter in NestJS?
2. When are Exception Filters executed?
3. What is the `ExceptionFilter` interface?
4. What parameters does the `catch()` method receive?

## Built-in Exceptions

5. What are the most commonly used built-in HTTP exceptions?
6. What is `HttpException` and how is it used?
7. When do you use `BadRequestException`?
8. When do you use `UnauthorizedException` vs `ForbiddenException`?
9. When do you use `NotFoundException`?
10. What is `InternalServerErrorException`?

## Creating Exception Filters

11. How do you create a custom Exception Filter?
12. What is `ArgumentsHost` and how do you use it?
13. How do you access the request and response objects?
14. How do you send custom error responses?

## Applying Exception Filters

15. How do you apply an Exception Filter using `@UseFilters()`?
16. How do you apply an Exception Filter globally?
17. What is `APP_FILTER` token?

## Custom Exceptions

18. How do you create a custom exception by extending `HttpException`?
19. How do you pass custom data in exceptions?
20. How do you set custom status codes and messages?

## Error Response Format

21. What is the default error response format in NestJS?
22. How do you customize the error response structure?
23. Should you include stack traces in production?

## Global Error Handling

24. How do you implement a global Exception Filter to catch all exceptions?
25. How do you handle validation errors from ValidationPipe?
26. How do you handle unknown/unhandled exceptions?

## Best Practices

27. Should you log errors in Exception Filters?
28. Should you send detailed error information to clients?
29. What is the difference between Exception Filters and Interceptors for error handling?

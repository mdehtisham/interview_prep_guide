# NestJS Pipes - Top Interview Questions

## Pipe Fundamentals

1. What is a Pipe in NestJS?
2. What are the two main use cases for Pipes (validation and transformation)?
3. When are Pipes executed in the request lifecycle?
4. What is the `PipeTransform` interface?
5. Can Pipes be async?

## Built-in Pipes

6. What are the most commonly used built-in Pipes?
7. What is `ValidationPipe` and when do you use it?
8. What is `ParseIntPipe` and how do you use it?
9. What is `ParseUUIDPipe` used for?
10. What is `DefaultValuePipe` for?

## Creating Custom Pipes

11. How do you create a custom Pipe implementing `PipeTransform`?
12. What parameters does the `transform()` method receive?
13. What is `ArgumentMetadata` and what properties does it contain?

## Applying Pipes

14. How do you apply a Pipe to a route handler parameter?
15. How do you apply ValidationPipe globally?
16. What is the `APP_PIPE` token?

## Validation with ValidationPipe

17. How do you use `ValidationPipe` for DTO validation?
18. What is a DTO (Data Transfer Object)?
19. What is `class-validator` library?
20. What is `class-transformer` library?
21. What important options can you pass to ValidationPipe (`whitelist`, `forbidNonWhitelisted`, `transform`)?
22. What does the `whitelist` option do?
23. What does the `transform` option enable?

## Validation Decorators

24. What are the most common validation decorators from class-validator?
25. How do you use `@IsString()`, `@IsNumber()`, `@IsEmail()`?
26. How do you use `@MinLength()` and `@MaxLength()`?
27. How do you use `@IsOptional()` for optional fields?
28. What is `@ValidateNested()` used for?

## Transformation

29. How do you use Pipes for data transformation?
30. How do you convert string to number using `ParseIntPipe`?
31. What is auto-transformation in ValidationPipe?

## Error Handling

32. What exception do Pipes throw on validation failure?
33. How do you customize validation error messages?
34. What is the structure of ValidationPipe error responses?

## File Upload Pipes

35. How do you validate file uploads using `ParseFilePipe`?
36. How do you restrict file size and types?

## Pipes vs Other Features

37. What is the difference between Pipes and Guards?
38. When should you use Pipes vs Guards?
39. What is the order of execution (Pipes come after Guards)?

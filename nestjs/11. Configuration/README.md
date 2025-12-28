# NestJS Configuration - Top Interview Questions

## Configuration Fundamentals

1. What is `@nestjs/config` package?
2. Why do you need configuration management?
3. What is the purpose of `.env` files?
4. How do you install and set up `ConfigModule`?

## ConfigModule Setup

5. How do you import `ConfigModule` globally using `ConfigModule.forRoot()`?
6. What options can you pass to `ConfigModule.forRoot()`?
7. What does `isGlobal: true` do?
8. How do you specify custom `.env` file paths using `envFilePath`?
9. What is `ignoreEnvFile` option used for?

## Accessing Configuration

10. What is `ConfigService` and how do you inject it?
11. How do you get configuration values using `ConfigService.get()`?
12. How do you provide default values in `get()` method?
13. How do you access nested configuration?

## Environment Variables

14. How do you define environment variables in `.env` files?
15. How do you access `process.env` in NestJS?
16. Should you commit `.env` files to version control?
17. What is `.env.example` file for?

## Validation

18. How do you validate environment variables using Joi?
19. What is `validationSchema` in ConfigModule?
20. How do you make environment variables required?
21. What happens if required env variables are missing?

## Custom Configuration Files

22. How do you create custom configuration files (e.g., database.config.ts)?
23. How do you use `registerAs()` to create namespaced configuration?
24. How do you load custom config files using `load` option?
25. How do you inject namespaced config using `@Inject()`?

## Type Safety

26. How do you create TypeScript interfaces for configuration?
27. How do you use generics with `ConfigService.get<T>()`?

## Multiple Environments

28. How do you manage different environments (development, production, testing)?
29. How do you load different `.env` files per environment?
30. What is `NODE_ENV` and how do you use it?

## Best Practices

31. Should you use ConfigService or process.env directly?
32. Where should you validate configuration?
33. How do you handle sensitive data like API keys?
34. Should configuration be synchronous or asynchronous?
35. How do you use ConfigModule with database connections (`forRootAsync`)?

## Common Patterns

36. How do you implement configuration for TypeORM using ConfigService?
37. How do you implement configuration for JWT using ConfigService?
38. How do you cache configuration values?

## Testing

39. How do you mock ConfigService in tests?
40. How do you test with different configuration values?

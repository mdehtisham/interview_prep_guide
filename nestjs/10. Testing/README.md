# NestJS Testing - Top Interview Questions

## Testing Fundamentals

1. What testing frameworks does NestJS support?
2. What is Jest and why is it the default in NestJS?
3. What are the types of tests (unit, integration, e2e)?
4. What is the difference between unit tests and integration tests?

## Unit Testing

5. How do you set up unit tests for services?
6. What is `Test.createTestingModule()` used for?
7. How do you mock dependencies in unit tests?
8. How do you mock repositories and external services?
9. How do you use `jest.fn()` to create mock functions?
10. How do you use `jest.spyOn()` for mocking methods?

## Testing Controllers

11. How do you unit test controllers?
12. How do you mock services in controller tests?
13. How do you test request and response objects?

## Testing Services

14. How do you unit test services?
15. How do you inject mocked dependencies into services?
16. How do you test async methods?
17. How do you test error handling in services?

## Mocking

18. What is `@nestjs/testing` package?
19. How do you create mock providers?
20. How do you override providers in tests using `overrideProvider()`?
21. How do you mock database repositories?
22. How do you use `useValue`, `useClass`, `useFactory` in mocks?

## Integration Testing

23. What are integration tests?
24. How do you set up a test database for integration tests?
25. How do you test multiple modules together?

## E2E (End-to-End) Testing

26. What is E2E testing and why is it important?
27. How do you set up E2E tests using `supertest`?
28. How do you test HTTP endpoints in E2E tests?
29. How do you test authentication in E2E tests?
30. How do you test protected routes?

## Testing Guards, Pipes, Interceptors

31. How do you test Guards?
32. How do you test Pipes?
33. How do you test Interceptors?
34. How do you mock `ExecutionContext` for Guard tests?

## Test Coverage

35. How do you measure test coverage using Jest?
36. What is a good test coverage percentage?
37. How do you generate coverage reports?

## Best Practices

38. Should you test private methods?
39. What is AAA pattern (Arrange, Act, Assert)?
40. How do you organize test files?
41. What should you test and what should you not test?
42. How do you handle database cleanup in tests?
43. How do you test error scenarios?

## Common Testing Patterns

44. How do you test database transactions?
45. How do you test file uploads?
46. How do you test WebSockets?
47. How do you test scheduled jobs/cron tasks?
48. How do you test event emitters?

## Test Performance

49. How do you speed up test execution?
50. Should you use in-memory databases for tests?

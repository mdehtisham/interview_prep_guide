# NestJS Logging & Monitoring - Top Interview Questions

## Logging Fundamentals

1. Why is logging important in applications?
2. What are the different log levels (error, warn, info, debug, verbose)?
3. When should you use each log level?
4. What is the built-in Logger in NestJS?

## Built-in Logger

5. How do you use the built-in `Logger` class?
6. How do you inject Logger into services?
7. How do you create a logger instance with context?
8. How do you disable logging?
9. Can you customize log format?

## Custom Loggers

10. How do you create a custom logger?
11. What is the `LoggerService` interface?
12. How do you implement custom log formatting?

## Third-Party Logging Libraries

13. What popular logging libraries work with NestJS (Winston, Pino)?
14. How do you integrate Winston logger?
15. What are the advantages of Winston (transports, formatting)?
16. How do you integrate Pino for high-performance logging?
17. What is the difference between Winston and Pino?

## Log Transports

18. What are log transports?
19. How do you log to files using Winston?
20. How do you log to external services (Loggly, Papertrail)?
21. How do you send logs to Elasticsearch/Kibana (ELK Stack)?

## Structured Logging

22. What is structured logging?
23. Why is JSON format preferred for logs?
24. How do you include metadata in logs (request ID, user ID)?
25. How do you implement correlation IDs for request tracking?

## Application Monitoring

26. What is Application Performance Monitoring (APM)?
27. What monitoring tools can you use (New Relic, Datadog, Sentry)?
28. How do you integrate Sentry for error tracking?
29. How do you monitor API response times?
30. How do you set up alerts for errors?

## Health Checks

31. What are health checks and why are they important?
32. How do you implement health check endpoints using `@nestjs/terminus`?
33. What is liveness vs readiness probe?
34. How do you check database health?
35. How do you check external API health?

## Metrics & Observability

36. What metrics should you track (latency, throughput, error rate)?
37. How do you integrate Prometheus for metrics?
38. What is the RED method (Rate, Errors, Duration)?
39. What is distributed tracing?

## Best Practices

40. Should you log sensitive information (passwords, tokens)?
41. How do you handle log rotation?
42. What log retention policies should you follow?
43. Should you log in production vs development differently?
44. How do you aggregate logs from multiple instances?
45. What information should every log entry include?

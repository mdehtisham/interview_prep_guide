# NestJS Performance Optimization - Top Interview Questions

## Performance Fundamentals

1. What are the key performance metrics (response time, throughput, latency)?
2. What tools can you use to measure NestJS performance?
3. What are common performance bottlenecks?

## Caching

4. What is caching and why is it important?
5. How do you implement caching in NestJS using `CacheModule`?
6. What is `CacheInterceptor` and how do you use it?
7. What cache stores does NestJS support (in-memory, Redis)?
8. How do you configure Redis as a cache store?
9. How do you set cache TTL (Time To Live)?
10. How do you invalidate cache?
11. What should you cache and what should you not cache?

## Database Optimization

12. What are N+1 query problems and how do you prevent them?
13. What is eager loading vs lazy loading in TypeORM?
14. How do you use database indexing to improve performance?
15. How do you use query optimization and EXPLAIN?
16. Should you use `find()` or QueryBuilder for complex queries?
17. How do you implement database connection pooling?
18. How do you use pagination to reduce query load?

## Compression

19. What is compression and why use it?
20. How do you enable response compression using `compression` middleware?
21. What content should be compressed?

## Rate Limiting & Throttling

22. What is rate limiting?
23. How do you implement rate limiting using `@nestjs/throttler`?
24. What is the `@Throttle()` decorator?
25. Why is rate limiting important for API security and performance?

## Lazy Loading Modules

26. What is lazy loading of modules?
27. How do you implement lazy-loaded modules?
28. When should you use lazy loading?

## Streaming

29. What is streaming and when should you use it?
30. How do you stream large responses?
31. How do you stream file downloads?

## Async Operations

32. How do you optimize async operations?
33. What is `Promise.all()` and when to use it for parallel execution?
34. Should you use callbacks, promises, or async/await?

## Load Balancing & Clustering

35. What is clustering in Node.js?
36. How do you use PM2 for clustering NestJS applications?
37. What is horizontal scaling vs vertical scaling?
38. How do you implement load balancing?

## Memory Management

39. What causes memory leaks in Node.js?
40. How do you detect memory leaks?
41. How do you profile memory usage?
42. What is garbage collection in Node.js?

## Monitoring & Profiling

43. How do you monitor application performance?
44. What tools can you use (New Relic, Datadog, Prometheus)?
45. How do you use Chrome DevTools for profiling?

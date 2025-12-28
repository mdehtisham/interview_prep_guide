# NestJS Scheduling & Background Jobs - Top Interview Questions

## Scheduling Fundamentals

1. What is task scheduling and when do you need it?
2. What is `@nestjs/schedule` package?
3. What types of scheduled tasks can you create (cron, intervals, timeouts)?

## Cron Jobs

4. What are cron jobs?
5. How do you create a cron job using `@Cron()` decorator?
6. What is cron syntax and how do you read it?
7. What are common cron patterns (every minute, hourly, daily)?
8. How do you schedule a job to run at midnight every day?
9. How do you use cron names for managing jobs?

## Intervals & Timeouts

10. How do you schedule tasks at intervals using `@Interval()`?
11. What is the difference between `@Interval()` and `@Cron()`?
12. How do you schedule one-time delayed tasks using `@Timeout()`?

## Dynamic Scheduling

13. How do you dynamically schedule/unschedule jobs?
14. What is `SchedulerRegistry` and how do you use it?
15. How do you add a cron job dynamically at runtime?
16. How do you delete a scheduled job?

## Use Cases

17. What are common use cases for scheduled jobs?
18. How do you implement data cleanup jobs?
19. How do you send scheduled email notifications?
20. How do you implement report generation?
21. How do you sync data with external APIs periodically?

## Queue-Based Jobs

22. What is a job queue and when should you use it?
23. What is Bull queue library?
24. How do you integrate Bull with NestJS using `@nestjs/bull`?
25. What is Redis and why is it used with Bull?
26. How do you create a queue processor?
27. How do you add jobs to a queue?
28. What is the difference between cron jobs and queues?

## Job Processing

29. How do you handle job failures?
30. How do you implement retry logic for failed jobs?
31. How do you implement job prioritization?
32. How do you process jobs concurrently?

## Background Workers

33. What are background workers?
34. When should you use background workers vs cron jobs?
35. How do you ensure jobs don't run twice in clustered environments?

## Monitoring & Logging

36. How do you monitor scheduled jobs?
37. How do you log job execution?
38. How do you handle long-running jobs?

## Best Practices

39. Should scheduled jobs contain heavy logic or delegate to services?
40. How do you test scheduled jobs?
41. How do you handle timezone considerations?
42. How do you prevent jobs from overlapping (job locking)?
43. Should you run scheduled jobs in production on all instances?
44. How do you use distributed locks to ensure single execution in multi-instance setups?
45. What are the performance implications of many scheduled jobs?

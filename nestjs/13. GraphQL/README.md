# NestJS GraphQL - Top Interview Questions

## GraphQL Fundamentals

1. What is GraphQL and how is it different from REST?
2. What are the advantages of GraphQL (single endpoint, client-specified data, type safety)?
3. What are Queries, Mutations, and Subscriptions?
4. What is a GraphQL Schema?

## NestJS GraphQL Setup

5. What are the two approaches in NestJS: Code First vs Schema First?
6. What is the difference between Code First and Schema First?
7. How do you install GraphQL dependencies (`@nestjs/graphql`, `@nestjs/apollo`)?
8. How do you configure `GraphQLModule` using `GraphQLModule.forRoot()`?

## Code First Approach

9. What is Code First approach?
10. How do you define GraphQL types using TypeScript classes?
11. What is `@ObjectType()` decorator?
12. What is `@Field()` decorator?
13. How do you define nullable fields?
14. What is `@InputType()` for input types?

## Resolvers

15. What is a Resolver in GraphQL?
16. What is `@Resolver()` decorator?
17. How do you create query resolvers using `@Query()`?
18. How do you create mutation resolvers using `@Mutation()`?
19. How do you define resolver return types?
20. How do you access arguments using `@Args()`?

## Queries

21. How do you implement a simple Query?
22. How do you pass arguments to queries?
23. How do you implement pagination in queries?
24. How do you implement filtering and sorting?

## Mutations

25. How do you implement a Mutation for creating data?
26. How do you implement a Mutation for updating data?
27. How do you implement a Mutation for deleting data?
28. How do you handle input validation in mutations?

## Subscriptions

29. What are GraphQL Subscriptions?
30. How do you implement real-time updates using Subscriptions?
31. What is `@Subscription()` decorator?
32. How do you use PubSub for subscriptions?
33. What is the difference between WebSockets and Subscriptions?

## Relations & Data Loading

34. How do you define relationships between types?
35. What is the N+1 problem in GraphQL?
36. What is DataLoader and how does it solve N+1 problem?
37. How do you implement `@ResolveField()` for field resolvers?

## Validation

38. How do you validate input in GraphQL?
39. How do you use class-validator with GraphQL inputs?
40. How do you use ValidationPipe with GraphQL?

## Authentication & Authorization

41. How do you implement authentication in GraphQL?
42. How do you use Guards with GraphQL resolvers?
43. How do you implement role-based authorization?
44. How do you access current user in resolvers using custom decorators?

## Error Handling

45. How do you handle errors in GraphQL?
46. What is the error format in GraphQL responses?
47. How do you throw custom errors?

## GraphQL Playground

48. What is GraphQL Playground?
49. How do you enable/disable Playground in production?
50. How do you test queries using Playground?

# NestJS Microservices - Top Interview Questions

## Microservices Fundamentals

1. What are Microservices?
2. What is the difference between Monolith and Microservices architecture?
3. What transport layers does NestJS support (TCP, Redis, MQTT, NATS, RabbitMQ, Kafka, gRPC)?
4. What is `@nestjs/microservices` package?

## Creating Microservices

5. How do you create a microservice application?
6. What is `createMicroservice()` method?
7. How do you configure transport options?
8. Can an application be both HTTP and Microservice?

## Message Patterns

9. What are Message Patterns?
10. What is `@MessagePattern()` decorator?
11. What is `@EventPattern()` decorator?
12. What is the difference between `@MessagePattern()` and `@EventPattern()`?

## Communication

13. How do services communicate in microservices?
14. What is request-response pattern?
15. What is event-based pattern?
16. What is `ClientProxy` and how do you use it?
17. How do you inject `ClientProxy` using `@Inject()`?

## Transports

18. What is TCP transport and when do you use it?
19. What is Redis transport and when do you use it?
20. What is RabbitMQ (AMQP) and when do you use it?
21. What is Kafka and when do you use it?
22. What is gRPC and what are its advantages?
23. What is MQTT transport used for?

## Request-Response

24. How do you send messages using `client.send()`?
25. How do you handle responses from microservices?
26. What is the return type of `send()` method (Observable)?
27. How do you handle errors in request-response?

## Event-Based Communication

28. How do you emit events using `client.emit()`?
29. What is the difference between `send()` and `emit()`?
30. Does `emit()` wait for a response?

## gRPC

31. What is gRPC and why use it over HTTP?
32. How do you define `.proto` files?
33. How do you implement gRPC services in NestJS?
34. What are the advantages of gRPC (type safety, performance)?

## RabbitMQ

35. How do you integrate RabbitMQ with NestJS?
36. What is a queue in RabbitMQ?
37. What is an exchange in RabbitMQ?

## Service Discovery

38. What is service discovery and why is it needed?
39. How do you implement service discovery in microservices?

## Error Handling

40. How do you handle errors in microservices?
41. What is `RpcException`?
42. How do you throw exceptions from microservices?

## Testing Microservices

43. How do you test microservices?
44. How do you mock `ClientProxy` in tests?

## Best Practices

45. When should you use microservices vs monolith?
46. How do you handle distributed transactions?
47. What is eventual consistency?
48. How do you implement inter-service authentication?
49. How do you handle versioning in microservices?
50. What are the challenges of microservices (complexity, debugging, data consistency)?

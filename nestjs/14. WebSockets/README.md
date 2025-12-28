# NestJS WebSockets - Top Interview Questions

## WebSocket Fundamentals

1. What are WebSockets and how do they work?
2. What is the difference between WebSockets and HTTP?
3. When should you use WebSockets vs HTTP?
4. What is Socket.IO?
5. What is the difference between Socket.IO and native WebSockets?

## NestJS WebSocket Setup

6. How do you install WebSocket dependencies (`@nestjs/websockets`, `@nestjs/platform-socket.io`)?
7. What is a WebSocket Gateway in NestJS?
8. What is `@WebSocketGateway()` decorator?
9. How do you configure WebSocket gateway options (port, namespace, cors)?

## Creating Gateways

10. How do you create a WebSocket Gateway?
11. What interfaces should a Gateway implement (`OnGatewayInit`, `OnGatewayConnection`, `OnGatewayDisconnect`)?
12. What is the `handleConnection()` lifecycle hook?
13. What is the `handleDisconnect()` lifecycle hook?

## Event Handling

14. How do you listen to client events using `@SubscribeMessage()`?
15. How do you emit events to clients using `@WebSocketServer()` and `server.emit()`?
16. What is the difference between `emit()`, `broadcast()`, and `to()` methods?
17. How do you send messages to a specific client?
18. How do you broadcast to all clients except sender?

## Rooms & Namespaces

19. What are Rooms in Socket.IO?
20. How do you join a room using `socket.join()`?
21. How do you leave a room using `socket.leave()`?
22. How do you emit to a specific room?
23. What are Namespaces in Socket.IO?
24. What is the difference between Rooms and Namespaces?

## Authentication

25. How do you implement authentication in WebSockets?
26. How do you access JWT tokens in WebSocket connections?
27. How do you use Guards with WebSocket Gateways?
28. How do you reject unauthorized connections?

## Message Format

29. What is the format of WebSocket messages?
30. How do you handle acknowledgments (callbacks)?
31. How do you send and receive JSON data?

## Error Handling

32. How do you handle errors in WebSocket connections?
33. What is `WsException`?
34. How do you use Exception Filters with WebSockets?

## Real-Time Use Cases

35. How do you implement a chat application?
36. How do you implement real-time notifications?
37. How do you implement presence tracking (online/offline status)?
38. How do you implement typing indicators?

## Performance & Scalability

39. How do you scale WebSocket connections across multiple servers?
40. What is a Redis Adapter for Socket.IO?
41. Why do you need Redis for multi-server WebSocket deployments?

## Testing

42. How do you test WebSocket Gateways?
43. How do you mock WebSocket clients in tests?

## Best Practices

44. When should you use WebSockets vs Server-Sent Events (SSE)?
45. How do you handle reconnection logic on the client side?

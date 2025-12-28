# NestJS Authentication & Authorization - Top Interview Questions

## Fundamentals

1. What is the difference between Authentication and Authorization?
2. What is Passport and how does it integrate with NestJS?
3. What are the common authentication strategies (JWT, Local, OAuth)?

## JWT Authentication

4. What is JWT (JSON Web Token)?
5. What are the three parts of a JWT (header, payload, signature)?
6. How do you install and configure `JwtModule`?
7. How do you generate and sign JWT tokens using `JwtService`?
8. What is a JWT secret and expiration time?
9. What is the difference between access tokens and refresh tokens?

## Local Strategy (Username/Password)

10. How do you implement Local Strategy for login?
11. What is the `validate()` method in Passport strategies?
12. How do you validate username and password?

## JWT Strategy

13. How do you create a `JwtStrategy` class?
14. How do you extract JWT from Authorization header using `ExtractJwt.fromAuthHeaderAsBearerToken()`?
15. How do you verify JWT tokens in the strategy?
16. What should the `validate()` method return?

## Guards

17. How do you create a JWT Auth Guard using `AuthGuard('jwt')`?
18. How do you apply Auth Guard to routes and controllers?
19. How do you make certain routes public using custom decorator?

## Login/Register Flow

20. How do you implement user registration?
21. How do you hash passwords using bcrypt?
22. How do you implement login endpoint and return JWT?

## Role-Based Access Control (RBAC)

23. What is RBAC?
24. How do you implement role-based authorization?
25. How do you create a `@Roles()` decorator?
26. How do you create a Roles Guard using Reflector?

## Refresh Tokens

27. What are refresh tokens and why are they needed?
28. How do you implement refresh token flow?
29. Where should you store refresh tokens?

## OAuth 2.0

30. What is OAuth 2.0?
31. How do you implement Google OAuth Strategy?

## Password Management

32. How do you implement password reset functionality?
33. How do you implement change password?

## Security Best Practices

34. Should you store JWT in cookies or headers?
35. What is httpOnly and secure cookie?
36. Should you include sensitive data in JWT payload?
37. How do you implement logout?
38. What is token blacklisting?
39. How do you prevent brute force attacks?

## User Context

40. How do you access current user in controllers using custom decorator?
41. How do you attach user to request in strategy?

## Common Issues

42. How do you handle token expiration?
43. How do you handle CORS with authentication?
44. What causes "Unauthorized" errors and how to debug?

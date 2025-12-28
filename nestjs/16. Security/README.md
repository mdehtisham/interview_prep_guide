# NestJS Security Best Practices - Top Interview Questions

## Security Fundamentals

1. What are the OWASP Top 10 security risks?
2. What security considerations are important for APIs?
3. What is the principle of least privilege?

## Authentication & Authorization

4. How do you implement secure authentication?
5. How do you hash passwords securely using bcrypt?
6. What is the difference between bcrypt, argon2, and scrypt?
7. How do you securely store JWT secrets?
8. Should JWT tokens be stored in localStorage or cookies?
9. What is httpOnly cookie and why is it secure?
10. What is the secure flag in cookies?
11. What is SameSite attribute in cookies?

## CORS (Cross-Origin Resource Sharing)

12. What is CORS and why is it important?
13. How do you enable CORS in NestJS?
14. How do you configure CORS options (origin, credentials, methods)?
15. What are the security implications of allowing all origins?

## Helmet

16. What is Helmet middleware?
17. How do you use Helmet in NestJS?
18. What HTTP headers does Helmet set (CSP, HSTS, X-Frame-Options)?
19. What is Content Security Policy (CSP)?
20. What is HTTP Strict Transport Security (HSTS)?

## Rate Limiting

21. Why is rate limiting important for security?
22. How do you implement rate limiting using `@nestjs/throttler`?
23. How do you prevent brute force attacks on login endpoints?
24. How do you implement API rate limiting per user/IP?

## Input Validation

25. Why is input validation critical for security?
26. How do you use ValidationPipe for input validation?
27. What is `class-validator` and how does it prevent injection attacks?
28. How do you sanitize user inputs?
29. What is the `whitelist` option in ValidationPipe?
30. What is `forbidNonWhitelisted` option?

## SQL Injection Prevention

31. What is SQL injection?
32. How does TypeORM prevent SQL injection?
33. Should you use raw queries or ORM methods?
34. How do you parameterize queries?

## XSS (Cross-Site Scripting) Prevention

35. What is XSS attack?
36. How do you prevent XSS in NestJS?
37. Should you escape user-generated content?
38. How do you sanitize HTML inputs?

## CSRF (Cross-Site Request Forgery) Prevention

39. What is CSRF attack?
40. How do you prevent CSRF attacks?
41. What is the CSRF token pattern?
42. How do you implement CSRF protection using `csurf` middleware?

## Data Encryption

43. What is encryption at rest vs encryption in transit?
44. How do you use HTTPS/SSL in production?
45. Should you encrypt sensitive data in the database?
46. How do you handle encryption keys?

## Environment Variables & Secrets

47. How do you securely manage environment variables?
48. Should you commit `.env` files to version control?
49. How do you use secret management tools (AWS Secrets Manager, HashiCorp Vault)?
50. What is the principle of keeping secrets out of code?

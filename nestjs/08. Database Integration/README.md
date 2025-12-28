# NestJS Database Integration - Top Interview Questions

## Database Fundamentals

1. What are the main database integration options in NestJS (TypeORM, Mongoose, Prisma, Sequelize)?
2. What is TypeORM and when do you use it?
3. What is Mongoose and when do you use it?
4. What databases does NestJS support?

## TypeORM Setup

5. How do you install and configure TypeORM in NestJS?
6. What is `TypeOrmModule` and how do you use it?
7. How do you configure database connection using `TypeOrmModule.forRoot()`?
8. How do you use environment variables for database configuration?

## Entities

9. What is an Entity in TypeORM?
10. What are the essential decorators: `@Entity()`, `@PrimaryGeneratedColumn()`, `@Column()`?
11. How do you define relationships: `@OneToMany()`, `@ManyToOne()`, `@ManyToMany()`, `@OneToOne()`?
12. What is cascade and eager loading in relationships?

## Repository Pattern

13. What is the Repository Pattern in TypeORM?
14. What is `TypeOrmModule.forFeature()` used for?
15. How do you inject a repository using `@InjectRepository()`?

## CRUD Operations

16. How do you create records using `save()` vs `insert()`?
17. How do you read records using `find()`, `findOne()`, `findOneBy()`?
18. How do you update records using `update()`?
19. How do you delete records using `delete()` vs `remove()`?

## Querying

20. How do you use `find()` with options (where, select, relations, order)?
21. How do you implement pagination using `skip` and `take`?
22. What is `findAndCount()` used for?

## Query Builder

23. What is QueryBuilder and when should you use it?
24. How do you use QueryBuilder for complex queries with joins?

## Transactions

25. What are transactions and why are they important?
26. How do you implement transactions in TypeORM using `QueryRunner`?
27. How do you use `@Transaction()` decorator?

## Migrations

28. What are migrations and why do you need them?
29. How do you generate and run migrations in TypeORM?

## Mongoose Integration

30. How do you integrate Mongoose with NestJS?
31. What is `MongooseModule` and how do you configure it?
32. What is `@Schema()` and `@Prop()` decorator?
33. How do you inject a Mongoose model using `@InjectModel()`?
34. How do you perform CRUD operations in Mongoose?
35. What is population in Mongoose?

## Multiple Databases

36. How do you configure multiple database connections?

## Performance

37. What are N+1 query problems and how do you prevent them?
38. What is the difference between eager and lazy loading?
39. How do you optimize database queries?

## Testing

40. How do you mock repositories in unit tests?
41. What is `TypeOrmModule.forRootAsync()` used for?

## Best Practices

42. Should business logic be in services or repositories?
43. How do you handle database errors?
44. How do you secure database credentials using environment variables?
45. How do you enable query logging in TypeORM?

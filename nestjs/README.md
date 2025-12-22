# Top NestJS Interview Questions

## Core Concepts & Architecture
1. What is NestJS and why is it built on top of Express/Fastify?
**Answer:**
NestJS is a progressive Node.js framework for building efficient, reliable, and scalable server-side applications. It solves a major problem in the Node.js ecosystem: **Architecture**. While Node.js is powerful, it doesn't provide a structure, often leading to "Spaghetti Code" in large projects.

NestJS provides an out-of-the-box architecture inspired by Angular (Modules, Providers, Controllers) which enforces separation of concerns and maintainability.

It is built on top of HTTP Server frameworks like **Express** (default) or **Fastify** to leverage their robust, battle-tested low-level capabilities (request/response handling) while providing a higher-level abstraction.
- **Express:** Mature, huge ecosystem, easy to use (Default).
- **Fastify:** High performance, low overhead (Optional).
This abstraction allows developers to switch underlying platforms without rewriting their business logic.

2. Explain the main architectural pattern NestJS follows.
**Answer:**
NestJS heavily relies on the **Modular Automation** system and the **Dependency Injection (DI)** pattern.

- **Modules:** The application is a graph of modules. The root module is the starting point. Modules group related components (Controllers + Services) together.
- **Controllers:** Handle incoming HTTP requests and delegate work to services.
- **Services (Providers):** Contain business logic and are injected into controllers.
- **Dependency Injection:** NestJS manages the lifecycle of classes. instead of manually creating instances (`new UserService()`), you ask Nest to provide it via the constructor. This makes testing and swapping implementations (e.g., MockService vs RealService) incredibly easy.

3. What is the difference between specific platform (e.g., `NestExpressApplication`) and platform-agnostic frameworks?
**Answer:**
NestJS is platform-agnostic by default. This means your controllers don't strictly return Express-specific objects.

- **Platform-Agnostic (Default):**
  You use standard NestJS decorators (`@Req()`, `@Res()`, `@Body()`). Nest translates these to the underlying driver's calls.
  *Benefit:* You can switch from Express to Fastify easily.
  
- **Platform-Specific (`NestExpressApplication`):**
  Sometimes you need features specific to Express (e.g., `app.set('trust proxy')` or specific template engines).
  By injecting the platform specific app interface:
  ```typescript
  const app = await NestFactory.create<NestExpressApplication>(AppModule);
  ```
  You gain access to Express-only methods. The downside is that your code is now tightly coupled to Express.

4. How does NestJS leverage TypeScript?
**Answer:**
Unlike many Node.js frameworks that tacked on TypeScript later, NestJS is built with **TypeScript first**.
- **Decorators:** It uses decorators heavily (`@Controller`, `@Get`, `@Injectable`) to provide declarative metadata. This drastically reduces boilerplate code.
- **Type Safety:** It ensures your dependency injection tokens, DTOs, and return types are checked at compile time, preventing a whole class of runtime errors.
- **Reflection:** It uses TypeScript's `emitDecoratorMetadata` to automatically resolve dependencies. For example, `constructor(private service: UserService)` tells Nest that this class needs `UserService` without you having to manually register a token in many cases.

5. What is the root module and why is it required?
**Answer:**
The **Root Module** (`AppModule` usually) is the entry point of the application graph. NestJS uses this module to build its internal "Dependency Graph".
- When the application starts (`NestFactory.create(AppModule)`), Nest looks at the `imports` of the root module.
- It traverses down the tree (Root -> UsersModule -> AuthModule...) to resolve all dependencies and instantiate providers in the correct order.
- Without a root module, Nest wouldn't know where to begin scanning for controllers and providers, effectively meaning the application has no structure.
6. Explain the concept of Dependency Injection in NestJS.
**Answer:**
Dependency Injection (DI) is an **Inversion of Control (IoC)** technique where you delegate the responsibility of instantiating dependencies to the framework (the IoC Container) instead of creating them manually.

In NestJS:
1.  **Define:** You mark a class as a "provider" using the `@Injectable()` decorator.
2.  **Register:** You add the class to the `providers` array in a `@Module`.
3.  **Inject:** You request the dependency in a class constructor (e.g., `constructor(private authService: AuthService) {}`).
4.  **Resolve:** At runtime, NestJS detects this dependency, checks its internal container for an existing instance (Singleton by default), creates one if needed, and passes it to the constructor.
*Benefit:* This decouples your code. Your Controller doesn't know *how* to create the Service, it just knows it *needs* one. This is crucial for unit testing (swapping real services for mocks).

7. How does the NestJS Command Line Interface (CLI) help in development?
**Answer:**
The CLI is a powerful productivity tool that automates project lifecycle tasks. Key features include:

-   **Scaffolding (`nest new`):** Generates a best-practice project structure.
-   **Generators (`nest g`):** Instantly creates headers files and automatically updates module imports.
    -   `nest g module users`: Creates a module.
    -   `nest g controller users`: Creates a controller and imports it into the UsersModule.
    -   `nest g resource users`: **The best command.** Creates a complete CRUD feature (Module, Controller, Service, DTOs, Entities, Tests) in one go.
-   **Execution (`nest start`):** Runs the application, supports "Watch Mode" (`--watch`) for hot-reload during development.
-   **Building (`nest build`):** Compiles the project to JavaScript (`dist` folder) for production deployment.

8. What is the difference between a standalone application and a standard NestJS application?
**Answer:**
-   **Standard Application:**
    -   Created via `NestFactory.create(AppModule)`.
    -   Starts a **Network Listener** (HTTP Server like Express or Fastify).
    -   Used for: REST APIs, GraphQL, Microservices, WebSockets.
    -   It stays alive indefinitely handling requests.

-   **Standalone Application:**
    -   Created via `NestFactory.createApplicationContext(AppModule)`.
    -   **Does NOT** start a network listener. It only initializes the DI container and modules.
    -   Used for: **CRON Jobs**, **CLI Scripts**, **Worker Processes**, or one-off tasks (e.g., a database migration script).
    -   It typically runs a specific task and then exits.

9. How does NestJS handle environment variables?
**Answer:**
NestJS provides a dedicated module `@nestjs/config` (typically wrapping the popular `dotenv` library) to handle configuration securely.

**Best Practice Workflow:**
1.  Variable definition in `.env` files (e.g., `DATABASE_HOST=localhost`).
2.  Import `ConfigModule.forRoot()` in the root `AppModule`.
3.  **Validation:** Use `Joi` inside the ConfigModule to strictly validate that required variables exist on startup (e.g., throw error if `DB_PASS` is missing).
4.  **Usage:** Inject `ConfigService` into classes to access values:
    ```typescript
    constructor(private configService: ConfigService) {}
    
    getDbUser() {
      return this.configService.get<string>('DATABASE_USER');
    }
    ```
This abstracts the source of the configuration (env file vs system env vars) and provides type safety.

10. What are decorators in NestJS and name a few common ones?
**Answer:**
Decorators are a TypeScript language feature that allows you to attach **metadata** to classes, methods, or properties. NestJS uses this metadata to organize the application and route requests.

Without decorators, you would have to manually configure routes and wiring in a central file (like in old Express apps). Decorators make this declarative.

**Common Decorators:**
-   **Class Level:** `@Module()`, `@Controller('path')`, `@Injectable()`.
-   **Method Level:** `@Get()`, `@Post()`, `@UseGuards()`, `@UseInterceptors()`.
-   **Param Level:** `@Body()`, `@Param()`, `@Query()`, `@Req()`, `@Headers()`.

## Modules
11. What is the purpose of a Module (`@Module`) in NestJS?
**Answer:**
A Module is a class annotated with the `@Module()` decorator. It serves several critical purposes:
-   **Organization:** It groups related controllers, services, and other providers into cohesive blocks (e.g., `UsersModule`, `AuthModule`).
-   **Compilation Context:** It tells NestJS how to compile dependencies. Nest uses modules to build the application graph.
-   **Encapsulation:** By default, providers defined in a module are **private**. They are not visible to other modules unless explicitly exported. This enforces a strict public API for every feature.

12. What are the four main properties of the `@Module` decorator?
**Answer:**
The `@Module()` decorator accepts a single object with these properties:
1.  **`providers`:** The services, repositories, factories, helpers, etc., that will be instantiated by the Nest injector and is shared at least across this module.
2.  **`controllers`:** The set of controllers defined in this module which have to be instantiated.
3.  **`imports`:** The list of *other* modules that export the providers which are required in this module.
4.  **`exports`:** The subset of `providers` that are provided by this module and should be available in other modules which import this module.

13. How do you share a service between two different modules?
**Answer:**
To make `ServiceA` (from `ModuleA`) available in `ModuleB`, you must follow two steps:

1.  **Export from Source:** In `ModuleA`, add `ServiceA` to the `exports` array.
    ```typescript
    @Module({
      providers: [ServiceA],
      exports: [ServiceA] // <--- Crucial step
    })
    export class ModuleA {}
    ```
2.  **Import in Target:** In `ModuleB`, add `ModuleA` to the `imports` array.
    ```typescript
    @Module({
      imports: [ModuleA], // <--- Now ModuleB has access to exported providers of ModuleA
    })
    export class ModuleB {}
    ```
Now you can inject `ServiceA` into any controller or provider within `ModuleB`.

14. What are Global Modules (`@Global`)? When should you use them?
**Answer:**
A Global Module is a module decorated with `@Global()`.
-   **Behavior:** You only need to import it **once** (usually in the root `AppModule`). Its exported providers are then available **everywhere** in the application without needing to import the module in other feature modules.
-   **Use Case:** Cross-cutting concerns like Database connections (`TypeOrmModule`), Configuration (`ConfigModule`), or generic Helpers that are used in almost every file.
-   **Warning:** Making everything global is considered a bad practice ("Global State" anti-pattern). It makes dependencies opaque and harder to track. Prefer explicit imports for most feature modules.

15. What are Dynamic Modules? How are they different from Static Modules?
**Answer:**
-   **Static Modules:** Their configuration is fixed at compile time. You just `@Module({})` and that's it.
-   **Dynamic Modules:** These are modules that can create providers **dynamically** based on runtime configuration. They are just functions that *return* a module definition.
    
    *Common Example:* `TypeOrmModule.forRoot({...})`. You pass the DB password and host *into* the module. The module then uses these values to create the DB connection provider.
    
    They typically implement a static method conventionally named `forRoot`, `register`, or `traverse` that returns a `DynamicModule` object.
16. Explain the `forRoot`, `register`, and `forFeature` patterns in Dynamic Modules.
**Answer:**
These are naming conventions (not hard rules) for configuring Dynamic Modules:
-   **`forRoot`**: Used when configuring a module **once** for the entire application (Singleton). Usually defines the database connection, global configuration, etc.
    *   *Example:* `MongooseModule.forRoot('mongodb://localhost/test')`
-   **`register`**: Used when you might want to configure the module **multiple times** with different settings in different modules.
    *   *Example:* `HttpModule.register({ timeout: 5000 })` in ModuleA and `HttpModule.register({ timeout: 1000 })` in ModuleB.
-   **`forFeature`**: Used to import strictly the "features" (Entities, Repositories) relevant to the current module scope, using the connection established by `forRoot`.
    *   *Example:* `TypeOrmModule.forFeature([UserEntity])` creates a `UserRepository` just for the module importing it.

17. How do you resolve Circular Dependencies between modules?
**Answer:**
Circular dependency occurs when Module A needs Module B, and Module B needs Module A. NestJS cannot resolve the graph order.
**Solution:** Use `forwardRef`.
1.  **In Module Imports:**
    ```typescript
    // In Module A
    imports: [forwardRef(() => ModuleB)]

    // In Module B
    imports: [forwardRef(() => ModuleA)]
    ```
2.  **In Services (if needed):** If the services themselves also depend on each other:
    ```typescript
    constructor(
      @Inject(forwardRef(() => ServiceB))
      private serviceB: ServiceB
    ) {}
    ```

18. Can a module import itself?
**Answer:**
**No.** A module importing itself would create an immediate infinite recursion (stack overflow) during the dependency resolution phase. The application will crash on startup.

19. What is the difference between `exports` in a module and `public` in a class?
**Answer:**
-   **`exports` (NestJS):** Controls **Dependency Injection Visibility**. If a provider is not in `exports`, another module *cannot* inject it, even if it imports the module. It effectively makes the provider "private" to the module's scope.
-   **`public` (TypeScript):** Controls **Class Member Access**. A `public` method can be called by any other class in code. However, even if a service class is `public`, if it is not exported by its Module, NestJS will throw an error when another module tries to ask the DI container for it.

## Controllers & Routing
20. What is the role of a Controller?
**Answer:**
The Controller's single responsibility is **handling incoming requests and returning responses** to the client.
-   It acts as the **entry point** for the application.
-   It inspects the request (URL, Query, Body, Headers).
-   It delegates the actual "work" (Business Logic) to a **Service**.
-   **Production Rule:** A Controller should be "Thin". It should never contain complex logic, database queries, or calculations. It should simply orchestrate the input/output.
21. How do you define a route in NestJS?
**Answer:**
You define a route by decorating a controller method with an HTTP verb decorator (`@Get`, `@Post`, `@Put`, `@Delete`, etc.). The path is a concatenation of the Controller's prefix and the Method's path.

```typescript
@Controller('users') // Prefix: /users
export class UsersController {
  @Get('profile') // Path: /users/profile
  getProfile() { ... }
}
```

22. Explain the usage of `@Body()`, `@Param()`, and `@Query()` decorators.
**Answer:**
These decorators extract data from the incoming request object:
-   **`@Body()`**: Extracts the entire **JSON body** (usually for POST/PUT).
    -   `create(@Body() dto: CreateUserDto)`
-   **`@Param('id')`**: Extracts a specific **route parameter** (variables in the URL path).
    -   Route: `users/:id` -> `findOne(@Param('id') id: string)`
-   **`@Query('search')`**: Extracts **query string parameters** (after the `?` in URL).
    -   URL: `/users?search=john` -> `findAll(@Query('search') search: string)`

23. How do you handle HTTP status codes in a controller?
**Answer:**
By default, NestJS returns **200 OK** (mostly) or **201 Created** (for POST).
To customize this:
1.  **Static:** Use the `@HttpCode()` decorator.
    ```typescript
    @Post()
    @HttpCode(204)
    create() { ... }
    ```
2.  **Dynamic/Error:** Throw an exception (e.g., `throw new NotFoundException()`) which Nest automatically translates to a 404 response.

24. How can you access the underlying request object (Express/Fastify) directly?
**Answer:**
You can inject the library-specific request object using the `@Req()` decorator (or `@Request()`).
```typescript
@Get()
findAll(@Req() request: Request) {
  console.log(request.headers); // Direct access
}
```
**Warning:** Doing this makes your controller **platform-specific** (harder to switch from Express to Fastify later) and harder to test. Avoid this if possible; use standard decorators (`@Body`, `@Headers`, `@Ip`) instead.

25. How do you implement a wildcard route?
**Answer:**
NestJS supports pattern based routing. The `*` character is used as a wildcard.
```typescript
@Get('ab*cd')
findAll() {
  return 'This route matches abcd, ab_cd, abRANDOMcd';
}
```
*Note:* The order of methods matters! Define specific routes *before* wildcard routes, otherwise the wildcard might intercept requests intended for other paths.
26. What is the difference between `@Res()` and returning a value directly from a controller method?
**Answer:**
-   **Standard (Returning value):** Recommended. You return an object/string/array, and NestJS automatically serializes it to JSON and sends it with a 200 OK. It effectively handles the response lifecycle for you.
-   **Library-Specific (`@Res()`):** You inject the response object (e.g., Express `res`).
    -   **Manual Mode:** Nest puts you in "Manual Mode". You act as if you are writing raw Express code.
    -   **Responsibility:** You **MUST** send the response yourself (`res.json(...)`, `res.send(...)`). If you forget, your server hangs indefinitely.
    -   *Use Case:* Streaming files, manipulating cookies manually, or special headers that Nest decorators don't cover.

27. How does NestJS handle content-type negotiation?
**Answer:**
Out of the box, NestJS defaults to `application/json`.
-   If you return a JS Object/Array -> sent as JSON.
-   If you return a String -> sent as `text/html`.
-   You can force a specific type using the `@Header()` decorator:
    ```typescript
    @Get()
    @Header('Content-Type', 'application/xml')
    getXml() { ... }
    ```

28. How do you perform a redirect from a controller?
**Answer:**
1.  **Static Redirect:** Use the `@Redirect()` decorator.
    ```typescript
    @Get()
    @Redirect('https://nestjs.com', 301)
    ```
2.  **Dynamic Redirect:** Return an object with `url` and `statusCode` properties from your method (this overrides the decorator params).
    ```typescript
    @Get('docs')
    @Redirect('https://docs.nestjs.com', 302)
    getDocs(@Query('version') version) {
        if (version === '5') {
            return { url: 'https://docs.nestjs.com/v5/' };
        }
    }
    ```

29. Explain how to manage headers in a response.
**Answer:**
1.  **Static Headers:** Use `@Header('Cache-Control', 'none')`.
2.  **Dynamic Headers:** You must switch to Library-Specific mode by injecting `@Res({ passthrough: true })`.
    -   **`passthrough: true`**: This is a key trick. It gives you the `res` object to set headers/cookies, *but* lets NestJS still handle the final JSON serialization and sending.
    ```typescript
    @Get()
    findAll(@Res({ passthrough: true }) res: Response) {
        res.header('X-Custom', '123');
        return { data: 'Nest still handles the body' };
    }
    ```

## Providers & Services
30. What is a Provider in NestJS?
**Answer:**
A provider is **any class that can be injected** as a dependency.
In NestJS, almost everything is a provider: Services, Repositories, Factories, Helpers.
-   Technically, it is a class decorated with `@Injectable()`.
-   Providers are registered in the `providers` array of a Module.
-   They are managed by the NestJS IoC (Inversion of Control) container.
31. What is the default scope of a provider in NestJS?
**Answer:**
**Singleton.**
By default, Nest.js creates only **one instance** of each provider and shares it across the entire application. This means the constructor of the service is called only once during bootstrap.

32. Explain the different scopes available (Singleton, Request, Transient).
**Answer:**
1.  **DEFAULT (Singleton):** One instance serves the whole app. Best for performance. State is shared (careful!).
2.  **REQUEST:** A new instance is created for **every incoming HTTP request**. Use this if you need to store request-specific data (e.g., Request ID, Authenticated User) inside the service class itself. *Warning:* High memory usage, slower garbage collection.
3.  **TRANSIENT:** A new instance is created **every time it is injected**. If ServiceA injects Helper, and ServiceB injects Helper, they get two different Helper instances.

33. How do you create a custom provider?
**Answer:**
You register it in the `@Module` using the verbose syntax:
```typescript
providers: [
  {
    provide: 'CONNECTION', // 1. The Token (string or class)
    useValue: customConnection // 2. The Value
  }
]
```
You then inject it using `@Inject('CONNECTION')`.

34. What is the difference between `useValue`, `useClass`, and `useFactory`?
**Answer:**
-   **`useValue`**: Injecs a constant value (string, number, object instance). Useful for mock objects in testing or config constants.
-   **`useClass`**: Allows you to swap the implementation class.
    *   *Example:* `provide: LoggerService, useClass: ProductionLoggerService`. (Use `MockLogger` in tests).
-   **`useFactory`**: Most powerful. Creates a provider dynamically. Can accept arguments (other providers) and run logic before returning the instance.
    *   *Example:* Reading a DB password from ConfigService before connecting.

35. What is `Optional()` injection?
**Answer:**
Sometimes, a dependency might not be available (e.g., a logging service that is only configured in Production).
Using `@Optional()` prevents Nest from crashing if the provider is missing. It just injects `undefined` instead.
```typescript
constructor(@Optional() private logger: LoggerService) {}
```
36. How does property-based injection work?
**Answer:**
Standard injection uses the **constructor**. Property-based injection allows you to inject directly into a class property using the `@Inject()` decorator.
```typescript
@Injectable()
export class HttpService {
  @Inject('CONFIG')
  private config: ConfigType;
}
```
**Use Case:** It is strictly recommended to use **Constructor Injection** for everything. Property injection is generally used only when extending a parent class (`super()`) to avoid manually passing dozens of dependencies from the Child to the Parent constructor.

37. Can you inject a provider into a standard class that is not decorated?
**Answer:**
**No.** The class must belong to the Dependency Graph (Managed by Nest IoC). Standard JS/TS classes (`new MyClass()`) are invisible to Nest.
*Exception:* You can pass a provider instance *into* a standard class manually, but the standard class itself cannot "ask" Nest for dependencies.

## Request Lifecycle (Middleware, Guards, Interceptors, Pipes, Filters)
38. Explain the execution order of Middleware, Guards, Interceptors, Pipes, and Exception Filters.
**Answer:**
This is the most critical interview question. The order is:
1.  **Middleware** (Express/Fastify level: Logging, Cors, Helmet)
2.  **Guards** (Authorization: Can this user proceed?)
3.  **Interceptors (Pre-Controller)** (caching, logging start time)
4.  **Pipes** (Validation/Transformation of input data)
5.  **Controller Method** (Business Logic)
6.  **Interceptors (Post-Controller)** (Transforming the response, Mapping data)
7.  **Exception Filters** (Catching errors if any occurred in steps 2-6)

39. What is Middleware in NestJS and how is it different from Express middleware?
**Answer:**
-   **Nest Middleware** is essentially the same as **Express Middleware**. It is a function that has access to `req`, `res`, and `next()`.
-   It runs **before** any NestJS context (Guards/Interceptors).
-   It is configured in the module's `configure()` method using `Unrestricted` syntax (Nest doesn't strictly control it via decorators).
-   *Difference:* Nest middleware can support Dependency Injection (you can inject services into a middleware class), whereas raw Express middleware cannot easily do this.

40. How do you implement a functional middleware?
**Answer:**
Middleware doesn't have to be a class. It can be a simple function (lighter, less boilerplate).
```typescript
export function logger(req, res, next) {
  console.log(`Request...`);
  next();
};

// Usage in Module
consumer.apply(logger).forRoutes('cats');
```
41. What is the primary purpose of a Guard?
**Answer:**
**Authorization.**
Guards have a single responsibility: To determine if the request should proceed or be stopped.
They implement the `CanActivate` interface and must return `true` (allow) or `false` (deny, throws 403 Forbidden).
*Why not Middleware?* Guards have access to the `ExecutionContext` (they know which controller method will be called next), whereas Middleware is dumb and only knows URL/Method.

42. How does an `AuthGuard` work?
**Answer:**
An `AuthGuard` (usually from `@nestjs/passport`) connects the strategy pattern to the Guard.
1.  It extracts the token (JWT) from the header.
2.  It validates the token (signature, expiration).
3.  If valid, it appends the user to the request object (`req.user = payload`).
4.  It returns `true`.
5.  If invalid, it throws `UnauthorizedException` (401).

43. What is an Interceptor? Give a common use case.
**Answer:**
Interceptors, inspired by Aspect Oriented Programming (AOP), wrap the execution stream. They can run extra logic **before** the method runs and **after** the method returns.
**Common Use Case: Response Mapping.**
-   Your DB returns `{ _id: 1, first: 'John', pass: 'hash' }`.
-   You want to send `{ id: 1, name: 'John' }` to the client.
-   An interceptor intercepts the `return` value and transforms it (using `class-transformer` `Exclude` logic) before the network response is sent.

44. How do you transform a response using an Interceptor?
**Answer:**
You use RxJS operators since Interceptors deal with streams (`Observable`).
```typescript
intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
  return next
    .handle()
    .pipe(
      map(data => ({ ...data, timestamp: new Date() }))
    );
}
```

45. What is a Pipe?
**Answer:**
To **Validate** or **Transform** incoming input arguments (Body, Query, Params) before they reach the controller method.
-   **Transformation:** Convert string `'123'` to number `123`.
-   **Validation:** Check if email is valid. If invalid, **Throw Error immediately**. The controller method is **never executed**.
*Why is this good?* It guarantees that inside your controller method, the `id` is definitely a Number and the `dto.email` is definitely valid, removing cluttering `if (isNaN(id))` checks from business logic.
46. What is the difference between a transformation pipe and a validation pipe?
**Answer:**
-   **Transformation Pipe:** Modifies the input. It returns a *new* value which replaces the original argument.
    -   Example: `ParseIntPipe`: Takes string `'1'` -> Returns number `1`.
-   **Validation Pipe:** Checks the input without modifying it. It returns the *same* value. If the check fails, it throws an exception.
    -   Example: Checks if the object has a valid `email` property.

47. How do you enable global validation pipes?
**Answer:**
You set it up in `main.ts` during bootstrap:
```typescript
async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  app.useGlobalPipes(new ValidationPipe({
    whitelist: true, // Strips properties that don't exist in the DTO
    forbidNonWhitelisted: true, // Throws error if extra properties exist
  }));
  await app.listen(3000);
}
```
This ensures *every* endpoint in the app is protected against bad data automatically.

48. What is an Exception Filter?
**Answer:**
The **last line of defense**.
When an exception isn't handled by your code (e.g., you `throw new Error('oops')`), it bubbles up to the Exception Layer.
-   Out of the box, Nest has a **Global Exception Filter** that catches everything and returns a friendly JSON `{ statusCode: 500, message: 'Internal Server Error' }`.
-   You typically write Custom Filters to catch specific errors (e.g., `TypeORMError` -> 409 Conflict) or to standardize error response formats.

49. How do you create a global exception filter to handle all HttpExceptions?
**Answer:**
1.  Create a class implementing `ExceptionFilter`.
2.  Decorate with `@Catch(HttpException)`.
3.  Register it globally in `main.ts` (`app.useGlobalFilters(new HttpErrorFilter())`) OR in `AppModule`.
```typescript
@Catch(HttpException)
export class HttpErrorFilter implements ExceptionFilter {
  catch(exception: HttpException, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse();
    const status = exception.getStatus();
    
    response.status(status).json({
      statusCode: status,
      timestamp: new Date().toISOString(),
      path: ctx.getRequest().url,
    });
  }
}
```

50. Can Guards modify the request object?
**Answer:**
**Yes.** This is a very common pattern (e.g., Authorization).
As seen in `AuthGuard`, we validate the token and then often attach the user payload: `req.user = user`.
This is why subsequent Guards or Controllers can access `req.user`.
*Note:* Pipes and Interceptors can also modify data, but Guards modify the Request Context itself.

## Data, Databases & DTOs
51. What is a DTO (Data Transfer Object) and why should you use it?
**Answer:**
A DTO is an object that defines how data will be sent over the network.
-   **Why use it?** It decouples your **Database Entity** from your **API Interface**. You typically don't want to expose your entire database schema (like passwords, internal flags) to the public API.
-   **Class vs Interface:** In NestJS, we use **Classes** (not Interfaces) for DTOs.
    -   *Reason:* Interfaces are removed during TypeScript compilation (runtime erasure). Classes preserve metadata, which libraries like `class-validator` and Swagger need at runtime.

52. How does `class-validator` and `class-transformer` work with NestJS pipes?
**Answer:**
They work hand-in-hand with the built-in `ValidationPipe`:
1.  **Request comes in:** Payload is just a plain JavaScript Object (JSON).
2.  **Transformation (`class-transformer`):** The Pipe uses the DTO class (from the method signature) and transforms the plain JSON into an actual **Instance** of that DTO class.
3.  **Validation (`class-validator`):** Now that it is a class instance, the decorators (`@IsString()`, `@IsEmail()`) are active. The validator runs against this instance.
4.  **Result:** If valid, the controller gets the clean object. If invalid, a 400 Bad Request is thrown.

53. How do you integrate TypeORM or Prisma with NestJS?
**Answer:**
-   **TypeORM:** NestJS has a dedicated `@nestjs/typeorm` package.
    1.  Import `TypeOrmModule.forRoot({ ...config })` in `AppModule`.
    2.  Create Entities (`@Entity()`).
    3.  Register entities in features: `TypeOrmModule.forFeature([User])`.
    4.  Inject Repository: `constructor(@InjectRepository(User) private repo: Repository<User>)`.
-   **Prisma:**
    1.  Install `prisma` and generate the client.
    2.  Create a `PrismaService` which extends `PrismaClient` and implements `OnModuleInit`.
    3.  Inject `PrismaService` heavily into any service that needs DB access.

54. What is the Repository pattern in NestJS?
**Answer:**
It is an abstraction layer between your business logic (Service) and your data source (Database).
-   **Role:** The Service should not know *how* to write SQL. It should just say `repo.save(user)`.
-   **Benefit:** Allows you to swap the underlying database logic without changing the business logic. It also makes unit testing services easy, as you can mock the Repository methods (`find`, `save`, `delete`) instead of mocking a database connection.

55. How do you handle transactions in NestJS?
**Answer:**
-   **TypeORM:** usage of `DataSource` (QueryRunner).
    ```typescript
    const queryRunner = this.dataSource.createQueryRunner();
    await queryRunner.connect();
    await queryRunner.startTransaction();
    try {
      await queryRunner.manager.save(user);
      await queryRunner.manager.save(profile);
      await queryRunner.commitTransaction();
    } catch (err) {
      await queryRunner.rollbackTransaction();
    } finally {
      await queryRunner.release();
    }
    ```
-   **Prisma:** usage of `$transaction`.
    ```typescript
    await this.prisma.$transaction([
      this.prisma.user.create({ ... }),
      this.prisma.profile.create({ ... })
    ]);
    ```
56. Explain how to perform database migrations in a NestJS project.
**Answer:**
Migrations track changes to the database schema over time (version control for DB).
1.  **TypeORM:**
    -   Setup a datasource config file.
    -   Generate: `typeorm migration:generate -d src/data-source.ts src/migrations/Init`
    -   Run: `typeorm migration:run -d src/data-source.ts`
2.  **Prisma:**
    -   Modify `schema.prisma`.
    -   Run: `npx prisma migrate dev --name init` (This generates SQL files and applies them automatically).

57. How do you handle relationships (One-to-One, Many-to-Many) in NestJS entities?
**Answer:**
You use decorators (if using TypeORM) or the schema language (if using Prisma).
**TypeORM Example (Many-to-One):**
```typescript
@Entity()
class Photo {
    @ManyToOne(() => User, (user) => user.photos)
    user: User
}

@Entity()
class User {
    @OneToMany(() => Photo, (photo) => photo.user)
    photos: Photo[]
}
```
In Prisma, this is defined in the `schema.prisma` file, and NestJS just uses the generated types.

## Authentication & Security
58. How do you implement JWT authentication in NestJS?
**Answer:**
1.  Install `@nestjs/jwt` and `@nestjs/passport`.
2.  Create `AuthService` with a `login` method that takes a user, validates password (bcrypt), and signs a token: `this.jwtService.sign(payload)`.
3.  Create `JwtStrategy` (extending `PassportStrategy`) that validates incoming tokens.
4.  Protect routes using `@UseGuards(AuthGuard('jwt'))`.

59. What is the role of Passport.js in NestJS?
**Answer:**
Passport is a general-purpose authentication middleware for Node.js.
-   NestJS doesn't reinvent authentication. It provides a wrapper `@nestjs/passport` around this ecosystem.
-   It allows you to use different "Strategies" (Local/Username-Password, JWT, OAuth/Google, Facebook) interchangeably.
-   Nest's `AuthGuard` executes these strategies.

60. How do you implement Rate Limiting (Throttling)?
**Answer:**
1.  Install `@nestjs/throttler`.
2.  Import `ThrottlerModule.forRoot({ ttl: 60, limit: 10 })` in AppModule (10 requests per 60 seconds).
3.  Apply globally via `APP_GUARD` or specifically via `@UseGuards(ThrottlerGuard)`.
4.  You can override limits per controller using `@Throttle(5, 60)`.
61. How do you enable CORS in a NestJS application?
**Answer:**
CORS (Cross-Origin Resource Sharing) is blocked by browsers by default for security.
To enable it:
```typescript
const app = await NestFactory.create(AppModule);
app.enableCors({
  origin: 'http://localhost:4200', // Allow only frontend
  methods: 'GET,POST', 
  credentials: true,
});
await app.listen(3000);
```

62. How do you protect routes based on User Roles?
**Answer:**
This requires a custom Guard and a custom Decorator.
1.  **Metadata:** Create `@Roles('admin')` decorator (using `SetMetadata`).
2.  **Guard:** Create `RolesGuard` that reflects on that metadata.
    ```typescript
    const requiredRoles = this.reflector.get<string[]>('roles', context.getHandler());
    const { user } = context.switchToHttp().getRequest();
    return requiredRoles.some((role) => user.roles?.includes(role));
    ```
3.  **Usage:** `@UseGuards(AuthGuard, RolesGuard)` `@Roles('admin')`.

63. Explain how to use Helmet for security headers.
**Answer:**
Helmet sets various HTTP headers to secure the app (e.g., XSS Protection, Content Security Policy).
1.  Install: `npm i helmet`
2.  Register in main.ts:
    ```typescript
    import helmet from 'helmet';
    // ...
    app.use(helmet());
    ```
*Note:* If using Fastify, install `@fastify/helmet`.

## Microservices & Advanced Topics
64. What transport layers does NestJS support for Microservices?
**Answer:**
NestJS is transport-agnostic. It supports many built-in transporters:
-   **TCP** (Default)
-   **Redis** (Pub/Sub)
-   **MQTT**
-   **NATS**
-   **RabbitMQ**
-   **Kafka**
-   **gRPC** (Google's high performance RPC)

65. What is the difference between Request-Response and Event-based communication?
**Answer:**
-   **Request-Response (`@MessagePattern`):**
    -   Synchronous-like behavior. Service A sends a message to Service B and **waits** for a return value.
    -   *Use Case:* Getting user details, calculating a price.
-   **Event-Based (`@EventPattern`):**
    -   Asynchronous (Fire and Forget). Service A emits an event and **does not wait**. Service B (and C, D) listens and reacts.
    -   *Use Case:* "User Created" event -> Send Email, Update Analytics.
66. How does a Hybrid Application work in NestJS?
**Answer:**
A Hybrid Application is an application that listens for HTTP requests **AND** Microservice messages at the same time.
-   **Method:** You create a standard HTTP app (`NestFactory.create`) and then call `app.connectMicroservice({...})` on it.
-   **Use Case:** You have a Monolith that serves a REST API to the frontend but also needs to consume messages from a RabbitMQ queue sent by another service.

67. What are Cron Jobs / Task Scheduling in NestJS?
**Answer:**
NestJS leverages the `@nestjs/schedule` package (wrapper around `node-cron`).
1.  **Setup:** Import `ScheduleModule.forRoot()` in `AppModule`.
2.  **Usage:** Use the `@Cron()` decorator on a service method.
    ```typescript
    @Cron('45 * * * * *') // Runs at 45th second of every minute
    handleCron() {
      this.logger.debug('Called when the second is 45');
    }
    ```
3.  It also supports Intervals (`@Interval`) and Timeouts (`@Timeout`).

68. How do you implement WebSockets (Gateways) in NestJS?
**Answer:**
NestJS provides a module `@nestjs/websockets` that works with either **Socket.io** (default) or **ws** (native implementation).
1.  **Gateway:** Create a class decorated with `@WebSocketGateway()`. This acts like a "Controller" for sockets.
2.  **Listeners:** Use `@SubscribeMessage('chat')` to listen for events.
3.  **Emitters:** Use `@MessageBody()` to read data and return data to send an acknowledgment, or inject `WebSocketServer` to broadcast messages.

69. What is CQRS (Command Query Responsibility Segregation) module in NestJS?
**Answer:**
CQRS is an advanced architectural pattern usually reserved for complex domains (DDD).
-   **Segregation:** It splits the application into two sides:
    1.  **Command Side (Write):** Changes state. Uses `CommandBus` to dispatch commands (`CreateUserCommand`). Handlers execute the logic.
    2.  **Query Side (Read):** Reads state. Uses `QueryBus` to dispatch queries (`GetUserQuery`).
-   **Benefit:** Allows scaling reads/writes independently and keeps complex write logic away from simple read logic.
-   **Module:** `@nestjs/cqrs`.

70. How does NestJS handle file uploads?
**Answer:**
NestJS uses the `multer` middleware (standard in the Node ecosystem) built-in.
1.  **Interceptor:** Use `FileInterceptor('fieldname')` on the controller route.
2.  **Decorator:** Use `@UploadedFile()` in the method params.
    ```typescript
    @Post('upload')
    @UseInterceptors(FileInterceptor('file'))
    uploadFile(@UploadedFile() file: Express.Multer.File) {
      console.log(file); // Metadata about file (buffer, size, mime)
    }
    ```

## Testing & Deployment
71. What testing framework is used by default in NestJS?
**Answer:**
**Jest** is the default testing framework.
-   NestJS provides a default configuration for generic Javascript/Typescript testing.
-   It includes `supertest` for HTTP assertions in E2E tests.
-   You typically run `npm run test` (for unit) or `npm run test:e2e`.

72. What is the difference between `.spec.ts` and `.e2e-spec.ts` files?
**Answer:**
-   **`.spec.ts` (Unit Tests):** These test **classes in isolation**. You manually instantiate the class or use a test module that mocks *all* dependencies. They are fast and focused.
    -   *Location:* usually right next to the source file (`cats.controller.ts` -> `cats.controller.spec.ts`).
-   **`.e2e-spec.ts` (End-to-End Tests):** These test the **entire running application**. It starts the full NestJS application (bootstraps `AppModule`), creates a real HTTP server (often mapped via `supertest`), and hits the actual endpoints (network request level).
    -   *Location:* usually in the `/test` directory at the root.

73. How do you mock a service in a Unit Test?
**Answer:**
You define a custom provider that replaces the real service token.
```typescript
const mockUsersService = {
  findAll: jest.fn().mockResolvedValue(['user1']),
  create: jest.fn(),
};

const module = await Test.createTestingModule({
  controllers: [UsersController],
  providers: [
    {
       provide: UsersService, // Replace this...
       useValue: mockUsersService // ...with this fake object
    }
  ],
}).compile();
```

74. How do you test a Controller with dependencies?
**Answer:**
1.  Use `Test.createTestingModule` to simulate the module.
2.  Provide the Controller in `controllers`.
3.  Provide **Mocks** for all the services the controller depends on (as shown in the previous answer).
4.  Get the instance: `controller = module.get<UsersController>(UsersController)`.
5.  Call methods: `expect(controller.findAll()).toEqual(...)`.

75. What is the `Test` class in `@nestjs/testing` used for?
**Answer:**
The `Test` class is a utility provided by NestJS to create an isolated **Testing Module** (similar to `NestFactory.create` but for tests).
-   It creates an IoC container just for that test suite.
-   It allows you to override providers easily (`.overrideProvider(Service).useValue(mock)`).
-   It avoids bootstrapping the entire app if you only want to test one small part.
76. How do you configure different environments (dev, prod, test) in NestJS?
**Answer:**
Typically via `ConfigModule` and `.env` files.
1.  **Environment Files:** Create `.env.development`, `.env.production`.
2.  **ConfigModule:** Configure it to load the correct file based on `NODE_ENV`.
    ```typescript
    ConfigModule.forRoot({
      envFilePath: `.env.${process.env.NODE_ENV}`,
      isGlobal: true,
    });
    ```
3.  **Cross-Env:** Use the `cross-env` package in `package.json` scripts to set `NODE_ENV` properly on Windows/Linux. (`"start:prod": "cross-env NODE_ENV=production node dist/main"`).

77. How does Hot Module Replacement (HMR) work in NestJS?
**Answer:**
Standard `npm run start:dev` restarts the **entire** server on every file change. HMR swaps **only the changed modules** without a full restart, preserving connection state.
-   It uses Webpack HMR under the hood.
-   Requires installing `run-script-webpack-plugin` and modifying `main.ts` to accept hot updates:
    ```typescript
    if (module.hot) {
      module.hot.accept();
      module.hot.dispose(() => app.close());
    }
    ```

78. What are build tags or how do you exclude files from the build?
**Answer:**
NestJS uses `tsconfig.json` to determine what to compile.
-   **Exclude:** To exclude files (like test files or docs) from the final `./dist` output, you modify the `exclude` array in `tsconfig.build.json`.
    ```json
    "exclude": ["node_modules", "test", "**/*spec.ts"]
    ```
-   **Assets:** To copy non-code assets (images, graphql files), use `nest-cli.json` `compilerOptions.assets`.

79. How do you deploy a NestJS application to a Docker container?
**Answer:**
1.  **Build:** Create a `Dockerfile`.
    -   Base Image: `node:18-alpine`.
    -   Install Deps: `npm ci`.
    -   Copy Source: `COPY . .`
    -   Build: `npm run build`.
2.  **Run:**
    -   Expose Port: `EXPOSE 3000`.
    -   Command: `CMD ["node", "dist/main"]`.
*Tip:* Use multi-stage builds to keep the final image small (exclude `src` and devDependencies).

80. How do you improve the startup time of a cold serverless NestJS function?
**Answer:**
Serverless (Lambda/Cloud Functions) has a "Cold Start" problem. NestJS is heavy.
1.  **Webpack:** Bundle the entire app into a single file to reduce disk I/O (built-in in Nest CLI).
2.  **Lazy Loading:** Do not import expensive modules in `AppModule`. Use `LazyModuleLoader` to load them only when needed.
3.  **Fastify:** Switch from Express to Fastify for slightly faster boot.
4.  **Keep Warm:** Use a ping mechanism to keep the lambda alive (though this costs money).

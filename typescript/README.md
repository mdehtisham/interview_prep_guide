# Top TypeScript Interview Questions

## Core Concepts & Basics
1. What is TypeScript and how is it different from JavaScript?
**Answer:**
TypeScript is a **superset** of JavaScript developed by Microsoft.
-   **Static Typing:** It adds optional static types to JavaScript (compile-time checking), whereas JS is dynamically typed (runtime checking).
-   **Features:** It includes features from future JS versions (Decorators, Enums, Interfaces) that compile down to standard JS.
-   **Execution:** TypeScript code cannot run directly in the browser or Node.js; it must be compiled (transpiled) into JavaScript.

2. What are the main benefits of using TypeScript?
**Answer:**
1.  **Early Error Detection:** Catches bugs (typos, type mismatches) at compile-time, reducing runtime crashes.
2.  **Tooling & Intelligent Code Completion:** Provides robust IntelliSense (autocompletion, parameter info, documentation) in IDEs like VS Code.
3.  **Refactoring:** Renaming variables or functions across a large codebase is safe and automated usage.
4.  **Readability:** Types act as "living documentation" for what functions expect and return.

3. How does TypeScript code execute in the browser?
**Answer:**
It **does not** execute in the browser.
-   Browsers only understand JavaScript.
-   TypeScript must be **Transpiled** (compiled) into JavaScript (ES5/ES6) using the TypeScript Compiler (`tsc`) or tools like Babel/SWC/Esbuild before it is served to the browser.
-   *Note:* There are loaders (like `ts-node` or specialized browser runners) for dev, but production always runs JS.

4. Explain the concept of "Transpilation".
**Answer:**
Transpilation is the process of taking source code written in one language and transforming it into another language that has a similar level of abstraction.
-   **Compiling:** High-level (C++) to Low-level (Assembly).
-   **Transpiling:** High-level (TypeScript) to High-level (JavaScript).
-   TypeScript converts `.ts` constructs (Interfaces, Types) into valid `.js` code (often erasing the types completely).

5. What is the difference between `any`, `unknown`, and `never`?
**Answer:**
-   **`any`:** Disables type checking. You can access any property on it. It defeats the purpose of TS. Use sparingly (only for migration).
-   **`unknown`:** A type-safe version of `any`. You can assign anything to it, but you **cannot** perform operations on it without "Narrowing" (checking the type) first.
    ```typescript
    let val: unknown = 'hello';
    // val.toUpperCase(); // Error!
    if (typeof val === 'string') val.toUpperCase(); // OK
    ```
-   **`never`:** Represents a value that **never** occurs. Used for functions that throw errors indefinitely (`while(true)`) or for "Exhaustive Checks" in switch statements (ensuring all cases are handled).

6. What is the `void` type and when is it used?
**Answer:**
`void` represents the absence of a value.
-   **Usage:** Commonly used as the return type of functions that do not return a value.
    ```typescript
    function logMessage(msg: string): void {
      console.log(msg);
      // implicitly returns 'undefined', which is compatible with void
    }
    ```
-   *vs undefined:* Technically a function returning `void` returns `undefined` at runtime, but TS treats `void` as "don't rely on this return value".

7. Explain Type Inference in TypeScript.
**Answer:**
Type Inference is the compiler's ability to automatically deduce the type of a variable based on its value, without explicit annotation.
-   **Example:**
    ```typescript
    let name = "John"; // TS knows 'name' is 'string'
    name = 5; // Error: Type 'number' is not assignable to type 'string'
    ```
-   **Best Practice:** Leverage inference! Don't write `let x: number = 5;`. Just write `let x = 5;`. It makes code cleaner.

8. What is "Type Assertion" (casting) and how do you do it?
**Answer:**
Type Assertion is telling the compiler "I know more about the type of this variable than you do". It overlaps the compiler's inference.
-   **Syntax 1 (`as`):** `const input = document.getElementById('my-input') as HTMLInputElement;`
-   **Syntax 2 (`<>`):** `const input = <HTMLInputElement>document.getElementById('my-input');` (Avoid in React/JSX files).
-   *Note:* It does NOT change the runtime data. It is purely a compile-time instruction.

9. What is the difference between `null` and `undefined` in TypeScript?
**Answer:**
-   **`undefined`:** A variable has been declared but not assigned a value.
-   **`null`:** An intentional absence of any object value.
-   **Strict Mode:** With `strictNullChecks: true`, `null` and `undefined` are distinct types. You cannot assign `null` to a generic `string` variable unless the type is explicitly `string | null`.

10. Can you compile a TypeScript file with errors?
**Answer:**
**Yes.** By default, the TypeScript compiler (`tsc`) will still emit `.js` files even if there are type errors (assuming syntax is valid).
-   **Rationale:** To allow incremental migration where full type safety isn't yet achieved.
-   **Prevention:** Set `"noEmitOnError": true` in `tsconfig.json` to prevent invalid code from being generated.

## Interfaces & Types
11. What is the difference between `interface` and `type` alias?
**Answer:**
-   **Interfaces:** Designed specifically for defining **object shapes** (contracts).
    -   Can be **merged** (Declaration Merging). A second interface with same name adds new properties.
    -   Better error messages (traditionally).
    -   Convention for defining public APIs of libraries.
-   **Type Aliases:** Can define objects, but also **primitives, unions, intersections, and tuples**.
    -   `type ID = string | number;` (Interface cannot do this).
    -   Cannot be merged. A second type with same name is a compile error.

12. When should you use an Interface vs a Type?
**Answer:**
-   **Use `interface` when:**
    -   Defining the shape of Objects or Classes.
    -   Authoring a library (allows users to extend your helper interfaces via merging).
-   **Use `type` when:**
    -   Defining Unions (`string | number`) or Tuples.
    -   Defining aliases for primitives (`type UUID = string`).
    -   If you need advanced features like Conditional Types.

13. Can Interfaces be extended? Can Types be extended?
**Answer:**
-   **Interfaces:** `interface Admin extends User { role: string; }`.
-   **Types:** Uses Intersection `&`. `type Admin = User & { role: string; };`.
    -   *Technically both can express "extension", but interfaces have a dedicated `extends` keyword.*

14. What is "Declaration Merging" and does it work with Types?
**Answer:**
Declaration Merging is when the compiler merges two or more separate declarations declared with the same name into a single definition.
-   **Works on:** Interfaces and Namespaces.
-   **Does NOT work on:** Classes or Type Aliases.
    ```typescript
    interface User { name: string; }
    interface User { age: number; }
    const u: User = { name: 'A', age: 20 }; // Valid: User now requires BOTH.
    ```

15. How do you define optional properties in an interface?
**Answer:**
Use the `?` suffix on the property name.
```typescript
interface Car {
  make: string;
  model?: string; // Optional
}
const myCar: Car = { make: 'Toyota' }; // Valid, model is missing
```
-   At runtime, `myCar.model` will be `undefined`.
16. What are "Readonly" properties?
**Answer:**
Properties marked with `readonly` can only be modified when the object is **first created** (initialization).
```typescript
interface Point {
  readonly x: number;
  y: number;
}
const p: Point = { x: 10, y: 20 };
// p.x = 5; // Error: Cannot assign to 'x' because it is a read-only property.
```
*Note:* This is a compile-time check only. At runtime, arrays/objects can still be mutated if you bypass TS constraints.

17. Explain "Index Signatures" in interfaces.
**Answer:**
Index signatures allow you to define types for properties if you don't know their names ahead of time (Dynamic objects/Dictionaries).
```typescript
interface Dictionary {
  [key: string]: number; // Any string key must have a number value
}
const scores: Dictionary = {
  "math": 90,
  "science": 85
};
```

18. What is a "Hybrid Type" in TypeScript?
**Answer:**
An object that acts as both a function and an object (has properties).
-   Common in older JavaScript libraries (like jQuery `$`).
```typescript
interface Counter {
    (start: number): string; // Callable signature
    interval: number; // Property signature
    reset(): void; // Method signature
}
```

## Classes & OOP
19. What are Access Modifiers (`public`, `private`, `protected`) in TypeScript?
**Answer:**
-   **`public` (Default):** Accessible anywhere.
-   **`private`:** Accessible **only inside** the containing class. Not accessible in derived classes (subclasses).
-   **`protected`:** Accessible inside the containing class **and** in derived classes.
*Note:* These are erased at runtime (Javascript classes don't inherently support these before ES2020 private fields).

20. What is the `readonly` modifier in a class property?
**Answer:**
Similar to interface `readonly`.
-   A class property marked `readonly` can only be set in the property declaration or in the **constructor**.
-   After the constructor finishes, it is immutable.
```typescript
class Person {
  readonly birthDate: Date;
  constructor(date: Date) {
    this.birthDate = date; // OK
  }
}
```
21. What is the difference between `private` and `#private` (private fields)?
**Answer:**
-   **`private` (Soft Private):** It is a **TypeScript-only** modifier.
    -   Erased at runtime.
    -   You can access the property using `(instance as any).prop`.
-   **`#prop` (Hard Private):** It is a standard **JavaScript (ECMAScript)** feature.
    -   Enforced at runtime by the JS engine.
    -   You **cannot** access it from outside the class, even with dirty hacks. It throws a syntax error or runtime error.

22. What are `abstract` classes and methods?
**Answer:**
-   **Abstract Class:** A base class that **cannot be directly instantiated**. It serves as a template for other classes.
-   **Abstract Method:** A method declared in an abstract class that has **no implementation**. Derived classes **must** implement it.
    ```typescript
    abstract class Animal {
      abstract makeSound(): void; // Contract
      move(): void { console.log('moving'); } // Implementation
    }
    ```

23. Can you implement multiple interfaces in a single class?
**Answer:**
**Yes.** TypeScript supports multiple interface implementation.
```typescript
class SmartPhone implements Phone, InternetBrowser, Camera {
    // Must implement methods from all three
}
```

24. How do getters and setters work in TypeScript classes?
**Answer:**
They use the `get` and `set` keywords, allowing you to intercept access to a property.
```typescript
class Circle {
  private _radius: number = 0;
  
  get radius(): number { return this._radius; }
  
  set radius(value: number) {
    if (value < 0) throw new Error('Cannot be negative');
    this._radius = value;
  }
}
```

25. Explain the concept of "Parameter Properties" in constructors.
**Answer:**
It is a shorthand syntax to declare and initialize class properties in one place.
-   **Without:**
    ```typescript
    class A {
      public x: number;
      constructor(x: number) { this.x = x; }
    }
    ```
-   **With (Parameter Property):**
    ```typescript
    class A {
      constructor(public x: number) {} 
    }
    ```
-   Adding `public`, `private`, or `readonly` before a constructor parameter automatically creates a field with that name and assigns the value.

## Advanced Types
26. What are Union Types and Intersection Types?
**Answer:**
-   **Union (`|`):** A value that can be one of several types.
    -   `type Status = 'open' | 'closed';` (It is X OR Y).
-   **Intersection (`&`):** A value that combines multiple types into one.
    -   `type Admin = User & permissions;` (It is X AND Y). The resulting object must have properties of *both*.

27. What is a "Type Guard"?
**Answer:**
A Type Guard is a technique/expression that checks the type of a variable at runtime, allowing TypeScript to **Narrow** the type within a specific scope.
-   **Examples:** `typeof x === 'string'`, `instanceof Error`, `Array.isArray(x)`.

28. Explain the usage of the `is` keyword (Custom Type Guards).
**Answer:**
It is used to define a function whose return value provides a **Type Predicate** to the compiler.
```typescript
function isFish(pet: Fish | Bird): pet is Fish {
    return (pet as Fish).swim !== undefined;
}
// Usage
if (isFish(myPet)) {
   myPet.swim(); // TS knows 'myPet' is Fish here
}
```
-   Without `pet is Fish`, TS would only know the function returns a `boolean` and wouldn't narrow the type inside the `if` block.

29. What are "Discriminated Unions" (Tagged Unions)?
**Answer:**
A pattern where a Union of Interfaces shares a common **literal** property (the tag). TS uses this tag to infer the specific type.
```typescript
interface Square { kind: "square"; size: number; }
interface Circle { kind: "circle"; radius: number; }
type Shape = Square | Circle;

function area(s: Shape) {
  if (s.kind === "square") {
     return s.size * s.size; // TS knows it's a Square
  }
}
```

30. What are "Literal Types" (String/Number/Boolean literals)?
**Answer:**
Instead of general types like `string`, you can specify exact values.
-   `type Direction = "North" | "South";`
-   `type DiceRoll = 1 | 2 | 3 | 4 | 5 | 6;`
-   They are extremely powerful when combined with Unions to enforce specific valid states.
31. What is the `keyof` operator?
**Answer:**
It takes an object type and yields a **Union of its keys** (as string literals).
```typescript
interface User { id: number; name: string; }
type UserKeys = keyof User; // "id" | "name"
```
It is essential for writing generic functions that access object properties safely.

32. What is the `typeof` operator in TypeScript context?
**Answer:**
In standard JS, `typeof` returns a string ("string", "object").
In TS Type Context, `typeof` returns the **Type** of an existing variable/object.
```typescript
const config = { width: 10, height: 20 };
type Config = typeof config; // { width: number; height: number; }
```
It is useful to generate types from actual code/data structures ("Source of Truth").

33. Explain "Mapped Types".
**Answer:**
Mapped types allow you to create new types by transforming properties of an existing type. It's like `.map()` for types.
```typescript
type ReadOnly<T> = {
  readonly [P in keyof T]: T[P];
};
```
It iterates over every key `P` in `keyof T` and modifies the property descriptor (e.g., adding `readonly` or `?`).

34. Explain "Conditional Types" (`T extends U ? X : Y`).
**Answer:**
It works exactly like a Ternary Operator in JavaScript, but for Types.
```typescript
type TypeName<T> = T extends string ? "text" :
                   T extends number ? "number" : "object";
```
-   **Narrowing:** If `T` matches `U`, type becomes `X`, else `Y`.
-   **Distributive:** If `T` is a union `A | B`, the condition runs for both: `(A extends U ?...) | (B extends U ?...)`.

35. What is the `infer` keyword?
**Answer:**
It allows you to **extract/deduce** a type from within a Conditional Type.
-   Common use: Extracting the return type of a function.
    ```typescript
    type ReturnType<T> = T extends (...args: any[]) => infer R ? R : any;
    ```
-   TS checks if `T` is a function. If yes, it "infers" the return type and assigns it to variable `R`, then returns `R`.
36. What acts are "Template Literal Types"?
**Answer:**
They allow you to build types by concatenating string literals, similar to template strings in JS.
```typescript
type Color = "red" | "blue";
type Quantity = "light" | "dark";
type Palette = `${Quantity}-${Color}`; 
// Result: "light-red" | "light-blue" | "dark-red" | "dark-blue"
```
Used heavily in UI libraries (e.g., generating CSS class names or event names like `on${Event}`).

37. What are "Recursive Types"?
**Answer:**
A type that references itself within its own definition.
-   Common for defining JSON or Tree structures.
```typescript
type JSONArray = JSONValue[];
type JSONValue = string | number | boolean | null | JSONArray | { [k: string]: JSONValue };
```

## Generics
38. What are Generics and why are they useful?
**Answer:**
Generics allow you to write **reusable, adaptable code** that works with a variety of types while maintaining type safety.
-   Instead of `any`, you use a variable type parameter `<T>`.
-   **Benefit:** The caller decides the type, and TS enforces usage of that specific type throughout the function/class.

39. How do you create a Generic Function?
**Answer:**
```typescript
function identity<T>(arg: T): T {
  return arg;
}
// Usage
const num = identity<number>(10); // T is number
const str = identity("hello"); // T is inferred as "hello" (literal) or string
```

40. How do you create a Generic Interface/Class?
**Answer:**
```typescript
interface Box<T> {
  contents: T;
}

class StorageBin<T> {
  private _item: T;
  constructor(item: T) { this._item = item; }
  getItem(): T { return this._item; }
}

const numBox: Box<number> = { contents: 100 };
```
41. Can you constrain a Generic Type (`<T extends ...>`)?
**Answer:**
**Yes.** You can restrict `T` so that it must satisfy a certain interface.
```typescript
interface Lengthwise { length: number; }

function logLength<T extends Lengthwise>(arg: T) {
  console.log(arg.length); // Safe to access .length
}
logLength([1, 2]); // OK (Arrays have length)
logLength(3); // Error (Numbers don't have length)
```

42. How do you specify default values for Generic Types?
**Answer:**
You can provide a default type using `=`.
```typescript
class Wrapper<T = string> { ... }
const w = new Wrapper(); // T is string
```

43. usage of `keyof` with Generics.
**Answer:**
A very common pattern to ensure one argument is a property name of another argument.
```typescript
function getProperty<T, K extends keyof T>(obj: T, key: K) {
  return obj[key];
}
const x = { a: 1, b: 2 };
getProperty(x, "a"); // OK
getProperty(x, "m"); // Error: "m" is not in "a" | "b"
```

## Utility Types (Built-in)
44. Explain `Partial<T>`.
**Answer:**
Constructs a type with all properties of `T` set to **optional**.
```typescript
interface User { id: number; name: string; }
type PartialUser = Partial<User>; // { id?: number; name?: string; }
```
Useful for "Patch" (update) operations where you only send changed fields.

45. Explain `Required<T>`.
**Answer:**
Constructs a type with all properties of `T` set to **required**. The opposite of `Partial`.
```typescript
interface Props { a?: number; b?: string; }
type AllProps = Required<Props>; // { a: number; b: string; }
```
46. Explain `Readonly<T>`.
**Answer:**
Constructs a type with all properties of `T` set to **readonly**.
```typescript
interface Todo { title: string; }
const todo: Readonly<Todo> = { title: "Buy milk" };
// todo.title = "Buy bread"; // Error
```
Useful for Redux states or Immutable patterns.

47. Explain `Pick<T, K>`.
**Answer:**
Constructs a type by **picking** a set of properties `K` from `T`.
```typescript
interface User { id: number; name: string; email: string; }
type UserPreview = Pick<User, "id" | "name">; // { id: number; name: string; }
```

48. Explain `Omit<T, K>`.
**Answer:**
Constructs a type by **omitting** (removing) a set of properties `K` from `T`.
```typescript
interface User { id: number; name: string; email: string; }
type UserInput = Omit<User, "id">; // { name: string; email: string; }
```
Very useful for removing database-generated IDs from DTOs.

49. Explain `Record<K, T>`.
**Answer:**
Constructs an object type whose property **keys** are `K` and whose property **values** are `T`.
```typescript
type PageInfo = { title: string };
type Pages = Record<"home" | "about" | "contact", PageInfo>;
// Ensures the object has exactly these 3 keys, all pointing to PageInfo objects.
```

50. Explain `Exclude<T, U>` and `Extract<T, U>`.
**Answer:**
-   **`Exclude<T, U>`:** Constructs a type by excluding from `T` all union members that are assignable to `U` (Remove U from T).
    -   `Exclude<"a" | "b" | "c", "a">` -> `"b" | "c"`
-   **`Extract<T, U>`:** Constructs a type by extracting from `T` all union members that are assignable to `U` (Keep only intersection of T and U).
    -   `Extract<"a" | "b" | "c", "a" | "f">` -> `"a"`
51. Explain `NonNullable<T>`.
**Answer:**
Constructs a type by excluding `null` and `undefined` from `T`.
```typescript
type T0 = string | null | undefined;
type T1 = NonNullable<T0>; // string
```

52. Explain `ReturnType<T>`.
**Answer:**
Constructs a type consisting of the **return type** of function `T`.
```typescript
function getUser() { return { name: "John", age: 30 }; }
type User = ReturnType<typeof getUser>; // { name: string; age: number; }
```
Extremely useful when dependent libraries don't export their internal types.

53. Explain `Parameters<T>`.
**Answer:**
Constructs a tuple type from the types used in the **parameters** of a function `T`.
```typescript
function save(id: number, active: boolean) {}
type SaveParams = Parameters<typeof save>; // [number, boolean]
```

54. What is `Awaited<T>`?
**Answer:**
It models the behavior of `await` specifically for types. It recursively unwraps Promises.
```typescript
type A = Awaited<Promise<string>>; // string
type B = Awaited<Promise<Promise<number>>>; // number
```

## Configuration (tsconfig.json)
55. What is the purpose of `tsconfig.json`?
**Answer:**
1.  **Project Root:** Marks the root directory of a TypeScript project.
2.  **Configuration:** Configures the compiler options (Output directory, Target JS version, Module resolution strategy).
3.  **Scope:** Defines which files to include/exclude from compilation.
56. What does `noImplicitAny` do?
**Answer:**
When enabled, it raises an error whenever TypeScript infers the type `any` for a variable because it couldn't figure out the type.
-   **Example:** `function fn(s) { console.log(s); }` -> Error: Parameter 's' implicitly has an 'any' type.
-   **Fix:** You must explicitly type it: `function fn(s: any)`. It forces you to be aware of your `any` usage.

57. What is `strictNullChecks`?
**Answer:**
When enabled (`true`):
-   `null` and `undefined` are **not** assignable to every other type.
-   `let x: number = null;` // Error.
-   You typically must handle nulls before using the variable (`if (x !== null) ...`). This effectively kills "Cannot read property of undefined" errors at compile time.

58. What does `strict: true` actually enable?
**Answer:**
It is a "Meta-flag" that enables a family of strict type-checking options, including:
-   `noImplicitAny`
-   `strictNullChecks`
-   `strictFunctionTypes`
-   `strictPropertyInitialization`
-   `noImplicitThis`
-   `alwaysStrict` (parses in JS strict mode)

59. What is the difference between `target` and `lib` in compiler options?
**Answer:**
-   **`target` (Output):** Determines the version of JavaScript generated (syntax).
    -   `target: "ES5"` -> Arrow functions converted to `function`.
    -   `target: "ES6"` -> Arrow functions kept.
-   **`lib` (Input):** Determines the **Type Definitions** available to your code (environment).
    -   Includes built-in JS APIs like `Array.find`, `Promise`.
    -   Includes DOM APIs like `window`, `document`.

60. What is `skipLibCheck`?
**Answer:**
When set to `true`, TS **skips type checking of declaration files (`.d.ts`)**.
-   **Why:** Often, 3rd party libraries (`node_modules`) have conflicting or erroneous type definitions. You don't want YOUR build to fail because `lodash` has a type error in line 1000. It speeds up compilation significantly.
61. What is "Path Mapping" (`paths`) in tsconfig?
**Answer:**
It allows you to define aliases for imports, avoiding long relative paths (`../../../../utils`).
```json
"paths": {
  "@utils/*": ["src/utils/*"],
  "@models/*": ["src/models/*"]
}
```
*Note:* You often need to configure your builder (Webpack/Vite) or runtime (ts-node-dev) to also understand these paths, as tsc only handles type checking and emit, but doesn't rewrite the paths in the output JS by default.

62. What is the difference between `module` and `moduleResolution`?
**Answer:**
-   **`module`:** Defines the **format** of the output JS module system (CommonJS, AMD, ESNext, SystemJS).
-   **`moduleResolution`:** Defines the **algorithm** TS uses to find the import files on disk (Node, Classic). It usually needs to match the runtime environment (usually "Node").

63. How do you include/exclude files from compilation?
**Answer:**
Using the `include` and `exclude` arrays in `tsconfig.json`.
-   **include:** List of glob patterns (`src/**/*`).
-   **exclude:** List of glob patterns to ignore (`node_modules`, `**/*.spec.ts`).

## Modules & Namespaces
64. What is the difference between a Module and a Namespace?
**Answer:**
-   **Module (External Module):** Standard ES6/CommonJS modules. Each file is a module. You use `import` and `export`. **Preferred in modern TS.**
-   **Namespace (Internal Module):** TypeScript-specific way to organize code using the `namespace` keyword.
    -   They compile to IIFEs (Immediately Invoked Function Expressions).
    -   Can span multiple files using `/// <reference path="..." />`.
    -   Mostly used for declaration files or legacy code.

65. When should you use Namespaces in modern TypeScript?
**Answer:**
**Rarely.**
-   You typically only use them when adding types to existing global libraries (via Declaration Merging) that rely on global variables.
-   For your own application code, ALWAYS use ES Modules.
66. Explain "Ambient Declarations" (`declare` keyword).
**Answer:**
The `declare` keyword tells the compiler: "Trust me, this variable/function exists completely somewhere else (e.g., loaded via script tag), so let me use it without errors."
-   `declare var JQuery: any;`
-   There is no JavaScript code emitted for ambient declarations.

67. What are `.d.ts` files?
**Answer:**
They are **Type Declaration Files**.
-   They contain **only** type definitions (interfaces, types, ambiant declarations), no logic/executable code.
-   **Purpose:** To describe the shape of an existing JavaScript library to TypeScript.
-   When you install `@types/react`, you are downloading `index.d.ts` so TS understands React.

68. How do you use 3rd party JavaScript libraries in TypeScript?
**Answer:**
1.  **Install the library:** `npm install lodash`.
2.  **Install types:** `npm install -D @types/lodash`.
3.  **Use:** `import _ from 'lodash';`.
-   If no types exist on DefinitelyTyped (`@types`), you can create a `typings.d.ts` file and add `declare module 'my-lib';` to suppress errors (implies `any`).

69. What is `DefinitelyTyped` (`@types`)?
**Answer:**
It is a huge community repository (on GitHub) of type definitions for popular JavaScript libraries.
-   NPM scope `@types/*` maps to this repo.
-   It allows TS developers to use thousands of pure JS libraries with full type safety.

70. How do you extend a global object like `Window`?
**Answer:**
You use **Declaration Merging** on the global interface.
```typescript
declare global {
  interface Window {
    myAnalytics: any;
  }
}
// Now valid
window.myAnalytics = {};
```
*Note:* If you are in a module (file has imports/exports), you must wrap it in `declare global`. In a script file, you can just `interface Window`.

## Tricky / Expert
71. What is type "Variance" (Covariance vs Contravariance)?
**Answer:**
It describes how complex types relate to each other when their components are related.
-   **Covariance (Arrays/Return Types):** If `Dog extends Animal`, then `Dog[]` extends `Animal[]`. You can use more specific array where generic array is expected.
-   **Contravariance (Function Arguments):** It's the opposite! If you need a function `(a: Animal) => void`, you CANNOT provide `(d: Dog) => void`. You MUST provide `(thing: any) => void` or `(a: Animal) => void`. You can accept *broader* types than requested.
-   *Mnemonic:* Be liberal in what you accept (arguments), and conservative in what you send (return values).

72. Why is `Function` bivariant in TypeScript?
**Answer:**
Strictly speaking, function arguments should be contravariant.
However, in TS (unless `strictFunctionTypes` is on), method arguments are Bivariant (both Co and Contra).
-   **Reason:** To allow common JS patterns (like event handlers or array prototyping) to work without excessive casting, even though it is technically unsound.

73. What is "Type Erasure"?
**Answer:**
It refers to the fact that **all type information is removed** during compilation to JavaScript.
-   There is no reflection at runtime.
-   `interface User` does not exist in the browser.
-   You cannot check `if (obj instanceof UserInterface)`.

74. How does `const` assertions (`as const`) work?
**Answer:**
It tells the compiler to infer the **most specific literal type** possible and make properties **readonly**.
```typescript
const config = {
  endpoint: "https://api.com",
  retries: 3
} as const;
// Type is: { readonly endpoint: "https://api.com"; readonly retries: 3; }
```
-   Without `as const`, type would be `{ endpoint: string; retries: number; }`.

75. What is the "Satisfies" operator (`satisfies`)?
**Answer:**
Introduced in TS 4.9. It validates that an expression matches a type **without changing the inferred type** of that expression.
```typescript
type Colors = Record<string, string | RGB>;
const palette = {
    red: "red",
    green: [0, 255, 0]
} satisfies Colors;

// palette.red is inferred as "string" (Still specific!)
palette.red.toUpperCase(); // OK
```
-   If we used `: Colors`, `palette.red` would be `string | RGB`, and `toUpperCase()` would fail.
76. Explain "Nominal Typing" vs "Structural Typing". Which one does TS use?
**Answer:**
-   **Structural Typing (Duck Typing):** Used by **TypeScript**. If object A has at least the same properties as object B, then A is compatible with B, regardless of their names/classes.
    ```typescript
    class Dog { name: string; }
    class Cat { name: string; }
    let p: Dog = new Cat(); // OK in TypeScript! Both have 'name'.
    ```
-   **Nominal Typing:** Used by Java/C#. Compatibility is determined by explicit inheritance and class names. A `Cat` is never a `Dog`.

77. How can you simulate Nominal Typing in TypeScript?
**Answer:**
By using **Branding** (or Opaque Types). You add a unique property that doesn't exist at runtime but distinguishes the type at compile time.
```typescript
type USD = number & { __brand: "USD" };
type EUR = number & { __brand: "EUR" };

function pay(amount: USD) {}
let dollars = 10 as USD;
let euros = 10 as EUR;
pay(euros); // Error!
```

78. What are "Tuple Types"?
**Answer:**
A Tuple is an array with a **fixed number of elements** whose types are known but need not be the same.
```typescript
let x: [string, number];
x = ["hello", 10]; // OK
x = [10, "hello"]; // Error
```
Useful for returning multiple values from a function (like React `useState`).

79. How do you solve the error "Property 'x' does not exist on type 'never'"?
**Answer:**
This usually happens inside a function with explicit type narrowing, where TypeScript has determined that **no possible value** can reach that line of code (Exhaustive check).
-   **Cause:** Your previous `if/else` or `switch` covered all possibilities, so the variable is type `never` in the `else` block.
-   **Fix:** Check your logic. You might be handling a case that shouldn't exist, or your previous type guards are too strict.

80. What is the difference between `object` (lowercase) and `Object` (uppercase)?
**Answer:**
-   **`object`:** Represents any **non-primitive** type. (Anything that is NOT number, string, boolean, symbol, null, or undefined).
    -   *Use Case:* `function create(o: object | null): void;`
-   **`Object`:** Describes functionality common to all JavaScript objects (Has `toString()`, `valueOf()`).
    -   *Danger:* It includes primitives! `const s: Object = "hello";` is VALID because strings have methods.
-   *Recommendation:* Always use `object` (lowercase) when you mean "a generic key-value map or class instance".

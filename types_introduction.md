# Types

## Table of contents

<!-- toc -->

- [Primitive (basic) types](#primitive-basic-types)
- [Arrays](#arrays)
- [Tuples](#tuples)
- [Functions](#functions)
- [Any and unknown](#any-and-unknown)
- [Type aliases](#type-aliases)
- [Type interfaces](#type-interfaces)
- [Function type](#function-type)
- [void](#void)
- [never](#never)
- [Enums](#enums)

<!-- tocstop -->

## Primitive (basic) types

TypeScript will automatically infer the following basic JavaScript data types:

```typescript
// string -> string
let fistName = 'John';
// boolean -> boolean
let isAdmin = false;
// number -> number
let age = 30;
// null -> any
let car = null;
// undefined -> any
let bicycle = undefined;
// Array -> string[]
let duties = ['write code', 'fix bugs'];
// Array -> (string | number)[]
let duties = ['write code', 'fix bugs', 100];
// Object -> { name: string }
let obj = { name: 'Bob' };
// bigint -> bigint
let salary = 50n;
// Symbol -> symbol
let logo = Symbol('emerald');
// Function -> () => void
let work = () => {
    console.log('working...')
}
```

TypeScript adds its own basic types:

- arrays
- tuples
- any
- unknown
- void
- never
- Enums


## Arrays

Arrays can be typed using different syntaxes which are functionally equivalent, for example: 

```typescript
// Type is automatically inferred:
const duties = ['write code'];
// Using array type literal syntax
const duties: string[] = ['write code'];
// Using generic array type syntax
const duties: Array<string> = ['write code'];
```

The `string[]` syntax is generally thought to be more readable and concise.
The `Array<string>` syntax is more verbose but can be clearer for more complex types.

## Tuples 

JavaScript doesn't include native tuple types, but in TypeScript they can be thought of as an array with fixed size and structure where the type and position of each element are known. This is particularly useful for representing a set of values that have different types but are logically connected, like a database row, a coordinate pair, etc.

A tuple like `[string, number]` immediately tells you that this structure holds a string and a number **in that order**. Once a tuple is initialized, its structure is set, for example:

```typescript
// Tuple is initialized using literal syntax 
// (TypeScript does not have a separate generic syntax for tuples)
let userCode: [string, number] = ['John', 345];
// You can safely reassign the values of the same type, in the same order:
userCode = ['Max', 6];
// This will fail: 
userCode = ['Max', 6, 'admin'];
// This will fail:
userCode = [6, 'Max'];
// This is fine:
userCode[0] = 'Mary';
// This will fail:
userCode[0] = 1;
// However, this will work since tuples compile to Arrays in JavaScript. 
// But it's expected that developers using TypeScript follow the intended use 
// of tuples and avoid actions like pushing additional elements:
userCode.push(12); //
```

While TypeScript tuples offer a way to enforce a certain structure at initialization, they don't completely prevent runtime modifications like push. They are more about providing clearer intent and compile-time checks rather than enforcing strict runtime constraints.

The part about tuples being a fixed length, is optional by design. For example, you can make a tuple where the first element is a string, followed by any number of numbers using the `...` rest operator on an array:

```typescript
let userCode: [string, ...number[]];
```

You can also label elements in tuples, enhancing code readability. For example:

```typescript
let userCode: [name: string, id: number];
```

## Functions

TypeScript functions let you define the type for the function's parameters as well as its return value. 

```typescript
// At the very least, I have to tell it what type the parameter is since 
// it can't be inferred. The return value however, can be inferred:
function greet(name: string) {
  return `Hello ${name}`;
}
// If I want to explicitly express the return value:
function greet(name: string): string {
  return `Hello ${name}`;
}
// As a function expression:
const greet = (name: string): string => {
  return `Hello ${name}`;
};
// If you use default params, the parameter type can be inferred:
function greet(name = 'Friend') {
  return `Hello ${name}`;
}
// You can include optional params with `?`:
function greet(name: string, greeting?: string): string {
  return `${greeting || 'Hello'} ${name}`;
}
```

Note this patten: `(name: string): string` is often referred to as a function's *call signature*.

So, should you let the return value be inferred or explicitly define it?

- **Simple Functions**: For simpler functions where the return type is obvious, the implicit return type (first approach) is often used due to its conciseness.
- **Complex Functions**: In more complex functions, or in codebases that prioritize explicit type annotations for clarity and maintainability, the explicit return type (second approach) is preferred.
- **Project or Team Standards**: The choice can also depend on the coding standards of a project or team. Some teams might enforce explicit types for consistency and clarity, while others might opt for the brevity of implicit types where appropriate.

Note the type is can also be inferred when the function is called:

```typescript
function greet(name: string): string {
  return `Hello ${name}`;
}

let myGreet = greet; // inferred type 

let myGreeting = myGreet('Sue'); // inferred type
```

This could be explicitly written as:

```typescript
function greet(name: string): string {
  return `Hello ${name}`;
}

let myGreet: (name: string) => string = greet; // explicit type

let myGreeting: string = myGreet('Sue'); // explicit type
```

Note when using the rest operator, remember that you must make the type an array. This really just makes it so that you pass individual string params to teh function instead of an array. For example:

```typescript
// You can use the rest operator here:
function greetMultiple(...names: string[]) {
  names.forEach((name) => {
    console.log(greet(name));
  });
}
// and therefor call with separate string params:
greetMultiple('John');
greetMultiple('John', 'Mary');

// Or remove the rest operator:
function greetMultiple(names: string[]) {
  names.forEach((name) => {
    console.log(greet(name));
  });
}
// and pass an array param:
greetMultiple(['John']);
greetMultiple(['John', 'Mary']);
```

## Any and unknown

The `any` type is used to completely disable any type checking:

```typescript
// Disable type checking:
let queryResult: any = 5;
queryResult = '5';
queryResult = [5];
```

This is often used when a codebase is migrating from JavaScript to TypeScript or when doing unit tests.

Initially you might see the `any` type as useful when returning content with `JSON.parse()` as it may not be known what you will end up with. However, in this case the type `unknown` is actually more correct.

```typescript
function getDataFromAPI(id: number): unknown {
  // Fetching data here
  return JSON.parse(data);
}
```

While `any` type essentially tells TypeScript to bypass its type checking system, `unknown` is considered its type-safe counterpart. A value of type `unknown` can't be used in most operations without first asserting or narrowing its type. The idea is by using `unknown`, you acknowledge the uncertainty, but write assertions to narrow down the `unknown` type to a more specific type that you expect.

```typescript
const result = getDataFromAPI(someId);

// type narrowing:
if (typeof result === 'string') {
  // Now safely treated as a string
}
if (typeof result === 'number') { 
  // Now safely treated as a number
};
if (typeof result === 'string' || typeof result === 'number') {
  // Intellisense in vscode will show which functions are common to 
  // both strings and numbers:
  result.valueOf();
};
```

## Type aliases 

Type aliases give us a way to create a custom type or to define the shape of an object. Type aliases are often used for complex types that you want to reuse throughout your code. They can represent a wide range of types, including primitives, unions, and tuples. However, they cannot be extended or implemented from (i.e., they are closed to modification after their declaration).

To define a type alias use the `type` keyword. When naming the type, then convention seems to be `UpperCamelCase` but it is not enforced in TypeScript itself:

```typescript
type Theme = 'dark' | 'light';
```

Here we're defining an object:

```typescript
type Point = {
  x: number;
  y: number;
};

function drawPoint(point: Point) {
  // Function implementation
}
```

Note the use of semi-colons `;` instead of commas `,`. 

Types can include *method signatures* which can be defined using two slightly different syntaxes:

```typescript
// type alias with a method signature
type Type1 = {
  id: number;
  greet: (name: string) => void;
};

// type alias with a method signature - shorthand syntax
type Type2 = {
  id: number;
  greet(name: string): void;
};
```

Also:

```typescript
// Types aliases can be used within other types:
type Theme = 'light' | 'dark' | 'medium';

// A property can be marked as optional using `?`
type User = {
  id: number;
  name: string;
  theme: Theme;
  greeting?: () => void;
  // Shorthand:
  // greeting?(): void;
};

// An object that uses the custom type (without optional function)
const user1: User = {
  id: 1,
  name: 'John',
  theme: 'dark'
};

// An object that uses the custom type (with optional function)
const user2: User = {
  id: 2,
  name: 'Bob',
  theme: 'medium',
  greeting: () => {
    console.log('Hello');
  }
};

// A type defined inline is considered anonymous:
const admin: {
  id: number;
  name: string;
  theme: Theme;
  roles: string[];
} = {
  id: 1,
  name: 'Mary',
  theme: 'light',
  roles: ['Developer', 'Lead']
};

// Note that when passing a type like this, TypeScript is not checking to see
// that the parameter IS of type User, but that it contains the same properties 
// of type User. For this reason, we can call `greetUser()` with `admin`.
function greetUser(user: User) {
  console.log('Hi ' + user.name);
  if (user.greeting) {
    user.greeting();
  }
}

greetUser(user1);  // Hi John
greetUser(user2);  // Hi Bob; Hello
greetUser(admin);  // Hi Mary
```

## Type interfaces

Type interfaces are similar to type aliases, but there are a few differences:

- Syntax: Defined using the `interface` keyword. Also note no assignment `=`.
- Use Case: Primarily used for defining the shape of objects (not unions or tuples).
- Extension: Can be extended or implemented by other interfaces or classes (they are open and can be augmented).


```typescript
interface Point {
  x: number;
  y: number;
};

function drawPoint(point: Point) {
  // Function implementation
}
```

Same as with Type alias, interfaces can include *method signatures* which can be defined using two slightly different syntaxes:

```typescript
// type interface with a method signature
interface Example1 {
  id: number;
  greet: (name: string) => void;
}

// type interface with a method signature - shorthand syntax
interface Example2 {
  id: number;
  greet(name: string): void;
}
```

Use *type aliases* when you need to describe a type that might not be an object or when you need a union or tuple type. Use *interfaces* when you want to define the shape of objects and need the ability to extend or implement them in other interfaces or classes. 

## Function type

The global type `Function` describes properties like bind, call, apply, and others present on all function values in JavaScript. It also has the special property that values of type `Function` can always be called; these calls return `any`:

```typescript
function doSomething(f: Function) {
  return f(1, 2, 3);
}
```

This is an untyped function call and is generally best avoided because of the unsafe `any` return type.

If you need to accept an arbitrary function but don’t intend to call it, the type `() => void` is generally safer.

## void

`void` represents the return value of functions which don’t return a value. It’s the inferred type any time a function doesn’t have any return statements, or doesn’t return any explicit value from those return statements:

```typescript
// The inferred return type is void
function noop() {
  return;
}

```

In JavaScript, a function that doesn’t return any value will implicitly return the value undefined. However, void and undefined are not the same thing in TypeScript.

## never

Some functions never return a value:

```typescript
function fail(msg: string): never {
  throw new Error(msg);
}
```

The `never` type represents values which are never observed. In a return type, this means that the function throws an exception or terminates execution of the program.

## Enums

> Enums are a feature added to JavaScript by TypeScript which allows for describing a value which could be one of a set of possible named constants. Unlike most TypeScript features, this is not a type-level addition to JavaScript but something added to the language and runtime. Because of this, it’s a feature which you should know exists, but maybe hold off on using unless you are sure.

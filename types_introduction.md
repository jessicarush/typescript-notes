# Types

## Table of contents

<!-- toc -->

- [Primitive (basic) types](#primitive-basic-types)
- [Arrays](#arrays)
- [Tuples](#tuples)
- [Functions](#functions)
- [Any and unknown](#any-and-unknown)
- [Type aliases](#type-aliases)
- [never ?](#never-)
- [Enums ?](#enums-)

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
// it can't be inferred. The return value however, could be inferred:
function greet(name: string) {
    return ('Hello ' + name)
}
// If I want to explicitly express the return value:
function greet(name: string): string {
    return ('Hello ' + name)
}
// As a function expression:
const greet = (name: string): string => {
  return 'Hello ' + name;
};
// If you use default params, the parameter type can be inferred:
function greet(name = 'Friend') {
  console.log('Hello ' + name);
}
```

So, should you let the return value be inferred or explicitly define it?

- **Simple Functions**: For simpler functions where the return type is obvious, the implicit return type (first approach) is often used due to its conciseness.
- **Complex Functions**: In more complex functions, or in codebases that prioritize explicit type annotations for clarity and maintainability, the explicit return type (second approach) is preferred.
- **Project or Team Standards**: The choice can also depend on the coding standards of a project or team. Some teams might enforce explicit types for consistency and clarity, while others might opt for the brevity of implicit types where appropriate.

```typescript
// Since the function has been defined, the type will get inferred here:
let greeting = greet('John');
```

When using the rest operator, remember that you must make the type an array. This really just makes it so that you pass individual string params to teh function instead of an array. For example:

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

Type aliases give us a way to create a new name for a type or to define the shape of an object. Type aliases are often used for complex types that you want to reuse throughout your code. To define a type alias use the `type` keyword:

```typescript
type MyType = string | number;
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




## never ?

## Enums ?

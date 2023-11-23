# Advanced types

## Table of contents

<!-- toc -->

- [Readonly](#readonly)
- [Index signatures](#index-signatures)
- [keyof operator](#keyof-operator)
- [Conditional types](#conditional-types)
- [Infer](#infer)
- [Literal types](#literal-types)
- [Mapped types](#mapped-types)
- [Utility types](#utility-types)
  * [Record](#record)

<!-- tocstop -->

## Readonly 

TypeScript has a `readonly` modifier for properties.

```typescript
interface User {
  readonly id: number;
  name: string;
}

const user: User = {id: 1, name: 'jessica'};
user.name = 'jeffery';
// user.id = 2; // error
```

`Readonly<T>` makes *all* properties readonly:

```typescript
interface User {
  id: number;
  name: string;
}

const user: Readonly<User> = {id: 1, name: 'jessica'};
// user.name = 'jeffery'; // error
// user.id = 2; // error
```

There's also a special `ReadonlyArray<T>` type that removes side-affecting methods and prevents writing to indices of the array:

```typescript
let a: ReadonlyArray<number> = [1, 2, 3];
let b: readonly number[] = [1, 2, 3];
// a.push(4); // error
// a[0] = 4; // error
// b.push(4); // error
// b[0] = 4; // error
```

You can also use a *const-assertion*, which operates on arrays and object literals:

```typescript
const user = {
  id: 1,
  name: 'jessica'
} as const;
// user.id = 2; // error
// user.name = 'jeffery'; // error

let a = [1, 2, 3] as const;
// a.push(4); // error
// a[0] = 4; // error
```

See how cont can be used in lieu of assigning a type:

```typescript
type Employee = {
  name: string;
  position: 'Programmer' | 'Manager' | 'HR' | 'Admin';
};

function paySalary(employee: Employee) {
  console.log(`Paying ${employee.name}...`);
}

// ❌ paySalary(employee) will error because even though object structure matches
// the structure of Employee, with this object, there's nothing stopping us from
// assigning a different value to employee.position or employee.name later on.
// const employee = {
//   name: 'Jessica',
//   position: 'Programmer'
// };

// ✅ This works because obviously
// const employee: Employee = {
//   name: 'Jessica',
//   position: 'Programmer'
// };

// ✅ This also works because not only does the object structure match, but we
// are also preventing any changes to the properties.
const employee = {
  name: 'Jessica',
  position: 'Programmer'
} as const;

paySalary(employee);
```

## Index signatures

Sometimes you don’t know all the names of a type’s properties ahead of time, but you do know the shape of the values. In those cases you can use an index signature to describe the types of possible values, for example:

```typescript
type Employee = {
  name: string;
  position: string;
  // extend the object
  [key: string]: string | number;
};

const john: Employee = {
  name: 'John',
  position: 'programmer',
  // extend the object
  email: 'john@company.com',
  age: 30
};
```

If you were working with an array, the index signature would be a number: 

```typescript
type StringArray = {
  [index: number]: string;
}

const myArray: StringArray = ['a', 'b', 'c'];
```

## keyof operator

The `keyof` operator takes an *object type* and produces a string or numeric *literal union* of its keys. 

The following type `P` is the same type as `type P = 'x' | 'y'`:

```typescript
type Point = { x: number; y: number };
type P = keyof Point;
```

This function uses `keyof` and generic constraints to ensure that we are warned by TypeScript if we try to pass a key that doesn't exist:

```typescript
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}
 
let x = { a: 1, b: 2, c: 3, d: 4 };
 
getProperty(x, 'a'); // will give us an Error if we don't pass a valid key
```

## Conditional types

Conditional types enable you to define types dynamically based on the values or properties of other types. These take a form that looks a little like conditional expressions in JavaScript:

```typescript
SomeType extends OtherType ? TrueType : DefaultType;
```

If the type to the left of `extends` is assignable to the type on the right, then you’ll get the `TrueType`; otherwise you’ll get the `DefaultType`.

```typescript
type Employee = {
  name: string;
  salary: number;
};

type Intern = {
  name: string;
  tasks: string[];
};

type SalaryOf<T> = T extends { salary: any } ? T['salary'] : never;

let someSalary: SalaryOf<Employee>; // type number
let noSalary: SalaryOf<Intern>; // type never
```

Note you can have multiple conditions:

```typescript
type User = {
  name: string;
  age: number;
};

type ArrayOfNames<T> = T extends User
  ? Array<T['name']>
  : T extends string
  ? T[]
  : never;

let test1: ArrayOfNames<User>; // string[]
let test2: ArrayOfNames<string>; // string[]
let test3: ArrayOfNames<number>; // never
```

Here's another example that uses `infer` which is described next.

```typescript
type User = {
  name: string;
  age: number;
  special: number;
};

type Admin = {
  name: string;
  age: number;
  // special property is optional, therefor the backup type is used
  special?: number;
}

type ExtractSpecial<T> = T extends { special: infer S } ? S : string;

// Example usage:
let test1: ExtractSpecial<User>; // type number
let test2: ExtractSpecial<Admin>; // type string
```

## Infer

The `infer` keyword is used in conditional types to declare a *type variable* that can be inferred based on a condition. This allows you to capture and assign the inferred type.

```typescript
type MyConditionalType<T> = T extends SomeType<infer U> ? U : DefaultType;
```

Let's say we want to create a type that will be a tuple of the values from another type:

```typescript
type User = {
  name: string;
  age: number;
};

type UserValues<T> = T extends { name: infer Name; age: infer Age }
  ? [Name, Age]
  : never;

let userValues: UserValues<User>; // userValues: [string, number]
```

Lets say we want to infer the type of items in an Array or Promise:

```typescript
type UnpackArray<T extends Array<any>> = T extends (infer R)[] ? R : never;
type UnpackPromise<T extends Promise<any>> = T extends Promise<infer R> ? R : never;

let test1: UnpackArray<string[]>; // type string
let test2: UnpackPromise<Promise<string>>; // type string

type Unpack<T> = T extends (infer R)[] ? R : T extends Promise<infer R> ? R : T;

let test3: Unpack<string>; // type string
let test4: Unpack<string[]>; // type string
let test5: Unpack<Promise<string>>; // type string
```

This example shows how you can use `infer` for part of a string using string interpolation.

```typescript
type Emails = 'Bob@company.com' | 'Sally@company.com' | 'Frank@company.com';

type GetNames<T> = T extends `${infer N}@company.com` ? N : never;

type Names = GetNames<Emails>; // type Names = "Bob" | "Sally" | "Frank"
```

You can even use infer more than once:

```typescript
type Emails = 'Bob@companyA.com' | 'Sally@companyB.com' | 'Frank@companyC.com';

type GetNamesAndCos<T> = T extends `${infer N}@${infer C}` ? [N, C] : never;

type NamesAndCos = GetNamesAndCos<Emails>; 
// type NamesAndCos = ["Bob", "companyA.com"] | ["Sally", "companyB.com"] | ["Frank", "companyC.com"]
```

## Literal types 

## Mapped types 

## Utility types

### Record<Keys, Type>

Constructs an object type whose property keys are `Keys` and whose property values are `Type`. This is helpful because if we add names to our `CatName` Keys, TypeScript will warn us that we need to add that new key to out Record.

```typescript
type CatName = 'miffy' | 'boris' | 'mordred';

interface CatInfo {
  age: number;
  breed: string;
}

const cats: Record<CatName, CatInfo> = {
  miffy: { age: 10, breed: 'Persian' },
  boris: { age: 5, breed: 'Maine Coon' },
  mordred: { age: 16, breed: 'British Shorthair' }
};
```


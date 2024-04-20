# Advanced types

## Table of contents

<!-- toc -->

- [Readonly](#readonly)
- [Index signatures](#index-signatures)
- [keyof operator](#keyof-operator)
- [Conditional types](#conditional-types)
- [Infer](#infer)
- [String literal types](#string-literal-types)
- [Mapped types](#mapped-types)
- [Utility types](#utility-types)
  * [Required](#required)
  * [Partial](#partial)
  * [Readonly](#readonly)
  * [Pick](#pick)
  * [Omit](#omit)
  * [ReturnType](#returntype)
  * [Awaited](#awaited)
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

See how `const` can be used in lieu of assigning a type:

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

## String literal types 

You can use string interpolation to build string types:

```typescript
type Level = 'Junior' | 'Senior' | 'Expert';
type Position = 'Programmer' | 'Admin' | 'Manager';

type LeveledPosition = `${Level} ${Position}`;

let test1: LeveledPosition = 'Junior Admin';

type Prefixed<P extends string, T extends string> = `${P} ${T}`;

let test2: Prefixed<'Awesome', Position> = 'Awesome Admin';
```

There are also a few *intrinsic string manipulation* types (these are also considered [utility types](#utility-types)):

```typescript
let lowerCasePositions: Lowercase<Position> = 'admin';
let upperCasePositions: Uppercase<Position> = 'ADMIN';
let uncapitalizedPositions: Uncapitalize<Position> = 'admin';
let capitalizedPositions: Capitalize<Position> = 'Admin';
```

## Mapped types 

A mapped type is a generic type which uses a union of `PropertyKey`s (frequently created via a `keyof`) to iterate through keys to create a type. You can iterate over a type alias/interface or an object value:

```typescript
type StarterMapType<T> = {
  [key in keyof T]: T[key];
};
```

The mapped type above simply returns a type that exactly the same as `T`. This shows us the pattern though and is a good starting point for when you want to make a new type.

```typescript
type MyMappedType<T> = {
  [K in keyof T]: T[K] | null;
};

const originalObject = {
  name: "John",
  age: 30,
  email: "john@example.com",
};

type NullableObject = MyMappedType<typeof originalObject>;
// type NullableObject = {
//   name: string | null;
//   age: number | null;
//   email: string | null;
// }
```

In this example, we define a mapped type `MyMappedType` that takes an input type `T` and creates a new type. It iterates over the keys (`name`, `age`, `email`) of the input type `T` using the `keyof` operator and transforms each property's type to include `null`. So, `NullableObject` will have the same keys as `originalObject`, but each property's type will be `T[K] | null`.

When you're mapping over a regular object (an instance of an object), use `typeof` to capture its type.

We can iterate over other types too, for example:

```typescript
type Role = 'Programmer' | 'Admin' | 'Manager';

// If we hardcode this, we have to remember to update both when new roles are added:
// type RoleDuties = {
//   Programmer: string[];
//   Admin: string[];
//   Manager: string[];
// };

type RoleDutiesMap = {
  [role in Role]: string[];
};
// type RoleDutiesMap = {
//   Programmer: string[];
//   Admin: string[];
//   Manager: string[];
// }

type GenericMap<T extends string> = {
  [key in T]: string[];
};

type RoleDuties = GenericMap<Role>;
// type RoleDuties = {
//   Programmer: string[];
//   Admin: string[];
//   Manager: string[];
// 
```

Note we are having to say `<T extends string>` because it restricts the type parameter `T` to only accept string literal types or unions of string literals. 

This type map makes a copy of another object type but with `readonly` properties:

```typescript
type ReadonlyMapType<T> = {
  readonly [key in keyof T]: T[key];
};
```

You could also remove any existing `readonly` keywords:

```typescript
type MutableMapType<T> = {
  -readonly [key in keyof T]: T[key];
};
```

You can also do *key remapping* using the `as` keyword:

```typescript
type User = {
  name: string;
  age: number;
};

type Verbose<T> = {
  [key in keyof T as `user${Capitalize<string & key>}`]: T[key];
};

type VerboseUser = Verbose<User>;
// type VerboseUser = {
//   userName: string;
//   userAge: number;
// }
```

Note the `<string & key>` in the *intrinsic string manipulation*. With this we ensure that the resulting keys are not just the original keys but are also of type string. This is important because the Capitalize utility type expects a string as input.

Mapped types are useful for scenarios where you want to make certain properties optional, change their types, or perform other transformations while preserving the overall structure of the original type.

## Utility types

There are many [utility types](https://www.typescriptlang.org/docs/handbook/utility-types.html). These are just a few.

Let's start with a type alias of an object that has some optional properties:

```typescript
type Employee = {
  name: string;
  position: string;
  salary: {
    amount: number;
    currency: string;
    bonus?: 10 | 20 | 30;
  };
  isAdmin: boolean;
  employedAt: string;
  team?: string;
};
```

### Required<Type>

```typescript
type RequiredEmployee = Required<Employee>;
// type RequiredEmployee = {
//     name: string;
//     position: string;
//     salary: {
//         amount: number;
//         currency: string;
//         bonus?: 10 | 20 | 30;
//     };
//     isAdmin: boolean;
//     employedAt: string;
//     team: string;
// }
```

### Partial<Type>

```typescript
type OptionalEmployee = Partial<Employee>;
// type OptionalEmployee = {
//     name?: string | undefined;
//     position?: string | undefined;
//     salary?: {
//         amount: number;
//         currency: string;
//         bonus?: 10 | 20 | 30 | undefined;
//     } | undefined;
//     isAdmin?: boolean | undefined;
//     employedAt?: string | undefined;
//     team?: string | undefined;
// }
```

### Readonly<Type>

```typescript
type ReadonlyEmployee = Readonly<Employee>;
// type ReadonlyEmployee = {
//     readonly name: string;
//     readonly position: string;
//     readonly salary: {
//         amount: number;
//         currency: string;
//         bonus?: 10 | 20 | 30;
//     };
//     readonly isAdmin: boolean;
//     readonly employedAt: string;
//     readonly team?: string | undefined;
// }
```

### Pick<Type, Keys>

```typescript
type SalaryPick = Pick<Employee, 'salary'>;
// type SalaryPick = {
//   salary: {
//       amount: number;
//       currency: string;
//       bonus?: 10 | 20 | 30;
//   };
// }
type Salary = Employee['salary'];
// type Salary = {
//   amount: number;
//   currency: string;
//   bonus?: 10 | 20 | 30 | undefined;
// }
```

You acn also pick multiple properties:

```typescript
type NameSalaryPick = Pick<Employee, 'name' | 'salary'>;
// type SalaryPick = {
//   name: string;
//   salary: {
//       amount: number;
//       currency: string;
//       bonus?: 10 | 20 | 30;
//   };
// }
type NameSalary = {
  name: Employee['name'];
  salary: Employee['salary'];
};
```

### Omit<Type, Keys>

```typescript
type SanitizedEmployee = Omit<Employee, 'employedAt'>;
// type SanitizedEmployee = {
//   salary: {
//       amount: number;
//       currency: string;
//       bonus?: 10 | 20 | 30;
//   };
//   name: string;
//   position: string;
//   isAdmin: boolean;
//   team?: string | undefined;
// }
```

To omit multiple properties use a pipe (`|`): `Omit<Employee, 'employedAt' | 'salary'>;`.

You can also use Omit to replace a property:

```typescript
type SanitizedEmployee = Omit<Employee, 'employedAt'> & {
  employedSince: Date;
};
// type SanitizedEmployee = {
//   salary: {
//       amount: number;
//       currency: string;
//       bonus?: 10 | 20 | 30;
//   };
//   name: string;
//   position: string;
//   isAdmin: boolean;
//   employedSince: Date;
//   team?: string | undefined;
// }
```

### ReturnType<Type>

```typescript
function getData(id: string) {
  // Fetching data...
  return {
    name: 'Bob',
    position: 'Developer',
    salary: 150000,
    teams: ['x', 'y']
  };
}

type Person = ReturnType<typeof getData>;
// type Person = {
//   name: string;
//   position: string;
//   salary: number;
//   teams: string[];
// }
```

Note, as with our first [Mapped type](#mapped-types) example where we're mapping over an object rather than an type object, we have to use the `typeof` keyword to ensure TypeScript gets the types.

### Awaited<Type>

This type is meant to model operations like `await` in `async` functions, or the `.then()` method on Promises - specifically, the way that they recursively unwrap Promises.

```typescript
async function getEmployees() {
  return Promise.resolve([
    {
      name: 'John',
      position: 'Programmer',
      salary: 100000
    }
  ]);
}

async function wrapper() {
  const employees = await getEmployees();
}

type EmployeeReturnType1 = ReturnType<typeof getEmployees>;
// type EmployeeReturnType = Promise<{
//     name: string;
//     position: string;
//     salary: number;
// }[]>

type EmployeeReturnType2 = Awaited<ReturnType<typeof getEmployees>>;
// type EmployeeReturnType2 = {
//   name: string;
//   position: string;
//   salary: number;
// }[]
```

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

For a simpler, practical example, I had a `jsx` component that I was converting to a `tsx` component. In it, I am accepting some props and setting some css variables based on those props by dynamically adding them to an object:

```jsx
export default function Button(props) {
  const {
    id,
    color,
    hoverColor,
    uppercase,
    variant = 'filled',
    children,
    onClick,
    ...rest
  } = props;

  const styles = {};

  if (typeof color === 'string') {
    styles[`--${variant}-color`] = color; // <-- typescript says no!
  }
  // ...
```

When converting to typescript, it gets mad because it thinks `styles` should be an empty object. To tell typescript that I want this object to contain key value pairs that are strings (e.g. `'--filled-color': '#000'`), I use `Record<string, string>`:

```tsx
interface ButtonProps extends Omit<ButtonHTMLAttributes<HTMLButtonElement>, 'color'> {
  id: string;
  color?: string | string[];
  hoverColor?: string | string[];
  uppercase?: boolean;
  variant?: 'outline' | 'underline' | 'filled';
}

export default function Button(props: ButtonProps) {
  const {
    id,
    color,
    hoverColor,
    uppercase,
    variant = 'filled',
    children,
    onClick,
    ...rest
  } = props;

  const styles: Record<string, string> = {};

  if (typeof color === 'string') {
    styles[`--${variant}-color`] = color;
  }
  // ...
```

Btw if you're wondering about the `extends Omit<ButtonHTMLAttributes<HTMLButtonElement>, 'color'>` part: 

Extending `<ButtonHTMLAttributes<HTMLButtonElement>` (where `ButtonHTMLAttributes` is imported from `react`), is a standard way to include all the standard Button attributes. This way I can accept props like `children` and `onClick` without specifically adding them to my interface. You'll notice I'm also using the `...rest` variable. This collects all other properties that haven't been destructured explicitly into a new object. I then apply that to my button:

```tsx
<button
  // ...
  onClick={onClick}
  style={styles}
  {...rest}>
    {children}
</button>
```

Lastly, I'm omitting the color attribute from the `ButtonHTMLAttributes` because technically the color attribute is deprecated and I happen to be using it differently, as in I am accepting a string or and array of strings so I need to override the old one for typescript to stfu.

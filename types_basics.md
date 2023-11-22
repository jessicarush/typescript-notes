# Types

## Table of contents

<!-- toc -->

- [Literal types](#literal-types)
- [Union types](#union-types)
- [Default values with types](#default-values-with-types)
- [Call signatures](#call-signatures)
- [Type narrowing ()](#type-narrowing-)
- [Type predicates and type assertion](#type-predicates-and-type-assertion)
- [Optional values](#optional-values)
- [Type interfaces](#type-interfaces)
- [Type intersection](#type-intersection)
- [Type interface extension vs type alias intersection](#type-interface-extension-vs-type-alias-intersection)
- [Enums](#enums)
- [never](#never)

<!-- tocstop -->

## Literal types 

```typescript
type Role = 'developer';
```

## Union types

```typescript
type Role = 'developer' | 'admin' | 'manager';
```

## Default values with types

```typescript
function getTheme(theme: 'dark' | 'light' = 'dark' ): void {
  if (theme === 'dark') {
    // do something
  }
  if (theme === 'light') {
    // do something
  }
}
```

## Call signatures

While I don't forsee wanting to do this, it should be noted that you can define a function's *call signature* in a separate type alias or interface, for example:

```typescript
type ThemeFunction = {
  (theme: 'dark' | 'light'): void;
}

// interface ThemeFunction {
//   (theme: 'dark' | 'light'): void;
// }

const getTheme: ThemeFunction = (theme = 'dark') => {
  if (theme === 'dark') {
    // do something
  }
  if (theme === 'light') {
    // do something
  }
}
```

That syntax seen in the Type alias and interface above is specifically a function call signature, not to be confused with a *method signature*:

```typescript
interface Example1 {
  id: number;
  // method signature
  greet: (name: string) => void;
}

interface Example2 {
  id: number;
  // method signature - shorthand syntax
  greet(name: string): void;
}

interface Example3 {
  // function call signature
  (name: string): void;
}
```

Note that if a Type alias or interface defines a function call signature, you will not have other properties in there as it would just require the function to also have those properties which we never really do in JavaScript (even though technically you can).

## Type narrowing ()

Literal type narrowing with `===`:

```typescript
function getTheme(theme: 'dark' | 'light'): void {
  if (theme === 'dark') {
    // do something
  }
  if (theme === 'light') {
    // do something
  }
}
```

Primitive type narrowing with `typeof`. Remember possible string values for `typeof` are:

- `string`
- `number`
- `bigint`
- `boolean`
- `symbol`
- `undefined`
- `object`
- `function`

```typescript
function getNumberValue(arg: unknown): number | never {
    if (typeof arg === 'number') {
        return arg;
    }
    if (typeof arg === 'string') {
        return Number(arg)
    }
    throw new Error(`Unsupported format: ${JSON.stringify(arg)}`);
}
```

Class type narrowing with `instanceof`:

```typescript
class Product {
  getDescription() {
    return 'Product...';
  }
}

class Activity {
  getDescription() {
    return 'Activity...';
  }
}

type Entries = Product | Activity;

function viewEntries(entry: Entries): void {
  if (entry instanceof Product) {
    // do something
  }
  if (entry instanceof Activity) {
    // do something
  }
}

viewEntries(new Product());
viewEntries(new Activity());
```

Object type narrowing:

```typescript
type Founder = {
    name: 'John Founder';
    car: 'Audi';
  } | {
    name: 'Bill Bicycle';
    bike: 'Wheels';
  };

function meetTheFounder(founder: Founder) {
  console.log(`Meet your fonder ${founder.name}`);
  if (founder.name === 'John Founder') {
    // do something
    console.log('John');
  }
  if ('bike' in founder) {
    // do something
    console.log(`bike: ${founder.bike}`);
  }
}

meetTheFounder({
  name: 'John Founder',
  car: 'Audi'
});

meetTheFounder({
  name: 'Bill Bicycle',
  bike: 'Wheels'
});
```

## Type predicates and type assertion

To define a user-defined type guard, we need to define a function whose return type is a *type predicate* (`pet is Fish`). When type predicates are used like this, we must return a boolean that answers the question. In order to check the arg for properties we have to use a *type assertion* (`pet as Fish`). It tells TypeScript to treat the arg as if it is of type Fish (even though it may not match the structure of Fish, it bypasses the TypeScript type checking so I can do the `.swim !== undefined` check):

```typescript
function isFish(pet: Fish | Bird): pet is Fish {
  return (pet as Fish).swim !== undefined;
}
```

In this next example, the parameter arg is of type `any`. This allows you to freely access properties of arg without TypeScript errors. However, using `any` can potentially lead to runtime errors if arg does not have the expected properties, as `any` bypasses TypeScript's type checking.

```typescript
type Salary = {
  amount: number;
};

// ❌ Not the best approach:
function isSalary(arg: any): arg is Salary {
  return 'amount' in arg && typeof arg.amount === 'number';
}

function paySalary(arg: unknown) {
  if (isSalary(arg)) {
    console.log(`Paying ${arg.amount}`);
  }
}
```

If we switch `any` to `unknown` then we have to switch to type assertions:

```typescript
// ❌ This is ok but could be even better:
function isSalary(arg: unknown): arg is Salary {
  return (
    (arg as Salary).amount !== undefined &&
    typeof (arg as Salary).amount === 'number'
  );
}
```

If we want handle this properly:

```typescript
// ✅ This covers all possible scenarios:
function isSalary(arg: unknown): arg is Salary {
  return (
    typeof arg === 'object' &&
    arg !== null &&
    'amount' in arg &&
    typeof (arg as Salary).amount === 'number'
  );
}
```

To explain:

First do an initial type check for Object:` typeof arg === 'object' && arg !== null` to ensure that `arg` is an `object` before proceeding with further checks. This is a common pattern when dealing with unknown types, as it guards against arg being a primitive, `null`, or `undefined`.

Then we can safely use the `in` operator to check for the existence of the amount property in arg. This is a concise way to check for property existence on an object.

Finally, type assertion `arg as Salary` is now only used once, in the final check for the type of amount.

Now consider this:

```typescript
type SimpleJob = {
  language: string;
  sourceControl: string;
};

type ComplicatedJob = {
  language: string;
  sourceControl: string;
  dailyMeetings: boolean;
  reports: string[];
};

let simpleJob: SimpleJob = { language: 'TS', sourceControl: 'git' };

let complicatedJob: ComplicatedJob = {
  language: 'TS',
  sourceControl: 'git',
  dailyMeetings: true,
  reports: ['daily', 'weekly']
};

// We can safely assign complicatedJob to simpleJob because it had all the 
// required properties
simpleJob = complicatedJob;
// This will result in a TypeScript error because simpleJob is missing properties
// required in complicatedJob
complicatedJob = simpleJob;
// Type assertion is a way of telling TypeScript, "Trust me, I know what I'm 
// doing; treat simpleJob as if it is of type ComplicatedJob". This bypasses 
// TypeScript's normal type checking. It allows you to do this because it can see
// that they have at least one property in common.
complicatedJob = simpleJob as ComplicatedJob;
// Alternate generic syntax:
complicatedJob = <ComplicatedJob>simpleJob;
```

> Important Note:
>
> - Type Safety: Type assertions should be used carefully. They can bypass TypeScript's type safety checks, potentially leading to runtime errors. In this case, since simpleJob doesn't have all the properties of ComplicatedJob, using it as such could lead to issues if the code expects those missing properties to be present.
> - Use Case: This approach is useful when you are certain about the nature of the data and TypeScript cannot infer it correctly. However, it's important to ensure that the data really conforms to the asserted type to avoid errors.

That being said, if you try to do an assertion where the objects have no properties in common, you will still get a TypeScript error.

```typescript
type SimpleJob = {
  language: string;
  sourceControl: string;
};

type QaJob = {
  scriptingLanguage: string;
  hasAutomatedTests: true;
};

let simpleJob: SimpleJob = { language: 'TS', sourceControl: 'git' };

let qaJob: QaJob = { scriptingLanguage: 'Python', hasAutomatedTests: true };

// This won't work because they have no properties in common
simpleJob = qaJob as simpleJob;
// This can be worked around using a double assertion
simpleJob = qaJob as unknown as simpleJob;
// Alternate generic syntax:
complicatedJob = <ComplicatedJob>(<unknown>qaJob);
```

While this practice is generally discouraged, there could be rare scenarios where it might be used:

- Interoperating with Dynamically Typed Code: When integrating with parts of a codebase that are dynamically typed or when dealing with data from external sources (like APIs) where you might not have control over the data structure.
- Working with Legacy Code: In cases where you're gradually migrating JavaScript code to TypeScript and encounter scenarios where strict typing is not immediately feasible.
- Testing and Mocking: Sometimes in testing, particularly when mocking objects or creating test fixtures, you might need to force types into a certain shape for the sake of the test environment.

One last use case for type assertions is if you're building an object in steps:

```typescript
// usage: build objects in steps
// advantage: autocomplete assistance
// disadvantage: the compiler won't complain about missing properties
const complicatedJob = {} as ComplicatedJob;
complicatedJob.language = 'TS';
complicatedJob.dailyMeetings = true;
complicatedJob.reports = [];
complicatedJob.sourceControl = 'git';
```

## Optional values

```typescript
// Optional properties
type Employee = {
  id: number;
  currency: 'USD' | 'EUR';
  yearlyBonus?: number;
};

// Optional parameters
function paySalary(employee: Employee, special?: string) {
  if (special) {
    // do something
  }
}

// Optional values in classes:
class Engineer {
  tasks?: string[];
}
```

Also should remind about optional chaining, even though that is a native JS feature (as of ES202).

```typescript
type Manager = {
  team?: {
    scrumMaster?: {
      holdScrumMeeting: () => void;
    };
  };
};

// ? - JavaScript optional operator
function manage(manager: Manager) {
  manager.team?.scrumMaster?.holdScrumMeeting();
}
```

## Type interfaces

Type interfaces differ from type aliases in that they:

- Are primarily used for defining the shape of objects (not unions or tuples).
- Can be extended or implemented by other interfaces or classes (they are open and can be augmented).

```typescript
interface User {
  id: number;
  name: string;
  theme: 'light' | 'dark';
};

let user: User = {
  id: 1,
  name: 'Bob',
  theme: 'dark'
}
```

Interfaces can be extended using the `extends` keyword: 

```typescript
interface Circle {
  radius: number;
}

interface ColorfulCircle extends Circle {
  color: string;
}

const redCircle: ColorfulCircle = {
  color: 'red',
  radius: 10
} 
```

Interface declaration merging: 

```typescript
interface Point {
  x: number;
  y: number;
};

interface Point {
  showPoint: () => void;
}

let location1: Point = {
  x: 100,
  y: 100,
  showPoint: () => {
    // show point
  }
}
```

I could see this being useful if you're working with some TypeScript library and you want to merge something into their interface, but I wouldn't want to do this within my own code.

## Type intersection

Interfaces allow us to build up new types from other types by *extending* them. TypeScript provides another construct called intersection types that is mainly used to combine existing object types.

An intersection type is defined using the `&` operator and can be used with type aliases or type interfaces. For example:

```typescript
interface Colorful {
  color: string;
}
interface Circle {
  radius: number;
}
 
type ColorfulCircle = Colorful & Circle; // type intersection

const redCircle: ColorfulCircle = {
  color: 'red',
  radius: 10
} 

// or

type ColorfulRectangle = Colorful & { // type intersection
  width: number;
  height: number;
}

const greenRectangle: ColorfulRectangle = {
  color: 'green',
  width: 10,
  height: 20
};
```

Working from type aliases:

```typescript
type SimpleJob = {
    language: string,
    sourceControl: string
}

type ComplicatedJob = SimpleJob & { // type intersection
    dailyMeetings: true,
    reports: string[],
}

let complicatedJob: ComplicatedJob = {
    language: 'TS',
    sourceControl: 'git',
    dailyMeetings: true,
    reports: ['daily', 'weekly']
}
```

## Type interface extension vs type alias intersection

> We just looked at two ways to combine types which are similar, but are actually subtly different. With interfaces, we could use an `extends` clause to extend from other types, and we were able to do something similar with intersections and name the result with a *type alias*. The principal difference between the two is how conflicts are handled, and that difference is typically one of the main reasons why you’d pick one over the other between an interface and a type alias of an intersection type.

Specifically they mean how these constructs behave when there are discrepancies or overlapping properties in the types being combined.

Interfaces extending

- Conflict Resolution: When extending interfaces, if there are conflicting properties, TypeScript enforces that the properties must be of the same type. If the same property exists in multiple interfaces with different types, TypeScript will treat this as an error.

Type aliases with intersections

- Handling Conflicts: When you create a type alias using intersection types (using the & operator), TypeScript combines the properties of all types. If there are conflicts (the same property in multiple types but with different types), TypeScript will treat the type of that property as the intersection of the types from each type in the intersection. This often results in never type if the intersected types are incompatible, effectively flagging a type error.

```typescript
interface A {
    prop: string;
}

interface B extends A {
    prop: number; // Error: Type 'number' is not assignable to type 'string'.
}
```

vs 

```typescript
type A = { prop: string };
type B = { prop: number };

// Type of 'prop' in C is 'never' because string and number types are incompatible
type C = A & B; /
```

## Enums

Enums (short for "enumerations") in programming are a way to give more meaningful names to sets of numeric values. Enums allow you to define named constants. This makes your code more readable and understandable, as you can use descriptive names instead of numeric values.

For example:

```typescript
enum Day {
  Sunday,
  Monday,
  Tuesday,
  Wednesday,
  Thursday,
  Friday,
  Saturday
}
```

Now you can use `Day.Thursday` instead of `4`.

```typescript
type Employee = {
  name: string;
  position: string;
  workFromHome: Day;
}

let employee: Employee = {
  name: 'Bob',
  position: 'Developer',
  workFromHome: Day.Friday
}
```

By default, in many languages like TypeScript, the underlying value of each enum member is a number (starting from 0), but you can also manually assign different values to them.

When you stop assignments, the enum values will continue to increment up by 1:

```typescript
enum Day {
  Sunday = 1,
  Monday,
  Tuesday,
  Wednesday,
  Thursday,
  Friday,
  Saturday
}
```

For more information see the [typescript docs on enums](https://www.typescriptlang.org/docs/handbook/enums.html).

## never

The `never` type represents values which are never observed. In a return type, this means that the function throws an exception or terminates execution of the program.

There is an interesting use case with never and enums (though you could probably do something similar without enums):

```typescript
enum Role {
  Developer,
  Admin,
  CEO,
}

type Employee = {
  name: string;
  salary: number;
  role: Role;
};

function payAnnualBonus(employee: Employee) {
  let bonusPercent: number = 0;
  const role = employee.role;
  switch (role) {
    case Role.Developer:
      bonusPercent = 0.2;
      break;
    case Role.Admin:
      bonusPercent = 0.8;
      break;
    case Role.CEO:
      bonusPercent = 200;
      break;
    default:
      // Given the current options in Role, we would never get to this
      // default case. As a result, in this case the value of role is `never`.
      // So the assignment below is valid. But, if we were add another role
      // to Role, this would now throw and error, reminding us to update 
      // the switch to include the new role.
      const remainingValues: never = role;
      break;
  }
  console.log(`Paying ${employee.salary * bonusPercent}  bonus to ${employee.name}`);
}
```
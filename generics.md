# TypeScript Generics

Generics are an extra layer of abstraction over regular types. They allow you to create components that can work over a variety of types rather than a single one. This can help in reducing code duplication. They're also necessary for working with [advanced types](advanced_types.md).

## Table of contents

<!-- toc -->

- [Generic type variables](#generic-type-variables)
- [Common generic types](#common-generic-types)
- [Generic constraints](#generic-constraints)
- [Multiple types](#multiple-types)
- [Generic classes](#generic-classes)
- [Generic types and interfaces](#generic-types-and-interfaces)

<!-- tocstop -->

## Generic type variables

Generics use a type variable, like `<T>` or `<Type>`, as a placeholder. This allows us to capture the type the user provides (e.g. number), so that we can use that information later. This way, you can write functions, interfaces, or classes that can work with any type, decided at the time of use. We say that this function is generic, as it works over a range of types.

```typescript
function getArray<Type>(items: Type[]): Type[] {
  return new Array().concat(items);
}
```

Here, `Type` is a generic type, and `getArray` can work with any array, whether it's an array of numbers, strings, or any custom type.

Consider a function that converts any type args into an array of that type:

```typescript
function toArray(...args: any[]) {
  return args;
}

const myArray1 = toArray('My name');    // myArray is type any[]
const myArray2 = toArray(1);            // myArray is type any[]
const myArray3 = toArray('My name', 1); // myArray is type any[]
```

With a generic type variable:

```typescript
function toArray<Type>(...args: Type[]): Type[] {
  return args;
}

const myArray1 = toArray('Alex');    // myArray is type string[]
const myArray2 = toArray(1);         // myArray is type number[]
const myArray3 = toArray('Alex', 1); // Error Argument of type 'number' is not 
                                     // assignable to parameter of type 'string'
```

In the above function calls we are using *type argument inference*. The compiler looked at the value passed to "toArray", and set `Type` to its type. While type argument inference can be helpful to keep code shorter and more readable, you may need to explicitly pass in the type arguments when the compiler fails to infer the type, as may happen in more complex examples.For example, TypeScript currently does not infer a union type for rest parameters automatically. 

We can explicitly pass in the type when calling the function like so:

```TypeScript
const myArray1 = toArray<string>('Alex');             // myArray is type string[]
const myArray2 = toArray<number>(1);                  // myArray is type number[]
const myArray3 = toArray<string | number>('Alex', 1); // myArray is type (string | number)[]
```

Note that the function itself can also be written using generic syntax:

```typescript
function toArray<Type>(...args: Array<Type>): Array<Type> {
  return args;
}
```

## Common generic types

Whenever we write out types like `number[]` or `string[]`, it’s just a shorthand for `Array<number>` and `Array<string>`. `Array` itself is a generic type.

Modern JavaScript also provides other data structures which are generic, like `Map<K, V>`, `Set<T>`, and `Promise<T>`. All this really means is that because of how `Map`, `Set`, and `Promise` behave, they can work with any sets of types.

```typescript
type Employee = {
  name: string;
  role: string;
};

async function getEmployees<Type>(url: string): Promise<Type[]> {
  const result = await fetch(url);
  const parsedResult = await result.json();
  return parsedResult;
}

async function wrapper() {
  const employees = await getEmployees<Employee>('internalEmployeeService.com');
}
```

## Generic constraints

You may sometimes want to write a generic function that works on a set of types where you have *some* knowledge about what capabilities that set of types will/should have. For example:

```typescript
function logIdentity<Type>(arg: Type): Type {
  // ❌ Error: Property 'length' does not exist on type 'Type'.
  console.log(arg.length);
  return arg;
}
```

Instead of working with any and all types, we’d like to *constrain* this function to work with any and all types that also have the `.length` property. As long as the type has this member, we’ll allow it. To do so, we must list our requirement as a constraint on what Type can be.

First, create an interface that describes our constraint. Then, use this interface and the `extends` keyword to denote our constraint:

> Remember the `extends` keyword is also used to extend interfaces, allowing you to create new interfaces that inherit or combine the properties and methods of other existing interfaces.

```typescript
interface Lengthwise {
  length: number;
}

function logIdentity<Type extends Lengthwise>(arg: Type): Type {
  // ✅ Property 'length' exists on 'Type'.
  console.log(arg.length);
  return arg;
}
```

Instead of using an interface, you could also do it inline which makes sense for some short constraint:

```typescript
function logIdentity<Type extends { length: number }>(arg: Type): Type {
  // ✅ Property 'length' exists on 'Type'.
  console.log(arg.length);
  return arg;
}
```

Here, the `Type` generic type parameter is constrained. It ensures that you can only pass objects with a `length` property to the `logIdentity` function.

Here's a harder one that not only uses more than one parameter but also uses they `keyof` keyword instead of a traditional property constraint. The goal is to get a property from an object given its name while ensuring that we’re not accidentally grabbing a property that doesn't exist on the object.

```typescript
function getProperty<Type, Key extends keyof Type>(obj: Type, key: Key): Type[Key] {
  return obj[key];
}
 
let x = { a: 1, b: 2, c: 3, d: 4 };
 
getProperty(x, 'a'); // will give us an Error if we don't pass a valid key
```

This leads us to...

## Multiple types

Consider this:

```typescript
const roleA = {
  holdsMeetings: false,
  teams: ['Team1', 'team2']
};

const roleB = {
  holdsMeetings: true,
  reportsTo: 'person1'
};

function mergeRoles(role1: object, role2: object): object {
  return { ...role1, ...role2 };
}

const roleC = mergeRoles(roleA, roleB);
console.log(roleC.holdsMeetings); // ❌ Error: property doesn't exist on type 'object'
```

Instead, add two generic type variables and add constraints that they must be objects. Since were are merging the two objects, we will say that the return value is `T & G`:

```typescript
const roleA = {
  holdsMeetings: false,
  teams: ['Team1', 'team2']
};

const roleB = {
  holdsMeetings: true,
  reportsTo: 'person1'
};

function mergeRoles<T extends object, G extends object>(role1: T, role2: G): T & G {
  return {...role1, ...role2};
}

const roleC = mergeRoles(roleA, roleB);
console.log(roleC.holdsMeetings); // ✅ Has IntelliSense
```

Just a reminder when using type intersections `T & G`, if there are duplicate properties, they must be of the same type, otherwise TypeScript will assign a type of `never` to the resulting property.

As an exercise, I tries to find a way to handle this situation elegantly. I worked with ChatGPT for over an hour with no good solutions. In this case the best I could come up with wath using type narrowing:

```typescript
const roleA = {
  holdsMeetings: false, // boolean
  teams: ['Team1', 'team2']
};

const roleB = {
  holdsMeetings: 'true', // string
  reportsTo: 'person1'
};

// If mergeRoles is typed to return `T & G`, we het the following error:
// The intersection '{ holdsMeetings: boolean; teams: string[]; } & 
// { holdsMeetings: string; reportsTo: string; } & Record<"holdsMeetings", unknown>'
// was reduced to 'never' because property 'holdsMeetings' has conflicting types 
// in some constituents
function mergeRoles<T extends object, G extends object>(role1: T, role2: G): object {
  return {...role1, ...role2};
}

const roleC = mergeRoles(roleA, roleB);

if ('holdsMeetings' in roleC) {
  console.log(roleC.holdsMeetings);
}
```

In a real-world scenario, you would likely write some function to normalize the objects first I guess.

## Generic classes

Classes can use generic type variables too.

```typescript
class MemoryDatabase<T> {
  protected items = new Array<T>();

  public addItem(item: T) {
    this.items.push(item);
  }

  public getItemByIndex(index: number): T | undefined {
    return this.items[index];
  }

  public listItems() {
    this.items.forEach((item) => {
      console.log(item);
    });
  }
}

const names = new MemoryDatabase<string>();
names.addItem('John');
const firstname = names.getItemByIndex(0); // John


class MemoryDatabaseWithDelete<T extends { id: string }> extends MemoryDatabase<T> {

  public delete(id: string) {
    const index = this.items.findIndex((x) => x.id === id);
    this.items.splice(index, 1);
  }
}

const withIds = new MemoryDatabaseWithDelete<{ id: string }>();
withIds.addItem({ id: '123' });
const firstId = withIds.getItemByIndex(0); // { id: '123' }
```

## Generic types and interfaces

So how to we identify the type of a generic function? For example:

```typescript
function identity<Type>(arg: Type): Type {
  return arg;
}

let myIdentity = identity; // inferred
```

The same way you would identify a non-generic function. We need to define its *call signature*: the function's parameters and return type. We usually do this using the *arrow function syntax* `() => void`:

```typescript
function identity<Type>(arg: Type): Type {
  return arg;
}

let myIdentity: <Type>(arg: Type) => Type = identity;
```

However, you also have the option of defining the call signature using *object literal syntax*:

```typescript
let myIdentity: { <Type>(arg: Type): Type } = identity;
```

This leads us to generic interfaces. The object literal syntax from the previous example can be moved into an interface:

```typescript
function identity<Type>(arg: Type): Type {
  return arg;
}

interface GenericIdentityFn {
  <Type>(arg: Type): Type;
}

let myIdentity: GenericIdentityFn = identity;
```

This is an interface with a generic call signature. The interface itself is not generic, but the function within it is.

With interfaces, you also have the option to move the generic parameter to be a parameter of the whole interface making it a *generic interface*. This makes the type parameter visible to all the other members of the interface. If we do this though, we will need to specify the variable type when we use `GenericIdentityFn`:

```typescript
function identity<Type>(arg: Type): Type {
  return arg;
}

interface GenericIdentityFn<Type> {
  (arg: Type): Type;
}

let numberIdentity: GenericIdentityFn<number> = identity;
```

Deciding where to place generic type parameters can change how the generic behavior is applied and interpreted in your code. 

When you put a generic type parameter directly on the call signature of a function, the generic aspect is specific to that function. This means each time the function is used, it can be called with a different type.

When you define a new generic type parameter on an interface, the entire interface becomes generic. This means that when you implement or use this interface, you lock in a specific type for all members of the interface, potentially including multiple methods or properties.
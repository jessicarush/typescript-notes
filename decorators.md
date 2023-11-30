# TypeScript 5 decorators (stage 3)

As of this writing the [official docs](https://www.typescriptlang.org/docs/handbook/decorators.html#class-decorators) are still using experimental stage 2 decorators. Stage 3 is described in [this microsoft blog post](https://devblogs.microsoft.com/typescript/announcing-typescript-5-0/#decorators) but it is far from complete. These notes should be updated once the official docs get updated with stage 3 decorators.

I should also note that decorators don't appear in the Handbook, only the Reference, which makes me think this is more of an advanced, less used feature. 

**JavaScript does not currently support the full range mentioned here and instead only implements Class & Method Decorators.**

See also:

- [All you need to know about Typescript Decorators (as introduced in 5.0)](https://medium.com/@templum.dev/all-you-need-to-know-about-typescript-decorators-as-introduced-in-5-0-b03866a8b213)
- [TS 5.0 Beta: New Decorators Are Here](https://plainenglish.io/blog/ts-5-0-beta-new-decorators-are-here)

## Table of contents

<!-- toc -->

- [Introduction](#introduction)
- [Class decorators](#class-decorators)
- [Property/Field decorators](#propertyfield-decorators)
- [Decorator factories](#decorator-factories)
- [Method decorators](#method-decorators)
- [Accessor decorators](#accessor-decorators)

<!-- tocstop -->

## Introduction

In TypeScript, [decorators](https://www.typescriptlang.org/docs/handbook/decorators.html) can only be attached to a classes and class members (methods, accessors, properties, or parameters). Decorators use the form `@expression`, where `expression` must evaluate to a function that will be called at runtime with information about the decorated declaration.

For example, given the decorator `@sealed` we might write the sealed function as follows:

```typescript
function sealed(target) {
  // do something with 'target' ...
}
```

> Note that while in the experimental *stage 2* phase, decorators are very different from how they ended up in their final implementation in *stage 3*, TypeScript 5. When searching for information on decorators, be sure to specify *stage 3* (TypeScript 5). One key difference is that decorator function parameters are no longer supported in stage 3. To enable stage 2 decorators you would set the tsconfig option: `"experimentalDecorators": true`.

Decorators follow this pattern:

```typescript
type Decorator = (
  target: Input,
  context: {
    kind: string;
    name: string | symbol;
    access: {
      get?(): unknown;
      set?(value: unknown): void;
    };
    private?: boolean;
    static?: boolean;
    addInitializer?(initializer: () => void): void;
  }
) => Output | void;
```

Another way to describe it:

```typescript
function Decorator(target: unknown, context: unknown) {
  // The kind of element, this can be one of:
  // ['class', 'method', 'getter', 'setter', 'field', 'accessor']
  context.kind;

  // The element's name is either a Symbol or a String
  context.name;

  // An object containing either get, set or both properties. 
  // It allows for reading/writing the underlying value of an object
  context.access;

  // Indicator if the target is private
  context.private;

  // Indicator if the target is static
  context.static;

  // A function that can be called to register a callback that is evaluated 
  // either when the class is defined or when an instance is created
  context.addInitializer(() => {});
}
```

Apart from metadata, the context object for methods has a useful function called `addInitializer`. It’s a way to hook into the beginning of the `constructor` (or the initialization of the class itself if we’re working with statics).

## Class decorators

Class decorators are used to decorating classes, and their corresponding types are declared as follows:

```typescript
type ClassDecorator = (
  target: Function,
  context: {
    kind: 'class';
    name: string | undefined;
    addInitializer(initializer: () => void): void;
  }
) => Function | void;
```

For a class decorator, it receives two parameters `target` and `context`, where the value of the `kind` property of the `context` parameter object is `'class'`. 

When the context is set to type `ClassDecoratorContext`, the decorator can only be used on a class (not a method or property).

```typescript
@withDate
class Person {
  name: string;

  constructor(name: string) {
    this.name = name;
  }
  greet() {
    console.log(`Persons name is ${this.name}`);
  }
}

function withDate<T extends { new (...args: any[]): {} }>(
  baseClass: T,
  context: ClassDecoratorContext
) {
  return class extends baseClass {
    date = new Date().toISOString();
    constructor(...args: any[]) {
      super(...args);
      console.log('Adding employment date to ' + baseClass.name);
    }
  };
}

const person = new Person('Bob');
console.log(person);
// Adding employment date to Person
// Person { name: 'Bob', date: '2023-11-29T21:15:08.112Z' }
```

Definition of withDate Decorator:

`withDate` is defined as a function that takes two parameters: `baseClass` and `context`.
`baseClass`: The class to which the decorator is applied.
`context`: Additional context information (not used in this example).
`<T extends { new (...args: any[]): {} }>` is a generic constraint ensuring that `T` is a constructable type (i.e., a class).

Decorator Functionality:

The `withDate` function returns a new anonymous class (let's call it `ExtendedClass`) that extends the `baseClass`. This `ExtendedClass` adds a new property `date`, initialized to the current date and time in ISO format. The constructor of `ExtendedClass` calls the constructor of the `baseClass` using `super(...args)` and then executes additional logic (logging a message in this case).

When the `Person` class is decorated with `@withDate`, it is effectively replaced with the `ExtendedClass`. When you create a `new Person` instance, the constructor of the `ExtendedClass` is invoked. This constructor first calls the original `Person` constructor (assigning name), then adds the date property and logs the message.

## Property/Field decorators 

The attribute decorator is used to decorate the attributes of the class, and its corresponding type is declared as follows:

```typescript
type ClassFieldDecorator = (
  target: undefined,
  context: {
    kind: 'field';
    name: string | symbol;
    access: { 
      get(): unknown; 
      set(value: unknown): void;
    };
    static: boolean;
    private: boolean;
  }
) => (initialValue: unknown) => unknown | void;
```

The property decorator, also receives two parameters `target` and `context`, where the value of the `kind` property of the `context` parameter object is `'field'`. Note the target will always be undefined for field decorators.

```typescript
type Task = {
  name: string;
  level: 'low' | 'medium' | 'high';
};

class Manager {
  @withInitialTask
  tasks: Task[] = [];
}

function withInitialTask<T, V extends Task[]>(
  target: undefined,
  context: ClassFieldDecoratorContext<T, V>
) {
  return function (args: V) {
    args.push({
      name: 'Do something',
      level: 'high'
    });
    return args;
  };
}

const manager = new Manager();
console.log(manager);
// Manager { tasks: [ { name: 'Do something', level: 'high' } ] }
```

## Decorator factories

The role of the decorator factory is to produce decorators, and its essence is a factory function. After calling the factory function, the decorator function will be returned, so the decorator factory is a higher-order function.

```typescript
type Task = {
  name: string;
  level: 'low' | 'medium' | 'high';
};

class Manager {
  @withTask({
    name: 'task',
    level: 'low'
  })
  tasks: Task[] = [];

  @withInitialTask()
  extraTasks: Task[] = [];
}

const manager = new Manager();
console.log(manager);

function withTask(task: Task) {
  return function <T, V extends Task[]>(
    target: undefined,
    context: ClassFieldDecoratorContext<T, V>
  ) {
    return function (args: V) {
      args.push(task);
      return args;
    };
  };
}

function withInitialTask() {
  return withTask({
    name: 'task',
    level: 'high'
  });
}
```

## Method decorators

The method decorator is used to decorate the method of the class, and its corresponding type declaration is as follows:

```typescript
type ClassMethodDecorator = (
  target: Function,
  context: {
    kind: 'method';
    name: string | symbol;
    access: { 
      get(): unknown 
    };
    static: boolean;
    private: boolean;
    addInitializer(initializer: () => void): void;
  }
) => Function | void;
```

Example:

```typescript
class Person {
  name: string;

  constructor(name: string) {
    this.name = name;
  }
  @loggedMethod
  greet() {
    console.log(`Persons name is ${this.name}`);
  }
}

function loggedMethod(originalMethod: any, context: ClassMethodDecoratorContext) {
  function replacementMethod(this: any, ...args: any[]) {
    console.log('Log: Entering method.');
    const result = originalMethod.call(this, ...args);
    console.log('Log: Exiting method.');
    return result;
  }
  return replacementMethod;
}

const person = new Person('Bob');
person.greet();
// Log: Entering method.
// Persons name is Bob
// Log: Exiting method.
```

## Accessor decorators

In addition to ordinary methods, a class may also contain special setter or getter methods. We can also develop corresponding decorators for setter/getter methods. Before developing specific setter/getter decorators, let’s take a look at their type definitions:

```typescript
type ClassSetterDecorator = (
  value: Function,
  context: {
    kind: 'setter';
    name: string | symbol;
    access: {
      set(value: unknown): void
    };
    static: boolean;
    private: boolean;
    addInitializer(initializer: () => void): void;
  }
) => Function | void;


type ClassGetterDecorator = (
  value: Function,
  context: {
    kind: 'getter';
    name: string | symbol;
    access: {
      get(): unknown
    };
    static: boolean;
    private: boolean;
    addInitializer(initializer: () => void): void;
  }
) => Function | void;
```

To be completed when I can find some better examples...

```typescript
// Todo
```
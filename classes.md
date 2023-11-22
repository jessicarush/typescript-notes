# Classes and OOP 

## Table of contents

<!-- toc -->

- [Typing classes](#typing-classes)
- [Access modifiers `public`, `private` and `protected`](#access-modifiers-public-private-and-protected)
- [Classes can use interfaces or types](#classes-can-use-interfaces-or-types)
- [Implements clauses](#implements-clauses)
- [Extends clauses](#extends-clauses)
- [Abstract classes](#abstract-classes)

<!-- tocstop -->

## Typing classes 

```typescript
class Contact {
  // These type declarations are for TypeScript to know what properties 
  // and types should be on instances of the class.
  name: string;
  email: string;

  // These types are specifically checking the args passed in when the 
  // instance in created.
  constructor(name: string, email: string) {
    this.name = name;
    this.email = email;
  }

  greeting(day: string) {
      console.log(`Hello ${this.name}, happy ${day}!`);
  }
}

// It makes sense to let this type be inferred as its so obvious
let contact = new Contact('Bob', 'email@email.com');
```

## Access modifiers `public`, `private` and `protected`

TypeScript has `public` keyword which can be used as a shorthand syntax that allows you to declare and initialize class properties directly in the constructor parameters:

```typescript
class Contact {
  constructor(public name: string, public email: string) {
    // No need for explicit property initialization
  }

  greeting(day: string) {
    console.log(`Hello ${this.name}, happy ${day}!`);
  }
}
```

`private` properties are only accessible from within the class. They are also not accessible to subclasses (extends).This keyword can also be added to methods.

```typescript
class Contact {
  constructor(public name: string, private email: string) {
    // No need for explicit property initialization
  }

  doSomething() {
    console.log(this.email);
  }
}

let contact = new Contact('Bob', 'email@email.com');
console.log(contact.name);
console.log(contact.email); // TypeScript error!
```

`protected` is the same as `private`, with the exception that subclasses can also access them.

## Classes can use interfaces or types

You can very much define classes that use other interfaces or types:

```typescript
interface ScrumMaster {
  name: string;
  holdScrumMeeting(): void;
}

interface Manager {
  name: string;
  holdScrumMeeting(): void;
  somethingPrivate(): void;
}

class Project {
  name: string;
  budget: number;
  scrumMaster: ScrumMaster;

  constructor(name: string, budget: number, scrumMaster: ScrumMaster) {
    this.name = name;
    this.budget = budget;
    this.scrumMaster = scrumMaster;
  }

  holdProjectMeeting() {
    this.scrumMaster.holdScrumMeeting();
  }
}
const scrumMaster: ScrumMaster = {
  name: 'Bob',
  holdScrumMeeting() {
    console.log('Playing scrum...');
  }
};

const manager: Manager = {
  name: 'Maeve',
  holdScrumMeeting() {
    console.log('Playing scrum...');
  },
  somethingPrivate() {
    console.log('Should not be accessible in project');
  }
};

const xProject = new Project('X project', 100, scrumMaster);
const yProject = new Project('Y project', 500, manager);
```

They only difference between using an interface vs a type alias, it that the interface can be extended. It also seems to be more common to use interfaces with classes in general.

## Implements clauses

In TypeScript, the difference between using an interface to define the shape of an object and using implements in a class to enforce that the class conforms to the structure of an interface is a matter of *defining a contract* versus *implementing a contract*.

**Defining a Contract**: When you use an interface, you are defining a contract or a shape that objects can follow. This doesn't create a tangible class or object but rather specifies what properties and methods should be present in objects that adhere to this interface.

**Implementing a Contract**: When a class uses implements with an interface, it's making a commitment to fulfill the contract defined by that interface. The class must provide concrete implementations of all the properties and methods declared in the interface.

You can use an `implements` clause to check that a class satisfies a particular interface:

```TypeScript
interface User {
  timeout: number;
  doSomething: () => void;
};

class Admin implements User {
  timeout = 60;
  doSomething () {
    console.log('Doing...');
  };
}
```

I personally would rather see the types defined in the class, but I could see this being useful if you had some same interface that you wanted to apply to multiple classes.

Classes may also `implement` multiple interfaces, e.g. `class C implements A, B {}`.

Implementing an interface with an optional property doesn’t create that property:

```typescript
interface A {
  x: number;
  y?: number;
}
class C implements A {
  x = 0;
}
const c = new C();
c.y = 10; // Property 'y' does not exist on type 'C'.
```

## Extends clauses

We can inherit from other classes using `extends` (this is a JavaScript feature). But TypeScript gives us a way to say we are intentionally overriding a base class method using the `override` keyword. This helps prevent typos and makes intent clear.

```typescript
class Contact {
  name: string;
  email: string;

  constructor(name: string, email: string) {
    this.name = name;
    this.email = email;
  }

  greeting() {
    console.log(`Hello ${this.name}`);
  }
}

class Family extends Contact {
  relationship: string;

  constructor(name: string, email: string, relationship: string) {
    super(name, email);
    this.relationship = relationship;
  }
  // This tells typescript that I intend to override a method in Contact
  override greeting() {
    console.log(`Hi`);
  }
}
```

## Abstract classes

In TypeScript, abstract classes are used as a base class from which other classes may be derived. They may not be instantiated directly.

Purpose: An abstract class typically includes abstract methods or properties that must be implemented by its subclasses. This is useful for defining a blueprint for a set of classes that share common functionalities but also have their own specific implementations.

```typescript
abstract class Animal {
  move(): void {
    console.log('Moving along!');
  }
  abstract makeSound(): void;
}

class Dog extends Animal {
  override makeSound(): void {
    console.log('Woof!');
  }
}

let dog = new Dog();
dog.move(); // Moving along!
dog.makeSound(); // Woof!
```

Note: for some reason TypeScript is fussier when using abstract classes in that I have to put all the return void types in there. You

